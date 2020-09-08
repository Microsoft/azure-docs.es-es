---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 08/07/2020
ms.author: glenga
ms.openlocfilehash: a4f03223b5067d18f5d6e747b3bb630a86031b8f
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689577"
---
Si no puede usar paquetes de extensión, utilice Azure Functions Core Tools localmente para instalar los paquetes de extensión específicos que requiere el proyecto.

> [!IMPORTANT]
> No se pueden instalar extensiones explícitamente en una aplicación de función que use conjuntos de extensiones. Antes de instalar las extensiones explícitamente, quite la sección `extensionBundle` en *host.json*.

En los siguientes elementos se describen algunos de los motivos por los que es posible que tenga que instalar las extensiones manualmente:

* Debe tener acceso a una versión específica de una extensión que no está disponible en un conjunto.
* Debe tener acceso a una extensión personalizada que no está disponible en un conjunto.
* Debe tener acceso a una combinación específica de extensiones que no está disponible en un único conjunto.

> [!NOTE]
> Para instalar manualmente extensiones mediante Core Tools, debe tener instalado el [SDK de .NET Core 2.x](https://dotnet.microsoft.com/download). Azure Functions Core Tools usa el SDK de .NET Core para instalar extensiones de NuGet. No es necesario tener conocimientos de .NET para usar las extensiones de Azure Functions.

Cuando se instalan las extensiones explícitamente, se agrega un archivo de proyecto .NET denominado extensions.csproj a la raíz del proyecto. Este archivo define el conjunto de paquetes NuGet requeridos por las funciones. Aunque puede trabajar con las [referencias de paquetes de NuGet](/nuget/consume-packages/package-references-in-project-files) en este archivo, las herramientas principales le permiten instalar extensiones sin tener que editar manualmente el archivo.

Hay varias maneras de usar las herramientas principales para instalar las extensiones necesarias en el proyecto local. 

#### <a name="install-all-extensions"></a>Instalación de todas las extensiones 

Utilice el comando siguiente para agregar automáticamente todos los paquetes de extensiones que usan los enlaces del proyecto local:

```dotnetcli
func extensions install
```
El comando lee el archivo *function.json* para ver qué paquetes necesita, los instala y, luego, recompila el proyecto de extensiones. Agrega los nuevos enlaces en la versión actual, pero no actualiza los enlaces existentes. Use la opción `--force` para actualizar los enlaces existentes a la versión más reciente cuando instale otros nuevos.

Si la aplicación de función usa enlaces que no reconoce Core Tools, debe instalar manualmente la extensión específica.

#### <a name="install-a-specific-extension"></a>Instalación de una extensión específica

Utilice el siguiente comando para instalar un paquete de extensiones específico en una versión específica, en este caso la extensión de almacenamiento:

```dotnetcli
func extensions install --package Microsoft.Azure.WebJobs.Extensions.Storage --version 4.0.2
```
