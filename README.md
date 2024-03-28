# Bootloader for Stm32f303re

💡 This repo for how to make custom bootloader for stm32f303re

- [x] First what is bootloader
- [x] STM32F bootloader Neclueo board
- [x] what is happening when u reset the board
- [x] Boot modes
- [x] Code placement in flash
- [x] Datasheet information
- [x] how to use flasher software from ST microcontrollers
- [x] Create ur custom bootloader

## First what is bootloader ?

- A bootloader is a small software program that resides on a microcontroller and is responsible for loading the main application code. It essentially acts as an intermediary between the microcontroller and the device used to program it.
  Here's a breakdown of why bootloaders are important: - Easier Programming: Traditionally, programming a microcontroller required specialized hardware tools. A bootloader eliminates this need by allowing you to upload new code through a communication interface like USB, serial, or even wirelessly. - Firmware Updates: Bootloaders make firmware updates a breeze. You can develop and upload new versions of the main application code without needing physical access to the microcontroller itself. - Flexibility: Bootloaders provide flexibility during development. If there are bugs in the main program, you can easily upload a fix without going through the entire programming process again.
  (ASK Gemini AI :) )

## STM32F bootloader Neclueo board ?

- STLink: Nucleo boards typically come with an integrated ST-Link debugger/programmer. This STLink acts as a separate device that can program the STM32F microcontroller on the board. In most cases, it's easier and the default method to use the STLink for programming rather than the internal bootloader.

- Internal Bootloader: Some STM32F microcontrollers may have an internal bootloader pre-programmed in their flash memory. This bootloader allows uploading new firmware through a communication interface like UART (serial) but requires additional configuration steps like setting specific pins to enter bootloader mode.

![1](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/d2cb15fe-9b27-4266-98c4-ca65f96a2851)

\*\* ** For stm32f303RE as shown in photo the board has on chip bootloader** \*\*

## what is happening when u reset the board ?

- PC (Program Counter) will be loaded with 0x00000000. So will start from the address 0x00000000.
- Since the address has an Initial Stack Pointer value, It will be fetched to the MSP (Main Stack Pointer). So, that value - will be starting of the stack.
- Then PC will be loaded with the Reset handler’s address.
- After that, the reset handler will perform the below operations.
- Initialize the system.
- Copy the Initialized global variable, static variable (.data) to SRAM.
- Copy the Un-initialized data (.bss) to SRAM and initialize it to 0.
- It Calls main().
- This is how your main() is getting called while power-up or press the reset button. You may understand clearly if you see the execution below.

## Boot modes

- To know boot mode we have to see the reference manual of stm23f303re (RM0316) (link https://www.st.com/resource/en/reference_manual/rm0316-stm32f303xbcde-stm32f303x68-stm32f328x8-stm32f358xc-stm32f398xe-advanced-armbased-mcus-stmicroelectronics.pdf )
- At page 62 u would see this table

![2](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/d19503bc-05c8-4003-86bf-626f335991ab)

you can figure out from the table
if boot1 don't care boot0 == 0 the boot area is main flash memory
if boot1 == 1 boot0 == 0 the boot area is system memory
if boot1 == 0 boot0 == 1 the boot area is Embedded SRAM

- Look at this Embedded bootloader u would find out that ur board connected through USART2 PA2,PA3
  ![3](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/3de06120-ec85-4ca0-8705-129d115c2c38)

## Code placement in flash

- At page 65 u would find Flash module organization which have multiple sectors
- ![4](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/fc490712-8110-4c02-90d3-3c174caffeea)

- usually the IDE start the user application in this section

- but in the examples we will change it to start from different page sector

## Datasheet information for bootloader

- check this application node AN2606 (https://www.st.com/resource/en/application_note/an2606-stm32-microcontroller-system-memory-boot-mode-stmicroelectronics.pdf)
- for ur controller at page 100
- you can use USART2 to make bootloader so follow this steps to test to boot out of IDE

## how to use flasher software from ST microcontrollers

download this software https://www.st.com/en/development-tools/flasher-stm32.html from stm

### follow this steps to test bootloading

      - connect ur board to PC
      - connect pin of boot0 to the vcc to make the flasher see the board
      - as shown in figure
      ![7](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/1fc5fa8e-26cc-467d-81d0-da7f34d1e564)

      - open program after connect the BOOT0 to VCC
     
![5](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/e36a7712-8bf6-4d11-a6a8-ac2d3d193401)

      - press on reset button (Blackbutton) as shown in figure and follow the figures using this binary file

![23](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/7a2716fd-82c9-411c-8852-df2eba535446)
![8](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/1c56a108-cb49-42b1-9129-257be971670e)
![9](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/db7a9e77-959f-4ff4-b12e-0fec7750f8bf)
![11](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/d1645b88-897a-4d7b-a82d-309db4f19650)
![10](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/622ff9d3-542d-4452-b228-df2113f0fad1)

The program is using systick timer to blink the LED if it blinking Bingooo

## Create ur custom bootloader

- First lets Explain what we are going to do :

  - The task is to create a program start with application with multiple options one for bootloader command and the other for user application to something like that we would use Flash module organization used before create multiple section in memory each section is full program it self (vector table + system init + main) once the program start choose the program u want and the bootloader will through u to the starting address holding this program
  - the figure show the addresses of each program and how to mange the space
![2_drawio](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/fcdc2f07-729f-4b89-b2b0-b3356bd0ade5)

![3_drawio](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/32b13d89-d4e2-47d3-8c96-d2fc33a091bd)

  The link below is the project custom bootloader

  
![IMG20221225205208](https://user-images.githubusercontent.com/76526170/209479323-8350920e-1ecf-4d79-b639-0b80fcf16598.gif)
