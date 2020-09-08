---
title: '¿Qué es Translator?: Translator'
titlesuffix: Azure Cognitive Services
description: Integre Translator en aplicaciones, sitios web, herramientas u otras soluciones para proporcionar experiencias de usuario en varios idiomas.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: overview
ms.date: 05/26/2020
ms.author: swmachan
ms.custom: seodec18
ms.openlocfilehash: 5da9e5b38b88a5a148bd3d85664daa523153086b
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88589459"
---
# <a name="what-is-the-translator"></a>¿Qué es Translator?

Translator es fácil de integrar en sus aplicaciones, sitios web, herramientas y soluciones. Permite agregar experiencias de usuario multilingüe en [más de 70 idiomas](languages.md) y se puede usar en cualquier plataforma de hardware con cualquier sistema operativo para la traducción de texto a texto.

Translator forma parte de la colección de algoritmos de aprendizaje automático y de inteligencia artificial en la nube, de [Azure Cognitive Services](https://docs.microsoft.com/azure/?pivot=products&panel=ai), que se pueden consumir fácilmente en los proyectos de desarrollo.

## <a name="about-microsoft-translator"></a>Acerca de Microsoft Translator

Translator es un servicio de traducción automática basado en la nube. En el centro de este servicio está Translator, que da servicio a diversos productos y servicios de Microsoft, y que utilizan miles de empresas en todo el mundo en sus aplicaciones y flujos de trabajo para que su contenido llegue a una audiencia mundial.

La traducción de voz, con la tecnología de Translator, también está disponible mediante el [servicio de Voz de Microsoft](https://docs.microsoft.com/azure/cognitive-services/speech-service/). Combina la funcionalidad de Translator Speech API y Custom Speech Service en un servicio totalmente personalizable y unificado. Speech Service reemplaza a Translator Speech API, que se retirará el 15 de octubre de 2019.

## <a name="language-support"></a>Compatibilidad con idiomas

Microsoft Translator proporciona compatibilidad con varios idiomas de traducción, transliteración, detección de idioma y diccionarios. Consulte [Compatibilidad con idiomas](language-support.md) para obtener una lista completa o puede acceder a la lista mediante programación con la [API REST](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-languages).  

## <a name="microsoft-translator-neural-machine-translation"></a>Traducción automática neuronal de Microsoft Translator

La traducción automática neuronal (NMT) es el nuevo estándar para las traducciones automáticas de alta calidad basadas en inteligencia artificial. Este estándar reemplaza la traducción automática estadística (SMT) que alcanzó un nivel estable a mediados de 2010.

NMT proporciona mejores traducciones que SMT, no solo a la hora de puntuar la calidad de la traducción bruta, sino también porque suenan más fluidas y humanas. El motivo principal de esta fluidez es que NMT usa el contexto completo de una frase para traducir las palabras. SMT solo tomaba el contexto inmediato de unas cuantas palabras antes y después de cada palabra.

Los modelos NMT se sitúan en el centro de la API y no son visibles para los usuarios finales. La única diferencia perceptible es una mejor calidad de la traducción, en especial en los idiomas chino, japonés y árabe.

Más información sobre [el funcionamiento de NMT](https://www.microsoft.com/en-us/translator/mt.aspx#nnt)

## <a name="language-customization"></a>Personalización de lenguaje

Traductor personalizado, una extensión del servicio Microsoft Translator principal, se puede usar en combinación con Translator para ayudar a personalizar el sistema de traducción neuronal y mejorar la traducción para su terminología y estilo específicos.

Con Custom Translator puede crear sistemas de traducción que administran la terminología usada en su propio negocio o sector. Luego, los sistemas de traducción personalizados se pueden integrar fácilmente en las aplicaciones, flujos de trabajo y sitios web ya existentes, en varios tipos de dispositivos, mediante el componente normal Translator, por medio del parámetro de categoría.

Más información sobre la [personalización de idioma](customization.md)

## <a name="next-steps"></a>Pasos siguientes

- [Regístrese](translator-text-how-to-signup.md) para obtener una clave de acceso.
- [Referencia de API](https://docs.microsoft.com/azure/cognitive-services/Translator/reference/v3-0-reference) proporciona la documentación técnica de las API.
- [Detalles de precios](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/)
