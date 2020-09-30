---
title: 'Administración de redes virtuales mediante Azure Portal en Azure Database for MySQL: Servidor flexible'
description: 'Creación y administración de redes virtuales para Azure Database for MySQL: Servidor flexible mediante Azure Portal'
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: 28506ea5afe085e9f52e40495d9931ae0aeddf32
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931902"
---
# <a name="create-and-manage-virtual-networks-for-azure-database-for-mysql---flexible-server-using-the-azure-portal"></a>Creación y administración de redes virtuales para Azure Database for MySQL: Servidor flexible mediante Azure Portal

> [!IMPORTANT]
> Actualmente, Azure Database for MySQL: Servidor flexible se encuentra en versión preliminar pública.

El Servidor flexible de Azure Database for MySQL admite dos métodos de conectividad de red mutuamente excluyentes para conectarse al servidor flexible. Las dos opciones son:

- Acceso público (direcciones IP permitidas)
- Acceso privado (integración con red virtual)

En este artículo nos vamos a centrar en la creación de un servidor de MySQL con **acceso privado (Integración con red virtual)** mediante Azure Portal. Con Acceso privado (integración con red virtual), puede implementar el servidor flexible en la propia instancia de [Azure Virtual Network](../../virtual-network/virtual-networks-overview.md). Las redes virtuales de Azure proporcionan una comunicación de red privada y segura. Con el acceso privado, las conexiones al servidor MySQL están restringidas a la red virtual. Para obtener más información al respecto, consulte [Acceso privado (Integración con red virtual)](./concepts-networking.md#private-access-vnet-integration).

Puede implementar el servidor flexible en una red virtual y una subred durante la creación del servidor. Una vez que se haya implementado el servidor flexible, no se puede trasladar a otra red virtual, subred o a *Acceso público (direcciones IP permitidas)* .

## <a name="prerequisites"></a>Requisitos previos
Para crear un servidor flexible en una red virtual, necesitará lo siguiente:
- Una [red virtual](../../virtual-network/quick-create-portal.md#create-a-virtual-network)
    > [!Note]
    > La red virtual y la subred deben estar en la misma región y suscripción que el servidor flexible.

-  [Delegar una subred](../../virtual-network/manage-subnet-delegation.md#delegate-a-subnet-to-an-azure-service) a **Microsoft.DBforMySQL/flexibleServers**. Esta delegación significa que solo los servidores flexibles de Azure Database for MySQL pueden utilizar esa subred. No puede haber otros tipos de recursos de Azure en la subred delegada.

## <a name="create-azure-database-for-mysql-flexible-server-in-an-already-existing-virtual-network"></a>Creación del Servidor flexible de Azure Database for MySQL en una red virtual existente

1. En la esquina superior izquierda del portal, seleccione **Crear un recurso** (+).
2. Seleccione **Bases de datos** > **Azure Database for MySQL**. También puede escribir **MySQL** en el cuadro de búsqueda para encontrar el servicio.
3. Seleccione **Servidor flexible** como opción de implementación.
4. Rellene el formulario **Datos básicos**.
5. Vaya a la pestaña **Redes** para configurar cómo se quiere conectar al servidor.
6. En **Método de conectividad**, seleccione **Acceso privado (integración con red virtual)** . Vaya a **Red virtual** y seleccione la *red virtual* y la *subred* ya existentes creadas como parte de los requisitos previos anteriores.
7. Seleccione **Revisar y crear** para revisar la configuración del servidor flexible.
8. Seleccione **Crear** para realizar el aprovisionamiento del servidor. El aprovisionamiento puede tardar unos minutos.

>[!Note]
> Una vez que se haya implementado el servidor flexible en una red virtual y una subred, no se puede trasladar a Acceso público (direcciones IP permitidas).

## <a name="next-steps"></a>Pasos siguientes
- [Creación y administración de redes virtuales del Servidor flexible de Azure Database for MySQL mediante la CLI de Azure](./how-to-manage-virtual-network-cli.md).
- Obtenga más información sobre [redes en el Servidor flexible de Azure Database for MySQL](./concepts-networking.md).
- Más información sobre [Redes virtuales del Servidor flexible de Azure Database for MySQL](./concepts-networking.md#private-access-vnet-integration).
