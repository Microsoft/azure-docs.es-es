---
title: 'Reaccionar a eventos del módulo de Blob Storage: Azure Event Grid IoT Edge | Microsoft Docs'
description: Reacción a eventos de módulo de Blob Storage
author: arduppal
manager: brymat
ms.author: arduppal
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 230e158a970f8c815b1575403c013e30749124c5
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87462027"
---
# <a name="tutorial-react-to-blob-storage-events-on-iot-edge-preview"></a>Tutorial: Reacción a eventos de Blob Storage en IoT Edge (versión preliminar)
En este artículo se muestra cómo implementar Azure Blob Storage en el módulo IoT, que actuaría como publicador Event Grid para enviar eventos en la creación de blobs y la eliminación de blobs a Event Grid.  

Para obtener información general sobre Azure Blob Storage en IoT Edge, vea [Azure Blob Storage en IoT Edge](../../iot-edge/how-to-store-data-blob.md) y sus características.

> [!WARNING]
> Integración de Azure Blob Storage en IoT Edge con Event Grid se encuentra en versión preliminar

Para realizar este tutorial, necesitará lo siguiente:

* Una **suscripción a Azure**: cree una [cuenta gratuita](https://azure.microsoft.com/free) si aún no tiene una. 
* **Dispositivo Azure IoT Hub e IoT Edge**: siga los pasos del inicio rápido para dispositivos [Linux](../../iot-edge/quickstart-linux.md) o [Windows](../../iot-edge/quickstart.md) si aún no tiene uno.

## <a name="deploy-event-grid-iot-edge-module"></a>Implementación del módulo de Event Grid en IoT Edge

Hay varias maneras de implementar módulos en un dispositivo IoT Edge, y todas ellas funcionan con Azure Event Grid en IoT Edge. En este artículo se describen los pasos para implementar Event Grid en IoT Edge desde Azure Portal.

>[!NOTE]
> En este tutorial, implementará el módulo de Event Grid sin persistencia. Esto significa que los temas y las suscripciones que cree en este tutorial se eliminarán al volver a implementar el módulo. Para más información sobre cómo configurar la persistencia, consulte los artículos siguientes: [Persistencia del estado en Linux](persist-state-linux.md) o [Persistencia del estado en Windows](persist-state-windows.md). En el caso de las cargas de trabajo de producción, se recomienda instalar el módulo de Event Grid con persistencia.


### <a name="select-your-iot-edge-device"></a>Selección del dispositivo IoT Edge

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com)
1. Vaya a su instancia de IoT Hub.
1. Seleccione **IoT Edge** en el menú de la sección **Administración automática de dispositivos**. 
1. Haga clic en el identificador del dispositivo de destino en la lista de dispositivos.
1. Seleccione **Set modules** (Establecer módulos). Mantenga la página abierta. Continuará con los pasos de la siguiente sección.

### <a name="configure-a-deployment-manifest"></a>Configuración de un manifiesto de implementación

Un manifiesto de implementación es un documento JSON que describe qué módulos se van a implementar, cómo fluyen los datos entre los módulos y las propiedades deseadas de los módulos gemelos. Azure Portal tiene un asistente que le guía en la creación de un manifiesto de implementación, en lugar de crear el documento JSON de forma manual.  Consta de tres pasos: **Adición de módulos**, **Especificación de rutas** y **Revisión de la implementación**.

### <a name="add-modules"></a>Adición de módulos

1. En la sección **Módulos de implementación**, seleccione **Agregar**.
1. En los tipos de módulos de la lista desplegable, seleccione **Módulo IoT Edge**
1. Proporcione el nombre, la imagen y las opciones de creación del contenedor:

   * **Nombre**: eventgridmodule
   * **URI de imagen**: `mcr.microsoft.com/azure-event-grid/iotedge:latest`
   * **Opciones de creación del contenedor**:

    ```json
        {
          "Env": [
           "inbound__serverAuth__tlsPolicy=enabled",
           "inbound__clientAuth__clientCert__enabled=false"
          ],
          "HostConfig": {
            "PortBindings": {
              "4438/tcp": [
                {
                    "HostPort": "4438"
                }
              ]
            }
          }
        }
    ```    

 1. Haga clic en **Guardar**
 1. Continúe con la sección siguiente para agregar el módulo de suscriptor de Azure Event Grid antes de implementarlos juntos.

    >[!IMPORTANT]
    > En este tutorial aprenderá a implementar el módulo de Event Grid para permitir solicitudes HTTP y HTTPS, con la autenticación de cliente deshabilitada. En el caso de las cargas de trabajo de producción, se recomienda habilitar solo las solicitudes y los suscriptores HTTPS con la autenticación de cliente habilitada. Para más información sobre cómo configurar el módulo de Event Grid de forma segura, consulte [Seguridad y autenticación](security-authentication.md).
    

## <a name="deploy-event-grid-subscriber-iot-edge-module"></a>Implementación del módulo IoT Edge de suscriptor de Event Grid

En esta sección se muestra cómo implementar otro módulo de IoT que funcionará como controlador de eventos al que se pueden entregar eventos.

### <a name="add-modules"></a>Adición de módulos

1. En la sección **Módulos de implementación**, seleccione de nuevo **Agregar**. 
1. En los tipos de módulos de la lista desplegable, seleccione **Módulo IoT Edge**
1. Proporcione el nombre, la imagen y las opciones de creación del contenedor:

   * **Nombre**: suscriptor
   * **URI de imagen**: `mcr.microsoft.com/azure-event-grid/iotedge-samplesubscriber:latest`
   * **Opciones de creación del contenedor**: None
1. Haga clic en **Guardar**
1. Continúe a la sección siguiente para agregar el módulo Azure Blob Storage

## <a name="deploy-azure-blob-storage-module"></a>Implementar módulo de Azure Blob Storage

En esta sección se muestra cómo implementar el módulo de Azure Blob Storage, que actuaría como un publicador de Event Grid que publica blobs creados y eventos eliminados.

### <a name="add-modules"></a>Adición de módulos

1. En la sección **Módulos de implementación**, seleccione **Agregar**.
2. En los tipos de módulos de la lista desplegable, seleccione **Módulo IoT Edge**
3. Proporcione el nombre, la imagen y las opciones de creación del contenedor:

   * **Nombre**: azureblobstorageoniotedge
   * **URI de la imagen**: mcr.microsoft.com/azure-blob-storage:latest
   * **Opciones de creación del contenedor**:

   ```json
       {
         "Env":[
           "LOCAL_STORAGE_ACCOUNT_NAME=<your storage account name>",
           "LOCAL_STORAGE_ACCOUNT_KEY=<your storage account key>",
           "EVENTGRID_ENDPOINT=http://<event grid module name>:5888"
         ],
         "HostConfig":{
           "Binds":[
               "<storage mount>"
           ],
           "PortBindings":{
             "11002/tcp":[{"HostPort":"11002"}]
           }
         }
       }
   ```

   > [!IMPORTANT]
   > - El módulo de Blob Storage puede publicar eventos mediante HTTPS y HTTP. 
   > - Si ha habilitado la autenticación basada en el cliente para Event Grid, asegúrese de actualizar el valor de EVENTGRID_ENDPOINT para permitir https de la siguiente manera: `EVENTGRID_ENDPOINT=https://<event grid module name>:4438`.
   > - Agregue también otra variable de entorno `AllowUnknownCertificateAuthority=true` al JSON anterior. Al comunicarse con Event Grid mediante HTTPS, **AllowUnknownCertificateAuthority** permite que el módulo de almacenamiento confíe en certificados de servidor de Event Grid autofirmados.

4. Actualice el archivo JSON que ha copiado con la siguiente información:

   - Reemplace `<your storage account name>` por un nombre que pueda recordar. Los nombres de cuenta tener de 3 a 24 caracteres, con letras en minúscula y números. No deben tener espacios.

   - Actualice `<your storage account key>` por una clave base64 de 64 bytes. Puede generar una clave con herramientas como [GeneratePlus](https://generate.plus/en/base64?gp_base64_base[length]=64). Estas credenciales las usará para acceder a Blob Storage desde otros módulos.

   - Reemplace `<event grid module name>` por el nombre del módulo de Event Grid.
   - Reemplace `<storage mount>` según el sistema operativo del contenedor.
     - En el caso de los contenedores de Linux, **my-volume:/blobroot**
     - En el caso de los contenedores de Windows,**my-volume:C:/BlobRoot**

5. Haga clic en **Guardar**
6. Seleccione **Siguiente** para ir a la sección de rutas

    > [!NOTE]
    > Si va a usar una máquina virtual de Azure como dispositivo perimetral, agregue una regla de puerto de entrada para permitir tráfico entrante en el puerto host usado en este tutorial: 4438, 5888, 8080 y 11002. Para instrucciones sobre cómo agregar la regla, consulte [Apertura de puertos en una máquina virtual](../../virtual-machines/windows/nsg-quickstart-portal.md).

### <a name="setup-routes"></a>Configuración de rutas

Mantenga las rutas predeterminadas y seleccione **Siguiente** para pasar a la sección de revisión.

### <a name="review-deployment"></a>Revisión de la implementación

1. La sección de revisión le muestra el manifiesto de implementación en formato JSON que se ha creado en función de las selecciones de la sección anterior. Confirme que ve los cuatro módulos siguientes: **$edgeAgent**, **$edgeHub**, **eventgridmodule**, **suscriptor** y **azureblobstorageoniotedge** que se están implementando.
2. Revise la información de implementación y seleccione **Enviar**.

## <a name="verify-your-deployment"></a>Comprobación de la implementación

1. Después de enviar la implementación, regresará a la página de IoT Edge del centro de IoT.
2. Seleccione el **dispositivo IoT Edge** elegido como destino con la implementación para abrir sus detalles.
3. En los detalles del dispositivo, compruebe que los módulos eventgridmodule, suscriptor y azureblobstorageoniotedge aparecen como **Especificado en la implementación** y **Notificado por el dispositivo**.

   Puede tardar unos minutos para que el módulo se inicie en el dispositivo y, a continuación, notifique a IoT Hub. Actualice la página para ver el estado actualizado.

## <a name="publish-blobcreated-and-blobdeleted-events"></a>Publicación de eventos BlobCreated y BlobDeleted

1. Este módulo crea automáticamente el tema **MicrosoftStorage**. Compruebe que sí existe
    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage?api-version=2019-01-01-preview
    ```

    Salida del ejemplo:

    ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/MicrosoftStorage",
            "name": "MicrosoftStorage",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/MicrosoftStorage/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
    ```

    > [!IMPORTANT]
    > - En el flujo HTTPS, si la autenticación del cliente se habilita mediante clave SAS, la clave SAS especificada anteriormente debe agregarse como encabezado. Por lo tanto, la solicitud curl será: `curl -k -H "Content-Type: application/json" -H "aeg-sas-key: <your SAS key>" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage?api-version=2019-01-01-preview`
    > - En el flujo de HTTPS, si la autenticación del cliente está habilitada mediante certificado, la solicitud curl será: `curl -k -H "Content-Type: application/json" --cert <certificate file> --key <certificate private key file> -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage?api-version=2019-01-01-preview`

2. Los suscriptores pueden registrarse en eventos publicados en un tema. Para recibir cualquier evento, deberá crear una suscripción de Event Grid para un tema de **MicrosoftStorage**.
    1. Cree el archivo blobsubscription.json con el siguiente contenido. Para más información sobre la carga, consulte la [Documentación de API](api.md)

       ```json
        {
          "properties": {
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "https://subscriber:4430"
              }
            }
          }
        }
       ```

       >[!NOTE]
       > La propiedad **endpointType** especifica que el suscriptor es un **webhook**.  La propiedad **endpointUrl** especifica la dirección URL en la que el suscriptor escucha los eventos. Esta dirección URL corresponde al ejemplo de Azure Functions que implementó anteriormente.

    2. Ejecute el siguiente comando para crear una suscripción para el tema. Confirme que se muestra el código de estado HTTP `200 OK`.

       ```sh
       curl -k -H "Content-Type: application/json" -X PUT -g -d @blobsubscription.json https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview
       ```

       > [!IMPORTANT]
       > - En el flujo HTTPS, si la autenticación del cliente se habilita mediante clave SAS, la clave SAS especificada anteriormente debe agregarse como encabezado. Por lo tanto, la solicitud curl será: `curl -k -H "Content-Type: application/json" -H "aeg-sas-key: <your SAS key>" -X PUT -g -d @blobsubscription.json https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview` 
       > - En el flujo de HTTPS, si la autenticación del cliente está habilitada mediante certificado, la solicitud curl será: `curl -k -H "Content-Type: application/json" --cert <certificate file> --key <certificate private key file> -X PUT -g -d @blobsubscription.json https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview`

    3. Ejecute el siguiente comando para comprobar que la suscripción se ha creado correctamente. Se devolverá el código de estado HTTP "200 - Correcto".

       ```sh
       curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview
       ```

       Salida del ejemplo:

       ```json
        {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription5",
          "properties": {
            "Topic": "MicrosoftStorage",
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "https://subscriber:4430"
              }
            }
          }
        }
       ```

       > [!IMPORTANT]
       > - En el flujo HTTPS, si la autenticación del cliente se habilita mediante clave SAS, la clave SAS especificada anteriormente debe agregarse como encabezado. Por lo tanto, la solicitud curl será: `curl -k -H "Content-Type: application/json" -H "aeg-sas-key: <your SAS key>" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview`
       > - En el flujo de HTTPS, si la autenticación del cliente está habilitada mediante certificado, la solicitud curl será: `curl -k -H "Content-Type: application/json" --cert <certificate file> --key <certificate private key file> -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview`

3. Descargue [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) y [conéctelo al almacenamiento local](../../iot-edge/how-to-store-data-blob.md#connect-to-your-local-storage-with-azure-storage-explorer)

## <a name="verify-event-delivery"></a>Comprobación de la entrega del evento

### <a name="verify-blobcreated-event-delivery"></a>Comprobar la entrega de eventos de BlobCreated

1. Cargue los archivos como blobs en bloques en el almacenamiento local desde Explorador de Azure Storage y el módulo publicará automáticamente los eventos de creación. 
2. Consulte los registros del suscriptor para crear el evento. Siga los pasos para [comprobar la entrega de eventos](pub-sub-events-webhook-local.md#verify-event-delivery)

    Resultado de ejemplo:

    ```json
            Received Event:
            {
              "id": "d278f2aa-2558-41aa-816b-e6d8cc8fa140",
              "topic": "MicrosoftStorage",
              "subject": "/blobServices/default/containers/cont1/blobs/Team.jpg",
              "eventType": "Microsoft.Storage.BlobCreated",
              "eventTime": "2019-10-01T21:35:17.7219554Z",
              "dataVersion": "1.0",
              "metadataVersion": "1",
              "data": {
                "api": "PutBlob",
                "clientRequestId": "00000000-0000-0000-0000-000000000000",
                "requestId": "ef1c387b-4c3c-4ac0-8e04-ff73c859bfdc",
                "eTag": "0x8D746B740DA21FB",
                "url": "http://azureblobstorageoniotedge:11002/myaccount/cont1/Team.jpg",
                "contentType": "image/jpeg",
                "contentLength": 858129,
                "blobType": "BlockBlob"
              }
            }
    ```

### <a name="verify-blobdeleted-event-delivery"></a>Comprobar la entrega de eventos de BlobDeleted

1. Elimine los blobs del almacenamiento local desde el Explorador de Azure Storage; el módulo publicará automáticamente los eventos de eliminación. 
2. Consulte los registros del suscriptor para eliminar el evento. Siga los pasos para [comprobar la entrega de eventos](pub-sub-events-webhook-local.md#verify-event-delivery)

    Resultado de ejemplo:
    
    ```json
            Received Event:
            {
              "id": "ac669b6f-8b0a-41f3-a6be-812a3ce6ac6d",
              "topic": "MicrosoftStorage",
              "subject": "/blobServices/default/containers/cont1/blobs/Team.jpg",
              "eventType": "Microsoft.Storage.BlobDeleted",
              "eventTime": "2019-10-01T21:36:09.2562941Z",
              "dataVersion": "1.0",
              "metadataVersion": "1",
              "data": {
                "api": "DeleteBlob",
                "clientRequestId": "00000000-0000-0000-0000-000000000000",
                "requestId": "2996bbfb-c819-4d02-92b1-c468cc67d8c6",
                "eTag": "0x8D746B740DA21FB",
                "url": "http://azureblobstorageoniotedge:11002/myaccount/cont1/Team.jpg",
                "contentType": "image/jpeg",
                "contentLength": 858129,
                "blobType": "BlockBlob"
              }
            }
    ```

Felicidades. Ha completado el tutorial. En las secciones siguientes se proporcionan detalles sobre las propiedades del evento.

### <a name="event-properties"></a>Propiedades de evento

Esta es la lista de propiedades de evento admitidas y sus tipos y descripciones. 

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| topic | string | Ruta de acceso completa a los recursos del origen del evento. En este campo no se puede escribir. Event Grid proporciona este valor. |
| subject | string | Ruta al asunto del evento definida por el anunciante. |
| eventType | string | Uno de los tipos de eventos registrados para este origen de eventos. |
| eventTime | string | La hora de generación del evento en función de la hora UTC del proveedor. |
| id | string | Identificador único para el evento |
| datos | object | Datos de eventos de Blob Storage. |
| dataVersion | string | Versión del esquema del objeto de datos. El publicador define la versión del esquema. |
| metadataVersion | string | Versión del esquema de los metadatos del evento. Event Grid define el esquema de las propiedades de nivel superior. Event Grid proporciona este valor. |

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| api | string | Operación que desencadenó el evento. Puede ser uno de los siguientes valores: <ul><li>BlobCreated: los valores permitidos son: `PutBlob` y `PutBlockList`</li><li>BlobDeleted: los valores permitidos son: `DeleteBlob`, `DeleteAfterUpload` y `AutoDelete`. <p>El evento `DeleteAfterUpload` se genera cuando se elimina el blob automáticamente porque la propiedad deseada deleteAfterUpload está establecida en "true". </p><p>El evento `AutoDelete` se genera cuando se elimina el blob automáticamente porque expiró el valor de la propiedad deseada deleteAfterMinutes.</p></li></ul>|
| clientRequestId | string | Id. de solicitud que proporciona el cliente para la operación de la API de Storage. Dicho id. se puede usar para establecer la correlación con los registros de diagnóstico de Azure Storage que usan el campo "client-request-id" en los registros y se puede proporcionar en las solicitudes de los clientes que usan el encabezado "x-ms-client-request-id". Para más detalles, vea [Formato de registro](/rest/api/storageservices/storage-analytics-log-format). |
| requestId | string | Id. de solicitud generado por el servicio para la operación de la API de Storage. Se puede usar para establecer la correlación con los registros de diagnóstico de Azure Storage que usan el campo "request-id-header" en los registros y se devuelve cuando se inicia la llamada API en el encabezado "x-ms-request-id". Consulte [Storage Analytics Log Format](/rest/api/storageservices/storage-analytics-log-format) (Formato de registro de Storage Analytics). |
| eTag | string | Valor que puede usar para realizar operaciones de manera condicional. |
| contentType | string | Tipo de contenido especificado para el blob. |
| contentLength | integer | Tamaño del blob en bytes. |
| blobType | string | El tipo de blob. Los valores válidos son "BlockBlob" o "PageBlob". |
| url | string | Ruta de acceso al blob. <br>Si el cliente usa una API de REST de Blob, la dirección URL tiene esta estructura: *\<storage-account-name\>.blob.core.windows.net/\<container-name\>/\<file-name\>* . <br>Si el cliente usa una API de REST de Data Lake Storage, la dirección URL tiene esta estructura: *\<storage-account-name\>.dfs.core.windows.net/\<file-system-name\>/\<file-name\>* . |


## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos de la documentación de Blob Storage:

- [Filtración de los eventos de Blob Storage](../../storage/blobs/storage-blob-event-overview.md#filtering-events)
- [Prácticas recomendadas para consumir eventos de Blob Storage](../../storage/blobs/storage-blob-event-overview.md#practices-for-consuming-events)

En este tutorial, ha publicado eventos mediante la creación o la eliminación de blobs en un Azure Blob Storage. Consulte los otros tutoriales para aprender a reenviar eventos a la nube (Azure Event Hub o Azure IoT Hub): 

- [Reenvío de eventos a Azure Event Grid](forward-events-event-grid-cloud.md)
- [Reenvío de eventos a Azure IoT Hub](forward-events-iothub.md)
