---
title: Uso de Azure Deployment Manager con plantillas de Resource Manager | Microsoft Docs
description: Use plantillas de Resource Manager con Azure Deployment Manager para implementar recursos de Azure.
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 04/02/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: a4f14a1e68042704ca8e8c49f1bd76b722c90d4d
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466298"
---
# <a name="tutorial-use-azure-deployment-manager-with-resource-manager-templates-public-preview"></a>Tutorial: Uso de Azure Deployment Manager con plantillas de Resource Manager (versión preliminar pública)

Aprenda a usar [Azure Deployment Manager](./deployment-manager-overview.md) para implementar sus aplicaciones en varias regiones. Para usar Deployment Manager, deberá crear dos plantillas:

* **Una plantilla de topología**: describe los recursos de Azure que constituyen las aplicaciones y dónde implementarlos.
* **Una plantilla de lanzamiento**: describe los pasos que se deben seguir al implementar las aplicaciones.

> [!IMPORTANT]
> Si la suscripción está marcada para Canary para probar las nuevas características de Azure, Azure Deployment Manager solo se puede usar para realizar implementaciones en las regiones de Canary. 

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Descripción del escenario
> * Descarga de los archivos del tutorial
> * Preparación de los artefactos
> * Creación de la identidad administrada definida por el usuario
> * Creación de la plantilla de topología del servicio
> * Creación de la plantilla de lanzamiento
> * Implementación de las plantillas
> * Comprobar la implementación
> * Implementación de la versión más reciente
> * Limpieza de recursos

Puede encontrar la referencia de la API REST de Azure Deployment Manager [aquí](https://docs.microsoft.com/rest/api/deploymentmanager/).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesitará lo siguiente:

* Algo de experiencia con el desarrollo de [plantillas Azure Resource Manager](./resource-group-overview.md).
* Azure Deployment Manager está en versión preliminar privada. Para registrarse con Azure Deployment Manager, rellene la [hoja de suscripción](https://aka.ms/admsignup). 
* Azure PowerShell. Para más información, consulte el artículo de [introducción a Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps).
* Cmdlets de Deployment Manager. Para instalar estos cmdlets en versión preliminar, necesita la versión más reciente de PowerShellGet. Para obtener la versión más reciente, consulte [Instalación de PowerShellGet](/powershell/gallery/installing-psget). Después de instalar PowerShellGet, cierre la ventana de PowerShell. Abra una nueva ventana con privilegios elevados de PowerShell y ejecute el siguiente comando:

    ```powershell
    Install-Module -Name Az.DeploymentManager
    ```

* [Explorador de Microsoft Azure Storage](https://azure.microsoft.com/features/storage-explorer/). El Explorador de Azure Storage no es necesario, pero facilita las cosas.

## <a name="understand-the-scenario"></a>Descripción del escenario

La plantilla de topología del servicio describe los recursos de Azure que constituyen el servicio y dónde implementarlos. La definición de topología del servicio tiene la siguiente jerarquía:

* Topología del servicio
  * Services
    * Unidades de servicio

En el diagrama siguiente se ilustra la topología de red usada en este tutorial:

![Diagrama del escenario del tutorial de Azure Deployment Manager](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-scenario-diagram.png)

Hay dos servicios asignados en las ubicaciones Oeste de EE. UU y Este de EE. UU.  Cada servicio tiene dos unidades de servicio: un aplicación web, un front-end y una cuenta de almacenamiento para el back-end. Las definiciones de unidad de servicio contienen vínculos a los archivos de plantilla y parámetros para crear las aplicaciones web y las cuentas de almacenamiento.

## <a name="download-the-tutorial-files"></a>Descarga de los archivos del tutorial

1. Descargue las [plantillas y los artefactos](https://armtutorials.blob.core.windows.net/admtutorial/ADMTutorial.zip) usados en este tutorial.
2. Descomprima los archivos en el equipo de la ubicación.

En la carpeta raíz, hay dos carpetas:

* **ADMTemplates**: contiene las plantillas de Deployment Manager, que incluyen:
  * CreateADMServiceTopology.json
  * CreateADMServiceTopology.Parameters.json
  * CreateADMRollout.json
  * CreateADMRollout.Parameters.json
* **ArtifactStore**: contiene los artefactos de plantilla y los artefactos binarios. Consulte [Preparación de los artefactos](#prepare-the-artifacts).

Tenga en cuenta que hay dos conjuntos de plantillas.  Un conjunto son las plantillas de Deployment Manager que se usan para implementar la topología del servicio y el lanzamiento; el otro conjunto se llama desde las unidades de servicio para crear servicios web y cuentas de almacenamiento.

## <a name="prepare-the-artifacts"></a>Preparación de los artefactos

La carpeta ArtifactStore de la descarga contiene dos carpetas:

![Diagrama de origen de artefacto del tutorial de Azure Deployment Manager](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-artifact-source-diagram.png)

* La carpeta **templates** contiene los artefactos de plantilla. **1.0.0.0** y **1.0.0.1** representan las dos versiones de los artefactos binarios. Dentro de cada versión, hay una carpeta para cada servicio (Service East U.S. y Service West U.S.). Cada servicio tiene un par de archivos de plantilla y parámetros: uno para crear una cuenta de almacenamiento y otro par para crear una aplicación web. La plantilla de aplicación web llama a un paquete comprimido que contiene los archivos de aplicación web. El archivo comprimido es un artefacto binario almacenado en la carpeta de archivos binarios.
* La carpeta **binaries** contiene los artefactos binarios. **1.0.0.0** y **1.0.0.1** representan las dos versiones de los artefactos binarios. Dentro de cada versión, hay un archivo ZIP para crear la aplicación web en la ubicación Oeste de EE. UU. y otro archivo ZIP para crear la aplicación web en la ubicación Este de EE. UU.

Las dos versiones (1.0.0.0 y 1.0.0.1) son para la [implementación de revisiones](#deploy-the-revision). Aunque los artefactos de plantilla y los artefactos binarios tienen dos versiones, solo los artefactos binarios son diferentes entre las dos versiones. En la práctica, los artefactos binarios se actualizan con más frecuencia que los artefactos de plantilla.

1. Abra **\ArtifactStore\templates\1.0.0.0\ServiceWUS\CreateStorageAccount.json** en un editor de texto. Se trata de una plantilla básica para crear una cuenta de almacenamiento.  
2. Abra **\ArtifactStore\templates\1.0.0.0\ServiceWUS\CreateWebApplication.json**. 

    ![Tutorial de Azure Deployment Manager: creación de una plantilla de aplicación web](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-create-web-application-packageuri.png)

    La plantilla llama a un paquete de implementación, que contiene los archivos de la aplicación web. En este tutorial, el paquete comprimido solo contiene un archivo index.html.
3. Abra **\ArtifactStore\templates\1.0.0.0\ServiceWUS\CreateWebApplicationParameters.json**. 

    ![Tutorial de Azure Deployment Manager: creación de los parámetros de plantilla de aplicación web containerRoot](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-create-web-application-parameters-deploypackageuri.png)

    El valor de deployPackageUri es la ruta de acceso al paquete de implementación. El parámetro contiene una variable **$containerRoot**. El valor de $containerRoot se proporciona en la [plantilla de lanzamiento](#create-the-rollout-template) mediante la concatenación de la ubicación de SAS de origen del artefacto, la raíz del artefacto y deployPackageUri.
4. Abra **\ArtifactStore\binaries\1.0.0.0\helloWorldWebAppWUS.zip\index.html**.  

    ```html
    <html>
      <head>
        <title>Azure Deployment Manager tutorial</title>
      </head>
      <body>
        <p>Hello world from west U.S.!</p>
        <p>Version 1.0.0.0</p>
      </body>
    </html>
    ```

    El código HTML muestra la ubicación y la información de versión. El archivo binario en la carpeta 1.0.0.1 muestra "Version 1.0.0.1". Después de implementar el servicio, puede examinar estas páginas.
5. Consulte otros archivos de artefacto. Le ayudará a comprender el mejor escenario.

Los artefactos de plantilla se usan en la plantilla de topología del servicio y los artefactos binarios se usan en la plantilla de lanzamiento. La plantilla de topología y la plantilla de lanzamiento definen un recurso de Azure de origen del artefacto, que es un recurso usado para que Resource Manager apunte a los artefactos binarios y de plantilla que se usan en la implementación. Para simplificar el tutorial, se usa una cuenta de almacenamiento para almacenar los artefactos de plantilla y los artefactos binarios. Ambos orígenes de artefacto apuntan a la misma cuenta de almacenamiento.

1. Cree una cuenta de Azure Storage. Para ver las instrucciones, consulte [Inicio rápido: Carga, descarga y enumeración de blobs mediante Azure Portal](../storage/blobs/storage-quickstart-blobs-portal.md).
2. Cree un contenedor de blobs en la cuenta de almacenamiento.
3. Copie las dos carpetas (archivos binarios y plantillas) y el contenido de las dos carpetas en el contenedor de blobs. El [Explorador de Microsoft Azure Storage](https://go.microsoft.com/fwlink/?LinkId=708343&clcid=0x409) admite la característica de arrastrar y colocar.
4. Obtenga la ubicación de SAS del contenedor mediante las siguientes instrucciones:

    1. Desde el Explorador de Azure Storage, vaya al contenedor de blobs.
    2. Haga clic con el botón derecho en el contenedor de blobs del panel de la izquierda y, luego, seleccione **Get Shared Access Signature** (Obtener firma de acceso compartido).
    3. Configure **Start time** (Hora de inicio) y **Expiry time** (Hora de expiración).
    4. Seleccione **Crear**.
    5. Realice una copia de la dirección URL. Esta dirección URL es necesaria para rellenar un campo en los dos archivos de parámetros, [archivo de parámetros de topología](#topology-parameters-file) y [archivo de parámetros de lanzamiento](#rollout-parameters-file).

## <a name="create-the-user-assigned-managed-identity"></a>Creación de la identidad administrada asignada por el usuario

Más adelante en el tutorial, implementará un lanzamiento. Para realizar las acciones de implementación se necesita una identidad administrada asignada por el usuario (por ejemplo, implementar las aplicaciones web y la cuenta de almacenamiento). Esta identidad debe tener acceso concedido a la suscripción de Azure en la que va a implementar el servicio y debe tener permisos suficientes para completar la implementación del artefacto.

Deberá crear una identidad administrada asignada por el usuario y configurar el control de acceso de la suscripción.

> [!IMPORTANT]
> La identidad administrada asignada por el usuario debe estar en la misma ubicación que el [lanzamiento](#create-the-rollout-template). Actualmente, los recursos de Deployment Manager, incluido el lanzamiento, solo se pueden crear en Centro de EE. UU. y Este de EE. UU. 2. Sin embargo, esto solo es cierto para los recursos de Deployment Manager (como por ejemplo, la topología del servicio, los servicios, las unidades del servicio, el lanzamiento y los pasos). Los recursos de destino se pueden implementar en cualquier región de Azure compatible. En este tutorial, por ejemplo, los recursos de Deployment Manager se implementan en la región Centro de EE. UU., pero los servicios se implementan en Este de EE. UU. y Oeste de EE. UU. Esta restricción se eliminará próximamente.

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
2. Cree una [identidad administrada asignada por el usuario](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md).
3. En el portal, seleccione **Suscripciones** en el menú izquierdo y, luego, seleccione su suscripción.
4. Seleccione **Control de acceso (IAM)** y después **Agregar asignación de rol**.
5. Escriba o seleccione los siguientes valores:

    ![Tutorial de Azure Deployment Manager: control de acceso de identidad administrada asignada por el usuario](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-access-control.png)

    * **Rol**: proporcione permisos suficientes para completar la implementación de los artefactos (las aplicaciones web y las cuentas de almacenamiento). Para los fines de este tutorial, seleccione **Colaborador**. En la práctica, querrá restringir los permisos al mínimo.
    * **Asignar acceso a**: seleccione **Identidad administrada asignada por el usuario**.
    * Seleccione la identidad administrada asignada por el usuario que creó anteriormente en el tutorial.
6. Seleccione **Guardar**.

## <a name="create-the-service-topology-template"></a>Creación de la plantilla de topología del servicio

Abra **\ADMTemplates\CreateADMServiceTopology.json**.

### <a name="the-parameters"></a>Los parámetros

La plantilla contiene los parámetros siguientes:

![Tutorial de Azure Deployment Manager: parámetros de plantilla de topología](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-parameters.png)

* **namePrefix**: este prefijo se usa para crear los nombres de los recursos de Deployment Manager. Por ejemplo, con el prefijo "jdoe", el nombre de la topología del servicio es **jdoe**ServiceTopology.  Los nombres de recursos se definen en la sección de variables de esta plantilla.
* **azureResourcelocation**: para simplificar el tutorial, todos los recursos comparten esta ubicación, a menos que se especifique lo contrario. Actualmente, los recursos de Azure Deployment Manager solo se pueden crear en **Centro de EE. UU.** o **Este de EE. UU. 2**.
* **artifactSourceSASLocation**: el URI de SAS del contenedor de blobs donde se almacenan los archivos de plantilla y parámetros de la unidad de servicio para la implementación.  Consulte [Preparación de los artefactos](#prepare-the-artifacts).
* **templateArtifactRoot**: la ruta de desplazamiento del contenedor de blobs donde se almacenan las plantillas y los parámetros. El valor predeterminado es **templates/1.0.0.0**. No cambie este valor a menos que quiera cambiar la estructura de carpetas que se explica en [Preparación de los artefactos](#prepare-the-artifacts). En este tutorial, se usan rutas de acceso relativas.  La ruta de acceso completa se construye mediante la concatenación de **artifactSourceSASLocation**, **templateArtifactRoot** y **templateArtifactSourceRelativePath** (o **parametersArtifactSourceRelativePath**).
* **targetSubscriptionID**: el identificador de suscripción con el que se van a implementar y facturar los recursos de Deployment Manager. Use el identificador de suscripción de este tutorial.

### <a name="the-variables"></a>Las variables

En la sección de variables se definen los nombres de los recursos, las ubicaciones de Azure de los dos servicios: **Service WUS** y **Service EUS** y las rutas de acceso de los artefactos:

![Tutorial de Azure Deployment Manager: variables de plantilla de topología](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-variables.png)

Compare las rutas de acceso de los artefactos con la estructura de carpetas que ha cargado en la cuenta de almacenamiento. Observe que las rutas de acceso de los artefactos son relativas. La ruta de acceso completa se construye mediante la concatenación de **artifactSourceSASLocation**, **templateArtifactRoot** y **templateArtifactSourceRelativePath** (o **parametersArtifactSourceRelativePath**).

### <a name="the-resources"></a>Los recursos

En el nivel raíz, hay dos recursos definidos: *un origen de artefacto* y *una topología del servicio*.

La definición de origen de artefacto es:

![Tutorial de Azure Deployment Manager: origen de artefacto de recursos de plantilla de topología](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-resources-artifact-source.png)

En la captura de pantalla siguiente solo se muestran algunas partes de la topología del servicio, los servicios y las definiciones de unidades de servicio:

![Tutorial de Azure Deployment Manager: topología del servicio de recursos de plantilla de topología](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-resources-service-topology.png)

* **artifactSourceId** se usa para asociar el recurso de origen de artefacto con el recurso de topología del servicio.
* **dependsOn**: todos los recursos de topología del servicio dependen del recurso de origen de artefacto.
* Los **artefactos** apuntan a los artefactos de plantilla.  Aquí se usan rutas de acceso relativas. La ruta de acceso completa se construye mediante la concatenación de artifactSourceSASLocation (definido en el origen de artefacto), artifactRoot (definido en el origen de artefacto) y templateArtifactSourceRelativePath (o parametersArtifactSourceRelativePath).

### <a name="topology-parameters-file"></a>Archivo de parámetros de topología

Creará un archivo de parámetros que se usa con la plantilla de la topología.

1. Abra **\ADMTemplates\CreateADMServiceTopology.Parameters** en Visual Studio Code o cualquier editor de texto.
2. Rellene los valores de parámetros:

    * **namePrefix**: escriba una cadena con cuatro o cinco caracteres. Este prefijo se usa para crear nombres de recursos de Azure únicos.
    * **azureResourceLocation**: si no está familiarizado con las ubicaciones de Azure, use **centralus** en este tutorial.
    * **artifactSourceSASLocation**: el URI de SAS del directorio raíz (el contenedor de blobs) donde se almacenan los archivos de plantilla y parámetros de la unidad de servicio para la implementación.  Consulte [Preparación de los artefactos](#prepare-the-artifacts).
    * **templateArtifactRoot**: a menos que cambie la estructura de carpetas de los artefactos, use **templates/1.0.0.0** en este tutorial.
    * **targetScriptionID**: Especifique el identificador de su suscripción a Azure.

> [!IMPORTANT]
> La plantilla de topología y la plantilla de lanzamiento comparten algunos parámetros comunes. Estos parámetros deben tener los mismos valores. Los parámetros son: **namePrefix**, **azureResourceLocation** y **artifactSourceSASLocation** (ambos orígenes de artefacto comparten la misma cuenta de almacenamiento en este tutorial).

## <a name="create-the-rollout-template"></a>Creación de la plantilla de lanzamiento

Abra **\ADMTemplates\CreateADMRollout.json**.

### <a name="the-parameters"></a>Los parámetros

La plantilla contiene los parámetros siguientes:

![Tutorial de Azure Deployment Manager: parámetros de plantilla de lanzamiento](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-parameters.png)

* **namePrefix**: este prefijo se usa para crear los nombres de los recursos de Deployment Manager. Por ejemplo, con el prefijo "jdoe", el nombre del lanzamiento es **jdoe**Rollout.  Los nombres se definen en la sección de variables de la plantilla.
* **azureResourcelocation**: para simplificar el tutorial, todos los recursos de Deployment Manager comparten esta ubicación, a menos que se especifique lo contrario. Actualmente, los recursos de Azure Deployment Manager solo se pueden crear en **Centro de EE. UU.** o **Este de EE. UU. 2**.
* **artifactSourceSASLocation**: el URI de SAS del directorio raíz (el contenedor de blobs) donde se almacenan los archivos de plantilla y parámetros de la unidad de servicio para la implementación.  Consulte [Preparación de los artefactos](#prepare-the-artifacts).
* **binaryArtifactRoot**:  el valor predeterminado es **binaries/1.0.0.0**. No cambie este valor a menos que quiera cambiar la estructura de carpetas que se explica en [Preparación de los artefactos](#prepare-the-artifacts). En este tutorial, se usan rutas de acceso relativas.  La ruta de acceso completa se construye mediante la concatenación de **artifactSourceSASLocation**, **binaryArtifactRoot** y el valor de **deployPackageUri** especificado en el archivo CreateWebApplicationParameters.json.  Consulte [Preparación de los artefactos](#prepare-the-artifacts).
* **managedIdentityID**: la identidad administrada asignada por el usuario que realiza las acciones de implementación. Consulte [Creación de la identidad administrada asignada por el usuario](#create-the-user-assigned-managed-identity).

### <a name="the-variables"></a>Las variables

En la sección de variables se definen los nombres de los recursos. Asegúrese de que el nombre de la topología del servicio, los nombres de servicio y los nombres de unidad de servicio coincidan con los nombres definidos en la [plantilla de topología](#create-the-service-topology-template).

![Tutorial de Azure Deployment Manager: variables de plantilla de lanzamiento](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-variables.png)

### <a name="the-resources"></a>Los recursos

En el nivel raíz, hay tres recursos definidos: un origen de artefacto, un paso y un lanzamiento.

La definición del origen de artefacto es idéntica a la definida en la plantilla de topología.  Para más información, consulte [Creación de la plantilla de topología de servicio](#create-the-service-topology-template).

En la siguiente captura de pantalla se muestra la definición del paso de espera:

![Tutorial de Azure Deployment Manager: paso de espera de recursos de plantilla de lanzamiento](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-resources-wait-step.png)

La duración usa el [estándar ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Durations). **PT1M** (se requieren letras mayúsculas) es un ejemplo de una espera de 1 minuto. 

En la siguiente captura de pantalla solo se muestran algunas partes de la definición de lanzamiento:

![Tutorial de Azure Deployment Manager: lanzamiento de recursos de plantilla de lanzamiento](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-resources-rollout.png)

* **dependsOn**: el recurso de lanzamiento depende el recurso de origen de artefacto y de cualquiera de los pasos definidos.
* **artifactSourceId**: se usa para asociar el recurso de origen de artefacto con el recurso de lanzamiento.
* **targetServiceTopologyId**: se usa para asociar el recurso de topología del servicio con el recurso de lanzamiento.
* **deploymentTargetId**: es el identificador del recurso de unidad de servicio del recurso de topología del servicio.
* **preDeploymentSteps** y **postDeploymentSteps**: contienen los pasos de lanzamiento. En la plantilla, se llama a un paso de espera.
* **dependsOnStepGroups**: configura las dependencias entre los grupos de pasos.

### <a name="rollout-parameters-file"></a>Archivo de parámetros de lanzamiento

Creará un archivo de parámetros que se usa con la plantilla de lanzamiento.

1. Abra **\ADMTemplates\CreateADMRollout.Parameters** en Visual Studio Code o en cualquier editor de texto.
2. Rellene los valores de parámetros:

    * **namePrefix**: escriba una cadena con cuatro o cinco caracteres. Este prefijo se usa para crear nombres de recursos de Azure únicos.
    * **azureResourceLocation**: Actualmente, los recursos de Azure Deployment Manager solo se pueden crear en **Centro de EE. UU.** o **Este de EE. UU. 2**.
    * **artifactSourceSASLocation**: el URI de SAS del directorio raíz (el contenedor de blobs) donde se almacenan los archivos de plantilla y parámetros de la unidad de servicio para la implementación.  Consulte [Preparación de los artefactos](#prepare-the-artifacts).
    * **binaryArtifactRoot**: a menos que cambie la estructura de carpetas de los artefactos, use **binaries/1.0.0.0** en este tutorial.
    * **managedIdentityID**: introduzca la identidad administrada asignada por el usuario. Consulte [Creación de la identidad administrada asignada por el usuario](#create-the-user-assigned-managed-identity). La sintaxis es:

        ```
        "/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/Microsoft.ManagedIdentity/userassignedidentities/<ManagedIdentityName>"
        ```

> [!IMPORTANT]
> La plantilla de topología y la plantilla de lanzamiento comparten algunos parámetros comunes. Estos parámetros deben tener los mismos valores. Los parámetros son: **namePrefix**, **azureResourceLocation** y **artifactSourceSASLocation** (ambos orígenes de artefacto comparten la misma cuenta de almacenamiento en este tutorial).

## <a name="deploy-the-templates"></a>Implementación de las plantillas

Azure PowerShell puede usarse para implementar las plantillas. 

1. Ejecute el script para implementar la topología del servicio.

    ```azurepowershell
    $resourceGroupName = "<Enter a Resource Group Name>"
    $location = "Central US"  
    $filePath = "<Enter the File Path to the Downloaded Tutorial Files>"

    # Create a resource group
    New-AzResourceGroup -Name $resourceGroupName -Location "$location"

    # Create the service topology
    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile "$filePath\ADMTemplates\CreateADMServiceTopology.json" `
        -TemplateParameterFile "$filePath\ADMTemplates\CreateADMServiceTopology.Parameters.json"
    ```

    > [!NOTE]
    > `New-AzResourceGroupDeployment` es una llamada asincrónica. El mensaje de operación correcta solo significa que la implementación ha comenzado correctamente. Para comprobar la implementación, consulte los pasos 2 y 4 de este procedimiento.

2. Compruebe la topología del servicio y que los recursos subyacentes se hayan creado correctamente mediante Azure Portal:

    ![Tutorial de Azure Deployment Manager: recursos de topología del servicio implementados](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-deployed-topology-resources.png)

    Se debe seleccionar **Mostrar tipos ocultos** para ver los recursos.

3. <a id="deploy-the-rollout-template"></a>Implemente la plantilla de lanzamiento:

    ```azurepowershell
    # Create the rollout
    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile "$filePath\ADMTemplates\CreateADMRollout.json" `
        -TemplateParameterFile "$filePath\ADMTemplates\CreateADMRollout.Parameters.json"
    ```

4. Compruebe el progreso del lanzamiento mediante el siguiente script de PowerShell:

    ```azurepowershell
    # Get the rollout status
    $rolloutname = "<Enter the Rollout Name>" # "adm0925Rollout" is the rollout name used in this tutorial
    Get-AzDeploymentManagerRollout `
        -ResourceGroupName $resourceGroupName `
        -Name $rolloutName `
        -Verbose
    ```

    Antes de ejecutar este cmdlet se deben instalar los cmdlets de PowerShell de Deployment Manager. Consulte Requisitos previos. El modificador -Verbose se puede usar para ver toda la salida.

    En el ejemplo siguiente se muestra el estado de ejecución:

    ```
    VERBOSE:

    Status: Succeeded
    ArtifactSourceId: /subscriptions/<AzureSubscriptionID>/resourceGroups/adm0925rg/providers/Microsoft.DeploymentManager/artifactSources/adm0925ArtifactSourceRollout
    BuildVersion: 1.0.0.0

    Operation Info:
        Retry Attempt: 0
        Skip Succeeded: False
        Start Time: 03/05/2019 15:26:13
        End Time: 03/05/2019 15:31:26
        Total Duration: 00:05:12

    Service: adm0925ServiceEUS
        TargetLocation: EastUS
        TargetSubscriptionId: <AzureSubscriptionID>

        ServiceUnit: adm0925ServiceEUSStorage
            TargetResourceGroup: adm0925ServiceEUSrg

            Step: Deploy
                Status: Succeeded
                StepGroup: stepGroup3
                Operation Info:
                    DeploymentName: 2F535084871E43E7A7A4CE7B45BE06510adm0925ServiceEUSStorage
                    CorrelationId: 0b6f030d-7348-48ae-a578-bcd6bcafe78d
                    Start Time: 03/05/2019 15:26:32
                    End Time: 03/05/2019 15:27:41
                    Total Duration: 00:01:08
                Resource Operations:

                    Resource Operation 1:
                    Name: txq6iwnyq5xle
                    Type: Microsoft.Storage/storageAccounts
                    ProvisioningState: Succeeded
                    StatusCode: OK
                    OperationId: 64A6E6EFEF1F7755

    ...

    ResourceGroupName       : adm0925rg
    BuildVersion            : 1.0.0.0
    ArtifactSourceId        : /subscriptions/<SubscriptionID>/resourceGroups/adm0925rg/providers/Microsoft.DeploymentManager/artifactSources/adm0925ArtifactSourceRollout
    TargetServiceTopologyId : /subscriptions/<SubscriptionID>/resourceGroups/adm0925rg/providers/Microsoft.DeploymentManager/serviceTopologies/adm0925ServiceTopology
    Status                  : Running
    TotalRetryAttempts      : 0
    OperationInfo           : Microsoft.Azure.Commands.DeploymentManager.Models.PSRolloutOperationInfo
    Services                : {adm0925ServiceEUS, adm0925ServiceWUS}
    Name                    : adm0925Rollout
    Type                    : Microsoft.DeploymentManager/rollouts
    Location                : centralus
    Id                      : /subscriptions/<SubscriptionID>/resourcegroups/adm0925rg/providers/Microsoft.DeploymentManager/rollouts/adm0925Rollout
    Tags                    :
    ```

    Una vez que el lanzamiento se ha implementado correctamente, verá que se han creado dos grupos de recursos más, uno para cada servicio.

## <a name="verify-the-deployment"></a>Comprobar la implementación

1. Abra [Azure Portal](https://portal.azure.com).
2. Vaya a las aplicaciones web recién creadas en los grupos de recursos creados mediante la implementación de lanzamiento.
3. Abra la aplicación web en un explorador web. Compruebe la ubicación y la versión del archivo index.html.

## <a name="deploy-the-revision"></a>Implementación de la revisión

Cuando tenga una nueva versión (1.0.0.1) de la aplicación web, puede usar el procedimiento siguiente para volver a implementar dicha aplicación.

1. Abra CreateADMRollout.Parameters.json.
2. Actualice **binaryArtifactRoot** a **binaries/1.0.0.1**.
3. Vuelva a implementar el lanzamiento como se indica en [Implementación de las plantillas](#deploy-the-rollout-template).
4. Compruebe la implementación como se indica en [Comprobación de la implementación](#verify-the-deployment). La página web muestra la versión 1.0.0.1.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando los recursos de Azure ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Use el campo **Filtrar por nombre** para limitar los grupos de recursos creados en este tutorial. Habrá 3 o 4:

    * **&lt;namePrefix>rg**: contiene los recursos de Deployment Manager.
    * **&lt;namePrefix>ServiceWUSrg**: contiene los recursos definidos por ServiceWUS.
    * **&lt;namePrefix>ServiceEUSrg**: contiene los recursos definidos por ServiceEUS.
    * El grupo de recursos de la identidad administrada definida por el usuario.
3. Seleccione el nombre del grupo de recursos.  
4. Seleccione **Eliminar grupo de recursos** del menú superior.
5. Repita los dos últimos pasos para eliminar otros grupos de recursos creados en este tutorial.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a usar Azure Deployment Manager. Para integrar la supervisión del estado en Azure Deployment Manager, consulte [Tutorial: Uso de la comprobación de estado en Azure Deployment Manager](./deployment-manager-tutorial-health-check.md).
