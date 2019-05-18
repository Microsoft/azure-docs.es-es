---
title: Envío de notificaciones push a aplicaciones de Android con Azure Notification Hubs y Google Cloud Messaging | Microsoft Docs
description: En este tutorial aprenderá a usar Azure Notification Hubs y Google Cloud Messaging para enviar notificaciones push a dispositivos Android.
services: notification-hubs
documentationcenter: android
keywords: notificaciones push,notificación push,notificaciones push android
author: jwargo
manager: patniko
editor: spelluru
ms.assetid: 8268c6ef-af63-433c-b14e-a20b04a0342a
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: tutorial
ms.custom: mvc
ms.date: 01/04/2019
ms.author: jowargo
ms.openlocfilehash: 929977913fdbf0c6f59d69ec536a2638bca7b97c
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232727"
---
# <a name="tutorial-push-notifications-to-android-devices-by-using-azure-notification-hubs-and-google-cloud-messaging-deprecated"></a>Tutorial: Envío de notificaciones push a dispositivos Android con Azure Notification Hubs y Google Cloud Messaging (en desuso)

[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

> [!WARNING]
> A partir del 10 de abril de 2018, Google dejó de ofrecer Google Cloud Messaging (GCM). El servidor de GCM y las API de cliente están en desuso y se quitarán el próximo 29 de mayo de 2019. Para obtener más información, consulte [GCM and FCM Frequently Asked Questions](https://developers.google.com/cloud-messaging/faq) (Preguntas frecuentes de GCM y FCM).

## <a name="overview"></a>Información general

Este tutorial muestra cómo puede usar Azure Notification Hubs para enviar notificaciones push a una aplicación de Android.
Creará una aplicación de Android en blanco que reciba notificaciones push mediante Google Cloud Messaging (GCM).

> [!IMPORTANT]
> Google Cloud Messaging (GCM) está en desuso y se quitará [pronto](https://developers.google.com/cloud-messaging/faq).

> [!IMPORTANT]
> En este tema se muestran las notificaciones push con el servicio Google Cloud Messaging (GCM). Si utiliza el servicio Firebase Cloud Messaging (FCM) de Google, consulte [Envío de notificaciones push a Android con Azure Notification Hubs](notification-hubs-android-push-notification-google-fcm-get-started.md).

El código completo de este tutorial se puede descargar de GitHub, [aquí](https://github.com/Azure/azure-notificationhubs-samples/tree/master/Android/GetStarted).

En este tutorial realizará lo siguiente:

> [!div class="checklist"]
> * Creación de un proyecto que admita Google Cloud Messaging.
> * Creación de un centro de notificaciones
> * Conexión de la aplicación al Centro de notificaciones
> * Prueba de la aplicación

## <a name="prerequisites"></a>Requisitos previos

* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* [Android Studio](https://go.microsoft.com/fwlink/?LinkId=389797).

## <a name="creating-a-project-that-supports-google-cloud-messaging"></a>Creación de un proyecto que admita Google Cloud Messaging

[!INCLUDE [mobile-services-enable-Google-cloud-messaging](../../includes/mobile-services-enable-google-cloud-messaging.md)]

## <a name="create-a-notification-hub"></a>Creación de un centro de notificaciones

[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-gcm-setting-for-the-notification-hub"></a>Configuración de los valores de GCM para el Centro de notificaciones

1. Seleccione **Google (GCM)** en **NOTIFICATION SETTINGS** (CONFIGURACIÓN DE NOTIFICACIÓN).
2. Escriba la **clave de API** que obtuvo en la consola de Google Cloud.
3. Seleccione **Guardar** en la barra de herramientas.

    ![Centros de notificaciones de Azure: Google (GCM)](./media/notification-hubs-android-get-started/notification-hubs-gcm-api.png)

El Centro de notificaciones está configurado para funcionar con GCM y el usuario dispone de las cadenas de conexión necesarias para registrar su aplicación para que reciba y envíe notificaciones push.

## <a id="connecting-app"></a>Conexión de la aplicación al Centro de notificaciones

### <a name="create-a-new-android-project"></a>Creación de un nuevo proyecto de Android

1. En Android Studio, inicie un nuevo proyecto de Android Studio.

   ![Android Studio: nuevo proyecto][13]
2. Elija el factor de forma **Teléfono y tableta** y el **SDK mínimo** que desea admitir. A continuación, haga clic en **Siguiente**.

   ![Android Studio: flujo de trabajo de creación de proyecto][14]
3. Elija **Empty Activity** (Actividad vacía) para la actividad principal, haga clic en **Next** (Siguiente) y, luego, en **Finish** (Finalizar).

### <a name="add-google-play-services-to-the-project"></a>Incorporación de los servicios de Google Play al proyecto

[!INCLUDE [Add Play Services](../../includes/notification-hubs-android-studio-add-google-play-services.md)]

### <a name="adding-azure-notification-hubs-libraries"></a>Incorporación de bibliotecas de Azure Notification Hubs

1. En el archivo `Build.Gradle` de la **aplicación**, agregue las siguientes líneas en la sección **dependencies**.

    ```gradle
    implementation 'com.microsoft.azure:notification-hubs-android-sdk:0.6@aar'
    implementation 'com.microsoft.azure:azure-notifications-handler:1.0.1@aar'
    ```
2. Agregue el repositorio siguiente después de la sección de **dependencias** .

    ```gradle
    repositories {
        maven {
            url "https://dl.bintray.com/microsoftazuremobile/SDK"
        }
    }
    ```

### <a name="updating-the-projects-androidmanifestxml"></a>Actualización del archivo AndroidManifest.xml del proyecto

1. Para admitir GCM, implemente un servicio de escucha Instance ID en el código, que se usará para [obtener tokens de registro](https://developers.google.com/cloud-messaging/android/client#sample-register) con [Instance ID API de Google](https://developers.google.com/instance-id/). En este tutorial, el nombre de la clase es `MyInstanceIDService`.

    Agregue la siguiente definición de servicio al archivo AndroidManifest.xml, en la etiqueta `<application>` . Reemplace el marcador de posición `<your package>` por el nombre real del paquete, que se muestra en la parte superior del archivo `AndroidManifest.xml`.
  
    ```xml
    <service android:name="<your package>.MyInstanceIDService" android:exported="false">
        <intent-filter>
            <action android:name="com.google.android.gms.iid.InstanceID"/>
        </intent-filter>
    </service>
    ```
2. Una vez la aplicación recibe el token de registro GCM de Instance ID API, lo usa para el [registro en el Centro de notificaciones de Azure](notification-hubs-push-notification-registration-management.md). El registro en segundo plano se realiza con un servicio `IntentService` llamado `RegistrationIntentService`. Este servicio es responsable de [actualizar el token de registro GCM](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens).

    Agregue la siguiente definición de servicio al archivo AndroidManifest.xml, en la etiqueta `<application>` . Reemplace el marcador de posición `<your package>` por el nombre real del paquete, que se muestra en la parte superior del archivo `AndroidManifest.xml`.

    ```xml
    <service
        android:name="<your package>.RegistrationIntentService"
        android:exported="false">
    </service>
    ```
3. Defina un receptor para las notificaciones. Agregue la siguiente definición de receptor al archivo AndroidManifest.xml, en la etiqueta `<application>` . Reemplace el marcador de posición `<your package>` por el nombre real del paquete, que se muestra en la parte superior del archivo `AndroidManifest.xml`.

    ```xml
    <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
        android:permission="com.google.android.c2dm.permission.SEND">
        <intent-filter>
            <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            <category android:name="<your package name>" />
        </intent-filter>
    </receiver>
    ```
4. Agregue los siguientes permisos de GCM necesarios bajo la etiqueta `</application>`. Reemplace `<your package>` por el nombre del paquete que se muestra en la parte superior del archivo `AndroidManifest.xml`.

    Para más información sobre estos permisos, consulte [Setup a GCM Client app for Android](https://developers.google.com/cloud-messaging/android/client#manifest)(Configuración de una aplicación de cliente GCM para Android).

    ```xml
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

    <permission android:name="<your package>.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="<your package>.permission.C2D_MESSAGE"/>
    ```

### <a name="adding-code"></a>Incorporación de código

1. En la Vista de proyecto, expanda **app** > **src** > **main** > **java**. Haga clic con el botón derecho en la carpeta del paquete en **java**, haga clic en **Nuevo** y, a continuación, haga clic en **Clase Java**. Agregue una clase nueva llamada `NotificationSettings`.

    ![Android Studio: nueva clase Java][6]

    Actualice los tres marcadores de posición en el código siguiente para la clase `NotificationSettings`:

   * `SenderId`: número de proyecto que obtuvo anteriormente en la [consola de Google Cloud](https://cloud.google.com/console).
   * `HubListenConnectionString`: la cadena de conexión `DefaultListenAccessSignature` para el centro. Para copiar dicha cadena de conexión, haga clic en **Directivas de acceso** en la página **Configuración** del centro de [Azure Portal].
   * `HubName`: use el nombre del centro de notificaciones que aparece en la página del centro de [Azure Portal].

     `NotificationSettings` :

     ```java
     public class NotificationSettings {
        public static String SenderId = "<Your project number>";
        public static String HubName = "<Your HubName>";
        public static String HubListenConnectionString = "<Your default listen connection string>";
     }
     ```
2. Agregue otra clase nueva llamada `MyInstanceIDService`. Será la implementación del servicio de escucha de Instance ID.

    El código de esta clase llama a la clase `IntentService` para [actualizar el token de GCM](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens) en segundo plano.

    ```java
    import android.content.Intent;
    import android.util.Log;
    import com.google.android.gms.iid.InstanceIDListenerService;

    public class MyInstanceIDService extends InstanceIDListenerService {

        private static final String TAG = "MyInstanceIDService";

        @Override
        public void onTokenRefresh() {

            Log.i(TAG, "Refreshing GCM Registration Token");

            Intent intent = new Intent(this, RegistrationIntentService.class);
            startService(intent);
        }
    };
    ```
3. Agregue otra clase nueva, llamada `RegistrationIntentService`, al proyecto. Esta clase implementa `IntentService`, que controla la [actualización del token de GCM](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens) y el [registro en el Centro de notificaciones](notification-hubs-push-notification-registration-management.md).

    Use el siguiente código para esta clase.

    ```java
    import android.app.IntentService;
    import android.content.Intent;
    import android.content.SharedPreferences;
    import android.preference.PreferenceManager;
    import android.util.Log;

    import com.google.android.gms.gcm.GoogleCloudMessaging;
    import com.google.android.gms.iid.InstanceID;
    import com.microsoft.windowsazure.messaging.NotificationHub;

    public class RegistrationIntentService extends IntentService {

        private static final String TAG = "RegIntentService";

        private NotificationHub hub;

        public RegistrationIntentService() {
            super(TAG);
        }

        @Override
        protected void onHandleIntent(Intent intent) {
            SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
            String resultString = null;
            String regID = null;

            try {
                InstanceID instanceID = InstanceID.getInstance(this);
                String token = instanceID.getToken(NotificationSettings.SenderId,
                        GoogleCloudMessaging.INSTANCE_ID_SCOPE);
                Log.i(TAG, "Got GCM Registration Token: " + token);

                // Storing the registration id that indicates whether the generated token has been
                // sent to your server. If it is not stored, send the token to your server,
                // otherwise your server should have already received the token.
                if ((regID=sharedPreferences.getString("registrationID", null)) == null) {
                    NotificationHub hub = new NotificationHub(NotificationSettings.HubName,
                            NotificationSettings.HubListenConnectionString, this);
                    Log.i(TAG, "Attempting to register with NH using token : " + token);

                    regID = hub.register(token).getRegistrationId();

                    // If you want to use tags...
                    // Refer to : https://azure.microsoft.com/documentation/articles/notification-hubs-routing-tag-expressions/
                    // regID = hub.register(token, "tag1", "tag2").getRegistrationId();

                    resultString = "Registered Successfully - RegId : " + regID;
                    Log.i(TAG, resultString);
                    sharedPreferences.edit().putString("registrationID", regID ).apply();
                } else {
                    resultString = "Previously Registered Successfully - RegId : " + regID;
                }
            } catch (Exception e) {
                Log.e(TAG, resultString="Failed to complete token refresh", e);
                // If an exception happens while fetching the new token or updating the registration data
                // on a third-party server, this ensures that we'll attempt the update at a later time.
            }

            // Notify UI that registration has completed.
            if (MainActivity.isVisible) {
                MainActivity.mainActivity.ToastNotify(resultString);
            }
        }
    }
    ```
4. En la clase `MainActivity`, agregue las siguientes instrucciones `import` al principio de la clase.

    ```java
    import com.google.android.gms.common.ConnectionResult;
    import com.google.android.gms.common.GoogleApiAvailability;
    import com.google.android.gms.gcm.*;
    import com.microsoft.windowsazure.notifications.NotificationsManager;
    import android.util.Log;
    import android.widget.TextView;
    import android.widget.Toast;
    import android.content.Intent;
    ```
5. Agregue los siguientes miembros privados en la parte superior de la clase. Este código [comprueba la disponibilidad de Google Play Services, tal como recomienda Google](https://developers.google.com/android/guides/setup#ensure_devices_have_the_google_play_services_apk).

    ```java
    public static MainActivity mainActivity;
    public static Boolean isVisible = false;
    private GoogleCloudMessaging gcm;
    private static final int PLAY_SERVICES_RESOLUTION_REQUEST = 9000;
    private static final String TAG = "MainActivity";
    ```
6. En la clase `MainActivity` , agregue el método siguiente a la disponibilidad de Google Play Services.

    ```java
    /**
        * Check the device to make sure it has the Google Play Services APK. If
        * it doesn't, display a dialog that allows users to download the APK from
        * the Google Play Store or enable it in the device's system settings.
        */
    private boolean checkPlayServices() {
        GoogleApiAvailability apiAvailability = GoogleApiAvailability.getInstance();
        int resultCode = apiAvailability.isGooglePlayServicesAvailable(this);
        if (resultCode != ConnectionResult.SUCCESS) {
            if (apiAvailability.isUserResolvableError(resultCode)) {
                apiAvailability.getErrorDialog(this, resultCode, PLAY_SERVICES_RESOLUTION_REQUEST)
                        .show();
            } else {
                Log.i(TAG, "This device is not supported by Google Play Services.");
                ToastNotify("This device is not supported by Google Play Services.");
                finish();
            }
            return false;
        }
        return true;
    }
    ```
7. En la clase `MainActivity`, agregue el siguiente código, que comprueba Google Play Services antes de llamar a `IntentService` para obtener el token de registro de GCM y realizar el registro en el Centro de notificaciones.

    ```java
    public void registerWithNotificationHubs()
    {
        Log.i(TAG, " Registering with Notification Hubs");

        if (checkPlayServices()) {
            // Start IntentService to register this application with GCM.
            Intent intent = new Intent(this, RegistrationIntentService.class);
            startService(intent);
        }
    }
    ```
8. En el método `OnCreate` de la clase `MainActivity`, agregue el código siguiente para iniciar el proceso de registro cuando se crea la actividad.

    ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mainActivity = this;
        NotificationsManager.handleNotifications(this, NotificationSettings.SenderId, MyHandler.class);
        registerWithNotificationHubs();
    }
    ```
9. Agregue estos métodos adicionales a `MainActivity` para comprobar el estado de la aplicación y el del informe de la aplicación.

    ```java
    @Override
    protected void onStart() {
        super.onStart();
        isVisible = true;
    }

    @Override
    protected void onPause() {
        super.onPause();
        isVisible = false;
    }

    @Override
    protected void onResume() {
        super.onResume();
        isVisible = true;
    }

    @Override
    protected void onStop() {
        super.onStop();
        isVisible = false;
    }

    public void ToastNotify(final String notificationMessage) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Toast.makeText(MainActivity.this, notificationMessage, Toast.LENGTH_LONG).show();
                TextView helloText = (TextView) findViewById(R.id.text_hello);
                helloText.setText(notificationMessage);
            }
        });
    }
    ```
10. El método `ToastNotify` usa el control *"Hello World"* `TextView` para informar del estado y de las notificaciones de forma persistente en la aplicación. En el archivo activity_main.xml, agregue el siguiente identificador para ese control.

    ```xml
    android:id="@+id/text_hello"
    ```
11. Agregue una subclase para el receptor que se define en el archivo AndroidManifest.xml. Agregue otra clase nueva, llamada `MyHandler`, al proyecto.
12. Agregue las siguientes instrucciones de importación en la parte superior de `MyHandler.java`:

    ```java
    import android.app.NotificationManager;
    import android.app.PendingIntent;
    import android.content.Context;
    import android.content.Intent;
    import android.os.Bundle;
    import android.support.v4.app.NotificationCompat;
    import com.microsoft.windowsazure.notifications.NotificationsHandler;
    import android.net.Uri;
    import android.media.RingtoneManager;
    ```
13. Agregue el siguiente código para la clase `MyHandler` para convertirla en una subclase de `com.microsoft.windowsazure.notifications.NotificationsHandler`.

    Este código invalida el método `OnReceive`, por lo que el controlador informa de las notificaciones que se reciben. El controlador también envía la notificación push al administrador de notificaciones de Android mediante el método `sendNotification()` . El método `sendNotification()` se debe ejecutar cuando se reciba una notificación y la aplicación no se esté ejecutando.

    ```java
    public class MyHandler extends NotificationsHandler {
        public static final int NOTIFICATION_ID = 1;
        private NotificationManager mNotificationManager;
        NotificationCompat.Builder builder;
        Context ctx;

        @Override
        public void onReceive(Context context, Bundle bundle) {
            ctx = context;
            String nhMessage = bundle.getString("message");
            sendNotification(nhMessage);
            if (MainActivity.isVisible) {
                MainActivity.mainActivity.ToastNotify(nhMessage);
            }
        }

        private void sendNotification(String msg) {

            Intent intent = new Intent(ctx, MainActivity.class);
            intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);

            mNotificationManager = (NotificationManager)
                    ctx.getSystemService(Context.NOTIFICATION_SERVICE);

            PendingIntent contentIntent = PendingIntent.getActivity(ctx, 0,
                    intent, PendingIntent.FLAG_ONE_SHOT);

            Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
            NotificationCompat.Builder mBuilder =
                    new NotificationCompat.Builder(ctx)
                            .setSmallIcon(R.mipmap.ic_launcher)
                            .setContentTitle("Notification Hub Demo")
                            .setStyle(new NotificationCompat.BigTextStyle()
                                    .bigText(msg))
                            .setSound(defaultSoundUri)
                            .setContentText(msg);

            mBuilder.setContentIntent(contentIntent);
            mNotificationManager.notify(NOTIFICATION_ID, mBuilder.build());
        }
    }
    ```
14. En la barra de menús de Android Studio, haga clic en **Build** (Compilar)  > **Rebuild Project** (Recompilar proyecto) para asegurarse de que no hay errores en el código.

## <a name="testing-your-app"></a>Prueba de la aplicación

### <a name="run-the-mobile-application"></a>Ejecución de la aplicación móvil

1. Ejecute la aplicación y observe que el Id. de registro se informa para un registro correcto.

      ![Prueba en Android: registro de canal][18]
2. Escriba un mensaje de notificación que se enviará a todos los dispositivos Android registrados con el centro.

      ![Prueba en Android - envío de un mensaje][19]

3. Presione **Enviar notificación**. Los dispositivos en los que se ejecuta la aplicación muestran una instancia de `AlertDialog` con el mensaje de notificación push. Los dispositivos en que la aplicación no esté en ejecución, pero que anteriormente se han registrado para recibir notificaciones push, reciben una notificación en el administrador de notificaciones de Android. Para ver los mensajes de notificación, deslice el dedo hacia abajo desde la esquina superior izquierda.

      ![Prueba en Android - notificaciones][21]

### <a name="test-send-push-notifications-from-the-azure-portal"></a>Prueba del envío de notificaciones push desde Azure Portal

Puede probar la recepción de notificaciones push en la aplicación; para ello, envíe alguna desde [Azure Portal].

1. En la sección **Solución de problemas**, seleccione **Envío de prueba**.
2. En **Plataformas**, seleccione **Android**.
3. Seleccione **Enviar** para enviar la notificación de prueba.
4. Confirme que aparece el mensaje de notificación en el dispositivo Android.

    ![Azure Notification Hubs: envío de prueba](./media/notification-hubs-android-get-started/notification-hubs-test-send.png)

[!INCLUDE [notification-hubs-sending-notifications-from-the-portal](../../includes/notification-hubs-sending-notifications-from-the-portal.md)]

#### <a name="push-notifications-in-the-emulator"></a>Notificaciones push en el emulador

Si desea probar notificaciones push en un emulador, asegúrese de que la imagen del emulador admita el nivel de API de Google que elija para la aplicación. Si la imagen no es compatible con las API de Google nativas, aparecerá la excepción **SERVICE\_NOT\_AVAILABLE**.

Además, asegúrese de haber agregado su cuenta de Google al emulador en ejecución en **Configuración** > **Cuentas**. De lo contrario, sus intentos de registrarse con GCM podrían generar la excepción **AUTHENTICATION\_FAILED**.

## <a name="optional-send-push-notifications-directly-from-the-app"></a>(Opcional) Envío de notificaciones push directamente desde la aplicación

Lo habitual es enviar notificaciones mediante un servidor back-end. En algunos casos, puede que quiera enviar notificaciones push directamente desde la aplicación cliente. En esta sección se explica cómo enviar notificaciones desde el cliente mediante la [API de REST del Centro de notificaciones de Azure](https://msdn.microsoft.com/library/azure/dn223264.aspx).

1. En la vista de proyecto de Android Studio, expanda **App** > **src** > **main** > **res** > **layout**. Abra el archivo de diseño `activity_main.xml` y haga clic en la pestaña **Texto** para actualizar el contenido de texto del archivo. Actualícelo con el siguiente código, que agrega nuevos controles `Button` y `EditText` para enviar mensajes de notificación push al Centro de notificaciones. Agregue este código al final, inmediatamente antes de `</RelativeLayout>`.

    ```xml
    <Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/send_button"
    android:id="@+id/sendbutton"
    android:layout_centerVertical="true"
    android:layout_centerHorizontal="true"
    android:onClick="sendNotificationButtonOnClick" />

    <EditText
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:id="@+id/editTextNotificationMessage"
    android:layout_above="@+id/sendbutton"
    android:layout_centerHorizontal="true"
    android:layout_marginBottom="42dp"
    android:hint="@string/notification_message_hint" />
    ```
2. En la vista de proyecto de Android Studio, expanda **App** > **src** > **main** > **res** > **values**. Abra el archivo `strings.xml` y agregue los valores de cadena a los que hacen referencia los nuevos controles `Button` y `EditText`. Agregue las líneas siguientes al final del archivo, justo antes de `</resources>`.

    ```xml
    <string name="send_button">Send Notification</string>
    <string name="notification_message_hint">Enter notification message text</string>
    ```
3. En su archivo `NotificationSetting.java`, agregue la siguiente configuración a la clase `NotificationSettings`.

    Actualice `HubFullAccess` con la cadena de conexión **DefaultFullSharedAccessSignature** de su centro. Esta cadena de conexión se puede copiar desde [Azure Portal]. Para ello, haga clic en **Directivas de acceso** en la página **Configuración** del Centro de notificaciones.

    ```java
    public static String HubFullAccess = "<Enter Your DefaultFullSharedAccess Connection string>";
    ```
4. En el archivo `MainActivity.java`, agregue las siguientes instrucciones `import` al principio del archivo.

    ```java
    import java.io.BufferedOutputStream;
    import java.io.BufferedReader;
    import java.io.InputStreamReader;
    import java.io.OutputStream;
    import java.net.HttpURLConnection;
    import java.net.URL;
    import java.net.URLEncoder;
    import javax.crypto.Mac;
    import javax.crypto.spec.SecretKeySpec;
    import android.util.Base64;
    import android.view.View;
    import android.widget.EditText;
    ```
5. En el archivo `MainActivity.java`, agregue los siguientes miembros al principio de la clase `MainActivity`.

    ```java
    private String HubEndpoint = null;
    private String HubSasKeyName = null;
    private String HubSasKeyValue = null;
    ```
6. Cree un token de firma de acceso a software (SaS) para autenticar una solicitud POST de envío de mensajes al centro de notificaciones. Analice los datos de clave de la cadena de conexión y cree el token SaS como se menciona en la referencia de API REST de [Conceptos comunes](https://msdn.microsoft.com/library/azure/dn495627.aspx). El código siguiente es un ejemplo de implementación.

    En `MainActivity.java`, agregue el método siguiente a la clase `MainActivity` para analizar la cadena de conexión.

    ```java
    /**
        * Example code from https://msdn.microsoft.com/library/azure/dn495627.aspx
        * to parse the connection string so a SaS authentication token can be
        * constructed.
        *
        * @param connectionString This must be the DefaultFullSharedAccess connection
        *                         string for this example.
        */
    private void ParseConnectionString(String connectionString)
    {
        String[] parts = connectionString.split(";");
        if (parts.length != 3)
            throw new RuntimeException("Error parsing connection string: "
                    + connectionString);

        for (int i = 0; i < parts.length; i++) {
            if (parts[i].startsWith("Endpoint")) {
                this.HubEndpoint = "https" + parts[i].substring(11);
            } else if (parts[i].startsWith("SharedAccessKeyName")) {
                this.HubSasKeyName = parts[i].substring(20);
            } else if (parts[i].startsWith("SharedAccessKey")) {
                this.HubSasKeyValue = parts[i].substring(16);
            }
        }
    }
    ```
7. En `MainActivity.java`, agregue el método siguiente a la clase `MainActivity` para crear un token de autenticación SaS.

    ```java
    /**
        * Example code from https://msdn.microsoft.com/library/azure/dn495627.aspx to
        * construct a SaS token from the access key to authenticate a request.
        *
        * @param uri The unencoded resource URI string for this operation. The resource
        *            URI is the full URI of the Service Bus resource to which access is
        *            claimed. For example,
        *            "http://<namespace>.servicebus.windows.net/<hubName>"
        */
    private String generateSasToken(String uri) {

        String targetUri;
        String token = null;
        try {
            targetUri = URLEncoder
                    .encode(uri.toString().toLowerCase(), "UTF-8")
                    .toLowerCase();

            long expiresOnDate = System.currentTimeMillis();
            int expiresInMins = 60; // 1 hour
            expiresOnDate += expiresInMins * 60 * 1000;
            long expires = expiresOnDate / 1000;
            String toSign = targetUri + "\n" + expires;

            // Get an hmac_sha1 key from the raw key bytes
            byte[] keyBytes = HubSasKeyValue.getBytes("UTF-8");
            SecretKeySpec signingKey = new SecretKeySpec(keyBytes, "HmacSHA256");

            // Get an hmac_sha1 Mac instance and initialize with the signing key
            Mac mac = Mac.getInstance("HmacSHA256");
            mac.init(signingKey);

            // Compute the hmac on input data bytes
            byte[] rawHmac = mac.doFinal(toSign.getBytes("UTF-8"));

            // Using android.util.Base64 for Android Studio instead of
            // Apache commons codec
            String signature = URLEncoder.encode(
                    Base64.encodeToString(rawHmac, Base64.NO_WRAP).toString(), "UTF-8");

            // Construct authorization string
            token = "SharedAccessSignature sr=" + targetUri + "&sig="
                    + signature + "&se=" + expires + "&skn=" + HubSasKeyName;
        } catch (Exception e) {
            if (isVisible) {
                ToastNotify("Exception Generating SaS : " + e.getMessage().toString());
            }
        }

        return token;
    }
    ```
8. En `MainActivity.java`, agregue el método siguiente a la clase `MainActivity` para controlar si se hace clic en el botón **Enviar notificación** y envíe el mensaje de notificación push al centro mediante la API de REST integrada.

    ```java
    /**
        * Send Notification button click handler. This method parses the
        * DefaultFullSharedAccess connection string and generates a SaS token. The
        * token is added to the Authorization header on the POST request to the
        * notification hub. The text in the editTextNotificationMessage control
        * is added as the JSON body for the request to add a GCM message to the hub.
        *
        * @param v
        */
    public void sendNotificationButtonOnClick(View v) {
        EditText notificationText = (EditText) findViewById(R.id.editTextNotificationMessage);
        final String json = "{\"data\":{\"message\":\"" + notificationText.getText().toString() + "\"}}";

        new Thread()
        {
            public void run()
            {
                try
                {
                    // Based on reference documentation...
                    // https://msdn.microsoft.com/library/azure/dn223273.aspx
                    ParseConnectionString(NotificationSettings.HubFullAccess);
                    URL url = new URL(HubEndpoint + NotificationSettings.HubName +
                            "/messages/?api-version=2015-01");

                    HttpURLConnection urlConnection = (HttpURLConnection)url.openConnection();

                    try {
                        // POST request
                        urlConnection.setDoOutput(true);

                        // Authenticate the POST request with the SaS token
                        urlConnection.setRequestProperty("Authorization",
                            generateSasToken(url.toString()));

                        // Notification format should be GCM
                        urlConnection.setRequestProperty("ServiceBusNotification-Format", "gcm");

                        // Include any tags
                        // Example below targets 3 specific tags
                        // Refer to : https://azure.microsoft.com/documentation/articles/notification-hubs-routing-tag-expressions/
                        // urlConnection.setRequestProperty("ServiceBusNotification-Tags",
                        //        "tag1 || tag2 || tag3");

                        // Send notification message
                        urlConnection.setFixedLengthStreamingMode(json.length());
                        OutputStream bodyStream = new BufferedOutputStream(urlConnection.getOutputStream());
                        bodyStream.write(json.getBytes());
                        bodyStream.close();

                        // Get response
                        urlConnection.connect();
                        int responseCode = urlConnection.getResponseCode();
                        if ((responseCode != 200) && (responseCode != 201)) {
                            BufferedReader br = new BufferedReader(new InputStreamReader((urlConnection.getErrorStream())));
                            String line;
                            StringBuilder builder = new StringBuilder("Send Notification returned " +
                                    responseCode + " : ")  ;
                            while ((line = br.readLine()) != null) {
                                builder.append(line);
                            }

                            ToastNotify(builder.toString());
                        }
                    } finally {
                        urlConnection.disconnect();
                    }
                }
                catch(Exception e)
                {
                    if (isVisible) {
                        ToastNotify("Exception Sending Notification : " + e.getMessage().toString());
                    }
                }
            }
        }.start();
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial se envían las notificaciones de difusión a todos los dispositivos Android registrados con el back-end. Para aprender a enviar notificaciones push a dispositivos Android específicos, pase al siguiente tutorial:  

 > [!div class="nextstepaction"]
 > [Envío de notificaciones push a dispositivos específicos](notification-hubs-aspnet-backend-android-xplat-segmented-gcm-push-notification.md)

<!-- Images. -->
[6]: ./media/notification-hubs-android-get-started/notification-hub-android-new-class.png
[12]: ./media/notification-hubs-android-get-started/notification-hub-connection-strings.png
[13]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-new-project.png
[14]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-choose-form-factor.png
[15]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app4.png
[16]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app5.png
[17]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app6.png
[18]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-registered.png
[19]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-set-message.png
[20]: ./media/notification-hubs-android-get-started/notification-hub-create-console-app.png
[21]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-received-message.png
[22]: ./media/notification-hubs-android-get-started/notification-hub-scheduler1.png
[23]: ./media/notification-hubs-android-get-started/notification-hub-scheduler2.png
[29]: ./media/mobile-services-android-get-started-push/mobile-eclipse-import-Play-library.png
[30]: ./media/notification-hubs-android-get-started/notification-hubs-debug-hub-gcm.png
[31]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-add-ui.png

<!-- URLs. -->
[Get started with push notifications in Mobile Services]: ../mobile-services-javascript-backend-android-get-started-push.md 
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[Referencing a library project]: https://go.microsoft.com/fwlink/?LinkId=389800
[Notification Hubs Guidance]: https://msdn.microsoft.com/library/jj927170.aspx
[Use Notification Hubs to push notifications to users]: notification-hubs-aspnet-backend-gcm-android-push-to-user-google-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-aspnet-backend-android-xplat-segmented-gcm-push-notification.md
[Azure Portal]: https://portal.azure.com
