---
title: ¿Qué es la administración de derechos? - Azure AD
description: Obtenga información general de la administración de derechos de Azure Active Directory y cómo puede utilizarla para administrar el acceso a grupos, aplicaciones y sitios de SharePoint Online para usuarios internos y externos.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.subservice: compliance
ms.date: 09/08/2020
ms.author: barclayn
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
ms.custom: contperfq1
ms.openlocfilehash: 2ce5f0c9b340e1b81fcf002ee70e7ec1cc74d38c
ms.sourcegitcommit: 1b320bc7863707a07e98644fbaed9faa0108da97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89594329"
---
# <a name="what-is-azure-ad-entitlement-management"></a>¿Qué es la administración de derechos de Azure AD?

La administración de derechos de Azure Active Directory (Azure AD) es una característica de [control de identidad](identity-governance-overview.md) que permite a las organizaciones administrar el ciclo de vida de identidad y acceso a escala, mediante la automatización de los flujos de trabajo de solicitud de acceso, las asignaciones de acceso, las revisiones y la expiración.

Los empleados de las organizaciones necesitan tener acceso a varios grupos, aplicaciones y sitios para realizar su trabajo. La administración de este acceso es complicada, dado que los requisitos cambian, se agregan nuevas aplicaciones o los usuarios necesitan derechos de acceso adicionales.  Este escenario se complica si se colabora con organizaciones externas: puede que no sepa qué usuarios de la otra organización necesitan acceder a los recursos de su organización y ellos no sabrán qué aplicaciones, grupos o sitios usa esta.

La administración de derechos de Azure AD puede ayudarle a administrar de manera más eficiente el acceso a grupos, aplicaciones y sitios de SharePoint Online para usuarios internos y también para usuarios de fuera de la organización que necesitan acceso a esos recursos.

## <a name="why-use-entitlement-management"></a>¿Qué es la administración de derechos?

Las organizaciones empresariales a menudo se enfrentan a desafíos a la hora de administrar el acceso de los empleados a recursos tales como:

- Es posible que los usuarios no sepan qué acceso deben tener y, incluso si lo saben, pueden tener dificultades para encontrar los usuarios adecuados que aprueben su acceso.
- Una vez que los usuarios buscan y reciben acceso a un recurso, pueden retener el acceso por más tiempo del necesario para fines empresariales.

Estos problemas se agravan para los usuarios que necesitan acceso desde otra organización, como los usuarios externos que pertenecen a organizaciones de la cadena de suministro u otros asociados comerciales. Por ejemplo:

- Es posible que nadie conozca a todas las personas de los directorios de otra organización para poder invitarlos.
- Incluso si las organizaciones pudieran invitar a estos usuarios, es posible que nadie de esa organización se acuerde de administrar todo el acceso de los usuarios de forma coherente.

La administración de derechos de Azure AD puede ayudar a abordar estos desafíos.  Para más información sobre cómo los clientes han estado usando la administración de derechos de Azure AD, puede leer el [caso de estudio de Avanade](https://customers.microsoft.com/story/avanade-professional-services-azure-canada) y el [caso de estudio de Centrica](https://customers.microsoft.com/story/757467-centrica-energy-azure).  Este vídeo proporciona información general sobre la administración de derechos y su valor:

>[!VIDEO https://www.youtube.com/embed/_Lss6bFrnQ8]

## <a name="what-can-i-do-with-entitlement-management"></a>¿Qué se puede hacer con la administración de derechos?

Estas son algunas de las funcionalidades de la administración de derechos:

- Delegue a los usuarios que no son administradores la posibilidad de crear paquetes de acceso. Estos paquetes de acceso contienen recursos que los usuarios pueden solicitar, y los administradores de paquetes de acceso delegados pueden definir directivas con reglas sobre lo que los usuarios pueden solicitar, quién debe aprobar su acceso y cuándo expira este.
- Seleccione las organizaciones conectadas cuyos usuarios pueden solicitar acceso.  Cuando un usuario que todavía no está en su directorio solicita acceso y este acceso se aprueba, se le invita automáticamente al directorio y se le asigna acceso.  Cuando expira su acceso, si no tiene otras asignaciones de paquete de acceso, su cuenta de B2B en el directorio se puede quitar automáticamente.

>[!NOTE]
>Si está preparado para probar la administración de derechos, puede empezar a trabajar con el [tutorial para crear el primer paquete de acceso](entitlement-management-access-package-first.md).

También puede leer los [escenarios comunes](entitlement-management-scenarios.md) o ver los vídeos siguientes:

- [Implementación de la administración de derechos de Azure AD en su organización](https://www.youtube.com/watch?v=zaaKvaaYwI4)
- [Supervisión y escala del uso de la administración de derechos de Azure AD](https://www.youtube.com/watch?v=omtNJ7ySjS0)
- [Delegación en la administración de derechos](https://www.youtube.com/watch?v=Fmp1eBxzrqw)

## <a name="what-are-access-packages-and-what-resources-can-i-manage-with-them"></a>¿Qué son los paquetes de acceso y qué recursos puedo administrar con ellos?

La administración de derechos introduce en Azure AD el concepto de un *paquete de acceso*. Un paquete de acceso es una agrupación de todos los recursos con el acceso que necesita un usuario para trabajar en un proyecto o realizar su tarea. Los paquetes de acceso se utilizan para controlar el acceso de los empleados internos y también de los usuarios de fuera de la organización.

 Estos son los tipos de recursos para los que puede administrar el acceso del usuario con la administración de derechos:

- Pertenencia a grupos de seguridad de Azure AD
- Pertenencia a Teams y Grupos de Microsoft 365
- Asignación a aplicaciones empresariales de Azure AD, incluidas las aplicaciones SaaS y las aplicaciones integradas personalizadas que admitan la federación o el inicio de sesión único o el aprovisionamiento
- Pertenencia a sitios de SharePoint Online

También puede controlar el acceso a otros recursos que dependen de los grupos de seguridad de Azure AD o de Grupos de Microsoft 365.  Por ejemplo:

- Puede conceder licencias a los usuarios para Microsoft 365 mediante un grupo de seguridad de Azure AD en un paquete de acceso y la configuración de [licencias basadas en grupos](../users-groups-roles/licensing-groups-assign.md) para ese grupo.
- Puede dar acceso a los usuarios para administrar los recursos de Azure mediante un grupo de seguridad de Azure AD en un paquete de acceso y la creación de una [asignación de roles de Azure](../../role-based-access-control/role-assignments-portal.md) para ese grupo.

## <a name="how-do-i-control-who-gets-access"></a>¿Cómo se controla quién tiene acceso?

Con un paquete de acceso, un administrador o un administrador de paquetes de acceso delegado enumera los recursos (grupos, aplicaciones y sitios) y los roles que los usuarios necesitan para esos recursos.

Los paquetes de acceso también incluyen una o varias *directivas*. Una directiva define las reglas o barreras para la asignación al paquete de acceso. Cada directiva puede usarse para garantizar que solo los usuarios adecuados puedan solicitar acceso, que haya aprobadores para su solicitud y que el acceso a esos recursos sea por tiempo limitado y expire si no se ha renovado.

![Paquete de acceso y directivas](./media/entitlement-management-overview/elm-overview-access-package.png)

Dentro de cada directiva, un administrador o el administrador de paquetes de acceso define:

- Los usuarios existentes (normalmente empleados o invitados) o las organizaciones asociadas de usuarios externos, que pueden solicitar acceso
- El proceso de aprobación y los usuarios que pueden aprobar o denegar el acceso
- La duración de la asignación del acceso de un usuario, una vez aprobado, antes de que expire la asignación.

El siguiente diagrama muestra un ejemplo de los diferentes elementos de la administración de derechos. Aparece un catálogo con dos paquetes de acceso de ejemplo.

- El **paquete de acceso 1** incluye un único grupo como un recurso. El acceso se define con una directiva que permite a un conjunto de usuarios del directorio solicitar acceso.
- El **paquete de acceso 2** incluye un grupo, una aplicación y un sitio de SharePoint Online como recursos. El acceso se define con dos directivas diferentes. La primera directiva permite a un conjunto de usuarios del directorio solicitar acceso. La segunda directiva permite a los usuarios de un directorio externo solicitar acceso.

![Introducción a la administración de derechos](./media/entitlement-management-overview/elm-overview.png)

## <a name="when-should-i-use-access-packages"></a>¿Cuándo debo usar paquetes de acceso?

Los paquetes de acceso no reemplazan a otros mecanismos de asignación de acceso.  Son más adecuadas en situaciones como las siguientes:

- Los empleados necesitan acceso por tiempo limitado para una tarea determinada.  Por ejemplo, podría usar licencias basadas en grupos y un grupo dinámico para asegurarse de que todos los empleados tengan un buzón de Exchange Online y usar luego los paquetes de acceso en situaciones en las que los empleados necesiten acceso adicional, por ejemplo, para leer los recursos de un departamento desde otro departamento.
- Acceso que requiera la aprobación del administrador de un empleado o de otras personas designadas.
- Los departamentos desean administrar sus propias directivas de acceso a los recursos sin la intervención del departamento de TI.  
- Dos o más organizaciones colaboran en un proyecto y, como resultado, varios usuarios de una organización deberán incorporarse mediante Azure AD B2B para tener acceso a los recursos de la otra organización.

## <a name="how-do-i-delegate-access"></a>¿Cómo se delega el acceso?

 Los paquetes de acceso se definen en contenedores llamados *catálogos*.  Puede tener un único catálogo para todos los paquetes de acceso o puede designar a personas para que creen o posean sus propios catálogos. Un administrador puede agregar recursos a cualquier catálogo, pero una persona que no sea administrador solo puede agregar a un catálogo los recursos que posea. El propietario de un catálogo puede agregar a otros usuarios como copropietarios del catálogo o como administradores de paquetes de acceso.  Estos escenarios se describen con más detalle en el artículo [Delegación y roles en la administración de derechos de Azure AD](entitlement-management-delegate.md).

## <a name="summary-of-terminology"></a>Resumen de la terminología

Para comprender mejor la administración de derechos y su documentación, puede consultar la siguiente lista de términos.

| Término | Descripción |
| --- | --- |
| paquete de acceso | Conjunto de recursos que un equipo o proyecto necesita y se rige por directivas. Un paquete de acceso siempre se encuentra en un catálogo. Un paquete de acceso se crearía en un escenario en el que los usuarios necesiten solicitar acceso.  |
| solicitud de acceso | Solicitud para acceder a los recursos de un paquete de acceso. Una solicitud suele pasa por un flujo de trabajo de aprobación.  Si se aprueba, el usuario solicitante recibe una asignación de paquete de acceso. |
| asignación | La asignación de un paquete de acceso a un usuario garantiza que el usuario tenga todos los roles de recursos de ese paquete de acceso.  Las asignaciones de paquetes de acceso suelen tener un límite de tiempo antes de que expiren. |
| catalog | Un contenedor de recursos relacionados y paquetes de acceso.  Los catálogos se emplean en la delegación, de modo que las personas que no son administradores pueden crear sus propios paquetes de acceso. Los propietarios de catálogos pueden agregar recursos de su propiedad a un catálogo. |
| creador de catálogos | Una colección de usuarios que están autorizados para crear catálogos.  Cuando un usuario que no es administrador y que está autorizado para ser un creador de catálogos crea un catálogo, se convierte automáticamente en el propietario de dicho catálogo. |
| organización conectada | Un directorio o dominio externo de Azure AD con el que tiene una relación. Los usuarios de una organización conectada se pueden especificar en una directiva como que tienen permiso para solicitar acceso. |
| policy | Un conjunto de reglas que define el ciclo de vida del acceso, como por ejemplo, cómo los usuarios obtienen acceso, quién puede aprobarlo y cuánto tiempo tienen acceso mediante una asignación. Una directiva está vinculada a un paquete de acceso. Por ejemplo, un paquete de acceso podría tener dos directivas: una para que los empleados soliciten acceso y una segunda para que lo hagan los usuarios externos. |
| resource | Un recurso, como un grupo de Office, un grupo de seguridad, una aplicación o un sitio de SharePoint Online, con un rol para el que se puede conceder permisos a un usuario. |
| directorio de recursos | Un directorio que tiene uno o más recursos para compartir. |
| rol de recurso | Una colección de permisos asociados a un recurso y definidos por él. Un grupo tiene dos roles: miembro y propietario. Los sitios de SharePoint suelen tener tres roles, pero pueden tener roles personalizados adicionales. Las aplicaciones pueden tener roles personalizados. |


## <a name="license-requirements"></a>Requisitos de licencia

[!INCLUDE [Azure AD Premium P2 license](../../../includes/active-directory-p2-license.md)]

Las nubes especializadas, como Azure Alemania y Azure China 21Vianet, no están actualmente disponibles para su uso.

### <a name="how-many-licenses-must-you-have"></a>¿Cuántas licencias debe tener?

Asegúrese de que el directorio tenga al menos tantas licencias de Azure AD Premium P2 como usted:

- Usuarios miembros que **pueden** solicitar un paquete de acceso.
- Usuarios miembros e invitados que solicitan un paquete de acceso.
- Usuarios miembros e invitados que aprueban solicitudes para un paquete de acceso.
- Usuarios miembros e invitados que tengan una asignación directa a un paquete de acceso.

Las licencias de Azure AD Premium P2 **no** son necesarias para las tareas siguientes:

- No se requiere ninguna licencia para usuarios con el rol de administrador global que configuran los catálogos iniciales, paquetes de acceso y directivas, y delegan tareas administrativas en otros usuarios.
- No se requiere ninguna licencia para usuarios en los que se han delegado tareas administrativas, como el creador de catálogos, el propietario de catálogos y el administrador de paquetes de acceso.
- No se requiere ninguna licencia para invitados que **pueden** solicitar paquetes de acceso, pero **no** solicitan un paquete de acceso.

Los precios de Azure AD External Identities (usuario invitado) se basan en los usuarios activos mensuales (MAU), que es el recuento de usuarios únicos con actividad de autenticación en un mes natural. Este modelo reemplaza el modelo de facturación con una relación 1:5, que permitía hasta cinco usuarios invitados para cada licencia Premium de Azure AD del inquilino. Cuando el inquilino está vinculado a una suscripción y usa las características de identidades externas para colaborar con los usuarios invitados, se le facturará automáticamente mediante el modelo de facturación basado en MAU. Para más información, consulte el modelo de facturación para [Azure AD External Identities](../external-identities/external-identities-pricing.md).

Para más información sobre las licencias, consulte [Asignación o eliminación de licencias mediante el portal de Azure Active Directory](../fundamentals/license-users-groups.md).

### <a name="example-license-scenarios"></a>Escenarios de licencia de ejemplo

Estos son algunos escenarios de licencia de ejemplo que le ayudarán a determinar el número de licencias que debe tener.

| Escenario | Cálculo | Número de licencias |
| --- | --- | --- |
| Un administrador global de Woodgrove Bank crea catálogos iniciales y delega tareas administrativas en otros seis usuarios. Una de las directivas especifica que **todos los empleados** (2000 empleados) pueden solicitar un conjunto específico de paquetes de acceso. 150 empleados solicitan los paquetes de acceso. | 2000 empleados que **pueden** solicitar los paquetes de acceso | 2\.000 |
| Un administrador global de Woodgrove Bank crea catálogos iniciales y delega tareas administrativas en otros seis usuarios. Una de las directivas especifica que **todos los empleados** (2000 empleados) pueden solicitar un conjunto específico de paquetes de acceso. Otra directiva especifica que algunos **usuarios del asociado Contoso** (invitados) pueden solicitar los mismos paquetes de acceso sujetos a aprobación. Contoso tiene 30 000 usuarios. 150 empleados solicitan los paquetes de acceso y 10 500 usuarios de Contoso solicitan acceso. | 2000 empleados + 500 usuarios invitados de Contoso que superan la relación 1:5 (10 500 - (2000 * 5)) | 2,500 |

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Creación del primer paquete de acceso](entitlement-management-access-package-first.md)
- [Escenarios comunes](entitlement-management-scenarios.md)