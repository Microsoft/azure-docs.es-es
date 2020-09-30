---
title: Seguimiento de cambios de datos en Azure SQL Edge
description: Obtenga información sobre el seguimiento de cambios y la captura de datos modificados en Azure SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: f7d5d5f74a816bf745faf5decf761cd453f40123
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90900069"
---
# <a name="track-data-changes-in-azure-sql-edge"></a>Seguimiento de cambios de datos en Azure SQL Edge

Azure SQL Edge proporciona dos características de SQL Server que realizan el seguimiento de los cambios en los datos de una base de datos: [seguimiento de cambios](https://docs.microsoft.com/sql/relational-databases/track-changes/track-data-changes-sql-server#Tracking) y [captura de datos modificados](https://docs.microsoft.com/sql/relational-databases/track-changes/track-data-changes-sql-server#Capture). Estas características permiten a las aplicaciones determinar los cambios de lenguaje de modificación de datos (operaciones de inserción, actualización y eliminación) que se realizaron en las tablas de usuario de una base de datos. Puede habilitar la captura de datos modificados y el seguimiento de cambios en la misma base de datos. No se requiere ninguna consideración especial.

La capacidad para consultar los datos que han cambiado en una base de datos es un requisito importante para conseguir que algunas aplicaciones sean eficaces. Normalmente, para determinar los datos modificados, los desarrolladores de aplicaciones deben implementar un método de seguimiento personalizado en sus aplicaciones utilizando una combinación de desencadenadores, columnas de marca de tiempo y tablas adicionales. La creación de estas aplicaciones implica normalmente mucho trabajo de implementación, provoca actualizaciones del esquema y, a menudo, supone una gran sobrecarga en el rendimiento.

En el caso de una solución de IoT, donde debe trasladar datos periódicamente desde el borde a una nube o un centro de datos, el seguimiento de cambios puede ser muy útil. Los usuarios pueden consultar de forma rápida y eficaz solo los cambios desde la última sincronización y cargar esos cambios en el destino del centro de datos o la nube. Para más detalles, consulte [Ventajas de utilizar la captura de datos modificados y el seguimiento de cambios](https://docs.microsoft.com/sql/relational-databases/track-changes/track-data-changes-sql-server#benefits-of-using-change-data-capture-or-change-tracking). 

Estas dos características no son iguales. Para más información, consulte [Diferencias de características entre la captura de datos modificados y el seguimiento de cambios](https://docs.microsoft.com/sql/relational-databases/track-changes/track-data-changes-sql-server#feature-differences-between-change-data-capture-and-change-tracking).

## <a name="change-data-capture"></a>captura de datos modificados

Para comprender los detalles de cómo funciona esta característica, consulte [Acerca de la captura de datos modificados (SQL Server)](https://docs.microsoft.com/sql/relational-databases/track-changes/about-change-data-capture-sql-server).

Para saber cómo habilitar o deshabilitar esta característica, consulte [Habilitar y deshabilitar la captura de datos modificados (SQL Server)](https://docs.microsoft.com/sql/relational-databases/track-changes/enable-and-disable-change-data-capture-sql-server).

Para administrar y supervisar esta característica, consulte [Administrar y supervisar la captura de datos modificados (SQL Server)](https://docs.microsoft.com/sql/relational-databases/track-changes/administer-and-monitor-change-data-capture-sql-server).

Para saber cómo consultar y trabajar con los datos modificados, consulte [Trabajar con datos modificados (SQL Server)](https://docs.microsoft.com/sql/relational-databases/track-changes/work-with-change-data-sql-server).

## <a name="change-tracking"></a>seguimiento de cambios

Para comprender los detalles de cómo funciona esta característica, consulte [Acerca del seguimiento de cambios (SQL Server)](https://docs.microsoft.com/sql/relational-databases/track-changes/about-change-tracking-sql-server).

Para saber cómo habilitar o deshabilitar esta característica, consulte [Habilitar y deshabilitar el seguimiento de cambios (SQL Server)](https://docs.microsoft.com/sql/relational-databases/track-changes/enable-and-disable-change-tracking-sql-server).

Para administrar y supervisar esta característica, consulte [Administrar el seguimiento de cambios (SQL Server)](https://docs.microsoft.com/sql/relational-databases/track-changes/manage-change-tracking-sql-server).

Para saber cómo consultar y trabajar con los datos modificados, consulte [Trabajar con datos modificados (SQL Server)](https://docs.microsoft.com/sql/relational-databases/track-changes/work-with-change-tracking-sql-server).

## <a name="temporal-tables"></a>Tablas temporales

Azure SQL Edge también admite la característica de tablas temporales de SQL Server. Esta característica (también conocida como *tablas temporales con versiones del sistema*) ofrece compatibilidad integrada para proporcionar información sobre los datos almacenados en la tabla en cualquier momento. La característica no solo proporciona información sobre los datos que son correctos en el momento actual.

Una tabla temporal con versiones del sistema es un tipo de tabla de usuario pensada para conservar un historial completo de los cambios de datos y para facilitar los análisis en un momento específico. Este tipo de tabla temporal se conoce como tabla temporal con versiones del sistema, porque el período de validez de cada fila lo administra el sistema (es decir, el motor de base de datos).

Cada tabla temporal tiene dos columnas definidas explícitamente, cada una con un tipo de datos `datetime2`. Estas columnas se conocen como columnas de *período*. El sistema utiliza estas columnas de período exclusivamente para registrar el período de validez de cada fila cada vez que se modifica una fila.

Aparte de estas columnas, una tabla temporal contiene también una referencia a otra tabla con un esquema reflejado. El sistema usa esta tabla para almacenar automáticamente la versión anterior de una fila de la tabla temporal cada vez que dicha fila se actualiza o elimina. Esta tabla adicional se conoce como tabla de *historial*, mientras que la tabla principal que almacena las versiones de fila actuales (reales) se conoce como tabla *actual* o, simplemente, como tabla temporal. Durante la creación de una tabla temporal, los usuarios pueden especificar una tabla de historial existente (debe ser compatible con el esquema) o dejar que el sistema cree la tabla de historial predeterminada.

Para más información, consulte [Tablas temporales](https://docs.microsoft.com/sql/relational-databases/tables/temporal-tables).

## <a name="next-steps"></a>Pasos siguientes

- [Streaming de datos en Azure SQL Edge](stream-data.md)
- [Aprendizaje automático e IA con ONNX en Azure SQL Edge](onnx-overview.md)
- [Configuración de la replicación en Azure SQL Edge](configure-replication.md)
- [Copia de seguridad y restauración de bases de datos en Azure SQL Edge](backup-restore.md)



