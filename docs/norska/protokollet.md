# Protokollet

Protokollet som skickas på den norska HAN-porten är formaterat enligt DLMS/COSEM men kan se olika ut beroende på elmätarens tillverkare. För mer information om en specifik typ av mätare rekommenderas det att gå igenom manualen för mätaren. Nedan listas manualerna från några vanliga tillverkare.

 * [Aidon](https://aidon.com/wp-content/uploads/2023/07/AIDONFD_RJ45_HAN_Interface_SV.pdf?_gl=1*1pnav0t*_up*MQ..*_ga*NDE3NDMwMjcwLjE2OTQ2MDkxMzY.*_ga_7C4RC0STZL*MTY5NDYwOTEzNS4xLjAuMTY5NDYwOTEzNS4wLjAuMA..)
 * [Kamstrup](https://documentation.kamstrup.com/docs/HAN_NVE/en-GB/Information_sheet/CONTAAA62C8B611D4C4EB3186FD81AEE0152/)
 * [Kaifa](https://piers.rocks/static/files/S1001_Kaifa.HAN.OBIS.codes.KFM_001.pdf)

!!! note "Ej verifierat"
    Vi har ingen mätare med denna typ av port och har inte verifierat om allt är helt korrekt.
    Men om någon har en mätare med denna port får de gärna verifiera och meddela oss.

## Datainnehåll

Datan är ett antal rader som liknar datan i den svenska standarden, men formaterad 
mer enligt DLMS. Exakt hur datan ser ut beror på elmätarens tillverkare.

Elmätare från Aidon skickar data som innehåller en rad för tid och datum, en rad för identifiering av mätaren
och flera rader som innehåller OBIS, värde, skalär och enhet.

Elmätare från Kamstrup skickar data som innehåller en rad för identifiering av mätaren och sedan fortsätter
listan med en OBIS-kod på varannan rad och tillhörande värde på nästa rad. Skalärer och enheter för varje 
OBIS-kod förväntas den som läser datat känna till sedan innan. Det är därför viktigt att hänvisa till manualen 
där detta finns dokumenterat.

Elmätare fråm Kaifa skickar bara en fördefinierad lista med värden och räknar med att den som läser ska veta 
vad värdena betyder och hur de ska tolkas. Man bör därför läsa manualen för att se vad värdena i listan betyder.

## Seriellt gränssnitt

Data skickas med 2400 baud och i byteformat `8N1` för de flesta elmätare (Aidon använder formatet `8E1`)

## Exempeldata

Följande exempel visar hur data från en Aidon mätare kan komma att se ut:

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
from dlms_cosem.protocol import xdlms
from dlms_cosem.hdlc import frames
from dlms_cosem.utils import parse_as_dlms_data
from dlms_cosem.cosem import Obis
from dlms_cosem.time import datetime_from_bytes

# 3-phase
hdlc_data_hex = (
    "7ea2434108831385ebe6e7000f4000000000011b020209060000010000ff090c07e30c1001073b28ff"
    "8000ff020309060100010700ff060000046202020f00161b020309060100020700ff06000000000202"
    "0f00161b020309060100030700ff06000005e302020f00161d020309060100040700ff060000000002"
    "020f00161d0203090601001f0700ff10000002020fff1621020309060100330700ff10004b02020fff"
    "1621020309060100470700ff10000002020fff1621020309060100200700ff12090302020fff162302"
    "0309060100340700ff1209c302020fff1623020309060100480700ff12090402020fff162302030906"
    "0100150700ff060000000002020f00161b020309060100160700ff060000000002020f00161b020309"
    "060100170700ff060000000002020f00161d020309060100180700ff060000000002020f00161d0203"
    "09060100290700ff060000046202020f00161b0203090601002a0700ff060000000002020f00161b02"
    "03090601002b0700ff06000005e202020f00161d0203090601002c0700ff060000000002020f00161d"
    "0203090601003d0700ff060000000002020f00161b0203090601003e0700ff060000000002020f0016"
    "1b0203090601003f0700ff060000000002020f00161d020309060100400700ff060000000002020f00"
    "161d020309060100010800ff060099598602020f00161e020309060100020800ff060000000802020f"
    "00161e020309060100030800ff060064ed4b02020f001620020309060100040800ff06000000050202"
    "0f001620be407e"
)

ui = frames.UnnumberedInformationFrame.from_bytes(bytes.fromhex(hdlc_data_hex))
dn = xdlms.DataNotification.from_bytes(
    ui.payload[3:]
)  # The first 3 bytes should be ignored.
result = parse_as_dlms_data(dn.body)

# First is date
date_row = result.pop(0)
clock_obis = Obis.from_bytes(date_row[0])
clock, stats = datetime_from_bytes(date_row[1])
print(f"Clock object: {clock_obis.to_string()}, datetime={clock}")

# rest is data
for item in result:
    obis = Obis.from_bytes(item[0])
    value = item[1]
    print(f"{obis.to_string()}={value}")

```
## Timing

Det är lite olika med vilken data som skickas och hur ofta. Vanligtvis har mätarna två eller 
tre olika listor som skickas med olika intervaller. Exempelvis kan en kort lista skickas 
var tionde sekund och en längre lista med ytterligare värden skickas varje timme. Detta finns
i så fall dokumenterat i tillverkarens manual.
