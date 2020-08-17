---
title: Administración de zonas DNS en Azure DNS - CLI de Azure | Microsoft Docs
description: Puede administrar zonas DNS con CLI de Azure. Este artículo muestra cómo actualizar, eliminar y crear zonas DNS en Azure DNS.
services: dns
documentationcenter: na
author: rohinkoul
ms.service: dns
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/27/2017
ms.author: rohink
ms.custom: devx-track-azurecli
ms.openlocfilehash: ff1abd48282a4ec9278d7182fea286178b5bd3ca
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495920"
---
# <a name="how-to-manage-dns-zones-in-azure-dns-using-the-azure-cli"></a>Cómo administrar zonas DNS en Azure DNS con la CLI de Azure

> [!div class="op_single_selector"]
> * [Portal](dns-operations-dnszones-portal.md)
> * [PowerShell](dns-operations-dnszones.md)
> * [CLI de Azure](dns-operations-dnszones-cli.md)


Esta guía muestra cómo administrar las zonas DNS mediante la CLI de Azure multiplataforma, que está disponible para Windows, Mac y Linux. También puede administrar sus zonas DNS mediante [Azure PowerShell](dns-operations-dnszones.md) o Azure Portal.

Esta guía trata específicamente con las zonas DNS públicas. Para más información acerca del uso de CLI de Azure para administrar zonas privadas en Azure DNS, consulte [Creación de una zona privada de Azure DNS mediante la CLI de Azure](private-dns-getstarted-cli.md).

## <a name="introduction"></a>Introducción

[!INCLUDE [dns-create-zone-about](../../includes/dns-create-zone-about-include.md)]

## <a name="set-up-azure-cli-for-azure-dns"></a>Configuración de la CLI de Azure para DNS de Azure

### <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar con la configuración, compruebe que dispone de los elementos siguientes.

* Suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).

* Instale la versión más reciente de la CLI de Azure, disponible para Windows, Linux o Mac. Puede consultar mas información disponible en [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-az-cli2).

### <a name="sign-in-to-your-azure-account"></a>Inicio de sesión en la cuenta de Azure.

Abra una ventana de consola y autentíquese con sus credenciales. Para más información, consulte [Inicio de sesión en Azure desde la CLI de Azure](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest).

```
az login
```

### <a name="select-the-subscription"></a>Selección de la suscripción

Compruebe las suscripciones para la cuenta.

```
az account list
```

Elección de la suscripción de Azure que se va a usar.

```azurecli
az account set --subscription "subscription name"
```

### <a name="optional-to-installuse-azure-dns-private-zones-feature"></a>Opcional: Para instalar o usar la característica Azure DNS Private Zones
La característica Azure DNS Private Zones está disponible en una extensión de la CLI de Azure. Instale la extensión de la CLI de Azure de "dns" 
```
az extension add --name dns
``` 

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Azure Resource Manager requiere que todos los grupos de recursos especifiquen una ubicación. Esta se utiliza como ubicación predeterminada para los recursos de ese grupo de recursos. Sin embargo, puesto que todos los recursos DNS son globales y no regionales, la elección de la ubicación del grupo de recursos no incide en DNS de Azure.

Puede omitir este paso si utiliza un grupo de recursos existente.

```azurecli
az group create --name myresourcegroup --location "West US"
```

## <a name="getting-help"></a>Ayuda

Los comandos de la CLI de Azure relacionados con Azure DNS comienzan por `az network dns`. La ayuda está disponible para cada comando con la opción `--help` (formato abreviado `-h`).  Por ejemplo:

```azurecli
az network dns --help
az network dns zone --help
az network dns zone create --help
```

## <a name="create-a-dns-zone"></a>Creación de una zona DNS

Una zona DNS se crea con el comando `az network dns zone create` . Para obtener ayuda, consulte `az network dns zone create -h`.

En el ejemplo siguiente, se crea una zona DNS llamada *contoso.com* en el grupo de recursos *MyResourceGroup*:

```azurecli
az network dns zone create --resource-group MyResourceGroup --name contoso.com
```

### <a name="to-create-a-dns-zone-with-tags"></a>Para crear una zona DNS con etiquetas.

En el ejemplo siguiente se muestra cómo crear una zona DNS con dos [etiquetas de Azure Resource Manager](dns-zones-records.md#tags), *project = demo* y *env = test*, con el parámetro `--tags` (forma abreviada `-t`):

```azurecli
az network dns zone create --resource-group MyResourceGroup --name contoso.com --tags "project=demo" "env=test"
```

## <a name="get-a-dns-zone"></a>Recuperación de una zona DNS

Para recuperar una zona DNS, use `az network dns zone show`. Para obtener ayuda, consulte `az network dns zone show --help`.

El ejemplo siguiente devuelve la zona DNS *contoso.com* y sus datos asociados del grupo de recursos *MyResourceGroup*. 

```azurecli
az network dns zone show --resource-group myresourcegroup --name contoso.com
```

El ejemplo siguiente es la respuesta.

```json
{
  "etag": "00000002-0000-0000-3d4d-64aa3689d201",
  "id": "/subscriptions/147a22e9-2356-4e56-b3de-1f5842ae4a3b/resourceGroups/myresourcegroup/providers/Microsoft.Network/dnszones/contoso.com",
  "location": "global",
  "maxNumberOfRecordSets": 5000,
  "name": "contoso.com",
  "nameServers": [
    "ns1-04.azure-dns.com.",
    "ns2-04.azure-dns.net.",
    "ns3-04.azure-dns.org.",
    "ns4-04.azure-dns.info."
  ],
  "numberOfRecordSets": 4,
  "resourceGroup": "myresourcegroup",
  "tags": {},
  "type": "Microsoft.Network/dnszones"
}
```

Tenga en cuenta que `az network dns zone show` no devuelve los registros DNS. Para enumerar los registros DNS, utilice `az network dns record-set list`.


## <a name="list-dns-zones"></a>Enumeración de zonas DNS

Para enumerar las zonas DNS, utilice `az network dns zone list`. Para obtener ayuda, consulte `az network dns zone list --help`.

Si se especifica el grupo de recursos, se enumeran solo esas zonas dentro del grupo de recursos:

```azurecli
az network dns zone list --resource-group MyResourceGroup
```

Si se omite el grupo de recursos, se enumeran todas las zonas en la suscripción:

```azurecli
az network dns zone list 
```

## <a name="update-a-dns-zone"></a>Actualización de una zona DNS

Los cambios en los recursos de una zona DNS se pueden realizar con `az network dns zone update`. Para obtener ayuda, consulte `az network dns zone update --help`.

Este comando no actualiza los conjuntos de registros DNS de la zona (consulte [Administración de registros DNS](dns-operations-recordsets-cli.md)). Solo se utiliza para actualizar las propiedades de los recursos de la zona. Estas propiedades están actualmente limitadas a las ["etiquetas" Azure Resource Manager ](dns-zones-records.md#tags) para los recursos de la zona.

En el ejemplo siguiente se muestra cómo actualizar las etiquetas en una zona DNS. Las etiquetas existentes se reemplazan por el valor especificado.

```azurecli
az network dns zone update --resource-group myresourcegroup --name contoso.com --set tags.team=support
```

## <a name="delete-a-dns-zone"></a>Eliminar una zona DNS

Las zonas DNS se pueden eliminar mediante `az network dns zone delete`. Para obtener ayuda, consulte `az network dns zone delete --help`.

> [!NOTE]
> Si se elimina una zona DNS, también se eliminan todos los registros DNS dentro de la zona. No se puede deshacer esta operación. Si la zona DNS está en uso, los servicios con la zona provocarán un error cuando se elimina la zona.
>
>Para protegerse contra la eliminación accidental de zona, consulte [Proteger registros y zonas DNS](dns-protect-zones-recordsets.md).

Este comando solicita confirmación. El modificador `--yes` opcional suprime este mensaje.

En el ejemplo siguiente se muestra cómo eliminar la zona *contoso.com* del grupo de recursos *MyResourceGroup*.

```azurecli
az network dns zone delete --resource-group myresourcegroup --name contoso.com
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía se explica cómo [administrar conjuntos de registros y registros](dns-getstarted-create-recordset-cli.md) en la zona DNS.

Aprenda a cómo [delegar el dominio a Azure DNS](dns-domain-delegation.md).

