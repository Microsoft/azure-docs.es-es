---
title: Preguntas frecuentes sobre los contenedores del servicio de voz
titleSuffix: Azure Cognitive Services
description: Instale y ejecute contenedores de voz. La conversión de voz en texto transcribe los flujos de audio en texto en tiempo real que sus aplicaciones, herramientas o dispositivos pueden usar o mostrar. Texto a voz convierte el texto de entrada en voz sintetizada similar a la humana.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/24/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: b13a6944290f58f5ede239dee60610d67fff8b1c
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88918475"
---
# <a name="speech-service-containers-frequently-asked-questions-faq"></a>Preguntas frecuentes sobre los contenedores del servicio de voz

Cuando use el servicio de voz con contenedores, consulte primero estas preguntas frecuentes antes de escalar el problema al soporte técnico. En este artículo se recopilan preguntas de distinto grado, desde las generales a las más técnicas. Para expandir una respuesta, haga clic en la pregunta.

## <a name="general-questions"></a>Preguntas generales

<details>
<summary>
<b>¿Cómo funcionan los contenedores de voz y cómo se configuran?</b>
</summary>

**Respuesta:** Al configurar el clúster de producción, hay varios aspectos que deben tenerse en cuenta. En primer lugar, la configuración de un solo idioma en varios contenedores y en la misma máquina no debe ser un problema importante. Si tiene problemas, podrían estar relacionados con el hardware, por lo que primero veremos los recursos, es decir, las especificaciones de la CPU y memoria.

Consideremos de momento el contenedor `ja-JP` y el modelo más reciente. El modelo acústico es la parte más exigente en cuanto a la CPU, mientras que el modelo de lenguaje es el que exige más memoria. Cuando comparamos el uso, se necesitan unos 0,6 núcleos de CPU para procesar una única solicitud de conversión de voz en texto cuando el audio fluye en tiempo real (por ejemplo, desde el micrófono). Si el audio entra más rápido que en tiempo real (por ejemplo, desde un archivo), ese uso puede duplicarse (1,2 veces más núcleos). Mientras tanto, la memoria que se muestra a continuación es la memoria operativa para descodificar la voz. *No* tiene en cuenta el tamaño total real del modelo de lenguaje, que residirá en la caché de archivos. Para `ja-JP`, son 2 GB adicionales; para `en-US`, pueden ser más (6 o 7 GB).

Si tiene una máquina con poca memoria e intenta implementar varios idiomas en ella, es posible que la caché de archivos se llene y el sistema operativo se vea forzado a agregar o quitar modelos de la página. En el caso de una transcripción en curso, esto podría ser desastroso ya que daría lugar a ralentizaciones y otros problemas de rendimiento.

Además, se han empaquetado previamente ejecutables para las máquinas con el conjunto de instrucciones de la [extensión de vector avanzada (AVX2)](speech-container-howto.md#advanced-vector-extension-support). Una máquina con el conjunto de instrucciones AVX512 requerirá generar código para ese destino, e iniciar 10 contenedores para 10 idiomas podría agotar temporalmente la CPU. Aparecerá un mensaje similar al siguiente en los registros de Docker:

```console
2020-01-16 16:46:54.981118943 
[W:onnxruntime:Default, tvm_utils.cc:276 LoadTVMPackedFuncFromCache]
Cannot find Scan4_llvm__mcpu_skylake_avx512 in cache, using JIT...
```

Por último, puede establecer el número de descodificadores que desee dentro de un *único* contenedor con la variable `DECODER MAX_COUNT`. Por tanto, básicamente, debemos empezar con su SKU (CPU/memoria) y podemos sugerir cómo sacar el máximo partido de ella. Un buen punto de partida es referirse a las especificaciones de recursos recomendados de la máquina host.

<br>
</details>

<details>
<summary>
<b>¿Puede ayudarme a planear la capacidad y estimar los costos de los contenedores de voz a texto locales?</b>
</summary>

**Respuesta:** Para calcular la capacidad del contenedor en el modo de procesamiento por lotes, cada descodificador podría controlar dos o tres veces el tiempo real, con dos núcleos de CPU, para un solo reconocimiento. No se recomienda mantener más de dos reconocimientos simultáneos por instancia de contenedor, sino que se recomienda ejecutar más instancias de contenedores, por motivos de confiabilidad y disponibilidad, detrás de un equilibrador de carga.

Aunque podríamos tener cada instancia de contenedor ejecutándose con más descodificadores. Por ejemplo, es posible configurar siete descodificadores por instancia de contenedor en una máquina de ocho núcleos (2 veces más cada una), lo que produce un rendimiento 15 veces mayor. Se debe tener en cuenta el parámetro `DECODER_MAX_COUNT`. En el caso extremo, surgen problemas de confiabilidad y latencia, con un aumento significativo del rendimiento. En el caso de un micrófono, será en tiempo real. El uso general debe ser aproximadamente un núcleo para un único reconocimiento.

En un escenario de procesamiento de 1000 horas al día en modo de procesamiento por lotes, en un caso extremo, podrían controlarlo tres máquinas virtuales en 24 horas, pero no se garantiza. Para controlar los días de pico, la conmutación por error y las actualizaciones, y para proporcionar una copia de seguridad o BCP mínima, se recomienda tener cuatro o cinco máquinas en lugar de tres por clúster, y con más de dos clústeres.

En el caso del hardware, usamos una máquina virtual de Azure estándar `DS13_v2` como referencia (cada núcleo debe ser de 2,6 GHz o superior, con el conjunto de instrucciones AVX2 habilitado).

| Instancia  | vCPU | RAM    | Almacenamiento temporal | Pago por uso con AHB | Reserva de un año con AHB (porcentaje de ahorro) | Reserva de tres años con AHB (porcentaje de ahorro) |
|-----------|---------|--------|--------------|------------------------|-------------------------------------|--------------------------------------|
| `DS13 v2` | 8       | 56 GiB | 112 GiB      | 0,598 dólares/hora            | 0,3528 dólares/hora (~41 %)                 | 0,2333 dólares/hora (~61 %)                  |

En función de la referencia de diseño (dos clústeres de cinco máquinas virtuales para controlar el procesamiento por lotes de audio de 1000 horas al día), el costo de hardware de un año será:

> 2 (clústeres) * 5 (máquinas virtuales por clúster) * 0,3528 dólares/hora * 365 (días) * 24 (horas) = 31 000 dólares/año

Al realizar la asignación a la máquina física, una estimación general es 1 vCPU = 1 núcleo de CPU físico. En realidad, 1 vCPU es más eficaz que un solo núcleo.

En el caso de un entorno local, todos estos factores adicionales entran en juego:

- En qué tipo de CPU física se encuentra y cuántos núcleos hay en ella
- Cuántas CPU se ejecutan juntas en el mismo cuadro o máquina
- Cómo se han configurado las máquinas virtuales
- Cómo se usa Hyper-Threading o multithreading
- Cómo se comparte la memoria
- El sistema operativo, etc.

Normalmente, no está tan bien optimizado como el entorno de Azure. Teniendo en cuenta otra sobrecarga, se podría decir que una estimación segura es de 10 núcleos de CPU físicas = 8 vCPU de Azure. Sin embargo, las CPU populares solo tienen ocho núcleos. Con la implementación local, el costo será mayor que con el uso de máquinas virtuales de Azure. Además, hay que tener en cuenta la tasa de amortización.

El costo del servicio es el mismo que el servicio en línea: 1 dólar/hora para la conversión de voz en texto. El servicio de voz es:

> 1 dólar * 1000 * 365 = 365 000 dólares

El costo de mantenimiento que se paga a Microsoft depende del nivel de servicio y del contenido del servicio. Varía de 29,99 dólares al mes para el nivel básico a cientos de miles si se trata de un servicio in situ. Un número aproximado es de 300 dólares por hora por servicio o mantenimiento. No se incluye el costo del personal. No se incluyen otros costos de infraestructura (como almacenamiento, redes y equilibradores de carga).

<br>
</details>

<details>
<summary>
<b>¿Por qué falta la puntuación en la transcripción?</b>
</summary>

**Respuesta:** El idioma `speech_recognition_language=<YOUR_LANGUAGE>` se debe configurar explícitamente en la solicitud si se está usando el cliente Carbon.

Por ejemplo:

```python
if not recognize_once(
    speechsdk.SpeechRecognizer(
        speech_config=speechsdk.SpeechConfig(
            endpoint=template.format("interactive"),
            speech_recognition_language="ja-JP"),
            audio_config=audio_config)):

    print("Failed interactive endpoint")
    exit(1)
```
Este es el resultado:

```cmd
RECOGNIZED: SpeechRecognitionResult(
    result_id=2111117c8700404a84f521b7b805c4e7, 
    text="まだ早いまだ早いは猫である名前はまだないどこで生まれたかとんと見当を検討をなつかぬ。
    何でも薄暗いじめじめした所でながら泣いていた事だけは記憶している。
    まだは今ここで初めて人間と言うものを見た。
    しかも後で聞くと、それは書生という人間中で一番同額同額。",
    reason=ResultReason.RecognizedSpeech)
```

<br>
</details>

<details>
<summary>
<b>¿Puedo usar un modelo acústico personalizado y un modelo de lenguaje con el contenedor de voz?</b>
</summary>

Actualmente solo se puede pasar un identificador de modelo, un modelo de lenguaje personalizado o un modelo acústico personalizado.

**Respuesta:** Se ha tomado la decisión de *no* admitir ambos modelos acústicos y de lenguaje simultáneamente. Esto permanecerá en vigor hasta que se cree un identificador unificado para reducir las interrupciones de la API. Por lo tanto, no se admite en este momento.

<br>
</details>

<details>
<summary>
<b>¿Pueden explicar estos errores del contenedor de conversión de voz en texto personalizada?</b>
</summary>

**Error 1:**

```cmd
Failed to fetch manifest: Status: 400 Bad Request Body:
{
    "code": "InvalidModel",
    "message": "The specified model is not supported for endpoint manifests."
}
```

**Respuesta 1:** Si realiza el entrenamiento con el modelo personalizado más reciente, no se admite en estos momentos. Si lo realiza con una versión anterior, debería ser posible usarla. Todavía estamos trabajando para admitir las versiones más recientes.

Esencialmente, los contenedores personalizados no admiten los modelos acústicos basados en Halide u ONNX (que es el modelo predeterminado en el portal de entrenamiento personalizado). Esto se debe a que los modelos personalizados no están cifrados y no queremos exponer modelos de ONNX; sin embargo, los modelos de lenguaje son correctos. El cliente tendrá que seleccionar explícitamente un modelo que no sea ONNX anterior para el entrenamiento personalizado. La precisión no se verá afectada. El tamaño del modelo puede ser mayor (100 MB).

> Modelo admitido > 20190220 (v4.5 Unified)

**Error 2:**

```cmd
HTTPAPI result code = HTTPAPI_OK.
HTTP status code = 400.
Reason:  Synthesis failed.
StatusCode: InvalidArgument,
Details: Voice does not match.
```

**Respuesta 2:** Debe proporcionar el nombre de voz correcto en la solicitud, que distingue mayúsculas de minúsculas. Consulte la asignación de nombres de servicio completa. Tiene que usar `en-US-JessaRUS`, ya que `en-US-JessaNeural` no está disponible en la versión del contenedor de conversión de texto en voz.

**Error 3:**

```json
{
    "code": "InvalidProductId",
    "message": "The subscription SKU \"CognitiveServices.S0\" is not supported in this service instance."
}
```

**Respuesta:** Puede crear un recurso de voz, no un recurso de Cognitive Services.


<br>
</details>

<details>
<summary>
<b>¿Qué protocolos de API se admiten, REST o WS?</b>
</summary>

**Respuesta:** Para los contenedores de conversión de voz en texto y conversión de voz en texto personalizada, actualmente solo se admite el protocolo basado en websocket. El SDK solo admite llamadas en WS pero no en REST. Hay un plan para agregar compatibilidad de REST, pero no hay plazo estimado por el momento. Consulte siempre la documentación oficial, como [Consulta del punto de conexión de predicción del contenedor](speech-container-howto.md#query-the-containers-prediction-endpoint).

<br>
</details>

<details>
<summary>
<b>¿Es compatible CentOS con los contenedores de voz?</b>
</summary>

**Respuesta:** El SDK de Python todavía no admite CentOS 7 y tampoco se admite Ubuntu 19.04.

El paquete del SDK de Voz de Python está disponible para estos sistemas operativos:
- **Windows**: x64 y x86
- **Mac**: macOS X versión 10.12 o posterior
- **Linux**: Ubuntu 16.04, Ubuntu 18.04, Debian 9 en x64

Para más información sobre la configuración del entorno, consulte [Inicio rápido: Configuración del entorno de desarrollo](quickstarts/setup-platform.md?pivots=programming-language-python). Por ahora, Ubuntu 18.04 es la versión recomendada.

<br>
</details>

<details>
<summary>
<b>¿Por qué obtengo errores al intentar llamar a los puntos de conexión de predicción de LUIS?</b>
</summary>

Utilizo el contenedor de LUIS en una implementación de IoT Edge e intento llamar al punto de conexión de predicción de LUIS desde otro contenedor. El contenedor de LUIS está escuchando en el puerto 5001 y la dirección URL que utilizo es esta:

```csharp
var luisEndpoint =
    $"ws://192.168.1.91:5001/luis/prediction/v3.0/apps/{luisAppId}/slots/production/predict";
var config = SpeechConfig.FromEndpoint(new Uri(luisEndpoint));
```

El error que obtengo es:

```cmd
WebSocket Upgrade failed with HTTP status code: 404 SessionId: 3cfe2509ef4e49919e594abf639ccfeb
```

Veo la solicitud en los registros del contenedor de LUIS y el mensaje dice:

```cmd
The request path /luis//predict" does not match a supported file type.
```

¿Qué significa? ¿Qué me falta? Estoy siguiendo el ejemplo del SDK de voz [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk). El escenario es que estamos detectando el audio directamente del micrófono del PC y estamos tratando de determinar la intención, basándonos en la aplicación LUIS que entrenamos. El ejemplo con el que lo he vinculado hace exactamente eso. Y funciona bien con el servicio basado en la nube de LUIS. El uso del SDK de voz parecía evitar tener que realizar una llamada explícita e independiente a la API de conversión de voz en texto y, después, una segunda llamada a LUIS.

Por lo tanto, todo lo que intento hacer es cambiar del escenario de usar LUIS en la nube a usar el contenedor de LUIS. No puedo imaginar que si el SDK de voz funciona para uno, no funcione para el otro.

**Respuesta:** El SDK de voz no debe usarse en un contenedor de LUIS. Para usar el contenedor de LUIS, se debe usar el SDK de LUIS o la API REST de LUIS. El SDK de voz debe usarse en un contenedor de voz.

Una nube es diferente de un contenedor. Una nube puede estar formada por varios contenedores agregados (a veces denominados microservicios). Por lo tanto, hay un contenedor de LUIS y también hay un contenedor de voz, dos contenedores independientes. El contenedor de voz solo se encarga de la voz. El contenedor de LUIS solo se encarga de LUIS. Como sabemos que ambos contenedores están implementados y que no es eficiente que un cliente remoto tenga que ir a la nube, encargarse de la voz, volver y luego ir a la nube de nuevo y encargarse de LUIS, proporcionamos una característica en la nube que permite al cliente encargarse de la voz, permanecer en la nube, encargarse de LUIS y luego volver al cliente. Por tanto, incluso en este escenario, el SDK de voz va al contenedor en la nube de voz con el audio y, a continuación, este contenedor se comunica con el contenedor en la nube de LUIS con el texto. El contenedor de LUIS no tiene el concepto de aceptar audio (no tendría sentido que el contenedor LUIS aceptase el audio de streaming, ya que LUIS es un servicio basado en texto). En el entorno local, no tenemos la certeza de que nuestro cliente haya implementado ambos contenedores, no presuponemos la orquestación entre contenedores en las instalaciones de nuestros clientes y, si ambos contenedores se implementan de forma local, dado que son más locales para el cliente, no es una carga para volver al SR primero, volver al cliente y hacer que el cliente tome ese texto y vaya a LUIS.

<br>
</details>

<details>
<summary>
<b>¿Por qué obtenemos errores con macOS, el contenedor de voz y el SDK de Python?</b>
</summary>

Cuando se envía un archivo *.wav* para su transcripción, el resultado se devuelve con:

```cmd
recognition is running....
Speech Recognition canceled: CancellationReason.Error
Error details: Timeout: no recognition result received.
When creating a websocket connection from the browser a test, we get:
wb = new WebSocket("ws://localhost:5000/speech/recognition/dictation/cognitiveservices/v1")
WebSocket
{
    url: "ws://localhost:5000/speech/recognition/dictation/cognitiveservices/v1",
    readyState: 0,
    bufferedAmount: 0,
    onopen: null,
    onerror: null,
    ...
}
```

Sabemos que el websocket está configurado correctamente.

**Respuesta:** En ese caso, consulte [este problema de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/310). Hay una solución alternativa [propuesta aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/310#issuecomment-527542722).

Carbon corrigió esto en la versión 1.8.


<br>
</details>

<details>
<summary>
<b>¿Cuáles son las diferencias en los puntos de conexión del contenedor de voz?</b>
</summary>

¿Podría ayudar a realizar las siguientes métricas de pruebas, incluidas las funciones que se van a probar y cómo probar las API REST y de SDK? Sobre todo, las diferencias entre "interactivo" y "conversación", que no veo en los documentos o ejemplos existentes.

| Punto de conexión                                                | Prueba funcional                                                   | SDK | API DE REST |
|---------------------------------------------------------|-------------------------------------------------------------------|-----|----------|
| `/speech/synthesize/cognitiveservices/v1`               | Sintetizar texto (conversión de texto a voz)                                  |     | Sí      |
| `/speech/recognition/dictation/cognitiveservices/v1`    | Punto de conexión de websocket V1 de modo dictado local de Cognitive Services        | Sí | No       |
| `/speech/recognition/interactive/cognitiveservices/v1`  | Punto de conexión de websocket V1 de modo interactivo local de Cognitive Services  |     |          |
| `/speech/recognition/conversation/cognitiveservices/v1` | Punto de conexión de websocket V1 de modo conversación local de Cognitive Services |     |          |

**Respuesta:** Esta es una fusión de:
- Personas que prueban el punto de conexión de dictado en los contenedores (no estoy seguro de cómo obtuvieron esa dirección URL)
- El punto de conexión de la primera<sup> </sup> entidad es el que está en un contenedor.
- El punto de conexión de la primera<sup> </sup> entidad que devuelve mensajes speech.fragment en lugar de mensajes `speech.hypothesis` que los puntos de conexión de la tercera<sup> </sup> devuelven para el punto de conexión de dictado.
- Todos los inicios rápidos de Carbon usan `RecognizeOnce` (modo interactivo)
- Carbon que requiere una aserción para los mensajes `speech.fragment` que no se devuelvan en modo interactivo.
- Carbon que desencadena las aserciones en las versiones de lanzamiento (terminado el proceso).

La solución es cambiar al uso del reconocimiento continuo en el código o (más rápido) conectarse a los puntos de conexión interactivos o continuos del contenedor.
En el código, establezca el punto de conexión en `host:port`/speech/recognition/interactive/cognitiveservices/v1.

Para los distintos modos, consulte los modos de voz a continuación:

[!INCLUDE [speech-modes](includes/speech-modes.md)]

La corrección adecuada está disponible con el SDK 1.8, que tiene soporte técnico local (seleccionará el punto de conexión correcto, por lo que no será peor que el servicio en línea). Mientras tanto, hay un ejemplo de reconocimiento continuo, ¿por qué no lo indicamos?

https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/6805d96bf69d9e95c9137fe129bc5d81e35f6309/samples/python/console/speech_sample.py#L196

<br>
</details>

<details>
<summary>
<b>¿Qué modo debo usar para varios archivos de audio?</b>
</summary>

**Respuesta:** A continuación se muestra un artículo de [inicio rápido con Python](quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-python). Puede encontrar vínculos a los demás idiomas en el sitio de documentación.

Solo para aclarar el modo interactivo, de conversación y de dictado, se trata de una forma avanzada de especificar la forma en que el servicio tratará la solicitud de voz. Lamentablemente, en el caso de los contenedores locales, tenemos que especificar el URI completo (ya que incluye la máquina local), por lo que esta información se perdió en la abstracción. Estamos trabajando con el equipo del SDK para que esto sea más fácil de usar en el futuro.

<br>
</details>

<details>
<summary>
<b>¿Cómo se puede comparar una medida aproximada de transacciones por segundo o núcleos?</b>
</summary>

**Respuesta:** Estas son algunas de las cifras aproximadas que se pueden esperar del modelo existente (cambiará para mejor en el que se enviará para disponibilidad general):

- En el caso de los archivos, la limitación se realizará en el SDK de voz, al doble. Los primeros cinco segundos del audio no se limitan. El descodificador es capaz de trabajar aproximadamente a 3 veces el tiempo real. Para ello, el uso total de la CPU estará cerca de dos núcleos para un solo reconocimiento.
- En el caso del micrófono, será a tiempo real. El uso general debe ser de aproximadamente un núcleo para un solo reconocimiento.

Todo esto se puede comprobar desde los registros de Docker. En realidad volcamos la línea con las estadísticas de la sesión y de la frase o expresión y eso incluye los números RTF.


<br>
</details>

<details>
<summary>
<b>¿Es habitual dividir los archivos de audio en fragmentos para el uso del contenedor de voz?</b>
</summary>

Mi plan actual es elegir un archivo de audio existente y dividirlo en fragmentos de 10 segundos y enviarlos mediante el contenedor. ¿Es un escenario aceptable?  ¿Hay una manera mejor de procesar archivos de audio de mayor tamaño con el contenedor?

**Respuesta:** Solo tiene que usar el SDK de voz y darle el archivo; hará lo tiene que hacer. ¿Por qué es necesario fragmentar el archivo?


<br>
</details>

<details>
<summary>
<b>¿Cómo se pueden ejecutar varios contenedores en el mismo host?</b>
</summary>

El documento indica que exponga un puerto diferente; lo hago, pero el contenedor de LUIS sigue escuchando en el puerto 5000.

**Respuesta:** Pruebe `-p <outside_unique_port>:5000`. Por ejemplo, `-p 5001:5000`.


<br>
</details>

## <a name="technical-questions"></a>Preguntas técnicas

<details>
<summary>
<b>¿Cómo puedo conseguir que las API que no sean por lotes controlen audio de &lt;15 segundos de duración?</b>
</summary>

**Respuesta:** `RecognizeOnce()` en el modo interactivo solo procesa un máximo de 15 segundos de audio, ya que el modo está diseñado para comandos de voz en los que se espera que las expresiones sean cortas. Si usa `StartContinuousRecognition()` para el dictado o la conversación, no hay ningún límite de 15 segundos.


<br>
</details>

<details>
<summary>
<b>¿Cuáles son los recursos, la CPU y la RAM recomendados para 50 solicitudes simultáneas?</b>
</summary>

¿Cuántas solicitudes simultáneas podrán tratar cuatro núcleos y 4 GB de RAM? Si tenemos que servir, por ejemplo, 50 solicitudes simultáneas, ¿cuántos núcleos y RAM se recomiendan?

**Respuesta:** En tiempo real, ocho con el `en-US` más reciente, por lo que se recomienda usar más contenedores de Docker a partir de seis solicitudes simultáneas. Se vuelve loco a partir de 16 núcleos y se convierte en un nodo sensible de acceso a la memoria no uniforme (NUMA). En la tabla siguiente se describe la asignación mínima y recomendada de recursos para cada contenedor de Voz.

# <a name="speech-to-text"></a>[Voz a texto](#tab/stt)

| Contenedor      | Mínima             | Recomendado         |
|----------------|---------------------|---------------------|
| Voz a texto | 2 núcleos, 2 GB de memoria | 4 núcleos, 4 GB de memoria |

# <a name="custom-speech-to-text"></a>[Conversión de voz a texto personalizada](#tab/cstt)

| Contenedor             | Mínima             | Recomendado         |
|-----------------------|---------------------|---------------------|
| Conversión de voz a texto personalizada | 2 núcleos, 2 GB de memoria | 4 núcleos, 4 GB de memoria |

# <a name="text-to-speech"></a>[Texto a voz](#tab/tts)

| Contenedor      | Mínima             | Recomendado         |
|----------------|---------------------|---------------------|
| Texto a voz | 1 núcleo, 2 GB de memoria | 2 núcleo, 3 GB de memoria |

# <a name="custom-text-to-speech"></a>[Conversión de texto a voz personalizada](#tab/ctts)

| Contenedor             | Mínima             | Recomendado         |
|-----------------------|---------------------|---------------------|
| Conversión de texto a voz personalizada | 1 núcleo, 2 GB de memoria | 2 núcleo, 3 GB de memoria |

***

- Cada núcleo debe ser de 2,6 GHz como mínimo.
- Para los archivos, la limitación se encontrará en el SDK de voz, al doble (los primeros cinco segundos del audio no están limitados).
- El descodificador es capaz de trabajar aproximadamente a 2 o 3 veces el tiempo real. Para ello, el uso total de la CPU estará cerca de dos núcleos para un solo reconocimiento. Por eso no se recomienda mantener más de dos conexiones activas, por instancia de contenedor. Lo opuesto sería poner unos diez descodificadores al doble del tiempo real en un equipo de ocho núcleos como `DS13_V2`. Para la versión del contenedor 1.3 y posteriores, hay un parámetro que podría intentar establecer `DECODER_MAX_COUNT=20`.
- En el caso de un micrófono, será a tiempo real. El uso general debe ser aproximadamente un núcleo para un único reconocimiento.

Tenga en cuenta el número total de horas de audio que tiene. Si el número es grande, para mejorar la confiabilidad y la disponibilidad, sugerimos que se ejecuten más instancias de contenedores, ya sea en un solo cuadro o en varios cuadros, detrás de un equilibrador de carga. La orquestación se puede realizar mediante Kubernetes (K8S) y Helm, o con Docker Compose.

Por ejemplo, para controlar 1000 horas/24 horas, hemos intentado configurar tres o cuatro máquinas virtuales, con diez instancias/descodificadores por máquina virtual.

<br>
</details>

<details>
<summary>
<b>¿Admite el contenedor de voz la puntuación?</b>
</summary>

**Respuesta:** En el contenedor local está disponible la capitalización (ITN). La puntuación depende del idioma y no se admite en algunos idiomas, como el idioma chino y el japonés.

*Admitimos* la puntuación implícita y básica con los contenedores existentes, pero está `off` de forma predeterminada. Esto significa que puede obtener el carácter `.` en el ejemplo, pero no el carácter `。`. Para habilitar esta lógica implícita, este es un ejemplo de cómo hacerlo en Python con nuestro SDK de voz (sería similar en otros lenguajes):

```python
speech_config.set_service_property(
    name='punctuation',
    value='implicit',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

<br>
</details>

<details>
<summary>
<b>¿Por qué obtengo errores de tipo 404 al intentar publicar datos en un contenedor de conversión de voz en texto?</b>
</summary>

A continuación se muestra un ejemplo de HTTP POST:

```http
POST /speech/recognition/conversation/cognitiveservices/v1?language=en-US&format=detailed HTTP/1.1
Accept: application/json;text/xml
Content-Type: audio/wav; codecs=audio/pcm; samplerate=16000
Transfer-Encoding: chunked
User-Agent: PostmanRuntime/7.18.0
Cache-Control: no-cache
Postman-Token: xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Host: 10.0.75.2:5000
Accept-Encoding: gzip, deflate
Content-Length: 360044
Connection: keep-alive
HTTP/1.1 404 Not Found
Date: Tue, 22 Oct 2019 15:42:56 GMT
Server: Kestrel
Content-Length: 0
```

**Respuesta:** No se admite la API REST en el contenedor de conversión de voz en texto, solo se admiten websockets mediante el SDK de voz. Consulte siempre la documentación oficial, como [Consulta del punto de conexión de predicción del contenedor](speech-container-howto.md#query-the-containers-prediction-endpoint).

<br>
</details>

<details>
<summary>
<b>Al usar el servicio de conversión de voz en texto, ¿por qué obtengo este error?</b>
</summary>

```cmd
Error in STT call for file 9136835610040002161_413008000252496:
{
    "reason": "ResultReason.Canceled",
    "error_details": "Due to service inactivity the client buffer size exceeded. Resetting the buffer. SessionId: xxxxx..."
}
```

**Respuesta:** Esto suele ocurrir cuando se alimenta el audio más rápido de lo que el contenedor de reconocimiento de voz puede asumir. Los búferes de cliente se llenan y se desencadena la cancelación. Es necesario controlar la simultaneidad y el formato RTF en el que se envía el audio.

<br>
</details>

<details>
<summary>
<b>¿Podría explicar estos errores del contenedor de conversión de texto en voz en los ejemplos C++?</b>
</summary>

**Respuesta:** Si la versión del contenedor es anterior a la 1.3, se debe usar este código:

```cpp
const auto endpoint = "http://localhost:5000/speech/synthesize/cognitiveservices/v1";
auto config = SpeechConfig::FromEndpoint(endpoint);
auto synthesizer = SpeechSynthesizer::FromConfig(config);
auto result = synthesizer->SpeakTextAsync("{{{text1}}}").get();
```

Los contenedores más antiguos no tienen el punto de conexión necesario para que Carbon funcione con la API de `FromHost`. Si los contenedores se usan para la versión 1.3, se debe usar este código:

```cpp
const auto host = "http://localhost:5000";
auto config = SpeechConfig::FromHost(host);
config->SetSpeechSynthesisVoiceName(
    "Microsoft Server Speech Text to Speech Voice (en-US, AriaRUS)");
auto synthesizer = SpeechSynthesizer::FromConfig(config);
auto result = synthesizer->SpeakTextAsync("{{{text1}}}").get();
```

Aquí tiene un ejemplo del uso de la API de `FromEndpoint`:

```cpp
const auto endpoint = "http://localhost:5000/cognitiveservices/v1";
auto config = SpeechConfig::FromEndpoint(endpoint);
config->SetSpeechSynthesisVoiceName(
    "Microsoft Server Speech Text to Speech Voice (en-US, AriaRUS)");
auto synthesizer = SpeechSynthesizer::FromConfig(config);
auto result = synthesizer->SpeakTextAsync("{{{text2}}}").get();
```

 Se llama a la función `SetSpeechSynthesisVoiceName` porque los contenedores con un motor de conversión de texto en voz actualizada requieren el nombre de la voz.

<br>
</details>

<details>
<summary>
<b>¿Cómo puedo usar la versión 1.7 del SDK de voz con un contenedor de voz?</b>
</summary>

**Respuesta:** Hay tres puntos de conexión en el contenedor de voz para distintos usos, que se definen como modos de voz. Consulte a continuación:

[!INCLUDE [speech-modes](includes/speech-modes.md)]

Se utiliza para distintos fines y de manera diferente.

[Ejemplos](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/python/console/speech_sample.py) de Python:
- Para el reconocimiento único (modo interactivo) con un punto de conexión personalizado (es decir, `SpeechConfig` con un parámetro de punto de conexión), consulte `speech_recognize_once_from_file_with_custom_endpoint_parameters()`.
- Para el reconocimiento continuo (modo de conversación) y solo modificar para usar un punto de conexión personalizado como se indicó anteriormente, consulte `speech_recognize_continuous_from_file()`.
- Para habilitar el dictado en ejemplos como el anterior (solo si realmente lo necesita), justo después de crear `speech_config`, agregue el código `speech_config.enable_dictation()`.

En C#, para habilitar el dictado, invoque la función `SpeechConfig.EnableDictation()`.

### <a name="fromendpoint-apis"></a>API de `FromEndpoint`
| Idioma | Detalles de la API |
|----------|:------------|
| C++ | <a href="https://docs.microsoft.com/en-us/cpp/cognitive-services/speech/speechconfig#fromendpoint" target="_blank">`SpeechConfig::FromEndpoint` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| C# | <a href="https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechconfig.fromendpoint?view=azure-dotnet" target="_blank">`SpeechConfig.FromEndpoint` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| Java | <a href="https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig.fromendpoint?view=azure-java-stable" target="_blank">`SpeechConfig.fromendpoint` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| Objective-C | <a href="https://docs.microsoft.com/en-us/objectivec/cognitive-services/speech/spxspeechconfiguration#initwithendpoint" target="_blank">`SPXSpeechConfiguration:initWithEndpoint;` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| Python | <a href="https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python" target="_blank">`SpeechConfig;` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| JavaScript | Actualmente no se admite, ni está planeada. |

<br>
</details>

<details>
<summary>
<b>¿Cómo puedo usar la versión 1.8 del SDK de voz con un contenedor de voz?</b>
</summary>

**Respuesta:** Hay una nueva API de `FromHost`. No reemplaza ni modifica ninguna API existente. Simplemente agrega una manera alternativa de crear una configuración de voz mediante un host personalizado.

### <a name="fromhost-apis"></a>API de `FromHost`

| Idioma | Detalles de API |
|--|:-|
| C# | <a href="https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechconfig.fromhost?view=azure-dotnet" target="_blank">`SpeechConfig.FromHost` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| C++ | <a href="https://docs.microsoft.com/en-us/cpp/cognitive-services/speech/speechconfig#fromhost" target="_blank">`SpeechConfig::FromHost` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| Java | <a href="https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig.fromhost?view=azure-java-stable" target="_blank">`SpeechConfig.fromHost` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| Objective-C | <a href="https://docs.microsoft.com/en-us/objectivec/cognitive-services/speech/spxspeechconfiguration#initwithhost" target="_blank">`SPXSpeechConfiguration:initWithHost;` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| Python | <a href="https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python" target="_blank">`SpeechConfig;` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| JavaScript | No se admite actualmente. |

> Parámetros: host (obligatorio), clave de suscripción (opcional, si puede utilizar el servicio sin ella).

El formato del host es `protocol://hostname:port` donde `:port` es opcional (consulte a continuación):
- Si el contenedor se ejecuta localmente, el nombre de host es `localhost`.
- Si el contenedor se ejecuta en un servidor remoto, utilice el nombre de host o la dirección IPv4 de dicho servidor.

Ejemplos de parámetros de host para la conversión de voz en texto:
- `ws://localhost:5000`: conexión no segura a un contenedor local mediante el puerto 5000
- `ws://some.host.com:5000`: conexión no segura a un contenedor que se ejecuta en un servidor remoto

Ejemplos de Python anteriores, pero usan el parámetro `host` en lugar de `endpoint`:

```python
speech_config = speechsdk.SpeechConfig(host="ws://localhost:5000")
```

<br>
</details>

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Contenedores de Cognitive Services](speech-container-howto.md)
