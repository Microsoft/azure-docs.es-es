---
title: Administración de costos de Azure con Automation
description: En este artículo se explica cómo administrar los costos de Azure con Automation.
author: bandersmsft
ms.author: banders
ms.date: 09/14/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: matrive
ms.openlocfilehash: eb6ed73305d55b4f76464a4567c6b53715b10c3a
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90526653"
---
# <a name="manage-costs-with-automation"></a>Administración de costos con Automation

Puede usar Automation de Cost Management para crear un conjunto personalizado de soluciones para recuperar y administrar los datos de costos. En este artículo se describen escenarios comunes de Automation de Cost Management y las opciones disponibles en función de su situación. Para el desarrollo mediante API, se presentan ejemplos de solicitudes de API comunes para ayudar a acelerar el proceso de desarrollo.

## <a name="automate-cost-data-retrieval-for-offline-analysis"></a>Automatización de la recuperación de datos de costos para el análisis sin conexión

Es posible que tenga que descargar los datos de costos de Azure para combinarlos con otros conjuntos de datos. O bien, puede que necesite integrar los datos de costos en sus propios sistemas. Hay distintas opciones disponibles en función de la cantidad de datos implicados. En todo caso, debe tener permisos de Cost Management en el ámbito adecuado para usar las API y las herramientas. Para más información, consulte [Asignar el acceso a los datos](https://docs.microsoft.com/azure/cost-management-billing/costs/assign-access-acm-data).

## <a name="suggestions-for-handling-large-datasets"></a>Sugerencias para administrar grandes conjuntos de datos

Si su organización tiene una gran presencia de Azure en muchos recursos o suscripciones, tendrá una gran cantidad de datos de detalles de uso. Por lo general, Excel no puede cargar estos archivos de gran tamaño. En esta situación, se recomiendan las siguientes opciones:

**Power BI**

Power BI se usa para ingerir y controlar grandes cantidades de datos. Si es un cliente de Contrato Enterprise, puede usar la aplicación de plantilla de Power BI para analizar los costos de la cuenta de facturación. El informe contiene las principales vistas que usan los clientes. Para más información, consulte [Análisis de los costos de Azure con la aplicación de la plantilla de Power BI](https://docs.microsoft.com/azure/cost-management-billing/costs/analyze-cost-data-azure-cost-management-power-bi-template-app).

**Conector de datos de Power BI**

Si desea analizar los datos diariamente, se recomienda usar el [conector de datos de Power BI](https://docs.microsoft.com/power-bi/connect-data/desktop-connect-azure-cost-management) para obtener datos para un análisis detallado. El conector mantendrá actualizados todos los informes que cree a medida que se acumulen más costos.

**Exportaciones de Cost Management**

Es posible que no sea necesario analizar los datos a diario. Si es así, considere la posibilidad de usar la característica [Exportaciones](https://docs.microsoft.com/azure/cost-management-billing/costs/tutorial-export-acm-data) de Cost Management para programar exportaciones de datos a una cuenta de Azure Storage. Después, puede cargar los datos en Power BI según sea necesario o analizarlos en Excel si el archivo es lo suficientemente pequeño. Las exportaciones están disponibles en Azure Portal o puede configurar las exportaciones con [Exports API](https://docs.microsoft.com/rest/api/cost-management/exports).

**Usage Details API**:

Considere la posibilidad de usar [Usage Details API](https://docs.microsoft.com/rest/api/consumption/usageDetails) si tiene un conjunto de datos de costos reducido. Si tiene una gran cantidad de datos de costos, debe solicitar la cantidad más pequeña de datos de uso posible durante un período. Para ello, especifique un intervalo de tiempo pequeño o use un filtro en la solicitud. Por ejemplo, en un escenario en el que necesite tres años de datos de costos, la API funciona mejor si realiza varias llamadas para diferentes intervalos de tiempo en lugar de una sola llamada. Desde allí, puede cargar los datos en Excel para realizar más análisis.

## <a name="automate-retrieval-with-usage-details-api"></a>Automatización de la recuperación con Usage Details API

[Usage Details API](https://docs.microsoft.com/rest/api/consumption/usageDetails) proporciona una manera sencilla de obtener datos de costo sin procesar y sin agregar que se correspondan con su factura de Azure. La API es útil cuando su organización necesita una solución de recuperación de datos mediante programación. Considere la posibilidad de usar la API si quiere analizar conjuntos de datos de costos más reducidos. Sin embargo, debe usar otras soluciones identificadas previamente si tiene conjuntos de datos de mayor tamaño. Los datos de los detalles de uso se proporcionan por medidor y por día. Se usan al calcular la factura mensual. La versión de disponibilidad general (GA) de estas API es `2019-10-01`. Use `2019-04-01-preview` para acceder a la versión preliminar de las compras de reservas y Azure Marketplace con las API.

### <a name="usage-details-api-suggestions"></a>Sugerencias de Usage Details API

**Solicitar programación**

Se recomienda _no realizar más de una solicitud_ a Usage Details API por día. Para obtener más información sobre la frecuencia con que se actualizan los datos de costo y cómo se controla el redondeo, consulte [Descripción de los datos de Cost Management](https://docs.microsoft.com/azure/cost-management-billing/costs/understand-cost-mgt-data#rated-usage-data-refresh-schedule).

**Establecer ámbitos de alto nivel sin filtrado como destino**

Use la API para obtener todos los datos que necesite en el ámbito de nivel más alto disponible. Espere hasta que se ingieran todos los datos necesarios antes de realizar ningún filtrado, agrupación o análisis agregado. La API está optimizada específicamente para proporcionar grandes cantidades de datos de costos sin procesar no agregados. Para más información sobre los ámbitos disponibles en Cost Management, consulte [Descripción y uso de ámbitos](https://docs.microsoft.com/azure/cost-management-billing/costs/understand-work-scopes). Una vez que haya descargado los datos necesarios para un ámbito, use Excel para analizar más datos con filtros y tablas dinámicas.

## <a name="example-usage-details-api-requests"></a>Ejemplo de solicitudes de Usage Details API

Los clientes de Microsoft usan las siguientes solicitudes de ejemplo para abordar escenarios comunes con los que se puede encontrar.

### <a name="get-usage-details-for-a-scope-during-specific-date-range"></a>Obtención de detalles de uso de un ámbito durante un intervalo de fechas específico

Los datos que devuelve la solicitud se corresponden con la fecha en la que el sistema de facturación recibió el uso. Podría incluir los costos de varias facturas.

```http
GET https://management.azure.com/{scope}/providers/Microsoft.Consumption/usageDetails?$filter=properties%2FusageStart%20ge%20'2020-02-01'%20and%20properties%2FusageEnd%20le%20'2020-02-29'&$top=1000&api-version=2019-10-01

```

### <a name="get-amortized-cost-details"></a>Obtención de detalles de costos amortizados

Si necesita los costos reales para mostrar las compras que se han acumulado, cambie *metric* a `ActualCost` en la solicitud siguiente. Para usar los costos amortizados y reales, debe usar la versión de `2019-04-01-preview`. La versión actual de la API funciona igual que la versión de `2019-10-01`, excepto en el nuevo atributo type/metric y los nombres de propiedad modificados. Si tiene un Contrato de cliente de Microsoft, los filtros son `startDate` y `endDate` en el ejemplo siguiente.

```http
GET https://management.azure.com/{scope}/providers/Microsoft.Consumption/usageDetails?metric=AmortizedCost&$filter=properties/usageStart+ge+'2019-04-01'+AND+properties/usageEnd+le+'2019-04-30'&api-version=2019-04-01-preview
```

## <a name="retrieve-large-cost-datasets-recurringly-with-exports"></a>Recuperación recurrente de conjuntos de datos de costos de gran tamaño con Exportaciones

Desde Cost Management es posible exportar de manera regular grandes cantidades de datos con exportaciones. La exportación es el método recomendado para recuperar datos de costo no agregados. Especialmente cuando los archivos de uso son demasiado grandes para realizar llamadas y descargas de forma confiable mediante Usage Details API. Los datos exportados se colocan en la cuenta de Azure Storage que elija. Desde allí, los puede cargar en sus propios sistemas y analizarlos en función de sus necesidades. Para configurar las exportaciones en Azure Portal, consulte [Exportar datos](tutorial-export-acm-data.md).

Si desea automatizar las exportaciones en varios ámbitos, la solicitud de la API de ejemplo de la próxima sección es un buen punto de partida. Exports API se puede usar para crear exportaciones automáticas como parte de la configuración general del entorno. Las exportaciones automática sirven de ayuda para asegurarse que se tienen los datos necesarios y se pueden usar en los sistemas de cualquier organización cuando se expande el uso de Azure.

### <a name="common-export-configurations"></a>Configuraciones de exportación comunes

Antes de crear la primera exportación, tenga en cuenta el escenario y las opciones de configuración necesarias para habilitarlo. Considere las siguientes opciones de exportación:

- **Periodicidad**: determina la frecuencia con que se ejecuta el trabajo de exportación y cuándo se colocan los archivos en la cuenta de Azure Storage. Se puede elegir entre las siguientes opciones, Diaria, Semanal y Mensual. Intente configurar su periodicidad para que coincida con los trabajos de importación de datos que usa el sistema interno de su organización.
- **Periodo de periodicidad**: determina el periodo de validez de la exportación. Los archivos solo se exportan durante el período de periodicidad.
- **Período de tiempo**: determina la cantidad de datos que genera la exportación en una ejecución determinada. Las opciones comunes son MonthToDate y WeekToDate.
- **StartDate**: configura cuándo se desea que se inicie la programación de las exportaciones. Las exportaciones se crean en StartDate y, posteriormente, se basan en el valor de Periodicidad.
- **Tipo**: hay tres tipos de exportación:
  - ActualCost: muestra el uso total y los costos del período especificado, tal como se acumulan y se muestran en la factura.
  - AmortizedCost: muestra el uso total y los costos del período especificado, con la amortización aplicada a los costos de compra de la reserva que se aplican.
  - Usage: todas las exportaciones creadas antes del 20 de julio de 2020 son del tipo Usage. Actualice todas las exportaciones programadas para que sean de los tipos ActualCost o AmortizedCost.
- **Columnas**: define los campos de datos que se desean incluir en el archivo de exportación. Se corresponden con los campos disponibles en Usage Details API. Para más información, consulte [Usage Details API](/rest/api/consumption/usagedetails/list).

### <a name="create-a-daily-month-to-date-export-for-a-subscription"></a>Creación de una exportación diaria hasta la fecha de una suscripción

Dirección URL de la solicitud: `PUT https://management.azure.com/{scope}/providers/Microsoft.CostManagement/exports/{exportName}?api-version=2020-06-01`

```json
{
  "properties": {
    "schedule": {
      "status": "Active",
      "recurrence": "Daily",
      "recurrencePeriod": {
        "from": "2020-06-01T00:00:00Z",
        "to": "2020-10-31T00:00:00Z"
      }
    },
    "format": "Csv",
    "deliveryInfo": {
      "destination": {
        "resourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/MYDEVTESTRG/providers/Microsoft.Storage/storageAccounts/{yourStorageAccount} ",
        "container": "{yourContainer}",
        "rootFolderPath": "{yourDirectory}"
      }
    },
    "definition": {
      "type": "ActualCost",
      "timeframe": "MonthToDate",
      "dataSet": {
        "granularity": "Daily",
        "configuration": {
          "columns": [
            "Date",
            "MeterId",
            "ResourceId",
            "ResourceLocation",
            "Quantity"
          ]
        }
      }
    }
}
```

### <a name="automate-alerts-and-actions-with-budgets"></a>Automatización de alertas y acciones con Presupuestos

Hay dos componentes esenciales para maximizar el valor de su inversión en la nube. Uno es la creación automática de presupuestos. Mientras que el otro es la configuración de la orquestación basada en el costo en respuesta a las alertas presupuestarias. Hay diferentes maneras de automatizar la creación de presupuestos de Azure. Cuando se superan los umbrales de alerta configurados se producen varias respuestas de alerta.

En las secciones siguientes se tratan las opciones disponibles y se proporcionan solicitudes de API de ejemplo para empezar a trabajar con la automatización de presupuestos.

#### <a name="how-costs-are-evaluated-against-your-budget-threshold"></a>Evaluación de costos con respecto al umbral del presupuesto

Los costos se evalúan con respecto al umbral del presupuesto una vez al día. Cuando se crea un presupuesto o el día en que se restablece el presupuesto, los costos en comparación con el umbral serán cero o nulos, ya que es posible que no se haya producido la evaluación.

Cuando Azure detecta que los costos han superado el umbral, se desencadena una notificación dentro la hora del período de detección.

#### <a name="view-your-current-cost"></a>Visualización del costo actual

Para ver los costos actuales, es preciso realizar una llamada GET mediante [Query API](/rest/api/cost-management/query).

Las llamadas GET a Budgets API no devolverán los costos actuales que se muestran en Análisis de costos. En su lugar, la llamada devuelve el último costo evaluado.

### <a name="automate-budget-creation"></a>Automatización de la creación de presupuestos

Para automatizar la creación de presupuestos, utilice [Budgets API](/rest/api/consumption/budgets). También puede crear un presupuesto con una [plantilla de presupuesto](quick-create-budget-template.md). Las plantillas son una forma sencilla de estandarizar las implementaciones de Azure, a la vez que garantizan que el control de costos está correctamente configurado y aplicado.

#### <a name="supported-locales-for-budget-alert-emails"></a>Configuraciones regionales admitidas para los correos electrónicos de alertas de presupuesto

Recibirá una notificación cuando los costos de un presupuesto superen el umbral establecido. Puede configurar hasta cinco destinatarios de correo electrónico por presupuesto. Los destinatarios reciben las alertas por correo electrónico en un plazo de 24 horas desde que se supere el umbral del presupuesto. Sin embargo, es posible que el destinatario tenga que recibir el correo electrónico en otro idioma. Budgets API se puede usar con los siguientes códigos de referencia cultural. Establezca el código de referencia cultural con el parámetro `locale`, de forma similar a este ejemplo.

```json
{
  "eTag": "\"1d681a8fc67f77a\"",
  "properties": {
    "timePeriod": {
      "startDate": "2020-07-24T00:00:00Z",
      "endDate": "2022-07-23T00:00:00Z"
    },
    "timeGrain": "BillingMonth",
    "amount": 1,
    "currentSpend": {
      "amount": 0,
      "unit": "USD"
    },
    "category": "Cost",
    "notifications": {
      "actual_GreaterThan_10_Percent": {
        "enabled": true,
        "operator": "GreaterThan",
        "threshold": 20,
        "locale": "en-us",
        "contactEmails": [
          "user@contoso.com"
        ],
        "contactRoles": [],
        "contactGroups": [],
        "thresholdType": "Actual"
      }
    }
  }
}

```

Idiomas admitidos por un código de referencia cultural:

| Código de referencia cultural| Lenguaje |
| --- | --- |
| es-es | Spanish (Traditional Sort) - Spain |
| ja-jp | Japonés (Japón) |
| zh-CN | Chino (simplificado, China) |
| de-de | Alemán (Alemania) |
| es-es | Español (España, internacional) |
| fr-fr | Francés (Francia) |
| it-it | Italiano (Italia) |
| ko-kr | Coreano (Corea) |
| pt-br | Portugués (Brasil) |
| ru-ru | Ruso (Rusia) |
| zh-tw | Chino (tradicional, Taiwán) |
| cs-cz | Checo (República Checa) |
| pl-pl | Polaco (Polonia) |
| tr-tr | Turco (Turquía) |
| da-dk | Danés (Dinamarca) |
| dn-gb | Inglés (Reino Unido) |
| hu-HU | Húngaro (Hungría) |
| nb-bo | Noruego Bokmal (Noruega) |
| nl-nl | Neerlandés (Países Bajos) |
| pt-pt | Portugués (Portugal) |
| sv-se | Sueco (Suecia) |

#### <a name="common-budgets-api-configurations"></a>Configuraciones comunes de Budgets API

Hay muchas formas de configurar presupuestos en el entorno de Azure. Para hacerlo, en primer lugar debe tener en cuenta el escenario y, después, identificar las opciones de configuración que lo habilitan. Examine las siguientes opciones:

- **Intervalo de agregación**: representa el período recurrente que utiliza el presupuesto para acumular y evaluar los costos. Las opciones más comunes son: Mensual, Trimestral y Anual.
- **Período de tiempo**: representa el plazo de validez de un presupuesto. El presupuesto supervisa activamente y le alerta solo mientras sigue siendo válido.
- **Notificaciones**
  - Correos electrónicos de contacto: las direcciones de correo electrónico reciben alertas cuando un presupuesto acumula costos y supera los umbrales definidos.
  - Roles de contacto: todos los usuarios que tienen un rol de Azure RBAC que coincida en el ámbito dado reciben alertas por correo electrónico con esta opción. Por ejemplo, los propietarios de suscripciones podrían recibir una alerta de un presupuesto creado en el ámbito de la suscripción.
  - Grupos de contactos: el presupuesto llama a los grupos de acciones configurados cuando se supera un umbral de alerta.
- **Filtros de dimensión de costo** : el mismo filtro que puede realizar en Análisis de costos o en Query API también se puede realizar en el presupuesto. Este filtro se usa para reducir el rango de costos que se supervisan con el presupuesto.

Una vez que haya identificado las opciones de creación de presupuesto que satisfagan sus necesidades, cree el presupuesto mediante la API. El ejemplo siguiente le ayudará a comenzar con una configuración de presupuesto común.

**Creación de un presupuesto con filtros para varios recursos y etiquetas**

Dirección URL de la solicitud: `PUT https://management.azure.com/subscriptions/{SubscriptionId} /providers/Microsoft.Consumption/budgets/{BudgetName}/?api-version=2019-10-01`

```json
{
  "eTag": "\"1d34d016a593709\"",
  "properties": {
    "category": "Cost",
    "amount": 100.65,
    "timeGrain": "Monthly",
    "timePeriod": {
      "startDate": "2017-10-01T00:00:00Z",
      "endDate": "2018-10-31T00:00:00Z"
    },
    "filter": {
      "and": [
        {
          "dimensions": {
            "name": "ResourceId",
            "operator": "In",
            "values": [
              "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{meterName}",
              "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{meterName}"
            ]
          }
        },
        {
          "tags": {
            "name": "category",
            "operator": "In",
            "values": [
              "Dev",
              "Prod"
            ]
          }
        },
        {
          "tags": {
            "name": "department",
            "operator": "In",
            "values": [
              "engineering",
              "sales"
            ]
          }
        }
      ]
    },
    "notifications": {
      "Actual_GreaterThan_80_Percent": {
        "enabled": true,
        "operator": "GreaterThan",
        "threshold": 80,
        "contactEmails": [
          "user1@contoso.com",
          "user2@contoso.com"
        ],
        "contactRoles": [
          "Contributor",
          "Reader"
        ],
        "contactGroups": [
          "/subscriptions/{subscriptionID}/resourceGroups/{resourceGroupName}/providers/microsoft.insights/actionGroups/{actionGroupName}
        ],
        "thresholdType": "Actual"
      }
    }
  }
}
```

### <a name="configure-cost-based-orchestration-for-budget-alerts"></a>Configuración de orquestación basada en costos para las alertas presupuestarias

Puede configurar presupuestos para iniciar acciones automatizadas mediante los grupos de acciones de Azure. Para más información sobre la automatización de acciones mediante presupuestos, consulte [Automatización con los presupuestos de Azure](../manage/cost-management-budget-scenario.md).

## <a name="data-latency-and-rate-limits"></a>Latencia de datos y límites de frecuencia

Se recomienda llamar a las API no más de una vez al día. Los datos de Cost Management se actualizan cada cuatro horas a medida que se reciben nuevos datos de uso de los proveedores de recursos de Azure. Llamar con mayor frecuencia no proporcionará datos adicionales. En su lugar, creará una carga mayor. Para más información sobre la frecuencia con que cambian los datos y cómo se controla su latencia, consulte [Descripción de los datos de Cost Management](understand-cost-mgt-data.md).

### <a name="error-code-429---call-count-has-exceeded-rate-limits"></a>Código de error 429: el recuento de llamadas ha superado los límites de frecuencia

Para permitir una experiencia coherente para todos los suscriptores de Cost Management, las Cost Management API tienen una frecuencia limitada. Cuando se alcance este límite, recibirá el código de estado HTTP `429: Too many requests`. Los límites de rendimiento actuales para nuestras API son los siguientes:

- 30 llamadas por minuto: se aplica por ámbito, por usuario o por aplicación.
- 200 llamadas por minuto: se aplica por inquilino, por usuario o por aplicación.

## <a name="next-steps"></a>Pasos siguientes

- [Análisis de los costos de Azure con la aplicación de la plantilla de Power BI](https://docs.microsoft.com/azure/cost-management-billing/costs/analyze-cost-data-azure-cost-management-power-bi-template-app).
- [Creación y administración de datos exportados](https://docs.microsoft.com/azure/cost-management-billing/costs/tutorial-export-acm-data) con Exportaciones.
- Obtenga más información sobre [Usage Details API](https://docs.microsoft.com/rest/api/consumption/usageDetails).