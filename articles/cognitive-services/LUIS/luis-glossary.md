---
title: 'Glosario: LUIS'
description: En el glosario se explican los términos que puede encontrar cuando use el servicio de API de LUIS.
ms.topic: reference
ms.date: 05/08/2020
ms.openlocfilehash: 44749648559994d986e4d84b06f38369850de6ca
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836418"
---
# <a name="language-understanding-glossary-of-common-vocabulary-and-concepts"></a>Glosario de Language Understanding de conceptos y vocabulario común
En el glosario de Language Understanding (LUIS) se explican los términos que pueden aparecer cuando se trabaja con el servicio LUIS.

## <a name="active-version"></a>Versión activa

La versión activa es la [versión](luis-how-to-manage-versions.md) de la aplicación que se actualiza cuando se realizan cambios en el modelo mediante el portal de LUIS. En el portal de LUIS, si quiere realizar cambios en una versión que no sea la versión activa, primero debe establecer esa versión como activa.

## <a name="active-learning"></a>Aprendizaje activo

El aprendizaje activo es una técnica de aprendizaje automático en la que se usa el modelo con aprendizaje automático para identificar nuevos ejemplos informativos para etiquetarlos. En LUIS, el aprendizaje activo hace referencia a la adición de expresiones del tráfico del punto de conexión cuyas predicciones actuales no son claras para mejorar el modelo. Haga clic en "Revisar las expresiones de punto de conexión" para ver las expresiones que se van a etiquetar.

Consulte también:
* [Información conceptual](luis-concept-review-endpoint-utterances.md)
* [Tutorial de revisión de las expresiones del punto de conexión](luis-tutorial-review-endpoint-utterances.md)
* Procedimientos para mejorar la aplicación LUIS mediante la [revisión de las expresiones del punto de conexión](luis-how-to-review-endpoint-utterances.md)

## <a name="application-app"></a>Aplicación

En LUIS, su aplicación es una colección de modelos con aprendizaje automático, basados en el mismo conjunto de datos, que funciona conjuntamente para predecir las intenciones y las entidades de un escenario determinado. Cada aplicación tiene un punto de conexión de predicción independiente.

Si compila un bot de recursos humanos, puede haber un conjunto de intenciones, como "programar el tiempo de descanso", "consultar los beneficios" y "actualizar la información personal" y entidades para cada una de las intenciones que se agrupan en una sola aplicación.

## <a name="authoring"></a>Creación

La creación es la capacidad de crear, administrar e implementar una aplicación LUIS, ya sea mediante el portal de LUIS o las API de creación.

### <a name="authoring-key"></a>Clave de creación

La [clave de creación](luis-how-to-azure-subscription.md) se usa para crear la aplicación. No se usa para enviar consultas de punto de conexión de nivel de producción. Para obtener más información, vea [Key limits](luis-limits.md#key-limits) (Límites de la clave).

### <a name="authoring-resource"></a>Recurso de creación

El [recurso de creación](luis-how-to-azure-subscription.md#azure-resources-for-luis) de LUIS es un elemento administrable que está disponible a través de Azure. El recurso es el acceso a las capacidades de creación, aprendizaje y publicación asociadas del servicio de Azure. El recurso incluye la información de autenticación, autorización y seguridad que necesita para acceder al servicio asociado de Azure.

El recurso de creación tiene una "clase" de Azure de `LUIS-Authoring`.

## <a name="batch-test"></a>Pruebas en lote

Las pruebas por lotes es la capacidad de validar los modelos de una aplicación LUIS actual con un conjunto de pruebas coherentes y conocidas de expresiones del usuario. La prueba por lotes se define en un [archivo con formato JSON](luis-concept-batch-test.md#batch-file-format).

Consulte también:
* [Conceptos](luis-concept-batch-test.md)
* [Procedimientos](luis-how-to-batch-test.md) para ejecutar una prueba por lotes
* [Tutorial](luis-tutorial-batch-testing.md): Creación y ejecución de una prueba por lotes

### <a name="f-measure"></a>Medida F

En las pruebas por lotes, una medida de la precisión de la prueba.

### <a name="false-negative-fn"></a>Falso negativo (FN)

En las pruebas por lotes, los puntos de datos representan expresiones en las que la aplicación predice incorrectamente la ausencia de la intención o entidad de destino.

### <a name="false-positive-fp"></a>Falso positivo (FP)

En las pruebas por lotes, los puntos de datos representan expresiones en las que la aplicación predice incorrectamente la existencia de la intención o entidad de destino.

### <a name="precision"></a>Precision
En las pruebas por lotes, la precisión (también conocida como el valor de predicción positivo) es la fracción de las expresiones pertinentes entre las expresiones recuperadas.

Un ejemplo para una prueba por lotes de animales es el número de ovejas predicho dividido por el número total de animales (ovejas y los que no son ovejas).

### <a name="recall"></a>Recuperación

En las pruebas por lotes, la recuperación (también conocida como sensibilidad), es la capacidad de generalizar de LUIS.

Un ejemplo para una prueba por lotes de animales es el número de ovejas predicho dividido por el número total de ovejas disponibles.

### <a name="true-negative-tn"></a>Verdadero negativo (TN)

Un verdadero negativo es cuando la aplicación predice correctamente que no hay coincidencia. En las pruebas por lotes, se produce un verdadero negativo cuando la aplicación predice una intención o entidad para un ejemplo que no se ha etiquetado con esa intención o entidad.

### <a name="true-positive-tp"></a>Verdadero positivo (TP)

Un verdadero positivo es cuando la aplicación predice correctamente una coincidencia. En las pruebas por lotes, se produce un verdadero positivo cuando la aplicación predice una intención o entidad para un ejemplo que se ha etiquetado con esa intención o entidad.

## <a name="classifier"></a>Clasificador

Un clasificador es un modelo con aprendizaje automático que predice en qué categoría o clase encaja una entrada.

Una [intención](#intent) es un ejemplo de un clasificador.

## <a name="collaborator"></a>Colaborador

Un colaborador de este tipo es conceptualmente lo mismo que un [colaborador](#contributor). A un colaborador se le concede acceso cuando un propietario agrega la dirección de correo electrónico del colaborador a una aplicación que no está controlada con el acceso basado en roles (RBAC de Azure). Si todavía usa colaboradores, debe migrar su cuenta de LUIS y usar recursos de creación de LUIS para administrar los colaboradores con RBAC.

## <a name="contributor"></a>Colaborador

Un colaborador no es el [propietario](#owner) de la aplicación, pero tiene los mismos permisos para agregar, editar y eliminar intenciones, entidades y expresiones. Un colaborador proporciona control de acceso basado en roles de Azure (RBAC de Azure) a una aplicación de LUIS.

Consulte también:
* [Procedimientos](luis-how-to-collaborate.md#add-contributor-to-azure-authoring-resource) para agregar colaboradores

## <a name="descriptor"></a>Descriptor

Un descriptor es el término que se usaba anteriormente para una [característica](#features) de aprendizaje automático.

## <a name="domain"></a>Domain

En el contexto de LUIS, un dominio es un área de conocimiento. El dominio es específico del escenario. Los diferentes dominios usan un lenguaje y terminología específicos, que tienen significado en el contexto del dominio. Por ejemplo, si compila una aplicación para reproducir música, la aplicación tendrá términos y lenguaje específicos de la música: palabras como "canción, pista, álbum, letra, cara b, artista". Para obtener ejemplos de dominios, consulte los [dominios precompilados](#prebuilt-domain).

## <a name="endpoint"></a>Punto de conexión

### <a name="authoring-endpoint"></a>Punto de conexión de creación

La dirección URL del punto de conexión de creación de LUIS es donde se crea, entrena y publica la aplicación. La dirección URL del punto de conexión contiene la región o el subdominio personalizados de la aplicación publicada, así como el id. de la aplicación.

Para más información sobre la creación de una aplicación mediante programación, consulte la [referencia para desarrolladores](developer-reference-resource.md#rest-endpoints)

### <a name="prediction-endpoint"></a>Punto de conexión de predicción

La dirección URL del punto de conexión de predicción de LUIS es donde se envían las consultas de LUIS después de crear y publicar la [aplicación LUIS](#application-app). La dirección URL del punto de conexión contiene la región o el subdominio personalizados de la aplicación publicada, así como el id. de la aplicación. Puede encontrar el punto de conexión en la página **[Recursos de Azure](luis-how-to-azure-subscription.md)** de la aplicación o bien puede obtener la dirección URL del punto de conexión desde la API [Get App Info](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c37).

El acceso al punto de conexión de predicción se autoriza mediante la clave de predicción de LUIS.

## <a name="entity"></a>Entidad

Las [entidades](luis-concept-entity-types.md) son palabras de las expresiones que describen la información usada para cumplimentar o identificar una intención. Si la entidad es compleja y quiere que el modelo identifique determinadas partes, puede dividir el modelo en subentidades. Por ejemplo, si quiere que el modelo prediga una dirección, pero también las subentidades de calle, ciudad, estado y código postal. Las entidades también se pueden usar como características para modelos. La respuesta de la aplicación LUIS incluirá tanto las intenciones predichas como todas las entidades.

### <a name="entity-extractor"></a>Extractor de entidades

Un extractor de entidades, a veces conocido solo como extractor, es el tipo de modelo con aprendizaje automático que LUIS usa para predecir entidades.

### <a name="entity-schema"></a>Esquema de entidad

El esquema de entidad es la estructura que se define para las entidades con aprendizaje automático con subentidades. El punto de conexión de predicción devuelve todas las entidades y subentidades extraídas definidas en el esquema.

### <a name="entitys-subentity"></a>Subentidad de entidad

Una subentidad es una entidad secundaria de una entidad con aprendizaje automático.

### <a name="non-machine-learning-entity"></a>Entidad que no es de aprendizaje automático

Una entidad que usa la coincidencia de texto para extraer datos:
* Entidad de lista
* Entidad de expresión regular

### <a name="list-entity"></a>Entidad de lista

Una [entidad de lista](reference-entity-list.md) representa un conjunto fijo y cerrado de palabras relacionadas y sus sinónimos. Las entidades de lista son coincidencias exactas, a diferencia de las entidades con aprendizaje automático.

La entidad se predecirá si una palabra de la entidad de lista se incluye en la lista. Por ejemplo, si tiene una entidad de lista llamada "tamaño" y tiene las palabras "pequeño, mediano, grande" en la lista, la entidad de tamaño se predecirá para todas las expresiones en las que se usen las palabras "pequeño", "medio" o "grande", independientemente del contexto.

### <a name="regular-expression"></a>Expresión regular

Una [entidad de expresión regular](reference-entity-regular-expression.md) representa una expresión regular. Las entidades de expresión regular son coincidencias exactas, a diferencia de las entidades con aprendizaje automático.
### <a name="prebuilt-entity"></a>Entidad creada previamente

Vea la entrada [entidad precompilada](#prebuilt-entity) del modelo precompilado

## <a name="features"></a>Características

En el aprendizaje automático, es una característica que ayuda al modelo a reconocer un concepto determinado. Es una sugerencia que LUIS puede usar, pero no es una regla rígida.

Este término también se conoce como **[característica con aprendizaje automático](luis-concept-feature.md)** .

Estas sugerencias se usan en combinación con las etiquetas para aprender a predecir nuevos datos. LUIS es compatible tanto con las listas de frases como con el uso de otros modelos como características.

### <a name="required-feature"></a>Característica obligatoria

Una característica obligatoria es una manera de restringir la salida de un modelo LUIS. Cuando una característica de una entidad se marca como obligatoria, debe estar presente en el ejemplo de la entidad que se va a predecir, independientemente de lo que prediga el modelo con aprendizaje automático.

Considere un ejemplo en el que tiene una característica de número precompilado que ha marcado como obligatorio en la entidad de cantidad para un bot de pedido de menús. Cuando el bot ve `I want a bajillion large pizzas?`, bajillion no se predice como una cantidad, independientemente del contexto en el que aparezca. Bajillion no es un número válido y el número de entidad precompilada no lo predecirá.

## <a name="intent"></a>Intención

Una [intención](luis-concept-intent.md) representa una tarea o acción que el usuario quiere realizar. Es un propósito u objetivo expresado en la entrada de un usuario, como reservar un vuelo o pagar una factura. En LUIS, una expresión en su conjunto se clasifica como una intención, pero las partes de la expresión se extraen como entidades.

## <a name="labeling-examples"></a>Ejemplos de etiquetado

Etiquetar, o marcar, es el proceso de asociar un ejemplo positivo o negativo a un modelo.

### <a name="labeling-for-intents"></a>Etiquetado para intenciones
En LUIS, las intenciones dentro de una aplicación son mutuamente excluyentes. Esto significa que cuando se agrega una expresión a una intención, se considera un ejemplo _positivo_ para esa intención y un ejemplo _negativo_ para todas las demás intenciones. Los ejemplos negativos no se deben confundir con la intención "Ninguno", que representa las expresiones que están fuera del ámbito de la aplicación.

### <a name="labeling-for-entities"></a>Etiquetado para entidades
En LUIS, se [etiqueta](label-entity-example-utterance.md) una palabra o frase en la expresión de ejemplo de una intención con una entidad como un ejemplo _positivo_. El etiquetado muestra la intención de lo que se debe predecir para esa expresión. Las expresiones etiquetadas se usan para entrenar la intención.

## <a name="luis-app"></a>Aplicación de LUIS

Consulte la definición de [aplicación](#application-app).

## <a name="model"></a>Modelo

Un modelo (con aprendizaje automático) es una función que realiza una predicción sobre los datos de entrada. En LUIS, hacemos referencia a los clasificadores de intenciones y a los extractores de entidades genéricamente como "modelos", y hacemos referencia a una colección de modelos que se entrenan, publican y consultan juntos como "aplicación".

## <a name="normalized-value"></a>Valor normalizado

Se agregan valores a las entidades de [lista](#list-entity). Cada uno de esos valores puede tener una lista de uno o más sinónimos. En la respuesta solo se devuelve el valor normalizado.

## <a name="overfitting"></a>Sobreajuste

El sobreajuste se produce cuando el modelo se fija solo en los ejemplos específicos y no puede generalizarse bien.

## <a name="owner"></a>Propietario

Cada aplicación tiene un propietario, que es la persona que la ha creado. El propietario administra los permisos para la aplicación en Azure Portal.

## <a name="phrase-list"></a>Lista de frases

Una [lista de frases](luis-concept-feature.md) es un tipo específico de característica de aprendizaje automático que incluye un grupo de valores (palabras o frases) que pertenecen a la misma clase y que se deben tratar de forma similar (por ejemplo, nombres de ciudades o productos).

## <a name="prebuilt-model"></a>Modelo precompilado

Un [modelo precompilado](luis-concept-prebuilt-model.md) es una intención, una entidad o una colección de ambas, junto con ejemplos con etiquetas. Estos modelos precompilados comunes se pueden agregar a la aplicación para reducir el trabajo de desarrollo del modelo necesario para la aplicación.

### <a name="prebuilt-domain"></a>Dominio creado previamente

Un dominio precompilado es una aplicación LUIS configurada para un dominio específico, como la automatización del hogar (HomeAutomation) o las reservas de un restaurante (RestaurantReservation). Las intenciones, expresiones y entidades están configuradas para este dominio.

### <a name="prebuilt-entity"></a>Entidad creada previamente

Una entidad precompilada es una entidad que LUIS proporciona para los tipos comunes de información, por ejemplo, número, dirección URL y correo electrónico. Se crean a partir de datos públicos. Puede agregar una entidad precompilada como una entidad independiente o como una característica a una entidad.

### <a name="prebuilt-intent"></a>Intención precompilada

Una intención precompilada es una intención que LUIS proporciona para tipos comunes de información y que incluye sus propias expresiones de ejemplo con etiquetas.

## <a name="prediction"></a>Predicción

Una predicción es una solicitud de REST al servicio de predicción de LUIS de Azure que toma nuevos datos (expresión del usuario) y aplica la aplicación entrenada y publicada a esos datos para determinar qué intenciones y entidades se encuentran en ellos.

### <a name="prediction-key"></a>Clave de predicción

La [clave de predicción](luis-how-to-azure-subscription.md) (anteriormente conocida como clave de suscripción) es la clave asociada al servicio LUIS que creó en Azure y que autoriza el uso del punto de conexión de predicción.

No es la clave de creación. Si tiene una clave de punto de conexión de predicción, se debe usar para las solicitudes de punto de conexión en lugar de la clave de creación. Puede ver la clave de predicción actual en la dirección URL del punto de conexión, en la parte inferior de la página de recursos de Azure del sitio web de LUIS. Es el valor del par de nombre y valor de la clave de suscripción.

### <a name="prediction-resource"></a>Recurso de predicción

El recurso de predicción de LUIS es un elemento administrable que está disponible a través de Azure. El recurso es el acceso a la predicción asociada del servicio de Azure. El recurso incluye predicciones.

El recurso de predicción tiene una "clase" de Azure de `LUIS`.

### <a name="prediction-score"></a>Puntuación de predicción

La [puntuación](luis-concept-prediction-score.md) es un número entre 0 y 1, que es una medida de la confianza del sistema de que una expresión de entrada determinada coincide con una intención concreta. Una puntuación más cercana a 1 significa que el sistema está seguro de su salida y una puntuación más cercana a 0 significa que el sistema está seguro de que la entrada no coincide con una salida determinada. Las puntuaciones centrales significan que el sistema no está muy seguro de cómo tomar la decisión.

Por ejemplo, emplee un modelo que se usa para identificar si algún texto del cliente incluye un pedido de comida. Podría dar una puntuación de 1 para "me gustaría pedir un café" (el sistema está muy seguro de que se trata de un pedido) y una puntuación de 0 para "mi equipo ganó el partido anoche" (el sistema está muy seguro de que NO es un pedido). Y puede tener una puntuación de 0,5 para "vamos a tomar té" (no está seguro de si se trata de un pedido o no).

## <a name="programmatic-key"></a>Clave de programación

Ha cambiado de nombre, ahora es la [clave de creación](#authoring-key).

## <a name="publish"></a>Publicar

[Publicar](luis-how-to-publish-app.md) significa poner una versión activa de LUIS a disposición del [punto de conexión](#endpoint) del almacenamiento provisional o de producción.

## <a name="quota"></a>Quota

La cuota de LUIS es la limitación del nivel de suscripción de Azure. La cuota de LUIS puede verse limitada por las solicitudes por segundo (estado HTTP 429) y el total de solicitudes en un mes (estado HTTP 403).

## <a name="schema"></a>Schema

El esquema incluye las intenciones y entidades junto con las subentidades. Inicialmente, se planea el esquema para iterarse a lo largo del tiempo. El esquema no incluye la configuración, las características o las expresiones de ejemplo de la aplicación.

## <a name="sentiment-analysis"></a>Análisis de sentimiento
El análisis de sentimiento proporciona valores positivos o negativos de las expresiones que proporciona [Text Analytics](../text-analytics/overview.md).

## <a name="speech-priming"></a>Preparación para la voz

La preparación para la voz mejora el reconocimiento del texto oral y las frases que se usan habitualmente en el escenario con los [Servicios de voz](../speech-service/overview.md). En el caso de las aplicaciones con la preparación para la voz habilitada, se usan todos los ejemplos con etiquetas de LUIS para mejorar la precisión del reconocimiento de voz mediante la creación de un modelo de voz personalizado para esta aplicación específica. Por ejemplo, en una partida de ajedrez debe asegurarse de que, cuando el usuario diga "Mover alfil", no se interprete como "Mover marfil". La aplicación de LUIS debe incluir ejemplos en los que "alfil" se etiquete como una entidad.

## <a name="starter-key"></a>Clave de inicio

Una clave gratuita que se usará al empezar a usar LUIS por primera vez.

## <a name="synonyms"></a>Sinónimos

En las [entidades de lista](reference-entity-list.md) de LUIS, puede crear un valor normalizado, y cada uno puede tener una lista de sinónimos. Por ejemplo, cree una entidad de tamaño con valores normalizados de pequeño, mediano, grande y extragrande. Puede crear sinónimos para cada valor de la siguiente manera:

|Valor normalizado| Sinónimos|
|--|--|
|Pequeña| diminuto, 8 onzas|
|Media| normal, 12 onzas|
|grande| considerable, 16 onzas|
|Extragrande| enorme, 24 onzas|

El modelo devolverá el valor normalizado de la entidad cuando se vea cualquiera de los sinónimos en la entrada.

## <a name="test"></a>Prueba

La [prueba](luis-concept-test.md) de una aplicación LUIS significa ver predicciones de modelo.

## <a name="timezone-offset"></a>Desplazamiento de zona horaria

El punto de conexión incluye [timezoneOffset](luis-concept-data-alteration.md#change-time-zone-of-prebuilt-datetimev2-entity). Este es el número de minutos que quiere agregar a la entidad datetimeV2 creada previamente, o bien quitar de esta. Por ejemplo, si la expresión es "¿Qué hora es?", el valor de datetimeV2 que se devuelve es la hora actual de la solicitud del cliente. Si la solicitud del cliente viene de un bot u otra aplicación que no es el mismo que el usuario de su bot, debe pasar el desplazamiento entre el bot y el usuario.

Vea [Cambiar la zona horaria de la entidad datetimeV2 creada previamente](luis-concept-data-alteration.md?#change-time-zone-of-prebuilt-datetimev2-entity).

## <a name="token"></a>Token
Un [token](luis-language-support.md#tokenization) es la unidad mínima de texto que LUIS puede reconocer. Es ligeramente diferente según el idioma.

Para el **inglés**, un token es un fragmento continuo (sin espacios ni puntuación) de letras y números. Un espacio NO es un token.

|Frase|Recuento de tokens|Explicación|
|--|--|--|
|`Dog`|1|Una sola palabra sin puntuación ni espacios.|
|`RMT33W`|1|Un número de localizador de registro. Puede contener números y letras, pero no tiene ningún signo de puntuación.|
|`425-555-5555`|5|Un número de teléfono. Cada signo de puntuación es un token único, por lo que `425-555-5555` sería 5 tokens:<br>`425`<br>`-`<br>`555`<br>`-`<br>`5555` |
|`https://luis.ai`|7|`https`<br>`:`<br>`/`<br>`/`<br>`luis`<br>`.`<br>`ai`<br>|

## <a name="train"></a>Train

[Entrenar](luis-how-to-train.md) es el proceso de enseñar a LUIS los cambios de la versión activa desde el último aprendizaje.

### <a name="training-data"></a>Datos de aprendizaje.

Los datos de aprendizaje son el conjunto de información que se necesita para entrenar un modelo. Esto incluye el esquema, las expresiones con etiquetas, las características y la configuración de la aplicación.

### <a name="training-errors"></a>Errores de aprendizaje

Los errores de aprendizaje son predicciones en los datos de aprendizaje que no coinciden con sus etiquetas.

## <a name="utterance"></a>Expresión

Una [expresión](luis-concept-utterance.md) es una entrada de usuario de un texto corto representativo de una oración en una conversación. Es una frase en lenguaje natural, como "reserva 2 billetes a Seattle para el próximo martes". Se agregan expresiones de ejemplo para enseñar al modelo y el modelo realiza predicciones en una nueva expresión en tiempo de ejecución.

## <a name="version"></a>Versión

Una [versión](luis-how-to-manage-versions.md) de LUIS es una instancia específica de una aplicación LUIS asociada a un id. de aplicación de LUIS y el punto de conexión publicado. Todas las aplicaciones de LUIS tienen al menos una versión.
