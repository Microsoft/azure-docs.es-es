---
title: Administración de los administradores locales en dispositivos unidos a Azure AD
description: Aprenda a asignar roles de Azure al grupo de administradores locales de un dispositivo Windows.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: how-to
ms.date: 06/28/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: ravenn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5d3082e3dc45102bc8700c7d1285ef832d09712a
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87419825"
---
# <a name="how-to-manage-the-local-administrators-group-on-azure-ad-joined-devices"></a>Administración del grupo de administradores locales en dispositivos unidos a Azure AD

Para administrar un dispositivo Windows, debe ser miembro del grupo de administradores locales. Como parte del proceso de unión a Azure Active Directory (Azure AD), Azure AD actualiza la pertenencia de este grupo en un dispositivo. Puede personalizar la actualización de la pertenencia para satisfacer los requisitos de su negocio. Una actualización de pertenencia es, por ejemplo, útil si desea permitir que el personal del soporte técnico realice tareas que requieran derechos de administrador en un dispositivo.

En este artículo se explica cómo funciona la actualización de la pertenencia de los administradores locales y cómo puede personalizarla durante una unión a Azure AD. El contenido de este artículo no se aplica a dispositivos **unidos a Azure AD híbrido**.

## <a name="how-it-works"></a>Funcionamiento

Al conectar un dispositivo Windows con Azure AD mediante una unión a Azure AD, Azure AD agrega las siguientes entidades de seguridad al grupo de administradores locales del dispositivo:

- El rol de administrador global de Azure AD
- El rol de administrador de dispositivos de Azure AD 
- El usuario que realiza la unión a Azure AD   

Al agregar los roles de Azure AD al grupo de administradores locales, puede actualizar los usuarios que pueden administrar un dispositivo en cualquier momento en Azure AD sin modificar nada en el dispositivo. Actualmente, no se pueden asignar grupos a un rol de administrador.
Azure AD también agrega el rol de administrador de dispositivos de Azure AD al grupo de administradores locales para admitir el principio de privilegio mínimo (PoLP). Además de los administradores globales, también puede habilitar a los usuarios a los que *solo* se ha asignado el rol de administrador de dispositivos para administrar uno. 

## <a name="manage-the-global-administrators-role"></a>Administración del rol de administrador global

Para ver y actualizar la pertenencia al rol de administrador global, consulte:

- [Visualización de todos los miembros de un rol de administrador en Azure Active Directory](../users-groups-roles/directory-manage-roles-portal.md)
- [Asignación de un usuario a roles de administrador en Azure Active Directory](../fundamentals/active-directory-users-assign-role-azure-portal.md)


## <a name="manage-the-device-administrator-role"></a>Administración del rol de administrador de dispositivos 

En Azure Portal, puede administrar el rol de administrador de dispositivos en la página **Dispositivos**. Para abrir la página **Dispositivos**:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global.
1. Busque y seleccione *Azure Active Directory*.
1. En la sección **Administrar**, haga clic en **Dispositivos**.
1. En la página **Dispositivos**, haga clic en **Configuración del dispositivo**.

Para modificar el rol de administrador de dispositivos, configure **Administradores locales adicionales en dispositivos unidos a Azure AD**.  

![Otros administradores locales](./media/assign-local-admin/10.png)

>[!NOTE]
> Esta opción requiere un inquilino de Azure AD Premium. 

Los administradores de dispositivos se asignan a todos los dispositivos unidos a Azure AD. No se puede definir el ámbito de los administradores de dispositivos a un conjunto específico de dispositivos. La actualización del rol de administrador de dispositivos no necesariamente tiene un impacto inmediato en los usuarios afectados. En los dispositivos en los que un usuario ya ha iniciado sesión, la elevación de los privilegios tiene lugar cuando las *dos* acciones siguientes tienen lugar:

- Han pasado hasta cuatro horas para que Azure AD emita un nuevo token de actualización principal con los privilegios adecuados. 
- El usuario cierra sesión y la vuelve a iniciar, sin bloquear o desbloquear, para actualizar su perfil.

>[!NOTE]
> Las acciones anteriores no se aplican a los usuarios que no han iniciado sesión en el dispositivo pertinente previamente. En este caso, los privilegios de administrador se aplican inmediatamente después de su primer inicio de sesión en el dispositivo. 

## <a name="manage-administrator-privileges-using-azure-ad-groups-preview"></a>Administración de los privilegios de administrador con grupos de Azure AD (versión preliminar)

>[!NOTE]
> Esta funcionalidad actualmente está en su versión preliminar.

A partir de la actualización de 2004 de Windows 10, se pueden usar grupos de Azure AD para administrar los privilegios de administrador en dispositivos unidos a Azure AD con la directiva de MDM [Grupos restringidos](/windows/client-management/mdm/policy-csp-restrictedgroups). Esta directiva permite asignar usuarios individuales o grupos de Azure AD al grupo de administradores locales en un dispositivo unido a Azure AD, lo que le proporciona la granularidad para configurar diferentes administradores para distintos grupos de dispositivos. 

Actualmente, no hay ninguna interfaz de usuario en Intune para administrar esta directiva y debe configurarse mediante [Configuración OMA-URI personalizada](/mem/intune/configuration/custom-settings-windows-10). A continuación, se indican algunas consideraciones para esta directiva: 

- La adición de grupos de Azure AD a través de la directiva requiere el SID del grupo, que se puede obtener mediante la ejecución de la API de grupos. El SID se define mediante la propiedad `securityIdentifier` de la API de grupos.
- Cuando se aplica la directiva de grupos restringidos, se quita cualquier miembro actual del grupo que no esté en la lista de miembros. Por lo tanto, la aplicación de esta directiva con nuevos miembros o grupos quitará a los administradores existentes, es decir, al usuario que se unió al dispositivo, el rol de administrador de dispositivos y el rol de administrador global del dispositivo. Para evitar la eliminación de los miembros existentes, debe configurarlos como parte de la lista de miembros en la Directiva de grupos restringidos. 
- Esta directiva solo se aplica a los siguientes grupos conocidos en un dispositivo con Windows 10: administradores, usuarios, invitados, usuarios avanzados, usuarios de Escritorio remoto y usuarios de Administración remota. 
- La administración de los administradores locales mediante la directiva de grupos restringidos no es aplicable a los dispositivos unidos a Azure AD híbrido o dispositivos registrados de Azure AD.
- Aunque la directiva de grupos restringidos existía antes de la actualización de 2004 de Windows 10, no admitía grupos de Azure AD como miembros del grupo de administradores locales de un dispositivo. 

## <a name="manage-regular-users"></a>Administración de los usuarios normales

De forma predeterminada, Azure AD agrega el usuario que realiza la unión a Azure AD al grupo de administradores del dispositivo. Si desea evitar que los usuarios normales se conviertan en administradores locales, tiene las siguientes opciones:

- [Windows Autopilot](/windows/deployment/windows-autopilot/windows-10-autopilot): Windows Autopilot le ofrece una opción para evitar que el usuario principal que realiza la unión se convierta en un administrador local. Puede hacerlo mediante la [creación de un perfil de Autopilot](/intune/enrollment-autopilot#create-an-autopilot-deployment-profile).
- [Inscripción masiva](/intune/windows-bulk-enroll): una unión a Azure AD que se realiza en el contexto de una inscripción masiva tiene lugar en el contexto de un usuario creado automáticamente. Los usuarios que inician sesión después de que se hayan unido un dispositivo no se agregan al grupo de administradores.   

## <a name="manually-elevate-a-user-on-a-device"></a>Elevación manual de un usuario en un dispositivo 

Además de usar el proceso de unión a Azure AD, también puede elevar manualmente un usuario regular para que se convierta en un administrador local en un dispositivo específico. Este paso requiere que ya sea miembro del grupo de administradores locales. 

A partir de la versión **Windows 10 1709**, puede realizar esta tarea en **Configuración -> Cuentas -> Otros usuarios**. Seleccione **Agregar un usuario de trabajo o escuela**, escriba el UPN del usuario en **Cuenta de usuario** y seleccione *Administrador* en **Tipo de cuenta**  
 
Además, también puede agregar usuarios mediante el símbolo del sistema:

- Si los usuarios del inquilino están sincronizados desde Active Directory local, utilice `net localgroup administrators /add "Contoso\username"`.
- Si se crean los usuarios del inquilino de Azure AD, utilice `net localgroup administrators /add "AzureAD\UserUpn"`.

## <a name="considerations"></a>Consideraciones 

No puede asignar grupos al rol de administrador de dispositivos, solo se permiten usuarios individuales.

Los administradores de dispositivos se asignan a todos los dispositivos unidos a Azure AD. No se puede limitar a un conjunto específico de dispositivos.

Cuando se quitan usuarios del rol de administrador de dispositivos, estos siguen teniendo el privilegio de administrador local en un dispositivo siempre que hayan iniciado sesión en él. El privilegio se revoca durante el siguiente inicio de sesión cuando se emite un nuevo token de actualización principal. Esta revocación, similar a la elevación de privilegios, puede tardar hasta cuatro horas.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre cómo administrar dispositivos en Azure Portal, vea [Managing devices using the Azure portal (Administración de dispositivos con Azure Portal)](device-management-azure-portal.md)
- Para más información sobre el acceso condicional basado en dispositivo, consulte [Configuración de directivas de acceso condicional basadas en dispositivo de Azure Active Directory](../conditional-access/require-managed-devices.md).
