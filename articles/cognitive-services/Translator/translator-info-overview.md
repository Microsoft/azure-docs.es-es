---
title: ¿Qué es Traductor? Translator
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
ms.openlocfilehash: 206e1bc86404fa41b84647c50482dbe63301c0af
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89425966"
---
# <a name="what-is-translator"></a>¿Qué es Traductor?

Azure Cognitive Services Translator es un servicio de traducción automática basado en la nube que forma parte de la familia de API de [Azure Cognitive Services](https://docs.microsoft.com/azure/?pivot=products&panel=ai) para crear aplicaciones inteligentes. Translator se integra con facilidad en cualquier aplicación, sitio web, herramienta y solución. Permite agregar experiencias de usuario multilingüe en [más de 70 idiomas](languages.md) y se puede usar en cualquier plataforma de hardware con cualquier sistema operativo para la traducción de texto a texto.

## <a name="about-microsoft-translator"></a>Acerca de Microsoft Translator

Azure Cognitive Services Translator es un servicio de traducción automática basado en la nube. Translator da servicio a diversos productos y servicios de Microsoft, y lo utilizan miles de empresas en todo el mundo en sus aplicaciones y flujos de trabajo para que su contenido llegue a una audiencia mundial.

La traducción de voz, con la tecnología de Translator, también está disponible mediante el [servicio de voz de Microsoft](https://docs.microsoft.com/azure/cognitive-services/speech-service/). Combina la funcionalidad de Translator Speech API y Custom Speech Service en un servicio totalmente personalizable y unificado. 

## <a name="language-support"></a>Compatibilidad con idiomas

Microsoft Translator proporciona compatibilidad con varios idiomas de traducción, transliteración, detección de idioma y diccionarios. Consulte [Compatibilidad con idiomas](language-support.md) para obtener una lista completa o puede acceder a la lista mediante programación con la [API REST](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-languages).  

## <a name="microsoft-translator-neural-machine-translation"></a>Traducción automática neuronal de Microsoft Translator

La traducción automática neuronal (NMT) es el nuevo estándar para las traducciones automáticas de alta calidad basadas en inteligencia artificial. Este estándar reemplaza la traducción automática estadística (SMT) que alcanzó un nivel estable a mediados de 2010.

NMT proporciona mejores traducciones que SMT, no solo a la hora de puntuar la calidad de la traducción bruta, sino también porque suenan más fluidas y humanas. El motivo principal de esta fluidez es que NMT usa el contexto completo de una frase para traducir las palabras. SMT solo tomaba el contexto inmediato de unas cuantas palabras antes y después de cada palabra.

Los modelos NMT se sitúan en el centro de la API y no son visibles para los usuarios finales. La única diferencia perceptible es una mejor calidad de la traducción, en especial en los idiomas chino, japonés y árabe.

Más información sobre [el funcionamiento de NMT](https://www.microsoft.com/en-us/translator/mt.aspx#nnt)

## <a name="language-customization"></a>Personalización de lenguaje

Custom Translator es una extensión del servicio Translator que se puede usar en combinación con Translator como ayuda para personalizar el sistema de traducción neuronal y mejorar la traducción para su terminología y estilo específicos.

Con Custom Translator puede crear sistemas de traducción que administran la terminología usada en su propio negocio o sector. Luego, los sistemas de traducción personalizados se pueden integrar fácilmente en las aplicaciones, flujos de trabajo y sitios web ya existentes, en varios tipos de dispositivos, mediante el componente normal Translator, por medio del parámetro de categoría.

Más información sobre la [personalización de idioma](customization.md)

## <a name="next-steps"></a>Pasos siguientes

- [Regístrese](translator-text-how-to-signup.md) para obtener una clave de acceso.
- [Referencia de API](https://docs.microsoft.com/azure/cognitive-services/Translator/reference/v3-0-reference) proporciona la documentación técnica de las API.
- [Detalles de precios](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/)
