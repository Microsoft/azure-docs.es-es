---
title: Aplicación de orquestación de revisiones de Azure Service Fabric | Microsoft Docs
description: Aplicación para automatizar la aplicación de revisiones de sistema operativo en un clúster de Service Fabric.
services: service-fabric
documentationcenter: .net
author: khandelwalbrijeshiitr
manager: chackdan
editor: ''
ms.assetid: de7dacf5-4038-434a-a265-5d0de80a9b1d
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 2/01/2019
ms.author: brkhande
ms.openlocfilehash: aca34ee40bfe10c55c478d9aaeb01a65d139e1e2
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522376"
---
# <a name="patch-the-windows-operating-system-in-your-service-fabric-cluster"></a>Revisión del sistema operativo Windows en el clúster de Service Fabric

> 
> [!IMPORTANT]
> Versión 1.2 de la aplicación. * es que salen de soporte técnico en el 30 de abril de 2019. Actualice a la versión más reciente.


Usar [actualizaciones automáticas de imágenes del SO del conjunto de escalado de máquinas virtuales de Azure](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade) es el procedimiento recomendado para mantener los sistemas operativos con las revisiones instaladas en Azure, y Patch Orchestration Application (POA) es un contenedor del servicio del sistema Administrador de reparaciones de Service Fabric que permite programar la instalación de revisiones del SO basada en configuración para clústeres hospedados en ubicaciones distintas de Azure. Aunque no se necesita POA para clústeres hospedados en ubicaciones distintas de Azure, es necesario programar la instalación de revisiones mediante Dominios de actualización para aplicar revisiones en los hosts de clústeres de Service Fabric sin tiempo de inactividad.

POA es una aplicación de Azure Service Fabric que automatiza la instalación de revisiones de sistema operativo en un clúster de Service Fabric sin tiempo de inactividad.

La aplicación de orquestación de revisiones ofrece las siguientes características:

- **Instalación automática de actualizaciones de sistema operativo**. Las actualizaciones de sistema operativo se descargan e instalan automáticamente. Los nodos de clúster se reinician según sea necesario sin tiempo de inactividad del clúster.

- **Integración de la aplicación de revisiones y el estado del clúster**. Al aplicar las actualizaciones, la aplicación de orquestación de revisiones supervisa el estado de los nodos de clúster. Los nodos de clúster se actualizan de uno en uno o por dominio de actualización. Si el estado del clúster deja de funcionar debido al proceso de aplicación de revisiones, la aplicación de revisiones se detiene para evitar agravar el problema.

## <a name="internal-details-of-the-app"></a>Detalles internos de la aplicación

La aplicación de orquestación de revisiones consta de los siguientes subcomponentes:

- **Coordinator Service**: este servicio con estado es responsable de:
    - Coordinar el trabajo de Windows Update en todo el clúster.
    - Almacenar el resultado de las operaciones completadas de Windows Update.
- **Node Agent Service**: es un servicio sin estado que se ejecuta en todos los nodos del clúster de Service Fabric. El servicio es responsable de:
    - Arrancar el servicio Node Agent NTService.
    - Supervisar el servicio Node Agent NTService.
- **Node Agent NTService**: este servicio de Windows NT se ejecuta con privilegios elevados (SYSTEM). En cambio, Node Agent Service y Coordinator Service se ejecutan con privilegios de nivel inferior (NETWORK SERVICE). El servicio es responsable de realizar las siguientes tareas de Windows Update en todos los nodos de clúster:
    - Deshabilitar las actualizaciones automáticas de Windows en el nodo.
    - Descargar e instalar Windows Update según la directiva que el usuario ha proporcionado.
    - Reiniciar el equipo después de la instalación de Windows Update.
    - Realizar la carga de los resultados de las actualizaciones de Windows al servicio Coordinator Service.
    - Realizar un informe de estado en caso de error en la operación después de agotar todos los reintentos.

> [!NOTE]
> La aplicación de orquestación de revisiones usa el servicio de sistema de administrador de reparaciones de Service Fabric para habilitar o deshabilitar el nodo, y llevar a cabo comprobaciones de estado. La tarea de reparación creada por la aplicación de orquestación de revisiones sigue el progreso de Windows Update en cada nodo.

## <a name="prerequisites"></a>Requisitos previos

> [!NOTE]
> .NET framework versión mínima necesaria es 4.6.

### <a name="enable-the-repair-manager-service-if-its-not-running-already"></a>Habilitar el servicio de administrador de reparaciones (si no se está ejecutando ya)

La aplicación de orquestación de revisiones requiere que el servicio del sistema de administrador de reparaciones esté habilitado en el clúster.

#### <a name="azure-clusters"></a>Clústeres de Azure

Los clústeres de Azure en el nivel de durabilidad Plata tendrán habilitado el administrador de reparaciones de forma predeterminada. Es posible que los clústeres de Azure en el nivel de durabilidad Oro tengan habilitado o no el servicio de administrador de reparaciones, dependiendo de cuándo se crearon esos clústeres. Los clústeres de Azure en el plan de durabilidad Bronce no tienen habilitado el servicio de administrador de reparaciones. Si el servicio ya está habilitado, puede ver que se ejecuta en la sección de servicios del sistema en Service Fabric Explorer.

##### <a name="azure-portal"></a>Azure Portal
Puede habilitar el administrador de reparaciones desde Azure Portal en el momento de la configuración del clúster. Seleccione la opción **Incluir administrador de reparaciones** en **Características complementarias** en el momento de la configuración del clúster.
![Imagen de la habilitación del administrador de reparaciones de Azure Portal](media/service-fabric-patch-orchestration-application/EnableRepairManager.png)

##### <a name="azure-resource-manager-deployment-model"></a>modelo de implementación de Azure Resource Manager
También puede usar el [modelo de implementación de Azure Resource Manager](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm) para habilitar el servicio del administrador de reparaciones en clústeres de Service Fabric nuevos y existentes. Obtenga la plantilla del clúster que desea implementar. Puede usar las plantillas de ejemplo o crear una plantilla del modelo de implementación de Azure Resource Manager personalizada. 

Para habilitar el servicio del administrador de reparaciones mediante la [plantilla del modelo de implementación de Azure Resource Manager](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm):

1. En primer lugar, compruebe que `apiversion` está establecido en `2017-07-01-preview` para el recurso `Microsoft.ServiceFabric/clusters`. Si es diferente, debe actualizar `apiVersion` al valor `2017-07-01-preview` o superior:

    ```json
    {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

2. Ahora, habilite el servicio de administrador de reparaciones agregando la sección `addonFeatures` siguiente después de la sección `fabricSettings`:

    ```json
    "fabricSettings": [
        ...      
    ],
    "addonFeatures": [
        "RepairManager"
    ],
    ```

3. Una vez actualizada la plantilla del clúster con estos cambios, aplíquelos para completar la actualización. Ahora puede ver el servicio del sistema de administrador de reparaciones en ejecución en el clúster. Se denomina `fabric:/System/RepairManagerService` en la sección de servicios del sistema en Service Fabric Explorer. 

### <a name="standalone-on-premises-clusters"></a>Clústeres locales independientes

Puede usar las [Opciones de configuración de clústeres de Windows independientes](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-manifest) para habilitar el servicio de administrador de reparaciones en clústeres de Service Fabric nuevos y existentes.

Para habilitar el servicio de administrador de reparaciones:

1. En primer lugar, compruebe que la `apiversion` en [Opciones generales de configuración de clústeres](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-manifest#general-cluster-configurations) está establecida en `04-2017` o superior:

    ```json
    {
        "name": "SampleCluster",
        "clusterConfigurationVersion": "1.0.0",
        "apiVersion": "04-2017",
        ...
    }
    ```

2. Ahora, habilite el servicio de administrador de reparaciones agregando la sección `addonFeatures` después de la sección `fabricSettings`, tal y como se muestra a continuación:

    ```json
    "fabricSettings": [
        ...      
    ],
    "addonFeatures": [
        "RepairManager"
    ],
    ```

3. Actualice el manifiesto de clúster con estos cambios, con el manifiesto de clúster actualizado [crear un nuevo clúster](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-for-windows-server) o [actualizar la configuración del clúster](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-upgrade-windows-server). Una vez que el clúster se ejecute con el manifiesto del clúster actualizado, podrá ver el servicio de sistema de administrador de reparaciones en ejecución en el clúster, denominado `fabric:/System/RepairManagerService`, en la sección de servicios del sistema en Service Fabric Explorer.

### <a name="configure-windows-updates-for-all-nodes"></a>Configurar las actualizaciones de Windows para todos los nodos

Es posible que las actualizaciones automáticas de Windows provoquen la pérdida de disponibilidad, ya que varios nodos de clúster pueden reiniciarse al mismo tiempo. La aplicación de orquestación de revisiones intenta deshabilitar las actualizaciones automáticas de Windows en cada nodo del clúster de forma predeterminada. En caso de que la configuración se administre mediante directivas de administrador o de grupo, se recomienda configurar explícitamente la directiva de Windows Update en "Notificar antes de descargar".

## <a name="download-the-app-package"></a>Descargar el paquete de la aplicación

Se puede descargar la aplicación junto con los scripts de instalación desde el [vínculo de archivo](https://go.microsoft.com/fwlink/?linkid=869566).

Se puede descargar la aplicación en formato de sfpkg desde el [vínculo sfpkg](https://aka.ms/POA/POA.sfpkg). Esto resulta útil para [la implementación de aplicaciones basada en Azure Resource Manager](service-fabric-application-arm-resource.md).

## <a name="configure-the-app"></a>Configuración de la aplicación

El comportamiento de la aplicación de orquestación de revisiones puede configurarse según sus necesidades. Invalide los valores predeterminados al pasar el parámetro de la aplicación durante la creación o actualización de la aplicación. Los parámetros de la aplicación se pueden proporcionar especificando `ApplicationParameter` en los cmdlets `Start-ServiceFabricApplicationUpgrade` o `New-ServiceFabricApplication`.

|**Parámetro**        |**Tipo**                          | **Detalles**|
|:-|-|-|
|MaxResultsToCache    |long                              | Número máximo de resultados de Windows Update que deben almacenarse en caché. <br>El valor predeterminado es 3000 suponiendo que: <br> - El número de nodos es 20. <br> - El número de actualizaciones en un nodo al mes es cinco. <br> - El número de resultados por cada operación es 10. <br> - Deben almacenarse los resultados de los últimos tres meses. |
|TaskApprovalPolicy   |Enum <br> { NodeWise, UpgradeDomainWise }                          |TaskApprovalPolicy indica la directiva que usará Coordinator Service para instalar las actualizaciones de Windows en todos los nodos del clúster de Service Fabric.<br>                         Los valores permitidos son: <br>                                                           <b>NodeWise</b>. Windows Update se instala en un nodo cada vez. <br>                                                           <b>UpgradeDomainWise</b>. Windows Update se instala en un dominio de actualización cada vez. (Como máximo, todos los nodos que pertenecen a un dominio de actualización son aptos para Windows Update).<br> Consulte en la sección de [preguntas más frecuentes](#frequently-asked-questions) cómo decidir cuál es la mejor directiva para su clúster.
|LogsDiskQuotaInMB   |long  <br> (Valor predeterminado: 1024)               |Tamaño máximo de los registros de la aplicación de orquestación de revisiones en MB que se pueden almacenar de forma persistente y local en un nodo.
| WUQuery               | string<br>(Valor predeterminado: "IsInstalled=0")                | Consulta para obtener las actualizaciones de Windows. Para más información, vea [WuQuery](https://msdn.microsoft.com/library/windows/desktop/aa386526(v=vs.85).aspx).
| InstallWindowsOSOnlyUpdates | Boolean <br> (valor predeterminado: false)                 | Use esta marca para controlar qué actualizaciones deben descargarse e instalarse. Se permiten los siguientes valores <br>True: instala solo actualizaciones del sistema operativo Windows.<br>False: instala todas las actualizaciones disponibles en la máquina.          |
| WUOperationTimeOutInMinutes | Int <br>(Valor predeterminado: 90)                   | Especifica el tiempo de espera para cualquier operación de Windows Update (buscar, descargar o instalar). Si la operación no se realiza en el tiempo de espera especificado, se anula.       |
| WURescheduleCount     | Int <br> (Valor predeterminado: 5)                  | El número máximo de veces que el servicio vuelve a programar la actualización de Windows en caso de error de la operación de forma persistente.          |
| WURescheduleTimeInMinutes | Int <br>(Valor predeterminado: 30) | El intervalo en el que el servicio vuelve a programar la actualización de Windows en caso de que el error persista. |
| WUFrequency           | Cadena separada por comas (valor predeterminado: "Weekly, Wednesday, 7:00:00")     | Frecuencia para la instalación de actualizaciones de Windows. El formato y los valores posibles son: <br>- Monthly, DD,HH:MM:SS, por ejemplo, Monthly, 5,12:22:32.<br>Los valores permitidos para el campo DD (day) son números comprendidos entre 1 y 28, y "last". <br> - Weekly, DÍA,HH:MM:SS, por ejemplo, Weekly, martes, 12:22:32.  <br> -   Daily, HH:MM:SS, por ejemplo, Daily, 12:22:32.  <br> -  None indica que no debe realizarse Windows Update.  <br><br> Tenga en cuenta que las horas están en formato UTC.|
| AcceptWindowsUpdateEula | Boolean <br>(Valor predeterminado: true) | Al establecer esta marca, la aplicación acepta el contrato de licencia del usuario final para Windows Update en nombre del propietario del equipo.              |

> [!TIP]
> Si quiere que Windows Update se ejecute de forma inmediata, establezca `WUFrequency` en relación con la hora de implementación de la aplicación. Por ejemplo, suponga que tiene un clúster de prueba de cinco nodos y planea implementar la aplicación aproximadamente a las 17:00 UTC. Si supone que la implementación o actualización de la aplicación tarda 30 minutos como máximo, establezca WUFrequency como "Daily, 17:30:00".

## <a name="deploy-the-app"></a>Implementar la aplicación

1. Finalice todos los pasos de requisitos previos para preparar el clúster.
2. Implemente la aplicación de orquestación de revisiones como cualquier otra aplicación de Service Fabric. Puede implementar la aplicación mediante PowerShell. Siga los pasos descritos en [Implementación y eliminación de aplicaciones con PowerShell](https://docs.microsoft.com/azure/service-fabric/service-fabric-deploy-remove-applications).
3. Para configurar la aplicación en el momento de la implementación, pase el `ApplicationParameter` al cmdlet `New-ServiceFabricApplication`. Para su comodidad, hemos proporcionado el script Deploy.ps1 junto con la aplicación. Para usar el script:

    - Conéctese a un clúster de Service Fabric mediante `Connect-ServiceFabricCluster`.
    - Ejecute el script de PowerShell Deploy.ps1 con el valor adecuado de `ApplicationParameter`.

> [!NOTE]
> Mantenga el script y la carpeta PatchOrchestrationApplication de la aplicación en el mismo directorio.

## <a name="upgrade-the-app"></a>Actualizar la aplicación

Para actualizar una aplicación existente de orquestación de revisiones mediante PowerShell, siga los pasos descritos en [Actualización de aplicaciones de Service Fabric con PowerShell](https://docs.microsoft.com/azure/service-fabric/service-fabric-application-upgrade-tutorial-powershell).

## <a name="remove-the-app"></a>Quitar la aplicación

Para quitar la aplicación, siga los pasos descritos en [Implementación y eliminación de aplicaciones con PowerShell](https://docs.microsoft.com/azure/service-fabric/service-fabric-deploy-remove-applications).

Para su comodidad, hemos proporcionado el script Undeploy.ps1 junto con la aplicación. Para usar el script:

  - Conéctese a un clúster de Service Fabric mediante ```Connect-ServiceFabricCluster```.

  - Ejecute el script de PowerShell Undeploy.ps1.

> [!NOTE]
> Mantenga el script y la carpeta PatchOrchestrationApplication de la aplicación en el mismo directorio.

## <a name="view-the-windows-update-results"></a>Ver los resultados de Windows Update

La aplicación de orquestación de revisiones expone API de REST para mostrar los resultados históricos al usuario. Un ejemplo del resultado JSON:
```json
[
  {
    "NodeName": "_stg1vm_1",
    "WindowsUpdateOperationResults": [
      {
        "OperationResult": 0,
        "NodeName": "_stg1vm_1",
        "OperationTime": "2017-05-21T11:46:52.1953713Z",
        "UpdateDetails": [
          {
            "UpdateId": "7392acaf-6a85-427c-8a8d-058c25beb0d6",
            "Title": "Cumulative Security Update for Internet Explorer 11 for Windows Server 2012 R2 (KB3185319)",
            "Description": "A security issue has been identified in a Microsoft software product that could affect your system. You can help protect your system by installing this update from Microsoft. For a complete listing of the issues that are included in this update, see the associated Microsoft Knowledge Base article. After you install this update, you may have to restart your system.",
            "ResultCode": 0
          }
        ],
        "OperationType": 1,
        "WindowsUpdateQuery": "IsInstalled=0",
        "WindowsUpdateFrequency": "Daily,10:00:00",
        "RebootRequired": false
      }
    ]
  },
  ...
]
```

Los campos del archivo JSON se describen a continuación.

Campo | Valores | Detalles
-- | -- | --
Resultado de la operación | 0: se realizó correctamente<br> 1: se realizó correctamente con errores<br> 2: con error<br> 3: anulada<br> 4: anulada con tiempo de espera | Indica el resultado de la operación global (que normalmente implica la instalación de una o más actualizaciones).
ResultCode | Igual que OperationResult | Este campo indica el resultado de la operación de instalación para una actualización individual.
OperationType | 1: instalación<br> 0: buscar y descargar.| La instalación es el único valor de OperationType que se muestra en los resultados de forma predeterminada.
WindowsUpdateQuery | El valor predeterminado es "IsInstalled=0" |La consulta de Windows Update que se utilizó para buscar actualizaciones. Para más información, vea [WuQuery](https://msdn.microsoft.com/library/windows/desktop/aa386526(v=vs.85).aspx).
RebootRequired | true: se requiere reinicio<br> false: no se requiere reinicio | Indica si se requiere reiniciar para completar la instalación de actualizaciones.

Si todavía no hay ninguna actualización programada, el resultado JSON está vacío.

Inicie sesión en el clúster a Windows Update consulta los resultados. Después, obtenga la dirección de la réplica principal del servicio Coordinador y diríjase a la dirección URL desde el explorador: http://&lt;IP DE RÉPLICA&gt;:&lt;PuertoDeAplicación&gt;/PatchOrchestrationApplication/v1/GetWindowsUpdateResults.

El punto de conexión REST para el servicio Coordinator Service tiene un puerto dinámico. Para comprobar la dirección URL exacta, consulte Service Fabric Explorer. Por ejemplo, los resultados están disponibles en `http://10.0.0.7:20000/PatchOrchestrationApplication/v1/GetWindowsUpdateResults`.

![Imagen de punto de conexión REST](media/service-fabric-patch-orchestration-application/Rest_Endpoint.png)


Si el proxy inverso está habilitado en el clúster, también puede tener acceso a la dirección URL desde fuera del clúster.
El punto de conexión al que debe dirigirse es: http://&lt;URL_SERVIDOR&gt;:&lt;PUERTO_PROXY_INVERSO&gt;/PatchOrchestrationApplication/CoordinatorService/v1/GetWindowsUpdateResults.

Para habilitar el proxy inverso en el clúster, siga los pasos descritos en [Proxy inverso en Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-reverseproxy). 

> 
> [!WARNING]
> Después de configurar el proxy inverso, se podrá acceder de manera externa a todos los microservicios del clúster que exponen un punto de conexión HTTP.

## <a name="diagnosticshealth-events"></a>Eventos de diagnóstico y mantenimiento

### <a name="diagnostic-logs"></a>Registros de diagnóstico

Los registros de aplicación de orquestación de revisiones se recopilan como parte de los registros en tiempo de ejecución de Service Fabric.

En caso de que desee capturar los registros a través de la canalización o herramienta de diagnóstico de su elección. Aplicación de orquestación de revisiones usa los identificadores de proveedor fijos siguientes para registrar eventos a través de [origen del evento](https://docs.microsoft.com/dotnet/api/system.diagnostics.tracing.eventsource?view=netframework-4.5.1)

- e39b723c-590c-4090-abb0-11e3e6616346
- fc0028ff-bfdc-499f-80dc-ed922c52c5e9
- 24afa313-0d3b-4c7c-b485-1047fd964b60
- 05dc046c-60e9-4ef7-965e-91660adffa68

### <a name="health-reports"></a>Informes de mantenimiento

La aplicación de orquestación de revisiones también publica los informes de estado en el servicio Coordinator Service o el servicio Node Agent Service en los siguientes casos:

#### <a name="a-windows-update-operation-failed"></a>Error en la operación de Windows Update

Si se produce un error en la operación de Windows Update en un nodo, se generará un informe de estado en el servicio Node Agent Service. Los detalles del informe de estado contienen el nombre del nodo problemático.

Después de aplicar correctamente la revisión en el nodo problemático, el informe se borra automáticamente.

#### <a name="the-node-agent-ntservice-is-down"></a>El servicio Node Agent Service está inactivo

Si el servicio Node Agent NTService está inactivo en un nodo, se genera un informe de estado de nivel de advertencia en el servicio Node Agent Service.

#### <a name="the-repair-manager-service-is-not-enabled"></a>El servicio de administrador de reparaciones no está habilitado

Si el servicio de administrador de reparaciones no se encuentra en el clúster, se genera un informe de estado de nivel de advertencia para el servicio Coordinator Service.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

P: **¿Por qué veo el clúster en estado de error cuando se ejecuta la aplicación de orquestación de revisiones?**

A. Durante el proceso de instalación, la aplicación de orquestación de revisiones deshabilita o reinicia nodos, lo que puede dar lugar temporalmente a un estado de no disponibilidad del clúster.

En función de la directiva de la aplicación, un nodo puede dejar de estar activo durante una operación de aplicación de revisiones, *o bien* un dominio de actualización completo puede dejar de estar activo.

Al final de la instalación de Windows Update, los nodos se vuelven a habilitar después del reinicio.

En el ejemplo siguiente, el clúster entró en estado de error temporalmente porque dos nodos dejaron de estar activos y se infringió la directiva MaxPercentageUnhealthyNodes. El error es temporal hasta que la operación de aplicación de revisiones finalice.

![Imagen de clúster en mal estado](media/service-fabric-patch-orchestration-application/MaxPercentage_causing_unhealthy_cluster.png)

Si el problema persiste, consulte la sección de solución de problemas.

P: **La aplicación de orquestación de revisiones está en estado de advertencia**

A. Compruebe si un informe de estado publicado a la aplicación es la causa principal. Normalmente la advertencia contiene los detalles del problema. Si el problema es transitorio, se espera que la aplicación se recupere automáticamente de este estado.

P: **¿Qué puedo hacer si el clúster está en mal estado y tengo que realizar una actualización urgente del sistema operativo?**

A. La aplicación de orquestación de revisiones no instala actualizaciones cuando el clúster está en mal estado. Intente devolver el clúster a un estado correcto para desbloquear el flujo de trabajo de la aplicación de orquestación de revisiones.

P: **¿Debo establecer TaskApprovalPolicy como 'NodeWise' o 'UpgradeDomainWise' para el clúster?**

A. "UpgradeDomainWise" agiliza la aplicación general de revisiones al clúster, ya que aplica revisiones a todos los nodos que pertenecen a un dominio de actualización en paralelo. Esto significa que los nodos que pertenecen a un dominio de actualización completo no estarían disponibles (en estado [deshabilitado](https://docs.microsoft.com/dotnet/api/system.fabric.query.nodestatus?view=azure-dotnet#System_Fabric_Query_NodeStatus_Disabled)) durante el proceso de aplicación de revisiones.

En cambio, "NodeWise" aplica las revisiones de una en una, lo que implica que la aplicación de revisiones de clúster general podría tardar más tiempo. Sin embargo, como máximo, solo habría un nodo no disponible (en estado [deshabilitado](https://docs.microsoft.com/dotnet/api/system.fabric.query.nodestatus?view=azure-dotnet#System_Fabric_Query_NodeStatus_Disabled)) durante el proceso de aplicación de revisiones.

Si el clúster puede tolerar la ejecución en un número N-1 de dominios de actualización durante el ciclo de aplicación de revisiones (donde N es el número total de dominios de actualización del clúster), puede establecer la directiva como "UpgradeDomainWise". En caso contrario, establézcala en "NodeWise".

P: **¿Cuánto se tarda en aplicar revisiones a un nodo?**

A. La aplicación de revisiones a un nodo puede tardar minutos (por ejemplo: [Actualizaciones de definiciones de Windows Defender](https://www.microsoft.com/en-us/wdsi/definitions)) a horas (por ejemplo: [Actualizaciones acumulativas de Windows](https://www.catalog.update.microsoft.com/Search.aspx?q=windows%20server%20cumulative%20update)). El tiempo necesario para aplicar revisiones en un nodo depende, principalmente, de 
 - El tamaño de las actualizaciones
 - El número de actualizaciones, que deben aplicarse en un periodo de revisión
 - El tiempo necesario para instalar las actualizaciones, reiniciar el nodo (si es necesario) y completar los pasos de instalación posteriores al reinicio
 - El rendimiento de la máquina o VM y las condiciones de red

P: **¿Cuánto se tarda en aplicar revisiones a todo un clúster?**

A. El tiempo necesario para aplicar revisiones a todo un clúster depende de los siguientes factores:

- Tiempo necesario para aplicar revisiones a un nodo.
- La directiva del servicio Coordinator Service. La directiva predeterminada, `NodeWise`, provoca que se apliquen las revisiones en un nodo a la vez, lo que puede tardar más que `UpgradeDomainWise`. Por ejemplo: si la aplicación de revisiones a un nodo tarda, aproximadamente, una hora, para aplicar revisiones en un clúster de 20 nodos (mismo tipo de nodos) con 5 dominios de actualización que contienen, cada uno, 4 nodos,
    - se tardaría, aproximadamente, 20 horas en aplicar revisiones a todo el clúster, si la directiva es `NodeWise`
    - Si la directiva es `UpgradeDomainWise`, se tardarían unas cinco horas
- Carga de clúster: cada operación de revisión requiere cambiar la ubicación de la carga de trabajo del cliente a otros nodos disponibles en el clúster. El nodo al que se aplicaría la revisión tendría el estado [deshabilitado](https://docs.microsoft.com/dotnet/api/system.fabric.query.nodestatus?view=azure-dotnet#System_Fabric_Query_NodeStatus_Disabling) durante este tiempo. Si el clúster se ejecuta cerca de la carga máxima, el proceso de deshabilitación podría tardar más. Por lo tanto, el proceso general de aplicación de revisiones puede parecer lento en estas condiciones.
- Errores de estado del clúster durante la aplicación de revisiones: cualquier [degradación](https://docs.microsoft.com/dotnet/api/system.fabric.health.healthstate?view=azure-dotnet#System_Fabric_Health_HealthState_Error) en el [estado del clúster](https://docs.microsoft.com/azure/service-fabric/service-fabric-health-introduction) interrumpiría el proceso de revisión. Esto se agregaría al tiempo general necesario para aplicar revisiones a todo el clúster.

P: **¿Por qué veo algunas actualizaciones en los resultados de Windows Update obtenidos a través de la API de REST, pero no en el historial de Windows Update en la máquina?**

A. Algunas actualizaciones del producto solo aparecen en su historial de actualizaciones o revisiones respectivo. Por ejemplo, puede que se muestren o que no se muestren las actualizaciones de Windows Defender en el historial de Windows Update en Windows Server 2016.

P: **¿La aplicación Patch Orchestration se puede utilizar para aplicar revisiones al clúster de desarrollo (clúster con un solo nodo)?**

A. No, la aplicación Patch Orchestration no se puede utilizar para aplicar revisiones a clústeres de un nodo. Esta limitación es así por naturaleza, ya que los [servicios del sistema de Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-technical-overview#system-services) o las aplicaciones del cliente van a registrar tiempos de inactividad y, por tanto, cualquier trabajo de reparación que se realice durante la aplicación de revisiones nunca conseguirá la aprobación del administrador de reparaciones.

P: **¿Cómo se pueden revisar nodos del clúster en Linux?**

A. Consulte [actualizaciones automáticas de la imagen de sistema operativo del conjunto de escalado de máquina virtual de Azure](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade) para orquestar actualizaciones en linux.

## <a name="disclaimers"></a>Declinación de responsabilidades

- La aplicación de orquestación de revisiones acepta el acuerdo de licencia del usuario final de Windows Update en nombre del usuario. Opcionalmente se puede desactivar la configuración en la configuración de la aplicación.

- La aplicación de orquestación de revisiones recopila datos de telemetría para realizar el seguimiento de uso y rendimiento. Los datos de telemetría de la aplicación siguen la configuración de telemetría del runtime de Service Fabric (que se activa de forma predeterminada).

## <a name="troubleshooting"></a>solución de problemas

### <a name="a-node-is-not-coming-back-to-up-state"></a>El nodo no vuelve a su estado activo

**Es posible que el nodo esté bloqueado en un estado deshabilitado porque**:

Hay una comprobación de seguridad pendiente. Para solucionar este problema, asegúrese de que hay suficientes nodos disponibles en estado correcto.

**Es posible que el nodo esté bloqueado en un estado deshabilitado porque**:

- El nodo se ha deshabilitado manualmente.
- El nodo se ha deshabilitado debido a un trabajo de infraestructura de Azure en curso.
- El nodo ha sido deshabilitado temporalmente por la aplicación de orquestación de revisiones para aplicar revisiones en el nodo.

**Es posible que el nodo esté bloqueado en un estado inactivo porque**:

- El nodo se ha desactivado manualmente.
- Se está llevando a cabo un reinicio en el nodo (que es posible que haya iniciado la aplicación de orquestación de revisiones).
- El nodo está inactivo por un problema en la máquina virtual o por problemas de conectividad de red.

### <a name="updates-were-skipped-on-some-nodes"></a>Las actualizaciones se omitieron en algunos nodos

La aplicación de orquestación de revisiones intenta instalar las actualizaciones de Windows según la directiva de reprogramación. El servicio puede intentar recuperar el nodo y omitir la actualización en función de la directiva de la aplicación.

En ese caso, se genera un informe de estado de nivel de advertencia en el servicio Node Agent Service. El resultado de Windows Update también contiene el motivo posible del error.

### <a name="the-health-of-the-cluster-goes-to-error-while-the-update-installs"></a>El estado del clúster es de error mientras se instala la actualización

Una actualización de Windows fallida puede desactivar el estado de una aplicación o un clúster en un nodo concreto o un dominio de actualización. La aplicación de orquestación de revisiones interrumpe las operaciones posteriores de Windows Update hasta que el estado del clúster vuelva a ser correcto.

Un administrador debe intervenir y determinar por qué la aplicación o el clúster pasó a ser incorrecto debido a Windows Update.

## <a name="release-notes"></a>Notas de la versión

### <a name="version-110"></a>Versión 1.1.0
- Versión pública

### <a name="version-111"></a>Versión 1.1.1
- Se ha corregido un error en SetupEntryPoint de NodeAgentService que impedía la instalación de NodeAgentNTService.

### <a name="version-120"></a>Versión 1.2.0

- Correcciones de errores en el flujo de trabajo de reinicio del sistema.
- Corrección de errores en la creación de tareas de RM debido a las cuales la comprobación del estado durante la preparación de las tareas de reparación no ocurría según lo previsto.
- Cambio en el modo de inicio del servicio de Windows POANodeSvc de automático a automático con retraso.

### <a name="version-121"></a>Versión 1.2.1

- Corrija los errores en el flujo de trabajo de reducción vertical de clúster. Introdujo la lógica de recolección de elementos no utilizados para las tareas de reparación POA pertenecientes a nodos inexistentes.

### <a name="version-122"></a>Versión 1.2.2

- Correcciones de errores varios.
- Ahora se firman los archivos binarios.
- Vínculo sfpkg agregado para la aplicación.

### <a name="version-130"></a>Versión 1.3.0

- Si establece InstallWindowsOSOnlyUpdates en false se instalarán todas las actualizaciones disponibles.
- Se ha cambiado la lógica de deshabilitación de las actualizaciones automáticas. Esto permite solucionar un problema por el que las actualizaciones automáticas no se deshabilitaban en Server 2016 o versiones posteriores.
- Restricción de colocación con parámetros para ambos de los microservicios de POA para casos de uso avanzadas.

### <a name="version-131"></a>Versión 1.3.1
- Corrección de una regresión en la que POA 1.3.0 no funcionará en Windows Server 2012 R2 o versión anterior debido a un error al deshabilitar las actualizaciones automáticas. 
- Corrección de un error donde la configuración InstallWindowsOSOnlyUpdates siempre se elige como True.
- Cambio del valor predeterminado de InstallWindowsOSOnlyUpdates a False.

### <a name="version-132"></a>Versión 1.3.2
- Corregir un problema que afecta el ciclo de vida de aplicación de revisiones en un nodo en caso de que hay nodos con nombre que es el subconjunto del nombre de nodo actual. Para esos nodos, es posible, que falte la aplicación de revisiones o que el reinicio esté pendiente. 
