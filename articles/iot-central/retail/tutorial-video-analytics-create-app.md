---
title: 'Tutorial: Creación de una aplicación de análisis de vídeo con detección de objetos y movimiento en Azure IoT Central'
description: En este tutorial se muestra cómo crear una aplicación de análisis de vídeo en IoT Central. Se crea, se personaliza y se conecta a otros servicios de Azure.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
author: KishorIoT
ms.author: nandab
ms.date: 07/31/2020
ms.openlocfilehash: b98406984f2c9f2adfca030369a6ea3d47a786f3
ms.sourcegitcommit: e2b36c60a53904ecf3b99b3f1d36be00fbde24fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88762784"
---
# <a name="tutorial-create-a-video-analytics---object-and-motion-detection-application-in-azure-iot-central"></a>Tutorial: Creación de una aplicación de análisis de vídeo con detección de objetos y movimiento en Azure IoT Central

Como creador de soluciones, aprenda a crear una aplicación de análisis de vídeo con la plantilla de aplicación *Video analytics - object and motion detection* (Análisis de vídeo: detección de objetos y movimiento) de IoT Central, dispositivos Azure IoT Edge y Azure Media Services. La solución usa un comercio minorista para mostrar cómo cumplir con la necesidad empresarial común de supervisar las cámaras de seguridad. La solución utiliza la detección automática de objetos de una fuente de vídeo para identificar y localizar rápidamente eventos de interés.

La aplicación de ejemplo incluye dos dispositivos simulados y una puerta de enlace IoT Edge. En los siguientes tutoriales se muestran dos enfoques para experimentar y comprender las funcionalidades de la puerta de enlace:

* Creación de una puerta de enlace IoT Edge en una máquina virtual de Azure y conexión de una cámara simulada.
* Creación de una puerta de enlace IoT Edge en un dispositivo real, como un Intel NUC y conexión de una cámara real.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Usar la plantilla de aplicación análisis de vídeo de Azure IoT Central para crear una aplicación de comercio minorista
> * Personalizar la configuración de la aplicación
> * Crear una plantilla de dispositivo para un dispositivo de puerta de enlace IoT Edge
> * Agregar un dispositivo de puerta de enlace a la aplicación de IoT Central

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta serie de tutoriales, necesitará lo siguiente:

* Suscripción a Azure. Si no tiene una suscripción de Azure, puede crear una en la [página de suscripción a Azure](https://aka.ms/createazuresubscription).
* Si usa una cámara real, necesita conectividad entre el dispositivo IoT Edge y la cámara y necesita el canal **Protocolo de streaming en tiempo real**.

## <a name="initial-setup"></a>Instalación inicial

En estos tutoriales, se actualizan y se usan varios archivos de configuración. Las versiones iniciales de estos archivos están disponibles en el repositorio de GitHub [LVA-gateway](https://github.com/Azure/live-video-analytics/tree/master/ref-apps/lva-edge-iot-central-gateway). El repositorio también incluye un archivo de texto para anotaciones rápidas que puede descargar y usar para registrar los valores de configuración de los servicios que implemente.

Cree una carpeta llamada *lva-configuration* en el equipo local para guardar copias de estos archivos. A continuación, haga clic con el botón derecho en cada uno de los vínculos siguientes y elija **Guardar como** para guardar el archivo en la carpeta *lva-configuration*:

- [Scratchpad.txt](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/Scratchpad.txt)
- [deployment.amd64.json](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/deployment.amd64.json)
- [LvaEdgeGatewayDcm.json](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/LvaEdgeGatewayDcm.json)
- [state.json](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/state.json)

> [!NOTE]
> El repositorio de GitHub también incluye el código fuente de los módulos IoT Edge **LvaEdgeGatewayModule** y **lvaYolov3**. Para más información sobre cómo trabajar con el código fuente, consulte [Compilación de los módulos de la puerta de enlace de LVA](tutorial-video-analytics-build-module.md).

## <a name="deploy-and-configure-azure-media-services"></a>Implementación y configuración de Azure Media Services

La solución usa una cuenta de Azure Media Services para almacenar los clips de vídeo de detección de objetos realizados por el dispositivo de puerta de enlace IoT Edge.

Al crear la cuenta de Media Services:

- Debe proporcionar un nombre de cuenta, una suscripción de Azure, una ubicación, un grupo de recursos y una cuenta de almacenamiento. Cree una nueva cuenta de almacenamiento con la configuración predeterminada mientras crea la cuenta de Media Services.

- Elija la región **Este de EE. UU.** para la ubicación.

- Cree un nuevo grupo de recursos llamado *lva-rg* en la región **Este de EE. UU.** para las cuentas de Media Services y de almacenamiento. Cuando termine los tutoriales, será fácil la limpieza todos los recursos mediante la eliminación el grupo de recursos *lva-rg*.

Cree la [cuenta de Media Services en Azure Portal](https://portal.azure.com/?r=1#create/Microsoft.MediaService).

> [!TIP]
> Estos tutoriales usan la región **Este de EE. UU.** en todos los ejemplos. Si lo prefiere, puede usar la región más cercana.

Anote el nombre de la cuenta de **Media Services** en el archivo *scratchpad.txt*.

Una vez finalizada la implementación, vaya a la página **Propiedades** de la cuenta de **Media Services**. Anote el valor de **Id. del recurso** en el archivo *scratchpad.txt*, usará este valor más adelante al configurar el módulo IoT Edge.

A continuación, configure una entidad de servicio de Azure Active Directory para el recurso de Media Services. Seleccione **Acceso de API** y, a continuación, seleccione **Autenticación de la entidad de servicio**. Cree una nueva aplicación de Azure Active Directory con el mismo nombre que el recurso de Media Services y cree un secreto con la descripción *IoT Edge Access* (Acceso a IoT Edge).

:::image type="content" source="./media/tutorial-video-analytics-create-app/media-service-authentication.png" alt-text="Configuración de la aplicación de AAD para AMS":::

Cuando se genere el secreto, desplácese hacia abajo hasta la sección **Copie las credenciales para conectarse a la aplicación de la entidad de servicio.** . A continuación, seleccione **JSON**. Puede copiar toda la información de credenciales desde aquí en un solo paso. Anote esta información en el archivo *scratchpad.txt*, la usará más adelante al configurar el módulo IoT Edge.

> [!WARNING]
> Esta es la única oportunidad de ver y guardar el secreto. Si lo pierde, tendrá que generar otro secreto.

## <a name="create-the-azure-iot-central-application"></a>Creación de la aplicación de Azure IoT Central

En esta sección, creará una nueva aplicación de Azure IoT Central a partir de una plantilla. Usará esta aplicación a lo largo de la serie de tutoriales para crear una solución completa.

Para crear una nueva aplicación de Azure IoT Central:

1. Vaya al sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral).

1. Inicie sesión con las credenciales que usa para acceder a la suscripción de Azure.

1. Para empezar a crear una nueva aplicación de Azure IoT Central, seleccione **New Application** (Nueva aplicación) en la página **Build** (Compilar).

1. Seleccione **Retail** (Comercio minorista). En la página de comercio minorista se muestran varias plantillas de aplicación de venta al por menor.

Para crear una nueva aplicación de análisis de vídeo:

1. Seleccione la plantilla de aplicación **Video analytics - object and motion detection** (Análisis de vídeo: detección de objetos y movimiento). Esta plantilla incluye plantillas de dispositivo para los dispositivos usados en el tutorial. La plantilla incluye paneles de ejemplo que los operadores pueden usar para realizar tareas como la supervisión y la administración de las cámaras.

1. Opcionalmente, elija un valor descriptivo para **Application name** (Nombre de la aplicación). Esta aplicación se basa en un comercio minorista ficticio llamado Northwind Traders. En el tutorial se usa el **Nombre de aplicación** *Northwind Traders video analytics*.

    > [!NOTE]
    > Aunque use un **Nombre de aplicación** descriptivo, debe usar un valor único para la **dirección URL** de la aplicación.

1. Si tiene una suscripción de Azure, seleccione el **Directorio**, la **suscripción de Azure** y **United States** (Estados Unidos) para la **Location** (Ubicación). Si no tiene una suscripción, puede activar **7-day free trial** (Evaluación gratuita de 7 días) y completar la información de contacto necesaria. En este tutorial se usan tres dispositivos: dos cámaras y un dispositivo IoT Edge, por lo que si no usa la evaluación gratuita, se le facturará el uso.

    Para más información sobre los directorios, las suscripciones y las ubicaciones, consulte la guía de inicio rápido [Creación de una aplicación](../core/quick-deploy-iot-central.md).

1. Seleccione **Crear**.

    :::image type="content" source="./media/tutorial-video-analytics-create-app/new-application.png" alt-text="Página de creación de una aplicación de Azure IoT Central":::

### <a name="retrieve-the-configuration-data"></a>Recuperación de los datos de configuración

Más adelante en este tutorial, al configurar la puerta de enlace IoT Edge, necesitará algunos datos de configuración de la aplicación de IoT Central. El dispositivo IoT Edge necesita esta información para registrarse y conectarse a la aplicación.

En la sección **Administration** (Administración), seleccione **su aplicación** y anote el valor de **Application URL** (Dirección URL de la aplicación) y de **Application ID** (Id. de la aplicación) en el archivo *scratchpad.txt*:

:::image type="content" source="./media/tutorial-video-analytics-create-app/administration.png" alt-text="Administración":::

Seleccione **API Tokens** (Tokens de API) y genere un nuevo token llamado **LVAEdgeToken** para el rol **Operator** (Operador):

:::image type="content" source="./media/tutorial-video-analytics-create-app/token.png" alt-text="Generar token":::

Anote el token en el archivo *scratchpad.txt* para más adelante. Cuando el cuadro de diálogo se cierre, no podrá volver a ver el token.

En la sección **Administration** (Administración), seleccione **Device connection** (Conexión del dispositivo) y, a continuación, seleccione **SAS-IoT-Devices** (Dispositivos-IoT-SAS).

Anote el valor de **Primary key** (Clave principal) de los dispositivos en el archivo *scratchpad.txt*. Usará este *token de firma de acceso compartido de grupo principal* más adelante cuando configure el dispositivo IoT Edge.

## <a name="edit-the-deployment-manifest"></a>Edición del manifiesto de implementación

Un módulo IoT Edge se implementa con un manifiesto de implementación. En IoT Central, puede importar el manifiesto como una plantilla de dispositivo y, a continuación, dejar que IoT Central administre la implementación del módulo.

Para preparar el manifiesto de implementación:

1. Abra el archivo *deployment.amd64.json*, que guardó en la carpeta *lva-configuration*, con un editor de texto.

1. Busque la configuración `LvaEdgeGatewayModule` y cambie el nombre de la imagen como se muestra en el siguiente fragmento de código:

    ```json
    "LvaEdgeGatewayModule": {
        "settings": {
            "image": "mcr.microsoft.com/lva-utilities/lva-edge-iotc-gateway:1.0-amd64",
    ```

1. Agregue el nombre de la cuenta de Media Services en el nodo `env` de la sección `LvaEdgeGatewayModule`. Anotó este nombre de cuenta en el archivo *scratchpad.txt*:

    ```json
    "env": {
        "lvaEdgeModuleId": {
            "value": "lvaEdge"
        },
        "amsAccountName": {
            "value": "<YOUR_AZURE_MEDIA_ACCOUNT_NAME>"
        }
    }
    ```

1. La plantilla no expone estas propiedades en IoT Central, por lo que debe agregar los valores de configuración de Media Services al manifiesto de implementación. Busque el módulo `lvaEdge` y reemplace los marcadores de posición por los valores que anotó en el archivo *scratchpad.txt* cuando creó la cuenta de Media Services.

    El elemento `azureMediaServicesArmId` es el **identificador de recurso** que anotó en el archivo *scratchpad.txt* cuando creó la cuenta de Media Services.

    Anotó los valores de `aadTenantId`, `aadServicePrincipalAppId` y `aadServicePrincipalSecret` en el archivo *scratchpad.txt* cuando creó la entidad de servicio para la cuenta de Media Services:

    ```json
    {
        "lvaEdge":{
        "properties.desired": {
            "applicationDataDirectory": "/var/lib/azuremediaservices",
            "azureMediaServicesArmId": "[Resource ID from scratchpad]",
            "aadTenantId": "[AADTenantID from scratchpad]",
            "aadServicePrincipalAppId": "[AadClientId from scratchpad]",
            "aadServicePrincipalSecret": "[AadSecret from scratchpad]",
            "aadEndpoint": "https://login.microsoftonline.com",
            "aadResourceId": "https://management.core.windows.net/",
            "armEndpoint": "https://management.azure.com/",
            "diagnosticsEventsOutputName": "AmsDiagnostics",
            "operationalMetricsOutputName": "AmsOperational",
            "logCategories": "Application,Event",
            "AllowUnsecuredEndpoints": "true",
            "TelemetryOptOut": false
            }
        }
    }
    ```

1. Guarde los cambios.

Opcionalmente, puede reemplazar el módulo Yolov3 por el [módulo de extensión de inteligencia artificial perimetral del servidor de modelos OpenVINO&trade;](https://github.com/openvinotoolkit/model_server/tree/ams/extras/ams_wrapper) de Intel, que está optimizado para hardware. Puede descargar el manifiesto de implementación de ejemplo [deployment.openvino.amd64.json](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/deployment.openvino.amd64.json). Este manifiesto reemplaza los valores de configuración del módulo `lvaYolov3` por la configuración siguiente:

```json
"OpenVINOModelServerEdgeAIExtensionModule": {
    "settings": {
        "image": "marketplace.azurecr.io/intel_corporation/open_vino",
        "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"4000/tcp\":[{\"HostPort\":\"4000\"}]}},\"Cmd\":[\"/ams_wrapper/start_ams.py\",\"--ams_port=4000\",\"--ovms_port=9000\"]}"
    },
    "type": "docker",
    "version": "1.0",
    "status": "running",
    "restartPolicy": "always"
}
```

## <a name="create-the-azure-iot-edge-gateway-device"></a>Creación del dispositivo de puerta de enlace IoT Edge de Azure

La aplicación de análisis de vídeo con detección de objetos y movimiento incluye la plantilla de dispositivo **LVA Edge Object Detector** (Detector de objetos LVA Edge) y la plantilla de dispositivo **LVA Edge Motion Detection** (Detección de movimiento LVA Edge). En esta sección, creará una plantilla de dispositivo de puerta de enlace mediante el manifiesto de implementación y agregará el dispositivo de puerta de enlace a la aplicación de IoT Central.

### <a name="create-a-device-template-for-the-lva-edge-gateway"></a>Creación de la plantilla de dispositivo LVA Edge Gateway (Puerta de enlace LVA Edge)

Para importar el manifiesto de implementación y crear la plantilla de dispositivo **LVA Edge Gateway** (Puerta de enlace LVA Edge):

1. En la aplicación de IoT Central, vaya a **Device Templates** (Plantillas de dispositivo) y seleccione **+ New** (+ Nueva).

1. En la página **Seleccionar tipo de plantilla**, seleccione el icono **Azure IoT Edge**. Después, seleccione **Next: Customize** (Personalizar)

1. En la página **Upload an Azure IoT Edge deployment manifest** (Cargar un manifiesto de implementación de Azure IoT Edge), escriba *LVA Edge Gateway* (Puerta de enlace LVA Edge) como nombre de la plantilla y active **Gateway device with downstream devices** (Dispositivo de puerta de enlace con dispositivos de bajada).

    No busque aún el manifiesto de implementación. Si lo hace, el asistente para la implementación espera una interfaz para cada módulo, pero solo se necesita exponer la interfaz para **LvaEdgeGatewayModule**. Cargue el manifiesto en un paso posterior.

    :::image type="content" source="./media/tutorial-video-analytics-create-app/upload-deployment-manifest.png" alt-text="No cargar el manifiesto de implementación":::

    Seleccione **Siguiente: Review** (Siguiente: revisar).

1. En la página **Revisar**, seleccione **Crear**.

### <a name="import-the-device-capability-model"></a>Importación del modelo de funcionalidad del dispositivo

La plantilla de dispositivo debe incluir un modelo de funcionalidad del dispositivo. En la página **LVA Edge Gateway** (Puerta de enlace LVA Edge), seleccione el icono **Import capability model** (Importar modelo de funcionalidad). Vaya a la carpeta *lva-configuration* que creó anteriormente y seleccione el archivo *LvaEdgeGatewayDcm.json*.

La plantilla de dispositivo **LVA Edge Gateway** (Puerta de enlace LVA Edge) ahora incluye el módulo **LVA Edge Gateway Module** (Módulo de puerta de enlace LVA Edge) junto con tres interfaces: **Device information** (Información del dispositivo), **LVA Edge Gateway Settings** (Configuración de puerta de enlace LVA Edge) y **LVA Edge Gateway Interface** (Interfaz de puerta de enlace LVA Edge).

### <a name="replace-the-manifest"></a>Reemplazo del manifiesto

En la página **LVA Edge Gateway** (Puerta de enlace LVA Edge), seleccione **+ Replace manifest** (+ Reemplazar manifiesto).

:::image type="content" source="./media/tutorial-video-analytics-create-app/replace-manifest.png" alt-text="Reemplazar manifiesto":::

Vaya a la carpeta *lva-configuration* y seleccione el archivo de manifiesto *deployment.amd64.json* que editó previamente. Seleccione **Cargar**. Una vez completada la validación, seleccione **Reemplazar**.

### <a name="add-relationships"></a>Adición de relaciones

En la plantilla de dispositivo **LVA Edge Gateway** (Puerta de enlace LVA Edge), en **Modules/LVA Edge Gateway Module** (Módulos/Módulo de puerta de enlace LVA Edge), seleccione **Relationships** (Relaciones). Seleccione **+ Add relationship** (+ Agregar relación) y agregue las dos relaciones siguientes:

|Display Name (Nombre para mostrar)               |Nombre          |Destino |
|-------------------------- |------------- |------ |
|LVA Edge Motion Detector (Detector de movimiento LVA Edge)   |Use el valor predeterminado   |LVA Edge Motion Detector Device (Dispositivo de detección de movimiento LVA Edge) |
|LVA Edge Object Detector (Detector de objetos LVA Edge)   |Use el valor predeterminado   |LVA Edge Object Detector Device (Dispositivo de detección de objetos LVA Edge) |

Después, seleccione **Guardar**.

:::image type="content" source="media/tutorial-video-analytics-create-app/relationships.png" alt-text="Agregar relaciones":::

### <a name="add-views"></a>Adición de vistas

La plantilla de dispositivo **LVA Edge Gateway** (Puerta de enlace LVA Edge) no incluye definiciones de vistas.

Para agregar una vista a la plantilla de dispositivo:

1. En la plantilla de dispositivo **LVA Edge Gateway** (Puerta de enlace LVA Edge), vaya a **Views** (Vistas) y seleccione el icono **Visualizing the device** (Visualización del dispositivo).

1. Escriba *LVA Edge Gateway device* (Dispositivo de puerta de enlace LVA Edge) como nombre de la vista.

1. Agregue los siguientes iconos a la vista:

    * Un icono con las propiedades de **Información del dispositivo**: **Device model** (Modelo del dispositivo), **Manufacturer** (Fabricante), **Operating system** (Sistema operativo), **Processor architecture** (Arquitectura del procesador), **Software version** (Versión de software), **Total memory** (Memoria total), and **Total storage** (Almacenamiento total).
    * Un icono de gráfico de líneas con los valores de telemetría **Free Memory** (Memoria libre) y **System Heartbeat** (Latido del sistema).
    * Un icono de historial de eventos con los siguientes eventos: **Create Camera** (Crear cámara), **Delete Camera** (Eliminar cámara), **Module Restart** (Módulo reiniciado), **Module Started** (Módulo iniciado), **Module Stopped** (Módulo detenido).
    * Un icono de último valor conocido de tamaño 2x1 que muestre el dato de telemetría **IoT Central Client State** (Estado del cliente de IoT Central).
    * Un icono de último valor conocido de tamaño 2x1 que muestre el dato de telemetría **Module State** (Estado del módulo).
    * Un icono de último valor conocido de tamaño 1x1 que muestre el dato de telemetría **System Heartbeat** (Latido del sistema).
    * Un icono de último valor conocido de tamaño 1x1 que muestre el dato de telemetría **Connected Cameras** (Cámaras conectadas).

    :::image type="content" source="media/tutorial-video-analytics-create-app/gateway-dashboard.png" alt-text="Panel":::

1. Seleccione **Guardar**.

### <a name="publish-the-device-template"></a>Publicación de la plantilla de dispositivo

Para poder agregar un dispositivo a la aplicación, debe publicar la plantilla de dispositivo:

1. En la plantilla de dispositivo **LVA Edge Gateway** (Puerta de enlace LVA Edge), seleccione **Publish** (Publicar).

1. En la página **Publish this device template to the application** (Publicar esta plantilla de dispositivo en la aplicación), seleccione **Publicar** (Publicar).

**LVA Edge Gateway** (Puerta de enlace LVA Edge) ahora está disponible como un tipo de dispositivo que se puede usar en la página **Devices** (Dispositivos) de la aplicación.

## <a name="add-a-gateway-device"></a>Adición de un dispositivo de puerta de enlace

Para agregar un dispositivo **LVA Edge Gateway** (Puerta de enlace LVA Edge) a la aplicación:

1. Vaya a la página **Devices** (Dispositivos) y seleccione la plantilla de dispositivo **LVA Edge Gateway** (Puerta de enlace LVA Edge).

1. Seleccione **+ Nuevo**.

1. En el cuadro de diálogo **Create a new device** (Crear un dispositivo), cambie el nombre de dispositivo a *LVA Gateway 001* y cambie el identificador de dispositivo a *lva-gateway-001*.

    > [!NOTE]
    > El identificador de dispositivo debe ser único en la aplicación.

1. Seleccione **Crear**.

El estado del dispositivo es **Registered** (Registrado).

### <a name="get-the-device-credentials"></a>Obtención de las credenciales del dispositivo

Necesita las credenciales que permiten que el dispositivo se conecte a la aplicación de IoT Central. Para obtener las credenciales del dispositivo:

1. En la página **Devices** (Dispositivos), seleccione el dispositivo **lva-gateway-001** que acaba de crear.

1. Seleccione **Conectar**.

1. En la página **Device connection** (Conexión del dispositivo), anote en el archivo *scratchpad.txt* los valores de **ID Scope** (Ámbito de id.), **Device ID** (Id. de dispositivo) y **Primary Key** (Clave principal) del dispositivo. Utilizará estos valores más adelante.

1. Asegúrese de que el método de conexión se establece en **Shared access signature** (Firma de acceso compartido).

1. Seleccione **Cerrar**.

## <a name="next-steps"></a>Pasos siguientes

Ahora ha creado una aplicación de IoT Central con la plantilla de aplicación **Video analytics - object and motion detection** (Análisis de vídeo: detección de objetos y movimiento), ha creado una plantilla de dispositivo para el dispositivo de puerta de enlace y ha agregado un dispositivo de puerta de enlace a la aplicación.

Si desea probar la aplicación de análisis de vídeo con detección de objetos y movimiento con módulos IoT Edge que ejecutan una máquina virtual en la nube con secuencias de vídeo simuladas:

> [!div class="nextstepaction"]
> [Creación de una instancia de IoT Edge para análisis de vídeo (máquina virtual Linux)](./tutorial-video-analytics-iot-edge-vm.md)

Si desea probar la aplicación de análisis de vídeo con detección de objetos y movimiento con módulos IoT Edge que ejecutan un dispositivo real con una cámara **ONVIF** real:

> [!div class="nextstepaction"]
> [Creación de una instancia de IoT Edge para análisis de vídeo (Intel NUC)](./tutorial-video-analytics-iot-edge-nuc.md)
