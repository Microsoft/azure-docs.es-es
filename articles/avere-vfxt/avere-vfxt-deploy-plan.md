---
title: Planear un sistema de Avere vFXT - Azure
description: Planee un clúster de Avere vFXT for Azure que responda a sus necesidades. Consulte las preguntas para formular antes de visitar Azure Marketplace o de crear máquinas virtuales.
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 01/21/2020
ms.author: rohogue
ms.openlocfilehash: e5c33144dcebe491757dfafb2879fe51545827ce
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88272797"
---
# <a name="plan-your-avere-vfxt-system"></a>Planear un sistema de Avere vFXT

En este artículo se explica cómo planear una nueva instancia de Avere vFXT para el clúster de Azure que tiene la posición y el tamaño adecuados para sus necesidades.

Antes de ir a Azure Marketplace o de crear máquinas virtuales, tenga en cuenta estos detalles:

* ¿Cómo interactuará el clúster con otros recursos de Azure?
* ¿Dónde deben estar ubicados los elementos de clúster en redes privadas y subredes?
* ¿Qué tipo de almacenamiento de back-end usará y cómo accederá el clúster a él?
* ¿Qué grado de eficacia tienen los nodos de clúster para admitir el flujo de trabajo?

Siga leyendo para obtener más información.

## <a name="learn-the-components-of-the-system"></a>Más información sobre los componentes del sistema

Puede ser útil comprender los componentes del sistema Avere vFXT for Azure al empezar a planear.

* Nodos de clúster: el clúster se compone de tres o más máquinas virtuales configuradas como nodos de clúster. Más nodos proporcionan al sistema mayor rendimiento y una memoria caché mayor.

* Caché: la capacidad de la memoria caché se divide equitativamente entre los nodos del clúster. Establezca el tamaño de la caché por nodo al crear el clúster; los tamaños de nodo se suman para convertirse en el tamaño total de la memoria caché.

* Controlador de clúster: el controlador de clúster es una máquina virtual adicional que se encuentra dentro de la misma subred que los nodos de clúster. El controlador es necesario para crear el clúster y para las tareas de administración en curso.

* Almacenamiento de back-end: los datos que quiere almacenar en caché se almacenan a largo plazo en un sistema de almacenamiento de hardware o en un contenedor de blobs de Azure. Puede agregar almacenamiento después de crear el clúster de Avere vFXT for Azure o, si usa el almacenamiento de blobs, puede agregar y configurar el contenedor al crear el clúster.

* Clientes: las máquinas cliente que usan los archivos en caché se conectan al clúster mediante una ruta de acceso de archivo virtual en lugar de acceder directamente a los sistemas de almacenamiento. (Puede obtener más información en [Montaje del clúster de Avere vFXT](avere-vfxt-mount-clients.md)).

## <a name="subscription-resource-group-and-network-infrastructure"></a>Suscripción, grupo de recursos e infraestructura de red

Piense dónde ubicará los elementos de su implementación de Avere vFXT para Azure. En el siguiente diagrama se muestra una posible disposición para los componentes de Avere vFXT para Azure:

![Diagrama que muestra el controlador y las máquinas virtuales del clúster dentro de una subred. En torno al límite de la subred hay un límite de red virtual. Igualmente, en la red virtual hay un hexágono que representa el punto de conexión del servicio de almacenamiento; tenga en cuenta que se conecta con una flecha discontinua a una instancia de Blob Storage que está fuera de la red virtual.](media/avere-vfxt-components-option.png)

Siga estas instrucciones cuando planee la infraestructura de red del clúster Avere vFXT:

* Cree una nueva suscripción para cada implementación de Avere vFXT for Azure. Administre todos los componentes de esta suscripción.

  Entre las ventajas del uso de una nueva suscripción para cada implementación se incluyen:
  * Seguimiento del costo más sencillo: puede ver y auditar todos los costos de los recursos, la infraestructura y los ciclos de proceso en una única suscripción.
  * Limpieza más sencilla: puede eliminar la suscripción completa una vez finalizado el proyecto.
  * Creación de particiones adecuada de las cuotas de recursos: aísle los clientes y el clúster de Avere vFXT en una suscripción única para proteger otras cargas de trabajo de la posible limitación de recursos. Esta separación evita conflictos al incorporar un gran número de clientes para un flujo de trabajo de computación de alto rendimiento.

* Busque sus sistemas de procesamiento de clientes cerca del clúster de vFXT. El almacenamiento de back-end puede ser aún más remoto.  

* Busque el clúster de vFXT y la máquina virtual del controlador de clúster juntos; en concreto, deben estar en:

  * La misma red virtual
  * El mismo grupo de recursos
  * La misma cuenta de almacenamiento
  
  En la mayoría de las situaciones, la plantilla de creación de clústeres se encarga de esta configuración.

* El clúster debe estar ubicado en su propia subred para evitar conflictos de direcciones IP con los clientes o con otros recursos del proceso.

* Ese la plantilla de creación de clústeres para crear la mayoría de los recursos de infraestructura necesarios para el clúster, incluidos los grupos de recursos, las redes virtuales, las subredes y las cuentas de almacenamiento.

  Si desea usar recursos que ya existen, asegúrese de que cumplen los requisitos de esta tabla.

  | Resource | ¿Usar existente? | Requisitos |
  |----------|-----------|----------|
  | Resource group | Sí, si está vacío | Debe estar vacío|
  | Cuenta de almacenamiento | **Sí**, si se conecta un contenedor de blobs existente después de la creación del clúster <br/>  **No**, si crea un contenedor de blobs durante la creación del clúster | El contenedor de blobs existente debe estar vacío <br/> &nbsp; |
  | Virtual network | Sí | Debe incluir un punto de conexión de servicio de almacenamiento si va a crear un nuevo contenedor de blobs de Azure. |
  | Subnet | Sí | No puede contener otros recursos |

## <a name="ip-address-requirements"></a>Requisitos de las direcciones IP

Asegúrese de que la subred del clúster tenga un rango de direcciones IP lo suficientemente grande como para admitir el clúster.

El clúster de Avere vFXT usa las siguientes direcciones IP:

* Una dirección IP de gestión de clúster. Esta dirección puede moverse de un nodo a otro del clúster según sea necesario para que siempre esté disponible. Use esta dirección para conectarse a la herramienta de configuración del panel de control de Avere.
* Para cada nodo del clúster:
  * Al menos una dirección IP orientada al cliente. (El *servidor virtual* del clúster se encarga de administrar todas las direcciones orientadas al cliente y puede mover las direcciones IP entre los nodos según sea necesario).
  * Una dirección IP para la comunicación del clúster.
  * Una dirección IP de la instancia (asignada a la máquina virtual).

Si usa Azure Blob Storage, también puede requerir direcciones IP de la red virtual del clúster:  

* Una cuenta de Azure Blob Storage requiere al menos cinco direcciones IP. Tenga en cuenta este requisito si ubica Blob Storage en la misma red virtual que el clúster.
* Si usa una instancia de Azure Blob Storage que esté fuera de la red virtual del clúster, cree un punto de conexión de servicio de almacenamiento en la red virtual. Tenga en cuenta que este punto de conexión no usa una dirección IP.

Asimismo, tiene la opción de ubicar los recursos de red y Blob Storage (si lo va a usar) en diferentes grupos de recursos del clúster.

## <a name="vfxt-node-size"></a>Tamaño de los nodos de vFXT

Las máquinas virtuales que sirven como nodos de clúster determinan el rendimiento de la solicitud y la capacidad de almacenamiento de la caché. <!-- The instance type offered has been chosen for its memory, processor, and local storage characteristics. You can choose from two instance types, with different memory, processor, and local storage characteristics. -->

Cada nodo de vFXT será idéntico. Es decir, si crea un clúster de tres nodos, tendrá tres máquinas virtuales del mismo tipo y tamaño.

| Tipo de instancia | vCPU | Memoria  | Almacenamiento local de SSD  | Discos de datos máx. | Rendimiento del disco sin almacenamiento en la caché | NIC (recuento) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_E32s_v3 | 32  | 256 GiB | 512 GB  | 32 | 51 200 IOPS <br/> 768 MBps | 16 000 MBps (8)  |

La caché de disco por nodo se puede configurar y puede oscilar entre los 1000 GB y los 8000 GB. El tamaño recomendado de la memoria caché es de 4 TB por nodo para los nodos Standard_E32s_v3.

Para obtener información adicional sobre estas máquinas virtuales, lea la documentación de Microsoft Azure: [Tamaños de máquina virtual optimizada para memoria](https://docs.microsoft.com/azure/virtual-machines/windows/sizes-memory)

## <a name="account-quota"></a>Cuotas de la cuenta

Asegúrese de que su suscripción tenga la capacidad de ejecutar el clúster de Avere vFXT, así como los sistemas de informática o cliente que vaya a usar. Lea [Cuota del clúster de vFXT](avere-vfxt-prereqs.md#quota-for-the-vfxt-cluster) para obtener más detalles.

## <a name="back-end-data-storage"></a>Almacenamiento de datos de back-end

Los sistemas de almacenamiento de back-end suministran archivos a la memoria caché del clúster y también reciben datos modificados de la memoria caché. Decida si el espacio de trabajo se almacenará a largo plazo en un nuevo contenedor de blobs o en un sistema de almacenamiento de existente (nube o hardware). Estos sistemas de almacenamiento de back-end se denominan *archivadores principales*.

### <a name="hardware-core-filers"></a>Archivadores principales de hardware

Agregue sistemas de almacenamiento de hardware al clúster de vFXT después de crearlo. Puede usar diversos sistemas de hardware conocidos, incluidos los sistemas locales, siempre que se pueda acceder al sistema de almacenamiento desde la subred del clúster.

Lea [Configure storage](avere-vfxt-add-storage.md) (Configurar el almacenamiento) para obtener instrucciones detalladas sobre cómo agregar un sistema de almacenamiento existente al clúster de Avere vFXT.

### <a name="cloud-core-filers"></a>Archivadores principales en la nube

El sistema Avere vFXT for Azure puede usar contenedores de blobs vacíos para el almacenamiento de back-end. Los contenedores deben estar vacíos cuando se agregan al clúster: el sistema vFXT debe poder administrar su almacén de objetos sin necesidad de conservar los datos existentes.

> [!TIP]
> Si quiere usar Azure Blob Storage para el back-end, cree un contenedor como parte de la creación del clúster de vFXT. La plantilla de creación de clústeres puede crear y configurar un nuevo contenedor de blobs para que esté listo para usarse en cuanto el clúster esté disponible. Agregar un contenedor más tarde es más complicado.
>
> Lea [Creación del clúster de Avere vFXT for Azure](avere-vfxt-deploy.md#create-the-avere-vfxt-for-azure) para más información.

Después de agregar el contenedor de Blob Storage vacío como archivador principal, puede copiar los datos en él a través del clúster. Use un mecanismo de copia multiproceso en paralelo. Lea [Moving data to the vFXT cluster](avere-vfxt-data-ingest.md) (Mover datos al clúster de vFXT) para aprender a copiar los datos en el nuevo contenedor del clúster de manera eficaz mediante el uso de máquinas cliente y la caché de Avere vFXT.

## <a name="cluster-access"></a>Acceso al clúster

El clúster de Avere vFXT for Azure se encuentra en una subred privada, y el clúster no tiene ninguna dirección IP pública. Debe tener algún modo de acceder a la subred privada para la administración del clúster y las conexiones de clientes.

Los puntos de acceso incluyen:

* Host de salto: asigne una dirección IP pública a una VM independiente dentro de la red privada y úsela para crear un túnel TLS hasta los nodos del clúster.

  > [!TIP]
  > Si configura una dirección IP pública en el controlador del clúster, puede usarla como host de salto. Lea [Controlador de clústeres como host de salto](#cluster-controller-as-jump-host) para más información.

* Red privada virtual (VPN): configure una VPN de sitio a sitio o de punto a sitio entre la red privada en Azure y las redes corporativas.

* Azure ExpressRoute: configure una conexión privada mediante un asociado de ExpressRoute.

Para más información sobre estas opciones, vea la [documentación de Azure Virtual Network relativa a la comunicación de Internet](../virtual-network/virtual-networks-overview.md#communicate-with-the-internet).

### <a name="cluster-controller-as-jump-host"></a>Controlador de clústeres como host de salto

Si configura una dirección IP pública en un controlador del clúster, puede usarla como un host de salto para conectarse al clúster de Avere vFXT desde fuera de la subred privada. Sin embargo, dado que el controlador tiene privilegios de acceso para modificar nodos del clúster, plantea un pequeño riesgo de seguridad.

Para mejorar la seguridad para un controlador con una dirección IP pública, el script de implementación crea automáticamente un grupo de seguridad de red que restrinja el acceso entrante solo al puerto 22. Puede proteger aún más el sistema al bloquear el acceso al intervalo de direcciones IP de origen, es decir, solo permitir las conexiones desde las máquinas que pretende usar para el acceso al clúster.

Al crear el clúster, puede elegir si desea crear o no una dirección IP pública en el controlador del clúster.

* Si crea una **red virtual** o **una subred**, se le asignará una dirección IP **pública** al controlador del clúster.
* Si selecciona una red virtual y una subred existentes, el controlador del clúster solo tendrá direcciones IP **privadas**.

## <a name="vm-access-roles"></a>Roles de acceso a máquinas virtuales

Azure usa el [control de acceso basado en roles](../role-based-access-control/index.yml) (RBAC) para autorizar que las máquinas virtuales del clúster realicen determinadas tareas. Por ejemplo, el controlador de clúster necesita autorización para crear y configurar las máquinas virtuales del nodo de clúster. Los nodos de clúster deben poder asignar o reasignar direcciones IP a otros nodos de clúster.

Se utilizan dos roles de Azure integrados para las máquinas de virtuales de Avere vFXT:

* El controlador de clúster utiliza el rol integrado [Colaborador de Avere](../role-based-access-control/built-in-roles.md#avere-contributor).
* Los nodos de clúster utilizan el rol integrado [Operador de Avere](../role-based-access-control/built-in-roles.md#avere-operator).

Si necesita personalizar los roles de acceso para los componentes de vFXT Avere, debe definir su propio rol y asignarlo a las máquinas virtuales al crearlos. No se puede usar la plantilla de implementación en Azure Marketplace. Consulte al servicio de Soporte técnico y Atención al cliente de Microsoft. abriendo una incidencia en Azure Portal, como se describe en [Obtener ayuda con su sistema](avere-vfxt-open-ticket.md).

## <a name="next-steps"></a>Pasos siguientes

[Introducción a la implementación](avere-vfxt-deploy-overview.md) ofrece una visión general de todos los pasos necesarios para crear una instancia del sistema Avere vFXT for Azure y prepararla para suministrar datos.
