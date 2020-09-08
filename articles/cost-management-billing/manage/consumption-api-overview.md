---
title: Información general sobre las API de consumo de Azure
description: Obtenga información sobre cómo las API de consumo de Azure ofrecen acceso mediante programación a los datos de costos y uso de los recursos de Azure.
author: bandersmsft
tags: billing
ms.service: cost-management-billing
ms.subservice: cost-management
ms.topic: reference
ms.date: 02/12/2020
ms.author: banders
ms.openlocfilehash: e2252d13dd39587e7730b141a6096afc0a34dbc1
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690024"
---
# <a name="azure-consumption-api-overview"></a>Información general sobre las API de consumo de Azure

Las API de consumo de Azure ofrecen acceso mediante programación a los datos de costos y uso de los recursos de Azure. Estas API actualmente solo admiten inscripciones a Enterprise y suscripciones a Web Direct (con algunas excepciones). Las API se actualizan continuamente para admitir otros tipos de suscripciones de Azure.

Las API de consumo de Azure ofrecen acceso a:
- Clientes de Enterprise y Web Direct
    - Detalles de uso
    - Cargos de Marketplace
    - Recomendaciones de reserva
    - Detalles de la reserva
    - Resúmenes de reservas
- Solo clientes de Enterprise
    - Hoja de precios
    - Presupuestos
    - Saldos

## <a name="usage-details-api"></a>API de detalles de uso

Use la API de detalles de uso para obtener los datos de cargos y uso para todos los recursos propios de Azure. La información se presenta en la forma de registros de detalles de uso, que actualmente se emiten una vez por cada medidor y por cada recurso al día. La información se puede utilizar para sumar los costos de todos los recursos o investigar los costos o el uso de recursos específicos.

La API incluye:

-   **Datos de consumo a nivel de medidor**: consulte datos como el costo de uso, el medidor que emite el cargo y a qué recurso de Azure pertenece el cargo. Todos los registros de detalles de uso se asignan a un cubo diario.
-   **Control de acceso basado en rol de Azure**: configure las directivas de acceso en [Azure Portal](https://portal.azure.com), con la [CLI de Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) o mediante [cmdlets de Azure PowerShell](https://docs.microsoft.com/powershell/azure/) para especificar qué usuarios o aplicaciones pueden obtener acceso a los datos de uso de la suscripción. Los autores de llamadas deben utilizar tokens de Azure Active Directory estándar para la autenticación. Agregue el autor de la llamada al rol Lector de facturación, Lector, Propietario o Colaborador para que obtenga acceso a los datos de uso de una suscripción de Azure específica.
-   **Filtrado**: recorte el conjunto de resultados de API para conseguir un conjunto más pequeño de registros de detalles de uso con los siguientes filtros:
    - Inicio/fin del uso
    - Grupo de recursos
    - Nombre de recurso
-   **Agregación de datos**: use OData para aplicar expresiones para agregar detalles de uso por etiquetas o propiedades de filtro
-   la **información de los detalles de uso** está disponible actualmente para los clientes de Enterprise y Web Direct.

Para más información, vea la especificación técnica para la [API de detalles de uso](https://docs.microsoft.com/rest/api/consumption/usagedetails).

## <a name="marketplace-charges-api"></a>API de cargos de Marketplace

Use la API de cargos de Marketplace para obtener datos de cobros y uso de todos los recursos de Marketplace (ofertas de terceros para Azure). Estos datos se pueden utilizar para sumar los costos de todos los recursos de Marketplace o investigar los costos o el uso de recursos específicos.

La API incluye:

-   **Datos de consumo a nivel de medidor**: consulte datos como el costo de uso de marketplace, el medidor que emite el cargo y a qué recurso pertenece el cargo. Todos los registros de detalles de uso se asignan a un cubo diario.
-   **Control de acceso basado en rol de Azure**: configure las directivas de acceso en [Azure Portal](https://portal.azure.com), con la [CLI de Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) o mediante [cmdlets de Azure PowerShell](https://docs.microsoft.com/powershell/azure/) para especificar qué usuarios o aplicaciones pueden obtener acceso a los datos de uso de la suscripción. Los autores de llamadas deben utilizar tokens de Azure Active Directory estándar para la autenticación. Agregue el autor de la llamada al rol Lector de facturación, Lector, Propietario o Colaborador para que obtenga acceso a los datos de uso de una suscripción de Azure específica.
-   **Filtrado**: recorte el conjunto de resultados de API para conseguir un conjunto más pequeño de registros del marketplace con los siguientes filtros:
    - Inicio/fin del uso
    - Grupo de recursos
    - Nombre de recurso
-   **Uso para diferentes tipos de ofertas**: la información del marketplace está disponible actualmente para los clientes de Enterprise y Web Direct.

Para más información, consulte la especificación técnica de la [API de cargos del marketplace](https://docs.microsoft.com/rest/api/consumption/marketplaces).

## <a name="balances-api"></a>API de saldos

Los clientes de Enterprise pueden usar la API de saldos para obtener un resumen mensual de información sobre saldos, nuevas compras, gastos de servicios en Azure Marketplace, ajustes y gastos de uso por encima del límite. Puede obtener esta información para el período de facturación actual o cualquier período en el pasado. Las empresas pueden utilizar estos datos para realizar una comparación con los cargos de resumen calculados manualmente. Esta API no proporciona información específica de recursos y una vista agregada de los costos.

La API incluye:

-   **Control de acceso basado en rol de Azure**: configure las directivas de acceso en [Azure Portal](https://portal.azure.com), con la [CLI de Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) o mediante [cmdlets de Azure PowerShell](https://docs.microsoft.com/powershell/azure/) para especificar qué usuarios o aplicaciones pueden obtener acceso a los datos de uso de la suscripción. Los autores de llamadas deben utilizar tokens de Azure Active Directory estándar para la autenticación. Agregue el autor de la llamada al rol Lector de facturación, Lector, Propietario o Colaborador para que obtenga acceso a los datos de uso de una suscripción de Azure específica.
-   **Solo clientes de Enterprise** Esta API solo está disponible para clientes del Contrato Enterprise.
    - Los clientes deben tener permisos de administrador de Enterprise para llamar a esta API.

Para obtener más información, vea la especificación técnica de la [API de saldos](https://docs.microsoft.com/rest/api/consumption/balances).

## <a name="budgets-api"></a>API de presupuestos

Los clientes de Enterprise pueden usar esta API para crear presupuestos de costos o uso para recursos, grupos de recursos o medidores de facturación. Una vez determinada esta información, se pueden configurar alertas para notificar al usuario cuando se exceden umbrales de presupuestos definidos.

La API incluye:

-   **Control de acceso basado en rol de Azure**: configure las directivas de acceso en [Azure Portal](https://portal.azure.com), con la [CLI de Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) o mediante [cmdlets de Azure PowerShell](https://docs.microsoft.com/powershell/azure/) para especificar qué usuarios o aplicaciones pueden obtener acceso a los datos de uso de la suscripción. Los autores de llamadas deben utilizar tokens de Azure Active Directory estándar para la autenticación. Agregue el autor de la llamada al rol Lector de facturación, Lector, Propietario o Colaborador para que obtenga acceso a los datos de uso de una suscripción de Azure específica.
-   **Solo clientes de Enterprise** Esta API solo está disponible para clientes del Contrato Enterprise.
-   **Notificaciones configurables**: especifique los usuarios a los que hay que avisar cuando se excede el presupuesto.
-   **Presupuestos basados en uso o costos**: cree un presupuesto basado en el consumo o el costo, en función de las necesidades de su escenario.
-   **Filtrado**: filtre el presupuesto para conseguir un subconjunto más pequeño de recursos con el uso de los siguientes filtros configurables
    - Grupo de recursos
    - Nombre de recurso
    - Medidor
-   **Períodos de tiempo configurables para presupuestos**: especifique con qué frecuencia es necesario restablecer el presupuesto y durante cuánto tiempo es válido el presupuesto.

Para más información, consulte la especificación técnica de la [API de presupuestos](https://docs.microsoft.com/rest/api/consumption/budgets).

## <a name="reservation-recommendations-api"></a>API de recomendaciones de reserva

Use esta API para obtener recomendaciones de compra de instancias reservadas de máquina virtual. Las recomendaciones están diseñadas para permitir a los clientes analizar las cantidades de compra y los ahorros en costos esperados.

La API incluye:

-   **Control de acceso basado en rol de Azure**: configure las directivas de acceso en [Azure Portal](https://portal.azure.com), con la [CLI de Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) o mediante [cmdlets de Azure PowerShell](https://docs.microsoft.com/powershell/azure/) para especificar qué usuarios o aplicaciones pueden obtener acceso a los datos de uso de la suscripción. Los autores de llamadas deben utilizar tokens de Azure Active Directory estándar para la autenticación. Agregue el autor de la llamada al rol Lector de facturación, Lector, Propietario o Colaborador para que obtenga acceso a los datos de uso de una suscripción de Azure específica.
-   **Filtrado**: adapte los resultados de las recomendaciones con los siguientes filtros:
    - Ámbito
    - Período de retrospectiva
-   **Información de reserva para diferentes tipos de ofertas**: la información de reserva está disponible actualmente para los clientes de Enterprise y Web Direct.

Para más información, vea la especificación técnica de la [API de recomendaciones de reserva](https://docs.microsoft.com/rest/api/consumption/reservationrecommendations).

## <a name="reservation-details-api"></a>API de detalles de reserva

Use la API de detalles de reserva para consultar la información sobre las reservas de máquinas virtuales adquiridas anteriormente, como cuánto consumo se ha reservado frente al que se ha utilizado realmente. Puede consultar datos con detalles de cada máquina virtual.

La API incluye:

-   **Control de acceso basado en rol de Azure**: configure las directivas de acceso en [Azure Portal](https://portal.azure.com), con la [CLI de Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) o mediante [cmdlets de Azure PowerShell](https://docs.microsoft.com/powershell/azure/) para especificar qué usuarios o aplicaciones pueden obtener acceso a los datos de uso de la suscripción. Los autores de llamadas deben utilizar tokens de Azure Active Directory estándar para la autenticación. Agregue el autor de la llamada al rol Lector de facturación, Lector, Propietario o Colaborador para que obtenga acceso a los datos de uso de una suscripción de Azure específica.
-   **Filtrado**: recorte el conjunto de resultados de API para conseguir un conjunto más pequeño de reservas con los siguientes filtros:
    - Intervalo de fechas
-   **Información de reserva para diferentes tipos de ofertas**: la información de reserva está disponible actualmente para los clientes de Enterprise y Web Direct.

Para más información, vea la especificación técnica para la [API de detalles de reserva](https://docs.microsoft.com/rest/api/consumption/reservationsdetails).

## <a name="reservation-summaries-api"></a>API de resúmenes de reservas

Use esta API para consultar información agregada sobre las reservas de máquinas virtuales adquiridas anteriormente, como cuánto consumo se ha reservado frente a cuánto se ha usado realmente en el agregado.

La API incluye:

-   **Control de acceso basado en rol de Azure**: configure las directivas de acceso en [Azure Portal](https://portal.azure.com), con la [CLI de Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) o mediante [cmdlets de Azure PowerShell](https://docs.microsoft.com/powershell/azure/) para especificar qué usuarios o aplicaciones pueden obtener acceso a los datos de uso de la suscripción. Los autores de llamadas deben utilizar tokens de Azure Active Directory estándar para la autenticación. Agregue el autor de la llamada al rol Lector de facturación, Lector, Propietario o Colaborador para que obtenga acceso a los datos de uso de una suscripción de Azure específica.
-   **Filtrado**: adapte los resultados al usar el nivel de detalle diario con el filtro siguiente:
    - Fecha de uso
-   **Información de reserva para diferentes tipos de ofertas**: la información de reserva está disponible actualmente para los clientes de Enterprise y Web Direct.
-   **Agregaciones diarias o mensuales**: los autores de llamada pueden especificar si desean que sus datos de resúmenes de reservas presenten un nivel de detalle diario o mensual.

Para más información, vea la especificación técnica de la [API de resúmenes de reserva](https://docs.microsoft.com/rest/api/consumption/reservationssummaries).

## <a name="price-sheet-api"></a>API de hoja de precios
Los clientes de Enterprise pueden usar esta API para recuperar sus precios personalizados para todos los medidores. Las empresas pueden utilizar esto en combinación con los detalles de uso y la información de uso de marketplace para realizar cálculos de costos en función de los datos de uso y del catálogo de soluciones.

La API incluye:

-   **Control de acceso basado en rol de Azure**: configure las directivas de acceso en [Azure Portal](https://portal.azure.com), con la [CLI de Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) o mediante [cmdlets de Azure PowerShell](https://docs.microsoft.com/powershell/azure/) para especificar qué usuarios o aplicaciones pueden obtener acceso a los datos de uso de la suscripción. Los autores de llamadas deben utilizar tokens de Azure Active Directory estándar para la autenticación. Agregue el autor de la llamada al rol Lector de facturación, Lector, Propietario o Colaborador para que obtenga acceso a los datos de uso de una suscripción de Azure específica.
-   **Solo clientes de Enterprise** Esta API solo está disponible para clientes del Contrato Enterprise. Los clientes de Web Direct deben usar la API RateCard para obtener los precios.

Para más información, consulte la especificación técnica de la [API de hoja de precios](https://docs.microsoft.com/rest/api/consumption/pricesheet).

## <a name="scenarios"></a>Escenarios

Estos son algunos de los escenarios posibles con las API de consumo:

-   **Conciliación de facturas**: ¿Microsoft me cobró el importe correcto?  ¿Cuál es mi factura y puedo calcularla yo mismo?
-   **Cargos cruzados**: ahora que ya sé cuánto se me va a cobrar, ¿quién debe encargarse de pagar en mi organización?
-   **Optimización de costos**: sé cuánto se me ha cobrado... ¿cómo puedo obtener mayor rendimiento del dinero que gasto en Azure?
-   **Seguimiento de costos**: me gustaría consultar lo que gasto y uso en Azure a lo largo del tiempo. ¿Cuáles son las tendencias? ¿Cómo podría hacerlo mejor?
-   **Gasto en Azure durante el mes**: ¿cuánto llevo gastado este mes? ¿Debo realizar algún ajuste en mi gasto o uso de Azure? ¿En qué momento del mes registro más consumo de Azure?
-   **Configurar alertas**: me gustaría configurar alertas basadas en el gasto o el consumo de recursos en función de un presupuesto.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre el uso de las API de facturación de Azure para consultar información detallada mediante programación sobre el uso de Azure, consulte la [información general sobre las API de facturación de Azure](usage-rate-card-overview.md).
