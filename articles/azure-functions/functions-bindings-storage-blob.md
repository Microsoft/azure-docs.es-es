---
title: Desencadenador y enlaces de Azure Blob Storage para Azure Functions
description: Obtenga información sobre cómo usar el desencadenador y los enlaces de Azure Blob Storage en Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/13/2020
ms.author: cshoe
ms.openlocfilehash: eead2d6ae2860bf06f1b878c1bb1692d74c06df6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87055970"
---
# <a name="azure-blob-storage-bindings-for-azure-functions-overview"></a>Introducción a los enlaces de Azure Blob Storage para Azure Functions

Azure Functions se integra con [Azure Storage](../storage/index.yml) mediante [desencadenadores y enlaces](./functions-triggers-bindings.md). La integración con Blob Storage permite crear funciones que reaccionan a los cambios en los datos del blob, así como a los valores de lectura y escritura.

| Acción | Tipo |
|---------|---------|
| Ejecución de una función cuando cambian los datos de Blob Storage | [Desencadenador](./functions-bindings-storage-blob-trigger.md) |
| Lectura de datos de Blob Storage en una función | [Enlace de entrada](./functions-bindings-storage-blob-input.md) |
| Escritura de datos de almacenamiento de blobs desde una función |[Enlace de salida](./functions-bindings-storage-blob-output.md) |

## <a name="add-to-your-functions-app"></a>Adición a la aplicación de Functions

### <a name="functions-2x-and-higher"></a>Functions 2.x y superiores

Para trabajar con el desencadenador y los enlaces, es necesario hacer referencia al paquete adecuado. En las bibliotecas de clases de .NET se usa el paquete NuGet, mientras que en los demás tipos de aplicaciones se emplea el conjunto de extensiones.

| Idioma                                        | Agregar mediante...                                   | Observaciones 
|-------------------------------------------------|---------------------------------------------|-------------|
| C#                                              | Instalación del [paquete NuGet], versión 3.x | |
| Script de C#, Java, JavaScript, Python, PowerShell | Registro de [conjunto de extensiones]          | Se recomienda usar la [extensión Azure Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack) con Visual Studio Code. |
| Script de C# (solo en línea en Azure Portal)         | Adición de un enlace                            | Para actualizar extensiones de enlace existentes sin tener que volver a publicar la aplicación de funciones, vea [Actualización de las extensiones]. |

[core tools]: ./functions-run-local.md
[conjunto de extensiones]: ./functions-bindings-register.md#extension-bundles
[Paquete NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage
[Actualización de las extensiones]: ./install-update-binding-extensions-manual.md
[Azure Tools extension]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="functions-1x"></a>Functions 1.x

Las aplicaciones de Functions 1.x tienen automáticamente una referencia al paquete NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs), versión 2.x.

[!INCLUDE [functions-storage-sdk-version](../../includes/functions-storage-sdk-version.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Ejecución de una función cuando cambian los datos de Blob Storage](./functions-bindings-storage-blob-trigger.md)
- [Lectura de datos de Blob Storage cuando se ejecuta una función](./functions-bindings-storage-blob-input.md)
- [Escritura de datos de Blob Storage desde una función](./functions-bindings-storage-blob-output.md)
