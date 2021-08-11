# stm32wb55_ble
BLE functionality of STM32WB55. This project implements a P2P (peer-to-peer, or point-to-point) communication between a BLE peripheral and a base station. Read below for more details.

# Hardware prerequisites:
1. The Nucleo-68 evaluation board from the P-NUCLEO-WB55 pack. MCU: STM32WB55RGV6U (VFQFPN68 package). This board is further referred as "a BLE peripheral" and acts as a GATT server. The MCU is further referred as the STM32WB55.
2. A smartphone based on the Android OS. This node is futher referred as "a central" or as "a base station". The ST BLE Sensor application (available at the Android Play Store) is installed on the smartphone. The base station plays the role of a GATT client.

The STM32WB55 is a dual-core MCU with a Cortex-M4 (CPU 1) for the end-user application and a Cortex-M0+ (CPU 2) for the BLE stack.

This project requires the stm32wb5x_BLE_Stack_full_fw.bin wireless stack binary to be programmed into the RF subsystem (CPU 2). For more details on how to flash this binary into the CPU, read, for example, UM2550, a user manual from ST Microelectronics.

# Software:
The code running on the STM32WB55 is written in the STM32CubeIDE. This IDE integrates STM32CubeMX, a graphical software configuration tool with GUI, allowing to generate the C initialization code. This project uses HAL, Hardware Abstraction Layer, API to control the STM32WB55's hardware blocks.

# Acknowledgements
This project is based on the public video tutorials posted on YouTube by ST Microelectronics.

# References:
STM32WB Workshop - 3 How to add BLE functionality. Available online: https://www.youtube.com/watch?v=zNfIGh30kSs&t=1192s. Last opened: August 10, 2021.
