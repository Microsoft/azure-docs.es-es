---
title: Realización de detección de intrusiones en la red con Azure Network Watcher y herramientas de código abierto | Microsoft Docs
description: En este artículo se describe cómo usar Azure Network Watcher y las herramientas de código abierto para realizar la detección de intrusiones en la red.
services: network-watcher
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
ms.assetid: 0f043f08-19e1-4125-98b0-3e335ba69681
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: kumud
ms.openlocfilehash: b5dc885611c6654c1dc1d236e41ed75ef2717cc1
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65605707"
---
# <a name="perform-network-intrusion-detection-with-network-watcher-and-open-source-tools"></a>Realización de detección de intrusiones en la red con Azure Network Watcher y herramientas de código abierto

Las capturas de paquetes son un componente clave de la implementación de sistemas de detección de intrusiones de red (IDS) y de la ejecución de supervisión de seguridad de la red (NSM). Existen varias herramientas IDS de código abierto que procesan las capturas de paquetes y buscan firmas de posibles intrusiones y actividad malintencionada. Con las capturas de paquetes que proporciona Network Watcher, puede analizar la red en busca de intrusiones o vulnerabilidades de carácter dañino.

Una de esas herramientas de código abierto es Suricata, un motor de IDS que usa conjuntos de reglas para supervisar el tráfico de red y desencadenar alertas cuando se produce algún evento sospechoso. Suricata ofrece un motor multiproceso, lo que significa que puede realizar el análisis del tráfico de red con una mayor velocidad y eficiencia. Para más información sobre Suricata y sus funcionalidades, visite este sitio web en https://suricata-ids.org/.

## <a name="scenario"></a>Escenario

En este artículo se explica cómo configurar el entorno para realizar la detección de intrusiones de red con Network Watcher, Suricata y Elastic Stack. Network Watcher proporciona las capturas de paquetes usadas para realizar la detección de intrusiones de red. Suricata procesa las capturas de paquetes y desencadena alertas en función de los paquetes que coinciden con su conjunto de reglas especificada de amenazas. Estas alertas se almacenan en un archivo de registro en la máquina local. Con Elastic Stack, los registros que genera Suricata se pueden indexar y usar para crear un panel de Kibana. Este panel proporciona una representación visual de los registros y un medio de obtener rápidamente información detallada sobre posibles vulnerabilidades de la red.  

![escenario sencillo de aplicación web][1]

Ambas herramientas de código abierto se pueden configurar en una máquina virtual de Azure, lo que le permite realizar este análisis dentro de su propio entorno de red de Azure.

## <a name="steps"></a>Pasos

### <a name="install-suricata"></a>Instalación de Suricata

Para todos los demás métodos de instalación, visite https://suricata.readthedocs.io/en/latest/install.html.

1. En el terminal de la línea de comandos de la máquina virtual, ejecute los siguientes comandos:

    ```
    sudo add-apt-repository ppa:oisf/suricata-stable
    sudo apt-get update
    sudo sudo apt-get install suricata
    ```

1. Para comprobar la instalación, ejecute el comando `suricata -h` para ver la lista completa de comandos.

### <a name="download-the-emerging-threats-ruleset"></a>Descarga del conjunto de reglas Emerging Threats

En esta fase, no tenemos ninguna regla para ejecutar con Suricata. Puede crear sus propias reglas si hay amenazas específicas para su red que quiera detectar, o también puede usar conjuntos de reglas desarrolladas por varios proveedores, como Emerging Threats o las reglas VRT de Snort. Aquí usaremos el conjunto de reglas de acceso gratuito Emerging Threats:

Descargue el conjunto de reglas y cópielo en el directorio:

```
wget https://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz
tar zxf emerging.rules.tar.gz
sudo cp -r rules /etc/suricata/
```

### <a name="process-packet-captures-with-suricata"></a>Procesamiento de las capturas de paquetes con Suricata

Para procesar las capturas de paquetes mediante Suricata, ejecute el siguiente comando:

```
sudo suricata -c /etc/suricata/suricata.yaml -r <location_of_pcapfile>
```
Para comprobar las alertas resultantes, lea el archivo fast.log:
```
tail -f /var/log/suricata/fast.log
```

### <a name="set-up-the-elastic-stack"></a>Configuración de Elastic Stack

Si bien los registros que produce Suricata contienen información valiosa sobre lo que sucede en la red, estos archivos de registro no son los más fáciles de leer y comprender. Mediante la conexión a Suricata con Elastic Stack, podemos crear un panel de Kibana que nos permita buscar, representar, analizar y deducir información de nuestros registros.

#### <a name="install-elasticsearch"></a>Instalación de Elasticsearch

1. La versión 5.0 y superiores de Elastic Stack requieren Java 8. Ejecute el comando `java -version` para comprobar la versión. Si no tiene instalado Java, consulte la documentación en los [JDK compatibles con Azure](https://aka.ms/azure-jdks).

1. Descargue el paquete binario correcto para su sistema:

    ```
    curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.0.deb
    sudo dpkg -i elasticsearch-5.2.0.deb
    sudo /etc/init.d/elasticsearch start
    ```

    Puede encontrar otros métodos de instalación en [Elasticsearch Installation](https://www.elastic.co/guide/en/beats/libbeat/5.2/elasticsearch-installation.html) (Instalación de Elasticsearch).

1. Compruebe que Elasticsearch se esté ejecutando con el comando:

    ```
    curl http://127.0.0.1:9200
    ```

    La respuesta debe ser similar a la siguiente:

    ```
    {
    "name" : "Angela Del Toro",
    "cluster_name" : "elasticsearch",
    "version" : {
        "number" : "5.2.0",
        "build_hash" : "8ff36d139e16f8720f2947ef62c8167a888992fe",
        "build_timestamp" : "2016-01-27T13:32:39Z",
        "build_snapshot" : false,
        "lucene_version" : "6.1.0"
    },
    "tagline" : "You Know, for Search"
    }
    ```

Para más instrucciones sobre cómo instalar Elasticsearch, consulte la página de [instalación](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/_installation.html).

### <a name="install-logstash"></a>Instalación de Logstash

1. Para instalar Logstash, ejecute los siguientes comandos:

    ```
    curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
    sudo dpkg -i logstash-5.2.0.deb
    ```
1. A continuación, se debe configurar Logstash para que lea la salida del archivo eve.json. Cree un archivo logstash.conf mediante:

    ```
    sudo touch /etc/logstash/conf.d/logstash.conf
    ```

1. Agregue el contenido siguiente al archivo (asegúrese de que la ruta de acceso al archivo eve.json sea correcta):

    ```ruby
    input {
    file {
        path => ["/var/log/suricata/eve.json"]
        codec =>  "json"
        type => "SuricataIDPS"
    }

    }

    filter {
    if [type] == "SuricataIDPS" {
        date {
        match => [ "timestamp", "ISO8601" ]
        }
        ruby {
        code => "
            if event.get('[event_type]') == 'fileinfo'
            event.set('[fileinfo][type]', event.get('[fileinfo][magic]').to_s.split(',')[0])
            end
        "
        }

        ruby{
        code => "
            if event.get('[event_type]') == 'alert'
            sp = event.get('[alert][signature]').to_s.split(' group ')
            if (sp.length == 2) and /\A\d+\z/.match(sp[1])
                event.set('[alert][signature]', sp[0])
            end
            end
            "
        }
    }

    if [src_ip]  {
        geoip {
        source => "src_ip"
        target => "geoip"
        #database => "/opt/logstash/vendor/geoip/GeoLiteCity.dat"
        add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
        add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
        }
        mutate {
        convert => [ "[geoip][coordinates]", "float" ]
        }
        if ![geoip.ip] {
        if [dest_ip]  {
            geoip {
            source => "dest_ip"
            target => "geoip"
            #database => "/opt/logstash/vendor/geoip/GeoLiteCity.dat"
            add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
            add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
            }
            mutate {
            convert => [ "[geoip][coordinates]", "float" ]
            }
        }
        }
    }
    }

    output {
    elasticsearch {
        hosts => "localhost"
    }
    }
    ```

1. Asegúrese de conceder los permisos correctos para el archivo eve.json de modo que Logstash pueda ingerir el archivo.
    
    ```
    sudo chmod 775 /var/log/suricata/eve.json
    ```

1. Para iniciar Logstash, ejecute el comando:

    ```
    sudo /etc/init.d/logstash start
    ```

Para obtener más instrucciones sobre cómo instalar Logstash, consulte la [documentación oficial](https://www.elastic.co/guide/en/beats/libbeat/5.2/logstash-installation.html).

### <a name="install-kibana"></a>Instalación de Kibana

1. Ejecute los siguientes comandos para instalar Kibana:

    ```
    curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-5.2.0-linux-x86_64.tar.gz
    tar xzvf kibana-5.2.0-linux-x86_64.tar.gz

    ```
1. Para ejecutar Kibana, use los comandos:

    ```
    cd kibana-5.2.0-linux-x86_64/
    ./bin/kibana
    ```

1. Para ver la interfaz web de Kibana, vaya a `http://localhost:5601`.
1. En este escenario, el patrón de índice usado para los registros de Suricata es "logstash-*".

1. Si quiere ver el panel de Kibana de forma remota, cree una regla NSG de entrada que permita acceder al **puerto 5601**.

### <a name="create-a-kibana-dashboard"></a>Creación de un panel de Kibana

En este artículo, hemos proporcionado un panel de ejemplo para que vea las tendencias y los detalles de sus alertas.

1. Descargue el archivo de panel [aquí](https://aka.ms/networkwatchersuricatadashboard), el archivo de visualización [aquí](https://aka.ms/networkwatchersuricatavisualization) y el archivo de búsqueda guardado [aquí](https://aka.ms/networkwatchersuricatasavedsearch).

1. En la pestaña **Management** (Administración) de Kibana, vaya a **Saved Objects** (Objetos guardados) e importe los tres archivos. A continuación, en la pestaña **Dashboard** (Panel), puede abrir y cargar el panel de ejemplo.

También puede crear visualizaciones y paneles propios que se adapten a las métricas que le interesen. Puede leer más sobre la creación de visualizaciones de Kibana en la [documentación oficial](https://www.elastic.co/guide/en/kibana/current/visualize.html) de Kibana.

![panel de kibana][2]

### <a name="visualize-ids-alert-logs"></a>Visualización de registros de alertas de IDS

El panel de ejemplo proporciona varias visualizaciones de los registros de alerta de Suricata:

1. Alertas por GeoIP: mapa que muestra la distribución de las alertas por su país o región de origen según la ubicación geográfica (determinada por IP)

    ![ip geográfica][3]

1. 10 alertas principales: resumen de las 10 alertas activadas más frecuentes y su descripción. Al hacer clic en una alerta individual se filtra el panel por la información relacionada con esa alerta específica.

    ![imagen 4][4]

1. Número de alertas: el número total de alertas activadas por el conjunto de reglas.

    ![imagen 5][5]

1. 20 puertos/direcciones IP de origen y destino principales: gráficos circulares que muestran los 20 puertos y direcciones IP principales en los que se activaron las alertas. Puede filtrar por direcciones IP o puertos específicos para ver cuántas alertas y de qué tipo se han activado.

    ![imagen 6][6]

1. Resumen de alertas: tabla que resume detalles específicos de cada alerta individual. Puede personalizar esta tabla para mostrar otros parámetros de interés para cada alerta.

    ![imagen 7][7]

Para ver más documentación sobre la creación de visualizaciones y paneles personalizados, consulte la [documentación oficial del Kibana](https://www.elastic.co/guide/en/kibana/current/introduction.html).

## <a name="conclusion"></a>Conclusión

Al combinar las capturas de paquetes proporcionadas por Network Watcher con herramientas IDS de código abierto como Suricata, puede realizar la detección de intrusiones para una amplia variedad de amenazas. Estos paneles permiten detectar rápidamente tendencias y anomalías de la red, así como escarbar en los datos para descubrir las causas principales de las alertas, como agentes de usuario malintencionados o puertos vulnerables. Con estos datos extraídos, puede tomar decisiones fundamentadas sobre cómo proteger su red y reaccionar ante intentos de intrusión perjudiciales en ella, y crear reglas para impedir que sucedan tales situaciones.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a desencadenar capturas de paquetes en función de alertas en el artículo [Uso de la captura de paquetes para realizar la supervisión proactiva de redes con Azure Functions](network-watcher-alert-triggered-packet-capture.md).

Aprenda a visualizar los registros de flujo de grupo de seguridad de red con Power BI en el artículo [Visualize NSG flows logs with Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md) (Visualización de registros de flujo de grupo de seguridad de red con Power BI).



<!-- images -->
[1]: ./media/network-watcher-intrusion-detection-open-source-tools/figure1.png
[2]: ./media/network-watcher-intrusion-detection-open-source-tools/figure2.png
[3]: ./media/network-watcher-intrusion-detection-open-source-tools/figure3.png
[4]: ./media/network-watcher-intrusion-detection-open-source-tools/figure4.png
[5]: ./media/network-watcher-intrusion-detection-open-source-tools/figure5.png
[6]: ./media/network-watcher-intrusion-detection-open-source-tools/figure6.png
[7]: ./media/network-watcher-intrusion-detection-open-source-tools/figure7.png
