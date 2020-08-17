---
title: Nodos y grupos en Azure Batch
description: Obtenga información sobre los grupos y nodos de proceso, y cómo se usan en un flujo de trabajo de Azure Batch desde el punto de vista del desarrollo.
ms.topic: conceptual
ms.date: 06/16/2020
ms.openlocfilehash: 16a5309711b9c8633da9ba473c1b55bc2e54c334
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87385762"
---
# <a name="nodes-and-pools-in-azure-batch"></a>Nodos y grupos en Azure Batch

En un flujo de trabajo de Azure Batch, un *nodo de proceso* (o *nodo*) es una máquina virtual que procesa una parte de la carga de trabajo de la aplicación. Un *grupo* es una colección de estos nodos para que la aplicación se ejecute. En este artículo se ofrece más información sobre nodos y grupos, junto con consideraciones al crearlos y usarlos en un flujo de trabajo de Azure Batch.

## <a name="nodes"></a>Nodos

Un nodo es una máquina virtual de Azure o una máquina virtual de servicio en la nube dedicada al procesamiento de una parte de la carga de trabajo de la aplicación. El tamaño de un nodo determina el número de núcleos de CPU, la capacidad de memoria y el tamaño del sistema de archivos local que se asignan al nodo.

Puede crear grupos de nodos de Windows o Linux mediante Azure Cloud Services, imágenes de [Azure Virtual Machines Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/category/compute?filters=virtual-machine-images&page=1) o imágenes personalizadas que prepare.

Los nodos pueden ejecutar cualquier archivo ejecutable o script compatible con el entorno de sistema operativo del nodo. Los scripts o archivos ejecutables incluyen scripts \*.exe, \*.cmd, \*.bat y de PowerShell (para Windows) además de archivos binarios y scripts de shell y de Python (para Linux).

Todos los nodos de proceso en Batch también incluyen:

- Una [estructura de carpetas](files-and-directories.md) estándar y [variables de entorno](jobs-and-tasks.md) asociadas, disponibles para que las tareas hagan referencia a ellas.
- **firewall** para controlar el acceso.
- [Acceso remoto](error-handling.md#connect-to-compute-nodes) a nodos de Windows (Protocolo de escritorio remoto (RDP)) y nodos de Linux (Secure Shell (SSH)).

De forma predeterminada, los nodos pueden comunicarse entre sí, pero no pueden comunicarse con máquinas virtuales que no forman parte del mismo grupo. Para permitir que los nodos se comuniquen de manera segura con otras máquinas virtuales o con una red local, puede aprovisionar el grupo [en una subred de una red de Azure Virtual Network (VNet)](batch-virtual-network.md). Al hacerlo, se puede tener acceso a los nodos a través de direcciones IP públicas. Batch crea estas direcciones IP públicas, que pueden cambiar durante la vigencia del grupo. También puede [crear un grupo con direcciones IP públicas estáticas](create-pool-public-ip.md) bajo su control, lo que garantiza que no cambiarán de forma inesperada.

## <a name="pools"></a>Grupos

Un grupo es la colección de nodos en la que se ejecuta la aplicación.

Los grupos de Azure Batch se basan en la plataforma de proceso principal de Azure. Proporcionan asignación a gran escala, instalación de aplicaciones, distribución de datos, supervisión de estado y ajuste flexible ([escalado](#automatic-scaling-policy)) del número de nodos de proceso en un grupo.

A cada nodo que se agrega a un grupo se le asigna un nombre y una dirección IP únicos. Cuando se quita un nodo de un grupo, los cambios realizados en el sistema operativo o los archivos se pierden, y su nombre y dirección IP se liberan para usarlos en un futuro. Cuando un nodo abandona un grupo, su vigencia finaliza.

Un grupo solo se puede usar con la cuenta de Batch en la que se creó. Una cuenta de Batch puede crear varios grupos para satisfacer los requisitos de recursos de las aplicaciones que ejecutará.

El grupo se puede crear de forma manual, o bien automáticamente por el servicio Batch cuando se especifica el trabajo que se va a realizar. Cuando se crea un grupo, puede especificar los siguientes atributos:

- [Sistema operativo y versión del nodo](#operating-system-and-version)
- [Tipo de nodo y número de nodos de destino](#node-type-and-target)
- [Tamaño del nodo](#node-size)
- [Directivas de escalado automático](#automatic-scaling-policy)
- [Directiva de programación de tareas](#task-scheduling-policy)
- [Estado de la comunicación](#communication-status)
- [Tareas de inicio](#start-tasks)
- [Paquetes de aplicación](#application-packages)
- [Configuración de red virtual (VNet) y de firewall](#virtual-network-vnet-and-firewall-configuration)
- [Duración](#pool-and-compute-node-lifetime)

> [!IMPORTANT]
> Las cuentas de Batch tienen una cuota predeterminada que limita el número de núcleos en ellas. El número de núcleos corresponde al número de nodos de proceso. Tanto las cuotas predeterminadas como las instrucciones para [aumentar una cuota](batch-quota-limit.md#increase-a-quota) se pueden encontrar en [Límites y cuotas del servicio de Batch de Azure](batch-quota-limit.md). Si el grupo no está consiguiendo su número de nodos de destino, la causa podría ser la cuota básica.

## <a name="operating-system-and-version"></a>Sistema operativo y versión

Al crear un grupo de Batch, se especifica la configuración de máquina virtual de Azure y el tipo de sistema operativo que quiere ejecutar en cada nodo de proceso del grupo.

## <a name="configurations"></a>Configurations

Hay dos tipos de configuraciones de grupo disponibles en Batch.

### <a name="virtual-machine-configuration"></a>Configuración de máquina virtual

La **Configuración de máquina virtual** especifica que el grupo está formado por máquinas virtuales de Azure. Estas máquinas virtuales pueden crearse a partir de imágenes de Linux o Windows.

Cuando crea un grupo basado en la configuración de máquina virtual, debe especificar no solo el tamaño de los nodos y el origen de las imágenes utilizadas para crearlos, sino también la **referencia de la imagen de máquina virtual** y la **SKU del agente de nodo** de Batch que desea instalar en los nodos. Para más información sobre cómo especificar estas propiedades del grupo, consulte [Aprovisionamiento de nodos de proceso de Linux en grupos del servicio Azure Batch](batch-linux-nodes.md). También puede asociar uno o más discos de datos vacíos a las máquinas virtuales del grupo creadas con imágenes de Marketplace, o bien incluir discos de datos en imágenes personalizadas usadas para crear las máquinas virtuales. Al incluir discos de datos, deberá montar y dar formato a los discos desde una máquina virtual para usarlos.

### <a name="cloud-services-configuration"></a>Configuración de Cloud Services

La **Configuración de Cloud Services** especifica que el grupo está formado por nodos de Azure Cloud Services. Cloud Services proporciona *solo* nodos de ejecución Windows.

La lista de los sistemas operativos disponibles para los grupos de configuración de Cloud Services aparece en [Matriz de compatibilidad del SDK y versiones del SO invitado de Azure](../cloud-services/cloud-services-guestos-update-matrix.md). Cuando crea un grupo que contiene nodos de Cloud Services, tiene que especificar el tamaño del nodo y su *familia de sistema operativo* (que determina qué versiones de .NET se instalan con el sistema operativo). Cloud Services se implementa en Azure de forma más rápida que las máquinas virtuales que ejecutan Windows. Si desea grupos de nodos de proceso Windows, es posible que Cloud Services proporcione una mejora en el rendimiento en términos de tiempo de implementación.

Al igual que con los roles de trabajo en Cloud Services, es posible especificar la *versión del sistema operativo* (para más información sobre los roles de trabajo, consulte la [introducción a Cloud Services](../cloud-services/cloud-services-choose-me.md)). Se recomienda especificar `Latest (*)` para la *versión del sistema operativo* de forma que los nodos se actualicen automáticamente; así no es necesario realizar ningún trabajo para las versiones recién publicadas. El principal caso de uso para seleccionar una versión específica del sistema operativo es asegurarse de la compatibilidad de las aplicaciones; debe ser posible realizar pruebas de compatibilidad con versiones anteriores antes de permitir la actualización de la versión. Después de la validación, se puede actualizar la *versión del sistema operativo* para el grupo e instalar la nueva imagen del sistema operativo. Cualquier tarea en ejecución se interrumpirá y se volverá a poner en cola.

### <a name="node-agent-skus"></a>SKU de agentes de nodos

Cuando crea un grupo, debe seleccionar la opción **nodeAgentSkuId** apropiada, en función del sistema operativo de la imagen base de su VHD. Puede obtener una asignación de identificadores de SKU de agentes de nodos disponibles a sus referencias de imágenes del sistema operativo mediante una llamada a la operación [List supported node agent SKUs](/rest/api/batchservice/list-supported-node-agent-skus) (Lista de SKU admitidas de agentes de nodos).

### <a name="custom-images-for-virtual-machine-pools"></a>Imágenes personalizadas de grupos de máquinas virtuales

Para obtener información sobre cómo crear un grupo con imágenes personalizadas, consulte [Uso de Shared Image Gallery para crear un grupo personalizado](batch-sig-images.md).

Como alternativa, puede crear un grupo personalizado de máquinas virtuales con un recurso de [imagen administrada](batch-custom-images.md). Para obtener información acerca de cómo preparar imágenes personalizadas de Linux desde máquinas virtuales de Azure, consulte [Creación de una imagen de una máquina virtual o un disco duro virtual](../virtual-machines/linux/capture-image.md). Para obtener información acerca de cómo preparar imágenes de Windows personalizadas a partir de máquinas virtuales de Azure, consulte [Captura de una imagen administrada de una máquina virtual generalizada en Azure](../virtual-machines/windows/capture-image-resource.md).

### <a name="container-support-in-virtual-machine-pools"></a>Compatibilidad con contenedores en grupos de máquinas virtuales

Al crear un grupo de configuración de máquinas virtuales mediante las API de Batch, puede configurar el grupo para ejecutar tareas en contenedores de Docker. Actualmente, debe crear el grupo con una imagen que admita contenedores de Docker. Use Windows Server 2016 Datacenter con la imagen Containers de Azure Marketplace, o proporcione una imagen de máquina virtual personalizada que incluya Docker Community Edition o Enterprise y los controladores necesarios. La configuración del grupo debe incluir una [configuración de contenedor](/rest/api/batchservice/pool/add) que copie las imágenes del contenedor en las máquinas virtuales cuando se crea el grupo. De esta forma, las tareas que se ejecutan en el grupo pueden hacer referencia a las imágenes y a las opciones de ejecución del contenedor.

Para más información, vea [Ejecución de aplicaciones de contenedor de Docker en Azure Batch](batch-docker-container-workloads.md).

## <a name="node-type-and-target"></a>Tamaño y destino de nodo

Al crear un grupo, puede especificar qué tipos de nodos le interesan y un número de destino para cada uno. Los dos tipos de nodos son los siguientes:

- **Nodos dedicados.** Los nodos de proceso dedicados están reservados para las cargas de trabajo que usted tenga. Son más caros que los nodos de prioridad baja, pero se garantiza que nunca se verán reemplazados.
- **Nodos de prioridad baja.** Los nodos de prioridad baja aprovechan la capacidad sobrante en Azure para ejecutar las cargas de trabajo de Batch. Los nodos de prioridad baja son menos costosos por hora que los nodos dedicados y habilitan las cargas de trabajo que requieren una considerable potencia de proceso. Para obtener más información, vea [Use low-priority VMs with Batch](batch-low-pri-vms.md) (Usar máquinas virtuales de prioridad baja con Batch).

Los nodos de prioridad baja se pueden reemplazar cuando Azure no tiene capacidad sobrante suficiente. Si se reemplaza un nodo mientras ejecuta tareas, las tareas se vuelven a poner en cola y se ejecutan de nuevo en cuanto está disponible un nodo de proceso. Los nodos de prioridad baja son una buena opción para las cargas de trabajo en las que el tiempo de finalización del trabajo es flexible y el trabajo se distribuye entre muchos nodos. Antes de decidirse a usar nodos de prioridad baja para un escenario, asegúrese de que los trabajos perdidos debido a la preferencia sean mínimos y fáciles de volver a crear.

Puede tener nodos de proceso de prioridad baja y dedicados en el mismo grupo. Cada tipo de nodo tiene su propia configuración de destino, para la que puede especificar el número deseado de nodos.

El número de nodos de proceso se conoce como *destino* porque, en algunas situaciones, es posible que el grupo no alcance el número deseado de nodos. Por ejemplo, un grupo podría no alcanzar el destino si alcanza primero la [cuota de núcleos](batch-quota-limit.md) de la cuenta de Batch. O bien, es posible que el grupo no alcance el destino si le ha aplicado una fórmula de escalado automático que limita el número máximo de nodos.

Para obtener información sobre los precios de los nodos de prioridad baja y dedicados, vea [Precios de Batch](https://azure.microsoft.com/pricing/details/batch/).

## <a name="node-size"></a>Tamaño del nodo

Cuando se crea un grupo de Azure Batch, puede elegir entre casi todas las familias y tamaños de máquinas virtuales disponibles en Azure. Azure ofrece varios tamaños de máquina virtual apropiados para las distintas cargas de trabajo, entre los que se incluyen los tamaños de máquina virtual especializados con [HPC](../virtual-machines/sizes-hpc.md) o [Habilitados para GPU](../virtual-machines/sizes-gpu.md). 

Para más información, consulte [Choose a VM size for compute nodes in an Azure Batch pool](batch-pool-vm-sizes.md) (Elección de un tamaño de máquina virtual para nodos de proceso en un grupo de Azure Batch).

## <a name="automatic-scaling-policy"></a>Directivas de escalado automático

Para cargas de trabajo dinámicas, puede aplicar una directiva de escalado automático a un grupo. El servicio Batch evaluará de forma periódica la fórmula y ajusta dinámicamente el número de nodos del grupo según la carga de trabajo actual y el uso de los recursos en el escenario de proceso. Esto le permite reducir el costo general de la ejecución de la aplicación usando solo los recursos que necesita, y liberando los que no.

Para habilitar el escalado automático, escriba una [fórmula de escalado automático](batch-automatic-scaling.md#autoscale-formulas) y asóciela con un grupo. El servicio Batch usa la fórmula para determinar el número objetivo de nodos del grupo para el siguiente intervalo de escalado (un intervalo que puede configurar). Puede especificar la configuración de escalado automático para un grupo cuando lo cree o habilitar el escalado en un grupo más adelante. También puede actualizar la configuración de escalado en un grupo habilitado para el escalado.

Como ejemplo, quizás un trabajo requiera que envíe un gran número de tareas para que se ejecuten. Puede asignar al grupo una fórmula de escalado que ajuste el número de nodos del grupo en función del número actual de tareas en cola y la tasa de finalización de las tareas del trabajo. El servicio Batch evalúa periódicamente la fórmula y cambia el tamaño del grupo en función de la carga de trabajo y del resto de ajustes de configuración de la fórmula. El servicio agrega nodos según sea necesario cuando haya un gran número de tareas en cola y quita los nodos cuando no haya ninguna tarea en cola o en ejecución.

Una fórmula de escalado puede basarse en las siguientes métricas:

- **Métricas de tiempo** : se basan en las estadísticas recopiladas cada cinco minutos en el número especificado de horas.
- **Métricas de recursos** : se basan en el uso de CPU, ancho de banda y memoria y en el número de nodos.
- **Métricas de tareas**: se basan en el estado de la tarea, como *Activa* (en cola), *En ejecución* o *Completada*.

Cuando el escalado automático reduce el número de nodos de proceso en un grupo, debe considerar cómo administrará las tareas que se están ejecutando en el momento en que se realiza la operación de reducción. Para ello, Batch proporciona una [*opción de desasignación de nodos*](/rest/api/batchservice/pool/removenodes#computenodedeallocationoption) que se puede incluir en las fórmulas. Por ejemplo, puede especificar que las tareas en ejecución se detengan de inmediato y se vuelvan a poner en cola para ejecutarlas en otro nodo o se dejen finalizar antes de que se quite el nodo del grupo. Tenga en cuenta que si se establece la opción de desasignación de nodos en `taskcompletion` o `retaineddata`, se evitarán las operaciones de cambio de tamaño de los grupos hasta que se completen todas las tareas o hasta que expiren todos los períodos de retención de tareas, respectivamente.

Para obtener más información sobre cómo escalar automáticamente una aplicación, consulte [Escalado automático de nodos de ejecución en un grupo de Azure Batch](batch-automatic-scaling.md).

> [!TIP]
> Para maximizar el uso de los recursos de proceso, establezca en cero el número objetivo de nodos al final de un trabajo, pero permita que las tareas en ejecución finalicen.

## <a name="task-scheduling-policy"></a>Directiva de programación de tareas

La opción de configuración [Máximo de tareas por nodo](batch-parallel-node-tasks.md) determina la cantidad máxima de tareas que se pueden ejecutar en paralelo en cada nodo de proceso del grupo.

La configuración predeterminada especifica que se ejecuta una sola tarea al mismo tiempo en un nodo. Sin embargo, hay situaciones en las que resulta ventajoso que se ejecuten simultáneamente dos o más tareas en un nodo. Consulte el [escenario de ejemplo](batch-parallel-node-tasks.md#example-scenario) en el artículo sobre [tareas simultáneas de nodo](batch-parallel-node-tasks.md) para ver cómo puede beneficiarse de la ejecución de varias tareas por nodo.

También puede especificar un *tipo de relleno* que determina si el servicio Batch distribuye las tareas uniformemente entre todos los nodos de un grupo o llena cada uno con el número máximo de tareas antes de asignar tareas a otro.

## <a name="communication-status"></a>Estado de la comunicación

En la mayoría de los escenarios, las tareas funcionan de forma independiente y no necesitan comunicarse entre sí. Sin embargo, hay algunas aplicaciones en las que las tareas deben comunicarse, como en los [escenarios MPI](batch-mpi.md).

Puede configurar un grupo que permita la **comunicación entre nodos**, de manera que los nodos de un grupo se puedan comunicar en tiempo de ejecución. Cuando se habilita la comunicación entre nodos, los nodos en grupos de configuración de Cloud Services pueden comunicarse entre sí en los puertos superiores a 1100, mientras que los grupos de configuración de máquina virtual no restringen el tráfico en ningún puerto.

Habilitar la comunicación entre nodos también afecta a la colocación de los nodos dentro de los clústeres y puede limitar el número máximo de nodos en un grupo a causa de las restricciones de implementación. Si la aplicación no requiere comunicación entre los nodos, el servicio Batch puede asignar un número potencialmente alto de nodos al grupo desde muchos clústeres y centros de datos diferentes para permitir el aumento de la potencia del procesamiento paralelo.

## <a name="start-tasks"></a>Tareas de inicio

Si es necesario, puede agregar una [tarea de inicio](jobs-and-tasks.md#start-task) que se ejecutará en cada nodo cada vez que se une al grupo y cada vez que se reinicia un nodo o se restablece su imagen inicial. La tarea de inicio resulta especialmente útil para preparar los nodos de proceso para la ejecución de tareas, como la instalación de las aplicaciones que las tareas ejecutan en los nodos de proceso.

## <a name="application-packages"></a>paquetes de aplicación

Puede especificar paquetes de aplicación que se implementarán en los nodos de proceso del grupo. Los paquetes de aplicación proporcionan una implementación simplificada y el control de las versiones de las aplicaciones que ejecutan las tareas. Los paquetes de aplicación que se especifiquen para un grupo se instalarán en todos los nodos que se unan al grupo cada vez que un nodo se reinicie o se restablezca la imagen inicial.

Para más información sobre el uso de los paquetes de aplicación para implementar las aplicaciones en los nodos de Batch, consulte [Implementación de aplicaciones en nodos de proceso con paquetes de aplicaciones de Batch](batch-application-packages.md).

## <a name="virtual-network-vnet-and-firewall-configuration"></a>Configuración de red virtual y de firewall

Al aprovisionar un grupo de nodos de proceso en Batch, dicho grupo se puede asociar a una subred de una [red virtual](../virtual-network/virtual-networks-overview.md) de Azure. Para usar una red virtual de Azure, la API de cliente de Batch debe usar la autenticación de Azure Active Directory (AD). La compatibilidad de Azure Batch con Azure AD se documenta en [Autenticación de soluciones de servicio de Batch con Active Directory](batch-aad-auth.md).

### <a name="vnet-requirements"></a>Requisitos de la red virtual

[!INCLUDE [batch-virtual-network-ports](../../includes/batch-virtual-network-ports.md)]

Para más información sobre cómo configurar un grupo de Batch en una red virtual, consulte [Create a pool of virtual machines with your virtual network](batch-virtual-network.md) (Creación de un grupo de máquinas virtuales con la red virtual).

> [!TIP]
> Para asegurarse de que las direcciones IP públicas usadas para tener acceso a los nodos no cambian, puede [crear un grupo con direcciones IP públicas especificadas bajo su control](create-pool-public-ip.md).

## <a name="pool-and-compute-node-lifetime"></a>Vigencia de grupo y nodo de proceso

Al diseñar la solución de Azure Batch, debe especificar cómo y cuándo se crean los grupos y cuánto tiempo se mantendrán disponibles los nodos de proceso dentro de esos grupos.

Por una parte, puede crear un grupo para cada trabajo que envíe el trabajo y eliminarlo a medida que se terminen de ejecutar las tareas. Esto maximiza su uso ya que los nodos solo se asignan cuando es necesario y se apagan en cuanto se vuelven inactivos. Aunque esto significa que el trabajo debe esperar a que se asignen los nodos, es importante tener en cuenta que las tareas se programarán para su ejecución tan pronto como los nodos se asignen de forma individual y se haya completado la tarea de inicio. Batch *no* espera a que todos los nodos de un grupo estén disponibles antes de asignarles tareas. Esto garantiza la utilización máxima de todos los nodos disponibles.

Por otro lado, si lo más importante es que los trabajos se inicien inmediatamente, se puede crear un grupo antes de tiempo y hacer que sus nodos estén disponibles antes de que se envíen los trabajos. En este escenario, las tareas se pueden iniciar inmediatamente, pero los nodos permanecerán inactivos mientras esperan a que se les asignen las tareas.

Normalmente, se usa un enfoque combinado para controlar una carga variable, pero en curso. Puede tener un grupo en el que se envíen varios trabajos y puede escalar o reducir verticalmente el número de nodos según la carga del trabajo. Puede realizar esto de manera reactiva, según la carga actual o de forma anticipada si se puede predecir la carga. Para obtener más información, vea [Directiva de escalado automático](#automatic-scaling-policy).

## <a name="security-with-certificates"></a>Seguridad con certificados

Normalmente necesitará usar certificados al cifrar o descifrar información confidencial para las tareas, como la clave de una [cuenta de Azure Storage](accounts.md#azure-storage-accounts). Para ello, puede instalar certificados en los nodos. Los secretos cifrados se pasan a las tareas mediante parámetros de línea de comandos o se insertan en uno de los recursos de tarea, y los certificados instalados se pueden usar para descifrarlos.

Use la operación [Agregar certificado](/rest/api/batchservice/certificate/add) (REST de Batch) o el método [CertificateOperations.CreateCertificate](/dotnet/api/microsoft.azure.batch.certificateoperations) (Batch para .NET) para agregar un certificado a una cuenta de Batch. Se puede asociar el certificado a un grupo nuevo o existente.

Cuando se asocia un certificado a un grupo, el servicio Batch instala el certificado en cada nodo del grupo. El servicio Batch instala los certificados adecuados cuando se inicia el nodo, antes de iniciar ninguna tarea (incluidas la de [inicio](jobs-and-tasks.md#start-task) y la del [administrador de trabajos](jobs-and-tasks.md#job-manager-task)).

Si agrega un certificado a un grupo existente, tendrá que reiniciar sus nodos de proceso para que el certificado se pueda aplicar a los nodos.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre [trabajos y tareas](jobs-and-tasks.md).
