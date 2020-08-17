---
title: 'Tutorial: copia de seguridad de la base de datos de SAP HANA en Azure mediante la CLI'
description: En este tutorial aprenderá a hacer una copia de seguridad de las bases de datos de SAP HANA que se ejecutan en una máquina virtual de Azure en un almacén de Recovery Services de Azure Backup mediante la CLI de Azure.
ms.topic: tutorial
ms.date: 12/4/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4113ba75f007bfa03fed5cfeaed7737797e37ed9
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489527"
---
# <a name="tutorial-back-up-sap-hana-databases-in-an-azure-vm-using-azure-cli"></a>Tutorial: Copia de seguridad de las bases de datos de SAP HANA en una máquina virtual de Azure con la CLI de Azure

La CLI de Azure se usa para crear y administrar los recursos de Azure desde la línea de comandos o mediante scripts. En esta documentación se detalla cómo realizar una copia de seguridad de una base de datos de SAP HANA y desencadenar copias de seguridad a petición, todo ello mediante la CLI de Azure. También puede llevar a cabo estos pasos con [Azure Portal](./backup-azure-sap-hana-database.md).

En este documento se supone que ya tiene una base de datos de SAP HANA instalada en una máquina virtual de Azure. (También puede [crear una máquina virtual mediante la CLI de Azure](../virtual-machines/linux/quick-create-cli.md)). Al final de este tutorial podrá realizar lo siguiente:

> [!div class="checklist"]
>
> * Creación de un almacén de Servicios de recuperación
> * Registrar la instancia de SAP HANA y detectar bases de datos en ella
> * Habilitar la copia de seguridad de una base de datos de SAP HANA
> * Desencadenamiento de una copia de seguridad a petición

Consulte los [escenarios admitidos actualmente](./sap-hana-backup-support-matrix.md#scenario-support) para SAP HANA.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Para instalar y usar la CLI localmente, debe ejecutar la versión xx.xxx.x de la CLI de Azure o una posterior. Para averiguar la versión de la CLI, ejecute `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Servicios de recuperación

Un almacén de Recovery Services es un contenedor lógico que almacena los datos de copia de seguridad de los recursos protegidos, como las máquinas virtuales de Azure o las cargas de trabajo que se ejecutan en estas (como las bases de datos de SQL o de HANA). Cuando se ejecuta el trabajo de copia de seguridad para un recurso protegido, crea un punto de recuperación en el almacén de Recovery Services. Posteriormente, se puede usar uno de estos puntos de recuperación para restaurar los datos a un momento dado en el tiempo.

Cree un almacén de Recovery Services con [az backup vault create](/cli/azure/backup/vault#az-backup-vault-create). Especifique el mismo grupo de recursos y ubicación que tenga la máquina virtual que desea proteger. Aprenda a crear una máquina virtual mediante la CLI de Azure con este [inicio rápido con las máquinas virtuales](../virtual-machines/linux/quick-create-cli.md).

Para este tutorial, usaremos lo siguiente:

* un grupo de recursos denominado *saphanaResourceGroup*,
* una máquina virtual llamada *saphanaVM* y
* los recursos de la ubicación *westus2*.

Vamos a crear un almacén denominado *saphanaVault*.

```azurecli-interactive
az backup vault create --resource-group saphanaResourceGroup \
    --name saphanaVault \
    --location westus2
```

De forma predeterminada, el almacén de Recovery Services se establece para el almacenamiento con redundancia geográfica. El almacenamiento con redundancia geográfica garantiza que se repliquen los datos de copia de seguridad en una región de Azure secundaria que se encuentra a cientos de kilómetros de distancia de la región primaria. Si es necesario modificar la configuración de redundancia del almacenamiento, utilice el cmdlet [az backup vault backup-properties set](/cli/azure/backup/vault/backup-properties?view=azure-cli-latest#az-backup-vault-backup-properties-set).

```azurecli
az backup vault backup-properties set \
    --name saphanaVault  \
    --resource-group saphanaResourceGroup \
    --backup-storage-redundancy "LocallyRedundant/GeoRedundant"
```

Para ver si el almacén se creó correctamente, use el cmdlet [az backup vault list](/cli/azure/backup/vault?view=azure-cli-latest#az-backup-vault-list). Visualizará la siguiente respuesta:

```output
Location   Name             ResourceGroup
---------  ---------------  -------------  
westus2    saphanaVault     saphanaResourceGroup
```

## <a name="register-and-protect-the-sap-hana-instance"></a>Registro y protección de la instancia de SAP HANA

Para que los servicios de Azure detecten la instancia de SAP HANA (la máquina virtual que tiene SAP HANA instalado), se debe ejecutar un [script antes del registro](https://aka.ms/scriptforpermsonhana) en la máquina de SAP HANA. Asegúrese de que se cumplen todos los [requisitos previos](./tutorial-backup-sap-hana-db.md#prerequisites) antes de ejecutar el script. Consulte la sección [Qué hace el script de registro previo](tutorial-backup-sap-hana-db.md#what-the-pre-registration-script-does) para más información sobre lo que hace el script.

Una vez ejecutado el script, la instancia de SAP HANA se puede registrar con el almacén de Recovery Services que se creó anteriormente. Para registrar la instancia, use el cmdlet [az backup container register](/cli/azure/backup/container?view=azure-cli-latest#az-backup-container-register). *VMResourceId* es el identificador de recurso de la máquina virtual que creó para instalar SAP HANA.

```azurecli-interactive
az backup container register --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --location westus2 \
    --workload-type SAPHANA \
    --backup-management-type AzureWorkload \
    --resource-id VMResourceId
```

>[!NOTE]
>Si la máquina virtual no está en el mismo grupo de recursos que el almacén, *saphanaResourceGroup* hará referencia al grupo de recursos donde se creó el almacén.

Al registrar la instancia de SAP HANA se detectan automáticamente todas sus bases de datos actuales. Sin embargo, para detectar las nuevas bases de datos que se puedan agregar en el futuro, consulte la sección [Detección de las nuevas bases de datos agregadas a SAP HANA tras su registro](tutorial-sap-hana-manage-cli.md#protect-new-databases-added-to-an-sap-hana-instance) de la instancia.

Para comprobar si la instancia de SAP HANA se ha registrado correctamente en el almacén, use el cmdlet [az backup container list](/cli/azure/backup/container?view=azure-cli-latest#az-backup-container-list). Visualizará la siguiente respuesta:

```output
Name                                                    Friendly Name    Resource Group        Type           Registration Status
------------------------------------------------------  --------------   --------------------  ---------      ----------------------
VMAppContainer;Compute;saphanaResourceGroup;saphanaVM   saphanaVM        saphanaResourceGroup  AzureWorkload  Registered
```

>[!NOTE]
> La columna "Name" de la salida anterior hace referencia al nombre del contenedor. Este nombre de contenedor se usará en las secciones siguientes para habilitar las copias de seguridad y desencadenarlas. En este caso es *VMAppContainer;Compute;saphanaResourceGroup;saphanaVM*.

## <a name="enable-backup-on-sap-hana-database"></a>Habilitación de la copia de seguridad de la base de datos de SAP HANA

El cmdlet [az backup protectable-item list](/cli/azure/backup/protectable-item?view=azure-cli-latest#az-backup-protectable-item-list) enumera todas las bases de datos detectadas en la instancia de SAP HANA que registró en el paso anterior.

```azurecli-interactive
az backup protectable-item list --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --workload-type SAPHANA \
    --output table
```

Debería encontrar la base de datos de la que desea realizar una copia de seguridad en esta lista, que tendrá el siguiente aspecto:

```output
Name                           Protectable Item Type    ParentName    ServerName    IsProtected
-----------------------------  ----------------------   ------------  -----------   ------------
saphanasystem;hxe              SAPHanaSystem            HXE           hxehost       NotProtected  
saphanadatabase;hxe;systemdb   SAPHanaDatabase          HXE           hxehost       NotProtected
saphanadatabase;hxe;hxe        SAPHanaDatabase          HXE           hxehost       NotProtected
```

Como puede ver en la salida anterior, el SID del sistema SAP HANA es HXE. En este tutorial configuraremos la copia de seguridad de la base de datos *saphanadatabase;hxe;hxe* que reside en el servidor *hxehost*.

Para proteger y configurar la copia de seguridad de una base de datos, de una en una, se usa el cmdlet [az backup protection enable-for-azurewl](/cli/azure/backup/protection?view=azure-cli-latest#az-backup-protection-enable-for-azurewl). Proporcione el nombre de la directiva que desee usar. Para crear una directiva mediante la CLI, use el cmdlet [az backup policy create](/cli/azure/backup/policy?view=azure-cli-latest#az-backup-policy-create). Para este tutorial usaremos la directiva *sapahanaPolicy*.

```azurecli-interactive
az backup protection enable-for-azurewl --resource-group saphanaResourceGroup \
    --policy-name saphanaPolicy \
    --protectable-item-name saphanadatabase;hxe;hxe  \
    --protectable-item-type SAPHANADatabse \
    --server-name hxehost \
    --workload-type SAPHANA \
    --output table
```

Puede comprobar si la configuración de copia de seguridad anterior se ha completado con el cmdlet [az backup job list](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-list). La salida aparecerá de la siguiente manera:

```output
Name                                  Operation         Status     Item Name   Start Time UTC
------------------------------------  ---------------   ---------  ----------  -------------------  
e0f15dae-7cac-4475-a833-f52c50e5b6c3  ConfigureBackup   Completed  hxe         2019-12-03T03:09:210831+00:00  
```

El cmdlet [az backup job list](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-list) enumera todos los trabajos de copia de seguridad (programados o a petición) que se han ejecutado o que se están ejecutando actualmente en la base de datos protegida, además de otras operaciones como el registro, la configuración de la copia de seguridad, la eliminación de datos de la copia de seguridad, etc.

>[!NOTE]
>Azure Backup no se ajusta automáticamente a los cambios del horario de verano cuando realiza la copia de seguridad de una base de datos SAP HANA en una máquina virtual de Azure.
>
>Modifique la directiva de forma manual según sea necesario.

## <a name="trigger-an-on-demand-backup"></a>Desencadenamiento de una copia de seguridad a petición

Mientras que en la sección anterior se detalla cómo configurar una copia de seguridad programada, en esta se habla de desencadenar una a petición. Para ello, usamos el cmdlet [az backup protection backup-now](/cli/azure/backup/protection#az-backup-protection-backup-now).

>[!NOTE]
> La directiva de retención de una copia de seguridad a petición viene determinada por la directiva de retención subyacente para la base de datos.

```azurecli-interactive
az backup protection backup-now --resource-group saphanaResourceGroup \
    --item-name saphanadatabase;hxe;hxe \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --backup-type Full
    --retain-until 01-01-2040
    --output table
```

La salida aparecerá de la siguiente manera:

```output
Name                                  ResourceGroup
------------------------------------  -------------
e0f15dae-7cac-4475-a833-f52c50e5b6c3  saphanaResourceGroup
```

La respuesta le proporcionará el nombre del trabajo. Este nombre de trabajo se puede usar para realizar el seguimiento del estado del trabajo mediante el cmdlet [az backup job show](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-show).

>[!NOTE]
>Además de programar una copia de seguridad completa o diferencial, también se pueden desencadenar estas manualmente. Las copias de seguridad de registros se desencadenan y administran desde SAP HANA interna y automáticamente.
>
> Azure Backup no admite actualmente las copias de seguridad incrementales.

## <a name="next-steps"></a>Pasos siguientes

* Para aprender a restaurar una base de datos de SAP HANA en una máquina virtual de Azure mediante la CLI, continúe con el tutorial: [Restauración de una base de datos de SAP HANA en una máquina virtual de Azure con la CLI](tutorial-sap-hana-restore-cli.md)

* Para aprender a realizar una copia de seguridad de una base de datos de SAP HANA que se ejecuta en una máquina virtual de Azure mediante Azure Portal, consulte [Copia de seguridad de bases de datos de SAP HANA en máquinas virtuales de Azure](./backup-azure-sap-hana-database.md)
