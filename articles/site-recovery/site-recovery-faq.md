---
title: Preguntas generales sobre el servicio Azure Site Recovery
description: En este artículo se analizan las preguntas generales más frecuentes acerca de Azure Site Recovery.
ms.topic: conceptual
ms.date: 7/14/2020
ms.author: raynew
ms.openlocfilehash: d77f62a57a75f13589b11e023f902c1a128a0d95
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88950500"
---
# <a name="general-questions-about-azure-site-recovery"></a>Preguntas generales acerca de Azure Site Recovery

En este artículo se resumen las preguntas más frecuentes sobre Azure Site Recovery. Para escenarios específicos, revise estos artículos

- [Preguntas acerca de la recuperación ante desastres de máquinas virtuales de Azure en Azure](azure-to-azure-common-questions.md)
- [Preguntas acerca de la recuperación ante desastres de máquinas virtuales de VMware en Azure](vmware-azure-common-questions.md)
- [Preguntas acerca de la recuperación ante desastres de máquinas virtuales de Hyper-V en Azure](hyper-v-azure-common-questions.md)
 
## <a name="general"></a>General

### <a name="what-does-site-recovery-do"></a>¿Qué hace Site Recovery?

Site Recovery contribuye a su estrategia de continuidad empresarial (BCDR por su sigla en inglés) y recuperación ante desastres mediante la coordinación y la automatización de la replicación de máquinas virtuales de Azure entre regiones, máquinas virtuales locales y servidores físicos a Azure, y máquinas locales a un centro de datos secundario. [Más información](site-recovery-overview.md).

### <a name="can-i-protect-a-virtual-machine-that-has-a-docker-disk"></a>¿Puedo proteger una máquina virtual que tenga un disco Docker?

No, se trata de un escenario no admitido.

### <a name="what-does-site-recovery-do-to-ensure-data-integrity"></a>¿Qué hace Site Recovery para garantizar la integridad de los datos?

Site Recovery toma varias medidas para garantizar la integridad de los datos. Se establece una conexión segura entre todos los servicios mediante el protocolo HTTPS. Esto garantiza que cualquier malware o entidad externa no pueda alterar los datos. Otra medida tomada es usar sumas de comprobación. La transferencia de datos entre el origen y el destino se ejecuta mediante el cálculo de sumas de comprobación de datos entre ellas. Esto garantiza que los datos transferidos sean coherentes.

## <a name="service-providers"></a>Proveedores de servicios

### <a name="im-a-service-provider-does-site-recovery-work-for-dedicated-and-shared-infrastructure-models"></a>Soy un proveedor de servicios. ¿Site Recovery funciona para los modelos de infraestructura compartida o específica?
Sí, Site Recovery admite ambos modelos de infraestructura, dedicados y compartidos.

### <a name="for-a-service-provider-is-the-identity-of-my-tenant-shared-with-the-site-recovery-service"></a>Para un proveedor de servicios, ¿la identidad de mi inquilino se comparte con el servicio Site Recovery?
No. La identidad del inquilino permanece anónima. Los inquilinos no necesitan acceso al portal de Site Recovery. Solo el administrador del proveedor de servicios realiza acciones en el portal.

### <a name="will-tenant-application-data-ever-go-to-azure"></a>¿Los datos de la aplicación de mis inquilinos llegarán a Azure?
Cuando se replica entre sitios que pertenecen al proveedor de servicios, los datos de la aplicación nunca llegan a Azure. Los datos se cifran en tránsito y se replican directamente entre los sitios del proveedor de servicios.

Si está replicando a Azure, los datos de la aplicación se envían al almacenamiento de Azure, pero no al servicio Site Recovery. Los datos se cifran en tránsito y permanecen cifrados en Azure.

### <a name="will-my-tenants-receive-a-bill-for-any-azure-services"></a>¿Recibirán mis inquilinos una factura por los servicios de Azure?
No. La relación de facturación de Azure se entabla directamente con el proveedor de servicios. Los proveedores de servicios son responsables de generar facturas específicas para sus inquilinos.

### <a name="if-im-replicating-to-azure-do-we-need-to-run-virtual-machines-in-azure-at-all-times"></a>Si se está replicando a Azure, ¿es necesario ejecutar máquinas virtuales en Azure en todo momento?
No, los datos se replican en Azure Storage en su suscripción. Al realizar una conmutación por error de prueba (obtención de detalles de recuperación ante desastres) o una conmutación por error real, Site Recovery crea automáticamente las máquinas virtuales en su suscripción.

### <a name="do-you-ensure-tenant-level-isolation-when-i-replicate-to-azure"></a>¿Se puede garantizar el aislamiento a nivel de inquilino al replicar a Azure?
Sí.

### <a name="what-platforms-do-you-currently-support"></a>¿Qué plataformas se admiten actualmente?
Se admite Azure Pack, el sistema de plataforma en la nube e implementaciones basadas en System Center (2012 y superiores). [Más información](/previous-versions/azure/windows-server-azure-pack/dn850370(v=technet.10)) acerca de la integración de Azure Pack y Site Recovery.

### <a name="do-you-support-single-azure-pack-and-single-vmm-server-deployments"></a>¿Se admite un paquete Azure Pack sencillo e implementaciones de servidor VMM individuales?
Sí, es posible replicar máquinas virtuales de Hyper-V en Azure, o entre sitios del proveedor de servicios.  Tenga en cuenta que si se replica entre sitios del proveedor de servicios, la integración de runbooks de Azure no estará disponible.

## <a name="pricing"></a>Precios

### <a name="where-can-i-find-pricing-information"></a>¿Dónde puedo encontrar información sobre precios?
Consulte los detalles de los [precios de Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).


### <a name="how-can-i-calculate-approximate-charges-during-the-use-of-site-recovery"></a>¿Cómo puedo calcular los cargos aproximados durante el uso de Site Recovery?

Puede usar la [calculadora de precios](https://aka.ms/asr_pricing_calculator) para estimar los costos al usar Site Recovery.

Para obtener una estimación detallada de los costos, ejecute la herramienta Deployment Planner para [VMware](https://aka.ms/siterecovery_deployment_planner) o [Hyper-V](https://aka.ms/asr-deployment-planner) y use el [informe de estimación de costos](https://aka.ms/asr_DP_costreport).


### <a name="managed-disks-are-now-used-to-replicate-vmware-vms-and-physical-servers-do-i-incur-additional-charges-for-the-cache-storage-account-with-managed-disks"></a>Ahora se usan discos administrados para replicar máquinas virtuales y servidores físicos de VMware. ¿Se producen cargos adicionales para la cuenta de almacenamiento de caché con discos administrados?

No, no hay cargos adicionales para la caché. Cuando se replica a la cuenta de almacenamiento estándar, este almacenamiento en caché forma parte de la misma cuenta de almacenamiento de destino.

### <a name="i-have-been-an-azure-site-recovery-user-for-over-a-month-do-i-still-get-the-first-31-days-free-for-every-protected-instance"></a>He sido usuario de Azure Site Recovery durante más de un mes. ¿Los treinta y un primeros días serán gratuitos para cada instancia protegida?

Sí. Ninguna instancia protegida incurre en cargos por Azure Site Recovery durante los primeros 31 días. Por ejemplo, si ha tenido protegidas 10 instancias durante los últimos 6 meses y conecta una 11.ª instancia a Azure Site Recovery, esta instancia no incurrirá en cargos durante los primeros 31 días. Las 10 primeras instancias continúan incurriendo en cargos por Azure Site Recovery porque han estado protegidas durante más de 31 días.

### <a name="during-the-first-31-days-will-i-incur-any-other-azure-charges"></a>Durante los primeros 31 días, ¿puedo incurrir en otros cargos de Azure?

Sí, aunque Site Recovery sea gratuito los primeros 31 días de una instancia protegida, puede incurrir en cargos por Azure Storage, transacciones de almacenamiento y transferencia de datos. Una máquina virtual recuperada también puede incurrir en cargos por proceso de Azure.


### <a name="is-there-a-cost-associated-to-perform-disaster-recovery-drillstest-failover"></a>¿Hay un costo asociado a realizar exploraciones de recuperación ente desastres y pruebas de conmutación por error?

No hay ningún costo independiente para exploraciones de recuperación ante desastres. Habrá cargos de proceso una vez creada la máquina virtual después de la prueba de conmutación por error.



## <a name="security"></a>Seguridad

### <a name="is-replication-data-sent-to-the-site-recovery-service"></a>¿Se envían mis datos de replicación al servicio de Site Recovery?
No, Site Recovery no intercepta los datos replicados ni tiene información sobre lo que se ejecuta en sus máquinas virtuales o servidores físicos.
Los datos de replicación se intercambian entre hosts locales de Hyper-V, hipervisores de VMware o servidores físicos y el Almacenamiento de Azure en el sitio secundario. Site Recovery no tiene capacidad para interceptar los datos. Únicamente se envían los metadatos necesarios para coordinar la replicación y la conmutación por error al servicio Site Recovery.  

Site Recovery está certificado según la norma ISO 27001:2013, 27018, además de HIPAA y DPA, y está completando sus evaluaciones de SOC2 y FedRAMP JAB.

### <a name="for-compliance-reasons-even-our-on-premises-metadata-must-remain-within-the-same-geographic-region-can-site-recovery-help-us"></a>Para garantizar el cumplimiento, incluso los metadatos de nuestros entornos locales deben permanecer dentro de la misma región geográfica. ¿Site Recovery puede ayudarnos?
Sí. Al crear un almacén de Site Recovery en una región, garantizamos que todos los metadatos que necesitamos para habilitar y coordinar la replicación y la conmutación por error permanezcan dentro del límite geográfico de esa región.

### <a name="does-site-recovery-encrypt-replication"></a>¿Site Recovery cifra la replicación?
Al replicar máquinas virtuales y servidores físicos entre sitios locales, se admite el cifrado en tránsito. En el caso de las máquinas virtuales y los servidores físicos que se replican en Azure, se admite tanto el cifrado en tránsito como el [cifrado en reposo (en Azure)](../storage/common/storage-service-encryption.md).

### <a name="does-azure-to-azure-site-recovery-use-tls-12-for-all-communications-across-microservices-of-azure"></a>¿Azure a Azure Site Recovery utiliza TLS 1.2 para todas las comunicaciones entre microservicios de Azure?
Sí, el protocolo TLS 1.2 se aplica de forma predeterminada en el escenario de Azure a Azure Site Recovery. 

### <a name="how-can-i-enforce-tls-12-on-vmware-to-azure-and-physical-server-to-azure-site-recovery-scenarios"></a>¿Cómo se puede aplicar TLS 1.2 en escenarios de VMware a Azure y Servidor físico a Azure Site Recovery?
Los agentes de Mobility instalados en los elementos replicados se comunican con el servidor de procesos solo a través de TLS 1.2. Sin embargo, la comunicación del servidor de configuración a Azure y del servidor de procesos a Azure podría usar TLS 1.1 o 1.0. Siga las [instrucciones](https://support.microsoft.com/en-us/help/3140245/update-to-enable-tls-1-1-and-tls-1-2-as-default-secure-protocols-in-wi) para aplicar TLS 1.2 en todos los servidores de configuración y servidores de procesos configurados por el usuario.

### <a name="how-can-i-enforce-tls-12-on-hyperv-to-azure-site-recovery-scenarios"></a>¿Cómo puedo aplicar TLS 1.2 en escenarios de HyperV a Azure Site Recovery?
Todas las comunicaciones entre los microservicios de Azure Site Recovery se producen en el protocolo TLS 1.2. Site Recovery usa proveedores de seguridad configurados en el sistema (SO) y utiliza el protocolo TLS más reciente disponible. Habrá que habilitar explícitamente TLS 1.2 en el Registro y entonces Site Recovery comenzará a utilizar TLS 1.2 para la comunicación con los servicios. 

### <a name="how-can-i-enforce-restricted-access-on-my-storage-accounts-which-are-accessed-by-site-recovery-service-for-readingwriting-replication-data"></a>¿Cómo se puede aplicar el acceso restringido a las cuentas de almacenamiento, a las que se accede con Site Recovery para leer o escribir datos de replicación?
Puede cambiar a la identidad administrada del almacén de Recovery Services desde la opción *Identidad*. Una vez que el almacén se registra en Azure Active Directory, puede ir a las cuentas de almacenamiento y proporcionar las siguientes asignaciones de roles al almacén:

- Cuentas de almacenamiento basadas en Resource Manager (tipo Estándar):
  - [Colaborador](../role-based-access-control/built-in-roles.md#contributor)
  - [Colaborador de datos de blobs de almacenamiento](../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)
- Cuentas de almacenamiento basadas en Resource Manager (tipo Premium):
  - [Colaborador](../role-based-access-control/built-in-roles.md#contributor)
  - [Propietario de datos de blobs de almacenamiento](../role-based-access-control/built-in-roles.md#storage-blob-data-owner)
- Cuentas de almacenamiento clásicas:
  - [Colaborador de cuentas de almacenamiento clásico](../role-based-access-control/built-in-roles.md#classic-storage-account-contributor)
  - [Rol de servicio de operador de claves de cuentas de almacenamiento clásicas](../role-based-access-control/built-in-roles.md#classic-storage-account-key-operator-service-role)

## <a name="disaster-recovery"></a>Recuperación ante desastres

### <a name="what-can-site-recovery-protect"></a>¿Qué se puede proteger con Site Recovery?
* **Máquinas virtuales de Azure**: Site Recovery puede replicar cualquier carga de trabajo que se ejecute en una máquina virtual de Azure compatible.
* **Máquinas virtuales de Hyper-V**: Site Recovery puede proteger cualquier carga de trabajo que se ejecute en una máquina virtual de Hyper-V.
* **Servidores físicos**: Site Recovery puede proteger servidores físicos con Windows o Linux.
* **Máquinas virtuales de VMware**: Site Recovery puede proteger cualquier carga de trabajo que se ejecute en una máquina virtual de VMware.

### <a name="what-workloads-can-i-protect-with-site-recovery"></a>¿Qué cargas de trabajo se pueden proteger con Site Recovery?
Puede usar Site Recovery para proteger la mayoría de las cargas de trabajo que se ejecutan en una máquina virtual o un servidor físico. Site Recovery proporciona compatibilidad para la replicación con reconocimiento de aplicaciones para que estas se puedan recuperar a un estado inteligente. Se integra con aplicaciones de Microsoft como SharePoint, Exchange, Dynamics, SQL Server y Active Directory; además, colabora estrechamente con los principales proveedores, como Oracle, SAP, IBM y Red Hat. [Obtenga más información](site-recovery-workload.md) acerca de la protección de la carga de trabajo.

### <a name="can-i-manage-disaster-recovery-for-my-branch-offices-with-site-recovery"></a>¿Es posible administrar la recuperación ante desastres para las sucursales con Site Recovery?
Sí. Si usa Site Recovery para coordinar la replicación y la conmutación por error en las sucursales, disfrutará de una coordinación unificada y una vista de todas las cargas de trabajo de las sucursales desde un mismo punto. Puede ejecutar con facilidad conmutaciones por error y administrar la recuperación ante desastres de todas las sucursales desde la sede central, sin necesidad de visitar estas sucursales.


### <a name="is-disaster-recovery-supported-for-azure-vms"></a>¿La recuperación ante desastres es compatible con las máquinas virtuales de Azure?

Sí, Site Recovery es compatible con la recuperación ante desastres de máquinas virtuales de Azure entre regiones de Azure. [Consulte las preguntas frecuentes](azure-to-azure-common-questions.md) acerca de la recuperación ante desastres de máquinas virtuales de Azure. Si desea replicar entre dos regiones de Azure en el mismo continente, use nuestra oferta de recuperación ante desastres de Azure a Azure. No es necesario configurar conexiones del servidor de configuración/servidor de procesos y ExpressRoute.

### <a name="is-disaster-recovery-supported-for-vmware-vms"></a>¿La recuperación ante desastres es compatible con las máquinas virtuales de VMware?

Sí, Site Recovery es compatible con la recuperación ante desastres de máquinas virtuales de VMware locales. [Consulte las preguntas frecuentes](vmware-azure-common-questions.md) acerca de la recuperación ante desastres de máquinas virtuales de VMware.

### <a name="is-disaster-recovery-supported-for-hyper-v-vms"></a>¿La recuperación ante desastres es compatible con las máquinas virtuales de Hyper-V?
Sí, Site Recovery es compatible con la recuperación ante desastres de máquinas virtuales de Hyper-V locales. [Consulte las preguntas frecuentes](hyper-v-azure-common-questions.md) acerca de la recuperación ante desastres de máquinas virtuales de Hyper-V.

### <a name="is-disaster-recovery-supported-for-physical-servers"></a>¿La recuperación ante desastres es compatible con los servidores físicos?
Sí, Site Recovery es compatible con la recuperación ante desastres de servidores físicos locales que ejecutan Windows y Linux en Azure o en un sitio secundario. Obtenga información acerca de los requisitos para la recuperación ante desastres en [Azure](vmware-physical-azure-support-matrix.md#replicated-machines)y en [un sitio secundario](vmware-physical-secondary-support-matrix.md#replicated-vm-support).
Tenga en cuenta que los servidores físicos se ejecutan como máquinas virtuales en Azure después de la conmutación por error. Actualmente, no se admite la conmutación por recuperación de Azure a un servidor físico local. Solo se puede realizar la conmutación por recuperación a una máquina virtual de VMware.





## <a name="replication"></a>Replicación

### <a name="can-i-replicate-over-a-site-to-site-vpn-to-azure"></a>¿Puedo replicar a través de una VPN de sitio a sitio en Azure?
Azure Site Recovery replica los datos en una cuenta de almacenamiento o discos administrados de Azure a través de un punto de conexión público. La replicación no se realiza a través de una VPN de sitio a sitio. 

### <a name="why-cant-i-replicate-over-vpn"></a>¿Por qué no puedo replicar a través de VPN?

Cuando se replica en Azure, el tráfico de replicación alcanza los puntos de conexión públicos de una cuenta de Azure Storage. Por lo tanto, solo puede replicar a través de una conexión a Internet pública o ExpressRoute (emparejamiento de Microsoft o un emparejamiento público existente).

### <a name="can-i-use-riverbed-steelheads-for-replication"></a>¿Puedo usar Riverbed SteelHeads para la replicación?

Nuestro socio, Riverbed, proporciona instrucciones detalladas sobre cómo trabajar con Azure Site Recovery. Consulte su [guía de solución](https://community.riverbed.com/s/article/DOC-4627).

### <a name="can-i-use-expressroute-to-replicate-virtual-machines-to-azure"></a>¿Puedo usar ExpressRoute para replicar máquinas virtuales en Azure?
Sí, [se puede usar ExpressRoute](concepts-expressroute-with-site-recovery.md) para replicar máquinas virtuales locales en Azure.

- Azure Site Recovery replica los datos en una cuenta de Azure Storage a través de un punto de conexión público. Con el fin de usar ExpressRoute para la replicación de Site Recovery, puede configurar el [emparejamiento de Microsoft](../expressroute/expressroute-circuit-peerings.md#microsoftpeering) o usar un [emparejamiento público](../expressroute/about-public-peering.md) existente (en desuso para los nuevos circuitos).
- El emparejamiento de Microsoft es el dominio de enrutamiento recomendado para la replicación.
- La replicación no se puede realizar a través de un enrutamiento privado.
- En caso de que esté protegiendo máquinas de VMware o máquinas físicas, asegúrese de que [los requisitos de Redes](vmware-azure-configuration-server-requirements.md#network-requirements) también se cumplan para el servidor de configuración. El servidor de configuración requiere conectividad a direcciones URL específicas para la orquestación de la replicación de Site Recovery. Para esta conectividad no se puede usar ExpressRoute.
- Una vez que las máquinas virtuales se hayan conmutado por error a una red virtual de Azure,podrá acceder a ellas mediante la configuración [entre pares privados](../expressroute/expressroute-circuit-peerings.md#privatepeering) con la red virtual de Azure.


### <a name="if-i-replicate-to-azure-what-kind-of-storage-account-or-managed-disk-do-i-need"></a>Si se replica a Azure, ¿qué tipo de cuenta de almacenamiento o disco administrado necesito?

Necesita un almacenamiento LRS o GRS. Se recomienda GRS para que los datos sean resistentes si se produce una interrupción regional o si no se puede recuperar la región principal. La cuenta debe estar en la misma región que el almacén de Recovery Services. Al implementar Site Recovery en Azure Portal, Premium Storage es compatible con máquinas virtuales de VMware, máquinas virtuales de Hyper-V y con la replicación de servidores físicos. Los discos administrados solo con compatibles con LRS.

### <a name="how-often-can-i-replicate-data"></a>¿Con qué frecuencia se pueden replicar los datos?
* **Hyper-V:** Las máquinas virtuales de Hyper-V se pueden replicar cada 30 segundos (excepto en el caso de Premium Storage), cinco minutos o 15 minutos.
* **Máquinas virtuales de Azure, máquinas virtuales de VMware y servidores físicos:** en este caso no es relevante la frecuencia de replicación. La replicación es continua.

### <a name="can-i-extend-replication-from-existing-recovery-site-to-another-tertiary-site"></a>¿Se puede ampliar la replicación desde el sitio de recuperación existente a otro tercer sitio?
No se admite la replicación extendida o encadenada. Solicite esta característica en el [foro de comentarios](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6097959).

### <a name="can-i-do-an-offline-replication-the-first-time-i-replicate-to-azure"></a>¿Se puede hacer una replicación sin conexión la primera vez que se replique en Azure?
No es una opción admitida. Solicite esta característica en el [foro de comentarios](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6227386-support-for-offline-replication-data-transfer-from).

### <a name="can-i-exclude-specific-disks-from-replication"></a>¿Se pueden excluir discos específicos de la replicación?
Esto es posible al replicar máquinas virtuales de VMware y máquinas virtuales de Hyper-V a Azure, mediante Azure Portal.

### <a name="can-i-replicate-virtual-machines-with-dynamic-disks"></a>¿Se pueden replicar máquinas virtuales con discos dinámicos?
Se admiten discos dinámicos al replicar máquinas virtuales de Hyper-V y también al replicar máquinas virtuales de VMware y máquinas físicas en Azure. El disco del sistema operativo debe ser un disco básico.


### <a name="can-i-throttle-bandwidth-allotted-for-replication-traffic"></a>¿Puedo limitar el ancho de banda asignado para el tráfico de replicación?
Sí. Puede obtener más información acerca de la limitación de ancho de banda en estos artículos:

* [Capacity planning for replicating VMware VMs and physical servers](site-recovery-plan-capacity-vmware.md)
* [Capacity planning for replicating Hyper-V VMs without VMM (Planeamiento de la capacidad para replicar máquinas virtuales VMware y servidores físicos)](./hyper-v-deployment-planner-overview.md)

### <a name="can-i-enable-replication-with-app-consistency-in-linux-servers"></a>¿Se puede habilitar la replicación con coherencia de aplicaciones en los servidores de Linux? 
Sí. Azure Site Recovery para el sistema operativo Linux admite scripts personalizados de aplicaciones para la coherencia de aplicaciones. El agente de movilidad de Azure Site Recovery usará el script personalizado con opciones previas y posteriores durante la coherencia de aplicaciones. A continuación se muestran los pasos necesarios para habilitarlo.

1. Inicie sesión como usuario raíz en la máquina.
2. Cambie el directorio a la ubicación de instalación del agente de movilidad de Azure Site Recovery. El valor predeterminado es "/usr/local/ASR".<br>
    `# cd /usr/local/ASR`
3. Cambie el directorio a "VX/scripts" en la ubicación de instalación.<br>
    `# cd VX/scripts`
4. Cree un script de Shell de Bash denominado "customscript.sh" con permisos de ejecución para el usuario raíz.<br>
    a. El script debe ser compatible con las opciones de línea de comandos "--pre" y "--post" (tenga en cuenta los guiones dobles).<br>
    b. Cuando se llama al script con la opción "pre", se debe inmovilizar la entrada/salida de la aplicación y, cuando se llama con la opción "post", se debe reanudar la entrada/salida de la aplicación.<br>
    c. Plantilla de ejemplo<br>

    `# cat customscript.sh`<br>

```
    #!/bin/bash

    if [ $# -ne 1 ]; then
        echo "Usage: $0 [--pre | --post]"
        exit 1
    elif [ "$1" == "--pre" ]; then
        echo "Freezing app IO"
        exit 0
    elif [ "$1" == "--post" ]; then
        echo "Thawed app IO"
        exit 0
    fi
```

5. Agregue los comandos para inmovilizar y liberar la entrada/salida en los pasos "pre" y "post" de las aplicaciones que requieren coherencia de la aplicación. Puede optar por agregar otro script que los especifique y llamarlo desde "customscript.sh" con las opciones "pre" y "post".

>[!Note]
>La versión del agente de Site Recovery debe ser 9.24 o superior para admitir scripts personalizados.

## <a name="replication-policy"></a>Directiva de replicación

### <a name="what-is-a-replication-policy"></a>¿Qué es una directiva de replicación?

Una directiva de replicación define la configuración del historial de retención de puntos de recuperación. La directiva también define la frecuencia de las instantáneas coherentes con la aplicación. De forma predeterminada, Azure Site Recovery crea una nueva directiva de replicación con la configuración predeterminada de:

- 24 horas para el historial de retención de puntos de recuperación.
- 4 horas para la frecuencia de las instantáneas coherentes con la aplicación.

### <a name="what-is-a-crash-consistent-recovery-point"></a>¿Qué es un punto de recuperación coherente con los bloqueos?

Un punto de recuperación coherente con los bloqueos tiene los datos en el disco, como si se tirara del cable de alimentación del servidor durante la instantánea. No incluye nada que estuviera en memoria cuando se tomó la instantánea.

Hoy en día, la mayoría de las aplicaciones se pueden recuperar bien a partir de instantáneas coherentes con los bloqueos. Un punto de recuperación coherente con los bloqueos suele ser suficiente para aplicaciones y sistemas operativos que no tienen bases de datos, como los servidores de archivos, los servidores DHCP y los servidores de impresión.

### <a name="what-is-the-frequency-of-crash-consistent-recovery-point-generation"></a>¿Cuál es la frecuencia de generación de puntos de recuperación coherentes frente al bloqueo?

Site Recovery crea un punto de recuperación coherente con los bloqueos cada 5 minutos.

### <a name="what-is-an-application-consistent-recovery-point"></a>¿Qué es un punto de recuperación coherente con la aplicación?

Los puntos de recuperación coherentes con la aplicación se crean a partir de instantáneas coherentes con la aplicación. Los puntos de recuperación coherentes con la aplicación capturan los mismos datos que las instantáneas coherentes con los bloqueos, al tiempo que también se capturan los datos en memoria y todas las transacciones en curso.

Debido a su contenido adicional, las instantáneas coherentes con la aplicación son las más complejas y las que más tardan. Se recomiendan puntos de recuperación coherentes con la aplicación para sistemas operativos de bases de datos y aplicaciones como SQL Server.

### <a name="what-is-the-impact-of-application-consistent-recovery-points-on-application-performance"></a>¿Cómo afectan los puntos de recuperación coherentes con la aplicación en el rendimiento de la aplicación?

Los puntos de recuperación coherentes con la aplicación capturan todos los datos en memoria y en curso. Como los puntos de recuperación capturan esas datos, requieren un marco como el Servicio de instantáneas de volumen de Windows para poner en modo inactivo la aplicación. Si el proceso de captura es frecuente, puede afectar al rendimiento cuando la carga de trabajo ya está ocupada. No se recomienda usar baja frecuencia con los puntos de recuperación coherentes con la aplicación en cargas de trabajo que no son de base de datos. Incluso con cargas de trabajo de base de datos, 1 hora es suficiente.

### <a name="what-is-the-minimum-frequency-of-application-consistent-recovery-point-generation"></a>¿Cuál es la frecuencia mínima de generación de puntos de recuperación coherentes frente a la aplicación?

Site Recovery puede crear un punto de recuperación coherente con la aplicación con una frecuencia mínima de 1 hora.

### <a name="how-are-recovery-points-generated-and-saved"></a>¿Cómo se generan y guardan los puntos de recuperación?

Para comprender cómo Site Recovery genera puntos de recuperación, veamos un ejemplo de una directiva de replicación. Esta directiva de replicación tiene un punto de recuperación con una ventana de retención de 24 horas y una instantánea con una frecuencia coherente con la aplicación de 1 hora.

Site Recovery crea un punto de recuperación coherente con los bloqueos cada 5 minutos. No se puede cambiar esta frecuencia. Durante la última hora, puede elegir entre 12 puntos coherentes con los bloqueos y 1 punto coherente con la aplicación. Conforme transcurre el tiempo, Site Recovery elimina todos los puntos de recuperación de hace más de una hora y solo guarda un punto de recuperación por hora.

La siguiente captura de pantalla ilustra el ejemplo. En la captura de pantalla:

- En la pasada hora, hay puntos de recuperación con una frecuencia de 5 minutos.
- Por encima de esta, Site Recovery conserva solo un punto de recuperación.

   ![Lista de puntos de recuperación generados](./media/azure-to-azure-troubleshoot-errors/recoverypoints.png)

### <a name="how-far-back-can-i-recover"></a>¿Hasta cuánto tiempo atrás puedo recuperar?

El punto de recuperación más antiguo que puede usar es de 72 horas.

### <a name="i-have-a-replication-policy-of-24-hours-what-will-happen-if-a-problem-prevents-site-recovery-from-generating-recovery-points-for-more-than-24-hours-will-my-previous-recovery-points-be-lost"></a>Tengo una directiva de replicación de 24 horas. ¿Qué ocurrirá si un problema impide que Site Recovery genere puntos de recuperación durante más de 24 horas? ¿Se perderán mis puntos de recuperación anteriores?

No, Site Recovery mantendrá todos los puntos de recuperación anteriores. En función de la ventana de retención de los puntos de recuperación, Site Recovery reemplaza el punto más antiguo solo si genera nuevos puntos. A consecuencia de este problema, Site Recovery no puede generar ningún punto de recuperación nuevo. Hasta que no haya nuevos puntos de recuperación, todos los puntos anteriores permanecerán después de alcanzar la ventana de retención.

### <a name="after-replication-is-enabled-on-a-vm-how-do-i-change-the-replication-policy"></a>Después de habilitar la replicación en una máquina virtual, ¿cómo puedo cambiar la directiva de replicación?

Vaya a **Almacén de Site Recovery** > **Site Recovery Infrastructure (Infraestructura de Site Recovery)**  > **Directivas de replicación**. Seleccione la directiva que quiera editar y guarde los cambios. Todos los cambios se aplicarán también en todas las replicaciones existentes.

### <a name="are-all-the-recovery-points-a-complete-copy-of-the-vm-or-a-differential"></a>¿Todos los puntos de recuperación son una copia completa de la máquina virtual o diferencial?

El primer punto de recuperación que se genera tiene la copia completa. Los puntos de recuperación sucesivos tienen los cambios diferenciales.

### <a name="does-increasing-the-retention-period-of-recovery-points-increase-the-storage-cost"></a>¿Al aumentar el período de retención de los puntos de recuperación se aumenta el costo de almacenamiento?

Sí. Si aumenta el período de retención de 24 a 72 horas, Site Recovery guardará los puntos de recuperación durante otras 48 horas. El tiempo adicional implicará cargos de almacenamiento. Por ejemplo, un único punto de recuperación podría tener cambios diferenciales de 10 GB con un costo por GB de 0,16 USD al mes. Los cargos adicionales serían de 1,60 USD × 48 al mes.


## <a name="failover"></a>Conmutación por error
### <a name="if-im-failing-over-to-azure-how-do-i-access-the-azure-vms-after-failover"></a>Si se realiza una conmutación por error a Azure, ¿cómo se puede tener acceso a las máquinas virtuales de Azure tras este proceso?

Es posible tener acceso a las máquinas virtuales de Azure a través de una conexión segura a Internet o a través de una VPN de sitio a sitio o mediante Azure ExpressRoute. Debe preparar algunas cosas para la conexión. [Más información](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover).


### <a name="if-i-fail-over-to-azure-how-does-azure-make-sure-my-data-is-resilient"></a>Si se realiza una conmutación por error a Azure, ¿cómo se asegura Azure de que los datos resistan el proceso?
Azure está diseñado para la resistencia. Site Recovery ya está diseñado para la conmutación por error en un centro de datos de Azure secundario, según el Acuerdo de Nivel de Servicio de Azure. Si esto ocurre, nos aseguraremos de que los metadatos y los almacenes permanecen en la misma región geográfica que eligió para su almacén.  

### <a name="if-im-replicating-between-two-datacenters-what-happens-if-my-primary-datacenter-experiences-an-unexpected-outage"></a>Si se replica entre dos centros de datos, ¿qué ocurre si el centro de datos principal experimenta una interrupción inesperada?
Puede desencadenar una conmutación por error no planeada desde el sitio secundario. Site Recovery no necesita conectividad desde el sitio principal para realizar la conmutación por error.

### <a name="is-failover-automatic"></a>¿La conmutación por error es automática?
La conmutación por error no es automática. Puede iniciar las conmutaciones por error con solo un clic en el portal o bien puede usar [Site Recovery PowerShell](/powershell/module/az.recoveryservices) para desencadenar una conmutación por error. La conmutación por recuperación también es una acción sencilla en el portal de Site Recovery.

Para automatizar estos procesos, puede utilizar Orchestrator u Operations Manager locales para detectar un error de la máquina virtual y desencadenar luego la conmutación por error mediante el SDK.

* [Obtenga más información](site-recovery-create-recovery-plans.md) sobre los planes de recuperación.
* [Más información](site-recovery-failover.md) acerca de la conmutación por error.
* [Más información](./vmware-azure-failback.md) acerca de la conmutación por recuperación de servidores físicos y máquinas virtuales de VMware

### <a name="if-my-on-premises-host-is-not-responding-or-crashed-can-i-fail-back-to-a-different-host"></a>Si mi host local no responde o se ha bloqueado, ¿puedo realizar la conmutación por recuperación a otro host?
Sí, puede usar la recuperación en una ubicación alternativa para realizar la conmutación por recuperación a otro host de Azure.

* [Para máquinas virtuales de VMware](concepts-types-of-failback.md#alternate-location-recovery-alr)
* [Para máquinas virtuales de Hyper-V](hyper-v-azure-failback.md#fail-back-to-an-alternate-location)

## <a name="automation"></a>Automation

### <a name="can-i-automate-site-recovery-scenarios-with-an-sdk"></a>¿Se pueden automatizar escenarios de Site Recovery con un SDK?
Sí. Puede automatizar los flujos de trabajo de Site Recovery mediante la API de Rest, PowerShell o el SDK de Azure. Escenarios admitidos actualmente para la implementación de Site Recovery con PowerShell:

* [Replicación de máquinas virtuales de Hyper-V en nubes VMM en Azure PowerShell Resource Manager](hyper-v-vmm-powershell-resource-manager.md)
* [Replicación de máquinas virtuales de Hyper-V sin VMM en Azure PowerShell Resource Manager](hyper-v-azure-powershell-resource-manager.md)
* [Replicación de máquinas virtuales de VMware en Azure con el Administrador de recursos de PowerShell](vmware-azure-disaster-recovery-powershell.md)

## <a name="componentprovider-upgrade"></a>Actualización del componente o proveedor

### <a name="where-can-i-find-the-release-notesupdate-rollups-of-site-recovery-upgrades"></a>¿Dónde puedo encontrar las notas de la versión o los paquetes acumulativos de revisiones de las actualizaciones de Site Recovery?

[Obtenga información](site-recovery-whats-new.md) acerca de nuevas actualizaciones y de [la acumulación](service-updates-how-to.md).

## <a name="next-steps"></a>Pasos siguientes
* Lea la [Información general sobre Site Recovery](site-recovery-overview.md)
