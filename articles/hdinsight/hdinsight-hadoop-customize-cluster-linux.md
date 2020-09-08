---
title: Personalización de los clústeres de Azure HDInsight mediante acciones de script en Azure
description: Agregue componentes personalizados a clústeres de HDInsight mediante acciones de script. Las acciones de script son scripts de Bash que se pueden usar para personalizar la configuración del clúster. También puede agregar servicios y utilidades adicionales, como Hue, Solr o R.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020
ms.date: 09/02/2020
ms.openlocfilehash: b30a7822511dc6b4c3ae7e852cba49ebff6e24ad
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400864"
---
# <a name="customize-azure-hdinsight-clusters-by-using-script-actions"></a>Personalización de los clústeres de Azure HDInsight mediante acciones de script en Azure

Azure HDInsight proporciona un método de configuración llamado **acciones de script**, que invoca scripts personalizados para personalizar el clúster. Estos scripts se usan para instalar componentes adicionales y para cambiar los valores de configuración. Las acciones de script pueden usarse durante la creación del clúster o después.

Las acciones de script también pueden publicarse en Azure Marketplace como una aplicación de HDInsight. Para más información sobre aplicaciones de HDInsight, consulte [Publicación de una aplicación de HDInsight en Azure Marketplace](hdinsight-apps-publish-applications.md).

## <a name="permissions"></a>Permisos

Para un clúster de HDInsight unido a un dominio, necesita dos permisos de Apache Ambari para usar acciones de script con el clúster:

* **AMBARI.RUN\_CUSTOM\_COMMAND**: el rol de administrador de Ambari tiene este permiso de forma predeterminada.
* **CLUSTER.RUN\_CUSTOM\_COMMAND**: el administrador de clústeres de HDInsight y el administrador de Ambari tienen este permiso de forma predeterminada.

Para más información sobre cómo trabajar con permisos con clústeres de HDInsight unidos a un dominio, consulte [Administración de clústeres de HDInsight con Enterprise Security Package](./domain-joined/apache-domain-joined-manage.md).

## <a name="access-control"></a>Control de acceso

Si no es el administrador o propietario de la suscripción de Azure, la cuenta debe tener al menos acceso de Colaborador al grupo de recursos que contiene el clúster de HDInsight.

Un usuario con al menos acceso de colaborador a la suscripción de Azure debe haber registrado previamente el proveedor. El registro del proveedor se produce cuando un usuario con acceso de colaborador a la suscripción crea un recurso. Para información sobre cómo se realiza este procedimiento sin crear un recurso, consulte cómo [registrar un proveedor mediante REST](https://msdn.microsoft.com/library/azure/dn790548.aspx).

Obtenga más información sobre cómo trabajar con la administración de acceso:

* [Introducción a la administración de acceso en Azure Portal](../role-based-access-control/overview.md)
* [Uso de asignaciones de roles para administrar el acceso a los recursos de la suscripción de Azure](../role-based-access-control/role-assignments-portal.md)

## <a name="understand-script-actions"></a>Descripción de las acciones de script

Una acción de script es un script de Bash que se ejecuta en los nodos de un clúster de HDInsight. Estas son algunas características de las acciones de script:

* Deben almacenarse en un URI accesible desde el clúster de HDInsight. A continuación, se proponen varias ubicaciones de almacenamiento posibles:

    * Para los clústeres normales:

      * ADLS Gen1: La entidad de servicio que HDInsight usa para acceder a Data Lake Storage debe tener acceso de lectura al script. El formato del identificador URI para scripts almacenados en Data Lake Storage Gen1 es `adl://DATALAKESTOREACCOUNTNAME.azuredatalakestore.net/path_to_file`.

      * Un blob de una cuenta de Azure Storage que sea la cuenta de almacenamiento principal o adicional del clúster de HDInsight. Durante la creación del clúster se concede acceso a HDInsight a estos dos tipos de cuentas de almacenamiento.

        > [!IMPORTANT]  
        > No cambie la clave de almacenamiento de esta cuenta de Azure Storage, ya que causará errores en las acciones de script subsecuentes de los scripts allí almacenados.

      * Existe un servicio de uso compartido de archivos al que se accede mediante rutas de acceso http://. Algunos ejemplos son Azure Blob, GitHub y OneDrive. Para obtener URI de ejemplo, consulte [Ejemplo de scripts de acción de script](#example-script-action-scripts).

     * Para clústeres con ESP, se admiten los URI wasb://, wasbs:// o http[s]://.

* Pueden restringirse para ejecutarse solo en determinados tipos de nodos. Algunos ejemplos son los nodos principales o nodos de trabajo.

* Pueden ser persistentes o `ad hoc`.

    Las acciones de scripts persistentes deben tener un nombre único. Los scripts persistentes se usan para personalizar nuevos nodos de trabajo agregados al clúster mediante operaciones de escalado. Un script persistente también puede aplicar cambios a otro tipo de nodo cuando se producen operaciones de escalado. Un ejemplo es un nodo principal.

    Los scripts `Ad hoc` no son persistentes. Las acciones de script usadas durante la creación de un clúster se guardan automáticamente. No se aplican a los nodos de trabajo agregados al clúster después de la ejecución del script. Posteriormente, un script `ad hoc` se puede promover a persistente y uno persistente puede disminuirse a `ad hoc`. Los scripts que dan error no se guardan como persistentes, aunque indique específicamente que así sea.

* Pueden aceptar parámetros que usa el script durante la ejecución.

* Se ejecutan con privilegios de nivel raíz en los nodos del clúster.

* Se pueden usar mediante Azure Portal, Azure PowerShell, la CLI de Azure o el SDK de .NET para HDInsight.

* Las acciones de script que quitan o modifican archivos de servicio en la VM pueden afectar al estado y la disponibilidad del servicio.

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

Puede realizar operaciones, como detener e iniciar servicios, incluidos los servicios relacionados con Apache Hadoop. Si detiene servicios, asegúrese de que los servicios de Ambari y otros relacionados con Hadoop estén en ejecución antes de que finalice el script. Estos servicios necesarios determinan el estado del clúster mientras se crea.

Durante la creación del clúster, puede usar diversas acciones de script a la vez. Estos scripts se invocan en el orden en el que se hayan especificado.

> [!IMPORTANT]  
> Las acciones de script deben finalizar en 60 minutos o agotarán el tiempo de espera. Durante el aprovisionamiento del clúster, el script se ejecuta a la vez con otros procesos de instalación y configuración. La competición por los recursos, como el ancho de banda de red o el tiempo de CPU, puede ocasionar que el script tarde más en terminar que en el entorno de desarrollo.
>
> Para minimizar el tiempo necesario para ejecutar el script, evite tareas tales como descargar y compilar aplicaciones desde el origen. Compile las aplicaciones previamente y almacene el archivo binario en Azure Storage.

### <a name="script-action-on-a-running-cluster"></a>Acción de script en un clúster en ejecución

Un error del script en un clúster ya en ejecución no hace que el clúster cambie automáticamente a un estado de error. Cuando un script finaliza, el clúster debe volver a un estado en ejecución. Aunque el clúster tenga el estado en ejecución, el script erróneo puede haber roto algo. Por ejemplo, un script podría eliminar archivos necesarios para el clúster.

Las acciones de script se ejecutan con privilegios raíz. Asegúrese de que comprende lo que hace un script antes de aplicarlo a un clúster.

Al aplicar un script a un clúster, se cambia el estado del clúster de **En ejecución** a **Aceptado**. A continuación, cambia a **Configuración de HDInsight** y, por último, regresa a **En ejecución** en el caso de los scripts correctos. El estado del script se registra en el historial de acciones de script. Esta información le indica si el script se ha completado correctamente o no. Por ejemplo, el cmdlet `Get-AzHDInsightScriptActionHistory` de PowerShell muestra el estado de un script. Devuelve información similar al texto siguiente:

```output
ScriptExecutionId : 635918532516474303
StartTime         : 8/14/2017 7:40:55 PM
EndTime           : 8/14/2017 7:41:05 PM
Status            : Succeeded
```

> [!IMPORTANT]  
> Si cambia la contraseña de usuario (admin) del clúster tras la creación de este, se puede producir un error en las acciones de script ejecutadas en este clúster. Si tiene alguna acción de script persistente cuyo destino sea algún nodo de trabajo, se puede producir un error en este script al escalar el clúster.

## <a name="example-script-action-scripts"></a>Ejemplo de scripts de acción de script

Los scripts de acciones de script se pueden usar con las utilidades siguientes:

* Azure portal
* Azure PowerShell
* Azure CLI
* SDK .NET de HDInsight

HDInsight proporciona scripts para instalar los siguientes componentes en clústeres de HDInsight:

| Nombre | Script |
| --- | --- |
| Adición de una cuenta de Azure Storage |`https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh`. Consulte [Adición de más cuentas de almacenamiento a HDInsight](hdinsight-hadoop-add-storage.md). |
| Instalación de Hue |`https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh`. Consulte [Instalación y uso de Hue en clústeres de Hadoop para HDInsight](hdinsight-hadoop-hue-linux.md). |
| Carga previa de las bibliotecas de Hive |`https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh`. Consulte [Incorporación de bibliotecas personalizadas de Apache Hive al crear el clúster de HDInsight](hdinsight-hadoop-add-hive-libraries.md). |

## <a name="script-action-during-cluster-creation"></a>Acción de script durante la creación de un clúster

En esta sección se explican las distintas maneras de usar acciones de script al crear un clúster de HDInsight.

### <a name="use-a-script-action-during-cluster-creation-from-the-azure-portal"></a>Uso de una acción de script durante la creación de un clúster desde Azure Portal

1. Empiece por crear un clúster según se describe en [Creación de clústeres basados en Linux en HDInsight con Azure Portal](hdinsight-hadoop-create-linux-clusters-portal.md). En la pestaña **Configuración y precios**, seleccione **+ Agregar acción de script**.

    ![Acción de script de clúster en Azure Portal](./media/hdinsight-hadoop-customize-cluster-linux/azure-portal-cluster-configuration-scriptaction.png)

1. Use la entrada __Seleccione un script__ para seleccionar un script prefabricado. Para utilizar un script personalizado, seleccione __Personalizado__. A continuación, proporcione el __nombre__ y el __URI de script de Bash__ del script.

    ![Agregar un script en el formulario de selección de script](./media/hdinsight-hadoop-customize-cluster-linux/hdinsight-select-script.png)

    En la siguiente tabla se describen los elementos del formulario:

    | Propiedad | Value |
    | --- | --- |
    | Seleccione un script | Para utilizar su propio script, seleccione __Custom__ (Personalizado). En caso contrario, seleccione uno de los que se proporcionan. |
    | Nombre |Especifique un nombre para la acción de script. |
    | URI de script de Bash |Especifique el URI del script. |
    | Head, Worker o ZooKeeper |Especifique los nodos en los que se ejecuta el script: **Head**, **Worker** o **ZooKeeper**. |
    | Parámetros |Especifique los parámetros, si lo requiere el script. |

    Use la entrada __Continuar con esta acción de script__ para asegurarse de que el script se aplica durante las operaciones de escalado.

1. Seleccione __Crear__ para guardar el script. Luego puede usar __+ Enviar nuevo__ para agregar otro script.

    ![Varias acciones de script de HDInsight](./media/hdinsight-hadoop-customize-cluster-linux/multiple-scripts-actions.png)

    Cuando haya terminado de agregar scripts, volverá a la pestaña **Configuración y precios**.

1. Complete los pasos de creación del clúster restantes como de costumbre.

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

En esta sección, se usa el cmdlet [Add-AzHDInsightScriptAction](https://docs.microsoft.com/powershell/module/az.hdinsight/add-azhdinsightscriptaction) para invocar scripts con el fin de personalizar un clúster. Antes de empezar, asegúrese de instalar y configurar Azure PowerShell. Para usar estos comandos de PowerShell, necesitará el [módulo AZ](https://docs.microsoft.com/powershell/azure/).

El script siguiente muestra cómo aplicar una acción de script al crear un clúster con PowerShell:

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=5-90)]

El clúster puede tardar varios minutos en crearse.

### <a name="use-a-script-action-during-cluster-creation-from-the-hdinsight-net-sdk"></a>Uso de una acción de script durante la creación de un clúster desde el SDK de .NET para HDInsight

El .NET SDK de HDInsight proporciona bibliotecas de cliente que facilitan el trabajo con HDInsight desde una aplicación .NET. Para obtener un ejemplo de código, vea [Acciones de script](https://docs.microsoft.com/dotnet/api/overview/azure/hdinsight?view=azure-dotnet#script-actions).

## <a name="script-action-to-a-running-cluster"></a>Acción de script a un clúster en ejecución

En esta sección se explica cómo aplicar acciones de script a un clúster en ejecución.

### <a name="apply-a-script-action-to-a-running-cluster-from-the-azure-portal"></a>Aplicación de una acción de script a un clúster en ejecución desde Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y localice su clúster.

1. En la vista predeterminada, en **Configuración**, seleccione **Acciones de script**.

1. En la parte superior de la página **Acciones de script**, seleccione **+ Enviar nuevo**.

    ![Agregar un script a un clúster en ejecución](./media/hdinsight-hadoop-customize-cluster-linux/add-script-running-cluster.png)

1. Use la entrada __Seleccione un script__ para seleccionar un script prefabricado. Para utilizar un script personalizado, seleccione __Personalizado__. A continuación, proporcione el __nombre__ y el __URI de script de Bash__ del script.

    ![Agregar un script en el formulario de selección de script](./media/hdinsight-hadoop-customize-cluster-linux/hdinsight-select-script.png)

    En la siguiente tabla se describen los elementos del formulario:

    | Propiedad | Value |
    | --- | --- |
    | Seleccione un script | Para utilizar su propio script, seleccione __Custom__ (Personalizado). En caso contrario, seleccione uno de los que se proporcionan. |
    | Nombre |Especifique un nombre para la acción de script. |
    | URI de script de Bash |Especifique el URI del script. |
    | Head, Worker o ZooKeeper |Especifique los nodos en los que se ejecuta el script: **Head**, **Worker** o **ZooKeeper**. |
    | Parámetros |Especifique los parámetros, si lo requiere el script. |

    Use la entrada __Continuar con esta acción de script__ para asegurarse de que el script se aplica durante las operaciones de escalado.

1. Por último, seleccione el botón **Crear** para aplicar el script al clúster.

### <a name="apply-a-script-action-to-a-running-cluster-from-azure-powershell"></a>Aplicación de una acción de script a un clúster en ejecución desde Azure PowerShell

Para usar estos comandos de PowerShell, necesitará el [módulo AZ](https://docs.microsoft.com/powershell/azure/). El ejemplo siguiente muestra cómo aplicar una acción de script a un clúster en ejecución:

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=105-117)]

Una vez terminada la operación, recibirá información similar al siguiente texto:

```output
OperationState  : Succeeded
ErrorMessage    :
Name            : Giraph
Uri             : https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh
Parameters      :
NodeTypes       : {HeadNode, WorkerNode}
```

### <a name="apply-a-script-action-to-a-running-cluster-from-the-azure-cli"></a>Aplicación de una acción de script a un clúster en ejecución desde la CLI de Azure

Antes de empezar, asegúrese de instalar y configurar la CLI de Azure. Asegúrese de disponer de la versión más reciente. Para más información, consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).

1. Autentíquese en la suscripción de Azure:

    ```azurecli
    az login
    ```

1. Aplique una acción de script a un clúster en ejecución:

    ```azurecli
    az hdinsight script-action execute --cluster-name CLUSTERNAME --name SCRIPTNAME --resource-group RESOURCEGROUP --roles ROLES
    ```

    Los roles válidos son `headnode`, `workernode`, `zookeepernode` y `edgenode`. Si el script se debe aplicar a varios tipos de nodo, especifique los roles con espacios. Por ejemplo, `--roles headnode workernode`.

    Para guardar el script como persistente, agregue `--persist-on-success`. También puede guardar el script como persistente más adelante mediante `az hdinsight script-action promote`.

### <a name="apply-a-script-action-to-a-running-cluster-by-using-rest-api"></a>Aplicación de una acción de script a un clúster en ejecución mediante la API de REST

Consulte [API de REST del clúster en Azure HDInsight](https://msdn.microsoft.com/library/azure/mt668441.aspx).

### <a name="apply-a-script-action-to-a-running-cluster-from-the-hdinsight-net-sdk"></a>Aplicación de una acción de script a un clúster en ejecución desde el SDK de .NET para HDInsight

Si desea ver un ejemplo de uso del SDK de .NET para aplicar scripts a un clúster, consulte cómo [aplicar una acción de script en un clúster de HDInsight basado en Linux en ejecución](https://github.com/Azure-Samples/hdinsight-dotnet-script-action).

## <a name="view-history-and-promote-and-demote-script-actions"></a>Visualización del historial, promoción y disminución de acciones de script

### <a name="the-azure-portal"></a>El Portal de Azure

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y localice su clúster.

1. En la vista predeterminada, en **Configuración**, seleccione **Acciones de script**.

1. En la sección Acciones de script aparece un historial de scripts de este clúster. Esta información incluye una lista de scripts persistentes. La siguiente captura de pantalla muestra que el script Solr se ha ejecutado en este clúster. La captura de pantalla no muestra ningún script persistente.

    ![Historial de accione de envío de script del portal](./media/hdinsight-hadoop-customize-cluster-linux/script-action-history.png)

1. Seleccione un script del historial para mostrar la sección **Propiedades** de ese script. Desde la parte superior de la pantalla, puede volver a ejecutar el script o subirlo de nivel.

    ![Propiedades de acciones de script: promover](./media/hdinsight-hadoop-customize-cluster-linux/promote-script-actions.png)

1. También puede usar los puntos suspensivos ( **...** ) a la derecha de las entradas de la sección Acciones de script para realizar acciones.

    ![Eliminación de acciones de script persistentes](./media/hdinsight-hadoop-customize-cluster-linux/hdi-delete-promoted-sa.png)

### <a name="azure-powershell"></a>Azure PowerShell

| cmdlet | Función |
| --- | --- |
| `Get-AzHDInsightPersistedScriptAction` |Recuperar información sobre acciones de script persistentes. Este cmdlet no deshace las acciones realizadas por un script, solo quita la marca persistente.|
| `Get-AzHDInsightScriptActionHistory` |Recuperar un historial de acciones de script aplicadas al clúster o detalles de un script específico. |
| `Set-AzHDInsightPersistedScriptAction` |Promover una acción de script `ad hoc` a una acción de script persistente. |
| `Remove-AzHDInsightPersistedScriptAction` |Disminuir de nivel una acción de script persistente a una acción `ad hoc`. |

El siguiente script de ejemplo muestra cómo utilizar los cmdlets para promover y luego disminuir de nivel un script.

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=123-140)]

### <a name="azure-cli"></a>Azure CLI

| Get-Help | Descripción |
| --- | --- |
| [`az hdinsight script-action delete`](https://docs.microsoft.com/cli/azure/hdinsight/script-action?view=azure-cli-latest#az-hdinsight-script-action-delete) |Elimina una acción de script persistente específica en un clúster determinado. Este comando no deshace las acciones realizadas por un script, solo quita la marca persistente.|
|[`az hdinsight script-action execute`](https://docs.microsoft.com/cli/azure/hdinsight/script-action?view=azure-cli-latest#az-hdinsight-script-action-execute)|Ejecución de acciones de script en el clúster de HDInsight especificado.|
| [`az hdinsight script-action list`](https://docs.microsoft.com/cli/azure/hdinsight/script-action?view=azure-cli-latest#az-hdinsight-script-action-list) |Muestra una lista de todas las acciones de script persistentes para el clúster especificado. |
|[`az hdinsight script-action list-execution-history`](https://docs.microsoft.com/cli/azure/hdinsight/script-action?view=azure-cli-latest#az-hdinsight-script-action-list-execution-history)|Muestra una lista del historial de ejecución de todos los scripts para el clúster especificado.|
|[`az hdinsight script-action promote`](https://docs.microsoft.com/cli/azure/hdinsight/script-action?view=azure-cli-latest#az-hdinsight-script-action-promote)|Promueve la ejecución de script ad hoc especificada a un script persistente.|
|[`az hdinsight script-action show-execution-details`](https://docs.microsoft.com/cli/azure/hdinsight/script-action?view=azure-cli-latest#az-hdinsight-script-action-show-execution-details)|Obtiene los detalles de ejecución del script para el identificador de ejecución de script dado.|

### <a name="hdinsight-net-sdk"></a>SDK .NET de HDInsight

Si desea ver un ejemplo de uso del SDK de .NET para recuperar el historial de scripts de un clúster, o bien promover scripts o disminuir el nivel de estos, consulte cómo [aplicar una acción de script en un clúster de HDInsight basado en Linux en ejecución](https://github.com/Azure-Samples/hdinsight-dotnet-script-action).

> [!NOTE]  
> En este ejemplo también se muestra cómo instalar una aplicación de HDInsight mediante el SDK. de NET.

## <a name="next-steps"></a>Pasos siguientes

* [Desarrollo de la acción de script con HDInsight](hdinsight-hadoop-script-actions-linux.md)
* [Adición de almacenamiento adicional a un clúster HDInsight](hdinsight-hadoop-add-storage.md)
* [Solución de problemas de acciones de script](troubleshoot-script-action.md)

[img-hdi-cluster-states]: ./media/hdinsight-hadoop-customize-cluster-linux/cluster-provisioning-states.png "Fases durante la creación del clúster"
