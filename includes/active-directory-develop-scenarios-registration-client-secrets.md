---
title: archivo de inclusión
description: archivo de inclusión para las páginas de aterrizaje del escenario de cliente confidencial (demonio, aplicación web, API web)
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/15/2020
ms.author: jmprieur
ms.custom: include file
ms.openlocfilehash: a3acdbb93dd20f0b89e4f99d64f5f7a30ce40623
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87102995"
---
## <a name="register-secrets-or-certificates"></a>Registro de certificados o secretos

Al igual que para cualquier aplicación cliente confidencial, debe registrar un secreto o certificado. Puede registrar los secretos de aplicación mediante la experiencia interactiva en [Azure Portal ](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredAppsPreview) o bien con herramientas de línea de comandos (por ejemplo, PowerShell).

### <a name="register-client-secrets-by-using-the-application-registration-portal"></a>Registro de secretos de cliente con el portal de registro de aplicación

La administración de credenciales de cliente se produce en la página **Certificados y secretos** de una aplicación:

![Página Certificados y secretos](../articles/active-directory/develop/media/quickstart-update-azure-ad-app-preview/credentials-certificates-secrets.png)

- Para crear un *secreto de cliente*, seleccione **Nuevo secreto de cliente** en el registro de la aplicación de Azure Portal. Al crear un secreto de cliente, _debe_ registrar la cadena del secreto antes de salir del panel **Certificates & secrets** (Certificados y secretos). La cadena del secreto nunca se vuelve a mostrar.
- Durante el registro de aplicación, se utiliza el botón **Cargar certificado** para cargarlo. Azure AD solo admite certificados que se registran directamente en la aplicación y no siguen las cadenas de certificados.

Para más información, consulte [Inicio rápido: Configuración de una aplicación cliente para tener acceso a las API web | Adición de credenciales a la aplicación](../articles/active-directory/develop/quickstart-configure-app-access-web-apis.md#add-credentials-to-your-web-application)

### <a name="register-client-secrets-by-using-powershell"></a>Registro de los secretos de cliente mediante PowerShell

Como alternativa, puede registrar la aplicación con Azure AD mediante herramientas de línea de comandos. El ejemplo [active-directory-dotnetcore-daemon-v2](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2) muestra cómo registrar un certificado o un secreto de aplicación con una aplicación de Azure AD:

- Para más información sobre cómo registrar un secreto de aplicación, consulte [AppCreationScripts/Configure.ps1](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/5199032b352a912e7cc0fce143f81664ba1a8c26/AppCreationScripts/Configure.ps1#L190).
- Para más información sobre cómo registrar un certificado con una aplicación, consulte [AppCreationScripts-withCert/Configure.ps1](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/5199032b352a912e7cc0fce143f81664ba1a8c26/AppCreationScripts-withCert/Configure.ps1#L162-L178).
