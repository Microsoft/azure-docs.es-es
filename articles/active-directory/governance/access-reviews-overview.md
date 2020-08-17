---
title: ¿Qué son las revisiones de acceso? Azure Active Directory | Microsoft Docs
description: Con las revisiones de acceso de Azure Active Directory, puede controlar la pertenencia a grupos y el acceso a las aplicaciones para cumplir con las iniciativas de cumplimiento, de administración de riesgos y de gobernanza de su organización.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.subservice: compliance
ms.date: 07/22/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: ec88d795d0e05c62f07ff415364ced651ad8f4bc
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87034619"
---
# <a name="what-are-azure-ad-access-reviews"></a>¿Qué son las revisiones de acceso de Azure AD?

Las revisiones de acceso de Azure Active Directory (Azure AD) permiten a las organizaciones administrar de forma eficiente la pertenencia a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

Este es un vídeo que proporciona una introducción rápida de las revisiones de acceso:

>[!VIDEO https://www.youtube.com/embed/kDRjQQ22Wkk]

## <a name="why-are-access-reviews-important"></a>¿Por qué son importantes las revisiones de acceso?

Azure AD le permite colaborar internamente dentro de su organización y con usuarios de organizaciones externas, como los asociados. Los usuarios pueden unirse a grupos, invitar a otros usuarios, conectarse a aplicaciones en la nube y trabajar de forma remota desde sus dispositivos de trabajo o personales. La comodidad de aprovechar el potencial del autoservicio ha llevado a una necesidad de mejores funcionalidades de administración del acceso.

- Cuando se unen nuevos empleados, ¿cómo se asegura de que tengan el acceso adecuado para que sean productivos?
- A medida que las personas cambian de equipo o abandonan la empresa, ¿cómo se asegura de que se quite su antiguo acceso, especialmente cuando hay invitados involucrados?
- Los derechos de acceso excesivos pueden provocar malos resultados en las auditorías y riesgos, ya que indican una falta de control sobre el acceso.
- Tendrá que trabajar de manera proactiva junto a los propietarios de los recursos para asegurarse de que revisen periódicamente quién tiene acceso a sus recursos.

## <a name="when-to-use-access-reviews"></a>¿Cuándo usar revisiones de acceso?

- **Demasiados usuarios en roles con privilegios:** es recomendable comprobar cuántos usuarios tienen acceso administrativo, cuántos de ellos son administradores globales, y si hay algún invitado o asociado que no se haya quitado después de que se haya asignado una tarea administrativa. Puede volver a certificar los usuarios con asignación de roles en [roles de Azure AD](../privileged-identity-management/pim-how-to-perform-security-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json), por ejemplo, administradores globales, o [roles de recursos de Azure](../privileged-identity-management/pim-resource-roles-perform-access-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json), por ejemplo, administrador de acceso de usuario, en la experiencia [Azure AD Privileged Identity Management (PIM)](../privileged-identity-management/pim-configure.md).
- **Cuando no es factible la automatización:** puede crear las reglas de pertenencia dinámica en grupos de seguridad o Grupos de Office 365, pero ¿qué ocurre si los datos de recursos humanos no se encuentran en Azure AD o si los usuarios todavía necesitan acceso después de abandonar el grupo para entrenar a su sustituto? Luego puede crear una revisión en ese grupo para asegurarse de que los usuarios que aún necesiten acceso sigan teniendo acceso.
- **Cuando se usa un grupo para una nueva finalidad:** si tiene un grupo que se va a sincronizar con Azure AD, o si va a habilitar la aplicación Salesforce para todos los usuarios del equipo de Ventas, sería útil solicitar al propietario del grupo que revise la pertenencia al grupo antes de usar el grupo en un contenido de riesgo distinto.
- **Acceso a datos críticos para la empresa:** para determinados recursos y con fines de auditoría, podría exigirse a personas ajenas a TI que cierren sesión periódicamente y justifiquen por qué necesitan acceso.
- **Para mantener la lista de excepciones de la directiva:** En un mundo ideal, todos los usuarios deberían seguir las directivas de acceso para proteger el acceso a los recursos de la organización. A veces, sin embargo, hay casos empresariales en los que hay que hacer excepciones. Como administrador de TI, puede administrar esta tarea, evitar las excepciones de omisiones de la directiva y proporcionar a los auditores prueba de que estas excepciones se revisan normalmente.
- **Pedir a los propietarios de grupos que confirmen que todavía necesitan invitados en sus grupos:** el acceso de los empleados se puede automatizar con algún tipo de IAM a nivel local, pero no los usuarios invitados. Si un grupo proporciona a los invitados acceso a contenido empresarial confidencial, es responsabilidad del propietario del grupo confirmar que los invitados todavía tienen una necesidad empresarial legítima de acceso.
- **Repetir las revisiones periódicamente:** puede configurar revisiones periódicas de acceso de usuarios a frecuencias establecidas, como semanal, mensual, trimestral o anualmente, donde los revisores reciban notificaciones al principio de cada revisión. Los revisores pueden aprobar o denegar el acceso con una interfaz sencilla y con la ayuda de recomendaciones inteligentes.

## <a name="where-do-you-create-reviews"></a>¿Donde se crean las revisiones?

Dependiendo de lo que quiera revisar, creará la revisión de acceso en Revisiones de acceso de Azure AD, aplicaciones de empresa de Azure AD (en versión preliminar) o Azure AD PIM.

| Derechos de acceso de los usuarios | Los revisores pueden ser | Revisión creada en | Experiencia del revisor |
| --- | --- | --- | --- |
| Miembros del grupo de seguridad</br>Miembros del grupo de Office | Revisores especificados</br>Propietarios del grupo</br>Autorrevisión | Revisiones de acceso de Azure AD</br>Grupos de Azure AD | Panel de acceso |
| Asignados a una aplicación conectada | Revisores especificados</br>Autorrevisión | Revisiones de acceso de Azure AD</br>Aplicaciones de empresa de Azure AD (en versión preliminar) | Panel de acceso |
| Rol de Azure AD | Revisores especificados</br>Autorrevisión | [Azure AD PIM](../privileged-identity-management/pim-how-to-start-security-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json) | Azure portal |
| Rol de recursos de Azure | Revisores especificados</br>Autorrevisión | [Azure AD PIM](../privileged-identity-management/pim-resource-roles-start-access-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json) | Azure portal |


## <a name="create-access-reviews"></a>Crear revisiones del acceso

Para crear una revisión de acceso, siga estos pasos:

1. Diríjase a [Azure Portal](https://portal.azure.com) para administrar las revisiones de acceso e inicie sesión como administrador global o administrador de usuarios.

1. Busque y seleccione **Azure Active Directory**.

      ![Búsqueda de Azure Portal para Azure Active Directory](media/access-reviews-overview/search-azure-active-directory.png)

1. Seleccione **Identity Governance**.

1. En la página Introducción, haga clic en el botón **Crear una revisión de acceso**.

   ![Página de inicio de revisiones de acceso](./media/access-reviews-overview/access-reviews-overview-create-access-reviews.png) 

### <a name="creating-access-review-on-a-group-that-can-be-assigned-to-azure-ad-role"></a>Creación de una revisión de acceso en un grupo que se puede asignar al rol de Azure AD
Si está en la versión más reciente de las revisiones de acceso (se dirige a sus revisores a **Mi acceso** de forma predeterminada), solo el administrador global puede crear una revisión de acceso en grupos asignables del rol. Sin embargo, si está en una versión anterior de las revisiones de acceso (se dirige a sus revisores al **Panel de acceso** de forma predeterminada), tanto el administrador global como el administrador de usuarios pueden crear una revisión de acceso en grupos asignables del rol.  

La nueva experiencia se implementará en todos los clientes el 1 de agosto de 2020, pero si desea actualizar antes, realice una solicitud en la página de las [revisiones de acceso de Azure AD sobre la experiencia del revisor actualizada en la suscripción a Mi acceso](https://forms.microsoft.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR5dv-S62099HtxdeKIcgO-NUOFJaRDFDWUpHRk8zQ1BWVU1MMTcyQ1FFUi4u).

[Obtenga más información sobre la asignación de grupos a los roles de Azure AD](https://go.microsoft.com/fwlink/?linkid=2103037).

## <a name="learn-about-access-reviews"></a>Más información sobre las revisiones de acceso

Para obtener más información sobre cómo crear y realizar las revisiones de acceso, vea esta breve demostración:

>[!VIDEO https://www.youtube.com/embed/6KB3TZ8Wi40]

Si está listo para implementar revisiones de acceso en su organización, siga estos pasos en el vídeo para incorporar y entrenar a los administradores, y crear su primera revisión de acceso.

>[!VIDEO https://www.youtube.com/embed/X1SL2uubx9M]

## <a name="license-requirements"></a>Requisitos de licencia

[!INCLUDE [Azure AD Premium P2 license](../../../includes/active-directory-p2-license.md)]

### <a name="how-many-licenses-must-you-have"></a>¿Cuántas licencias debe tener?

Asegúrese de que el directorio tiene al menos tantas licencias de Azure AD Premium P2 como empleados que van a realizar las tareas siguientes:

- Usuarios miembros e invitados asignados como revisores
- Usuarios miembros e invitados que realizan una autorrevisión
- Propietarios de grupos que realizan una revisión de acceso
- Propietarios de aplicaciones que realizan una revisión de acceso

Las licencias de Azure AD Premium P2 **no** son necesarias para las tareas siguientes:

- No se necesitan licencias para los usuarios con los roles Administrador global o Administrador de usuarios que configuran revisiones de acceso, configuran opciones o aplican las decisiones a partir de las revisiones.

Por cada licencia de Azure AD Premium P2 de pago que asigne a uno de los usuarios de la organización, puede usar la colaboración negocio a negocio (B2B) Azure AD para invitar hasta a cinco usuarios, gracias a la concesión de usuarios externos. Los usuarios invitados también pueden usar las características de Azure AD Premium P2. Para más información, consulte [Guía de concesión de licencias de colaboración B2B de Azure Active Directory](../b2b/licensing-guidance.md).

Para más información sobre las licencias, consulte [Asignación o eliminación de licencias mediante el portal de Azure Active Directory](../fundamentals/license-users-groups.md).

### <a name="example-license-scenarios"></a>Escenarios de licencia de ejemplo

Estos son algunos escenarios de licencia de ejemplo que le ayudarán a determinar el número de licencias que debe tener.

| Escenario | Cálculo | Número de licencias |
| --- | --- | --- |
| Un administrador crea una revisión de acceso del Grupo A con 75 usuarios y 1 propietario del grupo, y asigna el propietario del grupo como revisor. | 1 licencia para el propietario del grupo como revisor | 1 |
| Un administrador crea una revisión de acceso del Grupo B con 500 usuarios y 3 propietarios de grupo, y asigna los 3 propietarios de grupo como revisores. | 3 licencias para cada propietario de grupo como revisores | 3 |
| Un administrador crea una revisión de acceso del Grupo B con 500 usuarios. Realiza una autorrevisión. | 500 licencias para cada usuario como revisores | 500 |
| Un administrador crea una revisión de acceso del Grupo C con 50 usuarios miembros y 25 usuarios invitados. Realiza una autorrevisión. | 50 licencias para cada usuario como revisores<br/>(los usuarios invitados están cubiertos en la proporción requerida de 1:5) | 50 |
| Un administrador crea una revisión de acceso del Grupo D con 6 usuarios miembros y 108 usuarios invitados. Realiza una autorrevisión. | 6 licencias para cada usuario como autorrevisores + 16 licencias adicionales para abarcar a los 108 usuarios invitados de la proporción de 1:5 necesaria. 6 licencias, que suponen 6\*5=30 usuarios invitados. Para los (108-6\*5)=78 usuarios invitados restantes, se necesitan 78/5=16 licencias adicionales. Por lo tanto, en total, se necesitan 6 + 16 = 22 licencias. | 22 |

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una revisión de acceso de grupos o aplicaciones](create-access-review.md)
- [Creación de una revisión de acceso de los usuarios en un rol administrativo de Azure AD](../privileged-identity-management/pim-how-to-start-security-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)
- [Revisión del acceso a grupos o aplicaciones](perform-access-review.md)
- [Completar una revisión de acceso de grupos o aplicaciones](complete-access-review.md)
