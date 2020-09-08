---
title: Cifrado en reposo mediante claves administradas por el cliente
titleSuffix: Azure Cognitive Search
description: Complemente el cifrado del lado servidor con índices y mapas de sinónimos de Azure Cognitive Search usando claves que se crean y administran en Azure Key Vault.
manager: nitinme
author: NatiNimni
ms.author: natinimn
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/01/2020
ms.custom: references_regions
ms.openlocfilehash: 2dc7458dd905ff84455927c81b4ea93765d4f5cb
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88928826"
---
# <a name="configure-customer-managed-keys-for-data-encryption-in-azure-cognitive-search"></a>Configuración de claves administradas por el cliente para el cifrado de datos en Azure Cognitive Search

Azure Cognitive Search cifra automáticamente el contenido indexado en reposo con [claves administradas por el servicio](../security/fundamentals/encryption-atrest.md#azure-encryption-at-rest-components). Si se necesita más protección, puede complementar el cifrado predeterminado con un nivel de cifrado adicional con las claves que se crean y administran en Azure Key Vault. Este artículo le guía por los pasos necesarios para configurar el cifrado de CMK.

El cifrado de CMK depende de [Azure Key Vault](../key-vault/general/overview.md). Puede crear sus propias claves de cifrado y almacenarlas en un almacén de claves, o puede usar las API de Azure Key Vault para generar las claves de cifrado. Con Azure Key Vault, también puede auditar el uso de claves si [habilita el registro](../key-vault/general/logging.md).  

El cifrado con claves administradas por el cliente se aplica a los índices individuales o mapas de sinónimos cuando se crean esos objetos y no se especifica en el nivel del servicio de búsqueda. Solo se pueden cifrar los objetos nuevos. No se puede cifrar el contenido que ya existe.

No es necesario que todas las claves estén en el mismo almacén de claves. Un solo servicio de búsqueda puede hospedar varios índices o asignaciones de sinónimos cifrados, cada uno con sus propias claves de cifrado administradas por el cliente, almacenadas en almacenes de claves diferentes. También puede tener índices y mapas de sinónimos en el mismo servicio que no estén cifrados mediante claves administradas por el cliente. 

## <a name="double-encryption"></a>Doble cifrado

En el caso de los servicios creados después del 1 de agosto de 2020 y en regiones específicas, el ámbito del cifrado de CMK incluye discos temporales y se consigue un [doble cifrado completo](search-security-overview.md#double-encryption), disponible actualmente en estas regiones: 

+ Oeste de EE. UU. 2
+ Este de EE. UU.
+ Centro-sur de EE. UU.
+ US Gov - Virginia
+ US Gov: Arizona

Si usa una región diferente o un servicio creado antes del 1 de agosto, el cifrado de CMK se limita solo al disco de datos y se excluyen los discos temporales que usa el servicio.

## <a name="prerequisites"></a>Requisitos previos

En este ejemplo se usan los servicios siguientes. 

+ [Cree un servicio de Azure Cognitive Search](search-create-service-portal.md) o [busque un servicio existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices). 

+ [Cree un recurso de Azure Key Vault](../key-vault/secrets/quick-create-portal.md#create-a-vault) o busque un almacén existente en la misma suscripción que Azure Cognitive Search. Esta característica tiene un requisito de misma suscripción.

+ Se usan [Azure PowerShell](/powershell/azure/) o la [CLI de Azure](/cli/azure/install-azure-cli) para las tareas de configuración.

+ [Postman](search-get-started-postman.md), [Azure PowerShell](./search-get-started-powershell.md) y la versión preliminar del [SDK de .NET](https://aka.ms/search-sdk-preview) se pueden usar para llamar a la API REST que crea índices y asignaciones de sinónimos que incluyen un parámetro de claves de cifrado. En este momento, no se admite en el portal la adición de una clave a índices o mapas de sinónimos.

>[!Note]
> Debido a la naturaleza del cifrado con claves administradas por el cliente, Azure Cognitive Search no podrá recuperar los datos si se elimina la clave de Azure Key Vault. Para evitar la pérdida de datos causada por las eliminaciones accidentales de claves de Key Vault, debe habilitar las opciones de eliminación temporal y de protección de purgas en el almacén de claves. La eliminación temporal está habilitada de forma predeterminada, por lo que solo tendrá problemas si la deshabilitó intencionadamente. La protección de purga no está habilitada de forma predeterminada, pero es necesaria para el cifrado de CMK de Azure Cognitive Search. Para obtener más información, consulte las introducciones a la [eliminación temporal](../key-vault/general/soft-delete-overview.md) y la [protección de purga](../key-vault/general/soft-delete-overview.md#purge-protection).

## <a name="1---enable-key-recovery"></a>1\. Habilitación de la recuperación de claves

El almacén de claves debe tener la **eliminación temporal** y la **protección contra purgas** habilitada. Puede establecer estas características mediante el portal o los siguientes comandos de PowerShell o la CLI de Azure.

### <a name="using-powershell"></a>Usar PowerShell

1. Ejecute `Connect-AzAccount` para configurar las credenciales de Azure.

1. Ejecute el siguiente comando para conectarse al almacén de claves. Para hacerlo, reemplace `<vault_name>` por un nombre válido:

   ```powershell
   $resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "<vault_name>").ResourceId
   ```

1. Azure Key Vault se crea con la eliminación temporal habilitada. Si está deshabilitada en el almacén, ejecute el siguiente comando:

   ```powershell
   $resource.Properties | Add-Member -MemberType NoteProperty -Name "enableSoftDelete" -Value 'true'
   ```

1. Habilitación de la protección de purga:

   ```powershell
   $resource.Properties | Add-Member -MemberType NoteProperty -Name "enablePurgeProtection" -Value 'true'
   ```

1. Guarde las actualizaciones:

   ```powershell
   Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
   ```

### <a name="using-azure-cli"></a>Uso de la CLI de Azure

```azurecli-interactive
az keyvault update -n <vault_name> -g <resource_group> --enable-soft-delete --enable-purge-protection
```

## <a name="2---create-a-new-key"></a>2\. Creación de una nueva clave

Si usa una clave existente para cifrar el contenido de Azure Cognitive Search, omita este paso.

1. [Inicie sesión en Azure Portal](https://portal.azure.com) y abra la página de información general del almacén de claves.

1. Seleccione la configuración de las **claves** de panel de navegación izquierdo y haga clic en **+ Generate/Import** (+Generar/importar).

1. En el panel **Crear una clave**, forme la lista de **Opciones** y elija el método que quiera usar para crear una clave. También puede **generar** una nueva clave, **cargar** una clave existente, o usar **Restaurar copia de seguridad** para seleccionar una copia de seguridad de una clave.

1. Especifique un **nombre** para la clave y, opcionalmente, seleccione otras propiedades clave.

1. Haga clic en el botón **Crear** para iniciar la implementación.

Tome nota del identificador de la clave: se compone del **Uri del valor de clave**, el **nombre de clave** y la **versión de clave**. Necesitará esta información para definir un índice cifrado en Azure Cognitive Search.
 
![Crear una clave del almacén de claves](./media/search-manage-encryption-keys/create-new-key-vault-key.png "Crear una clave del almacén de claves")

## <a name="3---create-a-service-identity"></a>3\. Creación de una identidad de servicio

La asignación de una identidad a su servicio de búsqueda le permite conceder permisos de acceso a Key Vault a su servicio de búsqueda. El servicio de búsqueda usará su identidad para autenticarse con Azure Key Vault.

Azure Cognitive Search admite dos formas de asignar la identidad: una identidad administrada o una aplicación de Azure Active Directory administrada externamente. 

Si es posible, utilice una identidad administrada. Es la forma más sencilla de asignar una identidad al servicio de búsqueda y debería funcionar en la mayoría de los casos. Si está utilizando varias claves para índices y mapas de sinónimos, o si la solución se encuentra en una arquitectura distribuida que descalifica la autenticación basada en identidad, utilice el [enfoque avanzado de Azure Active Directory administrado externamente](#aad-app), que se describe al final de este artículo.

 En general, una identidad administrada permite que el servicio de búsqueda se autentique en Azure Key Vault sin almacenar las credenciales en código. El ciclo de vida de este tipo de identidad administrada está ligado al ciclo de vida del servicio de búsqueda, que solo puede tener una identidad administrada. [Más información sobre identidades administradas](../active-directory/managed-identities-azure-resources/overview.md).

1. [Inicie sesión en Azure Portal](https://portal.azure.com) y abra la página de información general del servicio de búsqueda. 

1. Haga clic en **Identidad** en el panel de navegación izquierdo, cambie su estado a **Activado** y haga clic en **Guardar**.

![Habilitar una identidad administrada](./media/search-enable-msi/enable-identity-portal.png "Habilitar una identidad administrada")

## <a name="4---grant-key-access-permissions"></a>4\. Concesión de permisos de acceso clave

Para habilitar el servicio de búsqueda y usar la clave de Key Vault, deberá conceder al servicio de búsqueda determinados permisos de acceso.

Los permisos de acceso se pueden revocar en cualquier momento. Una vez revocados, cualquier índice o mapa de sinónimos de servicio de búsqueda que utilice ese almacén de claves quedará inutilizable. La restauración de los permisos de acceso al almacén de claves en un momento posterior restaurará el acceso al índice o mapa de sinónimos. Para más información, consulte [Protección del acceso a un almacén de claves](../key-vault/general/secure-your-key-vault.md).

1. [Inicie sesión en Azure Portal](https://portal.azure.com) y abra la página de información general del almacén de claves. 

1. Seleccione la opción **Directivas de acceso** en el panel de navegación izquierdo y haga clic en **+ Agregar nuevo**.

   ![Agregar una nueva directiva de acceso del almacén de claves](./media/search-manage-encryption-keys/add-new-key-vault-access-policy.png "Agregar una nueva directiva de acceso del almacén de claves")

1. Haga clic en **Seleccionar la entidad de seguridad** y seleccione el servicio Azure Cognitive Search. Puede buscarlo por nombre o por el identificador de objeto que se mostró después de habilitar la identidad administrada.

   ![Seleccionar la entidad de seguridad de la directiva de acceso del almacén de claves](./media/search-manage-encryption-keys/select-key-vault-access-policy-principal.png "Seleccionar la entidad de seguridad de la directiva de acceso del almacén de claves")

1. Haga clic en **Permisos de clave** y seleccione *Obtener*, *Desencapsular clave* y *Encapsular clave*. Puede usar la plantilla *Azure Data Lake Storage o Azure Storage* para seleccionar rápidamente los permisos necesarios.

   A Azure Cognitive Search se le deben conceder los siguientes [permisos de acceso](../key-vault/keys/about-keys.md#key-operations):

   * *Obtener*: permite al servicio de búsqueda recuperar las partes públicas de una clave en un almacén de Key Vault
   * *Encapsular clave*: permite que el servicio de búsqueda utilice la clave para proteger la clave de cifrado interno
   * *Desencapsular clave*: permite que el servicio de búsqueda utilice la clave para desencapsular la clave de cifrado interno

   ![Seleccionar los permisos de claves de la directiva de acceso del almacén de claves](./media/search-manage-encryption-keys/select-key-vault-access-policy-key-permissions.png "Seleccionar los permisos de claves de la directiva de acceso del almacén de claves")

1. En **Permisos de secretos**, seleccione *Obtener*.

1. En **Permisos de certificado**, seleccione *Obtener*.

1. Haga clic en **Aceptar** y **guarde** los cambios de directiva de acceso.

> [!Important]
> El contenido cifrado de Azure Cognitive Search está configurado para usar una clave específica de Azure Key Vault con una **versión** concreta. Si cambia la clave o versión, debe actualizarse el índice o el mapa de sinónimos para utilizar la nueva versión de clave **antes de** eliminar la versión de clave anterior. Si no lo hace, el índice o el mapa de sinónimos quedará inutilizable, ya que no podrá descifrar el contenido una vez que se pierda la clave de acceso.   

## <a name="5---encrypt-content"></a>5\. Cifrado del contenido

Para agregar una clave administrada por el cliente en un índice o asignación de sinónimos, debe usar la [API REST de búsqueda](/rest/api/searchservice/) o un SDK. El portal no expone las asignaciones de sinónimos ni las propiedades de cifrado. Cuando se usa una API válida, los índices y los mapas de sinónimos admiten una propiedad **encryptionKey** de nivel superior. 

Con el **URI del almacén de claves**, el **nombre de la clave** y la  **versión de clave** de la clave del almacén de claves, cree una definición de **encryptionKey** como se indica a continuación:

```json
{
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660"
  }
}
```
> [!Note] 
> Ninguno de estos detalles del almacén de claves se considera secreto y se pueden recuperar fácilmente yendo a la página de claves relevante de Azure Key Vault en Azure Portal.

Si utiliza una aplicación de AAD para la autenticación del almacén de Key Vault en lugar de utilizar una identidad administrada, agregue las **credenciales de acceso** de la aplicación de AAD a la clave de cifrado: 
```json
{
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

## <a name="example-index-encryption"></a>Ejemplo: Cifrado del índice
Los detalles de la creación de un índice mediante la API REST se pueden encontrar en [Creación de un índice (API REST de Azure Cognitive Search)](/rest/api/searchservice/create-index), donde la única diferencia es la especificación de los detalles de la clave de cifrado como parte de la definición del índice: 

```json
{
 "name": "hotels",  
 "fields": [
  {"name": "HotelId", "type": "Edm.String", "key": true, "filterable": true},
  {"name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": true, "facetable": false},
  {"name": "Description", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "en.lucene"},
  {"name": "Description_fr", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.lucene"},
  {"name": "Category", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true},
  {"name": "Tags", "type": "Collection(Edm.String)", "searchable": true, "filterable": true, "sortable": false, "facetable": true},
  {"name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": true, "sortable": true, "facetable": true},
  {"name": "LastRenovationDate", "type": "Edm.DateTimeOffset", "filterable": true, "sortable": true, "facetable": true},
  {"name": "Rating", "type": "Edm.Double", "filterable": true, "sortable": true, "facetable": true},
  {"name": "Location", "type": "Edm.GeographyPoint", "filterable": true, "sortable": true},
 ],
 "encryptionKey": {
   "keyVaultUri": "https://demokeyvault.vault.azure.net",
   "keyVaultKeyName": "myEncryptionKey",
   "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660"
 }
}
```
Ahora puede enviar la solicitud de creación del índice y, a continuación, empiece a usar el índice con normalidad.

## <a name="example-synonym-map-encryption"></a>Ejemplo: Cifrado del mapa de sinónimos

Los detalles de la creación de un nuevo mapa de sinónimos mediante la API REST se pueden encontrar en [Creación de un mapa de sinónimos (API REST de Azure Cognitive Search)](/rest/api/searchservice/create-synonym-map), donde la única diferencia es la especificación de los detalles de la clave de cifrado como parte de la definición del mapa de sinónimos: 

```json
{   
  "name" : "synonymmap1",  
  "format" : "solr",  
  "synonyms" : "United States, United States of America, USA\n
  Washington, Wash. => WA",
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660"
  }
}
```
Ahora puede enviar la solicitud de creación del mapa de sinónimos y, a continuación, empiece a usarlo con normalidad.

>[!Important] 
> Aunque **encryptionKey** no se puede agregar a los índices ni a los mapas de sinónimos existentes de Azure Cognitive Search, se puede actualizar si se proporcionan diferentes valores para cualquiera de los tres datos del almacén de claves (por ejemplo, la actualización de la versión de la clave). Al cambiar a una nueva clave o a una nueva versión de clave de Key Vault, el índice o el mapa de sinónimos de Azure Cognitive Search que usen la clave deben actualizarse primero para utilizar la nueva versión o clave **antes** de eliminar las anteriores. Si no lo hace, el índice o el mapa de sinónimos quedarán inutilizables, ya que no podrá descifrar el contenido una vez que se pierda el acceso a la clave.   
> La restauración de los permisos de acceso al almacén de claves en un momento posterior restaurará el acceso al contenido.

## <a name="advanced-use-an-externally-managed-azure-active-directory-application"></a><a name="aad-app"></a> Avanzado: Uso de una aplicación de Azure Active Directory administrada externamente

Cuando una identidad administrada no es posible, puede crear una aplicación de Azure Active Directory con una entidad de seguridad para el servicio de Azure Cognitive Search. En concreto, una identidad administrada no es viable en estas condiciones:

* No puede conceder directamente a su servicio de búsqueda permisos de acceso al almacén de claves (por ejemplo, si el servicio de búsqueda se encuentra en un inquilino de Active Directory diferente al de Azure Key Vault).

* Se requiere un único servicio de búsqueda para hospedar varias asignaciones de sinónimos e índices cifrados, cada una de las cuales utiliza una clave diferente de un almacén de claves diferente, en el que cada almacén de claves debe utilizar una **identidad diferente** para la autenticación. Si no es necesario utilizar una identidad diferente para administrar distintos almacenes de claves, considere la posibilidad de utilizar la opción de identidad administrada anterior.  

Para dar cabida a estas topologías, Azure Cognitive Search admite el uso de aplicaciones de Azure Active Directory (AAD) para la autenticación entre el servicio de búsqueda y Key Vault.    
Para crear una aplicación AAD en el portal:

1. [Crear una aplicación de Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md).

1. [Obtenga la clave de autenticación y el identificador de aplicación](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) como los que se necesitarán para crear un índice cifrado. Los valores que deberá proporcionar incluyen el **identificador de la aplicación** y la **clave de autenticación**.

>[!Important]
> Cuando se decide usar una aplicación de autenticación de AAD en lugar de una identidad administrada, hay que tener en cuenta el hecho de que Azure Cognitive Search no está autorizado para administrar la aplicación de AAD en nombre del usuario, y que es el usuario quien debe administrar la aplicación de AAD, como la rotación periódica de la clave de autenticación de la aplicación.
> Al cambiar una aplicación de AAD o su clave de autenticación, el índice o el mapa de sinónimos de Azure Cognitive Search que usen esa aplicación deben actualizarse primero para utilizar la nueva clave o identificador de aplicación **antes** de eliminar la aplicación anterior o su clave de autorización, y antes de revocar el acceso del almacén de claves a esta.
> Si no lo hace, el índice o el mapa de sinónimos quedarán inutilizables, ya que no podrá descifrar el contenido una vez que se pierda el acceso a la clave.

## <a name="work-with-encrypted-content"></a>Trabajo con contenido cifrado

Con el cifrado de CMK, observará una latencia para la indexación y las consultas debido al trabajo de cifrado o descifrado adicional. Azure Cognitive Search no registra la actividad de cifrado, pero puede supervisar el acceso a la clave mediante el registro del almacén de claves. Se recomienda [habilitar el registro](../key-vault/general/logging.md) como parte de la configuración del almacén de claves.

Se espera que se produzca una rotación de claves con el tiempo. Cuando se realice la rotación de claves, es importante seguir esta secuencia:

1. [Determine la clave que usa un índice o asignación de sinónimos](search-security-get-encryption-keys.md).
1. [Cree una nueva clave en el almacén de claves](../key-vault/keys/quick-create-portal.md), pero deje la clave original disponible.
1. [Actualice las propiedades de encryptionKey](/rest/api/searchservice/update-index) en una asignación de sinónimo o índice para usar los nuevos valores. Solo se pueden actualizar los objetos que se crearon originalmente con esta propiedad para usar un valor diferente.
1. Deshabilite o elimine la clave anterior en el almacén de claves. Supervise el acceso a la clave para comprobar que se está usando la nueva clave.

Por motivos de rendimiento, el servicio de búsqueda almacena la clave en la memoria caché durante varias horas. Si deshabilita o elimina la clave sin proporcionar una nueva, las consultas seguirán funcionando de manera temporal hasta que expire la memoria caché. Sin embargo, una vez que el servicio de búsqueda no pueda descifrar el contenido, recibirá este mensaje: “Acceso prohibido. Puede que la clave de consulta se haya revocado. Vuelva a intentarlo.” 

## <a name="next-steps"></a>Pasos siguientes

Si no está familiarizado con la arquitectura de seguridad de Azure, revise la [documentación sobre seguridad de Azure](../security/index.yml), y en particular, este artículo:

> [!div class="nextstepaction"]
> [Cifrado en reposo de datos](../security/fundamentals/encryption-atrest.md)