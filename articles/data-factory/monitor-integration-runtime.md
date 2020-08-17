---
title: Supervisión de Integration Runtime en Azure Data Factory
description: Aprenda a supervisar los distintos tipos de instancia de Integration Runtime en Azure Data Factory.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 07/25/2020
author: djpmsft
ms.author: daperlov
manager: anandsub
ms.openlocfilehash: 14f9ab0b1c3b8b437e46a7b6a2d8b87f03442a02
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290548"
---
# <a name="monitor-an-integration-runtime-in-azure-data-factory"></a>Supervisión de Integration Runtime en Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]
  
**Integration Runtime** es la infraestructura de proceso que usa Azure Data Factory (ADF) para proporcionar varias funcionalidades de integración de datos en distintos entornos de red. Data Factory ofrece tres tipos de instancia de Integration Runtime:

- Tiempo de ejecución de integración de Azure
- Integration Runtime autohospedado
- Azure-SQL Server Integration Services (SSIS) Integration Runtime

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para obtener el estado de una instancia de Integration Runtime (IR), ejecute el siguiente comando de PowerShell: 

```powershell
Get-AzDataFactoryV2IntegrationRuntime -DataFactoryName MyDataFactory -ResourceGroupName MyResourceGroup -Name MyAzureIR -Status
``` 

El cmdlet devuelve información diferente para distintos tipos de instancia de Integration Runtime. En este artículo se explican las propiedades y los estados de cada tipo de instancia de Integration Runtime.  

## <a name="azure-integration-runtime"></a>Tiempo de ejecución de integración de Azure

El recurso de proceso para una instancia de Azure Integration Runtime se puede administrar completamente de manera elástica en Azure. En la tabla siguiente se proporcionan las descripciones de las propiedades que devuelve el comando **Get-AzDataFactoryV2IntegrationRuntime**:

### <a name="properties"></a>Propiedades

En la siguiente tabla se proporcionan las descripciones de las propiedades que devuelve el cmdlet para una instancia de Azure Integration Runtime:

| Propiedad | Descripción |
-------- | ------------- | 
| Nombre | Nombre de la instancia de Azure Integration Runtime. |  
| State | Estado de la instancia de Azure Integration Runtime. | 
| Location | Ubicación de la instancia de Azure Integration Runtime. Para más información sobre la ubicación de una instancia de Azure Integration Runtime, consulte la [introducción a Integration Runtime](concepts-integration-runtime.md). |
| DataFactoryName | Nombre de la instancia de Data Factory a la que pertenece la instancia de Azure Integration Runtime. | 
| ResourceGroupName | Nombre del grupo de recursos al que pertenece la instancia de Data Factory.  |
| Descripción | Descripción de la instancia de Integration Runtime.  |

### <a name="status"></a>Estado

En la tabla siguiente se proporcionan los estados posibles de una instancia de Azure Integration Runtime:

| Estado | Comentarios/Escenarios | 
| ------ | ------------------ |
| En línea | La instancia de Azure Integration Runtime está en línea y lista para su uso. | 
| Sin conexión | La instancia de Azure Integration Runtime está desconectada por un error interno. |

## <a name="self-hosted-integration-runtime"></a>Integration Runtime autohospedado

En la sección siguiente se proporcionan las descripciones de las propiedades que devuelve el cmdlet Get-AzDataFactoryV2IntegrationRuntime. 

> [!NOTE] 
> Las propiedades y estado devueltos contienen información sobre la instancia de Integration Runtime autohospedada y de los nodos en tiempo de ejecución.  

### <a name="properties"></a>Propiedades

En la tabla siguiente se proporcionan las descripciones de las propiedades de supervisión para **cada nodo**:

| Propiedad | Descripción | 
| -------- | ----------- | 
| Nombre | Nombre de la instancia de Integration Runtime autohospedada y sus nodos asociados. Un nodo es una máquina Windows local con la instancia de Integration Runtime autohospedada instalada. |  
| Estado | Estado de la instancia general de Integration Runtime autohospedada y de los nodos. Ejemplo: En línea, Sin conexión, Limitado, etc. Para información acerca de estos estados, consulte la siguiente sección. | 
| Versión | Versión de la instancia de Integration Runtime autohospedada y de los nodos. La versión de la instancia de Integration Runtime autohospedada se determina en función de la versión de la mayoría de los nodos del grupo. Si hay nodos con versiones diferentes en la configuración de la instancia de Integration Runtime autohospedada, solo los nodos con el mismo número de versión que la instancia lógica de Integration Runtime autohospedada funcionan correctamente. Los otros están en el modo limitado y deben actualizarse manualmente (solo en caso de que se produzca un error en la actualización automática). | 
| Memoria disponible | Memoria disponible en un nodo de una instancia de Integration Runtime autohospedada. Este valor es una instantánea casi en tiempo real. | 
| Uso de CPU | Uso de CPU de un nodo de una instancia de Integration Runtime autohospedada. Este valor es una instantánea casi en tiempo real. |
| Redes (Entrada/Salida) | Uso de las redes de un nodo de una instancia de Integration Runtime autohospedada. Este valor es una instantánea casi en tiempo real. | 
| Trabajos simultáneos (En ejecución / Límite) | **En ejecución**. Número de trabajos o tareas que se ejecutan en cada nodo. Este valor es una instantánea casi en tiempo real. <br/><br/>**Límite**. Límite significa el máximo número de trabajos simultáneos para cada nodo. Este valor se define basándose en el tamaño de la máquina. Puede aumentar el límite para escalar verticalmente la ejecución de trabajos simultáneos en escenarios avanzados, cuando se supera el tiempo de espera de las actividades, aunque la CPU, la memoria o la red estén infrautilizadas. Esta funcionalidad también está disponible con una instancia de Integration Runtime autohospedada de nodo único. |
| Role | En una instancia multinodo de Integration Runtime autohospedada hay dos tipos de roles: distribuidor y trabajo. Todos los nodos son trabajos, lo que significa que pueden usarse para ejecutar los trabajos. Hay solo un nodo distribuidor, que se usa para extraer los trabajos y las tareas de Cloud Services y enviarlos a los diferentes nodos de trabajo. El nodo distribuidor también es un nodo de trabajo. |

Algunos valores de las propiedades tienen más sentido cuando hay dos o más nodos en el entorno de ejecución de integración autohospedado (es decir, en un escenario de escalado horizontal).

#### <a name="concurrent-jobs-limit"></a>Límite de trabajos simultáneos

El valor predeterminado del límite de trabajos simultáneos se establece en función del tamaño de la máquina. Los factores utilizados para calcular este valor dependen de la cantidad de RAM y del número de núcleos de CPU del equipo. Por tanto, cuantos más núcleos y más memoria, mayor será el limite predeterminado de trabajos simultáneos.

Realice un escalado horizontal aumentando el número de nodos. Al aumentar el número de nodos, el límite de trabajos simultáneos es la suma de los valores límite de trabajos simultáneos de todos los nodos disponibles.  Por ejemplo, si un nodo le permite ejecutar un máximo de doce trabajos simultáneos, la incorporación de tres nodos más similares le permite ejecutar un máximo de 48 trabajos simultáneos (es decir, 4 x 12). Se recomienda aumentar el límite de trabajos simultáneos solo cuando vea un uso escaso de los recursos con los valores predeterminados en cada nodo.

El valor predeterminado calculado se puede invalidar en Azure Portal. Seleccione Autor > Conexiones > Runtimes de integración > Editar > Nodos > Modificar valor de trabajos simultáneos por nodo. También puede usar el comando [update-Azdatafactoryv2integrationruntimenode](https://docs.microsoft.com/powershell/module/az.datafactory/update-Azdatafactoryv2integrationruntimenode#examples) de PowerShell.
  
### <a name="status-per-node"></a>Estado (por nodo)

En la tabla siguiente se proporcionan los estados posibles de los nodos de una instancia de Integration Runtime autohospedada:

| Estado | Descripción |
| ------ | ------------------ | 
| En línea | Nodo conectado al servicio Data Factory. |
| Sin conexión | El nodo está sin conexión. |
| Actualizando | El nodo se está actualizando automáticamente. |
| Limitado | Debido a un problema de conectividad. Puede ser debido a un problema en el puerto HTTP 8050, a un problema de conectividad de Service Bus o a un problema de sincronización de credenciales. |
| Inactivo | El nodo tiene una configuración diferente de la configuración de la mayoría de los otros nodos. |

Un nodo puede estar inactivo cuando no se puede conectar a otros nodos.

### <a name="status-overall-self-hosted-integration-runtime"></a>Estado (instancia general de Integration Runtime autohospedada)

En la tabla siguiente se proporcionan los estados posibles de una instancia de Integration Runtime autohospedada. Este estado depende de los de todos los nodos que pertenecen a la instancia de Runtime. 

| Estado | Descripción |
| ------ | ----------- | 
| Need registration (Se necesita registro) | No hay nodos registrados para esta instancia de Integration Runtime autohospedada aún. |
| En línea | Todos los nodos están en línea. |
| Sin conexión | Ningún nodo está en línea. |
| Limitado | No todos los nodos de esta instancia de Integration Runtime autohospedada tienen un estado correcto. Este estado es una advertencia de que alguno de los nodos podría estar inactivo. Puede deberse a un problema de sincronización de credenciales en el nodo distribuidor o de trabajo. |

Use el cmdlet **Get-AzDataFactoryV2IntegrationRuntimeMetric** para capturar la carga de JSON que contiene las propiedades detalladas del entorno de ejecución de integración autohospedado y sus valores de instantánea durante el tiempo de ejecución del cmdlet.

```powershell
Get-AzDataFactoryV2IntegrationRuntimeMetric -name $integrationRuntimeName -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName | ConvertTo-Json 
```

Salida de ejemplo (se supone que hay dos nodos asociados a esta instancia de Integration Runtime autohospedada):

```json
{
    "IntegrationRuntimeName":  "<Name of your integration runtime>",
    "ResourceGroupName":  "<Resource Group Name>",
    "DataFactoryName":  "<Data Factory Name>",
    "Nodes":  [
        {
            "NodeName":  "<Node Name>",
            "AvailableMemoryInMB":  <Value>,
            "CpuUtilization":  <Value>,
            "ConcurrentJobsLimit":  <Value>,
            "ConcurrentJobsRunning":  <Value>,
            "MaxConcurrentJobs":  <Value>,
            "SentBytes":  <Value>,
            "ReceivedBytes":  <Value>
        },
        {
            "NodeName":  "<Node Name>",
            "AvailableMemoryInMB":  <Value>,
            "CpuUtilization":  <Value>,
            "ConcurrentJobsLimit":  <Value>,
            "ConcurrentJobsRunning":  <Value>,
            "MaxConcurrentJobs":  <Value>,
            "SentBytes":  <Value>,
            "ReceivedBytes":  <Value>
        }

    ]
} 
```

## <a name="azure-ssis-integration-runtime"></a>Integration Runtime de SSIS de Azure

Azure-SSIS IR es un clúster totalmente administrado de máquinas virtuales (o nodos) de Azure que se dedica a ejecutar los paquetes SSIS. Puede invocar las ejecuciones de paquetes SSIS en Azure-SSIS IR usando varios métodos; por ejemplo, a través de SQL Server Data Tools habilitado para Azure (SSDT), la utilidad de línea de comandos AzureDTExec, T-SQL en SQL Server Management Studio (SSMS)/Agente SQL Server y ejecutando actividades de paquetes SSIS en canalizaciones de ADF. Azure-SSIS IR no ejecuta ninguna otra actividad de ADF. Una vez aprovisionado, puede supervisar sus propiedades globales/específicas del nodo y estados a través de Azure PowerShell, Azure Portal y Azure Monitor.

### <a name="monitor-the-azure-ssis-integration-runtime-with-azure-powershell"></a>Supervisión de Azure-SSIS Integration Runtime con Azure PowerShell

Use el siguiente cmdlet de Azure PowerShell para supervisar las propiedades globales/específicas del nodo y los estados de Azure-SSIS IR.

```powershell
Get-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -Status
```

#### <a name="properties"></a>Propiedades

En la siguiente tabla se proporcionan las descripciones de las propiedades que devuelve el cmdlet anterior para una instancia de Azure-SSIS IR.

| Propiedad/estado              | Descripción                  |
| ---------------------------- | ---------------------------- |
| CreateTime                   | Hora UTC en la que se creó la instancia de Azure-SSIS IR. |
| Nodos                        | Nodos asignados/disponibles de la instancia de Azure-SSIS IR con estados específicos de nodo (Iniciando/Disponible/Reciclando/No disponible) y errores que requieren acción. |
| OtherErrors                  | Errores inespecíficos del nodo que no requieren acción en la instancia de Azure-SSIS IR. |
| LastOperation                | Resultado de la última operación de inicio/detención en la instancia de Azure-SSIS IR con errores que requieren acción, en caso de error de esta. |
| State                        | Estado general (Inicial/Iniciando/Iniciado/Deteniendo/Detenido) de la instancia de Azure-SSIS IR. |
| Location                     | Ubicación de la instancia de Azure-SSIS IR. |
| NodeSize                     | Tamaño de cada nodo en la instancia de Azure-SSIS IR. |
| NodeCount                    | Número de nodos de la instancia de Azure-SSIS IR. |
| MaxParallelExecutionsPerNode | Número máximo de ejecuciones en paralelo por nodo en la instancia de Azure-SSIS IR. |
| CatalogServerEndpoint        | Punto de conexión del servidor o la instancia administrada existente de Azure SQL Database para hospedar el catálogo de SSIS (SSISDB). |
| CatalogAdminUserName         | Nombre de usuario administrador del servidor o la instancia administrada de Azure SQL Database. ADF utiliza esta información para preparar y administrar SSISDB en su nombre. |
| CatalogAdminPassword         | Contraseña del administrador del servidor o la instancia administrada de Azure SQL Database. |
| CatalogPricingTier           | Plan de tarifa de SSISDB hospedado en el servidor de Azure SQL Database.  No es aplicable a la instancia administrada de Azure SQL que hospeda SSISDB. |
| VNetId                       | Identificador del recurso de red virtual para que se una la instancia de Azure-SSIS IR. |
| Subnet                       | Nombre de la subred para que se una la instancia de Azure-SSIS IR. |
| ID                           | Identificador del recurso de la instancia de Azure-SSIS IR. |
| Tipo                         | Tipo de IR (administrado/autohospedado) de instancia de Azure-SSIS IR. |
| ResourceGroupName            | Nombre del grupo de recursos de Azure donde se crearon las instancias de ADF y Azure-SSIS IR. |
| DataFactoryName              | Nombre de la instancia de ADF. |
| Nombre                         | Nombre de la instancia de Azure-SSIS IR. |
| Descripción                  | Descripción de la instancia de Azure-SSIS IR. |
  
#### <a name="status-per-azure-ssis-ir-node"></a>Estado (por nodo de Azure-SSIS IR)

En la tabla siguiente se proporcionan los estados posibles de un nodo de Azure-SSIS IR:

| Estado específico del nodo | Descripción |
| -------------------- | ----------- | 
| Iniciando             | Este nodo se está preparando. |
| Disponible            | Este nodo está listo para la implementación y ejecución de paquetes SSIS. |
| Recycling            | Este nodo se va a reparar/reiniciar. |
| No disponible          | Este nodo no está listo para implementar ni ejecutar paquetes SSIS y tiene errores que requieren acción o problemas que se pueden resolver. |

#### <a name="status-overall-azure-ssis-ir"></a>Estado (general de Azure-SSIS IR)

En la tabla siguiente se proporcionan los estados generales posibles de una instancia de Azure-SSIS IR. El estado general, a su vez, depende de los estados combinados de todos los nodos que pertenecen a la instancia de Azure-SSIS IR. 

| Estado general | Descripción | 
| -------------- | ----------- | 
| Initial        | Los nodos de la instancia de Azure-SSIS IR no se han asignado/preparado. | 
| Iniciando       | Los nodos de la instancia de Azure-SSIS IR se están asignando/preparando y la facturación ha comenzado. |
| Iniciado        | Los nodos de la instancia de Azure-SSIS IR se han asignado/preparado y están listos para la implementación/ejecución de paquetes SSIS. |
| Deteniéndose       | Se están liberando los nodos de la instancia de Azure-SSIS IR. |
| Detenido        | Los nodos de la instancia de Azure-SSIS IR se han liberado y la facturación se ha detenido. |

### <a name="monitor-the-azure-ssis-integration-runtime-in-azure-portal"></a>Supervisión de Azure-SSIS Integration Runtime en Azure Portal

Para supervisar la instancia de Azure-SSIS IR en Azure Portal, vaya a la página **Entornos de ejecución de integración** del centro **Supervisión** en la interfaz de usuario de ADF, donde puede ver todos los entornos de ejecución de integración.

![Supervisión de todos los entornos de ejecución de integración](media/monitor-integration-runtime/monitor-integration-runtimes.png)

A continuación, seleccione el nombre de la instancia de Azure-SSIS IR para abrir su página de supervisión, donde puede ver sus propiedades globales/específicas del nodo y estados.

![Supervisión de la instancia de Azure-SSIS IR](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime.png)

En el icono **ESTADO** de la página de supervisión de Azure-SSIS IR, puede ver su estado general; por ejemplo, **En ejecución** o **Detenido**. Al seleccionar el estado **En ejecución**, se abre una ventana con un botón dinámico **Detener** para detener la instancia de Azure-SSIS IR. Al seleccionar el estado **Detenido**, se abre una ventana con un botón dinámico **Iniciar** para iniciar la instancia de Azure-SSIS IR. La ventana emergente también tiene un botón **Ejecutar paquete SSIS** para generar automáticamente una canalización de ADF con la actividad Ejecutar paquete SSIS que se ejecuta en la instancia de Azure-SSIS IR (vea [Ejecución de paquetes SSIS como actividades Ejecutar paquete SSIS en las canalizaciones de ADF](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity)) y un cuadro de texto de **Id. de recurso**, desde el que puede copiar el id. de recurso de Azure-SSIS IR (`/subscriptions/YourAzureSubscripton/resourcegroups/YourResourceGroup/providers/Microsoft.DataFactory/factories/YourADF/integrationruntimes/YourAzureSSISIR`) que se puede usar para adquirir componentes de SSIS prémium o con licencia adicionales de fabricantes de software independientes (ISV) y enlazarlos a su instancia de Azure-SSIS IR (consulte [Instalación de componentes prémium o con licencia en la instancia de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/how-to-develop-azure-ssis-ir-licensed-components)).

![Supervisión de la instancia de Azure-SSIS IR: icono ESTADO](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime-status.png)

Si usa el modelo de implementación de proyectos en el que los paquetes se almacenan en SSISDB hospedado en su servidor de Azure SQL Database o instancia administrada, verá el icono **SSISDB SERVER ENDPOINT** (PUNTO DE CONEXIÓN DEL SERVIDOR DE SSISDB) en la página de supervisión de Azure-SSIS IR (consulte [Configuración de implementación de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/tutorial-deploy-ssis-packages-azure#deployment-settings-page)). En este icono, puede seleccionar un vínculo que designe el servidor de Azure SQL Database o la instancia administrada para que se muestre una ventana, donde puede copiar el punto de conexión del servidor desde un cuadro de texto y usarlo al conectarse desde SSMS para implementar, configurar, ejecutar y administrar los paquetes. En la ventana emergente, también puede seleccionar el icono **See your Azure SQL Database or managed instance settings** (Ver la configuración de Azure SQL Database o la instancia administrada) para volver a configurar o cambiar el tamaño de SSISDB en Azure Portal.

![Supervisión de la instancia de Azure-SSIS IR: icono SSISDB](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime-ssisdb.png)

Si une la instancia de Azure-SSIS IR a una red virtual, verá el icono **VALIDATE VNET / SUBNET** (VALIDAR RED VIRTUAL/SUBRED) en la página de supervisión de Azure-SSIS IR (consulte [Unión de la instancia de Azure-SSIS IR a una red virtual](https://docs.microsoft.com/azure/data-factory/join-azure-ssis-integration-runtime-virtual-network)). En este icono, puede seleccionar un vínculo que designe la red virtual y la subred para que aparezca una ventana, donde puede copiar el identificador de recurso de red virtual (`/subscriptions/YourAzureSubscripton/resourceGroups/YourResourceGroup/providers/Microsoft.Network/virtualNetworks/YourARMVNet`) y el nombre de subred de los cuadros de texto, así como validar las configuraciones de red virtual y subred para asegurarse de que los tráficos de red entrantes y salientes necesarios y la administración de la instancia de Azure-SSIS IR no estén obstruidos.

![Supervisión de la instancia de Azure-SSIS IR: icono VALIDAR](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime-validate.png)

En el icono **DIAGNOSTICAR CONECTIVIDAD** de la página de supervisión de Azure-SSIS IR, puede seleccionar el vínculo **Probar conexión** para mostrar una ventana, donde puede comprobar las conexiones entre la instancia de Azure-SSIS IR y los almacenes de paquetes, configuraciones y datos relevantes, así como los servicios de administración, a través de su dirección de nombre de dominio completo (FQDN)/IP y el puerto designado (consulte [Prueba de conexiones desde la instancia de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/ssis-integration-runtime-diagnose-connectivity-faq)).

![Supervisión de la instancia de Azure-SSIS IR: icono DIAGNOSTICAR](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime-diagnose.png)

Si usa el modelo de implementación de paquetes en el que los paquetes se almacenan en el sistema de archivos/Azure Files/base de datos de SQL Server (MSDB) hospedado en Azure SQL Managed Instance y administrado a través de almacenes de paquetes de Azure-SSIS IR, verá el icono **PACKAGE STORES** (ALMACENES DE PAQUETES) en la página de supervisión de Azure-SSIS IR (consulte [Configuración de implementación de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/tutorial-deploy-ssis-packages-azure#deployment-settings-page)). En este icono, puede seleccionar un vínculo que designe el número de almacenes de paquetes adjuntos a su instancia de Azure-SSIS IR para que se muestre una ventana, donde podrá volver a configurar los servicios vinculados relevantes para sus almacenes de paquetes de Azure-SSIS IR en el sistema de archivos/Azure Files/MSDB hospedado por Azure SQL Managed Instance.

![Supervisión de la instancia de Azure-SSIS IR: icono PAQUETE](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime-package.png)

Si hay problemas con el inicio, la detención, el mantenimiento o la actualización de su instancia de Azure-SSIS IR, verá un icono **ERRORES** adicional en la página de supervisión de Azure-SSIS IR. En este icono, puede seleccionar un vínculo que designe el número de errores generados por la instancia de Azure-SSIS IR para que se muestre una ventana, donde puede ver los errores con más detalles y copiarlos para encontrar las soluciones recomendadas en nuestra guía de solución de problemas (consulte [Solución de problemas de su instancia de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/ssis-integration-runtime-management-troubleshoot)).

### <a name="monitor-the-azure-ssis-integration-runtime-with-azure-monitor"></a>Supervisión de Azure-SSIS Integration Runtime con Azure Monitor

Para supervisar la instancia de Azure-SSIS IR con Azure Monitor, consulte [Supervisión de operaciones de SSIS con Azure Monitor](https://docs.microsoft.com/azure/data-factory/monitor-using-azure-monitor#monitor-ssis-operations-with-azure-monitor).

### <a name="more-info-about-the-azure-ssis-integration-runtime"></a>Más información sobre Integration Runtime de SSIS de Azure

Consulte los artículos siguientes para más información sobre Integration Runtime de SSIS de Azure:

- [Integration Runtime de SSIS de Azure](concepts-integration-runtime.md#azure-ssis-integration-runtime). En este artículo se proporciona información conceptual acerca de Integration Runtime en general, lo que incluye Integration Runtime de SSIS de Azure. 
- [Tutorial: Implementación de paquetes SSIS en Azure](tutorial-create-azure-ssis-runtime-portal.md). En este artículo se proporcionan instrucciones paso a paso para crear una instancia de Azure-SSIS IR y se usa SQL Database para hospedar el catálogo de SSIS. 
- [Cómo: Creación de una instancia de Integration Runtime de SSIS de Azure](create-azure-ssis-integration-runtime.md). En este artículo se amplía el tutorial y se proporcionan instrucciones sobre el uso de Instancia administrada de SQL y la unión de Integration Runtime a una red virtual. 
- [Administración de Integration Runtime de SSIS de Azure](manage-azure-ssis-integration-runtime.md). En este artículo se muestra cómo detener, iniciar o quitar una instancia de IR de SSIS de Azure. También se muestra cómo escalar horizontalmente IR de SSIS de Azure mediante la adición de más nodos a IR. 
- [Unión de una instancia de Integration Runtime para la integración de SSIS en Azure a una red virtual](join-azure-ssis-integration-runtime-virtual-network.md). En este artículo se proporciona información conceptual sobre cómo unir una instancia de Integration Runtime de SSIS de Azure a una red virtual de Azure. También se proporcionan los pasos para configurar la red virtual mediante Azure Portal para que se una la instancia de Integration Runtime de SSIS de Azure. 

## <a name="next-steps"></a>Pasos siguientes
Consulte los siguientes artículos para la supervisión de las canalizaciones de maneras diferentes: 

- [Guía de inicio rápido:creación de una instancia de Data Factory](quickstart-create-data-factory-dot-net.md)
- [Uso de Azure Monitor de Azure para supervisar las canalizaciones Data Factory](monitor-using-azure-monitor.md)