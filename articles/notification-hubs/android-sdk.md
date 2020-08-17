---
title: 'Envío de notificaciones push a Android mediante Azure Notification Hubs y el SDK de Firebase versión 1.0.0: versión preliminar 1'
description: En este tutorial aprenderá a usar Azure Notification Hubs y Google Cloud Messaging para enviar notificaciones push a dispositivos Android.
author: sethmanheim
ms.author: sethm
ms.date: 5/28/2020
ms.topic: tutorial
ms.service: notification-hubs
ms.reviewer: thsomasu
ms.lastreviewed: 05/27/2020
ms.openlocfilehash: d8c61c90456607d73fc0e8abc24709d901328ecf
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87829533"
---
# <a name="tutorial-send-push-notifications-to-android-devices-using-firebase-sdk-version-100-preview1"></a>Tutorial: Envío de notificaciones push a dispositivos Android mediante el SDK de Firebase versión 1.0.0: versión preliminar 1

En este tutorial se muestra cómo usar Azure Notification Hubs y la versión actualizada del SDK (versión 1.0.0: versión preliminar 1) de Firebase Cloud Messaging (FCM) para enviar notificaciones push a una aplicación de Android. En este tutorial, creará una aplicación de Android en blanco que recibirá notificaciones push mediante Firebase Cloud Messaging (FCM).

Puede descargar el código completo para este tutorial en [GitHub](https://github.com/Azure/azure-notificationhubs-android/tree/v1-preview/notification-hubs-test-app-refresh).

En este tutorial se describen los pasos siguientes:

- Crear un nuevo proyecto en Android Studio
- Crear un proyecto en Firebase que admita Firebase Cloud Messaging
- Crear un Centro de notificaciones
- Conectar la aplicación al centro.
- Probar la aplicación.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, deberá tener una cuenta de Azure activa. En caso de no tener cuenta, puede crear una de evaluación gratuita en tan solo unos minutos. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/free/).

También se necesitan los siguientes elementos:

- Se recomienda usar la última versión de [Android Studio](https://go.microsoft.com/fwlink/?LinkId=389797).
- La compatibilidad mínima es el nivel de API 16.

## <a name="create-an-android-studio-project"></a>Creación de un nuevo proyecto en Android Studio

El primer paso es crear un proyecto en Android Studio:

1. Inicie Android Studio.

2. Seleccione **File** (Archivo), después  **New** (Nuevo) y, a continuación, **New Project** (Nuevo proyecto).

3. En la página **Choose your project** (Elija el proyecto), seleccione **Empty Activity** (Actividad vacía) y, después **Next** (Siguiente).

4. En la página **Configure your project** (Configuración del proyecto), haga lo siguiente:
   1. Escriba un nombre para la aplicación.
   2. Especifique una ubicación para guardar los archivos del proyecto.
   3. Seleccione **Finish** (Finalizar).

   :::image type="content" source="media/android-sdk/configure-project.png" alt-text="Configurar proyecto":::

## <a name="create-a-firebase-project-that-supports-fcm"></a>Creación de un proyecto en Firebase que admita FCM

1. Inicie sesión en la [consola de Firebase](https://firebase.google.com/console/). Si aún no tiene uno, cree un nuevo proyecto de Firebase.

2. Después de crear el proyecto, seleccione **Agregar Firebase a una aplicación de Android**.

   :::image type="content" source="media/android-sdk/get-started.png" alt-text="Agregar Firebase":::

3. En la página **Agregar Firebase a una aplicación de Android**, realice los pasos siguientes:

   1. En **Nombre del paquete de Android**, copie el valor de**applicationId** en el archivo **build.gradle** de la aplicación. En este ejemplo, es `com.fabrikam.fcmtutorial1app`.

      :::image type="content" source="media/android-sdk/specify-package-name-fcm-settings.png" alt-text="Especificar el nombre del paquete":::

   2. Seleccione **Registrar aplicación**.

4. Seleccione **Descargar google-services.json**, guarde el archivo en la carpeta **app** del proyecto y, a continuación, seleccione **Siguiente**.

   :::image type="content" source="media/android-sdk/download-google-service-button.png" alt-text="Descargar el servicio de Google":::

5. En la consola Firebase, seleccione el icono de la rueda dentada del proyecto. Luego, seleccione **Project Settings** (Configuración del proyecto).

   :::image type="content" source="media/android-sdk/notification-hubs-firebase-console-project-settings.png" alt-text="Configuración del proyecto":::

6. Si aún no ha descargado el archivo **google-services.json** en la carpeta **app** del proyecto de Android Studio, puede hacerlo en esta página.

7. Cambie a la pestaña **Cloud Messaging**.

8. Copie y guarde la **clave del servidor** para su uso posterior. Usará este valor para configurar el concentrador.

## <a name="configure-a-notification-hub"></a>Configuración de un Centro de notificaciones

1. Inicie sesión en  [Azure Portal](https://portal.azure.com/).

2. Seleccione **Todos los servicios** en el menú de la izquierda y, a continuación, seleccione **Notification Hubs** en la sección **Móvil** . Seleccione el icono de estrella junto al nombre del servicio para agregarlo a la sección **FAVORITOS** en el menú de la izquierda. Tras agregar **Notification Hubs** a **FAVORITOS**, selecciónelo en el menú de la izquierda.

3. En la página **Notification Hubs**, seleccione **Agregar** en la barra de herramientas.

   :::image type="content" source="media/android-sdk/add-hub.png" alt-text="Agregar centro":::

4. En la página **Notification Hubs**, haga lo siguiente:

   1. Escriba un nombre en **Centro de notificaciones**.

   2. Escriba un nombre en **Crear un nuevo espacio de nombres**. Un espacio de nombres contiene uno o más centros.

   3. Seleccione un valor de la lista desplegable **Ubicación** . Este valor especifica la ubicación en la que quiere crear el centro.

   4. Seleccione un grupo de recursos existente en **Grupo de recursos** o cree uno nuevo.

   5. Seleccione  **Crear**.

      :::image type="content" source="media/android-sdk/create-hub.png" alt-text="Crear centro":::

5. Seleccione **Notificaciones** (el icono de campana) y, luego, **Ir al recurso**. También puede actualizar la lista en la página **Notification Hubs** y seleccionar su centro.

   :::image type="content" source="media/android-sdk/notification-hubs.png" alt-text="Seleccionar centro":::

6. Seleccione **Directivas de acceso** de la lista. Observe que las dos cadenas de conexión están disponibles. Las necesitará más adelante para gestionar las notificaciones push.

   :::image type="content" source="media/android-sdk/access-policies.png" alt-text="Directivas de acceso":::

   > [!IMPORTANT]
   > No use la directiva **DefaultFullSharedAccessSignature** en la aplicación. Esta directiva solo se usará en el back-end de la aplicación.

## <a name="configure-firebase-cloud-messaging-settings-for-the-hub"></a>Configuración de los valores de Firebase Cloud Messaging para el centro

1. En el panel izquierdo, en **Configuración** seleccione **Google (GCM/FCM)** .

2. Escriba la **clave de servidor** para el proyecto FCM que ha guardado anteriormente.

3. En la barra de herramientas, seleccione **Guardar**.

   :::image type="content" source="media/android-sdk/fcm-server-key.png" alt-text="Clave de servidor":::

4. Azure Portal muestra un mensaje de que el centro se ha actualizado correctamente. Se deshabilitará el botón **Guardar** .

El centro de notificaciones está ahora configurado para trabajar con Firebase Cloud Messaging. También tiene las cadenas de conexión que son necesarias para enviar notificaciones a un dispositivo y registrar una aplicación para recibir notificaciones.

## <a name="connect-your-app-to-the-notification-hub"></a>Conexión de la aplicación al Centro de notificaciones

### <a name="add-google-play-services-to-the-project"></a>Incorporación de los servicios de Google Play al proyecto

1. En Android Studio, seleccione **Tools**  (Herramientas) en el menú y, luego, seleccione **SDK Manager**.

2. Busque la versión de destino de Android SDK se usa en el proyecto. Luego, seleccione **Show Package Details** (Mostrar detalles del paquete).

   :::image type="content" source="media/android-sdk/notification-hubs-android-studio-sdk-manager.png" alt-text="Administrador de SDK":::

3. Seleccione **Google APIs** (API de Google), si aún no está instalado este componente.

   :::image type="content" source="media/android-sdk/google-apis-selected.png" alt-text="API":::

4. Cambie a la pestaña **SDK Tools** . Si no ha instalado todavía Google Play Services, seleccione **Google Play Services**, tal como se muestra en la imagen siguiente. A continuación, seleccione **Aplicar** para instalar. Tome nota de la ruta de acceso del SDK para usarla en el paso siguiente.

   :::image type="content" source="media/android-sdk/google-play-services-selected.png" alt-text="Servicios de Play":::

5. Si ve el cuadro de diálogo **Confirm Change** (Confirmar cambio), seleccione **OK** (Aceptar). El instalador de componentes se instala los componentes solicitados. Seleccione **Finish** (Finalizar) una vez instalados los componentes.

6. Seleccione **OK** (Aceptar) para cerrar el cuadro de diálogo **Settings for New Projects** (Configuración para nuevos proyectos).

### <a name="add-azure-notification-hubs-libraries"></a>Incorporación de las bibliotecas de Azure Notification Hubs

1. En el archivo **build.gradle** de la aplicación, agregue las siguientes líneas en la sección de dependencias:

   ```gradle
   implementation 'com.microsoft.azure:notification-hubs-android-sdk:1.0.0-preview1@aar'
   implementation 'androidx.appcompat:appcompat:1.0.0'

   implementation 'com.google.firebase:firebase-messaging:20.1.5'

   implementation 'com.android.volley:volley:1.1.1'
   ```

2. Agregue el siguiente repositorio después de la sección de dependencias:

   ```gradle
   repositories {
      maven {
         url "https://dl.bintray.com/microsoftazuremobile/SDK"
      }
   }
   ```

### <a name="add-google-firebase-support"></a>Adición de compatibilidad con Google Firebase

1. Agregue el siguiente complemento al final del archivo si todavía no existe.

   ```gradle
   apply plugin: 'com.google.gms.google-services'
   ```

2. Seleccione **Sincronizar ahora** en la barra de herramientas.

### <a name="add-code"></a>Incorporación de código

1. Cree un objeto **NotificationHubListener**, que controla la interceptación de los mensajes de Azure Notification Hubs.

   ```csharp
   public class CustomNotificationListener implements NotificationHubListener {

      @override

      public void onNotificationReceived(Context context, NotificationMessage message) {

      /* The following notification properties are available. */

      String title = message.getTitle();
      String message = message.getMessage();
      Map<String, String> data = message.getData();

      if (message != null) {
         Log.d(TAG, "Message Notification Title: " + title);
         Log.d(TAG, "Message Notification Body: " + message);
         }
      }
   }
   ```

2. En el método  `OnCreate` de la clase  `MainActivity`, agregue el código siguiente para iniciar el proceso de inicialización de Notification Hubs cuando se cree la actividad:

   ```java
   @Override
   protected void onCreate(Bundle savedInstanceState) {

      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity\_main);
      NotificationHub.setListener(new CustomNotificationListener());
      NotificationHub.initialize(this.getApplication(), “Connection-String”, "Hub Name");

   }
   ```

3. En Android Studio, en la barra de menús, seleccione **Build** (Compilar) y, a continuación, seleccione **Rebuild Project** (Recompilar proyecto) para asegurarse de que no hay ningún error en el código. Si recibe un error sobre el icono **ic_launcher**, quite la siguiente instrucción del archivo AndroidManifest.xml:

   ```xml
   android:icon="@mipmap/ic_launcher"
   ```

4. Asegúrese de que tiene un dispositivo virtual para ejecutar la aplicación. Si no tiene uno, agregue uno como sigue:

   1. :::image type="content" source="media/android-sdk/open-device-manager.png" alt-text="Administrador de dispositivos":::
   2. :::image type="content" source="media/android-sdk/your-virtual-devices.png" alt-text="Dispositivos virtuales":::
   3. Ejecute la aplicación en el dispositivo seleccionado y compruebe que se registra correctamente con el centro.

      :::image type="content" source="media/android-sdk/device-registration.png" alt-text="Registro de dispositivos":::

      > [!NOTE]
      > En el registro se puede producir un error durante la ejecución inicial hasta que se llama al método `onTokenRefresh()` del servicio de identificación de la instancia. Para iniciar un registro correcto en el centro de notificaciones no tiene más que actualizar.

## <a name="send-a-test-notification"></a>Envío de una notificación de prueba

Puede enviar notificaciones push a su centro de notificaciones desde [Azure Portal](https://portal.azure.com/), como se indica a continuación:

1. En Azure Portal, en la página Notification Hubs del centro, seleccione **Envío de prueba** en la sección **Solución de problemas** .

2. En **Plataformas**, seleccione **Android**.

3. Seleccione **Enviar**. No verá aún ninguna notificación en el dispositivo Android, porque no ha ejecutado la aplicación móvil en él. Después de ejecutar la aplicación móvil, vuelva a seleccionar el botón **Enviar** para ver el mensaje de notificación.

4. Vea el resultado de la operación en la lista de la parte inferior de la página del portal.

   :::image type="content" source="media/android-sdk/notification-hubs-test-send.png" alt-text="Enviar notificación de prueba":::

5. Verá el mensaje de notificación en el dispositivo.

Las notificaciones push se envían normalmente en un servicio back-end como Mobile Apps o ASP.NET mediante una biblioteca compatible. Si no hay disponible ninguna biblioteca para su back-end, también puede usar la API REST directamente para enviar mensajes de notificación.

## <a name="run-the-mobile-app-on-emulator"></a>Ejecución de la aplicación móvil en el emulador

Antes de probar las notificaciones push en un emulador, asegúrese de que la imagen del emulador admita el nivel de API de Google que elija para la aplicación. Si la imagen no admite las API de Google nativas, puede obtener una excepción **SERVICE_NOT_AVAILABLE** .

Además, asegúrese de haber agregado su cuenta de Google al emulador en ejecución en **Configuración** > **Cuentas**. De lo contrario, sus intentos de registrarse con FCM podrían generar una excepción  **AUTHENTICATION_FAILED** .

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se usa Firebase Cloud Messaging para enviar notificaciones de difusión a todos los dispositivos Android registrados en el servicio. Para aprender a enviar notificaciones push a dispositivos específicos, pase al siguiente tutorial:

> [!div class="nextstepaction"]
>[Tutorial: Envío de notificaciones push a usuarios específicos](push-notifications-android-specific-users-firebase-cloud-messaging.md)

A continuación se muestra una lista de otros tutoriales sobre el envío de notificaciones:

- Azure Mobile Apps: Para ver un ejemplo de cómo enviar notificaciones desde un back-end de Mobile Apps integrado con Notification Hubs, consulte  [Incorporación de notificaciones push a la aplicación iOS](/previous-versions/azure/app-service-mobile/app-service-mobile-ios-get-started-push).

- ASP.NET:  [Uso de Notification Hubs para enviar notificaciones push a los usuarios](notification-hubs-aspnet-backend-ios-apple-apns-notification.md).

- SDK de Java para Azure Notification Hubs: Consulte [Uso de Notification Hubs desde Java](notification-hubs-java-push-notification-tutorial.md) para enviar notificaciones desde Java. Esto se probó en Eclipse para el desarrollo de Android.

- PHP: [Uso de Notification Hubs desde PHP](notification-hubs-php-push-notification-tutorial.md).