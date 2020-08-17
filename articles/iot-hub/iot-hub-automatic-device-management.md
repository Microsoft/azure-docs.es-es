---
title: Administración de dispositivos automática a escala con Azure IoT Hub | Microsoft Docs
description: Uso de configuraciones automáticas de Azure IoT Hub para administrar varios dispositivos o módulos IoT
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 12/13/2019
ms.author: robinsh
ms.custom:
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 6d3661128008c13e5d4d459f6f8e7925aa18a9a4
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322778"
---
# <a name="automatic-iot-device-and-module-management-using-the-azure-portal"></a>Administración automática de dispositivos y módulos IoT mediante Azure Portal

[!INCLUDE [iot-edge-how-to-deploy-monitor-selector](../../includes/iot-hub-auto-device-config-selector.md)]

La administración de dispositivos automática de Azure IoT Hub automatiza muchas de las tareas repetitivas y complejas de administración de grandes flotas de dispositivos. Con la administración de dispositivos automática, puede tener como destino un conjunto de dispositivos según sus propiedades, definir una configuración que se quiera y luego permitir que IoT Hub actualice los dispositivos cuando estén dentro del ámbito. Esta actualización se realiza mediante una _configuración automática de dispositivos_ o una _configuración automática de módulos_, que permite resumir la integridad y el cumplimiento, administrar combinaciones y conflictos e implementar configuraciones por fases.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

La administración automática de dispositivos funciona mediante la actualización de un conjunto de dispositivos o módulos gemelos con las propiedades deseadas y la elaboración de un informe de resumen basado en las propiedades notificadas de los gemelos.  Presenta una nueva clase y documento JSON, llamado *Configuración*, que tiene tres partes:

* La **condición de destino** define el ámbito de los dispositivos o módulos gemelos que se van a actualizar. La condición de destino se especifica como una consulta de las propiedades notificadas o de las etiquetas de los gemelos.

* El **contenido de destino** define las propiedades deseadas que se van a agregar o actualizar en los dispositivos o módulos gemelos de destino. El contenido incluye una ruta de acceso a la sección de las propiedades que desea cambiar.

* Las **métricas** definen el número de resúmenes de los distintos estados de configuración, como **Correcto**, **En curso** y **Error**. Las métricas personalizadas se especifican como consultas de las propiedades notificadas de los gemelos.  Las métricas del sistema son las métricas predeterminadas que miden el estado de actualización de los gemelos, como el número de gemelos de destino y el número de gemelos que se han actualizado correctamente.

Las configuraciones automáticas se ejecutan por primera vez poco después de crear la configuración y posteriormente en intervalos de cinco minutos. Las consultas de métricas se ejecutan cada vez que se ejecuta la configuración automática.

## <a name="implement-twins"></a>Implementación de gemelos

Las configuraciones automáticas de dispositivos requieren el uso de dispositivos gemelos para sincronizar el estado entre la nube y los dispositivos.  Para más información, consulte [Información y uso de dispositivos gemelos en IoT Hub](iot-hub-devguide-device-twins.md).

Las configuraciones automáticas de módulos requieren el uso de módulos gemelos para sincronizar el estado entre la nube y los módulos. Para más información, consulte [Uso de módulos gemelos en IoT Hub](iot-hub-devguide-module-twins.md).

## <a name="use-tags-to-target-twins"></a>Uso de etiquetas para identificar los gemelos

Antes de crear una configuración, debe especificar a qué dispositivos o módulos quiere que se aplique. Azure IoT Hub identifica dispositivos mediante etiquetas en el dispositivo gemelo e identifica módulos mediante etiquetas en el módulo gemelo. Cada dispositivo o módulo puede tener varias etiquetas, y puede definirlas de cualquier manera que tenga sentido para su solución. Por ejemplo, si administra dispositivos en distintas ubicaciones, agregue las siguientes etiquetas a un dispositivo gemelo:

```json
"tags": {
    "location": {
        "state": "Washington",
        "city": "Tacoma"
    }
},
```

## <a name="create-a-configuration"></a>Creación de una configuración

1. En [Azure Portal](https://portal.azure.com), vaya hasta su instancia de IoT Hub. 

2. Seleccione **Configuración de dispositivo IoT**.

3. Seleccione **Agregar configuración del dispositivo** o **Agregar configuración del módulo**.

   ![Agregar configuración del dispositivo o configuración del módulo](./media/iot-hub-automatic-device-management/create-automatic-configuration.png)

Hay cinco pasos para crear una configuración. En las siguientes secciones se abordan cada uno de ellos. 

### <a name="name-and-label"></a>Nombre y etiqueta

1. Asigne a su configuración un nombre exclusivo de hasta 128 letras en minúscula. Evite los espacios y los siguientes caracteres no válidos: `& ^ [ ] { } \ | " < > /`.

2. Agregue etiquetas para realizar un mejor seguimiento de las configuraciones. Las etiquetas son pares de **nombre** y **valor** que describen la configuración. Por ejemplo, `HostPlatform, Linux` o `Version, 3.0.1`.

3. Haga clic en **Siguiente** para ir al paso siguiente. 

### <a name="specify-settings"></a>Especificar la configuración

En esta sección se define el contenido que se debe establecer en los dispositivos o módulos gemelos de destino. Hay dos entradas para cada conjunto de configuración. La primera es la ruta de acceso del gemelo, que coincide con la ruta de acceso a la sección JSON dentro de las propiedades deseadas del gemelo que se van a configurar.  La segunda es el contenido JSON que se va a insertar en dicha sección. 

Por ejemplo, puede establecer la ruta de acceso del gemelo en `properties.desired.chiller-water` y, a continuación, proporcionar el siguiente contenido JSON: 

```json
{
  "temperature": 66,
  "pressure": 28
}
```

![Definir el contenido y la ruta de acceso del gemelo](./media/iot-hub-automatic-device-management/module-config-twin-settings.png)


Para definir opciones de configuración individuales también puede especificar la ruta de acceso completa del gemelo y proporcionar el valor sin corchetes. Por ejemplo, con la ruta de acceso del gemelo `properties.desired.chiller-water.temperature`, establezca el contenido en `66`. A continuación, cree un nuevo ajuste del gemelo para la propiedad pressure. 

Si dos o más configuraciones tienen como destino la misma ruta de acceso del gemelo, se aplicará el contenido de la configuración con la prioridad más alta (la prioridad se define en el paso 4).

Si desea quitar una propiedad existente, especifique el valor de la propiedad en `null`.

Puede añadir opciones de configuración adicionales si selecciona **Agregar configuración de dispositivo gemelo** o **Agregar configuración de módulo gemelo**.

### <a name="specify-metrics-optional"></a>Especificar las métricas (opcional)

Las métricas proporcionan el número de resúmenes de los distintos estados que un dispositivo o módulo puede notificar después de aplicar el contenido de configuración. Por ejemplo, puede crear una métrica para los cambios de configuración pendientes, una métrica para los errores y una métrica para los cambios de configuración correctos.

Cada configuración puede tener hasta cinco métricas personalizadas. 

1. Escriba un nombre para **Nombre de métrica**.

2. Escriba una consulta para **Criterios de las métricas**.  La consulta se basa en las propiedades notificadas de dispositivos gemelos.  La métrica representa el número de filas devueltas por la consulta.

Por ejemplo:

```sql
SELECT deviceId FROM devices 
  WHERE properties.reported.chillerWaterSettings.status='pending'
```

Puede incluir una cláusula que se haya aplicado a la configuración, como por ejemplo: 

```sql
/* Include the double brackets. */
SELECT deviceId FROM devices 
  WHERE configurations.[[yourconfigname]].status='Applied'
```

Si va a crear una métrica para informar sobre los módulos configurados, seleccione `moduleId` de `devices.modules`. Por ejemplo:

```sql
SELECT deviceId, moduleId FROM devices.modules
  WHERE properties.reported.lastDesiredStatus.code = 200
```

### <a name="target-devices"></a>Dispositivos de destino

Use la propiedad tags de los gemelos para dirigirse a los dispositivos o módulos específicos que deberían recibir esta configuración. También puede definir como destino las propiedades notificadas de los gemelos.

Las configuraciones automáticas de dispositivos solo pueden tener como destino etiquetas de dispositivos gemelos, y las configuraciones automáticas de módulos solo pueden tener como destino etiquetas de módulos gemelos. 

Dado que varias configuraciones pueden tener como destino el mismo dispositivo o el mismo módulo, debe dar a cada configuración un número de prioridad. En caso de conflicto, gana la configuración con la prioridad más alta. 

1. Especifique un número entero positivo en el valor de **Prioridad** de la configuración. El valor numérico más alto se considera la máxima prioridad. Si dos configuraciones tienen el mismo número de prioridad, gana la que se creó más recientemente. 

2. Escriba una **condición de destino** para determinar qué dispositivos o módulos se establecerán como destino de esta configuración. La condición se basa en las etiquetas de los gemelos o en las propiedades notificadas de los gemelos, y debe coincidir con el formato de la expresión. 

   Para la configuración automática de dispositivos, puede especificar solo la etiqueta o la propiedad notificada de destino. Por ejemplo, `tags.environment='test'` o `properties.reported.chillerProperties.model='4000x'`. Puede especificar `*` para dirigirse a todos los dispositivos. 
   
   Para la configuración automática de módulos, use una consulta para especificar etiquetas o propiedades notificadas de los módulos registrados en IoT Hub. Por ejemplo, `from devices.modules where tags.environment='test'` o `from devices.modules where properties.reported.chillerProperties.model='4000x'`. El carácter comodín no se puede usar para establecer como destino todos los módulos. 

3. Seleccione **Siguiente** para pasar al último paso.

### <a name="review-configuration"></a>Revisión de la configuración

Revise la información de configuración y seleccione **Enviar**.

## <a name="monitor-a-configuration"></a>Supervisión de una configuración

Para ver los detalles de una configuración y supervisar los dispositivos que la ejecutan, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), vaya hasta su instancia de IoT Hub. 

2. Seleccione **Configuración de dispositivo IoT**.

3. Examine la lista de configuración. Para cada configuración, puede ver los detalles siguientes:

   * **ID** (Identificador): nombre de la configuración.

   * **Condición de destino**: consulta que se utiliza para definir los dispositivos o módulos de destino.

   * **Priority** (Prioridad): el número de prioridad asignado a la configuración.

   * **Creation time** (Hora de creación): la marca de tiempo de cuando se creó la configuración. Esta marca de tiempo se utiliza para dirimir cuando dos configuraciones tienen la misma prioridad. 

   * **System metrics** (Métricas del sistema): las métricas calculadas por IoT Hub y que los desarrolladores no pueden personalizar. Los destinos especifican el número de dispositivos gemelos que coinciden con la condición de destino. Aplica el número especificado de dispositivos gemelos modificados por la configuración, que puede incluir modificaciones parciales en caso de que una configuración independiente de mayor prioridad también realice cambios. 

   * **Métricas personalizadas**: métricas especificadas por el desarrollador como consultas de las propiedades notificadas de los gemelos.  Se pueden definir hasta cinco métricas personalizadas por configuración. 
   
4. Seleccione la configuración que desea supervisar.  

5. Inspeccione los detalles de configuración. Puede utilizar las pestañas para ver detalles específicos sobre los dispositivos que han recibido la configuración.

   * **Condición de destino**: dispositivos o módulos que coinciden con la condición de destino. 

   * **Metrics** (Métricas): una lista de métricas del sistema y métricas personalizadas.  Para ver una lista de los dispositivos o módulos que se tienen en cuenta para cada métrica, seleccione la métrica en el menú desplegable y luego seleccione **Ver dispositivos** o **Ver módulos**.

   * **Configuración de dispositivos gemelos**: ajustes de los gemelos definidos en la configuración **.** 

   * **Configuration Labels** (Etiquetas de configuración): pares clave-valor utilizados para describir una configuración.  Las etiquetas no tienen ningún impacto en la funcionalidad. 

## <a name="modify-a-configuration"></a>Modificación de una configuración

Cuando se modifica una configuración, los cambios se replican inmediatamente a todos los dispositivos o módulos seleccionados. 

Si actualiza la condición de destino, se producen las siguientes actualizaciones:

* Si un gemelo no cumplía la condición de destino anterior, pero cumple la nueva condición de destino y esta configuración es la prioridad más alta para ese gemelo, se aplica esta configuración. 

* Si un gemelo que ejecuta actualmente esta configuración ya no cumple la condición de destino, los valores de la configuración se quitarán y el gemelo se modificará con la configuración de la siguiente prioridad más alta. 

* Si un gemelo que ejecuta actualmente esta configuración ya no cumple la condición de destino ni la condición de destino de cualquier otra configuración, los valores de la configuración se quitarán y no se realizará ningún otro cambio en el gemelo. 

Para modificar una configuración, siga estos pasos: 

1. En [Azure Portal](https://portal.azure.com), vaya hasta su instancia de IoT Hub. 

2. Seleccione **Configuración de dispositivo IoT**. 

3. Seleccione la configuración que desea modificar. 

4. Realice las actualizaciones en los campos siguientes: 

   * Condición de destino 
   * Etiquetas 
   * Priority 
   * Métricas

4. Seleccione **Guardar**.

5. Siga los pasos de [Supervisión de una configuración](#monitor-a-configuration) para observar la implementación de los cambios. 

## <a name="delete-a-configuration"></a>Eliminación de una configuración

Al eliminar una configuración, los dispositivos gemelos adoptan la configuración con la siguiente prioridad más alta. Si los dispositivos gemelos no cumplen la condición de destino de cualquier otra configuración, no se aplica ningún otro ajuste. 

1. En [Azure Portal](https://portal.azure.com), vaya hasta su instancia de IoT Hub. 

2. Seleccione **Configuración de dispositivo IoT**. 

3. Utilice la casilla de verificación para seleccionar la configuración que desea eliminar. 

4. Seleccione **Eliminar**.

5. Un símbolo del sistema le pedirá que confirme la operación.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido cómo configurar y supervisar dispositivos IoT a escala. Siga estos vínculos para más información sobre la administración de Azure IoT Hub:

* [Administración de las identidades de dispositivo de IoT Hub de forma masiva](iot-hub-bulk-identity-mgmt.md)
* [Métricas de IoT Hub](iot-hub-metrics.md)
* [Supervisión de operaciones](iot-hub-operations-monitoring.md)

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md)
* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)

Para explorar el uso del servicio IoT Hub Device Provisioning para habilitar el aprovisionamiento Just-In-Time sin intervención del usuario, vea: 

* [Servicio Azure IoT Hub Device Provisioning](/azure/iot-dps)
