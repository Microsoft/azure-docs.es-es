---
title: Adición de Azure Storage (contenedor)
description: Aprenda a conectar un recurso compartido de red personalizado en una aplicación en contenedor de Azure App Service. Comparta archivos entre aplicaciones, administre el contenido estático de forma remota y acceda a él de forma local, etc.
author: msangapu-msft
ms.topic: article
ms.date: 7/01/2019
ms.author: msangapu
zone_pivot_groups: app-service-containers-windows-linux
ms.openlocfilehash: 8ca3688ad11bc7a694327990be22b8eccd8f8161
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962866"
---
# <a name="access-azure-storage-as-a-network-share-from-a-container-in-app-service"></a>Acceso a Azure Storage como un recurso compartido de red desde un contenedor de App Service

::: zone pivot="container-windows"

En esta guía se muestra cómo adjuntar archivos de Azure Storage como un recurso compartido de red a un contenedor de Windows en App Service. Solo se admiten [recursos compartidos de archivos de Azure Files](../storage/files/storage-how-to-use-files-cli.md) y [recursos compartidos de archivos Premium](../storage/files/storage-how-to-create-premium-fileshare.md). Entre las ventajas se incluye protección y portabilidad del contenido, acceso a varias aplicaciones y varios métodos de transferencia.

::: zone-end

::: zone pivot="container-linux"

En esta guía se muestra cómo adjuntar Azure Storage a una instancia de App Service en contenedor de Linux. Entre las ventajas se incluye protección y portabilidad del contenido, almacenamiento persistente, acceso a varias aplicaciones y varios métodos de transferencia.

::: zone-end

## <a name="prerequisites"></a>Prerrequisitos

::: zone pivot="container-windows"

- [Una aplicación de contenedor de Windows existente en Azure App Service](quickstart-custom-container.md)
- [Creación de un recurso compartido de Azure Files](../storage/files/storage-how-to-use-files-cli.md)
- [Carga de archivos en un recurso compartido de Azure Files](../storage/files/storage-files-deployment-guide.md)

::: zone-end

::: zone pivot="container-linux"

- Una [aplicación de App Service en Linux](index.yml) existente.
- Una [cuenta de Azure Storage](../storage/common/storage-account-create.md?tabs=azure-cli).
- Un [recurso compartido y un directorio de Azure](../storage/files/storage-how-to-use-files-cli.md).

::: zone-end

> [!NOTE]
> Azure Files es almacenamiento no predeterminado y se factura por separado, es decir, no se incluye con la aplicación web. No admite el uso de una configuración de firewall debido a limitaciones de infraestructura.
>

## <a name="limitations"></a>Limitaciones

::: zone pivot="container-windows"

- Azure Storage en App Service se encuentra **en versión preliminar** y **no es compatible** con los **escenarios de producción**.
- Actualmente, **no se admite** Azure Storage en App Service para incluir escenarios de código propios (aplicaciones de Windows no en contenedor).
- Azure Storage en App Service **no admite** el uso de la configuración del **firewall de Storage** debido a limitaciones de la infraestructura.
- Azure Storage con App Service le permite especificar **hasta cinco** puntos de montaje por aplicación.
- No se pude acceder a Azure Storage montado en una aplicación mediante los puntos de conexión de FTP/FTP de App Service. Use [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/).

::: zone-end

::: zone pivot="container-linux"

- Azure Storage en App Service está en **versión preliminar** para App Service en Linux y Web App for Containers. **No se admite** para **escenarios de producción**.
- Azure Storage en App Service admite el montaje de **contenedores de Azure Files** (lectura y escritura) y **contenedores de blobs de Azure** (solo lectura)
- Azure Storage en App Service **no admite** el uso de la configuración del **firewall de Storage** debido a limitaciones de la infraestructura.
- Azure Storage en App Service le permite especificar **hasta cinco** puntos de montaje por aplicación.
- No se pude acceder a Azure Storage montado en una aplicación mediante los puntos de conexión de FTP/FTP de App Service. Use [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/).

::: zone-end

## <a name="link-storage-to-your-app"></a>Vinculación del almacenamiento a la aplicación

::: zone pivot="container-windows"

Una vez que haya creado la [cuenta de Azure Storage, el recurso compartido y el directorio](#prerequisites), ahora puede configurar la aplicación con Azure Storage.

Para montar un recurso compartido de Azure Files en un directorio en la aplicación de App Service, use el comando [`az webapp config storage-account add`](/cli/azure/webapp/config/storage-account?view=azure-cli-latest#az-webapp-config-storage-account-add). El tipo de almacenamiento debe ser AzureFiles.

```azurecli
az webapp config storage-account add --resource-group <group-name> --name <app-name> --custom-id <custom-id> --storage-type AzureFiles --share-name <share-name> --account-name <storage-account-name> --access-key "<access-key>" --mount-path <mount-path-directory of form c:<directory name> >
```

Este paso debe realizarlo con cualquier otro directorio que quiera vincular a un recurso compartido de Azure Files.

::: zone-end

::: zone pivot="container-linux"

Una vez que haya creado la [cuenta de Azure Storage, el recurso compartido y el directorio](#prerequisites), ahora puede configurar la aplicación con Azure Storage.

Para montar una cuenta de almacenamiento en un directorio en la aplicación de App Service, use el comando [`az webapp config storage-account add`](/cli/azure/webapp/config/storage-account?view=azure-cli-latest#az-webapp-config-storage-account-add). El tipo de almacenamiento puede ser AzureBlob o AzureFiles. AzureFiles se usa en este ejemplo. La configuración de la ruta de acceso de montaje corresponde a la carpeta que desea montar desde Azure Storage. Al establecerla en '/' se monta la instancia completa de Azure Storage.


> [!CAUTION]
> El directorio especificado como ruta de acceso de montaje en la aplicación web debe estar vacío. Cualquier contenido almacenado en este directorio se eliminará cuando se agregue un montaje externo. Si va a migrar archivos de una aplicación existente, realice una copia de seguridad de la aplicación y de su contenido antes de comenzar.
>

```azurecli
az webapp config storage-account add --resource-group <group-name> --name <app-name> --custom-id <custom-id> --storage-type AzureFiles --share-name <share-name> --account-name <storage-account-name> --access-key "<access-key>" --mount-path <mount-path-directory>
```

Este paso debe realizarlo con cualquier otro directorio que quiera vincular a una cuenta de almacenamiento.

::: zone-end

## <a name="verify-linked-storage"></a>Comprobación del almacenamiento vinculado

Cuando el recurso compartido se vincula a la aplicación, puede ejecutar el siguiente comando para comprobarlo:

```azurecli
az webapp config storage-account list --resource-group <resource-group> --name <app-name>
```

## <a name="next-steps"></a>Pasos siguientes

::: zone pivot="container-windows"

- [Migración de software personalizado a Azure App Service mediante un contenedor personalizado](tutorial-custom-container.md?pivots=container-windows)

::: zone-end

::: zone pivot="container-linux"

- [Configurar un contenedor personalizado](configure-custom-container.md?pivots=platform-linux)

::: zone-end