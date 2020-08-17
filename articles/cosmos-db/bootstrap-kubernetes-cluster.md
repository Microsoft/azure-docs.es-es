---
title: Procedimientos de uso de Azure Kubernetes con Azure Cosmos DB
description: Obtenga información sobre cómo arrancar en Azure un clúster de Kubernetes que usa Azure Cosmos DB (versión preliminar)
author: SnehaGunda
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/06/2019
ms.author: sngun
ms.openlocfilehash: 31c3c02f2b32594e5b20450d0bb519f4cdf82807
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86497767"
---
# <a name="how-to-use-azure-kubernetes-with-azure-cosmos-db-preview"></a>Procedimientos de uso de Azure Kubernetes con Azure Cosmos DB (versión preliminar)

La API etcd de Azure Cosmos DB le permite usar Azure Cosmos DB como almacén de back-end de Azure Kubernetes. Azure Cosmos DB implementa el protocolo de transferencia etcd, que permite que los servidores de API del nodo principal usen Azure Cosmos DB del mismo modo que accedería a etcd instalado de forma local. La API etcd de Azure Cosmos DB está actualmente en versión preliminar. Cuando se usa la API etcd de Azure Cosmos como almacén de respaldo para Kubernetes, se obtienen las siguientes ventajas: 

* No es necesario configurar y administrar etcd manualmente.
* Alta disponibilidad de etcd, garantizada por Cosmos (99,99% en una sola región, 99,999% en varias regiones).
* Escalabilidad elástica de etcd.
* Seguro de forma predeterminada y preparado para la empresa.
* Líder del sector, Acuerdos de Nivel de Servicio exhaustivos.

Para obtener información sobre la API etcd de Azure Cosmos DB, consulte el artículo de [introducción](etcd-api-introduction.md). Este artículo muestra cómo usar [Azure Kubernetes Engine](https://github.com/Azure/aks-engine/blob/master/docs/tutorials/quickstart.md) (aks-engine) para arrancar un clúster de Kubernetes en Azure que usa [Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/) en lugar de etcd instalado y configurado localmente. 

## <a name="prerequisites"></a>Prerrequisitos

1. Instale la versión más reciente de la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest). Puede descargar la CLI de Azure específica para su sistema operativo e instalarla.

1. Instale la [versión más reciente](https://github.com/Azure/aks-engine/releases) de Azure Kubernetes Engine. Las instrucciones de instalación para los distintos sistemas operativos están disponibles en la página [Azure Kubernetes Engine](https://github.com/Azure/aks-engine/blob/master/docs/tutorials/quickstart.md#install-aks-engine). Solo son necesarios los pasos de la sección **Instalación de AKS Engine** de la documentación vinculada. Después de la descarga, extraiga el archivo zip.

   Azure Kubernetes Engine (**aks-engine**) genera plantillas de Azure Resource Manager para clústeres de Kubernetes en Azure. La entrada de aks-engine es un archivo de definición de clúster que describe el clúster deseado, incluidos el orquestador, las características y los agentes. La estructura de los archivos de entrada es similar a la API pública de Azure Kubernetes Service.

1. La API etcd de Azure Cosmos DB está actualmente en versión preliminar. Regístrese para usar la versión preliminar en: https://aka.ms/cosmosetcdapi-signup. Después de enviar el formulario, se permitirá la suscripción para usar la API etcd de Azure Cosmos. 

## <a name="deploy-the-cluster-with-azure-cosmos-db"></a>Implementación del clúster con Azure Cosmos DB

1. Abra una ventana del símbolo del sistema e inicie sesión en Azure con el siguiente comando:

   ```azurecli-interactive
   az login 
   ```

1. Si tiene más de una suscripción, cambie a la suscripción que se ha permitido para la API etcd de Azure Cosmos DB. Puede cambiar a la suscripción necesaria con el comando siguiente:

   ```azurecli-interactive
   az account set --subscription "<Name of your subscription>"
   ```
1. A continuación, cree un nuevo grupo de recursos donde se implementarán los recursos requeridos para el clúster de Azure Kubernetes. Asegúrese de crear el grupo de recursos en la región "centralus". No es obligatorio que el grupo de recursos esté en la región "centralus"; sin embargo, la API etcd de Azure Cosmos solo está disponible para su implementación en la región "centralus". Por lo que es mejor tener el clúster de Kubernetes con la instancia de etcd de Cosmos:

   ```azurecli-interactive
   az group create --name <Name> --location "centralus"
   ```

1. A continuación, cree una entidad de servicio para el clúster de Azure Kubernetes para que pueda comunicarse con los recursos que forman parte del mismo grupo de recursos. Puede crear a una entidad de servicio mediante la CLI de Azure, PowerShell o Azure Portal; en este ejemplo, se utiliza la CLI para su creación.

   ```azurecli-interactive
   az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<Your_Azure_subscription_ID>/resourceGroups/<Your_resource_group_name>"
   ```
   Este comando da como resultado los detalles de una entidad de servicio, por ejemplo:
   
   ```cmd
   Retrying role assignment creation: 1/36
   {
     "appId": "8415a4e9-4f83-46ca-a704-107457b2e3ab",
     "displayName": "azure-cli-2019-04-19-19-01-46",
     "name": "http://azure-cli-2019-04-19-19-01-46",
     "password": "102aecd3-5e37-4f3d-8738-2ac348c2e6a7",
     "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db47"
   }
   ```
   
   Tome nota de los campos **appId** y **password**, ya que usará estos parámetros en los pasos siguientes. 

1. Desde el símbolo del sistema, vaya a la carpeta donde se encuentra el ejecutable de Azure Kubernetes Engine. Por ejemplo, en el símbolo del sistema puede ir a la carpeta del modo siguiente:

   ```cmd
   cd "\aks-engine-v0.36.3-windows-amd64\aks-engine-v0.36.3-windows-amd64"
   ```

1. Abra el editor de texto que prefiera y defina una plantilla de Resource Manager que implementa el clúster de Azure Kubernetes con la API etcd de Azure Cosmos DB. Copie la siguiente definición en formato JSON en el editor de texto y guarde el archivo como `apiModel.json`:

   ```json

   {
    "apiVersion": "vlabs",
    "properties": {
        "orchestratorProfile": {
            "orchestratorType": "Kubernetes",
            "kubernetesConfig": {
                "useManagedIdentity": false
            }
        },
        "masterProfile": {
            "count": 1,
            "dnsPrefix": "",
            "vmSize": "Standard_D2_v3",
            "cosmosEtcd": true
        },
        "agentPoolProfiles": [
            {
                "name": "agent",
                "count": 1,
                "vmSize": "Standard_D2_v3",
                "availabilityProfile": "AvailabilitySet"
            }
        ],
        "linuxProfile": {
            "adminUsername": "azureuser",
            "ssh": {
                "publicKeys": [
                    {
                        "keyData": ""
                    }
                ]
            }
        }
    }
   }
   ```

   En el archivo de definición del clúster en formato JSON, el parámetro principal a tener en cuenta es **"cosmosEtcd": true**. Este parámetro se encuentra en las propiedades de "masterProfile" e indica que la implementación va a usar la API etcd de Azure Cosmos en lugar de etcd normal. 

1. Implemente el clúster de Azure Kubernetes que usa Azure Cosmos DB con el siguiente comando:

   ```cmd
   aks-engine deploy \
     --subscription-id <Your_Azure_subscription_ID> \
     --client-id <Service_ principal_appId> \
     --client-secret <Service_ principal_password> \
     --dns-prefix <Region_unique_dns_name > \
     --location centralus \
     --resource-group <Resource_Group_Name> \
     --api-model <Fully_qualified_path_to_the_template_file>  \
     --force-overwrite
   ```

   Azure Kubernetes Engine consume una definición de clúster que describe la forma, el tamaño y la configuración deseadas de Azure Kubernetes. Hay varias características que se pueden habilitar mediante la definición del clúster. En este ejemplo, usará los parámetros siguientes:

   * **subscription-id:** identificador de la suscripción de Azure que tiene habilitada la API etcd de Azure Cosmos DB.
   * **client-id:** appId de la entidad de servicio. `appId` se devolvió como salida en el paso 4.
   * **Client-secret:** contraseña de la entidad de servicio o una contraseña generada aleatoriamente. Este valor se devuelve como salida en el parámetro "password" en el paso 4. 
   * **dnsPrefix:** un nombre DNS único para la región. Este valor formará parte del nombre de host (algunos valores de ejemplo son: myprod1, staging).
   * **location:**  ubicación donde se debe implementar el clúster; actualmente, solo se admite "centralus".

   > [!Note]
   > La API etcd de Azure Cosmos actualmente solo está disponible para su implementación en la región "centralus". 
 
   * **api-model:** ruta de acceso completa al archivo de plantilla.
   * **force-overwrite:** esta opción se usa para sobrescribir automáticamente los archivos existentes en el directorio de salida.
 
   El comando siguiente muestra un ejemplo de implementación:

   ```cmd
   aks-engine deploy \
     --subscription-id 1234fc61-1234-1234-1234-be1234d12c1b \
     --client-id 1234a4e9-4f83-46ca-a704-107457b2e3ab \
     --client-secret 123aecd3-5e37-4f3d-8738-2ac348c2e6a7 \
     --dns-prefix aks-sg-test \
     --location centralus \
     --api-model "C:\Users\demouser\Downloads\apiModel.json"  \
     --force-overwrite
   ```

## <a name="verify-the-deployment"></a>Comprobar la implementación

La implementación de la plantilla tarda varios minutos en completarse. Después de que la implementación se haya completado correctamente, verá la siguiente salida en la línea de comandos:

```cmd
WARN[0006] apimodel: missing masterProfile.dnsPrefix will use "aks-sg-test"
WARN[0006] --resource-group was not specified. Using the DNS prefix from the apimodel as the resource group name: aks-sg-test
INFO[0025] Starting ARM Deployment (aks-sg-test-546247491). This will take some time...
INFO[0587] Finished ARM Deployment (aks-sg-test-546247491). Succeeded
```

El grupo de recursos ahora contiene recursos como: máquina virtual, cuenta de Azure Cosmos (API etcd), red virtual, conjunto de disponibilidad y otros recursos requeridos por el clúster de Kubernetes. 

El nombre de la cuenta de Azure Cosmos coincidirá con el prefijo DNS especificado anexado con k8s. La cuenta de Azure Cosmos se aprovisionará automáticamente con una base de datos llamada **EtcdDB** y un contenedor llamado **EtcdData**. El contenedor almacenará todos los datos relacionados con etcd. El contenedor se aprovisiona con un determinado número de unidades de solicitud y es posible [escalar (aumentar o disminuir) el rendimiento](scaling-throughput.md) según la carga de trabajo. 

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [trabajar con la base de datos, los contenedores y los elementos de Azure Cosmos](databases-containers-items.md)
* Obtenga información sobre la [Optimización del costo del rendimiento aprovisionado](optimize-cost-throughput.md)

