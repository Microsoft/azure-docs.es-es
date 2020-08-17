---
title: Actualización de los modelos de Azure Analysis Services con Azure Automation | Microsoft Docs
description: En este artículo se explica cómo codificar actualizaciones de modelos de Azure Analysis Services mediante Azure Automation.
author: chrislound
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 05/07/2020
ms.author: chlound
ms.openlocfilehash: 31dc1973af42a1785a2a65cb1887f479e44af162
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87553910"
---
# <a name="refresh-with-azure-automation"></a>Actualización con Azure Automation

Mediante el uso de Azure Automation y los runbooks de PowerShell, puede realizar operaciones de actualización de datos automatizadas en los modelos tabulares de análisis de Azure.  

El ejemplo de este artículo usa el [módulo SqlServer de PowerShell](https://docs.microsoft.com/powershell/module/sqlserver/?view=sqlserver-ps). Más adelante en este artículo, se proporciona un ejemplo de un runbook de PowerShell que muestra cómo actualizar un modelo.  

## <a name="authentication"></a>Authentication

Todas las llamadas se deben autenticar con un token válido de Azure Active Directory (OAuth 2).  En el ejemplo de este artículo se usa una entidad de servicio (SPN) para la autenticación en Azure Analysis Services. Para más información, consulte [Creación de una aplicación mediante Azure Portal](../active-directory/develop/howto-create-service-principal-portal.md).

## <a name="prerequisites"></a>Requisitos previos

> [!IMPORTANT]
> En los ejemplos siguientes, se supone que el firewall de Azure Analysis Services está deshabilitado. Si el firewall está habilitado, la dirección IP pública del iniciador de la solicitud debe estar incluida en una regla de firewall.

### <a name="install-sqlserver-modules-from-powershell-gallery"></a>Instalación de módulos SqlServer desde la galería de PowerShell

1. En su cuenta de Azure Automation, haga clic en **Módulos** y, luego, en **Examinar galería**.

2. En la barra de búsqueda, busque **SqlServer**.

    ![Buscar módulos](./media/analysis-services-refresh-azure-automation/1.png)

3. Seleccione SqlServer y, a continuación, haga clic en **Importar**.
 
    ![Importar el módulo](./media/analysis-services-refresh-azure-automation/2.png)

4. Haga clic en **OK**.
 
### <a name="create-a-service-principal-spn"></a>Creación de una entidad de servicio (SPN)

Para obtener información sobre cómo crear una entidad de servicio, consulte [Creación de una entidad de servicio mediante Azure Portal](../active-directory/develop/howto-create-service-principal-portal.md).

### <a name="configure-permissions-in-azure-analysis-services"></a>Configuración de permisos en Azure Analysis Services
 
La entidad de servicio que cree debe tener permisos de administrador de servidor en el servidor. Para más información, vea [Incorporación de una entidad de servicio al rol de administrador del servidor](analysis-services-addservprinc-admins.md).

## <a name="design-the-azure-automation-runbook"></a>Diseño del runbook de Azure Automation

1. En la cuenta de Automation, cree un recurso de **credenciales** que se usará para almacenar de forma segura la entidad de servicio.

    ![Crear una credencial](./media/analysis-services-refresh-azure-automation/6.png)

2. Escriba los detalles de la credencial. En **Nombre de usuario**, escriba el identificador de aplicación de la entidad de servicio (appid) y, a continuación, en **Contraseña**, escriba el secreto de la entidad de servicio.

    ![Crear una credencial](./media/analysis-services-refresh-azure-automation/7.png)

3. Importe el runbook de Automation.

    ![Importación del runbook](./media/analysis-services-refresh-azure-automation/8.png)

4. Busque el archivo [Refresh-Model.ps1](#sample-powershell-runbook), proporcione un **Nombre** y una **Descripción**  y, luego, haga clic en **Crear** .

    > [!NOTE]
    > Utilice el script de la sección [Runbook de PowerShell de ejemplo](#sample-powershell-runbook) al final de este documento para crear un archivo llamado Refresh-Model.ps1 y guárdelo en la máquina local para importarlo en el runbook.

    ![Importación del runbook](./media/analysis-services-refresh-azure-automation/9.png)

5. Una vez se haya creado el runbook, pasará automáticamente al modo de edición.  Seleccione **Publicar**.

    ![Publicar runbook](./media/analysis-services-refresh-azure-automation/10.png)

    > [!NOTE]
    > El runbook recupera el recurso de credencial que se creó anteriormente mediante el comando **Get-AutomationPSCredential**.  A continuación, este comando se pasa al comando de PowerShell **Invoke-ProcessASADatabase** para realizar la autenticación en Azure Analysis Services.

6. Para probar el runbook, haga clic en **Iniciar**.

    ![Iniciar el runbook](./media/analysis-services-refresh-azure-automation/11.png)

7. Rellene los parámetros **DATABASENAME**, **ANALYSISSERVER** y **REFRESHTYPE** y, a continuación, haga clic en **Aceptar**. El parámetro **WEBHOOKDATA** no es necesario cuando se ejecuta manualmente el runbook.

    ![Iniciar el runbook](./media/analysis-services-refresh-azure-automation/12.png)

Si el runbook se ejecutó correctamente, recibirá una salida similar a la siguiente:

![Ejecución correcta](./media/analysis-services-refresh-azure-automation/13.png)

## <a name="use-a-self-contained-azure-automation-runbook"></a>Uso de un runbook de Azure Automation independiente

El runbook puede configurarse para desencadenar la actualización del modelo de Azure Analysis Services de forma programada.

Se puede configurar como se indica a continuación:

1. En el runbook de Automation, haga clic en **Programaciones** y, a continuación, en **Agregar una programación**.
 
    ![Crear una programación](./media/analysis-services-refresh-azure-automation/14.png)

2. Haga clic en **Programación** > **Crear una programación nueva** y, a continuación, rellene los detalles.

    ![Configurar la programación](./media/analysis-services-refresh-azure-automation/15.png)

3. Haga clic en **Crear**.

4. Rellene los parámetros para la programación. Se utilizarán cada vez que se desencadene el runbook. El parámetro **WEBHOOKDATA** debe dejarse en blanco cuando se ejecuta a través de una programación.

    ![Configuración de parámetros](./media/analysis-services-refresh-azure-automation/16.png)

5. Haga clic en **OK**.

## <a name="consume-with-data-factory"></a>Consumo con Data Factory

Para consumir el runbook mediante Azure Data Factory, primero cree un **webhook** para el runbook. El **webhook** proporcionará una dirección URL a la que se puede llamar a través de una actividad web de Azure Data Factory.

> [!IMPORTANT]
> Para crear un **webhook**, el estado del runbook debe ser **Publicado**.

1. En el runbook de Automation, haga clic en **Webhooks** y, a continuación, en **Agregar webhook**.

   ![Agregar webhook](./media/analysis-services-refresh-azure-automation/17.png)

2. Asigne un nombre y una fecha de expiración al webhook.  El nombre solo identifica el webhook dentro del runbook de Automation y no forma parte de la dirección URL.

   >[!CAUTION]
   >Asegúrese de copiar la dirección URL antes de cerrar el asistente, ya que no puede recuperarla una vez lo cierre.
    
   ![Configurar el webhook](./media/analysis-services-refresh-azure-automation/18.png)

    Los parámetros del webhook pueden permanecer en blanco.  Al configurar la actividad web de Azure Data Factory, los parámetros se pueden pasar al cuerpo de la llamada web.

3. En Data Factory, configure un **actividad web**

### <a name="example"></a>Ejemplo

   ![Ejemplo de una actividad web](./media/analysis-services-refresh-azure-automation/19.png)

La **URL** es la dirección URL creada a partir del webhook.

El **cuerpo** es un documento JSON que debe contener las siguientes propiedades:


|Propiedad  |Value  |
|---------|---------|
|**AnalysisServicesDatabase**     |Nombre de la base de datos de Azure Analysis Services <br/> Ejemplo: AdventureWorksDB         |
|**AnalysisServicesServer**     |Nombre del servidor de Azure Analysis Services. <br/> Por ejemplo: https:\//westus.asazure.windows.net/servers/myserver/models/AdventureWorks/.         |
|**DatabaseRefreshType**     |Tipo de actualización que se debe realizar. <br/> Ejemplo: Completo         |

Ejemplo del cuerpo de JSON:

```json
{
    "AnalysisServicesDatabaseName": "AdventureWorksDB",
    "AnalysisServicesServer": "asazure://westeurope.asazure.windows.net/MyAnalysisServer",
    "DatabaseRefreshType": "Full"
}
```

Estos parámetros se definen en el script de PowerShell del runbook.  Cuando se ejecuta la actividad web, la carga de JSON que se pasa es WEBHOOKDATA.

Esto se deserializa y se almacena como parámetros de PowerShell, que luego usa el comando Invoke-ProcesASDatabase PowerShell.

![Webhook deserializado](./media/analysis-services-refresh-azure-automation/20.png)

## <a name="use-a-hybrid-worker-with-azure-analysis-services"></a>Uso de Hybrid Worker con Azure Analysis Services

Una máquina virtual de Azure con una dirección IP pública estática se puede usar como una instancia de Azure Automation Hybrid Worker.  Luego, esta dirección IP pública se puede agregar al firewall de Azure Analysis Services.

> [!IMPORTANT]
> Asegúrese de que la dirección IP pública de la máquina virtual esté configurada como estática.
>
>Para más información sobre la configuración de Azure Automation Hybrid Worker, consulte [Instalación de Hybrid Runbook Worker](../automation/automation-hybrid-runbook-worker.md#hybrid-runbook-worker-installation).

Una vez que se configura una instancia de Hybrid Worker, cree un webhook tal y como se describe en la sección [Consumir con Data Factory](#consume-with-data-factory).  La única diferencia en este caso es que debe seleccionar la opción **Ejecutar en** > **Hybrid Worker** al configurar el webhook.

Ejemplo de un webhook con Hybrid Worker:

![Ejemplo de un webhook con Hybrid Worker](./media/analysis-services-refresh-azure-automation/21.png)

## <a name="sample-powershell-runbook"></a>Ejemplo del runbook de PowerShell

El fragmento de código siguiente es un ejemplo de cómo realizar la actualización del modelo de Azure Analysis Services mediante un runbook de PowerShell.

```powershell
param
(
    [Parameter (Mandatory = $false)]
    [object] $WebhookData,

    [Parameter (Mandatory = $false)]
    [String] $DatabaseName,
    [Parameter (Mandatory = $false)]
    [String] $AnalysisServer,
    [Parameter (Mandatory = $false)]
    [String] $RefreshType
)

$_Credential = Get-AutomationPSCredential -Name "ServicePrincipal"

# If runbook was called from Webhook, WebhookData will not be null.
if ($WebhookData)
{ 
    # Retrieve AAS details from Webhook request body
    $atmParameters = (ConvertFrom-Json -InputObject $WebhookData.RequestBody)
    Write-Output "CredentialName: $($atmParameters.CredentialName)"
    Write-Output "AnalysisServicesDatabaseName: $($atmParameters.AnalysisServicesDatabaseName)"
    Write-Output "AnalysisServicesServer: $($atmParameters.AnalysisServicesServer)"
    Write-Output "DatabaseRefreshType: $($atmParameters.DatabaseRefreshType)"
    
    $_databaseName = $atmParameters.AnalysisServicesDatabaseName
    $_analysisServer = $atmParameters.AnalysisServicesServer
    $_refreshType = $atmParameters.DatabaseRefreshType
 
    Invoke-ProcessASDatabase -DatabaseName $_databaseName -RefreshType $_refreshType -Server $_analysisServer -ServicePrincipal -Credential $_credential
}
else 
{
    Invoke-ProcessASDatabase -DatabaseName $DatabaseName -RefreshType $RefreshType -Server $AnalysisServer -ServicePrincipal -Credential $_Credential
}
```


## <a name="next-steps"></a>Pasos siguientes

[Muestras](analysis-services-samples.md)  
[REST API](https://docs.microsoft.com/rest/api/analysisservices/servers)
