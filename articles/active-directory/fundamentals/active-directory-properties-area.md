---
title: 'Incorporación de información de privacidad de una organización: Azure Active Directory | Microsoft Docs'
description: Instrucciones sobre cómo agrear información de privacidad de su organización en el área de propiedades de Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
ms.date: 04/17/2018
ms.author: ajburnle
ms.reviewer: bpham
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: f47ae9b087615a77f6bd4d3f14f1b0052037cba6
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321980"
---
# <a name="add-your-organizations-privacy-info-using-azure-active-directory"></a>Incorporación de información de privacidad de su organización con Azure Active Directory
En este artículo se explica cómo un administrador de inquilinos puede agregar información relacionada con la privacidad al inquilino de Azure Active Directory (Azure AD) de una organización, a través de Azure Portal.

Se recomienda agregar su contacto de privacidad global y la declaración de privacidad de su organización, de modo que los empleados internos e invitados externos puedan revisar las directivas. Dado que las declaraciones de privacidad se crean de forma única y específica para cada negocio, es recomendable ponerse en contacto con un abogado para obtener ayuda.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="add-your-privacy-info-on-azure-ad"></a>Incorporación de la información de privacidad en Azure AD
Puede agregar información de privacidad de su organización en el área de **Propiedades** de Azure AD.

### <a name="to-access-the-properties-area-and-add-your-privacy-information"></a>Para acceder al área de propiedades y agregar la información de privacidad

1. Inicie sesión en Azure Portal como administrador de inquilinos.

2. En la barra de navegación izquierda, seleccione **Azure Active Directory** y, luego, seleccione **Propiedades**.

    Se muestra el área de **Propiedades**.

    ![Área de propiedades de Azure AD, donde se resalta el área de información de privacidad](media/active-directory-properties-area/properties-area.png)

3. Agregar la información de privacidad de sus empleados:

    - **Contacto técnico.** Escriba la dirección de correo electrónico de la persona de contacto para el soporte técnico de su organización.
    
    - **Contacto de privacidad global.** Escriba la dirección de correo electrónico de la persona de contacto para consultas sobre privacidad de los datos personales. Esta persona también es quien se pone en contacto con Microsoft si se produce una vulneración de datos. Si no aparece ninguna persona aquí, Microsoft se pone en contacto con los administradores globales.

    - **URL de la declaración de privacidad.** Escriba el vínculo del documento de su organización que describe la forma en que su organización controla la privacidad de datos interna y externa del invitado.

        >[!Important]
        >Si no incluye su propia declaración de privacidad o su contacto de privacidad, los invitados externos verán el texto en el cuadro de diálogo **Permisos de revisión** que dice: **<_nombre de su organización_> no ha proporcionado vínculos de sus términos que pueda revisar**. Por ejemplo, un usuario invitado verá este mensaje cuando reciba una invitación para acceder a una organización a través de la colaboración B2B.

        ![Cuadro de diálogo de permisos de revisión de colaboración B2B con el mensaje](media/active-directory-properties-area/active-directory-no-privacy-statement-or-contact.png)

4. Seleccione **Guardar**.

## <a name="next-steps"></a>Pasos siguientes
- [Canje de invitación de colaboración B2B de Azure Active Directory](../external-identities/redemption-experience.md)
- [Adición o modificación de la información de perfil de un usuario en Azure Active Directory](active-directory-users-profile-azure-portal.md)