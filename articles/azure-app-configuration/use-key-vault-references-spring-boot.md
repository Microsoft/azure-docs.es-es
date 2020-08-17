---
title: Tutorial sobre el uso de las referencias de Key Vault de Azure App Configuration en una aplicación de Java Spring Boot | Microsoft Docs
description: En este tutorial, aprenderá a usar las referencias de Key Vault de Azure App Configuration desde una aplicación de Java Spring Boot
services: azure-app-configuration
documentationcenter: ''
author: lisaguthrie
manager: maiye
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 12/16/2019
ms.author: lcozzens
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 31aaa0134ffe34d0424868221f01b68b64e4b088
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87371165"
---
# <a name="tutorial-use-key-vault-references-in-a-java-spring-app"></a>Tutorial: Uso de referencias de Key Vault en una aplicación de Java Spring

En este tutorial aprenderá a usar el servicio Azure App Configuration junto con Azure Key Vault. App Configuration y Key Vault son servicios complementarios que se usan de forma conjunta en la mayoría de las implementaciones de aplicaciones.

App Configuration ayuda a usar juntos los servicios, mediante la creación de claves que hacen referencia a valores almacenados en Key Vault. Cuando App Configuration crea estas claves, almacena los URI de los valores de Key Vault, en lugar de los propios valores.

La aplicación usa el proveedor de cliente de App Configuration para recuperar las referencias de Key Vault, tal como hace para cualquier otra clave almacenada en App Configuration. En este caso, los valores almacenados en App Configuration son los URI que hacen referencia a los valores de Key Vault. No son valores ni credenciales de Key Vault. Dado que el proveedor de cliente reconoce las claves como referencias de Key Vault, utiliza Key Vault para recuperar sus valores.

La aplicación es responsable de autenticarse correctamente en App Configuration y en Key Vault. Los dos servicios no se comunican directamente.

En este tutorial se muestra cómo se implementan las referencias de Key Vault en el código. Se basa en la aplicación web que se introdujo en los inicios rápidos. Antes de continuar, complete primero [Creación de una aplicación de Java Spring con Azure App Configuration](./quickstart-java-spring-app.md).

Para realizar los pasos de este tutorial, puede usar cualquier editor de código. Por ejemplo, [Visual Studio Code](https://code.visualstudio.com/) es un editor de código multiplataforma que está disponible para los sistemas operativos Windows, macOS y Linux.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una clave de App Configuration que hace referencia a un valor almacenado en Key Vault.
> * Acceder al valor de esta clave desde una aplicación de Java Spring.

## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
* Un [kit de desarrollo de Java (JDK)](https://docs.microsoft.com/java/azure/jdk) admitido, versión 8.
* [Apache Maven](https://maven.apache.org/download.cgi), versión 3.0 o posterior.

## <a name="create-a-vault"></a>Creación de un almacén

1. Seleccione la opción **Crear un recurso** en la esquina superior izquierda de Azure Portal:

    ![Salida tras completarse la creación del almacén de claves](./media/quickstarts/search-services.png)
1. En el cuadro de búsqueda, escriba **Key Vault**.
1. En la lista de resultados, seleccione **Key Vaults** a la izquierda.
1. En **Key Vaults**, seleccione **Agregar**.
1. En la sección **Crear almacén de claves**, a la derecha, proporcione la siguiente información:
    * Seleccione **Suscripción** para elegir una suscripción.
    * En **Grupo de recursos**, seleccione **Crear nuevo** y escriba un nombre de grupo de recursos.
    * En **Nombre del almacén de claves** se requiere un nombre único. Para este tutorial, escriba **Contoso-vault2**.
    * En la lista desplegable **Región**, elija una ubicación.
1. Deje las demás opciones de **Crear almacén de claves** con sus valores predeterminados.
1. Seleccione **Crear**.

En este momento, su cuenta de Azure es la única autorizada para acceder a este nuevo almacén.

![Salida tras completarse la creación del almacén de claves](./media/quickstarts/vault-properties.png)

## <a name="add-a-secret-to-key-vault"></a>Incorporación de un secreto a Key Vault

Para agregar un secreto al almacén, debe llevar a cabo algunos pasos adicionales. En este caso, agregará un mensaje que podrá usar para probar la recuperación de Key Vault. El mensaje se denominará **Message** y en él se almacenará el valor "Hello from Key Vault".

1. En las páginas de propiedades de Key Vault, seleccione **Secretos**.
1. Seleccione **Generar o importar**.
1. En el panel **Crear un secreto**, escriba los valores siguientes:
    * **Opciones de carga**: escriba **Manual**.
    * **Name**: escriba **Message**.
    * **Valor**: escriba **Hello from Key Vault**.
1. Deje las demás propiedades de **Crear un secreto** con sus valores predeterminados.
1. Seleccione **Crear**.

## <a name="add-a-key-vault-reference-to-app-configuration"></a>Incorporación de una referencia de Key Vault a App Configuration

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Todos los recursos** y después seleccione la instancia de almacén de App Configuration que creó en el inicio rápido.

1. Seleccione **Explorador de configuración**.

1. Seleccione **+ Crear** > **Referencia del almacén de claves** y, a continuación, especifique los valores siguientes:
    * **Clave**: seleccione **/application/config.keyvaultmessage**.
    * **Etiqueta**: deje este valor en blanco.
    * **Suscripción**, **Grupo de recursos** y **Key Vault**: escriba los valores correspondientes a los valores del almacén de claves que creó en la sección anterior.
    * **Secreto**: seleccione el secreto llamado **Message** que creó en la sección anterior.

## <a name="connect-to-key-vault"></a>Conexión a Key Vault

1. En este tutorial usará una entidad de servicio para la autenticación en Key Vault. Para crear esta entidad de servicio, use el comando [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) de la CLI de Azure:

    ```azurecli
    az ad sp create-for-rbac -n "http://mySP" --sdk-auth
    ```

    Esta operación devuelve una serie de pares clave-valor:

    ```console
    {
    "clientId": "7da18cae-779c-41fc-992e-0527854c6583",
    "clientSecret": "b421b443-1669-4cd7-b5b1-394d5c945002",
    "subscriptionId": "443e30da-feca-47c4-b68f-1636b75e16b3",
    "tenantId": "35ad10f1-7799-4766-9acf-f2d946161b77",
    "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
    "resourceManagerEndpointUrl": "https://management.azure.com/",
    "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
    "galleryEndpointUrl": "https://gallery.azure.com/",
    "managementEndpointUrl": "https://management.core.windows.net/"
    }
    ```

1. Ejecute el siguiente comando para permitir que la entidad de servicio acceda al almacén de claves:

    ```console
    az keyvault set-policy -n <your-unique-keyvault-name> --spn <clientId-of-your-service-principal> --secret-permissions delete get
    ```

1. Ejecute el siguiente comando para obtener el identificador de objeto y, a continuación, agréguelo a App Configuration.

    ```console
    az ad sp show --id <clientId-of-your-service-principal>
    az role assignment create --role "App Configuration Data Reader" --assignee-object-id <objectId-of-your-service-principal> --resource-group <your-resource-group>
    ```

1. Cree las siguientes variables de entorno, con los valores de la entidad de servicio mostrados en el paso anterior:

    * **AZURE_CLIENT_ID**: *clientId*
    * **AZURE_CLIENT_SECRET**: *clientSecret*
    * **AZURE_TENANT_ID**: *tenantId*

> [!NOTE]
> Estas credenciales de Key Vault se usan solo dentro de la aplicación.  La aplicación se autentica directamente con Key Vault mediante estas credenciales sin que intervenga el servicio App Configuration.  Key Vault proporciona autenticación para la aplicación y el servicio App Configuration sin compartir ni exponer las claves.

## <a name="update-your-code-to-use-a-key-vault-reference"></a>Actualización del código para usar una referencia de Key Vault

1. Cree una variable de entorno denominada **APP_CONFIGURATION_ENDPOINT**. Establezca su valor en el punto de conexión para el almacén de App Configuration. Puede encontrar el punto de conexión en la hoja **Claves de acceso** de Azure Portal.

1. Abra *bootstrap.properties* en la carpeta *resources*. Actualice este archivo para usar el punto de conexión de App Configuration, en lugar de una cadena de conexión.

    ```properties
    spring.cloud.azure.appconfiguration.stores[0].endpoint= ${APP_CONFIGURATION_ENDPOINT}
    ```

1. Abra *MessageProperties.java*. Agregue una nueva variable denominada *keyVaultMessage*:

    ```java
    private String keyVaultMessage;

    public String getKeyVaultMessage() {
        return keyVaultMessage;
    }

    public void setKeyVaultMessage(String keyVaultMessage) {
        this.keyVaultMessage = keyVaultMessage;
    }
    ```

1. Abra *HelloController.java*. Actualice el método *getMessage* para incluir el mensaje recuperado de Key Vault.

    ```java
    @GetMapping
    public String getMessage() {
        return "Message: " + properties.getMessage() + "\nKey Vault message: " + properties.getKeyVaultMessage();
    }
    ```

1. Cree un nuevo archivo llamado *AzureCredentials.java* y agregue el código siguiente.

    ```java
    package com.example.demo;

    import com.azure.core.credential.TokenCredential;
    import com.azure.identity.EnvironmentCredentialBuilder;
    import com.microsoft.azure.spring.cloud.config.AppConfigurationCredentialProvider;
    import com.microsoft.azure.spring.cloud.config.KeyVaultCredentialProvider;

    public class AzureCredentials implements AppConfigurationCredentialProvider, KeyVaultCredentialProvider{

        @Override
        public TokenCredential getKeyVaultCredential(String uri) {
            return getCredential();
        }

        @Override
        public TokenCredential getAppConfigCredential(String uri) {
            return getCredential();
        }

        private TokenCredential getCredential() {
            return new EnvironmentCredentialBuilder().build();
        }

    }
    ```

1. Cree un nuevo archivo llamado *AppConfiguration.java*. Y agregue el código siguiente.

    ```java
    package com.example.demo;

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class AppConfiguration {

        @Bean
        public AzureCredentials azureCredentials() {
            return new AzureCredentials();
        }
    }
    ```

1. Cree un nuevo archivo en el directorio META-INF de recursos llamado *spring.factories* y agréguelo.

    ```factories
    org.springframework.cloud.bootstrap.BootstrapConfiguration=\
    com.example.demo.AppConfiguration
    ```

1. Compile la aplicación de Spring Boot con Maven y ejecútela; por ejemplo:

    ```shell
    mvn clean package
    mvn spring-boot:run
    ```

1. Una vez que se está ejecutando la aplicación, puede usar *curl* para probarla, por ejemplo:

      ```shell
      curl -X GET http://localhost:8080/
      ```

    Verá el mensaje que escribió en el almacén de App Configuration. También verá el mensaje que escribió en Key Vault.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha creado una clave de App Configuration que hace referencia a un valor almacenado en Key Vault. Para obtener información sobre cómo usar las marcas de características en la aplicación de Java Spring, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Integración de identidades administradas](./quickstart-feature-flag-spring-boot.md)
