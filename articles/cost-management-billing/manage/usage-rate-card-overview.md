---
title: Obtención del uso de Azure con las API de facturación de Azure
description: Obtenga información sobre las API de RateCard y de uso de facturación de Azure que se usan para proporcionar información sobre el consumo de recursos y tendencias de Azure.
author: tonguyen
ms.reviewer: mumami
tags: billing
ms.service: cost-management-billing
ms.topic: reference
ms.date: 02/12/2020
ms.author: banders
ms.custom: seodec18
ms.openlocfilehash: 3ed655ed419e3be378a68b26b9f14b03c1af4796
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87039521"
---
# <a name="use-azure-billing-apis-to-programmatically-get-insight-into-your-azure-usage"></a>Use las API de facturación de Azure para obtener información mediante programación sobre cómo usa Azure
Use las API de facturación de Azure para extraer datos de uso y de recursos e incorporarlos en las herramientas de análisis de datos de su preferencia. Las API de RateCard y de uso de recursos de Azure pueden ayudarlo a predecir y administrar los costos de forma precisa. Las API se implementan como proveedor de recursos, como parte de la familia de API expuesta por Azure Resource Manager.  

## <a name="azure-invoice-download-api-preview"></a>API de descarga de facturas de Azure (vista previa)
Una vez [completada la suscripción](manage-billing-access.md#opt-in), descargue facturas con la versión de vista previa de [API de facturas](/rest/api/billing). Las características incluyen:

* **Control de acceso basado en rol de Azure**: configure las directivas de acceso en [Azure Portal](https://portal.azure.com) o mediante [cmdlets de Azure PowerShell](/powershell/azure/) para especificar qué usuarios o aplicaciones pueden obtener acceso a los datos de uso de la suscripción. Los autores de llamadas deben utilizar tokens de Azure Active Directory estándar para la autenticación. Agregue el autor de la llamada al rol Lector de facturación, Lector, Propietario o Colaborador para que obtenga acceso a los datos de uso de una suscripción de Azure específica.
* **Filtrado de fechas**: use el parámetro `$filter` para obtener todas las facturas en orden cronológico inverso por la fecha de finalización del periodo de facturación.

> [!NOTE]
> Esta característica se encuentra en su primera versión de vista previa y puede estar sujeta a cambios no compatibles con versiones anteriores. De momento no está disponible para determinadas ofertas de suscripción (EA, CSP, AIO no admitidas) y Azure Germany.

## <a name="azure-resource-usage-api-preview"></a>API de uso de recursos de Azure (vista previa)
Use la [API de uso de recursos](/previous-versions/azure/reference/mt219003(v=azure.100)) de Azure para obtener los datos estimados de consumo de Azure. La API incluye:

* **Control de acceso basado en rol de Azure**: configure las directivas de acceso en [Azure Portal](https://portal.azure.com) o mediante [cmdlets de Azure PowerShell](/powershell/azure/) para especificar qué usuarios o aplicaciones pueden obtener acceso a los datos de uso de la suscripción. Los autores de llamadas deben utilizar tokens de Azure Active Directory estándar para la autenticación. Agregue el autor de la llamada al rol Lector de facturación, Lector, Propietario o Colaborador para que obtenga acceso a los datos de uso de una suscripción de Azure específica.
* **Agregaciones cada hora o diarias** : los autores de llamadas pueden especificar si desean sus datos de uso de Azure en depósitos cada hora o diarios. El valor predeterminado es diario.
* **Metadatos de instancia (incluye etiquetas de recursos)** : obtenga detalles de nivel de instancia, como el URI de recurso completo (/subscriptions/{id-suscripción}/..), la información del grupo de recursos y las etiquetas de recursos. Estos metadatos lo ayudarán de forma determinista y mediante programación a asignar el uso mediante las etiquetas, para casos de uso como cargos cruzados.
* **Metadatos de recursos**: detalles de recursos, como el nombre del medidor, la categoría del medidor, la subcategoría del medidor, la unidad y la región permiten que el autor de la llamada comprenda mejor lo que se ha consumido. También estamos trabajando para alinear la terminología de metadatos de recursos en todo Azure Portal, CSV de uso de Azure, CSV de facturación de EA y otras experiencias orientadas al público, para permitirle que pueda poner en correlación los datos en todas las experiencias.
* **Uso para diferentes tipos de ofertas**: los datos de uso están disponibles para tipos de ofertas como el pago por uso, MSDN, compromiso monetario, crédito monetario y EA, excepto [CSP](https://docs.microsoft.com/partner-center).

## <a name="azure-resource-ratecard-api-preview"></a>API de RateCard de recursos de Azure (vista previa)
Use la [API de RateCard de recursos de Azure](/previous-versions/azure/reference/mt219005(v=azure.100)) para obtener la lista de recursos disponibles de Azure y una información de precios estimada para cada uno. La API incluye:

* **Control de acceso basado en rol de Azure**: configure las directivas de acceso en [Azure Portal](https://portal.azure.com) o mediante [cmdlets de Azure PowerShell](/powershell/azure/) para especificar qué usuarios o aplicaciones pueden obtener acceso a los datos de RateCard. Los autores de llamadas deben utilizar tokens de Azure Active Directory estándar para la autenticación. Agregue el autor de la llamada al rol Lector, Propietario o Colaborador para que obtenga acceso a los datos de uso de una suscripción de Azure determinada.
* **Compatibilidad con ofertas de pago por uso, MSDN, compromiso monetario y crédito monetario (no compatible con EA y [CSP](https://docs.microsoft.com/partner-center))** : esta API proporciona información de tarifas de nivel de oferta de Azure.  El autor de llamada de esta API debe pasar la información de oferta para obtener detalles y tarifas de recursos Actualmente no podemos proporcionar tarifas de EA porque las ofertas de EA tienen tarifas personalizadas por inscripción.

## <a name="scenarios"></a>Escenarios
Éstos son algunos de los escenarios posibles con la combinación de las API de uso y de RateCard:

* **Gasto en Azure durante el mes**: use la combinación de las API de uso y de RateCard para obtener información más detallada sobre el gasto por usar la nube durante el mes. Puede analizar los depósitos por hora y diarios de las estimaciones de usos y cargos.
* **Configurar alertas**: use las API de uso y de RateCard para obtener una estimación del consumo de la nube y los cargos, y configure alertas basadas en recursos o basadas en importes monetarios.
* **Predicción de facturas**: obtenga el gasto en la nube y el consumo estimados, y aplique algoritmos de aprendizaje automático para poder predecir cuál sería la factura al final del ciclo de facturación.
* **Análisis de costos previos al consumo**: use la API de RateCard para predecir cuál sería el importe de su factura para el uso esperado cuando mueva las cargas de trabajo a Azure. Si tiene cargas de trabajo existentes en otras nubes o nubes privadas, también puede asignar su uso con las tarifas de Azure para obtener una mejor estimación del gasto de Azure. Esta estimación le permite tener la capacidad de dinamizar una oferta y de comparar y contrastar los distintos tipos de ofertas que van más allá del pago por uso, como son el compromiso monetario y el crédito monetario. La API también le ofrece la posibilidad de ver las diferencias de costos por región y le permite realizar un análisis de hipótesis para ayudarlo a tomar decisiones en cuanto a la implementación.
* **Análisis de hipótesis** -

  * Puede determinar si es más rentable ejecutar las cargas de trabajo en otra región o en otra configuración del recurso de Azure. Los costos de los recursos de Azure pueden variar en función de la región de Azure que usa.
  * También puede determinar si otro tipo de oferta de Azure ofrece una mejor tarifa en un recurso de Azure.


## <a name="next-steps"></a>Pasos siguientes
* Vea los ejemplos de código en GitHub:
  * [Ejemplo de código de API de facturas](https://go.microsoft.com/fwlink/?linkid=845124)

  * [Ejemplo de código de API de uso](https://github.com/Azure-Samples/billing-dotnet-usage-api)

  * [Ejemplo de código de API de RateCard](https://github.com/Azure-Samples/billing-dotnet-ratecard-api)

* Para más información sobre Azure Resource Manager, vea [Información general de Azure Resource Manager](../../azure-resource-manager/management/overview.md).
