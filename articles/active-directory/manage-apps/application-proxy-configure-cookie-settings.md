---
title: 'Configuración de cookies para Application Proxy: Azure Active Directory | Microsoft Docs'
description: Azure Active Directory (Azure AD) cuenta con cookies de sesión y acceso para acceder a aplicaciones locales a través de Application Proxy. En este artículo, encontrará información sobre cómo usar y configurar los valores de las cookies.
services: active-directory
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 01/16/2019
ms.author: mimart
ms.reviewer: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: d2e7f1bb54ce316a10eca0d020519779b0536c9e
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65825742"
---
# <a name="cookie-settings-for-accessing-on-premises-applications-in-azure-active-directory"></a>Configuración de las cookies para el acceso a aplicaciones locales en Azure Active Directory

Azure Active Directory (Azure AD) cuenta con cookies de sesión y acceso para acceder a aplicaciones locales a través de Application Proxy. Obtenga información sobre cómo usar la configuración de cookies de Application Proxy. 

## <a name="what-are-the-cookie-settings"></a>¿Qué son las configuraciones de cookies?

[Application Proxy](application-proxy.md) utiliza los siguientes valores de configuración para cookies de sesión y acceso.

| Configuración de cookies | Valor predeterminado | DESCRIPCIÓN | Recomendaciones |
| -------------- | ------- | ----------- | --------------- |
| Usar cookie solo HTTP | **No** | **Sí** permite que Application Proxy incluya la marca HTTPOnly en los encabezados de respuesta HTTP. Esta marca proporciona ventajas de seguridad adicionales; por ejemplo, evita que el scripting del cliente (CSS) copie o modifique las cookies.<br></br><br></br>Antes de que se añadiera la compatibilidad con la configuración solo HTTP, Application Proxy cifraba y transmitía las cookies a través de un canal SSL seguro para protegerlas contra las modificaciones. | Use **Sí** para obtener las ventajas de seguridad adicionales.<br></br><br></br>Use **No** para los agentes de usuario o cliente que necesiten acceder a la cookie de sesión. Por ejemplo, use **No** para un cliente RDP o MTSC que se conecta a un servidor de puerta de enlace de Escritorio remoto a través Application Proxy.|
| Usar cookies seguras | **No** | **Sí** permite que Application Proxy incluya la marca Secure en los encabezados de respuesta HTTP. Las cookies seguras mejoran la seguridad al transmitir las cookies a través de un canal protegido mediante TLS, como HTTPS. Esto impide que personas no autorizadas observen las cookies, ya que no se transmiten como texto no cifrado. | Use **Sí** para obtener las ventajas de seguridad adicionales.|
| Usar cookies persistentes | **No** | **Sí** permite que Application Proxy configure las cookies de acceso de forma que no expiren cuando se cierre el explorador web. La persistencia dura hasta que expire el token de acceso o hasta que el usuario elimine manualmente las cookies persistentes. | Use **No** debido al riesgo de seguridad asociado con mantener autenticados a los usuarios.<br></br><br></br>Se recomienda usar **Sí** únicamente para aplicaciones más antiguas que no pueden compartir cookies entre procesos. Es mejor actualizar la aplicación para controlar el uso compartido de cookies entre procesos, en lugar de usar las cookies persistentes. Por ejemplo, puede necesitar las cookies persistentes para permitir que un usuario abra documentos de Office en la vista de explorador desde un sitio de SharePoint. Sin cookies persistentes, esta operación puede producir un error si las cookies de acceso no se comparten entre el explorador, el proceso del explorador y el proceso de Office. |

## <a name="set-the-cookie-settings---azure-portal"></a>Configuración de los valores de cookies: Azure Portal
Para configurar los valores de las cookies en Azure Portal:

1. Inicie sesión en el [Azure Portal](https://portal.azure.com). 
2. Vaya a **Azure Active Directory** > **Aplicaciones empresariales** > **Todas las aplicaciones**.
3. Seleccione la aplicación para la que quiere habilitar las configuraciones de cookies.
4. Haga clic en **Application Proxy**.
5. En **Configuración adicional**, establezca la configuración de la cookie en **Sí** o **No**.
6. Haga clic en **Guardar** para aplicar los cambios. 

## <a name="view-current-cookie-settings---powershell"></a>Ver la configuración actual de cookie - PowerShell

Para ver la configuración actual de la cookie para la aplicación, use este comando de PowerShell:  

```powershell
Get-AzureADApplicationProxyApplication -ObjectId <ObjectId> | fl * 
```

## <a name="set-cookie-settings---powershell"></a>Establecer a configuración de cookies - PowerShell

En los siguientes comandos de PowerShell, ```<ObjectId>``` es el valor de ObjectId de la aplicación. 

**Http-Only Cookie** 

```powershell
Set-AzureADApplicationProxyApplication -ObjectId <ObjectId> -IsHttpOnlyCookieEnabled $true 
Set-AzureADApplicationProxyApplication -ObjectId <ObjectId> -IsHttpOnlyCookieEnabled $false 
```

**Cookie segura**

```powershell
Set-AzureADApplicationProxyApplication -ObjectId <ObjectId> -IsSecureCookieEnabled $true 
Set-AzureADApplicationProxyApplication -ObjectId <ObjectId> -IsSecureCookieEnabled $false 
```

**Cookies persistentes**

```powershell
Set-AzureADApplicationProxyApplication -ObjectId <ObjectId> -IsPersistentCookieEnabled $true 
Set-AzureADApplicationProxyApplication -ObjectId <ObjectId> -IsPersistentCookieEnabled $false 
```
