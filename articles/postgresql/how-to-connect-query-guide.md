---
title: Conexión y consulta del Servidor único de PostgreSQL
description: Vínculos a los inicios rápidos de Azure My SQL Database que muestran cómo conectarse al servidor y ejecutar consultas.
services: postgresql
ms.service: postgresql
ms.topic: how-to
author: mksuni
ms.author: sumuth
ms.date: 09/21/2020
ms.openlocfilehash: 924dbadc07f57e5928ecc63a24bf5e57d6213670
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932167"
---
# <a name="connect-and-query-overview-for-azure-database-for-postgresql--single-server"></a>Información general de conexión y consulta de Azure Database for PostgreSQL: Servidor único

En el documento siguiente se incluyen vínculos a ejemplos que muestran cómo conectarse al Servidor único de Azure Database for PostgreSQL y realizar consultas en él. En esta guía también se incluyen las recomendaciones y la extensión de TLS que puede usar para conectarse al servidor en los idiomas admitidos a continuación.

## <a name="quickstarts"></a>Guías de inicio rápido

| Guía de inicio rápido | Descripción |
|---|---|
|[Pgadmin](https://www.pgadmin.org/)|Puede usar pgAdmin para conectarse al servidor y simplifica la creación, el mantenimiento y el uso de objetos de base de datos.|
|[psql en Azure Cloud Shell](quickstart-create-server-database-azure-cli.md#connect-to-the-azure-database-for-postgresql-server-by-using-psql)|En este artículo se muestra cómo ejecutar [**psql**](https://www.postgresql.org/docs/current/static/app-psql.html) en [Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview) para conectarse al servidor y, después, ejecutar instrucciones para consultar, insertar, actualizar y eliminar datos en la base de datos. Puede ejecutar **psql** si está instalado en el entorno de desarrollo.|
|[PostgreSQL con VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)|La extensión de bases de datos de Azure para VS Code (versión preliminar) permite examinar el servidor de PostgreSQL y consultarlo tanto localmente como en la nube mediante el uso de colecciones de recortes con IntelliSense enriquecido. |
|[PHP](connect-php.md)|En este inicio rápido se muestra cómo usar PHP para crear un programa que se conecte a una base de datos y use el trabajo con objetos de base de datos a fin de consultar los datos.|
|[Java](connect-java.md)|En este inicio rápido se muestra cómo utilizar Java para conectarse a una base de datos y luego usar el trabajo con objetos de base de datos a fin de consultar los datos.|
|[Node.js](connect-nodejs.md)|En este inicio rápido se muestra cómo usar Node.js para crear un programa que se conecte a una base de datos y use el trabajo con objetos de base de datos a fin de consultar los datos.|
|[.NET(C#)](connect-csharp.md)|En este inicio rápido se muestra cómo usar .NET (C#) para crear un programa de C# que se conecte a una base de datos y use el trabajo con objetos de base de datos a fin de consultar los datos.|
|[Go](connect-go.md)|En este inicio rápido se muestra cómo usar Go para conectarse a una base de datos. También se muestran las instrucciones Transact-SQL para consultar y modificar los datos.|
|[Python](connect-python.md)|En este inicio rápido se muestra cómo utilizar Python para conectarse a una base de datos y usar el trabajo con objetos de base de datos a fin de consultar los datos. |
|[Ruby](connect-ruby.md)|En este inicio rápido se muestra cómo usar Ruby para crear un programa que se conecte a una base de datos y use el trabajo con objetos de base de datos a fin de consultar los datos.|


## <a name="tls-considerations-for-database-connectivity"></a>Consideraciones de TLS para la conectividad de bases de datos

El protocolo Seguridad de capa de transporte (TLS) lo usan todos los controladores que Microsoft proporciona o admite para conectarse a bases de datos de Azure Database for PostgreSQL. No es necesaria ninguna configuración especial, pero se aplica TLS 1.2 para los servidores recientemente creados. Se recomienda que use TLS 1.0 y 1.1 y, después, actualice la versión de TLS para los servidores. Vea [ Procedimientos para configurar TLS](howto-tls-configurations.md)


## <a name="postgresql-extensions"></a>Extensiones de PostgreSQL
PostgreSQL ofrece la capacidad de ampliar la funcionalidad de su base de datos mediante extensiones. Las extensiones agrupan varios objetos SQL relacionados en un solo paquete que se puede cargar o quitar de la base de datos con un solo comando. Después de cargarse en la base de datos, las extensiones funcionan como características integradas.

- [Extensiones de Postgres 11](https://docs.microsoft.com/azure/postgresql/concepts-extensions#postgres-11-extensions)
- [Extensiones de Postgres 10](https://docs.microsoft.com/azure/postgresql/concepts-extensions#postgres-10-extensions)
- [Extensiones de Postgres 9.6](https://docs.microsoft.com/azure/postgresql/concepts-extensions#postgres-96-extensions)
- [Extensiones de Postgres 9.5](https://docs.microsoft.com/azure/postgresql/concepts-extensions#postgres-95-extensions)

Para obtener información detallada, vea [¿Cómo se utilizan las extensiones de PostgreSQL en el Servidor único?](concepts-extensions.md).

## <a name="next-steps"></a>Pasos a seguir 

- [Migración de datos mediante volcado y restauración](howto-migrate-using-dump-and-restore.md)
- [Migración de datos mediante importación y exportación](howto-migrate-using-export-and-import.md)
