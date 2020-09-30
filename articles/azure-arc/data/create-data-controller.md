---
title: Creación de un controlador de datos
description: Cree un controlador de datos de Azure Arc en un clúster típico de Kubernetes de varios nodos que ya haya implementado.
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 81c4558bde3ffdbec72a756562b972c2eed4a1ee
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932378"
---
# <a name="create-the-azure-arc-data-controller"></a>Creación del controlador de datos de Azure Arc

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="overview-of-creating-the-azure-arc-data-controller"></a>Información general sobre la creación del controlador de datos de Azure Arc

Los servicios de datos habilitados para Azure Arc se pueden crear en varios tipos diferentes de clústeres de Kubernetes y servicios de Kubernetes administrado mediante varios enfoques diferentes.

Actualmente, la lista admitida de servicios y distribuciones de Kubernetes es la siguiente:

- Azure Kubernetes Service (AKS)
- Motor de Azure Kubernetes Service (Motor de AKS) en Azure Stack
- Azure Kubernetes Service en Azure Stack HCl
- Azure RedHat OpenShift (ARO)
- OpenShift Container Platform (OCP)
- AWS Elastic Kubernetes Service (EKS)
- Google Cloud Kubernetes Engine (GKE)
- Código abierto de nivel superior de Kubernetes implementado normalmente mediante kubeadm

> [!IMPORTANT]
> * La versión mínima admitida de Kubernetes es la versión 1.14.
> * Consulte [Requisitos de conectividad](connectivity.md) para comprender qué conectividad se requiere entre su entorno y Azure.
> * Consulte [Configuración de almacenamiento](storage-configuration.md) para conocer los detalles acerca de cómo configurar el almacenamiento persistente.
> * si usa Azure Kubernetes Service, el tamaño de la máquina virtual del nodo de trabajo del clúster debe ser al menos **Standard_D8s_v3** y usar **discos Premium.** 
> * Si usa otra distribución o servicio de Kubernetes, debe asegurarse de que tiene un tamaño mínimo de nodo de 8 GB de memoria RAM y 4 núcleos y una capacidad total de 32 GB de RAM disponible entre todos los nodos de Kubernetes. Por ejemplo, podría tener 1 nodo con 32 GB de memoria RAM y 4 núcleos o bien tener 2 nodos con 16 GB de memoria RAM y 4 núcleos cada uno.

> [!NOTE]
> Si usa Red Hat OpenShift Container Platform en Azure, se recomienda usar la versión más reciente disponible.

En función de la opción que elija, algunas herramientas serán _obligatorias_, pero se recomienda [instalar todas las herramientas de cliente](install-client-tools.md) antes de comenzar la creación del controlador de datos de Azure Arc.

Independientemente de la opción que elija, durante el proceso de creación tendrá que proporcionar la siguiente información:

- **Nombre del controlador de datos**: nombre descriptivo del controlador de datos; por ejemplo, "Controlador de datos de producción" o "Controlador de datos de Seattle".
- **Nombre de usuario del controlador de datos**: nombre de usuario del usuario administrador del controlador de datos.
- **Contraseña del controlador de datos**: contraseña del usuario administrador del controlador de datos.
- **Nombre del espacio de nombres de Kubernetes**: nombre del espacio de nombres de Kubernetes en el que desea crear el controlador de datos.
- **Modo de conectividad**: [modo de conectividad](connectivity.md) del clúster. Actualmente solo se admite "indirecto".
- **Identificador de suscripción de Azure**: GUID de la suscripción de Azure en el que desea que se cree el recurso del controlador de datos en Azure.
- **Nombre del grupo de recursos de Azure**: nombre del grupo de recursos en el que desea que se cree el recurso del controlador de datos en Azure.
- **Ubicación de Azure**: ubicación de Azure en la que se almacenarán los metadatos del recurso del controlador de datos en Azure. Para obtener una lista de las regiones disponibles, consulte [Infraestructura global de Azure/Productos por región](https://azure.microsoft.com/global-infrastructure/services/?products=azure-arc).

## <a name="next-steps"></a>Pasos siguientes

Hay varias opciones para crear el controlador de datos de Azure Arc:

> **¿Solo desea realizar algunas pruebas?**  
> Empiece a trabajar rápidamente con [Azure Arc Jumpstart](https://github.com/microsoft/azure_arc#azure-arc-enabled-data-services) en Azure Kubernetes Service (AKS), AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) o en una máquina virtual de Azure.
> 
- [Creación de un controlador de datos con la CLI de datos de Azure (azdata)](create-data-controller-using-azdata.md)
- [Creación de un controlador de datos con Azure Data Studio](create-data-controller-azure-data-studio.md)
- [Creación de un controlador de datos desde Azure Portal mediante un cuaderno de Jupyter Notebook en Azure Data Studio](create-data-controller-resource-in-azure-portal.md)
- [Creación de un controlador de datos con herramientas de Kubernetes como kubectl u oc](create-data-controller-using-k8s-native-tools.md)
- [Creación de un controlador de datos con Azure Arc JumpStart para una experiencia acelerada de una implementación de prueba](https://github.com/microsoft/azure_arc#azure-arc-enabled-data-services)
