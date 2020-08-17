---
title: Compilación de la primera aplicación sin servidor en Visual Studio
description: Creación, implementación y administración de una aplicación sin servidor en Visual Studio con Azure Logic Apps y Azure Functions
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 06/20/2019
ms.openlocfilehash: 86ee81f3802b7ec7f8189123192c20143a08cf3d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090253"
---
# <a name="build-your-first-serverless-app-by-using-azure-logic-apps-and-azure-functions-in-visual-studio"></a>Creación de su primera aplicación sin servidor en Visual Studio con Azure Logic Apps y Azure Functions

Con las herramientas sin servidor y funciones de Azure, como [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y [Azure Functions](../azure-functions/functions-overview.md), se pueden desarrollar e implementar rápidamente aplicaciones en la nube. En este artículo se muestra cómo empezar a crear una aplicación sin servidor, que usa una aplicación lógica que llama a una función de Azure, en Visual Studio. Para más información acerca de las soluciones sin servidor en Azure, consulte [Introducción a Azure sin servidor con Functions y Logic Apps](../logic-apps/logic-apps-serverless-overview.md).

## <a name="prerequisites"></a>Prerrequisitos

Para compilar una aplicación sin servidor en Visual Studio se necesita lo siguiente:

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Las herramientas siguientes. Descárguelas e instálelas si no las tiene aún.

  * [Visual Studio 2019, 2017 o 2015 (Community u otra edición)](https://aka.ms/download-visual-studio). 
  Este inicio rápido usa Visual Studio Community 2017, que es gratuito.

    > [!IMPORTANT]
    > Al instalar Visual Studio de 2019 o 2017, asegúrese de que selecciona la carga de trabajo **desarrollo de Azure**.

  * [Microsoft Azure SDK para .NET (versión 2.9.1 o posterior)](https://azure.microsoft.com/downloads/). 
  Más información sobre [Azure SDK for .NET](/dotnet/azure/dotnet-tools?view=azure-dotnet).

  * [Azure PowerShell](https://github.com/Azure/azure-powershell#installation).

  * Herramientas de Azure Logic Apps para la versión de Visual Studio que desee:

    * [Visual Studio 2019](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2019)

    * [Visual Studio 2017](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2017)

    * [Visual Studio 2015](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2015)
  
    Puede descargar e instalar Herramientas de Azure Logic Apps directamente desde Visual Studio Marketplace o aprender [a instalar esta extensión desde Visual Studio](/visualstudio/ide/finding-and-using-visual-studio-extensions). 
    Asegúrese de reiniciar Visual Studio después de finalizar la instalación.

  * [Azure Functions Core Tools](https://www.npmjs.com/package/azure-functions-core-tools) para depurar Functions de forma local.

* Acceda a la web mientras usa el Diseñador de aplicaciones lógicas integrado.

  El diseñador requiere una conexión a Internet para crear recursos en Azure y leer las propiedades y los datos de los conectores de la aplicación lógica. 
  Por ejemplo, si usa el conector de Dynamics CRM Online, el diseñador comprueba en la instancia de CRM si hay propiedades predeterminadas y propiedades personalizadas disponibles.

## <a name="create-a-resource-group-project"></a>Creación de un proyecto de grupo de recursos

Para empezar, cree un [proyecto de grupo de recursos de Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md) para su aplicación sin servidor. En Azure, los recursos se crean dentro de un *grupo de recursos*, que es una recopilación de lógicas que se usa para organizar, administrar e implementar recursos en toda una aplicación como un solo recurso. Para una aplicación sin servidor de Azure, el grupo de recursos incluye recursos para Azure Logic Apps y Azure Functions. Más información sobre [grupos de recursos y recursos de Azure](../azure-resource-manager/management/overview.md).

1. Inicie Visual Studio y conéctese con su cuenta de Azure.

1. En el menú **Archivo**, seleccione **Nuevo** > **Proyecto**.

   ![Creación de un proyecto en Visual Studio](./media/logic-apps-serverless-get-started-vs/create-new-project-visual-studio.png)

1. En **Instalado**, seleccione **Visual C#** o **Visual Basic**. Después, seleccione **Nube** > **Grupo de recursos de Azure**.

   > [!NOTE]
   > Si la categoría **Nube** o el proyecto **Grupo de recursos de Azure** no existen, asegúrese de que ha instalado el SDK de Azure para Visual Studio.

   Si usa Visual Studio 2019, siga estos pasos:

   1. En el cuadro **Crear un proyecto**, seleccione la plantilla de proyectos **Grupo de recursos de Azure** para Visual C# o Visual Basic y luego seleccione **Siguiente**.

   1. Proporcione el nombre y otra información adicional del proyecto que quiera usar para el grupo de recursos de Azure. Seleccione **Crear** cuando haya terminado.

1. Asigne un nombre y una ubicación al proyecto y luego seleccione **Aceptar**.

   Visual Studio le pedirá que seleccione una plantilla en la lista de plantillas. 
   En este ejemplo se usa una plantilla de inicio rápido de Azure para la compilación de una aplicación sin servidor que incluye una aplicación lógica y una llamada a una función de Azure.

   > [!TIP]
   > En escenarios donde no quiera implementar previamente la solución en un grupo de recursos de Azure, puede usar la plantilla **Aplicación lógica** en blanco, que simplemente crea una aplicación lógica vacía.

1. En la lista **Mostrar las plantillas de esta ubicación**, seleccione **Inicio rápido de Azure (github.com/Azure/azure-quickstart-templates)** .

1. En el cuadro de búsqueda, escriba "logic-app" como filtro. En los resultados, seleccione la plantilla **101-logic-app-and-function-app**.

   ![Selección de una plantilla de inicio rápido de Azure](./media/logic-apps-serverless-get-started-vs/select-template.png)

   Visual Studio crea y abre una solución para el proyecto del grupo de recursos. 
   La plantilla de inicio rápido de Azure que ha seleccionado crea una plantilla de implementación llamada azuredeploy.json en el proyecto del grupo de recursos. Dicha plantilla incluye la definición de una aplicación lógica sencilla que la desencadena una solicitud HTTP, llama a una función de Azure y devuelve el resultado en forma de respuesta HTTP.

   ![Nueva solución sin servidor](./media/logic-apps-serverless-get-started-vs/create-serverless-solution.png)

1. Después, implemente la solución en Azure. Debe realizar esto para poder abrir la plantilla de implementación y revisar los recursos de la aplicación sin servidor.

## <a name="deploy-your-solution"></a>Implementación de la solución

Para poder abrir la aplicación lógica en el Diseñador de aplicaciones lógicas en Visual Studio, debe tener un grupo de recursos de Azure que ya esté implementado en Azure. A partir de ahí, el diseñador puede crear conexiones a los recursos y servicios de la aplicación lógica. Para esta tarea, siga estos pasos con el fin de implemente la solución de Visual Studio en Azure Portal.

1. En el Explorador de soluciones, desde el menú contextual del proyecto, seleccione **Implementar** > **Nuevo**.

   ![Crear una implementación nueva para el grupo de recursos](./media/logic-apps-serverless-get-started-vs/deploy.png)

1. Si no están seleccionadas, seleccione su suscripción de Azure y el grupo de recursos en el que quiera realizar la implementación. Después, seleccione **Implementar**.

   ![Configuración de implementación](./media/logic-apps-serverless-get-started-vs/deploy-to-resource-group.png)

1. Si aparece el cuadro **Editar parámetros**, especifique los nombres de recurso que va a usar para la aplicación lógica y la aplicación de función de Azure en la implementación y guarde la configuración. Asegúrese de que usa un nombre globalmente único para la aplicación de función.

   ![Especificar nombres para la aplicación lógica y la aplicación de función](./media/logic-apps-serverless-get-started-vs/logic-function-app-name-parameters.png)

   Cuando Visual Studio inicia la implementación en el grupo de recursos especificado, el estado de implementación de la solución aparece en la ventana **Salida** de Visual Studio. 
   Una vez finalizada la implementación, la aplicación lógica está activa en Azure Portal.

## <a name="edit-your-logic-app-in-visual-studio"></a>Edición de una aplicación lógica en Visual Studio

Para editar la aplicación lógica después de la implementación, abra la aplicación lógica con el Diseñador de aplicaciones lógicas en Visual Studio.

1. En el Explorador de soluciones, desde el menú contextual del archivo azuredeploy.json, seleccione **Abrir con el Diseñador de aplicaciones lógicas**.

   ![Apertura de azuredeploy.json en el Diseñador de aplicaciones lógicas](./media/logic-apps-serverless-get-started-vs/open-logic-app-designer.png)

   > [!TIP]
   > Si no tiene este comando en Visual Studio 2019, compruebe que tiene las actualizaciones más recientes de Visual Studio.

1. Después de que el cuadro **Propiedades de la aplicación lógica** aparezca, en **Suscripción**, seleccione su suscripción de Azure si todavía no está seleccionada. En **Grupo de recursos**, seleccione el grupo de recursos y la ubicación en la que ha implementado la solución y luego seleccione **Aceptar**.

   ![Propiedades de la aplicación lógica](./media/logic-apps-serverless-get-started-vs/logic-app-properties.png)

   Cuando se abra el Diseñador de aplicaciones lógicas, puede agregar más pasos o cambiar el flujo de trabajo y guardar las actualizaciones.

   ![Aplicación lógica abierta en el Diseñador de aplicación lógica](./media/logic-apps-serverless-get-started-vs/opened-logic-app.png)

## <a name="create-your-azure-functions-project"></a>Creación del proyecto de Azure Functions

Para crear el proyecto y la función de Functions con JavaScript, Python, F#, PowerShell, Batch o Bash, siga los pasos descritos en [Trabajar con Azure Functions Core Tools](../azure-functions/functions-run-local.md). Para desarrollar una función de Azure con C# en la solución, use una biblioteca de clases de C#, para lo que debe seguir los pasos descritos en [Publicar una biblioteca de clases .NET como una Function App](https://azure.microsoft.com/blog/).

## <a name="deploy-functions-from-visual-studio"></a>Implementación de funciones desde Visual Studio

La plantilla implementa las funciones de Azure que tiene en su solución desde el repositorio de Git que especifiquen las variables en el archivo azuredeploy.json. Si crea un proyecto de Functions en su solución, puede marcarlo en el control de código fuente de Git (por ejemplo, GitHub o Azure DevOps) y luego actualizar la variable `repo` para que la plantilla implemente la función de Azure.

## <a name="manage-logic-apps-and-view-run-history"></a>Administración de aplicaciones lógicas y visualización del historial de ejecuciones

En el caso de las aplicaciones lógicas ya implementadas en Azure, todavía puede editarlas, administrarlas, ver su historial de ejecución y deshabilitarlas desde Visual Studio.

1. En el menú **Ver** de Visual Studio, abra **Cloud Explorer**.

1. En **Todas las suscripciones**, seleccione la suscripción de Azure asociada a las aplicaciones lógicas que quiera administrar y seleccione **Aplicar**.

1. En **Logic Apps**, seleccione la aplicación lógica. En el menú contextual de la aplicación, seleccione **Abrir con el editor de aplicaciones lógicas**.

   > [!TIP]
   > Si no tiene este comando en Visual Studio 2019, compruebe que tiene las actualizaciones más recientes de Visual Studio.

Ya puede descargar la aplicación lógica que ha publicado en el proyecto del grupo de recursos. Por tanto, aunque pueda haber iniciado una aplicación lógica en Azure Portal, puede importarla y administrarla en Visual Studio. Para más información, consulte [Administración de aplicaciones lógicas con Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md).

## <a name="next-steps"></a>Pasos siguientes

* [Administración de aplicaciones lógicas con Visual Studio](manage-logic-apps-with-visual-studio.md)
