---
title: Escalabilidad de los tamaños de clúster en Azure HDInsight
description: Escale un clúster de HDInsight de Azure con total flexibilidad para que coincida con la carga de trabajo.
author: ashishthaps
ms.author: ashish
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/13/2019
ms.openlocfilehash: 59b9c2bf6e17dadc0d084d3e3f257f8ad91073ca
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595854"
---
# <a name="scale-hdinsight-clusters"></a>Escalabilidad de clústeres de HDInsight

HDInsight proporciona elasticidad, lo que ofrece la opción de escalar y reducir verticalmente el número de nodos de trabajo de los clústeres. Esto permite reducir un clúster después de horas o los fines de semana y expandirlo durante períodos de máxima demanda empresarial.

Por ejemplo, si tiene algún procesamiento por lotes produce una vez al día o una vez al mes, el clúster de HDInsight se puede escalar verticalmente unos minutos antes del evento programado por lo que habrá suficiente memoria y potencia de proceso de CPU.  Después, una vez realizado el procesamiento y de que el uso vuelva a bajar, puede reducir verticalmente el clúster de HDInsight a menos nodos de trabajo.

Puede escalar un clúster manualmente mediante uno de los métodos descritos a continuación o usar [escalado automático](hdinsight-autoscale-clusters.md) opciones para que el sistema automáticamente escalar vertical y horizontalmente en respuesta a la CPU, memoria y otras métricas.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="utilities-to-scale-clusters"></a>Utilidades para escalar clústeres

Microsoft proporciona las utilidades siguientes para escalar clústeres:

|Utilidad | DESCRIPCIÓN|
|---|---|
|[PowerShell Az](https://docs.microsoft.com/powershell/azure)|[Conjunto AzHDInsightClusterSize](https://docs.microsoft.com/powershell/module/az.hdinsight/set-azhdinsightclustersize) - ClusterName \<nombre del clúster > - TargetInstanceCount \<NewSize >|
|[PowerShell AzureRM](https://docs.microsoft.com/powershell/azure/azurerm) |[Conjunto AzureRmHDInsightClusterSize](https://docs.microsoft.com/powershell/module/azurerm.hdinsight/set-azurermhdinsightclustersize) - ClusterName \<nombre del clúster > - TargetInstanceCount \<NewSize >|
|[CLI de Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest)| [cambiar el tamaño de AZ hdinsight](https://docs.microsoft.com/cli/azure/hdinsight?view=azure-cli-latest#az-hdinsight-resize) : grupo de recursos \<grupo de recursos >--nombre \<nombre del clúster >--recuento de instancias de destino \<NewSize >|
|[CLI de Azure clásica](hdinsight-administer-use-command-line.md)|cambio de tamaño de clúster de hdinsight de Azure \<clusterName > \<recuento de instancias de destino > |
|[Azure Portal](https://portal.azure.com)|Abra el panel del clúster de HDInsight, seleccione **tamaño del clúster** en el menú izquierdo, en el panel de tamaño del clúster, escriba el número de nodos de trabajo y seleccione Guardar.|  

![Escalar clúster](./media/hdinsight-scaling-best-practices/scale-cluster-blade.png)

Con cualquiera de estos métodos, puede escalar o reducir verticalmente el clúster de HDInsight en cuestión de minutos.

> [!IMPORTANT]  
> * La CLI de Azure clásica está en desuso y solo debe usarse con el modelo de implementación clásica. Para todas las demás implementaciones, utilice el [CLI de Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest).  
> * El módulo AzureRM de PowerShell está en desuso.  Use la [módulo Az](https://docs.microsoft.com/powershell/azure/new-azureps-module-az?view=azps-1.4.0) siempre que sea posible.

## <a name="scaling-impacts-on-running-jobs"></a>Efectos del escalado en los trabajos en ejecución

Cuando se **agregan** nodos al clúster de HDInsight en ejecución, esto no afecta a los trabajos pendientes o en ejecución. Además, los trabajos nuevos pueden enviarse de forma segura mientras el proceso de escalado está en ejecución. Si se produce un error en las operaciones de escalado por cualquier motivo, el error se controla correctamente y el clúster sigue funcionando.

Sin embargo, si reduce verticalmente el clúster mediante la **eliminación** de nodos, se producirá un error en los trabajos pendientes o en ejecución cuando finalice la operación de escalado. Este error se debe a que algunos de los servicios se reinician durante el proceso.

Para solucionar este problema, puede esperar a que los trabajos se completen antes de reducir verticalmente el clúster, terminar los trabajos manualmente o reenviar los trabajos una vez que la operación de escalado haya concluido.

Para ver una lista de los trabajos pendientes y en ejecución, puede usar la interfaz de usuario de ResourceManager en YARN; para ello, siga estos pasos:

1. Inicie sesión en el [portal de Azure](https://portal.azure.com).
2. Desde la izquierda, vaya a **todos los servicios** > **Analytics** > **clústeres de HDInsight**y, a continuación, seleccione el clúster.
3. En la vista principal, vaya a **paneles de clúster** > **Ambari principal**. Escriba las credenciales de inicio de sesión del clúster.
4. En la UI de Ambari, seleccione **YARN** en la lista de servicios en el menú izquierdo.  
5. En la página YARN, seleccione **vínculos rápidos** y mantenga el mouse sobre el nodo principal activo, a continuación, seleccione **UI de ResourceManager**.

    ![UI de ResourceManager](./media/hdinsight-scaling-best-practices/resourcemanager-ui.png)

Puede acceder directamente a la UI de ResourceManager con `https://<HDInsightClusterName>.azurehdinsight.net/yarnui/hn/cluster`.

Verá una lista de trabajos, junto con su estado actual. En la captura de pantalla, hay un trabajo en ejecución actualmente:

![Aplicaciones de la UI de ResourceManager](./media/hdinsight-scaling-best-practices/resourcemanager-ui-applications.png)

Para terminar manualmente dicha aplicación en ejecución, ejecute el siguiente comando del shell de SSH:

```bash
yarn application -kill <application_id>
```

Por ejemplo:

```bash
yarn application -kill "application_1499348398273_0003"
```

## <a name="rebalancing-an-apache-hbase-cluster"></a>Reequilibrio de un clúster de Apache HBase

Los servidores regionales se equilibran automáticamente en unos pocos minutos tras completar la operación de escalado. Para equilibrar manualmente servidores regionales, siga estos pasos:

1. Conéctate al clúster de HDInsight con SSH: Para más información, consulte [Uso SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

2. Inicie el shell de HBase:

        hbase shell

3. Use el comando siguiente para equilibrar manualmente los servidores regionales:

        balancer

## <a name="scale-down-implications"></a>Reducción vertical de implicaciones

Como se mencionó anteriormente, los trabajos pendientes o en ejecución se terminan al realizar una operación de reducción vertical del escalado. Sin embargo, hay otras posibles implicaciones que la reducción vertical del escalado puede tener.

## <a name="hdinsight-name-node-stays-in-safe-mode-after-scaling-down"></a>El nodo de nombre de HDInsight permanece en modo seguro después de la reducción vertical de escalado

Si se reduce el clúster hasta el mínimo de un nodo de trabajo, Apache HDFS puede bloquearse en modo seguro cuando se reinician los nodos de trabajo debido a la aplicación de revisiones o inmediatamente después de la operación de escalado.

La causa principal de esto es que Hive usa algunos archivos `scratchdir` y, de forma predeterminada, espera tres réplicas de cada bloque, pero solo se puede realizar una réplica si se reduce verticalmente al mínimo de un nodo de trabajo. Como consecuencia, los archivos de `scratchdir` están *subreplicados*. Esto puede dar lugar a que HDFS permanezca en modo seguro cuando se reinician los servicios después de la operación de escalado.

Cuando se produce un intento de reducción vertical, HDInsight se basa en las interfaces de administración de Apache Ambari para retirar los nodos de trabajo adicionales no deseados, que replican sus bloques de HDFS en otros nodos de trabajo en línea, y, a continuación, con seguridad verticalmente el clúster. HDFS entra en modo seguro durante la ventana de mantenimiento, y se espera que salga de dicho modo una vez finalizado el escalado. En este punto, HDFS puede bloquearse en modo seguro.

HDFS se configura con un valor de 3 en `dfs.replication`. Por tanto, los bloques de los archivos temporales se subreplican siempre que haya menos de tres nodos de trabajo en línea, ya que no se dispone de las tres copias esperadas de cada bloque de archivos disponible.

Puede ejecutar un comando para que HDFS salga del modo seguro. Por ejemplo, si sabe que la única razón por la que el modo seguro está activado es que los archivos temporales están subreplicados, entonces puede salir de ese modo con seguridad. Esto se debe a que los archivos subreplicados son archivos temporales de Hive.

```bash
hdfs dfsadmin -D 'fs.default.name=hdfs://mycluster/' -safemode leave
```

Después de salir del modo seguro, puede quitar manualmente los archivos temporales o esperar a que Hive los limpie automáticamente.

### <a name="example-errors-when-safe-mode-is-turned-on"></a>Ejemplos de errores cuando el modo seguro está activado

* H070 No se puede abrir la sesión de Hive. org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.ipc.RetriableException): org.apache.hadoop.hdfs.server.namenode.SafeModeException: **No se puede crear el directorio** /tmp/hive/hive/819c215c-6d87-4311-97c8-4f0b9d2adcf0. **El nodo de nombre está en modo seguro**. Los 75 bloques notificados necesitan 12 bloques más para llegar al umbral de 0,9900 de un total de 87 bloques. El número de nodos de datos dinámicos de 10 ha alcanzado el número mínimo de 0. El modo seguro se desactivará automáticamente una vez alcanzados los umbrales.

* H100 No se puede enviar la instrucción para mostrar bases de datos: org.apache.thrift.transport.TTransportException: org.apache.http.conn.HttpHostConnectException: No se pudo conectar a hn0-clustername.servername.internal.cloudapp.net:10001 [hn0-clustername.servername. internal.cloudapp.net/1.1.1.1]: **Conexión rechazada**

* H020 No se pudo establecer la conexión con hn0-hdisrv.servername.bx.internal.cloudapp.net:10001: org.apache.thrift.transport.TTransportException: No se pudo crear la conexión http a http:\//hn0-hdisrv.servername.bx.internal.cloudapp.net:10001/. org.apache.http.conn.HttpHostConnectException: No se pudo conectar a hn0-hdisrv.servername.bx.internal.cloudapp.net:10001 [hn0-hdisrv.servername.bx.internal.cloudapp.net/10.0.0.28]: Conexión rechazada: org.apache.thrift.transport.TTransportException: No se pudo crear la conexión http a http:\//hn0-hdisrv.servername.bx.internal.cloudapp.net:10001/. org.apache.http.conn.HttpHostConnectException: No se pudo conectar a hn0-hdisrv.servername.bx.internal.cloudapp.net:10001 [hn0-hdisrv.servername.bx.internal.cloudapp.net/10.0.0.28]: **Conexión rechazada**

* Desde los registros de Hive: ADVERTENCIA [main]: server.HiveServer2 (HiveServer2.java:startHiveServer2(442)) – Error al iniciar HiveServer2 en el intento 21; se reintentará en 60 segundos java.lang.RuntimeException: Error al aplicar la directiva de autorización en la configuración de Hive: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.ipc.RetriableException): org.apache.hadoop.hdfs.server.namenode.SafeModeException: **No se puede crear el directorio** /tmp/hive/hive/70a42b8a-9437-466e-acbe-da90b1614374. **El nodo de nombre está en modo seguro**.
    Los 0 bloques notificados necesitan 9 bloques más para llegar al umbral de 0,9900 de un total de 9 bloques.
    El número de nodos de datos dinámicos de 10 ha alcanzado el número mínimo de 0. **El modo seguro se desactivará automáticamente una vez alcanzados los umbrales**.
    at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkNameNodeSafeMode(FSNamesystem.java:1324)

Puede revisar los registros del nodo de nombre en la carpeta `/var/log/hadoop/hdfs/`, cerca del momento en que se escaló el clúster, para ver cuándo entró en modo seguro. Los archivos de registro se denominan `Hadoop-hdfs-namenode-hn0-clustername.*`.

La causa de los errores anteriores es que Hive depende de los archivos temporales de HDFS durante la ejecución de consultas. Cuando HDFS entra en modo seguro, Hive no puede ejecutar las consultas porque no puede escribir en HDFS. Los archivos temporales de HDFS se encuentran en la unidad local montada en las máquinas virtuales de los nodos de trabajo individuales y se replican entre otros nodos de trabajo en tres réplicas, como mínimo.

El parámetro `hive.exec.scratchdir` de Hive se configura en `/etc/hive/conf/hive-site.xml`:

```xml
<property>
    <name>hive.exec.scratchdir</name>
    <value>hdfs://mycluster/tmp/hive</value>
</property>
```

### <a name="view-the-health-and-state-of-your-hdfs-file-system"></a>Consulta del estado y el mantenimiento del sistema de archivos HDFS

Puede consultar un informe de estado de cada nodo de nombre para ver si los nodos están en modo seguro. Para ver el informe, incluya SSH en cada nodo principal y ejecute el siguiente comando:

```
hdfs dfsadmin -D 'fs.default.name=hdfs://mycluster/' -safemode get
```

![Modo seguro desactivado](./media/hdinsight-scaling-best-practices/safe-mode-off.png)

> [!NOTE]  
> El conmutador `-D` es necesario porque el sistema de archivos predeterminado de HDInsight es Azure Storage o Azure Data Lake Storage. `-D` especifica que los comandos se ejecuten en el sistema de archivos HDFS local.

A continuación, puede consultar un informe en que se muestran los detalles del estado de HDFS:

```
hdfs dfsadmin -D 'fs.default.name=hdfs://mycluster/' -report
```

Este comando genera el resultado siguiente en un clúster correcto donde todos los bloques se replican al nivel esperado:

![Modo seguro desactivado](./media/hdinsight-scaling-best-practices/report.png)

HDFS admite el comando `fsck` para buscar incoherencias con varios archivos, como los bloques que faltan para un archivo o los bloques subreplicados. Para ejecutar el comando `fsck` en los archivos `scratchdir` (disco temporal):

```
hdfs fsck -D 'fs.default.name=hdfs://mycluster/' /tmp/hive/hive
```

Al realizar una ejecución en un sistema de archivos HDFS correcto sin bloques subreplicados, obtendrá un resultado similar al siguiente:

```
Connecting to namenode via http://hn0-scalin.name.bx.internal.cloudapp.net:30070/fsck?ugi=sshuser&path=%2Ftmp%2Fhive%2Fhive
FSCK started by sshuser (auth:SIMPLE) from /10.0.0.21 for path /tmp/hive/hive at Thu Jul 06 20:07:01 UTC 2017
..Status: HEALTHY
 Total size:    53 B
 Total dirs:    5
 Total files:   2
 Total symlinks:                0 (Files currently being written: 2)
 Total blocks (validated):      2 (avg. block size 26 B)
 Minimally replicated blocks:   2 (100.0 %)
 Over-replicated blocks:        0 (0.0 %)
 Under-replicated blocks:       0 (0.0 %)
 Mis-replicated blocks:         0 (0.0 %)
 Default replication factor:    3
 Average block replication:     3.0
 Corrupt blocks:                0
 Missing replicas:              0 (0.0 %)
 Number of data-nodes:          4
 Number of racks:               1
FSCK ended at Thu Jul 06 20:07:01 UTC 2017 in 3 milliseconds


The filesystem under path '/tmp/hive/hive' is HEALTHY
```

En cambio, al ejecutar el comando `fsck` en un sistema de archivos HDFS con algunos bloques subreplicados, el resultado será similar al siguiente:

```
Connecting to namenode via http://hn0-scalin.name.bx.internal.cloudapp.net:30070/fsck?ugi=sshuser&path=%2Ftmp%2Fhive%2Fhive
FSCK started by sshuser (auth:SIMPLE) from /10.0.0.21 for path /tmp/hive/hive at Thu Jul 06 20:13:58 UTC 2017
.
/tmp/hive/hive/4f3f4253-e6d0-42ac-88bc-90f0ea03602c/inuse.info:  Under replicated BP-1867508080-10.0.0.21-1499348422953:blk_1073741826_1002. Target Replicas is 3 but found 1 live replica(s), 0 decommissioned replica(s) and 0 decommissioning replica(s).
.
/tmp/hive/hive/e7c03964-ff3a-4ee1-aa3c-90637a1f4591/inuse.info: CORRUPT blockpool BP-1867508080-10.0.0.21-1499348422953 block blk_1073741825

/tmp/hive/hive/e7c03964-ff3a-4ee1-aa3c-90637a1f4591/inuse.info: MISSING 1 blocks of total size 26 B.Status: CORRUPT
 Total size:    53 B
 Total dirs:    5
 Total files:   2
 Total symlinks:                0 (Files currently being written: 2)
 Total blocks (validated):      2 (avg. block size 26 B)
  ********************************
  UNDER MIN REPL'D BLOCKS:      1 (50.0 %)
  dfs.namenode.replication.min: 1
  CORRUPT FILES:        1
  MISSING BLOCKS:       1
  MISSING SIZE:         26 B
  CORRUPT BLOCKS:       1
  ********************************
 Minimally replicated blocks:   1 (50.0 %)
 Over-replicated blocks:        0 (0.0 %)
 Under-replicated blocks:       1 (50.0 %)
 Mis-replicated blocks:         0 (0.0 %)
 Default replication factor:    3
 Average block replication:     0.5
 Corrupt blocks:                1
 Missing replicas:              2 (33.333332 %)
 Number of data-nodes:          1
 Number of racks:               1
FSCK ended at Thu Jul 06 20:13:58 UTC 2017 in 28 milliseconds


The filesystem under path '/tmp/hive/hive' is CORRUPT
```

También puede consultar el estado de HDFS en la interfaz de usuario de Ambari seleccionando el servicio **HDFS** a la izquierda, o bien con `https://<HDInsightClusterName>.azurehdinsight.net/#/main/services/HDFS/summary`.

![Estado de HDFS en Ambari](./media/hdinsight-scaling-best-practices/ambari-hdfs.png)

Puede que también vea uno o varios errores críticos en los nodos de nombre activos o en espera. Para consultar el estado de los bloques de nodos de nombre, seleccione el vínculo NameNode que se encuentra junto a la alerta.

![Estado de bloques de nodos de nombre](./media/hdinsight-scaling-best-practices/ambari-hdfs-crit.png)

Para limpiar los archivos temporales, que quita los errores de replicación de bloque, SSH en cada nodo principal y ejecute el siguiente comando:

```
hadoop fs -rm -r -skipTrash hdfs://mycluster/tmp/hive/
```

> [!NOTE]  
> Este comando puede interrumpir Hive si algunos trabajos siguen en ejecución.

### <a name="how-to-prevent-hdinsight-from-getting-stuck-in-safe-mode-due-to-under-replicated-blocks"></a>Cómo evitar que HDInsight quede bloqueado en modo seguro debido a bloques subreplicados

Hay varias maneras de evitar que HDInsight se quede en modo seguro:

* Detenga todos los trabajos de Hive antes de reducir verticalmente HDInsight. Como alternativa, programe el proceso de reducción vertical para evitar que entre en conflicto con los trabajos de Hive en ejecución.
* Limpie manualmente los archivos temporales del directorio `tmp` de Hive en HDFS antes de la reducción vertical.
* Solo reduzca verticalmente HDInsight a tres nodos de trabajo como mínimo. Evite reducir a un solo nodo de trabajo.
* Ejecute el comando para salir del modo seguro, si es necesario.

Estas opciones se describen en las secciones siguientes.

#### <a name="stop-all-hive-jobs"></a>Detención de todos los trabajos de Hive

Detenga todos los trabajos de Hive antes de reducir verticalmente a un nodo de trabajo. Si la carga de trabajo está programada, ejecute la reducción vertical una vez realizado el trabajo de Hive.

Esto ayuda a minimizar el número de archivos temporales en la carpeta tmp (si existe).

#### <a name="manually-clean-up-hives-scratch-files"></a>Limpieza manual de los archivos temporales de Hive

Si Hive ha dejado archivos temporales, puede limpiarlos manualmente antes de la reducción vertical para evitar la activación del modo seguro.

1. Detenga los servicios de Hive y asegúrese de que se hayan completado todas las consultas y los trabajos.

2. Enumere el contenido del directorio `hdfs://mycluster/tmp/hive/` para ver si contiene algún archivo:

    ```
    hadoop fs -ls -R hdfs://mycluster/tmp/hive/hive
    ```
    
    Este es un ejemplo de resultado cuando existen archivos:

    ```
    sshuser@hn0-scalin:~$ hadoop fs -ls -R hdfs://mycluster/tmp/hive/hive
    drwx------   - hive hdfs          0 2017-07-06 13:40 hdfs://mycluster/tmp/hive/hive/4f3f4253-e6d0-42ac-88bc-90f0ea03602c
    drwx------   - hive hdfs          0 2017-07-06 13:40 hdfs://mycluster/tmp/hive/hive/4f3f4253-e6d0-42ac-88bc-90f0ea03602c/_tmp_space.db
    -rw-r--r--   3 hive hdfs         27 2017-07-06 13:40 hdfs://mycluster/tmp/hive/hive/4f3f4253-e6d0-42ac-88bc-90f0ea03602c/inuse.info
    -rw-r--r--   3 hive hdfs          0 2017-07-06 13:40 hdfs://mycluster/tmp/hive/hive/4f3f4253-e6d0-42ac-88bc-90f0ea03602c/inuse.lck
    drwx------   - hive hdfs          0 2017-07-06 20:30 hdfs://mycluster/tmp/hive/hive/c108f1c2-453e-400f-ac3e-e3a9b0d22699
    -rw-r--r--   3 hive hdfs         26 2017-07-06 20:30 hdfs://mycluster/tmp/hive/hive/c108f1c2-453e-400f-ac3e-e3a9b0d22699/inuse.info
    ```

3. Si sabe que Hive ya no necesita estos archivos, puede eliminarlos. Asegúrese de que Hive no tiene ninguna consulta en ejecución; para ello, consulte la página de la interfaz de usuario de ResourceManager en Yarn.

    Línea de comandos de ejemplo para quitar archivos de HDFS:

    ```
    hadoop fs -rm -r -skipTrash hdfs://mycluster/tmp/hive/
    ```
    
#### <a name="scale--hdinsight-to-three-worker-nodes"></a>Escalado de HDInsight a tres nodos de trabajo

Si la activación del modo seguro es un problema persistente y los pasos anteriores no sirven para solucionarlo, debe evitar el problema realizando la reducción vertical solo a tres nodos de trabajo. Puede que no sea una opción óptima por las restricciones de costos en comparación con la reducción vertical a un solo nodo. Sin embargo, con un único nodo de trabajo, HDFS no puede garantizar la disponibilidad de tres réplicas de los datos en el clúster.

#### <a name="run-the-command-to-leave-safe-mode"></a>Ejecución del comando para salir del modo seguro

La última opción consiste en estar atento a los casos excepcionales en que HDFS entra en modo seguro, porque debe ejecute el comando para salir del modo seguro. Una vez haya determinado que la razón por la que HDFS ha entrado en modo seguro es porque los archivos de Hive están subreplicados, ejecute el siguiente comando para salir del modo seguro:

* HDInsight en Linux:

    ```bash
    hdfs dfsadmin -D 'fs.default.name=hdfs://mycluster/' -safemode leave
    ```
    
* HDInsight en Windows:

    ```bash
    hdfs dfsadmin -fs hdfs://headnodehost:9000 -safemode leave
    ```
    
## <a name="next-steps"></a>Pasos siguientes

* [Escalado automático de clústeres de HDInsight de Azure](hdinsight-autoscale-clusters.md)
* [Introducción a Azure HDInsight](hadoop/apache-hadoop-introduction.md)
* [Escalado de clústeres](hdinsight-administer-use-portal-linux.md#scale-clusters)
