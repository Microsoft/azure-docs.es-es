---
author: trevorbye
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 02/20/2020
ms.author: trbye
ms.openlocfilehash: abc9a42c7ced6a71f93c00968eb825f28f720843
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87375011"
---
En este inicio rápido, aprenderá a usar Speech Devices SDK para Android para crear un producto habilitado para voz o para utilizarlo como un dispositivo de [transcripción de conversaciones](../conversation-transcription-service.md).

Para esta guía se requiere una cuenta de [Azure Cognitive Services](../get-started.md) con un recurso del servicio de voz. Si no tiene una cuenta, puede usar la [evaluación gratuita](https://azure.microsoft.com/try/cognitive-services/) para obtener una clave de suscripción.

El código fuente de la aplicación de ejemplo se incluye con el SDK de dispositivos de voz. También está [disponible en GitHub](https://github.com/Azure-Samples/Cognitive-Services-Speech-Devices-SDK).

## <a name="prerequisites"></a>Requisitos previos

Para empezar a usar Speech Devices SDK, deberá:

- Seguir las instrucciones proporcionadas con su [kit de desarrollo](../get-speech-devices-sdk.md) para encender el dispositivo.

- Descargar la versión más reciente de [Speech Devices SDK](https://aka.ms/sdsdk-download) y extraer el archivo .zip en el directorio de trabajo.

  > [!NOTE]
  > En esta guía de inicio rápido se supone que la aplicación se extrae en C:\SDSDK\Android-Sample-Release

- Para obtener una [clave de suscripción de Azure para el servicio de voz](../get-started.md):

- Si tiene previsto usar la transcripción de conversaciones debe usar un [dispositivo de micrófono circular](../get-speech-devices-sdk.md) y esta característica actualmente solo está disponible para los idiomas "en-US" y "zh-CN" en las regiones, "centralus" y "eastasia". Debe tener una clave de voz en una de esas regiones para usar Transcripción de conversaciones.

- Si tiene previsto usar el servicio de voz para identificar las intenciones (o acciones) a partir de las expresiones de los usuarios, necesitará una suscripción al [servicio Language Understanding (LUIS)](https://docs.microsoft.com/azure/cognitive-services/luis/azureibizasubscription). Para más información acerca de LUIS y el reconocimiento de intenciones, consulte [Reconocimiento de las intenciones de voz con LUIS, C#](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-recognize-intents-from-speech-csharp).

  También puede [crear un modelo sencillo de LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/) o usar el modelo LUIS de ejemplo, LUIS-example.json. El ejemplo de modelo LUIS está disponible en el [sitio de descarga del SDK de dispositivos de voz](https://aka.ms/sdsdk-luis). Para cargar el archivo JSON del modelo en el [portal de LUIS](https://www.luis.ai/home), seleccione **Import new app** (Importar aplicación nueva) y elija el archivo JSON.

- Instale [Android Studio](https://developer.android.com/studio/) y [Vysor](https://vysor.io/download/) en su PC.

## <a name="set-up-the-device"></a>Configuración del dispositivo

1. Inicie Vysor en el equipo.

   ![Vysor](../media/speech-devices-sdk/qsg-3.png)

1. El dispositivo debe figurar en la lista **Choose a device** (Seleccionar un dispositivo). Haga clic en el botón **View** (Vista) situado junto al dispositivo.

1. Para conectarse a la red inalámbrica, haga clic en el icono de la carpeta y seleccione **Settings** (Configuración) > **WLAN**.

   ![WLAN de Vysor](../media/speech-devices-sdk/qsg-4.png)

   > [!NOTE]
   > Si su empresa tiene directivas respecto a la conexión de dispositivos al sistema Wi-Fi, deberá obtener la dirección MAC y ponerse en contacto con el departamento de TI para que le indiquen cómo conectarse al Wi-Fi de la empresa.
   >
   > Para buscar la dirección MAC del kit de desarrollo, seleccione el icono de la carpeta de archivos en el escritorio del kit de desarrollo.
   >
   > ![Carpeta de archivos de Vysor](../media/speech-devices-sdk/qsg-10.png)
   >
   > Seleccione **Configuración**. Busque "mac address" (dirección MAC) y luego seleccione **Mac address** (Dirección MAC) > **Advanced WLAN** (WLAN avanzada). Anote la dirección MAC que aparece cerca de la parte inferior del cuadro de diálogo.
   >
   > ![Dirección MAC de Vysor](../media/speech-devices-sdk/qsg-11.png)
   >
   > Es posible que algunas compañías tengan un tiempo límite en cuanto al tiempo que un dispositivo puede estar conectado a su sistema Wi-Fi. Además, puede que tras un determinado número de días deba ampliar el registro del kit de desarrollo en el sistema Wi-Fi.

## <a name="run-the-sample-application"></a>Ejecutar la aplicación de ejemplo

Para validar la configuración del kit de desarrollo, compile e instale la aplicación de ejemplo:

1. Inicie Android Studio.

1. Seleccione **Abrir un proyecto existente de Android Studio**.

   ![Android Studio: abrir un proyecto existente](../media/speech-devices-sdk/qsg-5.png)

1. Vaya a C:\SDSDK\Android-Sample-Release\example. Seleccione **Aceptar** para abrir el proyecto de ejemplo.

1. Configure Gradle para que haga referencia al SDK de Voz. Los siguientes archivos se pueden encontrar en **Gradle Scripts** (Scripts de Gradle) en Android Studio.

    Actualice **build.gradle(Project:example)** , el bloque allprojects debe coincidir con el que se indica a continuación, mediante la adición de líneas Maven.

    ```xml
    allprojects {
        repositories {
            google()
            jcenter()
            mavenCentral()
            maven {
                url 'https://csspeechstorage.blob.core.windows.net/maven/'
            }
        }
    }
    ```

    Actualice **build.gradle(Module:app)** agregando esta línea a la sección de dependencias. 
    
    ```xml
    implementation'com.microsoft.cognitiveservices.speech:client-sdk:1.13.0'
    ```
    
1. Agregue la clave de suscripción de Voz al código fuente. Si quiere probar el reconocimiento de intenciones, agregue también la clave de suscripción y el identificador de aplicación del [servicio Language Understanding](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/).

   Para Voz y LUIS, la información se trasladará a MainActivity.java:

   ```java
    // Subscription
    private static String SpeechSubscriptionKey = "<enter your subscription info here>";
    private static String SpeechRegion = "westus"; // You can change this if your speech region is different.
    private static String LuisSubscriptionKey = "<enter your subscription info here>";
    private static String LuisRegion = "westus2"; // you can change this, if you want to test the intent, and your LUIS region is different.
    private static String LuisAppId = "<enter your LUIS AppId>";
   ```

   Si usa la transcripción de conversaciones, la información de la clave y la región de Voz también se necesitará en conversation.java:

   ```java
    private static final String CTSKey = "<Conversation Transcription Service Key>";
    private static final String CTSRegion="<Conversation Transcription Service Region>";// Region may be "centralus" or "eastasia"
   ```

1. La palabra clave predeterminada es "Equipo". Además, puede probar una de las otras palabras clave proporcionadas, como "Máquina" o "Asistente". Los archivos de recursos de estas palabras clave alternativas pueden encontrarse en Speech Devices SDK en la carpeta de palabras clave. Por ejemplo, C:\SDSDK\Android-Sample-Release\keyword\Computer contiene los archivos que se usan para la palabra clave "Equipo".

   > [!TIP]
   > También puede [crear una palabra clave personalizada](../speech-devices-sdk-create-kws.md).

   Para usar una nueva palabra clave, actualice las dos líneas siguientes en `MainActivity.java` y copie el paquete de palabras clave en la aplicación. Por ejemplo, para usar la palabra clave "Máquina" desde el paquete de palabras clave kws-machine.zip:

   - Copie el paquete de palabras clave en la carpeta "C:\SDSDK\Android-Sample-Release\example\app\src\main\assets\".
   - Actualice `MainActivity.java` con la palabra clave y el nombre del paquete:

     ```java
     private static final String Keyword = "Machine";
     private static final String KeywordModel = "kws-machine.zip" // set your own keyword package name.
     ```

1. Actualice las siguientes líneas que contienen la configuración de geometría de matriz del micrófono:

   ```java
   private static final String DeviceGeometry = "Circular6+1";
   private static final String SelectedGeometry = "Circular6+1";
   ```

   Esta tabla enumera los valores admitidos:

   | Variable | Significado | Valores disponibles |
   | -------- | ------- | ---------------- |
   | `DeviceGeometry` | Configuración del micrófono físico | Para un kit de desarrollo circular: `Circular6+1` |
   |          |         | Para un kit de desarrollo lineal: `Linear4` |
   | `SelectedGeometry` | Configuración de micrófono de software | Para un kit de desarrollo circular que usa todos los micrófonos: `Circular6+1` |
   |          |         | Para un kit de desarrollo circular que usa cuatro micrófonos: `Circular3+1` |
   |          |         | Para un kit de desarrollo lineal que usa todos los micrófonos: `Linear4` |
   |          |         | Para un kit de desarrollo lineal que usa dos micrófonos: `Linear2` |

1. Para compilar la aplicación, en el menú **Run** (Ejecutar), seleccione **Run 'app'** (Ejecutar "aplicación"). Aparecerá el cuadro de diálogo **Select Deployment Target** (Seleccionar el destino de la implementación).

1. Elija el dispositivo y haga clic en **OK** (Aceptar) para implementar la aplicación en el dispositivo.

   ![Seleccione el cuadro de diálogo para elegir el destino de la implementación.](../media/speech-devices-sdk/qsg-7.png)

1. Se inicia la aplicación de ejemplo del SDK de dispositivos de voz y muestra las siguientes opciones:

   ![Opciones y aplicación de ejemplo del SDK de dispositivos de voz de ejemplo](../media/speech-devices-sdk/qsg-8.png)

1. Pruebe la nueva demostración de transcripción de conversaciones. Empiece a transcribir con "Iniciar sesión". De forma predeterminada, todos los usuarios son invitados. Sin embargo, si dispone de las firmas de voz del participante, estas se pueden colocar en un archivo `/video/participants.properties` en el dispositivo. Para generar la firma de voz, consulte [Transcripción de conversaciones (SDK)](../how-to-use-conversation-transcription-service.md).

   ![Aplicación de demostración de transcripción de conversaciones](../media/speech-devices-sdk/qsg-15.png)

1. Experimente.

## <a name="troubleshooting"></a>Solución de problemas

Si no se puede conectar a Speech Devices: Escriba el siguiente comando en una ventana del símbolo del sistema. Devolverá una lista de resultados:

```powershell
 adb devices
```

> [!NOTE]
> Este comando usa Android Debug Bridge, `adb.exe`, que forma parte de la instalación de Android Studio. Esta herramienta se encuentra en C:\Users\[nombre de usuario] \AppData\Local\Android\Sdk\platform-tools. Puede agregar este directorio a la ruta de acceso para que sea más práctico invocar `adb`. En caso contrario, debe especificar la ruta de acceso completa a la instalación de adb.exe en todos los comandos que invoca `adb`.
>
> Si ve un error `no devices/emulators found`, compruebe que el cable USB está conectado y que es un cable de alta calidad.
