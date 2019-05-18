---
title: Atributos de seguridad para Azure App Service
description: Una lista de comprobación de los atributos de seguridad para la evaluación de Azure App Service
services: app-service
documentationcenter: ''
author: msmbaldwin
manager: barbkess
ms.service: app-service
ms.topic: conceptual
ms.date: 05/08/2019
ms.author: mbaldwin
ms.openlocfilehash: d22fca27943be7ac7db8b8edd5882b9fa4ab3ab9
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65607264"
---
# <a name="security-attributes-for-azure-app-service"></a>Atributos de seguridad para Azure App Service

Este artículo documenta los atributos comunes de seguridad integrados en Azure App Service.

[!INCLUDE [Security attributes header](../../includes/security-attributes-header.md)]

## <a name="preventative"></a>Prevención

| Atributo de seguridad | Sí/No | Notas |
|---|---|--|
| Cifrado en reposo (por ejemplo, el cifrado del lado servidor, el cifrado del lado servidor con claves administradas por el cliente y otras características de cifrado) | Sí | Contenido del archivo de sitio Web se almacena en Azure Storage, que cifra automáticamente el contenido en reposo. Consulte [cifrado de Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md).<br><br>Los secretos de la proporcionada por el usuario se cifran en reposo. Los secretos se cifran en reposo mientras se almacenan en bases de datos de configuración de App Service.<br><br>Los discos conectados localmente, opcionalmente, pueden utilizarse como almacenamiento temporal por los sitios Web (D:\local y % TMP %). Los discos conectados localmente no se cifran en reposo. |
| Cifrado en tránsito (por ejemplo, cifrado de ExpressRoute, en el cifrado de red virtual y redes)| Sí | Los clientes pueden configurar los sitios web para requerir y usar HTTPS para el tráfico entrante. Consulte el blog [cómo hacer que Azure App Service HTTPS solo](https://blogs.msdn.microsoft.com/benjaminperkins/2017/11/30/how-to-make-an-azure-app-service-https-only/). |
| Control de clave de cifrado (CMK, BYOK, etc.)| Sí | Los clientes pueden elegir almacenar secretos de aplicación en Key Vault y recuperarlos en tiempo de ejecución. Consulte [hace referencia un uso de Key Vault para App Service y Azure Functions (versión preliminar)](app-service-key-vault-references.md).|
| Cifrado de nivel de columna (Azure Data Services)| N/D | |
| Llamadas a API cifradas| Sí | Se realizan llamadas de administración para configurar App Service a través de [Azure Resource Manager](../azure-resource-manager/index.yml) llamadas a través de HTTPS. |

## <a name="network-segmentation"></a>Segmentación de red

| Atributo de seguridad | Sí/No | Notas |
|---|---|--|
| Compatibilidad con punto de conexión de servicio| Sí | Disponible actualmente en versión preliminar de App Service. Consulte [restricciones de acceso de Azure App Service](app-service-ip-restrictions.md). |
| Compatibilidad con inserción de red virtual| Sí | Entornos de App Service son implementaciones privadas de App Service dedicado a un solo cliente insertado en una red virtual de cliente. Consulte [Introducción a App Service Environment](environment/intro.md). |
| Compatibilidad con aislamiento de red y Firewalling| Sí | La variación pública de varios inquilinos de App Service, los clientes pueden configurar las ACL (restricciones de IP) para bloquear el tráfico entrante permitido de red.  Consulte [restricciones de acceso de Azure App Service](app-service-ip-restrictions.md).  App Service Environment se implementa directamente en redes virtuales y, por tanto, se puede proteger con NSG. |
| Compatibilidad con tunelización forzada | Sí | Entornos de App Service se pueden implementar en una red virtual de cliente donde se configura la tunelización forzada. Los clientes tienen que seguir las instrucciones de [configurar App Service Environment con tunelización forzada](environment/forced-tunnel-support.md). |

## <a name="detection"></a>Detección

| Atributo de seguridad | Sí/No | Notas|
|---|---|--|
| Supervisión de soporte técnico (Log analytics, Application insights, etcetera) de Azure| Sí | App Service se integra con Application Insights para lenguajes compatibles con Application Insights (versión completa de .NET Framework, .NET Core, Java y Node.JS).  Consulte [rendimiento de supervisión de Azure App Service](../azure-monitor/app/azure-web-apps.md). App Service también envía las métricas de aplicación en Azure Monitor. Consulte [supervisar aplicaciones en Azure App Service](web-sites-monitor.md). |

## <a name="identity-and-access-management"></a>Administración de identidades y acceso

| Atributo de seguridad | Sí/No | Notas|
|---|---|--|
| Authentication| Sí | Los clientes pueden crear aplicaciones en App Service que se integran automáticamente con [Azure Active Directory (Azure AD)](../active-directory/index.md) , así como otros proveedores de identidades compatible con OAuth; vea [autenticación y autorización en Azure App Service](overview-authentication-authorization.md). Para administración de acceso a los recursos de App Service, todo el acceso se controla mediante una combinación de entidad de seguridad de Azure AD autenticado y roles de RBAC de Azure Resource Manager. |
| Autorización| Sí | Para administración de acceso a los recursos de App Service, todo el acceso se controla mediante una combinación de entidad de seguridad de Azure AD autenticado y roles de RBAC de Azure Resource Manager.  |


## <a name="audit-trail"></a>Pista de auditoría

| Atributo de seguridad | Sí/No | Notas|
|---|---|--|
| Registro y auditoría de planificación de control/administración| Sí | Todas las operaciones de administración realizadas en objetos de App Service se producen a través de [Azure Resource Manager](../azure-resource-manager/index.yml). Los registros históricos de estas operaciones están disponibles tanto en el portal a través de la CLI; consulte [operaciones del proveedor de recursos de Azure Resource Manager](../role-based-access-control/resource-provider-operations.md#microsoftweb) y [registro de actividad de monitor az](/cli/azure/monitor/activity-log). |
| Registro y auditoría del plano de datos | No | El plano de datos de App Service es un recurso compartido de archivos remoto que contiene el contenido del sitio web implementado de un cliente.  No hay ninguna auditoría del recurso compartido de archivos remoto. |

## <a name="configuration-management"></a>Administración de configuración

| Atributo de seguridad | Sí/No | Notas|
|---|---|--|
| Compatibilidad con la administración de configuración (control de versiones de configuración, etcetera.)| Sí | Para las operaciones de administración, el estado de una configuración de App Service se puede exportar como una plantilla de Azure Resource Manager y con control de versiones con el tiempo. Para las operaciones en tiempo de ejecución, los clientes pueden mantener versiones diferentes en vivo de una aplicación con la característica de ranuras de implementación de App Service. | 

