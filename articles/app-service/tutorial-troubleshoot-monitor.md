---
title: 'Tutorial: Solución de problemas con Azure Monitor'
description: Obtenga información sobre cómo Azure Monitor y Log Analytics ayudan a supervisar la aplicación web de App Service. Azure Monitor maximiza la disponibilidad mediante la entrega de una solución completa para la supervisión de los entornos.
author: msangapu-msft
ms.author: msangapu
ms.topic: tutorial
ms.date: 06/20/2020
ms.openlocfilehash: c34cf47a5b8c20c10b160ac6e55309b3c18448f3
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88959024"
---
# <a name="tutorial-troubleshoot-an-app-service-app-with-azure-monitor"></a>Tutorial: Solución de problemas de una aplicación de App Service con Azure Monitor

> [!NOTE]
> La integración de Azure Monitor con App Service se encuentra en [versión preliminar](https://aka.ms/appsvcblog-azmon).
>

En este tutorial se muestra cómo solucionar los problemas de una aplicación de [App Service](overview.md) mediante [Azure Monitor](../azure-monitor/overview.md). La aplicación de ejemplo incluye código destinado a agotar la memoria y provocar errores HTTP 500, de modo que puede diagnosticar y corregir el problema mediante Azure Monitor. Cuando haya terminado, tendrá una aplicación de ejemplo que se ejecuta en App Service en Linux integrado con [Azure Monitor](../azure-monitor/overview.md).

[Azure Monitor](../azure-monitor/overview.md) maximiza la disponibilidad y el rendimiento de las aplicaciones y los servicios con una completa solución que permite recopilar, analizar y actuar sobre datos de telemetría tanto en la nube como en entornos locales.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar una aplicación web con Azure Monitor
> * Enviar los registros de consola a Log Analytics
> * Usar consultas de registro para identificar y solucionar problemas de errores de la aplicación web

También puede seguir los pasos de este tutorial en macOS, Linux, Windows.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

- [Suscripción de Azure](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)
- [CLI de Azure](/cli/azure/install-azure-cli)
- [Git](https://git-scm.com/)

## <a name="create-azure-resources"></a>Creación de recursos de Azure

En primer lugar, se ejecutarán varios comandos de forma local para configurar una aplicación de ejemplo para usarla con este tutorial. Los comandos clonan una aplicación de ejemplo, crean recursos de Azure, crean un usuario de implementación e implementan la aplicación en Azure. Se le pedirá la contraseña que proporcionó como parte de la creación del usuario de implementación. 

```bash
git clone https://github.com/Azure-Samples/App-Service-Troubleshoot-Azure-Monitor
az group create --name myResourceGroup --location "South Central US"
az webapp deployment user set --user-name <username> --password <password>
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku B1 --is-linux
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --runtime "PHP|7.3" --deployment-local-git
git remote add azure <url_from_previous_step>
git push azure master
```

## <a name="configure-azure-monitor-preview"></a>Configuración de Azure Monitor (versión preliminar)

### <a name="create-a-log-analytics-workspace"></a>Creación de un área de trabajo de Log Analytics

Ahora que ha implementado la aplicación de ejemplo en Azure App Service, configurará la funcionalidad de supervisión para solucionar los problemas de la aplicación cuando estos surjan. Azure Monitor almacena datos de registro en un área de trabajo de Log Analytics. Un área de trabajo es un contenedor que incluye datos e información de configuración.

En este paso, creará un área de trabajo de Log Analytics para configurar Azure Monitor con la aplicación.

```bash
az monitor log-analytics workspace create --resource-group myResourceGroup --workspace-name myMonitorWorkspace
```

> [!NOTE]
> [Por Log Analytics de Azure Monitor, se paga por la ingesta de datos y la retención de datos.](https://azure.microsoft.com/pricing/details/monitor/)
>

### <a name="create-a-diagnostic-setting"></a>Creación de una configuración de diagnóstico

Las configuraciones de diagnóstico se pueden usar para recopilar métricas para determinados servicios de Azure en los registros de Azure Monitor para analizarlas con otros datos de supervisión mediante consultas de registro. En este tutorial, habilitará el servidor web y los registros de error y salida estándar. Consulte los [tipos de registro admitidos](./troubleshoot-diagnostic-logs.md#supported-log-types) para obtener una lista completa de los tipos de registro y sus descripciones.

Ejecute los siguientes comandos para crear la configuración de diagnóstico para AppServiceConsoleLogs (error y salida estándar) y AppServiceHTTPLogs (registros del servidor web). Reemplace _\<app-name>_ y _\<workspace-name>_ por sus valores. 

> [!NOTE]
> Los dos primeros comandos, `resourceID` y `workspaceID`, son variables que se utilizarán en el comando `az monitor diagnostic-settings create`. Consulte [Creación de una configuración de diagnóstico con la CLI de Azure](../azure-monitor/platform/diagnostic-settings.md#create-using-azure-cli) para más información sobre este comando.
>

```bash
resourceID=$(az webapp show -g myResourceGroup -n <app-name> --query id --output tsv)

workspaceID=$(az monitor log-analytics workspace show -g myResourceGroup --workspace-name <workspace-name> --query id --output tsv)

az monitor diagnostic-settings create --resource $resourceID \
 --workspace $workspaceID \
 -n myMonitorLogs \
 --logs '[{"category": "AppServiceConsoleLogs", "enabled": true},
  {"category": "AppServiceHTTPLogs", "enabled": true}]'
```

## <a name="troubleshoot-the-app"></a>Solución de problemas de la aplicación

Vaya a `http://<app-name>.azurewebsites.net`.

La aplicación de ejemplo, ImageConverter, convierte las imágenes incluidas de `JPG` a `PNG`. Se ha colocado deliberadamente un error en el código de este tutorial. Si selecciona suficientes imágenes, la aplicación genera un error HTTP 500 durante la conversión de imágenes. Imagine que este escenario no se tuvo en cuenta durante la fase de desarrollo. Usará Azure Monitor para solucionar el error.

### <a name="verify-the-app-is-works"></a>Comprobación del funcionamiento de la aplicación

Para convertir las imágenes, haga clic en `Tools` y seleccione `Convert to PNG`.

![Clic en "Tools" (Herramientas) y selección de "Convert to PNG" (Convertir en PNG)](./media/tutorial-azure-monitor/sample-monitor-app-tools-menu.png)

Seleccione las dos primeras imágenes y haga clic en `convert`. Esta conversión se realizará correctamente.

![Selección de las dos primeras imágenes](./media/tutorial-azure-monitor/sample-monitor-app-convert-two-images.png)

### <a name="break-the-app"></a>Interrupción de la aplicación

Ahora que ha comprobado que la aplicación convierte dos imágenes correctamente, intentaremos convertir las cinco primeras imágenes.

![Conversión de las cinco primeras imágenes](./media/tutorial-azure-monitor/sample-monitor-app-convert-five-images.png)

Esta acción no se realiza correctamente y genera un error `HTTP 500` que no se probó durante el desarrollo.

![Conversión con resultado de error HTTP 500](./media/tutorial-azure-monitor/sample-monitor-app-http-500.png)

## <a name="use-log-query-to-view-azure-monitor-logs"></a>Uso de una consulta de registro para ver los registros de Azure Monitor

Veamos qué registros están disponibles en el área de trabajo de Log Analytics. 

Haga clic en este [vínculo al área de trabajo de Log Analytics](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.OperationalInsights%2Fworkspaces) para acceder al área de trabajo en Azure Portal.

En Azure Portal, seleccione el área de trabajo de Log Analytics.

### <a name="log-queries"></a>Consultas de registros

Las consultas de registro ayudan a aprovechar al máximo el valor de los datos recopilados en registros de Azure Monitor. Las consultas de registro se usan para identificar los registros tanto en AppServiceHTTPLogs como en AppServiceConsoleLogs. Consulte [Introducción a las consultas de registro](../azure-monitor/log-query/log-query-overview.md) para más información sobre las consultas de registro.

### <a name="view-appservicehttplogs-with-log-query"></a>Visualización de AppServiceHTTPLogs con una consulta de registro

Ahora que hemos accedido a la aplicación, vamos a ver los datos asociados a las solicitudes HTTP, que se encuentran en `AppServiceHTTPLogs`.

1. Haga clic en `Logs` en el panel de navegación de la izquierda.

![Registros del área de trabajo de Log Analytics](./media/tutorial-azure-monitor/log-analytics-workspace-logs.png)

2. Busque `appservice` y haga doble clic en `AppServiceHTTPLogs`.

![Tablas del área de trabajo de Log Analytics](./media/tutorial-azure-monitor/log-analytics-workspace-app-service-tables.png)

3. Haga clic en `Run`.

![Registros HTTP de App Service del área de trabajo de Log Analytics](./media/tutorial-azure-monitor/log-analytics-workspace-app-service-http-logs.png)

La consulta `AppServiceHTTPLogs` devuelve todas las solicitudes de las últimas 24 horas. La columna `ScStatus` contiene el estado HTTP. Para diagnosticar los errores `HTTP 500`, limite `ScStatus` a 500 y ejecute la consulta, como se muestra a continuación:

```kusto
AppServiceHTTPLogs
| where ScStatus == 500
```

### <a name="view-appserviceconsolelogs-with-log-query"></a>Visualización de AppServiceConsoleLogs con una consulta de registro

Ahora que ha confirmado los errores HTTP 500, eche un vistazo a los errores y la salida estándar de la aplicación. Estos registros se encuentran en "AppServiceConsoleLogs".

(1) Haga clic en `+` para crear una nueva consulta. 

(2) Haga doble clic en la tabla `AppServiceConsoleLogs` y haga clic en `Run`. 

Dado que la conversión de cinco imágenes da como resultado errores de servidor, puede ver si la aplicación también escribe errores filtrando `ResultDescription` en busca de errores, como se muestra a continuación:

```kusto
AppServiceConsoleLogs |
where ResultDescription  contains "error" 
```

En la columna `ResultDescription`, verá el siguiente error:

<pre>
PHP Fatal error:  Allowed memory size of 134217728 bytes exhausted 
(tried to allocate 16384 bytes) in /home/site/wwwroot/process.php on line 20, 
referer: http://<app-name>.azurewebsites.net/
</pre>

### <a name="join-appservicehttplogs-and-appserviceconsolelogs"></a>Combinación de AppServiceHTTPLogs y AppServiceConsoleLogs

Ahora que ha identificado los errores HTTP 500 y los errores estándar, es preciso confirmar si hay una correlación entre estos mensajes. A continuación, se combinan las tablas en función de la marca de tiempo, `TimeGenerated`.

> [!NOTE]
> Se ha preparado una consulta que hace lo siguiente:
>
> - Filtra HTTPLogs en busca de errores 500
> - Consulta los registros de consola
> - Combina las tablas en función de `TimeGenerated`
>

Ejecute la siguiente consulta:

```kusto
let myHttp = AppServiceHTTPLogs | where  ScStatus == 500 | project TimeGen=substring(TimeGenerated, 0, 19), CsUriStem, ScStatus;  

let myConsole = AppServiceConsoleLogs | project TimeGen=substring(TimeGenerated, 0, 19), ResultDescription;

myHttp | join myConsole on TimeGen | project TimeGen, CsUriStem, ScStatus, ResultDescription;
```

En la columna `ResultDescription`, verá el siguiente error al mismo tiempo que los errores del servidor web:

<pre>
PHP Fatal error:  Allowed memory size of 134217728 bytes exhausted 
(tried to allocate 16384 bytes) in /home/site/wwwroot/process.php on line 20, 
referer: http://<app-name>.azurewebsites.net/
</pre>

El mensaje indica que se ha agotado la memoria en la línea 20 de `process.php`. Ahora ha confirmado que la aplicación generó un error durante el error HTTP 500. Echemos un vistazo al código para identificar el problema.

## <a name="identify-the-error"></a>Identificación del error

En el directorio local, abra el archivo `process.php` y mire en la línea 20. 

```php
imagepng($imgArray[$x], $filename);
```

El primer argumento, `$imgArray[$x]`, es una variable que contiene todos los archivos JPG (en memoria) que se deben convertir. Sin embargo, `imagepng` solo necesita la imagen que se va a convertir y no todas las imágenes. La carga previa de imágenes no es necesaria y puede causar el agotamiento de la memoria, lo que conduce a errores HTTP 500. Vamos a actualizar el código para cargar las imágenes a petición para ver si se resuelve el problema. A continuación, se mejorará el código para solucionar el problema de memoria.

## <a name="fix-the-app"></a>Corrección de la aplicación

### <a name="update-locally-and-redeploy-the-code"></a>Actualización local y nueva implementación del código

Realice los cambios siguientes en `process.php` para controlar el agotamiento de la memoria:

```php
<?php

//Retrieve query parameters
$maxImages = $_GET['images'];
$imgNames  = explode(",",$_GET['imgNames']);

//Load JPEGs into an array (in memory)
for ($x=0; $x<$maxImages; $x++){
    $filename = './images/converted_' . substr($imgNames[$x],0,-4) . '.png';
    imagepng(imagecreatefromjpeg("./images/" . $imgNames[$x]), $filename);
}
```

Confirme los cambios en Git y, después, inserte los cambios de código en Azure.

```bash
git commit -am "Load images on-demand in process.php"
git push azure master
```

### <a name="browse-to-the-azure-app"></a>Navegación hasta la aplicación de Azure

Vaya a `http://<app-name>.azurewebsites.net`. 

La conversión de imágenes no debe generar más errores HTTP 500.

![Aplicación PHP que se ejecuta en Azure App Service](./media/tutorial-azure-monitor/sample-monitor-app-working.png)

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

Elimine la configuración de diagnóstico con el siguiente comando:

```bash
az monitor diagnostic-settings delete --resource $resourceID -n myMonitorLogs
```
¿Qué ha aprendido?

> [!div class="checklist"]
> * Ha configurado una aplicación web con Azure Monitor
> * Ha enviado registros a Log Analytics
> * Ha utilizado consultas de registro para identificar y solucionar problemas de errores de la aplicación web

## <a name="next-steps"></a><a name="nextsteps"></a> Pasos siguientes
* [Consulta de registros con Azure Monitor](../azure-monitor/log-query/log-query-overview.md)
* [Troubleshooting Azure App Service in Visual Studio](troubleshoot-dotnet-visual-studio.md) (Solución de problemas de Azure App Service en Visual Studio)
* [Analyze app Logs in HDInsight](https://gallery.technet.microsoft.com/scriptcenter/Analyses-Windows-Azure-web-0b27d413) (Análisis de registros de aplicación en HDInsight)