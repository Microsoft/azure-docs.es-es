---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 08/11/2020
ms.author: trbye
ms.openlocfilehash: bee28c946242ecf227287fb36b5b03aa6defb1c2
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88170170"
---
[!INCLUDE [SPX Setup](../../spx-setup.md)]

## <a name="synthesize-speech-to-a-speaker"></a>Síntesis de voz en un altavoz

Ahora está listo para ejecutar la CLI de Voz para sintetizar voz a partir de texto. Desde la línea de comandos, cambie al directorio que contiene el archivo binario de la CLI de Voz. Luego, ejecute el siguiente comando.

```bash
spx synthesize --text "The speech synthesizer greets you!"
```

La CLI de Voz creará un lenguaje natural en inglés a través del altavoz del equipo.

## <a name="synthesize-speech-to-a-file"></a>Síntesis de voz en un archivo

Ejecute el siguiente comando para cambiar la salida del altavoz a un archivo `.wav`.

```bash
spx synthesize --text "The speech synthesizer greets you!" --audio output greetings.wav
```

La CLI de Voz generará lenguaje natural en inglés en el archivo de audio `greetings.wav`.
En Windows, puede reproducir el archivo de audio escribiendo `start greetings.wav`.