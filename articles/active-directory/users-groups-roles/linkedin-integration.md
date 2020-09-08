---
title: 'Consentimiento administrativo para las conexiones de cuentas de LinkedIn: Azure AD | Microsoft Docs'
description: Se explica cómo habilitar o deshabilitar conexiones de cuentas de integración de LinkedIn para aplicaciones de Microsoft en Azure Active Directory
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: how-to
ms.date: 11/08/2019
ms.author: curtand
ms.reviewer: beengen
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 08d6b12f897b8b7efb150b11ae4b3f5e2440f7c8
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797996"
---
# <a name="integrate-linkedin-account-connections-in-azure-active-directory"></a>Integración de conexiones de cuentas de LinkedIn en Azure Active Directory

Puede permitir que los usuarios de la organización accedan a sus conexiones de LinkedIn desde algunas aplicaciones de Microsoft. No se compartirá ningún dato hasta que los usuarios den su consentimiento para conectar sus cuentas. Puede integrar la organización en el [centro de administración](https://aad.portal.azure.com) de Azure Active Directory (Azure AD).

> [!IMPORTANT]
> La configuración de conexiones de cuentas de LinkedIn se encuentra actualmente en proceso de implementación en organizaciones de Azure AD. Cuando se implemente en la organización, estará habilitada de forma predeterminada.
>
> Excepciones:
>
> * La opción no está disponible para los clientes que usan Microsoft Cloud for US Government, Microsoft Cloud Germany, o bien Azure y Office 365 operado por 21Vianet en China.
> * La opción está desactivada de forma predeterminada para las organizaciones de Azure AD aprovisionadas en Alemania. Tenga en cuenta que la opción no está disponible para los clientes que usan Microsoft Cloud Germany.
> * La opción está desactivada de forma predeterminada para las organizaciones aprovisionadas en Francia.
>
> Una vez que se hayan habilitado las conexiones de cuentas de LinkedIn para la organización, las conexiones de cuentas funcionarán después de que los usuarios den su consentimiento para que las aplicaciones accedan a datos de la empresa en su nombre. Para obtener información sobre la configuración de consentimientos del usuario, vea [Cómo quitar el acceso de un usuario a una aplicación](../manage-apps/methods-for-removing-user-access.md).

## <a name="enable-linkedin-account-connections-in-the-azure-portal"></a>Habilitación de conexiones de cuentas de LinkedIn en Azure Portal

Puede habilitar las conexiones de cuentas de LinkedIn solo para los usuarios que quiera que tengan acceso, desde toda la organización a solo algunos usuarios seleccionados de su organización.

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com/) con una cuenta que tenga el rol de administrador global en la organización de Azure AD.
1. Seleccione **Usuarios**.
1. En la página **Usuarios**, seleccione **Configuración de usuario**.
1. En **Conexiones de cuentas de LinkedIn**, permita que los usuarios se conecten sus cuentas para acceder a sus conexiones de LinkedIn dentro de algunas aplicaciones de Microsoft. No se compartirá ningún dato hasta que los usuarios den su consentimiento para conectar sus cuentas.

    * Seleccione **Sí** para habilitar el servicio para todos los usuarios de la organización
    * Seleccione **Grupo seleccionado** para habilitar el servicio solo para un grupo de usuarios seleccionados de la organización
    * Seleccione **No** para retirar el consentimiento a todos los usuarios de la organización

    ![Integración de conexiones de cuentas de LinkedIn en la organización](./media/linkedin-integration/linkedin-integration.png)

1. Cuando haya terminado, haga clic en **Guardar** para guardar la configuración.

> [!Important]
> La integración de LinkedIn no estará habilitada por completo para los usuarios hasta que den su consentimiento para la conexión de sus cuentas. Al habilitar las conexiones de cuentas para los usuarios, no se comparte ningún dato.

### <a name="assign-selected-users-with-a-group"></a>Asignación de usuarios seleccionados a un grupo

Hemos sustituido la opción "Seleccionados" que especifica una lista de usuarios por la opción de seleccionar un grupo de usuarios, para que se pueda habilitar la capacidad de conectarse a cuentas de LinkedIn y Microsoft para un único grupo, en lugar de para muchos usuarios individuales. Si no dispone de conexiones de cuentas de LinkedIn habilitadas para usuarios individuales seleccionados, no es necesario hacer nada. Si ha habilitado previamente las conexiones de cuentas de LinkedIn para usuarios individuales seleccionados, debe:

1. Obtener la lista actual de usuarios individuales
1. Mover los usuarios individuales habilitados actualmente a un grupo
1. Usar el grupo del anterior como el grupo seleccionado en las conexiones de cuentas de LinkedIn en el centro de administración de Azure AD.

> [!NOTE]
> Incluso si no mueve los usuarios individuales seleccionados actualmente a un grupo, podrán seguir viendo información de LinkedIn en Microsoft Apps.

### <a name="move-currently-selected-users-to-a-group"></a>Desplazamiento de usuarios seleccionados a un grupo

1. Cree un archivo CSV de los usuarios seleccionados para las conexiones de cuentas de LinkedIn.
1. Inicie sesión en Microsoft 365 con la cuenta de administrador.
1. Inicie PowerShell.
1. Instale el módulo de Azure AD mediante la ejecución de `Install-Module AzureAD`.
1. Ejecute el siguiente script:

  ``` PowerShell
  $groupId = "GUID of the target group"
  
  $users = Get-Content 
  Path to the CSV file
  
  $i = 1
  foreach($user in $users} { Add-AzureADGroupMember -ObjectId $groupId -RefObjectId $user ; Write-Host $i Added $user ; $i++ ; Start-Sleep -Milliseconds 10 }
  ```

Para usar el grupo del paso 2 como grupo seleccionado en las conexiones de cuentas de LinkedIn en el centro de administración de Azure AD, vea [Habilitar las conexiones de cuentas de LinkedIn en Azure Portal](#enable-linkedin-account-connections-in-the-azure-portal).

## <a name="use-group-policy-to-enable-linkedin-account-connections"></a>Usar la directiva de grupo para habilitar conexiones de cuentas de LinkedIn

1. Descargue los [archivos de plantilla de administración de Office 2016 (ADMX/ADML)](https://www.microsoft.com/download/details.aspx?id=49030)
1. Extraiga los archivos **ADMX** y cópielos en el almacén central.
1. Abra la Administración de directivas de grupo.
1. Cree un objeto de directiva de grupo con la configuración siguiente: **Configuración de usuario** > **Plantillas administrativas** > **Microsoft Office 2016** > **Varios** > **Mostrar características de LinkedIn en aplicaciones de Office**.
1. Seleccione **Habilitado** o **Deshabilitado**.
  
   State | Efecto
   ------ | ------
   **Enabled** | La opción **Mostrar características de LinkedIn en aplicaciones de Office** de Opciones de Office 2016 está habilitada. Los usuarios de la organización no podrán usar las características de LinkedIn en las aplicaciones de Office 2016.
   **Deshabilitada** | La opción **Mostrar características de LinkedIn en aplicaciones de Office** de Opciones de Office 2016 está deshabilitada y los usuarios finales no pueden cambiarla. Los usuarios de la organización no podrán usar las características de LinkedIn en las aplicaciones de Office 2016.

Esta directiva de grupo afecta solo a las aplicaciones de Office 2016 de los equipos locales. Si los usuarios deshabilitan LinkedIn en sus aplicaciones de Office 2016, pueden seguir viendo las características de LinkedIn en Office 365.

## <a name="next-steps"></a>Pasos siguientes

* [Consentimiento del usuario y uso compartido para LinkedIn](linkedin-user-consent.md)

* [Información y características de LinkedIn en las aplicaciones de Microsoft](https://go.microsoft.com/fwlink/?linkid=850740)

* [Centro de ayuda de LinkedIn](https://www.linkedin.com/help/linkedin)

* [Visualización de la configuración de integración con LinkedIn actual en Azure Portal](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/UserManagementMenuBlade/UserSettings)