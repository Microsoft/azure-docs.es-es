---
title: Configuración de claves administradas por el cliente para la cuenta de Azure Batch con Azure Key Vault e Identidad administrada
description: Información sobre cómo cifrar datos de Batch mediante claves
author: pkshultz
ms.topic: how-to
ms.date: 07/17/2020
ms.author: peshultz
ms.openlocfilehash: 35780f915247e88a5de093594b653ddcebdfb06b
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89008886"
---
# <a name="configure-customer-managed-keys-for-your-azure-batch-account-with-azure-key-vault-and-managed-identity"></a>Configuración de claves administradas por el cliente para la cuenta de Azure Batch con Azure Key Vault e Identidad administrada

De forma predeterminada, Azure Batch usa claves administradas por la plataforma para cifrar todos los datos del cliente almacenados en el servicio Azure Batch, como certificados y metadatos de trabajos o tareas. Opcionalmente, puede usar sus propias claves, es decir, claves administradas por el cliente, para cifrar los datos almacenados en Azure Batch.

Las claves que proporcione deben haberse generado en [Azure Key Vault](../key-vault/general/basic-concepts.md), y las cuentas de Batch que quiera configurar con claves administradas por el cliente tienen que estar habilitadas con [Identidad administrada de Azure](../active-directory/managed-identities-azure-resources/overview.md).

> [!IMPORTANT]
> La compatibilidad con las claves administradas por el cliente de Azure Batch se encuentra actualmente en versión preliminar pública para las regiones Oeste de Europa, Norte de Europa, Norte de Suiza, Centro de EE. UU., Centro-sur de EE. UU., Centro-oeste de EE. UU., Este de EE. UU., Este de EE. UU. 2, Oeste de EE. UU. 2, US Gov Virginia y US Gov Arizona.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="create-a-batch-account-with-system-assigned-managed-identity"></a>Creación de una cuenta de Batch con la identidad administrada asignada por el sistema

### <a name="azure-portal"></a>Azure portal

En [Azure Portal](https://portal.azure.com/), al crear cuentas de Batch, seleccione **Asignado por el sistema** en el tipo de identidad en la pestaña **Opciones avanzadas**.

![Nueva cuenta de Batch con el tipo de identidad asignado por el sistema](./media/batch-customer-managed-key/create-batch-account.png)

Una vez creada la cuenta, puede encontrar un GUID único en el campo **Identity principal id** (ID. de entidad de seguridad de identidad) en la sección **Propiedad**. En **Tipo de identidad** se mostrará `SystemAssigned`.

![GUID único en el campo de id. de entidad de seguridad de identidad](./media/batch-customer-managed-key/linked-batch-principal.png)
 
### <a name="azure-cli"></a>Azure CLI

Al crear una nueva cuenta de Batch, especifique `SystemAssigned` para el parámetro `--identity`.

```powershell
resourceGroupName='myResourceGroup'
accountName='mybatchaccount'

az batch account create \
    -n $accountName \
    -g $resourceGroupName \
    --locations regionName='West US 2' \
    --identity 'SystemAssigned'
```

Una vez creada la cuenta, puede comprobar que se ha habilitado la identidad administrada asignada por el sistema en esta cuenta. Asegúrese de anotar el valor `PrincipalId`, ya que será necesario para conceder acceso a esta cuenta de Batch para Key Vault.

```powershell
az batch account show \
    -n $accountName \
    -g $resourceGroupName \
    --query identity
```

> [!NOTE]
> La identidad administrada asignada por el sistema creada en una cuenta de Batch solo se usa para recuperar las claves administradas por el cliente desde Key Vault. Esta identidad no está disponible en grupos de Batch.

## <a name="configure-your-azure-key-vault-instance"></a>Configuración de la instancia de Azure Key Vault

### <a name="create-an-azure-key-vault"></a>Crear una instancia de Azure Key Vault

Al crear una instancia de Azure Key Vault con claves administradas por el cliente para Azure Batch, asegúrese de que las opciones **Eliminación temporal** y **Protección de purga** están habilitadas.

![Pantalla de creación de Key Vault](./media/batch-customer-managed-key/create-key-vault.png)

### <a name="add-an-access-policy-to-your-azure-key-vault-instance"></a>Adición de una directiva de acceso a la instancia de Azure Key Vault

En Azure Portal, una vez creada la instancia de Key Vault, en **Directiva de acceso** bajo **Configuración**, agregue el acceso a la cuenta de Batch mediante la identidad administrada. En **Permisos de clave**, seleccione **Obtener**, **Encapsular clave** y **Desencapsular clave**. 

![Agregar directiva de acceso](./media/batch-customer-managed-key/key-permissions.png)

En el campo **Seleccionar** bajo **Entidad de seguridad**, rellene el valor `principalId` que recuperó anteriormente o el nombre de la cuenta de Batch.

![Escribir id. de entidad de seguridad](./media/batch-customer-managed-key/principal-id.png)

### <a name="generate-a-key-in-azure-key-vault"></a>Generación de una clave en Azure Key Vault

En Azure Portal, vaya a la instancia de Key Vault en la sección **clave** y seleccione **Generar/Importar**. En **Tipo de clave** seleccione `RSA` y en **Tamaño de la clave RSA** elija al menos `2048` bits. Los tipos de clave `EC` no se admiten actualmente como una clave administrada por el cliente en una cuenta de Batch.

![Crear una clave](./media/batch-customer-managed-key/create-key.png)

Una vez creada la clave, haga clic en la clave y la versión actual recién creadas, y copie el **Identificador de clave** en la sección **Propiedades**.  Asegúrese de que en **Operaciones permitidas**, **Encapsular clave** y **Desencapsular clave** están marcadas.

## <a name="enable-customer-managed-keys-on-azure-batch-account"></a>Habilitación de claves administradas por el cliente en una cuenta de Azure Batch

### <a name="azure-portal"></a>Azure portal

En [Azure Portal](https://portal.azure.com/), vaya a la página de la cuenta de Batch. En la sección **Cifrado**, habilite **Clave administrada por el cliente**. Puede usar directamente el identificador de clave, o bien seleccionar el almacén de claves y, a continuación, hacer clic en **Seleccione un almacén de claves y una clave**.

![En Cifrado, habilitar Clave administrada por el cliente](./media/batch-customer-managed-key/encryption-page.png)

### <a name="azure-cli"></a>Azure CLI

Una vez que se haya creado la cuenta de Batch con la identidad administrada asignada por el sistema y se haya concedido el acceso a Key Vault, actualice la cuenta de Batch con la dirección URL `{Key Identifier}` bajo el parámetro `keyVaultProperties`. Establezca también **encryption_key_source** como `Microsoft.KeyVault`.

```powershell
az batch account set \
    -n $accountName \
    -g $resourceGroupName \
    --encryption_key_source Microsoft.KeyVault \
    --encryption_key_identifier {YourKeyIdentifier} 
```

## <a name="update-the-customer-managed-key-version"></a>Actualización de la versión de la clave administrada por el cliente

Al crear una nueva versión de una clave, actualice la cuenta de Batch para que utilice la versión nueva. Siga estos pasos:

1. Vaya a la cuenta de Batch en Azure Portal y muestre las opciones de Configuración de cifrado.
2. Escriba el identificador URI de la nueva versión de la clave. Como alternativa, puede volver a seleccionar el almacén de claves y la clave para actualizar la versión.
3. Guarde los cambios.

También puede usar la CLI de Azure para actualizar la versión.

```powershell
az batch account set \
    -n $accountName \
    -g $resourceGroupName \
    --encryption_key_identifier {YourKeyIdentifierWithNewVersion} 
```
## <a name="use-a-different-key-for-batch-encryption"></a>Uso de una clave diferente para el cifrado de Batch

Para cambiar la clave que se usa para el cifrado de Batch, siga estos pasos:

1. Vaya a la cuenta de Batch y muestre las opciones de Configuración de cifrado.
2. Escriba el identificador URI de la nueva clave. Otra opción es seleccionar el almacén de claves y elegir una clave nueva.
3. Guarde los cambios.

También puede emplear la CLI de Azure para usar una clave diferente.

```powershell
az batch account set \
    -n $accountName \
    -g $resourceGroupName \
    --encryption_key_identifier {YourNewKeyIdentifier} 
```
## <a name="frequently-asked-questions"></a>Preguntas más frecuentes
  * **¿Son compatibles las claves administradas por el cliente con las cuentas existentes de Batch?** No. Las claves administradas por el cliente solo se admiten para las nuevas cuentas de Batch.
  * **¿Puedo seleccionar tamaños de clave RSA mayores de 2048 bits?** Sí, también se admiten los tamaños de clave RSA de `3072` y `4096` bits.
  * **¿Qué operaciones hay disponibles después de la revocación de una clave administrada por el cliente?** La única operación permitida es la eliminación de la cuenta si Batch pierde el acceso a la clave administrada por el cliente.
  * **¿Cómo se debe restaurar el acceso a la cuenta de Batch si se elimina accidentalmente la clave Key Vault?** Dado que la protección de purga y la eliminación temporal están habilitadas, puede restaurar las claves existentes. Para más información, consulte [Recuperación de una instancia de Azure Key Vault](../key-vault/general/soft-delete-cli.md#recovering-a-key-vault).
  * **¿Puedo deshabilitar claves administradas por el cliente?** Puede volver a establecer el tipo de cifrado de la cuenta de Batch en "Clave administrada por Microsoft" en cualquier momento. Después, puede eliminar o cambiar la clave.
  * **¿Cómo puedo girar mis claves?** Las claves administradas por el cliente no se giran automáticamente. Para girar la clave, actualice el identificador de clave con el que está asociada la cuenta.
  * **Después de restaurar el acceso, ¿cuánto tiempo tardará la cuenta de Batch en volver a funcionar?** La cuenta puede tardar hasta 10 minutos en ser accesible después de restaurar el acceso.
  * **Mientras la cuenta de Batch no está disponible ¿qué sucede con mis recursos?** Los grupos que se estén ejecutando al perder el acceso de Batch a las claves administradas por el cliente seguirán ejecutándose. Sin embargo, los nodos pasarán a un estado no disponible y las tareas dejarán de ejecutarse (y se volverán a poner en cola). Una vez restaurado el acceso, los nodos volverán a estar disponibles y se reiniciarán las tareas.
  * **¿Este mecanismo de cifrado se aplica a los discos de VM de un grupo de Batch?** No. En el caso de los grupos de configuración de servicio en la nube, no se aplica ningún cifrado para el sistema operativo y el disco temporal. En el caso de los grupos de configuración de máquina virtual, el sistema operativo y los discos de datos especificados se cifrarán con una clave administrada de la plataforma Microsoft de forma predeterminada. Actualmente, no puede especificar su propia clave para estos discos. Para cifrar el disco temporal de las VM de un grupo de Batch con una clave administrada de la plataforma Microsoft, debe habilitar la propiedad [diskEncryptionConfiguration](/rest/api/batchservice/pool/add#diskencryptionconfiguration) en el grupo de [configuración de máquina virtual](/rest/api/batchservice/pool/add#virtualmachineconfiguration). En entornos altamente confidenciales, se recomienda habilitar el cifrado de disco temporal y evitar el almacenamiento de datos confidenciales en discos de datos y del sistema operativo. Para más información, consulte [Creación de un grupo con el cifrado de disco habilitado](./disk-encryption.md).
  * **¿Está disponible en los nodos de proceso la identidad administrada asignada por el sistema de la cuenta de Batch?** No. Esta identidad administrada se usa actualmente solo para acceder a la clave administrada por el cliente en Azure Key Vault.
  
