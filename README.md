
This github repository contains [Ki-Cad](https://kicad.org) and other auxiliary files for building a solar charging circuit, designed to charge a Lithium Iron Phosphate (LiFePo4) battery, while the battery powers an ESP32 device.

![solar-esp32.png](/images/solar-esp32.png)

This circuit is designed to support the following features:

* Over-charge protection, to prevent over-charging of LiFePo4 battery;
* Maximum Power Point Tracking (MPPT), to optimize power extraction from the solar cell;
* Under-voltage protection, to prevent the ESP32 from draining the battery when it cannot deliver sufficient voltage;
* (optional) Charge and charge-done status LEDs;
* (optional) Monitoring of solar cell voltage output;
* (optional) Monitoring of battery voltage;
* (optional) ESP32 LED status pin;
* Small form-factor (1" square)

At the heart of the charging board is a [Consonance CN3801](http://www.consonance-elec.com/pdf/datasheet/DSE-CN3801.pdf), an integrated circuit that uses pulse-width modulation to control a P-channel MOS-FET, providing the appropriate voltage and current to charge a LiFePo4 battery, based on the battery's charge state.  This IC uses the "constant voltage" technique, via a voltage divider on the solar-esp32 charging board to track the maximum power point of the solar panel.  This voltage can be regulated, depending on the solar panel in operation.  (See the [Component Assembly](#component-assembly) section, below, for more information.)

> IMPORTANT.  This circuit is designed for use with Lithium Iron Phosphate (LiFePo4) batteries only.  Do _not_ attempt to use this charging circuit with any other battery chemistry.

With the Ki-Cad files in this repository, you can order a printed circuit board (PCB) at one of your favorite PCB print shops (I use [OshPark](https://oshpark.com)) and get a few copies sent to you for small change.  You will be responsible for procuring and assembling the components on the board (See the [Component Assembly](#component-assembly) section, below).

Using this circuit, I have been able to run an ESP32 indefinitely using a 1.5mAh 18650 LiFePo4 battery and a 6V 2W solar panel, with the ESP32 in deep sleep, waking at 1 minute intervals, and briefly connecting to WiFi.  Your mileage may vary, depending your application's power needs.

# Typical Circuit

The following block diagram illustrates a typical circuit using this charging board (solar-esp32):

               +------------+
               |   solar    |
               |   panel    |
               |            |
               |            |
               |            |
               |    +  -    |
               |    o  o    |
               +----|--|----+
                    |  |
                    |  |
                  +-|--|------------+              +---------------+
                  | o  o            |              |               |
                  | +  -     +3.3v o----------------o +3.3v        |
                  | Solar     vbat o----------------o io32 (ex)    |
                  |  in       vsol o----------------o io33 (ex)    |
                  |            led o----------------o io2 (ex)     |
                  | BAT         en o----------------o en           |
                  | +  -       gnd o----------------o gnd          |
                  | o  o            |              |               |
                  +-|--|------------+              +---------------+
                    |  | solar-esp32                          ESP32
                    |  |
                    |  +-----------------------------+
                    |                                |
                    |   +-------------------------+  |
                    |   |                         |  |
                    +-}| +    LiFePo4 battery   - |{-+
                        |                         |
                        +-------------------------+

The solar-esp32 charging board contains three sets of pins:

* solar in (+/-)
* bat (+/-)
* esp32 (+3.3v/vbat/vsol/led/en/gnd)

To use this charging board:

1. Connect the `+` and `-` terminals of the solar cell to the corresponding "Solar in" `+` and `-` pins on the solar-esp32 charging board;
1. Connect the `+` and `-` terminals of the LiFePo4 battery to the corresponding "BAT" `+` and `-` pins on the solar-esp32 charging board;
1. Connect the "+3.3v" pin of the ESP32 to the corresponding "+3.3v" pin on the solar-esp32 charging board;
1. Connect the "gnd" pin of the ESP32 to the corresponding "gnd" pin on the solar-esp32 charging board;
1. Connect the "en" (or "chip_pu") pin of the ESP32 to the corresponding "en" pin on the solar-esp32 charging board.

With this configuration, the ESP32 should run off the LiFePo4 battery, as long as the battery can supply adequate voltage (e.g., 3.2v), and the battery should take a charge from the solar panel as long as the panel can deliver sufficient power to the battery (optimized to the specifications of the solar panel).

See below for more information about the "vsol", "vbat", and "led" pins on the solar-esp32 charging board.

## Optional Voltage Monitoring

The solar-esp32 charging board supports pins for monitoring supply voltage from the solar panel (`vsol`), as well as supply voltage from the battery (`vbat`).

The voltage ranges for these pins are provided in the following table:

| solar-esp32 Pin | Voltage range (typical) |
| --- | --- |
| `vsol` | 0v - 1.205v |
| `vbat` | 0v - 1.1v |

You may design an application to read these voltage levels using the ADC pins (32-39) on the ESP32, or using an external ADC circuit (e.g., the ADS1115).  Note that if you use the ADC on the ESP32, you should only take readings when the WiFi is inactive.  You should adjust the ADC attenuation based on the voltage ranges of the two pins.

## Optional LED Status

The solar-esp32 charging board contains pads for 3 status LEDs:

* Charging LED (red, typically) -- This LED on when the solar panel is delivering charge to the battery (also blinks when the battery is disconnected and the board is receiving adequate voltage from the solar panel);
* Charing done LED (blue, typically) -- This LED on when the battery is done charging and is no longer delivering current to the battery;
* ESP32 status LED -- This LED is connected to the `led` pin on the solar-esp32 charging board and can be used by the application to deliver status to the user about the application, using any suitable I/O pin on the ESP32 device.

# Component Assembly

Users are responsible for procurement and assembly of components used on the solar-esp32 charging board.

Many of these components have fixed values that should not need to be adjusted.

The following components, however, may depend on the choice of solar panel, LiFePo4 battery, and attenuation settings for the ADC used to monitor battery voltage.

| Components | Subsystem | Recommended values | Description |
| --- | --- | --- | --- |
| R3, R5 | MPPT voltage divider | 300k, 100k ohms | This set of resistors should set the voltage at the MPPT pin to 1.205v when the solar panel reaches its maximum power point. Recommended values are based on a solar panel MPP of 5.2v, but your solar panel may have a different MPP. |
| R1 | current sense resistor | 0.24 ohms| This resistor sets the maximum current to deliver to the battery when charging.  The recommended value is based on a 500mA limit, e.g., for a 1.5mAh 18650 LiFePo4 battery. |
| R7, R9 | battery voltage divider | 226k, 100k ohms | This set of resistors sets the voltage range used to monitor battery voltage.  The recommended values sets the maximum expected voltage (~3.6v) at the `vbat` pin to 1.1v.  You may adjust these resistors, depending on the attenuation used to read voltage off this pin. |

This repository includes a [Bill of Materials](solar-sep32-BOM.csv) that you can use to select components from your favorite parts warehouse.
