---
title: 'Azure AD Connect: Escritura diferida de grupos'
description: En este artículo se describe la escritura diferida de grupos en Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.date: 06/11/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8c8fb54261f51d74f02b7b79c27f7a2043426686
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87385150"
---
# <a name="azure-ad-connect-group-writeback"></a>Escritura diferida de grupos en Azure AD Connect

La escritura diferida de grupos permite a los clientes aprovechar los grupos en la nube para sus necesidades híbridas. Si utiliza la característica Grupos de Office 365, estos grupos pueden estar representados en su instancia de Active Directory local. Esta opción **solo** está disponible si dispone de Exchange en su Active Directory local.

## <a name="pre-requisites"></a>Requisitos previos
Se deben cumplir los siguientes requisitos previos para habilitar la escritura diferida de grupos.
- Licencias de Azure Active Directory Premium para su inquilino.
- Una implementación híbrida configurada entre la organización de Exchange local y Office 365, así como la comprobación de que funciona correctamente.
- Instalación de una versión compatible de Exchange local
- Configuración del inicio de sesión único con Azure Active Directory Connect 

## <a name="enable-group-writeback"></a>Habilitación de la escritura diferida de grupos
Para habilitar la escritura diferida de grupos, siga estos pasos:

1. Abra el Asistente para Azure AD Connect, seleccione **Configurar** y, a continuación, haga clic en **Siguiente**.
2. Seleccione **Personalizar las opciones de sincronización** y haga clic en **Siguiente**.
3. En la página **Conectarse a Azure AD**, escriba sus credenciales. Haga clic en **Next**.
4. En la página **Características opcionales**, compruebe que las opciones que configuró anteriormente sigan estando seleccionadas.
5. Seleccione **Reescritura de grupos** y, a continuación, haga clic en **Siguiente**.
6. En la página **Escritura diferida**, seleccione una unidad organizativa (UO) de Active Directory para almacenar los objetos que se sincronizan entre Office 365 y la organización local. A continuación, haga clic en **Siguiente**.
7. En la página **Listo para configurar**, haga clic en **Configurar**.
8. Una vez completado el asistente, haga clic en **Salir** en la página Configuración completada.
9. Abra Windows PowerShell como administrador en el servidor de Azure Active Directory Connect y ejecute los siguientes comandos.

```Powershell
$AzureADConnectSWritebackAccountDN =  <MSOL_ account DN>
Import-Module "C:\Program Files\Microsoft Azure Active Directory Connect\AdSyncConfig\AdSyncConfig.psm1"
Set-ADSyncUnifiedGroupWritebackPermissions -ADConnectorAccountDN $AzureADConnectSWritebackAccountDN
```

Para obtener más información sobre la configuración de los grupos de Office 365, consulte [Configuración de grupos de Microsoft 365 con Exchange híbrido local](https://docs.microsoft.com/exchange/hybrid-deployment/set-up-microsoft-365-groups#enable-group-writeback-in-azure-ad-connect).

## <a name="disabling-group-writeback"></a>Deshabilitación de la escritura diferida de grupos
Para deshabilitar la escritura diferida de grupos, siga estos pasos: 


1. Inicie el Asistente para Azure Active Directory Connect y vaya a la página de tareas adicionales. Seleccione la tarea **Personalizar las opciones de sincronización** y haga clic en **Siguiente**.
2. En la página **Características opcionales**, desactive la escritura diferida de grupos.  Recibirá una advertencia que le indicará que los grupos se eliminarán.  Haga clic en **Sí**.
   >[!IMPORTANT]
   > Al deshabilitar la escritura diferida de grupos, los grupos creados anteriormente mediante esta característica se eliminarán de Active Directory local durante el próximo ciclo de sincronización. 

   ![Desactivar la casilla](media/how-to-connect-group-writeback/group2.png)
  
3. Haga clic en **Next**.
4. Haga clic en **Configurar**.

 >[!NOTE]
 > Al deshabilitar la escritura diferida de grupos, se establecerán las marcas de importación y sincronización completas en "true" en el conector de Azure Active Directory, lo que provocará que se propaguen los cambios de la regla durante el siguiente ciclo de sincronización y se eliminen los grupos que se habían escrito previamente en Active Directory.

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).
