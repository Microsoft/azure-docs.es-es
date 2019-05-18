---
title: 'Incorporación de perfiles de puntuación a un índice de búsqueda: Azure Search'
description: Potencie las puntuaciones de clasificación de búsqueda en los resultados de búsqueda de Azure Search mediante la incorporación de perfiles de puntuación.
ms.date: 05/02/2019
services: search
ms.service: search
ms.topic: conceptual
author: Brjohnstmsft
ms.author: brjohnst
ms.manager: cgronlun
translation.priority.mt:
- de-de
- es-es
- fr-fr
- it-it
- ja-jp
- ko-kr
- pt-br
- ru-ru
- zh-cn
- zh-tw
ms.openlocfilehash: 9ccb6944227208cee8601751cf43a53c111c09c6
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/02/2019
ms.locfileid: "65021635"
---
# <a name="add-scoring-profiles-to-an-azure-search-index"></a>Incorporación de perfiles de puntuación a un índice de Azure Search

  La puntuación hace referencia al cálculo de una *puntuación de búsqueda* para todos los artículos devueltos en los resultados de la búsqueda. La puntuación es un indicador de la importancia de un elemento en el contexto de la operación de búsqueda actual. Cuanto mayor sea la puntuación, mayor importancia tendrá el elemento. En los resultados de búsqueda, los artículos están ordenados de mayor a menor, según las puntuaciones de búsqueda calculadas para cada uno.  

 Azure Search usa la puntuación predeterminada para calcular una puntuación inicial, pero puede personalizar el cálculo mediante un *perfil de puntuación*. Los perfiles de puntuación proporcionan un mayor control sobre la clasificación de los elementos en los resultados de búsqueda. Por ejemplo, desea aumentar los elementos según su potencial de ingresos, promover elementos más recientes o quizás aumentar los elementos que han permanecido en inventario demasiado largo.  

 Un perfil de puntuación es parte de la definición del índice que se compone de campos, funciones y parámetros ponderados.  

 Para darle una idea del aspecto de un perfil de puntuación, el ejemplo siguiente muestra un perfil simple denominado «geográfico». Este perfil potencia los artículos que tengan el término de búsqueda en el campo **hotelName**. También usa la función `distance` para dar prioridad a elementos que están a menos de diez kilómetros de la ubicación actual. Si alguien busca el término "inn" y este término forma parte del nombre del hotel, los documentos que incluyan hoteles con "inn" en un radio de 10 km de la ubicación actual aparecerán primero en los resultados de búsqueda.  


```  
"scoringProfiles": [
  {  
    "name":"geo",
    "text": {  
      "weights": {  
        "hotelName": 5
      }                              
    },
    "functions": [
      {  
        "type": "distance",
        "boost": 5,
        "fieldName": "location",
        "interpolation": "logarithmic",
        "distance": {
          "referencePointParameter": "currentLocation",
          "boostingDistance": 10
        }                        
      }                                      
    ]                     
  }            
]
```  


 Para usar este perfil de puntuación, se formula la consulta para especificar el perfil en la cadena de consulta. En la siguiente consulta, observe el parámetro de consulta `scoringProfile=geo` en la solicitud.  

```  
GET /indexes/hotels/docs?search=inn&scoringProfile=geo&scoringParameter=currentLocation--122.123,44.77233&api-version=2019-05-06 
```  

 Esta consulta busca el término «inn» y pasa la ubicación actual. Tenga en cuenta que esta consulta incluye otros parámetros, como `scoringParameter`. Los parámetros de consulta se describen en [Search Documents &#40;Azure Search Service REST API&#41;](https://docs.microsoft.com/rest/api/searchservice/Search-Documents) (Búsqueda de documentos [API REST de Azure Search]).  

 Haga clic en [Ejemplo](#bkmk_ex) para revisar un ejemplo más detallado de un perfil de puntuación.  

## <a name="what-is-default-scoring"></a>¿Qué es la puntuación predeterminada?  
 La puntuación calcula una puntuación de la búsqueda para cada elemento de un conjunto de resultados ordenado por rango. A cada elemento de un conjunto de resultados de búsqueda se le asigna una puntuación de búsqueda, a continuación, se ordena de mayor a menor. Elementos con las puntuaciones más altas se devuelven a la aplicación. De forma predeterminada, se devuelven los 50 mejores, pero puede usar el parámetro `$top` para devolver un número mayor o menor de elementos (hasta 1000 en una sola respuesta).  

La puntuación de búsqueda se calcula en función de las propiedades estadísticas de los datos y la consulta. Azure Search busca documentos que contengan los términos de búsqueda de la cadena de consulta (algunas o todas, dependiendo de `searchMode`), lo cual favorece a los documentos que contienen muchas instancias del término de búsqueda. La puntuación de búsqueda aumenta todavía más si el término es poco frecuente en el índice de datos, pero común dentro del documento. La base de este enfoque de relevancia informática se denomina [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) o frecuencia de documento inverso de la frecuencia del término.  

 Suponiendo que no haya ninguna ordenación personalizada, los resultados se clasifican por el resultado de la búsqueda antes de que se devuelvan a la aplicación de llamada. Si $top no se especifica, se devolverán 50 artículos con la puntuación de búsqueda mayor.  

 Los valores de puntuación de búsqueda pueden repetirse a lo largo de un conjunto de resultados. Por ejemplo, puede tener 10 elementos con una puntuación de 1,2, 20 elementos con una puntuación de 1,0 y 20 elementos con una puntuación de 0,5. Cuando varios resultados tienen la misma puntuación de búsqueda, el orden de estos elementos puntuados no se define y no es estable. Vuelva a ejecutar la consulta verá cómo los elementos cambian de posición. Si dos elementos disponen de la misma puntuación, no hay ninguna garantía de cuál aparecerá en primer lugar.  

## <a name="when-to-use-custom-scoring"></a>Cuándo usar la puntuación personalizada  
 Debe crear uno o más perfiles de puntuación cuando el comportamiento de clasificación predeterminado no logre cumplir los objetivos de su empresa. Por ejemplo, podría decidir que la relevancia de la búsqueda debe favorecer a los elementos recién agregados. Asimismo, podría tener un campo que contenga el margen de beneficio, o algún otro campo que indique los ingresos potenciales. Aumentar los resultados que ofrecen beneficios a su empresa puede ser un factor importante a la hora de decidir usar perfiles de puntuación.  

 El orden basado en relevancia también se implementa a través de perfiles de puntuación. Tenga en cuenta los resultados de búsqueda que ha usado en el pasado que le permiten ordenar por relevancia, fecha, clasificación o precio. En Azure Search, los perfiles de puntuación determinan la opción «relevancia». La definición de relevancia está controlada por usted, se afirma en los objetivos empresariales y en el tipo de experiencia de búsqueda que desee ofrecer.  

##  <a name="bkmk_ex"></a> Ejemplo  
 Como se ha indicado anteriormente, la puntuación personalizada se implementa mediante uno o varios perfiles de puntuación definidos en un esquema de índice.  

 En este ejemplo se muestra el esquema de un índice con dos perfiles de puntuación (`boostGenre`, `newAndHighlyRated`). Las consultas sobre este índice que incluyen cualquiera de los perfiles como parámetro de consulta usará el perfil para puntuar el conjunto de resultados.  

```  
{  
  "name": "musicstoreindex",  
  "fields": [  
    { "name": "key", "type": "Edm.String", "key": true },  
    { "name": "albumTitle", "type": "Edm.String" },  
    { "name": "albumUrl", "type": "Edm.String", "filterable": false },  
    { "name": "genre", "type": "Edm.String" },  
    { "name": "genreDescription", "type": "Edm.String", "filterable": false },  
    { "name": "artistName", "type": "Edm.String" },  
    { "name": "orderableOnline", "type": "Edm.Boolean" },  
    { "name": "rating", "type": "Edm.Int32" },  
    { "name": "tags", "type": "Collection(Edm.String)" },  
    { "name": "price", "type": "Edm.Double", "filterable": false },  
    { "name": "margin", "type": "Edm.Int32", "retrievable": false },  
    { "name": "inventory", "type": "Edm.Int32" },  
    { "name": "lastUpdated", "type": "Edm.DateTimeOffset" }  
  ],  
  "scoringProfiles": [  
    {  
      "name": "boostGenre",  
      "text": {  
        "weights": {  
          "albumTitle": 1.5,  
          "genre": 5,  
          "artistName": 2  
        }  
      }  
    },  
    {  
      "name": "newAndHighlyRated",  
      "functions": [  
        {  
          "type": "freshness",  
          "fieldName": "lastUpdated",  
          "boost": 10,  
          "interpolation": "quadratic",  
          "freshness": {  
            "boostingDuration": "P365D"  
          }  
        },  
        {
          "type": "magnitude",  
          "fieldName": "rating",  
          "boost": 10,  
          "interpolation": "linear",  
          "magnitude": {  
            "boostingRangeStart": 1,  
            "boostingRangeEnd": 5,  
            "constantBoostBeyondRange": false  
          }  
        }  
      ]  
    }  
  ],  
  "suggesters": [  
    {  
      "name": "sg",  
      "searchMode": "analyzingInfixMatching",  
      "sourceFields": [ "albumTitle", "artistName" ]  
    }  
  ]   
}  
```  

## <a name="workflow"></a>Flujo de trabajo  
 Para implementar el comportamiento personalizado de puntuación, agregue un perfil de puntuación al esquema que define el índice. Puede tener hasta 100 perfiles de puntuación en un índice (consulte [Límites de servicio](search-limits-quotas-capacity.md)), pero solo puede especificar un perfil cada vez en una consulta concreta.  

 Comience con la [Plantilla](#bkmk_template) incluida en este tema.  

 Proporcione un nombre. Los perfiles de puntuación son opcionales, pero si agrega uno, se requerirá el nombre. Asegúrese de seguir las convenciones de nomenclatura de los campos (empieza con una letra, evita caracteres especiales y palabras reservadas). Consulte [Naming rules &#40;Azure Search&#41;](https://docs.microsoft.com/rest/api/searchservice/naming-rules) (Reglas de nomenclatura [Azure Search]) para ver la lista completa.  

 El cuerpo del perfil de puntuación se construye a partir de campos ponderados y funciones.  

|||  
|-|-|  
|**Pesos**|Especifique pares nombre-valor que asignen un peso relativo a un campo. En el [ejemplo](#bkmk_ex), los campos albumTitle, genre y artistName se han potenciado 1,5, 5 y 2 respectivamente. ¿Por qué se aumenta el género mucho mayor que los demás? Si la búsqueda se realiza sobre datos en cierto modo homogéneos (como es el caso de "genre" en `musicstoreindex`), es posible que necesite una mayor variación en los pesos relativos. Por ejemplo, en `musicstoreindex`, "rock" aparece como género y en las descripciones de género expresadas de forma idéntica. Si desea que el género supere en ponderación a la descripción del género, el campo del género necesitará una ponderación relativa mucho mayor.|  
|**Funciones**|Se usan cuando se requieren cálculos adicionales para contextos concretos. Los valores válidos son `freshness`, `magnitude`, `distance` y `tag`. Cada función tiene parámetros que son exclusivos de ella.<br /><br /> -   `freshness` debe usarse cuando se quiere potenciar en función de lo nuevo o antiguo que sea un artículo. Esta función solo puede usarse con campos `datetime` (edm.DataTimeOffset). Tenga en cuenta la `boostingDuration` atributo solo se usa con el `freshness` función.<br />-   `magnitude` debe usarse cuando se quiere potenciar en función de lo alto o bajo que sea un valor numérico. Entre los escenarios que requieren esta función se incluyen aumentar por margen de beneficio, precio máximo, precio mínimo o recuento de descargas. Esta función solo puede usarse con campos doble y de número entero.<br />     Para la función `magnitude`, puede invertir el rango, de mayor a menor, si quiere el patrón inverso (por ejemplo, potenciar los artículos de precio inferior más que los de precio superior). Dado un intervalo de precios de 100 $ a 1 $, establezca `boostingRangeStart` en 100 y `boostingRangeEnd` en 1 para potenciar los elementos de menor precio.<br />-   `distance` debe usarse cuando quiera potenciar por proximidad o ubicación geográfica. Esta función solo puede usarse con campos `Edm.GeographyPoint` .<br />-   `tag` debe usarse cuando quiera potenciar por etiquetas en común entre documentos y consultas de búsqueda. Esta función solo puede usarse con campos `Edm.String` y `Collection(Edm.String)`.<br /><br /> **Reglas de uso de funciones**<br /><br /> El tipo de función (`freshness`, `magnitude`, `distance`), `tag` debe estar en minúsculas.<br /><br /> Las funciones no pueden incluir valores nulos ni estar vacías. En concreto, si incluye un nombre de campo, deberá establecerlo en un valor.<br /><br /> Las funciones solo pueden aplicarse a los campos que se pueden filtrar. Consulte [Create Index &#40;Azure Search Service REST API&#41;](https://docs.microsoft.com/rest/api/searchservice/create-index) (Creación de un índice [API REST del servicio Azure Search]) para más información sobre los campos que se pueden filtrar.<br /><br /> Las funciones solo pueden aplicarse a los campos que se definen en la colección de campos de un índice.|  

 Una vez definido el índice, genere el índice mediante la carga del esquema de índice, seguido de documentos. Consulte [Create Index &#40;Azure Search Service REST API&#41;](https://docs.microsoft.com/rest/api/searchservice/create-index) (Creación de un índice [API REST del servicio Azure Search]) y [Add, Update or Delete Documents &#40;Azure Search Service REST API&#41;](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents) (Incorporación, actualización o eliminación de documentos [API REST del servicio Azure Search]) para obtener instrucciones sobre estas operaciones. Una vez creado el índice, debe tener un perfil de puntuación funcional que funcione con los datos de búsqueda.  

##  <a name="bkmk_template"></a> Plantilla  
 En esta sección se muestra la sintaxis y la plantilla de perfiles de puntuación. Consulte [Referencia de atributos de índice](#bkmk_indexref) en la sección siguiente para ver las descripciones de los atributos.  

```  
. . .   
"scoringProfiles": [  
  {   
    "name": "name of scoring profile",   
    "text": (optional, only applies to searchable fields) {   
      "weights": {   
        "searchable_field_name": relative_weight_value (positive #'s),   
        ...   
      }   
    },   
    "functions": (optional) [  
      {   
        "type": "magnitude | freshness | distance | tag",   
        "boost": # (positive number used as multiplier for raw score != 1),   
        "fieldName": "...",   
        "interpolation": "constant | linear (default) | quadratic | logarithmic",   

        "magnitude": {
          "boostingRangeStart": #,   
          "boostingRangeEnd": #,   
          "constantBoostBeyondRange": true | false (default)
        }  

        // ( - or -)  

        "freshness": {
          "boostingDuration": "..." (value representing timespan over which boosting occurs)   
        }  

        // ( - or -)  

        "distance": {
          "referencePointParameter": "...", (parameter to be passed in queries to use as reference location)   
          "boostingDistance": # (the distance in kilometers from the reference location where the boosting range ends)   
        }   

        // ( - or -)  

        "tag": {
          "tagsParameter":  "..."(parameter to be passed in queries to specify a list of tags to compare against target field)   
        }
      }
    ],   
    "functionAggregation": (optional, applies only when functions are specified) "sum (default) | average | minimum | maximum | firstMatching"   
  }   
],   
"defaultScoringProfile": (optional) "...",   
. . .  
```  

##  <a name="bkmk_indexref"></a> Referencia de atributos de índice  

> [!NOTE]  
>  Una función de puntuación solo puede aplicarse a campos filtrables.  

|Atributo|DESCRIPCIÓN|  
|---------------|-----------------|  
|`Name`|Necesario. Este es el nombre del perfil de puntuación. Sigue las mismas convenciones de nomenclatura de un campo. Debe comenzar por una letra, no puede contener puntos, dos puntos o símbolos @ y no puede comenzar con "azureSearch" (distingue mayúsculas de minúsculas).|  
|`Text`|Contiene la propiedad Weights.|  
|`Weights`|Opcional. Par de nombre-valor que especifica un nombre de campo y un peso relativo. El peso relativo debe ser un número entero positivo o un número de punto flotante. El valor máximo es int32.MaxValue.<br /><br /> Puede especificar el nombre del campo sin un peso correspondiente. Weights se usa para indicar la importancia de un campo en relación con otro.|  
|`Functions`|Opcional. Una función de puntuación solo puede aplicarse a campos filtrables.|  
|`Type`|Obligatorio para las funciones de puntuación. Indica el tipo de función que se usará. Entre los valores válidos se incluyen magnitud, índice de actualización, distancia y etiqueta. Puede incluir más de una función en cada perfil de puntuación. El nombre de la función debe estar en minúsculas.|  
|`Boost`|Obligatorio para las funciones de puntuación. Número positivo usado como multiplicador para el resultado sin formato. No puede ser igual a 1.|  
|`Fieldname`|Obligatorio para las funciones de puntuación. Una función de puntuación solo puede aplicarse a los campos que forman parte de la colección de campos del índice y que son filtrables. Además, cada tipo de función introduce restricciones adicionales (índice de actualización se usa con campos de fecha y hora, magnitud con número entero o campos dobles y distancia con campos de ubicación). Solo puede especificar un campo único por cada definición de función. Por ejemplo, para usar magnitud dos veces en el mismo perfil, necesitaría incluir dos magnitudes de definiciones, una para cada campo.|  
|`Interpolation`|Obligatorio para las funciones de puntuación. Define la pendiente con la que la potenciación de la puntuación aumenta desde el inicio del rango hasta su final. Entre los valores válidos se incluyen Lineal (valor predeterminado), Constante, Cuadrática y Logarítmico. Consulte [Establecer interpolaciones](#bkmk_interpolation) para más información.|  
|`magnitude`|La función de puntuación de la magnitud se usa para modificar las clasificaciones según el intervalo de valores de un campo numérico. Algunos de los ejemplos de uso más habituales sobre esto son:<br /><br /> -   **Clasificación por estrellas:** modifique la puntuación en función del valor del campo "Clasificación por estrellas". Cuando dos elementos son pertinentes, en primer lugar se mostrará el elemento con la clasificación superior.<br />-   **Margen:** cuando dos documentos son pertinentes, es posible que un distribuidor potencie los documentos que tengan mayores márgenes a las primeras posiciones.<br />-   **Recuentos de clics:** en las aplicaciones que hacen un seguimiento de los clics en productos o páginas, puede usar la magnitud para potenciar los artículos que tienden a atraer el máximo de tráfico.<br />-   **Recuentos de descargas:** en las aplicaciones que hacen el seguimiento de las descargas, la función magnitud le permite potenciar los artículos que tienen más descargas.|  
|`magnitude` &#124; `boostingRangeStart`|Establece el valor inicial del intervalo en el que se puntúa la magnitud. El valor debe ser un número entero o un número de punto flotante. En cuanto a las clasificaciones por estrellas de 1 a 4, esto equivaldría a 1. Para los márgenes superiores al 50%, esto equivaldría a 50.|  
|`magnitude` &#124; `boostingRangeEnd`|Establece el valor final del intervalo en el que se puntúa la magnitud. El valor debe ser un número entero o un número de punto flotante. En cuanto a las clasificaciones por estrellas de 1 a 4, esto equivaldría a 4.|  
|`magnitude` &#124; `constantBoostBeyondRange`|Los valores válidos son true o false (valor predeterminado). Cuando se establece en true, la potenciación completa continuará aplicándose a los documentos que tienen un valor para el campo de destino superior al extremo superior del intervalo. Si es false, la potenciación de esta función no se aplicará a los documentos que tengan un valor para el campo de destino que se encuentre fuera del intervalo.|  
|`freshness`|La función de puntuación de actualización se usa para modificar las puntuaciones de clasificación de los artículos basados en los valores de los campos `DateTimeOffset`. Por ejemplo, un elemento con una fecha más reciente puede clasificarse por encima de los elementos más antiguos.<br /><br /> También es posible clasificar elementos, como eventos del calendario con fechas futuras tal que los elementos más cercanos hasta la actual pueden clasificarse por superiores a los que los elementos adicionales en el futuro.<br /><br /> En la versión actual del servicio, uno de los extremos del intervalo se corregirá a la hora actual. El otro extremo es un momento en el pasado basado en el elemento `boostingDuration`. Para potenciar un intervalo de horas en el futuro, use un valor de `boostingDuration` negativo.<br /><br /> La velocidad a la que cambia la potenciación desde un intervalo máximo y mínimo viene determinada por la interpolación aplicada al perfil de puntuación (consulte la figura siguiente). Para invertir el factor de potenciación aplicado, seleccione un factor de potenciación de menos de 1.|  
|`freshness` &#124; `boostingDuration`|Establece un período de caducidad después del que se detendrá la potenciación de un documento determinado. Consulte [Establecer boostingDuration](#bkmk_boostdur) en la sección siguiente para obtener información sobre la sintaxis y ejemplos.|  
|`distance`|La función de puntuación de la distancia se usa para afectar a la puntuación de documentos en función de la cercanía o distancia respecto a una ubicación geográfica de referencia. La ubicación de referencia se proporciona como parte de la consulta en un parámetro (mediante la opción de cadena `scoringParameterquery`) como argumento lon,lat.|  
|`distance` &#124; `referencePointParameter`|Parámetro que se pasarán en las consultas a usar como ubicación de referencia. `scoringParameter` es un parámetro de consulta. Consulte [Search Documents &#40;Azure Search Service REST API&#41;](https://docs.microsoft.com/rest/api/searchservice/Search-Documents) (Búsqueda de documentos [API REST de Azure Search Service ]) para ver las descripciones de los parámetros de consulta.|  
|`distance` &#124; `boostingDistance`|Número que indica la distancia en kilómetros desde la ubicación de referencia donde finaliza el intervalo de potenciación.|  
|`tag`|La función de puntuación de etiquetas se usa para afectar a la puntuación de documentos basados en las etiquetas de documentos y las consultas de búsqueda. Los documentos que tienen etiquetas en común con la consulta de búsqueda se potenciarán. Las etiquetas de la consulta de búsqueda se proporcionan como un parámetro de puntuación en cada solicitud de búsqueda (mediante la opción de cadena `scoringParameterquery`).|  
|`tag` &#124; `tagsParameter`|Parámetro que se pasará en las consultas para especificar etiquetas para una solicitud en particular. `scoringParameter` es un parámetro de consulta. Consulte [Search Documents &#40;Azure Search Service REST API&#41;](https://docs.microsoft.com/rest/api/searchservice/Search-Documents) (Búsqueda de documentos [API REST de Azure Search Service ]) para ver las descripciones de los parámetros de consulta.|  
|`functionAggregation`|Opcional. Se aplica solo cuando se especifican las funciones. Los valores válidos son: suma (valor predeterminado), promedio, mínimo, máximo y firstMatching. Una puntuación de búsqueda es un valor único que se calcula a partir de varias variables, incluidas varias funciones. Este atributo indica cómo se combinan los valores de potenciación de todas las funciones en una única potenciación total que, luego, se aplica a la puntuación del documento base. La puntuación base se basa en el valor [tf-idf](http://www.tfidf.com/) calculado a partir del documento y la consulta de búsqueda.|  
|`defaultScoringProfile`|Al ejecutar una solicitud de búsqueda, si no se especifica ningún perfil de puntuación, se usará la puntuación predeterminada ([tf-idf](http://www.tfidf.com/) solamente).<br /><br /> Aquí puede establecerse un nombre de perfil de puntuación predeterminado, que provocará que Azure Search use ese perfil cuando no se proporcione ningún perfil específico en la solicitud de búsqueda.|  

##  <a name="bkmk_interpolation"></a> Establecimiento de interpolaciones  
 Las interpolaciones permiten establecer la forma de la pendiente usada para la puntuación. Dado que la puntuación es de alta a baja, la pendiente siempre disminuye, pero la interpolación determina la curva de la pendiente descendente. Es posible usar las siguientes interpolaciones:  

|||  
|-|-|  
|`Linear`|En los artículos que están dentro del intervalo máximo y mínimo, la potenciación aplicada al artículo se realizará en un grado que va decreciendo de manera constante. Lineal es la interpolación predeterminada de un perfil de puntuación.|  
|`Constant`|En los artículos que se encuentran dentro del intervalo de inicio y finalización, se aplicará una potenciación constante a los resultados de la clasificación.|  
|`Quadratic`|En comparación con una interpolación lineal que tiene una potentiación en reducción constante, Cuadrática provoca inicialmente una reducción a un ritmo inferior y, a continuación, a medida que se aproxima el final del intervalo, se reduce a un intervalo muy superior. Esta opción de interpolación no se permite en funciones de puntuación de etiquetas.|  
|`Logarithmic`|En comparación con una interpolación lineal que tiene una potenciación en reducción constante, Logarítmica provoca inicialmente una reducción a un ritmo superior y, a continuación, a medida que se aproxima el final del intervalo, se reduce a un intervalo muy inferior. Esta opción de interpolación no se permite en funciones de puntuación de etiquetas.|  

 ![Líneas de constante, lineal, cuadrática y log10 en el gráfico](media/scoring-profiles/azuresearch_scorefunctioninterpolationgrapht.png "AzureSearch_ScoreFunctionInterpolationGrapht")  

##  <a name="bkmk_boostdur"></a> Establecimiento de boostingDuration  
 `boostingDuration` es un atributo de la función `freshness`. Se usa para establecer un período de caducidad después del que se detendrá la potenciación de un documento determinado. Por ejemplo, para potenciar una línea de productos o marca durante un período de promoción de 10 días, debe especificar el período de 10 días como "P10D" para dichos documentos.  

 `boostingDuration` debe tener el formato de un valor "dayTimeDuration" XSD (subconjunto restringido de un valor de duración ISO 8601). El patrón de este valor es: "P[nD][T[nH][nM][nS]]".  

 La tabla siguiente proporciona varios ejemplos.  

|Duration|boostingDuration|  
|--------------|----------------------|  
|1 día|"P1D"|  
|2 días, 12 horas|"P2DT12H"|  
|15 minutos|"PT15M"|  
|30 días, 5 horas, 10 minutos y 6,334 segundos|"P30DT5H10M6.334S"|  

 Para ver más ejemplos, consulte el [esquema XML: Datatypes (sitio web de W3.org)](https://www.w3.org/TR/xmlschema11-2/#dayTimeDuration).  

## <a name="see-also"></a>Vea también  
 [API REST de Azure Search Service](https://docs.microsoft.com/rest/api/searchservice/)   
 [Create Index &#40;Azure Search Service REST API&#41;](https://docs.microsoft.com/rest/api/searchservice/create-index)  (Creación de un índice [API REST de Azure Search Service])  
 [SDK de .NET de Azure Search+](https://docs.microsoft.com/dotnet/api/overview/azure/search?view=azure-dotnet)  
