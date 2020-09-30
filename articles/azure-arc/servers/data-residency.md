---
title: Residencia de datos
description: Residencia de datos e información sobre los servidores habilitados para Azure Arc.
ms.topic: reference
ms.date: 09/02/2020
ms.custom: references_regions
ms.openlocfilehash: 8b4b8171bd7133e52928a5227c488bd6234ce686
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90908126"
---
# <a name="azure-arc-enabled-servers-data-residency"></a>Servidores habilitados para Azure Arc: Residencia de datos

En este artículo se explica el concepto de residencia de datos y cómo se aplica a los servidores habilitados para Azure Arc.

Los servidores habilitados para Azure Arc están **[disponibles](https://azure.microsoft.com/global-infrastructure/services/?products=azure-arc)** en **Estados Unidos, Europa o Asia Pacífico**.

## <a name="data-residency"></a>Residencia de datos

Los servidores habilitados para Azure Arc almacenan los valores de configuración de la [extensión de máquina virtual de Azure](manage-vm-extensions.md) (es decir, los valores de propiedad). Es necesario especificar la extensión antes de intentar habilitarla en la máquina conectada. Por ejemplo, cuando se habilita la extensión de máquina virtual de Log Analytics, se solicita el **id. de área de trabajo** y la **clave principal** de Log Analytics.

También se recopila información de metadatos sobre la máquina conectada. Concretamente:

* Nombre y versión del sistema operativo
* Nombre del equipo
* Nombre de dominio completo (FQDN) del equipo
* Versión del agente Connected Machine

Los servidores habilitados para Arc permiten especificar la región en la que se almacenarán los datos. Microsoft puede replicar en otras regiones para obtener resistencia de datos, pero no replica ni mueve datos fuera de la geografía. Estos datos se almacenan en la región donde se ha configurado el recurso de máquina de Azure Arc. Por ejemplo, si la máquina se registra en Arc en la región Este de EE. UU., estos datos se almacenan en la región de EE. UU.

Para obtener más información sobre la resistencia regional y la compatibilidad con el cumplimiento, consulte [Geografía de Azure](https://azure.microsoft.com/global-infrastructure/geographies/).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre el diseño de la [resistencia de Azure](/azure/architecture/reliability/architect).