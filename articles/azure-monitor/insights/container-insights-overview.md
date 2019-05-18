---
title: Introducción a Azure Monitor para contenedores | Microsoft Docs
description: En este artículo se describe Azure Monitor para contenedores, que supervisa la solución de AKS Container Insights y el valor que entrega mediante la supervisión del estado de los clústeres de AKS y Container Instances en Azure.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/25/2019
ms.author: magoedte
ms.openlocfilehash: d6321564672097fbf901d1d33afac9f606fcb63a
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65521832"
---
# <a name="azure-monitor-for-containers-overview"></a>Introducción a Azure Monitor para contenedores

Azure Monitor para contenedores es una característica diseñada para supervisar el rendimiento de las cargas de trabajo de contenedor implementadas en Azure Container Instances clústeres de Kubernetes administrados hospedados en Azure Kubernetes Service (AKS). La supervisión de los contenedores es fundamental, sobre todo cuando se ejecuta un clúster de producción, a escala, con varias aplicaciones.

Azure Monitor para contenedores le brinda la posibilidad de visibilizar el rendimiento mediante la recopilación de métricas del procesador y de la memoria de los controladores, nodos y contenedores disponibles en Kubernetes mediante la API de métricas. También se recopilan registros del contenedor.  Después de habilitar la supervisión de clústeres de Kubernetes, métricas y registros se recopilan automáticamente por usted a través de una versión del agente de Log Analytics para Linux en contenedor. Las métricas se escriben en el almacén de métricas y datos de registro se escriben en el almacén de registros asociado con su [Log Analytics](../log-query/log-query-overview.md) área de trabajo. 

![Azure Monitor para la arquitectura de contenedores](./media/container-insights-overview/azmon-containers-architecture-01.png)
 
## <a name="what-does-azure-monitor-for-containers-provide"></a>¿Cómo se proporciona Azure Monitor para contenedores?

Azure Monitor para contenedores ofrece una experiencia de supervisión integral con diferentes características de Azure Monitor que permite comprender el rendimiento y el estado del clúster de Kubernetes y las cargas de trabajo de contenedor. Con Azure Monitor para los contenedores, puede:

* Identificar los contenedores de AKS que se ejecutan en el nodo y su utilización media tanto del procesador como de la memoria. Este conocimiento puede ayudarle a identificar cuellos de botella en los recursos.
* Identificar el uso de procesador y memoria de grupos de contenedores y sus contenedores hospedados en Azure Container Instances.  
* Identificar dónde se encuentra el contenedor en un controlador o un pod. Este conocimiento puede ayudarle a ver el rendimiento general del controlador o del pod.
* Revisar el uso de recursos de las cargas de trabajo que se ejecutan en el host que no estén relacionadas con los procesos estándar que admite el pod.
* Conocer el comportamiento del clúster tanto con cargas medias como con las más pesadas. Este conocimiento puede ayudarle a identificar los requisitos de capacidad y determinar la carga máxima que el clúster puede admitir. 
* Configurar alertas para avisarle proactivamente o lo registra cuando el uso de CPU y memoria en los nodos o contenedores superan los umbrales.  

## <a name="how-do-i-access-this-feature"></a>¿Cómo se obtiene acceso a esta característica?
Puede acceder a Azure Monitor para contenedores de dos maneras: desde Azure Monitor o directamente desde el clúster de AKS seleccionado. Azure monitor tiene una perspectiva global de todos los contenedores implementan, que se supervisan y cuáles no, lo que le permite buscar y filtrar a través de las suscripciones y los grupos de recursos y, a continuación, profundizar en Azure Monitor para los contenedores de la contenedor seleccionado.  En caso contrario, puede tener acceso a la característica directamente desde un contenedor AKS seleccionado desde la página AKS.  

![Introducción a los métodos de acceso a Azure Monitor para contenedores](./media/container-insights-overview/azmon-containers-experience.png)

Si está interesado en la supervisión y la administración de hosts de contenedor de Docker y Windows y desea más información sobre la configuración, la auditoría y la utilización de recursos, consulte la [solución de supervisión de contenedores](../../azure-monitor/insights/containers.md).

## <a name="next-steps"></a>Pasos siguientes
Para comenzar a supervisar el clúster de AKS, revise [cómo habilitar el Monitor de Azure para contenedores](container-insights-onboard.md) para conocer los requisitos y los métodos disponibles para habilitar la supervisión.  
