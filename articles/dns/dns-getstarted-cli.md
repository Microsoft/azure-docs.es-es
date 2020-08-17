---
title: 'Inicio rápido: Creación de una zona y registro de Azure DNS - CLI de Azure'
titleSuffix: Azure DNS
description: 'Inicio rápido: Aprender a crear una zona y un registro DNS en Azure DNS. Esta es una guía paso a paso para crear y administrar su primera zona y su primer registro DNS con la CLI de Azure.'
services: dns
author: rohinkoul
ms.service: dns
ms.topic: quickstart
ms.date: 3/11/2019
ms.author: rohink
ms.custom: devx-track-azurecli
ms.openlocfilehash: a15209351f95e44c7cf10993513e03ba35b53d27
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494373"
---
# <a name="quickstart-create-an-azure-dns-zone-and-record-using-azure-cli"></a>Inicio rápido: Creación de una zona y un registro de Azure DNS mediante la CLI de Azure

Este artículo lo guía por los pasos necesarios para crear su primera zona y su primer registro DNS mediante la CLI de Azure, que está disponible para Windows, Mac y Linux. También puede llevar a cabo estos pasos con [Azure Portal](dns-getstarted-portal.md) o [Azure PowerShell](dns-getstarted-powershell.md).

Una zona DNS se usa para hospedar los registros DNS de un dominio concreto. Para iniciar el hospedaje de su dominio en DNS de Azure, debe crear una zona DNS para ese nombre de dominio. Cada registro DNS del dominio se crea luego en esta zona DNS. Finalmente, para publicar la zona DNS en Internet, debe configurar los servidores de nombres para el dominio. A continuación, se describen cada uno de estos pasos.

Azure DNS también admite zonas DNS privadas. Para más información acerca de las zonas DNS privadas, consulte [Using Azure DNS for private domains](private-dns-overview.md) (Uso de Azure DNS para dominios privados). Para ver un ejemplo de cómo crear una zona DNS privada, consulte [Get started with Azure DNS Private Zones using CLI](./private-dns-getstarted-cli.md) (Introducción al uso de la CLI de Azure 2.0 con las zonas privadas de Azure DNS).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-the-resource-group"></a>Creación del grupo de recursos

Antes de crear la zona DNS, cree un grupo de recursos que la contenga:

```azurecli
az group create --name MyResourceGroup --location "East US"
```

## <a name="create-a-dns-zone"></a>Creación de una zona DNS

Una zona DNS se crea con el comando `az network dns zone create` . Para ver la ayuda sobre este comando, escriba `az network dns zone create -h`.

En el ejemplo siguiente, se crea una zona DNS llamada *contoso.xyz* en el grupo de recursos *MyResourceGroup*. Utilice el ejemplo para crear una zona DNS, sustituyendo los valores por los suyos.

```azurecli
az network dns zone create -g MyResourceGroup -n contoso.xyz
```

## <a name="create-a-dns-record"></a>Creación de un registro de DNS

Para crear un registro de DNS, use el comando `az network dns record-set [record type] add-record`. Para obtener ayuda sobre los registros A, consulte `azure network dns record-set A add-record -h`.

En el ejemplo siguiente se crea un registro con el nombre relativo "www" en la zona DNS "contoso.xyz" del grupo de recursos "MyResourceGroup". El nombre completo del conjunto de registros es "www.contoso.xyz". El tipo de registro es "A", con la dirección IP "10.10.10.10" y un valor predeterminado TTL de 3600 segundos (1 hora).

```azurecli
az network dns record-set a add-record -g MyResourceGroup -z contoso.xyz -n www -a 10.10.10.10
```

## <a name="view-records"></a>Visualización de los registros

Para enumerar los registros DNS de su zona, ejecute:

```azurecli
az network dns record-set list -g MyResourceGroup -z contoso.xyz
```

## <a name="test-the-name-resolution"></a>Probar la resolución de nombres

Ahora que tiene una zona DNS de prueba, con un registro "A" de prueba, puede probar la resolución de nombres con una herramienta llamada *nslookup*. 

**Para probar la resolución de nombres DNS:**

1. Ejecute el siguiente cmdlet para obtener la lista de servidores de nombres para la zona:

   ```azurecli
   az network dns record-set ns show --resource-group MyResourceGroup --zone-name contoso.xyz --name @
   ```

1. Copie uno de los nombres de servidor de la salida del paso anterior.

1. Abra un símbolo del sistema y ejecute el comando siguiente:

   ```
   nslookup www.contoso.xyz <name server name>
   ```

   Por ejemplo:

   ```
   nslookup www.contoso.xyz ns1-08.azure-dns.com.
   ```

   Debe ver algo parecido a la pantalla siguiente:

   ![nslookup](media/dns-getstarted-portal/nslookup.PNG)

El nombre de host **www\.contoso.xyz** se resuelve en **10.10.10.10**, que es como lo ha configurado. Este resultado confirma que la resolución de nombres funciona correctamente.

## <a name="delete-all-resources"></a>Eliminación de todos los recursos

Cuando ya no los necesite, puede eliminar todos los recursos creados en este inicio rápido mediante la eliminación del grupo de recursos:

```azurecli
az group delete --name MyResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado su primera zona y su primer registro DNS mediante la CLI de Azure, puede crear registros para una aplicación web de un dominio personalizado.

> [!div class="nextstepaction"]
> [Creación de registros DNS para una aplicación web en un dominio personalizado](./dns-web-sites-custom-domain.md)
