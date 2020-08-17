---
title: Azure Disk Encryption en una red aislada
description: En este artículo se ofrecen sugerencias para solucionar problemas de Microsoft Azure Disk Encryption para máquinas virtuales Linux.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 02/27/2020
ms.custom: seodec18
ms.openlocfilehash: f0ca50d137ef7e6378d2bf8d0ab03127d49a9a88
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373596"
---
# <a name="azure-disk-encryption-on-an-isolated-network"></a>Azure Disk Encryption en una red aislada

Cuando la conectividad está limitada por un firewall, la configuración del grupo de seguridad de red (NSG) o el requisito de proxy, podría perderse la capacidad de la extensión para realizar las tareas necesarias. Esto puede dar lugar a mensajes de estado similares a "El estado de extensión no está disponible en la máquina virtual".

## <a name="package-management"></a>Administración de paquetes

Azure Disk Encryption depende de una serie de componentes, que suelen instalarse como parte de la habilitación de ADE si no están ya presentes. Cuando estén detrás de un firewall o aislados de Internet, estos paquetes deben estar preinstalados o disponibles localmente.

Estos son los paquetes que se necesitan para cada distribución. Para una lista completa de las distribuciones y tipos de volumen compatibles, vea [Máquinas virtuales y sistemas operativos compatibles](disk-encryption-overview.md#supported-vms-and-operating-systems).

- **Ubuntu 14.04, 16.04, 18.04**: lsscsi, psmisc, at, cryptsetup-bin, python-parted, python-six, procps
- **CentOS 7.2-7.7**: lsscsi, psmisc, lvm2, uuid, at, patch, cryptsetup, cryptsetup-reencrypt, pyparted, procps-ng, util-linux
- **CentOS 6.8**: lsscsi, psmisc, lvm2, uuid, at, cryptsetup-reencrypt, pyparted, python-six
- **RedHat 7.2-7.7**: lsscsi, psmisc, lvm2, uuid, at, patch, cryptsetup, cryptsetup-reencrypt, procps-ng, util-linux
- **RedHat 6.8**: lsscsi, psmisc, lvm2, uuid, at, patch, cryptsetup-reencrypt
- **openSUSE 42.3, SLES 12-SP4, 12-SP3**: lsscsi, cryptsetup

En Red Hat, cuando se necesite un proxy, debe asegurarse de que el administrador de la suscripción y yum están configurados correctamente. Para obtener más información, consulte [cómo solucionar problemas de administrador de suscripción y yum](https://access.redhat.com/solutions/189533).  

Cuando los paquetes se instalan manualmente, también se deben actualizar manualmente a medida que se publiquen nuevas versiones.

## <a name="network-security-groups"></a>Grupos de seguridad de red
Parte de la configuración del grupo de seguridad de red que se aplica debe permitir todavía que el punto de conexión cumpla con los requisitos previos documentados de la configuración de red para el cifrado del disco.  Consulte [Azure Disk Encryption: Requisitos de red](disk-encryption-overview.md#networking-requirements)

## <a name="azure-disk-encryption-with-azure-ad-previous-version"></a>Azure Disk Encryption con Azure AD (versión anterior)

Si usa [Azure Disk Encryption con Azure AD (versión anterior)](disk-encryption-overview-aad.md), la [biblioteca de Azure Active Directory](../../active-directory/azuread-dev/active-directory-authentication-libraries.md) deberá instalarse manualmente para todas las distribuciones (además de los paquetes adecuados para la distribución, [como se indica anteriormente](#package-management)).

Cuando se está habilitando el cifrado con [credenciales de Azure AD](disk-encryption-linux-aad.md), la máquina virtual de destino debe permitir la conectividad a los puntos de conexión de Azure Active Directory y Key Vault. Los puntos de conexión de autenticación actuales de Azure Active Directory se mantienen en las secciones 56 y 59 de la documentación sobre los [intervalos de direcciones IP y direcciones URL de Office 365](/office365/enterprise/urls-and-ip-address-ranges). En la documentación sobre cómo [acceder a Azure Key Vault detrás de un firewall](../../key-vault/general/access-behind-firewall.md) se proporcionan instrucciones de Key Vault.

### <a name="azure-instance-metadata-service"></a>Servicio de metadatos de instancia de Azure 

La máquina virtual debe poder acceder al punto de conexión de [Azure Instance Metadata Service](instance-metadata-service.md) que utiliza una dirección IP no enrutable conocida (`169.254.169.254`) a la que solo se puede acceder desde la máquina virtual.  No se admiten las configuraciones de proxy que modifican el tráfico HTTP local hacia esta dirección (por ejemplo, agregando un encabezado X-Forwarded-For).

## <a name="next-steps"></a>Pasos siguientes

- Consulte más pasos en [Solución de problemas de Azure Disk Encryption](disk-encryption-troubleshooting.md).
- [Cifrado de datos en reposo de Azure](../../security/fundamentals/encryption-atrest.md)
