---
title: Creación y uso de un recurso compartido de Azure Files para máquinas virtuales Windows
description: Cree y use un recurso compartido de Azure Files en Azure Portal. Conéctelo a una máquina virtual Windows, conéctese al recurso compartido de archivos y cargue un archivo en el recurso compartido de archivos.
author: roygara
ms.service: storage
ms.topic: quickstart
ms.date: 02/01/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 0cbbcc822765097f83fcd1a52ce775c69faba9fa
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836435"
---
# <a name="quickstart-create-and-manage-azure-files-share-with-windows-virtual-machines"></a>Inicio rápido: Creación y administración de un recurso compartido de Azure Files con Windows Virtual Machines

El artículo muestra los pasos básicos para crear y usar un recurso compartido de Azure Files. Este inicio rápido se centra en la configuración rápida de un recurso compartido de Azure Files para experimentar el funcionamiento del servicio. Si necesita instrucciones más detalladas para crear y usar recursos compartidos de archivos de Azure en su propio entorno, consulte [Uso de un recurso compartido de archivos de Azure con Windows](storage-how-to-use-files-windows.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="prepare-your-environment"></a>Preparación del entorno

En este inicio rápido, configurará los elementos siguientes:

- Una cuenta de Azure Storage y un recurso compartido de archivos de Azure
- Una máquina virtual Windows Server 2016 Datacenter

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Para poder trabajar con un recurso compartido de archivos de Azure, es necesario que cree una cuenta de Azure Storage. Una cuenta de almacenamiento v2 de uso general proporciona acceso a todos los servicios de Azure Storage: Blob, File, Queue y Table. El inicio rápido crea una cuenta de almacenamiento de uso general v2, pero los pasos para crear cualquier otro tipo de cuenta de almacenamiento son similares. Una cuenta de almacenamiento puede contener un número ilimitado de recursos compartidos. Un recurso compartido puede almacenar un número ilimitado de archivos, hasta los límites de capacidad de la cuenta de almacenamiento.

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

### <a name="create-an-azure-file-share"></a>Creación de un recurso compartido de archivos de Azure

A continuación creará un recurso compartido de archivos.

1. Cuando se complete la implementación de la cuenta de Azure Storage, seleccione **Ir al recurso**.
1. Seleccione **Archivos** en el panel de la cuenta de almacenamiento.

    ![Seleccionar Archivos](./media/storage-files-quick-create-use-windows/click-files.png)

1. Seleccione **Recurso compartido de archivos**.

    ![Haga clic en el botón para agregar un recurso compartido de archivos](./media/storage-files-quick-create-use-windows/create-file-share.png)

1. Dele al nuevo recurso compartido el nombre *qsfileshare* > escriba "1" para **Cuota** > seleccione **Crear**. El valor máximo de la cuota es 5 TiB, pero para esta guía de inicio rápido solo se necesita 1 GiB.
1. Cree un nuevo archivo txt denominado *qsTestFile* en la máquina local.
1. Seleccione el nuevo recurso compartido de archivos y en su ubicación, haga clic en **Cargar**.

    ![Cargar un archivo](./media/storage-files-quick-create-use-windows/create-file-share-portal5.png)

1. Vaya a la ubicación donde creó el archivo .txt > seleccione *qsTestFile.txt* > seleccione **Cargar**.

Ya ha creado una cuenta de Azure Storage y un recurso compartido de archivos con un solo archivo en Azure. Ahora creará la máquina virtual de Azure con Windows Server 2016 Datacenter para representar el servidor local en esta guía de inicio rápido.

### <a name="deploy-a-vm"></a>Implementación de una máquina virtual

1. A continuación, expanda el menú del lado izquierdo del portal y elija **Crear un recurso** en la esquina superior izquierda de Azure Portal.
1. En el cuadro de búsqueda que está encima de la lista de recursos de **Azure Marketplace**, busque **Windows Server 2016 Datacenter**, selecciónelo y, después, elija **Crear**.
1. En la pestaña **Conceptos básicos**, en **Detalles del proyecto**, seleccione el grupo de recursos que creó para esta guía de inicio rápido.

   ![Especificación de la información básica de la máquina virtual en la hoja del Portal](./media/storage-files-quick-create-use-windows/vm-resource-group-and-subscription.png)

1. En **Detalles de la instancia**, asigne a la máquina virtual el nombre *qsVM*.
1. Deje la configuración predeterminada en **Región**, **Opciones de disponibilidad**, **Imagen** y **Tamaño**.
1. En **Cuenta de administrador**, agregue *VMadmin* como el **Nombre de usuario** y escriba una **Contraseña** para la máquina virtual.
1. En **Reglas de puerto de entrada**, elija **Permitir los puertos seleccionados** y luego seleccione **RDP (3389)** y **HTTP** en la lista desplegable.
1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**. La creación de una máquina virtual nueva tardará varios minutos en completarse.

1. Después de completar la implementación de la máquina virtual, seleccione **Ir al recurso**.

Ya ha creado una nueva máquina virtual y ha conectado un disco de datos. Ahora tiene que conectarse a la máquina virtual.

### <a name="connect-to-your-vm"></a>Conexión a la máquina virtual

1. Seleccione **Conectar** en la página de propiedades de la máquina virtual.

   ![Conexión a una máquina virtual de Azure desde el portal](./media/storage-files-quick-create-use-windows/connect-vm.png)

1. En la página **Connect to virtual machine** (Conectarse a una máquina virtual), mantenga las opciones predeterminadas para conectarse por **dirección IP** a través del **número de puerto** *3389* y seleccione **Descargar archivo RDP**.
1. Abra el archivo RDP que descargó y haga clic en **Conectar** cuando se le solicite.
1. En la ventana **Seguridad de Windows**, seleccione **Más opciones** y, después, **Usar otra cuenta**. Escriba el nombre de usuario con el siguiente formato, *localhost\nombre de usuario*, siendo &lt;nombre de usuario&gt; el nombre de usuario administrador de la máquina virtual que creó. Escriba la contraseña que creó para la máquina virtual y, luego seleccione **Aceptar**.

   ![Más opciones](./media/storage-files-quick-create-use-windows/local-host2.png)

1. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Seleccione **Sí** o **Continuar** para crear la conexión.

## <a name="map-the-azure-file-share-to-a-windows-drive"></a>Asigne el recurso compartido de archivos de Azure a una unidad de Windows

1. En Azure Portal, vaya al recurso compartido de archivos *qsfileshare* y seleccione **Conectar**.
1. Copie el contenido de la segunda casilla y péguelo en el **Bloc de notas**.

   ![Ruta de acceso UNC en el panel Conectar de Azure Files](./media/storage-files-quick-create-use-windows/portal_netuse_connect2.png)

1. En la máquina virtual, abra **Explorador de archivos** y seleccione **Este equipo** en la ventana. Esta selección cambiará los menús disponibles en la barra de herramientas. En el menú **Equipo**, seleccione **Conectar a unidad de red**.
1. Seleccione la letra de unidad y escriba la ruta de acceso UNC. Si ha seguido las sugerencias de nomenclatura en esta guía de inicio rápido, copie *\\qsstorageacct.file.core.windows.net\qsfileshare* desde el **Bloc de notas**.

   Asegúrese de que ambas casillas están seleccionadas.

   ![Captura de pantalla del cuadro de diálogo "Conectar a unidad de red"](./media/storage-files-quick-create-use-windows/mountonwindows10.png)

1. Seleccione **Finalizar**.
1. En el cuadro de diálogo **Seguridad de Windows**:

   - Desde el Bloc de notas, copie el nombre de la cuenta de almacenamiento precedido de AZURE\ y péguelo en el cuadro de diálogo **Windows Security** como nombre de usuario. Si ha seguido las sugerencias de nomenclatura en esta guía de inicio rápido, copie *AZURE\qsstorageacct*.
   - Desde el Bloc de notas, copie la clave de la cuenta de almacenamiento y péguela en el cuadro de diálogo **Windows Security** como contraseña.

      ![Ruta de acceso UNC en el panel Conectar de Azure Files](./media/storage-files-quick-create-use-windows/portal_netuse_connect3.png)

## <a name="create-a-share-snapshot"></a>Creación de una instantánea de recurso compartido

Ahora que ha asignado la unidad, puede crear una instantánea.

1. En el portal, vaya al recurso compartido de archivos y seleccione **Crear instantánea**.

   ![Crear una instantánea](./media/storage-files-quick-create-use-windows/create-snapshot.png)

1. En la máquina virtual, abra *qstestfile.txt* y escriba "este archivo se ha modificado" > Guarde y cierre el archivo.
1. Cree otra instantánea.

## <a name="browse-a-share-snapshot"></a>Búsqueda de una instantánea de recurso compartido

1. En el recurso compartido de archivos, seleccione **Ver instantáneas**.
1. En el panel **Instantáneas de recursos compartidos de archivos**, seleccione la primera instantánea en la lista.

   ![Instantánea seleccionada en la lista de marcas de tiempo](./media/storage-files-quick-create-use-windows/snapshot-list.png)

1. En el panel de esa instantánea, seleccione *qsTestFile.txt*.

## <a name="restore-from-a-snapshot"></a>Restauración desde una instantánea

1. En la hoja de instantánea de recurso compartido de archivo, haga clic con el botón derecho en *qsTestFile* y seleccione el botón **Restaurar**.
1. Seleccione **Sobrescribir el archivo original**.

   ![Botones Descargar y Restaurar](./media/storage-files-quick-create-use-windows/snapshot-download-restore-portal.png)

1. En la máquina virtual, abra el archivo. Se ha restaurado la versión no modificada.

## <a name="delete-a-share-snapshot"></a>Eliminación de una instantánea de recurso compartido

1. En el recurso compartido de archivos, seleccione **Ver instantáneas**.
1. En el panel **Instantáneas de recursos compartidos de archivos**, seleccione las últimas instantáneas en la lista y haga clic en **Eliminar**.

   ![Botón Eliminar](./media/storage-files-quick-create-use-windows/portal-snapshots-delete.png)

## <a name="use-a-share-snapshot-in-windows"></a>Uso de una instantánea de recurso compartido en Windows

Al igual que con las instantáneas VSS en el entorno local, puede ver las instantáneas desde el recurso compartido de archivos de Azure montado en la pestaña Versiones anteriores.

1. Ubique el recurso compartido montado en el Explorador de archivos.

   ![Recurso compartido montado en el Explorador de archivos](./media/storage-files-quick-create-use-windows/snapshot-windows-mount.png)

1. Seleccione *qsTestFile.txt* > haga clic con el botón derecho y seleccione **Propiedades** en el menú.

   ![Haga clic con el botón derecho en el menú para ir a un directorio seleccionado](./media/storage-files-quick-create-use-windows/snapshot-windows-previous-versions.png)

1. Seleccione **Versiones anteriores** para ver la lista de instantáneas de recursos compartidos de este directorio.

1. Seleccione **Abrir** para abrir el archivo.

   ![Pestaña Versiones anteriores](./media/storage-files-quick-create-use-windows/snapshot-windows-list.png)

## <a name="restore-from-a-previous-version"></a>Restaurar desde una versión anterior

1. Seleccione **Restaurar**. Esta acción copia el contenido de todo un directorio de forma recursiva en la ubicación original en el momento de la creación de la instantánea del recurso compartido.

   ![Botón Restaurar en mensaje de advertencia](./media/storage-files-quick-create-use-windows/snapshot-windows-restore.png) Nota: Si el archivo no ha cambiado, no verá una versión anterior de ese archivo, ya que este tiene la misma versión que la instantánea. Esto es coherente con el modo en que funciona en un servidor de archivos de Windows.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [storage-files-clean-up-portal](../../../includes/storage-files-clean-up-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Uso de un recurso compartido de archivos de Azure con Windows](storage-how-to-use-files-windows.md)
