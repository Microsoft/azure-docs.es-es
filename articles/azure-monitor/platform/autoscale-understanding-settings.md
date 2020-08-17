---
title: Información sobre la configuración de escalado automático en Azure Monitor
description: Un análisis detallado de la configuración de escalado automático y de cómo funciona. Se aplica a Virtual Machines, Cloud Services, Web Apps
ms.topic: conceptual
ms.date: 12/18/2017
ms.subservice: autoscale
ms.openlocfilehash: 6d6b868f745803263339e6b27e2610aaca8f63fb
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87317474"
---
# <a name="understand-autoscale-settings"></a>Información acerca de la configuración de escalado automático
La configuración de escalado automático le ayuda a asegurarse de que tiene la cantidad adecuada de recursos en ejecución para administrar las fluctuaciones de carga de la aplicación. Puede configurar los valores de escalado automático para que se desencadene en función de métricas que indican carga o rendimiento, o para que se desencadene en una fecha y hora programadas. En este artículo se proporciona una visión detallada de la estructura de una configuración de escalado automático. El artículo comienza con el esquema y las propiedades de una configuración y luego le guía por los diferentes tipos de perfil que se pueden configurar. Por último, en el artículo se describe cómo la característica de escalado automático de Azure evalúa qué perfil ejecutar en un momento dado.

## <a name="autoscale-setting-schema"></a>Esquema de la configuración de escalado automático
Para ilustrar este esquema, se utiliza la siguiente configuración de escalado automático. Es importante tener en cuenta que esta configuración de escalado automático tiene:
- Un perfil. 
- Dos reglas de métricas en este perfil, una para el escalado horizontal y otra para la reducción horizontal.
  - La regla de escalado horizontal se desencadena cuando la métrica de porcentaje de CPU promedio del conjunto de escalado de máquinas virtuales es superior al 85 % durante los últimos 10 minutos.
  - La regla de reducción horizontal se desencadena cuando el promedio del conjunto de escalado de máquinas virtuales es inferior al 60 % durante el último minuto.

> [!NOTE]
> Una configuración puede tener varios perfiles. Para más información, consulte la sección de [perfiles](#autoscale-profiles). Un perfil también puede tener definidas varias reglas de escalado y reducción horizontal. Para ver cómo se evalúan, consulte la sección de [evaluación](#autoscale-evaluation).

```JSON
{
  "id": "/subscriptions/s1/resourceGroups/rg1/providers/microsoft.insights/autoscalesettings/setting1",
  "name": "setting1",
  "type": "Microsoft.Insights/autoscaleSettings",
  "location": "East US",
  "properties": {
    "enabled": true,
    "targetResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/vmss1",
    "profiles": [
      {
        "name": "mainProfile",
        "capacity": {
          "minimum": "1",
          "maximum": "4",
          "default": "1"
        },
        "rules": [
          {
            "metricTrigger": {
              "metricName": "Percentage CPU",
              "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/vmss1",
              "timeGrain": "PT1M",
              "statistic": "Average",
              "timeWindow": "PT10M",
              "timeAggregation": "Average",
              "operator": "GreaterThan",
              "threshold": 85
            },
            "scaleAction": {
              "direction": "Increase",
              "type": "ChangeCount",
              "value": "1",
              "cooldown": "PT5M"
            }
          },
    {
            "metricTrigger": {
              "metricName": "Percentage CPU",
              "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/vmss1",
              "timeGrain": "PT1M",
              "statistic": "Average",
              "timeWindow": "PT10M",
              "timeAggregation": "Average",
              "operator": "LessThan",
              "threshold": 60
            },
            "scaleAction": {
              "direction": "Decrease",
              "type": "ChangeCount",
              "value": "1",
              "cooldown": "PT5M"
            }
          }
        ]
      }
    ]
  }
}
```

| Sección | Nombre del elemento | Descripción |
| --- | --- | --- |
| Configuración | id | Identificador de recurso de la configuración de escalado automático. La configuración de escalado automático es un recurso de Azure Resource Manager. |
| Configuración | name | Nombre de la configuración de escalado automático. |
| Configuración | ubicación | Ubicación de la configuración de escalado automático. Esta ubicación puede ser diferente de la ubicación de los recursos que se van a escalar. |
| properties | targetResourceUri | Identificador de recurso del recurso que se va a escalar. Solo puede tener una configuración de escalado automático por recurso. |
| properties | profiles | Una configuración de escalado automático se compone de uno o varios perfiles. Cada vez que se ejecuta el motor de escalado automático, ejecuta un perfil. |
| perfile | name | Nombre del perfil. Puede elegir cualquier nombre que le ayude a identificar el perfil. |
| perfile | capacity.maximum | La capacidad máxima permitida. Garantiza que, al ejecutar este perfil, el escalado automático no escalará el recurso por encima de este número. |
| perfile | Capacity.minimum | La capacidad mínima permitida. Garantiza que, al ejecutar este perfil, el escalado automático no escalará el recurso por debajo de este número. |
| perfile | Capacity.default | Si hay algún problema al leer la métrica del recurso (en este caso, la CPU de "vmss1") y la capacidad actual es inferior a la predeterminada, el escalado automático escalará horizontalmente al valor predeterminado. De esta forma, se garantiza la disponibilidad del recurso. Si la capacidad actual ya es mayor que la predeterminada, el escalado automático no reduce horizontalmente. |
| perfile | reglas | El escalado automático permite escalar automáticamente entre las capacidades máxima y mínima mediante las reglas del perfil. Puede tener varias reglas en un perfil. Normalmente hay dos reglas: una para determinar cuándo escalar horizontalmente y la otra para determinar cuándo reducir horizontalmente. |
| rule | metricTrigger | Define la condición de métrica de la regla. |
| metricTrigger | metricName | El nombre de la métrica. |
| metricTrigger |  metricResourceUri | El identificador de recurso del recurso que emite esta métrica. En la mayoría de los casos, es el mismo que el recurso que se va a escalar. En algunos casos, puede ser diferente. Por ejemplo, puede escalar un conjunto de escalado de máquinas virtuales en función del número de mensajes en una cola de almacenamiento. |
| metricTrigger | timeGrain | La duración del muestreo de métricas. Por ejemplo, **TimeGrain = "PT1M"** significa que las métricas se deberían agregar cada minuto mediante el método de agregación especificado en "statistic". |
| metricTrigger | statistic | El método de agregación del período timeGrain. Por ejemplo, **statistic = "Average"** y **timeGrain = "PT1M"** significa que las métricas se deberían agregar cada minuto tomando la media. Esta propiedad determina cómo se muestrea la métrica. |
| metricTrigger | timeWindow | La cantidad de tiempo necesario para recuperar las métricas. Por ejemplo, **timeWindow = "PT10M"** significa que, cada vez que se ejecuta el escalado automático, se consultan las métricas de los últimos 10 minutos. La ventana de tiempo permite que las métricas se normalicen y evita que reaccionen a picos transitorios. |
| metricTrigger | timeAggregation | Método de agregación que se usa para agregar métricas muestreadas. Por ejemplo, **TimeAggregation = "Average"** agregará las métricas muestreadas teniendo en cuenta la media. En el caso anterior, toma las diez muestras de 1 minuto y hace la media. |
| rule | scaleAction | La acción que se realizará cuando se desencadene la propiedad metricTrigger de la regla. |
| scaleAction | direction | "Aumentar" para escalar horizontalmente o "Reducir" para reducir horizontalmente.|
| scaleAction | value | El grado de aumento o reducción de la capacidad del recurso. |
| scaleAction | cooldown | La cantidad de tiempo que debe transcurrir después de realizar una operación de escalado antes de poder iniciar otra. Por ejemplo, si **cooldown = "PT10M"** , el escalado automático no intenta escalar de nuevo durante otros 10 minutos. Cooldown permite que las métricas se estabilicen después de la adición o eliminación de instancias. |

## <a name="autoscale-profiles"></a>Perfiles de escalado automático

Hay tres tipos de perfiles de escalado automático:

- **Perfil normal:** el perfil más común. Si no es necesario escalar el recurso en función del día de la semana, o de un día determinado, puede usar un perfil normal. Luego, este perfil se puede configurar con reglas de métricas que determinen cuándo realizar un escalado horizontal y cuándo una reducción horizontal. Solo debe definir un perfil normal.

    El perfil de ejemplo que se usó anteriormente en este artículo es un ejemplo de perfil normal. También es posible establecer un perfil para escalar a un número de instancias estáticas para el recurso.

- **Perfil de fecha fija:** este perfil es para casos especiales. Por ejemplo, supongamos que tiene previsto un evento importante el 26 de diciembre de 2018 (PST). Quiere que las capacidades mínima y máxima del recurso sean diferentes ese día, pero seguir escalando sobre las mismas métricas. En este caso, debe agregar un perfil de fecha fija a la lista de perfiles de la configuración. El perfil se configura solo para que se ejecute en el día del evento. Para cualquier otro día, el escalado automático usa el perfil normal.

    ``` JSON
    "profiles": [{
    "name": " regularProfile",
    "capacity": {
    ...
    },
    "rules": [{
    ...
    },
    {
    ...
    }]
    },
    {
    "name": "eventProfile",
    "capacity": {
    ...
    },
    "rules": [{
    ...
    }, {
    ...
    }],
    "fixedDate": {
        "timeZone": "Pacific Standard Time",
               "start": "2017-12-26T00:00:00",
               "end": "2017-12-26T23:59:00"
    }}
    ]
    ```
    
- **Perfil de periodicidad:** este tipo de perfil le permite garantizar que este perfil se usará siempre un día concreto de la semana. Los perfiles de periodicidad solo tienen una hora de inicio. Se ejecutan hasta que el siguiente perfil de periodicidad o perfil de fecha fija está configurado para iniciarse. Una configuración de escalado automático que solo incluya un perfil de periodicidad ejecuta ese perfil incluso aunque haya un perfil normal definido en la misma configuración. En los dos ejemplos siguientes se muestra cómo se usa este perfil:

    **Ejemplo 1: Días laborables frente a fines de semana**
    
    Supongamos que quiere que los fines de semana su capacidad máxima sea 4. Como en los días laborables la carga esperada es mayor, quiere que su capacidad máxima sea 10. En este caso, la configuración contendría dos perfiles de periodicidad, uno para ejecutarse durante los fines de semana y el otro los días laborables.
    La configuración es parecida a esta:

    ``` JSON
    "profiles": [
    {
    "name": "weekdayProfile",
    "capacity": {
        ...
    },
    "rules": [{
        ...
    }],
    "recurrence": {
        "frequency": "Week",
        "schedule": {
            "timeZone": "Pacific Standard Time",
            "days": [
                "Monday"
            ],
            "hours": [
                0
            ],
            "minutes": [
                0
            ]
        }
    }}
    },
    {
    "name": "weekendProfile",
    "capacity": {
        ...
    },
    "rules": [{
        ...
    }]
    "recurrence": {
        "frequency": "Week",
        "schedule": {
            "timeZone": "Pacific Standard Time",
            "days": [
                "Saturday"
            ],
            "hours": [
                0
            ],
            "minutes": [
                0
            ]
        }
    }
    }]
    ```

    La configuración anterior muestra que cada perfil de periodicidad tiene una programación. Esta programación determina cuándo comienza a ejecutarse el perfil. El perfil se detiene cuando llega la hora de ejecutar otro perfil.

    Por ejemplo, en la configuración anterior, "weekdayProfile" está configurado para iniciarse los lunes a las 12:00 a.m. Eso significa que este perfil comienza a ejecutarse los lunes a las 12:00 a.m. Y continúa hasta el sábado a las 12:00 a.m., cuando está programado que comience a ejecutarse "weekendProfile".

    **Ejemplo 2: Horas laborables**
    
    Supongamos que tiene un umbral de métrica durante las horas laborables (de 9:00 a.m. a 5:00 p.m.) y otro para todas las demás horas. La configuración sería parecida a esta:
    
    ``` JSON
    "profiles": [
    {
    "name": "businessHoursProfile",
    "capacity": {
        ...
    },
    "rules": [{
        ...
    }],
    "recurrence": {
        "frequency": "Week",
        "schedule": {
            "timeZone": "Pacific Standard Time",
            "days": [
                "Monday", “Tuesday”, “Wednesday”, “Thursday”, “Friday”
            ],
            "hours": [
                9
            ],
            "minutes": [
                0
            ]
        }
    }
    },
    {
    "name": "nonBusinessHoursProfile",
    "capacity": {
        ...
    },
    "rules": [{
        ...
    }]
    "recurrence": {
        "frequency": "Week",
        "schedule": {
            "timeZone": "Pacific Standard Time",
            "days": [
                "Monday", “Tuesday”, “Wednesday”, “Thursday”, “Friday”
            ],
            "hours": [
                17
            ],
            "minutes": [
                0
            ]
        }
    }
    }]
    ```
    
    La configuración anterior muestra que "businessHoursProfile" comienza a ejecutarse el lunes a las 9:00 a.m. y continúa hasta las 5:00 p.m. En este momento, comienza a ejecutarse "nonBusinessHoursProfile". "nonBusinessHoursProfile" se ejecuta hasta las 9:00 a.m. del martes y después "businessHoursProfile" toma de nuevo el control. Este perfil se repite hasta el viernes a las 5:00 p.m. En ese momento, "nonBusinessHoursProfile" se ejecuta todo el tiempo hasta el lunes a las 9:00 a.m.
    
> [!Note]
> La interfaz de usuario de escalado automático de Azure Portal exige horas de finalización para los perfiles de periodicidad y empieza a ejecutar el perfil predeterminado de la configuración de escalado automático entre los perfiles de periodicidad.
    
## <a name="autoscale-evaluation"></a>Evaluación del escalado automático
Dado que la configuración de escalado automático puede incluir varios perfiles, y que cada perfil puede incluir varias reglas de métrica, es importante comprender cómo se evalúa una configuración de escalado automático. Cada vez que se ejecuta el trabajo de escalado automático, se empieza por elegir el perfil que se aplica. A continuación, el escalado automático evalúa los valores mínimo y máximo y las reglas de métrica del perfil, y decide si es necesario realizar una acción de escalado.

### <a name="which-profile-will-autoscale-pick"></a>¿Qué perfil seleccionará el escalado automático?

El escalado automático usa la siguiente secuencia para elegir el perfil:
1. En primer lugar, busca cualquier perfil de fecha fija que esté configurado para ejecutarse ahora. Si existe, el escalado automático lo ejecuta. Si hay varios perfiles de fecha fija que se supone que se pueden ejecutar, el escalado automático selecciona el primero de ellos.
2. Si no hay ningún perfil de fecha fija, el escalado automático busca perfiles de periodicidad. Si los encuentra, los ejecuta.
3. Si no hay perfiles de fecha fija ni de periodicidad, el escalado automático ejecuta el perfil normal.

### <a name="how-does-autoscale-evaluate-multiple-rules"></a>¿Cómo evalúa el escalado automático varias reglas?

Después de que el escalado automático determina qué perfil ejecutar, evalúa todas las reglas de escalado horizontal del perfil (son reglas con la **dirección = "Aumentar"** ).

Si se desencadenan una o varias reglas de escalado horizontal, el escalado automático calcula la nueva capacidad según la propiedad **scaleAction** de cada una de esas reglas. A continuación, realiza el escalado horizontal hasta alcanzar el máximo de esas capacidades para garantizar la disponibilidad del servicio.

Por ejemplo, supongamos que hay un conjunto de escalado de máquinas virtuales con una capacidad actual de 10. Hay dos reglas de escalado horizontal: una que aumenta la capacidad en un 10 por ciento y otra que la aumenta en 3 recuentos. La primera regla daría como resultado una nueva capacidad de 11 y la segunda regla generaría una capacidad de 13. Para garantizar la disponibilidad del servicio, el escalado automático elige la acción que tenga como resultado la capacidad máxima, por lo que, en este caso, se elige la segunda regla.

Si no se desencadena ninguna regla de escalado horizontal, el escalado automático evalúa todas las reglas de reducción horizontal (reglas con la **dirección = "Reducir"** ). El escalado automático solo realizará una acción de reducción horizontal si se desencadenan todas las reglas de reducción horizontal.

El escalado automático calcula la nueva capacidad en función de la propiedad **scaleAction** de cada una de esas reglas. A continuación, elige la acción de escalado que da como resultado el valor máximo de esas capacidades para garantizar la disponibilidad del servicio.

Por ejemplo, supongamos que hay un conjunto de escalado de máquinas virtuales con una capacidad actual de 10. Hay dos reglas de reducción horizontal: una que reduce la capacidad en un 50 por ciento y otra que la reduce en 3 recuentos. La primera regla daría como resultado una nueva capacidad de 5 y la segunda regla generaría una capacidad de 7. Para garantizar la disponibilidad del servicio, el escalado automático elige la acción que tenga como resultado la capacidad máxima, por lo que, en este caso, se elige la segunda regla.

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre el escalado automático, consulte los siguientes recursos:

* [Introducción a los registros de escalado automático](./autoscale-overview.md)
* [Métricas comunes de escalado automático de Azure Monitor](./autoscale-common-metrics.md)
* [Procedimientos recomendados de escalado automático en Azure Monitor](./autoscale-best-practices.md)
* [Uso de acciones de escalado automático para enviar notificaciones de alerta por correo electrónico y Webhook en Azure Insights](./autoscale-webhook-email.md)
* [API de REST de escalado automático](/rest/api/monitor/autoscalesettings)

