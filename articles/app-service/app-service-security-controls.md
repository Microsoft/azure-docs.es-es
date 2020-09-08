---
title: Controles de seguridad de Azure App Service
description: Busque una lista de comprobación de los controles de seguridad para evaluar Azure App Service para su organización.
author: msmbaldwin
ms.topic: conceptual
ms.date: 09/04/2019
ms.author: mbaldwin
ms.openlocfilehash: 93e5123e5b61c9013177f7f3c908578b68da52d6
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962934"
---
# <a name="security-controls-for-azure-app-service"></a>Controles de seguridad de Azure App Service

En este artículo se explican los controles de seguridad integrados en Azure App Service.

[!INCLUDE [Security controls header](../../includes/security-controls-header.md)]

## <a name="network"></a>Red

| Control de seguridad | Sí/No | Notas | Documentación
|---|---|--|
| Compatibilidad con punto de conexión de servicio| Sí | Disponible para App Service.| [Restricciones de acceso de Azure App Service](app-service-ip-restrictions.md)
| Compatibilidad con la inserción de redes virtuales| Sí | App Service Environment son implementaciones privadas de App Service dedicadas a un solo cliente insertado en una red virtual del cliente. | [Introducción a App Service Environment](environment/intro.md)
| Compatibilidad con el aislamiento de red y los firewalls| Sí | En la variación pública de varios inquilinos de App Service, los clientes pueden configurar las ACL (restricciones de IP) para bloquear el tráfico entrante permitido.  App Service Environment se implementa directamente en las redes virtuales y, por tanto, se puede proteger con NSG. | [Restricciones de acceso de Azure App Service](app-service-ip-restrictions.md)
| Compatibilidad con la tunelización forzada| Sí | App Service Environment se puede implementar en la red virtual de un cliente que tenga configurada la tunelización forzada. | [Configuración de un entorno de App Service Environment con tunelización forzada](environment/forced-tunnel-support.md)

## <a name="monitoring--logging"></a>Supervisión y registro

| Control de seguridad | Sí/No | Notas | Documentación
|---|---|--|
| Compatibilidad con la supervisión de Azure (Log Analytics, Application Insights, etc.)| Sí | App Service se integra con Application Insights para los lenguajes compatibles con Application Insights (edición completa de .NET Framework, .NET Core, Java y Node.JS).  Consulte [Supervisar el rendimiento de Azure App Service](../azure-monitor/app/azure-web-apps.md). App Service también envía las métricas de aplicación a Azure Monitor. | [Supervisión de aplicaciones en Azure App Service](web-sites-monitor.md)
| Registro y auditoría del plano de administración y de control| Sí | Todas las operaciones de administración realizadas en objetos de App Service se producen a través de [Azure Resource Manager](../azure-resource-manager/index.yml). Los registros históricos de estas operaciones están disponibles en el portal y mediante la CLI. | [Operaciones del proveedor de recursos de Azure Resource Manager](../role-based-access-control/resource-provider-operations.md#microsoftweb), [az monitor activity-log](/cli/azure/monitor/activity-log)
| Registro y auditoría del plano de datos | No | El plano de datos de App Service es un recurso compartido de archivos remoto que incluye el contenido del sitio web implementado de un cliente.  No hay ninguna auditoría del recurso compartido de archivos remoto. |

## <a name="identity"></a>Identidad

| Control de seguridad | Sí/No | Notas |  Documentación
|---|---|--|
| Authentication| Sí | Los clientes pueden crear aplicaciones en App Service que se integren automáticamente con [Azure Active Directory (Azure AD)](../active-directory/index.yml), así como con otros proveedores de identidades compatibles con OAuth. En el caso del acceso de administración a los recursos de App Service, todo el acceso se controla utilizando una entidad de seguridad autenticada de Azure AD y los roles RBAC de Azure. | [Autenticación y autorización en Azure App Service](overview-authentication-authorization.md)
| Authorization| Sí | Para el acceso de administración a los recursos de App Service, todo el acceso se controla mediante una combinación de las entidades de seguridad autenticadas de Azure AD y los roles RBAC de Azure.  | [Autenticación y autorización en Azure App Service](overview-authentication-authorization.md)

## <a name="data-protection"></a>Protección de los datos

| Control de seguridad | Sí/No | Notas | Documentación
|---|---|--|
| Cifrado del lado servidor en reposo: Claves administradas por Microsoft | Sí | El contenido de los archivos de sitio web se almacena en Azure Storage, que cifra automáticamente el contenido en reposo. <br><br>Los secretos suministrados por el cliente se cifran en reposo. Los secretos se cifran en reposo mientras se almacenan en las bases de datos de configuración de App Service.<br><br>Los sitios web pueden utilizar los discos conectados localmente como almacenamiento temporal (D:\local y %TMP%). Los discos conectados localmente no se cifran en reposo. | [Cifrado de Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md)
| Cifrado del lado servidor en reposo: claves administradas por el cliente (BYOK) | Sí | Los clientes pueden elegir almacenar secretos de aplicación en Key Vault y recuperarlos en el entorno en tiempo de ejecución. | [Uso de referencias de Key Vault para App Service y Azure Functions (versión preliminar)](app-service-key-vault-references.md)
| Cifrado de nivel de columna (Azure Data Services)| N/D | |
| Cifrado en tránsito (por ejemplo, cifrado de ExpressRoute, cifrado en la red virtual y cifrado de red virtual a red virtual)| Sí | Los clientes pueden configurar los sitios web para que exijan y usen HTTPS para el tráfico entrante.  | [Cómo hacer que Azure App Service use solo HTTPS](/archive/blogs/benjaminperkins/how-to-make-an-azure-app-service-https-only) (entrada de blog)
| Llamadas a API cifradas| Sí | Las llamadas de administración para configurar App Service se realizan mediante llamadas de [Azure Resource Manager](../azure-resource-manager/index.yml) con HTTPS. |

## <a name="configuration-management"></a>Administración de configuración

| Control de seguridad | Sí/No | Notas | Documentación
|---|---|--|
| Compatibilidad con la administración de configuración (control de versiones de configuración, etc.)| Sí | Para las operaciones de administración, el estado de una configuración de App Service se puede exportar como plantilla de Azure Resource Manager y con control de versiones a lo largo del tiempo. Para las operaciones en tiempo de ejecución, los clientes pueden mantener versiones diferentes en vivo de una aplicación con la característica de espacios de implementación de App Service. | 

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre los [controles de seguridad integrados en los servicios de Azure](../security/fundamentals/security-controls.md).