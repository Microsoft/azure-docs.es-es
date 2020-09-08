---
title: Acceso a máquina virtual Just-In-Time en Azure Security Center | Microsoft Docs
description: En este documento se explica la forma en que el acceso a máquinas virtuales Just-In-Time (JIT) en Azure Security Center le ayuda a controlar el acceso a máquinas virtuales de Azure.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 07/12/2020
ms.author: memildin
ms.openlocfilehash: 88f1924f69aed350b39f953cb7503a0dde9ca9ad
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056321"
---
# <a name="secure-your-management-ports-with-just-in-time-access"></a>Protección de los puertos de administración con acceso Just-in-Time

Bloquee el tráfico de entrada a Azure Virtual Machines con la característica de acceso a máquinas virtuales (VM) Just-in-Time (JIT) de Azure Security Center. Esto reduce la exposición a los ataques a la vez que proporciona un acceso sencillo cuando necesita conectarse a una máquina virtual.

Para obtener una explicación completa sobre cómo funciona JIT y la lógica subyacente, consulte la [explicación sobre Just-in-Time](just-in-time-explained.md).

Esta página le enseña cómo incluir JIT en el programa de seguridad. Aprenderá a: 

- **Habilitar JIT en las máquinas virtuales**: puede habilitar JIT con sus propias opciones personalizadas para una o más máquinas virtuales mediante Security Center, PowerShell o la API REST. Como alternativa, puede habilitar JIT con parámetros predeterminados codificados de forma rígida de las máquinas virtuales de Azure. Cuando se habilita, JIT bloquea el tráfico entrante a las máquinas virtuales de Azure mediante la creación de una regla en el grupo de seguridad de red.
- **Solicitar acceso a una máquina virtual con JIT habilitado**: el objetivo de JIT es asegurarse de que, aunque el tráfico entrante esté bloqueado, Security Center sigue facilitando el acceso para conectarse a las máquinas virtuales cuando sea necesario. Puede solicitar acceso a una máquina virtual habilitada para JIT desde Security Center, Azure Virtual Machines, PowerShell o la API REST.
- **Auditar la actividad**: para asegurarse de que las máquinas virtuales estén protegidas correctamente, revise los accesos a las máquinas virtuales habilitadas para JIT como parte de las comprobaciones de seguridad periódicas.   



## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponibilidad general|
|Precios:|Nivel Standard|
|Máquinas virtuales admitidas:|![Sí](./media/icons/yes-icon.png) Máquinas virtuales implementadas mediante Azure Resource Manager.<br>![No](./media/icons/no-icon.png) Máquinas virtuales con modelos de implementación clásica. [Más información acerca de estos modelos de implementación](../azure-resource-manager/management/deployment-models.md).<br>![No](./media/icons/no-icon.png) Máquinas virtuales que las instancias de Azure Firewall protegen y que [Azure Firewall Manager](https://docs.microsoft.com/azure/firewall-manager/overview) controla.|
|Roles y permisos necesarios:|Los roles **Lector** y **SecurityReader** pueden ver el estado y los parámetros de JIT.<br>Para crear roles personalizados que puedan funcionar con JIT, consulte [¿Cuáles son los permisos necesarios para configurar y usar Just-In-Time?](just-in-time-explained.md#what-permissions-are-needed-to-configure-and-use-jit).<br>Para crear un rol con privilegios mínimos para los usuarios que necesiten solicitar acceso JIT a una máquina virtual y no realizar ninguna otra operación JIT, use el [script Set-JitLeastPrivilegedRole](https://github.com/Azure/Azure-Security-Center/tree/master/Powershell%20scripts/JIT%20Custom%20Role) de las páginas de la comunidad de GitHub de Security Center.|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||


## <a name="enable-jit-vm-access"></a>Habilitación del acceso a máquinas virtuales JIT <a name="jit-configure"></a>

Puede habilitar el acceso a máquinas virtuales JIT con sus propias opciones personalizadas para una o más máquinas virtuales mediante Security Center o mediante programación. 

Como alternativa, puede habilitar JIT con parámetros predeterminados codificados de forma rígida de las máquinas virtuales de Azure.

Cada una de estas opciones se explica en una pestaña independiente a continuación.

### <a name="azure-security-center"></a>[**Azure Security Center**](#tab/jit-config-asc)

### <a name="enable-jit-on-your-vms-from-azure-security-center"></a>Habilitación de JIT en las máquinas virtuales desde Azure Security Center <a name="jit-asc"></a>

![Configuración del acceso a máquinas virtuales JIT en Azure Security Center](./media/security-center-just-in-time/jit-config-security-center.gif)

Desde Security Center, puede habilitar y configurar el acceso a máquinas virtuales JIT.

1. En el menú de Security Center, seleccione **Acceso de máquina virtual Just-In-Time**.

    Se abrirá la página **Acceso de máquina virtual Just-In-Time** con las máquinas virtuales agrupadas en las siguientes pestañas:

    - **Configurado**: máquinas virtuales que ya se han configurado para admitir el acceso a máquinas virtuales Just-In-Time. Para cada máquina virtual, la pestaña Configurado muestra:
        - el número de solicitudes JIT aprobadas en los últimos siete días
        - la fecha y hora del último acceso
        - los detalles de la conexión configurados
        - el último usuario
    - **No configurado**: máquinas virtuales sin JIT habilitado, pero que pueden admitir JIT. Se recomienda habilitar JIT para estas máquinas virtuales.
    - **No admitido**: máquinas virtuales sin JIT habilitado y que no admiten la característica. Es posible que su máquina virtual esté en esta pestaña por las siguientes razones:
      - Falta el grupo de seguridad de red (NSG): JIT requiere que se configure un NSG
      - Máquina virtual clásica: JIT es compatible con las máquinas virtuales que se implementan a través de Azure Resource Manager, no con la "implementación clásica". [Obtenga más información sobre los modelos de implementación clásica y de Resource Manager](../azure-resource-manager/management/deployment-models.md).
      - Otro: la máquina virtual puede estar en esta pestaña si la solución JIT está deshabilitada en la directiva de seguridad de la suscripción o el grupo de recursos.

1. En la pestaña **No configurado**, marque las máquinas virtuales que desea proteger con JIT y seleccione **Habilitar JIT en VM**. 

    Se abre la página de acceso a máquinas virtuales JIT en la que se enumeran los puertos que Security Center recomienda proteger:
    - 22 - SSH
    - 3389 - RDP
    - 5985 - WinRM 
    - 5986 - WinRM

    Para aceptar la configuración predeterminada, seleccione **Guardar**.

1. Para personalizar las opciones de JIT:

    - Agregue puertos personalizados con el botón **Agregar**. 
    - Modifique uno de los puertos predeterminados seleccionándolo en la lista.

    Para cada puerto (personalizado y predeterminado), el panel **Agregar configuración de puerto** ofrece las siguientes opciones:

    - **Protocolo**: el protocolo que se permite en este puerto cuando se aprueba una solicitud.
    - **Direcciones IP de origen permitidas**: los intervalos de direcciones IP que se permiten en este puerto cuando se aprueba una solicitud.
    - **Tiempo de solicitud máximo**: el período de tiempo máximo durante el que puede estar abierto un puerto específico.

     1. Establezca la seguridad del puerto según sus necesidades.

     1. Seleccione **Aceptar**.

1. Seleccione **Guardar**.



### <a name="edit-the-jit-configuration-on-a-jit-enabled-vm-using-security-center"></a>Edición de la configuración JIT en una máquina virtual habilitada para JIT mediante con Security Center <a name="jit-modify"></a>

Puede modificar la configuración Just-In-Time de una máquina virtual agregando y configurando un puerto nuevo que proteja dicha máquina virtual, o bien cambiando otro valor relacionado con un puerto ya protegido.

Para editar las reglas JIT existentes para una máquina virtual:

1. En el menú de Security Center, seleccione **Acceso de máquina virtual Just-In-Time**.

1. En la pestaña **Configurado**, haga clic con el botón derecho en la máquina virtual a la que desea agregar un puerto y seleccione Editar. 

    ![Edición de una configuración de acceso de máquina virtual JIT en Azure Security Center](./media/security-center-just-in-time/jit-policy-edit-security-center.png)

1. En **Configuración de acceso a máquina virtual del tipo JIT** , puede modificar la configuración existente de un puerto protegido, o bien agregar un puerto personalizado.

1. Cuando haya terminado de editar los puertos, seleccione **Guardar**.
 


### <a name="azure-virtual-machines"></a>[**Máquinas virtuales de Azure**](#tab/jit-config-avm)

### <a name="enable-jit-on-your-vms-from-azure-virtual-machines"></a>Habilitación de JIT en las máquinas virtuales desde máquinas virtuales de Azure

Puede habilitar JIT en una máquina virtual desde las páginas de máquinas virtuales de Azure Portal.

![Configuración del acceso a máquinas virtuales JIT en máquinas virtuales de Azure](./media/security-center-just-in-time/jit-config-virtual-machines.gif)

> [!TIP]
> Si una máquina virtual ya tiene Just-In-Time habilitado, cuando va a su página de configuración, verá que Just-In-Time está habilitado y que puede usar el vínculo para abrir la página de acceso a máquinas virtuales Just-In-Time en Security Center, y ver y cambiar la configuración.

1. En [Azure Portal](https://ms.portal.azure.com), busque y seleccione **Máquinas virtuales**. 

1. Seleccione la máquina virtual que desee proteger con JIT.

1. En el menú, seleccione **Configuración**.

1. En **Acceso Just-In-Time**, seleccione **Habilitar Just-In-Time**. 

    Esto habilita el acceso Just-In-Time para la máquina virtual con las siguientes configuraciones predeterminadas:

    - Máquinas Windows:
        - Puerto RDP 3389
        - Se permite un tiempo de acceso máximo de tres horas
        - Las direcciones IP de origen permitidas se establecen en Cualquiera
    - Máquinas Linux:
        - Puerto SSH 22
        - Se permite un tiempo de acceso máximo de tres horas
        - Las direcciones IP de origen permitidas se establecen en Cualquiera

1. Para editar cualquiera de estos valores o agregar más puertos a la configuración JIT, use la página Just-in-Time de Azure Security Center:

    1. En el menú de Security Center, seleccione **Acceso de máquina virtual Just-In-Time**.

    1. En la pestaña **Configurado**, haga clic con el botón derecho en la máquina virtual a la que desea agregar un puerto y seleccione Editar. 

        ![Edición de una configuración de acceso de máquina virtual JIT en Azure Security Center](./media/security-center-just-in-time/jit-policy-edit-security-center.png)

    1. En **Configuración de acceso a máquina virtual del tipo JIT** , puede modificar la configuración existente de un puerto protegido, o bien agregar un puerto personalizado.

    1. Cuando haya terminado de editar los puertos, seleccione **Guardar**.


### <a name="powershell"></a>[**PowerShell**](#tab/jit-config-powershell)

### <a name="enable-jit-on-your-vms-using-powershell"></a>Habilitación de JIT en las máquinas virtuales mediante PowerShell

Para habilitar el acceso a máquinas virtuales Just-In-Time mediante PowerShell, use el cmdlet de PowerShell de Azure Security Center oficial: `Set-AzJitNetworkAccessPolicy`.

**Ejemplo**: habilite el acceso a máquinas virtuales Just-in-Time en una máquina virtual específica con las siguientes reglas:

* Cierre los puertos 22 y 3389.
* Establezca una ventana de tiempo máximo de 3 horas para cada uno, para que así puedan abrirse según la solicitud aprobada.
* Deje que el usuario que solicita el acceso controle las direcciones IP de origen.
* Deje que el usuario que solicita el acceso establezca una sesión correcta tras una solicitud de acceso del tipo Just-In-Time aprobada.

Los siguientes comandos de PowerShell crean esta configuración JIT:

1. Asigne una variable que contenga las reglas de acceso a máquinas virtuales Just-in-Time para una máquina virtual:

    ```azurepowershell
    $JitPolicy = (@{
        id="/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Compute/virtualMachines/VMNAME";
        ports=(@{
             number=22;
             protocol="\*";
             allowedSourceAddressPrefix=@("\*");
             maxRequestAccessDuration="PT3H"},
             @{
             number=3389;
             protocol="\*";
             allowedSourceAddressPrefix=@("\*");
             maxRequestAccessDuration="PT3H"})})
    ```

1. Inserte las reglas de acceso a máquinas virtuales Just-in-Time de la máquina virtual en una matriz:
    
    ```azurepowershell
    $JitPolicyArr=@($JitPolicy)
    ```

1. Configure las reglas de acceso a máquinas virtuales Just-in-Time en la máquina virtual seleccionada:
    
    ```azurepowershell
    Set-AzJitNetworkAccessPolicy -Kind "Basic" -Location "LOCATION" -Name "default" -ResourceGroupName "RESOURCEGROUP" -VirtualMachine $JitPolicyArr
    ```

    Use el parámetro -Name para especificar una máquina virtual. Por ejemplo, para establecer la configuración JIT para dos máquinas virtuales diferentes, VM1 y VM2, use ```Set-AzJitNetworkAccessPolicy -Name VM1``` y ```Set-AzJitNetworkAccessPolicy -Name VM2```.


### <a name="rest-api"></a>[**API DE REST**](#tab/jit-config-api)

### <a name="enable-jit-on-your-vms-using-the-rest-api"></a>Habilitación de JIT en las máquinas virtuales mediante API REST

La característica de acceso a máquina virtual Just-in-Time se puede usar mediante la API de Azure Security Center. Use esta API para obtener más información acerca de las máquinas virtuales configuradas, cómo agregar otras nuevas o solicitar acceso a una máquina virtual, entre otras cosas. 

Obtenga más información en [Directivas de acceso a redes JIT](https://docs.microsoft.com/rest/api/securitycenter/jitnetworkaccesspolicies).


--- 










## <a name="request-access-to-a-jit-enabled-vm"></a>Solicitud de acceso a una máquina virtual habilitada para JIT

Puede solicitar acceso a una máquina virtual habilitada para JIT desde Azure Portal (en Security Center o Azure Virtual Machines) o mediante programación.

Cada una de estas opciones se explica en una pestaña independiente a continuación.

### <a name="azure-security-center"></a>[**Azure Security Center**](#tab/jit-request-asc)

### <a name="request-access-to-a-jit-enabled-vm-from-azure-security-center"></a>Solicitud de acceso a una máquina virtual habilitada para JIT desde Azure Security Center 

Cuando una máquina virtual tiene JIT habilitado, tiene que solicitar acceso para conectarse a ella. Puede solicitar acceso de cualquiera de las maneras admitidas, independientemente de cómo haya habilitado JIT.

![Solicitud de acceso JIT desde Azure Security Center](./media/security-center-just-in-time/jit-request-security-center.gif)

1. En la página **Acceso de máquina virtual Just-In-Time**, seleccione la pestaña **Configurado**.

1. Marque las máquinas virtuales a las que desea acceder.

    - El icono de la columna **Detalles de la conexión** indica si JIT está habilitado en el grupo de seguridad de red o el firewall. Si está habilitado en ambos, solo aparece el icono de firewall.

    - En la columna **Detalles de la conexión** se proporciona la información necesaria para conectarse a la máquina virtual y sus puertos abiertos.

1. Seleccione **Solicitar acceso**. Se abre la ventana **Solicitar acceso**.

1. En **Solicitar acceso**, configure para cada máquina virtual los puertos que desee que se abran y las direcciones IP de origen en que se abre el puerto y el tiempo durante el que permanecerá abierto. Solo será posible solicitar acceso a los puertos configurados. Cada puerto tiene un tiempo máximo permitido que se basa en la configuración JIT que ha creado.

1. Seleccione **Open ports** (Abrir puertos).

> [!NOTE]
> Si un usuario que solicita acceso está detrás de un proxy, es posible que la opción de **Mi IP** no funcione. Es posible que necesite definir todo el intervalo de direcciones IP de la organización.



### <a name="azure-virtual-machines"></a>[**Máquinas virtuales de Azure**](#tab/jit-request-avm)

### <a name="request-access-to-a-jit-enabled-vm-from-the-azure-virtual-machines-connect-page"></a>Solicitud de acceso a una máquina virtual habilitada para JIT desde la página de conexión de la máquina virtual de Azure

Cuando una máquina virtual tiene JIT habilitado, tiene que solicitar acceso para conectarse a ella. Puede solicitar acceso de cualquiera de las maneras admitidas, independientemente de cómo haya habilitado JIT.

  >![Solicitud Just-In-Time (JIT)](./media/security-center-just-in-time/jit-request-vm.png)


Para solicitar acceso desde máquinas virtuales de Azure:

1. En Azure Portal, abra las páginas de máquinas virtuales.

1. Seleccione la máquina virtual a la que desea conectarse y abra la página **Conectar**.

    Azure comprueba si JIT está habilitado en esa máquina virtual.

    - Si JIT no está habilitado para la máquina virtual, se le pedirá que lo habilite.

    - Si está JIT está habilitado, seleccione **Solicitar acceso** para pasar una solicitud de acceso con la dirección IP de solicitud, el intervalo de tiempo y los puertos que se configuraron para esa máquina virtual.

> [!NOTE]
> Después de aprobarse una solicitud para una VM protegida con Azure Firewall, Security Center proporciona al usuario los datos de conexión (asignación de puerto de la tabla DNAT) que debe usar para conectarse a la máquina virtual.



### <a name="powershell"></a>[**PowerShell**](#tab/jit-request-powershell)

### <a name="request-access-to-a-jit-enabled-vm-using-powershell"></a>Solicitud de acceso a una máquina virtual habilitada para JIT mediante PowerShell

En el siguiente ejemplo, puede ver una solicitud de acceso a máquina virtual del tipo Just-In-Time para una máquina virtual específica, en la que se solicita que se abra el puerto 22 para una dirección IP específica y durante un período concreto:

Ejecute lo siguiente en PowerShell:

1. Configure las propiedades de acceso de solicitud de la máquina virtual:

    ```azurepowershell
    $JitPolicyVm1 = (@{
        id="/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Compute/virtualMachines/VMNAME";
        ports=(@{
           number=22;
           endTimeUtc="2020-07-15T17:00:00.3658798Z";
           allowedSourceAddressPrefix=@("IPV4ADDRESS")})})
    ```

1. Inserte los parámetros de solicitud de acceso a la máquina virtual en una matriz:

    ```azurepowershell
    $JitPolicyArr=@($JitPolicyVm1)
    ```
        
1. Envíe el acceso de solicitud (use el id. de recurso del paso 1).

    ```azurepowershell
    Start-AzJitNetworkAccessPolicy -ResourceId "/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Security/locations/LOCATION/jitNetworkAccessPolicies/default" -VirtualMachine $JitPolicyArr
    ```

Para más información, consulte la [documentación del cmdlet de PowerShell](https://docs.microsoft.com/powershell/scripting/developer/cmdlet/cmdlet-overview).



### <a name="rest-api"></a>[**API DE REST**](#tab/jit-request-api)

### <a name="request-access-to-a-jit-enabled-vms-using-the-rest-api"></a>Solicitud de acceso a una máquina virtual habilitada para JIT mediante API REST

La característica de acceso a máquina virtual Just-in-Time se puede usar mediante la API de Azure Security Center. Use esta API para obtener más información acerca de las máquinas virtuales configuradas, cómo agregar otras nuevas o solicitar acceso a una máquina virtual, entre otras cosas. 

Obtenga más información en [Directivas de acceso a redes JIT](https://docs.microsoft.com/rest/api/securitycenter/jitnetworkaccesspolicies).

---








## <a name="audit-jit-access-activity-in-security-center"></a>Auditoría de la actividad de acceso JIT en Security Center

Puede usar la búsqueda de registros para obtener información sobre las actividades de las máquinas virtuales. Para ver los registros:

1. En **Acceso de máquina virtual Just-In-Time**, seleccione la pestaña **Configurado**.

1. Para la máquina virtual que desea auditar, abra el menú de puntos suspensivos al final de la fila.
 
1. Seleccione **Registro de actividades** en el menú.

   ![Selección del registro de actividad Just-in-Time (JIT)](./media/security-center-just-in-time/jit-select-activity-log.png)

   El registro de actividades proporciona una vista filtrada de las operaciones anteriores para esa máquina virtual, junto con el tiempo, la fecha y la suscripción.

1. Para descargar la información de registro, seleccione **Descargar como CSV**.








## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a configurar y usar el acceso a máquinas virtuales Just-in-Time. Para saber por qué se debe usar JIT, lea el artículo del concepto en el que se explican las amenazas contra las que se está defendiendo:

> [!div class="nextstepaction"]
> [JIT explicado](just-in-time-explained.md)