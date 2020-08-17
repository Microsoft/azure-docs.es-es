---
title: 'Administración del acceso de los invitados con las revisiones de acceso: Azure AD'
description: Administración de los usuarios invitados como miembros de un grupo o asignados a una aplicación con las revisiones de acceso de Azure Active Directory
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: compliance
ms.date: 07/22/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: 66f8020ec2561bf06716db0720a7ec4292f36eb3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87034129"
---
# <a name="manage-guest-access-with-azure-ad-access-reviews"></a>Administración del acceso de los invitados con las revisiones de acceso de Azure AD


Con Azure Active Directory (Azure AD), puede habilitar fácilmente la colaboración entre distintas organizaciones mediante la [característica B2B de Azure AD](../b2b/what-is-b2b.md). Los usuarios invitados de otros inquilinos pueden ser [invitados por los administradores](../b2b/add-users-administrator.md) o por [otros usuarios](../b2b/what-is-b2b.md). Esta capacidad también se aplica a las identidades sociales como las cuentas Microsoft.

También puede asegurarse fácilmente de que los usuarios invitados tengan el acceso adecuado. Puede pedir a los invitados o a quien decida en su lugar que participen en una revisión de acceso y vuelvan a certificar (o atestiguar) el acceso de los invitados. Los revisores pueden dar su aprobación para cada necesidad de acceso continuado de los usuarios, en función de las sugerencias de Azure AD. Cuando una revisión de acceso haya terminado, es posible hacer cambios y retirar la concesión de acceso a los invitados que ya no lo necesitan.

> [!NOTE]
> Este documento se centra en revisar el acceso de los usuarios invitados. Si desea revisar el acceso de todos los usuarios, no solo los invitados, vea [Administración del acceso de usuario con revisiones de acceso](manage-user-access-with-access-reviews.md). Si desea revisar la pertenencia de los usuarios a roles administrativos tales como administrador global, consulte [Inicio de una revisión de acceso en Azure AD Privileged Identity Management](../privileged-identity-management/pim-how-to-start-security-review.md).

## <a name="prerequisites"></a>Prerequisites

- Azure AD Premium P2

Para más información, vea [Requisitos de licencia](access-reviews-overview.md#license-requirements).

## <a name="create-and-perform-an-access-review-for-guests"></a>Creación y realización de una revisión de acceso para invitados

En primer lugar, como administrador global o administrador de usuarios, vaya a la [página Gobierno de identidades](https://portal.azure.com/#blade/Microsoft_AAD_ERM/DashboardBlade/) para asegurarse de que las revisiones de acceso están listas para su organización.

Azure AD ofrece varios escenarios para revisar el acceso de los usuarios invitados.

Puede revisar:

 - Un grupo de Azure AD con uno o más invitados como miembros.
 - Una aplicación conectada a Azure AD con uno o más usuarios invitados asignados a ella. 

A continuación, puede decidir si solicitar a cada invitado que revise su propio acceso o preguntar a uno o más usuarios que revise el acceso de cada invitado.

 Estos escenarios se tratan en las siguientes secciones.

### <a name="ask-guests-to-review-their-own-membership-in-a-group"></a>Se pide a los invitados que revisen su propia pertenencia a un grupo

Puede usar las revisiones de acceso para garantizar que los usuarios invitados y agregados a un grupo siguen necesitando acceso. Puede solicitar fácilmente a los invitados que revisen su propia pertenencia a ese grupo.

1. Para crear una revisión de acceso para el grupo, seleccione la revisión para incluir solo a los miembros que sean usuarios invitados y para que los miembros se revisen a sí mismos. Para más información, consulte [Creación de una revisión de acceso de los grupos o las aplicaciones](create-access-review.md).

2. Pida a cada invitado que revise su propia pertenencia. De forma predeterminada, cada invitado que haya aceptado una invitación recibirá un correo electrónico de Azure AD con un vínculo a la revisión de acceso. Azure AD tiene instrucciones para los invitados sobre cómo [revisar el acceso a grupos o aplicaciones](perform-access-review.md).

3. Cuando los revisores hayan proporcionado la información, detenga la revisión de acceso y aplique los cambios. Para más información, consulte [Revisión de acceso de los grupos o las aplicaciones](complete-access-review.md).

4. Además de los usuarios que negaron su necesidad de seguir teniendo acceso, puede también quitar a los usuarios que no respondieron. Es posible que los usuarios que no respondieron no reciban ya correos electrónicos.

5. Si el grupo no se usó para la administración de acceso, también puede quitar los usuarios que no estuvieran seleccionados para participar en la revisión porque no aceptaran su invitación. La no aceptación podría indicar que la dirección de correo electrónico del usuario invitado tiene un error de escritura. Si se utiliza un grupo como una lista de distribución, quizás algunos usuarios invitados no se seleccionaron para la participación porque son objetos de contacto.

### <a name="ask-a-sponsor-to-review-a-guests-membership-in-a-group"></a>Se pide a los patrocinadores que revisen la pertenencia de un invitado a un grupo

Puede solicitar a un patrocinador, por ejemplo al propietario de un grupo, que revise la necesidad de un invitado de seguir perteneciendo a un grupo.

1. Para crear una revisión de acceso para el grupo, seleccione la revisión para incluir solo a los miembros que sean usuarios invitados. Luego especifique uno o más revisores. Para más información, consulte [Creación de una revisión de acceso de los grupos o las aplicaciones](create-access-review.md).

2. Pida a los revisores que proporcionen sus datos de entrada. De forma predeterminada, cada uno recibe un correo electrónico de Azure AD con un vínculo al panel de acceso con el que podrán [realizar la revisión de acceso de los grupos o las aplicaciones](perform-access-review.md).

3. Cuando los revisores hayan proporcionado la información, detenga la revisión de acceso y aplique los cambios. Para más información, consulte [Revisión de acceso de los grupos o las aplicaciones](complete-access-review.md).

### <a name="ask-guests-to-review-their-own-access-to-an-application"></a>Se pide a los invitados que revisen su propio acceso a una aplicación

Puede usar revisiones de acceso para asegurarse de que los usuarios que han sido invitados a una aplicación concreta siguen necesitando el acceso. Puede solicitarles de manera fácil que revisen su propia necesidad de acceso.

1. Para crear una revisión de acceso para la aplicación, seleccione la revisión para incluir solo a los invitados y para que los usuarios revisen su propio acceso. Para más información, consulte [Creación de una revisión de acceso de los grupos o las aplicaciones](create-access-review.md).

2. Se pide a cada invitado que revise su propio acceso a la aplicación. De forma predeterminada, cada invitado que haya aceptado una invitación recibirá un correo electrónico de Azure AD. Dicho correo electrónico tiene un vínculo a la revisión de acceso en el panel de acceso de su organización. Azure AD tiene instrucciones para los invitados sobre cómo [revisar el acceso a grupos o aplicaciones](perform-access-review.md).

3. Cuando los revisores hayan proporcionado la información, detenga la revisión de acceso y aplique los cambios. Para más información, consulte [Revisión de acceso de los grupos o las aplicaciones](complete-access-review.md).

4. Además de los usuarios que negaron su necesidad de seguir teniendo acceso, puede también quitar a los usuarios invitados que no respondieron. Es posible que los usuarios que no respondieron no reciban ya correos electrónicos. También puede quitar los usuarios invitados que no estaban seleccionados para participar, especialmente si no recibieron recientemente ninguna invitación. Esos usuarios no aceptaron su invitación y, por lo tanto, no disponían de acceso a la aplicación. 

### <a name="ask-a-sponsor-to-review-a-guests-access-to-an-application"></a>Se pide a los patrocinadores que revisen su propio acceso a una aplicación

Puede solicitar a un patrocinador, por ejemplo, al propietario de una aplicación, que revise la necesidad de un invitado de seguir teniendo acceso a la aplicación.

1. Para crear una revisión de acceso para la aplicación, seleccione la revisión para incluir solo a los invitados. Luego especifique uno o más usuarios como revisores. Para más información, consulte [Creación de una revisión de acceso de los grupos o las aplicaciones](create-access-review.md).

2. Pida a los revisores que proporcionen sus datos de entrada. De forma predeterminada, cada uno recibe un correo electrónico de Azure AD con un vínculo al panel de acceso con el que podrán [realizar la revisión de acceso de los grupos o las aplicaciones](perform-access-review.md).

3. Cuando los revisores hayan proporcionado la información, detenga la revisión de acceso y aplique los cambios. Para más información, consulte [Revisión de acceso de los grupos o las aplicaciones](complete-access-review.md).

### <a name="ask-guests-to-review-their-need-for-access-in-general"></a>Se pide a los invitados que revisen el acceso que requieren en general

En algunas organizaciones, los invitados pueden no ser conscientes de a qué grupos pertenecen.

> [!NOTE]
> Las versiones anteriores de Azure Portal no permitieron el acceso administrativo a los usuarios con el UserType de Guest. En algunos casos, un administrador del directorio podría haber cambiado el valor UserType a Member mediante PowerShell. Si anteriormente se produjo este cambio en el directorio, la consulta anterior podría no incluir todos los usuarios invitados que históricamente tenían derechos de acceso administrativo. En este caso, debe cambiar el UserType del invitado o incluir manualmente el invitado en la pertenencia al grupo.

1. Cree un grupo de seguridad en Azure AD con los invitados como miembros, si aún no existe un grupo adecuado. Por ejemplo, puede crear un grupo con la pertenencia mantenida de forma manual para los invitados. O bien, puede crear un grupo dinámico con un nombre como "Invitados de Contoso" para los usuarios del inquilino Contoso que tengan el valor Guest en el atributo UserType.  Por motivos de eficacia, asegúrese de que el grupo está compuesto principalmente de invitados: no seleccione un grupo que tenga usuarios miembros, ya que no es necesario revisarlos.  Además, tenga en cuenta que un usuario invitado que sea miembro del grupo puede ver a los demás miembros del grupo.

2. Para crear una revisión de acceso para ese grupo, seleccione a los revisores para que sean los propios miembros. Para más información, consulte [Creación de una revisión de acceso de los grupos o las aplicaciones](create-access-review.md).

3. Pida a cada invitado que revise su propia pertenencia. De forma predeterminada, cada invitado que haya aceptado una invitación recibirá un correo electrónico de Azure AD con un vínculo a la revisión de acceso del panel de acceso de la organización. Azure AD tiene instrucciones para los invitados sobre cómo [revisar el acceso a grupos o aplicaciones](perform-access-review.md).  Los invitados que no se aceptaron su invitación aparecerán en los resultados de la revisión como "Sin notificar".

4. Cuando los revisores hayan proporcionado la información, detenga la revisión de acceso. Para más información, consulte [Revisión de acceso de los grupos o las aplicaciones](complete-access-review.md).

5. Quite el acceso de invitado para los invitados cuyo acceso se denegara, no completaran la revisión o no hubieran aceptado su invitación previamente. Si algunos de los invitados son contactos que no fueron seleccionados para participar en la revisión o que previamente no habían aceptado una invitación, puede deshabilitar sus cuentas mediante Azure Portal o PowerShell. Si el invitado ya no necesita el acceso y no es un contacto, puede quitar su objeto de usuario desde su directorio mediante Azure Portal o PowerShell para eliminar el objeto de usuario invitado.

## <a name="next-steps"></a>Pasos siguientes

[Creación de una revisión de acceso de grupos o aplicaciones](create-access-review.md)







