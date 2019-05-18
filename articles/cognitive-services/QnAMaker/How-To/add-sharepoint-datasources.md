---
title: 'Archivos de SharePoint: QnA Maker'
titleSuffix: Azure Cognitive Services
description: Agregar orígenes de datos protegidos de Sharepoint a la base de conocimiento para enriquecer la base de conocimiento con preguntas y respuestas que se pueden proteger con Active Directory.
services: cognitive-services
author: tulasim88
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: article
ms.date: 04/05/2019
ms.author: tulasim
ms.openlocfilehash: d02c3fbb762e83584a9ea277ef8d5cc15b391f37
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65792568"
---
# <a name="add-a-secured-sharepoint-data-source-to-your-knowledge-base"></a>Agregar un origen de datos protegido de Sharepoint a la base de conocimiento

Agregar orígenes de datos protegidos de Sharepoint a la base de conocimiento para enriquecer la base de conocimiento con preguntas y respuestas que se pueden proteger con Active Directory. 

Cuando se agrega un documento protegido de Sharepoint a la knowledge base, como el Administrador de QnA Maker, debe solicitar permiso de Active Directory para QnA Maker. Una vez que se concede este permiso desde el Administrador de Active Directory a QnA Maker para tener acceso a Sharepoint, no tiene que especificarse de nuevo. Cada adición de un documento posterior a la base de conocimiento no será necesario autorización si se encuentra en el mismo recurso de Sharepoint. 

Si el Administrador de base de conocimiento de QnA Maker no es el Administrador de Active Directory, necesita comunicarse con el Administrador de Active Directory para finalizar este proceso.

## <a name="add-supported-file-types-to-knowledge-base"></a>Agregar tipos de archivo compatibles para la base de conocimiento

Puede agregar todos los admitidos de QnA Maker [tipos de archivo](../Concepts/data-sources-supported.md) desde un servidor de Sharepoint a la base de conocimiento. Es posible que deba conceder [permisos](#permissions) si el recurso de archivo está protegido.

1. Desde el servidor de Sharepoint, seleccione el menú del botón de puntos suspensivos del archivo, `...`.
1. Copie la dirección URL del archivo.

    ![Obtenga la dirección URL del archivo Sharepoint seleccionando el menú de puntos suspensivos del archivo, a continuación, copie la dirección URL.](../media/add-sharepoint-datasources/get-sharepoint-file-url.png)

1. En el portal de QnA Maker, en el **configuración** página, [agregar la dirección URL](edit-knowledge-base.md#add-datasource) a la base de conocimiento. 

## <a name="permissions"></a>Permisos

Conceder permisos, se produce cuando se agrega un archivo protegido de un servidor de Sharepoint a una base de conocimiento. Según la configuración de Sharepoint arriba y los permisos de la persona que agrega el archivo, esto podría precisar:

* No hay pasos adicionales: la persona que agrega el archivo tiene todos los permisos necesarios.
* los pasos de ambos [Administrador de knowledge base](#knowledge-base-manager-add-sharepoint-data-source-in-qna-maker-portal) y [Administrador de Active Directory](#active-directory-manager-grant-file-read-access-to-qna-maker).

Consulte los pasos indicados a continuación. 

### <a name="knowledge-base-manager-add-sharepoint-data-source-in-qna-maker-portal"></a>El Administrador de base de conocimiento: Agregar origen de datos de Sharepoint en el portal de QnA Maker

Cuando el **manager QnA Maker** agrega un documento protegido de Sharepoint a una base de conocimiento, la base de conocimiento manager inicia una solicitud de permiso que debe completar el Administrador de Active Directory.

La solicitud comienza con un elemento emergente para autenticarse en una cuenta de Active Directory. 

![Autenticar la cuenta de usuario](../media/add-sharepoint-datasources/authenticate-user-account.png)

Una vez que el Administrador de QnA Maker selecciona la cuenta, el Administrador de Active Directory recibirá un aviso que debe permitir acceso a la aplicación (no el Administrador de QnA Maker) a los recursos de Sharepoint de QnA Maker. El Administrador de Active Directory debe hacer esto para todos los recursos de Sharepoint, pero no todos los documentos de ese recurso. 

### <a name="active-directory-manager-grant-file-read-access-to-qna-maker"></a>Administrador de Active directory: conceder acceso de lectura de archivo a QnA Maker

El Administrador de Active Directory (no el Administrador de QnA Maker) necesita conceder acceso a QnA Maker para tener acceso a los recursos de Sharepoint seleccionando [este vínculo](https://login.microsoftonline.com/common/oauth2/v2.0/authorize?response_type=id_token&scope=Files.Read%20Files.Read.All%20Sites.Read.All%20User.Read%20User.ReadBasic.All%20profile%20openid%20email&client_id=c2c11949-e9bb-4035-bda8-59542eb907a6&redirect_uri=https%3A%2F%2Fwww.qnamaker.ai%3A%2FCreate&state=68) para autorizar a la aplicación empresarial de Sharepoint de Portal de QnA Maker tener de lectura de archivo permisos. 

![El Administrador de Azure Active Directory concede permiso de forma interactiva](../media/add-sharepoint-datasources/aad-manager-grants-permission-interactively.png)

<!--
The Active Directory manager must grant QnA Maker access either by application name, `QnAMakerPortalSharepoint`, or by application ID, `c2c11949-e9bb-4035-bda8-59542eb907a6`. 
-->
<!--
### Grant access from the interactive pop-up window 

The Active Directory manager will get a pop-up window requesting permissions to the `QnAMakerPortalSharepoint` app. The pop-up window includes the QnA Maker Manager email address that initiated the request, an `App Info` link to learn more about **QnAMakerPortalSharepoint**, and a list of permissions requested. Select **Accept** to provide those permissions. 

![Azure Active Directory manager grants permission interactively](../media/add-sharepoint-datasources/aad-manager-grants-permission-interactively.png)
-->
<!--

### Grant access from the App Registrations list

1. The Active Directory manager signs in to the Azure portal and opens **[App registrations list](https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ApplicationsListBlade)**. 

1. Search for and select the **QnAMakerPortalSharepoint** app. Change the second filter box from **My apps** to **All apps**. The app information will open on the right side.

    ![Select QnA Maker app in App registrations list](../media/add-sharepoint-datasources/select-qna-maker-app-in-app-registrations.png)

1. Select **Settings**.

    [![Select Settings in the right-side blade](../media/add-sharepoint-datasources/select-settings-for-qna-maker-app-registration.png)](../media/add-sharepoint-datasources/select-settings-for-qna-maker-app-registration.png#lightbox)

1. Under **API access**, select **Required permissions**. 

    ![Select 'Settings', then under 'API access', select 'Required permission'](../media/add-sharepoint-datasources/select-required-permissions-in-settings-blade.png)

1. Do not change any settings in the **Enable Access** window. Select **Grant Permission**. 

    [![Under 'Grant Permission', select 'Yes'](../media/add-sharepoint-datasources/grant-app-required-permissions.png)](../media/add-sharepoint-datasources/grant-app-required-permissions.png#lightbox)

1. Select **YES** in the pop-up confirmation windows. 

    ![Grant required permissions](../media/add-sharepoint-datasources/grant-required-permissions.png)
-->
### <a name="grant-access-from-the-azure-active-directory-admin-center"></a>Conceder acceso desde el centro de administración de Azure Active Directory

1. El Administrador de Active Directory, inicie sesión en el portal de Azure y abre  **[aplicaciones empresariales](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps)**. 

1. Busque `QnAMakerPortalSharepoint` el, seleccione la aplicación de QnA Maker. 

    [![Buscar QnAMakerPortalSharepoint en la lista de aplicaciones de empresa](../media/add-sharepoint-datasources/search-enterprise-apps-for-qna-maker.png)](../media/add-sharepoint-datasources/search-enterprise-apps-for-qna-maker.png#lightbox)

1. En **seguridad**, vaya a **permisos**. Seleccione **conceder consentimiento del Administrador de organización**. 

    [![Seleccione el usuario autenticado para el Administrador de Active Directory](../media/add-sharepoint-datasources/grant-aad-permissions-to-enterprise-app.png)](../media/add-sharepoint-datasources/grant-aad-permissions-to-enterprise-app.png#lightbox)

1. Seleccione una cuenta de inicio de sesión con permisos para conceder permisos de Active Directory. 


  
<!--

## Add Sharepoint data source with APIs

You need to get the Sharepoint file's URI before adding it to QnA Maker. 

## Get Sharepoint File URI

Use the following steps to transform the Sharepoint URL into a sharing token.

1. Encode the URL using [base64](https://en.wikipedia.org/wiki/Base64). 

1. Convert the base64-encoded result to an unpadded base64url format with the following character changes. 

    * Remove the equal character, `=` from the end of the value. 
    * Replace `/` with `_`. 
    * Replace `+` with `-`. 
    * Append `u!` to be beginning of the string. 

1. Sign in to Graph explorer and run the following query, where `sharedURL` is ...:

    ```
    https://graph.microsoft.com/v1.0/shares/<sharedURL>/driveitem
    ```

    Get the **@microsoft.graph.downloadUrl** and use this as `fileuri` in the QnA Maker APIs.

### Add or update a Sharepoint File URI to your knowledge base

Use the **@microsoft.graph.downloadUrl** from the previous section as the `fileuri` in the QnA Maker API for [adding a knowledge base](https://go.microsoft.com/fwlink/?linkid=2092179) or [updating a knowledge base](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/update). The following fields are mandatory: name, fileuri, filename, source.

```
{
    "name": "Knowledge base name",
    "files": [
        {
            "fileUri": "<@microsoft.graph.downloadURL>",
            "fileName": "filename.xlsx",
            "source": "<sharepoint link>"
        }
    ],
    "urls": [],
    "users": [],
    "hostUrl": "",
    "qnaList": []
}
```



## Remove QnA Maker app from Sharepoint authorization

1. Use the steps in the previous section to find the Qna Maker app in the Active Directory admin center. 
1. When you select the **QnAMakerPortalSharepoint**, select **Overview**. 
1. Select **Delete** to remove permissions. 

-->

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Colaborar en la base de conocimiento](collaborate-knowledge-base.md)