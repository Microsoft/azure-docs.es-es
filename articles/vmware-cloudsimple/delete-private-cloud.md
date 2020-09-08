---
title: Eliminación de la nube privada de Azure VMware Solution by CloudSimple
description: Obtenga información sobre cómo eliminar una nube privada de CloudSimple. Al eliminar una nube privada, se eliminarán todos los clústeres.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/06/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 11fda35f5b236a4930b3d90eb7e3a62ea60207cf
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88142237"
---
# <a name="delete-a-cloudsimple-private-cloud"></a>Eliminación de una nube privada de CloudSimple

CloudSimple ofrece la flexibilidad de eliminar una nube privada.  Una nube privada se compone de uno o varios clústeres de vSphere. Cada clúster puede tener de 3 a 16 nodos. Al eliminar una nube privada, se eliminarán todos los clústeres.

## <a name="before-you-begin"></a>Antes de empezar

Con la eliminación de una nube privada se elimina toda la nube privada.  Todos los componentes de la nube privada se eliminan.  Si quiere conservar los datos, asegúrese de que ha realizado una copia de seguridad de ellos en el almacenamiento local o en Azure Storage.

Los componentes de una nube privada incluyen:

* Nodos de CloudSimple
* Máquinas virtuales
* VLAN o subredes
* Todos los datos de usuario almacenados en la nube privada
* Todos los datos adjuntos de reglas de firewall a una VLAN o subred

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="delete-a-private-cloud"></a>Eliminación de una nube privada

1. [Acceda al portal de CloudSimple](access-cloudsimple-portal.md).

2. Abra la página **Resources** (Recursos).

3. Haga clic en la nube privada que quiera eliminar.

4. En la página de resumen, haga clic en **Delete** (Eliminar).

    ![Eliminación de una nube privada](media/delete-private-cloud.png)

5. En la página de confirmación, escriba el nombre de la nube privada y haga clic en **Delete** (Eliminar). 

    ![Eliminación de la nube privada: confirmación](media/delete-private-cloud-confirm.png)

La nube privada está marcada para su eliminación.  El proceso de eliminación se inicia al cabo de tres horas y elimina la nube privada.

> [!CAUTION]
> Los nodos deben eliminarse después de la eliminación de la nube privada.  La medición de los nodos continuará hasta que se eliminen los nodos de la suscripción.

## <a name="next-steps"></a>Pasos siguientes

* [Eliminación de nodos](delete-nodes.md)
