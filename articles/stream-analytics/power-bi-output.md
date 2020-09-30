---
title: Salida de Power BI desde Azure Stream Analytics
description: En este artículo se describe cómo generar datos desde Azure Stream Analytics a Power BI.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: d398cfe063dbbb2bc87a3debf1669afa6a16b43e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90891995"
---
# <a name="power-bi-output-from-azure-stream-analytics"></a>Salida de Power BI desde Azure Stream Analytics

Puede usar [Power BI](https://powerbi.microsoft.com/) como salida para un trabajo de Stream Analytics a fin de ofrecer una amplia experiencia de visualización de los resultados del análisis. Puede usar esta funcionalidad con paneles operativos, generación de informes e informes basados en métricas.

La salida de Power BI desde Stream Analytics no está disponible actualmente en las regiones de Azure China (21Vianet) y Azure Alemania (T-Systems International).

## <a name="output-configuration"></a>Configuración de salida

En la siguiente tabla se encuentra una lista de nombres de propiedades y su descripción para configurar la salida de Power BI.

| Nombre de propiedad | Descripción |
| --- | --- |
| Alias de salida |Proporcione un nombre descriptivo que se usará en las consultas para dirigir la salida de la consulta a esta salida de Power BI. |
| Área de trabajo de grupo |Para habilitar el uso compartido de datos con otros usuarios de Power BI, puede seleccionar grupos dentro de su cuenta de Power BI o elegir **Mi área de trabajo** si no quiere escribir en un grupo. Actualizar un grupo existente requiere renovar la autenticación de Power BI. |
| Nombre del conjunto de datos |Proporcione el nombre de conjunto de datos que quiere que tenga la salida de Power BI. |
| Nombre de la tabla |Proporcione un nombre de tabla en el conjunto de datos de la salida de Power BI. Actualmente, la salida de Power BI de trabajos de Stream Analytics solo puede tener una tabla en un conjunto de datos. |
| Autorización de la conexión | Debe realizar la autorización con Power BI para configurar los valores de salida. Una vez que esta salida obtiene acceso a su panel de Power BI, puede revocarlo si cambia la contraseña de la cuenta de usuario, elimina la salida del trabajo o elimina el trabajo de Stream Analytics. | 

Para ver un tutorial sobre la configuración de una salida y panel de Power BI, consulte [Azure Stream Analytics y Power BI](stream-analytics-power-bi-dashboard.md).

> [!NOTE]
> No cree explícitamente el conjunto de datos y la tabla en el panel de Power BI. El conjunto de datos y la tabla se rellenan automáticamente cuando se inicia el trabajo y este comienza a enviar salidas a Power BI. Si la consulta de trabajo no genera ningún resultado, el conjunto de datos y la tabla no se crean. Si Power BI ya cuenta con un conjunto de datos y una tabla con el mismo nombre que el proporcionado en este trabajo de Stream Analytics, se sobrescriben los datos existentes.
>

### <a name="create-a-schema"></a>Creación de un esquema

Azure Stream Analytics crea un esquema de tabla y un conjunto de datos de Power BI para el usuario si todavía no existen. En todos los demás casos, la tabla se actualiza con los nuevos valores. Actualmente, solo puede existir una tabla dentro de un conjunto de datos. 

Power BI utiliza la directiva de retención primero en entrar, primero en salir (FIFO). Se recopilan los datos en una tabla hasta que alcance las 200 000 filas.

### <a name="convert-a-data-type-from-stream-analytics-to-power-bi"></a>Conversión de un tipo de datos de Stream Analytics a Power BI

Azure Stream Analytics actualiza el modelo de datos dinámicamente en tiempo de ejecución si cambia el esquema de salida. Se realiza un seguimiento de los cambios de nombre de columna, los cambios de tipo de columna y la adición o eliminación de columnas.

Esta tabla cubre las conversiones de tipos de datos de [tipos de datos de Stream Analytics](https://docs.microsoft.com/stream-analytics-query/data-types-azure-stream-analytics) a [tipos de Entity Data Model (EDM)](https://docs.microsoft.com/dotnet/framework/data/adonet/entity-data-model) de Power BI si no existen una tabla y un conjunto de datos de Power BI.

De Stream Analytics | A Power BI
-----|-----
bigint | Int64
nvarchar(max) | String
datetime | Datetime
FLOAT | Double
Matriz de registro | Tipo cadena, valor constante "IRecord" o "IArray"

### <a name="update-the-schema"></a>Revisión del esquema

Stream Analytics deduce el esquema de modelo de datos basándose en el primer conjunto de eventos en la salida. Más adelante, si es necesario, el esquema de modelo de datos se actualiza para dar cabida a los eventos entrantes que pueden coincidir con el esquema original.

Evite la consulta `SELECT *` para impedir la actualización de esquema dinámica en las filas. Además de las posibles implicaciones de rendimiento, podría dar lugar a incertidumbre respecto al tiempo necesario para los resultados. Seleccione los campos exactos que necesitan mostrarse en el panel de Power BI. Además, los valores de datos deben ser compatibles con el tipo de datos elegido.

Anterior o actual | Int64 | String | Datetime | Double
-----------------|-------|--------|----------|-------
Int64 | Int64 | String | String | Double
Double | Double | String | String | Double
String | String | String | String | String 
Datetime | String | String |  Datetime | String

## <a name="output-batch-size"></a>Tamaño de lote de salida

Para ver el tamaño del lote de salida, consulte los [límites de la API REST de Power BI](https://msdn.microsoft.com/library/dn950053.aspx).

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante la CLI de Azure](quick-create-azure-cli.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante una plantilla de ARM](quick-create-azure-resource-manager.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics con Visual Studio](stream-analytics-quick-create-vs.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-visual-studio-code.md)
