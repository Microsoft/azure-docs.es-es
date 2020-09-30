---
title: Configuración de agentes de seguridad
description: Aprenda a configurar los agentes de seguridad de Defender para IoT para su uso con el servicio de seguridad de Defender para IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: c348b800e9587f13e6ff004317a2aa12efb03394
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932285"
---
# <a name="tutorial-configure-security-agents"></a>Tutorial: Configuración de agentes de seguridad

Este artículo se explica qué son los agentes de seguridad de Defender para IoT y se ofrecen detalles sobre cómo cambiarlos y configurarlos.

> [!div class="checklist"]
> * Configuración de agentes de seguridad
> * Cambiar el comportamiento del agente editando las propiedades del módulo gemelo
> * Detectar la configuración predeterminada

## <a name="agents"></a>Agentes

Los agentes de seguridad de Defender para IoT recopilan datos de dispositivos IoT y llevan a cabo acciones de seguridad para mitigar las vulnerabilidades detectadas. La configuración de seguridad de los agentes se puede controlar mediante un conjunto de propiedades del módulo gemelo que se pueden personalizar. Por lo general, las actualizaciones secundarias de estas propiedades son poco frecuentes.

El objeto de configuración gemelo del agente de seguridad de Defender para IoT es un objeto con formato JSON. El objeto de configuración consiste en un conjunto de propiedades controlables que se pueden definir para controlar el comportamiento del agente.

Estas configuraciones ayudan a personalizar al agente en cada escenario donde sea necesario. Así, por ejemplo, son posibles medidas como excluir automáticamente algunos eventos o mantener el consumo de energía en un nivel mínimo si se configuran estas propiedades.

Utilice el [esquema](https://aka.ms/iot-security-github-module-schema) de configuración del agente de seguridad de Defender para IoT para realizar cambios.

## <a name="configuration-objects"></a>Objetos de configuración

Las propiedades relacionadas con cada agente de seguridad de Defender para IoT se encuentran en el objeto de configuración del agente, dentro de la sección de propiedades adecuada, del módulo **azureiotsecurity**.

Para modificar la configuración, cree y edite este objeto dentro de la identidad de módulo gemela de **azureiotsecurity**.

Si el objeto de configuración de agente no existe en el módulo gemelo de **azureiotsecurity**, todos los valores de propiedades del agente de seguridad se establecerán en los valores predeterminados correspondientes.

```json
"desired": {
  "ms_iotn:urn_azureiot_Security_SecurityAgentConfiguration": {
  }
}
```

## <a name="configuration-schema-and-validation"></a>Esquema de configuración y validación

Asegúrese de validar la configuración del agente con este [esquema](https://aka.ms/iot-security-github-module-schema). Un agente no se iniciará si el objeto de configuración no coincide con el esquema.

Si, mientras el agente se ejecuta, el objeto de configuración se cambia a una configuración no válida (la configuración no coincide con el esquema), el agente pasará por alto la configuración no válida y continuará con la configuración actual.

### <a name="configuration-validation"></a>Validación de la configuración

El agente de seguridad de Defender para IoT notifica su configuración actual dentro de la sección de propiedades notificadas de la identidad del módulo gemelo **azureiotsecurity**.
El agente informa de todas las propiedades disponibles; si el usuario no ha establecido ninguna propiedad, el agente informa de la configuración predeterminada.

Para validar la configuración, compare los valores establecidos en la sección deseada con los valores que se indican en la sección de informes.

Si no coinciden las propiedades deseadas con las notificadas, el agente no pudo analizar la configuración.

Valide las propiedades deseadas según el [esquema](https://aka.ms/iot-security-github-module-schema), corrija los errores y vuelva a establecer las propiedades que prefiera.

> [!NOTE]
> Se activará una alerta de error de configuración desde el agente en caso de que este último no pueda analizar la configuración deseada.
> Compare la sección notificada con la deseada para saber si la alerta se sigue aplicando.

## <a name="editing-a-property"></a>Editar una propiedad

Todas las propiedades personalizadas se deben establecer dentro del objeto de configuración del agente, en el módulo gemelo de **azureiotsecurity**.
Para usar un valor de propiedad predeterminado, quite la propiedad del objeto de configuración.

### <a name="setting-a-property"></a>Configurar una propiedad

1. En su IoT Hub, busque y seleccione el dispositivo que quiera cambiar.

1. Haga clic en el dispositivo y, después, en el módulo **azureiotsecurity**.

1. Haga clic en **Identidad de módulo gemela**.

1. Edite las propiedades que desea cambiar en el módulo de seguridad.

   Por ejemplo, use la siguiente configuración para configurar eventos de conexión como de prioridad alta y recopilar eventos de prioridad alta cada 7 minutos.

    ```json
    "desired": {
        "ms_iotn:urn_azureiot_Security_SecurityAgentConfiguration": {
            "highPriorityMessageFrequency": {
                "value": "PT7M"
            },
            "eventPriorityConnectionCreate": {
                "value": "High"
            }
        }
    }
    ```

1. Haga clic en **Save**(Guardar).

### <a name="using-a-default-value"></a>Usar un valor predeterminado

Para usar un valor de propiedad predeterminado, quite la propiedad del objeto de configuración.

## <a name="default-properties"></a>Propiedades predeterminadas

La siguiente tabla contiene las propiedades controlables de los agentes de seguridad de Defender para IoT.

Los valores predeterminados están disponibles en el esquema pertinente en [GitHub](https\://aka.ms/iot-security-module-default).

| Nombre| Status | Valores válidos| Valores predeterminados| Descripción |
|----------|------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|---------------|
|highPriorityMessageFrequency|Requerido: false |Valores válidos:  Duración en formato ISO 8601 |Valor predeterminado: PT7M |Intervalo de tiempo máximo antes de que los mensajes de prioridad alta se envíen.|
|lowPriorityMessageFrequency |Requerido: false|Valores válidos:  Duración en formato ISO 8601 |Valor predeterminado: PT5H |Tiempo máximo antes de que los mensajes de prioridad baja se envíen.|
|snapshotFrequency |Requerido: false|Valores válidos:  Duración en formato ISO 8601 |Valor predeterminado: PT13H |Intervalo de tiempo para la creación de instantáneas de estado del dispositivo.|
|maxLocalCacheSizeInBytes |Requerido: false |Valores válidos: |Valor predeterminado: 2560000, mayor que 8192 | Almacenamiento máximo (en bytes) permitido en la memoria caché de mensajes de un agente. Cantidad máxima de espacio permitido para almacenar mensajes en el dispositivo, antes de que se envíen mensajes.|
|maxMessageSizeInBytes |Requerido: false |Valores válidos:  Un número positivo, mayor que 8192 y menor que 262144. |Valor predeterminado: 204800 |Máximo tamaño permitido de un mensaje del agente a la nube. Esta configuración controla la cantidad de datos máxima que se envía en cada mensaje. |
|eventPriority${EventName} |Requerido: false |Valores válidos:  Alto, Bajo, Desactivado |Valores predeterminados: |Prioridad de cada evento generado por el agente. |

### <a name="supported-security-events"></a>Eventos de seguridad admitidos

|Nombre del evento| PropertyName | Valor predeterminado| Evento de instantánea| Estado de los detalles  |
|----------|------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|---------------|
|Evento de diagnóstico|eventPriorityDiagnostic| Off| False| Eventos de diagnóstico relacionados con el agente. Use este evento para el registro detallado.|
|Error de configuración |eventPriorityConfigurationError |Bajo |False |El agente no pudo analizar la configuración. Contraste la configuración con el esquema.|
|Estadísticas de eventos quitados |eventPriorityDroppedEventsStatistics |Bajo |True|Estadísticas de eventos relacionados con el agente. |
|Hardware conectado|eventPriorityConnectedHardware |Bajo |True |Instantánea de todo el hardware conectado al dispositivo.|
|Puertos de escucha|eventPriorityListeningPorts |Alto |True |Instantánea de todos los puertos de escucha en el dispositivo.|
|Crear proceso |eventPriorityProcessCreate |Bajo |False |Auditoría de la creación de procesos en el dispositivo.|
|Finalizar proceso|eventPriorityProcessTerminate |Bajo |False |Auditoría de la finalización de procesos en el dispositivo.|
|Información del sistema |eventPrioritySystemInformation |Bajo |True |Instantánea de la información del sistema (por ejemplo: SO o CPU).|
|Usuarios locales| eventPriorityLocalUsers |Alto |True|Instantánea de los usuarios locales registrados en el sistema. |
|Inicio de sesión|  eventPriorityLogin |Alto|False|Auditoría de los eventos de inicio de sesión del dispositivo (inicios de sesión tanto locales como remotos).|
|Crear conexión |eventPriorityConnectionCreate|Bajo|False|Auditoría de las conexiones TCP creadas desde y hacia el dispositivo. |
|Configuración de firewall| eventPriorityFirewallConfiguration|Bajo|True|Instantánea de configuración de firewall del dispositivo (reglas de firewall). |
|Línea base del SO| eventPriorityOSBaseline| Bajo|True|Instantánea de la comprobación de línea base del SO.|
|

## <a name="next-steps"></a>Pasos siguientes

- [Descripción de las recomendaciones de Defender para IoT](concept-recommendations.md)
- [Exploración de las alertas de Defender para IoT](concept-security-alerts.md)
- [Acceso a datos de seguridad sin procesar](how-to-security-data-access.md)
