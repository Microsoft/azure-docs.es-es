---
title: Referencia de la guía de operaciones generales de Azure Active Directory
description: Esta guía de referencia de operaciones se describen las comprobaciones y las acciones que debe realizar para proteger las operaciones generales.
services: active-directory
author: martincoetzer
manager: daveba
tags: azuread
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: fundamentals
ms.date: 10/31/2019
ms.author: martinco
ms.openlocfilehash: edb35986fcfc0e8855fa712b519ee7a8ca2f0ed8
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321963"
---
# <a name="azure-active-directory-general-operations-guide-reference"></a>Referencia de la guía de operaciones generales de Azure Active Directory

En esta sección de la [guía de referencia de operaciones de Azure AD](active-directory-ops-guide-intro.md) se describen las comprobaciones y las acciones que debe realizar para optimizar las operaciones generales de Azure Active Directory (Azure AD).

> [!NOTE]
> Estas recomendaciones están actualizadas hasta la fecha de publicación, pero pueden cambiar con el tiempo. Las organizaciones deben evaluar continuamente sus prácticas de operaciones a medida que los productos y servicios de Microsoft evolucionen con el tiempo.

## <a name="key-operational-processes"></a>Procesos operativos clave

### <a name="assign-owners-to-key-tasks"></a>Asignación de propietarios a las tareas clave

La administración de Azure Active Directory requiere la ejecución continua de tareas y procesos operativos clave que pueden no formar parte de un proyecto de lanzamiento. Aun así, es importante que configure estas tareas con miras a optimizar su entorno. Entre las tareas clave y sus propietarios recomendados se incluyen:

| Tarea | Propietario |
| :- | :- |
| Mejoras en la puntuación de seguridad de la identidad | Equipo de operaciones de InfoSec |
| Mantenimiento de servidores Azure AD Connect | Equipo de operaciones IAM |
| Ejecutar y evaluar periódicamente los informes de IdFix | Equipo de operaciones IAM |
| Evaluación de prioridades de las alertas de Azure AD Connect Health para la sincronización y AD FS | Equipo de operaciones IAM |
| Si no usa Azure AD Connect Health, el cliente tiene procesos y herramientas equivalentes para supervisar la infraestructura personalizada | Equipo de operaciones IAM |
| Si no usa AD FS, el cliente tiene procesos y herramientas equivalentes para supervisar la infraestructura personalizada | Equipo de operaciones IAM |
| Supervisar registros híbridos: Conectores de Azure AD Application Proxy | Equipo de operaciones IAM |
| Supervisar registros híbridos: Agentes para la autenticación de paso a través | Equipo de operaciones IAM |
| Supervisar registros híbridos: Servicio de escritura diferida de contraseñas | Equipo de operaciones IAM |
| Supervisar registros híbridos: Puerta de enlace de protección con contraseña local | Equipo de operaciones IAM |
| Supervisar registros híbridos: Extensión NPS para Azure MFA (si es aplicable) | Equipo de operaciones IAM |

A medida que revise la lista, es posible que tenga que asignar un propietario a las tareas que no tienen uno o ajustar la propiedad de aquellas tareas con propietarios que no coincidan con las recomendaciones anteriores.

#### <a name="owners-recommended-reading"></a>Lectura recomendada para propietarios

- [Asignación de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md)
- [Gobernanza en Azure](../../governance/index.yml)

## <a name="hybrid-management"></a>Administración híbrida

### <a name="recent-versions-of-on-premises-components"></a>Versiones recientes de componentes locales

Tener las versiones más actualizadas de los componentes locales proporciona al cliente todas las actualizaciones de seguridad más recientes, mejoras de rendimiento, así como funcionalidades que podrían ayudarle a simplificar aún más el entorno. La mayoría de los componentes tienen una configuración de actualización automática, que automatiza el proceso de actualización.

Estos componentes incluyen:

- Azure AD Connect
- Conectores de Azure AD Application Proxy
- Agentes de autenticación de paso a través de Azure AD
- Agentes de Azure AD Connect Health

A menos que se haya establecido uno, debe definir un proceso para actualizar estos componentes y así poder basarse en la característica de actualización automática siempre que sea posible. Si encuentra componentes que tienen seis o más meses de retraso, debe realizar la actualización lo antes posible.

#### <a name="hybrid-management-recommended-reading"></a>Lectura recomendada de administración híbrida

- [Azure AD Connect: actualización automática](../hybrid/how-to-connect-install-automatic-upgrade.md)
- [Descripción de los conectores de Azure AD Application Proxy | Actualizaciones automáticas](../manage-apps/application-proxy-connectors.md#automatic-updates)

### <a name="azure-ad-connect-health-alert-baseline"></a>Línea de base de las alertas de Azure AD Connect Health

Las organizaciones deben implementar [Azure AD Connect Health](../hybrid/whatis-azure-ad-connect.md#what-is-azure-ad-connect-health) para la supervisión y la generación de informes de Azure AD Connect y AD FS. Azure AD Connect y AD FS son componentes esenciales que pueden afectar a la administración y la autenticación del ciclo de vida y, por ello, provocar interrupciones. Azure AD Connect Health ayuda a supervisar y a conocer mejor su infraestructura de identidad local, lo que garantiza la confiabilidad de este entorno.

![Arquitectura de Azure AD Connect Health](./media/active-directory-ops-guide/active-directory-ops-img16.png)

A medida que supervisa el estado de su entorno, debe hacerse cargo inmediatamente cualquier alerta de gravedad alta, además de las alertas de gravedad inferior.

#### <a name="azure-ad-connect-health-recommended-reading"></a>Lectura recomendada de Azure AD Connect Health

- [Instalación del agente de Azure AD Connect Health](../hybrid/how-to-connect-health-agent-install.md)

### <a name="on-premises-agents-logs"></a>Registros de los agentes locales

Algunos servicios de administración de identidades y acceso requieren agentes locales para habilitar escenarios híbridos. Algunos ejemplos son el restablecimiento de contraseña, la autenticación de paso a través (PTA), Azure AD Application Proxy y la extensión NPS de Azure MFA. Es fundamental que el equipo de operaciones establezca la línea de base y supervise el estado de estos componentes archivando y analizando los registros del agente de componentes, gracias a soluciones como System Center Operations Manager o SIEM. Es igualmente importante que el equipo de operaciones de INFOSEC o el Departamento de soporte técnico sepan cómo solucionar problemas de patrones de errores.

#### <a name="on-premises-agents-logs-recommended-reading"></a>Lectura recomendada de registros de agentes locales

- [Solucionar problemas de Proxy de aplicación](../manage-apps/application-proxy-troubleshoot.md)
- [Solución de problemas del autoservicio de restablecimiento de contraseña: Azure Active Directory](../authentication/active-directory-passwords-troubleshoot.md#password-writeback-event-log-error-codes)
- [Descripción de los conectores del Proxy de aplicación de Azure AD](../manage-apps/application-proxy-connectors.md)
- [Azure AD Connect: Solución de problemas de autenticación de paso a través](../hybrid/tshoot-connect-pass-through-authentication.md#collecting-pass-through-authentication-agent-logs)
- [Solución de problemas de códigos de error para la extensión de NPS de Azure MFA](../authentication/howto-mfa-nps-extension-errors.md)

### <a name="on-premises-agents-management"></a>Administración de los agentes locales

La adopción de procedimientos recomendados puede ayudarle a optimizar el funcionamiento de los agentes locales. Puede usar los siguientes procedimientos recomendados:

- Se recomiendan varios conectores de Azure AD Application Proxy por grupo de conectores, para proporcionar un equilibrio de carga continuo y una alta disponibilidad evitando puntos únicos de error al obtener acceso a las aplicaciones proxy. Si actualmente solo tiene un conector en un grupo de conectores que controla las aplicaciones en producción, debe implementar al menos dos conectores para la redundancia.
- Crear y usar un grupo de conectores del proxy de aplicación para la depuración puede serle de utilidad en escenarios de solución de problemas y para incorporar nuevas aplicaciones locales. También es recomendable instalar herramientas de red como el Analizador de mensajes y Fiddler en los equipos de conector.
- Asimismo, se recomiendan varios agentes de autenticación de paso a través para proporcionar un equilibrio de carga sin problemas y una alta disponibilidad, ya que así podrá evitar un único punto de error durante el flujo de autenticación. Asegúrese de implementar al menos dos agentes de autenticación de paso a través en la redundancia.

#### <a name="on-premises-agents-management-recommended-reading"></a>Lectura recomendada para la administración de agentes locales

- [Descripción de los conectores del Proxy de aplicación de Azure AD](../manage-apps/application-proxy-connectors.md)
- [Autenticación de paso a través de Azure AD: inicio rápido](../hybrid/how-to-connect-pta-quick-start.md#step-4-ensure-high-availability)

## <a name="management-at-scale"></a>Administración a escala

### <a name="identity-secure-score"></a>Puntuación segura de identidad

La [puntuación de seguridad de la identidad](./identity-secure-score.md) proporciona una medida cuantificable de la posición de seguridad de su organización. Es fundamental revisar y abordar constantemente los hallazgos de los que tenga información, y procurar tener la máxima puntuación posible. La puntuación le ayuda a:

- Medir de forma objetiva su nivel de seguridad de la identidad
- Planear la realización de mejoras en la seguridad de la identidad
- Ver si las mejoras han logrado sus objetivos

![Puntuación segura](./media/active-directory-ops-guide/active-directory-ops-img17.png)

Si su organización no tiene ningún programa para supervisar los cambios en la puntuación de seguridad de la identidad, le recomendamos que implemente un plan y asigne propietarios para supervisar y controlar las acciones de mejora. Recuerde que las organizaciones deben corregir las acciones de mejora con un impacto de puntuación superior a 30 lo antes posible.

### <a name="notifications"></a>Notificaciones

Microsoft envía estas notificaciones por correo electrónico a los administradores, para informarles de los distintos cambios en el servicio, las actualizaciones de configuración necesarias y los errores que requieren la intervención del administrador. Es importante que los clientes escriban sus direcciones de correo electrónico de notificación, para que los mensajes se envíen a los miembros del equipo adecuados y que así puedan confirmar todas las notificaciones y actuar en consecuencia. Le recomendamos agregar varios destinatarios al [Centro de mensajes de Office 365](/office365/admin/manage/message-center) y solicitar que las notificaciones (incluidas las notificaciones de Azure AD Connect Health) se envíen a una lista de distribución o a un buzón compartido. Si solo tiene una cuenta de administrador global con una dirección de correo electrónico, asegúrese de configurar al menos dos cuentas compatibles con ese correo electrónico.

Existen dos direcciones de "remitente" que usa Azure AD: <o365mc@email2.microsoft.com>, que envía notificaciones del Centro de mensajes de Office 365 y <azure-noreply@microsoft.com>, que envía notificaciones relacionadas con lo siguiente:

- [Revisiones de acceso de Azure AD](../governance/access-reviews-overview.md)
- [Azure AD Connect Health](../hybrid/how-to-connect-health-operations.md#enable-email-notifications)
- [Azure AD Identity Protection](../identity-protection/howto-identity-protection-configure-notifications.md)
- [Azure AD Privileged Identity Management](../privileged-identity-management/pim-email-notifications.md)
- [Notificaciones de certificado que expirarán con una aplicación empresarial](../manage-apps/manage-certificates-for-federated-single-sign-on.md#add-email-notification-addresses-for-certificate-expiration)
- Aplicación empresarial: aprovisionamiento de las notificaciones de servicio

Consulte la tabla siguiente para obtener información sobre el tipo de notificaciones que se envían y dónde buscarlas:

| Origen de la notificación | Qué se envía | Dónde buscar |
|:-|:-|:-|
| Contacto técnico | Errores de sincronización | Azure Portal: hoja de propiedades |
| Centro de mensajes de Office 365 | Avisos de incidentes y degradación de servicios de identidad y servicios back-end de O365 | Portal de Office |
| Resumen semanal de Identity Protection | Resumen de Identity Protection | Hoja de Azure AD Identity Protection |
| Azure AD Connect Health | Notificaciones de alerta | Azure Portal: hoja de Azure AD Connect Health |
| Notificaciones de aplicaciones empresariales | Notificaciones acerca de cuándo expirarán los certificados y aprovisionamiento de errores | Azure Portal: hoja de aplicaciones empresariales (cada aplicación tiene su propia configuración de dirección de correo electrónico) |

#### <a name="notifications-recommended-reading"></a>Lectura recomendada de notificaciones

- [Cambie la dirección de la organización, el contacto técnico y más - Office 365](/office365/admin/manage/change-address-contact-and-more)

## <a name="operational-surface-area"></a>Área expuesta operativa

### <a name="ad-fs-lockdown"></a>Bloqueo de AD FS

Las organizaciones que configuran aplicaciones para que se autentiquen directamente en Azure AD se benefician del  [bloqueo inteligente de Azure AD](../authentication/concept-sspr-howitworks.md). Si usa AD FS en Windows Server 2012 R2, implemente la  [protección de bloqueo de extranet](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-soft-lockout-protection) de AD FS. Si usa AD FS en Windows Server 2016 o posterior, implemente el  [bloqueo inteligente de extranet](https://support.microsoft.com/help/4096478/extranet-smart-lockout-feature-in-windows-server-2016). Como mínimo, se recomienda que habilite el bloqueo de la extranet para poder contener el riesgo de ataques por fuerza bruta contra la instancia local de Active Directory. Sin embargo, si tiene AD FS en Windows 2016 o superior, también debe habilitar el bloqueo inteligente de la extranet que le ayudará a mitigar los ataques de [difusión de contraseñas](https://www.microsoft.com/microsoft-365/blog/2018/03/05/azure-ad-and-adfs-best-practices-defending-against-password-spray-attacks/).

Si AD FS solo se usa para la federación de Azure AD, hay algunos puntos de conexión que se pueden desactivar para minimizar el área expuesta a ataques. Por ejemplo, si AD FS solo se usa para Azure AD, debe deshabilitar los distintos puntos de conexión de WS-Trust de aquellos puntos de conexión habilitados para los elementos **usernamemixed** y **windowstransport**.

### <a name="access-to-machines-with-on-premises-identity-components"></a>Acceso a equipos con componentes de identidad locales

Las organizaciones deben bloquear el acceso a las máquinas con componentes híbridos locales de la misma manera que el dominio local. Por ejemplo, un operador de copia de seguridad o un administrador de Hyper-V no debe poder iniciar sesión en el servidor de Azure AD Connect para cambiar las reglas.

Este modelo de niveles administrativo de Active Directory se diseñó para proteger los sistemas de identidad mediante un conjunto de zonas de búfer que se encuentran entre el control total del entorno (nivel 0) y los recursos de la estación de trabajo de alto riesgo que los atacantes suelen poner en peligro. ![Diagrama que muestra las tres capas del modelo en niveles](./media/active-directory-ops-guide/active-directory-ops-img18.png)

El [modelo de niveles](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material) se compone de tres niveles y solo incluye las cuentas administrativas, no las cuentas de usuario estándar.

- **Nivel 0** : control directo de las identidades de empresa en el entorno. El nivel 0 incluye cuentas, grupos y otros recursos que tienen control administrativo directo o indirecto de los bosques, dominios o controladores de dominio de Active Directory y todos los recursos que haya en él. La sensibilidad de la seguridad de todos los recursos de nivel 0 es equivalente, ya que todos se controlan de forma efectiva entre sí.
- **Nivel 1** : control de aplicaciones y servidores empresariales. Los recursos de nivel 1 incluyen sistemas operativos de servidor, servicios en la nube y aplicaciones empresariales. Las cuentas de administrador de nivel 1 tienen el control administrativo de una cantidad considerable de valor empresarial que se hospeda en estos recursos. Una función común de ejemplo son los administradores de servidores que mantienen estos sistemas operativos con la capacidad de afectar a todos los servicios de empresa.
- **Nivel 2** : control de los dispositivos y estaciones de trabajo de usuarios. Las cuentas de administrador de nivel 2 tienen el control administrativo de una cantidad considerable de valor empresarial que se hospeda en dispositivos y estaciones de trabajo de usuarios. Algunos ejemplos son el departamento de soporte técnico y los administradores de soporte técnico del equipo, porque pueden afectar a la integridad de casi cualquier dato de usuario.

Bloquee el acceso a componentes de identidad locales tales como Azure AD Connect, AD FS y servicios de SQL, de la misma manera que lo hace en los controladores de dominio.

## <a name="summary"></a>Resumen

Una infraestructura de identidad segura está conformada por siete aspectos. Esta lista le ayudará a encontrar las acciones que debe realizar para optimizar las operaciones de Azure Active Directory (Azure AD).

- Asignar propietarios a las tareas principales.
- Automatice el proceso de actualización de los componentes híbridos locales.
- Implemente Azure AD Connect Health para supervisar y generar informes de Azure AD Connect y AD FS.
- Supervise el estado de los componentes híbridos locales archivando y analizando los registros del agente de componentes mediante System Center Operations Manager o una solución SIEM.
- Implemente mejoras de seguridad midiendo su posición de seguridad mediante la puntuación segura de identidad.
- Bloquee AD FS.
- Bloquee el acceso a equipos con componentes de identidad locales

## <a name="next-steps"></a>Pasos siguientes

Consulte los [planes de implementación de Azure AD](active-directory-deployment-plans.md) para obtener detalles de la implementación de cualquier funcionalidad que no haya implementado.