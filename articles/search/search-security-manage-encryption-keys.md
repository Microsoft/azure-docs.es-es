---
title: Cifrado en reposo con claves administradas por el cliente en Azure Key Vault (versión preliminar) - Azure Search
description: Cifrado del lado servidor de complemento a través de los índices y las asignaciones de sinónimos en Azure Search a través de las claves que se crean y administran en Azure Key Vault.
author: NatiNimni
manager: jlembicz
ms.author: natinimn
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.custom: ''
ms.openlocfilehash: 9d2cd2a2f4b3143d58d0ef03d67de094ea03303e
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523096"
---
# <a name="azure-search-encryption-using-customer-managed-keys-in-azure-key-vault"></a>Cifrado de Azure Search mediante claves administradas por el cliente en Azure Key Vault

> [!Note]
> Cifrado con claves administradas por el cliente está en versión preliminar y no se ha diseñado para su uso en producción. El [API de REST versión 2019-05-06-Preview](search-api-preview.md) proporciona esta característica. También puede usar la versión de .NET SDK 8.0-preview.
>
> Esta característica no está disponible para los servicios gratuitos. Debe usar un servicio de búsqueda facturables creado a partir de 2019-01-01. No hay ningún soporte técnico del portal en este momento.

De forma predeterminada, Azure Search cifra el contenido de usuario en reposo con [claves administradas del servicio](https://docs.microsoft.com/azure/security/azure-security-encryption-atrest#data-encryption-models). Puede complementar el cifrado predeterminado con un nivel de cifrado adicional con las claves que se crean y administran en Azure Key Vault. Este artículo le guiará a través de los pasos.

Se admite el cifrado de lado servidor mediante la integración con [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview). Puede crear sus propias claves de cifrado y almacenarlas en un almacén de claves, o puede usar las API de Azure Key Vault para generar las claves de cifrado. Con Azure Key Vault, también puede auditar el uso de claves. 

Cifrado con claves administradas por el cliente está configurado en el nivel de asignación de índice o sinónimo cuando esos objetos se crean y no en el nivel de servicio de búsqueda. No se puede cifrar el contenido que ya existe. 

Puede usar claves diferentes de los almacenes de claves diferentes. Esto significa que un servicio de búsqueda solo puede hospedar varios mapas de indexes\synonym cifrados, cada uno se cifran potencialmente con una clave diferente administrada por el cliente, junto con los mapas de indexes\synonym que no se cifran mediante claves administradas por el cliente. 

## <a name="prerequisites"></a>Requisitos previos

Los siguientes servicios se usan en este ejemplo. 

+ [Cree un servicio Azure Search](search-create-service-portal.md) o [busque un servicio existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) en su suscripción actual. Puede usar un servicio gratuito para este tutorial.

+ [Crear un recurso de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/quick-create-portal#create-a-vault) o buscar un almacén existente en su suscripción.

+ [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview) o [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli) se usa para tareas de configuración.

+ [Postman](search-fiddler.md), [Azure PowerShell](search-create-index-rest-api.md) y [SDK de Azure Search](https://aka.ms/search-sdk-preview) puede usarse para llamar a la API de REST de vista previa. No hay ningún portal o la compatibilidad con el SDK de .NET para el cifrado administradas por el cliente en este momento.

## <a name="1---enable-key-recovery"></a>1: habilitar la recuperación de claves

Este paso es opcional pero muy recomendado. Después de crear el recurso de Azure Key Vault, habilitar **eliminación temporal** y **protección purgar** en el almacén de claves seleccionado mediante la ejecución de los siguientes comandos de PowerShell o CLI de Azure:   

```powershell
$resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "<vault_name>").ResourceId

$resource.Properties | Add-Member -MemberType NoteProperty -Name "enableSoftDelete" -Value 'true'

$resource.Properties | Add-Member -MemberType NoteProperty -Name "enablePurgeProtection" -Value 'true'

Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
```

```azurecli-interactive
az keyvault update -n <vault_name> -g <resource_group> --enable-soft-delete --enable-purge-protection
```

>[!Note]
> Debido a la propia naturaleza del cifrado con la característica de claves administradas por el cliente, Azure Search no podrá recuperar los datos si se elimina la clave del almacén de claves de Azure. Para evitar la pérdida de datos causada por la eliminación accidental de claves de Key Vault, se recomienda habilitar eliminación temporal y purga de protección en el almacén de claves seleccionado. Para obtener más información, consulte [eliminación temporal de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-ovw-soft-delete).   

## <a name="2---create-a-new-key"></a>2 - Creación de una nueva clave

Si utiliza una clave existente para cifrar el contenido de Azure Search, omita este paso.

1. [Inicie sesión en Azure portal](https://portal.azure.com) y vaya al panel del almacén de claves.

1. Seleccione el **claves** de panel de navegación izquierdo y haga clic en **+ generar/importar**.

1. En el panel **Crear una clave**, forme la lista de **Opciones** y elija el método que quiera usar para crear una clave. También puede **generar** una nueva clave, **cargar** una clave existente, o usar **Restaurar copia de seguridad** para seleccionar una copia de seguridad de una clave.

1. Escriba un **nombre** para la clave y, opcionalmente, seleccione otras propiedades de clave.

1. Haga clic en el **crear** botón para iniciar la implementación.

Tome nota del identificador de clave: esto está compuesto por el **valor Uri de clave**, el **nombre de clave**y el **versión de la clave**. Necesitará estos para definir un índice cifrado en Azure Search.
 
![Cree una nueva clave de almacén de claves](./media/search-manage-encryption-keys/create-new-key-vault-key.png "crear una nueva clave de almacén de claves")

## <a name="3---create-a-service-identity"></a>3 - Creación de una identidad de servicio

Asignar una identidad para el servicio de búsqueda le permite conceder permisos de acceso de Key Vault para el servicio de búsqueda. El servicio de búsqueda usará su identidad para autenticar con Azure Key vault.

Azure Search admite dos formas de asignación de identidad: una identidad administrada o una aplicación de Azure Active Directory administrado externamente. 

Si es posible, utilice una identidad administrada. Es la manera más sencilla de asignar una identidad para el servicio de búsqueda y debería funcionar en la mayoría de los escenarios. Si usa varias claves para índices y las asignaciones de sinónimos, o si la solución está en una arquitectura distribuida que descalifica autenticación basada en identidades, use el avanzado [enfoque de Azure Active Directory administrado externamente](#aad-app)se describe al final de este artículo.

 En general, una identidad administrada permite que el servicio de búsqueda para autenticarse en Azure Key Vault sin almacenar credenciales en el código. El ciclo de vida de este tipo de identidad administrada está ligado al ciclo de vida del servicio search, que solo puede tener una identidad administrada. [Más información acerca de las identidades administrado](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

1. Para crear una identidad administrada, [inicie sesión en el portal de toAzure](https://portal.azure.com) y abra el panel de servicio de búsqueda. 

1. Haga clic en **identidad** en el panel de navegación izquierdo, cambie su estado a **en**y haga clic en **guardar**.

![Habilitar una identidad administrada](./media/search-enable-msi/enable-identity-portal.png "habilitar una identidad administrada")

## <a name="4---grant-key-access-permissions"></a>4: conceder permisos de acceso de clave

Para habilitar el servicio de búsqueda usar la clave de Key Vault, deberá conceder su búsqueda determinados permisos de acceso del servicio.

En un momento dado, se podrían revocar los permisos de acceso. Una vez que se ha revocado, cualquier búsqueda índice o sinónimo mapa de servicio que usa ese almacén de claves quedará inservible. Restaurar los permisos de acceso de Key vault en un momento posterior se restaurará el acceso de mapa de index\synonym. Para obtener más información, consulte [proteger el acceso a un almacén de claves](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault).

1. [Inicie sesión en Azure portal](https://portal.azure.com) y abra la página de información general del almacén de claves. 

1. Seleccione el **las directivas de acceso** de panel de navegación izquierdo y haga clic en **+ agregar nueva**.

   ![Agregar nueva directiva de acceso del almacén de claves](./media/search-manage-encryption-keys/add-new-key-vault-access-policy.png "agregar nueva directiva de acceso del almacén de claves")

1. Haga clic en **seleccione entidad** y seleccione el servicio Azure Search. Puede buscarlo por nombre o por el identificador de objeto que se mostró después de habilitar la identidad administrada.

   ![Entidad de directiva de acceso seleccione almacén de claves](./media/search-manage-encryption-keys/select-key-vault-access-policy-principal.png "entidad de directiva de acceso seleccione almacén de claves")

1. Haga clic en **permisos de clave** y seleccione *obtener*, *Unwrap Key* y *Wrap Key*. Puede usar el *Azure Data Lake Storage o Azure Storage* plantilla seleccionar rápidamente los permisos necesarios.

   Azure search se le debe conceder los siguientes [permisos de acceso](https://docs.microsoft.com/azure/key-vault/about-keys-secrets-and-certificates#key-operations):

   * *Obtener* -permite que el servicio de búsqueda recuperar las partes de su clave en un almacén de claves públicas
   * *Encapsular clave* -permite que el servicio de búsqueda usar la clave para proteger la clave de cifrado interno
   * *Desencapsular clave* -permite que el servicio de búsqueda usar su clave para desencapsular la clave de cifrado interno

   ![Seleccione los permisos de clave de directiva de acceso de almacén de claves](./media/search-manage-encryption-keys/select-key-vault-access-policy-key-permissions.png "seleccionar permisos de clave de directiva de acceso de almacén de claves")

1. Haga clic en **Aceptar** y **guardar** los cambios de directiva de acceso.

> [!Important]
> El contenido cifrado en Azure search está configurado para usar una clave específica de Azure Key Vault con un valor concreto **versión**. Si cambia la clave o una versión, la asignación de índice o el sinónimo debe actualizarse para usar el nuevo key\version **antes** eliminando el key\version anterior. Si no lo representará el índice o sinónimo inutilizable se asignan, en el no podrá descifrar el contenido una vez que se pierde el acceso de clave.   

## <a name="5---encrypt-content"></a>5 - cifrar contenido

Creación de una asignación de índice o una columna cifrada con claves administradas por el cliente todavía no es posible mediante Azure portal. Utilice la API de REST de Azure Search para crear este tipo una asignación de índice o el sinónimo.

Compatibilidad con un nuevo nivel superior de mapa de índice y sinónimo **encryptionKey** propiedad que se utiliza para especificar la clave. 

Mediante el **Uri de almacén de claves**, **nombre de clave** y **versión de la clave** de su clave de Key vault, podemos crear un **encryptionKey** definición:

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
> Ninguno de estos detalles del almacén de claves se considera secreto y podría recuperarse con facilidad, vaya a la página correspondiente de claves de Azure Key Vault en Azure portal.

Si usa una aplicación de AAD para la autenticación de Key Vault en lugar de usar una identidad administrada, agregar la aplicación de AAD **las credenciales de acceso** a la clave de cifrado: 
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

## <a name="example-index-encryption"></a>Ejemplo: Cifrado de índice
Los detalles de la creación de un nuevo índice a través de la API de REST se podrían encontrar en [Create Index (API de REST de servicio de Azure Search)](https://docs.microsoft.com/rest/api/searchservice/create-index), donde la única diferencia es que los detalles de la clave de cifrado especifica como parte de la definición del índice: 

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
Puede ahora enviar la solicitud de creación de índice y, a continuación, empezar a usar el índice con normalidad.

## <a name="example-synonym-map-encryption"></a>Ejemplo: Cifrado de mapa de sinónimos

Encontrará los detalles de la creación de una nueva asignación de sinónimos a través de la API de REST en [crear asignación de sinónimos (API de REST de servicio de Azure Search)](https://docs.microsoft.com/rest/api/searchservice/create-synonym-map), donde la única diferencia es que los detalles de la clave de cifrado especifica como parte de la definición del mapa de sinónimos: 

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
Puede ahora enviar la solicitud de creación del mapa de sinónimos y, a continuación, empezar a usarla con normalidad.

>[!Important] 
> Mientras **encryptionKey** no puede agregarse a los índices existentes de Azure Search o asignaciones de sinónimos, se puede actualizar al proporcionar valores diferentes para cualquiera de los tres detalles del almacén de claves (por ejemplo, actualizando la versión de la clave). Al cambiar a una nueva clave de Key Vault o una versión nueva de la clave, cualquier asignación de índice o el sinónimo de Azure Search que usa la clave en primer lugar debe actualizarse para usar el nuevo key\version **antes** eliminando el key\version anterior. Lo contrario, se representará el índice o asignación de sinónimos no utilizable, ya no podrá descifrar el acceso al contenido una vez clave se pierde.   
> Restaurar los permisos de acceso de Key vault en un momento posterior restaurará el acceso al contenido.

## <a name="aad-app"></a> Avanzado: Usar una aplicación de Azure Active Directory administrada externamente

Cuando una identidad administrada no es posible, que puede crear una aplicación de Azure Active Directory con la seguridad de una entidad de seguridad para el servicio Azure Search. En concreto, una identidad administrada no es viable en estas condiciones:

* Directamente no se puede conceder la búsqueda en los permisos de acceso de servicio para el almacén de claves (por ejemplo, si el servicio de búsqueda está en un inquilino de Active Directory diferente que el almacén de claves de Azure).

* Un servicio de búsqueda solo es necesario para hospedar varios mapas de indexes\synonym cifrados, cada uno con una clave diferente en un almacén de claves diferentes, donde se debe usar cada almacén de claves **una identidad diferente** para la autenticación. Si no es un requisito con una identidad diferente para administrar los almacenes de claves diferentes, considere el uso de la opción de identidad administrada anterior.  

Para dar cabida a estas topologías, Azure search admite el uso de las aplicaciones de Azure Active Directory (AAD) para la autenticación entre el servicio de búsqueda y Key Vault.    
Para crear una aplicación de AAD en el portal:

1. [Crear una aplicación de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#create-an-azure-active-directory-application).

1. [Obtener la clave de autenticación y el identificador de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-application-id-and-authentication-key) como serán necesarios para crear un índice cifrado. Deberá proporcionar los valores incluyen **Id. de aplicación** y **clave de autenticación**.

>[!Important]
> Al decidir usar una aplicación de AAD de autenticación en lugar de una identidad administrada, tenga en cuenta el hecho de que Azure Search no está autorizado para administrar la aplicación de AAD en su nombre, y depende de usted para administrar la aplicación de AAD, como la rotación periódica de la clave de autenticación de la aplicación.
> Al cambiar una aplicación de AAD o su clave de autenticación, cualquier asignación de índice o el sinónimo de Azure Search que usa esa aplicación en primer lugar debe actualizarse para usar la nueva aplicación ID\key **antes** eliminando la aplicación anterior o su autorización de claves y antes de revocar el acceso de Key Vault a él.
> Lo contrario, se representará el índice o asignación de sinónimos no utilizable, ya no podrá descifrar el acceso al contenido una vez clave se pierde.   

## <a name="next-steps"></a>Pasos siguientes

Si no está familiarizado con la arquitectura de seguridad de Azure, revise el [documentación de Azure Security](https://docs.microsoft.com/azure/security/)y en particular, este artículo:

> [!div class="nextstepaction"]
> [Cifrado en reposo de datos](https://docs.microsoft.com/azure/security/azure-security-encryption-atrest)
