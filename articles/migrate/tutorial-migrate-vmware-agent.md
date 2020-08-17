---
title: Migración de máquinas virtuales de VMware con la migración de servidores basada en agentes de Azure Migrate
description: Aprenda a ejecutar una migración de máquinas virtuales de VMware basada en agentes con Azure Migrate.
ms.topic: tutorial
ms.date: 06/09/2020
ms.custom: MVC
ms.openlocfilehash: 707a74d0e993625d32c60f7120bf69055dc9e885
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290197"
---
# <a name="migrate-vmware-vms-to-azure-agent-based"></a>Migración de máquinas virtuales de VMware a Azure (basada en agente)

En este artículo se muestra cómo migrar máquinas virtuales VMware locales a Azure mediante la migración basada en agente con la herramienta [Azure Migrate:Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool).  También puede migrar máquinas virtuales VMware mediante la migración basada en agente. [Compare](server-migrate-overview.md#compare-migration-methods) los métodos.


 En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Preparar Azure para trabajar con Azure Migrate.
> * Preparar la migración basada en agente. Configurar una cuenta de VMware para que Azure Migrate pueda detectar máquinas para la migración. Configurar una cuenta para que el agente del servicio Mobility se pueda instalar en las máquinas que desea migrar y preparar una máquina para que actúe como el dispositivo de replicación.
> * Incorporación de la herramienta Azure Migrate:Server Migration
> * Configurar el destino de replicación.
> * Replicar máquinas virtuales.
> * Ejecute una migración de prueba para asegurarse de que todo funciona de la forma esperada.
> * Ejecutar una migración completa a Azure.

> [!NOTE]
> En los tutoriales se muestra la ruta de implementación más sencilla para un escenario, de modo que pueda configurar rápidamente una prueba de concepto. En ellos se usan las opciones predeterminadas siempre que es posible y no muestran todos los valores y rutas de acceso posibles. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.


## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar este tutorial, [revise](./agent-based-migration-architecture.md) la arquitectura de la migración basada en agente de VMware.

## <a name="prepare-azure"></a>Preparación de Azure

Complete las tareas de la tabla para preparar Azure para la migración basada en agente.

**Task** | **Detalles**
--- | ---
**Crear un proyecto de Azure Migrate** | La cuenta de Azure necesita permisos de colaborador o propietario para crear un proyecto.
**Comprobar los permisos de la cuenta de Azure** | Su cuenta de Azure necesita permisos para crear una máquina virtual y escribir en un disco administrado de Azure.
**Configurar una red de Azure** | Configure una red a la que se unirán las máquinas virtuales de Azure después de la migración.

### <a name="assign-permissions-to-create-project"></a>Asignación de permisos para crear un proyecto
Si no tiene un proyecto de Azure Migrate, compruebe los permisos para crear uno.


1. En Azure Portal, abra la suscripción y seleccione **Control de acceso (IAM)** .
2. En **Comprobar acceso**, busque la cuenta correspondiente y haga clic en ella para ver los permisos.
3. Compruebe que tiene permisos de **Colaborador** o **Propietario**.

    - Si acaba de crear una cuenta de Azure gratuita, es el propietario de la suscripción.
    - Si no es el propietario, trabaje con él para asignar el rol.
    
### <a name="assign-azure-account-permissions"></a>Asignación de los permisos de la cuenta de Azure

Asigne a la cuenta el rol colaborador de máquina virtual, con el fin de que tenga permisos para:

- Crear una máquina virtual en el grupo de recursos seleccionado.
- Crear una máquina virtual en la red virtual seleccionada.
- Escribir en un disco administrado de Azure. 


### <a name="set-up-an-azure-network"></a>Configurar una red de Azure

[Configure una red de Azure](../virtual-network/manage-virtual-network.md#create-a-virtual-network). Las máquinas locales se replican en los discos administrados de Azure. Al conmutar por error a Azure para la migración, las máquinas virtuales de Azure se crean a partir de estos discos administrados y se unen a una red de Azure que configuró.

## <a name="prepare-for-migration"></a>Preparación para la migración

Compruebe los requisitos de compatibilidad y los permisos y prepárese para implementar un dispositivo de replicación. 

### <a name="prepare-an-account-to-discover-vms"></a>Preparación de una cuenta para detectar máquinas virtuales

La herramienta Azure Migrate Server Migration necesita acceso a los servidores VMware para detectar las máquinas virtuales que desea migrar. Cree la cuenta como se indica a continuación:

1. Para usar una cuenta dedicada, cree un rol en el nivel de vCenter. Asigne al rol un nombre como **Azure_Migrate**.
2. Asigne al rol los permisos que se resumen en la tabla siguiente.
3. Cree un usuario en el servidor vCenter o el host de vSphere. Asigne el rol al usuario.

#### <a name="vmware-account-permissions"></a>Permisos de cuenta de VMware

**Task** | **Rol o permisos** | **Detalles**
--- | --- | ---
**Detección de máquina virtual** | Al menos un usuario de solo lectura<br/><br/> Objeto de centro de datos  –> Propagar al objeto secundario, rol = solo lectura | El usuario se asigna en el nivel de centro de datos y tiene acceso a todos los objetos de este.<br/><br/> Para restringir el acceso, asigne el rol **Sin acceso** con **Propagate to child object** (Propagar a objeto secundario) a los objetos secundarios (hosts de vSphere, almacenes de datos, máquinas virtuales y redes).
**Replicación** |  Cree un rol (Azure_Site_Recovery) con los permisos necesarios y, después, asígnelo a un grupo o usuario de VMware.<br/><br/> Objeto de centro de datos –> Propagar a objeto secundario, rol = Azure_Site_Recovery<br/><br/> Almacén de datos -> Asignar espacio, examinar almacén de datos, operaciones de archivo de bajo nivel, quitar archivo, actualizar archivos de máquina virtual<br/><br/> Red -> Asignación de red<br/><br/> Recursos-> Asignar máquina virtual al grupo de recursos, migrar apagado de máquina virtual, migrar máquinas virtuales encendidas<br/><br/> Tareas -> Crear tarea, actualizar tarea<br/><br/> Máquina virtual -> Configuración<br/><br/> Máquina virtual -> Interactuar -> Responder a pregunta, conexión de dispositivos, configurar soporte de CD, Configurar soporte de disquete, apagar, encender, instalación de herramientas de VMware<br/><br/> Máquina virtual -> Inventario -> Crear, registrar, anular registro<br/><br/> Máquina virtual -> Aprovisionamiento -> Permitir descarga de máquina virtual, permitir carga de archivos de máquina virtual<br/><br/> Máquina virtual -> Instantáneas -> Quitar instantáneas | El usuario se asigna en el nivel de centro de datos y tiene acceso a todos los objetos de este.<br/><br/> Para restringir el acceso, asigne el rol **Sin acceso** con **Propagate to child object** (Propagar a objeto secundario) a los objetos secundarios (hosts de vSphere, almacenes de datos, máquinas virtuales y redes).

### <a name="prepare-an-account-for-mobility-service-installation"></a>Preparación de una cuenta para la instalación de Mobility Service

Mobility Service debe estar instalado en las máquinas que desee replicar.

- El dispositivo de replicación de Azure Migrate puede realizar una instalación de inserción de este servicio cuando se habilita la replicación de una máquina, o bien puede instalarlo manualmente o mediante herramientas de instalación.
- En este tutorial, se va a instalar Mobility Service con la instalación de inserción.
- Para esta instalación de inserción, debe preparar una cuenta que la herramienta Azure Migrate Server Migration pueda usar para acceder a la máquina virtual. Esta cuenta se usa solo para la instalación de inserción si no instala Mobility Service de forma manual.

Prepare la cuenta como se indica a continuación:

1. Prepare un dominio o cuenta local con permisos para instalar en la máquina virtual.
2. En máquinas virtuales Windows, si no va a usar una cuenta de dominio, deshabilite el control de acceso de usuario remoto en la máquina local: agregue la entrada DWORD **LocalAccountTokenFilterPolicy**, con un valor de 1, en el Registro, bajo **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System**.
3. En máquinas virtuales Linux, prepare una cuenta raíz en el servidor Linux de origen.


### <a name="prepare-a-machine-for-the-replication-appliance"></a>Preparación de un equipo para el dispositivo de replicación

El dispositivo se usa para replicar las máquinas en Azure. El dispositivo de replicación es una máquina virtual VMware local única, de alta disponibilidad, que hospeda estos componentes:

- **Servidor de configuración**: El servidor de configuración coordina la comunicación entre el entorno local y Azure, además de administrar la replicación de datos.
- **Servidor de proceso**: El servidor de procesos actúa como puerta de enlace de replicación. Recibe los datos de la replicación; los optimiza mediante el almacenamiento en la caché, la compresión y el cifrado, y los envía a una cuenta de almacenamiento en Azure. El servidor de procesos también instala el agente de Mobility Service en las máquinas virtuales que se desean replicar, además de realizar la detección automática de las máquinas virtuales de VMware locales.

Prepare el dispositivo de la manera siguiente:

- [Revise los requisitos del dispositivo](migrate-replication-appliance.md#appliance-requirements). Por lo general, el dispositivo de replicación se configura como una máquina virtual VMware mediante un archivo OVA descargado. La plantilla crea un dispositivo que cumple todos los requisitos.
- MySQL debe estar instalado en el dispositivo. [Revise](migrate-replication-appliance.md#mysql-installation) los métodos de instalación.
- Revise las direcciones URL de la [nube pública](migrate-replication-appliance.md#url-access) y las de [Azure Government](migrate-replication-appliance.md#azure-government-url-access) a las que necesita acceder la máquina del dispositivo.
- Revise los [puertos](migrate-replication-appliance.md#port-access) a los que necesita acceder la máquina del dispositivo.



### <a name="check-vmware-requirements"></a>Comprobación de los requisitos de VMware

Asegúrese de que los servidores y las máquinas virtuales de VMware cumplen los requisitos de migración a Azure. 

1. [Compruebe](migrate-support-matrix-vmware-migration.md#vmware-requirements-agent-based) los requisitos del servidor de VMware.
2. [Compruebe](migrate-support-matrix-vmware-migration.md#vm-requirements-agent-based) los requisitos de máquina virtual para la migración.
3. Compruebe la configuración de Azure. Las máquinas virtuales locales que replique en Azure tienen que cumplir los [requisitos de máquina virtual de Azure](migrate-support-matrix-vmware-migration.md#azure-vm-requirements).
4. Hay algunos cambios necesarios en las máquinas virtuales antes de migrarlas a Azure.
    - Es importante realizar estos cambios antes de comenzar la migración. Si migra la máquina virtual antes de realizar el cambio, es posible que la máquina virtual no arranque en Azure.
    - Revise los cambios en [Windows](prepare-for-migration.md#windows-machines) y [Linux](prepare-for-migration.md#linux-machines) que debe realizar.

> [!NOTE]
> La migración basada en agente con la herramienta Azure Migrate Server Migration se basa en las características del servicio Azure Site Recovery. Es posible que algunos requisitos lleven a la documentación de Site Recovery.



## <a name="add-the-azure-migrateserver-migration-tool"></a>Incorporación de la herramienta Azure Migrate:Server Migration

Si aún no tiene un proyecto de Azure Migrate, [configure](how-to-add-tool-first-time.md) uno ahora y agregue la herramienta Server Migration.

Si tiene un proyecto, agregue la herramienta de la siguiente manera:

1. En Azure Portal > **Todos los servicios**, busque **Azure Migrate**.
2. En **Servicios**, seleccione **Azure Migrate**.

    ![Configuración de Azure Migrate](./media/tutorial-migrate-vmware-agent/azure-migrate-search.png)

3. En **Información general**, haga clic en **Evaluar y migrar servidores**.
4. En **Detectar, evaluar y migrar servidores**, haga clic en **Evaluar y migrar servidores**.

    ![Detección y evaluación de servidores](./media/tutorial-migrate-vmware-agent/assess-migrate.png)

1. En **Detectar, evaluar y migrar servidores**, haga clic en **Agregar herramientas**.
2. En **Migrar proyecto**, seleccione la suscripción a Azure y cree un grupo de recursos, en caso de que no lo tenga.
3. En **Detalles del proyecto**, especifique el nombre del proyecto y la zona geográfica en la que quiere crearlo; después, haga clic en **Siguiente**. Revise las zonas geográficas admitidas para nubes [públicas](migrate-support-matrix.md#supported-geographies-public-cloud) y [nubes gubernamentales](migrate-support-matrix.md#supported-geographies-azure-government).

    ![Crear un proyecto de Azure Migrate](./media/tutorial-migrate-vmware-agent/migrate-project.png)


4. En **Seleccione una herramienta de evaluación**, seleccione **Omitir por ahora la adición de una herramienta de evaluación** > **Siguiente**.
5. En **Seleccione una herramienta de migración**, seleccione **Azure Migrate: Migración del servidor** > **Siguiente**.
6. En **Revisar y agregar herramientas**, revise la configuración y haga clic en **Agregar herramientas**.
7. Después de agregar la herramienta, aparece en el proyecto de Azure Migrate > **Servidores** > **Herramientas de migración**.

## <a name="set-up-the-replication-appliance"></a>Configuración del dispositivo de replicación

En este procedimiento se describe cómo configurar el dispositivo con una plantilla Open Virtualization Application (OVA) descargada. Si no puede usar este método, puede configurar el dispositivo [mediante un script](tutorial-migrate-physical-virtual-machines.md#set-up-the-replication-appliance). 

### <a name="download-the-replication-appliance-template"></a>Descarga de la plantilla del dispositivo de replicación

Descargue la plantilla de la siguiente manera:

1. En el proyecto de Azure Migrate, haga clic en **Servidores**, en **Objetivos de migración**.
2. En la página **Azure Migrate - Servidores** > **Azure Migrate: Server Migration**, haga clic en **Detectar**.

    ![Detectar máquinas virtuales](./media/tutorial-migrate-vmware-agent/migrate-discover.png)

3. En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , haga clic en **Sí, con VMware vSphere Hypervisor**.
4. En **¿Cómo quiere realizar la migración?** , seleccione **Mediante la replicación basada en agente**.
5. En **Región de destino**, seleccione la región de Azure a la que desea migrar las máquinas.
6. Seleccione **Confirme que la región de destino de la migración es nombreDeRegión**.
7. Haga clic en **Crear recursos**. Esto crea un almacén de Azure Site Recovery en segundo plano. Después de hacer clic en este botón ya no se puede cambiar la región de destino de este proyecto y todas las migraciones posteriores se dirigen a esta región.

    ![Crear almacén de Recovery Services](./media/tutorial-migrate-vmware-agent/create-resources.png)

8. En **¿Quiere instalar un nuevo dispositivo de replicación?** , seleccione **Instalar un dispositivo de replicación**.
9. Haga clic en **Descargar**. Se descarga una plantilla OVF.
    ![Descarga de OVA](./media/tutorial-migrate-vmware-agent/download-ova.png)
10. Anote el nombre del grupo de recursos y el almacén de Recovery Services. Los necesitará durante la implementación del dispositivo.


### <a name="import-the-template-in-vmware"></a>Importación de la plantilla en VMware

Después de descargar la plantilla OVF, puede importarla en VMware para crear la aplicación de replicación en una máquina virtual de VMware que ejecute Windows Server 2016.

1. Inicie sesión en el servidor VMware vCenter o el host vSphere ESXi con el cliente de VMware vSphere.
2. En el menú **File** (Archivo), seleccione **Deploy OVF Template** (Implementar plantilla OVF) para iniciar el **Asistente para implementar plantillas OVF**. 
3. En **Select source** (Seleccionar origen), especifique la ubicación de la plantilla OVF descargada.
4. En **Review details** (Revisar detalles), seleccione **Next** (Siguiente).
5. En **Select name and folder** (Seleccionar nombre y la carpeta) y **Select configuration** (Seleccionar configuración), acepte la configuración predeterminada.
6. En **Select storage** > **Select virtual disk format** (Seleccionar almacenamiento > Seleccionar formato de disco virtual), elija **Thick Provision Eager Zeroed** (Aprovisionamiento grueso diligente con ceros) para obtener un rendimiento mejor.
7. En el resto de las páginas del asistente, acepte la configuración predeterminada.
8. En **Ready to Complete** (Listo para completarse), para configurar la máquina virtual con la configuración predeterminada, seleccione **Power on after deployment** > **Finish** (Encender después de la implementación > Finalizar).

   > [!TIP]
   > Si desea agregar una NIC adicional, desactive **Power on after deployment** > **Finish** (Encender después de la implementación > Finalizar). De forma predeterminada, la plantilla contiene una sola NIC. Puede agregar NIC adicionales después de la implementación.

### <a name="start-appliance-setup"></a>Iniciar la instalación del dispositivo

1. Desde la consola de cliente de VMware vSphere, encienda la VM. La máquina virtual se inicia en una experiencia de instalación de Windows Server 2016.
2. Acepte el contrato de licencia y especifique una contraseña de administrador.
3. Una vez finalizada la instalación, inicie sesión en la máquina virtual como administrador, con la contraseña admin. La primera vez que inicia sesión, tras unos segundos se inicia la herramienta de configuración del dispositivo de replicación (la herramienta de configuración de Azure Site Recovery).
5. Escriba el nombre que usará para registrar el dispositivo con Server Migration. A continuación, haga clic en **Siguiente**.
6. La herramienta comprueba que la máquina virtual pueda conectarse a Azure. Una vez establecida la conexión, haga clic en **Iniciar sesión** para iniciar sesión en la suscripción de Azure.
7. Espere a que la herramienta termine de registrar una aplicación de Azure AD para identificar el dispositivo. El dispositivo se reinicia.
1. Inicie sesión de nuevo en la máquina. En pocos segundos, el Asistente para administración del servidor de configuración se inicia automáticamente.

### <a name="register-the-replication-appliance"></a>Registro del dispositivo de replicación

Termine de configurar y registrar el dispositivo de replicación.

1. En la configuración del dispositivo, seleccione **Configurar la conectividad**.
2. Seleccione la NIC (de forma predeterminada solo hay una NIC) que el dispositivo de replicación usa para la detección de máquinas virtuales y para realizar una instalación de inserción de Mobility Service en las máquinas de origen.
3. Seleccione la NIC que el dispositivo de replicación usa para la conectividad con Azure. Después, seleccione **Guardar**. Una vez configurada, esta opción no se puede cambiar.
4. Si el dispositivo se encuentra detrás de un servidor proxy, debe especificar la configuración de proxy.
    - Especifique el nombre del proxy como **http://ip-address** o **http://FQDN** . No se admiten servidores proxy HTTPS.
5. Cuando se le pidan los detalles de la suscripción, los grupos de recursos y el almacén, agregue los datos que anotó cuando descargó la plantilla del dispositivo.
6. En **Instalar software de terceros**, acepte el contrato de licencia. Seleccione **Descargar e instalar** para instalar MySQL Server.
7. Seleccione **Install VMware PowerCLI** (Instalar VMware PowerCLI). Asegúrese de que todas las ventanas del explorador están cerradas antes. Después, seleccione **Continuar**.
8. En **Validate appliance configuration** (Comprobar configuración del dispositivo), se comprueban los requisitos previos antes de continuar.
9. En **Configure vCenter Server/vSphere ESXi server** (Configurar vCenter Server/vSphere ESXi Server), especifique la dirección IP o FQDN del servidor vCenter o host de vSphere en los que se encuentran las máquinas virtuales donde desea realizar la replicación. Especifique el puerto en el que escucha el servidor. Escriba un nombre descriptivo que se usará para el servidor de VMware en el almacén.
10. Escriba las credenciales de la cuenta que [creó](#prepare-an-account-to-discover-vms) para la detección de VMware. Seleccione **Add** > **Continue** (Agregar > Continuar).
11. En **Configuración de credenciales de máquina virtual**, escriba las credenciales que [creó](#prepare-an-account-for-mobility-service-installation) para la instalación de inserción de Mobility Service, al habilitar la replicación para las máquinas virtuales.  
    - Para las máquinas Windows, la cuenta necesita privilegios de administrador local en las máquinas que se vayan a replicar.
    - Para Linux, proporcione los detalles de la cuenta raíz.
12. Seleccione **Finalize configuration** (Terminar configuración) para completar el registro.


Una vez registrado el dispositivo de replicación, Azure Migrate Server Assessment se conecta a los servidores de VMware con la configuración especificada y detecta las máquinas virtuales. Puede ver las máquinas virtuales detectadas en **Administrar** > **Elementos detectados**, en la pestaña **Otros**.



## <a name="replicate-vms"></a>Replicación de máquinas virtuales

Seleccione las máquinas virtuales para la migración.

> [!NOTE]
> En el portal puede seleccionar hasta 10 máquinas virtuales a la vez para la replicación. Si necesita replicar más, agrúpelas en lotes de 10.

1. En el proyecto de Azure Migrate > **Servidores**, **Azure Migrate: Migración del servidor**, haga clic en **Replicar**.

    ![Replicación de máquinas virtuales](./media/tutorial-migrate-vmware-agent/select-replicate.png)

2. En **Replicar** > **Configuración de origen** >  **¿Las máquinas están virtualizadas?** , seleccione **Sí, con VMware vSphere Hypervisor**.
3. En **Dispositivo local**, seleccione el nombre del dispositivo de Azure Migrate que configuró.
4. En **vCenter Server**, especifique el nombre de la instancia de vCenter Server que administra las máquinas virtuales o la instancia de vSphere Server en el que se hospedan.
5. En **Servidor de procesos**, seleccione el nombre del dispositivo de replicación.
6. En **Credenciales de invitado**, especifique la cuenta de administrador de máquinas virtuales que se usará para la instalación de inserción de Mobility Service. A continuación, haga clic en **Siguiente: Máquinas virtuales**.

    ![Replicación de máquinas virtuales](./media/tutorial-migrate-vmware-agent/source-settings.png)

7. En **Máquinas virtuales**, seleccione las máquinas que quiere replicar.

    - Si ha ejecutado una evaluación para las máquinas virtuales, puede aplicar las recomendaciones de tamaño y tipo de disco (Premium/estándar) de máquina virtual que sugieren los resultados de dicha evaluación. Para ello, en **¿Quiere importar la configuración de migración de evaluación de Azure Migrate?** , seleccione la opción **Sí**.
    - Si no ha ejecutado una evaluación o no desea usar la configuración de evaluación, seleccione la opción **No**.
    - Si ha decidido usar la evaluación, seleccione el grupo de máquinas virtuales y el nombre de la evaluación.

8. Compruebe todas las máquinas virtuales que desea migrar. A continuación, haga clic en **Siguiente: Configuración de destino**.
9. En **Configuración de destino**, seleccione la suscripción y la región de destino a la que va a migrar, y especifique el grupo de recursos en el que residirán las máquinas virtuales de Azure después de la migración.
10. En **Red virtual**, seleccione la red virtual o la subred de Azure a la que se unirán las máquinas virtuales de Azure después de la migración.
11. En **Ventaja híbrida de Azure**:

    - Seleccione **No** si no desea aplicar la Ventaja híbrida de Azure. A continuación, haga clic en **Siguiente**.
    - Seleccione **Sí** si tiene equipos con Windows Server que están incluidos en suscripciones activas de Software Assurance o Windows Server y desea aplicar el beneficio a las máquinas que va a migrar. A continuación, haga clic en **Siguiente**.

12. En **Proceso**, revise el nombre de la máquina virtual, su tamaño, el tipo de disco del sistema operativo y el conjunto de disponibilidad. Las máquinas virtuales deben cumplir los [requisitos de Azure](migrate-support-matrix-vmware-migration.md#azure-vm-requirements).

    - **Tamaño de VM**: si usa las recomendaciones de la evaluación, el menú desplegable de tamaño de VM contendrá el tamaño recomendado. De lo contrario, Azure Migrate elige un tamaño en función de la coincidencia más cercana en la suscripción de Azure. También puede elegir un tamaño de manera manual en **Tamaño de la máquina virtual de Azure**. 
    - **Disco del sistema operativo**: especifique el disco del sistema operativo (arranque) de la máquina virtual. Este es el disco que tiene el cargador de arranque y el instalador del sistema operativo. 
    - **Conjunto de disponibilidad**: si la máquina virtual debe estar incluida en un conjunto de disponibilidad de Azure después de la migración, especifique el conjunto. El conjunto debe estar en el grupo de recursos de destino que especifique para la migración.

13. En **Discos**, especifique si los discos de máquina virtual se deben replicar en Azure y seleccione el tipo de disco (discos SSD o HDD estándar o bien discos administrados premium) en Azure. A continuación, haga clic en **Siguiente**.
    - Puede excluir discos de la replicación.
    - Si excluye discos, no estarán presentes en la máquina virtual de Azure después de la migración. 

14. En **Revisar e iniciar la replicación**, revise la configuración y haga clic en **Replicar** para iniciar la replicación inicial de los servidores.

> [!NOTE]
> Puede actualizar la configuración de replicación en cualquier momento antes de que esta comience; para ello, vaya a **Administrar** > **Replicación de máquinas**. Una vez iniciada la replicación, su configuración no se puede cambiar.


## <a name="track-and-monitor"></a>Seguimiento y supervisión

1. Realice un seguimiento del estado del trabajo en las notificaciones del portal. 

    ![Seguimiento del trabajo](./media/tutorial-migrate-vmware-agent/jobs.png)
    
2. Para supervisar el estado de la replicación, haga clic en **Replicando servidores** en **Azure Migrate: Server Migration**.

    ![Supervisión de la replicación](./media/tutorial-migrate-vmware-agent/replicate-servers.png)

La replicación se produce de la manera siguiente:
- Cuando el trabajo de inicio de replicación finaliza correctamente, las máquinas comienzan su replicación inicial en Azure.
- Cuando finaliza esta replicación inicial, comienza la replicación diferencial. Los cambios incrementales de los discos locales se replican periódicamente en los discos de réplica de Azure.


## <a name="run-a-test-migration"></a>Ejecutar una migración de prueba


Cuando comienza la replicación diferencial, puede ejecutar una migración de prueba para las máquinas virtuales antes de ejecutar una migración completa a Azure. Le recomendamos encarecidamente que lo haga al menos una vez en cada máquina, antes de migrarla.

- La ejecución de una migración de prueba comprueba que la migración funcionará según lo previsto, sin afectar a las máquinas locales, que seguirán estando operativas y continuarán realizando la replicación. 
- Para simular la migración, la migración de prueba crea una máquina virtual de Azure usando datos replicados (normalmente, con una migración a una red virtual que no es de producción en la suscripción a Azure).
- Puede usar la máquina virtual de Azure de prueba replicada para validar la migración, realizar pruebas de aplicaciones y resolver los problemas antes de la migración completa.

Realice una migración de prueba como se indica a continuación:


1. En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Migration**, haga clic en **Probar servidores migrados**.

     ![Probar servidores migrados](./media/tutorial-migrate-vmware-agent/test-migrated-servers.png)

2. Haga clic con el botón derecho en la máquina virtual que va a probar y haga clic en **Migración de prueba**.

    ![Migración de prueba](./media/tutorial-migrate-vmware-agent/test-migrate.png)

3. En **Migración de prueba**, seleccione la red virtual de Azure en la que se ubicará la máquina virtual de Azure después de la migración. Se recomienda usar una red virtual que no sea de producción.
4. Comienza el trabajo de **Migración de prueba**. Supervise el trabajo en las notificaciones del portal.
5. Una vez finalizada la migración, la máquina virtual de Azure migrada se puede ver en **Máquinas virtuales** en Azure Portal. El nombre de la máquina tiene el sufijo **-Test**.
6. Una vez finalizada la prueba, haga clic con el botón derecho en la máquina virtual de Azure, en **Replicación de máquinas**, y haga clic en **Limpiar la migración de prueba**.

    ![Limpiar la migración](./media/tutorial-migrate-vmware-agent/clean-up.png)


## <a name="migrate-vms"></a>Migración de máquinas virtuales

Después de comprobar que la migración de prueba funciona según lo previsto, puede migrar las máquinas locales.

1. En el proyecto de Azure Migrate > **Servidores** > **Azure Migrate: Server Migration**, haga clic en **Replicando servidores**.

    ![Replicando servidores](./media/tutorial-migrate-vmware-agent/replicate-servers.png)

2. En **Replicación de máquinas**, haga clic con el botón derecho en la máquina virtual > **Migrar**.
3. En **Migrar** >  **¿Quiere apagar las máquinas virtuales y realizar una migración planificada sin perder datos?** , seleccione **Sí** > **Aceptar**.
    - De forma predeterminada, Azure Migrate apaga la máquina virtual local para garantizar una pérdida de datos mínima. 
    - Si no desea apagar la máquina virtual, seleccione **No**
4. Se inicia un trabajo de migración de la máquina virtual. Realice un seguimiento del trabajo en las notificaciones de Azure.
5. Una vez finalizado el trabajo, la máquina virtual puede ver y administrar desde la página **Máquinas virtuales**.

## <a name="complete-the-migration"></a>Completar la migración

1. Una vez finalizada la migración, haga clic con el botón derecho en la máquina virtual > **Detener migración**. Esto hace lo siguiente:
    - Detiene la replicación en la máquina local.
    - Quita la máquina del recuento de **Servidores en replicación** en Azure Migrate: Server Migration.
    - Limpia la información de estado de replicación de la máquina virtual.
2. Instale el agente de [Windows](../virtual-machines/extensions/agent-windows.md) o [Linux](../virtual-machines/extensions/agent-linux.md) de la máquina virtual de Azure en las máquinas migradas.
3. Realice los ajustes de la aplicación posteriores a la migración, como actualizar las cadenas de conexión de la base de datos y las configuraciones del servidor web.
4. Realice las pruebas finales de la aplicación y la aceptación de la migración en la aplicación migrada que ahora se ejecuta en Azure.
5. Pase el tráfico a la instancia de máquina virtual de Azure migrada.
6. Quite las máquinas virtuales locales del inventario de máquinas virtuales local.
7. Quite las máquinas virtuales locales de las copias de seguridad locales.
8. Actualice la documentación interna para mostrar la nueva ubicación y la dirección IP las máquinas virtuales de Azure. 

## <a name="post-migration-best-practices"></a>Procedimientos recomendados después de la migración

- Local
    - Mueva el tráfico de la aplicación hasta la aplicación que se ejecuta en la instancia de máquina virtual de Azure migrada.
    - Quite las máquinas virtuales locales del inventario de máquinas virtuales local.
    - Quite las máquinas virtuales locales de las copias de seguridad locales.
    - Actualice la documentación interna para mostrar la nueva ubicación y la dirección IP las máquinas virtuales de Azure.
- Ajustar la configuración de las VM de Azure después de la migración:
    - El [agente de máquina virtual de Azure](../virtual-machines/extensions/agent-windows.md) administra la interacción de una máquina virtual con el controlador de tejido de Azure. Se requiere para algunos servicios de Azure, como Azure Backup, Site Recovery y Azure Security. Al migrar VM de VMware con la migración basada en agentes, el instalador de Mobility Service instala el agente de VM de Azure en máquinas Windows. En VM Linux, se recomienda instalar al agente después de la migración.
    - Desinstale manualmente Mobility Service de la VM de Azure después de la migración.
    - Desinstale manualmente las herramientas de VMware después de la migración.
- En Azure:
    - Realice los ajustes de la aplicación posteriores a la migración, como actualizar las cadenas de conexión de la base de datos y las configuraciones del servidor web.
    - Realice las pruebas finales de la aplicación y la aceptación de la migración en la aplicación migrada que ahora se ejecuta en Azure.
- Continuidad empresarial/recuperación ante desastres
    - Proteja los datos mediante la copia de seguridad de máquinas virtuales de Azure mediante el servicio Azure Backup. [Más información](../backup/quick-backup-vm-portal.md).
    - Mantenga las cargas de trabajo en ejecución y disponibles continuamente mediante la replicación de máquinas virtuales de Azure en una región secundaria con Site Recovery. [Más información](../site-recovery/azure-to-azure-tutorial-enable-replication.md).
- Para aumentar la seguridad:
    - Bloquee y limite el acceso de tráfico de entrada con la [administración Just-In-Time de Azure Security Center](../security-center/security-center-just-in-time.md).
    - Restrinja el tráfico de red a los puntos de conexión de administración con [grupos de seguridad de red](../virtual-network/security-overview.md).
    - Implemente [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md) para ayudar a proteger discos y datos frente al robo y acceso no autorizado.
    - Obtenga más información sobre la [protección de recursos IaaS](https://azure.microsoft.com/services/virtual-machines/secure-well-managed-iaas/) y visite [Azure Security Center](https://azure.microsoft.com/services/security-center/).
- Para supervisión y administración:
    - Considere la posibilidad de implementar [Azure Cost Management](../cost-management-billing/cloudyn/overview.md) para supervisar el gasto y el uso de recursos.




 ## <a name="next-steps"></a>Pasos siguientes

Investigue el [proceso de la migración en la nube](/azure/architecture/cloud-adoption/getting-started/migrate) en el marco de Cloud Adoption Framework para Azure.
