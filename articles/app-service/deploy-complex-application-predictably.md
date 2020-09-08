---
title: Implementación de aplicaciones de forma predecible con ARM
description: Aprenda a implementar varias aplicaciones de Azure App Service como una sola unidad y de forma predecible mediante plantillas de Azure Resource Management y scripts de PowerShell.
ms.assetid: bb51e565-e462-4c60-929a-2ff90121f41d
ms.topic: article
ms.date: 01/06/2016
ms.custom: seodec18
ms.openlocfilehash: 6c45d2da8658740b5e5e7e3dceb7478ea28d712c
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962033"
---
# <a name="provision-and-deploy-microservices-predictably-in-azure"></a>Aprovisionamiento e implementación predecibles de microservicios en Azure
En este tutorial se explica cómo aprovisionar e implementar una aplicación formada por [microservicios](https://en.wikipedia.org/wiki/Microservices) en [Azure App Service](https://azure.microsoft.com/services/app-service/) como una sola unidad y de forma predecible con plantillas de grupo de recursos JSON y scripting de PowerShell. 

Al aprovisionar e implementar aplicaciones de gran escala que se componen de microservicios muy desacoplados, la repetibilidad y la previsión son fundamentales para que el proceso se realice correctamente. [Azure App Service](https://azure.microsoft.com/services/app-service/) le permite crear microservicios, como aplicaciones web, back-ends móviles y aplicaciones de API. [Azure Resource Manager](../azure-resource-manager/management/overview.md) le permite administrar todos los microservicios como una unidad, junto con las dependencias de recursos, como la configuración del control de origen y la base de datos. Ahora, también puede implementar esta aplicación mediante plantillas JSON y scripting sencillo de PowerShell. 

## <a name="what-you-will-do"></a>Lo que hará
En el tutorial, implementará una aplicación que incluye:

* Dos aplicaciones de App Service (es decir, dos microservicios)
* Una instancia de SQL Database backend
* Configuración de aplicaciones, cadenas de conexión y control de código fuente
* Información sobre aplicaciones, alertas y configuración de autoescala

## <a name="tools-you-will-use"></a>Herramientas que va a utilizar
En este tutorial, utilizará las siguientes herramientas. Habida cuenta de que no se trata de un tema dedicado íntegramente a las herramientas, voy a presentar un escenario de un extremo a otro y a dar solo una breve introducción acerca de cada una, además de indicar dónde se puede encontrar más información al respecto. 

### <a name="azure-resource-manager-templates-json"></a>Plantillas del Administrador de recursos de Azure (JSON)
Cada vez que crea una aplicación en Azure App Service (por ejemplo, Azure Resource Manager usa una plantilla JSON para crear el grupo de recursos completo con los recursos del componente). Una plantilla compleja de [Azure Marketplace](../marketplace/index.yml) puede incluir la base de datos, cuentas de almacenamiento, el plan de App Service, la propia aplicación, las reglas de alertas, la configuración de la aplicación, configuración de escalado automático, etc., y todas estas plantillas están disponibles en PowerShell. Para obtener más información sobre las plantillas del Administrador de recursos de Azure, consulte [Creación de plantillas de Administrador de recursos de Azure](../azure-resource-manager/templates/template-syntax.md)

### <a name="azure-sdk-26-for-visual-studio"></a>SDK de Azure 2.6 para Visual Studio
El SDK más reciente contiene mejoras en la compatibilidad de la plantilla del Administrador de recursos en el editor de JSON. Puede utilizar esto para crear rápidamente una plantilla de grupo de recursos desde el principio o abrir una plantilla existente de JSON (por ejemplo, una plantilla descargada de la Galería) para modificarla, rellenar el archivo de parámetros e incluso implementar el grupo de recursos directamente desde una solución de grupo de recursos de Azure.

Para obtener más información, consulte [SDK de Azure 2.6 para Visual Studio](https://azure.microsoft.com/blog/2015/04/29/announcing-the-azure-sdk-2-6-for-net/).

### <a name="azure-powershell-080-or-later"></a>Azure PowerShell 0.8.0 o posterior
A partir de la versión 0.8.0, la instalación de Azure PowerShell incluye el módulo Administrador de recursos de Azure además del módulo de Azure. Este nuevo módulo permite crear un script de la implementación de grupos de recursos.

Para obtener más información, consulte [Uso de Azure PowerShell con el Administrador de recursos de Azure](../azure-resource-manager/management/manage-resources-powershell.md)

### <a name="azure-resource-explorer"></a>Azure Resource Explorer
Esta [herramienta de vista previa](https://resources.azure.com) le permite explorar las definiciones de JSON de todos los grupos de recursos en su suscripción y los recursos individuales. En la herramienta, puede editar las definiciones de JSON de un recurso, eliminar una jerarquía completa de recursos y crear nuevos recursos.  La información disponible en esta herramienta es muy útil para la creación de plantillas, ya que muestra las propiedades que debe establecer para un tipo determinado de recursos, los valores correctos, etc. Incluso puede crear el grupo de recursos en [Azure Portal](https://portal.azure.com/); a continuación, examinar sus definiciones de JSON en la herramienta de explorador a fin de facilitarle el uso de plantillas para el grupo de recursos.

### <a name="deploy-to-azure-button"></a>Botón Implementación en Azure
Si utiliza GitHub para el control de código fuente, puede colocar un [botón de implementación en Azure](https://azure.microsoft.com/blog/2014/11/13/deploy-to-azure-button-for-azure-websites-2/) en el archivo README.MD, que habilita una IU de implementación inmediata para Azure. Aunque puede hacerlo para cualquier aplicación sencilla, se puede extender para habilitar la implementación de un grupo de recursos completo colocando un archivo azuredeploy.json en la raíz del repositorio. Este archivo JSON, que contiene la plantilla de grupo de recursos, lo utilizará el botón Implementación en Azure para crear el grupo de recursos. Para obtener un ejemplo, consulte el ejemplo [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) , que se utilizará en este tutorial.

## <a name="get-the-sample-resource-group-template"></a>Obtención de la plantilla de grupo de recursos de ejemplo
Ahora vamos a centrarnos en ello.

1. Navegue hasta el ejemplo de App Service [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) .
2. En readme.md, haga clic en **Implementación en Azure**.
3. Se le remite al sitio [deploy-to-azure](https://deploy.azure.com) de sitio y se le solicita que especifique los parámetros de implementación. Observe que la mayoría de los campos se rellenan con el nombre del repositorio y algunas cadenas aleatorias. Puede cambiar todos los campos si lo desea, pero lo único que tiene que escribir es el inicio de sesión administrativo de SQL Server y la contraseña y, a continuación, hacer clic en **Siguiente**.
   
   ![Muestra los parámetros de implementación de entrada en el sitio de implementación en Azure.](./media/app-service-deploy-complex-application-predictably/gettemplate-1-deploybuttonui.png)
4. A continuación, haga clic en **Implementar** para iniciar el proceso de implementación. Una vez que el proceso se ejecuta hasta completarse, haga clic en el vínculo http://todoapp*XXXX*.azurewebsites.net para examinar la aplicación implementada. 
   
   ![Muestra el proceso de implementación de la aplicación.](./media/app-service-deploy-complex-application-predictably/gettemplate-2-deployprogress.png)
   
   La interfaz de usuario podría ser un poco lenta la primera vez que obtiene acceso a ella porque las aplicaciones se acaban de iniciar, pero no le quepa duda de que se trata de una aplicación totalmente funcional.
5. En la página Implementar, haga clic en el vínculo **Administrar** para ver la nueva aplicación en Azure Portal.
6. En el menú desplegable **Essentials** , haga clic en el vínculo del grupo de recursos. Tenga en cuenta también que la aplicación ya está conectada al repositorio de GitHub en **Proyecto externo**. 
   
   ![Muestra el vínculo Grupo de recursos en el menú desplegable Essentials.](./media/app-service-deploy-complex-application-predictably/gettemplate-3-portalresourcegroup.png)
7. En la hoja del grupo de recursos, tenga en cuenta que ya existen dos aplicaciones y una instancia de SQL Database en el grupo de recursos.
   
   ![Muestra los recursos disponibles en el grupo de recursos.](./media/app-service-deploy-complex-application-predictably/gettemplate-4-portalresourcegroupclicked.png)

Todo lo que acabamos de ver en unos pocos minutos es una aplicación de dos microservicios totalmente implementada, con todos los componentes, las dependencias, la configuración, la base de datos y la publicación continua, configurada mediante una orquestación automatizada en el Administrador de recursos de Azure. Todo esto se realiza mediante dos elementos:

* El botón Implementación en Azure
* azuredeploy.json en la raíz del repositorio

Puede implementar la misma aplicación decenas, cientos o miles de veces y tener exactamente la misma configuración cada vez. La repetibilidad y la predicción de este enfoque le permite implementar aplicaciones de gran escala con facilidad y confianza.

## <a name="examine-or-edit-azuredeployjson"></a>Examine (o edite) AZUREDEPLOY.JSON
Ahora veamos cómo se ha configurado el repositorio de GitHub. Va a utilizar el editor de JSON en el SDK .NET de Azure, por lo que si no ha instalado todavía [SDK .NET de Azure 2.6](https://azure.microsoft.com/downloads/), hágalo ahora.

1. Clone el repositorio [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) con su herramienta git favorita. En la siguiente captura de pantalla, hago esto en Team Explorer en Visual Studio 2013.
   
   ![Muestra cómo usar una herramienta de Git para clonar el repositorio de ToDoApp.](./media/app-service-deploy-complex-application-predictably/examinejson-1-vsclone.png)
2. Desde la raíz del repositorio, abra azuredeploy.json en Visual Studio. Si no ve el panel de esquema de JSON, deberá instalar el SDK .NET de Azure.
   
   ![Muestra el panel de esquemas JSON en Visual Studio.](./media/app-service-deploy-complex-application-predictably/examinejson-2-vsjsoneditor.png)

No voy a describir todos los detalles del formato JSON, pero en la sección [Más recursos](#resources) hay vínculos para obtener información sobre el lenguaje de la plantilla de grupo de recursos. Aquí, voy a mostrar las interesantes características que le ayudarán a empezar a crear su propia plantilla personalizada para la implementación de la aplicación.

### <a name="parameters"></a>Parámetros
Eche un vistazo a la sección de parámetros para ver que la mayoría de ellos son los que el botón **Implementación en Azure** le pide que especifique. El sitio detrás del botón **Implementación en Azure** rellena la interfaz de usuario de entrada con los parámetros definidos en azuredeploy.json. Estos parámetros se usan en las definiciones de recursos, como los nombres de recursos, los valores de propiedad, etc.

### <a name="resources"></a>Recursos
En el nodo de recursos, puede ver que se definen cuatro recursos de nivel superior: una instancia de SQL Server, un plan de App Service y dos aplicaciones. 

#### <a name="app-service-plan"></a>Plan de App Service
Comencemos con un simple recurso de nivel de raíz en JSON. En el esquema de JSON, haga clic en el plan de App Service denominado **[hostingPlanName]** para resaltar el código correspondiente de JSON. 

![Muestra la sección [hostingPlanName] del código JSON.](./media/app-service-deploy-complex-application-predictably/examinejson-3-appserviceplan.png)

Tenga en cuenta que el elemento `type` especifica la cadena para un plan de App Service (hace mucho tiempo se conocía como una granja de servidores), y otros elementos y propiedades se rellenan con los parámetros definidos en el archivo JSON, y este recurso no tiene ningún recurso anidado.

> [!NOTE]
> Tenga en cuenta también que el valor de `apiVersion` indica a Azure con qué versión de la API de REST utilizar la definición de recursos de JSON, y esto puede afectar a qué formato se le debe dar al recurso dentro de `{}`. 
> 
> 

#### <a name="sql-server"></a>SQL Server
A continuación, haga clic en el recurso de SQL Server denominado **SQLServer** en el esquema de JSON.

![Muestra el recurso de SQL Server denominado SQLServer en el esquema de JSON.](./media/app-service-deploy-complex-application-predictably/examinejson-4-sqlserver.png)

Tenga en cuenta lo siguiente sobre el código resaltado de JSON:

* El uso de parámetros garantiza que los recursos creados son denominados y configurados de manera que sean coherentes entre sí.
* El recurso de SQL Server tiene dos recursos anidados, cada uno con un valor diferente para `type`.
* Los recursos anidados dentro de `“resources”: […]`, donde se definen las reglas de firewall y de la base de datos, tienen un elemento `dependsOn` que especifica el identificador de recurso del recurso SQLServer en el nivel de raíz. Esto indica al Administrador de recursos de Azure que antes de crear este recurso, el otro recurso ya debe existir; y si ese otro recurso está definido en la plantilla, cree ese primero.
  
  > [!NOTE]
  > Para obtener información detallada sobre cómo utilizar la función `resourceId()`, consulte [Funciones de la plantilla de Azure Resource Manager](../azure-resource-manager/templates/template-functions-resource.md#resourceid).
  > 
  > 
* El efecto del elemento `dependsOn` es que el Administrador de recursos de Azure puede saber qué recursos se pueden crear en paralelo y qué recursos deben crearse de forma secuencial. 

#### <a name="app-service-app"></a>Aplicación de App Service
Ahora, continuaremos con las aplicaciones en sí, que son más complicadas. Haga clic en la aplicación [variables(‘apiSiteName’)] en el esquema de JSON para resaltar su código JSON. Observará que las cosas se están poniendo mucho más interesantes. Para este propósito, hablaré de cada una de las características:

##### <a name="root-resource"></a>Recurso raíz
La aplicación depende de dos recursos distintos. Esto significa que Azure Resource Manager creará la aplicación solo después de haber creado el plan de App Service y la instancia de SQL Server.

![Muestra las dependencias de la aplicación en el plan de App Service y la instancia de SQL Server.](./media/app-service-deploy-complex-application-predictably/examinejson-5-webapproot.png)

##### <a name="app-settings"></a>Configuración de la aplicación
La configuración de la aplicación también se define como un recurso anidado.

![Muestra la configuración de la aplicación definida como un recurso anidado en el código JSON.](./media/app-service-deploy-complex-application-predictably/examinejson-6-webappsettings.png)

En el elemento `properties` para `config/appsettings`, tiene dos configuraciones para la aplicación con el formato `"<name>" : "<value>"`.

* `PROJECT` es una [configuración de KUDU](https://github.com/projectkudu/kudu/wiki/Customizing-deployments) que indica a la implementación de Azure qué proyecto usar en una solución de Visual Studio de varios proyectos. Le mostraré más adelante cómo se configura el control de código fuente, pero dado que el código de ToDoApp se encuentra en una solución de Visual Studio de varios proyectos, necesitamos esta configuración.
* `clientUrl` es simplemente una configuración de la aplicación que el código de la aplicación utiliza.

##### <a name="connection-strings"></a>Cadenas de conexión
Las cadenas de conexión también se definen como un recurso anidado.

![Muestra cómo las cadenas de conexión se definen como un recurso anidado en el código JSON.](./media/app-service-deploy-complex-application-predictably/examinejson-7-webappconnstr.png)

En el elemento `properties` para `config/connectionstrings`, cada cadena de conexión también se define como un par nombre-valor, con el formato específico de `"<name>" : {"value": "…", "type": "…"}`. Para el elemento `type`, los valores posibles son `MySql`, `SQLServer`, `SQLAzure` y `Custom`.

> [!TIP]
> Para obtener una lista definitiva de los tipos de cadena de conexión, ejecute el siguiente comando en Azure PowerShell: \[Enum]::GetNames("Microsoft.WindowsAzure.Commands.Utilities.Websites.Services.WebEntities.DatabaseType")
> 
> 

##### <a name="source-control"></a>Control de código fuente
La configuración del control de código fuente también se define como un recurso anidado. El Administrador de recursos de Azure usa este recurso para configurar la publicación continua (consulte la advertencia en `IsManualIntegration` más adelante) y también para iniciar la implementación de código de la aplicación automáticamente durante el procesamiento del archivo JSON.

![Muestra cómo se define el control de código fuente como un recurso anidado en el código JSON.](./media/app-service-deploy-complex-application-predictably/examinejson-8-webappsourcecontrol.png)

`RepoUrl` y `branch` deberían ser bastante intuitivos y deben apuntar al repositorio de Git y al nombre de la bifurcación desde la que realizar la publicación. De nuevo, se definen mediante parámetros de entrada. 

En el elemento `dependsOn`, tenga en cuenta que, además del recurso de aplicación, `sourcecontrols/web` depende también de `config/appsettings` y `config/connectionstrings`. Esto es porque una vez que `sourcecontrols/web` está configurado, el proceso de implementación de Azure intentará automáticamente implementar, generar e iniciar el código de aplicación. Por lo tanto, insertar esta dependencia ayuda a asegurarse de que la aplicación tenga acceso a la configuración de la aplicación y las cadenas de conexión necesarias antes de ejecutar el código de aplicación. 

> [!NOTE]
> Tenga en cuenta también que `IsManualIntegration` está establecido en `true`. Esta propiedad es necesaria en este tutorial, ya que no posee realmente el repositorio de GitHub y realmente no puede conceder permiso a Azure para configurar la publicación continua de [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) (es decir, la inserción automática de las actualizaciones del repositorio en Azure). Puede utilizar el valor predeterminado `false` para el repositorio especificado solo si ha configurado antes las credenciales de GitHub del propietario en el [Portal de Azure](https://portal.azure.com/) . En otras palabras, si ha configurado el control de código fuente para GitHub o BitBucket para cualquier aplicación en [Azure Portal](https://portal.azure.com/) anteriormente, mediante las credenciales de usuario, Azure recordará estas credenciales y las usará para implementar cualquier aplicación de GitHub o BitBucket en el futuro. Pero, si aún no ha completado esta tarea, se producirá un error en la implementación de la plantilla JSON cuando Azure Resource Manager intente configurar el control de código fuente de la aplicación porque no puede iniciar sesión en GitHub o Bitbucket con las credenciales del propietario del repositorio.
> 
> 

## <a name="compare-the-json-template-with-deployed-resource-group"></a>Comparación de la plantilla JSON con grupos de recursos implementados
Aquí puede consultar todas las hojas de la aplicación en [Azure Portal](https://portal.azure.com/), pero hay otra herramienta que también puede resultarle útil (incluso más útil). Vaya a la herramienta de vista previa [Explorador de recursos de Azure](https://resources.azure.com) , que proporciona una representación JSON de todos los grupos de recursos en las suscripciones, tal y como existen en el backend de Azure. También puede ver cómo la jerarquía JSON del grupo de recursos en Azure se corresponde con la jerarquía del archivo de plantilla utilizado para crearlo.

Por ejemplo, si voy a la herramienta [Explorador de recursos de Azure](https://resources.azure.com) y expando los nodos en el explorador, puedo ver el grupo de recursos y los recursos de nivel de raíz que se engloban en sus respectivos tipos de recursos.

![Vea el grupo de recursos y los recursos de nivel de raíz en la herramienta expandida Azure Resources Explorer.](./media/app-service-deploy-complex-application-predictably/ARM-1-treeview.png)

Si explora en profundidad una aplicación, verá que los detalles de configuración de esta son similares a los de la captura de pantalla siguiente:

![Explore en profundidad para ver los detalles de configuración de la aplicación.](./media/app-service-deploy-complex-application-predictably/ARM-2-jsonview.png)

Nuevamente, los recursos anidados deben tener una jerarquía muy similar a la de su archivo de plantilla JSON, y debería ver la configuración de la aplicación, las cadenas de conexión, etc. que se reflejan de forma adecuada en el panel JSON. La ausencia de configuración aquí puede indicar un problema con el archivo JSON y puede ayudarle a solucionar problemas con el archivo de plantilla JSON.

## <a name="deploy-the-resource-group-template-yourself"></a>Implementación de la plantilla de grupo de recursos manualmente
El botón **Implementación en Azure** es excelente, pero permite implementar la plantilla de grupo de recursos en azuredeploy.json sólo si ya se ha insertado azuredeploy.json en GitHub. El SDK .NET de Azure también proporciona las herramientas para implementar cualquier archivo de plantilla JSON directamente desde el equipo local. Para ello, siga estos pasos:

1. En Visual Studio, haga clic en **Archivo** > **Nuevo** > **proyecto**.
2. Haga clic en **Visual C#**  > **Nube** >  **&gt; Grupo de recursos de Azure** y, a continuación, haga clic en **Aceptar**.
   
   ![Cree un proyecto como un grupo de recursos de Azure en el SDK de .NET para Azure.](./media/app-service-deploy-complex-application-predictably/deploy-1-vsproject.png)
3. En **Seleccionar plantilla de Azure**, seleccione **Plantilla en blanco** y haga clic en **Aceptar**.
4. Arrastre azuredeploy.json a la carpeta **Plantilla** del nuevo proyecto.
   
   ![Muestra el resultado de arrastrar el archivo azuredeploy.json a la carpeta de plantillas del proyecto.](./media/app-service-deploy-complex-application-predictably/deploy-2-copyjson.png)
5. En el Explorador de soluciones, abra el archivo azuredeploy.json copiado.
6. Para ilustrar la demostración, vamos a agregar algunos recursos de Application Insights estándar a nuestro archivo JSON; para ello, hay que hacer clic en **Agregar recurso**. Si solo le interesa implementar el archivo JSON, omita los pasos de implementación.
   
   ![Muestra el botón Agregar recurso que puede usar para agregar recursos de Application Insights estándar al archivo JSON.](./media/app-service-deploy-complex-application-predictably/deploy-3-newresource.png)
7. Seleccione **Application Insights para aplicaciones web**; a continuación, asegúrese de que haya un plan de App Service y una aplicación seleccionados y, después, haga clic en **Agregar**.
   
   ![Muestra la selección de Application Insights para aplicaciones web, nombre, plan de App Service y Web Apps.](./media/app-service-deploy-complex-application-predictably/deploy-4-newappinsight.png)
   
   Ahora, ya podrá ver varios recursos nuevos que, según el recurso y su función, dependerán del plan de App Service o de la aplicación. Estos recursos no están habilitados por su definición existente y tiene que cambiar este ajuste.
   
   ![Ver los nuevos recursos que tienen dependencias en el plan de App Service o la aplicación.](./media/app-service-deploy-complex-application-predictably/deploy-5-appinsightresources.png)
8. En el esquema de JSON, haga clic en **appInsights AutoScale** para resaltar su código JSON. Se trata de la configuración de escalado del plan de App Service.
9. En el código resaltado de JSON, busque las propiedades `location` y `enabled` y configúrelas tal como se muestra a continuación.
   
   ![Muestra las propiedades de ubicación y habilitadas en el código JSON de escalabilidad automática de appInsights y los valores que debe establecer en ellas.](./media/app-service-deploy-complex-application-predictably/deploy-6-autoscalesettings.png)
10. En el esquema de JSON, haga clic en **CPUHigh appInsights** para resaltar su código JSON. Se trata de una alerta.
11. Busque las propiedades `location` y `isEnabled` y configúrelas tal como se muestra a continuación. Haga lo mismo para las otras tres alertas (bombilla púrpura).
    
    ![Muestra las propiedades Location e isEnabled en el código JSON de CPUHigh appInsights y los valores que debe establecer en ellas.](./media/app-service-deploy-complex-application-predictably/deploy-7-alerts.png)
12. Ahora ya puede realizar la implementación. A continuación, haga clic con el botón secundario en el proyecto y seleccione **Implementar** > **New Implementarment**.
    
    ![Muestra cómo implementar el nuevo proyecto.](./media/app-service-deploy-complex-application-predictably/deploy-8-newdeployment.png)
13. Inicie sesión en su cuenta de Azure si aún no lo ha hecho.
14. Seleccione un grupo de recursos existente en su suscripción o cree uno nuevo y seleccione **azuredeploy.json**, y, a continuación, haga clic en **Editar parámetros**.
    
    ![Muestra cómo editar los parámetros en el archivo azuredeploy.json.](./media/app-service-deploy-complex-application-predictably/deploy-9-deployconfig.png)
    
    Ahora podrá editar todos los parámetros definidos en el archivo de plantilla en una cómoda tabla. Los parámetros que definen los valores predeterminados ya tendrán sus valores predeterminados y los parámetros que definen una lista de valores permitidos se mostrarán como menús desplegables.
    
    ![Muestra los parámetros que definen una lista de valores permitidos como listas desplegables.](./media/app-service-deploy-complex-application-predictably/deploy-10-parametereditor.png)
15. Rellene todos los parámetros vacíos y use la [dirección de repositorio de GitHub para ToDoApp](https://github.com/azure-appservice-samples/ToDoApp.git) en **repoUrl**. A continuación, haga clic en **Guardar**.
    
    ![Muestra los parámetros que se acaban de rellenar para el archivo azuredeploy.json.](./media/app-service-deploy-complex-application-predictably/deploy-11-parametereditorfilled.png)
    
    > [!NOTE]
    > El escalado automático es una característica que se ofrece en el nivel **estándar** o posterior, y las alertas para planes son características que se ofrecen en el nivel **básico** o posterior; deberá establecer el parámetro **sku** en **Estándar** o **Premium** para ver todos los nuevos detalles mejorados de App Insights.
    > 
    > 
16. Haga clic en **Implementar**. Si ha seleccionado **Guardar contraseñas**, la contraseña se guardará en el archivo de parámetros como **texto sin formato**. De lo contrario, se le pedirá que escriba la contraseña de la base de datos durante el proceso de implementación.

¡Ya está! Ahora solo tiene que ir a [Azure Portal](https://portal.azure.com/) y a la herramienta [Explorador de recursos de Azure](https://resources.azure.com) para ver la configuración de escalado automático y las nuevas alertas que se han agregado a la aplicación implementada con JSON.

Con los pasos de esta sección se consigue lo siguiente:

1. Preparación del archivo de plantilla
2. Creación de un archivo de parámetros con el archivo de plantilla
3. Implementación del archivo de plantilla con el archivo de parámetros

El último paso se realiza fácilmente mediante un cmdlet de PowerShell. Para ver lo que Visual Studio hizo al implementar la aplicación, abra Scripts\Deploy-AzureResourceGroup.ps1. Hay una gran cantidad de código, pero solo voy a resaltar todo el código pertinente que necesita para implementar el archivo de plantilla con el archivo de parámetros.

![Muestra el código pertinente en el script que necesita para implementar el archivo de plantilla con el archivo de parámetros.](./media/app-service-deploy-complex-application-predictably/deploy-12-powershellsnippet.png)

El último cmdlet, `New-AzureResourceGroup`, es el que realmente realiza la acción. Todo esto debería demostrará que, con la ayuda de herramientas, es relativamente fácil implementar su aplicación de nube de manera predecible. Cada vez que ejecute el cmdlet en la misma plantilla con el mismo archivo de parámetros, obtendrá el mismo resultado.

## <a name="summary"></a>Resumen
En las operaciones de desarrollo, la repetibilidad y previsión son claves para cualquier implementación correcta de una aplicación de gran escala compuesta por microservicios. En este tutorial, ha implementado una aplicación de dos microservicios en Azure como un único grupo de recursos con la plantilla de Administrador de recursos de Azure. Espero que le haya aportado los conocimientos que necesita para iniciar la conversión de la aplicación de Azure a una plantilla a fin de que pueda aprovisionarla e implementarla de manera predecible. 

<a name="resources"></a>

## <a name="more-resources"></a>Más recursos
* [Idioma de la plantilla del Administrador de recursos de Azure](../azure-resource-manager/templates/template-syntax.md)
* [Creación de plantillas del Administrador de recursos de Azure](../azure-resource-manager/templates/template-syntax.md)
* [Funciones de la plantilla del Administrador de recursos de Azure](../azure-resource-manager/templates/template-functions.md)
* [Implementación de una aplicación con la plantilla del Administrador de recursos de Azure](../azure-resource-manager/templates/deploy-powershell.md)
* [Uso de Azure PowerShell con Azure Resource Manager](../azure-resource-manager/management/manage-resources-powershell.md)
* [Solución de problemas de implementaciones de grupo de recursos en Azure](../azure-resource-manager/templates/common-deployment-errors.md)

## <a name="next-steps"></a>Pasos siguientes

Para información sobre la sintaxis JSON y las propiedades de los tipos de recursos implementados en este artículo, consulte:

* [Microsoft.Sql/servers](/azure/templates/microsoft.sql/servers)
* [Microsoft.Sql/servers/databases](/azure/templates/microsoft.sql/servers/databases)
* [Microsoft.Sql/servers/firewallRules](/azure/templates/microsoft.sql/servers/firewallrules)
* [Microsoft.Web/serverfarms](/azure/templates/microsoft.web/serverfarms)
* [Microsoft.Web/sites](/azure/templates/microsoft.web/sites)
* [Microsoft.Web/sites/slots](/azure/templates/microsoft.web/sites/slots)
* [Microsoft.Insights/autoscalesettings](/azure/templates/microsoft.insights/autoscalesettings)