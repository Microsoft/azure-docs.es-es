---
title: Migración del SDK de Java a Maven
description: Actualice las aplicaciones de Java anteriores que usaban el SDK de Java de Service Fabric para capturar las dependencias de Java de Service Fabric desde Maven. Después de completar esta instalación, sus aplicaciones de Java anteriores podrían compilarse.
author: rapatchi
ms.topic: conceptual
ms.date: 08/23/2017
ms.custom: devx-track-java
ms.author: rapatchi
ms.openlocfilehash: f65194cde25deac8c03d6ddc3a7c0b928aaca171
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327317"
---
# <a name="update-your-previous-java-service-fabric-application-to-fetch-java-libraries-from-maven"></a>Actualice la aplicación de Java de Service Fabric anterior para capturar las bibliotecas de Java desde Maven
Los archivos binarios de Java de Service Fabric se han movido desde el SDK de Java de Service Fabric al hospedaje de Maven. Puede usar **mavencentral** para capturar las dependencias de Java de Service Fabric más recientes. Esta guía le ayudará a actualizar las aplicaciones de Java existentes creadas para el SDK de Java de Service Fabric, utilizando cualquier plantilla Yeoman o Eclipse, para que sean compatibles con la compilación basada en Maven.

## <a name="prerequisites"></a>Prerrequisitos

1. En primer lugar, desinstale el SDK de Java existente.

   ```bash
   sudo dpkg -r servicefabricsdkjava
   ```

2. Instale la CLI de Service Fabric más reciente siguiendo los pasos mencionados [aquí](service-fabric-cli.md).

3. Para compilar las aplicaciones de Java de Service Fabric y trabajar con ellas, asegúrese de que tiene JDK 1.8 y Gradle instalados. Si aún no se han instalado, puede ejecutar el siguiente procedimiento para instalar JDK 1.8 (openjdk-8-jdk) y Gradle:

   ```bash
   sudo apt-get install openjdk-8-jdk-headless
   sudo apt-get install gradle
   ```

4. Actualice los scripts de instalación o desinstalación de la aplicación para utilizar la nueva CLI de Service Fabric siguiendo los pasos mencionados [aquí](service-fabric-application-lifecycle-sfctl.md). Puede hacer referencia a nuestros [ejemplos](https://github.com/Azure-Samples/service-fabric-java-getting-started) de introducción como referencia.

>[!TIP]
> Después de desinstalar el SDK de Java de Service Fabric, Yeoman no funcionará. Siga los requisitos previos mencionados [aquí](service-fabric-create-your-first-linux-application-with-java.md) para que el generador de plantillas Java de Yeoman de Service Fabric funcione.

## <a name="service-fabric-java-libraries-on-maven"></a>Bibliotecas de Java de Service Fabric en Maven

Las bibliotecas de Java de Service Fabric han sido hospedadas en Maven. Puede agregar las dependencias en el archivo ``pom.xml`` o ``build.gradle`` de sus proyectos para usar las bibliotecas de Java de Service Fabric desde **mavenCentral**.

### <a name="actors"></a>Actores

Compatibilidad con Reliable Actor de Service Fabric para la aplicación.

  ```XML
  <dependency>
      <groupId>com.microsoft.servicefabric</groupId>
      <artifactId>sf-actors</artifactId>
      <version>1.0.0</version>
  </dependency>
  ```

  ```gradle
  repositories {
      mavenCentral()
  }
  dependencies {
      compile 'com.microsoft.servicefabric:sf-actors:1.0.0'
  }
  ```

### <a name="services"></a>Servicios

Compatibilidad con el servicio sin estado de Service Fabric para la aplicación.

  ```XML
  <dependency>
      <groupId>com.microsoft.servicefabric</groupId>
      <artifactId>sf-services</artifactId>
      <version>1.0.0</version>
  </dependency>
  ```

  ```gradle
  repositories {
      mavenCentral()
  }
  dependencies {
      compile 'com.microsoft.servicefabric:sf-services:1.0.0'
  }
  ```

### <a name="others"></a>Otros

#### <a name="transport"></a>Transporte

Compatibilidad de la capa de transporte para aplicaciones Java de Service Fabric. No es necesario agregar explícitamente esta dependencia a sus aplicaciones Reliable Actor o de servicio, a menos que programe la capa de transporte.

  ```XML
  <dependency>
      <groupId>com.microsoft.servicefabric</groupId>
      <artifactId>sf-transport</artifactId>
      <version>1.0.0</version>
  </dependency>
  ```

  ```gradle
  repositories {
      mavenCentral()
  }
  dependencies {
      compile 'com.microsoft.servicefabric:sf-transport:1.0.0'
  }
  ```

#### <a name="fabric-support"></a>Compatibilidad con Fabric

Compatibilidad en el nivel de sistema para Service Fabric, que se comunica con el entorno de tiempo de ejecución nativo de Service Fabric. No es necesario agregar explícitamente esta dependencia a sus aplicaciones Reliable Actor o de servicio. Se recuperan automáticamente desde Maven, cuando se incluyen las dependencias anteriores.

  ```XML
  <dependency>
      <groupId>com.microsoft.servicefabric</groupId>
      <artifactId>sf</artifactId>
      <version>1.0.0</version>
  </dependency>
  ```

  ```gradle
  repositories {
      mavenCentral()
  }
  dependencies {
      compile 'com.microsoft.servicefabric:sf:1.0.0'
  }
  ```

## <a name="migrating-service-fabric-stateless-service"></a>Migración del servicio sin estado de Service Fabric

Para poder crear su servicio de Java sin estado de Service Fabric existente con las dependencias de Service Fabric capturadas desde Maven, debe actualizar el archivo ``build.gradle`` dentro del servicio. Antes solían ser como las siguientes:

```gradle
dependencies {
    compile fileTree(dir: '/opt/microsoft/sdk/servicefabric/java/packages/lib', include: ['*.jar'])
    compile project(':Interface')
}
.
.
.
jar {
    manifest {
    attributes(
                'Main-Class': 'statelessservice.MyStatelessServiceHost',
                "Class-Path": configurations.compile.collect { 'lib/' + it.getName() }.join(' '))
    baseName "MyStateless"
    destinationDir = file('./../MyStatelessApplication/MyStatelessPkg/Code')
}
.
.
.
task copyDeps <<{
    copy {
        from("/opt/microsoft/sdk/servicefabric/java/packages/lib")
        into("./../MyStatelessApplication/MyStatelessPkg/Code/lib")
        include('*.jar')
    }
    copy {
        from("/opt/microsoft/sdk/servicefabric/java/packages/lib")
        into("./../MyStatelessApplication/MyStatelessPkg/Code/lib")
        include('libj*.so')
    }
}
```

Ahora, para capturar las dependencias desde Maven, el archivo **actualizado** ``build.gradle`` tendría los elementos correspondientes como se indica a continuación:

```gradle
repositories {
        mavenCentral()
}

configurations {
    azuresf
}

dependencies {
    compile project(':Interface')
    azuresf ('com.microsoft.servicefabric:sf-services:1.0.0')
    compile fileTree(dir: 'lib', include: '*.jar')
}

task explodeDeps(type: Copy, dependsOn:configurations.azuresf) { task ->
    configurations.azuresf.filter { it.toString().contains("native") }.each{
        from zipTree(it)
    }
    configurations.azuresf.filter { !it.toString().contains("native") }.each {
        from it
    }
    into "lib"
    include "libj*.so", "*.jar"
}

compileJava.dependsOn(explodeDeps)
.
.
.
jar {
    manifest {
        def mpath = configurations.compile.collect {'lib/'+it.getName()}.join (' ')
        mpath = mpath + ' ' + configurations.azuresf.collect {'lib/'+it.getName()}.join (' ')
        attributes(
                'Main-Class': 'statelessservice.MyStatelessServiceHost',
                "Class-Path": mpath)
    baseName "MyStateless"
    destinationDir = file('./../MyStatelessApplication/MyStatelessPkg/Code')
   }
}
.
.
.
task copyDeps <<{
    copy {
        from("lib/")
        into("./../MyStatelessApplication/MyStatelessPkg/Code/lib")
        include('*')
    }
}
```

En general, para hacerse una idea general acerca de cómo sería el script de compilación para un servicio Java sin estado de Service Fabric, puede remitirse a cualquiera de nuestros ejemplos de introducción. Este es el archivo [build.gradle](https://github.com/Azure-Samples/service-fabric-java-getting-started/blob/master/reliable-services-actor-sample/build.gradle) del ejemplo EchoServer.

## <a name="migrating-service-fabric-actor-service"></a>Migración del servicio Actor de Service Fabric

Para poder crear su aplicación Java Actor de Service Fabric existente con las dependencias de Service Fabric capturadas desde Maven, debe actualizar el archivo ``build.gradle`` dentro del paquete de interfaz y en el paquete de servicio. Si tiene un paquete TestClient, debe actualizarlo también. Así, para el actor ``Myactor``, lo siguiente sería localizar los lugares donde necesita actualizar:

```
./Myactor/build.gradle
./MyactorInterface/build.gradle
./MyactorTestClient/build.gradle
```

#### <a name="updating-build-script-for-the-interface-project"></a>Actualización del script de compilación para el proyecto de interfaz

Antes solían ser como las siguientes:

```gradle
dependencies {
    compile fileTree(dir: '/opt/microsoft/sdk/servicefabric/java/packages/lib', include: ['*.jar'])
}
.
.
```

Ahora, para capturar las dependencias desde Maven, el archivo **actualizado** ``build.gradle`` tendría los elementos correspondientes como se indica a continuación:

```gradle
repositories {
    mavenCentral()
}

configurations {
    azuresf
}

dependencies {
    azuresf ('com.microsoft.servicefabric:sf-actors:1.0.0')
    compile fileTree(dir: 'lib', include: '*.jar')
}

task explodeDeps(type: Copy, dependsOn:configurations.azuresf) { task ->
    configurations.azuresf.filter { it.toString().contains("native") }.each{
        from zipTree(it)
    }
    configurations.azuresf.filter { !it.toString().contains("native") }.each {
        from it
    }
    into "lib"
    include "libj*.so", "*.jar"
}

compileJava.dependsOn(explodeDeps)
.
.
```

#### <a name="updating-build-script-for-the-actor-project"></a>Actualización del script de compilación para el proyecto de actor

Antes solían ser como las siguientes:

```gradle
dependencies {
    compile fileTree(dir: '/opt/microsoft/sdk/servicefabric/java/packages/lib', include: ['*.jar'])
    compile project(':MyactorInterface')
}
.
.
.
jar {
    manifest {
    attributes(
                'Main-Class': 'reliableactor.MyactorHost',
                "Class-Path": configurations.compile.collect { 'lib/' + it.getName() }.join(' '))
      baseName "myactor"
    destinationDir = file('./../myjavaapp/MyactorPkg/Code')
    }
}
.
.
.
task copyDeps<< {
    copy {
        from("/opt/microsoft/sdk/servicefabric/java/packages/lib")
        into("./../myjavaapp/MyactorPkg/Code/lib")
        include('*.jar')
    }
    copy {
        from("/opt/microsoft/sdk/servicefabric/java/packages/lib")
        into("./../myjavaapp/MyactorPkg/Code/lib")
        include('libj*.so')
    }
    copy {
        from("../MyactorInterface/out/lib")
        into("./../myjavaapp/MyactorPkg/Code/lib")
        include('*.jar')
    }
}
```

Ahora, para capturar las dependencias desde Maven, el archivo **actualizado** ``build.gradle`` tendría los elementos correspondientes como se indica a continuación:

```gradle
repositories {
    mavenCentral()
}

configurations {
    azuresf
}

dependencies {
    compile project(':MyactorInterface')
    azuresf ('com.microsoft.servicefabric:sf-actors:1.0.0')
    compile fileTree(dir: 'lib', include: '*.jar')
}

task explodeDeps(type: Copy, dependsOn:configurations.azuresf) { task ->
    configurations.azuresf.filter { it.toString().contains("native") }.each{
        from zipTree(it)
    }
    configurations.azuresf.filter { !it.toString().contains("native") }.each {
        from it
    }
    into "lib"
    include "libj*.so", "*.jar"
}

compileJava.dependsOn(explodeDeps)
.
.
.
jar {
    manifest {
        def mpath = configurations.compile.collect {'lib/'+it.getName()}.join (' ')
        mpath = mpath + ' ' + configurations.azuresf.collect {'lib/'+it.getName()}.join (' ')
        attributes(
                'Main-Class': 'reliableactor.MyactorHost',
                "Class-Path": mpath)
    baseName "myactor"
    destinationDir = file('../myjavaapp/MyactorPkg/Code')}
 }
.
.
.
task copyDeps<< {
      copy {
              from("lib/")
              into("../myjavaapp/MyactorPkg/Code/lib")
              include('*')
      }
      copy {
              from("../MyactorInterface/out/lib")
              into("../myjavaapp/MyactorPkg/Code/lib")
              include('*.jar')
      }
}
```

#### <a name="updating-build-script-for-the-test-client-project"></a>Actualización del script de compilación para el proyecto de cliente de prueba

Los cambios aquí son similares a los que se describen en la sección anterior, es decir, el proyecto de actor. Anteriormente el script Gradle solía ser como el siguiente:

```gradle
dependencies {
    compile fileTree(dir: '/opt/microsoft/sdk/servicefabric/java/packages/lib', include: ['*.jar'])
      compile project(':MyactorInterface')
}
.
.
.
jar
{
    manifest {
    attributes(
        'Main-Class': 'reliableactor.test.MyactorTestClient',
        "Class-Path": configurations.compile.collect { 'lib/' + it.getName() }.join(' '))
    }
    baseName "myactor-test"
  destinationDir = file('out/lib')
}
.
.
.
task copyDeps<< {
        copy {
                from("/opt/microsoft/sdk/servicefabric/java/packages/lib")
                into("./out/lib/lib")
                include('*.jar')
        }
        copy {
                from("/opt/microsoft/sdk/servicefabric/java/packages/lib")
                into("./out/lib/lib")
                include('libj*.so')
        }
        copy {
                from("../MyactorInterface/out/lib")
                into("./out/lib/lib")
                include('*.jar')
        }
}
```

Ahora, para capturar las dependencias desde Maven, el archivo **actualizado** ``build.gradle`` tendría los elementos correspondientes como se indica a continuación:

```gradle
repositories {
    mavenCentral()
}

configurations {
    azuresf
}

dependencies {
    compile project(':MyactorInterface')
    azuresf ('com.microsoft.servicefabric:sf-actors:1.0.0')
    compile fileTree(dir: 'lib', include: '*.jar')
}

task explodeDeps(type: Copy, dependsOn:configurations.azuresf) { task ->
    configurations.azuresf.filter { it.toString().contains("native") }.each{
        from zipTree(it)
    }
    configurations.azuresf.filter { !it.toString().contains("native") }.each {
        from it
    }
    into "lib"
    include "libj*.so", "*.jar"
}

compileJava.dependsOn(explodeDeps)
.
.
.
jar
{
    manifest {
        def mpath = configurations.compile.collect {'lib/'+it.getName()}.join (' ')
        mpath = mpath + ' ' + configurations.azuresf.collect {'lib/'+it.getName()}.join (' ')
    attributes(
                'Main-Class': 'reliableactor.test.MyactorTestClient',
                "Class-Path": mpath)
    baseName "myactor-test"
    destinationDir = file('./out/lib')
        }
}
.
.
.
task copyDeps<< {
        copy {
                from("lib/")
                into("./out/lib/lib")
                include('*')
        }
        copy {
                from("../MyactorInterface/out/lib")
                into("./out/lib/lib")
                include('*.jar')
        }
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Creación e implementación de la primera aplicación de Java para Service Fabric en Linux con Yeoman](service-fabric-create-your-first-linux-application-with-java.md)
* [Creación e implementación de la primera aplicación de Java para Service Fabric en Linux con el complemento de Eclipse para Service Fabric](service-fabric-get-started-eclipse.md)
* [Interacción con los clústeres de Service Fabric mediante la CLI de Service Fabric](service-fabric-cli.md)
