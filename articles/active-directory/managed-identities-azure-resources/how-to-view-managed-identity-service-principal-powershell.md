---
title: 'Visualización de la entidad de servicio de una identidad administrada mediante PowerShell: Azure AD'
description: Instrucciones detalladas para ver la entidad de servicio de una identidad administrada mediante PowerShell.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/29/2018
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a9da4689a1b5579f90a1df0feb487e50d57a9d98
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269223"
---
# <a name="view-the-service-principal-of-a-managed-identity-using-powershell"></a>Visualización de la entidad de servicio de una identidad administrada mediante PowerShell

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo, obtendrá información sobre cómo visualizar la entidad de servicio de una identidad administrada mediante PowerShell.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerequisites

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md).
- Si aún no tiene una cuenta de Azure, [regístrese para obtener una cuenta gratuita](https://azure.microsoft.com/free/).
- Habilite la [identidad asignada por el sistema en una máquina virtual](./qs-configure-portal-windows-vm.md#system-assigned-managed-identity) o [aplicación](../../app-service/overview-managed-identity.md#add-a-system-assigned-identity).
- [Instalar la versión más reciente de Azure PowerShell](/powershell/azure/install-az-ps)

## <a name="view-the-service-principal"></a>Visualización de la entidad de servicio

El comando siguiente muestra cómo ver la entidad de servicio de una aplicación o máquina virtual con la identidad asignada por el sistema habilitada. Reemplazar `<VM or application name>` por sus propios valores.

```powershell
Get-AzADServicePrincipal -DisplayName <VM or application name>
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo ver las entidades de servicio de Azure AD mediante PowerShell, consulte [Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal).