---
title: Procedimientos de acceso seguro para administradores en Azure AD | Microsoft Docs
description: Asegúrese de que el acceso administrativo y las cuentas de administrador de la organización sean seguras. Para arquitectos de sistemas y profesionales de TI que configuran Azure AD, Azure y Microsoft Online Services.
services: active-directory
keywords: ''
author: curtand
manager: daveba
ms.author: curtand
ms.date: 04/30/2020
ms.topic: conceptual
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.custom: it-pro
ms.reviewer: martincoetzer; MarkMorow
ms.collection: M365-identity-device-management
ms.openlocfilehash: dd189db10b599c0bc6bd5a3dbae2b1bc21b53b0c
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88795922"
---
# <a name="securing-privileged-access-for-hybrid-and-cloud-deployments-in-azure-ad"></a>Protección del acceso con privilegios para las implementaciones híbridas y en la nube en Azure AD

La seguridad de los recursos empresariales depende de la integridad de las cuentas con privilegios que administran los sistemas de TI. Los atacantes cibernéticos usan ataques de robo de credenciales dirigidos a las cuentas de administrador y otros accesos con privilegios para intentar acceder a los datos confidenciales.

En el caso de los servicios en la nube, la prevención y la respuesta son responsabilidades conjuntas del proveedor de los servicios en la nube y del cliente. Para más información acerca de las amenazas más recientes a los puntos de conexión y a la nube, consulte el [informe de inteligencia de seguridad de Microsoft](https://www.microsoft.com/security/operations/security-intelligence-report). Este artículo puede ayudarlo a desarrollar un mapa de ruta que cubra el espacio existente entre sus planes actuales y las directrices que se describen aquí.

> [!NOTE]
> Microsoft se compromete a ofrecer los mayores niveles de confianza, transparencia, así como de cumplimiento de estándares y reglamentaciones. Aprenda no solo la forma en que el equipo de respuesta a incidentes globales de Microsoft mitiga los efectos de los ataques contra los servicios en la nube, sino también la forma en que se incorpora la seguridad en los productos empresariales y servicios en la nube de Microsoft en [Microsoft Trust Center - Seguridad](https://www.microsoft.com/trustcenter/security)y destinos de cumplimiento de Microsoft en [Microsoft Trust Center - Cumplimiento normativo](https://www.microsoft.com/trustcenter/compliance).

Tradicionalmente, la seguridad de la organización se centraba en los puntos de entrada y salida de una red, como el perímetro de seguridad. Sin embargo, las aplicaciones SaaS y los dispositivos personales en Internet han hecho que este enfoque sea menos eficaz. En Azure AD, se reemplaza el perímetro de seguridad de red por la autenticación en la capa de identidad de la organización, bajo el control de los usuarios asignados a roles administrativos con privilegios. Su acceso debe estar protegido, independientemente de si el entorno es local, en la nube o híbrido.

La seguridad del acceso con privilegios requiere que se realicen cambios en:

* Los procesos, las prácticas administrativas y la administración del conocimiento.
* Los componentes técnicos, como las defensas de los hosts, las protecciones de las cuentas y la administración de identidades.

Proteja el acceso con privilegios de manera que se administre y se notifique en los servicios de Microsoft que le interesan. Si tiene cuentas administrativas locales, consulte la guía para el acceso con privilegios a entornos locales e híbridos en Active Directory en [Protección del acceso con privilegios](/windows-server/identity/securing-privileged-access/securing-privileged-access).

> [!NOTE]
> Las directrices de este artículo hacen referencia principalmente a las características de Azure Active Directory que se incluyen en los planes P1 y P2 de Azure Active Directory Premium. Azure Active Directory Premium P2 se incluye en los conjuntos EMS E5 y Microsoft 365 E5. En esta guía se da por supuesto que la organización ya ha adquirido licencias de Azure AD Premium P2 para los usuarios. Si no es así, es posible que algunas de las instrucciones no sean válidas para la organización. Además, en este artículo, el término administrador global tiene el mismo significado que "administrador de la empresa" o "administrador de inquilinos".

## <a name="develop-a-roadmap"></a>Desarrollo de una hoja de ruta

Microsoft recomienda que se desarrolle una hoja de ruta, y se siga, para proteger el acceso con privilegios frente a ciberatacantes. Dicha hoja de ruta se puede ajustar en cualquier momento para adecuarla a las funcionalidades existentes y a los requisitos concretos de cualquier organización. Cada fase de la hoja de ruta debería no solo aumentar el costo que supone a los adversarios atacar el acceso con privilegios en los recursos locales, híbridos y en la nube, sino también dificultar dicho ataque. Microsoft recomienda las siguientes cuatro fases de la hoja de ruta. Programe primero las implementaciones más rápidas y eficaces. Este artículo puede ser una guía, basada en la experiencia de Microsoft con incidentes de ataques cibernéticos e implementación de respuestas. Las escalas de tiempo indicadas para esta hoja de ruta son aproximadas.

![Fases de la hoja de ruta con plazos:](./media/directory-admin-roles-secure/roadmap-timeline.png)

* Fase 1 (24 a 48 horas): elementos críticos que es aconsejable realizar de inmediato

* Fase 2 (2 a 4 semanas): mitigar las técnicas de ataque que se usan con más frecuencia

* Fase 3 (1 a 3 meses): crear visibilidad y crear control total de la actividad de administración

* Fase 4 (seis meses y más allá): seguir creando defensas que protejan aún más la plataforma de seguridad

El marco de esta hoja de ruta está diseñado para maximizar el uso de tecnologías de Microsoft que es posible que ya haya implementado. Considere la posibilidad de vincular cualquier herramienta de seguridad de otros proveedores que ya haya implementado o que esté pensando implementar.

## <a name="stage-1-critical-items-to-do-right-now"></a>Fase 1 Elementos críticos para hacer ahora

![Fase 1, elementos críticos que se hacen primero](./media/directory-admin-roles-secure/stage-one.png)

La fase 1 de la hoja de ruta se centra en tareas críticas que se implementan de forma rápida y sencilla. Se recomienda realizar estos pocos elementos de inmediato, en las primeras 24 a 48 horas, para garantizar un nivel básico de acceso con privilegios seguro. Esta fase de la hoja de ruta del acceso con privilegios protegido incluye las siguientes acciones:

### <a name="general-preparation"></a>Preparación general

#### <a name="turn-on-azure-ad-privileged-identity-management"></a>Activación de Azure AD Privileged Identity Management

Se recomienda activar Azure AD Privileged Identity Management (PIM) en el entorno de producción de Azure AD. Tras activar PIM, recibirá mensajes de correo electrónico de notificación si se producen cambios en el rol de acceso con privilegios. Estas notificaciones proporcionan una advertencia anticipada cuando se agregan más usuarios a roles con privilegios elevados.

Azure AD Privileged Identity Management se incluye en Azure AD Premium P2 o EMS E5. Para ayudar a proteger el acceso a las aplicaciones y los recursos locales y en la nube, regístrese para obtener la [prueba gratuita de 90 días de Enterprise Mobility + Security](https://www.microsoft.com/cloud-platform/enterprise-mobility-security-trial). Azure AD Privileged Identity Management y Azure AD Identity Protection supervisan la actividad de seguridad con los informes, las auditorías y las alertas de Azure AD.

Después de activar Azure AD Privileged Identity Management:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con una cuenta que sea un administrador global de la organización de producción de Azure AD.

2. Para seleccionar la organización de Azure AD en la que desea usar Privileged Identity Management, seleccione el nombre de usuario en la esquina superior derecha de Azure Portal.

3. En el menú de Azure Portal, seleccione **Todos los servicios** y filtre la lista por **Azure AD Privileged Identity Management**.

4. Abra Privileged Identity Management en la lista **Todos los servicios** y ánclelo al panel.

Asegúrese de que a la primera persona que usa PIM en la organización se le asignan los roles de **Administrador de seguridad** y **Administrador de roles con privilegios**. Los administradores de rol con privilegios son los únicos que pueden administrar las asignaciones de roles de directorio de Azure AD de los usuarios. El asistente de seguridad de PIM le guía en la experiencia inicial de detección y asignación. Ahora puede salir del asistente sin realizar ningún cambio adicional.

#### <a name="identify-and-categorize-accounts-that-are-in-highly-privileged-roles"></a>Identificación y clasificación de las cuentas que están en roles con privilegios elevados

Después de activar Azure AD Privileged Identity Management, puede ver los usuarios con los siguientes roles de Azure AD:

* Administrador global
* Administrador de roles con privilegios
* Administrador de Exchange
* Administrador de SharePoint

Si no dispone de Azure AD Privileged Identity Management en la organización, puede usar la [API de PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0). Comience por el rol de administrador global, ya que un administrador global tiene los mismos permisos en todos los servicios en la nube a los que la organización se ha suscrito. Estos permisos se conceden independientemente de dónde se hayan asignado: en el centro de administración de Microsoft 365, en Azure Portal o en el módulo de Azure AD para Microsoft PowerShell.

Quite las cuentas que ya no sean necesarias en esos roles. A continuación, clasifique las restantes que están asignadas a roles de administrador:

* Asignadas a usuarios administrativos, pero utilizadas también para fines no administrativos (por ejemplo, correo electrónico personal).
* Asignadas a usuarios administrativos y utilizadas solo para fines administrativos.
* Compartidas entre varios usuarios
* Para los escenarios de acceso de máxima emergencia
* Para scripts automatizados
* Para usuarios externos

#### <a name="define-at-least-two-emergency-access-accounts"></a>Definición de un mínimo de dos de cuentas de acceso de emergencia

Es posible que un usuario quede bloqueado de su rol accidentalmente. Por ejemplo, si un proveedor de identidades federado del entorno local no está disponible, los usuarios no pueden iniciar sesión ni activar una cuenta de administrador existente. Puede prepararse para la falta accidental de acceso mediante el almacenamiento de dos o más cuentas de acceso de emergencia.

Las cuentas de acceso de emergencia ayudan a restringir el acceso con privilegios en una organización de Azure AD. Estas cuentas tienen privilegios elevados y no se asignan a usuarios específicos. Las cuentas de acceso de emergencia se limitan a situaciones "excepcionales" o de emergencia en las que no se pueden usar las cuentas administrativas normales. Asegúrese de que controla y reduce el uso de la cuenta de emergencia solo al momento en el que sea necesario.

Evalúe las cuentas que están asignadas al rol de administrador global o que son aptas para él. Si no ve ninguna cuenta que se use solo en la nube con el dominio \*.onmicrosoft.com (para el acceso de emergencia "excepcional"), créela. Para más información, consulte [Administración de cuentas administrativas de acceso de emergencia en Azure AD](directory-emergency-access.md).

#### <a name="turn-on-multi-factor-authentication-and-register-all-other-highly-privileged-single-user-non-federated-admin-accounts"></a>Activación de la autenticación multifactor y registro de las restantes cuentas de administrador no federadas de usuario único con privilegios elevados

Requiera Azure Multi-Factor Authentication (MFA) en el inicio de sesión para todos los usuarios asignados de forma permanente a uno o varios de los roles de administrador de Azure AD: administrador global, administrador de roles con privilegios, administrador de Exchange y administrador de SharePoint. Use la guía para habilitar [Multi-Factor Authentication (MFA) en sus cuentas de administrador](../authentication/howto-mfa-userstates.md) y asegúrese de que todos esos usuarios se hayan registrado en [https://aka.ms/mfasetup](https://aka.ms/mfasetup). Puede encontrar más información en los pasos 2 y 3 de [Protección del acceso a datos y servicios de Office 365](https://support.office.com/article/Protect-access-to-data-and-services-in-Office-365-a6ef28a4-2447-4b43-aae2-f5af6d53c68e). 

## <a name="stage-2-mitigate-frequently-used-attacks"></a>Fase 2: Mitigación de los ataques usados con más frecuencia

![Fase 2, mitigación de ataques de que se usan con frecuencia](./media/directory-admin-roles-secure/stage-two.png)

La fase 2 de la hoja de ruta se centra en mitigar las técnicas de ataque que se usan con más frecuencia para el robo y uso de credenciales y puede implementarse en aproximadamente 2 a 4 semanas. Esta fase de la hoja de ruta del acceso con privilegios protegido incluye las siguientes acciones.

### <a name="general-preparation"></a>Preparación general

#### <a name="conduct-an-inventory-of-services-owners-and-admins"></a>Realice un inventario de los servicios, propietarios y administradores

El aumento de las directivas "Bring your own device" (BYOD) y del trabajo desde casa, y el crecimiento de la conectividad inalámbrica hacen que sea fundamental supervisar quiénes se conectan a la red. Una auditoría de seguridad puede revelar los dispositivos, las aplicaciones y los programas de la red que la organización no admite y que representan un riesgo alto. Para más información, consulte [Información general sobre la administración y la supervisión de la seguridad en Azure](../../security/fundamentals/management-monitoring-overview.md). Asegúrese de que incluye las tareas siguientes en el proceso de inventario.

* Identifique a los usuarios que tengan roles administrativos y los servicios donde pueden administrarlos.
* Use Azure AD PIM para averiguar qué usuarios de la organización tienen acceso de administrador a Azure AD.
* Más allá de los roles definidos en Azure AD, Office 365 incluye un conjunto de roles de administrador que se pueden asignar a los usuarios de una organización. Cada rol de administrador se asigna a funciones empresariales comunes y proporciona a las personas de su organización permisos para realizar tareas específicas en el [Centro de administración de Microsoft 365](https://admin.microsoft.com). Use el Centro de administración de Microsoft 365 para averiguar qué usuarios de la organización tienen acceso de administrador a Office 365, lo que incluye mediante roles no administrados en Azure AD. Para más información, consulte [Acerca de los roles de administrador de Office 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d) y [Procedimientos de seguridad para Office 365](/office365/servicedescriptions/office-365-platform-service-description/office-365-securitycompliance-center).
* Realice el inventario de los servicios utilizados en la organización, como Azure, Intune o Dynamics 365.
* Asegúrese de que las cuentas que se usan con fines de administración:

  * tienen asociadas direcciones de correo electrónico del trabajo y
  * se han registrado en Azure Multi-Factor Authentication o utilizan MFA en el entorno local.
* Pida a los usuarios su justificación comercial para el acceso administrativo.
* Quite el acceso de administrador a los usuarios y servicios que no lo necesiten.

#### <a name="identify-microsoft-accounts-in-administrative-roles-that-need-to-be-switched-to-work-or-school-accounts"></a>Identifique las cuentas de Microsoft que tengan roles administrativos y que deban cambiarse a cuentas profesionales o educativas

Si los administradores globales iniciales reutilizaron sus credenciales de cuenta de Microsoft existentes cuando comenzaron a usar Azure AD, reemplace las cuentas de Microsoft por cuentas individuales basadas en la nube o sincronizadas.

#### <a name="ensure-separate-user-accounts-and-mail-forwarding-for-global-administrator-accounts"></a>Asegúrese de que haya cuentas de usuario independientes y reenvío de correo electrónico para las cuentas de administrador globales

Los atacantes cibernéticos normalmente suplantan cuentas de correo electrónico personales, un riesgo que hace que las direcciones de correo electrónico personales no sean aceptables como cuentas de administrador global. Para ayudar a separar los riesgos de Internet de los privilegios administrativos, cree cuentas dedicadas para cada usuario con privilegios administrativos.

* Asegúrese de crear cuentas independientes para los usuarios que van a realizar tareas de administrador global.
* Asegúrese de que los administradores globales no abran mensajes de correo electrónico accidentalmente ni ejecuten programas con sus cuentas de administrador.
* Asegúrese de que el correo electrónico de dichas cuentas se reenvía a un buzón de trabajo.

#### <a name="ensure-the-passwords-of-administrative-accounts-have-recently-changed"></a>Asegúrese de que las contraseñas de las cuentas administrativas han cambiado recientemente

Asegúrese de que todos los usuarios han iniciado sesión en sus cuentas administrativas y han cambiado sus contraseñas al menos una vez en los últimos 90 días. Además, compruebe que las contraseñas de las cuentas compartidas han cambiado recientemente.

#### <a name="turn-on-password-hash-synchronization"></a>Activación de la sincronización de hashes de contraseñas

Azure AD Connect sincroniza un hash del hash de la contraseña de un usuario de Active Directory local con una organización de Azure AD basada en la nube. Puede usar la sincronización de hash de contraseña como copia de seguridad si usa la federación con Active Directory Federation Services (AD FS). Esta copia de seguridad puede ser útil si los servidores locales de Active Directory o AD FS no están disponibles temporalmente.

La sincronización de hash de contraseña permite que los usuarios inicien sesión en un servicio con la misma contraseña que usan para iniciar sesión en su instancia local de Active Directory. La sincronización de hash de contraseña permite a Identity Protection detectar credenciales en peligro mediante la comparación de los hashes de contraseña con contraseñas que se sabe que están en peligro. Para más información, consulte [Implementación de la sincronización de hash de contraseñas con la sincronización de Azure AD Connect](../hybrid/how-to-connect-password-hash-synchronization.md).

#### <a name="require-multi-factor-authentication-for-users-in-privileged-roles-and-exposed-users"></a>Requisito de autenticación multifactor para los usuarios en roles con privilegios elevados y usuarios expuestos

Azure AD recomienda aplicar la autenticación multifactor (MFA) a todos los usuarios. No olvide a los usuarios que podrían tener un impacto significativo si su cuenta estuviera en peligro (por ejemplo, los responsables financieros). MFA reduce el riesgo de ataques debidos a que una contraseña se ha puesto en peligro.

Active:

* [MFA usando directivas de acceso condicional](../authentication/howto-mfa-getstarted.md) para todos los usuarios de su organización.

Si usa Windows Hello para empresas, el requisito de MFA se puede cumplir con la experiencia de inicio de sesión de Windows Hello. Para más información, consulte [Windows Hello](/windows/uwp/security/microsoft-passport).

#### <a name="configure-identity-protection"></a>Configure Identity Protection

Azure AD Identity Protection es una herramienta de supervisión y generación de informes basada en algoritmos que detecta posibles vulnerabilidades que afectan a las identidades de la organización. Puede configurar respuestas automáticas a las actividades sospechosas detectadas y realizar la acción apropiada para resolverlas. Para obtener más información, consulte [Azure Active Directory Identity Protection](../identity-protection/overview-identity-protection.md).

#### <a name="obtain-your-office-365-secure-score-if-using-office-365"></a>Obtenga su Puntuación segura de Office 365 (si usa Office 365)

Puntuación de seguridad examina la configuración y las actividades de los servicios de Office 365 que usa, y las compara con una base de referencia establecida por Microsoft. Obtendrá una puntuación que se basa en su grado de alineación con los procedimientos de seguridad. Cualquiera que tenga permisos de administrador en una suscripción de Office 365 Business Premium o Enterprise puede acceder a Puntuación de seguridad en [https://securescore.office.com](https://securescore.office.com/).

#### <a name="review-the-office-365-security-and-compliance-guidance-if-using-office-365"></a>Consulte la guía de seguridad y cumplimiento de Office 365 (si usa Office 365).

El [plan de seguridad y cumplimiento](https://support.office.com/article/Plan-for-security-and-compliance-in-Office-365-dc4f704c-6fcc-4cab-9a02-95a824e4fb57) describe el enfoque para que los clientes configuren Office 365 y para habilitar otras funcionalidades de EMS. Luego, lea los pasos 3-6 de [Protección del acceso a datos y servicios de Office 365](https://support.office.com/article/Protect-access-to-data-and-services-in-Office-365-a6ef28a4-2447-4b43-aae2-f5af6d53c68e) y la guía para [supervisar la seguridad y el cumplimiento en Office 365](https://support.office.com/article/Monitor-security-and-compliance-in-Office-365-b62f1722-fd39-44eb-8361-da61d21509b6).

#### <a name="configure-office-365-activity-monitoring-if-using-office-365"></a>Configure la supervisión de la actividad de Office 365 (si usa Office 365)

Supervise a los usuarios de la organización que usan Office 365 para identificar al personal que tiene una cuenta de administrador, pero que podría no necesitar acceso a Office 365 porque no inician sesión en esos portales. Para más información, consulte [Informes de actividades en el Centro de administración de Microsoft 365](https://support.office.com/article/Activity-Reports-in-the-Office-365-admin-center-0d6dfb17-8582-4172-a9a9-aed798150263).

#### <a name="establish-incidentemergency-response-plan-owners"></a>Establezca los propietarios de plan de respuesta ante incidentes o emergencias

El establecimiento de una funcionalidad adecuada de respuesta ante incidentes requiere una planeación y unos recursos considerables. Debe supervisar continuamente en busca de ataques cibernéticos y establecer prioridades para el tratamiento de incidentes. Recopile, analice y genere informes de datos de incidentes para crear relaciones y establecer comunicación con otros grupos internos y propietarios del plan. Para más información, consulte [Centro de respuesta de seguridad de Microsoft](https://technet.microsoft.com/security/dn440717).

#### <a name="secure-on-premises-privileged-administrative-accounts-if-not-already-done"></a>Proteja las cuentas administrativas con privilegios locales, si no lo ha hecho

Si la organización de Azure Active Directory está sincronizada con la instancia local de Active Directory, siga las instrucciones que encontrará en [Mapa de ruta para proteger el acceso con privilegios](/windows-server/identity/securing-privileged-access/securing-privileged-access): Esta fase incluye:

* Creación de cuentas de administrador independientes para aquellos usuarios que deban llevar a cabo tareas administrativas en el entorno local
* Implementación de estaciones de trabajo con acceso con privilegios para los administradores de Active Directory
* Creación de contraseñas de administrador locales únicas para estaciones de trabajo y servidores

### <a name="additional-steps-for-organizations-managing-access-to-azure"></a>Pasos adicionales para las organizaciones que administran el acceso a Azure

#### <a name="complete-an-inventory-of-subscriptions"></a>Complete un inventario de suscripciones

Utilice Enterprise Portal y Azure Portal para identificar las suscripciones de la organización que hospedan aplicaciones de producción.

#### <a name="remove-microsoft-accounts-from-admin-roles"></a>Eliminación de cuentas de Microsoft de roles de administrador

Las cuentas Microsoft de otros programas, como Xbox, Live y Outlook, no deben usarse como cuentas de administrador para las suscripciones de la organización. Elimine el estado de administrador de todas las cuentas Microsoft y sustitúyalas por cuentas profesionales o educativas de Azure AD (por ejemplo, chris@contoso.com). Para fines administrativos, utilice cuentas que se autentican en Azure AD y no en otros servicios.

#### <a name="monitor-azure-activity"></a>Supervise la actividad de Azure

El registro de actividad de Azure proporciona un historial de los eventos de nivel de suscripción en Azure. Ofrece información acerca de quién creó, actualizó o eliminó los recursos y cuándo se produjeron estos eventos. Para más información, consulte, [Auditoría y recepción de notificaciones sobre las acciones importantes en su suscripción de Azure](../../azure-monitor/platform/alerts-activity-log.md).

### <a name="additional-steps-for-organizations-managing-access-to-other-cloud-apps-via-azure-ad"></a>Pasos adicionales para las organizaciones que administran el acceso a otras aplicaciones en la nube a través de Azure AD

#### <a name="configure-conditional-access-policies"></a>Configuración de directivas de acceso condicional

Prepare directivas de acceso condicional tanto para las aplicaciones locales como para las hospedadas en la nube. Si los usuarios han unido dispositivos al área de trabajo, obtenga más información de [Configuración del acceso condicional local mediante el registro de dispositivos de Azure Active Directory](../devices/overview.md).

## <a name="stage-3-take-control-of-admin-activity"></a>Fase 3: Toma de control de la actividad administrativa

![Fase 3: Toma de control de la actividad administrativa](./media/directory-admin-roles-secure/stage-three.png)

La fase 3 se basa en las mitigaciones de riesgos de la fase 2 y se implementará en aproximadamente 1 a 3 meses. Esta fase de la hoja de ruta del acceso con privilegios protegido incluye los siguientes componentes.

### <a name="general-preparation"></a>Preparación general

#### <a name="complete-an-access-review-of-users-in-administrator-roles"></a>Complete una revisión de acceso de los usuarios en roles de administrador

Un mayor número de usuarios corporativos obtienen acceso con privilegios mediante los servicios en la nube, lo que puede llevar a un acceso no administrado. Actualmente, los usuarios se pueden convertir en administradores globales de Office 365, administradores de suscripciones de Azure o tener acceso de administrador a las máquinas virtuales o mediante aplicaciones SaaS.

La organización debe hacer que los empleados realicen las transacciones empresariales ordinarias como usuarios sin privilegios y, después, conceder derechos de administrador solo cuando sea necesario. Realice revisiones del acceso para identificar y confirmar a aquellos usuarios que son aptos para activar privilegios administrativos.

Se recomienda que:

1. Determine qué usuarios son administradores de Azure AD, habilite el acceso de administrador Just-in-Time a petición y los controles de seguridad basados en rol.
2. Convierta los usuarios que no tengan una justificación clara para el acceso de administrador con privilegios a otro rol (si ninguna de las funciones es apta, quítelas).

#### <a name="continue-rollout-of-stronger-authentication-for-all-users"></a>Continúe con el lanzamiento de una autenticación más segura para todos los usuarios

Requiera que los usuarios con una exposición alta utilicen una autenticación sólida y moderna como Azure MFA o Windows Hello. Algunos ejemplos de usuarios con alta exposición son:

* Ejecutivos de máximo rango
* Administradores de alto nivel
* Personal crítico de TI y seguridad

#### <a name="use-dedicated-workstations-for-administration-for-azure-ad"></a>Use estaciones de trabajo dedicados para la administración de Azure AD

Los atacantes podrían intentar dirigirse a cuentas con privilegios con el fin de manipular la integridad y la autenticidad de los datos. A menudo usan código malintencionado que altera la lógica del programa o vigila si el administrador escribe una credencial. Las estaciones de trabajo con privilegios de acceso (PAW) proporcionan un sistema operativo dedicado para tareas delicadas protegido contra ataques de Internet y vectores de amenazas. La separación de estas tareas confidenciales y las cuentas de los dispositivos y las estaciones de trabajo de uso diario proporciona una protección segura frente a:

* Ataques de suplantación de identidad (phishing).
* Vulnerabilidades del sistema operativo y las aplicaciones.
* Ataques de suplantación.
* Ataques de robo de credenciales, como el registro de pulsaciones de teclas, Pass-the-Hash y Pass-the-Ticket.

Mediante la implementación de estaciones de trabajo de acceso con privilegios, se puede reducir el riesgo de que los administradores escriban las credenciales de administrador en un entorno de escritorio que no se haya protegido. Para más información, consulte [Estaciones de trabajo con privilegios de acceso](/windows-server/identity/securing-privileged-access/privileged-access-workstations).

#### <a name="review-national-institute-of-standards-and-technology-recommendations-for-handling-incidents"></a>Consulte las recomendaciones de control de incidentes que realiza el Instituto Nacional de Normas y Tecnología

El National Institute of Standards and Technology (NIST) proporciona directrices para el control de incidentes, especialmente para analizar los datos relacionados con estos y determinar la respuesta más apropiada en cada caso. Para más información, consulte [la guía para el control de incidentes de seguridad en los equipos del NIST (SP 800-61, revisión 2)](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf).

#### <a name="implement-privileged-identity-management-pim-for-jit-to-additional-administrative-roles"></a>Implemente Privileged Identity Management (PIM) para JIT en más roles administrativos

En el caso de Azure Active Directory, use la funcionalidad [Azure AD Privileged Identity Management](../privileged-identity-management/pim-configure.md). La activación por tiempo limitado de roles con privilegios funciona mediante la habilitación para:

* Activar los privilegios de administrador para llevar a cabo una tarea específica.
* Exigir MFA durante el proceso de activación
* Usar alertas para informar a los administradores acerca de los cambios fuera de banda
* Permitir a los usuarios conservar su acceso con privilegios durante un período previamente configurado.
* Permitir a los administradores de seguridad:

  * Detectar todas las identidades con privilegios.
  * Ver informes de auditoría.
  * Crear revisiones de acceso para identificar a todos aquellos usuarios que sean aptos para activar privilegios administrativos.

Si ya usa Azure AD Privileged Identity Management, ajuste los períodos de tiempo de los privilegios con tiempo limitado, según sea necesario (por ejemplo, las ventanas de mantenimiento).

#### <a name="determine-exposure-to-password-based-sign-in-protocols-if-using-exchange-online"></a>Determine la exposición a protocolos de inicio de sesión mediante contraseña (si usa Exchange Online)

Se recomienda identificar a todos los posibles usuarios con credenciales que resultaría catastrófico para la organización que se vieran en peligro. Para esos usuarios, aplique requisitos de autenticación seguros y use el acceso condicional de Azure AD para evitar que inicien sesión en su correo electrónico con el nombre de usuario y la contraseña. Puede bloquear la [autenticación heredada mediante el acceso condicional](../conditional-access/block-legacy-authentication.md) y [bloquear la autenticación básica](/exchange/clients-and-mobile-in-exchange-online/disable-basic-authentication-in-exchange-online) mediante Exchange Online.

#### <a name="complete-a-roles-review-assessment-for-office-365-roles-if-using-office-365"></a>Realice una valoración del examen de los roles de Office 365 (si usa Office 365)

Evalúe si todos los usuarios administradores están en los roles correctos (elimínelos y vuelva a asignarlos en función de esta evaluación).

#### <a name="review-the-security-incident-management-approach-used-in-office-365-and-compare-with-your-own-organization"></a>Examine el enfoque de administración de incidentes de seguridad usado en Office 365 y compárelo con su propia organización

Este informe se puede descargar de [Security Incident Management in Microsoft Office 365](https://www.microsoft.com/download/details.aspx?id=54302) (Administración de incidentes de seguridad en Microsoft Office 365).

#### <a name="continue-to-secure-on-premises-privileged-administrative-accounts"></a>Continúe con la protección de las cuentas administrativas con privilegios locales

Si Azure Active Directory está conectado con la instancia local de Active Directory, siga las instrucciones de [Hoja de ruta de la seguridad del acceso con privilegios](/windows-server/identity/securing-privileged-access/securing-privileged-access): Fase 2. En esta fase, puede:

* Implementar estaciones de trabajo con acceso con privilegios para todos los administradores.
* Requerir MFA
* Usar solo la administración necesaria para el mantenimiento del controlador de dominio, reduciendo así la superficie de ataque de los dominios.
* Implementar la evaluación de amenazas avanzada para la detección de ataques.

### <a name="additional-steps-for-organizations-managing-access-to-azure"></a>Pasos adicionales para las organizaciones que administran el acceso a Azure

#### <a name="establish-integrated-monitoring"></a>Establezca la supervisión integrada

[Azure Security Center](../../security-center/security-center-intro.md):

* Permite supervisar la seguridad y administrar las directivas integradas en las suscripciones de Azure.
* Ayuda a detectar amenazas que podrían pasar desapercibidas de otro modo.
* Funciona con una amplia gama de soluciones de seguridad

#### <a name="inventory-your-privileged-accounts-within-hosted-virtual-machines"></a>Realice un inventario de sus cuentas con privilegios en las máquinas virtuales hospedadas

Normalmente, no es necesario conceder a los usuarios permisos sin restricciones a todas las suscripciones o recursos de Azure. Use los roles de administrador de Azure AD para conceder solo el acceso que los usuarios necesitan para realizar su trabajo. Puede usar los roles de administrador de Azure AD para que un administrador pueda administrar solo las máquinas virtuales de una suscripción, mientras que otro pueda administrar las bases de datos SQL de la misma suscripción. Para más información, consulte [Introducción al control de acceso basado en roles en Azure Portal](../../role-based-access-control/overview.md).

#### <a name="implement-pim-for-azure-ad-administrator-roles"></a>Implemente PIM para los roles de administrador de Azure AD

Use Privileged Identity Management con los roles de administrador de Azure AD para administrar, controlar y supervisar el acceso a los recursos de Azure. El uso de PIM protege mediante la reducción del tiempo de exposición de los privilegios y el aumento de la visibilidad de su uso mediante alertas e informes. Para más información, consulte [Administración del acceso a los recursos de Azure con Privileged Identity Management](../../role-based-access-control/best-practices.md).

#### <a name="use-azure-log-integrations-to-send-relevant-azure-logs-to-your-siem-systems"></a>Use las integraciones de registros de Azure para enviar los registros de Azure relevantes a los sistemas de SIEM

Azure Log Integration permite integrar los registros sin procesar de los recursos de Azure en los sistemas de administración de eventos e información de seguridad (SIEM) existentes de la organización. La [integración de registros de Azure](/previous-versions/azure/security/fundamentals/azure-log-integration-overview) recopila los eventos de Windows a partir de los registros del Visor de eventos de Windows y los recursos de Azure desde:

* Registros de actividad de Azure
* Alertas de Azure Security Center
* Registros de recursos de Azure

### <a name="additional-steps-for-organizations-managing-access-to-other-cloud-apps-via-azure-ad"></a>Pasos adicionales para las organizaciones que administran el acceso a otras aplicaciones en la nube a través de Azure AD

#### <a name="implement-user-provisioning-for-connected-apps"></a>Implemente el aprovisionamiento de usuarios para las aplicaciones conectadas

Azure AD permite automatizar la creación y el mantenimiento de las identidades de usuario de aplicaciones en la nube como Dropbox, Salesforce y ServiceNow. Para más información, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

#### <a name="integrate-information-protection"></a>Integre la protección de la información

Microsoft Cloud App Security permite investigar archivos y establecer directivas en función de las etiquetas de clasificación de Azure Information Protection, lo que permite tener mayor visibilidad y control de los datos en la nube. Analice y clasifique los archivos de la nube y aplique las etiquetas de Azure Information Protection. Para más información, consulte [Integración de Azure Information Protection](/cloud-app-security/azip-integration).

#### <a name="configure-conditional-access"></a>Configuración del acceso condicional

Configure el acceso condicional en función del grupo, la ubicación y la confidencialidad de la aplicación para las [aplicaciones SaaS](https://azure.microsoft.com/overview/what-is-saas/) y las aplicaciones conectadas a Azure AD. 

#### <a name="monitor-activity-in-connected-cloud-apps"></a>Supervise la actividad en las aplicaciones en la nube conectadas

Para asegurarse de que el acceso de los usuarios también se protege en las aplicaciones conectadas, se recomienda usar [Microsoft Cloud App Security](/cloud-app-security/what-is-cloud-app-security). Esta característica protege el acceso de la empresa a las aplicaciones en la nube y protege sus cuentas de administrador, ya que le permite:

* Ampliar la visibilidad y el control a las aplicaciones en la nube
* Crear directivas de acceso, actividades y uso compartido de datos
* Identificar automáticamente las actividades de riesgo, comportamientos anómalos y amenazas
* Evitar la pérdida de datos
* Minimizar el riesgo, realizar una prevención automática de las amenazas y exigir la aplicación de directivas

El agente de SIEM de Cloud App Security integra Cloud App Security en el servidor de SIEM para habilitar la supervisión centralizada de las actividades y alertas de Office 365. Se ejecuta en el servidor y extrae las alertas y actividades de Cloud App Security y las transmite por secuencias al servidor de SIEM. Para más información, consulte [Integración de SIEM](/cloud-app-security/siem).

## <a name="stage-4-continue-building-defenses"></a>Fase 4: Continuar creando defensas

![Fase 4: Adopción de una posición de seguridad activa](./media/directory-admin-roles-secure/stage-four.png)

La fase 4 de la hoja de ruta se debe implementar a partir de los seis meses. Complete la hoja de ruta para fortalecer las protecciones de acceso con privilegios frente a los posibles ataques que se conocen hoy en día. Para las amenazas de seguridad futuras, se recomienda ver la seguridad como un proceso continuo para aumentar los costos y reducir la tasa de éxito de los adversarios que amenazan el entorno.

La protección del acceso con privilegios es importante para establecer garantías de seguridad para los recursos empresariales. Sin embargo, debe formar parte de un programa de seguridad completo que proporcione garantías de seguridad continuas. Dicho programa debe incluir elementos como:

* Directiva
* Operaciones
* Seguridad de la información
* Servidores
* APLICACIONES
* Equipos
* Dispositivos
* Tejido en la nube

Se recomiendan los siguientes procedimientos para administrar las cuentas con acceso con privilegios:

* Asegúrese de que los administradores realizan sus tareas cotidianas como usuarios sin privilegios.
* Conceda acceso con privilegios solo cuando sea necesario y revóquelo inmediatamente después (Just-In-Time).
* Conserve los registros de las actividades de auditoría relacionadas con las cuentas con privilegios.

Para más información acerca de cómo crear toda una hoja de ruta de seguridad, consulte [Recursos de arquitectura de TI de la nube de Microsoft](https://almbok.com/office365/microsoft_cloud_it_architecture_resources). Para solicitar los servicios de Microsoft para ayudarle a implementar cualquiera de las áreas de la hoja de ruta, póngase en contacto con su representante de Microsoft o consulte [Creación de ciberdefensas críticas para proteger la empresa](https://www.microsoft.com/en-us/microsoftservices/campaigns/cybersecurity-protection.aspx).

Esta fase final de la hoja de ruta del acceso con privilegios protegido incluye los siguientes componentes.

### <a name="general-preparation"></a>Preparación general

#### <a name="review-admin-roles-in-azure-ad"></a>Revisión de los roles de administrador en Azure AD

Determine si los roles de administrador de Azure AD integrados actuales siguen actualizados y asegúrese de que los usuarios solo están en los roles que necesitan. Con Azure AD puede asignar administradores independientes que desempeñen distintas funciones. Para más información, consulte [Asignación de roles de administrador en Azure Active Directory](directory-assign-admin-roles.md).

#### <a name="review-users-who-have-administration-of-azure-ad-joined-devices"></a>Examine los usuarios que tengan la administración de dispositivos unidos a Azure AD

Para más información, consulte [Configuración de dispositivos híbridos unidos a Azure Active Directory](../devices/hybrid-azuread-join-plan.md).

#### <a name="review-members-of-built-in-office-365-admin-roles"></a>Revise los miembros de los [roles de administrador de Office 365 integrados](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d)
Omita este paso si no usa Office 365.
‎
#### <a name="validate-incident-response-plan"></a>Valide el plan de respuesta ante incidentes

Para mejorar su plan, Microsoft recomienda validar periódicamente que funciona según lo esperado:

* Recorra la hoja de ruta existente para ver lo que falta
* En función del análisis post mortem, revise los procedimientos existentes o defina otros nuevos.
* Asegúrese de distribuir a toda la organización el plan de respuesta ante incidentes y los procedimientos actualizados.


### <a name="additional-steps-for-organizations-managing-access-to-azure"></a>Pasos adicionales para las organizaciones que administran el acceso a Azure 

Determine si necesita [transferir la propiedad de una suscripción de Azure a otra cuenta](../../cost-management-billing/manage/billing-subscription-transfer.md).
‎

## <a name="break-glass-what-to-do-in-an-emergency"></a>"Romper el cristal": qué hacer en caso de emergencia

![Cuentas para acceder en caso de emergencia](./media/directory-admin-roles-secure/emergency.jpeg)

1. Notifique la información acerca del incidente a los principales directivos y responsables de seguridad.

2. Revise el cuaderno de estrategias para repeler ataques.

3. Acceda a la combinación de nombre de usuario y contraseña de la cuenta de emergencia "excepcional" para iniciar sesión en Azure AD.

4. Obtenga ayuda de Microsoft, para lo que debe [abrir una solicitud de soporte técnico de Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md).

5. Examine los [informes de inicio de sesión de Azure AD](../reports-monitoring/overview-reports.md). Puede transcurrir un tiempo entre el momento en el que se produce un evento y el momento en el que se incluye en el informe.

6. En el caso de los entornos híbridos, si la infraestructura local federada y el servidor de AD FS no están disponibles, puede cambiar temporalmente de la autenticación federada al uso de la sincronización de hash de contraseña. Este cambio revierte la federación del dominio a la autenticación administrada hasta que el servidor de AD FS vuelva a estar disponible.

7. Supervisar el correo electrónico de las cuentas con privilegios.

8. Asegúrese de guardar copias de seguridad de los registros relevantes, por si es preciso realizar una investigación legal o forense.

Para más información acerca de la forma en que Microsoft Office 365 controla los incidentes de seguridad, consulte [Security Incident Management in Microsoft Office 365](https://aka.ms/Office365SIM) (Administración de incidentes de seguridad en Microsoft Office 365).

## <a name="faq-answers-for-securing-privileged-access"></a>PREGUNTAS MÁS FRECUENTES: Respuestas sobre la protección del acceso con privilegios  

**P:** ¿Qué hago si aún no he implementado ningún componente de acceso seguro?

**Respuesta:** Defina un mínimo de dos cuentas para casos de emergencia, asigne MFA a sus cuentas de administrador con privilegios y separe las cuentas de usuario de las cuentas de administrador global.

**P:** Después de una infracción, ¿qué problema es el primero que se debe solucionar?

**Respuesta:** Asegúrese de exigir la autenticación más segura posible a los usuarios con mayor exposición.

**P:** ¿Qué ocurre si se han desactivado nuestros administradores con privilegios?

**Respuesta:** Cree una cuenta de administrador global y manténgala siempre actualizada.

**P:** ¿Qué ocurre si solo queda un administrador global y no se le puede localizar?

**Respuesta:** Utilice una de las cuentas de emergencia para obtener acceso con privilegios de inmediato.

**P:** ¿Cómo puedo proteger a los administradores en mi organización?

**Respuesta:** Indique a los administradores que siempre deben realizar sus tareas cotidianas como usuarios estándar "sin privilegios".

**P:** ¿Cuáles son los procedimientos recomendados para la creación de cuentas de administrador en Azure AD?

**Respuesta:** Reserva el acceso con privilegios para tareas de administrador específicas.

**P:** ¿Qué herramientas existen para reducir el acceso de administrador constante?

**Respuesta:** Privileged Identity Management (PIM) y los roles de administrador de Azure AD.

**P:** ¿Cuál es la posición de Microsoft acerca de la sincronización de cuentas de administrador con Azure AD?

**Respuesta:** Las cuentas de administrador de nivel 0 solo se usan para las cuentas de AD local. Estas cuentas normalmente no se sincronizan con Azure AD en la nube. El nivel 0 de cuentas de administrador incluye cuentas, grupos y otros recursos que tienen control administrativo directo o indirecto de los bosques, dominios, controladores de dominio y recursos de Active Directory.

**P:** ¿Cómo se evita que los administradores asignen acceso de administrador aleatoriamente en el portal?

**Respuesta:** Use cuentas sin privilegios para todos los usuarios y la mayoría de los administradores. Para empezar, desarrolle una imagen de la organización para determinar qué cuentas de administrador deberían tener privilegios. Y supervise los usuarios administrativos recién creados.

## <a name="next-steps"></a>Pasos siguientes

* [Centro de confianza de Microsoft para la seguridad de los productos](https://www.microsoft.com/trustcenter/security): características de seguridad de los productos y servicios en la nube de Microsoft

* [Centro de confianza de Microsoft: cumplimiento](https://www.microsoft.com/trustcenter/compliance/complianceofferings): conjunto completo de ofertas de cumplimiento para los servicios en la nube de Microsoft

* [Guía para realizar una evaluación del riesgo](https://www.microsoft.com/trustcenter/guidance/risk-assessment): administre la seguridad y los requisitos de cumplimiento de los servicios en la nube de Microsoft

### <a name="other-microsoft-online-services"></a>Otros servicios de Microsoft Online Services

* [Seguridad de Microsoft Intune](https://www.microsoft.com/trustcenter/security/intune-security): Intune proporciona funcionalidades de administración de dispositivos móviles, aplicaciones móviles y PC desde la nube.

* [Seguridad de Microsoft Dynamics 365](https://www.microsoft.com/trustcenter/security/dynamics365-security): Dynamics 365 es la solución en la nube de Microsoft que unifica la administración de relaciones con clientes (CRM) y las funcionalidades de planificación de recursos empresariales (ERP).