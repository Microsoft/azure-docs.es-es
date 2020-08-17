---
title: CI/CD de Azure Spring Cloud con Acciones de GitHub
description: Creación de un flujo de trabajo de CI/CD para Azure Spring Cloud con Acciones de GitHub
author: MikeDodaro
ms.author: barbkess
ms.service: spring-cloud
ms.topic: how-to
ms.date: 01/15/2019
ms.custom: devx-track-java
ms.openlocfilehash: 960e480bc3271a24f1fa898e1af6c55bcaaada7b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87037566"
---
# <a name="azure-spring-cloud-cicd-with-github-actions"></a>CI/CD de Azure Spring Cloud con Acciones de GitHub

Acciones de GitHub admite un flujo de trabajo del ciclo de vida de desarrollo de software automatizado. Con Acciones de GitHub para Azure Spring Cloud puede crear flujos de trabajo en el repositorio para compilar, probar, empaquetar, publicar e implementar en Azure. 

## <a name="prerequisites"></a>Requisitos previos
En este ejemplo se requiere la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="set-up-github-repository-and-authenticate"></a>Configuración y autenticación de repositorios de GitHub
Para autorizar la acción de inicio de sesión de Azure se necesita una credencial de entidad de servicio de Azure. Para obtener una credencial de Azure, ejecute los siguientes comandos en una máquina local:
```
az login
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID> --sdk-auth 
```
Para acceder a un grupo de recursos concreto, puede reducir el ámbito:
```
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP> --sdk-auth
```
El comando debe generar un objeto JSON:
```JSON
{
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    ...
}
```

Aquí se usa el ejemplo de [Piggy Metrics](https://github.com/Azure-Samples/piggymetrics) de GitHub.  Bifurque el ejemplo, abra la página del repositorio de GitHub y haga clic en la pestaña **Settings** (Configuración). Abra el menú **Secrets** (Secretos) y haga clic en **Add a new secret** (Agregar un nuevo secreto):

 ![Agregar un nuevo secreto](./media/github-actions/actions1.png)

Establezca el nombre del secreto en `AZURE_CREDENTIALS` y su valor en la cadena JSON que encontró en el encabezado *Configuración y autenticación del repositorio de GitHub*.

 ![Establecer datos de secreto](./media/github-actions/actions2.png)

También puede obtener la credencial de inicio de sesión de Azure desde Key Vault en Acciones de GitHub, como se explica en [Autenticación de Azure Spring con Key Vault en Acciones de GitHub](./spring-cloud-github-actions-key-vault.md).

## <a name="provision-service-instance"></a>Aprovisionamiento de una instancia de servicio
Para aprovisionar una instancia del servicio Azure Spring Cloud, ejecute los siguientes comandos mediante la CLI de Azure.
```
az extension add --name spring-cloud
az group create --location eastus --name <resource group name>
az spring-cloud create -n <service instance name> -g <resource group name>
az spring-cloud config-server git set -n <service instance name> --uri https://github.com/xxx/piggymetrics --label config
```
## <a name="build-the-workflow"></a>Compilación del flujo de trabajo
El flujo de trabajo se define con las siguientes opciones.

### <a name="prepare-for-deployment-with-azure-cli"></a>Preparación de la implementación con la CLI de Azure
Actualmente, el comando `az spring-cloud app create` no es idempotente.  Este flujo de trabajo se recomienda en las instancias y aplicaciones de Azure Spring Cloud existentes.

Use los siguientes comandos de la CLI de Azure para la preparación:
```
az configure --defaults group=<service group name>
az configure --defaults spring-cloud=<service instance name>
az spring-cloud app create --name gateway
az spring-cloud app create --name auth-service
az spring-cloud app create --name account-service
```

### <a name="deploy-with-azure-cli-directly"></a>Implementación directa con la CLI de Azure
Cree el archivo de `.github/workflow/main.yml` en el repositorio:

```
name: AzureSpringCloud
on: push

env:
  GROUP: <resource group name>
  SERVICE_NAME: <service instance name>

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    
    - uses: actions/checkout@master
    
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    
    - name: maven build, clean
      run: |
        mvn clean package -DskipTests
    
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
      
    - name: Install ASC AZ extension
      run: az extension add --name spring-cloud
   
    - name: Deploy with AZ CLI commands
      run: |
        az configure --defaults group=$GROUP
        az configure --defaults spring-cloud=$SERVICE_NAME
        az spring-cloud app deploy -n gateway --jar-path ${{ github.workspace }}/gateway/target/gateway.jar
        az spring-cloud app deploy -n account-service --jar-path ${{ github.workspace }}/account-service/target/account-service.jar
        az spring-cloud app deploy -n auth-service --jar-path ${{ github.workspace }}/auth-service/target/auth-service.jar
```
### <a name="deploy-with-azure-cli-action"></a>Implementación con una acción de la CLI de Azure
El comando az `run` usará la versión más reciente de la CLI de Azure. Si hay cambios importantes, también puede usar una versión específica de la CLI de Azure con azure/CLI `action`. 

> [!Note] 
> Este comando se ejecutará en un contenedor nuevo, por lo que `env` no funcionará y es posible que el acceso a archivos entre acciones tenga restricciones adicionales.

Cree el archivo .github/workflow/main.yml en el repositorio:
```
name: AzureSpringCloud
on: push

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    
    - uses: actions/checkout@master
    
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    
    - name: maven build, clean
      run: |
        mvn clean package -DskipTests
        
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
              
    - name: Azure CLI script
      uses: azure/CLI@v1
      with:
        azcliversion: 2.0.75
        inlineScript: |
          az extension add --name spring-cloud
          az configure --defaults group=<service group name>
          az configure --defaults spring-cloud=<service instance name>
          az spring-cloud app deploy -n gateway --jar-path $GITHUB_WORKSPACE/gateway/target/gateway.jar
          az spring-cloud app deploy -n account-service --jar-path $GITHUB_WORKSPACE/account-service/target/account-service.jar
          az spring-cloud app deploy -n auth-service --jar-path $GITHUB_WORKSPACE/auth-service/target/auth-service.jar
```

## <a name="deploy-with-maven-plugin"></a>Implementación con el complemento Maven
Otra opción es usar el [complemento Maven](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-quickstart-launch-app-maven) para implementar el archivo .jar y actualizar la configuración de la aplicación. El comando `mvn azure-spring-cloud:deploy` es idempotente y creará automáticamente las aplicaciones si fuera necesario. No es preciso crear las aplicaciones correspondientes de antemano.

```
name: AzureSpringCloud
on: push

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    
    - uses: actions/checkout@master
    
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    
    - name: maven build, clean
      run: |
        mvn clean package -DskipTests
        
    # Maven plugin can cosume this authentication method automatically
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    # Maven deploy, make sure you have correct configurations in your pom.xml
    - name: deploy to Azure Spring Cloud using Maven
      run: |
        mvn azure-spring-cloud:deploy
```

## <a name="run-the-workflow"></a>Ejecución del flujo de trabajo
**Acciones** de GitHub se debe habilitar automáticamente después de insertar `.github/workflow/main.yml` en GitHub. La acción se desencadenará al insertar una nueva confirmación. Si crea este archivo en el explorador, la acción ya debería haberse ejecutado.

Para comprobar que la acción se ha habilitado, haga clic en la pestaña **Actions** (Acciones) de la página del repositorio de GitHub:

 ![Comprobar acción habilitado](./media/github-actions/actions3.png)

Si al ejecutarse la acción aparece un error, por ejemplo, si no ha establecido la credencial de Azure, puede volver a ejecutar las comprobaciones después de corregir el error. En la página del repositorio de GitHub, haga clic en **Actions** (Acciones), seleccione la tarea del flujo de trabajo específica y, después, haga clic en el botón **Rerun checks** (Volver a ejecutar comprobaciones) para volver a ejecutar las comprobaciones:

 ![Volver a ejecutar comprobaciones](./media/github-actions/actions4.png)

## <a name="next-steps"></a>Pasos siguientes
* [Key Vault para Acciones de GitHub en Spring Cloud](./spring-cloud-github-actions-key-vault.md)
* [Entidades de servicio de Azure Active Directory](https://docs.microsoft.com/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac)
* [Acciones de GitHub para Azure](https://github.com/Azure/actions/)
