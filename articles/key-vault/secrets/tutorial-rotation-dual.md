---
title: Tutorial de rotación para recursos con dos conjuntos de credenciales
description: Use este tutorial para aprender a automatizar la rotación de secretos para los recursos que usan dos conjuntos de credenciales de autenticación.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: rotation
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 06/22/2020
ms.author: jalichwa
ms.openlocfilehash: 0d2ee8fbcb71d8703702f2c72e0bf629563667b9
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87542202"
---
# <a name="automate-the-rotation-of-a-secret-for-resources-with-two-sets-of-authentication-credentials"></a>Automatización de la rotación de un secreto para recursos con dos conjuntos de credenciales de autenticación

Aunque la mejor forma de realizar la autenticación en los servicios de Azure es mediante una [identidad administrada](../general/managed-identity.md), hay algunos escenarios en los que no es posible. En esos casos, se usan claves de acceso o contraseñas. Las claves de acceso y las contraseñas se deben rotar con frecuencia.

En este tutorial se muestra cómo automatizar la rotación periódica de secretos de bases de datos y servicios que usan dos conjuntos de credenciales de autenticación. Específicamente, en este tutorial se rotan las claves de cuenta de Azure Storage almacenadas en Azure Key Vault como secretos mediante el uso de una función desencadenada por una notificación de Azure Event Grid :

> [!NOTE]
> las claves de la cuenta de almacenamiento se pueden administrar automáticamente en Key Vault; para ello, se proporcionan tokens de firma de acceso compartido para el acceso delegado a la cuenta de almacenamiento. Hay servicios que requieren una cadena de conexión de la cuenta de almacenamiento con la clave de acceso y, para ese escenario, se recomienda esta solución.

![Diagrama de la solución de rotación](../media/secrets/rotation-dual/rotation-diagram.png)

En la solución anterior, Azure Key Vault almacena las claves de acceso individuales de la cuenta de almacenamiento como versiones del mismo secreto y se alternan la clave principal y la secundaria en versiones posteriores. A medida que se almacena una clave de acceso en la última versión del secreto, la clave alternativa se regenera y se agrega a Key Vault como la versión nueva y más reciente del secreto. Esa solución proporciona a las aplicaciones un ciclo de rotación completo para actualizar a la clave regenerada más reciente. 

1. Treinta días antes de la fecha de expiración de un secreto, Key Vault publica un evento de "expiración cercana" en Event Grid.
1. Event Grid comprueba las suscripciones del evento y usa HTTP POST para llamar al punto de conexión de la aplicación de funciones suscrita al evento.
1. La aplicación de funciones identifica una clave alternativa (distinta de la más reciente) y llama a la cuenta de almacenamiento para regenerarla.
1. La aplicación de funciones agrega la nueva clave regenerada a Azure Key Vault como una nueva versión del secreto.

## <a name="prerequisites"></a>Requisitos previos
* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Azure Key Vault
* Dos cuentas de Azure Storage

Si no tiene una cuenta de almacenamiento y un almacén de claves existentes, puede usar el vínculo de implementación siguiente:

[![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjlichwa%2FKeyVault-Rotation-StorageAccountKey-PowerShell%2Fmaster%2Farm-templates%2FInitial-Setup%2Fazuredeploy.json)

1. En **Grupo de recursos**, seleccione **Crear nuevo**. Asigne al grupo el nombre **akvrotation** y haga clic en **Aceptar**.
1. Seleccione **Revisar y crear**.
1. Seleccione **Crear**

    ![Crear un grupo de recursos](../media/secrets/rotation-dual/dual-rotation-1.png)

Ahora tendrá un almacén de claves y dos cuentas de almacenamiento. Puede comprobar esta configuración en la CLI de Azure mediante la ejecución del comando siguiente:

```azurecli
az resource list -o table -g akvrotation
```

El resultado tendrá un aspecto similar a la salida siguiente:

```console
Name                     ResourceGroup         Location    Type                               Status
-----------------------  --------------------  ----------  ---------------------------------  --------
akvrotation-kv         akvrotation      eastus      Microsoft.KeyVault/vaults
akvrotationstorage     akvrotation      eastus      Microsoft.Storage/storageAccounts
akvrotationstorage2    akvrotation      eastus      Microsoft.Storage/storageAccounts
```

## <a name="create-and-deploy-storage-account-key-rotation-function"></a>Creación e implementación de la función de rotación de las claves de la cuenta de almacenamiento

A continuación, cree una aplicación de funciones con una identidad administrada por el sistema, además de los otros componentes necesarios e implemente las funciones de rotación de la clave de la cuenta de almacenamiento.

Las aplicación de funciones para la rotación requiere estos componentes y configuración:
- Un plan de Azure App Service
- Una cuenta de almacenamiento necesaria para la administración del desencadenador de la aplicación de funciones
- Una directiva de acceso para acceder a los secretos de Key Vault
- Asignación del rol de operador del servicio de claves de la cuenta de almacenamiento a la aplicación de funciones para acceder a las claves de acceso de la cuenta de almacenamiento
- Funciones de rotación de las claves de la cuenta de almacenamiento con desencadenador de eventos y desencadenador HTTP (rotación a petición)
- Suscripción a eventos de EventGrid para el evento **SecretNearExpiry**

1. Seleccione el vínculo de implementación de la plantilla de Azure: 

   [![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjlichwa%2FKeyVault-Rotation-StorageAccountKey-PowerShell%2Fmaster%2Farm-templates%2FInitial-Setup%2Fazuredeploy.json)

1. En la lista **Grupo de recursos**, seleccione **akvrotation**.
1. En **Nombre de la cuenta de almacenamiento**, escriba el nombre de la cuenta de almacenamiento con las claves de acceso para rotar.
1. En **Nombre del almacén de claves**, escriba el nombre del almacén de claves.
1. En **Nombre de la aplicación de funciones**, escriba el nombre de la aplicación de funciones.
1. En **Nombre del secreto**, escriba el nombre del secreto en el que se almacenarán las claves de acceso.
1. En **Dirección URL del repositorio**, escriba la ubicación del código de la función en GitHub ( **https://github.com/jlichwa/KeyVault-Rotation-StorageAccountKey-PowerShell.git** ).
1. Seleccione **Revisar y crear**.
1. Seleccione **Crear**

   ![Revisar y crear](../media/secrets/rotation-dual/dual-rotation-2.png)

Tras completar los pasos anteriores tendrá una cuenta de almacenamiento, una granja de servidores, una aplicación de funciones y Application Insights. Debería ver la pantalla siguiente una vez completada la implementación: ![Implementación completada](../media/secrets/rotation-dual/dual-rotation-3.png)
> [!NOTE]
> En caso de que se produzcan errores, puede hacer clic en **Implementar de nuevo** para finalizar la implementación de los componentes restantes.


Las plantillas de implementación y el código de las funciones de rotación se pueden encontrar en [GitHub](https://github.com/jlichwa/KeyVault-Rotation-StorageAccountKey-PowerShell).

## <a name="add-storage-account-access-key-to-key-vault"></a>Adición de la clave de acceso de la cuenta de almacenamiento a Key Vault

En primer lugar, establezca una directiva de acceso para conceder a los usuarios el permiso para *administrar secretos*:

```azurecli
az keyvault set-policy --upn <email-address-of-user> --name akvrotation-kv --secret-permissions set delete get list
```

Ahora puede crear un nuevo secreto con una clave de acceso de la cuenta de almacenamiento como su valor. También necesitará el identificador de recurso de la cuenta de almacenamiento, el período de validez del secreto y el identificador de clave para agregarlos al secreto, para que la función de rotación pueda volver a generar la clave en la cuenta de almacenamiento.

Recupere el identificador del recurso de la cuenta de almacenamiento. Puede encontrar el valor en la propiedad `id`.
```azurecli
az storage account show -n akvrotationstorage
```

Enumere las claves de acceso de la cuenta de almacenamiento para recuperar los valores de la clave.

```azurecli
az storage account keys list -n akvrotationstorage 
```

Rellene los valores recuperados para **key1Value** y **storageAccountResourceId**.

```azurecli
$tomorrowDate = (get-date).AddDays(+1).ToString("yyy-MM-ddThh:mm:ssZ")
az keyvault secret set --name storageKey --vault-name akvrotation-kv --value <key1Value> --tags "CredentialId=key1" "ProviderAddress=<storageAccountResourceId>" "ValidityPeriodDays=60" --expires $tomorrowDate
```

La creación de un secreto con una fecha de expiración corta publicará un evento `SecretNearExpiry` en unos minutos, lo que a su vez desencadenará la función para rotar el secreto.

Puede comprobar que se han vuelto a generar las claves de acceso mediante la recuperación y comparación de las claves de la cuenta de almacenamiento y el secreto de Key Vault.

Puede mostrar la información del secreto con el siguiente comando:
```azurecli
az keyvault secret show --vault-name akvrotation-kv --name storageKey
```
Observe que se actualiza el valor de `CredentialId` para alternar el valor de `keyName` y se regenera `value`. ![Mostrar secreto](../media/secrets/rotation-dual/dual-rotation-4.png)

Recupere las claves de acceso para comprobar el valor.
```azurecli
az storage account keys list -n akvrotationstorage 
```
![Lista de claves de acceso](../media/secrets/rotation-dual/dual-rotation-5.png)

## <a name="add-additional-storage-accounts-for-rotation"></a>Adición de cuentas de almacenamiento adicionales para la rotación

Se puede reutilizar la misma aplicación de funciones para rotar varias cuentas de almacenamiento. 

La adición de claves de cuenta de almacenamiento adicionales para su rotación a la función existente requiere:
- Asignación del rol de operador del servicio de claves de la cuenta de almacenamiento a la aplicación de funciones para acceder a las claves de acceso de la cuenta de almacenamiento
- Suscripción a eventos de EventGrid para el evento **SecretNearExpiry**

1. Seleccione el vínculo de implementación de la plantilla de Azure: 

   [![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjlichwa%2FKeyVault-Rotation-StorageAccountKey-PowerShell%2Fmaster%2Farm-templates%2FAdd-Event-Subscription%2Fazuredeploy.json)

1. En la lista **Grupo de recursos**, seleccione **akvrotation**.
1. En **Nombre de la cuenta de almacenamiento**, escriba el nombre de la cuenta de almacenamiento con las claves de acceso para rotar.
1. En **Nombre del almacén de claves**, escriba el nombre del almacén de claves.
1. En **Nombre de la aplicación de funciones**, escriba el nombre de la aplicación de funciones.
1. En **Nombre del secreto**, escriba el nombre del secreto en el que se almacenarán las claves de acceso.
1. Seleccione **Revisar y crear**.
1. Seleccione **Crear**

   ![Revisar y crear](../media/secrets/rotation-dual/dual-rotation-7.png)

### <a name="add-another-storage-account-access-key-to-key-vault"></a>Adición de otra clave de acceso de la cuenta de almacenamiento a Key Vault

Recupere el identificador del recurso de la cuenta de almacenamiento. Puede encontrar el valor en la propiedad `id`.
```azurecli
az storage account show -n akvrotationstorage2
```

Enumere las claves de acceso de la cuenta de almacenamiento para recuperar el valor de la clave 2.

```azurecli
az storage account keys list -n akvrotationstorage2 
```

Rellene los valores recuperados para **key2Value** y **storageAccountResourceId**.

```azurecli
$tomorrowDate = (get-date).AddDays(+1).ToString("yyy-MM-ddThh:mm:ssZ")
az keyvault secret set --name storageKey2 --vault-name akvrotation-kv --value <key2Value> --tags "CredentialId=key2" "ProviderAddress=<storageAccountResourceId>" "ValidityPeriodDays=60" --expires $tomorrowDate
```

Muestre la información del secreto con el siguiente comando:
```azurecli
az keyvault secret show --vault-name akvrotation-kv --name storageKey2
```
Observe que se actualiza el valor de `CredentialId` para alternar el valor de `keyName` y se regenera `value`. ![Mostrar secreto](../media/secrets/rotation-dual/dual-rotation-8.png)

Recupere las claves de acceso para comprobar el valor.
```azurecli
az storage account keys list -n akvrotationstorage 
```
![Lista de claves de acceso](../media/secrets/rotation-dual/dual-rotation-9.png)

## <a name="available-key-vault-dual-credential-rotation-functions"></a>Funciones disponibles de rotación de credenciales duales de Key Vault

- [Storage Account](https://github.com/jlichwa/KeyVault-Rotation-StorageAccountKey-PowerShell)
- [Redis Cache](https://github.com/jlichwa/KeyVault-Rotation-RedisCacheKey-PowerShell)

## <a name="learn-more"></a>Más información
- Introducción: [Supervisión de Key Vault con Azure Event Grid (versión preliminar)](../general/event-grid-overview.md)
- Procedimientos: [Creación de su primera función en Azure Portal](../../azure-functions/functions-create-first-azure-function.md)
- Procedimientos: [Recibir un correo electrónico al cambiar un secreto del almacén de claves](../general/event-grid-logicapps.md)
- [Esquema de eventos de Azure Event Grid para Azure Key Vault (versión preliminar)](../../event-grid/event-schema-key-vault.md)
