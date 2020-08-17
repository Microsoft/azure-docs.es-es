---
title: Eliminación de un clúster de Azure HDInsight
description: Información sobre las distintas formas de eliminar un clúster de Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: H1Hack27Feb2017,hdinsightactive
ms.date: 11/29/2019
ms.openlocfilehash: 3903500e87dd162779d1732a4cfe405ba34e702c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074750"
---
# <a name="delete-an-hdinsight-cluster-using-your-browser-powershell-or-the-azure-cli"></a>Eliminación de un clúster de HDInsight con el explorador, PowerShell o la CLI de Azure

La facturación del clúster de HDInsight se inicia una vez creado el clúster y solo se detiene cuando se elimina. La facturación se prorratea por minuto, por lo que siempre se debe eliminar el clúster cuando ya no está en uso. En este documento, aprenderá a eliminar un clúster mediante [Azure Portal](https://portal.azure.com), el [módulo Az de Azure PowerShell](https://docs.microsoft.com/powershell/azure/) y la [CLI de Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest).

> [!IMPORTANT]  
> Al eliminar un clúster de HDInsight, no se eliminan las cuentas de Azure Storage o Data Lake Storage asociadas a este. Puede volver a usar los datos almacenados en esos servicios en el futuro.

## <a name="azure-portal"></a>Portal de Azure

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Desde el menú de la izquierda, vaya a **Todos los servicios** > **Analytics** > **Clústeres de HDInsight** y seleccione su clúster.

3. En la vista predeterminada, seleccione el icono **Eliminar**. Siga las indicaciones para eliminar el clúster.

    ![Botón de eliminación de clúster de HDInsight](./media/hdinsight-delete-cluster/hdinsight-delete-cluster.png)

## <a name="azure-powershell"></a>Azure PowerShell

Reemplace `CLUSTERNAME` por el nombre del clúster de HDInsight en el siguiente código. Desde un símbolo del sistema de PowerShell, escriba el siguiente comando para eliminar el clúster:

```powershell
Remove-AzHDInsightCluster -ClusterName CLUSTERNAME
```

## <a name="azure-cli"></a>Azure CLI

Reemplace `CLUSTERNAME` por el nombre del clúster de HDInsight y `RESOURCEGROUP` por el nombre del grupo de recursos en el código siguiente.  Desde un símbolo del sistema, escriba lo siguiente para eliminar el clúster:

```azurecli
az hdinsight delete --name CLUSTERNAME --resource-group RESOURCEGROUP
```
