---
title: Prueba de DevOps para aplicaciones de LUIS
description: Cómo probar una aplicación de Language Understanding (LUIS) en un entorno de DevOps.
ms.topic: conceptual
ms.date: 06/3/2020
ms.openlocfilehash: 2556d2e904aff720bc02e4c7d58bf5a72af4d413
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86538078"
---
# <a name="testing-for-luis-devops"></a>Pruebas de DevOps de LUIS

Los ingenieros de software que desarrollan una aplicación de Language Understanding (LUIS) pueden aplicar prácticas de DevOps en torno al [control de código fuente](luis-concept-devops-sourcecontrol.md), las [compilaciones automatizadas](luis-concept-devops-automation.md), las [pruebas](luis-concept-devops-testing.md) y la [administración de versiones](luis-concept-devops-automation.md#release-management) según estas instrucciones.

En las metodologías de Agile Software Development, las pruebas desempeñan un papel integral en la creación de software de calidad. Cada cambio significativo en una aplicación de LUIS debe ir acompañado de pruebas diseñadas para probar la nueva funcionalidad que está creando el desarrollador en la aplicación. Estas pruebas se comprueban en el repositorio de código fuente junto con el archivo de código fuente `.lu` de la aplicación de LUIS. La implementación del cambio finaliza cuando la aplicación pasa satisfactoriamente las pruebas.

Las pruebas son una parte fundamental de los [flujos de trabajo de CI/CD](luis-concept-devops-automation.md). Cuando se proponen los cambios realizados en una aplicación de LUIS en una solicitud de incorporación de cambios (PR) o después de que los cambios se combinan en la rama principal, los flujos de trabajo de CI deben ejecutar las pruebas para comprobar que las actualizaciones no han causado ninguna regresión.

## <a name="how-to-do-unit-testing-and-batch-testing"></a>Cómo realizar pruebas unitarias y pruebas por lotes

En los flujos de trabajo de integración continua se deben realizar dos tipos diferentes de pruebas para una aplicación de LUIS:

- **Pruebas unitarias**: pruebas relativamente simples que comprueban la funcionalidad clave de la aplicación de LUIS. Una prueba unitaria se supera cuando se devuelven la intención esperada y las entidades esperadas para una expresión de prueba determinada. Se deben superar todas las pruebas unitarias para que la ejecución de prueba se complete correctamente.  
Este tipo de pruebas son similares a las [pruebas interactivas](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-test) que puede realizar en el [portal de LUIS](https://www.luis.ai/).

- **Pruebas por lotes**: las pruebas por lotes son una prueba completa sobre el modelo entrenado actual para medir su rendimiento. A diferencia de las pruebas unitarias, las pruebas por lotes no consisten en superar o no superar. La expectativa con las pruebas por lotes no es que todas las pruebas devuelvan la intención esperada y las entidades esperadas. En su lugar, una prueba por lotes ayuda a ver la precisión de cada intención y entidad de la aplicación y ayuda a comparar a lo largo del tiempo a medida que se realizan mejoras.  
Estas pruebas son del mismo tipo que las [pruebas por lotes](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-batch-test) que puede realizar de forma interactiva en el portal de LUIS.

Puede emplear pruebas unitarias desde el principio del proyecto. Las pruebas por lotes solo son realmente de valor una vez que se ha desarrollado el esquema de la aplicación de LUIS y se trabaja para mejorar su precisión.

En el caso de las pruebas unitarias y las pruebas por lotes, asegúrese de que las expresiones de prueba se mantienen separadas de las expresiones de entrenamiento. Si realiza pruebas sobre los mismos datos con los que entrena, obtendrá la falsa impresión de que la aplicación funciona bien cuando esta simplemente realiza un sobreajuste de los datos de prueba. Las pruebas deben ser invisibles para el modelo para probar si la generalización funciona bien.

### <a name="writing-tests"></a>Escritura de pruebas

Al escribir un conjunto de pruebas, para cada prueba debe definir:

* Probar expresión
* Intención esperada
* Entidades esperadas.

Use la [sintaxis de archivo por lotes](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-batch-test#batch-syntax-template-for-intents-with-entities) de LUIS para definir un grupo de pruebas en un archivo con formato JSON. Por ejemplo:

```JSON
[
  {
    "text": "example utterance goes here",
    "intent": "intent name goes here",
    "entities":
    [
        {
            "entity": "entity name 1 goes here",
            "startPos": 14,
            "endPos": 23
        },
        {
            "entity": "entity name 2 goes here",
            "startPos": 14,
            "endPos": 23
        }
    ]
  }
]
```

Algunas herramientas para pruebas, como [NLU.DevOps](https://github.com/microsoft/NLU.DevOps), también admiten archivos de pruebas con formato LUDown.

#### <a name="designing-unit-tests"></a>Diseño de pruebas unitarias

Las pruebas unitarias se deben diseñar para probar la funcionalidad básica de la aplicación de LUIS. En cada iteración (o sprint) del desarrollo de la aplicación, debe escribir un número suficiente de pruebas para comprobar que la funcionalidad clave que va a implementar en esa iteración funciona correctamente.

En cada prueba unitaria, para una expresión de prueba determinada, puede realizar lo siguiente:

* Probar que se devuelve la intención correcta
* Probar que se devuelven las entidades "clave": aquellas que son críticas para la solución.
* Probar que la [puntuación de predicción](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-prediction-score) de la intención y las entidades supera un umbral definido por el usuario. Por ejemplo, puede decidir que solo se considerará que se ha superado una prueba si la puntuación de predicción para la intención y las entidades clave supera 0,75.

En las pruebas unitarias, es una buena idea probar que se han devuelto las entidades clave en la respuesta de la predicción, pero para omitir los falsos positivos. Los *falsos positivos* son entidades que se encuentran en la respuesta de la predicción, pero que no se han definido en los resultados esperados de la prueba. Al omitir los falsos positivos es menos oneroso crear pruebas unitarias, a la vez que le permite centrarse en probar que se devuelven en la respuesta de la predicción los datos que resultan clave para la solución.

> [!TIP]
> La herramienta [NLU.DevOps](https://github.com/microsoft/NLU.DevOps) es compatible con todas las necesidades para pruebas de LUIS. Cuando se usa en [modo de prueba unitaria](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Analyze.md#unit-test-mode), el comando `compare` confirmará que se han superado todas las pruebas y omitirá los resultados con falsos positivos de entidades que no se han etiquetado en los resultados esperados.

#### <a name="designing-batch-tests"></a>Diseño de pruebas por lotes

Los conjuntos de pruebas por lotes deben contener un gran número de casos de prueba, diseñados para probar en todas las intenciones y todas las entidades de la aplicación de LUIS. Consulte [Pruebas por lotes en el portal de LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-batch-test) para obtener información sobre cómo definir un conjunto de pruebas por lotes.

### <a name="running-tests"></a>Ejecución de las pruebas

El portal de LUIS ofrece características para ayudar con las pruebas interactivas:

* Las [**pruebas interactivas**](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-test) le permiten enviar una expresión de ejemplo y obtener una respuesta de las intenciones y las entidades reconocidas por LUIS. Se comprueba que la prueba se ha realizado correctamente mediante una inspección visual.

* Las [**pruebas por lotes**](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-batch-test) utilizan un archivo de pruebas por lotes como entrada para validar la versión entrenada actual con el fin de medir la precisión de su predicción. Una prueba por lotes ayuda a ver la precisión de cada intención y entidad en la versión activa, mostrando los resultados en un gráfico.

#### <a name="running-tests-in-an-automated-build-workflow"></a>Ejecución de pruebas en un flujo de trabajo de compilación automatizada

Las características de pruebas interactivas del portal de LUIS son útiles, pero en el caso de DevOps, las pruebas automatizadas realizadas en un flujo de trabajo de CI/CD tienen ciertos requisitos:

* Las herramientas para pruebas se deben ejecutar en un paso del flujo de trabajo en un servidor de compilación. Esto significa que las herramientas deben ser capaces de ejecutarse en la línea de comandos.
* Las herramientas para pruebas deben ser capaces de ejecutar un grupo de pruebas en un punto de conexión y comprobar automáticamente los resultados esperados con respecto a los resultados reales.
* Si se produce un error en las pruebas, las herramientas para pruebas deben devolver un código de estado para detener el flujo de trabajo y "suspender la compilación".

LUIS no ofrece una herramienta de la línea de comandos ni una API de alto nivel que proporcione estas características. Se recomienda utilizar la herramienta [NLU.DevOps](https://github.com/microsoft/NLU.DevOps) para ejecutar las pruebas y comprobar los resultados, tanto en la línea de comandos como durante las pruebas automatizadas dentro de un flujo de trabajo de CI/CD.

Las funcionalidades de prueba que están disponibles en el portal de LUIS no requieren un punto de conexión publicado y forman parte de las funcionalidades de creación de LUIS. Cuando las pruebas se van a implementar en un flujo de trabajo de compilación automatizada, debe publicar la versión de la aplicación de LUIS que se va a probar en un punto de conexión para que las herramientas para pruebas como NLU.DevOps puedan enviar solicitudes de predicción como parte de las pruebas.

> [!TIP]
> * Si va a implementar su propia solución de pruebas y escribir código para enviar expresiones de prueba a un punto de conexión, recuerde que si usa la clave de creación de LUIS, la tasa de transacciones permitida se limita a 5 TPS. Limite la velocidad de envío o use una clave de predicción en su lugar.
> * Al enviar consultas de prueba a un punto de conexión, no olvide usar `log=false` en la cadena de consulta de la solicitud de predicción. Esto garantiza que las expresiones de prueba no se registran en LUIS ni se incluyen en la lista de revisión de expresiones del punto de conexión presentada por la característica de [aprendizaje activo](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-review-endpoint-utterances) de LUIS y, como resultado, se agreguen accidentalmente a las expresiones de entrenamiento de la aplicación.

#### <a name="running-unit-tests-at-the-command-line-and-in-cicd-workflows"></a>Ejecución de pruebas unitarias en la línea de comandos y en flujos de trabajo de CI/CD

Puede usar el paquete [NLU.DevOps](https://github.com/microsoft/NLU.DevOps) para ejecutar pruebas en la línea de comandos:

* Use el [comando test](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Test.md) de NLU.DevOps para enviar pruebas desde un archivo de pruebas a un punto de conexión y capturar los resultados de predicción reales en un archivo.
* Use el [comando compare](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Analyze.md) de NLU.DevOps para comparar los resultados reales con los resultados esperados definidos en el archivo de pruebas de entrada. El comando `compare` genera la salida de la prueba NUnit y, cuando se usa en [modo de prueba unitaria](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Analyze.md#unit-test-mode) mediante la marca `--unit-test`, confirmará que se han superado todas las pruebas.

### <a name="running-batch-tests-at-the-command-line-and-in-cicd-workflows"></a>Ejecución de pruebas por lotes en la línea de comandos y en flujos de trabajo de CI/CD

También puede usar el paquete NLU.DevOps para ejecutar pruebas por lotes en la línea de comandos.

* Use el [comando test](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Test.md) de NLU.DevOps para enviar pruebas desde un archivo de pruebas a un punto de conexión y capturar los resultados de predicción reales en un archivo, igual que en el caso de las pruebas unitarias.
* Use el [comando compare](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Analyze.md) de NLU.DevOps en [modo de prueba de rendimiento](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Analyze.md#performance-test-mode) para medir el rendimiento de la aplicación. También puede comparar el rendimiento de la aplicación con una prueba comparativa de rendimiento de línea base, por ejemplo, los resultados de la última confirmación en la versión principal o en la versión actual. En el modo de prueba de rendimiento, el comando `compare` genera la salida de la prueba NUnit y los [resultados de las pruebas por lotes](https://docs.microsoft.com/azure/cognitive-services/luis/luis-glossary#batch-test) en formato JSON.

## <a name="luis-non-deterministic-training-and-the-effect-on-testing"></a>Entrenamiento no determinista de LUIS y su efecto en las pruebas

Cuando LUIS entrena un modelo, como una intención, necesita tanto datos positivos, que son las expresiones de entrenamiento etiquetadas que ha proporcionado para entrenar la aplicación para el modelo, como datos negativos, que son aquellos datos que *no* son ejemplos válidos del uso de ese modelo. Durante el entrenamiento, LUIS crea los datos negativos de un modelo a partir de todos los datos positivos que ha proporcionado para los otros modelos, pero en algunos casos se puede producir un desequilibrio de datos. Para evitar este desequilibrio, LUIS muestrea un subconjunto de los datos negativos de un modo no determinista para su optimización para un conjunto de entrenamiento mejor equilibrado, un mejor rendimiento del modelo y un tiempo de entrenamiento más rápido.

El resultado de este entrenamiento no determinista es que puede obtener una [respuesta de predicción ligeramente distinta entre las distintas sesiones de entrenamiento](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-prediction-score), normalmente para las intenciones y entidades en las que la [puntuación de predicción](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-prediction-score) no es alta.

Si desea deshabilitar el entrenamiento no determinista para aquellas versiones de la aplicación de LUIS que se crean con el fin de realizar pruebas, use la [API de configuración de versiones](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/versions-update-application-version-settings) con la opción `UseAllTrainingData` establecida en `true`.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre la [implementación de flujos de trabajo de CI/CD](luis-concept-devops-automation.md).
* Obtenga información sobre cómo [implementar DevOps para LUIS con GitHub](luis-how-to-devops-with-github.md).