---
title: 'Visualización de la entidad de servicio de una identidad administrada: CLI de Azure: Azure AD'
description: Instrucciones detalladas para ver la entidad de servicio de una identidad administrada mediante la CLI de Azure.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/29/2018
ms.author: markvi
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurecli
ms.openlocfilehash: 82529ab954c5474345e2702840daa1c7bca2ff1d
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501191"
---
# <a name="view-the-service-principal-of-a-managed-identity-using-azure-cli"></a>Visualización de la entidad de servicio de una identidad administrada mediante la CLI de Azure

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo, obtendrá información sobre cómo visualizar la entidad de servicio de una identidad administrada mediante la CLI de Azure.

## <a name="prerequisites"></a>Prerequisites

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md).
- Si aún no tiene una cuenta de Azure, [regístrese para obtener una cuenta gratuita](https://azure.microsoft.com/free/).
- Habilite la [identidad asignada por el sistema en una máquina virtual](/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm#system-assigned-managed-identity) o [aplicación](/azure/app-service/overview-managed-identity#add-a-system-assigned-identity).
- Para ejecutar los ejemplos de script de la CLI, tiene tres opciones:
    - Usar [Azure Cloud Shell](../../cloud-shell/overview.md) desde Azure Portal (consulte la sección siguiente).
    - Usar Azure Cloud Shell integrado a través del botón "Pruébelo", situado en la esquina superior derecha de cada bloque de código.
    - [Instale la versión más reciente de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli) si prefiere usar una consola de la CLI local e inicie sesión en Azure con `az login`.
 
[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="view-the-service-principal"></a>Visualización de la entidad de servicio

El comando siguiente muestra cómo ver la entidad de servicio de una aplicación o máquina virtual con la identidad administrada habilitada. Reemplazar `<VM or application name>` por sus propios valores. 

```azurecli-interactive
az ad sp list --display-name <VM or application name>
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo administrar las entidades de servicio de Azure AD con la CLI de Azure, vea [az ad sp](/cli/azure/ad/sp).


