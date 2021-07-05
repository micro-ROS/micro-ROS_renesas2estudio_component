<br/>

<a href="https://www.zephyrproject.org">
   <p align="center">
      <img width="40%" src=".images/renesas_logo.gif">
      <img width="40%" src=".images/microros_logo.png">
   </p>
</a>
<br/>

# micro-ROS for Renesas e<sup>2</sup> studio

[![ ](https://github.com/micro-ROS/micro_ros_renesas_testbench/actions/workflows/ci_galactic.yml/badge.svg?branch=foxy)](https://github.com/micro-ROS/micro_ros_renesas_testbench/actions/workflows/ci_galactic.yml)


This package eases the integration of [micro-ROS](https://micro.ros.org/) in a [Renesas e<sup>2</sup> studio](https://www.renesas.com/us/en/software-tool/e-studio). This components targets [Renesas RA family](https://www.renesas.com/us/en/products/microcontrollers-microprocessors/ra-cortex-m-mcus), an ARM Cortex-M based MCU series, enabling a full micro-ROS compatibility for developing robotics and IoT applications.

- [micro-ROS for Renesas e<sup>2</sup> studio](#micro-ros-for-renesas-esup2sup-studio)
  - [- Known Issues/Limitations](#--known-issueslimitations)
  - [Supported platorms](#supported-platorms)
  - [Requeriments](#requeriments)
  - [Getting started](#getting-started)
  - [Integrating micro-ROS in your project](#integrating-micro-ros-in-your-project)
  - [Micro XRCE-DDS transport configuration](#micro-xrce-dds-transport-configuration)
    - [USB transport](#usb-transport)
    - [Serial transport](#serial-transport)
  - [License](#license)
  - [Known Issues/Limitations](#known-issueslimitations)
---
## Supported platorms

| MCU                                                                                                                                                                             | Family    | Reference board                                                                                                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [RA6M5](https://www.renesas.com/us/en/products/microcontrollers-microprocessors/ra-cortex-m-mcus/ra6m5-200mhz-arm-cortex-m33-trustzone-highest-integration-ethernet-and-can-fd) | RA Series | [EK-RA6M5](https://www.renesas.com/us/en/products/microcontrollers-microprocessors/ra-cortex-m-mcus/ek-ra6m5-evaluation-kit-ra6m5-mcu-group) |

## Requeriments

1. [Renesas e<sup>2</sup> studio](https://www.renesas.com/us/en/software-tool/e-studio) for Linux <sup>*</sup>
2. FSP board packs for Renesas e<sup>2</sup> studio: [Guide](Install_packs.md).

<small><sup>*</sup>Currently only support for Linux is avaible</small>

## Getting started

TODO: Link to a repo with example projects

## Integrating micro-ROS in your project

micro-ROS can be integrated with a Renesas e<sup>2</sup> studio project following these steps:

1. Clone this repository in your Renesas e<sup>2</sup> studio project folder.

2. Go to `Project -> Properties -> C/C++ Build -> Settings -> Build Steps Tab` and in `Pre-build steps` add:


```bash
cd ../micro_ros_renesas2estudio_component/library_generation && ./library_generation.sh "${cross_toolchain_flags}"
```


3. Add <b>micro-ROS include directory</b>.
   <details>
   <summary>Steps</summary>

      In `Project -> Settings -> C/C++ Build -> Settings -> Tool Settings Tab -> GNU ARM Cross C Compiler -> Includes`

      - add `"${workspace_loc:/${ProjName}/micro_ros_renesas2estudio_component/libmicroros/include}"` in `Include paths (-l)`

   </details>



4. Add the **micro-ROS precompiled library**.
   <details>
   <summary>Steps</summary>

     In `Project -> Settings -> C/C++ Build -> Settings -> Tool Settings Tab -> GNU ARM Cross C Linker -> Libraries`
      - add `"${workspace_loc:/${ProjName}/micro_ros_renesas2estudio_component/libmicroros}"` in `Library search path (-L)`
      - add `microros` in `Libraries (-l)`
   </details>

6. **Add the following source** code files to your project, dragging them to source folder:
      - `extra_sources/microros_time.c`
      - `extra_sources/microros_allocators.c`
      - `extra_sources/microros_allocators.h`
      - `extra_sources/microros_transports.h`

7. Configure **micro-ROS time reference**.

   <details>
   <summary>Steps</summary>

   Configure `g_timer0` as an `r_agt`
      1. Double click on the `configuration.xml` file of your project and go to the `Components` tab.
      2. Filter for `timer` and enable the `r_agt` timer:

         ![image](.images/Enable_timer.png)

      3. Go to the `Stacks` tab, then select `New Stack -> Driver -> Timers -> Timer Driver on r_agt`.
      4. Modify the clock period on the component properties (`Module g_timer0 Timer Driver on r_agt -> General -> Period`) to `0x800000`
      5. Modify the count source on the component properties (`Module g_timer0 Timer Driver on r_agt -> General -> Count Source`) to `PCLKB`
      6. Modify the interrupt callback on the component properties (`Module g_timer0 Timer Driver on r_agt -> Interrupt -> Callback`) to `micro_ros_timer_cb`
      7. Modify the underflow interrupt priority on the component properties (`Module g_timer0 Timer Driver on r_agt -> Interrupt -> Underflow Interrupt Priority`) to `Priority 15`

         ![image](.images/Timer_configuration.png)

      8. Make sure that PCLKB is set to 12500 kHz in `Clocks` tab:

         ![image](.images/Configure_timer_clock.png)

      9.  Save the modification using `ctrl + s` and click on `Generate Project Content`.

   </details>

8. Configure **micro-ROS memory requirements**:

   <details>
   <summary>Bare Metal and ThreadX</summary>

   Configure the stack and heap size:

   1. On the `configuration.xml` menu, go to the `BSP` tab.
   2. Go to the `RA Common` section and set the `Main stack size (bytes)` and `Heap size (bytes)` fields to 5000:

      ![image](.images/Configure_memory.png)

   3. Save the modification using `ctrl + s` and click on `Generate Project Content`.
   </details>

   <details>
   <summary>FreeRTOS</summary>
      TODO
   </details>

9.  Configure the transport: [Detail](##Micro-XRCE-DDS-transport-configuration)

10. Build and run your project

## Micro XRCE-DDS transport configuration
### USB transport
1. Copy the following files file to the source directory:
      - `extra_sources/microros_transports/usb_transport.c`
      - `extra_sources/microros_transports/usb_descriptor.c`
2. Double click on the `configuration.xml` file of your project and go to the `Components` tab.
3. Filter for `usb` and enable the `r_usb_basic` and `r_usb_pcdc` components:

   ![image](.images/Enable_usb.png)

4. Go to the `Stacks` tab, then select `New Stack -> Middleware -> USB -> USB PCDC driver on r_usb_pcdc`.
5. Go to `Clocks` tab and configure `UCLK` clock to match 48MHz (Match the values on the highlighted boxes):

   ![image](.images/Configure_usb_clock.png)

6. Save the modification using `ctrl + s` and click on `Generate Project Content`.

### Serial transport
1. Copy the following files file to the source directory:
      - `extra_sources/microros_transports/uart_transport.c`
2. Double click on the `configuration.xml` file of your project and go to the `Components` tab.
3. Filter for `uart` and enable the `r_sci_uart` component.
4. Go to the `Stacks` tab, then select `New Stack -> Driver -> Connectivity -> r_src_uart`.
5. Go to the component properties and configure the Tx/Rx pinout:

   ![image](.images/Configure_serial.png)

6. Save the modification using `ctrl + s` and click on `Generate Project Content`.

## License

This repository is open-sourced under the Apache-2.0 license. See the [LICENSE](LICENSE) file for details.

For a list of other open-source components included in this repository,
see the file [3rd-party-licenses.txt](3rd-party-licenses.txt).

## Known Issues/Limitations

There are no known limitations.
