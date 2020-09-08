---
title: Solución de problemas de excepciones de servicio no autorizado de Azure Cosmos DB
description: Descubra cómo diagnosticar y corregir excepciones de servicio no autorizado.
author: j82w
ms.service: cosmos-db
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: a4f51b641ca38b2b6f74bb77928537270d12f1e8
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88870857"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-unauthorized-exceptions"></a>Diagnóstico y solución de problemas de la excepción de servicio no autorizado de Azure Cosmos DB

HTTP 401: La firma MAC de la solicitud HTTP no es la misma que la firma calculada.
Si recibe el mensaje de error 401 ("La firma MAC de la solicitud HTTP no es la misma que la firma calculada"), puede deberse a los siguientes escenarios.

En los SDK más antiguos, la excepción puede aparecer como una excepción de archivo JSON no válido, en lugar de la excepción de servicio no autorizado 401 adecuada. Los SDK más recientes controlan correctamente este escenario y proporcionan un mensaje de error válido.

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas
La lista siguiente muestra las causas y las soluciones conocidas de excepciones de servicio no autorizado.

### <a name="the-key-wasnt-properly-rotated-is-the-most-common-scenario"></a>El escenario más habitual es que la rotación de la clave no se ha producido adecuadamente.
La firma MAC 401 aparece poco después de una rotación de claves y finalmente se detiene sin cambios. 

#### <a name="solution"></a>Solución:
La rotación de la clave no ha seguido los [procedimientos recomendados](secure-access-to-data.md#key-rotation). La rotación de claves de cuenta de Azure Cosmos DB puede tardar de unos pocos segundos a días, en función del tamaño de la cuenta de Cosmos DB.

### <a name="the-key-is-misconfigured"></a>La clave está mal configurada. 
El problema de la firma MAC 401 será consecuente y se producirá en todas las llamadas que usan esa clave.

#### <a name="solution"></a>Solución:
La clave está mal configurada en la aplicación, y se está usando la clave incorrecta para la cuenta o no se ha copiado la clave completa.

### <a name="the-application-is-using-the-read-only-keys-for-write-operations"></a>La aplicación utiliza las claves de solo lectura para las operaciones de escritura.
El problema de la firma MAC 401 solo se produce para operaciones de escritura como crear o reemplazar, pero la solicitud de lectura se realiza correctamente.

#### <a name="solution"></a>Solución:
Cambie la aplicación para que use una clave de lectura/escritura para permitir que las operaciones se completen correctamente.

### <a name="race-condition-with-create-container"></a>Condición de carrera con creación de contenedor
El problema de la firma MAC 401 aparece poco después de la creación de un contenedor. Este problema solo se produce hasta que se completa la creación del contenedor.

#### <a name="solution"></a>Solución:
Hay una condición de carrera con respecto a la creación de contenedores. Una instancia de la aplicación está intentando acceder al contenedor antes de que se complete la creación del contenedor. El escenario más común para esta condición de carrera se produce cuando la aplicación está en ejecución y el contenedor se elimina y se vuelve a crear con el mismo nombre. El SDK intentará usar el nuevo contenedor, pero la creación del contenedor está en curso, por lo que no tiene las claves.

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al utilizar el SDK de Azure Cosmos DB para .NET.
* Más información sobre las directrices de rendimiento de [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET v2](performance-tips.md).