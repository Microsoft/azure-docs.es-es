---
title: Creación de una máquina virtual Windows con zonas mediante Azure Portal
description: Creación de una máquina virtual Windows en una zona de disponibilidad con Azure Portal
documentationcenter: virtual-machines
author: cynthn
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.workload: infrastructure
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: ''
ms.openlocfilehash: 339c093e5444839f9b106cd08980438a864c0474
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87828921"
---
# <a name="create-a-windows-virtual-machine-in-an-availability-zone-with-the-azure-portal"></a>Creación de una máquina virtual Windows en una zona de disponibilidad con Azure Portal

En este artículo se describe el uso de Azure Portal para crear una máquina virtual en una zona de disponibilidad de Azure. Una [zona de disponibilidad](../../availability-zones/az-overview.md) es una zona separada físicamente en una región de Azure. Use zonas de disponibilidad para proteger sus datos y aplicaciones de la improbable pérdida o error de todo un centro de datos.

Para usar una zona de disponibilidad, cree la máquina virtual [en una región de Azure compatible](../../availability-zones/az-region.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure 

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-virtual-machine"></a>Crear máquina virtual

1. Haga clic en **Crear un recurso** en la esquina superior izquierda de Azure Portal.

2. Seleccione **Compute** y, después, seleccione **Windows Server 2016 Datacenter**. 

3. Escriba la información de la máquina virtual. El nombre de usuario y la contraseña que especifique aquí se usarán para iniciar sesión en la máquina virtual. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](faq.md#what-are-the-password-requirements-when-creating-a-vm). Elija una ubicación como Este de EE. UU. 2 que admita las zonas de disponibilidad. Cuando haya terminado, haga clic en **Aceptar**.

    ![Especificación de la información básica de la máquina virtual en la hoja del Portal](./media/create-portal-availability-zone/create-windows-vm-portal-basic-blade.png)

4. Elija un tamaño para la máquina virtual. Seleccione un tamaño recomendado o filtre en función de las características. Confirme que el tamaño esté disponible en la zona que desea usar.

    ![Selección de un tamaño de máquina virtual](./media/create-portal-availability-zone/create-windows-vm-portal-sizes.png)  

5. En **Configuración** > **Alta disponibilidad**, seleccione una de las zonas numeradas en la lista desplegable **Zona de disponibilidad**, mantenga el resto de valores predeterminados y haga clic en **Aceptar**.

    ![Seleccione una zona de disponibilidad.](./media/create-portal-availability-zone/create-windows-vm-portal-availability-zone.png)

6. En la página de resumen, haga clic en **Crear** para iniciar la implementación de la máquina virtual.

7. La máquina virtual se anclará al panel de Azure Portal. Una vez completada la implementación, se abrirá automáticamente el resumen de la máquina virtual.

## <a name="confirm-zone-for-managed-disk-and-ip-address"></a>Confirmación de zona para disco administrado y dirección IP

Cuando la máquina virtual se implementa en una zona de disponibilidad, se crea un disco administrado para la máquina virtual en la misma zona de disponibilidad. De forma predeterminada, también se crea una dirección IP pública en dicha zona.

Puede confirmar la configuración de zona para estos recursos en el portal.  

1. Haga clic en **Grupos de recursos** y en el nombre del grupo de recursos para la máquina virtual, como *myResourceGroup*.

2. Haga clic en el nombre del recurso de disco. La página **Información general** incluye detalles sobre la ubicación y la zona de disponibilidad del recurso.

    ![Zona de disponibilidad para disco administrado](./media/create-portal-availability-zone/create-windows-vm-portal-disk.png)

3. Haga clic en el nombre del recurso de la dirección IP pública. La página **Información general** incluye detalles sobre la ubicación y la zona de disponibilidad del recurso.

    ![Zona de disponibilidad para la dirección IP](./media/create-portal-availability-zone/create-windows-vm-portal-ip.png)



## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a crear una máquina virtual en una zona de disponibilidad. Aprenda más sobre la [disponibilidad](../availability.md) de las máquinas virtuales de Azure.