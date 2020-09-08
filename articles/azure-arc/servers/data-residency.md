---
title: Residencia de datos
description: Residencia de datos e información sobre los servidores habilitados para Azure Arc (versión preliminar).
ms.topic: reference
ms.date: 08/25/2020
ms.custom: references_regions
ms.openlocfilehash: 8f207f5889c1764eebcc6081960ff70c0d5bca3a
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89048863"
---
# <a name="azure-arc-enabled-servers-preview-data-residency"></a>Servidores habilitados para Azure Arc (versión preliminar): Residencia de datos

En este artículo se explica el concepto de residencia de datos y cómo se aplica a los servidores habilitados para Azure Arc (versión preliminar).

Los servidores habilitados para Azure Arc (versión preliminar) están **[disponibles en versión preliminar](https://azure.microsoft.com/global-infrastructure/services/?products=azure-arc)** en **Estados Unidos, Europa o Asia Pacífico**.

## <a name="data-residency"></a>Residencia de datos

Los servidores habilitados para Azure Arc (versión preliminar) almacenan los valores de configuración de la [extensión de máquina virtual de Azure](manage-vm-extensions.md) (es decir, los valores de propiedades). Es necesario especificar la extensión antes de intentar habilitarla en la máquina conectada. Por ejemplo, cuando se habilita la extensión de máquina virtual de Log Analytics, se solicita el **id. de área de trabajo** y la **clave principal** de Log Analytics.

También se recopila información de metadatos sobre la máquina conectada. Concretamente:

* Nombre y versión del sistema operativo
* Nombre del equipo
* Nombre de dominio completo (FQDN) del equipo
* Versión del agente Connected Machine

Los servidores con ARC habilitado (versión preliminar) permiten especificar la región en la que se almacenarán los datos. Microsoft puede replicar en otras regiones para obtener resistencia de datos, pero no replica ni mueve datos fuera de la geografía. Estos datos se almacenan en la región donde se ha configurado el recurso de máquina de Azure Arc. Por ejemplo, si la máquina se registra en Arc en la región Este de EE. UU., estos datos se almacenan en la región de EE. UU.

Para obtener más información sobre la resistencia regional y la compatibilidad con el cumplimiento, consulte [Geografía de Azure](https://azure.microsoft.com/global-infrastructure/geographies/).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre el diseño de la [resistencia de Azure](/azure/architecture/reliability/architect).