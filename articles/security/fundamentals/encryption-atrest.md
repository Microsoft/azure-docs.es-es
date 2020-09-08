---
title: Cifrado en reposo de datos de Microsoft Azure | Microsoft Docs
description: Este artículo proporciona información general sobre el cifrado en reposo de datos de Microsoft Azure, funcionalidades y consideraciones.
services: security
documentationcenter: na
author: msmbaldwin
manager: rkarlin
ms.assetid: 9dcb190e-e534-4787-bf82-8ce73bf47dba
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/13/2020
ms.author: mbaldwin
ms.openlocfilehash: 337deec076a25183a4dd866362c7dc55a485834d
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88224499"
---
# <a name="azure-data-encryption-at-rest"></a>Cifrado de datos en reposo de Azure

Microsoft Azure incluye herramientas para proteger los datos de acuerdo con las necesidades de seguridad y cumplimiento de su empresa. Este documento se centra en:

- Cómo se protegen los datos en reposo en Microsoft Azure.
- Describir los distintos componentes que forman parte de la implementación de protección de datos.
- Revisar las ventajas y desventajas de los distintos enfoques clave de protección de la administración.

El cifrado en reposo es un requisito de seguridad habitual. En Azure, las organizaciones pueden cifrar datos en reposo sin el riesgo o el costo de una solución de administración de claves personalizada. Las organizaciones tienen la opción de permitir a Azure administrar completamente el cifrado en reposo. Además, las organizaciones tienen varias opciones para administrar con detenimiento el cifrado y las claves de cifrado.

## <a name="what-is-encryption-at-rest"></a>¿Qué es el cifrado en reposo?

El cifrado en reposo es la codificación (cifrado) de datos cuando se conserva. Los diseños del cifrado en reposo de Azure utilizan cifrado asimétrico para cifrar o descifrar rápidamente grandes cantidades de datos según un modelo conceptual sencillo:

- Se usa una clave de cifrado simétrico para cifrar datos mientras se escriben en el almacenamiento.
- La misma clave de cifrado se utiliza para descifrar los datos tal y como se prepararon para su uso en la memoria.
- Se pueden particionar datos y se pueden usar claves diferentes para cada partición.
- Las claves deben almacenarse en una ubicación segura con el control de acceso basado en identidades y directivas de auditoría. Las claves de cifrado de datos se cifran a menudo con la clave de cifrado de claves en Azure Key Vault para limitar aún más el acceso.

En la práctica, los escenarios de control y administración de la clave, así como las convicciones de escala y disponibilidad, requieren construcciones adicionales. A continuación se describen los componentes y conceptos del cifrado en reposo de Microsoft Azure.

## <a name="the-purpose-of-encryption-at-rest"></a>Propósito del cifrado en reposo

El cifrado en reposo proporciona protección de datos para los datos almacenados (en reposo). Los ataques contra los datos en reposo incluyen intentos de obtener acceso físico al hardware en el que se almacenan los datos y, a continuación, poner en peligro los datos contenidos. En este tipo de ataque, la unidad del disco duro de un servidor puede utilizarse de forma incorrecta durante el mantenimiento permitiendo a un atacante eliminar la unidad de disco duro. Más adelante el atacante tendría que poner el disco duro en un equipo bajo su control para intentar obtener acceso a los datos.

El cifrado en reposo está diseñado para evitar que el atacante obtenga acceso a los datos sin cifrar asegurándose de que los datos se cifran en el disco. Si un atacante obtiene una unidad de disco duro con datos cifrados pero no las claves de cifrado, el atacante debe anular el cifrado para leer los datos. Este ataque es mucho más complejo y consume más recursos que el acceso a datos no cifrados en una unidad de disco duro. Por este motivo, el cifrado en reposo es muy recomendable y es un requisito de alta prioridad para muchas organizaciones.

También se requiere el cifrado en reposo por necesidad de la organización de los esfuerzos de cumplimiento y gobierno de datos. Las normas gubernamentales y del sector, como HIPAA, PCI y FedRAMP, diseñan las medidas de seguridad específicas a través de los requisitos de cifrado y la protección de datos. El cifrado en reposo es una medida obligatoria necesaria para el cumplimiento de algunas de esas regulaciones. Para más información sobre el enfoque de Microsoft en relación con la validación de FIPS 140-2, consulte [Publicación del estándar federal de procesamiento de información (FIPS) 140-2](https://docs.microsoft.com/microsoft-365/compliance/offering-fips-140-2).

Además de satisfacer los requisitos de cumplimiento y regulatorios, el cifrado en reposo proporciona protección de defensa en profundidad. Microsoft Azure proporciona una plataforma compatible para servicios, aplicaciones y datos. También proporciona servicios completos y seguridad física, control de acceso a los datos y auditoría. Sin embargo, es importante proporcionar medidas de seguridad "superpuestas" adicionales en caso de que se produzca un error en una de las otras medidas de seguridad y el cifrado en reposo proporciona dicha medida de seguridad.

Microsoft se compromete con el cifrado en las opciones de reposo a través de servicios en la nube y proporcionando a los clientes el control de las claves de cifrado y los registros de uso de claves. Además, Microsoft está trabajando para conseguir cifrar todos los datos en reposo de los clientes de forma predeterminada.

## <a name="azure-encryption-at-rest-components"></a>Componentes del cifrado en reposo de Azure

Como se describe anteriormente, el objetivo del cifrado en reposo es que los datos que se guardan en el disco se cifren con una clave de cifrado secreta. Para lograr la creación de una clave segura para el objetivo, se debe proporcionar almacenamiento, control del acceso y administración de las claves del cifrado. Aunque los detalles pueden variar, las implementaciones del cifrado en reposo de los servicios de Azure se pueden describir en los términos en los que se ilustran en el diagrama siguiente.

![Componentes](./media/encryption-atrest/azure-security-encryption-atrest-fig1.png)

### <a name="azure-key-vault"></a>Azure Key Vault

La ubicación del almacenamiento de las claves de cifrado y el control de acceso a esas claves es fundamental para un modelo de cifrado en reposo. Las claves deben ser muy seguras pero fáciles de administrar por parte de los usuarios especificados y deben estar disponibles para servicios concretos. Para los servicios de Azure, Azure Key Vault es la solución de almacenamiento de claves recomendada y proporciona una experiencia de administración habitual en los servicios. Las claves se almacenan y administran en los almacenes de claves, y se puede proporcionar acceso a Key Vault a los usuarios o servicios. Azure Key Vault admite la creación del cliente de claves o importación de claves de cliente para su uso en escenarios de clave de cifrado administrada por el cliente.

### <a name="azure-active-directory"></a>Azure Active Directory

Los permisos para usar las claves almacenadas en Azure Key Vault, además de para administrar o tener acceso a ellas para el cifrado en reposo y el descifrado, se pueden dar a las cuentas de Azure Active Directory.

### <a name="key-hierarchy"></a>Jerarquía de las claves

Se usa más de una clave de cifrado en una implementación de cifrado en reposo. Almacenar una clave de cifrado en Azure Key Vault garantiza el acceso de clave segura y la administración central de claves. Sin embargo, el acceso local del servicio a las claves de cifrado es más eficaz para el cifrado y descifrado masivo que la interacción con Key Vault para cada operación de datos, lo que permite un cifrado más seguro y un mejor rendimiento. Si se limita el uso de una clave de cifrado única, se reduce el riesgo de que la clave se encuentre en peligro y el costo de volver a cifrar cuando se debe reemplazar una clave. Los modelos de cifrado en reposo de Azure utilizan una jerarquía de claves formada por los siguientes tipos de claves a fin de abordar las necesidades que se indican a continuación:

- **Clave de cifrado de datos (DEK)** : Una clave simétrica AES256 usada para cifrar una partición o un bloque de datos.  Un único recurso puede tener muchas particiones y muchas claves de cifrado de datos. Cifrar cada bloque de datos con una clave diferente dificulta los ataques de análisis criptográficos. Se necesita acceso a las DEK por la instancia de proveedor o aplicación de recursos que cifra y descifra un bloque específico. Cuando se reemplaza una DEK con una nueva clave, solo se deben volver a cifrar los datos de su bloque asociado con una nueva clave.
- **Clave de cifrado de claves (KEK)** : una clave de cifrado utilizada para cifrar las claves de cifrado de datos. El uso de una clave de cifrado de claves que siempre permanece en Key Vault permite a las propias claves de cifrado de datos cifrarse y controlarse. La entidad que tiene acceso a la KEK puede ser diferente de la entidad que requiere la DEK. Una entidad puede adaptar el acceso a la DEK para limitar el acceso de cada DEK a una partición específica. Puesto que la KEK es necesaria para descrifrar la DEK, la KEK es de manera eficaz un punto único por el que se pueden eliminar de forma eficaz las DEK mediante la eliminación de la KEK.

Las claves de cifrado de datos cifradas con las claves de cifrado de claves se almacenan por separado y solo una entidad con acceso a la clave de cifrado de claves puede descifrar dichas claves de cifrado de datos. Se admiten diferentes modelos de almacenamiento de claves. Consulte los [modelos de cifrado de datos](encryption-models.md) para obtener más información.

## <a name="encryption-at-rest-in-microsoft-cloud-services"></a>Cifrado en reposo en servicios en la nube de Microsoft

Los Servicios en la nube de Microsoft se utilizan en los tres modelos de la nube: IaaS, PaaS, SaaS. A continuación encontrará ejemplos de cómo encajan en cada modelo:

- Servicios de software, que se conocen como software como servicio o SaaS, que tienen la aplicación proporcionada por la nube, como Office 365.
- Servicios de la plataforma de los que los clientes sacan provecho en la nube en sus aplicaciones, que usan la nube para tareas como almacenamiento, análisis y funcionalidad de bus de servicio.
- Servicios de infraestructura o infraestructura como servicio (IaaS) en los que el cliente implementa sistemas operativos y aplicaciones que se hospedan en la nube y, posiblemente, saca provecho de otros servicios en la nube.

### <a name="encryption-at-rest-for-saas-customers"></a>Cifrado en reposo para clientes de SaaS

Los clientes del software como servicio (SaaS) suelen tener el cifrado en reposo habilitado o disponible en cada servicio. Office 365 dispone de varias opciones para que los clientes comprueben o habiliten el cifrado en reposo. Para información sobre los servicios de Office 365, vea [Cifrado en Office 365](https://docs.microsoft.com/office365/securitycompliance/encryption).

### <a name="encryption-at-rest-for-paas-customers"></a>Cifrado en reposo para clientes PaaS

Los datos del cliente de la plataforma como servicio (PaaS) residen normalmente en un servicio de almacenamiento como Blob Storage, pero también pueden estar guardados en caché o almacenados en el entorno de ejecución de la aplicación, como una máquina virtual. Para ver las opciones disponibles del cifrado en reposo, examine la tabla siguiente para las plataformas de aplicación y almacenamiento que utiliza.

### <a name="encryption-at-rest-for-iaas-customers"></a>Cifrado en reposo para clientes de IaaS

Los clientes de la infraestructura como servicio (IaaS) pueden tener una variedad de servicios y aplicaciones en uso. Los servicios de IaaS pueden habilitar el cifrado en reposo en sus discos duros virtuales y máquinas virtuales que se hospedan en Azure mediante Azure Disk Encryption.

#### <a name="encrypted-storage"></a>Almacenamiento cifrado

Al igual que PaaS, las soluciones IaaS pueden sacar provecho de otros servicios de Azure que almacenan los datos que se cifran en reposo. En estos casos, puede habilitar el cifrado en el soporte del cifrado en reposo como proporciona cada servicio consumido de Azure. La siguiente tabla enumera las principales plataformas de aplicación, servicios y almacenamiento y el modelo de cifrado en reposo admitido.

#### <a name="encrypted-compute"></a>Compute de cifrado

Todos los discos, instantáneas e imágenes administrados están cifrados mediante Storage Service Encryption con una clave administrada por servicio. Una solución de cifrado en reposo más completa requiere que los datos no se conserven nunca en un formato no cifrado. Al procesar los datos en una máquina virtual, los datos se pueden conservar en el archivo de paginación de Windows o el archivo de intercambio de Linux, un archivo de volcado o en un registro de aplicaciones. Para asegurarse de que estos datos se cifran en reposo, las aplicaciones IaaS pueden usar Azure Disk Encryption en una máquina virtual de IaaS de Azure (Windows o Linux) y un disco virtual.

#### <a name="custom-encryption-at-rest"></a>Cifrado de datos en reposo personalizado

Se recomienda que siempre que sea posible, las aplicaciones IaaS saquen provecho de las opciones de cifrado en reposo y Azure Disk Encryption proporcionadas por los servicios de Azure consumidos. En algunos casos, como requisitos de cifrado irregulares o almacenamiento que no se basa en Azure, un desarrollador de una aplicación de IaaS podría necesitar implementar el cifrado en reposo. Las soluciones de los desarrolladores de IaaS podrían integrarse mejor con las expectativas de administración y del cliente de Azure mediante el aprovechamiento de ciertos componentes de Azure. En concreto, los desarrolladores deben usar el servicio Azure Key Vault para proporcionar almacenamiento seguro de claves, así como proporcionar a sus clientes opciones de administración de claves coherentes con la mayoría de los servicios de la plataforma de Azure. Además, las soluciones personalizadas deben usar identidades de servicio administradas de Azure para habilitar las cuentas de servicio para tener acceso a las claves de cifrado. Para encontrar información para desarrolladores sobre Azure Key Vault y las identidades de servicio administradas, consulte sus respectivos SDK.

## <a name="azure-resource-providers-encryption-model-support"></a>Compatibilidad con modelo de cifrado de proveedores de recursos de Azure

Los servicios de Microsoft Azure admitir uno o más modelos de cifrado en reposo. Para algunos servicios, sin embargo, podrían no ser aplicables uno o varios de los modelos de cifrado. Para los servicios que admiten escenarios clave administrados por el cliente, puede que estos solo admitan un subconjunto de los tipos de clave que admite Azure Key Vault para las claves de cifrado de claves. Además, los servicios pueden liberar compatibilidad para estos escenarios y tipos de claves en distintas programaciones. Esta sección describe el soporte del cifrado en reposo en el momento de redactar este artículo para cada uno de los servicios de almacenamiento de datos principales de Azure.

### <a name="azure-disk-encryption"></a>Azure Disk Encryption

Cualquier cliente mediante las características de la infraestructura de Azure como servicio (IaaS) puede lograr el cifrado en reposo para sus discos y máquinas virtuales de IaaS y discos mediante Azure Disk Encryption. Para más información sobre Azure Disk Encryption, vea la [documentación de Azure Disk Encryption](../azure-security-disk-encryption-overview.md).

#### <a name="azure-storage"></a>Almacenamiento de Azure

Todos los servicios de Azure Storage (Blob Storage, Queue Storage, Table Storage y Azure Files) admiten el cifrado en reposo en el lado servidor; algunos servicios admiten además el cifrado de las claves administradas por el cliente y el cifrado del lado cliente.

- Lado servidor: de forma predeterminada, todos los servicios de Azure Storage admiten el cifrado en el lado servidor mediante claves administradas por el servicio, lo que es transparente para la aplicación. Para más información, consulte [Cifrado del servicio Azure Storage para datos en reposo](../../storage/common/storage-service-encryption.md). Azure Blob Storage y Azure Files también admiten las claves RSA de 2048 bits administradas por el cliente en Azure Key Vault. Para más información, consulte [Cifrado del servicio Storage mediante claves administradas por el cliente en Azure Key Vault](../../storage/common/storage-encryption-keys-portal.md).
- Lado cliente: Azure Blobs, Tables y Queues admiten el cifrado en el lado cliente. Cuando se usa el cifrado del lado cliente, los clientes cifran los datos y los cargan como un blob cifrado. El cliente se encarga de la administración de claves. Consulte [Cifrado del lado de cliente y Azure Key Vault para Microsoft Azure Storage](../../storage/common/storage-client-side-encryption.md) para más información.

#### <a name="azure-sql-database"></a>Azure SQL Database

Azure SQL Database admite actualmente el cifrado en reposo para escenarios de cifrado en el lado cliente y en el lado servicio administrados por Microsoft.

Actualmente, la compatibilidad con el cifrado del servidor se proporciona a través de una característica de SQL denominada Cifrado de datos transparente. Una vez que un cliente de Azure SQL Database habilita la clave TDE, se crea y administra automáticamente para él. El cifrado en reposo puede habilitarse en los niveles de base de datos y servidor. Desde junio de 2017, el [cifrado de datos transparente (TDE)](https://msdn.microsoft.com/library/bb934049.aspx) se habilita de forma predeterminada en las bases de datos recién creadas. Azure SQL Database admite claves RSA de 2048 bits administradas por el cliente en Azure Key Vault. Para más información, consulte [Cifrado de datos transparente con BYOK (Bring Your Own Key) para Azure SQL Database y Azure SQL Data Warehouse](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-byok-azure-sql?view=azuresqldb-current).

Se admite el cifrado del lado cliente de los datos de Azure SQL Database a través de la característica [Always Encrypted](https://msdn.microsoft.com/library/mt163865.aspx). Always Encrypted utiliza una clave que el cliente crea y almacena. Los clientes pueden almacenar la clave maestra en el almacén de certificados de Windows, Azure Key Vault, o un módulo de seguridad de hardware. Al usar SQL Server Management Studio, los usuarios de SQL eligen qué clave les gustaría usar para cifrar cada columna.

## <a name="conclusion"></a>Conclusión

La protección de datos del cliente almacenados dentro de los servicios de Azure es de gran importancia para Microsoft. Todos los servicios hospedados en Azure se comprometen a proporcionar opciones de cifrado en reposo. Los servicios de Azure admiten claves administradas por el servicio, claves administradas por el cliente o cifrado del lado cliente. Los servicios de Azure están mejorando ampliamente la disponibilidad del cifrado en reposo y se planean nuevas opciones para la versión preliminar y la versión de disponibilidad general en los próximos meses.

## <a name="next-steps"></a>Pasos siguientes

- Consulte los [modelos de cifrado de datos](encryption-models.md) para obtener más información sobre las claves administradas por el servicio y las claves administradas por el cliente.
- Obtenga información sobre cómo usa Azure el [cifrado doble](double-encryption.md) para mitigar las amenazas que incluye el cifrado de datos.
