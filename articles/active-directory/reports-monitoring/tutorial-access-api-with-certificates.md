---
title: Tutorial de Reporting API de AD con certificados | Microsoft Docs
description: En este tutorial se explica cómo usar Reporting API de Azure AD con credenciales de certificado para obtener datos de directorios sin intervención del usuario.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: bc763a99c945925b80171738f4076e6305d92df9
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89229466"
---
# <a name="tutorial-get-data-using-the-azure-active-directory-reporting-api-with-certificates"></a>Tutorial: Obtención de datos mediante Reporting API de Azure Active Directory con certificados

Las [API de generación de informes de Azure Active Directory (Azure AD)](concept-reporting-api.md) proporcionan acceso mediante programación a los datos a través de un conjunto de API de REST. Estas API pueden llamarse desde una variedad de lenguajes de programación y herramientas. Si quiere acceder a Reporting API de Azure AD sin intervención del usuario, debe configurar el acceso para usar certificados.

En este tutorial, aprenderá a crear un certificado de prueba para acceder a Microsoft Graph API para la creación de informes. El uso de certificados de prueba no se recomienda en un entorno de producción. 

## <a name="prerequisites"></a>Prerrequisitos

1. Para obtener acceso a datos de inicio de sesión, asegúrese de que tiene un inquilino de Azure Active Directory con una licencia prémium (P1 y P2). Consulte [Introducción a Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md) para actualizar la edición de Azure Active Directory. Tenga en cuenta que si no tuvo ningún dato de actividades antes de la actualización, los datos tardarán un par de días en mostrarse en los informes después de actualizar a una licencia premium. 

2. Cree o cambie a una cuenta de usuario con el rol **Administrador global**, **Administrador de seguridad**, **Lector de seguridad** o **Lector de informes** en el inquilino. 

3. Complete los [requisitos previos para acceder a la API de generación de informes de Azure Active Directory](howto-configure-prerequisites-for-reporting-api.md). 

4. Descargue e instale [Azure AD PowerShell V2](https://github.com/Azure/azure-docs-powershell-azuread/blob/master/docs-conceptual/azureadps-2.0/install-adv2.md).

5. Instale [MSCloudIdUtils](https://www.powershellgallery.com/packages/MSCloudIdUtils/). Este módulo proporciona varios cmdlets de utilidades incluidos los siguientes:
    - Las bibliotecas ADAL necesarias para la autenticación
    - Los tokens de acceso de usuario, las claves de aplicación y los certificados mediante ADAL
    - Graph API con control de resultados paginados

6. Si es la primera vez que usa el módulo, ejecute **Install-MSCloudIdUtilsModule**; en caso contrario, puede importarlo mediante el comando de PowerShell **Import-Module**. La sesión debe tener un aspecto similar a esta pantalla: ![Windows PowerShell](./media/tutorial-access-api-with-certificates/module-install.png)
  
7. Use el commandlet **New-SelfSignedCertificate** de PowerShell para crear un certificado de prueba.

   ```
   $cert = New-SelfSignedCertificate -Subject "CN=MSGraph_ReportingAPI" -CertStoreLocation "Cert:\CurrentUser\My" -KeyExportPolicy Exportable -KeySpec Signature -KeyLength 2048 -KeyAlgorithm RSA -HashAlgorithm SHA256
   ```

8. Use el commandlet **Export-Certificate** para exportarlo en un archivo de certificado.

   ```
   Export-Certificate -Cert $cert -FilePath "C:\Reporting\MSGraph_ReportingAPI.cer"

   ```

## <a name="get-data-using-the-azure-active-directory-reporting-api-with-certificates"></a>Obtención de datos mediante Reporting API de Azure Active Directory con certificados

1. Vaya a [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**, luego, seleccione **Registros de aplicaciones** y elija la aplicación de la lista. 

2. Seleccione **Certificates & secrets** (Certificados y secretos) en la sección **Administrar** de la hoja de registro de aplicaciones y elija **Cargar certificado**.

3. Seleccione el archivo de certificado del paso anterior y elija **Agregar**. 

4. Anote el identificador de la aplicación y la huella digital del certificado que acaba de registrar con la aplicación. Para encontrar la huella digital, en la página de la aplicación en el portal, vaya a **Certificates & secrets** (Certificados y secretos) en la sección **Administrar**. La huella digital estará en la lista **Certificados**.

5. Abra el manifiesto de aplicación en el editor de manifiesto insertado y compruebe que la propiedad *keyCredentials* está actualizada con la nueva información del certificado, como se muestra a continuación. 

   ```
   "keyCredentials": [
        {
            "customKeyIdentifier": "$base64Thumbprint", //base64 encoding of the certificate hash
            "keyId": "$keyid", //GUID to identify the key in the manifest
            "type": "AsymmetricX509Cert",
            "usage": "Verify",
            "value":  "$base64Value" //base64 encoding of the certificate raw data
        }
    ]
   ``` 
6. Ahora, puede obtener un token de acceso para MS Graph API con este certificado. Use el cmdlet **Get-MSCloudIdMSGraphAccessTokenFromCert** del módulo MSCloudIdUtils de PowerShell pasando el identificador de la aplicación y la huella digital que obtuvo en el paso anterior. 

   ![Portal de Azure](./media/tutorial-access-api-with-certificates/getaccesstoken.png)

7. Use el token de acceso del script de Powershell para consultar Graph API. Use el cmdlet **Invoke-MSCloudIdMSGraphQuery** desde MSCloudIDUtils para enumerar el punto de conexión de SignIns y DirectoryAudits. Este cmdlet controla los resultados de paginado múltiple y los envía a la canalización de PowerShell.

8. Consulte el punto de conexión DirectoryAudits para recuperar los registros de auditoría. 
   ![Azure Portal](./media/tutorial-access-api-with-certificates/query-directoryAudits.png)

9. Consulte el punto de conexión de SignIns para recuperar los registros de inicio de sesión.
    ![Azure Portal](./media/tutorial-access-api-with-certificates/query-signins.png)

10. Ahora puede exportar estos datos en un archivo CSV y guardarlos en un sistema SIEM. También puede encapsular el script en una tarea programada para obtener datos de Azure AD de su inquilino periódicamente sin tener que almacenar las claves de aplicación en el código fuente. 

## <a name="next-steps"></a>Pasos siguientes

* [Obtención de una primera impresión de Reporting API](concept-reporting-api.md)
* [Referencia de la API de auditoría](/graph/api/resources/directoryaudit?view=graph-rest-beta) 
* [Referencia de la API de generación de informes de actividad de inicio de sesión](/graph/api/resources/signin?view=graph-rest-beta)