---
title: 'Asistentes de voz en Windows: Instrucciones sobre la implementación cuando el dispositivo está bloqueado'
titleSuffix: Azure Cognitive Services
description: Instrucciones para implementar la activación por voz y las funcionalidades que se pueden habilitar cuando el dispositivo está bloqueado para una aplicación de agente de voz.
services: cognitive-services
author: cfogg6
manager: trrwilson
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: travisw
ms.custom: devx-track-csharp
ms.openlocfilehash: a529875536c2feafe05695e5d20daed0873a95e6
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934453"
---
# <a name="implementing-voice-assistants-on-windows"></a>Implementación de asistentes de voz en Windows

En esta guía se explican detalles de implementación importantes para crear un asistente de voz en Windows.

## <a name="implementing-voice-activation"></a>Implementación de la activación por voz

Después de [configurar el entorno](how-to-windows-voice-assistants-get-started.md) y aprender [cómo funciona la activación por voz](windows-voice-assistants-overview.md#how-does-voice-activation-work), puede empezar a implementar la activación por voz de su propia aplicación de asistente de voz.

### <a name="registration"></a>Registro

#### <a name="ensure-that-the-microphone-is-available-and-accessible-then-monitor-its-state"></a>Asegúrese de que el micrófono está disponible y es accesible y, a continuación, supervise su estado.

MVA necesita que haya un micrófono presente y sea accesible para poder detectar una activación por voz. Use las clases [AppCapability](https://docs.microsoft.com/uwp/api/windows.security.authorization.appcapabilityaccess.appcapability?view=winrt-18362), [DeviceWatcher](https://docs.microsoft.com/uwp/api/windows.devices.enumeration.devicewatcher?view=winrt-18362) y [MediaCapture](https://docs.microsoft.com/uwp/api/windows.media.capture.mediacapture?view=winrt-18362) para comprobar el acceso de privacidad del micrófono, la presencia del dispositivo y el estado de este (por ejemplo, el volumen y la opción silenciar), respectivamente.

### <a name="register-the-application-with-the-background-service"></a>Registro de la aplicación con el servicio en segundo plano

Para que MVA inicie la aplicación en segundo plano, esta debe registrarse con el servicio en segundo plano. Consulte una guía completa para el registro del servicio en segundo plano [aquí](https://docs.microsoft.com/windows/uwp/launch-resume/register-a-background-task).

### <a name="unlock-the-limited-access-feature"></a>Desbloquear la característica de acceso limitado

Use la clave de la característica de acceso limitado que proporciona Microsoft para desbloquear la característica del asistente de voz. Utilice la clase [LimitedAccessFeature](https://docs.microsoft.com/uwp/api/windows.applicationmodel.limitedaccessfeatures?view=winrt-18362) de Windows SDK para hacerlo.

### <a name="register-the-keyword-for-the-application"></a>Registro de la palabra clave de la aplicación

La aplicación debe registrarse a sí misma, su modelo de palabra clave y su lenguaje con Windows.

Empiece por recuperar el detector de palabras clave. En este ejemplo de código, se recupera el primer detector, pero puede seleccionar el detector que desee seleccionándolo en `configurableDetectors`.

```csharp
private static async Task<ActivationSignalDetector> GetFirstEligibleDetectorAsync()
{
    var detectorManager = ConversationalAgentDetectorManager.Default;
    var allDetectors = await detectorManager.GetAllActivationSignalDetectorsAsync();
    var configurableDetectors = allDetectors.Where(candidate => candidate.CanCreateConfigurations
        && candidate.Kind == ActivationSignalDetectorKind.AudioPattern
        && (candidate.SupportedModelDataTypes.Contains("MICROSOFT_KWSGRAPH_V1")));

    if (configurableDetectors.Count() != 1)
    {
        throw new NotSupportedException($"System expects one eligible configurable keyword spotter; actual is {configurableDetectors.Count()}.");
    }

    var detector = configurableDetectors.First();

    return detector;
}
```

Después de recuperar el objeto ActivationSignalDetector, llame a su método `ActivationSignalDetector.CreateConfigurationAsync` con el identificador de señal, el identificador de modelo y el nombre para mostrar para registrar la palabra clave y recuperar la configuración `ActivationSignalDetectionConfiguration` de la aplicación. Los identificadores de señal y modelo deben ser GUID decididos por el desarrollador y permanecer coherentes con la misma palabra clave.

### <a name="verify-that-the-voice-activation-setting-is-enabled"></a>Comprobación de que la configuración de activación por voz está habilitada

Para usar la activación por voz, el usuario debe habilitarla para su sistema y para su aplicación. Puede encontrar las opciones en la "configuración de privacidad de Activación por voz" de Windows. Para comprobar el estado de la configuración de la activación por voz en la aplicación, use la instancia `ActivationSignalDetectionConfiguration` del registro de la palabra clave. El campo [AvailabilityInfo](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/blob/master/clients/csharp-uwp/UWPVoiceAssistantSample/UIAudioStatus.cs#L128) de `ActivationSignalDetectionConfiguration` contiene un valor de enumeración que describe el estado de configuración de la activación por voz.

### <a name="retrieve-a-conversationalagentsession-to-register-the-app-with-the-mva-system"></a>Recuperación de la clase ConversationalAgentSession para registrar la aplicación con el sistema MVA

`ConversationalAgentSession` es una clase de Windows SDK que permite a la aplicación actualizar Windows con el estado de la aplicación (Inactivo, Detectando, Escuchando, En funcionamiento, Hablando) y recibir eventos, como la detección de la activación, y cambios de estado del sistema, como el bloqueo de pantalla. La recuperación de una instancia de AgentSession también sirve para registrar la aplicación en Windows como activable por voz. Es un procedimiento recomendado mantener una referencia a la clase `ConversationalAgentSession`. Para recuperar la sesión, use la API `ConversationalAgentSession.GetCurrentSessionAsync`.

### <a name="listen-to-the-two-activation-signals-the-onbackgroundactivated-and-onsignaldetected"></a>Escucha de las dos señales de activación: OnBackgroundActivated y OnSignalDetected

Windows señalará a la aplicación cuando detecte una palabra clave de una de estas dos maneras. Si la aplicación no está activa (es decir, no tiene una referencia a una instancia no eliminada de `ConversationalAgentSession`), se iniciará la aplicación y se llamará al método OnBackgroundActivated en el archivo App.xaml.cs de la aplicación. Si el campo `BackgroundActivatedEventArgs.TaskInstance.Task.Name` de los argumentos de evento coincide con la cadena "AgentBackgroundTrigger", el inicio de la aplicación se desencadenó mediante la activación por voz. La aplicación necesita invalidar este método y recuperar una instancia de ConversationalAgentSession para indicar a Windows que ahora está activa. Cuando la aplicación está activa, Windows señala la aparición de una activación por voz mediante el evento `ConversationalAgentSession.OnSignalDetected`. Agregue un controlador de eventos a este evento en cuanto recupere la clase `ConversationalAgentSession`.

## <a name="keyword-verification"></a>Comprobación de la palabra clave

Una vez que la aplicación del agente de voz se ha activado por voz, el siguiente paso es comprobar que la detección de la palabra clave fue válida. Windows no proporciona una solución para la comprobación de palabras clave, pero permite a los asistentes de voz acceder al audio de la hipotética activación y completar la comprobación por sí misma.

### <a name="retrieve-activation-audio"></a>Recuperación del audio de activación

Cree un [AudioGraph](https://docs.microsoft.com/uwp/api/windows.media.audio.audiograph) y páselo a `CreateAudioDeviceInputNodeAsync` de `ConversationalAgentSession`. Esto cargará el búfer de audio del gráfico con el audio que *empezaba unos 3 segundos antes de que se detectara la palabra clave*. Este audio adicional inicial se incluye para dar cabida a una amplia gama de longitudes de palabras clave y velocidades de altavoz. A continuación, controle el evento [QuantumStarted](https://docs.microsoft.com/uwp/api/windows.media.audio.audiograph.quantumstarted?view=winrt-18362) del gráfico de audio para recuperar los datos de audio.

```csharp
var inputNode = await agentSession.CreateAudioDeviceInputNodeAsync(audioGraph);
var outputNode = inputGraph.CreateFrameOutputNode();
inputNode.AddOutgoingConnection(outputNode);
audioGraph.QuantumStarted += OnQuantumStarted;
```

Nota: El audio inicial incluido en el búfer de audio puede hacer que se produzca un error en la comprobación de palabras clave. Para solucionar este problema, recorte el principio del búfer de audio antes de enviar el audio para la comprobación de la palabra clave. Este recorte inicial debe adaptarse a cada asistente.

### <a name="launch-in-the-foreground"></a>Inicio en primer plano

Si la comprobación de la palabra clave se realiza correctamente, la aplicación debe pasar al primer plano para mostrar la interfaz de usuario. Llame a la API `ConversationalAgentSession.RequestForegroundActivationAsync` para mover la aplicación al primer plano.

### <a name="transition-from-compact-view-to-full-view"></a>Transición de vista compacta a vista completa

Cuando la aplicación se activa por primera vez mediante la voz, se inicia en una vista compacta. Lea la [guía de diseño de la versión preliminar de activación de voz](windows-voice-assistants-best-practices.md#design-guidance-for-voice-activation-preview) para obtener instrucciones sobre las diferentes vistas y las transiciones entre estas para los asistentes de voz en Windows.

Para realizar la transición de la vista compacta a la vista de la aplicación completa, use la API ApplicationView `TryEnterViewModeAsync`:

```csharp
var appView = ApplicationView.GetForCurrentView();
await appView.TryEnterViewModeAsync(ApplicationViewMode.Default);
```

## <a name="implementing-above-lock-activation"></a>Implementación de la activación cuando el dispositivo está bloqueado

En los pasos siguientes se describen los requisitos para habilitar un asistente de voz en Windows para que se ejecute cuando el dispositivo está bloqueado, incluidas las referencias al ejemplo de código y las directrices para administrar el ciclo de vida de la aplicación.

Para obtener instrucciones sobre el diseño de experiencias cuando el dispositivo está bloqueado, visite la [guía de procedimientos recomendados](windows-voice-assistants-best-practices.md).

Cuando una aplicación muestra una vista cuando el dispositivo está bloqueado, se considera que está en modo "pantalla completa". Para más información sobre la implementación de una aplicación que usa el modo de pantalla completa, consulte la [documentación del modo de pantalla completa](https://docs.microsoft.com/windows-hardware/drivers/partnerapps/create-a-kiosk-app-for-assigned-access).

### <a name="transitioning-above-lock"></a>Transición cuando el dispositivo está bloqueado

La activación cuando el dispositivo está bloqueado es parecida a la que se produce cuando no lo está. Si no hay instancias activas de la aplicación, se iniciará una nueva instancia en segundo plano y se llamará a `OnBackgroundActivated` en App.xaml.cs. Si hay una instancia de la aplicación, esa instancia recibirá una notificación mediante el evento `ConversationalAgentSession.SignalDetected`.

Si la aplicación no aparece ya cuando el dispositivo está bloqueado, debe llamar a `ConversationalAgentSession.RequestForegroundActivationAsync`. Esto desencadena el método `OnLaunched` de App.xaml.cs, que debe ir a la vista que se mostrará cuando el dispositivo esté bloqueado.

### <a name="detecting-lock-screen-transitions"></a>Detección de transiciones de la pantalla de bloqueo

La biblioteca ConversationalAgent de Windows SDK proporciona una API para que el estado de la pantalla de bloqueo y los cambios de este sean fácilmente accesibles. Para detectar el estado actual de la pantalla de bloqueo, compruebe el campo `ConversationalAgentSession.IsUserAuthenticated`. Para detectar cambios en el estado de bloqueo, agregue un controlador de eventos al evento `SystemStateChanged` del objeto `ConversationalAgentSession`. Se activará siempre que la pantalla cambie de desbloqueada a bloqueada o a la inversa. Si el valor de los argumentos del evento es `ConversationalAgentSystemStateChangeType.UserAuthentication`, el estado de la pantalla de bloqueo ha cambiado.

```csharp
conversationalAgentSession.SystemStateChanged += (s, e) =>
{
    if (e.SystemStateChangeType == ConversationalAgentSystemStateChangeType.UserAuthentication)
    {
        // Handle lock state change
    }
};
```

### <a name="detecting-above-lock-activation-user-preference"></a>Detección de las preferencias de usuario cuando el dispositivo está bloqueado

La entrada de la aplicación de la página de la configuración de privacidad de Activación por voz tiene un botón de alternancia para activar o desactivar la funcionalidad cuando el dispositivo está bloqueado. Para que la aplicación pueda iniciarse cuando el dispositivo esté bloqueado, el usuario tendrá que activar esta opción. El estado de los permisos cuando el dispositivo está bloqueado también se almacena en el objeto ActivationSignalDetectionConfiguration. El estado de AvailabilityInfo.HasLockScreenPermission refleja si el usuario ha dado el permiso para activar la aplicación cuando el dispositivo está bloqueado. Si esta configuración está deshabilitada, una aplicación de voz puede solicitar al usuario que vaya a la página de configuración adecuada en el vínculo "ms-settings:privacy-voiceactivation" con instrucciones sobre cómo habilitar la activación de la aplicación cuando el dispositivo está bloqueado.

## <a name="closing-the-application"></a>Cierre de la aplicación

Para cerrar correctamente la aplicación mediante programación cuando se ha activado con el dispositivo bloqueado o sin bloquear, use la API `WindowService.CloseWindow()`. Esto desencadena todos los métodos de ciclo de vida de la Plataforma universal de Windows, incluido el de suspensión, lo que permite a la aplicación eliminar su instancia de `ConversationalAgentSession` antes de cerrarse.

> [!NOTE]
> La aplicación puede cerrarse sin cerrar la [instancia con el dispositivo sin bloquear](https://docs.microsoft.com/windows-hardware/drivers/partnerapps/create-a-kiosk-app-for-assigned-access#add-a-way-out-of-assigned-access-). En este caso, la vista de la aplicación cuando el dispositivo está bloqueado necesita "limpiarse", lo que garantiza que una vez desbloqueada la pantalla, no habrá ningún controlador de eventos ni tareas que intenten manipular esa vista.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Visite la aplicación de ejemplo del asistente de voz de UWP para ver ejemplos y tutoriales de código](windows-voice-assistants-faq.md#the-uwp-voice-assistant-sample)
