# Protokollet

Protokollet som skickas på den norska HAN-porten är en DLMS/COSEM DataNotification APDU i en 
HDLC unnumbered information frame

!!! note "Ej verifierat"
    Vi har ingen mätare med denna typ av port och har inte verifierat om allt är helt korrekt.
    Men om någon har en mätare med denna port får de gärna verifiera och meddela oss.

## Datainnehåll

Datan är ett antal rader som liknar datan i den svenska standarden, men formaterad 
mer enligt DLMS. Den innehåller en rad för tid och datum, en rad för identifiering av mätaren
och flera rader som innehåller OBIS, värde, skalär och enhet.

## Seriellt gränssnitt

Data skickas med 2400 baud och i byteformat `8E1`

## Exempeldata

```
7e a10b 41 0883 13 fa7c e6e700
     0f 40000000 00
     010c
         0202 0906 0101000281ff 0a0b 4149444f4e5f5630303031
         0202 0906 0000600100ff 0a10 3733XXXXXXXXXXXXXXXXXXXXXXX13130
         0202 0906 0000600107ff 0a04 36353235
         0203 0906 0100010700ff 06 00000e90 0202 0f00 161b
         0203 0906 0100020700ff 06 00000000 0202 0f00 161b
         0203 0906 0100030700ff 06 0000001c 0202 0f00 161d
         0203 0906 0100040700ff 06 00000000 0202 0f00 161d
         0203 0906 01001f0700ff 10 0091     0202 0fff 1621
         0203 0906 0100470700ff 10 0090     0202 0fff 1621
         0203 0906 0100200700ff 12 0932     0202 0fff 1623
         0203 0906 0100340700ff 12 091e     0202 0fff 1623
         0203 0906 0100480700ff 12 0933     0202 0fff 1623
 95d4 7e

```
## Läs ut datan

HDLC frames är inramade av ascii tilde `~`/`0x7e`. 

Ett enkelt program som läser av ett seriellt gränssnitt kan läsa tills det hittar en 
frame-end och testa att parsa inläst data som en hdlc-frame. Om det inte går att parsa 
fortsätter man läsa tills nästa frame-end.

## Parsa data

```python
from dlms_cosem.prtocol import xdlms
from dlms_cosem.hdlc import frames
from dlms_cosem.utils import parse_as_dlms_data

hdlc_data = b"\x7e.....\x7e"
ui = frames.UnnumberdInformationFrame(hdlc_data)
dn = xdlms.DataNotification.from_bytes(ui.payload[3:])  # The first 3 bytes should be ignored.
result = parse_as_dlms_data(dn.body)


>>>[
     [bytearray(b'\x00\x00\x01\x00\x00\xff'),bytearray(b'\x07\xe3\x0c\x10\x01\x07;(\xff\x80\x00\xff')],  # Datum
     [bytearray(b'\x01\x00\x01\x07\x00\xff'), 1122, [0, 27]],  # OBIS, value, [skalär, enhet]
     [bytearray(b'\x01\x00\x02\x07\x00\xff'), 0, [0, 27]],
     [bytearray(b'\x01\x00\x03\x07\x00\xff'), 1507, [0, 29]],
     [bytearray(b'\x01\x00\x04\x07\x00\xff'), 0, [0, 29]],
     [bytearray(b'\x01\x00\x1f\x07\x00\xff'), 0, [-1, 33]],
     [bytearray(b'\x01\x003\x07\x00\xff'), 75, [-1, 33]],
     [bytearray(b'\x01\x00G\x07\x00\xff'), 0, [-1, 33]],
     [bytearray(b'\x01\x00 \x07\x00\xff'), 2307, [-1, 35]],
     [bytearray(b'\x01\x004\x07\x00\xff'), 2499, [-1, 35]],
     [bytearray(b'\x01\x00H\x07\x00\xff'), 2308, [-1, 35]],
     [bytearray(b'\x01\x00\x15\x07\x00\xff'), 0, [0, 27]],
     [bytearray(b'\x01\x00\x16\x07\x00\xff'), 0, [0, 27]],
     [bytearray(b'\x01\x00\x17\x07\x00\xff'), 0, [0, 29]],
     [bytearray(b'\x01\x00\x18\x07\x00\xff'), 0, [0, 29]],
     [bytearray(b'\x01\x00)\x07\x00\xff'), 1122, [0, 27]],
     [bytearray(b'\x01\x00*\x07\x00\xff'), 0, [0, 27]],
     [bytearray(b'\x01\x00+\x07\x00\xff'), 1506, [0, 29]],
     [bytearray(b'\x01\x00,\x07\x00\xff'), 0, [0, 29]],
     [bytearray(b'\x01\x00=\x07\x00\xff'), 0, [0, 27]],
     [bytearray(b'\x01\x00>\x07\x00\xff'), 0, [0, 27]],
     [bytearray(b'\x01\x00?\x07\x00\xff'), 0, [0, 29]],
     [bytearray(b'\x01\x00@\x07\x00\xff'), 0, [0, 29]],
     [bytearray(b'\x01\x00\x01\x08\x00\xff'), 10049926, [0, 30]],
     [bytearray(b'\x01\x00\x02\x08\x00\xff'), 8, [0, 30]],
     [bytearray(b'\x01\x00\x03\x08\x00\xff'), 6614347, [0, 32]],
     [bytearray(b'\x01\x00\x04\x08\x00\xff'), 5, [0, 32]]]

# Obis could be parsed like:
from dlms_cosem.cosem import Obis

obis = Obis.from_bytes(obis_bytes)

# The datetime can be parsed like:
from dlms_cosem.time import datetime_from_bytes

datetime, clock_status = datetime_from_bytes(time_bytes)

# Ex:
datetime_from_bytes(b'\x07\xe3\x0c\x10\x01\x07;(\xff\x80\x00\xff')
>>>(datetime.datetime(2019, 12, 16, 7, 59, 40), ClockStatus(invalid=True, doubtful=True, different_base=True, invalid_status=True, daylight_saving_active=True))

# The unit enums are not added in the DLMS library yet. (on roadmap)

```
## Timing

Det verkar vara lite olika med vilken data som skickas och hur ofta.
