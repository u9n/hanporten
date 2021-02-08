# Om HAN-porten

HAN-porten, eller P1-porten, sitter oftast under ett litet lock på din elmätare. Den 
består av ett RJ12 uttag, samma som på fasta telefoner.

!!! warning
    Om din mätare istället har ett RJ45 uttag (internet-sladd) så har din mätare en 
    HAN-port av den norska standarden. Koppla inte in någon utrustning i för den 
    svenska standarden då det riskerar att ha sönder utrustningen.

## Aktivera din port

Standarden i Sverige är att HAN-porten är inaktiverad på standard. För att aktivera den
kontaktar du ditt elnätsbolag som skall aktivera den utan någon extra kostnad för dig.

## Spänningsmatning av utrustning

Det är möjligt att spänningsmata din utrustning direkt från HAN-porten. Den levererar 
+5 V på en pinne som kan användas för att driva externa utrustning. Du kan som max dra 
100 mA från porten.

## PIN-layout

![HAN-port med PIN layout](images/han-port-with-numbers.svg)

Pin | Namn | Beskrivning | Kommentar
--- | --- | --- | ---
1 | +5V | +5V spänningsmatning | Kan användas för att driva utrustning
2 | Data Request | +5V på denna startar export av data | Om man alltid vill att utrustningen skall skicka data så bygla över +5V till denna
3 | Data Ground | Jord data | 
4 | NC | Inte ansluten | Används inte 
5 | Data | Data line | Output, öppen kollektor
6 | Power GND | Jord spänningsmätning | 

## Koppla in extra utrustning

Om du behöver mer än en enhet kan du använda en hub som delar datasignalen till flera 
HAN-portar. Men detta kan kräva att man måste spänningsmata utrustningen separat då det 
inte är säkert att HAN-porten kan driva flera enheter.







