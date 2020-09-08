---
title: 'Implementación de módulos a escala mediante Azure Portal: Azure IoT Edge'
description: Uso de Azure Portal para crear implementaciones automáticas para grupos de dispositivos IoT Edge
keywords: ''
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 4/21/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 0c1d83c2dac0163cd9b9cbc07969103381e85471
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855391"
---
# <a name="deploy-iot-edge-modules-at-scale-using-the-azure-portal"></a>Implementación de módulos de IoT Edge a escala mediante Azure Portal

Cree una **implementación automática de IoT Edge** en Azure Portal para administrar las implementaciones en curso de muchos dispositivos a la vez. Las implementaciones automáticas de IoT Edge forman parte de la característica de [Administración de dispositivos automática](/azure/iot-hub/iot-hub-automatic-device-management) de IoT Hub. Las implementaciones son procesos dinámicos que permiten implementar varios módulos en múltiples dispositivos, realizar un seguimiento del estado y del mantenimiento de los módulos, y realizar cambios cuando sea necesario.

Para más información, consulte el artículo [Descripción de las implementaciones automáticas de IoT Edge en un único dispositivo o a escala](module-deployment-monitoring.md).

## <a name="identify-devices-using-tags"></a>Identificación de dispositivos mediante etiquetas

Antes de crear una implementación, tendrá que especificar los dispositivos que desea afectar. Azure IoT Edge identifica los dispositivos mediante **etiquetas** en el dispositivo gemelo. Cada dispositivo puede tener varias etiquetas que puede definir de cualquier manera que tenga sentido para su solución.

Por ejemplo, si administra un recinto de edificios inteligentes, puede agregar etiquetas de ubicación, tipo de sala y entorno a un dispositivo:

```json
"tags":{
  "location":{
    "building": "20",
    "floor": "2"
  },
  "roomtype": "conference",
  "environment": "prod"
}
```

Para más información sobre dispositivos gemelos y etiquetas, consulte [Información y uso de dispositivos gemelos en IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md).

## <a name="create-a-deployment"></a>de una implementación

IoT Edge proporciona dos tipos diferentes de implementaciones automáticas que pueden usarse para personalizar el escenario. Puede crear una *implementación estándar*, que incluye los módulos del entorno de ejecución del sistema y cualquier módulo y ruta adicionales. Cada dispositivo solo puede aplicar una implementación. También, puede crear una *implementación superpuesta* que solo incluye módulos y rutas personalizados, no el entorno de ejecución del sistema. Se pueden combinar muchas implementaciones superpuestas en un dispositivo sobre una implementación estándar. Para más información sobre cómo funcionan conjuntamente los dos tipos de implementaciones automáticas, consulte [Descripción de las implementaciones automáticas de IoT Edge en un único dispositivo o a escala](module-deployment-monitoring.md).

Los pasos para crear una implementación y una implementación superpuesta son muy parecidos. Las diferencias se indican en los pasos siguientes.

1. En [Azure Portal](https://portal.azure.com), vaya a su instancia de IoT Hub.
1. En el menú del panel izquierdo, en **Administración automática de dispositivos**, seleccione **IoT Edge**.
1. En la barra superior, seleccione **Crear implementación** o **Crear la implementación superpuesta**.

Hay cinco pasos para crear una implementación. En las siguientes secciones se abordan cada uno de ellos.

### <a name="step-1-name-and-label"></a>Paso 1: Nombre y etiqueta

1. Asigne a su implementación un nombre exclusivo de hasta 128 letras en minúscula. Evite los espacios y los siguientes caracteres no válidos: `& ^ [ ] { } \ | " < > /`.
1. Puede agregar etiquetas como pares clave-valor para ayudarle a realizar un seguimiento de las implementaciones. Por ejemplo, **HostPlatform** y **Linux**, o **Versión** y **3.0.1**.
1. Seleccione **Siguiente: Módulos** para avanzar al paso dos.

### <a name="step-2-modules"></a>Paso 2: Módulos

Puede agregar hasta 50 módulos a una implementación. Si crea una implementación sin módulos, se quitan todos los módulos actuales de los dispositivos de destino.

En las implementaciones, puede administrar la configuración de los módulos de agente y centro de IoT Edge. Seleccione **Configuración del entorno de ejecución** para configurar los dos módulos del entorno de ejecución. En la implementación superpuesta, los módulos del entorno de ejecución no se incluyen, por lo que no se pueden configurar.

Puede agregar tres tipos de módulos:

* Módulo IoT Edge
* Módulo de Marketplace
* Módulo de Azure Stream Analytics

#### <a name="add-an-iot-edge-module"></a>Adición de un módulo de IoT Edge

Para agregar código personalizado como un módulo, o para agregar manualmente un módulo de servicio de Azure, siga estos pasos:

1. En la sección **Credenciales de Container Registry** de la página, proporcione los nombres y las credenciales de cualquier registro de contenedor privado que contenga las imágenes de módulo de esta implementación. El agente de IoT Edge notificará el error 500 si no puede encontrar las credenciales del registro del contenedor para una imagen de Docker.
1. En la sección **Módulos de IoT Edge** de la página, haga clic en **Agregar**.
1. Seleccione **Módulo de IoT Edge** en el menú desplegable.
1. Asigne al módulo un **nombre de módulo de IoT Edge**.
1. En el campo **URI de imagen**, introduzca la imagen de contenedor para el módulo.
1. Use el menú desplegable para seleccionar una **directiva de reinicio**. Elija entre las siguientes opciones:
   * **Siempre**: el módulo siempre se reinicia si se cierra por cualquier razón.
   * **Nunca**: el módulo nunca se reinicia si se cierra por cualquier razón.
   * **En caso de error**: el módulo se reinicia si se bloquea, pero no si se cierra sin problemas.
   * **En estado incorrecto**: el módulo se reinicia si se bloquea o devuelve un estado incorrecto. Depende de cada módulo la implementación de la función de estado de mantenimiento.
1. Use el menú desplegable para seleccionar el **estado deseado** para el módulo. Elija entre las siguientes opciones:
   * **En ejecución**: esta es la opción predeterminada. El módulo volverá a ejecutarse inmediatamente después de la implementación.
   * **Detenido**: tras la implementación, el módulo permanecerá inactivo hasta que el usuario u otro módulo lo llamen después del inicio.
1. Especifique cualquier **opción de creación de contenedor** que deba pasarse al contenedor. Para más información, consulte [docker create](https://docs.docker.com/engine/reference/commandline/create/).
1. Seleccione **Configuración de módulos gemelos** si quiere agregar etiquetas u otras propiedades al módulo gemelo.
1. Escriba las **Variables de entorno** para este módulo. Las variables de entorno proporcionan información de configuración a un módulo.
1. Seleccione **Agregar** para agregar el módulo a la implementación.

#### <a name="add-a-module-from-the-marketplace"></a>Adición de un módulo desde Marketplace

Para agregar un módulo desde Azure Marketplace, siga estos pasos:

1. En la sección **Módulos de IoT Edge** de la página, haga clic en **Agregar**.
1. En el menú desplegable, seleccione **Módulo de Marketplace**.
1. Elija un módulo en la página **Marketplace de módulos IoT Edge**. El módulo que seleccione se configura automáticamente para su suscripción, grupo de recursos y dispositivo. Posteriormente, aparece en la lista de módulos de IoT Edge. Es posible que este módulo necesite configuración adicional. Para más información, consulte [Implementación de módulos desde Azure Marketplace](how-to-deploy-modules-portal.md#deploy-modules-from-azure-marketplace).

#### <a name="add-a-stream-analytics-module"></a>Adición de un módulo de Stream Analytics

Para agregar un módulo desde Azure Stream Analytics, siga estos pasos:

1. En la sección **Módulos de IoT Edge** de la página, haga clic en **Agregar**.
1. Seleccione **Módulo de Azure Stream Analytics** en el menú desplegable.
1. En el panel derecho, elija su valor de **Suscripción**.
1. Elija su **trabajo de IoT Edge**.
1. Seleccione **Guardar** para agregar el módulo a la implementación.

#### <a name="configure-module-settings"></a>Configuración del módulo

Después de agregar un módulo a una implementación, puede seleccionar su nombre para abrir la página **Actualizar módulo IoT Edge**. En esta página, puede editar la configuración del módulo, las variables de entorno, las opciones de creación y el módulo gemelo. Si agregó un módulo desde Marketplace, puede que ya tenga rellenados algunos de estos parámetros.

Si va a crear una implementación superpuesta, puede que esté configurando un módulo que existe en otras implementaciones que se dirigen a los mismos dispositivos. Para actualizar el módulo gemelo sin sobrescribir otras versiones, abra la pestaña **Configuración de módulos gemelos**. Cree un valor de **Propiedad del módulo gemelo** con un nombre único para la subsección de las propiedades deseadas del módulo gemelo, por ejemplo `properties.desired.settings`. Si define propiedades solo en el campo `properties.desired`, se sobrescribirán las propiedades deseadas del módulo definidas en las implementaciones de prioridad más baja.

![Establecimiento de la propiedad del módulo gemelo para la implementación superpuesta](./media/how-to-deploy-monitor/module-twin-property.png)

Para más información sobre la configuración de los módulos gemelos en implementaciones superpuestas, consulte [Implementación superpuesta](module-deployment-monitoring.md#layered-deployment).

Cuando tenga configurados todos los módulos de una implementación, seleccione **Siguiente: Rutas** para avanzar al paso tres.

### <a name="step-3-routes"></a>Paso 3: Rutas

Las rutas definen cómo los módulos se comunican entre sí dentro de una implementación. De forma predeterminada, el asistente proporciona una ruta llamada **upstream** y definida como **FROM /messages/\* INTO $upstream**, lo que significa que los mensajes generados por cualquiera de los módulos se enviarán a su instancia de IoT Hub.  

Agregue o actualice las rutas con información de [Declaración de rutas](module-composition.md#declare-routes) y, luego, seleccione **Siguiente** para continuar con la sección de revisión.

Seleccione **Siguiente: Métricas**.

### <a name="step-4-metrics"></a>Paso 4: Métricas

Las métricas proporcionan el número de resúmenes de los distintos estados que un dispositivo puede notificar como resultado de la aplicación del contenido de configuración.

1. Escriba un nombre para **Nombre de métrica**.

1. Escriba una consulta para **Criterios de las métricas**. La consulta se basa en las [propiedades notificadas](module-edgeagent-edgehub.md#edgehub-reported-properties) del módulo gemelo del centro de IoT Edge. La métrica representa el número de filas devueltas por la consulta.

   Por ejemplo:

   ```sql
   SELECT deviceId FROM devices
     WHERE properties.reported.lastDesiredStatus.code = 200
   ```

Seleccione **Siguiente: Dispositivos de destino**.

### <a name="step-5-target-devices"></a>Paso 5: Dispositivos de destino

Use la propiedad de etiquetas en los dispositivos para dirigirse a los dispositivos específicos que deberían recibir esta implementación.

Como varias implementaciones pueden tener como destino el mismo dispositivo, debe dar a cada implementación un número de prioridad. En caso de conflicto, gana la implementación con la prioridad más alta (los valores más altos indican prioridad más alta). Si dos implementaciones tienen el mismo número de prioridad, gana la que se creó más recientemente.

Si varias implementaciones tienen como destino el mismo dispositivo, solo se aplicará la que tenga la prioridad más alta. Si varias implementaciones superpuestas tienen como destino el mismo dispositivo, se aplican todas. Sin embargo, si alguna de las propiedades está duplicada (por ejemplo, hay dos rutas con el mismo nombre), la de la implementación superpuesta de prioridad más alta sobrescribe el resto.

Cualquier implementación superpuesta que tenga como destino un dispositivo debe tener una prioridad más alta que la implementación base para que se aplique.

1. Especifique un número entero positivo en el valor de **Prioridad** de la implementación.
1. Escriba una **condición de destino** para determinar qué dispositivos se dirigirán a esta implementación. La condición se basa en las etiquetas del dispositivo gemelo o en las propiedades notificadas del dispositivo gemelo y debe coincidir con el formato de expresión. Por ejemplo, `tags.environment='test'` o `properties.reported.devicemodel='4000x'`.

Seleccione **Siguiente: Revisar y crear** para avanzar al paso final.

### <a name="step-6-review-and-create"></a>Paso 6: Revisar y crear

Revise la información de implementación y seleccione **Crear**.

Para supervisar la implementación, consulte [Supervisión de las implementaciones de IoT Edge](how-to-monitor-iot-edge-deployments.md).

## <a name="modify-a-deployment"></a>Modificación de una implementación

Cuando se modifica una implementación, los cambios se replican inmediatamente a todos los dispositivos seleccionados. Puede modificar la configuración y las características siguientes de una implementación existente:

* Condiciones de destino
* Métricas personalizadas
* Etiquetas
* Etiquetas
* Propiedades deseadas

### <a name="modify-target-conditions-custom-metrics-and-labels"></a>Modificación de las condiciones de destino, las métricas personalizadas y las etiquetas

1. En el centro de IoT, seleccione **IoT Edge** en el menú del panel de la izquierda.
1. Seleccione la pestaña **Implementaciones de IoT Edge** y luego seleccione la implementación que quiere configurar.
1. Seleccione la pestaña **Condición de destino**. Cambie la **Condición de destino** para apuntar a los dispositivos previstos. También puede ajustar la **Prioridad**.  Seleccione **Guardar**.

    Si actualiza la condición de destino, se producen las siguientes actualizaciones:

    * Si un dispositivo no cumplía la antigua condición de destino, pero cumple la nueva condición de destino y esta implementación es la prioridad más alta para ese dispositivo, esta implementación se aplica al dispositivo.
    * Si un dispositivo que actualmente ejecuta esta implementación ya no cumple la condición de destino, desinstala esta implementación y asume la siguiente implementación de mayor prioridad.
    * Si un dispositivo que actualmente ejecuta esta implementación ya no cumple la condición de destino y no cumple la condición de destino de cualquier otra implementación, no se produce ningún cambio en el dispositivo. El dispositivo sigue ejecutando los módulos actuales en su estado actual, pero ya no se administra como parte de esta implementación. Cuando se cumple la condición de destino de cualquier otra implementación, desinstala esta implementación y adopta una nueva.

1. Seleccione la pestaña **Métricas** y haga clic en el botón **Editar métricas**. Para agregar o modificar las métricas personalizadas, use la sintaxis de ejemplo como guía. Seleccione **Guardar**.

    ![Edición de las métricas personalizadas de una implementación](./media/how-to-deploy-monitor/metric-list.png)

1. Seleccione la pestaña **Etiquetas**, haga los cambios que quiera y seleccione **Guardar**.

## <a name="delete-a-deployment"></a>Eliminación de una implementación

Cuando se elimina una implementación, los dispositivos implementados adoptan la siguiente implementación de prioridad más alta. Si los dispositivos no cumplen la condición de destino de alguna implementación, los módulos no se quitan cuando se elimina la implementación.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a su instancia de IoT Hub.
1. Seleccione **IoT Edge**.
1. Seleccione la pestaña **Implementaciones de IoT Edge**.

   ![Visualización de las implementaciones de IoT Edge](./media/how-to-deploy-monitor/iot-edge-deployments.png)

1. Utilice la casilla de verificación para seleccionar la implementación que desea eliminar.
1. Seleccione **Eliminar**.
1. Un mensaje le informará de que esta acción eliminará esta implementación y volverá al estado anterior para todos los dispositivos. Se aplicará una implementación con una prioridad más baja. Si ninguna otra implementación está dirigida, no se quitará ningún módulo. Si quiere quitar todos los módulos del dispositivo, cree una implementación con cero módulos e impleméntela a los mismos dispositivos. Seleccione **Yes** (Sí) para continuar.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la [implementación de módulos en dispositivos IoT Edge](module-deployment-monitoring.md).
