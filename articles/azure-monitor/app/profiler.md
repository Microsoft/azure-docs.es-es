---
title: Generación de perfiles de aplicaciones activas de Azure App Service con Application Insights | Microsoft Docs
description: Cree perfiles de aplicaciones activas de Azure App Service con Application Insights Profiler.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 4f5328bbe21bb5f4a7947d5a495f4a0c8759c8da
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87315644"
---
# <a name="profile-live-azure-app-service-apps-with-application-insights"></a>Generación de perfiles de aplicaciones activas de Azure App Service con Application Insights

Puede ejecutar Profiler en aplicaciones ASP.NET y ASP.NET Core que se ejecutan en Azure App Service con el nivel de servicio Básico o superior. Actualmente, la habilitación de Profiler en Linux solo es posible a través de [este método](profiler-aspnetcore-linux.md).

## <a name="enable-profiler-for-your-app"></a><a id="installation"></a> Habilitación de Profiler en la aplicación
Para habilitar Profiler en una aplicación, siga las instrucciones que se indican a continuación. Si está ejecutando otro tipo de servicio de Azure, aquí encontrará instrucciones para habilitar Profiler en otras plataformas compatibles:
* [Cloud Services](./profiler-cloudservice.md?toc=%2fazure%2fazure-monitor%2ftoc.json)
* [Aplicaciones de Service Fabric](./profiler-servicefabric.md?toc=%2fazure%2fazure-monitor%2ftoc.json)
* [Máquinas virtuales](./profiler-vm.md?toc=%2fazure%2fazure-monitor%2ftoc.json)

Application Insights Profiler viene preinstalado en el entorno de ejecución de App Services. Los pasos siguientes le mostrarán cómo habilitarlo para App Service. Siga estos pasos incluso si ha incluido el SDK de App Insights en la aplicación en el momento de la compilación.

1. Vaya al panel de control de Azure y busque la instancia de App Service.
1. Habilite a la opción "Siempre disponible" para el servicio de aplicación. Puede encontrar esta opción en **Opciones**, página **Configuración** (consulte la captura de pantalla en el paso siguiente) y haga clic en la pestaña **Configuración general**.
1. Vaya a la página **Configuración > Application Insights**.

   ![Habilitación de App Insights en el portal de App Services](./media/profiler/AppInsights-AppServices.png)

1. Siga las instrucciones que aparecen en el panel para crear un nuevo recurso o seleccione un recurso de App Insights existente para supervisar la aplicación. También, asegúrese de que Profiler esté **activado**. Si el recurso de Application Insights está en otra suscripción de App Service, no puede usar esta página para configurar Application­ Insights. Sin embargo, puede hacerlo manualmente creando la configuración de la aplicación necesaria por usted mismo. [La siguiente sección contiene instrucciones para habilitar Profiler de forma manual.](#enable-profiler-manually-or-with-azure-resource-manager) 

   ![Adición de la extensión de sitio de App Insights][Enablement UI]

1. Profiler está habilitado con una configuración de aplicación de App Services.

    ![Configuración de aplicación de Profiler][profiler-app-setting]

## <a name="enable-profiler-manually-or-with-azure-resource-manager"></a>Habilitación de Profiler manualmente o con Azure Resource Manager
Application Insights Profiler puede habilitarse mediante la creación de la configuración de la aplicación para Azure App Service. La página con las opciones mostradas anteriormente crea esta configuración de aplicación de forma automática. Sin embargo, es posible automatizar la creación de estas opciones mediante una plantilla u otros medios. Esta configuración también funciona si el recurso de Application Insights está en otra suscripción de Azure App Service.
Estos son los valores necesarios para habilitar el generador de perfiles:

|Configuración de aplicación    | Value    |
|---------------|----------|
|APPINSIGHTS_INSTRUMENTATIONKEY         | iKey para el recurso de Application Insights    |
|APPINSIGHTS_PROFILERFEATURE_VERSION | 1.0.0 |
|DiagnosticServices_EXTENSION_VERSION | ~3 |


Puede establecer estos valores mediante las [plantillas de Azure Resource Manager](./azure-web-apps.md#app-service-application-settings-with-azure-resource-manager), [Azure PowerShell](/powershell/module/az.websites/set-azwebapp) o la [CLI de Azure](/cli/azure/webapp/config/appsettings?view=azure-cli-latest).

### <a name="enabling-profiler-for-other-clouds-manually"></a>Habilitación de Profiler para otras nubes manualmente

Si quiere habilitar Profiler para otras nubes, puede usar la siguiente configuración de aplicación.

|Configuración de aplicación    | Valores de US Government| Nube de China |   
|---------------|---------------------|-------------|
|ApplicationInsightsProfilerEndpoint         | `https://agent.serviceprofiler.azure.us`    | `https://profiler.applicationinsights.azure.cn` |
|ApplicationInsightsEndpoint | `https://dc.applicationinsights.us` | `https://dc.applicationinsights.azure.cn` |

## <a name="disable-profiler"></a>Deshabilitación de Profiler

Para detener o reiniciar Profiler en una instancia de la aplicación, vaya a **WebJobs** y detenga el trabajo web llamado ApplicationInsightsProfiler3. Incluso si el generador de perfiles se deshabilita mediante el modificador de la página de Application Insights como se describió anteriormente, el proceso del generador de perfiles se seguirá ejecutando. El generador de perfiles comprobará si está habilitado. Si está deshabilitado, entrará en suspensión durante un tiempo antes de volver a realizar la comprobación. Si está deshabilitado, no realizará la generación de perfiles. Si deshabilita este trabajo web, el proceso del generador de perfiles no se ejecutará, ni siquiera para comprobar si está habilitado.

  ![Deshabilitación de Profiler para un trabajo web][disable-profiler-webjob]

Se recomienda tener habilitado el generador de perfiles en todas las aplicaciones para detectar cualquier problema de rendimiento lo antes posible.

Los archivos de Profiler pueden eliminarse cuando se usa WebDeploy para implementar los cambios en la aplicación web. Puede evitar la eliminación si se excluye la carpeta App_Data de la eliminación durante la implementación. 


## <a name="next-steps"></a>Pasos siguientes

* [Trabajo con Application Insights en Visual Studio](./visual-studio.md)

[Enablement UI]: ./media/profiler/Enablement_UI.png
[profiler-app-setting]:./media/profiler/profiler-app-setting.png
[disable-profiler-webjob]: ./media/profiler/disable-profiler-webjob.png

