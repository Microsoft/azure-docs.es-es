---
title: Creación de flujos de trabajo de Encoding avanzado con el Diseñador de flujo de trabajo | Microsoft Docs
description: Aprenda a crear flujos de trabajo de codificación avanzada con el Diseñador de flujo de trabajo.
services: media-services
documentationcenter: ''
author: anilmur
manager: femila
editor: ''
ms.assetid: 004815f2-0761-4706-87a1-675ba36e0322
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: anilmur
ms.reviewer: juliako;johndeu
ms.openlocfilehash: a53027ec1f775c240164f67e7ee0473a44690964
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89262692"
---
# <a name="create-advanced-encoding-workflows-with-workflow-designer"></a>Creación de flujos de trabajo de Encoding avanzado con el Diseñador de flujo de trabajo

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Información general
El **Diseñador de flujo de trabajo** es una herramienta de escritorio de Windows que se usa para diseñar y crear flujos de trabajo personalizados para la codificación con el **Flujo de trabajo del Codificador multimedia premium**.
Mediante la eficacia de la herramienta del diseñador de flujos de trabajo, puede diseñar y crear complejos flujos de trabajo que se ejecutarán en el **Codificador multimedia Premium**.  

Los flujos de trabajo pueden incluir bifurcación y lógica de decisión de clientes en función de las propiedades del archivo de origen de entrada. Puede crear flujos de trabajo con propiedades reemplazables y valores dinámicos para facilitar la repetición incluso de las tareas de codificación más complejas y su personalización en la nube.

Entre los flujos de trabajo de ejemplo que puede crear se incluyen:

* Los flujos de trabajo basados en decisiones que inspeccionan el contenido de origen para resolución y solo codifican las pistas de salida deseadas.  Esto resulta de utilidad al eliminar las pistas desaprovechadas que se generarían al escalar el contenido de origen de manera involuntaria.
* Se pueden usar varios archivos de entrada para admitir títulos, superposiciones y contenido agrupado. 

Esta herramienta también se puede usar para modificar cualquiera de nuestros [flujos de trabajo publicados](media-services-workflow-designer.md#existing_workflows). 

> [!NOTE]
> Para obtener su copia de la herramienta Diseñador de flujo de trabajo, póngase en contacto con mepd@microsoft.com.

Una vez que creado un archivo de flujo de trabajo, se puede cargar como un activo y, a continuación, usarse para la codificación de archivos multimedia. Para más información sobre cómo codificar con el **Flujo de trabajo del Codificador multimedia premium** mediante **.NET**, consulte [Codificación avanzada con el Flujo de trabajo del Codificador multimedia premium](media-services-encode-with-premium-workflow.md).

## <a name="modify-existing-workflows"></a><a id="existing_workflows"></a>Modificación de flujos de trabajo existentes
Los [flujos de trabajo predeterminados](media-services-workflow-designer.md#existing_workflows) pueden modificarse con la herramienta de diseñador. Puede obtener los archivos de flujo de trabajo predeterminados [aquí](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows). La carpeta también contiene la descripción de estos archivos.

Los vídeos siguientes muestran cómo usar el diseñador.

### <a name="day-1--getting-started"></a>Día 1 – Introducción
El vídeo del día 1 trata de:

* Información general del diseñador
* Flujos de trabajo básicos: "Hello World"
* Creación de varios archivos MP4 de salida para usarlos con streaming de Azure Media Services

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Premium-Encoder-Workflow-Designer-Training-Videos-Day-1/player]
> 
> 

### <a name="day-2"></a>Día 2
El vídeo del día 2 trata de:

* Diversos escenarios de archivo de código fuente: control del audio
* Flujos de trabajo con lógica avanzada
* Fases de gráfico

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Premium-Encoder-Workflow-Designer-Training-Videos-Day-2/player]
> 
> 

### <a name="day-3"></a>Día 3
El vídeo del día 3 trata de:

* Scripting dentro de los flujos de trabajo y planos
* Restricciones con el codificador actual
* Preguntas y respuestas

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Premium-Encoder-Workflow-Designer-Training-Videos-Day-3/player]
> 
> 

## <a name="need-help"></a>¿Necesita ayuda?

Puede abrir una incidencia de soporte técnico si se desplaza a la [nueva solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-step"></a>Paso siguiente
Consulte las rutas de aprendizaje de Media Services.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Consulte también
[Azure Premium Encoder Workflow Designer Training Videos (Vídeos de aprendizaje del Diseñador de flujo de trabajo del codificador de Azure Premium)](http://johndeutscher.com/2015/07/06/azure-premium-encoder-workflow-designer-training-videos/)

