---
title: Creación de un entorno de ejecución de integración autohospedado compartido con PowerShell
description: Aprenda a crear un entorno de ejecución de integración autohospedado compartido en Azure Data Factory, para que varias factorías de datos puedan obtener acceso al entorno de ejecución de integración.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.author: abnarain
author: nabhishek
manager: anansub
ms.custom: seo-lt-2019
ms.date: 06/10/2020
ms.openlocfilehash: 28836d0b1109952d8cf81c66b44b1f98d9b770bf
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88136055"
---
# <a name="create-a-shared-self-hosted-integration-runtime-in-azure-data-factory"></a>Creación de un entorno de ejecución de integración autohospedado compartido en Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En esta guía se muestra cómo crear un entorno de ejecución de integración autohospedado compartido en Azure Data Factory. A continuación, puede usar el tiempo de ejecución de integración autohospedado compartido en otra factoría de datos.

## <a name="create-a-shared-self-hosted-ir-using-azure-data-factory-ui"></a>Creación de un IR autohospedado compartido mediante la interfaz de usuario de Azure Data Factory

Para crear un entorno de ejecución de integración autohospedado compartido mediante la interfaz de usuario de Azure Data Factory, puede llevar a cabo los siguientes pasos:

1. En el entorno de ejecución de integración autohospedado que se va a compartir, seleccione **Conceder permiso a otra factoría de datos** y, en la página "Configuración de Integration Runtime", seleccione la factoría de datos en la que desee crear el entorno de ejecución de integración vinculado.
      
    ![El botón para conceder el permiso en la pestaña Compartir](media/create-self-hosted-integration-runtime/grant-permissions-IR-sharing.png)  
    
2. Anote y copie el "Identificador de recurso" anterior del entorno de ejecución de integración autohospedado que se va a compartir.
         
3. En la factoría de datos en la que se concedieron los permisos, cree un nuevo IR autohospedado (vinculado) y escriba el identificador de recurso.
      
    ![Botón para crear un entorno de ejecución de integración autohospedado](media/create-self-hosted-integration-runtime/create-linkedir-1.png)
   
    ![Botón para crear un entorno de ejecución de integración autohospedado vinculado](media/create-self-hosted-integration-runtime/create-linkedir-2.png) 

    ![Cuadros para nombre y el identificador de recurso](media/create-self-hosted-integration-runtime/create-linkedir-3.png)

## <a name="create-a-shared-self-hosted-ir-using-azure-powershell"></a>Creación de un IR autohospedado compartido mediante Azure PowerShell

Para crear un entorno de ejecución de integración autohospedado compartido mediante Azure PowerShell, puede llevar a cabo los siguientes pasos: 
1. Creación de una factoría de datos. 
1. Cree una instancia de Integration Runtime autohospedada.
1. Comparta el entorno de ejecución de integración autohospedado con otras factorías de datos.
1. Cree un entorno de ejecución de integración vinculado.
1. Revoque el uso compartido.

### <a name="prerequisites"></a>Requisitos previos 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar. 

- **Azure PowerShell**. Siga las instrucciones de [Install Azure PowerShell on Windows with PowerShellGet](https://docs.microsoft.com/powershell/azure/install-az-ps) (Instalar Azure PowerShell en Windows con PowerShellGet). Ejecute un script con PowerShell para crear un entorno de ejecución de integración autohospedado que se pueda compartir con otras factorías de datos. 

> [!NOTE]  
> Para obtener una lista de las regiones de Azure en las que Data Factory está disponible actualmente, seleccione las regiones que le interesen en la página [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=data-factory).

### <a name="create-a-data-factory"></a>Crear una factoría de datos

1. Inicie Windows PowerShell Integrated Scripting Environment (ISE).

1. Cree las variables. Copie y pegue el siguiente script: Reemplace las variables, como **SubscriptionName** y **ResourceGroupName**, con valores reales: 

    ```powershell
    # If input contains a PSH special character, e.g. "$", precede it with the escape character "`" like "`$". 
    $SubscriptionName = "[Azure subscription name]" 
    $ResourceGroupName = "[Azure resource group name]" 
    $DataFactoryLocation = "EastUS" 

    # Shared Self-hosted integration runtime information. This is a Data Factory compute resource for running any activities 
    # Data factory name. Must be globally unique 
    $SharedDataFactoryName = "[Shared Data factory name]" 
    $SharedIntegrationRuntimeName = "[Shared Integration Runtime Name]" 
    $SharedIntegrationRuntimeDescription = "[Description for Shared Integration Runtime]"

    # Linked integration runtime information. This is a Data Factory compute resource for running any activities
    # Data factory name. Must be globally unique
    $LinkedDataFactoryName = "[Linked Data factory name]"
    $LinkedIntegrationRuntimeName = "[Linked Integration Runtime Name]"
    $LinkedIntegrationRuntimeDescription = "[Description for Linked Integration Runtime]"
    ```

1. Inicie sesión y seleccione una suscripción. Agregue el código siguiente al script para iniciar sesión y seleccione la suscripción de Azure:

    ```powershell
    Connect-AzAccount
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

1. Cree un grupo de recursos y una factoría de datos.

    > [!NOTE]  
    > Este paso es opcional. Si ya tiene una factoría de datos, omita este paso. 

    Cree un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) con el comando [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran recursos de Azure como un grupo. En el ejemplo siguiente se crea un grupo de recursos denominado `myResourceGroup` en la ubicación Europa Occidental. 

    ```powershell
    New-AzResourceGroup -Location $DataFactoryLocation -Name $ResourceGroupName
    ```

    Ejecute el comando siguiente para crear una factoría de datos: 

    ```powershell
    Set-AzDataFactoryV2 -ResourceGroupName $ResourceGroupName `
                             -Location $DataFactoryLocation `
                             -Name $SharedDataFactoryName
    ```

### <a name="create-a-self-hosted-integration-runtime"></a>Creación de una instancia de Integration Runtime autohospedada

> [!NOTE]  
> Este paso es opcional. Si ya tiene el entorno de ejecución de integración autohospedado que quiere compartir con otras factorías de datos, omita este paso.

Ejecute el siguiente comando para crear un entorno de ejecución de integración autohospedado:

```powershell
$SharedIR = Set-AzDataFactoryV2IntegrationRuntime `
    -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $SharedDataFactoryName `
    -Name $SharedIntegrationRuntimeName `
    -Type SelfHosted `
    -Description $SharedIntegrationRuntimeDescription
```

#### <a name="get-the-integration-runtime-authentication-key-and-register-a-node"></a>Obtención de la clave de autenticación del entorno de ejecución de integración y registro de un nodo

Ejecute el siguiente comando para obtener la clave de autenticación para el entorno de ejecución de integración autohospedado:

```powershell
Get-AzDataFactoryV2IntegrationRuntimeKey `
    -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $SharedDataFactoryName `
    -Name $SharedIntegrationRuntimeName
```

La respuesta contiene la clave de autenticación para este entorno de ejecución de integración autohospedado. Esta clave se usa al registrar el nodo del entorno de ejecución de integración.

#### <a name="install-and-register-the-self-hosted-integration-runtime"></a>Instalación y registro del entorno de ejecución de integración autohospedado

1. Descargue el programa de instalación del entorno de ejecución de integración autohospedado desde [Azure Data Factory Integration Runtime](https://aka.ms/dmg) (Entorno de ejecución de integración de Azure Data Factory).

2. Ejecute el programa de instalación para instalar la integración autohospedada en un equipo local.

3. Registre la nueva integración autohospedada con la clave de autenticación que obtuvo en el paso anterior.

### <a name="share-the-self-hosted-integration-runtime-with-another-data-factory"></a>Uso compartido del entorno de ejecución de integración autohospedado con otra factoría de datos

#### <a name="create-another-data-factory"></a>Creación de otra factoría de datos

> [!NOTE]  
> Este paso es opcional. Si ya dispone de la factoría de datos con la que quiere compartir contenido, omita este paso. Pero, para poder agregar o quitar asignaciones de roles en otra factoría de datos, debe tener permisos `Microsoft.Authorization/roleAssignments/write` y `Microsoft.Authorization/roleAssignments/delete` tales como [Administrador de acceso de usuarios](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#user-access-administrator) o [Propietario](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#owner).

```powershell
$factory = Set-AzDataFactoryV2 -ResourceGroupName $ResourceGroupName `
    -Location $DataFactoryLocation `
    -Name $LinkedDataFactoryName
```
#### <a name="grant-permission"></a>Concesión de permisos

Conceda permiso a la factoría de datos que necesite obtener acceso al entorno de ejecución de integración autohospedado que creó y registró.

> [!IMPORTANT]  
> ¡No omita este paso!

```powershell
New-AzRoleAssignment `
    -ObjectId $factory.Identity.PrincipalId ` #MSI of the Data Factory with which it needs to be shared
    -RoleDefinitionName 'Contributor' `
    -Scope $SharedIR.Id
```

### <a name="create-a-linked-self-hosted-integration-runtime"></a>Creación de un entorno de ejecución de integración autohospedado vinculado

Ejecute el siguiente comando para crear un entorno de ejecución de integración autohospedado vinculado:

```powershell
Set-AzDataFactoryV2IntegrationRuntime `
    -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $LinkedDataFactoryName `
    -Name $LinkedIntegrationRuntimeName `
    -Type SelfHosted `
    -SharedIntegrationRuntimeResourceId $SharedIR.Id `
    -Description $LinkedIntegrationRuntimeDescription
```

Ahora puede usar este entorno de ejecución de integración vinculado en cualquier servicio vinculado. El entorno de ejecución de integración vinculado usa el entorno de ejecución de integración compartido para ejecutar actividades.

### <a name="revoke-integration-runtime-sharing-from-a-data-factory"></a>Revocación del uso compartido de un entorno de ejecución de integración de una factoría de datos

Para revocar el acceso de una factoría de datos a un entorno de ejecución de integración compartido, ejecute el comando siguiente:

```powershell
Remove-AzRoleAssignment `
    -ObjectId $factory.Identity.PrincipalId `
    -RoleDefinitionName 'Contributor' `
    -Scope $SharedIR.Id
```

Para quitar el entorno de ejecución de integración vinculado existente, puede ejecutar el comando siguiente en el entorno de ejecución de integración compartido:

```powershell
Remove-AzDataFactoryV2IntegrationRuntime `
    -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $SharedDataFactoryName `
    -Name $SharedIntegrationRuntimeName `
    -Links `
    -LinkedDataFactoryName $LinkedDataFactoryName
```

### <a name="next-steps"></a>Pasos siguientes

- Revise los [conceptos del entorno de ejecución de integración en Azure Data Factory](https://docs.microsoft.com/azure/data-factory/concepts-integration-runtime).

- Aprenda a [crear un entorno de ejecución de integración autohospedado en Azure Portal](https://docs.microsoft.com/azure/data-factory/create-self-hosted-integration-runtime).
