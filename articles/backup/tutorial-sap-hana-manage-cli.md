---
title: 'Tutorial: Administración de las bases de datos de SAP HANA con copia de seguridad con la CLI'
description: En este tutorial aprenderá a administrar las bases de datos de SAP HANA con copia de seguridad que se ejecutan en una máquina virtual de Azure con la CLI de Azure.
ms.topic: tutorial
ms.date: 12/4/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: c47f03b2ac1640c12a833f8bdb53b5d6493d7eb6
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489442"
---
# <a name="tutorial-manage-sap-hana-databases-in-an-azure-vm-using-azure-cli"></a>Tutorial: Administración de las bases de datos de SAP HANA de una máquina virtual de Azure con la CLI de Azure

La CLI de Azure se usa para crear y administrar los recursos de Azure desde la línea de comandos o mediante scripts. En esta documentación se detalla cómo administrar una base de datos de SAP HANA con copia de seguridad en una máquina virtual de Azure, todo ello con la CLI de Azure. También puede llevar a cabo estos pasos con [Azure Portal](./sap-hana-db-manage.md).

Use [Azure Cloud Shell](tutorial-sap-hana-backup-cli.md) para ejecutar comandos de la CLI.

Al final de este tutorial podrá realizar lo siguiente:

> [!div class="checklist"]
>
> * Supervisar los trabajos de copia de seguridad y restauración
> * Proteger las nuevas bases de datos agregadas a una instancia de SAP HANA
> * Cambiar la directiva
> * Detener protección
> * Reanudar protección

Si ha usado [Realización de una copia de seguridad de una base de datos de SAP HANA en Azure mediante la CLI](tutorial-sap-hana-backup-cli.md) para realizar la copia de seguridad de la base de datos de SAP HANA, estará usando los siguientes recursos:

* un grupo de recursos denominado *saphanaResourceGroup*,
* un almacén denominado *saphanaVault*,
* un contenedor protegido denominado *VMAppContainer;Compute;saphanaResourceGroup;saphanaVM*,
* el elemento o la base de datos con copia de seguridad denominado *saphanadatabase;hxe;hxe* y
* los recursos de la región *westus2*.

La CLI de Azure facilita la administración de las bases de datos de SAP HANA que se ejecutan en una máquina virtual de Azure con copia de seguridad realizada mediante Azure Backup. En este tutorial se detallan todas las operaciones de administración.

## <a name="monitor-backup-and-restore-jobs"></a>Supervisar los trabajos de copia de seguridad y restauración

Para supervisar los trabajos completados o en ejecución (copia de seguridad o restauración), use el cmdlet [az backup job list](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-list). La CLI también permite [suspender un trabajo que se esté ejecutando actualmente](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-stop) o [esperar hasta que se complete un trabajo](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-wait).

```azurecli-interactive
az backup job list --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --output table
```

El resultado tendrá un aspecto similar al siguiente:

```output
Name                                  Operation              Status      Item Name       Start Time UTC
------------------------------------  ---------------        ---------   ----------      -------------------  
e0f15dae-7cac-4475-a833-f52c50e5b6c3  ConfigureBackup        Completed   hxe             2019-12-03T03:09:210831+00:00  
ccdb4dce-8b15-47c5-8c46-b0985352238f  Backup (Full)          Completed   hxe [hxehost]   2019-12-01T10:30:58.867489+00:00
4980af91-1090-49a6-ab96-13bc905a5282  Backup (Differential)  Completed   hxe [hxehost]   2019-12-01T10:36:00.563909+00:00
F7c68818-039f-4a0f-8d73-e0747e68a813  Restore (Log)          Completed   hxe [hxehost]   2019-12-03T05:44:51.081607+00:00
```

## <a name="change-policy"></a>Cambio de la directiva

Para cambiar la directiva subyacente a la configuración de copia de seguridad de SAP HANA, use el cmdlet [az backup policy set](/cli/azure/backup/policy?view=azure-cli-latest#az-backup-policy-set). El parámetro name de este cmdlet hace referencia al elemento de copia de seguridad cuya directiva se desea cambiar. En este tutorial vamos a reemplazar la directiva de nuestra base de datos de SAP HANA *saphanadatabase;hxe;hxe* con una nueva directiva *newsaphanaPolicy*. Las nuevas directivas se pueden crear con el cmdlet [az backup policy create](/cli/azure/backup/policy?view=azure-cli-latest#az-backup-policy-create).

```azurecli-interactive
az backup item set policy --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --policy-name newsaphanaPolicy \
    --name saphanadatabase;hxe;hxe \
```

El resultado debe ser similar al siguiente:

```output
Name                                  Resource Group
------------------------------------- --------------
cb110094-9b15-4c55-ad45-6899200eb8dd  SAPHANA
```

## <a name="protect-new-databases-added-to-an-sap-hana-instance"></a>Proteger las nuevas bases de datos agregadas a una instancia de SAP HANA

El [registro de una instancia de SAP HANA con un almacén de Recovery Services](tutorial-sap-hana-backup-cli.md#register-and-protect-the-sap-hana-instance) detecta automáticamente todas las bases de datos de esta instancia.

Sin embargo, en casos en los que las nuevas bases de datos se agreguen a la instancia de SAP HANA posteriormente, use el cmdlet [az backup protectable-item initialize](/cli/azure/backup/protectable-item?view=azure-cli-latest#az-backup-protectable-item-initialize). Este cmdlet detecta las nuevas bases de datos agregadas.

```azurecli-interactive
az backup protectable-item initialize --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --workload-type SAPHANA
```

A continuación, use el cmdlet [az backup protected-item list](/cli/azure/backup/protectable-item?view=azure-cli-latest#az-backup-protectable-item-list) para eenumerar todas las bases de datos que se hayan detectado en la instancia de SAP HANA. Sin embargo, esta lista excluye las bases de datos en las que ya se haya configurado la copia de seguridad. Una vez detectada la base de datos de la que se va a realizar la copia de seguridad, consulte [Habilitación de la copia de seguridad de la base de datos de SAP HANA](tutorial-sap-hana-backup-cli.md#enable-backup-on-sap-hana-database).

```azurecli-interactive
az backup protectable-item list --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --workload-type SAPHANA \
    --output table
```

La nueva base de datos de la que desea realizar una copia de seguridad se mostrará en esta lista, que tendrá el siguiente aspecto:

```output
Name                            Protectable Item Type    ParentName    ServerName    IsProtected
---------------------------     ----------------------   ------------  -----------   ------------
saphanasystem;hxe               SAPHanaSystem            HXE           hxehost       NotProtected  
saphanadatabase;hxe;systemdb    SAPHanaDatabase          HXE           hxehost       NotProtected
saphanadatabase;hxe;newhxe      SAPHanaDatabase          HXE           hxehost       NotProtected
```

## <a name="stop-protection-for-an-sap-hana-database"></a>Detención de la protección para una base de datos de SAP HANA

Puede dejar de proteger una base de datos SAP HANA de dos maneras:

* Detener todos los trabajos futuros de copia de seguridad y eliminar todos los puntos de recuperación.
* Detener todos los trabajos futuros de copia de seguridad y dejar intactos los puntos de recuperación.

Si decide dejar los puntos de recuperación, tenga en cuenta estos detalles:

* Todos los puntos de recuperación permanecerán intactos para siempre; al detenerse la protección de los datos se detendrá la eliminación de todos los puntos y se conservarán los datos.
* Se le cobrará la instancia protegida y el almacenamiento consumido.
* Si elimina un origen de datos sin detener las copias de seguridad, las nuevas copias de seguridad producirán errores.

Echemos un vistazo a cada una de las formas de detener la protección con más detalle.

### <a name="stop-protection-with-retain-data"></a>Detener la protección con conservación de datos

Para detener la protección y conservar los datos, use el cmdlet [az backup protection disable](/cli/azure/backup/protection?view=azure-cli-latest#az-backup-protection-disable).

```azurecli-interactive
az backup protection disable --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --workload-type SAPHANA \
    --output table
```

El resultado debe ser similar al siguiente:

```output
Name                                  ResourceGroup
------------------------------------  ---------------  
g0f15dae-7cac-4475-d833-f52c50e5b6c3  saphanaResourceGroup
```

Para comprobar el estado de esta operación, utilice el cmdlet [az backup job show](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-show).

### <a name="stop-protection-without-retain-data"></a>Detención de la protección sin conservar los datos

Para detener la protección sin conservar los datos, use el cmdlet [az backup protecion disable](/cli/azure/backup/protection?view=azure-cli-latest#az-backup-protection-disable).

```azurecli-interactive
az backup protection disable --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --workload-type SAPHANA \
    --delete-backup-data true \
    --output table
```

El resultado debe ser similar al siguiente:

```output
Name                                  ResourceGroup
------------------------------------  ---------------  
g0f15dae-7cac-4475-d833-f52c50e5b6c3  saphanaResourceGroup
```

Para comprobar el estado de esta operación, utilice el cmdlet [az backup job show](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-show).

## <a name="resume-protection"></a>Reanudar protección

Al detener la protección de la base de datos de SAP HANA y conservar los datos, se puede reanudar la protección más adelante. Si no conserva los datos de los que se realizó una copia de seguridad, no podrá reanudar la protección.

Para reanudar la protección, use el cmdlet [az backup protectin resume](/cli/azure/backup/protection?view=azure-cli-latest#az-backup-protection-resume).

```azurecli-interactive
az backup protection resume --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --policy-name saphanaPolicy \
    --output table
```

El resultado debe ser similar al siguiente:

```output
Name                                  ResourceGroup
------------------------------------  ---------------  
b2a7f108-1020-4529-870f-6c4c43e2bb9e  saphanaResourceGroup
```

Para comprobar el estado de esta operación, utilice el cmdlet [az backup job show](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-show).

## <a name="next-steps"></a>Pasos siguientes

* Para aprender a realizar una copia de seguridad de una base de datos de SAP HANA que se ejecuta en una máquina virtual de Azure mediante Azure Portal, consulte [Copia de seguridad de bases de datos de SAP HANA en máquinas virtuales de Azure](./backup-azure-sap-hana-database.md)

* Para aprender a administrar una base de datos de SAP HANA con copia de seguridad que se ejecuta en una máquina virtual de Azure mediante Azure Portal, consulte [Administración de bases de datos de SAP HANA con copia de seguridad en una máquina virtual de Azure](./sap-hana-db-manage.md)
