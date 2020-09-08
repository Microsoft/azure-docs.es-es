---
title: Introducción al Explorador de Storage | Microsoft Docs
description: Comience a administrar recursos de Azure Storage con el Explorador de Storage. Descargue e instale el Explorador de Azure Storage, conéctese a una cuenta de almacenamiento o servicio, y mucho más.
services: storage
author: cawaMS
ms.service: storage
ms.devlang: multiple
ms.topic: article
ms.date: 11/08/2019
ms.author: cawa
ms.openlocfilehash: 9922601d67393d103b95da7286d8db71d913434a
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88235252"
---
# <a name="get-started-with-storage-explorer"></a>Introducción al Explorador de Storage

## <a name="overview"></a>Información general

Explorador de Microsoft Azure Storage es una aplicación independiente que facilita el trabajo con los datos de Azure Storage en Windows, macOS y Linux. En este artículo aprenderá varias maneras de conectarse a cuentas de Azure Storage y de administrarlas.

![Explorador de Microsoft Azure Storage][0]

## <a name="prerequisites"></a>Requisitos previos

# <a name="windows"></a>[Windows](#tab/windows)

El Explorador de Storage es compatible con las siguientes versiones de Windows:

* Windows 10 (recomendado)
* Windows 8
* Windows 7

En todas las versiones de Windows, el Explorador de Storage requiere como mínimo .NET Framework 4.7.2.

# <a name="macos"></a>[macOS](#tab/macos)

El Explorador de Storage es compatible con las siguientes versiones de macOS:

* macOS 10.12 Sierra y versiones posteriores

# <a name="linux"></a>[Linux](#tab/linux)

El Explorador de Storage está disponible en [Snap Store](https://snapcraft.io/storage-explorer) para las distribuciones más comunes de Linux. Para esta instalación, se recomienda Snap Store. El complemento de Explorador de Storage instala todas sus dependencias y actualizaciones cuando se publican nuevas versiones en Snap Store.

Para saber cuáles son las distribuciones admitidas, consulte la [página de instalación de snapd](https://snapcraft.io/docs/installing-snapd).

El Explorador de Storage requiere que se use un administrador de contraseñas. Por consiguiente, es posible que tenga que conectarse a un administrador de contraseñas manualmente. Puede conectar el Explorador de Storage al administrador de contraseñas del sistema mediante la ejecución del siguiente comando:

```bash
snap connect storage-explorer:password-manager-service :password-manager-service
```

El Explorador de Storage también se puede descargar como *.tar.gz*. Las dependencias se deben instalar manualmente. Las siguientes distribuciones de Linux admiten la instalación mediante *.tar.gz*:

* Ubuntu 20.04 x64
* Ubuntu 18.04 x64
* Ubuntu 16.04 x64

La instalación mediante *.tar.gz* podría funcionar en otras distribuciones, pero las únicas oficialmente compatibles son las que aparecen en la lista.

Para más información sobre la instalación del Explorador de Storage en Linux, consulte el apartado acerca de las [dependencias de Linux](https://docs.microsoft.com/azure/storage/common/storage-explorer-troubleshooting#linux-dependencies) en la guía para la solución de problemas del Explorador de Azure Storage.

---

## <a name="download-and-install"></a>Descargar e instalar

Para descargar e instalar el Explorador de Storage, consulte [Explorador de Azure Storage](https://www.storageexplorer.com).

## <a name="connect-to-a-storage-account-or-service"></a>Conexión a una cuenta de almacenamiento o servicio

El Explorador de Microsoft Azure Storage proporciona varias maneras de conectar con las cuentas de almacenamiento. En general, puede:

* [Iniciar sesión en Azure para tener acceso a las suscripciones y sus recursos](#sign-in-to-azure)
* [Vincular un recurso específico de Storage o CosmosDB](#attach-a-specific-resource)

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

> [!NOTE]
> Para obtener acceso total a los recursos tras el inicio de sesión, el Explorador de Storage requiere permisos de administración (Azure Resource Manager) y en el nivel de datos. Esto significa que necesita permisos de Azure Active Directory que proporcionen acceso a la cuenta de Storage, los contenedores de la cuenta y los datos de los contenedores. Si solo tiene permisos en el nivel de datos, considere la posibilidad de [agregar un recurso mediante Azure AD](#add-a-resource-via-azure-ad). Para más información sobre los permisos específicos que requiere el Explorador de Storage, consulte la [guía para la solución de problemas del Explorador de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-explorer-troubleshooting#rbac-permissions-issues).

1. En el Explorador de Storage, seleccione **Ver** > **Administración de cuentas**, o bien seleccione el botón **Administrar cuentas**.

    ![Administración de cuentas][1]

1. En **ADMINISTRACIÓN DE CUENTAS** se muestran ahora todas las cuentas de Azure en las que ha iniciado sesión. Para conectarse a otra cuenta, seleccione **Agregar una cuenta**.

1. En **Conectar a Azure Storage**, seleccione una nube de Azure en el **entorno de Azure** para iniciar sesión en una nube nacional o en Azure Stack. Después de elegir el entorno, seleccione **Siguiente**.

    ![Opción para iniciar sesión][2]

    El Explorador de Storage abre una página para que inicie sesión. Para más información, consulte [Conexión del Explorador de Storage a una suscripción de Azure Stack o una cuenta de almacenamiento](/azure-stack/user/azure-stack-storage-connect-se).

1. Después de iniciar sesión correctamente con una cuenta de Azure, tanto la cuenta como las suscripciones de Azure asociadas con esa cuenta aparecen en **ADMINISTRACIÓN DE CUENTAS**. Seleccione **Todas las suscripciones** para alternar la opción elegida entre todas las suscripciones de Azure de la lista o ninguna. Elija las suscripciones de Azure con las que desea trabajar y luego seleccione **Aplicar**.

    ![Selección de suscripciones de Azure][3]

    **EXPLORADOR** muestra las cuentas de almacenamiento asociadas a las suscripciones de Azure seleccionadas.

    ![Suscripciones de Azure seleccionadas][4]

### <a name="attach-a-specific-resource"></a>Vincular un recurso específico

Hay varias formas de establecer una vinculación con un recurso en el Explorador de Storage:

* [Agregar un recurso a través de Azure AD](#add-a-resource-via-azure-ad). Si solo tiene permisos en el nivel de datos, puede usar esta opción para agregar un contenedor de blobs o un contenedor de almacenamiento de blobs de Azure Data Lake Storage Gen2.
* [Usar una cadena de conexión](#use-a-connection-string). Use esta opción si tiene una cadena de conexión a una cuenta de Storage. El Explorador de Storage admite las cadenas de conexión tanto de la clave como de la [firma de acceso compartido](storage/common/storage-dotnet-shared-access-signature-part-1.md).
* [Use un identificador URI de firma de acceso compartido](#use-a-shared-access-signature-uri). Si tiene un [identificador URI de firma de acceso compartido](storage/common/storage-dotnet-shared-access-signature-part-1.md) en un contenedor de blobs, recurso compartido de archivos, cola o tabla, úselo para establecer la asociación con el recurso. Para obtener un identificador URI de firma de acceso compartido, puede usar el [Explorador de Storage](#generate-a-sas-in-storage-explorer) o [Azure Portal](https://portal.azure.com).
* [Usar un nombre y una clave](#use-a-name-and-key). Si conoce cualquiera de las claves de cuenta para la cuenta de Storage, puede usar esta opción para conectarse rápidamente. Busque las claves en la página de la cuenta de almacenamiento; para ello, seleccione **Configuración** > **Claves de acceso** en [Azure Portal](https://portal.azure.com).
* [Asociar a un emulador local](#attach-to-a-local-emulator). Si usa uno de los emuladores de Azure Storage disponibles, use esta opción para conectarse fácilmente al emulador.
* [Conectarse a una cuenta de Azure Cosmos DB mediante una cadena de conexión](#connect-to-an-azure-cosmos-db-account-by-using-a-connection-string). Use esta opción si tiene una cadena de conexión para una instancia de Cosmos DB.
* [Conectarse a Azure Data Lake Store mediante el URI](#connect-to-azure-data-lake-store-by-uri). Use esta opción si tiene un URI para Azure Data Lake Store.

#### <a name="add-a-resource-via-azure-ad"></a>Agregar un recurso a través de Azure AD

1. Seleccione el símbolo **Conectar** para abrir **Conectar a Azure Storage**.

    ![Opción de conexión a Almacenamiento de Azure][9]

1. Si aún no lo ha hecho, use la opción **Agregar una cuenta de Azure** para iniciar sesión en la cuenta de Azure que tiene acceso al recurso. Después de iniciar sesión, vuelva a **Conectar a Azure Storage**.

1. Seleccione la opción **Agregar un recurso a través de Azure Active Directory (Azure AD)** y seleccione **Siguiente**.

1. Seleccione una cuenta de Azure y un inquilino. Estos valores deben tener acceso al recurso de Storage con el que desea realizar la asociación. Seleccione **Next** (Siguiente).

1. Elija el tipo de recurso que desea asociar. Escriba la información necesaria para establecer la conexión. 

   La información que escriba en esta página dependerá del tipo de recurso que va a agregar. Asegúrese de elegir el tipo de recurso correcto. Cuando haya especificado toda la información necesaria, seleccione **Siguiente**.

1. Consulte el **resumen de la conexión** para asegurarse de que toda la información es correcta. Si es así, seleccione **Conectar**. De lo contrario, seleccione **Atrás** para volver a las páginas anteriores para corregir cualquier información incorrecta.

Una vez que se agrega la conexión, el árbol de recursos va al nodo que representa la conexión. El recurso aparece en **Local y asociado** > **Cuentas de almacenamiento** >  **(Contenedores asociados)**  > **Contenedores de blob**. Si el Explorador de Storage no ha podido agregar la conexión o si no puede acceder a los datos después de agregar correctamente la conexión, consulte la [guía de solución de problemas del Explorador de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-explorer-troubleshooting).

#### <a name="use-a-connection-string"></a>Usar una cadena de conexión

1. Seleccione el símbolo **Conectar** para abrir **Conectar a Azure Storage**.

    ![Opción de conexión a Almacenamiento de Azure][9]

1. Seleccione **Usar una cadena de conexión** y, a continuación, seleccione **Siguiente**.

1. Elija un nombre para mostrar para la conexión y escríbalo en la cadena de conexión. Después, seleccione **Siguiente**.

1. Consulte el **resumen de la conexión** para asegurarse de que toda la información es correcta. Si es así, seleccione **Conectar**. De lo contrario, seleccione **Atrás** para volver a las páginas anteriores para corregir cualquier información incorrecta.

Una vez que se agrega la conexión, el árbol de recursos va al nodo que representa la conexión. El recurso aparece en **Local y asociado** > **Cuentas de almacenamiento**. Si el Explorador de Storage no ha podido agregar la conexión o si no puede acceder a los datos después de agregar correctamente la conexión, consulte la [guía de solución de problemas del Explorador de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-explorer-troubleshooting).

#### <a name="use-a-shared-access-signature-uri"></a>Uso de un identificador URI de firma de acceso compartido

1. Seleccione el símbolo **Conectar** para abrir **Conectar a Azure Storage**.

    ![Opción de conexión a Almacenamiento de Azure][9]

1. Seleccione **Usar un URI de firma de acceso compartido (SAS)** y, a continuación, seleccione **Siguiente**.

1. Elija un nombre para mostrar para la conexión y escriba el URI de firma de acceso compartido. El punto de conexión de servicio para el tipo de recurso que se va a asociar debería autorrellenarse. Si utiliza un punto de conexión personalizado, es posible que no lo haga. Seleccione **Next** (Siguiente).

1. Consulte el **resumen de la conexión** para asegurarse de que toda la información es correcta. Si es así, seleccione **Conectar**. De lo contrario, seleccione **Atrás** para volver a las páginas anteriores para corregir cualquier información incorrecta.

Una vez que se agrega la conexión, el árbol de recursos va al nodo que representa la conexión. El recurso aparece en **Local y asociado** > **Cuentas de almacenamiento** >  **(Contenedores asociados)**  > *el nodo de servicio del tipo de contenedor que ha asociado*. Si el Explorador de Storage no pudo agregar la conexión, consulte la [guía para la solución de problemas del Explorador de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-explorer-troubleshooting). Consulte la guía para la solución de problemas si no puede acceder a los datos después de agregar correctamente la conexión.

#### <a name="use-a-name-and-key"></a>Usar un nombre y una clave

1. Seleccione el símbolo **Conectar** para abrir **Conectar a Azure Storage**.

    ![Opción de conexión a Almacenamiento de Azure][9]

1. Seleccione **Usar un nombre y clave de la cuenta de almacenamiento** y, a continuación, seleccione **Siguiente**.

1. Elija un nombre para mostrar para la conexión.

1. Escriba el nombre de la cuenta de Storage y cualquiera de sus claves de acceso.

1. Elija el **dominio de Storage** que va a usar y, a continuación, seleccione **Siguiente**.

1. Consulte el **resumen de la conexión** para asegurarse de que toda la información es correcta. Si es así, seleccione **Conectar**. De lo contrario, seleccione **Atrás** para volver a las páginas anteriores para corregir cualquier información incorrecta.

Una vez que se agrega la conexión, el árbol de recursos va al nodo que representa la conexión. El recurso aparece en **Local y asociado** > **Cuentas de almacenamiento**. Si el Explorador de Storage no ha podido agregar la conexión o si no puede acceder a los datos después de agregar correctamente la conexión, consulte la [guía de solución de problemas del Explorador de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-explorer-troubleshooting).

#### <a name="attach-to-a-local-emulator"></a>Vincular un emulador local

Actualmente, el Explorador de Storage admite dos emuladores de almacenamiento oficiales:

* [Emulador de Azure Storage](storage/common/storage-use-emulator.md) (solo Windows)
* [Azurite](https://github.com/azure/azurite) (Windows, macOS o Linux)

Si el emulador escucha los puertos predeterminados, puede usar el nodo **Emulador: puertos predeterminados** para acceder a él. Busque **Emulador: puertos predeterminados** en **Local y asociado** > **Cuentas de almacenamiento**.

Si quiere usar otro nombre para la conexión o si el emulador no se está ejecutando en los puertos predeterminados, siga estos pasos:

1. Inicie el emulador. Escriba el comando `AzureStorageEmulator.exe status` para mostrar los puertos de cada tipo de servicio.

   > [!IMPORTANT]
   > Explorador de Storage no inicia automáticamente el emulador. Debe iniciarlo manualmente.

1. Seleccione el símbolo **Conectar** para abrir **Conectar a Azure Storage**.

    ![Opción de conexión a Almacenamiento de Azure][9]

1. Seleccione **Asociar a un emulador local** y **Siguiente**.

1. Elija un nombre para mostrar para la conexión y escríbalo en los puertos que está escuchando el emulador para cada tipo de servicio. **Asociar a un emulador local** sugiere los valores de puerto predeterminados para la mayoría de los emuladores. **Puerto de archivos** está en blanco porque ninguno de los emuladores oficiales admite actualmente el servicio Files. Si el emulador que utiliza es compatible con Files, puede especificar el puerto que va a usar. Después, seleccione **Siguiente**.

1. Consulte el **resumen de la conexión** y asegúrese de que toda la información es correcta. Si es así, seleccione **Conectar**. De lo contrario, seleccione **Atrás** para volver a las páginas anteriores para corregir cualquier información incorrecta.

Una vez que se agrega la conexión, el árbol de recursos va al nodo que representa la conexión. El nodo debería aparecer en **Local y asociado** > **Cuentas de almacenamiento**. Si el Explorador de Storage no ha podido agregar la conexión o si no puede acceder a los datos después de agregar correctamente la conexión, consulte la [guía de solución de problemas del Explorador de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-explorer-troubleshooting).

#### <a name="connect-to-an-azure-cosmos-db-account-by-using-a-connection-string"></a>Conexión a una cuenta de Azure Cosmos DB mediante una cadena de conexión

En lugar de administrar cuentas de Azure Cosmos DB mediante una suscripción de Azure, puede conectarse a Azure Cosmos DB mediante una cadena de conexión. Para conectarse, siga estos pasos:

1. En **EXPLORADOR**, expanda **Local y asociado**, haga clic con el botón derecho en **Cuentas de Azure Cosmos DB** y seleccione **Conectar a Azure Cosmos DB**.

    ![Conectarse a Azure Cosmos DB mediante una cadena de conexión][21]

1. Seleccione la API de Azure Cosmos DB, escriba los datos de la **cadena de conexión** y, a continuación, seleccione **Aceptar** para conectarse a la cuenta de Azure Cosmos DB. Para más información sobre cómo recuperar la cadena de conexión, consulte [Administración de una cuenta de Azure Cosmos](https://docs.microsoft.com/azure/cosmos-db/manage-account).

    ![Cadena de conexión][22]

#### <a name="connect-to-azure-data-lake-store-by-uri"></a>Conexión a Azure Data Lake Store mediante el URI

Puede acceder a un recurso que no esté en su suscripción. Necesita que un usuario que tenga acceso a ese recurso le proporcione el identificador URI del recurso. Después de iniciar sesión, conéctese a Data Lake Store mediante el identificador URI. Para conectarse, siga estos pasos:

1. En **EXPLORADOR**, expanda **Local y asociado**.

1. Haga clic con el botón derecho en **Data Lake Storage Gen1** y seleccione **Conectar a Data Lake Storage Gen1**.

    ![Menú contextual Connect to Data Lake Store (Conectar a Data Lake Store)](./media/vs-azure-tools-storage-manage-with-storage-explorer/storage-explorer-connect-data-lake-storage.png)

1. Escriba el identificador URI y seleccione **Aceptar**. Data Lake Store aparece en **Data Lake Storage**.

    ![Resultado de Connect to Data Lake Store (Conectar a Data Lake Store)](./media/vs-azure-tools-storage-manage-with-storage-explorer/storage-explorer-attach-data-lake-finished.png)

En este ejemplo se usa Data Lake Storage Gen1. Azure Data Lake Storage Gen2 ya está disponible. Para más información, consulte [¿Qué es Azure Data Lake Storage Gen1?](./data-lake-store/data-lake-store-overview.md).

## <a name="generate-a-shared-access-signature-in-storage-explorer"></a>Generación de una firma de acceso compartido en el Explorador de Storage<a name="generate-a-sas-in-storage-explorer"></a>

### <a name="account-level-shared-access-signature"></a>Firma de acceso compartido en el nivel de cuenta

1. Haga clic con el botón derecho en la cuenta de Storage que quiere compartir y seleccione **Obtener firma de acceso compartido**.

    ![Opción de menú contextual Obtener firma de acceso compartido][14]

1. En **Firma de acceso compartido**, especifique el período y los permisos que desee para la cuenta y seleccione **Crear**.

    ![Obtener una firma de acceso compartido][15]

1. Copie la **cadena de conexión** o la **cadena de consulta** sin formato en el Portapapeles.

### <a name="service-level-shared-access-signature"></a>Firma de acceso compartido en el nivel de servicio

Puede obtener una firma de acceso compartido en el nivel de servicio. Para más información, consulte [Obtención de la SAS para un contenedor de blobs](vs-azure-tools-storage-explorer-blobs.md#get-the-sas-for-a-blob-container).

## <a name="search-for-storage-accounts"></a>Búsqueda de cuentas de almacenamiento

Para buscar un recurso de almacenamiento, puede buscar en el panel **EXPLORADOR**.

A medida que escribe texto en el cuadro de búsqueda, el Explorador de Storage muestra todos los recursos que coinciden con el valor de búsqueda especificado hasta ese punto. En este ejemplo se muestra la búsqueda de **puntos de conexión**:

![Búsqueda de cuenta de almacenamiento][23]

> [!NOTE]
> Para agilizar la búsqueda, use **Administración de cuentas** para anular la selección de las suscripciones que no contengan el elemento que busca. También puede hacer clic con el botón derecho en un nodo y seleccionar **Buscar desde aquí** para comenzar a buscar desde un nodo específico.
>
>

## <a name="next-steps"></a>Pasos siguientes

* [Administración de recursos de Azure Blob Storage con el Explorador de Storage](vs-azure-tools-storage-explorer-blobs.md)
* [Uso de datos con el Explorador de Azure Storage](./cosmos-db/storage-explorer.md)
* [Administración de recursos de Azure Data Lake Store en el Explorador de Storage](./data-lake-store/data-lake-store-in-storage-explorer.md)

[0]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/Overview.png
[1]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ManageAccounts.png
[2]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/connect-to-azure-storage-azure-environment.png
[3]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/account-panel-subscriptions-apply.png
[4]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/SubscriptionNode.png
[5]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog.png
[7]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/PortalAccessKeys.png
[8]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/AccessKeys.png
[9]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog.png
[10]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog-AddWithKeySelected.png
[11]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog-NameAndKeyPage.png
[12]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/AttachedWithKeyAccount.png
[13]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/AttachedWithKeyAccount-Detach.png
[14]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/get-shared-access-signature-for-storage-explorer.png
[15]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/create-shared-access-signature-for-storage-explorer.png
[16]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog-WithConnStringOrSASSelected.png
[17]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog-ConnStringOrSASPage-1.png
[18]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/AttachedWithSASAccount.png
[19]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog-ConnStringOrSASPage-2.png
[20]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ServiceAttachedWithSAS.png
[21]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/connect-to-cosmos-db-by-connection-string.png
[22]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/connection-string-for-cosmos-db.png
[23]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/storage-explorer-search-for-resource.png
