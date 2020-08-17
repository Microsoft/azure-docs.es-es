---
title: Reparación de una máquina virtual Windows mediante los comandos de reparación de máquinas virtuales de Azure | Microsoft Docs
description: En este artículo se detalla cómo utilizar los comandos de reparación de máquinas virtuales de Azure para conectar el disco a otra máquina virtual Windows para corregir los errores y, posteriormente, reconstruir la máquina virtual original.
services: virtual-machines-windows
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: virtual-machines
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: azurecli
ms.date: 09/10/2019
ms.author: v-miegge
ms.openlocfilehash: 7addc87f3096a75a55d0ea3b5804fd0006d5cb8c
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86526493"
---
# <a name="repair-a-windows-vm-by-using-the-azure-virtual-machine-repair-commands"></a>Reparación de una máquina virtual Windows mediante los comandos de reparación de máquinas virtuales de Azure

Si la máquina virtual Windows de Azure se encuentra un error de disco o de arranque, deberá realizar la mitigación en el propio disco. Un ejemplo habitual sería una actualización de aplicación con error que impide que la máquina virtual se pueda arrancar correctamente. En este artículo se detalla cómo utilizar los comandos de reparación de máquinas virtuales de Azure para conectar el disco a otra máquina virtual Windows para corregir los errores y, posteriormente, reconstruir la máquina virtual original.

> [!IMPORTANT]
> * Los scripts de este artículo solo se aplican a las máquinas virtuales que usan [Azure Resource Manager](../../azure-resource-manager/management/overview.md).
> * La conectividad de salida de la máquina virtual (puerto 443) es necesaria para que se ejecute el script.
> * Solo se puede ejecutar un script a la vez.
> * Un script en ejecución no se puede cancelar.
> * El tiempo máximo que se puede ejecutar un script es de noventa minutos después de agotarse el tiempo de espera.
> * En el caso de las máquinas virtuales que usan Azure Disk Encryption, solo se admiten los discos administrados cifrados con el cifrado de un solo paso (con o sin KEK).


## <a name="repair-process-overview"></a>Información general del proceso de reparación

Ahora puede usar los comandos de reparación de máquinas virtuales de Azure para cambiar el disco del sistema operativo de una máquina virtual y ya no necesita eliminar y volver a crear dicha máquina virtual.

Siga estos pasos para solucionar el problema de la máquina virtual:

1. Inicio de Azure Cloud Shell
2. Ejecutar az extension add/update.
3. Ejecutar az vm repair create.
4. Ejecutar az vm repair run.
5. Ejecutar az vm repair restore.

Para documentación e instrucciones adicionales, consulte [az vm repair](/cli/azure/ext/vm-repair/vm/repair).

## <a name="repair-process-example"></a>Ejemplo del proceso de reparación

1. Inicio de Azure Cloud Shell

   Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Incluye las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta.

   Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede abrir Cloud Shell en una pestaña independiente visitando [https://shell.azure.com](https://shell.azure.com).

   Seleccione **Copiar** para copiar los bloques de código, luego pegue el código en Cloud Shell y, por último, seleccione **Entrar** para ejecutarlo.

   Si prefiere instalar y usar la CLI en un entorno local, para esta guía de inicio rápido se requiere la versión 2.0.30 de la CLI de Azure o una versión posterior. Ejecute ``az --version`` para encontrar la versión. Si necesita instalar o actualizar la CLI de Azure, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).
   
   Si tiene que iniciar sesión en Cloud Shell con una cuenta diferente a la que ha iniciado sesión actualmente en Azure Portal, puede usar ``az login`` [desde la referencia de az login](/cli/azure/reference-index?view=azure-cli-latest#az-login).  Para cambiar entre suscripciones asociadas a su cuenta, puede usar ``az account set --subscription`` [de la referencia de az account set](/cli/azure/account?view=azure-cli-latest#az-account-set).

2. Si es la primera vez que usa los comandos `az vm repair`, agregue la extensión de la CLI vm-repair.

   ```azurecli-interactive
   az extension add -n vm-repair
   ```

   Si ha usado anteriormente los comandos `az vm repair`, aplique las actualizaciones a la extensión vm-repair.

   ```azurecli-interactive
   az extension update -n vm-repair
   ```

3. Ejecute `az vm repair create`. Este comando creará una copia del disco del sistema operativo para la máquina virtual no funcional, creará una máquina virtual de reparación en un nuevo grupo de recursos y conectará la copia del disco del sistema operativo.  La máquina virtual de reparación tendrá el mismo tamaño y región que la máquina virtual no funcional especificada. El grupo de recursos y el nombre de la máquina virtual que se usan en todos los pasos serán los de la máquina virtual no funcional. Si la máquina virtual usa Azure Disk Encryption, el comando intentará desbloquear el disco cifrado para que sea accesible cuando se conecte a la máquina virtual de reparación.

   ```azurecli-interactive
   az vm repair create -g MyResourceGroup -n myVM --repair-username username --repair-password password!234 --verbose
   ```

4. Ejecute `az vm repair run`. Este comando ejecutará el script de reparación especificado en el disco conectado a través de la máquina virtual de reparación. Si la guía de solución de problemas que usa especifica un identificador de ejecución, úselo aquí; si no también puede usar `az vm repair list-scripts` para ver los scripts de reparación disponibles. El grupo de recursos y el nombre de la máquina virtual usados aquí son para la máquina virtual no funcional empleada en el paso 3.

   ```azurecli-interactive
   az vm repair run -g MyResourceGroup -n MyVM --run-on-repair --run-id win-hello-world --verbose
   ```

5. Ejecute `az vm repair restore`. Este comando cambiará el disco del sistema operativo que se reparó por el disco del sistema operativo original de la máquina virtual. El grupo de recursos y el nombre de la máquina virtual que se usan aquí son para la máquina virtual no funcional que se emplea en el paso 3.

   ```azurecli-interactive
   az vm repair restore -g MyResourceGroup -n MyVM --verbose
   ```

## <a name="verify-and-enable-boot-diagnostics"></a>Comprobación y habilitación del diagnóstico de arranque

En el ejemplo siguiente se habilita la extensión de diagnóstico en la máquina virtual denominada ``myVMDeployed`` en el grupo de recursos ``myResourceGroup``:

Azure CLI

```azurecli-interactive
az vm boot-diagnostics enable --name myVMDeployed --resource-group myResourceGroup --storage https://mystor.blob.core.windows.net/
```

## <a name="next-steps"></a>Pasos siguientes

* Si tiene problemas para conectarse a la máquina virtual, consulte [Solución de problemas de conexiones RDP a una máquina virtual de Azure](./troubleshoot-rdp-connection.md).
* Para problemas con el acceso a aplicaciones que se ejecutan en su máquina virtual, consulte [Solución de problemas de conectividad de aplicaciones en una máquina virtual en Azure](./troubleshoot-app-connection.md).
* Para más información sobre el uso de Resource Manager, consulte [Información general de Azure Resource Manager](../../azure-resource-manager/management/overview.md).
