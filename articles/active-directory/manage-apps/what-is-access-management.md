---
title: Administración del acceso a aplicaciones con Azure AD
description: Describe cómo Azure Active Directory permite a las organizaciones especificar las aplicaciones a las que cada usuario tiene acceso.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 05/16/2017
ms.author: kenwith
ms.openlocfilehash: 1b19f4aae7bf7477dbe5950f2d4df31e2de81372
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87562572"
---
# <a name="managing-access-to-apps"></a>Administración del acceso a las aplicaciones

El acceso continuo a las aplicaciones, la evaluación del uso y la generación de informes siguen siendo un desafío después de que una aplicación se integra en el sistema de identidad de su organización. En muchos casos, el administrador de TI o el departamento de soporte técnico deben asumir un rol activo en la administración del acceso a las aplicaciones. En ocasiones, la asignación la realiza un equipo de TI general o departamental. En ocasiones, se pretende que la decisión de asignación se delegue en el responsable de la toma de decisiones, lo que requiere su aprobación antes de que TI realice la asignación.  Otras organizaciones invierten en integración con un sistema automatizado existente de administración de identidades y acceso, como Control de acceso basado en rol (RBAC) o Control de acceso basado en atributos (ABAC). Tanto la integración como el desarrollo de reglas tienden a ser procesos especializados y caros. La supervisión o la generación de informes en cualquier enfoque de administración requieren su propia inversión aparte que resulta costosa y compleja.

## <a name="how-does-azure-active-directory-help"></a>¿Cómo ayuda Azure Active Directory?

Azure AD proporciona una exhaustiva administración del acceso para aplicaciones configuradas, lo que permite a las organizaciones lograr fácilmente las directivas de acceso adecuadas: desde asignación automática y basada en atributos (escenarios ABAC o RBAC) hasta la delegación y la administración de administradores. Con Azure AD, se pueden conseguir fácilmente directivas complejas al combinar varios modelos de administración para una sola aplicación, e incluso se pueden volver a usar las reglas de administración entre aplicaciones con las mismas audiencias.

Con Azure AD, los informes de asignación y uso están totalmente integrados, lo que permite a los administradores informar fácilmente sobre el estado y los errores de asignación, e incluso del uso.

### <a name="assigning-users-and-groups-to-an-app"></a>Asignación de usuarios y grupos a una aplicación

La asignación de aplicaciones de Azure AD se centra en dos modos de asignación principales:

* **Asignación individual** : un administrador de TI con permisos de administrador global de directorios puede seleccionar cuentas de usuario individuales y concederles acceso a la aplicación.

* **Asignación basada en grupos (requiere Azure AD Premium P1 o P2)** : un administrador de TI con permisos de administrador global de directorios puede asignar un grupo a la aplicación. El acceso de usuarios específicos se determina en función de si son miembros del grupo al momento de intentar acceder a la aplicación. En otras palabras, un administrador puede crear en la práctica una regla de asignación que diga "cualquier miembro actual del grupo asignado tiene acceso a la aplicación". Con esta opción de asignación, los administradores pueden beneficiarse de cualquier opción de administración de grupos de Azure AD, incluidos [grupos dinámicos basados en atributos](../fundamentals/active-directory-groups-create-azure-portal.md), grupos externos del sistema (por ejemplo, Active Directory local o Workday) o grupos administrados por el administrador o por autoservicio. Un único grupo se puede asignar fácilmente a varias aplicaciones, lo que garantiza que las aplicaciones con afinidad de asignación puedan compartir reglas de asignación y así reducir la complejidad de la administración en general. Tenga en cuenta que no se admiten las pertenencias a grupos anidadas para la asignación basada en grupos a aplicaciones en este momento.

Con estos dos modos de asignación, los administradores pueden conseguir cualquier enfoque deseable de administración de asignaciones.

### <a name="requiring-user-assignment-for-an-app"></a>Requerir la asignación del usuario a una aplicación

Con algunos tipos de aplicaciones, existe la opción de [requerir que los usuarios estén asignados a la aplicación](assign-user-or-group-access-portal.md#configure-an-application-to-require-user-assignment). Al hacerlo, se impide que todos los usuarios puedan iniciar sesión, salvo aquellos que se asignen expresamente a la aplicación. Los siguientes tipos de aplicaciones admiten esta opción:

* Aplicaciones configuradas para el inicio de sesión único (SSO) federado con autenticación basada en SAML
* Aplicaciones de proxy de aplicación que usan la autenticación previa de Azure Active Directory
* Aplicaciones integradas en la plataforma de aplicaciones de Azure AD que usan la autenticación de OAuth 2.0 u OpenID Connect después de que un usuario o un administrador han dado su consentimiento a esa aplicación. Algunas aplicaciones empresariales ofrecen un control adicional sobre quién puede iniciar sesión.

Cuando la asignación de usuarios *no es necesaria*, los usuarios sin asignar no ven la aplicación en sus paneles de acceso Mis aplicaciones, pero sí pueden iniciar sesión en la aplicación en sí (esto se conoce también como inicio de sesión iniciado por el proveedor de servicios). También pueden usar la dirección que aparece en **URL de acceso de usuario** en la página **Propiedades** de la aplicación (esto se conoce también como inicio de sesión iniciado por IDP).

En algunas aplicaciones, la opción para requerir la asignación de usuarios no está disponible en las propiedades de la aplicación. En estos casos, puede usar PowerShell para establecer la propiedad appRoleAssignmentRequired en la entidad de servicio.

### <a name="determining-the-user-experience-for-accessing-apps"></a>Determinación de la experiencia del usuario para acceder a las aplicaciones

Azure AD proporciona [varias maneras personalizables para implementar aplicaciones](end-user-experiences.md) para los usuarios finales de la organización:

* Panel de acceso Mis aplicaciones de Azure AD
* Iniciador de aplicaciones de Office 365
* Inicio de sesión directo en aplicaciones (service-pr)
* Vínculos profundos a aplicaciones federadas, con contraseña o existentes

Puede decidir si los usuarios asignados a una aplicación empresarial pueden ver dicha aplicación en el panel de acceso y en el iniciador de aplicaciones de Office 365.

## <a name="example-complex-application-assignment-with-azure-ad"></a>Ejemplo: Asignación de aplicaciones complejas con Azure AD
Considere una aplicación como Salesforce. En muchas organizaciones, Salesforce se usa principalmente en los equipos de ventas y marketing. A menudo, los miembros del equipo de marketing tienen acceso privilegiado a Salesforce, mientras que los miembros del equipo de ventas tienen acceso limitado. En muchos casos, una amplia población de trabajadores de la información tiene acceso restringido a la aplicación. Las excepciones a estas reglas complican el asunto. Con frecuencia, es prerrogativa de los equipos líderes de marketing o de ventas conceder acceso a un usuario o cambiar su rol independientemente de estas reglas genéricas.

Con Azure AD, las aplicaciones como Salesforce se pueden configurar previamente para el inicio de sesión único (SSO) y el aprovisionamiento automatizado. Después de configurar la aplicación, un administrador puede realizar la acción puntual de crear y asignar los grupos adecuados. En este ejemplo, un administrador puede ejecutar las siguientes asignaciones:

* [grupos dinámicos](../fundamentals/active-directory-groups-create-azure-portal.md) para representar automáticamente a todos los miembros de los equipos de ventas y marketing con atributos como departamento o rol:
  
  * Todos los miembros de los grupos de marketing se asignarían al rol "marketing" en Salesforce.
  * Todos los miembros de los grupos de equipo de ventas se asignarían al rol "ventas" en Salesforce. Para afinar más, se podrían usar varios grupos que representan los equipos de ventas regionales asignados a diferentes roles de Salesforce.

* Para habilitar el mecanismo de excepciones, se podría crear un grupo de autoservicio para cada rol. Por ejemplo, el grupo de "excepción de marketing de Salesforce" se puede crear como un grupo de autoservicio. El grupo se puede asignar al rol de marketing de Salesforce y los integrantes del equipo de liderazgo de marketing se pueden convertir en propietarios. Los miembros del equipo de liderazgo de marketing podrían agregar o quitar usuarios, establecer una directiva de unión o incluso aprobar o rechazar solicitudes de unión de usuarios individuales. Este mecanismo es posible mediante una experiencia adecuada del trabajador de la información en la que los propietarios o miembros no necesitan aprendizaje especializado.

En este caso, todos los usuarios asignados se aprovisionarían automáticamente a Salesforce, ya que como se agregan a diferentes grupos, su asignación de roles se actualizaría en Salesforce. Los usuarios podrán detectar Salesforce y acceder a esta aplicación mediante el panel de acceso a las aplicaciones de Microsoft, los clientes web de Office, o incluso navegando a su página de inicio de sesión organizativa de Salesforce. Los administradores podrán ver fácilmente el estado de uso y asignación mediante los informes de Azure AD.

Los administradores pueden emplear el [acceso condicional de Azure AD](../conditional-access/concept-conditional-access-users-groups.md) para establecer directivas de acceso para roles específicos. Estas directivas pueden incluir si se permite el acceso fuera del entorno corporativo e incluso los requisitos de Multi-Factor Authentication o de los dispositivo para obtener acceso en diversos casos.

## <a name="access-to-microsoft-applications"></a>Acceso a aplicaciones de Microsoft

Las aplicaciones de Microsoft (como Office 365 Exchange, SharePoint, Yammer, etc.) se asignan y administran de manera algo diferente que las aplicaciones SaaS de terceros u otras aplicaciones que integra con Azure AD para el inicio de sesión único.

Hay tres maneras principales en que un usuario puede acceder a una aplicación publicada por Microsoft.

- En el caso de aplicaciones de Office 365 u otros conjuntos de aplicaciones de pago, a los usuarios se les concede acceso mediante la **asignación de licencias** o mediante la funcionalidad de asignación de licencias basada en grupo.
- En las aplicaciones que Microsoft o un tercero publica de manera gratuita para que todo el mundo las use, se puede conceder acceso a los usuarios por medio del [consentimiento del usuario](configure-user-consent.md). Esto significa que inician sesión en la aplicación con su cuenta profesional o educativa de Azure AD, que les permite tener acceso a un conjunto limitado de datos en sus cuentas.
- Además, para aplicaciones que Microsoft o un tercero publica de manera gratuita para que todo el mundo las use, a los usuarios se les puede conceder acceso mediante [consentimiento del administrador](manage-consent-requests.md). Esto significa que un administrador ha determinado que todos los miembros de la organización pueden usar la aplicación, por lo que inicia sesión en la aplicación con una cuenta de administrador global y concede acceso a todos ellos.

Algunas aplicaciones combinan estos métodos. Por ejemplo, algunas aplicaciones de Microsoft forman parte de una suscripción a Office 365, pero siguen requiriendo consentimiento.

Los usuarios pueden acceder a las aplicaciones de Office 365 a través de sus portales de Office 365 correspondientes. También puede mostrar u ocultar las aplicaciones de Office 365 en el panel de acceso Mis aplicaciones con el [botón de alternancia de visibilidad de Office 365](hide-application-from-user-portal.md) en **Configuración de usuario**, en su directorio. 

Al igual que sucede con las aplicaciones empresariales, se pueden [asignar usuarios](assign-user-or-group-access-portal.md) a determinadas aplicaciones de Microsoft a través de Azure Portal o, si la opción de Portal no está disponible, con PowerShell.

## <a name="next-steps"></a>Pasos siguientes
* [Protección de aplicaciones con acceso condicional](../conditional-access/concept-conditional-access-cloud-apps.md)
* [Administración de grupos de autoservicio/SSAA](../users-groups-roles/groups-self-service-management.md)
