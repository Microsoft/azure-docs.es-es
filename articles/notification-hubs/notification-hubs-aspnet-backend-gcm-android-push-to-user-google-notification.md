---
title: Envío de notificaciones push a usuarios concretos de aplicaciones de Android mediante Azure Notification Hubs | Microsoft Docs
description: Aprenda a enviar notificaciones push a usuarios concretos mediante Azure Notification Hubs.
documentationcenter: android
services: notification-hubs
author: jwargo
manager: patniko
editor: spelluru
ms.assetid: ae0e17a8-9d2b-496e-afd2-baa151370c25
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: tutorial
ms.custom: mvc
ms.date: 01/04/2019
ms.author: jowargo
ms.openlocfilehash: d125e0c0818efbc6ec8f317122859411a37a0d20
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232755"
---
# <a name="tutorial-push-notification-to-specific-android-application-users-by-using-azure-notification-hubs-and-google-cloud-messaging-deprecated"></a>Tutorial: Envío de notificaciones push a usuarios concretos de aplicaciones de Android mediante Azure Notification Hubs y Google Cloud Messaging (en desuso)

> [!WARNING]
> A partir del 10 de abril de 2018, Google dejó de ofrecer Google Cloud Messaging (GCM). El servidor de GCM y las API de cliente están en desuso y se quitarán el próximo 29 de mayo de 2019. Para obtener más información, consulte [GCM and FCM Frequently Asked Questions](https://developers.google.com/cloud-messaging/faq) (Preguntas frecuentes de GCM y FCM).

[!INCLUDE [notification-hubs-selector-aspnet-backend-notify-users](../../includes/notification-hubs-selector-aspnet-backend-notify-users.md)]

Este tutorial muestra cómo puede utilizar los Centros de notificaciones de Azure para enviar notificaciones de inserción a un usuario de aplicaciones determinado en un dispositivo concreto. Se usa un back-end de ASP.NET WebAPI para autenticar clientes y generar notificaciones, tal como se muestra en el artículo de referencia [Administración de registros desde un back-end](notification-hubs-push-notification-registration-management.md#registration-management-from-a-backend). Este tutorial se basa en el centro de notificaciones que creó en el [Tutorial: Envío de notificaciones push a dispositivos Android con Azure Notification Hubs y Google Cloud Messaging](notification-hubs-android-push-notification-google-gcm-get-started.md).

En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Crear el proyecto de API Web de back-end que autentica a los usuarios.  
> * Actualizar la aplicación Android.
> * Prueba de la aplicación

## <a name="prerequisites"></a>Requisitos previos

Complete el [Tutorial: Envío de notificaciones push a dispositivos Android con Azure Notification y Google Cloud Messaging](notification-hubs-android-push-notification-google-gcm-get-started.md) antes de realizar este tutorial.

[!INCLUDE [notification-hubs-aspnet-backend-notifyusers](../../includes/notification-hubs-aspnet-backend-notifyusers.md)]

## <a name="create-the-android-project"></a>Creación del proyecto Android

El siguiente paso consiste en actualizar la aplicación Android creada en el [Tutorial: Envío de notificaciones push a dispositivos Android con Azure Notification Hubs y Google Cloud Messaging](notification-hubs-android-push-notification-google-gcm-get-started.md).

1. Abra su archivo`res/layout/activity_main.xml`, reemplace las definiciones de contenido siguientes:

    Así se agregan nuevos controles de EditText para iniciar sesión como usuario. Además, se agrega un campo para una etiqueta de nombre de usuario que formará parte de las notificaciones que envíe:

    ```xml
    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

    <EditText
        android:id="@+id/usernameText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="@string/usernameHint"
        android:layout_above="@+id/passwordText"
        android:layout_alignParentEnd="true" />
    <EditText
        android:id="@+id/passwordText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="@string/passwordHint"
        android:inputType="textPassword"
        android:layout_above="@+id/buttonLogin"
        android:layout_alignParentEnd="true" />
    <Button
        android:id="@+id/buttonLogin"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/loginButton"
        android:onClick="login"
        android:layout_above="@+id/toggleButtonGCM"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="24dp" />
    <ToggleButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textOn="WNS on"
        android:textOff="WNS off"
        android:id="@+id/toggleButtonWNS"
        android:layout_toLeftOf="@id/toggleButtonGCM"
        android:layout_centerVertical="true" />
    <ToggleButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textOn="GCM on"
        android:textOff="GCM off"
        android:id="@+id/toggleButtonGCM"
        android:checked="true"
        android:layout_centerHorizontal="true"
        android:layout_centerVertical="true" />
    <ToggleButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textOn="APNS on"
        android:textOff="APNS off"
        android:id="@+id/toggleButtonAPNS"
        android:layout_toRightOf="@id/toggleButtonGCM"
        android:layout_centerVertical="true" />
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/editTextNotificationMessageTag"
        android:layout_below="@id/toggleButtonGCM"
        android:layout_centerHorizontal="true"
        android:hint="@string/notification_message_tag_hint" />
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/editTextNotificationMessage"
        android:layout_below="@+id/editTextNotificationMessageTag"
        android:layout_centerHorizontal="true"
        android:hint="@string/notification_message_hint" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/send_button"
        android:id="@+id/sendbutton"
        android:onClick="sendNotificationButtonOnClick"
        android:layout_below="@+id/editTextNotificationMessage"
        android:layout_centerHorizontal="true" />
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        android:id="@+id/text_hello"
    />  
    </RelativeLayout>
    ```
2. Abra el archivo `res/values/strings.xml` y reemplace la definición `send_button` con las siguientes líneas, que redefinen la cadena para `send_button` y agregan cadenas para los demás controles:

    ```xml
    <string name="usernameHint">Username</string>
    <string name="passwordHint">Password</string>
    <string name="loginButton">1. Sign in</string>
    <string name="send_button">2. Send Notification</string>
    <string name="notification_message_hint">Notification message</string>
    <string name="notification_message_tag_hint">Recipient username</string>
    ```

    El diseño gráfico de `main_activity.xml` debe ser similar al de la siguiente imagen:

    ![][A1]
3. Cree una clase denominada `RegisterClient` en el mismo paquete que su clase `MainActivity`. Use el código siguiente para el archivo de la nueva clase.

    ```java
    import java.io.IOException;
    import java.io.UnsupportedEncodingException;
    import java.util.Set;

    import org.apache.http.HttpResponse;
    import org.apache.http.HttpStatus;
    import org.apache.http.client.ClientProtocolException;
    import org.apache.http.client.HttpClient;
    import org.apache.http.client.methods.HttpPost;
    import org.apache.http.client.methods.HttpPut;
    import org.apache.http.client.methods.HttpUriRequest;
    import org.apache.http.entity.StringEntity;
    import org.apache.http.impl.client.DefaultHttpClient;
    import org.apache.http.util.EntityUtils;
    import org.json.JSONArray;
    import org.json.JSONException;
    import org.json.JSONObject;

    import android.content.Context;
    import android.content.SharedPreferences;
    import android.util.Log;

    public class RegisterClient {
        private static final String PREFS_NAME = "ANHSettings";
        private static final String REGID_SETTING_NAME = "ANHRegistrationId";
        private String Backend_Endpoint;
        SharedPreferences settings;
        protected HttpClient httpClient;
        private String authorizationHeader;

        public RegisterClient(Context context, String backendEndpoint) {
            super();
            this.settings = context.getSharedPreferences(PREFS_NAME, 0);
            httpClient =  new DefaultHttpClient();
            Backend_Endpoint = backendEndpoint + "/api/register";
        }

        public String getAuthorizationHeader() {
            return authorizationHeader;
        }

        public void setAuthorizationHeader(String authorizationHeader) {
            this.authorizationHeader = authorizationHeader;
        }

        public void register(String handle, Set<String> tags) throws ClientProtocolException, IOException, JSONException {
            String registrationId = retrieveRegistrationIdOrRequestNewOne(handle);

            JSONObject deviceInfo = new JSONObject();
            deviceInfo.put("Platform", "gcm");
            deviceInfo.put("Handle", handle);
            deviceInfo.put("Tags", new JSONArray(tags));

            int statusCode = upsertRegistration(registrationId, deviceInfo);

            if (statusCode == HttpStatus.SC_OK) {
                return;
            } else if (statusCode == HttpStatus.SC_GONE){
                settings.edit().remove(REGID_SETTING_NAME).commit();
                registrationId = retrieveRegistrationIdOrRequestNewOne(handle);
                statusCode = upsertRegistration(registrationId, deviceInfo);
                if (statusCode != HttpStatus.SC_OK) {
                    Log.e("RegisterClient", "Error upserting registration: " + statusCode);
                    throw new RuntimeException("Error upserting registration");
                }
            } else {
                Log.e("RegisterClient", "Error upserting registration: " + statusCode);
                throw new RuntimeException("Error upserting registration");
            }
        }

        private int upsertRegistration(String registrationId, JSONObject deviceInfo)
                throws UnsupportedEncodingException, IOException,
                ClientProtocolException {
            HttpPut request = new HttpPut(Backend_Endpoint+"/"+registrationId);
            request.setEntity(new StringEntity(deviceInfo.toString()));
            request.addHeader("Authorization", "Basic "+authorizationHeader);
            request.addHeader("Content-Type", "application/json");
            HttpResponse response = httpClient.execute(request);
            int statusCode = response.getStatusLine().getStatusCode();
            return statusCode;
        }

        private String retrieveRegistrationIdOrRequestNewOne(String handle) throws ClientProtocolException, IOException {
            if (settings.contains(REGID_SETTING_NAME))
                return settings.getString(REGID_SETTING_NAME, null);

            HttpUriRequest request = new HttpPost(Backend_Endpoint+"?handle="+handle);
            request.addHeader("Authorization", "Basic "+authorizationHeader);
            HttpResponse response = httpClient.execute(request);
            if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                Log.e("RegisterClient", "Error creating registrationId: " + response.getStatusLine().getStatusCode());
                throw new RuntimeException("Error creating Notification Hubs registrationId");
            }
            String registrationId = EntityUtils.toString(response.getEntity());
            registrationId = registrationId.substring(1, registrationId.length()-1);

            settings.edit().putString(REGID_SETTING_NAME, registrationId).commit();

            return registrationId;
        }
    }
    ```

    Este componente implementa las llamadas REST necesarias para ponerse en contacto con el back-end de la aplicación y registrar las notificaciones push. También almacena localmente los identificadores *registrationIds* creados por el Centro de notificaciones, como se detalla en la sección [Administración de registros desde un back-end](notification-hubs-push-notification-registration-management.md#registration-management-from-a-backend). Usa un token de autorización almacenado localmente al hacer clic en el botón **Iniciar sesión**.
4. En su clase, elimine o convierta en comentario el campo privado de `NotificationHub` y agregue un campo para la clase `RegisterClient` y una cadena para el punto de conexión del back-end de ASP.NET. Asegúrese de reemplazar `<Enter Your Backend Endpoint>` por el punto de conexión del back-end obtenido anteriormente. Por ejemplo, `http://mybackend.azurewebsites.net`.

    ```java
    //private NotificationHub hub;
    private RegisterClient registerClient;
    private static final String BACKEND_ENDPOINT = "<Enter Your Backend Endpoint>";
    ```

5. En la clase `MainActivity`, en el método `onCreate`, quite o convierta en comentario la inicialización del campo `hub` y la llamada al método `registerWithNotificationHubs`. A continuación, agregue código para inicializar una instancia de la clase `RegisterClient` . El método debe contener las siguientes líneas:

    ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mainActivity = this;
        NotificationsManager.handleNotifications(this, NotificationSettings.SenderId, MyHandler.class);
        gcm = GoogleCloudMessaging.getInstance(this);

        //hub = new NotificationHub(HubName, HubListenConnectionString, this);
        //registerWithNotificationHubs();

        registerClient = new RegisterClient(this, BACKEND_ENDPOINT);

        setContentView(R.layout.activity_main);
    }
    ```
6. En la clase `MainActivity`, elimine o convierta en comentario todo el método `registerWithNotificationHubs`. No se usará en este tutorial.
7. Agregue las siguientes instrucciones `import` al archivo `MainActivity.java`.

    ```java
    import android.util.Base64;
    import android.view.View;
    import android.widget.EditText;

    import android.widget.Button;
    import android.widget.ToggleButton;
    import java.io.UnsupportedEncodingException;
    import android.content.Context;
    import java.util.HashSet;
    import android.widget.Toast;
    import org.apache.http.client.ClientProtocolException;
    import java.io.IOException;
    import org.apache.http.HttpStatus;

    import android.os.AsyncTask;
    import org.apache.http.HttpResponse;
    import org.apache.http.client.methods.HttpPost;
    import org.apache.http.entity.StringEntity;
    import org.apache.http.impl.client.DefaultHttpClient;

    import android.app.AlertDialog;
    import android.content.DialogInterface;
    ```
8. Reemplace el código del método onStart por el código siguiente:

    ```java
    super.onStart();
    Button sendPush = (Button) findViewById(R.id.sendbutton);
    sendPush.setEnabled(false);
    ```
9. A continuación, agregue los siguientes métodos para controlar el evento de clic del botón **Iniciar sesión** y enviar notificaciones push.

    ```java
    public void login(View view) throws UnsupportedEncodingException {
        this.registerClient.setAuthorizationHeader(getAuthorizationHeader());

        final Context context = this;
        new AsyncTask<Object, Object, Object>() {
            @Override
            protected Object doInBackground(Object... params) {
                try {
                    String regid = gcm.register(NotificationSettings.SenderId);
                    registerClient.register(regid, new HashSet<String>());
                } catch (Exception e) {
                    DialogNotify("MainActivity - Failed to register", e.getMessage());
                    return e;
                }
                return null;
            }

            protected void onPostExecute(Object result) {
                Button sendPush = (Button) findViewById(R.id.sendbutton);
                sendPush.setEnabled(true);
                Toast.makeText(context, "Logged in and registered.",
                        Toast.LENGTH_LONG).show();
            }
        }.execute(null, null, null);
    }

    private String getAuthorizationHeader() throws UnsupportedEncodingException {
        EditText username = (EditText) findViewById(R.id.usernameText);
        EditText password = (EditText) findViewById(R.id.passwordText);
        String basicAuthHeader = username.getText().toString()+":"+password.getText().toString();
        basicAuthHeader = Base64.encodeToString(basicAuthHeader.getBytes("UTF-8"), Base64.NO_WRAP);
        return basicAuthHeader;
    }

    /**
        * This method calls the ASP.NET WebAPI backend to send the notification message
        * to the platform notification service based on the pns parameter.
        *
        * @param pns     The platform notification service to send the notification message to. Must
        *                be one of the following ("wns", "gcm", "apns").
        * @param userTag The tag for the user who will receive the notification message. This string
        *                must not contain spaces or special characters.
        * @param message The notification message string. This string must include the double quotes
        *                to be used as JSON content.
        */
    public void sendPush(final String pns, final String userTag, final String message)
            throws ClientProtocolException, IOException {
        new AsyncTask<Object, Object, Object>() {
            @Override
            protected Object doInBackground(Object... params) {
                try {

                    String uri = BACKEND_ENDPOINT + "/api/notifications";
                    uri += "?pns=" + pns;
                    uri += "&to_tag=" + userTag;

                    HttpPost request = new HttpPost(uri);
                    request.addHeader("Authorization", "Basic "+ getAuthorizationHeader());
                    request.setEntity(new StringEntity(message));
                    request.addHeader("Content-Type", "application/json");

                    HttpResponse response = new DefaultHttpClient().execute(request);

                    if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                        DialogNotify("MainActivity - Error sending " + pns + " notification",
                            response.getStatusLine().toString());
                        throw new RuntimeException("Error sending notification");
                    }
                } catch (Exception e) {
                    DialogNotify("MainActivity - Failed to send " + pns + " notification ", e.getMessage());
                    return e;
                }

                return null;
            }
        }.execute(null, null, null);
    }
    ```

    El controlador `login` del botón **Iniciar sesión** genera un token de autenticación básica que usa el nombre de usuario y la contraseña de entrada (representa cualquier token que use el esquema de autenticación) y, después, usa `RegisterClient` para llamar al back-end para el registro.

    El método `sendPush` llama al back-end para desencadenar una notificación segura al usuario de acuerdo con la etiqueta del usuario. El servicio de notificaciones de plataforma al que apunta `sendPush` depende de la cadena `pns` transferida.

10. Agregue el siguiente método `DialogNotify` a la clase `MainActivity`.

    ```java
    protected void DialogNotify(String title, String message)
    {
        AlertDialog alertDialog = new AlertDialog.Builder(MainActivity.this).create();
        alertDialog.setTitle(title);
        alertDialog.setMessage(message);
        alertDialog.setButton(AlertDialog.BUTTON_NEUTRAL, "OK",
                new DialogInterface.OnClickListener() {
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                });
        alertDialog.show();
    }
    ```
11. En la clase `MainActivity`, actualice el método `sendNotificationButtonOnClick` para llamar al método `sendPush` con los servicios de notificaciones de plataforma seleccionados del usuario tal como sigue:

    ```java
    /**
    * Send Notification button click handler. This method sends the push notification
    * message to each platform selected.
    *
    * @param v The view
    */
    public void sendNotificationButtonOnClick(View v)
            throws ClientProtocolException, IOException {

        String nhMessageTag = ((EditText) findViewById(R.id.editTextNotificationMessageTag))
                .getText().toString();
        String nhMessage = ((EditText) findViewById(R.id.editTextNotificationMessage))
                .getText().toString();

        // JSON String
        nhMessage = "\"" + nhMessage + "\"";

        if (((ToggleButton)findViewById(R.id.toggleButtonWNS)).isChecked())
        {
            sendPush("wns", nhMessageTag, nhMessage);
        }
        if (((ToggleButton)findViewById(R.id.toggleButtonGCM)).isChecked())
        {
            sendPush("gcm", nhMessageTag, nhMessage);
        }
        if (((ToggleButton)findViewById(R.id.toggleButtonAPNS)).isChecked())
        {
            sendPush("apns", nhMessageTag, nhMessage);
        }
    }
    ```
12. En el archivo `build.gradle` agregue la siguiente línea a la sección `android` después de la sección `buildTypes`.

    ```java
    useLibrary 'org.apache.http.legacy'
    ```
13. Compile el proyecto.

## <a name="test-the-app"></a>Prueba de la aplicación

1. Ejecute la aplicación en un dispositivo o un emulador con Android Studio.
2. En la aplicación de Android, escriba un nombre de usuario y una contraseña. Ambos deben tener el mismo valor de cadena y no pueden contener espacios ni caracteres especiales.
3. En la aplicación de Android, haga clic en **Iniciar sesión**. Espere que aparezca un mensaje de aviso que indique que **se ha iniciado la sesión y se ha registrado**. Así se habilita el botón **Enviar notificación**.

    ![][A2]
4. Haga clic en los botones de alternancia para habilitar todas las plataformas en las que ejecutó la aplicación y registró un usuario.
5. Escriba el nombre del usuario que recibe el mensaje de notificación. Ese usuario debe estar registrado para recibir notificaciones en los dispositivos de destino.
6. Escriba un mensaje para que el usuario lo reciba como un mensaje de notificación push.
7. Haga clic en **Enviar notificación**.  Todos los dispositivos que tienen un registro con la etiqueta de nombre de usuario coincidente recibe la notificación push.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a enviar notificaciones push a usuarios concretos que tienen etiquetas asociadas a sus registros. Para aprender a enviar notificaciones push en función de la ubicación, pase al tutorial siguiente:

> [!div class="nextstepaction"]
>[Envío de notificaciones push basadas en la ubicación](notification-hubs-push-bing-spatial-data-geofencing-notification.md)

[A1]: ./media/notification-hubs-aspnet-backend-android-notify-users/android-notify-users.png
[A2]: ./media/notification-hubs-aspnet-backend-android-notify-users/android-notify-users-enter-password.png
