---
title: Creación de un recurso de clúster de Azure Kubernetes Service
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo crear recursos de Azure Kubernetes (AKS).
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: e7f5b6f3685a94b5497784360f8f12b22fb95012
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87298811"
---
## <a name="create-an-azure-kubernetes-service-cluster-resource"></a>Creación de un recurso de clúster de Azure Kubernetes Service

1. Vaya a [Azure Kubernetes Service](https://ms.portal.azure.com/#create/microsoft.aks) y seleccione **Crear**.

1. Introduzca la siguiente información de la pestaña **Aspectos básicos**:

    |Configuración|Value|
    |--|--|
    |Suscripción|Seleccione una suscripción adecuada.|
    |Resource group|Seleccione un grupo de recursos disponible.|
    |Nombre del clúster de Kubernetes|Escriba un nombre (en minúsculas).|
    |Region|Seleccione una ubicación cercana.|
    |Versión de Kubernetes|Cualquier valor se marca como **(valor predeterminado)** .|
    |Prefijo del nombre DNS|Se crea automáticamente, pero puede invalidarlo.|
    |Tamaño del nodo|Estándar DS2 v2:<br>`2 vCPUs`, `7 GB`|
    |Número de nodos|Deje el control deslizante en el valor predeterminado.|

1. En la pestaña **Grupos de nodos**, deje **Nodos virtuales** y **VM Scale Sets** con sus valores predeterminados.
1. En la pestaña **Authentication** (Autenticación), deje las opciones **Service principal** (Entidad de servicio) y **Enable RBAC** (Habilitar RBAC) establecidas en sus valores predeterminados.
1. En la pestaña **Networking** (Redes), escriba las siguientes selecciones:

    |Configuración|Value|
    |--|--|
    |Enrutamiento de aplicación HTTP|No|
    |Configuración de red|Básico|

1. En la pestaña **Integraciones**, asegúrese de que la opción **Supervisión de contenedores** esté establecida en **Habilitada** y deje **Área de trabajo de Log Analytics** como el valor predeterminado.
1. En la pestaña **Tags** (Etiquetas), deje los pares nombre/valor en blanco por ahora.
1. Seleccione **Revisar y crear**.
1. Una vez pasada la validación, seleccione **Crear**.

> [!NOTE]
> Si se produce un error en la validación, podría deberse a un error de la entidad de servicio. Vuelva a la pestaña **Autenticación** y, luego, vaya a **Revisar y crear**, donde se debe ejecutar y superar la validación.
