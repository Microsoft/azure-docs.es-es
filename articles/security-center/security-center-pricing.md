---
title: Precios de los niveles de Azure Security Center
description: 'Azure Security Center se ofrece en dos niveles: Gratis y Estándar. En esta página se muestra cómo actualizar de Gratis a Estándar.'
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
ms.openlocfilehash: 882f8ab7e4f55809759f3a94fd04fb2c410fe188
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089726"
---
# <a name="upgrade-to-standard-tier-for-enhanced-security"></a>Actualización al nivel Estándar para mejorar la seguridad

Azure Security Center ofrece una administración de seguridad unificada y una protección contra amenazas avanzada para las cargas de trabajo que se ejecutan en Azure, de forma local y en otras nubes. Ofrece visibilidad y control sobre las cargas de trabajo de nube híbrida, defensas activas que reducen la exposición a las amenazas y una detección inteligente, lo que le permitirá mantenerse al día de los ciberataques, que se expanden rápidamente.

## <a name="pricing-tiers"></a>Planes de tarifa
Security Center se ofrece en dos niveles:

- El nivel **Gratis** está habilitado en todas las suscripciones de Azure la primera vez que visite el panel de Azure Security Center en Azure Portal, o si se habilitó mediante programación a través de una API. El nivel gratuito proporciona recomendaciones de directivas de seguridad, de evaluación continua de la seguridad y de seguridad que requieren acción para que pueda proteger sus recursos de Azure.

- El nivel **Estándar** amplía las capacidades del nivel Gratis a las cargas de trabajo que se ejecutan en nubes privadas y otras nubes públicas, lo que ofrece una administración de seguridad unificada y protección contra amenazas en todas sus cargas de trabajo de nube híbrida. El nivel estándar también incorpora funcionalidades de protección contra amenazas, donde se emplea el análisis de comportamiento integrado y el aprendizaje automático para identificar ataques y vulnerabilidades de día cero, controles de acceso y de aplicación para reducir la exposición a los ataques a la red y al malware, etc. Además, el nivel estándar agrega detección de vulnerabilidades a las máquinas virtuales. Puede probar el nivel estándar de manera gratuita. El nivel estándar de Security Center es compatible con recursos de Azure, incluidas las máquinas virtuales, los conjuntos de escalado de máquinas virtuales, App Service, los servidores SQL Server y las cuentas de almacenamiento. Si tiene el nivel estándar de Azure Security Center, puede optar por no recibir soporte técnico en función del tipo de recurso. 

La mayoría de las evaluaciones de seguridad del nivel Gratis para las máquinas virtuales, así como muchas de las alertas de seguridad de nivel estándar, requieren la instalación de la funcionalidad del agente de Log Analytics. Puede habilitar el aprovisionamiento automático en Security Center para implementar automáticamente el agente en las máquinas virtuales de Azure.

## <a name="try-standard-tier-free-for-30-days"></a>Pruebe el nivel estándar de forma gratuita durante 30 días
El nivel estándar es gratuito durante los primeros 30 días. Después de 30 días, si decide seguir usando el servicio, se empezará a cobrar automáticamente por su uso.

Puede actualizar una suscripción a Azure entera al nivel estándar, que heredarán todos los recursos dentro de la suscripción.

Para obtener el nivel estándar:

1. En el menú principal de **Security Center**, seleccione **Pricing & settings** (Precios y configuración).
2. Seleccione la suscripción que quiera actualizar al nivel estándar.
3. Seleccione **Plan de tarifa**.
4. Seleccione **Estándar** para efectuar la actualización.
5. Haga clic en **Save**(Guardar).

[![Precios de Security Center](media/security-center-pricing/pricing-tier-page.png)](media/security-center-pricing/pricing-tier-page.png#lightbox)

> [!NOTE]
> Para habilitar todas las características de Security Center, lo que incluye las funcionalidades de protección contra amenazas, debe aplicar el plan de tarifa estándar a la suscripción que contiene las cargas de trabajo aplicables. La configuración de los precios para un área de trabajo no habilita el acceso a VM Just-In-Time, los controles de aplicación adaptables ni las detecciones de red para los recursos de Azure. 
>
> Puede habilitar la protección contra amenazas para **cuentas de Azure Storage** en el nivel de suscripción o de recursos.
> Puede habilitar la protección contra amenazas para **servidores SQL de Azure SQL Database** en el nivel de suscripción o de recursos.
> Puede habilitar la protección contra amenazas para **Azure Database for MariaDB/MySQL/PostgreSQL** solo en el nivel de recursos.


## <a name="why-upgrade-to-standard"></a>¿Por qué actualizar al nivel estándar?
Security Center ofrece una seguridad mejorada y una protección contra amenazas para las cargas de trabajo de nube híbrida, como, por ejemplo:

- **Seguridad híbrida**: Obtenga una vista unificada de la seguridad de todas sus cargas de trabajo locales y en la nube. Aplique directivas de seguridad y evalúe constantemente la seguridad de las cargas de trabajo de nube híbrida para garantizar el cumplimiento normativo con los estándares de seguridad. Recopile, busque y analice datos de seguridad de varios orígenes, incluidos firewalls y otras soluciones de partners.
- **Alertas de seguridad**: use análisis avanzado y Microsoft Intelligent Security Graph para adelantarse a los ciberataques en constante cambio. Aproveche la funcionalidad integrada de análisis del comportamiento y aprendizaje automático para identificar ataques y vulnerabilidades de seguridad de día cero. Supervise las redes, las máquinas y los servicios en la nube para detectar ataques entrantes y actividad posterior a una infracción de seguridad. Optimice la investigación con herramientas interactivas e inteligencia de amenazas contextual.
- **Detección de vulnerabilidades de máquinas virtuales**: implemente fácilmente un escáner en todas las máquinas virtuales que proporciona la solución más avanzada del sector para la administración de vulnerabilidades. Vea, investigue y corrija los hallazgos directamente dentro de Security Center. 
- **Controles de acceso y de aplicación**: Bloquee el malware y otras aplicaciones no deseadas aplicando recomendaciones de inclusión en lista blanca basadas en el aprendizaje automático, adaptadas a sus cargas de trabajo específicas. Reduzca la superficie de la red que está expuesta a ataques mediante un acceso Just-In-Time controlado a los puertos de administración de las VM de Azure. Esto reduce drásticamente la exposición a ataques por fuerza bruta y otros tipos de ataque de red.
- **Características de seguridad de contenedor**: Aproveche la administración de vulnerabilidades y la protección contra amenazas en tiempo real en los entornos en contenedores. Al habilitar el recurso de registros de contenedor, se puede tardar hasta 12 horas para que se habiliten todas las características. Los cargos se basan en el número de imágenes de contenedor únicas insertadas en el registro conectado. Una vez que se haya analizado una imagen, no se le cobrará de nuevo a menos que se modifique e inserte una vez más. 




## <a name="next-steps"></a>Pasos siguientes
En este artículo se han presentado los precios de Security Center. Para más información sobre la seguridad mejorada y la protección contra amenazas avanzada de nivel estándar, consulte:

- [Protección contra amenazas en Azure Security Center](threat-protection.md)
- [Control de acceso a VM Just-In-Time](security-center-just-in-time.md)
- [Introducción a la seguridad del contenedor](container-security.md)
- [Detalles de precios en la moneda de su elección y según su región](https://azure.microsoft.com/pricing/details/security-center/)