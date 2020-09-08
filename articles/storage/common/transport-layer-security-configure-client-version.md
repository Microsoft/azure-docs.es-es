---
title: Configuración de Seguridad de la capa de transporte (TLS) para una aplicación cliente
titleSuffix: Azure Storage
description: Configure una aplicación cliente para que se comunique con Azure Storage mediante una versión mínima de Seguridad de la capa de transporte (TLS).
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/08/2020
ms.author: tamram
ms.reviewer: fryu
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 37b8c79df5b208feea185292fa09c323b64fa27d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89001814"
---
# <a name="configure-transport-layer-security-tls-for-a-client-application"></a>Configuración de Seguridad de la capa de transporte (TLS) para una aplicación cliente

Por motivos de seguridad, las cuentas de Azure Storage pueden exigir a los clientes que usen una versión mínima de Seguridad de la capa de transporte (TLS) para enviar solicitudes. Si el cliente usa una versión de TLS inferior a la versión mínima necesaria, se producirá un error en las llamadas a Azure Storage. Por ejemplo, si una cuenta de almacenamiento requiere TLS 1.2, las solicitudes enviadas por un cliente que use TLS 1.1 generarán un error.

En este artículo se describe cómo configurar una aplicación cliente para usar una versión determinada de TLS. Para información sobre cómo configurar una versión mínima necesaria de TLS para una cuenta de Azure Storage, consulte [Configuración de la versión mínima necesaria de Seguridad de la capa de transporte (TLS) para una cuenta de almacenamiento](transport-layer-security-configure-minimum-version.md).

## <a name="configure-the-client-tls-version"></a>Configuración de la versión de TLS del cliente

Para que un cliente envíe una solicitud con una versión determinada de TLS, el sistema operativo debe ser compatible con esa versión.

En los siguientes ejemplos se muestra cómo establecer la versión de TLS del cliente en la 1.2 desde PowerShell o .NET. La versión de .NET Framework que utiliza el cliente debe admitir TLS 1.2. Para más información, consulte [Compatibilidad con TLS 1.2](/dotnet/framework/network-programming/tls#support-for-tls-12).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

En el ejemplo siguiente se muestra cómo habilitar TLS 1.2 en el cliente de PowerShell.

```powershell
# Set the TLS version used by the PowerShell client to TLS 1.2.
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12;

# Create a new container.
$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context
New-AzStorageContainer -Name "sample-container" -Context $ctx
```

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

En el ejemplo siguiente se muestra cómo habilitar TLS 1.2 en un cliente .NET con la versión 12 de la biblioteca cliente de Azure Storage:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Networking.cs" id="Snippet_ConfigureTls12":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

En el ejemplo siguiente se muestra cómo habilitar TLS 1.2 en un cliente .NET con la versión 11 de la biblioteca cliente de Azure Storage:

```csharp
static void EnableTls12()
{
    // Enable TLS 1.2 before connecting to Azure Storage
    System.Net.ServicePointManager.SecurityProtocol = System.Net.SecurityProtocolType.Tls12;

    // Add your connection string here.
    string connectionString = "";

    // Connect to Azure Storage and create a new container.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(connectionString);
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    CloudBlobContainer container = blobClient.GetContainerReference("sample-container");
    container.CreateIfNotExists();
}
```

---

## <a name="verify-the-tls-version-used-by-a-client"></a>Comprobación de la versión de TLS utilizada por un cliente

Para comprobar que el cliente usó la versión especificada de TLS para enviar una solicitud, puede usar [Fiddler](https://www.telerik.com/fiddler) o una herramienta similar. Abra Fiddler para empezar a capturar el tráfico de red del cliente y, luego, ejecute uno de los ejemplos de la sección anterior. Examine el seguimiento de Fiddler para confirmar que se usó la versión correcta de TLS para enviar la solicitud, como se muestra en la siguiente imagen.

:::image type="content" source="media/transport-layer-security-configure-client-version/fiddler-trace-tls-version.png" alt-text="Captura de pantalla que muestra el seguimiento de Fiddler que indica la versión de TLS usada en la solicitud":::

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de la versión mínima necesaria de Seguridad de la capa de transporte (TLS) para una cuenta de almacenamiento](transport-layer-security-configure-minimum-version.md)
- [Recomendaciones de seguridad para Blob Storage](../blobs/security-recommendations.md)
