---
title: 'Administración de Azure Key Vault mediante la CLI: Azure Key Vault | Microsoft Docs'
description: Use este artículo para automatizar tareas comunes de Key Vault mediante la CLI de Azure
services: key-vault
author: msmbaldwin
manager: rkarlin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 08/12/2019
ms.author: mbaldwin
ms.custom: devx-track-azurecli
ms.openlocfilehash: dc60d2b6cef8ad19526c5ec243ae1c43529954a6
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87504541"
---
# <a name="manage-key-vault-using-the-azure-cli"></a>Administración de Key Vault mediante la CLI de Azure 

En este artículo se explica cómo empezar a trabajar con Azure Key Vault mediante la CLI de Azure.  Puede ver información sobre:

- Procedimiento para crear un contenedor protegido (un almacén) en Azure.
- Agregar una clave, un secreto o un certificado al almacén de claves
- Registrar una aplicación con Azure Active Directory
- Autorizar una aplicación para que use una clave o un secreto
- Configuración de directivas de acceso avanzado de almacén de claves
- Trabajar con un módulo de seguridad de hardware (HSM)
- Eliminar el almacén de claves y las claves y los secretos asociados
- Otros comandos de la interfaz de la línea de comandos multiplataforma de Azure


Azure Key Vault está disponible en la mayoría de las regiones. Para obtener más información, consulte la [página de precios de Key Vault](https://azure.microsoft.com/pricing/details/key-vault/).

> [!NOTE]
> En este artículo no se incluyen instrucciones sobre cómo escribir la aplicación de Azure incluida en uno de los pasos, en el que se muestra cómo autorizar a una aplicación para que use una clave o un secreto del almacén de claves.
>

Para obtener una introducción a Azure Key Vault, consulte [¿Qué es Azure Key Vault?](overview.md). Si no tiene una suscripción de Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para usar los comandos de la CLI de Azure, debe tener los siguientes elementos:

* Una suscripción a Microsoft Azure. Si no tiene una, puede registrarse para una [evaluación gratuita](https://azure.microsoft.com/pricing/free-trial).
* Versión 2.0 o posterior de la Interfaz de la línea de comandos de Azure. Para instalar la versión más reciente, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).
* Aplicación que se configurará para utilizar la clave o contraseña creada en este artículo. Hay una aplicación de ejemplo disponible en el [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=45343). Para obtener instrucciones, consulte el archivo Léame incluido.

### <a name="getting-help-with-azure-cross-platform-command-line-interface"></a>Obtención de ayuda con la interfaz de la línea de comandos entre plataformas de Azure

En este artículo se asume que está familiarizado con la interfaz de la línea de comandos (Bash, Terminal, símbolo del sistema).

Los parámetros --help o -h se pueden usar para ver la ayuda de comandos específicos. También puede usar el formato Azure help [command] [options]. Si duda sobre los parámetros que necesita un comando, consulte la ayuda. Por ejemplo, todos los comandos siguientes devuelven la misma información:

```azurecli
az account set --help
az account set -h
```

También puede leer los artículos siguientes para familiarizarse con Azure Resource Manager en la interfaz de la línea de comandos entre plataformas de Azure:

* [Instalación de la CLI de Azure](/cli/azure/install-azure-cli)
* [Introducción a la CLI de Azure](/cli/azure/get-started-with-azure-cli)

## <a name="how-to-create-a-hardened-container-a-vault-in-azure"></a>Procedimiento para crear un contenedor protegido (un almacén) en Azure.

Los almacenes son contenedores protegidos respaldados por módulos de seguridad de hardware. Los almacenes ayudan a reducir las posibilidades de que se produzca una pérdida accidental de información de seguridad centralizando el almacenamiento de los secretos de aplicación. Key Vault también permite controlar y registrar el acceso a todo lo que está almacenado en ellos. Azure Key Vault puede administrar la solicitud y renovación de los certificados de Seguridad de la capa de transporte (TLS), proporcionando las características necesarias para conseguir una eficaz solución de administración del ciclo de vida de los certificados. En los pasos siguientes, creará un almacén.

### <a name="connect-to-your-subscriptions"></a>Conectarse a sus suscripciones

Para iniciar sesión de forma interactiva, use el comando siguiente:

```azurecli
az login
```
Para iniciar sesión con una cuenta profesional, puede introducir su nombre de usuario y contraseña.

```azurecli
az login -u username@domain.com -p password
```

Si tiene más de una suscripción y quiere especificar la que quiere usar, escriba lo siguiente para ver las suscripciones de su cuenta:

```azurecli
az account list
```

Especifique una suscripción con el parámetro de la suscripción.

```azurecli
az account set --subscription <subscription name or ID>
```

Para más información acerca de cómo configurar la interfaz de la línea de comandos entre plataformas de Azure, consulte el artículo sobre la [instalación de la CLI de Azure](/cli/azure/install-azure-cli).

### <a name="create-a-new-resource-group"></a>Creación de un nuevo grupo de recursos

Cuando se utiliza el Administrador de recursos de Azure, todos los recursos relacionados se crean dentro de un grupo de recursos. Puede crear un almacén de claves en un grupo de recursos existente. Si quiere usar un nuevo grupo de recursos, puede crearlo.

```azurecli
az group create -n "ContosoResourceGroup" -l "East Asia"
```

El primer parámetro es el nombre del grupo de recursos y el segundo parámetro es la ubicación. Para obtener una lista de todos los tipos de ubicaciones posibles:

```azurecli
az account list-locations
``` 

### <a name="register-the-key-vault-resource-provider"></a>Registro del proveedor de recursos de Key Vault

 Al intentar crear un almacén de claves, puede ver el error "La suscripción no está registrada para usar el espacio de nombres 'Microsoft.KeyVault'". Si aparece ese mensaje, asegúrese de que el proveedor de recursos de Key Vault está registrado en la suscripción. Se trata de una operación única para cada suscripción.

```azurecli
az provider register -n Microsoft.KeyVault
```

### <a name="create-a-key-vault"></a>Creación de un Almacén de claves

Utilice el comando `az keyvault create` para crear un Almacén de claves. Este script tiene tres parámetros obligatorios: el nombre del grupo de recursos, el nombre del Almacén de claves y la ubicación geográfica.

Para crear un nuevo almacén con el nombre **ContosoKeyVault**, en el grupo de recursos **ContosoResourceGroup**, que reside en la ubicación **Este de Asia**, escriba: 

```azurecli
az keyvault create --name "ContosoKeyVault" --resource-group "ContosoResourceGroup" --location "East Asia"
```

La salida de este comando muestra las propiedades del almacén de claves que ha creado. Las dos propiedades más importantes son:

* **name**: en el ejemplo, el nombre es ContosoKeyVault. Usará este nombre para otros comandos de Key Vault.
* **vaultUri**: en el ejemplo, el URI es https://contosokeyvault.vault.azure.net. Las aplicaciones que utilizan el almacén a través de su API de REST deben usar este identificador URI.

Su cuenta de Azure ahora está autorizada para realizar operaciones en este Almacén de claves. Hasta ahora, nadie más está autorizado.

## <a name="adding-a-key-secret-or-certificate-to-the-key-vault"></a>Agregar una clave, un secreto o un certificado al almacén de claves

Si quiere que Azure Key Vault cree una clave protegida mediante software, utilice el comando `az key create`.

```azurecli
az keyvault key create --vault-name "ContosoKeyVault" --name "ContosoFirstKey" --protection software
```

Si tiene una clave existente en un archivo .pem, puede cargarla a Azure Key Vault. Puede optar por proteger la clave con software o HSM. En este ejemplo se importa la clave desde el archivo .pem y se protege con software, mediante la contraseña "hVFkk965BuUv":

```azurecli
az keyvault key import --vault-name "ContosoKeyVault" --name "ContosoFirstKey" --pem-file "./softkey.pem" --pem-password "hVFkk965BuUv" --protection software
```

Ahora puede utilizar el URI para hacer referencia a la clave que creó o cargó en Azure Key Vault. Use **https://ContosoKeyVault.vault.azure.net/keys/ContosoFirstKey** para obtener siempre la versión actual. Use https://[keyvault-name].vault.azure.net/keys/[keyname]/[key-unique-id] para obtener la versión específica. Por ejemplo, **https://ContosoKeyVault.vault.azure.net/keys/ContosoFirstKey/cgacf4f763ar42ffb0a1gca546aygd87** . 

Agregue un secreto al almacén, que es una contraseña denominada SQLPassword y que tiene el valor "hVFkk965BuUv" en Azure Key Vault. 

```azurecli
az keyvault secret set --vault-name "ContosoKeyVault" --name "SQLPassword" --value "hVFkk965BuUv "
```

Haga referencia a esta contraseña usando su URI. Use **https://ContosoVault.vault.azure.net/secrets/SQLPassword** para obtener siempre la versión actual, y https://[keyvault-name].vault.azure.net/secret/[secret-name]/[secret-unique-id] para obtener la versión específica. Por ejemplo, **https://ContosoVault.vault.azure.net/secrets/SQLPassword/90018dbb96a84117a0d2847ef8e7189d** .

Importe un certificado al almacén mediante un archivo .pem o .pfx.

```azurecli
az keyvault certificate import --vault-name "ContosoKeyVault" --file "c:\cert\cert.pfx" --name "ContosoCert" --password "hVFkk965BuUv"
```

Veamos la clave, el secreto o el certificado que ha creado:

* Para ver las claves, escriba: 

```azurecli
az keyvault key list --vault-name "ContosoKeyVault"
```

* Para ver los secretos, escriba: 

```azurecli
az keyvault secret list --vault-name "ContosoKeyVault"
```

* Para ver los certificados, escriba: 

```azurecli
az keyvault certificate list --vault-name "ContosoKeyVault"
```

## <a name="registering-an-application-with-azure-active-directory"></a>Registrar una aplicación con Azure Active Directory

Este paso lo haría normalmente un programador en un equipo independiente. No es específico de Azure Key Vault, pero se incluye aquí para fines de información. Para completar el registro de la aplicación, su cuenta, el almacén y la aplicación deben estar en el mismo directorio de Azure.

Las aplicaciones que utilizan un Almacén de claves deben autenticarse utilizando un token de Azure Active Directory.  El propietario de la aplicación debe registrarla primero en Azure Active Directory. Al final del registro, el propietario de la aplicación obtiene los valores siguientes:

- Un **Id. de aplicación** (también conocido como id. de cliente de ADD o appID)
- **Clave de autenticación** (también conocida como "secreto compartido") 

Para obtener un token, la aplicación debe presentar estos dos valores a Azure Active Directory. La configuración de una aplicación para obtener un token dependerá de la aplicación. Para la [aplicación de ejemplo de Key Vault](https://www.microsoft.com/download/details.aspx?id=45343), el propietario de la aplicación establece estos valores en el archivo app.config.

Para obtener los pasos detallados para registrar una aplicación en Azure Active Directory, debería revisar los artículos titulados [Integración de aplicaciones con Azure Active Directory](../../active-directory/develop/active-directory-integrating-applications.md), [Uso del portal para crear una aplicación de Azure Active Directory y una entidad de servicio con acceso a los recursos](../../active-directory/develop/howto-create-service-principal-portal.md) y [Creación de una entidad de servicio de Azure con la CLI de Azure](/cli/azure/create-an-azure-service-principal-azure-cli).

Para registrar una aplicación en Azure Active Directory:

```azurecli
az ad sp create-for-rbac -n "MyApp" --password "hVFkk965BuUv" --skip-assignment
# If you don't specify a password, one will be created for you.
```

## <a name="authorizing-an-application-to-use-a-key-or-secret"></a>Autorizar una aplicación para que use una clave o un secreto

Para que la aplicación pueda acceder a la clave o el secreto en el almacén, use el comando `az keyvault set-policy` .

Por ejemplo, si el nombre del almacén es ContosoKeyVault, la aplicación tiene un valor de appID 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed y quiere que la aplicación tenga autorización para descifrar y firmar con claves en el almacén, use el comando siguiente:

```azurecli
az keyvault set-policy --name "ContosoKeyVault" --spn 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed --key-permissions decrypt sign
```

Para autorizar a la misma aplicación a leer secretos en su almacén, escriba el siguiente comando:

```azurecli
az keyvault set-policy --name "ContosoKeyVault" --spn 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed --secret-permissions get
```

## <a name="setting-key-vault-advanced-access-policies"></a><a name="bkmk_KVperCLI"></a> Configuración de directivas de acceso avanzado de almacén de claves

Use [az keyvault update](/cli/azure/keyvault#az-keyvault-update) para habilitar directivas avanzadas para el almacén de claves.

 Habilitar Key Vault para la implementación: permite que las máquinas virtuales recuperen certificados almacenados como secretos del almacén.

 ```azurecli
 az keyvault update --name "ContosoKeyVault" --resource-group "ContosoResourceGroup" --enabled-for-deployment "true"
 ```

Habilitar Key Vault para el cifrado de disco: necesario cuando se usa el almacén para el cifrado de discos de Azure.

 ```azurecli
 az keyvault update --name "ContosoKeyVault" --resource-group "ContosoResourceGroup" --enabled-for-disk-encryption "true"
 ```  

Habilitar Key Vault para la implementación de plantillas: permite que Resource Manager recupere secretos del almacén.

```azurecli
az keyvault update --name "ContosoKeyVault" --resource-group "ContosoResourceGroup" --enabled-for-template-deployment "true"
```

## <a name="working-with-hardware-security-modules-hsms"></a>Trabajar con un módulo de seguridad de hardware (HSM)

Para obtener seguridad adicional, puede importar o generar claves de módulos de seguridad de hardware (HSM) que no se salen nunca del límite de los HSM. Los HSM tienen la validación FIPS 140-2 de nivel 2. Si este requisito no es relevante para usted, omita esta sección y vaya al paso Eliminación del Almacén de claves junto con las claves y secretos asociados.

Para crear estas claves protegidas con HSM, debe contar con una suscripción de almacén que admita claves protegidas mediante HSM.

Cuando cree el almacén de claves, agregue el parámetro 'sku':

```azurecli
az keyvault create --name "ContosoKeyVaultHSM" --resource-group "ContosoResourceGroup" --location "East Asia" --sku "Premium"
```

A este almacén, se pueden agregar claves protegidas mediante software (tal como se ha mostrado anteriormente) y claves protegidas con HSM. Para crear una clave protegida con HSM, establezca el parámetro Destination en 'HSM':

```azurecli
az keyvault key create --vault-name "ContosoKeyVaultHSM" --name "ContosoFirstHSMKey" --protection "hsm"
```

Puede utilizar el siguiente comando para importar una clave desde un archivo .pem a su equipo. Este comando importa la clave a HSM en el servicio Key Vault:

```azurecli
az keyvault key import --vault-name "ContosoKeyVaultHSM" --name "ContosoFirstHSMKey" --pem-file "/.softkey.pem" --protection "hsm" --pem-password "PaSSWORD"
```

El comando siguiente importa un paquete BYOK ("traiga su propia clave"). Esto permite generar la clave en el HSM local y transferirla al HSM en el servicio de Key Vault, sin que la clave salga del límite del HSM:

```azurecli
az keyvault key import --vault-name "ContosoKeyVaultHSM" --name "ContosoFirstHSMKey" --byok-file "./ITByok.byok" --protection "hsm"
```

Para obtener instrucciones detalladas sobre cómo generar este paquete BYOK, consulte [Generación y transferencia de claves protegidas con HSM para Azure Key Vault](../keys/hsm-protected-keys.md).

## <a name="deleting-the-key-vault-and-associated-keys-and-secrets"></a>Eliminar el almacén de claves y las claves y los secretos asociados

Si ya no necesita el almacén de claves ni sus claves o secretos, puede eliminarla con el comando `az keyvault delete`:

```azurecli
az keyvault delete --name "ContosoKeyVault"
```

O bien puede eliminar un grupo de recursos de Azure completo, que incluye el Almacén de claves y otros recursos incluidos en dicho grupo:

```azurecli
az group delete --name "ContosoResourceGroup"
```

## <a name="miscellaneous-azure-cross-platform-command-line-interface-commands"></a>Otros comandos de la interfaz de la línea de comandos multiplataforma de Azure

Otros comandos que puede encontrar útiles para gestionar Azure Key Vault.

Este comando ofrece una presentación tabular de todas las claves y las propiedades seleccionadas.

```azurecli
az keyvault key list --vault-name "ContosoKeyVault"
```

Este comando muestra una lista completa de propiedades para la clave especificada.

```azurecli
az keyvault key show --vault-name "ContosoKeyVault" --name "ContosoFirstKey"
```

Este comando muestra una presentación tabular de todos nombres de secretos y las propiedades que se elijan.

```azurecli
az keyvault secret list --vault-name "ContosoKeyVault"
```

Ejemplo de cómo quitar una clave específica:

```azurecli
az keyvault key delete --vault-name "ContosoKeyVault" --name "ContosoFirstKey"
```

Ejemplo de cómo quitar un secreto específico:

```azurecli
az keyvault secret delete --vault-name "ContosoKeyVault" --name "SQLPassword"
```

## <a name="next-steps"></a>Pasos siguientes

- Para ver una referencia completa de la CLI de Azure para los comandos de Key Vault, consulte la [guía de referencia de la CLI de Key Vault](/cli/azure/keyvault).

- Para conocer las referencias de programación, consulte la [Guía del desarrollador de Azure Key Vault](developers-guide.md).

- Para obtener información sobre Azure Key Vault y los módulos de seguridad de hardware (HSM), vea [Uso de claves protegidas con HSM mediante Azure Key Vault](../keys/hsm-protected-keys.md).
