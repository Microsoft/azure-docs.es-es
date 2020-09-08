---
title: Hacer que expiren datos en Azure Cosmos DB con período de vida
description: Con TTL, Microsoft Azure Cosmos DB ofrece la posibilidad de que los documentos se purguen automáticamente del sistema tras un período de tiempo.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/02/2020
ms.reviewer: sngun
ms.openlocfilehash: 976cb096ca654c38d7c4c2534bc6938026be5771
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89397039"
---
# <a name="time-to-live-ttl-in-azure-cosmos-db"></a>Período de vida (TTL) en Azure Cosmos DB

Mediante el **período de vida** o TTL, Azure Cosmos DB proporciona la capacidad de eliminar automáticamente elementos de un contenedor después de un determinado período de tiempo. De forma predeterminada, puede establecer el período de vida en el nivel de contenedor e invalidar el valor en cada elemento. Después de establecer el TTL en el nivel de contenedor o de elemento, Azure Cosmos DB eliminará automáticamente estos elementos cuando haya pasado el período de tiempo seleccionado tras la hora de la última modificación. El valor del período de vida se configura en segundos. Cuando se configura el TTL, el sistema elimina automáticamente los elementos expirados en función del valor de TTL, sin necesidad de una operación de eliminación que se emita explícitamente desde la aplicación cliente. El valor máximo de TTL es 2147483647.

La eliminación de los elementos expirados es una tarea en segundo plano que usa las [unidades de solicitud](request-units.md) restantes, es decir, las unidades de solicitud que no han consumido las solicitudes del usuario. Incluso después de que expire el período de vida, si el contenedor está sobrecargado con solicitudes y no hay suficientes RU disponibles, se retrasa la eliminación de los datos. Los datos se eliminan cuando hay suficientes RU disponibles para realizar la operación de eliminación. Aunque se retrasa la eliminación de los datos, las consultas (por parte de cualquier API) no devuelven los datos después de que expira el período de vida.

## <a name="time-to-live-for-containers-and-items"></a>Período de vida para contenedores y elementos

El valor de período de vida se establece en segundos, y se interpreta como una delta desde el momento en que se modificó por última vez un elemento. Puede establecer el período de vida en un contenedor o en un elemento dentro del contenedor:

1. **Período de vida en un contenedor** (se establece mediante `DefaultTimeToLive`):

   - Si no existe (o está establecido en NULL), los elementos no expiran automáticamente.

   - Si existe y el valor se ha establecido en "-1", es igual a infinito y los documentos no expiran de forma predeterminada.

   - Si existe y el valor se ha establecido en un número *"n"*, los elementos expiran *"n"* segundos después de la última modificación.

2. **Período de vida en un elemento** (se establece mediante `ttl`):

   - Esta propiedad es aplicable solo si está presente el valor `DefaultTimeToLive` y no está establecido en NULL para el contenedor primario.

   - Si está presente, invalida el valor de `DefaultTimeToLive` del contenedor primario.

## <a name="time-to-live-configurations"></a>Configuraciones de período de vida

* Si el TTL se establece en *"n"* en un contenedor, los elementos de ese contenedor expirarán después de *n* segundos.  Si hay elementos en el mismo contenedor que tengan su propio período de vida establecido en -1 (lo que indica que no expiran), o si algunos elementos han invalidado la configuración del período de vida con un número distinto, estos elementos expirarán según su propio valor de TTL configurado. 

* Si no se establece el TTL en un contenedor, el período de vida en un elemento de este contenedor no tiene ningún efecto. 

* Si el TTL en un contenedor se establece en -1, un elemento de este contenedor que tenga el período de vida establecido en n, expirará después de n segundos, y los elementos restantes no expirarán.

## <a name="examples"></a>Ejemplos

En esta sección se muestra algunos ejemplos con distintos valores de período de vida asignados a los contenedores y los elementos:

### <a name="example-1"></a>Ejemplo 1

TTL de contenedor se establece en null (DefaultTimeToLive = null)

|TTL de elemento| Resultado|
|---|---|
|TTL = null|    TTL se deshabilita. El elemento nunca expirará (valor predeterminado).|
|TTL = -1   |TTL se deshabilita. El elemento nunca expirará.|
|TTL = 2000 |TTL se deshabilita. El elemento nunca expirará.|


### <a name="example-2"></a>Ejemplo 2

TTL de contenedor se establece en -1 (DefaultTimeToLive = -1)

|TTL de elemento| Resultado|
|---|---|
|TTL = null |TTL está habilitado. El elemento nunca expirará (valor predeterminado).|
|TTL = -1   |TTL está habilitado. El elemento nunca expirará.|
|TTL = 2000 |TTL está habilitado. El elemento expirará transcurridos 2000 segundos.|


### <a name="example-3"></a>Ejemplo 3

TTL de contenedor se establece en -1000 (DefaultTimeToLive = -1000)

|TTL de elemento| Resultado|
|---|---|
|TTL = null|    TTL está habilitado. El elemento expirará transcurridos 1000 segundos (valor predeterminado).|
|TTL = -1   |TTL está habilitado. El elemento nunca expirará.|
|TTL = 2000 |TTL está habilitado. El elemento expirará transcurridos 2000 segundos.|

## <a name="next-steps"></a>Pasos siguientes

Aprenda a configurar el período de vida con los siguientes artículos:

* [Cómo configurar el período de vida](how-to-time-to-live.md)
