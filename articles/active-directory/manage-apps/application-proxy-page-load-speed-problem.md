---
title: Tiempo excesivo de carga de una aplicación con el proxy de aplicación | Microsoft Docs
description: Solución de problemas de rendimiento de carga de páginas con el Proxy de aplicación de Azure AD
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
ms.date: 07/11/2017
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4ac1182d719d7c90129115e1fadf94f4f86a28e8
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65782642"
---
# <a name="an-application-proxy-application-takes-too-long-to-load"></a>Tiempo excesivo de carga de una aplicación con el proxy de aplicación

Este artículo le ayuda a comprender por qué una aplicación de Azure AD Application Proxy puede tardar mucho tiempo en cargarse. También se explica lo que puede hacer para resolver este problema.

## <a name="overview"></a>Información general
Aunque las aplicaciones funcionan, pueden experimentar una latencia elevada. Puede haber ajustes que puede realizar en la topología de red para mejorar la velocidad. Para una evaluación de diferentes topologías, consulte el [documento de consideraciones de red](application-proxy-network-topology.md).

Además de la topología de red, no hay actualmente recomendaciones adicionales sobre el ajuste del rendimiento. A medida que el servicio Application Proxy se expande, podría llegar a un centro de datos que esté físicamente cercano. La mayor proximidad puede ayudar con la latencia. Para obtener una lista de centros de datos de Azure, consulte la [página de prueba de latencia](http://www.azurespeed.com/Azure/Latency). 

Con la [herramienta de prueba de los puertos del conector](https://aadap-portcheck.connectorporttest.msappproxy.net/) encontrará los centros de datos con el servicio de Proxy de aplicación. 

## <a name="feedback-on-application-proxy-data-center-locations"></a>Comentarios sobre las ubicaciones de los centros de datos de Proxy de aplicación 
Puede haber centros de datos de Azure que todavía no incluyan Application Proxy, pero que le supongan una mejora importante de la latencia. Envíe la ubicación del centro de datos a aadapfeedback@microsoft.com. Microsoft usa sus comentarios con fines de expansión.

Microsoft está trabajando en funcionalidades adicionales para mejorar la latencia. Tan pronto como estas mejoras estén disponibles, se actualizará la documentación.

## <a name="next-steps"></a>Pasos siguientes
[Trabajo con servidores proxy locales existentes](application-proxy-configure-connectors-with-proxy-servers.md)
