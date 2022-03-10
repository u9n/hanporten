# Protokollet

Protokollet baseras på IEC62056-21 Mode D. Vissa skillnader förekommer som du kan läsa mer om längre ned på sidan.
Vill du fördjupa dig mer i protokollet - IEC62056-21, ta då gärna en titt in i vårt 
[open source bibliotek](https://github.com/pwitab/iec62056-21).

## Seriellt gränssnitt

Data läses av via ett seriellt gränssnitt. Till skillnad mot IEC62056-21 ska
teckenformatet vara `8N1` (8 databits, ingen paritet samt en stop bit). Standard för IEC62056-21 är `7E1`

Överföringshastigheten är fast till `115200 baud`

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

När HAN-porten får +5V på `DATA REQUEST` börjar porten skicka data på `DATA OUT`.

Data skickas var 10:e sekund.

Porten är enkelriktad som innebär att det endast går att **läsa** data från elmätaren.

## Meddelandestruktur

Den generella meddelandestrukturen är enligt följande format:

`/ XXXZ Ident CR LF CR LF Data ! CRC CR LF`

Varje del av meddelandet är separerad med `CR LF` d.v.s. line-break. När du utvecklar
program för att läsa av data, är det smidigt att använda en `readline`-funktion.
Denna funktion finns i de flesta programeringsspråk.

1. Första raden: `/ XXXZ Ident CR LF` innehåller information som identifierar mätaren. 
   De första 3 bokstäverna är ett [FLAG ID](https://www.dlms.com/eng/flag-id-list-44143.shtml) 
   som visar vilken tillverkar det är. Z visar baudrate, men den behöver inte 
   stämma eftersom HAN-porten kör på en fast baudrate. Ident är det unika ID:t på mätaren.
   
2. Efter kommer en tom rad som visar att datan börjar.

3. Ett flertal datarader. Se mer under [Datarader](protokollet.md#datarader)

4. Sista raden som börjar med `!` som visar att data delen är slut och sedan 2 bytes CRC för meddelandet.
   CRC:n (Cyclic Redundancy Check) används för att verifiera att man mottagit meddelandet på ett korrekt sätt.
   Alla bytes från `/` till `!`  ska användas i beräkningen för CRC.
   

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

OBIS | Beskrivning | Kommentar
--- | --- | ---
0-0.1.0.0 | Datum och tid | Formatet `YYMMDDhhmmssX`. 
1-0:1.8.0 | Mätarställning Aktiv Energi Uttag. | 
1-0:2.8.0 | Mätarställning Aktiv Energi Inmatning |
1-0:3.8.0 | Mätarställning Reaktiv Energi Uttag |
1-0:4.8.0 | Mätarställning Reaktiv Energi Inmatning |
1-0:1.7.0 | Aktiv Effekt Uttag | Momentan trefaseffekt
1-0:2.7.0 | Aktiv Effekt Inmatning | Momentan trefaseffekt
1-0:3.7.0 | Reaktiv Effekt Uttag | Momentan trefaseffekt
1-0:4.7.0 | Reaktiv Effekt Inmatning | Momentan trefaseffekt
1-0:21.7.0 | L1 Aktiv Effekt Uttag | Momentan effekt
1-0:22.7.0 | L1 Aktiv Effekt Inmatning | Momentan effekt
1-0:41.7.0 | L2 Aktiv Effekt Uttag | Momentan effekt
1-0:42.7.0 | L2 Aktiv Effekt Inmatning | Momentan effekt
1-0:61.7.0 | L3 Aktiv Effekt Uttag | Momentan effekt
1-0:62.7.0 | L3 Aktiv Effekt Inmatning | Momentan effekt
1-0:23.7.0 | L1 Reaktiv Effekt Uttag | Momentan effekt
1-0:24.7.0 | L1 Reaktiv Effekt Inmatning | Momentan effekt
1-0:43.7.0 | L2 Reaktiv Effekt Uttag | Momentan effekt
1-0:44.7.0 | L2 Reaktiv Effekt Inmatning | Momentan effekt
1-0:63.7.0 | L3 Reaktiv Effekt Uttag | Momentan effekt
1-0:64.7.0 | L3 Reaktiv Effekt Inmatning | Momentan effekt
1-0:32.7.0 | L1 Fasspänning | Momentant RMS-värde
1-0:52.7.0 | L2 Fasspänning | Momentant RMS-värde
1-0:72.7.0 | L3 Fasspänning | Momentant RMS-värde
1-0:31.7.0 | L1 Fasström | Momentant RMS-värde
1-0:51.7.0 | L2 Fasström | Momentant RMS-värde
1-0:71.7.0 | L3 Fasström | Momentant RMS-värde



