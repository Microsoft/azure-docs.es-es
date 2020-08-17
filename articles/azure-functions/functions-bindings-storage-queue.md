---
title: Introducción al desencadenador y enlaces de Azure Queue Storage para Azure Functions
description: Obtenga información acerca de cómo usar enlaces de salida y desencadenador de Azure Queue Storage en Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/18/2020
ms.author: cshoe
ms.custom: cc996988-fb4f-47
ms.openlocfilehash: 71321fa631bafde5b1dbaac2d165cb68fd8116d4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87055809"
---
# <a name="azure-queue-storage-trigger-and-bindings-for-azure-functions-overview"></a>Introducción al desencadenador y enlaces de Azure Queue Storage para Azure Functions

Azure Functions puede ejecutarse a medida que se crean nuevos mensajes de Azure Queue Storage y puede escribir mensajes de cola dentro de una función.

| Acción | Tipo |
|---------|---------|
| Ejecución de una función cuando cambian los datos de Queue Storage | [Desencadenador](./functions-bindings-storage-queue-trigger.md) |
| Escritura de mensajes de Queue Storage |[Enlace de salida](./functions-bindings-storage-queue-output.md) |

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

- [ de una función cuando cambian los datos de Queue Storage (desencadenador)](./functions-bindings-storage-queue-trigger.md)
- [Escritura de mensajes de Queue Storage (enlace de salida)](./functions-bindings-storage-queue-output.md)
