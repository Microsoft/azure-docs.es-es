---
title: Ejecución de una recuperación ante desastres de prueba (simulacro de recuperación ante desastres) en Azure en Azure Site Recovery
description: Obtenga información sobre cómo ejecutar una conmutación por error de prueba desde su entorno local a Azure con el servicio Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: 4233df62de48dd7a7253c488b0cd69c38cd8d445
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503504"
---
# <a name="run-a-test-failover-disaster-recovery-drill-to-azure"></a>Ejecución de una recuperación ante desastres de prueba (simulacro de recuperación ante desastres) en Azure 


En este artículo se describe cómo obtener detalles de recuperación ante desastres en Azure mediante una conmutación por error de prueba de Site Recovery.  

La conmutación por error de prueba se ejecuta para validar la estrategia de replicación y recuperación ante desastres sin perder datos ni tiempo de actividad. La conmutación por error de prueba no afecta a la replicación en curso ni al entorno de producción. Puede ejecutar una conmutación por error de prueba en una máquina virtual (VM) específica o en un [plan de recuperación](site-recovery-create-recovery-plans.md) que contenga varias máquinas virtuales.


## <a name="run-a-test-failover"></a>Ejecución de una conmutación por error de prueba
En este procedimiento se describe cómo ejecutar una conmutación por error de prueba para un plan de recuperación. Si desea ejecutar una conmutación por error de prueba para una sola máquina virtual, siga los pasos descritos [aquí](tutorial-dr-drill-azure.md#run-a-test-failover-for-a-single-vm)

![Captura de pantalla de la página Conmutación por error de prueba en Azure Portal.](./media/site-recovery-test-failover-to-azure/TestFailover.png)


1. En Site Recovery en Azure Portal, haga clic en **Planes de recuperación** > *recoveryplan_name* > **Conmutación por error de prueba**.
2. Seleccione un **Punto de recuperación** en el que realizar la conmutación por error. Puede seleccionar una de las siguientes opciones:
    - **Procesado más recientemente**: esta opción realiza una conmutación por error de todas las máquinas virtuales al último punto de recuperación procesado por Site Recovery. Para ver el último punto de recuperación de una máquina virtual específica, seleccione **Puntos de recuperación más recientes** en la configuración de la máquina virtual. Esta opción proporciona un objetivo de tiempo de recuperación (RTO) bajo, ya que no se invierte tiempo en el procesamiento de datos sin procesar.
    - **Más reciente coherente con la aplicación**: esta opción conmuta por error todas las máquinas virtuales del plan al punto de recuperación más reciente coherente con la aplicación que haya procesado Site Recovery. Para ver el último punto de recuperación de una máquina virtual específica, seleccione **Puntos de recuperación más recientes** en la configuración de la máquina virtual.
    - **Más reciente**: esta opción procesa primero todos los datos que se han enviado al servicio Site Recovery para crear un punto de recuperación para cada máquina virtual antes de conmutarla por error a dicho punto de recuperación. Esta opción ofrece el objetivo de punto de recuperación (RPO) mínimo, ya que la máquina virtual de Azure creada después de la conmutación por error tiene todos los datos replicados en Site Recovery al desencadenarse la conmutación por error.
    - **Último procesamiento de máquinas virtuales múltiples**: esta opción está disponible para los planes de recuperación con una o varias máquinas virtuales que tienen habilitada la coherencia entre varias máquinas virtuales. Máquinas virtuales con la conmutación por error habilitada para el último punto de recuperación común coherente con varias máquinas virtuales. Otras máquinas virtuales realizan la conmutación por error al último punto de recuperación procesado.  
    - **Último coherente con aplicación de múltiples VM**: esta opción está disponible para los planes de recuperación con una o varias máquinas virtuales que tienen habilitada la coherencia entre varias máquinas virtuales. Las máquinas virtuales que forman parte de un grupo de replicación conmutan por error al último punto de recuperación común coherente con la aplicación de varias máquinas virtuales. Otras máquinas virtuales conmutan por error a su punto de recuperación coherente con la aplicación más reciente.
    - **Personalizado**: use esta opción para conmutar por error una máquina virtual específica a un punto de recuperación concreto.
3. Seleccione la red virtual de Azure en la que se crearán las máquinas virtuales de prueba.

    - Site Recovery intentará crear las máquinas virtuales de prueba en una subred con el mismo nombre y dirección IP que se proporcionaron en la configuración de **Proceso y red** de la máquina virtual.
    - Si no hay ninguna subred con el mismo nombre disponible en la red virtual de Azure utilizada para la conmutación por error de prueba, la máquina virtual de prueba se creará en la primera subred que aparezca por orden alfabético.
    - Si la misma dirección IP no está disponible en la subred, la máquina virtual recibirá otra dirección IP que sí lo esté. [Más información](#create-a-network-for-test-failover).
4. Si realiza la conmutación por error a Azure y el cifrado de datos está habilitado, en **Clave de cifrado** seleccione el certificado que se emitió cuando habilitó el cifrado durante la instalación del proveedor. Puede omitir este paso si el cifrado no está habilitado.
5. Realice el seguimiento del progreso de la conmutación por error en la pestaña **Trabajos** . Debe poder ver la máquina de réplica de prueba en el Portal de Azure.
6. Para iniciar una conexión RDP con la máquina virtual de Azure, debe [agregar una dirección IP pública](https://aka.ms/addpublicip) en la interfaz de red de la máquina virtual conmutada por error.
7. Cuando todo funciona según lo esperado, haga clic en **Limpiar conmutación por error de prueba**. Así se eliminarán las máquinas virtuales que se crearon durante la conmutación por error de prueba.
8. En **Notas**, registre y guarde las observaciones asociadas a la conmutación por error de prueba.


![Captura de pantalla de la pestaña Trabajos de Conmutación por error de prueba.](./media/site-recovery-test-failover-to-azure/TestFailoverJob.png)

Cuando se desencadena una conmutación por error de prueba, ocurre lo siguiente:

1. **Requisitos previos**: se ejecuta una comprobación de requisitos previos para garantizar que se dan todas las condiciones necesarias para la conmutación por error.
2. **Conmutación por error**: la conmutación por error procesa y prepara los datos, por lo que se puede crear una máquina virtual de Azure a partir de ellos.
3. **Más reciente**: si ha elegido el punto de recuperación más reciente, se crea un punto de recuperación de los datos enviados al servicio.
4. **Iniciar**: en este paso se crea una máquina virtual de Azure con los datos procesados en el paso anterior.

### <a name="failover-timing"></a>Temporización de la conmutación por error

En los escenarios siguientes, la conmutación por error requiere un paso adicional intermedio que normalmente tarda alrededor de 8 a 10 minutos en completarse:

* Máquinas virtuales de VMware que ejecutan una versión de Mobility Service anterior a 9.8
* Servidores físicos
* Máquinas virtuales de VMware Linux
* Máquina virtual de Hyper-V protegida como servidor físico
* Máquina virtual de VMware en la que los siguientes controladores no son controladores de arranque:
    * storvsc
    * vmbus
    * storflt
    * intelide
    * atapi
* Máquina virtual de VMware que no tiene DHCP habilitado, independientemente de si usa DHCP o direcciones IP estáticas.

En todos los demás casos, no es necesario ningún paso intermedio y el tiempo necesario para la conmutación por error es significativamente menor.


## <a name="create-a-network-for-test-failover"></a>Creación de una red para conmutación por error de prueba

Se recomienda que para realizar una conmutación por error de prueba, elija una red que esté aislada de la del sitio de recuperación de producción concreto proporcionada en la configuración de **Proceso y red** de cada máquina virtual. De forma predeterminada, cualquier red virtual que se cree en Azure está aislada de otras redes. La red de prueba debe imitar la de producción:

- La red de prueba debe tener el mismo número de subredes que la de producción. Las subredes deben tener los mismos nombres.
- La red de prueba debe usar el mismo intervalo de direcciones IP.
- Actualice el DNS de la red de prueba con la dirección IP especificada para la máquina virtual de DNS en la configuración de **Proceso y red**. Lea [Consideraciones sobre la conmutación por error de prueba para Active Directory](site-recovery-active-directory.md#test-failover-considerations) para más información.


## <a name="test-failover-to-a-production-network-in-the-recovery-site"></a>Conmutación por error de prueba a una red de producción en el sitio de recuperación

Aunque se recomienda que utilice una red de prueba independiente de la red de producción, si desea probar una exploración de recuperación ante desastres en la red de producción, tenga en cuenta lo siguiente:

- Asegúrese de que la máquina virtual principal está apagada cuando se ejecuta la conmutación por error de prueba. Si no lo está, se ejecutarán al mismo tiempo dos máquinas virtuales con la misma identidad en la misma red. Esto puede provocar consecuencias inesperadas.
- Los cambios realizados en las máquinas virtuales creadas para la conmutación por error de prueba se perderán al limpiar la conmutación por error. Estos cambios no se replican en la máquina virtual principal.
- La realización de pruebas en el entorno de producción provocará un tiempo de inactividad en la aplicación de producción. Los usuarios no deben utilizar aplicaciones que se ejecuten en máquinas virtuales si la conmutación por error de prueba está en curso.  



## <a name="prepare-active-directory-and-dns"></a>Preparación de Active Directory y DNS

Para ejecutar una conmutación por error de prueba con el fin de probar la aplicación, necesita una copia del entorno de Active Directory de producción en el entorno de prueba. Lea [consideraciones de la conmutación por error de prueba para Active Directory](site-recovery-active-directory.md#test-failover-considerations) para más información.

## <a name="prepare-to-connect-to-azure-vms-after-failover"></a>Preparación para la conexión a las máquinas virtuales de Azure después de la conmutación por error

Si desea conectarse a máquinas virtuales de Azure mediante RDP/SSH después de la conmutación por error, siga los requisitos resumidos en la tabla.

**Conmutación por error** | **Ubicación** | **Acciones**
--- | --- | ---
**Máquina virtual de Azure que ejecuta Windows** | En la máquina local antes de la conmutación por error | Para acceder a la máquina virtual de Azure a través de Internet, habilite RDP, asegúrese de que se hayan agregado las reglas de TCP y UDP para **Público**, y que RDP esté permitido en **Firewall de Windows** > **Aplicaciones permitidas**, para todos los perfiles.<br/><br/> Para acceder a la máquina virtual de Azure a través de una conexión de sitio a sitio, habilite el protocolo RDP en la máquina y asegúrese de que este está permitido en **Firewall de Windows** -> **Aplicaciones y características permitidas** para redes **Dominio y Privadas**.<br/><br/>  Asegúrese de que la directiva SAN del sistema operativo está establecida en **OnlineAll**. [Más información](https://support.microsoft.com/kb/3031135).<br/><br/> Asegúrese de que no haya actualizaciones de Windows pendientes en la máquina virtual cuando se desencadena una conmutación por error. La actualización de Windows puede que comience cuando realice la conmutación por error y no podrá iniciar sesión en la máquina virtual hasta que se complete la actualización.
**Máquina virtual de Azure que ejecuta Windows** | Máquina virtual de Azure después de la conmutación por error |  [Agregue una dirección IP pública](https://aka.ms/addpublicip) para la máquina virtual.<br/><br/> Las reglas del grupo de seguridad de red de la máquina virtual conmutada por error (y la subred de Azure a la que esta se conecta) deben permitir las conexiones entrantes al puerto RDP.<br/><br/> Seleccione **Diagnósticos de arranque** para comprobar una captura de pantalla de la máquina virtual.<br/><br/> Si no puede conectarse, compruebe que se está ejecutando la máquina virtual y revise estas [sugerencias de solución de problemas](https://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx).
**Máquina virtual de Azure que ejecuta Linux** | En la máquina local antes de la conmutación por error | Asegúrese de que el servicio de Secure Shell en la máquina virtual está configurado para iniciarse automáticamente en el arranque del sistema.<br/><br/> Compruebe que las reglas de firewall permiten una conexión SSH.
**Máquina virtual de Azure que ejecuta Linux** | Máquina virtual de Azure después de la conmutación por error | Las reglas del grupo de seguridad de red de la máquina virtual conmutada por error (y la subred de Azure a la que esta se conecta) necesitan permitir las conexiones entrantes al puerto SSH.<br/><br/> [Agregue una dirección IP pública](https://aka.ms/addpublicip) para la máquina virtual.<br/><br/> Seleccione **Diagnósticos de arranque** para obtener una captura de pantalla de la máquina virtual.<br/><br/>

Siga los pasos descritos [aquí](site-recovery-failover-to-azure-troubleshoot.md) para solucionar problemas de conectividad tras la conmutación por error.

## <a name="next-steps"></a>Pasos siguientes
Después de haber completado una exploración de recuperación ante desastres, obtenga más información sobre otros tipos de [conmutación por error](site-recovery-failover.md).
