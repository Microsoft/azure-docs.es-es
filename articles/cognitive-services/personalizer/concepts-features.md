---
title: 'Características: Acción y contexto (Personalizer)'
titleSuffix: Azure Cognitive Services
description: Personalizer usa características, información acerca de las acciones y del contexto, para realizar sugerencias para mejorar la clasificación. Las características pueden ser genéricas o específicas de un elemento.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 10/14/2019
ms.openlocfilehash: 590416f077fc1ff9430e42e27217548476c9032f
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132779"
---
# <a name="features-are-information-about-actions-and-context"></a>Las características son información acerca de las acciones y del contexto

El servicio Personalizer aprende lo que la aplicación debería mostrar a los usuarios en un contexto determinado.

Personalizer usa **características**, que es la información acerca del **contexto actual**, para elegir la mejor **acción**. Las características representan toda la información que crea que es útil personalizar para lograr recompensas mayores. Las características pueden ser genéricas o específicas de un elemento. 

Por ejemplo, puede tener una **característica** acerca de:

* _Rol de usuario_, como `Sports_Shopper`. No debe ser un identificador de usuario individual. 
* El _contenido_, como por ejemplo, si un vídeo es de las categorías `Documentary`, `Movie` o `TV Series`, o si un artículo comercial está disponible en el almacén.
* El periodo _actual_, como qué día de la semana es.

Personalizer no prescribe, limita ni corrige las características que puede enviar a las acciones y el contexto:

* Puede enviar algunas características a unas acciones y no a otras, en caso de que no las tenga. Por ejemplo, las series de televisión pueden tener atributos que no tienen las películas.
* Es posible que algunas características estén disponibles solo en algunas ocasiones. Por ejemplo, una aplicación móvil puede proporcionar más información que una página web. 
* Con el tiempo puede agregar y quitar características acerca del contexto y de las acciones. Personalizer sigue aprendiendo de la información disponible.
* Debe haber al menos una característica para el contexto. Personalizer no admite un contexto vacío. Si solo envía un contexto fijo cada vez, Personalizer elegirá la acción para las calificaciones teniendo en cuenta solo las características de las acciones.
* En el caso de las características de categorías, no es necesario definir los valores posibles ni definir previamente los rangos para los valores numéricos.

## <a name="supported-feature-types"></a>Tipos de características que se admiten

Personalizer admite características de los tipos cadena, numérico y booleano.

### <a name="how-choice-of-feature-type-affects-machine-learning-in-personalizer"></a>Cómo afecta la elección del tipo de característica a Machine Learning en Personalizer

* **Cadenas**: Para los tipos de cadena, cada combinación de clave y valor crea nuevas ponderaciones en el modelo de aprendizaje automático de Personalizer. 
* **Numérico**: Debe usar valores numéricos cuando el número debe afectar proporcionalmente al resultado de la personalización. Esto depende mucho del escenario. En un ejemplo simplificado, al personalizar una experiencia de venta al por menor, NumberOfPetsOwned (número de mascotas en propiedad) podría ser una característica de tipo numérico si quiere que las personas con 2 o 3 mascotas tengan influencia en el resultado de la personalización el doble o el triple de las que tienen 1 mascota. Las características que se basan en unidades numéricas cuyo significado no es lineal, como la edad, la temperatura o la estatura de la persona, se podrían codificar mejor como cadenas y la calidad de la característica normalmente se puede mejorar mediante el uso de rangos. Por ejemplo, la edad podría codificarse como "Age":"0-5", "Age":"6-10", etc.
* Los valores **booleanos** enviados con el valor "false" actúan como si no se hubiesen enviado.

Las características que no estén presentes deben omitirse de la solicitud. Procure no enviar características con un valor nulo, ya que se procesarán como existentes y con un valor "null" al entrenar el modelo.

## <a name="categorize-features-with-namespaces"></a>Clasificación de características con espacios de nombres

Personalizer toma las características organizadas en espacios de nombres. El usuario, por su parte, es el que determina, en la aplicación, si se usan espacios de nombres y cuáles deben ser dichos espacios. Los espacios de nombres se utilizan para agrupar las características que son de un tema similar o que proceden de un origen determinado.

A continuación encontrará ejemplos de espacios de nombres de características usados por las aplicaciones:

* User_Profile_from_CRM
* Time
* Mobile_Device_Info
* http_user_agent
* VideoResolution
* UserDeviceInfo
* Tiempo
* Product_Recommendation_Ratings
* current_time
* NewsArticle_TextAnalytics

Al asignar nombres a los espacios de nombres de las características puede seguir sus propias convenciones, siempre sean claves JSON válidas. Los espacios de nombres se usan para organizar características en conjuntos distintos y para eliminar la ambigüedad de las características con nombres similares. Los espacios de nombres se pueden considerar como un "prefijo" que se agrega a los nombres de las características. Los espacios de nombres no se pueden anidar.


En el siguiente JSON, `user`, `state` y `device` son espacios de nombres de características. 

> [!Note]
> Actualmente, le recomendamos que use nombres para los espacios de nombres de características que se basen en el formato UTF-8 y comiencen por letras diferentes. Por ejemplo, los términos `user`, `state` y `device` empiezan por `u`, `s` y `d`. Actualmente, el uso de espacios de nombres con los mismos caracteres iniciales puede provocar colisiones en los índices usados para el aprendizaje automático.

Los objetos JSON pueden incluir objetos JSON anidados y propiedades y valores simples. Solo se puede incluir una matriz si los elementos de la matriz son números. 

```JSON
{
    "contextFeatures": [
        { 
            "user": {
                "profileType":"AnonymousUser",
                "latlong": [47.6, -122.1]
            }
        },
        {
            "state": {
                "timeOfDay": "noon",
                "weather": "sunny"
            }
        },
        {
            "device": {
                "mobile":true,
                "Windows":true
            }
        }
    ]
}
```

### <a name="restrictions-in-character-sets-for-namespaces"></a>Restricciones en los juegos de caracteres de los espacios de nombres

La cadena que se utiliza para asignar nombres al espacio de nombres debe seguir algunas restricciones: 
* No puede ser unicode.
* Puede usar algunos de los símbolos imprimibles con códigos < 256 para los nombres de los espacios de nombres. 
* No puede usar símbolos con códigos < 32 (no imprimibles), 32 (espacio), 58 (dos puntos), 124 (canalización) y 126 – 140.
* No debe comenzar con un carácter de subrayado "_" o la característica se omitirá.

## <a name="how-to-make-feature-sets-more-effective-for-personalizer"></a>Cómo hacer que los conjuntos de características sean más eficaces para Personalizer

Un buen conjunto de características ayuda a Personalizer a aprender a predecir la acción que determinará la mayor recompensa. 

Considere la posibilidad de enviar características a la API Personalizer Rank que sigan estas recomendaciones:

* Hay suficiente características para realizar la personalización. Cuanto más precisión deba aplicarse a la hora de dirigir el contenido, más características se necesitan.

* Hay suficientes características de diversas *densidades*. Una característica es *densa* si muchos elementos se agrupan en pocos cubos. Por ejemplo, miles de vídeos se pueden clasificar como "Long" (más de 5 minutos de duración) y "Short" (menos de 5 minutos de duración). Esta es una característica *muy densos*. Por otro lado, los mismos miles de elementos pueden tener un atributo llamado "Title", que casi nunca tendrá el mismo valor en dos elementos. Esta es una característica no muy densa o *rala*.  

La posesión de características de alta densidad ayuda a Personalizer a extrapolar el aprendizaje de un elemento a otro. Pero si hay solo unas pocas características y estas son muy densas, Personalizer intentará dirigir el contenido con precisión y habrá pocos cubos entre los que elegir.

### <a name="improve-feature-sets"></a>Mejora de los conjuntos de características 

Analice el comportamiento del usuario mediante una evaluación sin conexión. Esto permite examinar datos pasados para ver qué características contribuyen considerablemente a la hora de lograr recompensas positivas y cuáles contribuyen menos. Puede ver las características que sirven de ayuda y dependerá de usted y de la aplicación encontrar características mejores para enviarlas a Personalizer, con el fin de mejorar aún más los resultados.

En las secciones siguientes encontrará prácticas comunes para mejorar las características que se envían a Personalizer.

#### <a name="make-features-more-dense"></a>Aumento de la densidad de las características

Los conjuntos de características se pueden mejorar. Para ello, es preciso editarlos, con el fin de aumentar su tamaño y aumentar o reducir su densidad.

Por ejemplo, una marca de tiempo ajustada al segundo es una característica muy rala. Su densidad (eficacia) podría aumentar si el tiempo se clasifica en "mañana", "mediodía" y "tarde", etc.

La información de ubicación también suele beneficiarse de la creación de clasificaciones más amplias. Por ejemplo, una coordenada de latitud-longitud como Lat: 47,67402° N, Long: 122,12154° W es demasiado preciso y obliga al modelo a aprender la latitud y la longitud como dimensiones distintas. Cuando intenta personalizar según la información de ubicación, resulta de ayuda agrupar la información de ubicación en sectores más grandes. Una manera fácil de hacerlo es elegir una precisión de redondeo adecuada para los números de Lat-Long y combinar la latitud y la longitud en "áreas" convirtiéndolas en una sola cadena. Por ejemplo, una buena manera de representar 47,67402° N, Long: 122,12154° W en regiones aproximadamente de unos pocos kilómetros de ancho sería "ubicación": "34.3, 12.1".


#### <a name="expand-feature-sets-with-extrapolated-information"></a>Expansión de los conjuntos de características con información extrapolada

También puede obtener más características pensando en atributos sin explorar que se pueden derivar de la información que ya tiene. Por ejemplo, en una personalización ficticia de una lista de películas, ¿es posible que el comportamiento de los usuarios en el fin de semana sea distinto que en los días laborables? El tiempo puede ampliarse para tener un atributo "weekend" o "weekday". En los días festivos nacionales de carácter cultural, ¿hay más asistencia a determinados tipos de película? Por ejemplo, un atributo "Halloween" es útil en aquellos lugares en que es relevante. ¿Es posible que la lluvia afecte considerablemente a muchas personas a la hora de elegir una película? En cuanto al tiempo y al lugar, un servicio meteorológico puede proporcionar dicha información y esta se puede agregar como una característica adicional. 

#### <a name="expand-feature-sets-with-artificial-intelligence-and-cognitive-services"></a>Expansión de conjuntos de características con inteligencia artificial y Cognitive Services

Tanto la inteligencia artificial como una instancia de Cognitive Services lista para ejecutarse pueden ser una incorporación muy eficaz a Personalizer. 

Si se procesan previamente los elementos mediante servicios de inteligencia artificial, es posible extraer automáticamente información que es probable que sea relevante para la personalización.

Por ejemplo:

* Puede ejecutar un archivo de películas a través de [Video Indexer](https://azure.microsoft.com/services/media-services/video-indexer/) para extraer elementos de escenas, texto, opiniones y muchos otros atributos. La densidad de estos atributos pueden aumentar para reflejar las particularidades que no tenían los metadatos del elemento original. 
* Las imágenes se pueden ejecutar mediante la detección de objetos, las mediante las opiniones, etc.
* La información en forma de texto se puede aumentar mediante la extracción de entidades y opiniones, y la expansión de las entidades con el grafo de conocimientos de Bing, etc.

Se pueden usar otros elementos de [Azure Cognitive Services](https://www.microsoft.com/cognitive-services), como

* [Entity Linking](../entitylinking/home.md)
* [Text Analytics](../text-analytics/overview.md)
* [Emotion](../emotion/home.md)
* [Computer Vision](../computer-vision/home.md)

## <a name="actions-represent-a-list-of-options"></a>Las acciones representan una lista de opciones

Cada acción:

* Tiene un identificador de _evento_. Si ya tiene un identificador de evento, debe enviarlo. Si no tiene un identificador de evento, no envíe ninguno, Personalizer crea uno automáticamente y lo devuelve en respuesta a la solicitud de Rank. El identificador está asociado al evento de Rank, no al usuario. Si crea un identificador, un GUID funciona mejor. 
* Tiene una lista de características.
* La lista de características puede ser grande (centenares), pero se recomienda evaluar la eficacia de estas para quitar las que no contribuyen a obtener recompensas. 
* Las características de las **acciones** pueden tener, o no, correlación con las características del **contexto** que usa Personalizer.
* Las características para las acciones pueden estar presentes, y no estarlo en otras. 
* Las características de un identificador de acción determinado puede estar disponible un día, pero dejar de estarlo más adelante. 

Los algoritmos de aprendizaje automático de Personalizer funcionarán mejor cuando haya conjuntos de características estables, pero las llamadas a Rank podrán realizarse si el conjunto de características cambia con el paso del tiempo.

No envíe más de 50 acciones cuando clasifique por orden de prioridad las acciones. Pueden ser siempre las mismas 50 acciones o pueden cambiar cada vez. Por ejemplo, si tiene un catálogo de productos de 10 000 elementos para una aplicación de comercio electrónico, puede usar una recomendación o un motor de filtrado para determinar los 40 que más pueden gustar a un cliente y usar Personalizer para encontrar el que generará la máxima recompensa (por ejemplo, el usuario lo agregará a la cesta) para el contexto actual.


### <a name="examples-of-actions"></a>Ejemplos de acciones

Las acciones que envíe a la API Rank dependerán de lo que intente personalizar.

Estos son algunos ejemplos:

|Propósito|Acción|
|--|--|
|Personalizar qué artículo está resaltado en un sitio web de noticias.|Cada acción es una potencial noticia.|
|Optimizar la colocación de anuncios en un sitio web.|Cada acción será un diseño o reglas para crear un diseño para los anuncios (por ejemplo, en la parte superior, a la derecha, imágenes pequeñas, imágenes grandes).|
|Mostrar una clasificación personalizada de elementos recomendados en un sitio web de compras.|Cada acción es un producto concreto.|
|Sugerir elementos de la interfaz de usuario, como filtros, para aplicarlos a una foto concreta.|Cada acción puede ser un filtro diferente.|
|Elegir la respuesta de un bot de chat para clarificar la intención del usuario o sugerir una acción.|Cada acción es una opción de cómo interpretar la respuesta.|
|Elegir lo que se va a mostrar al principio de una lista de resultados de búsqueda|Cada acción es uno de los primeros resultados de la búsqueda.|


### <a name="examples-of-features-for-actions"></a>Ejemplos de características para acciones

Los siguientes son buenos ejemplos de características para acciones. Dependen mucho de cada aplicación.

* Características con las particularidades de las acciones. Por ejemplo, ¿es una película o una serie de televisión?
* Características acerca de cómo pueden haber interactuado los usuarios con esta acción en el pasado. Por ejemplo, esta película la ven mayoritariamente personas encuadradas en los grupos demográficos A o B, y normalmente no se reproduce más de una vez.
* Características sobre las particularidades de cómo el usuario *ve* las acciones. Por ejemplo, ¿incluye el cartel de la película que se muestra en la miniatura caras, automóviles o paisajes?

### <a name="load-actions-from-the-client-application"></a>Carga de acciones desde la aplicación cliente

Las características de las acciones normalmente pueden proceder de sistemas de administración de contenido, catálogos y sistemas de recomendación. La aplicación es responsable de cargar la información acerca de las acciones de las bases de datos y sistemas relevantes que tenga. Si las acciones no cambian o se cargan cada que hay un impacto innecesario en el rendimiento, puede agregar lógica a la aplicación para almacenar esta información en la caché.

### <a name="prevent-actions-from-being-ranked"></a>Evitar que se clasifiquen las acciones

En algunos casos, hay acciones que no desea que se muestren a los usuarios. La mejor forma de evitar que una acción se clasifique en el nivel máximo es no incluirla en la lista de acciones para la API Rank en primer lugar.

En algunos casos, solo se puede determinar más adelante en la lógica de negocios si una _acción_ resultante de una llamada de API Rank debe mostrarse a un usuario. Para estos casos, debe usar _Eventos inactivos_.

## <a name="json-format-for-actions"></a>Formato JSON para las acciones

Al llamar a Rank, enviará varias acciones entre las que elegir:

Los objetos JSON pueden incluir objetos JSON anidados y propiedades y valores simples. Solo se puede incluir una matriz si los elementos de la matriz son números. 

```json
{
    "actions": [
    {
      "id": "pasta",
      "features": [
        {
          "taste": "salty",
          "spiceLevel": "medium",
          "grams": [400,800]
        },
        {
          "nutritionLevel": 5,
          "cuisine": "italian"
        }
      ]
    },
    {
      "id": "ice cream",
      "features": [
        {
          "taste": "sweet",
          "spiceLevel": "none",
          "grams": [150, 300, 450]
        },
        {
          "nutritionalLevel": 2
        }
      ]
    },
    {
      "id": "juice",
      "features": [
        {
          "taste": "sweet",
          "spiceLevel": "none",
          "grams": [300, 600, 900]
        },
        {
          "nutritionLevel": 5
        },
        {
          "drink": true
        }
      ]
    },
    {
      "id": "salad",
      "features": [
        {
          "taste": "salty",
          "spiceLevel": "low",
          "grams": [300, 600]
        },
        {
          "nutritionLevel": 8
        }
      ]
    }
  ]
}
```

## <a name="examples-of-context-information"></a>Ejemplos de información de contexto

La información del _contexto_ depende de cada aplicación y caso de uso, pero habitualmente pueden incluir información como:

* Información demográfica y de perfil del usuario.
* Información extraída de los encabezados HTTP, como el agente de usuario, o derivada de la información de HTTP, como las búsquedas geográficas inversas basadas en direcciones IP.
* Información acerca de la hora actual, como el día de la semana, si es fin de semana o no, si por la mañana o por la tarde, si son vacaciones de Navidad o no, etc.
* Información extraída de aplicaciones para dispositivos móviles, como la ubicación, el movimiento o el nivel de batería.
* Elementos agregados al historial relativos al comportamiento de los usuarios (como cuáles son los géneros cinematográficos que más veces ha visto el usuario).

La aplicación es responsable de cargar la información acerca del contexto de las bases de datos, los sensores y los sistemas relevantes que pueda tener. Si la información de contexto no cambia, puede agregar la lógica de la aplicación para almacenar esta información en la caché antes de enviarla a la API Rank.

## <a name="json-format-for-context"></a>Formato JSON del contexto 

El contexto se expresa en forma de objeto JSON que se envía a la API Rank:

Los objetos JSON pueden incluir objetos JSON anidados y propiedades y valores simples. Solo se puede incluir una matriz si los elementos de la matriz son números. 

```JSON
{
    "contextFeatures": [
        { 
            "user": {
                "name":"Doug"
            }
        },
        {
            "state": {
                "timeOfDay": "noon",
                "weather": "sunny"
            }
        },
        {
            "device": {
                "mobile":true,
                "Windows":true,
                "screensize": [1680,1050]
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>Pasos siguientes

[Aprendizaje de refuerzo](concepts-reinforcement-learning.md) 
