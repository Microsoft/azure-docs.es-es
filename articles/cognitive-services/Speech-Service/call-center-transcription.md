---
title: 'Transcripción para los centros de llamadas: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Un escenario común para la conversión de voz a texto es la transcripción de grandes volúmenes de datos de telefonía que provienen de varios sistemas, como la respuesta de voz interactiva (IVR). Con el servicio Voz y el modelo de voz unificado, una empresa puede obtener transcripciones de alta calidad con sistemas de captura de audio.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: erhopf
ms.openlocfilehash: 84a1e2edcbe9e9da93a158ebc11e7bf638dae100
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89462741"
---
# <a name="speech-service-for-telephony-data"></a>Servicio Voz para datos de telefonía

Los datos de telefonía que se generan a través de líneas fijas, teléfonos móviles y radios son típicamente de baja calidad, y de banda estrecha en el rango de 8 KHz, lo que crea desafíos al convertir la voz en texto. Los últimos modelos de reconocimiento de voz del servicio Voz destacan en la transcripción de estos datos de telefonía, incluso en los casos en que los datos son difíciles de entender para un humano. Estos modelos están entrenados con grandes volúmenes de datos de telefonía y tienen la mejor precisión de reconocimiento del mercado, incluso en entornos ruidosos.

Un escenario común para la conversión de voz a texto es la transcripción de grandes volúmenes de datos de telefonía que pueden provenir de varios sistemas, como la respuesta de voz interactiva (IVR). El audio que proporcionan estos sistemas puede ser estéreo o mono, y sin procesar con poco o nada de procesamiento posterior realizando en la señal. Con el servicio Voz y el modelo de voz unificado, una empresa puede obtener transcripciones de alta calidad, sea cual sea el sistema de captura de audio.

Los datos de telefonía se pueden utilizar para comprender mejor las necesidades de sus clientes, identificar nuevas oportunidades de marketing o evaluar el rendimiento de los agentes del centro de llamadas. Después de que se transcriben los datos, la empresa puede utilizar la salida para mejorar la telemetría, identificar frases clave o analizar la opción del cliente.

Las tecnologías descritas en esta página son de Microsoft internamente para varios servicios de procesamiento de llamadas de soporte, tanto en tiempo real como por lotes.

Revisemos algunas de las tecnologías y características relacionadas que ofrece el servicio Voz.

> [!IMPORTANT]
> El modelo unificado del servicios Voz está entrenado con diversos datos y ofrece una solución de modelo único para una serie de escenarios que van desde el dictado hasta el análisis de la telefonía.

## <a name="azure-technology-for-call-centers"></a>Tecnología de Azure para centros de llamada

Más allá del aspecto funcional del servicio Voz, su objetivo principal, cuando se aplica al centro de llamadas, es mejorar la experiencia del cliente. A este respecto, existen tres ámbitos claros:

- Análisis posterior a la llamada, que esencialmente es el procesamiento por lotes de las grabaciones de llamadas después de la llamada.
- Análisis en tiempo real, que es el procesamiento de la señal de audio para extraer varias informaciones a medida que se realiza la llamada (siendo la opinión un caso de uso principal).
- Asistentes de voz (bots), ya sea conduciendo el diálogo entre el cliente y el bot en un intento de resolver el problema del cliente sin la participación del agente, o como aplicación de protocolos de inteligencia artificial (IA) para ayudar al agente.

Un diagrama de arquitectura típico de la implementación de un escenario por lotes se muestra en la siguiente ilustración ![Arquitectura de transcripción para el centro de llamadas](media/scenarios/call-center-transcription-architecture.png).

## <a name="speech-analytics-technology-components"></a>Componentes de la tecnología de análisis de voz

Ya sea que el dominio sea posterior a la llamada o en tiempo real, Azure ofrece un conjunto de tecnologías maduras y emergentes para mejorar la experiencia del cliente.

### <a name="speech-to-text-stt"></a>Conversión de voz en texto

La características de [conversión de voz en texto](speech-to-text.md) es la más buscada en cualquier solución de centro de llamadas. Como muchos de los procesos analíticos posteriores se basan en texto transcrito, la tasa de errores por palabra (_WER_) es de suma importancia. Uno de los principales desafíos en la transcripción para los centros de llamadas es el ruido que prevalece en dichos centros (por ejemplo, otros agentes que hablan en segundo plano), la rica variedad de idiomas locales y dialectos, así como la baja calidad de la señal telefónica real. WER está muy correlacionado con el nivel de formación de los modelos acústicos y de lenguaje para una determinada configuración regional, por lo que es importante poder personalizar el modelo a la suya. Nuestros modelos de la versión 4.x unificada más recientes son la solución perfecta tanto para la precisión de transcripción como para la latencia. Entrenados con decenas de miles de horas de datos acústicos y miles de millones de información léxica, los modelos unificados son los más precisos del mercado para transcribir los datos del centro de llamadas.

### <a name="sentiment"></a>Opinión

Evaluar si el cliente tuvo una buena experiencia es una de las áreas más importantes del análisis de voz cuando se aplica al espacio del centro de llamadas. Nuestra [API Batch Transcription](batch-transcription.md) ofrece análisis de las opiniones por expresión. Puede agregar el conjunto de valores obtenidos como parte de una transcripción de la llamada para determinar la opinión de la llamada tanto para sus agentes como para el cliente.

### <a name="silence-non-talk"></a>Silencio (sin hablar)

No es raro que el 35 por ciento de una llamada de soporte técnico sea lo que llamamos tiempo de silencios. Algunos de los escenarios en los que no se habla son: agentes que consultan el historial de casos anteriores con un cliente, agentes que utilizan herramientas que les permiten acceder al escritorio del cliente y realizar funciones o clientes que están en espera de una transferencia, entre otros. Es extremadamente importante evaluar cuándo se está produciendo el silencio en una llamada, ya que hay una serie de importantes sensibilidades de los clientes que se producen en torno a este tipo de escenarios y dónde se producen en la llamada.

### <a name="translation"></a>Traducción

Algunas empresas experimentan ahora con ofrecer transcripciones traducidas de llamadas de soporte técnico en idiomas extranjeros para que los administradores de entrega puedan comprender la experiencia de sus clientes de todo el mundo. Nuestras funcionalidades de [traducción](/azure/cognitive-services/speech-service/speech-translation) son insuperables. Podemos traducir audio a audio o audio a texto para un gran número de configuraciones regionales.

### <a name="text-to-speech"></a>Texto a voz

[Texto a voz](text-to-speech.md) es otra área importante en la implementación de bots que interactúan con los clientes. La vía típica es que el cliente habla, su voz se transcribe en texto, el texto se analiza en busca de intenciones, se sintetiza una respuesta basada en la intención reconocida y, a continuación, se muestra un recurso para el cliente o se genera una respuesta de voz sintetizada. Por supuesto, todo esto tiene que ocurrir rápidamente, por lo que la baja latencia es un componente importante para el éxito de estos sistemas.

Nuestra latencia de un extremo a otro es considerablemente baja si se tienen en cuenta las diversas tecnologías implicadas, tales como [Conversión de voz en texto](speech-to-text.md), [LUIS](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/), [Bot Framework](https://dev.botframework.com/) y [Texto a voz](text-to-speech.md).

Nuestras nuevas voces son también indistinguibles de las voces humanas. Puede usar nuestras voces para darle al bot su personalidad única.

### <a name="search"></a>Search

Otro elemento básico del análisis es identificar las interacciones en las que ha ocurrido un evento o experiencia específica. Esto se hace típicamente con uno de dos enfoques, ya sea una búsqueda ad hoc donde el usuario simplemente escribe una frase y el sistema responde, o una consulta más estructurada, donde un analista puede crear un conjunto de sentencias lógicas que identifican un escenario en una llamada, y luego cada llamada puede ser indexada contra ese conjunto de consultas. Un buen ejemplo de búsqueda es la omnipresente declaración de conformidad "Esta llamada se grabará con fines de calidad...". Muchas empresas quieren asegurarse de que sus agentes están proporcionando esta declinación de responsabilidades a los clientes antes de que se grabe la llamada. La mayoría de los sistemas de análisis muestran la tendencia de los comportamientos encontrados por los algoritmos de consulta o búsqueda, ya que este informe de tendencias es en última instancia una de las funciones más importantes de un sistema de análisis. Mediante el [directorio de Cognitive Services](https://azure.microsoft.com/services/cognitive-services/directory/search/), tu solución integral puede mejorar significativamente con funcionalidades de indexación y búsqueda.

### <a name="key-phrase-extraction"></a>Extracción de frases clave

Esta área es una de las aplicaciones de análisis más desafiantes y una que se está beneficiando de la aplicación de la inteligencia artificial y del aprendizaje automático. El escenario principal en este caso es inferir la intención del cliente. ¿Por qué llama el cliente? ¿Cuál es el problema del cliente? ¿Por qué el cliente ha tenido una experiencia negativa? Nuestro [servicio de análisis de texto](https://azure.microsoft.com/services/cognitive-services/text-analytics/) proporciona un conjunto de análisis de fábrica para actualizar rápidamente su solución completa y extraer esas palabras clave o frases importantes.

Veamos ahora con más detalle el procesamiento por lotes y las canalizaciones en tiempo real para el reconocimiento de voz.

## <a name="batch-transcription-of-call-center-data"></a>Transcripción en lote de los datos del centro de llamadas

Para transcribir la mayor parte del audio, se ha desarrollado la [API Batch Transcription](batch-transcription.md). La API Batch Transcription se desarrolló para transcribir grandes cantidades de datos de audio de forma asincrónica. En cuanto a la transcripción de datos del centro de llamadas, nuestra solución se basa en estos pilares:

- **Precisión**: con modelos unificados de cuarta generación, ofrecemos una calidad de transcripción insuperable.
- **Latencia**: entendemos que cuando se realizan transcripciones masivas, las transcripciones se necesitan rápidamente. Los trabajos de transcripción iniciados a través de la [API Batch Transcription](batch-transcription.md) se pondrán en cola inmediatamente y, una vez que el trabajo empiece a ejecutarse, se realizará más rápidamente que la transcripción en tiempo real.
- **Seguridad**: entendemos que las llamadas pueden contener información confidencial. Puede estar tranquilo porque la seguridad es una de nuestras mayores prioridades. Nuestro servicio ha obtenido las certificaciones ISO, SOC, HIPAA y PCI.

Los centros de llamadas generan grandes volúmenes de datos de audio diariamente. Si su empresa almacena datos de telefonía en una ubicación central, como Azure Storage, puede utilizar la [API Batch Transcription](batch-transcription.md) para solicitar y recibir transcripciones de forma asincrónica.

Una solución típica utiliza estos servicios:

- El servicio Voz se usan para convertir voz en texto. Se requiere una suscripción estándar (S0) al servicio Voz para usar la API Batch Transcription. Las suscripciones gratuitas (F0) no funcionarán.
- [Azure Storage](https://azure.microsoft.com/services/storage/) se utiliza para almacenar datos de telefonía y las transcripciones devueltas por la API Batch Transcription. Esta cuenta de almacenamiento debe utilizar notificaciones, específicamente para cuando se agregan nuevos archivos. Estas notificaciones se utilizan para desencadenar el proceso de transcripción.
- La solución [Azure Functions](https://docs.microsoft.com/azure/azure-functions/) se utiliza para crear un identificador URI de firmas de acceso compartido (SAS) para cada grabación, y desencadenar la petición HTTP POST para iniciar una transcripción. Además, Azure Functions se utiliza para crear solicitudes de recuperación y eliminación de transcripciones mediante la API Batch Transcription.

Internamente estamos utilizando las tecnologías anteriores para admitir las llamadas de los clientes de Microsoft en modo por lotes.
:::image type="content" source="media/scenarios/call-center-batch-pipeline.png" alt-text="Tecnologías utilizadas para admitir llamadas de clientes de Microsoft en el modo por lotes.":::

## <a name="real-time-transcription-for-call-center-data"></a>Transcripción en tiempo real para los datos del centro de llamadas

Algunas empresas deben transcribir las conversaciones en tiempo real. Se puede utilizar la transcripción en tiempo real para identificar palabras clave y desencadenar búsquedas de contenido y recursos relevantes para la conversación, para supervisar la opinión, para mejorar la accesibilidad o para proporcionar traducciones a los clientes y agentes que no son hablantes nativos.

Para escenarios que requieren transcripción en tiempo real, recomendamos usar [Speech SDK](speech-sdk.md). Actualmente, la conversión de voz a texto está disponible en [más de 20 idiomas](language-support.md) y el SDK está disponible en C++, C#, Java, Python, Node.js, Objective-C y JavaScript. En [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk) se pueden encontrar ejemplos en todos los idiomas. Para las últimas noticias y actualizaciones, consulte las [Notas de la versión](releasenotes.md).

Internamente estamos utilizando las tecnologías anteriores para analizar en tiempo real las llamadas de los clientes de Microsoft cuando se producen, como se muestra en el diagrama siguiente.

![Arquitectura por lotes](media/scenarios/call-center-reatime-pipeline.png)

## <a name="a-word-on-ivrs"></a>Una palabra sobre las IVR

El servicio Voz se puede integrar fácilmente en cualquier solución mediante el uso del [SDK de Voz](speech-sdk.md) o la [API REST](rest-apis.md). Sin embargo, la transcripción para el centro de llamadas puede requerir tecnologías adicionales. Normalmente, se requiere una conexión entre un sistema IVR y Azure. Aunque no ofrecemos tales componentes, esta es una descripción de lo que implica una conexión a un sistema IVR.

Varios productos de servicios de IVR o telefonía (como Genesys o AudioCodes) ofrecen funcionalidades de integración que pueden aprovecharse para permitir el paso de audio entrante y saliente a un servicio de Azure. Básicamente, un servicio personalizado de Azure puede proporcionar una interfaz específica para definir sesiones de llamadas telefónicas (tales como el inicio de llamada o el fin de llamada) y exponer una API de WebSocket para recibir el flujo de audio entrante que se utiliza con el servicio Voz. Las respuestas salientes, como la transcripción de conversaciones o las conexiones con Bot Framework, pueden sintetizarse con el servicio de texto a voz de Microsoft y devolverse al sistema IVR para su reproducción.

Otro escenario es la integración directa con el protocolo de inicio de sesión (SIP). Un servicio de Azure se conecta a un servidor SIP y obtiene así un flujo de entrada y un flujo de salida, que se utiliza para las fases de conversión de voz a texto y de texto a voz. Para conectarse a un servidor SIP, hay ofertas de software comercial, como Ozeki SDK, o [the Teams calling and meetings API](/graph/api/resources/communications-api-overview) (actualmente en versión beta), que están diseñados para admitir este tipo de escenario para llamadas de audio.

## <a name="customize-existing-experiences"></a>Personalización de la experiencias existentes

 El servicio Voz funciona bien con los modelos integrados. Sin embargo, es posible que desee personalizar y optimizar más la experiencia para su producto o entorno. Las opciones de personalización abarcan desde la optimización de modelos acústicos a fuentes de voz únicas para su marca. Una vez que haya creado un modelo personalizado, podrá usarlo con cualquiera de las características del servicio Voz en tiempo real o en modo por lotes.

| Servicio de voz | Modelo | Descripción |
| -------------- | ----- | ----------- |
| Voz a texto | [Modelo acústico](how-to-customize-acoustic-models.md) | Cree un modelo acústico personalizado para las aplicaciones, herramientas o dispositivos usados en entornos concretos como en un automóvil o en una planta de producción, cada uno con unas condiciones de grabación específicas. Los ejemplos incluyen el habla con acento, ruidos de fondo específicos o el uso de un micrófono específico para la grabación. |
|                | [Modelo de lenguaje](how-to-customize-language-model.md) | Cree un modelo de lenguaje personalizado para mejorar la transcripción de gramática y vocabulario específicos del sector, como terminología médica o jerga de TI. |
|                | [Modelo de pronunciación](how-to-customize-pronunciation.md) | Con un modelo de pronunciación personalizado, puede definir el formato fonético y mostrar una palabra o un término. Es útil para controlar términos personalizados, como nombres de producto o acrónimos. Basta con un archivo de pronunciación, que es un archivo `.txt` simple. |
| Texto a voz | [Fuente de voz](how-to-customize-voice-font.md) | Las fuentes de voz personalizadas le permiten crear una voz única y reconocible para su marca. Solo toma una pequeña cantidad de datos para empezar a trabajar. Cuantos más datos proporcione, más natural y similar a la humana sonará su fuente de voz. |

## <a name="sample-code"></a>Código de ejemplo

El código de ejemplo está disponible en GitHub para cada una de las características del servicio Voz. En estos ejemplos se tratan escenarios comunes como la lectura de audio de un archivo o flujo, el reconocimiento continuo y de una sola emisión, y el trabajo con modelos personalizados. Use estos vínculos para ver ejemplos de SDK y REST:

- [Ejemplos de traducción de voz y voz a texto (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
- [Ejemplos de transcripción de Azure Batch (REST)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/batch)
- [Ejemplos de texto a voz (REST)](https://github.com/Azure-Samples/Cognitive-Speech-TTS)

## <a name="reference-docs"></a>Documentos de referencia

- [Acerca del SDK de Voz](speech-sdk-reference.md)
- [Speech Devices SDK](speech-devices-sdk.md)
- [API REST: Speech-to-text](rest-speech-to-text.md) (API de REST: Voz a texto)
- [API REST: Text-to-speech](rest-text-to-speech.md) (API de REST: Texto a voz)
- [API REST: Batch transcription and customization](https://westus.cris.ai/swagger/ui/index) (API de REST: Transcripción y personalización de Azure Batch)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtenga una clave de suscripción gratuita a los servicios de Voz](get-started.md)
