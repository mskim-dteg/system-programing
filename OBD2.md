# OBD-II (On-Board Diagnostics II)
## referneces
* wikipedia
  + https://en.wikipedia.org/wiki/On-board_diagnostics
  + https://en.wikipedia.org/wiki/Data_link_connector
  + https://en.wikipedia.org/wiki/OBD-II_PIDs
* https://pinoutguide.com/CarElectronics/
## SAE J1962 connector
* connector types
  + type A

    ![OBD-II type A](https://upload.wikimedia.org/wikipedia/commons/thumb/8/8c/OBD-II_type_A_female_connector_pinout.svg/330px-OBD-II_type_A_female_connector_pinout.svg.png)
    - used for 12-volt vehicles
  + type B

    ![OBD-II type B](https://upload.wikimedia.org/wikipedia/commons/thumb/6/60/OBD-II_type_B_female_connector_pinout.svg/330px-OBD-II_type_B_female_connector_pinout.svg.png)
    - used for 24-volt vehicles
    - Wire placement is identical to type A, but the center groove is split in two
    - type B용 male connector는 type A female connector와 연결 가능
* pin assignments
  + pin 1 - Manufacturer discretion
    - GM: J2411 GMLAN/SWC/Single-Wire CAN.
    - Audi: Switched +12 to tell a scan tool whether the ignition is on.
    - VW: Switched +12 to tell a scan tool whether the ignition is on.
    - Mercedes (K-Line): Ignition control (EZS), air-conditioner (KLA), PTS, safety systems (Airbag, SRS, AB) and some other.
  + pin 2 - J1850 Bus+
    - SAE J1850 PWM and VPW
  + pin 3 - Manufacturer discretion
    - Ethernet TX+ (Diagnostics over IP)
    - Ford DCL(+) Argentina, Brazil (pre OBD-II) 1997–2000, USA, Europe, etc.
    - Chrysler CCD Bus(+)
    - Mercedes (TNA): TD engine rotation speed.
  + pin 4 - Chassis(Battery) ground
  + pin 5 - Signal ground
  + pin 6 - CAN high (ISO 15765-4 and SAE J2284)
  + pin 7 - K line (ISO 9141-2 & ISO 14230-4)
  + pin 8 - Manufacturer discretion
    - Activate Ethernet (Diagnostics over IP)
    - Many BMWs: A second K-line for non OBD-II (Body/Chassis/Infotainment) systems.
    - Mercedes: Ignition
  + pin 9 - Manufacturer discretion
    - GM: 8192 baud ALDL where fitted.
    - BMW: RPM signal.
    - Toyota: RPM signal.
    - Mercedes (K-Line): ABS, ASR, ESP, ETS, BAS diagnostic.
  + pin 10 - J1850 Bus-
    - SAE J1850 PWM only (not SAE 1850 VPW)
  + pin 11 - Manufacturer discretion
    - Ethernet TX- (Diagnostics over IP)
    - Ford DCL(-) Argentina, Brazil (pre OBD-II) 1997–2000, USA, Europe, etc.
    - Chrysler CCD Bus(-)
    - Mercedes (K-Line): Gearbox and other transmission components (EGS, ETC, FTC).
  + pin 12 - Manufacturer discretion
    - Ethernet RX+ (Diagnostics over IP)
    - Mercedes (K-Line): All activity module (AAM), Radio (RD), ICS (and more)
  + pin 13 - Manufacturer discretion
    - Ethernet RX- (Diagnostics over IP)
    - Ford: FEPS – Programming PCM voltage
    - Mercedes (K-Line): AB diagnostic – safety systems.
  + pin 14 - CAN low (ISO 15765-4 and SAE J2284)
  + pin 15 - L line (ISO 9141-2 and ISO 14230-4)
  + pin 16 - Battery power
    - +12 Volt for type A connector
    - +24 Volt for type B connector
* pin summary
  + power and ground: 4, 16
  + CAN bus: 6(+), 14(-)
  + legacy communications
    - Signal ground: 5
    - ISO 9141-2 or ISO 14230-4: 7, 15
    - J1850: 2, 10
  + manufacturer discretion: 1, 3, 8, 9, 11, 12, 13
    - Ethernet (Diagnostics over IP): 3(TX+), 8(Activate), 11(TX-), 12(RX+), 13(RX-)
## [Diagnostic Trouble Codes (DTCs)] - 4-digit, preceded by a letter: P for powertrain (engine and transmission), B for body, C for chassis, and U for network.
## variations
* EOBD (European On-Board Diagnostics)
  - same SAE J1962 diagnostic link connector and signal protocols being used in Europe, but with some differences in the supported PIDs and DTCs.
  - EOBD fault codes : P0xxx (generic), P1xxx (manufacturer-specific)
* JOBD (Japanese On-Board Diagnostics) - used in Japan, similar to OBD-II but with some differences in protocols and connectors.
* ADR 79/01 & 79/02 - Australia
* EMD/EMD+ - California, USA
# tools
* [CANgaroo](https://github.com/OpenAutoDiagLabs/CANgaroo) - A CAN bus sniffer and analyzer for automotive diagnostics.