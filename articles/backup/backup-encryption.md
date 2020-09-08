---
title: Cifrado en Azure Backup
description: Obtenga información sobre cómo las características de cifrado de Azure Backup le ayudan a proteger los datos de copia de seguridad y a satisfacer las necesidades de seguridad de su negocio.
ms.topic: conceptual
ms.date: 08/04/2020
ms.custom: references_regions
ms.openlocfilehash: 71183e99522707737812096567d877df740c4bae
ms.sourcegitcommit: e2b36c60a53904ecf3b99b3f1d36be00fbde24fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88763650"
---
# <a name="encryption-in-azure-backup"></a>Cifrado en Azure Backup

Todos los datos de copia de seguridad se cifran automáticamente cuando se almacenan en la nube mediante el cifrado de Azure Storage, lo que le ayuda a cumplir los compromisos de seguridad y cumplimiento. Los datos en reposo se cifran mediante el cifrado AES de 256 bits, uno de los cifrados de bloques más seguros que hay disponibles y que es compatible con FIPS 140-2. Además del cifrado en reposo, todos los datos de copia de seguridad en tránsito se transfieren a través de HTTPS y permanecen siempre en la red troncal de Azure.

## <a name="levels-of-encryption-in-azure-backup"></a>Niveles de cifrado en Azure Backup

Azure Backup incluye el cifrado en dos niveles:

- **Cifrado de datos en el almacén de Recovery Services**
  - **Usando claves administradas por la plataforma**: De forma predeterminada, todos los datos se cifran mediante claves administradas por la plataforma. No es necesario realizar ninguna acción explícita de su parte para habilitar este cifrado. Se aplica a todas las cargas de trabajo de las que se realiza una copia de seguridad en el almacén de Recovery Services.
  - **Usando claves administradas por el cliente**: al realizar una copia de seguridad de Azure Virtual Machines, puede elegir cifrar los datos mediante las claves de cifrado que posee y administra. Azure Backup le permite usar las claves RSA almacenadas en Azure Key Vault para cifrar las copias de seguridad. La clave de cifrado utilizada para cifrar las copias de seguridad puede ser diferente de la que se usa para el origen. Los datos se protegen mediante una clave de cifrado de datos (DEK) basada en AES 256, que, a su vez, está protegida con las claves del usuario. Esto le proporciona un control total sobre los datos y las claves. Para permitir el cifrado, se requiere que conceda acceso al almacén de Recovery Services a la clave de cifrado en Azure Key Vault. Puede deshabilitar la clave o revocar el acceso siempre que sea necesario. Sin embargo, debe habilitar el cifrado con las claves antes de intentar proteger los elementos en el almacén. [Obtenga más información aquí](encryption-at-rest-with-cmk.md).
  - **Cifrado de nivel de infraestructura**: Además de cifrar los datos en el almacén de Recovery Services mediante claves administradas por el cliente, también puede elegir tener una capa adicional de cifrado configurada en la infraestructura de almacenamiento. La plataforma administra este cifrado de infraestructura. Junto con el cifrado en reposo mediante claves administradas por el cliente, permite el cifrado de dos niveles de los datos de copia de seguridad. El cifrado de infraestructura solo se puede configurar si primero elige usar sus propias claves para el cifrado en reposo. El cifrado de infraestructura usa claves administradas por la plataforma para cifrar datos.
- **Cifrado específico de la carga de trabajo de la que se está realizando una copia de seguridad**  
  - **Copia de seguridad de máquinas virtuales de Azure**: Azure Backup admite la copia de seguridad de máquinas virtuales con discos cifrados mediante [claves administradas por la plataforma](https://docs.microsoft.com/azure/virtual-machines/windows/disk-encryption#platform-managed-keys), así como [claves administradas por el cliente](https://docs.microsoft.com/azure/virtual-machines/windows/disk-encryption#customer-managed-keys) que posee y administra. Además, también puede hacer una copia de seguridad de las máquinas virtuales de Azure que tienen el sistema operativo o los discos de datos cifrados mediante [Azure Disk Encryption](backup-azure-vms-encryption.md#encryption-support-using-ade). ADE usa BitLocker para las VM Windows y DM-Crypt para las VM Linux a fin de realizar el cifrado de invitado.

>[!NOTE]
>El cifrado de infraestructura se encuentra actualmente en versión preliminar limitada y solo está disponible en las regiones Este de EE. UU., Oeste de EE. UU. 2, Centro y Sur de EE. UU. y US Gov Arizona. Si desea usar la característica en alguna de estas regiones, rellene [este formulario](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR0H3_nezt2RNkpBCUTbWEapUN0VHNEpJS0ZUWklUNVdJSTEzR0hIOVRMVC4u) y envíelo por correo electrónico a [AskAzureBackupTeam@microsoft.com](mailto:AskAzureBackupTeam@microsoft.com).

## <a name="next-steps"></a>Pasos siguientes

- [Cifrado de Azure Storage para datos en reposo](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)
- [Preguntas más frecuentes de Azure Backup](backup-azure-backup-faq.md#encryption) para cualquier pregunta que pueda tener sobre el cifrado
