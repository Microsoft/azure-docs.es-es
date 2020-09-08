---
title: Comparación entre Azure Migrate y Site Recovery para la migración a Azure
description: En este artículo se resumen las ventajas del uso de Azure Migrate para la migración, en lugar de Site Recovery.
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 08/06/2020
ms.author: raynew
ms.openlocfilehash: 358efaa1493aa08fb76c9bb83e0e4289950e0969
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87844327"
---
# <a name="migrating-to-azure"></a>Migración a Azure

Para la migración de máquinas virtuales y servidores a Azure, se recomienda el uso del servicio Azure Migrate, en lugar del servicio Azure Site Recovery. [Más información](../migrate/migrate-services-overview.md) acerca de Azure Migrate.


## <a name="why-use-azure-migrate"></a>¿Por qué usar Azure Migrate?

El uso de Azure Migrate para la migración proporciona una serie de ventajas:
 
 
- Azure Migrate ofrece un centro de conectividad centralizado para la detección, evaluación y migración a Azure.
- El uso de Azure Migrate proporciona interoperabilidad y extensibilidad futura con herramientas de Azure Migrate, otros servicios de Azure y herramientas de terceros.
- La herramienta Azure Migrate:Server Migration está diseñada específicamente para la migración de servidores a Azure. Está optimizada para la migración. No es necesario obtener información sobre los conceptos y escenarios que no son directamente pertinentes para la migración. 
- No hay cargos por uso de herramientas para la migración durante 180 días a partir del momento en que se inicia la replicación de una máquina virtual. Esto le proporciona tiempo para completar la migración. Solo paga por los recursos de almacenamiento y de red que se usan en la replicación, y por los cargos de proceso consumidos durante las migraciones de prueba.
- Azure Migrate admite todos los escenarios de migración que admite Site Recovery. Además, en el caso de máquinas virtuales de VMware, Azure Migrate proporciona una opción de migración sin agente.
- Estamos dando prioridad a las nuevas características de migración solo de la herramienta Azure Migrate:Server Migration. Estas características no están dirigidas a Site Recovery.

## <a name="when-to-use-site-recovery"></a>¿Cuándo usar Site Recovery?

Se debe usar Site Recovery:

- Para la recuperación ante desastres de máquinas locales en Azure.
- Para la recuperación ante desastres de máquinas virtuales de Azure entre regiones de Azure.

Aunque se recomienda usar Azure Migrate para migrar servidores locales a Azure, si ya ha iniciado el recorrido de migración con Site Recovery, puede seguir utilizándolo para completar la migración.  

## <a name="next-steps"></a>Pasos siguientes

> [Revise las preguntas habituales](../migrate/resources-faq.md) sobre Azure Migrate.
