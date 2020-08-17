---
title: Creación de una VM Windows desde un VHD especializado en Azure Portal
description: Cree una nueva VM Windows desde un VHD en Azure Portal.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 01/18/2019
ms.author: cynthn
ms.openlocfilehash: 5a541dce94cc25958e3c3a6a058e015c8c5e3db0
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87283255"
---
# <a name="create-a-vm-from-a-vhd-by-using-the-azure-portal"></a>Creación de una máquina virtual a partir de un VHD mediante Azure Portal

Hay varias maneras de crear una máquina virtual en Azure: 

- Si ya tiene un disco duro virtual (VHD) que puede usar o quiere copiar el VHD de una máquina virtual existente para usarlo, puede crear una nueva máquina virtual *adjuntando* el VHD a la nueva máquina virtual como disco del sistema operativo. 

- Puede crear una nueva máquina virtual desde el VHD de una máquina virtual que haya sido eliminada. Por ejemplo, si tiene una máquina virtual de Azure que no funciona correctamente, puede eliminarla y usar su VHD para crear una nueva. Puede volver a usar el mismo VHD o crear una copia del VHD mediante la creación de una instantánea y, a continuación, de un nuevo disco administrado desde la instantánea. Aunque la creación de una instantánea conlleva unos pocos pasos más, conserva el VHD original y le proporcionará una reserva.

- Tome una máquina virtual clásica y use el disco duro virtual para crear una VM nueva que use los discos administrados y el modelo de implementación de Resource Manager. Para obtener los mejores resultados, **detenga** la máquina virtual clásica en Azure Portal antes de crear la instantánea.
 
- Puede crear una máquina virtual de Azure desde un VHD local mediante la carga del VHD local y su conexión a una nueva máquina virtual. Puede utilizar PowerShell u otra herramienta para cargar el VHD a una cuenta de almacenamiento y, a continuación, crear un disco administrado a partir del VHD. Para más información, consulte [Carga de un VHD especializado](create-vm-specialized.md#option-2-upload-a-specialized-vhd). 

No use un disco especializado si desea crear varias máquinas virtuales. En su lugar, para implementaciones más grandes, debe [crear una imagen](capture-image-resource.md) y, a continuación, [usarla para crear varias máquinas virtuales](create-vm-generalized-managed.md).

Se recomienda limitar el número de implementaciones simultáneas desde una sola instantánea o VHD a 20 máquinas virtuales. 

## <a name="copy-a-disk"></a>Copiar un disco

Cree una instantánea y, a continuación, cree un disco a partir de la instantánea. Esta estrategia le permite conservar el VHD original como reserva:

1. En [Azure Portal](https://portal.azure.com), seleccione **Todos los servicios** en el menú de la izquierda.
2. En el cuadro de búsqueda **Todos los servicios**, escriba **discos** y, a continuación, seleccione **discos** para mostrar la lista de discos disponibles.
3. Seleccione el disco que le gustaría utilizar. Aparece la página **Disco** de ese disco.
4. En el menú de la parte superior, seleccione **Crear instantánea**. 
5. Escriba un **nombre** para la instantánea.
6. Elija un **Grupo de recursos** para la instantánea. Puede usar un grupo de recursos existente o crear uno nuevo.
7. En **Tipo de cuenta**, elija almacenamiento **Estándar (HDD)** o **Premium (SSD)** .
8. Cuando haya terminado, seleccione **Crear** para crear la instantánea.
9. Una vez creada la instantánea, seleccione **Crear un recurso** en el menú izquierdo.
10. En el cuadro de búsqueda, escriba **disco administrado** y seleccione **Discos administrados** en la lista.
11. En la página **Discos administrados**, seleccione **Crear**.
12. Escriba un **Nombre** para el disco.
13. Elija un **Grupo de recursos** para el disco. Puede usar un grupo de recursos existente o crear uno nuevo. Esta selección también se usará como el grupo de recursos donde crear la máquina virtual desde el disco.
14. En **Tipo de cuenta**, elija almacenamiento **Estándar (HDD)** o **Premium (SSD)** .
15. En **Tipo de origen**, asegúrese de que está seleccionada la opción **Instantánea**.
16. En el desplegable **Instantánea de origen**, seleccione la instantánea que quiere usar.
17. Realice cualquier otro ajuste que sea necesario y, a continuación, seleccione **Crear** para crear el disco.

## <a name="create-a-vm-from-a-disk"></a>Creación de una VM desde un disco

Una vez que tenga el VHD del disco administrado que quiere usar, puede crear la máquina virtual en el portal:

1. En [Azure Portal](https://portal.azure.com), seleccione **Todos los servicios** en el menú de la izquierda.
2. En el cuadro de búsqueda **Todos los servicios**, escriba **discos** y, a continuación, seleccione **discos** para mostrar la lista de discos disponibles.
3. Seleccione el disco que le gustaría utilizar. Se abre la página **Disco** de ese disco.
4. En la página **Información general**, asegúrese de que **ESTADO DEL DISCO** aparezca como **Sin adjuntar**. Si no es el caso, es posible que tenga que desasociar el disco de la VM o eliminar la VM para liberar el disco.
4. En el menú de la parte superior de la página, seleccione **Crear máquina virtual**.
5. En la página **Aspectos básicos** de la nueva máquina virtual, escriba un **Nombre de máquina virtual** y seleccione un **Grupo de recursos** existente o cree uno nuevo.
6. En **Tamaño**, seleccione **Cambiar tamaño** para acceder a la página **Tamaño**.
7. Seleccione una fila de tamaño de máquina virtual y, a continuación, elija **Seleccionar**.
8. En la página **Redes**, puede dejar que el portal cree todos los recursos nuevos o puede seleccionar una **Red virtual** y un **Grupo de seguridad de red** existentes. El portal crea siempre una nueva interfaz de red y una dirección IP pública para la nueva máquina virtual. 
9. En la página **Administración**, realice los cambios oportunos en las opciones de supervisión.
10. En la página **Configuración de invitado**, agregue extensiones según sea necesario.
11. Seleccione **Revisar y crear** cuando haya terminado. 
12. Si la configuración de la máquina virtual supera la validación, seleccione **Crear** para iniciar la implementación.


## <a name="next-steps"></a>Pasos siguientes

También puede usar PowerShell para [cargar un VHD en Azure y crear una VM especializada](create-vm-specialized.md).


