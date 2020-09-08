---
title: Migración de un almacén HDFS local a Azure Storage con Azure Data Box
description: Migre datos de un almacén HDFS local a Azure Storage (Blob Storage o Data Lake Storage Gen2) mediante un dispositivo Data Box.
author: normesta
ms.service: storage
ms.date: 02/14/2019
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: jamesbak
ms.openlocfilehash: a50f85e76f16f1e5ba8823adb1ea1aa02157fcee
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88032567"
---
# <a name="migrate-from-on-prem-hdfs-store-to-azure-storage-with-azure-data-box"></a>Migración de un almacén HDFS local a Azure Storage con Azure Data Box

Puede migrar datos de un almacén HDFS local de su clúster de Hadoop a Azure Storage (Blob Storage o Data Lake Storage Gen2) mediante un dispositivo Data Box. Dispone de dos opciones en Data Box Disk: Data Box de 80 TB de datos o Data Box Heavy de 770 TB.

En este artículo le ayudamos a completar estas tareas:

> [!div class="checklist"]
> * Prepararse para migrar los datos.
> * Copie los datos en un dispositivo Azure Data Box Disk, Data Box o Data Box Heavy.
> * Devolver el dispositivo a Microsoft.
> * Aplicar permisos de acceso a archivos y directorios (solo Data Lake Storage Gen2)

## <a name="prerequisites"></a>Prerrequisitos

Necesita lo siguiente para completar la migración.

* Una cuenta de Azure Storage.

* Un clúster de Hadoop local que contenga los datos de origen.

* Un [dispositivo Azure Data Box](https://azure.microsoft.com/services/storage/databox/).

  * [Solicite su instancia de Data Box](https://docs.microsoft.com/azure/databox/data-box-deploy-ordered) o [Data Box Heavy](https://docs.microsoft.com/azure/databox/data-box-heavy-deploy-ordered). 

  * Cablee y conecte su instancia de [Data Box](https://docs.microsoft.com/azure/databox/data-box-deploy-set-up) o [Data Box Heavy](https://docs.microsoft.com/azure/databox/data-box-heavy-deploy-set-up) a una red local.

Si ya está listo, empecemos.

## <a name="copy-your-data-to-a-data-box-device"></a>Copia de los datos a un dispositivo Data Box

Si los datos caben en un solo dispositivo Data Box, cópielos en el dispositivo. 

Si el tamaño de los datos supera la capacidad del dispositivo Data Box, use el [procedimiento opcional para dividir los datos entre varios dispositivos Data Box](#appendix-split-data-across-multiple-data-box-devices) y, luego, realice este paso. 

Para copiar los datos del almacén HDFS local a un dispositivo Data Box, deberá configurar algunas cosas y, después, usar la herramienta [DistCp](https://hadoop.apache.org/docs/stable/hadoop-distcp/DistCp.html).

Siga estos pasos para copiar los datos mediante las API REST de Blob Storage o del almacenamiento de objetos en el dispositivo Data Box. La interfaz de la API REST hará que el dispositivo aparezca como un almacén HDFS para el clúster.

1. Antes de copiar los datos a través de REST, identifique las primitivas de seguridad y de conexión para conectarse a la interfaz de REST en la instancia de Data Box o Data Box Heavy. Inicie sesión en la interfaz de usuario web local de Data Box y vaya a la página **Conectar y copiar**. En la cuenta de Azure Storage para su dispositivo, en **Configuración de acceso**, busque **REST** y selecciónelo.

    ![Página Conectar y copiar](media/data-lake-storage-migrate-on-premises-HDFS-cluster/data-box-connect-rest.png)

2. En el cuadro de diálogo Acceder a la cuenta de almacenamiento y cargar datos, copie el valor de **Punto de conexión de Blob service** y la **Clave de cuenta de almacenamiento**. En el punto de conexión de Blob service, omita el `https://` y la barra diagonal final.

    En este caso, el punto de conexión es `https://mystorageaccount.blob.mydataboxno.microsoftdatabox.com/`. La parte del host del URI que va a usar es `mystorageaccount.blob.mydataboxno.microsoftdatabox.com`. Para obtener un ejemplo, vea cómo [conectarse con REST a través de http](/azure/databox/data-box-deploy-copy-data-via-rest). 

     ![Cuadro de diálogo Acceder a la cuenta de almacenamiento y cargar datos](media/data-lake-storage-migrate-on-premises-HDFS-cluster/data-box-connection-string-http.png)

3. Agregue el punto de conexión y la dirección IP del nodo de Data Box o Data Box Heavy a `/etc/hosts` en cada nodo.

    ```    
    10.128.5.42  mystorageaccount.blob.mydataboxno.microsoftdatabox.com
    ```

    Si usa otro mecanismo para DNS, debe asegurarse de que el punto de conexión de Data Box se pueda resolver.

4. Establezca la variable `azjars` del shell en la ubicación de los archivos jar `hadoop-azure` y `azure-storage`. Encontrará estos archivos en el directorio de instalación de Hadoop.

    Para determinar si existen estos archivos, use el siguiente comando: `ls -l $<hadoop_install_dir>/share/hadoop/tools/lib/ | grep azure`. Reemplace el marcador de posición `<hadoop_install_dir>` por la ruta de acceso al directorio en el que ha instalado Hadoop. Asegúrese de usar rutas de acceso completas.

    Ejemplos:

    `azjars=$hadoop_install_dir/share/hadoop/tools/lib/hadoop-azure-2.6.0-cdh5.14.0.jar` `azjars=$azjars,$hadoop_install_dir/share/hadoop/tools/lib/microsoft-windowsazure-storage-sdk-0.6.0.jar`

5. Cree el contenedor de almacenamiento que quiere usar para la copia de los datos. También debe especificar un directorio de destino como parte de este comando. En este momento, podría ser un directorio de destino ficticio.

    ```
    hadoop fs -libjars $azjars \
    -D fs.AbstractFileSystem.wasb.Impl=org.apache.hadoop.fs.azure.Wasb \
    -D fs.azure.account.key.<blob_service_endpoint>=<account_key> \
    -mkdir -p  wasb://<container_name>@<blob_service_endpoint>/<destination_directory>
    ```

    * Reemplace el marcador de posición `<blob_service_endpoint>` por el nombre del punto de conexión de Blob service.

    * Reemplace el marcador de posición `<account_key>` por la clave de acceso de la cuenta.

    * Reemplace el marcador de posición `<container-name>` por el nombre del contenedor.

    * Reemplace el marcador de posición `<destination_directory>` por el nombre del directorio en el que quiere copiar los datos.

6. Ejecute un comando de lista para asegurarse de que se han creado el contenedor y el directorio.

    ```
    hadoop fs -libjars $azjars \
    -D fs.AbstractFileSystem.wasb.Impl=org.apache.hadoop.fs.azure.Wasb \
    -D fs.azure.account.key.<blob_service_endpoint>=<account_key> \
    -ls -R  wasb://<container_name>@<blob_service_endpoint>/
    ```

   * Reemplace el marcador de posición `<blob_service_endpoint>` por el nombre del punto de conexión de Blob service.

   * Reemplace el marcador de posición `<account_key>` por la clave de acceso de la cuenta.

   * Reemplace el marcador de posición `<container-name>` por el nombre del contenedor.

7. Copie los datos de HDFS de Hadoop a Blob Storage de Data Box, en el contenedor que creó anteriormente. Si no se encuentra el directorio en el que se va a copiar, el comando lo crea automáticamente.

    ```
    hadoop distcp \
    -libjars $azjars \
    -D fs.AbstractFileSystem.wasb.Impl=org.apache.hadoop.fs.azure.Wasb \
    -D fs.azure.account.key.<blob_service_endpoint<>=<account_key> \
    -filters <exclusion_filelist_file> \
    [-f filelist_file | /<source_directory> \
           wasb://<container_name>@<blob_service_endpoint>/<destination_directory>
    ```

    * Reemplace el marcador de posición `<blob_service_endpoint>` por el nombre del punto de conexión de Blob service.

    * Reemplace el marcador de posición `<account_key>` por la clave de acceso de la cuenta.

    * Reemplace el marcador de posición `<container-name>` por el nombre del contenedor.

    * Reemplace el marcador de posición `<exlusion_filelist_file>` por el nombre del archivo que contiene la lista de exclusiones de archivos.

    * Reemplace el marcador de posición `<source_directory>` por el nombre del directorio que contiene los datos que quiere copiar.

    * Reemplace el marcador de posición `<destination_directory>` por el nombre del directorio en el que quiere copiar los datos.

    La opción `-libjars` se usa para hacer que el archivo `hadoop-azure*.jar` y el archivo dependiente `azure-storage*.jar` estén disponibles para `distcp`. Esto ya podría ocurrir para algunos clústeres.

    En el ejemplo siguiente se muestra cómo se usa el comando `distcp` para copiar datos.

    ```
     hadoop distcp \
    -libjars $azjars \
    -D fs.AbstractFileSystem.wasb.Impl=org.apache.hadoop.fs.azure.Wasb \
    -D fs.azure.account.key.mystorageaccount.blob.mydataboxno.microsoftdatabox.com=myaccountkey \
    -filter ./exclusions.lst -f /tmp/copylist1 -m 4 \
    /data/testfiles \
    wasb://hdfscontainer@mystorageaccount.blob.mydataboxno.microsoftdatabox.com/data
    ```
  
    Para mejorar la velocidad de copia:

    * Pruebe a cambiar el número de asignadores. (En el ejemplo anterior se usan `m` = 4 asignadores).

    * Pruebe a ejecutar varios `distcp` en paralelo.

    * Recuerde que los archivos grandes tienen un mejor rendimiento que los pequeños.

## <a name="ship-the-data-box-to-microsoft"></a>Envío de Data Box a Microsoft

Siga estos pasos para preparar y enviar el dispositivo Data Box a Microsoft.

1. En primer lugar, [prepárese para enviar Data Box o Data Box Heavy](https://docs.microsoft.com/azure/databox/data-box-deploy-copy-data-via-rest).

2. Una vez completada la preparación del dispositivo, descargue los archivos BOM. Usará estos archivos BOM o de manifiesto más adelante para comprobar los datos cargados en Azure.

3. Apague el dispositivo y quite los cables.

4. Póngase en contacto con UPS para programar la recogida del dispositivo.

    * Para los dispositivos Data Box, vea [Devolución de Data Box](https://docs.microsoft.com/azure/databox/data-box-deploy-picked-up).

    * Para los dispositivos Data Box Heavy, vea [Devolución de Data Box Heavy](https://docs.microsoft.com/azure/databox/data-box-heavy-deploy-picked-up).

5. Cuando Microsoft reciba el dispositivo, se conectará a la red del centro de datos, y los datos se cargarán a la cuenta de almacenamiento que especificó al realizar el pedido de dispositivo. Compruebe con los archivos BOM que todos los datos se han cargado en Azure. 

## <a name="apply-access-permissions-to-files-and-directories-data-lake-storage-gen2-only"></a>Aplicación de permisos de acceso a archivos y directorios (solo Data Lake Storage Gen2)

Ya tiene los datos en su cuenta de Azure Storage. Ahora aplicará los permisos de acceso a archivos y directorios.

> [!NOTE]
> Este paso solo es necesario si usa Azure Data Lake Storage Gen2 como almacén de datos. Si solo usa una cuenta de Blob Storage sin espacio de nombres jerárquico como almacén de datos, puede omitir esta sección.

### <a name="create-a-service-principal-for-your-azure-data-lake-storage-gen2-account"></a>Creación de una entidad de servicio para la cuenta de Azure Data Lake Storage Gen2

Para crear una entidad de servicio, consulte [Procedimientos para el uso del portal para crear una aplicación de Azure AD y una entidad de servicio que puedan acceder a los recursos](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal).

* Al realizar los pasos que se describen en la sección [Asignación de la aplicación a un rol](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#assign-a-role-to-the-application), asegúrese de asignar el rol de **Colaborador de datos de blobs de almacenamiento** a la entidad de servicio.

* Al realizar los pasos de la sección [Obtención de valores para iniciar sesión](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-values-for-signing-in) del artículo, guarde los valores del identificador de aplicación y el secreto de cliente en un archivo de texto. ya que los necesitará pronto.

### <a name="generate-a-list-of-copied-files-with-their-permissions"></a>Generación de una lista de los archivos copiados con sus permisos

En el clúster de Hadoop local, ejecute este comando:

```bash

sudo -u hdfs ./copy-acls.sh -s /{hdfs_path} > ./filelist.json
```

Este comando genera una lista de los archivos copiados con sus permisos.

> [!NOTE]
> En función del número de archivos en HDFS, este comando puede tardar mucho tiempo en ejecutarse.

### <a name="generate-a-list-of-identities-and-map-them-to-azure-active-directory-add-identities"></a>Generación de una lista de identidades y asignación a las identidades de Azure Active Directory (ADD)

1. Descargue el script `copy-acls.py`. Consulte la sección [Descarga de scripts auxiliares y configuración del nodo perimetral para ejecutarlos](#download-helper-scripts) de este artículo.

2. Ejecute este comando para generar una lista de identidades únicas.

   ```bash
   
   ./copy-acls.py -s ./filelist.json -i ./id_map.json -g
   ```

   Este script genera un archivo denominado `id_map.json` que contiene las identidades que necesita para asignarlas a identidades basadas en ADD.

3. Abra el archivo `id_map.json` en un editor de texto.

4. Para cada objeto JSON que aparece en el archivo, actualice con la identidad asignada adecuada el atributo `target` de un nombre principal de usuario (UPN) de AAD o de un ObjectId (OID). Cuando haya acabado, guarde el archivo, ya que lo necesitará en el paso siguiente.

### <a name="apply-permissions-to-copied-files-and-apply-identity-mappings"></a>Aplicación de permisos a los archivos copiados y de asignaciones de identidades

Ejecute este comando para aplicar permisos a los datos que ha copiado en la cuenta de Data Lake Storage Gen2:

```bash
./copy-acls.py -s ./filelist.json -i ./id_map.json  -A <storage-account-name> -C <container-name> --dest-spn-id <application-id>  --dest-spn-secret <client-secret>
```

* Reemplace el marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

* Reemplace el marcador de posición `<container-name>` por el nombre del contenedor.

* Reemplace los marcadores de posición `<application-id>` y `<client-secret>` por el identificador de aplicación y el secreto de cliente que recopiló al crear la entidad de servicio.

## <a name="appendix-split-data-across-multiple-data-box-devices"></a>Apéndice: Dividir datos entre varios dispositivos Data Box

Antes de mover los datos a un dispositivo Data Box, deberá descargar algunos scripts auxiliares, asegurarse de que los datos están organizados de modo que quepan en un dispositivo Data Box y excluir los archivos innecesarios.

<a id="download-helper-scripts"></a>

### <a name="download-helper-scripts-and-set-up-your-edge-node-to-run-them"></a>Descarga de scripts auxiliares y configuración del nodo perimetral para ejecutarlos

1. En el nodo principal o perimetral del clúster de Hadoop local, ejecute este comando:

   ```bash
   
   git clone https://github.com/jamesbak/databox-adls-loader.git
   cd databox-adls-loader
   ```

   Este comando clona el repositorio de GitHub que contiene los scripts auxiliares.

2. Asegúrese de que el paquete [jq](https://stedolan.github.io/jq/) está instalado en el equipo local.

   ```bash
   
   sudo apt-get install jq
   ```

3. Instale el paquete de Python [Requests](https://2.python-requests.org/en/master/).

   ```bash
   
   pip install requests
   ```

4. Establezca permisos de ejecución en los scripts necesarios.

   ```bash
   
   chmod +x *.py *.sh

   ```

### <a name="ensure-that-your-data-is-organized-to-fit-onto-a-data-box-device"></a>Comprobación de que los datos están organizados de modo que quepan en un dispositivo Data Box

Si el tamaño de los datos supera el tamaño de un dispositivo Data Box, tiene la opción de dividir los archivos en grupos que se pueden almacenar en varios dispositivos.

Si los datos no superan el tamaño de un dispositivo Data Box, puede pasar a la sección siguiente.

1. Con permisos elevados, ejecute el script `generate-file-list` que descargó con las instrucciones en la sección anterior.

   Esta es una descripción de los parámetros del comando:

   ```
   sudo -u hdfs ./generate-file-list.py [-h] [-s DATABOX_SIZE] [-b FILELIST_BASENAME]
                    [-f LOG_CONFIG] [-l LOG_FILE]
                    [-v {DEBUG,INFO,WARNING,ERROR}]
                    path

   where:
   positional arguments:
   path                  The base HDFS path to process.

   optional arguments:
   -h, --help            show this help message and exit
   -s DATABOX_SIZE, --databox-size DATABOX_SIZE
                        The size of each Data Box in bytes.
   -b FILELIST_BASENAME, --filelist-basename FILELIST_BASENAME
                        The base name for the output filelists. Lists will be
                        named basename1, basename2, ... .
   -f LOG_CONFIG, --log-config LOG_CONFIG
                        The name of a configuration file for logging.
   -l LOG_FILE, --log-file LOG_FILE
                        Name of file to have log output written to (default is
                        stdout/stderr)
   -v {DEBUG,INFO,WARNING,ERROR}, --log-level {DEBUG,INFO,WARNING,ERROR}
                        Level of log information to output. Default is 'INFO'.
   ```

2. Copie las listas de archivos generadas en HDFS para que estén accesibles para el trabajo [DistCp](https://hadoop.apache.org/docs/stable/hadoop-distcp/DistCp.html).

   ```
   hadoop fs -copyFromLocal {filelist_pattern} /[hdfs directory]
   ```

### <a name="exclude-unnecessary-files"></a>Exclusión de archivos innecesarios

Tendrá que excluir algunos directorios del trabajo DisCp. Por ejemplo, excluya los directorios con información de estado que mantienen el clúster en ejecución.

En el clúster de Hadoop local donde va a iniciar el trabajo DistCp, cree un archivo que especifique la lista de directorios que quiere excluir.

Este es un ejemplo:

```
.*ranger/audit.*
.*/hbase/data/WALs.*
```

## <a name="next-steps"></a>Pasos siguientes

Aprenda cómo funciona Data Lake Storage Gen2 con clústeres de HDInsight. Consulte [Uso de Data Lake Storage Gen2 con clústeres de Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md).
