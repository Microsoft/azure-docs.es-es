---
title: Solución de problemas de instalación en Azure AD Connect | Microsoft Docs
description: En este tema se proporcionan los pasos para solucionar problemas relacionados con la instalación de Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 01/31/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 25314b4a306678dc877a95194907b3d73979e4f8
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89275868"
---
# <a name="troubleshoot-azure-ad-connect-install-issues"></a>Solución de problemas: Problemas de instalación en Azure AD Connect

## <a name="recommended-steps"></a>**Pasos recomendados**
Compruebe qué [tipo de instalación de Azure AD Connect](./how-to-connect-install-select-installation.md) es adecuada para usted. Si cumple los criterios de instalación rápida, se recomienda que elija esa opción. La instalación rápida proporciona un número mínimo de opciones para finalizar la instalación; por lo tanto, la probabilidad de que surjan problemas es menor. 

Sin embargo, si no cumple los criterios para la instalación rápida y debe realizar la instalación personalizada, a continuación se describen algunos procedimientos recomendados que puede seguir para evitar problemas comunes. Para simplificar, solo se mencionan algunas opciones:

* Asegúrese de que es administrador en el equipo en el que está instalando AAD Connect. Inicie sesión en la máquina con esas mismas credenciales de administrador.

* Permita que todas las opciones de la siguiente página se queden en sus valores predeterminados, a excepción de "Use an existing SQL Server" (Usar una instancia de SQL Server existente) si quiere usar una instancia de SQL Server existente. En [este artículo](./how-to-connect-install-custom.md) encontrará más información sobre cómo usar las opciones de instalación personalizada. 

    ![Uso de una instancia de SQL Server existente](media/tshoot-connect-install-issues/tshoot-connect-install-issues/useexistingsqlserver.png)

* En la página siguiente, seleccione la opción "Crear una cuenta de AD", para evitar los problemas de permisos con la cuenta existente.

    ![Cuenta del bosque de AD](media/tshoot-connect-install-issues/tshoot-connect-install-issues/createnewaccount.png)

### <a name="common-issues"></a>**Problemas comunes**

* [Problemas de conectividad con Active Directory local](./reference-connect-adconnectivitytools.md).

* [Problemas de conectividad con Azure Active Directory en línea](./tshoot-connect-connectivity.md).

* [Problemas de permisos con Active Directory local](./how-to-connect-configure-ad-ds-connector-account.md).

## <a name="recommended-documents"></a>**Documentos recomendados**
* [Requisitos previos de Azure AD Connect](./how-to-connect-install-prerequisites.md)
* [Selección del tipo de instalación que se usará para Azure AD Connect](./how-to-connect-install-select-installation.md)
* [Introducción a Azure AD Connect mediante la configuración rápida](./how-to-connect-install-express.md)
* [Instalación personalizada de Azure AD Connect](./how-to-connect-install-custom.md)
* [Azure AD Connect: actualización de una versión anterior a la versión más reciente](./how-to-upgrade-previous-version.md)
* [Azure AD Connect: ¿Qué es un servidor de almacenamiento provisional?](./plan-connect-topologies.md#staging-server)
* [¿Qué es el módulo de PowerShell ADConnectivityTool?](./how-to-connect-adconnectivitytools.md)

## <a name="next-steps"></a>Pasos siguientes
- [Sincronización de Azure AD Connect](how-to-connect-sync-whatis.md).
- [¿Qué es la identidad híbrida?](whatis-hybrid-identity.md)