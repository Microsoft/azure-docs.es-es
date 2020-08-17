---
title: Solución de errores comunes
description: Obtenga información sobre cómo solucionar problemas con los diversos SDK mientras consulta recursos de Azure con Azure Resource Graph.
ms.date: 08/10/2020
ms.topic: troubleshooting
ms.openlocfilehash: 052c3e1c10f1b0ca94653f086e2df4f3d395070e
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056948"
---
# <a name="troubleshoot-errors-using-azure-resource-graph"></a>Solución de problemas relativos a errores con Azure Resource Graph

Puede que se produzcan errores al consultar los recursos de Azure con Azure Resource Graph. En este artículo se describen diversos errores que pueden producirse y cómo resolverlos.

## <a name="finding-error-details"></a>Búsqueda de detalles del error

La mayoría de los errores son el resultado de una incidencia al ejecutar una consulta con Azure Resource Graph. Cuando se produce un error en una consulta, el SDK proporciona detalles sobre la consulta errónea. Esta información indica la incidencia, de modo que se pueda corregir y que una consulta posterior se realice correctamente.

## <a name="general-errors"></a>Errores generales

### <a name="scenario-throttled-requests"></a><a name="throttled"></a>Escenario: solicitudes limitadas

#### <a name="issue"></a>Incidencia

Los clientes que realizan consultas de recursos de gran tamaño o frecuentes tienen solicitudes limitadas.

#### <a name="cause"></a>Causa

Azure Resource Graph asigna un número de cuota para cada usuario basado en una ventana de tiempo. Por ejemplo, un usuario puede enviar como máximo 15 consultas dentro de cada ventana de 5 segundos sin que se vea limitado. El valor de la cuota se determina según varios factores y está sujeto a cambios. Para obtener más información, consulte [Limitaciones en Azure Resource Graph](../overview.md#throttling).

#### <a name="resolution"></a>Resolución

Hay varios métodos para tratar las solicitudes limitadas:

- [Agrupación de consultas](../concepts/guidance-for-throttled-requests.md#grouping-queries)
- [Escalonamiento de las consultas](../concepts/guidance-for-throttled-requests.md#staggering-queries)
- [Consulta en paralelo](../concepts/guidance-for-throttled-requests.md#query-in-parallel)
- [Paginación](../concepts/guidance-for-throttled-requests.md#pagination)

### <a name="scenario-too-many-subscriptions"></a><a name="toomanysubscription"></a>Escenario: demasiadas suscripciones

#### <a name="issue"></a>Incidencia

Los clientes con acceso a más de 1000 suscripciones, incluidas las suscripciones entre inquilinos con [Azure Lighthouse](../../../lighthouse/overview.md), no pueden capturar datos en todas las suscripciones en una única llamada a Azure Resource Graph.

#### <a name="cause"></a>Causa

La CLI de Azure y PowerShell reenvían solo las primeras 1000 suscripciones a Azure Resource Graph. La API REST para Azure Resource Graph acepta un número máximo de suscripciones en las que realizar la consulta.

#### <a name="resolution"></a>Resolución

Solicitudes por lotes de la consulta con un subconjunto de suscripciones para permanecer debajo del límite de suscripción de 1000. La solución es usar el parámetro **Subscription** en PowerShell.

```azurepowershell-interactive
# Replace this query with your own
$query = 'Resources | project type'

# Fetch the full array of subscription IDs
$subscriptions = Get-AzSubscription
$subscriptionIds = $subscriptions.Id

# Create a counter, set the batch size, and prepare a variable for the results
$counter = [PSCustomObject] @{ Value = 0 }
$batchSize = 1000
$response = @()

# Group the subscriptions into batches
$subscriptionsBatch = $subscriptionIds | Group -Property { [math]::Floor($counter.Value++ / $batchSize) }

# Run the query for each batch
foreach ($batch in $subscriptionsBatch){ $response += Search-AzGraph -Query $query -Subscription $batch.Group }

# View the completed results of the query on all subscriptions
$response
```

### <a name="scenario-unsupported-content-type-rest-header"></a><a name="rest-contenttype"></a>Escenario: encabezado Content-Type de REST no admitido

#### <a name="issue"></a>Incidencia

Los clientes que consultan la API REST de Azure Resource Graph reciben una respuesta _500_ (error interno del servidor).

#### <a name="cause"></a>Causa

La API REST de Azure Resource Graph solo admite un elemento `Content-Type` de **application/json**. Algunas herramientas o agentes de REST se establecen de forma predeterminada en **texto/sin formato**, un valor que la API REST no admite.

#### <a name="resolution"></a>Resolución

Compruebe que la herramienta o el agente que está usando para realizar la consulta a Azure Resource Graph tiene el encabezado de la API REST `Content-Type` configurado para **application/json**.

### <a name="scenario-no-read-permission-to-all-subscriptions-in-list"></a><a name="rest-403"></a>Escenario: no tiene permiso de lectura para todas las suscripciones de la lista.

#### <a name="issue"></a>Incidencia

Los clientes que pasan explícitamente una lista de suscripciones con una consulta de Azure Resource Graph reciben una respuesta _403_ (Prohibido).

#### <a name="cause"></a>Causa

Si el cliente no tiene permiso de lectura para todas las suscripciones proporcionadas, se denegará la solicitud debido a la falta de los derechos de seguridad adecuados.

#### <a name="resolution"></a>Resolución

Incluya al menos una suscripción de la lista de suscripciones a la que el cliente que ejecuta la consulta tiene acceso de lectura como mínimo. Para obtener más información, vea [Permisos en Azure Resource Graph](../overview.md#permissions-in-azure-resource-graph).

## <a name="next-steps"></a>Pasos siguientes

Si su problema no aparece o es incapaz de resolverlo, visite uno de nuestros canales para obtener más soporte técnico:

- Obtenga respuestas de expertos de Azure en los [foros de Azure](https://azure.microsoft.com/support/forums/).
- Póngase en contacto con [@AzureSupport](https://twitter.com/azuresupport): la cuenta de Microsoft Azure oficial para mejorar la experiencia del cliente, que pone en contacto a la comunidad de Azure con los recursos adecuados: respuestas, soporte técnico y expertos.
- Si necesita más ayuda, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte**.