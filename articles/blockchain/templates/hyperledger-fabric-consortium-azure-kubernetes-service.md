---
title: Consorcio Hyperledger Fabric en Azure Kubernetes Service (AKS)
description: Implementación y configuración de una red del consorcio de Hyperledger Fabric en Azure Kubernetes Service
ms.date: 08/06/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.openlocfilehash: d23a0120aafb4dc3e6952b40959a20f9a3456614
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226884"
---
# <a name="hyperledger-fabric-consortium-on-azure-kubernetes-service-aks"></a>Consorcio Hyperledger Fabric en Azure Kubernetes Service (AKS)

Puede usar la plantilla de Hyperledger Fabric en Azure Kubernetes Service (AKS) para implementar y configurar una red del consorcio de Hyperledger Fabric en Azure.

Después de leer este artículo, habrá aprendido lo siguiente:

- Conocimiento práctico de Hyperledger Fabric y de los componentes que forman los bloques de creación de una red de la cadena de bloques de Hyperledger Fabric.
- Cómo implementar y configurar una red del consorcio de Hyperledger Fabric en Azure Kubernetes Service para sus escenarios de producción.

[!INCLUDE [Preview note](./includes/preview.md)]

## <a name="choose-an-azure-blockchain-solution"></a>Elegir una solución de Azure Blockchain

Antes de optar por usar la plantilla de solución, compare su escenario con los casos de uso comunes de las opciones de Azure Blockchain disponibles:

Opción | Modelo de servicio | Caso de uso común
-------|---------------|-----------------
Plantillas de solución | IaaS | Las plantillas de solución son plantillas de Azure Resource Manager que puede usar para aprovisionar una topología de red de cadena de bloques totalmente configurada. Las plantillas implementan y configuran servicios de proceso, redes y almacenamiento de Microsoft Azure para un tipo de red de cadena de bloques determinado. Las plantillas de solución se proporcionan sin un contrato de nivel de servicio. Use la [página de preguntas y respuestas de Microsoft](/answers/topics/azure-blockchain-workbench.html) para obtener soporte técnico.
[Azure Blockchain Service](../service/overview.md) | PaaS | Azure Blockchain Service Preview simplifica la formación, administración y regulación de las redes de cadena de bloques del consorcio. Use Azure Blockchain Service para soluciones que requieran PaaS, la administración del consorcio o la privacidad de contratos y transacciones.
[Azure Blockchain Workbench](../workbench/overview.md) | IaaS y PaaS | La versión preliminar de Azure Blockchain Workbench es una colección de servicios y funcionalidades de Azure que le ayudan a crear e implementar aplicaciones de cadena de bloques para compartir datos y procesos empresariales con otras organizaciones. Use Azure Blockchain Workbench para crear un prototipo de una solución de cadena de bloques o una prueba de concepto de una aplicación de cadena de bloques. Azure Blockchain Workbench se proporciona sin un contrato de nivel de servicio. Use la [página de preguntas y respuestas de Microsoft](/answers/topics/azure-blockchain-workbench.html) para obtener soporte técnico.

## <a name="hyperledger-fabric-consortium-architecture"></a>Arquitectura del consorcio de Hyperledger Fabric

Para compilar la red de Hyperledger Fabric en Azure, debe implementar el servicio de ordenación y la organización con nodos del mismo nivel. Con la plantilla de la solución de Hyperledger Fabric en Azure Kubernetes Service, puede crear nodos de ordenación o nodos del mismo nivel. Debe implementar la plantilla para cada nodo que quiera crear.

Los componentes fundamentales que se crean como parte de la implementación de la plantilla son:

- **Nodos solicitantes**: un nodo responsable del orden de las transacciones en el libro de contabilidad. Junto con otros nodos, los nodos ordenados forman el servicio de ordenación de la red de Hyperledger Fabric.

- **Nodos del mismo nivel**: un nodo que hospeda principalmente libros de contabilidad y contratos inteligentes, que son elementos fundamentales de la red.

- **Fabric CA**: la entidad de certificación (CA) para Hyperledger Fabric. Fabric CA permite inicializar e iniciar un proceso de servidor que hospeda la entidad de certificación. Permite administrar identidades y certificados. Cada clúster de AKS implementado como parte de la plantilla tendrá un pod de Fabric CA de forma predeterminada.

- **CouchDB o LevelDB**: bases de datos de estado global para los nodos del mismo nivel. LevelDB es la base de datos de estado predeterminada incrustada en el nodo del mismo nivel. Almacena los datos del código de cadena como pares clave/valor simples y admite solo consultas de clave, intervalo de claves y clave compuesta. CouchDB es una base de datos de estado alternativa opcional que admite consultas enriquecidas cuando los valores de los datos del código de cadena se modelan como JSON.

La plantilla de la implementación pone en marcha varios recursos de Azure en su suscripción. Los recursos de Azure implementados son:

- **Clúster de AKS**: clúster de Azure Kubernetes Service que se configura según los parámetros de entrada proporcionados por el cliente. El clúster de AKS tiene varios pods configurados para ejecutar los componentes de red de Hyperledger Fabric. Los pods creados son:

  - **Herramientas de Fabric**: herramientas que son responsables de la configuración de los componentes de Hyperledger Fabric.
  - **Pods solicitantes/del mismo nivel**: Los nodos de la red de Hyperledger Fabric.
  - **Proxy**: un pod de proxy de NGNIX a través del cual las aplicaciones cliente pueden comunicarse con el clúster de AKS.
  - **Fabric CA**: El pod que ejecuta la entidad Fabric CA.
- **PostgreSQL**: instancia de base de datos que mantiene las identidades de Fabric CA.

- **Almacén de claves**: instancia del servicio Azure Key Vault que se implementa para guardar las credenciales de Fabric CA y los certificados raíz proporcionados por el cliente. El almacén se usa en caso de reintento de implementación de plantilla para controlar la mecánica de la plantilla.
- **Disco administrado**: instancia del servicio Azure Managed Disks que proporciona un almacén persistente para el libro de contabilidad y para la base de datos de estado global del nodo del mismo nivel.
- **Dirección IP pública**: punto de conexión del clúster de AKS implementado para comunicarse con el clúster.

## <a name="deploy-the-orderer-and-peer-organization"></a>Implementación de la organización solicitante y del mismo nivel

Para comenzar, necesita una suscripción a Azure que pueda admitir la implementación de varias máquinas virtuales y cuentas de almacenamiento estándar. Si no tiene una suscripción a Azure, puede [crear una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

Para empezar a trabajar con la implementación de componentes de red de Hyperledger Fabric, vaya a [Azure Portal](https://portal.azure.com).

1. Seleccione **Crear un recurso** > **Cadena de bloques** y busque **Hyperledger Fabric en Azure Kubernetes Service (versión preliminar)** .

2. Escriba los detalles del proyecto en la pestaña **Datos básicos**.

    ![Captura de pantalla que muestra la ficha Datos básicos.](./media/hyperledger-fabric-consortium-azure-kubernetes-service/create-for-hyperledger-fabric-basics.png)

3. Escriba la siguiente información:
    - **Suscripción**: elija el nombre de la suscripción en la que desea implementar los componentes de red de Hyperledger Fabric.
    - **Grupo de recursos**: cree un grupo de recursos nuevo o elija uno vacío existente. El grupo de recursos contendrá todos los recursos implementados como parte de la plantilla.
    - **Región**: elija la región de Azure en la que desea implementar el clúster de Azure Kubernetes Service para los componentes de Hyperledger Fabric. La plantilla está disponible en todas las regiones en las que AKS está disponible. Elija una región en la que la suscripción no alcance el límite de cuota de máquina virtual (VM).
    - **Prefijo de recurso**: escriba un prefijo para el nombre de los recursos que se implementan. Debe tener menos de seis caracteres y la combinación de caracteres debe incluir números y letras.
4. Seleccione la pestaña **Fabric settings** (Configuración de Fabric) para definir los componentes de la red de Hyperledger Fabric que se implementarán.

    ![Captura de pantalla que muestra la pestaña Fabric settings (Configuración de Fabric).](./media/hyperledger-fabric-consortium-azure-kubernetes-service/create-for-hyperledger-fabric-settings.png)

5. Escriba la siguiente información:
    - **Nombre de la organización**: escriba el nombre de la organización de Hyperledger Fabric, que es necesario para varias operaciones del plano de datos. El nombre de la organización debe ser único en cada implementación.
    - **Componente de red de Fabric**: elija **Ordering service** (Servicio de ordenación) o **Peer nodes** (Nodos del mismo nivel), en función del componente de red de la cadena de bloques que quiera configurar.
    - **Número de nodos**: los siguientes son los dos tipos de nodos:
        - **Ordering service** (Servicio de ordenación): seleccione el número de nodos para proporcionar tolerancia a errores a la red. El número de nodos de orden admitidos es 3, 5 y 7.
        - **Nodos del mismo nivel**: puede elegir entre 1 y 10 nodos, según sus necesidades.
    - **Base de datos de estado mundial del nodo del mismo nivel**: elija entre LevelDB y CoucbDB. Este campo se muestra cuando elige **Peer nodes** (Nodos del mismo nivel) en el menú desplegable **Fabric network component** (Componente de la red de Fabric).
    - **Fabric CA username** (Nombre de usuario de Fabric CA): escriba el nombre de usuario que se usa para la autenticación de Fabric CA.
    - **Contraseña de Fabric CA**: Escriba la contraseña para la autenticación de Fabric CA.
    - **Confirmar contraseña**: Confirme la contraseña de Fabric CA.
    - **Certificados**: si quiere usar sus propios certificados raíz para inicializar Fabric CA, elija la opción **Upload root certificate for Fabric CA** (Cargar certificado raíz para Fabric CA). De lo contrario, Fabric CA crea certificados autofirmados de forma predeterminada.
    - **Certificado raíz**: cargue el certificado raíz (clave pública) con el que se debe inicializar Fabric CA. Se admiten los certificados en formato .pem. Los certificados deben ser válidos y de una zona horaria UTC.
    - **Clave privada de certificado raíz**: Cargue la clave privada del certificado raíz. Si tiene un certificado .pem, que tiene una clave pública y privada combinada, cárguela aquí también.


6. Seleccione la pestaña **AKS cluster Settings** (Configuración del clúster de AKS) para definir la configuración del clúster de Azure Kubernetes Service, que es la infraestructura subyacente en la que se van a configurar los componentes de la red de Hyperledger Fabric.

    ![Captura de pantalla que muestra la pestaña de configuración del clúster AKS.](./media/hyperledger-fabric-consortium-azure-kubernetes-service/create-for-hyperledger-fabric-aks-cluster-settings-1.png)

7. Escriba la siguiente información:
    - **Nombre del clúster de Kubernetes**: cambie el nombre del clúster de AKS, si es necesario. Este campo se rellena previamente según el prefijo de recurso proporcionado.
    - **Versión de Kubernetes**: elija la versión de Kubernetes que se implementará en el clúster. En función de la región seleccionada en la pestaña **Datos básicos**, las versiones admitidas disponibles pueden cambiar.
    - **Prefijo de DNS**: escriba un prefijo del nombre del Sistema de nombres de dominio (DNS) para el clúster de AKS. Usará DNS para conectarse a la API de Kubernetes al administrar los contenedores después de crear el clúster.
    - **Tamaño del nodo**: para el tamaño del nodo de Kubernetes, puede elegir en la lista de referencia de almacén (SKU) de máquinas virtuales disponible en Azure. Para obtener un rendimiento óptimo, se recomienda el estándar DS3 v2.
    - **Node count** (Número de nodos): escriba el número de nodos de Kubernetes que se van a implementar en el clúster. Se recomienda mantener este número de nodos igual o superior al número de nodos de Hyperledger Fabric especificados en la pestaña **Fabric settings** (Configuración de Fabric).
    - **Identificador de cliente de la entidad de servicio**: Puede especificar un id. de cliente de una entidad de servicio existente o crear uno. Se requiere una entidad de servicio para la autenticación de AKS. Siga los [pasos para crear una entidad de servicio](/powershell/azure/create-azure-service-principal-azureps?view=azps-3.2.0#create-a-service-principal).
    - **Secreto de cliente de la entidad de servicio**: Escriba el secreto de cliente de la entidad de servicio proporcionada en el id. de cliente de la entidad de servicio.
    - **Confirmar secreto del cliente**: Confirme el secreto de cliente para la entidad de servicio.
    - **Habilitar la supervisión de contenedores**: Opte por habilitar la supervisión de AKS, que permite que los registros de AKS se inserten en el área de trabajo de Log Analytics especificada.
    - **Área de trabajo de Log Analytics**: El área de trabajo de Log Analytics se rellenará con el área de trabajo predeterminada que se crea si la supervisión está habilitada.

8. Seleccione la pestaña **Revisar y crear**. Este paso desencadena la validación de los valores proporcionados.
9. Una vez pasada la validación, seleccione **Crear**.

    La implementación suele tardar entre 10 y 12 minutos. El tiempo puede variar en función del tamaño y el número de nodos de AKS especificados.
10. Cuando la implementación se complete correctamente, recibirá una notificación a través de las notificaciones de Azure en la esquina superior derecha.
11. Seleccione **Ir al grupo de recursos** para comprobar todos los recursos creados como parte de la implementación de la plantilla. Todos los nombres de recurso comenzarán por el prefijo proporcionado en la pestaña **Datos básicos**.

## <a name="build-the-consortium"></a>Compilación del consorcio

Para compilar el consorcio de la cadena de bloques que implementa el servicio de ordenación y los nodos del mismo nivel, tiene que ejecutar los siguientes pasos en orden. El script de Azure Hyperledger Fabric (azhlf) le ayuda a configurar el consorcio, crear el canal y realizar las operaciones de código de cadena.

> [!NOTE]
> El script de Azure Hyperledger Fabric (azhlf) se ha actualizado para proporcionar más funcionalidad. Si quiere hacer referencia al script anterior, consulte [Léame en GitHub](https://github.com/Azure/Hyperledger-Fabric-on-Azure-Kubernetes-Service/blob/master/consortiumScripts/README.md). Este script es compatible con Hyperledger Fabric en la plantilla de Azure Kubernetes Service versión 2.0.0 y posteriores. Para comprobar la versión de la implementación, siga los pasos descritos en la sección [Solución de problemas](#troubleshoot).

> [!NOTE]
> El script se proporciona solo como ayuda en los escenarios de demostración, desarrollo y prueba. El canal y el consorcio que crea este script tiene directivas básicas de Hyperledger Fabric para simplificar los escenarios de demostración, desarrollo y prueba. Para la configuración de producción, se recomienda actualizar las directivas de canal/consorcio de Hyperledger Fabric de acuerdo con las necesidades de cumplimiento de su organización mediante las API nativas de Hyperledger Fabric.


Todos los comandos para ejecutar el script de Azure Hyperledger Fabric se pueden ejecutar a través de la interfaz de la línea de comandos (CLI) de Azure Bash. Puede iniciar sesión en Azure Cloud Shell a través de la opción ![plantilla de Hyperledger Fabric en Azure Kubernetes Service](./media/hyperledger-fabric-consortium-azure-kubernetes-service/arrow.png) en la esquina superior derecha de Azure Portal. En el símbolo del sistema, escriba `bash` y seleccione la tecla Entrar para cambiar a la CLI de Bash o seleccione **Bash** en la barra de herramientas de Cloud Shell.

Para más información, consulte [Azure Cloud Shell](../../cloud-shell/overview.md).

![Captura de pantalla que muestra los comandos de Azure Cloud Shell.](./media/hyperledger-fabric-consortium-azure-kubernetes-service/hyperledger-powershell.png)


En la imagen siguiente se muestra el proceso paso a paso para crear el consorcio entre una organización solicitante y una organización del mismo nivel. En las siguientes secciones se muestran los comandos detallados para completar estos pasos.

![Diagrama del proceso para crear un consorcio.](./media/hyperledger-fabric-consortium-azure-kubernetes-service/process-to-build-consortium-flow-chart.png)

Después de finalizar la configuración inicial, use la aplicación cliente para realizar las operaciones siguientes:  

- Administración de canales
- Administración de consorcios
- Administración de códigos de cadena

### <a name="download-client-application-files"></a>Descarga de los archivos de la aplicación cliente

La primera configuración consiste en descargar los archivos de la aplicación cliente. Ejecute los comandos siguientes para descargar todos los archivos y paquetes necesarios:

```bash-interactive
curl https://raw.githubusercontent.com/Azure/Hyperledger-Fabric-on-Azure-Kubernetes-Service/master/azhlfToolSetup.sh | bash
cd azhlfTool
npm install
npm run setup
```

Estos comandos clonarán el código de la aplicación cliente de Azure Hyperledger Fabric del repositorio público de GitHub y, después, cargarán todos los paquetes npm dependientes. Después de ejecutar correctamente el comando, podrá ver una carpeta node_modules en el directorio actual. Todos los paquetes necesarios se cargan en la carpeta node_modules.

### <a name="set-up-environment-variables"></a>Configurar las variables de entorno

Todas las variables de entorno siguen la convención de nomenclatura de recursos de Azure.

#### <a name="set-environment-variables-for-the-orderer-organizations-client"></a>Configuración de las variables de entorno para el cliente de la organización solicitante

```bash
ORDERER_ORG_SUBSCRIPTION=<ordererOrgSubscription>
ORDERER_ORG_RESOURCE_GROUP=<ordererOrgResourceGroup>
ORDERER_ORG_NAME=<ordererOrgName>
ORDERER_ADMIN_IDENTITY="admin.$ORDERER_ORG_NAME"
CHANNEL_NAME=<channelName>
```

#### <a name="set-environment-variables-for-the-peer-organizations-client"></a>Configuración de las variables de entorno para el cliente de la organización del mismo nivel

```bash
PEER_ORG_SUBSCRIPTION=<peerOrgSubscritpion>
PEER_ORG_RESOURCE_GROUP=<peerOrgResourceGroup>
PEER_ORG_NAME=<peerOrgName>
PEER_ADMIN_IDENTITY="admin.$PEER_ORG_NAME"
CHANNEL_NAME=<channelName>
```

Según el número de organizaciones del mismo nivel en el consorcio, es posible que deba repetir los comandos del mismo nivel y establecer la variable de entorno según corresponda.

#### <a name="set-environment-variables-for-an-azure-storage-account"></a>Establecimiento de variables de entorno para una cuenta de almacenamiento de Azure predeterminada

```bash
STORAGE_SUBSCRIPTION=<subscriptionId>
STORAGE_RESOURCE_GROUP=<azureFileShareResourceGroup>
STORAGE_ACCOUNT=<azureStorageAccountName>
STORAGE_LOCATION=<azureStorageAccountLocation>
STORAGE_FILE_SHARE=<azureFileShareName>
```

Use los siguientes comandos para crear una cuenta de almacenamiento de Azure. Si ya tiene una cuenta de almacenamiento de Azure, omita este paso.

```bash
az account set --subscription $STORAGE_SUBSCRIPTION
az group create -l $STORAGE_LOCATION -n $STORAGE_RESOURCE_GROUP
az storage account create -n $STORAGE_ACCOUNT -g  $STORAGE_RESOURCE_GROUP -l $STORAGE_LOCATION --sku Standard_LRS
```

Use los siguientes comandos para crear un recurso compartido de archivos en una cuenta de almacenamiento de Azure. Si ya tiene un recurso compartido de archivos, omita este paso.

```bash
STORAGE_KEY=$(az storage account keys list --resource-group $STORAGE_RESOURCE_GROUP  --account-name $STORAGE_ACCOUNT --query "[0].value" | tr -d '"')
az storage share create  --account-name $STORAGE_ACCOUNT  --account-key $STORAGE_KEY  --name $STORAGE_FILE_SHARE
```

Use los comandos siguientes para generar una cadena de conexión para un recurso compartido de archivos de Azure.

```bash
STORAGE_KEY=$(az storage account keys list --resource-group $STORAGE_RESOURCE_GROUP  --account-name $STORAGE_ACCOUNT --query "[0].value" | tr -d '"')
SAS_TOKEN=$(az storage account generate-sas --account-key $STORAGE_KEY --account-name $STORAGE_ACCOUNT --expiry `date -u -d "1 day" '+%Y-%m-%dT%H:%MZ'` --https-only --permissions lruwd --resource-types sco --services f | tr -d '"')
AZURE_FILE_CONNECTION_STRING=https://$STORAGE_ACCOUNT.file.core.windows.net/$STORAGE_FILE_SHARE?$SAS_TOKEN

```

### <a name="import-an-organization-connection-profile-admin-user-identity-and-msp"></a>Importación del perfil de conexión, la identidad del usuario administrador y el MSP de la organización

Use los siguientes comandos para capturar el perfil de conexión de la organización, la identidad del usuario administrador y el Proveedor de servicios administrados (MSP) del clúster de Azure Kubernetes Service y, después, almacenar estas identidades en el almacén local de la aplicación cliente. Un ejemplo de un almacén local es el directorio *azhlfTool/stores*.

Para la organización solicitante:

```bash
./azhlf adminProfile import fromAzure -o $ORDERER_ORG_NAME -g $ORDERER_ORG_RESOURCE_GROUP -s $ORDERER_ORG_SUBSCRIPTION
./azhlf connectionProfile import fromAzure -g $ORDERER_ORG_RESOURCE_GROUP -s $ORDERER_ORG_SUBSCRIPTION -o $ORDERER_ORG_NAME   
./azhlf msp import fromAzure -g $ORDERER_ORG_RESOURCE_GROUP -s $ORDERER_ORG_SUBSCRIPTION -o $ORDERER_ORG_NAME
```

Para la organización del mismo nivel:

```bash
./azhlf adminProfile import fromAzure -g $PEER_ORG_RESOURCE_GROUP -s $PEER_ORG_SUBSCRIPTION -o $PEER_ORG_NAME
./azhlf connectionProfile import fromAzure -g $PEER_ORG_RESOURCE_GROUP -s $PEER_ORG_SUBSCRIPTION -o $PEER_ORG_NAME
./azhlf msp import fromAzure -g $PEER_ORG_RESOURCE_GROUP -s $PEER_ORG_SUBSCRIPTION -o $PEER_ORG_NAME
```

### <a name="create-a-channel"></a>Crear un canal

Desde el cliente de la organización solicitante, use el siguiente comando para crear un canal que contenga solo la organización solicitante.  

```bash
./azhlf channel create -c $CHANNEL_NAME -u $ORDERER_ADMIN_IDENTITY -o $ORDERER_ORG_NAME
```

### <a name="add-a-peer-organization-for-consortium-management"></a>Adición de una organización del mismo nivel para la administración de consorcio

>[!NOTE]
> Antes de comenzar con las operaciones de consorcio, asegúrese de que ha finalizado la configuración inicial de la aplicación cliente.  

Ejecute los comandos siguientes en el orden indicado para agregar una organización del mismo nivel en un canal y consorcio: 

1.  Desde el cliente de la organización del mismo nivel, cargue el MSP de la organización del mismo nivel en Azure Storage.

      ```bash
      ./azhlf msp export toAzureStorage -f  $AZURE_FILE_CONNECTION_STRING -o $PEER_ORG_NAME
      ```
2.  Desde el cliente de la organización solicitante, descargue el MSP de la organización del mismo nivel de Azure Storage. A continuación, emita el comando para agregar la organización del mismo nivel al canal y consorcio.

      ```bash
      ./azhlf msp import fromAzureStorage -o $PEER_ORG_NAME -f $AZURE_FILE_CONNECTION_STRING
      ./azhlf channel join -c  $CHANNEL_NAME -o $ORDERER_ORG_NAME  -u $ORDERER_ADMIN_IDENTITY -p $PEER_ORG_NAME
      ./azhlf consortium join -o $ORDERER_ORG_NAME  -u $ORDERER_ADMIN_IDENTITY -p $PEER_ORG_NAME
      ```

3.  Desde el cliente de la organización solicitante, cargue el perfil de conexión del solicitante en Azure Storage. De este modo, la organización del mismo nivel puede conectarse a los nodos solicitantes con dicho perfil de conexión.

      ```bash
      ./azhlf connectionProfile  export toAzureStorage -o $ORDERER_ORG_NAME -f $AZURE_FILE_CONNECTION_STRING
      ```

4.  Desde el cliente de la organización del mismo nivel, descargue el perfil de conexión del solicitante de Azure Storage. A continuación, ejecute el comando para agregar nodos del mismo nivel al canal.

      ```bash
      ./azhlf connectionProfile  import fromAzureStorage -o $ORDERER_ORG_NAME -f $AZURE_FILE_CONNECTION_STRING
      ./azhlf channel joinPeerNodes -o $PEER_ORG_NAME  -u $PEER_ADMIN_IDENTITY -c $CHANNEL_NAME --ordererOrg $ORDERER_ORG_NAME
      ```

Del mismo modo, para agregar más organizaciones del mismo nivel al canal, actualice las variables de entorno del mismo nivel según la organización del mismo nivel requerida y vuelva a realizar los pasos 1 a 4.

### <a name="set-anchor-peers"></a>Establecimiento de pares de anclaje

Desde el cliente de la organización del mismo nivel, ejecute el comando para establecer delimitadores del mismo nivel para la organización del mismo nivel en el canal especificado.

>[!NOTE]
> Antes de ejecutar este comando, asegúrese de que la organización del mismo nivel se agrega en el canal mediante los comandos de administración del consorcio.

```bash
./azhlf channel setAnchorPeers -c $CHANNEL_NAME -p <anchorPeersList> -o $PEER_ORG_NAME -u $PEER_ADMIN_IDENTITY --ordererOrg $ORDERER_ORG_NAME
```

`<anchorPeersList>` es una lista separada por espacios de nodos del mismo nivel que se establecerán como delimitadores del mismo nivel. Por ejemplo:

  - Establezca `<anchorPeersList>` como `"peer1"` si quiere establecer solo el nodo peer1 como delimitador del mismo nivel.
  - Establezca `<anchorPeersList>` como `"peer1" "peer3"` si quiere establecer los nodos peer1 y peer3 como delimitadores del mismo nivel.

## <a name="chaincode-management-commands"></a>Comandos de administración de código de cadena

>[!NOTE]
> Antes de comenzar con las operaciones de código de cadena, asegúrese de que ha finalizado la configuración inicial de la aplicación cliente.  

### <a name="set-the-chaincode-specific-environment-variables"></a>Establecimiento de variables de entorno específicas de código de cadena

```bash
# Peer organization name where the chaincode operation will be performed
ORGNAME=<PeerOrgName>
USER_IDENTITY="admin.$ORGNAME"  
# If you are using chaincode_example02 then set CC_NAME=“chaincode_example02”
CC_NAME=<chaincodeName>  
# If you are using chaincode_example02 then set CC_VERSION=“1” for validation
CC_VERSION=<chaincodeVersion>
# Language in which chaincode is written. Supported languages are 'node', 'golang', and 'java'  
# Default value is 'golang'  
CC_LANG=<chaincodeLanguage>  
# CC_PATH contains the path where your chaincode is placed.
# If you are using chaincode_example02 to validate then CC_PATH=“/home/<username>/azhlfTool/samples/chaincode/src/chaincode_example02/go”
CC_PATH=<chaincodePath>  
# Channel on which chaincode will be instantiated/invoked/queried  
CHANNEL_NAME=<channelName>  
```

### <a name="install-chaincode"></a>Instalación del código de cadena  

Ejecute el comando siguiente para instalar el código de cadena en la organización del mismo nivel.  

```bash
./azhlf chaincode install -o $ORGNAME -u $USER_IDENTITY -n $CC_NAME -p $CC_PATH -l $CC_LANG -v $CC_VERSION  

```
El comando instalará el código de cadena en todos los nodos del mismo nivel de la organización del mismo nivel que estén establecidos en la variable de entorno `ORGNAME`. Si hay dos o más organizaciones del mismo nivel en el canal y quiere instalar el código de cadena en todas ellas, ejecute este comando por separado para cada organización del mismo nivel.  

Siga estos pasos:  

1.  Establezca `ORGNAME` y `USER_IDENTITY` según `peerOrg1` y ejecute el comando `./azhlf chaincode install`.  
2.  Establezca `ORGNAME` y `USER_IDENTITY` según `peerOrg2` y ejecute el comando `./azhlf chaincode install`.  

### <a name="instantiate-chaincode"></a>Creación de instancia del código de cadena  

Desde la aplicación cliente del mismo nivel, ejecute el comando siguiente para crear una instancia de código de cadena en el canal.  

```bash
./azhlf chaincode instantiate -o $ORGNAME -u $USER_IDENTITY -n $CC_NAME -v $CC_VERSION -c $CHANNEL_NAME -f <instantiateFunc> --args <instantiateFuncArgs>
```

Pase el nombre de la función de creación de instancias y la lista de argumentos separados por espacios en `<instantiateFunc>` y `<instantiateFuncArgs>`, respectivamente. Por ejemplo, para crear una instancia del código de cadena chaincode_example02.go, establezca `<instantiateFunc>` en `init` y `<instantiateFuncArgs>` en `"a" "2000" "b" "1000"`.

También puede pasar el archivo JSON de configuración de la recopilación mediante la marca `--collections-config`. O bien, establezca los argumentos transitorios mediante la marca `-t` al crear una instancia de un código de cadena usado para las transacciones privadas.

Por ejemplo:

```bash
./azhlf chaincode instantiate -c $CHANNEL_NAME -n $CC_NAME -v $CC_VERSION -o $ORGNAME -u $USER_IDENTITY --collections-config <collectionsConfigJSONFilePath>
./azhlf chaincode instantiate -c $CHANNEL_NAME -n $CC_NAME -v $CC_VERSION -o $ORGNAME -u $USER_IDENTITY --collections-config <collectionsConfigJSONFilePath> -t <transientArgs>
```

La parte `<collectionConfigJSONFilePath>` es la ruta de acceso al archivo JSON que contiene las recopilaciones definidas para la creación de instancias de un código de cadena de datos privados. Puede encontrar un archivo JSON de configuración de recopilaciones de ejemplo en relación con el directorio *azhlfTool* en la siguiente ruta de acceso: `./samples/chaincode/src/private_marbles/collections_config.json`.
Pase `<transientArgs>` como JSON válido en formato de cadena. Aplique escape a cualquier carácter especial. Por ejemplo: `'{\\\"asset\":{\\\"name\\\":\\\"asset1\\\",\\\"price\\\":99}}'`

> [!NOTE]
> Ejecute el comando una vez desde una organización del mismo nivel en el canal. Después de enviar correctamente la transacción al solicitante, este la distribuye a todas las organizaciones del mismo nivel del canal. A continuación, se crea una instancia del código de cadena en todos los nodos del mismo nivel en todas las organizaciones del mismo nivel del canal.  

### <a name="invoke-chaincode"></a>Invocación del código de cadena  

Desde el cliente de la organización del mismo nivel, ejecute el siguiente comando para invocar la función del código de cadena:  

```bash
./azhlf chaincode invoke -o $ORGNAME -u $USER_IDENTITY -n $CC_NAME -c $CHANNEL_NAME -f <invokeFunc> -a <invokeFuncArgs>  
```

Pase el nombre de la función de invocación y la lista de argumentos separados por espacios en  `<invokeFunction>`  y  `<invokeFuncArgs>` , respectivamente. Siguiendo con el ejemplo de código de cadena chaincode_example02.go, para realizar una operación de invocación, establezca `<invokeFunction>` en `invoke` y `<invokeFuncArgs>` en `"a" "b" "10"`.  

>[!NOTE]
> Ejecute el comando una vez desde una organización del mismo nivel en el canal. Después de enviar correctamente la transacción al solicitante, este la distribuye a todas las organizaciones del mismo nivel del canal. A continuación, el estado global se actualiza en todos los nodos del mismo nivel de todas las organizaciones del mismo nivel del canal.  


### <a name="query-chaincode"></a>Consulta del código de cadena  

Ejecute el siguiente comando para consultar el código de cadena:  

```bash
./azhlf chaincode query -o $ORGNAME -p <endorsingPeers> -u $USER_IDENTITY -n $CC_NAME -c $CHANNEL_NAME -f <queryFunction> -a <queryFuncArgs> 
```
Los pares de aprobación son pares en los que se instala el código de cadena y se le llama para la ejecución de transacciones. Debe establecer el elemento `<endorsingPeers>` para que contenga los nombres de nodo del mismo nivel de la organización del mismo nivel actual. Enumere los pares de aprobación para una combinación de canal y código de cadena determinada separados por espacios. Por ejemplo: `-p "peer1" "peer3"`.

Si usa *azhlfTool* para instalar el código de cadena, pase los nombres de nodo del mismo nivel como un valor al argumento del par de aprobación. El código de cadena se instala en todos los nodos del mismo nivel para esa organización. 

Pase el nombre de la función de consulta y la lista de argumentos separados por espacios en `<queryFunction>` y `<queryFuncArgs>` , respectivamente. De nuevo, con el código de cadena chaincode_example02.go como referencia, para consultar el valor "a" en el conjunto de estado global, establezca  `<queryFunction>` en `query` y `<queryArgs>` en `"a"`.  

## <a name="troubleshoot"></a>Solución de problemas

Ejecute los siguientes comandos para buscar la versión de implementación de la plantilla.

Establezca las siguientes variables de entorno según el grupo de recursos en el que haya implementado la plantilla.

```bash

SWITCH_TO_AKS_CLUSTER() { az aks get-credentials --resource-group $1 --name $2 --subscription $3; }
AKS_CLUSTER_SUBSCRIPTION=<AKSClusterSubscriptionID>
AKS_CLUSTER_RESOURCE_GROUP=<AKSClusterResourceGroup>
AKS_CLUSTER_NAME=<AKSClusterName>
```
Ejecute el comando siguiente para imprimir la versión de la plantilla.

```bash
SWITCH_TO_AKS_CLUSTER $AKS_CLUSTER_RESOURCE_GROUP $AKS_CLUSTER_NAME $AKS_CLUSTER_SUBSCRIPTION
kubectl describe pod fabric-tools -n tools | grep "Image:" | cut -d ":" -f 3

```

## <a name="support-and-feedback"></a>Soporte y comentarios

Para mantenerse al día sobre las ofertas de servicio de Blockchain y la información del equipo de ingeniería de Azure Blockchain, visite el [blog de Azure Blockchain](https://azure.microsoft.com/blog/topics/blockchain/).

Para proporcionar comentarios sobre el producto o solicitar nuevas características, publique o vote una idea a través del [Foro de comentarios de Azure para Blockchain](https://aka.ms/blockchainuservoice).

### <a name="community-support"></a>Soporte técnico de la comunidad

Interactúe con los ingenieros de Microsoft y con expertos de la comunidad de Azure Blockchain:

- [Página de preguntas y respuestas de Microsoft](/answers/topics/azure-blockchain-workbench.html) 
   
  El soporte técnico para plantillas con cadena de bloques se limita a los problemas de implementación.
- [Comunidad tecnológica de Microsoft](https://techcommunity.microsoft.com/t5/Blockchain/bd-p/AzureBlockchain)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-blockchain-workbench)
