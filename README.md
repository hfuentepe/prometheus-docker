#Docketizar un Stack para monitorización basado en Prometheus y Grafana


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
  - [Configuración Alert manager](#configurar-alertmanager)

## Visión General

El objetivo principal es crear un stack para monitorización basado en [Prometheus](http://prometheus.io/) y [Grafana] (http://grafana.com) docketizado. El stack completo estará compuesto por los siguientes componentes (Marcado con X aquellos que ya se encuentran incluidos en el docker-compose):

- [X] **Prometheus:**
- [X] **Grafana:** 
- [X] **Alert Manager:** The [Alertmanager] ([https://prometheus.io/docs/alerting/alertmanager/](https://prometheus.io/docs/alerting/alertmanager/)) handles alerts sent by client applications such as the Prometheus server. It takes care of deduplicating, grouping, and routing them to the correct receiver integration such as email, PagerDuty, or OpsGenie. It also takes care of silencing and inhibition of alerts.
- [X] **Node Exporter:** The Prometheus [Node Exporter](https://prometheus.io/docs/guides/node-exporter/#monitoring-linux-host-metrics-with-the-node-exporter) exposes a wide variety of hardware- and kernel-related metrics.
- [ ] **cAdvisor (Container Advisor):** [cAdvisor] ([https://github.com/google/cadvisor](https://github.com/google/cadvisor)) provides container users an understanding of the resource usage and performance characteristics of their running containers.

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

Para realizar personalizaciones sobre la configuración de prometheus debe ir a la carpeta */prometheus-docker/prometheus* donde se encuentra el fichero de configuración *prometheus.yml*.

En este fichero podrá configurar el intervalo de scrape, alertas, alert manager y las fuentes donde vamos a obtener la información en la sección scrape_configs. Para más información puede consultar [Prometheus Configuration] (https://prometheus.io/docs/prometheus/latest/configuration/configuration/)

## Configuración Grafana

En primer lugar debe dirigirse a la carpeta */prometheus-docker/grafana* donde se encuentran los ficheros de configuración.

### Modificar contraseña Administrador

En el fichero de configuración config.monitoring hemos configurado un par de propiedades:
- GF_SECURITY_ADMIN_PASSWORD=**foobar**, cambiar la password de administrador
- GF_USERS_ALLOW_SIGN_UP=false

### Configurar Datasources

El fichero que contiene los datasources de Grafana se encuentra en *prometheus-docker/grafana/provisioning/datasources/datasources.yml*. Este fichero contiene un único datasource de tipo Prometheus y con nombre Prometheus. Para más información visite [Grafana Provisioning] (https://grafana.com/docs/grafana/latest/administration/provisioning/)

### Configurar Dashboards

El proyecto contiene el fichero **dashboard.yml** en la carpeta *prometheus-docker/grafana/provisioning/dashboards*. Este fichero permite posible administrar dashboards en Grafana (pueden existir varios). Cada archivo de configuración puede contener una lista de proveedores de paneles que cargarán paneles en Grafana desde el sistema de archivos local.

En nuestro caso solo hemos añadido un proveedor de dashboard con el nombre Prometheus. Además en la carpeta hay varios dashboards en formato json que se cargan de forma automática. Puede [crear sus propios] (https://grafana.com/docs/grafana/latest/features/dashboard/dashboards/) Dashboards o descargarlos de la [página oficial de grafana] (https://grafana.com/grafana/dashboards) .

En nuestro caso hemos descargado varios dashboards de la página oficial y en algunos de ellos hemos realizado una modificación para que se carguen correctamente y no tengamos errores porque no encuentra variables. Los dashboards modificados son jvm-micrometer_rev9.json y spring-boot-statistic_rev2.json. En ambos casos los dashboards tiene una **variable ${DS_PROMETHEUS}** para establecer el Datasource cuando se carga desde la UI de Grafana. Hemos **reemplazado** el valor ${DS_PROMETHEUS} por el nombre de nuestro Datasource: **Prometheus**

## Configurar Node Exporter

**Prometheus Node Exporter** nos permite monitorizar métricas de Linux Host de forma sencilla y las expone mediante un endPoint para Prometheus. En este caso, no tenemos una carpeta con una configuración específica, no obstante hay que tener en cuenta un par de detalles:
- En la definición del contenedor en el fichero docker-compose hemos modificado el valor de la variable *collector.filesystem.ignore-mount-points* por defecto excluye pseudo filesystem, de solo lectura y otros filesystem sin interés. En nuestro caso, hemos excluido ciertos directorios de Docker.

- En cuanto al fichero de configuración de Prometheus, hemos añadido un nuevo job_name para node-exporter. En esta ocasión vamos utilizar el descubrimiento basado en DNS [dns_sd_configs](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#dns_sd_config) para que Prometheus acceda al endPoint donde obtener la información. Es importante poner en names el nombre del servicio definido en el Docker-compose, ya que Docker-compose hace de DNS resolviendo la IP del contenedor.
   
      - job_name: 'node-exporter'     
            scrape_interval: 5s
            dns_sd_configs:
            - names:
              - 'node-exporter'
              type: 'A'
              port: 9100



## Configurar AlertManager

La configuración se encuentra en la carpeta *prometheus-docker/alertmanager/* en el fichero *config.yml*. En este fichero vamos a configurar los receptores de las alertas. Alertmanager puede enviar las notificaciones de las alertas a diferentes receptores como mail, **Hipchart**, **Pagerduty**, **Slack**, ... En nuestro caso hemos configurado un nodo y un receptor de tipo mail. No obstante, también podéis encontrar un ejemplo con Slack comentado.
route:
  receiver: 'notificaciones'

    receivers:
    - name: 'notificaciones'
      email_configs:
      - to: '<gmail_account>'
        from: '<gmail_account>'
        smarthost: smtp.gmail.com:587
        auth_username: '<gmail_account>'
        auth_identity: '<gmail_account>'
        auth_password: '<password>'
        send_resolved: true
    #- name: 'notificaciones'
    #  slack_configs:
    #  - send_resolved: true
    #    username: '<username>'
    #    channel: '#<channel-name>'
    #    api_url: '<incomming-webhook-url>'

Por otro lado, en Prometheus tenemos que configurar las alertas y la configuración para que Prometheus se comunique con Alertmanager. Los ficheros de alertas se encuentran en la carpeta *prometheus-docker/prometheus/*, en nuestro caso solo hemos creado una unica regla *alert.rules* que se lanzara cuando no haya ningún nodo de nodeExporter activo durante  3 minutos:

    groups:
    - name: Hardware alerts
      rules:
      - alert: Node down
        expr: up{job="node-exporter"} == 0
        for: 3m
        labels:
          severity: warning
        annotations:
          title: Node {{ $labels.instance }} is down
          description: Failed to scrape {{ $labels.job }} on {{ $labels.instance }} for more than 3 minutes. Node seems down.

Por ultimo, vamos a indicar a Prometheus que reglas tiene que cargar y donde se encuentra el services manager:

    # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
    rule_files:
      - 'alert.rules'

    # Alertmanager configuration
    alerting:
      alertmanagers:
      - scheme: http
        static_configs:
        - targets:
          - "alertmanager:9093"
