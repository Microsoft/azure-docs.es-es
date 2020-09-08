---
title: Guía de referencia de operaciones de administración de autenticación de Azure Active Directory
description: Esta guía de referencia de operaciones describe las comprobaciones y las acciones que debe realizar para proteger la administración de autenticación.
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
ms.openlocfilehash: b45a0608d78330eafd6afc3656f8a4addbdffd3b
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321725"
---
# <a name="azure-active-directory-authentication-management-operations-reference-guide"></a>Guía de referencia de operaciones de administración de autenticación de Azure Active Directory

En esta sección de la [guía de referencia de operaciones de Azure AD](active-directory-ops-guide-intro.md) se describen las comprobaciones y las acciones que debe realizar para proteger y administrar las credenciales, definir la experiencia de autenticación, delegar la asignación, medir el uso y definir directivas de acceso basadas en la posición de seguridad de la empresa.

> [!NOTE]
> Estas recomendaciones están actualizadas hasta la fecha de publicación, pero pueden cambiar con el tiempo. Las organizaciones deben evaluar continuamente sus prácticas de identidad a medida que los productos y servicios de Microsoft evolucionen.

## <a name="key-operational-processes"></a>Procesos operativos clave

### <a name="assign-owners-to-key-tasks"></a>Asignación de propietarios a las tareas clave

La administración de Azure Active Directory requiere la ejecución continua de tareas y procesos operativos clave que pueden no formar parte de un proyecto de lanzamiento. Aun así, es importante que configure estas tareas con miras a optimizar su entorno. Entre las tareas clave y sus propietarios recomendados se incluyen:

| Tarea | Propietario |
| :- | :- |
| Administrar el ciclo de vida de la configuración de inicio de sesión único (SSO) en Azure AD | Equipo de operaciones IAM |
| Diseñar directivas de acceso condicional para aplicaciones de Azure AD | Equipo de arquitectura de InfoSec |
| Archivar la actividad de inicio de sesión en un sistema SIEM | Equipo de operaciones de InfoSec |
| Archivar eventos de riesgo en un sistema SIEM | Equipo de operaciones de InfoSec |
| Evaluar las propiedades e investigar las alertas de seguridad | Equipo de operaciones de InfoSec |
| Evaluar las propiedades e investigar los eventos de seguridad | Equipo de operaciones de InfoSec |
| Evaluar las propiedades e investigar los usuarios marcados en informes de vulnerabilidades y riesgos desde Azure AD Identity Protection | Equipo de operaciones de InfoSec |

> [!NOTE]
> Azure AD Identity Protection requiere una licencia de Azure AD Premium P2. Para obtener la licencia correcta para sus requisitos, consulte  [Comparación de las características con disponibilidad general de las ediciones Azure AD Free y Azure AD Premium](https://azure.microsoft.com/pricing/details/active-directory/).

A medida que revise la lista, es posible que tenga que asignar un propietario a las tareas que no tienen uno o ajustar la propiedad de aquellas tareas cuyos propietarios no se ajustan a las recomendaciones anteriores.

#### <a name="owner-recommended-reading"></a>Lecturas recomendadas para propietarios

- [Asignación de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md)
- [Gobernanza en Azure](../../governance/index.yml)

## <a name="credentials-management"></a>Administración de credenciales

### <a name="password-policies"></a>Directivas de contraseña

Administrar contraseñas de forma segura es una de las partes más destacadas de la administración de identidades y acceso y, a menudo, el objetivo más importante de los ataques. Azure AD admite varias características que pueden ayudarle a evitar que un ataque tenga éxito.

Use la tabla siguiente para encontrar la solución recomendada para mitigar el problema que debe abordarse:

| Problema | Recomendación |
| :- | :- |
| No hay ningún mecanismo que le proteja contra contraseñas poco seguras | Habilite el [autoservicio de restablecimiento de contraseña (SSPR)](../authentication/concept-sspr-howitworks.md) y la [protección con contraseña](../authentication/concept-password-ban-bad-on-premises.md) de Azure AD. |
| No hay ningún mecanismo para detectar contraseñas filtradas | Habilite la [sincronización de hash de contraseñas](../hybrid/how-to-connect-password-hash-synchronization.md) (PHS) para obtener más detalles. |
| Se está usando AD FS y no es posible ir a la autenticación administrada | Habilite el [bloqueo inteligente de la extranet de AD FS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-smart-lockout-protection) o el [bloqueo inteligente de Azure AD](../authentication/howto-password-smart-lockout.md). |
| La directiva de contraseñas usa reglas basadas en la complejidad, como la longitud, los conjuntos de varios caracteres o la caducidad | Reconsidere usar los [procedimientos recomendados de Microsoft](https://www.microsoft.com/research/publication/password-guidance/?from=http%3A%2F%2Fresearch.microsoft.com%2Fpubs%2F265143%2Fmicrosoft_password_guidance.pdf),cambie su enfoque a la administración de contraseñas e implemente la [protección con contraseñas de Azure AD](../authentication/concept-password-ban-bad.md). |
| Los usuarios no están registrados para usar la autenticación multifactor (MFA) | [Registre toda la información de seguridad del usuario](../identity-protection/howto-identity-protection-configure-mfa-policy.md) de modo que se pueda usar como mecanismo para comprobar la identidad del usuario junto con su contraseña. |
| No hay ninguna revocación de contraseñas en función del riesgo del usuario | Implemente [directivas de riesgo de usuario de Azure AD Identity Protection](../identity-protection/howto-identity-protection-configure-risk-policies.md) para forzar cambios de contraseña en credenciales perdidas mediante SSPR. |
| No hay ningún mecanismo de bloqueo inteligente para proteger la autenticación malintencionada de actores no válidos procedentes de direcciones IP identificadas | Implemente la autenticación administrada mediante la nube con la sincronización de hash de contraseñas o mediante la [autenticación de paso a través](../hybrid/how-to-connect-pta-quick-start.md) (PTA). |

#### <a name="password-policies-recommended-reading"></a>Lectura recomendada de directivas de contraseñas

- [Procedimientos recomendados de Azure AD y AD FS: defensa contra ataques de difusión de contraseña: Enterprise Mobility + Security](https://cloudblogs.microsoft.com/enterprisemobility/2018/03/05/azure-ad-and-adfs-best-practices-defending-against-password-spray-attacks/)

### <a name="enable-self-service-password-reset-and-password-protection"></a>Habilitar el autoservicio de restablecimiento de contraseña y la protección con contraseña

Los usuarios que necesitan cambiar o restablecer sus contraseñas son uno de los principales orígenes de volumen y costo debido a las llamadas que realizan al departamento de soporte técnico. Además del costo, cambiar la contraseña como una herramienta para mitigar el riesgo de un usuario es un paso fundamental para mejorar la posición de seguridad de su organización.

Como mínimo, se recomienda implementar el [autoservicio de restablecimiento de contraseña](../authentication/concept-sspr-howitworks.md) (SSPR) de Azure AD y la [protección de contraseñas](../authentication/howto-password-ban-bad-on-premises-deploy.md) local para realizar lo siguiente:

- Desviar las llamadas del departamento de soporte técnico.
- Reemplazar el uso de contraseñas temporales.
- Reemplazar cualquier solución de autoservicio de administración de contraseñas existente que se base en una solución local.
- [Eliminar las contraseñas no seguras](../authentication/concept-password-ban-bad.md) de su organización.

> [!NOTE]
> En el caso de las organizaciones con una suscripción de Azure AD Premium P2, se recomienda implementar SSPR y usarlo como parte de una [directiva de riesgo de usuario de Identity Protection](../identity-protection/howto-identity-protection-configure-risk-policies.md).

### <a name="strong-credential-management"></a>Administración segura de credenciales

Las contraseñas por sí mismas no son lo suficientemente seguras para evitar que los actores no válidos obtengan acceso a su entorno. Como mínimo, cualquier usuario con una cuenta con privilegios debe estar habilitado para poder realizar la autenticación multifactor (MFA). Idealmente, debe habilitar el [registro combinado](../authentication/concept-registration-mfa-sspr-combined.md) y requerir que todos los usuarios se registren en MFA y SSPR mediante la [experiencia de registro combinada](../user-help/security-info-setup-signin.md). Asimismo, se recomienda adoptar una estrategia para [proporcionar resistencia](../authentication/concept-resilient-controls.md) y así reducir el riesgo de bloqueo debido a circunstancias imprevistas.

![Flujo de experiencia de usuario combinado](./media/active-directory-ops-guide/active-directory-ops-img4.png)

### <a name="on-premises-outage-authentication-resiliency"></a>Resistencia de la autenticación a las interrupciones locales

Además de las ventajas de la simplicidad y la posibilidad de habilitar la detección de credenciales filtradas, la sincronización de hash de contraseñas (PHS) de Azure AD y Azure MFA permiten a los usuarios obtener acceso a las aplicaciones SaaS y a Office 365 a pesar de las interrupciones locales debido a ciberataques, como [NotPetya](https://www.microsoft.com/security/blog/2018/02/05/overview-of-petya-a-rapid-cyberattack/). También es posible habilitar PHS mientras esté usando la federación. Si habilita PHS, podrá realizar una reserva de autenticación cuando los servicios de federación no estén disponibles.

Si la organización local no tiene una estrategia de resistencia frente a interrupciones o tiene una que no está integrada con Azure AD, debe implementar PHS de Azure AD y definir un plan de recuperación ante desastres que incluya PHS. Si habilita PHS de Azure AD, los usuarios podrán autenticarse en Azure AD si su instancia de Active Directory local no está disponible.

![flujo de sincronización de hash de contraseñas](./media/active-directory-ops-guide/active-directory-ops-img5.png)

Para comprender mejor las opciones de autenticación, consulte [Elegir el método de autenticación adecuado para la solución de identidad híbrida de Azure Active Directory](../hybrid/choose-ad-authn.md).

### <a name="programmatic-usage-of-credentials"></a>Uso de credenciales mediante programación

Los scripts de Azure AD que utilizan PowerShell o las aplicaciones que usan Microsoft Graph API requieren una autenticación segura. La mala administración de credenciales que ejecutan esos scripts y herramientas aumenta el riesgo de robo de credenciales. Si usa scripts o aplicaciones que se basan en contraseñas codificadas de forma rígida o en mensajes de contraseñas, primero debe revisar las contraseñas de los archivos de configuración o el código fuente y, a continuación, reemplazar esas dependencias y usar las identidades administradas de Azure, la autenticación integrada de Windows o los [certificados](../reports-monitoring/tutorial-access-api-with-certificates.md) siempre que sea posible. En cuanto a aplicaciones donde no se puedan aplicar las soluciones anteriores, use [Azure Key Vault](https://azure.microsoft.com/services/key-vault/).

Si ve que hay entidades de servicio con credenciales de contraseña y no está seguro de cómo los scripts o las aplicaciones protegen esas credenciales de contraseña, póngase en contacto con el propietario de la aplicación para comprender mejor los patrones de uso.

Microsoft también le recomienda que se ponga en contacto con los propietarios de las aplicaciones para comprender los patrones de uso, si es que existen entidades de servicio con credenciales de contraseña.

## <a name="authentication-experience"></a>Experiencia de autenticación

### <a name="on-premises-authentication"></a>Autenticación local

La autenticación federada con la Autenticación integrada de Windows (IWA) o la autenticación administrada de inicio de sesión único (SSO) de conexión directa con la sincronización de hash de contraseña o la autenticación de paso a través, es la mejor experiencia de usuario cuando use la red corporativa relacionada con los controladores de dominio locales. Gracias a ello, se minimiza la fatiga de la petición de credenciales y reduce el riesgo de que los usuarios caigan en ataques de suplantación de identidad. Si ya usa la autenticación administrada en la nube con PHS o PTA, pero los usuarios todavía tienen que escribir su contraseña al autenticarse de forma local, debe implementar de inmediato el [SSO de conexión directa](../hybrid/how-to-connect-sso.md). Por otro lado, si está federado con planes para migrar definitivamente a la autenticación administrada en la nube, debe implementar el SSO de conexión directa como parte del proyecto de migración.

### <a name="device-trust-access-policies"></a>Directivas de acceso de confianza de dispositivos

Al igual que un usuario de su organización, un dispositivo es una identidad principal que desea proteger. Puede usar una identidad de dispositivo para proteger los recursos en cualquier momento y ubicación. La autenticación del dispositivo y tener en cuenta su tipo de confianza mejoran la seguridad y facilidad de uso, ya que:

- Se evita la fricción, por ejemplo, con MFA, cuando el dispositivo es de confianza.
- Se bloquea el acceso desde dispositivos que no son de confianza.
- En el caso de los dispositivos Windows 10, es posible usar el [inicio de sesión único en los recursos locales sin problemas](../devices/azuread-join-sso.md).

Puede llevar a cabo este objetivo mediante la incorporación de identidades de dispositivo, y administrarlas en Azure AD mediante uno de los métodos siguientes:

- Las organizaciones pueden usar [Microsoft Intune](/intune/what-is-intune) para administrar el dispositivo y aplicar las directivas de cumplimiento, atestar el estado del dispositivo y establecer directivas de acceso condicional en función de si el dispositivo es compatible. Microsoft Intune puede administrar dispositivos iOS, dispositivos de escritorio Mac (a través de la integración JAMF), dispositivos de escritorio de Windows (de modo nativo mediante la administración de dispositivos móviles para Windows 10 y la administración conjunta con Microsoft Endpoint Configuration Manager) y dispositivos móviles Android.
- La [combinación de Azure AD híbrido](../devices/hybrid-azuread-join-managed-domains.md) proporciona la opción de administrar contenido con directivas de grupo o Microsoft Endpoint Configuration Manager en un entorno con equipos unidos a un dominio de Active Directory. Las organizaciones pueden implementar un entorno administrado a través de PHS o PTA con un SSO de conexión directa. Si lleva sus dispositivos a Azure AD, podrá maximizar la productividad de los usuarios a través de SSO en los recursos locales y en la nube, a la vez que protegerá el acceso a los recursos en la nube y locales con la opción de  [acceso condicional](../conditional-access/overview.md) al mismo tiempo.

Si tiene dispositivos Windows unidos a un dominio que no están registrados en la nube, o dispositivos Windows unidos a un dominio que sí están registrados en la nube, pero sin directivas de acceso condicional, debe registrar los dispositivos no registrados y, en cualquier caso, [usar la combinación de Azure AD híbrido como control](../conditional-access/require-managed-devices.md) de las directivas de acceso condicional.

![Una captura de pantalla para la concesión de acceso en la directiva de acceso condicional que requiere un dispositivo híbrido](./media/active-directory-ops-guide/active-directory-ops-img6.png)

Si está administrando dispositivos con MDM o Microsoft Intune, pero no usa controles de dispositivo en las directivas de acceso condicional, se recomienda usar la opción [Requerir que el dispositivo esté marcado como compatible](../conditional-access/require-managed-devices.md#require-device-to-be-marked-as-compliant) como control en esas directivas.

![Una captura de pantalla para la concesión de acceso en la directiva de acceso condicional que requiere un dispositivo compatible](./media/active-directory-ops-guide/active-directory-ops-img7.png)

#### <a name="device-trust-access-policies-recommended-reading"></a>Lectura recomendada de directivas de acceso de confianza al dispositivo

- [Cómo: Planificar la implementación de la combinación a Azure Active Directory híbrido](../devices/hybrid-azuread-join-plan.md)
- [Configuraciones de acceso de dispositivos e identidades](/microsoft-365/enterprise/microsoft-365-policies-configurations)

### <a name="windows-hello-for-business"></a>Windows Hello para empresas

En Windows 10, [Windows Hello para empresas](/windows/security/identity-protection/hello-for-business/hello-identity-verification) reemplaza las contraseñas por la autenticación en dos fases segura en equipos. Windows Hello para empresas permite obtener una experiencia de MFA más simplificada para los usuarios y reduce la dependencia de las contraseñas. Si no ha empezado a implementar dispositivos con Windows 10 o solo los ha implementado parcialmente, se recomienda actualizar a Windows 10 y [habilitar Windows Hello para empresas](/windows/security/identity-protection/hello-for-business/hello-manage-in-organization) en todos los dispositivos.

Si quiere obtener más información sobre la autenticación sin contraseñas, consulte [Un mundo sin contraseñas con Azure Active Directory](../authentication/concept-authentication-passwordless.md).

## <a name="application-authentication-and-assignment"></a>Asignación y autenticación de la aplicación

### <a name="single-sign-on-for-apps"></a>Inicio de sesión único para aplicaciones

Proporcionar un mecanismo estándar de inicio de sesión único para toda la empresa es fundamental para mejorar la experiencia del usuario, reducir el riesgo, generar informes y facilitar el gobierno. Si usa aplicaciones que admiten SSO con Azure AD pero actualmente están configuradas para usar cuentas locales, debe volver a configurar esas aplicaciones para poder usar SSO con Azure AD. Del mismo modo, si usa cualquier aplicación que admita SSO con Azure AD pero usa otro proveedor de identidades, debe volver a configurar esas aplicaciones para usar SSO con Azure AD también. En el caso de las aplicaciones que no admiten protocolos de federación, pero que admiten la autenticación basada en formularios, se recomienda que configure la aplicación para usar el [almacenamiento de contraseñas](../manage-apps/application-proxy-configure-single-sign-on-password-vaulting.md) con Azure AD Application Proxy.

![Inicio de sesión basado en contraseñas de AppProxy](./media/active-directory-ops-guide/active-directory-ops-img8.png)

> [!NOTE]
> Si no tiene un mecanismo para detectar aplicaciones no administradas en su organización, le recomendamos que implemente un proceso de detección mediante una solución de agente de seguridad de acceso a la nube (CASB), como [Microsoft Cloud App Security](https://www.microsoft.com/enterprise-mobility-security/cloud-app-security).

Por último, si tiene una galería de aplicaciones de Azure AD y usa aplicaciones que admiten SSO con Azure AD, es recomendable [enumerar la aplicación en la galería de aplicaciones](../azuread-dev/howto-app-gallery-listing.md).

#### <a name="single-sign-on-recommended-reading"></a>Lectura recomendada del inicio de sesión único

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

### <a name="migration-of-ad-fs-applications-to-azure-ad"></a>Migración de aplicaciones de AD FS a Azure AD

[La migración de aplicaciones desde AD FS a Azure AD](../manage-apps/migrate-adfs-apps-to-azure.md) habilita funcionalidades adicionales en cuanto a seguridad, capacidad de administración más coherente y una mejor experiencia de colaboración. Si tiene aplicaciones configuradas en AD FS que admiten SSO con Azure AD, debe volver a configurar esas aplicaciones para que usen SSO con Azure AD. Si tiene aplicaciones configuradas en AD FS con configuraciones poco comunes y que no admite Azure AD, debe ponerse en contacto con los propietarios de la aplicación para saber si la configuración especial es un requisito absoluto de la aplicación. Si no es necesario, debe volver a configurar la aplicación para usar SSO con Azure AD.

![Azure AD como proveedor de identidades principal](./media/active-directory-ops-guide/active-directory-ops-img9.png)

> [!NOTE]
> [Azure AD Connect Health para ADFS](../hybrid/how-to-connect-health-adfs.md) se puede usar para recopilar los detalles de configuración de cada aplicación que se pueda migrar a Azure AD.

### <a name="assign-users-to-applications"></a>Asignar usuarios a aplicaciones

La [asignación de usuarios a las aplicaciones](../manage-apps/assign-user-or-group-access-portal.md) se realiza mejor cuando se usan grupos, porque permiten una gran flexibilidad y la posibilidad de administrarlos a escala. Las ventajas de usar grupos incluyen las opciones de [pertenencia dinámica a grupos basada en atributos](../users-groups-roles/groups-dynamic-membership.md) y de [delegación a los propietarios de las aplicaciones](../fundamentals/active-directory-accessmanagement-managing-group-owners.md). Por lo tanto, si ya usa y administra grupos, es recomendable realizar las siguientes acciones para mejorar la administración a escala:

- Delegar la administración y la gobernanza de grupos a los propietarios de la aplicación.
- Permitir el acceso de autoservicio a la aplicación.
- Definir grupos dinámicos si los atributos de usuario pueden determinar de forma coherente el acceso a las aplicaciones.
- Implementar la atestación en los grupos que se usan para obtener acceso a las aplicaciones mediante la s[revisiones de acceso de Azure AD](../governance/access-reviews-overview.md).

Por otro lado, si encuentra aplicaciones que tienen una asignación a usuarios individuales, asegúrese de implementar la [gobernanza](../governance/index.yml) en esas aplicaciones.

#### <a name="assign-users-to-applications-recommended-reading"></a>Lectura recomendada para asignar usuarios a aplicaciones

- [Asignación de usuarios y grupos en una aplicación de Azure Active Directory](../manage-apps/assign-user-or-group-access-portal.md)
- [Delegación de permisos de registro de aplicaciones en Azure Active Directory](../users-groups-roles/roles-delegate-app-roles.md)
- [Reglas de pertenencia dinámica a grupos de Azure Active Directory](../users-groups-roles/groups-dynamic-membership.md)

## <a name="access-policies"></a>Directivas de acceso

### <a name="named-locations"></a>Ubicaciones con nombre

Gracias a las [ubicaciones con nombre](../reports-monitoring/quickstart-configure-named-locations.md) en Azure AD, puede etiquetar intervalos de direcciones IP de confianza en su organización. Azure AD usa las ubicaciones con nombre para:

- Prevenir falsos positivos en eventos de riesgo. Si se inicia sesión desde una ubicación de confianza se reduce el riesgo en el inicio de sesión del usuario.
- Configurar el [acceso condicional basado en la ubicación](../reports-monitoring/quickstart-configure-named-locations.md).

![Ubicación con nombre](./media/active-directory-ops-guide/active-directory-ops-img10.png)

Según la prioridad, use la tabla siguiente para encontrar la solución recomendada que mejor se adapte a las necesidades de su organización:

| **Prioridad** | **Escenario** | **Recomendación** |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| 1 | Si usa PHS o PTA y no se han definido las ubicaciones con nombre | Defina las ubicaciones con nombre para mejorar la detección de eventos de riesgo. |
| 2 | Si está federado y no usa la notificaciones de tipo "insideCorporateNetwork" y si no se han definido las ubicaciones con nombre | Defina las ubicaciones con nombre para mejorar la detección de eventos de riesgo. |
| 3 | Si no usa ubicaciones con nombre en las directivas de acceso condicional y no hay ningún riesgo ni controles de dispositivo en las directivas de acceso condicional | Configure la directiva de acceso condicional para incluir ubicaciones con nombre. |
| 4 | Si está federado y usa la notificación de tipo "insideCorporateNetwork" y no se han definido las ubicaciones con nombre | Defina las ubicaciones con nombre para mejorar la detección de eventos de riesgo. |
| 5 | Si usa direcciones IP de confianza con MFA en lugar de ubicaciones con nombre y las marca como de confianza | Defina ubicaciones con nombre y márquelas como de confianza para mejorar la detección de eventos de riesgo. |

### <a name="risk-based-access-policies"></a>Directivas de acceso basadas en riesgos

Azure AD puede calcular el riesgo de todos los inicios de sesión y de todos los usuarios. El uso del riesgo como criterio en las directivas de acceso puede proporcionar una experiencia de usuario mejor (por ejemplo, recibirá menos mensajes de autenticación y la seguridad será superior); solo debe preguntar a los usuarios cuando sea necesario y automatizar la respuesta y la corrección.

![Directiva de riesgo de inicio de sesión](./media/active-directory-ops-guide/active-directory-ops-img11.png)

Si ya posee licencias de Azure AD Premium P2 que admiten el uso del riesgo en las directivas de acceso, pero no se usan estas directivas, le recomendamos encarecidamente agregar riesgo a su posición de seguridad.

#### <a name="risk-based-access-policies-recommended-reading"></a>Lectura recomendada de directivas de acceso basadas en riesgos

- [Cómo: configurar la directiva de riesgo de inicio de sesión](../identity-protection/howto-identity-protection-configure-risk-policies.md)
- [Cómo: configurar la directiva de riesgo de usuario](../identity-protection/howto-identity-protection-configure-risk-policies.md)

### <a name="client-application-access-policies"></a>Directivas de acceso de aplicaciones cliente

La administración de aplicaciones de Microsoft Intune (MAM) ofrece la posibilidad de incorporar controles de protección de datos como el cifrado de almacenamiento, el PIN o la limpieza remota del almacenamiento a aplicaciones móviles cliente compatibles, como Outlook Mobile. Además, se pueden crear directivas de acceso condicional para [restringir el acceso](../conditional-access/app-based-conditional-access.md) a servicios en la nube, como Exchange Online, desde aplicaciones aprobadas o compatibles.

Si los empleados instalan aplicaciones compatibles con MAM, como aplicaciones móviles de Office, para obtener acceso a recursos corporativos como Exchange Online o SharePoint Online, y si también admite la opción BYOD (traiga su propio dispositivo), es recomendable implementar las directivas de MAM de la aplicación para administrar la configuración de la aplicación en dispositivos de su propiedad sin tener que realizar la inscripción de MDM; a continuación, solo debe actualizar las directivas de acceso condicional para permitir el acceso solo desde clientes compatibles con MAM.

![Control de concesiones de acceso condicional](./media/active-directory-ops-guide/active-directory-ops-img12.png)

Si los empleados instalan aplicaciones compatibles con MAM en recursos corporativos y el acceso está restringido en los dispositivos que administra Intune, debe considerar la posibilidad de implementar directivas MAM de aplicaciones, para así poder administrar la configuración de la aplicación en dispositivos personales. Actualice las directivas de acceso condicional para permitir el acceso solo desde clientes compatibles con MAM.

### <a name="conditional-access-implementation"></a>Implementación del acceso condicional

El acceso condicional es una herramienta esencial para mejorar la posición de seguridad de su organización. Por lo tanto, es importante que siga estos procedimientos recomendados:

- Asegúrese de que todas las aplicaciones SaaS tienen al menos una directiva aplicada.
- Evite combinar el filtro **Todas las aplicaciones** con el control de **bloqueo**, para evitar el riesgo de bloqueo.
- Evite usar la opción **Todos los usuarios** como filtro y agregar sin querer **Invitados**.
- **Migre todas las directivas "heredadas" a Azure Portal**
- Recopile todos los criterios de los usuarios, los dispositivos y las aplicaciones.
- Use las directivas de acceso condicional para [implementar MFA](../conditional-access/plan-conditional-access.md), en lugar de usar un **MFA por usuario**.
- Conserve un pequeño conjunto de directivas básicas que se puedan aplicar a varias aplicaciones.
- Defina grupos de excepciones vacíos y agréguelos a las directivas para tener una estrategia de excepción.
- Planifique las cuentas de [emergencia ](../users-groups-roles/directory-admin-roles-secure.md#break-glass-what-to-do-in-an-emergency) sin controles MFA.
- Asegúrese de que proporciona una experiencia coherente entre las aplicaciones cliente de Office 365 (por ejemplo, Teams, OneDrive para la Empresa o Outlook) al implementar el mismo conjunto de controles para servicios como Exchange Online y SharePoint Online.
- Recuerde que la asignación a directivas debe implementarse a través de grupos, no de personas.
- Realice revisiones periódicas de los grupos de excepciones que se usan en las directivas para limitar el tiempo que los usuarios no usan la posición de seguridad. Si tiene Azure AD P2, puede usar las revisiones de acceso para automatizar el proceso.

#### <a name="conditional-access-recommended-reading"></a>Lectura recomendada del acceso condicional

- [Procedimientos recomendados para el acceso condicional en Azure Active Directory](../conditional-access/best-practices.md)
- [Configuraciones de acceso de dispositivos e identidades](/microsoft-365/enterprise/microsoft-365-policies-configurations)
- [Referencia de configuración del acceso condicional de Azure Active Directory](../conditional-access/concept-conditional-access-conditions.md)
- [Directivas de acceso condicional habituales](../conditional-access/concept-conditional-access-policy-common.md)

## <a name="access-surface-area"></a>Área expuesta de acceso

### <a name="legacy-authentication"></a>Autenticación heredada

Las credenciales seguras, como MFA, no pueden proteger las aplicaciones que usan protocolos de autenticación heredados, por lo que se convierten en el objetivo de ataque preferido de los actores malintencionados. Bloquear la autenticación heredada es fundamental para mejorar la posición de seguridad de acceso.

La autenticación heredada es un término que hace referencia a los protocolos de autenticación que usan aplicaciones como:

- Clientes de Office antiguos que no usan la autenticación moderna (por ejemplo, un cliente de Office 2010).
- Clientes que usan protocolos de correo electrónico como IMAP/SMTP/POP.

Los atacantes prefieren estos protocolos; de hecho, casi el [100 % de los ataques de difusión de contraseñas](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Your-Pa-word-doesn-t-matter/ba-p/731984) usan protocolos de autenticación heredados. Los hackers usan protocolos de autenticación heredados, ya que no admiten el inicio de sesión interactivo, que es necesario para superar otros desafíos de seguridad como la autenticación multifactor y la autenticación de dispositivos.

Si la autenticación heredada se usa ampliamente en su entorno, debe planear la migración de los clientes heredados a clientes que admitan una [autenticación moderna](/office365/enterprise/modern-auth-for-office-2013-and-2016) lo antes posible. En el mismo token, si tiene algunos usuarios que ya usan la autenticación moderna, pero otros que siguen usando la autenticación heredada, debe realizar los siguientes pasos para bloquear a los clientes de autenticación heredada:

1. Use los [informes de actividad de inicio de sesión](../reports-monitoring/concept-sign-ins.md) para identificar a los usuarios que siguen usando la autenticación heredada y la corrección de planes:

   a. Actualice los clientes con capacidad de autenticación moderna a los usuarios afectados.
   
   b. Planifique un período de tiempo de transición para realizar un bloqueo según los pasos siguientes.
   
   c. Identifique las aplicaciones heredadas que tengan una dependencia permanente en la autenticación heredada. Consulte el paso 3 que tiene a continuación.

2. Deshabilite los protocolos heredados en el origen (por ejemplo, el buzón de correo de Exchange) para los usuarios que no usen la autenticación heredada para así evitar una mayor exposición.
3. En el caso de las cuentas restantes (idealmente, son las identidades no humanas, como las cuentas de servicio), use el [acceso condicional para restringir los protocolos heredados](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-Conditional-Access-support-for-blocking-legacy-auth-is/ba-p/245417) después de la autenticación.

#### <a name="legacy-authentication-recommended-reading"></a>Lectura recomendada de la autenticación heredada

- [Habilitar o deshabilitar el acceso POP3 o IMAP4 a los buzones de Exchange Server](/exchange/clients/pop3-and-imap4/configure-mailbox-access?view=exchserver-2019)

### <a name="consent-grants"></a>Otorgar consentimientos

En un ataque para otorgar consentimientos ilícito, el atacante crea una aplicación registrada en Azure AD que solicita acceso a ciertos datos, como la información de contacto, el correo electrónico o los documentos. Los usuarios pueden otorgar su consentimiento a aplicaciones malintencionadas a través de ataques de suplantación de identidad (phishing) al acceder a sitios web malintencionados.

A continuación se muestra una lista de aplicaciones con permisos que es posible quiera examinar para los servicios en la nube de Microsoft:

- Aplicaciones con permisos \*.ReadWrite delegados o de aplicación
- Aplicaciones con permisos delegados que pueden leer, enviar o administrar el correo electrónico en nombre del usuario
- Aplicaciones a las que se concede el uso de los siguientes permisos:

| Resource | Permiso |
| :- | :- |
| Office 365 Exchange Online | EAS.AccessAsUser.All |
| | EWS.AccessAsUser.All |
| | Mail.Read |
| Microsoft Graph API | Mail.Read |
| | Mail.Read.Shared |
| | Mail.ReadWrite |

- Aplicaciones a las que se le concede la suplantación completa del usuario que inició sesión. Por ejemplo:

|Resource | Permiso |
| :- | :- |
| Microsoft Graph API| Directory.AccessAsUser.All |
| API REST de Azure | user_impersonation |

Para evitar este escenario, consulte [Detectar y solucionar la concesión de consentimiento ilegal en Office 365](/office365/securitycompliance/detect-and-remediate-illicit-consent-grants) para identificar y corregir cualquier aplicación con concesiones ilícitas o aplicaciones que tengan más concesiones de las necesarias. A continuación, [quite el autoservicio por completo](../manage-apps/configure-user-consent.md) y [establezca los procedimientos de gobernanza](../manage-apps/configure-admin-consent-workflow.md). Por último, programe revisiones periódicas de los permisos de la aplicación y quítelos cuando no sean necesarios.

#### <a name="consent-grants-recommended-reading"></a>Lectura recomendada para otorgar consentimientos

- [Permisos para Microsoft Graph API](/graph/permissions-reference)

### <a name="user-and-group-settings"></a>Configuración de usuario y de grupo

A continuación, se muestra la configuración de usuario y de grupo que se puede bloquear si no existe una necesidad empresarial explícita:

#### <a name="user-settings"></a>Configuración de usuario

- **Usuarios externos**: la colaboración externa puede producirse de forma orgánica en la empresa mediante servicios como Teams, Power BI, SharePoint Online y Azure Information Protection. Si tiene restricciones explícitas para controlar cualquier colaboración externa que inicie el usuario, es recomendable que habilite usuarios externos mediante la [administración de derechos de Azure AD](../governance/entitlement-management-overview.md) o con una operación controlada como, por ejemplo, a través del departamento de soporte técnico. Si no quiere permitir la colaboración externa orgánica en los servicios, puede [impedir totalmente que los miembros inviten a usuarios externos](../external-identities/delegate-invitations.md). Como alternativa, también puede [permitir o bloquear dominios específicos](../external-identities/allow-deny-list.md) en invitaciones de usuarios externos.
- **Registros de aplicaciones**: cuando la característica Registros de aplicaciones está habilitada, los usuarios finales pueden incorporar aplicaciones y conceder acceso a sus datos. Un ejemplo típico de la característica Registro de aplicaciones, son los usuarios que habilitan los complementos de Outlook o los asistentes de voz como Alexa y Siri que se usan para leer los correos electrónicos y el calendario, o para enviar correos electrónicos en su nombre. Si el cliente decide desactivar Registro de aplicaciones, los equipos de InfoSec e IAM deben participar en la administración de excepciones (esto es, registros de aplicaciones que son necesarios según los requisitos empresariales), ya que tendrán que registrar las aplicaciones con una cuenta de administrador, y lo más probable es que necesite diseñar un proceso para poner en marcha esta operación.
- **Portal de administración**: las organizaciones pueden bloquear la hoja de Azure AD en Azure Portal para que los usuarios que no sean administradores no puedan obtener acceso a la administración de Azure AD en Azure Portal y se confundan debido a ello. Vaya a la configuración de usuario en el portal de administración de Azure AD para restringir el acceso:

![Acceso restringido del portal de administración](./media/active-directory-ops-guide/active-directory-ops-img13.png)

> [!NOTE]
> Los usuarios que no son administradores todavía pueden obtener acceso a las interfaces de administración de Azure AD a través de la línea de comandos u otras interfaces de programación.

#### <a name="group-settings"></a>Configuración de grupo

**Administración de grupos de autoservicio: los usuarios pueden crear grupos de seguridad o grupos de O365.** Si no hay ninguna iniciativa de autoservicio actual para los grupos en la nube, los clientes pueden decidir desactivarla hasta que estén listos para usar esta funcionalidad.

#### <a name="groups-recommended-reading"></a>Lectura recomendada de grupos

- [¿Qué es la colaboración B2B de Azure Active Directory?](../external-identities/what-is-b2b.md)
- [Integración de aplicaciones con Azure Active Directory](../develop/quickstart-register-app.md)
- [Aplicaciones, permisos y consentimiento en Azure Active Directory](../develop/quickstart-register-app.md)
- [Uso de grupos para administrar el acceso a recursos en Azure Active Directory](./active-directory-manage-groups.md)
- [Configuración de la administración del acceso a aplicaciones de autoservicio en Azure Active Directory](../users-groups-roles/groups-self-service-management.md)

### <a name="traffic-from-unexpected-locations"></a>Tráfico desde ubicaciones inesperadas

Los atacantes son de varias partes del mundo. Administre este riesgo mediante el uso de directivas de acceso condicional con la ubicación establecida como condición. La [condición de ubicación](../conditional-access/location-condition.md) de una directiva de acceso condicional le permite bloquear el acceso a ubicaciones donde no haya ningún motivo empresarial debido al cual se deba iniciar sesión.

![Creación de una nueva ubicación con nombre](./media/active-directory-ops-guide/active-directory-ops-img14.png)

Si está disponible, use una solución de Administración de eventos e información de seguridad (SIEM) para analizar y buscar patrones de acceso entre regiones. Si no usa un producto SIEM o no ingiere información de autenticación de Azure AD, se recomienda usar [Azure Monitor](../../azure-monitor/overview.md) para identificar patrones de acceso entre regiones.

## <a name="access-usage"></a>Uso de acceso

### <a name="azure-ad-logs-archived-and-integrated-with-incident-response-plans"></a>Registros de Azure AD archivados e integrados con planes de respuesta a incidentes

Tener acceso a la actividad de inicio de sesión, las auditorías y los eventos de riesgo de Azure AD es fundamental para la solución de problemas, el análisis de uso y las investigaciones de análisis forense. Azure AD proporciona acceso a estos orígenes a través de las API de REST que tienen un período de retención limitado. Un sistema de Administración de eventos e información de seguridad (SIEM) o una tecnología de archivo equivalente, es fundamental para el almacenamiento a largo plazo de auditorías y compatibilidad. Para habilitar el almacenamiento a largo plazo de los registros de Azure AD, debe agregarlos a la solución SIEM existente o usar [Azure Monitor](../reports-monitoring/concept-activity-logs-azure-monitor.md). Archive los registros que se puedan usar como parte de las investigaciones y los planes de respuesta a incidentes.

#### <a name="logs-recommended-reading"></a>Lectura recomendada de registros

- [Referencia de la API de auditoría de Azure Active Directory](/graph/api/resources/directoryaudit?view=graph-rest-beta%3fview%3dgraph-rest-beta)
- [Referencia de la API de informes de actividad de inicio de sesión de Azure Active Directory](/graph/api/resources/signin?view=graph-rest-beta%3fview%3dgraph-rest-beta)
- [Obtención de datos mediante la API de informes de Azure AD con certificados](../reports-monitoring/tutorial-access-api-with-certificates.md)
- [Microsoft Graph para Azure Active Directory Identity Protection](../identity-protection/howto-identity-protection-graph-api.md)
- [Referencia de la API de actividad de administración de Office 365](/office/office-365-management-api/office-365-management-activity-api-reference)
- [Uso del paquete de contenido de Power BI de Azure Active Directory](../reports-monitoring/howto-use-azure-monitor-workbooks.md)

## <a name="summary"></a>Resumen

Existen 12 aspectos en una infraestructura de identidades segura. Esta lista le permitirá administrar y asegurar las credenciales, definir la experiencia de autenticación, delegar la asignación, medir el uso y definir las directivas de acceso basadas en la postura de seguridad de la empresa.

- Asignar propietarios a las tareas principales.
- Implementar soluciones para detectar contraseñas débiles o filtradas, mejore la protección y la administración de contraseñas y proteja aún más el acceso de los usuarios a los recursos.
- Administrar la identidad de los dispositivos para proteger los recursos en cualquier momento y desde cualquier ubicación.
- Implementar la autenticación con contraseña.
- Proporcionar un mecanismo de inicio de sesión único estandarizado en toda la organización.
- Migrar aplicaciones desde AD FS a Azure AD para obtener una seguridad mejor y una capacidad de administración más coherente.
- Asignar usuarios a las aplicaciones mediante el uso de grupos para permitir una mayor flexibilidad y capacidad administrativa a escala.
- Configurar directivas de acceso basadas en riesgos.
- Bloquear los protocolos de autenticación heredados.
- Detectar y corrija las opciones ilícitas para otorgar consentimiento.
- Bloquear la configuración de usuario y de grupo.
- Habilitar el almacenamiento a largo plazo de registros de Azure AD para la solución de problemas, el análisis de uso y las investigaciones de análisis forense.

## <a name="next-steps"></a>Pasos siguientes

Comience a trabajar con las [comprobaciones y las acciones operativas de gobernanza de identidad](active-directory-ops-guide-govern.md).