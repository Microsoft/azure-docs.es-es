---
author: baanders
description: incluir archivo para operaciones de consulta de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 7/28/2020
ms.author: baanders
ms.openlocfilehash: 6dfddd9179569075abdbd94b6c1afa719db9ba1e
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87905605"
---
## <a name="query-language-features"></a>Características del lenguaje de consulta

Azure Digital Twins proporciona completas funciones de consulta en el grafo de gemelos. Las consultas se describen mediante una sintaxis de tipo SQL, en un lenguaje de consulta similar al [lenguaje de consulta de IoT Hub](../articles/iot-hub/iot-hub-devguide-query-language.md) con muchas características parecidas.

> [!NOTE]
> Todas las operaciones de consulta de Azure Digital Twins distinguen mayúsculas de minúsculas.

Estas son las operaciones disponibles en el lenguaje de consulta de Azure Digital Twins.

Obtención de gemelos digitales por…
* modelo (con el operador `IS_OF_MODEL`)
* propiedades (incluidas [propiedades de etiqueta](../articles/digital-twins/how-to-use-tags.md))
* interfaces
* relationships
  - propiedades de las relaciones

Puede mejorar aún más las consultas con las siguientes operaciones:
* Obtención de gemelos en varios tipos de relación (consultas `JOIN`). 
  - Durante la versión preliminar, se permiten hasta cinco niveles de `JOIN`.
* Selección de solo los principales resultados de la consulta (operador `Select TOP`)
* Uso de funciones escalares: `IS_BOOL`, `IS_DEFINED`, `IS_NULL`, `IS_NUMBER`, `IS_OBJECT`, `IS_PRIMITIVE`, `IS_STRING`, `STARTSWITH`, `ENDSWITH`.
* Uso de operadores de comparación de consulta: `IN`/`NIN`, `=`, `!=`, `<`, `>`, `<=` y `>=`.
* Uso de cualquier combinación (operador `AND`, `OR`, `NOT`) de `IS_OF_MODEL`, funciones escalares y operadores de comparación.
* Uso de la continuación: se crea una instancia del objeto de consulta con un tamaño de páginas (hasta 100). Puede recuperar de página en página los gemelos digitales. Para ello, proporcione el token de continuación en las llamadas subsiguientes a la API.