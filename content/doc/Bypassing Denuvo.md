So, I tried to reverse the HyperDbg fork and the loader. When i say loader I am referring to the hyperkd.sys wrapper which is a very simple loader/initializer. Here is what it does:
- Registers a kernel callback to detect system sleep and reinitialize the VMM upon wake up. So the hypervisor dosen't drop
- spawns a background thread (CounterThread) idk what else it does but i can assume it manages the hypervisor updates
- It also has a cpu feature detection routine which initializes flags for avx/sse support

Driver entry decompiled: 
```DriverEntry
NTSTATUS __stdcall DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath) {
  NTSTATUS Status; 
  unsigned __int16 UnicodeNameBuf[2]; 
  const wchar_t *PowerCallbackPath; 
  struct _OBJECT_ATTRIBUTES ObjAttr; 
  PCALLBACK_OBJECT CbPointer; 

  UnicodeNameBuf[0] = 0x28; // size
  UnicodeNameBuf[1] = 0x2A; // max Size
  PowerCallbackPath = L"\\\\Callback\\\\PowerState";
  
  ObjAttr.Length = 48;
  ObjAttr.ObjectName = (PUNICODE_STRING)UnicodeNameBuf;
  ObjAttr.RootDirectory = 0;
  ObjAttr.Attributes = 576; // OBJ_CASE_INSENSITIVE | OBJ_PERMANENT
  *(_OWORD *)&ObjAttr.SecurityDescriptor = 0;

  DriverObject->DriverUnload = (PDRIVER_UNLOAD)DriverUnload;

  if ( (unsigned __int8)CheckSystemSessionInformation() ) 
  {
	Status = ExCreateCallback(&CbPointer, &ObjAttr, 0, 1u);
    if ( Status < 0 )
      return Status;
	
    g_PowerCallbackRegistration = ExRegisterCallback(CbPointer, PowerStateCallbackHandler, 0);
    ObfDereferenceObject(CbPointer);
	
    if ( !g_PowerCallbackRegistration )
      return STATUS_INSUFFICIENT_RESOURCES;
	
    // This is the core vmm initialization
    if ( (unsigned __int8)InitializeHypervisorWrapper() )
    {
	       // Counter thread I was talkin about
	       if ( PsCreateSystemThread(&g_CounterThreadHandle, 0, 0, 0, 0, CounterUpdater, 0) >= 0 )
		return STATUS_SUCCESS;
    }
    
    ExUnregisterCallback(g_PowerCallbackRegistration);
  }
  return STATUS_UNSUCCESSFUL;
}
```

Everything summerized:
DriverEntry() checks if KVAS is disabled if yes it registers power state callback, then calls LoaderInitVmmAndDebugger(), Calls hyperhv.dll -> VmxInitialize(), Checks if the cpu support vmx via cpuid, Builds MTRR map, Creates EPT identity map for each cpu core, Executes vmxon on each core and finally, on each core via DPC it:
	VmxVirtualizeCurrentSystem(),
	Configures VMCS (guest and host states),
	Saves OrigLstar = __readmsr(IA32_LSTAR),
	__writemsr(IA32_LSTAR, SyscallHook) <- LSTAR hook installed,
	Sets msr bitmap to intercept LSTAR reads,
	Sets exception bitmap for DB
	__vmx_vmlaunch() <- GG system now runnin in VM

So that was the loader, starting with the hypervisor, it uses magic leaf value for communication. Normally, a cpuid would be intercepted by the hypervisor and nothing would happen ofc, but if we hardcode it to check the leaf for a specific value and do some stuff we can make it do stuff that way.

So what it is making it do specifically here is, for the magic value 0x69696969, it capture this process's cr3 register. This process will be protected by the hypervisor now. For magic value 0x336933, The game passes a um address in rcx, so when denuvo calls NtQuerySystemInformation, the hypervisor will redirect execution to THIS rcx address instead of the kernel. magic value 0x336943 tells hypervisor which syscall value corresponds to NtQuerySystemInformation, same thing they do with other magic value to register fake handler for QueryFullAttributesFile and QueryFullAttributesFile. Finally, for 0x1337 magic value gives the pid in rdx for KUSER_SHARED_DATA spoofing.  
 
I couldn't decompile it because it contains raw vm instructions like vmread which IDA cannot map to pseudocode, but here is the disassembly, this is exactly what it does:
```CpuidHandler
mov     [rsp+saved_rbx], rbx
push    rdi
sub     rsp, 50h
mov     rax, cs:__security_cookie
xor     rax, rsp
mov     qword ptr [rsp+58h+SecurityCookie+0Ch], rax

mov     rdi, [rcx+28h] ; rdi = VCpu->Regs (rcx contains VCpu pointer)
xor     r9d, r9d
mov     eax, 802h ; VMCS_GUEST_CS_SELECTOR
mov     r8d, [rdi] ; r8 = vcpu->regs->rax
mov     [rsp+20h], r9
mov     [rsp+58h+saved_rax_idx+4], r9

vmread  rax, rax ; __vmx_vmread(VMCS_GUEST_CS_SELECTOR, &rax)
mov     [rsp+58h+saved_rax_idx+4], rax 
test    al, 3
jz      ExecuteRealCpuidAndSpoof ; ignore magic leaves if cpl = 0 

mov     rax, [rcx+28h]; rax = vcpu->regs
cmp     qword ptr [rax], 69696969h ; vcpu->Regs->rax == 0x69696969 (lol)
jnz     short CheckNextMagicLeaf

; registers if 6969
cmp     cs:TargetCR3, r9 // already registered??
jnz     ReturnToGuest_Cleanup

mov     ebx, 6802h ; VMCS_GUEST_CR3 code
vmread  qword ptr [rsp+20h], rbx ; __vmx_vmread(VMCS_GUEST_CR3, &guest_cr3)
call    LayoutGetCurrentProcessCr3 ; get cr3 of the calling process
cmp     rax, [rsp+20h]
jnz     ReturnToGuest_Cleanup

; Bingooo
vmread  qword ptr cs:TargetCR3, rbx
jmp     ReturnToGuest_Cleanup

```

This was just the communication, No spoofing so far. here is how it begins
### Memory Integrity bypass

Denuvo hashes sections of it's code in memory at runtime, so if you patch a single byte to skip the license or anything, well then... gl mate. Also this is ONE OF THE antitamper mechanism not the only one.

How this crack works around it is by exploiting EPT. Since the hypervisor can control the EPT translation layer, it can show different physical pages depending on whether the CPU is reading or executing.

EPT maps the memory in 2mb chunks, so to control a single page hypervisor first needs to split the 2mb page into 4kb pages, So a total of 512 4kb pages. Once again, I couldn't get the decomplication but here is the disassembly version of how it handles it:

```EptSplitLargePage
mov     rbx, r8 ; rbx = PhysAddr

mov     rcx, rbx
mov     rax, rbx
shr     rcx, 1Eh ; PhysAddr >> 30 (PML3 array index)
shr     rax, 15h ; PhysAddr >> 21 (PML2 array index)
and     ecx, 1FFh
and     eax, 1FFh

add     rcx, 201h
shl     rcx, 9
add     rcx, rax

mov     rax, [rbp+228h] ; EptPageTable ptr 
mov     rcx, [rax+rcx*8] ; rcx = EptPageTable->PML2[PML3][PML2]

test    cl, cl ; largepage?
jns     loc_140011CDB

mov     rdx, rsi
mov     rcx, 0x2
call    PoolManagerRequestPool
mov     rdi, rax ; rdi = NewSplit structure
 
; then identity map the 512 replacement entries. basically: NewSplit->PML1[EntryIndex].PageFrameNumber = ((TargetEntry->PageFrameNumber * SIZE_2_MB) / PAGE_SIZE) + EntryIndex

mov     rax, [rdi+1048h] ; NewSplit->PML1 array physical addr
mov     [rcx], rax ; overwrite TargetEntry
  
mov     rcx, [rbp+220h]
call    VmmInvept ; flush tlb

mov     al, 1 

retn
```

After splitting the 2mb EPT entry into 4kb pages, the hypervisor installs a modified mapping for the target page. it maintains two views for the same guest physical page:
- original page (for denuvos integrity checks)
- modified page (contains the patched instructions)
It works because EPT permissions are configured like, execution is allowed on the modified page and data reads trigger an EPT violation, causing a vmexit and upon an EPT violation the hypervisor intercepts the access and fetches it data from the unmodified page which will ofc pass the integrity check. 

Now to place these hooks dynamically, when the guest executes "VMCALL_SET_HIDDEN_CC_BREAKPOINT" a vmexit occurs. The hypervisor dispatches the request through a handler (a jump table based on the VMCALL number), Below is the disasm code (again- lol):

```CreateEptHook
xor     esi, esi
mov     ecx, 681Eh ; VMCS_GUEST_RIP 

; Allocate tracking hooked pages
mov     [rsp+2Ch+arg_28], rsi
mov     dword ptr [rsp+2Ch+arg_28], esi
mov     [rsp+2Ch+arg_18], sil
mov     [rsp+2Ch+arg_18], dh

; Read the addr where the vmcall happened 
vmread  rcx, rcx ; extract guest RIP
mov     r8d, 1
mov     eax, 1
mov     [rsp+2Ch+arg_28], rcx
mov     dword ptr [rsp+2Ch+arg_28], ecx

lea     rdx, [rsp+2Ch+arg_18]
lea     edx, [rsp+2Ch+arg_18]

; MemoryMapperReadMemorySafe(VirtualTarget, &FakePageContents, PAGE_SIZE) - Not gonna go very deep but basically copies the real memory into tracking struct
call    MemoryMapperReadMemorySafe

mov     rcx, rdi ; Virtual machine state context (VCpu)
mov     ecx, edi
call    EptSplitLargePage

test    al, al
jz      loc_140013B3E

; check/apply the breakpoint
; *(BYTE *)TargetAddressInFakePageContent = 0xcc;
cmp     [rsp+2Ch+arg_18], 0CCh
jnz     loc_140013CFB

; Update PML1 ept perms
loc_140013B3E:
    mov     ecx, [rdi+38h]
    call    EptGetPml1Entry ; does what the name says
    
    test    al, al
    jnz     loc_140013CFB
    
    mov     rcx, rdi
    mov     ecx, edi
    call    ApplyEptPermissions ; does what the name says
    
    call    EptInveptSingleContext ; no surprise, once again, does what the name says
    
    add     rsp, 20h
    pop     rdi
    pop     rsi
    pop     rbx
    retn
```
In Brief, It:
- retrieves the guest RIP using vmread
- copies the original page content into a shadow buffer
- ensures the target page is split into 4kb ept entries
- injects breakpoint in the shadow copy
- updates the corresponding apt PML1 entry perms
- invalidates the ept context to apply changes/
### LSTAR Syscall Hook
Denuvo calls "NtQuerySystemInformation" to gather system info like loaded modules, process list, system firmware and "NtQueryFullAttributesFile" to check for certain files idk license-related or something.

On x86_64 systems when a program calls a syscall for NtQuerySystemInformation, when cpu executes the syscall instruction the cpu jumps to the address stored in the IA32_LSTAR msr. this should normally point to nt!KiSystemCall64 but in our case the hypervisor replaces LSTAR with its own handler at vmlaunch:
```
backupLstar = __readmsr(IA32_LSTAR);
__writemsr(IA32_LSTAR, LstarHook);
```
Below is the assembly handler extracted from hyperhv.dll:
``` SyscallHookHandler

SyscallHook:
	cmp     eax, cs:g_TargetSyscallId_1 ; cmp syscall with our target1
	jz      short Handle_Syscall_Target_1

	cmp     eax, cs:g_TargetSyscallId_2 ; cmp syscall with our target1
	jz      short Handle_Syscall_Target_2

	jmp     short Forward_To_Original_Kernel ; windows gonna deal with it


Handle_Syscall_Target_1:
	mov     rax, cr3
	cmp     rax, cs:g_TargetProcess_CR3 ; caller game target process??
	jnz     short Restore_And_Bypass_1

	mov     rax, 7FFFFFFFFFFFh
	cmp     rcx, rax
	ja      short Restore_And_Bypass_1 ; if returning to km, stop spoofing
   
	test    r10, r10
	jnz     short Restore_And_Bypass_1
   
	mov     rax, rcx
	mov     rcx, cs:g_FakeUsermodeHandler_1 ; Overwrite ret addr
    sysret

Restore_And_Bypass_1:
	mov     eax, cs:g_TargetSyscallId_1
	jmp     short Forward_To_Original_Kernel

Handle_Syscall_Target_2:
	mov     rax, cr3
	cmp     rax, cs:g_TargetProcess_CR3
	jnz     short Restore_And_Bypass_2

	mov     rax, 7FFFFFFFFFFFh
	cmp     rcx, rax
	ja      short Restore_And_Bypass_2

The hijack
	mov     rax, rcx
    mov     rcx, cs:g_FakeUsermodeHandler_2 ; Overwrite ret addr with seconmdary handler
	sysret

Restore_And_Bypass_2:
    mov     eax, cs:g_TargetSyscallId_2
	jmp     short $+2

Forward_To_Original_Kernel:
	jmp     qword ptr cs:g_Original_KiSystemCall64_Ptr ; real windows kerneel dispatch handler

``` 

sysretq instruction returns from a syscall back to um, Normally the kernel SHOULD execute it after finishing the syscall but here the hypervisors hook executes it before the kernel runs. So basically:
-  Denuvos code calls NtQuerySystemInformation(SystemBasicInformation
-  cpu executes syscall -> lands in SyscallHook
-  Hook checks EAX matches? cr3? Yes? voilla!! hijack.
-  sysretq returns directly to um but rcx now points to the fake handler
-  The fake handler in the games address space returns modified system information

### CPUID Spoofing
Denuvo reads cpuid to fingerprint the hardware, From what my eyes tell me it uses the CPU family, model, stepping and brand string as as part of the fingerprint to bind it to a encrypted ticket So, if someone donates an offline denuvo acitvation ticket AND their figerprints we could bypass this specific validation check. Since the hypervisor is ruling, for the target process only, hypervisor can return hardcoded values that match the specs tied to the donor PC, in this Black Myth Wukong crack, the donor PC specs were intel core i9-10900k, here is the disassembly:

```HwidSpoof
CPUID_Leaf_1_Spoof:
    mov     rax, [rcx+28h]
	mov     qword ptr [rax], 0A0655h ; Family 6, Model 0xA5 (Comet Lake)
	mov     qword ptr [rax+18h], 200800h ; Max logical CPUs
	mov     qword ptr [rax+8], 7BFAFBFFh ; Feature flags - disables XGETBV/OSXSAVE
	mov     ecx, 0BFEBFBFFh
	mov     [rax+10h], rcx          ; Feature flags
	jmp     Exit_Cpuid_Handler

CPUID_Leaf_80000002_Spoof: ; Intel(R) Core(TM)
	mov     rax, [rcx+28h]
	mov     qword ptr [rax], 65746E49h ; "Inte"
	mov     qword ptr [rax+18h], 2952286Ch ; "l(R)"
	mov     qword ptr [rax+8], 726F4320h ; " Cor"
	mov     qword ptr [rax+10h], 4D542865h ; "e(TM"
	jmp     Exit_Cpuid_Handler

CPUID_Leaf_80000003_Spoof: ; i9-10900K CPU
	mov     rax, [rcx+28h]
	mov     qword ptr [rax], 39692029h ; ") i9"
	mov     qword ptr [rax+18h], 3930312Dh ; "-109"
	mov     qword ptr [rax+8], 204B3030h ; "00K "
	mov     qword ptr [rax+10h], 20555043h ; "CPU "
	jmp     Exit_Cpuid_Handler

CPUID_Leaf_80000004_Spoof: ;  @ 3.70GHz
	mov     rax, [rcx+28h]
	mov     qword ptr [rax], 2E332040h ; " @ 3."
	mov     qword ptr [rax+18h], 48473037h ;"70GH"
	mov     qword ptr [rax+8], 7Ah ; "z"
	mov     qword ptr [rax+10h], 0
	jmp     Exit_Cpuid_Handler
```

### KUSER_SHARED_DATA Spoofing

At virtual address 0x7FFE0000 windows maps a page called KUSER_SHARED_DATA and every um program can read it without a syscall, The problem here is, since there is no syscall we don't get to intercept it the way we did before, It contains:
- os version numbers like major, minor build and stuff
- System time, interrupt time tick count (yes. denuvo does use timing based anti tampering technique)
- CPU feature flags and performance counters
to make our lives harder, denuvo reads these fields too. But if you remember from the first part, The loader created a CounterUpdater thread. It basically remaps the games KUSER_SHARED_DATA page and writes the donor version data into it. I also found out this thread runs in an infinite loop with 1ms sleeps

```CounterUpdater
void __fastcall TargetProcess_SharedData_Thread(PVOID StartContext)
{
  PEPROCESS Process;
  LARGE_INTEGER Interval;
  struct _KAPC_STATE ApcState;
  PMDL MemoryDescriptorList;
  __int64 *PteAddress;

  Interval.QuadPart = -10000i64;
  
  while ( !byte_14002010 )
  {
    if ( qword_14008A500 )
    {
      if ( PsLookupProcessByProcessId((HANDLE)qword_14008A500, &Process) >= 0 )
      {
        KeStackAttachProcess(Process, &ApcState); // Shift context to the game's address space
        
        PteAddress = (__int64 *)GetPteViaCr3Walker(0x7FFE0000i64, 1i64, qword_14008A508);
        
        if ( !qword_14008A518 )
        {
          *PteAddress |= 0x100ui64;
          
          MemoryDescriptorList = IoAllocateMdl((PVOID)0x7FFE0000, 0x1000u, 0, 0, 0i64);
          if ( MemoryDescriptorList )
          {
            MmProbeAndLockPages(MemoryDescriptorList, UserMode, IoWriteAccess);
            
            qword_14008A518 = (__int64)MmMapLockedPagesSpecifyCache(MemoryDescriptorList, KernelMode,
            MmCached, 0i64, 0, HighPagePriority);
          }
        }
        
        if ( qword_14008A518 )
        {
          Apply_SharedData_Spoofs((void *)qword_14008A518);
        }
        
        KeStackDetachProcess(&ApcState);
        ObfDereferenceObject(Process);
      }
    }
    KeDelayExecutionThread(KernelMode, 0, &Interval);
  }
}
```

Then it patches some specific offsets within the KUSER_SHARED_DATA structure:
```KUSER_SHARED_DATA_patch
if (NewKuserSharedData)
{
    UINT64 NewKusdAddress = (UINT64)NewKuserSharedData;

    // offset 0x260 = NtBuildNumber + NtProductType
    // 0x6658 = build 26200
    *(UINT64*)(NewKusdAddress + 0x260) = 0x0000000100006658;

    // suiteMask
    *(UINT32*)(NewKusdAddress + 0x268) = 0x00090001;

    // ntMajorVer = 10
    *(UINT32*)(NewKusdAddress + 0x26C) = 0x0000000A;

    // offset 0x270 = NtMinorVers = 0
    *(UINT32*)(NewKusdAddress + 0x270) = 0x00000000;

    // 0x274 - 0x2AC = ProcessorFeatures[] array, These are bool flags to match i9's feature set like, SSE, SSE2
    *(UINT64*)(NewKusdAddress + 0x274) = 0x0001000001010000;
    *(UINT64*)(NewKusdAddress + 0x27C) = 0x0001010100010101;
    *(UINT64*)(NewKusdAddress + 0x284) = 0x0101010100000100;
    *(UINT64*)(NewKusdAddress + 0x28C) = 0x0000000100000000;
    *(UINT64*)(NewKusdAddress + 0x294) = 0x0101010100000001;
    *(UINT64*)(NewKusdAddress + 0x29C) = 0x0000000000000001;
    *(UINT64*)(NewKusdAddress + 0x2A4) = 0x0000000000000000;
    *(UINT64*)(NewKusdAddress + 0x2AC) = 0x0000000000000000;

    // ActiveProcessorCount = 0x310 (bitmask???)
    *(UINT32*)(NewKusdAddress + 0x2D0) = 0x00000310;

    // QpcFrequency
    *(UINT32*)(NewKusdAddress + 0x2E8) = 0x007FB163;

    // ActiveGroupCount = 16
    *(UINT32*)(NewKusdAddress + 0x3C0) = 0x00000010;
}
```

But ofc fake static data isnt enough denuvo also reads the timing fields from KUSER_SHARED_DATA and if those freeze (like if we replaced the page with a static copy) denuvo will know we are tryna cook something. So the thread that was creating by the loader continuously copies real timing values every 1ms:
``` time_spoofing
liWaitPeriod.QuadPart = -10000LL;

while ( g_bExecuteSyncLoop )
{
    KeDelayExecutionThread(KernelMode, FALSE, &liWaitPeriod);

    if ( g_pSpoofedKuserBase )
    {
        PKUSER_SHARED_DATA pMdlKuser = (PKUSER_SHARED_DATA)g_pSpoofedKuserBase;
        PKUSER_SHARED_DATA pKernelKuser = (PKUSER_SHARED_DATA)0xFFFFF78000000000;

        // sync interrupt time = 0x08
        *(uint64_t*)&pMdlKuser->InterruptTime = *(uint64_t*)&pKernelKuser->InterruptTime;
        pMdlKuser->InterruptTime.High2Time = pMdlKuser->InterruptTime.High1Time;

        // sync system time = 0x14
        *(uint64_t*)&pMdlKuser->SystemTime = *(uint64_t*)&pKernelKuser->SystemTime;
        pMdlKuser->SystemTime.High2Time = pMdlKuser->SystemTime.High1Time;

        // sync RIT event tick count = 0x2E4
        pMdlKuser->LastSystemRITEventTickCount = pKernelKuser->LastSystemRITEventTickCount;

        // sync global tick count = 0x320
        *(uint64_t*)&pMdlKuser->TickCount = *(uint64_t*)&pKernelKuser->TickCount;
        pMdlKuser->TickCount.High2Time = pMdlKuser->TickCount.High1Time;

        // sync time update lock
        pMdlKuser->TimeUpdateLock = pKernelKuser->TimeUpdateLock;

        // sync QPC baseline values =  0x348
        uint64_t qpcBaseline = pKernelKuser->BaselineSystemTimeQpc;
        pMdlKuser->BaselineSystemTimeQpc = qpcBaseline;
        pMdlKuser->BaselineInterruptTimeQpc = qpcBaseline;
    }
}
```

So finally after all this when the denuvo reads KUSER_SHARED_DATA at 0x7FFE0000 it sees::
- Windows 10 build 26200 even if the real os is different
- Processor features matching i9-10900K
- continuously updating timestamps so the timing checks pass
Also another thing, this manipulation is PER PROCESS which means only the games copy of KUSER_SHARED_DATA is affected. Every other process on the sees the real values, the hypervisor changes the physical page that is backing the games 0x7FFE0000 to a private copy.

### SGDT/SIDT Interception

The SGDT register instruction returns the GDTs base address and limit, In a bare metal system the GDT limit is normally 7Fh which is 16 entries. But if the system is running under a hypervisor the host can use a different GDT with another limit and since i'm talking about it, no surprise Denuvo checks for this as well. To work around this, the hypervisor can enable Descriptor Table Exiting which is a VMCS control that causes a vm exit whenever the guest executes SGDT, SIDT, LGDT or LIDT:
```
uint32_t uSecondaryControls = 0x4101008; 

// Check if UMIP is enabled in cr4
// UMIP - bit 11 naturally prevents SGDT/SIDT in um, CPL > 0
unsigned __int64 uGuestCR4 = __readcr4();
uint16_t uGdtLimit = GetCurrentGdtLimit(); // SGDT

if ( (uGuestCR4 & CR4_UMIP_BIT) == 0 ) 
{
	if ( uGdtLimit <= 0x7F ) 
    {
        // enable bit 2 (DESCRIPTOR_TABLE_EXITING) to intercept SGDT/SIDT
        uSecondaryControls |= 0x4;
    }
}

// apply controls aga
inst the VMX MSRs (IA32_VMX_PROCBASED_CTLS2 = 0x48B)
uint32_t uFinalControls = AdjustVmxControls(uSecondaryControls, MSR_IA32_VMX_PROCBASED_CTLS2);

// commit
__vmwrite(VMCS_CTRL_SECONDARY_PROC_BASED, uFinalControls);
```

After this, whenever the game executes SGDT, the vm exit handler kicks in:
```
VOID HandleDescriptorTableIntercept(VIRTUAL_CPU* pVcpu)
{
    VMEXIT_INSTRUCTION_INFO info;
    uint16_t uGuestSelector;
    uintptr_t uGuestCr3;

    __vmx_vmread(VMCS_VMEXIT_INSTRUCTION_INFO, &info.AsUInt32);

    __vmx_vmread(VMCS_GUEST_CS_SELECTOR, &uGuestSelector);
    if ((uGuestSelector & RPL_MASK) != DPL_SYSTEM)
    {
        // Game process (???)
        __vmx_vmread(VMCS_GUEST_CR3, &uGuestCr3);
        if (g_TargetProcessCr3 && uGuestCr3 == g_TargetProcessCr3)
        {
            // SGDT info bit check 
            if ((info.AsUInt32 & 0x3F) == 0) 
            {
                // backup system GDT limit
                __vmx_vmread(VMCS_GUEST_GDTR_LIMIT, (uint64_t*)&pVcpu->uSavedGdtLimit);

                // spoof to 0x7F
                __vmx_vmwrite(VMCS_GUEST_GDTR_LIMIT, 0x7F);
            }
        }
    }

    // temporarily disable exit
    uint32_t uSecondaryProcCtls;
    __vmx_vmread(VMCS_CTRL_SECONDARY_PROC_BASED, &uSecondaryProcCtls);
    uSecondaryProcCtls &= ~0x4; // clear DESCRIPTOR_TABLE_EXITING bit
    __vmx_vmwrite(VMCS_CTRL_SECONDARY_PROC_BASED, uSecondaryProcCtls);

    // trap to reenable interception immediately after intercept
    __vmx_vmwrite(VMCS_CTRL_EXCEPTION_BITMAP, 0x27002);

    uint64_t uPendingDebug;
    uint64_t uGuestRflags;
    uint64_t uDebugCtl;
    __vmx_vmread(VMCS_GUEST_PENDING_DEBUG_EXCEPTIONS, &uPendingDebug);
    __vmx_vmread(VMCS_GUEST_RFLAGS, &uGuestRflags);
    __vmx_vmread(VMCS_GUEST_IA32_DEBUGCTL, &uDebugCtl);

    if ((uGuestRflags & RFLAGS_TF_BIT) && !(uDebugCtl & DEBUGCTL_BTF_BIT))
        uPendingDebug |= (1LL << 14); // set single step
    else
        uPendingDebug |= (1LL << 12); // set Enabled breakpoint

    __vmx_vmwrite(VMCS_GUEST_PENDING_DEBUG_EXCEPTIONS, uPendingDebug);

    uint32_t uInterruptState;
    __vmx_vmread(VMCS_GUEST_INTERRUPTIBILITY_STATE, &uInterruptState);
    uInterruptState |= 0x1; // Blocking by MOV SS
    __vmx_vmwrite(VMCS_GUEST_INTERRUPTIBILITY_STATE, uInterruptState);

    SuppressRipPromotion(pVcpu);
}
```

What i like about this is this technique is it is thoughtful about performance. What I mean is, it disables descriptor table exiting after handling the SGDT, then uses a debug exception to re enable it on the next instruction. So, this way the interception is only active when SGDT runs and it avoids the trapping every single descriptor table access for the whole system.
### Control flow
1. User loads hyperkd.sys driver
		DriverEntry() checks KVAS is disabled
		Registers power state callback
		Calls LoaderInitVmmAndDebugger()
		Calls into hyperhv.dll -> VmxInitialize()
			Checks CPU VMX support via CPUID
			Builds MTRR map (memory type ranges)
			Creates EPT identity map for each CPU core
            Executes VMXON on each core
			On each core via DPC:
	            VmxVirtualizeCurrentSystem()
	                 Configures VMCS (host and guest state)
	                 Saves OrigLstar = __readmsr(IA32_LSTAR)
                     __writemsr(IA32_LSTAR, SyscallHook)
	                 Sets msr  bitmap to intercept LSTAR reads
	                 Sets exception bitmap for DB
                     __vmx_vmlaunch()
		Starts CounterUpdater thread (1ms loop)
2. User starts cracked game executable:
	Game calls CPUID(0x69696969) -> hypervisor captures targetCR3
	Game calls CPUID(0x336943, syscall_id) -> sets NtQuerySystemInfo ID
	Game calls CPUID(0x336933, handler_addr) -> sets fake handler
	Game calls CPUID(0x336944, syscall_id) -> sets NtQueryFullAttributesFile ID
	Game calls CPUID(0x336934, handler_addr) -> sets fake handler
	Game calls CPUID(0x1337, pid) -> sets TargetProcessId for KUSD spoofing
3. CounterUpdater thread activates:
	   Finds TargetProcessId != NULL
	   Attaches to game process -> remaps KUSER_SHARED_DATA
	   Patches os version and cpu features into spoofed page
	   Sets up process exit notification callback
	   start 1ms timing sync loop
4. Game runs with Denuvo active:
	Denuvo reads code pages -> EPT gives original bytes -> hash passes
	CPU executes patched code -> EPT gives fake page -> crack works
	Denuvo calls NtQuerySystemInfo -> LSTAR hook -> fake handler -> spoof
	Denuvo reads CPUID -> hypervisor -> i9-10900K values -> spoof
	Denuvo reads KUSER_SHARED_DATA -> spoofed page -> fake os version -> spoof
	Denuvo executes SGDT -> vm exit -> limit spoofed to 0x7F
	Denuvo reads IA32_LSTAR -> msr bitmap -> original KiSystemCall64
5. Game exits:
	ProcessExitNotifyRoutine fires -> sets ProcessExitCleanup flag
	CounterUpdater thread detects flag -> CleanupKuser()
	Unmaps MDL, frees memory, resets all global state
	Hypervisor now goes dormant, still running but not protecting shit

> AI Transparency: No AI was used in this writeup. Every line of this was me staring at the IDA disassembly 

> Disclaimer: This analysis is completely based on the publicly released DenuvOwO crack for Black Myth: Wukong. They are the real Gs