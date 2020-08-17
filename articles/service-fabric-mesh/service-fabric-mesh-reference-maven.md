---
title: Referencia de Maven para Azure Service Fabric Mesh
description: Contiene la referencia sobre cómo usar el complemento de Maven para Service Fabric Mesh
author: suhuruli
ms.author: suhuruli
ms.date: 11/26/2018
ms.topic: reference
ms.custom: devx-track-java
ms.openlocfilehash: 3a1aa004f47ba700ef4b96004dfe5b835788dcc7
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372474"
---
# <a name="maven-plugin-for-service-fabric-mesh"></a>Complemento de Maven para Service Fabric Mesh

## <a name="prerequisites"></a>Prerequisites

- SDK de Java
- Maven
- CLI de Azure con la extensión de malla
- CLI de Service Fabric

## <a name="goals"></a>Objetivos

### `azure-sfmesh:init`
- Crea una carpeta `servicefabric` que contiene una carpeta `appresources` que tiene el `application.yaml` archivo. 

### `azure-sfmesh:addservice`
- Crea una carpeta dentro de la carpeta `servicefabric` con el nombre del servicio y crea el archivo YAML del servicio. 

### `azure-sfmesh:addnetwork`
- Genera un código YAML `network` con el nombre de red proporcionado en la carpeta `appresources` 

### `azure-sfmesh:addgateway`
- Genera un código YAML `gateway` con el nombre de puerta de enlace proporcionado en la carpeta `appresources` 

#### `azure-sfmesh:addvolume`
- Genera un código YAML `volume` con el nombre de volumen proporcionado en la carpeta `appresources`.

### `azure-sfmesh:addsecret`
- Genera un código YAML `secret` con el nombre de secreto proporcionado en la carpeta `appresources` 

### `azure-sfmesh:addsecretvalue`
- Genera un código YAML `secretvalue` con el secreto proporcionado y el nombre de valor de secreto en la carpeta `appresources` 

### `azure-sfmesh:deploy`
- Combina los códigos YAML desde la carpeta `servicefabric` y crea un JSON de plantilla de Azure Resource Manager en la carpeta actual.
- Implementa todos los recursos en el entorno de Azure Service Fabric Mesh 

### `azure-sfmesh:deploytocluster`
- Crea una carpeta (`meshDeploy`) que contiene los JSON de implementación generados a partir de códigos YAML que son aplicables para los clústeres de Service Fabric
- Implementa todos los recursos en el clúster de Service Fabric
 

## <a name="usage"></a>Uso

Para usar el complemento de Maven en la aplicación Java de Maven, agrega el siguiente fragmento al archivo pom.xml:

```XML
<project>
  ...
  <build>
    ...
    <plugins>
      ...
      <plugin>
        <groupId>com.microsoft.azure</groupId>
          <artifactId>azure-sfmesh-maven-plugin</artifactId>
          <version>0.1.0</version>
      </plugin>
    </plugins>
  </build>
</project>
```

## <a name="common-configuration"></a>Configuración común

El complemento de Maven actualmente no admite configuraciones comunes de los complementos de Maven para Azure.

## <a name="how-to"></a>Procedimiento

### <a name="initialize-maven-project-for-azure-service-fabric-mesh"></a>Inicialización del proyecto de Maven para Azure Service Fabric Mesh
Ejecute el siguiente comando para crear el archivo YAML de recursos de aplicación.

```cmd
mvn azure-sfmesh:init -DapplicationName=helloworldserver
```

- Crea una carpeta denominada `servicefabric->appresources` en la carpeta raíz que contiene una un código YAML de aplicación que se denomina `app_helloworldserver`

### <a name="add-resource-to-your-application"></a>Incorporación de un recurso a la aplicación

#### <a name="add-a-new-network-to-your-application"></a>Incorporación de una nueva red a la aplicación
Ejecute el comando siguiente para crear un código YAML de recursos de red. 

```cmd
mvn azure-sfmesh:addnetwork -DnetworkName=helloworldservicenetwork -DnetworkAddressPrefix=10.0.0.0/22
```

- Crea un código YAML de red en la carpeta `servicefabric->appresources` que se denomina `network_helloworldservicenetwork`

#### <a name="add-a-new-service-to-your-application"></a>Incorporación de un nuevo servicio a la aplicación
Ejecute el comando siguiente para crear un código YAML de servicio. 

```cmd
mvn azure-sfmesh:addservice -DapplicationName=helloworldserver -DserviceName=helloworldservice -DimageName=helloworldserver:latest -DlistenerPort=8080 -DnetworkRef=helloworldservicenetwork
```

- Crea un servicio YAML en la carpeta `servicefabric->helloworldservice` denominado `service_helloworldservice` que hace referencia a `helloworldservicenetwork` y a la aplciación `helloworldserver`
- El servicio escucharía en el puerto 8080
- El servicio utilizaría ***helloworldserver:latest*** como su imagen de contenedor.

#### <a name="add-a-new-gateway-resource-to-your-application"></a>Incorporación de un nuevo recurso de puerta de enlace a la aplicación
Ejecute el comando siguiente para crear un código YAML de recursos de puerta de enlace. 

```cmd
mvn azure-sfmesh:addgateway -DapplicationName=helloworldserver -DdestinationNetwork=helloworldservicenetwork -DgatewayName=helloworldgateway -DlistenerName=helloworldserviceListener -DserviceName=helloworldservice -DsourceNetwork=open -DtcpPort=80
```

- Crea un nuevo código YAML de puerta de enlace en la carpeta `servicefabric->appresources` que se denomina `gateway_helloworldgateway`
- Hace referencia `helloworldservicelistener` como el agente de escucha del servicio que está escuchando las llamadas de esta puerta de enlace. También hace referencia a `helloworldservice` como el servicio, `helloworldservicenetwork` como la red y `helloworldserver` como la aplicación. 
- Escucha las solicitudes en el puerto 80

#### <a name="add-a-new-volume-to-your-application"></a>Incorporación de un nuevo volumen a la aplicación
Ejecute el comando siguiente para crear un código YAML de recursos de volumen. 

```cmd
mvn azure-sfmesh:addvolume -DvolumeAccountKey=key -DvolumeAccountName=name -DvolumeName=vol1 -DvolumeShareName=share
```

- Crea un código YAML de volumen en la carpeta `servicefabric->appresources` que se denomina `volume_vol1`
- Establece las propiedades de los parámetros obligatorios, `volumeAccountKey`, y `volumeShareName` como anteriormente
- Para más información sobre cómo hacer referencia a este volumen creado, visite [Almacenamiento del estado en una aplicación Azure Service Fabric Mesh mediante el montaje de un volumen basado en Azure Files dentro del contenedor](service-fabric-mesh-howto-deploy-app-azurefiles-volume.md)

#### <a name="add-a-new-secret-resource-to-your-application"></a>Incorporación de un nuevo recurso de secreto a la aplicación
Ejecute el comando siguiente para crear un código YAML de recursos de secreto. 

```cmd
mvn azure-sfmesh:addsecret -DsecretName=secret1
```

- Crea un código YAML de secreto en la carpeta `servicefabric->appresources` que se denomina `secret_secret1`
- Para más información sobre cómo hacer referencia a este secreto creado, visite [Administración de secretos en aplicaciones de Service Fabric Mesh](service-fabric-mesh-howto-manage-secrets.md)

#### <a name="add-a-new-secretvalue-resource-to-your-application"></a>Incorporación de un nuevo recurso de secretvalue a la aplicación
Ejecute el comando siguiente para crear un código YAML de recursos de secretvalue. 

```cmd
mvn azure-sfmesh:addsecretvalue -DsecretValue=someVal -DsecretValueName=secret1/v1
```

- Creación de un código YAML de secretvalue en la carpeta `servicefabric->appresources` que se denomina `secretvalue_secret1_v1`

### <a name="run-the-application-locally"></a>Ejecución de la aplicación de forma local

Con la ayuda de `azure-sfmesh:deploytocluster` de objetivo, puede ejecutar la aplicación localmente mediante el comando siguiente:

```cmd
mvn azure-sfmesh:deploytocluster
```

De forma predeterminada este objetivo implementa recursos en clúster local. Si va a implementar en el clúster local, se supone que tiene un clúster de Service Fabric local en funcionamiento. El clúster de Service Fabric local para los recursos actualmente solo se admite en [Windows](service-fabric-mesh-howto-setup-developer-environment-sdk.md).

- Crea JSON desde códigos YAML que son aplicables para los clústeres de Service Fabric
- A continuación, se implementa en el punto de conexión del clúster

### <a name="deploy-application-to-azure-service-fabric-mesh"></a>Implementación de una aplicación en Azure Service Fabric Mesh

Con la ayuda de `azure-sfmesh:deploy` de objetivo, puede implementar en el entorno de Service Fabric Mesh ejecutando el comando siguiente:

```cmd
mvn azure-sfmesh:deploy -DresourceGroup=todoapprg -Dlocation=eastus
```

- Crea un grupo de recursos denominado `todoapprg` si no existe.
- Crea un JSON de plantilla de Azure Resource Manager mediante la combinación de códigos YAML. 
- Implementa JSON en el entorno de Azure Service Fabric Mesh.
