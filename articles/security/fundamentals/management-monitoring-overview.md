---
title: 'Administración y supervisión de las características de seguridad: Microsoft Azure | Microsoft Docs'
description: En este artículo se proporciona información general sobre los servicios y características de seguridad que proporciona Azure para ayudarle a administrar y supervisar servicios en la nube y máquinas virtuales de Azure.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 5cf2827b-6cd3-434d-9100-d7411f7ed424
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/28/2019
ms.author: terrylan
ms.openlocfilehash: 7e6fe4f0db31687d402f960f2752fe021d3ec9d3
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87448358"
---
# <a name="azure-security-management-and-monitoring-overview"></a>Información general sobre la administración y la supervisión de la seguridad en Azure
En este artículo se proporciona información general sobre los servicios y características de seguridad que proporciona Azure para ayudarle a administrar y supervisar servicios en la nube y máquinas virtuales de Azure.

## <a name="role-based-access-control"></a>Control de acceso basado en roles

El control de acceso basado en roles (RBAC) le permite administrar de forma avanzada el acceso a los recursos de Azure. Gracias a RBAC, puede conceder a los usuarios únicamente el grado de acceso que necesitan para realizar sus trabajos. Asimismo, RBAC le ayudará a garantizar que cuando los usuarios dejan la organización, pierden el acceso a los recursos en la nube.

Más información:

* [Blog del equipo de Active Directory sobre RBAC](https://cloudblogs.microsoft.com/enterprisemobility/?product=azure-active-directory)
* [Control de acceso basado en roles de Azure (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md)

## <a name="antimalware"></a>Antimalware

Con Azure, puede usar el software antimalware que ofrecen los principales proveedores de seguridad como Microsoft, Symantec, Trend Micro, McAfee y Kaspersky. Este software le ayudará a proteger las máquinas virtuales de archivos malintencionados, adware y otras amenazas.

Microsoft Antimalware para Azure Cloud Services y Virtual Machines le ofrece la posibilidad de instalar un agente antimalware tanto para roles PaaS como para máquinas virtuales. Esta característica, basada en System Center Endpoint Protection, lleva a la nube tecnología de seguridad local de demostrada eficacia.

También ofrecemos una profunda integración con los productos [Deep Security](https://www.trendmicro.com/us/enterprise/cloud-solutions/deep-security/) y [SecureCloud](https://www.trendmicro.com/us/enterprise/cloud-solutions/secure-cloud/) en la plataforma de Azure. Deep Security es una solución antivirus y SecureCloud es una solución de cifrado. Deep Security se implementa dentro de máquinas virtuales mediante un modelo de extensión. Gracias a la interfaz de usuario de Azure Portal y PowerShell, puede usar Deep Security en máquinas virtuales nuevas que se estén implementando, o en máquinas virtuales existentes que ya están implementadas.

Symantec Endpoint Protection (SEP) también se admite en Azure. Mediante la integración del portal, tiene la posibilidad de especificar su intención de usar SEP en una máquina virtual. SEP puede instalarse en una máquina virtual nueva a través de Azure Portal, o bien en una máquina virtual existente mediante PowerShell.

Más información:

* [Implementación de soluciones antimalware en Azure Virtual Machines](https://azure.microsoft.com/blog/deploying-antimalware-solutions-on-azure-virtual-machines/)
* [Microsoft Antimalware para Azure Cloud Services y Virtual Machines](antimalware.md)
* [Instalación y configuración de Trend Micro Deep Security como servicio en una máquina virtual de Azure](/azure/virtual-machines/windows/classic/install-trend)
* [Instalación y configuración de Endpoint Protection en una máquina virtual de Azure](/azure/virtual-machines/windows/classic/install-symantec)
* [New Antimalware Options for Protecting Azure Virtual Machines](https://azure.microsoft.com/blog/new-antimalware-options-for-protecting-azure-virtual-machines/) (Nuevas opciones de antimalware para proteger máquinas virtuales de Azure)

## <a name="multi-factor-authentication"></a>Multi-Factor Authentication

Azure Multi-Factor Authentication es un método de autenticación que requiere el uso de más de un método de comprobación. Proporciona una segunda capa de seguridad crítica a los inicios de sesión y transacciones de los usuarios.

Multi-Factor Authentication le ayudará a proteger el acceso a los datos y las aplicaciones, además de satisfacer la demanda de los usuarios de un proceso de inicio de sesión simple. Asimismo, proporciona autenticación sólida mediante una gran variedad de opciones de comprobación, como llamadas telefónicas, mensajes de texto, notificaciones de aplicaciones móviles, códigos de verificación y tokens OATH de terceros.

Más información:

* [Multi-Factor Authentication](https://azure.microsoft.com/documentation/services/multi-factor-authentication/)
* [¿Qué es Azure Multi-Factor Authentication?](/azure/active-directory/authentication/multi-factor-authentication)
* [Cómo funciona Azure Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md)

## <a name="expressroute"></a>ExpressRoute

Puede usar Azure ExpressRoute para ampliar sus redes locales en Microsoft Cloud a través de una conexión privada y dedicada que facilita un proveedor de conectividad. Con ExpressRoute se pueden establecer conexiones a servicios en la nube de Microsoft, como Azure, Office 365 y CRM Online. La conectividad puede provenir de:

* Una red cualquiera (IP VPN).
* Una red de ethernet de punto a punto.
* Una conexión cruzada virtual a través de un proveedor de conectividad en una instalación de colocalización.

Conexiones de ExpressRoute que no fluyen a través de la red pública de Internet. Ofrecen más confiabilidad, velocidades más altas, menores latencias y mayor seguridad que las conexiones habituales a través de Internet.

Más información:

* [Información técnica de ExpressRoute](../../expressroute/expressroute-introduction.md)

## <a name="virtual-network-gateways"></a>Puertas de enlace de red virtual

Las puertas de enlace de VPN, también llamadas puertas de enlace de Azure Virtual Network, se usan para enviar tráfico de red entre las redes virtuales y las ubicaciones locales. También se usan para enviar el tráfico entre varias redes virtuales dentro de Azure (de red a red). Las puertas de enlace de VPN proporcionan conectividad local segura entre Azure y su infraestructura.

Más información:

* [Información acerca de las puertas de enlace de VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md)
* [Azure Network Security Overview (Información general sobre Azure Network Security)](network-overview.md)

## <a name="privileged-identity-management"></a>Privileged Identity Management

En ocasiones, los usuarios necesitan llevar a cabo operaciones con privilegios en recursos de Azure u otras aplicaciones para SaaS. A menudo esto significa que las organizaciones tienen que concederles acceso con privilegios permanente en Azure Active Directory (Azure AD).

Este hecho representa un riesgo creciente para la seguridad de los recursos hospedados en la nube, ya que las organizaciones no pueden supervisar lo suficiente todo lo que hacen esos usuarios con su acceso privilegiado. Además, si se pone en peligro una cuenta de usuario que tiene acceso con privilegios, esta situación podría afectar a la seguridad de la organización en la nube global. Privileged Identity Management de Azure AD ayuda a resolver este riesgo al reducir el tiempo de exposición de los privilegios y aumentar la visibilidad sobre el uso.  

Privileged Identity Management introduce el concepto de administrador temporal para un rol o acceso de administrador "just in time". Este tipo de administrador es un usuario que necesita completar un proceso de activación para ese rol asignado. El proceso de activación cambia la asignación del usuario a un rol en Azure AD de inactiva a activa durante un período de tiempo especificado.

Más información:

* [Azure AD Privileged Identity Management](../../active-directory/privileged-identity-management/pim-configure.md)
* [Introducción a Privileged Identity Management de Azure AD](../../active-directory/privileged-identity-management/pim-getting-started.md)

## <a name="identity-protection"></a>Protección de identidad

Azure AD Identity Protection proporciona una vista consolidada de actividades de inicio de sesión sospechosas y posibles vulnerabilidades que ayudan a proteger su negocio. Identity Protection detecta actividades sospechosas en los usuarios e identidades con privilegios (administradores), en función de señales como:

* Ataques por fuerza bruta.
* Filtración de credenciales.
* Inicios de sesión desde ubicaciones desconocidas y dispositivos infectados.

Al proporcionar notificaciones y soluciones recomendadas, Identity Protection ayuda a mitigar los riesgos en tiempo real. Calcula la gravedad del riesgo para el usuario. Permite configurar directivas basadas en el riesgo que ayudan a proteger automáticamente el acceso a la aplicación frente a futuras amenazas.

Más información:

* [Azure Active Directory Identity Protection](/azure/active-directory/active-directory-identityprotection)
* [Channel 9: Azure AD and Identity Show: Identity Protection Preview](https://channel9.msdn.com/Series/Azure-AD-Identity/Azure-AD-and-Identity-Show-Identity-Protection-Preview)

## <a name="security-center"></a>Security Center

Azure Security Center ayuda a evita y a detectar las amenazas, además de a responder a ellas. Security Center aporta visibilidad mejorada y control sobre la seguridad de los recursos de Azure. Proporciona una supervisión de la seguridad y una administración de directivas integradas en suscripciones de Azure. Le ayuda a detectar amenazas que podrían pasar desapercibidas, y funciona con un amplio ecosistema de soluciones de seguridad.

Security Center ayuda a optimizar y supervisar la seguridad de los recursos de Azure de la manera siguiente:

* Le permite definir directivas para los recursos de suscripción de Azure de acuerdo con:
  * Las necesidades de seguridad de la empresa.
  * El tipo de aplicaciones o confidencialidad de los datos en cada suscripción.
* Supervisa el estado de las máquinas virtuales, las redes y las aplicaciones de Azure.
* Proporciona una lista de alertas de seguridad clasificadas por orden de prioridad, incluyendo alertas de soluciones de socios integradas. Asimismo, también proporciona la información que necesita para investigar rápidamente un ataque y recomendaciones sobre cómo remediarlo.

Más información:

* [Introducción al Centro de seguridad de Azure](../../security-center/security-center-intro.md)
* [Mejorar la puntuación segura de Azure Security Center](../../security-center/security-center-secure-score.md)

## <a name="intelligent-security-graph"></a>Intelligent Security Graph

Intelligent Security Graph proporciona una protección contra amenazas en tiempo real en los servicios y productos de Microsoft. Utiliza análisis avanzados que vinculan una enorme cantidad de datos de inteligencia y seguridad contra amenazas para proporcionar información detallada que pueda reforzar la seguridad de la organización. Microsoft utiliza análisis avanzados que procesan más de 450 000 millones de autenticaciones al mes, buscan malware y suplantaciones de identidad (phishing) en 400 000 de correos electrónicos y actualizan 1000 millones de dispositivos. Esta información puede ayudar a su organización a detectar y responder rápidamente a los ataques.

* [Intelligent Security Graph](https://www.microsoft.com/security/intelligence)

## <a name="next-steps"></a>Pasos siguientes
Información sobre el [modelo de responsabilidad compartida](shared-responsibility.md) y las tareas de seguridad que administra Microsoft y las que administra el usuario.

Para obtener más información sobre la administración de la seguridad, consulte [Administración de la seguridad en Azure](management.md).
