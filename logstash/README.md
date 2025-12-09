# LOGSTASH

## Ficheros y contenido

### jvm.options
Este fichero es para limitar cuantos recursos consume logstash.

### logstash.yml
Este fichero es para configurar logstash, en general no hace ni falta tocarlo.

### pipelines.yml
Este fichero es para configurar las pipelines con las que vas a ir trabajando.

## pipeline_cartas

### 01_input_cartas.conf
Aqui configuramos de donde va a obtener los datos, en este caso tenemos preparada una api.

input {
    http_poller {
        urls => {
            cartas_magic => {
            method => get
            url => "http://192.199.1.57:8000/cards-data" # direccion de la api y el puerto mas el metodo GET
             headers => { Accept => "application/json" } # llega en formato json
            }
        }
        request_timeout => 180        # espera 180 segundos hasta que, si no recibe nada se cierra
        schedule => { every => "2d" } # se ejecuta cada 2 dias
        codec => json                 # se formatea a json
    }

### 10_filter_cartas.conf
Aqui se configura el filtro y se transforman los datos, tambien se recojen fallos (con verificación de grok)

filter {
  # Solo procesa si existe el campo [message] y no es NilClass
  if [message] and [message] !~ /NilClass/ {
    
    # Divide el array en objetos individuales
    split { field => "[message]" }

    # Renombra el objeto resultante a "carta"
    mutate {
      rename => { "[message]" => "carta" }
    }

    # copia los campos dentro de carta al mismo nivel
    mutate {
      copy => { "[carta][oracle_id]"          => "oracle_id" }
      copy => { "[carta][parent_id]"          => "parent_id" }
      copy => { "[carta][face_number]"        => "face_number" }
      copy => { "[carta][name]"               => "name" }
      copy => { "[carta][lang]"               => "lang" }
      copy => { "[carta][released_at]"        => "released_at" }
      copy => { "[carta][image_png]"          => "image_png" }
      copy => { "[carta][mana_cost]"          => "mana_cost" }
      copy => { "[carta][cmc]"                => "cmc" }
      copy => { "[carta][type_line]"          => "type_line" }
      copy => { "[carta][oracle_text]"        => "oracle_text" }
      copy => { "[carta][power]"              => "power" }
      copy => { "[carta][toughness]"          => "toughness" }
      copy => { "[carta][commander_legality]" => "commander_legality" }
      copy => { "[carta][game_changer]"       => "game_changer" }
      copy => { "[carta][set_name]"           => "set_name" }
      copy => { "[carta][rarity]"             => "rarity" }
      copy => { "[carta][artist]"             => "artist" }
      copy => { "[carta][full_art]"           => "full_art" }
      copy => { "[carta][booster]"            => "booster" }
      copy => { "[carta][ranking]"            => "ranking" }
      copy => { "[carta][price_usd]"          => "price_usd" }
      copy => { "[carta][price_usd_foil]"     => "price_usd_foil" }
      copy => { "[carta][price_usd_etched]"   => "price_usd_etched" }
      copy => { "[carta][cardmarket_url]"     => "cardmarket_url" }
      copy => { "[carta][embedding]"          => "embedding" }
    }

    # Elimina el objeto anidado para que no quede duplicado
    mutate {
      remove_field => ["carta"]
    }

    # Elimina los campos especiales de Logstash
    mutate {
      remove_field => ["@timestamp", "@version"]
    }
  }
}

### 20_output_cartas.conf
manda los datos a los nodos levantados mirando en orden cual esta disponible usando una api key de solo escritura para el indice cartas_magic

output {
  # Output normal hacia Elasticsearch
  elasticsearch {
    hosts => [
      "https://192.199.1.55:9200",
      "https://192.199.1.56:9200",
      "https://192.199.1.57:9200"
    ]
    index => "cartas_magic"
    document_id => "%{oracle_id}" # usamos el campo unico oracle_id como ID para indexar las cartas
    action => "update"
    doc_as_upsert => true         # si existe hace update, si no existe hace insert
    api_key => "GC3kBJsB8sZQN9cjis7T:dgvX-5JJE_vTSPK58-ZwoA"

    #user => "elastic"
    #password => "Nicolbolas2026+"

    ssl_certificate_authorities => ["/home/g1/elasticsearch-9.2.1/config/certs/http_ca.crt"]
    ssl_verification_mode => "full"  # asegura que se valide el certificado
  }

  # Output adicional para errores detectados
  if "grok_error_obtencion_cartas" in [tags] {
    file {
      path => "/home/g1/logstash-9.2.1/conf/pipeline_cartas/grok_error_obtencion_cartas-%{+YYYY-MM-dd}.log"
      codec => line { format => "Error: %{message}" }
    }
  }

  # Para depuración
  stdout { codec => rubydebug }
}
