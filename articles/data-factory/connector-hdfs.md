---
title: Copia de datos de HDFS mediante Azure Data Factory
description: Obtenga información sobre cómo copiar datos desde un origen HDFS en la nube o en un entorno local a almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/06/2020
ms.author: jingwang
ms.openlocfilehash: 43ab59f109e311d9d7312b77d34321fa98a952d6
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926814"
---
# <a name="copy-data-from-the-hdfs-server-by-using-azure-data-factory"></a>Copia de datos desde un servidor HDFS mediante Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que está usando:"]
> * [Versión 1](v1/data-factory-hdfs-connector.md)
> * [Versión actual](connector-hdfs.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se describe cómo copiar datos desde el servidor del sistema de archivos distribuido de Hadoop (HDFS). Para información sobre Azure Data Factory, lea el [artículo de introducción](introduction.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

El conector HDFS es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen y receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

En concreto, el conector HDFS admite las siguientes funcionalidades:

- Copiar los archivos mediante la autenticación de *Windows* (Kerberos) o *anónima*.
- Copiar los archivos mediante el protocolo *webhdfs* o la compatibilidad con *DistCp integrada*.
- La copia de archivos tal cual, o bien el análisis o generación de archivos con los [códecs de compresión y los formatos de archivo compatibles](supported-file-formats-and-compression-codecs.md).

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

> [!NOTE]
> Asegúrese de que el entorno de ejecución de integración puede acceder a *todos* los [servidor de nodo de nombres]:[puerto de nodo de nombres] y [servidores de nodos de datos]:[puerto de nodo de datos] del clúster de Hadoop. El [puerto de nodo de nombres] predeterminado es 50070 y el [puerto de nodo de datos] es 50075.

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas de HDFS.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de HDFS:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad *type* debe establecerse en *Hdfs*. | Sí |
| url |Dirección URL a HDFS |Sí |
| authenticationType | Los valores permitidos son *Anónima* o *Windows*. <br><br> Para configurar el entorno local, consulte la sección [Uso de la autenticación Kerberos para el conector HDFS](#use-kerberos-authentication-for-the-hdfs-connector). |Sí |
| userName |Nombre de usuario para la autenticación de Windows. Para la autenticación Kerberos, especifique **\<username>@\<domain>.com**. |Sí (para la autenticación de Windows) |
| password |Contraseña para la autenticación de Windows. Marque este campo como SecureString para almacenarlo de forma segura en la factoría de datos, o bien [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). |Sí (para la autenticación de Windows) |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Para más información, consulte la sección [Requisitos previos](#prerequisites). Si no se especifica el entorno de ejecución de integración, el servicio usa la instancia predeterminada de Azure Integration Runtime. |No |

**Ejemplo: Uso de autenticación anónima**

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "authenticationType": "Anonymous",
            "userName": "hadoop"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo: Uso de autenticación de Windows**

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "authenticationType": "Windows",
            "userName": "<username>@<domain>.com (for Kerberos auth)",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si quiere ver una lista completa de las secciones y propiedades que están disponibles para definir conjuntos de datos, consulte [Conjuntos de datos en Azure Data Factory](concepts-datasets-linked-services.md). 

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Las propiedades siguientes se admiten para HDFS en la configuración `location` del conjunto de datos basado en formato:

| Propiedad   | Descripción                                                  | Obligatorio |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | La propiedad *type* de `location` del conjunto de datos se debe establecer en *HdfsLocation*. | Sí      |
| folderPath | Ruta de acceso a la carpeta. Si quiere usar un carácter comodín para filtrar la carpeta, omita este valor y especifique la ruta de acceso en la configuración del origen de actividad. | No       |
| fileName   | Nombre de archivo en la propiedad folderPath especificada. Si quiere usar un carácter comodín para filtrar archivos, omita este valor y especifique el nombre de archivo en la configuración del origen de actividad. | No       |

**Ejemplo**:

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<HDFS linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "HdfsLocation",
                "folderPath": "root/folder/subfolder"
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Para ver una lista completa de las secciones y propiedades que están disponibles para definir actividades, consulte [Canalizaciones y actividades en Azure Data Factory](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que el origen HDFS admite.

### <a name="hdfs-as-source"></a>HDFS como origen

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Las propiedades siguientes se admiten para HDFS en la configuración `storeSettings` del origen de copia basado en formato:

| Propiedad                 | Descripción                                                  | Obligatorio                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | La propiedad *type* de `storeSettings` se debe establecer en **HdfsReadSettings**. | Sí                                           |
| ***Buscar los archivos para copiar*** |  |  |
| OPCIÓN 1: ruta de acceso estática<br> | Copia de la ruta de acceso de archivo o carpeta especificada en el conjunto de datos. Si quiere copiar todos los archivos de una carpeta, especifique también `wildcardFileName` como `*`. |  |
| OPCIÓN 2: carácter comodín<br>- wildcardFolderPath | Ruta de acceso de carpeta con caracteres comodín para filtrar las carpetas de origen. <br>Los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter). Use `^` como escape si el nombre real de la carpeta contiene un carácter comodín o este carácter de escape. <br>Para ver más ejemplos, consulte [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | No                                            |
| OPCIÓN 2: carácter comodín<br>- wildcardFileName | Nombre de archivo con caracteres comodín en la propiedad folderPath o wildcardFolderPath especificada para filtrar los archivos de origen. <br>Los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter); use `^` para el escape si el nombre real de la carpeta tiene un carácter comodín o este carácter de escape dentro.  Para ver más ejemplos, consulte [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | Sí |
| OPCIÓN 3: una lista de archivos<br>- fileListPath | Indica que se copie un conjunto de archivos especificado. Apunte a un archivo de texto que incluya una lista de archivos que quiera copiar (un archivo por línea, con la ruta de acceso relativa a la ruta de acceso configurada en el conjunto de datos).<br/>Al usar esta opción, no especifique un nombre de archivo en el conjunto de datos. Para ver más ejemplos, consulte [Ejemplos de lista de archivos](#file-list-examples). |No |
| ***Configuración adicional*** |  | |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Cuando `recursive` se establece en *true* y el receptor es un almacén basado en archivos, no se copia ni crea ninguna carpeta o subcarpeta vacías en el receptor. <br>Los valores permitidos son: *True* (valor predeterminado) y *False*.<br>Esta propiedad no se aplica al configurar `fileListPath`. |No |
| modifiedDatetimeStart    | Los archivos se filtran en función del atributo *Last Modified*. <br>Los archivos se seleccionan si la hora de su última modificación está dentro del intervalo de `modifiedDatetimeStart` a `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato *2018-12-01T05:00:00Z*. <br> Las propiedades pueden ser NULL, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.<br/>Esta propiedad no se aplica al configurar `fileListPath`. | No                                            |
| maxConcurrentConnections | Número de conexiones que se pueden conectar al almacén de almacenamiento de forma simultánea. Especifique un valor solamente cuando desee limitar la conexión simultánea al almacén de datos. | No                                            |
| ***Configuración de DistCp*** |  | |
| distcpSettings | Grupo de propiedades que se va a usar al utilizar HDFS DistCp. | No |
| resourceManagerEndpoint | Punto de conexión de YARN (Yet Another Resource Negotiator) | Sí, si se utiliza DistCp |
| tempScriptPath | Ruta de acceso de carpeta que se usa para almacenar el script del comando DistCp temporal. Data Factory se encarga de crear el archivo de script que se eliminará después de que haya finalizado el trabajo de copia. | Sí, si se utiliza DistCp |
| distcpOptions | Opciones adicionales que se proporcionan al comando DistCp. | No |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromHDFS",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delimited text input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "DelimitedTextSource",
                "formatSettings":{
                    "type": "DelimitedTextReadSettings",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "HdfsReadSettings",
                    "recursive": true,
                    "distcpSettings": {
                        "resourceManagerEndpoint": "resourcemanagerendpoint:8088",
                        "tempScriptPath": "/usr/hadoop/tempscript",
                        "distcpOptions": "-m 100"
                    }
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>Ejemplos de filtros de carpetas y archivos

En esta sección se describe el comportamiento resultante si se usa un filtro de carácter comodín con la ruta de acceso de carpeta y el nombre de archivo.

| folderPath | fileName             | recursive | Resultado de estructura de carpeta de origen y filtro (se recuperan los archivos en **negrita**) |
| :--------- | :------------------- | :-------- | :----------------------------------------------------------- |
| `Folder*`  | (vacío, usar el valor predeterminado) | false     | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*`  | (vacío, usar el valor predeterminado) | true      | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*`  | `*.csv`              | false     | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*`  | `*.csv`              | true      | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |

### <a name="file-list-examples"></a>Ejemplos de lista de archivos

En esta sección se describe el comportamiento resultante de usar una ruta de acceso de la lista de archivos en el origen de la actividad de copia. Se asume que tiene la siguiente estructura de carpetas de origen y quiere copiar los archivos que están en negrita:

| Estructura de origen de ejemplo                                      | Contenido de FileListToCopy.txt                             | Configuración de Azure Data Factory                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| root<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Metadatos<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | File1.csv<br>Subfolder1/File3.csv<br>Subfolder1/File5.csv | **El conjunto de datos:**<br>- Ruta de acceso de la carpeta: `root/FolderA`<br><br>**En el origen de la actividad de copia:**<br>- Ruta de acceso de la lista de archivos: `root/Metadata/FileListToCopy.txt` <br><br>La ruta de acceso de la lista de archivos apunta a un archivo de texto en el mismo almacén de datos que incluye una lista de archivos que se quiere copiar (un archivo por línea, con la ruta de acceso relativa a la ruta de acceso configurada en el conjunto de datos). |

## <a name="use-distcp-to-copy-data-from-hdfs"></a>Uso de DistCp para copiar datos desde HDFS

[DistCp](https://hadoop.apache.org/docs/current3/hadoop-distcp/DistCp.html) es una herramienta de línea de comandos nativa de Hadoop para realizar una copia distribuida en un clúster de Hadoop. Al ejecutar un comando en DistCp, dicho comando primero enumera todos los archivos que se van a copiar y luego crea varios trabajos de asignación en el clúster de Hadoop. Cada trabajo de asignación realiza una copia binaria desde el origen al receptor.

La actividad de copia admite el uso de DistCp para copiar archivos tal cual en Azure Blob Storage (incluida la [copia almacenada provisionalmente](copy-activity-performance.md)) o en Azure Data Lake Store. En este caso, DistCp puede aprovechar la versatilidad del clúster en lugar de ejecutarse en el entorno de ejecución de integración autohospedado. El uso de DistCp proporciona el mejor rendimiento de la copia, sobre todo si el clúster es muy eficaz. En función de la configuración de la factoría de datos, la actividad de copia crea automáticamente un comando DistCp, lo envía a un clúster de Hadoop y supervisa el estado de la copia.

### <a name="prerequisites"></a>Requisitos previos

Para usar DistCp con el fin de copiar los archivos tal cual desde HDFS a Azure Blob (incluida la copia almacenada provisionalmente) o Azure Data Lake Store, asegúrese de que el clúster de Hadoop cumpla los siguientes requisitos:

* Los servicios de MapReduce y YARN están habilitados.  
* La versión de YARN es la 2.5 o una posterior.  
* El servidor HDFS se integra con el almacén de datos de destino: **Azure Blob Storage** o **Azure Data Lake Store (ADLS Gen1)** : 

    - El sistema de archivos de Azure Blob se admite de forma nativa en Hadoop 2.7 o superior. Solo necesita especificar la ruta de acceso de JAR en la configuración del entorno de Hadoop.
    - El sistema de archivos de Azure Data Lake Store se empaqueta a partir de Hadoop 3.0.0-alpha1. Si la versión del clúster de Hadoop es anterior a esa versión, debe importar manualmente los paquetes JAR relacionados con Azure Data Lake Store (azure-datalake-store.jar) en el clúster desde [aquí](https://hadoop.apache.org/releases.html) y especificar la ruta de acceso del archivo JAR en la configuración del entorno de Hadoop.

* Prepare una carpeta temporal en HDFS. Esta carpeta temporal se usa para almacenar un script de shell de DistCp, por lo que ocupa varios kilobytes de espacio.
* Asegúrese de que la cuenta de usuario que se proporciona en el servicio vinculado de HDFS tiene permiso para:
   * Enviar una aplicación en YARN.
   * Crear una subcarpeta y archivos de lectura y escritura en la carpeta temporal.

### <a name="configurations"></a>Configurations

Para configuraciones y ejemplos relacionados con DistCp, vaya a la sección [HDFS como origen](#hdfs-as-source).

## <a name="use-kerberos-authentication-for-the-hdfs-connector"></a>Uso de la autenticación Kerberos para el conector HDFS

Existen dos opciones para configurar el entorno local para usar la autenticación Kerberos para el conector HDFS. Puede elegir la que mejor se adapte a su situación.
* Opción 1: [Unir una máquina del entorno de ejecución de integración autohospedado en el dominio Kerberos](#kerberos-join-realm)
* Opción 2: [Habilitar la confianza mutua entre el dominio de Windows y el dominio Kerberos](#kerberos-mutual-trust)

### <a name="option-1-join-a-self-hosted-integration-runtime-machine-in-the-kerberos-realm"></a><a name="kerberos-join-realm"></a>Opción 1: Unir una máquina del entorno de ejecución de integración autohospedado en el dominio Kerberos

#### <a name="requirements"></a>Requisitos

* La máquina del entorno de ejecución de integración autohospedado debe unirse al dominio Kerberos y no a ninguno de Windows.

#### <a name="how-to-configure"></a>Cómo se configura

**En la máquina del entorno de ejecución de integración autohospedado:**

1.  Ejecute la utilidad Ksetup para configurar el dominio y el servidor del Centro de distribución de claves (KDC) de Kerberos.

    La máquina debe configurarse como miembro de un grupo de trabajo, porque un dominio Kerberos es diferente a un dominio de Windows. Para lograr esta configuración, establezca el dominio Kerberos y agregue un servidor KDC mediante la ejecución de los siguientes comandos. Reemplace *REALM.COM* por su propio nombre de dominio.

    ```console
    C:> Ksetup /setdomain REALM.COM
    C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
    ```

    Después de ejecutar estos comandos, reinicie el equipo.

2.  Compruebe la configuración con el comando `Ksetup`. La salida debe ser como la siguiente:

    ```output
    C:> Ksetup
    default realm = REALM.COM (external)
    REALM.com:
        kdc = <your_kdc_server_address>
    ```

**En la factoría de datos:**

* Configure el conector HDFS mediante la autenticación de Windows junto con el nombre y la contraseña de la entidad de seguridad de Kerberos para conectarse al origen de datos de HDFS. Para conocer los detalles de la configuración, consulte la sección [Propiedades del servicio vinculado](#linked-service-properties).

### <a name="option-2-enable-mutual-trust-between-the-windows-domain-and-the-kerberos-realm"></a><a name="kerberos-mutual-trust"></a>Opción 2: Habilitar la confianza mutua entre el dominio de Windows y el dominio Kerberos

#### <a name="requirements"></a>Requisitos

*   La máquina del entorno de ejecución de integración autohospedado debe unirse a un dominio de Windows.
*   Necesita permiso para actualizar la configuración del controlador de dominio.

#### <a name="how-to-configure"></a>Cómo se configura

> [!NOTE]
> Reemplace REALM.COM y AD.COM en el siguiente tutorial por su nombre de dominio y controlador de dominio.

**En el servidor KDC:**

1. Edite la configuración de KDC en el archivo *krb5.conf* para permitir que KDC confíe en el dominio de Windows; para ello, remítase a la siguiente plantilla de configuración. La configuración se encuentra de forma predeterminada en */etc/krb5.conf*.

   ```config
   [logging]
    default = FILE:/var/log/krb5libs.log
    kdc = FILE:/var/log/krb5kdc.log
    admin_server = FILE:/var/log/kadmind.log
            
   [libdefaults]
    default_realm = REALM.COM
    dns_lookup_realm = false
    dns_lookup_kdc = false
    ticket_lifetime = 24h
    renew_lifetime = 7d
    forwardable = true
            
   [realms]
    REALM.COM = {
     kdc = node.REALM.COM
     admin_server = node.REALM.COM
    }
   AD.COM = {
    kdc = windc.ad.com
    admin_server = windc.ad.com
   }
            
   [domain_realm]
    .REALM.COM = REALM.COM
    REALM.COM = REALM.COM
    .ad.com = AD.COM
    ad.com = AD.COM
            
   [capaths]
    AD.COM = {
     REALM.COM = .
    }
    ```

   Después de configurar el archivo, reinicie el servicio KDC.

2. Prepare una entidad de seguridad llamada *krbtgt/REALM.COM\@AD.COM* en el servidor KDC con el comando siguiente:

    ```cmd
    Kadmin> addprinc krbtgt/REALM.COM@AD.COM
    ```

3. En el archivo de configuración del servicio de HDFS *hadoop.security.auth_to_local*, agregue `RULE:[1:$1@$0](.*\@AD.COM)s/\@.*//`.

**En el controlador de dominio:**

1.  Ejecute los siguientes comandos `Ksetup` para agregar una entrada de dominio:

    ```cmd
    C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
    C:> ksetup /addhosttorealmmap HDFS-service-FQDN REALM.COM
    ```

2.  Establezca una confianza entre el dominio de Windows y el dominio Kerberos. [contraseña] es la contraseña de la entidad de seguridad *krbtgt/REALM.COM\@AD.COM*.

    ```cmd
    C:> netdom trust REALM.COM /Domain: AD.COM /add /realm /password:[password]
    ```

3.  Seleccione el algoritmo de cifrado que se usó en Kerberos.

    a. Vaya a **Administrador de servidores** > **Administración de directivas de grupo** > **Dominio** > **Objetos de directiva de grupo** > **Default or Active Domain Policy (Directiva de dominio predeterminada o activa)** y haga clic en **Editar**.

    b. En el panel **Editor de administración de directivas de grupo**, seleccione **Configuración del equipo** > **Directivas** > **Configuración de Windows** > **Configuración de seguridad** > **Directivas locales** > **Opciones de seguridad** y configure **Seguridad de red: Configure los tipos de cifrado permitidos para Kerberos**.

    c. Seleccione el algoritmo de cifrado que quiere usar cuando se conecte al servidor KDC. Puede seleccionar todas las opciones.

    ![Captura de pantalla de "Seguridad de red: Configurar tipos de cifrado permitidos para Kerberos"](media/connector-hdfs/config-encryption-types-for-kerberos.png)

    d. Use el comando `Ksetup` para especificar el algoritmo de cifrado que se usará en el dominio específico.

    ```cmd
    C:> ksetup /SetEncTypeAttr REALM.COM DES-CBC-CRC DES-CBC-MD5 RC4-HMAC-MD5 AES128-CTS-HMAC-SHA1-96 AES256-CTS-HMAC-SHA1-96
    ```

4.  Cree una asignación entre la cuenta de dominio y la entidad de seguridad de Kerberos para que pueda usar la entidad de seguridad de Kerberos en el dominio de Windows.

    a. Seleccione **Herramientas administrativas** > **Equipos y usuarios de Active Directory**.

    b. Configure las características avanzadas; para ello, seleccione **Ver** > **Características avanzadas**.

    c. En el panel **Características avanzadas**, haga clic con el botón derecho en la cuenta con la que quiera crear las asignaciones y, en el panel **Asignaciones de nombres**, seleccione la pestaña **Nombres Kerberos**.

    d. Agregue una entidad de seguridad del dominio Kerberos.

       ![Panel "Asignación de identidad de seguridad"](media/connector-hdfs/map-security-identity.png)

**En la máquina del entorno de ejecución de integración autohospedado:**

* Ejecute los siguientes comandos `Ksetup` para agregar una entrada de dominio.

   ```cmd
   C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
   C:> ksetup /addhosttorealmmap HDFS-service-FQDN REALM.COM
   ```

**En la factoría de datos:**

* Configure el conector de HDFS mediante la autenticación de Windows en combinación con la cuenta de dominio o la entidad de seguridad de Kerberos para conectarse al origen de datos de HDFS. Para conocer los detalles de la configuración, consulte la sección [Propiedades del servicio vinculado](#linked-service-properties).

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información sobre las propiedades de la actividad de búsqueda, consulte [Actividad de búsqueda en Azure Data Factory](control-flow-lookup-activity.md).

## <a name="legacy-models"></a>Modelos heredados

>[!NOTE]
>Estos modelos siguen siendo compatibles tal cuales con versiones anteriores. Se recomienda usar el nuevo modelo descrito anteriormente, ya que la UI de creación de Azure Data Factory ha cambiado para generar el nuevo modelo.

### <a name="legacy-dataset-model"></a>Modelo de conjunto de datos heredado

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad *type* del conjunto de datos debe establecerse en *FileShare*. |Sí |
| folderPath | Ruta de acceso a la carpeta. Se admite un filtro con caracteres comodín. Los caracteres comodín permitidos son `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter); use `^` como carácter de escape si el nombre de archivo real tiene un carácter comodín o este carácter de escape dentro. <br/><br/>Ejemplos: rootfolder/subfolder/ver más en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). |Sí |
| fileName |  Filtro de nombre o de comodín para los archivos de la ruta "folderPath" especificada. Si no especifica ningún valor para esta propiedad, el conjunto de datos apunta a todos los archivos de la carpeta. <br/><br/>Para filtrar, los caracteres comodín permitidos son `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter).<br/>- Ejemplo 1: `"fileName": "*.csv"`<br/>- Ejemplo 2: `"fileName": "???20180427.txt"`<br/>Use `^` como escape si el nombre real de la carpeta contiene un carácter comodín o este carácter de escape. |No |
| modifiedDatetimeStart | Los archivos se filtran en función del atributo *Last Modified*. Los archivos se seleccionan si la hora de su última modificación está dentro del intervalo de `modifiedDatetimeStart` a `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato *2018-12-01T05:00:00Z*. <br/><br/> Tenga en cuenta que el rendimiento general del movimiento de datos se ve afectado si habilita esta configuración cuando desea aplicar un filtro de archivo a grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser NULL, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.| No |
| modifiedDatetimeEnd | Los archivos se filtran en función del atributo *Last Modified*. Los archivos se seleccionan si la hora de su última modificación está dentro del intervalo de `modifiedDatetimeStart` a `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato *2018-12-01T05:00:00Z*. <br/><br/> Tenga en cuenta que el rendimiento general del movimiento de datos se ve afectado si habilita esta configuración cuando desea aplicar un filtro de archivo a grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser NULL, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.| No |
| format | Si desea copiar los archivos tal cual entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida.<br/><br/>Si quiere analizar archivos con un formato concreto, se admiten los siguientes tipos de formato de archivo: *TextFormat*, *JsonFormat*, *AvroFormat*, *OrcFormat*, *ParquetFormat*. Establezca la propiedad *type* de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](supported-file-formats-and-compression-codecs-legacy.md#text-format), [Formato Json](supported-file-formats-and-compression-codecs-legacy.md#json-format), [Formato Avro](supported-file-formats-and-compression-codecs-legacy.md#avro-format), [Formato ORC](supported-file-formats-and-compression-codecs-legacy.md#orc-format) y [Formato Parquet](supported-file-formats-and-compression-codecs-legacy.md#parquet-format). |No (solo para el escenario de copia binaria) |
| compression | Especifique el tipo y el nivel de compresión de los datos. Para más información, consulte el artículo sobre [códecs de compresión y formatos de archivo compatibles](supported-file-formats-and-compression-codecs-legacy.md#compression-support).<br/>Estos son los tipos que se admiten: *Gzip*, *Deflate*, *Bzip2* y *ZipDeflate*.<br/>Estos son los niveles que se admiten: *Optimal* y *Fastest*. |No |

>[!TIP]
>Para copiar todos los archivos en una carpeta, especifique solo **folderPath**.<br>Para copiar un único archivo con un nombre determinado, especifique **folderPath** con el elemento de carpeta y **fileName** con el nombre de archivo.<br>Para copiar un subconjunto de archivos en una carpeta, especifique **folderPath** con el elemento de carpeta y **fileName** con el filtro de comodín.

**Ejemplo**:

```json
{
    "name": "HDFSDataset",
    "properties": {
        "type": "FileShare",
        "linkedServiceName":{
            "referenceName": "<HDFS linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "folder/subfolder/",
            "fileName": "*",
            "modifiedDatetimeStart": "2018-12-01T05:00:00Z",
            "modifiedDatetimeEnd": "2018-12-01T06:00:00Z",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

### <a name="legacy-copy-activity-source-model"></a>Modelo de origen de actividad de copia heredada

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad *type* del origen de la actividad de copia debe establecerse en: *HdfsSource*. |Sí |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Cuando recursive se establece en *true* y el receptor es un almacén basado en archivos, no se copiará ni creará una subcarpeta o carpeta vacía en el receptor.<br/>Los valores permitidos son: *True* (valor predeterminado) y *False*. | No |
| distcpSettings | Grupo de propiedades cuando se usa la herramienta DistCp de HDFS. | No |
| resourceManagerEndpoint | Punto de conexión de Resource Manager de YARN | Sí, si se utiliza DistCp |
| tempScriptPath | Ruta de acceso de carpeta que se usa para almacenar el script del comando DistCp temporal. Data Factory se encarga de crear el archivo de script que se eliminará después de que haya finalizado el trabajo de copia. | Sí, si se utiliza DistCp |
| distcpOptions | Se proporcionan opciones adicionales para el comando DistCp. | No |
| maxConcurrentConnections | Número de conexiones que se pueden conectar al almacén de almacenamiento de forma simultánea. Especifique un valor solamente cuando desee limitar la conexión simultánea al almacén de datos. | No |

**Ejemplo: Origen HDFS de la actividad de copia mediante DistCp**

```json
"source": {
    "type": "HdfsSource",
    "distcpSettings": {
        "resourceManagerEndpoint": "resourcemanagerendpoint:8088",
        "tempScriptPath": "/usr/hadoop/tempscript",
        "distcpOptions": "-m 100"
    }
}
```

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
