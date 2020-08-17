---
title: Automated Backup para máquinas virtuales de Azure con SQL Server 2014
description: Se explica la característica Automated Backup para SQL Server 2014 en máquinas virtuales que se ejecutan en Azure. Este artículo trata exactamente sobre las máquinas virtuales que usan Resource Manager.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.assetid: bdc63fd1-db49-4e76-87d5-b5c6a890e53c
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/03/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 8e563e53ad0d5ec90fb9b728c8ffe2d239cf0763
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87920609"
---
# <a name="automated-backup-for-sql-server-2014-virtual-machines-resource-manager"></a>Automated Backup para SQL Server 2014 en máquinas virtuales (Resource Manager)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [SQL Server 2014](automated-backup-sql-2014.md)
> * [SQL Server 2016/2017](automated-backup.md)

Automated Backup configura automáticamente [Automated Backup para Microsoft Azure](https://msdn.microsoft.com/library/dn449496.aspx) para todas las bases de datos existentes y nuevas en una máquina virtual de Azure que ejecuta SQL Server 2014 Standard y Enterprise. Esto permite configurar copias periódicas de bases de datos que usan el almacenamiento duradero de Azure Blob Storage. Automated Backup depende de la [Extensión del agente de IaaS (infraestructura como servicio) de SQL Server](sql-server-iaas-agent-extension-automate-management.md).

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="prerequisites"></a>Requisitos previos
Para utilizar Automated Backup, tenga en cuenta los siguientes requisitos previos:

**Sistema operativo**:

- Windows Server 2012
- Windows Server 2012 R2
- Windows Server 2016

**Edición/versión de SQL Server**:

- SQL Server 2014 Standard
- SQL Server 2014 Enterprise

> [!IMPORTANT]
> Automated Backup funciona con SQL Server 2014. Si usa SQL Server 2016/2017, puede usar Automated Backup v2 para hacer copias de seguridad de las bases de datos. Para obtener más información, vea [Automated Backup v2 para SQL Server 2016 en máquinas virtual de Azure](automated-backup.md).

**Configuración de base de datos**:

- Las bases de datos de destino deben utilizar el modelo de recuperación completa. Para obtener más información sobre el impacto del modelo de recuperación completa en las copias de seguridad, vea [Copia de seguridad en el modelo de recuperación completa](https://technet.microsoft.com/library/ms190217.aspx).
- Las bases de datos de destino deben estar en la instancia predeterminada de SQL Server. La Extensión del agente de IaaS de SQL Server no admite instancias con nombre.

> [!NOTE]
> Automated Backup se basa en la Extensión Agente de IaaS de SQL Server. Las imágenes actuales de la galería de máquinas virtuales de SQL agregan esta extensión de manera predeterminada. Para más información, consulte la [extensión Agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md).

## <a name="settings"></a>Configuración

En la siguiente tabla se describen las opciones que pueden configurarse para Automated Backup. Los pasos de configuración reales varían si usa Azure Portal o comandos de Windows PowerShell de Azure.

| Configuración | Intervalo (valor predeterminado) | Descripción |
| --- | --- | --- |
| **Automated Backup** | Habilitar/deshabilitar (deshabilitado) | Habilita o deshabilita Automated Backup para una máquina virtual de Azure que ejecuta SQL Server 2014 Standard o Enterprise. |
| **Período de retención** | 1-30 días (30 días) | El número de días para retener una copia de seguridad. |
| **Storage Account** | Cuenta de almacenamiento de Azure | Una cuenta de almacenamiento de Azure que usar para almacenar archivos de Automated Backup en el almacenamiento de blobs. Se crea un contenedor en esta ubicación para guardar todos los archivos de copia de seguridad. La convención de nomenclatura del archivo de copia de seguridad incluye la fecha, la hora y el nombre de máquina. |
| **Cifrado** | Habilitar/deshabilitar (deshabilitado) | Habilita o deshabilita el cifrado. Cuando se habilita el cifrado, los certificados usados para restaurar la copia de seguridad se ubican en la cuenta de almacenamiento especificada en el mismo contenedor `automaticbackup` con la misma convención de nomenclatura. Si la contraseña cambia, se genera un nuevo certificado con esa contraseña, pero el certificado antiguo permanece para restaurar copias de seguridad anteriores. |
| **Contraseña** | Texto de contraseña | Una contraseña para claves de cifrado. Esto solo es necesario si se habilita el cifrado. Para restaurar una copia de seguridad cifrada, debe disponer de la contraseña correcta y del certificado relacionado que se usó en el momento en el que se realizó la copia de seguridad. |


## <a name="configure-new-vms"></a>Configuración de máquinas virtuales nuevas

Use Azure Portal para configurar la opción Automated Backup cuando cree una máquina virtual de SQL Server 2014 en el modelo de implementación de Resource Manager.

En la pestaña **Configuración de SQL Server**, desplácese hacia abajo hasta **Copia de seguridad automatizada** y seleccione **Habilitar**. La siguiente captura de pantalla de Azure Portal muestra la opción de configuración **Copia de seguridad automatizada de SQL**.

![Configuración de Automated Backup de SQL en Azure Portal](./media/automated-backup-sql-2014/azure-sql-arm-autobackup.png)

## <a name="configure-existing-vms"></a>Configuración de máquinas virtuales existentes

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

En las máquinas virtuales con SQL Server existentes, desde Azure Portal se pueden habilitar y deshabilitar las copias de seguridad automatizadas, cambiar el período de retención, especificar la cuenta de almacenamiento y habilitar el cifrado. 

Vaya al [recurso de máquinas virtuales SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource) de la máquina virtual con SQL Server 2014 y seleccione **Copias de seguridad**. 

![Automated Backup de SQL para máquinas virtuales existentes](./media/automated-backup-sql-2014/azure-sql-rm-autobackup-existing-vms.png)

Cuando termine, seleccione el botón **Aplicar** de la parte inferior de la página **Copias de seguridad** para guardar los cambios.

Si habilita Automated Backup por primera vez, Azure configura el Agente de IaaS de SQL Server en segundo plano. Durante este tiempo, es posible que Azure Portal no muestre que se ha configurado Automated Backup. Espere unos minutos hasta que el agente se instale y configure. Después, Azure Portal mostrará la configuración nueva.

> [!NOTE]
> También puede usar una plantilla para configurar Automated Backup. Para más información, consulte [la plantilla de inicio rápido de Azure para Automated Backup](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-sql-existing-autobackup-update).

## <a name="configure-with-powershell"></a>Configuración con PowerShell

Puede usar PowerShell para configurar la copia de seguridad automatizada. Antes de comenzar:

- [Descargue e instale la última versión de Azure PowerShell](https://aka.ms/webpi-azps).
- Abra Windows PowerShell y asócielo a su cuenta con el comando **Connect-AzAccount**. 

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

### <a name="install-the-sql-server-iaas-extension"></a>Instalación de la extensión IaaS de SQL Server
Si se aprovisiona una máquina virtual con SQL Server desde Azure Portal, la extensión de IaaS de SQL Server también debe estar instalada. Para determinar si está instalada para la VM, llame al comando **Get-AzVM** y examine la propiedad **Extensions**.

```powershell
$vmname = "vmname"
$resourcegroupname = "resourcegroupname"

(Get-AzVM -Name $vmname -ResourceGroupName $resourcegroupname).Extensions
```

Si la extensión del agente IaaS de SQL Server está instalada, debe aparecer como "SqlIaaSAgent" o "SQLIaaSExtension". **ProvisioningState** para la extensión también debería aparecer como "Correcto".

Si no está instalada o no se ha podido aprovisionar, puede instalarla con el comando siguiente. Además del grupo de recursos y del nombre de VM, también debe especificar la región ( **$region**) en que se encuentra dicha VM. Especifique el tipo de licencia de la máquina virtual con SQL Server, eligiendo entre pago por uso o traiga su propia licencia mediante la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/). Para más información acerca de las licencias, consulte [Modelo de licencia](licensing-model-azure-hybrid-benefit-ahb-change.md). 

```powershell
New-AzSqlVM  -Name $vmname `
    -ResourceGroupName $resourcegroupname `
    -Location $region -LicenseType <PAYG/AHUB>
```

> [!IMPORTANT]
> Si la extensión todavía no está instalada, su instalación reiniciará el servicio de SQL Server.

### <a name="verify-current-settings"></a><a id="verifysettings"></a> Verificación de la configuración actual

Si ha habilitado la copia de seguridad automatizada durante el aprovisionamiento, puede usar PowerShell para comprobar la configuración actual. Ejecute el comando **Get-AzVMSqlServerExtension** y examine la propiedad **AutoBackupSettings**:

```powershell
(Get-AzVMSqlServerExtension -VMName $vmname -ResourceGroupName $resourcegroupname).AutoBackupSettings
```

Debería obtener una salida similar a la siguiente:

```
Enable                      : False
EnableEncryption            : False
RetentionPeriod             : -1
StorageUrl                  : NOTSET
StorageAccessKey            : 
Password                    : 
BackupSystemDbs             : False
BackupScheduleType          : 
FullBackupFrequency         : 
FullBackupStartTime         : 
FullBackupWindowHours       : 
LogBackupFrequency          : 
```

Si la salida muestra que la opción **Habilitar** está establecida en **False**, tiene que habilitar la copia de seguridad automatizada. Lo bueno es que habilita y configura Automated Backup de la misma manera. Vea la sección siguiente para leer esta información.

> [!NOTE] 
> Si comprueba la configuración inmediatamente después de realizar un cambio, es posible que obtenga los valores de configuración anteriores. Espere unos minutos y compruebe la configuración de nuevo para asegurarse de que se hayan aplicado los cambios.

### <a name="configure-automated-backup"></a>Configurar Automated Backup
Puede usar PowerShell para habilitar Automated Backup, así como para modificar su configuración y comportamiento en cualquier momento.

En primer lugar, seleccione o cree una cuenta de almacenamiento para los archivos de copia de seguridad. El script siguiente selecciona una cuenta de almacenamiento o la crea si no existe.

```powershell
$storage_accountname = "yourstorageaccount"
$storage_resourcegroupname = $resourcegroupname

$storage = Get-AzStorageAccount -ResourceGroupName $resourcegroupname `
    -Name $storage_accountname -ErrorAction SilentlyContinue
If (-Not $storage)
    { $storage = New-AzStorageAccount -ResourceGroupName $storage_resourcegroupname `
    -Name $storage_accountname -SkuName Standard_GRS -Location $region }
```

> [!NOTE]
> Automated Backup no permite almacenar las copias de seguridad en Premium Storage, pero pueden realizar copias de seguridad de discos de VM que usan Premium Storage.

Luego use el comando **New-AzVMSqlServerAutoBackupConfig** para habilitar y configurar los valores de Copia de seguridad automatizada para almacenar copias de seguridad en la cuenta de Azure Storage. En este ejemplo, las copias de seguridad se conservan durante 10 días. El segundo comando, **Set-AzVMSqlServerExtension**, actualiza la VM de Azure especificada con esta configuración.

```powershell
$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -Enable `
    -RetentionPeriodInDays 10 -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

La instalación y configuración del agente de Iaas de SQL Server puede tardar algunos minutos.

> [!NOTE]
> Hay otras opciones de **New-AzVMSqlServerAutoBackupConfig** que solo se aplican a SQL Server 2016 y a Copia de seguridad automatizada v2. SQL Server 2014 no admite las siguientes opciones: **BackupSystemDbs**, **BackupScheduleType**, **FullBackupFrequency**, **FullBackupStartHour**, **FullBackupWindowInHours** y **LogBackupFrequencyInMinutes**. Si intenta configurar estas opciones en una máquina virtual de SQL Server 2014, no hay ningún error, pero no se aplica la configuración. Si quiere usar estas opciones en una máquina virtual de SQL Server 2016, vea [Automated Backup v2 para máquinas virtuales de Azure con SQL Server 2016](automated-backup.md).

Para habilitar el cifrado, modifique el script anterior para pasar el parámetro **EnableEncryption** junto con una contraseña (cadena segura) para el parámetro **CertificatePassword**. El siguiente script habilita la configuración de Automated Backup en el ejemplo anterior y agrega cifrado.

```powershell
$password = "P@ssw0rd"
$encryptionpassword = $password | ConvertTo-SecureString -AsPlainText -Force

$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -Enable `
    -EnableEncryption -CertificatePassword $encryptionpassword `
    -RetentionPeriodInDays 10 -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

Para confirmar que se ha aplicado la configuración, [verifique la configuración de Automated Backup](#verifysettings).

### <a name="disable-automated-backup"></a>Deshabilitar Automated Backup

Para deshabilitar Copia de seguridad automatizada, ejecute el mismo script sin el parámetro **-Enable** en el comando **New-AzVMSqlServerAutoBackupConfig**. La ausencia del parámetro **-Enable** indica al comando que deshabilite la característica. Al igual que la instalación, la deshabilitación de Automated Backup puede tardar algunos minutos.

```powershell
$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -ResourceGroupName $storage_resourcegroupname

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

### <a name="example-script"></a>Script de ejemplo

El script siguiente proporciona un conjunto de variables que se pueden personalizar para habilitar y configurar Automated Backup para la VM. En su caso, debe personalizar el script en función de sus requisitos. Por ejemplo, debe realizar cambios si desea deshabilitar la copia de seguridad de bases de datos del sistema o habilitar el cifrado.

```powershell
$vmname = "yourvmname"
$resourcegroupname = "vmresourcegroupname"
$region = "Azure region name such as EASTUS2"
$storage_accountname = "storageaccountname"
$storage_resourcegroupname = $resourcegroupname
$retentionperiod = 10

# ResourceGroupName is the resource group which is hosting the VM where you are deploying the SQL Server IaaS Extension

Set-AzVMSqlServerExtension -VMName $vmname `
    -ResourceGroupName $resourcegroupname -Name "SQLIaasExtension" `
    -Version "2.0" -Location $region

# Creates/use a storage account to store the backups

$storage = Get-AzStorageAccount -ResourceGroupName $resourcegroupname `
    -Name $storage_accountname -ErrorAction SilentlyContinue
If (-Not $storage)
    { $storage = New-AzStorageAccount -ResourceGroupName $storage_resourcegroupname `
    -Name $storage_accountname -SkuName Standard_GRS -Location $region }

# Configure Automated Backup settings

$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -Enable `
    -RetentionPeriodInDays $retentionperiod -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname

# Apply the Automated Backup settings to the VM

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

## <a name="monitoring"></a>Supervisión

Para supervisar Automated Backup en SQL Server 2014, tienes dos opciones principales. Puesto que Automated Backup usa la característica Copia de seguridad de administrada SQL Server, las mismas técnicas de supervisión se aplican a ambos.

En primer lugar, puede sondear el estado mediante una llamada a [msdb.smart_admin.sp_get_backup_diagnostics](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/managed-backup-sp-get-backup-diagnostics-transact-sql). O bien consultar la función con valores de tabla [msdb.smart_admin.fn_get_health_status](https://docs.microsoft.com/sql/relational-databases/system-functions/managed-backup-fn-get-health-status-transact-sql).

> [!NOTE]
> El esquema de Copia de seguridad administrada de SQL Server 2014 es **msdb.smart_admin**. En SQL Server 2016, cambió a **msdb.managed_backup**, y los temas de referencia utilizan este esquema más reciente. Pero para SQL Server 2014, debe seguir usando el esquema **smart_admin** para todos los objetos de Copia de seguridad administrada.

Otra opción es aprovechar las ventajas de la característica integrada Correo electrónico de base de datos para las notificaciones.

1. Llame al procedimiento almacenado [msdb.smart_admin.sp_set_parameter](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/managed-backup-sp-set-parameter-transact-sql) para asignar una dirección de correo electrónico al parámetro **SSMBackup2WANotificationEmailIds**. 
1. Habilite [SendGrid](../../../sendgrid-dotnet-how-to-send-email.md) para enviar los mensajes de correo electrónico desde la VM de Azure.
1. Utilice el nombre de usuario y el servidor SMTP para configurar Correo electrónico de base de datos. Puede configurar Correo electrónico de base de datos en SQL Server Management Studio o con comandos de Transact-SQL. Para obtener más información, consulte [Correo electrónico de base de datos](https://docs.microsoft.com/sql/relational-databases/database-mail/database-mail).
1. [Configurar el Agente SQL Server para que use el Correo electrónico de base de datos](https://docs.microsoft.com/sql/relational-databases/database-mail/configure-sql-server-agent-mail-to-use-database-mail).
1. Compruebe que el puerto SMTP está permitido tanto a través del firewall de VM local como del grupo de seguridad de red para la VM.

## <a name="next-steps"></a>Pasos siguientes

Automated Backup configura Copia de seguridad administrada en Azure Virtual Machines. Por lo tanto, es importante [revisar la documentación de Managed Backup en SQL Server 2014](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure).

Puede encontrar directrices adicionales sobre la copia de seguridad y la restauración para SQL Server en VM de Azure en el siguiente artículo: [Copias de seguridad y restauración para SQL Server en máquinas virtuales de Azure](backup-restore.md).

Para más información acerca de otras tareas de automatización disponibles, consulte la [extensión Agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md).

Para obtener más información sobre cómo ejecutar SQL Server en máquinas virtuales de Azure, vea [Introducción a SQL Server en máquinas virtuales de Azure](sql-server-on-azure-vm-iaas-what-is-overview.md).
