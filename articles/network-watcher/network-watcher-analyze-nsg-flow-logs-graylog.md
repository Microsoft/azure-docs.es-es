---
title: Análisis de registros de flujo de grupo de seguridad de red de Azure con Graylog | Microsoft Docs
description: Aprenda a administrar y analizar registros de flujo de grupo de seguridad de red en Azure con Network Watcher y Graylog.
services: network-watcher
documentationcenter: na
author: damendo
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/19/2017
ms.author: damendo
ms.openlocfilehash: 7a4aa4cc545d6941f144ce0657ede7199d4f8f57
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86497121"
---
# <a name="manage-and-analyze-network-security-group-flow-logs-in-azure-using-network-watcher-and-graylog"></a>Administre y analice registros de flujo de grupo de seguridad de red en Azure con Network Watcher y Graylog.

Los [registros de flujo de grupo de seguridad de red](network-watcher-nsg-flow-logging-overview.md) proporcionan información que se usa para comprender el tráfico IP de entrada y salida para las interfaces de red de Azure. Los registros de flujo muestran los flujos de entrada y salida en función de cada regla de grupo de seguridad de red, la interfaz de red a la que se aplica el flujo, información de 5-tupla (IP de origen y de destino, puerto de origen y de destino, protocolo) sobre el flujo, y si se permitió o denegó el tráfico.

Puede tener varios grupos de seguridad de red en la red con el registro de flujo habilitado. Puede resultar complicado analizar varios grupos de seguridad de red con el registro de flujo habilitado y obtener información de los registros. En este artículo se proporciona una solución para administrar de forma centralizada estos registros de flujo de grupo de seguridad de red mediante Graylog, una herramienta de análisis y administración de registros de código abierto y Logstash, una canalización de procesamiento de datos en el servidor de código abierto.

> [!Warning]
> En los pasos siguientes se trabajará con la versión 1 de los registros de flujo. Para obtener más detalles, vea [Introducción al registro de flujo de grupos de seguridad de red](network-watcher-nsg-flow-logging-overview.md). Las siguientes instrucciones no funcionarán con la versión 2 de los archivos de registro, sin ninguna modificación.

## <a name="scenario"></a>Escenario

Los registros de flujo de grupo de seguridad de red se habilitan mediante Network Watcher. Los registros de flujo se dirigen al almacenamiento de blobs de Azure. Se usa un complemento de Logstash para conectar y procesar los registros de flujo del almacenamiento de blobs y enviarlos a Graylog. Cuando los registros de flujo se almacenan en Graylog, se pueden analizar y visualizar en paneles personalizados.

![Flujo de trabajo de Graylog](./media/network-watcher-analyze-nsg-flow-logs-graylog/workflow.png)

## <a name="installation-steps"></a>Pasos de instalación

### <a name="enable-network-security-group-flow-logging"></a>Habilitación de los registros de flujo de grupo de seguridad de red

En este escenario, debe tener el registro de flujo de grupo de seguridad de red habilitado en al menos un grupo de seguridad de red de su cuenta. Para ver instrucciones para habilitar los registros de flujo de grupo de seguridad de red, consulte el artículo siguiente:  [Introducción a los registros de flujo de grupos de seguridad de red](network-watcher-nsg-flow-logging-overview.md).

### <a name="setting-up-graylog"></a>Configuración de Graylog

En este ejemplo, tanto Graylog como Logstash se configuran en un servidor Ubuntu 14.04, implementado en Azure.

- Consulte la [documentación](https://docs.graylog.org/en/2.2/pages/installation/os/ubuntu.html) en Graylog, para obtener instrucciones paso a paso acerca de cómo realizar la instalación en Ubuntu.
- Asegúrese de configurar también la interfaz web de Graylog siguiendo la [documentación](https://docs.graylog.org/en/2.2/pages/configuration/web_interface.html#configuring-webif).

En este ejemplo se utiliza la instalación mínima de Graylog (es decir, una única instancia de Graylog), pero se puede diseñar para escalar por los recursos en función de las necesidades del sistema y de producción. Para más información sobre consideraciones sobre la arquitectura o una guía de arquitectura de profundidad, consulte la [documentación](https://docs.graylog.org/en/2.2/pages/architecture.html) y la [guía de arquitectura](https://www.slideshare.net/Graylog/graylog-engineering-design-your-architecture) de Graylog.

Graylog puede instalarse de muchas maneras, dependiendo de sus preferencias y de la plataforma. Para obtener una lista completa de los posibles métodos de instalación , consulte la [documentación](https://docs.graylog.org/en/2.2/pages/installation.html) oficial de Graylog. La aplicación del servidor de Graylog se ejecuta en las distribuciones Linux y tiene los siguientes requisitos previos:

-  Java SE 8 o posterior: [documentación de Azul Azure JDK](https://aka.ms/azure-jdks)
-  Elasticsearch 2.x (2.1.0 o posterior): [documentación de instalación de Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/_installation.html)
-  MongoDB 2.4 o posterior: [documentación de instalación de MongoDB](https://docs.mongodb.com/manual/administration/install-on-linux/)

### <a name="install-logstash"></a>Instalación de Logstash

Logstash sirve para aplanar los registros de flujo con formato JSON a un nivel de tupla de flujo. El aplanamiento de los registros de flujo facilita la organización y búsqueda de registros en Graylog.

1. Para instalar Logstash, ejecute los siguientes comandos:

   ```bash
   curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
   sudo dpkg -i logstash-5.2.0.deb
   ```

2. Configure Logstash para analizar los registros de flujo y enviarlos a Graylog. Cree un archivo Logstash.conf:

   ```bash
   sudo touch /etc/logstash/conf.d/logstash.conf
   ```

3. Agregue el siguiente contenido al archivo. Cambie los valores resaltados para reflejar los detalles de la cuenta de almacenamiento:

   ```
    input {
        azureblob
        {
            storage_account_name => "mystorageaccount"
            storage_access_key => "NrUZmx7pJSKaRJzvQbeiZWi5nBRWOTr7Wwr9DrvK7YtDBrADYxT1y0oEExtSlkDnGRt7qcRiZzEBCCyRYND8SxSt"
            container => "insights-logs-networksecuritygroupflowevent"
            registry_create_policy => "start_over"
            codec => "json"
            file_head_bytes => 21
            file_tail_bytes => 9
            # Possible options: `do_not_break`, `with_head_tail`, `without_head_tail`
            break_json_down_policy  => 'with_head_tail'
            break_json_batch_count => 2
            interval => 5
        }
    }
    
    filter {
        split { field => "[records]" }
        split { field => "[records][properties][flows]"}
        split { field => "[records][properties][flows][flows]"}
        split { field => "[records][properties][flows][flows][flowTuples]"
    }
    
     mutate {
        split => { "[records][resourceId]" => "/"}
        add_field =>{
                    "Subscription" => "%{[records][resourceId][2]}"
                    "ResourceGroup" => "%{[records][resourceId][4]}"
                    "NetworkSecurityGroup" => "%{[records][resourceId][8]}"
        }
        convert => {"Subscription" => "string"}
        convert => {"ResourceGroup" => "string"}
        convert => {"NetworkSecurityGroup" => "string"}
        split => { "[records][properties][flows][flows][flowTuples]" => ","}
        add_field => {
                    "unixtimestamp" => "%{[records][properties][flows][flows][flowTuples][0]}"
                    "srcIp" => "%{[records][properties][flows][flows][flowTuples][1]}"
                    "destIp" => "%{[records][properties][flows][flows][flowTuples][2]}"
                    "srcPort" => "%{[records][properties][flows][flows][flowTuples][3]}"
                    "destPort" => "%{[records][properties][flows][flows][flowTuples][4]}"
                    "protocol" => "%{[records][properties][flows][flows][flowTuples][5]}"
                    "trafficflow" => "%{[records][properties][flows][flows][flowTuples][6]}"
                    "traffic" => "%{[records][properties][flows][flows][flowTuples][7]}"
        }
        add_field => {
                    "time" => "%{[records][time]}"
                    "systemId" => "%{[records][systemId]}"
                    "category" => "%{[records][category]}"
                    "resourceId" => "%{[records][resourceId]}"
                    "operationName" => "%{[records][operationName}}"
                    "Version" => "%{[records][properties][Version}}"
                    "rule" => "%{[records][properties][flows][rule]}"
                    "mac" => "%{[records][properties][flows][flows][mac]}"
        }
        convert => {"unixtimestamp" => "integer"}
        convert => {"srcPort" => "integer"}
        convert => {"destPort" => "integer"}
        add_field => { "message" => "%{Message}" }
    }
        date {
            match => ["unixtimestamp" , "UNIX"]
        }
    }
    output {
        stdout { codec => rubydebug }
        udp {
            host => "127.0.0.1"
            port => 12201
        }
    }
    ```
   El archivo de configuración de Logstash proporcionado consta de tres partes: la entrada, el filtro y la salida. La sección de entrada designa el origen de entrada de los registros que va a procesar Logstash; en este caso, vamos a usar un complemento de entrada de blog de Azure (instalado en los pasos siguientes) que permite acceder a los archivos JSON del registro de flujo de grupo de seguridad de red almacenados en el almacenamiento de blobs.

La sección Filtro aplana, después, cada archivo de registro de flujo para que cada tupla flujo individual y sus propiedades asociadas se convierta en un evento de Logstash independiente.

Por último, la sección de salida desvía cada evento de Logstash al servidor de Graylog. Modifique el archivo de configuración de Logstash según sea necesario de acuerdo con sus necesidades específicas.

   > [!NOTE]
   > El archivo de configuración anterior presupone que se ha configurado el servidor de Graylog en la dirección IP de bucle invertido del host local 127.0.0.1. Si no es así, asegúrese de cambiar el parámetro del host en la sección de salida por la dirección IP correcta.

Para obtener más instrucciones sobre cómo instalar Logstash, consulte la [documentación](https://www.elastic.co/guide/en/beats/libbeat/5.2/logstash-installation.html) correspondiente.

### <a name="install-the-logstash-input-plug-in-for-azure-blob-storage"></a>Instalación del complemento de entrada de Logstash para Azure Blob Storage

El complemento de Logstash le permite acceder directamente a los registros de flujo desde su cuenta de Blob Storage designada. Para instalar el complemento, desde el directorio de instalación predeterminado de Logstash (en este caso /usr/share/logstash/bin), ejecute el comando siguiente:

```bash
cd /usr/share/logstash/bin
sudo ./logstash-plugin install logstash-input-azureblob
```

Para más información sobre este complemento, consulte la [documentación](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob).

### <a name="set-up-connection-from-logstash-to-graylog"></a>Configuración de la conexión desde Logstash a Graylog

Ahora que hemos establecido una conexión a los registros de flujo mediante Logstash y configurado el servidor de Graylog, es necesario configurar Graylog para aceptar los archivos de registro entrantes.

1. Navegue a la interfaz web del servidor de Graylog mediante la dirección de URL configurada para ella. Puede acceder a la interfaz dirigiendo el explorador a `http://<graylog-server-ip>:9000/`

2. Para navegar a la página de configuración, seleccione el menú desplegable **Sistema** en la barra de navegación superior y, después, haga clic en **Entradas**.
   O bien, vaya a `http://<graylog-server-ip>:9000/system/inputs`.

   ![Introducción](./media/network-watcher-analyze-nsg-flow-logs-graylog/getting-started.png)

3. Para iniciar la nueva entrada, seleccione *GELF UDP* en la lista desplegable **Seleccionar entrada** y, después, rellene el formulario. GELF es el acrónimo de Graylog Extended Log Format, o formato de registro extendido de Graylog. El formato GELF está desarrollado por Graylog. Para más información sobre sus ventajas, consulte la [documentación](https://docs.graylog.org/en/2.2/pages/gelf.html) de Graylog.

   Asegúrese de que va a enlazar la entrada a la dirección IP en la que se ha configurado el servidor de Graylog. La dirección IP debe coincidir con el **host** campo de la salida UDP del archivo de configuración de Logstash. El puerto predeterminado debe ser *12201*. Asegúrese de que el puerto coincida con el campo del **puerto** de la salida de UDP designado en el archivo de configuración de Logstash.

   ![Entradas](./media/network-watcher-analyze-nsg-flow-logs-graylog/inputs.png)

   Una vez que se inicia la entrada, debería ver que aparece bajo la sección **Entradas locales**, como se muestra en la siguiente imagen:

   ![Captura de pantalla en la que se muestra la sección de entradas locales que contiene la entrada que ha iniciado.](./media/network-watcher-analyze-nsg-flow-logs-graylog/local-inputs.png)

   Para más información sobre las entradas de mensaje de Graylog, consulte la [documentación](https://docs.graylog.org/en/2.2/pages/sending_data.html#what-are-graylog-message-inputs).

4. Cuando se hayan realizado estas configuraciones, puede iniciar Logstash para comenzar la lectura de los registros de flujo con el siguiente comando: `sudo systemctl start logstash.service`.

### <a name="search-through-graylog-messages"></a>Búsqueda a través de los mensajes de Graylog

Después de dejar algún tiempo para que el servidor de Graylog recopile los mensajes, ya se puede realizar la búsqueda por los mensajes. Para comprobar los mensajes que se envían al servidor de Graylog, desde la página de configuración **Entradas**, haga clic en el botón "**Show received messages**" (Mostrar mensajes recibidos) de la entrada de UDP de GELF que ha creado. Se le dirigirá a una pantalla que tiene un aspecto similar a la siguiente imagen: 

![Histograma](./media/network-watcher-analyze-nsg-flow-logs-graylog/histogram.png)

Al hacer clic en el vínculo azul "%{Message}", se expande cada mensaje para mostrar los parámetros de cada tupla de flujo, tal como se muestra en la siguiente imagen:

![error de Hadoop](./media/network-watcher-analyze-nsg-flow-logs-graylog/messages.png)

De forma predeterminada, se incluyen en la búsqueda todos los campos de mensaje si no se selecciona un campo de mensaje específico por el que buscar. Si desea buscar mensajes específicos (es decir, tuplas de flujo de una dirección IP de origen determinada), puede usar el lenguaje de consulta de búsqueda de Graylog como está [documentado](https://docs.graylog.org/en/2.2/pages/queries.html).

## <a name="analyze-network-security-group-flow-logs-using-graylog"></a>Análisis de registros de flujo de grupo de seguridad de red mediante Graylog

Ahora que Graylog se está ejecutando, podemos usar algunas de sus funcionalidades para comprender mejor los datos de registro de flujo. Una manera es usando paneles para crear vistas específicas de los datos.

### <a name="create-a-dashboard"></a>Creación de un panel

1. En la barra de navegación superior, seleccione **Paneles** o navegue a `http://<graylog-server-ip>:9000/dashboards/`.

2. Desde allí, haga clic en el botón verde **Crear panel** y rellene el breve formulario con el título y la descripción del panel. Pulse el botón **Guardar** para crear el nuevo panel. Verá un panel similar al de la siguiente imagen:

    ![Paneles](./media/network-watcher-analyze-nsg-flow-logs-graylog/dashboards.png)

### <a name="add-widgets"></a>Adición de widgets

Puede hacer clic en el título del panel para verlo, pero en este momento está vacío, ya que no hemos agregado ningún widget. Un widget sencillo y útil para agregar al panel son los gráficos **Quick Values** (Valores rápidos), que muestran una lista de valores del campo seleccionado y su distribución.

1. Vuelva a los resultados de búsqueda de la entrada de UDP que recibe los registros de flujo mediante la selección de **Búsqueda** en la barra de navegación superior.

2. En el panel **Resultado de búsqueda**, a la izquierda de la pantalla, busque la pestaña **Campos**, que muestra los distintos campos de cada mensaje de tupla de flujo entrante.

3. Seleccione cualquier parámetro deseado en el que se va a visualizar (en este ejemplo hemos seleccionado la dirección IP de origen). Para mostrar la lista de posibles widgets, haga clic en la flecha de lista desplegable azul a la izquierda del campo y, después, seleccione **Quick values** (Valores rápidos) para generar el widget. Debería ver algo parecido a la siguiente imagen:

   ![IP de origen](./media/network-watcher-analyze-nsg-flow-logs-graylog/srcip.png)

4. Desde allí, puede seleccionar el botón **Agregar al panel**, situado en la esquina superior derecha del widget, y seleccione el panel correspondiente para agregar.

5. Desplácese al panel para ver el widget que acaba de agregar.

   Puede agregar muchos otros widgets, como recuentos e histogramas en el panel, para realizar un seguimiento de métricas importantes, como el panel de ejemplo que se muestra en la siguiente imagen:

   ![Panel de Flowlogs](./media/network-watcher-analyze-nsg-flow-logs-graylog/flowlogs-dashboard.png)

    Para obtener más explicaciones sobre los paneles y los otros tipos de widgets, consulte la [documentación](https://docs.graylog.org/en/2.2/pages/dashboards.html) de Graylog.

Mediante la integración de Network Watcher con Graylog, ahora tenemos una forma cómoda y centralizada de administrar y ver los registros de flujo de grupo de seguridad de red. Graylog tiene muchas otras características eficaces, como secuencias y alertas, que pueden usarse también para administrar aún más los registros de flujo y comprender mejor el tráfico de red. Ahora que Graylog está configurado y conectado a Azure, no dude en seguir explorando todas las demás funcionalidades que ofrece.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a visualizar los registros de flujo de grupo de seguridad de red con Power BI en el artículo  [Visualización de registros de flujo del grupo de seguridad de red con Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md).
