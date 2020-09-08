---
title: Seguridad y uso de directivas
description: Obtenga información sobre la seguridad y las directivas para máquinas virtuales en Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: security
ms.workload: infrastructure-services
ms.date: 11/27/2018
ms.author: cynthn
ms.topic: conceptual
ms.openlocfilehash: 1c2156975eb8d8dcb3580603dfbe1f5d1390b79a
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88852479"
---
# <a name="secure-and-use-policies-on-virtual-machines-in-azure"></a>Protección y uso de directivas en máquinas virtuales en Azure

Es importante proteger la máquina virtual (VM) para las aplicaciones que se ejecutan. Proteger las máquinas virtuales puede suponer que se incluyan uno o varios servicios y características de Azure que abarcan un acceso seguro a las máquinas virtuales y al almacenamiento seguro de los datos. Este artículo proporciona información que le permite proteger las máquinas virtuales y las aplicaciones.

## <a name="antimalware"></a>Antimalware

El panorama moderno de amenazas para los entornos de nube es dinámico, lo que aumenta la presión para mantener una protección eficaz a fin de satisfacer los requisitos de cumplimiento y seguridad. [Microsoft Antimalware para Azure](../security/fundamentals/antimalware.md) es una funcionalidad gratuita de protección en tiempo real que ayuda a identificar y eliminar virus, spyware y otro software malintencionado. Las alertas se pueden configurar para avisarle cuando software no deseado o malintencionado intenta instalarse o ejecutarse en una máquina virtual. No se admite en las máquinas virtuales que ejecutan Linux o Windows Server 2008.

## <a name="azure-security-center"></a>Azure Security Center

[Azure Security Center](../security-center/security-center-intro.md) ayuda a evita y a detectar las amenazas para las máquinas virtuales, además de a responder a ellas. Security Center proporciona funcionalidades de administración de directivas y supervisión de la seguridad integradas en las suscripciones de Azure, ayuda a detectar las amenazas que podrían pasar desapercibidas y funciona con un amplio ecosistema de soluciones de seguridad.

El acceso Just-In-Time de Security Center se puede aplicar en toda la implementación de la máquina virtual para bloquear el tráfico entrante a las máquinas virtuales de Azure, lo que reduce la exposición a ataques y, al mismo tiempo, proporciona acceso sencillo para conectarse a las máquinas virtuales cuando sea necesario. Cuando Just-In-Time está habilitado y un usuario solicita acceso a una máquina virtual, Security Center comprueba qué permisos tiene el usuario para la máquina virtual. Si tiene los permisos correctos, se aprueba la solicitud y Security Center configura automáticamente los grupos de seguridad de red (NSG) para permitir el tráfico entrante a los puertos seleccionados durante un período de tiempo limitado. Una vez transcurrido ese tiempo, Security Center restaura los NSG a su estado anterior. 

## <a name="encryption"></a>Cifrado

Se ofrecen dos métodos de cifrado para los discos administrados. Cifrado en el nivel de sistema operativo, que es Azure Disk Encryption, y cifrado en el nivel de plataforma, que es el cifrado del lado servidor.

### <a name="server-side-encryption"></a>Cifrado del servidor

Los discos administrados de Azure cifran automáticamente los datos de forma predeterminada cuando se guardan en la nube. El cifrado del lado servidor protege los datos y le ayuda a cumplir los compromisos de cumplimiento y seguridad de la organización. Los datos de los discos administrados de Azure se cifran de forma transparente mediante [cifrado AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) de 256 bits, uno de los cifrados de bloques más sólidos que hay disponibles, y son compatibles con FIPS 140-2.

El cifrado no afecta al rendimiento de los discos administrados. No hay ningún coste adicional por el cifrado.

Puede confiar en las claves administradas por la plataforma para el cifrado del disco administrado o puede administrar el cifrado con sus propias claves. Si opta por administrar el cifrado con sus propias claves, puede especificar una *clave administrada por el cliente* que se usará para cifrar y descifrar todos los datos de discos administrados. 

Para más información sobre el cifrado del lado servidor, consulte los artículos sobre [Windows](windows/disk-encryption.md) o [Linux](linux/disk-encryption.md).

### <a name="azure-disk-encryption"></a>Azure Disk Encryption

Para mejorar la seguridad y el cumplimiento en las [máquinas virtuales Windows](windows/disk-encryption-overview.md) y [Linux](linux/disk-encryption-overview.md), se pueden cifrar los discos virtuales en Azure. Los discos virtuales en VM Windows se cifran en reposo mediante BitLocker. Los discos virtuales en las máquinas virtuales de Linux se cifran en reposo mediante dm-crypt. 

El cifrado de los discos virtuales en Azure no conlleva ningún cargo. Las claves criptográficas se almacenan en Azure Key Vault con protección de software, o puede importar o generar las claves en módulos de seguridad de hardware (HSM) certificados conforme a las normas FIPS 140-2 de nivel 2. Las claves criptográficas se usan para cifrar y descifrar los discos virtuales conectados a la máquina virtual. Estas claves criptográficas se pueden controlar y se puede auditar su uso. Como las máquinas virtuales se encienden y se apagan, una entidad de servicio de Azure Active Directory proporciona un mecanismos seguro para la emisión de estas claves criptográficas.

## <a name="key-vault-and-ssh-keys"></a>Key Vault y claves de SSH

Los secretos y los certificados pueden ser modelados como recursos y ser proporcionados por [Key Vault](../key-vault/general/basic-concepts.md). Puede usar Azure PowerShell para crear almacenes de claves de [máquinas virtuales Windows](windows/key-vault-setup.md) y la CLI de Azure para las [máquinas virtuales Linux](linux/key-vault-setup.md). También puede crear claves para el cifrado.

Las directivas de acceso a los almacenes de claves conceden permisos a las claves, los secretos y los certificados por separado. Por ejemplo, se puede dar a un usuario acceso solo a las claves, pero darle permisos para los secretos. Sin embargo, los permisos para acceder a las claves, secretos o certificados se encuentran en el nivel del almacén. En otras palabras, la [directiva de acceso de un almacén de claves](../key-vault/general/secure-your-key-vault.md) no admite permisos de nivel de objeto.

Cuando se conecte a máquinas virtuales (VM), debe usar la criptografía de clave pública para proporcionar una forma más segura de iniciar sesión en ellas. Este proceso implica un intercambio de claves públicas y privadas mediante el comando de Secure Shell (SSH) para autenticarse, en lugar de un nombre de usuario y una contraseña. Las contraseñas son vulnerables a ataques por fuerza bruta, sobre todo en máquinas virtuales con acceso a Internet, como los servidores web. Con un par de claves de Secure Shell (SSH) puede crear una [máquina virtual Linux](linux/mac-create-ssh-keys.md) en Azure que use claves SSH para la autenticación, lo que elimina la necesidad de usar contraseñas para iniciar sesión. También puede usar claves de SSH para conectarse desde una [máquina virtual Windows](linux/ssh-from-windows.md) a una máquina virtual Linux.

## <a name="managed-identities-for-azure-resources"></a>Identidades administradas de recursos de Azure

Un desafío común al compilar aplicaciones en la nube consiste en el modo de administrar las credenciales del código para autenticar los servicios en la nube. Proteger las credenciales es una tarea esencial. Lo ideal sería que nunca aparecieran en las estaciones de trabajo de los desarrolladores y que no se controlaran en el código fuente. Azure Key Vault proporciona una manera segura de almacenar credenciales, secretos y otras claves pero el código tiene que autenticarse en Key Vault para recuperarlos. 

Las identidades administradas para recursos de Azure con las que cuenta Azure Active Directory (Azure AD) resuelven este problema. Esta característica proporciona a los servicios de Azure una identidad de sistema administrada automáticamente en Azure AD. Puede usar esta identidad para autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluido Key Vault, sin necesidad de credenciales en el código.  El código que se ejecuta en una máquina virtual puede solicitar un token en dos puntos de conexión que solo son accesibles desde la máquina virtual. Para obtener información detallada acerca de este servicio, revise la página de información general de [identidades administradas para los recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md).   

## <a name="policies"></a>Directivas

Las [directivas de Azure](../governance/policy/overview.md) pueden utilizarse para definir el comportamiento deseado de las [máquinas virtuales Windows](./windows/policy.md) y de las [máquinas virtuales Linux](./linux/policy.md) en su organización. Mediante las directivas, una organización puede aplicar varias convenciones y reglas en toda la empresa. La aplicación del comportamiento deseado puede ayudar a reducir el riesgo a la vez que se contribuye al éxito de la organización.

## <a name="role-based-access-control"></a>Control de acceso basado en rol

Mediante el [control de acceso basado en roles de Azure (RBAC de Azure)](../role-based-access-control/overview.md), puede repartir las tareas entre el equipo y conceder a los usuarios únicamente el nivel de acceso que necesitan en la máquina virtual para realizar su trabajo. En lugar de proporcionar a todos los empleados permisos no restringidos en la máquina virtual, puede permitir solo determinadas acciones. Puede configurar el control de acceso para la máquina virtual en [Azure Portal](../role-based-access-control/role-assignments-portal.md), usando la [CLI de Azure](https://docs.microsoft.com/cli/azure/role) o [Azure PowerShell](../role-based-access-control/role-assignments-powershell.md).


## <a name="next-steps"></a>Pasos siguientes
- Siga los pasos para supervisar la seguridad de la máquina virtual mediante Azure Security Center para [Linux](../security/fundamentals/overview.md) o [Windows](windows/tutorial-azure-security.md).

