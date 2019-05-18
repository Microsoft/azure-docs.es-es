---
title: Guía de seguridad de Azure Storage | Microsoft Docs
description: Detalles de los distintos métodos de seguridad de Azure Storage incluidos, entre otros, el control de acceso basado en rol, el cifrado del servicio Storage, el cifrado del lado cliente, SMB 3.0 y Azure Disk Encryption.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 03/21/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 3d5bfa2426d58fa5a09d2203272536eec7fa9c55
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65789952"
---
# <a name="azure-storage-security-guide"></a>Guía de seguridad de Azure Storage

Azure Storage proporciona un completo conjunto de funcionalidades de seguridad que, conjuntamente, permiten a los desarrolladores compilar aplicaciones seguras:

- Todos los datos (incluidos los metadatos) escritos en Azure Storage se cifran automáticamente con [Storage Service Encryption (SSE)](storage-service-encryption.md). Para obtener más información, consulte [Announcing Default Encryption for Azure Blobs, Files, Table and Queue Storage](https://azure.microsoft.com/blog/announcing-default-encryption-for-azure-blobs-files-table-and-queue-storage/) (Presentación del cifrado predeterminado de Azure Blobs, Files, Table y Queue Storage).
- Azure Active Directory (Azure AD) y el control de acceso basado en rol (RBAC) son compatibles con Azure Storage para las operaciones de administración de recursos y las operaciones de datos, como se indica a continuación:   
    - Puede asignar roles de RBAC en el ámbito de la cuenta de almacenamiento para las entidades de seguridad y utilizar Azure AD para autorizar las operaciones de administración de recursos, como la administración de claves.
    - Integración de Azure AD se admite para las operaciones de datos de blob y cola. Puede asignar roles de RBAC en el ámbito de una suscripción, un grupo de recursos, una cuenta de almacenamiento, un contenedor individual o una cola a una entidad de seguridad o identidad administrada para los recursos de Azure. Para obtener más información, consulte [Autenticación del acceso a Azure Storage con Azure Active Directory](storage-auth-aad.md).   
- Los datos se pueden proteger en tránsito entre una aplicación y Azure usando [cifrado de cliente](../storage-client-side-encryption.md), HTTPS o SMB 3.0.  
- Se puede establecer el cifrado de los discos de datos y del sistema operativo utilizados por Azure Virtual Machines mediante [Azure Disk Encryption](../../security/azure-security-disk-encryption.md). 
- Se puede conceder acceso delegado a los objetos de datos de Azure Storage mediante las [Firmas de acceso compartido](../storage-dotnet-shared-access-signature-part-1.md).

Este artículo ofrece una visión general de cada una de estas características de seguridad que se pueden usar con Azure Storage. Se incluyen vínculos a artículos en los que se detalla cada una de ellas, así que puede examinar un tema determinado con mayor detalle si así lo desea.

Estos son los temas que se tratarán en este artículo:

* [Seguridad en el plano de la administración](#management-plane-security): protección de la cuenta de almacenamiento

  El plano de la administración está compuesto por los recursos que se usan para administrar la cuenta de almacenamiento. En esta sección se habla sobre el modelo de implementación de Azure Resource Manager y cómo usar el control de acceso basado en roles (RBAC) para controlar el acceso a las cuentas de almacenamiento. También se explica cómo administrar las claves de cuenta de almacenamiento y cómo volver a generarlas.
* [Seguridad en el plano de los datos](#data-plane-security) : protección del acceso a los datos

  En esta sección, analizaremos el modo de permitir el acceso a los objetos de datos reales de la cuenta de almacenamiento, como blobs, archivos, colas y tablas, y el uso de Firmas de acceso compartido y Directivas de acceso almacenadas. Nos fijaremos en las Firmas de acceso compartido tanto a nivel de servicio como a nivel de cuenta. También veremos cómo limitar el acceso a una dirección IP específica (o un intervalo de direcciones IP), cómo limitar el protocolo utilizado para HTTPS y cómo revocar una Firma de acceso compartido sin esperar a que expire.
* [Cifrado en tránsito](#encryption-in-transit)

  En esta sección se describe cómo proteger los datos cuando se transfieren a o desde Azure Storage. Hablaremos sobre el uso recomendado de HTTPS y el cifrado que usa SMB 3.0 para los recursos compartidos de archivos de Azure. También echaremos un vistazo al Cifrado de cliente, que permite cifrar los datos antes de transferirlos a Storage en una aplicación cliente y descifrarlos una vez transferidos desde este servicio.
* [Cifrado en reposo](#encryption-at-rest)

  Se habla sobre el cifrado del servicio Storage (SSE), que se habilita ahora automáticamente para las cuentas de almacenamiento nuevas y existentes. También veremos cómo puede usar Azure Disk Encryption y exploraremos los casos y las diferencias básicas entre Disk Encryption, SSE y el cifrado del lado cliente. Asimismo, estudiaremos brevemente el cumplimiento de la norma FIPS para equipos del Gobierno de EE. UU.
* Uso de [Storage Analytics](#storage-analytics) para auditar el acceso de Azure Storage

  En esta sección se describe cómo buscar información en los registros de Storage Analytics para una solicitud. Echaremos un vistazo a datos de registro reales de Storage Analytics y aprenderemos a discernir si una solicitud se realiza con una clave de cuenta de Storage, una Firma de acceso compartido o de manera anónima, y si se pudo completar o no.
* [Habilitación de clientes basados en explorador mediante el uso compartido de recursos entre orígenes](#cross-origin-resource-sharing-cors)

  En esta sección se analiza cómo permitir el uso compartido de recursos entre orígenes. Abordaremos el acceso entre dominios y cómo administrarlo con las funcionalidades del uso compartido de recursos entre orígenes integradas en Azure Storage.

## <a name="management-plane-security"></a>Seguridad en el plano de la administración
El plano de la administración consta de las operaciones que afectan a la propia cuenta de almacenamiento. Por ejemplo, puede crear o eliminar una cuenta de almacenamiento, obtener una lista de cuentas de almacenamiento en una suscripción, recuperar las claves de cuenta de almacenamiento o volver a generar las claves de cuenta de almacenamiento.

Cuando se crea una nueva cuenta de almacenamiento, se selecciona un modelo de implementación clásico o de Resource Manager. El modelo clásico de creación de recursos de Azure únicamente permite acceder a toda la suscripción en su conjunto o bien a nada de ella, y lo mismo ocurre con la cuenta de almacenamiento.

Esta guía se centra en el modelo de Resource Manager, que es el medio recomendado para crear cuentas de almacenamiento. Con las cuentas de almacenamiento de Resource Manager, en lugar de proporcionar acceso a toda la suscripción, puede controlar el acceso en un nivel más detallado en el plano de administración mediante el control de acceso basado en rol (RBAC).

### <a name="how-to-secure-your-storage-account-with-role-based-access-control-rbac"></a>Protección de su cuenta de almacenamiento con el control de acceso basado en rol (RBAC)
Vamos a ver qué es el control de acceso basado en roles y cómo se utiliza. Cada una de las suscripciones de Azure está asociada a una instancia de Azure Active Directory. Es posible conceder acceso a los usuarios, los grupos y las aplicaciones desde ese directorio a la administración de recursos de la suscripción de Azure que usan el modelo de implementación de Resource Manager. A este tipo de seguridad se le conoce como control de acceso basado en rol (RBAC). Para administrar este acceso, puede utilizar [Azure Portal](https://portal.azure.com/), [las herramientas de la CLI de Azure](../../cli-install-nodejs.md), [PowerShell](/powershell/azureps-cmdlets-docs) o [las API de REST del proveedor de recursos de Azure Storage](https://msdn.microsoft.com/library/azure/mt163683.aspx).

Con el modelo de Resource Manager, se coloca la cuenta de almacenamiento en un grupo de recursos y se controla el acceso al plano de la administración de dicha cuenta de almacenamiento específica mediante Azure Active Directory. Por ejemplo, puede proporcionar a usuarios específicos la posibilidad de tener acceso a las claves de cuenta de almacenamiento, mientras que otros usuarios pueden ver información sobre la cuenta de almacenamiento pero no pueden tener acceso a sus claves.

#### <a name="granting-access"></a>Concesión de acceso
El acceso se concede mediante la asignación de rol RBAC adecuado a los usuarios, grupos y aplicaciones en el ámbito correcto. Para conceder acceso a toda la suscripción, asigne un rol a nivel de suscripción. Puede conceder acceso a todos los recursos de un grupo de recursos mediante la concesión de permisos al propio grupo de recursos. También puede asignar roles específicos a recursos específicos, como cuentas de almacenamiento.

Estos son los puntos principales que necesita saber acerca del uso del control de acceso basado en rol para tener acceso a las operaciones de administración de una cuenta de Azure Storage:

* Al asignar el acceso, básicamente asigna un rol a la cuenta a la que desea tener acceso. Puede controlar el acceso a las operaciones que se utilizan para administrar esa cuenta de almacenamiento, pero no a los objetos de datos de la cuenta. Por ejemplo, puede conceder permiso para recuperar las propiedades de la cuenta de almacenamiento (por ejemplo, redundancia), pero no a un contenedor o los datos dentro de un contenedor de Blob Storage.
* Para que otro usuario pueda tener acceso a los objetos de datos de la cuenta de almacenamiento, puede concederle permiso para leer las claves de cuenta de almacenamiento. De este modo, ese usuario podrá usar esas claves para tener acceso a blobs, colas, tablas y archivos.
* Los roles pueden asignarse específicamente a una cuenta de usuario, un grupo de usuarios o una aplicación.
* Cada rol tiene una lista de acciones y no acciones. Por ejemplo, el rol de colaborador de la máquina virtual tiene una acción "listKeys" que permite leer las claves de cuenta de almacenamiento. El colaborador tiene "no acciones", como actualizar el acceso de los usuarios en Active Directory.
* Entre los roles para el almacenamiento se incluyen, entre otros, los siguientes:

  * Propietario: puede administrar todo, incluido el acceso.
  * Colaborador: puede hacer lo mismo que el propietario, salvo asignar acceso. Un usuario con este rol puede ver y volver a generar las claves de cuenta de almacenamiento. Con las claves de cuenta de almacenamiento, puede tener acceso a los objetos de datos.
  * Lector: puede ver información sobre la cuenta de almacenamiento, excepto los secretos. Por ejemplo, si asigna un rol con permisos de lectura sobre la cuenta de almacenamiento a otro usuario, este puede ver las propiedades de la cuenta de almacenamiento, pero no puede realizar cambios en las propiedades ni ver las claves de cuenta de almacenamiento.
  * Colaborador de la cuenta de almacenamiento: puede administrar la cuenta de almacenamiento, leer los grupos de recursos y los recursos de la suscripción, y crear y administrar implementaciones de los grupos de recursos de la suscripción. También puede acceder a las claves de cuenta de almacenamiento, lo que a su vez implica que puede acceder al plano de los datos.
  * Administrador de acceso de usuarios: puede administrar el acceso de los usuarios a la cuenta de almacenamiento. Por ejemplo, puede conceder acceso de lectura a un usuario específico.
  * Colaborador de la máquina virtual: puede administrar máquinas virtuales, pero no la cuenta de almacenamiento o la red virtual a la que están conectadas. Este rol puede listar las claves de cuenta de almacenamiento, lo que significa que el usuario a quien asigne este rol puede actualizar el plano de los datos.

    Para que un usuario pueda crear una máquina virtual, tiene que poder crear el archivo VHD correspondiente en una cuenta de almacenamiento. Para ello, es necesario que pueda recuperar la clave de la cuenta de almacenamiento y pasarla a la API que está creando la máquina virtual. Por lo tanto, es necesario que disponga de este permiso para listar las claves de cuenta de almacenamiento.
* La posibilidad de definir roles personalizados es una característica que le permite crear un conjunto de acciones desde una lista de acciones disponibles que se puede llevar a cabo sobre recursos de Azure.
* Es necesario que el usuario se haya configurado en Azure Active Directory para poder asignarle un rol.
* Puede crear un informe de quién concedió/revocó qué tipo de acceso a quién y en qué ámbito con PowerShell o la CLI de Azure.

#### <a name="resources"></a>Recursos
* [Control de acceso basado en roles de Azure Active Directory](../../role-based-access-control/role-assignments-portal.md)

  En este artículo se explica el control de acceso basado en rol de Azure Active Directory y su funcionamiento.
* [RBAC: Roles integrados](../../role-based-access-control/built-in-roles.md)

  En este artículo se detalla todas las funciones integradas disponibles en el control de acceso basado en rol.
* [Descripción de la implementación de Resource Manager y la implementación clásica](../../azure-resource-manager/resource-manager-deployment-model.md)

  En este artículo se explican los modelos de implementación clásica y de Resource Manager, y se detallan las ventajas de utilizar Resource Manager y los grupos de recursos. Se explica cómo funcionan los proveedores de Azure Compute, Network y Storage en el modelo de Resource Manager.
* [Administración del control de acceso basado en rol con la API de REST](../../role-based-access-control/role-assignments-rest.md)

  En este artículo se muestra cómo utilizar la API de REST para administrar el control de acceso basado en rol.
* [Referencia de API de REST de proveedor de recursos de Azure Storage](https://msdn.microsoft.com/library/azure/mt163683.aspx)

  Esta referencia de API describe las API que puede usar para administrar la cuenta de almacenamiento mediante programación.
* [Uso de la API de autenticación de Resource Manager para acceder a suscripciones](../../azure-resource-manager/resource-manager-api-authentication.md)

  En este artículo se muestra cómo realizar la autenticación mediante las API de Resource Manager.
* [Role-Based Access Control for Microsoft Azure from Ignite (Control de acceso basado en rol para Microsoft Azure de Ignite)](https://channel9.msdn.com/events/Ignite/2015/BRK2707)

  Este es un vínculo a un vídeo de Channel 9 de la conferencia de MS Ignite de 2015. En esta sesión, se abordan las funciones de notificación y administración de acceso en Azure y se exploran los procedimientos recomendados en torno a la protección del acceso a suscripciones de Azure con Azure Active Directory.

### <a name="managing-your-storage-account-keys"></a>Administración de las claves de cuenta de almacenamiento
Las claves de la cuenta de almacenamiento son cadenas de 512 bits creadas por Azure que, junto con el nombre de la cuenta de almacenamiento, pueden usarse para acceder a los objetos de datos almacenados en la cuenta de almacenamiento; por ejemplo, blobs, entidades dentro de una tabla, mensajes de una cola y archivos en un recurso compartido de archivos de Azure. El control del acceso a las claves de cuenta de almacenamiento controla el acceso al plano de los datos de esa cuenta de almacenamiento.

Cada cuenta de almacenamiento tiene dos claves que se conocen como "Key 1" y "Key 2" en [Azure Portal](https://portal.azure.com/) y en los cmdlets de PowerShell. Es posible volver a generarlas manualmente con diferentes métodos, como el uso de [Azure Portal](https://portal.azure.com/), PowerShell o la CLI de Azure, o mediante programación con la biblioteca de clientes de Storage de .NET o la API de REST de los servicios de Azure Storage.

Hay varias razones que justifican volver a generar las claves de cuenta de almacenamiento.

* Puede cambiarlas de nuevo de manera periódica por motivos de seguridad.
* Podría volver a generar las claves de cuenta de almacenamiento si alguien consiguiera atacar una aplicación y recuperar la clave que se codificó de forma rígida o se guardó en un archivo de configuración, con lo que tendría acceso completo a la cuenta de almacenamiento.
* También podría volver a generar la clave si el equipo utilizara una aplicación de explorador de Storage que conserve la clave de la cuenta de almacenamiento y uno de los miembros abandonara el equipo. La aplicación seguiría funcionando y permitiría a ese antiguo miembro acceder a la cuenta de almacenamiento cuando ya no forme parte del equipo. Este es realmente el motivo principal por el que se crean Firmas de acceso compartido de nivel de cuenta, ya que al utilizarlas no es necesario almacenar las claves de acceso en un archivo de configuración.

#### <a name="key-regeneration-plan"></a>Plan de regeneración de claves
No es buena idea limitarse a volver a generar la clave que está utilizando sin planificación. Si no lo hace así, podría cortar todo el acceso a esa cuenta de almacenamiento, lo que podría provocar una interrupción importante. Este es el motivo por el cual hay dos claves. Puede volver a generar una de ellas y luego la otra en vez de hacerlo de manera simultánea.

Antes de volver a generar las claves, asegúrese de que dispone de una lista de todas las aplicaciones que dependen de la cuenta de almacenamiento, así como cualquier otro servicio que se use en Azure. Por ejemplo, si usa instancias de Azure Media Services que dependen de su cuenta de almacenamiento, debe volver a sincronizar las claves de acceso con los servicios multimedia después de volver a generar las claves. Si está utilizando otras aplicaciones, como un explorador de Storage, debe proporcionarles también las nuevas claves. Si tiene máquinas virtuales cuyos archivos VHD se almacenan en la cuenta de almacenamiento, estos no se verán afectados por la regeneración de las claves de cuenta de almacenamiento.

Puede volver a generar las claves en Azure Portal. Una vez que se vuelven a generar, las claves pueden tardar hasta diez minutos en sincronizarse en todos los servicios de Storage.

Cuando esté listo, este es el proceso general que detalla cómo se debe cambiar la clave. En este caso, suponemos que actualmente usa la Clave 1 y que va a cambiar todo para usar la Clave 2 en su lugar.

1. Vuelva a generar la Clave 2 para asegurarse de que sea segura. Puede hacerlo en Azure Portal.
2. En todas las aplicaciones donde se almacena la clave de almacenamiento, cámbiela para usar el nuevo valor de Key 2. Pruebe y publique la aplicación.
3. Una vez que todas las aplicaciones y los servicios estén activos y se estén ejecutando correctamente, vuelva a generar la Clave 1. De este modo se asegura de que nadie a quien no haya proporcionado expresamente la nueva clave seguirá teniendo acceso a la cuenta de almacenamiento.

Si actualmente utiliza la Clave 2, siga este mismo proceso pero invierta los nombres de las claves.

Puede migrar en un par de días, cambiando cada aplicación para que use la nueva clave y publicándola. Cuando lo haya hecho con todas, debe volver y generar de nuevo la clave antigua para que deje de funcionar.

Otra opción consiste en poner la clave de la cuenta de almacenamiento en un [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) como un secreto y hacer que las aplicaciones recuperen la clave desde esa ubicación. Así, cuando vuelva a generar la clave y actualice Azure Key Vault, no será necesario volver a implementar las aplicaciones porque tomarán la nueva clave desde dicho almacén automáticamente. Tenga en cuenta que puede hacer que la aplicación lea la clave cada vez que la necesite, o bien puede almacenarla en la memoria caché y, si se produce un error al usarla, se recuperaría de nuevo desde Azure Key Vault.

El uso de Azure Key Vault también agrega otro nivel de seguridad a las claves de almacenamiento. Si utiliza este método, nunca tendrá la clave de almacenamiento codificada de manera rígida en un archivo de configuración, lo que impide que alguien obtenga acceso a las claves sin permiso específico.

Otra ventaja de usar Azure Key Vault es que también puede controlar el acceso a las claves con Azure Active Directory. Esto significa que puede conceder acceso a la serie de aplicaciones que necesitan recuperar las claves de Azure Key Vault, y sabe que otras aplicaciones no podrán tener acceso a las claves sin concederles el permiso específicamente.

> [!NOTE]
> Microsoft recomienda usar solo una de las claves en todas las aplicaciones al mismo tiempo. Si utiliza la Clave 1 en algunos lugares y la Clave 2 en otros, no podrá rotar las claves sin que alguna aplicación pierda el acceso.

#### <a name="resources"></a>Recursos

* [Administración de la configuración de cuentas de almacenamiento en Azure Portal](storage-account-manage.md)
* [Referencia de API de REST de proveedor de recursos de Azure Storage](https://msdn.microsoft.com/library/mt163683.aspx)

## <a name="data-plane-security"></a>Seguridad en el plano de los datos
La seguridad en el plano de los datos hace referencia a los métodos utilizados para proteger los objetos de datos almacenados en Azure Storage: blobs, colas, tablas y archivos. Hemos visto métodos para cifrar los datos y la seguridad durante el tránsito de los datos, pero ¿qué hay sobre controlar el acceso a los objetos?

Tiene tres opciones para autorizar el acceso a los objetos de datos en Azure Storage, que incluyen:

- Uso de Azure AD para autorizar el acceso a los contenedores y las colas. Azure AD proporciona ventajas sobre otros enfoques para la autorización, incluida la eliminación de la necesidad de almacenar secretos en el código. Para obtener más información, consulte [Autenticación del acceso a Azure Storage con Azure Active Directory](storage-auth-aad.md). 
- Uso de las claves de la cuenta de almacenamiento para autorizar el acceso mediante una clave compartida. La autorización mediante una clave compartida requiere almacenar las claves de la cuenta de almacenamiento en la aplicación, por lo que Microsoft recomienda usar Azure AD en su lugar, siempre que sea posible.
- Uso de firmas de acceso compartido para conceder permisos controlados para objetos de datos específicos por un período de tiempo determinado.

Además, para Blog Storage, puede permitir el acceso público a los blobs estableciendo el nivel de acceso para el contenedor que almacena los blobs según corresponda. Si configura el acceso para un contenedor en Blob o Contenedor, permitirá el acceso de lectura público a los blobs en ese contenedor. Esto significa que cualquier usuario que tenga una dirección URL que apunte a un blob de ese contenedor podrá abrirlo en un explorador sin necesidad de una Firma de acceso compartido o las claves de cuenta de almacenamiento.

Además de limitar el acceso mediante la autorización, puede usar [firewalls y redes virtuales](storage-network-security.md) para limitar el acceso a la cuenta de almacenamiento basado en reglas de red.  Este método permite denegar el acceso al tráfico de Internet público y conceder acceso a determinadas redes virtuales de Azure o determinados intervalos de direcciones IP de Internet públicas.

### <a name="storage-account-keys"></a>Claves de cuenta de almacenamiento
Las claves de cuenta de almacenamiento son cadenas de 512 bits creadas por Azure que, junto con el nombre de la cuenta de almacenamiento, pueden utilizarse para tener acceso a los objetos de datos almacenados en la cuenta de almacenamiento.

Por ejemplo, puede leer blobs, escribir en colas, crear tablas y modificar los archivos. Muchas de estas acciones pueden realizarse a través de Azure Portal, o usando una de las muchas aplicaciones del Explorador de Storage. También puede escribir código para usar la API de REST o una de las bibliotecas de cliente de Storage para realizar estas operaciones.

Como se describe en la sección sobre la [Seguridad en el plano de la administración](#management-plane-security), el acceso a las claves de almacenamiento de una cuenta de almacenamiento clásica se puede conceder proporcionando acceso completo a la suscripción de Azure. El acceso a las claves de almacenamiento para una cuenta de almacenamiento mediante el modelo de Azure Resource Manager puede controlarse mediante el control de acceso basado en rol.

### <a name="how-to-delegate-access-to-objects-in-your-account-using-shared-access-signatures-and-stored-access-policies"></a>Delegación del acceso a objetos en la cuenta mediante Firmas de acceso compartido y Directivas de acceso almacenadas
Una Firma de acceso compartido es una cadena que contiene un token de seguridad que puede asociarse a un URI que le permite delegar el acceso a objetos de almacenamiento y especificar restricciones, como permisos o un intervalo de fecha y hora para el acceso.

Puede conceder acceso a blobs, contenedores, mensajes de colas, archivos y tablas. En el caso de las tablas, realmente puede conceder permiso para tener acceso a un intervalo de entidades en la tabla especificando los intervalos de clave de fila y partición a los que desea que el usuario tenga acceso. Por ejemplo, si tiene datos almacenados con una clave de partición por estados geográficos, podría dar a una persona acceso a los datos de California.

En otro ejemplo, podría proporcionar un token de Firma de acceso compartido a una aplicación web que le permitiera escribir entradas en una cola, y proporcionar otro token de Firma de acceso compartido a una aplicación de rol de trabajo para que obtenga mensajes de la cola y los procese. O bien, podría dar a un cliente un token de Firma de acceso compartido que pueda utilizar para cargar imágenes a un contenedor en Blob Storage y conceder permiso a una aplicación web para que lea esas imágenes. En ambos casos, se trata de procesos independientes: es posible dar a cada aplicación solo el acceso que requiere para realizar sus tareas. Esto es posible mediante el uso de Firmas de acceso compartido.

#### <a name="why-you-want-to-use-shared-access-signatures"></a>Razones para usar las Firmas de acceso compartido
¿Por qué desearía utilizar una Firma de acceso compartido en lugar de simplemente proporcionar la clave de la cuenta de almacenamiento, que es mucho más fácil? Si comparte la clave de su cuenta de almacenamiento es como si prestara a alguien las llaves de su reino de almacenamiento. Con ella, es posible acceder a todo el contenido. Cualquiera podría usar sus claves y cargar toda su biblioteca de música en la cuenta de almacenamiento. También podría reemplazar sus archivos por versiones infectadas o robar sus datos. Conceder acceso ilimitado a su cuenta de almacenamiento es algo que no debe hacerse a la ligera.

Con las Firmas de acceso compartido, es posible conceder a un cliente únicamente los permisos que necesite durante un período de tiempo limitado. Por ejemplo, si alguien está cargando un blob a su cuenta, puede concederle acceso de escritura durante un tiempo suficiente que le permita cargar el blob (dependerá del tamaño del blob, por supuesto). Y si cambia de opinión, puede revocar el acceso.

Además, puede especificar que las solicitudes realizadas mediante una Firma de acceso compartido se limiten a una cierta dirección IP o un intervalo de direcciones IP externas a Azure. También puede requerir que las solicitudes se realicen mediante un protocolo específico (HTTPS o HTTP/HTTPS). Esto significa que si solo desea permitir el tráfico HTTPS, puede establecer que el protocolo requerido sea solo HTTPS y se bloqueará el tráfico HTTP.

#### <a name="definition-of-a-shared-access-signature"></a>Definición de una Firma de acceso compartido
Una Firma de acceso compartido es un conjunto de parámetros de consulta anexo a la dirección URL que señala al recurso

que proporciona información sobre el acceso permitido y el intervalo de tiempo durante el que se permite el acceso. Aquí presentamos un ejemplo: este URI proporciona acceso de lectura a un blob durante cinco minutos. Tenga en cuenta que los parámetros de consulta de la Firma de acceso compartido deben presentar codificación URL; por ejemplo %3A para los dos puntos (:) o %20 para un espacio.

```
http://mystorage.blob.core.windows.net/mycontainer/myblob.txt (URL to the blob)
?sv=2015-04-05 (storage service version)
&st=2015-12-10T22%3A18%3A26Z (start time, in UTC time and URL encoded)
&se=2015-12-10T22%3A23%3A26Z (end time, in UTC time and URL encoded)
&sr=b (resource is a blob)
&sp=r (read access)
&sip=168.1.5.60-168.1.5.70 (requests can only come from this range of IP addresses)
&spr=https (only allow HTTPS requests)
&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D (signature used for the authentication of the SAS)
```

#### <a name="how-the-shared-access-signature-is-authorized-by-the-azure-storage-service"></a>Autorización de la Firma de acceso compartido por el servicio Azure Storage
Cuando el servicio de almacenamiento recibe la solicitud, toma los parámetros de consulta de entrada y crea una firma utilizando el mismo método que el programa que realiza la llamada. A continuación, compara las dos firmas. Si coinciden, el servicio de Storage puede comprobar la versión del servicio para asegurarse de que es válida, comprobar que la fecha y la hora actuales están dentro del período especificado, asegurarse de que el acceso solicitado corresponde a la solicitud realizada, etc.

Por ejemplo, con la dirección URL anterior, si la dirección URL apuntase a un archivo en lugar de a un blob, esta solicitud produciría un error porque especifica que la Firma de acceso compartido es para un blob. Si el comando REST al que se llama fuera para actualizar un blob, se produciría un error porque la Firma de acceso compartido especifica que se permite acceso de solo lectura.

#### <a name="types-of-shared-access-signatures"></a>Tipos de Firmas de acceso compartido
* Es posible utilizar una Firma de acceso compartido a nivel de servicio para tener acceso a recursos específicos de una cuenta de almacenamiento. Por ejemplo, para recuperar una lista de los blobs de un contenedor, descargar un blob, actualizar una entidad en una tabla, agregar mensajes a una cola o cargar un archivo en un recurso compartido de archivos.
* Una Firma de acceso compartido a nivel de cuenta se puede utilizar para tener acceso a lo mismo para lo que se puede usar una Firma de acceso compartido a nivel de servicio. Además, puede dar opciones a los recursos que no se permiten con una Firma de acceso compartido a nivel de servicio, como la posibilidad de crear contenedores, tablas, colas y recursos compartidos de archivos. También puede especificar el acceso a varios servicios a la vez. Por ejemplo, puede conceder a alguien acceso tanto a blobs como a archivos en su cuenta de almacenamiento.

#### <a name="creating-a-sas-uri"></a>Creación de un URI de Firma de acceso compartido
1. Puede crear un URI a petición, definiendo todos los parámetros de consulta cada vez.

   Este enfoque es realmente flexible, pero si tiene un conjunto lógico de parámetros que se parecen cada vez, es mejor utilizar una Directiva de acceso almacenada.
2. Puede crear una Directiva de acceso almacenada para todo un contenedor, el recurso compartido de archivos, la tabla o la cola. A continuación, puede utilizar esto como base para los URI de Firma de acceso compartido que cree. Los permisos basados en Directivas de acceso almacenadas se pueden revocar fácilmente. Puede tener hasta cinco directivas definidas en cada contenedor, cola, tabla o recurso compartido de archivos.

   Por ejemplo, si en su caso muchas personas fueran a leer los blobs de un contenedor determinado, podría crear una Directiva de acceso almacenada para "conceder acceso de lectura" y cualquier otro valor que se repita. A continuación, puede crear un URI de Firma de acceso compartido mediante la configuración de la Directiva de acceso almacenada y la especificación de la fecha y hora de expiración. Esto presenta la ventaja de que no es necesario especificar siempre todos los parámetros de consulta.

#### <a name="revocation"></a>Revocación
Supongamos que se ha puesto en riesgo su Firma de acceso compartido o que desea cambiarla porque así se lo exigen los requisitos de seguridad de la empresa o el cumplimiento normativo. ¿Cómo se podría revocar el acceso a un recurso que use esa Firma de acceso compartido? Depende de cómo haya creado el URI de la Firma de acceso compartido.

Si utiliza URI ad hoc, tiene tres opciones. Puede emitir tokens de Firmas de acceso compartido con directivas de expiración breves y esperar a que las firmas expiren. Puede cambiar el nombre del recurso o eliminarlo (suponiendo que el token estuviera limitado a un único objeto). Puede cambiar las claves de cuenta de almacenamiento. Esta última opción puede tener importantes repercusiones, en función de cuántos servicios estén utilizando esa cuenta de almacenamiento, y no suele ser una buena idea hacerlo sin planificarlo previamente.

Si está utilizando una Firma de acceso compartido derivada de una directiva de acceso almacenada, puede quitar el acceso revocando la Directiva de acceso almacenada (puede cambiar solo lo que ya ha expirado o bien quitarlo todo en su conjunto). Esto surte efecto inmediatamente e invalida todas las Firmas de acceso compartido creadas utilizando esa Directiva de acceso almacenada. El hecho de actualizar o quitar la Directiva de acceso almacenada puede repercutir en los usuarios que accedan específicamente a ese contenedor, recurso compartido de archivos, tabla o cola a través de la Firma de acceso compartido, pero si se han escrito los clientes de manera que soliciten una nueva Firma de acceso compartido cuando la anterior deje de ser válida, esto funcionará correctamente.

Dado que el uso de una Firma de acceso compartido derivada de una Directiva de acceso almacenada ofrece la posibilidad de revocar esa firma de acceso compartido de inmediato, se recomienda usar siempre las Directivas de acceso almacenadas cuando sea posible.

#### <a name="resources"></a>Recursos
Para más información sobre el uso de Firmas de acceso compartido y Directivas de acceso almacenadas, junto con ejemplos, consulte los artículos siguientes:

* Estos son los artículos de referencia.

  * [Ejemplos SAS del servicio.](https://msdn.microsoft.com/library/dn140256.aspx)

    Este artículo proporciona ejemplos de cómo utilizar una SAS de nivel de servicio con blobs, mensajes de cola, intervalos de tabla y archivos.
  * [Creación de una SAS de servicio](https://msdn.microsoft.com/library/dn140255.aspx)
  * [Creación de una SAS de cuenta](https://msdn.microsoft.com/library/mt584140.aspx)

* Este es un tutorial para usar la biblioteca de cliente de .NET para crear firmas de acceso compartido y directivas de acceso almacenadas.
  * [Uso de firmas de acceso compartido (SAS)](../storage-dotnet-shared-access-signature-part-1.md)

    Este artículo incluye una explicación del modelo de SAS, ejemplos de SAS y recomendaciones para el mejor uso práctico de SAS. También trata la revocación de los permisos concedidos.

* Authentication

  * [Autenticación para los servicios de Azure Storage](https://msdn.microsoft.com/library/azure/dd179428.aspx)
* Tutorial de introducción a las Firmas de acceso compartido

  * [Getting Started with Shared Access Signatures (SAS) (Introducción a las Firmas de acceso compartido)](https://github.com/Azure-Samples/storage-dotnet-sas-getting-started)

## <a name="encryption-in-transit"></a>Cifrado en tránsito
### <a name="transport-level-encryption--using-https"></a>Cifrado de nivel de transporte – Uso de HTTPS
Otro paso que debe seguir para garantizar la seguridad de los datos de Azure Storage es cifrar los datos entre el cliente y Azure Storage. La primera recomendación es utilizar siempre el protocolo [HTTPS](https://en.wikipedia.org/wiki/HTTPS), que garantiza una comunicación segura a través de la red pública de Internet.

Para tener un canal de comunicación seguro, siempre debe usar HTTPS al llamar a las API de REST u obtener acceso a objetos de almacenamiento. Además, las **Firmas de acceso compartido**, que pueden utilizarse para delegar el acceso a objetos de Azure Storage, incluyen una opción para especificar que se utilice solo el protocolo HTTPS con las Firmas de acceso compartido, lo que garantiza que cualquiera que envíe vínculos con tokens de SAS utilice el protocolo adecuado.

Puede exigir el uso de HTTPS al llamar a las API de REST para acceder a objetos de cuentas de almacenamiento habilitando la opción [Se requiere transferencia segura](../storage-require-secure-transfer.md) para la cuenta de almacenamiento. Una vez que esta opción esté habilitada, se rechazarán las conexiones que usan HTTP.

### <a name="using-encryption-during-transit-with-azure-file-shares"></a>Uso del cifrado durante el tránsito con recursos compartidos de archivos de Azure
[Azure Files](../files/storage-files-introduction.md) admite el cifrado mediante SMB 3.0 y con HTTPS cuando se usa la API REST File. Cuando el montaje se realiza fuera de la región de Azure en la que se encuentra el recurso compartido de archivos de Azure, por ejemplo, en local o en otra región de Azure, siempre es necesario SMB 3.0 con cifrado. SMB 2.1 no admite cifrado, por lo que, de forma predeterminada, solo se permiten las conexiones dentro de la misma región de Azure; pero se puede exigir la aplicación de SMB 3.0 con cifrado al [requerir transferencia segura](../storage-require-secure-transfer.md) para la cuenta de almacenamiento.

SMB 3.0 con cifrado está disponible en [todos los sistemas operativos Windows y Windows Server compatibles](../files/storage-how-to-use-files-windows.md) excepto Windows 7 y Windows Server 2008 R2, que solo admiten SMB 2.1. SMB 3.0 también es compatible con [macOS](../files/storage-how-to-use-files-mac.md) y con las distribuciones de [Linux](../files/storage-how-to-use-files-linux.md) que usan el kernel de Linux 4.11 y versiones posteriores. La compatibilidad con el cifrado de SMB 3.0 también ha sido portada a versiones anteriores del núcleo de Linux en varias distribuciones de Linux, consulte [Descripción de los requisitos del cliente SMB](../files/storage-how-to-use-files-linux.md#smb-client-reqs) para más información.

### <a name="using-client-side-encryption-to-secure-data-that-you-send-to-storage"></a>Uso del cifrado de cliente para proteger los datos que envía al almacenamiento
Otra opción que le ayuda a garantizar que sus datos están protegidos mientras se transfieren entre una aplicación cliente y el servicio Almacenamiento es el Cifrado de cliente. Los datos se cifran antes de ser transferidos a Azure Storage. En el proceso de recuperación de los datos de Azure Storage, estos se descifran una vez recibidos en el cliente. Aunque los datos se cifran al pasar por el cable, se recomienda que también se utilice HTTPS, ya sus comprobaciones incorporadas de la integridad de los datos ayudan a mitigar los errores de red que pueden incidir en esta.

El Cifrado de cliente es también un método para cifrar los datos en reposo, ya que los datos se almacenan en su forma cifrada. Hablaremos sobre esto con más detalle en la sección [Cifrado en reposo](#encryption-at-rest).

## <a name="encryption-at-rest"></a>Cifrado en reposo
Hay tres características de Azure que proporcionan cifrado en reposo. Azure Disk Encryption se utiliza para cifrar los discos de datos y del sistema operativo en máquinas virtuales de IaaS. El Cifrado de cliente y el Cifrado del servicio de almacenamiento se utilizan para cifrar los datos en Azure Storage. 

Aunque puede usar el Cifrado de cliente para cifrar los datos en tránsito (que también se almacenan en su forma cifrada en Almacenamiento), es preferible usar HTTPS durante la transferencia y disponer de algún método para que los datos se cifren automáticamente al almacenarse. Hay dos maneras de hacerlo: Azure Disk Encryption y SSE. Uno se utiliza para cifrar directamente los datos en los discos de datos y del sistema operativo usados por las máquinas virtuales, y el otro para cifrar los datos escritos en Azure Blob Storage.

### <a name="storage-service-encryption-sse"></a>cifrado del servicio de almacenamiento (SSE)

SSE está habilitado para todas las cuentas de almacenamiento y no puede deshabilitarse. SSE cifra automáticamente los datos al escribirlos en Azure Storage. Al leer datos de Azure Storage, Azure Storage los descifra antes de devolverlos. SSE permite proteger los datos sin tener que modificar el código o agregar código a las aplicaciones.

Puede usar claves administrada por Microsoft o sus propias claves personalizadas. Microsoft genera las claves administradas y se encarga de guardarlas en un lugar seguro y de cambiarlas de forma periódica de acuerdo con la directiva interna de Microsoft. Para más información sobre el uso de claves personalizadas, consulte [Uso de Storage Service Encryption con claves administradas por el cliente en Azure Key Vault](storage-service-encryption-customer-managed-keys.md).

SSE cifra automáticamente los datos de todos los niveles de rendimiento (Estándar y Premium), todos los modelos de implementación (Azure Resource Manager y clásico) y todos los servicios de Azure Storage (Blob, Queue, Table y File). 

### <a name="client-side-encryption"></a>cifrado de cliente
Hemos mencionado el Cifrado de cliente al hablar sobre el cifrado de los datos en tránsito. Esta característica le permite cifrar mediante programación los datos en una aplicación cliente antes de enviarlos a través del cable para que se escriban en el Azure Storage, y descifrar mediante programación los datos después de recuperarlos de Azure Storage.

Proporciona cifrado en tránsito, pero además incluye la característica de cifrado en reposo. Aunque los datos se cifran en tránsito, sigue siendo recomendable usar HTTPS para aprovechar las comprobaciones incorporadas de integridad de los datos que ayudan a mitigar los errores de red que pueden incidir en esta.

Por ejemplo, pongamos que tiene una aplicación web que almacena blobs y recupera blobs, y desea que la aplicación y los datos sean lo más seguros posibles. En ese caso, utilizaría el cifrado de cliente. El tráfico entre el cliente y Azure Blob service contiene el recurso cifrado, y nadie puede interpretar los datos en tránsito y reconstituirlos en los blobs privados.

El cifrado de cliente se integra en Java y las bibliotecas de cliente de Storage de .NET, que a su vez utilizan las API de Azure Key Vault, por lo que es fácil de implementar. El proceso de cifrado y descifrado de los datos usa la técnica del sobre, y almacena los metadatos usados por el cifrado en cada objeto de almacenamiento. Por ejemplo, en el caso de los blobs, los almacena en los metadatos del blob, mientras que para las colas, los agrega a cada mensaje de la cola.

Para el propio cifrado, puede generar y administrar sus propias claves de cifrado. También puede usar las claves generadas por la biblioteca de clientes de Azure Storage, o puede hacer que Azure Key Vault genere las claves. Puede almacenar las claves de cifrado en el almacenamiento de claves local o en Azure Key Vault. Azure Key Vault permite conceder acceso a los secretos de Azure Key Vault a usuarios específicos mediante Azure Active Directory. Esto significa que no todo el mundo puede leer Azure Key Vault y recuperar las claves que utiliza para el cifrado del cliente.

#### <a name="resources"></a>Recursos
* [Cifrado y descifrado de blobs en Microsoft Azure Storage con Azure Key Vault](../blobs/storage-encrypt-decrypt-blobs-key-vault.md)

  Este artículo muestra cómo utilizar el cifrado de cliente con Azure Key Vault, incluido cómo crear la KEK y almacenarlo en el almacén con PowerShell.
* [Cifrado del lado de cliente y Azure Key Vault para Microsoft Azure Storage](../storage-client-side-encryption.md)

  Este artículo ofrece una explicación del cifrado de cliente y proporciona ejemplos de uso de la biblioteca de clientes de Storage para cifrar y descifrar los recursos de los cuatro servicios de Storage. También trata Azure Key Vault.

### <a name="using-azure-disk-encryption-to-encrypt-disks-used-by-your-virtual-machines"></a>Uso de Azure Disk Encryption para cifrar discos usados por las máquinas virtuales
Azure Disk Encryption permite cifrar los discos del sistema operativo y discos de datos usados por una máquina Virtual de IaaS. Para Windows, las unidades se cifran mediante la tecnología de cifrado de BitLocker estándar del sector. Para Linux, los discos se cifran mediante la tecnología DM-Crypt. Se integra con Azure Key Vault para permitirle controlar y administrar las claves de cifrado del disco.

La solución admite los siguientes escenarios para las máquinas virtuales IaaS cuando se habilitan en Microsoft Azure:

* Integración con Azure Key Vault
* VM de nivel estándar: [VM IaaS de las series A, D, DS, G, GS, etc.](https://azure.microsoft.com/pricing/details/virtual-machines/)
* Habilitación del cifrado en máquinas virtuales IaaS Linux y Windows
* Deshabilitación del cifrado en las unidades de datos y del sistema operativo en máquinas virtuales IaaS Windows
* Deshabilitación del cifrado en unidades de datos en máquinas virtuales IaaS Linux
* Habilitación del cifrado en máquinas virtuales IaaS que ejecutan el sistema operativo cliente de Windows
* Habilitación del cifrado en volúmenes con rutas de montaje
* Habilitación del cifrado en máquinas virtuales Linux configuradas con seccionamiento de disco (RAID) mediante mdadm
* Habilitación del cifrado en máquinas virtuales con Linux mediante LVM para discos de datos
* Habilitación del cifrado en máquinas virtuales con Windows configuradas mediante Espacios de almacenamiento
* Se admiten todas las regiones públicas de Azure.

La solución no admite los siguientes escenarios, características y tecnologías en la versión:

* Máquinas virtuales IaaS de nivel básico
* Deshabilitación del cifrado en una unidad del sistema operativo para máquinas virtuales IaaS Linux
* Máquinas virtuales IaaS creadas con el método clásico de generación de máquinas virtuales
* Integración con el Servicio de administración de claves local
* Azure Files (sistema de archivos compartido), Network File System (NFS), volúmenes dinámicos y máquinas virtuales Windows configuradas con sistemas RAID basadas en software


> [!NOTE]
> El cifrado de disco del sistema operativo Linux es actualmente compatible en las distribuciones Linux siguientes: RHEL 7.2, CentOS 7.2 y Ubuntu 16.04.
>
>

Esta característica garantiza que todos los datos de los discos de máquinas virtuales se cifran en reposo en Azure Storage.

#### <a name="resources"></a>Recursos
* [Azure Disk Encryption para máquinas virtuales IaaS Linux y Windows](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)

### <a name="comparison-of-azure-disk-encryption-sse-and-client-side-encryption"></a>Comparación entre Azure Disk Encryption, SSE y cifrado del lado cliente

#### <a name="iaas-vms-and-their-vhd-files"></a>Máquinas virtuales de IaaS y sus archivos VHD

Para los discos de datos usados por las máquinas virtuales de IaaS, se recomienda utilizar Azure Disk Encryption. Si crea una máquina virtual con discos no administrados mediante una imagen de Azure Marketplace, Azure realiza una [copia superficial](https://en.wikipedia.org/wiki/Object_copying) de ella en la cuenta de almacenamiento de Azure Storage y no se cifra aunque tenga habilitado SSE. Cuando se cree la máquina virtual y comience a actualizarse la imagen, SSE iniciará el cifrado de los datos. Por este motivo, es mejor usar Azure Disk Encryption en máquinas virtuales con discos no administrados creadas a partir de imágenes de Azure Marketplace si quiere que estén totalmente cifradas. Si crea una máquina virtual con discos administrados, SSE cifra todos los datos de forma predeterminada mediante las claves de la plataforma administrada. 

Si trae una máquina virtual previamente cifrada a Azure desde el entorno local, podrá cargar las claves de cifrado en Azure Key Vault y mantener el cifrado para esa máquina virtual que se utilizaba de forma local. La configuración de Azure Disk Encryption le permite controlar este escenario.

Si dispone de un VHD no cifrado desde el entorno local, puede cargarlo en la galería como una imagen personalizada y aprovisionar una máquina virtual a partir del mismo. Si lo hace con las plantillas de Resource Manager, puede solicitar la activación de Azure Disk Encryption al arrancar la máquina virtual.

Al agregar un disco de datos y montarlo en la máquina virtual, puede activar Azure Disk Encryption en ese disco de datos. Cifrará primero ese disco de datos localmente y, a continuación, la capa del modelo de implementación clásica hará una escritura diferida en el almacenamiento para que se cifre el contenido del mismo.

#### <a name="client-side-encryption"></a>Cifrado de cliente
El Cifrado de cliente es el método más seguro para cifrar los datos, porque estos se cifran antes del tránsito.  Sin embargo, requiere que agregue código a sus aplicaciones mediante el almacenamiento, y quizá no desee hacerlo. En esos casos, puede usar HTTPS para proteger los datos en tránsito. Una vez que los datos llegan a Azure Storage, SSE los cifra.

Con el cifrado de cliente, puede cifrar las entidades de tabla, los mensajes de colas y los blobs. 

El Cifrado de cliente es administrado completamente por la aplicación. Es el método más seguro, pero es necesario realizar cambios mediante programación en la aplicación e implementar procesos de administración de claves. Es recomendable usarlo si desea una seguridad adicional durante el tránsito y desea que los datos almacenados se cifren.

El Cifrado de cliente supone más carga en el cliente; así pues, debe tener en cuenta este factor en los planes de escalabilidad, especialmente si está cifrando y transfiriendo una gran cantidad de datos.

#### <a name="storage-service-encryption-sse"></a>cifrado del servicio de almacenamiento (SSE)

SSE se administra mediante Azure Storage. SSE no proporciona seguridad para los datos en tránsito, pero cifra los datos a medida que se escriben en Azure Storage. SSE no afecta al rendimiento de Azure Storage.

Puede cifrar cualquier tipo de datos de la cuenta de almacenamiento mediante SSE (blobs en bloques, blobs en anexos, blobs en páginas, datos de tablas, datos de colas y archivos).

Si tiene un archivo o una biblioteca de archivos VHD que utiliza como base para la creación de nuevas máquinas virtuales, puede crear una cuenta de almacenamiento y luego cargar los archivos VHD en esa cuenta. Esos archivos VHD se cifrarán con el Azure Storage.

Si tiene habilitado Azure Disk Encryption para los discos de una máquina virtual, todos los datos recién escritos se cifran con SSE y Azure Disk Encryption.

## <a name="storage-analytics"></a>Storage Analytics
### <a name="using-storage-analytics-to-monitor-authorization-type"></a>Uso de Storage Analytics para supervisar el tipo de autorización
Para cada cuenta de almacenamiento, puede habilitar Azure Storage Analytics para realizar el registro y almacenar datos de métricas. Se trata de una excelente herramienta que puede usar para comprobar las métricas de rendimiento de una cuenta de almacenamiento o cuando necesite solucionar los problemas de rendimiento de una cuenta de almacenamiento.

Los registros de Storage Analytics contienen también el método de autenticación utilizado por un usuario al acceder al almacenamiento. Por ejemplo, con Blob Storage, puede ver si los usuarios utilizan una Firma de acceso compartido o las claves de cuenta de almacenamiento, o si el blob es de acceso público.

Esto puede resultar útil si se está esforzando por proteger el acceso al almacenamiento. Por ejemplo, en Blob Storage puede configurar todos los contenedores para que sean privados e implementar el uso de un servicio de Firma de acceso compartido en sus aplicaciones. Luego puede comprobar los registros periódicamente para ver si se ha accedido a los blobs con las claves de cuenta de almacenamiento, lo que podría indicar una infracción de seguridad, o si los blobs son públicos pero no deberían serlo.

#### <a name="what-do-the-logs-look-like"></a>¿Qué aspecto tienen los registros?
Después de habilitar las métricas de la cuenta de almacenamiento y el registro a través de Azure Portal, los datos del análisis empezarán a acumularse rápidamente. El registro y las métricas de cada servicio son independientes; el registro solo se escribe cuando hay actividad en esa cuenta de almacenamiento, mientras que las métricas se registrarán cada minuto, cada hora o cada día, dependiendo de la configuración.

Los registros se almacenan en blobs en bloques en un contenedor denominado $logs en la cuenta de almacenamiento. Este contenedor se crea automáticamente cuando se habilita Storage Analytics. Una vez creado este contenedor, no se puede eliminar, aunque sí puede eliminar su contenido.

En el contenedor $logs, hay una carpeta para cada servicio y luego hay subcarpetas para año/mes/día/hora. En la hora, se numeran los registros. Este es el aspecto que tendrá la estructura de directorios:

![Vista de archivos de registro](./media/storage-security-guide/image1.png)

Se registra cada solicitud para el Azure Storage. Esta es una instantánea de un archivo de registro donde se muestran algunos de los primeros campos.

![Instantánea de un archivo de registro](./media/storage-security-guide/image2.png)

Puede ver que puede usar los registros para realizar el seguimiento de cualquier tipo de llamadas a una cuenta de almacenamiento.

#### <a name="what-are-all-of-those-fields-for"></a>¿Para qué sirven todos estos campos?
Este es un artículo que aparece en los siguientes recursos y que proporciona la lista de los numerosos campos de los registros (explicando, además, para qué se utilizan). Esta es la lista de campos en orden:

![Instantánea de los campos de un archivo de registro](./media/storage-security-guide/image3.png)

Nos interesan las entradas de GetBlob y el modo de autorización, así que nos fijaremos en las entradas con el tipo de operación "Get-Blob" y comprobaremos el estado de la solicitud (cuarta</sup> columna) y el tipo de autorización (octava</sup> columna).

Por ejemplo, en las primeras filas del listado anterior, el estado de la solicitud es "Success" y el tipo de autorización es "authenticated". Esto significa que la solicitud se autorizó con la clave de la cuenta de almacenamiento.

#### <a name="how-is-access-to-my-blobs-being-authorized"></a>¿Cómo se autoriza el acceso a mis blobs?
Son tres los casos que nos interesan.

1. El blob es público y se tiene acceso a él mediante una dirección URL sin Firma de acceso compartido. En este caso, el estado de la solicitud es "AnonymousSuccess" y el tipo de autorización es "anonymous".

   1.0;2015-11-17T02:01:29.0488963Z;GetBlob;**AnonymousSuccess**;200;124;37;**anonymous**;;mystorage…
2. El blob es privado y se utilizó con una Firma de acceso compartido. En este caso, el estado de la solicitud es "SASSuccess" y el tipo de autorización es "sas".

   1.0;2015-11-16T18:30:05.6556115Z;GetBlob;**SASSuccess**;200;416;64;**sas**;;mystorage…
3. El blob es privado y la clave de almacenamiento se utilizó para tener acceso a él. En este caso, el estado de la solicitud es "**Success**" y el tipo de autorización es "**authenticated**".

   1.0;2015-11-16T18:32:24.3174537Z;GetBlob;**Success**;206;59;22;**authenticated**;mystorage…

Puede utilizar el Analizador de mensajes de Microsoft para ver y analizar estos registros. Incluye funcionalidades de búsqueda y filtrado. Por ejemplo, puede que desee buscar instancias de GetBlob para ver si efectivamente se les está dando el uso que espera; es decir, para asegurarse de que un usuario no tenga acceso la cuenta de almacenamiento incorrectamente.

#### <a name="resources"></a>Recursos
* [Storage Analytics](../storage-analytics.md)

  Este artículo ofrece una visión general de Storage Analytics y explica cómo habilitarlo.
* [Formato del registro de Storage Analytics](https://msdn.microsoft.com/library/azure/hh343259.aspx)

  Este artículo ilustra el formato del registro de Storage Analytics y detalla los campos que incluye, como el tipo de autenticación, que indica el tipo de autenticación utilizado para la solicitud.
* [Supervisión de una cuenta de almacenamiento en Azure Portal](../storage-monitor-storage-account.md)

  Este artículo muestra cómo configurar la supervisión de las métricas y el registro para una cuenta de almacenamiento.
* [Solución integral de problemas con los registros y métricas de Azure Storage, AzCopy y el analizador de mensajes](../storage-e2e-troubleshooting.md)

  Este artículo trata la solución de problemas mediante Storage Analytics y muestra cómo utilizar el Analizador de mensajes de Microsoft.
* [Guía de funcionamiento del analizador de mensajes de Microsoft](https://technet.microsoft.com/library/jj649776.aspx)

  Este artículo es la referencia para el Analizador de mensajes de Microsoft e incluye vínculos a un tutorial, al inicio rápido y al resumen de características.

## <a name="cross-origin-resource-sharing-cors"></a>Uso compartido de recursos entre orígenes (CORS)
### <a name="cross-domain-access-of-resources"></a>Acceso entre dominios de recursos
Cuando un explorador web que se ejecuta en un dominio realiza una solicitud HTTP para un recurso desde un dominio diferente, el proceso se denomina solicitud HTTP entre orígenes. Por ejemplo, una página HTML atendida desde contoso.com realiza una solicitud para un jpeg hospedado en fabrikam.blob.core.windows.net. Por motivos de seguridad, los exploradores restringen las solicitudes HTTP entre orígenes iniciadas desde scripts, como JavaScript. Esto significa que cuando el código JavaScript de una página web en contoso.com solicite jpeg en fabrikam.blob.core.windows.net, el explorador no permitirá la solicitud.

¿Qué relación guarda esto con el Azure Storage? Si almacena recursos estáticos como archivos de datos JSON o XML en Blob Storage con una cuenta de almacenamiento llamada Fabrikam, el dominio de los recursos será fabrikam.blob.core.windows.net y la aplicación web de contoso.com no podrá tener acceso a ellos mediante JavaScript, ya que los dominios son diferentes. Esto se cumple también si trata de llamar a uno de los servicios de Azure Storage, como Table Storage, que devuelven datos JSON para que los procese el cliente JavaScript.

#### <a name="possible-solutions"></a>Posibles soluciones
Una manera de resolver esto es asignar un dominio personalizado, como "storage.contoso.com", a fabrikam.blob.core.windows.net. El problema es que solo se puede asignar ese dominio personalizado a una cuenta de almacenamiento. ¿Qué ocurre si los recursos se almacenan en varias cuentas de almacenamiento?

Otra manera de resolver este problema es hacer que la aplicación web actúe como proxy para las llamadas de almacenamiento. Esto significa que si está cargando un archivo a Blob Storage, la aplicación web podría escribirlo localmente y, a continuación, copiarlo en Blob Storage; o bien leerlo entero en la memoria y, a continuación, escribirlo en Blob Storage. Como alternativa, podría escribir una aplicación web dedicada (como una API web) que cargue los archivos localmente y los escriba en Blob Storage. En cualquier caso, debe tener en cuenta esa función al determinar las necesidades de escalabilidad.

#### <a name="how-can-cors-help"></a>¿En qué sentido puede resultar útil el uso compartido de recursos entre orígenes?
Azure Storage permite habilitar el uso compartido de recursos entre orígenes. Puede especificar los dominios que pueden tener acceso a los recursos de cada una de las cuentas de almacenamiento. Por ejemplo, en nuestro caso descrito anteriormente, podemos habilitar el uso compartido de recursos entre orígenes en la cuenta de almacenamiento fabrikam.blob.core.windows.net y configurarlo para permitir el acceso a contoso.com. A continuación, la aplicación web contoso.com puede acceder directamente a los recursos en fabrikam.blob.core.windows.net.

Hay que destacar que el uso compartido de recursos entre orígenes permite el acceso, pero no proporciona autenticación, que es necesaria para todos los accesos no públicos de los recursos de almacenamiento. Esto significa que solo puede tener acceso a blobs si son públicos o se incluye una Firma de acceso compartido que le proporcione el permiso adecuado. Las tablas, las colas y los archivos no tienen acceso público y requieren una Firma de acceso compartido.

De forma predeterminada, el uso compartido de recursos entre orígenes está deshabilitado en todos los servicios. Puede habilitarlo mediante la API de REST o la biblioteca de clientes de Storage para llamar a uno de los métodos a fin de establecer las directivas del servicio. Cuando lo haga, debe incluir una regla de uso compartido de recursos entre orígenes, que está en formato XML. Aquí presentamos un ejemplo de regla de uso compartido de recursos entre orígenes que se ha establecido mediante la operación Set Service Properties para Blob Service para una cuenta de almacenamiento. Puede realizar la operación mediante la biblioteca de clientes de Storage o las API de REST para Azure Storage.

```xml
<Cors>    
    <CorsRule>
        <AllowedOrigins>http://www.contoso.com, http://www.fabrikam.com</AllowedOrigins>
        <AllowedMethods>PUT,GET</AllowedMethods>
        <AllowedHeaders>x-ms-meta-data*,x-ms-meta-target*,x-ms-meta-abc</AllowedHeaders>
        <ExposedHeaders>x-ms-meta-*</ExposedHeaders>
        <MaxAgeInSeconds>200</MaxAgeInSeconds>
    </CorsRule>
<Cors>
```

Esto es lo que significa cada fila:

* **AllowedOrigins**: señala qué dominios no coincidentes pueden solicitar y recibir datos del servicio Storage. En este caso, indica que contoso.com y fabrikam.com pueden solicitar datos desde Blob Storage para una cuenta de almacenamiento específica. También puede configurar este valor en un carácter comodín (\*) para permitir que todos los dominios accedan a las solicitudes.
* **AllowedMethods**: se trata de la lista de métodos (verbos de solicitud HTTP) que se puede utilizar al realizar la solicitud. En este ejemplo, se permiten solo GET y PUT. Puede configurar este valor en un carácter comodín (\*) para permitir que se usen todos los métodos.
* **AllowedHeaders**: se trata de los encabezados de solicitud que puede especificar el dominio de origen al realizar la solicitud. En el ejemplo anterior, se permiten todos los encabezados de metadatos que comienzan por x-ms-meta-data, x-ms-meta-target y x-ms-meta-abc. Tenga en cuenta que el carácter comodín (\*) indica que se permite cualquier encabezado que empiece con el prefijo especificado.
* **ExposedHeaders**: indica que el explorador debe exponer los encabezados de respuesta al emisor de la solicitud. En este ejemplo, se expondrán los encabezados que empiecen por "x-ms-meta-".
* **MaxAgeInSeconds**: el tiempo máximo que un explorador almacenará en la caché la solicitud preparatoria OPTIONS. (para más información acerca de la solicitud preparatoria, consulte el primer artículo de los que se presentan a continuación).

#### <a name="resources"></a>Recursos
Para más información sobre el uso compartido de recursos entre orígenes y cómo habilitarlo, consulte estos recursos.

* [Compatibilidad con Uso compartido de recursos entre orígenes (CORS) para los Servicios de Azure Storage en Azure.com](../storage-cors-support.md)

  Este artículo proporciona una visión general del uso compartido de recursos entre orígenes y explica cómo establecer las reglas para los diferentes servicios de Storage.
* [Compatibilidad con Uso compartido de recursos entre orígenes (CORS) para los Servicios de Azure Storage en MSDN](https://msdn.microsoft.com/library/azure/dn535601.aspx)

  Se trata de la documentación de referencia para la compatibilidad con el uso compartido de recursos entre orígenes para los servicios de Azure Storage. Contiene vínculos a artículos que se aplican a cada servicio de almacenamiento, se muestra un ejemplo y se explica cada elemento en el archivo del uso compartido de recursos entre orígenes.
* [Microsoft Azure Storage: Presentación de CORS](https://blogs.msdn.com/b/windowsazurestorage/archive/2014/02/03/windows-azure-storage-introducing-cors.aspx)

  Se trata de un vínculo al artículo inicial del blog en el que se anuncia el uso compartido de recursos entre orígenes y se muestra cómo utilizarlo.

## <a name="frequently-asked-questions-about-azure-storage-security"></a>Preguntas más frecuentes acerca de la seguridad de Azure Storage
1. **¿Cómo se puede comprobar la integridad de los blobs que estoy transfiriendo hacia o desde Azure Storage si no puedo usar el protocolo HTTPS?**

   Si por algún motivo debe utilizar HTTP en lugar de HTTPS y está trabajando con blobs en bloques, puede usar la comprobación de MD5 para ayudar a comprobar la integridad de los blobs que se transfieren. Esto le ayudará con la protección frente a errores de red o de la capa de transporte, pero no necesariamente con ataques de intermediarios.

   Si puede usar HTTPS, que proporciona seguridad de nivel de transporte, el uso de la comprobación de MD5 es redundante e innecesario.

   Para obtener más información, consulte [Windows Azure Blob MD5 Overview](https://blogs.msdn.com/b/windowsazurestorage/archive/2011/02/18/windows-azure-blob-md5-overview.aspx)(Información general de MD5 de Azure Blob service).
2. **¿Y qué ocurre con el cumplimiento de la norma FIPS para el Gobierno de EE. UU.?**

   La norma Federal Information Processing Standard (FIPS) de EE. UU. define los algoritmos criptográficos aprobados para su uso por los sistemas informáticos del Gobierno Federal de EE. UU para la protección de los datos confidenciales. La habilitación del modo FIPS en un servidor o escritorio de Windows indica al sistema operativo que solo se deben usar los algoritmos criptográficos validados por FIPS. Si una aplicación utiliza algoritmos no compatibles, se invalidarán las aplicaciones. Con .NET Framework en su versión 4.5.2 u otra posterior, la aplicación cambia automáticamente los algoritmos criptográficos para utilizar los algoritmos conformes a FIPS cuando el equipo está en modo FIPS.

   Microsoft permite que sea cada cliente quien decida si desea habilitar el modo FIPS. Creemos que los clientes que no están sujetos a reglamentaciones gubernamentales no tienen ninguna razón para habilitar el modo FIPS de forma predeterminada.

### <a name="resources"></a>Recursos
* [Why We're Not Recommending “FIPS Mode” Anymore](https://blogs.technet.microsoft.com/secguide/2014/04/07/why-were-not-recommending-fips-mode-anymore/) (Por qué ya no recomendamos el "modo FIPS")

  En este artículo de blog se proporciona información general de FIPS y se explica por qué no se habilita el modo FIPS de forma predeterminada.
* [FIPS 140 Validation (Validación FIPS 140)](https://technet.microsoft.com/library/cc750357.aspx)

  Este artículo proporciona información sobre el cumplimiento de los productos y los módulos criptográficos de Microsoft con la norma FIPS para el Gobierno Federal de EE. UU.
* ["Criptografía de sistema: usar algoritmos compatibles con FIPS para cifrado, firma y operaciones hash" efectos de la configuración de seguridad en Windows XP y en versiones posteriores de Windows](https://support.microsoft.com/kb/811833)

  Este artículo aborda el uso del modo FIPS en equipos más antiguos de Windows.
