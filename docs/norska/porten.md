# Norska porten

HAN-porten, eller P1-porten, sitter oftast under ett litet lock på din elmätare. 

Den norska varianten består av ett RJ45 uttag, samma som på vanliga internetsladdar.

## Spänningsmatning av utrustning

Eftersom det elektriska interfacet är MBus är det möjligt att spänningsmata din 
utrustning genom porten. Men det krävs en ordentligt Mbus slave implementation på 
hårdvarusidan. 

## PIN-layout

![Norsk HAN-port med PIN layout](../images/rj45.svg)

Pin | Namn | Beskrivning | Kommentar
--- | --- | --- | ---
1 | MBus + | MBus trådpar | 
2 | Mbus - | MBus trådpar startar export av data | 


## Koppla in extra utrustning

MBus i mätaren agerar master och det är möjligt att ha flera slavar på slingan som 
lyssnar pushad data. 







