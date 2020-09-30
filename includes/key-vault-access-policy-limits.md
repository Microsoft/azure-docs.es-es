---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 08/27/2020
ms.author: msmbaldwin
ms.openlocfilehash: c74f2543e96087378741d6388b7c27dc4d05ddc8
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89380558"
---
Key Vault admite hasta 1024 entradas de directiva de acceso, donde cada entrada concede un conjunto de permisos distinto a una entidad de servicio particular: Debido a esta limitación, se recomienda asignar directivas de acceso a grupos de usuarios, siempre que sea posible, en lugar de a usuarios individuales. El uso de grupos facilita la administración de permisos para varias personas de la organización. Para más información, consulte [Administración del acceso a recursos y aplicaciones con grupos en Azure Active Directory](/azure/active-directory/fundamentals/active-directory-manage-groups).

Para la información completa sobre el control de acceso a Key Vault, consulte [Seguridad en Azure Key Vault: Administración de identidades y acceso](/azure/key-vault/general/overview-security#identity-and-access-management). 
