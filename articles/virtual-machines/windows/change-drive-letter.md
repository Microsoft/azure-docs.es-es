---
title: 'Conversión de la unidad D: de una máquina virtual en un disco de datos '
description: 'Se describe cómo cambiar las letras de unidad de una máquina virtual de Windows para poder usar la unidad D: como unidad de datos.'
services: virtual-machines-windows
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 01/02/2018
ms.author: cynthn
ms.openlocfilehash: cae29954211e62601debb35d76f938fb6a92779e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284615"
---
# <a name="use-the-d-drive-as-a-data-drive-on-a-windows-vm"></a>Uso de la unidad de disco D: como unidad de datos en una máquina virtual Windows
Si su aplicación necesita usar la unidad D para almacenar datos, siga estas instrucciones para usar una unidad distinta para el disco temporal. Nunca use el disco temporal para almacenar los datos que desee conservar.

Si se cambia el tamaño o se selecciona **Detener (desasignar)** una máquina virtual, se puede desencadenar la colocación de la máquina virtual en un hipervisor nuevo. Un evento de mantenimiento planificado o no planificado también puede desencadenar esta colocación. En este escenario, el disco temporal se reasignará a la primera letra de unidad disponible. Si tiene una aplicación que necesita específicamente la unidad D:, debe seguir estos pasos para mover temporalmente el archivo pagefile.sys, adjuntar un nuevo disco de datos, asignarle la letra D y devolver el archivo pagefile.sys a la unidad temporal. Cuando lo haya hecho, Azure no devolverá D: si la máquina virtual se desplaza a un hipervisor diferente.

Para obtener más información sobre cómo usa Azure el disco temporal, consulte [Understanding the temporary drive on Microsoft Azure Virtual Machines](/archive/blogs/mast/understanding-the-temporary-drive-on-windows-azure-virtual-machines)

## <a name="attach-the-data-disk"></a>Conectar el disco de datos
En primer lugar, deberá conectar el disco de datos a la máquina virtual. Para hacerlo mediante el portal, consulte [How to attach a managed data disk in the Azure portal](attach-managed-disk-portal.md) (Conexión de un disco de datos administrado en Azure Portal).

## <a name="temporarily-move-pagefilesys-to-c-drive"></a>Mover temporalmente el archivo pagefile.sys a la unidad C
1. Conexión a una máquina virtual. 
2. Haga clic con el botón derecho en el menú **Inicio** y seleccione **Sistema**.
3. En el menú que aparece a la izquierda, seleccione **Configuración avanzada del sistema**.
4. En la sección **Rendimiento**, seleccione **Configuración**.
5. Seleccione la pestaña **Opciones avanzadas** .
6. En la sección **Memoria virtual**, seleccione **Cambiar**.
7. Seleccione la unidad **C**, luego haga clic en **Tamaño administrado por el sistema** y en **Establecer**.
8. Seleccione la unidad **D**, luego haga clic en **Sin archivo de paginación** y, a continuación, en **Establecer**.
9. Haga clic en Aplicar. Aparecerá una advertencia que indica que se debe reiniciar el equipo para que los cambios entren en vigor.
10. Reinicie la máquina virtual.

## <a name="change-the-drive-letters"></a>Cambiar las letras de las unidades
1. Una vez que se reinicia la máquina virtual, vuelva a iniciar sesión en ella.
2. Haga clic en el menú **Inicio**, escriba**diskmgmt.msc** y presione Entrar. Se iniciará la Administración de discos.
3. Haga clic con el botón derecho en **D**, la unidad de almacenamiento temporal, y seleccione **Cambiar la letra y rutas de acceso de unidad**.
4. En Letra de unidad, seleccione una unidad nueva como, por ejemplo, **T** y haga clic en**Aceptar**. 
5. Haga clic con el botón derecho en el disco de datos y seleccione **Cambiar la letra y rutas de acceso de unidad**.
6. En Letra de unidad, seleccione la unidad **D** y haga clic en**Aceptar**. 

## <a name="move-pagefilesys-back-to-the-temporary-storage-drive"></a>Devolver el archivo pagefile.sys a la unidad de almacenamiento temporal
1. Haga clic con el botón derecho en el menú **Inicio** y seleccione **Sistema**.
2. En el menú que aparece a la izquierda, seleccione **Configuración avanzada del sistema**.
3. En la sección **Rendimiento**, seleccione **Configuración**.
4. Seleccione la pestaña **Opciones avanzadas** .
5. En la sección **Memoria virtual**, seleccione **Cambiar**.
6. Seleccione la unidad del sistema operativo **C**, haga clic en **Sin archivo de paginación** y luego en **Establecer**.
7. Seleccione la unidad de almacenamiento temporal **T**, haga clic en **Tamaño administrado por el sistema** y, a continuación, en **Establecer**.
8. Haga clic en **Aplicar**. Aparecerá una advertencia que indica que se debe reiniciar el equipo para que los cambios entren en vigor.
9. Reinicie la máquina virtual.

## <a name="next-steps"></a>Pasos siguientes
* Puede aumentar el almacenamiento disponible para la máquina virtual [conectando un disco de datos adicional](attach-managed-disk-portal.md).
