---
title: Escritura de código para realizar el seguimiento de solicitudes con Azure Application Insights | Microsoft Docs
description: Escriba código para realizar un seguimiento de solicitudes con Application Insights para que pueda obtener perfiles para sus solicitudes.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.custom: devx-track-csharp
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: aaa1d6df9faa20b1a561bfccdfea682af7645c18
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88930254"
---
# <a name="write-code-to-track-requests-with-application-insights"></a>Escritura de código para realizar el seguimiento de solicitudes con Application Insights

Para ver los perfiles de la aplicación en la página Rendimiento, Azure Application Insights tiene que realizar un seguimiento de las solicitudes para la aplicación. Application Insights puede realizar automáticamente un seguimiento de solicitudes para las aplicaciones que se basan en marcos de trabajo ya instrumentados. Dos ejemplos son ASP.NET y ASP.NET Core. 

Pero para otras aplicaciones, como los roles de trabajo de Azure Cloud Services y las API sin estado de Service Fabric, es necesario escribir código para indicar a Application Insights dónde empiezan y terminan las solicitudes. Una vez escrito este código, la telemetría de solicitudes se envía a Application Insights. Puede ver la telemetría en la página Rendimiento y los perfiles se recopilan para dichas solicitudes. 

Para realizar un seguimiento de las solicitudes manualmente, haga lo siguiente:

  1. Agregue el código siguiente al principio de la duración de la aplicación:  

        ```csharp
        using Microsoft.ApplicationInsights.Extensibility;
        ...
        // Replace with your own Application Insights instrumentation key.
        TelemetryConfiguration.Active.InstrumentationKey = "00000000-0000-0000-0000-000000000000";
        ```
      Para más información acerca de la configuración de esta clave de instrumentación global, consulte [Using Service Fabric with Application Insights](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/blob/dev/appinsights/ApplicationInsights.md) (Uso de Service Fabric con Application Insights).  

  1. Con respecto a cualquier parte del código que quiera instrumentar, agregue una instrucción `StartOperation<RequestTelemetry>` **using**  alrededor, como se muestra en el ejemplo siguiente:

        ```csharp
        using Microsoft.ApplicationInsights;
        using Microsoft.ApplicationInsights.DataContracts;
        ...
        var client = new TelemetryClient();
        ...
        using (var operation = client.StartOperation<RequestTelemetry>("Insert_Your_Custom_Event_Unique_Name"))
        {
          // ... Code I want to profile.
        }
        ```

        No se permite llamar a `StartOperation<RequestTelemetry>` dentro de otro ámbito de `StartOperation<RequestTelemetry>`. En su lugar, puede usar `StartOperation<DependencyTelemetry>` en el ámbito anidado. Por ejemplo:  
        
        ```csharp
        using (var getDetailsOperation = client.StartOperation<RequestTelemetry>("GetProductDetails"))
        {
        try
        {
          ProductDetail details = new ProductDetail() { Id = productId };
          getDetailsOperation.Telemetry.Properties["ProductId"] = productId.ToString();
        
          // By using DependencyTelemetry, 'GetProductPrice' is correctly linked as part of the 'GetProductDetails' request.
          using (var getPriceOperation = client.StartOperation<DependencyTelemetry>("GetProductPrice"))
          {
              double price = await _priceDataBase.GetAsync(productId);
              if (IsTooCheap(price))
              {
                  throw new PriceTooLowException(productId);
              }
              details.Price = price;
          }
        
          // Similarly, note how 'GetProductReviews' doesn't establish another RequestTelemetry.
          using (var getReviewsOperation = client.StartOperation<DependencyTelemetry>("GetProductReviews"))
          {
              details.Reviews = await _reviewDataBase.GetAsync(productId);
          }
        
          getDetailsOperation.Telemetry.Success = true;
          return details;
        }
        catch(Exception ex)
        {
          getDetailsOperation.Telemetry.Success = false;
        
          // This exception gets linked to the 'GetProductDetails' request telemetry.
          client.TrackException(ex);
          throw;
        }
        }
        ```
