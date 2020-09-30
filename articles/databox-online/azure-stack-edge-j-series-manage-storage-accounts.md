---
title: Administración de cuentas de almacenamiento en Azure Stack Edge Pro con GPU | Microsoft Docs
description: En este artículo se explica cómo usar Azure Portal para administrar cuentas de almacenamiento en Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: a239b36a539cf7be3e880512b753d45ae4050bad
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904545"
---
# <a name="use-the-azure-portal-to-manage-edge-storage-accounts-on-your-azure-stack-edge-pro"></a>Uso de Azure Portal para administrar cuentas de almacenamiento en Azure Stack Edge Pro

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

En este artículo se explica cómo administrar cuentas de almacenamiento en Azure Stack Edge Pro. Azure Stack Edge Pro se puede administrar con Azure Portal o con la interfaz de usuario web local. Use Azure Portal para agregar cuentas de almacenamiento de Edge al dispositivo o eliminarlas.

## <a name="about-edge-storage-accounts"></a>Acerca de las cuentas de almacenamiento de Edge

Puede transferir datos desde el dispositivo Azure Stack Edge Pro a través de los protocolos SMB, NFS o REST. Para transferir datos al almacenamiento de blobs con las API REST, debe crear cuentas de almacenamiento de Edge en Azure Stack Edge Pro. 

Las cuentas de almacenamiento de Edge que agregue en el dispositivo Azure Stack Edge Pro se asignan a cuentas de Azure Storage. Los datos escritos en las cuentas de almacenamiento de Edge se insertan automáticamente en la nube.

A continuación, se muestra un diagrama que detalla los dos tipos de cuentas y cómo fluyen los datos desde cada una de estas cuentas a Azure:

![Diagrama de cuentas de almacenamiento de blobs](media/azure-stack-edge-j-series-manage-storage-accounts/ase-blob-storage.svg)

En este artículo aprenderá a:

> [!div class="checklist"]
> * Adición de una cuenta de almacenamiento de Edge
> * Eliminación de una cuenta de almacenamiento de Edge


## <a name="add-an-edge-storage-account"></a>Adición de una cuenta de almacenamiento de Edge

Para crear una cuenta de almacenamiento de Edge, realice el procedimiento siguiente:

[!INCLUDE [Add an Edge storage account](../../includes/azure-stack-edge-gateway-add-storage-account.md)]

## <a name="delete-an-edge-storage-account"></a>Eliminación de una cuenta de almacenamiento de Edge

Realice los pasos siguientes para eliminar una cuenta de almacenamiento de Edge.

1. Vaya a **Configuración > Cuentas de almacenamiento** en el recurso. En la lista de cuentas de almacenamiento, seleccione la cuenta de almacenamiento que vaya a eliminar. En la barra de comandos superior, seleccione **Eliminar cuenta de almacenamiento**.

    ![Ir a la lista de cuentas de almacenamiento](media/azure-stack-edge-j-series-manage-storage-accounts/delete-edge-storage-account-1.png)

2. En la hoja **Eliminar cuenta de almacenamiento**, confirme la cuenta de almacenamiento que quiere eliminar y seleccione **Eliminar**.

    ![Confirmación y eliminación de la cuenta de almacenamiento](media/azure-stack-edge-j-series-manage-storage-accounts/delete-edge-storage-account-2.png)

La lista de cuentas de almacenamiento se actualiza para reflejar la eliminación.


## <a name="add-delete-a-container"></a>Adición y eliminación de un contenedor

También puede agregar o eliminar los contenedores para estas cuentas de almacenamiento.

Para agregar un contenedor, siga estos pasos:

1. Seleccione la cuenta de almacenamiento que quiera administrar. En la barra de comandos superior, seleccione **+ Agregar contenedor**.

    ![Selección de la cuenta de almacenamiento para agregar el contenedor](media/azure-stack-edge-j-series-manage-storage-accounts/add-container-1.png)

2. Proporcione un nombre para el contenedor. Este contenedor se crea en la cuenta de almacenamiento de Edge y en la cuenta de Azure Storage asignada a esta cuenta. 

    ![Adición de un contenedor de Edge](media/azure-stack-edge-j-series-manage-storage-accounts/add-container-2.png)

La lista de contenedores se actualiza para reflejar el contenedor que se acaba de agregar.

![Lista de contenedores actualizada](media/azure-stack-edge-j-series-manage-storage-accounts/add-container-4.png)

Ahora puede seleccionar un contenedor de esta lista y seleccionar **+ Eliminar contenedor** en la barra de comandos superior. 

![Eliminación de un contenedor](media/azure-stack-edge-j-series-manage-storage-accounts/add-container-3.png)

## <a name="sync-storage-keys"></a>Sincronización de claves de almacenamiento

Puede sincronizar las claves de acceso de las cuentas de almacenamiento de Edge (local) en el dispositivo. 

Para sincronizar la clave de acceso de la cuenta de almacenamiento, siga estos pasos:

1. En el recurso, seleccione la cuenta de almacenamiento que quiera administrar. En la barra de comandos superior, seleccione **Sincronizar claves de almacenamiento**.

    ![Selección de la sincronización de claves de almacenamiento](media/azure-stack-edge-j-series-manage-storage-accounts/sync-storage-key-1.png)

2. Cuando se le pida confirmación, seleccione **Sí**.

    ![Selección de la sincronización de claves de almacenamiento](media/azure-stack-edge-j-series-manage-storage-accounts/sync-storage-key-2.png)

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [administrar usuarios desde Azure Portal](azure-stack-edge-j-series-manage-users.md).
