---
title: 'ExpressRoute: Configuración de alertas personalizadas para rutas anunciadas'
description: En este artículo se muestra cómo usar Azure Automation y Logic Apps para supervisar el número de rutas anunciadas desde la puerta de enlace de ExpressRoute a las redes locales con el fin de evitar alcanzar el límite de 200 rutas.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 05/29/2020
ms.author: duau
ms.openlocfilehash: f29f43234f1541abeb448e722d0b72ef7c0221c9
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401731"
---
# <a name="configure-custom-alerts-to-monitor-advertised-routes"></a>Configuración de alertas personalizadas para supervisar rutas anunciadas

Este artículo le ayuda a usar Azure Automation y Logic Apps para supervisar constantemente el número de rutas anunciadas desde la puerta de enlace de ExpressRoute a las redes locales. La supervisión puede evitar que se alcance el [límite de 200 rutas](expressroute-faqs.md#how-many-prefixes-can-be-advertised-from-a-vnet-to-on-premises-on-expressroute-private-peering).

**Azure Automation** permite automatizar la ejecución de un script de PowerShell personalizado almacenado en un *runbook*. Al usar la configuración de este artículo, el runbook contiene un script de PowerShell que realiza consultas en una o varias puertas de enlace de ExpressRoute. Recopila un conjunto de contenido que contiene el grupo de recursos, el nombre de la puerta de enlace de ExpressRoute y el número de prefijos de red anunciados localmente.

**Azure Logic Apps** programa un flujo de trabajo personalizado que llama al runbook de Azure Automation. La ejecución del runbook se realiza mediante un trabajo. Una vez ejecutada la recopilación de datos, el flujo de trabajo de Azure Logic Apps clasifica los datos y, en función de los criterios de coincidencia en el número de prefijos de red por encima o por debajo de un umbral predefinido, envía información a una dirección de correo electrónico de destino.

### <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

La configuración de una alerta personalizada se basa en tres pasos principales:

1. Creación de una cuenta de Automation con una cuenta y permisos de ejecución.

2. Creación y configuración de runbooks.

3. Creación de una aplicación lógica que activará la cuenta de Automation y enviará un mensaje de correo electrónico de alerta si el número supera un umbral (por ejemplo, 160).

## <a name="before-you-begin"></a><a name="before"></a>Antes de empezar

Antes de comenzar con la configuración, compruebe que se cumplen los criterios siguientes:

* Tiene al menos una puerta de enlace de ExpressRoute en su implementación.

* Posee conocimientos básicos de [cuentas de ejecución](../automation/manage-runas-account.md) en Azure Automation.

* Está familiarizado con [Azure Logic Apps](../logic-apps/logic-apps-overview.md).

* Está familiarizado con el uso de Azure PowerShell. Azure PowerShell es necesario para recopilar los prefijos de red en la puerta de enlace de ExpressRoute. Para más información sobre Azure PowerShell en general, consulte la [documentación de Azure PowerShell](https://docs.microsoft.com/powershell/azure/?view=azps-4.1.0).

### <a name="notes-and-limitations"></a><a name="limitations"></a>Notas y limitaciones

* La alerta personalizada que se describe en este artículo es un complemento para lograr un mejor funcionamiento y control. No es un reemplazo de las alertas nativas de ExpressRoute.
* La recopilación de datos de las puertas de enlace de ExpressRoute se ejecuta en segundo plano. El tiempo de ejecución puede ser más largo de lo esperado. Para evitar la puesta en cola de trabajos, la periodicidad del flujo de trabajo se debe configurar correctamente.
* Las implementaciones mediante scripts o plantillas de ARM pueden ser más rápidas que el desencadenador de alarma personalizado. Esto podría incrementar el número de prefijos de red en la puerta de enlace de ExpressRoute por encima del límite de 200 rutas.

## <a name="create-and-configure-accounts"></a><a name="accounts"></a>Creación y configuración de cuentas

Cuando crea una cuenta de Automation en Azure Portal, automáticamente se crea una cuenta [de ejecución](../automation/manage-runas-account.md#types-of-run-as-accounts). Esta cuenta realiza las siguientes acciones:

* Crea una aplicación de Azure Active Directory (Azure AD) con un certificado autofirmado. La propia cuenta de ejecución tiene un certificado que debe renovarse de forma predeterminada todos los años.

* Crea una cuenta de entidad de servicio para la aplicación en Azure AD.

* Se asigna a sí misma el rol Colaborador (RBAC) en la suscripción de Azure en uso. Este rol administra los recursos de Azure Resource Manager mediante runbooks.

Para crear una cuenta de Automation, necesita privilegios y permisos. Para más información, consulte [Permisos necesarios para crear una cuenta de Automation](../automation/automation-create-standalone-account.md#permissions-required-to-create-an-automation-account).

### <a name="1-create-an-automation-account"></a><a name="about"></a>1. Creación de una cuenta de automatización

Cree una cuenta de Automation con permisos de ejecución. Para obtener instrucciones, consulte [Creación de una cuenta de Azure Automation](../automation/automation-quickstart-create-account.md).

:::image type="content" source="./media/custom-route-alert-portal/create-account.png" alt-text="Agregar cuenta de Automation" lightbox="./media/custom-route-alert-portal/create-account-expand.png":::

### <a name="2-assign-the-run-as-account-a-role"></a><a name="about"></a>2. Asignación de un rol a la cuenta de ejecución

De forma predeterminada, se asigna el rol **Colaborador** a la entidad de servicio que utiliza la cuenta **de ejecución**. Puede mantener el rol predeterminado asignado a la entidad de servicio o puede restringir los permisos asignando un [rol integrado](../role-based-access-control/built-in-roles.md) (por ejemplo, Lector) o un [rol personalizado](../active-directory/users-groups-roles/roles-create-custom.md).

 Siga los pasos que se indican a continuación para determinar el rol que se asigna a la entidad de servicio que usa la cuenta de ejecución:

1. Vaya a su cuenta de Automation. Vaya a **Configuración de la cuenta**, seleccione **Cuentas de ejecución**.

2. Seleccione **Roles** para ver la definición de roles en uso.

   :::image type="content" source="./media/custom-route-alert-portal/run-as-account-permissions.png" alt-text="Asignación de un rol":::

## <a name="create-and-configure-runbooks"></a><a name="runbooks"></a>Creación y configuración de runbooks

### <a name="1-install-modules"></a><a name="install-modules"></a>1. Módulos de instalación

Para ejecutar los cmdlets de PowerShell en los runbooks de Azure Automation, debe instalar algunos módulos Az adicionales de Azure PowerShell. Utilice los siguientes pasos para instalar los módulos:

1. Abra su cuenta de Azure Automation y vaya a **Módulos**.

   :::image type="content" source="./media/custom-route-alert-portal/navigate-modules.png" alt-text="Ir a módulos":::

2. Busque en la galería e importe los siguientes módulos: **Az.Accounts**, **Az.Network**, **Az.Automation** y **Az.Profile**.

   :::image type="content" source="./media/custom-route-alert-portal/import-modules.png" alt-text="Buscar e importar módulos" lightbox="./media/custom-route-alert-portal/import-modules-expand.png":::
  
### <a name="2-create-a-runbook"></a><a name="create"></a>2. Crear un runbook

1. Para crear el runbook de PowerShell, vaya a la cuenta de Automation. En **Process Automation** (Automatización de procesos), seleccione el icono **Runbooks** y, después, seleccione **Crear un Runbook**.

   :::image type="content" source="./media/custom-route-alert-portal/create-runbook.png" alt-text="Crear un Runbook.":::

2. Seleccione **Crear** para crear el runbook.

   :::image type="content" source="./media/custom-route-alert-portal/create-runbook-2.png" alt-text="Seleccionar Crear.":::

3. Seleccione el runbook que acaba de crear y, a continuación, seleccione **Editar**.

   :::image type="content" source="./media/custom-route-alert-portal/edit-runbook.png" alt-text="Editar runbook":::

4. En **Editar**, pegue el script de PowerShell. El [script de ejemplo](#script) se puede modificar y usar para supervisar puertas de enlace de ExpressRoute en uno o varios grupos de recursos.

   En el script de ejemplo, observe los siguientes valores:

    * La matriz **$rgList** contiene la lista de grupos de recursos con puertas de enlace de ExpressRoute. Puede personalizar las puertas de enlace de ExpressRoute basadas en listas.
    * La variable **$thresholdNumRoutes** define el umbral del número de prefijos de red anunciados desde una puerta de enlace de ExpressRoute a las redes locales.

#### <a name="example-script"></a><a name="script"></a>Script de ejemplo

```powershell
  ################# Input parameters #################
# Resource Group Name where the ExR GWs resides in
$rgList= @('ASH-Cust10-02','ASH-Cust30')  
$thresholdNumRoutes = 160
###################################################

# Ensures you do not inherit an AzContext in your runbook
Disable-AzContextAutosave -Scope Process | Out-Null

Try {

   $conn = Get-AutomationConnection -Name 'AzureRunAsConnection'
   while(!($connectionResult) -And ($logonAttempt -le 5))
   {
        $LogonAttempt++
        # Logging in to Azure...
        $connectionResult =  Connect-AzAccount `
                               -ServicePrincipal `
                               -ApplicationId $conn.ApplicationId `
                               -Tenant $conn.TenantId `
                               -CertificateThumbprint $conn.CertificateThumbprint `
                               -Subscription $conn.SubscriptionId `
                               -Environment AzureCloud 
                               
        Start-Sleep -Seconds 10
    }
} Catch {
    if (!$conn)
    {
        $ErrorMessage = "Service principal not found."
        throw $ErrorMessage
    } 
    else
    {
        Write-Error -Message $_.Exception
        throw $_.Exception
    }
}

# Get the name of the Azure subscription
$subscriptionName=(Get-AzSubscription -SubscriptionId $conn.SubscriptionId).Name

#write-Output "<br>$(Get-Date) - selection of the Azure subscription: $subscriptionName" 
Select-AzSubscription -SubscriptionId $conn.SubscriptionId | Out-Null


$GtwList = @()
$results= @()

foreach ($rgName in $rgList)
{
## Collect all the ExpressRoute gateways in a Resource Group
$GtwList=Get-AzVirtualNetworkGateway -ResourceGroupName $rgName 

## For each ExpressRoute gateway, get the IP addresses of the BGP peers and collect the number of routes advertised 
foreach ($gw in $GtwList) {
  
  $peers = Get-AzVirtualNetworkGatewayBGPPeerStatus -VirtualNetworkGatewayName $gw.Name -ResourceGroupName $rgName


 if ($peers[0].State -eq 'Connected') {
   $routes1=$null
   $routes1 = Get-AzVirtualNetworkGatewayAdvertisedRoute -VirtualNetworkGatewayName $gw.Name -ResourceGroupName $rgName -Peer $peers[0].Neighbor
 }
  if ($peers[1].State -eq 'Connected') {
  
   $routes2=$null
   $routes2 = Get-AzVirtualNetworkGatewayAdvertisedRoute -VirtualNetworkGatewayName $gw.Name -ResourceGroupName $rgName -Peer $peers[1].Neighbor
 }
 
  $sampleTime=(Get-Date).ToString("dd-MM-yyyy HH:mm:ss")
  if ($routes1.Count -eq $routes2.Count)
  {
     
     if ($routes1.Count -lt $thresholdNumRoutes){
       $status='OK'
       $alertMsg='number of routes below threshold'
     } 
     else {
       $status='ALERT'
       $alertMsg='number of routes above threshold'
     }
  } 
  else
  {
     $status='WARNING'
     $alertMsg='check ER Gateway'
  }
  
  $obj = [psCustomObject]@{
            resourceGroup =$rgName
            nameGtw  = $gw.Name
            peer1 = $peers[0].Neighbor
            peer2 = $peers[1].Neighbor
            numRoutesPeer1=  $routes1.Count
            numRoutesPeer2=  $routes2.Count
            time=$sampleTime
            status=$status
            alertMessage = $alertMsg
        }
  $results += $obj
} ### end foreach gateways in each resource group
} ### end foreach resource group
$jsonResults= ConvertTo-Json $results -Depth 100
Write-Output  $jsonResults
 
 ```

### <a name="3-save-and-publish-the-runbook"></a><a name="publish"></a>3. Guardar y publicar el runbook

1. Seleccione **Guardar** para guardar una copia en borrador del runbook.
2. Seleccione **Publicar** para publicar el runbook como la versión oficial del runbook en la cuenta de Automation.

   :::image type="content" source="./media/custom-route-alert-portal/save-publish-runbook.png" alt-text="Guardar y publicar el runbook.":::

Cuando ejecuta el script de PowerShell, se recopila una lista de valores:
 
* Resource group

* Nombre de la puerta de enlace de ExpressRoute

* Dirección IP del primer par BGP (peer1)

* Dirección IP del segundo par BGP (peer2)

* Número de prefijos de red anunciados desde la puerta de enlace de ExpressRoute al primer par BGP (peer1)

* Número de prefijos de red anunciados desde la puerta de enlace de ExpressRoute al segundo par BGP (peer2)

* Timestamp

* Estado, que se puede clasificar como:

  * "OK" si el número de rutas es inferior a un valor de umbral
  * "ALERTA" si el número de rutas es superior a un valor de umbral
  * "ADVERTENCIA" si el número de prefijos de red anunciado para los dos pares BGP es diferente

* Mensaje de alerta, para una descripción detallada del estado (OK, ALERTA, ADVERTENCIA)

El script de PowerShell convierte la información recopilada en una salida JSON. El runbook usa el cmdlet de PowerShell [Write-Output](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Utility/Write-Output?) como flujo de salida para comunicar información al cliente.

### <a name="4-validate-the-runbook"></a><a name="validate"></a>4. Validación del runbook

Una vez creado el runbook, se debe validar. Seleccione **Iniciar** y compruebe las salidas y los errores de los diferentes flujos de trabajo.

:::image type="content" source="./media/custom-route-alert-portal/validate-runbook.png" alt-text="Validar el runbook" lightbox="./media/custom-route-alert-portal/validate-runbook-expand.png":::

## <a name="create-and-configure-a-logic-app"></a><a name="logic"></a>Creación y configuración de una aplicación lógica

Azure Logic Apps es el orquestador de todos los procesos de recopilación y acciones. En las secciones siguientes, se creará un flujo de trabajo mediante una aplicación lógica.

### <a name="workflow"></a>Flujo de trabajo

En este flujo de trabajo, se crea una aplicación lógica que supervisa con regularidad las puertas de enlace de ExpressRoute. Si existe algún elemento nuevo, la aplicación lógica envía un correo electrónico por cada elemento. Cuando haya terminado, la aplicación lógica se parecerá a este flujo de trabajo, en un alto nivel:

:::image type="content" source="./media/custom-route-alert-portal/logic-apps-workflow.png" alt-text="Flujo de trabajo de Logic Apps":::

### <a name="1-create-a-logic-app"></a>1. Creación de una aplicación lógica

En **Diseñador de aplicación lógica**, cree una aplicación lógica mediante la plantilla de **aplicación lógica en blanco**. Para conocer los pasos, consulte [Creación de aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md#create-your-logic-app).

:::image type="content" source="./media/custom-route-alert-portal/blank-template.png" alt-text="Plantilla vacía":::

### <a name="2-add-a-trigger"></a>2. Incorporación de un desencadenador

Cada una de las aplicaciones lógicas se inicia mediante un desencadenador. Un desencadenador se activa cuando sucede un evento específico o se cumple una condición determinada. Cada vez que el desencadenador se activa, el motor de Azure Logic Apps crea una instancia de aplicación lógica que inicia y ejecuta el flujo de trabajo.

Para ejecutar con regularidad una aplicación lógica que se basa en una programación de tiempo predefinida, agregue la **programación de periodicidad** integrada al flujo de trabajo. Escriba **programación** en el cuadro de búsqueda. Seleccione **Desencadenadores**. En la lista de desencadenadores, seleccione **Programación de periodicidad**.

:::image type="content" source="./media/custom-route-alert-portal/schedule.png" alt-text="Programación de periodicidad":::

En el desencadenador Programación de periodicidad, puede establecer la zona horaria y la periodicidad para la repetición de ese flujo de trabajo. Juntos, el intervalo y la frecuencia definen la programación para el desencadenador de la aplicación lógica. Para establecer una frecuencia de periodicidad mínima razonable, tenga en cuenta los siguientes factores:

* El script de PowerShell en el runbook de Automation tarda un tiempo en completarse. El tiempo de ejecución depende del número de puertas de enlace de ExpressRoute que se van a supervisar. Una frecuencia de periodicidad demasiado corta producirá la puesta en cola de los trabajos.

* El script de PowerShell se ejecuta como un trabajo en segundo plano. No se inicia inmediatamente, se ejecuta después de un retraso variable.

* Una frecuencia de periodicidad demasiado corta generará una carga innecesaria en las puertas de enlace de Azure ExpressRoute.

Al final de la configuración del flujo de trabajo, puede comprobar la coherencia de la frecuencia de periodicidad ejecutando el flujo de trabajo varias veces y, a continuación, comprobando el resultado en el **historial de ejecuciones**.

:::image type="content" source="./media/custom-route-alert-portal/recurrence.png" alt-text="Periodicidad" lightbox="./media/custom-route-alert-portal/recurrence-expand.png":::

### <a name="3-create-a-job"></a><a name="job"></a>3. Creación de un trabajo

Una aplicación lógica accede a otras aplicaciones, servicios y a la plataforma mediante conectores. El siguiente paso de este flujo de trabajo es seleccionar un conector para acceder a la cuenta de Azure Automation que se definió anteriormente.

1. En el **diseñador de Logic Apps**, debajo de **Periodicidad**, seleccione **Nuevo paso**. En **Elegir una acción** y en el cuadro de búsqueda, seleccione **Todas**.
2. En el cuadro de búsqueda, escriba **Azure Automation** y busque. Seleccione **Crear trabajo**. **Crear trabajo** se usará para activar el runbook de Automation que se creó anteriormente.

   :::image type="content" source="./media/custom-route-alert-portal/create-job.png" alt-text="Crear trabajo":::

3. Inicie sesión mediante una entidad de servicio. Puede usar una entidad de servicio existente o crear una. Para crear una entidad de servicio, consulte [Uso del portal para crear una entidad de servicio de Azure AD que pueda acceder a los recursos](../active-directory/develop/howto-create-service-principal-portal.md). Seleccione **Connect with Service Principal** (Conectarse a la entidad de servicio).

   :::image type="content" source="./media/custom-route-alert-portal/sign-in.png" alt-text="Sign in":::

4. Escriba un **nombre de conexión**, agregue el **identificador de cliente** (identificador de la aplicación), **secreto de cliente** y el **identificador de inquilino**. Seleccione **Crear**.

   :::image type="content" source="./media/custom-route-alert-portal/connect-service-principal.png" alt-text="Conectarse a la entidad de servicio":::

5. En la página **Crear trabajo**, la entidad de servicio debe tener el rol "Lector" en el **grupo de recursos** que hospeda la cuenta de Automation y "Operador de trabajos de Automation" en la **cuenta de Automation**. Además, compruebe que ha agregado el **nombre del runbook** como un nuevo parámetro.

   :::image type="content" source="./media/custom-route-alert-portal/roles.png" alt-text="Roles" lightbox="./media/custom-route-alert-portal/roles-expand.png":::

### <a name="4-get-the-job-output"></a><a name="output"></a>4. Obtención de la salida del trabajo

1. Seleccione **Nuevo paso**. Busque "Azure Automation". En la lista **Acciones**, seleccione **Obtener resultado del trabajo**.

   :::image type="content" source="./media/custom-route-alert-portal/get-output.png" alt-text="Obtener resultado del trabajo":::

2. En la página **Obtener resultado del trabajo**, especifique la información necesaria para acceder a la cuenta de Automation. Seleccione la **suscripción, grupo de recursos** y **cuenta de Automation** que desea usar. Haga clic en el cuadro **Id. del trabajo**. Cuando aparezca la lista de **contenido dinámico**, seleccione **Id. del trabajo**.

   :::image type="content" source="./media/custom-route-alert-portal/job-id.png" alt-text="Id. del trabajo" lightbox="./media/custom-route-alert-portal/job-id-expand.png":::

### <a name="5-parse-the-json"></a><a name="parse"></a>5. Análisis del archivo JSON

La información contenida en la salida de la "acción Crear trabajo de Azure Automation" (pasos anteriores) genera un objeto JSON. **Análisis de JSON** de Logic Apps es una acción integrada para crear tokens descriptivos a partir de las propiedades y valores del contenido JSON. Después, puede usar esas propiedades en el flujo de trabajo.

1. Agregue una acción. En la acción **Obtener resultado del trabajo**, seleccione **Nuevo paso**.
2. En el cuadro de búsqueda **Elegir una acción** escriba "analizar json" para buscar los conectores que ofrezcan esta acción. En la lista **Acciones**, seleccione la acción **Análisis del archivo JSON** para las operaciones de datos que desee usar.

   :::image type="content" source="./media/custom-route-alert-portal/parse-json.png" alt-text="Análisis del archivo JSON":::

3. Haga clic en el cuadro **Contenido**. Cuando aparezca la lista de contenido dinámico, seleccione **Contenido**.

   :::image type="content" source="./media/custom-route-alert-portal/content.png" alt-text="Contenido" lightbox="./media/custom-route-alert-portal/content-expand.png":::

4. El análisis de un archivo JSON requiere un esquema. El esquema se puede generar mediante la salida del runbook de Automation. Abra una nueva sesión del explorador web, ejecute el runbook de Automation y obtenga la salida. Vuelva a la acción **Operaciones de análisis de datos JSON de Logic Apps**. En la parte inferior de la página, seleccione **Use sample payload to generate schema** (Usar una carga de ejemplo para generar el esquema).

   :::image type="content" source="./media/custom-route-alert-portal/sample-payload.png" alt-text="Usar una carga de ejemplo para generar el esquema":::

5. En **Enter or paste a sample JSON payload** (Especificar o pegar una carga JSON de ejemplo), pegue la salida del runbook de Automation y seleccione **Done** (Listo).

   :::image type="content" source="./media/custom-route-alert-portal/paste-payload.png" alt-text="Pegar carga de ejemplo" lightbox="./media/custom-route-alert-portal/paste-payload-expand.png":::

6. Un esquema se genera automáticamente mediante el análisis de la carga de entrada JSON.

   :::image type="content" source="./media/custom-route-alert-portal/generate-schema.png" alt-text="Generar esquema" lightbox="./media/custom-route-alert-portal/generate-schema-expand.png":::

### <a name="6-define-and-initialize-a-variable"></a><a name="define-variable"></a>6. Definición e inicialización de una variable

En este paso del flujo de trabajo se crea una condición para enviar una alarma por correo electrónico. Para conseguir un formato flexible y personalizado del cuerpo del mensaje de correo electrónico, se introduce una variable auxiliar en el flujo de trabajo.

1. En la **acción Obtener resultado del trabajo**, seleccione **Nuevo paso**. En el cuadro de búsqueda, busque y seleccione **Variables**.

   :::image type="content" source="./media/custom-route-alert-portal/variables.png" alt-text="Variables":::

2. En la lista **Acciones**, seleccione la acción **Inicializar variable**.

   :::image type="content" source="./media/custom-route-alert-portal/initialize-variables.png" alt-text="Inicializar variables":::

3. Especifique el nombre de la variable. En **Tipo**, seleccione **Cadena**. El **valor** de la variable se asignará más adelante en el flujo de trabajo.

   :::image type="content" source="./media/custom-route-alert-portal/string.png" alt-text="String" lightbox="./media/custom-route-alert-portal/string-expand.png":::

### <a name="7-create-a-for-each-action"></a><a name="cycles-json"></a>7. Creación de una acción "for each"

Una vez que se analiza el archivo JSON, la acción **Analizar operaciones de datos JSON** almacena el contenido en la salida *Body*. Para procesar la salida, puede crear un bucle "for each" que repita una o varias acciones en cada elemento de la matriz.

1. En **Inicializar variable**, seleccione **Agregar una acción**. En el cuadro de búsqueda, escriba "para cada uno" como filtro.

   :::image type="content" source="./media/custom-route-alert-portal/control.png" alt-text="Control":::

2. En la lista **Acciones**, seleccione la acción **For each - Control**.

   :::image type="content" source="./media/custom-route-alert-portal/for-each.png" alt-text="For each- Control":::

3. Haga clic en el cuadro de texto **Select an output from previous steps** (Seleccionar una salida de los pasos anteriores). Cuando aparezca la lista **Contenido dinámico**, seleccione **Body** (Cuerpo), que es la salida del JSON analizado.

   :::image type="content" source="./media/custom-route-alert-portal/body.png" alt-text="Cuerpo":::

4. Para cada elemento del cuerpo JSON, queremos establecer una condición. En el grupo de acciones, seleccione **Control**.

   :::image type="content" source="./media/custom-route-alert-portal/condition-control.png" alt-text="Control":::

5. En la lista **Acciones**, seleccione **Condition-Control** (Condición-Control). Condition-Control (Condición-Control) es una estructura de control que compara los datos de un flujo de trabajo con valores o campos concretos. A continuación, se pueden especificar las distintas acciones a ejecutar en función de si los datos cumplen la condición, o no.

   :::image type="content" source="./media/custom-route-alert-portal/condition.png" alt-text="Condición - Control":::

6. En la raíz de la acción **Condition** (Condición), cambie la operación lógica a **OR**.

   :::image type="content" source="./media/custom-route-alert-portal/condition-or.png" alt-text="O" lightbox="./media/custom-route-alert-portal/condition-or-expand.png":::

7. Compruebe el valor del número de prefijos de red que una puerta de enlace de ExpressRoute anuncia a los dos pares BGP. El número de rutas está disponible en "numRoutePeer1" y "numRoutePeer2" en **Contenido dinámico**. En el cuadro de valores, escriba el valor de **numRoutePeer1**.

   :::image type="content" source="./media/custom-route-alert-portal/peer-1.png" alt-text="numRoutesPeer1":::

8. Para agregar otra fila a la condición, elija **Agregar -> Agregar fila**. En el segundo cuadro, en **Contenido dinámico**, seleccione **numRoutePeer2**.

   :::image type="content" source="./media/custom-route-alert-portal/peer-2.png" alt-text="numRoutesPeer2":::

9. La condición lógica es true cuando una de las dos variables dinámicas, numRoute1 o numRoute2, es mayor que el umbral. En este ejemplo, el umbral se ha fijado en 160 (80 % del valor máximo de 200 rutas). Puede cambiar el valor de umbral para que se adapte a sus requisitos. Para mantener la coherencia, el valor debe ser el mismo que se usa en el script de PowerShell del runbook.

   :::image type="content" source="./media/custom-route-alert-portal/logic-condition.png" alt-text="Condición lógica":::

10. En **If true** (Si es verdadero), asigne formato y cree las acciones para enviar la alerta por correo electrónico. En **Elegir una acción, busque y seleccione **Variables**.

    :::image type="content" source="./media/custom-route-alert-portal/condition-if-true.png" alt-text="Si es verdadero":::

11. En Variables, seleccione **Add an action** (Agregar una acción). En la lista de **acciones**, seleccione **Establecer variable**.

    :::image type="content" source="./media/custom-route-alert-portal/condition-set-variable.png" alt-text="Establecer variable":::

12. En **Nombre**, seleccione la variable denominada **EmailBody** que creó anteriormente. Como **Valor**, pegue el script HTML necesario para dar formato al correo electrónico de alerta. Use **Contenido dinámico** para incluir los valores del cuerpo JSON. Después de configurar estas opciones, el resultado es que la variable **Emailbody** contiene toda la información relacionada con la alerta, en formato HTML.

    :::image type="content" source="./media/custom-route-alert-portal/paste-script.png" alt-text="Establecer variable":::

### <a name="8-add-the-email-connector"></a><a name="email"></a>8. Incorporación del conector de correo electrónico

Logic Apps proporciona muchos conectores de correo electrónico. En este ejemplo, se agrega un conector de Outlook para enviar la alerta por correo electrónico. En **Establecer variable**, seleccione **Agregar una acción**. En **Elegir una acción**, escriba "enviar correo electrónico" en el cuadro de búsqueda.

1. Seleccione **Office 365 Outlook**.

   :::image type="content" source="./media/custom-route-alert-portal/email.png" alt-text="Enviar correo electrónico":::

2. En la lista **Acciones**, seleccione **Enviar un correo electrónico (V2)** .

   :::image type="content" source="./media/custom-route-alert-portal/email-v2.png" alt-text="Enviar correo electrónico (V2)":::

3. Inicie sesión para crear una conexión a Office 365 Outlook.

   :::image type="content" source="./media/custom-route-alert-portal/office-365.png" alt-text="Sign in":::

4. En el campo **Cuerpo**, haga clic en **Agregar contenido dinámico**. En el panel Contenido dinámico, agregue la variable **Emailbody**. Rellene los campos **Asunto** y **Para**.

   :::image type="content" source="./media/custom-route-alert-portal/emailbody.png" alt-text="Cuerpo":::

5. La acción **Enviar correo electrónico (V2)** completa la configuración del flujo de trabajo.

   :::image type="content" source="./media/custom-route-alert-portal/send-email-v2.png" alt-text="Enviar correo electrónico (v2)" lightbox="./media/custom-route-alert-portal/send-email-v2-expand.png":::

### <a name="9-workflow-validation"></a><a name="validation"></a>9. validación del flujo de trabajo

El paso final es la validación del flujo de trabajo. En **Logic Apps Overview** (Información general de Logic Apps), seleccione **Ejecutar desencadenador**. Seleccione **Periodicidad**. El flujo de trabajo se puede supervisar y comprobar en el **historial de ejecuciones**.

:::image type="content" source="./media/custom-route-alert-portal/trigger.png" alt-text="Ejecutar desencadenador":::

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo personalizar el flujo de trabajo, consulte [Azure Logic Apps](../logic-apps/logic-apps-overview.md).
