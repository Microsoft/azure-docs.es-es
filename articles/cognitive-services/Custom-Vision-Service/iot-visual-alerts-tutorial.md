---
title: 'Tutorial: Ejemplo de alertas visuales de IoT'
titleSuffix: Azure Cognitive Services
description: En este tutorial, usará Custom Vision con un dispositivo IoT para reconocer los estados visuales y notificarlos desde la fuente de vídeo de una cámara.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: tutorial
ms.date: 08/05/2020
ms.author: pafarley
ms.openlocfilehash: 5016191bb4f647c0bbfa71144bb4b4b554b080f4
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87834344"
---
# <a name="tutorial-use-custom-vision-with-an-iot-device-to-report-visual-states"></a>Tutorial: Uso de Custom Vision con un dispositivo IoT para notificar estados visuales

En esta aplicación de ejemplo se muestra cómo usar Custom Vision para entrenar un dispositivo con una cámara para detectar estados visuales. Puede ejecutar este escenario de detección en un dispositivo IoT mediante un modelo de ONNX exportado.

Un estado visual describe el contenido de una imagen: una sala vacía o con gente, una calzada vacía o con un camión, etc. En la imagen siguiente, puede ver la detección de la aplicación cuando un plátano o una manzana se colocan delante de la cámara.

![Animación de una interfaz de usuario que etiqueta la fruta delante de la cámara](./media/iot-visual-alerts-tutorial/scoring.gif)

Este tutorial le mostrará cómo:
> [!div class="checklist"]
> * Configurar la aplicación de ejemplo para usar sus propios recursos de Custom Vision e IoT Hub.
> * Usar la aplicación para entrenar el proyecto de Custom Vision.
> * Usar la aplicación para puntuar nuevas imágenes en tiempo real y enviar los resultados a Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar. 

## <a name="prerequisites"></a>Prerrequisitos

* [!INCLUDE [create-resources](includes/create-resources.md)]
    > [!IMPORTANT]
    > Este proyecto debe ser un proyecto de clasificación de imágenes **compacto**, ya que posteriormente se exportará el modelo a ONNX.
* También necesitará [crear un recurso de IoT Hub](https://ms.portal.azure.com/#create/Microsoft.IotHub) en Azure.
* [Visual Studio 2015 o posterior](https://www.visualstudio.com/downloads/)
* Opcionalmente, un dispositivo IoT que ejecute Windows 10 IoT Core versión 17763 o posterior. También puede ejecutar la aplicación directamente desde su equipo.
   * Si tiene Raspberry Pi 2 y 3, puede configurar Windows 10 directamente desde la aplicación del panel de IoT. Si tiene otros dispositivos, como DrangonBoard, deberá instalarla con el [método eMMC](https://docs.microsoft.com/windows/iot-core/tutorials/quickstarter/devicesetup#flashing-with-emmc-for-dragonboard-410c-other-qualcomm-devices). Si necesita ayuda para configurar un dispositivo nuevo, consulte [Configuración del dispositivo](https://docs.microsoft.com/windows/iot-core/tutorials/quickstarter/devicesetup) en la documentación de Windows IoT.

## <a name="about-the-visual-alerts-app"></a>Sobre la aplicación de alertas visuales

La aplicación de alertas visuales de IoT se ejecuta en un bucle continuo, que cambia entre cuatro estados diferenciados según corresponda:

* **No Model** (Sin modelo): estado no operativo. La aplicación se suspenderá continuamente durante un segundo y comprobará la cámara.
* **Capturing Training Images** (Capturando las imágenes de entrenamiento): en este estado, la aplicación captura una imagen y la carga como una imagen de entrenamiento en el proyecto de Custom Vision de destino. Después, la aplicación se suspende durante 500 ms y repite la operación hasta que se captura el número de imágenes de destino establecido. Seguidamente, se activa el entrenamiento del modelo de Custom Vision.
* **Waiting For Trained Model** (Esperando al modelo entrenado): en este estado, la aplicación llama a la API de Custom Vision cada segundo para comprobar si el proyecto de destino contiene una iteración entrenada. Cuando encuentra una, descarga el modelo de ONNX correspondiente en un archivo local y cambia al estado **Scoring** (Puntuando).
* **Scoring** (Puntuando): en este estado, la aplicación usa Windows ML para comparar un solo fotograma de la cámara con el modelo de ONNX local. La clasificación de imágenes resultante se muestra en la pantalla y se envía como un mensaje a IoT Hub. Después, la aplicación se suspende durante un segundo antes de puntuar una nueva imagen.

## <a name="understand-the-code-structure"></a>Estructura del código

Los siguientes archivos administran la funcionalidad principal de la aplicación.

| Archivo | Descripción |
|-------------|-------------|
| [MainPage.xaml](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/blob/master/IoTVisualAlerts/MainPage.xaml) | Este archivo define la interfaz de usuario de XAML. Hospeda el control de la cámara web y contiene las etiquetas usadas para las actualizaciones de estado.|
| [MainPage.xaml.cs](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/blob/master/IoTVisualAlerts/MainPage.xaml.cs) | Este código controla el comportamiento de la interfaz de usuario de XAML. Contiene el código de procesamiento de la máquina de estado.|
| [CustomVision\CustomVisionServiceWrapper.cs](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/blob/master/IoTVisualAlerts/CustomVision/CustomVisionServiceWrapper.cs) | Esta clase es un contenedor que administra la integración con el servicio Custom Vision.|
| [CustomVision\CustomVisionONNXModel.cs](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/blob/master/IoTVisualAlerts/CustomVision/CustomVisionONNXModel.cs) | Esta clase es un contenedor que administra la integración con Windows ML para cargar el modelo de ONNX y puntuar las imágenes con respecto a él.|
| [IoTHub\IotHubWrapper.cs](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/blob/master/IoTVisualAlerts/IoTHub/IoTHubWrapper.cs) | Esta clase es un contenedor que administra la integración con IoT Hub para cargar los resultados de puntuación en Azure.|

## <a name="set-up-the-visual-alerts-app"></a>Configuración de la aplicación de alertas visuales

Para que la aplicación de alertas visuales de IoT se ejecute en su equipo o dispositivo IoT, siga estos pasos.

1. Clone o descargue el [ejemplo IoTVisualAlerts](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/tree/master/IoTVisualAlerts) en GitHub.
1. Abra la solución _IoTVisualAlerts.sln_ en Visual Studio.
1. Integre su proyecto de Custom Vision:
    1. En el script _CustomVision\CustomVisionServiceWrapper.cs_, actualice la variable `ApiKey` con la clave de entrenamiento.
    1. Luego, actualice la variable `Endpoint` con la dirección URL del punto de conexión asociada a la clave.
    1. Actualice la variable `targetCVSProjectGuid` con el identificador correspondiente del proyecto Custom Vision que quiere usar. 
1. Configure el recurso de IoT Hub:
    1. En el script _IoTHub\IotHubWrapper.cs_, actualice la variable `s_connectionString` con la cadena de conexión adecuada para el dispositivo. 
    1. En Azure Portal, cargue la instancia de IoT Hub, haga clic en **Dispositivos IoT** en **Exploradores**, seleccione el dispositivo de destino (o cree uno si es necesario) y busque la cadena de conexión que se muestra en **Cadena de conexión principal**. La cadena contendrá el nombre de IoT Hub, el identificador de dispositivo y la clave de acceso compartida, con el siguiente formato: `{your iot hub name}.azure-devices.net;DeviceId={your device id};SharedAccessKey={your access key}`.

## <a name="run-the-app"></a>Ejecución la aplicación

Si va a ejecutar la aplicación en su equipo, seleccione **Máquina local** como dispositivo de destino en Visual Studio y seleccione **x64** o **x86** como plataforma de destino. Luego, presione F5 para ejecutar el programa. La aplicación debe iniciarse y mostrar la fuente en directo desde la cámara y un mensaje de estado.

Si va a realizar la implementación en un dispositivo IoT con un procesador ARM, deberá seleccionar **ARM** como plataforma de destino y **Máquina remota** como dispositivo de destino. Proporcione la dirección IP del dispositivo cuando se le solicite (debe estar en la misma red que el equipo). Puede obtener la dirección IP de la aplicación predeterminada de Windows IoT después de arrancar el dispositivo y conectarlo a la red. Presione F5 para ejecutar el programa.

Al ejecutar la aplicación por primera vez, no tendrá ninguna información sobre los estados visuales. Se mostrará un mensaje de estado que indica que no hay ningún modelo disponible. 

## <a name="capture-training-images"></a>Captura de las imágenes de entrenamiento

Para configurar un modelo, debe colocar la aplicación en el estado **Capturing Training Images** (Capturando imágenes de entrenamiento). Realice uno de los siguientes pasos:
* Si ejecuta la aplicación en el equipo, use el botón de la esquina superior derecha de la interfaz de usuario.
* Si ejecuta la aplicación en un dispositivo IoT, llame al método `EnterLearningMode` en el dispositivo mediante IoT Hub. Puede llamarlo mediante la entrada del dispositivo en el menú de IoT Hub en Azure Portal o con una herramienta como [IoT Hub Device Explorer](https://github.com/Azure/azure-iot-sdk-csharp).
 
Cuando la aplicación entre en el estado **Capturing Training Images** (Capturando imágenes de entrenamiento), capturará aproximadamente dos imágenes por segundo hasta que alcance el número de imágenes de destino. De forma predeterminada, son 30 imágenes, pero puede establecer este parámetro si pasa el número deseado como argumento al método `EnterLearningMode` de IoT Hub. 

Mientras la aplicación captura las imágenes, debe exponer la cámara a los tipos de estados visuales que quiera detectar (por ejemplo, una sala vacía o con gente, un escritorio vacío o con un camión de juguete, etc.).

## <a name="train-the-custom-vision-model"></a>Entrenamiento del modelo de Custom Vision

Cuando la aplicación haya terminado de capturar las imágenes, las cargará y cambiará al estado **Waiting For Trained Model** (Esperando al modelo entrenado). En este momento, debe ir al [portal de Custom Vision](https://www.customvision.ai/) y crear un modelo basado en las nuevas imágenes de entrenamiento. En la siguiente animación se muestra un ejemplo de este proceso.

![Animación: etiquetado de varias imágenes de plátanos](./media/iot-visual-alerts-tutorial/labeling.gif)

Para repetir este proceso con su propio escenario, siga estos pasos:

1. Inicie sesión en el [portal de Custom Vision](http://customvision.ai).
1. Busque el proyecto de destino, que ahora debe tener todas las imágenes de entrenamiento que cargó la aplicación.
1. Para cada estado visual que quiera identificar, seleccione las imágenes apropiadas y aplique manualmente la etiqueta.
    * Por ejemplo, si lo que quiere es distinguir entre una sala vacía y una con gente, se recomienda etiquetar cinco o más imágenes con gente como una nueva clase, **People**, y etiquetar cinco o más imágenes sin gente con la etiqueta **Negative**. De esta forma el modelo podrá diferenciar entre los dos estados.
    * Otro ejemplo: si lo que quiere es estimar lo llena que está una estantería, puede usar etiquetas como **EmptyShelf**, **PartiallyFullShelf** y **FullShelf**.
1. Cuando haya finalizado, seleccione el botón **Train** (Entrenar).
1. Una vez finalizado el entrenamiento, la aplicación detectará que hay disponible una iteración entrenada. Se iniciará el proceso de exportación del modelo entrenado a ONNX y se descargará en el dispositivo.

## <a name="use-the-trained-model"></a>Uso del modelo entrenado

Cuando la aplicación descargue el modelo entrenado, cambiará al estado **Scoring** (Puntuando) y comenzará a puntuar las imágenes de la cámara en un bucle continuo.

Cada vez que se capture una imagen, la aplicación mostrará la etiqueta superior en la pantalla. Si no se reconoce el estado visual, se mostrará **No Matches** (No hay resultados). La aplicación también envía estos mensajes a IoT Hub y, si se detecta una clase, el mensaje incluirá la etiqueta, la puntuación de confianza y una propiedad llamada `detectedClassAlert`, que pueden usar los clientes de IoT Hub interesados en realizar un enrutamiento rápido de mensajes basado en las propiedades.

Además, en el ejemplo se usa una [biblioteca de Sense HAT](https://github.com/emmellsoft/RPi.SenseHat) para detectar cuándo se ejecuta la aplicación en un dispositivo Raspberry PI con una unidad de Sense HAT. De este modo, se puede usar como presentación de salida y configurar todas las luces en rojo cada vez que se detecte una clase y en blanco cuando no se detecte nada.

## <a name="reuse-the-app"></a>Reutilización de la aplicación

Si quiere restablecer la aplicación a su estado original, haga clic en el botón situado en la esquina superior derecha de la interfaz de usuario o invoque el método `DeleteCurrentModel` mediante IoT Hub.

En cualquier momento, puede repetir el paso de cargar imágenes de entrenamiento; para ello, haga clic en el botón de la interfaz de usuario de la parte superior derecha o llame de nuevo al método `EnterLearningMode`.

Si ejecuta la aplicación en un dispositivo y necesita volver a recuperar la dirección IP (para establecer una conexión remota mediante el [cliente remoto de Windows IoT](https://www.microsoft.com/p/windows-iot-remote-client/9nblggh5mnxz#activetab=pivot:overviewtab), por ejemplo), puede llamar al método `GetIpAddress` por medio de IoT Hub.

## <a name="clean-up-resources"></a>Limpieza de recursos

Elimine el proyecto de Custom Vision si ya no quiere mantenerlo. En el [sitio web de Custom Vision](https://customvision.ai), vaya a **Projects** (Proyectos) y seleccione la papelera que aparece debajo del nuevo proyecto.

![Captura de pantalla de un panel denominado My New Project con el icono de papelera](./media/csharp-tutorial/delete_project.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha configurado y ha ejecutado una aplicación que detecta información de estado visual en un dispositivo IoT y envía los resultados a IoT Hub. Después, explore más el código fuente o realice una de las modificaciones sugeridas a continuación.

> [!div class="nextstepaction"]
> [Ejemplo de IoTVisualAlerts (GitHub)](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/tree/master/IoTVisualAlerts)

* Agregue un método de IoT Hub para cambiar la aplicación directamente al estado **Waiting For Trained Model** (Esperando al modelo entrenado). De este modo, puede entrenar el modelo con imágenes que no captura el propio dispositivo y, luego, insertar el nuevo modelo en el dispositivo con el comando.
* Siga el tutorial [Visualización de los datos del sensor en tiempo real](https://docs.microsoft.com/azure/iot-hub/iot-hub-live-data-visualization-in-power-bi) para crear un panel de Power BI y visualizar las alertas de IoT Hub que se han enviado con el ejemplo.
* Siga el tutorial [Supervisión remota de IoT](https://docs.microsoft.com/azure/iot-hub/iot-hub-monitoring-notifications-with-azure-logic-apps) para crear una aplicación lógica que responda a las alertas de IoT Hub cuando se detecten los estados visuales.
