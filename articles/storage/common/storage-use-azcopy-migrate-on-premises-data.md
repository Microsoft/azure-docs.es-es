---
title: 'Tutorial: Migración de datos locales a Azure Storage con AzCopy | Microsoft Docs'
description: En este tutorial, va a utilizar AzCopy para migrar datos al contenido de archivos, blobs y tablas o copiarlos. Realice la migración de los datos fácilmente desde el almacenamiento local a Azure Storage.
author: normesta
ms.service: storage
ms.topic: tutorial
ms.date: 05/14/2019
ms.author: normesta
ms.reviewer: seguler
ms.subservice: common
ms.openlocfilehash: f2b46e37241cf142f751f2f65a1ef9f092166ec8
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "86120643"
---
#  <a name="tutorial-migrate-on-premises-data-to-cloud-storage-with-azcopy"></a>Tutorial: Migración de datos locales a un almacenamiento en la nube con AzCopy

AzCopy es una herramienta de línea de comandos para copiar datos de Azure Blob Storage, Azure Files y Azure Table Storage (o en ellos) mediante comandos simples. Los comandos están diseñados para optimizar el rendimiento. Con AzCopy, puede copiar datos entre un sistema de archivos y una cuenta de almacenamiento o entre cuentas de almacenamiento. AzCopy se puede usar para copiar datos desde los datos locales a una cuenta de almacenamiento.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Cree una cuenta de almacenamiento. 
> * Use AzCopy para cargar todos los datos.
> * Modifique los datos para fines de prueba.
> * Cree una tarea programada o trabajo cron para identificar los nuevos archivos que cargar.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, descargue la versión más reciente de AzCopy. Consulte [Introducción a AzCopy](storage-use-azcopy-v10.md).

Si tiene Windows, necesitará [Schtasks](https://msdn.microsoft.com/library/windows/desktop/bb736357(v=vs.85).aspx), ya que este tutorial lo utiliza para programar una tarea. Los usuarios de Linux utilizarán en su lugar el comando crontab.

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

## <a name="create-a-container"></a>Crear un contenedor

El primer paso es crear un contenedor, ya que los blobs siempre se deben cargar en un contenedor. Los contenedores se utilizan como un método para organizar grupos de blobs de la misma forma que los archivos en el equipo, en carpetas.

Siga estos pasos para crear un contenedor:

1. Seleccione el botón **Cuentas de almacenamiento** de la página principal y la cuenta de almacenamiento que ha creado.
2. Seleccione **Blobs** en **Servicios** y **Contenedor**.

   ![Crear un contenedor](media/storage-azcopy-migrate-on-premises-data/CreateContainer.png)
 
El nombre de contenedor debe empezar por una letra o un número. Solo puede contener letras, números y el carácter de guion (-). Para conocer más reglas sobre la nomenclatura de los contenedores de almacenamiento, consulte [Asignación de nombres y referencias a contenedores, blobs y metadatos](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata).

## <a name="download-azcopy"></a>Descargar AzCopy

Descargue el archivo ejecutable AzCopy V10.

- [Windows](https://aka.ms/downloadazcopy-v10-windows) (zip)
- [Linux](https://aka.ms/downloadazcopy-v10-linux) (tar)
- [MacOS](https://aka.ms/downloadazcopy-v10-mac) (zip)

Coloque el archivo AzCopy en cualquier lugar en el equipo. Agregue la ubicación del archivo a la variable de la ruta de acceso del sistema para que pueda hacer referencia a este archivo ejecutable desde cualquier carpeta del equipo.

## <a name="authenticate-with-azure-ad"></a>Autenticación mediante Azure Active Directory

En primer lugar, asigne el rol de [Colaborador de datos de Storage Blob](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-queue-data-contributor) a su identidad. Consulte [Concesión de acceso a datos de blob y cola de Azure con RBAC en Azure Portal](https://docs.microsoft.com/azure/storage/common/storage-auth-aad-rbac-portal).

A continuación, abra un símbolo del sistema, escriba el siguiente comando y presione la tecla ENTRAR.

```azcopy
azcopy login
```

Este comando devuelve un código de autenticación y la dirección URL de un sitio web. Abra el sitio web, proporcione el código y, después, elija el botón **Siguiente**.

![Crear un contenedor](media/storage-use-azcopy-v10/azcopy-login.png)

Aparece una ventana de inicio de sesión. En esa ventana, inicie sesión en la cuenta de Azure con sus credenciales de la cuenta de Azure. Cuando haya iniciado sesión correctamente, puede cerrar la ventana del explorador y comenzar a usar AzCopy.

## <a name="upload-contents-of-a-folder-to-blob-storage"></a>Carga del contenido en una carpeta en Blob Storage

Puede usar AzCopy para cargar todos los archivos en una carpeta en Blob Storage en [Windows](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy) o [Linux](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-linux). Para cargar todos los blobs en una carpeta, escriba el siguiente comando de AzCopy:

```AzCopy
azcopy copy "<local-folder-path>" "https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>" --recursive=true
```

* Reemplace el marcador de posición `<local-folder-path>` por la ruta de acceso a una carpeta que contiene los archivos (por ejemplo: `C:\myFolder` o `/mnt/myFolder`).

* Reemplace el marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

* Reemplace el marcador de posición `<container-name>` por el nombre del contenedor que ha creado.

Para cargar el contenido del directorio especificado repetidamente en el almacenamiento de blob, especifique la opción `--recursive`. Al ejecutar AzCopy con esta opción, se cargan también todas las subcarpetas y sus archivos.

## <a name="upload-modified-files-to-blob-storage"></a>Carga de archivos modificados en Blob Storage

Puede utilizar AzCopy para cargar archivos según la hora de la última modificación. 

Para probar esto, modifique o cree archivos en el directorio de origen para fines de prueba. A continuación, utilice el comando `sync` de AzCopy.

```AzCopy
azcopy sync "<local-folder-path>" "https://<storage-account-name>.blob.core.windows.net/<container-name>" --recursive=true
```

* Reemplace el marcador de posición `<local-folder-path>` por la ruta de acceso a una carpeta que contiene los archivos (por ejemplo: `C:\myFolder` o `/mnt/myFolder`).

* Reemplace el marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

* Reemplace el marcador de posición `<container-name>` por el nombre del contenedor que ha creado.

Para más información sobre el comando `sync`, consulte [Sincronizar archivos](storage-use-azcopy-blobs.md#synchronize-files).

## <a name="create-a-scheduled-task"></a>Creación de una tarea programada

Puede crear una tarea programada o un trabajo cron que ejecute un script de AzCopy. El script identifica y carga los datos locales nuevos en el almacenamiento en la nube en un intervalo de tiempo específico.

Copie el comando de AzCopy en un editor de texto. Actualice los valores de parámetro del comando de AzCopy con los valores adecuados. Guarde el archivo como `script.sh` (Linux) o `script.bat` (Windows) para AzCopy. 

En estos ejemplos se supone que la carpeta se denomina `myFolder`, que el nombre de la cuenta de almacenamiento es `mystorageaccount` y que el nombre del contenedor es `mycontainer`.

> [!NOTE]
> El ejemplo de Linux anexa un token de SAS. Debe proporcionar uno en el comando. La versión actual de AzCopy V10 no es compatible con la autorización de Azure AD en los trabajos de Cron.

# <a name="linux"></a>[Linux](#tab/linux)

```bash
azcopy sync "/mnt/myfiles" "https://mystorageaccount.blob.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-05-30T06:57:40Z&st=2019-05-29T22:57:40Z&spr=https&sig=BXHippZxxx54hQn%2F4tBY%2BE2JHGCTRv52445rtoyqgFBUo%3D" --recursive=true
```

# <a name="windows"></a>[Windows](#tab/windows)

```bash
azcopy sync "C:\myFolder" "https://mystorageaccount.blob.core.windows.net/mycontainer" --recursive=true
```

---

En este tutorial, se utiliza [Schtasks](https://msdn.microsoft.com/library/windows/desktop/bb736357(v=vs.85).aspx) para crear una tarea programada en Windows. El comando [Crontab](http://crontab.org/) se utiliza para crear un trabajo cron en Linux.

 **Schtasks** permite a un administrador crear, eliminar, consultar, cambiar, ejecutar y finalizar tareas programadas en un equipo local o remoto. **Cron** permite a los usuarios de Unix y Linux ejecutar comandos o scripts en una fecha y hora especificadas mediante [expresiones cron](https://en.wikipedia.org/wiki/Cron#CRON_expression).

# <a name="linux"></a>[Linux](#tab/linux)

Para crear un trabajo cron en Linux, escriba el siguiente comando en un terminal:

```bash
crontab -e
*/5 * * * * sh /path/to/script.sh
```

Especificar la expresión cron `*/5 * * * *` en el comando indica que el script de shell `script.sh` debe ejecutarse cada cinco minutos. Puede programar que el script se ejecute en un momento determinado todos los días, meses o años. Para más información sobre cómo establecer la fecha y la hora de ejecución de un trabajo, consulte las [expresiones cron](https://en.wikipedia.org/wiki/Cron#CRON_expression).

# <a name="windows"></a>[Windows](#tab/windows)

Para crear una tarea programada en Windows, escriba el siguiente comando en un símbolo del sistema o en PowerShell:

En este ejemplo se asume que el script se encuentra en la unidad raíz del equipo, pero su el puede estar en cualquier lugar que quiera.

```cmd
schtasks /CREATE /SC minute /MO 5 /TN "AzCopy Script" /TR C:\script.bat
```

El comando utiliza:
- El parámetro `/SC` para especificar una programación por minuto.
- El parámetro `/MO` para especificar un intervalo de cinco minutos.
- El parámetro `/TN` para especificar el nombre de la tarea.
- El parámetro `/TR` para especificar la ruta de acceso al archivo `script.bat`.

Para más información sobre cómo crear una tarea programada en Windows, consulte [Schtasks](https://technet.microsoft.com/library/cc772785(v=ws.10).aspx#BKMK_minutes).

---

Para asegurarse de que la tarea programada o el trabajo cron se ejecutan correctamente, cree nuevos archivos en el directorio `myFolder`. Espere cinco minutos para confirmar que los nuevos archivos se han cargado en la cuenta de almacenamiento. Vaya al directorio de registro para ver los registros de salida de la tarea programada o el trabajo cron.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las formas de mover datos locales a Azure Storage y viceversa, siga este vínculo:

* [Movimiento de datos hacia Azure Storage y desde este](https://docs.microsoft.com/azure/storage/common/storage-moving-data?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)  

Para más información sobre AzCopy, consulte cualquiera de estos artículos:

* [Introducción a AzCopy](storage-use-azcopy-v10.md)

* [Transferencia de datos con AzCopy y Blob Storage](storage-use-azcopy-blobs.md)

* [Transferencia de datos con AzCopy y File Storage](storage-use-azcopy-files.md)

* [Transferencia de datos con AzCopy y cubos de Amazon S3](storage-use-azcopy-s3.md)
 
* [Configurar, optimizar y solucionar problemas de AzCopy](storage-use-azcopy-configure.md)
