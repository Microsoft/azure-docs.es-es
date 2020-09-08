---
title: 'Inicio rápido: Creación de un blob con el Explorador de Azure Storage'
titleSuffix: Azure Storage
description: Aprenda a usar el Explorador de Azure Storage para crear un contenedor y un blob, descargar el blob en el equipo local y ver todos los blobs del contenedor.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.date: 12/04/2019
ms.author: tamram
ms.openlocfilehash: f8768a1b2542eb0461ef0c50c96e22c80eced57a
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836469"
---
# <a name="quickstart-use-azure-storage-explorer-to-create-a-blob"></a>Inicio rápido: Uso del Explorador de Azure Storage para crear un blob

En esta guía de inicio rápido, aprenderá a usar el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) para crear un contenedor y un blob. A continuación, aprenderá a descargar el blob en un equipo local y a ver todos los blobs en un contenedor. También aprenderá a crear una instantánea de un blob, a administrar directivas de acceso al contenedor y a crear una firma de acceso compartido.

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

Esta guía de inicio rápido requiere que se instale el Explorador de Azure Storage. Para instalar el Explorador de Azure Storage para Windows, Macintosh o Linux, consulte [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/).

## <a name="log-in-to-storage-explorer"></a>Inicie sesión en el Explorador de Microsoft Azure Storage

La primera vez, se muestra la ventana **Explorador de Microsoft Azure Storage: Conectar**. El Explorador de Microsoft Azure Storage proporciona varias maneras de conectar con las cuentas de almacenamiento. En la tabla siguiente se enumeran las distintas maneras en que puede conectarse:

|Tarea|Propósito|
|---|---|
|Agregar una cuenta de Azure | Le redirige a la página de inicio de sesión de su organización para autenticarlo en Azure. |
|Usar una cadena de conexión o un identificador URI de firma de acceso compartido | Puede utilizarse para tener acceso directamente a un contenedor o a la cuenta de almacenamiento con un token de SAS o una cadena de conexión compartida. |
|Usar un nombre y clave de la cuenta de almacenamiento| Use el nombre y la clave de la cuenta de almacenamiento para conectarse a Azure Storage.|

Seleccione **Add an Azure Account** (Agregar una cuenta de Azure) y haga clic en **Iniciar sesión**. Siga las indicaciones de la pantalla para registrarse en su cuenta de Azure.

![Explorador de Microsoft Azure Storage: ventana Conectar](media/storage-quickstart-blobs-storage-explorer/connect.png)

Cuando se completa la conexión, el Explorador de Microsoft Azure Storage se carga y se muestra la pestaña **Explorador**. Esta vista proporciona una visión general de todas las cuentas de Azure Storage, así como del almacenamiento local que se configuró mediante el [emulador de almacenamiento Azurite](../common/storage-use-azurite.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json), las cuentas de [Cosmos DB](../../cosmos-db/storage-explorer.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) o los entornos de [Azure Stack](/azure-stack/user/azure-stack-storage-connect-se?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

![Explorador de Microsoft Azure Storage: ventana Conectar](media/storage-quickstart-blobs-storage-explorer/mainpage.png)

## <a name="create-a-container"></a>Crear un contenedor

Los blobs siempre se cargan en un contenedor. Esto le permite organizar grupos de blobs de la misma forma que organiza archivos en carpetas en el equipo.

Para crear un contenedor, expanda la cuenta de almacenamiento que creó en el paso anterior. Seleccione **Contenedor de blobs**, haga clic con el botón derecho y seleccione **Crear contenedor de blobs**. Escriba el nombre del contenedor de blobs. Para ver una lista de reglas y restricciones en la nomenclatura de contenedores de blobs, consulte la sección [Crear un contenedor](storage-quickstart-blobs-dotnet.md#create-a-container). Cuando haya finalizado, presione **Entrar** para crear el contenedor de blobs. Una vez que el contenedor de blobs se haya creado correctamente, se mostrará en la carpeta **Contenedores de blobs** de la cuenta de almacenamiento seleccionada.

## <a name="upload-blobs-to-the-container"></a>Carga de blobs al contenedor

Blob Storage admite blobs en bloques, blobs en anexos y blobs en páginas. Los archivos VHD utilizados para respaldar VM IaaS son blobs en páginas. Los blobs en anexos se utilizan para el registro, por ejemplo, cuando desea escribir en un archivo y luego sigue agregando más información. La mayoría de los archivos almacenados en Blob Storage son blobs en bloques.

En la cinta de opciones de contenedor, seleccione **Cargar**. Esta operación da la opción de cargar un archivo o una carpeta.

Elija los archivos o carpetas para cargar. Seleccione el **tipo de blob**. Opciones aceptables son **Anexar**, **Paginar** o **Bloquear** blob.

Si carga un archivo .vhd o .vhdx, elija **Upload .vhd/.vhdx files as page blobs (recommended)** [Cargar archivos.vhd/.vhdx como blobs de páginas (recomendado)].

En el campo **Cargar en carpeta (opcional)** , un nombre de carpeta para almacenar los archivos o carpetas en una carpeta bajo el contenedor. Si no se elige ninguna carpeta, los archivos se cargan directamente en el contenedor.

![Explorador de Microsoft Azure Storage: Carga de un blob](media/storage-quickstart-blobs-storage-explorer/uploadblob.png)

Cuando se selecciona **Aceptar**, los archivos seleccionados se ponen en cola para cargar y se cargan de uno en uno. Una vez finalizada la carga, los resultados se muestran en la ventana **Actividades**.

## <a name="view-blobs-in-a-container"></a>Visualización de los blobs de un contenedor

En la aplicación **Explorador de Azure Storage**, seleccione un contenedor en una cuenta de almacenamiento. El panel principal muestra una lista de los blobs en el contenedor seleccionado.

![Explorador de Microsoft Azure Storage: Enumeración de los blobs de un contenedor](media/storage-quickstart-blobs-storage-explorer/listblobs.png)

## <a name="download-blobs"></a>Descargar blobs

Para descargar los blobs mediante el **Explorador de Microsoft Azure Storage**, con un blob seleccionado, seleccione **Descargar** desde la cinta de opciones. Se abre un cuadro de diálogo de archivo que permite escribir un nombre de archivo. Seleccione **Guardar** para iniciar la descarga de un blob en la ubicación local.

## <a name="manage-snapshots"></a>Administración de instantáneas

El Explorador de Microsoft Azure Storage permite tomar y administrar [instantáneas](storage-blob-snapshots.md) de los blobs. Para tomar una instantánea de un blob, haga clic con el botón derecho en el blob y seleccione **Create Snapshot** (Crear instantánea). Para ver las instantáneas de un blob, haga clic con el botón derecho en el blob y seleccione **Manage Snapshots** (Administrar instantáneas). Una lista de las instantáneas para el blob se muestran en la pestaña actual.

![Explorador de Microsoft Azure Storage: Enumeración de los blobs de un contenedor](media/storage-quickstart-blobs-storage-explorer/snapshots.png)

## <a name="manage-access-policies"></a>Administración de directivas de acceso

El Explorador de Storage permite administrar las directivas de acceso para los contenedores dentro de su interfaz de usuario. Hay dos tipos de directivas de acceso seguro (SAS), nivel de servicio y nivel de cuenta. Las de nivel de cuenta tienen como destino la cuenta de almacenamiento y se pueden aplicar a varios servicios y recursos. Las de nivel de servicio se definen en un recurso y con un servicio determinado. Para generar una directiva de acceso seguro de nivel de servicio, haga clic con el botón derecho en un contenedor y seleccione **Manage Access Policies...** (Administrar directivas de acceso). Para generar una directiva de acceso seguro de nivel de cuenta, haga clic con el botón derecho en la cuenta de almacenamiento.

Seleccione **Agregar** para agregar una nueva directiva de acceso y definir sus permisos. Cuando haya finalizado, seleccione **Guardar** para guardar la directiva de acceso. Esta directiva ahora está disponible para usarse al configurar una firma de acceso compartido.

## <a name="work-with-shared-access-signatures"></a>Trabajo con firmas de acceso compartido

Las firmas de acceso compartido (SAS) se pueden recuperar mediante el Explorador de Storage. Haga clic con el botón derecho en una cuenta de almacenamiento, contenedor o blob, y elija **Get Shared Access Signature...**  (Obtener firma de acceso compartido). Elija la hora de inicio y de expiración, y los permisos para la dirección URL de la firma de acceso compartido y seleccione **Crear**. Se proporcionan la dirección URL completa con la cadena de consulta, así como la cadena de consulta solamente y se pueden copiar desde la pantalla siguiente.

![Explorador de Microsoft Azure Storage: Enumeración de los blobs de un contenedor](media/storage-quickstart-blobs-storage-explorer/sharedaccesssignature.png)

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido aprendió a transferir archivos entre un disco local y Azure Blob Storage mediante el **Explorador de Microsoft Azure Storage**. Para más información sobre cómo trabajar con Blob Storage, continúe con los procedimientos de Blob Storage.

> [!div class="nextstepaction"]
> [Procedimientos de las operaciones de Blob Storage](storage-how-to-use-blobs-powershell.md)
