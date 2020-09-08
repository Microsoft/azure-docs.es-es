---
title: Planeamiento de una implementación de revisiones de acceso de Azure Active Directory
description: Guía de planeamiento para la correcta implementación de revisiones de acceso
services: active-directory
documentationCenter: ''
author: BarbaraSelden
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 08/14/2020
ms.author: barclayn
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7eb39f1053abeb201c413db7c6bbd3e9f261bd95
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89011351"
---
# <a name="planning-azure-active-directory-access-reviews-deployment"></a>Planeamiento de una implementación de revisiones de acceso de Azure Active Directory

[Las revisiones de acceso de Azure Active Directory (Azure AD)](access-reviews-overview.md) ayudan a su organización a mantener la red más segura mediante la administración del [ciclo de vida de acceso a los recursos](identity-governance-overview.md). Las revisiones de acceso permiten:

* Programar revisiones periódicas o realizar revisiones ad hoc para ver quién tiene acceso a recursos específicos (por ejemplo, aplicaciones y grupos).

* Hacer un seguimiento de las revisiones con fines de información, cumplimiento normativo o directivas.

* Delegar las revisiones a administradores específicos, propietarios de empresas o usuarios finales que pueden atestiguar su necesidad de acceso continuado.

* Usar la información para determinar eficazmente si los usuarios deben seguir teniendo acceso.

* Automatizar los resultados de las revisiones (por ejemplo, eliminar el acceso de los usuarios a los recursos).

  ![Planeamiento de revisiones de acceso](./media/deploy-access-review/1-planning-review.png)

Las revisiones de acceso son una funcionalidad de [Azure AD Identity Governance](identity-governance-overview.md). Otras funcionalidades son la [administración de derechos](entitlement-management-overview.md), [Privileged Identity Management](../privileged-identity-management/pim-configure.md) y los [términos de uso](../conditional-access/terms-of-use.md). Juntas, permiten a las organizaciones dar respuesta a estas cuatro preguntas:

* ¿Qué usuarios deben tener acceso a qué recursos?

* ¿Qué hacen esos usuarios con el acceso concedido?

* ¿La organización dispone de un sistema de control eficaz para administrar el acceso?

* ¿Los auditores pueden comprobar qué controles funcionan?

Planear la implementación de revisiones de acceso es esencial para garantizar el cumplimiento de la estrategia de gobernanza que haya previsto para los usuarios de su organización.

### <a name="key-benefits"></a>Ventajas principales

Las principales ventajas de habilitar las revisiones de acceso son:

* **Administración de la colaboración**. Las revisiones de acceso permiten a las organizaciones administrar el acceso a todos los recursos que necesitan los usuarios. Cuando sus usuarios comparten información y colaboran, las organizaciones tienen la garantía de que únicamente los usuarios autorizados manejan la información.

* **Administración de los riesgos**. Las revisiones de acceso ofrecen a las organizaciones una forma de revisar el acceso a los datos y las aplicaciones. Así se reduce el riesgo de pérdida de datos. Se incluyen funciones que permiten revisar periódicamente el acceso por parte de asociados externos a los recursos corporativos. 

* **Solución para el cumplimiento y la gobernanza**. Mediante las revisiones de acceso, puede regular y volver a certificar el ciclo de vida del acceso a grupos, aplicaciones y sitios. Es posible regular las revisiones de seguimiento con fines de cumplimiento o para aplicaciones sensibles al riesgo específicas de su organización. 

* **Reducción del costo**. Las revisiones de acceso se crean en la nube y son compatibles de forma nativa con otros recursos en la nube, como grupos, aplicaciones y paquetes de acceso. El uso de revisiones de acceso es menos caro que si tiene que desarrollar herramientas propias o actualizar su conjunto de herramientas local.

### <a name="training-resources"></a>Recursos de aprendizaje

Los siguientes vídeos pueden ser útiles a medida que se familiarice con las revisiones de acceso:

* [Qué son las revisiones de acceso de Azure AD](https://youtu.be/kDRjQQ22Wkk)

* [Cómo crear revisiones de acceso en Azure AD](https://youtu.be/6KB3TZ8Wi40)

* [Cómo habilitar las revisiones de acceso en Azure AD](https://youtu.be/X1SL2uubx9M)

* [Cómo revisar el acceso mediante Mi acceso](https://youtu.be/tIKdQhdHLXU)

### <a name="licenses"></a>Licencias

Se necesita una licencia válida de Azure AD Premium (P2) para cada persona, sin contar a los administradores globales o los administradores de usuarios, que crearán o realizarán revisiones de acceso. Para obtener más información, consulte los [requisitos de licencia de las revisiones de acceso](access-reviews-overview.md).

Además es posible que necesite utilizar otras características de gobernanza de identidades, como la [administración del ciclo de vida de los derechos](entitlement-management-overview.md) o Privileged Identity Management. En tal caso, puede que también necesite licencias relacionadas. Para más información, consulte los [precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="plan-the-access-reviews-deployment-project"></a>Planeamiento del proyecto de implementación de revisiones de acceso

Tenga en cuenta las necesidades de su organización a la hora de determinar la estrategia de implementación de las revisiones de acceso en su entorno.

### <a name="engage-the-right-stakeholders"></a>Interactuar con las partes interesadas adecuadas

Cuando fracasan los proyectos tecnológicos, normalmente se debe a expectativas incorrectas relacionadas con el impacto, los resultados y las responsabilidades. Para evitar estos problemas, [asegúrese de que interactúa con las partes interesadas adecuadas](https://aka.ms/deploymentplans) y que los roles del proyecto están claros.

En el caso de las revisiones de acceso, probablemente tenga que incluir a representantes de los siguientes equipos de su organización:

* **Administración de TI**: administra la infraestructura de TI y las inversiones de su empresa en la nube, junto con las aplicaciones de software como servicio (SaaS). Este equipo se ocupará de:

   * Revisar el acceso con privilegios a la infraestructura y las aplicaciones, incluidos Office 365 y Azure AD.

   * Programar y ejecutar revisiones de acceso en grupos que suelen mantener listas de excepciones, o bien en proyectos piloto de TI para mantener las listas de acceso actualizadas.

   * Asegúrese de que el acceso mediante programación (scripts) a los recursos a través de entidades de servicio se regule y revise.

* **Equipos de desarrollo**: desarrollan y mantienen las aplicaciones para su organización. Este equipo se ocupará de:

   * Determinar quién puede acceder a los componentes de los recursos de SaaS, PaaS y IaaS que forman parte de las soluciones desarrolladas, y administrarlos.

   * Administrar grupos que tienen acceso a aplicaciones y herramientas para el desarrollo de aplicaciones internas.

   * Exigir identidades con privilegios que tengan acceso a software de producción o soluciones que estén hospedadas para sus clientes.

* **Unidades de negocio**: administran proyectos y tienen aplicaciones propias. Este equipo se ocupará de: 

   * Revisar y aprobar, o bien denegar, el acceso a grupos y aplicaciones para usuarios internos y externos.

   * Programar y llevar a cabo revisiones para atestiguar la necesidad de acceso continuado de empleados e identidades externas (por ejemplo, asociados).

* **Gobernanza corporativa**: garantiza que la organización siga las directivas internas y cumpla la normativa. Este equipo se ocupará de:

   * Solicitar o programar nuevas revisiones de acceso.

   * Evaluar los procesos y procedimientos para revisar el acceso (por ejemplo, la documentación y el mantenimiento de registros con fines de cumplimiento).

   * Examinar los resultados de las revisiones anteriores para los recursos más importantes.

> [!NOTE]
> Para ver las revisiones que requieren evaluaciones manuales, asegúrese de disponer de revisores y ciclos de revisión adecuados de acuerdo con sus necesidades de cumplimiento y directivas. Si los ciclos de revisión son demasiado frecuentes, o hay pocos revisores, la calidad podría disminuir. Además, es posible que haya demasiadas o muy pocas personas con acceso. 

### <a name="plan-communications"></a>Planeamiento de las comunicaciones

La comunicación es fundamental para que cualquier nuevo proceso de negocio tenga éxito. Comunique de forma proactiva a los usuarios cómo y cuándo cambiará su experiencia, y cómo obtener soporte técnico si experimentan problemas. 

#### <a name="communicate-changes-in-accountability"></a>Comunicar cambios que afectan a la rendición de cuentas

Las revisiones de acceso permiten a los propietarios de empresa cambiar la responsabilidad de revisar y tomar medidas en relación con el acceso continuado. Desvincular del equipo de TI las decisiones que afectan al acceso permite adoptar mejores decisiones sobre el acceso. Este es un cambio cultural en términos de rendición de cuentas y responsabilidad del propietario del recurso. Comunique de forma proactiva este cambio y asegúrese de que los propietarios de recursos estén preparados y capacitados para utilizar la información con el fin de tomar buenas decisiones.

Evidentemente, el equipo de TI querrá mantener el control sobre la decisión de acceso a toda la infraestructura, y la asignación de roles con privilegios. 

#### <a name="customize-email-communication"></a>Comunicación personalizada por correo electrónico

Al programar una revisión, tendrá que designar a los usuarios que se encargarán de hacerla. A continuación, se notificará a estos revisores por correo electrónico que tienen nuevas revisiones asignadas. Además recibirán recordatorios antes de que expire la revisión que tienen asignada.

Los administradores pueden optar por enviar esta notificación a mitad del plazo de revisión o el día antes de que expire. 

El correo electrónico enviado a los revisores puede personalizarse para incluir un breve mensaje que les inste a completar la revisión. Se recomienda utilizar texto adicional con el fin de:

* Incluir un mensaje personal para los revisores, de manera que sepan que se lo envía el departamento de TI o de cumplimiento de la empresa.

* Incluir un hipervínculo o una referencia a información interna sobre cuáles son las expectativas de la revisión, junto con material de referencia o aprendizaje adicional.

* Incluir un vínculo a las instrucciones sobre [cómo realizar una autorrevisión de acceso](review-your-access.md). 

  ![Correo electrónico del revisor](./media/deploy-access-review/2-plan-reviewer-email.png)

Después de seleccionar Iniciar revisión, los revisores serán dirigidos al [portal de MyAccess](https://myapplications.microsoft.com/) para las revisiones de acceso de grupos y aplicaciones. El portal les proporciona una visión general de todos los usuarios que tienen acceso al recurso que están revisando, junto con recomendaciones del sistema basadas en información sobre el último inicio de sesión y acceso.

### <a name="plan-a-pilot"></a>Planeamiento de un piloto

Se recomienda a los clientes que inicialmente prueben las revisiones de acceso con un grupo pequeño y que se centren en recursos que no sean críticos. Hacer pruebas piloto puede ayudarle a adaptar los procesos y las comunicaciones según sus necesidades, además de mejorar la habilidad de usuarios y revisores para cumplir los requisitos de seguridad y cumplimiento.

Para la prueba piloto, se recomienda que:

* Comience con revisiones en las que los resultados no se apliquen automáticamente, y que le permitan controlar las repercusiones.

* Asegúrese de que todos los usuarios tengan direcciones de correo electrónico válidas en Azure AD y de que reciban comunicaciones de correo electrónico para tomar las medidas oportunas. 

* Tome nota de cualquier acceso que se haya eliminado como parte de la prueba piloto por si necesita restaurarlo rápidamente.

* Supervise los registros de auditoría para asegurarse de que todos los eventos se auditen correctamente.

Para obtener más información, consulte los [procedimientos recomendados para una prueba piloto](../fundamentals/active-directory-deployment-plans.md).

## <a name="introduction-to-access-reviews"></a>Introducción a las revisiones de acceso

En esta sección se presentan los conceptos relacionados con las revisiones de acceso que debería conocer antes de planear revisiones.

### <a name="what-resource-types-can-be-reviewed"></a>¿Qué tipos de recursos se pueden revisar?

Una vez que integre los recursos de su organización con Azure AD (por ejemplo, usuarios, aplicaciones y grupos), se pueden administrar y revisar. 

Entre los objetivos típicos de revisión, se incluyen:

* [Aplicaciones integradas con Azure AD para el inicio de sesión único](../manage-apps/what-is-application-management.md) (por ejemplo, aplicaciones SaaS o de línea de negocio).

* La [pertenencia](../fundamentals/active-directory-manage-groups.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context) a un grupo (sincronizado con Azure AD o creado en Azure AD u Office 365, incluido Microsoft Teams).

* Un [paquete de acceso](/azure/active-directory/governance/entitlement-management-overview) que agrupa recursos (grupos, aplicaciones y sitios) en un único paquete para administrar el acceso.

* [Roles de recursos de Azure AD y de Azure Resource](../privileged-identity-management/pim-resource-roles-assign-roles.md), tal y como se definen en Privileged Identity Management.

### <a name="who-will-create-and-manage-access-reviews"></a>¿Quién creará y administrará las revisiones de acceso?

El rol administrativo que se necesita para crear, administrar o leer una revisión de acceso depende del tipo de recurso que se esté revisando. La tabla siguiente muestra los roles necesarios para cada tipo de recurso:

| Tipo de recurso| Crear y administrar revisiones de acceso (creadores)| Resultados de la revisión de acceso de lectura |
| - | - | -|
| Grupo o aplicación| Administrador global <p>Administrador de usuarios| Creadores y administrador de seguridad |
| Roles con privilegios en Azure AD| Administrador global <p>Administrador de roles con privilegios| Creadores <p>Lector de seguridad<p>Administrador de seguridad |
| Roles con privilegios en Azure (Resource)| Administrador global<p>Administrador de usuarios<p>Propietario del recurso| Creadores |
| Paquete de acceso| Administrador global<p>Creador del paquete de acceso| Solo administrador global |


Para obtener más información, consulte los [permisos del rol de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md).

### <a name="who-will-review-the-access-to-the-resource"></a>¿Quién revisará el acceso al recurso?

El creador de la revisión de acceso decide en el momento de creación quién hará la revisión. Esta configuración no se puede cambiar una vez iniciada la revisión. Los revisores están representados por tres roles:

* Propietarios de recursos, que son las empresas propietarias del recurso.

* Un conjunto de delegados seleccionados de forma individual por parte del administrador de revisiones de acceso.

* Usuarios finales que atestiguarán su necesidad de acceso continuado.

Al crear una revisión de acceso, los administradores pueden elegir uno o más revisores. Todos los revisores pueden iniciar y llevar a cabo una revisión; y elegir los usuarios que seguirán teniendo acceso a un recurso, o bien eliminarlos. 

### <a name="components-of-an-access-review"></a>Elementos de una revisión de acceso

Antes de implementar las revisiones de acceso, debe planear los tipos de revisión relevantes para su organización. Para ello, deberá tomar decisiones empresariales sobre lo que desea revisar y las medidas que deberán tomarse en función de esas revisiones.

Para crear una directiva de revisión de acceso, debe disponer de la información siguiente:

* ¿Cuáles son los recursos que se deben revisar?

* ¿Qué acceso se va revisar?

* ¿Con qué frecuencia se debe realizar la revisión?

* ¿Quién hará la revisión?

   * ¿Cómo se le notificará la revisión?

   * ¿Cuáles son las escalas de tiempo que se aplicarán a la revisión?

* ¿Qué medidas automáticas deberían aplicarse en función de la revisión?

   * ¿Qué ocurre si el revisor no responde a tiempo?

* ¿Qué medidas manuales deberán aplicarse como consecuencia de los resultados de la revisión?

* ¿Qué comunicaciones deben enviarse en función de las medidas adoptadas?


**Ejemplo de un plan de revisión de acceso**

| Componente| Value |
| - | - |
| **Recursos para revisar**| Acceso a Microsoft Dynamics |
| **Frecuencia de revisión**| Mensual |
| **Quién hace la revisión**| Administradores de programas del grupo de negocios de Dynamics |
| **Notificación**| Enviar un correo electrónico 24 horas antes de la revisión al alias Dynamics-Pms.<p>Incluir un mensaje personalizado a los revisores para garantizar su aceptación. |
| **Escala de tiempo**| 48 horas después de la notificación. |
|**Acciones automáticas**| Quitar el acceso a cualquier cuenta que no haya tenido un inicio de sesión interactivo en 90 días. Para ello, quite el usuario del grupo de seguridad dynamics-access. <p>*Tome medidas si la revisión no se realiza dentro de la escala de tiempo prevista.* |
| **Medidas manuales**| Los revisores pueden aprobar eliminaciones antes de la medida automatizada si se desea. |
| **Comunicaciones**| Enviar a los usuarios internos (miembros) que sean eliminados un correo electrónico explicando que se les ha quitado el acceso y cómo pueden recuperarlo. |


 

### <a name="automate-actions-based-on-access-reviews"></a>Automatización de medidas basada en las revisiones de acceso

Para automatizar la eliminación de accesos, establezca la opción Aplicar automáticamente los resultados al recurso en Habilitar.

  ![Planeamiento de revisiones de acceso](./media/deploy-access-review/3-automate-actions-settings.png)

Una vez completada y finalizada la revisión, los usuarios que no hayan recibido aprobación del revisor se eliminarán automáticamente del recurso, o bien se conservarán sin acceso. Esto podría implicar la eliminación de su pertenencia a grupos, la asignación de aplicaciones o la revocación del derecho a obtener un rol con privilegios elevados.

Aceptar recomendaciones

Las recomendaciones se muestran a los revisores como parte de su experiencia de usuario y señalan el último inicio de sesión de una persona en el inquilino o el último acceso a una aplicación. Esta información ayuda a los revisores a tomar la decisión de acceso adecuada. Si se selecciona la opción Aceptar recomendaciones, se aplicarán las recomendaciones de la revisión de acceso. Al final de una revisión de acceso, el sistema aplicará estas recomendaciones automáticamente a los usuarios por los que los revisores no hayan respondido.

Las recomendaciones se basan en los criterios de la revisión de acceso. Por ejemplo, si configura la revisión para que elimine el acceso en el caso de que no se haya producido un inicio de sesión interactivo durante 90 días, se recomendará eliminar a todos los usuarios que coincidan con los criterios. Microsoft trabaja continuamente para mejorar las recomendaciones. 

### <a name="review-guest-user-access"></a>Revisión del acceso de usuarios invitados

Utilice las revisiones de acceso para revisar y limpiar las identidades de los asociados de organizaciones externas. Configurar una revisión por cada asociado podría satisfacer los requisitos de cumplimiento.

Seleccione una de las siguientes medidas para permitir a las identidades externas acceder a los recursos de la empresa:

* Agregado a un grupo 

* Invitado a Teams 

* Asignado a un paquete de acceso o una aplicación empresarial

* Asignado a un rol con privilegios en Azure AD o a una suscripción de Azure

Vea un [script de ejemplo](https://github.com/microsoft/access-reviews-samples/tree/master/ExternalIdentityUse). El script mostrará dónde se usan las identidades externas invitadas al inquilino. Puede consultar la pertenencia a grupos, las asignaciones de roles y las asignaciones de aplicaciones del usuario externo en Azure AD. El script no mostrará ninguna asignación fuera de Azure AD, como la asignación de derechos directos a los recursos de SharePoint, sin utilizar grupos.

Cuando cree una revisión de acceso para grupos o aplicaciones, podrá permitir que el revisor se centre en todos los usuarios con acceso o únicamente en los usuarios invitados. Si se seleccionan únicamente los usuarios invitados, los revisores recibirán una lista específica de identidades externas de Azure AD B2B que tienen acceso al recurso.

 ![Revisión del acceso de usuarios invitados](./media/deploy-access-review/4-review-guest-users-admin-ui.png)

> [!IMPORTANT]
> Esto NO incluirá a miembros externos que tengan el atributo userType. Tampoco se incluirá a usuarios invitados que no colaboren en Azure AD B2B. Por ejemplo, aquellos usuarios que tengan acceso a contenido compartido directamente a través de SharePoint.

## <a name="plan-access-reviews-for-access-packages"></a>Planeamiento de revisiones de acceso para paquetes de acceso

Los [paquetes de acceso](entitlement-management-overview.md) pueden simplificar enormemente su estrategia de gobernanza y de revisión de acceso. Un paquete de acceso agrupa todos los recursos e incluye el acceso que un usuario necesita para trabajar en un proyecto o realizar sus funciones. Por ejemplo, tal vez desee crear un paquete de acceso que incluya todas las aplicaciones que los desarrolladores de su organización necesitan, o bien todas las aplicaciones a las que los usuarios externos deberían tener acceso. A continuación, un administrador, o un administrador de paquetes de acceso delegado, agrupará los recursos (grupos, aplicaciones y sitios) y los roles que los usuarios necesitan para utilizar esos recursos.

Al [crear un paquete de acceso](entitlement-management-access-package-create.md), podrá definir una o varias directivas de acceso para establecer las condiciones en las que los usuarios pueden solicitar un paquete de acceso, determinar cómo será el proceso de aprobación y especificar la frecuencia con la que un usuario tendrá que volver a solicitar acceso. Las revisiones de acceso se configuran al crear o editar una directiva de paquete de acceso.

Abra la pestaña Ciclo de vida y desplácese a Revisiones de acceso.

 ![Editar directiva](./media/deploy-access-review/5-plan-access-packages-admin-ui.png)

## <a name="plan-access-reviews-for-groups"></a>Planeamiento de revisiones de acceso para grupos

Además de los paquetes de acceso, revisar la pertenencia a grupos es la forma más eficaz de regular el acceso. Se recomienda asignar el acceso a los recursos mediante [grupos de seguridad o grupos de Office 365](../fundamentals/active-directory-manage-groups.md); y añadir usuarios a esos grupos para darles acceso.

Un único grupo puede obtener acceso a todos los recursos pertinentes. Se puede asignar acceso al grupo para recursos individuales, o bien a un paquete de acceso que agrupe aplicaciones y otros recursos. Este método permite revisar el acceso al grupo en lugar del acceso de un usuario a cada aplicación. 

La pertenencia a grupos pueden revisarla los siguientes perfiles: 

* Administradores

* Propietarios del grupo

* Usuarios seleccionados, con capacidad de revisión delegada al crear la revisión

* Miembros del grupo, que atestiguarán su necesidad de acceso

### <a name="group-ownership"></a>Propiedad de grupo

Se recomienda que los propietarios del grupo revisen la pertenencia, ya que están en mejor disposición para saber quién necesita acceso. La propiedad de los grupos es diferente al tipo de grupo:

Los grupos creados en Office 365 y Azure AD tienen uno o más propietarios bien definidos. En la mayoría de los casos, estos propietarios son los revisores idóneos para sus grupos, ya que saben quién debe tener acceso. 

Por ejemplo, Microsoft Teams usa grupos de Office 365 como el modelo de autorización subyacente para conceder a los usuarios acceso a los recursos disponibles en SharePoint, Exchange, OneNote u otros servicios de Office 365. El creador del equipo se convierte automáticamente en propietario y debería ser responsable de atestiguar la pertenencia a ese grupo. 

Los grupos que se crean manualmente en el portal de Azure AD o mediante scripting a través de Microsoft Graph no necesitan tener propietarios definidos. Se recomienda que definirlos mediante el portal de Azure AD, en la sección "Propietarios" del grupo, o a través de Graph.

Los grupos que se sincronizan desde una instancia local de Active Directory no pueden tener un propietario en Azure AD. Al crear una revisión de acceso a los grupos, debe seleccionar las personas más adecuadas para decidir sobre la pertenencia a ellos.

> [!NOTE]
> Se recomienda definir directivas empresariales que determinen cómo se crean los grupos para establecer la propiedad y responsabilidad de los grupos de forma clara y revisar periódicamente la pertenencia. 

### <a name="review-membership-of-exclusion-groups-in-ca-policies"></a>Revisión de la pertenencia a grupos de exclusión en directivas de CA 

Hay ocasiones en las que las directivas de acceso condicional (CA) diseñadas para proteger su red no se deben aplicar a todos los usuarios. Por ejemplo, una directiva de CA que solo permita a los usuarios iniciar sesión cuando están conectados a la red corporativa podría no aplicarse al equipo de ventas, que viaja constantemente. En ese caso, los miembros del equipo de ventas se incluirían en un grupo, y ese grupo se excluiría de la directiva de CA. 

Revise la pertenencia a este grupo con regularidad, ya que la exclusión plantea un riesgo potencial si los miembros equivocados se excluyen del requisito.

Puede hacer [uso de revisiones de acceso de Azure AD para administrar los usuarios a los que se les ha excluido de las directivas de acceso condicional](conditional-access-exclusion.md).

### <a name="review-external-users-group-memberships"></a>Revisión de la pertenencia a grupos de un usuario externo

A fin de minimizar el trabajo manual y los posibles errores asociados, valore la posibilidad de usar [grupos dinámicos](../users-groups-roles/groups-create-rule.md) para asignar la pertenencia a un grupo en función de los atributos de un usuario. Tal vez desee crear uno o varios grupos dinámicos para usuarios externos. El patrocinador interno puede actuar como revisor de la pertenencia al grupo. 

Nota: Los usuarios externos que se quiten de un grupo como consecuencia de una revisión de acceso no se eliminarán del inquilino. 

Se pueden eliminar de un inquilino de forma manual o mediante un script.

### <a name="review-access-to-on-premises-groups"></a>Revisión del acceso a grupos locales

Las revisiones de acceso no permiten cambiar la pertenencia a grupos que se sincronicen desde el entorno local con [Azure AD Connect](../hybrid/whatis-azure-ad-connect.md). Esto se debe a que el origen de autoridad es local.

No obstante, puede utilizar las revisiones de acceso para programar y mantener revisiones periódicas de los grupos locales. Posteriormente, los revisores tomarán medidas que afectan al grupo local. Esta estrategia utiliza las revisiones de acceso como una herramienta para todas las revisiones.

Puede usar los resultados de una revisión de acceso en grupos locales y procesarlos aún más. Para ello:

* Descargue el informe CSV de la revisión de acceso y aplique la medida manualmente.

* Utilice Microsoft Graph para acceder mediante programación a los resultados y las decisiones de las revisión de acceso completadas.

Por ejemplo, para acceder a los resultados de un grupo administrado mediante Windows AD, utilice este [script de ejemplo de PowerShell](https://github.com/microsoft/access-reviews-samples/tree/master/AzureADAccessReviewsOnPremises). El script describe las llamadas a Graph necesarias y exporta los comandos de Windows AD-PowerShell para hacer los cambios.

## <a name="plan-access-reviews-for-applications"></a>Planeamiento de revisiones de acceso para aplicaciones 

Al revisar el acceso a una aplicación, está comprobando el acceso de empleados e identidades externas a la información y los datos de la aplicación. Lleve a cabo una revisión cuando necesite saber quién tiene acceso a una aplicación específica, en lugar de a un paquete de acceso o un grupo. 

Se recomienda planear revisiones de aplicaciones en los escenarios siguientes:

* Los usuarios tienen acceso directo a una aplicación (al margen de un grupo o un paquete de acceso).

* La aplicación expone información crítica o confidencial.

* La aplicación tiene requisitos específicos de cumplimiento que usted debe atestiguar.

* Existen sospechas de acceso inadecuado.

Para crear revisiones de acceso para una aplicación, establezca la propiedad "¿Se requiere la asignación de usuarios?" en Sí. Si se establece en No, todos los usuarios del directorio, incluidas las identidades externas, podrán acceder a la aplicación y usted no podrá revisar el acceso a la aplicación. 

 ![planeamiento de asignaciones de aplicaciones](./media/deploy-access-review/6-plan-applications-assignment-required.png)

A continuación, debe [asignar los usuarios y grupos](../manage-apps/assign-user-or-group-access-portal.md) que desea que tengan acceso. 

### <a name="reviewers-for-an-application"></a>Revisores de una aplicación

Las revisiones de acceso pueden ser de los miembros de un grupo o de los usuarios que se hayan asignado a una aplicación. Las aplicaciones de Azure AD no necesitan tener un propietario. Esta es la razón por la que no se puede elegir al propietario de la aplicación como revisor. El ámbito de una revisión se puede limitar aún más para revisar únicamente a los usuarios invitados asignados a una aplicación, en lugar de revisar todos los accesos.

## <a name="plan-review-of-azure-ad-and-azure-resource-roles"></a>Planeamiento de una revisión de roles de Azure AD y de Azure Resource

[Privileged Identity Management (PIM)](../privileged-identity-management/pim-configure.md) simplifica el modo en que las empresas administran el acceso con privilegios a los recursos de Azure AD. Así se mantiene una lista más reducida de roles con privilegios, en [Azure AD](../users-groups-roles/directory-assign-admin-roles.md) y en [Azure Resource](../../role-based-access-control/built-in-roles.md), y aumenta la seguridad general del directorio.

Las revisiones de acceso permiten a los revisores atestiguar si los usuarios todavía necesitan pertenecer a un rol. Al igual que sucede con las revisiones de acceso para los paquetes de acceso, las revisiones de roles en Azure AD y Azure Resource se integran en la experiencia de usuario del administrador de PIM. Se recomienda revisar periódicamente las siguientes asignaciones de roles:

* Administrador global

* Administrador de usuarios

* Administrador de autenticación con privilegios

* Administrador de acceso condicional

* Administrador de seguridad

* Todas las funciones de administración del servicio Dynamics y de Office 365

Los roles seleccionados aquí incluyen roles permanentes y elegibles. 

En la sección Revisores, seleccione una o más personas para que revisen a todos los usuarios. También puede seleccionar que los miembros revisen su propio acceso.

 ![Editar directiva](./media/deploy-access-review/7-plan-azure-resources-reviewers-selection.png)

## <a name="deploy-access-reviews"></a>Implementación de revisiones de acceso

Después de preparar una estrategia y un plan para revisar el acceso a los recursos integrados con Azure AD, implemente y administre las revisiones mediante los recursos que se incluyen a continuación.

### <a name="review-access-packages"></a>Revisión de paquetes de acceso

Para reducir el riesgo de acceso por parte de dispositivos obsoletos, los administradores pueden habilitar revisiones periódicas de los usuarios que tienen asignaciones activas en un paquete de acceso. Siga las instrucciones del vínculo que se incluye a continuación:

| Artículos de procedimientos| Descripción |
| - | - |
| [Crear revisiones de acceso](entitlement-management-access-reviews-create.md)| Muestra cómo revisar un paquete de acceso. |
| [Realizar revisiones de acceso](entitlement-management-access-reviews-review-access.md)| Indica cómo realizar revisiones de acceso para otros usuarios asignados a un paquete de acceso. |
| [Paquetes de acceso con autorrevisión asignada](entitlement-management-access-reviews-self-review.md)| Aborda los paquetes de acceso con autorrevisión asignada. |


> [!NOTE]
> Los usuarios finales que hagan autorrevisión e informen de que ya no necesitan acceso no se eliminarán del paquete de acceso inmediatamente. Se les quitará del paquete de acceso al finalizar la revisión o si un administrador detiene la revisión.

### <a name="review-groups-and-apps"></a>Revisión de grupos y aplicaciones

Las necesidades de acceso a grupos y aplicaciones para empleados e invitados probablemente cambie a lo largo del tiempo. Para reducir el riesgo asociado a las asignaciones de acceso obsoletas, los administradores pueden crear revisiones de acceso para los miembros de un grupo o para el acceso a aplicaciones. Siga las instrucciones del vínculo que se incluye a continuación:

| Artículos de procedimientos| Descripción |
| - | - |
| [Crear revisiones de acceso](create-access-review.md)| Muestra cómo crear una o varias revisiones de acceso para los miembros de un grupo o para el acceso de aplicaciones. |
| [Realizar revisiones de acceso](perform-access-review.md)| Indica cómo realizar revisiones de acceso para los miembros de un grupo o los usuarios con acceso a una aplicación. |
| [Autorrevisión del acceso](review-your-access.md)| Los miembros revisan su propio acceso a un grupo o una aplicación. |
| [Finalización de una revisión de acceso](complete-access-review.md)| Explica cómo ver una revisión de acceso y aplicar los resultados. |
| [Tomar medidas para los grupos locales](https://github.com/microsoft/access-reviews-samples/tree/master/AzureADAccessReviewsOnPremises)| Muestra un script de PowerShell de ejemplo para aplicar a las revisiones de acceso para grupos locales. |


### <a name="review-azure-ad-roles"></a>Revisión de roles de Azure AD

Para reducir el riesgo asociado a las asignaciones de roles obsoletas, debe revisar periódicamente el acceso a roles de Azure AD con privilegios.

![revisión de roles de Azure AD](./media/deploy-access-review/8-review-azure-ad-roles-picker.png)

Siga las instrucciones del vínculo que se incluye a continuación:

| Artículos de procedimientos | Descripción |
| - | - |
 [Crear revisiones del acceso](../privileged-identity-management/pim-how-to-start-security-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)| Muestra cómo crear revisiones de acceso para los roles de Azure AD con privilegios en PIM. |
| [Autorrevisión del acceso](../privileged-identity-management/pim-how-to-perform-security-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)| Si se le ha asignado un rol administrativo, apruebe o deniegue el acceso al rol. |
| [Finalización de una revisión de acceso](../privileged-identity-management/pim-how-to-complete-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)| Explica cómo ver una revisión de acceso y aplicar los resultados. |


### <a name="review-azure-resource-roles"></a>Revisión de roles de Azure Resource

Para reducir el riesgo asociado con las asignaciones de roles obsoletas, debe revisar periódicamente el acceso a roles de Azure Resource con privilegios. 

![revisión de roles de Azure AD](./media/deploy-access-review/9-review-azure-roles-picker.png)

Siga las instrucciones del vínculo que se incluye a continuación:

| Artículos de procedimientos| Descripción |
| - | -|
| [Crear revisiones del acceso](../privileged-identity-management/pim-resource-roles-start-access-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)| Señala cómo crear revisiones de acceso para roles de Azure Resource con privilegios en PIM. |
| [Autorrevisión del acceso](../privileged-identity-management/pim-resource-roles-perform-access-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)| Si se le ha asignado un rol administrativo, apruebe o deniegue el acceso al rol. |
| [Finalización de una revisión de acceso](../privileged-identity-management/pim-resource-roles-complete-access-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)| Explica cómo ver una revisión de acceso y aplicar los resultados. |


## <a name="use-the-access-reviews-api"></a>Uso de la API de revisiones de acceso

Consulte los [métodos de Graph API](/graph/api/resources/accessreviews-root?view=graph-rest-beta) y las [comprobaciones de autorización de roles y aplicaciones](/graph/api/resources/accessreviews-root?view=graph-rest-beta) para interactuar con los recursos revisables y administrarlos. Los métodos de revisión de acceso de Microsoft Graph API están disponibles para los contextos de aplicación y de usuario. Al ejecutar scripts en el contexto de aplicación, se debe conceder el permiso "AccessReview.Read.All" a la cuenta utilizada para ejecutar la API (la entidad de servicio) para consultar la información sobre revisiones de acceso.

Algunas tareas populares para automatizar las revisiones de acceso mediante Graph API son:

* Crear e iniciar de una revisión de acceso.

* Finalizar manualmente una revisión de acceso antes de la finalización programada.

* Enumerar todas las revisiones de acceso en ejecución e indicar su estado.

* Ver el historial de una serie de revisiones, junto con las decisiones y las medidas tomadas en cada revisión.

* Recopilar las decisiones de una revisión de acceso.

* Recopilar las decisiones de revisiones completadas en las que el revisor tomó una decisión distinta a la recomendada por el sistema.

Al crear nuevas consultas de Graph API con fines de automatización, se recomienda usar el [Probador de Graph](https://developer.microsoft.com/en-us/graph/graph-explorer). Puede compilar y explorar las consultas de Graph antes de incluirlas en scripts y código. Esto puede ayudarle a iterar rápidamente la consulta para que obtenga exactamente los resultados que espera, sin cambiar el código del script.

## <a name="monitor-access-reviews"></a>Supervisión de revisiones de acceso

Las actividades de revisión de acceso se registran y están disponibles en los [registros de auditoría de Azure AD](../reports-monitoring/concept-audit-logs.md). Puede filtrar los datos de auditoría según la categoría, el tipo de actividad y el intervalo de fechas. Esta es una consulta de ejemplo:

| Category| Directiva |
| - | - |
| Tipo de actividad| Creación de revisión de acceso |
| | Actualizar revisión de acceso |
| | Revisión de acceso finalizada |
| | Eliminación de revisión de acceso |
| | Aprobar decisión |
| | Denegar decisión |
| | Restablecer decisión |
| | Aplicar decisión |
| Intervalo de fechas| siete días |


Para consultas y análisis más avanzados de las revisiones de acceso, y para hacer un seguimiento de los cambios y la finalización de las revisiones, se recomienda que exporte los registros de auditoría de Azure AD a [Azure Log Analytics](../reports-monitoring/quickstart-azure-monitor-route-logs-to-storage-account.md) o Azure Event Hub. Cuando las consultas se almacenan en Azure Log Analytics, es posible usar el [lenguaje de análisis eficaz](../reports-monitoring/howto-analyze-activity-logs-log-analytics.md) y crear sus propios paneles.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre las siguientes tecnologías relacionadas.

* [¿Qué es la administración de derechos de Azure AD?](entitlement-management-overview.md)

* [¿Qué es Azure AD Privileged Identity Management?](../privileged-identity-management/pim-configure.md)

