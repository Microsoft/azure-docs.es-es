---
title: Envío de notificaciones push con Azure Notification Hubs y Bing Spatial Data | Microsoft Docs
description: En este tutorial, aprenderá a entregar notificaciones push basadas en ubicaciones con Azure Notification Hubs y datos espaciales de Bing.
services: notification-hubs
documentationcenter: windows
keywords: push notifications,push notifications
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: f41beea1-0d62-4418-9ffc-c9d70607a1b7
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows-phone
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: 3065cd6f185c0fdfaea1d6c0d02d2fee69bd04c7
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87116928"
---
# <a name="tutorial-send-location-based-push-notifications-with-notification-hubs-and-bing-spatial-data"></a>Tutorial: Envío de notificaciones push basadas en la ubicación con Notification Hubs y Bing Spatial Data

En este tutorial, aprenderá a entregar notificaciones push basadas en ubicaciones con Azure Notification Hubs y datos espaciales de Bing.

En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Configuración del origen de datos
> * Configuración de la aplicación de UWP
> * Configuración del back-end
> * Prueba de envío de notificaciones push a aplicaciones de la Plataforma universal de Windows (UWP)

## <a name="prerequisites"></a>Requisitos previos

* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* [Visual Studio 2015 Update 1](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx) o posterior ([Community Edition](https://go.microsoft.com/fwlink/?LinkId=691978&clcid=0x409)).
* La versión más reciente del [SDK de Azure](https://azure.microsoft.com/downloads/).
* [Cuenta del Centro de desarrollo de Bing Maps](https://www.bingmapsportal.com/) (puede crear una gratis y asociarla a su cuenta de Microsoft).

## <a name="set-up-the-data-source"></a>Configuración del origen de datos

1. Inicie sesión en el [Centro de desarrollo de Bing Maps](https://www.bingmapsportal.com/).
2. En la barra de navegación superior, seleccione **Data sources** (Orígenes de datos) y **Manage Data Sources** (Administrar orígenes de datos).

    ![Captura de pantalla del centro de desarrollo de Bing Maps en la página Administrar orígenes de datos con la opción Cargar datos como origen de datos destacada en rojo.](./media/notification-hubs-geofence/bing-maps-manage-data.png)
3. Si no tiene ningún origen de datos, verá un vínculo para crear uno. Seleccione **Upload data as a data source** (Cargar datos como origen de datos). También puede usar el menú **Data sources** (Orígenes de datos) > **Upload data** (Cargar datos).

    ![Captura de pantalla del cuadro de diálogo Cargar origen de datos.](./media/notification-hubs-geofence/bing-maps-create-data.png)
4. Cree un archivo `NotificationHubsGeofence.pipe` en el disco duro con el siguiente contenido: En este tutorial se usa un archivo de ejemplo basado en canalización que enmarca un área del litoral de San Francisco:

    ```text
    Bing Spatial Data Services, 1.0, TestBoundaries
    EntityID(Edm.String,primaryKey)|Name(Edm.String)|Longitude(Edm.Double)|Latitude(Edm.Double)|Boundary(Edm.Geography)
    1|SanFranciscoPier|||POLYGON ((-122.389825 37.776598,-122.389438 37.773087,-122.381885 37.771849,-122.382186 37.777022,-122.389825 37.776598))
    ```

    El archivo de canalización representa esta entidad:

    ![Captura de pantalla de un mapa del litoral de San Francisco con un polígono rojo que delinea un área de los muelles.](./media/notification-hubs-geofence/bing-maps-geofence.png)
5. En la página **Upload a data source** (Cargar un origen de datos), realice las siguientes acciones:
   1. Seleccione **canalización** como **Formato de datos**.
   2. Busque y seleccione el archivo `NotificationHubGeofence.pipe` que creó en el paso anterior.
   3. Seleccione el botón **Cargar**.

      > [!NOTE]
      > Se le pedirá que especifique una nueva clave para la **clave maestra** que sea diferente de la **clave de consulta**. Simplemente cree una nueva clave en el panel y actualice la página de carga del origen de datos.
6. Una vez cargado en el archivo de datos, asegúrese de publicar el origen de datos. Selecciones **Data sources** (Orígenes de datos) -> **Manage Data Sources** (Administrar orígenes de datos) como hizo anteriormente.
7. Seleccione el origen de datos de la lista y elija **Publicar** en la columna **Acciones**.

    ![Captura de pantalla del Centro de desarrollo de Bing Maps en la página Administrar orígenes de datos con la pestaña Datos codificados y la opción Publicar destacadas en rojo.](./media/notification-hubs-geofence/publish-button.png)
8. Cambie a la pestaña **Published Data Sources** (Orígenes de datos publicados) y confirme que ve el origen de datos en la lista.

    ![Captura de pantalla del Centro de desarrollo de Bing Maps en la página Administrar orígenes de datos con la pestaña Orígenes de datos publicados seleccionada.](./media/notification-hubs-geofence/bing-maps-published-data.png)
9. Seleccione **Editar**. Verá (de un vistazo) qué ubicaciones introdujo en los datos.

    ![Captura de pantalla de la página Editar datos de entidad que muestra un mapa de la zona oeste de los Estados Unidos y un punto magenta en la costa de San Francisco.](./media/notification-hubs-geofence/bing-maps-data-details.png)

    En este punto, el portal no muestra los límites de la geovalla creada; todo lo necesario es una confirmación de que la ubicación especificada está en las proximidades correctas.
10. Ahora tiene todos los requisitos para el origen de datos. Para obtener los detalles de la URL de solicitud para la llamada de API, en el Centro de desarrollo de Mapas de Bing, elija **Data sources** (Orígenes de datos) y seleccione **Data Source Information** (Información de origen de datos).

    ![Captura de pantalla del Centro de desarrollo de Bing Maps en la página Información del origen de datos.](./media/notification-hubs-geofence/bing-maps-data-info.png)

    La **dirección URL de consulta** es el punto de conexión de ejecución de consultas para comprobar si el dispositivo está dentro de los límites de una ubicación o no. Para realizar esta comprobación, solo hay que ejecutar una llamada GET en la URL de consulta con los siguientes parámetros anexados:

    ```text
    ?spatialFilter=intersects(%27POINT%20LONGITUDE%20LATITUDE)%27)&$format=json&key=QUERY_KEY
    ```

    Bing Maps realiza los cálculos automáticamente para ver si el dispositivo está en la geovalla. Una vez ejecutada la solicitud a través de un explorador (o cURL), se obtiene una respuesta JSON estándar:

    ![Captura de pantalla de la respuesta JSON estándar.](./media/notification-hubs-geofence/bing-maps-json.png)

    Esta respuesta solo se produce cuando el punto realmente está dentro de los límites designados. Si no es así, obtendrá un cubo de **resultados** vacío:

    ![Captura de pantalla de una respuesta JSON con un depósito de resultados vacío.](./media/notification-hubs-geofence/bing-maps-nores.png)

## <a name="set-up-the-uwp-application"></a>Configuración de la aplicación de UWP

1. En Visual Studio, inicie un nuevo proyecto de tipo **Aplicación vacía (Windows universal)** .

    ![Captura de pantalla de un cuadro de diálogo Nuevo proyecto de Visual Studio con la aplicación en blanco (opción de Universal Windows Visual C# resaltada).](./media/notification-hubs-geofence/notification-hubs-create-blank-app.png)

    Una vez completada la creación del proyecto, debe tener la base para la propia aplicación. Ahora vamos a configurar todos los elementos de la infraestructura del geovallado. Como se van a usar los servicios de Bing para esta solución, hay un punto de conexión de API REST público que permite consultar marcos de ubicación específicos:

    ```text
    http://spatial.virtualearth.net/REST/v1/data/
    ```
    Especifique los siguientes parámetros para que funcione:

   * **Id. de origen de datos** y **Nombre de origen de datos**: en la API de Bing Maps, los orígenes de datos contienen distintos metadatos en depósitos, como las ubicaciones y el horario comercial de la operación.  
   * **Nombre de entidad** : la entidad que desea utilizar como punto de referencia para la notificación.
   * **Clave de API de Bing Maps**: la que obtuvo anteriormente al crear la cuenta del centro de desarrollo de Bing.

     Ahora que ya está preparado el origen de datos, se puede empezar a trabajar en la aplicación de UWP.
2. Habilite los servicios de ubicación de la aplicación. En el **Explorador de soluciones**, abra el archivo `Package.appxmanifest`.

    ![Captura de pantalla del Explorador de soluciones con el archivo Package.appxmanifest resaltado.](./media/notification-hubs-geofence/vs-package-manifest.png)
3. En la pestaña de propiedades del paquete que acaba de abrir, cambie a la pestaña **Funcionalidades** y seleccione **Ubicación**.

    ![Captura de pantalla del cuadro de diálogo Propiedades del paquete que muestra la pestaña Funcionalidades con la opción Ubicación resaltada.](./media/notification-hubs-geofence/vs-package-location.png)
4. Cree una nueva carpeta en la solución denominada `Core` y agregue un nuevo archivo en ella, llamado `LocationHelper.cs`:

    ![Captura de pantalla del Explorador de soluciones con la nueva carpeta principal resaltada.](./media/notification-hubs-geofence/vs-location-helper.png)

    La clase `LocationHelper` tiene código para obtener la ubicación del usuario a través de la API del sistema:

    ```csharp
    using System;
    using System.Threading.Tasks;
    using Windows.Devices.Geolocation;

    namespace NotificationHubs.Geofence.Core
    {
        public class LocationHelper
        {
            private static readonly uint AppDesiredAccuracyInMeters = 10;

            public async static Task<Geoposition> GetCurrentLocation()
            {
                var accessStatus = await Geolocator.RequestAccessAsync();
                switch (accessStatus)
                {
                    case GeolocationAccessStatus.Allowed:
                        {
                            Geolocator geolocator = new Geolocator { DesiredAccuracyInMeters = AppDesiredAccuracyInMeters };

                            return await geolocator.GetGeopositionAsync();
                        }
                    default:
                        {
                            return null;
                        }
                }
            }

        }
    }
    ```

    Para más información sobre cómo obtener la ubicación del usuario en las aplicaciones para UWP, consulte [Obtención de la ubicación del usuario](/windows/uwp/maps-and-location/get-location).
5. Para comprobar que la adquisición de ubicación funciona realmente, abra la parte de código de la página principal (`MainPage.xaml.cs`). Cree un nuevo controlador de eventos para el evento `Loaded` en el constructor `MainPage`.

    ```csharp
    public MainPage()
    {
        this.InitializeComponent();
        this.Loaded += MainPage_Loaded;
    }
    ```

    La implementación del controlador de eventos es como sigue:

    ```csharp
    private async void MainPage_Loaded(object sender, RoutedEventArgs e)
    {
        var location = await LocationHelper.GetCurrentLocation();

        if (location != null)
        {
            Debug.WriteLine(string.Concat(location.Coordinate.Longitude,
                " ", location.Coordinate.Latitude));
        }
    }
    ```
6. Ejecute la aplicación y permita que acceda a su ubicación.

    ![Captura de pantalla del cuadro de diálogo de acceso a la ubicación de Let Notification Hubs Geo Fence.](./media/notification-hubs-geofence/notification-hubs-location-access.png)
7. Cuando se inicie la aplicación, podrá ver las coordenadas en la ventana **Salida** :

    ![Captura de pantalla de la ventana de salida que muestra las coordenadas.](./media/notification-hubs-geofence/notification-hubs-location-output.png)

    Ahora que ya sabe que la adquisición de la ubicación funciona, puede quitar el controlador de eventos Loaded si lo desea, ya que no se volverá a usar.
8. El paso siguiente consiste en capturar los cambios de ubicación. En la clase `LocationHelper`, agregue el controlador de eventos para `PositionChanged`:

    ```csharp
    geolocator.PositionChanged += Geolocator_PositionChanged;
    ```

    La implementación muestra las coordenadas de ubicación en la ventana **Salida**:

    ```csharp
    private static async void Geolocator_PositionChanged(Geolocator sender, PositionChangedEventArgs args)
    {
        await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
        {
            Debug.WriteLine(string.Concat(args.Position.Coordinate.Longitude, " ", args.Position.Coordinate.Latitude));
        });
    }
    ```

## <a name="set-up-the-backend"></a>Configuración del back-end

1. Descargue el [ejemplo de back-end de .NET desde GitHub](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/NotifyUsers).
2. Una vez finalizada la descarga, abra la carpeta `NotifyUsers` y el archivo `NotifyUsers.sln` en Visual Studio.
3. Establezca el proyecto `AppBackend` como **proyecto de inicio** e inícielo.

    ![Captura de pantalla del menú contextual de la solución con la opción Establecer como proyecto de inicio resaltada.](./media/notification-hubs-geofence/vs-startup-project.png)

    El proyecto ya está configurado para enviar notificaciones push a dispositivos de destino, por lo que solo es necesario hacer dos cosas: especificar la cadena de conexión adecuada para el centro de notificaciones y agregar la identificación de límite para enviar la notificación solo cuando el usuario está dentro de la geovalla.

4. Para configurar la cadena de conexión, en la carpeta `Models` abra `Notifications.cs`. La función `NotificationHubClient.CreateClientFromConnectionString` debe contener la información sobre el centro de notificaciones que puede obtener en [Azure Portal](https://portal.azure.com) (busque dentro de la página **Directivas de acceso** en **Configuración**). Guarde el archivo de configuración actualizado.
5. Cree un modelo para el resultado de la API de Bing Maps. La manera más fácil de hacerlo es abrir la carpeta `Models` y elegir **Agregar** > **Clase**. Asígnele el nombre `GeofenceBoundary.cs`. Una vez hecho, copie el JSON de la respuesta de API que obtuvo en la primera sección. En Visual Studio, utilice **Editar** > **Pegado especial** > **Pegar JSON como clases**.

    De este modo se garantiza que se deserializa el objeto tal y como estaba previsto. El conjunto de clases resultante debe ser similar a esta clase:

    ```csharp
    namespace AppBackend.Models
    {
        public class Rootobject
        {
            public D d { get; set; }
        }

        public class D
        {
            public string __copyright { get; set; }
            public Result[] results { get; set; }
        }

        public class Result
        {
            public __Metadata __metadata { get; set; }
            public string EntityID { get; set; }
            public string Name { get; set; }
            public float Longitude { get; set; }
            public float Latitude { get; set; }
            public string Boundary { get; set; }
            public string Confidence { get; set; }
            public string Locality { get; set; }
            public string AddressLine { get; set; }
            public string AdminDistrict { get; set; }
            public string CountryRegion { get; set; }
            public string PostalCode { get; set; }
        }

        public class __Metadata
        {
            public string uri { get; set; }
        }
    }
    ```
6. A continuación, abra `Controllers` > `NotificationsController.cs`. Actualice la llamada Post a la cuenta para la latitud y la longitud de destino. Para ello, agregue dos cadenas a la signatura de función: `latitude` y `longitude`.

    ```csharp
    public async Task<HttpResponseMessage> Post(string pns, [FromBody]string message, string to_tag, string latitude, string longitude)
    ```
7. Cree una nueva clase en el proyecto denominado `ApiHelper.cs`: se usará para la conexión a Bing y la comprobación de las intersecciones de los límites de punto. Implemente una función `IsPointWithinBounds` como se muestra en el código siguiente:

    ```csharp
    public class ApiHelper
    {
        public static readonly string ApiEndpoint = "{YOUR_QUERY_ENDPOINT}?spatialFilter=intersects(%27POINT%20({0}%20{1})%27)&$format=json&key={2}";
        public static readonly string ApiKey = "{YOUR_API_KEY}";

        public static bool IsPointWithinBounds(string longitude,string latitude)
        {
            var json = new WebClient().DownloadString(string.Format(ApiEndpoint, longitude, latitude, ApiKey));
            var result = JsonConvert.DeserializeObject<Rootobject>(json);
            if (result.d.results != null && result.d.results.Count() > 0)
            {
                return true;
            }
            else
            {
                return false;
            }
        }
    }
    ```

    > [!IMPORTANT]
    > Asegúrese de sustituir el punto de conexión de API con la dirección URL de consulta que obtuvo anteriormente en el Centro de desarrollo de Bing (lo mismo se aplica a la clave de API).

    Si hay resultados para la consulta, significa que el punto especificado se encuentra dentro de los límites de la geovalla, por lo que la función devuelve `true`. Si no hay ningún resultado, Bing indica que el punto está fuera del marco de búsqueda, por lo que la función devuelve `false`.
8. En `NotificationsController.cs`, cree una comprobación justo antes de la instrucción switch:

    ```csharp
    if (ApiHelper.IsPointWithinBounds(longitude, latitude))
    {
        switch (pns.ToLower())
        {
            case "wns":
                //// Windows 8.1 / Windows Phone 8.1
                var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" +
                            "From " + user + ": " + message + "</text></binding></visual></toast>";
                outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);

                // Windows 10 specific Action Center support
                toast = @"<toast><visual><binding template=""ToastGeneric""><text id=""1"">" +
                            "From " + user + ": " + message + "</text></binding></visual></toast>";
                outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);

                break;
        }
    }
    ```

## <a name="test-push-notifications-in-the-uwp-app"></a>Prueba de las notificaciones push en la aplicación de UWP

1. En la aplicación de UWP, ahora debería poder probar las notificaciones. Dentro de la clase `LocationHelper`, cree una nueva función – `SendLocationToBackend`:

    ```csharp
    public static async Task SendLocationToBackend(string pns, string userTag, string message, string latitude, string longitude)
    {
        var POST_URL = "http://localhost:8741/api/notifications?pns=" +
            pns + "&to_tag=" + userTag + "&latitude=" + latitude + "&longitude=" + longitude;

        using (var httpClient = new HttpClient())
        {
            try
            {
                await httpClient.PostAsync(POST_URL, new StringContent("\"" + message + "\"",
                    System.Text.Encoding.UTF8, "application/json"));
            }
            catch (Exception ex)
            {
                Debug.WriteLine(ex.Message);
            }
        }
    }
    ```

    > [!NOTE]
    > Establezca `POST_URL` en la ubicación de la aplicación web implementada. Ya se puede ejecutar localmente, pero al trabajar en la implementación de una versión pública es preciso hospedarlo en un proveedor externo.
2. Registre la aplicación de UWP para las notificaciones push. En Visual Studio, elija **Proyecto** > **Tienda** > **Asociar aplicación con la Tienda**.

    ![Captura de pantalla del menú contextual de la solución con la aplicación Almacenar y asociar con las opciones de la tienda resaltadas.](./media/notification-hubs-geofence/vs-associate-with-store.png)
3. Cuando se inicia sesión en su cuenta de desarrollador, asegúrese de seleccionar una aplicación existente o de crear una nueva y asociar el paquete con ella.
4. Vaya al Centro de desarrollo y abra la aplicación que ha creado. Elija **Servicios** > **Notificaciones push** > **Sitio de Servicios Live**.

    ![Captura de pantalla del Centro de desarrollo de Windows que muestra la página Notificaciones de envío con el sitio de Servicios Live resaltado.](./media/notification-hubs-geofence/ms-live-services.png)
5. En el sitio, tome nota del **secreto de aplicación** y del **SID del paquete**. Los necesitará en Azure Portal: abra el centro de notificaciones, elija **Configuración** > **Notification Services** > **Windows (WNS)** y escriba la información en los campos obligatorios.

    ![Captura de pantalla que muestra la página de configuración con las opciones Notification Services y Windows (WNS) resaltadas y los valores de SID del paquete y clave de seguridad rellenados.](./media/notification-hubs-geofence/notification-hubs-wns.png)
6. Elija **Guardar**.
7. Abra **Referencias** en el **Explorador de soluciones** y seleccione **Administrar paquetes NuGet**. Agregue una referencia a la **biblioteca administrada de Microsoft Azure Service Bus**; simplemente busque `WindowsAzure.Messaging.Managed` y agréguelo al proyecto.

    ![Captura de pantalla del cuadro de diálogo Administrar paquetes Nuget con el paquete WindowsAzure.Messaging.Managed resaltado.](./media/notification-hubs-geofence/vs-nuget.png)
8. Para las pruebas, cree el controlador de eventos `MainPage_Loaded` una vez más y agréguele este fragmento de código:

    ```csharp
    var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

    var hub = new NotificationHub("HUB_NAME", "HUB_LISTEN_CONNECTION_STRING");
    var result = await hub.RegisterNativeAsync(channel.Uri);

    // Displays the registration ID so you know it was successful
    if (result.RegistrationId != null)
    {
        Debug.WriteLine("Reg successful.");
    }
    ```

    El código registra la aplicación con el centro de notificaciones. Y ya está todo listo.
9. En `LocationHelper`, dentro del controlador `Geolocator_PositionChanged`, puede agregar un fragmento de código de prueba que incluya forzosamente la ubicación dentro de la geovalla:

    ```csharp
    await LocationHelper.SendLocationToBackend("wns", "TEST_USER", "TEST", "37.7746", "-122.3858");
    ```

10. Como no se están superando las coordenadas reales (que pueden no estar dentro de los límites en ese momento) y se están usando valores de prueba predefinidos, verá una notificación aparecer en la actualización:

    ![Captura de pantalla de un escritorio de Windows que muestra el mensaje de prueba.](./media/notification-hubs-geofence/notification-hubs-test-notification.png)

## <a name="next-steps"></a>Pasos siguientes

Hay un par de pasos que quizá deba seguir para asegurarse de que la solución está preparada para producción.

1. Primero debe asegurarse de que las geovallas sean dinámicas. Esto requiere un trabajo adicional con la API de Bing para poder cargar los nuevos límites dentro del origen de datos existente. Para más información, consulte la [documentación de API de Bing Spatial Data Services](/bingmaps/spatial-data-services/).
2. En segundo lugar, cuando se trabaja para garantizar que se realiza la entrega a los participantes adecuados, puede querer dirigirlos mediante el [etiquetado](notification-hubs-tags-segment-push-message.md).

La solución de este tutorial describe un escenario en el que se puede tener una amplia variedad de plataformas de destino, por lo que se limita el geovallado a funcionalidades específicas del sistema. Dicho esto, Plataforma universal de Windows ofrece funcionalidades para [detectar geovallas directamente de fábrica](/windows/uwp/maps-and-location/set-up-a-geofence).
