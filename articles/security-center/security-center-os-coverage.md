---
title: Plataformas compatibles con Azure Security Center | Microsoft Docs
description: En este documento se proporciona una lista de plataformas compatibles con Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 70c076ef-3ad4-4000-a0c1-0ac0c9796ff1
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/31/2020
ms.author: memildin
ms.openlocfilehash: f02e45804234fc8f8aa2b966db7ddce5efc9cfd1
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998839"
---
# <a name="supported-platforms"></a>Plataformas compatibles 

En esta página se muestran las plataformas y los entornos compatibles con Azure Security Center.

## <a name="combinations-of-environments"></a>Combinaciones de entornos <a name="vm-server"></a>

Azure Security Center admite máquinas virtuales y servidores en diferentes tipos de entornos híbridos:

* Solo Azure
* Azure y entorno local
* Azure y otras nubes
* Azure, otras nubes y entorno local

En el caso de un entorno de Azure activado en una suscripción de Azure, Azure Security Center detectará automáticamente los recursos de IaaS que se implementan en la suscripción.

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

Security Center depende del [agente de Log Analytics](../azure-monitor/platform/agents-overview.md#log-analytics-agent). Asegúrese de que las máquinas ejecutan uno de los sistemas operativos compatibles con este agente, como se describe en las siguientes páginas:

* [Sistemas operativos Windows compatibles con el agente de Log Analytics](../azure-monitor/platform/agents-overview.md#supported-operating-systems)
* [Sistemas operativos Linux compatibles con el agente de Log Analytics](../azure-monitor/platform/agents-overview.md#supported-operating-systems)

Asegúrese también de que el agente de Log Analytics esté [configurado correctamente para enviar datos a Security Center](security-center-enable-data-collection.md#manual-agent)

> [!TIP]
> Para obtener más información acerca de las características de Security Center específicas que están disponibles en Windows y Linux, consulte [Cobertura de características para máquinas](security-center-services.md).

## <a name="managed-virtual-machine-services"></a>Servicios de máquinas virtuales administradas <a name="virtual-machine"></a>

También se crean máquinas virtuales en una suscripción de cliente como parte de algunos servicios administrados de Azure, como Azure Kubernetes (AKS), Azure Databricks, etc. Security Center detecta también estas máquinas virtuales y el agente de Log Analytics se puede instalar y configurar si hay disponible un sistema operativo compatible.

## <a name="cloud-services"></a>Cloud Services <a name="cloud-services"></a>

También se admiten máquinas virtuales que se ejecuten en un servicio en la nube. Se supervisan los roles de trabajo y web de servicios en la nube que se ejecutan en espacios de producción. Para más información sobre Cloud Services, consulte la [información general sobre Azure Cloud Services](../cloud-services/cloud-services-choose-me.md).

También se admite la protección para las VM que residan en Azure Stack. Para obtener más información acerca de la integración de Security Center con Azure Stack, consulte [Incorporación de máquinas virtuales de Azure Stack a Security Center](https://docs.microsoft.com/azure/security-center/quick-onboard-azure-stack).

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [Security Center recopila datos mediante el agente de Log Analytics](security-center-enable-data-collection.md).
- Obtenga información sobre cómo [Security Center administra y protege los datos](security-center-data-security.md).
- Aprenda a [planear y entender las consideraciones de diseño para adoptar Azure Security Center](security-center-planning-and-operations-guide.md).