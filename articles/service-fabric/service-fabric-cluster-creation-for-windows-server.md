---
title: Creación de un clúster de Azure Service Fabric independiente
description: Cree un clúster de Azure Service Fabric en cualquier máquina (física o virtual) que ejecute Windows Server, ya sea local o en una nube.
author: dkkapur
ms.topic: conceptual
ms.date: 2/21/2019
ms.author: dekapur
ms.openlocfilehash: 6fa27008ea22e1a2bd9a83ce3888370cf2213935
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87458083"
---
# <a name="create-a-standalone-cluster-running-on-windows-server"></a>Creación de un clúster independiente con Windows Server
Puede usar Azure Service Fabric para crear clústeres de Service Fabric en las máquinas virtuales o los equipos que ejecutan Windows Server. Es decir, podrá implementar y ejecutar aplicaciones de Service Fabric en cualquier entorno donde haya un conjunto de equipos con Windows Server que estén conectados entre sí, ya sea de manera local o con algún proveedor de servicios en la nube. Service Fabric proporciona un paquete de instalación para crear clústeres de Service Fabric, llamado paquete independiente de Windows Server. Los clústeres tradicionales de Service Fabric en Azure están disponibles como un servicio administrado, mientras que los clústeres independientes son de autoservicio. Para más información sobre las diferencias, consulte [Comparación de clústeres de Service Fabric de Azure e independientes](./service-fabric-deploy-anywhere.md).

Este artículo le guía por los pasos para crear un clúster independiente de Service Fabric.

> [!NOTE]
> Este paquete de Windows Server independiente está disponible de forma gratuita en el mercado y puede usarse para las implementaciones de producción. Además, puede contener nuevas características de Service Fabric que están en versión preliminar. Desplácese hacia abajo hasta "[Características de versión preliminar incluidas en este paquete](#previewfeatures_anchor)". Sección de la lista de las características de versión preliminar. Puede [descargar una copia de los términos de licencia](https://go.microsoft.com/fwlink/?LinkID=733084) ahora.
> 
> 

<a id="getsupport"></a>

## <a name="get-support-for-the-service-fabric-for-windows-server-package"></a>Soporte técnico para el paquete de Service Fabric para Windows Server
* Pregunte a la comunidad sobre el paquete independiente de Service Fabric para Windows Server en la [página de preguntas y respuestas de Microsoft sobre Azure Service Fabric](/answers/topics/azure-service-fabric.html).
* Abra una incidencia para obtener [soporte técnico profesional para Service Fabric](https://support.microsoft.com/oas/default.aspx?prid=16146).  Más información sobre el soporte técnico profesional de Microsoft[aquí](https://support.microsoft.com/en-us/gp/offerprophone?wa=wsignin1.0).
* También puede obtener soporte técnico para este paquete como parte del [soporte técnico Premier de Microsoft](https://support.microsoft.com/en-us/premier).
* Para más información, consulte [Opciones de soporte técnico de Azure Service Fabric](./service-fabric-support.md).
* Para recopilar registros como soporte técnico, ejecute el [recopilador de registros independiente de Service Fabric](service-fabric-cluster-standalone-package-contents.md).

<a id="downloadpackage"></a>

## <a name="download-the-service-fabric-for-windows-server-package"></a>Descarga del paquete de Service Fabric para Windows Server
Para crear el clúster, use el paquete de Service Fabric para Windows Server (versión 2012 R2 y posteriores) que se encuentra aquí: <br>
[Vínculo de descarga del paquete independiente de Service Fabric de Windows Server](https://go.microsoft.com/fwlink/?LinkId=730690)

Puede encontrar detalles sobre el contenido del paquete [aquí](service-fabric-cluster-standalone-package-contents.md).

El paquete en tiempo de ejecución de Service Fabric se descarga automáticamente en el momento de la creación del clúster. Si la implementación se realiza desde una máquina no está conectada a Internet, descargue el paquete en tiempo de ejecución fuera de banda desde aquí: <br>
[Vínculo de descarga del entorno de tiempo de ejecución de Service Fabric para Windows Server](https://go.microsoft.com/fwlink/?linkid=839354)

Puede encontrar ejemplos de configuración de clúster independiente en: <br>
[Ejemplos de configuración de clúster independiente](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)

<a id="createcluster"></a>

## <a name="create-the-cluster"></a>Creación del clúster
Se instalan varios archivos de configuración de clúster de ejemplo con el paquete de instalación. *ClusterConfig.Unsecure.DevCluster.json* es la configuración de clúster más sencilla: un clúster no seguro de tres nodos que se ejecuta en un único equipo.  Otros archivos de configuración describen clústeres de una o varias máquinas con certificados X.509 o seguridad de Windows.  No es necesario modificar los valores de configuración predeterminados para este tutorial, pero eche un vistazo al archivo de configuración y familiarícese con la configuración.  En la sección **nodos** se describen los tres nodos del clúster: nombre, dirección IP, [tipo de nodo, dominio de error y dominio de actualización](service-fabric-cluster-manifest.md#nodes-on-the-cluster).  En la sección **propiedades** se define la [seguridad, el nivel de confiabilidad, la recopilación de diagnósticos y los tipos de nodos](service-fabric-cluster-manifest.md#cluster-properties) para el clúster.

El clúster creado en este artículo no es seguro.  Cualquiera puede conectarse de forma anónima y realizar operaciones de administración, por lo que los clústeres de producción siempre deben protegerse mediante certificados X.509 o la seguridad de Windows.  La configuración solo se configura en el momento de creación del clúster y no es posible habilitar la seguridad una vez creado. Al actualizar el archivo de configuración, se habilita la [seguridad basada en certificados](service-fabric-windows-cluster-x509-security.md) o la [seguridad de Windows](service-fabric-windows-cluster-windows-security.md). Lea [Protección de un clúster](service-fabric-cluster-security.md) para más información sobre la seguridad en los clústeres de Service Fabric.

### <a name="step-1-create-the-cluster"></a>Paso 1: Creación del clúster

#### <a name="scenario-a-create-an-unsecured-local-development-cluster"></a>Escenario A: Creación de un clúster de desarrollo local poco seguro
Service Fabric se puede implementar en un clúster de desarrollo de una máquina mediante el archivo *ClusterConfig.Unsecure.DevCluster.json* que se incluye en los [ejemplos](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples).

Desempaquete el paquete independiente en su máquina, copie el archivo de configuración de ejemplo en la máquina local y ejecute luego el script *CreateServiceFabricCluster.ps1* mediante una sesión de administrador de PowerShell, desde la carpeta del paquete independiente.

```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json -AcceptEULA
```

Para detalles de solución de problemas, consulte la sección Configuración del entorno del artículo sobre el [planeamiento la preparación de la implementación del clúster](service-fabric-cluster-standalone-deployment-preparation.md).

Si ya ha terminado de ejecutar los escenarios de desarrollo, puede quitar el clúster de Service Fabric de la máquina; para ello, consulte los pasos de la sección "[Eliminación de un clúster](#removecluster_anchor)". 

#### <a name="scenario-b-create-a-multi-machine-cluster"></a>Escenario B: Creación de un clúster de varias máquinas
Después de haber realizado los pasos de planeamiento y preparación detallados en [Planeamiento y preparación de la implementación de clústeres](service-fabric-cluster-standalone-deployment-preparation.md), está listo para crear el clúster de producción mediante el archivo de configuración del clúster.

El administrador de clúster que implemente y configure el clúster debe tener privilegios de administrador en el equipo. No se puede instalar Service Fabric en un controlador de dominio.

1. El script *TestConfiguration.ps1* del paquete independiente se usa como analizador de procedimientos recomendados para validar si se puede implementar un clúster en un entorno determinado. En la [preparación para la implementación](service-fabric-cluster-standalone-deployment-preparation.md) se enumeran los requisitos previos y los requisitos del entorno. Ejecute el script para comprobar si puede crear el clúster de desarrollo:  

    ```powershell
    .\TestConfiguration.ps1 -ClusterConfigFilePath .\ClusterConfig.json
    ```

    Debería ver una salida similar a la siguiente. Si el campo inferior "Pasado" se devuelve como "True", se han superado las comprobaciones de integridad y el clúster parece ser implementable según la configuración de entrada.

    ```powershell
    Trace folder already exists. Traces will be written to existing trace folder: C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer\DeploymentTraces
    Running Best Practices Analyzer...
    Best Practices Analyzer completed successfully.
    
    LocalAdminPrivilege        : True
    IsJsonValid                : True
    IsCabValid                 : True
    RequiredPortsOpen          : True
    RemoteRegistryAvailable    : True
    FirewallAvailable          : True
    RpcCheckPassed             : True
    NoConflictingInstallations : True
    FabricInstallable          : True
    Passed                     : True
    ```

2. Cree el clúster:  ejecute el script *CreateServiceFabricCluster.ps1* para implementar el clúster de Service Fabric en todas las máquinas de la configuración. 
    ```powershell
    .\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.json -AcceptEULA
    ```

> [!NOTE]
> Se escriben seguimientos de implementación en la máquina virtual/máquina física en la que ejecutó el script CreateServiceFabricCluster.ps1 de PowerShell. Estos seguimientos se pueden encontrar en la subcarpeta DeploymentTraces, que se encuentra en el directorio desde el que se ejecutó el script. Para ver si Service Fabric se ha implementado correctamente en una máquina, busque los archivos instalados en el directorio FabricDataRoot, como se detalla en la sección FabricSettings del archivo de configuración del clúster (de forma predeterminada c:\ProgramData\SF). También, los procesos FabricHost.exe y Fabric.exe se pueden ver ejecutando el Administrador de tareas.
> 
> 

#### <a name="scenario-c-create-an-offline-internet-disconnected-cluster"></a>Escenario C: Creación de un clúster sin conexión (desconectado de Internet)
El paquete en tiempo de ejecución de Service Fabric se descarga automáticamente durante la creación del clúster. Al implementar un clúster en máquinas sin conexión a internet, debe descargar el paquete en tiempo de ejecución de Service Fabric por separado y proporcionar su ruta de acceso durante la creación del clúster.
El paquete en tiempo de ejecución se puede descargar por separado desde otra máquina conectada a internet, desde el [vínculo de descarga: Service Fabric en tiempo de ejecución para Windows Server](https://go.microsoft.com/fwlink/?linkid=839354). Copie el paquete en tiempo de ejecución donde vaya a implementar el clúster desconectado y cree el clúster mediante la ejecución de `CreateServiceFabricCluster.ps1` con el parámetro `-FabricRuntimePackagePath` incluido, tal y como se muestra en este ejemplo: 

```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.json -FabricRuntimePackagePath .\MicrosoftAzureServiceFabric.cab
```

*.\ClusterConfig.json* y *.\MicrosoftAzureServiceFabric.cab* son las rutas de acceso a la configuración del clúster y el archivo .cab en tiempo de ejecución, respectivamente.

### <a name="step-2-connect-to-the-cluster"></a>Paso 2: Conectarse al clúster
Conéctese al clúster para verificar que el clúster se está ejecutando y está disponible. El módulo ServiceFabric PowerShell está instalado con el paquete en tiempo de ejecución.  Puede conectarse al clúster desde uno de los nodos del clúster o desde un equipo remoto con el entorno de tiempo de ejecución de Service Fabric.  El cmdlet [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) establece una conexión con el clúster.

Para conectarse a un clúster no seguro, ejecute el siguiente comando de PowerShell:

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <*IPAddressofaMachine*>:<Client connection end point port>
```

Por ejemplo:
```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint 192.13.123.234:19000
```

Consulte [Conexión a un clúster seguro](service-fabric-connect-to-secure-cluster.md) para ver ejemplos de conexión a un clúster. Después de conectarse al clúster, use el cmdlet [Get-ServiceFabricNode](/powershell/module/servicefabric/get-servicefabricnode?view=azureservicefabricps) para mostrar una lista de nodos en el clúster e información sobre el estado de cada uno. **HealthState** debe ser *OK* para cada nodo.

```powershell
PS C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer> Get-ServiceFabricNode |Format-Table

NodeDeactivationInfo NodeName IpAddressOrFQDN NodeType  CodeVersion  ConfigVersion NodeStatus NodeUpTime NodeDownTime HealthState
-------------------- -------- --------------- --------  -----------  ------------- ---------- ---------- ------------ -----------
                     vm2      localhost       NodeType2 5.6.220.9494 0                     Up 00:03:38   00:00:00              OK
                     vm1      localhost       NodeType1 5.6.220.9494 0                     Up 00:03:38   00:00:00              OK
                     vm0      localhost       NodeType0 5.6.220.9494 0                     Up 00:02:43   00:00:00              OK
```

### <a name="step-3-visualize-the-cluster-using-service-fabric-explorer"></a>Paso 3: Visualización del clúster mediante Service Fabric Explorer
[Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) es una buena herramienta para visualizar el clúster y administrar aplicaciones.  Service Fabric Explorer es un servicio que se ejecuta en el clúster al que puede acceder navegando a `http://localhost:19080/Explorer` con un explorador.

El panel del clúster proporciona información general del clúster, incluido un resumen de la aplicación y del estado del nodo. En la vista de nodos se muestra el diseño físico del clúster. Para un nodo determinado, puede comprobar qué aplicaciones tienen el código implementado en ese nodo.

![Service Fabric Explorer][service-fabric-explorer]

## <a name="add-and-remove-nodes"></a>Agregar y quitar nodos
Puede agregar o quitar nodos del clúster de Service Fabric independiente a medida que cambien las necesidades empresariales. Lea [Incorporación o eliminación de nodos de un clúster de Service Fabric independiente](service-fabric-cluster-windows-server-add-remove-nodes.md) para obtener pasos detallados.

<a id="removecluster" name="removecluster_anchor"></a>
## <a name="remove-a-cluster"></a>Quitar un clúster
Para quitar un clúster, ejecute el script de PowerShell *RemoveServiceFabricCluster.ps1* desde la carpeta del paquete y proporcione la ruta al archivo de configuración JSON. También puede especificar una ubicación para el registro que se va a eliminar.

Este script puede ejecutarse en cualquier máquina que tenga acceso de administrador a todas las máquinas que se muestran como nodos en el archivo de configuración del clúster. La máquina donde se ejecuta este script no tiene que formar parte del clúster.

```powershell
# Removes Service Fabric from each machine in the configuration
.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.json -Force
```

```powershell
# Removes Service Fabric from the current machine
.\CleanFabric.ps1
```

<a id="telemetry"></a>

## <a name="telemetry-data-collected-and-how-to-opt-out-of-it"></a>Datos de telemetría recopilados y cómo omitirlos
De forma predeterminada, el producto recopila datos de telemetría sobre el uso de Service Fabric para mejorar el producto. El Analizador de procedimientos recomendados que se ejecuta como parte de la instalación, comprueba la conectividad a [https://vortex.data.microsoft.com/collect/v1](https://vortex.data.microsoft.com/collect/v1). Si no está accesible, el programa de instalación genera un error, a menos que decida no participar en la telemetría.

1. La canalización de telemetría intenta cargar los datos siguientes en [https://vortex.data.microsoft.com/collect/v1](https://vortex.data.microsoft.com/collect/v1) una vez al día. Es una carga de mejor esfuerzo y no tiene ningún impacto en la funcionalidad del clúster. Solo se envía la telemetría desde el nodo que ejecuta el administrador de conmutación por error principal. Ningún otro nodo envía telemetría.
2. La telemetría consta de lo siguiente:

* Número de servicios
* Número de ServiceTypes
* Número de aplicaciones
* Número de ApplicationUpgrades
* Número de FailoverUnits
* Número de InBuildFailoverUnits
* Número de UnhealthyFailoverUnits
* Número de réplicas
* Número de InBuildReplicas
* Número de StandByReplicas
* Número de OfflineReplicas
* CommonQueueLength
* QueryQueueLength
* FailoverUnitQueueLength
* CommitQueueLength
* Número de nodos
* IsContextComplete: True/False
* ClusterId: es un GUID generado aleatoriamente para cada clúster
* ServiceFabricVersion
* Dirección IP de la máquina virtual o la máquina desde la que se carga la telemetría

Para deshabilitar la telemetría, agregue lo siguiente al elemento *properties* en la configuración del clúster: *enableTelemetry: false*.

<a id="previewfeatures" name="previewfeatures_anchor"></a>

## <a name="preview-features-included-in-this-package"></a>Características de versión preliminar incluidas en este paquete
Ninguno.


> [!NOTE]
> Con la nueva [versión de GA del clúster independiente para Windows Server (versión 5.3.204.x)](https://azure.microsoft.com/blog/azure-service-fabric-for-windows-server-now-ga/), puede actualizar el clúster a versiones futuras, manual o automáticamente. Consulte el documento [Actualización del clúster de Service Fabric independiente en Windows Server](service-fabric-cluster-upgrade-windows-server.md) para obtener más información.
> 
> 

## <a name="next-steps"></a>Pasos siguientes
* [Implementación y eliminación de aplicaciones con PowerShell](service-fabric-deploy-remove-applications.md)
* [Opciones de configuración de clústeres de Windows independientes](service-fabric-cluster-manifest.md)
* [Incorporación o eliminación de nodos de un clúster de Service Fabric independiente](service-fabric-cluster-windows-server-add-remove-nodes.md)
* [Actualización de nodos de un clúster de Service Fabric independiente](service-fabric-cluster-upgrade-windows-server.md)
* [Creación de un clúster de Service Fabric independiente con máquinas virtuales de Azure con Windows](./service-fabric-cluster-creation-via-arm.md)
* [Proteger un clúster independiente en Windows mediante la seguridad de Windows](service-fabric-windows-cluster-windows-security.md)
* [Protección de un clúster de Windows independiente mediante certificados](service-fabric-windows-cluster-x509-security.md)

<!--Image references-->
[Trusted Zone]: ./media/service-fabric-cluster-creation-for-windows-server/TrustedZone.png
[service-fabric-explorer]: ./media/service-fabric-cluster-creation-for-windows-server/sfx.png
