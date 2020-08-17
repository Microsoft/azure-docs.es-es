---
title: Cifrado del lado de cliente con Java para el Microsoft Azure Storage | Microsoft Docs
description: La biblioteca de cliente de Azure Storage para Azure ofrece compatibilidad para el cifrado de cliente e integración con Azure Key Vault para obtener una seguridad máxima para sus aplicaciones de Azure Storage.
services: storage
author: tamram
ms.service: storage
ms.devlang: java
ms.topic: article
ms.date: 05/11/2017
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-java
ms.openlocfilehash: 230fac982e19c1106aa5757c9bf1c32d192740d7
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87314516"
---
# <a name="client-side-encryption-and-azure-key-vault-with-java-for-microsoft-azure-storage"></a>Cifrado del lado de cliente y Azure Key Vault para Microsoft Azure Storage
[!INCLUDE [storage-selector-client-side-encryption-include](../../../includes/storage-selector-client-side-encryption-include.md)]

## <a name="overview"></a>Información general
La [Biblioteca de cliente de Azure Storage para Java](https://mvnrepository.com/artifact/com.microsoft.azure/azure-storage) permite tanto el cifrado de datos dentro de las aplicaciones de cliente antes de cargarlos en Azure Storage, como el descifrado de datos mientras estos se descargan al cliente. Asimismo, la biblioteca también admite la integración con [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) para la administración de claves de la cuenta de almacenamiento.

## <a name="encryption-and-decryption-via-the-envelope-technique"></a>Cifrado y descifrado a través de la técnica de sobres
El proceso de cifrado y descifrado sigue la técnica de sobres.  

### <a name="encryption-via-the-envelope-technique"></a>Cifrado a través de la técnica de sobres
El cifrado mediante la técnica de sobres funciona de la siguiente manera:  

1. La biblioteca de cliente de almacenamiento de Azure genera una clave de cifrado de contenido (CEK), que es una clave simétrica de un solo uso.  
2. Los datos de usuario se cifran mediante esta CEK.   
3. Se encapsula la CEK (cifrada) con la clave de cifrado de clave (KEK). La KEK se identifica mediante un identificador de clave y puede ser un par de clave asimétrico o una clave simétrica que puede administrarse de forma local o guardarse en Azure Key Vault.  
   La propia biblioteca de cliente de almacenamiento no tiene nunca acceso a la KEK. La biblioteca invoca el algoritmo de encapsulado de clave proporcionado por Key Vault. Los usuarios pueden elegir utilizar proveedores personalizados para el ajuste y desajuste clave si lo desean.  
4. A continuación, se cargan los datos cifrados en el servicio Azure Storage. La clave encapsulada y algunos metadatos adicionales de cifrado se almacenan como metadatos (en un blob) o se interpolan con los datos cifrados (cola de mensajes y las entidades de tabla).

### <a name="decryption-via-the-envelope-technique"></a>Descifrado a través de la técnica de sobres
El descifrado mediante la técnica de sobres funciona de la siguiente manera:  

1. La biblioteca de cliente asume que el usuario está administrando la clave de cifrado de claves (KEK), ya sea localmente o en almacenes de claves de Azure. El usuario no necesita conocer la clave específica que se usó para el cifrado. En su lugar, se puede configurar y usar una resolución de clave que resuelva distintos identificadores de clave para las claves.  
2. La biblioteca de cliente descarga los datos cifrados junto con cualquier material de cifrado que esté almacenado en el servicio.  
3. A continuación, la clave de cifrado de contenido encapsulado (CEK) se desencapsula (descifra) usando la clave de cifrado de claves (KEK). Aquí nuevamente, la biblioteca de cliente no tiene acceso a la KEK. Simplemente, invoca el algoritmo de desencapsulado personalizado o el del proveedor de Key Vault.  
4. La clave de cifrado de contenido (CEK) se usa entonces para descifrar los datos cifrados del usuario.

## <a name="encryption-mechanism"></a>Mecanismo de cifrado
La biblioteca de cliente de almacenamiento usa [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) para cifrar los datos del usuario. En concreto, emplea el modo [Cipher Block Chaining (CBC)](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher-block_chaining_.28CBC.29) con AES. Cada servicio funciona de forma ligeramente diferente, por lo que describiremos aquí cada uno de ellos.

### <a name="blobs"></a>Datos BLOB
La biblioteca de cliente solo admite actualmente el cifrado de blobs completos. En concreto, se admite el cifrado cuando los usuarios emplean los métodos **upload*** o el método **openOutputStream**. En el caso de las descargas, se admiten tanto las descargas de intervalo como las completas.  

Durante el cifrado, la biblioteca de cliente generará un vector de inicialización (IV) aleatorio de 16 bytes, junto con una clave de cifrado de contenido (CEK) aleatoria de 32 bytes, y realiza el cifrado de sobres de los datos de blob con esta información. Posteriormente, la CEK encapsulada y algunos metadatos de cifrado adicionales se almacenan como metadatos de blob junto con el objeto blob cifrado en el servicio.

> [!WARNING]
> Si está modificando o cargando sus propios metadatos para el blob, deberá asegurarse de que estos metadatos se conserven. Si carga nuevos metadatos sin estos metadatos, la CEK encapsulada, IV y otros metadatos se perderán y el contenido del blob nunca podrá recuperarse nuevamente.
> 
> 

Descargar un blob cifrado implica recuperar el contenido del blob completo mediante los métodos de conveniencia **download**/**openInputStream**. La CEK encapsulada se desencapsula y se utiliza junto con el vector de inicialización (que se almacena como metadatos de blob, en este caso) para devolver los datos descifrados a los usuarios.

Descargar un intervalo arbitrario (métodos**downloadRange**) en el blob cifrado, implica ajustar el intervalo proporcionado por los usuarios para obtener una pequeña cantidad de datos adicionales que puedan usarse para descifrar correctamente el intervalo solicitado.  

Todos los tipos de blobs (blobs en bloques, blobs de anexión) se pueden cifrar y descifrar usando este esquema.

### <a name="queues"></a>Colas
Dado que la cola de mensajes puede tener cualquier formato, la biblioteca de cliente define un formato personalizado que incluye el vector de inicialización (IV) y la clave de cifrado de contenido (CEK) cifrada en el texto del mensaje.  

Durante el cifrado, la biblioteca de cliente genera un vector de inicialización aleatorio de 16 bytes junto con una CEK aleatoria de 32 bytes y realiza el cifrado de sobres del texto del mensaje de cola con esta información. La CEK encapsulada y algunos metadatos de cifrado adicionales se incluyen entonces en el mensaje de cola cifrado. Este mensaje modificado (que se muestra a continuación) se almacena en el servicio.

```
<MessageText>{"EncryptedMessageContents":"6kOu8Rq1C3+M1QO4alKLmWthWXSmHV3mEfxBAgP9QGTU++MKn2uPq3t2UjF1DO6w","EncryptionData":{…}}</MessageText>
```

Durante el descifrado, la clave encapsulada se extrae del mensaje de cola y se desencapsula. El vector de inicialización también se extrae del mensaje de cola y se utiliza junto con la clave desencapsulada para descifrar los datos de mensaje de cola. Tenga en cuenta que los metadatos de cifrado son pequeños (menos de 500 bytes), por lo que, aunque se tienen en cuenta para el límite de 64 KB para un mensaje de la cola, el impacto debería ser fácil de administrar.

### <a name="tables"></a>Tablas
La biblioteca de cliente admite el cifrado de propiedades de entidad para operaciones de insertar y reemplazar.

> [!NOTE]
> Las operaciones de combinación no se admiten actualmente. Puesto que un subconjunto de propiedades puede haberse cifrado previamente con una clave distinta, si simplemente se combinan las nuevas propiedades y se actualizan los metadatos, se producirá una pérdida de datos. Para realizar una combinación es necesario realizar llamadas de servicio adicionales para leer la entidad existente desde el servicio. También puede usar una nueva clave por propiedad. Ninguno de estos procedimientos es adecuado por motivos de rendimiento.
> 
> 

El cifrado de datos de tabla funciona de la siguiente forma:  

1. Los usuarios especifican las propiedades que se van a cifrar.  
2. La biblioteca de cliente genera un vector de inicialización (IV) aleatorio de 16 bytes junto con una clave de cifrado de contenido (CEK) aleatoria de 32 bytes para cada entidad. Después, realiza el cifrado de sobres en las propiedades individuales que se van a cifrar derivando un nuevo vector de inicialización por propiedad. La propiedad de cifrado se almacena como datos binarios.  
3. La CEK encapsulada y algunos metadatos de cifrado adicional se almacenan posteriormente como dos propiedades reservadas adicionales. La primera propiedad reservada (_ClientEncryptionMetadata1) es una propiedad de cadena que contiene la información sobre el vector de inicialización, la versión y la clave encapsulada. La segunda propiedad reservada (_ClientEncryptionMetadata2) es una propiedad binaria que contiene la información sobre las propiedades que se cifran. La información de esta segunda propiedad (_ClientEncryptionMetadata2) está ella misma cifrada.  
4. A causa de estas propiedades reservadas adicionales necesarias para el cifrado, es posible que los usuarios ahora tengan solo 250 propiedades personalizadas en lugar de 252. El tamaño total de la entidad debe ser inferior a 1 MB.  
   
   Tenga en cuenta que solo se pueden cifrar las propiedades de cadena. Si hay que cifrar otros tipos de propiedades, habrá que convertirlas en cadenas. Las cadenas cifradas se almacenan en el servicio como propiedades binarias y se convierten de nuevo en cadenas después del descifrado.
   
   Para las tablas, además de la directiva de cifrado, los usuarios deben especificar las propiedades que se van a cifrar. Para ello, pueden especificar un atributo [Encrypt] \(para las entidades POCO que se derivan de TableEntity) o una resolución de cifrado en las opciones de solicitud. Una resolución de cifrado es un delegado que toma una clave de partición, una clave de fila y un nombre de propiedad y devuelve un valor booleano que indica si se debe cifrar dicha propiedad. Durante el cifrado, la biblioteca de cliente usará esta información para decidir si se debe cifrar una propiedad mientras se escribe en la conexión. El delegado también proporciona la posibilidad de lógica con respecto a la forma de cifrar las propiedades. (Por ejemplo, si el valor es X, hay que cifrar la propiedad A; en caso contrario, hay que cifrar las propiedades A y B). Tenga en cuenta que no es necesario proporcionar esta información para leer o consultar entidades.

### <a name="batch-operations"></a>Operaciones por lotes
En las operaciones por lotes, se usará la misma KEK en todas las filas de esa operación por lotes porque la biblioteca de cliente solo permite un objeto de opciones (y, por lo tanto, una directiva/KEK) por cada operación por lotes. Sin embargo, la biblioteca de cliente generará internamente un nuevo vector de inicialización aleatorio y una CEK aleatoria por cada fila del lote. Los usuarios también pueden optar por cifrar diferentes propiedades para cada operación del lote mediante la definición de este comportamiento en la resolución de cifrado.

### <a name="queries"></a>Consultas
> [!NOTE]
> Dado que las entidades están cifradas, no se pueden ejecutar consultas que filtran por una propiedad cifrada.  Si lo intenta, los resultados serán incorrectos, porque el servicio estaría intentando comparar los datos cifrados con los datos sin cifrar.
> 
> 
> Para realizar operaciones de consulta, debe especificar a una resolución de clave que sea capaz de resolver todas las claves en el conjunto de resultados. Si una entidad incluida en el resultado de la consulta no se puede resolver en un proveedor, la biblioteca de cliente producirá un error. Para cualquier consulta que realice proyecciones del lado servidor, la biblioteca de cliente agregará las propiedades de metadatos de cifrado especiales (_ClientEncryptionMetadata1 y _ClientEncryptionMetadata2) a las columnas seleccionadas de forma predeterminada.

## <a name="azure-key-vault"></a>Azure Key Vault
Azure Key Vault ayuda a proteger claves criptográficas y secretos usados por servicios y aplicaciones en la nube. Con Azure Key Vault, los usuarios pueden cifrar claves y secretos (por ejemplo, claves de autenticación, claves de cuenta de almacenamiento, claves de cifrado de datos, archivos .PFX y contraseñas) usando claves que están protegidas por módulos de seguridad de hardware (HSM). Para obtener más información, consulte [¿Qué es Azure Key Vault?](../../key-vault/general/overview.md)

La biblioteca de cliente de almacenamiento utiliza la biblioteca básica de Key Vault para proporcionar un marco común en Azure para administrar las claves. Los usuarios obtienen también la ventaja adicional de usar la biblioteca de extensiones de Key Vault. La biblioteca de extensiones ofrece funciones útiles para los proveedores de claves en la nube y locales simétricas/RSA, así como para la agregación y el almacenamiento en caché.

### <a name="interface-and-dependencies"></a>Interfaz y dependencias
Hay tres paquetes de Key Vault:  

* azure-keyvault-core contiene IKey e IKeyResolver. Es un paquete pequeño sin dependencias. La biblioteca de cliente de almacenamiento para Java lo define como dependencia.
* azure-keyvault contiene el cliente de REST de Key Vault.  
* azure-keyvault-extensions contiene el código de extensión que incluye implementaciones de algoritmos criptográficos, además de una RSAKey y una SymmetricKey. Depende de los espacios de nombres principales y KeyVault. Proporciona funcionalidad para definir una resolución de agregado (cuando los usuarios desean utilizar varios proveedores de clave) y una resolución de clave de almacenamiento en caché. Aunque la biblioteca de cliente de almacenamiento no depende directamente de este paquete, si los usuarios desean usar Azure Key Vault para almacenar sus claves o utilizar las extensiones de Key Vault para consumir los proveedores de servicios criptográficos locales y en la nube, necesitarán este paquete.  
  
  Key Vault está diseñado para claves maestras de gran valor. Por su parte, los valores de limitación por cada almacén de claves se diseñan teniendo en cuenta este aspecto. Al realizar el cifrado en el lado cliente con Key Vault, el modelo preferido es usar las claves maestras simétricas almacenadas como secretos en Key Vault y almacenadas en caché localmente. Los usuarios deben hacer lo siguiente:  

1. Crear un secreto sin conexión y cargarlo en  Key Vault.  
2. Usar el identificador de base del secreto como un parámetro para resolver la versión actual del secreto para el cifrado y el almacenamiento en caché de esta información localmente. Usar CachingKeyResolver para el almacenamiento en caché (los usuarios no deben implementar su propia lógica de almacenamiento en caché).  
3. Utilizar la resolución de caché como una entrada al crear la directiva de cifrado.
   Puede encontrar más información acerca del uso de Key Vault en los ejemplos de código de cifrado.

## <a name="best-practices"></a>Procedimientos recomendados
La compatibilidad con el cifrado solo está disponible en la biblioteca de cliente de almacenamiento para Java.

> [!IMPORTANT]
> Tenga en cuenta estos puntos importantes al usar el cifrado del lado del cliente:
> 
> * Al leer desde un blob cifrado o escribir en él, utilice comandos de carga completa del blob y comandos de descarga de blobs de intervalo/completos. Evite escribir en un blob cifrado mediante operaciones de protocolo, como Colocar bloque, Colocar lista de bloque, Escribir páginas, Borrar páginas o Anexar bloque; de lo contrario, puede dañar el objeto blob cifrado y que no sea legible.
> * Para las tablas, existe una restricción similar. Tenga cuidado de no actualizar propiedades cifradas sin actualizar los metadatos de cifrado.  
> * Si establece los metadatos en el objeto blob cifrado, puede sobrescribir los metadatos relacionados con el cifrado necesarios para el descifrado, ya que el establecimiento de metadatos no es aditivo. Esto también se aplica a las instantáneas: evite la especificación de metadatos durante la creación de una instantánea de un blob cifrado. Si se deben establecer metadatos, asegúrese de llamar primero al método **downloadAttributes** para así obtener los metadatos de cifrado actuales y evitar las escrituras simultáneas mientras estos se establecen.  
> * Habilite la marca **requireEncryption** en las opciones de solicitud predeterminadas para aquellos usuarios que deban trabajar solo con datos cifrados. Vea a continuación para obtener más información.  
> 
> 

## <a name="client-api--interface"></a>Interfaz/API de cliente
Al crear un objeto de EncryptionPolicy, los usuarios pueden proporcionar solo una clave (implementación de IKey), solo una resolución (implementación de IKeyResolver) o ambas. IKey es el tipo de clave básico que se identifica mediante un identificador de claves y que proporciona la lógica para la encapsulación y desencapsulación. IKeyResolver se utiliza para resolver una clave durante el proceso de descifrado. Define un método ResolveKey que devuelve un IKey concreto (identificador de clave). Esto ofrece a los usuarios la posibilidad de elegir entre varias claves que se administran en varias ubicaciones.

* Para el cifrado, se utiliza siempre la clave. Si no hay clave, se producirá un error.  
* Para el descifrado:  
  
  * La resolución de claves se invoca si se especifica para obtener la clave. Si se especifica la resolución, pero no se proporciona una asignación para el identificador de clave, se produce un error.  
  * Si no se especifica la resolución, pero sí se especifica una clave, la clave se usa si su identificador coincide con el identificador de clave necesario. Si el identificador no coincide, se genera un error.  
    
    Los [ejemplos de cifrado](https://github.com/Azure/azure-storage-net/tree/master/Samples/GettingStarted/EncryptionSamples) muestran un escenario más detallado de un extremo a otro para blobs, colas y tablas, junto con la integración de Key Vault.

### <a name="requireencryption-mode"></a>Modo RequireEncryption
Los usuarios pueden habilitar opcionalmente un modo de operación en el que se deben cifrar todas las cargas y descargas. En este modo, los intentos de cargar datos sin una directiva de cifrado o de descargar datos no cifrados en el servicio generarán un error en el cliente. La marca **requireEncryption** del objeto de opciones de solicitud es la que controla este comportamiento. Si la aplicación va a cifrar todos los objetos almacenados en Azure Storage, puede establecer la propiedad **requireEncryption** en las opciones de solicitud predeterminadas del objeto de cliente de servicio.   

Por ejemplo, use **CloudBlobClient.getDefaultRequestOptions().setRequireEncryption(true)** para solicitar que se realice el cifrado de todas las operaciones de blob llevadas a cabo a través de ese objeto de cliente.

### <a name="blob-service-encryption"></a>Cifrado de Blob service
Cree un objeto **BlobEncryptionPolicy** y configúrelo en las opciones de solicitud (por API o en un nivel de cliente mediante el elemento **DefaultRequestOptions**). Todo lo demás lo controlará la biblioteca de cliente internamente.

```java
// Create the IKey used for encryption.
RsaKey key = new RsaKey("private:key1" /* key identifier */);

// Create the encryption policy to be used for upload and download.
BlobEncryptionPolicy policy = new BlobEncryptionPolicy(key, null);

// Set the encryption policy on the request options.
BlobRequestOptions options = new BlobRequestOptions();
options.setEncryptionPolicy(policy);

// Upload the encrypted contents to the blob.
blob.upload(stream, size, null, options, null);

// Download and decrypt the encrypted contents from the blob.
ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
blob.download(outputStream, null, options, null);
```

### <a name="queue-service-encryption"></a>Cifrado del servicio Cola
Cree un objeto **QueueEncryptionPolicy** y configúrelo en las opciones de solicitud (por API o en el nivel del cliente usando **DefaultRequestOptions**). Todo lo demás lo controlará la biblioteca de cliente internamente.

```java
// Create the IKey used for encryption.
RsaKey key = new RsaKey("private:key1" /* key identifier */);

// Create the encryption policy to be used for upload and download.
QueueEncryptionPolicy policy = new QueueEncryptionPolicy(key, null);

// Add message
QueueRequestOptions options = new QueueRequestOptions();
options.setEncryptionPolicy(policy);

queue.addMessage(message, 0, 0, options, null);

// Retrieve message
CloudQueueMessage retrMessage = queue.retrieveMessage(30, options, null);
```

### <a name="table-service-encryption"></a>Cifrado de Table service
Además de crear una directiva de cifrado y configurarla en las opciones de solicitud, debe especificar un elemento **EncryptionResolver** en **TableRequestOptions** o establecer el atributo [Encrypt] en el captador y establecedor de la entidad.

### <a name="using-the-resolver"></a>Uso de la resolución

```java
// Create the IKey used for encryption.
RsaKey key = new RsaKey("private:key1" /* key identifier */);

// Create the encryption policy to be used for upload and download.
TableEncryptionPolicy policy = new TableEncryptionPolicy(key, null);

TableRequestOptions options = new TableRequestOptions()
options.setEncryptionPolicy(policy);
options.setEncryptionResolver(new EncryptionResolver() {
    public boolean encryptionResolver(String pk, String rk, String key) {
        if (key == "foo")
        {
            return true;
        }
        return false;
    }
});

// Insert Entity
currentTable.execute(TableOperation.insert(ent), options, null);

// Retrieve Entity
// No need to specify an encryption resolver for retrieve
TableRequestOptions retrieveOptions = new TableRequestOptions()
retrieveOptions.setEncryptionPolicy(policy);

TableOperation operation = TableOperation.retrieve(ent.PartitionKey, ent.RowKey, DynamicTableEntity.class);
TableResult result = currentTable.execute(operation, retrieveOptions, null);
```

### <a name="using-attributes"></a>Uso de los atributos
Tal como se mencionó anteriormente, si la entidad implementa el elemento TableEntity, el captador y establecedor de las propiedades se pueden completar con el atributo [Encrypt] en lugar de especificar **EncryptionResolver**.

```java
private string encryptedProperty1;

@Encrypt
public String getEncryptedProperty1 () {
    return this.encryptedProperty1;
}

@Encrypt
public void setEncryptedProperty1(final String encryptedProperty1) {
    this.encryptedProperty1 = encryptedProperty1;
}
```

## <a name="encryption-and-performance"></a>Cifrado y rendimiento

Tenga en cuenta que el cifrado de sus resultados de datos de almacenamiento da lugar a la sobrecarga de rendimiento adicional. Se deben generar la clave de contenido e IV, se debe cifrar el propio contenido y se deben formatear y cargar metadatos adicionales. Esta sobrecarga variará según la cantidad de datos que se cifran. Se recomienda que los clientes prueben siempre sus aplicaciones para obtener un rendimiento durante el desarrollo.

## <a name="next-steps"></a>Pasos siguientes

* Descargue el [paquete Maven de la Biblioteca de cliente de Azure Storage para Java](https://mvnrepository.com/artifact/com.microsoft.azure/azure-storage)  
* Descargue el [Código fuente de la Biblioteca de cliente de Azure Storage para Java desde GitHub](https://github.com/Azure/azure-storage-java)
* Descargue la biblioteca Maven de Azure Key Vault para los paquetes Maven de Java:
  * [principal](https://mvnrepository.com/artifact/com.microsoft.azure/azure-keyvault-core)
  * [cliente](https://mvnrepository.com/artifact/com.microsoft.azure/azure-keyvault)
* Consulte la [documentación de Azure Key Vault](../../key-vault/general/overview.md)
