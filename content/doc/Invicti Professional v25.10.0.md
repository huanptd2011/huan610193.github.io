# How to Run Invicti (Netsparker) on Linux Using Bottles

## Step 1: Create a New Bottle

1. Open Bottles and click the **+** icon in the top-left corner
2. Choose any name for your bottle
3. Click **Custom** and configure:
   - **Architecture**: 32-bit
   - **Runner**: `lutris-ge` or use the default (`soda 9.0.1`)
4. Click **Create**

## Step 2: Configure Bottle Settings

### Components Tab
1. **DXVK**: Select the latest version (e.g., `dxvk-2.7.1`)
2. **VKD3D**: Select the latest version (e.g., `vkd3d-proton-2.14.1`)

### Display Tab
1. Enable **FidelityFX Super Resolution**:
   - Quality Mode: **Performance**
   - Sharpening Strength: **0**
2. Enable **Discrete Graphics**
3. Click **Advanced Display Settings**, scroll down and set **Renderer** to **Vulkan**

### Performance Tab
1. Enable **Feral Game Mode**
2. Set **Synchronization** to **Fsync**

### Compatibility Tab
1. Click **DLL Overrides** → **Create New DLL Override** and add:
   - `mscoree`
   - `gdiplus`
   
   *(Leave them as configured. Reference image: https://imgur.com/a/l7AxJSQ)*

## Step 3: Install Dependencies

1. Return to the bottle's main menu
2. Click the **Dependencies** tab
3. Install the following:
   - `vcredist2022`
   - `dotnet48` *(takes a long time)*
   - `dotnet40` *(takes a long time)*
   - `allfonts`
   - `gdiplus` *(critical)*
   - `msxml3`

## Step 4: Install Fonts and Registry Files

1. Download and extract this zip file: https://t.me/pwn3rzs_chat/292535
2. Download the registry file: https://t.me/pwn3rzs_chat/292536
3. Copy all **\*.ttf** files to:
   ```
   /home/(USERNAME)/.var/app/com.usebottles.bottles/data/bottles/bottles/(BOTTLE_NAME)/drive_c/windows/Fonts/
   ```
4. Import the registry file through the Registry Editor (as shown in: https://t.me/pwn3rzs_chat/292537)

## Step 5: Install and Run Invicti

1. Return to the bottle's main menu
2. Click the **3 dots** in the top-right corner → **Browse Files**
3. Open two file explorer windows:
   - One for the **Invicti-Professional-v25.10** folder
   - One for the bottle's `drive_c` folder (path from Step 4)
4. Copy the **Invicti-Professional-v25.10** folder into `drive_c/`
5. In Bottles, add a shortcut:
   - Navigate to `drive_c/Invicti-Professional-v25.10/`
   - Double-click **Netsparker.exe**
6. You should now see **Netsparker** in Bottles → Click **Run**

Done!
