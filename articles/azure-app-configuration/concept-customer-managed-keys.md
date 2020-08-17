---
title: Uso de claves administradas por el cliente para cifrar los datos de configuración
description: Cifrado de los datos de configuración mediante claves administradas por el cliente
author: lisaguthrie
ms.author: lcozzens
ms.date: 02/18/2020
ms.topic: conceptual
ms.service: azure-app-configuration
ms.openlocfilehash: 8942c93b7346613b8cfdc97d9afe09f1c473fb10
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87384878"
---
# <a name="use-customer-managed-keys-to-encrypt-your-app-configuration-data"></a>Uso de claves administradas por el cliente para cifrar los datos de configuración de App Configuration
Azure App Configuration [cifra la información confidencial en reposo](../security/fundamentals/encryption-atrest.md). El uso de claves administradas por el cliente proporciona una mejor protección de los datos al permitirle administrar las claves de cifrado.  Cuando se usa el cifrado de claves administradas, toda la información confidencial de App Configuration se cifra con una clave de Azure Key Vault proporcionada por el usuario.  De esta manera, existe la posibilidad de rotar la clave de cifrado a petición.  También existe la posibilidad de revocar el acceso de Azure App Configuration a información confidencial mediante la revocación del acceso de la instancia de Azure App Configuration a la clave.

## <a name="overview"></a>Información general 
Azure App Configuration cifra la información confidencial en reposo mediante una clave de cifrado AES de 256 bits proporcionada por Microsoft. Cada instancia de App Configuration tiene su propia clave de cifrado que administra el servicio y que se usa para cifrar la información confidencial. La información confidencial incluye los valores que se encuentran en los pares clave-valor.  Cuando está habilitada la funcionalidad de claves administradas por el cliente, App Configuration usa una identidad administrada asignada a la instancia de App Configuration para autenticarse con Azure Active Directory. Luego, la identidad administrada llama a Azure Key Vault y encapsula la clave de cifrado de la instancia de App Configuration. La clave de cifrado encapsulada se almacena y la clave de cifrado desencapsulada se almacena en caché en App Configuration durante una hora. App Configuration actualiza la versión desencapsulada de la clave de cifrado de la instancia de App Configuration cada hora. De esta forma, se garantiza la disponibilidad en condiciones de funcionamiento normales. 

>[!IMPORTANT]
> Si la identidad asignada a la instancia de App Configuration ya no está autorizada para desencapsular la clave de cifrado de la instancia, o si la clave administrada se elimina de forma permanente, ya no será posible descifrar la información confidencial almacenada en la instancia de App Configuration. El uso de la función [eliminación temporal](../key-vault/general/soft-delete-overview.md) de Azure Key Vault reduce la posibilidad de eliminación accidental de la clave de cifrado.

Cuando los usuarios habilitan la funcionalidad de claves administradas por el cliente en su instancia de App Configuration, controlan la capacidad del servicio para acceder a su información confidencial. La clave administrada funciona como clave de cifrado raíz. Un usuario puede revocar el acceso de la instancia de App Configuration a su clave administrada mediante la modificación de su directiva de acceso del almacén de claves. Cuando se revoca este acceso, App Configuration pierde la capacidad de descifrar los datos del usuario en una hora. En este momento, la instancia de App Configuration prohibirá todos los intentos de acceso. Esta situación es recuperable; solo hay que volver a conceder al servicio acceso a la clave administrada.  En una hora, App Configuration podrá descifrar los datos del usuario y funcionar en condiciones normales.

>[!NOTE]
>Todos los datos de Azure App Configuration se almacenan durante 24 horas como máximo en una copia de seguridad aislada. Esto incluye la clave de cifrado desencapsulada. Estos datos no están disponibles de forma inmediata para el servicio o el equipo de servicio. En el caso de una restauración de emergencia, Azure App Configuration se volverá a revocar a partir de los datos de la clave administrada.

## <a name="requirements"></a>Requisitos
Los componentes siguientes son necesarios para habilitar correctamente la funcionalidad de clave administrada por el cliente en Azure App Configuration:
- Instancia de Azure App Configuration de nivel estándar
- Azure Key Vault con las características de eliminación temporal y protección de purga habilitadas
- Una clave RSA o RSA-HSM en Key Vault
    - La clave no debe estar expirada, debe estar habilitada y debe tener activadas las funcionalidades de encapsulado y desencapsulado.

Una vez configurados estos recursos, quedan dos pasos para permitir que Azure App Configuration use la clave de Key Vault:
1. Asignación de una identidad administrada a la instancia de Azure App Configuration
2. Conceda los permisos `GET`, `WRAP` y `UNWRAP` en la directiva de acceso de Key Vault de destino.

## <a name="enable-customer-managed-key-encryption-for-your-azure-app-configuration-instance"></a>Habilitación del cifrado de claves administradas por el cliente en la instancia de Azure App Configuration
Para comenzar, necesitará una instancia de Azure App Configuration configurada correctamente. Si aún no tiene disponible una instancia de App Configuration, siga uno de estos inicios rápidos para configurar una:
- [Creación de una aplicación ASP.NET Core con Azure App Configuration](quickstart-aspnet-core-app.md)
- [Creación de una aplicación .NET Core con Azure App Configuration](quickstart-dotnet-core-app.md)
- [Creación de una aplicación de .NET Framework con Azure App Configuration](quickstart-dotnet-app.md)
- [Creación de una aplicación de Java Spring con Azure App Configuration](quickstart-java-spring-app.md)

>[!TIP]
> Azure Cloud Shell es un shell interactivo gratuito que se puede usar para ejecutar las instrucciones de línea de comandos en este artículo.  Tiene herramientas comunes de Azure preinstaladas, lo que incluye el SDK de .NET Core. Si ha iniciado sesión en su suscripción de Azure, inicie [Azure Cloud Shell](https://shell.azure.com) desde shell.azure.com.  Para más información sobre Azure Cloud Shell, [lea la documentación](../cloud-shell/overview.md).

### <a name="create-and-configure-an-azure-key-vault"></a>Creación y configuración de una instancia de Azure Key Vault
1. Creación de una instancia de Azure Key Vault mediante la CLI de Azure  Tenga en cuenta que tanto `vault-name` como `resource-group-name` los proporciona el usuario y deben ser únicos.  En estos ejemplos, se usará `contoso-vault` y `contoso-resource-group`.

    ```azurecli
    az keyvault create --name contoso-vault --resource-group contoso-resource-group
    ```
    
1. Habilite la eliminación temporal y la protección de purga para el almacén de claves. Sustituya los nombres de almacén de claves (`contoso-vault`) y del grupo de recursos (`contoso-resource-group`) creados en el paso 1.

    ```azurecli
    az keyvault update --name contoso-vault --resource-group contoso-resource-group --enable-purge-protection --enable-soft-delete
    ```
    
1. Cree una clave de Key Vault. Proporcione un valor único de `key-name` para esta clave y sustituya los nombres del almacén de claves (`contoso-vault`) creados en el paso 1. Especifique si prefiere el cifrado `RSA` o `RSA-HSM`.

    ```azurecli
    az keyvault key create --name key-name --kty {RSA or RSA-HSM} --vault-name contoso-vault
    ```
    
    La salida de este comando muestra el identificador de clave ("kid") de la clave generada.  Anótelo, ya que se usará más adelante en este ejercicio.  El identificador de clave tiene el formato: `https://{my key vault}.vault.azure.net/keys/{key-name}/{Key version}`.  El identificador de clave tiene tres componentes importantes:
    1. URI de Key Vault: https://{my key vault}.vault.azure.net
    1. Nombre de la clave de Key Vault: {Key Name}
    1. Versión de la clave de Key Vault: {Key version}

1. Cree una identidad administrada asignada por el sistema mediante la CLI de Azure y sustituya el nombre de la instancia de App Configuration y el grupo de recursos usado en los pasos anteriores. La identidad administrada se usará para acceder a la clave administrada. Se va a usar `contoso-app-config` para ilustrar el nombre de una instancia de App Configuration:
    
    ```azurecli
    az appconfig identity assign --name contoso-app-config --resource-group contoso-resource-group --identities [system]
    ```
    
    La salida de este comando incluye el identificador de entidad de seguridad ("principalId") y el identificador de inquilino ("tenandId") de la identidad asignada por el sistema.  Estos identificadores se usarán para conceder a la identidad acceso a la clave administrada.

    ```json
    {
    "principalId": {Principal Id},
    "tenantId": {Tenant Id},
    "type": "SystemAssigned",
    "userAssignedIdentities": null
    }
    ```

1. La identidad administrada de la instancia de Azure App Configuration necesita acceso a la clave para realizar la validación, el cifrado y el descifrado de claves. El conjunto específico de acciones a las que necesita acceso incluye: `GET`, `WRAP` y `UNWRAP` para las claves.  Para conceder el acceso se necesita el identificador de la entidad de seguridad de la identidad administrada de la instancia de App Configuration. Este valor se obtuvo en el paso anterior. Se muestra a continuación como `contoso-principalId`. Conceda permiso a la clave administrada mediante la línea de comandos:

    ```azurecli
    az keyvault set-policy -n contoso-vault --object-id contoso-principalId --key-permissions get wrapKey unwrapKey
    ```

1. Una vez que la instancia de Azure App Configuration puede acceder a la clave administrada, se puede habilitar la funcionalidad de claves administradas por el cliente en el servicio mediante la CLI de Azure. Recupere las siguientes propiedades registradas durante los pasos de creación de claves: `key name` `key vault URI`.

    ```azurecli
    az appconfig update -g contoso-resource-group -n contoso-app-config --encryption-key-name key-name --encryption-key-version key-version --encryption-key-vault key-vault-Uri
    ```

La instancia de Azure App Configuration ya está configurada para usar una clave administrada por el cliente almacenada en Azure Key Vault.

## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha configurado la instancia de Azure App Configuration para usar una clave administrada por el cliente para el cifrado.  Más información sobre cómo [integrar su servicio con identidades administradas de Azure](howto-integrate-azure-managed-service-identity.md).
