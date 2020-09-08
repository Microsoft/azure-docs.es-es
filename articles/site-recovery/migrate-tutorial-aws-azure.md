---
title: Migración de máquinas virtuales de AWS con Azure Migrate
description: En este artículo se describen las opciones para migrar instancias de AWS a Azure y se recomienda Azure Migrate.
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 07/27/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 83d8106b9f206da9095c37b6179bdda8001b439c
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852776"
---
# <a name="migrate-amazon-web-services-aws-vms-to-azure"></a>Migración de máquinas virtuales de Amazon Web Services (AWS) a Azure

En este artículo se describen las opciones para migrar instancias de Amazon Web Services (AWS) a Azure.

## <a name="migrate-with-azure-migrate"></a>Migración con Azure Migrate

Se recomienda migrar las instancias de EC2 de AWS a Azure mediante el servicio [Azure Migrate](../migrate/migrate-services-overview.md). Azure Migrate está diseñado específicamente para la migración de servidores. Azure Migrate proporciona un centro centralizado para la detección, evaluación y migración de máquinas locales a Azure.

[Obtenga información sobre cómo](../migrate/tutorial-migrate-aws-virtual-machines.md) migrar instancias de AWS con Azure Migrate. 


## <a name="migrate-with-site-recovery"></a>Migración con Site Recovery

Site Recovery debe usarse solo para la recuperación ante desastres, no para la migración.

Si ya usa Azure Site Recovery y quiere seguir usándolo para la migración de AWS, siga los mismos pasos que usa para configurar la [recuperación ante desastres de máquinas físicas](physical-azure-disaster-recovery.md).


> [!NOTE]
> Cuando ejecuta una conmutación por error para la recuperación ante desastres, como último paso, confirma la conmutación por error. Al migrar instancias de AWS, la opción **Confirmar** no es pertinente. En su lugar, seleccione la opción **Completar la migración**. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Revise las preguntas habituales](../migrate/resources-faq.md) sobre Azure Migrate.
