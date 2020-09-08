---
title: Establecimiento de roles de Azure para el acceso administrativo de Azure
titleSuffix: Azure Cognitive Search
description: Control de acceso basado en rol (RBAC) de Azure Portal para controlar y delegar tareas administrativas para la administración de Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/03/2020
ms.openlocfilehash: f0c8fe6b8df5efef0cf3948c8d628d20c79502ff
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88928690"
---
# <a name="set-azure-roles-for-administrative-access-to-azure-cognitive-search"></a>Configuración de roles de Azure para el acceso administrativo a Azure Cognitive Search

Azure proporciona un [modelo de autorización global basado en roles](../role-based-access-control/role-assignments-portal.md) para todos los servicios que se administran del portal o de las API de Resource Manager. Los roles de Propietario, Colaborador y Lector determinan el nivel de *administración de servicio* de los usuarios, grupos y entidades de seguridad de Active Directory asignados a cada rol. 

> [!Note]
> No hay ningún control de acceso basado en rol (RBAC) para proteger partes de un índice o un subconjunto de documentos. Para tener acceso basado en la identidad a través de los resultados de búsqueda, puede crear filtros de seguridad para recortar los resultados según la identidad y quitar los documentos para los que el solicitante no debe tener acceso. Para obtener más información, consulte [Filtros de seguridad](search-security-trimming-for-azure-search.md) y [Seguridad con Active Directory](search-security-trimming-for-azure-search-with-aad.md).

## <a name="management-tasks-by-role"></a>Tareas de administración por rol

Para Azure Cognitive Search, los roles están asociados a los niveles de permisos que admiten las siguientes tareas de administración:

| Role | Tarea |
| --- | --- |
| Propietario |Crear o eliminar el servicio o cualquier objeto del servicio, entre los que se incluyen claves de API, índices, indexadores, orígenes de datos de indexadores y programas de indexadores.<p>Ver el estado del servicio, incluidos el tamaño de almacenamiento y los recuentos.<p>Agregar o eliminar miembros del rol (solo un propietario puede administrar la pertenencia a roles).<p>Los administradores de suscripciones y los propietarios de servicios tienen pertenencia automática al rol Propietario. |
| Colaborador |El mismo nivel de acceso que el rol de Propietario, menos la administración de roles de Azure. Por ejemplo, un usuario con el rol Colaborador puede crear o eliminar objetos, o ver y regenerar [claves de API](search-security-api-keys.md), pero no puede modificar pertenencias a roles. |
| [Rol integrado Colaborador de Search Service](../role-based-access-control/built-in-roles.md#search-service-contributor) | Equivalente al rol Colaborador. |
| Lector |Ver métricas e información esencial del servicio. Los miembros de este rol no pueden ver el índice, el indizador, el origen de datos ni la información de la clave.  |

Los roles no otorgan derechos de acceso al punto de conexión de servicio. Las operaciones del servicio Search, como la administración de índices, el rellenado del índice y las consultas en datos de búsqueda, se controlan mediante claves de API, no a través de roles. Para más información, consulte [Administración de claves de API](search-security-api-keys.md).

## <a name="permissions-table"></a>Tabla de permisos

En la tabla siguiente se resumen las operaciones permitidas en Azure Cognitive Search y la clave que desbloquea el acceso a una operación determinada.

| Operación | Permisos |
|-----------|-------------------------|
| Crear un servicio | Titular de la suscripción de Azure |
| Escalar un servicio | Clave de administración, RBAC propietario o colaborador en el recurso  |
| Eliminar un servicio | Clave de administración, RBAC propietario o colaborador en el recurso |
| Crear, modificar y eliminar objetos en el servicio: <br>Índices y partes de componentes (incluidas las definiciones del analizador, los perfiles de puntuación y las opciones de CORS), indexadores, orígenes de datos, sinónimos, proveedores de sugerencias | Clave de administración, RBAC propietario o colaborador en el recurso |
| Consultar un índice | Clave de administrador o de consulta (RBAC no es aplicable) |
| Consultar la información del sistema, como devolver estadísticas, recuentos y listas de objetos | Clave de administración, RBAC en el recurso (propietario, colaborador y lector) |
| Administrar claves de administración | Clave de administración, RBAC propietario o colaborador en el recurso |
| Administrar claves de consulta |  Clave de administración, RBAC propietario o colaborador en el recurso  |

## <a name="see-also"></a>Consulte también

+ [Administración mediante PowerShell](search-manage-powershell.md) 
+ [Consideraciones sobre el rendimiento y la optimización de Azure Cognitive Search](search-performance-optimization.md)
+ [Introducción al control de acceso basado en roles en Azure Portal](../role-based-access-control/overview.md)