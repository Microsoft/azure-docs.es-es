---
author: trrwilson
ms.service: cognitive-services
ms.topic: include
ms.date: 02/10/2020
ms.author: travisw
ms.openlocfilehash: 84c9f61783d2fea425b83c214deede295c4ed581
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87374694"
---
1. Inicie Android Studio y seleccione **Start a new Android Studio project** (Iniciar un nuevo proyecto de Android Studio) en la ventana **Welcome** (Bienvenida).

    ![Captura de pantalla de la ventana Welcome (Bienvenida) de Android Studio](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-01-start-new-android-studio-project.png)

1. Aparece el asistente para la **elección del proyecto**. Seleccione **Phone and Tablet** (Teléfono y tableta) y **Empty Activity** (Actividad vacía) en el cuadro de selección de actividad. Seleccione **Next** (Siguiente).

   ![Captura de pantalla del asistente para la elección del proyecto](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-02-target-android-devices.png)

1. En la pantalla **Configure your project** (Configure su proyecto), escriba *Inicio rápido* como **nombre** y escriba *samples.speech.cognitiveservices.microsoft.com* como **nombre del paquete**. Después, seleccione un directorio de proyecto. En **Minimum API level** (Nivel de API mínimo), seleccione **API 23: Android Marshmallow 6.0 o posterior)** . Deje todas las demás casillas desactivadas y seleccione **Finalizar**.

   ![Captura de pantalla del asistente para configurar el proyecto](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-03-create-android-project.png)

Android Studio tardará unos minutos en preparar el nuevo proyecto de Android. A continuación, configure el proyecto para obtener información del SDK de Voz de Azure Cognitive Services y para usar Java 8.

[!INCLUDE [License notice](cognitive-services-speech-service-license-notice.md)]

La versión actual del SDK de voz de Cognitive Services es la 1.13.0.

El SDK de Voz para Android está empaquetado como una [biblioteca de Android (AAR)](https://developer.android.com/studio/projects/android-library), que incluye las bibliotecas necesarias, así como los permisos necesarios de Android.
Se hospeda en un repositorio de Maven en https:\//csspeechstorage.blob.core.windows.net/maven/.

Configure el proyecto para usar el SDK de Voz. Abra la ventana de la **estructura del proyecto** seleccionando **File** (Archivo) > **Project Structure** (Estructura del proyecto) desde la barra de menús de Android Studio. En la ventana **Project Structure** (Estructura del proyecto), realice los siguientes cambios:

1. En la lista, en el lado izquierdo de la ventana, seleccione **Project** (Proyecto). Edite la opción **Default Library Repository** (Repositorio de bibliotecas predeterminado). Para ello, debe anexar una coma y poner entre comillas simples la dirección URL del repositorio de Maven: 'https:\//csspeechstorage.blob.core.windows.net/maven/'.

   ![Captura de pantalla de la ventana Project Structure (Estructura de proyecto)](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-06-add-maven-repository.png)

1. En la misma pantalla, en el lado izquierdo, seleccione **App** (Aplicación). A continuación, seleccione la pestaña **Dependencies** (Dependencias) en la parte superior de la ventana. Seleccione el signo más verde ( **+** ) y seleccione **Library dependency** (Dependencia de biblioteca) en el menú desplegable.

   ![Captura de pantalla de la dependencia de biblioteca](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-07-add-module-dependency.png)

1. En la ventana que aparece, escriba el nombre y la versión del SDK de Voz para Android, *com.microsoft.cognitiveservices.speech:client-sdk:1.13.0*. Después, seleccione **Aceptar**.
   El SDK de Voz se agregará a la lista de dependencias, como se muestra a continuación:

   ![Captura de pantalla del SDK de Voz en la lista de dependencias](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-08-dependency-added-1.0.0.png)

1. Haga clic en la pestaña **Properties** (Propiedades). En **Source Compability** (Compatibilidad de origen) y **Target Compatibility** (Compatibilidad de destino), seleccione **1.9**.

   ![Captura de pantalla de Source Compability (Compatibilidad de origen) y Target Compatibility (Compatibilidad de destino)](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-09-dependency-added.png)

1. Seleccione **OK** (Aceptar) para cerrar la ventana de la **estructura del proyecto** y aplicar los cambios al proyecto.
