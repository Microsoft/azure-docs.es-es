---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 9/1/2020
ms.author: mikben
ms.openlocfilehash: 6922ab2aac8529da8ba55a98f465e3c0e3123b53
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932403"
---
## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Un recurso implementado de Communication Services. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- `User Access Token` para habilitar el cliente de llamada. Para más información sobre [cómo obtener un `User Access Token`](../../access-tokens.md)
- Opcional: Complete el inicio rápido para [empezar a agregar llamadas a su aplicación](../getting-started-with-calling.md)

## <a name="setting-up"></a>Instalación

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Use el comando `npm install` para instalar las bibliotecas cliente común y para llamadas de Azure Communication Services para JavaScript.

```console
npm install @azure/communication-common --save

npm install @azure/communication-calling --save

```

## <a name="object-model"></a>Modelo de objetos

Las clases e interfaces siguientes controlan algunas de las características principales de la biblioteca cliente para llamadas de Azure Communication Services:

| Nombre                             | Descripción                                                                                                                                 |
| ---------------------------------| ------------------------------------------------------------------------------------------------------------------------------------------- |
| CallClient                       | CallClient es el punto de entrada principal a la biblioteca cliente para llamadas.                                                                       |
| CallAgent                        | CallAgent se usa para iniciar y administrar llamadas.                                                                                            |
| AzureCommunicationUserCredential | La clase AzureCommunicationUserCredential implementa la interfaz CommunicationUserCredential interface que se usa para crear una instancia de CallAgent. |


## <a name="initialize-the-callclient-create-callagent-and-access-devicemanager"></a>Inicialización de CallClient, creación de CallAgent y acceso a DeviceManager

Cree una instancia nueva de `CallClient`. Se puede configurar con opciones personalizadas como una instancia del registrador.
Una vez que se crean instancias de `CallClient`, puede crear una instancia de `CallAgent` mediante una llamada al método `createCallAgent` en la instancia de `CallClient`. De este modo, se devuelve un objeto de instancia de `CallAgent` de manera asincrónica.
El método `createCallAgent` toma `CommunicationUserCredential` como argumento, que acepta un [token de acceso de usuario](../../access-tokens.md).
Para acceder a `DeviceManager`, primero se debe crear una instancia de callAgent. Luego, puede usar el método `getDeviceManager` en la instancia de `CallClient` para obtener DeviceManager.

```js
const userToken = '<user token>';
callClient = new CallClient(options);
const tokenCredential = new AzureCommunicationUserCredential(userToken);
const callAgent = await callClient.createCallAgent(tokenCredential);
const deviceManager = await callClient.getDeviceManager()
```

## <a name="place-an-outgoing-call"></a>Realización de una llamada saliente

Para crear e iniciar una llamada, debe usar una de las API en CallAgent y proporcionar un usuario que haya creado a través de la biblioteca cliente de administración de Communication Services.

La creación y el inicio de la llamada es sincrónico. La instancia de llamada le permite suscribirse a eventos de llamada.

## <a name="place-a-11-call-to-a-user-or-a-1n-call-with-users-and-pstn"></a>Realice una llamada de uno a uno a un usuario o una llamada de uno a varios con usuarios y RTC

Para realizar una llamada a otro usuario de Communication Services, invoque el método `call` en `callAgent` y pase la instancia de CommunicationUser que ha [creado con la biblioteca de administración de Communication Services](../../access-tokens.md).

```js
const oneToOneCall = callAgent.call([CommunicationUser]);
```

### <a name="place-a-1n-call-with-users-and-pstn"></a>Realice una llamada de uno a varios con usuarios y RTC
> [!WARNING]
> Llamadas RTC están actualmente en versión preliminar privada. Para acceder, [suscríbase al programa de usuarios pioneros](https://aka.ms/ACS-EarlyAdopter).
Para realizar una llamada de uno a varios a un usuario y a un número de RTC, debe especificar un valor CommunicationUser y un número de teléfono para ambos destinatarios.
El recurso de Communication Services debe estar configurado para permitir llamadas de RTC.
```js

const userCallee = { communicationUserId: <ACS_USER_ID> }
const pstnCallee = { phoneNumber: <PHONE_NUMBER>};
const groupCall = callClient.call([userCallee, pstnCallee], placeCallOptions);

```

### <a name="place-a-11-call-with-with-video-camera"></a>Realice una llamada de uno a uno con videocámara
> [!WARNING]
> Actualmente, no puede haber más de una secuencia de vídeo local saliente.
Para realizar una videollamada, debe enumerar las cámaras locales mediante la API `getCameraList` de deviceManager.
Una vez que seleccione la cámara deseada, úsela para crear una instancia de `LocalVideoStream` y pásela dentro de `videoOptions` como elemento dentro de la matriz `localVideoStream` al método `call`.
Una vez que la llamada se conecta, iniciará automáticamente el envío de una secuencia de vídeo desde la cámara seleccionada hasta los demás participantes.
```js
const deviceManager = await callClient.getDeviceManager();
const videoDeviceInfo = deviceManager.getCameraList()[0];
localVideoStream = new LocalVideoStream(videoDeviceInfo);
const placeCallOptions = {videoOptions: {localVideoStreams:[localVideoStream]}};
const call = callClient.call(['acsUserId'], placeCallOptions);

```

### <a name="join-a-group-call"></a>Unión a una llamada grupal
Para iniciar una llamada grupal nueva o unirse a una en curso, use el método "join" y pase un objeto con una propiedad `groupId`. Este valor debe ser un GUID.
```js

const context = { groupId: <GUID>}
const call = callClient.join(context);

```

## <a name="call-management"></a>Administración de llamadas

Durante una llamada, puede acceder a las propiedades de la llamada y realizar varias operaciones para administrar la configuración relacionada con el vídeo y el audio.

### <a name="call-properties"></a>Propiedades de llamada
* Obtenga el identificador único de esta llamada.
```js

const callId: string = call.id;

```

* Para información sobre los demás participantes de la llamada, revise la colección `remoteParticipant` de la instancia `call`.
```js
const remoteParticipants: RemoteParticipants = call.remoteParticipants;
```

* La identidad del autor de la llamada si la llamada es entrante.
```js

const callerIdentity = call.callerIdentity;

```

* Obtenga el estado de la llamada.
```js

const callState: CallState = call.state;

```
Esto devuelve una cadena que representa el estado actual de una llamada:
* "None": estado inicial de la llamada
* "Incoming': indica que hay una llamada entrante que se debe aceptar o rechazar
* "Connecting": estado de transición inicial una vez que se realiza o acepta la llamada
* "Ringing": en el caso de una llamada saliente, indica que se está llamando a los participantes remotos. Aparece con el estado "Incoming" para ellos
* "EarlyMedia": indica un estado en el que se reproduce un anuncio antes de conectar la llamada
* "Connected": se conecta la llamada
* "Hold": la llamada se pone en espera, no fluye ningún elemento multimedia entre el punto de conexión local y los participantes remotos
* "Disconnecting": estado de transición antes de que la llamada pase al estado "Disconnected"
* "Disconnected": estado final de la llamada


* Para descubrir por qué se finalizó una llamada determinada, revise la propiedad `callEndReason`.
```js

const callEndReason: CallEndReason = call.callEndReason;

```

* Para saber si la llamada actual es una llamada entrante, revise la propiedad `isIncoming`.
```js

const isIncoming: boolean = call.isIncoming;

```

*  Para comprobar si el micrófono actual está silenciado, revise la propiedad `muted`:
```js

const muted: boolean = call.isMicrophoneMuted;

```

* Para ver si la secuencia de uso compartido de pantalla se envía desde un punto de conexión determinado, revise la propiedad `isScreenSharingOn`:
```js

const isScreenSharingOn: boolean = call.isScreenSharingOn;

```

* Para revisar las secuencias de vídeo activas, compruebe la colección `localVideoStreams`:
```js

const localVideoStreams: LocalVideoStream[] = call.localVideoStreams;

```

### <a name="mute-and-unmute"></a>Silencio y reactivación del sonido

Para silenciar o reactivar el sonido del punto de conexión local, puede usar las API asincrónicas `mute` y `unmute`:

```js

//mute local device 
await call.mute();

//unmute device 
await call.unmute();

```

### <a name="start-and-stop-sending-local-video"></a>Inicio y detención del envío de vídeo local


Para iniciar un vídeo, tiene que enumerar las cámaras con el método `getCameraList` en el objeto `deviceManager`. Luego, cree una instancia de `LocalVideoStream` pasando la cámara deseada al método `startVideo` como argumento:


```js
const localVideoStream = new SDK.LocalVideoStream(videoDeviceInfo);
await call.startVideo(localVideoStream);

```

Una vez que logre empezar a enviar vídeo, se agregará una instancia `LocalVideoStream` a la colección `localVideoStreams` en una instancia de llamada.

```js

call.localVideoStreams[0] === localVideoStream;

```

Para detener el vídeo local, pase la instancia `localVideoStream` disponible en la colección `localVideoStreams`:

```js

await call.stopVideo(localVideoStream);

```

Puede cambiar a un dispositivo de cámara distinto mientras se envía vídeo si invoca `switchSource` en una instancia `localVideoStream`:

```js
const source callClient.getDeviceManager().getCameraList()[1];
localVideoStream.switchSource(source);

```

## <a name="remote-participants-management"></a>Administración de participantes remotos

El tipo `RemoteParticipant` representa a todos los participantes remotos, los que están disponibles a través de la colección `remoteParticipants` en una instancia de llamada.

### <a name="list-participants-in-a-call"></a>Lista de los participantes en una llamada
La colección `remoteParticipants` devuelve una lista de los participantes remotos en una llamada determinada:

```js

call.remoteParticipants; // [remoteParticipant, remoteParticipant....]

```

### <a name="remote-participant-properties"></a>Propiedades de los participantes remotos
El participante remoto tiene un conjunto de propiedades y colecciones asociadas.

* Obtenga el identificador de este participante remoto.
"Identity" es uno de los tipos de "Identifier":
```js

const identity: CommunicationUser | PhoneNumber | CallingApplication | UnknownIdentifier;

```

* Obtenga el estado de este participante remoto.
```js

const state: RemoteParticipantState = remoteParticipant.state;
```
El estado puede ser uno de los siguientes:
* "Idle": estado inicial
* "Connecting": estado de transición mientras el participante se conecta a la llamada
* "Connected": el participante se conecta a la llamada
* "Hold": el participante está en espera
* "EarlyMedia": se reproduce un anuncio antes de que el participante se conecte a la llamada
* "Disconnected": estado final; el participante se desconecta de la llamada

Para saber por qué el participante dejó la llamada, revise la propiedad `callEndReason`:
```js

const callEndReason: CallEndReason = remoteParticipant.callEndReason;

```

* Para comprobar si este participante remoto está silenciado o no, revise la propiedad `isMuted`:
```js

const isMuted: boolean = remoteParticipant.isMuted;

```

* Para comprobar si este participante remoto está hablando o no, revise la propiedad `isSpeaking`:
```js

const isSpeaking: boolean = remoteParticipant.isSpeaking;

```

* Para revisar todas las secuencias de vídeo que un participante determinado envía en esta llamada, compruebe la colección `videoStreams`:
```js

const videoStreams: RemoteVideoStream[] = remoteParticipant.videoStreams; // [RemoteVideoStream, ...]

```


### <a name="add-a-participant-to-a-call"></a>Incorporación de un participante a una llamada

Para agregar un participante a una llamada (ya sea un usuario o un número de teléfono), puede invocar `addParticipant`.
Proporcione uno de los tipos de "Identifier".
Esto devolverá de manera sincrónica la instancia del participante remoto.

```js
const userIdentifier = { communicationUserId: <ACS_USER_ID> };
const pstnIdentifier = { phoneNumber: <PHONE_NUMBER>}
const remoteParticipant = call.addParticipant(userIdentifier);
const remoteParticipant = call.addParticipant(pstnIdentifier);

```

### <a name="remove-participant-from-a-call"></a>Eliminación del participante de una llamada

Para quitar un participante de una llamada (ya sea un usuario o un número de teléfono), puede invocar `removeParticipant`.
Debe pasar uno de los tipos de "Identifier". Esto se resolverá de manera asincrónica una vez que el participante se quite de la llamada.
El participante también se quitará de la colección `remoteParticipants`.

```js
const userIdentifier = { communicationUserId: <ACS_USER_ID> };
const pstnIdentifier = { phoneNumber: <PHONE_NUMBER>}
await call.removeParticipant(userIdentifier);
await call.removeParticipant(pstnIdentifier);
```

## <a name="render-remote-participant-video-streams"></a>Representación de secuencias de vídeo de participantes remotos

Para una lista de las secuencias de vídeo y de las secuencias de uso compartido de pantalla de participantes remotos, revise las colecciones `videoStreams`:

```js

const remoteVideoStream: RemoteVideoStream = call.remoteParticipants[0].videoStreams[0];
const streamType: MediaStreamType = remoteVideoStream.type;
```
 
Para representar una `RemoteVideoStream`, se debe suscribir a un evento de `isAvailableChanged`.
Si la propiedad `isAvailable` cambia a `true`, un participante remoto envía una secuencia.
Cuando esto suceda, cree una instancia de `Renderer` y, luego, cree una instancia de `RendererView` nueva a través del método `createView` asincrónico.  Después puede adjuntar `view.target` a cualquier elemento de la interfaz de usuario.
Cada vez que cambia la disponibilidad de una secuencia remota, puede elegir destruir todo el representador, un `RendererView` específico o mantenerlos, pero esto hará que el fotograma del vídeo se vea en blanco.

```js
let renderer: Renderer;
const displayVideo = () => {
    renderer = new Renderer(remoteParticipantStream);
    const view = await renderer.createView();
    htmlElement.appendChild(view.target);
}
remoteParticipantStream.on('availabilityChanged', async () => {
    if (remoteParticipantStream.isAvailable) {
        displayVideo();
    } else {
        renderer.dispose();
    }
});
if (remoteParticipantStream.isAvailable) {
    displayVideo();
}
```

### <a name="remote-video-stream-properties"></a>Propiedades de secuencias de vídeo remotas
Las secuencias de vídeo remotas tienen las propiedades siguientes:

* `Id`: id. de una secuencia de vídeo remota
```js
const id: number = remoteVideoStream.id;
```

* `StreamSize`: tamaño (ancho y alto) de una secuencia de vídeo remota
```js
const size: {width: number; height: number} = remoteVideoStream.size;
```

* `MediaStreamType`: puede ser "Video" o "ScreenSharing"
```js
const type: MediaStreamType = remoteVideoStream.type;
```
* `isAvailable`: indica si el punto de conexión del participante remoto envía secuencias de manera activa
```js
const type: boolean = remoteVideoStream.isAvailable;
```

### <a name="renderer-methods-and-properties"></a>Métodos y propiedades del representador

* Cree una instancia `RendererView` que se pueda adjuntar posteriormente en la interfaz de usuario de la aplicación para representar la secuencia de vídeo remota.
```js
renderer.createView()
```

* Elimine el representador y todas las instancias `RendererView` asociadas.
```js
renderer.dispose()
```


### <a name="rendererview-methods-and-properties"></a>Métodos y propiedades de RendererView
Al crear un objeto `RendererView`, puede especificar las propiedades `scalingMode` y `mirrored`.
El modo de escalado puede ser "Stretch", "Crop" o "Fit". Si se especifica `Mirrored`, la secuencia representada se volteará de manera vertical.

```js
const rendererView: RendererView = renderer.createView({ scalingMode, mirrored });
```
Toda instancia `RendererView` determinada tiene una propiedad `target` que representa la superficie de representación. Se debe adjuntar en la interfaz de usuario de la aplicación:
```js
document.body.appendChild(rendererView.target);
```

Más adelante, puede invocar el método `updateScalingMode` para actualizar el modo de escalado.
```js
view.updateScalingMode('Crop')
```

## <a name="device-management"></a>Administración de dispositivos

`DeviceManager` permite enumerar los dispositivos locales que se pueden usar en una llamada para transmitir las secuencias de audio o vídeo. También permite solicitar permiso a un usuario para acceder a su micrófono y cámara mediante la API nativa del explorador.

Puede llamar al método `callClient.getDeviceManager()` para acceder a `deviceManager`.
> [!WARNING]
> Actualmente, se debe crear una instancia de un objeto `callAgent` para poder obtener acceso a DeviceManager.

```js

const deviceManager = await callClient.getDeviceManager();

```

### <a name="enumerate-local-devices"></a>Enumeración de los dispositivos locales

Para acceder a los dispositivos locales, puede usar métodos de enumeración en el Administrador de dispositivos. La enumeración es una acción sincrónica.

```js

//  Get a list of available video devices for use.
const localCameras = deviceManager.getCameraList(); // [VideoDeviceInfo, VideoDeviceInfo...]

// Get a list of available microphone devices for use.
const localMicrophones = deviceManager.getMicrophoneList(); // [AudioDeviceInfo, AudioDeviceInfo...]

// Get a list of available speaker devices for use.
const localSpeakers = deviceManager.getSpeakerList(); // [AudioDeviceInfo, AudioDeviceInfo...]

```

### <a name="set-default-microphonespeaker"></a>Establecimiento del micrófono o altavoz predeterminado

El administrador de dispositivos le permite establecer un dispositivo predeterminado que se usará al iniciar una llamada.
Si no se establecen los valores predeterminados del cliente, Communication Services revertirá a los valores predeterminados del sistema operativo.

```js

// Get the microphone device that is being used.
const defaultMicrophone = deviceManager.getMicrophone();

// Set the microphone device to use.
await deviceMicrophone.setMicrophone(AudioDeviceInfo);

// Get the speaker device that is being used.
const defaultSpeaker = deviceManager.getSpeaker();

// Set the speaker device to use.
await deviceManager.setSpeaker(AudioDeviceInfo);

```

### <a name="local-camera-preview"></a>Vista previa de la cámara local

Puede usar `DeviceManager` y `Renderer` para empezar a representar secuencias desde la cámara local. Esta secuencia no se enviará a los demás participantes, porque es una fuente de vista previa local. Se trata de una acción asincrónica.

```js
const localVideoDevice = deviceManager().getCameraList()[0];
const localCameraStream = new LocalVideoStream(localVideoDevice);
const renderer = new Renderer(localCameraStream);
const view = await renderer.createView();
document.body.appendChild(view.target);

```

### <a name="request-permission-to-cameramicrophone"></a>Solicitud de permiso para acceder a la cámara o micrófono

Pídale a un usuario que conceda permisos para la cámara o el micrófono de la manera siguiente:

```js
const result = await deviceManager.askDevicePermission(audio: true, video: true);
```
Esto se resolverá de manera asincrónica con un objeto que indica si se concedieron los permisos `audio` y `video`:
```js
console.log(result.audio);
console.log(result.video);
```

Puede revisar el estado actual del permiso de un tipo determinado mediante una llamada a `getPermissionState`:

```js

const result = deviceManager.getPermissionState('Microphone'); // for microphone permission state
const result = deviceManager.getPermissionState('Camera'); // for camera permission state

console.log(result); // 'Granted' | 'Denied' | 'Prompt' | 'Unknown';

```

## <a name="eventing-model"></a>Modelo de eventos

Puede suscribirse a la mayoría de las propiedades y colecciones para recibir notificaciones en caso de cambios en los valores.

### <a name="properties"></a>Propiedades
Para suscribirse a eventos de `property changed`:

```js

const eventHandler = () => {
    // check current value of a property, value is not passed to callback
    console.log(object.property);
};
object.on('propertyNameChanged',eventHandler);

// To unsubscribe:

object.off('propertyNameChanged',eventHandler);

```

### <a name="collections"></a>Colecciones
Para suscribirse a eventos de `collection updated`:

```js

const eventHandler = (e) => {
    // check added elements
    console.log(e.added);
    // check removed elements
    console.log(e.removed);
};
object.on('collectionNameUpdated',eventHandler);

// To unsubscribe:

object.off('collectionNameUpdated',eventHandler);

```
