---
title: Solución de errores de la API de informes de Azure Active Directory | Microsoft Docs
description: Le proporciona una resolución a los errores mientras llama a las API de informes de Azure Active Directory.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 0030c5a4-16f0-46f4-ad30-782e7fea7e40
ms.service: active-directory
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: abc8badf261e631dd6ceb7af9a6a0cb3676ae25d
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89231017"
---
# <a name="troubleshoot-errors-in-azure-active-directory-reporting-api"></a>Solución de errores de la API de informes de Azure Active Directory

En este artículo se enumeran los mensajes de error más comunes con los que puede encontrarse al acceder a los informes de actividad mediante Microsoft Graph API y los pasos para su resolución.

### <a name="500-http-internal-server-error-while-accessing-microsoft-graph-v2-endpoint"></a>Error de servidor interno HTTP 500 al obtener acceso al punto de conexión de Microsoft Graph V2

Actualmente no se admite el punto de conexión Microsoft Graph v2; asegúrese de acceder a los registros de actividad mediante el punto de conexión Microsoft Graph v1.

### <a name="error-neither-tenant-is-b2c-or-tenant-doesnt-have-premium-license"></a>Error: El inquilino no es B2C o el inquilino no tiene licencia Premium

El acceso a los informes de inicio de sesión requiere una licencia Premium 1 (P1) de Azure Active Directory. Si ve este mensaje de error al acceder a los inicios de sesión, asegúrese de que su inquilino tiene una licencia P1 de Azure AD.

### <a name="error-user-is-not-in-the-allowed-roles"></a>Error: El usuario no está en los roles permitidos 

Si ve este mensaje de error mientras intenta acceder a los registros de auditoría o a los inicios de sesión mediante la API, asegúrese de que su cuenta forma parte del rol **Lector de seguridad** o **Lector de informes** en su inquilino de Azure Active Directory. 

### <a name="error-application-missing-aad-read-directory-data-permission"></a>Error: Falta el permiso de AAD de la aplicación "Leer datos de directorio" 

Siga los pasos del artículo [Requisitos previos para acceder a la API de generación de informes de Azure Active Directory](howto-configure-prerequisites-for-reporting-api.md) para asegurarse de que la aplicación se ejecuta con el conjunto correcto de permisos. 

### <a name="error-application-missing-microsoft-graph-api-read-all-audit-log-data-permission"></a>Error: Falta el permiso de Microsoft Graph API de la aplicación "Leer todos los datos de registro de auditoría"

Siga los pasos del artículo [Requisitos previos para acceder a la API de generación de informes de Azure Active Directory](howto-configure-prerequisites-for-reporting-api.md) para asegurarse de que la aplicación se ejecuta con el conjunto correcto de permisos. 

## <a name="next-steps"></a>Pasos siguientes

[Uso de la referencia de la API de auditoría](/graph/api/resources/directoryaudit?view=graph-rest-beta)
[Uso de la referencia de la API de informes de actividad de inicio de sesión](/graph/api/resources/signin?view=graph-rest-beta)