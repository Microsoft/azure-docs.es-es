---
title: Procedimientos recomendados de seguridad de acceso e identidad de Azure | Microsoft Docs
description: Este artículo proporciona un conjunto de procedimientos recomendados para la administración de identidades y la seguridad del control de acceso mediante las funcionalidades integradas de Azure.
services: security
documentationcenter: na
author: terrylanfear
manager: RKarlin
editor: TomSh
ms.assetid: 07d8e8a8-47e8-447c-9c06-3a88d2713bc1
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/28/2019
ms.author: terrylan
ms.openlocfilehash: 852a8c8eb7e6a87739c58967eefd6c18e996b225
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87367408"
---
# <a name="azure-identity-management-and-access-control-security-best-practices"></a>Procedimientos recomendados para la administración de identidades y la seguridad del control de acceso en Azure

En este artículo, se trata un conjunto de procedimientos recomendados para la seguridad del control de acceso y la administración de identidades en Azure. Estos procedimientos recomendados proceden de nuestra experiencia con [Azure AD](../../active-directory/fundamentals/active-directory-whatis.md) y las experiencias de clientes como usted.

Para cada procedimiento recomendado, explicaremos:

* Qué es el procedimiento recomendado
* Por qué le conviene habilitar este procedimiento recomendado
* Cuál podría ser el resultado si no habilita el procedimiento recomendado
* Alternativas posibles al procedimiento recomendado
* Cómo aprender a habilitar el procedimiento recomendado

Este artículo sobre procedimientos recomendados para la seguridad del control de acceso y la administración de identidades en Azure se basa en una opinión consensuada y en las funcionalidades y conjuntos de características de la plataforma de Azure que existen en el momento de su publicación.

Este artículo se escribió con el fin de proporcionar una guía general para obtener una postura de seguridad más sólida después de la implementación guiada por nuestra lista de comprobación "[Cinco pasos para asegurar su infraestructura de identidad](steps-secure-identity.md)", que le guía por los servicios y características principales.

Las opiniones y las tecnologías cambian con el tiempo, por lo que se actualizará de forma periódica para reflejar esos cambios.

Los procedimientos recomendados para la seguridad del control de acceso y la administración de identidades en Azure que se describen en este artículo son:

* Tratar las amenazas como el perímetro de seguridad principal
* Centralizar la administración de identidades
* Administrar inquilinos conectados
* Habilitar el inicio de sesión único
* Activar el acceso condicional
* Planeación de mejoras de seguridad rutinarias
* Habilitación de la administración de contraseñas
* Exigir a los usuarios la verificación multifactor
* Uso del control de acceso basado en rol
* Menor exposición de las cuentas con privilegios
* Controlar las ubicaciones donde se encuentran los recursos
* Uso de Azure AD para la autenticación de almacenamiento

## <a name="treat-identity-as-the-primary-security-perimeter"></a>Tratar las amenazas como el perímetro de seguridad principal

Muchos consideran que la identidad es el perímetro principal para la seguridad. Se trata de un cambio desde el enfoque tradicional de seguridad de red. Los perímetros de red continúan volviéndose más porosos y esa defensa perimetral no puede ser tan eficaz como lo era antes de la explosión de dispositivos y aplicaciones en la nube de [BYOD](https://aka.ms/byodcg).

[Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md) es la solución de Azure para la administración de identidades y de acceso. Azure AD es un servicio de administración de identidades y directorios multiinquilino basado en la nube de Microsoft. Combina servicios de directorio fundamentales, la administración del acceso a las aplicaciones y la protección de identidades en una única solución.

En las siguientes secciones se enumeran los procedimientos recomendados de seguridad de acceso e identidades con Azure AD.

**Procedimiento recomendado**: Centre los controles y las detecciones de seguridad en torno a las identidades de usuario y servicio.
**Detalles**: Use Azure AD para colocar los controles e identidades.

## <a name="centralize-identity-management"></a>Centralizar la administración de identidades

En un escenario de [identidad híbrida](https://resources.office.com/ww-landing-M365E-EMS-IDAM-Hybrid-Identity-WhitePaper.html?), se recomienda integrar los directorios en el entorno local y en la nube. La integración permite al equipo de TI administrar las cuentas desde una ubicación, independientemente de donde se crea una cuenta. La integración también hace que los usuarios sean más productivos, ya que proporciona una identidad común para tener acceso a recursos de la nube y del entorno local.

**Procedimiento recomendado**: establecer una única instancia de Azure AD. La coherencia y tener un solo origen de autoridad aumentará la claridad y reducirá los riesgos de seguridad originados por errores humanos, así como la complejidad de la configuración.
**Detalles**: designe un solo directorio de Azure AD como origen de autoridad de cuentas corporativas y de la organización.

**Procedimiento recomendado**: Integración de los directorios locales con Azure AD.  
**Detalles**: use [Azure AD Connect](/azure/active-directory/connect/active-directory-aadconnect) para sincronizar el directorio local con el directorio en la nube.

> [!Note]
> Existen [factores que afectan al rendimiento de una instancia de Azure AD Connect](../../active-directory/hybrid/plan-connect-performance-factors.md). Asegúrese de que Azure AD Connect tiene capacidad suficiente para impedir que los sistemas con un rendimiento deficiente obstaculicen la seguridad y productividad. Las organizaciones grandes o complejas (organizaciones que aprovisionan más de 100 000 objetos) deben seguir las [recomendaciones](../../active-directory/hybrid/whatis-hybrid-identity.md) para optimizar su implementación de Azure AD Connect.

**Procedimiento recomendado**: no sincronizar las cuentas de Azure AD que tienen privilegios elevados en la instancia de Active Directory existente.
**Detalles**: no cambie el valor predeterminado de la [configuración de Azure AD Connect](../../active-directory/hybrid/how-to-connect-sync-configure-filtering.md) que filtra estas cuentas. Esta configuración reduce el riesgo de que un adversario pase de la nube a los recursos locales (lo que podría crear un incidente de primera magnitud).

**Procedimiento recomendado**: Activación de la sincronización de hashes de contraseñas.  
**Detalles**: la sincronización de hash de contraseñas es una característica que sirve para sincronizar hash de contraseñas de usuario de una instancia de Active Directory local con otra de Azure Active Directory (Azure AD) basada en la nube. Esta sincronización ayuda a protegerse de la reutilización de credenciales filtradas obtenidas en ataques anteriores.

Aunque decida usar la federación con Servicios de federación de Active Directory (AD FS) u otros proveedores de identidades, si quiere, puede configurar la sincronización de hashes de contraseñas para tener una opción alternativa si los servidores locales sufren un error o dejan de estar disponibles temporalmente. Esta sincronización permite que los usuarios inicien sesión en el servicio con la misma contraseña que usan para iniciar sesión en su instancia local de Active Directory. También permite que Identity Protection detecte las credenciales que están en peligro mediante la comparación de los hash de contraseña sincronizados con contraseñas que se sepa que están en peligro, si un usuario ha usado su misma dirección de correo electrónico y contraseña en otros servicios que no estén conectados a Azure AD.

Para más información, consulte [Implementación de la sincronización de hash de contraseñas con la sincronización de Azure AD Connect](/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-hash-synchronization).

**Procedimiento recomendado**: Para el desarrollo de nuevas aplicaciones, usar Azure AD para la autenticación.
**Detalles**: use las capacidades adecuadas para admitir la autenticación:

  - Azure AD para los empleados.
  - [B2B de Azure AD](../../active-directory/b2b/index.yml) para los usuarios invitados y socios externos.
  - [Azure AD B2C](../../active-directory-b2c/index.yml) para controlar el modo en que los clientes se suscriben, inician sesión y administran sus perfiles al usar las aplicaciones.

Las organizaciones que no integren la identidad del entorno local con la identidad en la nube pueden tener mayor sobrecarga para administrar cuentas. Esta sobrecarga aumenta la probabilidad de que haya errores e infracciones de seguridad.

> [!Note]
> Debe elegir en qué directorios van a residir las cuentas críticas y si la estación de trabajo de administración empleada se administra mediante servicios en la nube nuevos o mediante procesos ya existentes. Usar los procesos de aprovisionamiento de identidades y administración existentes puede reducir algunos riesgos, pero también puede crear el riesgo de que un atacante ponga en peligro una cuenta local y pase a la nube. Es posible que desee usar una estrategia diferente para distintos roles (por ejemplo, administradores de TI frente a administradores de unidades de negocio). Tiene dos opciones. La primera es crear cuentas de Azure AD que no estén sincronizadas con la instancia local de Active Directory. Una su estación de trabajo de administración a Azure AD, que se puede administrar y donde se pueden aplicar revisiones mediante Microsoft Intune. La segunda consiste en usar cuentas de administrador existentes mediante la sincronización de la instancia de Active Directory local. Use estaciones de trabajo existentes en el dominio de Active Directory para la administración y seguridad.

## <a name="manage-connected-tenants"></a>Administrar inquilinos conectados
Su organización de seguridad necesita visibilidad para evaluar los riesgos y determinar si se siguen las directivas de la organización, así como cualquier requisito normativo. Debe asegurarse de que su organización de seguridad tiene visibilidad en todas las suscripciones conectadas al entorno de producción y a la red (a través de [Azure ExpressRoute](../../expressroute/expressroute-introduction.md) o [VPN de sitio a sitio](../../vpn-gateway/vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)). Un [administrador global o un administrador de empresa](../../active-directory/users-groups-roles/directory-assign-admin-roles.md#company-administrator-permissions) en Azure AD puede elevar su acceso al rol [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator) y ver todas las suscripciones y los grupos administrados conectados al entorno.

Vea [Elevación de los privilegios de acceso para administrar todas las suscripciones y los grupos de administración de Azure](../../role-based-access-control/elevate-access-global-admin.md) para asegurarse de que usted y su grupo de seguridad pueden ver todas las suscripciones o grupos de administración conectados al entorno. Deberá quitar este acceso con privilegios elevados después de haber evaluado los riesgos.

## <a name="enable-single-sign-on"></a>Habilitar el inicio de sesión único

En un mundo donde la nube y la movilidad son lo primero, resulta útil habilitar el inicio de sesión único (SSO) en dispositivos, aplicaciones y servicios desde cualquier sitio, de modo que los usuarios puedan ser productivos en cualquier momento y lugar. Cuando es necesario administrar varias soluciones de identidad, esto supone un problema administrativo no solo para TI, sino también para los usuarios que tendrán que recordar varias contraseñas.

Mediante el uso de la misma solución de identidad para todas las aplicaciones y los recursos, podrá disfrutar del SSO. Además, los usuarios podrán usar el mismo conjunto de credenciales para iniciar sesión y acceder a los recursos que necesitan, con independencia de dónde se ubiquen estos recursos, tanto si es en el entorno local como en la nube.

**Procedimiento recomendado**: habilitar SSO.  
**Detalles**: Azure AD [extiende Active Directory del entorno local](/azure/active-directory/connect/active-directory-aadconnect) a la nube. Los usuarios pueden usar su cuenta profesional o educativa principal para los dispositivos unidos a un dominio, los recursos de la empresa y todas las aplicaciones web y SaaS que necesitan para realizar su trabajo. Los usuarios no tienen que recordar varios nombres de usuario y contraseñas y el acceso a las aplicaciones por parte de los usuarios se puede aprovisionar (o desaprovisionar) automáticamente, en función de su pertenencia a los grupos de la organización y de su estado como empleado. Además, puede controlar el acceso de las aplicaciones de la galería o de sus propias aplicaciones locales que ha desarrollado y publicado mediante el [proxy de la aplicación de Azure AD](/azure/active-directory/active-directory-application-proxy-get-started).

Use el SSO para permitir que los usuarios accedan a sus [aplicaciones SaaS](/azure/active-directory/active-directory-appssoaccess-whatis) en función de su cuenta profesional o educativa en Azure AD. Esto no solo es aplicable a las aplicaciones para SaaS de Microsoft, sino también a otras aplicaciones, como [Google Apps](/azure/active-directory/active-directory-saas-google-apps-tutorial) y [Salesforce](/azure/active-directory/active-directory-saas-salesforce-tutorial). Puede configurar su aplicación de modo que use Azure AD como un proveedor de [identidades basado en SAML](/azure/active-directory/fundamentals-identity). Como control de seguridad, Azure AD no emite ningún token que permita a los usuarios iniciar sesión en la aplicación, a menos que se les conceda acceso mediante Azure AD. Puede concederles acceso directamente o a través de un grupo al cual pertenezcan.

Las organizaciones que no crean ninguna identidad común para establecer el SSO para sus usuarios y aplicaciones están más expuestas a escenarios donde los usuarios tienen varias contraseñas. Estos escenarios aumentan la probabilidad de que los usuarios reutilicen las contraseñas o usen contraseñas débiles.

## <a name="turn-on-conditional-access"></a>Activar el acceso condicional

Los usuarios pueden acceder a los recursos de su organización mediante diversos dispositivos y aplicaciones desde cualquier lugar. Como administrador de TI, quiere asegurarse de que estos dispositivos cumplan los estándares de seguridad y cumplimiento. Ya no es suficiente con centrarse en quién puede acceder a un recurso.

Para equilibrar la seguridad y la productividad, también debe pensar en cómo se accede a un recurso antes de que pueda tomar una decisión de control de acceso. Con el acceso condicional de Azure AD, puede abordar este requisito. Con el acceso condicional, puede tomar decisiones de control de acceso automatizadas en función de las condiciones para acceder a las aplicaciones en la nube.

**Procedimiento recomendado**: Administración y control del acceso a los recursos corporativos.  
**Detalles**: configure las [directivas de acceso condicional](../../active-directory/conditional-access/concept-conditional-access-policy-common.md) comunes de Azure AD en función del grupo, la ubicación y la confidencialidad de las aplicaciones SaaS y las aplicaciones conectadas a Azure AD.

**Procedimiento recomendado**: bloquear los protocolos de autenticación heredados.
**Detalles**: Los atacantes aprovechan a diario los puntos débiles de protocolos anteriores, especialmente en ataques de difusión de contraseña. Configure el acceso condicional para que [bloquear los protocolos heredados](../../active-directory/conditional-access/howto-conditional-access-policy-block-legacy.md).

## <a name="plan-for-routine-security-improvements"></a>Planeación de mejoras de seguridad rutinarias

La seguridad está en constante evolución y es importante integrar en su plataforma de administración de identidades y la nube una manera de mostrar periódicamente el crecimiento y descubrir nuevas formas de proteger su entorno.

La puntuación de seguridad de la identidad es un conjunto de controles de seguridad recomendados que Microsoft publica para ofrecer una puntuación numérica que mide objetivamente su postura de seguridad y ayuda a planear futuras mejoras de seguridad. También puede comparar su puntuación con las de otros sectores, así como con sus propias tendencias a lo largo del tiempo.

**Procedimiento recomendado**: Planee revisiones y mejoras de seguridad rutinarias basadas en los procedimientos recomendados del sector.
**Detalles**: Use la característica de puntuación de seguridad de la identidad para clasificar las mejoras a lo largo del tiempo.

## <a name="enable-password-management"></a>Habilitación de la administración de contraseñas

Si tiene varios inquilinos o quiere permitir que los usuarios [restablezcan su propia contraseña](../../active-directory/user-help/active-directory-passwords-update-your-own-password.md), es importante utilizar directivas de seguridad adecuadas para evitar un uso inadecuado.

**Procedimiento recomendado**: Configuración del autoservicio de restablecimiento de contraseña (SSPR) para los usuarios.  
**Detalles**: use la característica de [autoservicio de restablecimiento de contraseña](/azure/active-directory-b2c/active-directory-b2c-reference-sspr) de Azure AD.

**Procedimiento recomendado**: Supervisar cómo se usa realmente SSPR o si se puede usar.  
**Detalles**: supervise los usuarios que se registran mediante el [informe de actividad de registro de restablecimiento de contraseña](/azure/active-directory/active-directory-passwords-get-insights) de Azure AD. La característica de creación de informes que proporciona Azure AD le ayuda a responder preguntas mediante informes creados previamente. Si está debidamente protegido por licencia, también puede crear consultas personalizadas.

**Procedimiento recomendado**: ampliar las directivas de contraseña basadas en la nube a la infraestructura local.
**Detalles**: mejore las directivas de contraseña en la organización realizando en el entorno local las mismas comprobaciones de cambios de contraseña que las que se efectúan en la nube. Instale [Protección con contraseña de Azure AD](/azure/active-directory/authentication/concept-password-ban-bad) en agentes de Windows Server Active Directory de forma local para ampliar las listas de contraseñas prohibidas a la infraestructura existente. Los usuarios y administradores que cambien, establezcan o restablezcan contraseñas locales deben cumplir la misma directiva de contraseñas que los usuarios que solo están en la nube.

## <a name="enforce-multi-factor-verification-for-users"></a>Exigir a los usuarios la verificación multifactor

Se recomienda exigir la verificación en dos pasos a todos los usuarios. Esto incluye a los administradores y otras personas de su organización, ya que el hecho de que su cuenta esté en peligro puede tener un impacto significativo (por ejemplo, los directores financieros).

Existen varias opciones para exigir la verificación en dos pasos. La mejor opción para usted depende de sus objetivos, la edición de Azure AD que ejecuta y su programa de licencias. Consulte [Exigencia de verificación en dos pasos para un usuario](/azure/active-directory/authentication/howto-mfa-userstates) para determinar la mejor opción para usted. Puede encontrar más información sobre licencias y precios en las páginas de precios de [Azure AD](https://azure.microsoft.com/pricing/details/active-directory/) y [Azure Multi-Factor Authentication](https://azure.microsoft.com/pricing/details/multi-factor-authentication/).

A continuación, se indican las opciones y ventajas para habilitar la verificación en dos pasos:

**Opción 1**: Habilite MFA para todos los usuarios y métodos de inicio de sesión con la **Ventaja** de los valores predeterminados de seguridad de Azure AD: Esta opción le permite aplicar de forma rápida y sencilla MFA para todos los usuarios de su entorno con una directiva estricta para:

* Desafiar a cuentas administrativas y mecanismos de inicio de sesión administrativo;
* Solicitar el desafío de MFA a través de Microsoft Authenticator para todos los usuarios;
* Restringir los protocolos de autenticación heredados.

Este método está disponible para todos los niveles de licencia, pero no se puede mezclar con las directivas de acceso condicional existentes. Puede encontrar más información en los [valores predeterminados de seguridad de Azure AD](/azure/active-directory/fundamentals/concept-fundamentals-security-defaults).

**Opción 2**: [habilitar Multi-factor Authentication mediante el cambio de estado de usuario](../../active-directory/authentication/howto-mfa-userstates.md).   
**Ventaja**: este es el método tradicional para exigir la verificación en dos pasos. Funciona tanto con [Azure Multi-factor Authentication en la nube como en el Servidor Azure Multi-factor Authentication](/azure/active-directory/authentication/concept-mfa-whichversion). El uso de este método requiere que los usuarios realicen la verificación en dos pasos cada vez que inicien sesión, e invalida las directivas de acceso condicional.

Para averiguar dónde debe habilitarse Multi-Factor Authentication, vea [¿Qué versión de Azure MFA es adecuada en mi organización?](/azure/active-directory/authentication/concept-mfa-whichversion)

**Opción 3**: [habilitar Multi-Factor Authentication con la directiva de acceso condicional](/azure/active-directory/authentication/howto-mfa-getstarted).
**Ventaja**: esta opción permite solicitar la verificación en dos pasos en condiciones específicas mediante el uso del [acceso condicional](../../active-directory/conditional-access/concept-conditional-access-policy-common.md). Las condiciones específicas pueden ser el inicio de sesión del usuario desde distintas ubicaciones, dispositivos no confiables o aplicaciones que considere de riesgo. Definir condiciones específicas donde exija la verificación en dos pasos le permite evitar pedirla constantemente a los usuarios, lo cual puede ser una experiencia desagradable para el usuario.

Esta es la forma más flexible de habilitar la verificación en dos pasos para los usuarios. Habilitar la directiva de acceso condicional solo funciona con Azure Multi-Factor Authentication en la nube y es una característica premium de Azure AD. Puede encontrar más información sobre este método en [Implementación de Azure Multi-factor Authentication en la nube](/azure/active-directory/authentication/howto-mfa-getstarted).

**Opción 4**: habilitar Multi-Factor Authentication con directivas de acceso condicional mediante la evaluación de [directivas de acceso condicional basadas en riesgos](../../active-directory/conditional-access/howto-conditional-access-policy-risk.md).   
**Ventaja**: esta opción le permite:

* Detectar posibles vulnerabilidades que afectan a las identidades de la organización.
* Configurar respuestas automatizadas a acciones sospechosas detectadas que están relacionadas con las identidades de la organización.
* Investigar incidentes sospechosos y tomar las medidas adecuadas para resolverlos.

Este método utiliza la evaluación de riesgos de Azure AD Identity Protection para determinar si se requiere la verificación en dos pasos en función de los riesgos del usuario y el inicio de sesión para todas las aplicaciones en la nube. Este método requiere una licencia de Azure Active Directory P2. Para obtener más información sobre este método, consulte [Azure Active Directory Identity Protection](/azure/active-directory/identity-protection/overview).

> [!Note]
> La opción 2, en la que se habilita Multi-Factor Authentication al cambiar el estado del usuario, invalida las directivas de acceso condicional. Dado que las opciones de 3 y 4 usan directivas de acceso condicional, no puede usar la opción 2 con ellas.

Las organizaciones que no agregan capas de protección de la identidad adicionales, como la verificación en dos pasos, son más susceptibles a ataques de robo de credenciales. Un ataque de robo de credenciales puede poner en peligro la seguridad de los datos.

## <a name="use-role-based-access-control"></a>Uso del control de acceso basado en rol

La administración de acceso de los recursos en la nube es importantísima en cualquier organización que use la nube. El [control de acceso basado en rol (RBAC)](/azure/role-based-access-control/overview) ayuda a administrar quién tiene acceso a los recursos de Azure, qué pueden hacer con esos recursos y a qué áreas puede acceder.

Designar grupos o roles individuales responsables de funciones específicas de Azure ayuda a evitar la confusión que puede desembocar en errores humanos y de automatización que suponen riesgos de seguridad. En organizaciones que quieren aplicar directivas de seguridad para el acceso a los datos, es imperativo restringir el acceso en función de los principios de seguridad [necesidad de saber](https://en.wikipedia.org/wiki/Need_to_know) y [mínimo privilegio](https://en.wikipedia.org/wiki/Principle_of_least_privilege).

El equipo de seguridad necesita visibilidad en los recursos de Azure para evaluar y corregir el riesgo. Si el equipo de seguridad tiene responsabilidades operativas, necesita más permisos para realizar su labor.

Puede usar [RBAC](/azure/role-based-access-control/overview) para asignar permisos a los usuarios, los grupos y las aplicaciones en un ámbito determinado. El ámbito de una asignación de roles puede ser una suscripción, un grupo de recursos o un único recurso.

**Procedimiento recomendado**: repartir las tareas entre el equipo y conceder a los usuarios únicamente el nivel de acceso que necesitan para realizar su trabajo. En lugar de proporcionar a todos los empleados permisos no restringidos en los recursos o la suscripción de Azure, permita solo determinadas acciones en un ámbito concreto.
**Detalles**: use [roles integrados de Azure](/azure/role-based-access-control/built-in-roles) en Azure para asignar privilegios a los usuarios.

> [!Note]
> Los permisos específicos conllevan una complejidad y una confusión innecesarias, y no hace más que alimentar una configuración "heredada" que es difícil de corregir sin riesgo repercutir negativamente en algo. Evite asignar permisos específicos de recursos. En su lugar, utilice grupos de administración para asignar permisos en toda la compañía y grupos de recursos para asignar permisos en las suscripciones. Evite asignar permisos específicos del usuario. En su lugar, asigne acceso a grupos en Azure AD.

**Procedimiento recomendado**: conceder acceso a los equipos de seguridad a responsabilidades de Azure para ver recursos de Azure y, así, poder evaluar y corregir riesgos.
**Detalles**: conceda a los equipos de seguridad el rol RBAC [Lector de seguridad](/azure/role-based-access-control/built-in-roles#security-reader). Puede usar el grupo de administración raíz o el grupo de administración de segmento, según el ámbito de responsabilidades:

* **Grupo de administración raíz** para equipos responsables de todos los recursos de la empresa.
* **Grupo de administración de segmento** para equipos con un ámbito limitado (normalmente, debido a límites organizativos legales o de otra índole).

**Procedimiento recomendado**: conceder los permisos adecuados a los equipos de seguridad que tienen responsabilidades operativas directas.
**Detalles**: revise los roles RBAC integrados para la asignación de rol adecuado. Si los roles integrados no satisfacen las necesidades específicas de la organización, puede crear [roles personalizados de Azure](/azure/role-based-access-control/custom-roles). Igual que los roles integrados, puede asignar roles personalizados a usuarios, grupos y entidades de servicio en los ámbitos de suscripción, grupo de recursos y recurso.

**Procedimientos recomendados**: conceder acceso de Azure Security Center a los roles de seguridad que lo necesiten. Security Center permite a los equipos de seguridad identificar y corregir riesgos con total rapidez.
**Detalles**: asigne a los equipos de seguridad con estas necesidades de seguridad el rol RBAC [Administrador de seguridad](/azure/role-based-access-control/built-in-roles#security-admin) para ver directivas de seguridad, ver estados de seguridad, editar directivas de seguridad, ver alertas y recomendaciones y descartar alertas y recomendaciones. Para ello, puede usar el grupo de administración raíz o el grupo de administración de segmento, según el ámbito de responsabilidades.

Es posible que las organizaciones que no apliquen el control de acceso a los datos mediante el uso de funcionalidades como RBAC estén concediendo más privilegios de los necesarios a sus usuarios. Esto puede poner en peligro los datos al permitir que los usuarios accedan a tipos de datos (por ejemplo, aquellos que son críticos para la empresa) a los que no deberían tener acceso.

## <a name="lower-exposure-of-privileged-accounts"></a>Menor exposición de las cuentas con privilegios

La protección del acceso con privilegios es un primer paso esencial para proteger los recursos empresariales. Minimizar el número de personas que tienen acceso a información segura o a recursos reduce la posibilidad de que los usuarios malintencionados obtengan acceso o de que un usuario autorizado, sin darse cuenta, afecte a un recurso confidencial.

Las cuentas con privilegios son cuentas que administran los sistemas de TI. Estas cuentas son el objetivo de los ciberatacantes, ya que les proporcionan acceso a los datos y los sistemas de una organización. Para proteger el acceso con privilegios, debe aislar las cuentas y los sistemas del riesgo de exposición a usuarios malintencionados.

Recomendamos el desarrollo y seguimiento de una hoja de ruta a fin de proteger el acceso con privilegios de los ciberatacantes. Para obtener información acerca de cómo crear una hoja de ruta detallada para proteger las identidades y el acceso que se administran o notifican en Azure AD, Microsoft Azure, Office 365 y otros servicios en la nube, revise el artículo [Protección del acceso con privilegios para las implementaciones híbridas y en la nube en Azure AD](/azure/active-directory/users-groups-roles/directory-admin-roles-secure).

A continuación, se resumen los procedimientos recomendados que se encuentran en el artículo [Protección del acceso con privilegios para las implementaciones híbridas y en la nube en Azure AD](/azure/active-directory/users-groups-roles/directory-admin-roles-secure):

**Procedimiento recomendado**: Administración, control y supervisión del acceso a las cuentas con privilegios.   
**Detalles**: active [Azure AD Privileged Identity Management](/azure/active-directory/privileged-identity-management/active-directory-securing-privileged-access). Tras activar Privileged Identity Management, recibirá mensajes de correo electrónico de notificación si se producen cambios en el rol de acceso con privilegios. Estas notificaciones muestran una advertencia anticipada cuando se agregan más usuarios a roles con privilegios elevados en el directorio.

**Procedimiento recomendado**: garantizar que todas las cuentas de administrador críticas son cuentas de Azure AD administradas.
**Detalles**: quite las cuentas de consumidor de los roles de administrador críticos (por ejemplo, cuentas Microsoft como hotmail.com, live.com y outlook.com).

**Procedimiento recomendado**: Asegúrese de que todos los roles de administrador críticos tienen una cuenta aparte para las tareas administrativas, a fin de evitar la suplantación de identidad y otros ataques que pueden poner en peligro los privilegios administrativos.
**Detalles**: cree una cuenta de administración aparte que tenga asignados los privilegios necesarios para realizar las tareas administrativas. Bloquee el uso de estas cuentas administrativas con herramientas de productividad diarias como el correo electrónico de Microsoft Office 365 o la exploración web arbitraria.

**Procedimiento recomendado**: Identificación y clasificación de las cuentas que están en roles con privilegios elevados.   
**Detalles**: después de activar Azure AD Privileged Identity Management, vea los usuarios que están en los roles de administrador global, con privilegios y otros con privilegios elevados. Quite todas las cuentas que no sean necesarias en los roles y clasifique las restantes que estén asignadas a roles de administrador:

* Se asigna de forma individual a los usuarios administrativos y se puede usar para fines no administrativos (por ejemplo, correo electrónico personal)
* Asignadas individualmente a usuarios administrativos y designadas solo para fines administrativos
* Compartidas entre varios usuarios
* Para los escenarios de acceso de emergencia
* Para scripts automatizados
* Para usuarios externos

**Procedimiento recomendado**: Implementar el acceso "Just-In-Time (JIT)" para reducir el tiempo de exposición de privilegios aún más y aumentar la visibilidad sobre el uso de cuentas con privilegios.   
**Detalles**: Azure AD Privileged Identity Management le permite:

* Limitar a los usuarios a aceptar solo sus privilegios JIT.
* Asignar roles para una duración reducida con confianza de que los privilegios se revocan automáticamente.

**Procedimiento recomendado**: Definición de un mínimo de dos de cuentas de acceso de emergencia.   
**Detalles**: las cuentas de acceso de emergencia ayudan a las organizaciones a restringir el acceso con privilegios en un entorno de Azure Active Directory existente. Estas cuentas tienen privilegios elevados y no se asignan a usuarios específicos. Las cuentas de acceso de emergencia se limitan a situaciones en las que no se pueden usar las cuentas administrativas normales. Las organizaciones deben limitar el uso de la cuenta de emergencia a la cantidad de tiempo necesaria únicamente.

Evalúe las cuentas que están asignadas al rol de administrador global o que son aptas para él. Si no ve que ninguna cuenta que se use solo en la nube mediante el dominio `*.onmicrosoft.com` (destinado al acceso de emergencia), créelas. Para más información, consulte [Administración de cuentas administrativas de acceso de emergencia en Azure AD](/azure/active-directory/users-groups-roles/directory-emergency-access).

**Procedimiento recomendado**: disponer de un proceso de "emergencia" por si surge una.
**Detalles**: siga los pasos descritos en [Protección del acceso con privilegios para las implementaciones híbridas y en la nube en Azure AD](/azure/active-directory/users-groups-roles/directory-admin-roles-secure).

**Procedimiento recomendado**: requerir que todas las cuentas de administrador críticas no tengan contraseña (opción preferida) o requerir el uso de Multi-Factor Authentication.
**Detalles**: use la [aplicación Microsoft Authenticator](/azure/active-directory/authentication/howto-authentication-phone-sign-in) para iniciar sesión en cualquier cuenta de Azure AD sin utilizar una contraseña. Al igual que [Windows Hello para empresas](/windows/security/identity-protection/hello-for-business/hello-identity-verification), Microsoft Authenticator usa la autenticación basada en claves para habilitar una credencial de usuario que está asociada a un dispositivo y usa una autenticación biométrica o un PIN.

Requiera Azure Multi-Factor Authentication en el inicio de sesión para todos los usuarios asignados de forma permanente a uno o varios de los roles de administrador de Azure AD: administrador global, administrador de roles con privilegios, administrador de Exchange Online y administrador de SharePoint Online. Habilite [Multi-Factor Authentication en sus cuentas de administrador](/azure/active-directory/authentication/howto-mfa-userstates) y asegúrese de que todos los usuarios con cuentas de administrador están registrados.

**Procedimiento recomendado**: en las cuentas de administrador críticas, tener una estación de trabajo de administración donde no se permitan tareas de producción (por ejemplo, las exploraciones o el correo electrónico). De este modo, protegerá las cuentas de administrador de vectores de ataque que usan la exploración y el correo electrónico, al tiempo que reducirá considerablemente el riesgo de que se produzca incidentes importantes.
**Detalles**: use una estación de trabajo de administración. Elija el nivel de seguridad de la estación de trabajo:

- Los dispositivos de productividad con una protección elevada proporcionan seguridad avanzada en operaciones de exploración y otras tareas de productividad.
- Las [estaciones de trabajo con privilegios de acceso (PAW)](/windows-server/identity/securing-privileged-access/privileged-access-workstations) proporcionan un sistema operativo dedicado que está protegido de ataques de Internet y vectores de amenazas al realizar tareas delicadas.

**Procedimiento recomendado**: desaprovisionar las cuentas de administrador de empleados que dejen la organización.
**Detalles**: tenga implantado un proceso que deshabilite o elimine las cuentas de administrador de los empleados que dejen la organización.

**Procedimiento recomendado**: probar con regularidad las cuentas de administrador usando técnicas de ataque actuales.
**Detalles**: use el Simulador de ataques de Office 365 o una oferta de terceros para ejecutar escenarios de ataque realistas dentro de la organización. Esto puede ayudar a detectar a los usuarios vulnerables antes de que se produzca un ataque real.

**Procedimiento recomendado**: Pasos necesarios para mitigar las técnicas de ataque usadas más frecuentemente.  
**Detalles**: [identifique las cuentas de Microsoft que tengan roles administrativos y que deban cambiarse a cuentas profesionales o educativas](/azure/active-directory/users-groups-roles/directory-admin-roles-secure#identify-microsoft-accounts-in-administrative-roles-that-need-to-be-switched-to-work-or-school-accounts)  

[Asegúrese de que haya cuentas de usuario independientes y de que se lleve a cabo el reenvío de correos electrónicos para las cuentas de administrador globales](/azure/active-directory/users-groups-roles/directory-admin-roles-secure)  

[Asegúrese de que las contraseñas de las cuentas administrativas hayan cambiado recientemente.](/azure/active-directory/users-groups-roles/directory-admin-roles-secure#ensure-the-passwords-of-administrative-accounts-have-recently-changed)  

[Active la sincronización de hashes de contraseñas](/azure/active-directory/users-groups-roles/directory-admin-roles-secure#turn-on-password-hash-synchronization)  

[Requiera Multi-Factor Authentication a los usuarios de todos los roles con privilegios, así como a los usuarios expuestos](/azure/active-directory/users-groups-roles/directory-admin-roles-secure#require-multi-factor-authentication-mfa-for-users-in-all-privileged-roles-as-well-as-exposed-users)  

[Obtenga su Office 365 Secure Score (si usa Office 365)](/azure/active-directory/users-groups-roles/directory-admin-roles-secure#obtain-your-office-365-secure-score-if-using-office-365)  

[Consulte la guía de seguridad y cumplimiento de Office 365 (si usa Office 365)](/azure/active-directory/users-groups-roles/directory-admin-roles-secure#review-the-office-365-security-and-compliance-guidance-if-using-office-365)  

[Configure la supervisión de la actividad de Office 365 (si usa Office 365)](/azure/active-directory/users-groups-roles/directory-admin-roles-secure#configure-office-365-activity-monitoring-if-using-office-365)  

[Establezca los propietarios del plan de respuesta ante incidentes o emergencias](/azure/active-directory/users-groups-roles/directory-admin-roles-secure#establish-incidentemergency-response-plan-owners)  

[Proteja las cuentas administrativas con privilegios locales](/azure/active-directory/users-groups-roles/directory-admin-roles-secure#turn-on-password-hash-synchronization)

Si no protege el acceso con privilegios, es posible que tenga demasiados usuarios en roles con privilegios elevados y sea más vulnerables a ataques. Los actores malintencionados, entre los que se incluyen ciberatacantes, a menudo tienen como objetivo las cuentas de administrador y otros elementos con acceso privilegiado para obtener acceso a datos confidenciales y a sistemas mediante el robo de credenciales.

## <a name="control-locations-where-resources-are-created"></a>Controlar las ubicaciones donde se crean los recursos

Es muy importante permitir que los operadores de nube realicen tareas pero al mismo tiempo impedir que transgredan las convenciones que son necesarias para administrar los recursos de la organización. Las organizaciones que quieran controlar las ubicaciones donde se crean los recursos deben codificarlas de forma segura.

Puede usar [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview) para crear directivas de seguridad cuyas definiciones describan las acciones o los recursos que se deniegan específicamente. Esas definiciones de directivas se asignan en el ámbito deseado, como la suscripción, el grupo de recursos o un recurso individual.

> [!NOTE]
> Las directivas de seguridad no son las mismas que RBAC. En realidad, usan RBAC para autorizar a los usuarios la creación de estos recursos.
>
>

Las organizaciones que no controlan cómo se crean los recursos son más susceptibles a que los usuarios puedan hacer un mal uso del servicio y crear más recursos de los necesarios. Dificultar el proceso de creación de recursos es un paso importante para proteger escenarios en los que intervienen varios inquilinos.

## <a name="actively-monitor-for-suspicious-activities"></a>Supervisión activa de actividades sospechosas

Un sistema de supervisión de identidades activo puede detectar un comportamiento sospechoso y desencadenar una alerta para que se investigue. En la tabla siguiente se muestran dos de las funcionalidades de Azure AD que pueden ayudar a las organizaciones a supervisar sus identidades:

**Procedimiento recomendado**: Tener un método de identificación:

- Intentos para iniciar sesión [sin realizar ningún seguimiento](/azure/active-directory/active-directory-reporting-sign-ins-from-unknown-sources).
- Ataques por [fuerza bruta](/azure/active-directory/active-directory-reporting-sign-ins-after-multiple-failures) contra una cuenta determinada.
- Intentos para iniciar sesión desde varias ubicaciones.
- Inicios de sesión desde [dispositivos infectados](/azure/active-directory/active-directory-reporting-sign-ins-from-possibly-infected-devices).
- Direcciones IP sospechosas.

**Detalles**: use los [informes de anomalías](/azure/active-directory/active-directory-view-access-usage-reports) de Azure AD Premium. Contar con procesos y procedimientos implementados para que los administradores de TI ejecuten dichos informes diariamente o a petición (normalmente en un escenario de respuesta a incidentes).

**Procedimiento recomendado**: Contar con un sistema de supervisión activo que le informa de los riesgos y que puede ajustar el nivel de riesgo (alto, medio o bajo) a sus requisitos empresariales.   
**Detalles**: use [Azure AD Identity Protection](/azure/active-directory/active-directory-identityprotection), que marca los riesgos actuales en su propio panel y envía notificaciones de resumen diarias por correo electrónico. Para ayudar a proteger las identidades de la organización, puede configurar directivas basadas en riesgos que respondan automáticamente a problemas detectados si se alcanza un nivel de riesgo específico.

Las organizaciones que no supervisen activamente sus sistemas de identidad corren el riesgo de comprometer las credenciales de los usuarios. Si las organizaciones no saben que están teniendo lugar actividades sospechosas a través de estas credenciales, no podrán mitigar este tipo de amenaza.

## <a name="use-azure-ad-for-storage-authentication"></a>Uso de Azure AD para la autenticación de almacenamiento
[Azure Storage](/azure/storage/common/storage-auth-aad) admite la autenticación y autorización con Azure AD para Blob Storage y Queue Storage. Con la autenticación de Azure AD, puede usar el control de acceso basado en roles de Azure para conceder permisos específicos a usuarios, grupos y aplicaciones hasta el ámbito de una cola o un contenedor de blobs individual.

Se recomienda usar [Azure AD para autenticar el acceso al almacenamiento](https://azure.microsoft.com/blog/azure-storage-support-for-azure-ad-based-access-control-now-generally-available/).

## <a name="next-step"></a>Paso siguiente

Vea [Patrones y procedimientos recomendados de seguridad en Azure](best-practices-and-patterns.md) para obtener más procedimientos recomendados de seguridad que pueda aplicar cuando diseñe, implemente y administre las soluciones en la nube mediante Azure.
