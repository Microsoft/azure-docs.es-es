---
title: Configuración de almacenamiento
description: Explica las opciones de configuración del almacenamiento de los servicios de datos habilitados para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: uc-msft
ms.author: umajay
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: conceptual
ms.openlocfilehash: 782a046b92c9d6cf755bfea0551d7f8153faa859
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932243"
---
# <a name="storage-configuration"></a>Configuración de almacenamiento

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="kubernetes-storage-concepts"></a>Conceptos de almacenamiento de Kubernetes

Kubernetes proporciona una capa de abstracción de la infraestructura sobre la pila de tecnología de virtualización subyacente (opcional) y el hardware. La forma en la que Kubernetes abstrae el almacenamiento es mediante **[clases de almacenamiento](https://kubernetes.io/docs/concepts/storage/storage-classes/)** . En el momento de aprovisionar un pod, se puede especificar la clase de almacenamiento que se usará para cada volumen. En el momento en el que se aprovisiona el pod, se llama al **[aprovisionador](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)** de la clase de almacenamiento para aprovisionar el almacenamiento y, a continuación, se crea un **[volumen persistente](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)** en el almacenamiento aprovisionado y, a continuación, el pod se monta en el volumen persistente mediante una **[notificación de volumen persistente](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)** .

Kubernetes proporciona una manera para que los proveedores de infraestructura de almacenamiento conecten controladores (también llamados "complementos") que amplían Kubernetes. Los complementos de almacenamiento deben ajustarse al **[estándar de interfaz de almacenamiento de contenedores](https://kubernetes.io/blog/2019/01/15/container-storage-interface-ga/)** . Hay docenas de complementos que se pueden encontrar en esta **[lista de controladores CSI](https://kubernetes-csi.github.io/docs/drivers.html)** no definitiva. El controlador CSI que use dependerá de factores como, por ejemplo, si la ejecución se realiza en un servicio de Kubernetes administrado hospedado en la nube o el proveedor de OEM que usa para el hardware.

Puede ver qué clases de almacenamiento hay configuradas en el clúster de Kubernetes mediante la ejecución de este comando:

``` terminal
kubectl get storageclass
```

Ejemplo de salida de un clúster de Azure Kubernetes Service (AKS):

``` terminal
NAME                PROVISIONER                AGE
azurefile           kubernetes.io/azure-file   15d
azurefile-premium   kubernetes.io/azure-file   15d
default (default)   kubernetes.io/azure-disk   4d3h
managed-premium     kubernetes.io/azure-disk   4d3h
```

Puede obtener detalles acerca de una clase de almacenamiento mediante la ejecución de este comando:

``` terminal
kubectl describe storageclass\<storage class name>
```

Ejemplo:

``` terminal
kubectl describe storageclass/azurefile

Name:            azurefile
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"allowVolumeExpansion":true,"apiVersion":"storage.k8s.io/v1beta1","kind":"StorageClass","metadata":{"annotations":{},"labels":{"kubernetes.io/cluster-service":"true"},"name":"azurefile"},"parameters":{"sku
Name":"Standard_LRS"},"provisioner":"kubernetes.io/azure-file"}

Provisioner:           kubernetes.io/azure-file
Parameters:            skuName=Standard_LRS
AllowVolumeExpansion:  True
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```

Puede ver los volúmenes persistentes aprovisionados actualmente y las notificaciones de volumen persistente mediante la ejecución de los siguientes comandos:

``` terminal
kubectl get persistentvolumes -n <namespace>

kubectl get persistentvolumeclaims -n <namespace>
```

Ejemplo de cómo mostrar los volúmenes persistentes:

``` terminal

kubectl get persistentvolumes -n arc

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                      STORAGECLASS   REASON   AGE
pvc-07fc7b9f-9a37-4796-9442-4405147120da   15Gi       RWO            Delete           Bound    arc/sqldemo11-data-claim   default                 7d3h
pvc-3e772f20-ed89-4642-b34d-8bb11b088afa   15Gi       RWO            Delete           Bound    arc/data-metricsdb-0       default                 7d14h
pvc-41b33bbd-debb-4153-9a41-02ce2bf9c665   10Gi       RWO            Delete           Bound    arc/sqldemo11-logs-claim   default                 7d3h
pvc-4ccda3e4-fee3-4a89-b92d-655c04fa62ad   15Gi       RWO            Delete           Bound    arc/data-controller        default                 7d14h
pvc-63e6bb4c-7240-4de5-877e-7e9ea4e49c91   10Gi       RWO            Delete           Bound    arc/logs-controller        default                 7d14h
pvc-8a1467fe-5eeb-4d73-b99a-f5baf41eb493   10Gi       RWO            Delete           Bound    arc/logs-metricsdb-0       default                 7d14h
pvc-8e2cacbc-e953-4901-8591-e77df9af309c   10Gi       RWO            Delete           Bound    arc/sqldemo10-logs-claim   default                 7d14h
pvc-9fb79ba3-bd3e-42aa-aa09-3090135d4513   15Gi       RWO            Delete           Bound    arc/sqldemo10-data-claim   default                 7d14h
pvc-a39c85d4-5cd9-4249-9915-68a70a9bb5e5   15Gi       RWO            Delete           Bound    arc/data-controldb         default                 7d14h
pvc-c9cbd74a-76ca-4be5-b598-0c7a45749bfb   10Gi       RWO            Delete           Bound    arc/logs-controldb         default                 7d14h
pvc-d576e9d4-0a09-4dd7-b806-be8ed461f8a4   10Gi       RWO            Delete           Bound    arc/logs-logsdb-0          default                 7d14h
pvc-ecd7d07f-2c2c-421d-98d7-711ec5d4a0cd   15Gi       RWO            Delete           Bound    arc/data-logsdb-0          default                 7d14h
```

Ejemplo de cómo mostrar las notificaciones de volumen persistente:

``` terminal

kubectl get persistentvolumeclaims -n arc

NAME                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-controldb         Bound    pvc-a39c85d4-5cd9-4249-9915-68a70a9bb5e5   15Gi       RWO            default        7d14h
data-controller        Bound    pvc-4ccda3e4-fee3-4a89-b92d-655c04fa62ad   15Gi       RWO            default        7d14h
data-logsdb-0          Bound    pvc-ecd7d07f-2c2c-421d-98d7-711ec5d4a0cd   15Gi       RWO            default        7d14h
data-metricsdb-0       Bound    pvc-3e772f20-ed89-4642-b34d-8bb11b088afa   15Gi       RWO            default        7d14h
logs-controldb         Bound    pvc-c9cbd74a-76ca-4be5-b598-0c7a45749bfb   10Gi       RWO            default        7d14h
logs-controller        Bound    pvc-63e6bb4c-7240-4de5-877e-7e9ea4e49c91   10Gi       RWO            default        7d14h
logs-logsdb-0          Bound    pvc-d576e9d4-0a09-4dd7-b806-be8ed461f8a4   10Gi       RWO            default        7d14h
logs-metricsdb-0       Bound    pvc-8a1467fe-5eeb-4d73-b99a-f5baf41eb493   10Gi       RWO            default        7d14h
sqldemo10-data-claim   Bound    pvc-9fb79ba3-bd3e-42aa-aa09-3090135d4513   15Gi       RWO            default        7d14h
sqldemo10-logs-claim   Bound    pvc-8e2cacbc-e953-4901-8591-e77df9af309c   10Gi       RWO            default        7d14h
sqldemo11-data-claim   Bound    pvc-07fc7b9f-9a37-4796-9442-4405147120da   15Gi       RWO            default        7d4h
sqldemo11-logs-claim   Bound    pvc-41b33bbd-debb-4153-9a41-02ce2bf9c665   10Gi       RWO            default        7d4h

```

## <a name="factors-to-consider-when-choosing-your-storage-configuration"></a>Factores a tener en cuenta al elegir la configuración de almacenamiento

La selección de la clase de almacenamiento correcta es muy importante para el rendimiento y la resistencia de los datos. La elección de una clase de almacenamiento incorrecta puede poner los datos en riesgo de una pérdida total en caso de que se produzca un error de hardware o puede dar lugar a un rendimiento menos óptimo.

Por lo general, hay dos tipos de almacenamiento:

- **Almacenamiento local**: almacenamiento que se aprovisiona en las unidades de disco duro local de un nodo determinado. Este tipo de almacenamiento puede ser idóneo en términos de rendimiento, pero requiere un diseño específico para la redundancia de datos mediante la replicación de los datos en varios nodos.
- **Almacenamiento remoto compartido**: almacenamiento que se aprovisiona en algún dispositivo de almacenamiento remoto (por ejemplo, un dispositivo SAN, NAS o un servicio de almacenamiento en la nube como EBS o Azure Files). Normalmente, este tipo de almacenamiento proporciona redundancia de datos automáticamente, pero generalmente no es tan rápido como puede ser el almacenamiento local.

> [!NOTE]
> Por ahora, si usa NFS, debe establecer allowRunAsRoot en true en el archivo del perfil de implementación antes de implementar el controlador de datos de Azure Arc.

### <a name="data-controller-storage-configuration"></a>Configuración de almacenamiento del controlador de datos

Algunos servicios de Azure Arc para servicios de datos dependen de que se configuren para usar almacenamiento compartido remoto, porque los servicios no tienen una funcionalidad para replicar los datos. Estos servicios se encuentran en la colección de pods del controlador de datos:

|**Servicio**|**Notificaciones de volumen persistente**|
|---|---|
|**ElasticSearch**|`<namespace>/logs-logsdb-0`, `<namespace>/data-logsdb-0`|
|**InfluxDB**|`<namespace>/logs-metricsdb-0`, `<namespace>/data-metricsdb-0`|
|**Instancia de SQL del controlador**|`<namespace>/logs-controldb`, `<namespace>/data-controldb`|
|**Servicio de API del controlador**|`<namespace>/data-controller`|

En el momento en que se aprovisiona el controlador de datos, se especifica la clase de almacenamiento que se va a usar para cada uno de estos volúmenes persistentes mediante el uso del parámetro --storage-class (-sc) en el comando `azdata arc dc create` o mediante el establecimiento de las clases de almacenamiento en el archivo de plantilla de implementación control.json que se usa.

Las plantillas de implementación que se proporcionan de forma predeterminada tienen una clase de almacenamiento predeterminada especificada que es adecuada para el entorno de destino, pero se puede invalidar en el momento de la implementación. Consulte los pasos detallados para [modificar el perfil de implementación](create-data-controller.md) para cambiar la configuración de la clase de almacenamiento para los pods del controlador de datos en el momento de la implementación.

Si establece la clase de almacenamiento mediante el parámetro --storage-class (-sc), la clase de almacenamiento se usará para las clases de almacenamiento de datos y de registro. Si establece las clases de almacenamiento en el archivo de plantilla de implementación, puede especificar clases de almacenamiento diferentes para los registros y los datos.

Factores importantes que se deben tener en cuenta al elegir una clase de almacenamiento para los pods del controlador de datos:

- **Debe** usar una clase de almacenamiento compartida remota para asegurar la durabilidad de los datos y, de este modo, si un pod o un nodo dejan de estar en funcionamiento, pueden conectarse de nuevo al volumen persistente cuando se recuperen.
- Los datos que se escriben en la instancia de SQL del controlador, la base de datos de métricas y la base de datos de registros suelen ser de un volumen bajo y no son sensibles a la latencia, por lo que un almacenamiento de rendimiento extremadamente rápido no es fundamental. Si tiene usuarios que usan con frecuencia las interfaces de Grafana y Kibana y tiene un gran número de instancias de base de datos, es posible que los usuarios se beneficien de un almacenamiento con un rendimiento más rápido.
- La capacidad de almacenamiento necesaria varía con el número de instancias de base de datos que ha implementado, porque se recopilan registros y métricas para cada instancia de base de datos. Los datos se conservan en las bases de datos de registros y métricas durante 2 semanas antes de que se purguen. TODO: ¿Cuánto almacenamiento se necesita por cada instancia de base de datos?
- Cambiar la clase de almacenamiento después de la implementación es muy difícil, no está documentado y no se admite. Asegúrese de elegir la clase de almacenamiento correcta en el momento de la implementación.

> **Nota:** Si no se especifica ninguna clase de almacenamiento, se usará la clase de almacenamiento predeterminada. Solo puede haber una clase de almacenamiento predeterminada por cada clúster de Kubernetes. Puede [cambiar la clase de almacenamiento predeterminada](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/).

### <a name="database-instance-storage-configuration"></a>Configuración del almacenamiento de la instancia de base de datos

Cada instancia de base de datos tiene volúmenes persistentes de datos, registros y copia de seguridad. Las clases de almacenamiento de estos volúmenes persistentes se pueden especificar en el momento de la implementación. Si no se especifica ninguna clase de almacenamiento, se usará la clase de almacenamiento predeterminada.

Al crear una instancia mediante los comandos `azdata arc sql mi create` o `azdata arc postgres server create`, hay dos parámetros que se pueden usar para establecer las clases de almacenamiento:

> **Nota:** Algunos de estos parámetros están en desarrollo y estarán disponibles en `azdata arc sql mi create` y `azdata arc postgres server create` en las próximas versiones.

|Nombre del parámetro, nombre abreviado|Se usa para|
|---|---|
|`--storage-class-data`, `-scd`|Se utiliza para especificar la clase de almacenamiento de todos los archivos de datos, incluidos los archivos del registro de transacciones.|
|`--storage-class-logs`, `-scl`|Se usa para especificar la clase de almacenamiento de todos los archivos de registro.|
|`--storage-class-data-logs`, `-scdl`|Se utiliza para especificar la clase de almacenamiento de los archivos del registro de transacciones de la base de datos. **Nota: No disponible todavía.**|
|`--storage-class-backups`, `-scb`|Se utiliza para especificar la clase de almacenamiento de todos los archivos de copia de seguridad. **Nota: No disponible todavía.**|

En la tabla siguiente se enumeran las rutas de acceso dentro del contenedor de Azure SQL Managed Instance que está asignado al volumen persistente para datos y registros:

|Nombre del parámetro, nombre abreviado|Ruta de acceso en el contenedor mssql-miaa|Descripción|
|---|---|---|
|`--storage-class-data`, `-scd`|/var/opt|Contiene directorios para la instalación de mssql y otros procesos del sistema. El directorio mssql contiene los directorios predeterminados para los datos (incluidos los registros de transacciones), el registro de errores y la copia de seguridad.|
|`--storage-class-logs`, `-scl`|/var/log|Contiene los directorios que almacenan la salida de la consola (stderr, stdout) y otra información de registro de los procesos dentro del contenedor.|

En la tabla siguiente se enumeran las rutas de acceso dentro del contenedor de la instancia de PostgreSQL que está asignado al volumen persistente para datos y registros:

|Nombre del parámetro, nombre abreviado|Ruta de acceso en el contenedor de Postgres|Descripción|
|---|---|---|
|`--storage-class-data`, `-scd`|/var/opt/postgresql|Contiene los directorios de datos y registro para la instalación de Postgres.|
|`--storage-class-logs`, `-scl`|/var/log|Contiene los directorios que almacenan la salida de la consola (stderr, stdout) y otra información de registro de los procesos dentro del contenedor.|

Cada instancia de base de datos tendrá un volumen persistente independiente para los archivos de datos, registros y copias de seguridad. Esto significa que se separará la E/S de cada uno de estos tipos de archivos, en función del modo en que el aprovisionador del volumen aprovisione el almacenamiento. Cada instancia de base de datos tiene sus propias notificaciones de volumen persistente y sus volúmenes persistentes.

Si hay varias bases de datos en una instancia de base de datos determinada, todas las bases de datos usarán la misma notificación de volumen persistente, volumen persistente y clase de almacenamiento. Todas las copias de seguridad, tanto las copias de seguridad diferenciales del registro como las copias de seguridad completas, usarán la misma notificación de volumen persistente y el mismo volumen persistente. A continuación se muestran las notificaciones de volumen persistente para los pods de la instancia de base de datos:

|**Instancia**|**Notificaciones de volumen persistente**|
|---|---|
|**Instancia administrada de Azure SQL**|`<namespace>/logs-<instance name>-0`, `<namespace>/data-<instance name>-0`|
|**Instancia de Azure Database for PostgreSQL**|`<namespace>/logs--<instance name>-0`, `<namespace>/data--<instance name>-0`|
|**Hiperescala de PostgreSQL de Azure**|`<namespace>/logs-<instance namme>-<ordinal>`, `<namespace>/data-<instance namme>-<ordinal>` *(intervalos ordinales de 0 a W, donde W es el número de nodos de trabajo)*|

Factores importantes que se deben tener en cuenta al elegir una clase de almacenamiento para los pods de la instancia de base de datos:

- Las instancias de base de datos se pueden implementar con un patrón de un único pod o con un patrón de varios pods. Una instancia de desarrollo de Azure SQL Managed Instance o un plan de tarifa de uso general de Azure SQL Managed Instance son ejemplos de patrón de un único pod. Un ejemplo de patrón de varios pods sería un plan de tarifa crítico para la empresa con alta disponibilidad de Azure SQL Managed Instance. (Nota: Los planes de tarifa están en desarrollo y no están aún disponibles para los clientes).  Las instancias de base de datos implementadas con el patrón de un único pod **deben** usar una clase de almacenamiento compartida remota para asegurar la durabilidad de los datos y, de este modo, si un pod o un nodo dejan de estar en funcionamiento, pueden conectarse de nuevo al volumen persistente cuando se recuperen. En cambio, una instancia de Azure SQL Managed Instance con alta disponibilidad usa Grupos de disponibilidad Always On para replicar los datos de una instancia a otra de forma sincrónica o asincrónica. Especialmente cuando los datos se replican de forma sincrónica, siempre hay varias copias de los datos; normalmente, 3 copias. Por este motivo, es posible usar clases de almacenamiento local o de almacenamiento remoto compartido para los archivos de datos y de registro. Si se usa el almacenamiento local, los datos se conservan incluso en caso de que se produzca un error en el pod, el nodo o el hardware de almacenamiento. Dada esta flexibilidad, puede optar por usar el almacenamiento local para mejorar el rendimiento.
- En gran medida, el rendimiento de la base de datos está en función del rendimiento de E/S de un dispositivo de almacenamiento determinado. Si la base de datos tiene lecturas pesadas o escrituras pesadas, debe elegir una clase de almacenamiento que tenga un hardware subyacente que esté diseñado para ese tipo de carga de trabajo. Por ejemplo, si la base de datos se usa principalmente para escrituras, puede elegir un almacenamiento local con RAID 0. Si la base de datos se usa principalmente para lecturas de una pequeña cantidad de "datos activos", pero hay un gran volumen de almacenamiento total de datos inactivos, puede elegir un dispositivo SAN con funcionalidad de almacenamiento en capas. La elección de la clase de almacenamiento adecuada no es realmente muy diferente a elegir el tipo de almacenamiento que se usaría para cualquier base de datos.
- Si usa un aprovisionador para el volumen de almacenamiento local, debe asegurarse de que los volúmenes locales aprovisionados para los datos, los registros y las copias de seguridad residen en dispositivos de almacenamiento subyacentes diferentes para evitar la contención de E/S de disco. El sistema operativo también debe estar en un volumen montado en un disco independiente. Esta es en esencia la misma guía que se seguiría para una instancia de base de datos en hardware físico.
- Dado que todas las bases de datos de una instancia determinada comparten una notificación de volumen persistente y un volumen persistente, asegúrese de no coubicar las instancias de base de datos ocupadas en la misma instancia de base de datos. Si es posible, separe las bases de datos ocupadas en sus propias instancias de base de datos para evitar la contención de E/S. Además, use las etiquetas de nodo para que las instancias de base de datos tengan como destino nodos independientes para distribuir el tráfico de E/S global entre varios nodos. Si usa virtualización, asegúrese de considerar la posibilidad de distribuir el tráfico de E/S no solo en el nivel de nodo, sino también la actividad combinada de E/S que se produce en todas las máquinas virtuales de nodo de un host físico determinado.

## <a name="estimating-storage-requirements"></a>Estimación de los requisitos de almacenamiento
Cada pod que contiene datos con estado usa dos volúmenes persistentes en esta versión: un volumen persistente para los datos y otro volumen persistente para los registros. En la tabla siguiente se muestra el número de volúmenes persistentes necesarios para un solo controlador de datos, una instancia de Azure SQL Managed Instance, una instancia de Azure Database for PostgreSQL y una instancia de Hiperescala de PostgreSQL de Azure:

|Tipo de recurso|Número de pods con estado|Número requerido de volúmenes persistentes|
|---|---|---|
|Controlador de datos|4 (`control`, `controldb`, `logsdb` y `metricsdb`)|4 * 2 = 8|
|Azure SQL Managed Instance|1|2|
|Instancia de Azure Database for PostgreSQL|1| 2|
|Hiperescala de PostgreSQL de Azure|1 + W (W = número de nodos de trabajo)|2 * (1 + W)|

En la tabla siguiente se muestra el número total de volúmenes persistentes necesarios para una implementación de ejemplo:

|Tipo de recurso|Número de instancias|Número requerido de volúmenes persistentes|
|---|---|---|
|Controlador de datos|1|4 * 2 = 8|
|Azure SQL Managed Instance|5|5 * 2 = 10|
|Instancia de Azure Database for PostgreSQL|5| 5 * 2 = 10|
|Hiperescala de PostgreSQL de Azure|2 (número de nodos de trabajo = 4 por instancia)|2 * 2 * (1 + 4) = 20|
|***Número total de volúmenes persistentes***||8 + 10 + 10 + 20 = 48|

Este cálculo se puede usar para planear el almacenamiento del clúster de Kubernetes en función del aprovisionador de almacenamiento o el entorno. Por ejemplo, si se usa el aprovisionador de almacenamiento local para un clúster de Kubernetes con 5 nodos, para la implementación de ejemplo anterior cada nodo requiere al menos almacenamiento para 10 volúmenes persistentes. Del mismo modo, al aprovisionar un clúster de Azure Kubernetes Service (AKS) con 5 nodos, es fundamental elegir un tamaño de máquina virtual adecuado para el grupo de nodos para que se puedan conectar 10 discos de datos. Puede encontrar más información sobre cómo ajustar el tamaño de los nodos para las necesidades de almacenamiento de los nodos de AKS [aquí](../../aks/operator-best-practices-storage.md#size-the-nodes-for-storage-needs).

## <a name="choosing-the-right-storage-class"></a>Elección de la clase de almacenamiento adecuada

### <a name="on-premises-and-edge-sites"></a>Sitios locales y perimetrales

Microsoft y sus asociados de OEM, SO y Kubernetes trabajan en un programa de certificación para servicios de datos de Azure Arc. Este programa proporcionará a los clientes resultados de pruebas comparables a partir de un kit de herramientas de pruebas de certificación. Las pruebas evaluarán la compatibilidad de las características, los resultados de las pruebas de esfuerzo y el rendimiento y la escalabilidad. Cada uno de estos resultados de las pruebas indicará el sistema operativo usado, la distribución de Kubernetes usada, el hardware usado, el complemento CSI usado y las clases de almacenamiento usadas. Esto ayudará a los clientes a elegir la clase de almacenamiento, el sistema operativo, la distribución de Kubernetes y el hardware más adecuados para sus requisitos. Se proporcionará más información sobre este programa y los resultados de las pruebas iniciales cerca de la disponibilidad general de los servicios de datos de Azure Arc.

#### <a name="public-cloud-managed-kubernetes-services"></a>Servicios de Kubernetes administrado en la nube pública

En el caso de los servicios de Kubernetes administrado basados en la nube pública, podemos hacer las siguientes recomendaciones:

|Servicio en la nube pública|Recomendación|
|---|---|
|**Azure Kubernetes Service (AKS)**|Azure Kubernetes Service (AKS) tiene dos tipos de almacenamiento: Azure Files y Azure Disks. Cada tipo de almacenamiento tiene dos niveles de precios y rendimiento: Estándar (HDD) y Premium (SSD). Por lo tanto, las cuatro clases de almacenamiento proporcionadas en AKS son `azurefile` (nivel Estándar de Azure Files), `azurefile-premium` (nivel Premium de Azure Files), `default` (nivel Estándar de Azure Disks) y `managed-premium` (nivel Premium de Azure Disks). La clase de almacenamiento predeterminada es `default` (nivel Estándar de Azure Disks). Hay **[diferencias de precios](https://azure.microsoft.com/en-us/pricing/details/storage/)** importantes entre los tipos y los niveles que se deben tener en cuenta en la decisión. En el caso de cargas de trabajo de producción con requisitos de alto rendimiento, se recomienda usar `managed-premium` para todas las clases de almacenamiento. En el caso de cargas de trabajo de desarrollo y pruebas, pruebas de concepto, etc., donde el costo es una consideración, `azurefile` es la opción menos costosa. Las cuatro opciones se pueden usar en situaciones que requieran almacenamiento remoto compartido, ya que todas utilizan dispositivos de almacenamiento conectados a la red de Azure. Más información sobre [Almacenamiento en AKS](../../aks/concepts-storage.md).|
|**AWS Elastic Kubernetes Service (EKS)**| Elastic Kubernetes Service de Amazon tiene una clase de almacenamiento principal basada en el [controlador de almacenamiento CSI EBS](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html). Se recomienda para las cargas de trabajo de producción. Hay un nuevo controlador de almacenamiento, el [controlador de almacenamiento CSI EFS](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html), que se puede agregar a un clúster de EKS, pero actualmente se encuentra en una fase beta y está sujeto a cambios. Aunque AWS indica que este controlador de almacenamiento se admite para producción, no se recomienda usarlo porque todavía está en versión beta y está sujeto a cambios. La clase de almacenamiento EBS es el valor predeterminado y se llama `gp2`. Más información sobre [Almacenamiento en EKS](https://docs.aws.amazon.com/eks/latest/userguide/storage-classes.html).|
|**Google Kubernetes Engine (GKE)**|Google Kubernetes Engine (GKE) tiene solo una clase de almacenamiento llamada `standard` que se usa para [discos persistentes GCE](https://kubernetes.io/docs/concepts/storage/volumes/#gcepersistentdisk). Al ser la única, también es la predeterminada. Aunque hay un [aprovisionador de volúmenes estáticos locales](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/local-ssd#run-local-volume-static-provisioner) para GKE que puede usar con discos SSD con conexión directa, no es recomendable usarlo, ya que Google no lo mantiene ni presta servicio técnico. Más información sobre [Almacenamiento en GKE](https://cloud.google.com/kubernetes-engine/docs/concepts/persistent-volumes).
