---
title: 'Azure Application Insights: recopilación automática de dependencias | Microsoft Docs'
description: Recopilación y visualización automática de dependencias de Application Insights
ms.topic: reference
ms.custom: devx-track-dotnet
author: mrbullwinkle
ms.author: mbullwin
ms.date: 05/06/2020
ms.openlocfilehash: ca1c63f042bd06c19f232c2ff8170d23741e73f2
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936442"
---
# <a name="dependency-auto-collection"></a>Recopilación automática de dependencias

A continuación encontrará la lista de las llamadas de dependencia admitida actualmente que se detectan automáticamente como dependencias sin requerir ninguna modificación adicional en el código de la aplicación. Estas dependencias se visualizan en el [Mapa de aplicación](./app-map.md) de Application Insights y las vistas de [Diagnóstico de transacciones](./transaction-diagnostics.md). Si la dependencia no se encuentra en la siguiente lista, todavía puede realizar un seguimiento manual con una [llamada a TrackDependency ](./api-custom-events-metrics.md#trackdependency).

## <a name="net"></a>.NET

| Marcos de trabajo de aplicaciones| Versiones |
| ------------------------|----------|
| WebForms de ASP.NET | 4.5+ |
| ASP.NET MVC | 4+ |
| WebAPI de ASP.NET | 4.5+ |
| ASP.NET Core | 1.1+ |
| <b>Bibliotecas de comunicaciones</b> |
| [HttpClient](https://www.microsoft.com/net/) | 4.5+, .NET Core 1.1+ |
| [SqlClient](https://www.nuget.org/packages/System.Data.SqlClient) | .NET Core 1.0+, NuGet 4.3.0 |
| [Microsoft.Data.SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient/1.1.2)| 1.1.0: versión estable más reciente. (Consulte la siguiente nota).
| [SDK de cliente de Event Hubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs) | 1.1.0 |
| [SDK de cliente de Service Bus](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus) | 3.0.0 |
| <b>Clientes de Storage</b>|  |
| ADO.NET | 4.5+ |

> [!NOTE]
> Hay un [problema conocido](https://github.com/microsoft/ApplicationInsights-dotnet/issues/1347) con versiones anteriores de Microsoft.Data.SqlClient. Se recomienda usar la versión 1.1.0 o posterior para mitigar este problema. Entity Framework Core no necesariamente se distribuye con la versión estable más reciente de Microsoft.Data.SqlClient, por lo que se recomienda confirmar que está usando al menos la versión 1.1.0 para evitar este problema.   


## <a name="java"></a>Java
| Servidores de aplicaciones | Versiones |
|-------------|----------|
| [Tomcat](https://tomcat.apache.org/) | 7, 8 | 
| [JBoss EAP](https://developers.redhat.com/products/eap/download/) | 6, 7 |
| [Jetty](https://www.eclipse.org/jetty/) | 9 |
| <b>Marcos de trabajo de aplicaciones</b> |  |
| [Spring](https://spring.io/) | 3.0 |
| [Spring Boot](https://spring.io/projects/spring-boot) | 1.5.9+<sup>*</sup> |
| Servlet de Java | 3.1+ |
| <b>Bibliotecas de comunicaciones</b> |  |
| [Cliente HTTP de Apache](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient) | 4.3+<sup>†</sup> |
| <b>Clientes de Storage</b> | |
| [SQL Server]( https://mvnrepository.com/artifact/com.microsoft.sqlserver/mssql-jdbc) | 1+<sup>†</sup> |
| [PostgreSQL (compatibilidad con la versión beta)](https://github.com/Microsoft/ApplicationInsights-Java/blob/master/CHANGELOG.md#version-240-beta) | |
| [Oracle]( https://www.oracle.com/technetwork/database/application-development/jdbc/downloads/index.html) | 1+<sup>†</sup> |
| [MySQL]( https://mvnrepository.com/artifact/mysql/mysql-connector-java) | 1+<sup>†</sup> |
| <b>Bibliotecas de registro</b> | |
| [Logback](https://logback.qos.ch/) | 1+ |
| [Log4j](https://logging.apache.org/log4j/) | 1.2+ |
| <b>Bibliotecas de métricas</b> |  |
| JMX | 1.0+ |

> [!NOTE]
> \* Excepto el soporte técnico de programación reactivo.
> <br>† Requiere la instalación del [agente de JVM](./java-agent.md#install-the-application-insights-agent-for-java).

## <a name="nodejs"></a>Node.js

| Bibliotecas de comunicaciones | Versiones |
| ------------------------|----------|
| [HTTP](https://nodejs.org/api/http.html), [HTTPS](https://nodejs.org/api/https.html) | 0.10+ |
| <b>Clientes de Storage</b> | |
| [Redis](https://www.npmjs.com/package/redis) | 2.x |
| [MongoDb](https://www.npmjs.com/package/mongodb); [MongoDb Core](https://www.npmjs.com/package/mongodb-core) | 2.x - 3.x |
| [MySQL](https://www.npmjs.com/package/mysql) | 2.0.0 - 2.16.x |
| [PostgreSQL](https://www.npmjs.com/package/pg); | 6.x - 7.x |
| [pg-pool](https://www.npmjs.com/package/pg-pool) | 1.x - 2.x |
| <b>Bibliotecas de registro</b> | |
| [Consola](https://nodejs.org/api/console.html) | 0.10+ |
| [Bunyan](https://www.npmjs.com/package/bunyan) | 1.x |
| [Winston](https://www.npmjs.com/package/winston) | 2.x - 3.x |

## <a name="javascript"></a>JavaScript

| Bibliotecas de comunicaciones | Versiones |
| ------------------------|----------|
| [XMLHttpRequest](https://developer.mozilla.org/docs/Web/API/XMLHttpRequest) | All |

## <a name="next-steps"></a>Pasos siguientes

- Configuración del seguimiento de dependencias personalizadas para [.NET](./asp-net-dependencies.md).
- Configuración del seguimiento de dependencias personalizadas para [Java](./java-agent.md).
- Configuración del seguimiento de dependencias personalizado para [OpenCensus Python](./opencensus-python-dependency.md).
- [Escritura de una telemetría de dependencia personalizada](./api-custom-events-metrics.md#trackdependency)
- Consulte [modelo de datos](./data-model.md) para los tipos y el modelo de datos de Application Insights.
- Consulte las [plataformas](./platforms.md) compatibles con Application Insights.

