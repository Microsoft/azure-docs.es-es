---
title: 'Uso de Service Management API (Python): guía de características'
description: Vea cómo ejecutar tareas comunes de administración de servicios mediante programación con Python.
services: cloud-services
documentationcenter: python
author: tanmaygore
manager: vashan
editor: ''
ms.assetid: 61538ec0-1536-4a7e-ae89-95967fe35d73
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: python
ms.topic: article
ms.date: 05/30/2017
ms.author: tagore
ms.custom: devx-track-python
ms.openlocfilehash: 35d0f77d5f4dde9e156d22e64881ba674189d327
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87851518"
---
# <a name="use-service-management-from-python"></a>Uso de la administración de servicios de Python
En esta guía se muestra cómo ejecutar tareas comunes de administración de servicios mediante programación con Python. La clase **ServiceManagementService** del [SDK de Azure para Python](https://github.com/Azure/azure-sdk-for-python) permite acceder mediante programación a gran parte de la funcionalidad relacionada con la administración de servicios que se encuentra disponible en [Azure Portal][management-portal]. Puede utilizar esta funcionalidad para crear, actualizar y eliminar servicios en la nube, implementaciones, servicios de administración de datos y máquinas virtuales. Esta funcionalidad puede resultar útil para compilar aplicaciones que precisan de acceso mediante programación a la administración de servicios.

## <a name="what-is-service-management"></a><a name="WhatIs"> </a>¿Qué es la administración de servicios?
Azure Service Management API proporciona acceso mediante programación a gran parte de la funcionalidad de administración de servicios disponible en [Azure Portal][management-portal]. Azure SDK para Python le permite administrar los servicios en la nube y las cuentas de almacenamiento.

Para usar Service Management API, necesita [crear una cuenta de Azure](https://azure.microsoft.com/pricing/free-trial/).

## <a name="concepts"></a><a name="Concepts"> </a>Conceptos
El SDK de Azure para Python cuenta con [Service Management API][svc-mgmt-rest-api], que es una API REST. Todas las operaciones de la API se realizan mediante TLS y se autentican mutuamente con los certificados X.509 v3. Se puede obtener acceso al servicio de administración desde un servicio que se ejecute en Azure. También se puede acceder directamente a través de Internet desde cualquier aplicación que envíe una solicitud HTTPS y reciba una respuesta HTTPS.

## <a name="installation"></a><a name="Installation"> </a>Instalación
Todas las características descritas en este artículo están disponibles en el paquete `azure-servicemanagement-legacy`, que puede instalarse con PIP. Para obtener más información sobre la instalación (por ejemplo, si no está familiarizado con Python), consulte [Instalación de Python y el SDK](/azure/developer/python/azure-sdk-install).

## <a name="connect-to-service-management"></a><a name="Connect"> </a>Conexión a la administración de servicios
Para conectarse al punto de conexión de administración de servicios, necesita el identificador de suscripción de Azure y un certificado de administración válido. Puede obtener el identificador de la suscripción en [Azure Portal][management-portal].

> [!NOTE]
> Ahora puede usar certificados creados con OpenSSL cuando use Windows. Se requiere Python 2.7.4 o posterior. Se recomienda usar OpenSSL en lugar de .pfx, ya que la compatibilidad con certificados .pfx probablemente se eliminará en el futuro.
>
>

### <a name="management-certificates-on-windowsmaclinux-openssl"></a>Certificados de administración en Windows/Mac/Linux (OpenSSL)
Puede usar [OpenSSL](https://www.openssl.org/) para crear el certificado de administración. Necesita crear dos certificados, uno para el servidor (un archivo `.cer`) y otro para el cliente (un archivo `.pem`). Para crear el archivo `.pem` , ejecute este comando:

```console
openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem
```

Para crear el certificado `.cer` , ejecute este comando:

```console
openssl x509 -inform pem -in mycert.pem -outform der -out mycert.cer
```

Para obtener más información sobre los certificados de Azure, consulte [Introducción a los certificados para Azure Cloud Services](cloud-services-certs-create.md). Para obtener una descripción completa de los parámetros de OpenSSL, consulte la documentación en [https://www.openssl.org/docs/apps/openssl.html](https://www.openssl.org/docs/apps/openssl.html).

Después de crear estos archivos, cargue el archivo `.cer` en Azure. En [Azure Portal][management-portal], en la pestaña **Configuración**, seleccione **Cargar**. Apunte dónde guardó el archivo `.pem`.

Después de obtener el identificador de suscripción, cree un certificado, cargue el archivo `.cer` en Azure y conéctese al punto de conexión de administración de Azure. Para conectarse, transfiera el identificador de suscripción y la ruta de acceso del archivo `.pem` a **ServiceManagementService**.

```python
from azure import *
from azure.servicemanagement import *

subscription_id = '<your_subscription_id>'
certificate_path = '<path_to_.pem_certificate>'

sms = ServiceManagementService(subscription_id, certificate_path)
```

En el ejemplo anterior, `sms` es un objeto **ServiceManagementService** . La clase **ServiceManagementService** es la clase principal usada para administrar los servicios de Azure.

### <a name="management-certificates-on-windows-makecert"></a>Certificados de administración en Windows (MakeCert)
Puede crear un certificado de administración autofirmado en la máquina con `makecert.exe`. Abra un **símbolo del sistema de Visual Studio** como **administrador** y use el siguiente comando, donde debe reemplazar *AzureCertificate* por el nombre del certificado que quiera usar:

```console
makecert -sky exchange -r -n "CN=AzureCertificate" -pe -a sha1 -len 2048 -ss My "AzureCertificate.cer"
```

El comando crea el archivo `.cer` y lo instala en el almacén de certificados **Personal**. Para obtener más información, consulte [Introducción a los certificados para Azure Cloud Services](cloud-services-certs-create.md).

Después de crear el certificado, cargue el archivo `.cer` en Azure. En [Azure Portal][management-portal], en la pestaña **Configuración**, seleccione **Cargar**.

Después de obtener el identificador de suscripción, cree un certificado, cargue el archivo `.cer` en Azure y conéctese al punto de conexión de administración de Azure. Para conectarse, transfiera el identificador de suscripción y la ubicación del certificado en su almacén de certificados **Personal** a **ServiceManagementService** (de nuevo, reemplace *AzureCertificate* por el nombre del certificado).

```python
from azure import *
from azure.servicemanagement import *

subscription_id = '<your_subscription_id>'
certificate_path = 'CURRENT_USER\\my\\AzureCertificate'

sms = ServiceManagementService(subscription_id, certificate_path)
```

En el ejemplo anterior, `sms` es un objeto **ServiceManagementService** . La clase **ServiceManagementService** es la clase principal usada para administrar los servicios de Azure.

## <a name="list-available-locations"></a><a name="ListAvailableLocations"> </a>Enumeración de ubicaciones disponibles
Para enumerar las ubicaciones disponibles para los servicios de hospedaje, use el método **list\_locations**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

result = sms.list_locations()
for location in result:
    print(location.name)
```

Al crear un servicio en la nube o un servicio de almacenamiento, deberá proporcionar una ubicación válida. El método **list\_locations** siempre devuelve una lista actualizada de las ubicaciones disponibles actualmente. En el momento de la redacción de este documento, las ubicaciones disponibles son:

* Oeste de Europa
* Norte de Europa
* Sudeste de Asia
* Este de Asia
* Centro de EE. UU.
* Centro-Norte de EE. UU
* Centro-sur de EE. UU.
* Oeste de EE. UU.
* Este de EE. UU.
* Japón Oriental
* Japón Occidental
* Sur de Brasil
* Este de Australia
* Sudeste de Australia

## <a name="create-a-cloud-service"></a><a name="CreateCloudService"> </a>Creación de un servicio en la nube
Cuando se crea una aplicación y se ejecuta en Azure, el conjunto del código y la configuración se denomina [servicio en la nube][cloud service] de Azure. (En versiones anteriores de Azure, se conocía como *servicio hospedado*). Puede usar el método **create\_hosted\_service** para crear un nuevo servicio hospedado. Para crear el servicio, debe proporcionar un nombre de servicio hospedado (que debe ser exclusivo en Azure), una etiqueta (codificada automáticamente como base 64), una descripción y una ubicación.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

name = 'myhostedservice'
label = 'myhostedservice'
desc = 'my hosted service'
location = 'West US'

sms.create_hosted_service(name, label, desc, location)
```

Puede enumerar todos los servicios hospedados para la suscripción con el método **list\_hosted\_services**.

```python
result = sms.list_hosted_services()

for hosted_service in result:
    print('Service name: ' + hosted_service.service_name)
    print('Management URL: ' + hosted_service.url)
    print('Location: ' + hosted_service.hosted_service_properties.location)
    print('')
```

Para obtener información acerca de un servicio hospedado en particular, transfiera el nombre del servicio hospedado al método **get\_hosted\_service\_properties**.

```python
hosted_service = sms.get_hosted_service_properties('myhostedservice')

print('Service name: ' + hosted_service.service_name)
print('Management URL: ' + hosted_service.url)
print('Location: ' + hosted_service.hosted_service_properties.location)
```

Después de crear un servicio en la nube, implemente el código en el servicio con el método **create\_deployment**.

## <a name="delete-a-cloud-service"></a><a name="DeleteCloudService"> </a>Eliminación de un servicio en la nube
Puede eliminar un servicio en la nube si transfiere el nombre del servicio al método **delete\_hosted\_service**.

```python
sms.delete_hosted_service('myhostedservice')
```

Antes de eliminar un servicio, primero es necesario eliminar todas las implementaciones de dicho servicio. Para obtener más información, consulte [Eliminación de una implementación](#DeleteDeployment).

## <a name="delete-a-deployment"></a><a name="DeleteDeployment"> </a>Eliminación de una implementación
Para eliminar una implementación, use el método **delete\_deployment**. En el ejemplo siguiente se muestra cómo eliminar una implementación llamada `v1`:

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

sms.delete_deployment('myhostedservice', 'v1')
```

## <a name="create-a-storage-service"></a><a name="CreateStorageService"> </a>Creación de un servicio de almacenamiento
Un [servicio de almacenamiento](../storage/common/storage-create-storage-account.md) le proporciona acceso a [blobs](../storage/blobs/storage-python-how-to-use-blob-storage.md), [tablas](../cosmos-db/table-storage-how-to-use-python.md) y [colas](../storage/queues/storage-python-how-to-use-queue-storage.md) de Azure. Para crear un servicio de almacenamiento, se necesita un nombre para el servicio (entre 3 y 24 caracteres en minúsculas y únicos dentro de Azure). También necesita una descripción, una etiqueta (hasta 100 caracteres, codificada automáticamente con base64) y una ubicación. En el ejemplo siguiente se muestra cómo crear un servicio de almacenamiento mediante la definición de una ubicación:

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

name = 'mystorageaccount'
label = 'mystorageaccount'
location = 'West US'
desc = 'My storage account description.'

result = sms.create_storage_account(name, desc, label, location=location)

operation_result = sms.get_operation_status(result.request_id)
print('Operation status: ' + operation_result.status)
```

En el ejemplo anterior, el estado de la operación **create\_storage\_account** puede recuperarse transfiriendo el resultado devuelto por **create\_storage\_account** al método **get\_operation\_status**. 

Puede enumerar las cuentas de almacenamiento y sus propiedades con el método **list\_storage\_accounts**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

result = sms.list_storage_accounts()
for account in result:
    print('Service name: ' + account.service_name)
    print('Location: ' + account.storage_service_properties.location)
    print('')
```

## <a name="delete-a-storage-service"></a><a name="DeleteStorageService"> </a>Eliminación de un servicio de almacenamiento
Para eliminar un servicio de almacenamiento, transfiera el nombre del servicio de almacenamiento al método **delete\_storage\_account**. Con la eliminación del servicio de almacenamiento también se eliminan todos los datos almacenados en el servicio (blobs, tablas y colas).

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

sms.delete_storage_account('mystorageaccount')
```

## <a name="list-available-operating-systems"></a><a name="ListOperatingSystems"> </a>Enumeración de los sistemas operativos disponibles
Para enumerar los sistemas operativos que están disponibles para los servicios de hospedaje, use el método **list\_operating\_systems**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

result = sms.list_operating_systems()

for os in result:
    print('OS: ' + os.label)
    print('Family: ' + os.family_label)
    print('Active: ' + str(os.is_active))
```

Como alternativa, también puede usar el método **list\_operating\_system\_families**, que agrupa los sistemas operativos por familia.

```python
result = sms.list_operating_system_families()

for family in result:
    print('Family: ' + family.label)
    for os in family.operating_systems:
        if os.is_active:
            print('OS: ' + os.label)
            print('Version: ' + os.version)
    print('')
```

## <a name="create-an-operating-system-image"></a><a name="CreateVMImage"> </a>Creación de una imagen de sistema operativo
Para agregar una imagen del sistema operativo al repositorio de imágenes, use el método **add\_os\_image**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

name = 'mycentos'
label = 'mycentos'
os = 'Linux' # Linux or Windows
media_link = 'url_to_storage_blob_for_source_image_vhd'

result = sms.add_os_image(label, media_link, name, os)

operation_result = sms.get_operation_status(result.request_id)
print('Operation status: ' + operation_result.status)
```

Para enumerar las imágenes del sistema operativo disponibles, use el método **list\_os\_images**. Se incluyen todas las imágenes de la plataforma y las imágenes de usuario.

```python
result = sms.list_os_images()

for image in result:
    print('Name: ' + image.name)
    print('Label: ' + image.label)
    print('OS: ' + image.os)
    print('Category: ' + image.category)
    print('Description: ' + image.description)
    print('Location: ' + image.location)
    print('Media link: ' + image.media_link)
    print('')
```

## <a name="delete-an-operating-system-image"></a><a name="DeleteVMImage"> </a>Eliminación de una imagen de sistema operativo
Para eliminar una imagen de usuario, use el método **delete\_os\_image**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

result = sms.delete_os_image('mycentos')

operation_result = sms.get_operation_status(result.request_id)
print('Operation status: ' + operation_result.status)
```

## <a name="create-a-virtual-machine"></a><a name="CreateVM"> </a> Creación de una máquina virtual
Para crear una máquina virtual, primero debe crear un [servicio en la nube](#CreateCloudService). A continuación, cree la implementación de máquina virtual con el método **create\_virtual\_machine\_deployment**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

name = 'myvm'
location = 'West US'

#Set the location
sms.create_hosted_service(service_name=name,
    label=name,
    location=location)

# Name of an os image as returned by list_os_images
image_name = 'OpenLogic__OpenLogic-CentOS-62-20120531-en-us-30GB.vhd'

# Destination storage account container/blob where the VM disk
# will be created
media_link = 'url_to_target_storage_blob_for_vm_hd'

# Linux VM configuration, you can use WindowsConfigurationSet
# for a Windows VM instead
linux_config = LinuxConfigurationSet('myhostname', 'myuser', 'mypassword', True)

os_hd = OSVirtualHardDisk(image_name, media_link)

sms.create_virtual_machine_deployment(service_name=name,
    deployment_name=name,
    deployment_slot='production',
    label=name,
    role_name=name,
    system_config=linux_config,
    os_virtual_hard_disk=os_hd,
    role_size='Small')
```

## <a name="delete-a-virtual-machine"></a><a name="DeleteVM"> </a>Eliminación de una máquina virtual
Para eliminar una máquina virtual, primero debe eliminar la implementación con el método **delete\_deployment**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

sms.delete_deployment(service_name='myvm',
    deployment_name='myvm')
```

El servicio en la nube puede eliminarse entonces con el método **delete\_hosted\_service**.

```python
sms.delete_hosted_service(service_name='myvm')
```

## <a name="create-a-virtual-machine-from-a-captured-virtual-machine-image"></a>Creación de una máquina virtual a partir de una imagen de máquina virtual capturada
Para capturar una imagen de máquina virtual, primero llame al método **capture\_vm\_image**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

# replace the below three parameters with actual values
hosted_service_name = 'hs1'
deployment_name = 'dep1'
vm_name = 'vm1'

image_name = vm_name + 'image'
image = CaptureRoleAsVMImage    ('Specialized',
    image_name,
    image_name + 'label',
    image_name + 'description',
    'english',
    'mygroup')

result = sms.capture_vm_image(
        hosted_service_name,
        deployment_name,
        vm_name,
        image
    )
```

Para asegurarse de que se capturó correctamente la imagen, use la API **list\_vm\_images**. Asegúrese de que la imagen aparezca en los resultados.

```python
images = sms.list_vm_images()
```

Para crear finalmente la máquina virtual con la imagen capturada, use el método **create\_virtual\_machine\_deployment** como se ha descrito anteriormente, pero esta vez páselo en vm_image_name.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

name = 'myvm'
location = 'West US'

#Set the location
sms.create_hosted_service(service_name=name,
    label=name,
    location=location)

sms.create_virtual_machine_deployment(service_name=name,
    deployment_name=name,
    deployment_slot='production',
    label=name,
    role_name=name,
    system_config=linux_config,
    os_virtual_hard_disk=None,
    role_size='Small',
    vm_image_name = image_name)
```

Para obtener más información sobre cómo capturar una máquina virtual Linux en el modelo de implementación clásica, consulte [Captura de una máquina virtual Linux](../virtual-machines/linux/classic/capture-image-classic.md).

Para obtener más información sobre cómo capturar una máquina virtual Windows en el modelo de implementación clásica, consulte [Captura de una máquina virtual Windows](../virtual-machines/windows/classic/capture-image-classic.md).

## <a name="next-steps"></a><a name="What's Next"> </a>Pasos siguientes
Ahora que ha aprendido los conceptos básicos de la administración de servicios, puede tener acceso a la [documentación de referencia completa de la API para el SDK de Azure para Python](https://azure-sdk-for-python.readthedocs.org/) y realizar tareas complejas fácilmente para administrar la aplicación Python.

Para más información, vea el [Centro para desarrolladores de Python](https://azure.microsoft.com/develop/python/).

[What is service management?]: #WhatIs
[Concepts]: #Concepts
[Connect to service management]: #Connect
[List available locations]: #ListAvailableLocations
[Create a cloud service]: #CreateCloudService
[Delete a cloud service]: #DeleteCloudService
[Create a deployment]: #CreateDeployment
[Update a deployment]: #UpdateDeployment
[Move deployments between staging and production]: #MoveDeployments
[Delete a deployment]: #DeleteDeployment
[Create a storage service]: #CreateStorageService
[Delete a storage service]: #DeleteStorageService
[List available operating systems]: #ListOperatingSystems
[Create an operating system image]: #CreateVMImage
[Delete an operating system image]: #DeleteVMImage
[Create a virtual machine]: #CreateVM
[Delete a virtual machine]: #DeleteVM
[Next steps]: #NextSteps
[management-portal]: https://portal.azure.com/
[svc-mgmt-rest-api]: https://msdn.microsoft.com/library/windowsazure/ee460799.aspx


[cloud service]:/azure/cloud-services/
