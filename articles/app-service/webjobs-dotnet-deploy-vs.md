---
title: Desarrollo e implementación de WebJobs mediante Visual Studio
description: Aprenda a desarrollar Azure WebJobs en Visual Studio y a implementarlos en Azure App Service, incluida la creación de una tarea programada.
author: ggailey777
ms.assetid: a3a9d320-1201-4ac8-9398-b4c9535ba755
ms.topic: conceptual
ms.custom: vs-azure
ms.date: 02/18/2019
ms.author: glenga
ms.reviewer: david.ebbo;suwatch;pbatum;naren.soni
ms.openlocfilehash: cf2437443350cf9b21b6abc0d0dd1dbd5d22e41f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083198"
---
# <a name="develop-and-deploy-webjobs-using-visual-studio---azure-app-service"></a>Desarrollo e implementación de WebJobs mediante Visual Studio - Azure App Service

En este artículo, se explica cómo usar Visual Studio para implementar un proyecto de aplicación de consola en una aplicación web de [App Service](overview.md) como [WebJob de Azure](https://go.microsoft.com/fwlink/?LinkId=390226). Para información sobre cómo implementar WebJobs con [Azure Portal](https://portal.azure.com), consulte [Ejecución de tareas en segundo plano con WebJobs](webjobs-create.md).

Puede publicar varios trabajos web en una sola aplicación web. Asegúrese de que cada uno de los trabajos web de una aplicación web tenga un nombre único.

La versión 3.x del [SDK de Azure WebJobs](webjobs-sdk-how-to.md) permite implementar trabajos web que se ejecutan como aplicaciones de .NET Core o como aplicaciones de .NET Framework, mientras que la versión 2.x solo es compatible con .NET Framework. El modo en que se implementa un proyecto de WebJobs es diferente en el caso de los proyectos de .NET Core y de los proyectos de .NET Framework.

## <a name="webjobs-as-net-core-console-apps"></a>Trabajos web como aplicaciones de consola de .NET

Si usa la versión 3.x de WebJobs, puede crear y publicar trabajos web como aplicaciones de consola de .NET Core. Si desea obtener instrucciones paso a paso para crear y publicar una aplicación de consola de .NET Core en Azure como un trabajo web, consulte [Introducción al SDK de Azure WebJobs para el procesamiento en segundo plano basado en eventos](webjobs-sdk-get-started.md).

> [!NOTE]
> Los trabajos web de .NET Core no se pueden vincular con proyectos web. Si necesita implementar el trabajo web con una aplicación web, debe [crearlo como una aplicación de consola de .NET Framework](#webjobs-as-net-framework-console-apps).  

### <a name="deploy-to-azure-app-service"></a>Implementación en Azure App Service

Para publicar un trabajo web de .NET Core en App Service desde Visual Studio, se utilizan las mismas herramientas que para publicar una aplicación de ASP.NET Core.

[!INCLUDE [webjobs-publish-net-core](../../includes/webjobs-publish-net-core.md)] 

### <a name="webjob-types"></a>Tipos de WebJob

De forma predeterminada, los trabajos web publicados desde un proyecto de consola de .NET Core solamente se ejecutan cuando se desencadenan o a petición. También puede actualizar el proyecto para que se [ejecute en función de una programación](#scheduled-execution) o de forma continuada.

[!INCLUDE [webjobs-alwayson-note](../../includes/webjobs-always-on-note.md)]

#### <a name="scheduled-execution"></a>Ejecución programada

Cuando se publica una aplicación de consola de .NET Core en Azure, se agrega un nuevo archivo *settings.job* al proyecto. Utilice este archivo para programar la ejecución del trabajo web. Para obtener más información, consulte [Programar un trabajo Web desencadenado](#scheduling-a-triggered-webjob).

#### <a name="continuous-execution"></a>Ejecución continua

Puede usar Visual Studio para cambiar el trabajo web de forma que se ejecute continuamente cuando Always On esté habilitado en Azure.

1. Si todavía no lo ha hecho, [publique el proyecto en Azure](#deploy-to-azure-app-service).

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Publicar**.

1. En la pestaña **Publicar**, seleccione **Configuración**. 

1. En el cuadro de diálogo **Configuración de perfil**, seleccione **Continua** en **Tipo de WebJob**  y haga clic en **Guardar**.

    ![Cuadro de diálogo de configuración de publicación de un trabajo web](./media/webjobs-dotnet-deploy-vs/publish-settings.png)

1. Seleccione **Publicar** para volver a publicar el trabajo web con la configuración actualizada.

## <a name="webjobs-as-net-framework-console-apps"></a>Trabajos web como aplicaciones de consola de .NET Framework  

Cuando Visual Studio implementa un proyecto de aplicación de consola de .NET Framework habilitado para WebJobs, copia los archivos del entorno de ejecución en la carpeta apropiada de la aplicación web (*App_Data/jobs/continuous* para los trabajos web continuos y *App_Data/jobs/triggered* para los trabajos web programados y a petición).

Un proyecto con funcionalidad WebJobs tiene los siguientes elementos agregados:

* El paquete de NuGet [Microsoft.Web.WebJobs.Publish](https://www.nuget.org/packages/Microsoft.Web.WebJobs.Publish/) .
* Un archivo [webjob-publish-settings.json](#publishsettings) que contiene configuración de implementación y del programador. 

![Diagram showing what is added to a Console App to enable deployment as a WebJob](./media/webjobs-dotnet-deploy-vs/convert.png)

Puede agregar estos elementos a un proyecto de aplicación de consola existente o usar una plantilla para crear un nuevo proyecto de aplicación de consola con funcionalidad WebJobs. 

Puede implementar un proyecto como un WebJob por sí mismo o vincularlo a un proyecto web que se implemente automáticamente siempre que implemente el proyecto web. Para vincular proyectos, Visual Studio incluye el nombre del proyecto con funcionalidad WebJob en un archivo [webjobs-list.json](#webjobslist) en el proyecto web.

![Diagram showing WebJob project linking to web project](./media/webjobs-dotnet-deploy-vs/link.png)

### <a name="prerequisites"></a>Prerrequisitos

Si usa Visual Studio 2015, instale el [Azure SDK para .NET (Visual Studio 2015)](https://azure.microsoft.com/downloads/).

Si usa Visual Studio 2017, instale la [carga de trabajo de desarrollo de Azure](https://docs.microsoft.com/visualstudio/install/install-visual-studio#step-4---choose-workloads).

### <a name="enable-webjobs-deployment-for-an-existing-console-application-project"></a><a id="convert"></a> Habilitación de la implementación de WebJobs para un proyecto de aplicación de consola existente

Tiene dos opciones:

* [Activación de la implementación automática con un proyecto web](#convertlink).

  Configure un proyecto de aplicación de consola existente de forma que se implemente automáticamente como un WebJob cuando implemente un proyecto web. Utilice esta opción cuando desee ejecutar su trabajo web en la misma aplicación web en la que ejecuta la aplicación web relacionada.

* [Activación de la implementación sin un proyecto web](#convertnolink).

  Configure un proyecto de aplicación de consola existente para implementar como WebJob por sí mismo, sin vínculo a un proyecto web. Use esta opción cuando desee ejecutar un trabajo web en una aplicación web por sí mismo, sin ninguna aplicación web ejecutándose en dicha aplicación web. Puede que desee hacer esto para poder escalar los recursos de su WebJob independientemente de los recursos de la aplicación web.

#### <a name="enable-automatic-webjobs-deployment-with-a-web-project"></a><a id="convertlink"></a> Activación de la implementación de WebJobs automática con un proyecto web

1. Haga clic con el botón derecho en el proyecto web en el **Explorador de soluciones** y luego haga clic en **Agregar** > **Proyecto existente como WebJob de Azure**.
   
    ![Proyecto existente como trabajo web de Azure](./media/webjobs-dotnet-deploy-vs/eawj.png)
   
    Aparecerá el cuadro de diálogo [Agregar WebJob de Azure](#configure) .
2. En la lista desplegable **Nombre de proyecto** , seleccione el proyecto de aplicación de consola para agregar como WebJob.
   
    ![Selecting project in Add Azure WebJob dialog](./media/webjobs-dotnet-deploy-vs/aaw1.png)
3. Complete el cuadro de diálogo [Agregar WebJob de Azure](#configure) y haga clic en **Aceptar**. 

#### <a name="enable-webjobs-deployment-without-a-web-project"></a><a id="convertnolink"></a> Activación de la implementación de WebJobs sin un proyecto web
1. Haga clic con el botón derecho en el proyecto de aplicación de consola en el **Explorador de soluciones** y haga clic en **Publicar como WebJob de Azure...** 
   
    ![Publicar como WebJob de Azure](./media/webjobs-dotnet-deploy-vs/paw.png)
   
    Aparecerá el cuadro de diálogo [Agregar WebJob de Azure](#configure) , con el proyecto seleccionado en el cuadro **Nombre de proyecto** .
2. Complete el cuadro de diálogo [Agregar WebJob de Azure](#configure) y haga clic en **Aceptar**.
   
   Aparece el asistente de **Publicación web** .  Si no quiere realizar la publicación inmediatamente, cierre el asistente. La configuración que ha escrito se guarda para cuando desee [implementar el proyecto](#deploy).

### <a name="create-a-new-webjobs-enabled-project"></a><a id="create"></a>Creación de un nuevo proyecto con funcionalidad WebJobs
Para crear un nuevo proyecto con funcionalidad WebJobs, puede usar la plantilla de proyecto de aplicación de consola y habilitar la implementación de WebJobs tal y como se explicó en la [sección anterior](#convert). Como alternativa, puede usar la plantilla para nuevos proyectos WebJobs:

* [Uso de la plantilla para nuevos proyectos de WebJobs para WebJob independiente](#createnolink)
  
    Cree un proyecto y configúrelo para implementarse por sí mismo como WebJob, sin vínculo a un proyecto web. Use esta opción cuando desee ejecutar un trabajo web en una aplicación web por sí mismo, sin ninguna aplicación web ejecutándose en dicha aplicación web. Puede que desee hacer esto para poder escalar los recursos de su WebJob independientemente de los recursos de la aplicación web.
* [Uso de la plantilla para nuevos proyectos de WebJobs para un WebJob vinculado a un proyecto web](#createlink)
  
    Cree un proyecto configurado para implementarse automáticamente como WebJob cuando se implementa un proyecto web en la misma solución. Utilice esta opción cuando desee ejecutar su trabajo web en la misma aplicación web en la que ejecuta la aplicación web relacionada.

> [!NOTE]
> La plantilla new-project de WebJobs instala automáticamente los paquetes NuGet e incluye código en *Program.cs* para el [SDK de WebJobs](https://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/getting-started-with-windows-azure-webjobs). Si no quiere usar el SDK de WebJobs, quite o cambie la instrucción `host.RunAndBlock` en *Program.cs*.
> 
> 

#### <a name="use-the-webjobs-new-project-template-for-an-independent-webjob"></a><a id="createnolink"></a> Uso de la plantilla para nuevos proyectos de WebJobs para WebJob independiente
1. Haga clic en **Archivo** > **Nuevo proyecto** y, en el cuadro de diálogo **Nuevo proyecto**, haga clic en **Nube** > **Azure WebJob (.NET Framework)** .
   
    ![New Project dialog showing WebJob template](./media/webjobs-dotnet-deploy-vs/np.png)
2. Siga las instrucciones mostradas anteriormente para [crear el proyecto de aplicación de consola para un proyecto de WebJobs independiente](#convertnolink).

#### <a name="use-the-webjobs-new-project-template-for-a-webjob-linked-to-a-web-project"></a><a id="createlink"></a> Uso de la plantilla para nuevos proyectos de WebJobs para un WebJob vinculado a un proyecto web
1. Haga clic con el botón derecho en el proyecto web en el **Explorador de soluciones** y haga clic en **Agregar** > **Nuevo proyecto de WebJob de Azure**.
   
    ![New Azure WebJob Project menu entry](./media/webjobs-dotnet-deploy-vs/nawj.png)
   
    Aparecerá el cuadro de diálogo [Agregar WebJob de Azure](#configure) .
2. Complete el cuadro de diálogo [Agregar WebJob de Azure](#configure) y haga clic en **Aceptar**.

### <a name="the-add-azure-webjob-dialog"></a><a id="configure"></a>Cuadro de diálogo Agregar WebJob de Azure
El cuadro de diálogo **Agregar Azure WebJob** le permite escribir el nombre de WebJob y ejecutar la configuración del modo de WebJob. 

![Add Azure WebJob dialog](./media/webjobs-dotnet-deploy-vs/aaw2.png)

Los campos de este cuadro de diálogo corresponden a los campos del cuadro de diálogo **Agregar WebJob** de Azure Portal. Para obtener más información, consulte [Ejecución de tareas en segundo plano con WebJobs](webjobs-create.md).

> [!NOTE]
> * Para obtener información sobre la implementación de línea de comandos, consulte [Activación de la línea de comandos o de la entrega continua de Azure WebJobs](https://azure.microsoft.com/blog/2014/08/18/enabling-command-line-or-continuous-delivery-of-azure-webjobs/).
> * Si implementa un WebJob y, a continuación, decide que desea cambiar el tipo de WebJob y volver a implementarlo, deberá eliminar el archivo *webjobs-publish-settings.json*. Esto hará que Visual Studio muestre de nuevo las opciones de publicación para que pueda cambiar el tipo de WebJob.
> * Si implementa un WebJob y más tarde cambia el modo de ejecución de continuo a no continuo o viceversa, Visual Studio crea un nuevo WebJob en Azure cuando vuelva a implementar. Si cambia otra configuración de programación pero deja el mismo modo de ejecución o cambia entre el modo Programado y Bajo demanda, Visual Studio actualiza el trabajo existente en lugar de crear uno nuevo.
> 
> 

### <a name="webjob-publish-settingsjson"></a><a id="publishsettings"></a>webjob-publish-settings.json
Cuando crea una aplicación de consola para la implementación de WebJobs, Visual Studio instala el paquete de NuGet [Microsoft.Web.WebJobs.Publish](https://www.nuget.org/packages/Microsoft.Web.WebJobs.Publish/) y almacena la información de programación en un archivo *webjob-publish-settings.json* en la carpeta *Properties* del proyecto WebJobs. A continuación se muestra un ejemplo de ese archivo:

```json
{
  "$schema": "http://schemastore.org/schemas/json/webjob-publish-settings.json",
  "webJobName": "WebJob1",
  "startTime": "null",
  "endTime": "null",
  "jobRecurrenceFrequency": "null",
  "interval": null,
  "runMode": "Continuous"
}
```

Puede editar este archivo directamente y Visual Studio proporciona IntelliSense. El archivo de esquema se almacena en [https://schemastore.org](http://schemastore.org/schemas/json/webjob-publish-settings.json) y se puede ver allí.  

### <a name="webjobs-listjson"></a><a id="webjobslist"></a>webjobs-list.json
Cuando vincule un proyecto con funcionalidad WebJobs a un proyecto web, Visual Studio almacenará el nombre del proyecto WebJobs en un archivo *webjobs-list.json* en la carpeta *Properties* del proyecto web. La lista puede contener varios proyectos de WebJobs, tal y como se muestra en el siguiente ejemplo:

```json
{
  "$schema": "http://schemastore.org/schemas/json/webjobs-list.json",
  "WebJobs": [
    {
      "filePath": "../ConsoleApplication1/ConsoleApplication1.csproj"
    },
    {
      "filePath": "../WebJob1/WebJob1.csproj"
    }
  ]
}
```

Puede editar este archivo directamente y Visual Studio proporciona IntelliSense. El archivo de esquema se almacena en [https://schemastore.org](http://schemastore.org/schemas/json/webjobs-list.json) y se puede ver allí.

### <a name="deploy-a-webjobs-project"></a><a id="deploy"></a>Implementación de un proyecto de WebJobs
Un proyecto WebJobs que ha vinculado a un proyecto web se implementa automáticamente con este último. Para información sobre la implementación de proyectos web, consulte las **guías** > **Implementar aplicación** del panel de navegación izquierdo.

Para implementar un proyecto de WebJobs por sí mismo, haga clic con el botón derecho en **Explorador de soluciones** y luego haga clic en **Publicar como Azure WebJob...** 

![Publicar como WebJob de Azure](./media/webjobs-dotnet-deploy-vs/paw.png)

Para un trabajo web independiente, aparece el mismo asistente **Publicación web** que se usa para los proyectos web, pero con menos configuraciones disponibles para cambiar.

## <a name="scheduling-a-triggered-webjob"></a>Programación de un trabajo web desencadenado

WebJobs utiliza un archivo *settings.job* para determinar cuándo se ejecuta un trabajo web. Utilice este archivo para programar la ejecución del trabajo web. El siguiente ejemplo se ejecuta cada hora de 9: 00 a 17:00:

```json
{
    "schedule": "0 0 9-17 * * *"
}
```

Este archivo debe estar ubicado en la raíz de la carpeta de WebJobs, junto con el script de WebJob; por ejemplo, `wwwroot\app_data\jobs\triggered\{job name}` o `wwwroot\app_data\jobs\continuous\{job name}`. Cuando implemente un WebJob desde Visual Studio, marque las propiedades del archivo `settings.job` como **Copiar si es posterior**. 

Si [crea un trabajo web en Azure Portal](webjobs-create.md), el archivo settings.job se crea automáticamente.

[!INCLUDE [webjobs-alwayson-note](../../includes/webjobs-always-on-note.md)]

### <a name="cron-expressions"></a>Expresiones CRON

WebJobs usa las mismas expresiones CRON para realizar la programación que el desencadenador de temporizador de Azure Functions. Para obtener más información sobre la compatibilidad con CRON, consulte [Desencadenador de temporizador para Azure Functions](../azure-functions/functions-bindings-timer.md#ncrontab-expressions).

[!INCLUDE [webjobs-cron-timezone-note](../../includes/webjobs-cron-timezone-note.md)]

### <a name="settingsjob-reference"></a>Referencia de settings.job

WebJobs admite las siguientes opciones de configuración:

| **Configuración** | **Tipo**  | **Descripción** |
| ----------- | --------- | --------------- |
| `is_in_place` | All | Permite que el trabajo se ejecute donde está, sin necesidad de copiarlo primero en una carpeta temporal. Para más información, consulte [WebJobs working directory](https://github.com/projectkudu/kudu/wiki/WebJobs#webjob-working-directory) (Directorio de trabajo de WebJobs). |
| `is_singleton` | Continuo | Solo debe ejecutar los trabajos web en una única instancia cuando escale horizontalmente. Para más información, consulte [Set a continuous job as singleton](https://github.com/projectkudu/kudu/wiki/WebJobs-API#set-a-continuous-job-as-singleton) (Establecer un trabajo continuo como singleton). |
| `schedule` | Desencadenado | Ejecute el trabajo web utilizando una programación basada en CRON. Para más información, consulte el [artículo de referencia del desencadenador de temporizador](../azure-functions/functions-bindings-timer.md#ncrontab-expressions). |
| `stopping_wait_time`| All | Permite controlar el comportamiento de apagado. Para más información, consulte [Graceful shutdown](https://github.com/projectkudu/kudu/wiki/WebJobs#graceful-shutdown) (Cierre estable). |

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre el SDK de WebJobs](webjobs-sdk-how-to.md)
