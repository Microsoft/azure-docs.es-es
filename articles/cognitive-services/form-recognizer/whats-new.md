---
title: Novedades en Form Recognizer
titleSuffix: Azure Cognitive Services
description: Comprenda los últimos cambios en la API de Form Recognizer.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: pafarley
ms.openlocfilehash: ddd1f61ada539ebb00341dd83919f1c851a0f3e1
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816378"
---
# <a name="whats-new-in-form-recognizer"></a>Novedades en Form Recognizer

El servicio Form Recognizer se actualiza de forma continuada. Use este artículo para mantenerse al día con mejoras de características, correcciones y actualizaciones de la documentación.

## <a name="august-2020"></a>Agosto de 2020

### <a name="new-features"></a>Nuevas características

**Ya está disponible la versión preliminar pública de Form Recognizer v2.1.** Se ha publicado V2.1-preview.1, incluidas las siguientes características: 


- **Está disponible la referencia de la API de REST**: vea la [referencia de v2.1-preview.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-1/operations/AnalyzeBusinessCardAsync) 
- **Se admiten nuevos idiomas además del inglés**: ahora se admiten los siguientes [idiomas](language-support.md) para `Layout` y `Train Custom Model`: inglés (`en`), chino (simplificado) (`zh-Hans`), neerlandés (`nl`), francés (`fr`), alemán (`de`), italiano (`it`), portugués (`pt`) y español (`es`).
- **Detección de marcas de selección o casillas**: Form Recognizer admite la detección y extracción de marcas de selección, como casillas y botones de radio. Las marcas de selección se extraen de `Layout` y ahora también es posible etiquetar y entrenar en `Train Custom Model` - _Entrenamiento con etiquetas_ para extraer pares clave-valor de marcas de selección. 
- **Composición de modelos** permite que se compongan varios modelos y que se llamen con un único identificador de modelo. Cuando se envía un documento para que se analice con un identificador de modelo compuesto, primero se realiza un paso de clasificación para enrutarlo al modelo personalizado correcto. La composición de modelos está disponible para `Train Custom Model` - _Entrenamiento con etiquetas_.
- **Nombre de modelo**: agregue un nombre descriptivo a los modelos personalizados para facilitar su administración y seguimiento.
- **[Nuevo modelo precompilado para tarjetas de presentación](concept-business-cards.md)** para extraer campos comunes de tarjetas de presentación (en inglés).
- **[Nuevas configuraciones regionales para recibos precompilados](concept-receipts.md)** : además de EN-US, ahora se admiten EN-AU, EN-CA, EN-GB y EN-IN.
- **Mejoras de calidad** para `Layout`, `Train Custom Model` - _Entrenamiento sin etiquetas_ y _Entrenamiento con etiquetas_.


La **versión 2.0** incluye la siguiente actualización:
-   Las [bibliotecas cliente](quickstarts/client-library.md) para NET, Python, Java y JavaScript ahora tienen disponibilidad general. 


Hay **nuevos ejemplos** disponibles en GitHub. 
- En el [cuaderno de estrategias de Forms: recetas de extracción de conocimientos](https://github.com/microsoft/knowledge-extraction-recipes-forms) se recopilan procedimientos recomendados de interacciones de clientes reales de Form Recognizer y se proporcionan ejemplos de código útiles, listas de comprobación y canalizaciones de ejemplo que se usan en el desarrollo de estos proyectos. 
- La [herramienta de etiquetado de ejemplo](https://github.com/microsoft/OCR-Form-Tools) se ha actualizado para admitir la nueva funcionalidad de v2.1. Vea este [inicio rápido](quickstarts/label-tool.md) para empezar a trabajar con la herramienta. 
- El ejemplo de Form Recognizer [Intelligent Kiosk](https://github.com/microsoft/Cognitive-Samples-IntelligentKiosk/blob/master/Documentation/FormRecognizer.md) muestra cómo integrar `Analyze Receipt` y `Train Custom Model` - _Entrenamiento sin etiquetas_.



## <a name="july-2020"></a>Julio de 2020

### <a name="new-features"></a>Nuevas características

* **Referencia de v2.0 disponible** Consulte la [Referencia de la API v2.0](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm) y los SDK actualizados para [.NET](https://docs.microsoft.com/dotnet/api/overview/azure/ai.formrecognizer-readme-pre?view=azure-dotnet), [Python](https://docs.microsoft.com/python/api/overview/azure/?view=azure-python), [Java](https://docs.microsoft.com/java/api/overview/azure/ai-formrecognizer-readme-pre?view=azure-java-preview) y [JavaScript](https://docs.microsoft.com/javascript/api/overview/azure/?view=azure-node-latest).
* Las **mejoras en las tablas y extracciones** incluyen mejoras de precisión y de extracciones de tablas, en concreto, la posibilidad de conocer los encabezados y las estructuras de las tablas en el _entrenamiento personalizado sin etiquetas_. 

* **La compatibilidad con divisas** permite detectar y extraer símbolos de divisas globales.
* **En Azure Gov** también está disponible Form Recognizer.
* **Características de seguridad mejoradas**: 
   * **Traiga su propia clave**: Form Recognizer cifra automáticamente los datos cuando se guardan en la nube para protegerlos y para ayudarle a satisfacer los compromisos de cumplimiento y de seguridad de la organización. De forma predeterminada, su suscripción usa claves de cifrado administradas por Microsoft. Ahora también puede administrar la suscripción con sus propias claves de cifrado. [Las claves administradas por el cliente (CMK), también se conoce como Bring Your Own Key (BYOK)](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/form-recognizer-encryption-of-data-at-rest), ofrecen más flexibilidad para crear, rotar, deshabilitar y revocar controles de acceso. También permite auditar las claves de cifrado que se usan para proteger los datos.  
   * **Los puntos de conexión privados** le permiten [acceder de forma segura a los datos a través de Private Link](https://docs.microsoft.com/azure/private-link/private-link-overview) en una red virtual (VNet).


## <a name="june-2020"></a>Junio de 2020

### <a name="new-features"></a>Nuevas características
* **Se ha agregado la CopyModel API a los SDK de cliente**: ahora puede usar los SDK de cliente para copiar modelos de una suscripción a otra. Consulte [Copia de seguridad y recuperación de modelos](./disaster-recovery.md) para obtener información general sobre esta característica.
* **Integración de Azure Active Directory**: ahora puede usar las credenciales de Azure AD para autenticar los objetos de cliente de Form Recognizer en los SDK.
* **Cambios específicos de cada SDK**: esto incluye la adición de características secundarias y cambios importantes. Consulte el registro de cambios del SDK para obtener más información.
  * [Registro de cambios del SDK para C# versión preliminar 3](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/formrecognizer/Azure.AI.FormRecognizer/CHANGELOG.md)
  * [Registro de cambios del SDK para Python versión preliminar 3](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/CHANGELOG.md)
  * [Registro de cambios del SDK para Java versión preliminar 3](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/CHANGELOG.md)
  * [Registro de cambios del SDK para JavaScript versión preliminar 3](https://github.com/Azure/azure-sdk-for-js/blob/%40azure/ai-form-recognizer_1.0.0-preview.3/sdk/formrecognizer/ai-form-recognizer/CHANGELOG.md)

## <a name="april-2020"></a>Abril de 2020

### <a name="new-features"></a>Nuevas características
* **Compatibilidad del SDK con la versión preliminar pública de la API de Form Recognizer v 2.0** Este mes hemos ampliado nuestro servicio de soporte técnico para incluir un SDK de versión preliminar para la versión v 2.0 (versión preliminar) de Form Recognizer. Use los vínculos siguientes para empezar a trabajar con el lenguaje que prefiera: 
   * [SDK de .NET](https://docs.microsoft.com/dotnet/api/overview/azure/ai.formrecognizer-readme-pre?view=azure-dotnet)
   * [SDK de Java](https://docs.microsoft.com/java/api/overview/azure/ai-formrecognizer-readme-pre?view=azure-java-preview)
   * [SDK de Python](https://docs.microsoft.com/python/api/overview/azure/ai-formrecognizer-readme-pre?view=azure-python-preview)
   * [SDK de JavaScript](https://docs.microsoft.com/javascript/api/overview/azure/ai-form-recognizer-readme-pre?view=azure-node-preview)

  El nuevo SDK es compatible con todas las características de la API REST v 2.0 de Form Recognizer. Por ejemplo, puede entrenar un modelo con o sin etiquetas y extraer texto, pares clave-valor y tablas de los formularios; extraer datos de las confirmaciones con el servicio de confirmaciones pregeneradas; y extraer texto y tablas con el servicio de diseño de los documentos. Puede compartir sus comentarios sobre los SDK mediante el [formulario de comentarios de SDK](https://aka.ms/FR_SDK_v1_feedback).
 
* **Copia del modelo personalizado** Ahora puede copiar modelos entre regiones y suscripciones mediante la nueva característica Copiar modelo personalizado. Antes de invocar la API Copy Custom Model, primero debe obtener la autorización para realizar copias en el recurso de destino mediante la llamada a la operación de autorización de copia en el punto de conexión del recurso de destino.
   * API REST [Generate a copy authorization](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/CopyCustomFormModelAuthorization)
   * API REST [Copy a custom model](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/CopyCustomFormModel) 

### <a name="security-improvements"></a>Mejoras de seguridad

* Las claves administradas por el cliente ahora están disponibles para Form Recognizer. Para obtener más información, consulte [Cifrado de datos en reposo para Form Recognizer](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/form-recognizer-encryption-of-data-at-rest).
* Use Identidades administradas para acceder a los recursos de Azure con Azure Active Directory. Para obtener más información, consulte [Autorización para obtener acceso a identidades administradas](https://docs.microsoft.com/azure/cognitive-services/authentication#authorize-access-to-managed-identities).

## <a name="march-2020"></a>Marzo de 2020 

### <a name="new-features"></a>Nuevas características

* **Tipos de valor para etiquetar**: ahora puede especificar los tipos de valores que va a etiquetar con la herramienta de etiquetado de ejemplo de Form Recognizer. Actualmente se admiten los siguientes tipos de valor y variaciones:
  * `string`
    * predeterminado, `no-whitespaces`, `alphanumeric`
  * `number`
    * predeterminado, `currency`
  * `date` 
    * predeterminado, `dmy`, `mdy`, `ymd`
  * `time`
  * `integer`

  Para aprender a usar esta característica, consulte la [herramienta de etiquetado de ejemplo](./quickstarts/label-tool.md#specify-tag-value-types).


* **Visualización de tablas**: la herramienta de etiquetado de ejemplo ahora muestra las tablas que se reconocieron en el documento. Esto le permite ver las tablas que se han reconocido y extraído del documento antes de etiquetarlas y analizarlas. Esta característica se puede activar o desactivar mediante la opción de capas.

  Este es un ejemplo de cómo se reconocen y extraen las tablas:

  > [!div class="mx-imgBorder"]
  > ![Visualización de tablas mediante la herramienta de etiquetado de ejemplo](./media/whats-new/formre-table-viz.png)

    Las tablas extraídas están disponible en la salida JSON en `"pageResults"`.

  > [!IMPORTANT]
  > No se admite el etiquetado de tablas. Si las tablas no se reconocen ni se extraen automáticamente, solo se pueden etiquetar como pares clave-valor. Al etiquetar tablas como pares clave-valor, etiquete cada celda como un valor único.

### <a name="extraction-enhancements"></a>Mejoras de extracción

Esta versión incluye mejoras de extracción y mejoras de precisión, en concreto, la capacidad de etiquetar y extraer varios pares clave-valor en la misma línea de texto. 
 
### <a name="sample-labeling-tool-is-now-open-source"></a>La herramienta de etiquetado de ejemplo ahora es de código abierto

La herramienta de etiquetado de ejemplo de Form Recognizer ahora está disponible como proyecto de código abierto. Puede integrarla en sus soluciones y realizar cambios específicos del cliente para satisfacer sus necesidades.

Para más información sobre la herramienta de etiquetado de ejemplo de Form Recognizer, revise la documentación disponible en [GitHub](https://github.com/microsoft/OCR-Form-Tools/blob/master/README.md).

### <a name="tls-12-enforcement"></a>Cumplimiento de TLS 1.2

TLS 1.2 ya se exige en todas las solicitudes HTTP para este servicio. Para más información, consulte [Seguridad de Azure Cognitive Services](../cognitive-services-security.md).

## <a name="january-2020"></a>Enero de 2020

En esta versión se introduce Form Recognizer 2.0 (versión preliminar). En las secciones siguientes encontrará más información sobre nuevas características, mejoras y cambios. 

### <a name="new-features"></a>Nuevas características

* **Modelo personalizado**
  * **Entrenar con etiquetas** Ahora puede entrenar un modelo personalizado con datos etiquetados manualmente. Esto da como resultado modelos con un mejor rendimiento y puede generar modelos que funcionen con formularios complejos o formularios que contengan valores sin claves.
  * **API asincrónica** Puede usar llamadas API asincrónicas para entrenar y analizar grandes conjuntos de datos y archivos.
  * **Compatibilidad con archivos TIFF** Ahora puede entrenar y extraer datos de documentos TIFF.
  * **Mejoras en la precisión de la extracción**

* **Modelo de recibo pregenerado**
  * **Importes de propinas** Ahora puede extraer importes de propina y otros valores escritos a mano.
  * **Extracción de elementos de línea** Puede extraer valores de elementos de línea de recibos.
  * **Valores de confianza** Puede ver la confianza del modelo para cada valor extraído.
  * **Mejoras en la precisión de la extracción**

* **Extracción de diseño** Ahora puede usar la API de diseño para extraer datos de texto y datos de tabla de los formularios.

### <a name="custom-model-api-changes"></a>Cambios en la API del modelo personalizado

Se ha cambiado el nombre de todas las API de entrenamiento y uso de modelos personalizados y algunos métodos sincrónicos son asincrónicos ahora. A continuación se indican los cambios principales:

* El proceso de entrenamiento de un modelo es ahora asincrónico. Inicie el entrenamiento mediante la llamada API **/custom/models**. Esta llamada devuelve un identificador de operación, que se puede pasar a **custom/models/{modelID}** para devolver los resultados del entrenamiento.
* La extracción de clave y valor se inicia ahora mediante la llamada API **/custom/models/{modelID}/analyze**. Esta llamada devuelve un identificador de operación, que se puede pasar a **custom/models/{modelID}/analyzeResults/{resultID}** para devolver los resultados de la extracción.
* Los identificadores de operación de la operación de entrenamiento se encuentran ahora en el encabezado **Location** de las respuestas HTTP, no en el encabezado **Operation-Location**.

### <a name="receipt-api-changes"></a>Cambios en la API del recibo

Se ha cambiado el nombre de las API para leer recibos de venta.

* La extracción de datos de recibo ahora se inicia mediante la llamada API **/prebuilt/receipt/analyze**. Esta llamada devuelve un identificador de operación, que se puede pasar a **/prebuilt/receipt/analyzeResults/{resultID}** para devolver los resultados de la extracción.

### <a name="output-format-changes"></a>Cambios del formato de salida

Las respuestas JSON para todas las llamadas API tienen nuevos formatos. Algunas claves y valores se han agregado, quitado o cambiado de nombre. Consulte las guías de inicio rápido para obtener ejemplos de los formatos JSON actuales.

## <a name="next-steps"></a>Pasos siguientes

Realice un [inicio rápido](quickstarts/curl-train-extract.md) para empezar a usar las [API de Form Recognizer](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm).
