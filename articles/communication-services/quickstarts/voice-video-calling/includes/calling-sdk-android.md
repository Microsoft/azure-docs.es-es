---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 9/1/2020
ms.author: mikben
ms.openlocfilehash: c0213b050745712a5c77d4861b9cfba4fc953dfd
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932402"
---
## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Un recurso de Communication Services implementado. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- `User Access Token` para habilitar el cliente de llamada. Para más información sobre [cómo obtener `User Access Token`](../../access-tokens.md)
- Opcional: Complete el inicio rápido para [empezar a agregar llamadas a su aplicación](../getting-started-with-calling.md)

## <a name="setting-up"></a>Instalación

### <a name="install-the-package"></a>Instalar el paquete

<!-- TODO: update with instructions on how to download, install and add package to project -->
Busque el nivel de proyecto build.gradle y asegúrese de agregar `mavenCentral()` a la lista de repositorios en `buildscript` y `allprojects`
```groovy
buildscript {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```

```groovy
allprojects {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```
A continuación, en el nivel de módulo build.gradle, agregue las siguientes líneas a la sección de dependencias

```groovy
dependencies {
    ...
    implementation 'com.azure.android:azure-communication-calling:1.0.0-beta.1'
    ...
}

```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de la biblioteca cliente de llamadas de Azure Communication Services:

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| CallClient| CallClient es el punto de entrada principal a la biblioteca cliente de llamadas.|
| CallAgent | CallAgent se usa para iniciar y administrar llamadas. |
| CommunicationUserCredential | CommunicationUserCredential se usa como la credencial de token para crear una instancia de CallAgent.|

## <a name="initialize-the-callclient-create-a-callagent-and-access-the-devicemanager"></a>Inicialización de CallClient, creación de CallAgent y acceso a DeviceManager

Para crear una instancia de `CallAgent` , debe llamar al método `createCallAgent` en una instancia de `CallClient`. De este modo, se devuelve un objeto de instancia de `CallAgent` de manera asincrónica.
El método `createCallAgent` toma `CommunicationUserCredential` como argumento, que encapsula un [token de acceso](../../access-tokens.md).
Para tener acceso a `DeviceManager`, debe crearse primero una instancia de callAgent y, a continuación, puede usar el método `CallClient.getDeviceManager` para obtener DeviceManager.

```java
String userToken = '<user token>';
CallClient callClient = new CallClient();
CommunicationUserCredential tokenCredential = new CommunicationUserCredential(userToken);
android.content.Context appContext = this.getApplicationContext(); // From within an Activity for instance
CallAgent callAgent = await callClient.createCallAgent((appContext, tokenCredential).get();
DeviceManage deviceManager = await callClient.getDeviceManager().get();
```

## <a name="place-an-outgoing-call-and-join-a-group-call"></a>Realización de una llamada saliente y unión a una llamada grupal

Para crear e iniciar una llamada, es necesario llamar al método `CallClient.call()` y proporcionar el `Identifier` de los destinatarios.
Para unirse a una llamada grupal, debe llamar al método `CallClient.join()` y proporcionar el groupId. Los identificadores de grupo deben tener el formato GUID o UUID.

La creación y el inicio de la llamada son sincrónicos. La instancia de llamada le permite suscribirse a todos los eventos de la llamada.

### <a name="place-a-11-call-to-a-user"></a>Realización de una llamada 1:1 a un usuario
Para realizar una llamada a otro usuario de Communication Services, invoque el método `call` en `callAgent` y pase un objeto con la clave `communicationUserId`.
```java
StartCallOptions startCallOptions = new StartCallOptions();
Context appContext = this.getApplicationContext();
CommunicationUser acsUserId = new CommunicationUser(<USER_ID>);
CommunicationUser participants[] = new CommunicationUser[]{ acsUserId };
call oneToOneCall = callAgent.call(appContext, participants, startCallOptions);
```

### <a name="place-a-1n-call-with-users-and-pstn"></a>Realización de una llamada 1:n con usuarios y RTC
> [!WARNING]
> Actualmente, la llamada RTC no está disponible. Para realizar una llamada 1:n a un usuario y un número de RTC, debe especificar el número de teléfono del destinatario.
El recurso de Communication Services debe estar configurado para permitir llamadas RTC:
```java
CommunicationUser acsUser1 = new CommunicationUser(<USER_ID>);
PhoneNumber acsUser2 = new PhoneNumber("<PHONE_NUMBER>");
CommunicationIdentifier participants[] = new CommunicationIdentifier[]{ acsUser1, acsUser2 };
StartCallOptions startCallOptions = new StartCallOptions();
Context appContext = this.getApplicationContext();
Call groupCall = callClient.call(participants, startCallOptions);
```

### <a name="place-a-11-call-with-with-video-camera"></a>Realización de una llamada 1:1 con videocámara
> [!WARNING]
> Actualmente, solo se admite una secuencia de vídeo local saliente. Para realizar una llamada con vídeo, debe enumerar las cámaras locales mediante la API `deviceManager` `getCameraList`.
Una vez que seleccione la cámara deseada, úsela para crear una instancia de `LocalVideoStream` y pásela a `videoOptions` como elemento de la matriz `localVideoStream` a un método `call`.
Una vez que la llamada se conecta, iniciará automáticamente el envío de una secuencia de vídeo desde la cámara seleccionada hasta los demás participantes.
```java
Context appContext = this.getApplicationContext();
VideoDeviceInfo desiredCamera = callClient.getDeviceManager().get().getCameraList().get(0);
LocalVideoStream currentVideoStream = new LocalVideoStream(desiredCamera, appContext);
VideoOptions videoOptions = new VideoOptions(currentVideoStream);

CommunicationUser[] participants = new CommunicationUser[]{ new CommunicationUser("<acs user id>") };
StartCallOptions startCallOptions = new StartCallOptions();
startCallOptions.setVideoOptions(videoOptions);
Call call = callAgent.call(context, participants, startCallOptions);
```

### <a name="join-a-group-call"></a>Unión a una llamada grupal
Para iniciar una llamada grupal nueva o unirse a una en curso, debe llamar al método "join" y pasar un objeto con una propiedad `groupId`. El valor debe ser un GUID.
```java
Context appContext = this.getApplicationContext();
GroupCallContext groupCallContext = new groupCallContext("<GUID>");
JoinCallOptions joinCallOptions = new JoinCallOptions();

call = callAgent.join(context, groupCallContext, joinCallOptions);
```

## <a name="push-notification"></a>Notificación de inserción

### <a name="overview"></a>Información general
Una notificación push móvil es la notificación emergente que recibe en un dispositivo móvil. En el caso de las llamadas, nos centraremos en las notificaciones push VoIP (voz sobre IP). Ofrecemos las funcionalidades necesarias para registrarse para recibir notificaciones push, para administrar las notificaciones push y para anular el registro y dejar de recibir notificaciones push.

### <a name="prerequisite"></a>Requisito previo

En este tutorial se da por supuesto que tiene habilitada la configuración de la cuenta de Firebase con Cloud Messaging (FCM) y que su instancia de Firebase Cloud Messaging está conectada a una instancia de Azure Notification Hubs (ANH). Consulte el artículo sobre cómo [conectar Firebase a Azure](https://docs.microsoft.com/azure/notification-hubs/notification-hubs-android-push-notification-google-fcm-get-started) para obtener más información al respecto.
Además, en el tutorial se da por supuesto que está usando la versión 3.6 de Android Studio o una versión posterior para compilar la aplicación.

Para la aplicación de Android son necesarios un conjunto de permisos a fin de poder recibir mensajes de notificaciones de FCM. En el archivo AndroidManifest.xml, agregue el siguiente conjunto de permisos justo después de *<manifest ...>* o debajo de la etiqueta *</application>*

```XML
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
```

### <a name="register-for-push-notification"></a>Registro de una notificación push

- A fin de registrarse para recibir notificaciones push, la aplicación debe llamar a registerPushNotification() en una instancia de *CallAgent* con un token de registro de dispositivos.

- Cómo obtener el token de registro de dispositivos
1. Asegúrese de agregar la biblioteca cliente de Firebase al archivo *build.gradle* del módulo de aplicación agregando las siguientes líneas en la sección *dependencias* si aún no está ahí:
```
    // Add the client library for Firebase Cloud Messaging
    implementation 'com.google.firebase:firebase-core:16.0.8'
    implementation 'com.google.firebase:firebase-messaging:20.2.4'
```

2. En el archivo *build.gradle* del nivel de proyecto, agregue lo siguiente en la sección *dependencias* si aún no está ahí
```
    classpath 'com.google.gms:google-services:4.3.3'
```

3. Agregue el siguiente complemento al principio del archivo si aún no está ahí
```
apply plugin: 'com.google.gms.google-services'
```

4. Seleccione *Sinc Now* (Sincronizar ahora) en la barra de herramientas

5. Agregue el siguiente fragmento de código para obtener el token de registro de dispositivos generado por la biblioteca cliente de FCM para la instancia de aplicación cliente 
- Agregue estas importaciones en el encabezado de la actividad principal de la instancia. Son necesarias para que el fragmento de código recupere el token
```
import com.google.android.gms.tasks.OnCompleteListener;
import com.google.android.gms.tasks.Task;
import com.google.firebase.iid.FirebaseInstanceId;
import com.google.firebase.iid.InstanceIdResult;
```
- Agregue este fragmento de código para recuperar el token
```
        FirebaseInstanceId.getInstance().getInstanceId()
                .addOnCompleteListener(new OnCompleteListener<InstanceIdResult>() {
                    @Override
                    public void onComplete(@NonNull Task<InstanceIdResult> task) {
                        if (!task.isSuccessful()) {
                            Log.w(TAG, "getInstanceId failed", task.getException());
                            return;
                        }

                        // Get new Instance ID token
                        String deviceToken = task.getResult().getToken();
                        // Log
                        Log.d(TAG, "Device Registration token retrieved successfully");
                    }
                });
```
6. Registre el token de registro de dispositivos con la biblioteca cliente de servicios de llamada para recibir notificaciones push de llamadas entrantes

```java
String deviceRegistrationToken = "some_token";
try {
    callAgent.registerPushNotification(deviceRegistrationToken).get();
}
catch(Exception e) {
    System.out.println("Something went wrong while registering for Incoming Calls Push Notifications.")
}
```

### <a name="push-notification-handling"></a>Manejo de notificaciones push

- Para recibir notificaciones push sobre llamadas entrantes, llame a *handlePushNotification()* en una instancia de *CallAgent* con una carga.

1. Para obtener la carga de FCM, estos son los pasos necesarios:
- Cree un nuevo servicio (Archivo > Nuevo > Servicio > Servicio) que amplíe la clase de biblioteca cliente de Firebase *FirebaseMessagingService* y asegúrese de invalidar el método *onMessageReceived*. Este método es el controlador de eventos al que se llama cuando FCM envía la notificación push a la aplicación.

```java
public class MyFirebaseMessagingService extends FirebaseMessagingService {
    private java.util.Map<String, String> pushNotificationMessageData;

    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        // Check if message contains a notification payload.
        if (remoteMessage.getNotification() != null) {
            Log.d(TAG, "Message Notification Body: " + remoteMessage.getNotification().getBody());
        }
        else {
            pushNotificationMessageData = serializeDictionaryAsJson(remoteMessage.getData());
        }
    }
}
```
- Asimismo, agregue la siguiente definición de servicio al archivo AndroidManifest.xml, en la etiqueta <application>.

```
        <service
            android:name=".MyFirebaseMessagingService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
```

- Una vez recuperada la carga, puede pasarse a la biblioteca cliente de *Communication Services* que se va a administrar llamando al método *handlePushNotification* en una instancia de *CallAgent*.

```java
java.util.Map<String, String> pushNotificationMessageDataFromFCM = remoteMessage.getData();
try {
    callAgent.handlePushNotification(pushNotificationMessageDataFromFCM).get();
}
catch(Exception e) {
    System.out.println("Something went wrong while handling the Incoming Calls Push Notifications.");
}
```
Cuando la administración del mensaje de notificación push se realice correctamente y todos los controladores de eventos se registren correctamente, la aplicación llamará.

### <a name="unregister-push-notification"></a>Anulación del registro de una notificación push

- Las aplicaciones pueden anular el registro de notificaciones push en cualquier momento. Solo debe llamar al método `unregisterPushNotification()` en callAgent.

```java
try {
    callAgent.unregisterPushNotifications().get();
}
catch(Exception e) {
    System.out.println("Something went wrong while un-registering for all Incoming Calls Push Notifications.")
}
```

## <a name="call-management"></a>Administración de llamadas
Durante una llamada, puede acceder a las propiedades de la llamada y realizar varias operaciones para administrar la configuración relacionada con el vídeo y el audio.

### <a name="call-properties"></a>Propiedades de llamada
* Obtenga el identificador único de esta llamada.
```java
String callId = call.getCallId();
```

* Para información sobre los demás participantes de la llamada, revise la colección `remoteParticipant` de la instancia de `call`:
```java
List<RemoteParticipant> remoteParticipants = call.getRemoteParticipants();
```

* La identidad del autor de la llamada si la llamada es entrante.
```java
CommunicationIdentifier callerId = call.getCallerId();
```

* Obtenga el estado de la llamada.
```java
CallState callState = call.getState();
```
Esto devuelve una cadena que representa el estado actual de una llamada:
* "None": estado inicial de la llamada
* "Incoming": indica que hay una llamada entrante que se debe aceptar o rechazar
* "Connecting": estado de transición inicial una vez que se realiza o acepta la llamada
* "Ringing": en el caso de una llamada saliente, indica que se está llamando a los participantes remotos. Aparece con el estado "Incoming" para ellos
* "EarlyMedia": indica un estado en el que se reproduce un anuncio antes de conectar la llamada
* "Connected": se conecta la llamada
* "Hold": la llamada se pone en espera, no fluye ningún elemento multimedia entre el punto de conexión local y los participantes remotos
* "Disconnecting": estado de transición antes de que la llamada pase al estado "Disconnected"
* "Disconnected": estado final de la llamada


* Para saber por qué ha finalizado una llamada, revise la propiedad `callEndReason`.
Contiene código/subcódigo (vínculo TODO a la documentación)
```java
CallEndReason callEndReason = call.getCallEndReason();
int code = callEndReason.getCode();
int subCode = callEndReason.getSubCode();
```

* Para ver si la llamada actual es una llamada entrante, revise la propiedad `isIncoming`:
```java
boolean isIncoming = call.getIsIncoming();
```

*  Para ver si el micrófono actual está silenciado, revise la propiedad `muted`:
```java
boolean muted = call.getIsMicrophoneMuted();
```

* Para revisar las secuencias de vídeo activas, compruebe la colección `localVideoStreams`:
```java
List<LocalVideoStream> localVideoStreams = call.getLocalVideoStreams();
```

### <a name="mute-and-unmute"></a>Silencio y reactivación del sonido
Para silenciar o reactivar el sonido del punto de conexión local, puede usar las API asincrónicas `mute` y `unmute`:
```java
call.mute().get();
call.unmute().get();
```

### <a name="start-and-stop-sending-local-video"></a>Inicio y detención del envío de vídeo local
Para iniciar un vídeo, tiene que enumerar las cámaras con la API `getCameraList` en el objeto `deviceManager`.
Luego, cree una instancia de `LocalVideoStream` pasando la cámara deseada a la API `startVideo` como argumento
```java
VideoDeviceInfo desiredCamera = <get-video-device>;
Context appContext = this.getApplicationContext();
currentVideoStream = new LocalVideoStream(desiredCamera, appContext);
videoOptions = new VideoOptions(currentVideoStream);
Future startVideoFuture = call.startVideo(currentVideoStream);
startVideoFuture.get();
```

Una vez que logre empezar a enviar vídeo, se agregará una instancia de `LocalVideoStream` a la colección `localVideoStreams` en la instancia de llamada.
```java
currentVideoStream == call.getLocalVideoStreams().get(0);
```

Para detener el vídeo local, pase la instancia `localVideoStream` disponible en la colección `localVideoStreams`:
```java
call.stopVideo(localVideoStream).get();
```

Puede cambiar a un dispositivo de cámara distinto mientras se envía vídeo si invoca `switchSource` en una instancia de `localVideoStream`:
```java
localVideoStream.switchSource(source).get();
```

## <a name="remote-participants-management"></a>Administración de participantes remotos

El tipo `RemoteParticipant` representa a todos los participantes remotos y estos están disponibles a través de la colección `remoteParticipants` en una instancia de llamada.

### <a name="list-participants-in-a-call"></a>Enumeración de participantes en una llamada
La colección `remoteParticipants` devuelve una lista de los participantes remotos en una llamada determinada:
```java
List<RemoteParticipant> remoteParticipants = call.getRemoteParticipants(); // [remoteParticipant, remoteParticipant....]
```

### <a name="remote-participant-properties"></a>Propiedades de los participantes remotos
Todo participante remoto especificado tiene un conjunto de propiedades y colecciones asociadas:

* Obtenga el identificador de este participante remoto.
"Identity" es uno de los tipos de "Identifier"
```java
CommunicationIdentifier participantIdentity = remoteParticipant.getId();
```

* Obtenga el estado de este participante remoto.
```java
ParticipantState state = remoteParticipant.getState();
```
El estado puede ser uno de los siguientes:
* "Idle": estado inicial
* "Connecting": estado de transición mientras el participante se conecta a la llamada
* "Connected": el participante se conecta a la llamada
* "Hold": el participante está en espera
* "EarlyMedia": se reproduce un anuncio antes de que el participante se conecte a la llamada
* "Disconnected": estado final; el participante se desconecta de la llamada


* Para saber por qué el participante dejó la llamada, revise la propiedad `callEndReason`:
```java
CallEndReason callEndReason = remoteParticipant.getCallEndReason();
```

* Para comprobar si este participante remoto está silenciado o no, revise la propiedad `isMuted`:
```java
boolean isParticipantMuted = remoteParticipant.getIsMuted();
```

* Para comprobar si este participante remoto está hablando o no, revise la propiedad `isSpeaking`:
```java
boolean isParticipantSpeaking = remoteParticipant.getIsSpeaking();
```

* Para revisar todas las secuencias de vídeo que un participante determinado envía en esta llamada, compruebe la colección `videoStreams`:
```java
List<RemoteVideoStream> videoStreams = remoteParticipant.getVideoStreams(); // [RemoteVideoStream, RemoteVideoStream, ...]
```


### <a name="add-a-participant-to-a-call"></a>Incorporación de un participante a una llamada

Para agregar un participante a una llamada (ya sea un usuario o un número de teléfono), puede invocar `addParticipant`. Esto devolverá de manera sincrónica la instancia del participante remoto.

```java
const acsUser = new CommunicationUser("<acs user id>");
const acsPhone = new PhoneNumber("<phone number>");
RemoteParticipant remoteParticipant1 = call.addParticipant(acsUser);
RemoteParticipant remoteParticipant2 = call.addParticipant(acsPhone);
```

### <a name="remove-participant-from-a-call"></a>Eliminación del participante de una llamada
Para quitar un participante de una llamada (ya sea un usuario o un número de teléfono), puede invocar `removeParticipant`.
Esto se resolverá de manera asincrónica una vez que el participante se quite de la llamada.
El participante también se quitará de la colección `remoteParticipants`.
```java
RemoteParticipant remoteParticipant = call.getParticipants().get(0);
call.removeParticipant(acsUser).get();
call.removeParticipant(acsPhone).get();
```

## <a name="render-remote-participant-video-streams"></a>Representación de secuencias de vídeo de participantes remotos
Para una lista de las secuencias de vídeo y de las secuencias de uso compartido de pantalla de participantes remotos, revise las colecciones `videoStreams`:
```java
RemoteParticipant remoteParticipant = call.getRemoteParticipants().get(0);
RemoteVideoStream remoteParticipantStream = remoteParticipant.getVideoStreams().get(0);
MediaStreamType streamType = remoteParticipantStream.getType(); // of type MediaStreamType.Video or MediaStreamType.ScreenSharing
```
 
Para representar una `RemoteVideoStream` de un participante remoto, debe suscribirse a un evento `OnVideoStreamsUpdated`.
Dentro del evento, el cambio de la propiedad `isAvailable` a true indica que el participante remoto envía actualmente una secuencia. Una vez que esto ocurra, cree una nueva instancia de `Renderer` y, a continuación, cree un nuevo elemento `RendererView` mediante la API `createView` asincrónica y asocie `view.target` en cualquier parte de la interfaz de usuario de la aplicación.
Cada vez que cambia la disponibilidad de una secuencia remota, puede elegir destruir todo el representador, un `RendererView` específico o mantenerlos, pero esto hará que el fotograma del vídeo se vea en blanco.

```java
Renderer remoteVideoRenderer = new Renderer(remoteParticipantStream, appContext);
View uiView = remoteVideoRenderer.createView(new RenderingOptions(ScalingMode.Fit));
layout.addView(uiView);

remoteParticipant.addOnVideoStreamsUpdatedListener(e -> onRemoteParticipantVideoStreamsUpdated(p, e));

void onRemoteParticipantVideoStreamsUpdated(RemoteParticipant participant, RemoteVideoStreamsEvent args) {
    for(RemoteVideoStream stream : args.getAddedRemoteVideoStreams()) {
        if(stream.getIsAvailable()) {
            startRenderingVideo();
        } else {
            renderer.dispose();
        }
    }
}
```

### <a name="remote-video-stream-properties"></a>Propiedades de secuencias de vídeo remotas
La secuencia de vídeo remota tiene un par de propiedades

* `Id`: id. de una secuencia de vídeo remota
```java
int id = remoteVideoStream.getId();
```

* `MediaStreamType`: puede ser "Video" o "ScreenSharing"
```java
MediaStreamType type = remoteVideoStream.getType();
```

* `isAvailable`: indica si el punto de conexión del participante remoto envía secuencias de manera activa
```java
boolean availability = remoteVideoStream.getIsAvailable();
```

### <a name="renderer-methods-and-properties"></a>Métodos y propiedades del representador
Objeto del representador que sigue a las API

* Cree una instancia de `RendererView` que se pueda adjuntar posteriormente en la interfaz de usuario de la aplicación para representar la secuencia de vídeo remota.
```java
// Create a view for a video stream
renderer.createView()
```
* Eliminación del representador y de todos los elementos `RendererView` asociados a este representador
```java
renderer.dispose()
```

* `StreamSize`: tamaño (ancho y alto) de una secuencia de vídeo remota
```java
StreamSize renderStreamSize = remoteVideoStream.getSize();
int width = renderStreamSize.getWidth();
int height = renderStreamSize.getHeight();
```


### <a name="rendererview-methods-and-properties"></a>Métodos y propiedades de RendererView
Al crear `RendererView` puede especificar las propiedades `scalingMode` y `mirrored` que se aplicarán a esta vista: El modo de escalado puede ser "Stretch"| "Crop" | "Fit". Si `mirrored` se establece en `true`, la secuencia representada se volteará de manera vertical.

```java
Renderer remoteVideoRenderer = new Renderer(remoteVideoStream, appContext);
RendererView rendererView = remoteVideoRenderer.createView(new RenderingOptions(ScalingMode.Fit));
```

A continuación, el elemento RendererView creado se puede asociar a la interfaz de usuario de la aplicación mediante el siguiente fragmento de código:
```java
layout.addView(rendererView);
```

Posteriormente podrá actualizar el modo de escalado invocando la API `updateScalingMode` en el objeto RendererView con ScalingMode.Stretch | ScalingMode.Crop | ScalingMode.Fit como argumento.
```java
// Update the scale mode for this view.
rendererView.updateScalingMode(ScalingMode.Crop)
```


## <a name="device-management"></a>Administración de dispositivos

`DeviceManager` permite enumerar los dispositivos locales que se pueden usar en una llamada para transmitir las secuencias de audio o vídeo. También permite solicitar permiso a un usuario para acceder a su micrófono y cámara mediante la API nativa del explorador.

Puede llamar al método `callClient.getDeviceManager()` para acceder a `deviceManager`.
> [!WARNING]
> Actualmente, se debe crear una instancia de un objeto `callAgent` para poder obtener acceso a DeviceManager.

```java
DeviceManager deviceManager = callClient.getDeviceManager().get();
```

### <a name="enumerate-local-devices"></a>Enumeración de los dispositivos locales

Para acceder a los dispositivos locales, puede usar métodos de enumeración en el Administrador de dispositivos. La enumeración es una acción sincrónica.

```java
//  Get a list of available video devices for use.
List<VideoDeviceInfo> localCameras = deviceManager.getCameraList(); // [VideoDeviceInfo, VideoDeviceInfo...]

// Get a list of available microphone devices for use.
List<AudioDeviceInfo> localMicrophones = deviceManager.getMicrophoneList(); // [AudioDeviceInfo, AudioDeviceInfo...]

// Get a list of available speaker devices for use.
List<AudioDeviceInfo> localSpeakers = deviceManager.getSpeakerList(); // [AudioDeviceInfo, AudioDeviceInfo...]
```

### <a name="set-default-microphonespeaker"></a>Establecimiento de micrófono/altavoz predeterminados

El administrador de dispositivos le permite establecer un dispositivo predeterminado que se usará al iniciar una llamada.
Si no se establecen los valores predeterminados del cliente, Communication Services revertirá a los valores predeterminados del sistema operativo.

```java

// Get the microphone device that is being used.
AudioDeviceInfo defaultMicrophone = deviceManager.getMicrophoneList().get(0);

// Set the microphone device to use.
deviceManager.setMicrophone(defaultMicrophone);

// Get the speaker device that is being used.
AudioDeviceInfo defaultSpeaker = deviceManager.getSpeakerList().get(0);

// Set the speaker device to use.
deviceManager.setSpeaker(defaultSpeaker);
```

### <a name="local-camera-preview"></a>Vista previa de la cámara local

Puede usar `DeviceManager` y `Renderer` para empezar a representar secuencias desde la cámara local. Esta secuencia no se enviará a los demás participantes, porque es una fuente de vista previa local. Se trata de una acción asincrónica.

```java
VideoDeviceInfo videoDevice = <get-video-device>;
Context appContext = this.getApplicationContext();
currentVideoStream = new LocalVideoStream(videoDevice, appContext);
videoOptions = new VideoOptions(currentVideoStream);

Renderer previewRenderer = new Renderer(currentVideoStream, appContext);
View uiView previewRenderer.createView(new RenderingOptions(ScalingMode.Fit));

// Attach the renderingSurface to a viewable location on the app at this point
layout.addView(uiView);
```

## <a name="eventing-model"></a>Modelo de eventos
Puede suscribirse a la mayoría de las propiedades y colecciones que se van a notificar al cambiar los valores.

### <a name="properties"></a>Propiedades
Para suscribirse a eventos `property changed`:

```java
// subscribe
PropertyChangedListener callStateChangeListener = new PropertyChangedListener()
{
    @Override
    public void onPropertyChanged(PropertyChangedEvent args)
    {
        Log.d("The call state has changed.");
    }
}
call.addOnCallStateChangedListener(callStateChangeListener);

//unsubscribe
call.removeOnCallStateChangedListener(callStateChangeListener);
```

### <a name="collections"></a>Colecciones
Para suscribirse a eventos `collection updated`:

```java
LocalVideoStreamsChangedListener localVideoStreamsChangedListener = new LocalVideoStreamsChangedListener()
{
    @Override
    public void onLocalVideoStreamsUpdated(LocalVideoStreamsEvent localVideoStreamsEventArgs) {
        Log.d(localVideoStreamsEventArgs.getAddedStreams().size());
        Log.d(localVideoStreamsEventArgs.getRemovedStreams().size());
    }
}
call.addOnLocalVideoStreamsChangedListener(localVideoStreamsChangedListener);
// To unsubscribe
call.removeOnLocalVideoStreamsChangedListener(localVideoStreamsChangedListener);
```
