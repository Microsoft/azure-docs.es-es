---
title: Ejecución de una recuperación ante desastres de Hyper-V en un sitio secundario con Azure Site Recovery
description: Obtenga información acerca de cómo ejecutar una exploración de la recuperación ante desastres para máquinas virtuales de Hyper-V en nubes de VMM en un centro de datos local secundario con Azure Site Recovery.
author: sideeksh
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/27/2018
ms.author: sideeksh
ms.openlocfilehash: 569af28f5773d843f49dd9c8143b45e308ae142e
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420420"
---
# <a name="run-a-dr-drill-for-hyper-v-vms-to-a-secondary-site"></a>Ejecución de una exploración de la recuperación ante desastres de máquinas virtuales de Hyper-V en un sitio secundario


En este artículo se describe cómo realizar una exploración de la recuperación ante desastres (DR) de máquinas virtuales de Hyper-V administradas en nubes de System Center Virtual Machine Manager (VMM) en un sitio secundario local mediante [Azure Site Recovery](site-recovery-overview.md).

La conmutación por error de prueba se ejecuta para validar la estrategia de replicación y para realizar una exploración de la recuperación ante desastres sin perder datos ni tiempo de actividad. La conmutación por error de prueba no afecta a la replicación en curso ni al entorno de producción. 

## <a name="how-do-test-failovers-work"></a>¿Cómo funciona la conmutación por error de prueba?

Puede ejecutar una conmutación por error de prueba del sitio principal al secundario. Si simplemente desea comprobar que una máquina virtual conmuta por error, puede ejecutar una conmutación por error de prueba sin tener que configurar nada en el sitio secundario. Si desea comprobar que la conmutación por error de la aplicación funciona según lo esperado, debe configurar las redes y la infraestructura en la ubicación secundaria.
- Puede ejecutar una conmutación por error de prueba en una sola máquina virtual o bien en un [plan de recuperación](site-recovery-create-recovery-plans.md).
- Puede ejecutar una conmutación por error de prueba sin una red, con una red existente o bien con una red creada automáticamente. En la tabla siguiente se proporcionan más detalles acerca de estas opciones.
    - Puede ejecutar una conmutación por error de prueba sin una red. Esta opción es útil si simplemente desea comprobar que una máquina virtual se puede conmutar por error, pero no podrá comprobar ninguna configuración de red.
    - Puede ejecutar la conmutación por error con una red existente. Se recomienda no usar una red de producción.
    - Puede ejecutar la conmutación por error y permitir que Site Recovery cree automáticamente una red de prueba. En este caso, Site Recovery crea la red automáticamente y procede a su limpieza una vez finalizada la conmutación por error de prueba.
- Debe seleccionar un punto de recuperación para la conmutación por error de prueba: 
    - **Procesado más recientemente**: con esta opción se realiza una conmutación por error de una máquina virtual al último punto de recuperación procesado por Site Recovery. Esta opción proporciona un objetivo de tiempo de recuperación (RTO) bajo, ya que no se invierte tiempo en el procesamiento de datos sin procesar.
    - **Más reciente coherente con la aplicación**: esta opción conmuta por error la máquina virtual al punto de recuperación más reciente coherente con la aplicación que haya procesado Site Recovery. 
    - **Más reciente**: esta opción procesa primero todos los datos que se han enviado al servicio Site Recovery para crear un punto de recuperación para cada máquina virtual antes de conmutarla por error a dicho punto de recuperación. Esta opción ofrece el objetivo de punto de recuperación (RPO) mínimo, ya que la máquina virtual de Azure creada después de la conmutación por error tiene todos los datos replicados en Site Recovery al desencadenarse la conmutación por error.
    - **Último procesamiento de máquinas virtuales múltiples**: disponible para los planes de recuperación con una o varias máquinas virtuales que tienen habilitada la coherencia entre varias máquinas virtuales. Máquinas virtuales con la conmutación por error habilitada para el último punto de recuperación común coherente con varias máquinas virtuales. Otras máquinas virtuales realizan la conmutación por error al último punto de recuperación procesado.
    - **Último coherente con aplicación de múltiples VM**: esta opción está disponible para los planes de recuperación con una o varias máquinas virtuales que tienen habilitada la coherencia entre varias máquinas virtuales. Las máquinas virtuales que forman parte de un grupo de replicación conmutan por error al último punto de recuperación común coherente con la aplicación de varias máquinas virtuales. Otras máquinas virtuales conmutan por error a su punto de recuperación coherente con la aplicación más reciente.
    - **Personalizado**: use esta opción para conmutar por error una máquina virtual específica a un punto de recuperación concreto.



## <a name="prepare-networking"></a>Preparación de las redes

Al ejecutar una conmutación por error de prueba se le pide que seleccione la configuración de red de las máquinas de réplica de prueba, tal como se resume en esta tabla.

| **Opción** | **Detalles** | |
| --- | --- | --- |
| **None** | La máquina virtual de prueba se crea en el host en el que se encuentra la máquina virtual de réplica. No se agrega a la nube y no está conectada a ninguna red.<br/><br/> La máquina se puede conectar a una red de máquinas virtuales una vez creada.| |
| **Usar existente** | La máquina virtual de prueba se crea en el host en el que se encuentra la máquina virtual de réplica. No se agrega a la nube.<br/><br/>Cree una red de máquinas virtuales aislada de la red de producción.<br/><br/>Si utiliza una red basada en VLAN, se recomienda crear una red lógica independiente en VMM para este propósito (no se utiliza en producción). Esta red lógica se utiliza para crear redes de máquinas virtuales para la conmutación por error de prueba.<br/><br/>La red lógica debe asociarse con al menos uno de los adaptadores de red de todos los servidores de Hyper-V que hospeden máquinas virtuales.<br/><br/>En el caso de las redes lógicas de VLAN, deben aislarse los sitios de red que agregue a la red lógica.<br/><br/>Si utiliza una red lógica basada en virtualización de red de Windows, Azure Site Recovery crea automáticamente redes de máquinas virtuales aisladas. | |
| **Crear una red** | Se crea automáticamente una red de prueba temporal en función de la configuración especificada en **Red lógica** y de sus sitios de red asociados.<br/><br/> La conmutación por error comprueba que se crean las máquinas virtuales.<br/><br/> Debe utilizar esta opción si el plan de recuperación usa más de una red de máquinas virtuales.<br/><br/> Si utiliza redes de virtualización de red de Windows, esta opción permite crear automáticamente redes de máquinas virtuales con la misma configuración (subredes y grupos de direcciones IP) en la red de la máquina virtual de réplica. Estas redes de máquinas virtuales se limpian automáticamente una vez completada la conmutación por error de prueba.<br/><br/> La máquina virtual de prueba se crea en el host en el que existe la máquina virtual de réplica. No se agrega a la nube.|

### <a name="best-practices"></a>Procedimientos recomendados

- Las pruebas en una red de producción conllevan un tiempo de inactividad para las cargas de trabajo de producción. Pida a los usuarios que no usen aplicaciones relacionadas mientras se realiza la exploración de recuperación ante desastres.

- La red de prueba no tiene por qué coincidir con el tipo de red lógica de VMM usado para la conmutación por error de prueba. En cambio, algunas combinaciones no funcionan:

     - Si la réplica utiliza DHCP y aislamiento basado en VLAN, la red de máquina virtual para la réplica no necesita un grupo de direcciones IP estáticas. Por lo tanto, el uso de la virtualización de red de Windows para la conmutación por error de prueba no funcionaría, ya que no hay grupos de direcciones disponibles. 
        
     - La conmutación por error de prueba no funcionará si la red de réplica no tiene aislamiento y la red de prueba utiliza la virtualización de red de Windows. Esto se debe a que una red sin aislamiento no tiene las subredes necesarias para crear una red de virtualización de red de Windows.
        
- Para la conmutación por error, le recomendamos que no use la red que seleccionó para la asignación de red.

- La manera en que las máquinas virtuales de réplica se conectan a las redes de máquinas virtuales asignadas después de la conmutación por error depende de cómo se configure la red de máquinas virtuales en la consola VMM.


### <a name="vm-network-configured-with-no-isolation-or-vlan-isolation"></a>Red de máquinas virtuales configurada sin aislamiento o con aislamiento de VLAN

Si se configura una red de máquinas virtuales en VMM sin aislamiento o con aislamiento de VLAN, tenga en cuenta lo siguiente:

- Si se define DHCP para la red de máquinas virtuales, la máquina virtual de réplica se conectará al identificador de VLAN mediante la configuración especificada para el sitio de red en la red lógica asociada. La máquina virtual recibe su dirección IP del servidor DHCP disponible.
- No es necesario definir un grupo de direcciones IP estáticas para la red de máquinas virtuales de destino. Si se utiliza un grupo de direcciones IP estáticas para la red de máquinas virtuales, la máquina virtual de réplica se conecta al identificador de VLAN mediante la configuración especificada para el sitio de red en la red lógica asociada.
- La máquina virtual recibe su dirección IP del grupo definido para la red de máquinas virtuales. Si no se ha definido un grupo de direcciones IP estáticas en la red de máquina virtual de destino, se producirá un error en la asignación de direcciones IP. Cree el grupo de direcciones IP en los servidores VMM de origen y de destino que se vayan a usar para la protección y la recuperación.

### <a name="vm-network-with-windows-network-virtualization"></a>Red de máquinas virtuales con virtualización de red de Windows

Si se configura una red de máquinas virtuales en VMM con virtualización de red de Windows, tenga en cuenta lo siguiente:

- Debe definirse un grupo estático para la red de máquinas virtuales de destino, independientemente de si la red de máquinas virtuales de origen se ha configurado para DHCP o un grupo de direcciones IP estáticas. 
- Si se define DHCP, el servidor VMM de destino actúa como servidor DHCP e indica una dirección IP del grupo definido para la red de máquinas virtuales de destino.
- Si se define el uso de un grupo de direcciones IP estáticas para el servidor de origen, el servidor VMM de destino asigna una dirección IP del grupo. En ambos casos, se producirá un error en la asignación de direcciones IP si no se define un grupo de direcciones IP estáticas.



## <a name="prepare-the-infrastructure"></a>Preparación de la infraestructura

Si simplemente desea comprobar que puede conmutar por error una máquina virtual, puede ejecutar una conmutación por error de prueba sin una infraestructura. Si desea hacer una exploración completa de la recuperación ante desastres para probar la conmutación por error de la aplicación, debe preparar la infraestructura en el sitio secundario:

- Si ejecuta una conmutación por error de prueba con una red existente, prepare Active Directory, DHCP y DNS en esa red.
- Si ejecuta una conmutación por error de prueba con la opción de crear una red de máquinas virtuales automáticamente, debe agregar los recursos de infraestructura a la red creada automáticamente antes de ejecutar la conmutación por error de prueba. En un plan de recuperación, puede facilitar esto mediante la adición de un paso manual antes del grupo 1 en el plan de recuperación que va a utilizar para la conmutación por error de prueba. A continuación, agregue los recursos de infraestructura a la red creada automáticamente antes de ejecutar la conmutación por error de prueba.


### <a name="prepare-dhcp"></a>Preparación de DHCP
Si las máquinas virtuales implicadas en la conmutación por error de prueba usan DHCP, cree un servidor DHCP de prueba dentro de la red aislada para la conmutación por error de prueba.


### <a name="prepare-active-directory"></a>Preparación de Active Directory
Para ejecutar una conmutación por error de prueba con el fin de probar la aplicación, necesita una copia del entorno de Active Directory de producción en el entorno de prueba. Para más información, consulte las [consideraciones sobre la conmutación por error de prueba para Active Directory](site-recovery-active-directory.md#test-failover-considerations).

### <a name="prepare-dns"></a>Preparación de DNS
Prepare un servidor DNS para la conmutación por error de prueba de la forma siguiente:

* **DHCP**: si las máquinas virtuales usan DHCP, debe actualizarse la dirección IP del DNS de prueba en el servidor DHCP de prueba. Si utiliza un tipo de red de virtualización de red de Windows, el servidor VMM actúa como servidor DHCP. Por lo tanto, la dirección IP de DNS debe actualizarse en la red de conmutación por error de prueba. En este caso, las máquinas virtuales se registran a sí mismas en el servidor DNS correspondiente.
* **Dirección estática**: si las máquinas virtuales usa una dirección IP estática, la dirección IP del servidor DNS de prueba debe actualizarse en la red de conmutación por error de prueba. Es  posible que deba actualizar el DNS con la dirección IP de las máquinas virtuales de prueba. Puede usar el siguiente script de ejemplo para este propósito:

  ```powershell
  Param(
  [string]$Zone,
  [string]$name,
  [string]$IP
  )
  $Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
  $newrecord = $record.clone()
  $newrecord.RecordData[0].IPv4Address  =  $IP
  Set-DnsServerResourceRecord -zonename $zone -OldInputObject $record -NewInputObject $Newrecord
  ```

## <a name="run-a-test-failover"></a>Ejecución de una conmutación por error de prueba

En este procedimiento se describe cómo ejecutar una conmutación por error de prueba para un plan de recuperación. También puede ejecutar la conmutación por error para una única máquina virtual en la pestaña **Máquinas virtuales**.

1. Seleccione **Recovery Plans** > *nombreDePlanDeRecuperación*. Haga clic en **Conmutación por error** > **Test Conmutación por error**.
2. En la hoja **Conmutación por error de prueba**, especifique cómo se deben conectar las máquinas virtuales de réplica a las redes después de la conmutación por error de prueba.
3. Realice el seguimiento del progreso de la conmutación por error en la pestaña **Trabajos** .
4. Cuando haya terminado la conmutación por error, compruebe que las máquinas virtuales se inician correctamente.
5. Cuando haya terminado, haga clic en **Cleanup test failover** (Limpieza de conmutación por error de prueba) en el plan de recuperación. En **Notas**, registre y guarde las observaciones asociadas a la conmutación por error de prueba. En este paso se eliminan las máquinas virtuales y las redes que crea Site Recovery durante la conmutación por error de prueba. 

![Conmutación por error de prueba](./media/hyper-v-vmm-test-failover/TestFailover.png)
 


> [!TIP]
> La dirección IP que se asigna a una máquina virtual durante la conmutación por error de prueba es la misma que obtendría la máquina virtual al realizar una conmutación por error planeada o no planeada (siempre que la dirección IP esté disponible en la red de conmutación por error de prueba). Si la misma dirección IP no está disponible en la red de conmutación por error de prueba, la máquina virtual recibe otra dirección IP que esté disponible en la red de conmutación por error de prueba.



### <a name="run-a-test-failover-to-a-production-network"></a>Ejecución de una conmutación por error de prueba en una red de producción

Se recomienda no ejecutar una conmutación por error de prueba en la red del sitio de recuperación de producción que especificó durante la asignación de red. Sin embargo, si realmente necesita validar la conectividad de un extremo a otro de la red en una máquina virtual que se ha conmutado por error, tenga en cuenta lo siguiente:

* Asegúrese de que la máquina virtual principal está apagada cuando se ejecuta la conmutación por error de prueba. Si no lo hace, dos máquinas virtuales con la misma identidad se ejecutarán en la misma red al mismo tiempo. Esta situación puede provocar consecuencias no deseadas.
* Al limpiar las máquinas virtuales de la conmutación por error de prueba, se perderán los cambios realizados en las máquinas virtuales de la conmutación por error de prueba. Estos cambios no se replican de vuelta en las máquinas virtuales principales.
* Realizar las pruebas de este modo conllevará un tiempo de inactividad de la aplicación de producción. Solicite a los usuarios de la aplicación que no la usen con la exploración de la recuperación ante desastres en curso.  


## <a name="next-steps"></a>Pasos siguientes
Después de ejecutar correctamente una exploración de la recuperación ante desastres, puede [ejecutar una conmutación por error completa](site-recovery-failover.md).



