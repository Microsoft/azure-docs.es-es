---
title: Usuarios marcados en riesgo en el portal de Azure Active Directory | Microsoft Docs
description: Aprenda sobre el informe de seguridad de usuarios marcados en riesgo en el portal de Azure Active Directory
services: active-directory
author: MarkusVi
manager: daveba
ms.assetid: addd60fe-d5ac-4b8b-983c-0736c80ace02
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 725b3b712313c199382cebe32bed98f2115fc262
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89231170"
---
# <a name="remediate-users-flagged-for-risk-in-the-azure-active-directory-portal"></a>Remedie los usuarios marcados como en riesgo en el portal de Azure Active Directory

Con los informes de seguridad de Azure Active Directory (Azure AD), puede medir la probabilidad de que haya cuentas de usuario en peligro en su entorno. Un usuario marcado como en peligro es un indicador de una cuenta de usuario que puede haber estado en peligro.

Microsoft se compromete a ayudar a mantener los entornos seguros. Como parte de este compromiso, Microsoft supervisa continuamente aquellas actividades que son inusuales o que coinciden con patrones de ataque conocidos. 

Si se detectan actividades inusuales que puedan indicar un acceso no autorizado a las cuentas de algunos de sus usuarios, recibirá las notificaciones pertinentes, lo que le permitirá realizar las acciones necesarias. Esto no significa que se hayan puesto en peligro los sistemas de Microsoft.

## <a name="access-the-users-flagged-for-risk-report"></a>Acceso al informe de usuarios marcados en riesgo

Puede revisar los usuarios marcados en riesgo mediante el [informe de usuarios en riesgo](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RiskyUsers) de Azure Portal. Si no tiene Azure AD, puede registrarse de forma gratuita en [https://aka.ms/AccessAAD](https://aka.ms/AccessAAD). 

Desde el informe de usuarios marcados en riesgo puede realizar cualquiera de las siguientes acciones para cada uno:

- Generar una contraseña temporal.
- Requerir al usuario que restablezca de forma segura su contraseña la próxima vez que inicie sesión.
- Descartar el riesgo del usuario sin realizar ninguna acción de corrección.

Para obtener más información, consulte [Reporte de seguridad de usuarios marcados en riesgo](../identity-protection/overview-identity-protection.md).

### <a name="azure-ad-subscription-for-office-365-customers"></a>Suscripción a Azure AD para clientes de Office 365

También puede utilizar sus credenciales de Office 365 para acceder al **Centro de administración de Azure**. Una vez que haya activado el acceso a Azure AD, se le redirigirá al portal de Azure AD. Con el nivel de suscripción básico, el nivel de detalle que se proporciona en los informes es limitado. Los suscriptores Premium de Azure tienen a su disposición más datos y análisis.

Para tener acceso a los informes de **usuarios marcados en riesgo** en el centro de administración de Microsoft 365:

1.  Desde el menú de navegación de la izquierda, seleccione **Centros de administración**. 
2.  Seleccione **Azure AD**.
3.  Inicie sesión en el **Centro de administración de Azure Active Directory**.
4.  Si en la parte superior de la página se muestra un banner que dice **Check out the new portal** (Eche un vistazo al nuevo portal), seleccione el vínculo.
4.  En el panel de navegación izquierdo, seleccione **Azure Active Directory**. 
5.  En el panel de navegación, seleccione **Usuarios marcados en riesgo** de la sección **Seguridad**.

## <a name="remediation-actions"></a>Acciones de corrección

Realice las acciones siguientes para ayudar a corregir las cuentas afectadas y proteger su entorno:

1.  [Valide la información correcta](https://aka.ms/MFAValid) de la autenticación multifactor y del restablecimiento de contraseña de autoservicio. 
2.  [Habilite la autenticación multifactor](https://aka.ms/MFAuth) para todos los usuarios. 
3.  Use este [script de corrección](https://aka.ms/remediate) para realizar automáticamente los pasos siguientes en todas las cuentas afectadas: 

    a. Restablezca la contraseña para proteger la cuenta y terminar las sesiones activas.

    b. Quite los delegados del buzón.

    c. Deshabilite las reglas de reenvío de correo a dominios externos.

    d. Quite la propiedad de reenvío de correo global del buzón.

    e. Habilite MFA en la cuenta del usuario.

    f. Establezca que la complejidad de la contraseña de la cuenta sea alta.

    g. Habilite la auditoría del buzón.

    h. Genere un registro de auditoría para que el administrador lo revise.

4. Investigue el inquilino de Office 365 y otras infraestructuras de TI, lo que incluye una revisión de todos los valores de inquilino, las cuentas de usuario y los valores de configuración por usuario para su posible modificación. Compruebe los indicadores de los métodos de persistencia, así como los indicadores que un intruso puede haber aprovechado como punto inicial de apoyo para obtener las credenciales de la VPN o acceso a otros recursos de la organización. 

5.  Como parte de la investigación, considere si debe enviar notificaciones a las autoridades gubernamentales, lo que incluye el cumplimiento de la ley.

Además, debe:

- Leer e implementar esta [guía para afrontar actividades inusuales](https://aka.ms/fixaccount). 
- [Habilitar la canalización de la auditoría](https://aka.ms/improvesecurity) para que le ayude a analizar la actividad de su empresa. Una vez completado, el almacén de auditoría empieza a rellenarse con los registros de actividad. En este momento, también puede aprovechar el [recurso de búsqueda e investigación del Centro de seguridad y cumplimiento](https://aka.ms/sccsearch). 
- Revise los permisos de delegado y las reglas de reenvío de correo de todos los buzones. Este [script de PowerShell](https://aka.ms/delegateforwardrules) se puede usar para realizar esta tarea. 

## <a name="next-steps"></a>Pasos siguientes

* [Azure Active Directory Identity Protection](../identity-protection/overview-identity-protection.md)
* [Usuarios marcados en riesgo](../identity-protection/overview-identity-protection.md)