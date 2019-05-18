---
title: Actualización al proxy de aplicación de Azure AD | Microsoft Docs
description: Elija qué solución de proxy es la mejor si va a actualizar desde Microsoft Forefront o Unified Access Gateway.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/27/2017
ms.author: mimart
ms.reviewer: harshja
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: a98a0394ff2a74aa256637d3c12007374200ef22
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65783461"
---
# <a name="compare-remote-access-solutions"></a>Comparación de soluciones de acceso remoto

El proxy de aplicación de Azure Active Directory es una de las dos soluciones de acceso remoto que Microsoft ofrece. La otra es Web Application Proxy, la versión local. Estas dos soluciones reemplazan a los productos anteriores que ofrecía Microsoft: Microsoft Forefront Threat Management Gateway (TMG) y Unified Access Gateway (UAG). En este artículo se comparan estas cuatro soluciones. Para quienes aún utilicen las soluciones TMG o UAG en desuso, este artículo puede servir de ayuda para planear la migración a uno de los proxys de aplicación. 


## <a name="feature-comparison"></a>Comparación de características

Use esta tabla para comprender cómo se comparan Threat Management Gateway (TMG), Unified Access Gateway (UAG), Web Application Proxy (WAP) y Azure AD Application Proxy (AP) entre sí.

| Característica | TMG | UAG | WAP | AP |
| ------- | --- | --- | --- | --- |
| Autenticación de certificados | Sí | Sí | - | - |
| Publicar de forma selectiva las aplicaciones de explorador | Sí | Sí | Sí | Sí |
| Autenticación previa e inicio de sesión único | Sí | Sí | Sí | Sí | 
| Firewall de capa 2/3 | Sí | Sí | - | - |
| Funcionalidades de proxy de reenvío | Sí | - | - | - |
| Funcionalidades de VPN | Sí | Sí | - | - |
| Amplia compatibilidad con protocolos | - | Sí | Sí, si se ejecuta a través de HTTP | Sí, si ejecuta a través de HTTP o a través de Puerta de enlace de escritorio remoto |
| Actúa como servidor proxy ADFS | - | Sí | Sí | - |
| Un portal para el acceso a las aplicaciones | - | Sí | - | Sí |
| Traducción de vínculos del cuerpo de respuesta | Sí | Sí | - | Sí | 
| Autenticación con encabezados | - | Sí | - | Sí, con PingAccess | 
| Seguridad de escala en la nube | - | - | - | Sí | 
| Acceso condicional | - | Sí | - | Sí |
| No existe ningún componente en la zona desmilitarizada (DMZ) | - | - | - | Sí |
| No hay conexiones entrantes | - | - | - | Sí |

Para la mayoría de los escenarios, se recomienda a Azure AD Application Proxy como la solución actual. Web Application Proxy solo se prefiere en los escenarios que requieran un servidor proxy para AD FS y no pueda usar dominios personalizados en Azure Active Directory. 

El proxy de aplicación de Azure AD ofrece ventajas exclusivas en comparación con productos similares, incluidos:

- Extensión de Azure AD para recursos locales
   - Seguridad y protección de escala en la nube
   - Las características como el acceso condicional y la autenticación multifactor son fáciles de habilitar
- No existe ningún componente en la zona desmilitarizada
- No se requieren conexiones entrantes
- Un panel de acceso que los usuarios pueden usar para todas sus aplicaciones, incluidas Office 365, las aplicaciones SassS integradas en Azure AD y las aplicaciones web locales. 


## <a name="next-steps"></a>Pasos siguientes

- [Uso de la aplicación Azure AD para proporcionar acceso remoto seguro a aplicaciones locales](application-proxy.md)
- [Transición de Forefront TMG y UAG al proxy de aplicación](https://blogs.technet.microsoft.com/isablog/2015/06/30/modernizing-microsoft-application-access-with-web-application-proxy-and-azure-active-directory-application-proxy/).
