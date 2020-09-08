---
title: Ejemplos de plantillas de Azure Resource Manager
description: Busque ejemplos de plantillas de Azure Resource Manager para algunos de los escenarios de App Service habituales. Aprenda a automatizar las tareas de administración o implementación de App Service.
author: tfitzmac
tags: azure-service-management
ms.topic: sample
ms.date: 08/26/2020
ms.author: tomfitz
ms.custom: mvc, fasttrack-edit
ms.openlocfilehash: 5129fccceb633991767cdd3b52bbb5b6af067270
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88891052"
---
# <a name="azure-resource-manager-templates-for-app-service"></a>Plantillas de Azure Resource Manager para App Service

En la tabla siguiente se incluyen vínculos a plantillas de Azure Resource Manager para Azure App Service. Para obtener recomendaciones sobre cómo evitar errores habituales al crear plantillas de aplicaciones, consulte [Guía de implementación de aplicaciones mediante plantillas de Azure Resource Manager](deploy-resource-manager-template.md).

Para obtener información sobre la sintaxis de JSON y las propiedades para los recursos de App Services, consulte [Tipos de recursos Microsoft.Web ](/azure/templates/microsoft.web/allversions).

| Implementación de una aplicación | Descripción |
|-|-|
| [App Service plan and basic Linux app](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-basic-linux) (Plan de App Service y aplicación básica para Linux) | Implementa una aplicación de App Service configurada para Linux. |
| [App Service plan and basic Windows web app](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-basic-windows) (Plan de App Service y aplicación web básica para Windows) | Implementa una aplicación de App Service configurada para Windows. |
| [App linked to a GitHub repository](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-github-deploy) (Aplicación vinculada a un repositorio de GitHub)| Implementa una aplicación de App Service que extrae código de GitHub. |
| [App with custom deployment slots](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-custom-deployment-slots) (Aplicación con ranuras de implementación personalizados)| Implementa una aplicación de App Service con ranuras o entornos de implementación personalizados. |
| [Aplicación con punto de conexión privado](https://github.com/Azure/azure-quickstart-templates/tree/master/101-private-endpoint-webapp)| Implementa una aplicación de App Service con un punto de conexión privado. |
|**Configuración de una aplicación**| **Descripción** |
| [App certificate from Key Vault](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-certificate-from-key-vault) (Certificado de aplicación de Key Vault)| Implementa un certificado de aplicación de App Service a partir de un secreto de Azure Key Vault y lo utiliza para el enlace TLS/SSL. |
| [App with a custom domain and SSL](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-custom-domain-and-ssl) (Aplicación con un dominio personalizado y SSL)| Implementa una aplicación de App Service con un nombre de host personalizado y obtiene un certificado de aplicación de Key Vault para el enlace TLS/SSL. |
| [App with a GoLang extension](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-with-golang) (Aplicación con una extensión de GoLang)| Implementa una aplicación de App Service con la extensión de sitio de Golang. A partir de ese momento se pueden ejecutar en Azure aplicaciones web desarrolladas en Golang. |
| [App with Java 8 and Tomcat 8](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-java-tomcat) (Aplicación con Java 8 y Tomcat 8)| Implementa una aplicación de App Service con Java 8 y Tomcat 8 habilitados. A continuación, puede ejecutar aplicaciones Java en Azure. |
| [Aplicación con integración de una red virtual regional](https://github.com/Azure/azure-quickstart-templates/tree/master/101-app-service-regional-vnet-integration)| Implementa una aplicación de App Service con la integración de una red virtual regional habilitada. |
|**Protección de una aplicación**| **Descripción** |
| [Aplicación integrada con Azure Application Gateway](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-with-app-gateway-v2)| Implementa una aplicación de App Service y una instancia de Application Gateway y aísla el tráfico mediante el punto de conexión de servicio y las restricciones de acceso. |
|**Linux app with connected resources** (Aplicación Linux con recursos conectados)| **Descripción** |
| [App on Linux with MySQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-linux-managed-mysql) (Aplicación en Linux con MySQL) | Implementa una aplicación de App Service en Linux con Azure Database for MySQL. |
| [App on Linux with PostgreSQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-linux-managed-postgresql) (Aplicación en Linux con PostgreSQL) | Implementa una aplicación de App Service en Linux con Azure Database for PostgreSQL. |
|**App with connected resources** (Aplicación con recursos conectados)| **Descripción** |
| [App with MySQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-managed-mysql) (Aplicación con MySQL)| Implementa una aplicación de App Service en Windows con Azure Database for MySQL. |
| [App with PostgreSQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-managed-postgresql) (Aplicación con PostgreSQL)| Implementa una aplicación de App Service en Windows con Azure Database for PostgreSQL. |
| [Creación de una base de datos de Azure SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-sql-database)| Implementa una aplicación de App Service y una base de datos de Azure SQL Database en el nivel de servicio Básico. |
| [App with a Blob storage connection](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-blob-connection) (Aplicación con una conexión a Azure Blob Storage)| Implementa una aplicación de App Service con una cadena de conexión de Azure Blob Storage. Luego, puede usar Blob Storage desde la aplicación. |
| [App with an Azure Cache for Redis](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-with-redis-cache) (Aplicación con una instancia de Azure Redis Cache)| Implementa una aplicación de App Service con una instancia de Azure Redis Cache. |
| [Aplicación conectada a una aplicación web de back-end](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-privateendpoint-vnet-injection)| Implementa dos aplicaciones web (front-end y back-end) conectadas de forma segura junto con inserción de red virtual y un punto de conexión privado. |
|**entorno de App Service**| **Descripción** |
| [Create an App Service environment v2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-create) (Creación de un entorno de App Service v2) | Crea un entorno de App Service v2 en la red virtual. |
| [Create an App Service environment v2 with an ILB address](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-ilb-create/) (Creación de un entorno de App Service v2 con una dirección ILB) | Crea un entorno de App Service v2 en la red virtual con una dirección de equilibrador de carga interno privado. |
| [Configure the default SSL certificate for an ILB App Service environment or an ILB App Service environment v2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-ase-ilb-configure-default-ssl) (Configuración del certificado SSL predeterminado para un entorno de App Service con ILB o un entorno de App Service v2 con ILB) | Configura el certificado TLS/SSL predeterminado para un entorno de App Service con ILB o de App Service v2 con ILB. |
| | |
