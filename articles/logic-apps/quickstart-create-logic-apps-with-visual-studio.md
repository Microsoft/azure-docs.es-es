---
title: Automatización de tareas y flujos de trabajo con Visual Studio
description: Cree, programe y ejecute flujos de trabajo automáticos para la integración empresarial mediante Azure Logic Apps y Visual Studio
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: quickstart
ms.custom: mvc
ms.date: 07/22/2020
ms.openlocfilehash: cd46821b74803d62be0361346166ed78a5f53286
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132371"
---
# <a name="quickstart-create-automated-tasks-processes-and-workflows-with-azure-logic-apps---visual-studio"></a>Inicio rápido: Creación y automatización de tareas, procesos y flujos de trabajo con Azure Logic Apps en Visual Studio

Con [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y Visual Studio, puede crear flujos de trabajo que automaticen las tareas y los procesos de integración de aplicaciones, datos, sistemas y servicios a través de empresas y organizaciones. En este inicio rápido se muestra cómo puede diseñar y compilar estos flujos de trabajo mediante la creación de aplicaciones lógicas en Visual Studio y la implementación de esas aplicaciones en Azure. Aunque puede realizar estas tareas en Azure Portal, Visual Studio le permite agregar aplicaciones lógicas para el control de código fuente, publicar versiones diferentes y crear plantillas de Azure Resource Manager para distintos entornos de implementación.

Si no está familiarizado con Azure Logic Apps y solo desea conocer los conceptos básicos, pruebe en su lugar con el [inicio rápido para crear una aplicación lógica en Azure Portal](../logic-apps/quickstart-create-first-logic-app-workflow.md). El Diseñador de aplicación lógica, tanto en Azure Portal como en Visual Studio, funciona de forma similar.

En este inicio rápido, crea la misma aplicación lógica que en el inicio rápido de Azure Portal, pero con Visual Studio. Esta aplicación lógica supervisa la fuente RSS de un sitio web y envía un correo electrónico para cada nuevo elemento de dicha fuente. La aplicación lógica finalizada se parecerá a este flujo de trabajo general:

![Aplicación lógica terminada](./media/quickstart-create-logic-apps-with-visual-studio/high-level-workflow-overview.png)

<a name="prerequisites"></a>

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* Descargue e instale estas herramientas si no las tiene aún:

  * [Visual Studio 2019, 2017 o 2015 Community Edition o superior](https://aka.ms/download-visual-studio). Este inicio rápido usa Visual Studio Community 2017.

    > [!IMPORTANT]
    > Al instalar Visual Studio de 2019 o 2017, asegúrese de que selecciona la carga de trabajo **desarrollo de Azure**.

  * [Microsoft Azure SDK para .NET (2.9.1 o una versión posterior)](https://azure.microsoft.com/downloads/). Más información sobre [Azure SDK for .NET](/dotnet/azure/dotnet-tools?view=azure-dotnet).

  * [Azure PowerShell](https://github.com/Azure/azure-powershell#installation)

  * Las últimas herramientas de Azure Logic Apps para la extensión de Visual Studio para la versión que necesita:

    * [Visual Studio 2019](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2019)

    * [Visual Studio 2017](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2017)

    * [Visual Studio 2015](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2015)
  
    Puede descargar e instalar Herramientas de Azure Logic Apps directamente desde Visual Studio Marketplace o aprender [a instalar esta extensión desde Visual Studio](/visualstudio/ide/finding-and-using-visual-studio-extensions). Asegúrese de reiniciar Visual Studio después de finalizar la instalación.

  * Para usar suscripciones de Azure Government con Visual Studio, consulte estos temas para conocer más datos de la configuración:

    * Visual Studio 2019: [Inicio rápido: Conexión a Azure Government con Visual Studio](../azure-government/documentation-government-connect-vs.md)

    * Visual Studio 2017: [Introducción a la extensión de Visual Studio del selector de entorno de Azure](https://devblogs.microsoft.com/azuregov/introducing-the-azure-environment-selector-visual-studio-extension/), que se puede descargar e instalar desde [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=SteveMichelotti.AzureEnvironmentSelector).

* Acceso a la web al usar el Diseñador de aplicaciones lógicas integrado

  El diseñador requiere una conexión a Internet para crear recursos en Azure y leer las propiedades y los datos de los conectores de la aplicación lógica.

* Una cuenta de correo electrónico que sea compatible con Logic Apps, como Office 365 Outlook, Outlook.com o Gmail. En el caso de otros proveedores, consulte la [lista de conectores que se muestra aquí](/connectors/). En este ejemplo se utiliza Office 365 Outlook. Si usa otro proveedor distinto, los pasos generales serán los mismos pero su interfaz de usuario puede ser ligeramente distinta.

  > [!IMPORTANT]
  > Si quiere usar el conector de Gmail, solo las cuentas empresariales de G-Suite pueden usar este conector sin restricciones en las aplicaciones lógicas. Si tiene una cuenta de consumidor de Gmail, puede usar este conector solo con servicios específicos aprobados por Google o puede [crear una aplicación cliente de Google para usarla en la autenticación con el conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Para más información, consulte [Directivas de privacidad y seguridad de datos de los conectores de Google en Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

<a name="create-resource-group-project"></a>

## <a name="create-azure-resource-group-project"></a>Creación de un proyecto de grupo de recursos de Azure

Para empezar, cree un [proyecto de grupo de recursos de Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md). Más información sobre [grupos de recursos y recursos de Azure](../azure-resource-manager/management/overview.md).

1. Inicie Visual Studio. Inicie sesión con su cuenta de Azure.

1. En el menú **Archivo**, seleccione **Nuevo** > **Proyecto**. (Teclado: CTRL + Mayús + N)

   ![En el menú "Archivo", seleccione "Nuevo" > "Proyecto"](./media/quickstart-create-logic-apps-with-visual-studio/create-new-visual-studio-project.png)

1. En **Instalado**, seleccione **Visual C#** o **Visual Basic**. Seleccione **Nube** > **Grupo de recursos de Azure**. Asigne un nombre al proyecto, por ejemplo:

   ![Creación de un proyecto de grupo de recursos de Azure](./media/quickstart-create-logic-apps-with-visual-studio/create-azure-cloud-service-project.png)

   > [!NOTE]
   > Los nombres de los grupos de recursos pueden contener solo letras,números, puntos (`.`),caracteres de subrayado (`_`), guiones (`-`) y paréntesis (`(`, `)`), pero no pueden *acabar* con puntos (`.`).
   >
   > Si **Nube** o **Grupo de recursos de Azure** no aparecen, asegúrese de que ha instalado el SDK de Azure para Visual Studio.

   Si usa Visual Studio 2019, siga estos pasos:

   1. En el cuadro **Crear un proyecto**, seleccione el proyecto **Grupo de recursos de Azure** para Visual C# o Visual Basic. Seleccione **Next** (Siguiente).

   1. Proporcione el nombre del grupo de recursos de Azure que desea usar y otra información adicional sobre el proyecto. Seleccione **Crear**.

1. En la lista de plantillas, seleccione la plantilla **Logic App**. Seleccione **Aceptar**.

   ![Selección de la plantilla de aplicación lógica](./media/quickstart-create-logic-apps-with-visual-studio/select-logic-app-template.png)

   Después de que Visual Studio cree el proyecto, el Explorador de soluciones se abre y muestra la solución. En la solución, el archivo**LogicApp.json** no solo almacena la definición de la aplicación lógica, sino que también es una plantilla de Azure Resource Manager que puede configurar para la implementación.

   ![El Explorador de soluciones muestra la nueva solución de aplicación lógica y el archivo de implementación](./media/quickstart-create-logic-apps-with-visual-studio/logic-app-solution-created.png)

## <a name="create-blank-logic-app"></a>Creación de una aplicación lógica en blanco

Cuando tenga el proyecto del grupo de recursos de Azure, cree la aplicación lógica a partir de la plantilla **Blank Logic App**.

1. En el Explorador de soluciones, abra el menú contextual del archivo **LogicApp.json**. Seleccione **Open with Logic App Designer** (Abrir con Diseñador de aplicación lógica). (Teclado: CTRL + L)

   ![Abrir archivo logicapp.json en el Diseñador de aplicación lógica](./media/quickstart-create-logic-apps-with-visual-studio/open-logic-app-designer.png)

   > [!TIP]
   > Si no tiene este comando en Visual Studio 2019, compruebe que tiene las actualizaciones más recientes de Visual Studio.

   Visual Studio necesita su suscripción de Azure y un grupo de recursos para crear e implementar recursos de la aplicación lógica y las conexiones.

1. En **Suscripción**, seleccione su suscripción de Azure. En **Grupo de recursos**, seleccione **Crear nuevo** para generar otro grupo de recursos de Azure.

   ![Selección de una suscripción de Azure, un grupo de recursos y una ubicación de recursos](./media/quickstart-create-logic-apps-with-visual-studio/select-azure-subscription-resource-group-location.png)

   | Configuración | Valor de ejemplo | Descripción |
   | ------- | ------------- | ----------- |
   | Cuenta de usuario | Fabrikam <br> sophia-owen@fabrikam.com | Cuenta que usó al iniciar sesión en Visual Studio |
   | **Suscripción** | Pay-As-You-Go <br> (sophia-owen@fabrikam.com) | Nombre de la suscripción de Azure y cuenta asociada |
   | **Grupo de recursos** | MyLogicApp-RG <br> (Oeste de EE. UU.) | Grupo de recursos de Azure y ubicación para almacenar e implementar recursos de la aplicación lógica |
   | **Ubicación** | **Same as Resource Group** (Igual que el grupo de recursos) | El tipo de ubicación y la ubicación específica para implementar la aplicación lógica. El tipo de ubicación puede ser una región de Azure o un [entorno del servicio de integración (ISE)](connect-virtual-network-vnet-isolated-environment.md) existente. <p>En esta guía de inicio rápido, mantenga el tipo de ubicación establecido en **Región** y la ubicación establecida en **Same as Resource Group** (Igual que el grupo de recursos). <p>**Nota**: Después de crear el proyecto del grupo de recursos, puede [cambiar el tipo de ubicación y la ubicación](manage-logic-apps-with-visual-studio.md#change-location), pero un tipo de ubicación diferente afecta a la aplicación lógica de varias maneras. |
   ||||

1. El Diseñador de aplicación lógica se abre y muestra un vídeo de introducción y desencadenadores utilizados frecuentemente. Desplácese hacia abajo más allá del vídeo y de los desencadenadores hasta **Plantillas**y seleccione **Blank Logic App**.

   ![Selección de "Blank Logic App"](./media/quickstart-create-logic-apps-with-visual-studio/choose-blank-logic-app-template.png)

## <a name="build-logic-app-workflow"></a>Creación de un flujo de trabajo de aplicación lógica

A continuación, añada un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts) RSS que se active cuando aparezca un nuevo elemento en la fuente. Cada aplicación lógica se inicia con un desencadenador, que se desencadena cuando se cumplen criterios específicos. Cada vez que el desencadenador se activa, el motor de Logic Apps crea una instancia de aplicación lógica que ejecuta el flujo de trabajo.

1. En el cuadro de búsqueda del Diseñador de aplicación lógica, seleccione **Todo**. En el cuadro de búsqueda, escriba "rss". En la lista de desencadenadores, seleccione este desencadenador: **Cuando se publica un elemento de fuente**

   ![Cree la aplicación lógica mediante la incorporación de un desencadenador y acciones](./media/quickstart-create-logic-apps-with-visual-studio/add-trigger-logic-app.png)

1. Cuando el desencadenador se muestra en el diseñador, termine de crear la aplicación lógica, siga los pasos del flujo de trabajo en el [Inicio rápido de Azure Portal](../logic-apps/quickstart-create-first-logic-app-workflow.md#add-rss-trigger) y, después, vuelva a este artículo. Cuando haya terminado, la aplicación lógica se parecerá a este ejemplo:

   ![Aplicación lógica terminada](./media/quickstart-create-logic-apps-with-visual-studio/finished-logic-app-workflow.png)

1. Guarde la solución de Visual Studio. (Teclado: Ctrl + S)

<a name="deploy-to-Azure"></a>

## <a name="deploy-logic-app-to-azure"></a>Implementación de la aplicación lógica en Azure

Antes de poder ejecutar y probar la aplicación lógica, implemente la aplicación desde Visual Studio en Azure.

1. En el Explorador de soluciones, en el menú contextual del proyecto, seleccione **Implementar** > **Nuevo**. Si se le pide, inicie sesión en su cuenta de Azure.

   ![Creación de implementación de la aplicación lógica](./media/quickstart-create-logic-apps-with-visual-studio/create-logic-app-deployment.png)

1. Para esta implementación, mantenga la suscripción de Azure, el grupo de recursos y otras configuraciones predeterminadas. Seleccione **Implementar**.

   ![Implementación de la aplicación lógica en el grupo de recursos de Azure](./media/quickstart-create-logic-apps-with-visual-studio/select-azure-subscription-resource-group-deployment.png)

1. Si el cuadro **Editar parámetros** aparece, proporcione un nombre al recurso de la aplicación lógica. Guarde la configuración.

   ![Nombre de la implementación para la aplicación lógica](./media/quickstart-create-logic-apps-with-visual-studio/edit-parameters-deployment.png)

   Cuando se inicia la implementación, aparece el estado de la implementación de la aplicación en la ventana **Salida** de Visual Studio. Si no aparece el estado, abra la lista **Mostrar salida de** y seleccione el grupo de recursos de Azure.

   ![Salida del estado de la implementación](./media/quickstart-create-logic-apps-with-visual-studio/logic-app-output-window.png)

   Si los conectores seleccionados necesitan que el usuario escriba datos, se abre una ventana de PowerShell en segundo plano y le solicita las contraseñas o claves secretas necesarias. Después de escribir esta información, la implementación continúa.

   ![Ventana de PowerShell](./media/quickstart-create-logic-apps-with-visual-studio/logic-apps-powershell-window.png)

   Una vez finalizada la implementación, la aplicación lógica está activa en Azure Portal y se ejecuta en función de la programación especificada (cada minuto). Si el desencadenador encuentra nuevos elementos de fuente, el desencadenador se activa, que crea una instancia de flujo de trabajo que ejecuta las acciones de la aplicación lógica. La aplicación lógica envía un correo electrónico por cada nuevo elemento. Si el desencadenador no encuentra nuevos elementos, no se activa y "omite" crear instancias del flujo de trabajo. La aplicación lógica espera hasta el siguiente intervalo antes de volver a realizar la comprobación.

   Estos son correos electrónicos de ejemplo que envía esta aplicación lógica. Si no obtiene los mensajes de correo electrónico, compruebe la carpeta de correo electrónico no deseado.

   ![Outlook envía un mensaje de correo electrónico para cada nuevo elemento RSS](./media/quickstart-create-logic-apps-with-visual-studio/outlook-email.png)

Ya ha creado e implementado correctamente la aplicación lógica con Visual Studio. Para administrar la aplicación lógica y revisar su historial de ejecución, consulte [Administración de aplicaciones lógicas con Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md).

## <a name="add-new-logic-app"></a>Incorporación de una nueva aplicación lógica

Si tiene un proyecto de grupo de recursos de Azure existente, puede agregar una nueva aplicación lógica mediante la ventana Esquema JSON.

1. En el Explorador de soluciones, abra el archivo `<logic-app-name>.json`.

1. En el menú **Vista**, seleccione **Otras ventanas** > **Esquema JSON**.

1. Para agregar un recurso al archivo de plantilla, seleccione **Agregar recurso** en la parte superior de la ventana Esquema JSON. O, en la ventana Esquema de JSON, abra el menú contextual **recursos** y seleccione **Agregar nuevo recurso**.

   ![Ventana Esquema JSON](./media/quickstart-create-logic-apps-with-visual-studio/json-outline-window-add-resource.png)

1. En el cuadro de diálogo **Agregar recurso**, en el cuadro de búsqueda, busque `logic app` y seleccione **Aplicación lógica**. Asigne un nombre a la aplicación lógica y seleccione **Agregar**.

   ![Agregar recurso](./media/quickstart-create-logic-apps-with-visual-studio/add-logic-app-resource.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite la aplicación lógica, elimine el grupo de recursos que contiene la aplicación lógica y los recursos relacionados.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con la misma cuenta utilizada para crear la aplicación lógica.

1. En el menú de Azure Portal, seleccione **Grupos de recursos** o busque y seleccione **Grupos de recursos** en cualquier página. Seleccione el grupo de recursos de la aplicación lógica.

1. En la página **Información general**, elija **Eliminar grupo de recursos**. Escriba el nombre del grupo de recursos como confirmación y seleccione **Eliminar**.

   !["Grupos de recursos" > "Introducción" > "Eliminar grupo de recursos"](./media/quickstart-create-logic-apps-with-visual-studio/clean-up-resources.png)

1. Elimine la solución de Visual Studio del equipo local.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, se compila, implementa y ejecuta la aplicación lógica con Visual Studio. Para obtener más información sobre cómo administrar y realizar la implementación avanzada de aplicaciones lógicas con Visual Studio, consulte estos artículos:

> [!div class="nextstepaction"]
> [Administración de aplicaciones lógicas con Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md)
