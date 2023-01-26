# Sonoff ZBDongle Plus (ZBDongle-P) Zigbee stick
This repository contains *experimental* **SDK 6.40.00.13** firmware builds for the Sonoff ZBDongle-Plus (ZBDongle-P) using the CC2652P chipset.  It is intended for use with Zigbee2MQTT but may also work with other Zigbee implementation such as Home Assistant's ZHA integration, however such use is untested.
  
## Disclaimer

This firmware presented here is released for non-production use. Installing this firmware requires knowledge of how to use the tools to flash firmware onto the Sonoff ZBDongle-P.  Please do not attempt to install this firmware if you do not possess knowledge or the tools to do-so, including how to use the built-in recovery facility to restore from a bad flash.

*Use this firmware at your own risk!!* This firmware has only been tested myself on two separate Zigbee2MQTT instances, each with more than 100 devices.  I am not responsible for anything that happens to your device (i.e. bricked) as a result of using this firmware.

There are 2 directories containing pre-built firmware;
```
*./no_hfc* is a standard build without hardware flow control
*./hfc* is a standard build with the hardware flow control patch applied
```

## Goal: Hardware Flow Control

Hardware Flow Control (CTS/RTS) should help improve performance of the ZBDongle-P by offloading flow control management from the CPU. On most systems this will not have a noticeable impact but is believed to help networks with heavier traffic.

The firmware located in the **hfc** folder attempts to enable the hardware flow control feature of the Sonoff ZBDongle-P using a patch maintained by [Koenkk](https://github.com/Koenkk/Z-Stack-firmware/blob/develop/coordinator/Z-Stack_3.x.0/). In order to use this firmware for hardware flow control, the ZBDongle-P must be disassembled to in order to turn on the hardware flow dip switch, and Z2M must be configured to enable it.


## Building it Yourself

To build this firmware yourself, start by following the instruction provided [on Koenkks repo](https://github.com/Koenkk/Z-Stack-firmware/blob/develop/coordinator/Z-Stack_3.x.0/COMPILE.md).  Once the patch has been applied, it recommended to perform a test build of all projects to ensure that the patch has worked.

Once the patch is applied, the following two files in the syscfg directory must be modified as described in the following diffs:

```
diff ti_drivers_config.h ~/syscfg.or/ti_drivers_config.h 
97,104d96
<   /* Owned by CONFIG_DISPLAY_UART as  */
< extern const uint_least8_t CONFIG_GPIO_3_CONST;
< #define CONFIG_GPIO_3 19
< 
< /* Owned by CONFIG_DISPLAY_UART as  */
< extern const uint_least8_t CONFIG_GPIO_4_CONST;
< #define CONFIG_GPIO_4 18
< 
225,226d216
<  *  CTS: DIO19
<  *  RTS: DIO18
```

and
```

diff ti_drivers_config.c ~/syscfg.or/ti_drivers_config.c
269,272c269,270
<     /* Owned by CONFIG_DISPLAY_UART as RTS */
<     GPIO_CFG_OUTPUT_INTERNAL | GPIO_CFG_OUT_STR_MED | GPIO_CFG_OUT_LOW, /* CONFIG_GPIO_4 */
<     /* Owned by CONFIG_DISPLAY_UART as CTS */
<     GPIO_CFG_INPUT_INTERNAL | GPIO_CFG_IN_INT_NONE | GPIO_CFG_PULL_DOWN_INTERNAL, /* CONFIG_GPIO_3 */
---
>     GPIO_CFG_NO_DIR, /* DIO_18 */
>     GPIO_CFG_NO_DIR, /* DIO_19 */
307,308d304
< const uint_least8_t CONFIG_GPIO_3_CONST = CONFIG_GPIO_3;
< const uint_least8_t CONFIG_GPIO_4_CONST = CONFIG_GPIO_4;
813,814c809,810
<     .ctsPin             = CONFIG_GPIO_3,
<     .rtsPin             = CONFIG_GPIO_4,
---
>     .ctsPin             = GPIO_INVALID_INDEX,
>     .rtsPin             = GPIO_INVALID_INDEX,
```

One thing to note, with Code Composer 12.2.0, ti_drivers_config.c is a derived file and cannot be edited manually as it will be overwritten with each build.  To overrride this file, open the znp.syscfg file in each project with the text editor and add the following to the top of each file:
```
scripting.excludeFromBuild("ti_drivers_config.c");
```
The next step is to copy the contents of ./default/syscfg/ti_drivers_config.c from each project to the project root folder (where the ti_drivers_config.h file is located).  Apply the ti_drivers_config.c diff to each project so the changes stick between builds.

Rebuild the project.

