---
description: >-
  En esta práctica se usará un conjunto de las siguientes aplicaciones para
  crear un dashboard con MQTT, Telegraf, InfluxDB y Grafana.
---

# Práctica 18 - IoTT Dashboard

**Todo el contenido utilizado para esta práctica ha salido del siguiente repositorio de GitHub:** 

**https://github.com/josejuansanchez/co2-celia**

Datos de acceso:

* Grafana: [http://3.238.26.95:3000/](http://3.238.26.95:3000/)
* Usuario: admin
* Clave: admin


Se creará una máquina en Amazon Web Services cuyo propósito será almacenar gráficas con los datos atmosféricos \(en concreto los niveles de CO2\) del aula número 22 del IES Celia Viñas.

Se implementarán los cuatro servicios utilizando Docker Compose. El archivo utilizado para ello contiene lo siguiente:

```text
version: '3'

services: 
  mosquitto:
    image: eclipse-mosquitto:2
    ports:
      - 1883:1883
    volumes:
      - ./mosquitto/mosquitto.conf:/mosquitto/config/mosquitto.conf
      - mosquitto_data:/mosquitto/data
      - mosquitto_log:/mosquitto/log

  telegraf:
    image: telegraf
    volumes:
      - ./telegraf/telegraf.conf:/etc/telegraf/telegraf.conf
    depends_on: 
      - influxdb

  influxdb:
    image: influxdb
    ports:
      - 8086:8086
    volumes:
      - influxdb_data:/var/lib/influxdb
    environment:
      - INFLUXDB_DB=${INFLUXDB_DB}
      - INFLUXDB_ADMIN_USER=${INFLUXDB_USERNAME}
      - INFLUXDB_ADMIN_PASSWORD=${INFLUXDB_PASSWORD}
      - INFLUXDB_HTTP_AUTH_ENABLED=true

  grafana:
    image: grafana/grafana:7.4.0
    ports:
      - 3000:3000
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana-provisioning/:/etc/grafana/provisioning
    depends_on:
      - influxdb

  chronograf:
    image: chronograf
    ports:
      - 8888:8888
    volumes:
      - chronograf_data:/var/lib/chronograf
    depends_on:
      - influxdb
    environment:
      - INFLUXDB_URL=http://influxdb:8086
      - INFLUXDB_USERNAME=${INFLUXDB_USERNAME}
      - INFLUXDB_PASSWORD=${INFLUXDB_PASSWORD}

volumes:
  mosquitto_data:
  mosquitto_log:
  node_red_user_data:
  influxdb_data:
  grafana_data:
  chronograf_data:
```

Se utilizará Mosquitto como servicio MQTT, InfluxDB como base de datos, Grafana como interfaz web para implementar los gráficos necesarios para la práctica, chronograf que comunicará los datos en InfluxDB con los de Grafana y por último telegraf como software que se utilizará de software para el sensor.

Los datos de entorno para el fichero de Docker Compose serán los siguientes:

```text
INFLUXDB_DB=iescelia_db
INFLUXDB_USERNAME=root
INFLUXDB_PASSWORD=root
```

Simplemente se indicará el nombre de la base de datos en InfluxDB que será "iescelia\_db" junto con sus credenciales en caso de necesitar conectarse a la misma.

Se ejecutará el archivo compose con el comando de siempre:

```text
$ sudo docker-compose up -d
```

Y se accederá a la ip pública de la máquina junto con el puerto de Grafana \(3000\):

![](https://i.ibb.co/MVt17gF/image-47.png)

Para acceder al panel de control se utilizarán los siguientes datos de acceso:

* USER: Admin
* CLAVE: Admin

Al iniciar sesión por primera vez, Grafana pedirá cambiar la contraseña por defecto, en mi caso seguirá igual. En la vista principal, se pulsará en el icono de los cuatro cuadrados \(Dashboards\) y en "Manage":

![](https://i.ibb.co/frFccC4/image-45.png)

Este será el dashboard creado junto con el contenedor de Docker, sus datos salen de un archivo ".json" incluido en el repositorio y referenciado en el archivo Compose para montarlo en la carpeta correspondiente de Grafana. Se seleccionará y nos mostrará dos gráficos que aparecen vacíos.

En este momento se deberían recoger los datos con un sensor, pero como en mi caso no dispongo del mismo, se utilizará Mosquitto para enviar datos inventados a Grafana y testear el funcionamiento. Se necesitarán dos comandos para hacer esto:

```text
$ sudo docker run --init -it --rm efrecon/mqtt-client sub -h IP -t "iescelia/#"
$ sudo docker run --init -it --rm efrecon/mqtt-client pub -h IP -p 1883 -t "iescelia/aula22/co2" -m DATO
```

El primer comando tiene un parámetro que deberá ser cambiado antes de ejecutarse \(-h\) sustituyendo la palabra "IP" por la IP pública o nombre DNS del servidor, una vez ejecutado se quedará a la escucha y ese terminal deberá dejarse abierto.

Con el segundo comando se hará exactamente lo mismo, y después del parámetro "-m" se introducirá un dato. Si el terminal de escucha sigue abierto, el dato se grabará dentro de Grafana, en concreto dentro de la gráfica bajo el nombre "Aula 22 - CO2". Si se desea introducir valores en la segunda gráfica, se cambiará la línea "iescelia/aula22/co2" a "iescelia/aula22/tvoc".

Debería quedar algo así tras introducir varios datos:

![](https://i.ibb.co/nz8Jt8n/image-46.png)
