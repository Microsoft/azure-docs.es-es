---
title: Uso de máquinas virtuales y grupos de seguridad de red en Azure Bastion
description: Puede usar grupo de seguridad de red con Azure Bastion. Obtenga información sobre las subredes necesarias para esta configuración.
services: bastion
author: charwen
ms.service: bastion
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: charwen
ms.openlocfilehash: 563c12f91e9553f802d4cf26519da0550880dfcd
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88270536"
---
# <a name="working-with-nsg-access-and-azure-bastion"></a>Trabajo con acceso a grupos de seguridad de red y Azure Bastion

Al trabajar con Azure Bastion, puede usar grupos de seguridad de red (NSG). Para más información, consulte [Grupos de seguridad](../virtual-network/security-overview.md).

:::image type="content" source="./media/bastion-nsg/figure-1.png" alt-text="NSG":::

En este diagrama:

* El host de Bastion se implementa en la red virtual.
* El usuario se conecta a Azure Portal con cualquier explorador HTML5.
* El usuario va a la máquina virtual de Azure para RDP/SSH.
* Integración de conexión: sesión de RDP/SSH con un solo clic dentro del explorador
* No se requiere ninguna dirección IP pública en la máquina virtual de Azure.

## <a name="network-security-groups"></a><a name="nsg"></a>Grupos de seguridad de red

En esta sección se muestra el tráfico de red entre el usuario y Azure Bastion, y para dirigirse a las máquinas virtuales de la red virtual:

### <a name="azurebastionsubnet"></a><a name="apply"></a>AzureBastionSubnet

Azure Bastion se implementa específicamente en ***AzureBastionSubnet***.

* **Tráfico de entrada:**

   * **Tráfico de entrada procedente de Internet pública:** Azure Bastion creará una dirección IP pública que necesita el puerto 443 habilitado en la dirección IP pública para el tráfico de entrada. El puerto 3389/22 no tiene que abrirse en la subred AzureBastionSubnet.
   * **Plano de control del tráfico de entrada procedente de Azure Bastion:** Para la conectividad del plano de control, habilite el puerto 443 entrante desde la etiqueta de servicio de **GatewayManager**. De este modo, se permite que el plano de control, es decir, el administrador de puerta de enlace, pueda comunicarse con Azure Bastion.


   :::image type="content" source="./media/bastion-nsg/inbound.png" alt-text="Entrante":::

* **Tráfico de salida:**

   * **Tráfico de salida a máquinas virtuales de destino:** Azure Bastion se comunicará con las máquinas virtuales de destino a través de la dirección IP privada. Los grupos de seguridad de red tienen que permitir el tráfico de salida a otras subredes de máquinas virtuales de destino para el puerto 3389 y 22.
   * **Salida del tráfico a otros puntos de conexión públicos de Azure:** Azure Bastion debe ser capaz de conectarse a varios puntos de conexión públicos dentro de Azure (por ejemplo, para almacenar registros de diagnóstico y los registros de medición). Por esta razón, Azure Bastion necesita una salida hacia 443 para la etiqueta de servicio **AzureCloud**.


   :::image type="content" source="./media/bastion-nsg/outbound.png" alt-text="Outbound":::

### <a name="target-vm-subnet"></a>Subred de máquina virtual de destino
Se trata de una subred que contiene la máquina virtual de destino a la que quiere conectarse mediante RDP/SSH.

   * **Tráfico de entrada procedente de Azure Bastion:** Azure Bastion se comunicará con la máquina virtual de destino a través de la dirección IP privada. Los puertos RDP/SSH (puertos 3389/22, respectivamente) tienen que abrirse en la máquina virtual de destino a través de la dirección IP privada. Como procedimiento recomendado, puede agregar el intervalo de direcciones IP de la subred de Azure Bastion en esta regla para permitir que solo Bastion pueda abrir estos puertos en las máquinas virtuales de destino de la subred de la máquina virtual de destino.


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Bastion, consulte las [preguntas frecuentes](bastion-faq.md).
