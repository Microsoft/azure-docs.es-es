---
title: Ejemplos de CLI
description: Busque ejemplos de la CLI de Azure para algunos de los escenarios de App Service comunes. Aprenda a automatizar las tareas de administración o implementación de App Service.
tags: azure-service-management
ms.assetid: 53e6a15a-370a-48df-8618-c6737e26acec
ms.topic: sample
ms.date: 07/07/2020
ms.custom: mvc
ms.openlocfilehash: beab87618b97da4e61b0525c0c5a6bdd134fb7f8
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "86169449"
---
# <a name="cli-samples-for-azure-app-service"></a>Ejemplos de la CLI para Azure App Service

En la tabla siguiente se incluyen vínculos a scripts de Bash creados con la CLI de Azure.

| Script | Descripción |
|-|-|
|**Creación de la aplicación**||
| [Creación de una aplicación e implementación de archivos con FTP](./scripts/cli-deploy-ftp.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una aplicación de App Service e implementa un archivo mediante FTP. |
| [Creación de una aplicación e implementación de código desde GitHub](./scripts/cli-deploy-github.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una aplicación de App Service e implementa código proveniente de un repositorio público de GitHub. |
| [Creación de una aplicación con implementación continua desde GitHub](./scripts/cli-continuous-deployment-github.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una publicación de App Service con publicación continua desde un repositorio de GitHub de su propiedad. |
| [Creación de una aplicación e implementación de código desde un repositorio local de GitHub](./scripts/cli-deploy-local-git.md?toc=%2fcli%2fazure%2ftoc.json) | Crea una aplicación de App Service y configura la inserción de código desde un repositorio de Git local. |
| [Creación de una aplicación e implementación de código en un entorno de ensayo](./scripts/cli-deploy-staging-environment.md?toc=%2fcli%2fazure%2ftoc.json) | Crea una aplicación de App Service con una ranura de implementación para cambios en el código de ensayo. |
| [Creación de una aplicación de ASP.NET Core en un contenedor de Docker](./scripts/cli-linux-docker-aspnetcore.md?toc=%2fcli%2fazure%2ftoc.json) | Crea una aplicación de App Service en Linux y carga una imagen de Docker desde Docker Hub. |
| [Creación de una aplicación y exposición con un punto de conexión privado](./scripts/cli-deploy-privateendpoint.md?toc=%2fcli%2fazure%2ftoc.json) | Crea una aplicación de App Service y un punto de conexión privado. |
|**Configuración de la aplicación**||
| [Asignar un dominio personalizado a una aplicación](./scripts/cli-configure-custom-domain.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una aplicación de App Service y le asigna un nombre de dominio personalizado. |
| [Enlace de un certificado TLS/SSL personalizado a una aplicación](./scripts/cli-configure-ssl-certificate.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una aplicación de App Service y enlaza a ella el certificado TLS/SSL de un nombre de dominio personalizado. |
|**Escalado de la aplicación**||
| [Escalado manual de una aplicación](./scripts/cli-scale-manual.md?toc=%2fcli%2fazure%2ftoc.json) | Crea una aplicación de App Service y la escala a través de dos instancias. |
| [Escalado de una aplicación en todo el mundo con una arquitectura de alta disponibilidad](./scripts/cli-scale-high-availability.md?toc=%2fcli%2fazure%2ftoc.json) | Crea dos aplicaciones de App Service en dos regiones geográficas diferentes y hace que estén disponibles mediante un punto de conexión único con Azure Traffic Manager. |
|**Protección de aplicaciones**||
| [Integración con Azure Application Gateway](./scripts/cli-integrate-app-service-with-application-gateway.md?toc=%2fcli%2fazure%2ftoc.json) | Crea una aplicación de App Service y la integra con Application Gateway mediante el punto de conexión de servicio y las restricciones de acceso. |
|**Conexión de la aplicación a recursos**||
| [Conexión de una aplicación a SQL Database](./scripts/cli-connect-to-sql.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una aplicación de App Service y una base de datos de Azure SQL Database y, luego, agrega la cadena de conexión de base de datos a la configuración de la aplicación. |
| [Conexión de una aplicación a una cuenta de almacenamiento](./scripts/cli-connect-to-storage.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una aplicación de App Service y una cuenta de almacenamiento y, a continuación, agrega la cadena de conexión de almacenamiento a la configuración de la aplicación. |
| [Conexión de una aplicación a Azure Cache for Redis](./scripts/cli-connect-to-redis.md?toc=%2fcli%2fazure%2ftoc.json) | Crea una aplicación de App Service y una instancia de Azure Cache for Redis, y después agrega los detalles de conexión de Redis a la configuración de la aplicación. |
| [Conexión de una aplicación a Cosmos DB](./scripts/cli-connect-to-documentdb.md?toc=%2fcli%2fazure%2ftoc.json) | Crea una aplicación de App Service y una instancia de Cosmos DB y, luego, agrega los detalles de conexión de Cosmos DB a la configuración de la aplicación. |
|**Copia de seguridad y restauración de la aplicación**||
| [Copia de seguridad de una aplicación](./scripts/cli-backup-onetime.md?toc=%2fcli%2fazure%2ftoc.json) | Crea una aplicación de App Service y una copia de seguridad única para ella. |
| [Creación de una copia de seguridad programada para una aplicación](./scripts/cli-backup-scheduled.md?toc=%2fcli%2fazure%2ftoc.json) | Crea una aplicación de App Service y una copia de seguridad programada para ella. |
| [Restauración de una aplicación a partir de una copia de seguridad](./scripts/cli-backup-restore.md?toc=%2fcli%2fazure%2ftoc.json) | Restaura una aplicación de App Service a partir de una copia de seguridad. |
|**Supervisión de la aplicación**||
| [Supervisión de una aplicación con registros de servidor web](./scripts/cli-monitor.md?toc=%2fcli%2fazure%2ftoc.json) | Crea una aplicación de App Service, habilita el registro para ella y descarga los registros en la máquina local. |
| | |
