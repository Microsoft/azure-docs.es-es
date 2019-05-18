---
title: ¿Qué es Azure AD Connect y Connect Health? | Microsoft Docs
description: Describe las herramientas utilizadas para sincronizar y supervisar el entorno local con Azure AD.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 02/26/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 48b81d508711f35a75efe1c93fe0a5556c5bb960
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784456"
---
# <a name="what-is-azure-ad-connect"></a>¿Qué es Azure AD Connect?

Azure AD Connect es la herramienta de Microsoft diseñada para satisfacer y lograr sus objetivos de identidad híbrida.  Ofrece las siguientes características:
    
- [Sincronización de hash de contraseñas](whatis-phs.md): un método de inicio de sesión que sincroniza el hash de la contraseña de un usuario de AD local con Azure AD.
- [Autenticación de paso a través](how-to-connect-pta.md): un método de inicio de sesión que permite a los usuarios usar la misma contraseña de forma local y en la nube, pero que no requiere la infraestructura adicional de un entorno federado.
- [Integración de federación](how-to-connect-fed-whatis.md): la federación es una parte opcional de Azure AD Connect y puede utilizarse para configurar un entorno híbrido mediante una infraestructura local de AD FS. También proporciona funcionalidades de administración de AD FS, como la renovación de certificados e implementaciones de servidor de AD FS adicionales.
- [Sincronización](how-to-connect-sync-whatis.md): responsable de la creación de usuarios, grupos y otros objetos.  También de asegurar que la información de identidad de los usuarios y los grupos de su entorno local coincide con la de la nube.  Esta sincronización también incluye los códigos hash de contraseña.
-   [Seguimiento de estado](whatis-hybrid-identity-health.md): Azure AD Connect Health puede proporcionar una sólida supervisión y una ubicación central en Azure Portal donde se puede ver esta actividad. 


![Qué es Azure AD Connect](./media/whatis-hybrid-identity/arch.png)



## <a name="what-is-azure-ad-connect-health"></a>¿Qué es Azure AD Connect Health?

Azure Active Directory (Azure AD) Connect Health proporciona una sólida supervisión de la infraestructura de identidad local. Permite mantener una conexión confiable con Office 365 y Microsoft Online Services.  Esta confiabilidad se consigue al proporcionar funcionalidades de supervisión para los componentes de identidad clave. Además, hace que los puntos de datos clave sobre estos componentes sean fácilmente accesibles.

Esta información se encuentra en el [portal de Azure AD Connect Health](https://aka.ms/aadconnecthealth). Utilice el portal de Azure AD Connect Health para ver alertas, supervisión del rendimiento y análisis de uso, entre otra información. Azure AD Connect Health habilita la lente de mantenimiento única para los componentes de identidad clave en un lugar único.

![Qué es Azure AD Connect Health](./media/whatis-hybrid-identity-health/aadconnecthealth2.png)

## <a name="why-use-azure-ad-connect"></a>¿Por qué usar Azure AD Connect?
la integración de directorios locales con Azure AD hace que los usuarios sean más productivos proporcionando una identidad común para tener acceso a recursos de nube y locales. Los usuarios y las organizaciones pueden aprovechar lo siguiente:

* Los usuarios pueden usar una única identidad para acceder a aplicaciones locales y servicios en la nube, como Office 365.
* Una sola herramienta que proporciona una experiencia de implementación simplificada para la sincronización y el inicio de sesión.
* Proporciona las funcionalidades más novedosas para sus escenarios de trabajo. Azure AD Connect sustituye a las versiones anteriores de las herramientas de integración de identidades, como DirSync y Sincronización de Azure AD. Para más información, consulte [Identidades híbridas: comparación de las herramientas para la integración de directorios de identidades híbridas](plan-hybrid-identity-design-considerations-tools-comparison.md).

## <a name="why-use-azure-ad-connect-health"></a>Motivos para usar Azure AD Connect Health
Cuando se integran con Azure AD, los usuarios son más productivos porque hay una identidad común para acceder tanto a los recursos en la nube como a los locales. Asegurar que el entorno sea confiable, para que los usuarios puedan acceder a estos recursos, se convierte en un reto.  Azure AD Connect Health ayuda a supervisar y a conocer mejor su infraestructura de identidad local, lo que garantiza la confiabilidad de este entorno. Es tan sencillo como instalar un agente en cada uno de los servidores de identidad locales.

Azure AD Connect Health para AD FS es compatible con AD FS 2.0 en Windows Server 2008 R2, Windows Server 2012, Windows Server 2012 R2 y Windows Server 2016. También permite supervisar el proxy de AD FS o los servidores proxy de la aplicación web que proporcionan compatibilidad con la autenticación para el acceso a la extranet. Con una instalación sencilla y rápida del agente de mantenimiento, Azure AD Connect Health para AD FS proporciona el siguiente conjunto de funcionalidades clave.

Principales ventajas y procedimientos recomendados:

|Ventajas principales|Prácticas recomendadas|
|-----|-----|
|Mayor seguridad|[Tendencias de bloqueo de extranet](how-to-connect-health-adfs.md#usage-analytics-for-ad-fs)</br>[Informe de inicios de sesión erróneos](how-to-connect-health-adfs-risky-ip.md)</br>[Cumple con los requisitos de privacidad](reference-connect-health-user-privacy.md)|
|Recepción de alertas de [todos los problemas críticos del sistema de ADFS](how-to-connect-health-alert-catalog.md#alerts-for-active-directory-federation-services)|Disponibilidad y configuración del servidor</br>[Rendimiento y conectividad](how-to-connect-health-adfs.md#performance-monitoring-for-ad-fs)</br>Mantenimiento periódico|
|Fácil de implementar y administrar|[Rápida instalación del agente](how-to-connect-health-agent-install.md#installing-the-azure-ad-connect-health-agent-for-ad-fs)</br>Actualización automática del agente a la versión más reciente</br>Datos disponibles en el portal en cuestión de minutos|
[Métricas de uso](how-to-connect-health-adfs.md#usage-analytics-for-ad-fs) enriquecidas|Uso de las aplicaciones principales</br>Ubicaciones de red y conexión TCP</br>Solicitudes de tokens por servidor|
|Mejor experiencia del usuario|Modo de panel de Azure Portal</br>[Alertas a través de mensajes de correo electrónico](how-to-connect-health-adfs.md#alerts-for-ad-fs)|


## <a name="license-requirements-for-using-azure-ad-connect"></a>Requisitos de licencia para usar Azure AD Connect

[!INCLUDE [active-directory-free-license.md](../../../includes/active-directory-free-license.md)]




## <a name="next-steps"></a>Pasos siguientes

- [Hardware y requisitos previos](how-to-connect-install-prerequisites.md) 
- [Configuración rápida](how-to-connect-install-express.md)
- [Configuración personalizada](how-to-connect-install-custom.md)
- [Instalación de agentes de Azure AD Connect Health](how-to-connect-health-agent-install.md) 
