# Apartado de configuracion de elasticsearch
## Explicacion inicial

En esta carpeta Elasticsearch puedes encontrar los ficheros que han sido modificados a mano, comentados para mejor comprension.
Los ficheros editados son:

carpeta: elasticsearch
    - carpeta: conf
        - jvm.options
        - elasticsearch.yml

## Numero de nodos

Este proyecto presta de 3 nodos bajo el cluster Blind-Eternities:

    - Emrakul: 
        - ip: 192.199.1.55
        - roles: master, data, ingest
    
    - Kozilek: 
        - ip: 192.199.1.56
        - roles: master, data, ingest

    - Ulamog: 
        - ip: 192.199.1.57
        - roles: master, data, ingest

