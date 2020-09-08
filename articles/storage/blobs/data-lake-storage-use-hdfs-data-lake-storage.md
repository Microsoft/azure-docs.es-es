---
title: Uso de la CLI de HDFS con Azure Data Lake Storage Gen2
description: Use la CLI de Hadoop Distributed File System (HDFS) CLI para Azure Data Lake Storage Gen2. Cree un contenedor, obtenga una lista de archivos o directorios, etc.
services: storage
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/06/2018
ms.author: normesta
ms.subservice: data-lake-storage-gen2
ms.reviewer: artek
ms.openlocfilehash: 1d720aed44358dd314bc4226adb9ad517139cd18
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836316"
---
# <a name="using-the-hdfs-cli-with-data-lake-storage-gen2"></a>Uso de la CLI de HDFS con Data Lake Storage Gen2

Puede acceder y administrar los datos en la cuenta de almacenamiento mediante una interfaz de línea de comandos como lo haría con un [Sistema de archivos distribuido de Hadoop (HDFS)](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html). En este artículo se dan algunos ejemplos que le ayudarán a empezar a trabajar.

HDInsight proporciona acceso al contenedor distribuido que se adjunta localmente a los nodos de ejecución. Puede acceder a este contenedor mediante el shell que interactúa directamente con HDFS y con otros sistemas de archivos que admiten Hadoop.

Para más información sobre la CLI de HDFS, consulte la [documentación oficial](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html) y la [Guía de permisos de HDFS](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html).

>[!NOTE]
>Si usa Azure Databricks, en lugar de HDInsight, y quiere interactuar con los datos mediante una interfaz de línea de comandos, puede usar la CLI de Databricks para interactuar con el sistema de archivos de Databricks. Consulte la [CLI de Databricks](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html).

## <a name="use-the-hdfs-cli-with-an-hdinsight-hadoop-cluster-on-linux"></a>Usar la CLI de HDFS con un clúster de HDInsight Hadoop en Linux

En primer lugar, establezca el [acceso remoto a los servicios](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-linux-information#remote-access-to-services). Si selecciona [SSH](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix) el ejemplo de código de PowerShell tendría el siguiente aspecto:

```powershell
#Connect to the cluster via SSH.
ssh sshuser@clustername-ssh.azurehdinsight.net
#Execute basic HDFS commands. Display the hierarchy.
hdfs dfs -ls /
#Create a sample directory.
hdfs dfs -mkdir /samplefolder
```
La cadena de conexión se encuentra en la sección "SSH e inicio de sesión del clúster" de la hoja del clúster de HDInsight en Azure Portal. Las credenciales SSH se especificaron en el momento de la creación del clúster

>[!IMPORTANT]
>La facturación del clúster de HDInsight se inicia una vez creado el clúster y se detiene cuando se elimina. Se facturan por minuto realizando una prorrata, por lo que siempre debe eliminar aquellos que ya no se estén utilizando. Para obtener información sobre cómo eliminar un clúster, consulte nuestro [artículo sobre el tema](../../hdinsight/hdinsight-delete-cluster.md). Sin embargo, los datos almacenados en una cuenta de almacenamiento que tiene Data Lake Storage Gen2 habilitado continúan incluso después de eliminar un clúster de HDInsight.

## <a name="create-a-container"></a>Crear un contenedor

`hdfs dfs -D "fs.azure.createRemoteFileSystemDuringInitialization=true" -ls abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/`

* Reemplace el marcador de posición `<container-name>` por el nombre que desee asignar al contenedor.

* Reemplace el marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

## <a name="get-a-list-of-files-or-directories"></a>Obtener una lista de archivos o directorios

`hdfs dfs -ls <path>`

Reemplace el marcador de posición `<path>` por el URI del contenedor o de la carpeta de contenedor.

Por ejemplo: `hdfs dfs -ls abfs://my-file-system@mystorageaccount.dfs.core.windows.net/my-directory-name`

## <a name="create-a-directory"></a>Creación de un directorio

`hdfs dfs -mkdir [-p] <path>`

Reemplace el marcador de posición `<path>` por el nombre del contenedor raíz o de una carpeta dentro del contenedor.

Por ejemplo: `hdfs dfs -mkdir abfs://my-file-system@mystorageaccount.dfs.core.windows.net/`

## <a name="delete-a-file-or-directory"></a>Eliminar un archivo o un directorio

`hdfs dfs -rm <path>`

Reemplace el marcador de posición `<path>` por el URI del archivo o de la carpeta que desee eliminar.

Por ejemplo: `hdfs dfs -rmdir abfs://my-file-system@mystorageaccount.dfs.core.windows.net/my-directory-name/my-file-name`

## <a name="display-the-access-control-lists-acls-of-files-and-directories"></a>Mostrar las listas de control de acceso (ACL) de archivos y directorios

`hdfs dfs -getfacl [-R] <path>`

Ejemplo:

`hdfs dfs -getfacl -R /dir`

Consulte [getfacl](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#getfacl).

## <a name="set-acls-of-files-and-directories"></a>Definir listas de control de acceso de archivos y directorios

`hdfs dfs -setfacl [-R] [-b|-k -m|-x <acl_spec> <path>]|[--set <acl_spec> <path>]`

Ejemplo:

`hdfs dfs -setfacl -m user:hadoop:rw- /file`

Consulte [setfacl](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#setfacl).

## <a name="change-the-owner-of-files"></a>Cambiar el propietario de los archivos

`hdfs dfs -chown [-R] <new_owner>:<users_group> <URI>`

Consulte [chown](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#chown).

## <a name="change-group-association-of-files"></a>Cambiar la asociación del grupo de archivos

`hdfs dfs -chgrp [-R] <group> <URI>`

Consulte [chgrp](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#chgrp).

## <a name="change-the-permissions-of-files"></a>Cambiar los permisos de archivos

`hdfs dfs -chmod [-R] <mode> <URI>`

Consulte [chmod](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#chmod).

Puede consultar una lista completa de comandos en el sitio web de la [guía de comandos shell del sistema de archivos de Apache Hadoop 2.4.1](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html).

## <a name="next-steps"></a>Pasos siguientes

* [Uso de una cuenta compatible de Azure Data Lake Storage Gen2 en Azure Databricks](./data-lake-storage-quickstart-create-databricks-account.md)

* [Información sobre las listas de control de acceso en archivos y directorios](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control)
