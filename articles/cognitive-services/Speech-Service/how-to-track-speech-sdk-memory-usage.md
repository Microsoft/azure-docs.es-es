---
title: 'Seguimiento del uso de memoria del SDK de voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: El SDK del servicio de voz admite un gran número de lenguajes de programación para la conversión de texto a voz y voz a texto, junto con la traducción de voz. En este artículo se describen las herramientas de administración de memoria integradas en el SDK.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 12/10/2019
ms.author: rhurey
ms.custom: devx-track-csharp
zone_pivot_groups: programming-languages-set-two
ms.openlocfilehash: c7e74ce8f4d0b7889d2e5bcd117eaa43e2b6991b
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934147"
---
# <a name="how-to-track-speech-sdk-memory-usage"></a>Seguimiento del uso de memoria del SDK de voz

El SDK de voz se basa en una base de código nativo que se proyecta en varios lenguajes de programación a través de una serie de niveles de interoperabilidad. Cada proyección específica del lenguaje tiene características correctas de forma idiomática para administrar el ciclo de vida de los objetos. Además, el SDK de voz incluye herramientas de administración de memoria para realizar el seguimiento del uso de recursos con el registro de objetos y los límites de objetos. 

## <a name="how-to-read-object-logs"></a>Cómo leer registros de objetos

Si [está habilitado el registro del SDK de voz](how-to-use-logging.md), se emiten etiquetas de seguimiento para habilitar la observación histórica de objetos. Estas etiquetas incluyen: 

* `TrackHandle` o `StopTracking` 
* El tipo de objeto
* El número actual de objetos de los que se realiza el seguimiento del tipo del objeto y el número actual del que se realiza el seguimiento.

Aquí tiene un registro de muestra: 

```terminal
(284): 8604ms SPX_DBG_TRACE_VERBOSE:  handle_table.h:90 TrackHandle type=Microsoft::CognitiveServices::Speech::Impl::ISpxRecognitionResult handle=0x0x7f688401e1a0, ptr=0x0x7f688401e1a0, total=19
```

## <a name="set-a-warning-threshold"></a>Defina un umbral de advertencia

Tiene la opción de crear un umbral de advertencia y, si se supera (suponiendo que el registro está habilitado), se registra un mensaje de advertencia. El mensaje de advertencia contiene un volcado de todos los objetos que existen junto con su recuento. Esta información se puede usar para comprender mejor los problemas. 

Para habilitar un umbral de advertencia, debe especificarse en un objeto `SpeechConfig`. Este objeto se comprueba cuando se crea un nuevo reconocedor. En los ejemplos siguientes, supongamos que ha creado una instancia de `SpeechConfig` denominada `config`:

::: zone pivot="programming-language-csharp"

```csharp
config.SetProperty("SPEECH-ObjectCountWarnThreshold", "10000");
```

::: zone-end

::: zone pivot="programming-language-cpp"

```C++
config->SetProperty("SPEECH-ObjectCountWarnThreshold", "10000");
```

::: zone-end

::: zone pivot="programming-language-java"

```java
config.setProperty("SPEECH-ObjectCountWarnThreshold", "10000");
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
speech_config.set_property_by_name("SPEECH-ObjectCountWarnThreshold", "10000")?
```

::: zone-end

::: zone pivot="programming-language-more"

```ObjectiveC
[config setPropertyTo:@"10000" byName:"SPEECH-ObjectCountWarnThreshold"];
```

::: zone-end

> [!TIP]
> El valor predeterminado de esta propiedad es 10 000.

## <a name="set-an-error-threshold"></a>Definir un umbral de error 

Con el SDK de voz, puede establecer el número máximo de objetos permitidos en un momento dado. Si esta opción está habilitada, cuando se alcance el número máximo, se producirá un error al intentar crear nuevos objetos de reconocedor. Los objetos existentes seguirán funcionando.

Este es un error de muestra:

```terminal
Runtime error: The maximum object count of 500 has been exceeded.
The threshold can be adjusted by setting the SPEECH-ObjectCountErrorThreshold property on the SpeechConfig object.
See http://https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-object-tracking-speech-sdk for more detailed information.
Handle table dump by ojbect type:
class Microsoft::CognitiveServices::Speech::Impl::ISpxRecognitionResult 0
class Microsoft::CognitiveServices::Speech::Impl::ISpxRecognizer 0
class Microsoft::CognitiveServices::Speech::Impl::ISpxAudioConfig 0
class Microsoft::CognitiveServices::Speech::Impl::ISpxSpeechConfig 0
```

Para habilitar un umbral de error, se debe especificar en un objeto `SpeechConfig`. Este objeto se comprueba cuando se crea un nuevo reconocedor. En los ejemplos siguientes, supongamos que ha creado una instancia de `SpeechConfig` denominada `config`:

::: zone pivot="programming-language-csharp"

```csharp
config.SetProperty("SPEECH-ObjectCountErrorThreshold", "10000");
```

::: zone-end

::: zone pivot="programming-language-cpp"

```C++
config->SetProperty("SPEECH-ObjectCountErrorThreshold", "10000");
```

::: zone-end

::: zone pivot="programming-language-java"

```java
config.setProperty("SPEECH-ObjectCountErrorThreshold", "10000");
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
speech_config.set_property_by_name("SPEECH-ObjectCountErrorThreshold", "10000")?
```

::: zone-end

::: zone pivot="programming-language-more"

```objc
[config setPropertyTo:@"10000" byName:"SPEECH-ObjectCountErrorThreshold"];
```

::: zone-end

> [!TIP]
> El valor predeterminado de esta propiedad es el valor máximo específico de la plataforma para un tipo de datos `size_t`. Un reconocimiento típico utilizará entre 7 y 10 objetos internos.

## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre el SDK de voz](speech-sdk.md)