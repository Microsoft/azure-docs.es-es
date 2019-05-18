---
title: Envío de notificaciones push a dispositivos Android concretos con Azure Notification Hubs y Google Cloud Messaging | Microsoft Docs
description: Aprenda a usar Notification Hubs para enviar notificaciones push a dispositivos Android con Azure Notification Hubs y Google Cloud Messaging.
services: notification-hubs
documentationcenter: android
author: jwargo
manager: patniko
editor: spelluru'
ms.assetid: 3c23cb80-9d35-4dde-b26d-a7bfd4cb8f81
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: tutorial
ms.custom: mvc
ms.date: 01/04/2019
ms.author: jowargo
ms.openlocfilehash: af08d3ca6eaa95663b0bb669f6dc82a13df5ab39
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233114"
---
# <a name="tutorial-push-notifications-to-specific-android-devices-using-azure-notification-hubs-and-google-cloud-messaging-deprecated"></a>Tutorial: Envío de notificaciones push a dispositivos Android concretos con Azure Notification Hubs y Google Cloud Messaging (en desuso)

> [!WARNING]
> A partir del 10 de abril de 2018, Google dejó de ofrecer Google Cloud Messaging (GCM). El servidor de GCM y las API de cliente están en desuso y se quitarán el próximo 29 de mayo de 2019. Para obtener más información, consulte [GCM and FCM Frequently Asked Questions](https://developers.google.com/cloud-messaging/faq) (Preguntas frecuentes de GCM y FCM).

[!INCLUDE [notification-hubs-selector-breaking-news](../../includes/notification-hubs-selector-breaking-news.md)]

## <a name="overview"></a>Información general

En este tutorial se muestra cómo usar Azure Notification Hubs para difundir notificaciones de noticias de última hora en una aplicación Android. Cuando lo complete, podrá registrar las categorías de noticias de última hora en las que esté interesado y recibir solo notificaciones de inserción para esas categorías. Este escenario es un patrón común para muchas aplicaciones en las que las notificaciones tienen que enviarse a grupos de usuarios que han mostrado previamente interés en ellas, por ejemplo, lectores RSS, aplicaciones para aficionados a la música, etc.

Los escenarios de difusión se habilitan mediante la inclusión de una o más *etiquetas* cuando se crea un registro en el Centro de notificaciones. Cuando las notificaciones se envían a una etiqueta, todos los dispositivos registrados para la etiqueta recibirán la notificación. Puesto que las etiquetas son cadenas simples, no tendrán que aprovisionarse antes. Para información sobre las etiquetas, consulte [Expresiones de etiqueta y enrutamiento de Notification Hubs](notification-hubs-tags-segment-push-message.md).

En este tutorial realizará lo siguiente:

> [!div class="checklist"]
> * Agregar una selección de categorías a la aplicación móvil.
> * Registrarse para notificaciones con etiquetas.
> * Enviar notificaciones con etiquetas.
> * Prueba de la aplicación

## <a name="prerequisites"></a>Requisitos previos

Este tutorial se basa en la aplicación que creó en el [Tutorial: Envío de notificaciones push a dispositivos Android con Azure Notification Hubs y Google Cloud Messaging][get-started]. Antes de comenzar este tutorial, realice el [Tutorial: Envío de notificaciones push a dispositivos Android con Azure Notification Hubs y Google Cloud Messaging][get-started].

## <a name="add-category-selection-to-the-app"></a>Adición de una selección de categorías a la aplicación

El primer paso es agregar los elementos de la interfaz de usuario a la actividad principal existente que permiten al usuario seleccionar las categorías para registrar. Las categorías seleccionadas por un usuario se almacenan en el dispositivo. Cuando la aplicación se inicia, se crea un registro de dispositivos en el Centro de notificaciones con las categorías seleccionadas como etiquetas.

1. Abra el archivo `res/layout/activity_main.xml file` y reemplace el contenido por lo siguiente:

    ```xml
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context="com.example.breakingnews.MainActivity"
        android:orientation="vertical">

            <CheckBox
                android:id="@+id/worldBox"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/label_world" />
            <CheckBox
                android:id="@+id/politicsBox"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/label_politics" />
            <CheckBox
                android:id="@+id/businessBox"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/label_business" />
            <CheckBox
                android:id="@+id/technologyBox"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/label_technology" />
            <CheckBox
                android:id="@+id/scienceBox"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/label_science" />
            <CheckBox
                android:id="@+id/sportsBox"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/label_sports" />
            <Button
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:onClick="subscribe"
                android:text="@string/button_subscribe" />
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Hello World!"
                android:id="@+id/text_hello"
            />
    </LinearLayout>
    ```
2. Abra el archivo `res/values/strings.xml` y agregue las siguientes líneas:

    ```xml
    <string name="button_subscribe">Subscribe</string>
    <string name="label_world">World</string>
    <string name="label_politics">Politics</string>
    <string name="label_business">Business</string>
    <string name="label_technology">Technology</string>
    <string name="label_science">Science</string>
    <string name="label_sports">Sports</string>
    ```

    El diseño gráfico de `main_activity.xml` debe ser similar al de la siguiente imagen:

    ![][A1]
3. Cree una clase `Notifications` en el mismo paquete que la clase `MainActivity`.

    ```java
    import java.util.HashSet;
    import java.util.Set;

    import android.content.Context;
    import android.content.SharedPreferences;
    import android.os.AsyncTask;
    import android.util.Log;
    import android.widget.Toast;
    import android.view.View;

    import com.google.android.gms.gcm.GoogleCloudMessaging;
    import com.microsoft.windowsazure.messaging.NotificationHub;

    public class Notifications {
        private static final String PREFS_NAME = "BreakingNewsCategories";
        private GoogleCloudMessaging gcm;
        private NotificationHub hub;
        private Context context;
        private String senderId;

        public Notifications(Context context, String senderId, String hubName,
                                String listenConnectionString) {
            this.context = context;
            this.senderId = senderId;

            gcm = GoogleCloudMessaging.getInstance(context);
            hub = new NotificationHub(hubName, listenConnectionString, context);
        }

        public void storeCategoriesAndSubscribe(Set<String> categories)
        {
            SharedPreferences settings = context.getSharedPreferences(PREFS_NAME, 0);
            settings.edit().putStringSet("categories", categories).commit();
            subscribeToCategories(categories);
        }

        public Set<String> retrieveCategories() {
            SharedPreferences settings = context.getSharedPreferences(PREFS_NAME, 0);
            return settings.getStringSet("categories", new HashSet<String>());
        }

        public void subscribeToCategories(final Set<String> categories) {
            new AsyncTask<Object, Object, Object>() {
                @Override
                protected Object doInBackground(Object... params) {
                    try {
                        String regid = gcm.register(senderId);

                        String templateBodyGCM = "{\"data\":{\"message\":\"$(messageParam)\"}}";

                        hub.registerTemplate(regid,"simpleGCMTemplate", templateBodyGCM,
                            categories.toArray(new String[categories.size()]));
                    } catch (Exception e) {
                        Log.e("MainActivity", "Failed to register - " + e.getMessage());
                        return e;
                    }
                    return null;
                }

                protected void onPostExecute(Object result) {
                    String message = "Subscribed for categories: "
                            + categories.toString();
                    Toast.makeText(context, message,
                            Toast.LENGTH_LONG).show();
                }
            }.execute(null, null, null);
        }

    }
    ```

    Esta clase usa el almacenamiento local para almacenar las categorías de noticias que este dispositivo ha de recibir. También contiene métodos para registrar estas categorías.
4. En su clase `MainActivity`, quite los campos privados de `NotificationHub` y `GoogleCloudMessaging`, y agregue un campo para `Notifications`:

    ```java
    // private GoogleCloudMessaging gcm;
    // private NotificationHub hub;
    private Notifications notifications;
    ```
5. A continuación, en el método `onCreate`, quite la inicialización del campo `hub` y el método `registerWithNotificationHubs`. Seguidamente, agregue las siguientes líneas, que inicializan una instancia de la clase `Notifications`.

    ```java
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mainActivity = this;

        NotificationsManager.handleNotifications(this, NotificationSettings.SenderId,
                MyHandler.class);

        notifications = new Notifications(this, NotificationSettings.SenderId, NotificationSettings.HubName, NotificationSettings.HubListenConnectionString);

        notifications.subscribeToCategories(notifications.retrieveCategories());
    }
    ```

    Confirme que el nombre del centro y la cadena de conexión están configuradas correctamente en la clase NotificationSettings.

    > [!NOTE]
    > Puesto que las credenciales que se distribuyen con una aplicación de cliente no son normalmente seguras, solo debe distribuir la clave para el acceso de escucha con la aplicación cliente. El acceso de escucha permite a la aplicación el registro de notificaciones, pero los registros existentes no pueden modificarse y las notificaciones no se pueden enviar. La clave de acceso completo se usa en un servicio back-end protegido para el envío de notificaciones y el cambio de registros existentes.

6. Luego, agregue las importaciones siguientes:

    ```java
    import android.widget.CheckBox;
    import java.util.HashSet;
    import java.util.Set;
    ```
7. Agregue el siguiente método `subscribe` para controlar el evento clic del botón de suscripción:

    ```java
    public void subscribe(View sender) {
        final Set<String> categories = new HashSet<String>();

        CheckBox world = (CheckBox) findViewById(R.id.worldBox);
        if (world.isChecked())
            categories.add("world");
        CheckBox politics = (CheckBox) findViewById(R.id.politicsBox);
        if (politics.isChecked())
            categories.add("politics");
        CheckBox business = (CheckBox) findViewById(R.id.businessBox);
        if (business.isChecked())
            categories.add("business");
        CheckBox technology = (CheckBox) findViewById(R.id.technologyBox);
        if (technology.isChecked())
            categories.add("technology");
        CheckBox science = (CheckBox) findViewById(R.id.scienceBox);
        if (science.isChecked())
            categories.add("science");
        CheckBox sports = (CheckBox) findViewById(R.id.sportsBox);
        if (sports.isChecked())
            categories.add("sports");

        notifications.storeCategoriesAndSubscribe(categories);
    }
    ```

    Este método crea una lista de categorías y usa la clase `Notifications` para almacenar la lista en el almacenamiento local y registrar las etiquetas correspondientes en el centro de notificaciones. Cuando se modifican las categorías, el registro vuelve a crearse con las nuevas categorías.

La aplicación ahora puede almacenar un conjunto de categorías en el almacenamiento local en el dispositivo y registrarse en el Centro de notificaciones siempre que el usuario cambie la selección de categorías.

## <a name="register-for-notifications"></a>Registro de notificaciones

Estos pasos permiten registrar el Centro de notificaciones en el inicio mediante las categorías que se han almacenado en el almacén local.

> [!NOTE]
> Dado que el identificador de registro asignado por Google Cloud Messaging (GCM) puede cambiar en cualquier momento, debe registrarse para recibir notificaciones frecuentemente para evitar errores de notificación. En este ejemplo se registra la notificación cada vez que se inicia la aplicación. En las aplicaciones que se ejecutan con frecuencia, más de una vez al día, es posible que pueda omitir el registro para conservar el ancho de banda si pasa menos de un día del registro previo.

1. Agregue el siguiente código al final del método `onCreate` de la clase `MainActivity`:

    ```java
    notifications.subscribeToCategories(notifications.retrieveCategories());
    ```

    Este código garantiza que cada vez que la aplicación se inicia, se recuperan las categorías del almacenamiento local y se solicita un registro de estas categorías.
2. Después actualice el método `onStart()` de la clase `MainActivity` de la siguiente manera:

    ```java
    @Override
    protected void onStart() {

        super.onStart();
        isVisible = true;

        Set<String> categories = notifications.retrieveCategories();

        CheckBox world = (CheckBox) findViewById(R.id.worldBox);
        world.setChecked(categories.contains("world"));
        CheckBox politics = (CheckBox) findViewById(R.id.politicsBox);
        politics.setChecked(categories.contains("politics"));
        CheckBox business = (CheckBox) findViewById(R.id.businessBox);
        business.setChecked(categories.contains("business"));
        CheckBox technology = (CheckBox) findViewById(R.id.technologyBox);
        technology.setChecked(categories.contains("technology"));
        CheckBox science = (CheckBox) findViewById(R.id.scienceBox);
        science.setChecked(categories.contains("science"));
        CheckBox sports = (CheckBox) findViewById(R.id.sportsBox);
        sports.setChecked(categories.contains("sports"));
    }
    ```

    Este código actualiza la actividad principal en función del estado de las categorías guardadas anteriormente.

La aplicación está ahora completa y puede almacenar un conjunto de categorías en el almacenamiento local del dispositivo usado para registrarse en el Centro de notificaciones cuando el usuario cambie la selección de categorías. A continuación, defina un back-end que pueda enviar notificaciones de categorías a esta aplicación.

## <a name="send-tagged-notifications"></a>Envío de notificaciones con etiquetas

[!INCLUDE [notification-hubs-send-categories-template](../../includes/notification-hubs-send-categories-template.md)]

## <a name="test-the-app"></a>Prueba de la aplicación

1. En Android Studio, ejecute la aplicación del emulador o dispositivo Android. La interfaz de usuario de la aplicación ofrece un conjunto de elementos de alternancia que le permiten seleccionar las categorías a las que suscribirse.
2. Habilite uno o más elementos de alternancia de las categorías y, a continuación, haga clic en **Suscribirse**. La aplicación convierte las categorías seleccionadas en etiquetas y solicita un nuevo registro de dispositivo para las etiquetas seleccionadas al Centro de notificaciones. Se devuelven las categorías registradas y se muestran en una notificación del sistema.

    ![Suscribirse a categorías](./media/notification-hubs-aspnet-backend-android-breaking-news/subscribe-for-categories.png)
3. Ejecute la aplicación de consola .NET, que envía notificaciones para cada categoría. Las notificaciones para las categorías seleccionadas aparecen como notificaciones del sistema.

    ![Notificaciones de noticias tecnológicas](./media/notification-hubs-aspnet-backend-android-breaking-news/technolgy-news-notification.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial se han enviado notificaciones de difusión a los dispositivos Android concretos a que se han registrado para las categorías. Para aprender a enviar notificaciones push a usuarios específicos, pase al siguiente tutorial:

> [!div class="nextstepaction"]
>[Envío de notificaciones push a usuarios específicos](notification-hubs-aspnet-backend-gcm-android-push-to-user-google-notification.md)

<!-- Images. -->
[A1]: ./media/notification-hubs-aspnet-backend-android-breaking-news/android-breaking-news1.PNG

<!-- URLs.-->
[get-started]: notification-hubs-android-push-notification-google-gcm-get-started.md
[Use Notification Hubs to broadcast localized breaking news]: notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md
[Notify users with Notification Hubs]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started/
[Notification Hubs Guidance]: https://msdn.microsoft.com/library/jj927170.aspx
[Notification Hubs How-To for Windows Store]: https://msdn.microsoft.com/library/jj927172.aspx
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[Azure portal]: https://portal.azure.com
[wns object]: https://go.microsoft.com/fwlink/p/?LinkId=260591
