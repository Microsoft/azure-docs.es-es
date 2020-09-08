---
title: 'Azure VMware Solution by CloudSimple: creación de un servicio de CloudSimple'
description: Obtenga información sobre cómo crear el servicio CloudSimple en Azure Portal. Revise la configuración necesaria antes de empezar.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 2a196e717dd6e02570e4bdf830a24ce342b47ece
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88140554"
---
# <a name="create-the-azure-vmware-solution-by-cloudsimple-service"></a>Creación de un servicio de Azure VMware Solution by CloudSimple

Para empezar a trabajar con Azure VMware Solution by CloudSimple, cree este servicio en Azure Portal.

## <a name="before-you-begin"></a>Antes de empezar

Asigne un bloque CIDR /28 para la subred de puerta de enlace. Se requiere una subred de puerta de enlace por cada servicio CloudSimple y es única para la región en la que se crea. La subred de puerta de enlace se usa para los servicios de red perimetral y requiere un bloque CIDR /28. El espacio de direcciones de subred de puerta de enlace debe ser único. No se debe superponer con otras redes que se comuniquen con el entorno de CloudSimple. Las redes que se comunican con CloudSimple incluyen redes locales y redes virtuales de Azure.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="create-the-service"></a>Creación del servicio

1. Seleccione **Todos los servicios**.
2. Busque **Servicios de CloudSimple**.
    ![Búsqueda del servicio CloudSimple](media/create-cloudsimple-service-search.png)
3. Seleccione **Servicios CloudSimple**.
4. Haga clic en **Agregar** para crear un servicio.
    ![Adición del servicio CloudSimple](media/create-cloudsimple-service-add.png)
5. Seleccione la suscripción en la que quiera crear el servicio CloudSimple.
6. Seleccione el grupo de recursos del servicio. Para agregar un nuevo grupo de recursos, haga clic en **Crear nuevo**.
7. Escriba un nombre para identificar el servicio.
8. Escriba el CIDR de la puerta de enlace de servicio. Especifique una subred /28 que no se superponga con ninguna de sus subredes locales, las subredes de Azure ni las subredes planeadas de CloudSimple. No puede cambiar el CIDR después de crear el servicio.

    ![Creación del servicio CloudSimple](media/create-cloudsimple-service.png)
9. Haga clic en **OK**.

El servicio se crea y se agrega a la lista de servicios.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [aprovisionar nodos](create-nodes.md).
* Aprenda a [crear una nube privada](create-private-cloud.md)
* Aprenda a [configurar un entorno de nube privada](quickstart-create-private-cloud.md)
