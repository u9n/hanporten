# Norska porten

HAN-porten, eller P1-porten, sitter oftast under ett litet lock på din elmätare. 

Den norska varianten består av ett RJ45 uttag, samma som på vanliga internetsladdar.

## Spänningsmatning av utrustning

Eftersom det elektriska interfacet är M-Bus (Meter-Bus)är det möjligt att 
spänningsmata din utrustning genom porten. Men det krävs en ordentligt 
Mbus slave implementation på hårdvarusidan. 

## PIN-layout

![Norsk HAN-port med PIN layout](../images/rj45.svg)

Pin | Namn | Beskrivning | Kommentar
--- | --- | --- | ---
1 | MBus + | MBus trådpar | 
2 | Mbus - | MBus trådpar startar export av data | 


## Koppla in extra utrustning

MBus i mätaren agerar master och det är möjligt att ha flera slavar på slingan som 
lyssnar pushad data. 


## Externa referenser

- https://www.nek.no/wp-content/uploads/2018/10/Kamstrup-HAN-NVE-interface-description_rev_3_1.pdf
- https://www.nek.no/wp-content/uploads/2017/10/AMS-HAN-personvernnotat-h%C3%B8ringsversjon.pdf
- https://www.energiforetagen.se/globalassets/energiforetagen/det-erbjuder-vi/publikationer/branschrekommendation-lokalt-granssnitt-v1-2-2018.pdf
- https://www.utomhusliv.se/wp-content/uploads/2020/10/Specifikation-f%C3%B6r-HAN-modulen-f%C3%B6r-elm%C3%A4tare-engelska.pdf
- https://www.skekraft.se/wp-content/uploads/2021/03/Aidon_Feature_description_RJ45_HAN_Interface_EN.pdf
- https://www.tekniskaverken.se/siteassets/tekniska-verken/elnat/aidonfd-rj45-han-interface-se-v13a.pdf
- https://www.kode24.no/guider/smart-meter-part-1-getting-the-meter-data/71287300
- https://xipher.dk/posts/2020-05-17-using-esp8266-to-monitor-kamstrup-omnipower/
- https://github.com/Claustn/esp8266-kamstrup-mqtt

