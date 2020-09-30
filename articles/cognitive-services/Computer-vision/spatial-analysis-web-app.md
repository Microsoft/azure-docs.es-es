---
title: Implementación de una aplicación web de análisis espacial
titleSuffix: Azure Cognitive Services
description: Aprenda a usar el análisis espacial en una aplicación web.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 06/10/2020
ms.author: aahi
ms.openlocfilehash: 440f901f06e431c371b7445f4a04499c475c9aa1
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932537"
---
# <a name="how-to-deploy-a-people-counting-web-application"></a>Procedimientos: Implementación de una aplicación web de recuento de personas

Use este artículo para aprender a integrar el análisis espacial en una aplicación web que comprenda el movimiento de las personas y supervise la cantidad de personas que ocupan un espacio físico. 

En este tutorial, aprenderá a:

* Implementar el contenedor de análisis espacial
* Configurar la operación y la cámara
* Configurar la conexión de IoT Hub en la aplicación web
* Implementar y probar la aplicación web

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* Conocimientos básicos de las configuraciones de implementación de Azure IoT Edge y una instancia de [Azure IoT Hub](https://docs.microsoft.com/azure/iot-hub/).
* Un [equipo host](spatial-analysis-container.md) configurado.

## <a name="deploy-the-spatial-analysis-container"></a>Implementación del contenedor de análisis espacial

Rellene la [aplicación de solicitud](https://aka.ms/csgate) para obtener acceso a la ejecución del contenedor. 

Siga [la configuración del equipo host](./spatial-analysis-container.md) para configurar el equipo host y conecte un dispositivo IoT Edge a Azure IoT Hub. 

### <a name="deploy-an-azure-iot-hub-service-in-your-subscription"></a>Implementación de un servicio Azure IoT Hub en la suscripción

En primer lugar, cree una instancia de un servicio Azure IoT Hub con el plan de tarifa Estándar (S1) o el nivel Gratis (S0). Siga estas instrucciones para crear esta instancia con la CLI de Azure.

Rellene los parámetros obligatorios:
* Suscripción: nombre o id. de la suscripción de Azure
* Grupo de recursos: cree un nombre para el grupo de recursos
* Nombre de la instancia de IoT Hub: cree un nombre para la instancia de IoT Hub
* Nombre de la instancia de IoTHub: nombre de la instancia de IoT Hub que creó 
* Nombre de dispositivo Edge: cree un nombre para el dispositivo Edge

```azurecli
az login
az account set --subscription <name or ID of Azure Subscription>
az group create --name "<Resource Group Name>" --location "WestUS"

az iot hub create --name "<IoT Hub Name>" --sku S1 --resource-group "test-resource-group"

az iot hub device-identity create --hub-name "<IoT Hub Name>" --device-id "<Edge Device Name>" --edge-enabled
```

### <a name="deploy-the-container-on-azure-iot-edge-on-the-host-computer"></a>Implementación del contenedor en Azure IoT Edge en el equipo host

Implemente el contenedor de análisis espacial como un módulo de IoT en el equipo host a través de la CLI de Azure. El proceso de implementación requiere un archivo de manifiesto de implementación que describa los contenedores, las variables y las configuraciones que se necesitan para la implementación. Puede encontrar un [manifiesto de implementación](https://github.com/Azure-Samples/cognitive-services-rest-api-samples/) de ejemplo en GitHub, que incluye una configuración de implementación básica para el contenedor *spatial-analysis*. 

> [!NOTE] 
> Los contenedores *spatial-analysis-telegraf* y *spatial-analysis-diagnostics* son opcionales. Puede optar por quitarlos del archivo *DeploymentManifest.json*. Para más información, consulte el artículo [Telemetría y solución de problemas](./spatial-analysis-logging.md). Puede encontrar un archivo *DeploymentManifest.json* de ejemplo en [on Github](https://go.microsoft.com/fwlink/?linkid=2142179). 

### <a name="set-environment-variables"></a>Establecimiento de variables de entorno

La mayoría de las **variables de entorno** para el módulo de IoT Edge ya están establecidas en el archivo *DeploymentManifest.json* de ejemplo anteriormente vinculado. En el archivo, busque las variables de entorno `BILLING_ENDPOINT` y `API_KEY`, que se muestran a continuación. Reemplace los valores por el URI de punto de conexión y la clave de API que creó anteriormente. Asegúrese de que el valor CLUF esté establecido en "accept". 

```json
"EULA": { 
    "value": "accept"
},

"BILLING_ENDPOINT":{ 
    "value": "<Use a key from your Computer Vision resource>"
},
"API_KEY":{
    "value": "<Use the endpoint from your Computer Vision resource>"
}
```

### <a name="configure-the-operation-parameters"></a>Configuración de los parámetros de la operación

Ahora que se completó la configuración inicial del contenedor *spatial-analysis*, el paso siguiente es configurar los parámetros de operaciones y agregarlos a la implementación. 

El primer paso es actualizar el [manifiesto de implementación](https://go.microsoft.com/fwlink/?linkid=2142179) de ejemplo y configurar el id. de operación `cognitiveservices.vision.spatialanalysis-personcount` como se muestra a continuación:


```json
"personcount": {
    "operationId": "cognitiveservices.vision.spatialanalysis-personcount",
    "version": 1,
    "enabled": true,
    "parameters": {
        "VIDEO_URL": "<Replace RTSP URL here>",
        "VIDEO_SOURCE_ID": "<Replace with friendly name>",
        "VIDEO_IS_LIVE":true,
        "DETECTOR_NODE_CONFIG": "{ \"gpu_index\": 0 }",
        "SPACEANALYTICS_CONFIG": "{\"zones\":[{\"name\":\"queue\",\"polygon\":[<Replace with your values>], \"events\": [{\"type\":\"count\"}], \"threshold\":<use 0 for no threshold.}]}"
    }
},
```

Una vez que se actualiza el [manifiesto de implementación](https://go.microsoft.com/fwlink/?linkid=2142179), siga las instrucciones del fabricante de la cámara para instalar la cámara, configurar la dirección URL de la cámara, el nombre de usuario y la contraseña. 

Luego, establezca `VIDEO_URL` en la dirección URL de RTSP de la cámara y las credenciales para conectarse a la cámara.

Si el dispositivo de Edge tiene más de una GPU, seleccione la GPU en la que se va a ejecutar esta operación. Asegúrese de equilibrar la carga de las operaciones donde no hay más de 8 operaciones en ejecución en una sola GPU a la vez.  

Luego, configure la zona en la que quiere contar personas. Para configurar el polígono de la zona, primer siga las instrucciones del fabricante para recuperar un fotograma de la cámara. Para determinar cada vértice del polígono, seleccione un punto en el fotograma, tome las coordenadas x, y del punto con respecto a la esquina superior izquierda del fotograma y divida por las dimensiones correspondientes del fotograma. Establezca los resultados como coordenadas x, y del vértice. Puede establecer la configuración del polígono de la zona en el campo `SPACEANALYTICS_CONFIG`.

Se trata de un fotograma de vídeo de ejemplo que muestra cómo se calculan las coordenadas del vértice correspondientes a un fotograma de tamaño 1920/1080.
![Fotograma de vídeo de ejemplo](./media/spatial-analysis/sample-video-frame.jpg)

También puede seleccionar un umbral de confianza para cuando se cuenten las personas detectadas y se generen eventos. Establezca el umbral en 0 si quiere que se generen todos los eventos.

### <a name="execute-the-deployment"></a>Ejecución de la implementación

Ahora que se completó el [manifiesto de implementación](https://go.microsoft.com/fwlink/?linkid=2142179), use este comando en la CLI de Azure para implementar el contenedor en el equipo host como un módulo de IoT Edge.

```azurecli
az login
az extension add --name azure-iot
az iot edge deployment create --deployment-id "<deployment name>" --hub-name "<IoT Hub name>" --content DeploymentManifest.json --target-condition "deviceId='<IoT Edge device name>'"--subscription "<subscriptionId>"
```

Rellene los parámetros obligatorios:

* Nombre de implementación: elija un nombre para esta implementación
* Nombre de la instancia de IoT Hub: nombre de la instancia de Azure IoT Hub
* Deployment.json: nombre del archivo de implementación
* Nombre de dispositivo IoT Edge: nombre del dispositivo IoT Edge del equipo host
* Suscripción: nombre o id. de la suscripción

Este comando iniciará la implementación y podrá ver el estado de la implementación en su instancia de Azure IoT Hub en Azure Portal. El estado se puede mostrar como *417: La configuración de implementación del dispositivo no está establecida* hasta que el dispositivo termine de descargar las imágenes de contenedor y empiece a ejecutarse.

### <a name="validate-that-the-deployment-was-successful"></a>Comprobación de que la implementación se realizó correctamente

Localice el *estado en tiempo de ejecución* en la Configuración de módulo IoT Edge del módulo de análisis espacial en la instancia de Azure IoT Hub en Azure Portal. El **valor deseado** y el **valor notificado** del *estado en tiempo de ejecución* debería indicar `Running`. A continuación se muestra el aspecto que tendrá en Azure Portal.

![Comprobación de implementación de ejemplo](./media/spatial-analysis/deployment-verification.png)

En este momento, el contenedor spatial-analysis ejecuta la operación. Emite conclusiones de IA para la operación `cognitiveservices.vision.spatialanalysis-personcount` y enruta estas conclusiones como telemetría a la instancia de Azure IoT Hub. Para configurar más cámaras, puede actualizar el archivo [manifiesto de implementación](https://go.microsoft.com/fwlink/?linkid=2142179) y volver a ejecutar la implementación.

## <a name="person-counting-web-application"></a>Aplicación web de recuento de personas

Esta aplicación web de recuento de personas permite configurar rápidamente una aplicación web de ejemplo y hospedarla en el entorno de Azure.

### <a name="get-the-person-counting-app-container"></a>Obtención del contenedor de la aplicación de recuento de personas

Hay disponible una forma de contenedor para esta aplicación en Azure Container Registry. Use el comando docker pull siguiente para descargarla. Póngase en contacto con Microsoft en projectarchon@microsoft.com para solicitar el token de acceso.

```bash
docker login rtvsofficial.azurecr.io -u <token name> -p <password>
docker pull rtvsofficial.azurecr.io/acceleratorapp.personcount:1.0
```

Inserte el contenedor en su instancia de Azure Container Registry (ACR).

```bash
az acr login --name <your ACR name>

docker tag rtvsofficial.azurecr.io/acceleratorapp.personcount:1.0 [desired local image name]

docker push [desired local image name]
```

Para instalar el contenedor, cree una nueva instancia de Azure Web App for Containers y rellene los parámetros obligatorios. Luego vaya a la pestaña **Docker**, seleccione **Contenedor único** y, luego, **Azure Container Registry**. Use la instancia de Azure Container Registry en la que insertó la imagen anterior.

![Ingreso de los detalles de la imagen](./media/spatial-analysis/solution-app-create-screen.png)

Después de escribir los parámetros anteriores, haga clic en **Revisar y crear** y cree la aplicación.

### <a name="configure-the-app"></a>Configuración de la aplicación 

Espere que se complete la configuración y vaya a su recurso en Azure Portal. Vaya a la sección **Configuración** y agregue las dos **configuraciones de la aplicación** siguientes.

* `EventHubConsumerGroup`: nombre de la cadena del grupo de consumidores de la instancia de Azure IoT Hub. Puede crear un grupo de consumidores nuevo en su instancia de IoT Hub o usar el grupo predeterminado. 
* `IotHubConnectionString`: cadena de conexión a la instancia de Azure IoT Hub, que se puede recuperar en la sección de claves del recurso de Azure IoT Hub ![Configurar los parámetros](./media/spatial-analysis/solution-app-config-page.png).

Una vez que se agreguen estas dos configuraciones, haga clic en **Guardar**. Luego, haga clic en la opción **Autenticación/autorización** en el menú de navegación de la izquierda y actualícela con el nivel de autenticación deseado. Se recomienda Azure Active Director (Azure AD) Express. 

### <a name="test-the-app"></a>Prueba de la aplicación

Vaya a la aplicación web de Azure y compruebe que la implementación se realizó correctamente y que la aplicación web está en ejecución. Vaya a la dirección URL configurada: `<yourapp>.azurewebsites.net` para ver la aplicación en ejecución.

![Prueba de la implementación](./media/spatial-analysis/solution-app-output.png)

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de operaciones de análisis espaciales](./spatial-analysis-operations.md)
* [Registro y solución de problemas](spatial-analysis-logging.md)
* [Guía de selección de ubicación de la cámara](spatial-analysis-camera-placement.md)
* [Guía de selección de ubicación de zonas y líneas](spatial-analysis-zone-line-placement.md)
