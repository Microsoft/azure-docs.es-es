---
title: 'Proxy de aplicación de Azure AD ofrece estas ventajas: Historial de lanzamiento de versiones | Microsoft Docs'
description: En este artículo se enumera todas las versiones de Azure AD Application Proxy y se describe las nuevas características y se han corregido problemas
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/05/2019
ms.subservice: manage-apps
ms.author: mimart
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7af05769861f98ec192789d90f8a61f5b6638d1d
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65783286"
---
# <a name="azure-ad-application-proxy-version-release-history"></a>Proxy de aplicación de Azure AD ofrece estas ventajas: Historial de lanzamiento de versiones
En este artículo se enumera las versiones y las características de Azure Active Directory (Azure AD) Application Proxy que se han publicado. El equipo de Azure AD actualiza periódicamente el Proxy de aplicación con nuevas características y funcionalidades. Los conectores del Proxy de aplicación se actualizan automáticamente cuando se lanza una nueva versión.

Presentamos una lista de los recursos relacionados:

Recurso |  Detalles
--------- | --------- |
Cómo habilitar el Proxy de aplicación | Requisitos previos para habilitar el Proxy de aplicación, instalar y registrar un conector se describen en este [tutorial](application-proxy-add-on-premises-application.md).
Descripción de los conectores del Proxy de aplicación de Azure AD | Obtenga más información sobre [administración del conector](application-proxy-connectors.md) y cómo los conectores [actualización automática](application-proxy-connectors.md#automatic-updates).
Descargue el conector del Proxy de aplicación de Azure AD |  [Descargue la versión más reciente connector](https://download.msappproxy.net/subscription/d3c8b69d-6bf7-42be-a529-3fe9c2e70c90/connector/download).

## <a name="156120"></a>1.5.612.0

### <a name="release-status"></a>Estado de la versión

20 de septiembre de 2018: publicado para descarga.

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras

- Se agregó compatibilidad con WebSocket para la aplicación de QlikSense. Para obtener más información sobre cómo integrar QlikSense con el Proxy de aplicación, consulte este [tutorial](application-proxy-qlik.md). 
- Ha mejorado el Asistente para la instalación para que sea más fácil de configurar a un proxy de salida. 
- Establecer TLS 1.2 como el protocolo predeterminado para los conectores. 
- Agrega un nuevo contrato de licencia del usuario final (CLUF).  

### <a name="fixed-issues"></a>Problemas corregidos

- Se ha corregido un error que causaba algunas pérdidas de memoria en el conector.
- Actualiza la versión de Azure Service Bus, que incluye una corrección de errores para los problemas de tiempo de espera del conector.

## <a name="154020"></a>1.5.402.0

### <a name="release-status"></a>Estado de la versión

19 de enero de 2018: publicado para descarga.

### <a name="fixed-issues"></a>Problemas corregidos

- Se ha agregado compatibilidad con dominios personalizados que necesite la traducción de dominio en la cookie.

## <a name="151320"></a>1.5.132.0

### <a name="release-status"></a>Estado de la versión 

25 de mayo de 2017: publicado para descarga. 

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras 

Mejor control sobre los límites de conexiones salientes de los conectores. 

## <a name="15360"></a>1.5.36.0

### <a name="release-status"></a>Estado de la versión

15 de abril de 2017: publicado para descarga.

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras

- Simplifica la incorporación y la administración con menos puertos necesarios. Proxy de aplicación ahora requiere abrir sólo dos puertos de salida estándares: 80 y 443. Proxy de aplicación sigue usando conexiones de solo salida, por lo que todavía no es necesario ningún componente en una red Perimetral. Para obtener más información, consulte nuestra [documentación sobre la configuración](application-proxy-add-on-premises-application.md).  
- Si admite el proxy externo o el firewall, puede abrir ahora la red con DNS en lugar de intervalo de direcciones IP. Servicios de Proxy de aplicación requieren conexiones a *. msappproxy.net y *. servicebus.windows.net solo.


## <a name="earlier-versions"></a>Versiones anteriores

Si usa una versión del conector de Proxy de aplicación anteriores a 1.5.36.0, actualice a la versión más reciente para asegurarse de que la versión más reciente características admitidas por completo.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre [el acceso remoto a aplicaciones locales a través de Azure AD Application Proxy](application-proxy.md).
- Para empezar a usar Application Proxy, consulte [Tutorial: Adición de una aplicación local para el acceso remoto mediante Application Proxy](application-proxy-add-on-premises-application.md).
