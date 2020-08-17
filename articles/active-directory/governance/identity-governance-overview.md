---
title: Identity Governance (Azure Active Directory) | Microsoft Docs
description: Azure Active Directory Identity Governance le permite equilibrar la productividad de los empleados y la seguridad que necesita su organización con la visibilidad y los procesos adecuados.
services: active-directory
documentationcenter: ''
author: ajburnle
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
ms.openlocfilehash: dcb7ab566d861e0c4256acda8f8facd1e94d4ad6
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87796784"
---
# <a name="what-is-azure-ad-identity-governance"></a>¿Qué es Azure AD Identity Governance?

Azure Active Directory (Azure AD) Identity Governance le permite equilibrar la productividad de los empleados y la seguridad que necesita su organización con la visibilidad y los procesos adecuados. Proporciona funcionalidades que garantizan que las personas adecuadas tienen el acceso adecuado a los recursos adecuados. Estas características de Azure AD y Enterprise Mobility + Security y relacionadas permiten reducir el riesgo del acceso al proteger, supervisar y auditar el acceso a los recursos críticos, al mismo tiempo que garantizan la productividad de empleados y asociados empresariales.  

Identity Governance ofrece a las organizaciones la posibilidad de realizar las siguientes tareas con empleados, asociados empresariales, proveedores, servicios y aplicaciones tanto en el entorno local como en la nube:

- Controlar el ciclo de vida de las identidades.
- Controlar el ciclo de vida de los accesos.
- Proteger el acceso con privilegios para la administración

Está especialmente diseñado para ayudar a las organizaciones a abordar estas cuatro preguntas claves:

- ¿Qué usuarios deben tener acceso a qué recursos?
- ¿Qué hacen esos usuarios con el acceso concedido?
- ¿La organización cuenta con controles eficaces para administrar el acceso?
- ¿Los auditores pueden comprobar qué controles funcionan?

## <a name="identity-lifecycle"></a>Ciclo de vida de las identidades

La gobernanza de identidades ayuda a las organizaciones a alcanzar un equilibrio entre *productividad* (con qué rapidez puede obtener acceso una persona a los recursos que necesita; por ejemplo, cuando se une a una organización) y *seguridad* (cómo debe cambiar el acceso de esa persona a lo largo del tiempo; por ejemplo, cuando varía su estado laboral).  La administración del ciclo de vida de las identidades es la piedra angular de la gobernanza de identidades y, para que esa gobernanza resulte eficaz a diferentes escalas, es preciso modernizar la infraestructura de administración del ciclo de vida de las identidades en las aplicaciones.

![Ciclo de vida de las identidades](./media/identity-governance-overview/identity-lifecycle.png)

En muchas organizaciones, el ciclo de vida de las identidades de los empleados está relacionado con la representación del usuario en un sistema HCM (administración del capital humano).  Azure AD Premium mantiene automáticamente las identidades de usuario de las personas representadas en Workday tanto en Active Directory como en Azure Active Directory, como se describe en el [tutorial sobre el aprovisionamiento de entrada en Workday](../saas-apps/workday-inbound-tutorial.md).  Azure AD Premium incluye también [Microsoft Identity Manager](/microsoft-identity-manager/), que permite importar registros desde los sistemas HCM locales, como SAP, Oracle eBusiness y Oracle PeopleSoft.

Cada vez son más los escenarios en los que es preciso colaborar con personas que están fuera de la organización. La [colaboración B2B de Azure AD](/azure/active-directory/b2b/) permite compartir de forma segura las aplicaciones y servicios corporativos con usuarios invitados y asociados externos de cualquier organización, a la vez que mantiene el control sobre sus propios datos corporativos.  La [administración de derechos de Azure AD](entitlement-management-overview.md) le permite seleccionar los usuarios de la organización que tienen permiso para solicitar acceso y que se pueden agregar como invitados B2B al directorio de la organización; también garantiza que estos invitados se eliminen cuando ya no necesiten acceso.

## <a name="access-lifecycle"></a>Ciclo de vida de los accesos

Las organizaciones necesitan un proceso que administre el acceso sin limitarse a lo que se aprovisionó inicialmente, cuando se creó la identidad del usuario.  Además, las organizaciones empresariales deben ser capaces de escalar sus recursos con eficacia para poder desarrollar y aplicar de forma continuada directivas y controles de acceso.

![Ciclo de vida de los accesos](./media/identity-governance-overview/access-lifecycle.png)

Normalmente, el departamento de TI delega las decisiones sobre la aprobación de los accesos en los responsables de la toma de decisiones de la empresa.  Por otro lado, el departamento de TI puede involucrar él mismo a los usuarios.  Por ejemplo, los usuarios que tienen acceso a los datos confidenciales de los clientes en una aplicación de marketing de una compañía en Europa tienen que conocer las directivas de la compañía. Es posible que los usuarios invitados no conozcan los requisitos de administración de los datos de una organización a la que han sido invitados.

Las organizaciones pueden automatizar el proceso del ciclo de vida de los accesos utilizando determinadas tecnologías, como los [grupos dinámicos](../users-groups-roles/groups-dynamic-membership.md), junto con el aprovisionamiento de usuarios en [aplicaciones SaaS](../saas-apps/tutorial-list.md) o [aplicaciones integradas con SCIM](../app-provisioning/use-scim-to-provision-users-and-groups.md).  Las organizaciones también pueden controlar qué [usuarios invitados tienen acceso a las aplicaciones locales](../b2b/hybrid-cloud-to-on-premises.md).  Estos derechos de acceso se pueden revisar periódicamente utilizando [revisiones de acceso de Azure AD](access-reviews-overview.md) recurrentes.   La [administración de derechos de Azure AD](entitlement-management-overview.md) también le permite definir cómo los usuarios solicitan acceso en los paquetes de pertenencias a grupos y equipos, roles de aplicación y roles de SharePoint Online.

Cuando un usuario intenta acceder a las aplicaciones, Azure AD impone directivas de [acceso condicional](/azure/active-directory/conditional-access/). Por ejemplo, las directivas de acceso condicional pueden especificar que se muestren los [términos de uso](../conditional-access/terms-of-use.md) para [garantizar que el usuario acepta los términos](../conditional-access/require-tou.md) antes de acceder a una aplicación.

## <a name="privileged-access-lifecycle"></a>Ciclo de vida de los accesos con privilegios

Históricamente, otros proveedores concebían el acceso con privilegios como una funcionalidad independiente de la gobernanza de identidades. Sin embargo, en Microsoft, creemos que el control del acceso con privilegios constituye una parte fundamental de la gobernanza de identidades, especialmente si tenemos en cuenta las consecuencias que podría tener para la organización un uso indebido con los derechos de administrador. Es preciso controlar a los empleados, los proveedores y los contratistas que tienen derechos administrativos.

![Ciclo de vida de los accesos con privilegios](./media/identity-governance-overview/privileged-access-lifecycle.png)

[Azure AD Privileged Identity Management (PIM)](../privileged-identity-management/pim-configure.md) dispone de controles adicionales que están adaptados para proteger los derechos de acceso de los recursos en Azure AD, Azure y otros servicios en línea de Microsoft.  El acceso Just-In-Time y las funcionalidades para enviar alertas cuando cambia un rol que están disponibles en Azure AD PIM, junto con la autenticación multifactor y el acceso condicional, ofrecen un completo conjunto de controles de gobernanza que le ayudan a proteger los recursos de la compañía (roles de recursos de directorio, de Azure y de Office 365). Al igual que con otras formas de acceso, las organizaciones pueden usar las revisiones de acceso para configurar nuevas certificaciones de acceso periódicas para todos los usuarios con roles de administrador.

## <a name="getting-started"></a>Introducción

Consulte la pestaña Introducción de **Identity Governance** de Azure Portal para comenzar a usar la administración de derechos, las revisiones de acceso, Privileged Identity Management y los términos de uso.

![Introducción al gobierno de identidades](./media/identity-governance-overview/getting-started.png)


Si tiene algún comentario sobre las características de Gobierno de identidades, haga clic en **¿Tiene algún comentario?** en Azure Portal para enviar sus comentarios. El equipo revisa periódicamente los comentarios.

Aunque no existe ninguna solución o recomendación perfecta para cada cliente, las siguientes guías de configuración también proporcionan las directivas de referencia que Microsoft recomienda seguir para garantizar unos recursos más seguros y productivos.

- [Configuraciones de acceso de dispositivos e identidades](/microsoft-365/enterprise/microsoft-365-policies-configurations)
- [Protección del acceso con privilegios](../users-groups-roles/directory-admin-roles-secure.md)

## <a name="appendix---least-privileged-roles-for-managing-in-identity-governance-features"></a>Apéndice: Roles con menos privilegios para administrar en características de Identity Governance

Se recomienda usar el rol con menos privilegios para realizar tareas administrativas en la gobernanza de identidades. Igualmente, para realizar estas tareas, se recomienda usar Azure AD PIM para activar un rol según sea necesario. A continuación se muestran los roles de directorio con menos privilegios para configurar las características de gobernanza de identidades:

| Característica | Rol con privilegios mínimos |
| ------- | --------------------- |
| Administración de derechos | Administrador de usuarios (a excepción de la adición de sitios de SharePoint Online a catálogos, que requiere un administrador global) |
| Revisiones de acceso | Administrador de usuarios (con la excepción de las revisiones de acceso de Azure o los roles de Azure AD, que requiere el administrador de roles con privilegios) |
|Privileged Identity Management | Administrador de roles con privilegios |
| Términos de uso | Administrador de seguridad o Administrador de acceso condicional |

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la administración de derechos de Azure AD?](entitlement-management-overview.md)
- [¿Qué son las revisiones de acceso de Azure AD?](access-reviews-overview.md)
- [¿Qué es Azure AD Privileged Identity Management?](../privileged-identity-management/pim-configure.md)
- [¿Qué puedo hacer con los Términos de uso?](active-directory-tou.md)


