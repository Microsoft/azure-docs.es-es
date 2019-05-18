---
title: Sincronizar los atributos de Azure AD para la asignación | Microsoft Docs
description: Obtenga información sobre cómo sincronizar los atributos de Active Directory local a Azure AD. Al configurar el aprovisionamiento de usuarios para aplicaciones SaaS, utilice la característica de extensión de directorio para agregar atributos de origen que no están sincronizados de forma predeterminada.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/13/2019
ms.author: mimart
ms.custom: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: a9460bc924ea662646360d1a3f5087949a39a03f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65806122"
---
# <a name="sync-an-attribute-from-your-on-premises-active-directory-to-azure-ad-for-provisioning-to-an-application"></a>Sincronizar un atributo de Active Directory local a Azure AD para el aprovisionamiento en una aplicación

Al personalizar las asignaciones de atributos para el aprovisionamiento de usuarios, es posible que el atributo que desea asignar no aparece en el **atributo de origen** lista. Este artículo muestra cómo agregar el atributo que falta mediante la sincronización desde su Active Directory (AD) en local a Azure Active Directory (Azure AD).

Azure AD debe contener todos los datos necesarios para crear un perfil de usuario al aprovisionar cuentas de usuario de Azure AD a una aplicación SaaS. En algunos casos, para que los datos estén disponibles puede necesita sincronizar atributos desde las instalaciones de AD a Azure AD. Azure AD Connect sincroniza automáticamente determinados atributos en Azure AD, pero no todos los atributos. Además, algunos atributos (por ejemplo, SAMAccountName) que se sincronizan de forma predeterminada no pueden exponerse a través de la API de Azure AD Graph. En estos casos, puede usar la característica de extensión de directorio de Azure AD Connect para sincronizar el atributo a Azure AD. De este modo, el atributo será visible para la API de Azure AD Graph y el servicio de aprovisionamiento de Azure AD.

Si los datos que necesita para el aprovisionamiento en Active Directory, pero no está disponibles para el aprovisionamiento debido a las razones descritas anteriormente, siga estos pasos.
 
## <a name="sync-an-attribute"></a>Sincronizar un atributo 

1. Abra el Asistente de Azure AD Connect, elija tareas y, a continuación, elija **personalizar las opciones de sincronización**.

   ![Azure Active Directory Connect página de tareas adicionales de Asistente](media/user-provisioning-sync-attributes-for-mapping/active-directory-connect-customize.png)
 
2. Inicie sesión como administrador Global de Azure AD. 

3. En el **características opcionales** página, seleccione **sincronización de atributos de extensión de directorios**.
 
   ![Azure Active Directory Connect página de características opcionales de Asistente](media/user-provisioning-sync-attributes-for-mapping/active-directory-connect-directory-extension-attribute-sync.png)

4. Seleccione los atributos que desea extender a Azure AD.
   > [!NOTE]
   > La búsqueda en **atributos disponibles** distingue mayúsculas de minúsculas.

   ![Azure Active Directory Connect Directory extensiones selección página del Asistente](media/user-provisioning-sync-attributes-for-mapping/active-directory-connect-directory-extensions.png)

5. Finalizar al Asistente de Azure AD Connect y permiten ejecutar un ciclo de sincronización completa. Cuando se complete el ciclo, se extiende el esquema y los nuevos valores se sincronizan entre sus instalaciones de AD y Azure AD.
 
6. En el portal de Azure, mientras estás [editar asignaciones de atributos de usuario](customize-application-attributes.md), **atributo de origen** lista contendrá el atributo agregado en el formato `<attributename> (extension_<appID>_<attributename>)`. Seleccione el atributo y asignarlo a la aplicación de destino para el aprovisionamiento.

   ![Azure Active Directory Connect Directory extensiones selección página del Asistente](media/user-provisioning-sync-attributes-for-mapping/attribute-mapping-extensions.png)

> [!NOTE]
> La capacidad de aprovisionar los atributos de referencia de AD local, como **managedby** o **DN/DistinguishedName**, actualmente no se admite. Puede solicitar esta característica en [User Voice](https://feedback.azure.com/forums/169401-azure-active-directory). 

## <a name="next-steps"></a>Pasos siguientes

* [Definir quién es en el ámbito de aprovisionamiento](define-conditional-rules-for-provisioning-user-accounts.md)
