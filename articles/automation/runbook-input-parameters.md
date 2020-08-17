---
title: Configuración de los parámetros de entrada de un runbook en Azure Automation
description: En este artículo se indica cómo configurar los parámetros de entrada de un runbook, que permiten pasar los datos a un runbook cuando se inicia.
services: automation
ms.subservice: process-automation
ms.date: 02/14/2019
ms.topic: conceptual
ms.openlocfilehash: 84e2eaf71326f59102800428479768aeba9ef9ab
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87042153"
---
# <a name="configure-runbook-input-parameters"></a>Configuración de los parámetros de entrada de un runbook

Los parámetros de entrada del runbook aumentan la flexibilidad del runbook porque permiten pasarle datos cuando se inicia. Estos parámetros permiten dirigir acciones del runbook a entornos y escenarios específicos. En este artículo se describe la configuración y el uso de los parámetros de entrada en los runbooks.

Los parámetros de entrada se pueden configurar para PowerShell, Flujo de trabajo de PowerShell, gráficos y runbooks de Python. Un Runbook puede tener varios parámetros con tipos de datos diferentes o sin parámetros. Los parámetros de entrada pueden ser obligatorios u opcionales y puede usar valores predeterminados para los parámetros opcionales.

La asignación de valores a los parámetros de entrada para un runbook se realiza al iniciarlo. Puede iniciar un runbook desde Azure Portal, un servicio web o PowerShell. También puede iniciarlo como Runbook secundario al que se llama insertado en otro Runbook.

### <a name="configure-input-parameters-in-powershell-runbooks"></a>Configuración de parámetros de entrada en runbooks de PowerShell

Los runbooks de PowerShell y de Flujo de trabajo de PowerShell en Azure Automation admiten parámetros de entrada que se definen mediante las siguientes propiedades. 

| **Propiedad** | **Descripción** |
|:--- |:--- |
| Tipo |Necesario. Tipo de datos previsto para el valor del parámetro. Es válido cualquier tipo .NET. |
| Nombre |Necesario. El nombre del parámetro. Este nombre debe ser único dentro del runbook, debe comenzar por una letra y puede contener solo letras, números o caracteres de subrayado. |
| Mandatory |Opcional. Valor booleano que especifica si el parámetro requiere un valor. Si se establece en True, es necesario proporcionar un valor cuando se inicia el runbook. Si se establece en False, el valor es opcional. Si no especifica un valor para la propiedad `Mandatory`, PowerShell considera el parámetro de entrada opcional de forma predeterminada. |
| Valor predeterminado |Opcional. Valor que se usa para el parámetro si no se pasa un valor de entrada al iniciar el runbook. El runbook puede establecer un valor predeterminado para cualquier parámetro. |

Windows PowerShell admite más atributos de parámetros de entrada de los enumerados anteriormente, como validación, alias y conjuntos de parámetros. Sin embargo, Azure Automation actualmente solo admite las propiedades de parámetros de entrada enumeradas.

Por ejemplo, echemos un vistazo a una definición de parámetro en un runbook de Flujo de trabajo de PowerShell. Esta definición tiene el siguiente formato general, donde los distintos parámetros están separados por comas.

```powershell
Param
(
  [Parameter (Mandatory= $true/$false)]
  [Type] $Name1 = <Default value>,

  [Parameter (Mandatory= $true/$false)]
  [Type] $Name2 = <Default value>
)
```

Ahora vamos a configurar los parámetros de entrada para un runbook de Flujo de trabajo de PowerShell que proporciona detalles sobre las máquinas virtuales, una o varias, de un grupo de recursos. Este runbook tiene dos parámetros, como se muestra en la siguiente captura de pantalla: el nombre de la máquina virtual (`VMName`) y el nombre del grupo de recursos (`resourceGroupName`).

![Automation, flujo de trabajo de PowerShell](media/automation-runbook-input-parameters/automation-01-powershellworkflow.png)

En esta definición de parámetros, los parámetros de entrada son parámetros simples de tipo cadena.

Tenga en cuenta que los runbooks de PowerShell y de Flujo de trabajo de PowerShell admiten todos los tipos simples y complejos, como `Object` o `PSCredential`, como parámetros de entrada. Si el runbook tiene un parámetro de entrada de tipo object, debe usar una tabla hash de PowerShell con pares nombre-valor para pasar un valor. Supongamos que tiene el siguiente parámetro en un runbook.

```powershell
[Parameter (Mandatory = $true)]
[object] $FullName
```

En este caso, puede pasar el siguiente valor al parámetro.

```powershell
@{"FirstName"="Joe";"MiddleName"="Bob";"LastName"="Smith"}
```

> [!NOTE]
> Cuando no se pasa un valor a un parámetro de cadena opcional con un valor predeterminado NULL, el valor del parámetro es una cadena vacía en lugar de NULL.

### <a name="configure-input-parameters-in-graphical-runbooks"></a>Configuración de parámetros de entrada en Runbooks gráficos

Para ilustrar la configuración de los parámetros de entrada para un runbook gráfico, vamos a crear un runbook que proporciona detalles sobre las máquinas virtuales, una o varias, de un grupo de recursos. Para más información, consulte [Mi primer runbook gráfico](./learn/automation-tutorial-runbook-graphical.md).

Un runbook gráfico usa estas actividades de runbooks principales:

* Configuración de la cuenta de ejecución de Azure para autenticarse en Azure. 
* Definición del cmdlet [Get-AzVM](/powershell/module/az.compute/get-azvm?view=azps-3.5.0) para obtener las propiedades de la máquina virtual.
* Uso de la actividad [Write-Output](/powershell/module/microsoft.powershell.utility/write-output) para escribir en la salida los nombres de máquina virtual. 

La actividad `Get-AzVM` define dos entradas, el nombre de la VM y el nombre del grupo de recursos. Puesto que estos nombres pueden ser diferentes cada vez que se inicia el runbook, debe agregar parámetros de entrada al runbook para aceptar estas entradas. Consulte [Creación gráfica en Azure Automation](automation-graphical-authoring-intro.md).

Siga estos pasos para configurar los parámetros de entrada.

1. Seleccione el runbook gráfico de la página Runbooks y, a continuación, haga clic en **Editar**.
2. En el editor gráfico, haga clic en el botón **Entrada y salida** y, a continuación, en **Agregar entrada** para abrir el panel Parámetro de entrada de runbook.

   ![Runbook gráfico de Automation](media/automation-runbook-input-parameters/automation-02-graphical-runbok-editor.png)

3. El control Entrada y salida muestra una lista de parámetros de entrada definidos para el runbook. Puede agregar un nuevo parámetro de entrada o editar la configuración de uno existente. Para agregar un nuevo parámetro al runbook, haga clic en **Agregar entrada** para abrir la hoja **Parámetro de entrada de runbook**, donde puede configurar parámetros mediante las propiedades definidas en [Creación gráfica en Azure Automation](automation-graphical-authoring-intro.md).

    ![Agregar nueva entrada](media/automation-runbook-input-parameters/automation-runbook-input-parameter-new.png)
4. Cree dos parámetros con las siguientes propiedades que usará la actividad `Get-AzVM` y, a continuación, haga clic en **Aceptar**.

   * Parámetro 1:
        * **Nombre** -- **VMName**
        * **Tipo**: cadena
        * **Obligatorio** -- **No**

   * Parámetro 2:
        * **Nombre** -- **resourceGroupName**
        * **Tipo**: cadena
        * **Obligatorio** -- **No**
        * **Valor predeterminado** -- **Custom**
        * Valor predeterminado personalizado: nombre del grupo de recursos que contiene las máquinas virtuales

5. Puede ver los parámetros en el control Entrada y salida. 
6. Haga clic de nuevo en **Aceptar** y, a continuación, haga clic en **Guardar**.
7. Haga clic en **Publicar** para publicar el runbook.

### <a name="configure-input-parameters-in-python-runbooks"></a>Configuración de parámetros de entrada en runbooks de Python

A diferencia de los runbooks gráficos, de PowerShell y de Flujo de trabajo de PowerShell, los runbooks de Python no toman parámetros con nombre. El editor de runbooks analiza todos los parámetros de entrada como una matriz de valores de argumento. Para obtener acceso a la matriz, puede importar el módulo `sys` al script de Python y usar la matriz `sys.argv`. Es importante tener en cuenta que el primer elemento de la matriz, `sys.argv[0]`, es el nombre del script. Por lo tanto, el primer parámetro de entrada real es `sys.argv[1]`.

Para obtener un ejemplo de cómo usar los parámetros de entrada en un runbook de Python, vea [My first Python runbook in Azure Automation](./learn/automation-tutorial-runbook-textual-python2.md) (Mi primer runbook de Python en Azure Automation).

## <a name="assign-values-to-input-parameters-in-runbooks"></a>Asignación de valores a parámetros de entrada en Runbooks

En esta sección se describen varias maneras de pasar valores a los parámetros de entrada en los runbooks. Puede asignar valores de parámetro en el momento de:

* [Iniciar un runbook](#start-a-runbook-and-assign-parameters)
* [Probar un runbook](#test-a-runbook-and-assign-parameters)
* [Vinculación de una programación para el runbook](#link-a-schedule-to-a-runbook-and-assign-parameters)
* [Creación de un webhook para el runbook](#create-a-webhook-for-a-runbook-and-assign-parameters)

### <a name="start-a-runbook-and-assign-parameters"></a>Inicio de un Runbook y asignación de parámetros

Se puede iniciar un runbook de muchas maneras: mediante Azure Portal, con un webhook, con cmdlets de PowerShell, con la API REST o con el SDK. 

#### <a name="start-a-published-runbook-using-the-azure-portal-and-assign-parameters"></a>Inicio de un runbook publicado mediante Azure Portal y asignación de parámetros

Al [iniciar el runbook](start-runbooks.md#start-a-runbook-with-the-azure-portal) en Azure Portal, se abre la hoja **Iniciar runbook** y puede escribir los valores de los parámetros que ha creado.

![Empezar a usar el portal](media/automation-runbook-input-parameters/automation-04-startrunbookusingportal.png)

En la etiqueta situada debajo del cuadro de entrada, puede ver las propiedades que se han establecido para definir los atributos del parámetro; por ejemplo, obligatorio u opcional, tipo o valor predeterminado. En el globo de ayuda junto al nombre del parámetro también se define la información importante que necesita para tomar decisiones acerca de los valores de entrada de los parámetros. 

> [!NOTE]
> Los parámetros de cadena admiten valores vacíos de tipo cadena. Si escribe `[EmptyString]` en el cuadro del parámetro de entrada, pasará una cadena vacía al parámetro. Además, los parámetros de cadena no admiten valores NULL. Si no pasa ningún valor a un parámetro de cadena, PowerShell lo interpreta como NULL.

#### <a name="start-a-published-runbook-using-powershell-cmdlets-and-assign-parameters"></a>Inicio de un runbook publicado mediante cmdlets de PowerShell y asignación de parámetros

* **Cmdlets de Azure Resource Manager**: puede iniciar un runbook de Automation creado en un grupo de recursos mediante [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook?view=azps-3.5.0).

   ```powershell
     $params = @{"VMName"="WSVMClassic";"resourceGroupeName"="WSVMClassicSG"}
  
     Start-AzAutomationRunbook -AutomationAccountName "TestAutomation" -Name "Get-AzureVMGraphical" –ResourceGroupName $resourceGroupName -Parameters $params
   ```

* **Cmdlets del modelo de implementación clásica de Azure:** puede iniciar un runbook de automatización creado en un grupo de recursos predeterminado mediante [Start-AzureRmAutomationRunbook](/powershell/module/servicemanagement/azure.service/start-azureautomationrunbook).
  
   ```powershell
     $params = @{"VMName"="WSVMClassic"; "ServiceName"="WSVMClassicSG"}
  
     Start-AzureAutomationRunbook -AutomationAccountName "TestAutomation" -Name "Get-AzureVMGraphical" -Parameters $params
   ```

> [!NOTE]
> Cuando se inicia un runbook mediante cmdlets de PowerShell, se crea un parámetro predeterminado, `MicrosoftApplicationManagementStartedBy`, con el valor `PowerShell`. Puede ver este parámetro en el panel Detalles del trabajo.  

#### <a name="start-a-runbook-using-an-sdk-and-assign-parameters"></a>Inicio de un runbook con un SDK y asignación de parámetros

* **Método de Azure Resource Manager:** puede iniciar un runbook mediante el SDK de un lenguaje de programación. A continuación, se muestra un fragmento de código C# para iniciar un Runbook en su cuenta de Automation. Puede ver todo el código en nuestro [repositorio de GitHub](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/automation/Microsoft.Azure.Management.Automation/tests/TestSupport/AutomationTestBase.cs).

   ```csharp
   public Job StartRunbook(string runbookName, IDictionary<string, string> parameters = null)
      {
        var response = AutomationClient.Jobs.Create(resourceGroupName, automationAccount, new JobCreateParameters
         {
            Properties = new JobCreateProperties
             {
                Runbook = new RunbookAssociationProperty
                 {
                   Name = runbookName
                 },
                   Parameters = parameters
             }
         });
      return response.Job;
      }
   ```

* **Método del modelo de implementación clásica de Azure:** puede iniciar un runbook mediante el SDK de un lenguaje de programación. A continuación, se muestra un fragmento de código C# para iniciar un Runbook en su cuenta de Automation. Puede ver todo el código en nuestro [repositorio de GitHub](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/automation/Microsoft.Azure.Management.Automation/tests/TestSupport/AutomationTestBase.cs).

   ```csharp
  public Job StartRunbook(string runbookName, IDictionary<string, string> parameters = null)
    {
      var response = AutomationClient.Jobs.Create(automationAccount, new JobCreateParameters
    {
      Properties = new JobCreateProperties
         {
           Runbook = new RunbookAssociationProperty
         {
           Name = runbookName
              },
                Parameters = parameters
              }
       });
      return response.Job;
    }
   ```

   Para iniciar este método, cree un diccionario para almacenar los parámetros del runbook, `VMName` y `resourceGroupName`, así como sus valores. A continuación, inicie el Runbook. A continuación, se muestra el fragmento de código C# para llamar al método definido más arriba.

   ```csharp
   IDictionary<string, string> RunbookParameters = new Dictionary<string, string>();
  
   // Add parameters to the dictionary.
  RunbookParameters.Add("VMName", "WSVMClassic");
   RunbookParameters.Add("resourceGroupName", "WSSC1");
  
   //Call the StartRunbook method with parameters
   StartRunbook("Get-AzureVMGraphical", RunbookParameters);
   ```

#### <a name="start-a-runbook-using-the-rest-api-and-assign-parameters"></a>Inicio de un runbook con la API REST y asignación de parámetros

Puede crear e iniciar un trabajo de runbook con la API de REST de Azure Automation mediante el método `PUT` con el siguiente URI de solicitud: `https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Automation/automationAccounts/{automationAccountName}/jobs/{jobName}?api-version=2017-05-15-preview`

En el identificador URI de solicitud, reemplace los siguientes parámetros:

* `subscriptionId`: el identificador de la suscripción a Azure.  
* `resourceGroupName`: nombre del grupo de recursos de la cuenta de Automation.
* `automationAccountName`: nombre de la cuenta de Automation hospedada en el servicio en la nube especificado.  
* `jobName`: GUID para el id. de trabajo. Los GUID de PowerShell se pueden crear mediante `[GUID]::NewGuid().ToString()*`.

Para pasar parámetros al trabajo de runbook, use el cuerpo de la solicitud. Admite la siguiente información, proporcionada en formato JSON:

* Nombre del runbook: Necesario. El nombre del Runbook para que se inicie el trabajo.  
* Parámetros del runbook: Opcional. Diccionario de la lista de parámetros en formato (nombre, valor), donde el nombre es de tipo cadena y el valor puede ser cualquier valor JSON válido.

Si desea iniciar el runbook **Get-AzureVMTextual** creado antes con `VMName` y `resourceGroupName` como parámetros, use el siguiente formato JSON para el cuerpo de la solicitud.

```json
    {
      "properties":{
        "runbook":{
        "name":"Get-AzureVMTextual"},
      "parameters":{
         "VMName":"WindowsVM",
         "resourceGroupName":"ContosoSales"}
        }
    }
```

Si el trabajo se crea correctamente, se devuelve un código de estado HTTP 201. Para más información sobre los encabezados y el cuerpo de la respuesta, consulte [Creación de un trabajo de runbook mediante la API REST](/rest/api/automation/job/create).

### <a name="test-a-runbook-and-assign-parameters"></a>Prueba de un Runbook y asignación de parámetros

Al [probar la versión de borrador del runbook](./manage-runbooks.md) mediante la opción de prueba, se abre la página Prueba. Use esta página para configurar los valores de los parámetros que ha creado.

![Prueba y asignación de parámetros](media/automation-runbook-input-parameters/automation-06-testandassignparameters.png)

### <a name="link-a-schedule-to-a-runbook-and-assign-parameters"></a>Vinculación de una programación a un Runbook y asignación de parámetros

También puede [vincular una programación](./shared-resources/schedules.md) a su Runbook para que este se inicie en un momento determinado. Asigne los parámetros de entrada al crear la programación y el runbook los usará cuando se inicie mediante la programación. No se puede guardar la programación hasta que se proporcionan todos los valores de los parámetros obligatorios.

![Programación y asignación de parámetros](media/automation-runbook-input-parameters/automation-07-scheduleandassignparameters.png)

### <a name="create-a-webhook-for-a-runbook-and-assign-parameters"></a>Creación de un Webhook para un Runbook y asignación de parámetros

Puede crear un [Webhook](automation-webhooks.md) para el Runbook y configurar los parámetros de entrada del Runbook. No se puede guardar el webhook hasta que se proporcionen todos los valores de los parámetros obligatorios.

![Creación de un Webhook y asignación de parámetros](media/automation-runbook-input-parameters/automation-08-createwebhookandassignparameters.png)

Cuando se ejecuta un runbook mediante un webhook, se envía un parámetro de entrada predefinido, `[WebhookData](automation-webhooks.md)`, junto con los parámetros de entrada que defina. 

![Parámetro WebhookData](media/automation-runbook-input-parameters/automation-09-webhook-data-parameters.png)

## <a name="pass-a-json-object-to-a-runbook"></a>Paso de un objeto JSON a un runbook

Puede ser útil almacenar los datos que desea pasar a un runbook en un archivo JSON. Por ejemplo, podría crear un archivo JSON que contiene todos los parámetros que desea pasar a un runbook. Para hacerlo, debe convertir el código JSON en una cadena y luego convertir esta cadena en un objeto de PowerShell antes de pasárselo al runbook.

Esta sección utiliza un ejemplo en el que un script de PowerShell llama a [Start-AzAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook?view=azps-3.5.0) para iniciar un runbook de PowerShell, pasando el contenido del archivo JSON al runbook. El runbook de PowerShell inicia una máquina virtual de Azure y recupera los parámetros de la máquina virtual desde el objeto JSON.

### <a name="create-the-json-file"></a>Creación del archivo JSON

Escriba el código siguiente en un archivo de texto y guárdelo como **test.json** en algún lugar del equipo local.

```json
{
   "VmName" : "TestVM",
   "ResourceGroup" : "AzureAutomationTest"
}
```

### <a name="create-the-runbook"></a>Creación del runbook

Cree un nuevo runbook de PowerShell llamado **Test-Json** en Azure Automation. Consulte [Mi primer runbook de PowerShell](./learn/automation-tutorial-runbook-textual-powershell.md).

Para aceptar los datos de JSON, el runbook debe tomar un objeto como parámetro de entrada. De ese modo, el runbook puede usar las propiedades definidas en el archivo JSON.

```powershell
Param(
     [parameter(Mandatory=$true)]
     [object]$json
)

# Connect to Azure account
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID `
    -ApplicationID $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

# Convert object to actual JSON
$json = $json | ConvertFrom-Json

# Use the values from the JSON object as the parameters for your command
Start-AzVM -Name $json.VMName -ResourceGroupName $json.ResourceGroup
```

Guarde y publique este runbook en su cuenta de Automation.

### <a name="call-the-runbook-from-powershell"></a>Llamada al runbook desde PowerShell

Ahora puede llamar al runbook desde la máquina local mediante Azure PowerShell. 

1. Inicie sesión en Azure como se muestra. Después, se le pedirá que escriba las credenciales de Azure.

   ```powershell
   Connect-AzAccount
   ```

    >[!NOTE]
    >En el caso de los runbooks de PowerShell, `Add-AzAccount` y `Add-AzureRMAccount` son alias de `Connect-AzAccount`. Tenga en cuenta que estos alias no están disponibles para los runbooks gráficos. Un runbook gráfico solo puede usar `Connect-AzAccount`.

1. Obtenga el contenido del archivo JSON guardado y conviértalo en una cadena. `JsonPath` indica la ruta de acceso donde se guardó el archivo JSON.

   ```powershell
   $json =  (Get-content -path 'JsonPath\test.json' -Raw) | Out-string
   ```

1. Convierta el contenido de la cadena `$json` en un objeto de PowerShell.

   ```powershell
   $JsonParams = @{"json"=$json}
   ```

1. Cree una tabla hash para los parámetros de `Start-AzAutomationRunbook`. 

   ```powershell
   $RBParams = @{
        AutomationAccountName = 'AATest'
        ResourceGroupName = 'RGTest'
        Name = 'Test-Json'
        Parameters = $JsonParams
   }
   ```

   Tenga en cuenta que está estableciendo el valor de `Parameters` en el objeto de PowerShell que contiene los valores del archivo JSON.
1. Inicie el runbook.

   ```powershell
   $job = Start-AzAutomationRunbook @RBParams
   ```

## <a name="next-steps"></a>Pasos siguientes

* Para preparar un runbook de texto, consulte [Edición de runbooks de texto en Azure Automation](automation-edit-textual-runbook.md).
* Para preparar un runbook gráfico, consulte [Creación de runbooks gráficos en Azure Automation](automation-graphical-authoring-intro.md).
