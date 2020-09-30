---
title: Seguridad de la empresa
titleSuffix: Azure Machine Learning
description: 'Use Azure Machine Learning de forma segura: autenticación, autorización, seguridad de red, cifrado de datos y supervisión.'
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 09/09/2020
ms.openlocfilehash: af32be357899090e0df96e2c67910a4f9ad5194d
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90988082"
---
# <a name="enterprise-security-for-azure-machine-learning"></a>Seguridad de empresa para Azure Machine Learning

En este artículo conocerá las características de seguridad disponibles con Azure Machine Learning.

Cuando se usa un servicio en la nube, se recomienda restringir el acceso solo a los usuarios que lo necesiten. Para empezar, es necesario comprender el modelo de autenticación y autorización que usa el servicio. Es posible que también quiera restringir el acceso a la red o unir de forma segura los recursos de la red local con la nube. También es fundamental el cifrado de datos, tanto si están en reposo como si se mueven entre los servicios. Por último, debe ser capaz de supervisar el servicio y generar un registro de auditoría de todas las actividades.

> [!NOTE]
> La información de este artículo se aplica a la versión 1.0.83.1 o a versiones posteriores del SDK de Python de Azure Machine Learning.

## <a name="authentication"></a>Authentication

Si Azure Active Directory (Azure AD) se ha configurado para usar la autenticación multifactor, también se admite aquí. Este es el proceso de autenticación:

1. El cliente inicia sesión en Azure AD y recibe un token de Azure Resource Manager.  Los usuarios y las entidades de servicio son totalmente compatibles.
1. El cliente presenta el token a Azure Resource Manager y a todas las instancias de Azure Machine Learning.
1. Machine Learning Service proporciona un token al destino de proceso de usuario (por ejemplo, Proceso de Machine Learning). El destino de proceso de usuario usa este token para volver a llamar a Machine Learning Service una vez completada la ejecución. El ámbito se limita al área de trabajo.

[![Autenticación en Azure Machine Learning](media/concept-enterprise-security/authentication.png)](media/concept-enterprise-security/authentication.png#lightbox)

Para más información, vea el artículo [Configuración de la autenticación para recursos y flujos de trabajo de Azure Machine Learning](how-to-setup-authentication.md). En este artículo se proporcionan información y ejemplos de autenticación, como el uso de entidades de servicio y flujos de trabajo automatizados.

### <a name="authentication-for-web-service-deployment"></a>Autenticación para la implementación de servicios web

Azure Machine Learning admite dos formas de autenticación de los servicios web: una clave y un token. Cada servicio web solo puede permitir una forma de autenticación a la vez.

|Método de autenticación|Descripción|Azure Container Instances|AKS|
|---|---|---|---|
|Clave|Las claves son estáticas y no es necesario actualizarlas. Las claves se pueden regenerar manualmente.|Deshabilitado de forma predeterminada| Habilitado de forma predeterminada|
|Token|Los tokens expiran después de un período de tiempo especificado y se deben actualizar.| No disponible| Deshabilitado de forma predeterminada |

Para consultar ejemplos de código, vaya a la sección [Autenticación de servicio web](how-to-setup-authentication.md#web-service-authentication).

## <a name="authorization"></a>Authorization

Puede crear varias áreas de trabajo, y cada área de trabajo se puede compartir entre varias personas. Al compartir un área de trabajo, puede controlar el acceso a ella si asigna los siguientes roles a los usuarios:

* Propietario
* Colaborador
* Lector

En la tabla siguiente se muestran algunas de las principales operaciones de Azure Machine Learning y los roles que pueden realizarlas:

| Operación de Azure Machine Learning | Propietario | Colaborador | Lector |
| ---- |:----:|:----:|:----:|
| Creación del espacio de trabajo | ✓ | ✓ | |
| Compartir área de trabajo | ✓ | |  |
| Crear el destino de proceso | ✓ | ✓ | |
| Asociar el destino de proceso | ✓ | ✓ | |
| Asociar almacenes de datos | ✓ | ✓ | |
| Ejecutar experimento | ✓ | ✓ | |
| Ver ejecuciones/métricas | ✓ | ✓ | ✓ |
| Registro del modelo | ✓ | ✓ | |
| Crear una imagen | ✓ | ✓ | |
| Implementar el servicio web | ✓ | ✓ | |
| Ver modelos/imágenes | ✓ | ✓ | ✓ |
| Llamar a un servicio web | ✓ | ✓ | ✓ |

Si los roles integrados no satisfacen sus necesidades, puede crear roles personalizados. Los roles personalizados se admiten para controlar todas las operaciones dentro de un área de trabajo, como la creación de un proceso, el envío de una ejecución, el registro de un almacén de datos o la implementación de un modelo. Los roles personalizados pueden tener permisos de lectura, escritura o eliminación en los distintos recursos de un área de trabajo, como clústeres, almacenes de datos, modelos y puntos de conexión. Puede hacer que el rol esté disponible en un nivel de área de trabajo específico, un nivel de grupo de recursos específico o un nivel de suscripción específico. Para más información, consulte [Administración de usuarios y roles en un área de trabajo de Azure Machine Learning](how-to-assign-roles.md).

> [!WARNING]
> Azure Machine Learning es compatible con la colaboración de negocio a negocio de Azure Active Directory, pero no es compatible actualmente con la colaboración de negocio a consumidor de Azure Active Directory.

### <a name="securing-compute-targets-and-data"></a>Protección de los datos y destinos de proceso

Los propietarios y colaboradores pueden usar todos los destinos de proceso y almacenes de datos conectados al área de trabajo.  

Cada área de trabajo también tiene una identidad administrada asignada por el sistema asociada con el mismo nombre que el área de trabajo. La identidad administrada tiene los siguientes permisos en los recursos asociados usados en el área de trabajo.

Para más información sobre las identidades administradas, consulte [Identidades administradas para recursos de Azure](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

| Recurso | Permisos |
| ----- | ----- |
| Área de trabajo | Colaborador |
| Cuenta de almacenamiento | Colaborador de datos de blobs de almacenamiento |
| Almacén de claves | Acceso a todas las claves, secretos, certificados |
| Azure Container Registry | Colaborador |
| El grupo de recursos que contiene el área de trabajo | Colaborador |
| El grupo de recursos que contiene el almacén de claves (si es diferente al que contiene el área de trabajo) | Colaborador |

No se recomienda que los administradores revoquen el acceso de la identidad administrada a los recursos mencionados en la tabla anterior. Puede restaurar el acceso mediante la operación de resincronización de claves.

Azure Machine Learning crea una aplicación adicional (el nombre empieza por `aml-` o `Microsoft-AzureML-Support-App-`) con acceso de nivel de colaborador en la suscripción para cada región del área de trabajo. Por ejemplo, si tiene un área de trabajo en la región Este de EE. UU. y otra en la región Norte de Europa en la misma suscripción, verá dos de estas aplicaciones. Estas aplicaciones permiten que Azure Machine Learning le ayude a administrar los recursos de proceso.

## <a name="network-security"></a>Seguridad de las redes

Azure Machine Learning depende de otros servicios de Azure para los recursos de proceso. Los recursos de proceso (destinos de proceso) se usan para entrenar e implementar modelos. Puede crear estos destinos de proceso en una red virtual. Por ejemplo, puede usar Azure Data Science Virtual Machine para entrenar un modelo y, después, implementarlo en AKS.  

Para más información, consulte [Información general sobre aislamiento y privacidad de redes virtuales](how-to-network-security-overview.md).

También puede habilitar Azure Private Link para el área de trabajo. Private Link le permite restringir las comunicaciones con el área de trabajo desde una instancia de Azure Virtual Network. Para más información, consulte [Configuración de Private Link](how-to-configure-private-link.md).

## <a name="data-encryption"></a>Cifrado de datos

> [!IMPORTANT]
> Para el cifrado de nivel de producción durante el __aprendizaje__, Microsoft recomienda usar el clúster de proceso de Azure Machine Learning. Para el cifrado de nivel de producción durante la __inferencia__, Microsoft recomienda usar Azure Kubernetes Service.
>
> La instancia de proceso de Azure Machine Learning es un entorno de desarrollo/pruebas. Cuando se usa, se recomienda almacenar los archivos, como cuadernos s y scripts, en un recurso compartido de archivos. Los datos deben almacenarse en un almacén de datos.

### <a name="encryption-at-rest"></a>Cifrado en reposo

> [!IMPORTANT]
> Si su área de trabajo contiene datos confidenciales, se recomienda establecer el parámetro [hbi_workspace](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py#&preserve-view=truecreate-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--exist-ok-false--show-output-true-) a la hora de crearla. La marca `hbi_workspace` solo se puede establecer cuando se crea un área de trabajo. No se puede cambiar en un área de trabajo existente.

La marca `hbi_workspace` controla la cantidad de [datos que Microsoft recopila para fines de diagnóstico](#microsoft-collected-data) y permite el [cifrado adicional en entornos administrados por Microsoft](../security/fundamentals/encryption-atrest.md). Además, permite las siguientes acciones:

* Inicia el cifrado del disco temporal local en el clúster de proceso de Azure Machine Learning, siempre que no haya creado ningún clúster anterior en esa suscripción. En caso contrario, debe generar una incidencia de soporte técnico para habilitar el cifrado del disco temporal de los clústeres de proceso. 
* Limpia el disco temporal local entre ejecuciones.
* Pasa de forma segura las credenciales de la cuenta de almacenamiento, el registro de contenedor y la cuenta SSH desde la capa de ejecución a los clústeres de proceso mediante el almacén de claves.
* Habilita el filtrado de IP para asegurarse de que los servicios externos que no sean AzureMachineLearningService no puedan llamar a los grupos de lotes subyacentes.

#### <a name="azure-blob-storage"></a>Azure Blob Storage

Azure Machine Learning almacena instantáneas, salidas y registros en la cuenta de Azure Blob Storage vinculada al área de trabajo de Azure Machine Learning y la suscripción. Todos los datos almacenados en Azure Blob Storage se cifran en reposo con claves administradas por Microsoft.

Para más información sobre el uso de claves propias para los datos almacenados en Azure Blob Storage, consulte [Cifrado de Azure Storage con claves administradas por el cliente en Azure Key Vault](../storage/common/storage-encryption-keys-portal.md).

Los datos de entrenamiento también se almacenan en Azure Blob Storage, de modo que están accesibles para los destinos de proceso de entrenamiento. Este almacenamiento no está administrado por Azure Machine Learning, sino que está montado en destinos de proceso como un sistema de archivos remoto.

Si necesita __girar o revocar__ la clave, puede hacerlo en cualquier momento. Al rotar una clave, la cuenta de almacenamiento comenzará a usar la nueva clave (versión más reciente) para cifrar los datos en reposo. Al revocar (deshabilitar) una clave, la cuenta de almacenamiento se encarga de las solicitudes con error. Normalmente, se tarda una hora para que la rotación o la revocación surtan efecto.

Para más información sobre cómo regenerar una clave de acceso, vea el artículo [Regeneración de las claves de la cuenta de almacenamiento](how-to-change-storage-access-key.md).

#### <a name="azure-cosmos-db"></a>Azure Cosmos DB

Azure Machine Learning almacena métricas y metadatos en una instancia de Azure Cosmos DB. Esta instancia está asociada a una suscripción de Microsoft administrada por Azure Machine Learning. Todos los datos almacenados en Azure Cosmos DB se cifran en reposo con claves administradas por Microsoft.

Si quiere usar sus propias claves (administradas por el cliente) para cifrar su instancia de Azure Cosmos DB, puede crear una instancia de Cosmos DB dedicada para usarla con el área de trabajo. Recomendamos este método si quiere almacenar los datos, como la información del historial de ejecución, fuera de la instancia de Cosmos DB multiinquilino hospedada en la suscripción de Microsoft. 

Para habilitar el aprovisionamiento de una instancia de Cosmos DB en su suscripción con claves administradas por el cliente, realice estas acciones:

* Si aún no lo ha hecho, registre los proveedores de recursos de Microsoft.MachineLearning y Microsoft.DocumentDB en su suscripción.

* Use los parámetros que se indican más abajo al crear el área de trabajo de Azure Machine Learning. Ambos parámetros son obligatorios y se admiten en SDK, CLI, API REST y plantillas de Resource Manager.

    * `resource_cmk_uri`: Este parámetro es el URI de recurso completo de la clave administrada por el cliente en el almacén de claves e incluye la [información de versión de la clave](../key-vault/about-keys-secrets-and-certificates.md#objects-identifiers-and-versioning). 

    * `cmk_keyvault`: Este parámetro es el identificador de recurso del almacén de claves de su suscripción. Este almacén de claves debe estar en la misma región y suscripción que usará para el área de trabajo de Azure Machine Learning. 
    
        > [!NOTE]
        > Esta instancia del almacén de claves puede ser diferente a la creada por Azure Machine Learning al aprovisionar el área de trabajo. Si quiere usar la misma instancia del almacén de claves para el área de trabajo, pase el mismo almacén de claves al aprovisionar el área de trabajo mediante el [parámetro key_vault](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py#&preserve-view=truecreate-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--exist-ok-false--show-output-true-). 

Esta instancia de Cosmos DB se crea en un grupo de recursos administrados por Microsoft en su suscripción, junto con los recursos que necesita. El grupo de recursos administrado se denomina con el formato `<AML Workspace Resource Group Name><GUID>`. Si el área de trabajo de Azure Machine Learning usa un punto de conexión privado, también se crea una red virtual para la instancia de Cosmos DB. Esta red virtual se usa para proteger la comunicación entre Cosmos DB y Azure Machine Learning.

> [!IMPORTANT]
> * No elimine el grupo de recursos que contiene esta instancia de Cosmos DB, ni ninguno de los recursos que se crean automáticamente en este grupo. Si necesita eliminar el grupo de recursos, la instancia de Cosmos DB, etc., primero debe eliminar el área de trabajo de Azure Machine Learning que la usa. El grupo de recursos, la instancia de Cosmos DB y los otros recursos que se crean automáticamente se eliminan cuando se elimina el área de trabajo asociada.
> * Las [__Unidades de solicitud__](../cosmos-db/request-units.md) predeterminadas para esta cuenta de Cosmos DB son __8000__. Este valor no se puede cambiar.
> * No puede proporcionar su propia red virtual para usar con la instancia de Cosmos DB que se crea. Tampoco puede modificar la red virtual. Por ejemplo, no puede cambiar el rango de direcciones IP que usa.

Si necesita __girar o revocar__ la clave, puede hacerlo en cualquier momento. Al rotar una clave, Cosmos DB comenzará a usar la nueva clave (versión más reciente) para cifrar los datos en reposo. Al revocar (deshabilitar) una clave, Cosmos DB se encarga de las solicitudes con error. Normalmente, se tarda una hora para que la rotación o la revocación surtan efecto.

Para más información sobre las claves administradas por el cliente con Cosmos DB, vea el artículo [Configuración de claves administradas por el cliente para la cuenta de Azure Cosmos DB](../cosmos-db/how-to-setup-cmk.md).

#### <a name="azure-container-registry"></a>Azure Container Registry

Todas las imágenes del contenedor en el registro (Azure Container Registry) se cifran en reposo. Azure cifra automáticamente una imagen antes de almacenarla y la descifra cuando Azure Machine Learning la extrae.

Para usar sus propias claves (administradas por el cliente) para cifrar su instancia de Azure Container Registry, debe crear un ACR y adjuntarlo al aprovisionar el área de trabajo, o bien cifrar la instancia predeterminada que se crea en el momento del aprovisionamiento del área de trabajo.

> [!IMPORTANT]
> Azure Machine Learning requiere que la cuenta de administrador esté habilitada en el Azure Container Registry. De forma predeterminada, esta opción se deshabilita al crear un registro de contenedor. Para información sobre cómo habilitar la cuenta de administrador, consulte [Cuenta de administrador](/azure/container-registry/container-registry-authentication#admin-account).
>
> Cuando se ha creado una instancia de Azure Container Registry para un área de trabajo, no la elimine. Si lo hace, se interrumpirá el área de trabajo de Azure Machine Learning.

Para ver ejemplos de cómo crear un área de trabajo con una instancia existente de Azure Container Registry, vea los siguientes artículos:

* [Creación de un área de trabajo para Azure Machine Learning con la CLI de Azure](how-to-manage-workspace-cli.md).
* [Uso de una plantilla de Azure Resource Manager para crear un área de trabajo para Azure Machine Learning](how-to-create-workspace-template.md).

#### <a name="azure-container-instance"></a>Azure Container Instances

Puede cifrar un recurso de Azure Container Instance (ACI) implementado mediante claves administradas por el cliente. La clave administrada por el cliente que se usa para ACI puede almacenarse en el almacén Azure Key Vault del área de trabajo. Para obtener información sobre cómo generar una clave, consulte [Cifrado de datos con una clave administrada por el cliente](../container-instances/container-instances-encrypt-data.md#generate-a-new-key).

Para usar la clave al implementar un modelo en una instancia de Azure Container Instances, cree una nueva configuración de implementación mediante `AciWebservice.deploy_configuration()`. Proporcione la información de clave con los parámetros siguientes:

* `cmk_vault_base_url`: La dirección URL del almacén de claves que contiene la clave.
* `cmk_key_name`: El nombre de la clave.
* `cmk_key_version`: La versión de la clave.

Para obtener más información sobre la creación y el uso de una configuración de implementación, vea los siguientes artículos:

* Referencia de [AciWebservice.deploy_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aci.aciwebservice?view=azure-ml-py#&preserve-view=truedeploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none-)
* [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md)
* [Implementación de un modelo en Azure Container Instances](how-to-deploy-azure-container-instance.md)

Para obtener más información sobre el uso de una clave administrada por el cliente con ACI, consulte [Cifrado de datos con una clave administrada por el cliente](../container-instances/container-instances-encrypt-data.md#encrypt-data-with-a-customer-managed-key).

#### <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Puede cifrar un recurso implementado de Azure Kubernetes Service mediante claves administradas por el cliente en cualquier momento. Para más información, consulte [Traiga sus propias claves (BYOK) con discos de Azure en Azure Kubernetes Service (AKS)](../aks/azure-disk-customer-managed-keys.md). 

Este proceso permite cifrar los datos y el disco del sistema operativo de las máquinas virtuales implementadas en el clúster de Kubernetes.

> [!IMPORTANT]
> Este proceso solo funciona con la versión 1.17 de AKS K8s o con versiones posteriores. Azure Machine Learning agregó compatibilidad con AKS 1.17 el 13 de enero de 2020.

#### <a name="machine-learning-compute"></a>Proceso de Machine Learning

El disco del sistema operativo de cada nodo de proceso almacenado en Azure Storage se cifra mediante claves administradas por Microsoft en las cuentas de almacenamiento de Azure Machine Learning. Este destino de proceso es efímero y, por lo general, los clústeres se reducen verticalmente cuando no hay ninguna ejecución en cola. La máquina virtual subyacente se desaprovisiona y el disco del sistema operativo se elimina. Azure Disk Encryption no se admite con el disco del sistema operativo.

Cada máquina virtual tiene también un disco local temporal para las operaciones del sistema operativo. Si quiere, puede usar el disco para almacenar temporalmente los datos de entrenamiento. El disco se cifra de forma predeterminada para las áreas de trabajo con el parámetro `hbi_workspace` establecido en `TRUE`. Este entorno solo dura el tiempo de la ejecución, y la compatibilidad con el cifrado se limita únicamente a las claves administradas por el sistema.

#### <a name="azure-databricks"></a>Azure Databricks

Azure Databricks se puede utilizar en canalizaciones de Azure Machine Learning. De forma predeterminada, el sistema de archivos que usa Azure Databricks se cifra mediante una clave administrada por Microsoft. Para configurar Azure Databricks para que utilice las claves administradas por el cliente, consulte [Configuración de claves administradas por el cliente en DBFS (raíz) predeterminado](/azure/databricks/security/customer-managed-keys-dbfs).

### <a name="encryption-in-transit"></a>Cifrado en tránsito

Azure Machine Learning usa TLS para proteger la comunicación interna entre varios microservicios de Azure Machine Learning. Todo el acceso de Azure Storage se realiza también a través de un canal seguro.

Para proteger las llamadas externas realizadas al punto de conexión de puntuación, Azure Machine Learning usa TLS. Para obtener más información, vea [Uso de TLS para proteger un servicio web mediante Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/how-to-secure-web-service).

### <a name="using-azure-key-vault"></a>Uso de Azure Key Vault

Azure Machine Learning usa la instancia de Azure Key Vault asociada al área de trabajo para almacenar credenciales de varios tipos:

* La cadena de conexión de cuenta de almacenamiento asociada
* Contraseñas para las instancias de Azure Container Repository
* Cadenas de conexión a almacenes de datos

Las contraseñas y las claves SSH para destinos de proceso, como Azure HDInsight y las máquinas virtuales, se almacenan en un almacén de claves independiente asociado a la suscripción de Microsoft. Azure Machine Learning no almacena las contraseñas ni las claves proporcionadas por los usuarios. En cambio, genera, autoriza y almacena sus propias claves SSH para conectarse a las máquinas virtuales y HDInsight y ejecutar los experimentos.

Cada área de trabajo lleva asociada una identidad administrada asignada por el sistema con el mismo nombre que el área de trabajo. Esta identidad administrada tiene acceso a todas las claves, secretos y certificados del almacén de claves.

## <a name="data-collection-and-handling"></a>Recopilación y tratamiento de los datos

### <a name="microsoft-collected-data"></a>Datos recopilados por Microsoft

Microsoft puede recopilar información de identificación no relacionada con el usuario, como los nombres de los recursos (por ejemplo, el nombre del conjunto de datos o el nombre del experimento de aprendizaje automático) o las variables de entorno de trabajo con fines de diagnóstico. Todos estos datos se almacenan mediante claves administradas por Microsoft en el almacenamiento hospedado en suscripciones propiedad de Microsoft, y siguen los [estándares de tratamiento de los datos y la directiva de privacidad estándar de Microsoft](https://privacy.microsoft.com/privacystatement).

Microsoft también recomienda no almacenar información confidencial (como secretos de clave de cuenta) en variables de entorno. Nosotros no encargamos de registrar, cifrar y almacenar las variables de entorno. Del mismo modo, al asignar nombres [run_id](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run%28class%29?view=azure-ml-py&preserve-view=true), evite incluir información confidencial, como nombres de usuario o nombres de proyectos secretos. Esta información puede aparecer en los registros de telemetría a los que pueden acceder los ingenieros de Soporte técnico de Microsoft.

Puede dejar de participar en la recopilación de datos de diagnóstico si establece el parámetro `hbi_workspace` en `TRUE` mientras aprovisiona el área de trabajo. Esta función se admite cuando se usa el SDK de Python de AzureML, la CLI, las API REST o las plantillas de Azure Resource Manager.

### <a name="microsoft-generated-data"></a>Datos generados por Microsoft

Cuando usa servicios como el aprendizaje automático automatizado, Microsoft puede generar datos transitorios previamente procesados para entrenar varios modelos. Estos datos se guardan en un almacén de datos en el área de trabajo, lo que permite aplicar los controles de acceso y el cifrado de forma adecuada.

También puede que quiera cifrar la [información de diagnóstico registrada desde el punto de conexión implementado](how-to-enable-app-insights.md) en su instancia de Azure Application Insights.

## <a name="monitoring"></a>Supervisión

### <a name="metrics"></a>Métricas

Puede usar las métricas de Azure Monitor para ver y supervisar las métricas del área de trabajo de Azure Machine Learning. En [Azure Portal](https://portal.azure.com), seleccione el área de trabajo y, a continuación, **Métricas**:

[![Captura de pantalla que muestra las métricas de ejemplo de un área de trabajo](media/concept-enterprise-security/workspace-metrics.png)](media/concept-enterprise-security/workspace-metrics-expanded.png#lightbox)

Las métricas incluye información sobre ejecuciones, implementaciones y registros.

Para más información, consulte [Métricas en Azure Monitor](/azure/azure-monitor/platform/data-platform-metrics).

### <a name="activity-log"></a>Registro de actividades

Puede examinar el registro de actividad de un área de trabajo para ver diversas operaciones que se realizan en el área de trabajo. El registro incluye información básica como el nombre de la operación, el iniciador del evento y la marca de tiempo.

En esta captura de pantalla se muestra el registro de actividad de un área de trabajo:

[![Captura de pantalla que muestra el registro de actividad de un área de trabajo](media/concept-enterprise-security/workspace-activity-log.png)](media/concept-enterprise-security/workspace-activity-log-expanded.png#lightbox)

Los detalles de la solicitud de puntuación se almacenan en Application Insights. Application Insights se crea en la suscripción cuando se crea un área de trabajo. La información registrada incluye campos como estos:

* HTTPMethod
* UserAgent
* ComputeType
* RequestUrl
* StatusCode
* RequestId
* Duration

> [!IMPORTANT]
> Algunas acciones del área de trabajo de Azure Machine Learning no registran información en el registro de actividad. Por ejemplo, el inicio de una ejecución de entrenamiento y el registro de un modelo no se registran.
>
> Algunas de estas acciones aparecen en el área **Actividades** del área de trabajo, pero estas notificaciones no indican quién inició la actividad.

## <a name="data-flow-diagrams"></a>Diagramas de flujo de datos

### <a name="create-workspace"></a>Creación del espacio de trabajo

En el siguiente diagrama se muestra el flujo de trabajo de creación del área de trabajo.

* El usuario inicia sesión en Azure AD desde cualquiera de los clientes de Azure Machine Learning admitidos (la CLI de Azure, el SDK de Python o Azure Portal) y solicita el token de Azure Resource Manager adecuado.
* El usuario llama a Azure Resource Manager para crear el área de trabajo. 
* Azure Resource Manager se pone en contacto con el proveedor de recursos de Azure Machine Learning para aprovisionar el área de trabajo.

Durante la creación del área de trabajo, se crean recursos adicionales en la suscripción del usuario:

* Key Vault (para almacenar secretos)
* Una cuenta de Azure Storage (que incluye blob y recurso compartido de archivos)
* Azure Container Registry (para almacenar imágenes de Docker para inferencia, puntuación y experimentación)
* Application Insights (para almacenar datos de telemetría)

El usuario también puede aprovisionar otros destinos de proceso que estén asociados a un área de trabajo (como Azure Kubernetes Service o máquinas virtuales) según sea necesario.

[![Creación del flujo de trabajo del área de trabajo](media/concept-enterprise-security/create-workspace.png)](media/concept-enterprise-security/create-workspace.png#lightbox)

### <a name="save-source-code-training-scripts"></a>Almacenamiento del código fuente (scripts de entrenamiento)

En el siguiente diagrama se muestra el flujo de trabajo de la instantánea de código.

Existen directorios asociados con un área de trabajo de Azure Machine Learning (experimentos), que contienen el código fuente (scripts de entrenamiento). Estos scripts se almacenan en la máquina local y en la nube (en el almacenamiento de blobs de Azure de su suscripción). Las instantáneas de código se utilizan para la ejecución o inspección de auditorías históricas.

[![Flujo de trabajo de instantánea de código](media/concept-enterprise-security/code-snapshot.png)](media/concept-enterprise-security/code-snapshot.png#lightbox)

### <a name="training"></a>Cursos

En el siguiente diagrama se muestra el flujo de trabajo de entrenamiento.

* Para llamar a Azure Machine Learning se usa el identificador de la instantánea de código guardada en la sección anterior.
* Azure Machine Learning crea un identificador de ejecución (opcional) y un token de Machine Learning Service, que más adelante se usan en los destinos de proceso, como Proceso de Machine Learning o las máquinas virtuales, para comunicarse con este servicio.
* Puede elegir un destino de proceso administrado (como Proceso de Machine Learning) o un destino de proceso no administrado (como máquinas virtuales) para ejecutar los trabajos de entrenamiento. Estos son los flujos de datos de ambos escenarios:
   * Máquinas virtuales/HDInsight: se accede a ellos mediante las credenciales de SSH de un almacén de claves de la suscripción de Microsoft. Azure Machine Learning ejecuta código de administración en el destino de proceso que:

   1. Prepara el entorno. (Docker es una opción para máquinas virtuales y equipos locales. Consulte los siguientes pasos de Proceso de Machine Learning para entender cómo funciona la ejecución de experimentos en el contenedor de Docker).
   1. Descarga el código.
   1. Configura las variables de entorno y realiza las configuraciones.
   1. Ejecuta scripts de usuario (la instantánea de código mencionada en la sección anterior).

   * Proceso de Machine Learning, al que se accede mediante una identidad administrada por el área de trabajo.
Dado que Proceso de Machine Learning es un destino de proceso administrado (es decir, está administrado por Microsoft), se ejecuta en la suscripción de Microsoft.

   1. Si es necesario, pone en marcha la construcción de un Docker remoto.
   1. El código de administración se escribe en el recurso compartido de Azure Files del usuario.
   1. El contenedor se inicia con un comando inicial. Es decir, el código de administración que se describe en el paso anterior.

#### <a name="querying-runs-and-metrics"></a>Consulta de ejecuciones y métricas

En el diagrama de flujo siguiente, este paso se produce cuando el destino de proceso de entrenamiento escribe las métricas de ejecución de nuevo en Azure Machine Learning desde el almacenamiento de la base de datos de Cosmos DB. Los clientes pueden llamar a Azure Machine Learning. Machine Learning, a su vez, extrae las métricas de la base de datos de Cosmos DB y las devuelve al cliente.

[![Flujo de trabajo de entrenamiento](media/concept-enterprise-security/training-and-metrics.png)](media/concept-enterprise-security/training-and-metrics.png#lightbox)

### <a name="creating-web-services"></a>Creación de servicios web

En el siguiente diagrama se muestra el flujo de trabajo de inferencia. La inferencia, o puntuación de modelos, es la fase en la que se usa el modelo implementado para la predicción, muy frecuentemente con datos de producción.

Estos son los detalles:

* El usuario registra un modelo mediante un cliente, como el SDK de Azure Machine Learning.
* El usuario crea una imagen mediante un modelo, un archivo de puntuación y otras dependencias del modelo.
* La imagen de Docker se crea y se almacena en Azure Container Registry.
* El servicio web se implementa en el destino de proceso (Container Instances/AKS) mediante la imagen creada en el paso anterior.
* Los detalles de la solicitud de puntuación se almacenan en Application Insights, que se incluye en la suscripción del usuario.
* También se insertan datos de telemetría en la suscripción de Microsoft o Azure.

[![Flujo de trabajo de inferencia](media/concept-enterprise-security/inferencing.png)](media/concept-enterprise-security/inferencing.png#lightbox)

## <a name="audit-and-manage-compliance"></a>Auditoría y administración del cumplimiento

[Azure Policy](/azure/governance/policy) es una herramienta de gobierno que le permite asegurarse de que los recursos de Azure son compatibles con las directivas. Con Azure Machine Learning, puede asignar las siguientes directivas:

* **Clave administrada por el cliente**: audite o exija si las áreas de trabajo deben usar una clave administrada por el cliente.
* **Vínculo privado**: audite si las áreas de trabajo usan un punto de conexión privado para comunicarse con una red virtual.

Para obtener más información sobre Azure Policy, consulte la documentación de [Azure Policy](/azure/governance/policy/overview).

Para obtener más información sobre las directivas específicas de Azure Machine Learning, consulte [Auditoría y administración del cumplimiento con Azure Policy](how-to-integrate-azure-policy.md).

## <a name="next-steps"></a>Pasos siguientes

* [Protección de los servicios web Azure Machine Learning con TLS](how-to-secure-web-service.md)
* [Consumir un modelo de Machine Learning implementado como un servicio web](how-to-consume-web-service.md)
* [Uso de Azure Machine Learning con Azure Firewall](how-to-access-azureml-behind-firewall.md)
* [Uso de Azure Machine Learning con Azure Virtual Network](how-to-network-security-overview.md)
* [Procedimientos recomendados para compilar sistemas de recomendaciones](https://github.com/Microsoft/Recommenders)
* [Compilación de una API de recomendaciones en tiempo real en Azure](https://docs.microsoft.com/azure/architecture/reference-architectures/ai/real-time-recommendation)
