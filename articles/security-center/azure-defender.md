---
title: Introducción a Azure Defender y los planes disponibles
description: Conozca los planes, las protecciones y las alertas de Azure Defender. Después, continúe con la habilitación de Azure Defender en sus suscripciones.
author: memildin
ms.author: memildin
ms.date: 9/22/2020
ms.topic: conceptual
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: bb1c1e6443b5087b48aad7c3171bef557707adb1
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90977298"
---
# <a name="introduction-to-azure-defender"></a>Introducción a Azure Defender

Las características de Azure Security Center abarcan los dos extensos pilares de seguridad en la nube:

- **Administración de la posición de seguridad en la nube (CSPM)**
- **Protección de cargas de trabajo en la nube (CWP)**

Las características de CSPM de Security Center, como la puntuación segura y la detección de errores de configuración de seguridad en máquinas Windows y Linux de Azure, forman parte de la experiencia gratuita de Security Center disponible para todos los usuarios de Azure. Use estas características de CSPM para fortalecer su posición y garantizar el cumplimiento normativo.

**Azure Defender** es la plataforma de protección de cargas de trabajo en la nube (CWPP) integrada en Security Center para la protección avanzada e inteligente de las cargas de trabajo híbridas y de Azure.

Este es el panel de Azure Defender en Azure Security Center:

:::image type="content" source="./media/azure-defender/sample-defender-dashboard.png" alt-text="Ejemplo del panel de Azure Defender" lightbox="./media/azure-defender/sample-defender-dashboard.png":::

## <a name="what-resource-types-can-azure-defender-secure"></a>¿Qué tipos de recursos puede proteger Azure Defender?

Azure Defender proporciona alertas de seguridad y protección contra amenazas avanzada para máquinas virtuales, bases de datos SQL, contenedores, aplicaciones web, la red y mucho más.

Al habilitar Azure Defender en el área **Precios y configuración** de Azure Security Center, se habilitan a la vez los siguientes planes de Defender, que proporcionan defensas completas para las capas de proceso, datos y servicio de su entorno:

- [Azure Defender para servidores](defender-for-servers-introduction.md)
- [Azure Defender para App Service](defender-for-app-service-introduction.md)
- [Azure Defender para Storage](defender-for-storage-introduction.md)
- [Azure Defender para SQL](defender-for-sql-introduction.md)
- [Azure Defender para IoT](defender-for-iot-introduction.md)
- [Azure Defender para Kubernetes](defender-for-kubernetes-introduction.md)
- [Azure Defender para registros de contenedor](defender-for-container-registries-introduction.md)
- [Azure Defender para Key Vault](defender-for-key-vault-introduction.md)

Cada uno de estos planes se explica por separado en la documentación de Security Center.


## <a name="hybrid-cloud-protection"></a>Protección de la nube híbrida

Además de defender su entorno de Azure, puede agregar funcionalidades de Azure Defender al entorno de nube híbrida:

- Protección de los servidores que no son de Azure
- Proteja sus máquinas virtuales de otras nubes (como AWS y GCP)
- Protección de los dispositivos IoT

Obtenga inteligencia sobre amenazas personalizada y alertas prioritarias según su entorno específico para que pueda centrarse en lo que más le importa.

Implemente [Azure Arc](https://azure.microsoft.com/services/azure-arc/) y permita que Azure Defender amplíe la protección a las máquinas virtuales locales y de varias nubes y a las bases de datos SQL. Azure Arc para servidores es un servicio gratuito, pero los servicios que se usan en servidores habilitados para Arc, por ejemplo, Azure Defender, se cobran según los precios de ese servicio.

[Más información sobre Azure Arc](https://docs.microsoft.com/azure/azure-arc/overview).


## <a name="azure-defender-alerts"></a>Alertas de Azure Defender 

Cuando Azure Defender detecta una amenaza en cualquiera de las áreas del entorno, genera una alerta. Estas alertas describen los detalles de los recursos afectados, los pasos de corrección sugeridos y, en algunos casos, una opción para desencadenar una aplicación lógica como respuesta.

Tanto si Security Center genera una alerta, como si la recibe de un producto de seguridad integrado, puede exportarse. Para exportar las alertas a Azure Sentinel, a cualquier SIEM de terceros o a cualquier otra herramienta externa, siga las instrucciones de [Exportación de alertas a un SIEM](continuous-export.md).

> [!NOTE]
> Las alertas de orígenes diferentes pueden tardar un tiempo distinto en aparecer. Por ejemplo, las alertas que requieren un análisis del tráfico de red pueden tardar más en aparecer que las alertas relacionadas con procesos sospechosos que se ejecutan en máquinas virtuales.


## <a name="azure-defender-advanced-protection-capabilities"></a>Funcionalidades de protección avanzada de Azure Defender

Azure Defender usa análisis avanzado para las recomendaciones adaptadas relacionadas con los recursos. 

Entre las protecciones se incluyen la protección de los puertos de administración de las máquinas virtuales con acceso Just-in-Time y controles de aplicaciones adaptables para crear listas de permitidos con las aplicaciones que deben o no ejecutarse en las máquinas. 

Use los iconos de protección avanzada del panel de Azure Defender para supervisar y configurar cada una de estas protecciones. 

## <a name="vulnerability-assessment-and-management"></a>Evaluación y administración de vulnerabilidades

Azure Defender incluye el examen de vulnerabilidades de las máquinas virtuales y de los registros de contenedor sin costo adicional. Los exámenes se basan en la tecnología de Qualys, pero no se necesita ninguna licencia ni cuenta de Qualys, ya que todo se administra íntegramente en Security Center. 

Revise los resultados de estos exámenes de vulnerabilidades y responda a todos ellos desde Security Center. De esta forma, Security Center se aproxima a convertirse en el panel por excelencia para todos los trabajos de seguridad en la nube.

Más información en las siguientes páginas:

- [Solución de evaluación de vulnerabilidades integrada en Security Center para Azure Virtual Machines](deploy-vulnerability-assessment-vm.md)
- [Identificación de vulnerabilidades en las imágenes de los registros de contenedor de Azure](defender-for-container-registries-usage.md#identify-vulnerabilities-in-images-in-other-container-registries)



## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió las ventajas de Azure Defender. 

> [!div class="nextstepaction"]
> [Habilitación de Azure Defender](security-center-pricing.md)