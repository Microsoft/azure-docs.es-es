---
title: Procedimientos recomendados para el almacenamiento y las copias de seguridad
titleSuffix: Azure Kubernetes Service
description: Obtenga más información acerca de los procedimientos recomendados del operador de clústeres para el almacenamiento, el cifrado de datos y las copias de seguridad en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 722fe393ad7637be20360463a4c3b6234224a036
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653977"
---
# <a name="best-practices-for-storage-and-backups-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para el almacenamiento y las copias de seguridad en Azure Kubernetes Service (AKS)

A medida que se crean y gestionan clústeres en Azure Kubernetes Service (AKS), las aplicaciones a menudo necesitan almacenamiento. Es importante comprender las necesidades de rendimiento y los métodos de acceso a los pods para que pueda proporcionar el almacenamiento adecuado a las aplicaciones. El tamaño del nodo de AKS puede afectar a estas opciones de almacenamiento. También debe planear las formas de realizar copias de seguridad y probar el proceso de restauración en busca de almacenamiento conectado.

Este artículo de procedimientos recomendados se centra en las consideraciones de almacenamiento de los operadores de clústeres. En este artículo, aprenderá lo siguiente:

> [!div class="checklist"]
> * ¿Qué tipos de almacenamiento están disponibles?
> * Cómo dimensionar correctamente el tamaño de los nodos de AKS para el rendimiento del almacenamiento
> * Diferencias entre el aprovisionamiento dinámico y estático de volúmenes
> * Formas de realizar copias de seguridad y proteger los volúmenes de datos

## <a name="choose-the-appropriate-storage-type"></a>Elección del tipo de almacenamiento adecuado

**Orientación con procedimientos recomendados**: comprenda las necesidades de la aplicación para seleccionar el almacenamiento correcto. Utilice almacenamiento de alto rendimiento respaldado por SSD para cargas de trabajo de producción. Planee el almacenamiento basado en red cuando se necesiten varias conexiones simultáneas.

Las aplicaciones a menudo requieren diferentes tipos y velocidades de almacenamiento. ¿Necesitan sus aplicaciones almacenamiento que se conecte a pods individuales o que se comparta entre varios pods? ¿Es el almacenamiento para acceso de solo lectura a datos, o para escribir grandes cantidades de datos estructurados? Estas necesidades de almacenamiento determinan el tipo más adecuado de almacenamiento que se va a usar.

En la tabla siguiente se describen los tipos de almacenamiento disponibles y sus capacidades:

| Caso de uso | Complemento de volumen | Lectura/escritura una vez | Solo lectura varias veces | Lectura/escritura varias veces | Compatibilidad con contenedores de Windows Server |
|----------|---------------|-----------------|----------------|-----------------|--------------------|
| Configuración compartida       | Azure Files   | Sí | Sí | Sí | Sí |
| Datos de la aplicación estructurados        | Azure Disks   | Sí | No  | No  | Sí |
| Datos no estructurados, operaciones del sistema de archivos | [BlobFuse][blobfuse] | Sí | Sí | Sí | No |

Los dos tipos principales de almacenamiento proporcionados para volúmenes en AKS están respaldados por Azure Disks o Azure Files. Para mejorar la seguridad, ambos tipos de almacenamiento usan Azure Storage Service Encryption (SSE) de manera predeterminada para cifrar los datos en reposo. Actualmente no se pueden cifrar los discos con Azure Disk Encryption en el nivel de nodo de AKS.

Tanto Azure Files como Azure Disks están disponibles en los niveles de rendimiento Estándar y Premium:

- Los discos *premium* están respaldados por discos de estado sólido (SSD) de alto rendimiento. Los discos premium son aconsejables para cargas de trabajo de producción.
- Los discos *estándar* están respaldados por discos giratorios normales (HDD) y son adecuados para archivar o guardar datos a los que se accede con poca frecuencia.

Comprender las necesidades de rendimiento de la aplicación y los patrones de acceso para elegir el nivel de almacenamiento adecuado. Para más información sobre los tamaños y niveles de rendimiento de Managed Disks, consulte [Introducción a los discos administrados de Azure][managed-disks]

### <a name="create-and-use-storage-classes-to-define-application-needs"></a>Crear y utilizar clases de almacenamiento para definir las necesidades de la aplicación

El tipo de almacenamiento que usa se define con las *clases de almacenamiento* de Kubernetes. A continuación, se hacer referencia a la clase de almacenamiento en la especificación de almacenamiento o en el pod. Estas definiciones funcionan conjuntamente para crear el almacenamiento adecuado y conectarlo a los pods. Para más información, consulte [Clases de almacenamiento en AKS][aks-concepts-storage-classes].

## <a name="size-the-nodes-for-storage-needs"></a>Cambiar el tamaño de los nodos según las necesidades de almacenamiento

**Orientación con procedimientos recomendados**: cada tamaño de nodo es compatible con un número máximo de discos. Los diferentes tamaños de nodos también proporcionan distintas cantidades de almacenamiento local y de ancho de banda de red. Planee las demandas dde las aplicaciones para implementar el tamaño adecuado de los nodos.

Los nodos de AKS se ejecutan como VM de Azure. Existen distintos tipos y tamaños de VM disponibles. Cada tamaño de VM proporciona una cantidad diferente de recursos principales, como CPU y memoria. Estos tamaños de VM tienen un número máximo de discos que se pueden conectar. El rendimiento del almacenamiento también varía en función de los tamaños de VM para el IOPS (operaciones de entrada/salida por segundo) máximo de discos conectados y locales.

Si las aplicaciones requieren la tecnología Azure Disks como su solución de almacenamiento, planee y elija un tamaño de VM de nodo apropiado. La cantidad de CPU y memoria no es el único factor a la hora de elegir un tamaño de VM. Las capacidades de almacenamiento también son importantes. Por ejemplo, los tamaños de VM *Standard_B2ms* y *Standard_DS2_v2* incluyen una cantidad similar de recursos de CPU y memoria. Su rendimiento de almacenamiento potencial es diferente, como se muestra en la tabla siguiente:

| Tamaño y tipo de nodo | vCPU | Memoria (GiB) | Discos de datos máx. | IOPS de disco máximo sin almacenamiento en caché | Rendimiento máximo sin almacenamiento en caché (MBps) |
|--------------------|------|--------------|----------------|------------------------|--------------------------------|
| Standard_B2ms      | 2    | 8            | 4              | 1\.920                  | 22.5                           |
| Standard_DS2_v2    | 2    | 7            | 8              | 6\.400                  | 96                             |

En este caso, *Standard_DS2_v2* permite duplicar el número de discos conectados y proporciona entre tres o cuatro veces la cantidad de IOPS y de rendimiento de disco. Si solo examinó los recursos de proceso principales y comparó los costos, es posible que elija el tamaño de VM *Standard_B2ms* y sufra limitaciones y un bajo rendimiento de almacenamiento. Trabaje con su equipo de desarrollo de aplicaciones para comprender sus necesidades de rendimiento y de capacidad de almacenamiento. Elija el tamaño de VM adecuado para los nodos de AKS para cumplir o superar las necesidades de rendimiento. Establezca regularmente las aplicaciones para ajustar el tamaño de VM según sea necesario.

Para más información sobre los tamaños de máquina virtual disponibles, consulte [Tamaños de las máquinas virtuales Linux en Azure][vm-sizes].

## <a name="dynamically-provision-volumes"></a>Aprovisionar volúmenes dinámicamente

**Orientación con procedimientos recomendados**: para reducir la sobrecarga de administración y escalar, no cree ni asigne estáticamente volúmenes persistentes. Use el aprovisionamiento dinámico. En las clases de almacenamiento, defina la directiva de reclamación adecuada para reducir los costos de almacenamiento innecesarios una vez que se eliminan los pods.

Cuando necesite conectar almacenamiento a los pods, use los volúmenes persistentes. Estos volúmenes persistentes se pueden crear manual o dinámicamente. La creación manual de volúmenes persistentes agrega sobrecarga de administración y limita su capacidad para escalar. Use el aprovisionamiento de volumen persistente para simplificar la administración de almacenamiento y permitir que las aplicaciones crezcan y se escalen según sea necesario.

![Notificaciones de volúmenes persistentes en un clúster de Azure Kubernetes Service (AKS)](media/concepts-storage/persistent-volume-claims.png)

Una notificación de volumen persistente (PVC) le permite crear almacenamiento dinámicamente según sea necesario. Los discos subyacentes de Azure se crean a medida que se solicitan los pods. En la definición del pod, se solicita un volumen para que se cree y se conecte a una ruta de acceso de montaje designada.

Para los conceptos sobre cómo crear y usar volúmenes dinámicamente, consulte [Notificaciones de volúmenes persistentes][aks-concepts-storage-pvcs].

Para ver los volúmenes en acción, consulte cómo crear y usar dinámicamente un volumen persistente con [Azure Disks][dynamic-disks] o [Azure Files][dynamic-files].

Como parte de las definiciones de clase de almacenamiento, configure la directiva *reclaimPolicy* apropiada. Esta directiva reclaimPolicy controla el comportamiento del recurso de almacenamiento subyacente de Azure cuando se elimina el pod y el volumen persistente puede dejar de ser necesario. El recurso de almacenamiento subyacente se puede eliminar o conservar para su uso con un pod futuro. Puede establecer la directiva reclaimPolicy en *conservar* o *eliminar*. Comprenda las necesidades de su aplicación e implemente comprobaciones regulares para saber el almacenamiento que se conserva para minimizar la cantidad de espacio de almacenamiento no utilizado que se usa y se factura.

Para más información sobre las opciones de clase de almacenamiento, consulte las [directivas de reclamación de almacenamiento][reclaim-policy].

## <a name="secure-and-back-up-your-data"></a>Proteja y realice una copia de seguridad de los datos

**Orientación sobre los procedimientos recomendados**: realice una copia de seguridad de los datos con una herramienta apropiada para su tipo de almacenamiento, como Velero o Azure Backup. Compruebe la integridad y seguridad de las copias de seguridad.

Cuando las aplicaciones almacenan y consumen datos que persisten en los discos o en los archivos, es necesario realizar copias de seguridad o instantáneas periódicamente de esos datos. La tecnología Azure Disks puede usar la tecnología integrada de instantánea. Es posible que necesite establecer que las aplicaciones vacíen las escrituras en el disco antes de realizar la operación de instantánea. [Velero][velero] puede realizar copias de seguridad de volúmenes persistentes junto con recursos y configuraciones de clústeres adicionales. Si no puede [eliminar el estado de sus aplicaciones][remove-state], realice una copia de seguridad de los datos de los volúmenes persistentes y pruebe regularmente las operaciones de restauración para verificar la integridad de los datos y los procesos necesarios.

Comprenda las limitaciones de los diferentes enfoques de las copias de seguridad de datos y si necesita dejar sus datos en reposo antes de la instantánea. Las copias de seguridad de datos no necesariamente le permiten restaurar el entorno de la aplicación de la implementación del clúster. Para más información acerca de estos escenarios, consulte [Procedimientos recomendados para continuidad empresarial y recuperación ante desastres en AKS][best-practices-multi-region].

## <a name="next-steps"></a>Pasos siguientes

Este artículo se centra en el procedimiento recomendado de almacenamiento en AKS. Para más información sobre los conceptos básicos de almacenamiento en Kubernetes, consulte [Opciones de almacenamiento de aplicaciones en Azure Kubernetes Service (AKS)][aks-concepts-storage].

<!-- LINKS - External -->
[velero]: https://github.com/heptio/velero
[blobfuse]: https://github.com/Azure/azure-storage-fuse

<!-- LINKS - Internal -->
[aks-concepts-storage]: concepts-storage.md
[vm-sizes]: ../virtual-machines/sizes.md
[dynamic-disks]: azure-disks-dynamic-pv.md
[dynamic-files]: azure-files-dynamic-pv.md
[reclaim-policy]: concepts-storage.md#storage-classes
[aks-concepts-storage-pvcs]: concepts-storage.md#persistent-volume-claims
[aks-concepts-storage-classes]: concepts-storage.md#storage-classes
[managed-disks]: ../virtual-machines/managed-disks-overview.md
[best-practices-multi-region]: operator-best-practices-multi-region.md
[remove-state]: operator-best-practices-multi-region.md#remove-service-state-from-inside-containers
