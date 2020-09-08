---
title: Creación de conjuntos de registros y zonas DNS con el SDK de .NET
titleSuffix: Azure DNS
description: En esta ruta de aprendizaje, comience a crear conjuntos de registros y zonas DNS en Azure DNS mediante el SDK de .NET.
services: dns
documentationcenter: na
author: rohinkoul
manager: kumudD
ms.assetid: eed99b87-f4d4-4fbf-a926-263f7e30b884
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/19/2016
ms.author: rohink
ms.custom: devx-track-csharp
ms.openlocfilehash: 92509ff96af0ed2a7d77799edca8447ff6839ab6
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019664"
---
# <a name="create-dns-zones-and-record-sets-using-the-net-sdk"></a>Creación de conjuntos de registros y zonas DNS con el SDK de .NET

Puede automatizar las operaciones para crear, eliminar o actualizar zonas, conjuntos de registros y registros de DNS mediante el SDK de DNS con la biblioteca de administración de DNS de .NET. Un proyecto completo de Visual Studio se encuentra disponible [aquí](https://www.microsoft.com/en-us/download/details.aspx?id=47268&WT.mc_id=DX_MVP4025064&e6b34bbe-475b-1abd-2c51-b5034bcdd6d2=True).

## <a name="create-a-service-principal-account"></a>Creación de una cuenta de entidad de servicio

Normalmente, el acceso a recursos de Azure mediante programación se concede mediante una cuenta dedicada en lugar de por medio de sus propias credenciales de usuario. Estas cuentas dedicadas se denominan cuentas de 'entidad de servicio'. Para usar el proyecto de ejemplo del SDK de DNS de Azure, primero debe crear una cuenta de entidad de servicio y asignarle los permisos correctos.

1. Siga [estas instrucciones](../active-directory/develop/howto-authenticate-service-principal-powershell.md) para crear una cuenta de entidad de servicio (en el proyecto de ejemplo del SDK de DNS de Azure se asume una autenticación basada en contraseña).
2. Cree un grupo de recursos ([aquí se explica cómo](../azure-resource-manager/templates/deploy-portal.md)).
3. Use RBAC de Azure para conceder a la cuenta de entidad de servicio permisos de 'Colaborador de zona DNS' al grupo de recursos ([aquí le decimos cómo](../role-based-access-control/role-assignments-portal.md)).
4. Si va a usar el proyecto de ejemplo del SDK de DNS de Azure, edite el archivo 'program.cs' de la manera siguiente:

   * Inserte los valores correctos para `tenantId`, `clientId` (también conocido como identificador de cuenta), `secret` (contraseña de la cuenta de entidad de servicio) y `subscriptionId` usados en el paso 1.
   * Escriba el nombre del grupo de recursos seleccionado en el paso 2.
   * Escriba un nombre de zona DNS de su elección.

## <a name="nuget-packages-and-namespace-declarations"></a>Paquetes NuGet y declaraciones de espacio de nombres

Para usar el SDK de .NET de DNS de Azure, debe instalar el paquete NuGet de la **biblioteca de administración de DNS de Azure** y otros paquetes de Azure necesarios.

1. En **Visual Studio**, abra un proyecto o cree uno nuevo.
2. Vaya a **Herramientas** **>** **Administrador de paquetes NuGet** **>** **Administrar paquetes NuGet para la solución**.
3. Haga clic en **Examinar**, active la casilla **Incluir versión previa** y escriba **Microsoft.Azure.Management.Dns** en el cuadro de búsqueda.
4. Seleccione el paquete y haga clic en **Instalar** para agregarlo a su proyecto de Visual Studio.
5. Repita el proceso anterior para instalar también los siguientes paquetes: **Microsoft.Rest.ClientRuntime.Azure.Authentication** y **Microsoft.Azure.Management.ResourceManager**.

## <a name="add-namespace-declarations"></a>Incorporación de declaraciones de espacio de nombres

Adición de las siguientes declaraciones de espacio de nombres

```cs
using Microsoft.Rest.Azure.Authentication;
using Microsoft.Azure.Management.Dns;
using Microsoft.Azure.Management.Dns.Models;
```

## <a name="initialize-the-dns-management-client"></a>Inicialización del cliente de administración de DNS

`DnsManagementClient` contiene los métodos y las propiedades necesarios para administrar los conjuntos de registros y las zonas DNS.  El código siguiente inicia una sesión en la cuenta de entidad de servicio y crea un objeto `DnsManagementClient`.

```cs
// Build the service credentials and DNS management client
var serviceCreds = await ApplicationTokenProvider.LoginSilentAsync(tenantId, clientId, secret);
var dnsClient = new DnsManagementClient(serviceCreds);
dnsClient.SubscriptionId = subscriptionId;
```

## <a name="create-or-update-a-dns-zone"></a>Creación o actualización de una zona DNS

Para crear una zona DNS, primero se crea un objeto "Zona" para que contenga los parámetros de la zona DNS. Como las zonas DNS no están vinculadas a una región específica, la ubicación se establece en "global". En este ejemplo, también se agrega una ['etiqueta' de Azure Resource Manager](https://azure.microsoft.com/updates/organize-your-azure-resources-with-tags/) a la zona.

Para crear o actualizar realmente la zona en Azure DNS, el objeto de zona que contiene los parámetros de zona se pasa al método `DnsManagementClient.Zones.CreateOrUpdateAsyc`.

> [!NOTE]
> DnsManagementClient admite tres modos de funcionamiento: sincrónico ('CreateOrUpdate'), asincrónico ('CreateOrUpdateAsync') o asincrónico con acceso a la respuesta HTTP ('CreateOrUpdateWithHttpMessagesAsync').  Puede elegir cualquiera de estos modos, dependiendo de sus necesidades de aplicación.

DNS de Azure admite la simultaneidad optimista denominada [Etags](dns-getstarted-create-dnszone.md). En este ejemplo, al especificar "*" para el encabezado 'If-None-Match', indica a DNS de Azure que cree una zona DNS si no existe ya una.  La llamada produce error si una zona con el nombre dado ya existe en el grupo de recursos dado.

```cs
// Create zone parameters
var dnsZoneParams = new Zone("global"); // All DNS zones must have location = "global"

// Create an Azure Resource Manager 'tag'.  This is optional.  You can add multiple tags
dnsZoneParams.Tags = new Dictionary<string, string>();
dnsZoneParams.Tags.Add("dept", "finance");

// Create the actual zone.
// Note: Uses 'If-None-Match *' ETAG check, so will fail if the zone exists already.
// Note: For non-async usage, call dnsClient.Zones.CreateOrUpdate(resourceGroupName, zoneName, dnsZoneParams, null, "*")
// Note: For getting the http response, call dnsClient.Zones.CreateOrUpdateWithHttpMessagesAsync(resourceGroupName, zoneName, dnsZoneParams, null, "*")
var dnsZone = await dnsClient.Zones.CreateOrUpdateAsync(resourceGroupName, zoneName, dnsZoneParams, null, "*");
```

## <a name="create-dns-record-sets-and-records"></a>Creación de registros y conjuntos de registros de DNS

Los registros DNS se administran como un conjunto de registros. Un conjunto de registros es un conjunto de registros con el mismo nombre y tipo de registro dentro de una zona.  El nombre del conjunto de registros es relativo al nombre de zona, no al nombre DNS completo.

Para crear o actualizar un conjunto de registros, se crea un objeto de parámetros "RecordSet" y se pasa a `DnsManagementClient.RecordSets.CreateOrUpdateAsync`. Al igual que con las zonas DNS, existen tres modos de funcionamiento: sincrónico ('CreateOrUpdate'), asincrónico ('CreateOrUpdateAsync') o asincrónico con acceso a la respuesta HTTP ('CreateOrUpdateWithHttpMessagesAsync').

Al igual que con las zonas DNS, las operaciones en los conjuntos de registros incluyen compatibilidad con la simultaneidad optimista.  En este ejemplo, como no se especifican 'If-Match' ni 'If-None-Match', siempre se crea el conjunto de registros.  Esta llamada sobrescribe cualquier conjunto de registros existente con el mismo nombre y tipo de registro de esta zona DNS.

```cs
// Create record set parameters
var recordSetParams = new RecordSet();
recordSetParams.TTL = 3600;

// Add records to the record set parameter object.  In this case, we'll add a record of type 'A'
recordSetParams.ARecords = new List<ARecord>();
recordSetParams.ARecords.Add(new ARecord("1.2.3.4"));

// Add metadata to the record set.  Similar to Azure Resource Manager tags, this is optional and you can add multiple metadata name/value pairs
recordSetParams.Metadata = new Dictionary<string, string>();
recordSetParams.Metadata.Add("user", "Mary");

// Create the actual record set in Azure DNS
// Note: no ETAG checks specified, will overwrite existing record set if one exists
var recordSet = await dnsClient.RecordSets.CreateOrUpdateAsync(resourceGroupName, zoneName, recordSetName, RecordType.A, recordSetParams);
```

## <a name="get-zones-and-record-sets"></a>Obtención de zonas y conjuntos de registros

Los métodos `DnsManagementClient.Zones.Get` y `DnsManagementClient.RecordSets.Get` recuperan zonas individuales y conjuntos de registros respectivamente. Los conjuntos de registros se identifican por su tipo y su nombre y por la zona y el grupo de recursos en que se encuentran. Las zonas se identifican por su nombre y el grupo de recursos en que se encuentran.

```cs
var recordSet = dnsClient.RecordSets.Get(resourceGroupName, zoneName, recordSetName, RecordType.A);
```

## <a name="update-an-existing-record-set"></a>Actualización de un conjunto de registros existente

Para actualizar un conjunto de registros de DNS existente, primero recupere el conjunto de registros, y luego actualice el contenido del conjunto de registros y envíe el cambio.  En este ejemplo, especificamos "Etag" del conjunto de registros recuperados en el parámetro 'If-Match'. La llamada produce error si una operación simultánea ha modificado mientras tanto el conjunto de registros.

```cs
var recordSet = dnsClient.RecordSets.Get(resourceGroupName, zoneName, recordSetName, RecordType.A);

// Add a new record to the local object.  Note that records in a record set must be unique/distinct
recordSet.ARecords.Add(new ARecord("5.6.7.8"));

// Update the record set in Azure DNS
// Note: ETAG check specified, update will be rejected if the record set has changed in the meantime
recordSet = await dnsClient.RecordSets.CreateOrUpdateAsync(resourceGroupName, zoneName, recordSetName, RecordType.A, recordSet, recordSet.Etag);
```

## <a name="list-zones-and-record-sets"></a>Enumeración de zonas y conjuntos de registros

Para mostrar las zonas, use los métodos *DnsManagementClient.Zones.List...* , que admiten el listado de todas las zonas de un grupo de recursos dado o todas las zonas de una suscripción de Azure concreta (entre grupos de recursos). Para mostrar conjuntos de registros, use los métodos *DnsManagementClient.RecordSets.List...* , que admiten el listado de todos los conjuntos de registros de una zona dada y solo los de un tipo específico.

Al mostrar zonas y conjuntos de registros, tenga en cuenta que los resultados pueden estar paginados.  En el ejemplo siguiente se muestra cómo iterar a través d las páginas de resultados. (Un tamaño de página artificialmente pequeño de '2' se usa para forzar la paginación; en la práctica, este parámetro debe omitirse y usarse el tamaño de página predeterminado).

```cs
// Note: in this demo, we'll use a very small page size (2 record sets) to demonstrate paging
// In practice, to improve performance you would use a large page size or just use the system default
int recordSets = 0;
var page = await dnsClient.RecordSets.ListAllInResourceGroupAsync(resourceGroupName, zoneName, "2");
recordSets += page.Count();

while (page.NextPageLink != null)
{
    page = await dnsClient.RecordSets.ListAllInResourceGroupNextAsync(page.NextPageLink);
    recordSets += page.Count();
}
```

## <a name="next-steps"></a>Pasos siguientes

Descargue el [proyecto de ejemplo del SDK de .NET de Azure DNS](https://www.microsoft.com/en-us/download/details.aspx?id=47268&WT.mc_id=DX_MVP4025064&e6b34bbe-475b-1abd-2c51-b5034bcdd6d2=True), que incluye más ejemplos de cómo usar el SDK de .NET de Azure DNS, además de ejemplos de otros tipos de registros de DNS.
