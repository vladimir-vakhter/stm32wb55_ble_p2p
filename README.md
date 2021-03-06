# stm32wb55_ble

# Brief
BLE functionality of STM32WB55. This project implements a P2P (peer-to-peer, or point-to-point) communication between a [BLE peripheral](#peripheral) and a [central](#central). The application allows to eluminate a blue LED on the peripheral and to receive a notification on the central in the event when a push button was pressed on the peripheral. Read below for more details.

# Hardware
## Devices
### Peripheral
The Nucleo-68 evaluation board from the P-NUCLEO-WB55 pack. MCU: STM32WB55RGV6U (VFQFPN68 package). This board is further referred as "a BLE peripheral" and acts as a GATT server. The MCU is further referred as the STM32WB55.
### Central
A smartphone based on the Android OS. This node is futher referred as "a central" or as "a base station". The ST BLE Sensor application (available at the Android Play Store) is installed on the smartphone. The base station plays the role of a GATT client.

## Configuration
The STM32WB55 is a dual-core MCU with a Cortex-M4 (CPU1) for the end-user application and a Cortex-M0+ (CPU2) for the BLE stack. Inter processor communication controller (IPCC) is used for the communication of CPU1 with CPU2. 

This project requires the stm32wb5x_BLE_Stack_full_fw.bin wireless stack binary to be programmed into the RF subsystem (CPU 2). For more details on how to flash this binary into the CPU, read, for example, UM2550, a user manual from ST Microelectronics.

During the configuration phase (STM32CubeIDE, .ioc file), to work with BLE, we need to:
* Activate HSEM (hardware semaphore: go to System Core / HSEM in STM32CubeIDE), which is used for the synchronization of processes and managing the access to shared peripherals (such as registers, RTC, etc.).
* Activate RF1 (Connectivity / RF1).
* Activate RTC (Timers / RTC / Activate Clock Source).
* Enable the middleware (Middleware / STM32WPAN / BLE).
* Clock tree: LSE (32.768 kHz) is used for RTC. System clock MUX should enable HSE_SYS (HSE_PRES = 1) (HSE is 32MHz). Both CPU1 and CPU2 should run at 32 MHz. RFWKP Clock Mux should enable LSE.
* While configuring the MCU (.ioc file), go to Project Manager/Advanced Settings and make sure that STM32_WPAN middleware is activated at the most end, after all the peripherals are enabled.

# Software
## Technologies
The code running on the STM32WB55 is written in the STM32CubeIDE. This IDE integrates STM32CubeMX, a graphical software configuration tool with GUI, allowing to generate the C initialization code. This project uses HAL, Hardware Abstraction Layer, API to control the STM32WB55's hardware blocks.

This application is based on STM32Cube_FW_WB_V1.11.1 firmware package.

## Architecture
The architecture is based on a sequencer. The sequencer provides a simple background scheduling function. We register various tasks, which are then set in a switch-case block upon an corresponding event occurs. A sequencer is used to execute the tasks in background and enter the low-power mode when there is no activity. After we get through all the tasks, we get to the IDLE state (low-power mode).

### The instruction to register and set the task should be:
a. UTIL_SEQ_RegTask(1<<CFG_TASK_SW1_BUTTON_PUSHED_ID, 0, P2PS_Send_Notification)<br>
b. UTIL_SEQ_SetTask(1<<CFG_TASK_SW1_BUTTON_PUSHED_ID, CFG_SCH_PRIO_0);

## Project's directories' content 
Directory                                                            | Function/Content
-------------------------------------------------------------------- | ----------------------------------------------------------------------------------------
Core/Startup/startup_stm32wb55rgvx.s                                 | Reset_Handler()
Core/Src/main.c                                                      | main()<br>Initialize the system (HAL, Clock, etc.)<br>Infinite loop for run mode
Core/Src/app_entry.c                                                 | APPE_Init()<br>Initialize the BSP (LEDs, buttons, etc.)<br>Wait for initialization done
STM32_WPAN/App/app_ble.c                                             | APP_BLE_Init()<br>Initialize the BLE communications<br>Manage GAP events
Core/Inc/app_conf.h<br>Core/Inc/app_entry.h              		         | Parameters configuration file of the application
Middlewares/ST/STM32_WPAN/ble/svc/Src/svc_ctl.c                      | SVCCTL_Init()<br>Initialize GATT services
Middlewares/ST/STM32_WPAN/ble/svc/Src/p2p_stm.c                      | aci_gatt_add_service(), aci_gatt_add_char()<br>Add a service/characteristic
STM32_WPAN/App/p2p_server_app.c                                      | P2P server application<br>GATT even handler
Drivers                                                              | CMSIS and STM32WBxx_HAL_Driver
Core/Src/stm32wbxx_hal_msp.c                                         | HSE tuning
Core/Src/stm32wbxx_it.c                                              | Interrupt service routines / handlers
Utilities/sequencer/stm32_seq.h<br>Utilities/sequencer/stm32_seq.c   | Sequencer
STM32_WPAN/Target/hw_ipcc.c      		                                 | IPCC Driver
Core/Src/system_stm32wbxx.c     		                                 | STM32WBxx system source file
Core/Inc/stm32wbxx_hal_conf.h                                        | HAL configuration file
STM32_WPAN/App/ble_conf.h                                            | BLE Services configuration
STM32_WPAN/App/ble_dbg_conf.h                                        | BLE Traces configuration of the BLE services
Core/Inc/hw_conf.h                                                   | Configuration file of the HW
Core/Inc/utilities_conf.h    		                                     | Configuration file of the utilities
Core/Src/stm32_lpm_if.c			                                         | Low Power Manager Interface
Core/Src/hw_timerserver.c 		                                       | Timer Server based on RTC
Core/Inc/app_conf.h                                                  | Application Configuration File

The above table is based on the information provided in [1] and [2].

# Usage
* Power on the peripheral with this application flashed
* On your smartphone, activare Bluetooth and launch the ST BLE Sensor application
* Connect to the peripheral (name = BLETEST)
* Eluminate the Blue LED on the board from your phone, push SW1 button of the board to get notifications on your phone

Note: advertising stops in 60 seconds automatically if we did not connect our phone to the board. In this case, press the reset button on the board and reconnect it to your phone. The interval of 60 sec until the board becomes non-discoverable may be adjusted by the user to set the desired value.

# Testing
The application was tested to ensure that:
* One is able to toggle an LED on the peripheral by clicking the blue LED icon in the ST BLE Sensor application.
* The ST BLE application is capable to display the notifications sent by the peripheral on each SW1 push button's press event.

## Enable Debugger
To enable the debugger, modify app_conf.h:
1. #define CFG_DEBUGGER_SUPPORTED 1     // keep the debugger enabled while in any low-power mode
2. #define CFG_DEBUG_BLE_TRACE 1        // trace BLE
3. #define CFG_DEBUG_APP_TRACE 1        // trace firmware application
4. #define CFG_DEBUG_TRACE_FULL 1       // the trace are output with the API name, the file name and the line number

# Acknowledgements
This project is based on the public video tutorials posted on YouTube by ST Microelectronics [1].

# References
1. STM32WB Workshop - 3 How to add BLE functionality. Available online: https://www.youtube.com/watch?v=zNfIGh30kSs&t=1192s. Last opened: August 10, 2021.
2. STM32Cube MCU Package for STM32WB series: STM32Cube_FW_WB_V1.11.1\Projects\P-NUCLEO-WB55.Nucleo\Applications\BLE\BLE_p2pServer. Available online: https://www.st.com/en/embedded-software/stm32cubewb.html. Last opened: August 11, 2021.
3. STM32WB Getting Started Series: Part 6, CubeWB Peer to Peer. Available online: https://www.youtube.com/watch?v=WgsXMVZfoO8. Last opened: August 16, 2021.
