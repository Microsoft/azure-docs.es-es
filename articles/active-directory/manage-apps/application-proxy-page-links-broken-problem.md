---
title: Los vínculos de la página no funcionan para una aplicación de Proxy de aplicación | Microsoft Docs
description: Cómo solucionar problemas relacionados con vínculos rotos en aplicaciones de Proxy de aplicación integradas con Azure AD
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/10/2018
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: f8bb7326ed22217e56bdaf9a119529ba775b69a3
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65783241"
---
# <a name="links-on-the-page-dont-work-for-an-application-proxy-application"></a>Los vínculos de la página no funcionan para una aplicación de Proxy de aplicación

Este artículo le ayudará a entender por qué los vínculos de la aplicación Azure Active Directory Application Proxy no funcionan correctamente.

## <a name="overview"></a>Información general 
Después de publicar una aplicación de Proxy de aplicación, los únicos vínculos que funcionan de forma predeterminada son los vínculos cuyo destino aparece en la dirección URL raíz publicada. Los vínculos de las aplicaciones no funcionan, probablemente la dirección URL interna de la aplicación no incluye todos los vínculos de destino en la aplicación.

**¿Por qué ocurre esto?** Al hacer clic en un vínculo en una aplicación, el Proxy de aplicación intenta resolver la dirección URL ya sea como dirección URL interna de la misma aplicación o como dirección URL disponible externamente. Si el vínculo apunta a una dirección URL interna que no está dentro de la misma aplicación, no pertenece a ninguno de estos depósitos y se producirá un error de que no se encuentra.

## <a name="ways-you-can-resolve-broken-links"></a>Maneras de resolver vínculos rotos

Hay tres maneras de resolver este problema. Las siguientes opciones se ordenan por complejidad ascendente.

1.  Asegúrese de que la dirección URL interna es una raíz que contiene todos los vínculos relevantes para la aplicación. Esto permite que todos los vínculos se resuelvan como contenido publicado dentro de la misma aplicación.

    Si cambia la dirección URL interna pero no desea que cambie la página de aterrizaje de los usuarios, cambie la dirección URL de la página principal a la dirección URL interna publicada anteriormente. Para ello, vaya a "Azure Active Directory" -&gt; Registros de aplicaciones -&gt; seleccione la aplicación -&gt; Propiedades. En esta pestaña de propiedades, verá el campo "Dirección URL de la página principal", que podrá ajustar a la página de aterrizaje que desee.

2.  Si las aplicaciones usan nombres de dominio completos (FQDN), use [dominios personalizados](application-proxy-configure-custom-domain.md) para publicar las aplicaciones. Esta característica permite usar la misma dirección URL de manera tanto interna como externa.

    Esta opción garantiza que los vínculos de la aplicación son accesibles desde el exterior a través de Proxy de aplicación, ya que los vínculos dentro de la aplicación para las direcciones URL internas también se reconocen externamente. Todos los vínculos tienen que pertenecer a una aplicación publicada. Sin embargo, con esta opción no es necesario que los vínculos pertenezcan a la misma aplicación y pueden pertenecer a varias.

3.  Si ninguna de estas opciones es factible, existen varias opciones para habilitar la traducción de vínculos en línea. Estas opciones incluyen el uso de Intune Managed Browser, la extensión Mis aplicaciones, o bien usar la opción de traducción de vínculos en la aplicación. Para obtener más información sobre cada una de estas opciones y cómo habilitarlas, vea [Redirección de los vínculos codificados de manera rígida para las aplicaciones publicadas con el Proxy de aplicación de Azure AD](application-proxy-configure-hard-coded-link-translation.md).

## <a name="next-steps"></a>Pasos siguientes
[Trabajo con servidores proxy locales existentes](application-proxy-configure-connectors-with-proxy-servers.md)

