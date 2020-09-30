---
title: Precios de Azure Security Center
description: 'Azure Security Center se ofrece en dos modos: con y sin Azure Defender.'
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 4d1364cd-7847-425a-bb3a-722cb0779f78
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/28/2020
ms.author: memildin
ms.openlocfilehash: 6d2127af2c2c9e04141551dae72f0177f495b165
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904781"
---
# <a name="pricing-of-azure-security-center"></a>Precios de Azure Security Center
Azure Security Center ofrece una administración de seguridad unificada y una protección contra amenazas avanzada para las cargas de trabajo que se ejecutan en Azure, de forma local y en otras nubes. Ofrece visibilidad y control sobre las cargas de trabajo de nube híbrida, defensas activas que reducen la exposición a las amenazas y una detección inteligente, lo que le permitirá mantenerse al día de los ciberataques, que se expanden rápidamente.


## <a name="free-option-vs-azure-defender-enabled"></a>Opción gratis frente a Azure Defender habilitado

Security Center se ofrece en dos modos:

- **Azure Defender DESACTIVADO** (Gratis): Security Center sin Azure Defender está habilitado en todas las suscripciones de Azure la primera vez que visite el panel de Azure Security Center en Azure Portal, o si se habilitó mediante programación a través de una API. Este modo gratis proporciona recomendaciones de directivas de seguridad, de evaluación continua de la seguridad y de seguridad que requieren acción para que pueda proteger sus recursos de Azure.

- **Azure Defender ACTIVADO**: la habilitación de Azure Defender amplía las capacidades del nivel Gratis a las cargas de trabajo que se ejecutan en nubes privadas y otras nubes públicas, lo que ofrece una administración de seguridad unificada y protección contra amenazas en todas sus cargas de trabajo de nube híbrida. Algunas de las principales características de Azure Defender:

    - **Seguridad híbrida**: Obtenga una vista unificada de la seguridad de todas sus cargas de trabajo locales y en la nube. Aplique directivas de seguridad y evalúe constantemente la seguridad de las cargas de trabajo de nube híbrida para garantizar el cumplimiento normativo con los estándares de seguridad. Recopile, busque y analice datos de seguridad de varios orígenes, incluidos firewalls y otras soluciones de partners.
    - **Alertas de protección contra amenazas**: el análisis de comportamiento avanzado y Microsoft Intelligent Security Graph proporcionan una ventaja frente a los ataques cibernéticos en evolución. Aproveche la funcionalidad integrada de análisis del comportamiento y aprendizaje automático para identificar ataques y vulnerabilidades de seguridad de día cero. Supervise las redes, las máquinas y los servicios en la nube para detectar ataques entrantes y actividad posterior a una infracción de seguridad. Optimice la investigación con herramientas interactivas e inteligencia de amenazas contextual.
    - **Detección de vulnerabilidades de máquinas virtuales y registros de contenedor**: implemente fácilmente un escáner en todas las máquinas virtuales que proporciona la solución más avanzada del sector para la administración de vulnerabilidades. Vea, investigue y corrija los hallazgos directamente dentro de Security Center. 
    - **Controles de acceso y de aplicación**: bloquee el malware y otras aplicaciones no deseadas aplicando recomendaciones de basadas en el aprendizaje automático, adaptadas a sus cargas de trabajo específicas, para crear listas de permitidos y denegados. Reduzca la superficie de la red que está expuesta a ataques mediante un acceso Just-In-Time controlado a los puertos de administración de las VM de Azure. Esto reduce drásticamente la exposición a ataques por fuerza bruta y otros tipos de ataque de red.
    - **Características de seguridad de contenedor**: Aproveche la administración de vulnerabilidades y la protección contra amenazas en tiempo real en los entornos en contenedores. Al habilitar **Azure Defender para registros de contenedor**, pueden pasar hasta 12 horas para que se habiliten todas las características. Los cargos se basan en el número de imágenes de contenedor únicas insertadas en el registro conectado. Una vez que se haya analizado una imagen, no se le cobrará de nuevo a menos que se modifique e inserte una vez más. 

## <a name="try-azure-defender-free-for-30-days"></a>Prueba de Azure Defender gratis durante 30 días

Azure Defender es gratis durante los primeros 30 días. Después de 30 días, si decide continuar usando el servicio, empezaremos a cobrar automáticamente el uso.

## <a name="enable-azure-defender"></a>Habilitación de Azure Defender

Puede proteger toda una suscripción de Azure con Azure Defender y todos los recursos de la suscripción heredarán las protecciones.

Para habilitar Azure Defender:

1. En el menú principal de Security Center, seleccione **Precios y configuración**.
1. Seleccione la suscripción que quiere actualizar.
1. Seleccione **Azure Defender está activado** para actualizar.
1. Seleccione **Guardar**.

A continuación se muestra la página de precios de una suscripción de ejemplo. Observará que cada plan de Azure Defender tiene un precio independiente y se puede activar o desactivar de forma independiente.

:::image type="content" source="./media/security-center-pricing/pricing-tier-page.png" alt-text="Página de precios de Security Center en el portal":::

> [!NOTE]
> Para habilitar todas las características de Security Center, incluidas las funcionalidades de protección contra amenazas, debe habilitar Azure Defender en la suscripción que contenga las cargas de trabajo aplicables. La habilitación en el nivel de área de trabajo no habilita el acceso a VM Just-In-Time, los controles de aplicación adaptables ni las detecciones de red para los recursos de Azure. 
>
> Puede habilitar **Azure Defender para cuentas de Storage** en el nivel de suscripción o de recursos.
> Puede habilitar **Azure Defender para SQL** en el nivel de suscripción o de recursos.
> Puede habilitar la protección contra amenazas para **Azure Database for MariaDB/MySQL/PostgreSQL** solo en el nivel de recursos.


## <a name="next-steps"></a>Pasos siguientes
En este artículo se han presentado los precios de Security Center. Para obtener material relacionado, vea lo siguiente:

- [Optimización de los costos de la carga de trabajo de Azure](https://azure.microsoft.com/blog/how-to-optimize-your-azure-workload-costs/)
- [Detalles de precios en la moneda de su elección y según su región](https://azure.microsoft.com/pricing/details/security-center/)
- Quizá quiera administrar los costos y limitar la cantidad de datos recopilados para una solución mediante la limitación a un conjunto determinado de agentes. La [selección de destino de solución](../operations-management-suite/operations-management-suite-solution-targeting.md) permite aplicar un ámbito a la solución y tener como destino un subconjunto de equipos en el área de trabajo. Si usa la selección de destino de solución, Security Center muestra el área de trabajo como si no tuviera una solución.