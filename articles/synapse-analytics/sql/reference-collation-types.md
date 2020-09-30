---
title: Compatibilidad con la intercalación
description: Tipos de intercalación compatibles en SQL de Azure Synapse
author: filippopovic
ms.service: synapse-analytics
ms.topic: reference
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: ef9d0eb3dd97795f3b6757759679351f857891ab
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90032867"
---
# <a name="database-collation-support-for-synapse-sql"></a>Compatibilidad con la intercalación de bases de datos para SQL de Synapse

Las intercalaciones proporcionan la configuración regional, la página de códigos, el criterio de ordenación y las reglas de distinción de caracteres para los tipos de datos basados en caracteres. Una vez que se han elegido estas opciones, todas las columnas y expresiones que requieren información de intercalación heredan la intercalación elegida de la configuración de base de datos. La herencia predeterminada se puede reemplazar si se indica explícitamente una intercalación diferente para un tipo de datos basado en caracteres.

Puede cambiar la intercalación de bases de datos predeterminada desde Azure Portal al crear una base de datos del grupo de SQL. Esta capacidad facilita aún más la creación de una base de datos mediante una de las 3800 intercalaciones de bases de datos compatibles.

Puede especificar la intercalación predeterminada de base de datos de SQL a petición de Synapse en el momento de la creación mediante la instrucción CREATE DATABASE.

## <a name="change-collation"></a>Definir intercalación
Para cambiar la intercalación predeterminada para una base de datos del grupo de SQL, simplemente debe actualizar el campo Intercalación en la experiencia de aprovisionamiento. Por ejemplo, si quiere cambiar la intercalación predeterminada para que distinga mayúsculas de minúsculas, basta con que cambie el nombre de la intercalación de SQL_Latin1_General_CP1_CI_AS a SQL_Latin1_General_CP1_CS_AS. 

Para cambiar la intercalación predeterminada para la base de datos de SQL a petición, puede usar la instrucción ALTER DATABASE.

## <a name="list-of-unsupported-collation-types"></a>Lista de los tipos de intercalación que no se admiten
*    Japanese_Bushu_Kakusu_140_BIN
*    Japanese_Bushu_Kakusu_140_BIN2
*    Japanese_Bushu_Kakusu_140_CI_AI_VSS
*    Japanese_Bushu_Kakusu_140_CI_AI_WS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AI_KS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AI_KS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AS_KS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AS_KS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AI_VSS
*    Japanese_Bushu_Kakusu_140_CS_AI_WS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AI_KS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AI_KS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AS_KS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AS_KS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AI
*    Japanese_Bushu_Kakusu_140_CI_AI_WS
*    Japanese_Bushu_Kakusu_140_CI_AI_KS
*    Japanese_Bushu_Kakusu_140_CI_AI_KS_WS
*    Japanese_Bushu_Kakusu_140_CI_AS
*    Japanese_Bushu_Kakusu_140_CI_AS_WS
*    Japanese_Bushu_Kakusu_140_CI_AS_KS
*    Japanese_Bushu_Kakusu_140_CI_AS_KS_WS
*    Japanese_Bushu_Kakusu_140_CS_AI
*    Japanese_Bushu_Kakusu_140_CS_AI_WS
*    Japanese_Bushu_Kakusu_140_CS_AI_KS
*    Japanese_Bushu_Kakusu_140_CS_AI_KS_WS
*    Japanese_Bushu_Kakusu_140_CS_AS
*    Japanese_Bushu_Kakusu_140_CS_AS_WS
*    Japanese_Bushu_Kakusu_140_CS_AS_KS
*    Japanese_Bushu_Kakusu_140_CS_AS_KS_WS
*    Japanese_XJIS_140_BIN
*    Japanese_XJIS_140_BIN2
*    Japanese_XJIS_140_CI_AI_VSS
*    Japanese_XJIS_140_CI_AI_WS_VSS
*    Japanese_XJIS_140_CI_AI_KS_VSS
*    Japanese_XJIS_140_CI_AI_KS_WS_VSS
*    Japanese_XJIS_140_CI_AS_VSS
*    Japanese_XJIS_140_CI_AS_WS_VSS
*    Japanese_XJIS_140_CI_AS_KS_VSS
*    Japanese_XJIS_140_CI_AS_KS_WS_VSS
*    Japanese_XJIS_140_CS_AI_VSS
*    Japanese_XJIS_140_CS_AI_WS_VSS
*    Japanese_XJIS_140_CS_AI_KS_VSS
*    Japanese_XJIS_140_CS_AI_KS_WS_VSS
*    Japanese_XJIS_140_CS_AS_VSS
*    Japanese_XJIS_140_CS_AS_WS_VSS
*    Japanese_XJIS_140_CS_AS_KS_VSS
*    Japanese_XJIS_140_CS_AS_KS_WS_VSS
*    Japanese_XJIS_140_CI_AI
*    Japanese_XJIS_140_CI_AI_WS
*    Japanese_XJIS_140_CI_AI_KS
*    Japanese_XJIS_140_CI_AI_KS_WS
*    Japanese_XJIS_140_CI_AS
*    Japanese_XJIS_140_CI_AS_WS
*    Japanese_XJIS_140_CI_AS_KS
*    Japanese_XJIS_140_CI_AS_KS_WS
*    Japanese_XJIS_140_CS_AI
*    Japanese_XJIS_140_CS_AI_WS
*    Japanese_XJIS_140_CS_AI_KS
*    Japanese_XJIS_140_CS_AI_KS_WS
*    Japanese_XJIS_140_CS_AS
*    Japanese_XJIS_140_CS_AS_WS
*    Japanese_XJIS_140_CS_AS_KS
*    Japanese_XJIS_140_CS_AS_KS_WS

Además, el grupo de SQL no admite los siguientes tipos de intercalación:

*    SQL_EBCDIC1141_CP1_CS_AS
*    SQL_EBCDIC277_2_CP1_CS_AS
*    UTF-8

## <a name="check-the-current-collation"></a>Comprobar de la intercalación actual
Para comprobar la intercalación actual de la base de datos, puede ejecutar el siguiente fragmento de código de T-SQL:
```sql
SELECT DATABASEPROPERTYEX(DB_NAME(), 'Collation') AS Collation;
```
Cuando se pasa "Collation" como parámetro de propiedad, la función DatabasePropertyEx devuelve la intercalación actual de la base de datos especificada. Puede obtener más información sobre la función DatabasePropertyEx en MSDN.

## <a name="next-steps"></a>Pasos siguientes

En los siguientes artículos puede encontrar más información sobre los procedimientos recomendados para el grupo de SQL y SQL a petición:

- [Procedimientos recomendados para grupos de SQL](best-practices-sql-pool.md)
- [Procedimientos recomendados para SQL a petición](best-practices-sql-on-demand.md)


