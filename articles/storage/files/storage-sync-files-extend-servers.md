---
title: 'Tutorial: Ampliación de servidores de archivos Windows con Azure File Sync | Microsoft Docs'
description: Aprenda a ampliar servidores de archivos Windows con Azure File Sync, desde el principio del proceso al final.
author: roygara
ms.service: storage
ms.topic: tutorial
ms.date: 10/23/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 80fcebec76788ca9ec754b35c57f9965f38c2c0e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87037106"
---
# <a name="tutorial-extend-windows-file-servers-with-azure-file-sync"></a>Tutorial: Extensión de servidores de archivos de Windows con Azure File Sync

En este artículo se demuestran los pasos básicos para ampliar la capacidad de almacenamiento de un servidor con Windows Server mediante Azure File Sync. Aunque el tutorial presenta un servidor con Windows Server como una máquina virtual de Azure, lo habitual es realizar este proceso para servidores locales. En el artículo [Implementación de Azure File Sync](storage-sync-files-deployment-guide.md) puede encontrar instrucciones para implementar Azure File Sync en su propio entorno.

> [!div class="checklist"]
> * Implementación del servicio de sincronización de almacenamiento
> * Preparación de Windows Server para su uso con Azure File Sync
> * Instalación del agente de Azure File Sync
> * Registro de un servidor de Windows Server en el servicio de sincronización de almacenamiento
> * Creación de un grupo de sincronización y un punto de conexión en la nube
> * Creación de un punto de conexión de servidor

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="prepare-your-environment"></a>Preparación del entorno

Para este tutorial, deberá realizar las siguientes operaciones antes de implementar Azure File Sync:

- Crear una cuenta de Azure Storage y un recurso compartido
- Configurar una máquina virtual con Windows Server 2016 Datacenter
- Preparar la máquina virtual con Windows Server para Azure File Sync

### <a name="create-a-folder-and-txt-file"></a>Creación de una carpeta y un archivo .txt

En el equipo local, cree una carpeta nueva denominada _FilesToSync_ y agregue un archivo de texto denominado _mytestdoc.txt_. Dicho archivo se cargará en el recurso compartido de archivos más adelante.

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

### <a name="create-a-file-share"></a>Creación de un recurso compartido de archivos

Tras implementar una cuenta de Azure Storage, cree un recurso compartido de archivos.

1. En Azure Portal, seleccione **Ir al recurso**.
1. Seleccione **Archivos** en el panel de la cuenta de almacenamiento.

    ![Seleccionar Archivos](./media/storage-sync-files-extend-servers/click-files.png)

1. Seleccione **+ Recurso compartido de archivos**.

    ![Haga clic en el botón para agregar un recurso compartido de archivos](./media/storage-sync-files-extend-servers/create-file-share-portal2.png)

1. Asigne el nombre _afsfileshare_ al nuevo recurso compartido de archivos. Escriba "1" en **Cuota** y seleccione **Crear**. El valor máximo de la cuota es 5 TiB, pero para este tutorial solo se necesita 1 GB.

    ![Especificar el nombre y la cuota del nuevo recurso compartido de archivos](./media/storage-sync-files-extend-servers/create-file-share-portal3.png)

1. Seleccione el recurso compartido de archivos nuevo. En la ubicación del recurso compartido de archivos, seleccione **Cargar**.

    ![Cargar un archivo](./media/storage-sync-files-extend-servers/create-file-share-portal5.png)

1. Vaya a la carpeta _FilesToSync_ en la que creó el archivo .txt, seleccione _mytestdoc.txt_ y, después, seleccione **Cargar**.

    ![Examinar recurso compartido de archivos](./media/storage-sync-files-extend-servers/create-file-share-portal6.png)

Ya ha creado una cuenta de almacenamiento y un recurso compartido de archivos con un archivo en él. A continuación, implementará una máquina virtual de Azure con Windows Server 2016 Datacenter que representará el servidor local en este tutorial.

### <a name="deploy-a-vm-and-attach-a-data-disk"></a>Implementación de una máquina virtual y conexión de un disco de datos

1. Vaya a Azure Portal y expanda el menú de la izquierda. Elija **Crear un recurso** en la esquina superior izquierda.
1. En el cuadro de búsqueda que está encima de la lista de recursos de **Azure Marketplace**, busque **Windows Server 2016 Datacenter** y selecciónelo en los resultados. Seleccione **Create**.
1. Vaya a la pestaña **Datos básicos**. En **Detalles del proyecto**, seleccione el grupo de recursos que creó para este tutorial.

   ![Especificar información básica acerca de la máquina virtual en la hoja del portal](./media/storage-sync-files-extend-servers/vm-resource-group-and-subscription.png)

1. En **Detalles de instancia**, especifique el nombre de la máquina virtual. Por ejemplo, use _MiVM_.
1. No cambie la configuración predeterminada en **Región**, **Opciones de disponibilidad**, **Imagen** y **Tamaño**.
1. En **Cuenta de administrador**, especifique el **nombre de usuario** y la **contraseña** de la máquina virtual.
1. En **Reglas de puerto de entrada**, elija **Permitir los puertos seleccionados** y luego seleccione **RDP (3389)** y **HTTP** en el menú desplegable.

1. Antes de crear la máquina virtual, es preciso crear un disco de datos.

   1. Seleccione **Siguiente: Discos**.

      ![Adición de discos de datos](./media/storage-sync-files-extend-servers/vm-add-data-disk.png)

   1. En la pestaña **Discos**, en **Opciones de disco**, deje los valores predeterminados.
   1. En **DISCOS DE DATOS**, seleccione **Crear y adjuntar un nuevo disco**.

   1. Use la configuración predeterminada, salvo en **Tamaño (GiB)** , que se puede cambiar a **1 GB** para este tutorial.

      ![Detalles del disco de datos](./media/storage-sync-files-extend-servers/vm-create-new-disk-details.png)

   1. Seleccione **Aceptar**.
1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.

   Puede seleccionar el icono **Notificaciones** para ver el **progreso de la implementación**. La creación de una máquina virtual tarda varios minutos en completarse.

1. Después de que se complete la implementación de la máquina virtual, seleccione **Ir al recurso**.

   ![Ir al recurso](./media/storage-sync-files-extend-servers/vm-gotoresource.png)

Ya ha creado una nueva máquina virtual y ha conectado un disco de datos. Después, conéctese a la máquina virtual.

### <a name="connect-to-your-vm"></a>Conexión a la máquina virtual

1. En Azure Portal, seleccione **Conectar** en la página de propiedades de la máquina virtual.

   ![Conexión a una máquina virtual de Azure desde el portal](./media/storage-sync-files-extend-servers/connect-vm.png)

1. En la página **Conectarse a una máquina virtual**, mantenga las opciones predeterminadas para conectarse por la **dirección IP** a través del puerto 3389. Seleccione **Descargar archivo RDP**.

   ![Descarga del archivo RDP](./media/storage-sync-files-extend-servers/download-rdp.png)

1. Abra el archivo RDP que descargó y haga clic en **Conectar** cuando se le solicite.
1. En la ventana **Seguridad de Windows**, seleccione **Más opciones** y, después, **Usar otra cuenta**. Escriba el nombre de usuario con el siguiente formato, *localhost\nombre de usuario*, escriba la contraseña que creó para la máquina virtual y seleccione **Aceptar**.

   ![Más opciones](./media/storage-sync-files-extend-servers/local-host2.png)

1. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Seleccione **Sí** o **Continuar** para crear la conexión.

### <a name="prepare-the-windows-server"></a>Preparación del servidor de Windows Server

En el caso del servidor de Windows Server 2016 Datacenter, deshabilite Configuración de seguridad mejorada de Internet Explorer. Este paso solo es necesario para el registro inicial del servidor. Se puede volver a habilitar una vez registrado el servidor.

En la máquina virtual con Windows Server 2016 Datacenter, el Administrador del servidor se abre automáticamente.  Si el Administrador del servidor no se abre de forma predeterminada, búsquelo en el menú Inicio.

1. En el **Administrador del servidor**, seleccione **Servidor local**.

   !["Servidor local" a la izquierda de la interfaz de usuario Administrador del servidor](media/storage-sync-files-extend-servers/prepare-server-disable-ieesc-1.png)

1. En el panel **Propiedades**, seleccione el vínculo de **Configuración de seguridad mejorada de IE**.  

    ![El panel "Configuración de seguridad mejorada de IE" en la interfaz de usuario del Administrador del servidor](media/storage-sync-files-extend-servers/prepare-server-disable-ieesc-2.png)

1. En el cuadro de diálogo **Configuración de seguridad mejorada de Internet Explorer**, seleccione **Desactivado** en **Administradores** y **Usuarios**.

    ![Ventana emergente Configuración de seguridad mejorada de Internet Explorer con "Desactivado" seleccionado](media/storage-sync-files-extend-servers/prepare-server-disable-ieesc-3.png)

Ya puede agregar el disco de datos a la máquina virtual.

### <a name="add-the-data-disk"></a>Incorporación del disco de datos

1. En la máquina virtual de **Windows Server 2016 Datacenter**, haga clic en **Files and storage services** > **Volumes** > **Disks** (Archivos y servicios de almacenamiento > Volúmenes > Discos).

    ![Disco de datos](media/storage-sync-files-extend-servers/your-disk.png)

1. Haga clic con el botón derecho en el disco de 1 GB llamado **Msft Virtual Disk** y seleccione **New volume** (Volumen nuevo).
1. Finalice el asistente. Use la configuración predeterminada y tome nota de la letra de unidad asignada.
1. Seleccione **Crear**.
1. Seleccione **Cerrar**.

   Ya está el disco en línea y ha creado un volumen. Abra el Explorador de archivos en la máquina virtual de Windows Server para confirmar la presencia del disco de datos recién agregado.

1. En el Explorador de archivos de la máquina virtual, expanda **Este PC** y abra la nueva unidad. En este ejemplo es la unidad F:.
1. Haga clic con el botón derecho y seleccione **Nueva** > **carpeta**. Asigne a la carpeta el nombre _FilesToSync_.
1. Abra la carpeta **FilesToSync**.
1. Haga clic con el botón derecho y seleccione **Nuevo** > **Documento de texto**. Asigne el archivo de texto el nombre _MyTestFile_.

    ![Agregar un archivo de texto nuevo](media/storage-sync-files-extend-servers/new-file.png)

1. Cierre el **Explorador de archivos** y el **Administrador del servidor**.

### <a name="download-the-azure-powershell-module"></a>Descarga del módulo Azure PowerShell

A continuación, en la máquina virtual con Windows Server 2016 Datacenter, instale el módulo de Azure PowerShell en el servidor.

1. En la máquina virtual, abra una ventana de PowerShell con privilegios elevados.
1. Ejecute el siguiente comando:

   ```powershell
   Install-Module -Name Az
   ```

   > [!NOTE]
   > Si tiene una versión de NuGet anterior a la 2.8.5.201, se le pedirá que descargue e instale la versión más reciente.

   De forma predeterminada, la Galería de PowerShell no está configurada como un repositorio de confianza para PowerShellGet. La primera vez que use PSGallery verá el siguiente mensaje:

   ```output
   Untrusted repository

   You are installing the modules from an untrusted repository. If you trust this repository, change its InstallationPolicy value by running the Set-PSRepository cmdlet.

   Are you sure you want to install the modules from 'PSGallery'?
   [Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"):
   ```

1. Responda **Sí** o **Sí a todo** para continuar con la instalación.

El módulo `Az` es un módulo acumulativo para los cmdlets de Azure PowerShell. Al instalarlo, se descargan todos los módulos disponibles de Azure Resource Manager y hace que sus cmdlets estén disponibles para su uso.

Ya ha configurado el entorno para el tutorial y está listo para implementar el servicio de sincronización de almacenamiento.

## <a name="deploy-the-service"></a>Implementación del servicio

Para implementar Azure File Sync, en primer lugar coloque un recurso del **servicio de sincronización de almacenamiento** en un grupo de recursos de la suscripción seleccionada. El servicio de sincronización de almacenamiento hereda los permisos de acceso de su suscripción y del grupo de recursos.

1. En Azure Portal, seleccione **Crear un recurso** y busque **Azure File Sync**.
1. En los resultados de la búsqueda, haga clic en **Azure File Sync**.
1. Seleccione **Crear** para abrir la pestaña **Implementar la sincronización del almacenamiento**.

   ![Implementar la sincronización del almacenamiento](media/storage-sync-files-extend-servers/afs-info.png)

   En el panel que se abre, escriba la siguiente información:

   | Value | Descripción |
   | ----- | ----- |
   | **Nombre** | Un nombre único (por suscripción) para el servicio de sincronización de almacenamiento.<br><br>Use _afssyncservice02_ para este tutorial. |
   | **Suscripción** | La suscripción de Azure que utiliza para este tutorial. |
   | **Grupos de recursos** | el grupo de recursos que contiene el servicio de sincronización de almacenamiento.<br><br>Use _afsresgroup101918_ para este tutorial. |
   | **Ubicación** | Este de EE. UU. |

1. Cuando haya terminado, seleccione **Crear** para implementar el **servicio de sincronización de almacenamiento**.
1. Seleccione la pestaña **Notificaciones** > **Ir al recurso**.

## <a name="install-the-agent"></a>Instalación del agente

El agente de Azure File Sync es un paquete descargable que permite la sincronización de Windows Server con un recurso compartido de archivos de Azure.

1. En la máquina virtual con **Windows Server 2016 Datacenter**, abra **Internet Explorer**.
1. Vaya al [Centro de descarga de Microsoft](https://go.microsoft.com/fwlink/?linkid=858257). Desplácese hacia abajo hasta la sección **Azure File Sync Agent** (agente de Azure File Sync) y haga clic en **Download** (Descargar).

   ![Descarga del agente de File Sync](media/storage-sync-files-extend-servers/sync-agent-download.png)

1. Seleccione la casilla de **StorageSyncAgent_V3_WS2016. EXE** y, después, **Siguiente**.

   ![Seleccionar agente](media/storage-sync-files-extend-servers/select-agent.png)

1. Seleccione **Permitir una vez** > **Ejecutar** > **Abrir**.
1. Si no lo ha hecho, cierre la ventana de PowerShell.
1. Acepte los valores predeterminados en el **Asistente para instalación de agente de sincronización de almacenamiento**.
1. Seleccione **Instalar**.
1. Seleccione **Finalizar**.

Ha implementado el Servicio de sincronización de Azure e instalado el agente en la máquina virtual con Windows Server 2016 Datacenter. Ahora tiene que registrar la máquina virtual en el servicio de sincronización de almacenamiento.

## <a name="register-windows-server"></a>Registro de Windows Server

Si el servidor de Windows Server se registra en un servicio de sincronización de almacenamiento, se establece una relación de confianza entre el servidor (o el clúster) y el servicio. Un servidor no se puede registrar en más de un servicio de sincronización de almacenamiento. No obstante, se puede sincronizar con otros servidores y recursos compartidos de archivos de Azure asociados con dicho servicio.

Tras la instalación del agente de Azure File Sync, la interfaz de usuario de Registro del servidor se debería abrir automáticamente. Si no es así, puede abrirla manualmente desde su ubicación de archivo: `C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe.`

1. Cuando la interfaz de usuario de Registro del servidor se abra en la máquina virtual, seleccione **Aceptar**.
1. Para comenzar, seleccione **Iniciar sesión**.
1. Inicie sesión con las credenciales de su cuenta de Azure y seleccione **Iniciar sesión**.
1. Proporcione la siguiente información:

   ![Captura de pantalla de la interfaz de usuario Registro del servidor](media/storage-sync-files-extend-servers/signin.png)

   | Value | Descripción |
   | ----- | ----- |
   | **Suscripción de Azure** | La suscripción que contiene el servicio de sincronización de almacenamiento de este tutorial. |
   | **Grupo de recursos** | el grupo de recursos que contiene el servicio de sincronización de almacenamiento. Use _afsresgroup101918_ para este tutorial. |
   | **Servicio de sincronización de almacenamiento** | El nombre del servicio de sincronización de almacenamiento. Use _afssyncservice02_ para este tutorial. |

1. Seleccione **Registrar** para completar el registro del servidor.
1. Como parte del proceso de registro, se le solicita un que vuelva a iniciar sesión. Inicie sesión y seleccione **Siguiente**.
1. Seleccione **Aceptar**.

## <a name="create-a-sync-group"></a>Creación de un grupo de sincronización

Un grupo de sincronización define la topología de sincronización de un conjunto de archivos. Un grupo de sincronización debe contener un punto de conexión en la nube, que representa un recurso compartido de archivos de Azure. También debe contener uno o varios puntos de conexión de servidor. Un punto de conexión de servidor representa una ruta de acceso en un servidor registrado. Para crear un grupo de sincronización:

1. En [Azure Portal](https://portal.azure.com/), seleccione **+ Grupo de sincronización** en el servicio de sincronización de almacenamiento. Use *afssyncservice02* para este tutorial.

   ![Creación de un grupo de sincronización en Azure Portal](media/storage-sync-files-extend-servers/add-sync-group.png)

1. Escriba la siguiente información para crear un grupo de sincronización con un punto de conexión en la nube:

   | Value | Descripción |
   | ----- | ----- |
   | **Nombre del grupo de sincronización** | Este nombre debe ser único dentro del servicio de sincronización de almacenamiento, pero puede ser cualquier nombre que considere lógico. Use *afssyncgroup* para este tutorial.|
   | **Suscripción** | La suscripción en la que se ha implementado el servicio de sincronización de almacenamiento de este tutorial. |
   | **Cuenta de almacenamiento** | Elija **Seleccionar cuenta de almacenamiento**. En el panel que aparece, seleccione la cuenta de almacenamiento que tiene el recurso compartido de archivos de Azure que ha creado. Use *afsstoracct101918* para este tutorial. |
   | **Recurso compartido de archivos de Azure** | El nombre del recurso compartido de archivos de Azure que ha creado. Use *afsfileshare* para este tutorial. |

1. Seleccione **Crear**.

Si selecciona el grupo de sincronización, puede ver que ahora tiene uno **punto de conexión en la nube**.

## <a name="add-a-server-endpoint"></a>Adición de un punto de conexión de servidor

Un punto de conexión del servidor representa una ubicación concreta en un servidor registrado. Por ejemplo, una carpeta en un volumen de servidor. Para agregar un punto de conexión del servidor:

1. Seleccione al grupo de sincronización recién creado y, después, seleccione **Agregar punto de conexión del servidor**.

   ![Adición de un nuevo punto de conexión de servidor al panel Grupo de sincronización](media/storage-sync-files-extend-servers/add-server-endpoint.png)

1. En el panel **Agregar punto de conexión del servidor**, escriba la siguiente información para crear un punto de conexión del servidor:

   | Value | Descripción |
   | ----- | ----- |
   | **Servidor registrado** | El nombre del servidor que ha creado. Use *afsvm101918* para este tutorial. |
   | **Path** | La ruta de acceso del servidor de Windows Server a la unidad que ha creado. Use *f:\filestosync* en este tutorial. |
   | **Nube por niveles** | Déjelo deshabilitado para este tutorial. |
   | **Espacio disponible del volumen** | Déjelo en blanco para este tutorial. |

1. Seleccione **Crear**.

Los archivos ya están sincronizados entre el recurso compartido de archivos de Azure y Windows Server.

![Azure Storage se ha sincronizado correctamente](media/storage-sync-files-extend-servers/files-synced-in-azurestorage.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [storage-files-clean-up-portal](../../../includes/storage-files-clean-up-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido los pasos básicos necesarios para ampliar la capacidad de almacenamiento de un servidor con Windows Server mediante Azure File Sync. Para obtener una visión más completa del planeamiento de una implementación de Azure File Sync, consulte:

> [!div class="nextstepaction"]
> [Planeamiento de una implementación de Azure Files Sync](./storage-sync-files-planning.md)
