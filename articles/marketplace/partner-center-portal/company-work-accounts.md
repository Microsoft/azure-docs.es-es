---
title: Las cuentas de trabajo de empresa y centro de partners
description: Cómo comprobar si su empresa tiene una cuenta de trabajo configurado con Microsoft, cree una nueva cuenta de trabajo o configurar varias cuentas de trabajo para usar con el centro de partners.
author: mattwojo
manager: evansma
ms.author: parthp
ms.service: marketplace
ms.topic: how-to
ms.date: 05/30/2019
ms.openlocfilehash: df8ab370e8874e07f8985ecfb3a772848a2e2b21
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65806287"
---
# <a name="company-work-accounts-and-partner-center"></a>Las cuentas de trabajo de empresa y centro de partners

Centro de partners usa cuentas profesionales de empresa, también conocido como los inquilinos de Azure Active Directory (AD) para administrar el acceso de cuenta para varios usuarios, permisos de control, grupos host y las aplicaciones y mantener los datos de perfil. Al vincular el dominio de cuenta de correo electrónico de trabajo de su empresa a su cuenta de centro de partners, los empleados de su empresa pueden iniciar sesión al centro de partners para administrar las ofertas de marketplace mediante su propio trabajo nombres de usuario y contraseñas.

## <a name="check-whether-your-company-already-has-a-work-account"></a>Comprobar si su empresa ya tiene una cuenta profesional

Si su empresa se suscribe a un servicio de nube de Microsoft, como Azure, Microsoft Intune u Office 365, ya tiene un trabajo cuenta dominio de correo electrónico (que también se denomina un inquilino de Azure Active Directory) que se puede usar con el centro de partners.

Siga estos pasos para comprobar:
1. Inicie sesión en el portal de administración de Azure en https://portal.azure.com.
2. Seleccione **Azure Active Directory** en el menú de navegación izquierda y, a continuación, seleccione **nombres de dominio personalizados**.
3. Si ya tiene una cuenta profesional, se mostrará el nombre de dominio.

Si su empresa ya no tiene una cuenta profesional, se creará automáticamente durante el proceso de inscripción de centro de partners.

## <a name="set-up-multiple-work-accounts"></a>Configurar varias cuentas de trabajo

Antes de decidirse a usar una cuenta de trabajo existente, considere la posibilidad de cuántos usuarios en la cuenta de trabajo necesitará acceso a centro de partners. Si tiene usuarios en la cuenta de trabajo que no necesita tener acceso a centro de partners, puede considerar la creación de varias cuentas de trabajo, para que solo aquellos usuarios que necesitan acceso a centro de partners se representan en una cuenta determinada.

## <a name="create-a-new-work-account"></a>Crear una nueva cuenta de trabajo

Para crear una nueva cuenta profesional para su empresa, siga estos pasos. Es posible que deba solicitar asistencia de la persona que tenga permisos administrativos en la cuenta de Microsoft Azure de su compañía.

1. Inicie sesión en el [Portal de Microsoft Azure](https://portal.azure.com).
2. En el menú de navegación izquierdo, seleccione el **Azure Active Directory** -> **usuarios**.
3. Seleccione **nuevo usuario** y crear una nueva cuenta de Azure profesional escribiendo una dirección de correo electrónico y nombre. Asegúrese de que el **rol del directorio** está establecido en **usuario** y seleccione el **Mostrar contraseña** casilla en la parte inferior para ver y tome nota de la contraseña generada automáticamente.
4. Seleccione **crear** para guardar el nuevo usuario.

La dirección de correo electrónico para la cuenta de usuario debe ser un nombre de dominio comprobado en el directorio. Puede enumerar todos los dominios comprobados en el directorio seleccionando **Azure Active Directory** -> **nombres de dominio personalizados** en el menú de navegación izquierda.

Para más información acerca de cómo agregar dominios personalizados en Azure Active Directory, consulte [agregar o asociar un dominio de Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-add-domain).

## <a name="troubleshoot-work-email-sign-in"></a>Solución de problemas de inicio de sesión de correo electrónico de trabajo

Si tiene dificultades para iniciar sesión su cuenta de trabajo (también conocido como inquilino de Azure AD), busque el escenario en el diagrama siguiente que mejor coincida con su situación y siga los pasos recomendados.

![Diagrama de solución de problemas de inicio de sesión de cuenta de trabajo](./media/onboarding-aad-flow.png)

## <a name="next-steps"></a>Pasos siguientes

- [Administrar su cuenta de Marketplace comercial en el centro de partners](./manage-account.md) 
