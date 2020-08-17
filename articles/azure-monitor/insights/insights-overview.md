---
title: Introducción a Insights en Azure Monitor | Microsoft Docs
description: Insights proporcionan una experiencia de supervisión personalizada en Azure Monitor para determinadas aplicaciones y servicios. En este artículo se proporciona una breve descripción de cada una de las informaciones que están actualmente disponibles.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/22/2019
ms.openlocfilehash: e3eefb97c85b3865abbf116b9912dc45f6db8ce8
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326875"
---
# <a name="overview-of-insights-in-azure-monitor"></a>Introducción a Insights en Azure Monitor
Insights proporciona una experiencia de supervisión personalizada para determinadas aplicaciones y servicios. Almacena datos en la [plataforma de datos de Azure Monitor](../platform/data-platform.md) y aprovecha otras características de Azure Monitor para el análisis y la creación de alertas, pero puede recopilar datos adicionales y proporcionar una experiencia de usuario única en Azure Portal. Obtener acceso a información a partir de la sección **Insights** del menú de Azure Monitor en Azure Portal.


Las secciones siguientes proporcionan una breve descripción de la información actualmente disponible en Azure Monitor. Consulte la documentación detallada para obtener información sobre cada una.

## <a name="application-insights"></a>Application Insights
Application Insights es un servicio de Application Performance Management (APM) extensible para desarrolladores web en varias plataformas. Úselo para supervisar la aplicación web en directo. Funciona con diversas aplicaciones y en una amplia variedad de plataformas, como .NET, Node.js o Java EE, hospedadas en el entorno local, de forma híbrida o en cualquier nube pública. También se integra con el proceso de DevOps y tiene puntos de conexión a numerosas herramientas de desarrollo.

Vea [¿Qué es Application Insights?](../app/app-insights-overview.md).

![Application Insights](media/insights-overview/app-insights.png)

## <a name="azure-monitor-for-containers"></a>Azure Monitor para contenedores
Azure Monitor para contenedores supervisa el rendimiento de las cargas de trabajo de contenedor implementadas en Azure Container Instances o en clústeres de Kubernetes administrados hospedados en Azure Kubernetes Service (AKS). La supervisión de los contenedores es fundamental, sobre todo cuando se ejecuta un clúster de producción, a escala, con varias aplicaciones.

Vea [Introducción a Azure Monitor para contenedores](./container-insights-overview.md).

![Azure Monitor para contenedores](media/insights-overview/container-insights.png)

## <a name="azure-monitor-for-resource-groups-preview"></a>Azure Monitor para grupos de recursos (versión preliminar)
Azure Monitor para grupo de recursos ayuda a clasificar y diagnosticar cualquier problema que encuentren sus recursos individuales, a la vez que se ofrece un contexto en cuanto al estado y el rendimiento del grupo de recursos como un todo.

Vea [Supervisar los grupos de recursos con Azure Monitor (versión preliminar)](./resource-group-insights.md).

![Azure Monitor para grupos de recursos](media/insights-overview/resource-group-insights.png)

## <a name="azure-monitor-for-vms-preview"></a>Azure Monitor para VM (versión preliminar)
Azure Monitor para VM supervisa las máquinas virtuales (VM) y los conjuntos de escalado de máquinas virtuales de Azure. El servicio analiza el rendimiento y el estado de las VM Windows y Linux, y supervisa sus procesos y dependencias en otros recursos y procesos externos.

Vea [¿Qué es Azure Monitor para máquinas virtuales?](vminsights-overview.md)

![Azure Monitor para máquinas virtuales](media/insights-overview/vm-insights.png)

## <a name="azure-monitor-for-networks-preview"></a>Azure Monitor for Networks (versión preliminar)
[Azure Monitor for Networks](network-insights-overview.md) proporciona una vista completa del estado y las métricas de todos los recursos de red. La capacidad de búsqueda avanzada ayuda a identificar dependencias de recursos, lo que habilita escenarios como la identificación de recursos que hospedan el sitio web con una simple búsqueda del nombre del sitio web.

![Azure Monitor para Redes](media/insights-overview/network-insights.png)

## <a name="next-steps"></a>Pasos siguientes
* Más información sobre la [plataforma de datos de Azure Monitor](../platform/data-platform.md) y la información que proporciona.
* Obtenga información sobre los diferentes [orígenes de datos usados por Azure Monitor](../platform/data-sources.md) y los diferentes tipos de datos recopilados por cada una de las opciones de información.

