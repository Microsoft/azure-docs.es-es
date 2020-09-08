---
title: Administración y búsqueda de datos en Azure Blob Storage con el Índice de blobs (versión preliminar)
description: Aprenda a usar etiquetas del Índice de blobs para categorizar, administrar y consultar a fin de detectar objetos de blobs.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 08/01/2020
ms.service: storage
ms.subservice: common
ms.topic: conceptual
ms.reviewer: hux
ms.openlocfilehash: ed70a05e0a6213ce00a6e0514f0741e8abbaeef9
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690381"
---
# <a name="manage-and-find-data-on-azure-blob-storage-with-blob-index-preview"></a>Administración y búsqueda de datos en Azure Blob Storage con el Índice de blobs (versión preliminar)

A medida que los conjuntos de datos crecen más y más, buscar un objeto específico en un mar de datos puede resultar difícil y frustrante. El Índice de blobs proporciona funcionalidades de administración y detección de datos mediante atributos de etiqueta de índice de clave-valores, que permiten categorizar y buscar objetos en un único contenedor o en todos los contenedores de su cuenta de almacenamiento. Más adelante, cuando cambien los requisitos de los datos, los objetos se pueden recategorizarse de manera dinámica mediante la actualización de sus etiquetas de índice, a la vez que se mantienen en contexto con su organización de contenedores actual. El uso del Índice de blobs puede simplificar el desarrollo, ya que consolida los datos de blobs y los atributos del índice asociados en el mismo servicio; esto le permite crear aplicaciones eficientes y escalables mediante características nativas. 

El Índice de blobs le permite:

- Categorizar dinámicamente los blobs mediante etiquetas de índice de clave-valor para la administración de datos
- Buscar rápidamente de blobs etiquetados específicos en un único contenedor o en toda una cuenta de almacenamiento
- Especificar comportamientos condicionales para las API de blobs en función de la evaluación de etiquetas de índice
- Usar etiquetas de índice para controles avanzados de las características de la plataforma de blobs, como la [administración del ciclo de vida](storage-lifecycle-management-concepts.md)

Considere un escenario en el que tiene millones de blobs en su cuenta de almacenamiento, donde son muchas aplicaciones diferentes las que los escriben y acceden a ellos. Quiere buscar todos los datos relacionados de un solo proyecto, pero no está seguro de cuál es el ámbito, ya que los datos se pueden estar distribuidos entre varios contenedores con distintas convenciones de nomenclatura de blobs. Sin embargo, sabe que sus aplicaciones cargan todos los datos con etiquetas en función del proyecto y de la descripción de la identificación correspondientes. En lugar de buscar a través de millones de blobs y comparar nombres y propiedades, puede simplemente usar `Project = Contoso` como criterio de detección. El Índice de blobs filtrará todos los contenedores en toda la cuenta de almacenamiento para buscar y devolver rápidamente solo el conjunto de 50 blobs de `Project = Contoso`. 

Para empezar a trabajar con ejemplos sobre cómo usar el Índice de blobs, consulte [Uso del Índice de blobs para administrar y buscar datos](storage-blob-index-how-to.md).

## <a name="blob-index-tags-and-data-management"></a>Etiquetas y administración de datos del Índice de blobs

Los prefijos de nombre de contenedores y blobs son una categorización unidimensional para los datos almacenados. El Índice de blobs ahora permite la categorización multidimensional de todos los [tipos de datos de blobs (bloque, anexo o página)](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs) con la aplicación de etiquetas de atributos. Esta categorización multidimensional está indexada de forma nativa y está a su disposición para que pueda consultar y encontrar sus datos rápidamente.

Considere los siguientes cinco blobs en su cuenta de almacenamiento:
>
> container1/transaction.csv  
> container2/campaign.docx  
> photos/bannerphoto.png  
> archives/completed/2019review.pdf  
> logs/2020/01/01/logfile.txt  
>

Actualmente, estos blobs se separan mediante un prefijo de contenedor/carpeta virtual/nombre de blob. Con el Índice de blobs, puede establecer un atributo de etiqueta de índice de `Project = Contoso` en estos cinco blobs para categorizarlos juntos, a la vez que mantiene su organización de prefijos actual. Esto elimina la necesidad de mover los datos al exponer la capacidad de filtrar y buscar datos mediante el índice multidimensional de la plataforma de almacenamiento.

## <a name="setting-blob-index-tags"></a>Configuración de las etiquetas del Índice de blobs

Las etiquetas del Índice de blobs son atributos de clave-valor que se pueden aplicar a objetos nuevos o existentes dentro de la cuenta de almacenamiento. Puede especificar las etiquetas de índice durante el proceso de carga mediante las operaciones PutBlob, PutBlockList o CopyBlob y el encabezado x-mx-tags opcional. Si ya tiene blobs en la cuenta de almacenamiento, puede llamar a SetBlobTags con un documento XML con formato que especifique los atributos de etiqueta del índice de blobs en el cuerpo de la solicitud. 

Analice en los ejemplos siguientes las etiquetas que se pueden establecer

Puede aplicar una única etiqueta al blob para describir cuándo se han terminado de procesar los datos.
>
> "processedDate" = '2020-01-01'
>
Puede aplicar varias etiquetas en el blob para una mayor descripción de los datos.
>
> "Project" = 'Contoso'  
> "Classified" = 'True'  
> "Status" = 'Unprocessed'  
> "Priority" = '01' 
>

Para modificar los atributos de etiqueta del índice existentes, primero debe recuperar los atributos de etiqueta existentes, modificar los atributos de etiqueta y reemplazar por la operación SetBlobTags. Para quitar todas las etiquetas de índice del blob, llame a la operación SetBlobTags sin especificar atributos de etiqueta. Como las etiquetas de índice del blob son un recurso secundario del contenido de los datos del blob, SetBlobTags no modifica ningún contenido subyacente y no cambia la hora de la última modificación o ETag (etiqueta de entidad) del blob. Puede crear o modificar etiquetas de índice para todos los blobs base actuales y versiones anteriores. Sin embargo, las etiquetas de las instantáneas o los blobs eliminados temporalmente no se pueden modificar. 

Los límites siguientes se aplican a las etiquetas del Índice de blobs:
- Cada blob puede tener hasta 10 etiquetas de índice del blob
- Las claves de etiqueta deben tener entre 1 y 128 caracteres.
- Los valores de etiqueta deben tener entre 0 y 256 caracteres.
- Las claves y los valores de etiqueta no distinguen entre mayúsculas y minúsculas.
- Las claves y los valores de etiqueta solo admiten tipos de datos de cadena; todo número, fecha, hora o carácter especial se guardará como cadena.
- Las claves y los valores de etiqueta deben cumplir las siguientes reglas de nomenclatura:
  - Caracteres alfanuméricos: a-z, A-Z, 0-9
  - Caracteres especiales: espacio, más, menos, punto, dos puntos, igual, guion bajo, barra diagonal

## <a name="getting-and-listing-blob-index-tags"></a>Obtención y enumeración de etiquetas de índice del blob

Las etiquetas de índice del blob se almacenan como un subrecurso junto con los datos del blob y se pueden recuperar independientemente del contenido subyacente de los datos del blob. Una vez establecidas, las etiquetas de índice del blob para un único blob se pueden recuperar y revisar inmediatamente con la operación GetBlobTags. La operación ListBlobs con el parámetro `include:tags` también devolverá todos los blobs de un contenedor, junto con las etiquetas de índice del blob aplicadas. 

En el caso de los blobs con al menos una etiqueta de índice de blob, se devuelve x-mx-tag-count en las operaciones ListBlobs, GetBlob y GetBlobProperties, lo que indica que el recuento de etiquetas de índice del blob existe en el blob.

## <a name="finding-data-using-blob-index-tags"></a>Búsqueda de datos mediante etiquetas de índice de blob

Cuando las etiquetas de índice de blobs se han establecido en los blobs, el motor de indexación expone esos atributos de clave y valor en un índice multidimensional. Aunque las etiquetas de índice existan en el blob y puedan recuperarse de inmediato, puede pasar algún tiempo antes de que el índice de blobs se actualice con los atributos de la etiqueta de índice actualizados. Una vez actualizado el índice de blobs, ahora puede aprovechar las funcionalidades de detección y consulta nativas que ofrece el almacenamiento de blobs.

La operación FindBlobsByTags le permite obtener un conjunto devuelto de blobs filtrado cuyas etiquetas de índice coinciden con una expresión de consulta de índice de blobs determinada. El Índice de blobs admite el filtrado entre todos los contenedores de la cuenta de almacenamiento, o puede limitar el filtrado a un único contenedor. Dado que todas las claves y valores de las etiquetas de índice de blobs son cadenas, los operadores relacionales admitidos usan una ordenación lexicográfica en los valores de la etiqueta de índice.

Los criterios siguientes se aplican al filtrado de índices de blobs:
-   Las claves de etiqueta deben ir entre comillas dobles (").
-   Los valores de etiqueta y los nombres de contenedor deben ir entre comillas simples (').
-   El carácter @ solo se permite para filtrar por nombre de un contenedor específico (es decir, @container = 'NombreDeContenedor').
- Los filtros se aplican con la ordenación lexicográfica a las cadenas.
-   Las operaciones de intervalo del mismo lado en la misma clave no son válidas (es decir, "Rank" > '10' AND "Rank" >= '15').
- Al usar REST para crear una expresión de filtro, los caracteres deben estar codificados por URI.

En la tabla siguiente se muestran todos los operadores válidos para FindBlobsByTags:

|  Operator  |  Descripción  | Ejemplo |
|------------|---------------|---------|
|     =      |     Igual     | "Status" = 'In Progress' | 
|     >      |  Mayor que |  "Date" > '2018-06-18' |
|     >=     |  Mayor que o igual a | "Priority" >= '5' | 
|     <      |  Menor que    | "Age" < '32' |
|     <=     |  Menor o igual que  | "Company" <= 'Contoso' |
|    y     |  Y lógico  | "Rank" >= '010' AND "Rank" < '100' |
| @container |  Ámbito de un contenedor específico   | @container = 'videofiles' AND "status" = 'done' |

> [!NOTE]
> Familiarícese con la ordenación lexicográfica al establecer y realizar consultas en etiquetas.
> - Los números se ordenan antes que las letras. Los números se ordenan en función del primer dígito.
> - Las mayúsculas se ordenan antes que las minúsculas.
> - Los símbolos no son estándar. Algunos símbolos se ordenan antes que los valores numéricos. Otros símbolos se ordenan antes o después que las letras.
>

## <a name="conditional-blob-operations-with-blob-index-tags"></a>Operaciones de blob condicionales con etiquetas del índice de blobs
En las versiones de REST 2019-10-10 y posteriores, la mayoría de las [API de Blob service](https://docs.microsoft.com/rest/api/storageservices/operations-on-blobs) ahora admiten un encabezado condicional, x-ms-if-tags, de modo que la operación solo se completará correctamente si se cumple la condición especificada del índice de blobs. Si la condición no se cumple, obtendrá `error 412: The condition specified using HTTP conditional header(s) is not met`.

El encabezado x-mx-if-tags puede combinarse con los demás encabezados condicionales HTTP existentes (If-Match, If-None-Match, etc.).  Si en una solicitud se proporcionan varios encabezados condicionales, todos deben evaluarse como verdaderos para que la operación se complete correctamente.  Todos los encabezados condicionales se combinan eficazmente con AND lógico. 

En la tabla siguiente se muestran todos los operadores válidos para las operaciones condicionales:

|  Operator  |  Descripción  | Ejemplo |
|------------|---------------|---------|
|     =      |     Igual     | "Status" = 'In Progress' |
|     <>     |   No igual   | "Status" <> 'Done'  | 
|     >      |  Mayor que |  "Date" > '2018-06-18' |
|     >=     |  Mayor que o igual a | "Priority" >= '5' | 
|     <      |  Menor que    | "Age" < '32' |
|     <=     |  Menor o igual que  | "Company" <= 'Contoso' |
|    y     |  Y lógico  | "Rank" >= '010' AND "Rank" < '100' |
|     O BIEN     |  OR lógico   | "Status" = 'Done' OR "Priority" >= '05' |

> [!NOTE]
> Hay dos operadores adicionales, Not Equal y OR lógico, que se permiten en el encabezado condicional x-mx-if-tags condicional para la operación de blobs, pero no existen en la operación FindBlobsByTags.

## <a name="platform-integrations-with-blob-index-tags"></a>Integraciones de la plataforma con etiquetas del Índice de blobs

Las etiquetas del Índice de blobs no solo le ayudan a categorizar, administrar y buscar en los datos de blobs, sino que también proporcionan integraciones con otras características de Blob service, como la [administración del ciclo de vida](storage-lifecycle-management-concepts.md). 

### <a name="lifecycle-management"></a>Administración del ciclo de vida

Con el nuevo blobIndexMatch como filtro de reglas en la administración del ciclo de vida, puede mover los datos a los niveles de acceso esporádico o eliminar los datos en función de las etiquetas de índice aplicadas a los blobs. Esto le permite obtener una mayor granularidad en las reglas y solo mover o eliminar datos si coinciden con los criterios de etiquetas especificados.

Puede establecer una coincidencia del índice de blobs como un conjunto de filtros independiente en una regla del ciclo de vida para aplicar acciones a los datos etiquetados. O bien, puede combinar tanto una coincidencia de prefijo como una coincidencia de índice de blobs para que coincidan con conjuntos de datos más específicos. La aplicación de varios filtros a una regla del ciclo de vida se trata como una operación AND lógica, de modo que la acción solo se aplicará si se cumplen todos los criterios de filtro. 

El siguiente ejemplo de regla de administración del ciclo de vida se aplica a los blobs en bloques en el contenedor "videofiles" y en los blobs de niveles para archivar el almacenamiento solo si los datos coinciden con los criterios de la etiqueta de índice del blob de ```"Status" = 'Processed' AND "Source" == 'RAW'```.

# <a name="portal"></a>[Portal](#tab/azure-portal)
![Ejemplo de regla de coincidencia del Índice de blobs para la administración del ciclo de vida en Azure Portal](media/storage-blob-index-concepts/blob-index-lifecycle-management-example.png)
   
# <a name="json"></a>[JSON](#tab/json)
```json
{
    "rules": [
        {
            "enabled": true,
            "name": "ArchiveProcessedSourceVideos",
            "type": "Lifecycle",
            "definition": {
                "actions": {
                    "baseBlob": {
                        "tierToArchive": {
                            "daysAfterModificationGreaterThan": 0
                        }
                    }
                },
                "filters": {
                    "blobIndexMatch": [
                        {
                            "name": "Status",
                            "op": "==",
                            "value": "Processed"
                        },
                        {
                            "name": "Source",
                            "op": "==",
                            "value": "RAW"
                        }
                    ],
                    "blobTypes": [
                        "blockBlob"
                    ],
                    "prefixMatch": [
                        "videofiles/"
                    ]
                }
            }
        }
    ]
}
```
---

## <a name="permissions-and-authorization"></a>Permisos y autorización

Puede autorizar el acceso al Índice de blobs mediante uno de los métodos siguientes:

- Control de acceso basado en roles (RBAC) para conceder permisos a una entidad de seguridad de Azure Active Directory (Azure AD). Microsoft recomienda usar Azure AD para mayor seguridad y facilidad de uso. Para obtener más información sobre el uso de Azure AD con las operaciones de blobs, consulte [Autorización del acceso a los blobs y las colas con Azure Active Directory](../common/storage-auth-aad.md).
- Firma de acceso compartido (SAS) para delegar el acceso al índice de blobs. Para obtener más información sobre las firmas de acceso compartido, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../common/storage-sas-overview.md).
- Claves de acceso de cuenta para autorizar las operaciones con Clave compartida. Para más información, consulte el artículo sobre la [Autorización con clave compartida](/rest/api/storageservices/authorize-with-shared-key).

Las etiquetas del Índice de blobs son un subrecurso de los datos de blobs. Un usuario con permisos o un token de SAS para leer o escribir blobs puede no tener acceso a las etiquetas de índice de blobs. 

### <a name="role-based-access-control"></a>Control de acceso basado en roles 
A los autores de llamadas que usan una [identidad de AAD](../common/storage-auth-aad.md) se les pueden conceder los permisos siguientes para operar con etiquetas de índice de blobs. 

|   Operaciones con blobs   |  Acción de RBAC   |
|---------------------|----------------|
| Buscar blobs por etiquetas  | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/filter/action |
| Establecer etiquetas de blobs         | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write | 
| Obtener etiquetas de blobs         | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/read |

Se requieren permisos adicionales independientes de los datos de blobs subyacentes para operar con las etiquetas. Se concederán los tres permisos al rol de colaborador de datos de Storage Blob. Al lector de datos de Storage Blob se le concederán solo los permisos Buscar blobs por etiquetas y Obtener etiquetas de blobs.

### <a name="sas-permissions"></a>Permisos de SAS 
A los autores de llamadas que usan una [firma de acceso compartido (SAS)](../common/storage-sas-overview.md) se les puede conceder permisos limitados para operar con etiquetas de blobs.
#### <a name="blob-sas"></a>SAS de blobs
Se pueden conceder los siguientes permisos en una SAS de Blob service para permitir el acceso a las etiquetas de índice de blobs. Los permisos de lectura y escritura de blobs por sí solos no son suficientes para permitir la lectura o escritura de sus etiquetas de índice.

|  Permiso  |  Símbolo de URI  | Operaciones permitidas |
|--------------|--------------|--------------------|
|  Etiquetas de índice  |      t         | Obtener y establecer etiquetas de índice de blob para un blob |

#### <a name="container-sas"></a>SAS de contenedor
Se pueden conceder los siguientes permisos en una SAS de servicio de contenedor para permitir el filtrado de las etiquetas de blobs.  El permiso de lista de blobs no es suficiente para permitir el filtrado de blobs por sus etiquetas de índice.

|  Permiso  |  Símbolo de URI  | Operaciones permitidas |
|--------------|--------------|--------------------|
| Etiquetas de índice     |      f       | Búsqueda de blobs con etiquetas de índice de blobs | 

## <a name="choosing-between-metadata-and-blob-index-tags"></a>Elección entre metadatos y etiquetas del Índice de blobs 
Tanto los metadatos como las etiquetas del Índice de blobs dan la capacidad de almacenar propiedades arbitrarias de clave-valor definidas por el usuario junto con un recurso de blob. Ambos se pueden recuperar y establecer directamente, sin devolver ni modificar el contenido del blob. Es posible utilizar tanto los metadatos como las etiquetas de índice.

Sin embargo, solo las etiquetas del Índice de blobs se indexan automáticamente y se pueden consultar mediante el Blob service nativo. Los metadatos no se pueden indexar y consultar de forma nativa, a menos que se use un servicio independiente, como [Azure Search](../../search/search-blob-ai-integration.md). Las etiquetas del Índice de blobs también tienen permisos adicionales de lectura, filtrado y escritura que son independientes de los datos del blob subyacentes. Los metadatos usan los mismos permisos que el blob y se devuelven como encabezados HTTP en las operaciones GetBlob o GetBlobProperties. Las etiquetas del Índice de blobs se cifran en reposo mediante una [clave administrada por Microsoft](../common/storage-service-encryption.md), mientras que los metadatos se cifran en reposo con la misma clave de cifrado especificada para los datos del blob. 

En la tabla siguiente se resumen las diferencias entre los metadatos y las etiquetas del Índice de blobs:

|              |   Metadatos   |   Etiquetas del Índice de blobs  |
|--------------|--------------|--------------------|
| **Límites**         | Sin límite numérico; 8 KB en total; no distingue mayúsculas de minúsculas | 10 etiquetas por blob máx.; 768 bytes por etiqueta; distingue mayúsculas de minúsculas |
| **Actualizaciones**      | No se permiten en el nivel de archivo; SetBlobMetadata reemplaza todos los metadatos existentes; SetBlobMetadata cambia la hora de la última modificación del blob. | Se permiten en todos los niveles de acceso; SetBlobTags reemplaza todas las etiquetas existentes; SetBlobTags no cambia la hora de la última modificación del blob. |
| **Storage**        | Se almacenan con los datos del blob |  Subrecurso a los datos del blob | 
| **Indexación y consulta** | No disponible de forma nativa; se debe usar un servicio independiente, como Azure Search | Sí, funcionalidades nativas de indexación y consulta integradas en el almacenamiento de blobs |
| **Cifrado** | Cifrado en reposo con la misma clave de cifrado que se usa para los datos del blob |  Cifrado en reposo con una clave de cifrado administrada por Microsoft |
| **Precios**   | El tamaño de los metadatos se incluye en los costos de almacenamiento de un blob |    Costo fijo por etiqueta de índice | 
| **Respuesta del encabezado** | Los metadatos se devuelven como encabezados en GetBlob y GetBlobProperties | TagCount se devuelve en GetBlob o GetBlobProperties; las etiquetas se devuelven solo en GetBlobTags y ListBlobs |
| **Permisos**  |    Los permisos de lectura o escritura para los datos del blob se extienden a los metadatos |    Se requieren permisos adicionales para leer, filtrar o escribir etiquetas |

## <a name="pricing"></a>Precios
Los precios del Índice de blobs se encuentran actualmente en versión preliminar pública y están sujetos a cambios para disponibilidad general. A los clientes se les cobra el número total de etiquetas de índice de blobs dentro de una cuenta de almacenamiento, promediado a lo largo del mes. El motor de indexación no tiene ningún costo. Las solicitudes a SetBlobTags, GetBlobTags y FindBlobsByTags se cobran de acuerdo con sus tipos de operación respectivos. Para obtener más información, consulte [Precios de los blobs en bloques](https://azure.microsoft.com/pricing/details/storage/blobs/).

## <a name="regional-availability-and-storage-account-support"></a>Disponibilidad región y compatibilidad para cuentas de almacenamiento

El Índice de blobs solo está disponible actualmente en cuentas De uso general v2 (GPv2) con un espacio de nombres jerárquico (SNP) deshabilitado. No se admiten las cuentas De uso general (GPv1), pero puede actualizar cualquier cuenta GPv1 a una cuenta GPv2. Para más información sobre las cuentas de almacenamiento, vea [Introducción a las cuentas de Azure Storage](../common/storage-account-overview.md).

El Índice de blobs, en versión preliminar pública, solo está disponible en las siguientes regiones seleccionadas:
- Centro de Canadá
- Este de Canadá
- Centro de Francia
- Sur de Francia

Para empezar, consulte [Uso del Índice de blobs para administrar y buscar datos](storage-blob-index-how-to.md).

> [!IMPORTANT]
> Consulte la sección de condiciones de este artículo. Para inscribirse en la versión preliminar, consulte la sección Registro de la suscripción de este artículo. Debe registrar la suscripción para poder usar el Índice de blobs en las cuentas de almacenamiento.

### <a name="register-your-subscription-preview"></a>Registro de la suscripción (versión preliminar)
Dado que el Índice de blobs solo está en versión preliminar pública, deberá registrar su suscripción para poder usar la característica. Para enviar una solicitud, ejecute los siguientes comandos de PowerShell o de la CLI.

#### <a name="register-by-using-powershell"></a>Registro mediante PowerShell
```powershell
Register-AzProviderFeature -FeatureName BlobIndex -ProviderNamespace Microsoft.Storage
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

#### <a name="register-by-using-azure-cli"></a>Registro mediante la CLI de Azure
```azurecli
az feature register --namespace Microsoft.Storage --name BlobIndex
az provider register --namespace 'Microsoft.Storage'
```

## <a name="conditions-and-known-issues-preview"></a>Condiciones y problemas conocidos (versión preliminar)
En esta sección se describen los problemas conocidos y las condiciones de la versión preliminar pública actual del Índice de blobs. Como sucede con la mayoría de las versiones preliminares, esta característica no debe usarse para cargas de trabajo de producción hasta que se lance la disponibilidad general, ya que los comportamientos pueden cambiar.

-   En el caso de la versión preliminar, primero debe registrar la suscripción para poder usar el Índice de blobs en la cuenta de almacenamiento en las regiones de versión preliminar.
-   Actualmente solo las cuentas GPv2 se admiten en la versión preliminar. Las cuentas de Blob, BlockBlobStorage y DataLake Gen2 compatibles habilitadas para HNS no se admiten actualmente con el Índice de blobs. No se admitirán las cuentas GPv1.
-   La carga de blobs en páginas con etiquetas de índice actualmente no conserva las etiquetas. Debe establecer las etiquetas después de cargar un blob en páginas.
-   Cuando el filtrado se limita a un único contenedor, @container solo se puede pasar si todas las etiquetas de índice de la expresión de filtro son comprobaciones de igualdad (clave=valor). 
-   Al usar el operador de intervalo con la condición AND, solo puede especificar el mismo nombre de clave de etiqueta de índice (Age > ‘013’ AND Age < ‘100’).
-   El control de versiones y el Índice de blobs no se admiten actualmente. Las etiquetas del Índice de blobs se conservan para las versiones, pero actualmente no se pasan al motor de índices de blobs.
-   Actualmente no se admite la conmutación por error de cuentas. Es posible que el índice de blobs no se actualice correctamente después de la conmutación por error.
-   La administración del ciclo de vida actualmente solo admite comprobaciones de igualdad con Coincidencia del índice de blobs.
-   CopyBlob no copia las etiquetas de índice de blobs del blob de origen en el nuevo blob de destino. Puede especificar las etiquetas que desea que se apliquen al blob de destino durante la operación de copia. 
- El elemento CopyBlob (copia asincrónica) de otra cuenta de almacenamiento con etiquetas aplicadas en el blob de destino hace que el motor de índices de blobs no devuelva el blob y sus etiquetas en el conjunto de filtros. Se recomienda usar CopyBlob desde la dirección URL (copia de sincronización) de forma provisional.
-   Las etiquetas se conservan al crear la instantánea; sin embargo, la promoción de una instantánea no se admite actualmente y puede dar lugar a un conjunto de etiquetas vacío.

## <a name="faq"></a>Preguntas más frecuentes

### <a name="can-blob-index-help-me-filter-and-query-content-inside-my-blobs"></a>¿Puede el Índice de blobs ayudarme a filtrar y consultar el contenido dentro de mis blobs? 
No, las etiquetas del Índice de blobs pueden ayudarle a encontrar los blobs que está buscando. Si tiene que buscar dentro de los blobs, use Aceleración de consultas o Azure Search.

### <a name="are-there-any-special-considerations-regarding-blob-index-tag-values"></a>¿Existen consideraciones especiales con respecto a los valores de etiqueta del Índice de blobs?
Las etiquetas del Índice de blobs solo admiten tipos de datos de cadena y las consultas devuelven resultados con ordenación lexicográfica. En el caso de los números, se recomienda rellenar el número con ceros a la izquierda. Para las fechas y horas, se recomienda almacenar como formato compatible con ISO 8601.

### <a name="are-blob-index-tags-and-azure-resource-manager-tags-related"></a>¿Se relacionan las etiquetas del Índice de blobs y las etiquetas de Azure Resource Manager?
No, las etiquetas de Azure Resource Manager ayudan a organizar los recursos del plano de control, como suscripciones, grupos de recursos y cuentas de almacenamiento. Las etiquetas del Índice de blobs permiten la administración y detección de objetos en recursos del plano de datos, como blobs dentro de una cuenta de almacenamiento.

## <a name="next-steps"></a>Pasos siguientes

Para ver un ejemplo de cómo se usan los índices de blob, consulte [Uso del índice de blobs para administrar y buscar datos](storage-blob-index-how-to.md).

Obtenga información sobre la [administración del ciclo de vida](storage-lifecycle-management-concepts.md) y establezca una regla con coincidencia de índice de blobs.

