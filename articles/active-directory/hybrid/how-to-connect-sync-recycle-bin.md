---
title: 'Sincronización de Azure AD Connect: Habilitación de la papelera de reciclaje de AD | Microsoft Docs'
description: En este tema, se recomienda el uso de la característica Papelera de reciclaje de AD con Azure AD Connect.
services: active-directory
keywords: Papelera de reciclaje de AD, eliminación accidental, sourceAnchor
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: afec4207-74f7-4cdd-b13a-574af5223a90
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 12/17/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 12073a75cd248c9226c7ce5ecc21b64617823b32
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279642"
---
# <a name="azure-ad-connect-sync-enable-ad-recycle-bin"></a>Sincronización de Azure AD Connect: Habilitar la papelera de reciclaje de AD
Se recomienda que habilite la característica Papelera de reciclaje de AD en sus instancias locales de Active Directory, que se sincronizan con Azure AD. 

Si ha eliminado accidentalmente un objeto de usuario local de AD y lo restaura con esta característica, Azure AD restaura el objeto de usuario de Azure AD correspondiente.  Para más información acerca de la característica Papelera de reciclaje de AD, consulte el artículo [Scenario Overview for Restoring Deleted Active Directory Objects](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd379542(v=ws.10)) (Información general sobre el escenario de restauración de objetos de Active Directory eliminados).

## <a name="benefits-of-enabling-the-ad-recycle-bin"></a>Ventajas de habilitar la Papelera de reciclaje de AD
Esta característica ayuda a restaurar objetos de usuario de Azure AD mediante los pasos siguientes:

* Si ha eliminado accidentalmente un objeto de usuario de una implementación de AD local, se eliminará el objeto de usuario de Azure AD correspondiente en el siguiente ciclo de sincronización. De forma predeterminada, Azure AD mantiene el objeto de usuario de Azure AD eliminado en estado de eliminación temporal durante 30 días.

* Si tiene habilitada la característica Papelera de reciclaje de su instancia local de AD, puede restaurar el objeto de usuario de su instancia de AD local sin cambiar el valor de sourceAnchor. Cuando el objeto de usuario de la instancia de AD local recuperado se sincroniza con Azure AD, Azure AD restaurará el objeto correspondiente de usuario de Azure AD en estado de eliminación temporal. Para más información sobre el atributo delimitador de origen, consulte el artículo [Azure AD Connect: conceptos de diseño](./plan-connect-design-concepts.md#sourceanchor).

* Si no tiene habilitada la característica Papelera de reciclaje de su instancia de AD local, será necesario crear un objeto de usuario de AD para reemplazar el objeto eliminado. Si Azure AD Connect Synchronization Service está configurado para usar un atributo de AD generado por el sistema (por ejemplo, ObjectGuid) para el atributo sourceAnchor, el objeto de usuario de AD recién creado no tendrá el mismo valor de sourceAnchor que el objeto de usuario de AD eliminado. Cuando el objeto de usuario de AD recién creado se sincroniza con Azure AD, Azure AD crea un nuevo objeto de usuario de Azure AD en lugar de restaurar el objeto de usuario de Azure AD en estado de eliminación temporal.

> [!NOTE]
> De forma predeterminada, Azure AD mantiene los objetos de usuario de Azure AD eliminados en estado de eliminación temporal durante 30 días antes de eliminarlos permanentemente. Sin embargo, los administradores pueden acelerar la eliminación de dichos objetos. Una vez que los objetos se eliminan permanentemente, ya no se puedan recuperar, aunque esté habilitada la característica Papelera de reciclaje de la instancia de AD local.

## <a name="next-steps"></a>Pasos siguientes
**Temas de introducción**

* [Sincronización de Azure AD Connect: comprender y personalizar la sincronización](how-to-connect-sync-whatis.md)

* [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md)