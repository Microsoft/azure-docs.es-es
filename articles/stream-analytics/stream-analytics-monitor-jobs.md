---
title: Supervisión y administración de los trabajos de Azure Stream Analytics mediante programación
description: En este artículo se describe cómo supervisar mediante programación los trabajos de Stream Analytics creados a través de REST API, Azure SDK o PowerShell.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 04/20/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 196d722e57abe743fc51692dde231e8f93682c46
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002273"
---
# <a name="programmatically-create-a-stream-analytics-job-monitor"></a>Creación de supervisión de trabajos de Stream Analytics mediante programación

En este artículo se demuestra cómo habilitar la supervisión de un trabajo de Stream Analytics. Los trabajos de Stream Analytics creados a través de las API de REST, el SDK de Azure o PowerShell no tienen habilitada la supervisión de forma predeterminada. Puede habilitarla manualmente en Azure Portal dirigiéndose a la página Supervisión del trabajo y haciendo clic en el botón Habilitar, o bien puede automatizar este proceso siguiendo los pasos que se describen en este artículo. Los datos de supervisión se mostrarán en el área Métricas de Azure Portal para el trabajo de Stream Analytics.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar con este proceso, debe cumplir los siguientes requisitos previos:

* Visual Studio 2019 o 2015
* [SDK de .NET para Azure](https://azure.microsoft.com/downloads/) descargado e instalado
* Un trabajo de Stream Analytics existente que requiera la habilitación de supervisión

## <a name="create-a-project"></a>Crear un proyecto

1. Cree una aplicación de consola .NET de Visual Studio C#.
2. En la consola del administrador de paquetes, ejecute los siguientes comandos para instalar los paquetes NuGet. El primero es el SDK de .NET de administración de Azure Stream Analytics. El segundo es el SDK de Azure Monitor que se usará para habilitar la supervisión. El último es el cliente de Azure Active Directory que se usará para autenticación.
   
   ```powershell
   Install-Package Microsoft.Azure.Management.StreamAnalytics
   Install-Package Microsoft.Azure.Insights -Pre
   Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
   ```
3. Agregue la siguiente sección appSettings al archivo App.config:
   
   ```csharp
   <appSettings>
     <!--CSM Prod related values-->
     <add key="ResourceGroupName" value="RESOURCE GROUP NAME" />
     <add key="JobName" value="YOUR JOB NAME" />
     <add key="StorageAccountName" value="YOUR STORAGE ACCOUNT"/>
     <add key="ActiveDirectoryEndpoint" value="https://login.microsoftonline.com/" />
     <add key="ResourceManagerEndpoint" value="https://management.azure.com/" />
     <add key="WindowsManagementUri" value="https://management.core.windows.net/" />
     <add key="AsaClientId" value="1950a258-227b-4e31-a9cf-717495945fc2" />
     <add key="RedirectUri" value="urn:ietf:wg:oauth:2.0:oob" />
     <add key="SubscriptionId" value="YOUR AZURE SUBSCRIPTION ID" />
     <add key="ActiveDirectoryTenantId" value="YOUR TENANT ID" />
   </appSettings>
   ```
   Reemplace los valores para *SubscriptionId* y *ActiveDirectoryTenantId* por sus identificadores de inquilino y de suscripción de Azure. Para obtener estos valores, ejecute el siguiente cmdlet de PowerShell:
   
   ```powershell
   Get-AzureAccount
   ```
4. Agregue las siguientes instrucciones using al archivo de origen (Program.cs) en el proyecto.
   
   ```csharp
     using System;
     using System.Configuration;
     using System.Threading;
     using Microsoft.Azure;
     using Microsoft.Azure.Management.Insights;
     using Microsoft.Azure.Management.Insights.Models;
     using Microsoft.Azure.Management.StreamAnalytics;
     using Microsoft.Azure.Management.StreamAnalytics.Models;
     using Microsoft.IdentityModel.Clients.ActiveDirectory;
   ```
5. Agregue un método auxiliar de autenticación:

   ```csharp   
   public static string GetAuthorizationHeader()
   {
      AuthenticationResult result = null;
      var thread = new Thread(() =>
      {
         try
         {
             var context = new AuthenticationContext(
                ConfigurationManager.AppSettings["ActiveDirectoryEndpoint"] +
                ConfigurationManager.AppSettings["ActiveDirectoryTenantId"]);
             result = context.AcquireToken(
                 resource: ConfigurationManager.AppSettings["WindowsManagementUri"],
                 clientId: ConfigurationManager.AppSettings["AsaClientId"],
                 redirectUri: new Uri(ConfigurationManager.AppSettings["RedirectUri"]),
                 promptBehavior: PromptBehavior.Always);
         }
         catch (Exception threadEx)
         {
             Console.WriteLine(threadEx.Message);
         }
     });

     thread.SetApartmentState(ApartmentState.STA);
     thread.Name = "AcquireTokenThread";
     thread.Start();
     thread.Join();
   
     if (result != null)
     {
         return result.AccessToken;
     }
         throw new InvalidOperationException("Failed to acquire token");
   }
   ```

## <a name="create-management-clients"></a>Creación de clientes de administración

El código siguiente configurará las variables y los clientes de administración necesarios.

   ```csharp
    string resourceGroupName = "<YOUR AZURE RESOURCE GROUP NAME>";
    string streamAnalyticsJobName = "<YOUR STREAM ANALYTICS JOB NAME>";

    // Get authentication token
    TokenCloudCredentials aadTokenCredentials =
        new TokenCloudCredentials(
            ConfigurationManager.AppSettings["SubscriptionId"],
            GetAuthorizationHeader());

    Uri resourceManagerUri = new
    Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

    // Create Stream Analytics and Insights management client
    StreamAnalyticsManagementClient streamAnalyticsClient = new
    StreamAnalyticsManagementClient(aadTokenCredentials, resourceManagerUri);
    InsightsManagementClient insightsClient = new
    InsightsManagementClient(aadTokenCredentials, resourceManagerUri);
   ```

## <a name="enable-monitoring-for-an-existing-stream-analytics-job"></a>Habilitación de supervisión para un trabajo de Stream Analytics existente

El código siguiente habilitará la supervisión de un trabajo de Stream Analytics **existente**. La primera parte del código realiza una solicitud GET en el servicio Stream Analytics para recuperar información sobre el trabajo de Stream Analytics en concreto. Usa la propiedad *ID* (recuperada de la solicitud GET) como parámetro del método Put en la segunda mitad del código que envía una solicitud PUT al servicio Insights para habilitar la supervisión para el trabajo de Stream Analytics.

> [!WARNING]
> Si previamente ha habilitado la supervisión de otro trabajo de Stream Analytics, a través del Azure Portal o mediante programación con el siguiente código, **es recomendable proporcionar el mismo nombre de cuenta de almacenamiento que usó cuando habilitó anteriormente la supervisión.**
> 
> La cuenta de almacenamiento está vinculada a la región en la que se ha creado el trabajo de Stream Analytics, no específicamente al trabajo.
> 
> Todos los trabajos de Stream Analytics (y todos los demás recursos de Azure) de esa misma región comparten esta cuenta de almacenamiento para almacenar los datos de supervisión. Si proporciona otra cuenta de almacenamiento, puede provocar efectos secundarios no deseados en la supervisión de sus otros trabajos de Stream Analytics u otros recursos de Azure.
> 
> El nombre de la cuenta de almacenamiento utilizado para reemplazar `<YOUR STORAGE ACCOUNT NAME>` en el siguiente código debe ser una cuenta de almacenamiento que esté en la misma suscripción que el trabajo de Stream Analytics para el que está habilitando la supervisión.
> 
> 
>    ```csharp
>    // Get an existing Stream Analytics job
>     JobGetParameters jobGetParameters = new JobGetParameters()
>     {
>         PropertiesToExpand = "inputs,transformation,outputs"
>     };
>     JobGetResponse jobGetResponse = streamAnalyticsClient.StreamingJobs.Get(resourceGroupName, streamAnalyticsJobName, jobGetParameters);
>
>    // Enable monitoring
>    ServiceDiagnosticSettingsPutParameters insightPutParameters = new ServiceDiagnosticSettingsPutParameters()
>    {
>            Properties = new ServiceDiagnosticSettings()
>           {
>               StorageAccountName = "<YOUR STORAGE ACCOUNT NAME>"
>           }
>    };
>   insightsClient.ServiceDiagnosticSettingsOperations.Put(jobGetResponse.Job.Id, insightPutParameters);
>   ```


## <a name="get-support"></a>Obtención de soporte técnico

Para más ayuda, pruebe nuestra [Página de preguntas y respuestas de Microsoft sobre Azure Stream Analytics](https://docs.microsoft.com/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Introducción al uso de Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalación de trabajos de Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)
