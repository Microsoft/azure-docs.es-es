---
title: Conexión de orígenes de datos a través de Logstash a Azure Sentinel| Microsoft Docs
description: Aprenda a usar Logstash para reenviar registros desde orígenes de datos externos a Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/10/2020
ms.author: yelevin
ms.openlocfilehash: 7fe47289dcc6b6d6af4d13b36b5c3b1dae3baaf5
ms.sourcegitcommit: f8d2ae6f91be1ab0bc91ee45c379811905185d07
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89663397"
---
# <a name="use-logstash-to-connect-data-sources-to-azure-sentinel"></a>Uso de Logstash para conectar orígenes de datos a Azure Sentinel

> [!IMPORTANT]
> La ingesta de datos mediante el complemento de salida de Logstash se encuentra actualmente en versión preliminar pública. Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Con el nuevo complemento de salida de Azure Sentinel para el **motor de recopilación de datos de Logstash**, ahora puede enviar cualquier tipo de registro que quiera a través de Logstash directamente a su área de trabajo de Log Analytics en Azure Sentinel. Los registros se enviarán a una tabla personalizada que definirá mediante el complemento de salida.

Para obtener más información sobre cómo trabajar con el motor de recopilación de datos de Logstash, consulte la página de [introducción a Logstash](https://www.elastic.co/guide/en/logstash/current/getting-started-with-logstash.html).

## <a name="overview"></a>Información general

### <a name="architecture-and-background"></a>Arquitectura e información

![Diagrama de la arquitectura de Logstash.](./media/connect-logstash/logstash-architecture.png)

El motor de Logstash consta de tres componentes:

- Complementos de entrada: colección personalizada de datos de diversos orígenes.
- Complementos de filtro: manipulación y normalización de los datos según los criterios especificados.
- Complementos de salida: envío personalizado de datos recopilados y procesados a varios destinos.

> [!NOTE]
> Azure Sentinel solo admite el complemento de salida propio que ofrece. No admite complementos de salida de terceros para Azure Sentinel ni cualquier otro complemento de Logstash de ningún tipo.

El complemento de salida de Azure Sentinel para Logstash envía datos con formato JSON al área de trabajo de Log Analytics, mediante la API REST de recopilación de datos HTTP de Log Analytics. Los datos se ingieren en los registros personalizados.

- Más información sobre la [API REST de Log Analytics](https://docs.microsoft.com/rest/api/loganalytics/create-request).
- Más información sobre los [registros personalizados](https://docs.microsoft.com/azure/azure-monitor/platform/data-sources-custom-logs).

## <a name="deploy-the-azure-sentinel-output-plugin-in-logstash"></a>Implementación del complemento de salida de Azure Sentinel en Logstash

### <a name="step-1-installation"></a>Paso 1: Instalación

El complemento de salida de Azure Sentinel está disponible en la colección de Logstash.

- Siga las instrucciones que aparecen en el documento de Logstash sobre cómo [usar los complementos](https://www.elastic.co/guide/en/logstash/current/working-with-plugins.html) para instalar el complemento ***microsoft-logstash-output-azure-loganalytics***.
   
- Si el sistema de Logstash no tiene acceso a Internet, siga las instrucciones del documento de Logstash sobre [administración de complementos sin conexión](https://www.elastic.co/guide/en/logstash/current/offline-plugins.html) para preparar y usar un paquete de complementos sin conexión. Para ello, hay que compilar otro sistema de Logstash con acceso a Internet.

### <a name="step-2-configuration"></a>Paso 2: Configuración

Use la información del documento [Estructura de un archivo de configuración](https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html) de Logstash y agregue el complemento de salida de Azure Sentinel a la configuración con las claves y los valores siguientes. (La sintaxis correcta del archivo de configuración se muestra después de la tabla).

| Nombre del campo | Tipo de datos | Descripción |
|----------------|---------------|-----------------|
| `workspace_id` | string | Escriba el GUID del identificador del área de trabajo. * |
| `workspace_key` | string | Escriba el GUID de la clave principal del área de trabajo. * |
| `custom_log_table_name` | string | Establezca el nombre de la tabla en la que se van a ingerir los registros. Solo se puede configurar un nombre de tabla por complemento de salida. La tabla de registros aparecerá en Azure Sentinel en **Registros**, en **Tablas**, categoría **Registros personalizados**, con un sufijo `_CL`. |
| `endpoint` | string | Campo opcional. De forma predeterminada, es el punto de conexión de Log Analytics. Utilice este campo para establecer un punto de conexión alternativo. |
| `time_generated_field` | string | Campo opcional. Esta propiedad invalida el campo predeterminado **TimeGenerated** en Log Analytics. Escriba el nombre del campo de marca de tiempo en el origen de datos. Los datos del campo deben ajustarse al formato ISO 8601 (`YYYY-MM-DDThh:mm:ssZ`) |
| `key_names` | array | Escriba una lista de campos de esquema de salida de Log Analytics. Cada elemento de lista se debe incluir entre comillas simples, los elementos deben estar separados por comas y la lista completa entre corchetes. Consulte el ejemplo siguiente. |
| `plugin_flush_interval` | number | Campo opcional. Establézcalo para definir el intervalo máximo (en segundos) entre las transmisiones de mensajes a Log Analytics. El valor predeterminado es 5. |
    | `amount_resizing` | boolean | True o false. Habilite o deshabilite el mecanismo de escalabilidad automática, que ajusta el tamaño del búfer de mensajes en función del volumen de datos de registro recibidos. |
| `max_items` | number | Campo opcional. Solo se aplica si `amount_resizing` se establece en "false". Úselo para establecer un límite en el tamaño del búfer de mensajes (en registros). El valor predeterminado es 2000.  |

\* Puede encontrar el identificador y la clave principal del área de trabajo en el recurso del área de trabajo, en **Agents management** (Administración de agentes).

#### <a name="sample-configurations"></a>Configuraciones de ejemplo

Estas son algunas configuraciones de ejemplo que usan algunas opciones diferentes.

- Una configuración básica que usa una canalización de entrada Filebeat:

   ```ruby
    input {
        beats {
            port => "5044"
        }
    }
    filter {
    }
    output {
        microsoft-logstash-output-azure-loganalytics {
          workspace_id => "4g5tad2b-a4u4-147v-a4r7-23148a5f2c21" # <your workspace id>
          workspace_key => "u/saRtY0JGHJ4Ce93g5WQ3Lk50ZnZ8ugfd74nk78RPLPP/KgfnjU5478Ndh64sNfdrsMni975HJP6lp==" # <your workspace key>
          custom_log_table_name => "tableName"
        }
    }
   ```
    
- Una configuración básica que usa una canalización de entrada TCP:

   ```ruby
    input {
        tcp {
            port => "514"
            type => syslog #optional, will effect log type in table
        }
    }
    filter {
    }
    output {
        microsoft-logstash-output-azure-loganalytics {
          workspace_id => "4g5tad2b-a4u4-147v-a4r7-23148a5f2c21" # <your workspace id>
          workspace_key => "u/saRtY0JGHJ4Ce93g5WQ3Lk50ZnZ8ugfd74nk78RPLPP/KgfnjU5478Ndh64sNfdrsMni975HJP6lp==" # <your workspace key>
          custom_log_table_name => "tableName"
        }
    }
   ```
  
- Una configuración avanzada:

   ```ruby    
    input {
        tcp {
            port => 514
            type => syslog
        }
    }
    filter {
        grok {
            match => { "message" => "<%{NUMBER:PRI}>1 (?<TIME_TAG>[0-9]{4}-[0-9]{1,2}-[0-9]{1,2}T[0-9]{1,2}:[0-9]{1,2}:[0-9]{1,2})[^ ]* (?<HOSTNAME>[^ ]*) %{GREEDYDATA:MSG}" }
        }
    }
    output {
        microsoft-logstash-output-azure-loganalytics {
            workspace_id => "<WS_ID>"
            workspace_key => "${WS_KEY}"
            custom_log_table_name => "logstashCustomTable"
            key_names => ['PRI','TIME_TAG','HOSTNAME','MSG']
            plugin_flush_interval => 5
        }
    } 
   ```

   > [!NOTE]
   > Visite el [repositorio de GitHub](https://github.com/Azure/Azure-Sentinel/tree/master/DataConnectors/microsoft-logstash-output-azure-loganalytics) del complemento de salida para más información sobre sus trabajos internos, la configuración y las opciones de rendimiento.

### <a name="step-3-restart-logstash"></a>Paso 3: Reinicio de Logstash

### <a name="step-4-view-incoming-logs-in-azure-sentinel"></a>Paso 4: Visualización de registros entrantes en Azure Sentinel

1. Compruebe que los mensajes se envían al complemento de salida.

1. En el menú de navegación de Azure Sentinel, abra **Registros**. En el encabezado **Tablas**, expanda la categoría **Registros personalizados**. Busque y haga clic en el nombre de la tabla que especificó (con un sufijo `_CL`) en la configuración.

   :::image type="content" source="./media/connect-logstash/logstash-custom-logs-menu.png" alt-text="Captura de pantalla de registros personalizados de Logstash.":::

1. Para ver los registros de la tabla, consulte la tabla usando su nombre como esquema.

   :::image type="content" source="./media/connect-logstash/logstash-custom-logs-query.png" alt-text="Captura de pantalla de una consulta de registros personalizados de Logstash.":::

## <a name="monitor-output-plugin-audit-logs"></a>Supervisión de registros de auditoría de complementos de salida

Para supervisar la conectividad y la actividad del complemento de salida de Azure Sentinel, habilite el archivo de registro de Logstash adecuado. Consulte el documento [Diseño del directorio de Logstash](https://www.elastic.co/guide/en/logstash/current/dir-layout.html#dir-layout) para conocer la ubicación del archivo de registro.

Si no ve ningún dato en este archivo de registro, genere y envíe algunos eventos localmente (a través de los complementos de entrada y filtro) para asegurarse de que el complemento de salida está recibiendo datos. Azure Sentinel solo ofrece soporte técnico para los problemas relacionados con el complemento de salida.

## <a name="next-steps"></a>Pasos siguientes

En este documento, aprendió a usar Logstash para conectar orígenes de datos externos a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a detectar amenazas con Azure Sentinel mediante las reglas [integradas](tutorial-detect-threats-built-in.md) o [personalizadas](tutorial-detect-threats-custom.md).
