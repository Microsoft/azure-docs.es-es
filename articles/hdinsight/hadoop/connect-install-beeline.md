---
title: 'Instalación o conexión a Apache Beeline: Azure HDInsight'
description: Aprenda a conectarse al cliente de Apache Beeline para ejecutar consultas de Hive con Hadoop en HDInsight. Beeline es una utilidad para trabajar con HiveServer2 sobre JDBC.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: contperfq1
ms.date: 05/27/2020
ms.openlocfilehash: 5495e6c6392ba2e824a0a70717bd19747db9b754
ms.sourcegitcommit: afa1411c3fb2084cccc4262860aab4f0b5c994ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/23/2020
ms.locfileid: "88754963"
---
# <a name="connect-to-apache-beeline-on-hdinsight-or-install-it-locally"></a>Conexión a Apache Beeline en HDInsight o instalación local

[Apache Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline–NewCommandLineShell) es un cliente de Hive que se incluye en los nodos principales del clúster de HDInsight. En este artículo se describe cómo conectarse al cliente de Beeline instalado en el clúster de HDInsight a través de diferentes tipos de conexiones. También se explica cómo [instalar el cliente de Beeline localmente](#install-beeline-client). 

## <a name="types-of-connections"></a>Tipos de conexiones

### <a name="from-an-ssh-session"></a>Desde una sesión de SSH

Si se conecta desde una sesión de SSH a un nodo principal del clúster, puede conectarse a la dirección `headnodehost` en el puerto `10001`:

```bash
beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http'
```

### <a name="over-an-azure-virtual-network"></a>A través de una instancia de Azure Virtual Network

Si se conecta desde un cliente a HDInsight a través de una instancia de Azure Virtual Network, debe proporcionar el nombre de dominio completo (FQDN) de un nodo principal del clúster. Puesto que esta conexión se realiza directamente a los nodos del clúster, se usa el puerto `10001`:

```bash
beeline -u 'jdbc:hive2://<headnode-FQDN>:10001/;transportMode=http'
```

Reemplace `<headnode-FQDN>` por el nombre de dominio completo de un nodo principal primario. Para buscar el nombre de dominio completo de un nodo principal, use la información del documento [Administración de clústeres de HDInsight con la API REST de Apache Ambari](../hdinsight-hadoop-manage-ambari-rest-api.md#get-the-fqdn-of-cluster-nodes).

### <a name="to-hdinsight-enterprise-security-package-esp-cluster-using-kerberos"></a>Al clúster de Enterprise Security Package (ESP) de HDInsight mediante Kerberos

Al conectarse desde un cliente a un clúster de Enterprise Security Package (ESP) unido a Azure Active Directory (AAD)-DS en una máquina del mismo dominio del clúster, también debe especificar el nombre de dominio `<AAD-Domain>` y el nombre de una cuenta de usuario de dominio que tenga permisos para acceder al clúster `<username>`:

```bash
kinit <username>
beeline -u 'jdbc:hive2://<headnode-FQDN>:10001/default;principal=hive/_HOST@<AAD-Domain>;auth-kerberos;transportMode=http' -n <username>
```

Reemplace `<username>` con el nombre de una cuenta en el dominio con permisos para obtener acceso al clúster. Reemplace `<AAD-DOMAIN>` por el nombre de la instancia de Azure Active Directory (AAD) a la que está unido el clúster. Use una cadena en mayúsculas para el valor `<AAD-DOMAIN>`; de lo contrario, la credencial no se encontrará. Consulte en `/etc/krb5.conf` los nombres de dominio kerberos si es necesario.

Para buscar la dirección URL de JDBC desde Ambari:

1. En un explorador web, vaya a `https://CLUSTERNAME.azurehdinsight.net/#/main/services/HIVE/summary`, donde `CLUSTERNAME` es el nombre del clúster. Asegúrese de que HiveServer2 está en ejecución.

1. Use el portapapeles para copiar la dirección URL de JDBC de HiveServer2.

### <a name="over-public-or-private-endpoints"></a>A través de puntos de conexión públicos o privados

Al conectarse a un clúster mediante los puntos de conexión públicos o privados, debe proporcionar el nombre de la cuenta de inicio de sesión (valor predeterminado `admin`) y la contraseña del clúster. Por ejemplo, al usar Beeline desde un sistema cliente para conectarse a la dirección `clustername.azurehdinsight.net`. Esta conexión se realiza a través del puerto `443` y se cifra mediante TLS/SSL.

Reemplace `clustername` por el nombre del clúster de HDInsight. Reemplace `admin` por la cuenta de inicio de sesión del clúster. Para los clústeres de ESP, use el UPN completo (por ejemplo, user@domain.com). Reemplace `password` por la contraseña de la cuenta de inicio de sesión del clúster.

```bash
beeline -u 'jdbc:hive2://clustername.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/hive2' -n admin -p 'password'
```

o para el punto de conexión privado:

```bash
beeline -u 'jdbc:hive2://clustername-int.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/hive2' -n admin -p 'password'
```

Los puntos de conexión privados apuntan a un equilibrador de carga básico al que solo se puede acceder desde las redes virtuales emparejadas en la misma región. Consulte cuáles son las [restricciones relacionadas con Emparejamiento de VNet global y los equilibradores de carga](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers) para más información. Puede usar el comando `curl` con la opción `-v` para solucionar cualquier problema de conectividad con puntos de conexión públicos o privados antes de usar Beeline.

### <a name="use-beeline-with-apache-spark"></a>Uso de Beeline con Apache Spark

Apache Spark proporciona su propia implementación de HiveServer2 que, en algunas ocasiones, se denomina servidor Thrift de Spark. Este servicio usa Spark SQL para resolver consultas en lugar de Hive. Además, puede proporcionar un mejor rendimiento en función de la consulta.

#### <a name="through-public-or-private-endpoints"></a>A través de puntos de conexión públicos o privados

La cadena de conexión usada es ligeramente distinta. En lugar de contener `httpPath=/hive2`, usa `httpPath/sparkhive2`. Reemplace `clustername` por el nombre del clúster de HDInsight. Reemplace `admin` por la cuenta de inicio de sesión del clúster. Para los clústeres de ESP, use el UPN completo (por ejemplo, user@domain.com). Reemplace `password` por la contraseña de la cuenta de inicio de sesión del clúster.

```bash
beeline -u 'jdbc:hive2://clustername.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/sparkhive2' -n admin -p 'password'
```

o para el punto de conexión privado:

```bash
beeline -u 'jdbc:hive2://clustername-int.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/sparkhive2' -n admin -p 'password'
```

Los puntos de conexión privados apuntan a un equilibrador de carga básico al que solo se puede acceder desde las redes virtuales emparejadas en la misma región. Consulte cuáles son las [restricciones relacionadas con Emparejamiento de VNet global y los equilibradores de carga](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers) para más información. Puede usar el comando `curl` con la opción `-v` para solucionar cualquier problema de conectividad con puntos de conexión públicos o privados antes de usar Beeline.

#### <a name="from-cluster-head-or-inside-azure-virtual-network-with-apache-spark"></a>Desde el nodo principal del clúster o en Azure Virtual Network con Apache Spark

Con la conexión directa desde el nodo principal del clúster o desde un recurso de la misma red virtual de Azure que la del clúster de HDInsight, debe utilizarse el puerto `10002` para el servidor Thrift de Spark en lugar de `10001`. El ejemplo siguiente muestra cómo conectarse directamente al nodo principal:

```bash
/usr/hdp/current/spark2-client/bin/beeline -u 'jdbc:hive2://headnodehost:10002/;transportMode=http'
```

## <a name="install-beeline-client"></a>Instalación de un cliente de Beeline

Aunque Beeline se incluye en los nodos principales, puede que desee instalarlo localmente.  Los pasos de instalación en una máquina local se basan en un [subsistema de Windows para Linux](https://docs.microsoft.com/windows/wsl/install-win10).

1. Listas de actualizaciones. Escriba los comandos siguientes en su shell de Bash:

    ```bash
    sudo apt-get update
    ```

1. Instale Java si no está instalado. Puede hacerlo con el comando `which java`.

    1. Si no hay instalado ningún paquete de Java, escriba el siguiente comando:

        ```bash
        sudo apt install openjdk-11-jre-headless
        ```

    1. Abra el archivo bashrc (a menudo se encuentra en ~/.bashrc): `nano ~/.bashrc`.

    1. Modifique el archivo bashrc. Agregue la siguiente línea al final del archivo:

        ```bash
        export JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64
        ```

        Para guardar el archivo, presione **Ctrl+x**, luego **Y** y después Entrar.

1. Descargue los archivos de Hadoop y Beeline, y escriba los siguientes comandos:

    ```bash
    wget https://archive.apache.org/dist/hadoop/core/hadoop-2.7.3/hadoop-2.7.3.tar.gz
    wget https://archive.apache.org/dist/hive/hive-1.2.1/apache-hive-1.2.1-bin.tar.gz
    ```

1. Desempaquete los archivos y escriba los comandos siguientes:

    ```bash
    tar -xvzf hadoop-2.7.3.tar.gz
    tar -xvzf apache-hive-1.2.1-bin.tar.gz
    ```

1. Modifique aún más el archivo bashrc. Deberá identificar la ruta de acceso en la que se desempaquetan los archivos. Si usa el [subsistema de Windows para Linux](https://docs.microsoft.com/windows/wsl/install-win10) y siguió los pasos de manera exacta, la ruta de acceso sería `/mnt/c/Users/user/`, donde `user` es el nombre de usuario.

    1. Abra el archivo `nano ~/.bashrc`

    1. Modifique los comandos siguientes con la ruta de acceso adecuada y, a continuación, escríbalos al final del archivo bashrc:

        ```bash
        export HADOOP_HOME=/path_where_the_archives_were_unpacked/hadoop-2.7.3
        export HIVE_HOME=/path_where_the_archives_were_unpacked/apache-hive-1.2.1-bin
        PATH=$PATH:$HIVE_HOME/bin
        ```

    1. Para guardar el archivo, presione **Ctrl+x**, luego **Y** y después Entrar.

1. Cierre y vuelva a abrir la sesión de Bash.

1. Pruebe la conexión. Use el formato de conexión de [A través de puntos de conexión públicos o privados](#over-public-or-private-endpoints), arriba.

## <a name="next-steps"></a>Pasos siguientes

* Para ver ejemplos en los que se usa el cliente de Beeline con Apache Hive, visite [Uso de Apache Beeline con Apache Hive](apache-hadoop-use-hive-beeline.md)
* Para obtener más información general sobre Hive en HDInsight, consulte [Uso de Apache Hive con Apache Hadoop en HDInsight](hdinsight-use-hive.md).
