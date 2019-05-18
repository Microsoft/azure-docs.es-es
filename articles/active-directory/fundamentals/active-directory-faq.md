---
title: 'Preguntas más frecuentes: Azure Active Directory | Microsoft Docs'
description: Preguntas y respuestas frecuentes sobre Azure y Azure Active Directory, administración de contraseñas y acceso a las aplicaciones.
services: active-directory
author: eross-msft
manager: daveba
ms.assetid: b8207760-9714-4871-93d5-f9893de31c8f
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 11/12/2018
ms.author: lizross
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 06112361434a8217df6714a70d0059c175e98ac8
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65521716"
---
# <a name="frequently-asked-questions-about-azure-active-directory"></a>Preguntas más frecuentes sobre Azure Active Directory
Azure Active Directory (Azure AD) es una completa solución de identidad como servicio (IDaaS) que abarca todos los aspectos de la identidad, la administración de acceso y la seguridad.

Para más información, consulte [¿Qué es Azure Active Directory?](active-directory-whatis.md)


## <a name="access-azure-and-azure-active-directory"></a>Acceso a Azure y a Azure Active Directory
**P: ¿Por qué veo "No se encontraron suscripciones" cuando intento acceder a Azure AD en Azure Portal?**

**R:** Para acceder a Azure Portal, los usuarios necesitan permiso con una suscripción de Azure. Si tiene una suscripción de pago a Office 365 o Azure AD, vaya a [https://aka.ms/accessAAD](https://aka.ms/accessAAD) para realizar el paso de la activación única. En caso contrario, deberá activar una [cuenta de Azure](https://azure.microsoft.com/pricing/free-trial/) gratis o una suscripción de pago.

Para más información, consulte:

* [Asociación de las suscripciones de Azure con Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)

- - -
**P: ¿Cuál es la relación entre Azure AD, Office 365 y Azure?**

**R:** Azure AD ofrece funcionalidades de acceso e identidad comunes a todos los servicios web. Tanto si usa Office 365 como Microsoft Azure, Intune u otros servicios, ya está utilizando Azure AD para habilitar el inicio de sesión y la administración del acceso de todos estos servicios.

Todos los usuarios que están configurados para usar servicios web se definen como cuentas de usuario en una o varias instancias de Azure AD. Puede configurar gratis en estas cuentas las funcionalidades de Azure AD, como el acceso a aplicaciones en la nube.

Los servicios de pago de Azure AD, como Enterprise Mobility + Security, complementan otros servicios web como Office 365 y Microsoft Azure con completas soluciones de administración y seguridad para empresas.

- - -

**P:  ¿Cuáles son las diferencias entre el propietario y el administrador global?**

**R:** De forma predeterminada, a la persona que se suscribe a una suscripción a Azure se le asigna el rol de propietario para los recursos de Azure. Un propietario puede usar una cuenta de Microsoft o una cuenta profesional o educativa del directorio al que está asociada la suscripción a Azure.  Este rol tiene autorización para administrar servicios en Azure Portal.

Si otros usuarios necesitan iniciar sesión y acceder a los servicios con la misma suscripción, puede asignarles el [rol integrado](../../role-based-access-control/built-in-roles.md) adecuado. Para obtener información adicional, consulte [Administración del acceso mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md).

De forma predeterminada, a la persona que se suscribe a una suscripción a Azure se le asigna el rol de administrador global para el directorio. El administrador global tiene acceso a todas las características del directorio de Azure AD. Azure AD tiene un conjunto diferente de roles de administrador para gestionar las características relacionadas con la identidad y el directorio. Estos administradores tendrán acceso a varias características de Azure Portal. El rol de administrador determina qué puede hacer, como crear o editar usuarios, asignar roles administrativos a otros, restablecer contraseñas de usuario, administrar licencias de usuario o administrar dominios.  Para obtener más información sobre los administradores de directorios de Azure AD y sus roles, consulte [Asignación de un usuario a roles de administrador en Azure Active Directory](active-directory-users-assign-role-azure-portal.md) y [Asignación de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md).

Además, los servicios de pago de Azure AD, como Enterprise Mobility + Security, complementan otros servicios web como Office 365 y Microsoft Azure con completas soluciones de administración y seguridad para empresas.

- - -
**P: ¿Existe un informe que muestra cuándo expirarán mis licencias de usuario de Azure AD?**

**R:** No.  No está disponible actualmente.

- - -

## <a name="get-started-with-hybrid-azure-ad"></a>Introducción a Azure AD híbrido


**P: ¿Cómo dejo un inquilino cuando se me ha agregado como colaborador?**

**R:** Cuando se agrega a otro inquilino de la organización como colaborador, puede usar el "cambio de inquilino " de la esquina superior derecha para cambiar entre los inquilinos.  Actualmente, no hay forma de abandonar la organización que invita y Microsoft está trabajando para proporcionar esta funcionalidad.  Hasta que esta característica esté disponible, puede pedir a la organización que invita que le quite de su inquilino.
- - -
**P: ¿Cómo conecto mi directorio local a Azure AD?**

**R:** Puede conectar su directorio local a Azure AD mediante Azure AD Connect.

Para más información, consulte [Integración de las identidades locales con Azure Active Directory](../hybrid/whatis-hybrid-identity.md).

- - -
**P: ¿Cómo se configura el SSO entre mi directorio local y las aplicaciones de nube?**

**R:** Solo es preciso configurar el inicio de sesión único (SSO) entre el directorio local y Azure AD. Mientras tenga acceso a las aplicaciones en la nube mediante Azure AD, el servicio lleva automáticamente a los usuarios a autenticarse correctamente con sus credenciales locales.

La implementación del SSO desde el entorno local se puede lograr fácilmente con soluciones de federación como Active Directory Federation Services (AD FS) o configurando la sincronización de la sincronización de hash de contraseña. Puede implementar fácilmente ambas opciones con el Asistente para configuración de Azure AD Connect.

Para más información, consulte [Integración de las identidades locales con Azure Active Directory](../hybrid/whatis-hybrid-identity.md).

- - -
**P: ¿Proporciona Azure AD un portal de autoservicio para usuarios en mi organización?**

**R:** Sí, Azure AD proporciona el [panel de acceso de Azure AD](https://myapps.microsoft.com) para el autoservicio de los usuarios y el acceso a las aplicaciones. Si es un cliente de Office 365, puede encontrar muchas de las mismas funcionalidades en el [portal de Office 365](https://portal.office.com).

Para más información, consulte [Introducción al Panel de acceso](../user-help/active-directory-saas-access-panel-introduction.md).

- - -
**P: ¿Me ayuda Azure AD a administrar la infraestructura local?**

**R:** Sí. Azure AD Premium Edition incluye Azure AD Connect Health. Azure AD Connect Health le ayuda a supervisar y a comprender mejor su infraestructura de identidad local y los servicios de sincronización.  

Para más información, consulte [Supervisión de la infraestructura de identidad local y los servicios de sincronización en la nube](../hybrid/whatis-hybrid-identity-health.md).  

- - -
## <a name="password-management"></a>Administración de contraseñas
**P: ¿Puedo usar la escritura diferida de contraseñas de Azure AD sin sincronización de contraseñas? (En este escenario, ¿se puede usar el restablecimiento de contraseña de autoservicio de Azure AD (SSPR) con la escritura diferida de contraseñas y no almacenar las contraseñas en la nube?)**

**R:** No es necesario sincronizar las contraseñas de Active Directory con Azure AD para habilitar la escritura diferida. En un entorno federado, el inicio de sesión único (SSO) de Azure AD se basa en el directorio local para autenticar al usuario. En este caso no se necesita la contraseña local para realizar el seguimiento en Azure AD.

- - -
**P: ¿Cuánto se tarda en escribir una contraseña en diferido en Active Directory local?**

**R:** La escritura diferida de contraseñas funciona en tiempo real.

Para más información, consulte [Introducción a la administración de contraseñas](../authentication/quickstart-sspr.md).

- - -
**P: ¿Puedo usar la escritura diferida de contraseñas con las que administra un administrador?**

**R:** Sí, si tiene la escritura diferida de contraseñas habilitada, las operaciones de contraseña realizadas por un administrador se escriben de manera diferida en el entorno local.  

Para ver más respuestas a preguntas relativas a las contraseñas, consulte [Preguntas más frecuentes sobre la administración de contraseñas](../authentication/active-directory-passwords-faq.md).
- - -
**P:  ¿Qué hago si no recuerdo mi contraseña de Office 365 o Azure AD cuando intento cambiarla?**

**R:** En este tipo de situaciones, dispone de un par de opciones.  Use el restablecimiento de la contraseña de autoservicio (SSPR), si está disponible.  El funcionamiento de SSPR dependerá de cómo esté configurado.  Para más información, consulte la sección [¿Cómo funciona el portal de restablecimiento de contraseñas?](../authentication/howto-sspr-deployment.md)

Para los usuarios de Office 365, el administrador puede restablecer la contraseña mediante los pasos que se describen en [Administradores: restablecer contraseñas de usuario](https://support.office.com/article/Admins-Reset-user-passwords-7A5D073B-7FAE-4AA5-8F96-9ECD041ABA9C?ui=en-US&rs=en-US&ad=US).

Para las cuentas de Azure AD, los administradores pueden restablecer las contraseñas mediante uno de los siguientes procedimientos:

- [Restablecimiento de cuentas en Azure Portal](active-directory-users-reset-password-azure-portal.md)
- [Uso de PowerShell](/powershell/module/msonline/set-msoluserpassword?view=azureadps-1.0)


- - -
## <a name="security"></a>Seguridad
**P: ¿Se bloquean las cuentas después de un número determinado de intentos erróneos o se usa una estrategia más sofisticada?**

Usamos una estrategia más sofisticada para bloquear cuentas, que  se basa en la dirección IP de la solicitud y las contraseñas escritas. El tiempo que dure el bloqueo también aumenta en función de la probabilidad de que sea un ataque.  

**P:  Algunas contraseñas (comunes) se rechazan con mensajes del tipo "esta contraseña se ha usado demasiadas veces". ¿Se refiere esto a las contraseñas usadas en la instancia de Active Directory actual?**

Se refiere a las contraseñas que son comunes a nivel global, como las variantes de "Contraseña" y "123456".

**P: ¿Se bloqueará una solicitud de inicio de sesión de origen dudoso (botnets, punto de conexión Tor) en un inquilino B2C o se requerirá un inquilino de la edición Básica o Premium?**

Tenemos una puerta de enlace que filtra las solicitudes y proporciona alguna protección contra los botnets, y se aplica a todos los inquilinos B2C.

## <a name="application-access"></a>Acceso a la aplicación

**P: ¿Dónde puedo encontrar una lista de las aplicaciones preintegradas en Azure AD y sus funcionalidades?**

**R:** Azure AD cuenta con más de 2 600 aplicaciones preintegradas de Microsoft, proveedores de servicios de aplicaciones y asociados. Todas las aplicaciones preintegradas deben ser compatibles con el inicio de sesión único (SSO). El SSO permite utilizar las credenciales de su organización para acceder a las aplicaciones. Algunas de las aplicaciones también admiten el aprovisionamiento y el desaprovisionamiento automatizados.

Para ver una lista exhaustiva de las aplicaciones preintegradas, consulte [Active Directory Marketplace](https://azure.microsoft.com/marketplace/active-directory/).

- - -
**P: ¿Qué ocurre si no encuentro la aplicación que necesito en Marketplace de Azure AD?**

**R:** Con Azure AD Premium puede agregar y configurar cualquier aplicación. Según las funcionalidades de la aplicación y sus preferencias, puede configurar el SSO y el aprovisionamiento automatizado.  

Para más información, consulte:

* [Configuración del inicio de sesión único en aplicaciones que no están en la Galería de aplicaciones de Azure Active Directory](../manage-apps/configure-federated-single-sign-on-non-gallery-applications.md)
* [Uso de SCIM para habilitar el aprovisionamiento automático de usuarios y grupos de Azure Active Directory a aplicaciones](../manage-apps/use-scim-to-provision-users-and-groups.md)

- - -
**P: ¿Cómo inician los usuarios sesión en aplicaciones con Azure AD?**

**R:** Azure AD proporciona varias formas de que los usuarios vean y accedan a sus aplicaciones, como:

* El Panel de acceso de Azure AD
* El iniciador de aplicaciones de Office 365
* Inicio de sesión directo en aplicaciones federadas
* Vínculos profundos a aplicaciones federadas, con contraseña o existentes

Para obtener más información, consulte [Experiencias de usuario final para aplicaciones](../manage-apps/end-user-experiences.md).

- - -
**P: ¿Cuáles son las distintas formas en que Azure AD permite la autenticación y el inicio de sesión único en aplicaciones?**

**R:** Azure AD admite muchos protocolos estandarizados para la autenticación y la autorización, como SAML 2.0, OpenID Connect, OAuth 2.0 y WS-Federation. Además, Azure AD admite las funcionalidades de inicio de sesión automatizado y de almacenamiento de contraseñas para las aplicaciones que solo sean compatibles con la autenticación basada en formularios.  

Para más información, consulte:

* [Escenarios de autenticación para Azure AD](../develop/authentication-scenarios.md)
* [Protocolos de autenticación de Active Directory](https://msdn.microsoft.com/library/azure/dn151124.aspx)
* [Inicio de sesión único para aplicaciones de Azure AD](../manage-apps/what-is-single-sign-on.md)

- - -
**P: ¿Puedo agregar aplicaciones que ejecuto de manera local?**

**R:** Azure Active Directory Application Proxy proporciona un acceso fácil y seguro a las aplicaciones web locales que elija. Puede acceder a estas aplicaciones como si se trataran de sus aplicaciones de software como servicio (SaaS) en Azure AD. No se necesita una VPN ni cambiar la infraestructura de red.  

Para más información, consulte [Provisión de acceso remoto seguro a aplicaciones locales](../manage-apps/application-proxy.md).

- - -
**P: ¿Cómo requiero la autenticación multifactor para usuarios con acceso a una aplicación determinada?**

**R:** Con el acceso condicional de Azure AD, puede asignar una directiva de acceso única a cada aplicación. En la directiva, puede solicitar la autenticación multifactor siempre o solo cuando los usuarios no estén conectados a la red local.  

Para más información, consulte [Protección del acceso a Office 365 y otras aplicaciones conectadas a Azure Active Directory](../active-directory-conditional-access-azure-portal.md).

- - -
**P: ¿Qué es el aprovisionamiento automático de usuarios para aplicaciones SaaS?**

**R:** Use Azure AD para automatizar la creación, el mantenimiento y la eliminación de identidades de usuario en muchas aplicaciones SaaS en la nube conocidas.

Para más información, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../manage-apps/user-provisioning.md)

- - -
**P:  ¿Puedo configurar una conexión LDAP segura con Azure Active Directory?**

**R:**  No. Azure AD no admite el protocolo de protocolo ligero de acceso a directorios (LDAP) o el protocolo LDAP seguro directamente. Sin embargo, es posible habilitar la instancia de Azure AD Domain Services (Azure AD DS) en su inquilino de Azure AD con grupos de seguridad de red configuradas correctamente a través de redes de Azure para lograr la conectividad LDAP. Para obtener más información, consulta https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-admin-guide-configure-secure-ldap.
