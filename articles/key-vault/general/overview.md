---
title: 'Introducción a Azure Key Vault: Azure Key Vault | Microsoft Docs'
description: Azure Key Vault es un almacén de secretos seguro, proporciona administración de secretos, claves y certificados, todo ello respaldado por módulos de seguridad de hardware.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: overview
ms.custom: mvc
ms.date: 01/07/2019
ms.author: mbaldwin
ms.openlocfilehash: 500648b3037a81b39f474538ec062ef922b6e2df
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89421652"
---
# <a name="about-azure-key-vault"></a>Acerca de Azure Key Vault

Azure Key Vault ayuda a solucionar los problemas siguientes:

- **Administración de secretos**: Azure Key Vault se puede utilizar para almacenar de forma segura y controlar de manera estricta el acceso a los tokens, contraseñas, certificados, claves de API y otros secretos.
- **Administración de claves**: también se puede usar Azure Key Vault como una solución de administración de claves. Azure Key Vault facilita la creación y control de las claves de cifrado utilizadas para cifrar los datos. 
- **Administración de certificados**: Azure Key Vault también es un servicio que le permite aprovisionar, administrar e implementar fácilmente certificados públicos y privados de la Seguridad de la capa de transporte y de la Capa de sockets seguros (TLS/SSL) para su uso con Azure y sus recursos internos conectados. 
- **Almacenamiento de secretos con el respaldo de los módulos de Microsoft Hardware Security**: los secretos, las claves y los certificados del almacén se cifran con una clave de software (nivel Estándar) o con una clave de HSM validada del nivel 2 de FIPS 140-2 ( nivel Premium) 

## <a name="why-use-azure-key-vault"></a>Motivos para usar Azure Key Vault

### <a name="centralize-application-secrets"></a>Centralización de secretos de aplicación

La centralización del almacenamiento de los secretos de aplicación le permite controlar su distribución. Key Vault reduce en gran medida las posibilidades de que se puedan filtrar por accidente los secretos. Cuando usan Key Vault, los desarrolladores de aplicaciones ya no tienen que almacenar la información de seguridad en su aplicación. No tener que almacenar información de seguridad en las aplicaciones elimina la necesidad de que esta información sea parte del código. Por ejemplo, puede que una aplicación necesite conectarse a una base de datos. En lugar de almacenar la cadena de conexión en el código de la aplicación, puede almacenarla de forma segura en Key Vault.

Las aplicaciones pueden acceder de manera protegida a la información que necesitan a través de URI. Estos URI permiten que las aplicaciones recuperen versiones específicas de un secreto. No hay ninguna necesidad de escribir código personalizado para proteger información secreta almacenada en Key Vault.

### <a name="securely-store-secrets-and-keys"></a>Almacenamiento seguro de secretos y claves

Los secretos y las claves se protegen mediante Azure, para lo que se usan algoritmos estándar del sector, longitudes de clave y módulos de seguridad de hardware (HSM). Los módulos de seguridad de hardware se validan mediante los Estándares federales de procesamiento de información (FIPS) 140-2 de nivel 2.

El acceso a un almacén de claves requiere una autorización y autenticación correctas antes de que un autor de llamada (usuario o aplicación) pueda obtener acceso. La autenticación establece la identidad del autor de la llamada, mientras que la autorización determina las operaciones que puede realizar.

La autenticación se realiza a través de Azure Active Directory. La autorización puede realizarse mediante el control de acceso basado en rol (RBAC) o la directiva de acceso de Key Vault. Se usa el control de acceso basado en rol cuando se trata de la administración de los almacenes y la directiva de acceso de Key Vault cuando se intenta acceder a los datos almacenados en un almacén.

Las instancias de Azure Key Vault se pueden proteger mediante software o con módulos de seguridad de hardware (HSM). En aquellos casos en los que necesita obtener seguridad adicional, puede importar o generar claves en módulos de seguridad de hardware (HSM) que no se salen nunca del límite de los HSM. Microsoft utiliza los módulos de seguridad de hardware de nCipher. Puede usar las herramientas de nCipher para mover una clave desde el módulo de seguridad de hardware a Azure Key Vault.

Además, Azure Key Vault está diseñado de modo que Microsoft no pueda ver ni extraer sus datos.

### <a name="monitor-access-and-use"></a>Supervisión del acceso y uso

Una vez que ha creado un par de almacenes de claves, querrá supervisar cómo y cuándo se accede a sus claves y secretos. Para supervisar la actividad, puede habilitar el registro de los almacenes. Puede configurar Azure Key Vault para:

- Archivar en una cuenta de almacenamiento.
- Transmitir a un centro de eventos.
- Envíe los registros a los registros de Azure Monitor.

Tiene el control sobre los registros y puede protegerlos de forma segura restringiendo el acceso y, además, puede eliminar los registros que ya no necesita.

### <a name="simplified-administration-of-application-secrets"></a>Administración simplificada de secretos de aplicación

Al almacenar datos importantes, debe realizar varios pasos. Se debe proteger la información de seguridad, esta debe seguir un ciclo de vida y debe tener alta disponibilidad. Azure Key Vault simplifica el proceso de cumplimiento de estos requisitos mediante:

- La eliminación de la necesidad de poseer conocimientos internos sobre los módulos de Hardware Security.
- El escalado vertical en un breve plazo de tiempo para adaptarse a los picos de uso de la organización.
- La replicación del contenido de una instancia de Key Vault de una región en una región secundaria. La replicación de datos garantiza la alta disponibilidad y elimina la necesidad de intervención del administrador para desencadenar la conmutación por error.
- La disposición de opciones estándar de administración de Azure a través del portal, la CLI de Azure y PowerShell.
- La automatización de determinadas tareas de los certificados que adquiere de entidades de certificación públicas, como la inscripción y la renovación.

Además, Azure Key Vault le permite segregar los secretos de aplicación. Las aplicaciones solo pueden acceder al almacén para el que tengan permiso, y se puede limitar el acceso para realizar únicamente operaciones específicas. Puede crear una instancia de Azure Key Vault por aplicación y restringir los secretos almacenados en un almacén de claves a una aplicación y a un equipo de desarrolladores concretos.

### <a name="integrate-with-other-azure-services"></a>Integración con otros servicios de Azure

Como almacén seguro de Azure, Key Vault se ha usado para simplificar escenarios como:
-  [Azure Disk Encryption](../../security/fundamentals/encryption-overview.md)
-  La funcionalidad de [cifrado siempre]( https://docs.microsoft.com/sql/relational-databases/security/encryption/always-encrypted-database-engine) y [Cifrado de datos transparente]( https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption?view=sql-server-ver15) del servidor de SQL Server y Azure SQL Database
- [Azure App Service]( https://docs.microsoft.com/azure/app-service/configure-ssl-certificate). 

Key Vault se puede integrar con cuentas de almacenamiento e instancias de Event Hubs y Log Analytics.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [claves, secretos y certificados](about-keys-secrets-certificates.md)
- [Inicio rápido: Creación de una instancia de Azure Key Vault mediante la CLI](../secrets/quick-create-cli.md)
- [Autenticación, solicitudes y respuestas](../general/authentication-requests-and-responses.md)
- [Guía del desarrollador de Key Vault](../general/developers-guide.md)
