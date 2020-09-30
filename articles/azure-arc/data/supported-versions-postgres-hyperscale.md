---
title: Versiones admitidas de Postgres con Hiperescala de PostgreSQL habilitada para Azure Arc
description: Versiones admitidas de Postgres con Hiperescala de PostgreSQL habilitada para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 0f3f9e7f4566800bdea6871cae1c5bafb3b37b95
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932242"
---
# <a name="supported-versions-of-postgres-with-azure-arc-enabled-postgresql-hyperscale"></a>Versiones admitidas de Postgres con Hiperescala de PostgreSQL habilitada para Azure Arc

En este artículo se explica qué versiones de Postgres están disponibles con Hiperescala de PostgreSQL habilitada para Azure Arc.
La lista de versiones admitidas cambia con el tiempo. Actualmente, las versiones principales admitidas son las siguientes:
- Postgres 12 (versión predeterminada)
- Postgres 11

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="how-to-chose-between-versions"></a>Procedimiento para elegir entre versiones
Se recomienda consultar para qué versiones se han diseñado las aplicaciones y cuáles son las capacidades de cada una de las versiones. Para obtener más detalles, lea la información de cada versión en el sitio oficial de Postgres:
- [Postgres 12 (versión predeterminada)](https://www.postgresql.org/docs/12/index.html)
- [Postgres 11](https://www.postgresql.org/docs/11/index.html)

## <a name="how-to-create-a-particular-version-in-azure-arc-enabled-postgresql-hyperscale"></a>Procedimiento para crear una versión determinada en Hiperescala de PostgreSQL habilitada para Azure Arc
En el momento de la creación, tiene la posibilidad de indicar qué versión se va a crear pasando el parámetro _--engine-version_. Si no indica la información de versión, de forma predeterminada, se creará un grupo de servidores de la versión 12 de Postgres.

## <a name="how-do-be-notified-when-other-versions-are-available"></a>Procedimiento para recibir notificaciones si hay otras versiones disponibles
Vuelva a consultar este artículo más adelante, se actualizará según corresponda. También puede enumerar los tipos de definiciones de recursos personalizadas (CRD) en el controlador de datos de Arc en su clúster de Kubernetes.
Ejecute el siguiente comando:
```console
kubectl get crds
```

Se devolverá una salida como la siguiente:
```console
NAME                                            CREATED AT
datacontrollers.arcdata.microsoft.com           2020-08-31T20:15:16Z
postgresql-11s.arcdata.microsoft.com            2020-08-31T20:15:16Z
postgresql-12s.arcdata.microsoft.com            2020-08-31T20:15:16Z
sqlmanagedinstances.sql.arcdata.microsoft.com   2020-08-31T20:15:16Z
```

En este ejemplo, esta salida indica que hay dos tipos de CRD relacionadas con Postgres:
- postgresql-12s.arcdata.microsoft.com es la CRD para Postgres 12
- postgresql-11s.arcdata.microsoft.com es la CRD para Postgres 11

Se trata de definiciones de recursos personalizadas, no de grupos de servidores. La presencia de una CRD no es una indicación de que se ha creado o no un grupo de servidores de esa versión.
La CRD indica el tipo de recursos que se pueden crear.

## <a name="next-steps"></a>Pasos siguientes:
- [Obtenga información sobre la creación de Hiperescala de PostgreSQL habilitada para Azure Arc](create-postgresql-hyperscale-server-group.md).
- [Obtenga información acerca de cómo obtener una lista de los grupos de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc creados en el controlador de datos de Arc](list-server-groups-postgres-hyperscale.md).
