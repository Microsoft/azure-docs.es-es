---
title: 'Tutorial: Importación de un certificado en Key Vault mediante Azure Portal | Microsoft Docs'
description: Tutorial que muestra cómo importar un certificado en Azure Key Vault
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: tutorial
ms.custom: mvc, devx-track-azurecli
ms.date: 04/16/2020
ms.author: sebansal
ms.openlocfilehash: ebf687716c8898acffb5e081fbf2f6217fe0f943
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503130"
---
# <a name="tutorial-import-a-certificate-in-azure-key-vault"></a>Tutorial: Importación de un certificado en Azure Key Vault

Azure Key Vault es un servicio de almacenamiento seguro de secretos en la nube. Puede almacenar de forma segura claves, contraseñas, certificados y otros secretos. Las instancias de Azure Key Vault se pueden crear y administrar a través de Azure Portal. En este tutorial creará un almacén de claves y lo usará para importar un certificado. Para más información sobre Key Vault, consulte esta [introducción](../general/overview.md).

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Cree un almacén de claves.
> * Importe un certificado en Key Vault mediante el portal.
> * Importe un certificado en Key Vault mediante la CLI.
> * Importe un certificado en Key Vault mediante PowerShell.


Antes de empezar, lea los [conceptos básicos de Key Vault](../general/basic-concepts.md). 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-a-vault"></a>Creación de un almacén

1. En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**.
2. En el cuadro de búsqueda, escriba **Key Vault**.
3. En la lista de resultados, elija **Key Vault**.
4. En la sección Key Vault, elija **Crear**.
5. En la sección **Crear Key Vault**, proporcione la siguiente información:
    - **Name**: se requiere un nombre único. En este inicio rápido se usará **Example-Vault**. 
    - **Suscripción**: Elija una suscripción.
    - En **Grupo de recursos** elija **Crear nuevo** y escriba un nombre para el grupo de recursos.
    - En el menú desplegable **Ubicación**, elija una ubicación.
    - Deje las restantes opciones con sus valores predeterminados.
6. Después de proporcionar la información descrita anteriormente, seleccione **Crear**.

Tome nota de las dos propiedades siguientes:

* **Nombre del almacén**: En el ejemplo, es **Example-Vault**. Utilizará este nombre para otros pasos.
* **URI de almacén**: en el ejemplo es https://example-vault.vault.azure.net/. Las aplicaciones que utilizan el almacén a través de su API de REST deben usar este identificador URI.

En este momento, su cuenta de Azure es la única autorizada para realizar operaciones en este nuevo almacén.

![Salida tras completarse la creación de Key Vault](../media/certificates/tutorial-import-cert/vault-properties.png)

## <a name="import-a-certificate-to-key-vault"></a>Importación de un certificado en Key Vault

Para importar un certificado en el almacén, debe tener un archivo de certificado PEM o PFX en el disco. En este caso, se importará un certificado con el nombre de archivo llamado **ExampleCertificate**.

> [!IMPORTANT]
> En Azure Key Vault, los formatos de certificado admitidos son PFX y PEM. 
> - El formato de archivo .pem contiene uno o varios archivos de certificado X509.
> - El formato de archivo .pfx es un formato de archivo de almacenamiento para almacenar varios objetos criptográficos en un único archivo, por ejemplo, un certificado de servidor (emitido para su dominio), una clave privada coincidente y puede incluir opcionalmente una CA intermedia.  

1. En las páginas de propiedades de Key Vault, seleccione **Certificados**.
2. Haga clic en **Generar o Importar**.
3. En la pantalla **Creación de certificado**, elija los siguientes valores:
    - **Método de creación de certificados**: Importación.
    - **Nombre del certificado**: ExampleCertificate.
    - **Cargar el archivo de certificado**: seleccione el archivo de certificado del disco.
    - **Contraseña**: si va a cargar un archivo de certificado protegido con contraseña, proporcione esa contraseña aquí. De lo contrario, déjelo en blanco. Una vez que el archivo de certificado se haya importado correctamente, Key Vault quitará esa contraseña.
4. Haga clic en **Crear**.

![Propiedades del certificado](../media/certificates/tutorial-import-cert/cert-import.png)

Al agregar un certificado mediante el método **Import**, Azure Key Vault rellenará automáticamente los parámetros de certificado (es decir, el período de validez, el nombre del emisor, la fecha de activación, etc.).

Una vez recibido el mensaje de que el certificado se ha importado correctamente, puede hacer clic en él en la lista para ver sus propiedades. 

![Propiedades del certificado](../media/certificates/tutorial-import-cert/current-version-hidden.png)

## <a name="import-a-certificate-using-azure-cli"></a>Importación de un certificado mediante la CLI de Azure

Importe un certificado en un almacén de claves especificado. Para importar en Azure Key Vault un certificado válido existente que contiene una clave privada, el archivo que se va a importar puede tener el formato PFX o PEM. Si el certificado está en formato PEM, el archivo PEM debe contener la clave, así como los certificados X509. Esta operación requiere el permiso certificados/importar.

```azurecli
az keyvault certificate import --file
                               --name
                               --vault-name
                               [--disabled {false, true}]
                               [--only-show-errors]
                               [--password]
                               [--policy]
                               [--subscription]
                               [--tags]
```

[Más información sobre los parámetros](https://docs.microsoft.com/cli/azure/keyvault/certificate?view=azure-cli-latest#az-keyvault-certificate-import).

Después de importar el certificado, puede verlo mediante el comando para [ver el certificado](https://docs.microsoft.com/cli/azure/keyvault/certificate?view=azure-cli-latest#az-keyvault-certificate-show).


```azurecli
az keyvault certificate show [--id]
                             [--name]
                             [--only-show-errors]
                             [--subscription]
                             [--vault-name]
                             [--version]
```

Ahora, ha creado un almacén de claves, ha importado un certificado y ha visto sus propiedades.

## <a name="import-a-certificate-using-azure-powershell"></a>Importación de un certificado mediante Azure PowerShell

```
Import-AzureKeyVaultCertificate
      [-VaultName] <String>
      [-Name] <String>
      -FilePath <String>
      [-Password <SecureString>]
      [-Tag <Hashtable>]
      [-DefaultProfile <IAzureContextContainer>]
      [-WhatIf]
      [-Confirm]
      [<CommonParameters>]
```

[Más información sobre los parámetros](https://docs.microsoft.com/powershell/module/azurerm.keyvault/import-azurekeyvaultcertificate?view=azurermps-6.13.0).


## <a name="clean-up-resources"></a>Limpieza de recursos

Otras guías de inicio rápido y tutoriales de Key Vault se basan en esta. Si tiene pensado seguir trabajando en otras guías de inicio rápido y tutoriales, considere la posibilidad de dejar estos recursos activos.
Cuando ya no lo necesite, elimine el grupo de recursos; de este modo se eliminarán también Key Vault y los recursos relacionados. Para eliminar el grupo de recursos mediante el portal:

1. Escriba el nombre del grupo de recursos en el cuadro de búsqueda de la parte superior del portal. Cuando vea el grupo de recursos que se utiliza en esta guía de inicio rápido en los resultados de búsqueda, selecciónelo.
2. Seleccione **Eliminar grupo de recursos**.
3. En el cuadro **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS:** escriba el nombre del grupo de recursos y seleccione **Eliminar**.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un almacén de claves e importado un certificado en él. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Más información sobre la [administración de la creación de certificados en Azure Key Vault](https://docs.microsoft.com/azure/key-vault/certificates/create-certificate-scenarios).
- Consulte ejemplos de [Importación de certificados mediante las API REST](/rest/api/keyvault/importcertificate/importcertificate).
- Consulte los [procedimientos recomendados de Azure Key Vault](../general/best-practices.md).
