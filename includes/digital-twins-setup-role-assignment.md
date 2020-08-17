---
author: baanders
description: archivo de inclusión para el paso de permisos de acceso en la configuración de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 7/17/2020
ms.author: baanders
ms.openlocfilehash: 6f43a55b3a409a84e3baf99dae24af7616ea4385
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87408208"
---
Azure Digital Twins usa [Azure Active Directory (Azure AD)](../articles/active-directory/fundamentals/active-directory-whatis.md) para el control de acceso basado en roles (RBAC). Esto significa que antes de que un usuario pueda hacer llamadas al plano de datos a la instancia de Azure Digital Twins, se debe asignar un rol a ese usuario con los permisos adecuados.

En el caso de Azure Digital Twins, este rol es _**Propietario de Azure Digital Twins (versión preliminar)**_. Puede leer más sobre los roles y la seguridad en [*Conceptos: Seguridad para las soluciones de Azure Digital Twins*](../articles/digital-twins/concepts-security.md).

En esta sección se muestra cómo crear una asignación de roles para un usuario en la instancia de Azure Digital Twins, mediante el correo electrónico del usuario en el inquilino de Azure AD de la suscripción de Azure. En función del rol de su organización, puede configurar este permiso para usted mismo o en nombre de otra persona que vaya a administrar la instancia de Azure Digital Twins.

### <a name="assign-the-role"></a>Asignación del rol

Para conceder a un usuario permisos para administrar una instancia de Azure Digital Twins, debe asignarlo al rol _**Propietario de Azure Digital Twins (versión preliminar)**_ dentro de la instancia.

> [!NOTE]
> Tenga en cuenta que este rol es distinto del rol *Propietario* de Azure AD, que también se puede asignar en el ámbito de la instancia de Azure Digital Twins. Se trata de dos roles de administración distintos y el rol *Propietario* de Azure AD no concede acceso a las características del plano de datos que se conceden con *Propietario de Azure Digital Twins (versión preliminar)* .