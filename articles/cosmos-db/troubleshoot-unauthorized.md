---
title: Solución de problemas de la excepción de recurso no autorizado de Azure Cosmos DB
description: Diagnóstico y corrección de la excepción de recurso no autorizado
author: j82w
ms.service: cosmos-db
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 872780ff65dabe5a931ae712c0f99b6e4f3e726d
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293849"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-unauthorized-exception"></a>Diagnóstico y solución de problemas de la excepción de recurso no autorizado de Azure Cosmos DB

HTTP 401: La firma MAC que se encuentra en la solicitud HTTP no es la misma que la firma calculada.
Si recibe el siguiente mensaje de error 401: "La firma MAC que se encuentra en la solicitud HTTP no es la misma que la firma calculada". Puede deberse a los siguientes escenarios.

En SDK más antiguos, la excepción puede aparecer como una excepción JSON no válida en lugar de la excepción 401 No autorizado correcta. Los SDK más recientes controlan correctamente este escenario y proporcionan un mensaje de error válido.

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas
La lista siguiente contiene las causas y las soluciones conocidas para la excepción de recurso no autorizado.

### <a name="1-key-was-not-properly-rotated-is-the-most-common-scenario"></a>1. El escenario más habitual es que la clave no ha rotado correctamente.
La firma MAC 401 aparece poco después de una rotación de claves y finalmente se detiene sin realizar ningún cambio. 

#### <a name="solution"></a>Solución:
La clave se rotó y no siguió los [procedimientos recomendados](secure-access-to-data.md#key-rotation). La rotación de claves de cuenta de Cosmos DB puede tardar entre unos segundos hasta días, según el tamaño de la cuenta de Cosmos DB.

### <a name="2-the-key-is-misconfigured"></a>2. La clave está mal configurada. 
El problema de la firma MAC 401 será coherente y se producirá en todas las llamadas que usan esa clave.

#### <a name="solution"></a>Solución:
La clave está mal configurada en la aplicación y se está usando la clave incorrecta para la cuenta o no se ha copiado la clave completa.

### <a name="3-the-application-is-using-the-read-only-keys-for-write-operations"></a>3. La aplicación utiliza las claves de solo lectura para las operaciones de escritura.
El problema de la firma MAC 401 solo se produce para operaciones de escritura como crear o reemplazar, pero la solicitud de lectura se realiza correctamente.

#### <a name="solution"></a>Solución:
Cambie la aplicación para que use una clave de lectura/escritura para permitir que las operaciones se completen correctamente.

### <a name="4-race-condition-with-create-container"></a>4. Condición de carrera con creación de contenedor
El problema de la firma MAC 401 aparece poco después de la creación de un contenedor. Esto solo ocurre hasta que se completa la creación del contenedor.

#### <a name="solution"></a>Solución:
Hay una condición de carrera con la creación de contenedores. Una instancia de la aplicación está intentando tener acceso al contenedor antes de completar la creación de dicho contenedor. Esto comúnmente sucede para esta condición de carrera si la aplicación está en ejecución y el contenedor se elimina y se vuelve a crear con el mismo nombre. El SDK intentará usar el nuevo contenedor, pero la creación de este seguirá en curso, por lo que no tendrá las claves.

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al usar el SDK de .NET de Azure Cosmos DB
* Más información sobre las directrices de rendimiento para [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET V2](performance-tips.md)