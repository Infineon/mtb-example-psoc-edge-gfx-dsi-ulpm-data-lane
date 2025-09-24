[Click here](../README.md) to view the README.

## Design and implementation

This project uses [4.3-inch Capacitive Touch IPS Display for Raspberry Pi, DSI Interface, 800 x 480 pixel](https://www.waveshare.com/4.3inch-dsi-lcd.htm) from Waveshare. The LCD houses a Chipone ICN6211 display controller and uses the MIPI DSI interface.

The design of this application is minimalistic to get started with code examples on PSOC&trade; Edge MCU devices. All PSOC&trade; Edge E84 MCU applications have a dual-CPU three-project structure to develop code for the CM33 and CM55 cores. The CM33 core has two separate projects for the secure processing environment (SPE) and non-secure processing environment (NSPE). A project folder consists of various subfolders, each denoting a specific aspect of the project. The three project folders are as follows:

**Table 1. Application projects**

Project | Description
--------|------------------------
*proj_cm33_s* | Project for CM33 secure processing environment (SPE)
*proj_cm33_ns* | Project for CM33 non-secure processing environment (NSPE)
*proj_cm55* | CM55 project

<br>

In this code example, at device reset, the secure boot process starts from the ROM boot with the secure enclave (SE) as the root of trust (RoT). From the secure enclave, the boot flow is passed on to the system CPU subsystem where the secure CM33 application starts. After all necessary secure configurations, the flow is passed on to the non-secure CM33 application. Resource initialization for this example is performed by this CM33 non-secure project. It configures the system clocks, pins, clock to peripheral connections, and other platform resources. It then enables the CM55 core using the `Cy_SysEnableCM55()` function and puts itself (CM33) into DeepSleep mode. The main firmware executes on the CM55 core.

The CM55 application drives the LCD and renders the image using PSOC&trade; Edge graphics subsystem. The graphics subsystem of PSOC&trade; Edge MCU houses an independent 2.5D graphics processing unit (GPU), a display controller (DC), and a MIPI DSI host controller with MIPI D-PHY physical layer interface.

The `main()` function of CM55 application initializes all the system clocks and resources using BSP initialization function. It configures the retarget-io middleware to use the debug UART, initializes user LED2, and user button 1. Once all the initializations are complete, it enables interrupt on user button 1 press event, creates a FreeRTOS task `cm55_ns_gfx_task` and start the scheduler. 

The `cm55_ns_gfx_task` configures the graphics subsystem and locks entering into DeepSleep mode. It then initializes the LCD panel and displays an Infineon logo image (832x480 pixel) on the LCD in DSI Video Burst mode. On user button 1 (**USER BTN1**) press, the LCD panel turns OFF, and the DSI PHY data lane enters in ULPM mode. After 6 seconds in OFF mode, the ULPM on the data lane is exited, the LCD switches back ON and display the same image as before with a text notifying that the exit from ULPM is successful. You can re-exercise the DSI ULPM by pressing the **USER BTN1** on the kit.

The image `infineon_logo`(*infineon_logo.h*) is of format RGB565 and is initially copied from flash to system SRAM frame buffer. The display controller (DC) continuously reads the LCD frame buffer from the system SRAM and sends it to the DSI host. The DSI host in turn sends the data in burst mode (DCS commands) via the DSI PHY to the display, which then decodes the DSI packets and display it on the LCD.

- **USER LED2 ON (green color):** DSI PHY data lane in ULPM mode
- **USER LED2 OFF:** DSI PHY data lane in run mode