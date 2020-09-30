---
title: 'Administración regulada por la nube de Azure AD para cargas de trabajo locales: Azure'
description: En este tema se describe la administración regulada por la nube para cargas de trabajo locales.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9cb101e415499150cd3d825fe5f42ce0dbc766fb
ms.sourcegitcommit: f8d2ae6f91be1ab0bc91ee45c379811905185d07
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89662523"
---
# <a name="how-azure-ad-delivers-cloud-governed-management-for-on-premises-workloads"></a>Cómo proporciona Azure AD administración regulada por la nube para cargas de trabajo locales

Azure Active Directory (Azure AD) es una solución de identidad como servicio (IDaaS) completa que usan millones de organizaciones y que abarca todos los aspectos de la identidad, la administración de acceso y la seguridad. Azure AD contiene más de mil millones de identidades de usuario y ayuda a los usuarios a iniciar sesión y acceder de forma segura a:

* Recursos externos, como Microsoft 365, Azure Portal y miles de otras aplicaciones de software como servicio (SaaS).
* recursos internos, como las aplicaciones de la red corporativa y la intranet de una organización, junto con todas las aplicaciones en la nube desarrolladas por esa organización.

Las organizaciones pueden usar Azure AD si son de tipo "nube pura", o bien como una implementación "híbrida" si tienen cargas de trabajo locales. Una implementación híbrida de Azure AD puede formar parte de la estrategia de una organización para migrar sus recursos de TI a la nube, o bien para continuar la integración de la infraestructura local existente junto con los nuevos servicios en la nube.

Históricamente, las organizaciones "híbridas" han considerado a Azure AD una extensión de su infraestructura local existente. En estas implementaciones, la administración de la gobernanza de identidades locales, Windows Server Active Directory u otros sistemas de directorio internos, son los puntos de control, y los usuarios y grupos se sincronizan desde esos sistemas en un directorio en la nube como Azure AD. Una vez que esas identidades se encuentran en la nube, se pueden poner a disposición de Microsoft 365, Azure y otras aplicaciones.

![Ciclo de vida de las identidades](media/cloud-governed-management-for-on-premises//image1.png)

A medida que las organizaciones mueven una mayor parte de su infraestructura de TI junto con sus aplicaciones a la nube, muchas buscan las funciones de seguridad mejorada y administración simplificada de la administración de identidades como un servicio. Las características de IDaaS proporcionadas por la nube en Azure AD aceleran la transición a la administración regulada por la nube mediante la oferta de soluciones y funciones que permiten a las organizaciones adoptar rápidamente y mover una mayor parte de su administración de identidades desde sistemas tradicionales locales a Azure AD, mientras siguen admitiendo aplicaciones existentes y nuevas.

En este documento se describe la estrategia de Microsoft para IDaaS híbridas y cómo las organizaciones pueden usar Azure AD para sus aplicaciones existentes.

## <a name="the-azure-ad-approach-to-cloud-governed-identity-management"></a>El enfoque de Azure AD a la administración de identidades regulada por la nube

A medida que las organizaciones realizan la transición a la nube, necesitan garantías de que disponen de control de todo el entorno: más seguridad y visibilidad de las actividades, respaldada por la automatización, e información proactiva. En "**Administración regulada por la nube**" se describe cómo las organizaciones administran y controlan sus usuarios, aplicaciones, grupos y dispositivos desde la nube.

En este mundo moderno, las organizaciones necesitan poder administrar de forma eficaz a escala, debido a la proliferación de las aplicaciones SaaS y el papel creciente de las identidades de colaboración y externas. El nuevo panorama de riesgo de la nube significa que una organización debe tener más capacidad de respuesta: un actor malintencionado que ponga en peligro a un usuario en la nube podría afectar a las aplicaciones en la nube y locales.

En concreto, las organizaciones híbridas necesitan poder delegar y automatizar las tareas, que históricamente el departamento de TI ha realizado de forma manual. Para automatizar las tareas, necesitan API y procesos que organicen el ciclo de vida de los distintos recursos relacionados con la identidad (usuarios, grupos, aplicaciones, dispositivos), para poder delegar la administración diaria de esos recursos a más terceras personas externas al personal de TI principal. Azure AD aborda estos requisitos a través de la administración de cuentas de usuario y la autenticación nativa para los usuarios sin necesidad de una infraestructura de identidad local. La no creación de una infraestructura local puede beneficiar a las organizaciones que tienen nuevas comunidades de usuarios, como socios comerciales, que no han surgido en su directorio local, pero cuya administración de acceso es fundamental para conseguir resultados empresariales.

Además, la administración no está completa sin la gobernanza, y en este nuevo panorama la gobernanza es un elemento integrado del sistema de identidades en lugar de un complemento. La gobernanza de identidades permite a las organizaciones administrar el ciclo de vida de las identidades y el acceso entre empleados, asociados, proveedores, servicios y aplicaciones.

La incorporación de la gobernanza de identidades facilita la transición de la organización a la administración regulada por la nube, permite la escala de TI, aborda nuevos desafíos con los invitados y proporciona información y automatización más profundas de las que los clientes tenían con la infraestructura local. En este nuevo mundo, la gobernanza significa la capacidad de una organización para disponer de transparencia, visibilidad y los controles adecuados en el acceso a los recursos dentro de ella. Con Azure AD, las operaciones de seguridad y los equipos de auditoría tienen visibilidad sobre quién tiene acceso (y quién debería tenerlo) a qué recursos de la organización (en qué dispositivos), qué hacen los usuarios con ese acceso y si la organización tiene y usa los controles adecuados para quitar o restringir el acceso de acuerdo a las directivas de la empresa o legales.

El nuevo modelo de administración beneficia a las organizaciones con aplicaciones SaaS y de línea de negocio (LOB), ya que pueden administrar y proteger el acceso a esas aplicaciones más fácilmente. Mediante la integración de las aplicaciones con Azure AD, las organizaciones podrán usar y administrar el acceso entre las identidades originadas en el entorno local y en la nube de forma coherente. La administración del ciclo de vida de las aplicaciones se automatiza más y Azure AD proporciona información detallada sobre el uso de las aplicaciones que no se podía obtener fácilmente en la administración de identidades local. Mediante Azure AD, los grupos de Microsoft 365 y las características de autoservicio de Teams, las organizaciones pueden crear grupos con facilidad para la colaboración y la administración del acceso, y agregar o quitar usuarios en la nube para permitir la colaboración y acceder a los requisitos de administración.

La selección de las funciones correctas de Azure AD para la administración regulada por la nube depende de las aplicaciones que se van a usar, y cómo se van a integrar con Azure AD. En las secciones siguientes se describen los enfoques que adoptar para aplicaciones integradas en AD y aplicaciones en las que se usan protocolos de federación (por ejemplo, SAML, OAuth u OpenID Connect).

## <a name="cloud-governed-management-for-ad-integrated-applications"></a>Administración regulada por la nube para aplicaciones integradas en AD

Azure AD mejora la administración de las aplicaciones integradas en Active Directory locales de una organización mediante el acceso remoto seguro y el acceso condicional a esas aplicaciones. Además, Azure AD proporciona administración del ciclo de vida de las cuentas y administración de credenciales para las cuentas de AD existentes del usuario, incluido lo siguiente:

* **Acceso remoto seguro y acceso condicional para aplicaciones locales**

Para muchas organizaciones, el primer paso para administrar el acceso desde la nube para las aplicaciones web integradas en AD locales y basadas en Escritorio remoto consiste en implementar el [proxy de aplicación](../manage-apps/application-proxy.md) por delante de esas aplicaciones para proporcionar acceso remoto seguro.

Después de un inicio de sesión único en Azure AD, los usuarios pueden acceder a las aplicaciones locales y en la nube mediante una dirección URL externa o un portal de aplicaciones interno. Por ejemplo, Application Proxy proporciona acceso remoto e inicio de sesión único para Escritorio remoto, SharePoint, y también aplicaciones como Tableau y Qlik, y de línea de negocio (LOB). Además, las directivas de acceso condicional pueden incluir que se muestren los [términos de uso](../conditional-access/terms-of-use.md) y [garantizar que el usuario los acepta](../conditional-access/require-tou.md) antes de poder acceder a una aplicación.

![Arquitectura del Proxy de aplicación](media/cloud-governed-management-for-on-premises/image2.png)

* **Administración automática del ciclo de vida para las cuentas de Active Directory**

La gobernanza de identidades ayuda a las organizaciones a alcanzar un equilibrio entre *productividad* (con qué rapidez puede obtener acceso un usuario a los recursos que necesita, por ejemplo, cuando se une a la organización) y *seguridad* (cómo debe cambiar el acceso de ese usuario en el tiempo, por ejemplo, cuando varía su estado laboral). La administración del ciclo de vida de las identidades es la piedra angular de la gobernanza de identidades y, para que esa gobernanza resulte eficaz a diferentes escalas, es preciso modernizar la infraestructura de administración del ciclo de vida de las identidades en las aplicaciones.

En muchas organizaciones, el ciclo de vida de las identidades de los empleados está relacionado con la representación del usuario en un sistema de administración del capital humano (HCM). Para las organizaciones que usan Workday como su sistema HCM, Azure AD puede asegurar que las cuentas de usuario en AD [se aprovisionan y desaprovisionan de forma automática para los trabajadores en Workday](../saas-apps/workday-inbound-tutorial.md). Esto mejora la productividad del usuario mediante la automatización de cuentas por derecho y administra los riesgos garantizando que el acceso a la aplicación se actualiza de forma automática cuando un usuario cambia de roles o abandona la organización. El [plan de implementación](https://aka.ms/WorkdayDeploymentPlan) de aprovisionamiento de usuarios controlado por Workday es una guía paso a paso en la que se describe a las organizaciones la implementación de procedimientos recomendados de Workday en la solución de aprovisionamiento de usuarios de Active Directory a través de un proceso de cinco pasos.

Azure AD Premium incluye también Microsoft Identity Manager, que permite importar registros desde otros sistemas HCM locales, incluidos SAP, Oracle eBusiness y Oracle PeopleSoft.

Cada vez es más habitual que en la colaboración de negocio a negocio sea necesario conceder acceso a usuarios ajenos a la organización. La [colaboración B2B de Azure AD](/azure/active-directory/b2b/) permite que las organizaciones compartan de forma segura sus aplicaciones y servicios con usuarios invitados y asociados externos, a la vez que mantienen el control sobre sus propios datos corporativos.

Azure AD puede [crear de forma automática cuentas de AD para usuarios invitados](../external-identities/hybrid-cloud-to-on-premises.md) según sea necesario, lo que permite a los invitados empresariales acceder a aplicaciones locales integradas en AD sin necesidad de otra contraseña. Las organizaciones pueden configurar directivas de [autenticación multifactor (MFA) para usuarios invitados](../external-identities/conditional-access.md), de forma que las comprobaciones de MFA se realicen durante la autenticación del proxy de aplicación. Además, todas [las revisiones de acceso](../governance/manage-guest-access-with-access-reviews.md) que se realizan en los usuarios de B2B en la nube se aplican a los usuarios locales. Por ejemplo, si el usuario en la nube se elimina mediante directivas de administración del ciclo de vida, también se elimina el usuario local.

**Administración de credenciales para cuentas de Active Directory** El autoservicio de restablecimiento de contraseña de Azure AD permite que los usuarios que han olvidado sus contraseñas se vuelvan a autenticar y restablezcan sus contraseñas, y las contraseñas cambiadas [se escriben en Active Directory local](../authentication/concept-sspr-writeback.md). El proceso de restablecimiento de contraseña también puede usar las directivas de contraseña de Active Directory locales: cuando un usuario restablece su contraseña, se comprueba que cumpla la directiva de Active Directory local antes de confirmarla en ese directorio. En el [plan de implementación](https://aka.ms/deploymentplans/sspr) del autoservicio de restablecimiento de contraseña se describen los procedimientos recomendados para implementarlo a los usuarios mediante experiencias web y de integración en Windows.

![Arquitectura SSPR de Azure AD](media/cloud-governed-management-for-on-premises/image3.png)

Por último, para las organizaciones que permiten que los usuarios cambien sus contraseñas en Active Directory, AD se puede configurar para usar la misma directiva de contraseña que la organización usa en Azure AD a través de la [característica de protección con contraseña de Azure AD](../authentication/concept-password-ban-bad-on-premises.md), actualmente en versión preliminar pública.

Cuando una organización está lista para mover una aplicación integrada en AD a la nube mediante el desplazamiento a Azure del sistema operativo que hospeda la aplicación, [Azure AD Domain Services](../../active-directory-domain-services/overview.md) proporciona servicios de dominio compatibles con AD (por ejemplo, unión a un dominio, directiva de grupo, LDAP y autenticación Kerberos o NTLM). Azure AD Domain Services se integra con el inquilino de Azure AD existente de la organización, para que los usuarios puedan iniciar sesión con sus credenciales corporativas. Además, se pueden usar los grupos y las cuentas de usuario existentes para proteger el acceso a los recursos, lo que garantiza una migración "lift-and-shift" más fluida de los recursos locales a los servicios de infraestructura de Azure.

![Azure AD Domain Services](media/cloud-governed-management-for-on-premises/image4.png)

## <a name="cloud-governed-management-for-on-premises-federation-based-applications"></a>Administración regulada por la nube para aplicaciones locales basadas en federación

Para una organización que ya usa un proveedor de identidades local, el cambio de las aplicaciones a Azure AD permite un acceso más seguro y una experiencia administrativa más sencilla para la administración de la federación. Azure AD permite la configuración pormenorizada de los controles de acceso por aplicación, incluido Azure Multi-Factor Authentication, mediante el acceso condicional de Azure AD. Azure AD admite más funcionalidades, incluidos certificados de firma de token específicos de la aplicación y fechas de caducidad de certificados configurables. Estas funciones, herramientas e instrucciones permiten a las organizaciones retirar sus proveedores de identidades locales. El propio departamento de TI de Microsoft, por ejemplo, ha movido 17.987 aplicaciones desde Servicios de federación de Active Directory (AD FS) de Microsoft a Azure AD.

![Evolución de Azure AD](media/cloud-governed-management-for-on-premises/image5.png)

Para comenzar a migrar aplicaciones federadas a Azure AD como proveedor de identidades, consulte https://aka.ms/migrateapps, que incluye vínculos a:

* El informe [Migración de las aplicaciones a Azure Active Directory](https://aka.ms/migrateapps/whitepaper), en el que se presentan las ventajas de la migración y se describe cómo planearla en cuatro fases bien definidas: detección, clasificación, migración y administración continua. Se le guiará sobre cómo debe pensar en el proceso y desglosar el proyecto en partes fáciles de consumir. Este documento incluye vínculos a recursos importantes que le ayudarán a lo largo de este proceso.

* La guía de soluciones [Migración de la autenticación de aplicaciones desde Servicios de federación de Active Directory a Azure Active Directory](https://aka.ms/migrateapps/adfssolutionguide), en la que se exploran con más detalle las mismas cuatro fases de planeamiento y ejecución de un proyecto de migración de aplicaciones. En esta guía, aprenderá a aplicar estas fases al objetivo concreto de mover una aplicación desde Servicios de federación de Active Directory (AD FS) a Azure AD.

* El [script de preparación para la migración de Servicios de federación de Active Directory](https://aka.ms/migrateapps/adfstools) se puede ejecutar en servidores de Servicios de federación de Active Directory (AD FS) locales existentes para determinar la preparación de las aplicaciones para la migración a Azure AD.

## <a name="ongoing-access-management-across-cloud-and-on-premises-applications"></a>Administración del acceso continuo entre aplicaciones locales y en la nube

Las organizaciones necesitan un proceso de administración del acceso que sea escalable. Los usuarios siguen acumulando derechos de acceso y acaban con más de los que inicialmente se les han aprovisionado. Además, las organizaciones empresariales deben ser capaces de escalar con eficacia para desarrollar y aplicar de forma continuada directivas y controles de acceso.

Normalmente, el departamento de TI delega las decisiones sobre la aprobación de los accesos en los responsables de la toma de decisiones de la empresa. Por otro lado, el departamento de TI puede involucrar él mismo a los usuarios. Por ejemplo, los usuarios que tienen acceso a los datos confidenciales de los clientes en una aplicación de marketing de una compañía en Europa tienen que conocer las directivas de la compañía. También es posible que los usuarios invitados desconozcan los requisitos de control de los datos de una organización a la que han sido invitados.

Las organizaciones pueden automatizar el proceso del ciclo de vida de los accesos a través de tecnologías como los [grupos dinámicos](../users-groups-roles/groups-dynamic-membership.md), junto con el aprovisionamiento de usuarios en [aplicaciones SaaS](../saas-apps/tutorial-list.md) o [aplicaciones integradas con el estándar System for Cross-Domain Identity Management (SCIM](../app-provisioning/use-scim-to-provision-users-and-groups.md)). Las organizaciones también pueden controlar qué [usuarios invitados tienen acceso a las aplicaciones locales](../external-identities/hybrid-cloud-to-on-premises.md). Estos derechos de acceso se pueden revisar periódicamente utilizando [revisiones de acceso de Azure AD](../governance/access-reviews-overview.md) recurrentes.

## <a name="future-directions"></a>Direcciones en el futuro

En los entornos híbridos, la estrategia de Microsoft consiste en habilitar implementaciones donde la **nube es el plano de control de las identidades**, y los directorios locales y otros sistemas de identidad, como Active Directory y otras aplicaciones locales, son el destino para el aprovisionamiento de acceso a los usuarios. Esta estrategia seguirá garantizando los derechos, las identidades y el acceso en esas aplicaciones y en las cargas de trabajo que dependen de ellas. En este estado final, las organizaciones podrán impulsar la productividad del usuario final completamente desde la nube.

![Arquitectura de Azure AD](media/cloud-governed-management-for-on-premises/image6.png)

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo emprender este viaje, vea los planes de implementación de Azure AD, que encontrará en <https://aka.ms/deploymentplans>. Proporcionan instrucciones completas sobre cómo implementar funcionalidades de Azure Active Directory (Azure AD). En cada plan se explica el valor de negocio, las consideraciones de planificación, el diseño y los procedimientos operativos necesarios para implementar correctamente funcionalidades comunes de Azure AD. Microsoft actualiza continuamente los planes de implementación con procedimientos recomendados obtenidos de implementaciones de clientes y otros comentarios cuando se agregan funciones para la administración de la nube con Azure AD.