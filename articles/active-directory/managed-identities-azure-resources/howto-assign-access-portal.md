---
title: 'Asignación de acceso de una identidad administrada a un recurso mediante Azure Portal: Azure AD'
description: Instrucciones paso a paso para asignar a una identidad administrada de un recurso acceso a otro recurso, mediante Azure Portal.
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
ms.date: 09/14/2017
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: ee28addd5f6ac0faaf03d29d712c4e750ed6e293
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89177489"
---
# <a name="assign-a-managed-identity-access-to-a-resource-by-using-the-azure-portal"></a>Asignación de acceso de una identidad administrada a un recurso mediante Azure Portal

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Una vez que haya configurado un recurso de Azure con una identidad administrada, puede dar acceso de identidad administrada a otro recurso, al igual que cualquier entidad de seguridad. En este artículo se muestra cómo otorgar acceso a la identidad administrada de un conjunto de escalado de máquinas virtuales o una máquina virtual de Azure para una cuenta de Azure Storage mediante Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.

## <a name="use-azure-rbac-to-assign-a-managed-identity-access-to-another-resource"></a>Uso de Azure RBAC para asignar el acceso de una identidad administrada a otro recurso

Después de habilitar la identidad administrada en un recurso de Azure, como una [máquina virtual de Azure](qs-configure-portal-windows-vm.md) o un [conjunto de escalado de máquinas virtuales de Azure](qs-configure-portal-windows-vmss.md):

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure en la que ha configurado la identidad administrada.

2. Vaya al recurso para el que desea modificar el control de acceso. En este ejemplo, vamos a dar a una máquina virtual de Azure acceso a una cuenta de almacenamiento, de modo que navegamos a la cuenta de almacenamiento.

3. Seleccione la página **Control de acceso (IAM)** del recurso y seleccione **+ Agregar asignación de roles**. Luego, especifique el **rol**, configure la opción **Asignar acceso a** y especifique la **suscripción** correspondiente. En el área de criterios de búsqueda, debería ver el recurso. Seleccione el recurso y seleccione **Guardar**. 

   ![Captura de pantalla Control de acceso (IAM)](./media/msi-howto-assign-access-portal/assign-access-control-iam-blade-before.png)  
     
## <a name="next-steps"></a>Pasos siguientes

- [Identidades administradas de recursos de Azure](overview.md)
- Para habilitar la identidad administrada de una máquina virtual de Azure, consulte [Configuración de identidades administradas de recursos de Azure en una VM de Azure mediante Azure Portal](qs-configure-portal-windows-vm.md).
- Para habilitar la identidad administrada en un conjunto de escalado de máquinas virtuales de Azure, consulte [Configuración de identidades administradas de recursos de Azure en un conjunto de escalado de máquinas virtuales mediante Azure Portal](qs-configure-portal-windows-vmss.md).


