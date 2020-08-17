---
title: Proteger las comunicaciones remotas con Java
description: Aprenda a proteger las comunicaciones remotas de los servicios de Reliable Services escritos en Java que se ejecutan en un clúster de Azure Service Fabric.
author: PavanKunapareddyMSFT
ms.topic: conceptual
ms.date: 06/30/2017
ms.custom: devx-track-java
ms.author: pakunapa
ms.openlocfilehash: 01a64fbcfef9f56abb0e1aa6cf7f5d821dd3763b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325719"
---
# <a name="secure-service-remoting-communications-in-a-java-service"></a>Protección de las comunicaciones remotas de un servicio escrito en Java
> [!div class="op_single_selector"]
> * [C# en Windows](service-fabric-reliable-services-secure-communication.md)
> * [Java en Linux](service-fabric-reliable-services-secure-communication-java.md)
>
>

La seguridad es uno de los aspectos más importantes de la comunicación. El marco de trabajo de la aplicación de Reliable Services proporciona una serie de pilas de comunicación creadas previamente y herramientas que puede utilizar para mejorar la seguridad. En este artículo se describe sobre cómo mejorar la seguridad cuando se utiliza la comunicación remota en un servicio escrito en Java. Vamos a usar un [ejemplo](service-fabric-reliable-services-communication-remoting-java.md) existente que explica cómo configurar la comunicación remota para servicios de Reliable Services escritos en Java. 

Para ayudar a proteger un servicio cuando se usa la comunicación remota con servicios escritos en Java, siga estos pasos:

1. Cree una interfaz, `HelloWorldStateless`, que defina los métodos que estarán disponibles para la llamada a procedimiento remoto en su servicio. El servicio usará `FabricTransportServiceRemotingListener`, que se declara en el paquete `microsoft.serviceFabric.services.remoting.fabricTransport.runtime`. Se trata de una implementación de `CommunicationListener` que ofrece capacidades de comunicación remota.

    ```java
    public interface HelloWorldStateless extends Service {
        CompletableFuture<String> getHelloWorld();
    }

    class HelloWorldStatelessImpl extends StatelessService implements HelloWorldStateless {
        @Override
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
            listeners.add(new ServiceInstanceListener((context) -> {
                return new FabricTransportServiceRemotingListener(context,this);
            }));
        return listeners;
        }

        public CompletableFuture<String> getHelloWorld() {
            return CompletableFuture.completedFuture("Hello World!");
        }
    }
    ```
2. Agregue la configuración de escucha y las credenciales de seguridad.

    Asegúrese de que el certificado que desea utilizar para ayudar a proteger la comunicación de servicio esté instalado en todos los nodos del clúster. Para los servicios que se ejecutan en Linux, el certificado debe estar disponible como un archivo con formato PEM, ya sea un archivo `.pem` que contiene el certificado y la clave privada, o un archivo `.crt` que contiene el certificado y un archivo `.key` que contiene la clave privada. Para más información, consulte [Ubicación y formato de los certificados X.509 en nodos Linux](./service-fabric-configure-certificates-linux.md#location-and-format-of-x509-certificates-on-linux-nodes).
    
    Hay dos formas de proporcionar la configuración de escucha y las credenciales de seguridad:

   1. Proporcionarlas utilizando un [paquete de configuración](service-fabric-application-and-service-manifests.md):

       Agregue una sección llamada `TransportSettings` al archivo settings.xml.

       ```xml
       <!--Section name should always end with "TransportSettings".-->
       <!--Here we are using a prefix "HelloWorldStateless".-->
        <Section Name="HelloWorldStatelessTransportSettings">
            <Parameter Name="MaxMessageSize" Value="10000000" />
            <Parameter Name="SecurityCredentialsType" Value="X509_2" />
            <Parameter Name="CertificatePath" Value="/path/to/cert/BD1C71E248B8C6834C151174DECDBDC02DE1D954.crt" />
            <Parameter Name="CertificateProtectionLevel" Value="EncryptandSign" />
            <Parameter Name="CertificateRemoteThumbprints" Value="BD1C71E248B8C6834C151174DECDBDC02DE1D954" />
        </Section>

       ```

       En este caso, el método `createServiceInstanceListeners` tendrá este aspecto.

       ```java
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
            listeners.add(new ServiceInstanceListener((context) -> {
                return new FabricTransportServiceRemotingListener(context,this, FabricTransportRemotingListenerSettings.loadFrom(HelloWorldStatelessTransportSettings));
            }));
            return listeners;
        }
       ```

        Si agrega una sección `TransportSettings` en el archivo settings.xml sin ningún prefijo, `FabricTransportListenerSettings` cargará toda la configuración de esta sección de forma predeterminada.

        ```xml
        <!--"TransportSettings" section without any prefix.-->
        <Section Name="TransportSettings">
            ...
        </Section>
        ```
        En este caso, el método `CreateServiceInstanceListeners` tendrá este aspecto.

        ```java
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
            listeners.add(new ServiceInstanceListener((context) -> {
                return new FabricTransportServiceRemotingListener(context,this);
            }));
            return listeners;
        }
       ```
3. Al invocar métodos en un servicio protegido con una pila de comunicación remota, en lugar de utilizar la clase `microsoft.serviceFabric.services.remoting.client.ServiceProxyBase` para crear un proxy de servicio, utilice `microsoft.serviceFabric.services.remoting.client.FabricServiceProxyFactory`.

    Si el código de cliente se ejecuta como parte de un servicio, puede cargar la clase `FabricTransportSettings` desde el archivo settings.xml. Cree una sección TransportSettings similar al código de servicio, tal y como se mostró anteriormente. Realice los siguientes cambios en el código de cliente:

    ```java

    FabricServiceProxyFactory serviceProxyFactory = new FabricServiceProxyFactory(c -> {
            return new FabricTransportServiceRemotingClientFactory(FabricTransportRemotingSettings.loadFrom("TransportPrefixTransportSettings"), null, null, null, null);
        }, null)

    HelloWorldStateless client = serviceProxyFactory.createServiceProxy(HelloWorldStateless.class,
        new URI("fabric:/MyApplication/MyHelloWorldService"));

    CompletableFuture<String> message = client.getHelloWorld();

    ```
