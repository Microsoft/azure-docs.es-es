---
title: Generación de perfiles de aplicaciones de Azure Service Fabric en vivo con Application Insights
description: Habilitación de Profiler para una aplicación de Service Fabric
ms.topic: conceptual
ms.custom: devx-track-dotnet
author: cweining
ms.author: cweining
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 317c02fe03a56ea3cd3a98b00f8a0080b3917108
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88930288"
---
# <a name="profile-live-azure-service-fabric-applications-with-application-insights"></a>Generación de perfiles de aplicaciones de Azure Service Fabric en vivo con Application Insights

También puede implementar Application Insights Profiler en estos servicios:
* [Azure App Service](profiler.md?toc=/azure/azure-monitor/toc.json)
* [Azure Cloud Services](profiler-cloudservice.md?toc=/azure/azure-monitor/toc.json)
* [Azure Virtual Machines](profiler-vm.md?toc=/azure/azure-monitor/toc.json)

## <a name="set-up-the-environment-deployment-definition"></a>Configuración de la definición de implementación del entorno

Application Insights Profiler se incluye con Azure Diagnostics. Puede instalar la extensión de Azure Diagnostics mediante una plantilla de Azure Resource Manager para el clúster de Service Fabric. Obtenga una [plantilla que instale Azure Diagnostics en un clúster de Service Fabric](https://github.com/Azure/azure-docs-json-samples/blob/master/application-insights/ServiceFabricCluster.json).

Para configurar su entorno, realice las siguientes acciones:

1. Profiler admite .NET Framework y .NET Core. Si usa .NET Framework, asegúrese de que usa [.NET Framework 4.6.1](/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed) o una versión posterior. Basta con confirmar que el sistema operativo implementado es `Windows Server 2012 R2` o posterior. Profiler admite .NET Core 2.1 y aplicaciones más recientes.

1. Busque la extensión de [Azure Diagnostics](../platform/diagnostics-extension-overview.md) en el archivo de plantilla de implementación.

1. Agregue la siguiente sección `SinksConfig` como elemento secundario de `WadCfg`. Reemplace el valor de la propiedad `ApplicationInsightsProfiler` por su propia clave de instrumentación de Application Insights:  

      ```json
      "SinksConfig": {
        "Sink": [
          {
            "name": "MyApplicationInsightsProfilerSink",
            "ApplicationInsightsProfiler": "00000000-0000-0000-0000-000000000000"
          }
        ]
      }
      ```

      Para obtener información acerca de cómo agregar la extensión de Diagnostics a una plantilla de implementación, consulte [Uso de la supervisión y el diagnóstico con una máquina virtual Windows y plantillas de Azure Resource Manager](../../virtual-machines/extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json).

1. Implemente el clúster de Service Fabric mediante la plantilla de Azure Resource Manager.  
  Si la configuración es correcta, Application Insights Profiler se instala y se habilita cuando está instalada la extensión de Azure Diagnostics. 

1. Agregue Application Insights a la aplicación de Service Fabric.  
  Para que Profiler pueda recopilar los perfiles de las solicitudes, la aplicación debe supervisar las operaciones con Application Insights. Para las API sin estado, puede consultar las instrucciones para el [seguimiento de las solicitudes para la generación de perfiles](profiler-trackrequests.md?toc=/azure/azure-monitor/toc.json). Para obtener más información sobre la supervisión de operaciones personalizadas en otros tipos de aplicaciones, consulte [Seguimiento de las operaciones personalizadas con el SDK de .NET para Application Insights](custom-operations-tracking.md?toc=/azure/azure-monitor/toc.json).

1. Vuelva a implementar la aplicación.


## <a name="next-steps"></a>Pasos siguientes

* Genere tráfico para su aplicación (por ejemplo, inicie una [prueba de disponibilidad](monitor-web-app-availability.md)). A continuación, espere de 10 a 15 minutos para que se empiecen a enviar seguimientos a la instancia de Application Insights.
* Consulte [Seguimientos de Profiler](profiler-overview.md?toc=/azure/azure-monitor/toc.json) en Azure Portal.
* Para obtener ayuda para solucionar problemas de Profiler, consulte la sección [Solución de problemas de Profiler](profiler-troubleshooting.md?toc=/azure/azure-monitor/toc.json).
