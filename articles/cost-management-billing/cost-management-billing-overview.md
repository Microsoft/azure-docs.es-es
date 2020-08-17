---
title: Información general sobre Azure Cost Management y facturación
description: Use las características de Azure Cost Management y facturación para realizar tareas administrativas de facturación y administrar el acceso de facturación a los costos. También puede usar las características para supervisar y controlar el gasto de Azure y para optimizar el uso de los recursos de Azure.
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 07/29/2020
ms.topic: overview
ms.service: cost-management-billing
ms.custom: ''
ms.openlocfilehash: 76121dd8126922de2843e430a4e01cae1c7c0d53
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87413007"
---
# <a name="what-is-azure-cost-management--billing"></a>¿Qué es Azure Cost Management y facturación?

Mediante la nube de Microsoft, puede mejorar significativamente el rendimiento técnico de las cargas de trabajo de su empresa. También puede reducir los costos y la sobrecarga necesarios para administrar los recursos de la organización. No obstante, la oportunidad de negocio crea un riesgo debido a la posibilidad de pérdidas e ineficiencias que genera en las implementaciones en la nube. Azure Cost Management y facturación es un conjunto de herramientas que proporciona Microsoft para ayudarle a analizar, administrar y optimizar los costos de sus cargas de trabajo. El uso del conjunto de herramientas ayuda a garantizar que su organización está aprovechando las ventajas que proporciona la nube.

Puede pensar en las cargas de trabajo de Azure como en las luces de su hogar. Cuando sale para pasar fuera todo el día, ¿deja las luces encendidas? ¿Puede usar diferentes bombillas más eficaces para reducir la factura de energía mensual? ¿Alguna habitación tiene más luces de las necesarias? Puede usar Azure Cost Management y facturación para aplicar un proceso de pensamiento similar a las cargas de trabajo que usa su organización.

Con los productos y servicios de Azure, solo paga por lo que usa. A medida que cree y use recursos de Azure, se le cobrará por ellos. Debido a lo fácil que es implementar nuevos recursos, los costos de las cargas de trabajo pueden variar de forma significativa si no se realiza un análisis y supervisión adecuados. Use las características de Azure Cost Management y facturación para:

- Realizar tareas administrativas de facturación, como pagar su factura.
- Administrar el acceso de facturación a los costos.
- Descargar los datos de costo y uso que se usaron para generar la factura mensual.
- Aplicar análisis de datos a sus costos de forma proactiva.
- Establecer umbrales de gasto.
- Identificar oportunidades para cambiar las cargas de trabajo que pueden optimizar los gastos.

Para obtener más información sobre cómo abordar la administración de costos como organización, eche un vistazo al artículo [Azure Cost Management best practices](./costs/cost-mgt-best-practices.md) (Procedimientos recomendados de Azure Cost Management).

## <a name="understand-azure-billing"></a>Descripción de la facturación de Azure

Las características de facturación de Azure se usan para revisar los costos facturados y administrar el acceso a la información de facturación. En las grandes organizaciones, los equipos de adquisiciones y finanzas son los que normalmente realizan las tareas de facturación.

Cuando se registra para usar Azure, se crea una cuenta de facturación. Use su cuenta de facturación para administrar las facturas, los pagos y hacer seguimiento de los costos. Puede tener acceso a varias cuentas de facturación. Por ejemplo, es posible que te hayas registrado en Azure para tus proyectos personales. Por lo tanto, es posible que tenga una suscripción individual de Azure con una cuenta de facturación. También se puede tener acceso a Azure a través del Contrato Enterprise de la organización o del contrato de cliente de Microsoft. Para cada escenario, dispondría de una cuenta de facturación independiente.

### <a name="billing-accounts"></a>Cuentas de facturación

Azure Portal admite actualmente los siguientes tipos de cuentas de facturación:

- **Microsoft Online Services Program**: se crea una cuenta de facturación individual para el programa de Microsoft Online Services cuando se registra en el sitio web de Azure. Por ejemplo, cuando se registra para obtener una [cuenta gratuita de Azure](https://azure.microsoft.com/offers/ms-azr-0044p/), como una [cuenta con tarifas de pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/) o como un [suscriptor de Visual Studio](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/).

- **Contrato Enterprise**: Se crea una cuenta de facturación para un Contrato Enterprise cuando la organización firma este [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/) para utilizar Azure.

- **Contrato de cliente de Microsoft**: Se crea una cuenta de facturación para un contrato de cliente de Microsoft cuando una organización trabaja con un representante de Microsoft para firmar un contrato de cliente de Microsoft. Algunos clientes en regiones seleccionadas, que se registran en el sitio web de Azure para una [cuenta con tarifas de pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/) o actualizar su [cuenta gratuita de Azure](https://azure.microsoft.com/offers/ms-azr-0044p/), también pueden tener una cuenta de facturación para un contrato de cliente de Microsoft. Para obtener más información, consulte [Empezar a trabajar con la cuenta de facturación para un contrato de cliente de Microsoft](./understand/mca-overview.md).

### <a name="scopes-for-billing-accounts"></a>Ámbitos para las cuentas de facturación
Un ámbito es un nodo dentro de una cuenta de facturación que los usuarios utilizan para ver y administrar la facturación. Es donde se administran los datos de facturación, los pagos, las facturas y se realiza la administración de cuentas generales.

#### <a name="microsoft-online-services-program"></a>Microsoft Online Services Program

|Ámbito  |Definición  |
|---------|---------|
|Cuenta de facturación     | Representa a un propietario único (administrador de cuenta) para una o varias suscripciones de Azure. Un administrador de cuenta está autorizado para llevar a cabo varias tareas de facturación, como crear suscripciones, ver facturas o cambiar la facturación de las suscripciones.  |
|Suscripción     |  Representa una agrupación de los recursos de Azure. Se genera una factura en el ámbito de la suscripción. Tiene sus propios métodos de pago que se usan para pagar esta factura.|

#### <a name="enterprise-agreement"></a>Contrato Enterprise

|Ámbito  |Definición  |
|---------|---------|
|Cuenta de facturación    | Representa una inscripción de Contrato Enterprise. La factura se genera en el ámbito de la cuenta de facturación. Se estructura mediante el uso de cuentas de inscripción y departamentos.  |
|department     |  Agrupación opcional de las cuentas de inscripción.      |
|Cuenta de inscripción     |  Representa al propietario de una cuenta única. Las suscripciones de Azure se crean en el ámbito de las cuentas de inscripción.  |

#### <a name="microsoft-customer-agreement"></a>Contrato de cliente de Microsoft

|Ámbito  |Tareas  |
|---------|---------|
|Cuenta de facturación     |   Representa un contrato de cliente para varios productos y servicios de Microsoft. La cuenta de facturación se estructura mediante el uso de perfiles de facturación y las secciones de la factura.   |
|Perfil de facturación     |  Representa una factura y sus métodos de pago. La factura se genera en este ámbito. El perfil de facturación puede tener varias secciones de factura.      |
|Sección de factura     |   Representa un grupo de costos en una factura. Las suscripciones y otras compras están asociadas al ámbito de las secciones de facturas.    |

## <a name="understand-azure-cost-management"></a>Descripción de Azure Cost Management

Aunque están relacionados, la facturación difiere de administración de costos. La facturación es el proceso de facturar a los clientes por bienes o servicios y administrar la relación comercial.

Cost Management muestras los costos de organización y los patrones de uso con análisis avanzados. Los informes de Cost Management muestra los costos basados en el uso consumidos por los servicios de Azure y las ofertas de Marketplace de terceros. Los costos dependen de los precios negociados y del factor de reserva y los descuentos de Ventaja híbrida de Azure. En conjunto, en los informes se muestran los costos internos y externos del uso y los cargos de Azure Marketplace. En los informes todavía no se muestran otros cargos, como las compras de reservas, el soporte técnico y los impuestos. Los informes le ayudan a comprender los gastos y el uso de recursos, y pueden ayudar a encontrar anomalías de gastos. Los análisis predictivos también están disponibles. Cost Management usa grupos de administración de Azure, presupuestos y recomendaciones para mostrar claramente cómo se organizan los gastos y cómo se pueden reducir los costos.

Puede usar Azure Portal o varias API para la automatización de la exportación, de manera que se integren los datos de costos con los procesos y sistemas externos. También están disponibles la exportación de datos de facturación automatizada y los informes programados.

Vea el [vídeo de información general de Azure Cost Management](https://www.youtube.com/watch?v=el4yN5cHsJ0) para consultar una breve introducción sobre cómo Azure Cost Management puede ayudarle a ahorrar dinero en Azure. Para ver otros vídeos, visite el [canal de YouTube de Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/el4yN5cHsJ0]

### <a name="plan-and-control-expenses"></a>Planificación y control de los gastos

A continuación se mencionan las formas mediante las que Cost Management le ayuda a planear y controlar los costos: análisis de costos, presupuestos, recomendaciones y exportación de datos de administración de costos.

Los análisis de costos se usan para explorar y analizar los costos de su organización. Puede ver los costos agregados por organización para saber dónde se acumulan estos e identificar las tendencias de gasto. Además, puede ver los costos acumulados con el tiempo para estimar las tendencias de costos mensual, trimestral o incluso anualmente con respecto a un presupuesto.

Los presupuestos le ayudarán a planear y cumplir con las responsabilidades financieras de la organización. Ayudan a evitar que se superen los umbrales o los límites de costos. Los presupuestos también pueden ayudar a informar a otros usuarios sobre sus gastos para que administren de manera proactiva los costos. Y con ellos, puede ver cómo los gastos crecen con el tiempo.

Las recomendaciones le muestran cómo optimizar y mejorar la eficiencia mediante la identificación de recursos inactivos e infrautilizados. O bien, pueden mostrar las opciones de recurso menos costosas. Al implementar las recomendaciones, puede cambiar la forma en que usa sus recursos para ahorrar dinero. Para hacerlo, primero debe ver las recomendaciones de optimización de costos, para así ver posibles deficiencias de uso. A continuación, puede implementar una recomendación al cambiar su uso del recurso de Azure por una opción más rentable. Después, puede verificar la acción para asegurarse de que el cambio se realizó correctamente.

Si usa los sistemas externos para acceder o revisar los datos de administración de costos, puede exportar los datos fácilmente desde Azure. Y puede establecer una exportación programada diariamente en formato CSV y almacenar los archivos de datos en el almacenamiento de Azure. A continuación, puede tener acceso a los datos desde el sistema externo.

### <a name="cloudyn-deprecation"></a>Desuso de Cloudyn

[Cloudyn](./cloudyn/overview.md) es un servicio de Azure relacionado con Cost Management que pasará a estar en desuso a finales de 2020. Las características de Cloudyn existentes se están integrando directamente en Azure Portal siempre que es posible. En este momento no se incorporan nuevos clientes, pero se mantendrá el soporte técnico para el producto hasta que esté completamente en desuso.
 
Vea el [vídeo de Azure Cost Management y Cloudyn](https://www.youtube.com/watch?v=15DzKPMBRxM) para saber cuándo debe usar Azure Cost Management o Cloudyn, según sus necesidades empresariales. Para ver otros vídeos, visite el [canal de YouTube de Cost Management](https://www.youtube.com/c/AzureCostManagement).
 
>[!VIDEO https://www.youtube.com/embed/15DzKPMBRxM]

### <a name="additional-azure-tools"></a>Herramientas de Azure adicionales

Azure tiene otras herramientas que no forman parte del conjunto de características de Azure Cost Management y facturación. Sin embargo, tienen un rol importante en el proceso de administración de costos. Para aprender más sobre dichas herramientas, consulte los siguientes vínculos.

- [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/): utilice esta herramienta para calcular los en la nube costos por adelantado.
- [Azure Migrate](../migrate/migrate-overview.md): evalué la carga de trabajo actual de su centro de datos para obtener información sobre lo que se necesita de la solución de reemplazo de Azure.
- [Azure Advisor](../advisor/advisor-overview.md): identifique las máquinas virtuales no utilizadas y reciba recomendaciones sobre la compra de instancias reservadas de Azure.
- [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/): use sus licencias locales de Windows Server o SQL Server para máquinas virtuales en Azure para el guardado.

## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con Cost Management y la facturación, el siguiente paso es que empiece a usar el servicio.

- Empiece por usar Azure Cost Management para [analizar los costos](./costs/quick-acm-cost-analysis.md).
- También puede leer más sobre [Azure Cost Management best practices](./costs/cost-mgt-best-practices.md) (Procedimientos recomendados de Azure Cost Management).
