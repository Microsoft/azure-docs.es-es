---
title: 'Azure Defender para Kubernetes: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para Kubernetes.
author: memildin
ms.author: memildin
ms.date: 9/12/2020
ms.topic: conceptual
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: e72875135243733a6acf03bb3aa6fb9405392d9d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932449"
---
# <a name="introduction-to-azure-defender-for-kubernetes"></a>Introducción a Azure Defender para Kubernetes

Azure Kubernetes Service (AKS) es un servicio que administra Microsoft y que permite desarrollar, implementar y administrar las aplicaciones en contenedores.

Azure Security Center y AKS forman la mejor oferta de seguridad de Kubernetes nativa en la nube, y juntos proporcionan protección del entorno, protección de la carga de trabajo y protección en tiempo de ejecución, tal y como se describe a continuación.

Para la detección de amenazas en los clústeres de Kubernetes, habilite **Azure Defender para Kubernetes**.

La detección de amenazas a nivel de host en los nodos de AKS de Linux está disponible si habilita [Azure Defender para servidores](defender-for-servers-introduction.md).

## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponible con carácter general|
|Precios:|**Azure Defender para Kubernetes** se factura como se muestra en la [página de precios](security-center-pricing.md).|
|Roles y permisos necesarios:|**Administración de seguridad** puede descartar las alertas.<br>**El lector de seguridad** puede ver los resultados.|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||

## <a name="what-are-the-benefits-of-azure-defender-for-kubernetes"></a>¿Cuáles son las ventajas de Azure Defender para Kubernetes?

### <a name="run-time-protection"></a>Protección en tiempo de ejecución

Gracias al análisis continuo de los siguientes orígenes de AKS, Security Center ofrece protección frente a amenazas en tiempo real de los entornos en contenedores y genera alertas para las amenazas y la actividad malintencionada detectadas en el nivel de host *y* de clúster de AKS. Puede usar esta información para corregir problemas de seguridad y mejorar la seguridad de los contenedores rápidamente.

Security Center proporciona protección contra amenazas en diferentes niveles: 

- **Nivel de host (proporcionado por Azure Defender para servidores)** : con el mismo agente de Log Analytics que Security Center usa en otras máquinas virtuales, Azure Defender supervisa los nodos de AKS de Linux en busca de actividades sospechosas, como la detección de shell web y la conexión con direcciones IP sospechosas conocidas. El agente también supervisa análisis específicos de contenedores, como la creación de contenedores con privilegios, las actividades sospechosas de acceso a los servidores de API y los servidores de Secure Shell (SSH) que se ejecutan dentro de un contenedor de Docker.

    Para obtener una lista de las alertas de nivel de host de AKS, consulte la [Tabla de referencia de alertas](https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-containerhost).

    >[!IMPORTANT]
    > Si decide no instalar los agentes en los hosts, solo recibirá un subconjunto de las ventajas de la protección contra amenazas y las alertas de seguridad. Aún así, seguirá recibiendo alertas relacionadas con el análisis de redes y las comunicaciones con servidores malintencionados.

- **Nivel de clúster de AKS (proporcionado por Azure Defender para Kubernetes)** : en el nivel de clúster, la protección contra amenazas se basa en analizar los registros de auditoría de Kubernetes. Para habilitar esta supervisión **sin agente**, habilite Azure Defender. Para generar alertas en este nivel, Security Center supervisa los servicios que administra AKS con los registros que recupera el mismo AKS. Entre los ejemplos de eventos en este nivel se incluyen los paneles de Kubernetes expuestos y la creación de roles con privilegios elevados y de montajes confidenciales.

    Para obtener una lista de las alertas de nivel de clúster de AKS, consulte la [Tabla de referencia de alertas](https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-akscluster).

    >[!NOTE]
    > Igualmente, Security Center genera alertas de seguridad para las acciones y las implementaciones de Azure Kubernetes Service que se producen después de que la opción de Kubernetes esté habilitada en la configuración de la suscripción. 

Además, nuestro equipo global de investigadores de seguridad supervisa constantemente el panorama de las amenazas. Agregan alertas específicas del contenedor y vulnerabilidades a medida que se detectan.

> [!TIP]
> Puede simular las alertas de contenedor al seguir las instrucciones de [esta entrada de blog](https://techcommunity.microsoft.com/t5/azure-security-center/how-to-demonstrate-the-new-containers-features-in-azure-security/ba-p/1011270).



## <a name="how-does-security-centers-kubernetes-protection-work"></a>¿Cómo funciona la protección de Kubernetes de Security Center?

A continuación, se muestra un diagrama de alto nivel de la interacción entre Azure Security Center, Azure Kubernetes Service y Azure Policy.

Puede ver que los elementos recibidos y analizados por Security Center incluyen:

- registros de auditoría del servidor de API,
- eventos de seguridad sin procesar del agente de Log Analytics,
- información de configuración del clúster de AKS y
- configuración de la carga de trabajo en Azure Policy (mediante el complemento de **Azure Policy para Kubernetes**). [Más información sobre los procedimientos recomendados de protección de cargas de trabajo con el control de admisión de Kubernetes](container-security.md#workload-protection-best-practices-using-kubernetes-admission-control)

:::image type="content" source="./media/defender-for-kubernetes-intro/kubernetes-service-security-center-integration-detailed.png" alt-text="Arquitectura de alto nivel de la interacción entre Azure Security Center, Azure Container Service y Azure Policy" lightbox="./media/defender-for-kubernetes-intro/kubernetes-service-security-center-integration-detailed.png":::




## <a name="azure-defender-for-kubernetes---faq"></a>Preguntas frecuentes de Azure Defender para Kubernetes

### <a name="can-i-still-get-aks-protections-without-the-log-analytics-agent"></a>¿Puedo obtener protecciones de AKS aún sin el agente de Log Analytics?

Como se mencionó anteriormente, el plan opcional de **Azure Defender para Kubernetes** proporciona protecciones a nivel de clúster, y el agente de Log Analytics de **Azure Defender para servidores** protege los nodos. 

Se recomienda implementar ambos para obtener la protección más completa posible.

Si decide no instalar el agente en los hosts, solo recibirá un subconjunto de las ventajas de la protección contra amenazas y las alertas de seguridad. Aún así, seguirá recibiendo alertas relacionadas con el análisis de redes y las comunicaciones con servidores malintencionados.


## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre la protección de Kubernetes de Security Center, incluido Azure Defender para Kubernetes. 

Para obtener material relacionado, consulte los siguientes artículos: 

- [Habilitación de Azure Defender](security-center-pricing.md)
- [Exportación de alertas a una instancia de Azure Sentinel o a un SIEM de terceros](continuous-export.md)
- [Tabla de referencia de alertas](alerts-reference.md)