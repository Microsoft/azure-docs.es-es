---
title: Protección de aplicaciones PaaS con Azure Storage | Microsoft Docs
description: Obtenga información sobre los procedimientos recomendados de seguridad de Azure Storage para proteger aplicaciones web y móviles PaaS.
services: security
documentationcenter: na
author: TomShinder
manager: barbkess
editor: ''
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/28/2018
ms.author: tomsh
ms.openlocfilehash: fa23637500755f43bb380a9f20cbe3acc7c3a394
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87925811"
---
# <a name="best-practices-for-securing-paas-web-and-mobile-applications-using-azure-storage"></a>Procedimientos recomendados para proteger aplicaciones web y móviles PaaS con Azure Storage
En este artículo se explican una serie de procedimientos recomendados de seguridad de Azure Storage para proteger las aplicaciones web y móviles PaaS (plataforma como servicio). Estos procedimientos recomendados proceden de nuestra experiencia con Azure y las experiencias de clientes como usted.

Azure permite implementar y usar el almacenamiento de formas inauditas con un entorno local. Gracias a Azure Storage, se pueden alcanzar altos niveles de escalabilidad y disponibilidad con relativamente poco esfuerzo. Azure Storage no solo constituye la base de Azure Virtual Machines con Windows y Linux, sino que también puede admitir aplicaciones distribuidas de gran tamaño.

Azure Storage proporciona los cuatro servicios siguientes: Blob storage, Table storage, Queue storage y File storage. Para obtener más información, consulte [Introducción a Microsoft Azure Storage](/azure/storage/common/storage-introduction).

La [Guía de seguridad de Azure Storage](/azure/storage/common/storage-security-guide) es una excelente fuente para obtener información detallada sobre Azure Storage y la seguridad. Este artículo de procedimientos recomendados aborda de forma resumida algunos de los conceptos de la guía de seguridad. Además, se ofrecen vínculos a dicha guía y otras fuentes de información.

En este artículo se explican los siguientes procedimientos recomendados:

- Firmas de acceso compartido (SAS)
- Control de acceso basado en roles (RBAC)
- Cifrado del lado cliente para datos de gran valor
- Cifrado del servicio Storage


## <a name="use-a-shared-access-signature-instead-of-a-storage-account-key"></a>Usar una firma de acceso compartido en lugar de una clave de cuenta de almacenamiento
El control de acceso es fundamental. Para ayudar a controlar el acceso a Azure Storage, Azure genera dos claves de cuenta de almacenamiento (SAK) de 512 bits al crear una cuenta de almacenamiento. El nivel de redundancia de claves permite evitar interrupciones del servicio durante la rotación de claves rutinaria. 

Las claves de acceso de almacenamiento son secretos de prioridad alta y solo deben estar accesibles a los responsables del control de acceso de almacenamiento. Si los usuarios equivocados consiguen acceder a estas claves, controlarían por completo el almacenamiento y podrían reemplazar, eliminar o agregar archivos en él. Por ejemplo, software malintencionado y otro tipo de contenido que puede poner en peligro a sus clientes u organización.

Seguirá necesitando una forma de proporcionar acceso a los objetos del almacenamiento. Para proporcionar un acceso más detallado, puede aprovechar la firma de acceso compartido (SAS). La SAS permite compartir objetos específicos en el almacenamiento durante intervalo de tiempo predefinido y con permisos concretos. Una firma de acceso compartido permite definir lo siguiente:

- El intervalo durante el que la SAS es válida, incluida la hora de inicio y la hora de caducidad.
- Los permisos concedidos por la SAS. Por ejemplo, una SAS de un blob podría conceder a un usuario permisos de lectura y escritura para el blob, pero no permisos de eliminación.
- Una dirección IP opcional o un intervalo de direcciones IP de las que Azure Storage aceptará la SAS. Por ejemplo, podría especificar un intervalo de direcciones IP que pertenezcan a su organización. Esto proporciona otra medida de seguridad para su SAS.
- El protocolo a través del cual Azure Storage aceptará la SAS. Puede usar este parámetro opcional para restringir el acceso a los clientes mediante HTTPS.

SAS permite compartir contenido de la forma que se quiera sin tener que proporcionar las claves de cuenta de almacenamiento. Utilizar siempre SAS en la aplicación es una forma segura de compartir los recursos de almacenamiento sin poner en peligro las claves de cuenta de almacenamiento.

Para obtener más información sobre la firma de acceso compartido, vea [Uso de firmas de acceso compartido (SAS)](/azure/storage/common/storage-dotnet-shared-access-signature-part-1). 

## <a name="use-role-based-access-control"></a>Uso del control de acceso basado en rol
Otra forma administrar el acceso es usando la funcionalidad [Control de acceso basado en roles de Azure](/azure/role-based-access-control/overview) (RBAC de Azure). Gracias a RBAC, podrá centrarse en proporcionar a los empleados los permisos exactos que necesiten, según la necesidad de información y los principios de seguridad de privilegios mínimos. Un número elevado de permisos puede provocar que la cuenta esté expuesta a los atacantes. Si se conceden muy pocos, los empleados no podrán realizar su trabajo de manera eficaz. RBAC ayuda a abordar este problema, ya que es posible realizar una administración avanzada del acceso para Azure. Esto es fundamental para las organizaciones que deseen aplicar directivas de seguridad para el acceso a los datos.

En Azure, puede usar roles integrados de Azure para asignar privilegios a los usuarios. Por ejemplo, use Colaborador de la cuenta de almacenamiento para los operadores en la nube que necesiten administrar cuentas de almacenamiento, y el rol Colaborador de la cuenta de almacenamiento clásica para administrar cuentas de almacenamiento clásicas. En el caso de los operadores en la nube que necesitan administrar máquinas virtuales, pero no la red virtual ni la cuenta de almacenamiento a la que están conectadas, puede agregarlos al rol Colaborador de la máquina virtual.

Puede que las organizaciones que no aplican el control de acceso de datos mediante funcionalidades como RBAC estén concediendo más privilegios de los necesarios a sus usuarios. Esto puede poner en peligro los datos si algunos usuarios acceden a datos que no les conciernen.

Para obtener más información sobre RBAC, consulte los siguientes artículos:

- [Administración del acceso mediante RBAC y Azure Portal](/azure/role-based-access-control/role-assignments-portal)
- [Roles integrados en los recursos de Azure](/azure/role-based-access-control/built-in-roles)
- [Guía de seguridad de Azure Storage](/azure/storage/common/storage-security-guide) 

## <a name="use-client-side-encryption-for-high-value-data"></a>Uso del cifrado del lado cliente para datos de gran valor
El cifrado del lado cliente permite cifrar datos en tránsito mediante programación antes de cargarlos en Azure Storage y descifrarlos de la misma manera al recuperarlos. De este modo, se cifran tanto los datos en tránsito como los que están en reposo. El cifrado del lado cliente el método más seguro para cifrar los datos, pero hay que realizar cambios mediante programación en la aplicación e implementar procesos de administración de claves.

El cifrado del lado cliente también permite controlar de forma exclusiva las claves de cifrado. Puede generar y administrar sus propias claves de cifrado. Usa una técnica de sobre que consiste en que la biblioteca cliente de Azure Storage genera una clave de cifrado de contenido (CEK) que se encapsula (se cifra) mediante la clave de cifrado de claves (KEK). La KEK se identifica mediante un identificador de clave y puede ser un par de clave asimétrico o una clave simétrica que puede administrarse de forma local o guardarse en [Azure Key Vault](/azure/key-vault/key-vault-overview).

El cifrado del lado cliente se integra en las bibliotecas de cliente de almacenamiento. de Java y .NET. Vea [Cifrado del lado de cliente y Azure Key Vault para Microsoft Azure Storage](/azure/storage/common/storage-client-side-encryption) para obtener información sobre cómo cifrar los datos en las aplicaciones cliente y cómo generar y administrar claves de cifrado propias.

## <a name="enable-storage-service-encryption-for-data-at-rest"></a>Habilitar Storage Service Encryption para datos en reposo
Cuando el [cifrado del servicio Storage](/azure/storage/common/storage-service-encryption) está habilitado en File Storage, los datos se cifran automáticamente con el cifrado AES-256. Microsoft controla todo el proceso de cifrado, descifrado y administración de claves. Esta característica está disponible en los tipos de redundancia LRS y GRS.

## <a name="next-steps"></a>Pasos siguientes

En este artículo se presenta una serie de procedimientos recomendados de seguridad de Azure Storage para proteger aplicaciones web y móviles PaaS. Para obtener más información sobre cómo proteger las implementaciones de PaaS, vea:

- [Protección de implementaciones de PaaS](paas-deployments.md)
- [Protección de aplicaciones web y móviles PaaS con Azure App Services](paas-applications-using-app-services.md)
- [Protección de bases de datos PaaS en Azure](paas-applications-using-sql.md)
