---
title: Optimización de la inversión en la nube con Azure Cost Management
description: Este artículo le ayuda a obtener el máximo partido de las inversiones en la nube, reducir los costos y evaluar dónde se invierte su dinero.
author: bandersmsft
ms.author: banders
ms.date: 05/27/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.custom: seodec18
ms.openlocfilehash: 729150188e918d4964f8abe729c66032d8e6c9c0
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689157"
---
# <a name="how-to-optimize-your-cloud-investment-with-azure-cost-management"></a>Optimización de la inversión en la nube con Azure Cost Management

Azure Cost Management le proporciona herramientas para planear, analizar y reducir los gastos para maximizar la inversión en la nube. Este documento le proporciona un enfoque metódico para la administración de costos y resalta las herramientas disponibles para abordar los desafíos de la organización en materia de costos. Azure facilita la creación e implementación de soluciones en la nube. Sin embargo, es importante que esas soluciones estén optimizadas para minimizar el costo para la organización. Seguir los principios descritos en este documento y el uso de las herramientas le ayudarán a asegurarse de que la organización está preparada para el éxito.

## <a name="methodology"></a>Metodología

La administración del costo es un problema de la organización y debe ser una práctica en curso que se inicia antes de invertir dinero en recursos en la nube. Para implementar la administración de costos y optimizar los costos correctamente, la organización debe:

- Estar preparada con las herramientas adecuadas para el éxito
- Hacerse responsable de los costos
- Tomar las medidas adecuadas para optimizar el gasto

Los tres grupos principales descritos a continuación deben estar alineados en la organización para asegurarse de que se administran correctamente los costos.

- **Finanzas**: personas responsables de aprobar las solicitudes de presupuestos de la organización en función de las previsiones de gastos en la nube. Pagan las facturas correspondientes y asignan los costos a los distintos equipos para su contabilización.
- **Administradores**: toman las decisiones de negocio de una organización y necesitan comprender los gastos en la nube para encontrar los mejores resultados de gasto.
- **Equipos de aplicaciones**: ingenieros que administración los recursos en la nube de forma cotidiana y desarrollan servicios para satisfacer las necesidades de la organización. Estos equipos necesitan flexibilidad para sacar el máximo partido de los presupuestos que se han definido.

### <a name="key-principles"></a>Principios fundamentales

Use los principios descritos a continuación para posicionar a la organización para el éxito en la administración de los costos de la nube.

Para más información, vea el vídeo [Configuración de Cost Management para el éxito](https://www.youtube.com/watch?v=dVuwITdSAZ4). Para ver otros vídeos, visite el [canal de YouTube de Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/dVuwITdSAZ4]

#### <a name="planning"></a>Planificación

Un planeamiento integral por adelantado permite adaptar el uso de la nube a los requisitos empresariales específicos. Pregúntese lo siguiente:

- ¿Qué problema empresarial estoy solucionando?
- ¿Qué patrones de uso se esperan de los recursos?

Las respuestas le ayudarán a seleccionar las ofertas adecuadas para usted. Determinan la infraestructura que se va a utilizar y cómo se usa para maximizar la eficacia de Azure.

#### <a name="visibility"></a>Visibilidad

Cuando está bien estructurado, Cost Management le ayuda a informar a los usuarios sobre los costos de Azure de los que son responsables o del dinero que gastan. Azure tiene servicios diseñados para ofrecer una visión detallada acerca de *dónde* se gasta su dinero. Saque partido de estas herramientas. Pueden ayudarle a encontrar los recursos que están infrautilizados, eliminar los residuales y maximizar las oportunidades de ahorro de costos.

#### <a name="accountability"></a>Rendición de cuentas

Realice una atribución de los costos de la organización para asegurarse de que las personas adecuadas son responsables del gasto de su equipo. Para comprender el gasto de Azure en la organización, debe organizar los recursos para maximizar la visión sobre la atribución de costos. Una buena organización ayuda a administrar y reducir los costos y hace a las personas responsables de un gasto eficiente en la organización.

#### <a name="optimization"></a>Optimization

Actúe para reducir los gastos. Aproveche al máximo en función de las conclusiones recopiladas mediante el planeamiento y el aumento de la visibilidad de los costos. Puede considerar optimizaciones en las adquisiciones y las licencias junto con cambios en la implementación de la infraestructura que se describen en detalle más adelante en este documento.

#### <a name="iteration"></a>Iteración

Todas las personas de la organización deben participar en el ciclo de vida de la administración de costos. Deben permanecer implicados de forma continuada para optimizar los costos. Sea riguroso en este proceso iterativo y conviértalo en una pieza clave de la gobernanza responsable de la nube en la organización.

![Diagrama de principios clave que muestra la visibilidad, responsabilidad y optimización](./media/cost-mgt-best-practices/principles.png)

## <a name="plan-with-cost-in-mind"></a>Planear teniendo en cuenta los costos

Antes de implementar recursos en la nube, evalúe los siguientes elementos:

- La oferta de Azure que mejor se adapta a sus necesidades
- Los recursos que se van a usar
- Cuánto podrían costar

Azure proporciona herramientas para ayudarle en el proceso de evaluación. Las herramientas pueden proporcionarle una buena idea de la inversión necesaria para admitir las cargas de trabajo. A continuación, puede seleccionar la mejor configuración para su situación.

### <a name="azure-onboarding-options"></a>Opciones de incorporación de Azure

El primer paso para maximizar su experiencia con Cost Management es investigar y decidir qué oferta de Azure es la mejor para usted. Piense en cómo va a usar Azure en el futuro. Considere también cómo desea configurar el modelo de facturación. Cuando tome la decisión, tenga en cuenta las siguientes preguntas:

- ¿Durante cuánto tiempo tengo previsto usar Azure? ¿Estoy probando o voy a crear una infraestructura a largo plazo?
- ¿Cómo quiero pagar por Azure? ¿Debería pagar por adelantado para obtener un precio reducido u obtener una factura al final del mes?

Para más información sobre las distintas opciones, consulte las [Instrucciones para contratar Azure](https://azure.microsoft.com/pricing/purchase-options/). A continuación, se identifican algunos de los modelos de facturación más comunes.

#### <a name="free"></a>[Gratis](https://azure.microsoft.com/free/)

- 12 meses de servicios populares gratis
- 200 USD en crédito para explorar los servicios durante 30 días
- Más de 25 servicios siempre son gratis

#### <a name="pay-as-you-go"></a>[Pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p)

- Sin mínimos ni compromisos
- Precios competitivos
- Pague solo por lo que usa
- Cancelación en cualquier momento

#### <a name="enterprise-agreement"></a>[Contrato Enterprise](https://azure.microsoft.com/pricing/enterprise-agreement/)

- Opciones para compromisos monetarios por adelantado
- Acceso a la reducción de precios de Azure

#### <a name="azure-in-csp"></a>[Azure en CSP](https://azure.microsoft.com/offers/ms-azr-0145p/)

- Los asociados de CSP son el primer punto de contacto para las necesidades de sus clientes y el centro de la relación con los clientes
- Los asociados de CSP aprovisionan nuevos clientes, ordenan suscripciones, administran suscripciones y realizan tareas de administración en nombre de sus clientes
- Los asociados de CSP agrupan servicios con soluciones únicas o revenden Azure mientras controlan los precios, los términos y la facturación

## <a name="estimate-the-cost-of-your-solution"></a>Estimación del costo de la solución

Antes de implementar cualquier infraestructura, debe evaluar cuánto costará la solución. La evaluación le ayudará a crear un presupuesto para la organización de la carga de trabajo por adelantado. A continuación, puede usar un presupuesto a lo largo del tiempo para realizar pruebas comparativas de la validez de la estimación inicial. Y puede compararlo con el costo real de la solución implementada.

### <a name="azure-pricing-calculator"></a>Calculadora de precios de Azure

La calculadora de precios de Azure le permite trabajar con diferentes combinaciones de servicios de Azure para ver una estimación de los costos. Puede implementar la solución de diferentes maneras en Azure y cada una de ellas podría influir en el gasto general. Pensar lo antes posible sobre todas las necesidades de infraestructura de la implementación en la nube le ayudará a usar la herramienta de la forma más eficaz. Puede ayudarle a obtener una estimación sólida de los gastos estimados en Azure.

Para más información, consulte la [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator).

### <a name="azure-migrate"></a>Azure Migrate

Azure Migrate es un servicio que evalúa las cargas de trabajo actuales de la organización en los centros de datos locales. Proporciona información sobre lo que podría necesitar en una solución de reemplazo de Azure. En primer lugar, Migrate analiza las máquinas locales para determinar si la migración es factible. A continuación, recomienda el tamaño de máquina virtual en Azure para maximizar el rendimiento. Por último, también crea una estimación del costo de una solución basada en Azure.

Para más información, consulte [Azure Migrate](https://docs.microsoft.com/azure/migrate/migrate-services-overview).

## <a name="analyze-and-manage-your-costs"></a>Análisis y administración de los costos

Manténgase informado acerca de cómo evolucionan los costos de la organización con el tiempo. Use las técnicas siguientes para comprender y administrar los gastos correctamente.

### <a name="organize-resources-to-maximize-cost-insights-and-accountability"></a>Organización de los recursos para maximizar la información y responsabilidad de costos

Una estructura organizativa bien planeada para las jerarquías de recursos y de facturación de Azure ayuda a proporcionar una buena comprensión y control sobre los costos a medida que se crea la infraestructura de la nube. Vea el vídeo [Configuración de jerarquías de entidades](https://www.youtube.com/watch?v=n3TLRaYJ1NY) para comprender mejor las herramientas organizativas que están disponibles y cómo aprovecharlas. Para ver otros vídeos, visite el [canal de YouTube de Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/n3TLRaYJ1NY]

A medida que evalúe y cree una jerarquía que satisfaga sus necesidades, hágase las siguientes preguntas.

*¿Qué jerarquía de facturación tengo disponible y cuáles son los diferentes ámbitos que puedo usar?*

Identifique la disposición de facturación de la organización mediante la determinación del tipo de oferta de Azure. Los ámbitos disponibles para cada disposición de facturación de Azure se documentan en [Descripción y uso de ámbitos](understand-work-scopes.md).

*Si tengo varios equipos, ¿cómo debo organizar mis suscripciones y grupos de recursos?*

La creación de una suscripción o un grupo de recursos para cada equipo es una práctica común. Pueden ayudar a diferenciar los costos y hacer que los equipos rindan cuentas. Sin embargo, los costos se enlazan a la suscripción o al grupo de recursos.

Si ya tiene equipos con varias suscripciones, considere la posibilidad de agrupar las suscripciones en grupos de administración para analizar todos los costos juntos. Los grupos de administración, las suscripciones y los grupos de recursos forman parte de la jerarquía de RBAC de Azure. Úselos colectivamente para el control de acceso de sus equipos.

Los recursos pueden abarcar varios ámbitos, especialmente cuando se comparten entre varios equipos o cargas de trabajo. Considere la posibilidad de identificar los recursos con etiquetas. Las etiquetas se describen con más detalle en la sección siguiente.

*¿Tengo entornos de desarrollo y producción?*

Considere la posibilidad de crear suscripciones de desarrollo y pruebas para los entornos de desarrollo con el fin de aprovechar los precios reducidos. Si las cargas de trabajo abarcan varios equipos o ámbitos de Azure, puede usar etiquetas para identificarlas.

### <a name="tag-shared-resources"></a>Recursos compartidos de etiquetas

Las etiquetas son una manera eficaz de comprender los costos que abarcan varios equipos y ámbitos de Azure. Por ejemplo, podría tener un recurso como un servidor de correo electrónico que usan muchos equipos. Puede colocar un recurso compartido, como el servidor de correo electrónico, en una suscripción dedicada a recursos compartidos o colocarlo en una suscripción existente. Si lo coloca en una suscripción existente, es posible que el propietario de la suscripción no quiera que su costo se acumule en su equipo cada mes. En este ejemplo, puede usar una etiqueta para identificar el recurso como compartido.

Del mismo modo, también puede tener aplicaciones web o entornos, como pruebas o producción, que usan recursos en varias suscripciones que pertenecen a distintos equipos. Para comprender mejor el costo total de las cargas de trabajo, etiquete los recursos que usan. Cuando las etiquetas se aplican correctamente, puede aplicarlas como filtro en análisis de costos para comprender mejor las tendencias.

Después de planear el etiquetado de recursos, puede configurar una directiva de Azure para aplicar el etiquetado en los recursos. Vea el vídeo [Revisión de las directivas de etiquetas con Azure Cost Management](https://www.youtube.com/watch?v=nHQYcYGKuyw) para conocer las herramientas disponibles para aplicar el etiquetado de recursos escalable en su organización. Para ver otros vídeos, visite el [canal de YouTube de Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/nHQYcYGKuyw]

### <a name="use-cost-analysis"></a>Uso del análisis de costos

El análisis de costos permite analizar los costos de la organización en profundidad mediante la desagregación de los costos con las propiedades estándar de los recursos. Tenga en cuenta las siguientes preguntas comunes como guía para el análisis. Responder a estas preguntas de forma regular le ayudará a mantenerse más informado y permite decisiones más conscientes de los costos.

- **Costos estimados para el mes actual**: ¿en cuánto costo he incurrido este mes hasta el momento? ¿Me mantendré dentro de mi presupuesto?
- **Investigar las anomalías**: realice comprobaciones rutinarias para asegurarse de que los costos permanecen dentro de un intervalo razonable de uso normal. ¿Cuáles son las tendencias? ¿Existen valores atípicos?
- **Conciliación de facturas**: ¿mi último costo facturado es mayor que el mes anterior? ¿Cómo cambian los hábitos de gasto de mes a mes?
- **Contracargo interno**: ahora que sé cuánto se me está cobrando, ¿cómo deben dichos cargos dividirse en mi organización?

Para más información, consulte [Análisis de costos](quick-acm-cost-analysis.md).

### <a name="export-billing-data-on-a-schedule"></a>Exportación de los datos de facturación según una programación

¿Es necesario importar los datos de facturación en un sistema externo como un panel o un sistema financiero? Configure las exportación automatizadas en Azure Storage y evite descargar manualmente archivos de manera mensual. Luego, puede configurar fácilmente las integraciones automáticas con otros sistemas para mantener sincronizados los datos de facturación.

Para más información acerca de cómo exportar datos de facturación, consulte [Creación y administración de datos exportados](tutorial-export-acm-data.md).

### <a name="create-budgets"></a>Creación de presupuestos

Una vez que ha identificado y analizado los patrones de gasto, es importante empezar a establecer límites para usted y sus equipos. Los presupuestos de Azure le permiten establecer un costo o un presupuesto según el uso con diversos umbrales y alertas. Asegúrese de revisar los presupuestos que cree con regularidad para ver el progreso de la evolución del presupuesto y realizar los cambios necesarios. Los presupuestos de Azure también permiten configurar un desencadenador de automatización cuando se alcanza un umbral presupuestario determinado. Por ejemplo, puede configurar el servicio para que apague las máquinas virtuales. O bien, puede mover la infraestructura a un plan de tarifa diferente en respuesta a un desencadenador presupuestario.

Para más información, consulte [Presupuestos de Azure](tutorial-acm-create-budgets.md).

Para más información acerca de la automatización basada en presupuestos, consulte [Automatización basada en presupuestos](../manage/cost-management-budget-scenario.md).

## <a name="act-to-optimize"></a>Actuaciones para optimizar
Utilice los siguientes métodos para optimizar el gasto.

### <a name="cut-out-waste"></a>Recorte de los gastos residuales

Después de implementar la infraestructura en Azure, es importante asegurarse de que se está usando. La manera más fácil de empezar a ahorrar inmediatamente es revisar los recursos y eliminar los que no se usan. A partir de ahí, debe determinar si los recursos se emplean de la forma más eficaz posible.

#### <a name="azure-advisor"></a>Azure Advisor

Azure Advisor es un servicio que, entre otras cosas, identifica las máquinas virtuales con un uso reducido desde un punto de vista de uso de CPU o de red. Con esta información, puede decidir apagar o cambiar el tamaño de la máquina en función del costo estimado para continuar la ejecución de las máquinas. Advisor también proporciona recomendaciones para la compra de instancias reservadas. Las recomendaciones se basan en los últimos 30 días de uso de las máquinas virtuales. Cuando se actúa en consecuencia, las recomendaciones pueden ayudarle a reducir los gastos.

Para más información, consulte [Azure Advisor](../../advisor/advisor-overview.md).

### <a name="size-your-vms-properly"></a>Ajuste del tamaño adecuado de las máquinas virtuales

El ajuste del tamaño de la máquina virtual tiene un impacto significativo en el costo general de Azure. El número de máquinas virtuales que se necesitan en Azure podría no ser igual al que tenga implementado actualmente en un centro de datos local. Asegúrese de elegir el tamaño correcto para las cargas de trabajo que va a ejecutar.

Para más información, consulte [IaaS de Azure: ajuste del tamaño adecuado y costo](https://azure.microsoft.com/resources/videos/azurecon-2015-azure-iaas-proper-sizing-and-cost/).

### <a name="use-purchase-discounts"></a>Uso de descuentos de compra

Azure tiene muchos descuentos que su organización puede aprovechar para ahorrar dinero.

#### <a name="azure-reservations"></a>Reservas de Azure

Las reservas de Azure le permiten realizar el pago por adelantado por un año o tres años de máquina virtual o capacidad de proceso de SQL Database. El pago por adelantado le permite obtener un descuento en los recursos que utiliza. Las reservas de Azure pueden reducir significativamente los costos de las máquinas virtuales o de proceso de SQL Database (hasta un 72 % con respecto a los precios de pago por uso) con el compromiso por adelantado de uno o tres años. Las reservas ofrecen un descuento en la facturación y no afectan al estado del entorno de ejecución de las máquinas virtuales o bases de datos SQL.

Para más información, consulte [¿Qué son las reservas de Azure?](../reservations/save-compute-costs-reservations.md).

#### <a name="use-azure-hybrid-benefit"></a>Uso de la Ventaja híbrida de Azure

Si ya tiene licencias de Windows Server o de SQL Server en las implementaciones locales, puede usar el programa de Ventaja híbrida de Azure para ahorrar en Azure. Con la ventaja de Windows Server, cada licencia cubre el costo del sistema operativo (en dos máquinas virtuales como máximo) y solo tiene que pagar los costos de proceso básicos. Puede usar licencias de SQL Server existentes para ahorrar hasta un 55 por ciento en las opciones de SQL Database basadas en núcleos virtuales. Las opciones incluyen SQL Server en Azure Virtual Machines y SQL Server Integration Services.

Para más información, consulte [Calculadora de ahorro de la Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/).

### <a name="other-resources"></a>Otros recursos

Azure también tiene un servicio que le permite crear servicios que aprovechan la capacidad sobrante en Azure para disponer de tarifas reducidas. Para más información, consulte [Uso de máquinas virtuales de prioridad baja con Batch](../../batch/batch-low-pri-vms.md).

## <a name="next-steps"></a>Pasos siguientes
- Si no está familiarizado con Cost Management, puede consultar [¿Qué es Azure Cost Management?](../cost-management-billing-overview.md) para obtener información sobre cómo le ayuda a supervisar y controlar el gasto de Azure y para optimizar el uso de recursos.
