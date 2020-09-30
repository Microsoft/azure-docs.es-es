---
title: Habilitación de Automanage para máquinas virtuales a través de Azure Policy
description: Obtenga información sobre cómo habilitar Azure Automanage para máquinas virtuales a través de una instancia de Azure Policy integrada en Azure Portal.
author: ju-shim
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: how-to
ms.date: 09/04/2020
ms.author: jushiman
ms.openlocfilehash: 47f4085ff01526853fab29da2c1bc1a3e8998d23
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931957"
---
# <a name="enable-automanage-for-virtual-machines-through-azure-policy"></a>Habilitación de Automanage para máquinas virtuales a través de Azure Policy

Si desea habilitar Automanage para muchas máquinas virtuales, puede hacerlo mediante una instancia de [Azure Policy](..\governance\azure-management.md) integrada. En este artículo se le guiará a través de la búsqueda de la directiva adecuada y de su asignación para habilitar Automanage en Azure Portal.


## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/) antes de empezar.

> [!NOTE]
> Las cuentas de evaluación gratuita no tienen acceso a las máquinas virtuales que se usan en este tutorial. Actualice a una suscripción de pago por uso.

> [!IMPORTANT]
> El siguiente permiso de RBAC es necesario para habilitar Automanage: Rol **Propietario** o **Colaborador** junto con roles **Administrador de acceso de usuario**.


## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com/).


## <a name="locate-and-assign-the-policy"></a>Búsqueda y asignación de la directiva

1. Vaya a **Directiva** en Azure Portal.
1. Vaya al panel **Definiciones**.
1. Haga clic en la lista desplegable **Categorías** para ver las opciones disponibles.
1. Seleccione la opción **Enable Automanage – Azure virtual machine best practices** (Habilitar Automanage: procedimientos recomendados para máquinas virtuales de Azure)
1. Ahora se actualizará la lista para mostrar una directiva integrada con un nombre que empiece por *Habilitar Automanage...* .
1. Haga clic en el nombre de la directiva integrada *Enable Automanage - Azure virtual machine best practices* (Habilitar Automanage: procedimientos recomendados para máquinas virtuales de Azure).
1. Ahora, después de hacer clic en la directiva, puede ver la pestaña **Definición**.

    > [!NOTE]
    > La definición de Azure Policy se usa para establecer parámetros de Automanage como la cuenta o el perfil de configuración. También establece filtros que garantizan la aplicación de la directiva solo a las máquinas virtuales correctas.

1. Haga clic en el botón **Asignar** para crear una asignación.
1. En la pestaña **Conceptos básicos**, rellene **Ámbito** estableciendo la *Suscripción* y *Grupo de recursos*.

    > [!NOTE]
    > El ámbito permite definir las máquinas virtuales donde se aplica esta directiva. Puede establecer la aplicación en el nivel de suscripción o en el nivel de grupo de recursos. Si establece un grupo de recursos, todas las máquinas virtuales que estén actualmente en ese grupo de recursos o en cualquier máquina virtual futura que agreguemos al mismo tendrán habilitado automáticamente Automanage. 

1. Haga clic en la pestaña **Parámetros** y establezca la **cuenta de Automanage** y el **perfil de configuración** deseado. 
1. En la pestaña **Revisar y crear**, revise la configuración.
1. Aplique la asignación haciendo clic en **Crear**.
1. Vea las asignaciones en la pestaña **Asignaciones** situada junto a **Definición**.

> [!NOTE]
> La directiva tardará un poco en empezar a surtir efecto en las máquinas virtuales que actualmente se encuentran en la suscripción y el grupo de recursos.


## <a name="next-steps"></a>Pasos siguientes 

Aprenda otra forma de habilitar Azure Automanage para máquinas virtuales a través de Azure Portal. 

> [!div class="nextstepaction"]
> [Habilitación de Automanage para máquinas virtuales en Azure Portal](quick-create-virtual-machines-portal.md)