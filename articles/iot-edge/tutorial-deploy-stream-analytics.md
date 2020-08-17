---
title: 'Tutorial: Stream Analytics en el área perimetral con Azure IoT Edge'
description: En este tutorial, va a implementar Azure Stream Analytics como un módulo en un dispositivo de Iot Edge.
author: kgremban
ms.author: kgremban
ms.date: 07/29/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: c259e913a8ee5181bc58aea651af62324cf01fcb
ms.sourcegitcommit: 14bf4129a73de2b51a575c3a0a7a3b9c86387b2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87439414"
---
# <a name="tutorial-deploy-azure-stream-analytics-as-an-iot-edge-module"></a>Tutorial: Implementación de Azure Stream Analytics como un módulo de IoT Edge

Muchas soluciones de IoT usan servicios de análisis para obtener información sobre los datos a medida que llegan a la nube desde los dispositivos IoT. Con Azure IoT Edge, puede tomar la lógica de [Azure Stream Analytics](https://docs.microsoft.com/azure/stream-analytics/) y llevarla al propio dispositivo. Al procesar los flujos de telemetría en el perímetro, puede reducir la cantidad de datos cargados y reducir el tiempo que se tarda en reaccionar ante la información procesable.

Azure IoT Edge y Azure Stream Analytics se integran para simplificar el desarrollo de cargas de trabajo. Puede crear un trabajo de Azure Stream Analytics en Azure Portal y, después, implementarlo como un módulo de IoT Edge sin código adicional.  

Azure Stream Analytics proporciona una sintaxis de consulta con una estructura compleja para el análisis de datos, tanto en la nube como en los dispositivos IoT Edge. Para obtener más información, vea [Documentación de Azure Stream Analytics](../stream-analytics/stream-analytics-edge.md).

El módulo de Stream Analytics de este tutorial calcula la temperatura media en una ventana gradual de 30 segundos. Cuando dicho promedio alcanza el valor de 70, el módulo envía una alerta para que el dispositivo tome medidas. En este caso, esa acción es restablecer el sensor de temperatura simulado. En un entorno de producción, podría usar esta funcionalidad para apagar una máquina o tomar medidas preventivas cuando la temperatura alcance niveles peligrosos.

En este tutorial, aprenderá a:
> [!div class="checklist"]
>
> * Crear un trabajo de Azure Stream Analytics para procesar los datos en los dispositivos perimetrales.
> * Conectar el trabajo de Azure Stream Analytics nuevo con otros módulos de IoT Edge.
> * Implemente el trabajo de Azure Stream Analytics en un dispositivo IoT Edge desde Azure Portal.

<center>

![Diagrama: Tutorial sobre la arquitectura, las fases y la implementación de un trabajo de ASA](./media/tutorial-deploy-stream-analytics/asa-architecture.png)
</center>

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Un dispositivo de Azure IoT Edge:

* Puede usar una máquina de Azure como dispositivo de IoT Edge; para ello, siga los pasos que se indican en el artículo de inicio rápido para dispositivos [Linux](quickstart-linux.md) o [Windows](quickstart.md).

Recursos en la nube:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.

## <a name="create-an-azure-stream-analytics-job"></a>Creación de un trabajo de Azure Stream Analytics

En esta sección, creará un trabajo de Azure Stream Analytics que realizará estos pasos:

* Recepción de datos desde el dispositivo IoT Edge.
* Consulta de los datos de telemetría para identificar valores que estén fuera de un intervalo específico.
* Acciones en el dispositivo IoT Edge según los resultados de la consulta.

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Cuando se crea un trabajo de Azure Stream Analytics para ejecutarse en un dispositivo de IoT Edge, se debe almacenar de forma que se pueda llamar desde el dispositivo. Puede usar una cuenta de Azure Storage o crear una nueva ahora.

1. En Azure Portal, vaya a **Crear un recurso** > **Almacenamiento** > **Cuenta de almacenamiento**.

1. Especifique los siguientes valores para crear una cuenta de almacenamiento:

   | Campo | Value |
   | ----- | ----- |
   | Suscripción | Elija la misma suscripción que IoT Hub. |
   | Resource group | Se recomienda usar el mismo grupo de recursos para todos los recursos de prueba en las guías de inicio rápido y los tutoriales de IoT Edge. Por ejemplo, **IoTEdgeResources**. |
   | Nombre | Especifique un nombre único para la cuenta de almacenamiento. |
   | Location | Elija una ubicación cercana a usted. |

1. No modifique los valores predeterminados en el resto de los campos y seleccione **Revisar y crear**.

1. Revise la configuración y seleccione **Crear**.

### <a name="create-a-new-job"></a>Crear un trabajo

1. En Azure Portal, vaya a **Crear un recurso** > **Internet de las cosas** > **Trabajo de Stream Analytics**.

1. Especifique los siguientes valores para crear un trabajo:

   | Campo | Value |
   | ----- | ----- |
   | Nombre del trabajo | Especifique el nombre del trabajo. Por ejemplo, **IoTEdgeJob** |
   | Suscripción | Elija la misma suscripción que IoT Hub. |
   | Resource group | Se recomienda usar el mismo grupo de recursos para todos los recursos de prueba que se crean en las guías de inicio rápido y los tutoriales de IoT Edge. Por ejemplo, **IoTEdgeResources**. |
   | Location | Elija una ubicación cercana a usted. |
   | Entorno de hospedaje | Seleccionar **Edge**. |

1. Seleccione **Crear**.

### <a name="configure-your-job"></a>Configuración del trabajo

Una vez que se ha creado el trabajo de Stream Analytics en Azure Portal, puede configurarlo con una entrada, una salida y una consulta para ejecutarlo en los datos que lo atraviesan.

Mediante los tres elementos de entrada, salida y consulta, esta sección crea un trabajo que recibe los datos de temperatura del dispositivo IoT Edge. Analiza dichos datos en una ventana de 30 segundos con desplazamiento. Si la temperatura media en dicha ventana supera los 70 grados, se envía una alerta al dispositivo IoT Edge. En la siguiente sección, en la que implementa el trabajo, especificará exactamente de dónde proceden los datos y a dónde se dirigen.  

1. Vaya al trabajo de Stream Analytics en Azure Portal.

1. En **Topología de trabajo**, seleccione **Entradas** y, después, **Agregar entrada de flujo**.

   ![Azure Stream Analytics: adición de entrada](./media/tutorial-deploy-stream-analytics/asa-input.png)

1. Elija **Centro de Microsoft Edge** en la lista desplegable.

1. En el panel **Nueva entrada**, escriba **Temperatura** como alias de entrada.

1. Mantenga los valores predeterminados en los restantes campos y seleccione **Guardar**.

1. En **Topología de trabajo**, abra **Salidas** y seleccione **Agregar**.

   ![Azure Stream Analytics: adición de salida](./media/tutorial-deploy-stream-analytics/asa-output.png)

1. Elija **Centro de Microsoft Edge** en la lista desplegable.

1. En el panel **Nueva salida**, escriba **alerta** como alias de salida.

1. Mantenga los valores predeterminados en los restantes campos y seleccione **Guardar**.

1. En **Topología de trabajo**, seleccione **Consulta**.

1. Reemplace el texto predeterminado por la siguiente consulta. El código SQL envía un comando de restablecimiento a la salida de la alerta si la temperatura media del equipo en una ventana de 30 segundos alcanza los 70 grados. El comando de restablecimiento se ha programado previamente en el sensor como una acción que se puede realizar.

    ```sql
    SELECT  
        'reset' AS command
    INTO
       alert
    FROM
       temperature TIMESTAMP BY timeCreated
    GROUP BY TumblingWindow(second,30)
    HAVING Avg(machine.temperature) > 70
    ```

1. Seleccione **Guardar consulta**.

### <a name="configure-iot-edge-settings"></a>Configuración de los valores de IoT Edge

Para preparar el trabajo de Stream Analytics que se va a implementar en un dispositivo IoT Edge, es preciso asociar el trabajo con un contenedor en una cuenta de almacenamiento. Al importar un trabajo, la definición del mismo se exporta en el contenedor de almacenamiento.

1. En **Configurar**, seleccione **Configuración de la cuenta de almacenamiento** y, después, **Agregar cuenta de almacenamiento**.

   ![Azure Stream Analytics: agregar cuenta de almacenamiento](./media/tutorial-deploy-stream-analytics/add-storage-account.png)

1. En el menú desplegable, seleccione la **cuenta de almacenamiento** que ha creado al principio de este tutorial.

1. Para el campo **Contenedor**, seleccione **Crear nuevo** y proporcione un nombre para el contenedor de almacenamiento.

1. Seleccione **Guardar**.

## <a name="deploy-the-job"></a>Implementación del trabajo

Ya está listo para implementar el trabajo de Azure Stream Analytics en el dispositivo de IoT Edge.

En esta sección, se usa el asistente de **Establecer módulos** en Azure Portal para crear un *manifiesto de implementación*. Un manifiesto de implementación es un archivo JSON que describe todos los módulos que se implementarán en un dispositivo, los registros de contenedores que almacenan las imágenes del módulo, cómo se deben administrar los módulos y la forma en que los módulos pueden comunicarse entre sí. El dispositivo IoT Edge recupera su manifiesto de implementación de IoT Hub y, después, usa su información para implementar y configurar todos los módulos que tiene asignados.

En este tutorial se implementan dos módulos. La primera opción es **SimulatedTemperatureSensor**, un módulo que simula un sensor de temperatura y humedad. El segundo es su trabajo de Stream Analytics. El módulo del sensor proporciona el flujo de datos que se analizará en la consulta de trabajo.

1. En Azure Portal, vaya hasta el centro de IoT.

1. Vaya a **IoT Edge** y, después, abra la página de detalles de su dispositivo IoT Edge.

1. Seleccione **Set modules** (Establecer módulos).  

1. Si ha implementado anteriormente el módulo SimulatedTemperatureSensor en este dispositivo, puede que el campo se rellene automáticamente. Si no lo hace, siga estos pasos para agregar el módulo:

   1. Haga clic en **Add** (Agregar) y seleccione **IoT Edge Module** (Módulo de IoT Edge).
   1. En el nombre, escriba **SimulatedTemperatureSensor**.
   1. Para el URI de la imagen, escriba **mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0**.
   1. Deje los restantes valores sin cambios y seleccione **Add** (Agregar).

1. Agregue el trabajo de Edge de Azure Stream Analytics con los siguientes pasos:

   1. Haga clic en **Add** (Agregar) y seleccione **Azure Stream Analytics Module** (Módulo de Azure Stream Analytics).
   1. Seleccione la suscripción y el trabajo Edge de Azure Stream Analytics que creó.
   1. Seleccione **Guardar**.

   Después de guardar los cambios, los detalles del trabajo de Stream Analytics se publicarán en el contenedor de almacenamiento que ha creado.

1. Cuando se agregue el módulo de Stream Analytics a la lista de módulos, seleccione su nombre para ver cómo está estructurado y actualice su configuración en la página **Actualizar módulo IoT Edge**.

   La pestaña **Configuración del módulo** tiene el **URI de imagen** que apunta a una imagen estándar de Azure Stream Analytics. Esta imagen se usa para todos los módulos de Stream Analytics que se implementan en un dispositivo IoT Edge.

   En la pestaña **Configuración de módulos gemelos** se muestra el JSON que define la propiedad de Azure Stream Analytics (ASA) llamada **ASAJobInfo**. El valor de dicha propiedad apunta a la definición del trabajo en el contenedor de almacenamiento. Esta propiedad indica cómo se configura la imagen de Stream Analytics con la información específica de su trabajo.

   De forma predeterminada, el módulo de Stream Analytics usa el mismo nombre que el trabajo en el que se basa. Si lo prefiere, puede cambiar el nombre del módulo en esta página, pero no es necesario.

1. Seleccione **Update** (Actualizar) o **Cancel** (Cancelar).

1. Anote el nombre del módulo de Stream Analytics, ya que lo necesitará en el paso siguiente. Después, seleccione **Next (Siguiente): Rutas** para continuar.

1. En la pestaña **Rutas**, se define cómo se pasan los mensajes entre los módulos de IoT Hub. Los mensajes se construyen mediante pares de nombre-valor. Reemplace el par de nombre-valor `route` y `upstream` predeterminado por los que se muestran en la siguiente tabla (los siguientes pares de nombre-valor) y reemplace las instancias de _{moduleName}_ por el nombre de su módulo de Azure Stream Analytics.

    | Nombre | Value |
    | --- | --- |
    | `telemetryToCloud` | `FROM /messages/modules/SimulatedTemperatureSensor/* INTO $upstream` |
    | `alertsToCloud` | `FROM /messages/modules/{moduleName}/* INTO $upstream` |
    | `alertsToReset` | `FROM /messages/modules/{moduleName}/* INTO BrokeredEndpoint("/modules/SimulatedTemperatureSensor/inputs/control")` |
    | `telemetryToAsa` | `FROM /messages/modules/SimulatedTemperatureSensor/* INTO BrokeredEndpoint("/modules/{moduleName}/inputs/temperature")`|

    Las rutas que se declaran aquí definen el flujo de datos que atraviesa el dispositivo IoT Edge. Los datos de telemetría de SimulatedTemperatureSensor se envían a IoT Hub y a la entrada de **temperatura** que se ha configurado en el trabajo de Stream Analytics. Los mensajes de salida de **alerta** se envían a IoT Hub y al módulo SimulatedTemperatureSensor para desencadenar el comando de restablecimiento.

1. Seleccione **Siguiente: Revisar y crear**.

1. En la pestaña **Revisar y crear**, puede ver que la información que ha proporcionado en el asistente se ha convertido en un manifiesto de implementación en formato JSON. Cuando termine de revisar el manifiesto, seleccione **Crear**.

1. Volverá a la página de detalles del dispositivo. Seleccione **Refresh** (Actualizar).  

    Verá cómo se ejecuta el módulo de Stream Analytics, además del agente de IoT Edge y los módulos del centro de IoT Edge. Pueden transcurrir unos minutos hasta que la información llegue al dispositivo IoT Edge y se inicien los nuevos módulos. Si los módulos no se ejecutan de inmediato, siga actualizando la página.

    ![Módulo de ASA y SimulatedTemperatureSensor detectado por el dispositivo](./media/tutorial-deploy-stream-analytics/module-output2.png)

## <a name="view-data"></a>Visualización de datos

Ahora, puede ir a su dispositivo IoT Edge para comprobar la interacción entre el módulo de Azure Stream Analytics y el módulo SimulatedTemperatureSensor.

1. Compruebe que todos los módulos se ejecutan en Docker:

   ```cmd/sh
   iotedge list  
   ```

1. Vea todos los registros del sistema y datos de métrica. Use el nombre del módulo de Stream Analytics:

   ```cmd/sh
   iotedge logs -f {moduleName}  
   ```

1. Vea los registros del sensor para comprobar cómo afecta el comando de restablecimiento al módulo SimulatedTemperatureSensor:

   ```cmd/sh
   iotedge logs SimulatedTemperatureSensor
   ```

   Verá como la temperatura del equipo asciende de forma progresiva hasta que alcanza 70 grados durante 30 segundos. Después, el módulo de Stream Analytics desencadena un restablecimiento y la temperatura de la máquina vuelve a bajar a 21.

   ![Restablecimiento de la salida del comando en los registros del módulo](./media/tutorial-deploy-stream-analytics/docker_log.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si prevé seguir con el siguiente artículo recomendado, puede mantener los recursos y las configuraciones que ya ha creado y volverlos a utilizar. También puede seguir usando el mismo dispositivo de IoT Edge como dispositivo de prueba.

En caso contrario, para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que creó en este artículo.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha configurado un trabajo de Azure Stream Analytics para analizar datos desde un dispositivo IoT Edge. A continuación, ha cargado este módulo de Azure Stream Analytics en su dispositivo IoT Edge para procesar y reaccionar al aumento de temperatura localmente, así como para enviar el flujo de datos agregado a la nube. Para ver cómo Azure IoT Edge puede crear más soluciones para la empresa, siga con los otros tutoriales.

> [!div class="nextstepaction"]
> [Implementación de un modelo de Azure Machine Learning como módulo](tutorial-deploy-machine-learning.md)
