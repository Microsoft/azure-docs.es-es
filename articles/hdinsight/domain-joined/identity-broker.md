---
title: 'Uso del Agente de identidad (versión preliminar) para la administración de credenciales: Azure HDInsight'
description: Obtenga información sobre el Agente de identidad de HDInsight para simplificar la autenticación de clústeres de Apache Hadoop unidos a un dominio.
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 12/12/2019
ms.openlocfilehash: 3b2807ccd6d83511dd0c9a32a177ea9fe2c4b642
ms.sourcegitcommit: f8d2ae6f91be1ab0bc91ee45c379811905185d07
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89662092"
---
# <a name="use-id-broker-preview-for-credential-management"></a>Uso del Agente de identidad (versión preliminar) para la administración de credenciales

En este artículo se describe cómo configurar y usar la característica Agente de identidad en Azure HDInsight. Puede usar esta característica para iniciar sesión en Apache Ambari mediante Multi-Factor Authentication y obtener los vales de Kerberos necesarios sin necesidad de hash de contraseña en Azure Active Directory Domain Services (Azure AD DS).

## <a name="overview"></a>Información general

El Agente de identidad simplifica las configuraciones de autenticación complejas en los escenarios siguientes:

* La organización se basa en la federación para autenticar a los usuarios para acceder a los recursos en la nube. Anteriormente, para usar clústeres de Enterprise Security Package (ESP) de HDInsight, tenía que habilitar la sincronización de hash de contraseñas desde el entorno local a Azure Active Directory. Este requisito puede ser difícil o no deseable para algunas organizaciones.

* Está en el proceso de creación de soluciones que usan tecnologías que dependen de mecanismos de autenticación distintos. Por ejemplo, Apache Hadoop y Apache Ranger se basan en Kerberos, mientras que Azure Data Lake Storage se basa en OAuth.

El Agente de identidad proporciona una infraestructura de autenticación unificada y elimina el requisito de sincronizar los hash de contraseña con Azure AD DS. El Agente de identidad consta de componentes que se ejecutan en una máquina virtual con Windows Server (nodo del Agente de identidad), junto con los nodos de puerta de enlace de clúster. 

En el diagrama siguiente se muestra el flujo de autenticación para todos los usuarios, incluidos los usuarios federados, una vez habilitado el Agente de identidad:

![Flujo de autenticación con el Agente de identidad](./media/identity-broker/identity-broker-architecture.png)

El Agente de identidad permite iniciar sesión en clústeres de ESP mediante Multi-Factor Authentication, sin proporcionar ninguna contraseña. Si ya ha iniciado sesión en otros servicios de Azure, como Azure Portal, puede iniciar sesión en el clúster de HDInsight con una experiencia de inicio de sesión único (SSO).

## <a name="enable-hdinsight-id-broker"></a>Habilitación del Agente de identidad de HDInsight

Para crear un clúster de ESP con el Agente de identidad habilitado, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Siga los pasos de creación básicos para un clúster de ESP. Para más información, consulte [Creación de un clúster de HDInsight con ESP](apache-domain-joined-configure-using-azure-adds.md#create-an-hdinsight-cluster-with-esp).
1. Seleccione **Habilitar Agente de identidad de HDInsight**.

La característica Agente de identidad agregará una máquina virtual adicional al clúster. Esta máquina virtual es el nodo del Agente de identidad e incluye componentes de servidor para admitir la autenticación. El nodo del Agente de identidad está unido al dominio de Azure AD DS.

![Opción para habilitar el Agente de identidad](./media/identity-broker/identity-broker-enable.png)

### <a name="using-azure-resource-manager-templates"></a>Uso de plantillas del Administrador de recursos de Azure
Si agrega un nuevo rol denominado `idbrokernode` con los siguientes atributos al perfil de proceso de la plantilla, el clúster se creará con el nodo de agente de identidad habilitado:

```json
.
.
.
"computeProfile": {
    "roles": [
        {
            "autoscale": null,
            "name": "headnode",
           ....
        },
        {
            "autoscale": null,
            "name": "workernode",
            ....
        },
        {
            "autoscale": null,
            "name": "idbrokernode",
            "targetInstanceCount": 1,
            "hardwareProfile": {
                "vmSize": "Standard_A2_V2"
            },
            "virtualNetworkProfile": {
                "id": "string",
                "subnet": "string"
            },
            "scriptActions": [],
            "dataDisksGroups": null
        }
    ]
}
.
.
.
```

## <a name="tool-integration"></a>Integración de herramientas

El [complemento IntelliJ](https://docs.microsoft.com/azure/hdinsight/spark/apache-spark-intellij-tool-plugin#integrate-with-hdinsight-identity-broker-hib) de HDInsight se ha actualizado para admitir OAuth. Puede usar este complemento para conectarse al clúster y enviar trabajos.

También puede usar [Herramientas de Spark y Hive para VS Code](https://docs.microsoft.com/azure/hdinsight/hdinsight-for-vscode) a fin de aprovechar los trabajos de cuadernos y de envío.

## <a name="ssh-access-without-a-password-hash-in-azure-ad-ds"></a>Acceso mediante SSH sin hash de contraseña en Azure AD DS

Una vez habilitado el Agente de identidad, todavía necesitará un hash de contraseña almacenado en Azure AD DS para escenarios de SSH con cuentas de dominio. Para acceder mediante SSH a una máquina virtual unida a un dominio o para ejecutar el comando `kinit`, debe proporcionar una contraseña. 

La autenticación de SSH requiere que el hash esté disponible en Azure AD DS. Si desea usar SSH solo para escenarios administrativos, puede crear una cuenta solo en la nube y usarla para acceder mediante SSH al clúster. Otros usuarios pueden usar las herramientas de Ambari o HDInsight (por ejemplo, el complemento IntelliJ) sin tener el hash de contraseña disponible en Azure AD DS.

Para solucionar problemas de autenticación, consulte esta [guía](https://docs.microsoft.com/azure/hdinsight/domain-joined/domain-joined-authentication-issues).

## <a name="clients-using-oauth-to-connect-to-hdinsight-gateway-with-id-broker-setup"></a>Clientes que usan OAuth para conectarse a la puerta de enlace de HDInsight con la configuración del agente de identidad

En la configuración del agente de identidad, las aplicaciones personalizadas y los clientes que se conectan a la puerta de enlace se pueden actualizar para adquirir primero el token de OAuth que necesitan. Puede seguir los pasos que aparecen en este [documento](https://docs.microsoft.com/azure/storage/common/storage-auth-aad-app) para adquirir el token con la información siguiente:

*   URI del recurso de OAuth: `https://hib.azurehdinsight.net` 
* AppId: 7865c1d2-f040-46cc-875f-831a1ef6a28a
*   Permiso: (nombre: Cluster.ReadWrite; id.: 8f89faa0-ffef-4007-974d-4989b39ad77d)

Después de adquirir el token de OAuth, puede usarlo en el encabezado de autorización de la solicitud HTTP para la puerta de enlace del clúster (por ejemplo, <clustername>-int.azurehdinsight.net). Por ejemplo, un comando cURL de ejemplo para la Livy API podría ser similar al siguiente:
    
```bash
curl -k -v -H "Authorization: Bearer Access_TOKEN" -H "Content-Type: application/json" -X POST -d '{ "file":"wasbs://mycontainer@mystorageaccount.blob.core.windows.net/data/SparkSimpleTest.jar", "className":"com.microsoft.spark.test.SimpleFile" }' "https://<clustername>-int.azurehdinsight.net/livy/batches" -H "X-Requested-By:<username@domain.com>"
``` 

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de un clúster de HDInsight con Enterprise Security Package mediante Azure Active Directory Domain Services](apache-domain-joined-configure-using-azure-adds.md)
* [Sincronización de los usuarios de Azure Active Directory en un clúster de HDInsight](../hdinsight-sync-aad-users-to-cluster.md)
* [Supervisión del rendimiento del clúster](../hdinsight-key-scenarios-to-monitor.md)
