---
title: 'Introducción a Habla personalizada: Servicio de voz'
titleSuffix: Azure Cognitive Services
description: Habla personalizada es un conjunto de herramientas en línea que permiten evaluar y mejorar la precisión de la conversión de voz a texto para las aplicaciones, las herramientas y los productos. Para comenzar solo se necesita una serie de archivos de audio de prueba. Siga los vínculos que se incluyen a continuación para empezar a crear una experiencia personalizada de conversión de voz a texto.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/17/2020
ms.author: trbye
ms.openlocfilehash: b9ac473c3864b4be06c978e7d6a3555bb4550d2a
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056846"
---
# <a name="what-is-custom-speech"></a>¿Qué es Habla personalizada?

[Habla personalizada](https://aka.ms/customspeech) es un conjunto de herramientas en línea que permiten evaluar y mejorar la precisión de la conversión de voz a texto de Microsoft para las aplicaciones, herramientas y productos. Para comenzar solo se necesita una serie de archivos de audio de prueba. Siga los vínculos que se incluyen a continuación para empezar a crear una experiencia personalizada de conversión de voz a texto.

## <a name="whats-in-custom-speech"></a>¿Qué incluye Habla personalizada?

Para utilizar Habla personalizada, necesitará una cuenta de Azure y una suscripción al Servicio de voz. Cuando ya tenga la cuenta, podrá preparar los datos, entrenar y probar sus modelos, inspeccionar la calidad del reconocimiento, evaluar la precisión y, en última instancia, implementar y utilizar el modelo de conversión de voz a texto personalizado.

Este diagrama resalta las partes que componen el [portal de Habla personalizada](https://aka.ms/customspeech). Use los siguientes vínculos para obtener más información sobre cada paso.

![Se resaltan los distintos componentes que constituyen el portal de Habla personalizada.](./media/custom-speech/custom-speech-overview.png)

1. [Suscríbase y cree un proyecto](#set-up-your-azure-account): cree una cuenta de Azure y suscríbase al servicio de voz. Esta suscripción unificada proporciona acceso a la conversión de voz a texto, la conversión de texto a voz, la traducción de voz y el [portal de Habla personalizada](https://speech.microsoft.com/customspeech). A continuación, mediante la suscripción al Servicio de voz, cree su primer proyecto de Habla personalizada.

2. [Cargue datos de prueba](how-to-custom-speech-test-data.md): cargue datos de prueba (archivos de audio) para evaluar la oferta de conversión de voz a texto para sus aplicaciones, herramientas y productos.

3. [Inspeccione la calidad del reconocimiento](how-to-custom-speech-inspect-data.md): use el [portal de Habla personalizada](https://speech.microsoft.com/customspeech) para reproducir el audio cargado e inspeccionar la calidad del reconocimiento de voz de los datos de prueba. Para conocer las medidas cuantitativas, consulte [Inspección de los datos](how-to-custom-speech-inspect-data.md).

4. [Evalúe la precisión](how-to-custom-speech-evaluate-data.md): evalúe la precisión del modelo de conversión de voz a texto. El [portal de Habla personalizada](https://speech.microsoft.com/customspeech) proporcionará una *tasa de errores de palabras*, que puede utilizarse para determinar si se necesita más entrenamiento. Si está satisfecho con la precisión, puede usar directamente las API del servicio de voz. Si desea mejorar la precisión en una media relativa del 5 al 20 %, use la pestaña **Entrenamiento** del portal para cargar datos de entrenamiento adicionales, como transcripciones con etiqueta humana y texto relacionado.

5. [Mejore la precisión](how-to-custom-speech-improve-accuracy.md): elija estratégicamente datos de entrenamiento adicionales para mejorar la calidad del modelo de conversión de voz a texto según su escenario.

6. [Entrene el modelo](how-to-custom-speech-train-model.md): mejore la precisión del modelo de conversión de voz a texto uniendo transcripciones escritas (de 10 a 1000 horas) y texto relacionado (< 200 MB) a datos de prueba de audio. Estos datos ayudan a entrenar el modelo de conversión de voz a texto. Después del entrenamiento, vuelva a probar; si le satisface el resultado, ya puede implementar el modelo.

7. [Implemente el modelo](how-to-custom-speech-deploy-model.md): cree un punto de conexión personalizado para el modelo de conversión de voz a texto y úselo en sus aplicaciones, herramientas o productos.

## <a name="set-up-your-azure-account"></a>Configuración de la cuenta de Azure

Para poder usar el [portal de Habla personalizada](https://speech.microsoft.com/customspeech) y crear un modelo personalizado, se necesita una suscripción al servicio de voz. Siga estas instrucciones para crear una suscripción estándar al servicio de voz: [Creación de una suscripción a Voz](get-started.md#new-resource).

> [!NOTE]
> Asegúrese de crear suscripciones estándar (S0); no se admiten las suscripciones gratuitas (F0).

Después de crear la cuenta de Azure y la suscripción al servicio de voz, deberá iniciar sesión en el [portal de Habla personalizada](https://speech.microsoft.com/customspeech) y conectarse a su suscripción.

1. Obtenga la clave de la suscripción del servicio de voz en Azure Portal.
2. Inicie sesión en el [portal de Habla personalizada](https://aka.ms/custom-speech).
3. Seleccione la suscripción que necesita para trabajar y crear un proyecto de voz.
4. Si desea modificar la suscripción, utilice el icono **engranaje** situado en el panel de navegación superior.

## <a name="how-to-create-a-project"></a>Creación de un proyecto

El contenido, como datos, modelos, pruebas y puntos de conexión, se organiza en **proyectos** en el [portal de Habla personalizada](https://speech.microsoft.com/customspeech). Cada proyecto es específico de un dominio y un país o idioma. Por ejemplo, puede crear un proyecto para centros de llamadas que usan el inglés en Estados Unidos.

Para crear su primer proyecto, seleccione **Speech-to-text/Custom speech** (Conversión de voz a texto/Conversión de voz personalizada) y, a continuación, haga clic en **New project** (Nuevo proyecto). Siga las instrucciones del asistente para crear el proyecto. Después de crear el proyecto, verá cuatro pestañas: **Datos**, **Pruebas**, **Entrenamiento** e **Implementación**. Use los vínculos incluidos en [Pasos siguientes](#next-steps) para aprender a usar cada pestaña.

> [!IMPORTANT]
> ¡El [portal de Custom Speech](https://aka.ms/custom-speech) se ha actualizado recientemente! Si creó datos, modelos y pruebas anteriores y publicó puntos de conexión en el portal CRIS.ai o con API, debe crear un nuevo proyecto en el nuevo portal para conectarse a estas entidades antiguas.

## <a name="next-steps"></a>Pasos siguientes

* [Preparación y prueba de los datos](how-to-custom-speech-test-data.md)
* [Inspección de los datos](how-to-custom-speech-inspect-data.md)
* [Evaluación de los datos](how-to-custom-speech-evaluate-data.md)
* [Entrenamiento del modelo](how-to-custom-speech-train-model.md)
* [Implementación del modelo](how-to-custom-speech-deploy-model.md)
