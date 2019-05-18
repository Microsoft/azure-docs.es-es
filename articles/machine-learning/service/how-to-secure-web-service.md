---
title: Protección de servicios web con SSL
titleSuffix: Azure Machine Learning service
description: Obtenga información sobre cómo proteger un servicio web implementado con el servicio de Azure Machine Learning mediante la habilitación de HTTPS. HTTPS protege los datos enviados por los clientes que usan la seguridad de capa de transporte (TLS), un sustituto para las capas de sockets seguros (SSL). También se usa por los clientes para comprobar la identidad del servicio web.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: aashishb
author: aashishb
ms.date: 04/29/2019
ms.custom: seodec18
ms.openlocfilehash: 0487fe0331bfce3d0302fe997562cb124ac317d6
ms.sourcegitcommit: 179918af242d52664d3274370c6fdaec6c783eb6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65561079"
---
# <a name="use-ssl-to-secure-web-services-with-azure-machine-learning-service"></a>Uso de SSL para proteger servicios web con Azure Machine Learning Service

En este artículo, aprenderá a proteger un servicio web implementado con el servicio Azure Machine Learning. Puede restringir el acceso a servicios web y proteger los datos enviados por los clientes que usan [protocolo seguro (HTTPS)](https://en.wikipedia.org/wiki/HTTPS).

HTTPS se utiliza para proteger las comunicaciones entre un cliente y el servicio web mediante el cifrado de las comunicaciones entre los dos. El cifrado se controla mediante [capa de transporte (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security). TLS es todavía conoce a veces como Secure Sockets Layer (SSL), que fue el predecesor TLS.

> [!TIP]
> El SDK de Azure Machine Learning usa el término 'SSL' para las propiedades relacionadas con la habilitación de las comunicaciones seguras. Esto no significa que no se utiliza TLS mediante el servicio web, simplemente que SSL es el término más reconocible para muchos lectores.

TLS y SSL dependen de ambas __certificados digitales__, que se usan para realizar la comprobación de identidad y cifrado. Para obtener más información sobre el trabajo de certificados digitales, vea la entrada de Wikipedia en [infraestructura de clave pública (PKI)](https://en.wikipedia.org/wiki/Public_key_infrastructure).

> [!Warning]
> Si no habilita y usar HTTPS para el servicio web, los datos enviados hacia y desde el servicio pueden ser visibles a otros usuarios en internet.
>
> HTTPS también permite al cliente comprobar la autenticidad del servidor que se está conectando. Esto protege a los clientes contra [man-in-the-middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) ataques.

El proceso general de protección de un servicio web nuevo o uno existente es como sigue:

1. Obtenga un nombre de dominio.

2. Obtenga un certificado digital.

3. Implemente o actualice el servicio web con la configuración de SSL habilitada.

4. Actualice el DNS para que apunte al servicio web.

> [!IMPORTANT]
> Si va a implementar en Azure Kubernetes Service (AKS), puede proporcionar su propio certificado o usar un certificado proporcionado por Microsoft. Si usa el certificado de proporcionadas por Microsoft, no es necesario obtener un nombre de dominio o un certificado SSL. Para obtener más información, consulte el [habilitar SSL e implementar](#enable) sección.

Hay ligeras diferencias al proteger los servicios web mediante los [destinos de implementación](how-to-deploy-and-where.md).

## <a name="get-a-domain-name"></a>Obtención de un nombre de dominio

Si todavía no tiene un nombre de dominio, puede comprar uno en un __registrador de nombres de dominio__. El proceso varía entre los distintos registradores, al igual que el costo. El registrador también proporciona herramientas para administrar el nombre de dominio. Estas herramientas se usan para asignar un nombre de dominio completo (por ejemplo, www\.contoso.com) a la dirección IP que hospeda el servicio web.

## <a name="get-an-ssl-certificate"></a>Obtener un certificado SSL

Hay muchas maneras de obtener un certificado SSL (certificado digital). La más común es comprar uno en una __entidad de certificación__ (CA). Independientemente de dónde se obtiene el certificado, se necesitan estos archivos:

* Un __certificado__. El certificado debe incluir la cadena de certificados completa y debe estar codificado en PEM.
* Una __clave__. La clave debe estar codificada en PEM.

Cuando solicite un certificado, debe proporcionar el nombre de dominio completo (FQDN) de la dirección que pretende usar para el servicio web. Por ejemplo, www\.contoso.com. La dirección que aparece en el certificado y la dirección que usan los clientes se comparan cuando se valida la identidad del servicio web. Si no coinciden, se generará un error.

> [!TIP]
> Si la entidad de certificación no puede proporcionar el certificado y la clave como archivos codificados en PEM, puede usar una utilidad como [OpenSSL](https://www.openssl.org/) para cambiar el formato.

> [!WARNING]
> Los certificados autofirmados solo se deben usar para desarrollo. No se deben usar en el entorno de producción. Los certificados autofirmados pueden provocar problemas en las aplicaciones cliente. Para más información, consulte la documentación de las bibliotecas de red utilizadas en la aplicación cliente.

## <a id="enable"></a> Habilitar SSL e implementar

Para implementar el servicio con SSL habilitado (o volver a implementar), establezca el `ssl_enabled` parámetro `True`, siempre que sea aplicable. Establezca el parámetro `ssl_certificate` en el valor del archivo __certificate__ y `ssl_key` en el valor del archivo __key__.

+ **Implementar en Azure Kubernetes Service (AKS) y FPGA**

  > [!NOTE]
  > La información de esta sección también se aplica al implementar un servicio web seguro para la interfaz visual. Si no está familiarizado con el uso del SDK de Python, consulte el [información general sobre el SDK de Python de Azure Machine Learning.](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py).

  Cuando se implementa en AKS, puede crear un nuevo clúster AKS o asociar uno existente. Crear un nuevo clúster usa [AksCompute.provisionining_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#provisioning-configuration-agent-count-none--vm-size-none--ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--location-none--vnet-resourcegroup-name-none--vnet-name-none--subnet-name-none--service-cidr-none--dns-service-ip-none--docker-bridge-cidr-none-) mientras que la asociación de un clúster existente utiliza [AksCompute.attach_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#attach-configuration-resource-group-none--cluster-name-none--resource-id-none-). Ambos devuelven un objeto de configuración que tiene un `enable_ssl` método.

  El `enable_ssl` método puede usar un certificado proporcionado por Microsoft o uno que proporcione.

  * Cuando se usa un certificado __proporcionadas por Microsoft__, debe usar el `leaf_domain_label` parámetro. Mediante este parámetro, se creará el servicio mediante un certificado proporcionado por Microsoft. El `leaf_domain_label` se usa para generar el nombre DNS para el servicio. Por ejemplo, un valor de `myservice` crea un nombre de dominio de `myservice<6-random-characters>.<azureregion>.cloudapp.azure.com`, donde `<azureregion>` es la región que contiene el servicio. Si lo desea, puede usar el `overwrite_existing_domain` parámetro para sobrescribir la etiqueta de dominio existente de hoja.

    Para implementar (o volver a implementar) el servicio con el protocolo SSL habilitado, establezca el parámetro `ssl_enabled` en `True`, cuando proceda. Establezca el parámetro `ssl_certificate` en el valor del archivo __certificate__ y `ssl_key` en el valor del archivo __key__.

    > [!IMPORTANT]
    > Cuando se usa un certificado proporcionado por Microsoft, no es necesario adquirir su propio certificado o nombre de dominio.

    El ejemplo siguiente muestra cómo crear configuraciones que permiten a un certificado SSL creado por Microsoft:

    ```python
    from azureml.core.compute import AksCompute
    # Config used to create a new AKS cluster and enable SSL
    provisioning_config = AksCompute.provisioning_configuration()
    provisioning_config.enable_ssl(leaf_domain_label = "myservice")
    # Config used to attach an existing AKS cluster to your workspace and enable SSL
    attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                          cluster_name = cluster_name)
    attach_config.enable_ssl(leaf_domain_label = "myservice")
    ```

  * Cuando se usa __un certificado que compró__, utilice el `ssl_cert_pem_file`, `ssl_key_pem_file`, y `ssl_cname` parámetros. En el ejemplo siguiente se muestra cómo crear configuraciones que usan un certificado SSL que se proporcionan mediante `.pem` archivos:

    ```python
    from azureml.core.compute import AksCompute
    # Config used to create a new AKS cluster and enable SSL
    provisioning_config = AksCompute.provisioning_configuration()
    provisioning_config.enable_ssl(ssl_cert_pem_file="cert.pem",
                                        ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    # Config used to attach an existing AKS cluster to your workspace and enable SSL
    attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                         cluster_name = cluster_name)
    attach_config.enable_ssl(ssl_cert_pem_file="cert.pem",
                                        ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    ```

  Para obtener más información sobre `enable_ssl`, consulte [AksProvisioningConfiguration.enable_ssl()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.aksprovisioningconfiguration?view=azure-ml-py#enable-ssl-ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--leaf-domain-label-none--overwrite-existing-domain-false-) y [AksAttachConfiguration.enable_ssl()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.aksattachconfiguration?view=azure-ml-py#enable-ssl-ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--leaf-domain-label-none--overwrite-existing-domain-false-).

+ **Implementación en Azure Container Instances (ACI)**

  Al implementar en ACI, proporcione valores para los parámetros relacionados con SSL, tal como se muestra en el fragmento de código:

    ```python
    from azureml.core.webservice import AciWebservice

    aci_config = AciWebservice.deploy_configuration(ssl_enabled=True, ssl_cert_pem_file="cert.pem", ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    ```

  Para obtener más información, consulte [AciWebservice.deploy_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aciwebservice?view=azure-ml-py#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none-).

## <a name="update-your-dns"></a>Actualización del DNS

Después, debe actualizar el DNS para que apunte al servicio web.

+ **Para ACI**:

  Use las herramientas que proporciona el registrador de nombres de dominio para actualizar el registro de DNS de su nombre de dominio. El registro debe apuntar a la dirección IP del servicio.

  En función del registrador y del período de vida (TTL) configurado para el nombre de dominio, pueden pasar de varios minutos a varias horas antes de que los clientes puedan resolver el nombre de dominio.

+ **Para AKS**:

  > [!WARNING]
  > Si ha usado el `leaf_domain_label` para crear el servicio con un certificado proporcionado por Microsoft, no actualice manualmente el valor DNS para el clúster. El valor debe establecerse automáticamente.

  Actualice el DNS en la pestaña "Configuración" de la "Dirección IP pública" del clúster de AKS, tal como se muestra en la imagen. Puede encontrar la dirección IP pública como uno de los tipos de recurso creados bajo el grupo de recursos que contiene los nodos del agente de AKS y otros recursos de red.

  ![Azure Machine Learning Service: protección de servicios web con SSL](./media/how-to-secure-web-service/aks-public-ip-address.png)



## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre cómo:
+ [Consumir un modelo de Machine Learning implementado como un servicio web](how-to-consume-web-service.md)
+ [Ejecución segura de experimentos e inferencias dentro de una instancia de Azure Virtual Network](how-to-enable-virtual-network.md)

