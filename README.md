# LinearActuatorArduinoControl

## Description
This Repo conatins all necessary information to control a Firgelli Linear Actuator with a custom Human Machine Interface (HMI) using an Arduino UNO R3 controller.

The provided Arduino R3 UNO c++ code is specificially to control this hardware in the context of a custom reciprocating robot. The purpose of this robot is to:
1. Open a parallel plate MEA test fixture to allow loading of an MEA
1. Close test fixture and apply a defined pressure to the MEA
1. Maintain the defined pressure on the MEA during operation/experimentation

To operate in manual mode, all triggers and setpoints for Actuator operation

For full automation, all triggers and setpoints for Actuator operation

## Materials
### Main Components
- [Firgelli Linear Actuator](https://www.firgelliauto.com/products/linear-actuators?variant=843926567)
- [Linear Actuator High Current Driver](https://www.firgelliauto.com/products/high-current-dc-motor-drice-43a)
- [Arduino UNO R3](https://www.firgelliauto.com/products/arduino-uno-r3-microcontroller)
- [OMEGA Load Cell](https://www.dwyeromega.com/en-us/1-5-in-od-through-hole-compression-load-cells/p/LC8150)
- [OMEGA Strain Meter](https://www.dwyeromega.com/en-us/strain-meter-6-digit-display-with-3-relay-outputs/p/DPS20-Series?srsltid=AfmBOorU0sL5Dnp0OtEN925Pg16xj5YpMygIKmax6tw7DnrusbJYX7FH)

### Miscellaneous Components
- Power Sources
- Electrical Wiring
- Potentiometer
- SPST Switch
- Aligator clips, or other wire splicing mechanism


----------------------------------------------------

## Modes of Operation
It utilizes a custom HMI to implement 3 usage modes: 
- External Emergency Stop (No Action) 
- Manual Positioning
- Maintain Force 
