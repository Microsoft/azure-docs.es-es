---
title: Publicación de eventos con dominios de eventos mediante Azure Event Grid
description: Muestra cómo administrar grandes conjuntos de temas de Azure Event Grid y publicar eventos en ellos con dominios de eventos.
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9016d26384827279a5a89afecff59f572d7ce273
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502042"
---
# <a name="manage-topics-and-publish-events-using-event-domains"></a>Administrar temas y publicar eventos con dominios de eventos

En este artículo se muestra cómo:

* Crear un dominio de Event Grid
* Suscribirse a temas de Event Grid
* Enumeración de claves
* Publicar eventos en un dominio

Para más información acerca de dominios de eventos, consulte [Dominios de eventos para administrar temas de Event Grid](event-domains.md).

[!INCLUDE [requires-azurerm](../../includes/requires-azurerm.md)]

## <a name="install-preview-feature"></a>Instalación de la característica en vista previa

[!INCLUDE [event-grid-preview-feature-note.md](../../includes/event-grid-preview-feature-note.md)]

## <a name="create-an-event-domain"></a>Crear un dominio de eventos

Para administrar grandes conjuntos de temas, cree un dominio de eventos.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli)

```azurecli-interactive
# If you haven't already installed the extension, do it now.
# This extension is required for preview features.
az extension add --name eventgrid

az eventgrid domain create \
  -g <my-resource-group> \
  --name <my-domain-name> \
  -l <location>
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)
```azurepowershell-interactive
# If you have not already installed the module, do it now.
# This module is required for preview features.
Install-Module -Name AzureRM.EventGrid -AllowPrerelease -Force -Repository PSGallery

New-AzureRmEventGridDomain `
  -ResourceGroupName <my-resource-group> `
  -Name <my-domain-name> `
  -Location <location>
```
---

Una creación correcta devolverá los siguientes valores:

```json
{
  "endpoint": "https://<my-domain-name>.westus2-1.eventgrid.azure.net/api/events",
  "id": "/subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>",
  "inputSchema": "EventGridSchema",
  "inputSchemaMapping": null,
  "location": "westus2",
  "name": "<my-domain-name>",
  "provisioningState": "Succeeded",
  "resourceGroup": "<my-resource-group>",
  "tags": null,
  "type": "Microsoft.EventGrid/domains"
}
```

Anote `endpoint` y `id`, ya que se requieren para administrar el dominio y publicar eventos.

## <a name="manage-access-to-topics"></a>Administración del acceso a temas

La administración de acceso a temas se realiza mediante [asignación de roles](../role-based-access-control/role-assignments-cli.md). La asignación de roles usa el control de acceso basado en roles para limitar las operaciones en recursos de Azure a los usuarios autorizados en un determinado ámbito.

Event Grid tiene dos roles integrados, que puede utilizar para asignar acceso a determinados usuarios a varios temas dentro de un dominio. Estos roles son `EventGrid EventSubscription Contributor (Preview)`, que permite la creación y eliminación de suscripciones, y `EventGrid EventSubscription Reader (Preview)`, que permite solo enumerar las suscripciones a eventos.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli)
El siguiente comando de la CLI de Azure limita `alice@contoso.com` para crear y eliminar suscripciones de eventos solo en el tema `demotopic1`:

```azurecli-interactive
az role assignment create \
  --assignee alice@contoso.com \
  --role "EventGrid EventSubscription Contributor (Preview)" \
  --scope /subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>/topics/demotopic1
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)
El siguiente comando de PowerShell limita `alice@contoso.com` para crear y eliminar suscripciones de eventos solo en el tema `demotopic1`:

```azurepowershell-interactive
New-AzureRmRoleAssignment `
  -SignInName alice@contoso.com `
  -RoleDefinitionName "EventGrid EventSubscription Contributor (Preview)" `
  -Scope /subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>/topics/demotopic1
```
---

Para más información sobre la administración de acceso para las operaciones de Event Grid, consulte [Autenticación y seguridad de Event Grid](./security-authentication.md).

## <a name="create-topics-and-subscriptions"></a>Creación de temas y suscripciones

El servicio Event Grid crea y administra automáticamente el tema correspondiente en un dominio basado en la llamada para crear una suscripción de eventos para un tema de dominio. No hay ningún paso más para crear un tema en un dominio. De forma similar, cuando se elimina la última suscripción del evento para un tema, también se elimina el tema.

Suscribirse a un tema en un dominio es lo mismo que suscribirse a cualquier otro recurso de Azure. Para el identificador de recurso de origen, especifique el identificador de dominio de evento que se devolvió al crear el dominio anteriormente. Para especificar el tema al que quiere suscribirse, agregue `/topics/<my-topic>` al final del identificador del recurso de origen. Para crear una suscripción de eventos del ámbito de dominio que reciba todos los eventos del dominio, especifique el identificador de dominio de evento sin especificar ningún tema.

Normalmente, el usuario al que concedió acceso en la sección anterior crearía la suscripción. Para simplificar este artículo, cree la suscripción. 

# <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli)

```azurecli-interactive
az eventgrid event-subscription create \
  --name <event-subscription> \
  --source-resource-id "/subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>/topics/demotopic1" \
  --endpoint https://contoso.azurewebsites.net/api/updates
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurepowershell-interactive
New-AzureRmEventGridSubscription `
  -ResourceId "/subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>/topics/demotopic1" `
  -EventSubscriptionName <event-subscription> `
  -Endpoint https://contoso.azurewebsites.net/api/updates
```

---

Si necesita un punto de conexión de prueba al que suscribir sus eventos, siempre puede implementar una [aplicación web precompilada](https://github.com/Azure-Samples/azure-event-grid-viewer) que muestre los eventos entrantes. Puede enviar los eventos a su sitio web de prueba en `https://<your-site-name>.azurewebsites.net/api/updates`.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"  alt="Button to Deploy to Aquent." /></a>

Los permisos que se establecen para un tema se almacenan en Azure Active Directory y se deben eliminar explícitamente. Eliminar una suscripción de eventos no revoca el acceso de un usuario para crear suscripciones de eventos si tiene acceso de escritura en un tema.


## <a name="publish-events-to-an-event-grid-domain"></a>Publicar eventos en un dominio de Event Grid

Publicar eventos en un dominio es lo mismo que [publicar en un tema personalizado](./post-to-custom-topic.md). Sin embargo, en lugar de publicar en el tema personalizado, debe publicar todos los eventos en el punto de conexión del dominio. En los datos del evento JSON, especifique el tema al que desea que vayan los eventos. La siguiente matriz de eventos enviará eventos con `"id": "1111"` al tema `demotopic1`, mientras que el evento con `"id": "2222"` se enviará al tema `demotopic2`:

```json
[{
  "topic": "demotopic1",
  "id": "1111",
  "eventType": "maintenanceRequested",
  "subject": "myapp/vehicles/diggers",
  "eventTime": "2018-10-30T21:03:07+00:00",
  "data": {
    "make": "Contoso",
    "model": "Small Digger"
  },
  "dataVersion": "1.0"
},
{
  "topic": "demotopic2",
  "id": "2222",
  "eventType": "maintenanceCompleted",
  "subject": "myapp/vehicles/tractors",
  "eventTime": "2018-10-30T21:04:12+00:00",
  "data": {
    "make": "Contoso",
    "model": "Big Tractor"
  },
  "dataVersion": "1.0"
}]
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli)
Para obtener el punto de conexión del dominio con la CLI de Azure, use

```azurecli-interactive
az eventgrid domain show \
  -g <my-resource-group> \
  -n <my-domain>
```

Para obtener las claves para un dominio, use:

```azurecli-interactive
az eventgrid domain key list \
  -g <my-resource-group> \
  -n <my-domain>
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)
Para obtener el punto de conexión del dominio con PowerShell, use

```azurepowershell-interactive
Get-AzureRmEventGridDomain `
  -ResourceGroupName <my-resource-group> `
  -Name <my-domain>
```

Para obtener las claves para un dominio, use:

```azurepowershell-interactive
Get-AzureRmEventGridDomainKey `
  -ResourceGroupName <my-resource-group> `
  -Name <my-domain>
```
---

Después, use su método favorito para crear una solicitud HTTP POST para publicar los eventos en el dominio de Event Grid.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los conceptos de alto nivel en dominios de eventos y saber por qué son útiles, consulte la [información general conceptual acerca de dominios de eventos](event-domains.md).
