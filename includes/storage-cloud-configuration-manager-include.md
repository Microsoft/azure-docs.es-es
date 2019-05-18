---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 10/26/2018
ms.author: tamram
ms.openlocfilehash: 761c3a9aecadd9c1eabdb586f95c47e2988720d8
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2019
ms.locfileid: "65756368"
---
La [biblioteca del Administrador de configuración de Microsoft Azure para .NET](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/) proporciona una clase para analizar una cadena de conexión desde un archivo de configuración. La clase [CloudConfigurationManager](https://msdn.microsoft.com/library/azure/mt634650.aspx) analiza las opciones de configuración con independencia de si la aplicación cliente se ejecuta en el equipo de escritorio, en un dispositivo móvil, en una máquina virtual de Azure o en un servicio en la nube de Azure.

Para hacer referencia al paquete CloudConfigurationManager, agregue la siguiente directiva `using`:

```csharp
using Microsoft.Azure; //Namespace for CloudConfigurationManager
using Microsoft.WindowsAzure.Storage;
```

Este es un ejemplo que muestra cómo recuperar una cadena de conexión de un archivo de configuración:

```csharp
// Parse the connection string and return a reference to the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));
```

El uso del Administrador de configuración de Azure es opcional. También se puede usar una API, como la clase [ConfigurationManager](https://msdn.microsoft.com/library/system.configuration.configurationmanager.aspx) de .NET Framework.

