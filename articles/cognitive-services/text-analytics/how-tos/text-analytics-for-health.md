---
title: Uso de Text Analytics para el estado
titleSuffix: Azure Cognitive Services
description: Aprenda a extraer y etiquetar la información médica de textos clínicos no estructurados con Text Analytics para el estado.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 07/28/2020
ms.author: aahi
ms.openlocfilehash: dbd0699924268b38d69bc576a5886e8d31fa1208
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373477"
---
# <a name="how-to-use-text-analytics-for-health-preview"></a>Procedimientos: Uso de Text Analytics para el estado (versión preliminar)

> [!IMPORTANT] 
> Text Analytics para el estado es una funcionalidad de versión preliminar que se proporciona "tal cual" y "con todos los errores". Como tal, no se debe implementar **Text Analytics para el estado (versión preliminar) en ningún uso de producción.** Text Analytics para el estado no está previsto ni está disponible para su uso como dispositivo médico, soporte clínico, herramienta de diagnóstico u otra tecnología que se pretenda usar en el diagnóstico, la cura, la mitigación, el tratamiento o la prevención de la enfermedad u otras condiciones, y Microsoft no concede ninguna licencia o derecho para usar esta funcionalidad con este fin. Esta funcionalidad no está diseñada o prevista para implementarse como un sustituto del asesoramiento médico profesional o la opinión de la atención sanitaria, el diagnóstico, el tratamiento o la opinión clínica de un profesional sanitario y no debe usarse como tal. El cliente es el único responsable de cualquier uso de Text Analytics para el estado. Microsoft no garantiza que Text Analytics para el estado o los materiales que se proporcionen en relación con la capacidad sean suficientes para fines médicos o que de otro modo cumplan los requisitos sanitarios o médicos de cualquier persona. 


Text Analytics para el estado es un servicio de contenedor que extrae y etiqueta información médica pertinente de textos no estructurados, como notas del doctor, resúmenes de descarga, documentos clínicos y registros electrónicos de salud.  

## <a name="features"></a>Características

En la actualidad, el contenedor de Text Analytics para el estado realiza el reconocimiento de entidades con nombre (NER), la extracción de relaciones, la negación de entidades y la vinculación de entidades para el texto en inglés en su propio entorno de desarrollo que satisfaga los requisitos específicos de seguridad y gobernanza de datos.

#### <a name="named-entity-recognition"></a>[Reconocimiento de entidades con nombre](#tab/ner)

El reconocimiento de entidades con nombre detecta palabras y frases mencionadas en texto no estructurado que se pueden asociar a uno o varios tipos semánticos, como el diagnóstico, el nombre de los medicamentos, el síntoma, el signo o la edad.

> [!div class="mx-imgBorder"]
> ![Health NER](../media/ta-for-health/health-named-entity-recognition.png)

#### <a name="relation-extraction"></a>[Extracción de relaciones](#tab/relation-extraction)

La extracción de relaciones identifica conexiones significativas entre los conceptos mencionados en el texto. Por ejemplo, una relación "tiempo de condición" se encuentra asociando un nombre de condición con una hora. 

> [!div class="mx-imgBorder"]
> ![Health RE](../media/ta-for-health/health-relation-extraction.png)


#### <a name="entity-linking"></a>[Entity Linking](#tab/entity-linking)

La vinculación de entidad anula las distintas entidades al asociar las entidades con nombre mencionadas en el texto a los conceptos que se encuentran en una base de datos de conceptos predefinida. Por ejemplo, el sistema de lenguaje unificado de modelado (UMLS).

> [!div class="mx-imgBorder"]
> ![Health EL](../media/ta-for-health/health-entity-linking.png)

Text Analytics para el estado admite la vinculación a los vocabularios biomédicos y de salud que se encuentran en el origen de la información del metadiccionario de sinónimos del Sistema unificado de lenguaje médico ([UMLS](https://www.nlm.nih.gov/research/umls/sourcereleasedocs/index.html)).

#### <a name="negation-detection"></a>[Detección de negación](#tab/negation-detection) 

El significado del contenido médico se ve muy afectado por los modificadores como, por ejemplo, la negación, que puede tener una implicación crítica si se ha diagnosticado erróneamente. Text Analytics para el estado admite la detección de negación para las distintas entidades mencionadas en el texto. 

> [!div class="mx-imgBorder"]
> ![Health NEG](../media/ta-for-health/health-negation.png)

---

Consulte las [categorías de entidad](../named-entity-types.md?tabs=health) devueltas por Text Analytics para el estado a fin de obtener una lista completa de las entidades admitidas.

## <a name="supported-languages"></a>Idiomas compatibles

Text Analytics para el estado solo admite documentos en inglés.

## <a name="request-access-to-the-container-registry"></a>Solicitud de acceso al registro de contenedor

Rellene y envíe el [formulario de solicitud de contenedores de Cognitive Services](https://aka.ms/cognitivegate) para solicitar acceso al contenedor. Actualmente, no se le facturará el uso de Text Analytics para el estado. 

[!INCLUDE [Request access to the container registry](../../../../includes/cognitive-services-containers-request-access-only.md)]

[!INCLUDE [Authenticate to the container registry](../../../../includes/cognitive-services-containers-access-registry.md)]

## <a name="install-the-container"></a>Instalación del contenedor

Hay varias maneras de instalar y ejecutar el contenedor. 

- Utilice [Azure Portal](text-analytics-how-to-install-containers.md?tabs=healthcare) para crear un recurso de Text Analytics y Docker para obtener su contenedor.
- Usar los siguientes scripts de PowerShell y la [CLI de Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) para automatizar la configuración del contenedor de implementación de recursos.

### <a name="install-the-container-using-azure-web-app-for-containers"></a>Instalación del contenedor con el Web App for Containers de Azure

La [Web App for Containers](https://azure.microsoft.com/services/app-service/containers/) de Azure es un recurso de Azure dedicado a la ejecución de contenedores en la nube. Ofrece funcionalidades preparadas como el escalado automático, la compatibilidad con contenedores de Docker y Docker Compose, la compatibilidad con HTTPS y mucho más.

> [!NOTE]
> Con la aplicación Web de Azure, obtendrá automáticamente un dominio en forma de `<appservice_name>.azurewebsites.net`.

Ejecute este script de PowerShell con la CLI de Azure para crear una Web App for Containers mediante su suscripción y la imagen del contenedor a través de HTTPS. Espere a que se complete el script (aproximadamente 20 minutos) antes de enviar la primera solicitud.

```bash
$subscription_name = ""                    # THe name of the subscription you want you resource to be created on.
$resource_group_name = ""                  # The name of the resource group you want the AppServicePlan
                                           #    and AppSerivce to be attached to.
$resources_location = ""                   # This is the location you wish the AppServicePlan to be deployed to.
                                           #    You can use the "az account list-locations -o table" command to
                                           #    get the list of available locations and location code names.
$appservice_plan_name = ""                 # This is the AppServicePlan name you wish to have.
$appservice_name = ""                      # This is the AppService resource name you wish to have.
$TEXT_ANALYTICS_RESOURCE_API_KEY = ""      # This should be taken from the Text Analytics resource.
$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT = "" # This should be taken from the Text Analytics resource.
$DOCKER_REGISTRY_SERVER_PASSWORD = ""      # This will be provided separately.
$DOCKER_REGISTRY_SERVER_USERNAME = ""      # This will be provided separately.
$DOCKER_IMAGE_NAME = "containerpreview.azurecr.io/microsoft/cognitive-services-healthcare:latest"

az login
az account set -s $subscription_name
az appservice plan create -n $appservice_plan_name -g $resource_group_name --is-linux -l $resources_location --sku P3V2
az webapp create -g $resource_group_name -p $appservice_plan_name -n $appservice_name -i $DOCKER_IMAGE_NAME -s $DOCKER_REGISTRY_SERVER_USERNAME -w $DOCKER_REGISTRY_SERVER_PASSWORD
az webapp config appsettings set -g $resource_group_name -n $appservice_name --settings Eula=accept Billing=$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT ApiKey=$TEXT_ANALYTICS_RESOURCE_API_KEY

# Once deployment complete, the resource should be available at: https://<appservice_name>.azurewebsites.net
```

### <a name="install-the-container-using-azure-container-instance"></a>Instalación del contenedor con la instancia de contenedor de Azure

También puede usar una instancia de contenedor de Azure (ACI) para facilitar la implementación. ACI es un recurso que le permite ejecutar contenedores de Docker a petición en un entorno de Azure administrado y sin servidor. 

Consulte [Cómo usar Azure Container Instances](text-analytics-how-to-use-container-instances.md) para conocer los pasos necesarios para implementar un recurso ACI con Azure Portal. También puede usar el siguiente script de PowerShell con CLI de Azure, que creará un ACI en su suscripción con la imagen del contenedor.  Espere a que se complete el script (aproximadamente 20 minutos) antes de enviar la primera solicitud.

> [!NOTE] 
> Azure Container Instances no incluye compatibilidad con HTTPS para los dominios integrados. Si necesita HTTPS, tendrá que configurarlo manualmente, incluida la creación de un certificado y el registro de un dominio. Puede encontrar instrucciones para hacerlo con NGINX a continuación.

```bash
$subscription_name = ""                    # The name of the subscription you want you resource to be created on.
$resource_group_name = ""                  # The name of the resource group you want the AppServicePlan
                                           # and AppService to be attached to.
$resources_location = ""                   # This is the location you wish the web app to be deployed to.
                                           # You can use the "az account list-locations -o table" command to
                                           # Get the list of available locations and location code names.
$azure_container_instance_name = ""        # This is the AzureContainerInstance name you wish to have.
$TEXT_ANALYTICS_RESOURCE_API_KEY = ""      # This should be taken from the Text Analytics resource.
$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT = "" # This should be taken from the Text Analytics resource.
$DOCKER_REGISTRY_SERVER_PASSWORD = ""      # This will be provided separately.
$DOCKER_REGISTRY_SERVER_USERNAME = ""      # This will be provided separately.
$DNS_LABEL = ""                            # This is the DNS label name you wish your ACI will have
$DOCKER_REGISTRY_LOGIN_SERVER = "containerpreview.azurecr.io"
$DOCKER_IMAGE_NAME = "containerpreview.azurecr.io/microsoft/cognitive-services-healthcare:latest"

az login
az account set -s $subscription_name
az container create --resource-group $resource_group_name --name $azure_container_instance_name --image $DOCKER_IMAGE_NAME --cpu 5 --memory 12 --registry-login-server $DOCKER_REGISTRY_LOGIN_SERVER --registry-username $DOCKER_REGISTRY_SERVER_USERNAME --registry-password $DOCKER_REGISTRY_SERVER_PASSWORD --port 5000 --dns-name-label $DNS_LABEL --environment-variables Eula=accept Billing=$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT ApiKey=$TEXT_ANALYTICS_RESOURCE_API_KEY

# Once deployment complete, the resource should be available at: http://<unique_dns_label>.<resource_group_region>.azurecontainer.io:5000
```

### <a name="secure-aci-connectivity"></a>Conectividad ACI segura

De forma predeterminada, no se proporciona ninguna seguridad al usar ACI con el API del contenedor. Esto es porque la mayoría de las veces el contenedor se ejecuta como parte de un pod que está protegido desde fuera por un puente de red. Sin embargo, puede modificar un contenedor con un componente orientado hacia delante, manteniendo como privado el punto de conexión del contenedor. En los ejemplos siguientes se usa [NGINX](https://www.nginx.com) como puerta de enlace de entrada para admitir HTTPS/SSL y la autenticación del certificado de cliente.

> [!NOTE]
> NGINX es un servidor HTTP y proxy de alto rendimiento de código abierto. Un contenedor NGINX se puede usar para terminar una conexión TLS para un único contenedor. También se pueden producir soluciones de terminación TLS más complejas basadas en la entrada NGINX.

#### <a name="set-up-nginx-as-an-ingress-gateway"></a>Configuración de NGINX como puerta de enlace de entrada

NGINX usa [archivos de configuración](https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/) para habilitar características en runtime. Para habilitar la terminación TLS para otro servicio, debe especificar un certificado SSL para finalizar la conexión TLS y `proxy_pass` para especificar una dirección para el servicio. Se proporciona una muestra a continuación.


> [!NOTE]
> `ssl_certificate` espera que se especifique una ruta de acceso en el sistema de archivos local del contenedor de NGINX. La dirección especificada para `proxy_pass` debe estar disponible en la red del contenedor de NGINX.

El contenedor NGINX cargará todos los archivos del `_.conf_` que se montan bajo `/etc/nginx/conf.d/` en la ruta de acceso de configuración de HTTP.

```nginx
server {
  listen              80;
  return 301 https://$host$request_uri;
}
server {
  listen              443 ssl;
  # replace with .crt and .key paths
  ssl_certificate     /cert/Local.crt;
  ssl_certificate_key /cert/Local.key;

  location / {
    proxy_pass http://cognitive-service:5000;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP  $remote_addr;
  }
}
```

#### <a name="example-docker-compose-file"></a>Archivo de muestra de Docker Compose

En el ejemplo siguiente se muestra cómo se puede crear un archivo de [Docker Compose](https://docs.docker.com/compose/reference/overview) a fin de implementar los contenedores de NGINX y Text Analytics para el estado:

```yaml
version: "3.7"
services:
  cognitive-service:
    image: {IMAGE_ID}
    ports:
      - 5000:5000
    environment:
      - eula=accept
      - billing={ENDPOINT_URI}
      - apikey={API_KEY}
      - Logging:Disk:Format=json
    volumes:
        # replace with path to logs folder
      - <path-to-logs-folder>:/output
  nginx:
    image: nginx
    ports:
      - 443:443
    volumes:
        # replace with paths for certs and conf folders
      - <path-to-certs-folder>:/cert
      - <path-to-conf-folder>:/etc/nginx/conf.d/
```

Para iniciar este archivo de Docker Compose, ejecute el siguiente comando desde una consola en el nivel raíz del archivo:

```bash
docker-compose up
```

Para obtener más información, consulte la documentación de NGINX sobre la [Terminación SSL de NGINX](https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/).


## <a name="example-api-request"></a>Solicitud de API de ejemplo
El contenedor proporciona varias API de puntos de conexión de predicción de consultas basadas en REST.

Use la siguiente solicitud de dirección de URL como ejemplo para enviar una consulta al contenedor que ha implementado reemplazando la variable `serverURL` por el valor adecuado.

```bash
curl -X POST 'http://<serverURL>:5000/text/analytics/v3.0-preview.1/domains/health' --header 'Content-Type: application/json' --header 'accept: application/json' --data-binary @example.json

```

El siguiente JSON es un ejemplo de un archivo JSON adjunto al cuerpo POST de la solicitud de la API de Text Analytics para el estado:

```json
example.json

{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Patient reported itchy sores after swimming in the lake."
    },
    {
      "language": "en",
      "id": "2",
      "text": "Prescribed 50mg benadryl, taken twice daily."
    }
  ]
}
```

## <a name="api-response-body"></a>Cuerpo de respuesta del API

El siguiente JSON es un ejemplo del cuerpo de respuesta de la API de Text Analytics para el estado:

```json
{
    "documents": [
        {
            "id": "1",
            "entities": [
                {
                    "id": "0",
                    "offset": 17,
                    "length": 11,
                    "text": "itchy sores",
                    "type": "SYMPTOM_OR_SIGN",
                    "score": 0.97,
                    "isNegated": false
                }
            ]
        },
        {
            "id": "2",
            "entities": [
                {
                    "id": "0",
                    "offset": 11,
                    "length": 4,
                    "text": "50mg",
                    "type": "DOSAGE",
                    "score": 1.0,
                    "isNegated": false
                },
                {
                    "id": "1",
                    "offset": 16,
                    "length": 8,
                    "text": "benadryl",
                    "type": "MEDICATION_NAME",
                    "score": 0.99,
                    "isNegated": false,
                    "links": [
                        {
                            "dataSource": "UMLS",
                            "id": "C0700899"
                        },
                        {
                            "dataSource": "CHV",
                            "id": "0000044903"
                        },
                        {
                            "dataSource": "MMSL",
                            "id": "899"
                        },
                        {
                            "dataSource": "MSH",
                            "id": "D004155"
                        },
                        {
                            "dataSource": "NCI",
                            "id": "C300"
                        },
                        {
                            "dataSource": "NCI_DTP",
                            "id": "NSC0033299"
                        },
                        {
                            "dataSource": "PDQ",
                            "id": "CDR0000039163"
                        },
                        {
                            "dataSource": "PSY",
                            "id": "05760"
                        },
                        {
                            "dataSource": "RXNORM",
                            "id": "203457"
                        }
                    ]
                },
                {
                    "id": "2",
                    "offset": 32,
                    "length": 11,
                    "text": "twice daily",
                    "type": "FREQUENCY",
                    "score": 1.0,
                    "isNegated": false
                }
            ],
            "relations": [
                {
                    "relationType": "DOSAGE_OF_MEDICATION",
                    "score": 1.0,
                    "entities": [
                        {
                            "id": "0",
                            "role": "ATTRIBUTE"
                        },
                        {
                            "id": "1",
                            "role": "ENTITY"
                        }
                    ]
                },
                {
                    "relationType": "FREQUENCY_OF_MEDICATION",
                    "score": 1.0,
                    "entities": [
                        {
                            "id": "1",
                            "role": "ENTITY"
                        },
                        {
                            "id": "2",
                            "role": "ATTRIBUTE"
                        }
                    ]
                }
            ]
        }
    ],
    "errors": [],
    "modelVersion": "2020-05-08"
}
```

> [!NOTE] 
> Con la detección de negación, en algunos casos un solo término de negación puede abordar varios términos a la vez. La negación de una entidad reconocida se representa en la salida JSON mediante el valor booleano de la marca `isNegated`:

```json
{
  "id": "2",
  "offset": 90,
  "length": 10,
  "text": "chest pain",
  "type": "SYMPTOM_OR_SIGN",
  "score": 0.9972,
  "isNegated": true,
  "links": [
    {
      "dataSource": "UMLS",
      "id": "C0008031"
    },
    {
      "dataSource": "CHV",
      "id": "0000023593"
    },
    ...
```

## <a name="see-also"></a>Consulte también

* [Información general de Text Analytics](../overview.md)
* [Categorías de entidad con nombres](../named-entity-types.md)
* [Novedades](../whats-new.md)
