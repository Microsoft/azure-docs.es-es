---
title: Implementación de Azure Monitor a escala mediante Azure Policy
description: Implemente características de Azure Monitor a escala mediante Azure Policy.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 06/08/2020
ms.openlocfilehash: a69a58da85cf1ee03046626bb076c5cd44196279
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87828717"
---
# <a name="deploy-azure-monitor-at-scale-using-azure-policy"></a>Implementación de Azure Monitor a escala mediante Azure Policy
Aunque algunas características de Azure Monitor se configuran una vez o un número limitado de veces, otras se deben repetir para cada recurso que desee supervisar. En este artículo se describen los métodos para usar Azure Policy para implementar Azure Monitor a escala con el fin de asegurarse de que la supervisión se configura de forma coherente y precisa para todos los recursos de Azure.

Por ejemplo, debe crear una configuración de diagnóstico para todos los recursos de Azure existentes y para cada recurso nuevo que cree. También debe tener un agente instalado y configurado cada vez que cree una máquina virtual. Puede usar métodos como PowerShell o la CLI para realizar estas acciones, ya que estos métodos están disponibles para todas las características de Azure Monitor. Con Azure Policy, puede tener en funcionamiento una lógica que realizará automáticamente la configuración adecuada cada vez que cree o modifique un recurso.


## <a name="azure-policy"></a>Azure Policy
En esta sección se proporciona una breve introducción a [Azure Policy](../governance/policy/overview.md), que le permite evaluar y aplicar los estándares de la organización en toda la suscripción de Azure o en un grupo de administración con un esfuerzo mínimo. Para más información, consulte la [documentación de Azure Policy](../governance/policy/overview.md).

Con Azure Policy puede especificar los requisitos de configuración para todos los recursos que se creen e identificar los recursos que no son compatibles, bloquear la creación de los recursos o agregar la configuración necesaria. Funciona mediante la interceptación de las llamadas para crear un nuevo recurso o modificar un recurso existente. Puede responder con efectos tales como denegar la solicitud si no coincide con las propiedades esperadas en una definición de directiva, marcarla como no compatible o implementar un recurso relacionado. Puede corregir los recursos existentes con una definición de directiva **deployIfNotExists** (implementar si no existe) o **modify** (modificar).

Azure Policy consta de los objetos de la tabla siguiente. Consulte [Objetos de Azure Policy](../governance/policy/overview.md#azure-policy-objects) para una explicación más detallada de cada uno.

| Elemento | Descripción |
|:---|:---|
| Definición de directiva | Describe las condiciones de cumplimiento de los recursos y el efecto que se debe realizar si se cumple una condición. Puede tratarse de todos los recursos de un tipo determinado o solo de aquellos recursos que coinciden con determinadas propiedades. El efecto puede ser simplemente marcar el recurso para el cumplimiento o implementar un recurso relacionado. Las definiciones de directiva se escriben mediante código JSON, tal como se describe en [Estructura de las definiciones de Azure Policy](../governance/policy/concepts/definition-structure.md). Los efectos se describen en [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md).
| Iniciativa de directiva | Grupo de definiciones de directiva que se deben aplicar juntas. Por ejemplo, podría tener una definición de directiva para enviar los registros de recursos a un área de trabajo de Log Analytics y otra para enviar los registros de recursos a Event hubs. Cree una iniciativa que incluya ambas definiciones de directiva y aplique la iniciativa a los recursos en lugar de a las definiciones de directiva individuales. Las iniciativas se escriben mediante código JSON, tal como se describe en [Estructura de las iniciativas de Azure Policy](../governance/policy/concepts/initiative-definition-structure.md). |
| Asignación | Una definición de directiva o una iniciativa no tienen efecto hasta que se asignan a un ámbito. Por ejemplo, asigne una directiva a un grupo de recursos para aplicarla a todos los recursos creados en ese recurso o aplíquela a una suscripción para aplicarla a todos los recursos de esa suscripción.  Para más información, consulte [Estructura de las asignaciones de Azure Policy](../governance/policy/concepts/assignment-structure.md). |

## <a name="built-in-policy-definitions-for-azure-monitor"></a>Definiciones de directivas integradas para Azure Monitor
Azure Policy incluye varias definiciones creadas previamente relacionadas con Azure Monitor. Puede asignar estas definiciones de directiva a la suscripción existente o utilizarlas como base para crear sus propias definiciones personalizadas. Para obtener una lista completa de la directivas integradas de la categoría **Supervisión**, consulte [Definiciones de directivas integradas de Azure Policy para Azure Monitor](samples/policy-samples.md).

Para ver las definiciones de directivas integradas relacionadas con la supervisión, realice lo siguiente:

1. Vaya a **Azure Policy** en Azure Portal.
2. Seleccione **Definiciones**.
3. En **Tipo**, seleccione *Integrada* y, en **Categoría**, seleccione *Supervisión*.

  ![Definiciones de directiva integradas](media/deploy-scale/builtin-policies.png)


## <a name="diagnostic-settings"></a>Configuración de diagnóstico
La [Configuración de diagnóstico](platform/diagnostic-settings.md) recopila registros de recursos y métricas de los recursos de Azure en varias ubicaciones, normalmente en un área de trabajo de Log Analytics, que permite analizar los datos con [consultas de registro](log-query/log-query-overview.md) y [alertas de registro](platform/alerts-log.md). Use Azure Policy para crear automáticamente una configuración de diagnóstico cada vez que cree un recurso.

Cada tipo de recurso de Azure tiene un conjunto único de categorías que se deben enumerar en la configuración de diagnóstico. Debido a esto, cada tipo de recurso requiere una definición de directiva independiente. Algunos tipos de recursos tienen definiciones de directivas integradas que se pueden asignar sin modificaciones. Para otros tipos de recursos, debe crear una definición personalizada.

### <a name="built-in-policy-definitions-for-azure-monitor"></a>Definiciones de directivas integradas para Azure Monitor
Hay dos definiciones de directivas integradas para cada tipo de recurso, una para enviar datos a un área de trabajo de Log Analytics y otra a Event Hubs. Si solo necesita una ubicación, asigne esa directiva al tipo de recurso. Si necesita ambas, asigne ambas definiciones de directiva al recurso.

Por ejemplo, la siguiente imagen muestra las definiciones de directiva de configuración de diagnóstico integradas para Data Lake Analytics.

  ![Definiciones de directiva integradas](media/deploy-scale/builtin-diagnostic-settings.png)

### <a name="custom-policy-definitions"></a>Definiciones de directivas personalizadas
En el caso de los tipos de recursos que no tienen una directiva integrada, debe crear una definición de directiva personalizada. Puede hacerlo manualmente en Azure Portal mediante la copia de una directiva integrada existente y, a continuación, su modificación para el tipo de recurso. No obstante, es más eficaz crear la directiva mediante programación con un script de la Galería de PowerShell.

El script [Create-AzDiagPolicy](https://www.powershellgallery.com/packages/Create-AzDiagPolicy) crea archivos de directivas para un tipo de recurso determinado que se pueden instalar mediante PowerShell o la CLI. Use el procedimiento siguiente para crear una definición de directiva personalizada para la configuración de diagnóstico.


1. Asegúrese de que tiene instalado [Azure PowerShell](/powershell/azure/install-az-ps).
2. Instale el script con el comando siguiente:
  
    ```azurepowershell
    Install-Script -Name Create-AzDiagPolicy
    ```

3. Ejecute el script con parámetros para especificar dónde enviar los registros. Se le pedirá que especifique una suscripción y un tipo de recurso. Por ejemplo, para crear una definición de directiva que envíe datos a un área de trabajo de Log Analytics y a Event Hubs, use el comando siguiente.

   ```azurepowershell
   Create-AzDiagPolicy.ps1 -ExportLA -ExportEH -ExportDir ".\PolicyFiles"  
   ```

4. Como alternativa, puede especificar una suscripción y un tipo de recurso en el comando. Por ejemplo, para crear una definición de directiva que envíe datos a un área de trabajo de Log Analytics y a Event Hubs para bases de datos de Azure SQL Server, use el comando siguiente.

   ```azurepowershell
   Create-AzDiagPolicy.ps1 -SubscriptionID xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -ResourceType Microsoft.Sql/servers/databases  -ExportLA -ExportEH -ExportDir ".\PolicyFiles"  
   ```

5. El script crea carpetas independientes para cada definición de directiva, cada una de las cuales contiene tres archivos llamados azurepolicy.json, azurepolicy.rules.json y azurepolicy.parameters.json. Si desea crear la directiva manualmente en Azure Portal, puede copiar y pegar el contenido del archivo azurepolicy.json, ya que incluye la definición de directiva completa. Use los otros dos archivos con PowerShell o la CLI para crear la definición de directiva desde la línea de comandos.

    En los siguientes ejemplos se muestra cómo instalar la definición de directiva desde PowerShell y la CLI. Cada uno incluye metadatos para especificar la categoría **Supervisión** para agrupar la nueva definición de directiva con las definiciones de directivas integradas.

      ```azurepowershell
      New-AzPolicyDefinition -name "Deploy Diagnostic Settings for SQL Server database to Log Analytics workspace" -policy .\Apply-Diag-Settings-LA-Microsoft.Sql-servers-databases\azurepolicy.rules.json -parameter .\Apply-Diag-Settings-LA-Microsoft.Sql-servers-databases\azurepolicy.parameters.json -mode All -Metadata '{"category":"Monitoring"}'
      ```

      ```azurecli
      az policy definition create --name 'deploy-diag-setting-sql-database--workspace' --display-name 'Deploy Diagnostic Settings for SQL Server database to Log Analytics workspace'  --rules 'Apply-Diag-Settings-LA-Microsoft.Sql-servers-databases\azurepolicy.rules.json' --params 'Apply-Diag-Settings-LA-Microsoft.Sql-servers-databases\azurepolicy.parameters.json' --subscription 'AzureMonitor_Docs' --mode All
      ```

### <a name="initiative"></a>Iniciativa
En lugar de crear una asignación para cada definición de directiva, una estrategia habitual consiste en crear una iniciativa que incluya las definiciones de directivas para crear la configuración de diagnóstico para cada servicio de Azure. Cree una asignación entre la iniciativa y un grupo de administración, una suscripción o un grupo de recursos, en función de cómo administre el entorno. Esta estrategia ofrece las siguientes ventajas:

- Cree una única asignación para la iniciativa en lugar de varias asignaciones para cada tipo de recurso. Use la misma iniciativa para varios grupos de supervisión, suscripciones o grupos de recursos.
- Modifique la iniciativa cuando necesite agregar un nuevo tipo de recurso o destino. Por ejemplo, los requisitos iniciales pueden ser enviar datos solo a un área de trabajo de Log Analytics, pero más adelante puede que desee agregar Event Hubs. Modifique la iniciativa en lugar de crear nuevas asignaciones.

Consulte [Creación y asignación de una definición de iniciativa](../governance/policy/tutorials/create-and-manage.md#create-and-assign-an-initiative-definition) para más información sobre cómo crear una iniciativa. Tenga en cuenta las recomendaciones siguientes:

- Establezca **Categoría** en **Supervisión** para agruparla con las definiciones de directivas integradas y personalizadas relacionadas.
- En lugar de especificar los detalles del área de trabajo de Log Analytics y Event Hubs para la definición de directiva que se incluye en la iniciativa, use un parámetro de iniciativa común. Esto le permite especificar fácilmente un valor común para todas las definiciones de directivas y cambiar ese valor si es necesario.

![Definición de iniciativa](media/deploy-scale/initiative-definition.png)

### <a name="assignment"></a>Asignación 
Asigne la iniciativa a un grupo de administración, una suscripción o un grupo de recursos de Azure en función del ámbito de los recursos que se van a supervisar. Un [grupo de administración](../governance/management-groups/overview.md) es especialmente útil para dar un ámbito a la directiva, especialmente si la organización tiene varias suscripciones.

![Asignación de iniciativa](media/deploy-scale/initiative-assignment.png)

Mediante el uso de parámetros de iniciativa, puede especificar el área de trabajo o cualquier otro detalle para todas las definiciones de directivas de la iniciativa. 

![Parámetros de iniciativa](media/deploy-scale/initiative-parameters.png)

### <a name="remediation"></a>Corrección
La iniciativa se aplicará a cada máquina virtual a medida que esta se cree. Una [tarea de corrección](../governance/policy/how-to/remediate-resources.md) implementa las definiciones de directivas de la iniciativa en los recursos existentes, de modo que esto le permite crear una configuración de diagnóstico para los recursos que ya se han creado. Al crear la asignación mediante Azure Portal, tiene la opción de crear una tarea de corrección al mismo tiempo. Consulte [Corrección de los recursos no compatibles con Azure Policy](../governance/policy/how-to/remediate-resources.md) para más información sobre la corrección.

![Corrección de la iniciativa](media/deploy-scale/initiative-remediation.png)


## <a name="azure-monitor-for-vms"></a>Azure Monitor para máquinas virtuales
[Azure Monitor para VM](insights/vminsights-overview.md) es la herramienta principal de Azure Monitor para supervisar máquinas virtuales. La habilitación de Azure Monitor para VM instala el agente de Log Analytics y el agente de dependencias. En lugar de realizar estas tareas manualmente, use Azure Policy para asegurarse de que todas las máquinas virtuales se configuran a medida que se crean.

> [!NOTE]
> Azure Monitor para VM incluye una característica denominada **Cobertura de directiva de Azure Monitor para VM** que permite detectar y corregir VM no compatibles en el entorno. Puede usar esta característica en lugar de trabajar directamente con Azure Policy para VM de Azure y para máquinas virtuales híbridas conectadas a Azure Arc. En el caso de los conjuntos de escalado de máquinas virtuales de Azure, debe crear la asignación mediante Azure Policy.
 

Azure Monitor para VM incluye las siguientes iniciativas integradas que instalan ambos agentes para habilitar la supervisión completa. 

|Nombre |Descripción |
|:---|:---|
|Habilitar Azure Monitor para VM | Instala el agente de Log Analytics y Dependency Agent en VM de Azure y VM híbridas conectadas a Azure Arc. |
|Habilitar Azure Monitor para conjunto de escalado de máquinas virtuales | Instala el agente de Log Analytics y Dependency Agent en el conjunto de escalado de máquinas virtuales de Azure. |


### <a name="virtual-machines"></a>Máquinas virtuales
En lugar de crear asignaciones para estas iniciativas mediante la interfaz de Azure Policy, Azure Monitor para VM incluye una característica que le permite inspeccionar el número de máquinas virtuales de cada ámbito para determinar si se ha aplicado la iniciativa. A continuación, puede configurar el área de trabajo y crear las asignaciones necesarias mediante esa interfaz.

Para más información sobre este proceso, consulte [Habilitación de Azure Monitor para VM mediante Azure Policy](./insights/vminsights-enable-policy.md).

![Directiva de Azure Monitor para VM](media/deploy-scale/vminsights-policy.png)

### <a name="virtual-machine-scale-sets"></a>Conjuntos de escalado de máquinas virtuales
Para usar Azure Policy para habilitar la supervisión de conjuntos de escalado de máquinas virtuales, asigne la iniciativa **Habilitar Azure Monitor para conjuntos de escalado de máquinas virtuales** a un grupo de administración, suscripción o grupo de recursos de Azure en función del ámbito de los recursos que se supervisarán. Un [grupo de administración](../governance/management-groups/overview.md) es especialmente útil para dar un ámbito a la directiva, especialmente si la organización tiene varias suscripciones.

![Asignación de iniciativa](media/deploy-scale/virtual-machine-scale-set-assign-initiative.png)

Seleccione el área de trabajo a la que se enviarán los datos. Esta área de trabajo debe tener instalada la solución *VMInsights*, como se describe en []().

![Selección del área de trabajo](media/deploy-scale/virtual-machine-scale-set-workspace.png)

Cree una tarea de corrección si tiene un conjunto de escalado de máquinas virtuales existente al que sea necesario asignar esta directiva.

![Tarea de corrección](media/deploy-scale/virtual-machine-scale-set-remediation.png)

### <a name="log-analytics-agent"></a>Agente de Log Analytics
Es posible que tenga escenarios en los que quiera instalar el agente de Log Analytics pero no Dependency Agent. No existe ninguna iniciativa integrada solo para el agente, pero puede crear su propia iniciativa a partir de las definiciones de directivas integradas proporcionadas por Azure Monitor para VM.

> [!NOTE]
> No debería haber ninguna razón para implementar Dependency Agent por sí mismo, ya que requiere que el Log Analytics agente entregue sus datos a Azure Monitor.


|Nombre |Descripción |
|-----|------------|
|Auditoría de la implementación del agente de Log Analytics: la imagen de la VM (SO) no está en la lista |Notifica que las máquinas virtuales no son compatibles si la imagen de la máquina virtual (SO) no está definida en la lista y el agente no está instalado. |
|Implementar el agente de Log Analytics en máquinas virtuales de Linux |Se implementa el agente de Log Analytics en máquinas virtuales Linux si la imagen de la máquina virtual (SO) está en la lista definida y el agente no está instalado. |
|Implementar el agente de Log Analytics en máquinas virtuales Windows |Se implementa el agente de Log Analytics en máquinas virtuales Windows si la imagen de la máquina virtual (SO) está en la lista definida y el agente no está instalado. |
| [Versión preliminar]\: El agente de Log Analytics debe estar instalado en las máquinas Linux de Azure Arc |Informa las máquinas de Azure Arc híbridas como no compatibles con VM Linux si la imagen de la VM (SO) está definida en la lista y el agente no está instalado. |
| [Versión preliminar]\: El agente de Log Analytics debe estar instalado en las máquinas Windows de Azure Arc |Informa las máquinas de Azure Arc híbridas como no compatibles con VM Windows si la imagen de la VM (SO) está definida en la lista y el agente no está instalado. |
| [Versión preliminar]\: Implementar el agente de Log Analytics en máquinas de Azure Arc con Linux |Implemente el agente de Log Analytics en las máquinas de Azure Arc híbridas con Linux si la imagen de la VM (SO) está definida en la lista y el agente no está instalado. |
| [Versión preliminar]\: Implementar el agente de Log Analytics en máquinas de Azure Arc con Windows |Implemente el agente de Log Analytics en las máquinas de Azure Arc híbridas con Windows si la imagen de la VM (SO) está definida en la lista y el agente no está instalado. |
|Auditoría de implementación de Dependency Agent en conjuntos de escalado de máquinas virtuales: la imagen de la VM (SO) no está en la lista |Notifica que los conjuntos de escalado de máquinas virtuales no son compatibles si la imagen de la máquina virtual (SO) no está definida en la lista y el agente no está instalado. |
|Auditoría de implementación del agente de Log Analytics en conjuntos de escalado de máquinas virtuales: la imagen de la VM (SO) no está en la lista |Notifica que los conjuntos de escalado de máquinas virtuales no son compatibles si la imagen de la máquina virtual (SO) no está definida en la lista y el agente no está instalado. |
|Implementar el agente de Log Analytics para conjuntos de escalado de máquinas virtuales Linux |Se implementa el agente de Log Analytics en los conjuntos de escalado de máquinas virtuales Linux si la imagen de la máquina virtual (SO) está en la lista definida y el agente no está instalado. |
|Implementar el agente de Log Analytics para conjuntos de escalado de máquinas virtuales Windows |Se implementa el agente de Log Analytics en los conjuntos de escalado de máquinas virtuales Windows si la imagen de la máquina virtual (SO) está en la lista definida y el agente no está instalado. |


## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de [Azure Policy](../governance/policy/overview.md).
- Más información sobre la [configuración de diagnóstico](platform/diagnostic-settings.md).