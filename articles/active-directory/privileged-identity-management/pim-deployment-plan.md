---
title: 'Implementación de Privileged Identity Management (PIM): Azure AD | Microsoft Docs'
description: Se describe cómo planear la implementación de Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: pim
ms.date: 08/27/2020
ms.author: curtand
ms.custom: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6bfe0fee14ed463e265dc4e7e4177c702b051c81
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89050206"
---
# <a name="deploy-azure-ad-privileged-identity-management-pim"></a>Implementación de Azure AD Privileged Identity Management (PIM)

Este artículo es una guía detallada que describe cómo planear la implementación de Privileged Identity Management (PIM) en su organización de Azure Active Directory (Azure AD). Reasignará a los usuarios de roles con privilegios elevados a roles integrados o personalizados menos eficaces, siempre que sea posible, y planeará las asignaciones de roles Just-In-Time para los roles con más privilegios. En este artículo, se realizarán recomendaciones para la implementación y la planeación de la implementación.

> [!TIP]
> En este artículo, verá los elementos marcados como:
>
> :heavy_check_mark: **Microsoft recomienda**
>
> Las siguientes son recomendaciones generales que debe implementar únicamente si se aplican a sus necesidades empresariales específicas.

## <a name="licensing-requirements"></a>Requisitos de concesión de licencia

Para usar Privileged Identity Management, el directorio debe tener una de las siguientes licencias de pago o de evaluación. Para más información, consulte [Requisitos de licencia para usar Privileged Identity Management](subscription-requirements.md).

- Azure AD Premium P2
- Enterprise Mobility + Security (EMS) E5
- Microsoft 365 Education A5
- Microsoft 365 Enterprise E5

## <a name="how-pim-works"></a>Cómo funciona PIM

En esta sección se proporciona una revisión de las finalidades de planeación de las partes pertinentes del proceso de Privileged Identity Management. Para más información, vea [¿Qué es Azure AD Privileged Identity Management?](pim-configure.md)

1. Comience a usar Privileged Identity Management de forma que los usuarios sean aptos para roles con privilegios.
1. Cuando un usuario apto necesita utilizar sus roles con privilegios, activa el rol mediante Privileged Identity Management.
1. Puede que sea necesaria la intervención del usuario en la configuración para:

    - Usar la autenticación multifactor
    - Solicitar aprobación para la activación
    - Proporcionar un motivo empresarial para la activación

1. Una vez que el usuario activa el rol correctamente, tendrá los permisos de rol para una duración establecida.
1. Los administradores pueden ver un historial de todas las actividades de Privileged Identity Management en el registro de auditoría. También pueden proteger más a sus organizaciones de Azure AD y satisfacer el cumplimiento con las características de Privileged Identity Management, como las revisiones de acceso y las alertas.

## <a name="roles-that-can-be-managed-by-pim"></a>Roles que se pueden administrar con PIM

**Roles de Azure AD**: están todos en Azure Active Directory (por ejemplo, administrador global, administrador de Exchange y administrador de seguridad). Puede leer más sobre los roles y sus funcionalidades en [Permisos de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md). Para obtener ayuda con la determinación de qué roles asignar a los administradores, consulte [Roles con privilegios mínimos por tarea](../users-groups-roles/roles-delegate-by-task.md).

**Roles de Azure**: son roles que están vinculados a un recurso, grupo de recursos, suscripción o grupo de administración de Azure. Puede usar PIM para proporcionar acceso Just-In-Time a roles de Azure integrados, como propietario, administrador de acceso de usuario y colaborador, así como a [roles personalizados](../../role-based-access-control/custom-roles.md). Para más información sobre los roles de Azure, consulte [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](../../role-based-access-control/overview.md).

Para más información, consulte [Roles que no se pueden administrar en Privileged Identity Management](pim-roles.md).

## <a name="deployment-plan"></a>El plan de implementación

Antes de implementar Privileged Identity Management en la organización, siga las instrucciones y comprenda los conceptos de esta sección para ayudarle a crear un plan adaptado a los requisitos de identidad con privilegios de la organización.

### <a name="identify-your-stakeholders"></a>Identificación de las partes interesadas

La siguiente sección le ayuda a identificar a todas las partes interesadas que intervienen en el proyecto y necesitan firmar, revisar o mantenerse informados. Incluye tablas independientes para implementar PIM para roles de Azure AD y PIM para roles de Azure. Agregue partes interesadas a la siguiente tabla según proceda para su organización.

- SO = aprobar este proyecto
- R = revisar este proyecto y proporcionar entrada
- I = Informado de este proyecto

#### <a name="stakeholders-privileged-identity-management-for-azure-ad-roles"></a>Partes interesadas: Privileged Identity Management para roles de Azure AD

| Nombre | Role | Acción |
| --- | --- | --- |
| Nombre y correo electrónico | **Arquitecto de identidades o administrador global de Azure**<br/>Un representante del equipo de administración de identidades responsable de definir cómo alinear este cambio con la infraestructura de administración de identidades principal de su organización. | SO/R/I |
| Nombre y correo electrónico | **Propietario del servicio/Administrador de línea**<br/>Un representante de los propietarios de TI de un servicio o un grupo de servicios. Son fundamentales para tomar decisiones y ayudar a implementar Privileged Identity Management para su equipo. | SO/R/I |
| Nombre y correo electrónico | **Propietario de seguridad**<br/>Un representante del equipo de seguridad que puede aprobar que el plan cumple los requisitos de seguridad de la organización. | SO/R |
| Nombre y correo electrónico | **Administrador de soporte técnico de TI o departamento de soporte técnico**<br/>Un representante de la organización de soporte técnico de TI que puede proporcionar comentarios sobre la compatibilidad de este cambio desde una perspectiva del departamento de soporte técnico. | R/I |
| Nombre y correo electrónico de los usuarios piloto | **Usuarios de roles con privilegios**<br/>El grupo de usuarios para el que se implementa la administración de identidades con privilegios. Necesitan saber cómo activar sus roles una vez que se implementa Privileged Identity Management. | I |

#### <a name="stakeholders-privileged-identity-management-for-azure-roles"></a>Partes interesadas: Privileged Identity Management para roles de Azure

| Nombre | Role | Acción |
| --- | --- | --- |
| Nombre y correo electrónico | **Propietario de recurso/suscripción**<br/>Un representante de los propietarios de TI de cada suscripción o recurso para los que desea implementar Privileged Identity Management. | SO/R/I |
| Nombre y correo electrónico | **Propietario de seguridad**<br/>Un representante del equipo de seguridad que puede aprobar que el plan cumple los requisitos de seguridad de la organización. | SO/R |
| Nombre y correo electrónico | **Administrador de soporte técnico de TI o departamento de soporte técnico**<br/>Un representante de la organización de soporte técnico de TI que puede proporcionar comentarios sobre la compatibilidad de este cambio desde una perspectiva del departamento de soporte técnico. | R/I |
| Nombre y correo electrónico de los usuarios piloto | **Usuarios con rol de Azure**<br/>El grupo de usuarios para el que se implementa la administración de identidades con privilegios. Necesitan saber cómo activar sus roles una vez que se implementa Privileged Identity Management. | I |

### <a name="start-using-privileged-identity-management"></a>Empiece a usar Privileged Identity Management

Como parte del proceso de planeamiento, debe seguir nuestro artículo sobre cómo [empezar a usar Privileged Identity Management](pim-getting-started.md) a fin de preparar Privileged Identity Management. Privileged Identity Management le da acceso a algunas características diseñadas para facilitar la implementación.

Si su objetivo es implementar Privileged Identity Management para recursos de Azure, debe seguir las indicaciones del artículo [sobre la detección de recursos de Azure que se administran en Privileged Identity Management](pim-resource-roles-discover-resources.md). Solo los propietarios de suscripciones y los grupos de administración pueden administrar estos recursos mediante Privileged Identity Management. Una vez administrados, la funcionalidad de PIM está disponible para los propietarios en todos los niveles, incluidos el grupo de administración, la suscripción, el grupo de recursos y el recurso. Si es administrador global e intenta implementar Privileged Identity Management para los recursos de Azure, puede [elevar el acceso para administrar todas las suscripciones de Azure](../../role-based-access-control/elevate-access-global-admin.md?toc=%2fazure%2factive-directory%2fprivileged-identity-management%2ftoc.json) y tener acceso a todos los recursos de Azure en el directorio a fin de poder detectarlos. Sin embargo, se recomienda obtener la aprobación de cada uno de los propietarios de las suscripciones antes de administrar sus recursos con Privileged Identity Management.

### <a name="enforce-principle-of-least-privilege"></a>Aplicación del principio de privilegio mínimo

Es importante asegurarse de que ha aplicado el principio de privilegio mínimo en la organización para los roles de Azure y Azure AD.

#### <a name="plan-least-privilege-delegation"></a>Planear la delegación de privilegios mínimos

Para los roles de Azure AD, es habitual que las organizaciones asignen el rol de administrador global a varios administradores, cuando la mayoría de ellos solo necesitan uno o dos roles de administrador específicos y menos eficaces. Con un gran número de administradores globales u otros roles con privilegios elevados, es difícil realizar un seguimiento meticuloso de las asignaciones de roles con privilegios.

Siga estos pasos para implementar el principio de privilegio mínimo para los roles de Azure AD.

1. Para conocer la granularidad de los roles, lea y entienda los [roles de administrador de Azure AD disponibles](../users-groups-roles/directory-assign-admin-roles.md#available-roles). Usted y su equipo también deben consultar [Roles de administrador por tarea de administrador en Azure Active Directory](../users-groups-roles/roles-delegate-by-task.md), donde se explica el rol con privilegios mínimos para tareas específicas.

1. Especifique quién tiene roles con privilegios en su organización. Puede usar la página [Detección e información (versión preliminar)](pim-security-wizard.md) en Privileged Identity Management para reducir la exposición.

    ![Página Detección e información (versión preliminar) para reducir la exposición mediante roles con privilegios](./media/pim-deployment-plan/new-preview-page.png)

1. Analice por qué todos los administradores globales de la organización necesitan el rol. Después, quítelos del rol de administrador global y asigne roles integrados o roles personalizados con menos privilegios dentro de Azure Active Directory. A título informativo, actualmente, Microsoft solamente tiene unos diez administradores con el rol de administrador global. Obtenga más información sobre [cómo usa Microsoft la característica Privileged Identity Management](https://www.microsoft.com/itshowcase/Article/Content/887/Using-Azure-AD-Privileged-Identity-Management-for-elevated-access).

1. Para todos los demás roles de Azure AD, revise la lista de asignaciones, identifique a los administradores que ya no necesitan el rol y quítelos de sus asignaciones.

Para automatizar los dos últimos pasos, puede usar las revisiones de acceso en Privileged Identity Management. Con los pasos descritos en [Inicio de una revisión de acceso para los roles de directorio de Azure AD en Privileged Identity Management](pim-how-to-start-security-review.md), puede configurar una revisión de acceso de cada rol de Azure AD que tenga uno o varios miembros.

![Panel Crear una revisión de acceso de roles de Azure AD](./media/pim-deployment-plan/create-access-review.png)

Establezca los revisores como **Miembros (autoasignado)** . Todos los usuarios del rol recibirán un correo electrónico en el que se les pedirá que confirmen que necesitan el acceso. También debe activar **Requerir motivo de la aprobación** en la configuración avanzada para que los usuarios deban indicar por qué necesitan el rol. En función de esta información, puede quitar usuarios de roles innecesarios o delegarlos a roles de administrador más granulares.

Las revisiones de acceso se basan en los correos electrónicos para recordar a los usuarios que deben revisar su acceso a los roles. Si tiene cuentas con privilegios que no tienen correos electrónicos vinculados, asegúrese de rellenar el campo de correo electrónico secundario de dichas cuentas. Para más información, vea la información sobre el [atributo proxyAddresses en Azure AD](https://support.microsoft.com/help/3190357/how-the-proxyaddresses-attribute-is-populated-in-azure-ad).

#### <a name="plan-azure-resource-role-delegation"></a>Planeación de la delegación de roles de recursos de Azure

Para las suscripciones y los recursos de Azure, puede configurar un proceso de revisión de acceso similar para revisar los roles de cada suscripción o recurso. El objetivo de este proceso es minimizar las asignaciones de administrador de acceso de usuario y propietario conectadas a cada suscripción o recurso y quitar las asignaciones innecesarias. Sin embargo, las organizaciones suelen delegar dichas tareas al propietario de cada suscripción o recurso porque este conoce mejor los roles específicos (sobre todo los soles personalizados).

Si tiene el rol de administrador global que intenta implementar PIM para los roles de Azure en su organización, puede [elevar el acceso para administrar todas las suscripciones de Azure](../../role-based-access-control/elevate-access-global-admin.md?toc=%2fazure%2factive-directory%2fprivileged-identity-management%2ftoc.json) y poder acceder a cada suscripción. A continuación, puede encontrar a cada propietario de la suscripción y trabajar con él para quitar las asignaciones innecesarias y minimizar la asignación del rol de propietario.

Los usuarios con el rol de propietario para una suscripción de Azure también pueden utilizar las [revisiones de acceso para recursos de Azure](pim-resource-roles-start-access-review.md) a fin de auditar y quitar las asignaciones de roles innecesarias con un proceso similar al descrito anteriormente para los roles de Azure AD.

### <a name="decide-which-role-assignments-should-be-protected-by-privileged-identity-management"></a>Decidir qué asignaciones de roles deben protegerse mediante Privileged Identity Management

Después de limpiar las asignaciones de roles con privilegios en su organización, deberá decidir qué roles desea proteger con Privileged Identity Management.

Si un rol está protegido con Privileged Identity Management, los usuarios aptos asignados a él deben elevarse para usar los privilegios concedidos por el rol. El proceso de elevación también podría incluir la obtención de la aprobación, el uso de la autenticación multifactor y la explicación del motivo por el que se realiza la activación. Privileged Identity Management también puede realizar un seguimiento de las elevaciones a través de las notificaciones y los registros de eventos de auditoría de Privileged Identity Management y Azure AD.

La elección de los roles que se desean proteger con Privileged Identity Management puede resultar un proceso difícil que variará para cada organización. En esta sección se proporciona asesoramiento sobre procedimientos recomendados para roles de Azure y Azure AD.

#### <a name="azure-ad-roles"></a>Roles de Azure AD

Es importante dar prioridad a la protección de los roles de Azure AD que tienen la mayoría de permisos. Tomando como referencia los patrones de uso de todos los clientes de Privileged Identity Management, los 10 principales roles de Azure AD administrados por Privileged Identity Management son los siguientes:

1. Administrador global
1. Administrador de seguridad
1. Administrador de usuarios
1. Administrador de Exchange
1. Administrador de SharePoint
1. Administrador de Intune
1. Lector de seguridad
1. Administrador de servicios
1. Administrador de facturación
1. Administrador de Skype Empresarial

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** administrar todos los administradores globales y administradores de seguridad mediante Privileged Identity Management como primer paso porque son los usuarios que pueden causar más daños al vulnerar su seguridad.

Es importante tener en cuenta qué datos y permisos son más sensibles para su organización. Por ejemplo, puede que algunas organizaciones deseen proteger su rol de administrador de Power BI o su rol de administrador de equipos con Privileged Identity Management, ya que pueden acceder a los datos y cambiar los flujos de trabajo principales.

Si existen roles con usuarios invitados asignados, serán vulnerables a ataques.

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** administrar todos los roles con usuarios invitados mediante Privileged Identity Management para reducir los riesgos asociados a las cuentas de usuario invitado en peligro.

Los roles de lector, como lector de directorio, lector del Centro de mensajes y lector de seguridad, a veces se consideran menos importantes que otros roles porque no tienen permiso de escritura. Sin embargo, tenemos algunos clientes que también protegen estos roles porque los atacantes con acceso a estas cuentas podrían leer información confidencial, como datos personales. Tenga este riesgo en cuenta al decidir si los roles de lector de la organización deben administrase con Privileged Identity Management.

#### <a name="azure-roles"></a>Roles de Azure

Al decidir qué asignaciones de roles deben administrarse mediante Privileged Identity Management para recursos de Azure, primero debe identificar las suscripciones y los recursos que son más vitales para su organización. Ejemplos de tales suscripciones y recursos son:

- Recursos que hospedan la información más confidencial.
- Recursos de los que dependen las principales aplicaciones orientadas al cliente.

Si es un administrador global y tiene problemas para decidir qué suscripciones y recursos son más importantes, debe ponerse en contacto con los propietarios de la suscripción de su organización para recopilar una lista de los recursos administrados por cada suscripción. A continuación, colabore con los propietarios de la suscripción para agrupar los recursos en función del nivel de gravedad en caso de que estén en peligro (bajo, medio o alto). Dé prioridad a la administración de recursos con Privileged Identity Management en función de este nivel de gravedad.

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** colaborar con los propietarios de las suscripciones y recursos de servicios críticos para configurar el flujo de trabajo de Privileged Identity Management de todos los roles dentro de suscripciones y recursos confidenciales.

Privileged Identity Management para recursos de Azure es compatible con las cuentas de servicio de tiempo limitado. Debe tratar las cuentas de servicio exactamente de la misma forma en que trataría una cuenta de usuario normal.

En el caso de las suscripciones y los recursos que no son tan críticos, no necesitará configurar Privileged Identity Management para todos los roles. Sin embargo, todavía debe proteger los roles de administrador de acceso de usuario y propietario con Privileged Identity Management.

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** que administre los roles de propietario y de administrador de acceso de usuario de todas las suscripciones y recursos mediante Privileged Identity Management.

### <a name="decide-whether-to-use-a-group-to-assign-roles"></a>Decisión de si se va a usar un grupo para asignar roles

El hecho de asignar un rol a un grupo en lugar de a usuarios individuales es una decisión estratégica. En el momento de la planeación, considere la posibilidad de asignar un rol a un grupo para administrar las asignaciones de roles cuando:

- Muchos usuarios se asignan a un rol
- Desea delegar la asignación del rol

#### <a name="many-users-are-assigned-to-a-role"></a>Muchos usuarios se asignan a un rol

La realización de un seguimiento de quién está asignado a un rol y de la administración de sus asignaciones en función del momento en que se necesitan pueden tardar tiempo cuando se realiza manualmente. Para asignar un grupo a un rol, en primer lugar [cree un grupo asignable de roles](../users-groups-roles/roles-groups-create-eligible.md) y luego asigne el grupo como válido para un rol. Esta acción somete a todos los miembros del grupo al mismo proceso de activación que los usuarios individuales que son aptos para elevarse al rol. Los miembros del grupo activan de forma individual sus asignaciones al grupo mediante el proceso de aprobación y solicitud de activación de Privileged Identity Management. El grupo no está activado, solo la pertenencia a grupos del usuario.

#### <a name="you-want-to-delegate-assigning-the-role"></a>Desea delegar la asignación del rol

Un propietario de grupo puede administrar la pertenencia de un grupo. Para los grupos a los que se pueden asignar roles de Azure AD, solo el administrador de roles con privilegios, el administrador global y los propietarios de grupos pueden administrar la pertenencia a grupos. Al agregar nuevos miembros al grupo, el miembro obtiene acceso a los roles a los que se asigna el grupo si la asignación es apta o está activa. Use propietarios de grupo para delegar la administración de la pertenencia a grupos de un rol asignado para reducir la amplitud del privilegio necesario. Para más información sobre la asignación de un propietario a un grupo al crear el grupo, consulte [Creación de un grupo al que se pueden asignar roles en Azure Active Directory](../users-groups-roles/roles-groups-create-eligible.md).

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** que, mediante Privileged Identity Management, administre grupos a los que se pueden asignar roles de Azure AD. Una vez que PIM administra un grupo al que se pueden asignar roles, se denomina grupo de acceso privilegiado. Use PIM para requerir que los propietarios de grupos activen su asignación de roles de propietario antes de poder administrar la pertenencia a grupos. Para más información sobre la administración de grupos mediante PIM, consulte [Administración de grupos de acceso con privilegios (versión preliminar) en Privileged Identity Management](groups-discover-groups.md).

### <a name="decide-which-role-assignments-should-be-permanent-or-eligible"></a>Decidir qué asignaciones de roles deben ser permanentes o válidas

Una vez que haya decidido la lista de roles que se deben administrar con Privileged Identity Management, debe decidir qué usuarios deben obtener el rol válido frente al rol activo permanentemente. Los roles activos permanentemente son roles normales asignados mediante recursos de Azure y Azure Active Directory, mientras que los roles válidos solo se pueden asignar en Privileged Identity Management.

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** tener cero asignaciones activas permanentemente para los roles de Azure AD y los roles de Azure aparte de las [dos cuentas de acceso para escenarios de máxima emergencia](../users-groups-roles/directory-emergency-access.md) recomendadas, que deben tener el rol de administrador global permanente.

Aunque se recomienda no tener ningún administrador permanente, a veces resulta complicado para las organizaciones conseguir esto con rapidez. A continuación, se indican los aspectos que se deben tener en cuenta al tomar esta decisión:

- Frecuencia de elevación: si el usuario solo necesita la asignación con privilegios una vez, no debería tener la asignación permanente. Por otro lado, si el usuario necesita el rol para su trabajo cotidiano y el uso de Privileged Identity Management reduciría considerablemente su productividad, se puede considerar asignarle el rol permanente.
- Casos específicos de su organización: si la persona a la que se va asignar el rol válido pertenece a un equipo remoto o se trata de un ejecutivo de alto rango, hasta el punto que la comunicación y la aplicación del proceso de elevación resultan complicadas, se puede considerar asignarle el rol permanente.

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** configurar revisiones de acceso recurrentes para los usuarios con asignaciones de roles permanentes (en caso de que haya alguno). Obtenga más información sobre la revisión de acceso recurrente en la última sección de este plan de implementación.

### <a name="draft-your-privileged-identity-management-settings"></a>Esbozo de la configuración de Privileged Identity Management

Antes de implementar la solución Privileged Identity Management, es aconsejable realizar un borrador de la configuración de Privileged Identity Management para cada rol con privilegios que su organización usa. Esta sección incluye algunos ejemplos de configuración de Privileged Identity Management para roles particulares (son solo para referencia y pueden ser diferentes para su organización). Cada una de estas configuraciones se explica de forma detallada con las recomendaciones de Microsoft proporcionadas después de las tablas.

#### <a name="privileged-identity-management-settings-for-azure-ad-roles"></a>Configuración de Privileged Identity Management para roles de Azure AD

| Role | Requerir MFA | Notification | Vale de incidente | Requerir aprobación | Aprobador | Duración de la activación | Administrador permanente |
| --- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Administrador global | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | Otros administradores globales | 1 hora | Cuentas de acceso de emergencia |
| Administrador de Exchange | :heavy_check_mark: | :heavy_check_mark: | :x: | :x: | None | 2 horas | None |
| Administrador del departamento de soporte técnico | :x: | :x: | :heavy_check_mark: | :x: | None | 8 horas | None |

#### <a name="privileged-identity-management-settings-for-azure-roles"></a>Configuración de Privileged Identity Management para roles de Azure

| Role | Requerir MFA | Notification | Requerir aprobación | Aprobador | Duración de la activación | Administrador activo | Expiración de rol activo | Expiración de rol válido |
| --- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Propietario de las suscripciones críticas | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | Otros propietarios de la suscripción | 1 hora | None | N/D | 3 meses |
| Administrador de acceso de usuario de suscripciones menos críticas | :heavy_check_mark: | :heavy_check_mark: | :x: | None | 1 hora | None | N/D | 3 meses |
| Colaborador de la máquina virtual | :x: | :heavy_check_mark: | :x: | None | 3 horas | None | N/D | 6 meses |

En la tabla siguiente se describe cada configuración.

| Configuración | Descripción |
| --- | --- |
| Role | Nombre del rol para el que se define la configuración. |
| Requerir MFA | Si el usuario apto necesita realizar MFA antes de activar el rol.<br/><br/> :heavy_check_mark: **Microsoft recomienda** aplicar MFA para todos los roles de administrador, especialmente si los roles tienen usuarios invitados. |
| Notification | Si se establece en true, el administrador global, el administrador de roles con privilegios y el administrador de seguridad de la organización recibirán una notificación por correo electrónico cuando un usuario apto activa el rol.<br/><br/>**Nota:** Algunas organizaciones no tienen una dirección de correo electrónico asociada a sus cuentas de administrador para obtener estas notificaciones por correo electrónico; para obtenerlas, debe establecer una dirección de correo electrónico alternativa para que los administradores reciban estos correos. |
| Vale de incidente | Si el usuario apto necesita registrar un número de vale de incidente al activar el rol. Esta configuración ayuda a una organización a identificar cada activación con un número de incidente interno para mitigar las activaciones no deseadas.<br/><br/> :heavy_check_mark: **Microsoft recomienda** sacar provecho de los números de vales de incidentes para vincular Privileged Identity Management con el sistema interno. Este método puede ser útil para los aprobadores que necesitan contexto para la activación. |
| Requerir aprobación | Si el usuario apto necesita obtener aprobación para activar el rol.<br/><br/> :heavy_check_mark: **Microsoft recomienda** configurar la aprobación con el permiso máximo. Según los patrones de uso de todos los clientes de Privileged Identity Management, los roles de administrador global, administrador de usuarios, administrador de Exchange, administrador de seguridad y administrador de contraseñas son los más comunes con la configuración de la aprobación. |
| Aprobador | Si se requiere aprobación para activar el rol válido, indique qué personas deben aprobar la solicitud. De forma predeterminada, Privileged Identity Management establece que el aprobador sean todos los usuarios que sean administradores de roles con privilegios, ya sean permanentes o válidos.<br/><br/>**Nota:** Si un usuario es apto para un rol de Azure AD y un aprobador del rol, no podrá realizar aprobaciones para sí mismo.<br/><br/> :heavy_check_mark: **Microsoft recomienda** elegir como aprobadores a los usuarios que conozcan mejor el rol y sus usuarios frecuentes, en lugar de a un administrador global. |
| Duración de la activación | El período de tiempo durante el cual el usuario estará activado en el rol antes de que este expire. |
| Administrador permanente | Enumere los usuarios que serán administradores permanentes del rol (nunca tienen que ser activos).<br/><br/> :heavy_check_mark: **Microsoft recomienda** tener cero administradores permanente para todos los roles, excepto en el caso de los administradores globales. Lea más información al respecto en la sección de este plan que trata sobre quién debe establecerse como válido y quién como activo permanentemente. |
| Administrador activo | Para recursos de Azure, el administrador activo es la lista de usuarios que nunca tendrán que ser activos para usar el rol. Esta lista no se conoce como administrador permanente como en los roles de Azure AD, porque puede establecer un tiempo de expiración para cuándo el usuario perderá este rol. |
| Expiración de rol activo | Las asignaciones de roles activos para los roles de Azure expiran después de la duración configurada. Puede elegir entre 15 días, 1 mes, 3 meses, 6 meses, 1 año o activo permanentemente. |
| Expiración de rol válido | Las asignaciones de roles aptos para los roles de Azure expiran después de esta duración. Puede elegir entre 15 días, 1 mes, 3 meses, 6 meses, 1 año o válido permanentemente. |

## <a name="implementation-plan"></a>Plan de implementación

La base para un planeamiento adecuado es aquella sobre la que se puede implementar una aplicación correctamente con Azure Active Directory.  Proporciona una integración y una seguridad inteligentes, lo que simplifica la incorporación a la vez que reduce el tiempo necesario para realizar implementaciones correctas.  Esta combinación garantiza una integración sencilla de la aplicación, a la vez que mitiga el tiempo de inactividad para los usuarios finales.

### <a name="identify-test-users"></a>Identificación de los usuarios de prueba

Utilice esta sección para identificar un conjunto de usuarios y grupos de usuarios para validar la implementación. Según la configuración seleccionada en la sección de planeamiento, identifique a los usuarios que desea probar para cada rol.

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** establecer a los propietarios del servicio de cada rol de Azure AD como usuarios de prueba para que estos se familiaricen con el proceso y se conviertan en un autor interno de la implementación.

En esta tabla, identifique a los usuarios de prueba que verificarán si la configuración para los roles funciona.

| Nombre de rol | Usuarios de prueba |
| --- | --- |
| &lt;Nombre de rol&gt; | &lt;Usuarios para probar el rol&gt; |
| &lt;Nombre de rol&gt; | &lt;Usuarios para probar el rol&gt; |

### <a name="test-implementation"></a>Prueba de la implementación

Ahora que ha identificado a los usuarios de prueba, use este paso para configurar Privileged Identity Management para los usuarios de prueba. Si la organización desea incorporar el flujo de trabajo de Privileged Identity Management en su propia aplicación interna en lugar de utilizar Privileged Identity Management en Azure Portal, todas las operaciones de Privileged Identity Management también se admitirán con [Graph API](/graph/api/resources/privilegedidentitymanagement-root).

#### <a name="configure-privileged-identity-management-for-azure-ad-roles"></a>Configuración de Privileged Identity Management para roles de Azure AD

1. [Defina la configuración del rol de Azure AD](pim-how-to-change-default-settings.md) en función de lo que haya planeado.

1. Vaya a **Roles de Azure AD**, seleccione **Roles** y luego seleccione el rol que acaba de configurar.

1. Para el grupo de usuarios de prueba, si estos ya están establecidos como un administrador permanente, puede establecerlos como válidos buscándolos y convirtiéndolos de permanentes a válidos seleccionando los tres puntos de su fila. Si todavía no tienen ninguna asignación de roles, puede [realizar una nueva asignación válida](pim-how-to-add-role-to-user.md#make-a-user-eligible-for-a-role).

1. Repita los pasos 1 a 3 para todos los roles que desea probar.

1. Una vez haya configurado los usuarios de prueba, deberá volver a enviarles el vínculo para saber cómo [activar su rol de Azure AD](pim-how-to-activate-role.md).

#### <a name="configure-privileged-identity-management-for-azure-roles"></a>Configuración de Privileged Identity Management para roles de Azure

1. [Defina la configuración del rol de recursos de Azure](pim-resource-roles-configure-role-settings.md) para un rol dentro de una suscripción o un recurso que desee probar.

1. Vaya a **Recursos de Azure** para esa suscripción, seleccione **Roles** y luego elija el rol que ha configurado.

1. Para el grupo de usuarios de prueba, si ya son un administrador activo, puede convertirlos en válidos buscándolos y [actualizando su asignación de roles](pim-resource-roles-assign-roles.md#update-or-remove-an-existing-role-assignment). Si todavía no tienen ningún rol, puede [asignar uno nuevo](pim-resource-roles-assign-roles.md#assign-a-role).

1. Repita los pasos 1 a 3 para todos los roles que desea probar.

1. Una vez haya configurado los usuarios de prueba, deberá volver a enviarles el vínculo para saber cómo [activar su rol de recursos de Azure](pim-resource-roles-activate-your-roles.md).

Esta fase debe usarse para verificar si toda la configuración establecida para los roles funciona correctamente. Utilice la siguiente tabla para documentar las pruebas. También debe usar esta fase para optimizar la comunicación con los usuarios afectados.

| Role | Comportamiento esperado durante la activación | Resultados reales |
| --- | --- | --- |
| Administrador global | (1) Requerir MFA<br/>(2) Requerir aprobación<br/>(3) El aprobador recibe la notificación y puede aprobar<br/>(4) El rol expira después del tiempo preestablecido |  |
| Propietario de la suscripción *X* | (1) Requerir MFA<br/>(2) La asignación válida expira después del período de tiempo configurado |  |

### <a name="communicate-privileged-identity-management-to-affected-stakeholders"></a>Comunicación de Privileged Identity Management con las partes interesadas afectadas

La implementación de Privileged Identity Management requerirá pasos adicionales para los usuarios de roles con privilegios. Aunque Privileged Identity Management reduce significativamente los problemas de seguridad asociados con las identidades con privilegios, es necesario comunicar el cambio de forma eficaz antes de la implementación en toda la organización. Dependiendo del número de administradores afectados, las organizaciones a menudo optan por crear un documento interno, un vídeo o un correo electrónico sobre el cambio. En estas comunicaciones suele incluirse la siguiente información:

- ¿Qué es PIM?
- ¿Qué ventaja aporta para la organización?
- ¿A quién afectará?
- ¿Cuándo se implementará PIM?
- ¿Qué pasos adicionales serán necesarios para que los usuarios activen su rol?
    - Debe enviar vínculos a la documentación:
    - [Activación de roles de Azure AD](pim-how-to-activate-role.md)
    - [Activación de roles de Azure](pim-resource-roles-activate-your-roles.md)
- Información de contacto o vínculo al departamento de soporte técnico para problemas asociados con PIM

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** dedicar tiempo al equipo/departamento de soporte técnico para ofrecerle orientación sobre el flujo de trabajo de Privileged Identity Management (en caso de que su organización disponga de un equipo interno de soporte técnico de TI). Debe proporcionarle los documentos apropiados y su información de contacto.

### <a name="move-to-production"></a>Paso a producción

Una vez haya finalizado las pruebas satisfactoriamente, mueva Privileged Identity Management a producción repitiendo todos los pasos de las fases de prueba para todos los usuarios de cada rol que haya definido en la configuración de Privileged Identity Management. En el caso de Privileged Identity Management para roles de Azure AD, las organizaciones suelen probar e implementar Privileged Identity Management para los administradores globales antes de probar e implementar Privileged Identity Management para otros roles. En cambio, en el caso de los recursos de Azure, las organizaciones suelen probar e implementar Privileged Identity Management en una suscripción de Azure cada vez.

### <a name="if-a-rollback-is-needed"></a>Si se necesita una reversión

Si Privileged Identity Management no funciona según lo previsto en el entorno de producción, los pasos de reversión siguientes pueden ayudarle a revertir a un estado correcto conocido antes de la configuración de Privileged Identity Management:

#### <a name="azure-ad-roles"></a>Roles de Azure AD

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Abra **Azure AD Privileged Identity Management**.
1. Seleccione **Roles de Azure AD** y luego elija **Roles**.
1. Para cada rol que haya configurado, seleccione los puntos suspensivos ( **...** ) correspondientes a todos los usuarios que tengan una asignación válida.
1. Seleccione la opción **Hacer permanente** para que la asignación de roles sea permanente.

#### <a name="azure-roles"></a>Roles de Azure

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Abra **Azure AD Privileged Identity Management**.
1. Seleccione **Recursos de Azure** y luego elija una suscripción o un recurso que desee revertir.
1. Seleccione **Roles**.
1. Para cada rol que haya configurado, seleccione los puntos suspensivos ( **...** ) correspondientes a todos los usuarios que tengan una asignación válida.
1. Seleccione la opción **Hacer permanente** para que la asignación de roles sea permanente.

## <a name="next-steps-after-deploying"></a>Pasos siguientes después de implementar

La implementación correcta de Privileged Identity Management en producción es un importante paso adelante para proteger las identidades con privilegios de su organización. La implementación de Privileged Identity Management incluye características de Privileged Identity Management adicionales que debe usar para la seguridad y el cumplimiento.

### <a name="use-privileged-identity-management-alerts-to-safeguard-your-privileged-access"></a>Uso de alertas de Privileged Identity Management para proteger el acceso con privilegios

Para más información sobre el uso de la funcionalidad integrada de alertas de Privileged Identity Management para proteger mejor su organización, consulte [Alertas de seguridad](pim-how-to-configure-security-alerts.md#security-alerts). Estas alertas se incluyen: los administradores no usan roles con privilegios, los roles se asignan fuera de Privileged Identity Management, se activan roles con demasiada frecuencia y muchas más. Para ofrecer una protección total a su organización, debe consultar regularmente la lista de alertas y corregir los problemas. Puede ver y corregir las alertas de la manera siguiente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Abra **Azure AD Privileged Identity Management**.
1. Seleccione **Roles de Azure AD** y luego elija **Alertas**.

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** solucionar inmediatamente todas las alertas marcadas con gravedad alta. En el caso de las alertas de seguridad de gravedad media y baja, debe mantenerse informado y realizar cambios si considera que existe una amenaza de seguridad.

Si alguna de las alertas específicas no es útil o no es aplicable a su organización, siempre puede descartarla en la página de alertas. Siempre puede revertir este descargue más tarde en la página de configuración de Azure AD.

### <a name="set-up-recurring-access-reviews-to-regularly-audit-your-organizations-privileged-identities"></a>Configuración de revisiones de acceso periódicas para auditar regularmente las identidades con privilegios de su organización

Las revisiones de acceso son la mejor forma de preguntar a los usuarios con roles con privilegios asignados o a revisores específicos si cada usuario necesita la identidad con privilegios. Las revisiones de acceso son excelentes si desea reducir la superficie expuesta a ataques y mantener el cumplimiento. Para más información sobre cómo iniciar una revisión de acceso, vea los artículos sobre [revisiones de acceso de roles de Azure AD](pim-how-to-start-security-review.md) y [revisiones de acceso de roles de Azure](pim-resource-roles-start-access-review.md). En algunas organizaciones, es necesario realizar revisiones de acceso periódicas para cumplir las leyes y los reglamentos, mientras que, en otras, las revisiones de acceso son la mejor manera de aplicar el principio del privilegio mínimo en toda la organización.

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** configurar revisiones de acceso trimestrales para todos los roles de Azure AD y Azure.

En la mayoría de los casos, los revisores de los roles de Azure AD son los propios usuarios, mientras que los revisores de los roles de Azure son los propietarios de la suscripción en la que se encuentra el rol. Sin embargo, suele darse el caso en el que las empresas tienen cuentas con privilegios que no están vinculadas con la dirección de correo electrónico de una persona concreta. En esos casos, nadie lee ni revisa el acceso.

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** agregar una dirección de correo electrónico secundaria para todas las cuentas con asignaciones de roles con privilegios que no están vinculadas a una dirección de correo electrónico revisada con regularidad.

### <a name="get-the-most-out-of-your-audit-log-to-improve-security-and-compliance"></a>Sacar el máximo partido de su registro de auditoría para mejorar la seguridad y el cumplimiento

El registro de auditoría le permite mantenerse informado y cumplir los reglamentos. Actualmente, Privileged Identity Management almacena el historial de toda la organización durante 30 días dentro de su registro de auditoría, con información que incluye:

- Activación/desactivación de roles válidos
- Actividades de asignación de roles dentro y fuera de Privileged Identity Management
- Cambios en la configuración de roles
- Actividades de solicitud, aprobación o denegación de activación de roles con la configuración de aprobación
- Actualización de alertas

Puede acceder a los registros de auditoría si es un administrador global o un administrador de roles con privilegios. Para más información, vea el [historial de auditoría para los roles de Azure AD](pim-how-to-use-audit-log.md) y el [historial de auditoría para los roles de Azure](azure-pim-resource-rbac.md).

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** tener al menos un administrador que lea todos los eventos de auditoría cada semana y los exporte una vez al mes.

Si desea almacenar automáticamente los eventos de auditoría durante un período de tiempo más largo, el registro de auditoría de Privileged Identity Management se sincroniza automáticamente en los [registros de auditoría de Azure AD](../reports-monitoring/concept-audit-logs.md).

> [!TIP]
> :heavy_check_mark: **Microsoft recomienda** configurar la [supervisión de registros de Azure](../reports-monitoring/concept-activity-logs-azure-monitor.md) para archivar los eventos de auditoría en una cuenta de almacenamiento de Azure para mayor seguridad y cumplimiento.
