---
title: Guía para solicitudes limitadas
description: Aprenda a agrupar, escalonar, paginar y consultar en paralelo las solicitudes para evitar que Azure Resource Graph las limite.
ms.date: 08/03/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: c8576fe38433026a28a3fb09a03332b5dd756bab
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89006013"
---
# <a name="guidance-for-throttled-requests-in-azure-resource-graph"></a>Guía de solicitudes limitadas en Azure Resource Graph

Al crear datos mediante programación y de uso frecuente Azure Resource Graph, se debe tener en cuenta cómo la limitación afectará a los resultados de las consultas. Cambiar la manera en que se solicitan los datos puede ayudar a evitar la limitación de la organización y mantener el flujo de datos puntuales en relación con los recursos de Azure.

En este artículo se describen cuatro áreas y patrones relacionados con la creación de consultas en Azure Resource Graph:

- Descripción de los encabezados de limitación
- Agrupación de consultas
- Escalonamiento de las consultas
- Impacto de la paginación

## <a name="understand-throttling-headers"></a>Descripción de los encabezados de limitación

Azure Resource Graph asigna un número de cuota para cada usuario en función de una ventana de tiempo. Por ejemplo, un usuario puede enviar como máximo 15 consultas dentro de cada ventana de 5 segundos sin que se vea limitado. El valor de la cuota se determina según varios factores y está sujeto a cambios.

En cada respuesta de consulta, Azure Resource Graph agrega dos encabezados de limitación:

- `x-ms-user-quota-remaining` (int): la cuota de recurso restante para el usuario. Este valor se asigna al número de consultas.
- `x-ms-user-quota-resets-after` (hh:mm:ss): el tiempo transcurrido hasta que se restablece el consumo de la cuota de un usuario.

Cuando una entidad de seguridad tiene acceso a más de 5000 suscripciones en el inquilino o el [ámbito de consulta](./query-language.md#query-scope) del grupo de administración, la respuesta se limita a las primeras 5000 suscripciones y el encabezado `x-ms-tenant-subscription-limit-hit` se devuelve como `true`.

Para ilustrar cómo funcionan los encabezados, echemos un vistazo a una respuesta de consulta que tiene el encabezado y los valores de `x-ms-user-quota-remaining: 10` y `x-ms-user-quota-resets-after: 00:00:03`.

- Dentro de los próximos 3 segundos, se pueden enviar como máximo 10 consultas sin que se sufra una limitación.
- En 3 segundos, los valores de `x-ms-user-quota-remaining` y `x-ms-user-quota-resets-after` se restablecerán a `15` y `00:00:05`, respectivamente.

Para ver un ejemplo del uso de los encabezados para el _retroceso_ de las solicitudes de consulta, consulte el ejemplo en [Consulta en paralelo](#query-in-parallel).

## <a name="grouping-queries"></a>Agrupación de consultas

La agrupación de consultas según la suscripción, grupo de recursos o recurso individual es más eficaz que realizar consultas en paralelo. El coste de la cuota de una consulta de mayor tamaño con frecuencia es menor que el coste de la cuota de muchas consultas pequeñas y dirigidas. Se recomienda que el tamaño del grupo sea inferior a _300_.

- Ejemplo de un método mal optimizado

  ```csharp
  // NOT RECOMMENDED
  var header = /* your request header */
  var subscriptionIds = /* A big list of subscriptionIds */

  foreach (var subscriptionId in subscriptionIds)
  {
      var userQueryRequest = new QueryRequest(
          subscriptions: new[] { subscriptionId },
          query: "Resoures | project name, type");

      var azureOperationResponse = await this.resourceGraphClient
          .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
          .ConfigureAwait(false);

  // ...
  }
  ```

- Ejemplo 1 de un método de agrupación optimizado

  ```csharp
  // RECOMMENDED
  var header = /* your request header */
  var subscriptionIds = /* A big list of subscriptionIds */

  const int groupSize = 100;
  for (var i = 0; i <= subscriptionIds.Count / groupSize; ++i)
  {
      var currSubscriptionGroup = subscriptionIds.Skip(i * groupSize).Take(groupSize).ToList();
      var userQueryRequest = new QueryRequest(
          subscriptions: currSubscriptionGroup,
          query: "Resources | project name, type");

      var azureOperationResponse = await this.resourceGraphClient
          .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
          .ConfigureAwait(false);

    // ...
  }
  ```

- Ejemplo 2 de un método de agrupación optimizado para obtener varios recursos en una consulta

  ```kusto
  Resources | where id in~ ({resourceIdGroup}) | project name, type
  ```

  ```csharp
  // RECOMMENDED
  var header = /* your request header */
  var resourceIds = /* A big list of resourceIds */

  const int groupSize = 100;
  for (var i = 0; i <= resourceIds.Count / groupSize; ++i)
  {
      var resourceIdGroup = string.Join(",",
          resourceIds.Skip(i * groupSize).Take(groupSize).Select(id => string.Format("'{0}'", id)));
      var userQueryRequest = new QueryRequest(
          subscriptions: subscriptionList,
          query: $"Resources | where id in~ ({resourceIdGroup}) | project name, type");

      var azureOperationResponse = await this.resourceGraphClient
          .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
          .ConfigureAwait(false);

    // ...
  }
  ```

## <a name="staggering-queries"></a>Escalonamiento de las consultas

Debido a la manera en que se aplica la limitación, se recomienda que las consultas se escalonen. Es decir, en lugar de enviar 60 consultas al mismo tiempo, escalone las consultas en cuatro ventanas de 5 segundos:

- Programación de consultas no escalonadas

  | Número de consultas         | 60  | 0    | 0     | 0     |
  |---------------------|-----|------|-------|-------|
  | Intervalo de tiempo (s) | 0-5 | 5-10 | 10-15 | 15-20 |

- Programación de consultas escalonadas

  | Número de consultas         | 15  | 15   | 15    | 15    |
  |---------------------|-----|------|-------|-------|
  | Intervalo de tiempo (s) | 0-5 | 5-10 | 10-15 | 15-20 |

A continuación proporcionamos un ejemplo de respetar los encabezados de limitación al consultar Azure Resource Graph:

```csharp
while (/* Need to query more? */)
{
    var userQueryRequest = /* ... */
    // Send post request to Azure Resource Graph
    var azureOperationResponse = await this.resourceGraphClient
        .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
        .ConfigureAwait(false);

    var responseHeaders = azureOperationResponse.response.Headers;
    int remainingQuota = /* read and parse x-ms-user-quota-remaining from responseHeaders */
    TimeSpan resetAfter = /* read and parse x-ms-user-quota-resets-after from responseHeaders */
    if (remainingQuota == 0)
    {
        // Need to wait until new quota is allocated
        await Task.Delay(resetAfter).ConfigureAwait(false);
    }
}
```

### <a name="query-in-parallel"></a>Consulta en paralelo

Aunque la agrupación es recomendable en comparación con la paralelización, hay ocasiones en las que las consultas no se pueden agrupar fácilmente. En estos casos, puede consultar Azure Resource Graph al enviar varias consultas en paralelo. A continuación proporcionamos un ejemplo de cómo hacer un _retroceso_ según los encabezados de limitación en este tipo de escenario:

```csharp
IEnumerable<IEnumerable<string>> queryGroup = /* Groups of queries  */
// Run groups in parallel.
await Task.WhenAll(queryGroup.Select(ExecuteQueries)).ConfigureAwait(false);

async Task ExecuteQueries(IEnumerable<string> queries)
{
    foreach (var query in queries)
    {
        var userQueryRequest = new QueryRequest(
            subscriptions: subscriptionList,
            query: query);
        // Send post request to Azure Resource Graph.
        var azureOperationResponse = await this.resourceGraphClient
            .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
            .ConfigureAwait(false);
        
        var responseHeaders = azureOperationResponse.response.Headers;
        int remainingQuota = /* read and parse x-ms-user-quota-remaining from responseHeaders */
        TimeSpan resetAfter = /* read and parse x-ms-user-quota-resets-after from responseHeaders */
        if (remainingQuota == 0)
        {
            // Delay by a random period to avoid bursting when the quota is reset.
            var delay = (new Random()).Next(1, 5) * resetAfter;
            await Task.Delay(delay).ConfigureAwait(false);
        }
    }
}
```

## <a name="pagination"></a>Paginación

Dado que Azure Resource Graph devuelve 1000 entradas como máximo en una respuesta de consulta única, es posible que tenga que [paginar](./work-with-data.md#paging-results) las consultas para obtener el conjunto de datos completo que está buscando. Sin embargo, algunos clientes de Azure Resource Graph controlan la paginación de forma diferente a otros.

- SDK DE C#

  Al usar el SDK de ResourceGraph, tendrá que controlar la paginación al pasar el token de omisión que se devuelve de la respuesta de la consulta anterior a la siguiente consulta paginada. Este diseño significa que necesita recopilar los resultados de todas las llamadas paginadas y combinarlos al final. En este caso, cada consulta paginada que envíe toma una cuota de consulta:

  ```csharp
  var results = new List<object>();
  var queryRequest = new QueryRequest(
      subscriptions: new[] { mySubscriptionId },
      query: "Resources | project id, name, type | top 5000");
  var azureOperationResponse = await this.resourceGraphClient
      .ResourcesWithHttpMessagesAsync(queryRequest, header)
      .ConfigureAwait(false);
  while (!string.Empty(azureOperationResponse.Body.SkipToken))
  {
      queryRequest.SkipToken = azureOperationResponse.Body.SkipToken;
      // Each post call to ResourceGraph consumes one query quota
      var azureOperationResponse = await this.resourceGraphClient
          .ResourcesWithHttpMessagesAsync(queryRequest, header)
          .ConfigureAwait(false);
      results.Add(azureOperationResponse.Body.Data.Rows);

      // Inspect throttling headers in query response and delay the next call if needed.
  }
  ```

- CLI de Azure o Azure PowerShell

  Cuando se usa la CLI de Azure o Azure PowerShell, las consultas a Azure Resource Graph se paginan automáticamente para capturar 5000 entradas como máximo. Los resultados de la consulta devuelven una lista combinada de las entradas de todas las llamadas paginadas. En este caso, según el número de entradas en el resultado de la consulta, una sola consulta paginada puede consumir más de una cuota de consulta. Por ejemplo, en el ejemplo siguiente, una única ejecución de la consulta puede consumir hasta cinco cuotas de consulta:

  ```azurecli-interactive
  az graph query -q 'Resources | project id, name, type' --first 5000
  ```

  ```azurepowershell-interactive
  Search-AzGraph -Query 'Resources | project id, name, type' -First 5000
  ```

## <a name="still-get-throttled"></a>¿Todavía se ve limitado?

Si se ve limitado después de aplicar las recomendaciones anteriores, póngase en contacto con el equipo en [resourcegraphsupport@microsoft.com](mailto:resourcegraphsupport@microsoft.com).

Proporcione esta información:

- Sus necesidades de controlador de casos prácticos y empresariales de un límite superior.
- ¿A cuántos recursos tiene acceso? ¿Cuántos de ellos se devuelven desde una consulta única?
- ¿Qué tipos de recursos le interesa?
- ¿Cuál es el modelo de consulta? X consultas por Y segundo, etc.

## <a name="next-steps"></a>Pasos siguientes

- Vea el lenguaje en uso en[Consultas básicas](../samples/starter.md).
- Vea los usos avanzados en [Consultas avanzadas](../samples/advanced.md).
- Obtenga más información sobre cómo [explorar recursos](explore-resources.md).