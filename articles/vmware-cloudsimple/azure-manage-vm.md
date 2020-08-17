---
title: 'Azure VMware Solution by CloudSimple: administración de máquinas virtuales de nube privada de Azure'
description: Se describe cómo administrar máquinas virtuales de la nube privada de CloudSimple en Azure Portal, por ejemplo, agregar discos, cambiar la capacidad de la máquina virtual y agregar interfaces de red.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 82aa1ba6ec20e8242568a2725899ddc515a4dc0f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082076"
---
# <a name="manage-your-cloudsimple-private-cloud-virtual-machines-in-azure"></a>Administración de las máquinas virtuales de la nube privada de CloudSimple en Azure

Para administrar las máquinas virtuales que [creó para la nube privada de CloudSimple](azure-create-vm.md), inicie sesión en [Azure Portal](https://portal.azure.com). Busque y seleccione la máquina virtual (busque en **Todos los servicios** o **Máquinas virtuales** en el menú lateral).

## <a name="control-virtual-machine-operation"></a>Control del funcionamiento de la máquina virtual

Los siguientes controles están disponibles en la página **Overview** (Información general) de la máquina virtual seleccionada.

| Control | Descripción |
| ------------ | ------------- |
| Conectar | Conecta a la máquina virtual especificada.  |
| Start | Inicia la máquina virtual especificada.  |
| Reinicio | Apaga y enciende la máquina virtual especificada.  |
| Stop | Apaga la máquina virtual específica.  |
| Capturar | Captura una imagen de la máquina virtual especificada para que se pueda usar para crear otras máquinas virtuales. Consulte [Creación de una imagen administrada de una máquina virtual generalizada en Azure](../virtual-machines/windows/capture-image-resource.md).   |
| Move | Mueve a la máquina virtual especificada.  |
| Eliminar | Quita la máquina virtual especificada.  |
| Actualizar | Actualiza los datos de la pantalla.  |

### <a name="view-performance-information"></a>Visualización de información de rendimiento

Los gráficos del área inferior de la página **Overview** (Información general) presentan datos de rendimiento para el intervalo seleccionado (desde última hora hasta últimos 30 días; el valor predeterminado es última hora). Dentro de cada gráfico, puede mostrar los valores numéricos de cualquier hora dentro del intervalo con solo mover el cursor adelante y atrás.

Se muestran los siguientes gráficos.

| Elemento | Descripción |
| ------------ | ------------- |
| CPU (average) (CPU [promedio]) | Porcentaje de uso promedio de la CPU durante el intervalo seleccionado.   |
| Red | Tráfico entrante y saliente de la red (MB) durante el intervalo seleccionado.  |
| Disk Bytes (Bytes de disco) | Datos totales leídos y escritos en el disco (MB) durante el intervalo seleccionado.  |
| Disk Operations (Operaciones de disco) | Velocidad media de las operaciones de disco (operaciones/segundo) durante el intervalo seleccionado. |

## <a name="manage-vm-disks"></a>Administración de discos de máquinas virtuales

Para agregar un disco de máquina virtual, abra la página **Disks** (Discos) de la máquina virtual seleccionada. Para agregar un disco, haga clic en **Add disk** (Agregar disco). Para configurar cada uno de los siguientes valores, escriba o seleccione una opción insertada. Haga clic en **Save**(Guardar).

   | Elemento | Descripción |
   | ------------ | ------------- |
   | Nombre | Escriba un nombre para identificar el disco.  |
   | Size | Seleccione uno de los tamaños disponibles.  |
   | SCSI Controller (Controladora SCSI) | Seleccione una controladora SCSI. Las controladoras disponibles varían según los distintos sistemas operativos admitidos.  |
   | Mode | Determina cómo participa el disco en las instantáneas. Elija una de estas opciones: <br> - Independent persistent (Independiente persistente): todos los datos escritos en el disco se escriben de forma permanente.<br> - Independent non-persistent (Independiente no persistente): los cambios escritos en el disco se descartan cuando se apaga o restablece la máquina virtual.  Este modo le permite reiniciar siempre la máquina virtual en el mismo estado. Para más información, consulte la [documentación de VMware](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-8B6174E6-36A8-42DA-ACF7-0DA4D8C5B084.html). |

Para eliminar un disco, selecciónelo y haga clic en **Delete** (Eliminar).

## <a name="change-the-capacity-of-the-vm"></a>Cambio de la capacidad de la máquina virtual

Para cambiar la capacidad de la máquina virtual, abra la página **Size** (Tamaño) de la máquina virtual seleccionada. Especifique una de las siguientes opciones y haga clic en **Save** (Guardar).

| Elemento | Descripción |
| ------------ | ------------- |
| Número de núcleos | Número de núcleos asignados a la máquina virtual.  |
| Hardware virtualization (Virtualización de hardware) | Active la casilla para exponer la virtualización de hardware al SO invitado. Consulte el artículo de VMware [Exponer la virtualización asistida por hardware de VMware](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-2A98801C-68E8-47AF-99ED-00C63E4857F6.html). |
| Tamaño de memoria | Seleccione la cantidad de memoria que se va a asignar a la máquina virtual.  

## <a name="manage-network-interfaces"></a>Administración de las interfaces de red

Para agregar una interfaz, haga clic en **Add network interface** (Agregar interfaz de red). Para configurar cada uno de los siguientes valores, escriba o seleccione una opción insertada. Haga clic en **Save**(Guardar).

   | Control | Descripción |
   | ------------ | ------------- |
   | Nombre | Escriba un nombre para identificar la interfaz.  |
   | Red | Seleccione de la lista de redes configuradas en la nube privada de vSphere.  |
   | Adapter (Adaptador) | Seleccione un adaptador de vSphere en la lista de tipos disponibles configurados para la máquina virtual. Para más información, consulte el artículo de la base de conocimiento de VMware [Elegir un adaptador de red para la máquina virtual](https://kb.vmware.com/s/article/1001805). |
   | Power on at Boot (Encender al arrancar) | Elija si quiere habilitar el hardware de NIC al arrancar la máquina virtual. El valor predeterminado es **Habilitar**. |

Para eliminar una interfaz de red, selecciónela y haga clic en **Delete** (Eliminar).
