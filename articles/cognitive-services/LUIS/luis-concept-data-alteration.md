---
title: Modificación de datos (LUIS)
description: Obtenga información sobre cómo se pueden modificar los datos antes de las predicciones en Language Understanding (LUIS)
ms.topic: conceptual
ms.date: 05/06/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: c675ac246f397b5949c870ad91ab936bbd92c7ef
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934674"
---
# <a name="alter-utterance-data-before-or-during-prediction"></a>Modificación de datos de expresiones antes o durante la predicción
LUIS proporciona distintos métodos para manipular la expresión antes o durante la predicción. Entre ellos se incluyen la [corrección de la ortografía](luis-tutorial-bing-spellcheck.md) y la solución de los problemas de la zona horaria para elementos [datetimeV2](luis-reference-prebuilt-datetimev2.md) creados previamente.

## <a name="correct-spelling-errors-in-utterance"></a>Corregir errores de ortografía de una expresión


### <a name="v3-runtime"></a>Runtime V3

Preprocese el texto por correcciones ortográficas antes de enviar la expresión a LUIS. Use expresiones de ejemplo con la ortografía correcta para asegurarse de obtener las predicciones correctas.

Utilice [Bing Spell Check](../bing-spell-check/overview.md) para corregir el texto antes de enviarlo a LUIS.

### <a name="prior-to-v3-runtime"></a>Antes de runtime V3

LUIS usa [Bing Spell Check API V7](../Bing-Spell-Check/overview.md) para corregir los errores de ortografía de la expresión. LUIS necesita la clave asociada a ese servicio. Cree la clave y agréguela como parámetro de cadena de consulta en el [punto de conexión](https://go.microsoft.com/fwlink/?linkid=2092356).

El punto de conexión requiere dos parámetros para que las correcciones ortográficas funcionen:

|Parámetro|Value|
|--|--|
|`spellCheck`|boolean|
|`bing-spell-check-subscription-key`|Clave de punto de conexión de [Bing Spell Check API V7](https://azure.microsoft.com/services/cognitive-services/spell-check/)|

Cuando [Bing Spell Check API V7](https://azure.microsoft.com/services/cognitive-services/spell-check/) detecta un error, se devuelven la expresión original y la expresión corregida junto con las predicciones del punto de conexión.

#### <a name="v2-prediction-endpoint-response"></a>[Respuesta de punto de conexión de predicción de V2](#tab/V2)

```JSON
{
  "query": "Book a flite to London?",
  "alteredQuery": "Book a flight to London?",
  "topScoringIntent": {
    "intent": "BookFlight",
    "score": 0.780123
  },
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[Respuesta de punto de conexión de predicción de V3](#tab/V3)

```JSON
{
    "query": "Book a flite to London?",
    "prediction": {
        "normalizedQuery": "book a flight to london?",
        "topIntent": "BookFlight",
        "intents": {
            "BookFlight": {
                "score": 0.780123
            }
        },
        "entities": {},
    }
}
```

* * *

### <a name="list-of-allowed-words"></a>Lista de palabras permitidas
Bing Spell Check API que se utiliza en LUIS no admite una lista de palabras para omitir durante las alteraciones de la corrección ortográfica. Si necesita admitir una lista de palabras o acrónimos, procese la expresión en la aplicación cliente antes de enviar la expresión a LUIS para la predicción de intenciones.

## <a name="change-time-zone-of-prebuilt-datetimev2-entity"></a>Cambiar la zona horaria de la entidad datetimeV2 creada previamente
Cuando una aplicación de LUIS usa la entidad [datetimeV2](luis-reference-prebuilt-datetimev2.md) creada previamente, se puede devolver un valor de fecha y hora en la respuesta de la predicción. La zona horaria de la solicitud se usa para determinar la fecha y hora correctas que se van a devolver. Si la solicitud procede de un bot o de otra aplicación centralizada antes de llegar a LUIS, corrija la zona horaria que usa LUIS.

### <a name="v3-prediction-api-to-alter-timezone"></a>API de predicción V3 para modificar la zona horaria

En la versión 3, `datetimeReference` determina el desplazamiento de la zona horaria. Más información sobre las [predicciones de la versión 3](luis-migration-api-v3.md#v3-post-body).

### <a name="v2-prediction-api-to-alter-timezone"></a>API de predicción V2 para modificar la zona horaria
La zona horaria se puede corregir agregando la zona horaria del usuario al punto de conexión mediante el parámetro `timezoneOffset` basado en la versión de la API. El valor del parámetro debe ser un número positivo o negativo (en minutos) para modificar la hora.

#### <a name="v2-prediction-daylight-savings-example"></a>Ejemplo de horario de verano de predicción V2
Si necesita que la entidad datetimeV2 creada previamente que se ha devuelto se ajuste al horario de verano, debe usar el parámetro de cadena de consulta con un valor +/- en minutos para la consulta del [punto de conexión](https://go.microsoft.com/fwlink/?linkid=2092356).

Agregar 60 minutos:

`https://{region}.api.cognitive.microsoft.com/luis/v2.0/apps/{appId}?q=Turn the lights on?timezoneOffset=60&verbose={boolean}&spellCheck={boolean}&staging={boolean}&bing-spell-check-subscription-key={string}&log={boolean}`

Quitar 60 minutos:

`https://{region}.api.cognitive.microsoft.com/luis/v2.0/apps/{appId}?q=Turn the lights on?timezoneOffset=-60&verbose={boolean}&spellCheck={boolean}&staging={boolean}&bing-spell-check-subscription-key={string}&log={boolean}`

#### <a name="v2-prediction-c-code-determines-correct-value-of-parameter"></a>El código en C# de la predicción de V2 determina el valor correcto del parámetro

El siguiente código de C# usa el método [FindSystemTimeZoneById](https://docs.microsoft.com/dotnet/api/system.timezoneinfo.findsystemtimezonebyid#examples) de la clase [TimeZoneInfo](https://docs.microsoft.com/dotnet/api/system.timezoneinfo) para determinar el valor de desplazamiento correcto según la hora del sistema:

```csharp
// Get CST zone id
TimeZoneInfo targetZone = TimeZoneInfo.FindSystemTimeZoneById("Central Standard Time");

// Get local machine's value of Now
DateTime utcDatetime = DateTime.UtcNow;

// Get Central Standard Time value of Now
DateTime cstDatetime = TimeZoneInfo.ConvertTimeFromUtc(utcDatetime, targetZone);

// Find timezoneOffset/datetimeReference
int offset = (int)((cstDatetime - utcDatetime).TotalMinutes);
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Corregir errores de ortografía con este tutorial](luis-tutorial-bing-spellcheck.md)
