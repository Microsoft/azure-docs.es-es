---
title: Arquitectura de recuperación ante desastres del servidor físico en Azure Site Recovery
description: En este artículo se proporciona información general de los componentes y la arquitectura que se usan durante la recuperación ante desastres de servidores físicos locales en Azure con el servicio Azure Site Recovery.
ms.topic: conceptual
ms.date: 02/11/2020
ms.openlocfilehash: f2184654a8169cb353fb40fa76f0a7fe9b3df6f6
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422664"
---
# <a name="physical-server-to-azure-disaster-recovery-architecture"></a>Arquitectura de recuperación ante desastres de un servidor físico en Azure

En este artículo se describen la arquitectura y los procesos que se usan al replicar, conmutar por error y recuperar servidores físicos Windows y Linux entre un sitio local y Azure, mediante el servicio [Azure Site Recovery](site-recovery-overview.md).

## <a name="architectural-components"></a>Componentes de la arquitectura

En la tabla y el gráfico siguientes se proporciona una visión general de los componentes que se usan para la replicación física de servidores en Azure.

| **Componente** | **Requisito** | **Detalles** |
| --- | --- | --- |
| **Azure** | Una suscripción y una red de Azure. | Los datos replicados desde máquinas físicas locales se almacenan en discos administrados de Azure. Las máquinas virtuales de Azure se crean con los datos replicados cuando se ejecuta una conmutación por error desde el entorno local en Azure. Las máquinas virtuales de Azure se conectan a la red virtual de Azure cuando se crean. |
| **Servidor de proceso** | Se instala de forma predeterminada junto con el servidor de configuración. | Actúa como puerta de enlace de replicación. Recibe datos de replicación, los optimiza con almacenamiento en caché, compresión y cifrado y los envía al almacenamiento de Azure.<br/><br/> El servidor de procesos también instala Mobility Service en los servidores que desee replicar.<br/><br/> A medida que crece la implementación, puede agregar más servidores de procesos independientes para controlar mayores volúmenes de tráfico de replicación. |
| **Servidor de destino principal** | Se instala de forma predeterminada junto con el servidor de configuración. | Controla los datos de replicación durante la conmutación por recuperación desde Azure.<br/><br/> En el caso de las implementaciones de gran tamaño, puede agregar un servidor de destino maestro independiente adicional para la conmutación por recuperación. |
| **Servidores replicados** | Mobility Service se instala en cada servidor que se replique. | Se recomienda permitir la instalación automática desde el servidor de procesos. No obstante, también puede instalar manualmente el servicio o usar un método de implementación automatizada, como Configuration Manager. |

**Arquitectura de dispositivo físico a Azure**

![Componentes](./media/physical-azure-architecture/arch-enhanced.png)

## <a name="set-up-outbound-network-connectivity"></a>Configuración de la conectividad de red saliente

Para que Site Recovery funcione de la forma esperada, debe modificar la conectividad de red saliente para permitir la replicación de su entorno.

> [!NOTE]
> Site Recovery no admite el uso de un proxy de autenticación para controlar la conectividad de la red.

### <a name="outbound-connectivity-for-urls"></a>Conectividad de salida para las direcciones URL

Si usa un proxy de firewall basado en direcciones URL para controlar la conectividad de salida, debe permitir el acceso a ellas:

| **Nombre**                  | **Comercial**                               | **Gobierno**                                 | **Descripción** |
| ------------------------- | -------------------------------------------- | ---------------------------------------------- | ----------- |
| Storage                   | `*.blob.core.windows.net`                  | `*.blob.core.usgovcloudapi.net`               | Permite que los datos se puedan escribir desde la máquina virtual a la cuenta de almacenamiento de caché en la región de origen. |
| Azure Active Directory    | `login.microsoftonline.com`                | `login.microsoftonline.us`                   | Proporciona autorización y autenticación de las direcciones URL del servicio Site Recovery. |
| Replicación               | `*.hypervrecoverymanager.windowsazure.com` | `*.hypervrecoverymanager.windowsazure.com`   | Permite que la máquina virtual se comunique con el servicio Site Recovery. |
| Azure Service Bus               | `*.servicebus.windows.net`                 | `*.servicebus.usgovcloudapi.net`             | Permite que la máquina virtual escriba los datos de diagnóstico y supervisión de Site Recovery. |


## <a name="replication-process"></a>Proceso de replicación

1. Configure la implementación, incluidos los componentes de Azure y locales. En el almacén de Recovery Services, especifica el origen y el destino de la replicación, configura el servidor de configuración, crea una directiva de replicación y habilita la replicación.
1. Las máquinas se replican mediante la directiva de replicación, y una copia inicial de los datos del servidor se replica en Azure Storage.
1. Una vez terminada la replicación inicial, comienza la de los cambios incrementales en Azure. Las marcas de revisión de una máquina se conservan en un archivo con la extensión _.hrl_.
   - Las máquinas se comunican con el servidor de configuración en el puerto HTTPS 443 de entrada para la administración de la replicación.
   - Las máquinas envían los datos de replicación al servidor de procesos en el puerto HTTPS 9443 de entrada (se puede modificar).
   - El servidor de configuración organiza la administración de la replicación con Azure a través del puerto HTTPS 443 de salida.
   - El servidor de procesos recibe datos de las máquinas de origen, los optimiza y los cifra para enviarlos después a Azure Storage a través del puerto HTTPS 443 de salida.
   - Si habilita la coherencia entre varias máquinas virtuales, las máquinas del grupo de replicación se comunican entre sí a través del puerto 20004. La implementación de varias máquinas virtuales se usa si agrupa varias máquinas en grupos de replicación que tienen puntos de recuperación coherentes con los bloqueos y coherentes con la aplicación cuando conmutan por error. Estos grupos resultan útiles si las máquinas ejecutan la misma carga de trabajo y deben ser coherentes.
1. El tráfico se replica en los puntos de conexión públicos del almacenamiento de Azure a través de Internet. Como alternativa, puede usar el [emparejamiento público](../expressroute/about-public-peering.md) de Azure ExpressRoute.

   > [!NOTE]
   > No se admite la replicación a través de una VPN de sitio a sitio desde un sitio local o un [emparejamiento privado](concepts-expressroute-with-site-recovery.md#on-premises-to-azure-replication-with-expressroute) de Azure ExpressRoute.

**Proceso de replicación de dispositivo físico a Azure**

![Proceso de replicación](./media/physical-azure-architecture/v2a-architecture-henry.png)

## <a name="failover-and-failback-process"></a>Proceso de conmutación por error y conmutación por recuperación

Una vez que la replicación está configurada, puede ejecutar una exploración de la recuperación ante desastres (conmutación por error de prueba) para comprobar que todo funciona según lo previsto. Después, puede realizar la conmutación por error y la conmutación por recuperación, según sea necesario. Considere los siguientes aspectos:

- No se admite la conmutación por error planeada.
- Es necesario realizar la conmutación por recuperación en una máquina virtual de VMware local. Necesita una infraestructura de VMware local, aunque replique servidores físicos locales en Azure.
- La conmutación por error se realiza en una sola máquina, o bien se crean planes de recuperación para realizar la conmutación por error en varias máquinas a la vez.
- Cuando se ejecuta una conmutación por error, se crean máquinas virtuales de Azure a partir de los datos replicados en Azure Storage.
- Una vez desencadenada la conmutación por error inicial, debe confirmarla para que se inicie. Para ello, acceda a la carga de trabajo desde la máquina virtual de Azure.
- Cuando el sitio local principal esté disponible de nuevo, podrá realizar una conmutación por recuperación.
- Configure una infraestructura de conmutación por recuperación que incluya:
  - **Servidor de procesos temporal de Azure**: para realizar una conmutación por recuperación desde Azure, debe configurar una máquina virtual de Azure para que actúe como servidor de procesos y controle la replicación desde Azure. Dicha máquina virtual se puede eliminar cuando finalice la conmutación por recuperación.
  - **Conexión VPN**: necesita una conmutación VPN o Azure ExpressRoute desde la red de Azure al sitio local.
  - **Servidor de destino maestro independiente**: de manera predeterminada, la conmutación por recuperación la controla el servidor de destino maestro que se instaló con el servidor de configuración en la máquina virtual local de VMware. Si necesita realizar la conmutación por recuperación en grandes volúmenes de tráfico, debe configurar un servidor de destino maestro local independiente.
  - **Directiva de conmutación por recuperación**: para replicar de nuevo en el sitio local, necesita una directiva de conmutación por recuperación. La directiva se creó automáticamente durante la creación de la directiva de replicación entre el entorno local y Azure.
  - **Infraestructura de VMware**: Para las conmutaciones por recuperación, se necesita una infraestructura de VMware. No se puede realizar la conmutación por recuperación a un servidor físico.
- Una vez instalados los componentes, la conmutación por recuperación se produce en tres fases:
  - **Fase 1:** : Vuelva a proteger las máquinas virtuales de modo que realicen la replicación desde Azure de vuelta a las máquinas virtuales VMware locales.
  - **Fase 2**: Ejecute una conmutación por error en el sitio local.
  - **Fase 3**: Una vez que las cargas de trabajo realizaron la conmutación por recuperación, vuelve a habilitar la replicación.

**Conmutación por recuperación VMware desde Azure**

![Conmutación por recuperación](./media/physical-azure-architecture/enhanced-failback.png)

## <a name="next-steps"></a>Pasos siguientes

Para configurar la recuperación ante desastres de servidores físicos en Azure, consulte la [guía paso a paso](physical-azure-disaster-recovery.md).
