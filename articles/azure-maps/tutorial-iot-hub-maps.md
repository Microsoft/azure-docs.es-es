---
title: 'Tutorial: Implementación del análisis espacial de IoT con Microsoft Azure Maps'
description: Integre IoT Hub con las API del servicio Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/01/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 4150464b5c59b631afea0c788b1e351dee5185f9
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299411"
---
# <a name="tutorial-implement-iot-spatial-analytics-using-azure-maps"></a>Tutorial: Implementación del análisis espacial de IoT mediante Azure Maps

En un escenario de IoT, son típicos el seguimiento y la captura de los eventos pertinentes que se producen en el espacio y en el tiempo. Los escenarios de ejemplo incluyen aplicaciones de administración de flotas, seguimiento de activos, movilidad y ciudad inteligente. Este tutorial le guía por una solución que supervisa el movimiento de alquiler de automóviles usados mediante las API de Azure Maps.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Crear una cuenta de Azure Storage para registrar los datos de seguimiento de automóviles.
> * Cargar una geovalla en el servicio Data de Azure Maps con Data Upload API.
> * Crear una instancia de IoT Hub y registrar un dispositivo.
> * Crear una función en Azure Functions, para lo que debe implementar una lógica de negocios basada en el análisis espacial de Azure Maps.
> * Suscribirse a eventos de telemetría de dispositivos IoT desde la función de Azure a través de Event Grid.
> * Filtrar los eventos de telemetría mediante el enrutamiento de mensajes de IoT Hub.

## <a name="prerequisites"></a>Requisitos previos

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. [Cree una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).

3. [Obtenga una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción. Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](how-to-manage-authentication.md).

4. [Cree un grupo de recursos](https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal#create-resource-groups). En este tutorial, denominaremos el grupo de recursos *ContosoRental*, pero puede elegir el nombre que quiera.

5. Descargue el [proyecto de C# rentalCarSimulation](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/tree/master/src/rentalCarSimulation).

En este tutorial se usa la aplicación [Postman](https://www.postman.com/), pero puede elegir un entorno de desarrollo de API diferente.

## <a name="use-case-rental-car-tracking"></a>Caso de uso: seguimiento de automóviles de alquiler

Este tutorial muestra el siguiente escenario: Una empresa de alquiler de automóviles quiere registrar la información de ubicación, la distancia recorrida y el estado de ejecución de sus automóviles de alquiler. Además, la empresa quiere almacenar esta información cada vez que un automóvil abandone la región geográfica autorizada adecuada.

En nuestro caso, los vehículos de alquiler están equipados con dispositivos de IoT que envían datos de telemetría a Azure IoT Hub de forma periódica. Los datos de telemetría incluyen la ubicación actual e indican si el motor del vehículo está en funcionamiento. El esquema de ubicación del dispositivo cumple el [esquema IoT Plug and Play para datos geoespaciales](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v1-preview/schemas/geospatial.md). El esquema de los datos de telemetría del dispositivo del vehículo de alquiler tiene un aspecto similar al código JSON siguiente:

```JSON
{
    "data": {
        "properties": {
            "Engine": "ON"
        },
        "systemProperties": {
            "iothub-content-type": "application/json",
            "iothub-content-encoding": "utf-8",
            "iothub-connection-device-id": "ContosoRentalDevice",
            "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
            "iothub-connection-auth-generation-id": "636959817064335548",
            "iothub-enqueuedtime": "2019-06-18T00:17:20.608Z",
            "iothub-message-source": "Telemetry"
        },
        "body": {
            "location": {
                "type": "Point",
                "coordinates": [ -77.025988698005662, 38.9015330523316 ]
            }
        }
    }
}
```

En este tutorial, solo realizaremos el seguimiento de un vehículo. Después de configurar los servicios de Azure, deberá descargar el [proyecto rentalCarSimulation de C#](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/tree/master/src/rentalCarSimulation) para ejecutar el simulador de vehículos. El proceso completo, desde el evento hasta la ejecución de la función, se resume en los pasos siguientes:

1. El dispositivo en el vehículo envía datos de telemetría a IoT Hub.

2. Si el motor de automóviles se está ejecutando, el centro de IoT publica los datos de telemetría en Event Grid.

3. Una función de Azure se desencadena debido a su suscripción de eventos a los eventos de telemetría de dispositivo.

4. La función registrará las coordenadas de ubicación del dispositivo del vehículo, la hora del evento y el identificador del dispositivo. A continuación, usará [Spatial Geofence Get API](https://docs.microsoft.com/rest/api/maps/spatial/getgeofence) para determinar si el coche se ha conducido fuera de la geovalla. Si ha viajado fuera de los límites de la geovalla, la función almacena los datos de ubicación recibidos del evento en el contenedor de blobs. Además, la función consulta [Reverse Address Search](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse) para convertir la ubicación de la coordenada en una dirección postal y almacenarla con el resto de los datos de ubicación del dispositivo.

En el siguiente diagrama se proporciona información general del sistema.

   :::image type="content" source="./media/tutorial-iot-hub-maps/system-diagram.png" border="false" alt-text="Información general del sistema":::

La siguiente ilustración resalta el área de geovalla en azul. La ruta del vehículo de alquiler se indica mediante una línea verde.

   :::image type="content" source="./media/tutorial-iot-hub-maps/geofence-route.png" border="false" alt-text="Ruta de geovalla":::

## <a name="create-an-azure-storage-account"></a>Creación de una cuenta de Azure Storage

Para almacenar datos de seguimiento de infracciones de automóviles, crearemos una [cuenta de almacenamiento de uso general v2](https://docs.microsoft.com/azure/storage/common/storage-account-overview#general-purpose-v2-accounts) en el grupo de recursos. Si no ha creado un grupo de recursos, siga las instrucciones que se indican en [Crear grupos de recursos](https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal#create-resource-groups). En este tutorial, denominaremos el grupo de recursos *ContosoRental*.

Para crear una cuenta de almacenamiento, siga las instrucciones de [Creación de una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-account-create?tabs=azure-portal). En este tutorial, vamos a denominar la cuenta de almacenamiento *contosorentalstorage*, pero puede asignarle el nombre que quiera.

Una vez que la cuenta de almacenamiento se haya creado correctamente, será necesario crear un contenedor para almacenar los datos de registro.

1. Vaya a la cuenta de almacenamiento recién creada. Haga clic en el vínculo **Contenedores** en la sección Essentials.

    :::image type="content" source="./media/tutorial-iot-hub-maps/containers.png" alt-text="Contenedores de Blob Storage":::

2. Haga clic en el botón **+ Contenedor** en la esquina superior izquierda. Se mostrará un panel en el lado derecho del explorador. Denomine el contenedor *contoso-rental-logs* y haga clic en **Crear**.

     :::image type="content" source="./media/tutorial-iot-hub-maps/container-new.png" alt-text="Creación de un contenedor de blobs":::

3. Vaya a la hoja **Claves de acceso** de la cuenta de almacenamiento y copie el **nombre de la cuenta de almacenamiento** y el valor de la **clave** en la sección **key1**. Necesitaremos ambos valores en la sección [Creación de una función de Azure y adición de una suscripción a Event Grid](#create-an-azure-function-and-add-an-event-grid-subscription).

    :::image type="content" source="./media/tutorial-iot-hub-maps/access-keys.png" alt-text="Copia del nombre y la clave de la cuenta de almacenamiento":::

## <a name="upload-a-geofence"></a>Carga de una geovalla

Ahora usaremos la [aplicación Postman](https://www.getpostman.com) para [cargar la geovalla](https://docs.microsoft.com/azure/azure-maps/geofence-geojson) en el servicio Azure Maps. La geovalla define el área geográfica autorizada para nuestro vehículo de alquiler.  Usaremos la geovalla en nuestra función de Azure para determinar si un automóvil se ha movido fuera del área de la geovalla.

Abra la aplicación Postman y siga estos pasos para cargar la geovalla mediante Data Upload API de Azure Maps.  

1. Abra la aplicación Postman. Cerca de la parte superior de la aplicación Postman, seleccione**New** (Nuevo). En la ventana **Create New** (Crear nuevo), seleccione **Collection** (Colección).  Asigne un nombre a la colección y seleccione el botón **Create** (Crear).

2. Para crear la solicitud, seleccione **New** (Nuevo) otra vez. En la ventana **Create New** (Crear nuevo), seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para la solicitud. Seleccione la colección que creó en el paso anterior y haga clic en **Save** (Guardar).

3. Seleccione el método HTTP **POST** en la pestaña del generador y escriba la siguiente dirección URL para cargar la geovalla en Data Upload API. Asegúrese de reemplazar `{subscription-key}` por la clave de suscripción principal.

    ```HTTP
    https://atlas.microsoft.com/mapData/upload?subscription-key={subscription-key}&api-version=1.0&dataFormat=geojson
    ```

    El valor "geojson" del parámetro `dataFormat` de la ruta de acceso de la dirección URL representa el formato de datos de los datos que se cargan.

4. Haga clic en **Cuerpo**, seleccione el formato de entrada **sin procesar** y elija **JSON** como formato de entrada en la lista desplegable. Abra el archivo de datos JSON [aquí](https://raw.githubusercontent.com/Azure-Samples/iothub-to-azure-maps-geofencing/master/src/Data/geofence.json?token=AKD25BYJYKDJBJ55PT62N4C5LRNN4) y copie el JSON en la sección del cuerpo. Haga clic en **Enviar**.

5. Haga clic en el botón azul **Send** (Enviar) y espere a que se procese la solicitud. Una vez finalizada la solicitud, vaya a la pestaña **Headers** (Encabezados) de la respuesta. Copie el valor de la clave **Location** (Ubicación), que es `status URL`.

    ```http
    https://atlas.microsoft.com/mapData/operations/<operationId>?api-version=1.0
    ```

6. Para comprobar el estado de la llamada de API, cree una solicitud HTTP **GET** en el elemento `status URL`. Tendrá que anexar la clave de suscripción principal a la dirección URL para realizar la autenticación. La solicitud **GET** debe ser como la siguiente dirección URL:

   ```HTTP
   https://atlas.microsoft.com/mapData/<operationId>/status?api-version=1.0&subscription-key={subscription-key}

7. When the **GET** HTTP request completes successfully, it will return a `resourceLocation`. The `resourceLocation` contains the unique `udid` for the uploaded content. You'll need to copy this `udid` for later use in this tutorial.

      ```json
      {
          "status": "Succeeded",
          "resourceLocation": "https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0"
      }
      ```

## <a name="create-an-azure-iot-hub"></a>Crear un centro de IoT con Azure IoT Hub

Azure IoT Hub permite una comunicación bidireccional confiable y segura entre una aplicación de IoT y los dispositivos que esta administra.  En nuestro escenario, queremos obtener información del dispositivo en el vehículo para determinar la ubicación del coche de alquiler. En esta sección, vamos a crear un centro de IoT en el grupo de recursos *ContosoRental*. El centro de IoT será responsable de la publicación de los eventos de telemetría de dispositivos.

> [!NOTE]
> La funcionalidad del centro de IoT para publicar eventos de telemetría de dispositivos en Event Grid está en versión preliminar pública. Las características en versión preliminar pública están disponibles en todas las regiones, excepto en **Este de EE. UU., Oeste de EE. UU., Oeste de Europa, Azure Government, Azure China 21Vianet** y **Azure Alemania**.

Para crear un centro de IoT en el grupo de recursos *ContosoRental*, siga los pasos descritos en [Crear un centro de IoT](https://docs.microsoft.com/azure/iot-hub/quickstart-send-telemetry-dotnet#create-an-iot-hub).

## <a name="register-a-device-in-iot-hub"></a>Registro de un dispositivo en el centro de IoT

Los dispositivos no pueden conectarse al centro de IoT a menos que estén registrados en el registro de identidades del centro de IoT. En nuestro escenario, vamos a crear un único dispositivo denominado *InVehicleDevice*. Para crear y registrar el dispositivo en el centro de IoT, siga los pasos descritos en [Registro de un nuevo dispositivo en el centro de IoT](https://docs.microsoft.com/azure/iot-hub/iot-hub-create-through-portal#register-a-new-device-in-the-iot-hub). Asegúrese de copiar la **cadena de conexión principal** del dispositivo, ya que la usaremos en un paso posterior.

## <a name="create-an-azure-function-and-add-an-event-grid-subscription"></a>Creación de una función de Azure y adición de una suscripción a Event Grid

Azure Functions es un servicio de proceso sin servidor que permite ejecutar pequeñas piezas de código ("funciones") sin necesidad de aprovisionar ni administrar explícitamente la infraestructura de proceso. Para obtener más información sobre Azure Functions, vea la documentación de [Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-overview).

Un determinado evento "desencadena" una función. En nuestro escenario, vamos a crear una función desencadenada por un desencadenador de Event Grid. Creamos la relación entre el desencadenador y la función mediante la creación de una suscripción de eventos para eventos de telemetría de dispositivos del centro de IoT. Cuando se produce un evento de telemetría de dispositivo, la función se llamará como punto de conexión y recibirá los datos pertinentes para el [dispositivo que se registró anteriormente en el centro de IoT](#register-a-device-in-iot-hub).

El código de script de C# que contendrá la función puede verse [aquí](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/blob/master/src/Azure%20Function/run.csx).

Ahora, configuraremos nuestra función de Azure.

1. En el panel de Azure Portal, seleccione **Crear un recurso**. Escriba **Function App** en el cuadro de búsqueda. Haga clic en **Function App**. Haga clic en **Crear**.

2. En la página de creación de la **Aplicación de funciones**, asigne un nombre a la aplicación de funciones. En **Grupo de recursos**, seleccione *ContosoRental* en la lista desplegable.  Seleccione *.NET Core* como la **pila en tiempo de ejecución**. Haga clic en **Siguiente: Hosting >** en la parte inferior de la página.

    :::image type="content" source="./media/tutorial-iot-hub-maps/rental-app.png" alt-text="Creación de una aplicación de función":::

3. En **Cuenta de almacenamiento**, seleccione la cuenta de almacenamiento que creó en [Crear una cuenta de almacenamiento](#create-an-azure-storage-account). Haga clic en **Revisar + crear**.

4. Revise los detalles de la aplicación de funciones y seleccione **Crear**.

5. Una vez creada la aplicación, es preciso agregarle una función. Vaya a la aplicación de función. Haga clic en la hoja **Funciones**. Haga clic en **+ Agregar** en la parte superior de la página. Se mostrará el panel de plantillas de función. Desplácese hacia abajo en el panel. Haga clic en **Desencadenador de Azure Event Grid**.

     >[!WARNING]
    > Las plantillas del **desencadenador de Azure Event Hub** y el **desencadenador de Azure Event Grid** tienen nombres similares. Asegúrese de hacer clic en la plantilla del **desencadenador de Azure Event Grid**.

    :::image type="content" source="./media/tutorial-iot-hub-maps/function-create.png" alt-text="Creación de una función":::

6. Asigne un nombre a la función. En este tutorial, usaremos el nombre *GetGeoFunction*, pero puede usar el que quiera. Haga clic en **Crear función**.

7. Haga clic en la hoja **Código y prueba** en el menú de la izquierda. Copie y pegue el [script de C#](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/blob/master/src/Azure%20Function/run.csx) en la ventana de código.

     :::image type="content" source="./media/tutorial-iot-hub-maps/function-code.png" alt-text="Ventana Copiar y pegar código en la función":::

8. En el código de C#, reemplace los siguientes parámetros. Haga clic en **Save**(Guardar). No haga clic en **Probar o ejecutar** todavía.
    * Reemplace **SUBSCRIPTION_KEY** por la clave de suscripción principal de su cuenta de Azure Maps.
    * Reemplace **UDID** por el `udid` de la geovalla que cargó en [Carga de geovalla](#upload-a-geofence).
    * La función **CreateBlobAsync** del script crea un blob por evento en la cuenta de almacenamiento de datos. Reemplace **ACCESS_KEY**, **ACCOUNT_NAME** y **STORAGE_CONTAINER_NAME** por la clave de acceso de la cuenta de almacenamiento, el nombre de la cuenta y el contenedor de almacenamiento de datos. Estos valores se generaron al crear la cuenta de almacenamiento en [Crear una cuenta de almacenamiento de Azure](#create-an-azure-storage-account).

9. Haga clic en la hoja **Integración** en el menú de la izquierda. Haga clic en **Desencadenador de Event Grid** en el diagrama. Escriba un nombre para el desencadenador, como *eventCarTelemetry*, y haga clic en **Crear suscripción de Event Grid**.

     :::image type="content" source="./media/tutorial-iot-hub-maps/function-integration.png" alt-text="Agregar una suscripción a evento":::

10. Rellene los detalles de la suscripción. Asigne un nombre a la suscripción del evento. En *Esquema de eventos*, seleccione *Esquema de Event Grid*. En **Tipos de tema**, seleccione *Cuentas de Azure IoT Hub*. En **Grupos de recursos**, seleccione el grupo de recursos que creó al principio de este tutorial. En **Recurso**, seleccione el centro de IoT que creó en [Creación de un IoT Hub](#create-an-azure-iot-hub). En **Filtro para tipos de evento**, seleccione *Telemetría del dispositivo*. Después de elegir estas opciones, verá que **Tipo de tema** cambia a *IoT Hub*. En **Nombre del tema del sistema**, puede usar el mismo nombre del recurso.  Por último, haga clic en **Seleccionar un punto de conexión** en la sección **Detalles del punto de conexión**. Acepte toda la configuración y haga clic en **Confirmar selección**.

    :::image type="content" source="./media/tutorial-iot-hub-maps/function-create-event-subscription.png" alt-text="Creación de la suscripción de eventos":::

11. Revise la configuración. Asegúrese de que el punto de conexión especifica la función que creó al principio de esta sección. Haga clic en **Crear**.

    :::image type="content" source="./media/tutorial-iot-hub-maps/function-create-event-subscription-confirm.png" alt-text="Confirmación de la creación de una suscripción de eventos":::

12. Ahora vuelva al panel **Editar desencadenador**. Haga clic en **Save**(Guardar).

## <a name="filter-events-using-iot-hub-message-routing"></a>Filtrado de eventos mediante el enrutamiento de mensajes de IoT Hub

Cuando se agrega una suscripción de Event Grid a la función de Azure, se crea automáticamente una ruta de mensajería en el centro IoT especificado. El enrutamiento de mensajes le permite enrutar distintos tipos de datos a varios puntos de conexión. Por ejemplo, puede enrutar los mensajes de telemetría del dispositivo, los eventos del ciclo de vida del dispositivo y los eventos de cambio de dispositivo gemelo. Para más información acerca del enrutamiento de mensajes en IoT Hub, consulte [Uso del enrutamiento de mensajes de IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-d2c).

:::image type="content" source="./media/tutorial-iot-hub-maps/hub-route.png" alt-text="Enrutamiento de mensajes en el centro de IoT":::

En el escenario de ejemplo, solo queremos recibir mensajes cuando el coche de alquiler se mueve. Vamos a crear una consulta de enrutamiento para filtrar los eventos en los que la propiedad `Engine` es igual a **"ON"** . Para crear una consulta de enrutamiento, haga clic en la ruta **RouteToEventGrid**, reemplace la **consulta de enrutamiento** por **"Engine='ON'"** y haga clic en **Guardar**. Ahora, IoT Hub solo publicará los datos de telemetría del dispositivo cuando el valor de Engine sea ON.

:::image type="content" source="./media/tutorial-iot-hub-maps/hub-filter.png" alt-text="Filtro de mensajes de enrutamiento":::

>[!TIP]
>Hay varias maneras de consultar los mensajes del dispositivo IoT a la nube. Para más información acerca de la sintaxis de enrutamiento de mensajes, consulte el artículo acerca del [enrutamiento de mensajes de IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-routing-query-syntax).

## <a name="send-telemetry-data-to-iot-hub"></a>Envío de datos de telemetría a IoT Hub

Una vez que la función de Azure esté en funcionamiento, se pueden enviar los datos de telemetría al centro de IoT, que los enrutará a Event Grid. Usaremos una aplicación creada en C# para simular los datos de ubicación de un dispositivo situado en un automóvil de alquiler. Para ejecutar la aplicación, se necesita el SDK de .NET Core 2.1.0, o una versión posterior, en el equipo de desarrollo. Siga los pasos que se indican a continuación para enviar datos de telemetría simulados a IoT Hub.

1. Si todavía no lo ha hecho, descargue el proyecto [rentalCarSimulation](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/tree/master/src/rentalCarSimulation) de C#.

2. Abra el archivo simulatedCar.CS en el editor de texto que prefiera, reemplace el valor de `connectionString` por el que guardó cuando registró el dispositivo y guarde los cambios en el archivo.

3. Asegúrese de que tiene .NET Core instalado en la máquina. En la ventana del terminal local, vaya a la carpeta raíz del proyecto de C# y ejecute el siguiente comando para instalar los paquetes necesarios para la aplicación de dispositivo simulado:

    ```cmd/sh
    dotnet restore
    ```

4. En el mismo terminal, ejecute el comando siguiente para compilar la aplicación de simulación de automóvil de alquiler y ejecutarla:

    ```cmd/sh
    dotnet run
    ```

  El terminal local tendrá el aspecto siguiente.

:::image type="content" source="./media/tutorial-iot-hub-maps/terminal.png" alt-text="Salida del terminal":::

Si abre el contenedor de almacenamiento de blobs ahora, puede ver cuatro blobs, que corresponden a las ubicaciones en las que el vehículo estuvo fuera de la geovalla.

:::image type="content" source="./media/tutorial-iot-hub-maps/blob.png" alt-text="Visualización de los blobs de un contenedor":::

En el mapa siguiente se muestran cuatro puntos de ubicación del vehículo fuera de la geovalla. Cada ubicación se registró en intervalos de tiempo regulares.

:::image type="content" source="./media/tutorial-iot-hub-maps/violation-map.png" alt-text="Mapa de la infracción":::

## <a name="explore-azure-maps-and-iot"></a>Explorar Azure Maps e IoT

Para explorar las API de Azure Maps que se usan en este tutorial, consulte:

* [Obtener la dirección de búsqueda inversa](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse)
* [Get Geofence](https://docs.microsoft.com/rest/api/maps/spatial/getgeofence)

Para obtener una lista completa de las API REST de Azure Maps, consulte:

* [API REST de Azure Maps](https://docs.microsoft.com/rest/api/maps/spatial/getgeofence)

Para más información acerca de IoT Plug and Play, consulte:

* [IoT Plug and Play](https://docs.microsoft.com/azure/iot-pnp)

Para obtener una lista de los dispositivos que tienen la certificación de Azure para IoT, visite:

* [Dispositivos certificados de Azure](https://catalog.azureiotsolutions.com/)

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de cómo enviar datos de telemetría del dispositivo a la nube y viceversa, consulte:

> [!div class="nextstepaction"]
> [Envío de datos de telemetría desde un dispositivo](https://docs.microsoft.com/azure/iot-hub/quickstart-send-telemetry-dotnet)
