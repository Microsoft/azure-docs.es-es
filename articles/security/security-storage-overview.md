---
title: Características de seguridad que se pueden usar con Azure Storage | Microsoft Docs
description: Este artículo ofrece una visión general de las principales características de seguridad de Azure que se pueden usar con Azure Storage.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/28/2019
ms.author: terrylan
ms.openlocfilehash: 74c24e05d974c4b3bb15a242185b645231f74580
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65792780"
---
# <a name="azure-storage-security-overview"></a>Información general sobre seguridad de Azure Storage

Igualmente, en este artículo se ofrece información general de las características de seguridad de Azure que se pueden usar con Azure Storage. Azure Storage es la solución de almacenamiento en la nube para las aplicaciones modernas que dependen de la durabilidad, la disponibilidad y la escalabilidad para satisfacer las necesidades de sus clientes. Azure Storage proporciona un conjunto completo de funcionalidades de seguridad. Puede:

* Proteger la cuenta de almacenamiento mediante el control de acceso basado en roles (RBAC) y Azure Active Directory.
* Proteger los datos en tránsito entre una aplicación y Azure mediante cifrados de cliente, HTTPS o SMB 3.0.
* Establecer una configuración para que los datos se cifren automáticamente cuando se escriben en Azure Storage mediante Storage Service Encryption.
* Establecer el cifrado de los discos de datos y del sistema operativo que usan las máquinas virtuales mediante Azure Disk Encryption.
* Conceder acceso delegado a los objetos de datos de Azure Storage mediante las firmas de acceso compartido (SAS).
* Usar análisis para seguir el método de autenticación que alguien esté usando cuando obtienen acceso a Storage.

Para obtener más información sobre la seguridad en Azure Storage, consulte la [guía de seguridad de Azure Storage](../storage/common/storage-security-guide.md). En esta guía se proporciona información detallada de las características de seguridad de Azure Storage. Estas características incluyen las claves de cuenta de almacenamiento, el cifrado de datos en tránsito y en reposo y el análisis de almacenamiento.

## <a name="role-based-access-control"></a>Control de acceso basado en rol

Puede proteger su cuenta de almacenamiento mediante el control de acceso basado en roles (RBAC). En organizaciones que quieren aplicar directivas de seguridad para el acceso a los datos, es imperativo restringir el acceso en función de los principios de seguridad [necesidad de saber](https://en.wikipedia.org/wiki/Need_to_know) y [mínimo privilegio](https://en.wikipedia.org/wiki/Principle_of_least_privilege). Estos derechos de acceso se conceden al asignar el rol RBAC adecuado a grupos y aplicaciones de un determinado ámbito. Puede aprovechar los [roles integrados de RBAC](../role-based-access-control/built-in-roles.md), tales como el de colaborador de la cuenta de almacenamiento, para asignar privilegios a los usuarios.

Más información:

* [Control de acceso basado en roles de Azure Active Directory](../role-based-access-control/role-assignments-portal.md)

## <a name="delegated-access-to-storage-objects"></a>Acceso delegado a objetos de almacenamiento

Una firma de acceso compartido ofrece acceso delegado a recursos en la cuenta de almacenamiento. Esto significa que puede conceder a un cliente permisos limitados para objetos en su cuenta de almacenamiento durante un período específico y con un conjunto determinado de permisos sin tener que compartir las claves de acceso a las cuentas.

La SAS es un URI que incluye en sus parámetros de consulta toda la información necesaria para el acceso autenticado a un recurso de almacenamiento. Para obtener acceso a los recursos de almacenamiento con SAS, el cliente solo tiene que proporcionar SAS al método o constructor adecuados.

Más información:

* [Firmas de acceso compartido, Parte 1: Descripción del modelo SAS](../storage/common/storage-dotnet-shared-access-signature-part-1.md)
* [Crear y usar una SAS con Blob Storage](../storage/common/storage-dotnet-shared-access-signature-part-1.md)

## <a name="encryption-in-transit"></a>Cifrado en tránsito

El cifrado en tránsito es un mecanismo para proteger datos cuando se transmiten a través de redes. Gracias a Azure Storage, puede proteger los datos mediante:

* [El cifrado de nivel de transporte](../storage/common/storage-security-guide.md#encryption-in-transit), como HTTPS, para transferir datos a o desde Azure Storage.
* [El cifrado en el cable](../storage/common/storage-security-guide.md#using-encryption-during-transit-with-azure-file-shares), como el cifrado SMB 3.0, para recursos compartidos de Azure File.
* [Cifrado de cliente](../storage/common/storage-security-guide.md#using-client-side-encryption-to-secure-data-that-you-send-to-storage), para cifrar los datos antes de transferirlos a Storage y descifrarlos una vez transferidos desde este servicio.

Más información acerca del cifrado de cliente:

* [Cifrado del lado cliente para Microsoft Azure Storage (vista previa)](https://blogs.msdn.microsoft.com/windowsazurestorage/2015/04/28/client-side-encryption-for-microsoft-azure-storage-preview/)
* [Serie de controles de seguridad en la nube: Cifrado de datos en tránsito](https://cloudblogs.microsoft.com/microsoftsecure/2015/08/10/cloud-security-controls-series-encrypting-data-in-transit/)

## <a name="encryption-at-rest"></a>Cifrado en reposo

Para muchas organizaciones, el [cifrado de los datos en reposo](https://cloudblogs.microsoft.com/microsoftsecure/2015/09/10/cloud-security-controls-series-encrypting-data-at-rest/) es un paso obligatorio en lo que respecta a la privacidad de los datos, el cumplimiento y la soberanía de los datos. Hay tres características de Azure que proporcionan el cifrado de datos que están en reposo:

* [Storage Service Encryption](../storage/common/storage-security-guide.md#encryption-at-rest) siempre está habilitado y cifra automáticamente los datos del servicio de almacenamiento al escribirlos en Azure Storage.
* [Cifrado de cliente](../storage/common/storage-security-guide.md#client-side-encryption), que también proporciona la característica de cifrado en reposo.
* [Azure Disk Encryption](../storage/common/storage-security-guide.md#using-azure-disk-encryption-to-encrypt-disks-used-by-your-virtual-machines), que le permite cifrar los discos de datos y del sistema operativo que usa una máquina virtual de IaaS.

Más información acerca del cifrado de servicio Storage

* [Azure Storage Service Encryption](https://azure.microsoft.com/services/storage/) está disponible para [Azure Blob Storage](https://azure.microsoft.com/services/storage/blobs/). Para más información sobre otros tipos de almacenamiento de Azure, consulte [Azure Files](https://azure.microsoft.com/services/storage/files/), [Table Storage](https://azure.microsoft.com/services/storage/tables/) y [Queue Storage](https://azure.microsoft.com/services/storage/queues/).
* [Cifrado del servicio Azure Storage para datos en reposo (versión preliminar)](../storage/common/storage-service-encryption.md)

## <a name="azure-disk-encryption"></a>Azure Disk Encryption

Azure Disk Encryption para máquinas virtuales le ayuda a abordar la seguridad de la organización y los requisitos de cumplimiento. Se encarga de cifrar los discos de las máquinas virtuales (incluidos los discos de arranque y de datos) mediante el uso de claves y directivas que puede controlar en [Azure Key Vault](https://azure.microsoft.com/services/key-vault/).

Cifrado de discos para máquinas virtuales funciona en sistemas operativos Linux y Windows. También utiliza Key Vault para ayudarle a proteger, administrar y auditar el uso de las claves de cifrado del disco. Todos los datos de los discos de máquinas virtuales se cifran en reposo mediante la tecnología de cifrado estándar del sector en las cuentas de Azure Storage. La solución Cifrado de discos para Windows se basa en la tecnología de [Cifrado de unidad BitLocker de Microsoft](https://technet.microsoft.com/library/cc732774.aspx), y la solución de Linux se basa en [dm-crypt](https://en.wikipedia.org/wiki/Dm-crypt).

Obtenga más información

* [Azure Disk Encryption for Windows and Linux Azure Virtual Machines (Azure Disk Encryption para máquinas virtuales IaaS de Linux y Windows)](https://gallery.technet.microsoft.com/Azure-Disk-Encryption-for-a0018eb0)

## <a name="firewalls-and-virtual-networks"></a>Firewalls y redes virtuales

El almacenamiento de Azure le permite habilitar las reglas de firewall para las cuentas de almacenamiento. Una vez habilitadas, se bloquearán las solicitudes entrantes para los datos, incluidas las solicitudes de otros servicios de Azure. Puede configurar excepciones para permitir el tráfico. Las reglas de firewall pueden habilitarse en las cuentas de almacenamiento existentes o durante el tiempo de creación.

Debe usar esta funcionalidad para limitar las cuentas de almacenamiento a un conjunto específico de redes permitidas.

Para obtener más información sobre los firewalls y redes virtuales de Azure Storage, consulte el artículo [Configuración de redes virtuales y firewalls de Azure Storage](../storage/common/storage-network-security.md).

## <a name="azure-data-box"></a>Azure Data Box

Los dispositivos Data Box, Data Box Disk y Data Box Heavy facilitan la transferencia de grandes cantidades de datos a Azure cuando la red no es una opción. Estos dispositivos de transferencia de datos sin conexión se envían entre su organización y el centro de datos Azure. Utilizan cifrado AES para proteger los datos en tránsito y se someten a un estricto proceso de saneamiento después de la carga para eliminar los datos del dispositivo.

Data Box Edge y Data Box Gateway son productos para la transferencia de datos en línea que actúan como puertas de enlace de almacenamiento en la red con el fin de administrar los datos entre su centro de datos y Azure. Data Box Edge, un dispositivo de red local, transfiere los datos a Azure, y al contrario, y utiliza el proceso perimetral con inteligencia artificial (IA) para procesar los datos. Data Box Gateway es un dispositivo virtual con funcionalidades de puerta de enlace de almacenamiento.

Más información:

* [Azure Data Box](https://azure.microsoft.com/services/storage/databox/)
* [Azure Data Box Edge](../databox-online/data-box-edge-overview.md)
* [Azure Data Box Gateway](..//databox-online/data-box-gateway-overview.md)

## <a name="advanced-threat-protection"></a>Protección contra amenazas avanzada

Azure Storage proporciona Advanced Threat Protection para ofrecer un nivel adicional de inteligencia de seguridad que detecte intentos poco habituales y potencialmente peligrosos de acceder o vulnerar la cuenta de almacenamiento. Advanced Threat Protection supervisa los registros de diagnóstico de Azure Storage para detectar solicitudes sospechosas de lectura, escritura o eliminación en Blob Storage.

Las alertas de Advanced Threat Protection se pueden ver en [Azure Security Center](https://azure.microsoft.com/services/security-center/). Azure Security Center proporciona detalles sobre cualquier actividad sospechosa detectada y recomienda acciones para investigar y solucionar la amenaza potencial.

Más información:

* [Introducción a Advanced Threat Protection de Azure Storage](../storage/common/storage-advanced-threat-protection.md)

## <a name="azure-key-vault"></a>Azure Key Vault

Azure Disk Encryption usa [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) para ayudarle a controlar y administrar las claves y los secretos de cifrado de discos en la suscripción de Key Vault. Gracias a ello, se garantiza que todos los datos de los discos de máquinas virtuales se cifren en reposo en Azure Storage. Debe usar Key Vault para auditar el uso de la directiva y las claves.

Obtenga más información

* [¿Qué es Azure Key Vault?](../key-vault/key-vault-overview.md)
