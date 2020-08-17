---
title: Registro masivo de máquinas virtuales con SQL en Azure con el proveedor de recursos de VM de SQL | Microsoft Docs
description: Registre de forma masiva máquinas virtuales con SQL Server con el proveedor de recursos de VM de SQL para mejorar la manejabilidad.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 10/21/2019
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 5d1d74f1240f919bb20d8893755a86abc19fa828
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533808"
---
# <a name="register-multiple-sql-virtual-machines-in-azure-with-the-sql-vm-resource-provider"></a>Registro de varias máquinas virtuales con SQL en Azure con el proveedor de recursos de máquinas virtuales con SQL
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se describe cómo registrar de forma masiva máquinas virtuales de SQL Server en Azure con el proveedor de recursos de máquinas virtuales con SQL mediante el cmdlet `Register-SqlVMs` de PowerShell.

El cmdlet `Register-SqlVMs` se puede usar para registrar todas las máquinas virtuales en una lista determinada de suscripciones, grupos de recursos o una lista de máquinas virtuales específicas. El cmdlet registrará las máquinas virtuales en el modo de administración _ligero_ y luego generará un [informe y un archivo de registro](#output-description). 

El proceso de registro no conlleva ningún riesgo, no tiene ningún tiempo de inactividad y no reiniciará SQL Server ni la máquina virtual. 

Para más información sobre el proveedor de recursos, vea [Registro de una máquina virtual con SQL Server en Azure con el proveedor de recursos de máquina virtual con SQL](sql-vm-resource-provider-register.md). 

## <a name="prerequisites"></a>Requisitos previos

Para registrar una máquina virtual con SQL Server con el proveedor de recursos, necesita lo siguiente: 

- Una [suscripción de Azure](https://azure.microsoft.com/free/) que se ha [registrado con el proveedor de recursos](sql-vm-resource-provider-register.md#register-subscription-with-rp) y que contiene máquinas virtuales con SQL Server no registradas. 
- Las credenciales de cliente que se usan para registrar las máquinas virtuales existen en cualquiera de los siguientes roles de Azure: **colaborador de la máquina virtual**, **colaborador** o **propietario**. 
- La versión más reciente de [Azure PowerShell](/powershell/azure/new-azureps-module-az). 
- La versión más reciente de [Az.SqlVirtualMachine](https://www.powershellgallery.com/packages/Az.SqlVirtualMachine/0.1.0).

## <a name="get-started"></a>Introducción

Antes de continuar, primero debe crear una copia local del script, importarlo como un módulo de PowerShell y conectarse a Azure. 

### <a name="create-the-script"></a>Cree el script

Para crear el script, copie el [script completo](#full-script) del final de este artículo y guárdelo localmente como `RegisterSqlVMs.psm1`. 

### <a name="import-the-script"></a>Importación del script

Una vez creado el script, puede importarlo como un módulo en el terminal de PowerShell. 

Abra un terminal de PowerShell administrativo y navegue hasta la ubicación en la que guardó el archivo `RegisterSqlVMs.psm1`. Después, ejecute el siguiente cmdlet de PowerShell para importar el script como un módulo: 

```powershell-interactive
Import-Module .\RegisterSqlVMs.psm1
```

### <a name="connect-to-azure"></a>Conexión con Azure

Use el siguiente cmdlet de PowerShell para conectarse a Azure:

```powershell-interactive
Connect-AzAccount
```


## <a name="register-all-vms-in-a-list-of-subscriptions"></a>Registro de todas las máquinas virtuales de una lista de suscripciones 

Use el siguiente cmdlet para registrar todas las máquinas virtuales con SQL Server en una lista de suscripciones:

```powershell-interactive
Register-SqlVMs -SubscriptionList SubscriptionId1,SubscriptionId2
```

Salida de ejemplo: 

```
Number of subscriptions registration failed for 
because you do not have access or credentials are wrong: 1
Total VMs Found: 10
VMs Already registered: 1
Number of VMs registered successfully: 4
Number of VMs failed to register due to error: 1
Number of VMs skipped as VM or the guest agent on VM is not running: 3
Number of VMs skipped as they are not running SQL Server On Windows: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="register-all-vms-in-a-single-subscription"></a>Registro de todas las máquinas virtuales de una sola suscripción

Use el siguiente cmdlet para registrar todas las máquinas virtuales con SQL Server en una sola suscripción: 

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1
```

Salida de ejemplo:

```
Total VMs Found: 10
VMs Already registered: 1
Number of VMs registered successfully: 5
Number of VMs failed to register due to error: 1
Number of VMs skipped as VM or the  guest agent on VM is not running: 2
Number of VMs skipped as they are not running SQL Server On Windows: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="register-all-vms-in-multiple-resource-groups"></a>Registro de todas las máquinas virtuales de varios grupos de recursos

Use el siguiente cmdlet para registrar todas máquinas virtuales con SQL Server en varios grupos de recursos dentro de una sola suscripción:

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1 -ResourceGroupList ResourceGroup1,ResourceGroup2
```

Salida de ejemplo:

```
Total VMs Found: 4
VMs Already registered: 1
Number of VMs registered successfully: 1
Number of VMs failed to register due to error: 1
Number of VMs skipped as they are not running SQL Server On Windows: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="register-all-vms-in-a-resource-group"></a>Registro de todas las máquinas virtuales de un grupo de recursos

Use el siguiente cmdlet para registrar todas las máquinas virtuales con SQL Server en un solo grupo de recursos: 

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1 -ResourceGroupName ResourceGroup1
```

Salida de ejemplo:

```
Total VMs Found: 4
VMs Already registered: 1
Number of VMs registered successfully: 1
Number of VMs failed to register due to error: 1
Number of VMs skipped as VM or the guest agent on VM is not running: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="register-specific-vms-in-a-single-resource-group"></a>Registro de VM específicas de un solo grupo de recursos

Use el siguiente cmdlet para registrar máquinas virtuales con SQL Server específicas dentro de un solo grupo de recursos:

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1 -ResourceGroupName ResourceGroup1 -VmList VM1,VM2,VM3
```

Salida de ejemplo:

```
Total VMs Found: 3
VMs Already registered: 0
Number of VMs registered successfully: 1
Number of VMs skipped as VM or the guest agent on VM is not running: 1
Number of VMs skipped as they are not running SQL Server On Windows: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="register-a-specific-vm"></a>Registro de una máquina virtual específica

Use el siguiente cmdlet para registrar una máquina virtual con SQL Server específica: 

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1 -ResourceGroupName ResourceGroup1 -Name VM1
```

Salida de ejemplo:

```
Total VMs Found: 1
VMs Already registered: 0
Number of VMs registered successfully: 1

Please find the detailed report in  file RegisterSqlVMScriptReport1571314821.txt
```


## <a name="output-description"></a>Descripción de la salida

Se genera un informe y un archivo de registro cada vez que se usa el cmdlet `Register-SqlVMs`. 

### <a name="report"></a>Informe

El informe se genera como un archivo `.txt` denominado `RegisterSqlVMScriptReport<Timestamp>.txt` en el que la marca de tiempo es la hora en la que se inició el cmdlet. El informe enumera los detalles siguientes:

| **Valor de salida** | **Descripción** |
| :--------------  | :-------------- | 
| No se pudo registrar el número de suscripciones porque no tiene acceso o las credenciales son incorrectas | Esto proporciona el número y la lista de suscripciones que tenían problemas con la autenticación proporcionada. El error detallado se puede encontrar en el registro buscando el identificador de la suscripción. | 
| Número de suscripciones que no se pudieron intentar porque no están registradas en RP | Esta sección contiene el recuento y la lista de suscripciones que no se han registrado en el proveedor de recursos de la máquina virtual de SQL. |
| Máquinas virtuales totales encontradas | Recuento de máquinas virtuales que se encontraron en el ámbito de los parámetros pasados al cmdlet. | 
| Máquinas virtuales ya registradas | Recuento de máquinas virtuales que se omitieron porque ya estaban registradas con el proveedor de recursos. |
| Número de máquinas virtuales registradas correctamente | Recuento de máquinas virtuales que se registraron correctamente después de ejecutar el cmdlet. Enumera las máquinas virtuales registradas en el formato `SubscriptionID, Resource Group, Virtual Machine`. | 
| Número de máquinas virtuales que no se pudieron registrar debido a un error | Recuento de máquinas virtuales que no se pudieron registrar debido a algún error. Los detalles del error se pueden encontrar en el archivo de registro. | 
| Número de máquinas virtuales omitidas porque la máquina virtual o el agente invitado de la máquina virtual no se está ejecutando | Recuento y la lista de máquinas virtuales que no se pudieron registrar porque la máquina virtual o el agente invitado de la máquina virtual no se estaba ejecutando. Se pueden reintentar una vez que se haya iniciado la máquina virtual o el agente invitado. Se puede encontrar más información en el archivo de registro. |
| Número de máquinas virtuales omitidas porque no están ejecutando SQL Server en Windows | Recuento de máquinas virtuales que se omitieron porque no están ejecutando SQL Server o no son una máquina virtual de Windows. Las máquinas virtuales se enumeran en el formato `SubscriptionID, Resource Group, Virtual Machine`. | 
| &nbsp; | &nbsp; |

### <a name="log"></a>Log 

Los errores se registran en el archivo de registro denominado `VMsNotRegisteredDueToError<Timestamp>.log`, donde la marca de tiempo es la hora a la que se inició el script. Si el error se encuentra en el nivel de suscripción, el registro contiene el identificador de suscripción separado por comas y el mensaje de error. Si el error está en el registro de la máquina virtual, el registro contiene el identificador de la suscripción, el nombre del grupo de recursos, el nombre de la máquina virtual, el código de error y el mensaje, separados por comas. 

## <a name="remarks"></a>Observaciones

Al registrar máquinas virtuales con SQL Server con el proveedor de recursos mediante el script proporcionado, tenga en cuenta lo siguiente:

- El registro con el proveedor de recursos requiere un agente invitado que se ejecute en la máquina virtual con SQL Server. Las imágenes de Windows Server 2008 no tienen un agente invitado, por lo que se producirá un error en estas máquinas virtuales y se deben registrar manualmente mediante el [modo de administración NoAgent](sql-vm-resource-provider-register.md#management-modes).
- Hay una lógica de reintento integrada para solucionar errores transparentes. Si la máquina virtual se ha registrado correctamente, es una operación rápida. Sin embargo, si se produce un error en el registro, se reintentará cada máquina virtual.  Como tal, debe permitir un tiempo considerable para completar el proceso de registro (aunque el requisito de tiempo real depende del tipo y número de errores). 

## <a name="full-script"></a>Script completo

Para el script completo en GitHub, consulte [Registro de forma masiva de máquinas virtuales con SQL con Az PowerShell](https://github.com/Azure/azure-docs-powershell-samples/blob/master/sql-virtual-machine/register-sql-vms/RegisterSqlVMs.psm1). 

Copie el script completo y guárdelo como `RegisterSqLVMs.psm1`.

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-virtual-machine/register-sql-vms/RegisterSqlVMs.psm1 "Bulk register SQL Server virtual machines")]

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos. 

* [Introducción a SQL Server en máquinas virtuales Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Preguntas más frecuentes de SQL Server en máquinas virtuales Windows](frequently-asked-questions-faq.md)
* [Orientación de precios de SQL Server para máquinas virtuales de Azure](pricing-guidance.md)
* [Notas de la versión de SQL Server en Azure Virtual Machines](../../database/doc-changes-updates-release-notes.md)
