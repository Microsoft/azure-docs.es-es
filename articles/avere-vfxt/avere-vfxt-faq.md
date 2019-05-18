---
title: Preguntas frecuentes sobre Avere vFXT for Azure
description: Preguntas frecuentes sobre Avere vFXT for Azure
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 02/28/2019
ms.author: v-erkell
ms.openlocfilehash: 47a4b38d39c52992b51284776ec34cb9491020e7
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595416"
---
# <a name="avere-vfxt-for-azure-faq"></a>Preguntas frecuentes acerca de Avere vFXT for Azure

En este artículo encontrará respuestas que pueden ayudarle a decidir si la solución de Avere vFXT for Azure es adecuada para sus necesidades. Proporciona información básica sobre Avere vFXT y explica cómo funciona con otros componentes de Azure y con productos de proveedores externos. 

## <a name="general"></a>General 

### <a name="what-is-avere-vfxt-for-azure"></a>¿Qué es Avere vFXT for Azure?

Avere vFXT for Azure es un sistema de archivos de alto rendimiento que almacena en caché datos activos en el proceso de Azure para un procesamiento eficaz de cargas de trabajo críticas.

### <a name="is-avere-vfxt-a-storage-solution"></a>¿Es Avere vFXT una solución de almacenamiento?

No. Avere vFXT es una *memoria caché* de sistema de archivos que se adjunta a los entornos de almacenamiento, como NAS de EMC, NetApp o un contenedor de blobs de Azure. Avere vFXT optimiza las solicitudes de datos de los clientes y almacena en caché los datos que sirven para mejorar el rendimiento a escala y a lo largo del tiempo. Avere vFXT en sí no almacena datos. No tiene información sobre la cantidad de datos que se almacenan tras de sí.

### <a name="is-avere-vfxt-a-tiering-solution"></a>¿Es Avere vFXT una solución de niveles?

Avere vFXT no apila automáticamente datos entre los niveles de acceso frecuente y esporádico.  

### <a name="how-do-i-know-if-an-environment-is-right-for-avere-vfxt"></a>¿Cómo se puede saber si un entorno es adecuado para Avere vFXT?

La mejor manera de pensar acerca de esta pregunta es: "¿Puede almacenarse en caché la carga de trabajo?". Es decir, ¿tiene la carga de trabajo un alto índice de lectura a escritura? Por ejemplo, 80/20 o 70/30 lecturas/escrituras.

Tenga en cuenta Avere vFXT for Azure si tiene una canalización de análisis basada en archivos que se ejecuta en un gran número de máquinas virtuales de Azure y cumple una o varias de las condiciones siguientes:

* El rendimiento general es lento o incoherente debido a largos tiempos de acceso a archivos (decenas de milisegundos o segundos, dependiendo de los requisitos). Esta latencia es inaceptable para el cliente.

* Los datos necesarios para el procesamiento se encuentran en el extremo de un entorno WAN y mover esos datos de forma permanente no resulta práctico. Los datos podrían estar en otra región de Azure o en un centro de datos del cliente.

* Un número significativo de clientes solicita los datos: por ejemplo, en un clúster de informática de alto rendimiento (HPC). El gran número de solicitudes simultáneas puede aumentar la latencia.

* El cliente quiere ejecutar la canalización actual "tal cual" en máquinas virtuales de Azure y necesita una solución de almacenamiento compartido (o almacenamiento en caché) basada en POSIX para la escalabilidad. Al usar Avere vFXT for Azure, no es necesario rediseñar la canalización de trabajo para hacer llamadas nativas al almacenamiento de Azure Blob.

* La aplicación HPC se basa en clientes NFSv3. (En algunas circunstancias, se pueden usar clientes SMB 2.1, pero el rendimiento es limitado).

El siguiente diagrama simplifica la respuesta a esta pregunta. Cuanto más se acerque el flujo de trabajo a la esquina superior derecha, es más probable que la solución de almacenamiento en caché de Avere sea adecuada para su entorno.

![Diagrama que muestra que las cargas con mucha actividad de lectura y miles de clientes son más adecuadas para Avere vFXT](media/avere-vfxt-fit-assessment.png)

### <a name="at-what-scale-of-clients-does-the-avere-vfxt-solution-make-the-most-sense"></a>¿Con qué escala de clientes tiene más sentido la solución Avere vFXT?

La solución de caché Avere vFXT está diseñada para controlar cientos, miles o decenas de miles de núcleos de proceso. Si tiene algunas máquinas con poco trabajo, Avere vFXT no es la solución adecuada.

Los clientes habituales de Avere vFXT ejecutan cargas de trabajo exigentes, a partir de aproximadamente 1000 núcleos de CPU. Estos entornos pueden llegar a 50 000 núcleos o más. Dado que Avere vFXT es escalable, puede agregar nodos para admitir estas cargas de trabajo a medida que crecen para requerir más rendimiento o más IOPS.

### <a name="how-much-data-can-an-avere-vfxt-environment-store"></a>¿Cuántos datos puede almacenar un entorno de Avere vFXT?

Avere vFXT es una memoria caché. No almacena datos en concreto. Usa una combinación de RAM y SSD para almacenar los datos en caché. Los datos se almacenan de forma permanente en un sistema de almacenamiento back-end (por ejemplo, un sistema NAS de NetApp o un contenedor de blobs). El sistema Avere vFXT no tiene información sobre la cantidad de datos que se almacenan tras de sí. Avere vFXT solo almacena en caché el subconjunto de los datos que los clientes solicitan.  

### <a name="what-regions-are-supported"></a>¿Qué regiones se admiten?

Avere vFXT para Azure es compatible con todas las regiones excepto para regiones soberana (China, Alemania). Asegúrese de que la región que quiere usar sea compatible con la gran cantidad de núcleos de proceso, así como con las instancias de VM necesarias para crear el clúster de Avere vFXT.

### <a name="how-do-i-get-help-with-avere-vfxt"></a>¿Cómo puedo obtener ayuda con Avere vFXT?

Un grupo de soporte técnico especializado ofrece ayuda con Avere vFXT for Azure. Siga las instrucciones de [Ayuda con el sistema](avere-vfxt-open-ticket.md#open-a-support-ticket-for-your-avere-vfxt) para abrir una incidencia de soporte técnico desde Azure Portal. 

### <a name="is-avere-vfxt-highly-available"></a>¿Tiene Avere vFXT Avere alta disponibilidad?

Sí, Avere vFXT se ejecuta exclusivamente como solución de alta disponibilidad.

### <a name="does-avere-vfxt-for-azure-also-support-other-cloud-services"></a>¿Admite Avere vFXT for Azure también otros servicios en la nube?

Sí, los clientes pueden usar más de un proveedor de nube con el clúster de Avere vFXT. Es compatible con cubos estándar de AWS S3 y cubos estándar de Google Cloud Services, así como contenedores de Azure Blob. 

> [!NOTE] 
> Se aplica una tarifa de software para usar Avere vFXT en AWS o Google Cloud, pero no con Azure.

## <a name="technical-compute"></a>Técnico: Proceso

### <a name="can-you-describe-what-an-avere-vfxt-environment-looks-like"></a>¿Cómo puede describirse el "aspecto" de un entorno de Avere vFXT?

Avere vFXT es una aplicación en clúster conformada por varias máquinas virtuales de Azure. Una biblioteca de Python controla la creación, eliminación y modificación de clústeres. Lea [¿Qué es Avere vFXT for Azure?](avere-vfxt-overview.md) para más información. 

### <a name="what-kind-of-azure-virtual-machines-does-avere-vfxt-run-on"></a>¿En qué tipo de máquinas virtuales de Azure se ejecuta Avere vFXT?  

Un vFXT Avere para el clúster de Azure usa máquinas virtuales de Microsoft Azure E32s_v3. 

<!-- ### Can I mix and match virtual machine types for my cluster?

No, you must choose one virtual machine type or the other.
    
### Can I move between virtual machine types?

Yes, there is a migration path to move from one VM type to the other. [Open a support ticket](avere-vfxt-open-ticket.md#open-a-support-ticket-for-your-avere-vfxt) to learn how.

-->

### <a name="does-the-avere-vfxt-environment-scale"></a>¿Puede escalarse el entorno de Avere vFXT?

El clúster de Avere vFXT puede ser tan pequeña como tres nodos de máquina virtual o tan grandes como 24 nodos. Póngase en contacto con soporte técnico de Azure para obtener ayuda de planeación si cree que necesita un clúster de más de nueve nodos. Cuantos más nodos tenga, mayor será la arquitectura de implementación.

### <a name="does-the-avere-vfxt-environment-autoscale"></a>¿Puede "escalarse automáticamente" el entorno de Avere vFXT?

No. Puede escalar o reducir verticalmente el tamaño del clúster, pero agregar o quitar nodos de clúster es un paso manual.

### <a name="can-i-run-the-avere-vfxt-cluster-as-a-virtual-machine-scale-set"></a>¿Puedo ejecutar el clúster de Avere vFXT como conjunto de escalado de máquinas virtuales?

Avere vFXT no admite la implementación de un conjunto de escalado de máquinas virtuales. Se han diseñado varios mecanismos de soporte técnico de disponibilidad integrada únicamente para VM atómicas que participan en un clúster.  

### <a name="can-i-run-the-avere-vfxt-cluster-on-low-priority-vms"></a>¿Puedo ejecutar el clúster de Avere vFXT en VM de prioridad baja?

No, el sistema requiere un conjunto estable subyacente de máquinas virtuales.

### <a name="can-i-run-the-avere-vfxt-cluster-in-containers"></a>¿Puedo ejecutar el clúster de Avere vFXT en contenedores?

No, Avere vFXT debe implementarse como aplicación independiente.

### <a name="do-the-avere-vfxt-vms-count-against-my-compute-quota"></a>¿Cuentas las VM de Avere vFXT para la cuota de proceso?

Sí. Asegúrese de tener cuota suficiente en la región para admitir el clúster.  

### <a name="can-i-run-the-avere-vfxt-cluster-machines-in-different-availability-zones"></a>¿Puedo ejecutar las máquinas del clúster de Avere vFXT en zonas de disponibilidad diferentes?

No. El modelo de alta disponibilidad en Avere vFXT actualmente no es compatible con los miembros individuales del clúster de Avere vFXT ubicados en zonas de disponibilidad diferentes.

### <a name="can-i-clone-avere-vfxt-virtual-machines"></a>¿Puedo clonarse las máquinas virtuales de Avere vFXT?

No, debe usar el script de Python compatible para agregar o quitar nodos del clúster de Avere vFXT. Para más información, lea [Manage the Avere vFXT cluster](avere-vfxt-manage-cluster.md) (Administración del clúster de Avere vFXT).  

### <a name="is-there-a-vm-version-of-the-software-i-can-run-in-my-own-local-environment"></a>¿Existe una versión de "VM" del software que puedo ejecutar en mi propio entorno local?

No, el sistema se ofrece como dispositivo en clúster y se ha probado en tipos de máquina virtual específicos. Esta restricción ayuda a los clientes a evitar la creación de un sistema que no sea compatible con los requisitos de alto rendimiento de un flujo de trabajo típico de Avere vFXT. 

## <a name="technical-disks"></a>Técnico: Discos

### <a name="what-types-of-disks-are-supported-for-the-azure-vms"></a>¿Qué tipos de discos son compatibles con las VM de Azure?

Avere vFXT for Azure puede usar configuraciones de SSD Premium de 1 TB o 4 TB. La configuración de SSD Premium se puede implementar como varios discos administrados.

### <a name="does-the-cluster-support-unmanaged-disks"></a>¿Admite el clúster discos no administrados?

No, el clúster requiere discos administrados.

### <a name="does-the-system-support-local-attached-ssds"></a>¿Admite el sistema SSD locales (conectados)?

Avere vFXT for Azure no admite actualmente SSD locales. Los discos usados para Avere vFXT deben poder apagarse y reiniciarse, pero los SSD locales conectados en esta configuración solo se pueden finalizar.

### <a name="does-the-system-support-ultra-ssds"></a>¿Admite el sistema SSD ultra?

No, el sistema admite configuraciones de SSD Premium únicamente.

### <a name="can-i-detach-my-premium-ssds-and-reattach-them-later-to-preserve-cache-contents-between-use"></a>¿Puedo desasociar los SSD Premium y volver a conectarlos más adelante para conservar el contenido en caché entre un uso y otro?

No se admite desasociar y reconectar SSD. Los metadatos o el contenido de los archivos en el origen podrían haber cambiado entre los usos, lo que podrían provocar problemas de integridad de los datos.

### <a name="does-the-system-encrypt-the-cache"></a>¿El sistema cifra la memoria caché?

Los datos se reparten entre los discos, pero no se cifran. Sin embargo, puede cifrar los discos en sí. Para más información, consulte [Protección y uso de directivas en máquinas virtuales en Azure](https://docs.microsoft.com/azure/virtual-machines/linux/security-policy#encryption).

## <a name="technical-networking"></a>Técnico: Redes

### <a name="what-network-is-recommended"></a>¿Qué red se recomienda?

Si se usa almacenamiento local con Avere vFXT, debe tener una conexión de red o de 1 Gbps o mejor. Si tiene una pequeña cantidad de datos y está dispuesto a copiar datos en la nube antes de ejecutar los trabajos, puede ser suficiente la conectividad VPN. 

> [!TIP] 
> Cuanto más lento sea el vínculo de red, más lentas serán las lecturas en frío iniciales. Las lecturas lentas aumentan la latencia de la canalización de trabajos. 

### <a name="can-i-run-avere-vfxt-in-a-different-virtual-network-than-my-compute-cluster"></a>¿Puedo ejecutar Avere vFXT en una red virtual diferente de mi clúster de proceso?

Sí, puede crear el sistema de Avere vFXT en una red virtual diferente. Lea [Planeamiento de un sistema de Avere vFXT](avere-vfxt-deploy-plan.md) para más detalles.

### <a name="does-avere-vfxt-require-its-own-subnet"></a>¿Requiere Avere vFXT su propia subred?

Sí. Avere vFXT ejecuta estrictamente como un clúster de alta disponibilidad (HA) y requiere varias direcciones IP para que funcione. Si el clúster está en su propia subred, se evita el riesgo de conflictos de direcciones IP, lo que puede causar problemas de instalación y de normal funcionamiento. La subred del clúster puede estar dentro de la red virtual existente, siempre y cuando no se superponga ninguna dirección IP.

### <a name="can-i-run-avere-vfxt-on-infiniband"></a>¿Puedo ejecutar Avere vFXT en InfiniBand?

No, Avere vFXT usa solo Ethernet/IP.

### <a name="how-do-i-access-my-on-premises-nas-environment-from-avere-vfxt"></a>¿Cómo accedo a mi entorno de NAS local desde Avere vFXT?

El entorno de Avere vFXT es similar a cualquier otra VM de Azure, en la medida en que requiere acceso enrutado a través de una puerta de enlace de red o VPN al centro de datos del cliente (y a la inversa). Considere el uso de conectividad de Azure ExpressRoute si está disponible en su entorno.

### <a name="what-are-the-bandwidth-requirements-for-avere-vfxt"></a>¿Cuáles son los requisitos de ancho de banda para Avere vFXT?

El requisito de ancho de banda total depende de dos factores: 

* La cantidad de datos que se solicitan desde el origen 
* La tolerancia del sistema del cliente para la latencia durante la carga inicial de datos  

En entornos sensibles a la latencia, debe usar una solución de fibra con una velocidad de vínculo mínima de 1 Gbps. Use ExpressRoute si está disponible.  

### <a name="can-i-run-avere-vfxt-with-public-ip-addresses"></a>¿Puedo ejecutar Avere vFXT con direcciones IP públicas?

No, Avere vFXT está pensada para funcionar en un entorno de red protegido a través de los procedimientos recomendados.  

### <a name="can-i-restrict-internet-access-from-my-clusters-virtual-network"></a>¿Puedo restringir el acceso a internet desde la red virtual de mi clúster? 

En general, puede configurar seguridad adicional en la red virtual según sea necesario, pero algunas restricciones pueden interferir con el funcionamiento del clúster.

Por ejemplo, restringir el acceso saliente a internet desde la red virtual provoca problemas para el clúster a menos que agregue también una regla que permita explícitamente acceso a la nube de Azure. Esta situación se describe en [documentación complementaria en GitHub](https://github.com/Azure/Avere/tree/master/src/vfxt/internet_access.md).

Para obtener ayuda con seguridad personalizada, póngase en contacto con soporte técnico como se describe en [obtener ayuda con su sistema](avere-vfxt-open-ticket.md#open-a-support-ticket-for-your-avere-vfxt).

## <a name="technical-back-end-storage-core-filers"></a>Técnica: Almacenamiento de back-end (archivadores de core)

### <a name="how-many-core-filers-does-a-single-avere-vfxt-environment-support"></a>¿Cuántos archivadores de core admite un único entorno de Avere vFXT?

Un clúster de Avere vFXT admite a un máximo de 20 archivadores de core. 

### <a name="how-does-the-avere-vfxt-environment-store-data"></a>¿De qué manera el entorno de Avere vFXT almacena los datos?

Avere vFXT no es almacenamiento. Es una memoria caché que lee y escribe datos procedentes de varios destinos de almacenamiento denominados archivadores de core. Los datos almacenados en discos SSD Premium de Avere vFXT son transitorios y finalmente se vacían en el almacenamiento de los archivadores de core back-end.

### <a name="which-core-filers-does-avere-vfxt-support"></a>¿Qué archivadores de core admite Avere vFXT?

En términos generales, Avere vFXT for Azure admite los siguientes sistemas como archivadores de core: 

* Dell EMC Isilon (OneFS 7.1, 7.2, 8.0 y 8.1) 
* NetApp ONTAP (Clustered Mode 9.4, 9.3, 9.2, 9.1P1, 8.0-8.3 - 7) y (7-Mode 8.0-8.3.*, 8.0 - 8.3) 

  > [!NOTE] 
  > Los archivos de NetApp Azure no se admite actualmente. 

* Contenedores de blobs de Azure (solo almacenamiento con redundancia local) 
* Cubos de AWS S3 
* Cubos de Google Cloud

### <a name="why-doesnt-avere-vfxt-support-all-nfs-filers"></a>¿Por qué Avere vFXT no admite todos los sistemas de archivadores NFS?

Si bien todas las plataformas NFS cumplen con los mismos estándares de IETF, en la práctica, cada implementación tiene sus propias particularidades. Estos detalles afectan a cómo Avere vFXT interactúa con el sistema de almacenamiento. Los sistemas admitidos son las plataformas más usadas en el mercado.

### <a name="does-avere-vfxt-support-private-object-storage-such-as-swiftstack"></a>¿Admite Avere vFXT almacenamiento de objetos privados (por ejemplo, Swiftstack)?

Avere vFXT no admite el almacenamiento de objetos privados.

### <a name="how-can-i-get-a-specific-storage-product-under-support"></a>¿Cómo puedo obtener compatibilidad para un producto de almacenamiento específico?

La compatibilidad se basa en la cantidad de demanda en el campo. Si hay suficientes solicitudes que generen ingresos para admitir una solución NAS, la tendremos en cuenta. Haga las solicitudes a través del soporte técnico de Azure.

### <a name="can-i-use-azure-blob-storage-as-a-core-filer"></a>¿Puedo usar almacenamiento de Azure Blob como archivador de core?

Sí, Avere vFXT for Azure puede usar un contenedor de blobs de bloque como archivador de core en la nube.  

### <a name="what-are-the-storage-account-requirements-for-a-blob-core-filer"></a>¿Cuáles son los requisitos de la cuenta de almacenamiento para un archivador de core de Blob?

La cuenta de almacenamiento debe ser una cuenta de uso general v2 (GPv2) y estar configurada para el almacenamiento con redundancia local únicamente. No se admiten el almacenamiento con redundancia geográfica ni el almacenamiento con redundancia de zona.

### <a name="can-i-use-archive-blob-storage"></a>¿Puedo usar almacenamiento de blobs de archivo?

No. El contrato de nivel de servicio (SLA) de almacenamiento de archivos no es compatible con las necesidades de acceso a directorios y archivos en tiempo real del sistema Avere vFXT. 

### <a name="can-i-use-cool-blob-storage"></a>¿Puedo usar el almacenamiento de blobs esporádico?

Puede usar el nivel de acceso esporádico, pero tenga en cuenta que la tasa de operaciones será mucho mayor. 

### <a name="how-do-i-encrypt-the-blob-container"></a>¿Cómo se puede cifrar el contenedor de blobs?

Puede configurar el cifrado de blobs en Azure (opción preferida) o en el nivel del archivador de core de Avere vFXT.  

### <a name="can-i-use-my-own-encryption-key-for-a-blob-core-filer"></a>¿Puedo usar mi propia clave de cifrado para un archivador de core de blobs?

De forma predeterminada, los datos se cifran mediante claves administradas por Microsoft para Azure Blob Storage, Table Storage y Queue Storage, además de Azure Files. Puede traer su propia clave de cifrado para Blob Storage y Azure Files. Si decide usar cifrado de Avere vFXT, debe usar la clave generada por Avere y almacenarla localmente. 

## <a name="purchasing"></a>Compra

### <a name="how-do-i-get-avere-vfxt-for-azure-licensing"></a>¿Cómo se puede obtener licencias de Avere vFXT for Azure?

Obtener una licencia de Avere vFXT for Azure es fácil a través de Azure Marketplace. Registre una cuenta de Azure y, luego, siga las instrucciones de [Implementación del clúster de Avere vFXT](avere-vfxt-deploy.md) para crear un clúster de Avere vFXT. 

### <a name="how-much-does-avere-vfxt-cost"></a>¿Cuánto cuesta Avere vFXT?

En Azure, no hay ningún cargo adicional de licencias para usar clústeres de Avere vFXT. Los clientes son responsables de los cargos de almacenamiento y otras cuotas de consumo de Azure.

### <a name="can-avere-vfxt-vms-be-run-as-low-priority"></a>¿Pueden las VM de Avere vFXT ejecutarse como de prioridad baja?

No, los clústeres de Avere vFXT requieren un servicio "siempre disponible". Los clústeres pueden desactivarse cuando no sean necesarios. 

## <a name="next-steps"></a>Pasos siguientes

Para empezar a trabajar con Avere vFXT for Azure, lea estos artículos para obtener información sobre cómo planear e implementar su propio sistema:

* [Planeamiento de un sistema de Avere vFXT](avere-vfxt-deploy-plan.md)
* [Descripción general de la implementación](avere-vfxt-deploy-overview.md)
* [Preparación para la creación de un clúster de Avere vFXT](avere-vfxt-prereqs.md)
* [Implementación del clúster Avere de vFXT](avere-vfxt-deploy.md)

Para más información sobre las funcionalidades y casos de uso de Avere vFXT, visite [Avere vFXT for Azure](https://azure.microsoft.com/services/storage/avere-vfxt/).
