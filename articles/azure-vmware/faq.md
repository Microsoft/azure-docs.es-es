---
title: Preguntas más frecuentes
description: En este artículo se ofrecen respuestas a algunas de las preguntas más frecuentes sobre Azure VMware Solution.
ms.topic: conceptual
ms.date: 05/04/2020
ms.author: dikamath
ms.openlocfilehash: 7b4abc2d711a3da6a6df125854759e083d7e04a7
ms.sourcegitcommit: 0fd1f3fe7817ad44d878d580ec167e1508051795
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/19/2020
ms.locfileid: "90817843"
---
# <a name="frequently-asked-questions-about-azure-vmware-solution-preview"></a>Preguntas frecuentes sobre Azure VMware Solution en versión preliminar

Respuestas a las preguntas frecuentes sobre Azure VMware Solution.

## <a name="general"></a>General

### <a name="what-is-azure-vmware-solution"></a>¿Qué es Azure VMware Solution?

A medida que las empresas buscan estrategias de modernización de la TI para mejorar la agilidad empresarial, reducir costos y acelerar la innovación, las plataformas de nube híbrida han surgido como los grandes facilitadores de la transformación digital de los clientes. Azure VMware Solution combina el Centro de datos definido por software (SDDC) de VMware con el ecosistema global de servicios en la nube de Microsoft Azure. Azure VMware Solution se administra para cumplir los requisitos de rendimiento, disponibilidad, seguridad y cumplimiento.

## <a name="azure-vmware-solution-service"></a>Servicio de Azure VMware Solution

### <a name="where-is-azure-vmware-solution-available-today"></a>¿Dónde está disponible actualmente Azure VMware Solution?

El servicio se agrega continuamente a regiones nuevas, por lo que debe consultar la [información de disponibilidad de servicio más reciente](https://azure.microsoft.com/global-infrastructure/services/?products=azure-vmware) para más detalles. 

### <a name="can-workloads-running-in-an-azure-vmware-solution-instance-consume-or-integrate-with-azure-services"></a>¿Pueden las cargas de trabajo que se ejecutan en una instancia de Azure VMware Solution consumir o integrarse en los servicios de Azure?

Todos los servicios de Azure estarán disponibles para los clientes de Azure VMware Solution. Las limitaciones de rendimiento y disponibilidad de los servicios específicos deberán abordarse para cada caso.

### <a name="do-i-use-the-same-tools-that-i-use-now-to-manage-private-cloud-resources"></a>¿Utilizo las mismas herramientas que uso ahora para administrar recursos de nube privada?

Sí. Azure Portal se utiliza para la implementación y varias operaciones de administración. vCenter y NSX Manager se utilizan para administrar los recursos de vSphere y NSX-T.

### <a name="can-i-manage-a-private-cloud-with-my-on-premises-vcenter"></a>¿Puedo administrar una nube privada con mi instancia de vCenter local?

En un principio, Azure VMware Solution no admitirá una única experiencia de administración en entornos locales y de nube privada. Los clústeres de nube privada se administrarán con vCenter y NSX Manager de local a una nube privada.

### <a name="can-i-use-vrealize-suite-running-on-premises"></a>¿Puedo usar vRealize Suite que se ejecute en el entorno local? 

Las integraciones y los casos de uso específicos se pueden evaluar en cada caso.

### <a name="can-i-migrate-vsphere-vms-from-on-premises-environments-to-azure-vmware-solution-private-clouds"></a>¿Puedo migrar VM de vSphere de entornos locales a nubes privadas de Azure VMware Solution?

Sí. La migración de VM y vMotion se pueden usar para mover VM a una nube privada si se cumplen los [requisitos de vMotion](https://kb.vmware.com/s/article/210695) estándar de vCenter.

### <a name="is-a-specific-version-of-vsphere-required-in-on-premises-environments"></a>¿Se requiere una versión específica de vSphere en entornos locales?

Dado que todos los entornos de nube cuentan con HCX, se requiere vSphere 5.5 o posterior en entornos locales para vMotion.

### <a name="what-does-the-change-control-process-look-like"></a>¿Qué aspecto tiene el proceso de control de cambios?

Las actualizaciones realizadas en el propio servicio seguirán el proceso estándar de administración de cambios de Microsoft Azure. Los clientes son responsables de las tareas de administración de las cargas de trabajo y de los procesos asociados de administración de cambios.

### <a name="how-is-this-different-from-azure-vmware-solution-by-cloudsimple"></a>¿En qué se diferencia de Azure VMware Solution by CloudSimple?

Con la nueva solución Azure VMware Solution, Microsoft y VMware tienen una asociación directa con el proveedor de la nube. La solución nueva está diseñada, creada y respaldada completamente por Microsoft, y aprobada por VMware. Desde el punto de vista de la arquitectura, las soluciones son coherentes con la pila de tecnología de VMware que se ejecuta en una infraestructura dedicada de Azure.

### <a name="if-im-an-existing-azure-vmware-solution-customer-what-does-this-preview-mean-for-me"></a>Si soy cliente de Azure VMware Solution, ¿qué significa esta versión preliminar en mi caso?

No hay ningún cambio en la solución Azure VMware Solution by CloudSimple existente. La solución se puede usar aún en Azure. Azure VMware Solution está respaldada por nuestro Acuerdo de Nivel de Servicio [(SLA)](https://aka.ms/CSVMwareSLA). Los clientes deben seguir usando el servicio con las cargas de trabajo de producción; se trata de una solución disponible regulada por los [términos de servicio de Microsoft](https://azure.microsoft.com/support/legal/).

### <a name="can-i-migrate-from-azure-vmware-solution-by-cloudsimple-to-this-new-solution"></a>¿Puedo migrar desde Azure VMware Solution by CloudSimple a esta nueva solución?

Sí, Azure VMware Solution admite la migración mediante herramientas conocidas de VMware, como HCX. En el caso de los clientes interesados en migrar a la solución nueva, colabore con el equipo de cuentas de Microsoft para explorar las opciones y la compatibilidad disponible.



## <a name="compute-network-and-storage"></a>Proceso, red y almacenamiento

### <a name="is-there-more-than-one-type-of-host-available"></a>¿Hay más de un tipo de host disponible?

Hay solo un tipo de host disponible.

### <a name="what-are-the-cpu-specifications-in-each-type-of-host"></a>¿Cuáles son las especificaciones de CPU en cada tipo de host?

Los servidores tienen dos CPU Intel de 18 núcleos de 2,3 GHz.

### <a name="how-much-memory-is-in-each-host"></a>¿Cuánta memoria hay en cada host?

Los servidores tienen 576 GB de RAM.

### <a name="what-is-the-storage-capacity-of-each-host"></a>¿Cuál es la capacidad de almacenamiento de cada host?

Cada host ESXi tiene dos grupos de discos vSAN con un nivel de capacidad de 15,2 TB y un nivel de caché NVMe de 3,2 TB (1,6 TB en cada grupo de discos).

### <a name="how-much-network-bandwidth-is-available-in-each-esxi-host"></a>¿Cuánto ancho de banda de red hay disponible en cada host ESXi?

Cada host ESXi de Azure VMware Solution está configurado con cuatro adaptadores de red de 25 Gbps, con dos adaptadores de red aprovisionados para el tráfico del sistema ESXi y otros dos aprovisionados para el tráfico de carga de trabajo. 

### <a name="is-data-stored-on-the-vsan-datastores-encrypted-at-rest"></a>¿Se cifran en reposo los datos almacenados en los almacenes de datos vSAN?

Sí, todos los datos vSAN se cifran de forma predeterminada mediante claves que se almacenan en Azure Key Vault.

## <a name="hosts-clusters-and-private-clouds"></a>Hosts, clústeres y nubes privadas

### <a name="is-the-underlying-infrastructure-shared"></a>¿Se comparte la infraestructura subyacente?

No, los hosts y clústeres de nube privada son dedicados y se borran de forma segura antes y después del uso.

### <a name="what-are-the-minimum-and-maximum-number-of-hosts-per-cluster"></a>¿Cuál es el número mínimo y máximo de hosts por clúster?

Los clústeres se pueden escalar entre 3 y 16 hosts ESXi. Los clústeres de prueba están limitados a tres hosts.

### <a name="can-i-scale-my-private-cloud-clusters"></a>¿Puedo escalar mis clústeres de nube privada?

Sí, los clústeres se escalan entre el número mínimo y máximo de hosts ESXi. Los clústeres de prueba están limitados a tres hosts.

### <a name="what-are-trial-clusters"></a>¿Qué son los clústeres de prueba?

Los clústeres de prueba son tres clústeres de hosts que se usan para las evaluaciones de un mes de las nubes privadas de Azure VMware Solution.

### <a name="can-i-use-high-end-hosts-for-trial-clusters"></a>¿Puedo usar hosts de alto nivel para los clústeres de prueba?

No. Los hosts ESXi de alto nivel están reservados para su uso en clústeres de producción.

## <a name="azure-vmware-solution-and-vmware-software"></a>Azure VMware Solution y software de VMware

### <a name="what-versions-of-vmware-software-is-used-in-private-clouds"></a>¿Qué versiones del software de VMware se usan en las nubes privadas?

Las nubes privadas usan vSphere 6.7, vSAN 6.7, HCX y NSX-T 2.5.  

### <a name="do-private-clouds-use-vmware-nsx"></a>¿Usan las nubes privadas VMware NSX?

Sí, NSX-T 2.5 se usa para las redes definidas por software en las nubes privadas de Azure VMware Solution.

### <a name="can-i-use-vmware-nsx-v-in-a-private-cloud"></a>¿Puedo usar VMware NSX-V en una nube privada?

No. La única versión admitida de NSX es NSX-T.

### <a name="is-nsx-required-in-on-premises-environments-or-networks-that-connect-to-a-private-cloud"></a>¿Se requiere NSX en entornos locales o redes que se conectan a una nube privada?

No, no es necesario usar NSX en el entorno local.

### <a name="what-is-the-upgrade-and-update-schedule-for-vmware-software-in-a-private-cloud"></a>¿Cuál es el cronograma de actualización del software de VMware en una nube privada?

Las actualizaciones del conjunto de productos de software de la nube privada se realizan para mantener el software dentro de una de las versiones más recientes del conjunto de productos de software de VMware. Las versiones de software de la nube privada pueden ser diferentes de las versiones más recientes de los componentes de software individuales (ESXi, NSX-T, vCenter, vSAN).

### <a name="how-often-will-the-private-cloud-software-stack-be-updated"></a>¿Con qué frecuencia se actualizará la pila de software de nube privada?

El software de la nube privada se actualiza según un cronograma que sigue la publicación del conjunto de productos de software de VMware. Su nube privada no requiere tiempo de inactividad para las actualizaciones.

## <a name="connectivity"></a>Conectividad

### <a name="what-network-ip-address-planning-is-required-to-incorporate-private-clouds-with-on-premises-environments"></a>¿Qué planeación de direcciones IP de red se necesita para incorporar nubes privadas en entornos locales?

Se requiere un espacio de direcciones de red privada /22 para implementar una nube privada de Azure VMware Solution. Este espacio de direcciones privadas no debe superponerse con otras redes virtuales de una suscripción ni con redes locales.
 
### <a name="how-do-i-connect-from-on-premises-environments-to-an-azure-vmware-solution-private-cloud"></a>¿Cómo me conecto desde entornos locales a una nube privada de Azure VMware Solution?

Puede conectarse al servicio mediante uno de dos métodos: 

- Con una VM o puerta de enlace de aplicaciones implementadas en una red virtual de Azure emparejada a través de ExpressRoute a la nube privada.
- A través de Global Reach de ExpressRoute desde el centro de datos local a un circuito de Azure ExpressRoute.

### <a name="how-do-i-connect-a-workload-vm-to-the-internet-or-an-azure-service-endpoint"></a>¿Cómo conecto una VM de cargas de trabajo a Internet o a un punto de conexión de servicio de Azure?

En Azure Portal, habilite la conectividad a Internet para una nube privada. Con NSX-T Manager, cree un enrutador NSX-T T1 y un conmutador lógico. A continuación, use vCenter para implementar una VM en el segmento de red definido por el conmutador lógico. Esa VM tendrá acceso de red a Internet y a los servicios de Azure.

### <a name="do-i-need-to-restrict-access-from-the-internet-to-vms-on-logical-networks-in-a-private-cloud"></a>¿Es necesario restringir el acceso desde Internet a las VM en redes lógicas en una nube privada?

No. No se permite el tráfico de red entrante desde Internet directamente a nubes privadas.

### <a name="do-i-need-to-restrict-internet-access-from-vms-on-logical-networks-to-the-internet"></a>¿Es necesario restringir el acceso a Internet desde VM en redes lógicas?

Sí. Deberá usar NSX-T Manager para crear un firewall que restrinja el acceso de las VM a Internet.

## <a name="accounts-and-privileges"></a>Cuentas y privilegios

### <a name="what-accounts-and-privileges-will-i-get-with-my-new-azure-vmware-solution-private-cloud"></a>¿Qué cuentas y privilegios obtendré con mi nueva nube privada de Azure VMware Solution?

Recibirá credenciales para un usuario cloudadmin en vCenter y acceso de administrador en NSX-T Manager. También hay un grupo CloudAdmin que se puede usar para incorporar Azure Active Directory. Para obtener más información, consulte [Conceptos de acceso y de identidad](concepts-identity.md).

### <a name="can-have-administrator-access-to-esxi-hosts"></a>¿Puedo tener acceso de administrador a los hosts de ESXi?

No, el acceso de administrador a ESXi está restringido para cumplir con los requisitos de seguridad de la solución.

### <a name="what-privileges-and-permissions-will-i-have-in-vcenter"></a>¿Qué privilegios y permisos tendré en vCenter?

Tendrá los privilegios del grupo CloudAdmin. Para obtener más información, consulte [Conceptos de acceso y de identidad](concepts-identity.md).

### <a name="what-privileges-and-permissions-will-i-have-on-the-nsx-t-manager"></a>¿Qué privilegios y permisos tendré en NSX-T Manager?

Tendrá privilegios completos de administrador en NSX-T y podrá administrar el control de acceso basado en roles, tal como lo haría con NSX-T Data Center en el entorno local. Para obtener más información, consulte [Conceptos de acceso y de identidad](concepts-identity.md).

> [!NOTE]
> Se crea y configura un enrutador T0 como parte de una implementación de nube privada. Cualquier modificación en el enrutador lógico o en las VM del nodo perimetral de NSX-T podría afectar a la conectividad a la nube privada.

## <a name="billing-and-support"></a>Facturación y soporte técnico

### <a name="how-will-i-be-billed-during-the-preview-of-azure-vmware-solution"></a>¿Cómo se me facturará durante la versión preliminar de Azure VMware Solution?

La facturación de Azure VMware Solution durante la versión preliminar es mensual en una base de pago por uso. Habrá opciones adicionales disponibles con la disponibilidad general.

### <a name="how-will-pricing-be-structured-during-the-preview-of-azure-vmware-solution"></a>¿Cómo se me facturará durante la versión preliminar de Azure VMware Solution?

Para ver las preguntas generales sobre los precios, consulte la página de [precios](https://azure.microsoft.com/pricing/details/azure-vmware) de la solución de VMware en Azure. Los precios de la versión preliminar están disponibles a petición; póngase en contacto con su equipo de cuentas o siga el vínculo de la página de precios para ponerse en contacto con Ventas.

### <a name="who-supports-azure-vmware-solution"></a>¿Quién proporciona el soporte técnico de Azure VMware Solution?

Microsoft ofrece el soporte técnico para Azure VMware Solution. Tenga en cuenta que, según nuestras instrucciones de versión preliminar, proporcionaremos soporte técnico durante las horas laborables de 9 a 5 p. m. PST, de lunes a viernes. Puede generar una incidencia de soporte técnico desde [este vínculo](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

### <a name="what-accounts-do-i-need-to-create-an-azure-vmware-solution-private-cloud"></a>¿Qué cuentas necesito para crear una nube privada de Azure VMware Solution?

Necesitará una cuenta de Azure en una suscripción a Azure.

### <a name="how-do-i-request-a-host-quota-increase-for-azure-vmware-solution"></a>¿Cómo se solicita un aumento de la cuota de host para Azure VMware Solution?

* Necesitará un [Contrato Enterprise (EA) de Azure](https://docs.microsoft.com/azure/cost-management-billing/manage/ea-portal-agreements) con Microsoft.
* Necesitará una cuenta de Azure en una suscripción a Azure.

Antes de crear el recurso de Azure VMware Solution, debe enviar una incidencia de soporte técnico para que se asignen los nodos. Una vez que el equipo de soporte técnico recibe su solicitud, tardan hasta cinco días laborables en confirmarla y asignar los nodos. Si tiene una nube privada de Azure VMware Solution y quiere asignar más nodos, pasará por el mismo proceso.


1. En Azure Portal, en **Ayuda y soporte técnico**, cree una **[solicitud de soporte técnico](https://rc.portal.azure.com/#create/Microsoft.Support)** y proporcione la siguiente información para el vale:
   - **Tipo de problema**: Requisitos previos técnicos
   - **Subscription** (Suscripción): Seleccione su suscripción.
   - **Servicio:** Todos los servicios > Azure VMware Solution
   - **Recurso:** Pregunta general 
   - **Resumen:** Necesidad de capacidad
   - **Tipo de problema**: problemas de administración de la capacidad.
   - **Subtipo de problema:** solicitud de cliente de capacidad o cuota de host adicional.

1. En el campo **Descripción** de la incidencia de soporte técnico, en la pestaña **Detalles**, proporcione la siguiente información:

   - POC o Producción 
   - Nombre de región
   - Número de nodos
   - Cualquier otro detalle

   >[!NOTE]
   >Azure VMware Solution recomienda como mínimo tres nodos para poner en marcha la nube privada y N+1 nodos para la redundancia. 

1. Seleccione **Revisar + crear** para enviar la solicitud.

   Un representante de soporte técnico tardará hasta cinco días laborables en confirmar su solicitud.

   >[!IMPORTANT] 
   >Si ya tiene una instancia de Azure VMware Solution y solicita más nodos, tenga en cuenta que necesitamos cinco días laborables para asignarlos. 

1. Antes de aprovisionar los nodos, asegúrese de registrar el proveedor de recursos **Microsoft.AVS** en Azure Portal.  

   ```azurecli-interactive
   az provider register -n Microsoft.AVS --subscription <your subscription ID>
   ```

   Para ver otras formas de registrar el proveedor de recursos, consulte [Tipos y proveedores de recursos de Azure](../azure-resource-manager/management/resource-providers-and-types.md).

<!-- LINKS - external -->
[kb2106952]: https://kb.vmware.com/s/article/2106952

<!-- LINKS - internal -->
[Access and Identity Concepts]: concepts-identity.md
