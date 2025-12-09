# Instalacion
Para empezar, vamos a necesitar descargarnos las imagenes de elastic, kibana, logstash y metricbeat.
En todas las maquinas se ha descargado elastic y metricbeat.
En este proyecto se ha descargado en la maquina Emrakul kibana, y logstash en Kozilek, en Ulamog esta docker con streamlit, el scrapper, la api y el modelo para el chatbot.
este proyecto se ha realizado en ubuntu server (Linux).

## descarga
Puedes leer como instalar elastic y los demas componentes en la [guia oficial](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-from-archive-on-linux-macos)

## configuracion
Una vez tengamos descargado todo, vamos a editar los ficheros de configuracion de cada componente, para ello nos podemos basar en la configuracion que ya viene en las carpetas de este proyecto.

para elastic los ficheros estan en la carpeta elastic-x.x.x/config/
editar los ficheros que compartan nombre con los de este proyecto y solo añadir las lineas no aouto-generadas.

para kibana lo mismo, en la carpeta kibana-x.x.x/config/

en logstash es logstash-x.x.x/config
aqui tenemos el pipelines y el .yml, para añadir nuestras pipelines en el proyecto hemos creado una carpeta llamada pipeline_cartas y ahi hemos separado el input, filter y output.

en metricbeat la ruta es metricbeat-x.x.x/
aqui esta el .yml 
en metricbeat-x.x.x/modules.d/ estan todos los modulos, en este proyecto usamos 3, los xpack de elastic, kibana y logstash.
para activarlos el comando siguiente se tiene que lanzar desde la carpeta de metricbeat: ./meticbeat modules activate elasticsearch-xpack
lo mismo con los otros dos, la configuracion esta en el proyecto.

Todos los ficheros de jvm.options que esten bien configurados, estas herramientas acaparan muchos recursos a menos que los limites, no es raro que una maquina colapse por tener elastic, kibana y logstah a la vez.

## Inicio
Empezamos levantando el primer nodo de elastic (bin/elasticsearch en la carpeta de elastic), cuando termine nos va a soltar un chorro de texto en el que va a salir un campo llamado kibana token, copialo, en otra ventana sacamos el enrollment token (bin/elasticsearch-create-enrollment-token -s node en la misma carpeta) e iniciamos el resto de nodos con el comando bin/elasticsearch --enrollment-token <tu-token>

antes de nada vamos a reiniciar la contraseña de elastic, para introducir una que nosotros queramos (bin/elasticsearch-reset-password -u usuario --interactive)

vamos a levantar kibana, para ello nos situamos en la carpeta de kibana y ejecutamos el comando bin/kibana-setup --enrollment-token <kibana-token>
cuando termine de configurarse, podras viajar a la ip+puerto que has configurado en kibana, si tienes todo bien, te pedira que te logees, el usuario es elastic y la contraseña la que has introducido al reiniciarla.

para subir datos, si tienes una fuente ya disponible y has configurado las pipelines de logstash, puedes ir a la carpeta y ejecutar bin/logstash.

para activar metricbeat consultar [esta guia](https://drive.google.com/file/d/1mwqq1tQQwdbNQhnyduQDQmkIN9_Ugwby/view)

Apartir de aqui, es implementar index_templates, lifecycle policies si los datos son historicos y quieres tener un control sobre la cantidad de datos, etc...