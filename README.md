
ndice
  - [Visión General](##visión-general)
  - [Pre-requisitos](#pre-requisitos)
  - [Instalación](#installation)
  - [Configuracion Prometheus](#configuracion-prometheus)
  - [Configuración Grafana](#configuracion-grafana)
    - [Añadir Datasources](#add-datasources)
    - [Añadir Dashboards](#add-dashboards)

## Visión General

El objetivo principal es crear un stack para monitorización basado en [Prometheus](http://prometheus.io/) y [Grafana] (http://grafana.com) docketizado. El stack completo estara compuesto por los siguientes componentes (Marcado con X aquellos que ya se encuentran incluidos en el docker-compose):

- [X] Prometheus
- [X] Grafana
- [ ] Alert Manager
- [ ] Node Exporter
- [ ] cAdvisor (Container Advisor)

## Pre-Requisitos

Debe tener instalada la ultima versión de [Docker] (https://www.docker.com/) y [Docker-Compose] (https://docs.docker.com/compose/install/) .

## Instalación

En primer lugar debe cloner el proyecto en su maquina.

En caso de realizar configuracion sobre los diferentes componentes vaya a la sección de configuración correspondiente de cada uno de ellos:
  - [Prometheus](#configuracion-prometheus)
  - [Grafana](#configuracion-grafana)
  
Una vez haya realizado las configuraciones, se debe posicionar en la carpeta prometheus-docker y ejecute el siguiente comando:
  
    $ docker-compose -f docker-compose.yml up -d
  
Para verificar el log de la creación del stack puede ver los logs:
  
    $ docker-compose logs
