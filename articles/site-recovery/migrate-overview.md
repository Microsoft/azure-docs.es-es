---
title: Migración de servidores y máquinas virtuales a Azure con Azure Site Recovery
description: Describe cómo migrar máquinas virtuales IaaS de Azure y locales con el servicio Azure Site Recovery.
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 07/27/2020
ms.author: raynew
ms.openlocfilehash: aaf01dcd63c21e4741456f4f7fccaf22b8fbfffc
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281317"
---
# <a name="about-migration"></a>Información sobre la migración

Use Azure Migrate para migrar máquinas virtuales y servidores a Azure, en lugar del servicio Azure Site Recovery. [Más información](../migrate/migrate-services-overview.md) acerca de Azure Migrate.


## <a name="why-use-azure-migrate"></a>¿Por qué usar Azure Migrate?

El uso de Azure Migrate para la migración proporciona una serie de ventajas:
 
 
- Azure Migrate ofrece un centro de conectividad centralizado para la detección, evaluación y migración a Azure.
- El uso de Azure Migrate proporciona interoperabilidad y extensibilidad futura con herramientas de Azure Migrate, otros servicios de Azure y herramientas de terceros.
- La herramienta Azure Migrate:Server Migration está diseñada específicamente para la migración de servidores a Azure. Está optimizada para la migración. No es necesario obtener información sobre los conceptos y escenarios que no son directamente pertinentes para la migración. 
- No hay cargos por uso de herramientas para la migración durante 180 días a partir del momento en que se inicia la replicación de una máquina virtual. Esto le proporciona tiempo para completar la migración. Solo paga por los recursos de almacenamiento y de red que se usan en la replicación, y por los cargos de proceso consumidos durante las migraciones de prueba.
- En el caso de las máquinas virtuales de VMware, Azure Migrate proporciona migración sin agente y con agente.
- Estamos dando prioridad a las nuevas características de migración solo de la herramienta Azure Migrate:Server Migration.

## <a name="when-to-use-site-recovery"></a>¿Cuándo usar Site Recovery?

Se debe usar Site Recovery:

- Para la recuperación ante desastres de máquinas locales en Azure.
- Para la recuperación ante desastres de máquinas virtuales de Azure entre regiones de Azure.

Aunque se recomienda usar Azure Migrate para migrar servidores locales a Azure, si ya ha iniciado el recorrido de migración con Site Recovery, puede seguir utilizándolo para completar la migración.  

## <a name="next-steps"></a>Pasos siguientes

> [Revise las preguntas habituales](../migrate/resources-faq.md) sobre Azure Migrate.
