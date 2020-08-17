---
title: Supervisión y administración de clústeres de Hadoop mediante la API REST de Ambari en Azure HDInsight
description: Aprenda a usar Ambari para supervisar y administrar clústeres de Hadoop en Azure HDInsight. En este documento, aprenderá a usar la API REST de Ambari incluida con clústeres de HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/29/2020
ms.openlocfilehash: 45b9c158aca85d62b02d65282876d5e40129878f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081073"
---
# <a name="manage-hdinsight-clusters-by-using-the-apache-ambari-rest-api"></a>Administración de clústeres de HDInsight mediante la API REST de Apache Ambari

[!INCLUDE [ambari-selector](../../includes/hdinsight-ambari-selector.md)]

Aprenda a usar la API REST de Apache Ambari para administrar y supervisar clústeres de Apache Hadoop en Azure HDInsight.

## <a name="what-is-apache-ambari"></a>¿Qué es Apache Ambari?

Apache Ambari simplifica la administración y la supervisión de los clústeres de Hadoop al proporcionar una interfaz de usuario web fácil de usar y respaldada por sus [API REST](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md).  De manera predeterminada, Ambari viene con los clústeres de HDInsight basado en Linux.

## <a name="prerequisites"></a>Prerrequisitos

* Un clúster de Hadoop en HDInsight. Consulte [Introducción a HDInsight en Linux](hadoop/apache-hadoop-linux-tutorial-get-started.md).

* Bash en Ubuntu en Windows 10.  Los ejemplos de este artículo usan el shell de Bash en Windows 10. Consulte la [Guía de instalación del subsistema de Windows para Linux para Windows 10](https://docs.microsoft.com/windows/wsl/install-win10) para conocer los pasos de instalación.  Otros [shells de Unix](https://www.gnu.org/software/bash/) también funcionarán.  Los ejemplos, con algunas pequeñas modificaciones, pueden funcionar en un símbolo del sistema de Windows.  También puede usar Windows PowerShell.

* jq, un procesador JSON de línea de comandos.  Vea [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/).

* Windows PowerShell.  También puede usar Bash.

## <a name="base-uniform-resource-identifier-for-ambari-rest-api"></a>Identificador uniforme de recursos base para la API REST de Ambari

 El identificador uniforme de recursos (URI) base de la API REST de Ambari en HDInsight es `https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME`, donde `CLUSTERNAME` es el nombre del clúster.  Los nombres de clúster en los URI **distinguen mayúsculas de minúsculas**.  Si bien el nombre del clúster en la parte del nombre de dominio completo (FQDN) del URI (`CLUSTERNAME.azurehdinsight.net`) no distingue entre mayúsculas y minúsculas, otras apariciones en el identificador URI sí lo hacen.

## <a name="authentication"></a>Authentication

La conexión a Ambari en HDInsight requiere HTTPS. Utilice el nombre de la cuenta de administrador (el valor predeterminado es **admin**) y la contraseña que proporcionó durante la creación del clúster.

En el caso de los clústeres de Enterprise Security Package, en lugar de `admin`, use un nombre de usuario completo, como `username@domain.onmicrosoft.com`.

## <a name="examples"></a>Ejemplos

### <a name="setup-preserve-credentials"></a>Configuración (conservar las credenciales)

Conserve las credenciales para evitar volver a escribirlas en cada ejemplo.  El nombre del clúster se conservará en un paso independiente.

**A. Bash**  
Edite el siguiente script, reemplazando `PASSWORD` por su contraseña real.  Luego, escriba el comando.

```bash
export password='PASSWORD'
```  

**B. PowerShell**  

```powershell
$creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
```

### <a name="identify-correctly-cased-cluster-name"></a>Identificación del nombre del clúster con las mayúsculas y minúsculas correctas

El uso de mayúsculas o minúsculas en el nombre del clúster puede ser diferente del esperado.  Estos pasos mostrarán las mayúsculas y minúsculas reales y después las almacenarán en una variable para todos los ejemplos posteriores.

Edite los siguientes scripts para reemplazar `CLUSTERNAME` por su nombre de clúster. Luego, escriba el comando. (El nombre del clúster para el nombre de dominio completo no distingue mayúsculas de minúsculas).

```bash
export clusterName=$(curl -u admin:$password -sS -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')
echo $clusterName
```  

```powershell
# Identify properly cased cluster name
$resp = Invoke-WebRequest -Uri "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters" `
    -Credential $creds -UseBasicParsing
$clusterName = (ConvertFrom-Json $resp.Content).items.Clusters.cluster_name;

# Show cluster name
$clusterName
```

### <a name="parsing-json-data"></a>Análisis de datos JSON

En el ejemplo siguiente se utiliza [jq](https://stedolan.github.io/jq/) o [ConvertFrom-Json](https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/convertfrom-json) para analizar el documento de respuesta de JSON y mostrar solo la información de `health_report` de los resultados.

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName" \
| jq '.Clusters.health_report'
```  

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.Clusters.health_report
```

### <a name="get-the-fqdn-of-cluster-nodes"></a>obtener el FQDN de los nodos de clúster

Es posible que tenga que conocer el nombre de dominio completo (FQDN) del nodo de un clúster. Puede recuperar fácilmente el FQDN de varios nodos del clúster mediante los siguientes ejemplos:

**Todos los nodos**  

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts" \
| jq -r '.items[].Hosts.host_name'
```  

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.items.Hosts.host_name
```

**Nodos principales**  

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/HDFS/components/NAMENODE" \
| jq -r '.host_components[].HostRoles.host_name'
```

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/HDFS/components/NAMENODE" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.host_components.HostRoles.host_name
```

**Nodos de trabajo**  

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/HDFS/components/DATANODE" \
| jq -r '.host_components[].HostRoles.host_name'
```

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/HDFS/components/DATANODE" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.host_components.HostRoles.host_name
```

**Nodos de Zookeeper**

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" \
| jq -r ".host_components[].HostRoles.host_name"
```

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.host_components.HostRoles.host_name
```

### <a name="get-the-internal-ip-address-of-cluster-nodes"></a>obtener la dirección IP interna de los nodos de clúster

No se puede acceder directamente mediante Internet a las direcciones IP devueltas por los ejemplos de esta sección. El acceso a ellas solo es posible dentro de la instancia de Azure Virtual Network que contiene el clúster de HDInsight.

Para más información acerca del uso con HDInsight y redes virtuales, consulte [Planeamiento de una red virtual para HDInsight](hdinsight-plan-virtual-network-deployment.md).

Para buscar la dirección IP, debe conocer el nombre de dominio completo (FQDN) interno de los nodos del clúster. Una vez que tenga el FQDN, ya puede obtener la dirección IP del host. En los ejemplos siguientes primero se consulta en Ambari el FQDN de todos los nodos de host. La siguiente consulta que se realiza en Ambari es la dirección IP de cada host.

```bash
for HOSTNAME in $(curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts" | jq -r '.items[].Hosts.host_name')
do
    IP=$(curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts/$HOSTNAME" | jq -r '.Hosts.ip')
  echo "$HOSTNAME <--> $IP"
done
```  

```powershell
$uri = "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts" 
$resp = Invoke-WebRequest -Uri $uri -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
foreach($item in $respObj.items) {
    $hostName = [string]$item.Hosts.host_name
    $hostInfoResp = Invoke-WebRequest -Uri "$uri/$hostName" `
        -Credential $creds -UseBasicParsing
    $hostInfoObj = ConvertFrom-Json $hostInfoResp
    $hostIp = $hostInfoObj.Hosts.ip
    "$hostName <--> $hostIp"
}
```

### <a name="get-the-default-storage"></a>obtener el almacenamiento predeterminado

Los clústeres de HDInsight tienen que usar una cuenta de Azure Storage o Data Lake Storage como almacenamiento predeterminado. Puede usar Ambari para recuperar esta información una vez creado el clúster. Por ejemplo, si desea leer y escribir datos en el contenedor fuera de HDInsight.

En los ejemplos siguientes se recupera la configuración de almacenamiento predeterminada del clúster:

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" \
| jq -r '.items[].configurations[].properties["fs.defaultFS"] | select(. != null)'
```

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.items.configurations.properties.'fs.defaultFS'
```

> [!IMPORTANT]  
> Estos ejemplos devuelven la primera configuración aplicada al servidor (`service_config_version=1`), que contiene esta información. Si recupera un valor que se modificó después de crear el clúster, es posible que deba enumerar las versiones de configuración y recuperar la más reciente.

El valor devuelto es similar a uno de los ejemplos siguientes:

* `wasbs://CONTAINER@ACCOUNTNAME.blob.core.windows.net`: este valor indica que el clúster usa una cuenta de Azure Storage para el almacenamiento predeterminado. El valor `ACCOUNTNAME` es el nombre de la cuenta de almacenamiento. La parte `CONTAINER` es el nombre del contenedor de blobs de la cuenta de almacenamiento. El contenedor es la raíz del almacenamiento HDFS compatible del clúster.

* `abfs://CONTAINER@ACCOUNTNAME.dfs.core.windows.net`: este valor indica que el clúster usa Azure Data Lake Storage Gen2 como almacenamiento predeterminado. Los valores `ACCOUNTNAME` y `CONTAINER` tienen el mismo significado que para Azure Storage indicado anteriormente.

* `adl://home`: este valor indica que el clúster usa Azure Data Lake Storage Gen1 como almacenamiento predeterminado.

    Para buscar el nombre de cuenta de Data Lake Storage, utilice los siguientes ejemplos:

    ```bash
    curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" \
    | jq -r '.items[].configurations[].properties["dfs.adls.home.hostname"] | select(. != null)'
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" `
        -Credential $creds -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.items.configurations.properties.'dfs.adls.home.hostname'
    ```

    El valor devuelto es similar a `ACCOUNTNAME.azuredatalakestore.net`, donde `ACCOUNTNAME` es el nombre de la cuenta de Data Lake Storage.

    Para encontrar el directorio en Data Lake Storage que contiene el almacenamiento para el clúster, use los siguientes ejemplos:

    ```bash
    curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" \
    | jq -r '.items[].configurations[].properties["dfs.adls.home.mountpoint"] | select(. != null)'
    ```  

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" `
        -Credential $creds -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.items.configurations.properties.'dfs.adls.home.mountpoint'
    ```

    El valor devuelto es similar a `/clusters/CLUSTERNAME/`. Este valor es una ruta de acceso dentro de la cuenta de Data Lake Storage. Esta ruta de acceso es la raíz del sistema de archivos HDFS compatible para el clúster.  

> [!NOTE]  
> El cmdlet [Get-AzHDInsightCluster](https://docs.microsoft.com/powershell/module/az.hdinsight/get-azhdinsightcluster) proporcionado por [Azure PowerShell](/powershell/azure/) también devuelve la información de almacenamiento para el clúster.

### <a name="get-all-configurations"></a>Obtención de todas las configuraciones

Obtenga las configuraciones que están disponibles para el clúster.

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName?fields=Clusters/desired_configs"
```

```powershell
$respObj = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName`?fields=Clusters/desired_configs" `
    -Credential $creds -UseBasicParsing
$respObj.Content
```

Este ejemplo devuelve un documento JSON que contiene la configuración actual de los componentes instalados. Vea el valor *etiqueta*. Por ejemplo, el siguiente es un extracto de los datos devueltos de un tipo de clúster Spark.

```json
"jupyter-site" : {
  "tag" : "INITIAL",
  "version" : 1
},
"livy2-client-conf" : {
  "tag" : "INITIAL",
  "version" : 1
},
"livy2-conf" : {
  "tag" : "INITIAL",
  "version" : 1
},
```

### <a name="get-configuration-for-specific-component"></a>Obtención de la configuración del componente específico

Obtenga la configuración del componente en el que está interesado. En el ejemplo siguiente, reemplace `INITIAL` por el valor de etiqueta devuelto por la solicitud anterior.

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations?type=livy2-conf&tag=INITIAL"
```

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations?type=livy2-conf&tag=INITIAL" `
    -Credential $creds -UseBasicParsing
$resp.Content
```

Este ejemplo devuelve un documento JSON que contiene la configuración actual del componente `livy2-conf`.

### <a name="update-configuration"></a>actualizar la configuración

1. Cree `newconfig.json`.  
   Modifique y, a continuación, escriba los comandos siguientes:

   * Reemplace `livy2-conf` por el nuevo componente.
   * Reemplace `INITIAL` por el valor real recuperado para `tag` en [Obtención de todas las configuraciones](#get-all-configurations).

     **A. Bash**

     ```bash
     curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations?type=livy2-conf&tag=INITIAL" \
     | jq --arg newtag $(echo version$(date +%s%N)) '.items[] | del(.href, .version, .Config) | .tag |= $newtag | {"Clusters": {"desired_config": .}}' > newconfig.json
     ```

     **B. PowerShell**  
     El script de PowerShell usa [jq](https://stedolan.github.io/jq/).  Edite la ruta de acceso `C:\HD\jq\jq-win64` siguiente para indicar la ruta de acceso real y la versión de [jq](https://stedolan.github.io/jq/).

     ```powershell
     $epoch = Get-Date -Year 1970 -Month 1 -Day 1 -Hour 0 -Minute 0 -Second 0
     $now = Get-Date
     $unixTimeStamp = [math]::truncate($now.ToUniversalTime().Subtract($epoch).TotalMilliSeconds)
     $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations?type=livy2-conf&tag=INITIAL" `
       -Credential $creds -UseBasicParsing
     $resp.Content | C:\HD\jq\jq-win64 --arg newtag "version$unixTimeStamp" '.items[] | del(.href, .version, .Config) | .tag |= $newtag | {"Clusters": {"desired_config": .}}' > newconfig.json
     ```

     Jq se utiliza para convertir los datos recuperados de HDInsight en una nueva plantilla de configuración. En concreto, estos ejemplos realizan las siguientes acciones:

   * Crea un valor único que contiene la cadena "version" y la fecha, que se almacena en `newtag`.

   * Crea un documento raíz para la nueva configuración.

   * Obtiene el contenido de la matriz `.items[]` y lo agrega al elemento **desired_config**.

   * Elimina los elementos `href`, `version` y `Config`, porque no son necesarios para enviar una nueva configuración.

   * Agrega un nuevo elemento `tag` con un valor de `version#################`. La parte numérica se basa en la fecha actual. Cada configuración debe tener una etiqueta única.

     Por último, los datos se guardan en el documento `newconfig.json`. La estructura del documento será similar a la del siguiente ejemplo:

     ```json
     {
       "Clusters": {
         "desired_config": {
           "tag": "version1552064778014",
           "type": "livy2-conf",
           "properties": {
             "livy.environment": "production",
             "livy.impersonation.enabled": "true",
             "livy.repl.enableHiveContext": "true",
             "livy.server.csrf_protection.enabled": "true",
               ....
           },
         },
       }
     }
     ```

2. Edite `newconfig.json`.  
   Abra el documento `newconfig.json` y modifique o agregue valores en el objeto `properties`. En el ejemplo siguiente se cambia el valor `"livy.server.csrf_protection.enabled"` de `"true"` a `"false"`.

    ```json
    "livy.server.csrf_protection.enabled": "false",
    ```

    Cuando haya terminado de realizar modificaciones, guarde el archivo.

3. Envíe `newconfig.json`.  
   Use los comandos siguientes para enviar la configuración actualizada a Ambari.

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" -X PUT -d @newconfig.json "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName"
    ```

    ```powershell
    $newConfig = Get-Content .\newconfig.json
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName" `
        -Credential $creds -UseBasicParsing `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body $newConfig
    $resp.Content
    ```  

    Estos comandos envían el contenido del archivo **newconfig.json** al clúster como nueva configuración. La solicitud devuelve un documento JSON. El elemento **versionTag** de este documento debería coincidir con la versión que envió y el objeto **configs** contiene los cambios de configuración que solicitó.

### <a name="restart-a-service-component"></a>reiniciar un componente de servicio

En este momento, la IU web de Ambari indica que el servicio Spark tiene que reiniciarse para que la nueva configuración surta efecto. Siga los siguientes pasos para reiniciar el servicio.

1. Use los siguientes comandos para habilitar el modo de mantenimiento para el servicio Spark2:

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" \
    -X PUT -d '{"RequestInfo": {"context": "turning on maintenance mode for SPARK2"},"Body": {"ServiceInfo": {"maintenance_state":"ON"}}}' \
    "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2"
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" `
        -Credential $creds -UseBasicParsing `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body '{"RequestInfo": {"context": "turning on maintenance mode for SPARK2"},"Body": {"ServiceInfo": {"maintenance_state":"ON"}}}'
    ```

2. Verifique el modo de mantenimiento.  

    Estos comandos envían un documento JSON al servidor que activa el modo de mantenimiento. Para comprobar que el servicio ahora está en modo de mantenimiento, puede usar la siguiente solicitud:

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" \
    "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" \
    | jq .ServiceInfo.maintenance_state
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" `
        -Credential $creds -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.ServiceInfo.maintenance_state
    ```

    El valor devuelto es `ON`.

3. A continuación, use los siguientes comandos para desactivar el servicio Spark2:

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" \
    -X PUT -d '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK2","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"INSTALLED"}}}' \
    "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2"
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" `
        -Credential $creds -UseBasicParsing `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK2","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"INSTALLED"}}}'
    $resp.Content
    ```

    La respuesta es similar al siguiente ejemplo:

    ```json
    {
        "href" : "http://10.0.0.18:8080/api/v1/clusters/CLUSTERNAME/requests/29",
        "Requests" : {
            "id" : 29,
            "status" : "Accepted"
        }
    }
    ```

    > [!IMPORTANT]  
    > El valor `href` devuelto para este identificador URI está usando la dirección IP interna del nodo de clúster. Para usarlo desde fuera del clúster, reemplace la parte `10.0.0.18:8080` con el FQDN del clúster.  

4. Compruebe la solicitud.  
    Edite el comando siguiente reemplazando `29` por el valor real de `id` devuelto en el paso anterior.  Los comandos siguientes recuperan el estado de la solicitud:

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" \
    "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/requests/29" \
    | jq .Requests.request_status
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/requests/29" `
        -Credential $creds -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.Requests.request_status
    ```

    Una respuesta de `COMPLETED` indica que la solicitud ha finalizado.

5. Una vez finalizada la solicitud anterior, use lo siguiente para iniciar el servicio Spark2.

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" \
    -X PUT -d '{"RequestInfo":{"context":"_PARSE_.START.SPARK2","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"STARTED"}}}' \
    "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2"
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" `
        -Credential $creds -UseBasicParsing `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body '{"RequestInfo":{"context":"_PARSE_.START.SPARK2","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"STARTED"}}}'
    $resp.Content
    ```

    El servicio ya usa la nueva configuración.

6. Por último, use lo siguiente para desactivar el modo de mantenimiento.

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" \
    -X PUT -d '{"RequestInfo": {"context": "turning off maintenance mode for SPARK2"},"Body": {"ServiceInfo": {"maintenance_state":"OFF"}}}' \
    "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2"
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" `
        -Credential $creds -UseBasicParsing `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body '{"RequestInfo": {"context": "turning off maintenance mode for SPARK2"},"Body": {"ServiceInfo": {"maintenance_state":"OFF"}}}'
    ```

## <a name="next-steps"></a>Pasos siguientes

Para obtener una referencia completa de la API REST, consulte [Referencia de Apache Ambari API V1](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md).  Consulte también [Autorización de usuarios para Apache Ambari Views](./hdinsight-authorize-users-to-ambari.md)
