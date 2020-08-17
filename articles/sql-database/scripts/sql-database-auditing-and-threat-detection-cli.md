---
title: 'Ejemplo de CLI de auditoría y Advanced Threat Protection: Azure SQL Database'
description: Ejemplo de script de la CLI de Azure para configurar la auditoría y Advanced Threat Protection en una base de datos Azure SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: security
ms.devlang: azurecli
ms.topic: sample
author: ronitr
ms.author: ronitr
ms.reviewer: carlrab, vanto
ms.date: 08/05/2019
ms.openlocfilehash: ac945b14091ae01cfad66c3776c84406583e491f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87037124"
---
# <a name="use-cli-to-configure-sql-database-auditing-and-advanced-threat-protection"></a>Uso de la CLI para configurar la auditoría y Advanced Threat Protection en SQL Database

Este ejemplo de script de la CLI de Azure configura la auditoría y Advanced Threat Protection en SQL Database.

Si decide instalar y usar la CLI localmente, para este tema es preciso que ejecute la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="sample-script"></a>Script de ejemplo

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

```azurecli-interactive
$subscription = "<subscriptionId>" # add subscription here

az account set -s $subscription # ...or use 'az login'
```

### <a name="run-the-script"></a>Ejecute el script.

[!code-azurecli-interactive[main](../../../cli_scripts/sql-database/database-auditing-and-threat-detection/database-auditing-and-threat-detection.sh "Configure auditing and threat detection")]

### <a name="clean-up-deployment"></a>Limpieza de la implementación

Use el siguiente comando para quitar el grupo de recursos y todos los recursos que tenga asociados.

```azurecli-interactive
az group delete --name $resource
```

## <a name="sample-reference"></a>Referencia de ejemplo

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Descripción |
|---|---|
| [az sql db audit-policy](/cli/azure/sql/db/audit-policy) | Establece la directiva de auditoría para una base de datos. |
| [az sql db threat-policy](/cli/azure/sql/db/threat-policy) | Establece una directiva de Advanced Threat Protection en una base de datos. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de la CLI de SQL Database en la [documentación de Azure SQL Database](../../azure-sql/database/az-cli-script-samples-content-guide.md).
