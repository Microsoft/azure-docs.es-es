---
title: 'Creación de una aplicación web de Java en Linux: Azure App Service'
description: En esta guía de inicio rápido, implementará su primera aplicación Hola mundo de Java en Azure App Service en Linux en cuestión de minutos.
keywords: azure, app service, aplicación web, linux, java, maven, inicio rápido
services: app-service\web
documentationcenter: ''
author: msangapu
manager: jeconnoc
editor: ''
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: quickstart
ms.date: 03/27/2019
ms.author: msangapu
ms.custom: mvc, seo-java-july2019, seo-java-august2019, seo-java-september2019
ms.openlocfilehash: 2d486a5e181e9131ef7a1e91f52018fb2be82dc1
ms.sourcegitcommit: 1c9858eef5557a864a769c0a386d3c36ffc93ce4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/18/2019
ms.locfileid: "71105240"
---
# <a name="quickstart-create-a-java-app-on-azure-app-service-on-linux"></a>Inicio rápido: Creación de una aplicación de Java en Azure App Service en Linux

[App Service en Linux](app-service-linux-intro.md), proporciona un servicio de hospedaje web muy escalable y con aplicación automática de revisiones que usa el sistema operativo Linux. En este inicio rápido se muestra cómo usar la [CLI de Azure](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) con el [complemento Maven para Azure App Service](https://github.com/Microsoft/azure-maven-plugins/tree/develop/azure-webapp-maven-plugin) para implementar un archivo web (WAR) de Java en el sistema operativo Linux.

> [!NOTE]
>
> También se puede hacer lo mismo con IDE populares, como IntelliJ y Eclipse. Consulte nuestros documentos similares en [Inicio rápido de Azure Toolkit for IntelliJ](/java/azure/intellij/azure-toolkit-for-intellij-create-hello-world-web-app) o [Inicio rápido de Azure Toolkit for Eclipse](/java/azure/eclipse/azure-toolkit-for-eclipse-create-hello-world-web-app).
>
![Aplicación de ejemplo que se ejecuta en Azure App Service](media/quickstart-java/java-hello-world-in-browser-azure-app-service.png)

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-java-app"></a>Creación de una aplicación Java

Ejecute el siguiente comando de Maven en el símbolo del sistema de Cloud Shell para crear una aplicación denominada `helloworld`:

```bash
mvn archetype:generate -DgroupId=example.demo -DartifactId=helloworld -DarchetypeArtifactId=maven-archetype-webapp
```

## <a name="configure-the-maven-plugin"></a>Configuración del complemento Maven

Para implementar desde Maven, use el editor de código de Cloud Shell para abrir el archivo de proyecto `pom.xml` en el directorio `helloworld`. 

```bash
code pom.xml
```

Luego agregue la siguiente definición del complemento al elemento `<build>` del archivo `pom.xml`.

```xml
<plugins>
    <!--*************************************************-->
    <!-- Deploy to Tomcat in App Service Linux           -->
    <!--*************************************************-->
    <plugin>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-webapp-maven-plugin</artifactId>
        <version>1.7.0</version>       
    </plugin>
</plugins>
```

El proceso de implementación en Azure App Service usa las credenciales de cuenta desde la CLI de Azure. [Inicie sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli?view=azure-cli-latest) antes de continuar.

```azurecli
az login
```

Luego puede configurar la implementación, ejecutar el comando `mvn azure-webapp:config` de Maven en el símbolo del sistema y usar las configuraciones predeterminadas presionando **ENTRAR** hasta recibir la solicitud **Confirm (Y/N)** [Confirmar (S/N)], donde deberá presionar **"y"** para completar la configuración.

```cmd
~@Azure:~/helloworld$ mvn azure-webapp:config
[INFO] Scanning for projects...
[INFO]
[INFO] ----------------------< example.demo:helloworld >-----------------------
[INFO] Building helloworld Maven Webapp 1.0-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO]
[INFO] --- azure-webapp-maven-plugin:1.7.0:config (default-cli) @ helloworld ---
[WARNING] The plugin may not work if you change the os of an existing webapp.
Define value for OS(Default: Linux):
1. linux [*]
2. windows
3. docker
Enter index to use:
Define value for javaVersion(Default: jre8):
1. jre8 [*]
2. java11
Enter index to use:
Define value for runtimeStack(Default: TOMCAT 8.5):
1. TOMCAT 9.0
2. jre8
3. TOMCAT 8.5 [*]
4. WILDFLY 14
Enter index to use:
Please confirm webapp properties
AppName : helloworld-1558400876966
ResourceGroup : helloworld-1558400876966-rg
Region : westeurope
PricingTier : Premium_P1V2
OS : Linux
RuntimeStack : TOMCAT 8.5-jre8
Deploy to slot : false
Confirm (Y/N)? : Y
```

> [!NOTE]
> En este artículo solo se trabaja con aplicaciones Java empaquetadas en archivos WAR. El complemento también admite aplicaciones web JAR, visite [Implementación de un archivo JAR de SE de Java en App Service en Linux](https://docs.microsoft.com/java/azure/spring-framework/deploy-spring-boot-java-app-with-maven-plugin?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json) para probarlo.

Vaya nuevamente a `pom.xml` para ver si se actualizó la configuración del complemento. Puede modificar otras configuraciones de App Service directamente en el archivo POM si es necesario. Algunas configuraciones comunes se muestran a continuación:

 Propiedad | Obligatorio | DESCRIPCIÓN | Versión
---|---|---|---
`<schemaVersion>` | false | Especifique la versión del esquema de configuración. Los valores admitidos son: `v1`, `v2`. | 1.5.2
`<resourceGroup>` | true | Grupo de recursos de Azure para la aplicación web. | 0.1.0+
`<appName>` | true | El nombre de la aplicación web. | 0.1.0+
[`<region>`](/java/api/overview/azure/maven/azure-webapp-maven-plugin/readme#region) | true | Especifica la región donde se hospedará la aplicación web. El valor predeterminado es **westus**. Todas las regiones válidas en la sección [Regiones admitidas](/java/api/overview/azure/maven/azure-webapp-maven-plugin/readme#region). | 0.1.0+
[`<pricingTier>`](/java/api/overview/azure/maven/azure-webapp-maven-plugin/readme##pricingtier) | false | El plan de tarifa de la aplicación web. El valor predeterminado es **P1V2**.| 0.1.0+
[`<runtime>`](/java/api/overview/azure/maven/azure-webapp-maven-plugin/readme#runtimesetting) | true | La configuración del entorno en tiempo de ejecución. Puede ver los detalles [aquí](/java/api/overview/azure/maven/azure-webapp-maven-plugin/readme#runtimesetting). | 0.1.0+
[`<deployment>`](/java/api/overview/azure/maven/azure-webapp-maven-plugin/readme#deploymentsetting) | true | La configuración de implementación. Puede ver los detalles [aquí](/java/api/overview/azure/maven/azure-webapp-maven-plugin/readme#deploymentsetting). | 0.1.0+

## <a name="deploy-the-app"></a>Implementar la aplicación

Implemente la aplicación Java en Azure mediante el siguiente comando:

```bash
mvn package azure-webapp:deploy
```

Una vez que se haya completado la implementación, vaya a la aplicación implementada mediante la siguiente dirección URL en el explorador web, por ejemplo, `http://<webapp>.azurewebsites.net`. 

![Aplicación de ejemplo que se ejecuta en Azure App Service](media/quickstart-java/java-hello-world-in-browser-azure-app-service.png)

**¡Enhorabuena!** Ha implementado su primera aplicación Java en App Service en Linux.

## <a name="clean-up-resources"></a>Limpieza de recursos

En los pasos anteriores, creó recursos de Azure en un grupo de recursos. Si prevé que no necesitará estos recursos en el futuro, elimine el grupo de recursos ejecutando el siguiente comando en Cloud Shell:

```azurecli-interactive
az group delete --name <your resource group name; for example: helloworld-1558400876966-rg> --yes
```

Este comando puede tardar varios segundos en ejecutarse.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Aplicación Java Enterprise con PostgreSQL](tutorial-java-enterprise-postgresql-app.md)

> [!div class="nextstepaction"]
> [Configuración de una aplicación Java](configure-custom-container.md)

> [!div class="nextstepaction"]
> [CI/CD con Jenkins](/azure/jenkins/deploy-jenkins-app-service-plugin)

> [!div class="nextstepaction"]
> [Otros recursos de Azure para desarrolladores de Java](/java/azure/)
