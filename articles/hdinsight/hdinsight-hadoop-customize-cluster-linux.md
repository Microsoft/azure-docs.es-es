---
title: Personalización de clústeres de HDInsight de Azure mediante el uso de las acciones de script
description: Agregue componentes personalizados a clústeres de HDInsight basados en Linux mediante acciones de script. Las acciones de script son scripts de Bash que se pueden usar para personalizar la configuración del clúster o para agregar servicios adicionales y utilidades como Hue, Solr o R.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 04/02/2019
ms.openlocfilehash: 63f81c331db619323f74b77e48627fd8b432565f
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65518888"
---
# <a name="customize-azure-hdinsight-clusters-by-using-script-actions"></a>Personalización de clústeres de HDInsight de Azure mediante el uso de las acciones de script

Azure HDInsight proporciona un método de configuración llamado **acciones de script**, que invoca scripts personalizados para personalizar el clúster. Estos scripts se usan para instalar componentes adicionales y para cambiar los valores de configuración. Las acciones de script pueden usarse durante la creación del clúster o después.

Las acciones de script también pueden publicarse en Azure Marketplace como una aplicación de HDInsight. Para más información sobre aplicaciones de HDInsight, consulte [Publicación de una aplicación de HDInsight en Azure Marketplace](hdinsight-apps-publish-applications.md).

## <a name="permissions"></a>Permisos

Para un clúster de HDInsight unido a un dominio, necesita dos permisos de Apache Ambari para usar acciones de script con el clúster:

* **AMBARI.RUN\_CUSTOM\_COMMAND**: el rol de administrador de Ambari tiene este permiso de forma predeterminada.
* **CLUSTER.RUN\_CUSTOM\_COMMAND**: el administrador de clústeres de HDInsight y el administrador de Ambari tienen este permiso de forma predeterminada.

Para más información sobre cómo trabajar con permisos con clústeres de HDInsight unidos a un dominio, consulte [Administración de clústeres de HDInsight con Enterprise Security Package](./domain-joined/apache-domain-joined-manage.md).

## <a name="access-control"></a>Control de acceso

Si no es el administrador o propietario de la suscripción de Azure, la cuenta debe tener al menos acceso de Colaborador al grupo de recursos que contiene el clúster de HDInsight.

Si crea un clúster de HDInsight, un usuario con al menos acceso de Colaborador a la suscripción de Azure debe haber registrado previamente el proveedor para HDInsight. El registro del proveedor se produce cuando un usuario con acceso de Colaborador a la suscripción crea un recurso por primera vez en la suscripción. También puede realizarse sin crear ningún recurso si [registra un proveedor con REST](https://msdn.microsoft.com/library/azure/dn790548.aspx).

Obtenga más información sobre cómo trabajar con la administración de acceso:

* [Introducción a la administración de acceso en Azure Portal](../role-based-access-control/overview.md)
* [Uso de asignaciones de roles para administrar el acceso a los recursos de la suscripción de Azure](../role-based-access-control/role-assignments-portal.md)

## <a name="understand-script-actions"></a>Descripción de las acciones de script

Una acción de script es un script de Bash que se ejecuta en los nodos de un clúster de HDInsight. Estas son algunas características de las acciones de script:

* Deben almacenarse en un URI accesible desde el clúster de HDInsight. A continuación, se proponen varias ubicaciones de almacenamiento posibles:
    
    * Para los clústeres regulares:
    
      * Gen1 ADLS: La entidad de servicio que HDInsight usa para acceder a Data Lake Storage debe tener acceso de lectura al script. El formato del identificador URI para scripts almacenados en Data Lake Storage Gen1 es `adl://DATALAKESTOREACCOUNTNAME.azuredatalakestore.net/path_to_file`.
      
      * Un blob de una cuenta de Azure Storage que sea la cuenta de almacenamiento principal o adicional del clúster de HDInsight. Durante la creación del clúster se concede acceso a HDInsight a estos dos tipos de cuentas de almacenamiento.

      * Un uso compartido de archivos servicio público accesible a través de las rutas de acceso de http://. Algunos ejemplos son Azure Blob, GitHub, OneDrive.

        Para obtener URI de ejemplo, consulte [Ejemplo de scripts de acción de script](#example-script-action-scripts).

     * Para clústeres con ESP:
         
         * El wasb [s] :// o http [s] :// se admiten los URI.
            
* Pueden restringirse para ejecutarse solo en determinados tipos de nodos. Algunos ejemplos son los nodos principales o nodos de trabajo.

* Pueden ser persistentes o ad hoc.

    Los scripts persistentes se usan para personalizar nuevos nodos de trabajo agregados al clúster mediante operaciones de escalado. Un script persistente también puede aplicar cambios a otro tipo de nodo cuando se producen operaciones de escalado. Un ejemplo es un nodo principal.

  > [!IMPORTANT]  
  > Las acciones de scripts persistentes deben tener un nombre único.

    Los scripts ad hoc no son persistentes. No se aplican a los nodos de trabajo agregados al clúster después de la ejecución del script. Posteriormente, un script ad hoc se puede promover a persistente y uno persistente puede disminuirse a ad hoc.

  > [!IMPORTANT]  
  > Las acciones de script usadas durante la creación de un clúster se guardan automáticamente.
  >
  > Los scripts que dan error no se guardan como persistentes, aunque indique específicamente que así sea.

* Pueden aceptar parámetros que usa el script durante la ejecución.

* Se ejecutan con privilegios de nivel raíz en los nodos del clúster.

* Se pueden usar mediante Azure Portal, Azure PowerShell, la CLI clásica de Azure o el SDK de .NET para HDInsight.

El clúster conserva un historial de todos los scripts que se han ejecutado. El historial resulta útil cuando se necesita buscar el identificador de un script para las operaciones de promoción o disminución de nivel.

> [!IMPORTANT]  
> No hay ninguna forma automática de deshacer los cambios realizados por una acción de script. Los cambios se revierten de forma manual o se proporciona un script que lo hace.

### <a name="script-action-in-the-cluster-creation-process"></a>Acción de script en el proceso de creación de clústeres

Las acciones de script usadas durante la creación de un clúster son algo diferentes de las ejecutadas en un clúster existente:

* El script se guarda automáticamente como persistente.

* Un error del script puede impedir que el proceso de creación del clúster se efectúe correctamente.

En el siguiente diagrama se ilustra el momento en que la acción de script se ejecuta durante el proceso de creación:

![Fases y personalización de clústeres de HDInsight durante la creación de clústeres][img-hdi-cluster-states]

El script se ejecuta mientras se configura HDInsight. El script se ejecuta de forma paralela en todos los nodos especificados del clúster. Se ejecuta con privilegios raíz en los nodos.

> [!NOTE]  
> Puede realizar operaciones, como detener e iniciar servicios, incluidos los servicios relacionados con Apache Hadoop. Si detiene los servicios, asegúrese de que los servicios de Ambari y otros servicios de Hadoop estén en ejecución antes de que finalice el script. Estos servicios son necesarios para determinar correctamente el estado del clúster mientras se crea.


Durante la creación del clúster, puede usar diversas acciones de script a la vez. Estos scripts se invocan en el orden en el que se hayan especificado.

> [!IMPORTANT]  
> Las acciones de script deben finalizar en 60 minutos o agotarán el tiempo de espera. Durante el aprovisionamiento del clúster, el script se ejecuta a la vez con otros procesos de instalación y configuración. La competición por los recursos, como el ancho de banda de red o el tiempo de CPU, puede ocasionar que el script tarde más en terminar que en el entorno de desarrollo.
>
> Para minimizar el tiempo necesario para ejecutar el script, evite tareas tales como descargar y compilar aplicaciones desde el origen. Compile las aplicaciones previamente y almacene el archivo binario en Azure Storage.


### <a name="script-action-on-a-running-cluster"></a>Acción de script en un clúster en ejecución

Un error en un script ejecutado en un clúster ya en ejecución no hace que el clúster cambie automáticamente a un estado de error. Cuando un script finaliza, el clúster debe volver a un estado en ejecución.

> [!IMPORTANT]  
> Aunque el clúster tenga el estado en ejecución, el script erróneo puede haber roto algo. Por ejemplo, un script podría eliminar archivos necesarios para el clúster.
>
> Las acciones de script se ejecutan con privilegios raíz. Asegúrese de que comprende lo que hace un script antes de aplicarlo a un clúster.

Al aplicar un script a un clúster, se cambia el estado del clúster de **En ejecución** a **Aceptado**. A continuación, cambia a **Configuración de HDInsight** y, por último, regresa a **En ejecución** en el caso de los scripts correctos. El estado del script se registra en el historial de acciones de script. Esta información le indica si el script se ha completado correctamente o no. Por ejemplo, el cmdlet `Get-AzHDInsightScriptActionHistory` de PowerShell muestra el estado de un script. Devuelve información similar al texto siguiente:

    ScriptExecutionId : 635918532516474303
    StartTime         : 8/14/2017 7:40:55 PM
    EndTime           : 8/14/2017 7:41:05 PM
    Status            : Succeeded

> [!IMPORTANT]  
> Si cambia la contraseña de usuario (admin) del clúster tras la creación de este, se puede producir un error en las acciones de script ejecutadas en este clúster. Si tiene alguna acción de script persistente cuyo destino sea algún nodo de trabajo, se puede producir un error en este script al escalar el clúster.

## <a name="example-script-action-scripts"></a>Ejemplo de scripts de acción de script

Los scripts de acciones de script se pueden usar con las utilidades siguientes:

* El Portal de Azure
* Azure PowerShell
* La CLI clásica de Azure
* Un SDK .NET de HDInsight

HDInsight proporciona scripts para instalar los siguientes componentes en clústeres de HDInsight:

| NOMBRE | Script |
| --- | --- |
| Agregar una cuenta de Azure Storage |`https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh`  Consulte [Adición de más cuentas de almacenamiento a HDInsight](hdinsight-hadoop-add-storage.md). |
| Instalar Hue |`https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh`  Consulte [Instalación y uso de Hue en clústeres de Hadoop para HDInsight](hdinsight-hadoop-hue-linux.md). |
| Instalar Presto |`https://raw.githubusercontent.com/hdinsight/presto-hdinsight/master/installpresto.sh`  Consulte [Instalación y uso de Presto en clústeres de Hadoop para HDInsight](hdinsight-hadoop-install-presto.md). |
| Instalar Giraph |`https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh`. Consulte [Instalación de Apache Giraph en clústeres de Hadoop en HDInsight](hdinsight-hadoop-giraph-install-linux.md). |
| Carga previa de las bibliotecas de Hive |`https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh`. Consulte [Incorporación de bibliotecas personalizadas de Apache Hive al crear el clúster de HDInsight](hdinsight-hadoop-add-hive-libraries.md). |

## <a name="use-a-script-action-during-cluster-creation"></a>Uso de una acción de script durante la creación de un clúster

En esta sección se explican las distintas maneras de usar acciones de script al crear un clúster de HDInsight.

### <a name="use-a-script-action-during-cluster-creation-from-the-azure-portal"></a>Uso de una acción de script durante la creación de un clúster desde Azure Portal

1. Empiece a crear un clúster tal y como se describe en [Configuración de clústeres en HDInsight con Apache Hadoop, Apache Spark, Apache Kafka, etc.](hdinsight-hadoop-provision-linux-clusters.md) Durante la creación del clúster, llegará a la página __Resumen del clúster__. En la página __Resumen del clúster__, seleccione el vínculo __editar__ de __Configuración avanzada__.

    ![Vínculo Configuración avanzada](./media/hdinsight-hadoop-customize-cluster-linux/advanced-settings-link.png)

3. En la sección __Configuración avanzada__, seleccione __Acciones de script__. En la sección __Acciones de script__, seleccione __+ Enviar nuevo__.

    ![Enviar una nueva acción de script](./media/hdinsight-hadoop-customize-cluster-linux/add-script-action.png)

4. Use la entrada __Seleccione un script__ para seleccionar un script prefabricado. Para utilizar un script personalizado, seleccione __Personalizado__. A continuación, proporcione el __nombre__ y el __URI de script de Bash__ del script.

    ![Agregar un script en el formulario de selección de script](./media/hdinsight-hadoop-customize-cluster-linux/select-script.png)

    En la siguiente tabla se describen los elementos del formulario:

    | Propiedad | Valor |
    | --- | --- |
    | Seleccione un script | Para utilizar su propio script, seleccione __Custom__ (Personalizado). En caso contrario, seleccione uno de los que se proporcionan. |
    | NOMBRE |Especifique un nombre para la acción de script. |
    | URI de script de Bash |Especifique el URI del script. |
    | Principal, Trabajo o Zookeeper |Especifique los nodos en los que se ejecuta el script: **Principal**, **Trabajo** o **ZooKeeper**. |
    | Parámetros |Especifique los parámetros, si lo requiere el script. |

    Use la entrada __Continuar con esta acción de script__ para asegurarse de que el script se aplica durante las operaciones de escalado.

5. Seleccione __Crear__ para guardar el script. Luego puede usar __+ Enviar nuevo__ para agregar otro script.

    ![Varias acciones de script](./media/hdinsight-hadoop-customize-cluster-linux/multiple-scripts.png)

    Cuando haya terminado de agregar scripts, seleccione el botón __Seleccionar__ y el botón __Siguiente__ para volver a la sección de __resumen del clúster__.

3. Para crear el clúster, seleccione __Crear__ en la sección de __resumen del clúster__.

### <a name="use-a-script-action-from-azure-resource-manager-templates"></a>Uso de una acción de script desde las plantillas de Azure Resource Manager

Las acciones de script pueden usarse con plantillas de Azure Resource Manager. Para ver un ejemplo, consulte [Creación de un clúster de HDInsight Linux y ejecución de una acción de script](https://azure.microsoft.com/resources/templates/hdinsight-linux-run-script-action/).

En este ejemplo, la acción de script se agrega utilizando el código siguiente:

```json
"scriptActions": [
    {
        "name": "setenvironmentvariable",
        "uri": "[parameters('scriptActionUri')]",
        "parameters": "headnode"
    }
]
```

Obtenga más información sobre cómo implementar una plantilla:

* [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)

* [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-cli)

### <a name="use-a-script-action-during-cluster-creation-from-azure-powershell"></a>Uso de una acción de script durante la creación de un clúster desde Azure PowerShell

En esta sección, usará el [agregar AzHDInsightScriptAction](https://docs.microsoft.com/powershell/module/az.hdinsight/add-azhdinsightscriptaction) cmdlet para invocar scripts para personalizar un clúster. Antes de empezar, asegúrese de instalar y configurar Azure PowerShell. Para usar estos comandos de PowerShell, necesitará el [módulo AZ](https://docs.microsoft.com/powershell/azure/overview).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

El script siguiente muestra cómo aplicar una acción de script al crear un clúster con PowerShell:

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=5-90)]

El clúster puede tardar varios minutos en crearse.

### <a name="use-a-script-action-during-cluster-creation-from-the-hdinsight-net-sdk"></a>Uso de una acción de script durante la creación de un clúster desde el SDK de .NET para HDInsight

El .NET SDK de HDInsight proporciona bibliotecas de cliente que facilitan el trabajo con HDInsight desde una aplicación .NET. Para ver un ejemplo de código, consulte [Crear clústeres basados en Linux en HDInsight con el SDK de .NET](hdinsight-hadoop-create-linux-clusters-dotnet-sdk.md#use-script-action).

## <a name="apply-a-script-action-to-a-running-cluster"></a>Aplicación de una acción de script a un clúster en ejecución

En esta sección se explica cómo aplicar acciones de script a un clúster en ejecución.

### <a name="apply-a-script-action-to-a-running-cluster-from-the-azure-portal"></a>Aplicación de una acción de script a un clúster en ejecución desde Azure Portal

Vaya a [Azure Portal](https://portal.azure.com):

1. En el menú izquierdo, seleccione **Todos los servicios**.

1. Seleccione **Clústeres de HDInsight** en **ANALYTICS**.

1. Seleccione el clúster en la lista, lo que abrirá la vista predeterminada.

1. En la vista predeterminada, en **Configuración**, seleccione **Acciones de script**.

1. En la parte superior de la página **Acciones de script**, seleccione **+ Enviar nuevo**.

    ![Agregar un script a un clúster en ejecución](./media/hdinsight-hadoop-customize-cluster-linux/add-script-running-cluster.png)

4. Use la entrada __Seleccione un script__ para seleccionar un script prefabricado. Para utilizar un script personalizado, seleccione __Personalizado__. A continuación, proporcione el __nombre__ y el __URI de script de Bash__ del script.

    ![Agregar un script en el formulario de selección de script](./media/hdinsight-hadoop-customize-cluster-linux/select-script.png)

    En la siguiente tabla se describen los elementos del formulario:

    | Propiedad | Valor |
    | --- | --- |
    | Seleccione un script | Para utilizar su propio script, seleccione __Custom__ (Personalizado). En caso contrario, seleccione uno de los que se proporcionan. |
    | NOMBRE |Especifique un nombre para la acción de script. |
    | URI de script de Bash |Especifique el URI del script. |
    | Principal, Trabajo o Zookeeper |Especifique los nodos en los que se ejecuta el script: **Principal**, **Trabajo** o **ZooKeeper**. |
    | Parámetros |Especifique los parámetros, si lo requiere el script. |

    Use la entrada __Continuar con esta acción de script__ para asegurarse de que el script se aplica durante las operaciones de escalado.

5. Por último, seleccione el botón **Crear** para aplicar el script al clúster.

### <a name="apply-a-script-action-to-a-running-cluster-from-azure-powershell"></a>Aplicación de una acción de script a un clúster en ejecución desde Azure PowerShell

Para usar estos comandos de PowerShell, necesitará el [módulo AZ](https://docs.microsoft.com/powershell/azure/overview).

El ejemplo siguiente muestra cómo aplicar una acción de script a un clúster en ejecución:

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=105-117)]

Una vez terminada la operación, recibirá información similar al siguiente texto:

    OperationState  : Succeeded
    ErrorMessage    :
    Name            : Giraph
    Uri             : https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh
    Parameters      :
    NodeTypes       : {HeadNode, WorkerNode}

### <a name="apply-a-script-action-to-a-running-cluster-from-the-azure-cli"></a>Aplicación de una acción de script a un clúster en ejecución desde la CLI de Azure

Antes de empezar, asegúrese de instalar y configurar la CLI de Azure. Para más información, consulte [Instalación de la CLI clásica de Azure](https://docs.microsoft.com/cli/azure/install-classic-cli?view=azure-cli-latest).

[!INCLUDE [classic-cli-warning](../../includes/requires-classic-cli.md)]

1. Cambie al modo de Azure Resource Manager:

    ```bash
    azure config mode arm
    ```

2. Autentíquese en la suscripción de Azure:

    ```bash
    azure login
    ```

3. Aplique una acción de script a un clúster en ejecución:

    ```bash
    azure hdinsight script-action create <clustername> -g <resourcegroupname> -n <scriptname> -u <scriptURI> -t <nodetypes>
    ```

    Si omite los parámetros de este comando, se le solicitarán. Si el script que especifica con `-u` acepta parámetros, puede especificarlos mediante el parámetro `-p`.

    Los tipos de nodo válidos son `headnode`, `workernode` y `zookeeper`. Si el script se debe aplicar a varios tipos de nodo, especifique los tipos separados por punto y coma `;`. Por ejemplo, `-n headnode;workernode`.

    Para guardar el script como persistente, agregue `--persistOnSuccess`. También puede guardar el script como persistente más adelante mediante `azure hdinsight script-action persisted set`.

    Cuando el trabajo haya finalizado, obtendrá una salida similar al siguiente texto:

        info:    Executing command hdinsight script-action create
        + Executing Script Action on HDInsight cluster
        data:    Operation Info
        data:    ---------------
        data:    Operation status:
        data:    Operation ID:  b707b10e-e633-45c0-baa9-8aed3d348c13
        info:    hdinsight script-action create command OK

### <a name="apply-a-script-action-to-a-running-cluster-by-using-rest-api"></a>Aplicación de una acción de script a un clúster en ejecución mediante la API de REST

Consulte [API de REST del clúster en Azure HDInsight](https://msdn.microsoft.com/library/azure/mt668441.aspx).

### <a name="apply-a-script-action-to-a-running-cluster-from-the-hdinsight-net-sdk"></a>Aplicación de una acción de script a un clúster en ejecución desde el SDK de .NET para HDInsight

Si desea ver un ejemplo de uso del SDK de .NET para aplicar scripts a un clúster, consulte cómo [aplicar una acción de script en un clúster de HDInsight basado en Linux en ejecución](https://github.com/Azure-Samples/hdinsight-dotnet-script-action).

## <a name="view-history-and-promote-and-demote-script-actions"></a>Visualización del historial, promoción y disminución de acciones de script

### <a name="the-azure-portal"></a>El Portal de Azure

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).

1. En el menú izquierdo, seleccione **Todos los servicios**.

1. Seleccione **Clústeres de HDInsight** en **ANALYTICS**.

1. Seleccione el clúster en la lista, lo que abrirá la vista predeterminada.

1. En la vista predeterminada, en **Configuración**, seleccione **Acciones de script**.

4. En la sección Acciones de script aparece un historial de scripts de este clúster. Esta información incluye una lista de scripts persistentes. La siguiente captura de pantalla muestra que el script Solr se ha ejecutado en este clúster. La captura de pantalla no muestra ningún script persistente.

    ![Acciones de script](./media/hdinsight-hadoop-customize-cluster-linux/script-action-history.png)

5. Seleccione un script del historial para mostrar la sección **Propiedades** de ese script. Desde la parte superior de la pantalla, puede volver a ejecutar el script o subirlo de nivel.

    ![Acciones de script, Propiedades](./media/hdinsight-hadoop-customize-cluster-linux/promote-script-actions.png)

6. También puede usar los puntos suspensivos (**...**) de la derecha de las entradas de la sección Acciones de script para realizar acciones.

    ![Acciones de script, puntos suspensivos](./media/hdinsight-hadoop-customize-cluster-linux/deletepromoted.png)

### <a name="azure-powershell"></a>Azure PowerShell

| cmdlet | Función |
| --- | --- |
| `Get-AzHDInsightPersistedScriptAction` |Recuperar información sobre acciones de script persistentes. |
| `Get-AzHDInsightScriptActionHistory` |Recuperar un historial de acciones de script aplicadas al clúster o detalles de un script específico. |
| `Set-AzHDInsightPersistedScriptAction` |Promover una acción de script ad hoc a una acción de script persistente. |
| `Remove-AzHDInsightPersistedScriptAction` |Disminuir de nivel una acción de script persistente a una acción ad hoc. |

> [!IMPORTANT]  
> `Remove-AzHDInsightPersistedScriptAction` no deshace las acciones realizadas por un script. Este cmdlet solo quita la marca persistente.

El siguiente script de ejemplo muestra cómo utilizar los cmdlets para promover y luego disminuir de nivel un script.

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=123-140)]

### <a name="the-azure-classic-cli"></a>La CLI clásica de Azure

| cmdlet | Función |
| --- | --- |
| `azure hdinsight script-action persisted list <clustername>` |Recuperar una lista de las acciones de script persistentes. |
| `azure hdinsight script-action persisted show <clustername> <scriptname>` |Recuperar información sobre una acción de script persistente específica. |
| `azure hdinsight script-action history list <clustername>` |Recuperar un historial de acciones de script aplicadas al clúster. |
| `azure hdinsight script-action history show <clustername> <scriptname>` |Recuperar información sobre una acción de script específica. |
| `azure hdinsight script action persisted set <clustername> <scriptexecutionid>` |Promover una acción de script ad hoc a una acción de script persistente. |
| `azure hdinsight script-action persisted delete <clustername> <scriptname>` |Disminuir de nivel una acción de script persistente a una acción ad hoc. |

> [!IMPORTANT]  
> `azure hdinsight script-action persisted delete` no deshace las acciones realizadas por un script. Este cmdlet solo quita la marca persistente.

### <a name="the-hdinsight-net-sdk"></a>El SDK de .NET para HDInsight

Si desea ver un ejemplo de uso del SDK de .NET para recuperar el historial de scripts de un clúster, o bien promover scripts o disminuir el nivel de estos, consulte cómo [aplicar una acción de script en un clúster de HDInsight basado en Linux en ejecución](https://github.com/Azure-Samples/hdinsight-dotnet-script-action).

> [!NOTE]  
> En este ejemplo también se muestra cómo instalar una aplicación de HDInsight mediante el SDK. de NET.

## <a name="support-for-open-source-software-used-on-hdinsight-clusters"></a>Soporte técnico para el software de código abierto utilizado en clústeres de HDInsight

El servicio Microsoft Azure HDInsight usa un ecosistema de tecnologías de código abierto formado en torno a Apache Hadoop. Microsoft Azure proporciona un nivel general de soporte técnico para las tecnologías de código abierto. Para más información, vea la sección de **ámbito de soporte técnico** de [Preguntas más frecuentes de soporte técnico de Azure](https://azure.microsoft.com/support/faq/). Además, el servicio HDInsight ofrece un nivel adicional de soporte técnico para los componentes incorporados.

Hay dos tipos de componentes de código abierto disponibles en el servicio de HDInsight:

* **Componentes integrados**. Estos componentes están instalados previamente en clústeres de HDInsight y proporcionan la funcionalidad básica del clúster. Los siguientes componentes pertenecen a esta categoría:

  * [Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html) ResourceManager.
  * El lenguaje de consulta de Hive [HiveQL](https://cwiki.apache.org/confluence/display/Hive/LanguageManual).
  * [Apache Mahout](https://mahout.apache.org/). 
    
    Hay una lista completa de componentes del clúster disponible en [¿Cuáles son los componentes y versiones de Apache Hadoop disponibles con HDInsight?](hdinsight-component-versioning.md).

* **Componentes personalizados**. Como usuario del clúster, puede instalar o usar en la carga de trabajo cualquier componente que esté disponible en la comunidad o que haya creado personalmente.

> [!WARNING]  
> Los componentes proporcionados con el clúster de HDInsight son totalmente compatibles. Soporte técnico de Microsoft ayuda a aislar y a solucionar problemas relacionados con estos componentes.
>
> Los componentes personalizados reciben soporte técnico comercialmente razonable para ayudarle a solucionar el problema. El soporte técnico de Microsoft podría solucionar el problema. También podría sugerirle que consulte los canales disponibles para las tecnologías de código abierto donde se concentran los conocimientos más amplios sobre cada tecnología. Hay diversos sitios de la comunidad que se pueden usar, por ejemplo, el [foro de MSDN para HDInsight](https://social.msdn.microsoft.com/Forums/azure/home?forum=hdinsight) y [Stack Overflow](https://stackoverflow.com). 
>
> Los proyectos de Apache también tienen sitios de proyecto en el [sitio web de Apache](https://apache.org). Un ejemplo es [Hadoop](https://hadoop.apache.org/).

El servicio HDInsight proporciona varias maneras de utilizar los componentes personalizados. Se aplica el mismo nivel de soporte técnico independientemente de cómo se use un componente o cómo se instale en el clúster. La siguiente lista describe las formas más comunes de usar los componentes personalizados en clústeres de HDInsight:

1. **Envío de trabajos**. Se pueden enviar al clúster trabajos de Hadoop o de otros tipos que ejecuten o usen componentes personalizados.

2. **Personalización del clúster**. Durante la creación del clúster puede especificar una configuración adicional y componentes personalizados que se instalan en los nodos del clúster.

3. **Ejemplos**. Para los componentes personalizados más populares, Microsoft y otros pueden proporcionar ejemplos de cómo estos componentes se pueden utilizar en los clústeres de HDInsight. Estas muestras se proporcionan sin soporte técnico.

## <a name="troubleshooting"></a>solución de problemas

Puede usar la interfaz de usuario web de Ambari para ver la información registrada por las acciones de script. Si se produce un error en el script durante la creación del clúster, los registros también están disponibles en la cuenta de almacenamiento predeterminada asociada al clúster. Esta sección proporciona información sobre cómo recuperar los registros mediante ambas opciones.

### <a name="the-apache-ambari-web-ui"></a>Interfaz de usuario web de Apache Ambari

1. En el explorador, vaya a https://CLUSTERNAME.azurehdinsight.net. Reemplace **CLUSTERNAME** por el nombre del clúster de HDInsight.

    Cuando se le solicite, escriba el nombre de cuenta de administrador (**admin**) y la contraseña de administrador para el clúster. Es posible que tenga que volver a escribir las credenciales de administrador en un formulario web.

2. En la barra de la parte superior de la página, seleccione la entrada **ops**. Una lista muestra las operaciones actuales y anteriores realizadas en el clúster mediante Ambari.

    ![Barra de interfaz de usuario web de Ambari con ops seleccionado](./media/hdinsight-hadoop-customize-cluster-linux/ambari-nav.png)

3. Busque las entradas que tienen **run\_customscriptaction** en la columna **Operaciones**. Estas entradas se crean al ejecutarse las acciones de script.

    ![Captura de pantalla de operaciones](./media/hdinsight-hadoop-customize-cluster-linux/ambariscriptaction.png)

    Para ver la salida **STDOUT** y **STDERR**, seleccione la entrada **run\customscriptaction** y profundice mediante los vínculos. Esta salida se genera cuando se ejecuta el script y puede contener información útil.

### <a name="access-logs-from-the-default-storage-account"></a>Acceso a los registros desde la cuenta de almacenamiento predeterminada

Si se produce un error al crear el clúster debido a un error de script, los registros permanecen en la cuenta de almacenamiento del clúster.

* Los registros de almacenamiento están disponibles en `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\CLUSTER_NAME\DATE`.

    ![Captura de pantalla de operaciones](./media/hdinsight-hadoop-customize-cluster-linux/script_action_logs_in_storage.png)

    En este directorio, los registros se organizan por separado para el **nodo principal**, el **nodo de trabajo** y el **nodo de Zookeeper**. Consulte los siguientes ejemplos:

    * **Nodo principal**: `<uniqueidentifier>AmbariDb-hn0-<generated_value>.cloudapp.net`

    * **Nodo de trabajo**: `<uniqueidentifier>AmbariDb-wn0-<generated_value>.cloudapp.net`

    * **Nodo de Zookeeper**: `<uniqueidentifier>AmbariDb-zk0-<generated_value>.cloudapp.net`

* Todos los **stdout** y **stderr** del host correspondiente se cargan en la cuenta de almacenamiento. Hay un archivo **output-\*.txt** y **errors-\*.txt** para cada acción de script. El archivo **output-*.txt** contiene información sobre el identificador URI del script que se ejecutó en el host. El siguiente texto es un ejemplo de esta información:

        'Start downloading script locally: ', u'https://hdiconfigactions.blob.core.windows.net/linuxrconfigactionv01/r-installer-v01.sh'

* Es posible crear repetidamente un clúster de la acción de script con el mismo nombre. En tal caso, puede distinguir los registros pertinentes según el nombre de la carpeta de **fecha**. Por ejemplo, la estructura de carpetas de un clúster (**mycluster**) creado en diferentes fechas es similar a las siguientes entradas de registro:

    `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\mycluster\2015-10-04``\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\mycluster\2015-10-05`

* Si crea un clúster de acción de script con el mismo nombre en el mismo día, puede usar el prefijo único para identificar los archivos de registro correspondientes.

* Si crea un clúster cerca de las 12:00 a. m. (medianoche), es posible que los archivos de registro abarquen dos días. En tal caso, verá dos carpetas de fecha diferentes para el mismo clúster.

* La carga de los archivos de registro en el contenedor predeterminado puede tardar hasta 5 minutos, especialmente para grandes clústeres. Por lo tanto, si desea tener acceso a los registros, no debe eliminar inmediatamente el clúster si se produce un error en una acción de script.

### <a name="ambari-watchdog"></a>Guardián Ambari

> [!WARNING]  
> No cambie la contraseña del guardián Ambari (hdinsightwatchdog) en el clúster de HDInsight basado en Linux. El cambio de la contraseña de esta cuenta impide ejecutar nuevas acciones de script en el clúster de HDInsight.

### <a name="cant-import-name-blobservice"></a>No se puede importar el nombre BlobService

__Síntomas__. se produce un error en la acción del script. Se muestra un texto similar al error siguiente al ver la operación en Ambari:

```
Traceback (most recent call list):
  File "/var/lib/ambari-agent/cache/custom_actions/scripts/run_customscriptaction.py", line 21, in <module>
    from azure.storage.blob import BlobService
ImportError: cannot import name BlobService
```

__Causa__. Este error se produce si actualiza el cliente de Azure Storage de Python que se incluye con el clúster de HDInsight. HDInsight espera el cliente de Azure Storage 0.20.0.

__Resolución__. Para resolver este error, conéctese manualmente a cada nodo del clúster mediante `ssh`. Ejecute el siguiente comando para volver a instalar la versión del cliente de almacenamiento correcta:

```bash
sudo pip install azure-storage==0.20.0
```

Para obtener información acerca de cómo conectarse al clúster con SSH, consulte [Conexión a HDInsight (Apache Hadoop) con SSH](hdinsight-hadoop-linux-use-ssh-unix.md).

### <a name="history-doesnt-show-the-scripts-used-during-cluster-creation"></a>El historial no muestra los scripts usados durante la creación del clúster

Si el clúster se creó antes del 15 de marzo de 2016, puede que no vea una entrada en el historial de acciones de script. Cambiar el tamaño del clúster hace que los scripts aparezcan en el historial de acciones de script.

Hay dos excepciones:

* El clúster se creó antes del 1 de septiembre de 2015. Esta es la fecha en que se presentaron las acciones de script. Cualquier clúster creado con anterioridad a esta fecha no ha podido usar acciones de script para su creación.

* Durante la creación del clúster, ha usado varias acciones de script o bien ha utilizado el mismo nombre para varios scripts, o el mismo nombre y el mismo identificador URI, pero diferentes parámetros para varios scripts. En estos casos, aparece el siguiente error:

    No se puede ejecutar ninguna acción de script nueva en este clúster debido al conflicto entre nombres de scripts existentes. Los nombres de script proporcionados en la creación del clúster deben ser únicos. Los scripts existentes se ejecutan con el cambio de tamaño.

## <a name="next-steps"></a>Pasos siguientes

* [Desarrollo de la acción de script con HDInsight](hdinsight-hadoop-script-actions-linux.md)
* [Instalación y uso de Apache Giraph en clústeres de HDInsight](hdinsight-hadoop-giraph-install-linux.md)
* [Adición de almacenamiento adicional a un clúster HDInsight](hdinsight-hadoop-add-storage.md)

[img-hdi-cluster-states]: ./media/hdinsight-hadoop-customize-cluster-linux/HDI-Cluster-state.png "Fases durante la creación del clúster"
