---
title: 'Tutorial: Identidad administrada para conectar Key Vault a una aplicación de Azure Spring Cloud'
description: Configuración de una identidad administrada para conectar Key Vault a una aplicación de Azure Spring Cloud
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 07/08/2020
ms.custom: devx-track-java
ms.openlocfilehash: 646b95e7e106b8657f8aeec2426b88cd6da20357
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90885649"
---
# <a name="tutorial-use-a-managed-identity-to-connect-key-vault-to-an-azure-spring-cloud-app"></a>Tutorial: Uso de una identidad administrada para conectar Key Vault a una aplicación de Azure Spring Cloud

**Este artículo se aplica a:** ✔️ Java

En este artículo se describe cómo crear una identidad administrada para una aplicación de Azure Spring Cloud y cómo usarla para acceder a Azure Key Vault.

Azure Key Vault se puede utilizar para almacenar de forma segura y controlar estrechamente el acceso a los tokens, contraseñas, certificados, claves de API y otros secretos de una aplicación. Puede crear una identidad administrada en Azure Active Directory (AAD) y autenticarse en cualquier servicio que admita la autenticación de AAD, incluido Key Vault, sin tener que mostrar las credenciales en el código.

## <a name="prerequisites"></a>Requisitos previos

* [Registro para obtener una suscripción a Azure](https://azure.microsoft.com/free/)
* [Instalación de la CLI de Azure versión 2.0.67 o superior](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true)
* [Instalación de Maven 3.0, o cualquier versión superior](https://maven.apache.org/download.cgi)

## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Cree un grupo de recursos que contenga tanto el almacén de claves como Spring Cloud mediante el comando [az group create](/cli/azure/group?view=azure-cli-latest&preserve-view=true#az-group-create):

```azurecli-interactive
az group create --name "myResourceGroup" -l "EastUS"
```

## <a name="set-up-your-key-vault"></a>Configuración de un almacén de claves
Para crear un almacén de claves, utilice el comando [az keyvault create](/cli/azure/keyvault?view=azure-cli-latest&preserve-view=true#az-keyvault-create):

> [!Important]
> Cada instancia de Key Vault debe tener un nombre único. Reemplace <your-keyvault-name> con el nombre de su almacén de claves en los ejemplos siguientes.

```azurecli-interactive
az keyvault create --name "<your-keyvault-name>" -g "myResourceGroup"
```

Tome nota del valor `vaultUri` devuelto, que tendrá el formato "https://<suNombreDeAlmacénDeClaves>.vault.azure.net/". Se usará en el paso siguiente.

Ahora puede agregar un secreto a su almacén de claves mediante el comando [az keyvault secret set](/cli/azure/keyvault/secret?view=azure-cli-latest&preserve-view=true#az-keyvault-secret-set):

```azurecli-interactive
az keyvault secret set --vault-name "<your-keyvault-name>" \
    --name "connectionString" \
    --value "jdbc:sqlserver://SERVER.database.windows.net:1433;database=DATABASE;"
```

## <a name="create-azure-spring-cloud-service-and-app"></a>Creación de una aplicación y un servicio de Azure Spring Cloud
Después de instalar la extensión correspondiente, cree una instancia de Azure Spring Cloud con el comando de la CLI de Azure `az spring-cloud create`. 

```azurecli-interactive
az extension add --name spring-cloud
az spring-cloud create -n "myspringcloud" -g "myResourceGroup"
```
En el ejemplo siguiente, se crea una aplicación denominada `springapp` con una identidad administrada asignada por el sistema, según solicitó el parámetro `--assign-identity`.

```azurecli
az spring-cloud app create -n "springapp" -s "myspringcloud" -g "myResourceGroup" --is-public true --assign-identity
export SERVICE_IDENTITY=$(az spring-cloud app show --name "springapp" -s "myspringcloud" -g "myResourceGroup" | jq -r '.identity.principalId')
```

Tome nota del valor de `url` devuelto, que tendrá el formato "https://<suNombreDeAplicación>.azuremicroservices.io". Se usará en el paso siguiente.


## <a name="grant-your-app-access-to-key-vault"></a>Concesión a la aplicación de acceso a Key Vault
Use `az keyvault set-policy` para conceder el acceso adecuado en Key Vault para la aplicación.
```azurecli
az keyvault set-policy --name "<your-keyvault-name>" --object-id ${SERVICE_IDENTITY} --secret-permissions set get list
```

## <a name="build-a-sample-spring-boot-app-with-spring-boot-starter"></a>Compilación de una aplicación de Spring Boot de ejemplo con el código de inicio de Spring Boot
Esta aplicación tendrá acceso a secretos de Azure Key Vault. Uso de la aplicación de inicio: [Inicio de Spring Boot con los secretos de Azure Key Vault](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-starter-keyvault-secrets).  Azure Key Vault se agrega como una instancia de **PropertySource** de Spring.  Se puede acceder a los secretos almacenados en Azure Key Vault y usarlos de forma apropiada como cualquier propiedad de configuración externa, por ejemplo, las propiedades de los archivos. 

1. Genere un proyecto de ejemplo a partir de start.spring.io con Azure Key Vault Spring Starter. 
    ```azurecli
    curl https://start.spring.io/starter.tgz -d dependencies=web,azure-keyvault-secrets -d baseDir=springapp -d bootVersion=2.3.1.RELEASE -d javaVersion=1.8 | tar -xzvf -
    ```

2. Especifique el almacén de claves en la aplicación. 

    ```azurecli
    cd springapp
    vim src/main/resources/application.properties
    ```

    Para usar la identidad administrada para las aplicaciones de Azure Spring Cloud, agregue propiedades con el siguiente contenido a src/main/resources/application.properties.

    ```
    azure.keyvault.enabled=true
    azure.keyvault.uri=https://<your-keyvault-name>.vault.azure.net
    ```
    > [!Note] 
    > Debe agregar la dirección URL del almacén de claves en `application.properties`, como se indicó anteriormente. De lo contrario, es posible que la dirección URL del almacén de claves no se capture durante la ejecución.

3. Agregue el ejemplo de código a src/main/java/com/example/demo/DemoApplication.java. Recupera la cadena de conexión del almacén de claves. 

    ```Java
    package com.example.demo;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;

    @SpringBootApplication
    @RestController
    public class DemoApplication implements CommandLineRunner {

        @Value("${connectionString}")
        private String connectionString;

        public static void main(String[] args) {
          SpringApplication.run(DemoApplication.class, args);
        }

        @GetMapping("get")
        public String get() {
          return connectionString;
        }

        public void run(String... varl) throws Exception {
          System.out.println(String.format("\nConnection String stored in Azure Key Vault:\n%s\n",connectionString));
        }
      }
    ```

    Si abre el archivo pom.xml, verá la dependencia de `azure-keyvault-secrets-spring-boot-starter`. Agregue esta dependencia al proyecto en el archivo pom.xml. 

    ```xml
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-keyvault-secrets-spring-boot-starter</artifactId>
    </dependency>
    ```

4. Empaquete la aplicación de ejemplo. 

    ```azurecli
    mvn clean package
    ```

5. Ahora puede implementar la aplicación en Azure con el comando `az spring-cloud app deploy` de la CLI de Azure. 

    ```azurecli
    az spring-cloud app deploy -n "springapp" -s "myspringcloud" -g "myResourceGroup" --jar-path target/demo-0.0.1-SNAPSHOT.jar
    ```

6.  Para probar la aplicación, acceda al punto de conexión público o al punto de conexión de prueba.

    ```azurecli
    curl https://myspringcloud-springapp.azuremicroservices.io/get
    ```

    Verá un mensaje similar a "Se ha conseguido correctamente el valor de connectionString del almacén de claves https://<suNombreDeAlmacénDeClaves>.vault.azure.net/: jdbc:sqlserver://SERVER.database.windows.net:1433;database=DATABASE;".

## <a name="build-sample-spring-boot-app-with-java-sdk"></a>Compilación de una aplicación de Spring Boot de ejemplo con el SDK de Java

Este ejemplo puede establecer y obtener secretos desde Azure Key Vault. La [biblioteca de cliente Azure Key Vault Secret para Java](https://docs.microsoft.com/java/api/overview/azure/security-keyvault-secrets-readme?view=azure-java-stablelibrary&preserve-view=true) proporciona compatibilidad con la autenticación de tokens de Azure Active Directory en el SDK de Azure. Proporciona un conjunto de implementaciones de **TokenCredential** que se pueden usar para construir clientes del SDK de Azure para admitir la autenticación de tokens de AAD.

La biblioteca de cliente de secretos de Azure Key Vault permite almacenar y controlar de forma segura el acceso a los tokens, las contraseñas, las claves de API y otros secretos. La biblioteca ofrece operaciones para crear, recuperar, actualizar, eliminar, purgar, realizar copias de seguridad, restaurar y enumerar los secretos y sus versiones.

1. Clone un proyecto de ejemplo. 

    ```azurecli
    git clone https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples.git
    ```

2. Especifique el almacén de claves en la aplicación. 

    ```azurecli
    cd Azure-Spring-Cloud-Samples/managed-identity-keyvault
    vim src/main/resources/application.properties
    ```

    Para usar la identidad administrada para las aplicaciones de Azure Spring Cloud, agregue propiedades con el siguiente contenido a *src/main/resources/application.properties*.

    ```
    azure.keyvault.enabled=true
    azure.keyvault.uri=https://<your-keyvault-name>.vault.azure.net
    ```

3. Incluya [ManagedIdentityCredentialBuilder](https://docs.microsoft.com/java/api/com.azure.identity.managedidentitycredentialbuilder?view=azure-java-stable&preserve-view=true) para obtener el token de Azure Active Directory y [SecretClientBuilder ](https://docs.microsoft.com/java/api/com.azure.security.keyvault.secrets.secretclientbuilder?view=azure-java-stable&preserve-view=true) para establecer u obtener secretos de Key Vault en el código.

    Obtenga el ejemplo de [MainController.Java](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/blob/master/managed-identity-keyvault/src/main/java/com/microsoft/azure/MainController.java#L28) del proyecto de ejemplo clonado.

    Incluya también `azure-identity` y `azure-security-keyvault-secrets` como dependencia en el archivo pom.xml. Obtenga el ejemplo de [pom.xml](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/blob/master/managed-identity-keyvault/pom.xml#L21) del proyecto de ejemplo clonado. 

4. Empaquete la aplicación de ejemplo. 

    ```azurecli
    mvn clean package
    ```

5. Ahora implemente la aplicación en Azure con el comando `az spring-cloud app deploy` de la CLI de Azure. 

    ```azurecli
    az spring-cloud app deploy -n "springapp" -s "myspringcloud" -g "myResourceGroup" --jar-path target/asc-managed-identity-keyvault-sample-0.1.0.jar
    ```

6. Acceda al punto de conexión público o al punto de conexión de prueba para probar la aplicación. 

    En primer lugar, obtenga el valor de su secreto que estableció en Azure Key Vault. 
    ```azurecli
    curl https://myspringcloud-springapp.azuremicroservices.io/secrets/connectionString
    ```

    Verá un mensaje similar a "Se ha conseguido correctamente el valor de connectionString del almacén de claves https://<suNombreDeAlmacénDeClaves>.vault.azure.net/: jdbc:sqlserver://SERVER.database.windows.net:1433;database=DATABASE;".

    Ahora cree un secreto y luego recupérelo mediante el SDK de Java. 
    ```azurecli
    curl -X PUT https://myspringcloud-springapp.azuremicroservices.io/secrets/test?value=success

    curl https://myspringcloud-springapp.azuremicroservices.io/secrets/test
    ```

    Verá un mensaje similar a "Se obtuvo correctamente el valor de la prueba secreta desde Key Vault https://<suNombreDeAlmacénDeClaves>.vault.azure.net: correcto". 

## <a name="next-steps"></a>Pasos siguientes

* [Procedimiento para obtener acceso al blob de almacenamiento con identidad administrada en Azure Spring Cloud](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/managed-identity-storage-blob)
* [Habilitación de la identidad administrada asignada por el sistema para una aplicación de Azure Spring Cloud](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-howto-enable-system-assigned-managed-identity)
* [Más información sobre las identidades administradas para recursos de Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory/managed-identities-azure-resources/overview.md)
* [Autenticación de Azure Spring Cloud con Key Vault en Acciones de GitHub](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-github-actions-key-vault)

