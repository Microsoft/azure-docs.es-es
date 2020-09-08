---
title: 'Inicio rápido: la primera consulta de .NET Core'
description: En este inicio rápido, dará los pasos necesarios para habilitar los paquetes NuGet de Resource Graph para .NET Core y ejecutará la primera consulta.
ms.date: 06/29/2020
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: 0470f100b6323f760bc48af70e8a6c11b94dfb3c
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89005877"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-net-core"></a>Inicio rápido: ejecución de la primera consulta de Resource Graph mediante .NET Core

El primer paso para usar Azure Resource Graph es comprobar que están instalados los paquetes necesarios para .NET Core. Este inicio rápido lo guiará a través del proceso de agregar los paquetes a la instalación de .NET Core.

Cuando finalice, habrá agregado los paquetes a la instalación de .NET Core y ejecutado la primera consulta de Resource Graph.

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Una entidad de servicio de Azure, que incluya _clientId_ y _clientSecret_. Si no tiene una entidad de servicio para usarla con Resource Graph o desea crear una, consulte el artículo sobre las [bibliotecas de administración de Azure para la autenticación de .NET](/dotnet/azure/sdk/authentication#mgmt-auth).
  Omita el paso en el que se instalan los paquetes de .NET Core, ya que lo haremos en los pasos siguientes.

## <a name="create-the-resource-graph-project"></a>Creación del proyecto de Resource Graph

Para habilitar .NET Core para consultar Azure Resource Graph, cree una aplicación de consola e instale los paquetes necesarios.

1. Compruebe que está instalada la versión más reciente de .NET Core (al menos la **3.1.5**). Si aún no lo está, descárguela en [dotnet.microsoft.com](https://dotnet.microsoft.com/download/dotnet-core).

1. Inicialice una nueva aplicación de consola de .NET Core denominada "argQuery":

   ```dotnetcli
   dotnet new console --name "argQuery"
   ```

1. Cambie los directorios a la nueva carpeta del proyecto e instale los paquetes necesarios para Azure Resource Graph:

   ```dotnetcli
   # Add the Resource Graph package for .NET Core
   dotnet add package Microsoft.Azure.Management.ResourceGraph --version 2.0.0

   # Add the Azure app auth package for .NET Core
   dotnet add package Microsoft.Azure.Services.AppAuthentication --version 1.5.0
   ```

1. Reemplace el archivo `program.cs` predeterminado por el código siguiente y guarde el archivo actualizado:

   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Threading.Tasks;
   using Microsoft.IdentityModel.Clients.ActiveDirectory;
   using Microsoft.Rest;
   using Microsoft.Azure.Management.ResourceGraph;
   using Microsoft.Azure.Management.ResourceGraph.Models;

   namespace argQuery
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               string strTenant = args[0];
               string strClientId = args[1];
               string strClientSecret = args[2];
               string strSubscriptionId = args[3];
               string strQuery = args[4];

               AuthenticationContext authContext = new AuthenticationContext("https://login.microsoftonline.com/" + strTenant);
               AuthenticationResult authResult = await authContext.AcquireTokenAsync("https://management.core.windows.net", new ClientCredential(strClientId, strClientSecret));
               ServiceClientCredentials serviceClientCreds = new TokenCredentials(authResult.AccessToken);

               ResourceGraphClient argClient = new ResourceGraphClient(serviceClientCreds);
               QueryRequest request = new QueryRequest();
               request.Subscriptions = new List<string>(){ strSubscriptionId };
               request.Query = strQuery;

               QueryResponse response = argClient.Resources(request);
               Console.WriteLine("Records: " + response.Count);
               Console.WriteLine("Data:\n" + response.Data);
           }
       }
   }
   ```

1. Compile y publique la aplicación de consola `argQuery`:

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

## <a name="run-your-first-resource-graph-query"></a>Ejecutar la primera consulta de Resource Graph

Con la aplicación de consola de .NET Core compilada y publicada, es el momento de probar una consulta simple de Resource Graph. La consulta devolverá los cinco primeros recursos de Azure con el **nombre** y el **tipo de recurso** de cada recurso.

En cada llamada a `argQuery`, hay variables que se usan que debe reemplazar por sus propios valores:

- `{tenantId}`: reemplácelo por su identificador de inquilino
- `{clientId}`: reemplácelo por el identificador de cliente de la entidad de servicio
- `{clientSecret}`: reemplácelo por el secreto de cliente de la entidad de servicio
- `{subscriptionId}`: reemplácelo por el identificador de suscripción

1. Cambie los directorios a `{run-folder}`, que ha definido con el comando `dotnet publish` anterior.

1. Ejecute la primera consulta de Azure Resource Graph mediante la aplicación de consola de .NET Core compilada:

   ```bash
   argQuery "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | limit 5"
   ```

   > [!NOTE]
   > Como esta consulta de ejemplo no proporciona un modificador de ordenación como `order by`, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud.

1. Cambie el parámetro final a `argQuery.exe` y cambie la consulta a `order by` en la propiedad **Nombre**:

   ```bash
   argQuery "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | limit 5 | order by name asc"
   ```

   > [!NOTE]
   > Al igual que con la primera consulta, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud. El orden de los comandos de consulta es importante. En este ejemplo, el `order by` viene después del `limit`. Este orden de comandos limita primero los resultados de la consulta y, luego, los ordena.

1. Cambie el parámetro final a `argQuery.exe` y cambie la consulta para realizar la operación `order by` primero en la propiedad **Nombre** y, luego, para realizar la operación `limit` a los cinco primeros resultados:

   ```bash
   argQuery "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | order by name asc | limit 5"
   ```

Cuando la consulta final se ejecuta varias veces, suponiendo que nada cambie en su entorno, los resultados devueltos serán coherentes y estarán ordenados por la propiedad **Nombre**, pero todavía limitados a los cinco primeros resultados.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea quitar la aplicación de consola de .NET Core y los paquetes instalados, puede hacerlo eliminando la carpeta de proyecto de `argQuery`.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una aplicación de consola de .NET Core con los paquetes de Resource Graph necesarios y ha ejecutado su primera consulta. Para más información sobre el lenguaje de Resource Graph, vaya a la página de detalles del lenguaje de consulta.

> [!div class="nextstepaction"]
> [Obtener más información sobre el lenguaje de consulta](./concepts/query-language.md)