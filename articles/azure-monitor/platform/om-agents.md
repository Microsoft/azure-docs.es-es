---
title: Conexión de Operations Manager con Azure Monitor | Microsoft Docs
description: Para mantener su inversión existente en System Center Operations Manager y usar las capacidades ampliadas con Log Analytics, puede integrar Operations Manager con el área de trabajo.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/24/2020
ms.openlocfilehash: 2a4f24da51b9e9e78c3df3e7d1437a380306e300
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318358"
---
# <a name="connect-operations-manager-to-azure-monitor"></a>Conexión de Operations Manager con Azure Monitor

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

Para mantener su inversión existente en [System Center Operations Manager](/system-center/scom/key-concepts?view=sc-om-1807) y usar las funcionalidades ampliadas con Azure Monitor, puede integrar Operations Manager con el área de trabajo de Log Analytics. Esto le permite aprovechar las oportunidades de los registros en Azure Monitor mientras sigue utilizando Operations Manager para:

* Supervisar el estado de los servicios de TI con Operations Manager
* Mantener la integración con sus soluciones ITSM compatibles con la administración de incidentes y problemas
* Administrar el ciclo de vida de los agentes implementados en máquinas virtuales IaaS en una nube pública y en entornos locales supervisados con Operations Manager

La integración con System Center Operations Manager agrega valor a su estrategia de operaciones de servicio al usar la velocidad y la eficacia de Azure Monitor para recopilar, almacenar y analizar datos de Operations Manager. Gracias a las consultas de registros de Azure Monitor podrá correlacionar y trabajar para identificar los errores de los problemas, así como usar opciones de periodicidad que le ayudarán a respaldar el proceso de administración de problemas existente. La flexibilidad del motor de consulta para examinar el rendimiento, eventos y datos de alertas con abundantes paneles y funcionalidades de informes para exponer estos datos de maneras significativas muestra la solidez que ofrece Azure Monitor como complemento de Operations Manager.

Los agentes que informan al grupo de administración de Operations Manager recopilan datos de los servidores en función de los [orígenes de datos de Log Analytics](agent-data-sources.md) y de las soluciones habilitadas en el área de trabajo. Dependiendo de las soluciones habilitadas, los datos se envían directamente desde un servidor de administración de Operations Manager al servicio o, debido al volumen de los datos recopilados en el sistema que administran los agentes, se envían directamente desde el agente a un área de trabajo de Log Analytics. El servidor de administración reenvía los datos directamente al servicio. No se escriben nunca en la base de datos operacional o de almacenamiento de datos. Cuando un servidor de administración pierde conectividad con Azure Monitor, almacena en caché los datos localmente hasta que la comunicación se restablezca. Si el servidor de administración está sin conexión debido a un mantenimiento planeado o a una interrupción imprevista, otro servidor de administración del grupo de administración reanudará la conectividad de Azure Monitor.  

En el siguiente diagrama se muestra la conexión entre los agentes y servidores de administración de un grupo de administración de System Center Operations Manager y Azure Monitor, incluidos los puertos y la dirección.

![oms-operations-manager-integration-diagram](./media/om-agents/oms-operations-manager-connection.png)

Si las directivas de seguridad de TI no permiten que los equipos de la red se conecten a Internet, se pueden configurar los servidores de administración de forma que se conecten a la puerta de enlace de Log Analytics para recibir información de configuración y enviar los datos recopilados según las soluciones que haya habilitado. Para obtener más información y conocer los pasos para configurar el grupo de administración de Operations Manager con el fin de que se comunique mediante una puerta de enlace de Log Analytics con Azure Monitor, consulte [Conexión de equipos a Azure Monitor con la puerta de enlace de Log Analytics](./gateway.md).  

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, revise los siguientes requisitos.

* Azure Monitor solo admite System Center Operations Manager 2016 y versiones posteriores, Operations Manager 2012 SP1 UR6 y versiones posteriores, y Operations Manager 2012 R2 UR2 y versiones posteriores. Se agregó compatibilidad con proxy en Operations Manager 2012 SP1 UR7 y Operations Manager 2012 R2 UR3.
* La integración de System Center Operations Manager 2016 con la nube del US Gov requiere un módulo de administración de Advisor actualizado incluido con el paquete acumulativo de actualizaciones 2 o posterior. System Center Operations Manager 2012 R2 requiere un módulo de administración de Advisor actualizado incluido con el paquete acumulativo de actualizaciones 3 o posterior.
* Todos los agentes de Operations Manager deben cumplir los requisitos mínimos de compatibilidad. Asegúrese de que los agentes están actualizados con los requisitos mínimos ya que, de lo contrario, se podría producir un error de comunicación en el agente de Windows así como errores en el registro de eventos de Operations Manager.
* Un área de trabajo de Log Analytics. Para más información, consulte la [introducción a las áreas de trabajo de Log Analytics](design-logs-deployment.md).
* El usuario se autentica en Azure con una cuenta que es miembro del [rol Colaborador de Log Analytics](manage-access.md#manage-access-using-azure-permissions).

* Regiones admitidas: System Center Operations Manager solo es compatible con las siguientes regiones de Azure para conectarse a un área de trabajo de Log Analytics:
    - Centro-Oeste de EE. UU.
    - Sudeste de Australia
    - Oeste de Europa
    - Este de EE. UU.
    - Sudeste de Asia
    - Japón Oriental
    - Sur de Reino Unido 2
    - Centro de la India
    - Centro de Canadá
    - Oeste de EE. UU. 2

>[!NOTE]
>Los cambios recientes en las API de Azure impedirán que los clientes puedan configurar con éxito la integración entre el grupo de administración y Azure Monitor por primera vez. Para los clientes que ya han integrado su grupo de administración con el servicio, no se verán afectados a menos que necesiten reconfigurar su conexión existente.  
>Se ha publicado un nuevo módulo de administración para las siguientes versiones de Operations Manager:
> - En System Center Operations Manager 2019, este módulo de administración se incluye con los medios de origen y se instala durante la instalación de un nuevo grupo de administración o durante una actualización.
>- El módulo de administración de Operations Manager 1801 también es aplicable a Operations Manager 1807.
>- En el caso de System Center Operations Manager 1801, descargue el módulo de administración [aquí](https://www.microsoft.com/download/details.aspx?id=57173).
>- En el caso de System Center 2016 Operations Manager, descargue el módulo de administración [aquí](https://www.microsoft.com/download/details.aspx?id=57172).  
>- En el caso de System Center Operations Manager 2012 R2, descargue el módulo de administración [aquí](https://www.microsoft.com/download/details.aspx?id=57171).  


### <a name="network"></a>Red

A continuación, se muestra la información de configuración de proxy y firewall requerida para que el agente de Operations Manager, los servidores de administración y la Consola del operador se comuniquen con Azure Monitor. El tráfico de cada componente es de salida (de la red a Azure Monitor).

|Resource | Número de puerto| Omitir inspección de HTTP|  
|---------|------|-----------------------|  
|**Agent**|||  
|\*.ods.opinsights.azure.com| 443 |Sí|  
|\*.oms.opinsights.azure.com| 443|Sí|  
|\*blob.core.windows.net| 443|Sí|  
|\*.azure-automation.net| 443|Sí|  
|**Servidor de administración**|||  
|\*.service.opinsights.azure.com| 443||  
|\*blob.core.windows.net| 443| Sí|  
|\*.ods.opinsights.azure.com| 443| Sí|  
|*.azure-automation.net | 443| Sí|  
|**Consola de Operations Manager a Azure Monitor**|||  
|service.systemcenteradvisor.com| 443||  
|\*.service.opinsights.azure.com| 443||  
|\*.live.com| 80 y 443||  
|\*.microsoft.com| 80 y 443||  
|\*.microsoftonline.com| 80 y 443||  
|\*.mms.microsoft.com| 80 y 443||  
|login.windows.net| 80 y 443||  
|portal.loganalytics.io| 80 y 443||
|api.loganalytics.io| 80 y 443||
|docs.loganalytics.io| 80 y 443||  

### <a name="tls-12-protocol"></a>Protocolo TLS 1.2

Para garantizar la seguridad de los datos en tránsito a Azure Monitor, se recomienda encarecidamente configurar el agente y el grupo de administración para que use al menos Seguridad de la capa de transporte (TLS) 1.2. Las versiones anteriores de TLS/Capa de sockets seguros (SSL) han demostrado ser vulnerables y, si bien todavía funcionan para permitir la compatibilidad con versiones anteriores, **no se recomiendan**. Para información adicional, revise [Sending data securely using TLS 1.2](./data-security.md#sending-data-securely-using-tls-12) (Envío de datos de forma segura mediante TLS 1.2).

## <a name="connecting-operations-manager-to-azure-monitor"></a>Conectar Operations Manager con Azure Monitor

Ejecute la siguiente serie de pasos para configurar el grupo de administración de Operations Manager para conectarse a una de las áreas de trabajo de Log Analytics.

Durante el registro inicial del grupo de administración de Operations Manager con un área de trabajo de Log Analytics, la opción para especificar la configuración de proxy para el grupo de administración no está disponible en la consola del operador.  El grupo de administración tiene que estar registrado correctamente con el servicio para que esta opción esté disponible.  Para solucionar esto, tiene que actualizar la configuración de proxy del sistema con Netsh en el sistema desde el que se esté ejecutando la consola del operador para configurar la integración y todos los servidores de administración del grupo de administración.  

1. Abra un símbolo del sistema con privilegios elevados.
   a. Vaya a **Inicio** y escriba **cmd**.
   b. Haga clic con el botón derecho en el **símbolo del sistema** y seleccione Ejecutar como administrador\*\*.
1. Escriba el siguiente comando y presione **Entrar**:

    `netsh winhttp set proxy <proxy>:<port>`

Después de completar la integración con Azure Monitor, puede quitar la configuración; para hacerlo, ejecute `netsh winhttp reset proxy` y, después, use la opción **Configurar servidor proxy** de la Consola del operador para especificar el proxy o el servidor de puerta de enlace de Log Analytics.

1. En la consola de Operations Manager, seleccione el área de trabajo **Administración** .
1. Expanda el nodo de Operations Management Suite y haga clic en **Conexión**.
1. Haga clic en el vínculo **Register to Operations Management Suite** (Registrar en Operations Management Suite).
1. En la página **Asistente para incorporación a Operations Management Suite: Autenticación**, escriba la dirección de correo electrónico o el número de teléfono y la contraseña de la cuenta de administrador asociados a su suscripción de OMS y, después, haga clic en **Iniciar sesión**.

   >[!NOTE]
   >Se ha retirado el nombre de Operations Management Suite.

1. Una vez autenticado correctamente, en la página **Asistente para incorporación a Operations Management Suite: Seleccionar área de trabajo**, se le pedirá que seleccione el inquilino de Azure, la suscripción y el área de trabajo de Log Analytics. Si tiene más de un área de trabajo, seleccione el que desee registrar con el grupo de administración de Operations Manager en la lista desplegable y haga clic en **Siguiente**.

   > [!NOTE]
   > Operations Manager solo admite un área de trabajo de Log Analytics a la vez. Se quitan de Azure Monitor la conexión y los equipos que se registraron en Azure Monitor con el área de trabajo anterior.
   >
   >
1. En la página **Asistente para incorporación a Operations Management Suite: Resumen**, confirme la configuración y, si es correcta, haga clic en **Crear**.
1. En la página **Asistente para incorporación a Operations Management Suite: Finalizar**, haga clic en **Cerrar**.

### <a name="add-agent-managed-computers"></a>Adición de equipos administrados por agente

Después de configurar la integración con el área de trabajo de Log Analytics, esto solo establece una conexión con el servicio, y no se recopilan datos de los agentes que informan al grupo de administración. Esto no será posible hasta que configure qué equipos específicos administrados por agente recopilan los datos de registro para Azure Monitor. Puede seleccionar los objetos de equipo individualmente o un grupo que contenga objetos de equipos Windows. No puede seleccionar un grupo que contenga instancias de otra clase, como discos lógicos o bases de datos SQL.

1. Abra la consola de Operations Manager y seleccione el área de trabajo **Administración** .
1. Expanda el nodo de Operations Management Suite y haga clic en **Conexión**.
1. Haga clic en el vínculo **Agregar un equipo/grupo** debajo del título Acciones en el lado derecho del panel.
1. En el cuadro de diálogo **Búsqueda de equipos**, puede buscar equipos o grupos supervisados por Operations Manager. Seleccione equipos o grupos, incluido el servidor de administración de Operations Manager que se va a incorporar a Azure Monitor, haga clic en **Agregar** y, a continuación, haga clic en **Aceptar**.

Puede ver los equipos y grupos configurados para recopilar datos desde el nodo Equipos administrados de Operations Management Suite en el área de trabajo **Administración** de la consola de Operations. Desde aquí puede agregar o quitar equipos y grupos según sea necesario.

### <a name="configure-proxy-settings-in-the-operations-console"></a>Configurar el proxy en la Consola del operador

Realice los pasos siguientes si un servidor proxy interno se encuentra entre el grupo de administración y Azure Monitor. Esta configuración se administra centralmente desde el grupo de administración y se distribuye en sistemas administrados por agentes que se incluyen en el ámbito para que recopilen datos de registro para Azure Monitor.  Esto es beneficioso cuando determinadas soluciones omiten el servidor de administración y envían datos directamente al servicio.

1. Abra la consola de Operations Manager y seleccione el área de trabajo **Administración** .
1. Expanda Operations Management Suite y haga clic en **Conexiones**.
1. En la vista Conexión de OMS, haga clic en **Configurar servidor proxy**.
1. En la página **Asistente para configuración de Operations Management Suite: Proxy Server**, seleccione **Use un servidor proxy para obtener acceso al servicio Operations Management Suite**, escriba la dirección URL con el número de puerto, por ejemplo, http://corpproxy:80 y después haga clic en **Finalizar**.

Si el servidor proxy necesita autenticación, siga este procedimiento para configurar las credenciales y opciones necesarias para realizar la propagación a los equipos administrados que notifican a Azure Monitor en el grupo de administración.

1. Abra la consola de Operations Manager y seleccione el área de trabajo **Administración** .
1. En **RunAs Configuration**, seleccione **Perfiles**.
1. Abra el perfil de **proxy del perfil Ejecutar como de System Center Advisor** .
1. En el Asistente para crear perfiles de ejecución, haga clic en Agregar para usar una cuenta de ejecución. Puede crear una [cuenta de ejecución](/previous-versions/system-center/system-center-2012-R2/hh321655(v=sc.12)) o usar una existente. Esta cuenta debe contar con los permisos suficientes para pasar a través del servidor proxy.
1. Para establecer la cuenta que se va a administrar, elija **una clase seleccionada, un grupo o un objeto**, haga clic en **Seleccionar** y, después, en **Agrupar**. Para abrir el cuadro **Búsqueda de grupos**.
1. Busque y seleccione **Microsoft System Center Advisor Monitoring Server Group**(Grupo de servidores de supervisión de Microsoft System Center Advisor). Haga clic en **Aceptar** después de seleccionar el grupo para cerrar el cuadro **Búsqueda de grupos**.
1. Haga clic en **Aceptar** para cerrar el cuadro **Agregar una ejecución como cuenta**.
1. Haga clic en **Guardar** para completar el asistente y guardar los cambios.

Después de crear la conexión y de configurar qué agentes van a recopilar los datos de registro e informar sobre ellos a Azure Monitor, se aplica la siguiente configuración en el grupo de administración, no necesariamente en orden:

* Se crea la cuenta de ejecución **Microsoft.SystemCenter.Advisor.RunAsAccount.Certificate** . Está asociada con el perfil de ejecución **Microsoft System Center Advisor Run As Profile Blob** (Blob del perfil de ejecución de Microsoft System Center Advisor) y está dirigida a dos clases: **Servidor de recopilación** y **Grupo de administración de Operations Manager**.
* Se crean dos conectores.  El primero se denomina **Microsoft.SystemCenter.Advisor.DataConnector** y se configura automáticamente con una suscripción que reenvía todas las alertas generadas desde instancias de todas las clases en el grupo de administración a Azure Monitor. El segundo conector es **Advisor Connector**, que es responsable de comunicarse con Azure Monitor y de compartir datos.
* Se agregan los agentes y los grupos que haya seleccionado para recopilar datos del grupo de administración al **grupo de servidores de supervisión de Microsoft System Center Advisor**.

## <a name="management-pack-updates"></a>Actualizaciones del módulo de administración

Después de finalizar la configuración, el grupo de administración de Operations Manager establece una conexión con Azure Monitor. El servidor de administración se sincroniza con el servicio web y recibe información de configuración actualizada en forma de módulos de administración para las soluciones que ha habilitado que se integran con Operations Manager. Operations Manager busca actualizaciones de estos módulos de administración y los descargará e importa automáticamente cuando estén disponibles. Hay dos reglas en particular que controlan este comportamiento:

* **Microsoft.SystemCenter.Advisor.MPUpdate**: actualiza los módulos de administración base de Azure Monitor. Se ejecuta cada 12 horas de forma predeterminada.
* **Microsoft.SystemCenter.Advisor.Core.GetIntelligencePacksRule** : Actualiza los módulos de administración de las soluciones habilitados en el área de trabajo. Se ejecuta cada cinco (5) minutos de forma predeterminada.

Puede invalidar estas dos reglas para evitar la descarga automática mediante la deshabilitación o modificación de la frecuencia de sincronización del servidor de administración con Azure Monitor para determinar si está disponible un nuevo módulo de administración y si debe descargarse. Siga los pasos [Cómo reemplazar una regla o monitor](/previous-versions/system-center/system-center-2012-R2/hh212869(v=sc.12)) para modificar el parámetro **Frequency** con un valor en segundos para cambiar la programación de sincronización o modificar el parámetro **Enabled** para deshabilitar las reglas. Dirige las invalidaciones para todos los objetos de la clase del grupo de administración de Operations Manager.

Para seguir el proceso de control de cambios existente para controlar las versiones del módulo de administración en el grupo de administración de producción, puede deshabilitar las reglas y habilitarlas durante determinadas horas cuando se permiten las actualizaciones. Si tiene un grupo de desarrollo o de administración de control de calidad en el entorno y tiene conectividad a Internet, puede configurar ese grupo de administración con un área de trabajo de Log Analytics para admitir este escenario. Esto le permite revisar y evaluar las versiones iterativas de los módulos de administración de Azure Monitor antes de publicarlas en el grupo de administración de producción.

## <a name="switch-an-operations-manager-group-to-a-new-log-analytics-workspace"></a>Cambiar un grupo de Operations Manager a una nueva área de trabajo de Log Analytics

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
1. En Azure Portal, haga clic en **Más servicios**, en la esquina inferior izquierda. En la lista de recursos, escriba **Log Analytics**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Log Analytics** y, a continuación, cree un área de trabajo.  
1. Abra la consola de Operations Manager con una cuenta que sea miembro del rol Administradores de Operations Manager y seleccione el área de trabajo **Administración** .
1. Expanda Log Analytics y seleccione **Conexiones**.
1. Seleccione el vínculo **Re-configure Operation Management Suite** (Volver a configurar Operation Management Suite) en el centro del panel.
1. Siga los pasos del **Asistente para incorporación a Log Analytics** y escriba la dirección de correo electrónico o el número de teléfono y la contraseña de la cuenta de administrador asociada a la nueva área de trabajo de Log Analytics.

   > [!NOTE]
   > En la página **Asistente para incorporación a Operations Management Suite: Seleccionar área de trabajo**, se presenta el área de trabajo existente que se está usando.
   >
   >

## <a name="validate-operations-manager-integration-with-azure-monitor"></a>Validar la integración de Operations Manager con Azure Monitor

Use la siguiente consulta para obtener las instancias conectadas de Operations Manager:

```azurepowershell
union *
| where isnotempty(MG)
| where not(ObjectName == 'Advisor Metrics' or ObjectName == 'ManagedSpace')
| summarize LastData = max(TimeGenerated) by lowerCasedComputerName=tolower(Computer), MG, ManagementGroupName
| sort by lowerCasedComputerName asc
```

## <a name="remove-integration-with-azure-monitor"></a>Quitar la integración con Azure Monitor

Cuando ya no necesite la integración entre el grupo de administración de Operations Manager y el área de trabajo de Log Analytics, es necesario realizar varios pasos para quitar correctamente la conexión y la configuración del grupo de administración. El siguiente procedimiento actualiza el área de trabajo de Log Analytics. Para ello, elimina la referencia del grupo de administración, elimina los conectores de Azure Monitor y, a continuación, elimina los módulos de administración que admiten la integración con el servicio.  

Los módulos de administración para las soluciones que ha habilitado y que se integran con Operations Manager, así como los módulos de administración necesarios para admitir la integración con Azure Monitor, no se pueden eliminar fácilmente del grupo de administración. Esto es porque algunos de los módulos de administración de Azure Monitor tienen dependencias en otros módulos de administración relacionados. Para eliminar los módulos de administración con una dependencia en otros módulos de administración, descargue el script para [quitar un módulo de administración con dependencias](https://gallery.technet.microsoft.com/scriptcenter/Script-to-remove-a-84f6873e) desde el centro de scripts de TechNet.  

1. Abra el shell de comandos de Operations Manager con una cuenta que sea miembro del rol Administradores de Operations Manager.

    > [!WARNING]
    > Compruebe que no tiene ningún módulo de administración personalizado con la palabra Advisor o IntelligencePack en el nombre antes de continuar. De lo contrario, los siguientes pasos los eliminarán del grupo de administración.
    >

1. Desde el símbolo del shell de comandos, escriba `Get-SCOMManagementPack -name "*Advisor*" | Remove-SCOMManagementPack -ErrorAction SilentlyContinue`
1. Después, escriba `Get-SCOMManagementPack -name "*IntelligencePack*" | Remove-SCOMManagementPack -ErrorAction SilentlyContinue`
1. Para quitar los módulos de administración restantes que tienen una dependencia en otros módulos de administración de System Center Advisor, use el script *RecursiveRemove.ps1* que descargó antes desde el centro de scripts de TechNet.  

    > [!NOTE]
    > El paso para quitar los módulos de administración de Advisor con PowerShell no eliminará automáticamente los módulos de administración interna de Microsoft System Center Advisor o Microsoft System Center Advisor.  No intente eliminarlos.  
    >  

1. Abra la consola de operaciones de Operations Manager con una cuenta que sea miembro del rol Administradores de Operations Manager.
1. Bajo **Administración**, seleccione el nodo **Módulos de administración** y, en el cuadro **Buscar**, escriba **Advisor** y compruebe que los siguientes módulos de administración todavía se importan en el grupo de administración:

   * Microsoft System Center Advisor
   * Microsoft System Center Advisor Internal

1. En Azure Portal, haga clic en el icono de **Configuración**.
1. Seleccione **Connected Sources**(Orígenes conectados).
1. En la tabla de la sección de System Center Operations Manager, verá el nombre del grupo de administración que desea quitar del área de trabajo. En la columna **Last Data** (Últimos datos), haga clic en **Quitar**.  

    > [!NOTE]
    > El vínculo **Quitar** no estará disponible hasta después de 14 días si no hay ninguna actividad detectada en el grupo de administración conectado.  
    >

1. Aparecerá una ventana que le pide que confirme que desea continuar con la eliminación.  Haga clic en **Sí** para continuar.

Para eliminar los dos conectores (Microsoft.SystemCenter.Advisor.DataConnector y Advisor Connector), guarde el siguiente script de PowerShell en el equipo y ejecútelo mediante los ejemplos siguientes:

```
    .\OM2012_DeleteConnectors.ps1 "Advisor Connector" <ManagementServerName>
    .\OM2012_DeleteConnectors.ps1 "Microsoft.SystemCenter.Advisor.DataConnector" <ManagementServerName>
```

> [!NOTE]
> El equipo desde el que se ejecuta este script, si no es un servidor de administración, debe tener el shell de comandos de Operations Manager instalado según la versión del grupo de administración.
>
>

```powershell
    param(
    [String] $connectorName,
    [String] $msName="localhost"
    )
    $mg = new-object Microsoft.EnterpriseManagement.ManagementGroup $msName
    $admin = $mg.GetConnectorFrameworkAdministration()
    ##########################################################################################
    # Configures a connector with the specified name.
    ##########################################################################################
    function New-Connector([String] $name)
    {
         $connectorForTest = $null;
         foreach($connector in $admin.GetMonitoringConnectors())
    {
    if($connectorName.Name -eq ${name})
    {
         $connectorForTest = Get-SCOMConnector -id $connector.id
    }
    }
    if ($connectorForTest -eq $null)
    {
         $testConnector = New-Object Microsoft.EnterpriseManagement.ConnectorFramework.ConnectorInfo
         $testConnector.Name = $name
         $testConnector.Description = "${name} Description"
         $testConnector.DiscoveryDataIsManaged = $false
         $connectorForTest = $admin.Setup($testConnector)
         $connectorForTest.Initialize();
    }
    return $connectorForTest
    }
    ##########################################################################################
    # Removes a connector with the specified name.
    ##########################################################################################
    function Remove-Connector([String] $name)
    {
        $testConnector = $null
        foreach($connector in $admin.GetMonitoringConnectors())
       {
        if($connector.Name -eq ${name})
       {
         $testConnector = Get-SCOMConnector -id $connector.id
       }
      }
     if ($testConnector -ne $null)
     {
        if($testConnector.Initialized)
     {
     foreach($alert in $testConnector.GetMonitoringAlerts())
     {
       $alert.ConnectorId = $null;
       $alert.Update("Delete Connector");
     }
     $testConnector.Uninitialize()
     }
     $connectorIdForTest = $admin.Cleanup($testConnector)
     }
    }
    ##########################################################################################
    # Delete a connector's Subscription
    ##########################################################################################
    function Delete-Subscription([String] $name)
    {
      foreach($testconnector in $admin.GetMonitoringConnectors())
      {
      if($testconnector.Name -eq $name)
      {
        $connector = Get-SCOMConnector -id $testconnector.id
      }
    }
    $subs = $admin.GetConnectorSubscriptions()
    foreach($sub in $subs)
    {
      if($sub.MonitoringConnectorId -eq $connector.id)
      {
        $admin.DeleteConnectorSubscription($admin.GetConnectorSubscription($sub.Id))
      }
     }
    }
    #New-Connector $connectorName
    write-host "Delete-Subscription"
    Delete-Subscription $connectorName
    write-host "Remove-Connector"
    Remove-Connector $connectorName
```

En el futuro, si planea volver a conectar su grupo de administración a un área de trabajo de Log Analytics, necesitará volver a importar el archivo del módulo de administración `Microsoft.SystemCenter.Advisor.Resources.\<Language>\.mpb`. Según la versión de System Center Operations Manager implementada en el entorno, puede encontrar este archivo en la siguiente ubicación:

* En los medios de origen de la carpeta `\ManagementPacks` para System Center 2016 - Operations Manager y versiones posteriores.
* Desde el paquete acumulativo de actualizaciones más reciente que se aplica al grupo de administración. En Operations Manager 2012, la carpeta de origen es `%ProgramFiles%\Microsoft System Center 2012\Operations Manager\Server\Management Packs for Update Rollups` y, para la versión 2012 R2, se encuentra en `System Center 2012 R2\Operations Manager\Server\Management Packs for Update Rollups`.

## <a name="next-steps"></a>Pasos siguientes

Para agregar funcionalidad y recopilar información, consulte [Incorporación de soluciones de Azure Monitor desde la galería de soluciones](../insights/solutions.md).

