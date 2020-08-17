---
title: 'Conexión de un disco de datos administrado a una máquina virtual de Windows: Azure'
description: Cómo conectar un disco de datos administrado a una máquina virtual Windows en Azure Portal.
author: roygara
ms.service: virtual-machines-windows
ms.topic: how-to
ms.date: 02/06/2020
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: b100676cd8cfbffd273ea6ad50b63545f2d6037a
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836231"
---
# <a name="attach-a-managed-data-disk-to-a-windows-vm-by-using-the-azure-portal"></a>Conexión de un disco de datos administrado a una máquina virtual Windows en Azure Portal

En este artículo se muestra cómo conectar un nuevo disco de datos administrado a una máquina virtual Windows en Azure Portal. El tamaño de la máquina virtual determina el número de discos que se puede conectar. Para más información, consulte [Tamaños de las máquinas virtuales Linux en Azure](../sizes.md).


## <a name="add-a-data-disk"></a>Agregar un disco de datos

1. Vaya [Azure Portal](https://portal.azure.com) para agregar un disco de datos. Busque y seleccione **Máquinas virtuales**.
2. Seleccione una máquina virtual de la lista.
3. En la página **Máquina virtual**, seleccione **Discos**.
4. En la página **Discos**, seleccione **Agregar disco de datos**.
5. En la lista desplegable para el nuevo disco, seleccione **Crear disco**.
6. En la página **Crear disco administrado**, escriba un nombre para el disco y ajuste las demás opciones de configuración según sea necesario. Seleccione **Crear** cuando haya terminado.
7. En la página **Discos**, seleccione **Guardar** para guardar la configuración del nuevo disco de la máquina virtual.
8. Una vez que Azure crea el disco y lo asocia a la máquina virtual, el nuevo disco aparece en la configuración de disco de la máquina virtual en el apartado **Discos de datos**.


## <a name="initialize-a-new-data-disk"></a>Inicio de un nuevo disco de datos

1. Conéctese a la máquina virtual.
1. Seleccione el menú **Inicio** de Windows en la máquina virtual en ejecución y escriba **diskmgmt.msc** en el cuadro de búsqueda. Se abre la consola **Administración de discos**.
2. Administración de discos reconocerá que hay un nuevo disco sin inicializar y se abrirá la ventana **Inicializar disco**.
3. Asegúrese de que el nuevo disco está seleccionado y seleccione **Aceptar** para inicializarlo.
4. El nuevo disco aparecerá como **sin asignar**. Haga clic con el botón derecho en cualquier lugar del disco y seleccione **Nuevo volumen simple**. Se abre la ventana **Asistente para nuevo volumen simple**.
5. Siga los pasos del asistente, conserve todos los valores predeterminados y, cuando haya terminado, seleccione **Finalizar**.
6. Cierre **Administración de discos**.
7. Se abre una ventana emergente que indica que es necesario dar formato al nuevo disco para poder usarlo. Seleccione **Dar formato al disco**.
8. En la ventana **Dar formato al nuevo disco**, active las opciones y seleccione **Iniciar**.
9. Recibirá una advertencia que indica que al dar formato a los discos se borrarán todos los datos. Seleccione **Aceptar**.
10. Cuando se complete el formato, seleccione **Aceptar**.

## <a name="next-steps"></a>Pasos siguientes

- También puede [conectar un disco de datos mediante PowerShell](attach-disk-ps.md).
- Si la aplicación necesita usar la unidad *D:* para almacenar datos, puede [cambiar la letra de la unidad del disco temporal de Windows](change-drive-letter.md).