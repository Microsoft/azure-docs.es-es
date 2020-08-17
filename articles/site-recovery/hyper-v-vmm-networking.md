---
title: Configuración del direccionamiento IP después de una conmutación por error a un sitio secundario con Azure Site Recovery
description: Se describe cómo configurar el direccionamiento IP para conectarse a las máquinas virtuales de un sitio local secundario tras la recuperación ante desastres y la conmutación por error con Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/12/2019
ms.author: raynew
ms.openlocfilehash: 43942c20a353ff69383f3e721679e4c95ab9d230
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495950"
---
# <a name="set-up-ip-addressing-to-connect-to-a-secondary-on-premises-site-after-failover"></a>Configuración del direccionamiento IP para conectarse a un sitio secundario local tras la conmutación por error

Después de conmutar las máquinas virtuales de Hyper-V en nubes de System Center Virtual Machine Manager (VMM) a un sitio secundario, debe ser capaz de conectarse a las máquinas virtuales de réplica. En este artículo le ayudará a hacerlo. 

## <a name="connection-options"></a>Opciones de conexión

Después de la conmutación por error, hay un par de formas de controlar el direccionamiento IP para las máquinas virtuales de réplica: 

- **Conservar la misma dirección IP después de la conmutación por error**: en este escenario, la máquina virtual replicada tiene la misma dirección IP que la máquina virtual principal. Esto simplifica los problemas relacionados con la red después de la conmutación por error, pero requiere algún trabajo de infraestructura.
- **Usar una dirección IP diferente después de la conmutación por error**: en este escenario, la máquina virtual obtiene una nueva dirección IP después de la conmutación por error. 
 

## <a name="retain-the-ip-address"></a>Conservación de la dirección IP

Si desea conservar las direcciones IP del sitio principal en el sitio secundario tras la conmutación por error, puede hacer lo siguiente:

- Implementar una subred extendida entre los sitios principal y secundario.
- Ejecutar una conmutación por error completa de la subred desde el sitio principal al secundario. Debe actualizar las rutas para indicar la nueva ubicación de las direcciones IP.


### <a name="deploy-a-stretched-subnet"></a>Implementación de una subred extendida

En una configuración extendida, la subred está disponible a la vez en el sitio principal y el secundario. En una subred extendida, cuando se mueven una máquina y su configuración de dirección IP (Capa 3) al sitio secundario, la red enruta automáticamente el tráfico a la nueva ubicación. 

- Desde una perspectiva del nivel 2 (nivel de vínculo de datos), necesita un equipo de redes que pueda administrar una red VLAN estirada.
- Mediante la extensión de la VLAN, el dominio de error potencial se extiende a ambos sitios. Esto lo convierte en un único punto de error. Si bien es improbable, en este escenario es posible que no pueda aislar un incidente, como una tormenta de difusión. 


### <a name="fail-over-a-subnet"></a>Conmutación por error de una subred

Puede ejecutar una conmutación por error de toda la subred para aprovechar las ventajas de la subred extendida, sin extenderla realmente. En esta solución, una subred está disponible en el sitio de origen o en el de destino, pero no en ambos a la vez.

- Para mantener el espacio de direcciones IP en el caso de una conmutación por error, puede organizar mediante programación la infraestructura del enrutador para mover las subredes de un sitio a otro.
- Cuando se produce la conmutación por error, las subredes se mueven con sus máquinas virtuales asociadas.
- El principal inconveniente de este enfoque es que es necesario mover la subred completa si se produce un error.

#### <a name="example"></a>Ejemplo

Este es un ejemplo de conmutación por error de la subred completa. 

- Antes de la conmutación por error, el sitio principal tiene aplicaciones que se ejecutan en la subred 192.168.1.0/24.
- Durante la conmutación por error, todas las máquinas virtuales de esta subred se conmutan por error al sitio secundario y conservan sus direcciones IP. 
- Es necesario modificar las rutas para reflejar el hecho de que todas las máquinas virtuales de la subred 192.168.1.0/24 se han movido al sitio secundario.

En los gráficos siguientes se muestran las subredes antes y después de la conmutación por error.


**Antes de la conmutación por error**

![Diagrama que muestra las subredes antes de la conmutación por error.](./media/hyper-v-vmm-networking/network-design2.png)

**Después de la conmutación por error**

![Diagrama que muestra las subredes después de la conmutación por error.](./media/hyper-v-vmm-networking/network-design3.png)

Después de la conmutación por error, Site Recovery asigna una dirección IP para cada interfaz de red de la máquina virtual. La dirección se asigna desde el grupo de direcciones IP estáticas de la red correspondiente para cada instancia de máquina virtual.

- Si el grupo de direcciones IP del sitio secundario es el mismo que el del sitio de origen, Site Recovery asigna la misma dirección IP (de la máquina virtual de origen) a la máquina virtual de réplica. La dirección IP está reservada en VMM, pero no está establecida como dirección IP de conmutación por error en el host de Hyper-V. La dirección IP de conmutación por error de un host de Hyper-V se establece inmediatamente antes de la conmutación por error.
- Si no está disponible la misma dirección IP, Site Recovery asigna otra dirección IP disponible del grupo.
- Si las máquinas virtuales usan DHCP, Site Recovery no administra las direcciones IP. Es necesario que compruebe que el servidor DHCP del sitio secundario puede asignar direcciones desde el mismo intervalo que el sitio de origen.

### <a name="validate-the-ip-address"></a>Validar la dirección IP

Después de habilitar la protección para una máquina virtual, puede usar el siguiente script de ejemplo para comprobar la dirección asignada a la máquina virtual. Esta dirección IP se establece como dirección IP de conmutación por error y se asigna a la máquina virtual en el momento de la conmutación por error:

```powershell
$vm = Get-SCVirtualMachine -Name <VM_NAME>
$na = $vm[0].VirtualNetworkAdapters>
$ip = Get-SCIPAddress -GrantToObjectID $na[0].id
$ip.address
```

## <a name="use-a-different-ip-address"></a>Uso de una dirección IP distinta

En este escenario, se cambian las direcciones IP de las máquinas virtuales que se conmutan por error. El inconveniente de esta solución es el mantenimiento necesario.  Las entradas de DNS y caché deben actualizarse. Esto puede producir un tiempo de inactividad, que puede ser mitigada como sigue:

- Utilice valores de TTL bajos para las aplicaciones de intranet.
- Utilice el siguiente script en un plan de recuperación de Site Recovery para una actualización puntual del servidor DNS. No necesita el script si usa un registro de DNS dinámico.

    ```powershell
    param(
    string]$Zone,
    [string]$name,
    [string]$IP
    )
    $Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
    $newrecord = $record.clone()
    $newrecord.RecordData[0].IPv4Address  =  $IP
    Set-DnsServerResourceRecord -zonename $zone -OldInputObject $record -NewInputObject $Newrecord
    ```
    
### <a name="example"></a>Ejemplo 

En este ejemplo, tenemos diferentes direcciones IP en los sitios principal y secundario y existe un tercer sitio desde el que se puede tener acceso a las aplicaciones hospedadas en el sitio principal o de recuperación.

- Antes de la conmutación por error, las aplicaciones están hospedadas en la subred 192.168.1.0/24 del sitio principal.
- Después de la conmutación por error, las aplicaciones se configuran en la subred 172.16.1.0/24 del sitio secundario.
- Los tres sitios tienen acceso entre ellos.
- Después de la conmutación por error, las aplicaciones se restaurarán en la subred de recuperación.
- En este escenario no es necesario conmutar por error toda la subred y no se requiere ningún cambio para volver a configurar las rutas de red o VPN. La conmutación por error y algunas actualizaciones de DNS garantizan que las aplicaciones sigan siendo accesibles.
- Si el DNS está configurado para permitir actualizaciones dinámicas, las máquinas virtuales se registrarán a sí mismas con la nueva dirección IP al iniciarse después de la conmutación por error.

**Antes de la conmutación por error**

![Diagrama que muestra distintas direcciones IP antes de la conmutación por error.](./media/hyper-v-vmm-networking/network-design10.png)

**Después de la conmutación por error**

![Diagrama que muestra distintas direcciones IP después de la conmutación por error.](./media/hyper-v-vmm-networking/network-design11.png)


## <a name="next-steps"></a>Pasos siguientes

[Ejecución de una conmutación por error](hyper-v-vmm-failover-failback.md)

