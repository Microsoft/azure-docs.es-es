---
title: Tutorial de censura de rostros con Azure Media Analytics | Microsoft Docs
description: En este tema se muestran instrucciones paso a paso sobre cómo ejecutar un flujo de trabajo de censura completa mediante el Explorador de Azure Media Services (AMSE) y Azure Media Redactor Visualizer (herramienta de código abierto).
services: media-services
documentationcenter: ''
author: Lichard
manager: femila
editor: ''
ms.assetid: d6fa21b8-d80a-41b7-80c1-ff1761bc68f2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/20/2019
ms.author: ril
ms.reviewer: juliako
ms.openlocfilehash: df35b48a4da05d068760f0b04394ea53d788000e
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269701"
---
# <a name="redact-faces-with-azure-media-analytics-walkthrough"></a>Tutorial de censura de rostros con Azure Media Analytics

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Información general

**Redactor multimedia de Azure** es un procesador multimedia (MP) de [Análisis multimedia de Azure](media-services-analytics-overview.md) que ofrece censura de rostros escalable en la nube. La censura de rostros le permite modificar un vídeo con el fin de difuminar las caras de personas seleccionadas. Puede usar el servicio de censura de rostros en escenarios de seguridad pública y de noticias en los medios de comunicación. Unos minutos de material de archivo que contenga varias caras puede tardar horas en censurarse manualmente, pero con este servicio, el proceso de censura de caras requiere solamente unos pocos pasos sencillos. Para más información, consulte [este blog](https://azure.microsoft.com/blog/azure-media-redactor/).

Para obtener más información sobre **Azure Media Redactor**, consulte el tema de [información general sobre Censura de rostros](media-services-face-redaction.md).

En este tema se muestran instrucciones paso a paso sobre cómo ejecutar un flujo de trabajo de censura completa mediante el Explorador de Azure Media Services (AMSE) y Azure Media Redactor Visualizer (herramienta de código abierto).

Para más información, consulte [este blog](https://azure.microsoft.com/blog/redaction-preview-available-globally) .

## <a name="azure-media-services-explorer-workflow"></a>Flujo de trabajo del Explorador de Azure Media Services

La manera más fácil de empezar a trabajar con Redactor es usar la herramienta AMSE de código abierto en GitHub. Puede ejecutar un flujo de trabajo simplificado a través del modo **combinado** si no necesita acceso al JSON de anotaciones o a las imágenes JPG de rostros.

### <a name="download-and-setup"></a>Descarga e instalación

1. Descargue la herramienta AMSE para AMS v2 desde [aquí](https://aka.ms/amseforv2).
1. Inicie sesión en su cuenta de Media Services mediante la clave del servicio.

    Para obtener el nombre de la cuenta y la información de la clave, vaya a [Azure Portal](https://portal.azure.com/) y seleccione la cuenta de Azure Media Services. A continuación, seleccione Configuración > Claves. Aparece la ventana Administrar claves que muestra el nombre de la cuenta y la clave principal y la secundaria. Copie los valores del nombre de la cuenta y la clave principal.

![Censura de rostros](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough001.png)

### <a name="first-pass--analyze-mode"></a>Primer paso: Modo Analizar

1. Cargue el archivo multimedia a través de Activo – > Cargar, o bien arrástrelo y suéltelo. 
1. Haga clic en el botón derecho y procese el archivo multimedia utilizando Análisis multimedia > Azure Media Redactor – > Modo Analizar. 


![Censura de rostros](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough002.png)

![Censura de rostros](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough003.png)

El resultado incluirá un archivo JSON de anotaciones con datos de ubicación de rostros, así como un JPG de cada rostro detectado. 

![Censura de rostros](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough004.png)

### <a name="second-pass--redact-mode"></a>Segundo paso: Modo Redact (Censurar)

1. Cargue el recurso de vídeo original en la salida del primer paso y establézcalo como un recurso principal. 

    ![Censura de rostros](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough005.png)

2. Opcional: cargue un archivo Dance_idlist.txt que incluya una lista delimitada de nueva línea de los identificadores que se van a censurar. 

    ![Censura de rostros](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough006.png)

3. Opcional: realice las modificaciones en el archivo annotations.json, por ejemplo, aumentar los límites del cuadro de límite. 
4. Haga clic con el botón derecho en el recurso de salida del primer paso, seleccione Redactor y ejecútelo con el modo **Redact** (Censurar). 

    ![Censura de rostros](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough007.png)

5. Descargue o comparta el recurso de salida censurado final. 

    ![Censura de rostros](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough008.png)

## <a name="azure-media-redactor-visualizer-open-source-tool"></a>Herramienta de código abierto Azure Media Redactor Visualizer

La [herramienta de visualización](https://github.com/Microsoft/azure-media-redactor-visualizer) de código abierto está diseñada para ayudar a los desarrolladores a iniciarse en el formato de anotaciones analizando y utilizando la salida.

Después de clonar el repositorio, con el fin de ejecutar el proyecto, necesitará descargar FFMPEG desde el [sitio oficial](https://ffmpeg.org/download.html).

Si como desarrollador trata de analizar los datos de anotaciones del JSON, busque dentro de Models.MetaData ejemplos de código.

### <a name="set-up-the-tool"></a>Configuración de la herramienta

1.  Descargue y genere la solución completa. 

    ![Censura de rostros](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough009.png)

2.  Descargue FFMPEG desde [aquí](https://ffmpeg.org/download.html). Este proyecto se desarrolló originalmente mediante la versión be1d324 (2016-10-04) con vinculación estática. 
3.  Copie ffmpeg.exe y ffprobe.exe en la misma carpeta de salida que AzureMediaRedactor.exe. 

    ![Censura de rostros](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough010.png)

4. Ejecute AzureMediaRedactor.exe. 

### <a name="use-the-tool"></a>Uso de la herramienta

1. Procese sus vídeos en su cuenta de Azure Media Services con el módulo de administración de Redactor en Modo Analizar. 
2. Descargue el archivo de vídeo original y la salida de Redaction (Censura) - trabajo Analizar. 
3. Ejecute la aplicación de visualización y elija los archivos anteriores. 

    ![Censura de rostros](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough011.png)

4. Obtenga una vista previa del archivo. Seleccione qué rostros le gustaría desenfocar a través de la barra lateral de la derecha. 
    
    ![Censura de rostros](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough012.png)

5.  El campo de texto de la parte inferior se actualizará con los identificadores de los rostros. Cree un archivo denominado "idlist.txt" con estos identificadores como una lista delimitada de nueva línea. 

    >[!NOTE]
    > El archivo idlist.txt debe guardarse en ANSI. Puede usar el Bloc de notas para guardarlo en tal codificación.
    
6.  Cargue el archivo en el recurso de salida del paso 1. Cargue el vídeo original en este recurso y establézcalo como recurso principal. 
7.  Ejecute el trabajo Redaction (Censura) de este recurso con el modo "Redact" (Censurar) para obtener el vídeo censurado final. 

## <a name="next-steps"></a>Pasos siguientes 

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>Vínculos relacionados
[Información general de análisis de Azure Media Services](media-services-analytics-overview.md)

[Demostraciones de Azure Media Analytics](https://azuremedialabs.azurewebsites.net/demos/Analytics.html)

[Anuncio de función de censura facial para Azure Media Analytics](https://azure.microsoft.com/blog/azure-media-redactor/)
