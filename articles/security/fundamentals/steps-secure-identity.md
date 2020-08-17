---
title: Proteja su infraestructura de identidad de Azure AD
titleSuffix: Azure Active Directory
description: En este documento se proporciona una lista de acciones importantes que los administradores deberían implementar para proteger su organización mediante las funcionalidades de Azure AD.
author: martincoetzer
manager: manmeetb
tags: azuread
ms.service: security
ms.subservice: security-fundamentals
ms.topic: conceptual
ms.workload: identity
ms.date: 01/29/2020
ms.author: martinco
ms.openlocfilehash: d05670d55d8a4f9ad7ab691da3f05f1c3e0cd177
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87386153"
---
# <a name="five-steps-to-securing-your-identity-infrastructure"></a>Cinco pasos para asegurar su infraestructura de identidad

Si está leyendo este documento, es consciente de la importancia de la seguridad. Es probable que ya tenga la responsabilidad de proteger su organización. Si necesita convencer a otros de la importancia de la seguridad, hágales leer el último [informe de inteligencia de seguridad de Microsoft](https://go.microsoft.com/fwlink/p/?linkid=2073747).

Este documento le ayudará a conseguir una posición más segura gracias a las funcionalidades de Azure Active Directory; para ello, deberá usar una lista de comprobación de cinco pasos para proteger su organización contra los ciberataques.

Esta lista de comprobación le ayudará a implementar rápidamente las acciones críticas recomendadas para proteger su organización de inmediato; en ella se explica cómo:

* Reforzar las credenciales.
* Reducir el área de la superficie de ataque.
* Automatizar la respuesta frente a amenazas.
* Usar Cloud Intelligence.
* Habilitar el autoservicio del usuario final.

Asegúrese de realizar un seguimiento de las características y los pasos que se han completado al leer esta lista de comprobación.

> [!NOTE]
> Muchas de las recomendaciones de este documento solo son válidas para aquellas aplicaciones que estén configuradas para usar Azure Active Directory como proveedor de identidades. La configuración de aplicaciones para el inicio de sesión único garantiza las ventajas que ofrecen las políticas de credenciales, la detección de amenazas, la auditoría, el registro y otras características que se agregan a esas aplicaciones. [Administración de aplicaciones de Azure AD](../../active-directory/manage-apps/what-is-application-management.md) es la base sobre la que se establecen todas estas recomendaciones.

Las recomendaciones de este documento se basan en las especificaciones de la [puntuación de seguridad de identidades](../../active-directory/fundamentals/identity-secure-score.md), una valoración automatizada de la configuración de seguridad de las identidades de su inquilino de Azure AD. Las organizaciones pueden usar la página Puntuación de seguridad de identidad del portal de Azure AD para detectar deficiencias en la configuración de seguridad actual y garantizar que se siguen las [prácticas recomendadas](identity-management-best-practices.md) por Microsoft para la seguridad. Si se implementan todas las recomendaciones de la página Puntuación segura, la puntuación aumentará y se podrá hacer un seguimiento del progreso. Asimismo, esto ayudará a comparar la implementación con la de otras organizaciones de tamaño similar o con el conjunto del sector.

![Puntuación segura de identidad](./media/steps-secure-identity/azure-ad-sec-steps0.png)

> [!NOTE]
> Muchas de las características que se describen aquí requieren una suscripción de Azure AD Premium, mientras que otras son gratuitas. Revise nuestras secciones [Precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/) y [Lista de comprobación de implementación de Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-deployment-checklist-p2) para más información.

## <a name="before-you-begin-protect-privileged-accounts-with-mfa"></a>Antes de empezar: proteja sus cuentas con privilegios con MFA

Antes de comenzar a trabajar con esta lista de comprobación, asegúrese de que no se encontrará con problemas mientras lo hace. En primer lugar, necesita proteger sus cuentas con privilegios.

Los atacantes que tomen el control de cuentas con privilegios pueden producir enormes daños, por lo que es fundamental proteger estas cuentas primero. Habilite y solicite que todos los administradores de la organización usen el servicio [Azure Multi-Factor Authentication](../../active-directory/authentication/multi-factor-authentication.md) (MFA) mediante la [Valores predeterminados de Azure AD](../../active-directory/fundamentals/concept-fundamentals-security-defaults.md) o [Acceso condicional](../../active-directory/conditional-access/plan-conditional-access.md). Si no ha implementado MFA, ¡hágalo ahora! Es importantísimo.

¿Todo listo? Comencemos con la lista de comprobación.

## <a name="step-1---strengthen-your-credentials"></a>Paso 1: Refuerce sus credenciales

La mayoría de infracciones de seguridad que sufren las empresas se originan con una cuenta que se ha visto comprometida con uno de los diversos métodos existentes, como por ejemplo, los ataques de difusión de contraseña, la reproducción de infracciones de seguridad o la suplantación de identidad (phishing). Obtenga más información acerca de este tipo de ataques en este vídeo (45 minutos):
> [!VIDEO https://www.youtube.com/embed/uy0j1_t5Hd4]

### <a name="make-sure-your-organization-uses-strong-authentication"></a>Asegúrese de que la organización use un método de autenticación sólida

Dada la frecuencia de las contraseñas que se adivinan, se suplantan, se roban con malware o se reutilizan, es fundamental fortalecer la contraseña con algún tipo de credencial sólida: obtenga más información sobre [Azure Multi-Factor Authentication](../../active-directory/authentication/multi-factor-authentication.md).

Para habilitar con facilidad el nivel básico de seguridad de identidad, puede usar la habilitación de un solo clic con [Valores predeterminados de seguridad de Azure AD](../../active-directory/fundamentals/concept-fundamentals-security-defaults.md). Los valores predeterminados de seguridad imponen Azure MFA para todos los usuarios de un inquilino y bloquean los inicios de sesión de los protocolos heredados en todo el inquilino.

### <a name="start-banning-commonly-attacked-passwords-and-turn-off-traditional-complexity-and-expiration-rules"></a>Comience a prohibir las contraseñas que sufren ataques con más frecuencia y olvídese de la complejidad y las reglas de expiración tradicionales.

Muchas organizaciones usan la complejidad tradicional (por ejemplo, el uso de caracteres especiales, números, mayúsculas y minúsculas), así como reglas de expiración de contraseñas. La [investigación de Microsoft](https://aka.ms/passwordguidance) ha demostrado que estas políticas hacen que los usuarios elijan contraseñas que son más fáciles de adivinar.

La característica de [contraseña dinámica prohibida](../../active-directory/authentication/concept-password-ban-bad.md) de Azure AD utiliza el comportamiento actual del atacante para evitar que los usuarios establezcan contraseñas que puedan adivinarse fácilmente. Esta funcionalidad está siempre activada cuando se crean usuarios en la nube, pero ahora también está disponible en las organizaciones híbridas cuando implementan la [protección con contraseña de Azure AD para Windows Server Active Directory](../../active-directory/authentication/concept-password-ban-bad-on-premises.md). La protección con contraseña de Azure AD impide que los usuarios puedan elegir estas contraseñas comunes y se puede ampliar para que bloquee también las contraseñas que contengan las palabras clave personalizadas que se especifiquen. Por ejemplo, puede impedir que los usuarios elijan contraseñas que contengan nombres de productos de la compañía o el nombre de un equipo deportivo local.

Microsoft recomienda adoptar la directiva moderna de contraseñas basada en las [directrices del NIST](https://pages.nist.gov/800-63-3/sp800-63b.html) que se describe a continuación:

1. Solicitar contraseñas que tengan al menos 8 caracteres. Que sean más largas no quiere decir que sean mejores, ya que hacen que los usuarios elijan contraseñas predecibles, guarden las contraseñas en archivos o las acaben apuntando.
2. Deshabilitar las reglas de expiración, que solo consiguen que los usuarios elijan contraseñas fáciles de adivinar, como **Spring2019!**
3. Deshabilitar los requisitos de composición de caracteres y evitar que los usuarios elijan contraseñas comúnmente atacadas, ya que solo se consigue que los usuarios elijan sustituciones de caracteres predecibles en las contraseñas.

Si crea identidades directamente en Azure AD, puede usar [PowerShell para evitar que las contraseñas de los usuarios expiren](../../active-directory/authentication/concept-sspr-policy.md). Las organizaciones híbridas deben implementar estas directivas utilizando la [configuración de la directiva de grupo de dominio](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/hh994572(v%3dws.10)) o [Windows PowerShell](https://docs.microsoft.com/powershell/module/addsadministration/set-addefaultdomainpasswordpolicy).

### <a name="protect-against-leaked-credentials-and-add-resilience-against-outages"></a>Protéjase contra la filtración de credenciales y mejore la resistencia contra las interrupciones

Si la organización utiliza una solución de identidad híbrida con federación o autenticación de paso a través, debe habilitar la sincronización de hash de contraseñas por estos dos motivos:

* El informe [Usuarios con credenciales filtradas](../../active-directory/reports-monitoring/concept-risk-events.md) de la administración de Azure AD le advierte sobre los pares de nombre de usuario y contraseña que se expusieron en la "web oscura". Se filtra un volumen increíble de contraseñas a través del phishing, el malware y la reutilización de contraseñas en sitios de terceros que luego se vulneran. Cuando Microsoft encuentre estas credenciales filtradas, le indicará en este informe si coinciden con las credenciales de la organización; sin embargo, para ello, debe [habilitar la sincronización del hash de contraseñas](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md).
* Si se produce una interrupción en el entorno local (por ejemplo, si sufre un ataque de ransomware), podrá cambiar a la [autenticación en la nube mediante la sincronización del hash de contraseñas](choose-ad-authn.md). Este método de autenticación de copias de seguridad le permitirá continuar accediendo a las aplicaciones configuradas, gracias a la autenticación con Azure Active Directory, incluyendo Office 365. En este caso, el personal de TI no necesitará recurrir a las cuentas de correo electrónico personales para compartir los datos hasta que la interrupción del entorno local se resuelva.

Obtenga más información sobre el funcionamiento de la [sincronización del hash de contraseñas](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md).

> [!NOTE]
> Si habilita la sincronización de hash de contraseñas y usa Azure AD Domain Services, los hash de Kerberos (AES 256) y, opcionalmente, de NTLM (RC4, sin valor "salt") se cifrarán y sincronizarán con Azure AD.

### <a name="implement-ad-fs-extranet-smart-lockout"></a>Implementación del bloqueo inteligente de la extranet de AD FS

Las organizaciones que configuran aplicaciones para que se autentiquen directamente en Azure AD se benefician del [bloqueo inteligente de Azure AD ](../../active-directory/authentication/concept-sspr-howitworks.md). Si usa AD FS en Windows Server 2012 R2, implemente la [protección de bloqueo de extranet](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-soft-lockout-protection) de AD FS. Si usa AD FS en Windows Server 2016, implemente el [bloqueo inteligente de extranet](https://support.microsoft.com/help/4096478/extranet-smart-lockout-feature-in-windows-server-2016). El bloqueo inteligente de extranet de AD FS le protege contra ataques de fuerza bruta que se dirigen a AD FS, a la vez que evitan que los usuarios sean bloqueados en Active Directory.

### <a name="take-advantage-of-intrinsically-secure-easier-to-use-credentials"></a>Aproveche las credenciales intrínsecamente seguras y fáciles de usar

Gracias a [Windows Hello](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-identity-verification), puede reemplazar las contraseñas mediante una  autenticación sólida en dos fases en su PC y en dispositivos móviles. Este método de autenticación consiste en un nuevo tipo de credencial de usuario que está asociado de forma segura a un dispositivo y utiliza datos biométricos o un PIN.

## <a name="step-2---reduce-your-attack-surface"></a>Paso 2: reduzca su superficie de ataque

Dado el amplio número de contraseñas que se ven en riesgo, es fundamental minimizar la superficie de ataque en su organización. Eliminar el uso de los protocolos más antiguos y menos seguros, limitar los puntos de entrada de acceso y ejercer un mayor control del acceso administrativo a los recursos puede ayudarle a reducir el área de superficie de ataque.

### <a name="block-legacy-authentication"></a>Bloquear la autenticación heredada

Las aplicaciones que usan sus propios métodos heredados para autenticarse con Azure AD y acceder a los datos de la empresa, representan otro riesgo para las organizaciones. Varios ejemplos de aplicaciones que usan autenticación heredada son clientes POP3, IMAP4 o SMTP. Las aplicaciones de autenticación heredada se autentican en nombre del usuario e impiden que Azure AD realice evaluaciones de seguridad avanzadas. Por ello, la autenticación alternativa y moderna reducirá los riesgos de seguridad, ya que admite la autenticación multifactor y el acceso condicional. Le recomendamos que realice las siguientes tres acciones:

1. Bloquee la [autenticación heredada si usa AD FS](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/access-control-policies-w2k12).
2. Configure [SharePoint Online y Exchange Online para que usen una autenticación moderna](../../active-directory/conditional-access/conditional-access-for-exo-and-spo.md).
3. Si tiene Azure AD Premium, use directivas de acceso condicional para [bloquear la autenticación heredada](../../active-directory/conditional-access/howto-conditional-access-policy-block-legacy.md). De lo contrario, use [valores predeterminados de seguridad de Azure AD](../../active-directory/fundamentals/concept-fundamentals-security-defaults.md).

### <a name="block-invalid-authentication-entry-points"></a>Bloquear los puntos de entrada de autenticación no válidos

Al usar los recursos de vulneración de seguridad, debe reducir el impacto de las credenciales de usuario comprometidas cuando se produce un ataque. Teniendo en cuenta cada aplicación del entorno, considere los casos de uso válidos: qué grupos, qué redes, qué dispositivos y qué más elementos están autorizados y, a continuación, bloquee el resto. Mediante el [acceso condicional de Azure AD](../../active-directory/conditional-access/overview.md), puede controlar la manera en que los usuarios autorizados acceden a aplicaciones y recursos, en función de las condiciones específicas que defina.

### <a name="restrict-user-consent-operations"></a>Restricción de las operaciones de consentimiento del usuario

Es importante comprender las distintas [experiencias de consentimiento de la aplicación de Azure AD](../../active-directory/develop/application-consent-experience.md), los [tipos de permisos y consentimiento](../../active-directory/develop/v2-permissions-and-consent.md), y sus implicaciones en la postura de seguridad de la organización. De forma predeterminada, todos los usuarios de Azure AD pueden conceder acceso a los datos de su organización a las aplicaciones que utilizan la Plataforma de identidad de Microsoft. Aunque permitir que los usuarios den su consentimiento por su cuenta les permite adquirir fácilmente aplicaciones útiles que se integran con Microsoft 365, Azure y otros servicios, puede representar un riesgo si no se utiliza y supervisa con precaución.

Microsoft recomienda restringir el consentimiento del usuario para ayudar a reducir el área expuesta y a mitigar este riesgo. También puede usar [directivas de consentimiento de la aplicación (versión preliminar)](../../active-directory/manage-apps/configure-user-consent.md) para restringir el consentimiento del usuario final solo a los publicadores verificados y solo para los permisos que seleccione. Si el consentimiento del usuario final está restringido, se seguirán respetando las concesiones de consentimiento anteriores, pero todas las operaciones de consentimiento futuras tendrá que realizarlas un administrador. En casos restringidos, los usuarios pueden solicitar el consentimiento del administrador a través de un [flujo de trabajo de solicitud de consentimiento del administrador](../../active-directory/manage-apps/configure-admin-consent-workflow.md) integrado o a través de sus propios procesos de soporte técnico. Antes de restringir el consentimiento del usuario final, use nuestras [recomendaciones](../../active-directory/manage-apps/manage-consent-requests.md) para planear este cambio en su organización. En el caso de las aplicaciones a las que desea permitir el acceso a todos los usuarios, considere la posibilidad de [conceder consentimiento en nombre de todos los usuarios](../../active-directory/develop/v2-admin-consent.md) y asegúrese de que los usuarios que todavía no hayan dado su consentimiento de forma individual podrán acceder a la aplicación. Si no quiere que estas aplicaciones estén disponibles para todos los usuarios en todos los escenarios, use la [asignación de aplicación](../../active-directory/manage-apps/assign-user-or-group-access-portal.md) y el acceso condicional para restringir el acceso de los usuarios a [aplicaciones específicas](../../active-directory/conditional-access/concept-conditional-access-cloud-apps.md).

Asegúrese de que los usuarios pueden solicitar la aprobación del administrador para las nuevas aplicaciones a fin de reducir la fricción del usuario, minimizar el volumen de soporte técnico y evitar que los usuarios se registren en las aplicaciones con credenciales que no sean de Azure AD. Una vez que haya regulado las operaciones de consentimiento, los administradores deben auditar los permisos de la aplicación y consentidos con regularidad.


### <a name="implement-azure-ad-privileged-identity-management"></a>Implementar Azure AD Privileged Identity Management

Otro efecto de la "suposición de vulneración de seguridad", es la necesidad de minimizar la probabilidad de que una cuenta en riesgo pueda funcionar con un rol con privilegios.

[Azure AD Privileged Identity Management (PIM)](../../active-directory/privileged-identity-management/pim-configure.md) le permitirá minimizar los privilegios de la cuenta gracias a que podrá:

* Identificar y administrar los usuarios asignados a roles administrativos.
* Detectar los roles de privilegios excesivos o que no se usan, y que debe eliminar.
* Establecer reglas para asegurarse de que los roles con privilegios estén protegidos mediante la autenticación multifactor.
* Establecer reglas para asegurarse de que los roles con privilegios se otorguen solo el tiempo suficiente para realizar la tarea con privilegios.

Habilite Azure AD PIM, consulte a qué usuarios se les asignaron roles administrativos y elimine las cuentas innecesarias de esos roles. En cuanto a los usuarios con privilegios restantes, cambie su estado de "permanentes" a "válidos". Finalmente, establezca las directivas apropiadas para asegurarse de que, cuando necesiten obtener acceso a esos roles con privilegios, puedan hacerlo de forma segura con el control de cambios necesario.

Como parte del proceso de implementación de la cuenta con privilegios, siga los [procedimientos recomendados para crear al menos dos cuentas de emergencia](../../active-directory/users-groups-roles/directory-admin-roles-secure.md) y así asegurarse de que aún tiene acceso a Azure AD si se bloquea su cuenta.

## <a name="step-3---automate-threat-response"></a>Paso 3: automatizar la respuesta a amenazas

Azure Active Directory tiene varias funcionalidades que interceptan ataques automáticamente, y que le permiten eliminar la latencia entre la detección y la respuesta. Puede reducir los costos y riesgos a la vez que reduce el tiempo que usan los delincuentes para integrarse en su entorno. Estos son los pasos concretos que puede tomar.

### <a name="implement-user-risk-security-policy-using-azure-ad-identity-protection"></a>Implementar la directiva de seguridad de riesgo de usuario con Azure AD Identity Protection

El riesgo de usuario indica la probabilidad de que la identidad de un usuario se haya visto comprometida, y se calcula en función de las [detecciones de riesgo de usuario](../../active-directory/identity-protection/overview.md) que están asociadas con la identidad del usuario. Una directiva de riesgo de usuario es una directiva de acceso condicional que evalúa el nivel de riesgo para un usuario o grupo de usuarios específico. Según el nivel de riesgo (bajo, medio o alto), se puede configurar la directiva para que bloquee el acceso o solicite un cambio de contraseña segura mediante la autenticación multifactor. Microsoft le recomienda exigir un cambio de contraseña seguro a aquellos usuarios de alto riesgo.

![Usuarios marcados con riesgo](./media/steps-secure-identity/azure-ad-sec-steps1.png)

### <a name="implement-sign-in-risk-policy-using-azure-ad-identity-protection"></a>Implementar la directiva de riesgo de inicio de sesión con Azure AD Identity Protection

El riesgo de inicio de sesión es la probabilidad de que alguien que no sea el propietario de la cuenta intente iniciar sesión mediante su identidad. Una [directiva de riesgo de inicio de sesión](../../active-directory/identity-protection/overview.md) es una directiva de acceso condicional que evalúa el nivel de riesgo para un usuario o grupo de usuarios específico. En función del nivel de riesgo (alto / medio / bajo), se puede configurar una política para bloquear el acceso o forzar la autenticación multifactor. Asegúrese de forzar la autenticación multifactor en inicios de sesión de riesgo medio o superior.

![Iniciar sesión desde direcciones IP anónimas](./media/steps-secure-identity/azure-ad-sec-steps2.png)

## <a name="step-4---utilize-cloud-intelligence"></a>Paso 4: Uso de Cloud Intelligence

La auditoría, el registro de eventos y las alertas relacionados con la seguridad son componentes importantes en una estrategia de protección de datos eficaz. Los registros e informes de seguridad proporcionan un registro electrónico de actividades sospechosas y le ayudan a detectar patrones que puedan indicar un acceso externo a la red, así como ataques internos. Puede usar la auditoría para supervisar la actividad del usuario y el cumplimiento normativo de documentos, realizar análisis forenses y mucho más. Además, las alertas le proporcionarán notificaciones acerca de eventos de seguridad.

### <a name="monitor-azure-ad"></a>Supervisar Azure AD

Las características y los servicios de Microsoft Azure proporcionan opciones de seguridad de registro y auditoría que le ayudarán a identificar carencias en las directivas y mecanismos de seguridad, y a resolver esas carencias para evitar infracciones. Puede usar el [registro y auditoría de Azure](log-audit.md) y usar los [informes de actividades de la auditoría en el portal de Azure Active Directory](../../active-directory/reports-monitoring/concept-audit-logs.md).

### <a name="monitor-azure-ad-connect-health-in-hybrid-environments"></a>Supervisar Azure AD Connect Health en entornos híbridos

En [Supervisión de AD FS mediante Azure AD Connect Health](../../active-directory/hybrid/how-to-connect-health-adfs.md), obtendrá más información sobre los posibles problemas y la visibilidad de los ataques en la infraestructura de AD FS. Azure AD Connect Health proporciona alertas con detalles, pasos de resolución y enlaces a documentación relacionada y, además, analiza el uso de varias métricas relacionadas con el tráfico de autenticación y supervisa y crea informes de rendimiento.

![Azure AD Connect Health](./media/steps-secure-identity/azure-ad-sec-steps4.png)

### <a name="monitor-azure-ad-identity-protection-events"></a>Supervisar eventos de Azure AD Identity Protection

[Azure AD Identity Protection](../../active-directory/identity-protection/overview.md) es una herramienta de supervisión, notificación y generación de informes que se puede usar para detectar posibles vulnerabilidades que afecten a las identidades de una organización. Descubre detecciones de riesgo, como credenciales filtradas, viajes imposibles e inicios de sesión desde dispositivos infectados, direcciones IP anónimas, direcciones IP asociadas con la actividad sospechosa y ubicaciones desconocidas. Habilite las alertas de notificación para recibir correos electrónicos de usuarios en riesgo o un correo electrónico semanal de resumen.

Azure AD Identity Protection dispone de dos informes importantes que debe supervisar a diario:
1. Los informes de inicios de sesión de riesgo ponen de manifiesto las actividades de inicio de sesión de los usuarios que deben investigarse, ya que es posible que el inicio de sesión no lo haya realizado el propietario legítimo.
2. Los informes de riesgo de usuarios ponen de manifiesto las cuentas de usuario que podrían haberse visto comprometidas; por ejemplo, las cuentas en las que se ha detectado una filtración de credenciales o donde el usuario ha iniciado sesión desde una ubicación diferente, lo que indicaría que se ha producido un viaje que no es posible.

![Usuarios marcados con riesgo](./media/steps-secure-identity/azure-ad-sec-steps3.png)

### <a name="audit-apps-and-consented-permissions"></a>Aplicaciones de auditoría y permisos consentidos

Es posible que los usuarios sean víctimas de un engaño y sean conducidos a aplicaciones o sitios web comprometidos que podrían obtener acceso a la información de perfil y los datos del usuario, como el correo electrónico. Un individuo malintencionado podría usar los permisos consentidos que ha recibido para cifrar el contenido del buzón y pedir un rescate para recuperar los datos. [Los administradores deben revisar y auditar](https://docs.microsoft.com/office365/securitycompliance/detect-and-remediate-illicit-consent-grants) los permisos proporcionados por los usuarios o deshabilitar la capacidad de los usuarios de dar su consentimiento de forma predeterminada.

Además de auditar los permisos proporcionados por los usuarios, puede [encontrar aplicaciones de OAuth peligrosas o no deseadas](https://docs.microsoft.com/cloud-app-security/investigate-risky-oauth) en entornos Premium.

## <a name="step-5---enable-end-user-self-service"></a>Paso 5: habilitar la autoayuda del usuario final

En la medida de lo posible, querrá equilibrar la seguridad con la productividad. Teniendo en cuenta que está preparando el camino para conseguir asentar las bases de un sistema de seguridad a largo plazo, puede limar las asperezas en su organización al dar más autoridad a sus usuarios a la vez que se mantiene en guardia.

### <a name="implement-self-service-password-reset"></a>Implementar el restablecimiento de contraseña de autoservicio

El [autoservicio de restablecimiento de contraseña (SSPR)](../../active-directory/authentication/quickstart-sspr.md) de Azure AD ofrece a los administradores de TI un medio simple para permitir a los usuarios restablecer o desbloquear sus contraseñas o cuentas sin la intervención del departamento de soporte técnico o del administrador. El sistema incluye informes detallados del seguimiento de los usuarios que restablecen sus contraseñas, además de notificaciones de alerta de posibles abusos o usos indebidos.

### <a name="implement-self-service-group-and-application-access"></a>Implementar el acceso a grupos y aplicaciones de autoservicio

Azure AD ofrece la posibilidad de que los usuarios que no son administradores administren el acceso a los recursos mediante grupos de seguridad, grupos de Office 365, roles de aplicación y catálogos de paquetes de acceso.  [La administración de grupos de autoservicio](../../active-directory/users-groups-roles/groups-self-service-management.md) permite a los propietarios de grupos administrar sus propios grupos, sin necesidad de tener asignado un rol como administrador. Los usuarios también pueden crear y administrar grupos de Office 365 sin depender de los administradores para administrar sus solicitudes y los grupos no usados expiran automáticamente.  [La administración de derechos de Azure AD](../../active-directory/governance/entitlement-management-overview.md) habilita la delegación y visibilidad, con flujos de trabajo de solicitud de acceso completos y la expiración automática.  Puede delegar a los usuarios que no son administradores la capacidad de configurar sus propios paquetes de acceso para grupos, equipos, aplicaciones y sitios de SharePoint Online de su propiedad, con directivas personalizadas para quien se necesita aprobar el acceso, incluido el establecimiento, como aprobadores, de los administradores de los empleados y los patrocinadores socios comerciales.

### <a name="implement-azure-ad-access-reviews"></a>Implementar revisiones de acceso de Azure AD

Gracias a las [revisiones de acceso de Azure AD](../../active-directory/governance/access-reviews-overview.md), puede administrar la pertenencia a paquete de acceso y grupos, obtener acceso a las aplicaciones de la empresa y asignar funciones con privilegios para asegurarse de mantener un estándar de seguridad.  La supervisión habitual por parte de los usuarios, los propietarios de recursos y otros revisores garantiza que los usuarios no conservan el acceso durante períodos de tiempo prolongados cuando ya no lo necesitan.

## <a name="summary"></a>Resumen

Existen muchos aspectos en una infraestructura de identidad segura, pero esta lista de comprobación de cinco pasos le ayudará a lograr rápidamente una infraestructura de identidad segura:

* Reforzar las credenciales.
* Reducir el área de la superficie de ataque.
* Automatizar la respuesta frente a amenazas.
* Usar Cloud Intelligence.
* Habilitar un método de seguridad para el usuario final que sea más predecible y completo, y que cuente con autoayuda.

Le agradecemos todo el esfuerzo que ha dedicado en mejorar la seguridad de la identidad, y esperamos que este documento sea una hoja de ruta útil para conseguir mejorar la seguridad de su organización.

## <a name="next-steps"></a>Pasos siguientes

Si necesita ayuda para planificar e implementar las recomendaciones, consulte los [planes de implementación del proyecto de Azure AD](https://aka.ms/deploymentplans) para obtener ayuda.

Si está seguro de que se han completado todos estos pasos, use la [puntuación de seguridad de la identidad](../../active-directory/fundamentals/identity-secure-score.md) de Microsoft, que le permitirá mantenerse al día con los [últimos procedimientos recomendados](identity-management-best-practices.md) y las amenazas de seguridad.
