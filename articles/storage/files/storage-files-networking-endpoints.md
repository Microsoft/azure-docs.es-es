---
title: Configuración de puntos de conexión de red de Azure Files | Microsoft Docs
description: Una introducción a las opciones de red para Azure Files.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 08/17/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: a38528e32061f57e3239ef4be26cdd437f4f8746
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078599"
---
# <a name="configuring-azure-files-network-endpoints"></a>Configuración de puntos de conexión de red de Azure Files

Azure Files proporciona dos tipos principales de puntos de conexión para el acceso a los recursos compartidos de archivos de Azure: 
- Puntos de conexión públicos, que tienen una dirección IP pública y a los que se puede acceder desde cualquier parte del mundo.
- Puntos de conexión privados, que existen dentro de una red virtual y tienen una dirección IP privada en el espacio de direcciones de esa red virtual.

Los puntos de conexión públicos y privados existen en la cuenta de almacenamiento de Azure. Una cuenta de almacenamiento es una construcción de administración que representa un grupo compartido de almacenamiento en el que puede implementar varios recursos compartidos de archivos u otros recursos de almacenamiento, como contenedores de blobs o colas.

Este artículo se centra en cómo configurar los puntos de conexión de una cuenta de almacenamiento para acceder directamente al recurso compartido de archivos de Azure. La mayoría de los detalles proporcionados en este documento también se aplican al modo en que Azure File Sync interopera con puntos de conexión públicos y privados de la cuenta de almacenamiento; sin embargo, para más información sobre las consideraciones de redes para una implementación de Azure File Sync, consulte [Configuración del proxy y el firewall de Azure File Sync](storage-sync-files-firewall-and-proxy.md).

Se recomienda leer [Consideraciones de redes para Azure Files](storage-files-networking-overview.md) antes de pasar a esta guía de procedimientos.

## <a name="prerequisites"></a>Requisitos previos

- En este artículo se supone que ya ha creado una suscripción a Azure. Si todavía no tiene una suscripción, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- En este artículo se supone que ya ha creado un recurso compartido de archivos de Azure en una cuenta de almacenamiento a la que le gustaría conectarse desde el entorno local. Para aprender a crear un recurso compartido de archivos de Azure, consulte [Creación de un recurso compartido de archivos de Azure](storage-how-to-create-file-share.md).
- Si planea usar Azure PowerShell, [instale la versión más reciente](https://docs.microsoft.com/powershell/azure/install-az-ps).
- Si planea usar la CLI de Azure, [instale la versión más reciente](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="endpoint-configurations"></a>Configuraciones de punto de conexión

Puede configurar los puntos de conexión para restringir el acceso de red a su cuenta de almacenamiento. Existen dos enfoques para restringir el acceso de una cuenta de almacenamiento a una red virtual:

- [Cree uno o varios puntos de conexión privados para la cuenta de almacenamiento](#create-a-private-endpoint) y restrinja todo el acceso al punto de conexión público. De esta forma se garantiza que solo el tráfico que se origina en las redes virtuales deseadas pueda acceder a los recursos compartidos de archivos de Azure dentro de la cuenta de almacenamiento.
- [Restrinja el punto de conexión público a una o más redes virtuales](#restrict-public-endpoint-access). Para ello, se usa una funcionalidad de la red virtual llamada *puntos de conexión de servicio*. Al restringir el tráfico a una cuenta de almacenamiento a través de un punto de conexión de servicio, sigue teniendo acceso a la cuenta de almacenamiento a través de la dirección IP pública, pero el acceso solo es posible desde las ubicaciones especificadas en la configuración.

### <a name="create-a-private-endpoint"></a>Creación de un punto de conexión privado

La creación de un punto de conexión privado para la cuenta de almacenamiento hará que se implementen los siguientes recursos de Azure:

- **Punto de conexión privado**: un recurso de Azure que representa el punto de conexión privado de la cuenta de almacenamiento. Puede considerarlo como un recurso que conecta una cuenta de almacenamiento y una interfaz de red.
- **Interfaz de red (NIC)** : la interfaz de red que mantiene una dirección IP privada dentro de la red virtual o subred especificadas. Es el mismo recurso exacto que se implementa cuando se implementa una máquina virtual, pero en lugar de asignarlo a una máquina virtual, es propiedad del punto de conexión privado.
- **Zona DNS privada**: si nunca ha implementado antes un punto de conexión privado para esta red virtual, se implementará una nueva zona DNS privada para ella. También se creará en esta zona DNS un registro D de DNS para la cuenta de almacenamiento. Si ya ha implementado un punto de conexión privado en esta red virtual, se agregará a la zona DNS existente un nuevo registro D para la cuenta de almacenamiento. La implementación de una zona DNS es opcional, sin embargo, es muy recomendable y necesaria si va a montar los recursos compartidos de archivos de Azure con una entidad de servicio de AD o mediante la API FileREST.

> [!Note]  
> En este artículo se usa el sufijo DNS de la cuenta de almacenamiento para las regiones públicas de Azure `core.windows.net`. Este comentario también se aplica a nubes soberanas de Azure, como la nube de Azure US Government y la nube de Azure China; simplemente sustituya los sufijos adecuados en su entorno. 

# <a name="portal"></a>[Portal](#tab/azure-portal)
[!INCLUDE [storage-files-networking-endpoints-private-portal](../../../includes/storage-files-networking-endpoints-private-portal.md)]

Si tiene una máquina virtual dentro de la red virtual o ha configurado el reenvío de DNS tal como se describe en [Configuración del reenvío de DNS para Azure Files](storage-files-networking-dns.md), puede comprobar que el punto de conexión privado se ha configurado correctamente mediante la ejecución de los siguientes comandos desde PowerShell, la línea de comandos o el terminal (sirve para Windows, Linux o macOS). Debe reemplazar `<storage-account-name>` por el nombre adecuado de la cuenta de almacenamiento:

```
nslookup <storage-account-name>.file.core.windows.net
```

Si todo ha funcionado correctamente, verá la siguiente salida, donde `192.168.0.5` es la dirección IP privada del punto de conexión privado de la red virtual (salida mostrada para Windows):

```Output
Server:  UnKnown
Address:  10.2.4.4

Non-authoritative answer:
Name:    storageaccount.privatelink.file.core.windows.net
Address:  192.168.0.5
Aliases:  storageaccount.file.core.windows.net
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!INCLUDE [storage-files-networking-endpoints-private-powershell](../../../includes/storage-files-networking-endpoints-private-powershell.md)]

Si tiene una máquina virtual dentro de la red virtual o ha configurado el reenvío de DNS tal como se describe en [Configuración del reenvío de DNS para Azure Files](storage-files-networking-dns.md), puede comprobar que el punto de conexión privado se ha configurado correctamente mediante los siguientes comandos:

```PowerShell
$storageAccountHostName = [System.Uri]::new($storageAccount.PrimaryEndpoints.file) | `
    Select-Object -ExpandProperty Host

Resolve-DnsName -Name $storageAccountHostName
```

Si todo ha funcionado correctamente, verá la siguiente salida, donde `192.168.0.5` es la dirección IP privada del punto de conexión privado de la red virtual:

```Output
Name                             Type   TTL   Section    NameHost
----                             ----   ---   -------    --------
storageaccount.file.core.windows CNAME  60    Answer     storageaccount.privatelink.file.core.windows.net
.net

Name       : storageaccount.privatelink.file.core.windows.net
QueryType  : A
TTL        : 600
Section    : Answer
IP4Address : 192.168.0.5
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
[!INCLUDE [storage-files-networking-endpoints-private-cli](../../../includes/storage-files-networking-endpoints-private-cli.md)]

Si tiene una máquina virtual dentro de la red virtual o ha configurado el reenvío de DNS tal como se describe en [Configuración del reenvío de DNS para Azure Files](storage-files-networking-dns.md), puede comprobar que el punto de conexión privado se ha configurado correctamente mediante los siguientes comandos:

```bash
httpEndpoint=$(az storage account show \
        --resource-group $storageAccountResourceGroupName \
        --name $storageAccountName \
        --query "primaryEndpoints.file" | \
    tr -d '"')

hostName=$(echo $httpEndpoint | cut -c7-$(expr length $httpEndpoint) | tr -d "/")
nslookup $hostName
```

Si todo ha funcionado correctamente, debería ver la siguiente salida, donde `192.168.0.5` es la dirección IP privada del punto de conexión privado de la red virtual. Debe seguir usando storageaccount.file.core.windows.net para montar el recurso compartido de archivos, en lugar de la ruta de acceso de `privatelink`.

```Output
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
storageaccount.file.core.windows.net      canonical name = storageaccount.privatelink.file.core.windows.net.
Name:   storageaccount.privatelink.file.core.windows.net
Address: 192.168.0.5
```

---

### <a name="restrict-public-endpoint-access"></a>Restricción del acceso al punto de conexión público

Para limitar el acceso al punto de conexión público es necesario deshabilitar el acceso general al punto de conexión público. Deshabilitar el acceso al punto de conexión público no afecta a los puntos de conexión privados. Una vez deshabilitado el punto de conexión público, puede seleccionar redes o direcciones IP específicas que puedan seguir accediendo a él. En general, la mayoría de las directivas de firewall de una cuenta de almacenamiento restringen el acceso de red a una o varias redes virtuales.

#### <a name="disable-access-to-the-public-endpoint"></a>Deshabilitación del acceso al punto de conexión público

Si el acceso al punto de conexión público está deshabilitado, aún se puede acceder a la cuenta de almacenamiento a través de los puntos de conexión privados. De lo contrario, se rechazarán las solicitudes válidas al punto de conexión público de la cuenta de almacenamiento, a menos que provengan de [un origen permitido específicamente](#restrict-access-to-the-public-endpoint-to-specific-virtual-networks). 

# <a name="portal"></a>[Portal](#tab/azure-portal)
[!INCLUDE [storage-files-networking-endpoints-public-disable-portal](../../../includes/storage-files-networking-endpoints-public-disable-portal.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!INCLUDE [storage-files-networking-endpoints-public-disable-powershell](../../../includes/storage-files-networking-endpoints-public-disable-powershell.md)]

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
[!INCLUDE [storage-files-networking-endpoints-public-disable-cli](../../../includes/storage-files-networking-endpoints-public-disable-cli.md)]

---

#### <a name="restrict-access-to-the-public-endpoint-to-specific-virtual-networks"></a>Restricción del acceso al punto de conexión público a redes virtuales específicas

Al restringir la cuenta de almacenamiento a redes virtuales específicas, se permiten solicitudes al punto de conexión público en las redes virtuales especificadas. Para ello, se usa una funcionalidad de la red virtual llamada *puntos de conexión de servicio*. Esta funcionalidad se puede usar con o sin puntos de conexión privados.

# <a name="portal"></a>[Portal](#tab/azure-portal)
[!INCLUDE [storage-files-networking-endpoints-public-restrict-portal](../../../includes/storage-files-networking-endpoints-public-restrict-portal.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!INCLUDE [storage-files-networking-endpoints-public-restrict-powershell](../../../includes/storage-files-networking-endpoints-public-restrict-powershell.md)]

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
[!INCLUDE [storage-files-networking-endpoints-public-restrict-cli](../../../includes/storage-files-networking-endpoints-public-restrict-cli.md)]

---

## <a name="see-also"></a>Consulte también

- [Consideraciones de redes para Azure Files](storage-files-networking-overview.md)
- [Configuración del reenvío de DNS para Azure Files](storage-files-networking-dns.md)
- [Configuración de VPN S2S para Azure Files](storage-files-configure-s2s-vpn.md)
