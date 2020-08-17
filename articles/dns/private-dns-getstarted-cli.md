---
title: 'Inicio rápido: Creación de una zona DNS privada de Azure mediante la CLI de Azure'
description: En este inicio rápido se crean y se prueban una zona DNS privada y un registro en Azure DNS. Esta es una guía detallada para crear y administrar la primera zona DNS privada y el primer registro mediante la CLI de Azure.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: quickstart
ms.date: 10/05/2019
ms.author: rohink
ms.custom: devx-track-azurecli
ms.openlocfilehash: 8fdf78c0a3dd2f7a130d827751ce93c5539575df
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502965"
---
# <a name="quickstart-create-an-azure-private-dns-zone-using-the-azure-cli"></a>Inicio rápido: Creación de una zona DNS privada de Azure mediante la CLI de Azure

Este inicio rápido le guiará por los pasos necesarios para crear una zona DNS privada y un registro con la CLI de Azure.

Una zona DNS se usa para hospedar los registros DNS de un dominio concreto. Para iniciar el hospedaje de su dominio en DNS de Azure, debe crear una zona DNS para ese nombre de dominio. Cada registro DNS del dominio se crea luego en esta zona DNS. Para publicar una zona DNS privada en la red virtual, especifique la lista de redes virtuales que pueden resolver registros en ella.  Se denominan redes virtuales *vinculadas*. Cuando se habilita el registro automático, Azure DNS también actualiza los registros de zona cuando se crea una máquina virtual, se cambia su dirección IP o se elimina.

En esta guía de inicio rápido, ha aprendido a hacer lo siguiente:

> [!div class="checklist"]
> * Crear una zona DNS privada
> * Creación de máquinas virtuales de prueba
> * Creación de un registro de DNS adicional
> * Prueba de la zona privada

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Si lo prefiere, puede completar este inicio rápido mediante [Azure PowerShell](private-dns-getstarted-powershell.md).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-the-resource-group"></a>Creación del grupo de recursos

En primer lugar, cree un grupo de recursos que contenga la zona DNS: 

```azurecli
az group create --name MyAzureResourceGroup --location "East US"
```

## <a name="create-a-private-dns-zone"></a>Crear una zona DNS privada

En el ejemplo siguiente se crea una red virtual denominada **myAzureVNet**. Luego, se crea una zona DNS denominada **private.contoso.com** en el grupo de recursos **MyAzureResourceGroup**, se vincula esa zona DNS a la red virtual **MyAzureVnet** y se habilita el registro automático.

```azurecli
az network vnet create \
  --name myAzureVNet \
  --resource-group MyAzureResourceGroup \
  --location eastus \
  --address-prefix 10.2.0.0/16 \
  --subnet-name backendSubnet \
  --subnet-prefixes 10.2.0.0/24

az network private-dns zone create -g MyAzureResourceGroup \
   -n private.contoso.com

az network private-dns link vnet create -g MyAzureResourceGroup -n MyDNSLink \
   -z private.contoso.com -v myAzureVNet -e true
```

Si quiere crear una zona únicamente para la resolución de nombres (sin registro de nombre de host automático), puede usar el parámetro `-e false`.

### <a name="list-dns-private-zones"></a>Listado de zonas privadas de DNS

Para enumerar las zonas DNS, utilice `az network private-dns zone list`. Para obtener ayuda, consulte `az network dns zone list --help`.

Si se especifica el grupo de recursos, se enumeran solo esas zonas dentro del grupo de recursos:

```azurecli
az network private-dns zone list \
  -g MyAzureResourceGroup
```

Si se omite el grupo de recursos, se enumeran todas las zonas en la suscripción:

```azurecli
az network private-dns zone list 
```

## <a name="create-the-test-virtual-machines"></a>Creación de las máquinas virtuales de prueba

Ahora, cree dos máquinas virtuales para poder probar su zona DNS privada:

```azurecli
az vm create \
 -n myVM01 \
 --admin-username AzureAdmin \
 -g MyAzureResourceGroup \
 -l eastus \
 --subnet backendSubnet \
 --vnet-name myAzureVnet \
 --nsg NSG01 \
 --nsg-rule RDP \
 --image win2016datacenter

az vm create \
 -n myVM02 \
 --admin-username AzureAdmin \
 -g MyAzureResourceGroup \
 -l eastus \
 --subnet backendSubnet \
 --vnet-name myAzureVnet \
 --nsg NSG01 \
 --nsg-rule RDP \
 --image win2016datacenter
```

Esta operación tardará algunos minutos en completarse.

## <a name="create-an-additional-dns-record"></a>Creación de un registro de DNS adicional

Para crear un registro de DNS, use el comando `az network private-dns record-set [record type] add-record`. Para obtener ayuda para agregar registros D, por ejemplo, vea `az network private-dns record-set A add-record --help`.

 En el ejemplo siguiente se crea un registro con el nombre relativo **db** en la zona DNS **private.contoso.com** del grupo de recursos **MyAzureResourceGroup**. El nombre completo del conjunto de registros es **db.private.contoso.com**. El tipo de registro es "D" y la dirección IP "10.2.0.4".

```azurecli
az network private-dns record-set a add-record \
  -g MyAzureResourceGroup \
  -z private.contoso.com \
  -n db \
  -a 10.2.0.4
```

### <a name="view-dns-records"></a>Visualización de registros DNS

Para enumerar los registros DNS de su zona, ejecute:

```azurecli
az network private-dns record-set list \
  -g MyAzureResourceGroup \
  -z private.contoso.com
```

## <a name="test-the-private-zone"></a>Prueba de la zona privada

Ya puede probar la resolución de nombres de la zona privada **private.contoso.com**.

### <a name="configure-vms-to-allow-inbound-icmp"></a>Configuración de máquinas virtuales para permitir ICMP de entrada

Puede usar el comando ping para probar la resolución de nombres. Por tanto, configure el firewall en ambas máquinas virtuales para permitir paquetes ICMP entrantes.

1. Conéctese a myVM01 y abra una ventana de Windows PowerShell con privilegios de administrador.
2. Ejecute el siguiente comando:

   ```powershell
   New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
   ```

Repita la operación con myVM02.

### <a name="ping-the-vms-by-name"></a>Realización de ping en las máquinas virtuales por nombre

1. En el símbolo del sistema de Windows PowerShell de myVM02, haga ping a myVM01 con el nombre de host registrado automáticamente:

   ```
   ping myVM01.private.contoso.com
   ```

   La salida es similar a esta:

   ```
   PS C:\> ping myvm01.private.contoso.com

   Pinging myvm01.private.contoso.com [10.2.0.4] with 32 bytes of data:
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time=1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128

   Ping statistics for 10.2.0.4:
       Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
   Approximate round trip times in milli-seconds:
       Minimum = 0ms, Maximum = 1ms, Average = 0ms
   PS C:\>
   ```

2. Ahora haga ping en el nombre de la **base de datos** que creó anteriormente:

   ```
   ping db.private.contoso.com
   ```

   La salida es similar a esta:

   ```
   PS C:\> ping db.private.contoso.com

   Pinging db.private.contoso.com [10.2.0.4] with 32 bytes of data:
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128

   Ping statistics for 10.2.0.4:
       Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
   Approximate round trip times in milli-seconds:
       Minimum = 0ms, Maximum = 0ms, Average = 0ms
   PS C:\>
   ```

## <a name="delete-all-resources"></a>Eliminación de todos los recursos

Cuando ya no lo necesite, elimine el grupo de recursos **MyAzureResourceGroup** para eliminar los recursos que ha creado en este inicio rápido.

```azurecli
az group delete --name MyAzureResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Escenarios de Azure DNS Private Zones](private-dns-scenarios.md)

