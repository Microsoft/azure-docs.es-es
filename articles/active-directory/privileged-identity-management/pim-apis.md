---
title: Microsoft Graph API para PIM (versión preliminar) Azure AD | Microsoft Docs
description: Proporciona información acerca del uso de instancias de Microsoft Graph API para Azure AD Privileged Identity Management (PIM) (versión preliminar).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.subservice: pim
ms.topic: how-to
ms.date: 01/02/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 95b0b026b75b9b77c94451245ac4f18d487fc2e4
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783525"
---
# <a name="microsoft-graph-apis-for-privileged-identity-management-preview"></a>Instancias de Microsoft Graph API para Privileged Identity Management (versión preliminar)

Puede realizar todas las tareas de Privileged Identity Management mediante las instancias de [Microsoft Graph API](/graph/overview) para Azure Active Directory. En este artículo se describen conceptos importantes a la hora de usar instancias de Microsoft Graph API para Privileged Identity Management.

Para más información acerca de las instancias de Microsoft Graph API, consulte la [referencia de las API de Azure AD Privileged Identity Management](/graph/api/resources/privilegedidentitymanagement-root?view=graph-rest-beta).

> [!IMPORTANT]
> Las API de la versión /beta de Microsoft Graph están en versión preliminar y sujetas a cambios. No se admite su uso en aplicaciones de producción.

## <a name="required-permissions"></a>Permisos necesarios

Para llamar a las instancias de Microsoft Graph API para Privileged Identity Management, debe tener **uno o varios** de los siguientes permisos:

- `Directory.AccessAsUser.All`
- `Directory.Read.All`
- `Directory.ReadWrite.All`
- `PrivilegedAccess.ReadWrite.AzureAD`

### <a name="set-permissions"></a>Establecimiento de permisos

Para que las aplicaciones llamen a la instancias de Microsoft Graph API para Privileged Identity Management, deben tener los permisos necesarios. La forma más fácil de especificar los permisos necesarios es usar el [marco de consentimiento de Azure AD](../develop/consent-framework.md).

### <a name="set-permissions-in-graph-explorer"></a>Establecimiento de permisos en el Probador de Graph

Si utiliza el Probador de Graph para probar las llamadas, puede especificar los permisos en dicha herramienta.

1. Inicie sesión en [Probador de Graph](https://developer.microsoft.com/graph/graph-explorer) como administrador global.

1. Haga clic en **modificar permisos**.

    ![Probador de Graph: modificar permisos](./media/pim-apis/graph-explorer.png)

1. Seleccione las casillas situadas junto a los permisos que desea incluir. `PrivilegedAccess.ReadWrite.AzureAD` aún no está disponible en el Probador de Graph.

    ![Probador de Graph: modificar permisos](./media/pim-apis/graph-explorer-modify-permissions.png)

1. Haga clic en **Modificar permisos** para aplicar los cambios de permisos.

## <a name="next-steps"></a>Pasos siguientes

- [Referencia de las API de Azure AD Privileged Identity Management](/graph/api/resources/privilegedidentitymanagement-root?view=graph-rest-beta)