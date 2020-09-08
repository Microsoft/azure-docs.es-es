---
title: Uso del Explorador de Azure Storage para administrar discos administrados de Azure
description: Aprenda a cargar, descargar y migrar un disco administrado de Azure entre regiones, y también a crear una instantánea de un disco administrado, mediante el Explorador de Azure Storage.
author: roygara
ms.author: rogarana
ms.date: 09/25/2019
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: 9dfce7b76eed5bfc9f4979c0e3041b6c65c28422
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88749376"
---
# <a name="use-azure-storage-explorer-to-manage-azure-managed-disks"></a>Uso del Explorador de Azure Storage para administrar discos administrados de Azure

Explorador de Storage 1.10.0 permite a los usuarios cargar, descargar y copiar discos administrados, así como crear instantáneas. Debido a estas capacidades adicionales, puede usar Explorador de Storage para migrar datos desde una ubicación local a Azure y migrar datos entre regiones de Azure.

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesitará lo siguiente:
- Una suscripción de Azure
- Uno o varios discos administrados de Azure
- La versión más reciente de [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/)

## <a name="connect-to-an-azure-subscription"></a>Conexión a una suscripción de Azure

Si el Explorador de Storage no está conectado a Azure, no podrá usarlo para administrar los recursos. En esta sección se explica cómo conectarse a su cuenta de Azure para que pueda administrar los recursos mediante Explorador de Storage.

1. Inicie Explorador de Azure Storage y haga clic en el icono de **complemento** de la izquierda.

    ![Haga clic en el icono del complemento.](media/disks-upload-vhd-to-managed-disk-storage-explorer/plug-in-icon.png)

1. Seleccione **Agregar una cuenta de Azure** y haga clic en **Siguiente**.

    ![Agregar una cuenta de Azure](media/disks-upload-vhd-to-managed-disk-storage-explorer/connect-to-azure.png)

1. En el cuadro de diálogo **Iniciar sesión en Azure**, escriba sus credenciales de Azure.

    ![Cuadro de diálogo de inicio de sesión en Azure](media/disks-upload-vhd-to-managed-disk-storage-explorer/sign-in.png)

1. Seleccione la suscripción en la lista y luego haga clic en **Aplicar**.

    ![Seleccione su suscripción](media/disks-upload-vhd-to-managed-disk-storage-explorer/select-subscription.png)

## <a name="upload-a-managed-disk-from-an-on-prem-vhd"></a>Cargar un disco administrado desde un VHD local

1. En el panel izquierdo, expanda **Discos** y seleccione el grupo de recursos en el que desea cargar el disco.

    ![Seleccionar el grupo de recursos 1](media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg1.png)

1. Seleccione **Cargar**.

    ![Selección de carga](media/disks-upload-vhd-to-managed-disk-storage-explorer/upload-button.png)

1. En **Cargar VHD** especifique el VHD de origen, el nombre del disco, el tipo de sistema operativo, la región a la que desea cargar el disco y el tipo de cuenta. En algunas regiones, se admiten zonas de disponibilidad, para las cuales puede seleccionar la zona deseada.
1. Seleccione **Crear** para empezar a cargar el disco.

    ![Cuadro de diálogo Cargar VHD](media/disks-upload-vhd-to-managed-disk-storage-explorer/upload-vhd-dialog.png)

1. El estado de la carga ahora se mostrará en **Actividades**.

    ![Estado de la carga](media/disks-upload-vhd-to-managed-disk-storage-explorer/activity-uploading.png)

1. Si la carga ha finalizado y no ve el disco en el panel derecho, seleccione **Actualizar**.

## <a name="download-a-managed-disk"></a>Descargar un disco administrado

En los pasos siguientes se explica cómo descargar un disco administrado en un VHD local. Para que un disco se pueda descargar, su estado debe ser **Sin conectar**. No puede descargar un disco con el estado **Conectado**.

1. En el panel izquierdo, si aún no está expandido, expanda **Discos** y seleccione el grupo de recursos desde el que desea descargar el disco.

    ![Seleccionar el grupo de recursos 1](media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg1.png)

1. En el panel derecho, seleccione el disco que desea descargar.
1. Seleccione **Descargar** y elija dónde desea guardar el disco.

    ![Descargar un disco administrado](media/disks-upload-vhd-to-managed-disk-storage-explorer/download-button.png)

1. Seleccione **Guardar** y el disco comenzará a descargarse. El estado de la descarga se mostrará en **Actividades**.

    ![Estado de descarga](media/disks-upload-vhd-to-managed-disk-storage-explorer/activity-downloading.png)

## <a name="copy-a-managed-disk"></a>Copia de un disco administrado

Con Explorador de Storage, puede copiar un disco superpuesto dentro o entre regiones. Para copiar un disco:

1. En la lista desplegable **Discos** de la izquierda, seleccione el grupo de recursos que contiene el disco que desea copiar.

    ![Seleccionar el grupo de recursos 1](media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg1.png)

1. En el panel derecho, elija el disco que desea copiar y seleccione **Copiar**.

    ![Copia de un disco administrado](media/disks-upload-vhd-to-managed-disk-storage-explorer/copy-button.png)

1. En el panel izquierdo, seleccione el grupo de recursos en el que desea pegar el disco.

    ![Seleccionar el grupo de recursos 2](media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg2.png)

1. Seleccione **Pegar** en el panel derecho.

    ![Pegar un disco administrado](media/disks-upload-vhd-to-managed-disk-storage-explorer/paste-button.png)

1. En el cuadro de diálogo **Pegar un disco**, rellene los valores. También puede especificar una zona de disponibilidad en las regiones admitidas.

    ![Cuadro de diálogo Pegar un disco](media/disks-upload-vhd-to-managed-disk-storage-explorer/paste-disk-dialog.png)

1. Seleccione **Pegar** y el disco comenzará a copiarse. El estado se muestra en **Actividades**.

    ![Estado de copia y pegado](media/disks-upload-vhd-to-managed-disk-storage-explorer/activity-copying.png)

## <a name="create-a-snapshot"></a>Crear una instantánea

1. En la lista desplegable **Discos** de la izquierda, seleccione el grupo de recursos que contiene el disco del que desea crear una instantánea.

    ![Seleccionar el grupo de recursos 1](media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg1.png)

1. A la derecha, seleccione el disco del que desea crear una instantánea y seleccione **Crear instantánea**.

    ![Crear una instantánea](media/disks-upload-vhd-to-managed-disk-storage-explorer/create-snapshot-button.png)

1. En **Crear instantánea**, especifique el nombre de la instantánea, así como el grupo de recursos en el que desea crearla. Seleccione **Crear**.

    ![Cuadro de diálogo Crear instantánea](media/disks-upload-vhd-to-managed-disk-storage-explorer/create-snapshot-dialog.png)

1. Una vez creada la instantánea, puede seleccionar **Abrir en el portal** en **Actividades** para ver la instantánea en el Azure Portal.

    ![Abrir instantánea en el portal](media/disks-upload-vhd-to-managed-disk-storage-explorer/open-in-portal.png)

## <a name="next-steps"></a>Pasos siguientes


Aprenda a [crear una máquina virtual a partir de un disco duro virtual mediante Azure Portal](windows/create-vm-specialized-portal.md).

Aprenda a [conectar un disco de datos administrado a una máquina virtual Windows con Azure Portal](windows/attach-managed-disk-portal.md).