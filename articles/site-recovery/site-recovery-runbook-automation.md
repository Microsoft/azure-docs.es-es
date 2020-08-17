---
title: Incorporación de runbooks de Azure Automation a los planes de recuperación de Site Recovery
description: Aprenda a ampliar los planes de recuperación con Azure Automation para la recuperación ante desastres con Azure Site Recovery.
author: rajani-janaki-ram
manager: gauravd
ms.service: site-recovery
ms.topic: conceptual
ms.date: 09/18/2019
ms.author: rajanaki
ms.openlocfilehash: 1df17ccb41c974c8939710850b5bb71ee3a5ef1b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87021586"
---
# <a name="add-azure-automation-runbooks-to-recovery-plans"></a>Incorporación de runbooks de Azure Automation a los planes de recuperación

En este artículo se describe cómo integrar runbooks de Azure Automation para ampliar los planes de recuperación de [Azure Site Recovery](site-recovery-overview.md). Le mostraremos cómo automatizar tareas sencillas que, de lo contrario, requerirían intervención manual y le enseñaremos a convertir una recuperación de varios pasos en una acción de un solo clic.

## <a name="recovery-plans"></a>Planes de recuperación 

Puede usar planes de recuperación al conmutar por error máquinas locales o máquinas virtuales de Azure. Los planes de recuperación le ayudan a definir un proceso de recuperación sistemático que define cómo se conmutan por error las máquinas y cómo se inician y recuperan después de la conmutación por error. 

La recuperación de aplicaciones de gran tamaño puede ser compleja. Los planes de recuperación ayudan a poner orden para que la recuperación sea constantemente precisa, repetible y automatizada. Puede automatizar las tareas de un plan de recuperación mediante scripts y con runbooks de Azure Automation. Los ejemplos típicos podrían ser la configuración de las opciones de una máquina virtual de Azure después de una conmutación por error o la reconfiguración de una aplicación que se ejecuta en esa máquina.

- [Obtenga más información](recovery-plan-overview.md) sobre los planes de recuperación.
- [Más información](../automation/automation-runbook-types.md) sobre los runbooks de Azure Automation.



## <a name="runbooks-in-recovery-plans"></a>Runbooks en los planes de recuperación

Puede agregar una cuenta de Azure Automation y runbooks a un plan de recuperación. El runbook se invoca al ejecutarse el plan de recuperación.

- La cuenta de Automation puede estar en cualquier región de Azure, pero tiene que residir en la misma suscripción que el almacén de Site Recovery. 
- Un runbook se puede ejecutar en un plan de recuperación durante la conmutación por error de una ubicación principal a una secundaria, o durante la conmutación por recuperación desde la ubicación secundaria a la principal.
- Los runbooks de un plan de recuperación se ejecutan en serie, uno después de otro, en el orden establecido.
- Si los runbooks de un plan de recuperación configuran las máquinas virtuales para que se inicien en grupos diferentes, el plan de recuperación continuará solo cuando Azure notifique que todas las máquinas virtuales están en ejecución.
- Los planes de recuperación continuarán ejecutándose incluso si se produce un error en un script.

### <a name="recovery-plan-context"></a>Contexto del plan de recuperación

Cuando se ejecuta un script, este inserta un contexto del plan de recuperación en el runbook. El contexto contiene las variables resumidas en esta tabla.

| **Nombre de la variable** | **Descripción** |
| --- | --- |
| RecoveryPlanName |Nombre del plan de recuperación. Se utiliza en acciones basadas en el nombre. |
| FailoverType |Especifica si se trata de una conmutación por error de prueba o de producción. 
| FailoverDirection | Especifica si la recuperación es en una ubicación principal o secundaria. |
| GroupID |Identifica el número de grupo del plan de recuperación cuando este se está ejecutando. |
| VmMap |Matriz de todas las máquinas virtuales del grupo. |
| Clave de VMMap |Clave única (GUID) para cada máquina virtual. |
| SubscriptionId |Identificador de la suscripción de Azure en la que se ha creado la máquina virtual. |
| ResourceGroupName | Nombre del grupo de recursos en el que se encuentra la máquina virtual.
| CloudServiceName |Nombre del servicio en la nube de Azure en el que se ha creado la máquina virtual. |
| RoleName |El nombre de la máquina virtual de Azure. |
| RecoveryPointId|La marca de tiempo de la recuperación de la máquina virtual. |

>[!Note]
>El valor de la variable "FailoverDirection" será "PrimaryToSecondary" en caso de conmutación por error y "SecondaryToPrimary" en caso de conmutación por recuperación.

El ejemplo siguiente muestra una variable de contexto:

```
{"RecoveryPlanName":"hrweb-recovery",
"FailoverType":"Test",
"FailoverDirection":"PrimaryToSecondary",
"GroupId":"1",
"VmMap":{"7a1069c6-c1d6-49c5-8c5d-33bfce8dd183":
    { "SubscriptionId":"7a1111111-c1d6-49c5-8c5d-111ce8dd183",
    "ResourceGroupName":"ContosoRG",
    "CloudServiceName":"pod02hrweb-Chicago-test",
    "RoleName":"Fabrikam-Hrweb-frontend-test",
    "RecoveryPointId":"TimeStamp"}
    }
}
```

Si desea tener acceso a todas las máquinas virtuales de VMMap en un bucle, puede usar el código siguiente:

```
$VMinfo = $RecoveryPlanContext.VmMap | Get-Member | Where-Object MemberType -EQ NoteProperty | select -ExpandProperty Name
$vmMap = $RecoveryPlanContext.VmMap
    foreach($VMID in $VMinfo)
    {
        $VM = $vmMap.$VMID                
            if( !(($VM -eq $Null) -Or ($VM.ResourceGroupName -eq $Null) -Or ($VM.RoleName -eq $Null))) {
            #this check is to ensure that we skip when some data is not available else it will fail
    Write-output "Resource group name ", $VM.ResourceGroupName
    Write-output "Rolename " = $VM.RoleName
            }
        }
```


El blog de Aman Sharma en [Harvesting Clouds](http://harvestingclouds.com) incluye un ejemplo útil de un [script de contexto del plan de recuperación](http://harvestingclouds.com/post/script-sample-azure-automation-runbook-for-asr-recovery-plan/).



## <a name="before-you-start"></a>Antes de comenzar

- Si no está familiarizado con Azure Automation, puede [registrarse](https://azure.microsoft.com/services/automation/) y [descargar ejemplos de scripts](https://azure.microsoft.com/documentation/scripts/).
- Asegúrese de que la cuenta de Automation tenga los siguientes módulos:
    - AzureRM.profile
    - AzureRM.Resources
    - AzureRM.Automation
    - AzureRM.Network
    - AzureRM.Compute

    Todos los módulos deben ser de versiones compatibles. La manera más sencilla es usar siempre las versiones más recientes de todos los módulos.



## <a name="customize-the-recovery-plan"></a>Personalización de planes de recuperación

1. En el almacén seleccione **Planes de recuperación (Site Recovery)** .
2. Para crear un plan de recuperación, haga clic en **+Plan de recuperación**. [Más información](site-recovery-create-recovery-plans.md). Si ya tiene un plan de recuperación, selecciónelo para abrirlo.
3. En la página del plan de recuperación, haga clic en **Personalizar**.

    ![Clic en la pestaña Personalizar](media/site-recovery-runbook-automation-new/custom-rp.png)

2. Haga clic en los puntos suspensivos (...) junto a **Grupo 1: Iniciar** > **Agregar acción posterior**.
3. En **Insertar acción**, compruebe que está seleccionado **Script** y especifique un nombre para el script (**Hola mundo**).
4. Especifique una cuenta de Automation y seleccione un runbook. Para guardar el script, haga clic en **Aceptar**. El script se agrega a **Grupo 1: Pasos posteriores**.


## <a name="reuse-a-runbook-script"></a>Reutilización de un script de runbook

Puede usar un solo script de runbook en varios planes de recuperación gracias a las variables externas. 

- Puede usar las [variables de Azure Automation](../automation/shared-resources/variables.md) para almacenar los parámetros para la ejecución de un plan de recuperación.
- Al agregar el nombre del plan de recuperación como prefijo a la variable, puede crear variables individuales para cada plan de recuperación. Luego, use las variables como parámetros.
- Puede cambiar un parámetro sin cambiar el script y aun así cambiar la forma en que el script funciona.

### <a name="use-a-simple-string-variable-in-a-runbook-script"></a>Uso de una variable de cadena simple en un script de runbook

En este ejemplo, un script toma la entrada de un grupo de seguridad de red (NSG) y la aplica a las máquinas virtuales de un plan de recuperación. 

1. Para que el script pueda detectar qué plan de recuperación se está ejecutando, use este contexto del plan de recuperación:

    ```
    workflow AddPublicIPAndNSG {
        param (
              [parameter(Mandatory=$false)]
              [Object]$RecoveryPlanContext
        )

        $RPName = $RecoveryPlanContext.RecoveryPlanName
    ```

2. Anote el nombre y el grupo de recursos del grupo de seguridad de red. Use estas variables como entradas para los scripts del plan de recuperación. 
1. En los recursos de la cuenta de Automation, cree una variable para almacenar el nombre del grupo de seguridad de red. Agregue un prefijo al nombre de la variable con el nombre del plan de recuperación.

    ![Creación de una variable de nombre de NSG](media/site-recovery-runbook-automation-new/var1.png)

2. Cree una variable para almacenar el nombre del grupo de recursos del grupo de seguridad de red. Agregue un prefijo al nombre de la variable con el nombre del plan de recuperación.

    ![Creación de un nombre de grupo de recursos de NSG](media/site-recovery-runbook-automation-new/var2.png)


3.  En el script, use este código de referencia para obtener los valores de variable:

    ```
    $NSGValue = $RecoveryPlanContext.RecoveryPlanName + "-NSG"
    $NSGRGValue = $RecoveryPlanContext.RecoveryPlanName + "-NSGRG"

    $NSGnameVar = Get-AutomationVariable -Name $NSGValue
    $RGnameVar = Get-AutomationVariable -Name $NSGRGValue
    ```

4.  Use las variables del runbook para aplicar el NSG a la interfaz de red de la máquina virtual conmutada por error:

    ```
    InlineScript {
    if (($Using:NSGname -ne $Null) -And ($Using:NSGRGname -ne $Null)) {
            $NSG = Get-AzureRmNetworkSecurityGroup -Name $Using:NSGname -ResourceGroupName $Using:NSGRGname
            Write-output $NSG.Id
            #Apply the NSG to a network interface
            #$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet
            #Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name FrontEnd `
            #  -AddressPrefix 192.168.1.0/24 -NetworkSecurityGroup $NSG
        }
    }
    ```


Para cada plan de recuperación, cree variables independientes para poder volver a usar el script. Agregue un prefijo mediante el nombre del plan de recuperación. 

Para obtener un script completo, de un extremo a otro, para este escenario, revise [este script](https://gallery.technet.microsoft.com/Add-Public-IP-and-NSG-to-a6bb8fee).


### <a name="use-a-complex-variable-to-store-more-information"></a>Uso de una variable compleja para almacenar más información

En algunos escenarios, es posible que no pueda crear variables independientes para cada plan de recuperación. Considere un escenario en el que quiera que un único script asigne una dirección IP pública en máquinas virtuales concretas. En otro escenario, podría querer aplicar diferentes NSG en diferentes máquinas virtuales (no en todas las máquinas virtuales). Observe lo siguiente:

- Puede crear un script que se pueda volver a usar para cualquier plan de recuperación.
- Cada plan de recuperación puede tener un número variable de máquinas virtuales.
- Por ejemplo, una recuperación de SharePoint tiene dos servidores front-end. Una aplicación básica de línea de negocio (LOB) tiene solo un servidor front-end.
- En este escenario, no puede crear variables independientes para cada plan de recuperación.

En el siguiente ejemplo se crea una [variable compleja](/powershell/module/servicemanagement/azure.service/set-azureautomationvariable) en la cuenta de Azure Automation.

Para ello se especifican varios valores mediante Azure PowerShell.

1. En PowerShell, inicie sesión en la suscripción de Azure:

    ```
    Connect-AzureRmAccount
    $sub = Get-AzureRmSubscription -Name <SubscriptionName>
    $sub | Select-AzureRmSubscription
    ```

2. Para almacenar los parámetros, cree la variable compleja mediante el nombre del plan de recuperación:

    ```
    $VMDetails = @{"VMGUID"=@{"ResourceGroupName"="RGNameOfNSG";"NSGName"="NameOfNSG"};"VMGUID2"=@{"ResourceGroupName"="RGNameOfNSG";"NSGName"="NameOfNSG"}}
        New-AzureRmAutomationVariable -ResourceGroupName <RG of Automation Account> -AutomationAccountName <AA Name> -Name <RecoveryPlanName> -Value $VMDetails -Encrypted $false
    ```

3. En esta variable compleja, **VMDetails** es el identificador de la máquina virtual protegida. Para obtener el identificador de la máquina virtual, en Azure Portal, vea las propiedades de la máquina virtual. En la captura de pantalla siguiente se muestra una variable que almacena los detalles de dos máquinas virtuales:

    ![Empleo del identificador de la máquina virtual como GUID](media/site-recovery-runbook-automation-new/vmguid.png)

4. Use esta variable en el runbook. Si el GUID indicado de la máquina virtual se encuentra en el contexto del plan de recuperación, aplique el NSG en la máquina virtual:

    ```
    $VMDetailsObj = (Get-AutomationVariable -Name $RecoveryPlanContext.RecoveryPlanName).ToObject([hashtable])
    ```

4. En el runbook, cree un bucle con las máquinas virtuales del contexto del plan de recuperación. Compruebe si la máquina virtual existe en **$VMDetailsObj**. En caso afirmativo, acceda a las propiedades de la variable para aplicar el NSG:

    ```
        $VMinfo = $RecoveryPlanContext.VmMap | Get-Member | Where-Object MemberType -EQ NoteProperty | select -ExpandProperty Name
        $vmMap = $RecoveryPlanContext.VmMap

        foreach($VMID in $VMinfo) {
            $VMDetails = $VMDetailsObj[$VMID].ToObject([hashtable]);
            Write-output $VMDetails
            if ($VMDetails -ne $Null) { #If the VM exists in the context, this will not be Null
                $VM = $vmMap.$VMID
                # Access the properties of the variable
                $NSGname = $VMDetails.NSGName
                $NSGRGname = $VMDetails.NSGResourceGroupName

                # Add code to apply the NSG properties to the VM
            }
        }
    ```

Puede usar el mismo script para distintos planes de recuperación. Escriba distintos parámetros al almacenar el valor que corresponde a un plan de recuperación en variables diferentes.

## <a name="sample-scripts"></a>Muestras de scripts

Para implementar scripts de ejemplo en la cuenta de Automation, haga clic en el botón **Implementar en Azure**.

[![Implementación en Azure](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/c4803408-340e-49e3-9a1f-0ed3f689813d.png)](https://aka.ms/asr-automationrunbooks-deploy)

Este vídeo proporciona otro ejemplo. En él se muestra cómo recuperar una aplicación de WordPress de dos niveles en Azure:


> [!VIDEO https://channel9.msdn.com/Series/Azure-Site-Recovery/One-click-failover-of-a-2-tier-WordPress-application-using-Azure-Site-Recovery/player]


## <a name="next-steps"></a>Pasos siguientes

- Más información sobre una [cuenta de ejecución de Azure Automation](../automation/manage-runas-account.md)
- Revise los [ejemplos de scripts de Azure Automation](https://gallery.technet.microsoft.com/scriptcenter/site/search?f%5B0%5D.Type=User&f%5B0%5D.Value=SC%20Automation%20Product%20Team&f%5B0%5D.Text=SC%20Automation%20Product%20Team).
- [Aprenda más](site-recovery-failover.md) sobre la ejecución de conmutaciones por error.
