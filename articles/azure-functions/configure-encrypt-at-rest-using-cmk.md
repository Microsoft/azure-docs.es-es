---
title: Cifrado del origen de la aplicación en reposo
description: Cifre los datos de la aplicación en Azure Storage e impleméntelos como un archivo de paquete.
ms.topic: article
ms.date: 03/06/2020
ms.openlocfilehash: 1dd0d11baa16a325a22a501d40e22e5bad6adb21
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87282337"
---
# <a name="encrypt-your-application-data-at-rest-using-customer-managed-keys"></a>Cifrado de los datos en reposo de la aplicación mediante claves administradas por el cliente

Para el cifrado de datos en reposo de la aplicación de funciones se requiere una cuenta de Azure Storage y una instancia de Azure Key Vault. Estos servicios se usan al ejecutar la aplicación desde un paquete de implementación.

  - [Azure Storage proporciona cifrado en reposo](../storage/common/storage-service-encryption.md). Puede usar las claves proporcionadas por el sistema o sus propias claves administradas por el cliente. Aquí es donde se almacenan los datos de la aplicación cuando no se ejecutan en una aplicación de funciones en Azure.
  - La [ejecución desde un paquete de implementación](run-functions-from-deployment-package.md) es una característica de implementación de App Service. Permite implementar el contenido del sitio desde una cuenta de Azure Storage mediante una dirección URL de Firma de acceso compartido (SAS).
  - Las [referencias Key Vault](../app-service/app-service-key-vault-references.md) son una característica de seguridad de App Service. Permite importar secretos en tiempo de ejecución como opciones de configuración de la aplicación. Úsela para cifrar la dirección URL de SAS de su cuenta de Azure Storage.

## <a name="set-up-encryption-at-rest"></a>Configuración del cifrado en reposo

### <a name="create-an-azure-storage-account"></a>Creación de una cuenta de Azure Storage

En primer lugar, [cree una cuenta de Azure Storage](../storage/common/storage-account-create.md) y [cífrela con claves administradas por el cliente](../storage/common/encryption-customer-managed-keys.md). Una vez creada la cuenta de almacenamiento, use el [Explorador de Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md) para cargar archivos de paquete.

A continuación, use el Explorador de Storage para [generar una SAS](../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows#generate-a-sas-in-storage-explorer). 

> [!NOTE]
> Guarde esta dirección URL de SAS, que se usará más adelante para habilitar el acceso seguro del paquete de implementación en tiempo de ejecución.

### <a name="configure-running-from-a-package-from-your-storage-account"></a>Configuración de la ejecución desde un paquete de la cuenta de almacenamiento
  
Cuando haya cargado el archivo en Blob Storage y tenga una dirección URL de SAS para el archivo, establezca la configuración de la aplicación `WEBSITE_RUN_FROM_PACKAGE` en la dirección URL de SAS. En el ejemplo siguiente se usa la CLI de Azure:

```
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings WEBSITE_RUN_FROM_PACKAGE="<your-SAS-URL>"
```

Si agrega esta configuración de la aplicación, se reiniciará la aplicación de funciones. Una vez que reiniciada la aplicación, búsquela y asegúrese de que se inició correctamente con el paquete de implementación. Si la aplicación no se inició correctamente, consulte la [Guía de solución de problemas de ejecución desde paquete](run-functions-from-deployment-package.md#troubleshooting).

### <a name="encrypt-the-application-setting-using-key-vault-references"></a>Cifrado de la configuración de la aplicación mediante referencias de Key Vault

Ahora puede reemplazar el valor de la configuración de la aplicación `WEBSITE_RUN_FROM_PACKAGE` por una referencia Key Vault a la dirección URL codificada con SAS. Al hacerlo, se conserva la dirección URL de SAS cifrada en Key Vault, lo que proporciona un nivel de seguridad adicional.

1. Utilice el comando [`az keyvault create`](/cli/azure/keyvault#az-keyvault-create) siguiente para crear una instancia de Key Vault.       

    ```azurecli    
    az keyvault create --name "Contoso-Vault" --resource-group <group-name> --location eastus    
    ```    

1. Siga [estas instrucciones para conceder a la aplicación acceso](../app-service/app-service-key-vault-references.md#granting-your-app-access-to-key-vault) a su almacén de claves:

1. Use el comando [`az keyvault secret set`](/cli/azure/keyvault/secret#az-keyvault-secret-set) siguiente para agregar la dirección URL externa como un secreto en su almacén de claves:   

    ```azurecli    
    az keyvault secret set --vault-name "Contoso-Vault" --name "external-url" --value "<SAS-URL>"    
    ```    

1.  Use el comando [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set) siguiente para crear la configuración de la aplicación `WEBSITE_RUN_FROM_PACKAGE` con el valor como una referencia de Key Vault a la dirección URL externa:

    ```azurecli    
    az webapp config appsettings set --settings WEBSITE_RUN_FROM_PACKAGE="@Microsoft.KeyVault(SecretUri=https://Contoso-Vault.vault.azure.net/secrets/external-url/<secret-version>"    
    ```

    `<secret-version>` estará en la salida del comando `az keyvault secret set` anterior.

Al actualizar esta configuración de la aplicación, se reiniciará la aplicación de funciones. Una vez que reiniciada la aplicación, búsquela y asegúrese de que se inició correctamente con la referencia de Key Vault.

## <a name="how-to-rotate-the-access-token"></a>Cómo girar el token de acceso

Se recomienda girar periódicamente la clave de SAS de la cuenta de almacenamiento. Para garantizar que la aplicación de funciones no pierda el acceso accidentalmente, debe actualizar también la dirección URL de SAS en Key Vault.

1. Para girar la clave de SAS, navegue a la cuenta de almacenamiento en Azure Portal. En **Configuración** > **Claves de acceso**, haga clic en el icono para girar la clave de SAS.

1. Copie la nueva dirección URL de SAS y use el comando siguiente para establecerla en el almacén de claves:

    ```azurecli    
    az keyvault secret set --vault-name "Contoso-Vault" --name "external-url" --value "<SAS-URL>"    
    ``` 

1. Actualice la referencia del almacén de claves en la configuración de la aplicación a la nueva versión del secreto:

    ```azurecli    
    az webapp config appsettings set --settings WEBSITE_RUN_FROM_PACKAGE="@Microsoft.KeyVault(SecretUri=https://Contoso-Vault.vault.azure.net/secrets/external-url/<secret-version>"    
    ```

    `<secret-version>` estará en la salida del comando `az keyvault secret set` anterior.

## <a name="how-to-revoke-the-function-apps-data-access"></a>Revocación del acceso a datos de la aplicación de funciones

Existen dos métodos para revocar el acceso de la aplicación de funciones a la cuenta de almacenamiento. 

### <a name="rotate-the-sas-key-for-the-azure-storage-account"></a>Rotación de la clave de SAS para la cuenta de almacenamiento de Azure

Si se gira la clave de SAS de la cuenta de almacenamiento, la aplicación de funciones dejará de tener acceso a la cuenta de almacenamiento, pero continuará ejecutándose con la última versión descargada del archivo de paquete. Reinicie la aplicación de funciones para borrar la última versión descargada.

### <a name="remove-the-function-apps-access-to-key-vault"></a>Eliminación del acceso de la aplicación de funciones a Key Vault

Para revocar el acceso de la aplicación de funciones a los datos del sitio, puede deshabilitar el acceso de la aplicación de funciones a Key Vault. Para ello, quite la directiva de acceso de la identidad de la aplicación de funciones. Se trata de la misma identidad que creó anteriormente al configurar las referencias del almacén de claves.

## <a name="summary"></a>Resumen

Los archivos de aplicación ahora se cifran en reposo en la cuenta de almacenamiento. Cuando se inicia la aplicación de funciones, recupera la dirección URL de SAS de Key Vault. Por último, la aplicación de funciones carga los archivos de aplicación de la cuenta de almacenamiento. 

Si necesita revocar el acceso de la aplicación de funciones a su cuenta de almacenamiento, puede revocar el acceso al almacén de claves o girar las claves de la cuenta de almacenamiento, lo que invalidará la dirección URL de SAS.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="is-there-any-additional-charge-for-running-my-function-app-from-the-deployment-package"></a>¿Se aplica algún cargo adicional por ejecutar la aplicación de funciones desde el paquete de implementación?

Solo el costo asociado a la cuenta de almacenamiento de Azure y los cargos de salida aplicables.

### <a name="how-does-running-from-the-deployment-package-affect-my-function-app"></a>¿Cómo afecta la ejecución del paquete de implementación a la aplicación de funciones?

- La ejecución de la aplicación desde el paquete de implementación hace que `wwwroot/` sea de solo lectura. La aplicación recibe un error cuando intenta escribir en este directorio.
- No se admiten los formatos de archivo TAR y GZIP.
- Esta característica no es compatible con la caché local.

## <a name="next-steps"></a>Pasos siguientes

- [Referencias de Key Vault para App Service](../app-service/app-service-key-vault-references.md)
- [Cifrado de Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md)
