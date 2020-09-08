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
ms.date: 08/18/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.custom: contperfq1
ms.openlocfilehash: c69a5e153377eee86eaf0c43d6c982dee2938ddf
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783678"
---
# <a name="what-are-azure-ad-access-reviews"></a>¿Qué son las revisiones de acceso de Azure AD?

Las revisiones de acceso de Azure Active Directory (Azure AD) permiten a las organizaciones administrar de forma eficiente la pertenencia a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

Este es un vídeo que proporciona una introducción rápida de las revisiones de acceso:

>[!VIDEO https://www.youtube.com/embed/kDRjQQ22Wkk]

## <a name="why-are-access-reviews-important"></a>¿Por qué son importantes las revisiones de acceso?

Azure AD le permite colaborar con usuarios de dentro y fuera de la organización. Los usuarios pueden unirse a grupos, invitar a otros usuarios, conectarse a aplicaciones en la nube y trabajar de forma remota desde sus dispositivos de trabajo o personales. La comodidad de usar el autoservicio ha llevado a una necesidad de mejores funcionalidades de administración del acceso.

- Cuando se unen nuevos empleados, ¿cómo estar seguro de que tienen el acceso que necesitan para ser productivos?
- A medida que los usuarios salen de equipos o abandonan la empresa, ¿cómo tener la seguridad de que se les retira su acceso antiguo?
- Unos derechos de acceso excesivos son peligrosos.
- También pueden llevar a auditorías al indicar una falta de control sobre el acceso.
- Tendrá que trabajar de manera proactiva junto a los propietarios de los recursos para asegurarse de que revisen periódicamente quién tiene acceso a sus recursos.

## <a name="when-should-you-use-access-reviews"></a>¿Cuándo se deben usar las revisiones de acceso?

- **Demasiados usuarios en roles con privilegios:** es recomendable comprobar cuántos usuarios tienen acceso administrativo, cuántos de ellos son administradores globales, y si hay algún invitado o asociado que no se haya quitado después de que se haya asignado una tarea administrativa. Puede volver a certificar los usuarios con asignación de roles en [roles de Azure AD](../privileged-identity-management/pim-how-to-perform-security-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json), por ejemplo, administradores globales, o [roles de recursos de Azure](../privileged-identity-management/pim-resource-roles-perform-access-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json), por ejemplo, administrador de acceso de usuario, en la experiencia [Azure AD Privileged Identity Management (PIM)](../privileged-identity-management/pim-configure.md).
- **Cuando no es posible la automatización:** puede crear las reglas de pertenencia dinámica en grupos de seguridad o Grupos de Office 365, pero ¿qué ocurre si los datos de recursos humanos no se encuentran en Azure AD o si los usuarios todavía necesitan acceso después de abandonar el grupo para entrenar a su sustituto? Luego puede crear una revisión en ese grupo para asegurarse de que los usuarios que aún necesiten acceso sigan teniendo acceso.
- **Cuando se usa un grupo para una nueva finalidad:** si tiene un grupo que se va a sincronizar con Azure AD, o si va a habilitar la aplicación Salesforce para todos los usuarios del equipo de Ventas, sería útil solicitar al propietario del grupo que revise la pertenencia al grupo antes de usar el grupo en un contenido de riesgo distinto.
- **Acceso a datos críticos para la empresa:** para determinados recursos y con fines de auditoría, podría exigirse a personas ajenas a TI que cierren sesión periódicamente y justifiquen por qué necesitan acceso.
- **Para mantener la lista de excepciones de la directiva:** En un mundo ideal, todos los usuarios deberían seguir las directivas de acceso para proteger el acceso a los recursos de la organización. A veces, sin embargo, hay casos empresariales en los que hay que hacer excepciones. Como administrador de TI, puede administrar esta tarea, evitar las excepciones de omisiones de la directiva y proporcionar a los auditores prueba de que estas excepciones se revisan normalmente.
- **Pedir a los propietarios de grupos que confirmen que todavía necesitan invitados en sus grupos:** el acceso de los empleados se podría automatizar con algún tipo de Administración de identidad y acceso (IAM) a nivel local, pero no el de los usuarios invitados. Si un grupo proporciona a los invitados acceso a contenido empresarial confidencial, es responsabilidad del propietario del grupo confirmar que los invitados todavía tienen una necesidad empresarial legítima de acceso.
- **Repetir las revisiones periódicamente:** puede configurar revisiones periódicas de acceso de usuarios a frecuencias establecidas, como semanal, mensual, trimestral o anualmente, donde los revisores reciban notificaciones al principio de cada revisión. Los revisores pueden aprobar o denegar el acceso con una interfaz sencilla y con la ayuda de recomendaciones inteligentes.

>[!NOTE]
>Si está listo para probar las revisiones de acceso, eche un vistazo a [Creación de una revisión de acceso de grupos o aplicaciones](create-access-review.md).

## <a name="where-do-you-create-reviews"></a>¿Donde se crean las revisiones?

Dependiendo de lo que quiera revisar, creará la revisión de acceso en Revisiones de acceso de Azure AD, aplicaciones de empresa de Azure AD (en versión preliminar) o Azure AD PIM.

| Derechos de acceso de los usuarios | Los revisores pueden ser | Revisión creada en | Experiencia del revisor |
| --- | --- | --- | --- |
| Miembros del grupo de seguridad</br>Miembros del grupo de Office | Revisores especificados</br>Propietarios del grupo</br>Autorrevisión | Revisiones de acceso de Azure AD</br>Grupos de Azure AD | Panel de acceso |
| Asignados a una aplicación conectada | Revisores especificados</br>Autorrevisión | Revisiones de acceso de Azure AD</br>Aplicaciones de empresa de Azure AD (en versión preliminar) | Panel de acceso |
| Rol de Azure AD | Revisores especificados</br>Autorrevisión | [Azure AD PIM](../privileged-identity-management/pim-how-to-start-security-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json) | Azure portal |
| Rol de recursos de Azure | Revisores especificados</br>Autorrevisión | [Azure AD PIM](../privileged-identity-management/pim-resource-roles-start-access-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json) | Azure portal |

## <a name="license-requirements"></a>Requisitos de licencia

[!INCLUDE [Azure AD Premium P2 license](../../../includes/active-directory-p2-license.md)]

### <a name="how-many-licenses-must-you-have"></a>¿Cuántas licencias debe tener?

El directorio debe tener al menos tantas licencias de Azure AD Premium P2 como número de empleados que vayan a realizar las siguientes tareas:

- Usuarios miembros e invitados asignados como revisores
- Usuarios miembros e invitados que realizan una autorrevisión
- Propietarios de grupos que realizan una revisión de acceso
- Propietarios de aplicaciones que realizan una revisión de acceso

Las licencias de Azure AD Premium P2 **no** son necesarias para los usuarios con los roles Administrador global o Administrador de usuarios que configuran revisiones de acceso, realizan configuraciones o aplican las decisiones a partir de los resultados de las revisiones.

Por cada licencia de Azure AD Premium P2 de pago que asigne a uno de los usuarios de la organización, puede usar la colaboración negocio a negocio (B2B) Azure AD para invitar hasta a cinco usuarios, gracias a la concesión de usuarios externos. Los usuarios invitados también pueden usar las características de Azure AD Premium P2. Para más información, consulte [Guía de concesión de licencias de colaboración B2B de Azure Active Directory](../external-identities/licensing-guidance.md).

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