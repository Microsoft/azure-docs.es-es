---
title: Concesión de acceso limitado a datos con firmas de acceso compartido (SAS)
titleSuffix: Azure Storage
description: Obtenga información acerca de cómo usar firmas de acceso compartido (SAS) para delegar recursos de Azure Storage, incluidos blobs, colas, tablas y archivos.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 08/17/2020
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: common
ms.openlocfilehash: b9882168cd063cb4448269cc6a4949778fe93fb1
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88509865"
---
# <a name="grant-limited-access-to-azure-storage-resources-using-shared-access-signatures-sas"></a>Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)

Una firma de acceso compartido (SAS) ofrece acceso delegado a los recursos en la cuenta de almacenamiento sin poner en peligro la seguridad de los datos. Con una SAS, tiene control granular sobre la forma en que un cliente puede tener acceso a los datos. Puede controlar a qué recursos puede tener acceso el cliente, qué permisos tienen en esos recursos y cuánto tiempo es válida la SAS, entre otros parámetros.

## <a name="types-of-shared-access-signatures"></a>Tipos de firmas de acceso compartido

Azure Storage admite tres tipos de firmas de acceso compartido:

- **SAS de delegación de usuarios.** Una SAS de delegación de usuarios está protegida con credenciales de Azure Active Directory (Azure AD) y también con los permisos especificados para la SAS. Una SAS de delegación de usuarios solo se aplica a Blob Storage.

    Para más información sobre la SAS de delegación de usuarios, consulte [Create a user delegation SAS (REST API)](/rest/api/storageservices/create-user-delegation-sas) (Creación de una SAS de delegación de usuarios [API REST]).

- **SAS de servicio.** Una SAS de servicio está protegida con la clave de cuenta de almacenamiento. Una SAS de servicio administra el acceso a un recurso en solo uno de los servicios de Azure Storage: Blob Storage, Queue Storage, Table Storage o Azure Files.

    Para más información sobre la SAS de servicio, consulte [Create a service SAS (REST API)](/rest/api/storageservices/create-service-sas) (Creación de una SAS de servicio [API REST]).

- **SAS de cuenta.** Una SAS de cuenta está protegida con la clave de cuenta de almacenamiento. SAS de cuenta delega el acceso a los recursos en uno o varios de los servicios de almacenamiento. Todas las operaciones disponibles con una SAS de servicio o delegación de usuarios están también disponibles con una SAS de cuenta. Además, con la SAS de cuenta, puede delegar el acceso a las operaciones que se aplican a nivel de servicio, como las operaciones **Get/Set Service Properties** y **Get Service Stats**. También puede delegar el acceso para leer, escribir y eliminar operaciones en contenedores de blobs, tablas, colas y recursos compartidos de archivos que no están permitidos con SAS de servicio. 

    Para obtener más información sobre la SAS de cuenta, consulte [Create an account SAS (REST API)](/rest/api/storageservices/create-account-sas) (Creación de una SAS de cuenta [API REST]).

> [!NOTE]
> Microsoft recomienda usar credenciales de Azure AD cuando sea posible como procedimiento recomendado de seguridad, en lugar de usar la clave de cuenta, que se puede poner en peligro más fácilmente. Cuando el diseño de la aplicación requiera firmas de acceso compartido para el acceso a Blob Storage, utilice credenciales de Azure AD para crear una SAS de delegación de usuarios cuando sea posible para una seguridad superior. Para más información, vea [Autenticación del acceso a blobs y colas con Azure Active Directory](storage-auth-aad.md).

Una firma de acceso compartido puede presentar una de estas dos formas:

- **SAS ad-hoc:** cuando cree una SAS ad-hoc, la hora de inicio, la hora de expiración y los permisos para la SAS se especifican en el URI de SAS (o se encuentran implícitos en el caso de que se omita la hora de inicio). Cualquier tipo de SAS puede ser una SAS ad-hoc.
- **SAS de servicio con directiva de acceso almacenada:** se define una directiva de acceso almacenada en un contenedor de recursos, que puede ser un contenedor de blobs, tabla, cola o recurso compartido de archivos. Una directiva de acceso almacenada puede usarse para administrar las restricciones de una o varias firmas de acceso compartido de servicio. Cuando asocia una SAS de servicio a una directiva de acceso almacenada, la SAS hereda las restricciones &mdash;(hora de inicio, hora de expiración y permisos)&mdash; definidas para la directiva de acceso almacenada.

> [!NOTE]
> Una SAS de delegación de usuarios o una SAS de cuenta debe ser una SAS ad-hoc. Las directivas de acceso almacenadas no son compatibles con la SAS de delegación de usuarios ni de cuenta.

## <a name="how-a-shared-access-signature-works"></a>Funcionamiento de una firma de acceso compartido

Una firma de acceso compartido es un URI firmado que señala a uno o más recursos de almacenamiento e incluye un token que contiene un conjunto especial de parámetros de consulta. El token indica cómo puede el cliente tener acceso a los recursos. Uno de los parámetros de consulta, la firma, se construye a partir de parámetros SAS y se firma con la clave que se usó para crear la SAS. Azure Storage utiliza esta firma para autorizar el acceso al recurso de almacenamiento.

### <a name="sas-signature-and-authorization"></a>Firma y autorización de SAS

Puede firmar un token de SAS de dos maneras:

- Con una *clave de delegación de usuarios* creada con las credenciales de Azure Active Directory (Azure AD). Una SAS de delegación de usuarios está firmada con la clave de delegación de usuarios.

    Para obtener la clave de delegación de usuarios y crear la SAS, a una entidad de seguridad de Azure AD se le debe asignar un rol de Azure que incluya la acción **Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey**. Para obtener información detallada sobre los roles de Azure con permisos para obtener la clave de delegación de usuarios, consulte [Creación de una SAS de delegación de usuarios (API REST)](/rest/api/storageservices/create-user-delegation-sas).

- Con la clave de la cuenta de almacenamiento (clave compartida) Tanto una SAS de servicio como una SAS de cuenta se firman con la clave de cuenta de almacenamiento. Para crear una SAS firmada con la clave de cuenta, una aplicación debe tener acceso a la clave de cuenta.

Cuando una solicitud incluye un token de SAS, esa solicitud se autoriza en función de cómo se firma el token de SAS. La clave de acceso o las credenciales que se usan para crear un token de SAS también se usan en Azure Storage para conceder acceso a un cliente que posea el token de SAS.

En la tabla siguiente se resume el modo en que cada tipo de token de SAS está autorizado cuando se incluye en una solicitud a Azure Storage:

| Tipo de SAS | Tipo de autorización |
|-|-|
| SAS de delegación de usuarios (solo para Blob Storage) | Azure AD |
| SAS de servicio | Clave compartida |
| SAS de cuenta | Clave compartida |

Microsoft recomienda el uso de una SAS de delegación de usuario siempre que sea posible para mayor seguridad.

### <a name="sas-token"></a>Token de SAS

El token de SAS es una cadena que se genera del lado cliente, por ejemplo, mediante una de las bibliotecas cliente de Azure Storage. Azure Storage no realiza un seguimiento del token de SAS de ninguna manera. Puede crear un número ilimitado de tokens de SAS en el cliente. Después de crear una SAS, puede distribuirla a las aplicaciones cliente que requieran acceso a los recursos de la cuenta de almacenamiento.

Cuando una aplicación cliente proporciona un URI de SAS para Azure Storage como parte de una solicitud, el servicio comprueba los parámetros de SAS y la firma para comprobar que es válido para autorizar la solicitud. Si el servicio confirma que la firma es válida, la solicitud se autoriza. De lo contrario, la solicitud se rechaza con el código de error 403 (prohibido).

Este es un ejemplo de un URI de SAS de servicio, que muestra el URI de recurso y el token de SAS:

![Componentes de un URI de SAS de servicio](./media/storage-sas-overview/sas-storage-uri.png)

## <a name="when-to-use-a-shared-access-signature"></a>¿Cuándo debe usar una firma de acceso compartido?

Use una SAS cuando quiera proporcionar acceso seguro a los recursos de la cuenta de almacenamiento a cualquier cliente que, de otro modo, no tenga permisos para esos recursos.

Un escenario común en el que es útil una SAS es un servicio en el que los usuarios leen y escriben sus propios datos en la cuenta de almacenamiento. Existen dos patrones de diseño típicos en los escenarios en los que una cuenta de almacenamiento guarda datos de usuario:

1. Los clientes cargan y descargan datos mediante un servicio de proxy front-end que realiza la autenticación. Este servicio de proxy front-end cuenta con la ventaja de permitir la validación de reglas de negocio, pero para grandes cantidades de datos o transacciones de gran volumen, la creación de un servicio que pueda escalarse para satisfacer la demanda puede ser complicada o costosa.

   ![Diagrama del escenario: servicio de proxy de front-end](./media/storage-sas-overview/sas-storage-fe-proxy-service.png)

1. Un servicio ligero realiza la autenticación del cliente según sea necesario y, luego, genera una SAS. Una vez que la aplicación cliente recibe la SAS, puede obtener acceso a los recursos de la cuenta de almacenamiento directamente con los permisos definidos por la SAS y para el intervalo permitido por ella. La SAS mitiga la necesidad de enrutar todos los datos a través del servicio de proxy front-end.

   ![Diagrama del escenario: servicio de proveedor de SAS](./media/storage-sas-overview/sas-storage-provider-service.png)

Muchos servicios reales pueden usar una combinación de estos dos enfoques. Por ejemplo, se pueden procesar y validar algunos datos mediante el proxy de front-end, mientras que otros datos se guardan o leen directamente mediante SAS.

Además, en ciertos escenarios, se requiere una SAS para autorizar el acceso al objeto de origen en las operaciones de copia:

- Cuando copia un blob en otro blob que reside en una cuenta de almacenamiento diferente, debe usar una SAS para autorizar el acceso al blob de origen. Si lo desea, también puede usar una SAS para autorizar el acceso al blob de destino.
- Cuando copia un archivo en otro archivo que se encuentra en una cuenta de almacenamiento diferente, debe usar una SAS para autorizar el acceso al archivo de origen. Si lo desea, también puede usar una SAS para autorizar el acceso al archivo de destino.
- Si va a copiar un blob en un archivo, o un archivo en un blob, tiene que usar una SAS para autorizar el acceso al objeto de origen, incluso si los objetos de origen y destino residen dentro la misma cuenta de almacenamiento.

## <a name="best-practices-when-using-sas"></a>Procedimientos recomendados al usar SAS

Cuando use firmas de acceso compartido en sus aplicaciones, debe tener en cuenta dos posibles riesgos:

- Si se perdió una SAS, cualquier persona que la consiga puede usarla, lo que puede poner en riesgo su cuenta de almacenamiento.
- Si una SAS proporcionada para una aplicación cliente expira y la aplicación no puede recuperar una nueva SAS del servicio, la funcionalidad de la aplicación puede verse afectada.

Las siguientes recomendaciones para el uso de firmas de acceso compartido pueden ayudar a mitigar estos riesgos:

- **Siempre use HTTPS** para crear una SAS o distribuirla. Si se pasa una SAS a través de HTTP y se intercepta, un atacante que realice un ataque de tipo "Man in the middle" puede leer la SAS y, luego, usarla como lo podría hacer el usuario previsto. Esto puede poner en riesgo la información confidencial o permitir que un usuario malintencionado provoque daños en los datos.
- **Use una SAS de delegación de usuarios siempre que sea posible.** Una SAS de delegación de usuarios proporciona más seguridad que una SAS de servicio o a una SAS de cuenta. Una SAS de delegación de usuarios está protegida con credenciales de Azure AD, por lo que no es necesario almacenar la clave de cuenta con el código.
- **Tenga un plan de revocación en vigor para una SAS.** Asegúrese de que está preparado para responder si una SAS está en peligro.
- **Defina una directiva de acceso almacenada para una SAS de servicio.** Las directivas de acceso almacenadas le ofrecen la posibilidad de revocar permisos para una SAS de servicio sin tener que volver a generar las claves de cuenta de almacenamiento. Establezca su expiración en un futuro muy lejano (o infinito) y asegúrese de que se actualiza regularmente para trasladarla a un punto posterior en el tiempo.
- **Use horas de expiración a corto plazo en una SAS ad-hoc, SAS de servicio o SAS de cuenta.** De esta manera, incluso si una SAS está en peligro, es válida solo durante un breve período. Esta práctica es especialmente importante si no puede hacer referencia a una directiva de acceso almacenada. Las expiraciones a corto plazo también limitan la cantidad de datos que puede escribirse en un blob mediante la limitación del tiempo disponible para cargarlos.
- **Haga que los clientes renueven automáticamente la SAS si fuese necesario.** Los clientes deben renovar la SAS correctamente antes de la expiración para que exista tiempo para los reintentos si el servicio que ofrece la SAS no está disponible. Si la SAS se ha creado para usarse en un número reducido de operaciones de corta duración e inmediatas, que se espera que se va a completar dentro del tiempo de expiración determinado, es posible que no sea necesario ese procedimiento, ya que no se espera que la SAS se renueve. Sin embargo, si dispone de un cliente que realice solicitudes de forma rutinaria a través de la SAS, existe la posibilidad de la expiración. La consideración clave es equilibrar la necesidad de que la SAS sea de corta duración (como se indicó anteriormente) con el requisito de garantizar que el cliente solicitan la renovación con la suficiente antelación como para evitar la interrupción debida a la expiración de SAS antes de una renovación correcta.
- **Tenga cuidado con la hora de inicio de la SAS.** Si establece que la hora de inicio de un SAS es la hora actual, es probable que aparezcan errores intermitentes en los primeros minutos debido a que las diferentes máquinas tienen leves variaciones en la hora actual (lo que se conoce como sesgo del reloj). En general, establezca la hora de inicio sea al menos 15 minutos en el pasado. O, no establezca esta opción en absoluto, lo que hará que sea válido inmediatamente en todos los casos. Normalmente se aplica lo mismo a la hora de expiración. Recuerde que debe tener en cuenta hasta 15 minutos de desplazamiento del reloj en cualquier dirección en una solicitud. Para los clientes con una versión REST anterior a 2012-02-12: la duración máxima de una SAS que no hace referencia a una directiva de acceso almacenada es de 1 hora y se producirá un error en las directivas que especifican un período más largo.
- **Tenga cuidado con el formato de fecha y hora de SAS.** Si establece la hora de inicio y la expiración de una SAS, para algunas utilidades (por ejemplo, la utilidad de línea de comandos AzCopy) necesitará que el formato de fecha y hora sea "+%Y-%m-%dT%H:%M:%SZ" y que incluya específicamente los segundos para que funcione con el token de SAS.  
- **Sea específico con el recurso al que se va a tener acceso.** Un procedimiento recomendado de seguridad es proporcionar al usuario los privilegios mínimos necesarios. Si un usuario solo necesita acceso de lectura en una única entidad, concédale acceso de lectura a esa única entidad y no acceso de lectura, escritura o eliminación a todas las entidades. Esto también ayuda a reducir los daños si se pone en peligro una SAS porque la SAS tiene menor menos poder en manos de un atacante.
- **Comprenda que se le hará un cargo en la cuenta por cualquier uso, incluido el realizado con la SAS.** Si proporciona acceso de escritura a un blob, el usuario puede seleccionar cargar un blob de 200 GB. Si le proporciona también acceso de lectura, puede seleccionar descargarlo 10 veces, lo que le supone 2 TB de costos de salida. Proporcione de nuevo permisos limitados para ayudar a mitigar acciones potenciales de usuarios malintencionados. Use una SAS de corta duración para reducir esa amenaza (pero tenga en cuenta el desplazamiento del reloj y la hora final).
- **Valide los datos escritos mediante una SAS.** Cuando una aplicación cliente escribe datos en la cuenta de almacenamiento, tenga en cuenta que pueden existir problemas con esos datos. Si la aplicación requiere que se validen o autoricen los datos antes de que estén listos para usarlos, debe realizar la validación después de que se escriban los datos y antes de que la aplicación los use. Esta práctica también le protege frente a los datos erróneos o malintencionados que se escriben en la cuenta, ya sea mediante un usuario que adquirió correctamente la SAS o un usuario que aproveche una SAS errónea.
- **Sepa cuándo no usar una SAS.** En ocasiones, los riesgos asociados a una operación determinada en la cuenta de almacenamiento superan las ventajas del uso de una SAS. Para esas operaciones, cree un servicio de nivel medio que escriba en la cuenta de almacenamiento después de llevar a cabo una auditoría, autenticación o validación de la regla de negocio. A veces también es más sencillo administrar el acceso de otras formas. Por ejemplo, si desea que todos los blobs de un contenedor puedan leerse públicamente, puede hacer que el contenedor sea público en lugar de proporcionar un SAS a cada cliente para obtener acceso.
- **Use registros de Azure Monitor y Azure Storage para supervisar la aplicación.** Puede hacer uso de registros de Azure Monitor y Storage Analytics para observar cualquier pico en los errores de autorización producidos por la interrupción del servicio del proveedor de SAS o la eliminación involuntaria de una directiva de acceso almacenada. Para obtener más información, consulte [Métricas de Azure Storage en Azure Monitor](monitor-storage.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) y [Registro de Azure Storage Analytics](storage-analytics-logging.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

## <a name="get-started-with-sas"></a>Introducción a SAS

Para comenzar a usar firmas de acceso compartido, consulte los siguientes artículos para cada tipo de SAS.

### <a name="user-delegation-sas"></a>SAS de delegación de usuarios

- [Creación de una SAS de delegación de usuarios para un contenedor o blob con PowerShell](../blobs/storage-blob-user-delegation-sas-create-powershell.md)
- [Creación de una SAS de delegación de usuarios para un contenedor o blob con la CLI de Azure](../blobs/storage-blob-user-delegation-sas-create-cli.md)
- [Creación de una SAS de delegación de usuarios para un contenedor o blob con .NET](../blobs/storage-blob-user-delegation-sas-create-dotnet.md)

### <a name="service-sas"></a>SAS de servicio

- [Create a service SAS for a container or blob with .NET](../blobs/storage-blob-service-sas-create-dotnet.md) (Creación de una SAS de servicio para un contenedor o blob con .NET)

### <a name="account-sas"></a>SAS de cuenta

- [Create an account SAS with .NET](storage-account-sas-create-dotnet.md) (Creación de una SAS de cuenta con .NET)

## <a name="next-steps"></a>Pasos siguientes

- [Delegate access with a shared access signature (REST API)](/rest/api/storageservices/delegate-access-with-shared-access-signature) (Delegación de acceso con una firma de acceso compartido [API REST])
- [Creación de una SAS de delegación de usuario (API de REST)](/rest/api/storageservices/create-user-delegation-sas)
- [Create a service SAS (REST API)](/rest/api/storageservices/create-service-sas) (Creación de una SAS de servicio [API REST])
- [Create an account SAS (REST API)](/rest/api/storageservices/create-account-sas) (Creación de una SAS de cuenta [API REST])
