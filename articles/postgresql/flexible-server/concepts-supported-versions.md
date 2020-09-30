---
title: 'Versiones admitidas en Azure Database for PostgreSQL: Servidor flexible'
description: 'Se describen las versiones principales y secundarias de Postgres admitidas en Azure Database for PostgreSQL: Servidor flexible.'
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 9d651a38361ccaa7f8788fb4a375b39b2953228b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931845"
---
# <a name="supported-postgresql-major-versions-in-azure-database-for-postgresql---flexible-server"></a>Versiones principales de PostgreSQL admitidas en Azure Database for PostgreSQL: Servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible en versión preliminar

Azure Database for PostgreSQL: Servidor flexible actualmente admite las versiones principales siguientes:

## <a name="postgresql-version-12"></a>PostgreSQL, versión 12

La versión secundaria actual es la 12.1. Consulte la [documentación de PostgreSQL](https://www.postgresql.org/docs/12/static/release-12-1.html) para obtener más información sobre las mejoras y correcciones de esta versión secundaria.

## <a name="postgresql-version-11"></a>PostgreSQL, versión 11

La versión secundaria actual es la 11.8. Consulte la [documentación de PostgreSQL](https://www.postgresql.org/docs/11/static/release-11-8.html) para obtener más información sobre las mejoras y correcciones de esta versión secundaria.

## <a name="postgresql-version-10-and-older"></a>PostgreSQL, versión 10 y anteriores

No se admite la versión 10 de PostgreSQL ni las anteriores para Azure Database for PostgreSQL: Servidor flexible. Use la opción de implementación [Servidor único](https://docs.microsoft.com/azure/postgresql/concepts-supported-versions) si necesita versiones anteriores.

## <a name="managing-upgrades"></a>Administración de actualizaciones

El proyecto de PostgreSQL publica periódicamente versiones secundarias para corregir errores detectados. Azure Database for PostgreSQL aplica automáticamente revisiones a los servidores con versiones secundarias durante las implementaciones mensuales del servicio.

Todavía no se admite la actualización automática de la versión principal. Por ejemplo, actualmente no hay una actualización automática de PostgreSQL 11 a PostgreSQL 12.<!-- To upgrade to the next major version, create a [database dump and restore](howto-migrate-using-dump-and-restore.md) to a server that was created with the new engine version.-->

<!--
## Next steps

For information on supported PostgreSQL extensions, see [the extensions document](concepts-extensions.md).
-->
