---
title: 'Inicio rápido: Reconocimiento de la voz a través de un micrófono en C# (Unity): servicio de voz'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.custom: devx-track-csharp
ms.topic: include
ms.date: 04/02/2020
ms.author: erhopf
ms.openlocfilehash: 8307cfee90fdf65d119bbc1e55eabb0533a4a6ad
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88926468"
---
> [!NOTE]
> El SDK de Voz para Unity es compatible con Windows Desktop (x86 y x64) o con la Plataforma universal de Windows (x86, x64, ARM o ARM64), Android (x86, ARM32/64) e iOS (simulador de x64, ARM32 y ARM64)

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar:

> [!div class="checklist"]
> * [Ha creado un recurso de Voz de Azure](../../../../get-started.md)
> * [Configuración del entorno de desarrollo y creación de un proyecto vacío](../../../../quickstarts/setup-platform.md?tabs=unity&pivots=programming-language-csharp)
> * Asegúrese de que tiene acceso a un micrófono para capturar el audio.

Si ya lo ha hecho, perfecto. Vamos a continuar.

## <a name="create-a-unity-project"></a>Creación de un proyecto de Unity

1. Abra Unity. Si es la primera vez que usa Unity, aparece la ventana **Unity Hub** (Centro de Unity) *<version number>* . (También puede abrir Unity Hub directamente para obtener acceso a esta ventana).

   [![Ventana de Unity Hub](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-hub.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-hub.png#lightbox)
1. Seleccione **Nuevo**. Aparece la ventana **Create a new project with Unity** *<version number>* (Crear un proyecto con Unity).

   [![Crear un proyecto en Unity Hub](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-create-a-new-project.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-create-a-new-project.png#lightbox)
1. En **Project Name** (Nombre del proyecto), escriba **csharp-unity**.
1. En **Templates** (Plantillas), si **3D** no está seleccionado todavía, selecciónelo.
1. En **Location** (Ubicación), seleccione o cree una carpeta en la que guardar el proyecto.
1. Seleccione **Crear**.

Después de un tiempo, aparecerá la ventana del editor de Unity.



## <a name="add-ui"></a>Incorporación de la interfaz de usuario

Ahora vamos a agregar una pequeña interfaz de usuario a nuestra escena. Esta interfaz de usuario consiste en un botón para desencadenar el reconocimiento de voz y un campo de texto para que aparezca el resultado. En la ventana [**Hierarchy** (Jerarquía) ](https://docs.unity3d.com/Manual/Hierarchy.html), aparece una escena de muestra que Unity ha creado con el nuevo proyecto.

1. En la parte superior de la ventana **Hierarchy** (Jerarquía), seleccione **Create** (Crear)  > **UI** (IU) > **Button** (Botón).

   De esta forma se crean tres objetos de juego que se pueden ver en la ventana **Hierarchy** (Jerarquía): un objeto **Button**, un objeto **Canvas** que contiene el botón y un objeto **EventSystem**.

   [![Entorno del editor de Unity](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-editor-window.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-editor-window.png#lightbox)

1. [Vaya a la vista **Scene** (Escena) ](https://docs.unity3d.com/Manual/SceneViewNavigation.html) para tener una buena vista del lienzo y del botón en la vista [**Scene**](https://docs.unity3d.com/Manual/UsingTheSceneView.html).

1. En la ventana [**Inspector**](https://docs.unity3d.com/Manual/UsingTheInspector.html) (de forma predeterminada, a la derecha), establezca las propiedades **Pos X** y **Pos Y** en **0**, para que el botón se sitúe en el centro del lienzo.

1. En la ventana **Hierarchy** (Jerarquía), seleccione **Create** (Crear)  > **UI** (IU)  > **Text** (Texto) para crear un objeto **Text**.

1. En la ventana **Inspector**, establezca las propiedades **Pos X** y **Pos Y** en **0** y **120**, y establezca las propiedades **Width** (Ancho) y **Height** (Alto) en **240** y **120**. Estos valores garantizan que el campo de texto y el botón no se superponen.

Cuando haya terminado, la vista **Scene** (Escena) debe ser similar a esta captura de pantalla:

[![Vista Scene en el editor de Unity](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-02-ui-inline.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-02-ui-inline.png#lightbox)

## <a name="add-the-sample-code"></a>Incorporación del código de ejemplo

Para agregar el código de script de muestra para el proyecto de Unity, siga estos pasos:

1. En la [ventana Proyecto](https://docs.unity3d.com/Manual/ProjectView.html), seleccione **Crear** > **Script C#** para agregar un nuevo script C#.

   [![Ventana Project en el editor de Unity](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-project-window.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-project-window.png#lightbox)
1. Llame `HelloWorld` al script.

1. Haga doble clic en `HelloWorld` para editar el script que acaba de crear.

   > [!NOTE]
   > Para configurar el editor de código de modo que Unity lo use para la edición, seleccione **Edit** (Editor)  > **Preferences** (Preferencias) y, después, vaya a las preferencias de **External Tools** (Herramientas externas). Para obtener más información, consulte el [Manual de usuario de Unity](https://docs.unity3d.com/Manual/Preferences.html).

1. Reemplace el script por el código siguiente:

   [!code-csharp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/unity/from-microphone/Assets/Scripts/HelloWorld.cs#code)]

1. Busque la cadena `YourSubscriptionKey` y reemplácela por su clave de suscripción del servicio de voz.

1. Busque y reemplace la cadena `YourServiceRegion` por el **identificador de región** de la [región](https://aka.ms/speech/sdkregion) asociada a su suscripción.

1. Guarde los cambios en el script.

Ahora, vuelva al editor de Unity y agregue el script como componente a uno de los objetos del juego:

1. En la ventana **Hierarchy** (Jerarquía), seleccione el objeto **Canvas**.

1. En la ventana **Inspector**, seleccione el botón **Add Component** (Agregar componente).

   [![Ventana Inspector en el editor de Unity](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-inspector-window.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-inspector-window.png#lightbox)

1. En la lista desplegable, busque el script `HelloWorld` que hemos creado anteriormente y agréguelo. Aparece una sección **Hellow World (Script)** [Hola mundo (Script)] en la ventana **Inspector** en la que se enumeran dos propiedades no inicializadas: **Output Text** (Texto de salida) y **Start Reco Button** (Botón Iniciar reconocimiento). Estas propiedades de componente de Unity coinciden con las propiedades públicas de la clase `HelloWorld`.

1. Seleccione el selector de objetos de la propiedad **Start Reco Button** (Botón Iniciar reconocimiento) (el icono de círculo pequeño situado a la derecha de la propiedad) y elija el objeto **Button** que creó anteriormente.

1. Seleccione el selector de objetos de la propiedad **Output Text** (Texto de salida) y elija el objeto **Text** (Texto) que creó anteriormente.

   > [!NOTE]
   > El botón también tiene un objeto Text anidado. Asegúrese de que no lo selecciona accidentalmente para la salida de texto (o cambie el nombre de uno de los objetos de texto mediante el campo **Name** (Nombre) de la ventana **Inspector** para evitar la confusión).

## <a name="run-the-application-in-the-unity-editor"></a>Ejecución de la aplicación en el editor de Unity

Ahora está preparado para ejecutar la aplicación en el editor de Unity.

1. En la barra de herramientas del editor de Unity (debajo de la barra de menús), seleccione el botón **Play** (Reproducir) (el triángulo que apunta a la derecha).

1. Vaya a la vista [**Game** (Juego) ](https://docs.unity3d.com/Manual/GameView.html) y espere a que el objeto **Text** (Texto) muestre **Click button to recognize speech** (Haga clic en el botón para reconocer la voz). [Muestra **New Text** (Nuevo texto) cuando la aplicación no se ha iniciado o no está preparada para responder].

1. Seleccione el botón y pronuncie una expresión o frase en inglés en el micrófono del equipo. Lo que diga se transmitirá a servicio de voz y se transcribirá en texto, que aparecerá en la ventana **Game** (Juego).

   [![Vista Game en el editor de Unity](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-03-output-inline.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-03-output-inline.png#lightbox)

1. Compruebe en la ventana [**Console** (Consola)](https://docs.unity3d.com/Manual/Console.html) los mensajes de depuración. Si la ventana **Console** no se muestra, vaya a la barra de menús y seleccione **Window** (Ventana)  > **General** > **Console** para mostrarla.

1. Cuando haya terminado el reconocimiento de voz, seleccione el botón **Play** (Reproducir) en la barra de herramientas del editor de Unity para detener la aplicación.

## <a name="additional-options-to-run-this-application"></a>Opciones adicionales para ejecutar esta aplicación

También se puede implementar esta aplicación como una aplicación Android, una aplicación independiente de Windows o una aplicación de UWP.
Para más información, consulte el [repositorio de muestras](https://aka.ms/csspeech/samples). La carpeta `quickstart/csharp-unity` describe la configuración de estos destinos adicionales.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [Speech recognition basics](../../speech-to-text-next-steps.md)]

