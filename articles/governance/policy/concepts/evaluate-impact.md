---
title: Evaluación del efecto de una nueva definición de Azure Policy
description: Comprenda el proceso que debe seguir al introducir una nueva definición de directiva en el entorno de Azure.
ms.date: 08/17/2020
ms.topic: conceptual
ms.openlocfilehash: 243f04cc20e1fb7167306b925a0e494b34cf1267
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544709"
---
# <a name="evaluate-the-impact-of-a-new-azure-policy-definition"></a>Evaluación del efecto de una nueva definición de Azure Policy

Azure Policy es una herramienta eficaz para administrar los recursos de Azure a los estándares empresariales y satisfacer las necesidades de cumplimiento normativo. Cuando las personas, los procesos o las canalizaciones crean o actualizan recursos, Azure Policy revisa la solicitud. Cuando el efecto de la definición de directiva es [Append](./effects.md#deny) o [DeployIfNotExists](./effects.md#deployifnotexists), la directiva modifica la solicitud o la agrega. Cuando el efecto de la definición de directiva es [Audit](./effects.md#audit) o [AuditIfNotExists](./effects.md#auditifnotexists), la directiva provoca la creación de una entrada del registro de actividad. Y cuando el efecto de la definición de directiva es [Deny](./effects.md#deny), la directiva detiene la creación o modificación de la solicitud.

Estos resultados son exactamente lo que se desea cuando se sabe que la directiva se ha definido correctamente. Sin embargo, es importante validar que una nueva directiva funciona según lo previsto antes de permitirle cambiar o bloquear el trabajo. La validación debe asegurarse de que solo se determina que los recursos deseados no son compatibles y que no se incluyen correctamente los recursos compatibles (lo que se conoce como _falso positivo_) en los resultados.

El enfoque recomendado para validar una nueva definición de directiva es seguir estos pasos:

- Definición de la directiva de forma estricta
- Maximización de los recursos existentes
- Auditoría de las solicitudes de recursos nuevas o actualizadas
- Implementación de la directiva en los recursos
- Supervisión continua

## <a name="tightly-define-your-policy"></a>Definición de la directiva de forma estricta

Es importante comprender cómo se implementa la directiva empresarial como una definición de directiva y la relación de los recursos de Azure con otros servicios de Azure. Este paso se realiza mediante la [identificación de los requisitos](../tutorials/create-custom-policy-definition.md#identify-requirements) y la [determinación de las propiedades del recurso](../tutorials/create-custom-policy-definition.md#determine-resource-properties).
Pero también es importante ver más allá de la definición de directiva empresarial. ¿Su estado de directiva es que "Todas las máquinas virtuales tienen que..."? ¿Qué ocurre con otros servicios de Azure que hacen uso de máquinas virtuales, como HDInsight o AKS? Al definir una directiva, debemos tener en cuenta cómo afecta esta directiva a los recursos utilizados por otros servicios.

Por esta razón, las definiciones de directiva deben ser tan definidas y centradas en los recursos y las propiedades que se necesitan para evaluar su cumplimiento como sea posible.

## <a name="audit-existing-resources"></a>Auditoría de recursos existentes

Antes de considerar la administración de recursos nuevos o actualizados con la nueva definición de directiva, es mejor ver cómo evalúa un subconjunto limitado de recursos existentes, como un grupo de recursos de prueba. Use el [modo de cumplimiento](./assignment-structure.md#enforcement-mode) _Deshabilitado_ (DoNotEnforce) en la asignación de directiva para evitar que se desencadene el [efecto](./effects.md) o que se creen las entradas del registro de actividad.

Este paso le ofrece la oportunidad de evaluar los resultados de cumplimiento de la nueva directiva en los recursos existentes sin afectar al flujo de trabajo. Compruebe que los recursos no compatibles estén marcados en consecuencia (_falso positivo_) y que todos los recursos que espera que sean no compatibles se marquen correctamente.
Cuando el subconjunto inicial de recursos se valida según lo previsto, amplíe lentamente la evaluación a todos los recursos existentes.

La evaluación de los recursos existentes de esta manera también ofrece la posibilidad de corregir recursos no compatibles antes de la implementación completa de la nueva directiva. Esta limpieza puede realizarse manualmente o a través de una [tarea de corrección](../how-to/remediate-resources.md) si el efecto de definición de directiva es _DeployIfNotExists_.

## <a name="audit-new-or-updated-resources"></a>Auditoría de recursos nuevos o actualizados

Cuando haya validado la nueva definición de directiva que se está notificando correctamente en los recursos existentes, es el momento de examinar el impacto de la directiva cuando se crean o actualizan recursos. Si la definición de directiva admite la parametrización de efectos, use [Audit](./effects.md#audit). Esta configuración permite supervisar la creación y actualización de recursos para ver si la nueva definición de directiva desencadena una entrada en el registro de actividad de Azure para un recurso que no es compatible sin que afecte a las solicitudes o al trabajo existente.

Se recomienda que actualice y cree nuevos recursos que coincidan con la definición de directiva para ver que el efecto _Audit_ se está desencadenando correctamente cuando se espera. Esté atento a las solicitudes de recursos que no deben verse afectadas por la nueva definición de directiva que desencadena el efecto _Audit_.
Estos recursos afectados son otro ejemplo de _falsos positivos_ y deben corregirse en la definición de directiva antes de la implementación completa.

Si la definición de directiva se cambia en esta fase de pruebas, se recomienda comenzar el proceso de validación con la auditoría de los recursos existentes. Es probable que un cambio en la definición de directiva para un _falso positivo_ en los recursos nuevos o actualizados también afecte a los recursos existentes.

## <a name="deploy-your-policy-to-resources"></a>Implementación de la directiva en los recursos

Después de completar la validación de la nueva definición de directiva con los recursos existentes y las solicitudes de recursos nuevos o actualizados, comienza el proceso de implementación de directiva. Se recomienda crear en primer lugar la asignación de directiva para la nueva definición de directiva en un subconjunto de todos los recursos, como un grupo de recursos. Después de validar la implementación inicial, extienda el ámbito de la directiva a niveles cada vez más amplios, como suscripciones y grupos de administración. Esta expansión se consigue con la eliminación de la asignación y la creación de una nueva en los ámbitos de destino hasta que se asigna al ámbito completo de los recursos que se van a cubrir con la nueva definición de directiva.

Durante el lanzamiento, si se encuentran recursos que deben estar exentos de la nueva definición de directiva, puede solucionarlos de una de las siguientes maneras:

- Actualización de la definición de directiva para que sea más explícita para reducir el impacto imprevisto
- Cambio del ámbito de la asignación de directivas (al eliminar y crear una nueva asignación)
- Adición del grupo de recursos a la lista de exclusión para la asignación de directivas

Cualquier cambio en el ámbito (nivel o exclusiones) debe validarse y comunicarse completamente con las organizaciones de seguridad y cumplimiento para garantizar que no haya lagunas en la cobertura.

## <a name="monitor-your-policy-and-compliance"></a>Supervisión de la directiva y cumplimiento

La implementación y asignación de la definición de directiva no es el paso final. Supervise continuamente el nivel de [cumplimiento](../how-to/get-compliance-data.md) de los recursos con la nueva definición de directiva y configure las [alertas y notificaciones de Azure Monitor](../../../azure-monitor/platform/alerts-overview.md) adecuadas para la configuración de los dispositivos no compatibles. También se recomienda evaluar la definición de directiva y las asignaciones relacionadas de forma programada para validar que la definición de directiva cumple las necesidades de cumplimiento y directivas empresariales. Las directivas deben quitarse si ya no se necesitan. Las directivas también necesitan actualizarse de vez en cuando a medida que los recursos de Azure subyacentes evolucionan y agregan nuevas propiedades y funcionalidades.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [estructura de la definición de directiva](./definition-structure.md).
- Más información sobre la [estructura de asignaciones de directivas](./assignment-structure.md)
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](../how-to/programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](../how-to/get-compliance-data.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.