---
title: Preguntas más frecuentes sobre Text to Speech
titleSuffix: Azure Cognitive Services
description: Obtenga respuestas a las preguntas más frecuentes sobre el servicio de Text to Speech.
services: cognitive-services
author: PanosPeriorellis
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: panosper
ms.openlocfilehash: f06fda777cbebd8034ac4fd9254a4bd172d011a4
ms.sourcegitcommit: d18a59b2efff67934650f6ad3a2e1fe9f8269f21
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88661510"
---
# <a name="text-to-speech-frequently-asked-questions"></a>Preguntas más frecuentes sobre Text to Speech

Si no encuentra respuestas a sus preguntas en estas P+F, [consulte otras opciones de soporte técnico](support.md).

## <a name="general"></a>General

**P: ¿Cuál es la diferencia entre un modelo de voz estándar y un modelo de voz personalizado?**

**R.** : El modelo de voz estándar (también denominado _fuente de voz_) se ha entrenado con datos de Microsoft y ya está implementado en la nube. Puede usar un modelo de voz personalizado para adaptar un modelo promedio y transferir la expresión y el timbre de acuerdo con el estilo de voz del orador, o bien entrenar un nuevo modelo completo en función de los datos de entrenamiento que preparó el usuario. Actualmente son cada vez más los clientes que quieren una voz única y distintiva para sus bots. La plataforma de compilación de voz personalizada es la opción correcta para eso.

**P: ¿Dónde empiezo si quiero usar un modelo de voz estándar?**

**R.** : Hay más de 80 modelos de voz estándar en más de 45 idiomas disponibles mediante solicitudes HTTP. Primero, obtenga una [clave suscripción](https://docs.microsoft.com/azure/cognitive-services/speech-service/get-started). Para realizar llamadas REST a los modelos de voz ya implementados, consulte la [API de REST](https://docs.microsoft.com/azure/cognitive-services/speech-service/rest-apis).

**P: Si quiero usar un modelo de voz personalizado, ¿es la API la misma que la usada para las voces estándar?**

**R.** : Cuando se crea e implementa un modelo de voz personalizado, obtiene un punto de conexión único para el modelo. Para usar la voz para hablar en las aplicaciones, deberá especificar el punto de conexión en las solicitudes HTTP. La misma funcionalidad que está disponible a través de la API de REST para el servicio Text to Speech está disponible para el punto de conexión personalizado. Aprenda a [crear y usar el punto de conexión personalizado](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-customize-voice-font#create-and-use-a-custom-voice-endpoint).

**P: ¿Es necesario preparar los datos de entrenamiento para crear modelos de voz personalizados por mi cuenta?**

**R.** : Sí, debe preparar los datos de entrenamiento por sí mismo para un modelo de voz personalizado.

Se requiere una colección de datos de voz para crear un modelo de voz personalizado. Esta colección se compone de un conjunto de archivos de audio de las grabaciones de voz y un archivo de texto de la transcripción de cada uno de los archivos de audio. El resultado de su voz digital se basa principalmente en la calidad de los datos de entrenamiento. Para generar una buena voz de texto a voz, es importante que las grabaciones se realicen en una sala silenciosa con un micrófono de pedestal de alta calidad. El volumen constante, la velocidad de la conversación, el tono al hablar e incluso la coherencia en las particularidades expresivas del habla son esenciales para compilar una gran voz digital. Se recomienda encarecidamente grabar las voces en un estudio de grabación.

Actualmente, no proporcionamos compatibilidad con la grabación en línea ni tenemos ninguna recomendación sobre estudios de grabación. Si quiere conocer el requisito de formato, consulte [cómo preparar las grabaciones y transcripciones](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-custom-voice-create-voice).

**P: ¿Qué guiones debo usar para grabar los datos de voz para el entrenamiento de voz personalizado?**

**R.** : No hay límite en cuanto a los guiones que se pueden usar para las grabaciones de voz. Puede usar sus propios guiones para grabar el discurso. Solo debe asegurarse de tener la cobertura fonética suficiente en los datos de voz. Para entrenar una voz personalizada, puede empezar con un volumen pequeño de datos de voz, que podrían ser 50 frases distintas (unos 3 a 5 minutos de habla). Cuantos más datos proporcione, más natural será su voz. Para empezar a entrenar una fuente de voz completa, puede proporcionar grabaciones de más de 2000 frases (unas 3 a 4 horas de habla). Para obtener una voz completa de alta calidad, deberá preparar grabaciones de más de 6000 frases (unas 8 a 10 horas de habla).

Proporcionamos servicios adicionales para ayudarlo a preparar guiones para la grabación. Póngase en contacto con la [asistencia al cliente de Voz personalizada](mailto:customvoice@microsoft.com?subject=Inquiries%20about%20scripts%20generation%20for%20Custom%20Voice%20creation) en caso de consultas.

**P: ¿Qué pasa si necesito más simultaneidad que el valor predeterminado o la que se ofrece en el portal?**

**R.** : Se puede escalar verticalmente el modelo en incrementos de 20 solicitudes simultáneas. Póngase en contacto con la [asistencia al cliente de Voz personalizada](mailto:customvoice@microsoft.com?subject=Inquiries%20about%20scripts%20generation%20for%20Custom%20Voice%20creation) en caso de consultas sobre un mayor escalamiento.

**P: ¿Puedo descargar mi modelo y ejecutarlo localmente?**

**R.** : Los modelos no se pueden descargar ni ejecutar localmente.

**P: ¿Están limitadas mis solicitudes?**

**R.** : Consulte [Cuotas y límites de los servicios de Voz](speech-services-quotas-and-limits.md).

## <a name="next-steps"></a>Pasos siguientes

- [Solución de problemas](troubleshooting.md)
- [Notas de la versión](releasenotes.md)
