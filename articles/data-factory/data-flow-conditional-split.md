---
title: Transformación División condicional de Azure Data Factory Mapping Data Flow
description: Transformación División condicional de Azure Data Factory Data Flow
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/03/2019
ms.openlocfilehash: f9fd346d4c4eaed0797d564fe52dd44e9f0e240a
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65795633"
---
# <a name="mapping-data-flow-conditional-split-transformation"></a>Asignación de flujo de datos condicional transformación División

[!INCLUDE [notes](../../includes/data-factory-data-flow-preview.md)]

![Conditional split del cuadro de herramientas](media/data-flow/conditionalsplit2.png "conditional split del cuadro de herramientas")

La transformación División condicional puede redirigir filas de datos a diferentes secuencias según el contenido de los datos. La implementación de la transformación División condicional es similar a una estructura de decisión CASE en un lenguaje de programación. La transformación evalúa las expresiones y, según los resultados, dirige la fila de datos a la secuencia especificada. Esta transformación también proporciona una salida predeterminada, por lo que si una fila no coincide con ninguna expresión, se dirige a la salida predeterminada.

![división condicional](media/data-flow/conditionalsplit1.png "opciones de división condicional")

## <a name="multiple-paths"></a>Varias rutas de acceso

Para agregar condiciones adicionales, seleccione "Agregar secuencia" en el panel de configuración de la parte inferior y haga clic en el cuadro de texto Generador de expresiones para crear la expresión.

![Conditional split multi](media/data-flow/conditionalsplit3.png "conditional split múltiple")

## <a name="next-steps"></a>Pasos siguientes

Transformaciones de flujo de datos común utilizadas con división condicional: [Únase a la transformación](data-flow-join.md), [Loopup transformación](data-flow-lookup.md), [Seleccionar transformación](data-flow-select.md)
