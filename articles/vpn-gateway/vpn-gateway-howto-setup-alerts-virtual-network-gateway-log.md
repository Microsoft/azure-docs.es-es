---
title: 'Azure VPN Gateway: Configuración de alertas en eventos de registro de recursos de diagnóstico'
description: Obtenga información sobre cómo configurar alertas basadas en eventos de registro de recursos de Azure VPN Gateway con Azure Monitor Log Analytics.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: alzam
ms.openlocfilehash: e2451796cb99f57501ed74aba1dc7a3077b51881
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89441619"
---
# <a name="set-up-alerts-on-resource-log-events-from-vpn-gateway"></a>Configuración de alertas de eventos de registro de recursos de VPN Gateway

En este artículo se le ayuda a configurar alertas basadas en eventos de registro de recursos de Azure VPN Gateway con Azure Monitor Log Analytics. 

Los siguientes registros de recursos están disponibles en Azure:

|***Nombre*** | ***Descripción*** |
|---        | ---               |
|GatewayDiagnosticLog | Contiene los registros de recursos para los eventos de configuración de puerta de enlace, los cambios principales y los eventos de mantenimiento |
|TunnelDiagnosticLog | Contiene los eventos de cambio del estado del túnel. Los eventos de conexión/desconexión del túnel tienen un motivo resumido para el cambio de estado si es aplicable |
|RouteDiagnosticLog | Registra los cambios en rutas estáticas y eventos BGP que se producen en la puerta de enlace |
|IKEDiagnosticLog | Registra mensajes de control de IKE y eventos en la puerta de enlace |
|P2SDiagnosticLog | Registra mensajes de control de punto a sitio y eventos en la puerta de enlace. La información del origen de conexión se proporciona solo para conexiones IKEv2 |

## <a name="set-up-alerts-in-the-azure-portal"></a><a name="setup"></a>Configuración de alertas en Azure Portal

Los pasos del ejemplo siguiente crean una alerta para un evento de desconexión que implica un túnel VPN de sitio a sitio:


1. En Azure Portal, seleccione **Log Analytics** en **Todos los servicios** y seleccione **Áreas de trabajo de Log Analytics**.

   ![Selecciones para acceder a las áreas de trabajo de Log Analytics](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert0.png "Crear")

2. Seleccione **Crear** en la página **Log Analytics**.

   ![Página de Log Analytics con el botón Crear](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert1.png  "Seleccionar")

3. Seleccione **Crear nuevo** y rellene los detalles.

   ![Detalles para crear un área de trabajo de Log Analytics](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert2.png  "Seleccionar")

4. Busque la instancia de VPN Gateway en la hoja **Monitor** > **Configuración de diagnóstico**.

   ![Selecciones para buscar la puerta de enlace de VPN en Configuración de diagnóstico](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert3.png  "Seleccionar")

5. Para activar los diagnósticos, haga doble clic en la puerta de enlace y, después, seleccione **Activar diagnósticos**.

   ![Selecciones para activar los diagnósticos](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert4.png  "Seleccionar")

6. Rellene los detalles y asegúrese de que **Enviar a Log Analytics** y **TunnelDiagnosticLog** están seleccionados. Elija el área de trabajo de Log Analytics que creó en el paso 3.

   ![Casillas seleccionadas](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert5.png  "Seleccionar")

   > [!NOTE]
   > Inicialmente, los datos pueden tardar unas horas en mostrarse.

7. Vaya a la introducción del recurso de la puerta de enlace de red virtual y seleccione **Alertas** en la pestaña **Supervisión**. Después, cree una regla de alertas o modifique una existente.

   ![Selecciones para crear una nueva regla de alertas](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert6.png  "Seleccionar")

   ![point-to-site](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert6.png  "Seleccionar")
8. Seleccione el área de trabajo de Log Analytics y el recurso.

   ![Selecciones para el área de trabajo y el recurso](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert7.png  "Seleccionar")

9. Seleccione **Búsqueda de registros personalizada** como la lógica de la señal en **Agregar condición**.

   ![Selecciones para una búsqueda de registros personalizada](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert8.png  "Seleccionar")

10. Escriba la consulta siguiente en el cuadro de texto **Consulta de búsqueda**. Reemplace los valores de <> y TimeGenerated según corresponda.

    ```
    AzureDiagnostics
    | where Category == "TunnelDiagnosticLog"
    | where _ResourceId == tolower("<RESOURCEID OF GATEWAY>")
    | where TimeGenerated > ago(5m) 
    | where remoteIP_s == "<REMOTE IP OF TUNNEL>"
    | where status_s == "Disconnected"
    | project TimeGenerated, OperationName, instance_s, Resource, ResourceGroup, _ResourceId 
    | sort by TimeGenerated asc
    ```

    Establezca el valor del umbral en 0 y seleccione **Listo**.

    ![Escritura de una consulta y selección de un umbral](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert9.png  "Seleccionar")

11. En la página **Crear regla**, seleccione **Crear nueva** en la sección **GRUPOS DE ACCIONES**. Rellene los detalles y seleccione **Aceptar**.

    ![Detalles para un nuevo grupo de acciones](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert10.png  "Seleccionar")

12. En la página **Crear regla**, rellene los detalles de **Personalizar acciones** y asegúrese de que aparece el nombre correcto en la sección **NOMBRE DEL GRUPO DE ACCIONES**. Seleccione **Crear regla de alerta** para crear la regla.

    ![Selecciones para la creación de una regla](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert11.png  "Seleccionar")

## <a name="set-up-alerts-by-using-powershell"></a><a name="setuppowershell"></a>Configuración de alertas mediante PowerShell

Los pasos del ejemplo siguiente crearán una alerta para un evento de desconexión que implica un túnel VPN de sitio a sitio.

1. Cree de un área de trabajo de Log Analytics:

   ```powershell
   $Location           = 'westus2'
   $ResourceGroupName  = 'TestRG1'
   $Sku                = 'pergb2018'
   $WorkspaceName      = 'LogAnalyticsWS123'

   New-AzOperationalInsightsWorkspace -Location $Location -Name $WorkspaceName -Sku $Sku -ResourceGroupName $ResourceGroupName
   ```

2. Active el diagnóstico para la puerta de enlace de VPN:

   ```powershell
   $ResourceGroupName  = 'TestRG1'
   $VpnGatewayName     = 'VNet1GW'
   $WorkspaceName      = 'LogAnalyticsWS123'

   $VpnGateway         = Get-AzVirtualNetworkGateway -Name $VpnGatewayName -ResourceGroupName $ResourceGroupName
   $Workspace          = Get-AzOperationalInsightsWorkspace -Name $WorkspaceName -ResourceGroupName $ResourceGroupName

   Set-AzDiagnosticSetting `
       -Name 'VPN tunnel' `
       -ResourceId $VpnGateway.Id `
       -WorkspaceId $Workspace.ResourceId `
       -Enabled $true `
       -Category 'TunnelDiagnosticLog'
   ```

3. Cree un grupo de acciones.

   Este código crea un grupo de acciones que envía una notificación por correo electrónico cuando se desencadena una alerta:

   ```powershell
   $ActionGroupName            = 'EmailAdmins'   # Max. 60 characters long
   $ActionGroupShortName       = 'EmailAdmins'   # Max. 12 characters long
   $ActionGroupReceiverName    = 'My receiver Name'
   $EmailAddress               = 'xyz@contoso.com'
   $ResourceGroupName          = 'TestRG1'

   $ActionGroupReceiver = New-AzActionGroupReceiver -Name $ActionGroupReceiverName -UseCommonAlertSchema -EmailReceiver -EmailAddress $EmailAddress

   Set-AzActionGroup `
      -ResourceGroupName $ResourceGroupName `
      -Name $ActionGroupName `
      -ShortName $ActionGroupShortName `
      -Receiver @($ActionGroupReceiver)
   ```

4. Cree una regla de alerta basada en una búsqueda de registros personalizada:

   ```powershell
   $ActionGroupName    = 'EmailAdmins'
   $EmailSubject       = 'Redmond VPN tunnel is disconnected'
   $Location           = 'westus2'
   $RemoteIp           = '104.42.209.46'
   $ResourceGroupName  = 'TestRG1'
   $VpnGatewayName     = 'VNet1GW'
   $WorkspaceName      = 'LogAnalyticsWS123'

   $VpnGateway         = Get-AzVirtualNetworkGateway -Name $VpnGatewayName -ResourceGroupName $ResourceGroupName
   $Workspace          = Get-AzOperationalInsightsWorkspace -Name $WorkspaceName -ResourceGroupName $ResourceGroupName

   $Query = @"
   AzureDiagnostics |
   where Category == "TunnelDiagnosticLog" |
   where TimeGenerated > ago(5m) |
   where _ResourceId == tolower("$($VpnGateway.id)") |
   where remoteIP_s == "$($RemoteIp)" |
   where status_s == "Disconnected" |
   project TimeGenerated, OperationName, instance_s, Resource, ResourceGroup, _ResourceId |
   sort by TimeGenerated asc
   "@

   $Source             = New-AzScheduledQueryRuleSource -Query $Query -DataSourceId $Workspace.ResourceId
   $Schedule           = New-AzScheduledQueryRuleSchedule -FrequencyInMinutes 5 -TimeWindowInMinutes 5
   $TriggerCondition   = New-AzScheduledQueryRuleTriggerCondition -ThresholdOperator 'GreaterThan' -Threshold 0

   $ActionGroup        = Get-AzActionGroup -ResourceGroupName $ResourceGroupName -Name $ActionGroupName
   $AznsActionGroup    = New-AzScheduledQueryRuleAznsActionGroup -ActionGroup $ActionGroup.Id -EmailSubject $EmailSubject
   $AlertingAction     = New-AzScheduledQueryRuleAlertingAction -AznsAction $AznsActionGroup -Severity '1' -Trigger $TriggerCondition

   New-AzScheduledQueryRule `
       -ResourceGroupName $ResourceGroupName `
       -Location $Location `
       -Action $AlertingAction `
       -Enabled $true `
       -Description 'The tunnel between Azure and Redmond with IP address 104.42.209.46 is disconnected' `
       -Schedule $Schedule `
       -Source $Source `
       -Name 'The Azure to Redmond tunnel is disconnected'
   ```

## <a name="next-steps"></a>Pasos siguientes

Para configurar alertas en las métricas de túnel, vea [Configuración de alertas en métricas de VPN Gateway](vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric.md).
