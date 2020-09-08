---
title: Adjuntar Cognitive Services a un conjunto de aptitudes
titleSuffix: Azure Cognitive Search
description: Instrucciones para asociar una suscripción todo en uno de Cognitive Services a una canalización de enriquecimiento de inteligencia artificial en Azure Cognitive Search.
manager: nitinme
author: LuisCabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 12/17/2019
ms.openlocfilehash: c9f6a5ebc4f3242181196bd40b62f7522d025b84
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88924984"
---
# <a name="attach-a-cognitive-services-resource-to-a-skillset-in-azure-cognitive-search"></a>Asociación de un recurso de Cognitive Services con un conjunto de aptitudes en Azure Cognitive Search 

Al configurar una canalización de enriquecimiento en Azure Cognitive Search, se puede enriquecer un número limitado de documentos de forma gratuita. En el caso de cargas de trabajo más grandes y más frecuentes, hay que asociar un recurso facturable de Cognitive Services.

En este artículo, aprenderá a asociar un recurso asignando una clave a un conjunto de aptitudes que define una canalización de enriquecimiento.

## <a name="resources-used-during-enrichment"></a>Recursos usados durante el enriquecimiento

Azure Cognitive Search tiene una dependencia en Cognitive Services, lo que incluye [Computer Vision](https://azure.microsoft.com/services/cognitive-services/computer-vision/) para el análisis de imágenes y el reconocimiento óptico de caracteres (OCR), [Text Analytics](https://azure.microsoft.com/services/cognitive-services/text-analytics/) para el procesamiento de lenguaje natural y otros enriquecimientos como [Translator Text](https://azure.microsoft.com/services/cognitive-services/translator-text-api/). Tal como los utiliza Azure Cognitive Search con fines de enriquecimiento de contenido, los algoritmos con IA se incluyen en una *aptitud*, ubicada en un *conjunto de aptitudes* y mencionada por un *indexador* durante la indexación.

## <a name="how-billing-works"></a>Cómo funciona la facturación

+ Azure Cognitive Search usa la clave de recurso de Cognitive Services proporcionada por el usuario en un conjunto de aptitudes con objeto de facturar el enriquecimiento de imágenes y texto. La ejecución de aptitudes facturables tiene lugar al [precio de pago por uso de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/).

+ La extracción de imágenes es una operación de Azure Cognitive Search que se produce al descifrar documentos antes del enriquecimiento. La extracción de imágenes es facturable. Para ver los precios de la extracción de imágenes, consulte la [página de precios de Azure Cognitive Search](https://azure.microsoft.com/pricing/details/search/).

+ La extracción de texto también se produce durante la fase de descifrado de documentos. No es facturable.

+ Las aptitudes que no llaman a Cognitive Services, como las aptitudes Condicional, Conformador, Combinación de texto y División de texto, no son facturables.

## <a name="same-region-requirement"></a>Requisito de la misma región

Se requiere la existencia de Azure Cognitive Search y Azure Cognitive Services en la misma región. De lo contrario, recibirá este mensaje en tiempo de ejecución: `"Provided key is not a valid CognitiveServices type key for the region of your search service."` 

No existe forma alguna de migrar un servicio entre regiones. Si recibe este error, debe crear un nuevo recurso de Cognitive Services en la misma región que Azure Cognitive Search.

> [!NOTE]
> Algunas habilidades integradas se basan en una instancia de Cognitive Services no regional (por ejemplo, la [habilidad de traducción de texto](cognitive-search-skill-text-translation.md)). El uso de una habilidad no regional implica que la solicitud podría ser atendida en una región distinta de la región de Azure Cognitive Search. Para obtener más información sobre los servicios no regionales, consulte la página de [productos de Cognitive Services por región](https://aka.ms/allinoneregioninfo).

## <a name="use-free-resources"></a>Uso de recursos gratis

Puede usar una opción de procesamiento limitada y gratuita para completar los ejercicios del tutorial y la guía de inicio rápido de enriquecimiento con inteligencia artificial.

Los recursos gratuitos (enriquecimientos limitados) se restringen a 20 documentos al día por indexador. Puede eliminar y volver a crear el indexador para restablecer el contador.

1. Abra el Asistente para la importación de datos:

   ![Abra el Asistente para la importación de datos](media/search-get-started-portal/import-data-cmd.png "Abra el Asistente para la importación de datos")

1. Elija un origen de datos y siga para **Add AI enrichment (Optional)** (Agregar el enriquecimiento con IA [opcional]). Para un tutorial paso a paso de este Asistente, consulte [Creación de un índice de Azure Search en Azure Portal](search-get-started-portal.md).

1. Expanda **Adjuntar Cognitive Services** y seleccione **Gratis (enriquecimientos limitados)** :

   ![Sección Adjuntar Cognitive Services expandida](./media/cognitive-search-attach-cognitive-services/attach1.png "Sección Adjuntar Cognitive Services expandida")

1. Ahora puede seguir con los próximos pasos, incluido **Agregar aptitudes cognitivas**.

## <a name="use-billable-resources"></a>Uso de recursos facturables

Para cargas de trabajo que creen más de 20 enriquecimientos al día, asegúrese de asociar un recurso facturable de Cognitive Services. Se recomienda asociar siempre un recurso facturable de Cognitive Services, incluso si nunca ha querido llamar a las API de Cognitive Services. Al asociar un recurso se invalida el límite diario.

Solo se aplican cargos por las aptitudes que llaman a Cognitive Services APIs. No se le facturará por las [aptitudes personalizadas](cognitive-search-create-custom-skill-example.md) o aptitudes como la de [combinación de texto](cognitive-search-skill-textmerger.md), la de [división de texto](cognitive-search-skill-textsplit.md) y la de [conformador](cognitive-search-skill-shaper.md), que no están basadas en API.

1. Abra el Asistente para la importación de datos, elija un origen de datos y siga a **Add AI enrichment (Optional)** (Agregar enriquecimiento con IA [opcional]).

1. Expanda **Adjuntar Cognitive Services** y, a continuación, seleccione **Crear nuevo recurso de Cognitive Services**. Se abre una nueva pestaña para poder crear el recurso:

   ![Cree un recurso de Cognitive Services](./media/cognitive-search-attach-cognitive-services/cog-services-create.png "Creación de un recurso de Cognitive Services")

1. En la lista **Ubicación**, seleccione la región donde se encuentra su instancia de Azure Cognitive Search. Asegúrese de usar esta región por motivos de rendimiento. Al usar esta región, también se anulan los cobros por ancho de banda saliente entre regiones.

1. En la lista **Plan de tarifa**, seleccione **S0** para obtener la colección todo en uno de características de Cognitive Services, incluidas las características Visión e Idioma que respaldan las aptitudes integradas que proporciona Azure Cognitive Search.

   Para el nivel S0, encontrará tarifas para cargas de trabajo específicas en la [página de precios de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/).
  
   + En la lista **Select Offer** (Seleccionar oferta), asegúrese de que **Cognitive Services** está seleccionado.
   + En las características **Idioma**, las tarifas para **Text Analytics Standard** se aplican a la indexación de inteligencia artificial.
   + En las características **Visión**, se aplican las tarifas para **Computer Vision S1**.

1. Seleccione **Crear** para aprovisionar el nuevo recurso de Cognitive Services.

1. Vuelva a la pestaña anterior, donde se encuentra el Asistente para la importación de datos. Seleccione **Actualizar** para mostrar el recurso de Cognitive Services y, luego, seleccione el recurso:

   ![Seleccione el recurso de Cognitive Services](./media/cognitive-search-attach-cognitive-services/attach2.png "Seleccione el recurso de Cognitive Services")

1. Expanda la sección **Agregar aptitudes cognitivas** para seleccionar las aptitudes cognitivas específicas que desea ejecutar en los datos. Realice los pasos restantes del asistente.

## <a name="attach-an-existing-skillset-to-a-cognitive-services-resource"></a>Asociación de un conjunto de aptitudes existentes con un recurso de Cognitive Services

Si ya tiene un conjunto de aptitudes, puede asociarlo a un recurso de Cognitive Services nuevo o distinto.

1. En la página **Información general del servicio**, seleccione **Conjuntos de aptitudes**:

   ![Pestaña Conjuntos de aptitudes](./media/cognitive-search-attach-cognitive-services/attach-existing1.png "Pestaña Conjuntos de aptitudes")

1. Seleccione el nombre del conjunto de aptitudes y un recurso existente o cree uno nuevo. Seleccione **Aceptar** para confirmar los cambios.

   ![Lista de recursos del conjunto de aptitudes](./media/cognitive-search-attach-cognitive-services/attach-existing2.png "Lista de recursos del conjunto de aptitudes")

   Recuerde que **Gratis (enriquecimientos limitados)** se limita a 20 documentos diarios y que puede usar **Crear nuevo recurso de Cognitive Services** para aprovisionar un recurso facturable nuevo. Si crea un nuevo recurso, seleccione **Actualizar** para actualizar la lista de recursos de Cognitive Services y, a continuación, seleccione el recurso.

## <a name="attach-cognitive-services-programmatically"></a>Asociación de Cognitive Services mediante programación

Cuando defina un conjunto de aptitudes mediante programación, agregue una sección `cognitiveServices` al conjunto de aptitudes. En la sección, incluya la clave del recurso de Cognitive Services que desea asociar con el conjunto de aptitudes. Recuerde que el recurso debe estar en la misma región que su recurso de Azure Cognitive Search. Además incluya `@odata.type` y establézcalo en `#Microsoft.Azure.Search.CognitiveServicesByKey`.

En el siguiente ejemplo se muestra este patrón. Observe la sección `cognitiveServices` al final de la definición.

```http
PUT https://[servicename].search.windows.net/skillsets/[skillset name]?api-version=2020-06-30
api-key: [admin key]
Content-Type: application/json
```
```json
{
    "name": "skillset name",
    "skills": 
    [
      {
        "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
        "categories": [ "Organization" ],
        "defaultLanguageCode": "en",
        "inputs": [
          {
            "name": "text", "source": "/document/content"
          }
        ],
        "outputs": [
          {
            "name": "organizations", "targetName": "organizations"
          }
        ]
      }
    ],
    "cognitiveServices": {
        "@odata.type": "#Microsoft.Azure.Search.CognitiveServicesByKey",
        "description": "mycogsvcs",
        "key": "<your key goes here>"
    }
}
```

## <a name="example-estimate-costs"></a>Ejemplo: cálculo de los costos

Para calcular los costos asociados con la indexación de búsqueda cognitiva, empiece con una idea de cómo debe verse un documento promedio para que pueda ejecutar algunos números. Un cálculo aproximado, por ejemplo, sería:

+ 1000 archivos PDF.
+ Seis páginas cada uno.
+ Una imagen por página (6000 imágenes).
+ 3000 caracteres por página.

Supongamos que hay una canalización que consta del descifrado de cada documento PDF con extracción de imágenes y texto, reconocimiento óptico de caracteres (OCR) de las imágenes y reconocimiento de entidad de las organizaciones.

Los precios mostrados en este artículo son hipotéticos. Se usan para ilustrar el proceso de estimación. Sus costos podrían ser más bajos. Para ver los precios reales de las transacciones, consulte [Precios de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services).

1. Para el descifrado de documentos con contenido de texto e imagen, la extracción de texto actualmente es gratis. Para 6000 imágenes, suponga 1 USD por cada 1000 imágenes extraídas. Eso es un costo de 6 USD para este paso.

2. Para realizar el reconocimiento óptico de caracteres (OCR) de 6000 imágenes en inglés, la aptitud de reconocimiento de OCR usa el mejor algoritmo (DescribeText). Suponiendo un precio de 2,50 USD por cada 1000 imágenes que se analizan, tendríamos que pagar 15,00 USD en este paso.

3. Para la extracción de entidades, tendríamos un total de tres registros de texto por página. Cada registro tiene 1000 caracteres. Tres registros de texto por página multiplicados por 6000 páginas equivale a 18 000 registros de texto. Suponiendo un costo de 2,00 USD por cada 1,000 registros de texto, en este paso tendríamos un costo de 36,00 USD.

En resumen, pagaría unos 57 USD por la ingesta de 1000 documentos PDF de este tipo con el conjunto de aptitudes descrito.

## <a name="next-steps"></a>Pasos siguientes
+ [Página de precios de Azure Cognitive Search](https://azure.microsoft.com/pricing/details/search/)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)
+ [Crear un conjunto de aptitudes (REST)](/rest/api/searchservice/create-skillset)
+ [Cómo asignar campos enriquecidos](cognitive-search-output-field-mapping.md)