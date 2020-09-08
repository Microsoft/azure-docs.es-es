---
title: Ejecución de Azure Stream Analytics en Azure Stack (versión preliminar)
description: Cree un trabajo perimetral de Azure Stream Analytics e impleméntelo en Azure Stack Hub mediante el entorno de ejecución de IoT Edge.
ms.service: stream-analytics
author: raan
ms.author: raan
ms.reviewer: mamccrea
ms.topic: how-to
ms.date: 08/21/2020
ms.custom: seodec18
ms.openlocfilehash: 1fe035d99f8a5962406d5aae3f093d71d432b310
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88860470"
---
# <a name="run-azure-stream-analytics-on-azure-stack-preview"></a>Ejecución de Azure Stream Analytics en Azure Stack (versión preliminar)

> [!IMPORTANT]
> Esta funcionalidad se encuentra en versión preliminar y su uso no se recomienda en producción.

Puede ejecutar Azure Stream Analytics en Azure Stack Hub como módulo de IoT Edge. Se han agregado configuraciones al módulo de IoT Edge, que le permiten interactuar con Blob Storage, Event Hubs e IoT Hub que se ejecutan en una suscripción de Azure Stack Hub ya que permiten las direcciones URL personalizadas que se encuentran en cada empleo de Azure Stack Hub.

Con Stream Analytics en Azure Stack, puede crear arquitecturas realmente híbridas para el procesamiento de secuencias en su propia nube autónoma y privada, que se puede conectar o desconectar con aplicaciones nativas de nube mediante servicios de Azure coherentes locales. 

En este artículo se muestra cómo transmitir datos desde IoT Hub o centro de eventos a otro centro de eventos o Blob Storage en una suscripción de Azure Stack Hub y procesarlos con Stream Analytics.

## <a name="set-up-environments"></a>Configurar entornos

Azure Stream Analytics es un servicio híbrido en Azure Stack Hub. Se trata de un módulo de IoT Edge que se configura en Azure, pero que se puede ejecutar en Azure Stack Hub.  

Si no está familiarizado con Azure Stack Hub o IoT Edge, siga las instrucciones que se indican a continuación para configurar los entornos.

### <a name="prepare-the-azure-stack-hub-environment"></a>Preparación del entorno de Azure Stack Hub

Cree una suscripción a Azure Stack Hub. Para más información, consulte el [tutorial para crear una suscripción a Azure Stack Hub.](https://docs.microsoft.com/azure-stack/user/azure-stack-subscribe-services/)

Si quiere evaluar Azure Stack Hub en su propio servidor, puede usar el Kit de desarrollo de Azure Stack (ASDK).  Para más información sobre ASDK, consulte la [información general sobre ASDK](https://docs.microsoft.com/azure-stack/asdk/).

### <a name="install-the-iot-edge-runtime"></a>Instalación del entorno de ejecución de IoT Edge

Para ejecutar Azure Stream Analytics en Azure Stack Hub, el dispositivo debe tener el entorno de ejecución de IoT Edge y conectividad de red con Azure Stack Hub o ser una máquina virtual que se ejecute en Azure Stack Hub. El entorno de ejecución de IoT Edge permite que los trabajos perimetrales de Stream Analytics se integren con Azure Storage y Azure Event Hubs que se ejecutan en Azure Stack Hub. Para más información, vea [Azure Stream Analytics en IoT Edge](stream-analytics-edge.md). 

Además de tener acceso de red para los recursos de Azure Stack Hub, el dispositivo IoT Edge (o máquina virtual) necesita acceso a Azure IoT Hub en la nube pública de Azure para configurar el módulo de Stream Analytics. 

En las guías siguientes se muestra cómo configurar el entorno de ejecución de IoT Edge en el dispositivo o máquina virtual:

* [Instalación del entorno de ejecución de Azure IoT Edge en Windows](../iot-edge/how-to-install-iot-edge-windows.md)
* [Instalación del entorno de ejecución de Azure IoT Edge en sistemas Linux basados en Debian](../iot-edge/how-to-install-iot-edge-linux.md)


## <a name="create-an-azure-stream-analytics-edge-job"></a>Creación de un trabajo perimetral de Azure Stream Analytics

Los trabajos de ASA en Edge se ejecutan en contenedores implementados en dispositivos de Azure IoT Edge. Constan de dos partes:
* Una parte en la nube que es responsable de la definición del trabajo: los usuarios definen las entradas, la salida, las consultas y otros valores (eventos que no funcionan, etc.) en la nube.
* Un módulo que se ejecuta en dispositivos IoT. Contiene el motor de ASA y recibe la definición del trabajo de la nube.

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Cuando se crea un trabajo de Azure Stream Analytics para ejecutarse en un dispositivo de IoT Edge, se debe almacenar de forma que se pueda llamar desde el dispositivo. Puede usar una cuenta de Azure Storage o crear una nueva ahora.
1. En Azure Portal, vaya a **Crear un recurso > Almacenamiento > Cuenta de almacenamiento: blob, archivo, tabla, cola**.
2. Especifique los siguientes valores para crear una cuenta de almacenamiento:

   | Campo | Value |
   | --- | --- |
   | Nombre | Especifique un nombre único para la cuenta de almacenamiento. |
   | Location | Elija una ubicación cercana a usted.|
   | Suscripción | Elija la misma suscripción que IoT Hub.|
   | Grupo de recursos | Se recomienda usar el mismo grupo de recursos para todos los recursos de prueba que se crean en las guías de [inicio rápido de IoT Edge](https://docs.microsoft.com/azure/iot-edge/quickstart) y los tutoriales. Por ejemplo, **IoTEdgeResources**. |

3. Mantenga los valores predeterminados en los restantes campos y seleccione **Crear**.


### <a name="create-a-new-job"></a>Crear un trabajo

1. En Azure Portal, vaya a **Crear un recurso > Internet de las cosas > Trabajo de Stream Analytics.**
2. Especifique los siguientes valores para crear una cuenta de almacenamiento:

   | Campo | Value |
   | --- | --- |
   | Nombre del trabajo | Especifique el nombre del trabajo. Por ejemplo, **IoTEdgeJob** |
   | Suscripción | Elija la misma suscripción que IoT Hub.|
   | Grupo de recursos | Se recomienda usar el mismo grupo de recursos para todos los recursos de prueba que se crean en las [guías de inicio rápido de IoT Edge](https://docs.microsoft.com/azure/iot-edge/quickstart) y los tutoriales. Por ejemplo, **IoTEdgeResources**. |
   | Location | Elija una ubicación cercana a usted. |
   | Entorno de hospedaje | Seleccionar **Edge**. |

3. Seleccione **Crear**.

### <a name="configure-your-job"></a>Configuración del trabajo

Una vez que se ha creado el trabajo de Stream Analytics en Azure Portal, puede configurarlo con una entrada, una salida y una consulta para ejecutarlo en los datos que lo atraviesan. Puede especificar manualmente las entradas de una instancia de IoT Hub o de un centro de eventos en una suscripción de Azure Stack Hub.

1. Vaya al trabajo de Stream Analytics en Azure Portal.
2. En **Configurar**, seleccione **Configuración de cuenta de almacenamiento** y elija la cuenta de almacenamiento que creó en el paso anterior.
   > [!div class="mx-imgBorder"]
   > [ ![Configuración de la cuenta de almacenamiento del trabajo](media/on-azure-stack/storage-account-settings.png) ](media/on-azure-stack/storage-account-settings.png#lightbox)
3. En **Topología de trabajo**, seleccione **Entradas** y, después, **Agregar entrada de flujo**.
4. Elija **IoT Hub**, **Centro de eventos** o **Centro de Microsoft Edge** en la lista desplegable. 
5. Si la entrada es un centro de eventos o IoT Hub en una suscripción de Azure Stack Hub, proporcione información manualmente como se muestra a continuación.

   #### <a name="event-hub"></a>Centro de eventos

   | Campo | Value |
   | --- | --- |
   | Alias de entrada | Nombre descriptivo que se usará en la consulta del trabajo para hacer referencia a esta entrada. |
   | Espacio de nombres de Service Bus | El espacio de nombres es un contenedor para un conjunto de entidades de mensajería. Al crear un centro de eventos, también se crea el espacio de nombres. (Ejemplo: *sb://<Event Hub Name>.eventhub.shanghai.azurestack.corp.microsoft.com*) |
   | Nombre del centro de eventos | Nombre del centro de eventos que se usa como entrada. |
   | Nombre de la directiva del centro de eventos | Directiva de acceso compartido que proporciona acceso al centro de eventos. Cada directiva de acceso compartido tiene un nombre, los permisos establecidos y las claves de acceso. Esta opción se rellena automáticamente, a menos que elija proporcionar la configuración del centro de eventos manualmente. |
   | Clave de la directiva del Centro de eventos | Clave de acceso compartido que se usa para autorizar el acceso al centro de eventos. Esta opción se rellena automáticamente, a menos que elija proporcionar la configuración del centro de eventos manualmente. Puede encontrarlo en la configuración del centro de eventos. |
   | Grupo de consumidores del centro de eventos (opcional) | Se recomienda encarecidamente usar un grupo de consumidores distinto para cada trabajo de Stream Analytics. Esta cadena identifica el grupo de consumidores que se usa para la ingesta de datos desde el centro de eventos. Si no se especifica ningún grupo de consumidores, el trabajo de Stream Analytics usa el grupo de consumidores $Default. |
   | Recuento de particiones | Recuento de particiones es el número de particiones en un centro de eventos. |

   > [!div class="mx-imgBorder"]
   > [ ![Entrada de centro de eventos](media/on-azure-stack/event-hub-input.png) ](media/on-azure-stack/event-hub-input.png#lightbox)

   #### <a name="iot-hub"></a>IoT Hub

   | Campo | Value |
   | --- | --- |
   | Alias de entrada | Nombre descriptivo que se usará en la consulta del trabajo para hacer referencia a esta entrada. |
   | IoT Hub | Nombre de la instancia de IoT Hub que se usa como entrada. (Ejemplo: *<IoT Hub Name>.shanghai.azurestack.corp.microsoft.com*) |
   | El nombre de la directiva de acceso compartido | Directiva de acceso compartido que proporciona acceso a IoT Hub. Cada directiva de acceso compartido tiene un nombre, los permisos establecidos y las claves de acceso. |
   | Clave de directiva de acceso compartido | Clave de acceso compartido que se usa para autorizar el acceso a IoT Hub. Esta opción se rellena automáticamente, a menos que elija proporcionar la configuración de IoT Hub manualmente. |
   | Grupo de consumidores (opcional) | Se recomienda encarecidamente usar un grupo de consumidores distinto para cada trabajo de Stream Analytics. El grupo de consumidores que se usa para ingerir datos desde Azure IoT Hub. Stream Analytics usa el grupo de consumidores $Default, a menos que se especifique lo contrario. |
   | Recuento de particiones | Recuento de particiones es el número de particiones en un centro de eventos. |

   > [!div class="mx-imgBorder"]
   > [ ![Entrada de IoT Hub](media/on-azure-stack/iot-hub-input.png) ](media/on-azure-stack/iot-hub-input.png#lightbox)

6. Mantenga los valores predeterminados en los restantes campos y seleccione Guardar.
7. En Topología de trabajo, abra Salidas y seleccione Agregar.
8. Elija Blob Storage, Centro de eventos o Centro de Microsoft Edge en la lista desplegable.
9. Si la salida es un centro de eventos o Blob Storage en una suscripción de Azure Stack Hub, proporcione información manualmente como se muestra a continuación.

   #### <a name="event-hub"></a>Centro de eventos

   | Campo | Value |
   | --- | --- |
   | Alias de salida | Nombre descriptivo usado en las consultas para dirigir la salida de la consulta a este centro de eventos. |
   | Espacio de nombres de Service Bus | Contenedor para un conjunto de entidades de mensajería. Cuando creó un nuevo centro de eventos, también creó un espacio de nombres de Service Bus. (Ejemplo: *sb://<Event Hub Name>.eventhub.shanghai.azurestack.corp.microsoft.com*) |
   | Nombre del centro de eventos | Nombre de la salida del centro de eventos. |
   | Nombre de la directiva del centro de eventos | Directiva de acceso compartido, que puede crear en la pestaña Configurar del centro de eventos. Cada directiva de acceso compartido tiene un nombre, los permisos establecidos y las claves de acceso. |
   | Clave de la directiva del Centro de eventos | Clave de acceso compartido usada para autenticar el acceso al espacio de nombres del centro de eventos. |

   > [!div class="mx-imgBorder"]
   > [ ![Salida de centro de eventos](media/on-azure-stack/event-hub-output.png) ](media/on-azure-stack/event-hub-output.png#lightbox)

   #### <a name="blob-storage"></a>Blob Storage 

   | Campo | Value |
   | --- | --- |
   | Alias de salida | Un nombre descriptivo usado en las consultas para dirigir la salida de la consulta a este almacenamiento de blobs. |
   | Cuenta de almacenamiento | El nombre de la cuenta de almacenamiento a la que va a enviar la salida. (Ejemplo: *<Storage Account Name>.blob.shanghai.azurestack.corp.microsoft.com*) |
   | Clave de cuenta de almacenamiento | La clave secreta asociada con la cuenta de almacenamiento. Esta opción se rellena automáticamente, a menos que elija proporcionar la configuración de Blob Storage manualmente. |

> [!NOTE]
> El formato Parquet no se admite para los trabajos perimetrales en Azure Stack Hub. Para las filas mínimas y el tiempo máximo, especifique 0 o deje la opción en blanco.


## <a name="deploy-stream-analytics-on-a-vm-or-device-connected-to-azure-stack"></a>Implementación de Stream Analytics en una máquina virtual o un dispositivo conectado a Azure Stack

1. En Azure Portal, abra IoT Hub. Vaya a **IoT Edge** y haga clic en el dispositivo (máquina virtual) que va a ser el destino de esta implementación.
2. Seleccione **Set modules** (Establecer módulos). A continuación, seleccione **+ Agregar** y, elija **Módulo de Azure Stream Analytics**. 
3. Seleccione la suscripción y el trabajo perimetral de Stream Analytics que creó. Haga clic en **Guardar** y seleccione **Siguiente: Rutas**.

   > [!div class="mx-imgBorder"]
   > [ ![Adición de módulos](media/on-azure-stack/edge-modules.png) ](media/on-azure-stack/edge-modules.png#lightbox)

4. Haga clic en **Revisar y crear >** .
5. En la pestaña **Revisar + crear**, seleccione **Crear**. 
   > [!div class="mx-imgBorder"]
   > [ ![Manifiesto](media/on-azure-stack/module-content.png) ](media/on-azure-stack/module-content.png#lightbox)
6. Confirme que el módulo se agrega a la lista.
   > [!div class="mx-imgBorder"]
   > [ ![Página de implementación](media/on-azure-stack/edge-deployment.png) ](media/on-azure-stack/edge-deployment.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes
- [Azure Stream Analytics en IoT Edge](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-edge)
- [Desarrollo de trabajos perimetrales de Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
