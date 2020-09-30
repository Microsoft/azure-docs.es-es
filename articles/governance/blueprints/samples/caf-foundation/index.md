---
title: Ejemplo de plano técnico de Fundación CAF, introducción
description: Introducción y arquitectura del ejemplo de plano técnico de una base de Cloud Adoption Framework (CAF) para Azure.
ms.date: 09/14/2020
ms.topic: sample
ms.openlocfilehash: cce5ea001b32ae92542f38f8f11427e85075cee9
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90531702"
---
# <a name="overview-of-the-microsoft-cloud-adoption-framework-for-azure-foundation-blueprint-sample"></a>Introducción al ejemplo de plano técnico de fundamentos de Microsoft Cloud Adoption Framework para Azure

El plano técnico de fundamentos de Microsoft Cloud Adoption Framework para Azure (CAF) implementa un conjunto de recursos de infraestructura principales y controles de directiva necesarios para la primera aplicación de Azure para producción. Este plano técnico de fundamentos se basa en el patrón recomendado que se encuentra en CAF.

## <a name="architecture"></a>Architecture

En el ejemplo de plano técnico de fundamentos de CAF se implementan los recursos de infraestructura recomendados en Azure que pueden usar las organizaciones para poner en marcha los controles fundamentales necesarios para administrar su propia nube. En este ejemplo se implementarán y aplicarán los recursos, las directivas y las plantillas que permitirán a una organización comenzar con confianza en Azure.

:::image type="complex" source="../../media/caf-blueprints/caf-foundation-architecture.png" alt-text="Fundación CAF, en la imagen se describe lo que se instala como parte de la guía de CAF para la creación de una base para empezar a trabajar con Azure" border="false":::.
   Describe una arquitectura de Azure que se consigue mediante la implementación del plano técnico de Fundación CAF.  Se puede aplicar a cualquier suscripción que tenga grupos de recursos y que conste de una cuenta de almacenamiento para almacenar registros y análisis de registros configurados para almacenarse en la cuenta de almacenamiento. También se muestra Azure Key Vault configurado con los valores estándar de Azure Security Center. Para acceder a todas estas infraestructuras principales se debe usar Azure Active Directory, mientras que para aplicarlas se debe usar Azure Policy.     
:::image-end:::

Esta implementación incorpora varios servicios de Azure que se usan para proporcionar una base segura, totalmente supervisada y preparada para la empresa. Este entorno se compone de:

- Una instancia de [Azure Key Vault](../../../../key-vault/general/overview.md) utilizada para hospedar los secretos empleados por las máquinas virtuales implementadas en el entorno de servicios compartidos
- La implementación de [Log Analytics](../../../../azure-monitor/overview.md) se realiza para garantizar que todas las acciones y servicios se registran en una ubicación central desde el momento en que empieza la implementación segura en las [cuentas de almacenamiento](../../../../storage/common/storage-introduction.md) para el registro de diagnóstico.
- La implementación de [Azure Security Center](../../../../security-center/security-center-intro.md) (versión estándar) proporciona protección contra amenazas para las cargas de trabajo migradas.
- El plano técnico también define e implementa las [directivas de Azure](../../../policy/overview.md), para lo siguiente: 
  - El etiquetado (CostCenter) aplicado a los grupos de recursos
  - Los recursos anexados al grupo de recursos con la etiqueta CostCenter
  - La región de Azure permitida para recursos y grupos de recursos
  - Las SKU de la cuenta de almacenamiento permitida (se eligen durante la implementación)
  - SKU de máquinas virtuales de Azure permitidas (se eligen durante la implementación)
  - La implementación de Network Watcher requerida 
  - El cifrado de transferencia segura de la cuenta de almacenamiento de Azure requerido
  - Tipos de recursos denegados (se elige durante la implementación)  
- Iniciativas
  - Habilitación de la supervisión en Azure Security Center (definiciones de más de cien directivas)

Todos estos elementos se atienen a los procedimientos probados que se publicaron en [Centro de arquitectura de Azure: Arquitecturas de referencia](/azure/architecture/reference-architectures/).

> [!NOTE]
> Los fundamentos de CAF diseñan una arquitectura básica para las cargas de trabajo.
> Deberá implementar cargas de trabajo adicionales más allá de esta arquitectura básica.

Para más información, consulte el paso de [preparación a Microsoft Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/ready/).

## <a name="next-steps"></a>Pasos siguientes

Ya ha revisado la información general y la arquitectura del ejemplo de plano técnico de fundamentos de CAF.

> [!div class="nextstepaction"]
> [Plano técnico de fundamentos de CAF: Pasos de implementación](./deploy.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).
