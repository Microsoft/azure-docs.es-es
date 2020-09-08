---
title: 'Eliminación de nodos de VMware Solution by CloudSimple: Azure'
description: Aprenda a eliminar nodos de su instancia de VMWare con implementación de CloudSimple. Los nodos CloudSimple se pueden medir. Los nodos que no se usan se eliminan de Azure Portal.
author: sharaths-cs
ms.author: dikamath
ms.date: 08/05/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 569bc6350b1bfa01228d49d28a1d12e2ab62f6f0
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88142271"
---
# <a name="delete-nodes-from-azure-vmware-solution-by-cloudsimple"></a>Eliminación de nodos de Azure VMware Solution by CloudSimple

Los nodos de CloudSimple se miden una vez que se crean.  Los nodos deben eliminarse para detener la medición.  Los nodos que no se usan se eliminan en Azure Portal.

## <a name="before-you-begin"></a>Antes de empezar

Un nodo solo se puede eliminar en las siguientes condiciones:

* Se elimina una nube privada creada con los nodos.  Para eliminar una nube privada, consulte [Eliminación de una nube privada de Azure VMware Solution by CloudSimple](delete-private-cloud.md).
* El nodo se ha quitado de la nube privada mediante la reducción de esta.  Para reducir una nube privada, consulte [Reducción de una nube privada de Azure VMware Solution by CloudSimple](shrink-private-cloud.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="delete-cloudsimple-node"></a>Eliminación de un nodo de CloudSimple

1. Seleccione **Todos los servicios**.

2. Busque **Nodos CloudSimple**.

   ![Búsqueda de los nodos CloudSimple](media/create-cloudsimple-node-search.png)

3. Seleccione **Nodos CloudSimple**.

4. Seleccione los nodos que no pertenecen a una nube privada para eliminarlos.  En la columna **PRIVATE CLOUD NAME** (NOMBRE DE LA NUBE PRIVADA) se muestra el nombre de la nube privada a la que pertenece el nodo.  Si un nodo no se usa en una nube privada, el valor estará vacío. 

    ![Selección de nodos de CloudSimple](media/select-delete-cloudsimple-node.png)

> [!NOTE]
> Solo se pueden eliminar los nodos que no formen parte de la nube privada.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre la [nube privada](cloudsimple-private-cloud.md).
