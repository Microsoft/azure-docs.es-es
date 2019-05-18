---
title: Migre cargas de trabajo de Hive de Azure HDInsight 3.6 a 4.0 de HDInsight
description: Obtenga información sobre cómo migrar cargas de trabajo de Apache Hive en HDInsight 3.6 a 4.0 de HDInsight.
ms.service: hdinsight
author: msft-tacox
ms.author: tacox
ms.reviewer: jasonh
ms.topic: howto
ms.date: 04/24/2019
ms.openlocfilehash: b39279e560cb1738ff9b33ec587562efd2ed4e8d
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65800949"
---
# <a name="migrate-azure-hdinsight-36-hive-workloads-to-hdinsight-40"></a>Migre cargas de trabajo de Hive de Azure HDInsight 3.6 a 4.0 de HDInsight

Este documento muestra cómo migrar las cargas de trabajo de Apache Hive y LLAP en HDInsight 3.6 a 4.0 de HDInsight. HDInsight 4.0 proporciona las características más recientes de Hive y LLAP como vistas materializadas y almacenamiento en caché de resultados de consulta. Al migrar las cargas de trabajo a HDInsight 4.0, puede utilizar muchas características más recientes de Hive 3 que no están disponibles en HDInsight 3.6.

En este artículo se trata a los siguientes temas:

* Migración de los metadatos de Hive en HDInsight 4.0
* Migración segura de ácido y tablas sin ácido
* Conservación de las directivas de seguridad de Hive en las versiones de HDInsight
* Ejecución de consultas y la depuración de HDInsight 3.6 a 4.0 de HDInsight

## <a name="migrate-apache-hive-metadata-to-hdinsight-40"></a>Migrar los metadatos de Apache Hive en HDInsight 4.0

Una ventaja de Hive es la capacidad de exportar los metadatos a una base de datos externo (denominado la tienda de metadatos de Hive). El **Hive Metastore** es responsable de almacenar las estadísticas de tabla, incluida la ubicación de almacenamiento de tabla, nombres de columna e información de índices de tabla. El esquema de base de datos de la tienda de metadatos difiere entre las versiones de Hive. Siga este procedimiento para actualizar una tienda de metadatos de Hive de HDInsight 3.6 para que sea compatible con HDInsight 4.0.

1. Cree una nueva copia de la tienda de metadatos externo. La versión 3.6 de HDInsight y HDInsight 4.0 requieren esquemas diferentes de la tienda de metadatos y no pueden compartir una tienda de metadatos único. Consulte [utilizan almacenes de metadatos externos en Azure HDInsight](../hdinsight-use-external-metadata-stores.md) para más información acerca de cómo adjuntar una tienda de metadatos externo a un clúster de HDInsight. 
2. Inicie una acción de script en el clúster de HDI 3.6 con "Nodos principales" como el tipo de nodo para su ejecución. Pegue el siguiente URI en el cuadro de texto marcadas "URI de Script de Bash": https://hdiconfigactions.blob.core.windows.net/hivemetastoreschemaupgrade/launch-schema-upgrade.sh. En el cuadro de texto marcada "Argumentos", escriba el servername, base de datos, nombre de usuario y contraseña para el **copian** Hive metastore, separado por espacios. No incluya ". database.windows.net" al especificar el nombre del servidor.

> [!Warning]
> No se puede revertir la actualización que convierte el esquema de metadatos 3.6 de HDInsight en el esquema de HDInsight 4.0.

## <a name="migrate-hive-tables-to-hdinsight-40"></a>Migrar las tablas de Hive en HDInsight 4.0

Después de completar el conjunto de pasos para migrar la tienda de metadatos de Hive en HDInsight 4.0 anterior, las tablas y bases de datos que se registran en la tienda de metadatos será visibles desde dentro del clúster de HDInsight 4.0 mediante la ejecución de `show tables` o `show databases` desde dentro del clúster . Consulte [ejecución de consultas en las versiones de HDInsight](#query-execution-across-hdinsight-versions) para obtener información sobre la ejecución de consultas en los clústeres de HDInsight 4.0.

Los datos reales de las tablas, sin embargo, no son accesibles hasta que el clúster tiene acceso a las cuentas de almacenamiento necesarios. Para asegurarse de que el clúster de HDInsight 4.0 puede acceder a los mismos datos que el antiguo clúster de HDInsight 3.6, complete los pasos siguientes:

1. Determinar la cuenta de almacenamiento de Azure de la tabla o base de datos mediante describir el formato.
2. Si ya se está ejecutando el clúster de HDInsight 4.0, asociar a la cuenta de almacenamiento de Azure al clúster a través de Ambari. Si todavía no ha creado el clúster de HDInsight 4.0, asegúrese de que la cuenta de almacenamiento de Azure se especifica como la réplica principal o una cuenta de almacenamiento de clúster secundario. Para obtener más información acerca de cómo agregar cuentas de almacenamiento para clústeres de HDInsight, consulte [agregar más cuentas de almacenamiento a HDInsight](../hdinsight-hadoop-add-storage.md).

> [!Note]
> Tablas se tratan de manera diferente en la versión 3.6 de HDInsight y HDInsight 4.0. Por este motivo, no pueden compartir las mismas tablas para los clústeres de distintas versiones. Si desea usar 3.6 de HDInsight al mismo tiempo que HDInsight 4.0, debe tener copias independientes de los datos para cada versión.

La carga de trabajo de Hive puede incluir una mezcla de ácido y tablas que no sean de ácido. Una diferencia clave entre Hive en HDInsight 3.6 (sección 2) y Hive en HDInsight 4.0 (Hive 3) es el cumplimiento de ACID para las tablas. En HDInsight 3.6, habilitar el cumplimiento de ácido de Hive requiere configuración adicional, pero en 4.0 de HDInsight son compatibles con el ácido tablas de forma predeterminada. La única acción necesaria antes de ejecutar una compactación principal en la tabla de ácido en el clúster 3,6 migración. Desde la vista de Hive o Beeline, ejecute la consulta siguiente:

```bash
alter table myacidtable compact 'major';
```

Esta compactación es necesaria porque las tablas de la versión 3.6 de HDInsight y HDInsight 4.0 ácido comprender deltas ACID diferente. Compactación exige una pizarra limpia que garantiza la coherencia. Sección 4 de la [Hive documentación de migración](https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-upgrade-major/content/prepare_hive_for_upgrade.html) contiene instrucciones para la compactación de forma masiva de tablas de HDInsight 3.6 ácido.

Una vez completados los pasos de migración y la compactación de tienda de metadatos, puede migrar el almacenamiento real. Después de completar la migración de almacenamiento de Hive, el almacenamiento de HDInsight 4.0 tendrá las siguientes propiedades:

* Las tablas externas en HDInsight 3.6 estará tablas externas en HDInsight 4.0
* Tablas administradas no transaccional en HDInsight 3.6 estará tablas externas en HDInsight 4.0
* Transaccionales tablas administradas en HDInsight 3.6 estará tablas administradas en HDInsight 4.0

Es posible que deba ajustar las propiedades del almacén antes de ejecutar la migración. Por ejemplo, si espera que se accederá a alguna tabla por terceros (por ejemplo, un clúster de HDInsight 3.6), esa tabla debe ser externa una vez completada la migración. En HDInsight 4.0, todas las tablas administradas son transaccionales. Por lo tanto, solo se deben acceder a tablas administradas en HDInsight 4.0 en los clústeres de HDInsight 4.0.

Una vez establecidas las propiedades de tabla correctamente, ejecute la herramienta de migración de almacenamiento de Hive desde uno de los nodos principales del clúster mediante el shell de SSH:

1. Conéctese a su nodo principal del clúster mediante SSH. Para obtener instrucciones, consulte [conectarse a HDInsight mediante SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)
1. Abra un shell de inicio de sesión como el usuario de Hive mediante la ejecución `sudo su - hive`
1. Determinar la versión de la pila de Hortonworks Data Platform ejecutando `ls /usr/hdp`. Esto mostrará una cadena de versión que debe usar en el siguiente comando.
1. Ejecute el siguiente comando desde el shell. Reemplace `${{STACK_VERSION}}` con la cadena de versión en el paso anterior:

```bash
/usr/hdp/${{STACK_VERSION}}/hive/bin/hive --config /etc/hive/conf --service  strictmanagedmigration --hiveconf hive.strict.managed.tables=true  -m automatic  automatic  --modifyManagedTables --oldWarehouseRoot /apps/hive/warehouse
```

Una vez finalizada la herramienta de migración, el almacenamiento de Hive estará listo para HDInsight 4.0. 

> [!Important]
> No se deben tener acceso a tablas administradas en 4.0 HDInsight (incluidas tablas se migran desde 3.6) por otros servicios o aplicaciones, incluidos los clústeres de HDInsight 3.6.

## <a name="secure-hive-across-hdinsight-versions"></a>Proteja Hive en las versiones de HDInsight

Desde HDInsight 3.6, HDInsight se integra con Azure Active Directory mediante el paquete de seguridad empresarial de HDInsight (ESP). ESP utiliza Kerberos y Apache Ranger para administrar los permisos de recursos específicos dentro del clúster. Las directivas de Ranger implementadas con Hive en HDInsight 3.6 se pueden migrar a HDInsight 4.0 con los pasos siguientes:

1. Vaya al panel de Ranger Service Manager en el clúster de HDInsight 3.6.
2. Vaya a la directiva denominada **HIVE** y exportar la directiva a un archivo json.
3. Asegúrese de que todos los usuarios que se hace referencia en el json de directiva exportadas existen en el nuevo clúster. Si un usuario se conoce en el json de directiva, pero no existe en el nuevo clúster, agregue el usuario al nuevo clúster o quitar la referencia de la directiva.
4. Navegue hasta la **Ranger Service Manager** panel en el clúster de HDInsight 4.0.
5. Vaya a la directiva denominada **HIVE** e importar el archivo json de directiva de ranger del paso 2.

## <a name="query-execution-across-hdinsight-versions"></a>Ejecución de consultas en las versiones de HDInsight

Hay dos maneras de ejecutar y depurar consultas de Hive/LLAP dentro de un clúster de HDInsight 3.6. HiveCLI proporciona una experiencia de línea de comandos y la vista de Hive/vista de Tez proporciona un flujo de trabajo basado en GUI.

En HDInsight 4.0, se ha reemplazado HiveCLI con Beeline. HiveCLI es un cliente de thrift para 1 Hiveserver y Beeline es un cliente JDBC que proporciona acceso a Hiveserver 2. BEELINE también se puede usar para conectarse a cualquier otro extremo de la base de datos compatible con JDBC. BEELINE es disponibles de fábrica en HDInsight 4.0 sin ninguna instalación necesitado.

En HDInsight 3.6, el cliente de la interfaz gráfica de usuario para interactuar con el servidor de Hive es la vista de Hive de Ambari. HDInsight 4.0 reemplaza a la vista de Hive con Hortonworks Data Analytics Studio (DAS). DAS no se suministra con clústeres de HDInsight out-of-box y no es un paquete oficialmente compatible. Sin embargo, DAS puede instalarse en el clúster como sigue:

Inicie una acción de script en el clúster, con "Nodos principales" como el tipo de nodo para su ejecución. Pegue el siguiente URI en el cuadro de texto marcada "URI de Script de Bash": https://hdiconfigactions.blob.core.windows.net/dasinstaller/LaunchDASInstaller.sh

Studio de análisis de datos se puede iniciar con la dirección URL: https://<clustername>.azurehdinsight.net/das/



Una vez instalado DAS, si no ve las consultas que se haya ejecutado en el Visor de consultas, realice los pasos siguientes:

1. Establezca las configuraciones de Hive, Tez y DAS, como se describe en [esta guía para solucionar problemas de instalación de DAS](https://docs.hortonworks.com/HDPDocuments/DAS/DAS-1.2.0/troubleshooting/content/das_queries_not_appearing.html).
2. Asegúrese de que las configuraciones de directorio de almacenamiento de Azure siguientes son blobs en páginas, y que aparecen en `fs.azure.page.blob.dirs`:
    * `hive.hook.proto.base-directory`
    * `tez.history.logging.proto-base-dir`
3. Reinicie DAS, Tez, Hive y HDFS en ambos nodos principales.

## <a name="next-steps"></a>Pasos siguientes

* [Anuncio de HDInsight 4.0](../hdinsight-version-release.md)
* [Profundización en HDInsight 4.0](https://azure.microsoft.com/blog/deep-dive-into-azure-hdinsight-4-0/)
* [3 tablas de ácido de Hive](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.1.0/using-hiveql/content/hive_3_internals.html)
