---
title: Protección de servicios web con TLS
titleSuffix: Azure Machine Learning
description: Más información sobre cómo habilitar HTTPS con el fin de proteger un servicio Web que se implementa a través de Azure Machine Learning. Azure Machine Learning usa la versión 1.2 de TLS para proteger modelos implementados como servicios web.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: jmartens
ms.author: aashishb
author: aashishb
ms.date: 03/05/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: b497e0369ab31cc009c9524ffd63ff472013268e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90886017"
---
# <a name="use-tls-to-secure-a-web-service-through-azure-machine-learning"></a>Uso de TLS para proteger un servicio web con Azure Machine Learning


En este artículo se muestra cómo proteger un servicio web implementado con Azure Machine Learning.

Se usa [HTTPS](https://en.wikipedia.org/wiki/HTTPS) para restringir el acceso a los servicios web y proteger los datos que los clientes envían. HTTPS le ayuda a proteger las comunicaciones entre un cliente y un servicio web mediante el cifrado de las comunicaciones entre los dos. El cifrado usa [Seguridad de la capa de transporte (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security). TLS se conoce a veces todavía como *Capa de sockets seguros* (SSL), que fue su predecesor.

> [!TIP]
> El SDK de Azure Machine Learning usa el término "SSL" para las propiedades relacionadas con las comunicaciones seguras. Esto no significa que su servicio web no use *TLS*. SSL es simplemente un término más comúnmente reconocido.
>
> En concreto, los servicios web implementados a través de Azure Machine Learning solo admiten la versión 1.1 de TLS.

TLS y SSL dependen ambos de los *certificados digitales*, que ayudan con la comprobación de la identidad y el cifrado. Para obtener más información sobre cómo funcionan los certificados digitales, vea el tema de Wikipedia [Public key infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) (Infraestructura de clave pública).

> [!WARNING]
> Si no usa HTTPS para el servicio web, los datos que se envían hacia y desde el servicio podrían ser visibles para otros usuarios en Internet.
>
> HTTPS también permite al cliente comprobar la autenticidad del servidor al que se está conectando. Esta característica protege a los clientes contra ataques de tipo [Man in the middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).

Este es el proceso general para proteger un servicio web:

1. Obtenga un nombre de dominio.

2. Obtenga un certificado digital.

3. Implemente o actualice el servicio web con el protocolo TLS habilitado.

4. Actualice el DNS para que apunte al servicio web.

> [!IMPORTANT]
> Si va a implementar en Azure Kubernetes Service (AKS), puede adquirir su propio certificado o usar un certificado proporcionado por Microsoft. Si usa un certificado de Microsoft, no es necesario obtener un nombre de dominio ni un certificado TLS/SSL. Para obtener más información, consulte la sección [Habilitación de TLS e implementación](#enable) de este artículo.

Hay ligeras diferencias al proteger los servicios web a través de los [destinos de implementación](how-to-deploy-and-where.md).

## <a name="get-a-domain-name"></a>Obtención de un nombre de dominio

Si todavía no tiene un nombre de dominio, compre uno en un *registrador de nombres de dominio*. El proceso y los precios difieren entre los registradores. El registrador proporciona herramientas para administrar el nombre de dominio. Estas herramientas se usan para asignar un nombre de dominio completo (como www\.contoso.com ) a la dirección IP que hospeda el servicio web.

## <a name="get-a-tlsssl-certificate"></a>Obtención de un certificado TLS/SSL

Hay muchas maneras de obtener un certificado TLS/SSL (certificado digital). La más común es comprar uno en una *entidad de certificación* (CA). Independientemente de dónde obtenga el certificado, se necesitan estos archivos:

* Un **certificado**. El certificado debe contener la cadena de certificados completa y debe estar codificado en PEM.
* Una **clave**. La clave debe estar codificada en PEM.

Cuando solicite un certificado, debe proporcionar el nombre de dominio completo (FQDN) de la dirección que pretende usar para el servicio web (por ejemplo, www\.contoso.com). La dirección que aparece en el certificado y la que usan los clientes se comparan para comprobar la identidad del servicio web. Si esas direcciones no coinciden, el cliente obtiene un mensaje de error.

> [!TIP]
> Si la entidad de certificación no puede proporcionar el certificado y la clave como archivos codificados en PEM, puede usar una utilidad como [OpenSSL](https://www.openssl.org/) para cambiar el formato.

> [!WARNING]
> Use certificados *autofirmados* solo para desarrollo. No deben usarse en entornos de producción. Los certificados autofirmados pueden provocar problemas en las aplicaciones cliente. Para más información, consulte la documentación de las bibliotecas de red utilizadas en la aplicación cliente.

## <a name="enable-tls-and-deploy"></a><a id="enable"></a> Habilitación de TLS e implementación

Para implementar (o volver a implementar) el servicio con el protocolo TLS habilitado, establezca el parámetro *ssl_enabled* en "True", cuando proceda. Establezca el parámetro *ssl_certificate* en el valor del archivo *certificate*. Establezca *ssl_key* en el valor del archivo *key*.

### <a name="deploy-on-aks-and-field-programmable-gate-array-fpga"></a>Implementación en AKS y en una matriz de puertas programables (FPGA)

  > [!NOTE]
  > La información de esta sección también se aplica al implementar un servicio web seguro para el diseñador. Si no está familiarizado con el uso del SDK para Python, consulte [What is the Azure Machine Learning SDK for Python?](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true) (¿Qué es el SDK de Azure Machine Learning para Python?).

Cuando se implementa en AKS, puede crear un nuevo clúster de AKS o asociar uno existente. Para más información sobre cómo crear o adjuntar un clúster, consulte [Implementación de un modelo en un clúster de Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md).
  
-  Si crea un clúster, se usa **[AksCompute.provisionining_configuration()](/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#&preserve-view=trueprovisioning-configuration-agent-count-none--vm-size-none--ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--location-none--vnet-resourcegroup-name-none--vnet-name-none--subnet-name-none--service-cidr-none--dns-service-ip-none--docker-bridge-cidr-none--cluster-purpose-none--load-balancer-type-none--load-balancer-subnet-none-)** .
- Si se conecta a un clúster existente, se usa **[AksCompute.attach_configuration()](/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#&preserve-view=trueattach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)** . Ambos devuelven un objeto de configuración que tiene un método **enable_ssl**.

El método **enable_ssl** puede usar un certificado proporcionado por Microsoft o que adquiera.

  * Cuando se usa un certificado de Microsoft, debe usar el parámetro *leaf_domain_label*. Este parámetro genera el nombre DNS para el servicio. Por ejemplo, el valor "contoso" crea el nombre de dominio "contoso\<six-random-characters>.\<azureregion>.cloudapp.azure.com", donde \<azureregion> es la región que contiene el servicio. Si lo desea, puede usar el parámetro *overwrite_existing_domain* para sobrescribir el *leaf_domain_label* existente.

    Para implementar (o volver a implementar) el servicio con el protocolo TLS habilitado, establezca el parámetro *ssl_enabled* en "True", cuando proceda. Establezca el parámetro *ssl_certificate* en el valor del archivo *certificate*. Establezca *ssl_key* en el valor del archivo *key*.

    > [!IMPORTANT]
    > Si usa un certificado de Microsoft, no es necesario adquirir su propio nombre de dominio ni certificado.

    El ejemplo siguiente muestra cómo crear una configuración que habilita un certificado TLS/SSL de Microsoft:

    ```python
    from azureml.core.compute import AksCompute
    # Config used to create a new AKS cluster and enable TLS
    provisioning_config = AksCompute.provisioning_configuration()
    # Leaf domain label generates a name using the formula
    #  "<leaf-domain-label>######.<azure-region>.cloudapp.azure.net"
    #  where "######" is a random series of characters
    provisioning_config.enable_ssl(leaf_domain_label = "contoso")


    # Config used to attach an existing AKS cluster to your workspace and enable TLS
    attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                          cluster_name = cluster_name)
    # Leaf domain label generates a name using the formula
    #  "<leaf-domain-label>######.<azure-region>.cloudapp.azure.net"
    #  where "######" is a random series of characters
    attach_config.enable_ssl(leaf_domain_label = "contoso")
    ```

  * Cuando se usa *un certificado que compró*, se emplean los parámetros *ssl_cert_pem_file*, *ssl_key_pem_file* y *ssl_cname*. El ejemplo siguiente muestra cómo usar archivos *.pem* para crear una configuración que use un certificado TLS/SSL adquirido:

    ```python
    from azureml.core.compute import AksCompute
    # Config used to create a new AKS cluster and enable TLS
    provisioning_config = AksCompute.provisioning_configuration()
    provisioning_config.enable_ssl(ssl_cert_pem_file="cert.pem",
                                        ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    # Config used to attach an existing AKS cluster to your workspace and enable SSL
    attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                         cluster_name = cluster_name)
    attach_config.enable_ssl(ssl_cert_pem_file="cert.pem",
                                        ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    ```

Para más información acerca de *enable_ssl*, consulte [AksProvisioningConfiguration.enable_ssl()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.aksprovisioningconfiguration?view=azure-ml-py#&preserve-view=trueenable-ssl-ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--leaf-domain-label-none--overwrite-existing-domain-false-) y [AksAttachConfiguration.enable_ssl()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.aksattachconfiguration?view=azure-ml-py#&preserve-view=trueenable-ssl-ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--leaf-domain-label-none--overwrite-existing-domain-false-).

### <a name="deploy-on-azure-container-instances"></a>Implementación en Azure Container Instances

Al implementar en Azure Container Instances, proporcione valores para los parámetros relacionados con TLS, como el siguiente fragmento de código:

```python
from azureml.core.webservice import AciWebservice

aci_config = AciWebservice.deploy_configuration(
    ssl_enabled=True, ssl_cert_pem_file="cert.pem", ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
```

Para más información, consulte [AciWebservice.deploy_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aciwebservice#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none-).

## <a name="update-your-dns"></a>Actualización del DNS

Después, debe actualizar el DNS para que apunte al servicio web.

+ **Para Container Instances:**

  Use las herramientas del registrador de nombres de dominio para actualizar el registro de DNS de su nombre de dominio. El registro debe apuntar a la dirección IP del servicio.

  Puede haber una demora de varios minutos o de horas antes de que los clientes puedan resolver el nombre de dominio, en función del registrador y del período de vida (TTL) configurado para el nombre de dominio.

+ **Para AKS:** :

  > [!WARNING]
  > Si ha usado *leaf_domain_label* para crear el servicio mediante un certificado de Microsoft, no actualice manualmente el valor DNS para el clúster. El valor debe establecerse automáticamente.

  Actualice el DNS de la Dirección IP pública del clúster de AKS en la pestaña **Configuración** debajo de **Configuración** en el panel izquierdo. (Vea la siguiente imagen). La dirección IP pública es uno de los tipos de recurso creados bajo el grupo de recursos que contiene los nodos del agente de AKS y otros recursos de red.

  [![Azure Machine Learning: Protección de servicios web con TLS](./media/how-to-secure-web-service/aks-public-ip-address.png)](./media/how-to-secure-web-service/aks-public-ip-address-expanded.png)

## <a name="update-the-tlsssl-certificate"></a>Actualización del certificado TLS/SSL

Los certificados TLS/SSL expiran y se deben renovar. Normalmente esto sucede cada año. Utilice la información de las secciones siguientes para actualizar y renovar el certificado para los modelos implementados en Azure Kubernetes Service:

### <a name="update-a-microsoft-generated-certificate"></a>Actualización de un certificado generado por Microsoft

Si Microsoft generó el certificado originalmente (al utilizar *leaf_domain_label* para crear el servicio), use uno de los ejemplos siguientes para actualizar el certificado:

> [!IMPORTANT]
> * Si el certificado existente sigue siendo válido, use `renew=True` (SDK) o `--ssl-renew` (CLI) para forzar que la configuración lo renueve. Por ejemplo, si el certificado existente sigue siendo válido durante 10 días y no usa `renew=True`, es posible que el certificado no se renueve.
> * Cuando el servicio se implementó originalmente, `leaf_domain_label` se usa para crear un nombre DNS con el patrón `<leaf-domain-label>######.<azure-region>.cloudapp.azure.net`. Para conservar el nombre existente (incluidos los 6 dígitos generados originalmente), use el valor `leaf_domain_label` original. No incluya los 6 dígitos que se generaron.

**Uso del SDK**

```python
from azureml.core.compute import AksCompute
from azureml.core.compute.aks import AksUpdateConfiguration
from azureml.core.compute.aks import SslConfiguration

# Get the existing cluster
aks_target = AksCompute(ws, clustername)

# Update the existing certificate by referencing the leaf domain label
ssl_configuration = SslConfiguration(leaf_domain_label="myaks", overwrite_existing_domain=True, renew=True)
update_config = AksUpdateConfiguration(ssl_configuration)
aks_target.update(update_config)
```

**Uso de la CLI**

```azurecli
az ml computetarget update aks -g "myresourcegroup" -w "myresourceworkspace" -n "myaks" --ssl-leaf-domain-label "myaks" --ssl-overwrite-domain True --ssl-renew
```

Para más información, consulte los siguientes documentos de referencia:

* [SslConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.sslconfiguration?view=azure-ml-py&preserve-view=true)
* [AksUpdateConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.aksupdateconfiguration?view=azure-ml-py&preserve-view=true)

### <a name="update-custom-certificate"></a>Actualización del certificado personalizado

Si una entidad de certificación generó originalmente el certificado, siga estos pasos:

1. Use la documentación proporcionada por la entidad de certificación para renovar el certificado. Este proceso crea nuevos archivos de certificado.

1. Use el SDK o la CLI para actualizar el servicio con el nuevo certificado:

    **Uso del SDK**

    ```python
    from azureml.core.compute import AksCompute
    from azureml.core.compute.aks import AksUpdateConfiguration
    from azureml.core.compute.aks import SslConfiguration
    
    # Read the certificate file
    def get_content(file_name):
        with open(file_name, 'r') as f:
            return f.read()

    # Get the existing cluster
    aks_target = AksCompute(ws, clustername)
    
    # Update cluster with custom certificate
    ssl_configuration = SslConfiguration(cname="myaks", cert=get_content('cert.pem'), key=get_content('key.pem'))
    update_config = AksUpdateConfiguration(ssl_configuration)
    aks_target.update(update_config)
    ```

    **Uso de la CLI**

    ```azurecli
    az ml computetarget update aks -g "myresourcegroup" -w "myresourceworkspace" -n "myaks" --ssl-cname "myaks"--ssl-cert-file "cert.pem" --ssl-key-file "key.pem"
    ```

Para más información, consulte los siguientes documentos de referencia:

* [SslConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.sslconfiguration?view=azure-ml-py&preserve-view=true)
* [AksUpdateConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.aksupdateconfiguration?view=azure-ml-py&preserve-view=true)

## <a name="disable-tls"></a>Deshabilitación de TLS

Para deshabilitar TLS para un modelo implementado en Azure Kubernetes Service, cree un `SslConfiguration` con `status="Disabled"` y, a continuación, realice una actualización:

```python
from azureml.core.compute import AksCompute
from azureml.core.compute.aks import AksUpdateConfiguration
from azureml.core.compute.aks import SslConfiguration

# Get the existing cluster
aks_target = AksCompute(ws, clustername)

# Disable TLS
ssl_configuration = SslConfiguration(status="Disabled")
update_config = AksUpdateConfiguration(ssl_configuration)
aks_target.update(update_config)
```

## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre cómo:
+ [Consumir un modelo de Machine Learning implementado como un servicio web](how-to-consume-web-service.md)
+ [Información general sobre la privacidad y el aislamiento de la red virtual](how-to-network-security-overview.md)
