---
title: 'Servidores en Azure Database for PostgreSQL: Servidor flexible (versión preliminar)'
description: 'En este artículo se proporcionan consideraciones e instrucciones para configurar y administrar Azure Database for PostgreSQL: Servidor flexible.'
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 5cd4454d5b77d18940f6693a25a691a01f4fedec
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932472"
---
# <a name="servers---azure-database-for-postgresql---flexible-server"></a>Servidores en Azure Database for PostgreSQL: Servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible en versión preliminar

En este artículo se proporcionan consideraciones e instrucciones para trabajar con Azure Database for PostgreSQL: Servidor flexible.

## <a name="what-is-an-azure-database-for-postgresql-server"></a>¿Qué es un servidor de Azure Database for PostgreSQL?

Un servidor en la opción de implementación Azure Database for PostgreSQL: Servidor flexible es un punto central de administración de varias bases de datos. Se trata de la misma construcción de servidores de PostgreSQL con la que puede estar familiarizado en entornos locales. En concreto, el servicio PostgreSQL se administra, ofrece garantías de rendimiento y expone el acceso y las características a nivel de servidor.

Un servidor de Azure Database for PostgreSQL:

- Se crea dentro de una suscripción de Azure.
- Es el recurso principal de las bases de datos.
- Proporciona un espacio de nombres para las bases de datos.
- Es un contenedor con semántica de duración segura. Si se elimina un servidor, se eliminan las bases de datos que contiene.
- Coloca recursos en una región.
- Proporciona un punto de conexión para el acceso a la base de datos y al servidor.
- Proporciona el ámbito de las directivas de administración que se aplican a sus bases de datos: inicio de sesión, firewall, usuarios, roles, configuración, etc.
- Está disponible en varias versiones. Para más información, consulte las [versiones de base de datos admitidas de PostgreSQL](concepts-supported-versions.md).
- Es ampliable por los usuarios. Para más información, consulte la información sobre [extensiones de PostgreSQL](concepts-extensions.md).

Dentro del servidor de Azure Database for PostgreSQL, puede crear una o varias bases de datos. Puede optar por crear una sola base de datos por servidor para que use todos los recursos, o bien crear varias para que los compartan. El precio está estructurado por servidor y se basa en la configuración del plan de tarifa, los núcleos virtuales y el almacenamiento (GB). Para obtener más información, vea [Opciones de proceso y almacenamiento](concepts-compute-storage.md).

## <a name="how-do-i-connect-and-authenticate-to-the-database-server"></a>¿Cómo conectarse a un servidor de bases de datos y autenticarse en él?

Los elementos siguientes ayudan a garantizar el acceso seguro a la base de datos:

|||
|:--|:--|
| **Autenticación y autorización** | El servidor de Azure Database for PostgreSQL admite la autenticación nativa de PostgreSQL. Puede conectarse al servidor y autenticarse en él con el inicio de sesión de administrador del servidor. |
| **Protocolo** | El servicio admite un protocolo basado en mensajes utilizado por PostgreSQL. |
| **TCP/IP** | Se admite el protocolo a través de TCP/IP y a través de sockets de dominio de Unix. |
| **Firewall** | Para mantener los datos protegidos, una regla de firewall impide todo acceso al servidor y a las bases de datos, hasta que se especifique qué equipos cuentan con permiso. Vea la información sobre las [reglas de firewall del servidor de Azure Database for PostgreSQL](how-to-manage-firewall-portal.md). |

## <a name="managing-your-server"></a>Administración del servidor

Puede administrar servidores de Azure Database for PostgreSQL en [Azure Portal](https://portal.azure.com) o con la [CLI de Azure](/cli/azure/postgres).

Al crear un servidor, configure las credenciales del usuario administrador. El usuario administrador es el usuario con más privilegios en todo el servidor. Pertenece al rol azure_pg_admin. Este rol no tiene permisos de superusuario completos. 

El atributo de superusuario de PostgreSQL se asigna a azure_superuser, que pertenece al servicio administrado. No tiene acceso a este rol.

Un servidor de Azure Database for PostgreSQL tiene bases de datos predeterminadas: 

- **postgres**: una base de datos predeterminada a la que puede conectarse una vez creado el servidor.
- **azure_maintenance**: esta base de datos se usa para separar los procesos que proporciona el servicio administrado de las acciones del usuario. No dispone de acceso a esta base de datos.

## <a name="server-parameters"></a>Parámetros del servidor

Los parámetros de servidor de PostgreSQL determinan la configuración del servidor. En Azure Database for PostgreSQL, la lista de parámetros puede verse y editarse mediante Azure Portal o la CLI de Azure.

Como servicio administrado para Postgres, los parámetros configurables en Azure Database for PostgreSQL son un subconjunto de los parámetros en una instancia local de Postgres (para obtener más información sobre los parámetros de Postgres, consulte la [documentación de PostgreSQL](https://www.postgresql.org/docs/12/static/runtime-config.html)). El servidor de Azure Database for PostgreSQL está habilitado con los valores predeterminados para cada parámetro en el momento de la creación. El usuario no puede configurar algunos parámetros que requieren un reinicio del servidor o acceso de superusuario para que los cambios se apliquen.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener una visión general del servicio, consulte [Introducción a Azure Database for PostgreSQL](overview.md).
- Para obtener información sobre las cuotas y las limitaciones aplicables a recursos específicos en función de su **configuración**, vea [Opciones de proceso y almacenamiento](concepts-compute-storage.md).
- Vea y edite los parámetros del servidor a través de [Azure Portal](howto-configure-server-parameters-using-portal.md) o la [CLI de Azure](howto-configure-server-parameters-using-cli.md).
