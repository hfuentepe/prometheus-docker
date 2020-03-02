# Docketizar un Stack para monitorización basado en Prometheus y Grafana

Índice
  - [Visión General](#visión-general)
  - [Pre-requisitos](#pre-requisitos)
  - [Instalación](#instalación)
  - [Configuración Prometheus](#configuración-prometheus)
  - [Configuración Grafana](##configuración-grafana)
    - [Contraseña Administrador](#modificar-contraseña-administrador)
    - [Configuración de Datasources](#configurar-datasources)
    - [Configuración de Dashboards](#configurar-dashboards)
  - [Configuración Node Exporter](#configurar-node-exporter)

## Visión General

El objetivo principal es crear un stack para monitorización basado en [Prometheus](http://prometheus.io/) y [Grafana] (http://grafana.com) docketizado. El stack completo estará compuesto por los siguientes componentes (Marcado con X aquellos que ya se encuentran incluidos en el docker-compose):

- [X] Prometheus
- [X] Grafana
- [ ] Alert Manager
- [X] Node Exporter: The Prometheus [Node Exporter](https://prometheus.io/docs/guides/node-exporter/#monitoring-linux-host-metrics-with-the-node-exporter) exposes a wide variety of hardware- and kernel-related metrics.
- [ ] cAdvisor (Container Advisor)

## Pre-Requisitos

Debe tener instalada la última versión de [Docker] (https://www.docker.com/) y [Docker-Compose] (https://docs.docker.com/compose/install/) .

## Instalación

En primer lugar debe clonar el proyecto en su máquina.

En caso de realizar configuraciones sobre los diferentes componentes vaya a la sección de configuración correspondiente de cada uno de ellos:
  - [Prometheus](#configurar-prometheus)
  - [Grafana](#configurar-grafana)

Una vez haya realizado las configuraciones, se debe posicionar en la carpeta prometheus-docker y ejecute el siguiente comando:

    $ docker-compose -f docker-compose.yml up -d

Para verificar el log de la creación del stack puede ver los logs:

    $ docker-compose logs

## Configuración Prometheus

Para realizar personalizaciones sobre la configuración de prometheus debe ir a la carpeta /prometheus-docker/prometheus donde se encuentra el fichero de configuración prometheus.yml. 

En este fichero podrá configurar el intervalo de scrape, alertas, alert manager y las fuentes donde vamos a obtener la información en la sección scrape_configs. Para más información puede consultar [Prometheus Configuration] (https://prometheus.io/docs/prometheus/latest/configuration/configuration/)

## Configuración Grafana

En primer lugar debe dirigirse a la carpeta */prometheus-docker/grafana* donde se encuentran los ficheros de configuración.

### Modificar contraseña Administrador

En el fichero de configuración config.monitoring hemos configurado un par de propiedades:
- GF_SECURITY_ADMIN_PASSWORD=**foobar**, cambiar la password de administrador
- GF_USERS_ALLOW_SIGN_UP=false

### Configurar Datasources

El fichero que contiene los datasources de Grafana se encuentra en *prometheus/grafana/provisioning/datasources/datasources.yml*. Este fichero contiene un único datasource de tipo Prometheus y con nombre Prometheus. Para más información visite [Grafana Provisioning] (https://grafana.com/docs/grafana/latest/administration/provisioning/)

### Configurar Dashboards

El proyecto contiene el fichero **dashboard.yml** en la carpeta *prometheus/grafana/provisioning/dashboards*. Este fichero permite posible administrar dashboards en Grafana (pueden existir varios). Cada archivo de configuración puede contener una lista de proveedores de paneles que cargarán paneles en Grafana desde el sistema de archivos local.

En nuestro caso solo hemos añadido un proveedor de dashboard con el nombre Prometheus. Además en la carpeta hay varios dashboards en formato json que se cargan de forma automática. Puede [crear sus propios] (https://grafana.com/docs/grafana/latest/features/dashboard/dashboards/) Dashboards o descargarlos de la [página oficial de grafana] (https://grafana.com/grafana/dashboards) .

En nuestro caso hemos descargado varios dashboards de la página oficial y en algunos de ellos hemos realizado una modificación para que se carguen correctamente y no tengamos errores porque no encuentra variables. Los dashboards modificados son jvm-micrometer_rev9.json y spring-boot-statistic_rev2.json. En ambos casos los dashboards tiene una **variable ${DS_PROMETHEUS}** para establecer el Datasource cuando se carga desde la UI de Grafana. Hemos **reemplazado** el valor ${DS_PROMETHEUS} por el nombre de nuestro Datasource: **Prometheus**

## Configurar Node Exporter

Prometheus Node Exporter nos permite monitorizar métricas de Linux Host de forma sencilla y las expone mediante un endPoint para Prometheus. En este caso, no tenemos una carpeta con una configuración específica, no obstante hay que tener en cuenta un par de detalles.

- En la definición del contenedor en el fichero docker-compose hemos modificado el valor de la variable collector.filesystem.ignore-mount-points por defecto excluye pseudo filesystem, de solo lectura y otros filesystem sin interés. En nuestro caso, hemos excluido ciertos directorios de Docker.


- En cuanto al fichero de configuración de Prometheus, hemos añadido un nuevo job_name para node-exporter. En esta ocasión vamos utilizar el descubrimiento basado en DNS [dns_sd_configs](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#dns_sd_config) para que Prometheus acceda al endPoint donde obtener la información. Es importante poner en names el nombre del servicio definido en el Docker-compose, ya que Docker-compose hace de DNS resolviendo la IP del contenedor.
