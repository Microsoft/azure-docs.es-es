---
title: 'Creación de un clúster en Azure HDInsight: lista de errores'
description: Obtenga información sobre cómo solucionar los errores que se producen al crear clústeres de Azure HDInsight
author: karkrish
ms.author: v-todmc
ms.reviewer: hrasheed
ms.service: hdinsight
ms.topic: troubleshooting
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 08/24/2020
ms.openlocfilehash: cae8647d970020a22d59dc49b058d43fe28dd00c
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816463"
---
# <a name="azure-hdinsight-cluster-creation-errors"></a>Azure HDInsight: Errores de creación de clúster

En este artículo se describen las soluciones a errores que pueden surgir al crear clústeres.

> [!NOTE]
> Los tres primeros errores descritos en este artículo son errores de validación. Pueden producirse cuando un producto de Azure HDInsight usa la clase **CsmDocument_2_0**.

## <a name="error-codedeploymentdocument-csmdocument_2_0-failed-the-validation"></a>Código de error: Error de DeploymentDocument "CsmDocument_2_0" para su validación.

**Error**: "No se puede acceder a la ubicación de la acción de script con el URI: \<SCRIPT ACTION URL\>"

### <a name="error-message-1"></a>Mensaje de error 1

"El servidor remoto devolvió un error: 404 No encontrado".

#### <a name="cause"></a>Causa

El servicio HDInsight no puede acceder a la dirección URL de la acción de script que proporcionó como parte de la solicitud de creación del clúster. El servicio recibe el mensaje de error anterior cuando intenta acceder a la acción de script.

#### <a name="resolution"></a>Solución

- En el caso de una dirección URL HTTP o HTTPS, compruebe la dirección URL; para ello, intente ir a ella desde una ventana del explorador de incógnito.
- Si es una dirección URL de WASB, asegúrese de que el script existe en la cuenta de almacenamiento que se especifica en la solicitud. Además, asegúrese de que la clave de almacenamiento de esta cuenta de almacenamiento sea correcta.
- Si es una dirección URL de ADLS, asegúrese de que el script existe en la cuenta de almacenamiento.

---

### <a name="error-message-2"></a>Mensaje de error 2

"El URI de script especificado \<SCRIPT_URI\> está en ADLS, pero este clúster no tiene ninguna entidad de seguridad de Data Lake Storage"

#### <a name="cause"></a>Causa

El servicio HDInsight no puede acceder a la dirección URL de la acción de script que proporcionó como parte de la solicitud de creación del clúster. El servicio recibe el mensaje de error anterior cuando intenta acceder a la acción de script.

#### <a name="resolution"></a>Solución

Agregue al clúster la cuenta de Azure Data Lake Storage Gen 1 correspondiente. Agregue también al clúster la entidad de servicio que tiene acceso a la cuenta de Data Lake Storage Gen 1.

---

### <a name="error-message-3"></a>Mensaje de error 3

"El tamaño de la máquina virtual \<CUSTOMER_SPECIFIED_VM_SIZE\> proporcionado en la solicitud no es válido o no se admite para el rol \<ROLE\>. Los valores válidos son: \<VALID_VM_SIZE_FOR_ROLE\>".

#### <a name="cause"></a>Causa

El tamaño de la máquina virtual que especificó no está permitido para el rol. Este error puede producirse porque el valor de tamaño de la máquina virtual no funciona como se esperaba o no es adecuado para el rol de equipo.

#### <a name="resolution"></a>Solución

El mensaje de error muestra los valores válidos de tamaño de máquina virtual. Seleccione uno de estos valores e intente realizar de nuevo la solicitud de creación del clúster.

---

## <a name="error-codeinvalidvirtualnetworkid"></a>Código de error: InvalidVirtualNetworkId  

### <a name="error"></a>Error

"The VirtualNetworkId is not valid. VirtualNetworkId '\<USER_VIRTUALNETWORKID\>'*"

### <a name="cause"></a>Causa

El valor de **VirtualNetworkId** especificado durante la creación del clúster no tiene el formato correcto.

### <a name="resolution"></a>Solución

Asegúrese de que los valores de **VirtualNetworkId** y de la subred tengan el formato correcto. Para obtener el valor de **VirtualNetworkId**:

1. Vaya a Azure Portal.
1. Seleccione la red virtual.
1. Seleccione el elemento de menú **Propiedades**. El valor de la propiedad **ResourceID** es el valor de **VirtualNetworkId**.

Este es un ejemplo de identificador de red virtual:

"/subscriptions/c15fd9b8-e2b8-1d4e-aa85-2e668040233b/resourceGroups/myresourcegroup/providers/Microsoft.Network/virtualNetworks/myvnet"

---

## <a name="error-code-customizationfailederrorcode"></a>Código de error: CustomizationFailedErrorCode

### <a name="error"></a>Error

"Cluster deployment failed due to an error in the custom script action. Acciones con error: \<SCRIPT_NAME\>. Vaya a la interfaz de usuario de Ambari para seguir depurando el error".

### <a name="cause"></a>Causa

El script personalizado que se proporcionó durante la solicitud de creación del clúster se ejecuta después de que el clúster se haya implementado correctamente. Este código de error indica que se ha producido un error durante la ejecución del script personalizado denominado \<SCRIPT_NAME\>.

### <a name="resolution"></a>Resolución

Dado que el script es el script personalizado, le recomendamos que solucione el problema y, si es necesario, vuelva a ejecutarlo. Para solucionar el error del script, examine los registros de la carpeta /var/lib/ambari-agent/*. También puede abrir la página de **operaciones** de la interfaz de usuario de Ambari y seleccionar la operación **run_customscriptionaction** para ver los detalles del error.

---

## <a name="error-codeinvaliddocumenterrorcode"></a>Código de error: InvalidDocumentErrorCode

### <a name="error"></a>Error

"La versión del esquema de metastore de \<META_STORE_TYPE\> \<METASTORE_MAJOR_VERSION\> de la base de datos \<DATABASE_NAME\> no es compatible con la versión del clúster \<CLUSTER_VERSION\>"

### <a name="cause"></a>Causa

El repositorio metastore personalizado es incompatible con la versión de clúster de HDInsight seleccionada. Actualmente, los clústeres de HDInsight 4.0 solo admiten la versión 3.0 de Metastore y posterior, mientras que los clústeres de HDInsight 3.6 no admiten la versión 3.0 y posterior de Metastore.

### <a name="resolution"></a>Solución

Use solo las versiones de Metastore que admita la versión del clúster de HDInsight. Si no se especifica un almacén de metadatos personalizado, HDInsight crea internamente uno y, luego, lo elimina tras la eliminación del clúster.

---

## <a name="error-code-failedtoconnectwithclustererrorcode"></a>Código de error: FailedToConnectWithClusterErrorCode 

### <a name="error"></a>Error

"Unable to connect to cluster management endpoint to perform scaling operation. Verify that network security rules are not blocking external access to the cluster, and that the cluster manager (Ambari) UI can be successfully accessed". (No se puede conectar con el punto de conexión de administración de clúster para realizar la operación de escalado. Compruebe que no haya reglas de seguridad de red que bloqueen el acceso externo al clúster y que se pueda acceder correctamente a la interfaz de usuario del administrador de clústeres [Ambari])

### <a name="cause"></a>Causa

Tiene una regla de firewall en el grupo de seguridad de red (NSG) que está bloqueando la comunicación del clúster con los servicios críticos de mantenimiento y administración de Azure.

### <a name="resolution"></a>Solución

Si tiene previsto usar grupos de seguridad de red para controlar el tráfico de red, realice las siguientes acciones antes de instalar HDInsight:

- Identificar la región de Azure que va a usar para HDInsight.
- Identificar las direcciones IP requeridas para HDInsight. Para más información, consulte [Direcciones IP de administración de HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-management-ip-addresses).
  - Cree o modifique los grupos de seguridad de red de la subred en la que tiene previsto instalar HDInsight.
  - Para los grupos de seguridad de red, permita el tráfico entrante en el puerto 443 desde las direcciones IP. Esta configuración garantiza que los servicios de administración de HDInsight puedan comunicarse con el clúster desde fuera de la red virtual.

---

## <a name="error-code-storagepermissionsblockedformsi"></a>Código de error: StoragePermissionsBlockedForMsi

### <a name="error"></a>Error

"The Managed Identity does not have permissions on the storage account. Compruebe que el rol "Propietario de datos de Storage Blob" está asignado a la identidad administrada en la cuenta de almacenamiento. Storage: /subscriptions/ \<Subscription ID\> /resourceGroups/\< Resource Group Name\> /providers/Microsoft.Storage/storageAccounts/ \<Storage Account Name\>, Managed Identity: /subscriptions/ \<Subscription ID\> /resourceGroups/ /\< Resource Group Name\> /providers/Microsoft.ManagedIdentity/userAssignedIdentities/ \<User Managed Identity Name\>"

### <a name="cause"></a>Causa

No proporcionó los permisos necesarios para administrar la identidad. La identidad administrada asignada por el usuario no tiene el rol de colaborador de Blob Storage en la cuenta de almacenamiento de Azure Data Lake Storage Gen2.

### <a name="resolution"></a>Solución

1. Abra Azure Portal.
1. Vaya a la cuenta de almacenamiento.
1. Mire en **Access Control (IAM)** .
1. Asegúrese de que el usuario tenga asignado el rol de colaborador de datos de blob de almacenamiento o el rol de propietario de datos de blob de almacenamiento.

Para más información, vea [Configuración de permisos para la identidad administrada en la cuenta de Data Lake Storage Gen2](hdinsight-hadoop-use-data-lake-storage-gen2.md).

---

## <a name="error-code-invalidnetworksecuritygroupsecurityrules"></a>Código de error: InvalidNetworkSecurityGroupSecurityRules

### <a name="error"></a>Error

"Las reglas de seguridad del grupo de seguridad de red /subscriptions/\<SubscriptionID\>/resourceGroups/<Nombre del grupo de recursos\> default/providers/Microsoft.Network/networkSecurityGroups/\<Network Security Group Name\> configurado con la subred subnet /subscriptions/\<SubscriptionID\>/resourceGroups/\<Resource Group name\> RG-westeurope-vnet-tomtom-default/providers/Microsoft.Network/virtualNetworks/\<Virtual Network Name\>/subnets/\<Subnet Name\> no permiten la conectividad entrante o saliente necesaria. For more information, please visit [Plan a virtual network for Azure HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-plan-virtual-network-deployment), or contact support". (Las reglas de seguridad del grupo de seguridad de red /subscriptions/<SubscriptionID>/resourceGroups/<nombre del grupo de recursos> default/providers/Microsoft.Network/networkSecurityGroups/<nombre del grupo de seguridad de red> configurado con la subred /subscriptions/<SubscriptionID>/resourceGroups/<nombre del grupo de recursos> RG-westeurope-vnet-tomtom-default/providers/Microsoft.Network/virtualNetworks/<nombre de la red virtual>/subnets/<nombre de subred> no permite la conectividad de entrada o salida requerida. Para más información, visite <Planificación de una red virtual para Azure HDInsight> o póngase en contacto con el servicio de soporte técnico)

### <a name="cause"></a>Causa

Si grupos de seguridad de red o rutas definidas por el usuario (UDR) controlan el tráfico entrante al clúster de HDInsight, asegúrese de que el clúster pueda comunicarse con los servicios críticos de mantenimiento y administración de Azure.

### <a name="resolution"></a>Solución

Si tiene previsto usar grupos de seguridad de red para controlar el tráfico de red, realice las siguientes acciones antes de instalar HDInsight:

- Identifique la región de Azure que va a usar para HDInsight y cree una lista segura de las direcciones IP de su región. Para más información, consulte [Servicios de mantenimiento y administración: regiones específicas](https://docs.microsoft.com/azure/hdinsight/hdinsight-management-ip-addresses#health-and-management-services-specific-regions).
- Identifique las direcciones IP que necesita HDInsight. Para más información, vea  [Direcciones IP de administración de HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-management-ip-addresses).
- Cree o modifique los grupos de seguridad de red de la subred en la que tiene previsto instalar HDInsight. Para los grupos de seguridad de red, permita el tráfico entrante en el puerto 443 desde las direcciones IP. Esta configuración garantiza que los servicios de administración de HDInsight puedan comunicarse con el clúster desde fuera de la red virtual.

---

## <a name="error-code-cluster-setup-failed-to-install-components-on-one-or-more-hosts"></a>Código de error: El programa de instalación del clúster no pudo instalar los componentes en uno o más hosts.

### <a name="error"></a>Error

"Cluster setup failed to install components on one or more hosts. Please retry your request". (El programa de instalación del clúster no pudo instalar los componentes en uno o más hosts. Vuelva a intentar realizar la solicitud)

### <a name="cause"></a>Causa 

Este error se genera normalmente cuando hay un problema transitorio o una interrupción de Azure.

### <a name="resolution"></a>Solución

Consulte la página [Estado de Azure](https://status.azure.com) para ver las interrupciones de Azure que podrían afectar a la implementación del clúster. Si no existen interrupciones, intente implementar el clúster de nuevo.

---

## <a name="error-code-failedtoconnectwithclustererrorcode"></a>Código de error: FailedToConnectWithClusterErrorCode

### <a name="error"></a>Error

No se puede conectar con el punto de conexión de administración de clúster. Inténtelo de nuevo más tarde.

### <a name="cause"></a>Causa

El servicio HDInsight no se puede conectar al clúster mientras se intenta crear el clúster

### <a name="resolution"></a>Solución

Si usa rutas de grupo de seguridad de red (NSG) de VNet y por el usuario (UDR) personalizadas, asegúrese de que el clúster puede comunicarse con los servicios de administración de HDInsight. Para obtener información adicional, consulte [Direcciones IP de administración de HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-management-ip-addresses).

---

## <a name="error-code-deployments-failed-due-to-policy-violation-resource-resource-uri-was-disallowed-by-policy-policy-identifiers-policyassignmentnamepolicy-name-idprovidersmicrosoftmanagementmanagementgroupsmanagement-group-name-providersmicrosoftauthorizationpolicyassignmentspolicy-namepolicydefinition-policy-definition"></a>Código de error: No se pudieron realizar las implementaciones debido a una infracción de directiva: La directiva no permitió el recurso "<Resource URI>". Identificadores de directiva: '[{"policyAssignment":{"name":"<Policy Name> ","id":"/providers/Microsoft.Management/managementGroups/<Management Group Name> providers/Microsoft.Authorization/policyAssignments/<Policy Name>"},"policyDefinition": <Policy Definition>

### <a name="cause"></a>Causa

Las directivas de Azure basadas en suscripciones pueden denegar la creación de direcciones IP públicas. Para crear clústeres de HDInsight se necesitan dos direcciones IP públicas.

Normalmente, las siguientes directivas afectan a la creación del clúster:

* Directivas que impiden la creación de direcciones IP y equilibradores de carga en la suscripción.
* Directiva que impide la creación de cuentas de almacenamiento.
* Directiva que impide la eliminación de recursos de red, como direcciones IP o equilibradores de carga.

### <a name="resolution"></a>Resolución

Elimine o deshabilite la asignación de Azure Policy basada en suscripciones al crear el clúster de HDInsight.

---

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la solución de errores de creación de clústeres, consulte [Solución de errores de creación de clústeres con Azure HDInsight](https://docs.microsoft.com/azure/hdinsight/hadoop/hdinsight-troubleshoot-cluster-creation-fails).
