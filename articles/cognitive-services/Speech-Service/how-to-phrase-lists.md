---
title: 'Listas de frases: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Aprenda cómo proporcionar al servicio de voz una lista de frases mediante el objeto `PhraseListGrammar` para mejorar los resultados de reconocimiento de voz a texto.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/04/2020
ms.author: trbye
zone_pivot_groups: programming-languages-speech-services-one-nomore-no-go
ms.custom: devx-track-javascript
ms.openlocfilehash: 69cd1ffaeb4cbc7bff8ad2c0c4e0611162d43b84
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87403981"
---
# <a name="phrase-lists-for-speech-to-text"></a>Listas de frases para reconocimiento de voz a texto

Al proporcionar al servicio de voz una lista de frases, puede mejorar la precisión del reconocimiento de voz. Las listas de frases se usan para identificar frases conocidas en datos de audio, como el nombre de una persona o una ubicación específica.

Por ejemplo, si tiene el comando "Mover a" y "Cerca" como posible destino que se puede decir, puede añadir la entrada "Mover a Cerca". Al agregar una frase, aumentará la probabilidad de que, cuando se reconozca el audio, se reconozca "Mover a Cerca" en lugar de "Mover acerca".

A una lista de frases se pueden agregar palabras solas o frases completas. Durante el reconocimiento, se usa una entrada de una lista de frases si el audio incluye una coincidencia exacta de toda una frase como frase independiente. Si no se encuentra una coincidencia exacta con la frase, el reconocimiento no está asistido.

>[!Note]
> Actualmente, las listas de frases solo admiten el inglés para la conversión de voz en texto.

## <a name="how-to-use-phrase-lists"></a>Cómo usar las listas de frases

Los ejemplos siguientes muestran cómo crear una lista de frases mediante el objeto `PhraseListGrammar`.

::: zone pivot="programming-language-csharp"

```cs
PhraseListGrammar phraseList = PhraseListGrammar.FromRecognizer(recognizer);
phraseList.AddPhrase("Move to Ward");
phraseList.AddPhrase("Move to Bill");
phraseList.AddPhrase("Move to Ted");
```

::: zone-end

::: zone pivot="programming-language-cpp"

```C++
auto phraselist = PhraseListGrammar::FromRecognizer(recognizer);
phraselist->AddPhrase("Move to Ward");
phraselist->AddPhrase("Move to Bill");
phraselist->AddPhrase("Move to Ted");
```

::: zone-end

::: zone pivot="programming-language-java"

```Java
PhraseListGrammar phraseListGrammar = PhraseListGrammar.fromRecognizer(recognizer);
phraseListGrammar.addPhrase("Move to Ward");
phraseListGrammar.addPhrase("Move to Bill");
phraseListGrammar.addPhrase("Move to Ted");
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
phrase_list_grammar = speechsdk.PhraseListGrammar.from_recognizer(reco)
phrase_list_grammar.addPhrase("Move to Ward")
phrase_list_grammar.addPhrase("Move to Bill")
phrase_list_grammar.addPhrase("Move to Ted")
```

::: zone-end

::: zone pivot="programming-language-javascript"

```JavaScript
var phraseListGrammar = SpeechSDK.PhraseListGrammar.fromRecognizer(reco);
phraseListGrammar.addPhrase("Move to Ward");
phraseListGrammar.addPhrase("Move to Bill");
phraseListGrammar.addPhrase("Move to Ted");
```

::: zone-end

>[!Note]
> El número máximo de listas de frases que usará el servicio de voz para establecer coincidencias con la voz es 1024 frases.

También puede borrar las frases asociadas con `PhraseListGrammar` llamando a clear().

::: zone pivot="programming-language-csharp"

```cs
phraseList.Clear();
```

::: zone-end

::: zone pivot="programming-language-cpp"

```C++
phraselist->Clear();
```

::: zone-end

::: zone pivot="programming-language-java"

```Java
phraseListGrammar.clear();
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
phrase_list_grammar.clear()
```

::: zone-end

::: zone pivot="programming-language-javascript"

```JavaScript
phraseListGrammar.clear();
```

::: zone-end

> [!NOTE]
> Los cambios realizados en el objeto `PhraseListGrammar` se aplican en el siguiente reconocimiento o tras una reconexión con el servicio de voz.

## <a name="next-steps"></a>Pasos siguientes

* [Documentación de referencia del SDK de voz](speech-sdk.md)
