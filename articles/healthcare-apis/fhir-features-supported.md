---
title: 'Características de FHIR admitidas en Azure: Azure API for FHIR'
description: En este artículo se explica qué características de la especificación de FHIR se implementan en Azure API for FHIR
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: bdf328222fef1f763776bd26c47f5cd4d65e487e
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89000012"
---
# <a name="features"></a>Características

Azure API for FHIR proporciona una implementación totalmente administrada Microsoft FHIR Server para Azure. El servidor es una implementación del estándar [FHIR](https://hl7.org/fhir). En este documento se enumeran las características principales de FHIR Server.

## <a name="fhir-version"></a>Versión de FHIR

Versión más reciente admitida: `4.0.1`

Entre las versiones anteriores también admitidas actualmente se incluye: `3.0.2`

## <a name="rest-api"></a>API DE REST

| API                            | Admitida: PaaS | Admitida: OSS (SQL) | Admitida: OSS (Cosmos DB) | Comentario                                             |
|--------------------------------|-----------|-----------|-----------|-----------------------------------------------------|
| leer                           | Sí       | Sí       | Sí       |                                                     |
| vread                          | Sí       | Sí       | Sí       |                                                     |
| update                         | Sí       | Sí       | Sí       |                                                     |
| update with optimistic locking | Sí       | Sí       | Sí       |                                                     |
| update (conditional)           | Sí       | Sí       | Sí       |                                                     |
| patch                          | No        | No        | No        |                                                     |
| delete                         | Sí       | Sí       | Sí       |                                                     |
| delete (conditional)           | No        | No        | No        |                                                     |
| create                         | Sí       | Sí       | Sí       | Admite POST y PUT                               |
| create (conditional)           | Sí       | Sí       | Sí       |                                                     |
| búsqueda                         | Parcial   | Parcial   | Parcial   | Consulte a continuación                                           |
| búsqueda encadenada                 | No        | Sí       | No        |                                           |
| búsqueda encadenada inversa         | No        | No        | No        |                                            |
| capabilities                   | Sí       | Sí       | Sí       |                                                     |
| proceso por lotes                          | Sí       | Sí       | Sí       |                                                     |
| transaction                    | No        | Sí       | No        |                                                     |
| history                        | Sí       | Sí       | Sí       |                                                     |
| paging                         | Parcial   | Parcial   | Parcial   | `self` y `next` se admiten                     |
| intermediaries                 | No        | No        | No        |                                                     |

## <a name="search"></a>Search

Se admiten todos los tipos de parámetro de búsqueda. 

| Tipo de parámetro de búsqueda | Admitida: PaaS | Admitida: OSS (SQL) | Admitida: OSS (Cosmos DB) | Comentario |
|-----------------------|-----------|-----------|-----------|---------|
| Number                | Sí       | Sí       | Sí       |         |
| Date/DateTime         | Sí       | Sí       | Sí       |         |
| String                | Sí       | Sí       | Sí       |         |
| Token                 | Sí       | Sí       | Sí       |         |
| Referencia             | Sí       | Sí       | Sí       |         |
| Compuesto             | Sí       | Sí       | Sí       |         |
| Cantidad              | Sí       | Sí       | Sí       |         |
| URI                   | Sí       | Sí       | Sí       |         |
| Especial               | No        | No        | No        |         |


| Modificadores             | Admitida: PaaS | Admitida: OSS (SQL) | Admitida: OSS (Cosmos DB) | Comentario |
|-----------------------|-----------|-----------|-----------|---------|
|`:missing`             | Sí       | Sí       | Sí       |         |
|`:exact`               | Sí       | Sí       | Sí       |         |
|`:contains`            | Sí       | Sí       | Sí       |         |
|`:text`                | Sí       | Sí       | Sí       |         |
|`:in` (token)          | No        | No        | No        |         |
|`:below` (token)       | No        | No        | No        |         |
|`:above` (token)       | No        | No        | No        |         |
|`:not-in` (token)      | No        | No        | No        |         |
|`:[type]` (referencia)  | No        | No        | No        |         |
|`:below` (uri)         | Sí       | Sí       | Sí       |         |
|`:not`                 | No        | No        | No        |         |
|`:above` (uri)         | No        | No        | No        | Problema [#158](https://github.com/Microsoft/fhir-server/issues/158) |

| Parámetro de búsqueda común | Admitida: PaaS | Admitida: OSS (SQL) | Admitida: OSS (Cosmos DB) | Comentario |
|-------------------------| ----------| ----------| ----------|---------|
| `_id`                   | Sí       | Sí       | Sí       |         |
| `_lastUpdated`          | Sí       | Sí       | Sí       |         |
| `_tag`                  | Sí       | Sí       | Sí       |         |
| `_profile`              | Sí       | Sí       | Sí       |         |
| `_security`             | Sí       | Sí       | Sí       |         |
| `_text`                 | No        | No        | No        |         |
| `_content`              | No        | No        | No        |         |
| `_list`                 | Sí       | Sí       | Sí       |         |
| `_has`                  | No        | No        | No        |         |
| `_type`                 | Sí       | Sí       | Sí       |         |
| `_query`                | No        | No        | No        |         |

| Operaciones de búsqueda       | Admitida: PaaS | Admitida: OSS (SQL) | Admitida: OSS (Cosmos DB) | Comentario |
|-------------------------|-----------|-----------|-----------|---------|
| `_filter`               | No        | No        | No        |         |
| `_sort`                 | No        | Parcial   | Parcial        |   `_sort=_lastUpdated` se admite       |
| `_score`                | No        | No        | No        |         |
| `_count`                | Sí       | Sí       | Sí       |         |
| `_summary`              | Parcial   | Parcial   | Parcial   | `_summary=count` se admite |
| `_include`              | No        | Sí       | No        |         |
| `_revinclude`           | No        | Sí       | No        | Los elementos incluidos se limitan a 100. |
| `_contained`            | No        | No        | No        |         |
| `_elements`             | No        | No        | No        |         |

## <a name="persistence"></a>Persistencia

Microsoft FHIR Server tiene un módulo de persistencia conectable (consulte [`Microsoft.Health.Fhir.Core.Features.Persistence`](https://github.com/Microsoft/fhir-server/tree/master/src/Microsoft.Health.Fhir.Core/Features/Persistence)).

Actualmente, el código fuente abierto de FHIR Server incluye una implementación para [Azure Cosmos DB](../cosmos-db/index-overview.md) y [SQL Database](https://azure.microsoft.com/services/sql-database/).

Cosmos DB es una base de datos (SQL API, MongoDB API, etc.) de varios modelos distribuida de forma global. Admite diferentes [niveles de coherencia](../cosmos-db/consistency-levels.md). La plantilla de implementación predeterminada configura FHIR Server con una coherencia de `Strong`, pero la directiva de coherencia puede modificarse (generalmente de forma más relajada) en función de cada solicitud mediante el encabezado de solicitud `x-ms-consistency-level`.

## <a name="role-based-access-control"></a>Control de acceso basado en rol

FHIR Server utiliza [Azure Active Directory](https://azure.microsoft.com/services/active-directory/) para el control de acceso. En concreto, se aplica el control de acceso basado en roles (RBAC) si el parámetro de configuración `FhirServer:Security:Enabled` se establece en `true`, y todas las solicitudes (excepto `/metadata`) a FHIR Server deben tener el encabezado de solicitud `Authorization` establecido en `Bearer <TOKEN>`. El token debe contener uno o varios roles, tal como se define en la notificación `roles`. Se permitirá una solicitud si el token contiene un rol que permite la acción especificada en el recurso especificado.

Actualmente, las acciones permitidas para un rol determinado se aplican *globalmente* en la API.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha leído sobre las características admitidas de FHIR en Azure API for FHIR. Después, implemente Azure API for FHIR.
 
>[!div class="nextstepaction"]
>[Implementación de Azure API for FHIR](fhir-paas-portal-quickstart.md)
