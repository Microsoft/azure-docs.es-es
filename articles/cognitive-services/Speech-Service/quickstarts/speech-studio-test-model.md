---
title: Prueba de un modelo mediante archivos de audio en Speech Studio
titleSuffix: Azure Cognitive Services
description: En este procedimiento, se usa Speech Studio para probar el reconocimiento de voz en un archivo de audio.
services: cognitive-services
author: v-demjoh
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 09/04/2020
ms.author: v-demjoh
ms.openlocfilehash: 9101944a567df5000d3584ed48eff24e4c5e0057
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89565810"
---
# <a name="test-a-model-using-an-audio-file-in-speech-studio"></a>Prueba de un modelo mediante archivos de audio en Speech Studio

En este procedimiento, se usa Speech Studio para convertir la voz de un archivo de audio en texto. Speech Studio le permite probar, comparar, mejorar e implementar modelos de reconocimiento de voz mediante el texto relacionado, el audio con transcripciones con etiqueta humana y las instrucciones pronunciación que proporcione.

## <a name="prerequisites"></a>Requisitos previos

Antes de usar el portal de Voz, [siga estas instrucciones para crear una cuenta de Azure y suscribirse al servicio Voz](../how-to-custom-speech.md#set-up-your-azure-account). Esta suscripción unificada proporciona acceso a la conversión de voz a texto, la conversión de texto a voz, la traducción de voz y el portal de Habla personalizada.

## <a name="download-an-audio-file"></a>Descarga de un archivo de audio

Siga estos pasos para descargar un archivo de audio que contenga voz y empaquetarlo en un archivo .zip.

1. Para descargar el **[archivo wav desde este vínculo](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-speech-sdk/f9807b1079f3a85f07cbb6d762c6b5449d536027/samples/cpp/windows/console/samples/whatstheweatherlike.wav)** , haga clic con el botón derecho en el vínculo y seleccione **Guardar vínculo como**. Haga clic en **Guardar** para descargar el archivo `whatstheweatherlike.wav`.
2. Mediante un explorador de archivos o una ventana de terminal con una herramienta de compresión, cree un archivo .zip denominado `whatstheweatherlike.zip` que contenga el archivo `whatstheweatherlike.wav` descargado. En Windows, puede abrir el Explorador de Windows, ir a la carpeta `Downloads`, hacer clic con el botón derecho en `whatstheweatherliike.wav`, hacer clic en **Enviar a**, hacer clic en **Carpeta comprimida (en zip)** y presionar Entrar para aceptar el nombre de archivo predeterminado.

## <a name="create-a-project-in-the-custom-speech-portal"></a>Creación de un proyecto en el portal de Habla personalizada

Siga estos pasos para crear un proyecto que contenga el archivo .zip de un archivo de audio.

1. Abra [Speech Studio](https://speech.microsoft.com/) y haga clic en **Nuevo proyecto**. Escriba un nombre para este proyecto y haga clic en **Crear**. El proyecto aparece en la lista Habla personalizada.
2. Haga clic en el nombre del proyecto. Vaya a la pestaña Datos, haga clic en **Cargar datos**.
3. El tipo de datos de voz se establece de forma predeterminada en **Solo audio**. Por tanto, haga clic en **Siguiente**.
4. Asigne al nuevo conjunto de datos de voz el nombre `MyZipOfAudio` y haga clic en **Siguiente**.
5. Haga clic en **Examinar archivos**, navegue hasta el archivo `whatstheweatherlike.zip` y haga clic en **Abrir**.
6. Haga clic en el botón **Upload** . El explorador carga el archivo .zip en Speech Studio, y Speech Studio procesa el contenido.

## <a name="test-a-model"></a>Prueba de un modelo

Después de que Speech Studio procese el contenido del archivo .zip, puede reproducir el audio de origen mientras examina la transcripción para buscar errores u omisiones. Siga estos pasos para examinar la calidad de la transcripción en el explorador.

1. Haga clic en la pestaña **Pruebas** y, a continuación, en **Agregar prueba**.
2. En esta prueba, vamos a inspeccionar la calidad de los datos de solo audio, así que haga clic en **Siguiente** para aceptar este tipo de prueba.
3. Asigne a esta prueba el nombre `MyModelTest` y haga clic en **Siguiente**.
4. Haga clic en el botón de opción situado a la izquierda de `MyZipOfAudio` y, a continuación, en **Siguiente**.
5. De forma predeterminada, el valor de la lista desplegable **Modelo 1** se establece en el modelo de reconocimiento más reciente, por lo que debe hacer clic en **Crear**. Después de procesar el contenido del conjunto de cambios de audio, el estado de la prueba cambiará a **Correcto**.
6. Haga clic en **MyModelTest**. Aparecen los resultados del reconocimiento de voz. Haga clic en el triángulo que señala a la derecha dentro del círculo para oír el audio y compare lo que escucha con el texto del círculo.

## <a name="download-detailed-results"></a>Descarga de los resultados detallados

Puede descargar archivos que describan transcripciones con mucho más detalle. Los archivos incluyen la forma léxica de la voz en los archivos de audio, y archivos JSON que contienen detalles de confianza de desplazamiento, duración y transcripción de cada palabra. Siga los pasos que se indican a continuación para ver estos archivos.

1. Haga clic en **Descargar**.
2. En el cuadro de diálogo Descargar, anule la selección de **Audio** y haga clic en **Descargar**.
3. Descomprima el archivo .zip descargado y examine los archivos extraídos.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo mejorar la precisión del reconocimiento de voz mediante el [entrenamiento de un modelo personalizado](../how-to-custom-speech-test-and-train.md).