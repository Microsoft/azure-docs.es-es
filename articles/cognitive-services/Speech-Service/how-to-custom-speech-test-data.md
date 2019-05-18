---
title: Preparar los datos de prueba para la conversión de voz personalizados - servicios de voz
titlesuffix: Azure Cognitive Services
description: Es si se está probando para ver el grado de precisión reconocimiento de voz de Microsoft o sus propios modelos de entrenamiento, tendrá datos (en forma de audio o texto). En esta página, trataremos los tipos de datos, cómo se usan y cómo administrarlas.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: af801600eebed7c0d4ff01dd1edf01fa595840eb
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65785773"
---
# <a name="prepare-data-for-custom-speech"></a>Preparar los datos de voz personalizado

Es si se está probando para ver el grado de precisión reconocimiento de voz de Microsoft o sus propios modelos de entrenamiento, necesitará los datos en forma de audio y el texto. En esta página, trataremos los tipos de datos, cómo se usan y cómo administrarlas.

## <a name="data-types"></a>Tipos de datos

Esta tabla enumeran los tipos de datos aceptados, cuándo se debe usar cada tipo de datos y la cantidad recomendada. No todos los tipos de datos es necesario para crear un modelo. Requisitos de datos variará dependiendo de si está creando una prueba o entrenar un modelo.

| Tipo de datos | Se utiliza de las pruebas | Cantidad | Usado para el entrenamiento | Cantidad |
|-----------|-----------------|----------|-------------------|----------|
| [Audio](#audio-data-for-testing) | Sí<br>Utilizado para su inspección visual | archivos de audio de más de 5 | No | N/a |
| [Transcripciones de audio + humanos con etiqueta](#audio--human-labeled-transcript-data-for-testingtraining) | Sí<br>Utilizado para evaluar la precisión | 0.5 - 5 horas de audio | Sí | 1 - 1000 horas de audio |
| [Texto relacionado](##related-text-data-for-training) | No | N/a | Sí | 1-200 MB de texto relacionado |

Los archivos deben ser agrupados por tipo en un conjunto de datos y cargar como un archivo zip. Cada conjunto de datos solo puede contener un tipo de datos único.

## <a name="upload-data"></a>Carga de datos

Cuando esté listo para cargar los datos, haga clic en **cargar datos** para iniciar el asistente y crear su primer conjunto de datos. Le pedirá que seleccione un tipo de datos de voz para el conjunto de datos, antes de permitir cargar los datos.

![Seleccione el audio desde el Portal de voz](./media/custom-speech/custom-speech-select-audio.png)

Cada conjunto de datos que cargue debe cumplir los requisitos para el tipo de datos que elija. Es importante dar formato correctamente a los datos antes de cargarse. Esto garantiza que el servicio de voz personalizado se procesará con precisión los datos. Los requisitos aparecen en las secciones siguientes.

Una vez cargado el conjunto de datos, tiene algunas opciones:

* Puede navegar a la **pruebas** pestaña e inspeccionar visualmente solo de audio o de datos de transcripción de audio + etiqueta humanos.
* Puede navegar a la **entrenamiento** pestaña y nosotros, los datos de la transcripción de audio + humano o datos de texto relacionado para entrenar un modelo personalizado.

## <a name="audio-data-for-testing"></a>Datos de audio para las pruebas

Datos de audio están óptimas para probar la precisión del modelo de voz a texto de línea de base de Microsoft o un modelo personalizado. Tenga en cuenta, los datos de audio se utilizan para inspeccionar la precisión de voz con respecto al rendimiento de un modelo específico. Si busca para cuantificar la precisión de un modelo, use [datos transcripción de audio + etiqueta humanos](#audio--human-labeled-transcript-data-for-testingtraining).

Use esta tabla para asegurarse de que los archivos de audio tienen el formato correcto para su uso con voz personalizada:

| Propiedad | Valor |
|----------|-------|
| Formato de archivo | RIFF (WAV) |
| Frecuencia de muestreo | 8.000 o 16.000 Hz |
| Canales | 1 (mono) |
| Longitud máxima por audio | 2 horas |
| Formato de ejemplo | PCM, 16 bits |
| Formato de archivo | .zip |
| Tamaño de archivo máximo | 2 GB |

Si el audio no satisfaga estas propiedades o desea comprobar si lo hace, se recomienda descargar [sox](http://sox.sourceforge.net) para comprobar o convierta el audio. A continuación se muestran algunos ejemplos de cómo cada una de estas actividades puede realizarse a través de la línea de comandos:

| Actividad | DESCRIPCIÓN | Comando de la ley SOX |
|----------|-------------|-------------|
| Compruebe el formato de audio | Utilice este comando para comprobar el formato de archivo de audio. | `soxi <filename>.wav` |
| Convertir el formato de audio | Use este comando para convertir el archivo de audio para el canal único, 16 bits, 48 KHz. | `sox <filename>.wav -b 16 -3 signed-integer -c l -r 48k -t wav <filename>.wav` |

## <a name="audio--human-labeled-transcript-data-for-testingtraining"></a>Datos de transcripción de audio + etiqueta humanos para las pruebas y aprendizaje

Para medir la exactitud de la precisión de texto a voz de Microsoft al procesar los archivos de audio, debe proporcionar la etiqueta humanos transcripciones (palabra por palabra) para la comparación. Mientras la transcripción de la etiqueta humanos consume mucho tiempo, es necesario para evaluar la precisión y para entrenar el modelo para los casos de uso. Tenga en cuenta, las mejoras en el reconocimiento solo será tan buenas como los datos proporcionados. Por ese motivo, es importante que las transcripciones de alta calidad solo se cargan.  

| Propiedad | Valor |
|----------|-------|
| Formato de archivo | RIFF (WAV) |
| Frecuencia de muestreo | 8.000 o 16.000 Hz |
| Canales | 1 (mono) |
| Longitud máxima por audio | 60 s |
| Formato de ejemplo | PCM, 16 bits |
| Formato de archivo | .zip |
| Tamaño máximo de archivo zip | 2 GB |

Para solucionar problemas, como la eliminación de word o sustitución, una cantidad significativa de datos es necesario para mejorar el reconocimiento. Por lo general, se recomienda para proporcionar las transcripciones de palabra por palabra de aproximadamente el 10 y 1.000 horas de audio. Las transcripciones para todos los archivos WAV deben incluirse en un único archivo de texto sin formato. Cada línea del archivo de transcripción debe contener el nombre de uno de los archivos de audio, seguido de la transcripción correspondiente. El nombre de archivo y la transcripción deben estar separados por un carácter de tabulación (\t).

  Por ejemplo:
```
  speech01.wav  speech recognition is awesome
  speech02.wav  the quick brown fox jumped all over the place
  speech03.wav  the lazy dog was not amused
```
> [!NOTE]
> La transcripción debería estar codificada como una marca BOM UTF-8.

Las transcripciones son texto normalizado para que el sistema pueda procesarlas. Sin embargo, hay algunas normalizaciones importantes que debe hacer el usuario _antes_ para cargar los datos a Custom Speech Service. Para que el idioma correspondiente que se utilizará al preparar sus transcripciones, consulte [cómo crear una transcripción de la etiqueta humanos](how-to-custom-speech-human-labeled-transcriptions.md)

Una vez que haya recopilado los archivos de audio y transcripciones correspondientes, se debe empaquetar como un único archivo .zip antes de cargarlo en el portal de voz personalizado. Se trata de un conjunto de datos de ejemplo con tres archivos de audio y un archivo de transcripción de la etiqueta humanos:

![Seleccione el audio desde el Portal de voz](./media/custom-speech/custom-speech-audio-transcript-pairs.png)

## <a name="related-text-data-for-training"></a>Datos de texto relacionado para el entrenamiento

Si tiene nombres de productos o características que son únicas y desea asegurarse de que se reconocen correctamente, es importante incluir datos de texto relacionado para el entrenamiento. Pueden proporcionar dos tipos de datos de texto relacionado para mejorar el reconocimiento:

| Tipo de datos | Cómo mejoran el reconocimiento de estos datos |
|-----------|------------------------------------|
| Grabaciones de voz o frases | Estos pueden mejorar la precisión al reconocer los nombres de producto o específicas del sector vocabulario dentro del contexto de una frase. |
| Pronunciaciones | Estos pueden mejorar la pronunciación de términos raras, acrónimos u otras palabras, con las pronunciaciones no definidas. |

Grabaciones de voz se pueden proporcionar como un único o varios archivos de texto. Cuanto más se acerque el texto están lo que se proporcionarán datos, mayor será la probabilidad de que se ha mejorado la precisión. Las pronunciaciones deben proporcionarse como un único archivo de texto. Todo lo que puede empaquetar como un solo archivo zip y cargado en el portal de voz personalizado.

### <a name="guidelines-to-create-an-utterances-file"></a>Directrices para crear un archivo de grabaciones de voz

Para crear un modelo personalizado con texto relacionado, deberá proporcionar una lista de declaraciones de ejemplo. Estas grabaciones de voz no es necesario ser frases completas o gramaticalmente corregir, pero deben reflejar con precisión la entrada de voz que espera en producción. Si desea que determinados términos a han aumentado el peso, puede agregar varias frases al archivo de datos relacionados que incluyan estos términos específicos.

Use esta tabla para asegurarse de que el archivo de datos relacionados para las grabaciones de voz se formateó correctamente:

| Propiedad | Valor |
|----------|-------|
| Codificación de texto | BOM UTF-8 |
| Número de expresiones por línea | 1 |
| Tamaño de archivo máximo | 200 MB |

Además, querrá tener en cuenta las siguientes restricciones:

* Evitar repetir caracteres más de cuatro veces. Por ejemplo: "aaaa" o "uuuu".
* No utilice caracteres especiales ni caracteres de UTF-8 anteriormente 00A1 U +.
* Se rechazarán los URI.

### <a name="guidelines-to-create-a-pronunciation-file"></a>Directrices para crear un archivo de pronunciación

Si hay términos raras sin pronunciaciones estándares que se encuentra o usar los usuarios, puede proporcionar un archivo pronunciación personalizado para mejorar el reconocimiento.

> [!IMPORTANT]
> No se recomienda usar esta característica para modificar la pronunciación de las palabras comunes.

Esto incluye ejemplos de una declaración de voz y una pronunciación personalizada para cada uno:

| Formato hablado | Reconoce y muestra el formulario |
|--------------|--------------------------|
| tres p c o | 3CPO |  
| c n t k | CNTK |
| i triple e | IEEE |

La forma hablada es la secuencia fonética deletreada. Puede estar formada de letra, palabras, sílabas o una combinación de los tres.

Pronunciación personalizado está disponible en inglés (en-US) y en alemán (de-DE). Esta tabla muestran los caracteres admitidos por idioma:

| Idioma | Configuración regional | Caracteres |
|----------|--------|------------|
| Inglés | es-ES | a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z |
| Alemán | de-DE | ä, ö, ü, a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z |

Use esta tabla para asegurarse de que el archivo de datos relacionados para las pronunciaciones se formateó correctamente. Pronunciación de los archivos es pequeños y no debería superar KBs unos.

| Propiedad | Valor |
|----------|-------|
| Codificación de texto | BOM de UTF-8 (ANSI también se admite para inglés) |
| n.º de pronunciaciones por línea | 1 |
| Tamaño de archivo máximo | 1 MB (1 KB por cada nivel gratis) |

## <a name="next-steps"></a>Pasos siguientes

* [Inspeccione los datos](how-to-custom-speech-inspect-data.md)
* [Evaluar los datos](how-to-custom-speech-evaluate-data.md)
* [Entrenar el modelo.](how-to-custom-speech-train-model.md)
* [Implementar el modelo](how-to-custom-speech-deploy-model.md)
