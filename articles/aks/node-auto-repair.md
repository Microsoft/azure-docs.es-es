---
title: Reparación automática de nodos de Azure Kubernetes Service (AKS)
description: Obtenga información sobre la funcionalidad de reparación automática de nodos y cómo AKS corrige los nodos de trabajo interrumpidos.
services: container-service
ms.topic: conceptual
ms.date: 06/02/2020
ms.openlocfilehash: 7fcb7b380f3694aaf34328019c3e09f5157c9e64
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87542049"
---
# <a name="azure-kubernetes-service-aks-node-auto-repair"></a>Reparación automática de nodos de Azure Kubernetes Service (AKS)

AKS comprueba de forma continua el estado de mantenimiento de los nodos de trabajo y, si son incorrectos, los repara automáticamente. Este documento informa a los operadores sobre cómo se comporta la funcionalidad de reparación automática del nodo. Además de las reparaciones de AKS, la plataforma de VM de Azure [realiza el mantenimiento en máquinas virtuales][vm-updates] que también experimentan problemas. AKS y las VM de Azure trabajan de forma conjunta para minimizar las interrupciones del servicio de los clústeres.

## <a name="limitations"></a>Limitaciones

* En la actualidad, no se admiten los grupos de nodos de Windows.

## <a name="how-aks-checks-for-unhealthy-nodes"></a>Comprobaciones de nodos en mal estado en AKS

AKS usa reglas para determinar si un nodo es incorrecto y necesita reparación. AKS usa las reglas siguientes para determinar si es necesaria la reparación automática.

* El nodo notifica el estado de **NotReady** de las comprobaciones consecutivas en un período de 10 minutos.
* El nodo no notifica un estado en 10 minutos.

Puede comprobar manualmente el estado de mantenimiento de los nodos con kubectl.

```
kubectl get nodes
```

## <a name="how-automatic-repair-works"></a>Funcionamiento de la reparación automática

> [!Note]
> AKS inicia las operaciones de reparación con la cuenta de usuario **aks-remediator**.

Si se determina que un nodo es incorrecto de acuerdo con las reglas anteriores y se mantiene en este estado durante 10 minutos consecutivos, AKS reinicia el nodo. Si los nodos siguen siendo incorrectos después de la operación de reparación inicial, los ingenieros de AKS investigan las correcciones adicionales.
  
Si hay varios nodos incorrectos durante una comprobación de estado, cada nodo se repara individualmente antes de que comience otra reparación.

## <a name="next-steps"></a>Pasos siguientes

Use [Availability Zones][availability-zones] para aumentar la alta disponibilidad con las cargas de trabajo del clúster de AKS.

<!-- LINKS - External -->

<!-- LINKS - Internal -->
[availability-zones]: ./availability-zones.md
[vm-updates]: ../virtual-machines/maintenance-and-updates.md
