---
title: 'Ejecución de tareas en cuentas de usuario: Azure Batch | Microsoft Docs'
description: Configure cuentas de usuario para ejecutar tareas en Azure Batch
services: batch
author: laurenhughes
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 05/22/2017
ms.author: lahugh
ms.custom: seodec18
ms.openlocfilehash: bd5c16d755ef9b71f36b3d499838b12e6099ba6d
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595385"
---
> [!NOTE] 
> Las cuentas de usuario descritas en este artículo son diferentes de los usuarios las cuentas utilizadas para el protocolo de escritorio remoto (RDP) o Secure Shell (SSH), por motivos de seguridad. 
>
> Para conectarse a un nodo que ejecuta la configuración de máquina virtual Linux mediante SSH, consulte [Uso del escritorio remoto a una máquina virtual Linux en Azure](../virtual-machines/virtual-machines-linux-use-remote-desktop.md). Para conectarse a nodos que ejecutan Windows a través de RDP, consulte [Conexión a una máquina virtual de Windows Server](../virtual-machines/windows/connect-logon.md).<br /><br />
> Para conectarse a un nodo que ejecuta la configuración del servicio en la nube a través de RDP, consulte [Habilitación de la conexión a Escritorio remoto para un rol de Azure Cloud Services](../cloud-services/cloud-services-role-enable-remote-desktop-new-portal.md).
>
>


# <a name="run-tasks-under-user-accounts-in-batch"></a>Ejecución de tareas en cuentas de usuario en Batch

En Azure Batch, las tareas siempre se ejecutan en una cuenta de usuario. De forma predeterminada, las tareas se ejecutan en cuentas de usuario estándar, sin permisos de administrador. Esta configuración de cuenta de usuario predeterminada es normalmente suficiente. Sin embargo, para determinados escenarios, resulta útil poder configurar la cuenta de usuario en que desea que se ejecute una tarea. En este artículo se describen los tipos de cuentas de usuario y cómo puede configurarlos para su escenario.

## <a name="types-of-user-accounts"></a>Tipos de cuentas de usuario

Azure Batch proporciona dos tipos de cuentas de usuario para ejecutar tareas:

- **Cuentas de usuario automático.** Las cuentas de usuario automático son cuentas de usuario integradas que el servicio Batch crea automáticamente. De forma predeterminada, las tareas se ejecutan en una cuenta de usuario automático. Puede configurar la especificación de usuario automático de una tarea para indicar en qué cuenta de usuario automático se debería ejecutar esta. Además, permite especificar el nivel de elevación y el ámbito de la cuenta de usuario automático que ejecutará la tarea. 

- **Una cuenta de usuario con nombre.** Puede especificar una o varias cuentas de usuario con nombre para un grupo cuando lo crea. Se crea una cuenta de usuario en cada nodo del grupo. Además del nombre de la cuenta, especifique la contraseña de la cuenta de usuario, el nivel de elevación y, para grupos de Linux, la clave privada SSH. Cuando agregue una tarea, puede especificar la cuenta de usuario con nombre en que la tarea se debe ejecutar.

> [!IMPORTANT] 
> La versión 2017-01-01.4.0 del servicio Batch incluye un cambio importante que requiere que se actualice el código para llamar a esa versión. Si va a migrar código desde una versión anterior de Batch, tenga en cuenta que la propiedad **runElevated** ya no se admite en las bibliotecas de cliente de la API de REST o Batch. Use la nueva propiedad **userIdentity** de una tarea para especificar el nivel de elevación. Consulte la sección titulada [Actualización del código a la biblioteca de cliente de Batch más reciente](#update-your-code-to-the-latest-batch-client-library) para ver instrucciones rápidas para actualizar el código de Batch si usa una de las bibliotecas de cliente.
>
>

## <a name="user-account-access-to-files-and-directories"></a>Acceso de la cuenta de usuario a archivos y directorios

Tanto las cuentas de usuario automático como las de usuario con nombre tienen acceso de lectura y escritura al directorio de trabajo, el directorio compartido y el directorio de tareas de instancias múltiples de la tarea. Ambos tipos de cuenta tienen acceso de lectura a los directorios de inicio y preparación del trabajo.

Si una tarea se ejecuta en la misma cuenta que se usó para ejecutar una tarea de inicio, la tarea tiene acceso de lectura y escritura al directorio de la tarea de inicio. De forma similar, si una tarea se ejecuta en la misma cuenta que se usó para ejecutar una tarea de preparación del trabajo, la tarea tiene acceso de lectura y escritura al directorio de preparación del trabajo de la tarea. Si una tarea se ejecuta en una cuenta diferente a la de la tarea de inicio o de preparación del trabajo, la tarea solo tiene acceso de lectura al directorio correspondiente.

Para más información sobre cómo acceder a archivos y directorios desde una tarea, consulte [Desarrollo de soluciones de procesos paralelos a gran escala con Batch](batch-api-basics.md#files-and-directories).

## <a name="elevated-access-for-tasks"></a>Acceso con privilegios elevados para tareas 

El nivel de elevación de la cuenta de usuario indica si una tarea se ejecuta con acceso con privilegios elevados. Tanto las cuentas de usuario automático como las de usuario con nombre se pueden ejecutar con acceso con privilegios elevados. Las dos opciones para el nivel de elevación son:

- **NonAdmin:** la tarea se ejecuta como un usuario estándar sin acceso con privilegios elevados. El nivel de elevación predeterminado para una cuenta de usuario de Batch es siempre **NonAdmin**.
- **Admin:** la tarea se ejecuta como un usuario con acceso con privilegios elevados y funciona con permisos de administrador completos. 

## <a name="auto-user-accounts"></a>Cuentas de usuario automático

De forma predeterminada, las tareas se ejecutan en Batch en una cuenta de usuario automático, como usuario estándar sin acceso con privilegios elevados y con el ámbito de la tarea. Cuando se configura la especificación de usuario automático para el ámbito de la tarea, el servicio Batch crea una cuenta de usuario automático solo para esa tarea.

La alternativa al ámbito de la tarea es el ámbito de grupo. Cuando se configura la especificación de usuario automático de una tarea para el ámbito de grupo, la tarea se ejecuta en una cuenta de usuario automático que está disponible para cualquier tarea del grupo. Para más información acerca del ámbito del grupo, consulte la sección Ejecución de una tarea como usuario automático con ámbito de grupo.   

El ámbito predeterminado es diferente en los nodos de Windows y Linux:

- En los nodos de Windows, las tareas se ejecutan en el ámbito de la tarea de forma predeterminada.
- En los nodos de Linux, se ejecutan siempre en el ámbito de grupo.

Existen cuatro configuraciones posibles para la especificación de usuario automático, cada una de las cuales corresponde a una cuenta de usuario automático única:

- Acceso sin privilegios de administrador con ámbito de la tarea (la especificación de usuario automático predeterminada)
- Acceso de administrador (elevado) con ámbito de la tarea
- Acceso sin privilegios de administrador con ámbito de grupo
- Acceso de administrador con ámbito de grupo

> [!IMPORTANT] 
> Las tareas que se ejecutan en el ámbito de la tarea no tienen acceso de hecho a otras tareas en un nodo. Sin embargo, un usuario malintencionado con acceso a la cuenta podría eludir esta restricción enviando una tarea que se ejecuta con privilegios de administrador y accediendo a los otros directorios de tareas. Un usuario malintencionado también podría utilizar RDP o SSH para conectarse a un nodo. Es importante proteger el acceso a las claves de la cuenta de Batch para evitar este escenario. Si sospecha que su cuenta está en peligro, asegúrese de volver a generar las claves.
>
>

### <a name="run-a-task-as-an-auto-user-with-elevated-access"></a>Ejecución de una tarea como usuario automático con acceso con privilegios elevados

Puede configurar la especificación de usuario automático con privilegios de administrador cuando necesite ejecutar una tarea con acceso con privilegios elevados. Por ejemplo, una tarea de inicio puede necesitar acceso con privilegios elevados para instalar software en el nodo.

> [!NOTE] 
> Por lo general, es mejor usar el acceso con privilegios elevados solo cuando sea necesario. Según los procedimientos recomendados, es aconsejable conceder los privilegios mínimos necesarios para lograr el resultado deseado. Por ejemplo, si una tarea de inicio instala software para el usuario actual, en lugar de para todos los usuarios, es posible que pueda evitar conceder acceso con privilegios elevados a tareas. Puede configurar la especificación de usuario automático con ámbito de grupo y sin acceso con privilegios elevados para todas las tareas que haya que ejecutar en la misma cuenta, incluida la de inicio. 
>
>

En los fragmentos de código siguientes, se muestra cómo configurar la especificación de usuario automático. En los ejemplos se establece el nivel de elevación en `Admin` y el ámbito en `Task`. El ámbito de tarea es la configuración predeterminada, pero se incluye aquí a modo informativo.

#### <a name="batch-net"></a>.NET de Batch

```csharp
task.UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.Admin, scope: AutoUserScope.Task));
```
#### <a name="batch-java"></a>Java de Batch

```java
taskToAdd.withId(taskId)
        .withUserIdentity(new UserIdentity()
            .withAutoUser(new AutoUserSpecification()
                .withElevationLevel(ElevationLevel.ADMIN))
                .withScope(AutoUserScope.TASK));
        .withCommandLine("cmd /c echo hello");                        
```

#### <a name="batch-python"></a>Python de Batch

```python
user = batchmodels.UserIdentity(
    auto_user=batchmodels.AutoUserSpecification(
        elevation_level=batchmodels.ElevationLevel.admin,
        scope=batchmodels.AutoUserScope.task))
task = batchmodels.TaskAddParameter(
    id='task_1',
    command_line='cmd /c "echo hello world"',
    user_identity=user)
batch_client.task.add(job_id=jobid, task=task)
```

### <a name="run-a-task-as-an-auto-user-with-pool-scope"></a>Ejecución de una tarea como usuario automático con ámbito de grupo

Cuando se aprovisiona un nodo, se crean dos cuentas de usuario automático para todo el grupo en cada nodo del grupo, una con acceso con privilegios elevados y otra sin él. Si se establece el ámbito del usuario automático en el ámbito de grupo para una tarea determinada, se ejecuta la tarea en una de estas dos cuentas de usuario automático para todo el grupo. 

Cuando se especifica el ámbito de grupo para el usuario automático, todas las tareas que se ejecutan con acceso de administrador se ejecutan en la misma cuenta de usuario automático para todo el grupo. De forma similar, las tareas que se ejecutan sin permisos de administrador también se ejecutan en una sola cuenta de usuario automático para todo el grupo. 

> [!NOTE] 
> Las dos cuentas de usuario automático para todo el grupo son independientes. Las tareas que se ejecutan en la cuenta administrativa para todo el grupo no pueden compartir datos con las que se ejecutan en la cuenta estándar y viceversa. 
>
>

La ventaja de ejecutarlas en la misma cuenta de usuario automático es que las tareas pueden compartir datos con otras tareas que se ejecuten en el mismo nodo.

Un escenario en que se comparten secretos entre tareas es un ejemplo en el que resulta útil ejecutar tareas en una de las dos cuentas de usuario automático para todo el grupo. Por ejemplo, suponga que una tarea de inicio necesita aprovisionar un secreto en el nodo que pueden usar otras tareas. Podría usar la API de protección de datos de Windows (DPAPI), pero requiere privilegios de administrador. En su lugar, puede proteger el secreto en el nivel de usuario. Las tareas que se ejecutan en la misma cuenta de usuario pueden acceder al secreto sin necesidad del acceso con privilegios elevados.

Otro escenario donde podría ejecutar tareas en una cuenta de usuario automático con ámbito de grupo es un recurso compartido de archivos de interfaz de paso de mensajes (MPI). Un recurso compartido de archivos MPI es útil cuando los nodos de la tarea MPI necesitan trabajar en los mismos datos de archivo. El nodo principal crea un recurso compartido de archivos al que los nodos secundarios pueden acceder si se ejecutan en la misma cuenta de usuario automático. 

En el fragmento de código siguiente se establece el ámbito del usuario automático en el ámbito de grupo para una tarea en .NET de Batch. Se omite el nivel de elevación, de forma que la tarea se ejecuta en la cuenta de usuario automático estándar para todo el grupo.

```csharp
task.UserIdentity = new UserIdentity(new AutoUserSpecification(scope: AutoUserScope.Pool));
```

## <a name="named-user-accounts"></a>Cuentas de usuario con nombre

Puede definir cuentas de usuario con nombre cuando cree un grupo. Una cuenta de usuario con nombre tiene el nombre y la contraseña que proporcione. Puede especificar el nivel de elevación para una cuenta de usuario con nombre. Para los nodos de Linux, también puede proporcionar una clave privada SSH.

Existe una cuenta de usuario con nombre en todos los nodos del grupo y está disponible para todas las tareas que se ejecuten en esos nodos. Puede definir cualquier número de usuarios con nombre para un grupo. Cuando agregue una tarea o una colección de tareas, puede especificar que la tarea se ejecute en una de las cuentas de usuario con nombre definidas en el grupo.

Una cuenta de usuario con nombre es útil cuando desea ejecutar todas las tareas de un trabajo en la misma cuenta de usuario, al mismo tiempo que las mantiene aisladas de las tareas que se ejecutan en otros trabajos. Por ejemplo, puede crear un usuario con nombre para cada trabajo y ejecutar las tareas de cada trabajo en dicha cuenta de usuario con nombre. Entonces, cada trabajo puede compartir un secreto con sus propias tareas, pero no con las que se ejecutan en otros trabajos.

También puede usar una cuenta de usuario con nombre para ejecutar una tarea que establezca permisos en recursos externos, como recursos compartidos de archivos. Con una cuenta de usuario con nombre, controla la identidad del usuario y puede usarla para establecer permisos.  

Las cuentas de usuario con nombre permiten SSH sin contraseña entre nodos de Linux. Puede usar una cuenta de usuario con nombre con nodos de Linux que necesiten ejecutar tareas de instancias múltiples. Cada nodo del grupo puede ejecutar tareas en una cuenta de usuario definida en el grupo completo. Para más información acerca de las tareas de instancias múltiples, consulte [Uso de tareas de instancias múltiples para ejecutar aplicaciones MPI](batch-mpi.md).

### <a name="create-named-user-accounts"></a>Creación de cuentas de usuario con nombre

Para crear cuentas de usuario con nombre en Batch, agregue una colección de cuentas de usuario al grupo. En los fragmentos de código siguientes se muestra cómo crear cuentas de usuario con nombre en .NET, Java y Python. En estos fragmentos de código se muestra cómo crear cuentas con nombre con y sin derechos administrativos en un grupo. En los ejemplos se crean grupos que usan la configuración del servicio en la nube, pero se utiliza el mismo enfoque al crear un grupo de Windows o Linux con la configuración de máquina virtual.

#### <a name="batch-net-example-windows"></a>Ejemplo de .NET de Batch (Windows)

```csharp
CloudPool pool = null;
Console.WriteLine("Creating pool [{0}]...", poolId);

// Create a pool using the cloud service configuration.
pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 3,
    virtualMachineSize: "standard_d1_v2",
    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));   

// Add named user accounts.
pool.UserAccounts = new List<UserAccount>
{
    new UserAccount("adminUser", "xyz123", ElevationLevel.Admin),
    new UserAccount("nonAdminUser", "123xyz", ElevationLevel.NonAdmin),
};

// Commit the pool.
await pool.CommitAsync();
```

#### <a name="batch-net-example-linux"></a>Ejemplo de .NET de Batch (Linux)

```csharp
CloudPool pool = null;

// Obtain a collection of all available node agent SKUs.
List<NodeAgentSku> nodeAgentSkus =
    batchClient.PoolOperations.ListNodeAgentSkus().ToList();

// Define a delegate specifying properties of the VM image to use.
Func<ImageReference, bool> isUbuntu1404 = imageRef =>
    imageRef.Publisher == "Canonical" &&
    imageRef.Offer == "UbuntuServer" &&
    imageRef.Sku.Contains("14.04");

// Obtain the first node agent SKU in the collection that matches
// Ubuntu Server 14.04. 
NodeAgentSku ubuntuAgentSku = nodeAgentSkus.First(sku =>
    sku.VerifiedImageReferences.Any(isUbuntu1404));

// Select an ImageReference from those available for node agent.
ImageReference imageReference =
    ubuntuAgentSku.VerifiedImageReferences.First(isUbuntu1404);

// Create the virtual machine configuration to use to create the pool.
VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration(imageReference, ubuntuAgentSku.Id);

Console.WriteLine("Creating pool [{0}]...", poolId);

// Create the unbound pool.
pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 3,                                             
    virtualMachineSize: "Standard_A1",                                      
    virtualMachineConfiguration: virtualMachineConfiguration);                  

// Add named user accounts.
pool.UserAccounts = new List<UserAccount>
{
    new UserAccount(
        name: "adminUser",
        password: "xyz123",
        elevationLevel: ElevationLevel.Admin,
        linuxUserConfiguration: new LinuxUserConfiguration(
            uid: 12345,
            gid: 98765,
            sshPrivateKey: new Guid().ToString()
            )),
    new UserAccount(
        name: "nonAdminUser",
        password: "123xyz",
        elevationLevel: ElevationLevel.NonAdmin,
        linuxUserConfiguration: new LinuxUserConfiguration(
            uid: 45678,
            gid: 98765,
            sshPrivateKey: new Guid().ToString()
            )),
};

// Commit the pool.
await pool.CommitAsync();
```


#### <a name="batch-java-example"></a>Ejemplo de Java de Batch

```java
List<UserAccount> userList = new ArrayList<>();
userList.add(new UserAccount().withName(adminUserAccountName).withPassword(adminPassword).withElevationLevel(ElevationLevel.ADMIN));
userList.add(new UserAccount().withName(nonAdminUserAccountName).withPassword(nonAdminPassword).withElevationLevel(ElevationLevel.NONADMIN));
PoolAddParameter addParameter = new PoolAddParameter()
        .withId(poolId)
        .withTargetDedicatedNodes(POOL_VM_COUNT)
        .withVmSize(POOL_VM_SIZE)
        .withCloudServiceConfiguration(configuration)
        .withUserAccounts(userList);
batchClient.poolOperations().createPool(addParameter);
```

#### <a name="batch-python-example"></a>Ejemplo de Python de Batch

```python
users = [
    batchmodels.UserAccount(
        name='pool-admin',
        password='******',
        elevation_level=batchmodels.ElevationLevel.admin)
    batchmodels.UserAccount(
        name='pool-nonadmin',
        password='******',
        elevation_level=batchmodels.ElevationLevel.nonadmin)
]
pool = batchmodels.PoolAddParameter(
    id=pool_id,
    user_accounts=users,
    virtual_machine_configuration=batchmodels.VirtualMachineConfiguration(
        image_reference=image_ref_to_use,
        node_agent_sku_id=sku_to_use),
    vm_size=vm_size,
    target_dedicated=vm_count)
batch_client.pool.add(pool)
```

### <a name="run-a-task-under-a-named-user-account-with-elevated-access"></a>Ejecución de una tarea en una cuenta de usuario con nombre con acceso con privilegios elevados

Para ejecutar una tarea como usuario con privilegios elevados, establezca la propiedad **UserIdentity** de la tarea en una cuenta de usuario con nombre que se creó con la propiedad **ElevationLevel** establecida en `Admin`.

En este fragmento de código se especifica que la tarea debe ejecutarse en una cuenta de usuario con nombre. Esta cuenta de usuario con nombre se definió en el grupo cuando este se creó. En este caso, la cuenta de usuario con nombre se creó con permisos de administrador:

```csharp
CloudTask task = new CloudTask("1", "cmd.exe /c echo 1");
task.UserIdentity = new UserIdentity(AdminUserAccountName);
```

## <a name="update-your-code-to-the-latest-batch-client-library"></a>Actualización del código a la biblioteca de cliente de Batch más reciente

La versión 2017-01-01.4.0 del servicio Batch incluye un cambio importante: se reemplaza la propiedad **runElevated** disponible en versiones anteriores por la propiedad **userIdentity**. En las siguientes tablas se proporciona una asignación sencilla que sirve para actualizar el código de versiones anteriores de las bibliotecas de cliente.

### <a name="batch-net"></a>.NET de Batch

| Si el código usa…                  | Actualícelo a…                                                                                                 |
|---------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `CloudTask.RunElevated = true;`       | `CloudTask.UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.Admin));`    |
| `CloudTask.RunElevated = false;`      | `CloudTask.UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.NonAdmin));` |
| `CloudTask.RunElevated` sin especificar | No se requiere actualización                                                                                               |

### <a name="batch-java"></a>Java de Batch

| Si el código usa…                      | Actualícelo a…                                                                                                                       |
|-------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| `CloudTask.withRunElevated(true);`        | `CloudTask.withUserIdentity(new UserIdentity().withAutoUser(new AutoUserSpecification().withElevationLevel(ElevationLevel.ADMIN));`    |
| `CloudTask.withRunElevated(false);`       | `CloudTask.withUserIdentity(new UserIdentity().withAutoUser(new AutoUserSpecification().withElevationLevel(ElevationLevel.NONADMIN));` |
| `CloudTask.withRunElevated` sin especificar | No se requiere actualización                                                                                                                     |

### <a name="batch-python"></a>Python de Batch

| Si el código usa…                      | Actualícelo a…                                                                                                                       |
|-------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| `run_elevated=True`                       | `user_identity=user`, donde <br />`user = batchmodels.UserIdentity(`<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`auto_user=batchmodels.AutoUserSpecification(`<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`elevation_level=batchmodels.ElevationLevel.admin))`                |
| `run_elevated=False`                      | `user_identity=user`, donde <br />`user = batchmodels.UserIdentity(`<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`auto_user=batchmodels.AutoUserSpecification(`<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`elevation_level=batchmodels.ElevationLevel.nonadmin))`             |
| `run_elevated` sin especificar | No se requiere actualización                                                                                                                                  |


## <a name="next-steps"></a>Pasos siguientes

* Para información general más detallada acerca de Batch, consulte [Desarrollo de soluciones de procesos paralelos a gran escala con Batch](batch-api-basics.md).
