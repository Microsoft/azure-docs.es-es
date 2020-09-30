---
title: Características de Azure Security Center para los recursos de PaaS de Azure compatibles.
description: En esta página se muestra la disponibilidad de características de Azure Security Center para los recursos de PaaS de Azure compatibles.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/01/2020
ms.author: memildin
ms.openlocfilehash: 77bf0f4cd60ba6e85763334a4d864975945ec2c3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90894785"
---
# <a name="feature-coverage-for-azure-paas-services"></a>Cobertura de características para los servicios de PaaS de Azure <a name="paas-services"></a>

En esta tabla se muestra la disponibilidad de características de Azure Security Center para los recursos de PaaS de Azure compatibles.

|Servicio|Recomendaciones (gratuito)|Alertas de seguridad (Azure Defender)|Evaluación de vulnerabilidades (Azure Defender)|
|:----|:----:|:----:|:----:|
|Azure App Service|✔|✔|-|
|Cuenta de Azure Automation|✔|-|-|
|Cuenta de Azure Batch|✔|-|-|
|Azure Blob Storage|✔|✔|-|
|Azure Cache for Redis|✔|-|-|
|Azure Cloud Services|✔|-|-|
|Azure Cognitive Search|✔|-|-|
|Azure Container Registry|-|-|✔|
|Azure Cosmos DB*|-|✔|-|
|Análisis con Azure Data Lake|✔|-|-|
|Azure Data Lake Storage|✔|-|-|
|Azure Database for MySQL*|✔|✔|-|
|Azure Database for PostgreSQL*|✔|✔|-|
|Espacio de nombres de Azure Event Hubs|✔|-|-|
|Aplicación Azure Functions|✔|-|-|
|Azure Key Vault|✔|✔|-|
|Azure Kubernetes Service|✔|✔|-|
|Azure Load Balancer|✔|-|-|
|Azure Logic Apps|✔|-|-|
|Azure SQL Database|✔|✔|✔|
|Instancia administrada de Azure SQL|✔|✔|✔|
|Espacio de nombres de Azure Service Bus|✔|-|-|
|Cuenta de Azure Service Fabric|✔|-|-|
|Cuentas de Azure Storage|✔|-|-|
|Azure Stream Analytics|✔|-|-|
|Suscripción de Azure|✔ **|✔|-|
|Azure Virtual Network</br> (incl. subredes, NIC y grupos de seguridad de red)|✔|-|-|

\* Estas características se admiten actualmente en la versión preliminar.

\*\* Las recomendaciones de Azure Active Directory (Azure AD) solo están disponibles para las suscripciones con Azure Defender habilitado.