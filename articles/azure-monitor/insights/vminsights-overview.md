---
title: ¿Qué es Azure Monitor para VM (versión preliminar)? | Microsoft Docs
description: Azure Monitor para VM es una característica de Azure Monitor que combina la supervisión de rendimiento y de mantenimiento del sistema operativo de la VM de Azure, así como la detección automática de las dependencias y los componentes de aplicación con otros recursos, y asigna la comunicación entre ellos. En este artículo se proporciona información general.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: ''
ms.service: azure-monitor
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/13/2019
ms.author: magoedte
ms.openlocfilehash: 7d86b3fe9aeddd603d0c40b1c760cabdee42e396
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522107"
---
# <a name="what-is-azure-monitor-for-vms-preview"></a>¿Qué es Azure Monitor para VM (versión preliminar)?

Azure Monitor para VM supervisa las máquinas virtuales (VM) y los conjuntos de escalado de máquinas virtuales de Azure. El servicio analiza el rendimiento y el estado de las VM Windows y Linux, y supervisa sus procesos y dependencias en otros recursos y procesos externos. 

Como solución, Azure Monitor para VM permite supervisar el rendimiento y las dependencias de las aplicaciones en VM hospedadas en el entorno local o en otro proveedor en la nube. Tres características clave ofrecen información detallada:

* **Componentes lógicos de VM de Azure que ejecutan Windows y Linux**: Se miden con respecto a criterios de mantenimiento configurados previamente, y le avisen cuando se cumple la condición evaluada.  

* **Gráficos de rendimiento de las tendencias predefinidos**: Muestra las principales métricas de rendimiento del sistema operativo de la VM.

* **Mapa de dependencias**: Muestra los componentes interconectados con la VM de varios grupos de recursos y suscripciones.  

Las características se organizan en tres perspectivas:

* Health
* Rendimiento
* Map

>[!NOTE]
>Actualmente, la característica de mantenimiento se ofrece solo para máquinas virtuales de Azure. Rendimiento y características de mapa admiten Azure Virtual Machines, conjuntos de escalado de máquina virtual de Azure y máquinas virtuales que se hospedan en su entorno u otro proveedor de nube.

La integración con los registros de Azure Monitor ofrece agregación y filtrado eficaces, y puede analizar la tendencias de los datos a lo largo del tiempo. Dicha supervisión integral de las cargas de trabajo no se puede lograr únicamente con Azure Monitor o Service Map.  

Puede ver estos datos directamente en una sola VM desde la máquina virtual, o puede usar a Azure Monitor para ofrecer una vista agregada de las VM. Esta vista se basa en la perspectiva de la característica:

* **Estado**: VM relacionadas con un grupo de recursos.
* **Asignación** y **Rendimiento**: Máquinas virtuales configuradas para informar a un área de trabajo de Log Analytics específica.

![Perspectiva de conclusiones de la máquina virtual desde Azure Portal](./media/vminsights-overview/vminsights-azmon-directvm-01.png)

Azure Monitor para VM puede ofrecer un rendimiento y disponibilidad predecibles de aplicaciones vitales. Identifica los eventos críticos del sistema operativo, los cuellos de botella de rendimiento y problemas de red. Azure Monitor para VM también puede ayudarle a entender si un problema está relacionado con otras dependencias.  

## <a name="data-usage"></a>Uso de datos 

Cuando implementa Azure Monitor para VM, los datos que recopilen sus VM se ingieren y almacenan en Azure Monitor. Las métricas de los criterios de estado se almacenan en Azure Monitor en una base de datos de series temporales, rendimiento y la dependencia de los datos recopilados se almacenan en un área de trabajo de Log Analytics. Según los precios que se publican en la [página de precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/), Azure Monitor para VM se factura según:

* Los datos que se ingieren y almacenan.
* El número de series temporales de métricas de criterios de mantenimiento que se supervisan.
* Las reglas de alerta que se crean.
* Las notificaciones que se envían. 

El tamaño del registro varía según las longitudes de cadena de los contadores de rendimiento, y puede aumentar con el número de discos lógicos y adaptadores de red asignados a la máquina virtual. Si ya tiene un área de trabajo y está recopilando estos contadores, no se aplicará ningún cargo duplicado. Si ya usa Service Map, el único cambio que verá son los datos de conexión adicionales que se envían a Azure Monitor.

## <a name="next-steps"></a>Pasos siguientes
Para conocer los requisitos y los métodos para habilitar la supervisión de máquinas virtuales, consulte [Implementación de Azure Monitor para VM](vminsights-enable-overview.md).
