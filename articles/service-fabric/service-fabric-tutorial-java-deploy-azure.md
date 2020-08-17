---
title: Implementación de una aplicación Java en un clúster de Service Fabric en Azure
description: En este tutorial, aprenderá a implementar una aplicación de Service Fabric para Java en un clúster de Azure Service Fabric.
author: suhuruli
ms.topic: tutorial
ms.date: 02/26/2018
ms.author: suhuruli
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 50ae4d34cd7b238403ee95bd36a97ac82392f3ba
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320738"
---
# <a name="tutorial-deploy-a-java-application-to-a-service-fabric-cluster-in-azure"></a>Tutorial: Implementación de una aplicación para Java en un clúster de Service Fabric en Azure

Este tutorial forma parte de una serie y muestra cómo implementar una aplicación de Service Fabric en un clúster de Azure.

En la tercera parte de la serie, se aprende a:

> [!div class="checklist"]
> * Creación de un clúster seguro de Linux en Azure
> * Implementación de una aplicación en el clúster

En esta serie de tutoriales, se aprende a:

> [!div class="checklist"]
> * [Compilar una aplicación Java de Reliable Services en Service Fabric](service-fabric-tutorial-create-java-app.md)
> * [Implementar y depurar la aplicación en un clúster local](service-fabric-tutorial-debug-log-local-cluster.md)
> * Implementar la aplicación en un clúster de Azure
> * [Configurar la supervisión y el diagnóstico para la aplicación](service-fabric-tutorial-java-elk.md)
> * [Configure CI/CD](service-fabric-tutorial-java-jenkins.md)

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar este tutorial:

* Si no tiene ninguna suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest)
* Instalación del paquete del SDK de Service Fabric para [Mac](service-fabric-get-started-mac.md) o [Linux](service-fabric-get-started-linux.md)
* [Instalación de Python 3](https://wiki.python.org/moin/BeginnersGuide/Download)

## <a name="create-a-service-fabric-cluster-in-azure"></a>Creación de un clúster de Service Fabric en Azure

Con los pasos siguientes se crean los recursos necesarios para implementar la aplicación en un clúster de Service Fabric. Además, se configuran los recursos necesarios para supervisar el estado de la solución con la pila ELK (Elasticsearch, Logstash, Kibana). En concreto, se utiliza [Event Hubs](https://azure.microsoft.com/services/event-hubs/) como receptor para los registros de Service Fabric. Se configura para enviar registros desde el clúster de Service Fabric hasta la instancia de Logstash.

1. Abra un terminal y descargue el siguiente paquete con los scripts auxiliares y las plantillas necesarias para crear los recursos en Azure.

    ```bash
    git clone https://github.com/Azure-Samples/service-fabric-java-quickstart.git
    ```

2. Inicio de sesión en la cuenta de Azure.

    ```azurecli
    az login
    ```

3. Configure la suscripción de Azure que quiera usar para crear los recursos

    ```azurecli
    az account set --subscription [SUBSCRIPTION-ID]
    ```

4. Desde la carpeta *service-fabric-java-quickstart/AzureCluster*, ejecute el siguiente comando para crear un certificado de clúster en Key Vault. Este certificado se usa para proteger el clúster de Service Fabric. Proporcione la región (debe ser la misma que la del clúster de Service Fabric), el nombre del grupo de recursos del almacén de claves, el nombre del almacén de claves, una contraseña de certificado y el nombre DNS del clúster.

    ```bash
    ./new-service-fabric-cluster-certificate.sh [REGION] [KEY-VAULT-RESOURCE-GROUP] [KEY-VAULT-NAME] [CERTIFICATE-PASSWORD] [CLUSTER-DNS-NAME-FOR-CERTIFICATE]

    Example: ./new-service-fabric-cluster-certificate.sh 'westus' 'testkeyvaultrg' 'testkeyvault' '<password>' 'testservicefabric.westus.cloudapp.azure.com'
    ```

    El comando anterior devuelve la siguiente información, anótela para usarla más adelante.

    ```output
    Source Vault Resource Id: /subscriptions/<subscription_id>/resourceGroups/testkeyvaultrg/providers/Microsoft.KeyVault/vaults/<name>
    Certificate URL: https://<name>.vault.azure.net/secrets/<cluster-dns-name-for-certificate>/<guid>
    Certificate Thumbprint: <THUMBPRINT>
    ```

5. Cree un grupo de recursos para la cuenta de Storage que almacena los registros.

    ```azurecli
    az group create --location [REGION] --name [RESOURCE-GROUP-NAME]

    Example: az group create --location westus --name teststorageaccountrg
    ```

6. Cree una cuenta de Storage que se usará para almacenar los registros que se produzcan.

    ```azurecli
    az storage account create -g [RESOURCE-GROUP-NAME] -l [REGION] --name [STORAGE-ACCOUNT-NAME] --kind Storage

    Example: az storage account create -g teststorageaccountrg -l westus --name teststorageaccount --kind Storage
    ```

7. Acceda a [Azure Portal](https://portal.azure.com) y vaya hasta la pestaña **Firma de acceso compartido** de la cuenta de Storage. Genere el token de la firma de acceso compartido como se indica a continuación.

    ![Generación de la firma de acceso compartido para Storage](./media/service-fabric-tutorial-java-deploy-azure/storagesas.png)

8. Copie la dirección URL de la firma de acceso compartido de la cuenta y establézcala como reserva para usarla al crear el clúster de Service Fabric. Se parece a la siguiente dirección URL:

    ```output
    ?sv=2017-04-17&ss=bfqt&srt=sco&sp=rwdlacup&se=2018-01-31T03:24:04Z&st=2018-01-30T19:24:04Z&spr=https,http&sig=IrkO1bVQCHcaKaTiJ5gilLSC5Wxtghu%2FJAeeY5HR%2BPU%3D
    ```

9. Cree un grupo de recursos que contenga los recursos de Event Hubs. Con Event Hubs se envían mensajes de Service Fabric al servidor que ejecuta los recursos de ELK.

    ```azurecli
    az group create --location [REGION] --name [RESOURCE-GROUP-NAME]

    Example: az group create --location westus --name testeventhubsrg
    ```

10. Cree un recurso de Event Hubs mediante el siguiente comando. Siga las instrucciones para escribir detalles en namespaceName, eventHubName, consumerGroupName, sendAuthorizationRule y receiveAuthorizationRule.

    ```azurecli
    az group deployment create -g [RESOURCE-GROUP-NAME] --template-file eventhubsdeploy.json

    Example:
    az group deployment create -g testeventhubsrg --template-file eventhubsdeploy.json
    Please provide string value for 'namespaceName' (? for help): testeventhubnamespace
    Please provide string value for 'eventHubName' (? for help): testeventhub
    Please provide string value for 'consumerGroupName' (? for help): testeventhubconsumergroup
    Please provide string value for 'sendAuthorizationRuleName' (? for help): sender
    Please provide string value for 'receiveAuthorizationRuleName' (? for help): receiver
    ```

    Copie el contenido del campo de **salida** en la salida JSON del comando anterior. La información del remitente se utiliza al crear el clúster de Service Fabric. El nombre del receptor y la clave deben guardarse para su uso en el tutorial siguiente, una vez configurado el servicio Logstash para recibir mensajes de Event Hubs. El siguiente blob es una salida JSON de ejemplo:

    ```json
    "outputs": {
        "receiver Key": {
            "type": "String",
            "value": "[KEY]"
        },
        "receiver Name": {
            "type": "String",
            "value": "receiver"
        },
        "sender Key": {
            "type": "String",
            "value": "[KEY]"
        },
        "sender Name": {
            "type": "String",
            "value": "sender"
        }
    }
    ```

11. Ejecute el script *eventhubssastoken.py* para generar la dirección URL de firma de acceso compartido para el recurso de Event Hubs que creó. Esta dirección URL de firma de acceso compartido la usa el clúster de Service Fabric para enviar registros a Event Hubs. Como resultado, con la directiva del **remitente** se genera la dirección URL. El script devuelve la dirección URL de la firma de acceso compartido del recurso de Event Hubs que se utiliza en el paso siguiente:

    ```python
    python3 eventhubssastoken.py 'testeventhubs' 'testeventhubs' 'sender' '[PRIMARY-KEY]'
    ```

    Copie el valor del campo **sr** del JSON devuelto. El valor del campo **sr** es el token de firma de acceso compartido de Event Hubs. La dirección URL siguiente es un ejemplo de campo **sr**:

    ```output
    https%3A%2F%testeventhub.servicebus.windows.net%testeventhub&sig=7AlFYnbvEm%2Bat8ALi54JqHU4i6imoFxkjKHS0zI8z8I%3D&se=1517354876&skn=sender
    ```

    La dirección URL de la firma de acceso compartido de Event Hubs sigue la estructura: `https://<namespacename>.servicebus.windows.net/<eventhubsname>?sr=<sastoken>`. Por ejemplo: `https://testeventhubnamespace.servicebus.windows.net/testeventhub?sr=https%3A%2F%testeventhub.servicebus.windows.net%testeventhub&sig=7AlFYnbvEm%2Bat8ALi54JqHU4i6imoFxkjKHS0zI8z8I%3D&se=1517354876&skn=sender`

12. Abra el archivo *sfdeploy.parameters.json* y reemplace el contenido siguiente de los pasos anteriores. [SAS-URL-STORAGE-ACCOUNT] se anotó en el paso 8. [SAS-URL-EVENT-HUBS] se anotó en el paso 11.

    ```json
    "applicationDiagnosticsStorageAccountName": {
        "value": "teststorageaccount"
    },
    "applicationDiagnosticsStorageAccountSasToken": {
        "value": "[SAS-URL-STORAGE-ACCOUNT]"
    },
    "loggingEventHubSAS": {
        "value": "[SAS-URL-EVENT-HUBS]"
    }
    ```

13. Se abre **sfdeploy.parameters.json**. Cambie los parámetros siguientes y, a continuación, guarde el archivo.
    - **clusterName**. Use solo letras minúsculas y números.
    - **adminUserName** (para un valor distinto de cero)
    - **adminPassword** (para un valor distinto de cero)

14. Ejecute el comando siguiente para crear el clúster de Service Fabric.

    ```azurecli
    az sf cluster create --location 'westus' --resource-group 'testlinux' --template-file sfdeploy.json --parameter-file sfdeploy.parameters.json --secret-identifier <certificate_url_from_step4>
    ```

## <a name="deploy-your-application-to-the-cluster"></a>Implemente la aplicación en el clúster.

1. Antes de implementar la aplicación, debe agregar el siguiente fragmento de código al archivo *Voting/VotingApplication/ApplicationManifest.xml*. El campo **X509FindValue** es la huella digital devuelta del paso 4 de la sección **Creación de un clúster de Service Fabric en Azure**. Este fragmento de código está anidado en el campo **ApplicationManifest** (raíz).

    ```xml
    <Certificates>
          <SecretsCertificate X509FindType="FindByThumbprint" X509FindValue="[CERTIFICATE-THUMBPRINT]" />
    </Certificates>
    ```

2. Para implementar la aplicación en este clúster, debe utilizar SFCTL para establecer una conexión con el clúster. SFCTL requiere un archivo PEM con la clave pública y la privada para conectarse al clúster. Ejecute el siguiente comando para generar un archivo PEM con las claves pública y privada. 

    ```bash
    openssl pkcs12 -in <clustername>.<region>.cloudapp.azure.com.pfx -out sfctlconnection.pem -nodes -passin pass:<password>
    ```

3. Ejecute el siguiente comando para conectarse al clúster.

    ```bash
    sfctl cluster select --endpoint https://<clustername>.<region>.cloudapp.azure.com:19080 --pem sfctlconnection.pem --no-verify
    ```

4. Para implementar la aplicación, vaya a la carpeta *Voting/Scripts* y ejecute el script **Install.sh**.

    ```bash
    ./install.sh
    ```

5. Para acceder a Service Fabric Explorer, abra su explorador favorito y escriba `https://testlinuxcluster.westus.cloudapp.azure.com:19080`. Seleccione del almacén de certificados el que desee usar para conectarse a este punto de conexión. En las máquinas Linux, los certificados generados que generó el script *new-service-fabric-cluster-certificate.sh* deben importarse en Chrome para que aparezca Service Fabric Explorer. En las máquinas Mac es necesario instalar el archivo PFX en la cadena de claves. Observe que la aplicación se ha instalado en el clúster.

    ![SFX con Java en Azure](./media/service-fabric-tutorial-java-deploy-azure/sfxjavaonazure.png)

6. Para acceder a la aplicación, escriba `https://testlinuxcluster.westus.cloudapp.azure.com:8080`

    ![Aplicación de Voting con Java en Azure](./media/service-fabric-tutorial-java-deploy-azure/votingappjavaazure.png)

7. Para desinstalar la aplicación del clúster, ejecute el script *uninstall.sh* de la carpeta **Scripts**

    ```bash
    ./uninstall.sh
    ```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Creación de un clúster seguro de Linux en Azure
> * Creación de los recursos necesarios para la supervisión con ELK

Avance hasta el siguiente tutorial:
> [!div class="nextstepaction"]
> [Configuración de Supervisión y diagnóstico](service-fabric-tutorial-java-elk.md)
