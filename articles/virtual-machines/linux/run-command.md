---
title: Ejecución de scripts de shell en una VM Linux de Azure
description: Este tema describe cómo ejecutar scripts dentro de una máquina virtual Linux de Azure mediante la función Ejecutar comando
services: automation
ms.service: virtual-machines
author: bobbytreed
ms.author: robreed
ms.date: 04/26/2019
ms.topic: how-to
manager: carmonm
ms.openlocfilehash: 7e8ccc832cdf12176cd88cce0157c08d8bf92507
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372593"
---
# <a name="run-shell-scripts-in-your-linux-vm-by-using-run-command"></a>Ejecución de scripts de shell en la máquina virtual Linux con Ejecutar comando

La función Ejecutar comando usa el agente de máquina virtual (VM) para ejecutar los scripts de shell de una VM Linux de Azure. Puede usar estos scripts para la administración general de máquinas o aplicaciones. Pueden ayudarle a diagnosticar y corregir rápidamente el acceso a la máquina virtual y los problemas de red, así como a revertir la máquina virtual a un buen estado.

## <a name="benefits"></a>Ventajas

Puede acceder a las máquinas virtuales de varias maneras. Ejecutar comando puede ejecutar scripts en sus máquinas virtuales de forma remota con el agente de VM. La función Ejecutar comando se usa a través de Azure Portal, la [API REST](/rest/api/compute/virtual%20machines%20run%20commands/runcommand) o la [CLI de Azure ](/cli/azure/vm/run-command?view=azure-cli-latest#az-vm-run-command-invoke) para las VM con Linux.

Esta funcionalidad es útil en todos los escenarios en los que quiera ejecutar un script en una máquina virtual. Es uno de los únicos métodos para solucionar problemas y corregir una máquina virtual que no tiene abierto el puerto RDP o SSH debido a una configuración incorrecta de red o de usuario administrativo.

## <a name="restrictions"></a>Restricciones

Las siguientes consideraciones se aplican al usar el Ejecutar comando:

* La salida se limita a los últimos 4096 bytes.
* El tiempo mínimo para ejecutar un script es de aproximadamente 20 segundos.
* Los scripts se ejecutan de forma predeterminada como usuario con privilegios elevados en Linux.
* Solo se puede ejecutar un script a la vez.
* No se admiten los scripts que solicitan información (modo interactivo).
* No se puede cancelar un script en ejecución.
* El tiempo máximo que se puede ejecutar un script es de 90 minutos. Pasado este tiempo, se agotará el tiempo de espera del script.
* La conectividad saliente de la máquina virtual es necesaria para devolver los resultados del script.

> [!NOTE]
> Para poder funcionar correctamente, Ejecutar comando requiere conectividad (puerto 443) a las direcciones IP públicas de Azure. Si la extensión no tiene acceso a estos puntos de conexión, los scripts pueden ejecutarse correctamente, pero no devuelven los resultados. Si va a bloquear el tráfico de la máquina virtual, puede usar las [etiquetas de servicio](../../virtual-network/security-overview.md#service-tags) para permitir el tráfico a las direcciones IP públicas de Azure mediante la etiqueta `AzureCloud`.

## <a name="available-commands"></a>Comandos disponibles

Esta tabla muestra la lista de comandos disponibles para máquinas virtuales Linux. Puede usar el comando **RunShellScript** para ejecutar cualquier script personalizado. Al usar la CLI de Azure o PowerShell para ejecutar un comando, el valor que se proporciona para el parámetro `--command-id` o `-CommandId` debe ser uno de los siguientes valores. Cuando se especifica un valor que no es un comando disponible, se recibe este error:

```error
The entity was not found in this Azure location
```

|**Nombre**|**Descripción**|
|---|---|
|**RunShellScript**|Ejecuta un script de shell de Linux.|
|**ifconfig**| Obtiene la configuración de todas las interfaces de red.|

## <a name="azure-cli"></a>Azure CLI

El siguiente ejemplo utiliza el comando [az vm run-command](/cli/azure/vm/run-command?view=azure-cli-latest#az-vm-run-command-invoke) para ejecutar un script de shell en una máquina virtual Linux de Azure.

```azurecli-interactive
az vm run-command invoke -g myResourceGroup -n myVm --command-id RunShellScript --scripts "apt-get update && apt-get install -y nginx"
```

> [!NOTE]
> Para ejecutar comandos como un usuario diferente, introduzca `sudo -u` para especificar una cuenta de usuario.

## <a name="azure-portal"></a>Azure Portal

Navegue hasta una máquina virtual en [Azure Portal](https://portal.azure.com) y seleccione **Ejecutar comando** en **OPERACIONES**. Se mostrará una lista de los comandos disponibles para ejecutarse en la máquina virtual.

![Lista de comandos](./media/run-command/run-command-list.png)

Elija un comando para ejecutar. Algunos de los comandos pueden tener parámetros de entrada obligatorios u opcionales. Para estos comandos, los parámetros se presentan como campos de texto para que pueda proporcionar los valores de entrada. Para cada comando, puede ver el script que se está ejecutando si expande **Ver script**. **RunShellScript** es diferente de los otros comandos, ya que permite proporcionar sus propios scripts personalizados.

> [!NOTE]
> Los comandos integrados no son editables.

Después de elegir el comando, seleccione **Ejecutar** para ejecutar el script. Una vez finalizada la ejecución del script, se devuelven el resultado y los errores en la ventana de salida. La siguiente captura de pantalla muestra un ejemplo de salida tras ejecutar el comando **ifconfig**.

![Salida del script del comando Ejecutar](./media/run-command/run-command-script-output.png)

### <a name="powershell"></a>PowerShell

En el siguiente ejemplo se utiliza el cmdlet [Invoke-AzVMRunCommand](/powershell/module/az.compute/invoke-azvmruncommand) para ejecutar un script de PowerShell en una VM de Azure. El cmdlet espera que el script al que se hace referencia en el parámetro `-ScriptPath` tenga una ubicación local allí donde se está ejecutando el cmdlet.

```powershell-interactive
Invoke-AzVMRunCommand -ResourceGroupName '<myResourceGroup>' -Name '<myVMName>' -CommandId 'RunPowerShellScript' -ScriptPath '<pathToScript>' -Parameter @{"arg1" = "var1";"arg2" = "var2"}
```

## <a name="limiting-access-to-run-command"></a>Limitación del acceso al comando Ejecutar

Para enumerar los comandos ejecutados o mostrar los detalles de un comando, se requiere el permiso `Microsoft.Compute/locations/runCommands/read`. El rol de [lector](../../role-based-access-control/built-in-roles.md#reader) integrado tiene este permiso, al igual que los roles superiores.

La ejecución de un comando requiere el permiso `Microsoft.Compute/virtualMachines/runCommand/action`. El rol [colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) tiene este permiso, al igual que los roles superiores.

Puede usar uno de los roles [integrados](../../role-based-access-control/built-in-roles.md) o crear uno [personalizado](../../role-based-access-control/custom-roles.md) para usar Ejecutar comando.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre otras maneras de ejecutar comandos y scripts de forma remota en la máquina virtual, consulte [Ejecución de scripts en una máquina virtual Linux](run-scripts-in-vm.md).
