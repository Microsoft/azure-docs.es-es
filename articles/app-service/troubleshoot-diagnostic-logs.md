---
title: Habilitación del registro de diagnósticos
description: Obtenga información acerca de cómo habilitar el registro de diagnóstico y agregar la instrumentación a su aplicación, así como la manera de acceder a la información registrada por Azure.
ms.assetid: c9da27b2-47d4-4c33-a3cb-1819955ee43b
ms.topic: article
ms.date: 09/17/2019
ms.custom: devx-track-csharp, seodec18
ms.openlocfilehash: 89162a0b8ca20e59319802f9e2359c2f27ff163f
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962186"
---
# <a name="enable-diagnostics-logging-for-apps-in-azure-app-service"></a>Habilitar el registro de diagnósticos para las aplicaciones de Azure App Service
## <a name="overview"></a>Información general
Azure integra diagnósticos para ayudar a depurar [aplicaciones de App Service](overview.md). En este artículo se ofrece información acerca de cómo habilitar el registro de diagnóstico, agregar instrumentación a la aplicación y obtener acceso a la información que registra Azure.

En este artículo se usa [Azure Portal](https://portal.azure.com) y la CLI de Azure para trabajar con registros de diagnóstico. Para obtener información acerca de cómo trabajar con registros de diagnóstico mediante Visual Studio, consulte [Solución de problemas de Azure en Visual Studio](troubleshoot-dotnet-visual-studio.md).

> [!NOTE]
> Además de las instrucciones de registro de este artículo, hay una nueva funcionalidad de registro integrada con Supervisión de Azure. Encontrará más información sobre esta funcionalidad en la sección [Envío de registros a Azure Monitor (versión preliminar)](#send-logs-to-azure-monitor-preview). 
>
>

|Tipo|Plataforma|Location|Descripción|
|-|-|-|-|
| Registro de aplicaciones | Windows, Linux | Sistema de archivos de App Service o blobs de Azure Storage | Registra los mensajes generados por el código de aplicación. Los mensajes se pueden generar en el marco web que elija o directamente desde el código de aplicación mediante el patrón de registro estándar del lenguaje. A cada mensaje se le asigna una de las siguientes categorías: **Crítico**, **Error**, **Advertencia**, **Información**, **Depuración** y **Seguimiento**. Puede seleccionar el grado de detalle que quiere que tenga el registro; para ello, establezca el nivel de gravedad al habilitar el registro de la aplicación.|
| Registro del servidor web| Windows | Sistema de archivos de App Service o blobs de Azure Storage| Datos de solicitud HTTP sin procesar en el [formato de archivo de registro extendido W3C](/windows/desktop/Http/w3c-logging). Cada mensaje de registro incluye datos como el método HTTP, el URI del recurso, la dirección IP del cliente, el puerto del cliente, el agente de usuario, el código de respuesta, etc. |
| Mensajes de error detallados| Windows | Sistema de archivos de App Service | Copias de las páginas de error *.htm* que se habrían enviado al explorador del cliente. Por motivos de seguridad, no se deben enviar páginas de error detalladas a los clientes en producción, pero App Service puede guardar la página de error cada vez que se produzca un error de aplicación que tenga el código HTTP 400 o superior. La página puede contener información útil para determinar por qué el servidor devuelve el código de error. |
| Seguimiento de solicitudes con error | Windows | Sistema de archivos de App Service | Información de seguimiento detallada sobre las solicitudes con error, lo que incluye un seguimiento de los componentes de IIS usados para procesar la solicitud y el tiempo dedicado a cada componente. Resulta útil si desea mejorar el rendimiento del sitio o aislar un error HTTP específico. Se genera una carpeta para cada solicitud con error, que contiene el archivo de registro XML y la hoja de estilos XSL con la que ver el archivo de registro. |
| Registro de implementación | Windows, Linux | Sistema de archivos de App Service | Registros al publicar contenido en una aplicación. El registro de implementación tiene lugar automáticamente, no hay valores configurables. Ayuda a determinar por qué no se realizó una implementación. Por ejemplo, si usa un [script de implementación personalizado](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script), puede usar el registro de implementación para determinar por qué el script da error. |

> [!NOTE]
> App Service proporciona una herramienta de diagnóstico interactiva dedicada para ayudarle a solucionar problemas de su aplicación. Para más información, consulte [Introducción a los diagnósticos de Azure App Service](overview-diagnostics.md).
>
> Además, puede usar otros servicios de Azure para mejorar las funcionalidades de registro y supervisión de la aplicación, como [Azure Monitor](../azure-monitor/app/azure-web-apps.md).
>

## <a name="enable-application-logging-windows"></a>Habilitación del registro de aplicaciones (Windows)

> [!NOTE]
> El registro de aplicaciones para Blob Storage solo puede usar cuentas de almacenamiento de la misma región que App Service.

Para habilitar el registro de aplicaciones para aplicaciones Windows, en [Azure Portal](https://portal.azure.com), vaya a la aplicación y seleccione **Registros de App Service**.

Seleccione **Activado** en **Registro de la aplicación (sistema de archivos)** o **Registro de la aplicación (Blob)** , o en ambos. 

La opción **Sistema de archivos** es para fines de depuración temporales y se desactiva en 12 horas. La opción **Blob** es para el registro a largo plazo y necesita un contenedor de almacenamiento de blobs en el que escribir los registros.  La opción **Blob** también incluye información adicional en los mensajes de registro, como el identificador de la instancia de máquina virtual de origen del mensaje de registro (`InstanceId`), el identificador de subproceso (`Tid`) y una marca de tiempo más pormenorizada ([`EventTickCount`](/dotnet/api/system.datetime.ticks)).

> [!NOTE]
> Actualmente, solo los registros de aplicación de .NET pueden escribirse en el almacenamiento de blobs. Los registros de aplicaciones de Java, PHP, Node.js y Python solo se pueden almacenar en el sistema de archivos de App Service (sin modificaciones de código para escribir registros en almacenamiento externo).
>
> Además, si [regenera las claves de acceso de su cuenta de almacenamiento](../storage/common/storage-account-create.md), deberá restablecer la configuración de registro correspondiente para usar las claves de acceso actualizadas. Para ello, siga estos pasos:
>
> 1. En la pestaña **Configurar**, establezca la característica de registro correspondiente de **Desactivar**. Guarde la configuración.
> 2. Vuelva a habilitar el registro en el blob de la cuenta de almacenamiento. Guarde la configuración.
>
>

Seleccione el nivel o el **nivel** de detalle que quiere registrar. En la tabla siguiente se muestran las categorías de registro incluidas en cada nivel:

| Nivel | Categorías incluidas |
|-|-|
|**Deshabilitada** | None |
|**Error** | Error, Crítico |
|**Warning (ADVERTENCIA)** | Advertencia, Error, Crítico|
|**Información** | Información, Advertencia, Error, Crítico|
|**Detallado** | Seguimiento, Depurar, Información, Advertencia, Error, Crítico (todas las categorías) |

Cuando termine, seleccione **Guardar**.

## <a name="enable-application-logging-linuxcontainer"></a>Habilitación del registro de aplicaciones (Linux o contenedor)

Para habilitar el registro de aplicaciones para aplicaciones Linux o aplicaciones de contenedor personalizadas, en [Azure Portal](https://portal.azure.com), vaya a la aplicación y seleccione **Registros de App Service**.

En **Registro de aplicaciones**, seleccione **Sistema de archivos**.

En **Cuota (MB)** , especifique la cuota de disco para los registros de aplicaciones. En **Período de retención (días)** , establezca el número de días que se deben conservar los registros.

Cuando termine, seleccione **Guardar**.

## <a name="enable-web-server-logging"></a>Habilitar el registro de servidor web

Para habilitar el registro de servidor web para las aplicaciones Windows, en [Azure Portal](https://portal.azure.com), vaya a la aplicación y seleccione **Registros de App Service**.

Para el **registro de servidor web**, seleccione **Almacenamiento** para almacenar los registros en Blob Storage, o **Sistema de archivos** para almacenar los registros en el sistema de archivos de App Service. 

En **Período de retención (días)** , establezca el número de días que se deben conservar los registros.

> [!NOTE]
> Si se [regeneran las claves de acceso de su cuenta de almacenamiento](../storage/common/storage-account-create.md), deberá restablecer la configuración de registro correspondiente para usar las claves actualizadas. Para ello, siga estos pasos:
>
> 1. En la pestaña **Configurar**, establezca la característica de registro correspondiente de **Desactivar**. Guarde la configuración.
> 2. Vuelva a habilitar el registro en el blob de la cuenta de almacenamiento. Guarde la configuración.
>
>

Cuando termine, seleccione **Guardar**.

## <a name="log-detailed-errors"></a>Registro de errores detallados

Para guardar la página de error o el seguimiento de las solicitudes con error para las aplicaciones Windows, en [Azure Portal](https://portal.azure.com), vaya a la aplicación y seleccione **Registros de App Service**.

En **Registro de error detallado** o **Error del seguimiento de solicitudes**, seleccione **Activado**, y, luego, elija **Guardar**.

Ambos tipos de registros se almacenan en el sistema de archivos de App Service. Se conservan hasta 50 errores (archivos o carpetas). Cuando el número de archivos HTML supera este límite, se eliminan automáticamente los 26 errores más antiguos.

## <a name="add-log-messages-in-code"></a>Adición de mensajes de registro en el código

En el código de la aplicación, se usan las funciones de registro habituales para enviar mensajes de registro a los registros de aplicaciones. Por ejemplo:

- Las aplicaciones de ASP.NET pueden usar la clase [System.Diagnostics.Trace](/dotnet/api/system.diagnostics.trace) para registrar información en el registro de diagnóstico de la aplicación. Por ejemplo:

    ```csharp
    System.Diagnostics.Trace.TraceError("If you're seeing this, something bad happened");
    ```

- De forma predeterminada, ASP.NET Core usa el proveedor de registro [Microsoft.Extensions.Logging.AzureAppServices](https://www.nuget.org/packages/Microsoft.Extensions.Logging.AzureAppServices). Para más información, consulte el artículo sobre el [registro de ASP.NET Core en Azure](/aspnet/core/fundamentals/logging/).

## <a name="stream-logs"></a>Transmisión de registros

Antes de transmitir registros en tiempo real, habilite el tipo de registro que quiera. Toda la información escrita en los archivos terminados en .txt, .log o .htm que se almacena en el directorio */LogFiles* (d:/home/logfiles) se transmite mediante App Service.

> [!NOTE]
> Algunos tipos de búfer de registros se escriben en el archivo de registro, lo que puede ocasionar la transmisión de eventos desordenados. Por ejemplo, una entrada de registro de aplicaciones que se genera cuando un usuario visita una página se puede visualizar en la transmisión antes de la entrada de registro HTTP correspondiente para la solicitud de la página.
>

### <a name="in-azure-portal"></a>En Azure Portal

Para transmitir registros, en [Azure Portal](https://portal.azure.com), vaya a la aplicación y seleccione **Secuencia de registro**. 

### <a name="in-cloud-shell"></a>En Cloud Shell

Para transmitir registros en directo en [Cloud Shell](../cloud-shell/overview.md), use el siguiente comando:

```azurecli-interactive
az webapp log tail --name appname --resource-group myResourceGroup
```

Para filtrar eventos específicos, como errores, use el parámetro **--Filter** . Por ejemplo:

```azurecli-interactive
az webapp log tail --name appname --resource-group myResourceGroup --filter Error
```
Para filtrar tipos de registros específicos, como HTTP, use el parámetro **--Path** . Por ejemplo:

```azurecli-interactive
az webapp log tail --name appname --resource-group myResourceGroup --path http
```

### <a name="in-local-terminal"></a>En el terminal local

Para transmitir registros en la consola local, [instale la CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión en su cuenta](/cli/azure/authenticate-azure-cli). Cuando haya iniciado sesión, siga las [instrucciones para Cloud Shell](#in-cloud-shell).

## <a name="access-log-files"></a>Acceso a los archivos de registro

Si configura la opción de blobs de Azure Storage para un tipo de registro, necesitará una herramienta de cliente que funcione con Azure Storage. Para más información, consulte [Herramientas de cliente de Azure Storage](../storage/common/storage-explorers.md).

En el caso de los registros almacenados en el sistema de archivos de App Service, la manera más fácil es descargar el archivo ZIP en el explorador en:

- Aplicaciones Linux o de contenedor: `https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip`
- Aplicaciones Windows: `https://<app-name>.scm.azurewebsites.net/api/dump`

En el caso de las aplicaciones Linux o de contenedor, el archivo ZIP contiene registros de salida de la consola para el host de Docker y el contenedor de Docker. En el caso de las aplicaciones escaladas horizontalmente, el archivo ZIP contiene un conjunto de registros para cada instancia. En el sistema de archivos de App Service, estos archivos de registro son el contenido del directorio */home/LogFiles*.

En el caso de las aplicaciones Windows, el archivo ZIP incluye el contenido del directorio *D:\Home\LogFiles* en el sistema de archivos de App Service. Tiene la siguiente estructura:

| Tipo de registro | Directorio | Descripción |
|-|-|-|
| **Registros de aplicaciones** |*/LogFiles/Application/* | Contiene uno o varios archivos de texto. El formato de los mensajes de registro depende del proveedor de registro que se use. |
| **Seguimiento de solicitudes con error** | */LogFiles/W3SVC#########/* | Contiene archivos XML y un archivo XSL. Puede ver los archivos XML con formato en el explorador. |
| **Registros de errores detallados** | */LogFiles/DetailedErrors/* | Contiene archivos de error HTM. Puede ver los archivos HTM en el explorador.<br/>Otra manera sencilla de ver los seguimientos de las solicitudes con error consiste en ir a la página de la aplicación en el portal. En el menú izquierdo, seleccione **Diagnosticar y solucionar problemas**, busque **registros de seguimiento de solicitudes erróneas** y haga clic en el icono para examinar y ver el seguimiento que desee. |
| **Registros de servidor web** | */LogFiles/http/RawLogs/* | Contiene archivos de texto con [formato de archivo de registro extendido W3C](/windows/desktop/Http/w3c-logging). Esta información se puede leer con un editor de texto o una utilidad como [Log Parser](https://go.microsoft.com/fwlink/?LinkId=246619).<br/>App Service no admite los campos `s-computername`, `s-ip` ni `cs-version`. |
| **Registros de implementación** | */LogFiles/Git/* y */deployments/* | Contienen registros generados por los procesos de implementación internos, así como registros para implementaciones de Git. |

## <a name="send-logs-to-azure-monitor-preview"></a>Envío de registros a Azure Monitor (versión preliminar)

Con la nueva [integración de Azure Monitor](https://aka.ms/appsvcblog-azmon), puede [crear configuraciones de diagnóstico (versión preliminar)](https://azure.github.io/AppService/2019/11/01/App-Service-Integration-with-Azure-Monitor.html#create-a-diagnostic-setting) para enviar registros a cuentas de almacenamiento, Event Hubs y Log Analytics. 

> [!div class="mx-imgBorder"]
> ![Configuración de diagnóstico (versión preliminar)](media/troubleshoot-diagnostic-logs/diagnostic-settings-page.png)

### <a name="supported-log-types"></a>Tipos de registro admitidos

En la tabla siguiente se muestran las descripciones y los tipos de registros admitidos: 

| Tipo de registro | Soporte técnico de Windows | Compatibilidad con Linux (Docker) | Descripción |
|-|-|-|
| AppServiceConsoleLogs | TBA | Sí | Salida estándar y error estándar |
| AppServiceHTTPLogs | Sí | Sí | Registros de servidor web |
| AppServiceEnvironmentPlatformLogs | Sí | Sí | App Service Environment: escalado, cambios de configuración y registros de estado|
| AppServiceAuditLogs | Sí | Sí | Actividad de inicio de sesión a través de FTP y KUDU |
| AppServiceFileAuditLogs | Sí | TBD | Cambios de archivos a través de FTP y KUDU |
| AppServiceAppLogs | TBA | Java SE y Tomcat | Registros de aplicación |
| AppServiceIPSecAuditLogs  | Sí | Sí | Solicitudes de reglas IP |
| AppServicePlatformLogs  | TBA | Sí | Registros de contenedor |

## <a name="next-steps"></a><a name="nextsteps"></a> Pasos siguientes
* [Consulta de registros con Azure Monitor](../azure-monitor/log-query/log-query-overview.md)
* [Control de Supervisión de aplicaciones en Azure App Service](web-sites-monitor.md)
* [Troubleshooting Azure App Service in Visual Studio](troubleshoot-dotnet-visual-studio.md) (Solución de problemas de Azure App Service en Visual Studio)
* [Analyze app Logs in HDInsight](https://gallery.technet.microsoft.com/scriptcenter/Analyses-Windows-Azure-web-0b27d413) (Análisis de registros de aplicación en HDInsight)