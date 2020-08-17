---
title: Configuración de la preferencia de enrutamiento para una dirección IP pública mediante la CLI de Azure
titlesuffix: Azure Virtual Network
description: Obtenga información sobre cómo crear una dirección IP pública con una preferencia de enrutamiento de tráfico de Internet
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/18/2020
ms.author: mnayak
ms.custom: devx-track-azurecli
ms.openlocfilehash: 64284b198fc76c219ffe0dfbc57461b587b23130
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87504609"
---
# <a name="configure-routing-preference-for-a-public-ip-address-using-azure-cli"></a>Configuración de la preferencia de enrutamiento para una dirección IP pública mediante la CLI de Azure

En este artículo se muestra cómo configurar las preferencias de enrutamiento a través de la red de ISP (opción de **Internet**) para una dirección IP pública mediante la CLI de Azure. Después de crear la dirección IP pública, puede asociarla a los siguientes recursos de Azure para el tráfico entrante y saliente de Internet:

* Máquina virtual
* Conjunto de escalado de máquina virtual
* Azure Kubernetes Service (AKS)
* Equilibrador de carga accesible desde Internet
* Application Gateway
* Azure Firewall

De forma predeterminada, la preferencia de enrutamiento de la dirección IP pública se establece en la red global de Microsoft para todos los servicios de Azure y puede asociarse a cualquier servicio de Azure.

> [!IMPORTANT]
> La preferencia de enrutamiento se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) ahora.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]
Si, en su lugar, decide instalar y usar la CLI de Azure en un entorno local, para esta guía de inicio rápido se necesita la versión 2.0.49 de la CLI de Azure o una versión posterior. Ejecute `az --version` para buscar la versión instalada. Consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli) para obtener información sobre la instalación o actualización.

## <a name="register-the-feature-for-your-subscription"></a>Registro de la característica de la suscripción
La característica de preferencias de enrutamiento se encuentra actualmente en versión preliminar. Registre de la característica de la suscripción tal como se muestra a continuación:
```azurecli
az feature register --namespace Microsoft.Network --name AllowRoutingPreferenceFeature
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group#az-group-create). En el siguiente ejemplo se crea un grupo de recursos en la región **Este de EE. UU.** de Azure:

```azurecli
  az group create --name myResourceGroup --location eastus
```
## <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Cree una dirección IP pública con preferencia de enrutamiento y de tipo "Internet" mediante el comando [az network public-ip create](/cli/azure/network/public-ip?view=azure-cli-latest#az-network-public-ip-create), con el formato que se muestra a continuación.

El siguiente comando crea una nueva dirección IP pública con preferencia de enrutamiento de **Internet** en la región de Azure del **Este de EE. UU.** .

```azurecli
az network public-ip create \
--name MyRoutingPrefIP \
--resource-group MyResourceGroup \
--location eastus \
--ip-tags 'RoutingPreference=Internet' \
--sku STANDARD \
--allocation-method static \
--version IPv4
```

> [!NOTE]
>  Actualmente, la preferencia de enrutamiento solo admite direcciones IP públicas IPV4.

Puede asociar la dirección IP pública creada anteriormente con una máquina virtual de [Windows](../virtual-machines/windows/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [Linux](../virtual-machines/linux/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Use la sección CLI en la siguiente página del tutorial: [asociación de una dirección IP pública a una máquina virtual](associate-public-ip-address-vm.md#azure-cli) para asociar la dirección IP pública a la VM. Asimismo, puede asociar una dirección IP pública que haya creado con anterioridad con una instancia de [Azure Load Balancer](../load-balancer/load-balancer-overview.md) asignándola a la configuración de **front-end** del equilibrador de carga. La dirección IP pública actúa como dirección IP virtual (VIP) de carga equilibrada.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre la [preferencia de enrutamiento en direcciones IP públicas](routing-preference-overview.md). 
- [Configure la preferencia de enrutamiento de una VM mediante la CLI de Azure](configure-routing-preference-virtual-machine-cli.md).

