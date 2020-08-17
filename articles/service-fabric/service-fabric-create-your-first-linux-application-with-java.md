---
title: Creación de una aplicación de Java de Reliable Actors de Azure Service Fabric en Linux
description: Aprenda a crear e implementar una aplicación Java de Reliable Actors de Service Fabric en cinco minutos.
ms.topic: conceptual
ms.date: 06/18/2018
ms.custom: devx-track-java
ms.openlocfilehash: 28ddc4f0e47d853df6b026cf2495d710bebfa980
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87368938"
---
# <a name="create-your-first-java-service-fabric-reliable-actors-application-on-linux"></a>Creación de su primera aplicación Java de Reliable Actors de Service Fabric en Linux
> [!div class="op_single_selector"]
> * [Java - Linux](service-fabric-create-your-first-linux-application-with-java.md)
> * [C# - Linux](service-fabric-create-your-first-linux-application-with-csharp.md)
>
>

Este inicio rápido le ayuda a crear su primera aplicación Java de Azure Service Fabric en un entorno de desarrollo de Linux en tan solo unos minutos.  Cuando haya terminado, tendrá una sencilla aplicación Java con un único servicio en el clúster de desarrollo local.  

## <a name="prerequisites"></a>Prerrequisitos
Antes de comenzar, instale el SDK de Service Fabric, la CLI de Service Fabric y Yeoman; asimismo, configure el entorno de desarrollo de Java y un clúster de desarrollo en su [entorno de desarrollo de Linux](service-fabric-get-started-linux.md). Si usa Mac OS X, puede [configurar un entorno de desarrollo en Mac mediante Docker](service-fabric-get-started-mac.md).

Instale también la [CLI de Service Fabric](service-fabric-cli.md).

### <a name="install-and-set-up-the-generators-for-java"></a>Instalación y configuración de los generadores para Java
Service Fabric proporciona herramientas de scaffolding que le ayudarán a crear una aplicación Java de Service Fabric desde el terminal mediante el generador de plantillas Yeoman.  Si Yeoman no está instalado aún, consulte [Introducción a Service Fabric con Linux](service-fabric-get-started-linux.md#set-up-yeoman-generators-for-containers-and-guest-executables) para obtener instrucciones sobre cómo configurar Yeoman. Ejecute el siguiente comando para instalar el generador de plantillas Yeoman de Service Fabric para Java.

  ```bash
  npm install -g generator-azuresfjava
  ```

## <a name="basic-concepts"></a>Conceptos básicos
Para empezar a trabajar con Reliable Actors, solo es necesario comprender cuatro conceptos básicos:

* **Servicio de actor**. Reliable Actors se incluye en Reliable Services, que puede implementarse en la infraestructura de Service Fabric. Las instancias de actor se activan en una instancia de servicio con nombre.
* **Registro de actor**. Como con Reliable Services, un servicio de Reliable Actors debe estar registrado en el runtime de Service Fabric. Además, el tipo de actor debe registrarse con el tiempo de ejecución de Actor.
* **Interfaz de actor**. La interfaz de actor se usa para definir una interfaz pública fuertemente tipada de un actor. En la terminología del modelo de Reliable Actors, la interfaz de actor define los tipos de mensajes que el actor puede entender y procesar. Otros actores y aplicaciones de cliente usan la interfaz de actor para "enviar" (asincrónicamente) mensajes al actor. Reliable Actors pueden implementar varias interfaces.
* **Clase ActorProxy**. La clase ActorProxy la usan las aplicaciones cliente para invocar los métodos expuestos a través de una interfaz de actor. La clase ActorProxy ofrece dos funciones importantes:
  
  * Resolución de nombres: Es capaz de ubicar el actor en el clúster (encontrar el nodo en que se hospeda el clúster).
  * Control de errores: Puede volver a intentar invocar a los métodos y volver a resolver la ubicación del actor, por ejemplo, tras un error que requiera que el actor se reubique en otro nodo del clúster.

Conviene destacar las siguientes reglas que pertenecen a las interfaces de actor:

* No se pueden sobrecargar los métodos de interfaz de actor.
* Los métodos de interfaz de actor no deben tener parámetros out, ref u opcionales.
* No se admiten las interfaces genéricas.

## <a name="create-the-application"></a>Creación de la aplicación
Una aplicación de Service Fabric contiene uno o varios servicios, cada uno de ellos con un rol específico a la hora de proporcionar la funcionalidad de la aplicación. El generador que instaló en la última sección facilita la creación de su primer servicio y agregar más servicios posteriormente.  También puede crear, compilar e implementar aplicaciones Java de Service Fabric mediante un complemento para Eclipse. Consulte [Creación e implementación de la primera aplicación Java mediante Eclipse](service-fabric-get-started-eclipse.md). En este inicio rápido, use Yeoman para crear una aplicación con un único servicio que almacena y obtiene un valor de contador.

1. En un terminal, escriba ``yo azuresfjava``.
2. Asigne un nombre a la aplicación.
3. Elija el tipo del primer servicio y asígnele un nombre. En este tutorial, elija un servicio de Reliable Actor. Para más información sobre los otros tipos de servicios, consulte [Información general del modelo de programación de Service Fabric](service-fabric-choose-framework.md).
   ![Generador Yeoman de Service Fabric para Java][sf-yeoman]

Si asigna el nombre "HelloWorldActorApplication" a la aplicación y "HelloWorldActor" al actor, se crea el siguiente scaffolding:

```bash
HelloWorldActorApplication/
├── build.gradle
├── HelloWorldActor
│   ├── build.gradle
│   ├── settings.gradle
│   └── src
│       └── reliableactor
│           ├── HelloWorldActorHost.java
│           └── HelloWorldActorImpl.java
├── HelloWorldActorApplication
│   ├── ApplicationManifest.xml
│   └── HelloWorldActorPkg
│       ├── Code
│       │   ├── entryPoint.sh
│       │   └── _readme.txt
│       ├── Config
│       │   ├── _readme.txt
│       │   └── Settings.xml
│       ├── Data
│       │   └── _readme.txt
│       └── ServiceManifest.xml
├── HelloWorldActorInterface
│   ├── build.gradle
│   └── src
│       └── reliableactor
│           └── HelloWorldActor.java
├── HelloWorldActorTestClient
│   ├── build.gradle
│   ├── settings.gradle
│   ├── src
│   │   └── reliableactor
│   │       └── test
│   │           └── HelloWorldActorTestClient.java
│   └── testclient.sh
├── install.sh
├── settings.gradle
└── uninstall.sh
```
## <a name="reliable-actors-basic-building-blocks"></a>Bloques de creación básicos de Reliable Actors
Los conceptos básicos descritos anteriormente se traducen en los bloques de creación básicos de un servicio de Reliable Actors.

### <a name="actor-interface"></a>Interfaz de actor
Contiene la definición de la interfaz del actor. Esta interfaz define el contrato de actor que comparten la implementación del actor y los clientes que llaman al actor. Por tanto, suele tener sentido definirlas en un lugar independiente de la implementación del actor y que se pueda compartir entre varios servicios o aplicaciones cliente.

`HelloWorldActorInterface/src/reliableactor/HelloWorldActor.java`:

```java
public interface HelloWorldActor extends Actor {
    @Readonly   
    CompletableFuture<Integer> getCountAsync();

    CompletableFuture<?> setCountAsync(int count);
}
```

### <a name="actor-service"></a>Servicio de actor
Contiene la implementación del actor y el código de registro del actor. La clase actor implementa la interfaz de actor. Aquí es donde el actor realiza su trabajo.

`HelloWorldActor/src/reliableactor/HelloWorldActorImpl`:

```java
@ActorServiceAttribute(name = "HelloWorldActorService")
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
public class HelloWorldActorImpl extends FabricActor implements HelloWorldActor {
    private Logger logger = Logger.getLogger(this.getClass().getName());

    public HelloWorldActorImpl(FabricActorService actorService, ActorId actorId){
        super(actorService, actorId);
    }

    @Override
    protected CompletableFuture<?> onActivateAsync() {
        logger.log(Level.INFO, "onActivateAsync");

        return this.stateManager().tryAddStateAsync("count", 0);
    }

    @Override
    public CompletableFuture<Integer> getCountAsync() {
        logger.log(Level.INFO, "Getting current count value");
        return this.stateManager().getStateAsync("count");
    }

    @Override
    public CompletableFuture<?> setCountAsync(int count) {
        logger.log(Level.INFO, "Setting current count value {0}", count);
        return this.stateManager().addOrUpdateStateAsync("count", count, (key, value) -> count > value ? count : value);
    }
}
```

### <a name="actor-registration"></a>Registro del actor
El servicio de actor debe registrarse con un tipo de servicio en el runtime de Service Fabric. Para que el servicio de actor ejecute las instancias de actor, el tipo de actor debe estar registrado también con el servicio de actor. El método de registro `ActorRuntime` realiza este trabajo para los actores.

`HelloWorldActor/src/reliableactor/HelloWorldActorHost`:

```java
public class HelloWorldActorHost {

private static final Logger logger = Logger.getLogger(HelloWorldActorHost.class.getName());
    
public static void main(String[] args) throws Exception {
        
        try {

            ActorRuntime.registerActorAsync(HelloWorldActorImpl.class, (context, actorType) -> new FabricActorService(context, actorType, (a,b)-> new HelloWorldActorImpl(a,b)), Duration.ofSeconds(10));
            Thread.sleep(Long.MAX_VALUE);
        } catch (Exception e) {
            logger.log(Level.SEVERE, "Exception occurred", e);
            throw e;
        }
    }
}
```

## <a name="build-the-application"></a>Compilar la aplicación
Las plantillas de Yeoman de Service Fabric incluyen un script de compilación para [Gradle](https://gradle.org/), que se puede usar para compilar la aplicación desde el terminal.
Las dependencias de Java de Service Fabric se recuperan de Maven. Para crear y trabajar con las aplicaciones de Java de Service Fabric debe asegurarse de que tiene JDK y Gradle instalados. Si no está instalado aún, consulte [Introducción a Service Fabric con Linux](service-fabric-get-started-linux.md#set-up-java-development) para obtener instrucciones sobre cómo instalar JDK y Gradle.

Para compilar y empaquetar la aplicación, ejecute el siguiente código:

  ```bash
  cd HelloWorldActorApplication
  gradle
  ```

## <a name="deploy-the-application"></a>Implementación de la aplicación
Una vez compilada la aplicación, se puede implementar en el clúster local.

1. Conéctese al clúster local de Service Fabric (el clúster debe estar [configurado y en ejecución](service-fabric-get-started-linux.md#set-up-a-local-cluster)).

    ```bash
    sfctl cluster select --endpoint http://localhost:19080
    ```

2. Ejecute el script de instalación proporcionado en la plantilla para copiar el paquete de aplicación en el almacén de imágenes del clúster, registrar el tipo de aplicación y crear una instancia de la aplicación.

    ```bash
    ./install.sh
    ```

La aplicación compilada se implementa de la misma forma que cualquier otra aplicación de Service Fabric. Consulte la documentación sobre la [Administración de una aplicación de Service Fabric con la CLI de Service Fabric](service-fabric-application-lifecycle-sfctl.md) para obtener instrucciones detalladas.

Los parámetros de estos comandos se pueden encontrar en los manifiestos generados dentro del paquete de aplicación.

Una vez que se haya implementado la aplicación, abra un explorador y vaya a [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) en `http://localhost:19080/Explorer`.
Luego, expanda el nodo **Applications** y observe que ahora hay una entrada para su tipo de aplicación y otra para la primera instancia de dicho tipo.

> [!IMPORTANT]
> Para implementar la aplicación en un clúster de Linux seguro en Azure, debe configurar un certificado para validar la aplicación con el sistema de tiempo de ejecución de Service Fabric. Esto permite a los servicios de Reliable Actors comunicarse con las API en tiempo de ejecución subyacentes de Service Fabric. Para obtener más información, consulte [Configure a Reliable Services app to run on Linux clusters](./service-fabric-configure-certificates-linux.md#configure-a-reliable-services-app-to-run-on-linux-clusters) (Configurar una aplicación de Reliable Services para ejecutarla en clústeres Linux).  
>

## <a name="start-the-test-client-and-perform-a-failover"></a>Inicio del cliente de prueba y ejecución de una conmutación por error
Los actores no hacen nada por sí solos, necesitan que otro servicio o cliente les envíe mensajes. La plantilla de actor incluye un sencillo script de prueba que puede usar para interactuar con el servicio de actor.

> [!Note]
> El cliente de prueba usa la clase ActorProxy para comunicarse con los actores, que se deben ejecutar dentro del mismo clúster que el servicio de actor o compartir el mismo espacio de direcciones IP.  Puede ejecutar el cliente de prueba en el mismo equipo que el clúster de desarrollo local.  Sin embargo, para comunicarse con los actores en un clúster remoto, debe implementar una puerta de enlace en el clúster que administre la comunicación externa con los actores.

1. Ejecute el script con la utilidad de inspección para ver la salida del servicio de actor.  El script de prueba llamad al método `setCountAsync()` en el servicio de actor para aumentar el contador, llama al método `getCountAsync()` en el servicio de actor para obtener el nuevo valor de contador y muestra ese valor en la consola.

   En el caso de MAC OS X, debe copiar la carpeta HelloWorldTestClient en alguna ubicación dentro del contenedor; para ello, ejecute los siguientes comandos adicionales.    
    
    ```bash
     docker cp HelloWorldTestClient [first-four-digits-of-container-ID]:/home
     docker exec -it [first-four-digits-of-container-ID] /bin/bash
     cd /home
     ```

    ```bash
    cd HelloWorldActorTestClient
    watch -n 1 ./testclient.sh
    ```

2. En Service Fabric Explorer, busque el nodo que hospeda la réplica principal del servicio de actor. En la captura de pantalla siguiente, es el nodo 3. La réplica de servicio principal gestiona las operaciones de lectura y escritura.  Los cambios en el estado del servicio se replican entonces en las réplicas secundarias, que se ejecutan en los nodos 0 y 1 en la siguiente captura de pantalla.

    ![Búsqueda de la réplica principal en Service Fabric Explorer][sfx-primary]

3. En **Nodos**, haga clic en el nodo que encontró en el paso anterior y seleccione **Desactivar (reiniciar)** en el menú Acciones. Esta acción reinicia el nodo que ejecuta la réplica principal del servicio y fuerza una conmutación por error a una de las réplicas secundarias que se ejecutan en otro nodo.  Esa réplica secundaria se promueve a principal, se crea otra réplica secundaria en un nodo diferente y la réplica principal comienza a aceptar operaciones de lectura/escritura. Cuando se reinicie el nodo, preste atención a la salida del cliente de prueba y observe que el contador sigue incrementándose a pesar de la conmutación por error.

## <a name="remove-the-application"></a>Eliminación de la aplicación
Use el script de desinstalación proporcionado en la plantilla para eliminar la instancia de la aplicación, anular el registro del paquete de aplicación y quitar dicho paquete del almacén de imágenes del clúster.

```bash
./uninstall.sh
```

En Service Fabric Explorer, verá que la aplicación y el tipo de aplicación ya no aparecen en el nodo **Aplicaciones**.

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

Compatibilidad con Reliable Services de Service Fabric para la aplicación.

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

## <a name="next-steps"></a>Pasos siguientes

* [Creación de la primera aplicación Java de Service Fabric en Linux](service-fabric-get-started-eclipse.md)
* [Más información acerca de Reliable Actors](service-fabric-reliable-actors-introduction.md)
* [Interacción con los clústeres de Service Fabric mediante la CLI de Service Fabric](service-fabric-cli.md)
* Más información sobre las [opciones de soporte técnico de Service Fabric](service-fabric-support.md)
* [Introducción a la CLI de Service Fabric](service-fabric-cli.md)

<!-- Images -->
[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-yeoman.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-java/sfx-primary.png
[sf-eclipse-templates]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-eclipse-templates.png
