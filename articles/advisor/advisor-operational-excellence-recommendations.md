---
title: Mejora de la excelencia operativa con Advisor
description: Use Azure Advisor para optimizar y madurar la excelencia operativa de sus suscripciones de Azure.
ms.topic: article
ms.date: 10/24/2019
ms.openlocfilehash: 036adb7e7d59bd78980c72b210ad41faea277d00
ms.sourcegitcommit: ef055468d1cb0de4433e1403d6617fede7f5d00e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/16/2020
ms.locfileid: "88258476"
---
# <a name="achieve-operational-excellence-by-using-azure-advisor"></a>Obtención de la excelencia operativa con Azure Advisor

Las recomendaciones de excelencia operativa en Azure Advisor pueden ayudarle con: 
- Eficiencia de procesos y flujos de trabajo.
- Capacidad de administración de recursos.
- Procedimientos recomendados de implementación 

Puede consultar estas recomendaciones en la pestaña de **Excelencia operativa** del panel de Advisor.

## <a name="create-azure-service-health-alerts-to-be-notified-when-azure-problems-affect-you"></a>Creación de alertas de Azure Service Health para recibir notificaciones cuando se produzcan problemas de Azure que le afecten

Se recomienda que configure Azure Service Health alertas para que se le notifique cuando los problemas del servicio de Azure le afecten. [Azure Service Health](https://azure.microsoft.com/features/service-health/) es un servicio gratuito que ofrece una guía personalizada y soporte técnico cuando experimenta un problema con algún servicio de Azure. Advisor identifica las suscripciones que no tienen alertas configuradas y recomienda su configuración.


## <a name="design-your-storage-accounts-to-prevent-reaching-the-maximum-subscription-limit"></a>Diseño de las cuentas de almacenamiento para evitar alcanzar el límite máximo de la suscripción

Una región de Azure puede admitir un máximo de 250 cuentas de almacenamiento por suscripción. Una vez alcanzado el límite, no podrá crear cuentas de almacenamiento en esa combinación de región y suscripción. Advisor comprueba las suscripciones y ofrece recomendaciones para que pueda diseñar menos cuentas de almacenamiento para cualquier región/suscripción que esté a punto de alcanzar el límite.

## <a name="ensure-you-have-access-to-azure-cloud-experts-when-you-need-it"></a>Asegúrese de tener acceso a los expertos en la nube de Azure cuando lo necesite

Cuando se ejecuta una carga de trabajo crítica para la empresa, es importante tener acceso al soporte técnico cuando sea necesario. Advisor identifica las posibles suscripciones críticas para la empresa que no tienen soporte técnico incluido en su plan de soporte técnico. Recomienda actualizar a una opción que incluye soporte técnico.

## <a name="delete-and-re-create-your-pool-to-remove-a-deprecated-internal-component"></a>Eliminación del grupo para quitar un componente interno en desuso y su nueva creación

Si el grupo usa un componente interno en desuso, elimine y vuelva a crear el grupo para mejorar la estabilidad y el rendimiento.

## <a name="repair-invalid-log-alert-rules"></a>Reparación de las reglas de alertas de registro no válidas

Azure Advisor detecta las reglas de alertas que tienen consultas no válidas especificadas en la sección de condiciones. Puede crear reglas de alertas de registro en Azure Monitor y usarlas para ejecutar consultas de análisis en intervalos especificados. Los resultados de la consulta determinan si se debe desencadenar una alerta. Las consultas de análisis pueden convertirse en no válidas con el paso del tiempo debido a cambios en los comandos, las tablas o los recursos a los que se hace referencia. Advisor le recomienda que corrija la consulta en la regla de alertas para evitar que se deshabilite automáticamente y garantizar la cobertura de la supervisión de los recursos de Azure. [Más información sobre cómo solucionar problemas de las reglas de alertas](https://aka.ms/aa_logalerts_queryrepair).

## <a name="use-azure-policy-recommendations"></a>Uso de recomendaciones de Azure Policy

Azure Policy es un servicio de Azure que se usa para crear, asignar y administrar directivas. Estas directivas aplican reglas y efectos en los recursos. A continuación se muestran las recomendaciones de Azure Policy para ayudarle a lograr la excelencia operativa: 

**Administre etiquetas.** Esta directiva agrega o reemplaza la etiqueta y el valor especificados cuando se crea o actualiza cualquier recurso. Los recursos existentes se pueden corregir con una tarea de corrección. Esta directiva no modifica las etiquetas de los grupos de recursos.

**Aplique los requisitos de compatibilidad geográfica.** Esta directiva permite restringir las ubicaciones que la organización puede especificar al implementar los recursos. 

**Especifique las SKU de máquina virtual permitidas para las implementaciones.** Esta directiva permite especificar un conjunto de SKU de máquina virtual que su organización puede implementar.

**Aplique *Auditar las máquinas virtuales que no utilizan discos administrados*.**

**Habilite *Heredar una etiqueta del grupo de recursos*.** Esta directiva agrega o reemplaza la etiqueta y el valor del grupo de recursos primario especificados cuando se crea o actualiza cualquier recurso. Los recursos existentes se pueden corregir con una tarea de corrección.

## <a name="no-validation-environment-enabled"></a>No se habilitó ningún entorno de validación.
Azure Advisor ha determinado que no tiene un entorno de validación habilitado en la suscripción actual. Al crear los grupos de hosts, ha seleccionado \"No\" para \"el entorno de validación\"en la pestaña Propiedades. Tener al menos un grupo de hosts con un entorno de validación habilitado garantiza la continuidad del negocio mediante las implementaciones del servicio Windows Virtual Desktop con la detección temprana de posibles problemas. [Más información](https://docs.microsoft.com/azure/virtual-desktop/create-validation-host-pool)

## <a name="ensure-production-non-validation-environment-to-benefit-from-stable-functionality"></a>Garantice el entorno de producción (sin validación) para beneficiarse de la funcionalidad estable.
Azure Advisor ha determinado que muchos de los grupos de hosts tienen habilitado el entorno de validación. Para que los entornos de validación sirvan mejor a su propósito, debe tener al menos uno, pero nunca más de la mitad de los grupos de hosts en el entorno de validación. Al tener un equilibrio correcto entre los grupos de hosts con el entorno de validación habilitado y los que están deshabilitados, podrá usar mejor las ventajas de las implementaciones a varios niveles que ofrece Windows Virtual Desktop con ciertas actualizaciones. Para corregir este problema, abra las propiedades del grupo de hosts y seleccione\"No\" junto a la opción \"Entorno de validación\".

## <a name="enable-traffic-analytics-to-view-insights-into-traffic-patterns-across-azure-resources"></a>Habilitación de Análisis de tráfico para ver información detallada sobre los patrones de tráfico en los recursos de Azure
Análisis de tráfico es una solución basada en la nube, que proporciona visibilidad de la actividad de usuarios y aplicaciones en Azure. Análisis de tráfico analiza los registros de flujo del grupo de seguridad de red (NSG) de Network Watcher para proporcionar conclusiones sobre el flujo de tráfico. Con el análisis de tráfico, puede ver los principales llamadores en Azure e implementaciones que no son de Azure, investigar puertos abiertos, protocolos y flujos malintencionados en su entorno y optimizar la implementación de red para mejorar el rendimiento. Puede procesar registros de flujo a intervalos de procesamiento de 10 minutos y 60 minutos, lo que le permite realizar un análisis más rápido del tráfico. Se recomienda habilitar Análisis de tráfico para los recursos de Azure. 


## <a name="next-steps"></a>Pasos siguientes

Para aprender más sobre las recomendaciones de Advisor, consulte:
* [Introducción a Advisor](advisor-overview.md)
* [Introducción](advisor-get-started.md)
* [Recomendaciones sobre costo de Advisor](advisor-cost-recommendations.md)
* [Recomendaciones sobre rendimiento de Advisor](advisor-performance-recommendations.md)
* [Recomendaciones de confiabilidad de Advisor](advisor-high-availability-recommendations.md)
* [Recomendaciones sobre seguridad de Advisor](advisor-security-recommendations.md)
* [API REST de Advisor](/rest/api/advisor/)
