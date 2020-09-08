---
title: 'Creación de un host de Azure Bastion: Azure Portal'
description: En este artículo, aprenderá a crear un host de Azure Bastion, que proporciona conectividad RDP/SSH a todas las máquinas virtuales de una red virtual.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: how-to
ms.date: 04/24/2020
ms.author: cherylmc
ms.openlocfilehash: 1e0bc9a17a12afac6d91867b19aba0f90b7f6edb
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88270757"
---
# <a name="create-an-azure-bastion-host-using-the-portal"></a>Creación de un host de Azure Bastion mediante el portal

En este artículo se muestra cómo crear un host de Azure Bastion mediante Azure Portal. Una vez que haya aprovisionado el servicio Azure Bastion en la red virtual, ya puede disponer de la completa experiencia de RDP/SSH en todas las máquinas virtuales de la misma red virtual. La implementación de Azure Bastion se realiza por red virtual, no por suscripción o cuenta, ni por máquina virtual.

Puede crear un recurso de host bastión en el portal mediante la especificación de todos los valores de configuración manualmente o mediante los valores de configuración que correspondan a una máquina virtual existente. Para crear un host bastión con configuración de máquina virtual, consulte el artículo de [inicio rápido](quickstart-host-portal.md). Opcionalmente, puede usar [Azure PowerShell](bastion-create-host-powershell.md) para crear un host de Azure Bastion.

## <a name="before-you-begin"></a>Antes de empezar

Bastion está disponible en las regiones públicas de Azure siguientes:

[!INCLUDE [available regions](../../includes/bastion-regions-include.md)]

## <a name="create-a-bastion-host"></a><a name="createhost"></a>Creación de un host de Bastion

Esta sección le ayuda a crear un recurso de Azure Bastion desde Azure Portal.

1. En el menú [Azure Portal](https://portal.azure.com) o en la página **Inicio**, seleccione **Crear un recurso**.

1. En la página **Nuevo**, en el campo *Buscar en Marketplace*, escriba **Bastion** y haga clic en **Entrar** para ir a los resultados de la búsqueda.

1. En los resultados, haga clic en **Bastion**. Asegúrese de que el publicador es *Microsoft* y la categoría es *Redes*.

1. En la página **Bastion**, haga clic en **Crear** para abrir la página **Crear un bastión**.

1. En la página **Create a bastion** (Crear un recurso de Bastion), configure un nuevo recurso de Bastion. Especifique las opciones de configuración del recurso de Bastion.

    ![Creación de un recurso de Bastion](./media/bastion-create-host-portal/settings.png)

    * **Suscripción**: suscripción de Azure que quiere usar para crear un recurso de Bastion.
    * **Grupo de recursos**: grupo de recursos en el que se creará el recurso de Bastion. Si no tiene un grupo de recursos existente, puede crear uno.
    * **Name**: nombre del nuevo recurso de Bastion.
    * **Región**: región pública de Azure en la que se creará el recurso.
    * **Red virtual**: red virtual en la que se creará el recurso de Bastion. Puede crear una red virtual en el portal durante este proceso, o bien usar una ya existente. Si usa una red virtual existente, asegúrese de que tenga suficiente espacio de direcciones libre para adaptarse a los requisitos de subred de Bastion.
    * **Subred**: subred de la red virtual en la que se implementará el nuevo host de Bastion. La subred estará dedicada al host de Bastion y debe tener el nombre **AzureBastionSubnet**. Esta subred debe ser /27 o mayor.
    
       **AzureBastionSubnet** no admite [rutas definidas por el usuario](../virtual-network/virtual-networks-udr-overview.md#custom-routes), pero admite [grupos de seguridad de red](bastion-nsg.md).
    * **Dirección IP pública**: dirección IP pública del recurso de Bastion en la que se accederá a RDP/SSH (a través del puerto 443). Cree una dirección IP pública o use una existente. La dirección debe estar en la misma región que el recurso de Bastion que está creando.
    * **Nombre de dirección IP pública**: nombre del recurso de la dirección IP pública.
    * **SKU de la dirección IP pública**: Esta configuración se rellena de forma predeterminada como **Estándar**. Azure Bastion solo usa o admite la SKU de IP pública estándar.
    * **Asignación**: Esta configuración se rellena de forma predeterminada como **Estática**.

1. Cuando termine de especificar la configuración, haga clic en **Revisar + Crear**. Esto valida los valores. Una vez que se apruebe la validación, podrá comenzar el proceso de creación.
1. En la página **Create a bastion** (Crear un recurso de Bastion), haga clic en **Crear**.
1. Verá un mensaje en el que se le indica que la implementación está en curso. El estado se mostrará en esta página a medida que se creen los recursos. El recurso de Bastion tarda aproximadamente cinco minutos en crearse e implementarse.

## <a name="next-steps"></a>Pasos siguientes

* Para más información, lea las [P+F sobre Bastion](bastion-faq.md).

* Para usar grupos de seguridad de red con la subred de Azure Bastion, consulte [Trabajo con grupos de seguridad de red](bastion-nsg.md).
