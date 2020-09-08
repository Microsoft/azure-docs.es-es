---
title: Solución de problemas de replicación en la migración de máquinas virtuales VMware sin agente
description: Obtenga ayuda para los errores del ciclo de replicación
author: anvar-ms
ms.manager: bsiva
ms.author: anvar
ms.topic: troubleshooting
ms.date: 08/17/2020
ms.openlocfilehash: 6318f426e42612f21da7a43c9857894ae610f68e
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88871194"
---
# <a name="troubleshooting-replication-issues-in-agentless-vmware-vm-migration"></a>Solución de problemas de replicación en la migración de máquinas virtuales VMware sin agente

En este artículo se describen algunos problemas comunes y errores específicos que pueden surgir al replicar máquinas virtuales de VMware locales con Azure Migrate: Método sin agente para la migración de servidores.

Cuando se replica una máquina virtual de VMware mediante el método de replicación sin agente, los datos de los discos de máquinas virtuales (vmdk) se replican en discos administrados de réplica en la suscripción de Azure. Cuando se inicia la replicación de una máquina virtual, se produce un ciclo de replicación inicial en el que se replican las copias completas de los discos. Una vez completada la replicación inicial, se programan periódicamente ciclos de replicación incremental para transferir los cambios que se hayan producido desde el ciclo de replicación anterior.

En ocasiones, es posible que vea errores en los ciclos de replicación de una máquina virtual. Estos errores pueden producirse debido a razones que varían desde problemas de configuración de red local a problemas en el back-end del servicio en la nube de Azure Migrate. En este artículo haremos lo siguiente:

 - Mostrarle cómo puede supervisar el estado de la replicación y resolver los errores.
 - Enumerar algunos de los errores de replicación que se producen con frecuencia y sugerir pasos adicionales para corregirlos.

## <a name="monitor-replication-status-using-the-azure-portal"></a>Supervisión del estado de la replicación mediante Azure Portal

Siga los pasos que se indican a continuación para supervisar el estado de replicación de las máquinas virtuales:

  1. Vaya a la página Servidores de Azure Migrate en Azure Portal.
  2. Para ir a la página "Replicación de máquinas", haga clic en "Replicando servidores" en el icono de migración del servidor.
  3. Verá una lista de servidores de replicación junto con información adicional, como el estado, el mantenimiento, la hora de la última sincronización, etc. La columna health (mantenimiento) indica el estado de replicación actual de la máquina virtual. Un valor "Crítical" (Crítico) o "Warning" (Advertencia) en la columna se estado suele indicar que se produjo un error en el ciclo de replicación anterior de la máquina virtual. Para obtener más información, haga clic con el botón derecho en la máquina virtual y seleccione "Error Details" (Detalles del error). La página "Error Details" (Detalles del error) contiene información sobre el error y detalles adicionales sobre cómo solucionar problemas. También verá un vínculo "Recent Events" (Eventos recientes) que se puede usar para ir a la página de eventos de la máquina virtual.
  4. Haga clic en "Recent Events" (Eventos recientes) para ver los errores anteriores del ciclo de replicación para la máquina virtual. En la página de eventos, busque el más reciente del tipo "Replication cycle failed" (Error en el ciclo de replicación) o "Replication cycle failed for disk" (Error en el ciclo de replicación para el disco) para la máquina virtual.
  5. Haga clic en el evento para conocer las posibles causas del error y los pasos de corrección recomendados. Utilice la información proporcionada para solucionar problemas y corregir el error.
    
## <a name="common-replication-errors"></a>Errores de replicación comunes

En esta sección se describen algunos de los errores comunes y cómo puede solucionarlos.

## <a name="key-vault-operation-failed-error-when-trying-to-replicate-vms"></a>Error en la operación de Key Vault al intentar replicar las máquinas virtuales

**Error:** “Key Vault operation failed. Operation : Configure managed storage account, Key Vault: Key-vault-name, Storage Account: storage account name failed with the error:” (Error de la operación de Key Vault. Operación: Configuración de la cuenta de almacenamiento administrada, Key Vault: nombre del almacén de claves, cuenta de almacenamiento: error del nombre de la cuenta de almacenamiento:)

**Error:** “Key Vault operation failed. Operation : Generate shared access signature definition, Key Vault: Key-vault-name, Storage Account: storage account name failed with the error:” (Error de la operación de Key Vault. Operación: Generación de la definición de la firma de acceso compartido, Key Vault: nombre del almacén de claves, cuenta de almacenamiento: error del nombre de la cuenta de almacenamiento:)

![Key Vault](./media/troubleshoot-changed-block-tracking-replication/key-vault.png)

Este error suele deberse a que la directiva de acceso de usuarios de Key Vault no proporciona al usuario que ha iniciado sesión los permisos necesarios para configurar las cuentas de almacenamiento que se van a administrar con Key Vault. Para comprobar la directiva de acceso de usuarios del almacén de claves, vaya a la página del almacén de claves en el portal de este servicio y seleccione las directivas de acceso. 

Cuando el portal crea el almacén de claves, también agrega una directiva de acceso de usuarios que concede al usuario que ha iniciado sesión los permisos para configurar las cuentas de almacenamiento que se van a administrar con Key Vault. En esta situación, pueden producirse errores por dos motivos

- El usuario que ha iniciado sesión es una entidad de seguridad remota en el inquilino de Azure de los clientes (una suscripción de CSP, y el usuario que ha iniciado sesión es el administrador del asociado). La solución alternativa en este caso es eliminar el almacén de claves, cerrar la sesión en el portal y, luego, iniciar sesión con una cuenta de usuario desde el inquilino de los clientes (no una entidad de seguridad remota) y volver a intentar la operación. Normalmente, el asociado de CSP tendrá una cuenta de usuario en el inquilino de Azure Active Directory de los clientes que puede usar. Si no, puede crear una cuenta de usuario propia en el inquilino de Azure Active Directory de los clientes, iniciar sesión en el portal como nuevo usuario y, luego, volver a intentar la operación de replicación. La cuenta usada debe tener concedidos permisos de propietario o colaborador además de los de administrador de acceso de usuarios a la cuenta en el grupo de recursos (migración del grupo de recursos del proyecto).

- El otro caso que puede darse es cuando un usuario (user1) intenta configurar la replicación inicialmente y encuentra un error, pero el almacén de claves ya se ha creado (y la directiva de acceso de usuarios se ha asignado como corresponde a este usuario). Ahora, más adelante, un usuario diferente (user2) intenta configurar la replicación, pero la operación de configuración de la cuenta de almacenamiento administrada o de generación de la definición de SAS produce un error, ya que no hay ninguna directiva de acceso de usuario correspondiente al usuario2 en el almacén de claves.

**Solución:** Para solucionar este problema, cree una directiva de acceso de usuarios para user2 en el permiso keyvault granting user2 para configurar la cuenta de almacenamiento administrada y generar definiciones de SAS. Para ello, user2 puede usar los siguientes cmdlets de Azure PowerShell:

$userPrincipalId = $(Get-AzureRmADUser -UserPrincipalName "user2_email_address").Id

Set-AzureRmKeyVaultAccessPolicy -VaultName "keyvaultname" -ObjectId $userPrincipalId -PermissionsToStorage get, list, delete, set, update, regeneratekey, getsas, listsas, deletesas, setsas, recover, backup, restore, purge


## <a name="disposeartefactstimedout"></a>DisposeArtefactsTimedOut

**Identificador del error:** 181008

**Mensaje de error:** Máquina virtual VMName. Error: Se encontró el evento de tiempo de espera "DisposeArtefactsTimeout" en el estado &'['Gateway.Service.StateMachine.SnapshotReplication.SnapshotReplicationEngine+WaitingForArtefactsDisposalPreCycle' ('WaitingForArtefactsDisposalPreCycle')]'.

**Causas posibles:**

El componente que intenta replicar datos en Azure está inactivo o no responde. Estas son algunas causas posibles:

- El servicio de puerta de enlace que se ejecuta en el dispositivo de Azure Migrate está inactivo.
- El servicio de puerta de enlace está experimentando problemas de conectividad con la cuenta de Service Bus, el centro de eventos o el almacenamiento de aplicaciones.

**Identificación de la causa exacta de DisposeArtefactsTimedOut y la resolución correspondiente:**

1. Asegúrese de que el dispositivo con Azure Migrate está en funcionamiento.
2. Compruebe si el servicio de puerta de enlace se está ejecutando en el dispositivo:
   1.  Inicie sesión en el dispositivo de Azure Migrate con el escritorio remoto y haga lo siguiente.

   2.  Abra el complemento MMC de servicios de Microsoft (ejecute > services.msc) y compruebe si el servicio Microsoft Azure Gateway se está ejecutando. Si el servicio está detenido o no se está ejecutando, inícielo. Como alternativa, puede abrir el símbolo del sistema o PowerShell, y ejecutar lo siguiente: "Net Start asrgwy"

3. Compruebe si hay problemas de conectividad entre el dispositivo de Azure Migrate y la cuenta de almacenamiento del dispositivo: 

    Ejecute el siguiente comando después de descargar azcopy en el dispositivo de Azure Migrate:
    
    _azcopy bench https://[account].blob.core.windows.net/[container]?SAS_
    
    **Pasos para ejecutar la prueba comparativa de rendimiento:**
    
      1. [Descargue](https://go.microsoft.com/fwlink/?linkid=2138966) azcopy
        
      2. Busque la cuenta de almacenamiento del dispositivo en el grupo de recursos. La cuenta de almacenamiento tiene un nombre similar a migrategwsa\*\*\*\*\*\*\*\*\*\*. Este es el valor del parámetro [account] (cuenta) en el comando anterior.
        
      3. Busque la cuenta de almacenamiento en Azure Portal. Asegúrese de que la suscripción que usa para buscar es la misma (suscripción de destino) en la que se crea la cuenta de almacenamiento. Vaya a Contenedores en la sección Blob Service. Haga clic en +Contenedor y cree un contenedor. Deje el nivel de acceso público en el valor predeterminado seleccionado.
        
      4. Vaya a Firma de acceso compartido en Configuración. Seleccione Contenedor en "Tipo de recurso permitido". Haga clic en Generar la cadena de conexión y SAS. Copie el valor de SAS.
        
      5. Ejecute el comando anterior en el símbolo del sistema; para ello, reemplace la cuenta, el contenedor y SAS con los valores obtenidos en los pasos 2, 3 y 4, respectivamente.
        
      También puede [descargar](https://go.microsoft.com/fwlink/?linkid=2138967) Azure Storage Explorer en el dispositivo e intentar cargar 10 blobs de ~64 MB en las cuentas de almacenamiento. Si no hay ningún problema, la carga debe realizarse correctamente.
        
    **Resolución:** si se produce un error en esta prueba, hay un problema de red. Póngase en contacto con su equipo de red local para comprobar si hay problemas de conectividad. Normalmente, puede haber algunos valores de configuración del firewall que causen los errores.
    
4.  Compruebe si hay problemas de conectividad entre el dispositivo de Azure Migrate y Service Bus:

    En esta prueba, se comprueba si el dispositivo de Azure Migrate puede comunicarse con el back-end del servicio en la nube de Azure Migrate. El dispositivo se comunica con el back-end del servicio mediante las colas de mensajes del centro de eventos y Service Bus. Para comprobar la conectividad entre el dispositivo y Service Bus, [descargue](https://go.microsoft.com/fwlink/?linkid=2139104) Service Bus Explorer, intente conectarse al dispositivo de Service Bus y ejecute el mensaje de envío/recepción. Si no hay ningún problema, debería ejecutarse de forma correcta.

    **Pasos para ejecutar la prueba:**

    1. Copie la cadena de conexión desde Service Bus que se ha creado en el proyecto de migración.
    2. Abra Service Bus Explorer.
    3. Vaya a Archivo y, a continuación, conéctese.
    4. Pegue la cadena de conexión y haga clic en Conectar.
    5. Se abrirá el espacio de nombres de Service Bus.
    6. Seleccione Snapshot Manager en el tema. Haga clic con el botón derecho en Snapshot Manager, seleccione "Receive Messages" (Recibir mensajes) > seleccione "peek" (ver el código sin salir) y haga clic en OK (Aceptar).
    7. Si la conexión se establece correctamente, verá "[x] messages received" ([x] mensajes recibidos) en la salida de la consola. Si no, verá un mensaje que indica que se ha producido un error en la conexión.
    
    **Resolución:** si se produce un error en esta prueba, hay un problema de red. Póngase en contacto con su equipo de red local para comprobar si hay problemas de conectividad. Normalmente, puede haber algunos valores de configuración del firewall que causen los errores.

5. Problemas de conectividad entre el dispositivo de Azure Migrate y Azure Key Vault:

    En esta prueba se examina si hay problemas de conectividad entre el dispositivo de Azure Migrate y Azure Key Vault. Key Vault se utiliza para administrar el acceso de la cuenta de almacenamiento que se usa para la replicación.
    
    **Pasos para comprobar la conectividad:**
    
    1. Recupere el URI de Key Vault de la lista de recursos del grupo de recursos correspondiente al proyecto de Azure Migrate.
    
    1. Abra PowerShell en el dispositivo de Azure Migrate y ejecute el siguiente comando:
    
    _test-netconnection Key Vault URI -P 443_
    
    Este comando intentará establecer una conexión TCP y devolverá una salida.
    
     - En la salida, compruebe el campo "_TcpTestSucceeded_". Si el valor es "_true_", no hay ningún problema de conectividad entre el dispositivo de Azure Migrate y Azure Key Vault. Si el valor es "false", hay un problema de conectividad.
    
    **Resolución:** si se produce un error en esta prueba, hay un problema de conectividad entre el dispositivo de Azure Migrate y Azure Key Vault. Póngase en contacto con su equipo de red local para comprobar si hay problemas de conectividad. Normalmente, puede haber algunos valores de configuración del firewall que causen los errores.
    
## <a name="diskuploadtimedout"></a>DiskUploadTimedOut

**Identificador del error:** 1011

**Mensaje de error:** la carga de datos para el disco DiskPath, DiskId de VMName de máquina virtual; VMId no se completó en el tiempo esperado.

Este error suele indicar que el dispositivo de Azure Migrate que realiza la replicación no puede conectarse a Azure Cloud Services o que la replicación progresa lentamente, lo que hace que se agote el tiempo de espera del ciclo de replicación.

Estas son algunas causas posibles:

- El dispositivo de Azure Migrate está apagado.
- El servicio de puerta de enlace de replicación en el dispositivo no se está ejecutando.
- El servicio de puerta de enlace de replicación está experimentando problemas de conectividad con uno de los siguientes componentes del servicio de Azure que se usan para la replicación: Service Bus, centro de eventos, cuenta de almacenamiento en caché de Azure, Azure Key Vault.
- Se está limitando el servicio de puerta de enlace en el nivel de vCenter mientras se intenta leer el disco.

**Identificación de la causa principal y resolución del problema:**

1. Asegúrese de que el dispositivo con Azure Migrate está en funcionamiento.
2. Compruebe si el servicio de puerta de enlace se está ejecutando en el dispositivo:
   1.  Inicie sesión en el dispositivo de Azure Migrate con el escritorio remoto y haga lo siguiente.

   2.  Abra el complemento MMC de servicios de Microsoft (ejecutar > services.msc) y compruebe si "Microsoft Azure Gateway Service"  (Servicio de puerta de enlace de Microsoft Azure) está ejecutándose. Si el servicio está detenido o no se está ejecutando, inícielo. Como alternativa, puede abrir el símbolo del sistema o PowerShell, y ejecutar lo siguiente: "Net Start asrgwy".


3. **Compruebe si hay problemas de conectividad entre el dispositivo de Azure Migrate y la cuenta de almacenamiento en caché:** 

    Ejecute el siguiente comando después de descargar azcopy en el dispositivo de Azure Migrate:
    
    _azcopy bench https://[account].blob.core.windows.net/[container]?SAS_
    
    **Pasos para ejecutar la prueba comparativa de rendimiento:**
    
      1. [Descargue](https://go.microsoft.com/fwlink/?linkid=2138966) azcopy
        
      2. Busque la cuenta de almacenamiento del dispositivo en el grupo de recursos. La cuenta de almacenamiento tiene un nombre similar a migratelsa\*\*\*\*\*\*\*\*\*\*. Este es el valor del parámetro [account] (cuenta) en el comando anterior.
        
      3. Busque la cuenta de almacenamiento en Azure Portal. Asegúrese de que la suscripción que usa para buscar es la misma (suscripción de destino) en la que se crea la cuenta de almacenamiento. Vaya a Contenedores en la sección Blob Service. Haga clic en +Contenedor y cree un contenedor. Deje el nivel de acceso público en el valor predeterminado seleccionado.
        
      4. Vaya a Firma de acceso compartido en Configuración. Seleccione Contenedor en "Tipo de recurso permitido". Haga clic en Generar la cadena de conexión y SAS. Copie el valor de SAS.
        
      5. Ejecute el comando anterior en el símbolo del sistema; para ello, reemplace la cuenta, el contenedor y SAS con los valores obtenidos en los pasos 2, 3 y 4, respectivamente.
        
      También puede [descargar](https://go.microsoft.com/fwlink/?linkid=2138967) Azure Storage Explorer en el dispositivo e intentar cargar 10 blobs de ~64 MB en las cuentas de almacenamiento. Si no hay ningún problema, la carga debe realizarse correctamente.
        
    **Resolución:** si se produce un error en esta prueba, hay un problema de red. Póngase en contacto con su equipo de red local para comprobar si hay problemas de conectividad. Normalmente, puede haber algunos valores de configuración del firewall que causen los errores.
                
4.  **Problemas de conectividad entre el dispositivo de Azure Migrate y Azure Service Bus:**

    En esta prueba se examinará si el dispositivo de Azure Migrate puede comunicarse con el back-end del servicio en la nube de Azure Migrate. El dispositivo se comunica con el back-end del servicio mediante las colas de mensajes del centro de eventos y Service Bus. Para comprobar la conectividad entre el dispositivo y Service Bus, [descargue](https://go.microsoft.com/fwlink/?linkid=2139104) Service Bus Explorer, intente conectarse al dispositivo de Service Bus y ejecute el mensaje de envío/recepción. Si no hay ningún problema, debería ejecutarse de forma correcta.

    **Pasos para ejecutar la prueba:**
    
    1. Copie la cadena de conexión del Service Bus que se obtuvo en el grupo de recursos correspondiente al proyecto de Azure Migrate.
    
    1. Abra Service Bus Explorer.
    
    1. Vaya a Archivo > Conectar.
    
    1. Pegue la cadena de conexión que copió en el paso 1 y haga clic en Conectar.
    
    1. Se abrirá el espacio de nombres de Service Bus.
    
    1. Seleccione Snapshot Manager en el tema del espacio de nombres. Haga clic con el botón derecho en Snapshot Manager, seleccione "Receive Messages" (Recibir mensajes) > seleccione "peek" (ver el código sin salir) y haga clic en OK (Aceptar).
    
    Si la conexión se establece correctamente, verá "[x] messages received" ([x] mensajes recibidos) en la salida de la consola. Si la conexión no se realiza correctamente, verá un mensaje que indica que se ha producido un error en la conexión.
    
    **Resolución:** si se produce un error en esta prueba, hay un problema de conectividad entre el dispositivo de Azure Migrate y Service Bus. Póngase en contacto con su equipo de red local para comprobar estos problemas de conectividad. Normalmente, puede haber algunos valores de configuración del firewall que causen los errores.
    
 5. **Problemas de conectividad entre el dispositivo de Azure Migrate y Azure Key Vault:**

    En esta prueba se examina si hay problemas de conectividad entre el dispositivo de Azure Migrate y Azure Key Vault. Key Vault se utiliza para administrar el acceso de la cuenta de almacenamiento que se usa para la replicación.
    
    **Pasos para comprobar la conectividad:**
    
    1. Recupere el URI de Key Vault de la lista de recursos del grupo de recursos correspondiente al proyecto de Azure Migrate.
    
    1. Abra PowerShell en el dispositivo de Azure Migrate y ejecute el siguiente comando:
    
    _test-netconnection Key Vault URI -P 443_
    
    Este comando intentará establecer una conexión TCP y devolverá una salida.
    
    1. En la salida, compruebe el campo "_TcpTestSucceeded_". Si el valor es "_true_", no hay ningún problema de conectividad entre el dispositivo de Azure Migrate y Azure Key Vault. Si el valor es "false", hay un problema de conectividad.
    
    **Resolución:** si se produce un error en esta prueba, hay un problema de conectividad entre el dispositivo de Azure Migrate y Azure Key Vault. Póngase en contacto con su equipo de red local para comprobar si hay problemas de conectividad. Normalmente, puede haber algunos valores de configuración del firewall que causen los errores.
    
## <a name="encountered-an-error-while-trying-to-fetch-changed-blocks"></a>Error al intentar capturar los bloques modificados.

Mensaje de error: "Encountered an error while trying to fetch change blocks" (Error al intentar capturar bloques de cambios)

El método de replicación sin agente utiliza la tecnología de seguimiento de bloques modificados (CBT) de VMware para replicar los datos en Azure. CBT permite que la herramienta Changed Block Tracking realice el seguimiento y replique solo los bloques que hayan cambiado desde el último ciclo de replicación. Este error se produce si se restablece el seguimiento de bloques cambiados para una máquina virtual de replicación o si el archivo de seguimiento de bloques cambiados está dañado.

Este error se puede resolver de las dos maneras siguientes:

- Si ha optado por reparar la replicación automáticamente al seleccionar "Yes" (Sí) cuando desencadenó la replicación de la máquina virtual, la herramienta intentará repararlo. Haga clic con el botón derecho en la máquina virtual y seleccione "Repair Replication" (Reparar la replicación).
- Si no ha optado por reparar automáticamente la replicación o el paso anterior no ha funcionado, detenga la replicación de la máquina virtual, [restablezca el seguimiento de bloques cambiados](https://go.microsoft.com/fwlink/?linkid=2139203) en la máquina virtual y, a continuación, vuelva a configurar la replicación.

Un problema conocido que puede provocar un restablecimiento de CBT de la máquina virtual en VMware vSphere 5.5 se describe en el artículo [VMware KB 2048201: Se restablece el seguimiento de bloques cambiados](https://go.microsoft.com/fwlink/?linkid=2138888) tras una operación de almacenamiento de vMotion en vSphere 5.x. Si se encuentra en VMware vSphere 5.5 asegúrese de que aplica las actualizaciones descritas en este artículo de KB.

O bien, puede [restablecer el seguimiento de bloques cambiados de VMware en una máquina virtual con PowerCLI de VMware.

## <a name="an-internal-error-occurred"></a>Error interno.

En ocasiones, puede producirse un error debido a problemas en el entorno o la API de VMware. Hemos identificado el siguiente conjunto de errores relacionados con el entorno de VMware. Estos errores tienen un formato fijo.

_Mensaje de error: Error interno. [Mensaje de error]_

Por ejemplo: Mensaje de error: Error interno. [Se detectó una configuración de instantánea no válida].

En la siguiente sección se enumeran algunos de los errores de VMware más frecuentes y cómo puede mitigarlos.

### <a name="error-message-an-internal-error-occurred-server-refused-connection"></a>Mensaje de error: Error interno. [Server Refused Connection] ([El servidor ha rechazado la conexión])

El problema es un problema conocido de VMware y se produce en VDDK 6.7. Debe detener el servicio de puerta de enlace que se ejecuta en el dispositivo de Azure Migrate, [descargar una actualización desde KB de VMware](https://go.microsoft.com/fwlink/?linkid=2138889) y reiniciar el servicio de puerta de enlace.

Pasos para detener el servicio de puerta de enlace:

1. Presione Windows + R, abra services.msc. Haga clic en "Microsoft Azure Gateway Service" (Servicio de puerta de enlace de Microsoft Azure) y deténgalo.
2. Como alternativa, puede abrir el símbolo del sistema o PowerShell, y ejecutar lo siguiente: Net Stop asrgwy. Asegúrese de esperar hasta que reciba el mensaje que indique que el servicio ya no se está ejecutando.

Pasos para iniciar el servicio de puerta de enlace:

1. Presione Windows + R, abra services.msc. Haga clic con el botón derecho en "Microsoft Azure Gateway Service" (Servicio de puerta de enlace de Microsoft Azure) e inícielo.
2. Como alternativa, puede abrir el símbolo del sistema o PowerShell, y ejecutar lo siguiente: Net Start asrgwy.

### <a name="error-message-an-internal-error-occurred-an-invalid-snapshot-configuration-was-detected"></a>Mensaje de error: Error interno. [Se detectó una configuración de instantánea no válida.]

Si tiene una máquina virtual con varios discos, es posible que se produzca este error si quita un disco de la máquina virtual. Para solucionar este problema, consulte los pasos descritos en [este artículo de VMware](https://go.microsoft.com/fwlink/?linkid=2138890).

### <a name="error-message-an-internal-error-occurred-generate-snapshot-hung"></a>Mensaje de error: Error interno. [Generate Snapshot Hung] ([Generar bloqueo de instantánea])

Este problema se produce cuando la generación de instantáneas deja de responder. Cuando ocurre este problema, puede ver que la tarea para crear instantánea se detiene en 95 % o en 99 %. Consulte esta [KB de VMware](https://go.microsoft.com/fwlink/?linkid=2138969) para solucionar este problema.

### <a name="error-message-an-internal-error-occurred-failed-to-consolidate-the-disks-on-vm-_reasons_"></a>Mensaje de error: Error interno. [Failed to consolidate the disks on VM [Reasons]] ([No se pudo consolidar los discos en la máquina virtual _[razones]_ ])

Cuando se consolidan los discos al final del ciclo de replicación, se produce un error en la operación. Siga las instrucciones de la [KB de VMware](https://go.microsoft.com/fwlink/?linkid=2138970); para ello, seleccione la _razón_ adecuada para resolver el problema.

Los errores siguientes se producen cuando ocurre un error relacionado con las operaciones de instantáneas de VMware para crear, eliminar o consolidar discos. Siga las instrucciones de la sección siguiente para corregir los errores:

### <a name="error-message-an-internal-error-occurred-another-task-is-already-in-progress"></a>Mensaje de error: Error interno. [Otra tarea ya está en curso]

Este problema se produce cuando hay tareas de máquina virtual en conflicto que se ejecutan en segundo plano o cuando se agota el tiempo de espera de una tarea en vCenter Server. Siga la solución que se proporciona en el siguiente [KB de VMware](https://go.microsoft.com/fwlink/?linkid=2138891).

### <a name="error-message-an-internal-error-occurred-operation-not-allowed-in-current-state"></a>Mensaje de error: Error interno. [Operation not allowed in current state] ([Operación no permitida en el estado actual])

Este problema se produce cuando los agentes de administración de vCenter Server dejan de funcionar. Para resolver este problema, consulte la resolución en el siguiente [KB de VMware](https://go.microsoft.com/fwlink/?linkid=2138971).

### <a name="error-message-an-internal-error-occurred-snapshot-disk-size-invalid"></a>Mensaje de error: Error interno. [Snapshot Disk size invalid] ([Tamaño de disco de instantánea no válido])

Se trata de un problema conocido de VMware en el que el tamaño de disco indicado por la instantánea se convierte en cero. Siga la solución que se proporciona en el [KB de VMware](https://go.microsoft.com/fwlink/?linkid=2138972).

### <a name="error-message-an-internal-error-occurred-memory-allocation-failed-out-of-memory"></a>Mensaje de error: Error interno. [Memory allocation failed. Out of memory.] ([Error de asignación de memoria. Memoria insuficiente.])

Esto sucede cuando el búfer del host NFC no tiene memoria suficiente. Para resolver este problema, debe trasladar la máquina virtual (proceso vMotion) a un host diferente que disponga de recursos.

## <a name="next-steps"></a>Pasos siguientes

Continúe con la replicación de la máquina virtual y realice la [migración de prueba](https://go.microsoft.com/fwlink/?linkid=2139333).