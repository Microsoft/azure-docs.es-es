---
title: Diagnósticos de arranque de máquinas virtuales en Azure | Microsoft Docs
description: Introducción a las dos características de depuración para máquinas virtuales en Azure
services: virtual-machines
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: delhan
ms.openlocfilehash: ef907aea940a4f01475cebf72528b9bcdde50f4d
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89180242"
---
# <a name="how-to-use-boot-diagnostics-to-troubleshoot-virtual-machines-in-azure"></a>Uso de diagnósticos de arranque para solucionar problemas de máquinas virtuales en Azure

Puede haber muchas razones por las que una máquina virtual entra en un estado de imposibilidad de arranque. Para abordar los problemas que surgen con las máquinas virtuales creadas con el modelo de implementación de Resource Manager puede usar las siguientes características de depuración: Compatibilidad con Salida de consola y Captura de pantalla para máquinas virtuales de Azure. 

En las máquinas virtuales Linux, puede ver la salida de su registro de consola desde el portal. En las máquinas virtuales Windows y Linux, Azure le permite ver una captura de pantalla de la máquina virtual desde el hipervisor. Ambas características son compatibles con Azure Virtual Machines en todas las regiones. Tenga en cuenta que las capturas de pantalla y la salida pueden tardar hasta 10 minutos en aparecer en la cuenta de almacenamiento.

Puede seleccionar la opción **Diagnósticos de arranque** para ver el registro y la captura de pantalla.

![Resource Manager](./media/virtual-machines-common-boot-diagnostics/screenshot1.png)

## <a name="common-boot-errors"></a>Errores comunes de arranque

- [0xC000000E](https://support.microsoft.com/help/4010129)
- [0xC000000F](https://support.microsoft.com/help/4010130)
- [0xC0000011](https://support.microsoft.com/help/4010134)
- [0xC0000034](https://support.microsoft.com/help/4010140)
- [0xC0000098](https://support.microsoft.com/help/4010137)
- [0xC00000BA](https://support.microsoft.com/help/4010136)
- [0xC000014C](https://support.microsoft.com/help/4010141)
- [0xC0000221](https://support.microsoft.com/help/4010132)
- [0xC0000225](https://support.microsoft.com/help/4010138)
- [0xC0000359](https://support.microsoft.com/help/4010135)
- [0xC0000605](https://support.microsoft.com/help/4010131)
- [No se encontró un sistema operativo](https://support.microsoft.com/help/4010142)
- [Error de arranque o INACCESSIBLE_BOOT_DEVICE](https://support.microsoft.com/help/4010143)

## <a name="enable-diagnostics-on-a-virtual-machine-created-using-the-azure-portal"></a>Habilitar diagnósticos en una máquina virtual creada mediante Azure Portal

El procedimiento siguiente muestra una máquina virtual de Azure creada con el modelo de implementación de Resource Manager.

En la pestaña **Administración**, en la sección **Supervisión**, asegúrese de que la opción **Diagnósticos de arranque** está activada. En la lista desplegable **Cuenta de almacenamiento de diagnóstico**, seleccione una cuenta de almacenamiento para colocar los archivos de diagnóstico.
 
![Creación de una máquina virtual](./media/virtual-machines-common-boot-diagnostics/enable-boot-diagnostics-vm.png)

> [!NOTE]
> La característica Diagnósticos de arranque no es compatible con las cuenta de almacenamiento premium. Si usa la cuenta de almacenamiento premium con Diagnósticos de arranque, podría recibir el error StorageAccountTypeNotSupported al iniciar la máquina virtual.
>

### <a name="deploying-from-an-azure-resource-manager-template"></a>Implementación a partir de una plantilla de Azure Resource Manager

Si va a realizar la implementación con una plantilla de Azure Resource Manager, vaya al recurso de máquina virtual y anexe la sección de perfil de diagnóstico. Establezca el encabezado de versión de la API en "2015-06-15" o posterior. La versión más reciente es "2018-10-01".

```json
{
  "apiVersion": "2018-10-01",
  "type": "Microsoft.Compute/virtualMachines",
  … 
```

El perfil de diagnóstico le permite seleccionar la cuenta de almacenamiento donde desea colocar estos registros.

```json
    "diagnosticsProfile": {
    "bootDiagnostics": {
    "enabled": true,
    "storageUri": "[concat('https://', parameters('newStorageAccountName'), '.blob.core.windows.net')]"
    }
    }
    }
}
```

Para más información sobre la implementación de recursos mediante plantillas, consulte [Inicio rápido: Creación e implementación de plantillas de Azure Resource Manager mediante Azure Portal](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

## <a name="enable-boot-diagnostics-on-existing-virtual-machine"></a>Habilitación de Diagnósticos de arranque en una máquina virtual existente 

Para habilitar Diagnósticos de arranque en una máquina virtual existente, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y, a continuación, seleccione la máquina virtual.
2. En la sección **Soporte técnico y solución de problemas**, seleccione **Diagnósticos de arranque** y, después, seleccione la pestaña **Configuración**.
3. En **Diagnósticos de arranque**, cambie el estado a **Activado** y en la lista desplegable **Cuenta de almacenamiento** seleccione una cuenta de almacenamiento. 
4. Guarde el cambio.

    ![Actualización de una máquina virtual existente](./media/virtual-machines-common-boot-diagnostics/enable-for-existing-vm.png)

### <a name="enable-boot-diagnostics-using-the-azure-cli"></a>Habilitación de diagnósticos de arranque mediante la CLI de Azure

Puede usar la CLI de Azure para habilitar los diagnósticos de arranque en una máquina virtual de Azure ya existente. Para más información, consulte [az vm boot-diagnostics](/cli/azure/vm/boot-diagnostics?view=azure-cli-latest).
