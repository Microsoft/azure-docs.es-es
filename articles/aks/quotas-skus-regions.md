---
title: Límites de recursos, SKU, regiones
titleSuffix: Azure Kubernetes Service
description: Obtenga información sobre las cuotas predeterminadas, los tamaños de SKU de VM del nodo restringido y la disponibilidad de la región de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 04/09/2019
ms.openlocfilehash: 9592b03f5bd9e9436db5665e040b4afd3b68b764
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87562742"
---
# <a name="quotas-virtual-machine-size-restrictions-and-region-availability-in-azure-kubernetes-service-aks"></a>Cuotas, restricciones de tamaño de máquinas virtuales y disponibilidad de regiones en Azure Kubernetes Service (AKS)

Todos los servicios de Azure establecen límites y cuotas predeterminados para recursos y características. Algunos SKU de máquinas virtuales (VM) también están restringidos para su uso.

En este artículo se detallan los límites de recursos predeterminados para los recursos de Azure Kubernetes Service (AKS) y la disponibilidad de AKS en las regiones de Azure.

## <a name="service-quotas-and-limits"></a>Límites y cuotas del servicio

[!INCLUDE [container-service-limits](../../includes/container-service-limits.md)]

## <a name="provisioned-infrastructure"></a>Infraestructura aprovisionada

A la infraestructura aprovisionada se le aplican las demás limitaciones de red, proceso y almacenamiento. Consulte [límites de suscripción y servicios de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md) para conocer los límites pertinentes.

> [!IMPORTANT]
> Cuando actualiza un clúster de AKS, los recursos adicionales se consumen temporalmente. Estos recursos incluyen las direcciones IP disponibles en una subred de red virtual o una cuota de vCPU de máquina virtual. Si usa los contenedores de Windows Server, el único enfoque aprobado para aplicar las últimas actualizaciones a los nodos es realizar una operación de actualización. Un proceso de actualización de clúster erróneo puede indicar que no tiene el espacio de direcciones IP disponible o la cuota de vCPU necesaria para administrar estos recursos temporales. Para obtener más información sobre el proceso de actualización del nodo de Windows Server, consulte [Actualización de un grupo de nodos en AKS][nodepool-upgrade].

## <a name="restricted-vm-sizes"></a>Tamaños de VM restringidos

Cada nodo en un clúster de AKS contiene una cantidad fija de recursos de proceso, como la vCPU y la memoria. Si un nodo de AKS no tiene suficientes recursos de proceso, es posible que los pods no se ejecuten correctamente. Para asegurarse de que los pods de *kube-system*  necesarios y las aplicaciones puedan programarse de manera confiable, **no use las siguientes SKU de VM en AKS**:

- Standard_A0
- Standard_A1
- Standard_A1_v2
- Standard_B1s
- Standard_B1ms
- Standard_F1
- Standard_F1s

Para obtener más información sobre los tipos de VM y sus recursos de proceso, consulte [Tamaños de las máquinas virtuales en Azure][vm-skus].

## <a name="region-availability"></a>Disponibilidad en regiones

Para obtener la lista más reciente con los lugares donde puede implementar y ejecutar clústeres, consulte [Disponibilidad de regiones de AKS][region-availability].

## <a name="next-steps"></a>Pasos siguientes

Pueden aumentarse ciertas cuotas y límites predeterminados. Si su recurso admite un aumento, solicítelo mediante una [solicitud de soporte técnico de Azure][azure-support] (en **Tipo de problema**, seleccione **Cuota**).

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
[region-availability]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service

<!-- LINKS - Internal -->
[vm-skus]: ../virtual-machines/sizes.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
