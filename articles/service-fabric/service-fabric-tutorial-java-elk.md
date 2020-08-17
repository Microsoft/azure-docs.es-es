---
title: Supervisión de las aplicaciones en Service Fabric mediante ELK en Azure
description: En este tutorial, aprenderá a configurar ELK y a supervisar las aplicaciones de Service Fabric.
author: suhuruli
ms.topic: tutorial
ms.date: 02/26/2018
ms.author: suhuruli
ms.custom: mvc, devx-track-java
ms.openlocfilehash: b27711158c10b0f7fa78875aec7f9c63c166258c
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327181"
---
# <a name="tutorial-monitor-your-service-fabric-applications-using-elk"></a>Tutorial: Supervisión de las aplicaciones de Service Fabric mediante ELK

Este tutorial es la cuarta parte de una serie. En él se muestra cómo usar ELK (Elasticsearch, Logstash y Kibana) para supervisar las aplicaciones de Service Fabric que se ejecutan en Azure.

En la parte número cuatro de la serie, se aprende a:
> [!div class="checklist"]
> * Configurar el servidor ELK en Azure
> * Configurar Logstash para recibir registros de Event Hubs
> * Visualizar registros de aplicaciones y de la plataforma en Kibana

En esta serie de tutoriales, se aprende a:
> [!div class="checklist"]
> * [Compilar una aplicación Java de Reliable Services en Service Fabric](service-fabric-tutorial-create-java-app.md)
> * [Implementar y depurar la aplicación en un clúster local](service-fabric-tutorial-debug-log-local-cluster.md)
> * [Implementar la aplicación en un clúster de Azure](service-fabric-tutorial-java-deploy-azure.md)
> * Configurar la supervisión y el diagnóstico para la aplicación
> * [Configure CI/CD](service-fabric-tutorial-java-jenkins.md)

## <a name="prerequisites"></a>Prerequisites

Antes de empezar este tutorial:

* Si no tiene ninguna suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Configure la aplicación para emitir registros a la ubicación especificada en la [parte dos](service-fabric-tutorial-debug-log-local-cluster.md).
* Realice la [tercera parte](service-fabric-tutorial-java-deploy-azure.md) y tenga configurado un clúster de Service Fabric en ejecución para enviar registros a Event Hubs.
* La directiva de Event Hubs que tiene el permiso de escucha y la clave principal asociada de la serie tres.

## <a name="download-the-voting-sample-application"></a>Descarga de la aplicación de ejemplo de votación

Si no compiló la aplicación de ejemplo de votación en la [primera parte de esta serie de tutoriales](service-fabric-tutorial-create-java-app.md), puede descargarla. En una ventana Comandos, ejecute el comando siguiente para clonar el repositorio de la aplicación de ejemplo en la máquina local.

```bash
git clone https://github.com/Azure-Samples/service-fabric-java-quickstart
```

## <a name="create-an-elk-server-in-azure"></a>Creación de un servidor ELK en Azure

Para los fines de este tutorial, puede usar un entorno de ELK preconfigurado y, si ya tiene uno, pasar a la sección **Setup Logstash** (Configurar Logstash). Sin embargo, si no tiene uno, en los siguientes pasos se crea en Azure.

1. Cree un ELK certificado por [Bitnami](https://ms.portal.azure.com/#create/bitnami.elk4-6) en Azure. Para los fines del tutorial, no hay ninguna especificación en especial que se deba seguir para crear este servidor.

2. Vaya al recurso en Azure Portal y, en la sección **Support + Troubleshooting** (Soporte técnico y solución de problemas), haga clic en la pestaña **Diagnósticos de arranque**. A continuación, haga clic en la pestaña **Registro serie**.

    ![Boot Diagnostics](./media/service-fabric-tutorial-java-elk/bootdiagnostics.png)
3. Busque en los registros la contraseña requerida para acceder a la instancia de Kibana. Se parece al siguiente fragmento de código:

    ```bash
    [   25.932766] bitnami[1496]: #########################################################################
    [   25.948656] bitnami[1496]: #                                                                       #
    [   25.962448] bitnami[1496]: #        Setting Bitnami application password to '[PASSWORD]'           #
    [   25.978137] bitnami[1496]: #        (the default application username is 'user')                   #
    [   26.004770] bitnami[1496]: #                                                                       #
    [   26.029413] bitnami[1496]: #########################################################################
    ```

4. Presione el botón de conexión en la página Información general del servidor en Azure Portal para obtener los detalles de inicio de sesión.

    ![Conexión de la máquina virtual](./media/service-fabric-tutorial-java-elk/vmconnection.png)

5. Conéctese mediante SSH al servidor que hospeda la imagen de ELK mediante el siguiente comando:

    ```bash
    ssh [USERNAME]@[CONNECTION-IP-OF-SERVER]

    Example: ssh testaccount@104.40.63.157
    ```

## <a name="set-up-elk"></a>Configuración de ELK

1. El primer paso consiste en cargar el entorno de ELK.

    ```bash
    sudo /opt/bitnami/use_elk
    ```

2. Si va a usar un entorno existente, tiene que ejecutar el siguiente comando para detener el servicio Logstash.

    ```bash
    sudo /opt/bitnami/ctlscript.sh stop logstash
    ```

3. Ejecute el siguiente comando para instalar el complemento Logstash para Event Hubs.

    ```bash
    logstash-plugin install logstash-input-azureeventhub
    ```

4. Cree o modifique el archivo de configuración de Logstash existente con el siguiente contenido: si va a crear el archivo, lo debe hacer en ```/opt/bitnami/logstash/conf/access-log.conf``` si usa la imagen de Bitnami de ELK en Azure.

    ```json
    input
    {
        azureeventhub
        {
            key => "[RECEIVER-POLICY-KEY-FOR-EVENT-HUB]"
            username => "[RECEIVER-POLICY-NAME]"
            namespace => "[EVENTHUB-NAMESPACENAME]"
            eventhub => "[EVENTHUB-NAME]"
            partitions => 4
        }
    }

    output {
         elasticsearch {
             hosts => [ "127.0.0.1:9200" ]
         }
     }
    ```

5. Para actualizar la configuración, ejecute el siguiente comando:

    ```bash
    /opt/bitnami/logstash/bin/logstash -f /opt/bitnami/logstash/conf/ --config.test_and_exit
    ```

6. Inicio del servicio Logstash

    ```bash
    sudo /opt/bitnami/ctlscript.sh start logstash
    ```

7. Compruebe su instancia de Elasticsearch para asegurarse de que recibe datos

    ```bash
    curl 'localhost:9200/_cat/indices?v'
    ```

8. Acceda al panel de Kibana en **http:\//SERVER-IP** y escriba el nombre de usuario y la contraseña de Kibana. Si usó la imagen de ELK en Azure, el nombre de usuario predeterminado es "user" y la contraseña es la que se obtiene en la pestaña **Diagnósticos de arranque**.

    ![Kibana](./media/service-fabric-tutorial-java-elk/kibana.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Poner en funcionamiento un servidor de ELK en Azure
> * Configurar el servidor para recibir información de diagnóstico del clúster de Service Fabric

Avance hasta el siguiente tutorial:
> [!div class="nextstepaction"]
> [Configuración de CI/CD con Jenkins](service-fabric-tutorial-java-jenkins.md)
