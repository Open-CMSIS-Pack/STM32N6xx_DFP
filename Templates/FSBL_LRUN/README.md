
# FSBL_LRUN Template for STM32N6 series

The [first stage bootloader (FSBL)](https://community.st.com/t5/stm32-mcus/stm32n6-fsbl-explained/ta-p/764307) is a key component in the boot process of STM32N6 microcontrollers. It is responsible for initializing the system, configuring the hardware, and loading the application code from external memory into the internal or external memories for execution. 

This FSBL LRUN *csolution template project* can be used to build a firmware application (`Appli.cproject.yml`) that loads from external Flash and executes from internal RAM (LRUN).
The `ExtMemLoader.cproject.yml` implements a *flash algorithm* that generates a binary library capable of programming an application into external memory.

## Introduction

The bootROM copies FSBL image from external Flash (Octo SPI Flash Memory) into internal RAM (AXI SRAM2) and begins execution to initialize the caches and configure the clocks. Once this is complete, the application binary is copied from external flash into internal RAM (AXI SRAM2). After the copy operation finishes, the application begins execution.

## Steps to Create, Build, Load and Debug using the Basic Template csolution project

> **Prerequisites:**
>
>- **Required Packs:**
>   - [Keil.STM32N6xx_DFP](https://github.com/Open-CMSIS-Pack/STM32N6xx_DFP)
>   - Board specific pack
>     > STM32N6570-DK board: [Keil.STM32N6570-DK_BSP](https://github.com/Open-CMSIS-Pack/STM32N6570-DK_BSP)
>- **Required CMSIS Tools and Extensions:**
>   - Arm CMSIS Solution 1.64.2
>   - Arm CMSIS Debugger 1.3.0
>- **Required ST tools and Firmware Package:**
>   - [STM32CubeMX 6.16.1](https://www.st.com/en/development-tools/stm32cubemx.html)
>     - [STM32Cube_FW_N6 1.3.0](https://www.st.com/en/embedded-software/stm32cuben6.html)
>   - [STM32CubeProgrammer 2.21.0](https://www.st.com/en/development-tools/stm32cubeprog.html)
>     - STM32_SigningTool_CLI: Verify the environment variable `STM32_PRG_PATH` points to the folder that contains `STM32_SigningTool_CLI.exe`

## Project Creation in VSCode

### In Activity bar under CMSIS view click **Create Solution**

> `target` is Device name or Board name.

- Select `target` and **FSBL_LRUN** (from DFP) and click **Create**
- Select **AC6** compiler and click **OK**
- Launch STM32CubeMX (CMSIS view → expand Project `target` → expand and locate Device:CubeMX component) and click **Run Configuration Generator**

## Configuration in STM32CubeMX

Configure `target` in STM32CubeMX

> [STM32N6570-DK board](https://github.com/Open-CMSIS-Pack/STM32N6570-DK_BSP/blob/main/Examples/FSBL_LRUN/Configuration.md#configuration-in-stm32cubemx) STM32CubeMX configuration 

## Configuration in VSCode

### FSBL/FSBL.cproject.yml

- #### Add linker script

```yaml
  # Linker script definition
  linker:
    - script: ../STM32CubeMX/STM32N657X0HxQ/STM32CubeMX/MDK-ARM/FSBL/stm32n657xx_axisram2_fsbl.sct
      for-compiler: AC6
```

- #### Add post-build command to add header to the generated binary

```yaml
  # Post-build commands to add header
  executes:
    - execute: Generate_trusted_bin
      run: $ENV{STM32_PRG_PATH}/STM32_SigningTool_CLI.exe -bin $input$ -s -nk -of 0x80000000 -align -t fsbl -o $output$ -hv 2.3
      input:
        - $bin()$
      output:
        - $OutDir()$/FSBL-trusted.bin
```

> The OTP configuration for flash source selection is configurable via fuses in BOOTROM_CONFIG_2[8:5], OTP_WORD11 using **STM32CubeProgrammer**. Requires the **default** boot configuration to have sNOR device attached boot. For more information, please check [UM3234](https://www.st.com/resource/en/user_manual/um3234-how-to-proceed-with-boot-rom-on-stm32n6-mcus-stmicroelectronics.pdf).

### STM32CubeMX/STM32N657X0HxQ/FSBL.cgen.yml

- #### Comment redundant files

```yaml
  # - file: ./STM32CubeMX/Middlewares/ST/STM32_ExtMem_Loader/core/memory_wrapper.c
  # - file: ./STM32CubeMX/Middlewares/ST/STM32_ExtMem_Loader/core/systick_management.c
  # - file: ./STM32CubeMX/Middlewares/ST/STM32_ExtMem_Loader/MDK-ARM/FlashDev.c
  # - file: ./STM32CubeMX/Middlewares/ST/STM32_ExtMem_Loader/MDK-ARM/FlashPrg.c
  # - file: ./STM32CubeMX/Middlewares/ST/STM32_ExtMem_Loader/STM32Cube/stm32_device_info.c
  # - file: ./STM32CubeMX/Middlewares/ST/STM32_ExtMem_Loader/STM32Cube/stm32_loader_api.c
```

### Appli/Appli.cproject.yml

- #### Add linker script

```yaml
  # Linker script definition
  linker:
    - script: ../STM32CubeMX/STM32N657X0HxQ/STM32CubeMX/MDK-ARM/Appli/stm32n657xx_LRUN.sct
      for-compiler: AC6
```

- #### Add post-build command to add header to the generated binary

```yaml
  # Post-build commands to add header
  executes:
    - execute: Generate_trusted_bin
      run: $ENV{STM32_PRG_PATH}/STM32_SigningTool_CLI.exe -bin $input$ -s -nk -of 0x80000000 -align -t fsbl -o $output$ -hv 2.3
      input:
        - $bin()$
      output:
        - $OutDir()$/Appli-trusted.bin
```

### STM32CubeMX/STM32N657X0HxQ/Appli.cgen.yml

- #### Comment following redundant files (temporarily issue with cmsis toolbox extension)

```yaml
  # - file: ./STM32CubeMX/Middlewares/ST/STM32_ExtMem_Loader/core/memory_wrapper.c
  # - file: ./STM32CubeMX/Middlewares/ST/STM32_ExtMem_Loader/core/systick_management.c
  # - file: ./STM32CubeMX/Middlewares/ST/STM32_ExtMem_Loader/MDK-ARM/FlashDev.c
  # - file: ./STM32CubeMX/Middlewares/ST/STM32_ExtMem_Loader/MDK-ARM/FlashPrg.c
  # - file: ./STM32CubeMX/Middlewares/ST/STM32_ExtMem_Loader/STM32Cube/stm32_device_info.c
  # - file: ./STM32CubeMX/Middlewares/ST/STM32_ExtMem_Loader/STM32Cube/stm32_loader_api.c
```

### STM32CubeMX/STM32N657X0HxQ/STM32CubeMX/Appli/Src/main.c

- #### Add code to blink LED

```c
    /* USER CODE BEGIN 3 */
    HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
    HAL_Delay(500);
   }
    /* USER CODE END 3 */
```

### ExtMemLoader/ExtMemLoader.cproject.yml

- #### Modify TrustZone mode from secure to "off"

- #### Add linker script

```yaml
  # Linker script definition
  linker:
    - script: ../STM32CubeMX/STM32N657X0HxQ/STM32CubeMX/MDK-ARM/ExtMemLoader/stm32n6xx_extmemloader_mdkarm.sct
      for-compiler: AC6
```

- #### Add post build commands to move .axf to root

```yaml
  # Post-build commands
  executes:
    - execute: Copy_ExtMemLoader_to_root
      run: ${CMAKE_COMMAND} -E copy $input$ $output$
      input:
        - ../out/ExtMemLoader/STM32N657X0HxQ/Debug/ExtMemLoader.axf
      output:
        - ../ExtMemLoader.axf
```

### STM32CubeMX/STM32N657X0HxQ/ExtMemLoader.cgen.yml

- #### Comment redundant file

```yaml
  # - file: ./STM32CubeMX/MDK-ARM/startup_stm32n657xx_fsbl.c
```

> [STM32N6570-DK board](https://github.com/Open-CMSIS-Pack/STM32N6570-DK_BSP/blob/main/Examples/FSBL_LRUN/Configuration.md#configuration-in-vscode) VSCode configuration 

## Build and Load in VSCode

### In Activity bar under CMSIS click **Manage Solution Settings**

- Select **ExtMemLoader** Target Set and click **Build solution**
  - **ExtMemLoader** project should successfully build to have configured flash algorithm (check in root folder if ExtMemLoader.axf file appears)
- Continue with select **FSBL_Appli** Target Set
- Ensure **ST-Link@pyOCD** Debug Adapter is selected and **Update launch.json and tasks.json** checkbox is selected and click **Save** then click **Build solution**
  - FSBL and Appli projects should successfully build into out folder
  - Set the boot mode configuration in **development mode** and reset board
  - To flash `target` click **Views and More Actions** and click **Load application to target**
  - Set the boot mode configuration in **flash mode** and reset board
  - Configured LED should blink (in Appli/Src/main.c)

## Debug in VSCode

- To debug application in:
  - **FLASH MODE:**
    - Set the boot mode configuration in **flash mode** and reset board
    > To flash an unprogrammed (virgin) `target`, ensure that the board is in development mode.
    - Open .vscode\launch.json file and modify configuration named "STLink@pyOCD (launch)" under **initCommands** and **customResetCommands** commands:
      - Modify the command name from **tbreak main** to **thbreak main**
    - Click **Load & Debug application** button and now program should wait in main function to start debug
    - With Continue (F5) button, LED should blink in flash mode

  - **DEVELOPMENT MODE:**
    - Set the boot mode configuration in **development mode** and reset board
    - Open .vscode\launch.json file and modify configuration named "STLink@pyOCD (launch)"
      - Comment line

      ```jsonc
      // "preLaunchTask": "CMSIS Load",
      ```

      - add commands into initCommands

      ```json
      "initCommands": [
          "monitor reset halt",
          "load out/Appli/<target>/Debug/Appli.hex",
          "set $pc = Reset_Handler",
          "set $sp = (int) &Image$$ARM_LIB_STACK$$ZI$$Limit",
          "thbreak main"
      ```

      - add commands into customResetCommands

      ```json
      "customResetCommands": [
          "monitor reset halt",
          "maintenance flush register-cache",
          "maintenance flush dcache",
          "load out/Appli/<target>/Debug/Appli.hex",
          "set $pc = Reset_Handler",
          "set $sp = (int) &Image$$ARM_LIB_STACK$$ZI$$Limit",
          "thbreak main",
          "continue"
      ```

    - Save launch.json
    - Click **Load & Debug application** button and now program should wait in main function to start debug
    - With Continue (F5) button, configured LED should blink in development mode

> [STM32N6570-DK board](https://github.com/Open-CMSIS-Pack/STM32N6570-DK_BSP/blob/main/Examples/FSBL_LRUN#build-and-load-in-vscode) Build, Load and Debug configuration 
