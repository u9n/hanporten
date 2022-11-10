# Protokollet

Protokollet baseras på IEC62056-21 Mode D. Men några skillnader som anges nedan.
Om du vill fördjupa dig i IEC62056-21 så kan du kolla in vårt 
[open source bibliotek](https://github.com/pwitab/iec62056-21) för 
protokollet.

## Seriellt gränssnitt

Man läser av data via ett seriellt gränssnitt. Till skillnad på IEC62056-21 så skall
teckenformatet vara `8N1` (8 databits, ingen paritet, en stop bit). Standard IEC62056-21 är 7E1

Överföringshastigheten är satt fast till `115200 baud`

!!! example "Kodexempel för seriell port"

    === "Python"
        ```python
        import serial

        serial_port = serial.Serial(
            "/dev/ttyUSB0",  # your serial port name
            baudrate=115200,
            parity=serial.PARITY_NONE,
            stopbits=serial.STOPBITS_ONE,
            bytesize=serial.EIGHTBITS,
            timeout=30
            rtscts=False,
            dsrdtr=False,
            xonxoff=False,
        )
        serial_port.open()
        
        ...
        
        # do something
        
        ...
        
        serial_port.close()
        ```

    === "Node.js"
        ```javascript
        const SerialPort = require('serialport')

        const options = {baudRate: 115200, dataBits: 8, stopBits: 1, parity: none}
        cost serialPort = new SerialPort("/dev/ttyUSB0", options)
        serialPort.open()
        ...
        // do something
        ...
        serialPort.close()
        ```

## Timing

När HAN-porten får +5V på `DATA REQUEST` börjar den skicka data på `DATA OUT`.

Data skickas var 10:e sekund.

Porten är enkelriktad så det är inte möjligt att skicka någon data till mätaren, 
vilket gör det enkelt endast lyssna efter data.

## Meddelandestruktur

Den generella meddelandestrukturen för ett meddelande på HAN-porten är:

`/ XXXZ Ident CR LF CR LF Data ! CRC CR LF`

Varje del av meddelandet är separerat med `CR LF` dvs line-breaks. Så när man gör ett 
program är det smidigt att använda en `readline` funktion som finns i de flesta 
programspråk.

1. Första raden: `/ XXXZ Ident CR LF` innehåller information som identifierar mätaren. 
   De första 3 bokstäverna är ett [FLAG ID](https://www.dlms.com/eng/flag-id-list-44143.shtml) 
   som visar vilken tillverkar det är. Z visar på vilken baudrate, men den behöver inte 
   stämma eftersom HAN-porten kör på en fast baudrate. Ident är det unika ID:t på mätaren.
   
2. Sedan kommer en tom rad som visar att datan börjar.

3. Ett flertal datarader. Se mer under [Datarader](protokollet.md#datarader)

4. Sista raden som börjar med `!` som visar att data delen är slut och sedan 2 bytes CRC för meddelandet.
   CRC:n (Cyclic Redundancy Check) används för att verifiera att man mottagit meddelandet på ett korrekt sätt.
   Alla bytes från `/` till `!`  skall användas i beräkningen för CRC.
   

!!! example "CRC beräkning"

````python
import libscrc

example_data = (
    b"/ELL5\x5c253833635_A\r\n\r\n"
    b"0-0:1.0.0(210217184019W)\r\n"
    b"1-0:1.8.0(00006678.394*kWh)\r\n"
    b"1-0:2.8.0(00000000.000*kWh)\r\n"
    b"1-0:3.8.0(00000021.988*kvarh)\r\n"
    b"1-0:4.8.0(00001020.971*kvarh)\r\n"
    b"1-0:1.7.0(0001.727*kW)\r\n"
    b"1-0:2.7.0(0000.000*kW)\r\n"
    b"1-0:3.7.0(0000.000*kvar)\r\n"
    b"1-0:4.7.0(0000.309*kvar)\r\n"
    b"1-0:21.7.0(0001.023*kW)\r\n"
    b"1-0:41.7.0(0000.350*kW)\r\n"
    b"1-0:61.7.0(0000.353*kW)\r\n"
    b"1-0:22.7.0(0000.000*kW)\r\n"
    b"1-0:42.7.0(0000.000*kW)\r\n"
    b"1-0:62.7.0(0000.000*kW)\r\n"
    b"1-0:23.7.0(0000.000*kvar)\r\n"
    b"1-0:43.7.0(0000.000*kvar)\r\n"
    b"1-0:63.7.0(0000.000*kvar)\r\n"
    b"1-0:24.7.0(0000.009*kvar)\r\n"
    b"1-0:44.7.0(0000.161*kvar)\r\n"
    b"1-0:64.7.0(0000.138*kvar)\r\n"
    b"1-0:32.7.0(240.3*V)\r\n"
    b"1-0:52.7.0(240.1*V)\r\n"
    b"1-0:72.7.0(241.3*V)\r\n"
    b"1-0:31.7.0(004.2*A)\r\n"
    b"1-0:51.7.0(001.6*A)\r\n"
    b"1-0:71.7.0(001.7*A)\r\n!"
)
crc16 = libscrc.ibm(example_data).to_bytes(2, 'big').hex()
>> '7945'
````

## Datarader

En datarad har formatet:

`OBIS ( Värde * Enhet ) CR LF`

### OBIS

OBIS (OBject Identifier System) är en del av DLMS/COSEM och visar vilket värde dataraden gäller.

Formatet är `A-B:C.D.E.F`       (ex: 1-0:1.8.0.255)

Varje del kan vara mellan 0-255. Del `F` är nästan alltid 255 (används ej) och i den 
svenska branchstandarden skickas det inte med.

Värdegrupp | Beskrivning
--- | ---
A | Medíum, Ex: Elektricitet
B | Kanal
C | Mätvärde, Ex: Effekt Fas 1
D | Mätvärdeshantering, Ex: Momentan, Kumulativ.
E | Tariff, Ex: dag/natt
F | Faktureringsperiod eller historisk data.

!!! example "Exempel OBIS"

    1-0:1.8.0.255 -> El, Aktiv effekt förbrukning på alla faser, tidsintegral 1 (energi)

    Så det anger hur mycket energi som konsumerats genom mätaren. (Mätarställning)

### Värde och enhet

Värde och enhet är avdelat med `*`. Vissa värden har ingen enhet (till exempel tid och datum)
och då finns där inget * inom parenteserna.


## Svenska branschrekommendationer

Energiföretagen har tagit fram en 
[branschrekommendation](https://www.energiforetagen.se/forlag/elnat/branschrekommendation-for-lokalt-kundgranssnitt-for-elmatare/) 
om hur HAN-porten borde fungera och vilken data som bör publiceras på HAN-porten.

!!! note "Lite förtydlingar"
   
      Med uttag menas uttag från elnätet. Det vill säga det du förbrukar. Med inmating 
      menas det du matar in i elnätet (till exampel om du har solceller)

      Den sista delen i OBIS har tagits bort då den inte tillför någon information.

      Tiden som visas är i svensk normaltid (UTC+1). Elmätare i Sverige ändrar inte om 
      för sommartid. X anger om det är sommartid eller ej och kan ignoreras.

OBIS | Beskrivning | Kommentar | Exempel-värde
--- | --- | --- | ---
0-0.1.0.0 | Datum och tid | Formatet `YYMMDDhhmmssX` | 210217184019W
1-0:1.8.0 | Mätarställning Aktiv Energi Uttag |  | 00006678.394\*kWh
1-0:2.8.0 | Mätarställning Aktiv Energi Inmatning | | 00000000.000\*kWh
1-0:3.8.0 | Mätarställning Reaktiv Energi Uttag | | 00000021.988\*kvarh
1-0:4.8.0 | Mätarställning Reaktiv Energi Inmatning | | 00001020.971\*kvarh
1-0:1.7.0 | Aktiv Effekt Uttag | Momentan trefaseffekt | 0001.727\*kW
1-0:2.7.0 | Aktiv Effekt Inmatning | Momentan trefaseffekt | 0000.000\*kW
1-0:3.7.0 | Reaktiv Effekt Uttag | Momentan trefaseffekt | 0000.000\*kvar
1-0:4.7.0 | Reaktiv Effekt Inmatning | Momentan trefaseffekt | 0000.309\*kvar
1-0:21.7.0 | L1 Aktiv Effekt Uttag | Momentan effekt | 0001.023\*kW
1-0:22.7.0 | L1 Aktiv Effekt Inmatning | Momentan effekt | 0000.350\*kW
1-0:41.7.0 | L2 Aktiv Effekt Uttag | Momentan effekt | 0000.353\*kW
1-0:42.7.0 | L2 Aktiv Effekt Inmatning | Momentan effekt | 0000.000\*kW
1-0:61.7.0 | L3 Aktiv Effekt Uttag | Momentan effekt | 0000.000\*kW
1-0:62.7.0 | L3 Aktiv Effekt Inmatning | Momentan effekt | 0000.000\*kW
1-0:23.7.0 | L1 Reaktiv Effekt Uttag | Momentan effekt | 0000.000\*kvar
1-0:24.7.0 | L1 Reaktiv Effekt Inmatning | Momentan effekt | 0000.000\*kvar
1-0:43.7.0 | L2 Reaktiv Effekt Uttag | Momentan effekt | 0000.000\*kvar
1-0:44.7.0 | L2 Reaktiv Effekt Inmatning | Momentan effekt | 0000.009\*kvar
1-0:63.7.0 | L3 Reaktiv Effekt Uttag | Momentan effekt | 0000.161\*kvar
1-0:64.7.0 | L3 Reaktiv Effekt Inmatning | Momentan effekt | 0000.138\*kvar
1-0:32.7.0 | L1 Fasspänning | Momentant RMS-värde | 240.3\*V
1-0:52.7.0 | L2 Fasspänning | Momentant RMS-värde | 240.1\*V
1-0:72.7.0 | L3 Fasspänning | Momentant RMS-värde | 241.3\*V
1-0:31.7.0 | L1 Fasström | Momentant RMS-värde | 004.2\*A
1-0:51.7.0 | L2 Fasström | Momentant RMS-värde | 001.6\*A
1-0:71.7.0 | L3 Fasström | Momentant RMS-värde | 001.7\*A
