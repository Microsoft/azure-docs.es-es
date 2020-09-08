---
title: 'Inicio rápido: Reconocimiento de la voz a través de un micrófono en C# (Xamarin): servicio de voz'
titleSuffix: Azure Cognitive Services
description: En este artículo se crea una aplicación C# Xamarin multiplataforma para la Plataforma universal de Windows (UWP), Android e iOS mediante el SDK de Voz de Cognitive Services. Transcriba voz a texto en tiempo real desde el micrófono de su dispositivo o simulador. La aplicación se crea con el paquete NuGet del SDK de voz y Microsoft Visual Studio 2019.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 04/02/2020
ms.author: erhopf
ms.custom: devx-track-csharp
ms.openlocfilehash: 48a80d7367943c2c86fd3822a5c236b0cdf11846
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88926462"
---
## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar:

> [!div class="checklist"]
> * [Ha creado un recurso de Voz de Azure](../../../../get-started.md)
> * [Configuración del entorno de desarrollo y creación de un proyecto vacío](../../../../quickstarts/setup-platform.md?tabs=xamarin&pivots=programming-language-csharp)
> * Asegúrese de que tiene acceso a un micrófono para capturar el audio.

Si ya lo ha hecho, perfecto. Vamos a continuar.

## <a name="add-sample-code-for-the-common-helloworld-project"></a>Adición de código de ejemplo al proyecto común "helloworld"

El proyecto "helloworld" común contiene implementaciones independientes de la plataforma para una aplicación multiplataforma. Ahora, agregue el código XAML que define la interfaz de usuario de la aplicación y agregue el código C# que está detrás de la implementación.

1. En el **Explorador de soluciones**, en el proyecto "helloworld" común, abra `MainPage.xaml`.

1. En la vista XAML del diseñador, inserte el fragmento de código XAML siguiente en la etiqueta **Cuadrícula** entre `<StackLayout>` y `</StackLayout>`:

   [!code-xml[UI elements](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld/MainPage.xaml)]

1. En **Explorador de soluciones**, abra el archivo de código fuente subyacente `MainPage.xaml.cs`. Se agrupa en `MainPage.xaml`.

1. Reemplace todo el código que contiene por el fragmento siguiente:

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld/MainPage.xaml.cs)]

1. En el controlador `OnRecognitionButtonClicked` del archivo de código fuente, busque la cadena `YourSubscriptionKey` y reemplácela por su clave de suscripción.


1. En el controlador `OnRecognitionButtonClicked`, busque la cadena `YourServiceregion` y reemplácela por el **identificador de región** de la [región](https://aka.ms/speech/sdkregion) asociada a su suscripción. 

1. A continuación, debe crear un [servicio Xamarin](https://docs.microsoft.com/xamarin/android/app-fundamentals/services/creating-a-service/), que se usa para consultar los permisos de micrófono desde proyectos de distintas plataformas como UWP, Android e iOS. Para ello, agregue una nueva carpeta denominada *Services* en el proyecto "helloworld" y cree un nuevo archivo de origen C# en ella. Puede hacer clic con el botón derecho en la carpeta *Services* y seleccionar **Agregar** > **Nuevo elemento** > **Archivo de código**. Cambie el nombre del archivo a `IMicrophoneService.cs` y coloque todo el código del siguiente fragmento en ese archivo:

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld/Services/IMicrophoneService.cs)]

#### <a name="android"></a>[Android](#tab/x-android)
## <a name="add-sample-code-for-the-helloworldandroid-project"></a>Adición de código de ejemplo al proyecto `helloworld.Android`

Ahora, agregue el código C# que define la parte específica de Android de la aplicación.

1. En el **Explorador de soluciones**, en el proyecto helloworld.Android, abra `MainActivity.cs`.

1. Reemplace todo el código que contiene por el fragmento siguiente:

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld.Android/MainActivity.cs)]

1. Después, agregue la implementación específica de Android para `MicrophoneService` mediante la creación de una carpeta *Services* en el proyecto helloworld.Android. Después, cree un nuevo archivo de código fuente C# en ella. Cambie el nombre del archivo a `MicrophoneService.cs`. Copie y pegue el siguiente fragmento de código en ese archivo:

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld.Android/Services/MicrophoneService.cs)]

1. Después, abra `AndroidManifest.xml` en la carpeta *Properties*. Agregue el siguiente valor de permisos de usuario para el micrófono entre `<manifest>` y `</manifest>`:

   ```xml
   <uses-permission android:name="android.permission.RECORD_AUDIO" />
   ```
   
#### <a name="ios"></a>[iOS](#tab/ios)
## <a name="add-sample-code-for-the-helloworldios-project"></a>Adición de código de ejemplo al proyecto `helloworld.iOS`

Ahora, agregue el código C# que define la parte específica de iOS de la aplicación. Cree también configuraciones específicas del dispositivo de Apple en el proyecto helloworld.iOS.

1. En el **Explorador de soluciones**, en el proyecto helloworld.iOS, abra `AppDelegate.cs`.

1. Reemplace todo el código que contiene por el fragmento siguiente:

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld.iOS/AppDelegate.cs)]

1. Después, agregue la implementación específica de iOS para `MicrophoneService` mediante la creación de una carpeta *Services* en el proyecto helloworld.iO. Después, cree un nuevo archivo de código fuente C# en ella. Cambie el nombre del archivo a `MicrophoneService.cs`. Copie y pegue el siguiente fragmento de código en ese archivo:

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld.iOS/Services/MicrophoneService.cs)]

1. Abra `Info.plist` en el proyecto helloworld.iOS en el editor de texto. Agregue el siguiente par clave-valor en la sección dict:

   <key>NSMicrophoneUsageDescription</key>
   <string>Esta aplicación de ejemplo requiere acceso al micrófono</string>

   > [!NOTE]
   > Si va a compilar para un dispositivo iPhone, asegúrese de que `Bundle Identifier` coincida con el identificador de la aplicación del perfil de aprovisionamiento del dispositivo. En caso contrario, la compilación dará error. Con iPhoneSimulator puede dejarlo tal cual.

1. Si va a compilar en un PC Windows, debe establecer una conexión con el dispositivo Mac para compilar mediante **Tools** (Herramientas) > **iOS** > **Pair to Mac** (Emparejar con Mac). Siga el Asistente para instrucciones de Visual Studio para habilitar la conexión con el dispositivo Mac.

#### <a name="uwp"></a>[UWP](#tab/helloworlduwp)
## <a name="add-sample-code-for-the-helloworlduwp-project"></a>Adición de código de ejemplo al proyecto `helloworld.UWP`

## <a name="add-sample-code-for-the-helloworlduwp-project"></a>Adición de código de ejemplo al proyecto helloworld.UWP

Ahora, agregue el código C# que define la parte específica de UWP de la aplicación.

1. En el **Explorador de soluciones**, en el proyecto helloworld.UWP, abra `MainPage.xaml.cs`.

1. Reemplace todo el código que contiene por el fragmento siguiente:

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld.UWP/MainPage.xaml.cs)]

1. Después, agregue la implementación específica de UWP para `MicrophoneService` mediante la creación de una carpeta *Services* en el proyecto helloworld.UWP. Después, cree un nuevo archivo de código fuente C# en ella. Cambie el nombre del archivo a `MicrophoneService.cs`. Copie y pegue el siguiente fragmento de código en ese archivo:

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld.UWP/Services/MicrophoneService.cs)]

1. A continuación, haga doble clic en el archivo `Package.appxmanifest` en el proyecto helloworld.UWP de Visual Studio. En **Funcionalidades**, asegúrese de que está seleccionado **Micrófono** y guarde el archivo.

1. Luego, haga doble clic en el archivo `Package.appxmanifest` del proyecto `helloworld.UWP` en Visual Studio y compruebe que en **Funcionalidades** > **Micrófono** está marcado y guarde el archivo.
   > Nota: Si ve la siguiente advertencia: Certificate file does not exist: helloworld.UWP_TemporaryKey.pfx, please check [speech to text](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-csharp&tabs=uwp) sample for more information (El archivo de certificado no existe: helloworld.UWP_TemporaryKey.pfx, consulte el ejemplo de conversión de voz en texto para más información).

1. En la barra de menús, seleccione **Archivo** > **Guardar todo** para guardar los cambios.

## <a name="build-and-run-the-uwp-application"></a>Compilación y ejecución de la aplicación UWP

1. Establezca helloworld.UWP como proyecto de inicio. Haga clic con el botón derecho en el proyecto helloworld.UWP y seleccione **Crear** para crear la aplicación.

1. Seleccione **Depurar** > **Iniciar depuración** (o seleccione **F5**) para iniciar la aplicación. Aparece la ventana **HelloWorld**.

   ![Ejemplo de aplicación de reconocimiento de voz de UWP en C#: inicio rápido](../../../../media/sdk/qs-csharp-xamarin-helloworld-uwp-window.png)

1. Seleccione **Habilitar micrófono**. Cuando aparezca la solicitud de permiso de acceso, seleccione **Sí**.

   ![Solicitud de permiso de acceso al micrófono](../../../../media/sdk/qs-csharp-xamarin-uwp-access-prompt.png)

1. Seleccione **Iniciar reconocimiento de voz**  y diga una frase en inglés en el micrófono del dispositivo. Lo que diga se transmitirá al servicio de Voz y se transcribe en texto, que aparece en la misma ventana.

   ![Interfaz de usuario del reconocimiento de voz](../../../../media/sdk/qs-csharp-xamarin-uwp-ui-result.png)
* * *

## <a name="build-and-run-the-android-and-ios-applications"></a>Compilación y ejecución de aplicaciones de iOS y Android

La compilación y ejecución de aplicaciones de iOS y Android en el dispositivo o en el simulador se producen de manera similar a como lo hacen en UWP. Asegúrese de que todos los SDK requeridos en la sección "Requisitos previos" de este artículo están instalados correctamente.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [Speech recognition basics](../../speech-to-text-next-steps.md)]
