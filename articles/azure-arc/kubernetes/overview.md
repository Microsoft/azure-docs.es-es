---
title: Información general de Kubernetes habilitado para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 05/19/2020
ms.topic: overview
author: mlearned
ms.author: mlearned
description: En este artículo se proporciona información general de Kubernetes habilitado para Azure Arc.
keywords: Kubernetes, Arc, Azure, containers
ms.custom: references_regions
ms.openlocfilehash: fb8a7b7c2e1e5b3de7d1ccdb4054e44825231458
ms.sourcegitcommit: 80b9c8ef63cc75b226db5513ad81368b8ab28a28
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90604808"
---
# <a name="what-is-azure-arc-enabled-kubernetes-preview"></a>¿Qué es la versión preliminar de Kubernetes habilitado para Azure Arc?

Puede asociar y configurar clústeres de Kubernetes dentro o fuera de Azure mediante la versión preliminar de Kubernetes habilitado para Azure Arc. Cuando un clúster de Kubernetes esté asociado a Azure Arc, aparecerá en Azure Portal. Tendrá un identificador de Azure Resource Manager y una identidad administrada. Los clústeres se asocian a suscripciones estándar de Azure, viven en un grupo de recursos y pueden recibir etiquetas como cualquier otro recurso de Azure. 

Para conectar un clúster de Kubernetes a Azure, es preciso que el administrador de clústeres implemente agentes. Estos agentes se ejecutan en un espacio de nombres de Kubernetes denominado `azure-arc` y son implementaciones estándar de Kubernetes. Los agentes son responsables de la conectividad con Azure, de la recopilación de registros y métricas de Azure Arc y de la supervisión de solicitudes de configuración. 

Kubernetes habilitado para Azure Arc admite SSL estándar del sector para proteger los datos en tránsito. Además, los datos se almacenan cifrados en reposo en una base de datos de Azure Cosmos DB para garantizar la confidencialidad de los datos.
 
> [!NOTE]
> Kubernetes habilitado para Azure Arc está en versión preliminar. No es aconsejable para cargas de trabajo de producción.

## <a name="supported-kubernetes-distributions"></a>Distribuciones de Kubernetes admitidas

Kubernetes habilitado para Azure Arc funciona con cualquier clúster de Kubernetes certificado de Cloud Native Computing Foundation (CNCF), como AKS-Engine en Azure, AKS-Engine en Azure Stack Hub, GKE, EKS y clúster de VMware vSphere, con Azure Arc.

El equipo de Arc ha probado las características de Kubernetes habilitado para Azure Arc en las siguientes distribuciones:
* RedHat OpenShift 4.3
* Rancher RKE 1.0.8
* Canonical Charmed Kubernetes 1.18
* Motor de AKS
* Motor de AKS en Azure Stack Hub
* Proveedor de Cluster API de Azure

## <a name="supported-scenarios"></a>Escenarios admitidos 

Kubernetes habilitado para Azure Arc admite estos escenarios: 

* Conexión de Kubernetes cuando se ejecuta fuera de Azure para la realización de tareas de inventario, agrupación y etiquetado.

* Implementación de aplicaciones y aplicación de la configuración mediante la administración de configuraciones basada en GitOps. 

* Uso de Azure Monitor en los contenedores para ver y supervisar los clústeres. 

* Aplicación de directivas mediante Azure Policy para Kubernetes. 

[!INCLUDE [azure-lighthouse-supported-service](../../../includes/azure-lighthouse-supported-service.md)]

## <a name="supported-regions"></a>Regiones admitidas 

Kubernetes habilitado para Azure Arc se admite actualmente en estas regiones: 

* Este de EE. UU. 
* Oeste de Europa


## <a name="next-steps"></a>Pasos siguientes

* [Conexión de un clúster](./connect-cluster.md)
