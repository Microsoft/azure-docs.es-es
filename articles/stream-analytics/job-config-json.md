---
title: Campos del archivo JobConfig.json de Azure Stream Analytics
description: En este artículo se enumeran los campos admitidos del archivo JobConfig.json de Azure Stream Analytics que se usan para crear trabajos en Visual Studio Code.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 02/14/2020
ms.openlocfilehash: f2dd759203655746601699f665436c78ee0758f8
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90885503"
---
# <a name="azure-stream-analytics-jobconfigjson-fields"></a>Campos del archivo JobConfig.json de Azure Stream Analytics

Los campos siguientes se admiten en el archivo *JobConfig.json* y se usan para [crear trabajos de Azure Stream Analytics mediante Visual Studio Code](quick-create-visual-studio-code.md).

```json
{
    "DataLocale": "string",
    "OutputErrorPolicy": "string",
    "EventsLateArrivalMaxDelayInSeconds": "integer",
    "EventsOutOfOrderMaxDelayInSeconds": "integer",
    "EventsOutOfOrderPolicy": "string",
    "StreamingUnits": "integer",
    "CompatibilityLevel": "string",
    "UseSystemAssignedIdentity": "boolean",
    "GlobalStorage": {
        "AccountName": "string",
        "AccountKey": "string",
    },
    "DataSourceCredentialDomain": "string",
    "ScriptType": "string",
    "Tags": {}
}
```

|Nombre|Tipo|Obligatorio|Value|
|----|----|--------|-----|
|DataLocale|string|No|La configuración regional de los datos del trabajo de análisis del flujo de datos. El valor debe ser el nombre de un trabajo compatible. El valor predeterminado es "en-US" si no se ha especificado ninguno.|
|OutputErrorPolicy|string|No|Indica la directiva que se aplica a los eventos que llegan a la salida y no se pueden escribir en el almacenamiento externo porque están mal formados (faltan valores de las columnas o el tamaño o tipo de los valores de las columnas no son correctos). -Detener o eliminar|
|EventsLateArrivalMaxDelayInSeconds|integer|No|El retraso tolerable máximo, en segundos, que pueden tener los eventos que llegan tarde para ser incluidos. El rango admitido es -1 a 1814399 (20.23:59:59 días) y-1 se usa para especificar que la espera es indefinida. Si la propiedad no está presente, se interpreta que tiene el valor -1.|
|EventsOutOfOrderMaxDelayInSeconds|integer|No|El retraso máximo tolerable, en segundos, en que se pueden ajustar los eventos que no funcionan para que vuelvan a estar en orden.|
|EventsOutOfOrderPolicy|string|No|Indica la directiva que aplicar a los eventos que llegan desordenados en el flujo de eventos de entrada. - Ajustar o eliminar|
|StreamingUnits|integer|Sí|Especifica el número de unidades de streaming que utiliza el trabajo de streaming.|
|CompatibilityLevel|string|No|Controla ciertos comportamientos del runtime del trabajo de streaming. - Los valores aceptables son "1.0", "1.1", "1.2"|
|UseSystemAssignedIdentity|boolean|No|Seleccione true para que este trabajo pueda comunicarse como tal con otros servicios de Azure mediante una identidad administrada de Azure Active Directory.|
|GlobalStorage.AccountName|string|No|La cuenta de almacenamiento global se utiliza para almacenar contenido relacionado con su trabajo de Stream Analytics, como instantáneas de datos de referencia de SQL.|
|GlobalStorage.AccountKey|string|No|Clave correspondiente de la cuenta de almacenamiento global.|
|DataSourceCredentialDomain|string|No|Propiedad reservada para el almacenamiento local de credenciales.|
|ScriptType|string|Sí|Propiedad reservada para indicar el tipo de este archivo de origen. El valor aceptable es "JobConfig" para JobConfig.json.|
|Etiquetas|Pares de clave-valor de JSON|No|Las etiquetas son pares nombre-valor que permiten categorizar los recursos y ver una facturación consolidada mediante la aplicación de la misma etiqueta en varios recursos y grupos de recursos. Los nombres de las etiquetas no distinguen mayúsculas de minúsculas, mientras que sus valores distinguen mayúsculas de minúsculas.|

## <a name="next-steps"></a>Pasos siguientes

* [Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-visual-studio-code.md)
* [Prueba de consultas de Stream Analytics localmente con datos de ejemplo mediante Visual Studio Code](visual-studio-code-local-run.md)
* [Prueba de las consultas de Stream Analytics localmente en una entrada de streaming en vivo mediante Visual Studio Code](visual-studio-code-local-run-live-input.md)
*[Implementación de un trabajo de Azure Stream Analytics mediante el paquete npm de CI/CD](setup-cicd-vs-code.md)