---
title: Asignación de variables con SQL de Synapse
description: En este artículo, encontrará sugerencias para asignar variables de T-SQL en SQL de Synapse.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 04/15/2020
ms.author: v-stazar
ms.reviewer: jrasnick
ms.openlocfilehash: c3bd04d037f7852bd7d9ea51719abf27f09093b0
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495682"
---
# <a name="assign-variables-with-synapse-sql"></a>Asignación de variables con SQL de Synapse

En este artículo, encontrará sugerencias para asignar variables de T-SQL con SQL de Synapse.

## <a name="set-variables-with-declare"></a>Definición de variables con DECLARE

Las variables de SQL de Synapse se establecen mediante las instrucciones `DECLARE` o `SET`. Inicializar variables con DECLARE es una de las maneras más flexibles de establecer el valor de una variable en el SQL de Synapse.

```sql
DECLARE @v  int = 0
;
```

También puede utilizar DECLARE para establecer más de una variable a la vez. No se puede usar SELECT ni UPDATE para hacer lo siguiente:

```sql
DECLARE @v  INT = (SELECT TOP 1 c_customer_sk FROM Customer where c_last_name = 'Smith')
,       @v1 INT = (SELECT TOP 1 c_customer_sk FROM Customer where c_last_name = 'Jones')
;
```

No se puede inicializar ni utilizar una variable en la misma instrucción DECLARE. A modo ilustrativo, el ejemplo siguiente no está permitido, ya que *\@p1* se inicializa y se usa en la misma instrucción DECLARE. El siguiente ejemplo produce un error.

```sql
DECLARE @p1 int = 0
,       @p2 int = (SELECT COUNT (*) FROM sys.types where is_user_defined = @p1 )
;
```

## <a name="set-values-with-set"></a>Definición de valores con SET

SET es un método muy común para configurar una sola variable.

Las instrucciones siguientes son formas válidas de establecer una variable con SET:

```sql
SET     @v = (Select max(database_id) from sys.databases);
SET     @v = 1;
SET     @v = @v+1;
SET     @v +=1;
```

Solo puede establecer una variable al mismo tiempo con SET. Aunque los operadores compuestos están permitidos.

## <a name="limitations"></a>Limitaciones

No se puede usar UPDATE para la asignación de variables.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más sugerencias sobre desarrollo, consulte el artículo [Información general sobre desarrollo de SQL de Synapse](develop-overview.md).
