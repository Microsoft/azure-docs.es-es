---
title: Guía de inicio rápido para administrar recursos compartidos de archivos de Azure mediante Azure Portal
description: Vea cómo se crean y administran los recursos compartidos de archivos de Azure en Azure Portal. Cree una cuenta de almacenamiento, cree un recurso compartido de archivos de Azure y úselo.
author: roygara
ms.service: storage
ms.topic: quickstart
ms.date: 10/18/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: f68d9f620b686ab20e93584bdd7c024d8a943577
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88036545"
---
# <a name="quickstart-create-and-manage-azure-file-shares-with-the-azure-portal"></a>Guía de inicio rápido: Creación y administración de recursos compartidos de archivos de Azure con Azure Portal 
[Azure Files](storage-files-introduction.md) es el sencillo sistema de archivos en la nube de Microsoft. Los recursos compartidos de archivos de Azure se pueden montar en Windows, Linux y macOS. En esta guía se describen los conceptos básicos sobre cómo trabajar con recursos compartidos de archivos de Azure mediante [Azure Portal](https://portal.azure.com/).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento
[!INCLUDE [storage-files-create-storage-account-portal](../../../includes/storage-files-create-storage-account-portal.md)]

## <a name="create-an-azure-file-share"></a>Creación de un recurso compartido de archivos de Azure
Para crear un recurso compartido de archivos de Azure:

1. Seleccione la cuenta de almacenamiento desde el panel.
2. En la página de la cuenta de almacenamiento, en la sección **Services** (Servicios), seleccione **Files** (Archivos).
    ![Captura de pantalla de la sección de servicios de la cuenta de almacenamiento; seleccione el servicio Files](media/storage-how-to-use-files-portal/create-file-share-1.png)

3. En el menú de la parte superior de la página **File service**, haga clic en **Recurso compartido de archivos **. Se abre la página **New file share** (Nuevo recurso compartido de archivos).
4. En **Name** (Nombre), escriba *myshare*.
5. Haga clic en **Aceptar** para crear el recurso compartido de archivos de Azure.

Los nombres de recursos compartidos deben estar formados por letras minúsculas, números y guiones sencillos, pero no pueden empezar con un guion. Para obtener detalles completos sobre cómo asignar un nombre a recursos compartidos y archivos, consulte [Asignación de nombres y referencia a recursos compartidos, directorios, archivos y metadatos](https://docs.microsoft.com/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata).

## <a name="use-your-azure-file-share"></a>Uso de un recurso compartido de archivos de Azure
Azure Files proporciona dos métodos para trabajar con archivos y carpetas dentro de un recurso compartido de archivos de Azure: el [protocolo de bloque de mensajes del servidor (SMB)](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx) estándar del sector y el [REST de archivo](https://docs.microsoft.com/rest/api/storageservices/file-service-rest-api). 

Para montar un recurso compartido de archivos con SMB, consulte el siguiente documento según su sistema operativo:
- [Windows](storage-how-to-use-files-windows.md)
- [Linux](storage-how-to-use-files-linux.md)
- [macOS](storage-how-to-use-files-mac.md)

### <a name="using-an-azure-file-share-from-the-azure-portal"></a>Uso de un recurso compartido de archivos de Azure desde Azure Portal
Todas las solicitudes efectuadas a través de Azure Portal se realizan con la API REST de File, lo que le permite crear, modificar y eliminar archivos y directorios en los clientes sin acceso a SMB. Se puede trabajar directamente con el protocolo REST de archivo (es decir, escribir a mano las llamadas a HTTP de REST), pero la manera más habitual de usar este protocolo (que no sea con Azure Portal) es mediante el [módulo de AzureRM PowerShell](storage-how-to-use-files-powershell.md), la [CLI de Azure](storage-how-to-use-files-cli.md) o un SDK de Azure Storage; todos ellos proporcionan un buen contenedor para el protocolo REST de archivo en el lenguaje de programación o script de su elección. 

Cabe esperar que la mayoría de los usuarios de Azure Files deseen trabajar con el recurso compartido de archivos de Azure a través del protocolo SMB, dado que permite usar las aplicaciones y herramientas existentes que se desean usar; sin embargo, existen varias razones por las que es beneficioso usar la API REST de archivo en lugar de SMB, como por ejemplo:

- Debe realizar un cambio rápido en el recurso compartido de archivos de Azure sobre la marcha, por ejemplo, desde un portátil sin acceso a SMB, una tableta o un dispositivo móvil.
- Debe ejecutar un script o una aplicación desde un cliente que no puede montar recursos compartidos de SMB, como los clientes locales que no tienen desbloqueado el puerto 445.
- Se quieren aprovechar las ventajas de los recursos sin servidor, como [Azure Functions](../../azure-functions/functions-overview.md). 

En los ejemplos siguientes se muestra cómo usar Azure Portal para manipular el recurso compartido de archivos de Azure con el protocolo REST de archivo. 

Ahora que ha creado un recurso compartido de archivos de Azure, puede montarlo con SMB en [Windows](storage-how-to-use-files-windows.md), [Linux](storage-how-to-use-files-linux.md) o [macOS](storage-how-to-use-files-mac.md). Como alternativa, puede trabajar con el recurso compartido de archivos de Azure en Azure Portal. 

#### <a name="create-a-directory"></a>Creación de un directorio
Para crear un nuevo directorio denominado *myDirectory* en la raíz del recurso compartido de archivos de Azure:

1. En la página **File Service** (Servicio File), seleccione el recurso compartido de archivos **myshare**. Se abre la página del recurso compartido de archivos.
2. En el menú que aparece en la parte superior de la página, seleccione **+ Add directory** (Agregar directorio) y su cuenta. Se abre la página **New directory** (Nuevo directorio).
3. Escriba *myDirectory* y, luego, haga clic en **OK** (Aceptar).

#### <a name="upload-a-file"></a>Cargar un archivo 
Para demostrar la carga de un archivo, primero debe crear o seleccionar un archivo para cargar. Puede hacerlo por cualquier medio que considere oportuno. Una vez haya seleccionado el archivo que desea cargar:

1. Haga clic en el directorio **myDirectory**. Se abre el panel **myDirectory**.
2. En el menú de la parte superior, haga clic en **Upload** (Cargar). Se abre el panel **Upload files** (Cargar archivos).  
    ![Captura de pantalla del panel de carga de archivos](media/storage-how-to-use-files-portal/upload-file-1.png)

3. Haga clic en el icono de carpeta para abrir una ventana donde examinar los archivos locales. 
4. Seleccione un archivo y haga clic en **Open** (Abrir). 
5. En la página **Upload files** (Cargar archivos), compruebe el nombre de archivo y, a continuación, haga clic en **Upload** (Cargar).
6. Cuando termine, el archivo debe aparecer en la lista en la página **myDirectory**.

#### <a name="download-a-file"></a>Descarga de un archivo
Para descargar una copia del archivo cargado, haga clic en él con el botón derecho. Después de hacer clic en el botón de descarga, la experiencia exacta dependerá del sistema operativo y el explorador que use.

## <a name="clean-up-resources"></a>Limpieza de recursos
[!INCLUDE [storage-files-clean-up-portal](../../../includes/storage-files-clean-up-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [¿Qué es Azure Files?](storage-files-introduction.md)
