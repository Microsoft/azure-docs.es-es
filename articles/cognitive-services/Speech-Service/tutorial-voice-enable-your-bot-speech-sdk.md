---
title: 'Tutorial: Habilitación del bot con voz mediante el SDK de voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: En este tutorial, creará un bot de eco con Microsoft Bot Framework, lo implementará en Azure y lo registrará en el canal Direct Line Speech de Bot Framework. A continuación, configurará una aplicación cliente de ejemplo para Windows que le permita hablar con el bot y oír su respuesta.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/25/2020
ms.author: trbye
ms.openlocfilehash: 38a2dd42c010e5b213fd89aa046c7c5b1e1d0a7f
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87758954"
---
# <a name="tutorial-voice-enable-your-bot-using-the-speech-sdk"></a>Tutorial: Habilitación del bot con voz mediante el SDK de voz

Puede usar el servicio de voz para habilitar un bot de chat.

En este tutorial, creará un bot que repite lo que le dijo.
Creará el bot con Microsoft Bot Framework, lo implementará en Azure y lo registrará en el canal Direct Line Speech de Bot Framework.
A continuación, configurará una aplicación cliente de ejemplo para Windows que le permita hablar con el bot y oír su respuesta.

Este tutorial está pensado para desarrolladores que empiezan a trabajar con Azure, con bots de Bot Framework, con Direct Line Speech o con el SDK de voz y quieren crear rápidamente un sistema de trabajo con codificación limitada. No es necesario tener experiencia ni estar familiarizado con estos servicios.

El bot de chat habilitado para voz que realice en este tutorial sigue estos pasos:

1. La aplicación cliente de ejemplo está configurada para conectarse al canal Direct Line Speech y al bot de eco.
1. Cuando el usuario presiona un botón, el audio de voz se transmite desde el micrófono. (O el audio se graba continuamente cuando se usa una palabra clave personalizada).
1. Si se usa una palabra clave personalizada, la detección de palabras clave se produce en el dispositivo local y se canaliza el streaming de audio a la nube.
1. Con el SDK de voz, la aplicación cliente de ejemplo se conecta al canal Direct Line Speech y transmite el audio.
1. Opcionalmente, se produce una comprobación de la palabra clave de mayor precisión en el servicio.
1. El audio se pasa al servicio de reconocimiento de voz y se transcribe en texto.
1. El texto reconocido se pasa al bot de eco como una actividad de Bot Framework.
1. El texto de la respuesta se convierte en audio mediante el servicio de conversión de texto a voz (TTS) y se vuelve a transmitir a la aplicación cliente para su reproducción.

![diagram-tag](media/tutorial-voice-enable-your-bot-speech-sdk/diagram.png "El flujo de canales de voz")

> [!NOTE]
> Los pasos de este tutorial no requieren un servicio de pago. Como nuevo usuario de Azure, podrá usar créditos de su suscripción de prueba de Azure y del nivel gratis del servicio de voz para completar este tutorial.

Este tutorial abarca lo siguiente:
> [!div class="checklist"]
> * Crear nuevos recursos de Azure
> * Compilar, probar e implementar el ejemplo de bot de eco en una instancia de Azure App Service
> * Registrar el bot con el canal Direct Line Speech
> * Compilar y ejecutar el cliente del asistente de voz de Windows para interactuar con un bot de eco
> * Agregar la activación de la palabra clave personalizada
> * Aprender cómo cambiar el idioma de la voz hablada y reconocida

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesitará lo siguiente:

- Un equipo con Windows 10 con un micrófono y altavoces (o auriculares) que funcionen
- [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/) o posterior que tenga instalada la carga de trabajo de **desarrollo web y ASP.NET**
- [Entorno de ejecución de .NET Framework 4.6.1](https://dotnet.microsoft.com/download) o cualquier versión posterior
- Una cuenta de Azure. [Regístrese gratis](https://azure.microsoft.com/free/ai/).
- Una cuenta [GitHub](https://github.com/)
- [Git para Windows](https://git-scm.com/download/win)

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

La aplicación cliente que creará en este tutorial utiliza una serie de servicios de Azure. Para reducir el tiempo de ida y vuelta de las respuestas del bot, deberá asegurarse de que estos servicios se encuentran en la misma región de Azure. En esta sección, se crea un grupo de recursos en la región **Oeste de EE. UU.** Este grupo de recursos se usará al crear recursos individuales para Bot Framework, el canal Direct Line Speech y el servicio de voz.

1. <a href="https://ms.portal.azure.com/#create/Microsoft.ResourceGroup" target="_blank">Creación de un grupo de recursos <span class="docon docon-navigate-external x-hidden-focus"></span></a>
1. Se le pedirá que proporcione algo de información:
   * Establezca **Suscripción** en **Evaluación gratuita** (también puede usar una suscripción existente).
   * Escriba un nombre para el **grupo de recursos**. Se recomienda **SpeechEchoBotTutorial-ResourceGroup**.
   * En la lista desplegable **Región**, seleccione **Oeste de EE. UU.**
1. Haga clic en **Revisar y crear**. Debería ver un banner que indica **Validación superada**.
1. Haga clic en **Crear**. La creación del grupo de recursos tardará unos minutos.
1. Al igual que con los recursos que creará más adelante en este tutorial, es una buena idea anclar este grupo de recursos al panel para facilitar el acceso. Si quiere anclar este grupo de recursos, haga clic en el icono de anclaje del nombre del grupo de recursos.

### <a name="choosing-an-azure-region"></a>Elección de una región de Azure

Si quiere usar una región diferente para este tutorial, estos factores pueden limitar las opciones:

* Asegúrese de que usa una [región de Azure compatible](regions.md#voice-assistants).
* El canal Direct Line Speech utiliza el servicio de conversión de texto a voz, que tiene voces estándar y neuronales. Las voces neuronales se [limitan a regiones específicas de Azure](regions.md#standard-and-neural-voices).
* Las claves de la evaluación gratuitas pueden estar restringidas a una región específica.

Para más información sobre las regiones, consulte [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/).

## <a name="create-resources"></a>Crear recursos

Ahora que tiene un grupo de recursos en una región admitida, el siguiente paso es crear recursos individuales para cada servicio que usará en este tutorial.

### <a name="create-a-speech-service-resource"></a>Creación de recurso del servicio de voz

Para crear un recurso de voz, siga estas instrucciones:

1. <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices" target="_blank">Creación de recurso del servicio de voz <span class="docon docon-navigate-external x-hidden-focus"></span></a>
4. Se le pedirá que proporcione algo de información:
   * Asigne un **nombre** al recurso. Se recomienda **SpeechEchoBotTutorial-Speech**.
   * En **Suscripción**, asegúrese de que esté seleccionada la opción **Evaluación gratuita**.
   * En **Ubicación**, seleccione **Oeste de EE. UU.**
   * En **Plan de tarifa**, seleccione **F0**. Es el plan gratuito.
   * En **Grupo de recursos**, seleccione **SpeechEchoBotTutorial-ResourceGroup**.
5. Después de haber escrito la información necesaria, haga clic en **Crear**. La creación del recurso puede tardar unos minutos.
6. Más adelante en este tutorial necesitará las claves de suscripción para este servicio. Puede acceder a estas claves en cualquier momento desde **Información general** del recurso (Administrar claves) o **Claves**.

En este momento, compruebe que el grupo de recursos (**SpeechEchoBotTutorial-ResourceGroup**) tiene un recurso de voz:

| Nombre | Tipo  | Location |
|------|-------|----------|
| SpeechEchoBotTutorial-Speech | Cognitive Services | Oeste de EE. UU. |

### <a name="create-an-azure-app-service-plan"></a>Crear un plan de Azure App Service

El primer paso consiste en crear un plan de App Service. Un plan de App Service define un conjunto de recursos de proceso para que una aplicación web se ejecute.

1. <a href="https://ms.portal.azure.com/#create/Microsoft.AppServicePlanCreate" target="_blank">Creación de un plan de Azure App Service <span class="docon docon-navigate-external x-hidden-focus"></span></a>
4. Se le pedirá que proporcione algo de información:
   * Establezca **Suscripción** en **Evaluación gratuita** (también puede usar una suscripción existente).
   * En **Grupo de recursos**, seleccione **SpeechEchoBotTutorial-ResourceGroup**.
   * Asigne un **nombre** al recurso. Se recomienda **SpeechEchoBotTutorial-AppServicePlan**
   * En **Sistema operativo**, seleccione **Windows**.
   * En **Región**, seleccione **Oeste de EE. UU.**
   * En **Plan de tarifa**, asegúrese de que está seleccionado **Estándar S1**. Este debería ser el valor predeterminado. Si no es así, asegúrese de establecer el **Sistema operativo** en **Windows** tal y como se describió anteriormente.
5. Haga clic en **Revisar y crear**. Debería ver un banner que indica **Validación superada**.
6. Haga clic en **Crear**. La creación del grupo de recursos tardará unos minutos.

En este momento, compruebe que el grupo de recursos (**SpeechEchoBotTutorial-ResourceGroup**) tiene dos recursos:

| Nombre | Tipo  | Location |
|------|-------|----------|
| SpeechEchoBotTutorial-AppServicePlan | Plan de servicio de aplicación | Oeste de EE. UU. |
| SpeechEchoBotTutorial-Speech | Cognitive Services | Oeste de EE. UU. |

## <a name="build-an-echo-bot"></a>Creación de un bot de eco

Ahora que ha creado algunos recursos, vamos a crear un bot. Vamos a empezar con el ejemplo del bot de eco, que, como su nombre implica, muestra el texto que ha escrito como respuesta. No se preocupe, el código de ejemplo está listo para su uso sin precisar ningún cambio. Está configurado para funcionar con el canal Direct Line Speech, al que nos conectaremos después de haber implementado el bot en Azure.

> [!NOTE]
> Las instrucciones siguientes, así como información adicional sobre el bot de eco, están disponibles en el [archivo Léame del ejemplo en GitHub](https://github.com/microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/02.echo-bot/README.md).

### <a name="run-the-bot-sample-on-your-machine"></a>Ejecución del bot de ejemplo en su máquina

1. Clone el repositorio de ejemplos:

   ```bash
   git clone https://github.com/Microsoft/botbuilder-samples.git
   ```

2. Inicie Visual Studio.
3. En la barra de herramientas, seleccione **Archivo** > **Abrir** > **Proyecto o solución** y abra la solución de proyecto Bot de eco:

   ```
   samples\csharp_dotnetcore\02.echo-bot\EchoBot.sln
   ```

4. Después de haber cargado el proyecto, presione <kbd>F5</kbd> para compilar y ejecutar el proyecto.
5. Se iniciará un explorador y verá una pantalla similar a esta.
    > [!div class="mx-imgBorder"]
    > [![echobot-running-on-localhost](media/tutorial-voice-enable-your-bot-speech-sdk/echobot-running-on-localhost.png "Bot de eco que se ejecuta en un host local")](media/tutorial-voice-enable-your-bot-speech-sdk/echobot-running-on-localhost.png#lightbox)

### <a name="test-the-bot-sample-with-the-bot-framework-emulator"></a>Prueba del ejemplo de bot con Bot Framework Emulator

[Bot Framework Emulator](https://github.com/microsoft/botframework-emulator) es una aplicación de escritorio que permite que los desarrolladores de bots prueben y depuren sus bots de manera local (o remota mediante un túnel). Emulator acepta texto escrito como entrada (no voz). El bot también responderá con texto. Siga estos pasos para usar Bot Framework Emulator para probar la ejecución local de Bot de eco, con entrada de texto y salida de texto. Después de implementar el bot de Azure, se probará con entrada de voz y salida de voz.

1. Instalación de [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases/latest) versión 4.3.0 o superior
2. Inicie Bot Framework Emulator y abra el bot:
   * **Archivo** -> **Open Bot** (Abrir bot).
3. Escriba la dirección URL del bot. Por ejemplo:

   ```
   http://localhost:3978/api/messages
   ```
   y presione "Conectar".
4. El bot debe saludarle con el mensaje de bienvenida. de la directiva). Escriba un mensaje de texto y confirme que obtiene una respuesta del bot.
5. Este es el aspecto que podría tener un intercambio de comunicación con una instancia de Bot de eco: [![bot-framework-emulator](media/tutorial-voice-enable-your-bot-speech-sdk/bot-framework-emulator.png "Bot Framework Emulator")](media/tutorial-voice-enable-your-bot-speech-sdk/bot-framework-emulator.png#lightbox)

## <a name="deploy-your-bot-to-an-azure-app-service"></a>Implementación del bot en Azure App Service

El siguiente paso es implementar el bot de eco en Azure. Hay varias maneras de implementar un bot, pero en este tutorial nos centraremos en publicarlo directamente desde Visual Studio.

> [!NOTE]
> También, puede implementar un bot con la [CLI de Azure](https://docs.microsoft.com/azure/bot-service/bot-builder-deploy-az-cli) y las [plantillas de implementación](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/adaptive-dialog/03.core-bot).

> [!NOTE]
> Si la opción **Publicar...** no aparece al realizar los pasos siguientes, use el Instalador de Visual Studio para agregar la carga de trabajo de **desarrollo web y ASP.NET**.

1. En Visual Studio, abra el bot de eco que se ha configurado para su uso con el canal Direct Line Speech:

   ```
   samples\csharp_dotnetcore\02.echo-bot\EchoBot.sln
   ```

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **EchoBot** y seleccione **Publicar...**
1. Se abre una nueva ventana denominada **Publicar**.
1. Seleccione **Azure**, haga clic en **Siguiente**, seleccione **Azure App Service (Windows)** , haga clic en **Siguiente** y, a continuación, haga clic en **Crear una nueva Azure App Service...** mediante el signo más de color verde.
1. Cuando aparezca la ventana **App Service (Windows)** :
   * Haga clic en **Agregar una cuenta** e inicie sesión con las credenciales de la cuenta de Azure. Si ya ha iniciado sesión, seleccione la cuenta que desee en la lista desplegable.
   * En **Nombre**, deberá escribir un nombre único global para el bot. Este nombre se usa para crear una dirección URL única del bot. Se rellenará un valor predeterminado que incluye la fecha y la hora (por ejemplo: "EchoBot20190805125647"). En este tutorial, utilice los valores predeterminados.
   * En **Suscripción**, establézcala en **Evaluación gratuita**.
   * En **Grupo de recursos**, seleccione **SpeechEchoBotTutorial-ResourceGroup**.
   * En **Plan de hospedaje**, seleccione **SpeechEchoBotTutorial-AppServicePlan**.
1. Haga clic en **Crear**. En la pantalla final del asistente, haga clic en **Finalizar**.
1. Haga clic en **Publicar** en el lado derecho de la pantalla Publicar. Visual Studio implementa el bot en Azure.
1. Debería ver un mensaje de operación correcta en la ventana de salida de Visual Studio que tiene el siguiente aspecto:

   ```
   Publish Succeeded.
   Web App was published successfully https://EchoBot20190805125647.azurewebsites.net/
   ```

1. Debería abrirse el explorador predeterminado y mostrar una página que diga: "El bot está listo".
1. En este punto, compruebe el grupo de recursos **SpeechEchoBotTutorial-ResourceGroup** en Azure Portal y confirme estos tres recursos:

| Nombre | Tipo  | Location |
|------|-------|----------|
| EchoBot20190805125647 | App Service | Oeste de EE. UU. |
| SpeechEchoBotTutorial-AppServicePlan | Plan de App Service | Oeste de EE. UU. |
| SpeechEchoBotTutorial-Speech | Cognitive Services | Oeste de EE. UU. |

## <a name="enable-web-sockets"></a>Habilitación de sockets web

Tendrá que hacer un pequeño cambio de configuración para que el bot pueda comunicarse con el canal Direct Line Speech mediante sockets web. Siga estos pasos para habilitar los sockets web:

1. Vaya a [Azure Portal](https://portal.azure.com) y haga clic en su App Service. El recurso debe tener un nombre similar a **EchoBot20190805125647** (el nombre único de la aplicación).
2. En el panel de navegación izquierdo, en **Configuración**, haga clic en **Configuración**.
3. Seleccione la pestaña **Configuración general**.
4. Busque el botón de alternancia para **Sockets web** y establézcalo en **Activado**.
5. Haga clic en **Save**(Guardar).

> [!TIP]
> Puede usar los controles que se encuentran en la parte superior de la página de Azure App Service para detener o reiniciar el servicio. Esto puede resultar útil para solucionar problemas.

## <a name="create-a-channel-registration"></a>Creación de un registro de canal

Ahora que ha creado una instancia de Azure App Service para hospedar el bot, el siguiente paso es crear un **registro de canales de bot**. La creación de un registro de canal es un requisito previo para registrar el bot con canales de Bot Framework, incluido el canal Direct Line Speech. Para más información sobre cómo los bots usan los canales, consulte [Conexión de un bot a canales](https://docs.microsoft.com/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0).

1. <a href="https://ms.portal.azure.com/#create/Microsoft.BotServiceConnectivityGalleryPackage" target="_blank">Creación de un registro de canales de bot de Azure <span class="docon docon-navigate-external x-hidden-focus"></span></a>
2. Se le pedirá que proporcione algo de información:
   * En **Bot handle** (Identificador de bot), escriba **SpeechEchoBotTutorial-BotRegistration-####** y reemplace **####** por el número que prefiera. Tenga en cuenta que el identificador de bot debe ser único globalmente. Si especifica un identificador de bot pero aparece el mensaje de error _The requested bot ID is not available_ (El identificador de bot solicitado no está disponible), elija otro número. En los ejemplos siguientes usamos el 8726.
   * En **Suscripción**, seleccione **Evaluación gratuita**.
   * En **Grupo de recursos**, seleccione **SpeechEchoBotTutorial-ResourceGroup**.
   * En **Ubicación**, seleccione **Oeste de EE. UU.**
     * En **Plan de tarifa**, seleccione **F0**.
     * En **Messaging endpoint** (Punto de conexión de mensajería), escriba la dirección URL de la aplicación web con la ruta de acceso `/api/messages` anexada al final. Por ejemplo, si el nombre de la aplicación globalmente único era **EchoBot20190805125647**, el punto de conexión de mensajería sería `https://EchoBot20190805125647.azurewebsites.net/api/messages/`.
     * En **Application Insights**, puede establecerlo en **Desconectar**. Para más información, consulte [Análisis del bot](https://docs.microsoft.com/azure/bot-service/bot-service-manage-analytics?view=azure-bot-service-4.0).
     * Ignore **Auto create App ID and password** (Creación automática del identificador y contraseña de la aplicación).
5. En la parte inferior de la hoja **Registro de canales de bot**, haga clic en **Crear**.

En este punto, compruebe el grupo de recursos **SpeechEchoBotTutorial-ResourceGroup** en Azure Portal. Ahora debería mostrar al menos cuatro recursos:

| Nombre | Tipo  | Location |
|------|-------|----------|
| EchoBot20190805125647 | App Service | Oeste de EE. UU. |
| SpeechEchoBotTutorial-AppServicePlan | Plan de App Service | Oeste de EE. UU. |
| SpeechEchoBotTutorial-BotRegistration-8726 | Registro de canales de bot | Global |
| SpeechEchoBotTutorial-Speech | Cognitive Services | Oeste de EE. UU. |

> [!IMPORTANT]
> El recurso del registro de canales de bot mostrará la región global aunque haya seleccionado Oeste de EE. UU. Se espera que esto sea así.

## <a name="optional-test-in-web-chat"></a>Opcional: Probar en el Chat en web

La página Azure Bot Channels Registration tiene la opción **Probar en el Chat en web** en **Administración de bots**. No funcionará de forma predeterminada con su bot, ya que el chat en web debe autenticarse con su bot. Si desea probar el bot que implementado con entrada de texto, siga los pasos que se indican a continuación. Tenga en cuenta que estos pasos son opcionales, no son necesarios para continuar con los pasos siguientes del tutorial. 

1. Busque el recurso **EchoBotTutorial-BotRegistration-####** en [Azure Portal](https://portal.azure.com) y ábralo.
1. En **Administración de bots**, seleccione **Configuración**. Copie el valor de **Microsoft App ID** (Id. de aplicación de Microsoft).
1. Abra la solución EchoBot de Visual Studio. En el Explorador de soluciones, busque **appsettings.json** y haga doble clic en él.
1. Reemplace la cadena vacía que está junto a **MicrosoftAppId** en el archivo JSON por el valor del identificador copiado.
1. De vuelta en Azure Portal, en **Administración de bots**, seleccione **Configuración** y haga clic en **(Administrar)** , al lado de **Microsoft App ID** (Id. de aplicación de Microsoft).
1. Haga clic en **Nuevo secreto de cliente** (Nuevo secreto de cliente). Agregue una descripción (p.ej., "chat en web") y haga clic en **Agregar**. Copie el nuevo secreto.
1. Reemplace la cadena vacía que está junto a **MicrosoftAppPassword** en el archivo JSON por el valor del secreto copiado.
1. Guarde el archivo JSON. Debe tener el siguiente aspecto:
```json
{
  "MicrosoftAppId": "3be0abc2-ca07-475e-b6c3-90c4476c4370",
  "MicrosoftAppPassword": "-zRhJZ~1cnc7ZIlj4Qozs_eKN.8Cq~U38G"
}
```
9. Vuelva a publicar la aplicación (haga clic con el botón derecho en el proyecto **EchoBot** en el Explorador de soluciones de Visual Studio, seleccione **Publicar...** y haga clic en el botón **Publicar**)
10. Ya está listo para probar el bot en un chat en web.

## <a name="register-the-direct-line-speech-channel"></a>Registro del canal Direct Line Speech

Ahora es el momento de registrar el bot con el canal Direct Line Speech. Este canal crea una conexión entre el bot y una aplicación cliente compilada con el SDK de voz.

1. Busque y abra el recurso **SpeechEchoBotTutorial-BotRegistration-####** en [Azure Portal](https://portal.azure.com).
1. En **Administración de bots**, seleccione **Canales**.
   * En **Más canales**, haga clic en **Direct Line Speech**.
   * Revise el texto de la página titulada **Configurar Direct Line Speech** y, a continuación, expanda el menú desplegable de la **cuenta de servicio de Cognitive**.
   * Seleccione el recurso de voz que creó anteriormente (por ejemplo, **SpeechEchoBotTutorial-Speech**) en el menú para asociar el bot a la clave de suscripción de voz.
   * Ignore los restantes campos opcionales.
   * Haga clic en **Save**(Guardar).

1. En el panel de navegación **Bot management** (Administración de bots), haga clic en **Configuración**.
   * Active la casilla de verificación **Enable Streaming Endpoint** (Habilitar punto de conexión de streaming). Esto es necesario para crear un protocolo de comunicación basado en sockets web entre el bot y el canal Direct Line Speech.
   * Haga clic en **Save**(Guardar).

> [!TIP]
> Si quiere más información, consulte [Conexión de un bot a Direct Line Speech](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-directlinespeech?view=azure-bot-service-4.0). En esta página se incluye información adicional así como problemas conocidos.

## <a name="run-the-windows-voice-assistant-client"></a>Ejecución del cliente del asistente de voz de Windows

En este paso, ejecutará el cliente del asistente de voz de Windows. El cliente es una aplicación de Windows Presentation Foundation (WPF) en C# que usa el [SDK de voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk) para administrar la comunicación con el bot mediante el canal Direct Line Speech. Úselo para interactuar con el bot y probarlo antes de escribir una aplicación cliente personalizada. Es código abierto, por lo que puede descargar el archivo ejecutable y ejecutarlo, o bien compilarlo usted mismo.

El cliente del asistente de voz de Windows tiene una interfaz de usuario simple que permite configurar la conexión al bot, ver la conversación de texto, ver actividades de Bot Framework en formato JSON y mostrar tarjetas adaptables. También admite el uso de palabras clave personalizadas. Va a utilizar este cliente para hablar con el bot y recibir una respuesta de voz.

> [!NOTE]
> En este momento, confirme que el micrófono y los altavoces estén habilitados y en perfecto funcionamiento.

1. Vaya al repositorio de GitHub para buscar el [cliente del asistente de voz de Windows](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/blob/master/clients/csharp-wpf/README.md).
1. Siga las instrucciones que se proporcionan aquí para
   * descargar un archivo ejecutable precompilado en un paquete ZIP para ejecutarlo, o bien
   * compilar usted mismo el ejecutable mediante la clonación del repositorio y la compilación del proyecto.

1. Inicie la aplicación cliente `VoiceAssistantClient.exe` y configúrela para conectarse a su bot. Para ello, siga las instrucciones del repositorio de GitHub.
1. Haga clic en **Volver a conectar** y asegúrese de que ve el mensaje **Se ha iniciado una conversación: escriba o presione el botón de micrófono**.
1. Vamos a probarlo. Haga clic en el botón de micrófono y diga algunas palabras en inglés. El texto reconocido aparecerá mientras habla. Cuando haya terminado de hablar, el bot responderá en su propia voz, diciendo "echo" (eco) seguido de las palabras reconocidas.
1. También puede utilizar el texto para comunicarse con el bot. Simplemente escriba el texto en la barra inferior. 

### <a name="troubleshooting-errors-in-windows-voice-assistant-client"></a>Solución de errores en el cliente del asistente de voz de Windows

Si recibe un mensaje de error en la ventana principal de la aplicación, use esta tabla para identificarlo y solucionarlo:

| Error | ¿Qué debería hacer? |
|-------|----------------------|
|Error (AuthenticationFailure): WebSocket Upgrade failed with an authentication error (401). Check for correct subscription key (or authorization token) and region name [Error AuthenticationFailure: No se pudo actualizar el socket web con un error de autenticación (401)]| En la página Configuración de la aplicación, asegúrese de que ha especificado correctamente la clave de suscripción de voz y su región.<br>Asegúrese de que la clave de voz y la región de la clave se escribieron correctamente. |
|Error (ConnectionFailure): Connection was closed by the remote host. Código de error: 1011. Detalles del error: We could not connect to the bot before sending a message (Error ConnectionFailure: El host remoto cerró la conexión. Código de error: 1011. Detalles del error: No se pudo conectar al bot antes de enviar un mensaje) | Asegúrese de [activar la casilla "Enable Streaming Endpoint"](#register-the-direct-line-speech-channel) (Habilitar el punto de conexión de streaming) o de [alternar los **sockets web**](#enable-web-sockets) en Activado.<br>Asegúrese de que la instancia de Azure App Service se ejecuta. Si es así, pruebe a reiniciar la instancia de App Service.|
|Error (ConnectionFailure): Connection was closed by the remote host. Código de error: 1002. Detalles del error: El servidor devolvió el código de estado ''503" cuando se esperaba el código "101". | Asegúrese de [activar la casilla "Enable Streaming Endpoint"](#register-the-direct-line-speech-channel) (Habilitar el punto de conexión de streaming) o de [alternar los **sockets web**](#enable-web-sockets) en Activado.<br>Asegúrese de que la instancia de Azure App Service se ejecuta. Si es así, pruebe a reiniciar la instancia de App Service.|
|Error (ConnectionFailure): Connection was closed by the remote host. Código de error: 1011. Detalles del error: Response status code does not indicate success: 500 (InternalServerError) [Error ConnectionFailure: El host remoto cerró la conexión. Código de error: 1011. Detalles del error: El código de estado de respuesta no indica una operación correcta]| El bot especificó una voz neuronal en el campo [Speak](https://github.com/microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md#speak) (Hablar) de salida, pero la región de Azure asociada a la clave de suscripción de voz no es compatible con las voces neuronales. Consulte [Voces estándares y neuronales](https://docs.microsoft.com/azure/cognitive-services/speech-service/regions#standard-and-neural-voices).|

Si el problema no se soluciona en la tabla, consulte [Asistentes de voz: Frequently asked questions](faq-voice-assistants.md) (inicio de sesión único de conexión directa de Azure Active Directory: preguntas más frecuentes). Si sigue sin poder resolver el problema después de seguir todos los pasos de este tutorial, envíe una nueva incidencia en la [página de GitHub del asistente de voz](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/issues).

#### <a name="a-note-on-connection-time-out"></a>Nota sobre el tiempo de espera de la conexión

Si está conectado a un bot y no se ha producido ninguna actividad en los últimos 5 minutos, el servicio cerrará automáticamente la conexión del websocket con el cliente y con el bot. es así por diseño. Aparecerá un mensaje en la barra inferior: *"Active connection timed out but ready to reconnect on demand"* (Se agotó el tiempo de espera de la conexión activa, que está lista para volver a conectar bajo demanda). No es necesario presionar el botón "Volver a conectar"; simplemente presione el botón de micrófono y empiece a hablar, escriba un mensaje de texto o indique la palabra clave (si está habilitada). La conexión se restablecerá automáticamente.  
### <a name="view-bot-activities"></a>Ver actividades de bot

Cada bot envía y recibe mensajes de **actividad**. En la ventana **Activity Log** (Registro de actividades) del cliente del asistente de voz de Windows, verá los registros con marca de tiempo con cada actividad en la que el cliente ha recibido del bot. También puede ver las actividades que el cliente ha enviado al bot mediante el método [`DialogServiceConnector.SendActivityAsync`](https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.dialog.dialogserviceconnector.sendactivityasync). Al seleccionar un elemento de registro, se mostrarán los detalles de la actividad asociada como JSON.

Este es un ejemplo de JSON de una actividad que el cliente ha recibido:

```json
{
    "attachments":[],
    "channelData":{
        "conversationalAiData":{
             "requestInfo":{
                 "interactionId":"8d5cb416-73c3-476b-95fd-9358cbfaebfa",
                 "version":"0.2"
             }
         }
    },
    "channelId":"directlinespeech",
    "conversation":{
        "id":"129ebffe-772b-47f0-9812-7c5bfd4aca79",
        "isGroup":false
    },
    "entities":[],
    "from":{
        "id":"SpeechEchoBotTutorial-BotRegistration-8726"
    },
    "id":"89841b4d-46ce-42de-9960-4fe4070c70cc",
    "inputHint":"acceptingInput",
    "recipient":{
        "id":"129ebffe-772b-47f0-9812-7c5bfd4aca79|0000"
    },
    "replyToId":"67c823b4-4c7a-4828-9d6e-0b84fd052869",
    "serviceUrl":"urn:botframework:websocket:directlinespeech",
    "speak":"<speak version='1.0' xmlns='https://www.w3.org/2001/10/synthesis' xml:lang='en-US'><voice name='Microsoft Server Speech Text to Speech Voice (en-US, AriaRUS)'>Echo: Hello and welcome.</voice></speak>",
    "text":"Echo: Hello and welcome.",
    "timestamp":"2019-07-19T20:03:51.1939097Z",
    "type":"message"
}
```

Para obtener más información sobre lo que se devuelve en la salida JSON, consulte los [campos de la actividad](https://github.com/microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md). Para este tutorial, puede centrarse en los campos [Text](https://github.com/microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md#text) (Texto) y [Speak](https://github.com/microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md#speak) (Hablar).

### <a name="view-client-source-code-for-calls-to-the-speech-sdk"></a>Visualización del código fuente de cliente para las llamadas al SDK de voz

El cliente del asistente de voz de Windows usa el paquete NuGet [Microsoft.CognitiveServices Account.Speech](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech/), que contiene Speech SDK. Un buen lugar para empezar a revisar el código de ejemplo es el método InitSpeechConnector() en el archivo [`VoiceAssistantClient\MainWindow.xaml.cs`](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/blob/master/clients/csharp-wpf/VoiceAssistantClient/MainWindow.xaml.cs), que crea estos dos objetos del SDK de voz:
- [`DialogServiceConfig`](https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.dialog.dialogserviceconfig): para opciones de configuración (por ejemplo, clave de suscripción de voz, región de la clave)
- [`DialogServiceConnector`](https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.dialog.dialogserviceconnector.-ctor): para administrar la conexión del canal y los eventos de la suscripción de cliente para controlar las respuestas de voz y bot reconocidas.

## <a name="add-custom-keyword-activation"></a>Agregar la activación de la palabra clave personalizada

El SDK de Voz admite la activación de palabras clave personalizadas. De forma similar a "Hola Cortana" del ayudante de Microsoft, puede escribir una aplicación que escuche continuamente la palabra clave que prefiera. Tenga en cuenta que una palabra clave puede ser una sola palabra o una frase con varias palabras.

> [!NOTE]
> El término *palabra clave* se suele usar indistintamente con el término *palabra de reactivación*, y verá que ambos se usan en la documentación de Microsoft.

La detección de la palabra clave se realiza en la aplicación cliente. Si se usa una palabra clave, el audio solo se transmite al canal de Direct Line Speech si se detecta dicha palabra clave. El canal Direct Line Speech incluye un componente denominado *comprobación de palabras clave (KWV)* , que realiza un procesamiento más complejo en la nube para comprobar que la palabra clave que ha elegido está al principio de la secuencia de audio. Si la comprobación de palabras clave se realiza correctamente, el canal se comunicará con el bot.

Siga estos pasos para crear un modelo de palabra clave, configurar el cliente del asistente de voz de Windows para usar este modelo y, por último, probarlo con el bot.

1. Siga estas instrucciones para [crear una palabra clave personalizada mediante el servicio Voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-devices-sdk-create-kws).
2. Descomprima el archivo de modelo que descargó en el paso anterior. Debe tener el nombre de la palabra clave. Está buscando un archivo denominado `kws.table`.
3. En el cliente del asistente de voz de Windows, busque el menú **Settings** (Configuración) (busque el icono del engranaje en la parte superior derecha). Busque la **ruta de acceso del archivo de modelo** y escriba el nombre de la ruta de acceso completa del archivo `kws.table` del paso 2.
4. Asegúrese de activar la casilla denominada **Enabled** (Habilitada). Debería ver este mensaje junto a la casilla: "Will listen for the keyword upon next connection" (Escuchará la palabra clave después de la siguiente conexión). Si ha proporcionado un archivo incorrecto o una ruta de acceso no válida, debería ver un mensaje de error.
5. Escriba la **clave de suscripción** de voz, la **región de clave de suscripción** y, a continuación, haga clic en **Aceptar** para cerrar el menú de **configuración**.
6. Haga clic en **Volver a conectar**. Verá un mensaje que indica lo siguiente: "New conversation started - type, press the microphone button, or say the keyword" (Nueva conversación iniciada: escriba, presione el botón de micrófono o la palabra clave). La aplicación ahora escucha continuamente.
7. Diga cualquier frase que empiece con la palabra clave. Por ejemplo: " **[su palabra clave]** , what time is it?". No es necesario hacer una pausa después de pronunciar la palabra clave. Cuando termine, sucederán dos cosas:
   * Verá una transcripción de lo que ha hablado.
   * Poco después, oirá la respuesta del bot.
8. Continúe experimentando con los tres tipos de entradas que admite el bot:
   * Escribir texto en la barra inferior
   * Presionar el icono del micrófono y hablar
   * Decir una frase que empiece con la palabra clave

### <a name="view-the-source-code-that-enables-keyword"></a>Ver el código fuente que habilita la palabra clave

En el código fuente del cliente del asistente de voz de Windows, eche un vistazo a estos archivos para revisar el código que se usa para habilitar la detección de palabras clave:

1. [`VoiceAssistantClient\Models.cs`](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/blob/master/clients/csharp-wpf/VoiceAssistantClient/Models.cs) incluye una llamada al método [`KeywordRecognitionModel.fromFile()`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/keywordrecognitionmodel?view=azure-node-latest#fromfile-string-) del SDK de voz, que se usa para crear una instancia del modelo a partir de un archivo local en el disco.
1. [`VoiceAssistantClient\MainWindow.xaml.cs`](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/blob/master/clients/csharp-wpf/VoiceAssistantClient/MainWindow.xaml.cs) incluye una llamada al método [`DialogServiceConnector.StartKeywordRecognitionAsync()`](https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.dialog.dialogserviceconnector.startkeywordrecognitionasync) del SDK de Voz, que activa la detección continua de la palabra clave.

## <a name="optional-change-the-language-and-bot-voice"></a>(Opcional) Cambio del idioma y la voz del bot

El bot que creó escuchará y responderá en inglés, y la voz predeterminada para la conversión de texto a voz será en inglés (EE.UU.). Sin embargo, no está limitado a usar el idioma inglés ni una voz predeterminada. En esta sección, aprenderá a cambiar el idioma en el que el bot escuchará y responderá. También aprenderá a seleccionar una voz diferente para ese idioma.

### <a name="change-the-language"></a>Cambio del idioma

Puede elegir uno de los idiomas que se mencionan en la tabla [voz a texto](language-support.md#speech-to-text). En el ejemplo siguiente, se cambiará el idioma a alemán.

1. Abra la aplicación cliente del asistente de voz de Windows, haga clic en el botón de configuración (icono del engranaje de la parte superior derecha) y escriba `de-de` en el campo Language (Idioma) (este es el valor de configuración regional que se menciona en la tabla [de voz a texto](language-support.md#speech-to-text)). Esto establece el idioma hablado para que se reconozca y se sustituya el valor predeterminado `en-us`. Esta acción también indica al canal de Direct Line Speech que use una voz alemana predeterminada para la respuesta del bot.
2. Cierre la página Configuración y haga clic en el botón Volver a conectar para establecer una nueva conexión con el bot de eco.
3. Haga clic en el botón de micrófono y diga una frase en alemán. Verá el texto reconocido y el bot de eco responderá con la voz alemana predeterminada.

### <a name="change-the-default-bot-voice"></a>Cambio de la voz de bot predeterminada

Puede seleccionar la voz de conversión de texto a voz y controlar la pronunciación si el bot especifica la respuesta en forma de [lenguaje de marcado de síntesis de voz](speech-synthesis-markup.md) (SSML) en lugar de texto simple. El bot de eco no usa SSML, pero se puede modificar fácilmente el código para que lo use. En el ejemplo siguiente, se agrega SSML a la respuesta del bot de eco. De este modo, se utilizará la voz alemana de Stefan Apollo (una voz de hombre) en lugar de la voz de mujer predeterminada. Consulte la lista de [voces estándar](language-support.md#standard-voices) y [voces neuronales](language-support.md#neural-voices) compatibles con su idioma.

1. Comencemos por abrir `samples\csharp_dotnetcore\02.echo-bot\echo-bot.cs`.
2. Busque estas dos líneas:
    ```csharp
    var replyText = $"Echo: {turnContext.Activity.Text}";
    await turnContext.SendActivityAsync(MessageFactory.Text(replyText, replyText), cancellationToken);
    ```
3. Reemplácelas por:
    ```csharp
    var replyText = $"Echo: {turnContext.Activity.Text}";
    var replySpeak = @"<speak version='1.0' xmlns='https://www.w3.org/2001/10/synthesis' xml:lang='de-DE'>
                    <voice name='Microsoft Server Speech Text to Speech Voice (de-DE, Stefan, Apollo)'>" +
                    $"{replyText}" + "</voice></speak>";
    await turnContext.SendActivityAsync(MessageFactory.Text(replyText, replySpeak), cancellationToken);
    ```
4. Compile la solución en Visual Studio y corrija los errores de compilación.

El segundo argumento del método "MessageFactory.Text" establece el [campo Texto hablado de la actividad](https://github.com/Microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md#speak) en la respuesta del bot. Con el cambio anterior, se realizó el cambio de texto simple a SSML para especificar una voz alemana no predeterminada.

### <a name="redeploy-your-bot"></a>Reimplementación del bot

Ahora que ha realizado el cambio necesario en el bot, el paso siguiente consiste en volver a publicarlo en la instancia de Azure App Service y probarlo:

1. En la ventana Explorador de soluciones, haga clic con el botón derecho en el proyecto **EchoBot** y seleccione **Publicar**.
2. La configuración de implementación anterior ya se ha cargado como valor predeterminado. Simplemente haga clic en **Publicar** junto a **EchoBot20190805125647 - Web Deploy**.
3. El mensaje **Publicación correcta** aparecerá en la ventana de salida de Visual Studio y se iniciará una página web con el mensaje "Your bot is ready!" (El bot está listo).
4. Abra la aplicación cliente del asistente de voz de Windows, haga clic en el botón de configuración (icono del engranaje de la parte superior derecha) y asegúrese de que el valor `de-de` sigue en el campo Language (Idioma).
5. Siga las instrucciones de la sección [Ejecución del cliente del asistente de voz de Windows](#run-the-windows-voice-assistant-client) para volver a conectar con el bot recién implementado, hablar en el nuevo idioma y oír la respuesta del bot en ese idioma con la nueva voz.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando el bot eco que ha implementado en este tutorial, puede quitarlo y todos los recursos de Azure asociados simplemente eliminando el grupo de recursos de Azure **SpeechEchoBotTutorial-ResourceGroup**.

1. En [Azure Portal](https://portal.azure.com), seleccione **Grupos de recursos** en el panel de navegación **Servicios de Azure**.
2. Busque el grupo de recursos denominado: **SpeechEchoBotTutorial-ResourceGroup**. Haga clic en los tres puntos (...).
3. Seleccione **Eliminar grupo de recursos**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de la propia aplicación cliente con el SDK de voz](quickstart-voice-assistant-csharp-uwp.md)

## <a name="see-also"></a>Consulte también

* Implementación en una [región de Azure próxima](https://azure.microsoft.com/global-infrastructure/locations/) para ver la mejora del tiempo de respuesta de bot
* Implementación en una [región de Azure que admite voces TTS neuronales de alta calidad](https://docs.microsoft.com/azure/cognitive-services/speech-service/regions#standard-and-neural-voices)
* Precios asociados al canal Direct Line Speech:
  * [Precios de Bot Service](https://azure.microsoft.com/pricing/details/bot-service/)
  * [Servicio Voz](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/)
* Creación e implementación del propio bot habilitado para voz:
  * Creación de un [bot de Bot Framework](https://dev.botframework.com/). Registro con el [canal Direct Line Speech](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-directlinespeech?view=azure-bot-service-4.0) y [personalización del bot para voz](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0)
  * Exploración de las [soluciones de Bot Framework](https://microsoft.github.io/botframework-solutions/index) existentes: Compilación de un [asistente virtual](https://microsoft.github.io/botframework-solutions/overview/virtual-assistant-solution/) y [su ampliación a Direct Line Speech](https://microsoft.github.io/botframework-solutions/clients-and-channels/tutorials/enable-speech/1-intro/)
