---
title: Características del aprendizaje automático con LUIS
description: Agregue características a un modelo de lenguaje para proporcionar sugerencias sobre cómo reconocer la entrada que quiera etiquetar o clasificar.
ms.topic: conceptual
ms.date: 06/10/2020
ms.openlocfilehash: 02a6fd27dbe22a40b29b47515edec5506d3b2075
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075168"
---
# <a name="machine-learning-features"></a>Características del aprendizaje automático

En el aprendizaje automático, una  *característica*  es un rasgo distintivo o un atributo de datos que el sistema observa y del que aprende.

Las características del aprendizaje automático proporcionan a LUIS indicaciones importantes sobre dónde buscar los elementos que distinguen un concepto. Son sugerencias que LUIS puede usar, pero no reglas rígidas. LUIS utiliza estas sugerencias en combinación con las etiquetas para buscar los datos.

Una característica se puede describir como una función, como f(x) = y. En la expresión de ejemplo, la característica indica dónde buscar el rasgo distintivo. Use esta información como ayuda para crear su esquema.

## <a name="types-of-features"></a>Tipos de características

LUIS admite listas de frases y modelos como características:

* Característica de lista de frases 
* Modelo (intención o entidad) como una característica

Las características deben considerarse como una parte necesaria del diseño del esquema.

## <a name="find-features-in-your-example-utterances"></a>Búsqueda de características en las expresiones de ejemplo

Dado que LUIS es una aplicación basada en lenguaje, las características se basan en texto. Seleccione el texto que indica el rasgo que quiere distinguir. En LUIS, la unidad más pequeña es el *token*. En el idioma inglés, un token es una extensión contigua, sin espacios ni puntuación, de letras y números.

Dado que los espacios y la puntuación no son tokens, céntrese en las pistas de texto que puede usar como características. Recuerde incluir variaciones de palabras, como:

* formas plurales
* tiempos verbales
* abreviaturas
* ortografías y errores ortográficos

Determine si el texto, ya que distingue un rasgo, tiene que:

* Coincidir con una palabra o frase exacta: considere la posibilidad de agregar una entidad de expresión regular o una entidad de lista como característica a la entidad o la intención.
* Coincidir con un concepto conocido como fechas, horas o nombres de personas: use una entidad precompilada como característica para la entidad o la intención.
* Aprender nuevos ejemplos con el tiempo: use una lista de frases de algunos ejemplos del concepto como característica para la entidad o la intención.

## <a name="combine-features"></a>Combinación de características

Puede usar más de una característica para describir un rasgo o concepto. Una combinación común consiste en usar una característica de la lista de frases y un tipo de entidad que se use a menudo como característica:

 * entidad precompilada
 * entidad de la expresión regular
 * entidad de lista

### <a name="ticket-booking-entity-example"></a>Ejemplo de entidad de reserva de billete

Como primer ejemplo, imagine una aplicación para reservar un vuelo con una intención de reserva de vuelo y una entidad de reserva de billete.

La entidad de reserva de billete es una entidad de aprendizaje automático para el destino del vuelo. Para ayudar a extraer la ubicación, use dos características para ayudar:

* Una lista de frases de palabras pertinentes, como **avión**, **vuelo**, **reserva** o **billete**.
* Una entidad precompilada **geographyV2** como característica para la entidad.

### <a name="pizza-entity-example"></a>Ejemplo de entidad de pizza

Como otro ejemplo, imagine una aplicación para pedir una pizza con una intención de creación de pedido de pizza y una entidad de pizza.

La entidad de pizza es una entidad de aprendizaje automático para los detalles de la pizza. Para ayudar a extraer los detalles, use dos características:

* Una lista de frases de palabras pertinentes, como **queso**, **masa**, **pepperoni** o **mozzarella**.
* Una entidad precompilada **number** como característica para la entidad.

## <a name="create-a-phrase-list-for-a-concept"></a>Creación de una lista de frases para un concepto

Una lista de frases es una lista de palabras o frases que describe un concepto. Una lista de frases se aplica como coincidencia sin distinción entre mayúsculas y minúsculas en el nivel de token.

Al agregar una lista de frases, puede establecer la característica como **[global](#global-features)** . Una característica global se aplica a toda la aplicación.

### <a name="when-to-use-a-phrase-list"></a>Cuándo usar una lista de frases

Use una lista de frases si es necesario que la aplicación LUIS generalice e identifique los elementos nuevos del concepto. Las listas de frases son similares a un vocabulario específico de un dominio. Mejoran la calidad de la comprensión de las intenciones y las entidades.

### <a name="how-to-use-a-phrase-list"></a>Cómo utilizar una lista de frases

Con una lista de frases, LUIS considera el contexto y lo generaliza para identificar los elementos que son similares, pero que no son una coincidencia de texto exacta. Siga estos pasos para utilizar una lista de frases:

1. Empezar con una entidad de aprendizaje automático:
    1. Agregue expresiones de ejemplo.
    1. Etiquete con una entidad de aprendizaje automático.
1. Agregar una lista de frases:
    1. Agregue palabras con un significado similar. No agregue todas las palabras o frases posibles. En su lugar, agregue solo algunas cada vez. A continuación, vuelva a entrenar y a publicar la aplicación.
    1. Revise y agregue palabras sugeridas.

### <a name="a-typical-scenario-for-a-phrase-list"></a>Escenario típico para una lista de frases

Un escenario típico para una lista de frases es potenciar las palabras relacionadas con una idea concreta.

Un buen ejemplo de palabras que pueden necesitar una lista de frases para definir mejor su significado son los términos médicos. Estos términos pueden tener un significado específico físico, químico, terapéutico o abstracto. LUIS no sabrá que los términos son importantes para el dominio del tema sin una lista de frases.

Para extraer términos médicos:

1. Cree expresiones de ejemplo y etiquete los términos médicos dentro de esas expresiones.
2. Cree una lista de frases con ejemplos de los términos dentro del dominio del tema. En esta lista de frases se debe incluir el término real que etiquetó y otros términos que describen el mismo concepto.
3. Agregue la lista de frases a la entidad o subentidad que extrae el concepto utilizado en la lista de frases. El escenario más común es un componente (secundario) de una entidad de aprendizaje automático. Si la lista de frases debe aplicarse a todos las intenciones o entidades, marque la lista de frases como lista de frases global. La marca **enabledForAllModels** controla este ámbito del modelo en la API.

### <a name="token-matches-for-a-phrase-list"></a>Coincidencias de token para una lista de frases

Una lista de frases se aplica siempre en el nivel de token. En la tabla siguiente se muestra cómo se aplica una lista de frases que contiene la palabra **Ann** a variaciones de los mismos caracteres en ese orden.


| Variación de token de **Ann** | Coincidencia de lista de frases cuando se encuentra el token |
|--------------------------|---------------------------------------|
| **ANN**<br>**aNN**<br>           | Sí: el token es **Ann**                  |
| **Ann's**                    | Sí: el token es **Ann**                  |
| **Anne**                     | No: el token es **Anne**                  |

<a name="how-to-use-phrase-lists"></a>
<a name="how-to-use-a-phrase-lists"></a>
<a name="phrase-lists-help-identify-simple-exchangeable-entities"></a>

## <a name="a-model-as-a-feature-helps-another-model"></a>Modelo como una característica que ayuda a otro modelo

Puede agregar un modelo (intención o entidad) como una característica para otro modelo (intención o entidad). Al agregar una intención o entidad existente como una característica, agrega un concepto bien definido con ejemplos con etiquetas.

Al agregar un modelo como una característica, puede establecer la característica como:
* **[Obligatoria](#required-features)** . Se deben encontrar las características que son obligatorias para que se devuelva el modelo desde el punto de conexión de predicción.
* **[Global](#global-features)** . Una característica global se aplica a toda la aplicación.

### <a name="when-to-use-an-entity-as-a-feature-to-an-intent"></a>Cuándo usar una entidad como una característica para una intención

Agregue una entidad como una característica a una intención cuando la detección de esa entidad sea significativa para la intención.

Por ejemplo, si la intención es reservar un vuelo, como **BookFlight**, y la entidad es la información sobre el billete (como el número de asiento, el origen y el destino), la búsqueda de la entidad de información del billete debe agregar un peso significativo a la predicción de la intención **BookFlight**.

### <a name="when-to-use-an-entity-as-a-feature-to-another-entity"></a>Cuándo usar una entidad como una característica para otra entidad

Una entidad (A) debe agregarse como una característica a otra entidad (B), cuando la detección de esa entidad (A) es importante para la predicción de la entidad (B).

Por ejemplo, si una entidad de dirección de envío contiene una subentidad de dirección postal, la búsqueda de la subentidad de dirección postal agrega peso significativo a la predicción para la entidad de dirección de envío.

* Dirección de envío (entidad de aprendizaje automático):

    * Número de la calle (subentidad)
    * Dirección postal (subentidad)
    * Ciudad (subentidad)
    * Estado o provincia (subentidad)
    * País o región (subentidad)
    * Código postal (subentidad)

## <a name="nested-subentities-with-features"></a>Subentidades anidadas con características

Una subentidad de aprendizaje automático indica que hay un concepto en la entidad primaria. El elemento primario puede ser otra subentidad o la entidad superior. El valor de la subentidad actúa como una característica de su elemento primario.

Una subentidad puede tener tanto una lista de frases como un modelo (otra entidad) como característica.

Cuando la subentidad tiene una lista de frases, aumentará el vocabulario del concepto, pero no agregará información a la respuesta JSON de la predicción.

Cuando la subentidad tiene una característica de otra entidad, la respuesta JSON incluye los datos extraídos de esa otra entidad.


## <a name="required-features"></a>Características obligatorias

Se deben encontrar las características que son obligatorias para que se devuelva el modelo desde el punto de conexión de predicción. Use una característica obligatoria si sabe que los datos entrantes deben coincidir con la característica.

Si el texto de la expresión no coincide con la característica requerida, no se extraerá.

Una característica obligatoria utiliza una entidad sin aprendizaje automático:

* Entidad de la expresión regular
* Entidad de lista
* Entidad creada previamente

Si está seguro de que se encontrará el modelo en los datos, establezca la característica como obligatoria. Una característica obligatoria no devuelve nada si no se encuentra.

Continuando con el ejemplo de la dirección de envío:

Dirección de envío (entidad con aprendizaje automático)

 * Número de la calle (subentidad) 
 * Dirección postal (subentidad) 
 * Nombre de la calle (subentidad) 
 * Ciudad (subentidad) 
 * Estado o provincia (subentidad) 
 * País o región (subentidad) 
 * Código postal (subentidad)

### <a name="required-feature-using-prebuilt-entities"></a>Característica obligatoria con entidades precompiladas

La ciudad, el estado y el país o región suelen ser un conjunto cerrado de listas, lo que significa que no cambian mucho con el tiempo. Estas entidades podrían tener las características recomendadas pertinentes y estas características podrían marcarse como obligatorias. Significa que no se devuelve la dirección de envío completa si no se encuentran las entidades con características obligatorias.

¿Qué ocurre si la ciudad, el estado o el país se encuentran en la expresión, pero en una ubicación o una jerga que LUIS no espera? Si desea proporcionar algún procesamiento posterior para ayudar a resolver la entidad, debido a una puntuación de confianza baja de LUIS, no marque la característica como obligatoria.

Otro ejemplo de una característica obligatoria para la dirección de envío es hacer que el número de la calle sea un número [precompilado](luis-reference-prebuilt-entities.md) obligatorio. Esto permite que el usuario pueda escribir "1 Microsoft Way" o "One Microsoft Way". Ambos se resolverán en el numeral "1" para la subentidad de número de la calle.

### <a name="required-feature-using-list-entities"></a>Característica obligatoria con entidades de lista

Una [entidad de lista](reference-entity-list.md) se utiliza como una lista de nombres canónicos junto con sus sinónimos. Como característica obligatoria, si la expresión no incluye el nombre canónico o un sinónimo, la entidad no se devuelve como parte del punto de conexión de predicción.

Supongamos que su empresa solo realiza envíos a un conjunto limitado de países o regiones. Puede crear una entidad de lista que incluya varias maneras en las que el cliente puede hacer referencia al país o región. Si LUIS no encuentra una coincidencia exacta en el texto de la expresión, la entidad (que tiene la característica obligatoria de la entidad de lista) no se devuelve en la predicción.

|Nombre canónico|Sinónimos|
|--|--|
|Estados Unidos|EE. UU.<br>EE. UU.<br>US<br>EE. UU.<br>0|

Una aplicación cliente, como un bot de chat, puede hacer una pregunta de seguimiento para ayudar. Esto ayuda al cliente a entender que la selección del país o región es limitada y *obligatoria*.

### <a name="required-feature-using-regular-expression-entities"></a>Característica obligatoria con entidades de expresión regular

Una [entidad de expresión regular](reference-entity-regular-expression.md) utilizada como característica obligatoria proporciona funciones de coincidencia de texto enriquecidas.

En el ejemplo de la dirección de envío, puede crear una expresión regular que capture las reglas de sintaxis de los códigos postales de los países o regiones.

## <a name="global-features"></a>Características globales

Aunque el uso más común es aplicar una característica a un modelo específico, puede configurar la característica como **característica global** para aplicarla a toda la aplicación.

El uso más común de una característica global es agregar un vocabulario adicional a la aplicación. Por ejemplo, si los clientes usan un idioma principal, pero se espera poder usar otro idioma en la misma expresión, puede agregar una característica que incluya palabras del idioma secundario.

Dado que el usuario espera usar el idioma secundario en cualquier intención o entidad, agregue palabras del idioma secundario a la lista de frases. Configure la lista de frases como una característica global.

## <a name="best-practices"></a>Procedimientos recomendados

Obtenga información sobre los [procedimientos recomendados](luis-concept-best-practices.md).

## <a name="next-steps"></a>Pasos siguientes

* [Extienda](schema-change-prediction-runtime.md) los modelos de aplicación en tiempo de ejecución de predicción.
* Vea [Agregar características](luis-how-to-add-features.md) para obtener más información sobre cómo agregar características a la aplicación de LUIS.
