---
title: 'Solución de VMware en Azure: migración con Azure Data Box'
description: Uso de Azure Data Box para migrar datos en bloque a la solución de VMware en Azure.
author: sharaths-cs
ms.author: dikamath
ms.date: 09/27/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: ab772bd9cb415045ef70cb4cf9a518791befb192
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86507677"
---
# <a name="migrating-data-to-azure-vmware-solution-by-using-azure-data-box"></a>Migración de datos a la solución de VMware en Azure con Azure Data Box

La solución en la nube Microsoft Azure Data Box le permite enviar terabytes de datos a Azure de forma rápida, económica y confiable. La transferencia de datos segura se acelera mediante el envío de un dispositivo de almacenamiento de Data Box de propiedad. Cada dispositivo de almacenamiento tiene una capacidad de almacenamiento máximo utilizable de 80 TB y se transporta al centro de datos mediante un operador regional. El dispositivo tiene una carcasa resistente para proteger los datos durante el envío.

Al utilizar un dispositivo Data Box, puede realizar una migración de forma masiva de los datos de VMware a la nube privada. Los datos del entorno de VMware vSphere local se copian en Data Box mediante el protocolo Network File System (NFS). La migración de datos en bloque implica guardar una copia en un momento dado de las máquinas virtuales, la configuración y los datos asociados en el dispositivo Data Box y enviarlos manualmente a Azure.

En este artículo, aprenderá lo siguiente:

* Configuración de un dispositivo Data Box.
* Copia de datos desde el entorno VMware local al dispositivo Data Box mediante NFS.
* Preparación para la devolución del dispositivo Data Box.
* Preparación de los datos del blob para copiarlos en la solución de VMware en Azure.
* Copia de los datos de Azure a la nube privada.

## <a name="scenarios"></a>Escenarios

Utilice Data Box en los escenarios siguientes para la migración de datos en bloque:

* Para migrar una gran cantidad de datos de una solución de VMware en Azure local. En este método se establece una base de referencia y se sincronizan las diferencias a través de la red.
* Para migrar un gran número de máquinas virtuales que están desconectadas (máquinas virtuales en frío).
* Para migrar los datos de las máquinas virtuales y configurar entornos de desarrollo y pruebas.
* Para migrar un gran número de plantillas de máquina virtual, archivos ISO o discos de máquina virtual.

## <a name="before-you-begin"></a>Antes de empezar

* Compruebe los requisitos previos y [solicite el dispositivo Data Box](../databox/data-box-deploy-ordered.md) mediante Azure Portal. Durante el proceso de pedido, debe seleccionar una cuenta de almacenamiento, que permite realizar el almacenamiento de blobs. Después de recibir el dispositivo Data Box, conéctelo a la red local y [configúrelo](../databox/data-box-deploy-set-up.md) con una dirección IP accesible desde la red de administración de vSphere.

* Cree una red virtual y una cuenta de almacenamiento en la misma región donde se aprovisiona la solución de VMware en Azure.

* Cree una [conexión de red virtual de Azure](cloudsimple-azure-network-connection.md) desde la nube privada a la red virtual donde se ha creado la cuenta de almacenamiento mediante los pasos mencionados en el artículo [Conexión de la red virtual de Azure a CloudSimple mediante ExpressRoute](virtual-network-connection.md).

## <a name="set-up-data-box-for-nfs"></a>Configuración del dispositivo de Data Box para NFS

Conéctese a la interfaz de usuario web local del dispositivo Data Box siguiendo los pasos descritos en la sección "Conexión al dispositivo" del [Tutorial: Cableado y conexión de un dispositivo Azure Data Box](../databox/data-box-deploy-set-up.md).  Configure el dispositivo Data Box para permitir el acceso a los clientes NFS:

1. En la interfaz de usuario web local, vaya a la página **Connect and copy** (Conectar y copiar). En **NFS settings** (Configuración de NFS), haga clic en **NFS client access** (Acceso de cliente NFS). 

    ![Configuración del acceso de cliente NFS 1](media/nfs-client-access.png)

2. Escriba la dirección IP de los hosts de VMware ESXi y haga clic en **Add** (Agregar). Puede configurar el acceso para todos los hosts del clúster de vSphere mediante la repetición de este paso. Seleccione **Aceptar**.

    ![Configuración del acceso de cliente NFS 2](media/nfs-client-access2.png)
> [!IMPORTANT]
> **Cree siempre una carpeta para los archivos que se va a copiar en el recurso compartido y, después, copie los archivos en ella**. La carpeta que se creó en los recursos compartidos de blob en bloques y blob en páginas representa un contenedor en el que los datos se cargan como blobs. No se pueden copiar los archivos directamente en la carpeta *root* de la cuenta de almacenamiento.

En los recursos compartidos de blob en bloques y en páginas, las entidades de primer nivel son contenedores y las entidades de segundo nivel son blobs. En los recursos compartidos de Azure Files, las entidades de primer nivel son los recursos compartidos y las entidades de segundo nivel son los archivos.

En la tabla siguiente se muestra la ruta de acceso UNC a los recursos compartidos en la dirección URL de la ruta de acceso de Data Box y Azure Storage donde se cargan los datos. La dirección URL final de la ruta de acceso de Azure Storage se puede derivar a partir de la ruta de acceso UNC al recurso compartido.
 
| Blobs y archivos | Ruta de acceso y URL |
|---------------- | ------------ |
| Blobs en bloques de Azure | <li>Ruta de acceso UNC a recursos compartidos: `//<DeviceIPAddress>/<StorageAccountName_BlockBlob>/<ContainerName>/files/a.txt`</li><li>Dirección URL de Azure Storage: `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li> |  
| Blobs en páginas de Azure  | <li>Ruta de acceso UNC a recursos compartidos: `//<DeviceIPAddres>/<StorageAccountName_PageBlob>/<ContainerName>/files/a.txt`</li><li>Dirección URL de Azure Storage: `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li>   |  
| Azure Files       |<li>Ruta de acceso UNC a recursos compartidos: `//<DeviceIPAddres>/<StorageAccountName_AzFile>/<ShareName>/files/a.txt`</li><li>Dirección URL de Azure Storage: `https://<StorageAccountName>.file.core.windows.net/<ShareName>/files/a.txt`</li>        |

> [!NOTE]
> Use los blobs en bloques de Azure para copiar datos de VMware.

## <a name="mount-the-nfs-share-as-a-datastore-on-your-on-premises-vcenter-cluster-and-copy-the-data"></a>Montaje de un recurso compartido NFS como almacén de datos en un clúster de vCenter local y copia de datos

El recurso compartido NFS del dispositivo Data Box debe montarse como un almacén de datos en el clúster de vCenter local o en el host de VMware ESXi para copiar los datos del almacén de datos de NFS:

1. Inicie sesión en el servidor de vCenter local.

2. Haga clic con el botón secundario en **Datacenter** (Centro de recursos), seleccione **Storage** (Almacenamiento), después **New Datastore** (Nuevo almacén de datos) y, a continuación, seleccione **Next** (Siguiente).

   ![Incorporación de un nuevo almacén de datos](media/databox-migration-add-datastore.png)

3. En el paso 1 del asistente para agregar un almacén de datos, seleccione el tipo **NFS** en **Type** (Tipo).

   ![Incorporación de un nuevo almacén de datos: tipo](media/databox-migration-add-datastore-type.png)

4. En el paso 2 del asistente, seleccione **NFS 3** como la versión de NFS y, a continuación, seleccione **Next** (Siguiente).

   ![Incorporación de un nuevo almacén de datos: versión de NFS](media/databox-migration-add-datastore-nfs-version.png)

5. En el paso 3, especifique el nombre del almacén de datos, la ruta de acceso y el servidor. Puede usar la dirección IP del dispositivo Data Box como servidor. La ruta de acceso de la carpeta tendrá el formato `/<StorageAccountName_BlockBlob>/<ContainerName>/`.

   ![Incorporación de un nuevo almacén de datos: configuración de NFS](media/databox-migration-add-datastore-nfs-configuration.png)

6. En el paso 4, seleccione los hosts ESXi en los que desea montar el almacén de datos y, después, seleccione **Next** (Siguiente).  En un clúster, seleccione todos los hosts para garantizar la migración de las máquinas virtuales.

   ![Incorporación de un nuevo almacén de datos: selección de hosts](media/databox-migration-add-datastore-nfs-select-hosts.png)

7. En el paso 5 del asistente, revise el resumen y seleccione **Finish** (Finalizar).

## <a name="copy-data-to-the-data-box-nfs-datastore"></a>Copia de datos al almacén de datos NFS del dispositivo Data Box

Las máquinas virtuales se pueden migrar o clonar en el nuevo almacén de datos.  Las máquinas virtuales no utilizadas que desee migrar se pueden migrar al almacén de datos NFS del dispositivo Data Box mediante la opción **Storage vMotion**. Las máquinas virtuales activas se pueden clonar al almacén de datos NFS de Data Box.

* Identifique y enumere las máquinas virtuales que se pueden **mover**.
* Identifique y enumere las máquinas virtuales que se deben **clonar**.

### <a name="move-a-virtual-machine-to-a-data-box-datastore"></a>Traslado de una máquina virtual al almacén de datos del dispositivo Data Box

1. Haga clic con el botón derecho en la máquina virtual que desea trasladar al almacén de datos del dispositivo Data Box y seleccione **Migrate** (Migrar).

    ![Migración de una máquina virtual](media/databox-migration-vm-migrate.png)

2. Seleccione **Change storage only** (Cambiar solo almacenamiento) como tipo de migración y, después, seleccione **Next** (Siguiente).

    ![Migración de una máquina virtual: solo almacenamiento](media/databox-migration-vm-migrate-change-storage.png)

3. Seleccione el **almacén de datos del dispositivo Data Box** como destino y, después, seleccione **Next** (Siguiente).

    ![Migración de una máquina virtual: selección del almacén de datos](media/databox-migration-vm-migrate-change-storage-select-datastore.png)

4. Revise la información y seleccione **Finish** (Finalizar).

5. Repita los pasos del 1 al 4 para las máquinas virtuales adicionales.

> [!TIP]
> Puede seleccionar varias máquinas virtuales con el mismo estado de energía (encendido o apagado) y migrarlas en bloque.

La máquina virtual se migrará al almacén de datos NFS del dispositivo Data Box. Después de que se hayan migrado todas las máquinas virtuales, podrá apagar las máquinas virtuales encendidas como preparación para la migración de datos a la solución de VMware en Azure.

### <a name="clone-a-virtual-machine-or-a-virtual-machine-template-to-the-data-box-datastore"></a>Clonación de una máquina virtual o una plantilla de máquina virtual en el almacén de datos del dispositivo Data Box

1. Haga clic con el botón derecho en una máquina virtual o en una plantilla de máquina virtual que desee clonar. Seleccione **Clone** > **Clone to virtual machine** (Clone > Clonar en máquina virtual).

    ![Clonación de una máquina virtual](media/databox-migration-vm-clone.png)

2. Seleccione un nombre para la máquina virtual o la plantilla de máquina virtual clonada.

3. Seleccione la carpeta en la que desea colocar el objeto clonado y, después, seleccione **Next** (Siguiente).

4. Seleccione el clúster o el grupo de recursos donde desea incluir el objeto clonado y, después, haga clic en **Next** (Siguiente).

5. Seleccione **Databox-Datastore** como ubicación de almacenamiento y, después, seleccione **Next** (Siguiente)

    ![Clonación de una máquina virtual: selección del almacén de datos](media/databox-migration-vm-clone-select-datastore.png)

6. Si desea personalizar alguna otra opción del objeto clonado, seleccione las opciones de personalización y, después, seleccione **Next** (Siguiente).

7. Revise las configuraciones y seleccione **Finish** (Finalizar).

8. Repita los pasos del 1 al 7 para otras máquinas virtuales o plantillas de máquina virtual adicionales.

Las máquinas virtuales se clonarán y almacenarán en el almacén de datos NFS del dispositivo Data Box. Después de clonar las máquinas virtuales, asegúrese de que se apagan en preparación para la migración de datos a la solución de VMware en Azure.

### <a name="copy-iso-files-to-the-data-box-datastore"></a>Copia de archivos ISO en el almacén de datos del dispositivo Data Box

1. Desde la interfaz de usuario web del vCenter local, vaya a **Storage** (Almacenamiento).  Seleccione **Databox-Datastore** y, después, seleccione **Files** (Archivos). Cree una carpeta para almacenar los archivos ISO.

    ![Copia de archivos ISO: creación de una nueva carpeta](media/databox-migration-create-folder.png)

2. Asigne un nombre a la carpeta donde se almacenarán los archivos ISO.

3. Haga doble clic en la carpeta que acaba de crear para abrirla.

4. Haga clic en **Upload files** (Cargar archivos) y seleccione los archivos ISO que quiere cargar.
    
    ![Copia de archivos ISO: carga de archivos](media/databox-migration-upload-iso.png)

> [!TIP]
> Si ya tiene archivos ISO en el almacén de datos local, puede seleccionarlos y copiarlos con **Copy to** (Copiar en) en el almacén de datos NFS del dispositivo Data Box.


## <a name="prepare-data-box-for-return"></a>Preparación del dispositivo Data Box para la devolución

Después de que todos los datos de las máquinas virtuales, los datos de las plantillas de máquina virtual y los archivos ISO se hayan copiado en el almacén de datos NFS del dispositivo Data Box, se puede desconectar el almacén de datos del servidor vCenter. Todas las máquinas virtuales y las plantillas de máquina virtual deben quitarse del inventario antes de que se pueda desconectar el almacén de datos.

### <a name="remove-objects-from-inventory"></a>Eliminación de objetos del inventario

1. Desde la interfaz de usuario web del vCenter local, vaya a **Storage** (Almacenamiento). Seleccione **Databox-Datastore** y, después, seleccione **VMs** (Máquinas virtuales).

    ![Eliminación de máquinas virtuales del inventario: apagado](media/databox-migration-select-databox-vm.png)

2. Asegúrese de que todas las máquinas virtuales están apagadas.

3. Seleccione todas las máquinas virtuales, haga clic con el botón derecho y seleccione **Remove from inventory** (Quitar del inventario).

    ![Eliminación de máquinas virtuales del inventario](media/databox-migration-remove-vm-from-inventory.png)

4. Seleccione **VM Templates in Folders** (Plantillas de máquina virtual en carpetas) y repita el paso 3.

### <a name="remove-the-data-box-nfs-datastore-from-vcenter"></a>Eliminación del almacén de datos NFS del dispositivo Data Box de vCenter

Se debe desconectar el almacén de datos NFS del dispositivo Data Box de los hosts VMware ESXi antes de preparar la devolución.

1. Desde la interfaz de usuario web del vCenter local, vaya a **Storage** (Almacenamiento).

2. Haga clic con el botón derecho en **Databox-Datastore** y seleccione **Unmount Datastore** (Desmontar almacén de datos).

    ![Desmontaje del almacén de datos de Data Box](media/databox-migration-unmount-datastore.png)

3. Seleccione todos los hosts ESXi donde esté montado el almacén de datos y seleccione **OK** (Aceptar).

    ![Desmontaje del almacén de datos de Data Box: selección de hosts](media/databox-migration-unmount-datastore-select-hosts.png)

4. Revise y acepte las advertencias y seleccione **OK** (Aceptar).

### <a name="prepare-data-box-for-return-and-then-return-it"></a>Preparación del dispositivo Data Box para la devolución y devolución del dispositivo

Siga los pasos descritos en el artículo [Devolución de Azure Data Box y comprobación de la carga de datos en Azure](../databox/data-box-deploy-picked-up.md) para devolver el dispositivo Data Box. Compruebe el estado de la copia de datos en su cuenta de almacenamiento de Azure. Cuando el estado se muestre como completado, puede comprobar los datos de la cuenta de almacenamiento de Azure.

## <a name="copy-data-from-azure-storage-to-azure-vmware-solution"></a>Copia de datos del almacenamiento de Azure a la solución de VMware en Azure

Los datos copiados en el dispositivo Data Box estarán disponibles en la cuenta de almacenamiento de Azure después de que el estado del pedido del dispositivo Data Box se muestre como completado. En ese momento los datos podrán copiarse en la solución de VMware en Azure. Los datos de la cuenta de almacenamiento deben copiarse en el almacén de datos vSAN de la nube privada mediante el protocolo NFS. 

En primer lugar, copie los datos del almacenamiento de blobs en un disco administrado de una máquina virtual Linux en Azure mediante **AzCopy**. Haga que el disco administrado esté disponible mediante NFS, monte el recurso compartido NFS como un almacén de datos en la nube privada y, después, copie los datos. Este método permite una copia más rápida de los datos en la nube privada.

### <a name="copy-data-to-your-private-cloud-using-a-linux-virtual-machine-and-managed-disks-and-then-export-as-nfs-share"></a>Copia de datos en la nube privada con una máquina virtual Linux y discos administrados, y exportación como recurso compartido NFS

1. Cree una [máquina virtual Linux](../virtual-machines/linux/quick-create-portal.md) en Azure, en la misma región donde se creó la cuenta de almacenamiento y que tiene una conexión de red virtual de Azure a la nube privada.

2. Cree un disco administrado cuya capacidad de almacenamiento sea mayor que la cantidad de datos de blob y [conéctelo a la máquina virtual Linux](../virtual-machines/linux/attach-disk-portal.md).  Si la cantidad de datos de blob es mayor que la capacidad del disco administrado más grande disponible, se deben copiar los datos en varios pasos o se deben usar varios discos administrados.

3. Conéctese a la máquina virtual Linux y monte el disco administrado.

4. Instale [AzCopy en la máquina virtual Linux](../storage/common/storage-use-azcopy-v10.md).

5. Descargue los datos del almacenamiento de blobs de Azure en el disco administrado mediante AzCopy.  Sintaxis del comando: `azcopy copy "https://<storage-account-name>.blob.core.windows.net/<container-name>/*" "<local-directory-path>/"`.  Reemplace `<storage-account-name>` por el nombre de la cuenta de almacenamiento de Azure y `<container-name>` por el contenedor que contiene los datos copiados mediante el dispositivo Data Box.

6. Instale el servidor NFS en la máquina virtual Linux:

    - En una distribución Ubuntu/Debian: `sudo apt install nfs-kernel-server`.
    - En una distribución Enterprise Linux: `sudo yum install nfs-utils`.

7. Cambie el permiso de la carpeta en el disco administrado donde se copiaron los datos del almacenamiento de blobs de Azure.  Cambie los permisos para todas las carpetas que desea exportar como recurso compartido NFS.

    ```bash
    chmod -R 755 /<folder>/<subfolder>
    chown nfsnobody:nfsnobody /<folder>/<subfolder>
    ```

8. Asigne los permisos para que las direcciones IP de cliente accedan al recurso compartido NFS mediante la edición del archivo `/etc/exports`.

    ```bash
    sudo vi /etc/exports
    ```
    
    Escriba las siguientes líneas en el archivo para cada dirección IP de host ESXi de la nube privada.  Si va a crear recursos compartidos para varias carpetas, agregue todas las carpetas.

    ```bash
    /<folder>/<subfolder> <ESXiNode1IP>(rw,sync,no_root_squash,no_subtree_check)
    /<folder>/<subfolder> <ESXiNode2IP>(rw,sync,no_root_squash,no_subtree_check)
    .
    .
    ```

9. Exporte los recursos compartidos NFS mediante el comando `sudo exportfs -a`.

10. Reinicie el servidor de kernel NFS con el comando `sudo systemctl restart nfs-kernel-server`.


### <a name="mount-the-linux-virtual-machine-nfs-share-as-a-datastore-on-a-private-cloud-vcenter-cluster-and-then-copy-data"></a>Montaje del recurso compartido NFS de la máquina virtual Linux como almacén de datos en un clúster de vCenter de nube privada y, después, copia de los datos

El recurso compartido NFS de la máquina virtual Linux debe montarse como un almacén de datos en el clúster de vCenter de la nube privada. Una vez montado, se pueden copiar los datos desde el almacén de datos NFS al almacén de datos vSAN de la nube privada.

1. Inicie sesión en el servidor vCenter de la nube privada.

2. Haga clic con el botón secundario en **Datacenter** (Centro de recursos), seleccione **Storage** (Almacenamiento), después **New Datastore** (Nuevo almacén de datos) y, a continuación, seleccione **Next** (Siguiente).

   ![Incorporación de un nuevo almacén de datos](media/databox-migration-add-datastore.png)

3. En el paso 1 del asistente para agregar un almacén de datos, seleccione el tipo **NFS**.

   ![Incorporación de un nuevo almacén de datos: tipo](media/databox-migration-add-datastore-type.png)

4. En el paso 2 del asistente, seleccione **NFS 3** como la versión de NFS y, a continuación, seleccione **Next** (Siguiente).

   ![Incorporación de un nuevo almacén de datos: versión de NFS](media/databox-migration-add-datastore-nfs-version.png)

5. En el paso 3, especifique el nombre del almacén de datos, la ruta de acceso y el servidor.  Puede usar la dirección IP de la máquina virtual Linux como servidor.  La ruta de acceso de la carpeta tendrá el formato `/<folder>/<subfolder>/`.

   ![Incorporación de un nuevo almacén de datos: configuración de NFS](media/databox-migration-add-datastore-nfs-configuration.png)

6. En el paso 4, seleccione los hosts ESXi en los que desea montar el almacén de datos y, después, seleccione **Next** (Siguiente).  En un clúster, seleccione todos los hosts para garantizar la migración de las máquinas virtuales.

   ![Incorporación de un nuevo almacén de datos: selección de hosts](media/databox-migration-add-datastore-nfs-select-hosts.png)

7. En el paso 5 del asistente, revise el resumen y, después, seleccione **Finish** (Finalizar).

### <a name="add-virtual-machines-and-virtual-machine-templates-from-an-nfs-datastore-to-the-inventory"></a>Adición de máquinas virtuales y plantillas de máquina virtual desde el almacén de datos NFS al inventario

1. Desde la interfaz de usuario web de vCenter de la nube privada, vaya a **Storage** (Almacenamiento).  Seleccione el almacén de datos NFS de la máquina virtual Linux y, después, seleccione **Files** (Archivos).

    ![Selección de archivos de un almacén de datos NFS](media/databox-migration-datastore-select-files.png)

2. Seleccione una carpeta que contenga una máquina virtual o una plantilla de máquina virtual.  En el panel de detalles, seleccione un archivo .vmx para una máquina virtual o un archivo .vmtx para una plantilla de máquina virtual.

3. Seleccione **Register VM** (Registrar máquina virtual) para registrar la máquina virtual en vCenter de la nube privada.

    ![Registro de una máquina virtual](media/databox-migration-datastore-register-vm.png)

4. Seleccione el centro de datos, la carpeta y el clúster o el grupo de recursos donde desea registrar la máquina virtual.

4. Repita los pasos 3 y 4 para todas las máquinas virtuales y plantillas de máquina virtual.

5. Vaya a la carpeta que contiene los archivos ISO.  Seleccione los archivos ISO y, a continuación, seleccione **Copy to** (Copiar en) para copiar los archivos en una carpeta en el almacén de archivos vSAN.

Ahora las máquinas virtuales y las plantillas de máquina virtual están disponibles en el vCenter de la nube privada. Estas máquinas virtuales se deben mover desde el almacén de datos NFS al almacén de datos vSAN antes de poder encenderlas. Puede utilizar la opción **storage vMotion** y seleccione el almacén de datos vSAN como destino para las máquinas virtuales.

Las plantillas de máquina virtual se deben clonar desde el almacén de datos NFS de la máquina virtual Linux al almacén de datos vSAN.

### <a name="clean-up-your-linux-virtual-machine"></a>Limpieza de la máquina virtual Linux

Cuando todos los datos se hayan copiado a la nube privada, puede quitar el almacén de datos NFS de esta.

1. Asegúrese de que todas las máquinas virtuales y las plantillas se han movido y clonado en el almacén de datos vSAN.

2. Quite del inventario todas las plantillas de máquina virtual del almacén de datos NFS.

3. Desmonte el almacén de datos de la máquina virtual Linux del vCenter de la nube privada.

4. Elimine la máquina virtual y el disco administrado de Azure.

5. Si no desea mantener los datos transferidos mediante un dispositivo Data Box en la cuenta de almacenamiento, elimine la cuenta de almacenamiento de Azure.  
    


## <a name="next-steps"></a>Pasos siguientes

* Más información sobre el dispositivo [Data Box](../databox/data-box-overview.md).
* Más información sobre las diferentes opciones para [migrar las cargas de trabajo a la nube privada](migrate-workloads.md).
