---
title: Supervisión continua con Azure Monitor | Microsoft Docs
description: Describe pasos específicos para usar Azure Monitor y habilitar la supervisión continua en todos sus flujos de trabajo.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/12/2018
ms.openlocfilehash: 0173d324da4f6cf2a24b825fe44fc07b7fe50b84
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318137"
---
# <a name="continuous-monitoring-with-azure-monitor"></a>Supervisión continua con Azure Monitor

La supervisión continua hace referencia al proceso y la tecnología necesarios para incorporar la supervisión en cada fase los ciclos de vida de las operaciones de TI y DevOps. Contribuye a garantizar de forma continua el mantenimiento, el rendimiento y la fiabilidad de su aplicación e infraestructura, ya que abarca desde el desarrollo hasta la producción. La supervisión continua se basa en los conceptos de integración continua e implementación continua (CI/CD) que le ayudan a desarrollar y ofrecer software más rápido y de forma más fiable para proporcionar valor continuo a sus usuarios.

[Azure Monitor](overview.md) es la solución de supervisión unificada de Azure que proporciona observabilidad de pila completa entre las aplicaciones y la infraestructura en la nube y el entorno local. Funciona perfectamente con [Visual Studio y Visual Studio Code](https://visualstudio.microsoft.com/) durante el proceso de desarrollo y pruebas, y se integra con [Azure DevOps](/azure/devops/user-guide/index) para la administración de versiones y la administración de elementos de trabajo durante la implementación y las operaciones. Incluso se integra en las herramientas de ITSM y SIEM de su elección para ayudar a realizar un seguimiento de problemas e incidentes en sus procesos de TI existentes.

En este artículo se describen pasos específicos para usar Azure Monitor y habilitar la supervisión continua en todos sus flujos de trabajo. Incluye vínculos a otra documentación que proporciona detalles sobre la implementación de distintas características.


## <a name="enable-monitoring-for-all-your-applications"></a>Habilitación de la supervisión de todas sus aplicaciones
Para obtener observabilidad en todo su entorno, debe habilitar la supervisión en todos sus servicios y aplicaciones web. Esto le permitirá visualizar con facilidad transacción y conexiones de un extremo a otro en todos los componentes.

- [Azure DevOps Projects](../devops-project/overview.md) ofrece una experiencia simplificada con su código existente y el repositorio de Git, o elegir una de las aplicaciones de ejemplo para crear una canalización de integración continua (CI) y entrega continua (CD) en Azure.
- [La supervisión continua en su canalización de versión de DevOps](./app/continuous-monitoring.md) le permite programar o revertir su implementación en función de los datos de supervisión.
- [Monitor de estado](./app/monitor-performance-live-website-now.md) le permite instrumentar una aplicación de .NET activa en Windows con Azure Application Insights sin tener que modificar ni volver a implementar el código.
- Si tiene acceso al código de su aplicación, habilite la supervisión completa con [Application Insights](./app/app-insights-overview.md) instalando el SDK de Application Insights de Azure Monitor para [.NET](./learn/quick-monitor-portal.md), [Java](./app/java-get-started.md), [Node.js](./learn/nodejs-quick-start.md) o [cualquier otro lenguaje de programación](./app/platforms.md). Esto le permite especificar eventos, métricas o vistas de página personalizados que son pertinentes para su aplicación y su empresa.



## <a name="enable-monitoring-for-your-entire-infrastructure"></a>Habilitación de la supervisión para toda su infraestructura
Las aplicaciones solo son tan fiables como su infraestructura subyacente. Tener habilitada la supervisión en toda su infraestructura le ayudará a conseguir una observabilidad completa y facilitar la detección de una posible causa raíz cuando se produce un error. Azure Monitor le ayuda a realizar un seguimiento del mantenimiento y el rendimiento de toda su infraestructura híbrida, incluidos recursos como máquinas virtuales, contenedores, almacenamiento y red.

- Automáticamente obtendrá [métricas de plataforma, registros de actividad y registros de diagnósticos](platform/data-sources.md) desde la mayoría de sus recursos de Azure sin ninguna configuración.
- Habilite una supervisión más profunda de las máquinas virtuales con [Azure Monitor para máquinas virtuales](insights/vminsights-overview.md).
-  Habilite una supervisión más profunda de los clústeres de AKS con [Azure Monitor para contenedores](insights/container-insights-overview.md).
- Agregue [soluciones de supervisión](./monitor-reference.md) para diferentes aplicaciones y servicios en su entorno.


La [infraestructura como código](/azure/devops/learn/what-is-infrastructure-as-code) es la administración de infraestructura en un modelo descriptivo, empleando las mismas versiones usadas por los equipos de DevOps para el código fuente. Agrega fiabilidad y escalabilidad a su entorno y le permite aprovechar procesos similares que solían administrar sus aplicaciones.

-  Use [plantillas de Resource Manager](platform/template-workspace-configuration.md) para habilitar la supervisión y configurar alertas a través de un gran conjunto de recursos.
- Use [Azure Policy](../governance/policy/overview.md) para hacer cumplir distintas reglas a través de sus recursos. Esto garantiza que esos recursos se mantengan compatibles con los estándares corporativos y los acuerdos de nivel de servicio. 


##  <a name="combine-resources-in-azure-resource-groups"></a>Combinación de recursos en los grupos de recursos de Azure
Una aplicación típica en Azure hoy incluye varios recursos como máquinas virtuales y App Services o microservicios hospedados en Cloud Services, clústeres de AKS o Service Fabric. Estas aplicaciones suelen usar dependencias como Event Hubs, Storage, SQL y Service Bus.

- Combine recursos en los grupos de recursos de Azure para obtener una visibilidad completa en todos sus recursos que constituyen sus diversas aplicaciones. [Azure Monitor para grupos de recursos](./insights/resource-group-insights.md) proporciona una forma sencilla de hacer un seguimiento del mantenimiento y el rendimiento de toda su aplicación de pila completa y permite profundizar en los respectivos componentes para la realización de cualquier investigación o depuración.

## <a name="ensure-quality-through-continuous-deployment"></a>Garantía de calidad a través de la implementación continua
Tanto la integración continua como la implementación continua le permite integrar e implementar cambios de código automáticamente en su aplicación en función de los resultados de las pruebas automatizadas. Optimiza el proceso de implementación y garantiza la calidad de cualquier cambio antes de pasar a la fase de producción.


- Use [Azure Pipelines](/azure/devops/pipelines) para implementar la implementación continua y automatizar todo su proceso desde la confirmación del código hasta la producción en función de sus pruebas de CI/CD.
- Use [puertas de calidad](/azure/devops/pipelines/release/approvals/gates) para integrar la supervisión en su implementación anterior o posterior. Esto garantiza su cumplimiento de las métricas de mantenimiento o rendimiento clave (KPI), ya que sus aplicaciones abarcan desde el desarrollo hasta la producción, y ninguna diferencia en el entorno de infraestructura o escala afecta negativamente a sus KPI.
- [Mantenga instancias de supervisión independientes](./app/separate-resources.md) entre sus diferentes entornos de implementación, por ejemplo desarrollo, prueba, valor controlado y producto. Esto garantiza que los datos recopilados sean pertinentes en las aplicaciones e infraestructura asociadas. Si debe poner datos en correlación entre entornos, puede usar [gráficos de varios recursos en el Explorador de métricas](./platform/metrics-charts.md) o crear [consultas entre recursos en Azure Monitor](log-query/cross-workspace-query.md).


## <a name="create-actionable-alerts-with-actions"></a>Creación de alertas accionables con acciones
Un aspecto fundamental de supervisión informa de forma proactiva a los administradores sobre los problemas actuales y previstos. 

- Creación de [alertas en Azure Monitor](./platform/alerts-overview.md) en función de los registros y las métricas para identificar los estados de error previstos. Debe tener el objetivo de hacer que todas las alertas sean accionables, es decir, que representen condiciones críticas reales y quieran reducir los falsos positivos. Use [umbrales dinámicos](platform/alerts-dynamic-thresholds.md) para calcular automáticamente líneas de base en datos métricos en lugar de definir sus propios umbrales estáticos. 
- Defina las acciones para que las alertas usen los medios más eficaces para informar a sus administradores. Las [acciones para la notificación](platform/action-groups.md#create-an-action-group-by-using-the-azure-portal) disponibles son SMS, correos electrónicos, notificaciones push o llamadas de voz.
- Use acciones más avanzadas para [conectarse a su herramienta de ITSM](platform/itsmc-overview.md) u otros sistemas de administración de alertas a través de [webhooks](platform/activity-log-alerts-webhook.md).
- Corrija también situaciones identificadas en alertas con [runbooks de Azure Automation](../automation/automation-webhooks.md) o [Logic Apps](/connectors/custom-connectors/create-webhook-trigger) que se pueden iniciar a partir de una alerta con webhooks. 
- Use el [escalado automático](./learn/tutorial-autoscale-performance-schedule.md) para aumentar y reducir de forma dinámica sus recursos de proceso en función de las métricas recopiladas.

## <a name="prepare-dashboards-and-workbooks"></a>Preparación de paneles y libros
Garantizar que su desarrollo y operaciones tengan acceso a la misma telemetría y herramientas les permite ver patrones en todo su entorno y minimizar sus instancias de Mean Time To Detect (MTTD) y Mean Time to Restore (MTTR).

- Prepare [paneles personalizados](./learn/tutorial-app-dashboards.md) en función de métricas y registros comunes para los diversos roles de su organización. Los paneles pueden combinar datos a partir de todos los recursos de Azure.
- Prepare [libros](./platform/workbooks-overview.md) para garantizar el uso compartido del conocimiento entre el desarrollo y las operaciones. Estos podrían prepararse como informes dinámicos con gráficos de métricas y consultas de registro, o incluso como guías para la solución de problemas preparadas por desarrolladores que ayudan a las operaciones o la asistencia al cliente a afrontar los problemas básicos.

## <a name="continuously-optimize"></a>Optimización continua
 La supervisión es uno de los aspectos fundamentales de la popular filosofía Build-Measure-Learn (crear, medir y aprender), que recomienda realizar un seguimiento continuo de sus KPI y las métricas de comportamiento del usuario y, a continuación, esforzarse por optimizarlos a través de iteraciones de planeación. Azure Monitor le ayuda a recopilar métricas y registros pertinentes para su empresa y agregar nuevos puntos de datos en la próxima implementación tal como se requiere.

- Use herramientas en Application Insights para [realizar un seguimiento de la involucración y el comportamiento del usuario final](./learn/tutorial-users.md).
- Use el [análisis de impacto](./app/usage-impact.md) para que le ayude a priorizar las áreas en las que debe centrarse para alcanzar los KPI importantes.


## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre los componentes de diferencia de [Azure Monitor](overview.md).
- [Agregue la supervisión continua](./app/continuous-monitoring.md) a su canalización de versión.

