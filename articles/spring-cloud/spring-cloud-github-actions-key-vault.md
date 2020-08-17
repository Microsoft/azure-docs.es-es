---
title: Autenticación de Azure Spring Cloud con Key Vault en Acciones de GitHub
description: Uso de Key Vault con un flujo de trabajo de CI/CD para Azure Spring Cloud con Acciones de GitHub
author: MikeDodaro
ms.author: barbkess
ms.service: spring-cloud
ms.topic: how-to
ms.date: 01/20/2019
ms.custom: devx-track-java
ms.openlocfilehash: 433cd9e7b8cfe69ce5008366db884659cccbc149
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076010"
---
# <a name="authenticate-azure-spring-cloud-with-key-vault-in-github-actions"></a>Autenticación de Azure Spring Cloud con Key Vault en Acciones de GitHub
Key Vault es un lugar seguro para almacenar claves. Los usuarios empresariales deben almacenar las credenciales de los entornos de CI/CD en un ámbito que controlen. La clave para obtener las credenciales en Key Vault se debe limitar al ámbito de los recursos.  Solo tiene acceso al ámbito de Key Vault, no a todo el ámbito de Azure. Es como una llave que solo puede abrir una caja fuerte, no una llave maestra que puede abrir todas las puertas de un edificio. Es una forma de obtener una clave con otra clave, lo que es útil en un flujo de trabajo de CI/CD. 

## <a name="generate-credential"></a>Generación de una credencial
Para generar una clave para acceder a Key Vault, ejecute el comando siguiente en el equipo local:
```
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.KeyVault/vaults/<KEY_VAULT> --sdk-auth
```
El ámbito especificado por el parámetro `--scopes` limita el acceso de la clave al recurso.  Solo puede acceder a la caja fuerte.

Con resultados:
```
{
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
    "resourceManagerEndpointUrl": "https://management.azure.com/",
    "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
    "galleryEndpointUrl": "https://gallery.azure.com/",
    "managementEndpointUrl": "https://management.core.windows.net/"
}
```
Luego, guarde los resultados en **secrets** de GitHub, como se describe en [Autenticación y configuración de un repositorio de GitHub con Azure](./spring-cloud-howto-github-actions.md#set-up-github-repository-and-authenticate).

## <a name="add-access-policies-for-the-credential"></a>Incorporación de directivas de acceso a la credencial
La credencial que ha creado anteriormente solo puede obtener información general sobre Key Vault, no sobre el contenido que almacena.  Para obtener los secretos almacenados en Key Vault, es preciso establecer directivas de acceso para la credencial.

Vaya al panel **Key Vault** en Azure Portal, haga clic en el menú **Control de acceso** y abra la pestaña **Asignaciones de roles**. Seleccione **Aplicaciones** en **Tipo** y `This resource` en **Ámbito**.  Debería ver la credencial que ha creado en el paso anterior:

 ![Establecer la directiva de acceso](./media/github-actions/key-vault1.png)

Copie el nombre de la credencial, por ejemplo, `azure-cli-2020-01-19-04-39-02`. Abra el menú **Directivas de acceso** y haga clic en el vínculo **+Agregar directiva de acceso**.  Seleccione `Secret Management` en **Plantilla** y, después, seleccione **Entidad de seguridad**. Pegue el nombre de la credencial en el cuadro de entrada **Entidad de seguridad**/**Seleccionar**:

 ![Seleccionar](./media/github-actions/key-vault2.png)

 Haga clic en el botón **Agregar** en el cuadro de diálogo **Agregar directiva de acceso** y, después, haga clic en **Guardar**.

## <a name="generate-full-scope-azure-credential"></a>Generación de una credencial de Azure de ámbito completo
Esta es la llave maestra que abre todas las puertas del edificio. El procedimiento es similar al paso anterior, pero aquí cambiamos el ámbito para generar la llave maestra:

```
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID> --sdk-auth
```

Una vez más, resultados:
```
{
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
    "resourceManagerEndpointUrl": "https://management.azure.com/",
    "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
    "galleryEndpointUrl": "https://gallery.azure.com/",
    "managementEndpointUrl": "https://management.core.windows.net/"
}
```
Copie toda la cadena JSON.  Vuelva al panel de **Key Vault**. Abra el menú **Secretos** y, después, haga clic en el botón **Generate/Import** (Generar o importar). Escriba el nombre del secreto, como `AZURE-CREDENTIALS-FOR-SPRING`. Pegue la cadena de la credencial JSON en el cuadro de entrada **Valor**. Es posible que observe que el cuadro de entrada Valor es un campo de texto de una línea, no un área de texto de varias líneas.  Puede pegar la cadena JSON completa ahí.

 ![Credencial de ámbito completo](./media/github-actions/key-vault3.png)

## <a name="combine-credentials-in-github-actions"></a>Combinación de credenciales en Acciones de GitHub
Establezca las credenciales que se usan cuando se ejecuta la canalización CI/CD:

```
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}           # Strong box key you generated in the first step
    - uses: Azure/get-keyvault-secrets@v1.0
      with:
        keyvault: "<Your Key Vault Name>"
        secrets: "AZURE-CREDENTIALS-FOR-SPRING"           # Master key to open all doors in the building
      id: keyvaultaction
    - uses: azure/login@v1
      with:
        creds: ${{ steps.keyvaultaction.outputs.AZURE-CREDENTIALS-FOR-SPRING }}
    - name: Azure CLI script
      uses: azure/CLI@v1
      with:
        azcliversion: 2.0.75
        inlineScript: |
          az extension add --name spring-cloud             # Spring CLI commands from here
          az spring-cloud list

```

## <a name="next-steps"></a>Pasos siguientes
* [Acciones de GitHub en Spring Cloud](./spring-cloud-howto-github-actions.md)
