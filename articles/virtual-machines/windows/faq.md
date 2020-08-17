---
title: Preguntas más frecuentes sobre máquinas virtuales Windows en Azure
description: Proporciona respuestas a algunas de las preguntas frecuentes sobre las máquinas virtuales Windows creadas con el modelo de Resource Manager.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 05/08/2019
ms.author: cynthn
ms.openlocfilehash: d5dda567d0cfa89863731a3e8e651aede4ab5957
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87825198"
---
# <a name="frequently-asked-question-about-windows-virtual-machines"></a>Preguntas más frecuentes sobre máquinas virtuales Windows
En este artículo se responden algunas preguntas frecuentes que los usuarios plantean sobre las máquinas virtuales Windows creadas en Azure mediante el modelo de implementación de Resource Manager. Para la versión de Linux de este tema, vea [Preguntas frecuentes sobre las máquinas virtuales de Linux](../linux/faq.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="what-can-i-run-on-an-azure-vm"></a>¿Qué puedo ejecutar en una máquina virtual de Azure?
Todos los suscriptores pueden ejecutar software de servidor en una máquina virtual de Azure. Para saber más sobre la directiva de soporte de software de servidor de Microsoft ejecutado en Azure, vea [Soporte técnico del software de servidor de Microsoft para máquinas virtuales de Microsoft Azure](https://support.microsoft.com/kb/2721672).

Existen determinadas versiones de Windows 7, Windows 8.1 y Windows 10 para suscriptores de ventajas de MSDN Azure y suscriptores de pago por uso de desarrollo y prueba de MSDN, para tareas de desarrollo y prueba. Para obtener más información, como instrucciones y limitaciones, consulte [Imágenes de cliente de Windows para los suscriptores de MSDN](https://azure.microsoft.com/blog/2014/05/29/windows-client-images-on-azure/). 

## <a name="how-much-storage-can-i-use-with-a-virtual-machine"></a>¿Cuánto almacenamiento puedo usar con una máquina virtual?
Cada disco de datos puede ser de hasta 32 767 GiB. El número de discos de datos que puede usar depende del tamaño de la máquina virtual. Para obtener más información, consulte [Tamaños de máquinas virtuales](../sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Azure Managed Disks son las nuevas ofertas de almacenamiento en disco recomendadas que se pueden usar con Azure Virtual Machines para almacenar los datos de forma persistente. Puede usar varios de estos discos con cada máquina virtual. Managed Disks ofrece dos tipos de opciones de almacenamiento duraderas: discos administrados premium y estándar. Para más información, consulte los [precios de Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks).

Las cuentas de Azure Storage también pueden proporcionar almacenamiento para el disco del sistema operativo y los discos de datos. Cada disco es un archivo .vhd almacenado como un blob en páginas. Para obtener información detallada sobre los precios, consulte [Detalles de precios de almacenamiento](https://azure.microsoft.com/pricing/details/storage/).

## <a name="how-can-i-access-my-virtual-machine"></a>¿Cómo puedo tener acceso a mi máquina virtual?
Establezca una conexión remota mediante conexión a Escritorio remoto (RDP) para una máquina virtual Windows. Para obtener instrucciones, vea [Conexión a una máquina virtual de Azure donde se ejecuta Windows e inicio de sesión en ella](connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Se admite un máximo de 2 conexiones simultáneas, a menos que el servidor esté configurado como un host de sesión de servicios de escritorio remoto.  

Si tiene problemas con el Escritorio remoto, consulte [Solución de problemas de conexiones del Escritorio remoto a una máquina virtual de Azure con Windows](../troubleshooting/troubleshoot-rdp-connection.md?toc=/azure/virtual-machines/windows/toc.json). 

Si está familiarizado con Hyper-V, puede que esté buscando una herramienta similar a VMConnect. Azure no ofrece una herramienta similar porque no admite el acceso de la consola a una máquina virtual.

## <a name="can-i-use-the-temporary-disk-the-d-drive-by-default-to-store-data"></a>¿Puedo usar el disco temporal (la unidad D: de forma predeterminada) para almacenar datos?
No utilice el disco temporal para almacenar datos. Solo proporciona almacenamiento de forma temporal, por lo que se arriesgaría a perder datos que no se pueden recuperar. La pérdida de datos puede ocurrir cuando se mueve la máquina virtual a un host diferente. Cambiar el tamaño de una máquina virtual, actualizar el host o un error de hardware en el host son algunas de las razones por las que se puede mover una máquina virtual.

Si tiene una aplicación que necesita usar la letra de unidad D:, puede reasignar las letras de unidad para que el disco temporal utilice otra distinta a D:. Para obtener instrucciones, consulte [Cambio de la letra de unidad del disco temporal de Windows](change-drive-letter.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).


## <a name="how-can-i-change-the-drive-letter-of-the-temporary-disk"></a>¿Cómo puedo cambiar la letra de la unidad del disco temporal?
Para cambiar la letra de la unidad, mueva el archivo de paginación y reasigne las letras de unidad; sin embargo, debe asegurarse de realizar los pasos en un orden específico. Para obtener instrucciones, consulte [Cambio de la letra de unidad del disco temporal de Windows](change-drive-letter.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).

## <a name="can-i-add-an-existing-vm-to-an-availability-set"></a>¿Puedo agregar una máquina virtual existente a un conjunto de disponibilidad?
No. Si desea que la máquina virtual forme parte de un conjunto de disponibilidad, debe crearla dentro del conjunto. Actualmente no es posible agregar una máquina virtual a un conjunto de disponibilidad una vez creada.

## <a name="can-i-upload-a-virtual-machine-to-azure"></a>¿Puedo cargar una máquina virtual en Azure?
Sí. Para obtener instrucciones, consulte [Migrating on-premises VMs to Azure](on-prem-to-azure.md) (Migración de máquinas virtuales locales a Azure).

## <a name="can-i-resize-the-os-disk"></a>¿Puedo cambiar el tamaño del disco del sistema operativo?
Sí. Para obtener instrucciones, consulte [Cómo ampliar la unidad de sistema operativo de una máquina virtual en un grupo de recursos de Azure](expand-os-disk.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="can-i-copy-or-clone-an-existing-azure-vm"></a>¿Puedo copiar o clonar una máquina virtual de Azure existente?
Sí. Mediante el uso de imágenes administradas, puede crear una imagen de una máquina virtual y luego usar la imagen para crear varias máquinas virtuales nuevas. Para obtener instrucciones, consulte [Creación de una imagen personalizada de una máquina virtual](tutorial-custom-images.md).

## <a name="why-am-i-not-seeing-canada-central-and-canada-east-regions-through-azure-resource-manager"></a>¿Por qué no veo las regiones de Centro de Canadá y Este de Canadá por medio de Azure Resource Manager?

Las dos nuevas áreas Centro de Canadá y Este de Canadá no se registran automáticamente para la creación de máquinas virtuales en las suscripciones de Azure existentes. Este registro se realizará automáticamente cuando se implementa una máquina virtual mediante el Portal de Azure en cualquier otra región usando Azure Resource Manager. Después de implementar una máquina virtual en cualquier otra región de Azure, las áreas nuevas deberán estar disponibles para las máquinas virtuales siguientes.

## <a name="does-azure-support-linux-vms"></a>¿Se admiten máquinas virtuales Linux en Azure?
Sí. Para crear rápidamente una máquina virtual Linux de prueba, consulte [Creación de una máquina virtual de Linux en Azure mediante el Portal](../linux/quick-create-portal.md).

## <a name="can-i-add-a-nic-to-my-vm-after-its-created"></a>¿Puedo agregar una NIC a mi máquina virtual después de crearla?
Sí, ahora es posible. En primer lugar, la máquina virtual debe detenerse desasignada. A continuación, puede agregar o quitar una NIC (a menos que sea la última en la máquina virtual). 

## <a name="are-there-any-computer-name-requirements"></a>¿Hay algún requisito de nombre de equipo?
Sí. El nombre del equipo puede tener un máximo de 15 caracteres. Consulte el artículo sobre las [convenciones y restricciones de nomenclatura](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging) para más información sobre la denominación de los recursos.

## <a name="are-there-any-resource-group-name-requirements"></a>¿Existen algunos requisitos para los nombres de grupos de recursos?
Sí. El nombre del grupo de recursos puede tener como máximo 90 caracteres. Consulte el artículo sobre las [convenciones y restricciones de nomenclatura](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#resource-naming) para más información sobre los grupos de recursos.

## <a name="what-are-the-username-requirements-when-creating-a-vm"></a>¿Cuáles son los requisitos de nombre de usuario cuando se crea una VM?

Los nombres de usuario pueden tener un máximo de 20 caracteres y no pueden terminar con un punto ("."). 

No se permiten los siguientes nombres de usuario:

- `1`
- `123`
- `a`
- `actuser`
- `adm`
- `admin`
- `admin1`
- `admin2`
- `administrator`
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

## <a name="what-are-the-password-requirements-when-creating-a-vm"></a>¿Cuáles son los requisitos de contraseña cuando se crea una VM?

Hay distintos requisitos de longitud de contraseña, en función de la herramienta que use:
 - Portal: entre 12 y 72 caracteres
 - PowerShell: entre 8 y 123 caracteres
 - CLI: entre 12 y 123 caracteres

* Deben incluir caracteres en minúsculas.
* Deben incluir caracteres en mayúsculas.
* Deben incluir un dígito.
* Deben incluir un carácter especial (REGEX.MATCH [\W_]).

No se permiten las siguientes contraseñas:

<table>
    <tr>
        <td>abc@123</td>
        <td>iloveyou!</td>
        <td>P@$$w0rd</td>
        <td>P@ssw0rd</td>
        <td>P@ssword123</td>
    </tr>
    <tr>
        <td>Pa$$word</td>
        <td>pass@word1</td>
        <td>Password!</td>
        <td>Password1</td>
        <td>Password22</td>
    </tr>
</table>