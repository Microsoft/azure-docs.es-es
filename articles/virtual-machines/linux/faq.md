---
title: Preguntas más frecuentes sobre VM Linux en Azure
description: Proporciona respuestas a algunas de las preguntas frecuentes sobre las máquinas virtuales de Linux creadas con el modelo de Resource Manager.
author: cynthn
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 05/08/2019
ms.author: cynthn
ms.openlocfilehash: 71bffacb3f865ffe487ebdd4bee0c0c229be332d
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87830332"
---
# <a name="frequently-asked-question-about-linux-virtual-machines"></a>Preguntas frecuentes sobre las máquinas virtuales de Linux
En este artículo se responden algunas preguntas comunes que los usuarios plantean sobre las máquinas virtuales Linux creadas en Azure mediante el modelo de implementación de Resource Manager. Para ver la versión de Windows de este tema, consulte [Preguntas más frecuentes sobre máquinas virtuales Windows](../windows/faq.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

## <a name="what-can-i-run-on-an-azure-vm"></a>¿Qué puedo ejecutar en una máquina virtual de Azure?
Todos los suscriptores pueden ejecutar software de servidor en una máquina virtual de Azure. Para más información, consulte [Linux en distribuciones aprobadas por Azure](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="how-much-storage-can-i-use-with-a-virtual-machine"></a>¿Cuánto almacenamiento puedo usar con una máquina virtual?
Cada disco de datos puede ser de hasta 32 767 GiB. El número de discos de datos que puede usar depende del tamaño de la máquina virtual. Para obtener más información, consulte [Tamaños de máquinas virtuales](../sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

Azure Managed Disks son las nuevas ofertas de almacenamiento en disco recomendadas que se pueden usar con Azure Virtual Machines para almacenar los datos de forma persistente. Puede usar varios de estos discos con cada máquina virtual. Managed Disks ofrece dos tipos de opciones de almacenamiento duraderas: discos administrados premium y estándar. Para más información, consulte los [precios de Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks).

Las cuentas de Azure Storage también pueden proporcionar almacenamiento para el disco del sistema operativo y los discos de datos. Cada disco es un archivo .vhd almacenado como un blob en páginas. Para obtener información detallada sobre los precios, consulte [Detalles de precios de almacenamiento](https://azure.microsoft.com/pricing/details/storage/).

## <a name="how-can-i-access-my-virtual-machine"></a>¿Cómo puedo tener acceso a mi máquina virtual?
Establezca una conexión remota para iniciar sesión en la máquina virtual mediante Secure Shell (SSH). Consulte las instrucciones sobre cómo conectarse [desde Windows](ssh-from-windows.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) o [desde Linux y Mac](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). De forma predeterminada, SSH permite un máximo de 10 conexiones simultáneas. Puede aumentar este número editando el archivo de configuración.

Si tiene problemas, consulte [Solución de problemas de conexiones de Secure Shell (SSH)](../troubleshooting/troubleshoot-ssh-connection.md?toc=/azure/virtual-machines/linux/toc.json).

## <a name="can-i-use-the-temporary-disk-devsdb1-to-store-data"></a>¿Puedo usar el disco temporal (/dev/sdb1) para almacenar datos?
No utilice el disco temporal (/dev/sdb1) para almacenar datos. Solo existe para el almacenamiento temporal. Corre el riesgo de perder datos que no se podrán recuperar.

## <a name="can-i-copy-or-clone-an-existing-azure-vm"></a>¿Puedo copiar o clonar una máquina virtual de Azure existente?
Sí. Para obtener instrucciones, consulte [Creación de una copia de una máquina virtual Linux en el modelo de implementación de Resource Manager](copy-vm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="why-am-i-not-seeing-canada-central-and-canada-east-regions-through-azure-resource-manager"></a>¿Por qué no veo las regiones de Centro de Canadá y Este de Canadá por medio de Azure Resource Manager?
Las dos nuevas áreas Centro de Canadá y Este de Canadá no se registran automáticamente para la creación de máquinas virtuales en las suscripciones de Azure existentes. Este registro se realizará automáticamente cuando se implementa una máquina virtual mediante el Portal de Azure en cualquier otra región usando Azure Resource Manager. Después de implementar una máquina virtual en cualquier otra región de Azure, las áreas nuevas deberán estar disponibles para las máquinas virtuales siguientes.

## <a name="can-i-add-a-nic-to-my-vm-after-its-created"></a>¿Puedo agregar una NIC a mi máquina virtual después de crearla?
Sí, ahora es posible. En primer lugar, la máquina virtual debe detenerse desasignada. A continuación, puede agregar o quitar una NIC (a menos que sea la última en la máquina virtual). 

## <a name="are-there-any-computer-name-requirements"></a>¿Hay algún requisito de nombre de equipo?
Sí. El nombre del equipo puede tener un máximo de 64 caracteres. Consulte el artículo sobre las [convenciones y restricciones de nomenclatura](/azure/architecture/best-practices/resource-naming) para más información sobre la denominación de los recursos.

## <a name="are-there-any-resource-group-name-requirements"></a>¿Existen algunos requisitos para los nombres de grupos de recursos?
Sí. El nombre del grupo de recursos puede tener como máximo 90 caracteres. Consulte el artículo sobre las [convenciones y restricciones de nomenclatura](/azure/architecture/best-practices/resource-naming) para más información sobre los grupos de recursos.

## <a name="what-are-the-username-requirements-when-creating-a-vm"></a>¿Cuáles son los requisitos de nombre de usuario cuando se crea una VM?

Los nombres de usuario deben tener entre 1 y 32 caracteres.

No se permiten los siguientes nombres de usuario:

- `1`
- `123`
- `a`
- `actuser`
- `adm`
- `admin`
- `admin1`
- `admin2`
-`administrator`
- `aspnet`
- `backup`
- `console`
- `david`
- `guest`
- `john`
- `owner`
- `root`
- `server`
- `sql`
- `support_388945a0`
- `support`
- `sys`
- `test`
- `test1`
- `test2`
- `test3`
- `user`
- `user1`
- `user2`
- `user3`
- `user4`
- `user5`
- `video`


## <a name="what-are-the-password-requirements-when-creating-a-vm"></a>¿Cuáles son los requisitos de contraseña cuando se crea una VM?

Hay distintos requisitos de longitud de contraseña, en función de la herramienta que use:
 - Portal: entre 12 y 72 caracteres
 - PowerShell: entre 8 y 123 caracteres
 - CLI: entre 12 y 123 caracteres
 - Plantillas de Azure Resource Manager (ARM): entre 12 y 72 caracteres, y no es permiten caracteres de control
 

Las contraseñas también deben cumplir 3 de estos 4 requisitos de complejidad:

* Deben incluir caracteres en minúsculas.
* Deben incluir caracteres en mayúsculas.
* Deben incluir un dígito.
* Deben incluir un carácter especial (REGEX.MATCH [\W_]).

No se permiten las siguientes contraseñas:

<table>
    <tr>
        <td style="text-align:center">abc@123</td>
        <td style="text-align:center">P@$$w0rd</td>
        <td style="text-align:center">P@ssw0rd</td>
        <td style="text-align:center">P@ssword123</td>
        <td style="text-align:center">Pa$$word</td>
    </tr>
    <tr>
        <td style="text-align:center">pass@word1</td>
        <td style="text-align:center">Password!</td>
        <td style="text-align:center">Password1</td>
        <td style="text-align:center">Password22</td>
        <td style="text-align:center">iloveyou!</td>
    </tr>
</table>