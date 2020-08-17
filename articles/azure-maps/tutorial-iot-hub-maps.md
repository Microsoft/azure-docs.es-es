---
title: 'Tutorial: Implementación del análisis espacial de IoT | Microsoft Azure Maps'
description: Integre IoT Hub con las API del servicio Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 2bb5876424730e55d15cc52aeb98aa04af040821
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87133408"
---
# <a name="tutorial-implement-iot-spatial-analytics-using-azure-maps"></a>Tutorial: Implementación del análisis espacial de IoT mediante Azure Maps

En un escenario de IoT, son típicos el seguimiento y la captura de los eventos pertinentes que se producen en el espacio y en el tiempo. Los escenarios de ejemplo incluyen aplicaciones de administración de flotas, seguimiento de activos, movilidad y ciudad inteligente. Este tutorial le guía por un patrón de soluciones para usar las API de Azure Maps. IoT Hub captura los eventos correspondientes utilizando el modelo de suscripción de eventos proporcionado por Event Grid.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Cree un centro de IoT Hub.
> * Cargar el área de la geovalla en el servicio de datos de Azure Maps con Data Upload API.
> * Crear una función en Azure Functions, para lo que debe implementar una lógica de negocios basada en el análisis espacial de Azure Maps.
> * Suscribirse a eventos de telemetría de dispositivos IoT desde la función de Azure a través de Event Grid.
> * Filtrar los eventos de telemetría mediante el enrutamiento de mensajes de IoT Hub.
> * Crear una cuenta de almacenamiento para almacenar los datos de los eventos pertinentes.
> * Simular un dispositivo IoT en un vehículo.
    

## <a name="use-case"></a>Caso de uso

Esta solución muestra un escenario en el que una empresa de alquiler de automóviles planea supervisar y registrar los eventos de los automóviles que alquila. Las compañías de alquiler de vehículos suelen operar en una región geográfica específica. Necesitan realizar el seguimiento de la ubicación de los vehículos cuando están alquilados. Se deben registrar las instancias de un vehículo que abandona la región geográfica elegida. Los datos de registro garantizan que las directivas, las cuotas y otros aspectos empresariales se gestionan correctamente.

En nuestro caso, los vehículos de alquiler están equipados con dispositivos de IoT que envían datos de telemetría a Azure IoT Hub de forma periódica. Los datos de telemetría incluyen la ubicación actual e indican si el motor del vehículo está en funcionamiento. El esquema de ubicación del dispositivo cumple el [esquema IoT Plug and Play para datos geoespaciales](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v1-preview/schemas/geospatial.md). El esquema de los datos de telemetría del dispositivo del vehículo de alquiler es como este:

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

Vamos a usar los datos de telemetría del dispositivo del vehículo para lograr nuestro objetivo. Queremos ejecutar reglas de geovalla y responder cada vez que recibamos un evento que indique que el vehículo se ha movido. Para ello, nos suscribiremos a los eventos de telemetría del dispositivo desde IoT Hub mediante Event Grid. 

Hay varias maneras de suscribirse a Event Grid y en este tutorial usaremos Azure Functions. Azure Functions reacciona a los eventos publicados en Event Grid. También implementa la lógica de negocios de alquiler de automóviles, que se basa en el análisis espacial de Azure Maps. 

El código de la función de Azure comprueba si el vehículo ha abandonado la geovalla. Si el vehículo ha abandonado la geovalla, la función de Azure recopila información adicional, como la dirección asociada a la ubicación actual. La función también implementa la lógica para almacenar los datos significativos del evento en un almacenamiento de blobs de datos que ayudan a proporcionar una descripción de las circunstancias del evento. 

Las circunstancias de los eventos pueden ser útiles para la empresa de alquiler de automóviles y el cliente de alquiler. En el siguiente diagrama se proporciona información general del sistema.

 
  <center>

  ![Información general del sistema](./media/tutorial-iot-hub-maps/system-diagram.png)
  
  </center>

La siguiente ilustración representa el área de geovalla resaltada en azul. La ruta del vehículo del alquiler se indica mediante una línea verde.

  ![Ruta de geovalla](./media/tutorial-iot-hub-maps/geofence-route.png)


## <a name="prerequisites"></a>Prerrequisitos 

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para completar los pasos de este tutorial, antes es preciso crear un grupo de recursos en Azure Portal. Para crear un grupo de recursos, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Seleccione **Grupos de recursos**.
    
   ![Grupos de recursos](./media/tutorial-iot-hub-maps/resource-group.png)

3. En **Grupos de recursos**, seleccione **Agregar**.
    
   ![Agregar un grupo de recursos](./media/tutorial-iot-hub-maps/add-resource-group.png) 

4. Escriba los valores de las siguientes propiedades:
    * **Subscription** (Suscripción): Seleccione su suscripción a Azure.
    * **Grupo de recursos:** Escriba "ContosoRental" como nombre del grupo de recursos.
    * **Región:** Seleccione una región para el grupo de recursos.  

    ![Detalles del grupo de recursos](./media/tutorial-iot-hub-maps/resource-details.png)

    Seleccione **Revisar y crear** y, a continuación, seleccione **Crear** en la página siguiente.

### <a name="create-an-azure-maps-account"></a>Crear una cuenta de Azure Maps 

Para implementar la lógica de negocios basada en el análisis espacial de Azure Maps, es preciso crear una cuenta de Azure Maps en el grupo de recursos que hemos creado. Siga las instrucciones de [Crear una cuenta](quick-demo-map-app.md#create-an-azure-maps-account) para crear una suscripción de cuenta de Azure Maps en el plan de tarifa S1. Siga los pasos descritos en la sección de [obtención de la clave principal](quick-demo-map-app.md#get-the-primary-key-for-your-account) para obtener la clave principal de la cuenta. Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](how-to-manage-authentication.md).



### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Para registrar datos de evento, vamos a crear una cuenta de almacenamiento **v2storage** de uso general que proporciona acceso a todos los servicios de Azure Storage: Blob, File, Queue, Table y Disk.  Deberá colocar esta cuenta de almacenamiento en el grupo de recursos "ContosoRental" para almacenar los datos como blobs. Para crear una cuenta de almacenamiento, siga las instrucciones del artículo [Creación de una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&tabs=azure-portal). Después, crearemos un contenedor para almacenar los blobs. Para ello, siga estos pasos:

1. En la "cuenta de almacenamiento: blob, archivo, tabla, cola", vaya a Contenedores.

    ![blobs](./media/tutorial-iot-hub-maps/blobs.png)

2. Haga clic en el botón del contenedor en la parte superior izquierda, asigne al contenedor el nombre "contoso-rental-logs" y haga clic en "Aceptar".

    ![blob-container](./media/tutorial-iot-hub-maps/blob-container.png)

3. Vaya a la hoja **Claves de acceso** de la cuenta de almacenamiento y copie el "nombre de la cuenta de almacenamiento" y la "clave de acceso". Se necesitan en un paso posterior.

    ![access-keys](./media/tutorial-iot-hub-maps/access-keys.png)


Ahora tenemos una cuenta de almacenamiento y un contenedor para registrar los datos de los eventos. Después, crearemos un centro de IoT.

### <a name="create-an-iot-hub"></a>Creación de un IoT Hub

IoT Hub es un servicio administrado en la nube. IoT Hub actúa como centro de mensajes común para la comunicación bidireccional entre la aplicación de IoT y los dispositivos que administra. Para enrutar los mensajes de telemetría del dispositivo a una instancia de Event Grid, cree un centro de IoT en el grupo de recursos "ContosoRental". Configure una integración de ruta de mensajes en la que se filtrarán los mensajes según el estado del motor del automóvil. También enviaremos mensajes de telemetría del dispositivo a Event Grid cada vez que se mueva el automóvil.

> [!Note] 
> La funcionalidad de IoT Hub para publicar eventos de telemetría de dispositivos en Event Grid está en versión preliminar pública. Las características en versión preliminar pública están disponibles en todas las regiones, excepto en **Este de EE. UU., Oeste de EE. UU., Oeste de Europa, Azure Government, Azure China 21Vianet** y **Azure Alemania**. 

Cree un centro de IoT, para lo que debe seguir los pasos de la [sección sobre cómo crear un centro de IoT](https://docs.microsoft.com/azure/iot-hub/quickstart-send-telemetry-dotnet#create-an-iot-hub).


### <a name="register-a-device"></a>Registrar un dispositivo 

Para conectarse al centro de IoT, el dispositivo debe estar registrado. Para registrar un dispositivo en el centro de IoT, siga estos pasos:

1. En su centro de IoT, haga clic en la hoja "Dispositivos IoT" y, después, en "Nuevo".

    ![add-device](./media/tutorial-iot-hub-maps/add-device.png)

2. En la página de creación de dispositivos, asigne un nombre al dispositivo IoT y haga clic en "Guardar".
    
    ![register-device](./media/tutorial-iot-hub-maps/register-device.png)

3. Guarde la **cadena de conexión principal** del dispositivo para usarla en un paso posterior, en el que necesita cambiar un marcador de posición con esta cadena de conexión.

    ![add-device](./media/tutorial-iot-hub-maps/connection-string.png)

## <a name="upload-geofence"></a>Carga de geovalla

Usaremos la [aplicación Postman](https://www.getpostman.com) para [cargar la geovalla](https://docs.microsoft.com/azure/azure-maps/geofence-geojson) en el servicio Azure Maps mediante Data Upload API de Azure Maps. Cuando el automóvil esté fuera de esta geovalla se registrarán todos los eventos.

Abra la aplicación Postman y siga estos pasos para cargar la geovalla mediante Data Upload API de Azure Maps.  

1. En la aplicación Postman, haga clic en New | Create new (Nuevo | Crear) y seleccione Request (Solicitud). Escriba un nombre de solicitud para la carga de datos de geovalla, seleccione una colección o carpeta donde guardarlo y haga clic en Save (Guardar).

    ![Carga de geovallas con Postman](./media/tutorial-iot-hub-maps/postman-new.png)

2. Seleccione el método POST HTTP en la pestaña del generador e introduzca la siguiente URL para realizar una solicitud POST.

    ```HTTP
    https://atlas.microsoft.com/mapData/upload?subscription-key={subscription-key}&api-version=1.0&dataFormat=geojson
    ```
    
    El valor "geojson" del parámetro `dataFormat` de la ruta de acceso de la dirección URL representa el formato de datos de los datos que se cargan.

3. Haga clic en **Params** (Parámetros) y escriba los siguientes pares de clave-valor que se usarán para la dirección URL de la solicitud POST. Reemplace el valor de subscription-key por su clave de Azure Maps.
   
    ![Parámetros de clave-valor de Postman](./media/tutorial-iot-hub-maps/postman-key-vals.png)

4. Haga clic en **Body** (Cuerpo), seleccione el formato de entrada **raw** (sin procesar) y elija **JSON (application/text)** [JSON (aplicación/texto)] como formato de entrada en la lista desplegable. Abra el archivo de datos JSON [aquí](https://raw.githubusercontent.com/Azure-Samples/iothub-to-azure-maps-geofencing/master/src/Data/geofence.json?token=AKD25BYJYKDJBJ55PT62N4C5LRNN4) y cópielo en la sección Body (Cuerpo) como los datos que se van a cargar y haga clic en **Send** (Enviar).
    
    ![publicar datos](./media/tutorial-iot-hub-maps/post-json-data.png)
    
5. Examine los encabezados de la respuesta. Tras una solicitud correcta, el encabezado **Location** (Ubicación) contendrá el identificador URI de estado para comprobar el estado actual de la solicitud de carga. El identificador URI de estado tendrá el siguiente formato. 

   ```HTTP
   https://atlas.microsoft.com/mapData/{uploadStatusId}/status?api-version=1.0
   ```

6. Copie el URI de estado y anexe un parámetro `subscription-key` a él. Asigne el valor de la clave de suscripción de la cuenta de Azure Maps al parámetro `subscription-key`. El formato de URI de estado debe ser similar al siguiente y `{Subscription-key}` debe reemplazarse por la clave de suscripción.

   ```HTTP
   https://atlas.microsoft.com/mapData/{uploadStatusId}/status?api-version=1.0&subscription-key={Subscription-key}
   ```

7. Para obtener `udId`, abra una pestaña nueva en la aplicación Postman, seleccione el método GET HTTP en la pestaña del generador y realice una solicitud GET en el identificador URI de estado. Si la carga de datos se realizó correctamente, recibirá un UDID en el cuerpo de la respuesta. Copie el UDID para su uso posterior.

   ```JSON
   {
    "udid" : "{udId}"
   }
   ```


A continuación, crearemos una función de Azure en el grupo de recursos "ContosoRental" y configuraremos una ruta de mensajería en IoT Hub para filtrar los mensajes de telemetría del dispositivo.


## <a name="create-an-azure-function-and-add-an-event-grid-subscription"></a>Creación de una función de Azure y adición de una suscripción a Event Grid

Azure Functions es un servicio de proceso sin servidor que permite ejecutar código a petición sin necesidad de aprovisionar ni administrar explícitamente la infraestructura de proceso. Para más información acerca de Azure Functions, consulte la documentación de [Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-overview). 

La lógica que implementamos en la función usa los datos de ubicación procedentes de los datos de telemetría del dispositivo del vehículo para evaluar el estado de la geovalla. En caso de que un vehículo determinado salga de la geovalla, la función recopilará más información, como la dirección de su ubicación, mediante [Get Search Address Reverse API](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse). Esta API convierte las coordenadas de una ubicación determinada en una dirección postal inteligible. 

Toda la información pertinente de los eventos se guarda en el almacén de blobs. En el paso 5 siguiente se apunta al código ejecutable que implementa dicha lógica. Siga los pasos que se indican a continuación para crear una función de Azure que envíe registros de datos al contenedor de blobs de la cuenta de almacenamiento de blobs y agréguele una suscripción a Event Grid.

1. En el panel de Azure Portal, seleccione Crear un recurso. Seleccione **Proceso** en la lista de tipos de recursos disponibles y, después, seleccione **Function App**.

    ![create-resource](./media/tutorial-iot-hub-maps/create-resource.png)

2. En la página de creación de la **Aplicación de funciones**, asigne un nombre a la aplicación de funciones. En **Grupo de recursos**, seleccione **Usar existente** y seleccione "ContosoRental" en la lista desplegable. Seleccione ".NET Core" como la pila en tiempo de ejecución. En **Hospedaje**, para **Cuenta de almacenamiento**, seleccione el nombre de la cuenta de almacenamiento de un paso anterior. En el paso anterior, hemos llamado a la cuenta de almacenamiento **v2storage**.  Seleccione **Revisar y crear**.
    
    ![create-app](./media/tutorial-iot-hub-maps/rental-app.png)

2. Revise los detalles de la aplicación de funciones y seleccione "Crear".

3. Una vez creada la aplicación, es preciso agregarle una función. Vaya a la aplicación de función. Haga clic en **Nueva función** para agregar una función. Elija **En el portal** como entorno de desarrollo. Después, seleccione **Continuar**.

    ![create-function](./media/tutorial-iot-hub-maps/function.png)

4. Elija **Más plantillas** y haga clic en **Finalizar y ver plantillas**. 

5. Seleccione la plantilla con un **desencadenador de Azure Event Grid**. Instale las extensiones si se le solicita, asigne un nombre a la función y seleccione **Crear**.

    ![function-template](./media/tutorial-iot-hub-maps/eventgrid-funct.png)
    
    El **desencadenador del centro de eventos de Azure** y el **desencadenador de Azure Event Grid** tienen iconos similares. Asegúrese de seleccionar el **desencadenador de Azure Event Grid**.

6. Copie el [código C#](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/blob/master/src/Azure%20Function/run.csx) en la función.
 
7. En el script de C#, reemplace los siguientes parámetros. Haga clic en **Save**(Guardar). No haga clic en **Ejecutar** todavía.
    * Reemplace **SUBSCRIPTION_KEY** por la clave de suscripción principal de su cuenta de Azure Maps.
    * Reemplace **UDID** por el UDID de la geovalla que cargó. 
    * La función **CreateBlobAsync** del script crea un blob por evento en la cuenta de almacenamiento de datos. Reemplace **ACCESS_KEY**, **ACCOUNT_NAME** y **STORAGE_CONTAINER_NAME** por la clave de acceso de la cuenta de almacenamiento, el nombre de la cuenta y el contenedor de almacenamiento de datos.

10. Haga clic en **Agregar suscripción a Event Grid**.
    
    ![add-event-grid](./media/tutorial-iot-hub-maps/add-egs.png)

11. Rellene los detalles de la suscripción. En **EVENT SUBSCRIPTION DETAILS**, asigne un nombre a la suscripción de eventos. En el esquema de eventos, elija "Esquema de Event Grid". En **Detalles del tema**, seleccione "Cuentas de Azure IoT Hub" como tipo de tema. Elija la misma suscripción que usó para crear el grupo de recursos, seleccione "ContosoRental" como "Grupo de recursos" y, en "Recurso", elija la instancia de IoT Hub que creó. Seleccione **Device Telemetry** (Telemetría del dispositivo) en Tipo de evento. Después de elegir estas opciones, verá que "Tipo de tema" cambia automáticamente a "IoT Hub".

    ![event-grid-subscription](./media/tutorial-iot-hub-maps/af-egs.png)
 

## <a name="filter-events-using-iot-hub-message-routing"></a>Filtrado de eventos mediante el enrutamiento de mensajes de IoT Hub

Después de agregar una suscripción a Event Grid a la función de Azure, verá una ruta de mensajería predeterminada a Event Grid en la hoja **Message Routing** (Enrutamiento de mensajes) de IoT Hub. El enrutamiento de mensajes le permite enrutar distintos tipos de datos a varios puntos de conexión. Por ejemplo, puede enrutar los mensajes de telemetría del dispositivo, los eventos del ciclo de vida del dispositivo y los eventos de cambio de dispositivo gemelo. Para más información acerca del enrutamiento de mensajes en IoT Hub, consulte [Uso del enrutamiento de mensajes de IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-d2c).

![hub-EG-route](./media/tutorial-iot-hub-maps/hub-route.png)

En el escenario de ejemplo, queremos filtrar todos los mensajes en los que el vehículo de alquiler se mueve. Para publicar estos eventos de telemetría del dispositivo en Event Grid, usaremos la consulta de enrutamiento para filtrar los eventos en los que la propiedad `Engine` se encuentre en **"ON"** . Hay varias maneras de consultar los mensajes del dispositivo IoT a la nube. Para más información acerca de la sintaxis de enrutamiento de mensajes, consulte el artículo acerca del [enrutamiento de mensajes de IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-routing-query-syntax). Para crear una consulta de enrutamiento, haga clic en la ruta **RouteToEventGrid**, reemplace la **consulta de enrutamiento** por **"Engine='ON'"** y haga clic en **Guardar**. Ahora, IoT Hub solo publicará los datos de telemetría del dispositivo cuando el valor de Engine sea ON.

![hub-EG-filter](./media/tutorial-iot-hub-maps/hub-filter.png)


## <a name="send-telemetry-data-to-iot-hub"></a>Envío de datos de telemetría a IoT Hub

Una vez que la función de Azure esté en funcionamiento, se pueden enviar los datos de telemetría al centro de IoT, que los enrutará a Event Grid. Usaremos una aplicación creada en C# para simular los datos de ubicación de un dispositivo situado en un automóvil de alquiler. Para ejecutar la aplicación, se necesita el SDK de .NET Core 2.1.0, o una versión posterior, en el equipo de desarrollo. Siga los pasos que se indican a continuación para enviar datos de telemetría simulados a IoT Hub.

1. Descargue el proyecto de C# [rentalCarSimulation](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/tree/master/src/rentalCarSimulation). 

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

  ![Salida del terminal](./media/tutorial-iot-hub-maps/terminal.png)

Si abre el contenedor de almacenamiento de blobs ahora, verá cuatro blobs, que corresponden a las ubicaciones en las que el vehículo estuvo fuera de la geovalla.

![Acceso al blob](./media/tutorial-iot-hub-maps/blob.png)

En el mapa siguiente se muestran cuatro puntos en los que el vehículo estuvo fuera de la geovalla, registrados a intervalos regulares.

![mapa de la infracción](./media/tutorial-iot-hub-maps/violation-map.png)

## <a name="next-steps"></a>Pasos siguientes

Para explorar las API de Azure Maps que se usan en este tutorial, consulte:

* [Get Search Address Reverse](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse)
* [Get Geofence](https://docs.microsoft.com/rest/api/maps/spatial/getgeofence)

Para obtener una lista completa de las API REST de Azure Maps, consulte:

* [API REST de Azure Maps](https://docs.microsoft.com/rest/api/maps/spatial/getgeofence)

Para más información acerca de IoT Plug and Play, consulte:

* [IoT Plug and Play](https://docs.microsoft.com/azure/iot-pnp)

Para obtener una lista de los dispositivos que tienen la certificación de Azure para IoT, visite:

* [Dispositivos certificados de Azure](https://catalog.azureiotsolutions.com/)

Para más información acerca de cómo enviar datos de telemetría del dispositivo a la nube y viceversa, consulte:

* [Envío de datos de telemetría desde un dispositivo](https://docs.microsoft.com/azure/iot-hub/quickstart-send-telemetry-dotnet)
