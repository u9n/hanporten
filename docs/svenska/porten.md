# Svenska porten

HAN-porten, eller P1-porten, sitter oftast under ett litet lock på din elmätare. Den 
består av ett RJ12 uttag. Den svenska porten är baserad på
[DSMR P1 Companion Standard](https://www.netbeheernederland.nl/_upload/Files/Slimme_meter_15_a727fce1f1.pdf)

## Spänningsmatning av utrustning

Det är möjligt att spänningsmata din utrustning direkt från HAN-porten. Den levererar 
+5 V på en pinne som kan användas för att driva extern utrustning. Du kan dra max 
250 mA från porten.

!!! note "Hör med ditt nätbolag"
    Enligt rekommendationerna är det inte ett krav att leverera en HAN-port med 
    spänningsmatning från mätarna. Så det är möjligt att den mätare ditt elbolag köpt 
    in inte kan spänningsmata utrustningen. Men de största bolagen levererar port med 
    spänningsmatning. 

## PIN-layout

![HAN-port med PIN layout](../images/han-port-with-numbers.svg)

Pin | Namn | Beskrivning | Kommentar
--- | --- | --- | ---
1 | VCC | +5V spänningsmatning | Kan användas för att driva utrustning
2 | DATA REQUEST | +5V på denna startar export av data | Om man alltid vill att utrustningen skall skicka data så bygla över +5V till denna
3 | DATA GND | Jord data | 
4 | NC | Inte ansluten | Används inte 
5 | DATA OUT | Data output | Öppen kollektor
6 | GND | Jord spänningsmatning | 

## Koppla in extra utrustning

Om du behöver mer än en enhet kan du använda en hub som delar datasignalen till flera 
HAN-portar. Men detta kan kräva att man måste spänningsmata utrustningen separat då det 
inte är säkert att HAN-porten kan driva flera enheter.







