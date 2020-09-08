---
title: Uso de scripts de implementación de plantillas | Microsoft Docs
description: Aprenda a usar scripts de implementación en plantillas de Azure Resource Manager.
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: carmonm
editor: ''
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 08/25/2020
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: e1094befcc6b3a6e9d56ba3b603dc45fcb91ba13
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88825501"
---
# <a name="tutorial-use-deployment-scripts-to-create-a-self-signed-certificate-preview"></a>Tutorial: Uso de scripts de implementación para crear un certificado autofirmado (versión preliminar)

Aprenda a usar scripts de implementación en plantillas de Azure Resource Manager. Los scripts de implementación se pueden usar para realizar pasos personalizados que no se pueden llevar a cabo con plantillas de Resource Manager. Por ejemplo, para crear un certificado autofirmado.  En este tutorial, creará una plantilla para implementar un almacén de claves de Azure, usará un recurso `Microsoft.Resources/deploymentScripts` en la misma plantilla para crear un certificado y agregará el certificado al almacén de claves. Para más información sobre el script de implementación, consulte cómo [usar los scripts de implementación en las plantillas de Azure Resource Manager](./deployment-script-template.md).

> [!IMPORTANT]
> En el mismo grupo de recursos que se usa para la ejecución de scripts y la solución de problemas, se crean dos recursos de script de implementación, una cuenta de almacenamiento y una instancia de contenedor. Estos recursos los suele eliminar el servicio del script cuando el script llega a un estado terminal de ejecución. Los recursos se le facturarán hasta que se eliminen. Para más información, consulte [Limpieza de los recursos del script de implementación](./deployment-script-template.md#clean-up-deployment-script-resources).

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Abra una plantilla de inicio rápido.
> * Edición de la plantilla
> * Implementación de la plantilla
> * Depuración del script con errores
> * Limpieza de recursos

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesitará lo siguiente:

* **[Visual Studio Code](https://code.visualstudio.com/) con la extensión Resource Manager Tools**. Consulte [Quickstart: Creación de plantillas de Azure Resource Manager con Visual Studio Code](./quickstart-create-templates-use-visual-studio-code.md).

* **Una identidad administrada asignada por el usuario con el rol de colaborador en el nivel de suscripción**. Esta identidad se usa para ejecutar scripts de implementación. Para crear una, consulte [Identidad administrada asignada por el usuario](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md). Necesitará el identificador de identidad al implementar la plantilla. El formato de la identidad es:

  ```json
  /subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<IdentityID>
  ```

  Use el script de la CLI siguiente para obtener el identificador; para ello, proporcione el nombre del grupo de recursos y el nombre de la identidad.

  ```azurecli-interactive
  echo "Enter the Resource Group name:" &&
  read resourceGroupName &&
  az identity list -g $resourceGroupName
  ```

## <a name="open-a-quickstart-template"></a>Abra una plantilla de inicio rápido.

En lugar de crear una plantilla desde cero, abra una plantilla en las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/). Plantillas de inicio rápido de Azure es un repositorio de plantillas de Azure Resource Manager.

La plantilla usada en este inicio rápido se llama [Create an Azure Key Vault and a secret](https://azure.microsoft.com/resources/templates/101-key-vault-create/) (Crear un almacén de claves de Azure y un secreto). La plantilla crea un almacén de claves y agrega un secreto a él.

1. En Visual Studio Code, seleccione **Archivo**>**Abrir archivo**.
2. En **Nombre de archivo**, pegue el código URL siguiente:

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-key-vault-create/azuredeploy.json
    ```

3. Seleccione **Abrir** para abrir el archivo.
4. Seleccione **Archivo**>**Guardar como** para guardar el archivo como **azuredeploy.json** en el equipo local.

## <a name="edit-the-template"></a>Edición de la plantilla

Realice los siguientes cambios en la plantilla:

### <a name="clean-up-the-template-optional"></a>Limpieza de la plantilla (opcional)

La plantilla original agrega un secreto al almacén de claves.  Para simplificar el tutorial, quite el recurso siguiente:

* **Microsoft.KeyVault/vaults/secrets**

Quite las dos definiciones de parámetros siguientes:

* **secretName**
* **secretValue**

Si decide no quitar estas definiciones, debe especificar los valores de los parámetros durante la implementación.

### <a name="configure-the-key-vault-access-policies"></a>Configuración de las directivas de acceso del almacén de claves

El script de implementación agrega un certificado al almacén de claves. Configure las directivas de acceso a almacén de claves para conceder el permiso a la identidad administrada:

1. Agregue un parámetro para obtener el identificador de identidad administrada:

    ```json
    "identityId": {
      "type": "string",
      "metadata": {
        "description": "Specifies the ID of the user-assigned managed identity."
      }
    },
    ```

    > [!NOTE]
    > La extensión de plantilla de Resource Manager de Visual Studio Code aún no puede dar formato a los scripts de implementación. No use [MAYÚS] + [ALT] + F para dar formato a los recursos de deploymentScripts, como el siguiente.

1. Agregue un parámetro para configurar las directivas de acceso al almacén de claves para que la identidad administrada pueda agregar certificados al almacén de claves.

    ```json
    "certificatesPermissions": {
      "type": "array",
      "defaultValue": [
        "get",
        "list",
        "update",
        "create"
      ],
      "metadata": {
      "description": "Specifies the permissions to certificates in the vault. Valid values are: all, get, list, update, create, import, delete, recover, backup, restore, manage contacts, manage certificate authorities, get certificate authorities, list certificate authorities, set certificate authorities, delete certificate authorities."
      }
    }
    ```

1. Actualice las directivas de acceso al almacén de claves existentes a:

    ```json
    "accessPolicies": [
      {
        "objectId": "[parameters('objectId')]",
        "tenantId": "[parameters('tenantId')]",
        "permissions": {
          "keys": "[parameters('keysPermissions')]",
          "secrets": "[parameters('secretsPermissions')]",
          "certificates": "[parameters('certificatesPermissions')]"
        }
      },
      {
        "objectId": "[reference(parameters('identityId'), '2018-11-30').principalId]",
        "tenantId": "[parameters('tenantId')]",
        "permissions": {
          "keys": "[parameters('keysPermissions')]",
          "secrets": "[parameters('secretsPermissions')]",
          "certificates": "[parameters('certificatesPermissions')]"
        }
      }
    ],
    ```

    Hay dos directivas definidas, una para el usuario con sesión iniciada y la otra para la identidad administrada.  El usuario con sesión iniciada solo necesita el permiso *list* para comprobar la implementación.  Para simplificar el tutorial, se asigna el mismo certificado a la identidad administrada y a los usuarios que han iniciado sesión.

### <a name="add-the-deployment-script"></a>Incorporación del script de implementación

1. Agregue los tres parámetros que usa el script de implementación.

    ```json
    "certificateName": {
      "type": "string",
      "defaultValue": "DeploymentScripts2019"
    },
    "subjectName": {
      "type": "string",
      "defaultValue": "CN=contoso.com"
    },
    "utcValue": {
      "type": "string",
      "defaultValue": "[utcNow()]"
    }
    ```

1. Agregue un recurso deploymentScripts:

    > [!NOTE]
    > Dado que los scripts de implementación en línea se incluyen entre comillas dobles, las cadenas dentro de los scripts de implementación deben incluirse entre comillas simples. El carácter de escape de PowerShell es **&#92;** .

    ```json
    {
      "type": "Microsoft.Resources/deploymentScripts",
      "apiVersion": "2019-10-01-preview",
      "name": "createAddCertificate",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.KeyVault/vaults', parameters('keyVaultName'))]"
      ],
      "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
          "[parameters('identityId')]": {
          }
        }
      },
      "kind": "AzurePowerShell",
      "properties": {
        "forceUpdateTag": "[parameters('utcValue')]",
        "azPowerShellVersion": "3.0",
        "timeout": "PT30M",
        "arguments": "[format(' -vaultName {0} -certificateName {1} -subjectName {2}', parameters('keyVaultName'), parameters('certificateName'), parameters('subjectName'))]", // can pass an argument string, double quotes must be escaped
        "scriptContent": "
          param(
            [string] [Parameter(Mandatory=$true)] $vaultName,
            [string] [Parameter(Mandatory=$true)] $certificateName,
            [string] [Parameter(Mandatory=$true)] $subjectName
          )

          $ErrorActionPreference = 'Stop'
          $DeploymentScriptOutputs = @{}

          $existingCert = Get-AzKeyVaultCertificate -VaultName $vaultName -Name $certificateName

          if ($existingCert -and $existingCert.Certificate.Subject -eq $subjectName) {

            Write-Host 'Certificate $certificateName in vault $vaultName is already present.'

            $DeploymentScriptOutputs['certThumbprint'] = $existingCert.Thumbprint
            $existingCert | Out-String
          }
          else {
            $policy = New-AzKeyVaultCertificatePolicy -SubjectName $subjectName -IssuerName Self -ValidityInMonths 12 -Verbose

            # private key is added as a secret that can be retrieved in the Resource Manager template
            Add-AzKeyVaultCertificate -VaultName $vaultName -Name $certificateName -CertificatePolicy $policy -Verbose

            $newCert = Get-AzKeyVaultCertificate -VaultName $vaultName -Name $certificateName

            # it takes a few seconds for KeyVault to finish
            $tries = 0
            do {
              Write-Host 'Waiting for certificate creation completion...'
              Start-Sleep -Seconds 10
              $operation = Get-AzKeyVaultCertificateOperation -VaultName $vaultName -Name $certificateName
              $tries++

              if ($operation.Status -eq 'failed')
              {
                throw 'Creating certificate $certificateName in vault $vaultName failed with error $($operation.ErrorMessage)'
              }

              if ($tries -gt 120)
              {
                throw 'Timed out waiting for creation of certificate $certificateName in vault $vaultName'
              }
            } while ($operation.Status -ne 'completed')

            $DeploymentScriptOutputs['certThumbprint'] = $newCert.Thumbprint
            $newCert | Out-String
          }
        ",
        "cleanupPreference": "OnSuccess",
        "retentionInterval": "P1D"
      }
    }
    ```

    El recurso `deploymentScripts` depende del recurso del almacén de claves y del recurso de asignación de roles.  Tiene estas propiedades:

    * **identity**: el script de implementación utiliza una identidad administrada asignada por el usuario para ejecutar los scripts.
    * **kind**: especifica el tipo de script. Actualmente solo es compatible el script de PowerShell.
    * **forceUpdateTag**: determina si el script de implementación debe ejecutarse aunque el origen del script no haya cambiado. Puede ser una marca de tiempo actual o un identificador único. Para más información, consulte la sección [Ejecución de un script varias veces](./deployment-script-template.md#run-script-more-than-once).
    * **azPowerShellVersion**: especifica la versión del módulo de Azure PowerShell que se va a usar. Actualmente, el script de implementación es compatible con las versiones 2.7.0; 2.8.0 y 3.0.0.
    * **timeout**: especifique el tiempo máximo de ejecución de scripts permitido en [formato ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). El valor predeterminado es **P1D**.
    * **arguments**: Especifique los valores de los parámetros. Los valores se separan con espacios.
    * **scriptContent**: especifica el contenido del script. Para ejecutar un script externo, use **primaryScriptURI** en su lugar. Para más información, consulte [Uso de scripts externos](./deployment-script-template.md#use-external-scripts).
        Solo es necesario declarar **$DeploymentScriptOutputs** al probar el script en una máquina local. La declaración de la variable permite que el script se ejecute en una máquina local y en un recurso deploymentScript sin tener que realizar cambios. El valor asignado a $DeploymentScriptOutputs está disponible como salidas en las implementaciones. Para más información, consulte [Trabajo con salidas de los scripts de implementación de PowerShell](./deployment-script-template.md#work-with-outputs-from-powershell-script) o [Trabajo con salidas de los scripts de implementación de la CLI](./deployment-script-template.md#work-with-outputs-from-cli-script).
    * **cleanupPreference**: especifica la preferencia sobre cuándo se deben eliminar los recursos del script de implementación.  El valor predeterminado es **Always**, lo que significa que los recursos del script de implementación se eliminan independientemente del estado terminal (Succeeded, Failed, Canceled). En este tutorial se usa **OnSuccess** para que pueda ver los resultados de la ejecución del script.
    * **retentionInterval**: especifica el tiempo durante el cual el servicio guarda los recursos del script una vez alcanzado el estado terminal. Los recursos se eliminarán cuando expire este periodo. La duración se basa en la norma ISO 8601. En este tutorial se usa P1D, es decir, un día.  Esta propiedad se usa cuando **cleanupPreference** se establece en **OnExpiration**. Esta propiedad no está habilitada actualmente.

    El script de implementación toma tres parámetros: el nombre del almacén de claves, el nombre del certificado y el nombre de la persona.  Se crea un certificado y se agrega el certificado al almacén de claves.

    **$DeploymentScriptOutputs** se utiliza para almacenar el valor de salida.  Para más información, consulte [Trabajo con salidas de los scripts de implementación de PowerShell](./deployment-script-template.md#work-with-outputs-from-powershell-script) o [Trabajo con salidas de los scripts de implementación de la CLI](./deployment-script-template.md#work-with-outputs-from-cli-script).

    Puede encontrar la plantilla rellena [aquí](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-keyvault.json).

1. Para ver el proceso de depuración, agregue la siguiente línea al script de implementación para incluir un error en el código:

    ```powershell
    Write-Output1 $keyVaultName
    ```

    El comando correcto es **Write-Output**, no **Write-Output1**.

1. Seleccione **File**>**Guardar** para guardar los cambios.

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Inicio de sesión en [Azure Cloud Shell](https://shell.azure.com)

1. Elija el entorno que prefiera; para ello, seleccione **PowerShell** o **Bash** (para CLI) en la esquina superior izquierda.  Es necesario reiniciar el shell cuando realiza el cambio.

    ![Archivo de carga de Cloud Shell de Azure Portal](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Seleccione **Cargar/descargar archivos** y, después, seleccione **Cargar**. Consulte la captura de pantalla anterior.  Seleccione el archivo que guardó en la sección anterior. Después de cargar el archivo, puede usar el comando **ls** y el comando **cat** para comprobar que la operación de carga se ha realizado correctamente.

1. Ejecute el siguiente script de PowerShell para implementar la plantilla.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $upn = Read-Host -Prompt "Enter your email address used to sign in to Azure"
    $identityId = Read-Host -Prompt "Enter the user-assigned managed identity ID"

    $adUserId = (Get-AzADUser -UserPrincipalName $upn).Id
    $resourceGroupName = "${projectName}rg"
    $keyVaultName = "${projectName}kv"

    New-AzResourceGroup -Name $resourceGroupName -Location $location

    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateFile "$HOME/azuredeploy.json" -identityId $identityId -keyVaultName $keyVaultName -objectId $adUserId

    Write-Host "Press [ENTER] to continue ..."
    ```

    El servicio de script de implementación debe crear recursos de script de implementación adicionales para la ejecución de scripts. La preparación y el proceso de limpieza pueden tardar hasta un minuto en completarse además del tiempo de ejecución real del script.

    Se ha producido un error en la implementación debido al uso en el script del comando **Write-Output1** no válido. Recibirá un error que indica:

    ```error
    The term 'Write-Output1' is not recognized as the name of a cmdlet, function, script file, or operable
    program.\nCheck the spelling of the name, or if a path was included, verify that the path is correct and try again.\n
    ```

    El resultado de la ejecución del script de implementación se almacena en los recursos del script de implementación para la solución de problemas.

## <a name="debug-the-failed-script"></a>Depuración del script con errores

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Abra el grupo de recursos. Es el nombre del proyecto con **rg** anexado. Verá dos recursos adicionales en el grupo de recursos. Estos recursos se denominan *recursos del script de implementación*.

    ![Recursos del script de implementación de la plantilla de Resource Manager](./media/template-tutorial-deployment-script/resource-manager-template-deployment-script-resources.png)

    Ambos archivos tienen el sufijo **azscripts**. Uno es una cuenta de almacenamiento y el otro es una instancia de contenedor.

    Seleccione **Mostrar tipos ocultos** para que aparezca el recurso deploymentScripts.

1. Seleccione la cuenta de almacenamiento con el sufijo **azscripts**.
1. Seleccione el icono **Recursos compartidos de archivos**. Verá una carpeta **azscripts**.  Esta contiene los archivos de ejecución del script de implementación.
1. Seleccione **azscripts**. Verá dos carpetas: **azscriptinput** y **azscriptoutput**.  La carpeta de entrada contiene un archivo de script de PowerShell del sistema y los archivos de script de implementación del usuario. La carpeta de salida contiene un archivo **executionresult.json** y el archivo de salida del script. Puede ver el mensaje de error en **executionresult.json**. El archivo de salida no está allí porque se produjo un error en la ejecución.

Quite la línea **Write-Output1** y vuelva a implementar la plantilla.

Cuando la segunda implementación se ejecute correctamente, el servicio de script quitará los recursos del script de implementación, ya que la propiedad **cleanupPreference** está establecida en **OnSuccess**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando los recursos de Azure ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.  Verá un total de seis recursos en el grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a usar un script de implementación en plantillas de Resource Manager. Para aprender a implementar recursos de Azure según condiciones, consulte:

> [!div class="nextstepaction"]
> [Condiciones de uso](./template-tutorial-use-conditions.md)
