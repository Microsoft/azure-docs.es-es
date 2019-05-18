---
title: Habilitar Monitor de Azure para máquinas virtuales con Azure Policy | Microsoft Docs
description: En este artículo se describe cómo habilitar a Azure Monitor para las máquinas virtuales para varias máquinas virtuales o conjuntos de escalado de máquinas virtuales mediante la directiva de Azure.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/07/2019
ms.author: magoedte
ms.openlocfilehash: 9664ad5272ffeb55bd85e3d4c4f4b70d05e97702
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65524151"
---
# <a name="enable-azure-monitor-for-vms-preview-using-azure-policy"></a>Habilitar a Monitor de Azure para máquinas virtuales mediante la directiva de Azure (versión preliminar)

En este artículo se explica cómo habilitar a Azure Monitor para las máquinas virtuales (versión preliminar) para máquinas virtuales o conjuntos de escalado de máquinas virtuales mediante la directiva de Azure. Al final de este proceso correctamente habrá configurado habilitación de Log Analytics y los agentes de dependencia e identifica las máquinas virtuales que no son compatibles.

Para detectar, administrar y habilitar el Monitor de Azure para máquinas virtuales para todas sus máquinas virtuales o conjuntos de escalado de máquinas virtuales, puede usar Directiva de Azure o Azure PowerShell. Directiva de Azure es el método recomendado porque puede administrar las definiciones de directiva para controlar de forma eficaz sus suscripciones para garantizar el cumplimiento coherente y habilitación automática del recién aprovisiona las máquinas virtuales. Estas definiciones de directiva:

* Implementan el agente de Log Analytics y Dependency Agent.
* Informan sobre los resultados de cumplimiento.
* Corrigen las máquinas virtuales no compatibles.

Si está interesado en llevar a cabo esto con la plantilla de Azure PowerShell o Azure Resource Manager, consulte [habilitar el uso de la plantilla de Resource Manager o Azure PowerShell](vminsights-enable-at-scale-powershell.md). 

## <a name="manage-policy-coverage-feature-overview"></a>Introducción a la característica de cobertura de directivas de administración

Originalmente se realizó la experiencia con la directiva de Azure para administrar e implementar las definiciones de directiva de Azure Monitor para las máquinas virtuales exclusivamente a partir de la directiva de Azure. Con el **administrar Directiva cobertura** característica, resulta más sencillo y fácil de detectar, administrar y habilitar a escala el **habilitar Azure Monitor para las máquinas virtuales** iniciativa, que incluye las definiciones de directiva se ha mencionado anteriormente. Puede tener acceso a esta nueva característica de la **comenzar** ficha en Azure Monitor para las máquinas virtuales mediante la selección **administrar Directiva de cobertura**. Se abre la página de cobertura de directivas de máquinas virtuales. 

![Azure Monitor desde la pestaña de empezar a trabajar con máquinas virtuales](./media/vminsights-enable-at-scale-policy/get-started-page-01.png)

Desde aquí puede comprobar y administrar la cobertura de la iniciativa a través de los grupos de administración y suscripciones, así como conocer cuántas máquinas virtuales existen en cada uno de los grupos de administración, suscripciones y su estado de cumplimiento.   

![Azure Monitor para la página de la directiva de administración de máquinas virtuales](./media/vminsights-enable-at-scale-policy/manage-policy-page-01.png)

Esta información es útil para planear y ejecutar el escenario de gobierno de Azure Monitor para máquinas virtuales desde una ubicación central. Aunque Azure Policy ofrece una vista de compatibilidad cuando una iniciativa de directiva se asigna a un ámbito, con esta nueva página puede detectar donde la iniciativa de directiva no está asignado y asignarla en contexto. Todas las acciones (asignar, ver, editar) redirigir a Azure Policy directamente. Azure Monitor para la página de cobertura de directivas de máquinas virtuales es una experiencia integrada y expandida para solo la iniciativa **habilitar Azure Monitor para las máquinas virtuales**. 

Desde esta página también puede configurar el área de trabajo de Log Analytics de Azure Monitor para las máquinas virtuales, que realiza lo siguiente:

- Instala las soluciones de instalación de Service Map y Infrastructure Insights
- Habilita los contadores de rendimiento del sistema operativo utilizados por los gráficos de rendimiento, libros y sus consultas de registro personalizado y las alertas.

![Azure Monitor para máquinas virtuales de configurar el área de trabajo](./media/vminsights-enable-at-scale-policy/manage-policy-page-02.png)

Esta opción no está relacionada con las acciones de directiva y está disponible para proporcionar una manera fácil para satisfacer la [requisitos previos](vminsights-enable-overview.md) necesarios para habilitar el Monitor de Azure para máquinas virtuales.  

### <a name="what-information-is-available-on-this-page"></a>¿Qué información está disponible en esta página?
En la tabla siguiente proporciona un desglose de la información presentada en la página de cobertura de directivas y cómo interpretarlo.

| Función | DESCRIPCIÓN | 
|----------|-------------| 
| **Ámbito** | Grupo de administración y suscripciones que tenga o hereda el acceso a la capacidad de explorar en profundidad a través de la administración de la jerarquía de grupos.|
| **Rol** | El rol para el ámbito, podría ser el propietario de lector o colaborador. En algunos casos, puede aparecer en blanco para indicar que puede tener acceso a la suscripción, pero no para el grupo de administración que pertenece. Información de otras columnas variará en función del rol ya que es clave para determinar qué datos puede ver y acciones que puede realizar en cuanto a la asignación de iniciativa de directiva (propietario), su edición o visualización de cumplimiento. |
| **Total de máquinas virtuales** | Número de máquinas virtuales en ese ámbito. Para un grupo de administración, es una suma de las máquinas virtuales anidadas bajo las suscripciones o el grupo de administración secundario. |
| **Cobertura de asignación** | Porcentaje de máquinas virtuales que están cubiertas por la iniciativa de directiva. |
| **Estado de asignación** | En esta columna, puede encontrar información sobre el estado de la directiva o iniciativa asignación. |
| **Máquinas virtuales compatibles** | Número de máquinas virtuales que son compatibles con la iniciativa de directiva. Para la iniciativa **habilitar Azure Monitor para las máquinas virtuales** es el número de máquinas virtuales que tienen el agente de Log Analytics y agente de dependencia. En algunos casos, puede aparecer en blanco debido a que ninguna asignación, o ninguna máquina virtual o no tiene permisos suficientes. Se proporciona información en el estado de cumplimiento. |
| **Cumplimiento normativo** | El número total de cumplimiento es la suma de los distintos recursos conforme dividido por la suma de todos los recursos distintos. |
| **Estado de cumplimiento** | Información sobre el estado de cumplimiento de la directiva o iniciativa asignación.|

Al asignar la iniciativa de directiva, el ámbito seleccionado en la asignación podría ser el ámbito aparece o un subconjunto del mismo. Por ejemplo, es posible que ha creado una asignación para una suscripción (ámbito de directiva) y no un grupo de administración (ámbito de cobertura). En este caso, el valor de **asignación cobertura** indicaría el ámbito de las máquinas virtuales de la directiva (o una iniciativa) dividido entre las máquinas virtuales en el ámbito de cobertura. En otro caso, es posible que haya excluido algunas máquinas virtuales, grupos de recursos o una suscripción de ámbito de la directiva. Si el valor está en blanco, indica que la iniciativa de directiva no existe o no tiene permiso (se proporciona información de estado de asignación).

## <a name="enable-using-azure-policy"></a>Habilitar el uso de Azure Policy

Para habilitar Azure Monitor para VM mediante Azure Policy en su inquilino:

- Asigne la iniciativa a un ámbito: grupo de administración, suscripción o grupo de recursos
- Revise y corrección corrija resultados del cumplimiento

Para más información acerca de la asignación de Azure Policy, consulte [Información general de Azure Policy](../../governance/policy/overview.md#policy-assignment) y consulte la [introducción a los grupos de administración](../../governance/management-groups/index.md) antes de continuar.

### <a name="policies-for-azure-vms"></a>Directivas para máquinas virtuales de Azure

Las definiciones de directiva para una máquina virtual de Azure se muestran en la tabla siguiente:

|NOMBRE |DESCRIPCIÓN |Type |
|-----|------------|-----|
|[Versión preliminar]: Habilitar Azure Monitor para VM |Se habilita Azure Monitor para VM en el ámbito especificado (grupo de administración, suscripción o grupo de recursos). Toma el área de trabajo de Log Analytics como parámetro. |Iniciativa |
|[Versión preliminar]: Auditoría de la implementación de Dependency Agent: la imagen de la VM (SO) no está en la lista |Notifica que las máquinas virtuales no son compatibles si la imagen de la máquina virtual (SO) no está definida en la lista y el agente no está instalado. |Directiva |
|[Versión preliminar]: Auditar la implementación del agente de Log Analytics. La imagen de la VM (SO) no está en la lista |Notifica que las máquinas virtuales no son compatibles si la imagen de la máquina virtual (SO) no está definida en la lista y el agente no está instalado. |Directiva |
|[Versión preliminar]: Implementar Dependency Agent en máquinas virtuales Linux |Se implementa Dependency Agent en las máquinas virtuales Linux si la imagen de la máquina virtual (SO) está en la lista definida y el agente no está instalado. |Directiva |
|[Versión preliminar]: Implementar Dependency Agent en máquinas virtuales Windows |Se implementa Dependency Agent en las máquinas virtuales Windows si la imagen de la máquina virtual (SO) está en la lista definida y el agente no está instalado. |Directiva |
|[Versión preliminar]: Implementar el agente de Log Analytics en máquinas virtuales Linux |Se implementa el agente de Log Analytics en máquinas virtuales Linux si la imagen de la máquina virtual (SO) está en la lista definida y el agente no está instalado. |Directiva |
|[Versión preliminar]: Implementar el agente de Log Analytics en máquinas virtuales Windows |Se implementa el agente de Log Analytics en máquinas virtuales Windows si la imagen de la máquina virtual (SO) está en la lista definida y el agente no está instalado. |Directiva |

### <a name="policies-for-azure-virtual-machine-scale-sets"></a>Conjuntos de escalado de directivas para la máquina virtual de Azure

Las definiciones de directiva para un conjunto de escalado de máquina virtual de Azure se muestran en la tabla siguiente:

|NOMBRE |DESCRIPCIÓN |Type |
|-----|------------|-----|
|[Versión preliminar]: Habilitar a Monitor de Azure para conjuntos de escalado de máquinas virtuales (VMSS) |Habilitar a Monitor de Azure para los conjuntos de escalado de máquinas virtuales en el ámbito especificado (grupo administración, suscripción o grupo de recursos). Usa el área de trabajo de Log Analytics como parámetro. Nota: si su upgradePolicy del conjunto de escalado está establecido en Manual, se debe aplicar la extensión a todas las máquinas virtuales en el conjunto mediante una llamada de actualización en ellos. En la CLI de esto sería az vmss update-instances. |Iniciativa |
|[Versión preliminar]: Implementación del agente de dependencia de VMSS: imagen de máquina virtual (SO) dados de baja de auditoría |Escalado de máquinas virtuales de los informes se establece como no conforme si no se ha definido la imagen de máquina virtual (SO) en la lista y el agente no está instalado. |Directiva |
|[Versión preliminar]: Implantación de agentes de análisis de registro de auditoría en VMSS – dados de baja de imagen de la máquina virtual (SO) |Escalado de máquinas virtuales de los informes se establece como no conforme si no se ha definido la imagen de máquina virtual (SO) en la lista y el agente no está instalado. |Directiva |
|[Versión preliminar]: Implementar al agente de dependencia para conjuntos de escalado de máquina virtual Linux (VMSS) |Implementar el agente de dependencia para conjuntos de escalado de máquina virtual si la imagen de máquina virtual (SO) se define en la lista y no está instalado el agente de Linux. |Directiva |
|[Versión preliminar]: Implementar al agente de dependencia para conjuntos de escalado de máquinas virtuales (VMSS) de Windows |Implementar el agente de dependencia para Windows virtual machine scale sets con si la imagen de máquina virtual (SO) se define en la lista y el agente no está instalado. |Directiva |
|[Versión preliminar]: Implementar el agente de Log Analytics para VM Scale Sets (VMSS) para Linux |Implementar el agente de Log Analytics para conjuntos de escalado de máquina virtual si la imagen de máquina virtual (SO) se define en la lista y no está instalado el agente de Linux. |Directiva |
|[Versión preliminar]: Implementar el agente de Log Analytics para VM Scale Sets (VMSS) para Windows |Implementación de Log Analytics del agente para Windows virtual machine scale sets con si la imagen de máquina virtual (SO) se define en la lista y el agente no está instalado. |Directiva |

La directiva independiente (no se incluye con la iniciativa) se describe a continuación:

|NOMBRE |DESCRIPCIÓN |Type |
|-----|------------|-----|
|[Versión preliminar]: Auditar área de trabajo de Log Analytics en la máquina virtual: error de coincidencia del informe |Se notifica que las máquinas virtuales no son compatibles si no se registran en el área de trabajo de Log Analytics especificada en la asignación de la directiva o iniciativa. |Directiva |

### <a name="assign-the-azure-monitor-initiative"></a>Asignar la iniciativa de Azure Monitor
Para crear la asignación de directiva desde el Monitor de Azure para la página de cobertura de directivas de máquinas virtuales, realice los pasos siguientes. Para saber cómo completar estos pasos, consulte  [Create a policy assignment from the Azure portal](../../governance/policy/assign-policy-portal.md) (Creación de una asignación de directiva desde Azure Portal).

Al asignar la iniciativa de directiva, el ámbito seleccionado en la asignación podría ser el ámbito enumerados aquí o un subconjunto del mismo. Por ejemplo, es posible que ha creado la asignación de suscripción (ámbito de directiva) y no el grupo de administración (ámbito de cobertura) en el que el % de cobertura de casos indicaría dividido entre las máquinas virtuales en el ámbito de la cobertura de ámbito de las máquinas virtuales de la directiva (o una iniciativa). En cualquier otro caso, es posible que haya excluido algunas máquinas virtuales o grupos de recursos o suscripción de ámbito de la directiva.  En caso de que está en blanco, indica que una directiva de la iniciativa no existe o no tiene los permisos (información proporcionada en el estado de asignación)  

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).

2. En Azure Portal, seleccione **Monitor**. 

3. Elija **Máquinas virtuales (versión preliminar)** en la sección **Conclusiones**.
 
4. Seleccione el **comenzar** ficha y, en la página Seleccione **administrar Directiva de cobertura**.

5. Seleccione una suscripción o el grupo de administración de la tabla y seleccione **ámbito** haciendo clic en los puntos suspensivos (...).      En nuestro ejemplo, un ámbito limita la asignación de directiva a una agrupación de máquinas virtuales para su aplicación.

6. En el **asignación de directiva de Azure** página se rellena previamente con la iniciativa **habilitar Azure Monitor para las máquinas virtuales**. 
    El **nombre de la asignación** cuadro se rellena automáticamente con el nombre de la iniciativa, pero puede cambiarlo. También puede agregar una descripción, si lo desea. El cuadro **Asignado por** se rellena automáticamente en función de quién haya iniciado sesión y este valor es opcional.

7. (Opcional) Para quitar uno o varios recursos del ámbito, seleccione **Exclusiones**.

8. En la lista desplegable **Área de trabajo de Log Analytics** para la región admitida, seleccione un área de trabajo.

   > [!NOTE]
   > Si el área de trabajo está fuera del ámbito de la asignación, conceda los permisos de *colaborador de Log Analytics* al identificador de la entidad de seguridad. de la asignación de la directiva. Si no lo hace, es posible que vea un error de implementación, como: `The client '343de0fe-e724-46b8-b1fb-97090f7054ed' with object id '343de0fe-e724-46b8-b1fb-97090f7054ed' does not have authorization to perform action 'microsoft.operationalinsights/workspaces/read' over scope ...` Para conceder acceso, revise [cómo configurar manualmente la identidad administrada](../../governance/policy/how-to/remediate-resources.md#manually-configure-the-managed-identity).
   > 
   >  La casilla **Identidad administrada** se selecciona porque la iniciativa que se asigna incluye una directiva con el efecto *deployIfNotExists*.
    
9. En la lista desplegable **Ubicación de la identidad administrada**, seleccione la región adecuada.

10. Seleccione **Asignar**.

Una vez creada la asignación, el Monitor de Azure para la página de cobertura de directivas de máquinas virtuales se actualizará la cobertura de asignación, el estado de asignación, cumplimiento de las máquinas virtuales y el estado de cumplimiento para reflejar los cambios. 

La siguiente matriz asigna cada estado de compatibilidad posibles de la iniciativa.  

| Estado de compatibilidad | DESCRIPCIÓN | 
|------------------|-------------|
| **Compatible con** | Todas las máquinas virtuales en el ámbito tienen los agentes de Log Analytics y dependencia implementados en ellos.|
| **No compatible** | No todas las máquinas virtuales en el ámbito tienen el análisis de registro y los agentes de dependencia implementan en ellos y pueden requerir la corrección.|
| **No iniciado** | Se ha agregado una nueva asignación. |
| **Bloqueo** | No dispone de suficientes privilegios para el grupo de administración. <sup>1</sup> | 
| **En blanco** | Ninguna directiva asignada. | 

<sup>1</sup> si no tiene acceso al grupo de administración, solicite un propietario para proporcionar acceso o ver el cumplimiento y administrar las asignaciones a través de suscripciones o grupos de administración secundarios. 

En la tabla siguiente se asigna cada estado posible asignación de la iniciativa.

| Estado de la asignación | DESCRIPCIÓN | 
|------------------|-------------|
| **Success** | Todas las máquinas virtuales en el ámbito tienen los agentes de Log Analytics y dependencia implementados en ellos.|
| **Warning (ADVERTENCIA)** | La suscripción no está en un grupo de administración.|
| **No iniciado** | Se ha agregado una nueva asignación. |
| **Bloqueo** | No dispone de suficientes privilegios para el grupo de administración. <sup>1</sup> | 
| **En blanco** | Ninguna máquina virtual existe o no asigna la directiva. | 
| **Acción** | Asignar directiva o editar asignación | 

<sup>1</sup> si no tiene acceso al grupo de administración, solicite un propietario para proporcionar acceso o ver el cumplimiento y administrar las asignaciones a través de suscripciones o grupos de administración secundarios. 

## <a name="review-and-remediate-the-compliance-results"></a>Revisión y corrección de los resultados del cumplimiento

El ejemplo siguiente es para una máquina virtual de Azure, pero también se aplica a conjuntos de escalado de máquinas virtuales. Puede aprender a revisar los resultados de cumplimiento mediante la lectura de [identificación de los resultados de no cumplimiento](../../governance/policy/assign-policy-portal.md#identify-non-compliant-resources). En el Monitor de Azure en la página de cobertura de directivas de máquinas virtuales, seleccione una suscripción o el grupo de administración de la tabla y seleccione **Ver conformidad** haciendo clic en los puntos suspensivos (...).   

![Cumplimiento de directivas en máquinas virtuales de Azure](./media/vminsights-enable-at-scale-policy/policy-view-compliance-01.png)

En función de los resultados de las directivas incluidas con la iniciativa, las máquinas virtuales se notifican como no compatibles en los siguientes escenarios:

* Log Analytics o Dependency Agent no están implementados.  
    Este escenario es típico de un ámbito con las máquinas virtuales existentes. Para mitigarlo, implemente los agentes necesarios. Para ello, [cree tareas de corrección](../../governance/policy/how-to/remediate-resources.md) en una directiva no compatible.  
    - [Versión preliminar]: Deploy Dependency Agent for Linux VMs
    - [Versión preliminar]: Deploy Dependency Agent for Windows VMs
    - [Versión preliminar]: Deploy Log Analytics Agent for Linux VMs
    - [Versión preliminar]: Deploy Log Analytics Agent for Windows VMs

* La imagen de la máquina virtual (SO) no está identificada en la definición de la directiva.  
    Los criterios de la directiva de implementación solo incluyen las máquinas virtuales que se implementan a partir de imágenes de máquina virtual de Azure conocidas. Consulte en la documentación si el sistema operativo de VM es compatible. Si no lo es, duplique la directiva de implementación y actualícela o modifíquela para que la imagen sea compatible.  
    - [Versión preliminar]: Auditoría de la implementación de Dependency Agent: la imagen de la VM (SO) no está en la lista
    - [Versión preliminar]: Auditar la implementación del agente de Log Analytics. La imagen de la VM (SO) no está en la lista

* Las máquinas virtuales no inician sesión en el área de trabajo de Log Analytics especificada.  
    Es posible que algunas máquinas virtuales en el ámbito de la iniciativa inicien sesión en un área de trabajo de Log Analytics distinta de la que se especifica en la asignación de directiva. Esta directiva es una herramienta para identificar qué máquinas virtuales notifican a un área de trabajo no compatible.  
    - [Versión preliminar]: Audit Log Analytics Workspace for VM - Report Mismatch

## <a name="edit-initiative-assignment"></a>Editar asignación de iniciativa

En cualquier momento después de asignar una iniciativa a un grupo de administración o la suscripción, puede editarlo para modificar las propiedades siguientes:

- Nombre de asignación
- DESCRIPCIÓN
- Asignado por
- Área de trabajo de Log Analytics
- Excepciones

## <a name="next-steps"></a>Pasos siguientes

Ahora que la supervisión está habilitada para las máquinas virtuales, esta información está disponible para su análisis con Azure Monitor para las máquinas virtuales. Para obtener información sobre cómo usar la característica de mantenimiento, consulte [Descripción del estado de las máquinas virtuales de Azure con Azure Monitor para VM (versión preliminar)](vminsights-health.md). Para ver las dependencias de las aplicaciones detectadas, consulte [Uso de la asignación de Azure Monitor para VM (versión preliminar) para conocer los componentes de una aplicación](vminsights-maps.md). Para identificar cuellos de botella y el uso general con el rendimiento de sus VM, consulte el artículo sobre cómo [ver el rendimiento de la VM de Azure](vminsights-performance.md), o bien, para ver las dependencias de la aplicación detectadas, consulte cómo [ver el Azure Monitor para la asignación de VM](vminsights-maps.md).