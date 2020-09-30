---
title: Desarrollo e implementación de WebJobs mediante Visual Studio
description: Aprenda a desarrollar Azure WebJobs en Visual Studio y a implementarlos en Azure App Service, incluida la creación de una tarea programada.
author: ggailey777
ms.assetid: a3a9d320-1201-4ac8-9398-b4c9535ba755
ms.topic: conceptual
ms.custom: devx-track-csharp, vs-azure
ms.date: 07/30/2020
ms.author: glenga
ms.reviewer: david.ebbo;suwatch;pbatum;naren.soni
ms.openlocfilehash: de10903be86b52b3415b57a53be81e7fd1661f63
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226036"
---
# <a name="develop-and-deploy-webjobs-using-visual-studio"></a>Desarrollo e implementación de WebJobs mediante Visual Studio

En este artículo, se explica cómo usar Visual Studio para implementar un proyecto de aplicación de consola en una aplicación web de [App Service](overview.md) como [WebJob de Azure](https://go.microsoft.com/fwlink/?LinkId=390226). Para información sobre cómo implementar WebJobs con [Azure Portal](https://portal.azure.com), consulte [Ejecución de tareas en segundo plano con WebJobs en Azure App Service](webjobs-create.md).

Puede elegir desarrollar un WebJob que se ejecute como una [aplicación .NET Core](#webjobs-as-net-core-console-apps) o una [aplicación .NET Framework](#webjobs-as-net-framework-console-apps). La versión 3.x del [SDK de Azure WebJobs](webjobs-sdk-how-to.md) permite implementar trabajos web que se ejecutan como aplicaciones de .NET Core o como aplicaciones de .NET Framework, mientras que la versión 2.x solo es compatible con .NET Framework. El modo en que se implementa un proyecto de WebJobs es diferente según sea un proyecto de .NET Core o un proyecto de .NET Framework.

Puede publicar varios WebJobs en una sola aplicación web, siempre que cada WebJob de una aplicación web tenga un nombre único.

## <a name="webjobs-as-net-core-console-apps"></a>Trabajos web como aplicaciones de consola de .NET

Con la versión 3.x del SDK de Azure WebJobs, puede crear y publicar WebJobs como aplicaciones de consola de .NET Core. Puede encontrar instrucciones paso a paso para crear y publicar una aplicación de consola de .NET Core en Azure como un WebJob en [Introducción al SDK de Azure WebJobs para el procesamiento en segundo plano basado en eventos](webjobs-sdk-get-started.md).

> [!NOTE]
> Los WebJobs de .NET Core no se pueden vincular con proyectos web. Si necesita implementar el WebJob con una aplicación web, debe [crearlo como una aplicación de consola de .NET Framework](#webjobs-as-net-framework-console-apps).  

### <a name="deploy-to-azure-app-service"></a>Implementación en Azure App Service

Para publicar un WebJob de .NET Core en Azure App Service desde Visual Studio, se utilizan las mismas herramientas que para publicar una aplicación de ASP.NET Core.

[!INCLUDE [webjobs-publish-net-core](../../includes/webjobs-publish-net-core.md)] 

## <a name="webjobs-as-net-framework-console-apps"></a>Trabajos web como aplicaciones de consola de .NET Framework  

Si usa Visual Studio para implementar un proyecto de aplicación de consola de .NET Framework habilitado para WebJobs, los archivos del entorno de ejecución se copian en la carpeta apropiada de la aplicación web (*App_Data/jobs/continuous* para WebJobs continuos y *App_Data/jobs/triggered* para WebJobs programados o a petición).

Visual Studio agrega los siguientes elementos a los proyectos habilitados para WebJobs:

* El paquete de NuGet [Microsoft.Web.WebJobs.Publish](https://www.nuget.org/packages/Microsoft.Web.WebJobs.Publish/) .
* Un archivo [webjob-publish-settings.json](#publishsettings) que contiene configuración de implementación y del programador. 

![Diagrama que muestra lo que se agrega a una aplicación de consola para habilitar la implementación como un WebJob](./media/webjobs-dotnet-deploy-vs/convert.png)

Puede agregar estos elementos a un proyecto de aplicación de consola existente o usar una plantilla para crear un nuevo proyecto de aplicación de consola con funcionalidad WebJobs. 

Implemente un proyecto como un WebJob por sí mismo o vincúlelo a un proyecto web que se implemente automáticamente siempre que implemente el proyecto web. Para vincular proyectos, Visual Studio incluye el nombre del proyecto con funcionalidad WebJob en un archivo [webjobs-list.json](#webjobslist) en el proyecto web.

![Diagram showing WebJob project linking to web project](./media/webjobs-dotnet-deploy-vs/link.png)

### <a name="prerequisites"></a>Prerrequisitos

Instale Visual Studio 2017 o Visual Studio 2019 con la [carga de trabajo de desarrollo de Azure](/visualstudio/install/install-visual-studio#step-4---choose-workloads).

### <a name="enable-webjobs-deployment-for-an-existing-console-app-project"></a><a id="convert"></a> Habilitación de la implementación de WebJobs para un proyecto de aplicación de consola existente

Tiene dos opciones:

* [Activación de la implementación automática con un proyecto web](#convertlink).

  Configure un proyecto de aplicación de consola existente de forma que se implemente automáticamente como un WebJob cuando se implemente un proyecto web. Utilice esta opción cuando desee ejecutar su trabajo web en la misma aplicación web en la que ejecuta la aplicación web relacionada.

* [Activación de la implementación sin un proyecto web](#convertnolink).

  Configure un proyecto de aplicación de consola existente para implementarlo como un WebJob por sí solo, sin vínculo a un proyecto web. Use esta opción cuando desee ejecutar un trabajo web en una aplicación web por sí mismo, sin ninguna aplicación web ejecutándose en dicha aplicación web. Puede que quiera hacerlo para escalar los recursos del WebJob de forma independiente de los recursos de la aplicación web.

#### <a name="enable-automatic-webjobs-deployment-with-a-web-project"></a><a id="convertlink"></a> Activación de la implementación de WebJobs automática con un proyecto web

1. Haga clic con el botón derecho en el proyecto web en el **Explorador de soluciones** y, luego, seleccione **Agregar** > **Proyecto existente como WebJob de Azure**.
   
    ![Proyecto existente como trabajo web de Azure](./media/webjobs-dotnet-deploy-vs/eawj.png)
   
    Aparecerá el cuadro de diálogo [Agregar WebJob de Azure](#configure) .
2. En la lista desplegable **Nombre de proyecto**, seleccione el proyecto de aplicación de consola para agregar como WebJob.
   
    ![Selección del proyecto en el cuadro de diálogo Agregar Azure WebJob](./media/webjobs-dotnet-deploy-vs/aaw1.png)
3. Complete el cuadro de diálogo [Agregar Azure WebJob](#configure) y haga clic en **Aceptar**. 

#### <a name="enable-webjobs-deployment-without-a-web-project"></a><a id="convertnolink"></a> Activación de la implementación de WebJobs sin un proyecto web
1. Haga clic con el botón derecho en el proyecto de aplicación de consola en el **Explorador de soluciones** y, luego, seleccione **Publicar como WebJob de Azure**. 
   
    ![Publicar como WebJob de Azure](./media/webjobs-dotnet-deploy-vs/paw.png)
   
    Aparecerá el cuadro de diálogo [Agregar WebJob de Azure](#configure) , con el proyecto seleccionado en el cuadro **Nombre de proyecto** .
2. Complete el cuadro de diálogo [Agregar Azure WebJob](#configure) y haga clic en **Aceptar**.
   
   Aparece el asistente de **Publicación web** . Si no quiere realizar la publicación inmediatamente, cierre el asistente. La configuración que ha escrito se guarda para cuando desee [implementar el proyecto](#deploy).

### <a name="create-a-new-webjobs-enabled-project"></a><a id="create"></a>Creación de un nuevo proyecto con funcionalidad WebJobs
Para crear un proyecto habilitado para WebJobs, use la plantilla de proyecto de aplicación de consola y habilite la implementación de WebJobs tal y como se explicó en la [sección anterior](#convert). Como alternativa, puede usar la plantilla para nuevos proyectos WebJobs:

* [Uso de la plantilla para nuevos proyectos de WebJobs para WebJob independiente](#createnolink)
  
    Cree un proyecto y configúrelo para implementarse por sí mismo como WebJob, sin vínculo a un proyecto web. Use esta opción cuando desee ejecutar un trabajo web en una aplicación web por sí mismo, sin ninguna aplicación web ejecutándose en dicha aplicación web. Puede que quiera hacerlo para escalar los recursos del WebJob de forma independiente de los recursos de la aplicación web.
* [Uso de la plantilla para nuevos proyectos de WebJobs para un WebJob vinculado a un proyecto web](#createlink)
  
    Cree un proyecto configurado para implementarse automáticamente como WebJob cuando se implementa un proyecto web en la misma solución. Utilice esta opción cuando desee ejecutar su trabajo web en la misma aplicación web en la que ejecuta la aplicación web relacionada.

> [!NOTE]
> La plantilla new-project de WebJobs instala automáticamente los paquetes NuGet e incluye código en *Program.cs* para el [SDK de WebJobs](https://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/getting-started-with-windows-azure-webjobs). Si no quiere usar el SDK de WebJobs, quite o cambie la instrucción `host.RunAndBlock` en *Program.cs*.
> 
> 

#### <a name="use-the-webjobs-new-project-template-for-an-independent-webjob"></a><a id="createnolink"></a> Uso de la plantilla para nuevos proyectos de WebJobs para WebJob independiente
1. Seleccione **File (Archivo)**  > **New (Nuevo)**  > **Project (Proyecto)** . En el cuadro de diálogo **Crear un proyecto nuevo**, busque y seleccione **Azure WebJob (.NET Framework)** para C#.
   
2. Siga las instrucciones anteriores para [convertir el proyecto de aplicación de consola en un proyecto de WebJobs independiente](#convertnolink).

#### <a name="use-the-webjobs-new-project-template-for-a-webjob-linked-to-a-web-project"></a><a id="createlink"></a> Uso de la plantilla para nuevos proyectos de WebJobs para un WebJob vinculado a un proyecto web
1. Haga clic con el botón derecho en el proyecto web en el **Explorador de soluciones** y, luego, seleccione **Agregar** > **Nuevo proyecto WebJob de Azure**.
   
    ![New Azure WebJob Project menu entry](./media/webjobs-dotnet-deploy-vs/nawj.png)
   
    Aparecerá el cuadro de diálogo [Agregar WebJob de Azure](#configure) .
2. Complete el cuadro de diálogo [Agregar Azure WebJob](#configure) y haga clic en **Aceptar**.


### <a name="webjob-publish-settingsjson-file"></a><a id="publishsettings"></a>Archivo webjob-publish-settings.json
Cuando se configura una aplicación de consola para la implementación de WebJobs, Visual Studio instala el paquete de NuGet [Microsoft.Web.WebJobs.Publish](https://www.nuget.org/packages/Microsoft.Web.WebJobs.Publish/) y almacena la información de programación en un archivo *webjob-publish-settings.json* en la carpeta *Propiedades* del proyecto de WebJobs. A continuación se muestra un ejemplo de ese archivo:

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

### <a name="webjobs-listjson-file"></a><a id="webjobslist"></a>Archivo webjobs-list.json
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

Este archivo se puede editar directamente en Visual Studio, con IntelliSense. El esquema del archivo se almacena en [https://schemastore.org](http://schemastore.org/schemas/json/webjobs-list.json).

### <a name="deploy-a-webjobs-project"></a><a id="deploy"></a>Implementación de un proyecto de WebJobs
Un proyecto de WebJobs que se haya vinculado a un proyecto web se implementa automáticamente con este último. Para información sobre la implementación de proyectos web, consulte **How-to guides** > **Deploy the app** (Guías de procedimientos > Implementar la aplicación) en el panel de navegación izquierdo.

Para implementar un proyecto de WebJobs por sí solo, haga clic con el botón derecho en el proyecto en el **Explorador de soluciones** y, luego, seleccione **Publicar como WebJob de Azure**. 

![Publicar como WebJob de Azure](./media/webjobs-dotnet-deploy-vs/paw.png)

Para un trabajo web independiente, aparece el mismo asistente **Publicación web** que se usa para los proyectos web, pero con menos configuraciones disponibles para cambiar.

### <a name="add-azure-webjob-dialog-box"></a><a id="configure"></a>Cuadro de diálogo Agregar Azure WebJob
El cuadro de diálogo **Agregar Azure WebJob** le permite escribir el nombre del WebJob y configurar el modo de ejecución de WebJob. 

![Cuadro de diálogo Agregar Azure WebJob](./media/webjobs-dotnet-deploy-vs/aaw2.png)

Algunos de los campos de este cuadro de diálogo corresponden a los campos del cuadro de diálogo **Agregar WebJob** de Azure Portal. Para más información, consulte [Ejecución de tareas en segundo plano con WebJobs en Azure App Service](webjobs-create.md).

Información de implementación de WebJob:

* Para obtener información sobre la implementación de línea de comandos, consulte [Activación de la línea de comandos o de la entrega continua de Azure WebJobs](https://azure.microsoft.com/blog/2014/08/18/enabling-command-line-or-continuous-delivery-of-azure-webjobs/).

* Si implementa un WebJob y posteriormente decide que quiere cambiar el tipo de WebJob y volver a implementarlo, elimine el archivo *webjobs-publish-settings.json*. Al hacerlo, Visual Studio vuelve a mostrar las opciones de publicación, de forma que puede cambiar el tipo de WebJob.

* Si implementa un WebJob y más tarde cambia el modo de ejecución de continuo a no continuo o viceversa, Visual Studio crea un nuevo WebJob en Azure cuando vuelva a implementar. Si cambia otra configuración de programación, pero deja el mismo modo de ejecución o cambia entre el modo Programado y A petición, Visual Studio actualiza el trabajo existente en lugar de crear uno nuevo.

## <a name="webjob-types"></a>Tipos de WebJob

El tipo de un WebJob puede ser *desencadenado* o *continuo*:

- Desencadenado (valor predeterminado): los WebJobs desencadenados se inician en función de un evento de enlace, según una [programación](#scheduling-a-triggered-webjob), o cuando se desencadenan manualmente (a petición). Se ejecuta en una sola instancia en la que se ejecuta la aplicación web.

- Continuo: los WebJobs [continuos](#continuous-execution) se inician inmediatamente cuando se crea el WebJob. Se ejecuta en todas las instancias escaladas de la aplicación web de forma predeterminada, pero se puede configurar para que se ejecute como una instancia única a través de *settings.job*.

[!INCLUDE [webjobs-alwayson-note](../../includes/webjobs-always-on-note.md)]

### <a name="scheduling-a-triggered-webjob"></a>Programación de un trabajo web desencadenado

Al publicar una aplicación de consola en Azure, Visual Studio establece el tipo de WebJob en **Desencadenado** de forma predeterminada, y agrega un nuevo archivo *settings.job* al proyecto. En el caso de los tipos de WebJob desencadenados, puede usar este archivo para establecer una programación de ejecución para el WebJob.

Utilice el archivo *settings.job* para programar la ejecución del trabajo WebJob. El siguiente ejemplo se ejecuta cada hora de 9: 00 a 17:00:

```json
{
    "schedule": "0 0 9-17 * * *"
}
```

Este archivo está ubicado en la raíz de la carpeta de WebJobs, junto con el script de WebJob; por ejemplo, `wwwroot\app_data\jobs\triggered\{job name}` o `wwwroot\app_data\jobs\continuous\{job name}`. Cuando implemente un WebJob desde Visual Studio, marque las propiedades del archivo *settings.job* en Visual Studio como **Copiar si es más nuevo**.

Si [crea un WebJob en Azure Portal](webjobs-create.md), el archivo *settings.job* se crea automáticamente.

#### <a name="cron-expressions"></a>Expresiones CRON

WebJobs usa las mismas expresiones CRON para realizar la programación que el desencadenador de temporizador de Azure Functions. Para obtener más información sobre la compatibilidad con CRON, consulte [Desencadenador de temporizador para Azure Functions](../azure-functions/functions-bindings-timer.md#ncrontab-expressions).

[!INCLUDE [webjobs-cron-timezone-note](../../includes/webjobs-cron-timezone-note.md)]

#### <a name="settingsjob-reference"></a>Referencia de settings.job

WebJobs admite las siguientes opciones de configuración:

| **Configuración** | **Tipo**  | **Descripción** |
| ----------- | --------- | --------------- |
| `is_in_place` | All | Permite que el WebJob se ejecute en su lugar sin copiarse primero en una carpeta temporal. Para más información, consulte [Directorio de trabajo de WebJob](https://github.com/projectkudu/kudu/wiki/WebJobs#webjob-working-directory). |
| `is_singleton` | Continuo | Ejecute solo el WebJob en una única instancia cuando escale horizontalmente. Para más información, consulte [Establecimiento de un trabajo continuo como singleton](https://github.com/projectkudu/kudu/wiki/WebJobs-API#set-a-continuous-job-as-singleton). |
| `schedule` | Desencadenado | Ejecute el trabajo web utilizando una programación basada en CRON. Para más información, consulte [Expresiones NCRONTAB](../azure-functions/functions-bindings-timer.md#ncrontab-expressions). |
| `stopping_wait_time`| All | Permite controlar el comportamiento de apagado. Para más información, consulte el [cierre correcto](https://github.com/projectkudu/kudu/wiki/WebJobs#graceful-shutdown). |

### <a name="continuous-execution"></a>Ejecución continua

Si habilita **Siempre activado** en Azure, puede usar Visual Studio para cambiar el WebJob para que se ejecute de forma continua:

1. Si todavía no lo ha hecho, [publique el proyecto en Azure](#deploy-to-azure-app-service).

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Publicar**.

1. En la pestaña **Publicar**, elija **Editar**. 

1. En el cuadro de diálogo **Configuración de perfil**, seleccione **Continuo** en **Tipo de WebJob**  y, luego, elija **Guardar**.

    ![Cuadro de diálogo Configuración de publicación de un WebJob](./media/webjobs-dotnet-deploy-vs/publish-settings.png)

1. Seleccione **Publicar** en la pestaña **Publicar** para volver a publicar el WebJob con la configuración actualizada.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre el SDK de WebJobs](webjobs-sdk-how-to.md)