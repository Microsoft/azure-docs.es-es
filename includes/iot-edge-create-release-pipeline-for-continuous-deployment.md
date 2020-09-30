---
author: v-tcassi
ms.service: iot-edge
ms.topic: include
ms.date: 08/26/2020
ms.author: v-tcassi
ms.openlocfilehash: 706b2306fbe9f2a744d2874a8b55f78fa2fc8e4d
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89301110"
---
## <a name="create-a-release-pipeline-for-continuous-deployment"></a>Creación de una canalización de versión para la implementación continua

En esta sección, se crea una canalización de versión que está configurada para ejecutarse automáticamente cuando la canalización de compilación coloca artefactos, y mostrará los registros de implementación en Azure Pipelines.

Creación de una nueva canalización y adición de una nueva fase:

1. En la pestaña **Versiones**, en **Canalizaciones**, elija **+ Nueva canalización**. O bien, si ya tiene canalizaciones de versión, elija el botón **+ Nuevo** y seleccione **+ Nueva canalización de versión**.  

    ![Adición de una canalización de compilación mediante el botón + Nueva canalización](./media/iot-edge-create-release-pipeline-for-continuous-deployment/add-release-pipeline.png)

2. Cuando se le pida que seleccione una plantilla, elija empezar con una **Fase vacía**.

    ![Comenzar con un trabajo vacío para la canalización de versión](./media/iot-edge-create-release-pipeline-for-continuous-deployment/start-with-empty-release-job.png)

3. La nueva canalización de versión se inicializa con una fase, llamada **Fase 1**. Cambie el nombre de la fase 1 a **dev** y trátela como una canalización de implementación continua para el entorno de desarrollo. Por lo general, las canalizaciones de implementación continua tienen varias fases, como **desarrollo**, **almacenamiento provisional** y **producción**. Puede usar distintos nombres y crear más según su práctica de DevOps. Cierre la ventana de detalles de la fase una vez que haya cambiado el nombre.

   También puede cambiar el nombre de la canalización de versión. Para hacerlo, seleccione el texto "Nueva canalización de versión" en la parte superior.

4. Vincule la versión con los artefactos de compilación que publica la canalización de compilación. Haga clic en **Agregar** en el área de artefactos.

   ![Haga clic en Agregar en el área de artefactos de la interfaz](./media/iot-edge-create-release-pipeline-for-continuous-deployment/add-artifacts.png)

5. En la página **Agregar un artefacto**, seleccione **Compilación** como **Tipo de origen**. Elija el proyecto y la canalización de compilación que ha creado. Si quiere, puede cambiar el **alias de origen** por algo más descriptivo. Después, seleccione **Agregar**.

   ![En la página Agregar un artefacto, seleccione Agregar para crearlo](./media/iot-edge-create-release-pipeline-for-continuous-deployment/add-artifact.png)

6. Abra los desencadenadores de artefacto y seleccione el botón de alternancia para habilitar el desencadenador de implementación continua. Ahora, se creará una nueva versión cada vez que haya disponible una nueva compilación.

   ![Abra los desencadenadores de artefacto y seleccione el botón de alternancia para habilitar el desencadenador de implementación continua.](./media/iot-edge-create-release-pipeline-for-continuous-deployment/add-trigger.png)

7. La fase **dev** está preconfigurada con un trabajo y cero tareas. En el menú de la canalización, seleccione **Tareas** y, a continuación, elija la fase **dev**. Seleccione el **Trabajo de agente** y cambie su **Nombre para mostrar** por **QA**. Puede configurar los detalles del trabajo del agente, pero la tarea de implementación no distingue entre plataformas, por lo que puede usar cualquier **Especificación del agente** del **Grupo de agentes** seleccionado.

   ![Ver las tareas de la fase de desarrollo de la pestaña Tareas](./media/iot-edge-create-release-pipeline-for-continuous-deployment/view-stage-tasks.png)

8. En el trabajo QA, seleccione el signo más ( **+** ) para agregar dos tareas. Busque y agregue **Azure IoT Edge** dos veces.

9. Seleccione la primera tarea de **Azure IoT Edge** y configúrela con los valores siguientes:

    | Parámetro | Descripción |
    | --- | --- |
    | Nombre para mostrar | El nombre para mostrar se actualiza automáticamente cuando cambia el campo Acción. |
    | Acción | Seleccione `Generate deployment manifest`. |
    | archivo .template.json | Especifique la ruta de acceso: `$(System.DefaultWorkingDirectory)/Drop/drop/deployment.template.json`. La ruta de acceso se publica desde la canalización de compilación. |
    | Plataforma predeterminada | Seleccione el sistema operativo adecuado para los módulos en función de su dispositivo IoT Edge de destino. |
    | Ruta de acceso de resultados| especifique la ruta de acceso `$(System.DefaultWorkingDirectory)/Drop/drop/configs/deployment.json`. Esta es la ruta de acceso del archivo de manifiesto de implementación de IoT Edge final. |

    Estas configuraciones ayudan a reemplazar las direcciones URL del módulo en el archivo `deployment.template.json`. **Generate deployment manifest** (Generar manifiesto de implementación) también ayuda a reemplazar las variables con el valor exacto que especificó en el archivo `deployment.template.json`. En VS o VS Code, el valor real se especifica en un archivo `.env`. En Azure Pipelines, el valor se establece en la pestaña **Variables de canalización de versión**. Vaya a la pestaña **Variables** y configure el nombre y valor de la siguiente manera:

    * **ACR_ADDRESS**: Valor del **servidor de inicio de sesión** de Azure Container Registry. Puede recuperar el servidor de inicio de sesión en la página de información general del registro de contenedor en Azure Portal.
    * **ACR_PASSWORD**: la contraseña de Azure Container Registry.
    * **ACR_USER**: el nombre de usuario de Azure Container Registry.

    Si tiene otras variables en el proyecto, puede especificar el nombre y el valor en esta pestaña. **Generate deployment manifest** (Generar manifiesto de implementación) solo puede reconocer las variables del tipo `${VARIABLE}`. Asegúrese de que está usando este tipo en los archivos `*.template.json`.

    ![Configuración de las variables para la canalización de versión en la pestaña Variables](./media/iot-edge-create-release-pipeline-for-continuous-deployment/configure-variables.png)

10. Seleccione la segunda tarea de **Azure IoT Edge** y configúrela con los valores siguientes:

    | Parámetro | Descripción |
    | --- | --- |
    | Nombre para mostrar | El nombre para mostrar se actualiza automáticamente cuando cambia el campo Acción. |
    | Acción | Seleccione `Deploy to IoT Edge devices`. |
    | Archivo de implementación | especifique la ruta de acceso `$(System.DefaultWorkingDirectory)/Drop/drop/configs/deployment.json`. Esta es la ruta de acceso del archivo de manifiesto de implementación de IoT Edge. |
    | Suscripción de Azure | Seleccione la suscripción que contiene el centro de IoT.|
    | Nombre de IoT Hub | Seleccione IoT Hub.|
    | Elegir uno o varios dispositivos | Elija si quiere que la canalización de versión se implemente en uno o varios dispositivos. Si implementa en un único dispositivo, escriba el **IoT Edge device ID** (Id. de dispositivo IoT Edge). Si va a implementar en varios dispositivos, especifique la **condición de destino** del dispositivo. La condición de destino es un filtro para asociar un conjunto de dispositivos de IoT Edge en IoT Hub. Si quiere usar etiquetas de dispositivo como condición, debe actualizar las etiquetas de dispositivo correspondientes con el dispositivo gemelo de IoT Hub. Actualice **IoT Edge deployment ID** (Id. de implementación de IoT Edge) e **IoT Edge deployment priority** (Prioridad de implementación de IoT Edge) en la configuración avanzada. Para más información acerca de cómo crear una implementación para varios dispositivos, consulte [Descripción de las implementaciones automáticas de IoT Edge](../articles/iot-edge/module-deployment-monitoring.md). |
    | Id. de dispositivo o condición de destino | En función de la selección anterior, especifique un id. de dispositivo o [condición de destino](../articles/iot-edge/module-deployment-monitoring.md#target-condition) que implementar en varios dispositivos. |
    | Avanzado | Para el id. de implementación de IoT Edge, especifique `$(System.TeamProject)-$(Release.EnvironmentName)`. Esta variable asigna el proyecto y el nombre de la versión con el id. de implementación de IoT Edge. |

    ![Adición de tareas de Azure IoT Edge para la fase de desarrollo](./media/iot-edge-create-release-pipeline-for-continuous-deployment/add-quality-assurance-task.png)

11. Seleccione **Guardar** para guardar los cambios en la nueva canalización de versión. Para volver a la vista de canalización, seleccione la pestaña **Canalización** en el menú.
