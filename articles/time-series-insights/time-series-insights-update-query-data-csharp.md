---
title: 'Consulta de datos desde un entorno de segunda generación mediante C#: Azure Time Series Insights | Microsoft Docs'
description: Aprenda a consultar datos desde un entorno de segunda generación de Azure Time Series Insights mediante una aplicación escrita en C#.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 08/14/2020
ms.custom: seodec18
ms.openlocfilehash: fcb38f761a72bf2c6e5dee8bb993cec9143e7241
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998958"
---
# <a name="query-data-from-the-azure-time-series-insights-gen2-environment-using-c-sharp"></a>Consulta de datos desde el entorno de Azure Time Series Insights Gen2 mediante C Sharp

En este ejemplo de C# se muestra cómo consultar datos desde las [API de acceso a datos de segunda generación](https://docs.microsoft.com/rest/api/time-series-insights/reference-data-access-overview) en los entornos de Azure Time Series Insights de segunda generación.

> [!TIP]
> Consulte ejemplos de código C# de segunda generación en [https://github.com/Azure-Samples/Azure-Time-Series-Insights](https://github.com/Azure-Samples/Azure-Time-Series-Insights/tree/master/gen2-sample/csharp-tsi-gen2-sample).

## <a name="summary"></a>Resumen

En el código de ejemplo siguiente se muestran las siguientes características:

* Compatibilidad con la generación automática del SDK desde [Azure AutoRest](https://github.com/Azure/AutoRest).
* Cómo adquirir un token de acceso a través de Azure Active Directory mediante [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/).
* Cómo pasar el token de acceso adquirido en el encabezado `Authorization` de las posteriores solicitudes de Data Access API.
* El ejemplo proporciona una interfaz de consola que muestra cómo se realizan las solicitudes HTTP en:
  * [API de entornos de segunda generación](https://docs.microsoft.com/rest/api/time-series-insights/reference-environments-apis)
    * [Get Environments Availability API](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/getavailability) y [Get Event Schema API](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/geteventschema)
  * [API de consulta de segunda generación](https://docs.microsoft.com/rest/api/time-series-insights/reference-query-apis)
    * [Get Events API](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/execute#getevents), [Get Series API](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/execute#getseries) y [Get Aggregate Series API](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/execute#aggregateseries)
  * [API de modelo de serie temporal](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/execute#aggregateseries)
    * [Get Hierarchies API](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeserieshierarchies) y [Hierarchies Batch API](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeserieshierarchies/executebatch)
    * [Get Types API](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeseriestypes) y [Types Batch API](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/executebatch)
    * [Get Instances API](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances) e [Instances Batch API](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/executebatch)

* Funcionalidades avanzadas de [búsqueda](https://docs.microsoft.com/rest/api/time-series-insights/reference-model-apis#search-features) y [TSX](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax).

## <a name="prerequisites-and-setup"></a>Requisitos previos y configuración

Realice los pasos siguientes antes de compilar y ejecutar el código de ejemplo:

1. [Aprovisione un entorno de segunda generación de Azure Time Series Insights](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-update-how-to-manage#create-the-environment).
1. Configure el entorno de Azure Time Series Insights para Azure Active Directory, tal como se describe en [Autenticación y autorización](time-series-insights-authentication-and-authorization.md).
1. Ejecute el archivo [GenerateCode.bat](https://github.com/Azure-Samples/Azure-Time-Series-Insights/blob/master/gen2-sample/csharp-tsi-gen2-sample/DataPlaneClient/GenerateCode.bat) como se especifica en el archivo [Readme.md](https://github.com/Azure-Samples/Azure-Time-Series-Insights/blob/master/gen2-sample/csharp-tsi-gen2-sample/DataPlaneClient/Readme.md) para generar las dependencias de cliente de Azure Time Series Insights de segunda generación.
1. Abra la solución `TSIPreviewDataPlaneclient.sln` y establezca `DataPlaneClientSampleApp` como proyecto predeterminado en Visual Studio.
1. Instale las dependencias del proyecto necesarias, para lo cual debe seguir los pasos que se describen [a continuación](#project-dependencies), y compile el ejemplo en un archivo `.exe` ejecutable.
1. Ejecute el archivo `.exe` haciendo doble clic en él.

## <a name="project-dependencies"></a>Dependencias del proyecto

Se recomienda usar la versión más reciente de Visual Studio:

* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/), versión 16.4.2+

El código de ejemplo tiene varias dependencias requeridas, que se pueden ver en el archivo [packages.config](https://github.com/Azure-Samples/Azure-Time-Series-Insights/blob/master/gen2-sample/csharp-tsi-gen2-sample/DataPlaneClientSampleApp/packages.config).

Descargue los paquetes en Visual Studio 2019, para lo cual debe seleccionar la opción **Compilar** > **Compilar solución**.

Como alternativa, agregue cada paquete con [NuGet 2.12+](https://www.nuget.org/). Por ejemplo:

* `dotnet add package Microsoft.IdentityModel.Clients.ActiveDirectory --version 4.5.1`

## <a name="c-sample-code"></a>Código de ejemplo de C#

Consulte el repositorio [Azure Time Series Insights](https://github.com/Azure-Samples/Azure-Time-Series-Insights/tree/master/gen2-sample/csharp-tsi-gen2-sample) para acceder al código de ejemplo de C#. 

> [!NOTE]
>
> * El código de ejemplo se puede ejecutar sin modificar los valores de entorno predeterminados.
> * El código de ejemplo se compilará en una aplicación de consola ejecutable de .NET.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre la consulta, vea la [referencia sobre las API de consulta](https://docs.microsoft.com/rest/api/time-series-insights/reference-query-apis).

* Lea cómo [conectar una aplicación de JavaScript mediante el SDK de cliente](https://github.com/microsoft/tsiclient) a Azure Time Series Insights.
