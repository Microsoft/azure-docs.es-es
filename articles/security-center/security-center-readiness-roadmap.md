---
title: Mapa de ruta de preparación para Azure Security Center | Microsoft Docs
description: Este documento le proporciona un mapa de ruta de preparación para sacar el máximo partido de Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: fece670cc-df70-445d-9773-b32cbaba8d4a
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/03/2018
ms.author: memildin
ms.openlocfilehash: 3a3dd1c0df32e7d6a67e63c9e148327d30f0082c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904725"
---
# <a name="azure-security-center-readiness-roadmap"></a>Mapa de ruta de preparación para Azure Security Center
Este documento le proporciona un mapa de ruta de preparación que le ayudará a empezar a trabajar con Azure Security Center.

## <a name="understanding-security-center"></a>Descripción de Security Center
Azure Security Center ofrece una administración de seguridad unificada y una protección contra amenazas avanzada para las cargas de trabajo que se ejecutan en Azure, de forma local y en otras nubes. 

Use los siguientes recursos para empezar a trabajar con Security Center.

Artículos
- [Introducción al Centro de seguridad de Azure](https://docs.microsoft.com/azure/security-center/security-center-intro)
- [Guía de inicio rápido de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-get-started)

Vídeos
- [Vídeo de presentación rápida](https://azure.microsoft.com/resources/videos/introduction-to-azure-security-center/)
- [Introducción a las funcionalidades de prevención, detección y respuesta de Security Center](https://azure.microsoft.com/resources/videos/azurecon-2015-new-azure-security-center-helps-you-prevent-detect-and-respond-to-threats/)

## <a name="planning-and-operations"></a>Planeamiento y operaciones

Para sacar el máximo partido a Security Center, es importante comprender cómo distintas personas o equipos de su organización usarán el servicio para satisfacer las necesidades relativas a las operaciones seguras, la supervisión, la gobernanza y la respuesta a incidentes.

Use los siguientes recursos como ayuda durante los procesos de planeamiento y de operaciones.

- [Guía de planeamiento y operaciones de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide)


### <a name="onboarding-computers-to-security-center"></a>Incorporación de equipos a Security Center
Security Center detecta automáticamente las suscripciones o áreas de trabajo de Azure que no están protegidas por Azure Defender. Aquí se incluyen las suscripciones de Azure que usan Security Center gratis y las áreas de trabajo que no tiene la solución de seguridad habilitada.

Use los siguientes recursos como ayuda durante los procesos de incorporación.

- [Incorporar equipos que no son de Azure](quickstart-onboard-machines.md)
- [Introducción sobre Azure Security Center híbrido](https://youtu.be/NMa4L_M597k)

## <a name="mitigating-security-issues-using-security-center"></a>Mitigación de problemas de seguridad mediante Security Center
Security Center recopila, analiza e integra automáticamente los datos de registro de los recursos de Azure, la red y las soluciones de asociados conectados, como firewalls y soluciones de protección de puntos de conexión, para detectar amenazas reales y reducir los falsos positivos.

Utilice los siguientes recursos como ayuda para administrar las alertas de seguridad y proteger los recursos.

Artículos    
- [Supervisión del estado de seguridad en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-monitoring)
- [Protección de las redes en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-network-recommendations)
- [Protección del servicio SQL de Azure en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-sql-service-recommendations)


Vídeo    
- [Mitigación de problemas de seguridad mediante Azure Security Center](https://channel9.msdn.com/Blogs/Azure-Security-Videos/Mitigating-Security-Issues-using-Azure-Security-Center)

### <a name="security-center-for-incident-response"></a>Security Center para respuesta a incidentes
Para reducir los costos y los daños, es importante tener un plan de respuesta a incidentes implantado antes de que se produzca un ataque. Azure Security Center puede usarse en distintas fases de una respuesta a incidentes.

Use los siguientes recursos para comprender cómo se puede incorporar Security Center al proceso de respuesta a incidentes.

Vídeos    
* [Azure Security Center in Incident Response](https://channel9.msdn.com/Blogs/Azure-Security-Videos/Azure-Security-Center-in-Incident-Response) (Uso de Azure Security Center para dar respuesta a incidentes)
* [Respond quickly to threats with next-generation security operation, and investigation](https://youtu.be/e8iFCz5RM4g) (Respuesta rápida ante amenazas mediante investigación y operaciones de última generación)

Artículos    
* [Uso de Azure Security Center para dar respuesta a incidentes](https://docs.microsoft.com/azure/security-center/security-center-incident-response)
* [Automatización de respuestas con automatización de flujos de trabajo](workflow-automation.md)

## <a name="advanced-cloud-defense"></a>Protección en la nube avanzada

Las máquinas virtuales de Azure pueden aprovechar las funcionalidades de protección en la nube avanzada de Security Center. Estas funcionalidades incluyen el acceso a máquinas virtuales "Just-In-Time" y controles de aplicación adaptables.

Use los siguientes recursos para aprender a utilizar estas funcionalidades de Security Center.

Vídeos    
* [Azure Security Center – Just-in-Time VM Access](https://youtu.be/UOQb2FcdQnU) (Azure Security Center: Acceso a máquinas virtuales "Just-in-Time")
* [Azure Security Center - Adaptive Application Controls](https://youtu.be/wWWekI1Y9ck) (Azure Security Center: Controles de aplicación adaptables)

Artículos    
* [Administración del acceso a máquina virtual mediante Just-In-Time](https://docs.microsoft.com/azure/security-center/security-center-just-in-time)
* [Controles de aplicación adaptables en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-adaptive-application)

## <a name="hands-on-activities"></a>Actividades prácticas

* [Laboratorios prácticos de Security Center](https://www.microsoft.com/handsonlabs/SelfPacedLabs/?storyGuid=78871abf-6f35-4aa0-840f-d801f5cdbd72)
* [Guión de procedimientos con recomendaciones de firewall de aplicaciones web (WAF) en Security Center](https://gallery.technet.microsoft.com/ASC-Playbook-Protect-38bd47ff)
* [Cuaderno de estrategias de Azure Security Center: alertas de seguridad](https://gallery.technet.microsoft.com/Azure-Security-Center-f621a046)

## <a name="additional-resources"></a>Recursos adicionales
* [Página Documentación de Security Center](https://docs.microsoft.com/azure/security-center/)
* [Página Documentación de API de REST de Security Center](https://msdn.microsoft.com/library/mt704034.aspx)
* [Preguntas más frecuentes sobre Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-faq)
* [Página Precios de Azure Security Center](https://azure.microsoft.com/pricing/details/security-center/)
* [Identificación de procedimientos recomendados de seguridad](https://docs.microsoft.com/azure/security/fundamentals/identity-management-best-practices)
* [Procedimientos recomendados de seguridad de red](https://docs.microsoft.com/azure/security/fundamentals/network-best-practices)
* [Recomendaciones sobre PaaS](https://docs.microsoft.com/azure/security/security-paas-deployments)
* [Cumplimiento normativo](https://www.microsoft.com/trustcenter/compliance/due-diligence-checklist)
* [Los clientes de Log Analytics pueden usar ahora Azure Security Center para proteger sus cargas de trabajo en la nube híbrida](https://blogs.technet.microsoft.com/msoms/2017/09/25/oms-customers-can-now-use-azure-security-center-to-protect-their-hybrid-cloud-workloads/).

## <a name="community-resources"></a>Recursos de la comunidad

* [Security Center UserVoice](https://feedback.azure.com/forums/347535-azure-security-center)
* [Página de preguntas y respuestas sobre Security Center](https://docs.microsoft.com/answers/topics/azure-security-center.html)