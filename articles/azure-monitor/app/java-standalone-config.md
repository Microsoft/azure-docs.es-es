---
title: 'Supervisión de aplicaciones Java en cualquier lugar: Application Insights de Azure Monitor'
description: Supervisión del rendimiento de aplicaciones sin código para aplicaciones Java que se ejecutan en cualquier entorno sin instrumentar la aplicación. Descubra la causa principal de los problemas mediante el seguimiento distribuido y el mapa de aplicación.
ms.topic: conceptual
ms.date: 04/16/2020
ms.custom: devx-track-java
ms.openlocfilehash: ca3094197deb7c74ba9b51422a78ee0f5d3687d2
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87374293"
---
# <a name="configuration-options---java-standalone-agent-for-azure-monitor-application-insights"></a>Opciones de configuración: agente independiente de Java para Application Insights de Azure Monitor



## <a name="connection-string-and-role-name"></a>Cadena de conexión y nombre de rol

```json
{
  "instrumentationSettings": {
    "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
    "preview": {
      "roleName": "my cloud role name"
    }
  }
}
```

La cadena de conexión es obligatoria y el nombre del rol es importante cada vez que se envían datos de diferentes aplicaciones al mismo recurso de Application Insights.

Encontrará más detalles y opciones de configuración adicionales a continuación.

## <a name="configuration-file-path"></a>Ruta del archivo de configuración

De forma predeterminada, Application Insights Java 3.0 Preview espera que el archivo de configuración se denomine `ApplicationInsights.json` y que se encuentre en el mismo directorio que el archivo `applicationinsights-agent-3.0.0-PREVIEW.5.jar`.

Puede especificar la ruta de acceso a su propio archivo de configuración mediante

* la variable de entorno `APPLICATIONINSIGHTS_CONFIGURATION_FILE`, o
* la propiedad del sistema Java `applicationinsights.configurationFile`.

Si especifica una ruta de acceso relativa, se resolverá de forma relativa al directorio en el que se encuentra `applicationinsights-agent-3.0.0-PREVIEW.5.jar`.

## <a name="connection-string"></a>Cadena de conexión

Este es un paso necesario. Puede encontrar la cadena de conexión en el recurso de Application Insights:

:::image type="content" source="media/java-ipa/connection-string.png" alt-text="Cadena de conexión a Application Insights":::

También puede establecer la cadena de conexión mediante la variable de entorno `APPLICATIONINSIGHTS_CONNECTION_STRING`.

## <a name="cloud-role-name"></a>Nombre del rol en la nube

Este se usa para etiquetar el componente en el mapa de aplicación.

Si quiere establecer el nombre del rol de nube, haga lo siguiente:

```json
{
  "instrumentationSettings": {
    "preview": {   
      "roleName": "my cloud role name"
    }
  }
}
```

Si no se establece el nombre del rol en la nube, se usará el nombre del recurso de Application Insights para etiquetar el componente en el mapa de aplicación.

También puede establecer el nombre de rol en la nube mediante la variable de entorno `APPLICATIONINSIGHTS_ROLE_NAME`.

## <a name="cloud-role-instance"></a>Instancia de rol en la nube

La instancia de rol de la nube tiene como valor predeterminado el nombre del equipo.

Si quiere establecer la instancia de rol de nube en un valor diferente en lugar del nombre del equipo, use lo siguiente:

```json
{
  "instrumentationSettings": {
    "preview": {
      "roleInstance": "my cloud role instance"
    }
  }
}
```

También puede establecer la instancia de rol en la nube mediante la variable de entorno `APPLICATIONINSIGHTS_ROLE_INSTANCE`.

## <a name="application-log-capture"></a>Captura del registro de aplicaciones

Application Insights Java 3.0 Preview captura automáticamente el registro de aplicaciones mediante Log4j, Logback y java.util.logging.

De forma predeterminada, capturará todos los registros realizados en el nivel `WARN` o superior.

Si quiere cambiar este umbral, use lo siguiente:

```json
{
  "instrumentationSettings": {
    "preview": {
      "instrumentation": {
        "logging": {
          "threshold": "ERROR"
        }
      }
    }
  }
}
```

Los siguientes son los valores `threshold` válidos que puede especificar en el archivo `ApplicationInsights.json` y cómo se corresponden con los niveles de registro en diferentes marcos de registro:

| `threshold`  | Log4j  | Logback | JUL     |
|--------------|--------|---------|---------|
| Apagado          | Apagado    | Apagado     | Apagado     |
| FATAL        | FATAL  | ERROR   | SEVERE  |
| ERROR/SEVERE | ERROR  | ERROR   | SEVERE  |
| WARN/WARNING | WARN   | WARN    | WARNING |
| INFO         | INFO   | INFO    | INFO    |
| CONFIG       | DEBUG  | DEBUG   | CONFIG  |
| DEBUG/FINE   | DEBUG  | DEBUG   | FINE    |
| FINER        | DEBUG  | DEBUG   | FINER   |
| TRACE/FINEST | TRACE  | TRACE   | FINEST  |
| ALL          | ALL    | ALL     | ALL     |

## <a name="jmx-metrics"></a>Métricas JMX

Si tiene alguna métrica de JMX que le interese capturar, use lo siguiente:

```json
{
  "instrumentationSettings": {
    "preview": {
        "jmxMetrics": [
        {
          "objectName": "java.lang:type=ClassLoading",
          "attribute": "LoadedClassCount",
          "display": "Loaded Class Count"
        },
        {
          "objectName": "java.lang:type=MemoryPool,name=Code Cache",
          "attribute": "Usage.used",
          "display": "Code Cache Used"
        }
      ]
    }
  }
}
```

## <a name="micrometer-including-metrics-from-spring-boot-actuator"></a>Micrometer (incluidas las métricas del actuador de Spring Boot)

Si la aplicación usa [Micrometer](https://micrometer.io), Application Insights 3.0 (a partir de Preview.2) captura ahora las métricas enviadas al registro global de Micrometer.

Si la aplicación usa el [actuador de Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html), Application Insights 3.0 (a partir de Preview.4) captura ahora las métricas configuradas por este (que usa Micrometer, pero no usa el registro global de Micrometer).

Si quiere deshabilitar estas características, haga lo siguiente:

```json
{
  "instrumentationSettings": {
    "preview": {
      "instrumentation": {
        "micrometer": {
          "enabled": false
        }
      }
    }
  }
}
```

## <a name="heartbeat"></a>Latido

De forma predeterminada, Application Insights Java 3.0 Preview envía una métrica de latido cada 15 minutos. Si está usando la métrica de latido para desencadenar las alertas, puede aumentar la frecuencia de este latido:

```json
{
  "instrumentationSettings": {
    "preview": {
        "heartbeat": {
            "intervalSeconds": 60
        }
    }
  }
}
```

> [!NOTE]
> No puede reducir la frecuencia del latido, ya que los datos del mismo también se usan para supervisar el uso de Application Insights.

## <a name="sampling"></a>muestreo

El muestreo resulta útil si necesita reducir costos.
Este se realiza como una función en el id. de operación (también conocido como id. de seguimiento), por lo que el mismo id. de operación siempre dará como resultado la misma decisión de muestreo. Esto garantiza que no se realizará el muestreo de algunas partes de una transacción distribuida mientras se muestrean otras.

Por ejemplo, si establece el muestreo en 10 %, solo verá el 10 % de sus transacciones, pero cada una de ese 10 % tendrá los detalles completos de las transacciones de un extremo a otro.

El siguiente es un ejemplo de cómo establecer el muestreo en **10 % de todas las transacciones**. Asegúrese de establecer la velocidad de muestreo correcta para su caso de uso:

```json
{
  "instrumentationSettings": {
    "preview": {
        "sampling": {
            "fixedRate": {
                "percentage": 10
            }
          }
        }
    }
}
```

## <a name="http-proxy"></a>Proxy HTTP

Si su aplicación está protegida por un firewall y no puede conectarse directamente a Application Insights (consulte [Direcciones IP que emplea Application Insights](./ip-addresses.md)), puede configurar Application Insights Java 3.0 Preview para que use un proxy HTTP:

```json
{
  "instrumentationSettings": {
    "preview": {
      "httpProxy": {
        "host": "myproxy",
        "port": 8080
      }
    }
  }
}
```

## <a name="self-diagnostics"></a>Diagnóstico automático

"Diagnóstico automático" hace referencia al registro interno de Application Insights Java 3.0 Preview.

Puede ser útil para detectar y diagnosticar problemas con Application Insights.

De forma predeterminada, se registra en la consola con el nivel `warn`, que corresponde a la siguiente configuración:

```json
{
  "instrumentationSettings": {
    "preview": {
        "selfDiagnostics": {
            "destination": "console",
            "level": "WARN"
        }
    }
  }
}
```

Los niveles válidos son `OFF`, `ERROR`, `WARN`, `INFO`, `DEBUG` y `TRACE`.

Si quiere realizar el registro en un archivo en lugar de en la consola:

```json
{
  "instrumentationSettings": {
    "preview": {
        "selfDiagnostics": {
            "destination": "file",
            "directory": "/var/log/applicationinsights",
            "level": "WARN",
            "maxSizeMB": 10
        }    
    }
  }
}
```

Cuando se usa el registro de archivos, una vez que el archivo alcanza el límite `maxSizeMB`, se realiza una sustitución, de modo que se conserva solo el archivo de registro completado más recientemente, además del archivo de registro actual.
