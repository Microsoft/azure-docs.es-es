---
title: Configurar el almacenamiento de Avere vFXT - Azure
description: Obtenga información sobre cómo agregar un sistema de almacenamiento de back-end para un clúster en Avere vFXT for Azure. Si creó un contenedor de blobs de Azure con el clúster, está listo para usarse.
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 01/13/2020
ms.author: rohogue
ms.openlocfilehash: c17d3c7cd2cf6fe5bca725cf94344b2d2cb50bf2
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88271148"
---
# <a name="configure-storage"></a>Configuración del almacenamiento

En este paso podrá configurar un sistema de almacenamiento de back-end para el clúster de vFXT.

> [!TIP]
> Si creó un contenedor de blobs de Azure junto con el clúster de Avere vFXT, ese contenedor ya está configurado y listo para ser utilizado.

Siga estas instrucciones si no creó un contenedor de blobs con el clúster o si quiere agregar un hardware adicional o un sistema de almacenamiento basado en la nube.

Hay dos tareas principales que debe realizar:

1. [Crear un archivador principal](#create-a-core-filer) para conectar el clúster de vFXT a un sistema de almacenamiento existente o a un contenedor de Azure Storage.

1. [Crear una unión de espacio de nombres](#create-a-junction) para definir la ruta de acceso que usarán los clientes.

Tenga en cuenta que en estos pasos usará el panel de control de Avere. Lea [Access the vFXT cluster](avere-vfxt-cluster-gui.md) (Obtener acceso al clúster de vFXT) para aprender a usarlo.

## <a name="create-a-core-filer"></a>Crear un archivador principal

"Archivo principal" es un término de vFXT que hace referencia a un sistema de almacenamiento de back-end. El almacenamiento puede ser un dispositivo NAS de hardware como NetApp o Isilon, o puede ser un almacén de objetos en la nube. Puede encontrar más información sobre los archivadores principales en la [guía de configuración de clústeres de Avere](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/settings_overview.html#managing-core-filers).

Para agregar un archivador principal, elija uno de los dos tipos principales que tiene disponibles:

* [Archivador principal de NAS](#nas-core-filer): describe cómo agregar un archivador principal de NAS.
* [Archivador principal en la nube de Azure Storage](#azure-blob-storage-cloud-core-filer): describe cómo agregar un contenedor de Azure Blob Storage como archivador principal en la nube.

### <a name="nas-core-filer"></a>Archivador principal de NAS

Un archivador principal de NAS puede ser un dispositivo local de NetApp o Isilon o un punto de conexión de NAS en la nube. El sistema de almacenamiento debe tener una conexión confiable de alta velocidad al clúster de Avere vFXT; por ejemplo, una conexión de ExpressRoute de 1 Gbps (no una VPN). Asimismo, debe proporcionar acceso a la raíz del clúster a las exportaciones de NAS que se vayan a usar.

Siga estos pasos para agregar un archivador principal de NAS:

1. En el panel de control de Avere, haga clic en la ficha **Settings** (Configuración) que se encuentra en la parte superior.

1. Haga clic en **Core Filer (Archivador principal)**  > **Manage Core Filers (Administrar archivadores principales)** a la izquierda.

1. Haga clic en **Crear**.

   ![Captura de pantalla de la página para agregar un nuevo archivador principal con un cursor sobre el botón Crear](media/avere-vfxt-add-core-filer-start.png)

1. Rellene la información necesaria en el asistente:

   * Póngale nombre a su archivador principal.
   * Proporcione el nombre de dominio completo (FQDN) si estuviera disponible. En caso contrario, proporcione una dirección IP o el nombre de host que se resuelve en su archivador principal.
   * Elija la clase de archivador de la lista. Si no está seguro, elija **Other** (Otro).

     ![Captura de pantalla de la página para agregar un nuevo archivador principal con el nombre del archivador principal y su nombre de dominio completo](media/avere-vfxt-add-core-filer.png)
  
   * A continuación, haga clic en **Next** (Siguiente) y elija una directiva de caché.
   * Haga clic en **Add filer** (Agregar archivador).
   * Para obtener más información, consulte [Adding a new NAS core filer](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/new_core_filer_nas.html) (Agregar un nuevo archivador principal de NAS) en la guía de configuración del clúster de Avere.

A continuación, [cree una unión](#create-a-junction).  

### <a name="azure-blob-storage-cloud-core-filer"></a>Archivador principal en la nube de Azure Blob Storage

Para poder usar Azure Blob Storage como almacenamiento de back-end en el clúster de vFXT, necesita un contenedor vacío que pueda agregar como archivo central.

Para poder agregar Blob Storage al clúster debe realizar estas tareas:

* Crear una cuenta de almacenamiento (paso 1, que se detalla a continuación).
* Crear un contenedor de blobs vacío (pasos 2 y 3).
* Agregar la clave de acceso de almacenamiento como una credencial en la nube para el clúster de vFXT (pasos 4 a 6).
* Agregar el contenedor de blobs como un archivador principal para el clúster de vFXT (pasos 7 a 9).
* Crear una unión de espacio de nombres que los clientes usarán para montar el archivador principal (Sección [Crear una unión](#create-a-junction), que es la misma tanto para el almacenamiento de hardware como para el almacenamiento en la nube).

> [!TIP]
> Si crea un contenedor de blobs al crear un clúster de Avere vFXT for Azure, la plantilla de implementación configurará automáticamente el contenedor como archivador principal. (Esto también se aplica si usa el script de creación, que está disponible a petición). No es necesario configurar después el archivador principal.
>
> La herramienta de creación de clústeres realiza automáticamente estas tareas de configuración:
>
> * Crea un contenedor de blobs en la cuenta de almacenamiento especificada
> * Define el contenedor como archivador principal
> * Crea una unión de espacio de nombres con el contenedor
> * Crea un punto de conexión de servicio de almacenamiento dentro de la red virtual del clúster

Para agregar Blob Storage después de crear el clúster, siga estos pasos.

1. Cree una cuenta de almacenamiento V2 de uso general mediante la siguiente configuración:

   * **Suscripción**: la misma que usó para el clúster de vFXT.
   * **Grupo de recursos**: el mismo que usó para el grupo de clústeres de vFXT (opcional).
   * **Ubicación**: la misma que usó para el clúster de vFXT.
   * **Rendimiento**: estándar (no se admite el almacenamiento Premium).
   * **Tipo de cuenta**: uso general V2 (StorageV2).
   * **Replicación**: almacenamiento con redundancia local (LRS).
   * **Nivel de acceso**: frecuente.
   * **Es necesaria una transferencia segura**: deshabilite esta opción (valor no predeterminado).
   * **Redes virtuales** : no requeridas.

   Puede usar Azure Portal o hacer clic en el botón "Implementar en Azure".

   [![botón para crear la cuenta de almacenamiento](media/deploytoazure.png)](https://ms.portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAvere%2Fmaster%2Fsrc%2Fvfxt%2Fstorageaccount%2Fazuredeploy.json)

1. Una vez creada la cuenta, vaya a la página de la cuenta de almacenamiento.

   ![Nueva cuenta de almacenamiento en Azure Portal](media/avere-vfxt-new-storage-acct.png)

1. Cree un contenedor de blobs: Haga clic en **Contenedores** en la página información general y luego haga clic en **+Contenedor**. Use cualquier nombre de contenedor y asegúrese de que el acceso está establecido como **Privado**.

   ![Página Blobs de almacenamiento con el botón +Contenedor rodeado por un recuadro y creación de un contenedor en una página emergente](media/avere-vfxt-new-blob.png)

1. Obtenga la clave de cuenta de Azure Storage; para ello, haga clic en la opción **Claves de acceso** en **Configuración**. Copie una de las claves proporcionadas.

   ![GUI de Azure Portal para copiar la clave](media/avere-vfxt-copy-storage-key.png)

1. Abra el panel de control de Avere para el clúster. Haga clic en **Settings** (Configuración) y abra la opción **Cluster (Clúster)** > **Cloud Credentials (Credenciales en la nube)** en el panel de navegación izquierdo. En la página de credenciales en la nube, haga clic en **Add Credential** (Agregar credencial).

   ![Haga clic en el botón Add Credential (Agregar credencial); este se encuentra en la página de configuración de credenciales en la nube](media/avere-vfxt-new-credential-button.png)

1. Rellene la información siguiente para crear una credencial para el archivador principal en la nube:

   | Campo | Value |
   | --- | --- |
   | Nombre de credencial | cualquier nombre descriptivo |
   | Tipo de servicio | (seleccione la clave de acceso de Azure Storage) |
   | Inquilino | el nombre de cuenta de almacenamiento |
   | Subscription | Id. de suscripción |
   | Clave de acceso de almacenamiento | Clave de cuenta de Azure Storage (copiado en el paso anterior) |

   Haga clic en **Enviar**.

   ![Se completó el formulario de credenciales en la nube en el panel de control de Avere](media/avere-vfxt-new-credential-submit.png)

1. A continuación, cree el archivador principal. En el lado izquierdo del panel de control Avere, haga clic en **Core Filer (Archivador principal)**  >  **Manage Core Filers (Administrar archivadores principales)** .

1. Haga clic en el botón **Create** (Crear) situado en la página de configuración **Manage Core Filers** (Administrar archivadores principales).

1. Complete los pasos del asistente:

   * Seleccione la **nube** del tipo de archivador.
   * Póngale nombre al nuevo archivador principal y haga clic en **Next** (Siguiente).
   * Acepte la directiva de caché predeterminada y vaya a la tercera página.
   * En **Service type** (Tipo de servicio), elija **Azure Storage**.
   * Elija la credencial que creó anteriormente.
   * Establezca el **contenido de cubo** como **Empty** (Vacío).
   * Cambie la**comprobación del certificado** a **Disabled** (Deshabilitado).
   * Cambie el **modo de compresión** a **None** (Ninguno).
   * Haga clic en **Next**.
   * En la cuarta página, escriba el nombre del contenedor en **Bucket name (Nombre de cubo)** como *storage_account_name*/*container_name*.
   * Además, puede establecer el **tipo de cifrado** a **None** (Ninguno).  Azure Storage se cifra de forma predeterminada.
   * Haga clic en **Add filer** (Agregar archivador).

   Para obtener más información, consulte [Adding a new cloud core filer](<https://azure.github.io/Avere/legacy/ops_guide/4_7/html/new_core_filer_cloud.html>) (Agregar un nuevo archivador principal en la nube) en la guía de configuración del clúster de Avere.

La página se actualizará (o puede actualizarla usted mismo) para mostrar el nuevo archivador principal.

A continuación, debe [crear una unión](#create-a-junction).

## <a name="create-a-junction"></a>Crear una unión

Una unión es una ruta de acceso que se crea para los clientes. Los clientes deben usar la ruta de acceso para llegar al destino que usted elija.

Por ejemplo, puede crear `/vfxt/files` para asignarlo a la exportación del archivador principal de NetApp `/vol0/data` y al subdirectorio `/project/resources`.

Puede encontrar más información sobre las uniones en la [sección del espacio de nombres de la guía de configuración del clúster de Avere](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_namespace.html).

Siga estos pasos en la interfaz del panel de control de Avere:

* Haga clic en **VServer** > **Namespace (Espacio de nombres)** en la parte superior izquierda.
* Debe iniciar la ruta de acceso del espacio de nombres con una barra diagonal (/), como ``/vfxt/data``.
* Elija su archivador principal.
* Elija la exportación del archivador principal.
* Haga clic en **Next**.

  ![Captura de pantalla de la página "Agregar nueva unión" con los campos completados para la unión, el archivador principal y la exportación](media/avere-vfxt-add-junction.png)

La unión aparecerá pasados unos segundos. Recuerde que puede crear uniones adicionales si fuera necesario.

Una vez creada la Unión, los clientes utilizan la ruta de acceso del espacio de nombres para acceder a los archivos desde el sistema de almacenamiento.

## <a name="next-steps"></a>Pasos siguientes

* [Montaje del clúster de Avere de vFXT](avere-vfxt-mount-clients.md)
* Más información sobre formas eficaces de [mover datos a un nuevo contenedor de blobs](avere-vfxt-data-ingest.md)
