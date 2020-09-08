---
author: baanders
description: archivo de inclusión para los pasos información general y requisitos previos de permisos en la instalación de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 8/7/2020
ms.author: baanders
ms.openlocfilehash: b87d45a8be68a77df7b06ebd6e80562ccbe0e444
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009680"
---
>[!NOTE]
>Estas operaciones están pensadas para que las realice un usuario con la capacidad de administrar los recursos y el acceso de usuario en la suscripción de Azure. Aunque algunos pasos se pueden realizar con permisos inferiores, será necesaria la colaboración de alguien con estos permisos para configurar por completo una instancia que se pueda usar. Para más información, consulte la sección [*Requisitos previos: permisos necesarios*](#prerequisites-permission-requirements) que aparece a continuación.

La configuración completa de una instancia nueva de Azure Digital Twins consta de tres partes:
1. **Creación de la instancia**
2. **Configuración de permisos de acceso de usuarios**: los usuarios de Azure deben tener el rol *Propietario de Azure Digital Twins (versión preliminar)* en la instancia de Azure Digital Twins para poder administrarla y administrar sus datos. En este paso, como propietario o administrador de la suscripción de Azure, se asignará este rol a la persona que vaya a administrar la instancia de Azure Digital Twins. Puede ser usted mismo o alguna otra persona de la organización.
3. **Configuración de los permisos de acceso para aplicaciones cliente**: es habitual escribir una aplicación cliente que se utilizará para interactuar con la instancia. Para que la aplicación cliente acceda a su instancia de Azure Digital Twins, debe configurar un *registro de aplicaciones* en [Azure Active Directory](../articles/active-directory/fundamentals/active-directory-whatis.md) que la aplicación cliente usará para autenticarse en la instancia.

Para continuar, necesitará una suscripción de Azure. [Aquí](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) puede configurar una gratis.

## <a name="prerequisites-permission-requirements"></a>Requisitos previos: Requisitos de permisos

Para poder realizar todos los pasos de este artículo, debe tener un [rol en la suscripción](../articles/role-based-access-control/rbac-and-directory-admin-roles.md) que tenga los permisos siguientes:
* Crear y administrar recursos de Azure
* Administrar el acceso de los usuarios a los recursos de Azure (incluida la concesión y delegación de permisos)

Los roles comunes que cumplen este requisito son *propietario*, *administrador de cuentas* o la combinación de *administrador de acceso de usuarios* y *colaborador*. Puede encontrar una explicación completa de los roles y permisos, como, por ejemplo, los permisos que se incluyen con otros roles, en [*Roles de administrador de la suscripción clásica, roles de Azure y roles de Azure AD*](../articles/role-based-access-control/rbac-and-directory-admin-roles.md) en la documentación de RBAC de Azure.

Para ver el rol de la suscripción, visite la [página de suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) en Azure Portal (puede usar este vínculo o buscar *Suscripciones* con la barra de búsqueda del portal). Busque el nombre de la suscripción que usa y vea el rol que tiene en la columna *Mi rol*:

:::image type="content" source="../articles/digital-twins/media/how-to-set-up-instance/portal/subscriptions-role.png" alt-text="Vista de la página Suscripciones en Azure Portal que muestra el usuario como propietario" lightbox="../articles/digital-twins/media/how-to-set-up-instance/portal/subscriptions-role.png":::

Si observa que el valor es *Colaborador* u otro rol que no tenga los permisos necesarios descritos anteriormente, puede ponerse en contacto con el usuario de la suscripción que *tenga* estos permisos (por ejemplo, propietario de la suscripción o administrador de cuentas) y continuar de una de las siguientes maneras:
* Solicite que realicen los pasos de este artículo en su nombre.
* Solicite que eleven los privilegios de su rol en la suscripción para que tenga los permisos necesarios para continuar por su cuenta. Que esto sea adecuado depende de la organización y del rol que tenga.