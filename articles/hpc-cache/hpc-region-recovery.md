---
title: Redundancia regional y recuperación de conmutación por error con Azure HPC Cache
description: Técnicas para proporcionar funcionalidades de conmutación por error para la recuperación ante desastres con Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: conceptual
ms.date: 10/30/2019
ms.author: v-erkel
ms.openlocfilehash: 9159807f55ae52393b8fccec339fcc94c3e4ebb0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87061383"
---
# <a name="use-multiple-caches-for-regional-failover-recovery"></a>Uso de varias cachés para la recuperación de la conmutación por error regional

Cada instancia de Azure HPC Cache se ejecuta en una suscripción concreta y en una región. Esto significa que es posible que el flujo de trabajo de la memoria caché se interrumpa si la región sufre una interrupción completa.

En este artículo se describe una estrategia para reducir el riesgo de interrupción del trabajo mediante el uso de una segunda región para la conmutación por error de la caché.

La clave usa un almacenamiento de back-end que es accesible desde varias regiones. Este almacenamiento puede ser un sistema NAS local con compatibilidad con DNS adecuada o una instancia de Azure Blob Storage que resida en una región distinta de la caché.

A medida que se realiza el flujo de trabajo en la región primaria, los datos se guardan en el almacenamiento a largo plazo fuera de la región. Si la región de la caché deja de estar disponible, puede crear una instancia de Azure HPC Cache duplicada en una región secundaria, conectarse al mismo almacenamiento y reanudar el trabajo desde la nueva caché.

## <a name="planning-for-regional-failover"></a>Planeación de la conmutación por error regional

Para configurar una memoria caché preparada para una posible conmutación por error, siga estos pasos:

1. Asegúrese de que el almacenamiento de back-end es accesible en una segunda región.
1. Al planear la creación de la instancia de caché principal, también debe preparar la replicación de este proceso de configuración en la segunda región. Incluya estos elementos:

   1. Red virtual y estructura de subred
   1. Capacidad de la memoria caché
   1. Detalles del destino de almacenamiento, nombres y rutas de acceso de espacio de nombres
   1. Detalles acerca de las máquinas cliente, si se encuentran en la misma región que la memoria caché
   1. Comando de montaje para que lo usen los clientes de caché

   > [!NOTE]
   > La instancia de Azure HPC Cache se puede crear mediante programación, mediante una [plantilla de Azure Resource Manager](../azure-resource-manager/templates/overview.md) o mediante el acceso directo a su API. Póngase en contacto con el equipo de Azure HPC Cache para más información.

## <a name="failover-example"></a>Ejemplo de conmutación por error

Por ejemplo, imagine que desea ubicar la instancia de Azure HPC Cache en la región Este de EE. UU. Tendrá acceso a los datos almacenados en el centro de datos local.

Puede usar una caché en la región Oeste de EE. UU. 2 como copia de seguridad de la conmutación por error.

Al crear la caché en la región Este de EE. UU., prepare una segunda caché para su implementación en la región Oeste de EE. UU. 2. Puede usar scripts o plantillas para automatizar esta preparación.

En caso de que se produzca un error en toda la región Este de EE. UU., cree la caché que ha preparado en la región Oeste de EE. UU. 2.

Una vez creada la caché, agregue destinos de almacenamiento que apunten a los mismos almacenes de datos locales y use las mismas rutas de acceso de espacios de nombres agregadas que los destinos de almacenamiento de la caché anterior.

Si los clientes originales se ven afectados, cree nuevos clientes en la región Oeste de EE. UU. 2 para su uso con la nueva caché.

Todos los clientes tendrán que montar la nueva caché, incluso si estos no se vieron afectados por la interrupción de la región. La nueva memoria caché tiene diferentes direcciones de montaje con respecto a la antigua.

## <a name="learn-more"></a>Más información

La guía de arquitectura de aplicaciones de Azure incluye más información acerca de la [recuperación después de una interrupción del servicio en toda la región](<https://docs.microsoft.com/azure/architecture/resiliency/recovery-loss-azure-region>).
