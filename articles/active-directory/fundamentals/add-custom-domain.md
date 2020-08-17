---
title: 'Incorporación de un dominio personalizado: Azure Active Directory | Microsoft Docs'
description: Instrucciones sobre cómo agregar un dominio personalizado mediante Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 10/25/2019
ms.author: ajburnle
ms.reviewer: elkuzmen
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: c8035744cd7ff7bb3d8347a7d26796531a666bbd
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87796818"
---
# <a name="add-your-custom-domain-name-using-the-azure-active-directory-portal"></a>Incorporación del nombre de dominio personalizado mediante el portal de Azure Active Directory

Cada inquilino de Azure AD nuevo incluye un nombre de dominio inicial, *\<domainname>.onmicrosoft.com*. No se puede cambiar o eliminar el nombre de dominio inicial, pero puede agregar nombres de la organización. La incorporación de nombres de dominio personalizados le ayuda a crear nombres de usuario que resultan familiares a los usuarios, como *alain\@contoso.com*.

## <a name="before-you-begin"></a>Antes de empezar

Para poder agregar un nombre de dominio personalizado, cree el nombre de dominio con un registrador de dominios. Para un registrador de dominios acreditado, consulte [ICANN: Registradores acreditados](https://www.icann.org/registrar-reports/accredited-list.html).

## <a name="create-your-directory-in-azure-ad"></a>Creación del directorio en Azure AD

Después de obtener el nombre de dominio, puede crear su primer directorio de Azure AD. Inicie sesión en Azure Portal para su directorio con una cuenta que tenga el rol **Propietario** de la suscripción.

Cree el nuevo directorio siguiendo los pasos descritos en [Creación de un nuevo inquilino para la organización](active-directory-access-create-new-tenant.md#create-a-new-tenant-for-your-organization).

>[!IMPORTANT]
>La persona que crea el inquilino es automáticamente el administrador global de ese inquilino. El administrador global puede agregar administradores adicionales al inquilino.

Para más información sobre los roles de suscripción, consulte [Roles de Azure](../../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles).

>[!TIP]
> Si tiene previsto federar la instancia local de Windows Server AD en Azure AD, tiene que seleccionar la opción **Voy a configurar este dominio para el inicio de sesión único con mi Active Directory local** cuando ejecute la herramienta Azure AD Connect para sincronizar los directorios.
>
> También tiene que registrar el mismo nombre de dominio que seleccione para la federación con su directorio local en el paso **Dominio de Azure AD** del asistente. Para ver el aspecto de la configuración, consulte [Comprobación del dominio de Azure AD seleccionado para la federación](../hybrid/how-to-connect-install-custom.md#verify-the-azure-ad-domain-selected-for-federation). Si no tiene la herramienta Azure AD Connect, puede [descargarla aquí](https://go.microsoft.com/fwlink/?LinkId=615771).

## <a name="add-your-custom-domain-name-to-azure-ad"></a>Adición del nombre de dominio personalizado en Azure AD

Después de crear el directorio, puede agregar el nombre de dominio personalizado.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con una cuenta de administrador global para el directorio.

1. Busque y seleccione *Azure Active Directory* en cualquier página. Luego, seleccione **Nombres de dominio personalizado** > **Agregar dominio personalizado**.

    ![Página de nombres de dominio personalizados, con Agregar un dominio personalizado](media/add-custom-domain/add-custom-domain.png)

1. En **Nombre de dominio personalizado**, escriba el nombre nuevo de la organización, en este ejemplo, *contoso.com*. Seleccione **Add domain** (Agregar dominio).

    ![Página de nombres de dominio personalizados, con la página Agregar un dominio personalizado](media/add-custom-domain/add-custom-domain-blade.png)

    >[!IMPORTANT]
    >Debe incluir *.com*, *.net* o cualquier otra extensión de nivel superior para un correcto funcionamiento.

    Se agrega el dominio sin comprobar. Aparece la página **contoso.com**, donde se muestra la información de DNS. Guarde esta información. La necesitará más adelante para crear un registro TXT y configurar DNS.

    ![Página de Contoso con la información de la entrada de DNS](media/add-custom-domain/contoso-blade-with-dns-info.png)

## <a name="add-your-dns-information-to-the-domain-registrar"></a>Adición de la información de DNS en el registrador de dominios

Después de agregar el nombre de dominio personalizado a Azure AD, debe volver al registrador de dominios y agregar la información de DNS de Azure AD desde el archivo TXT copiado. La creación de este registro TXT en el dominio comprueba la propiedad del nombre de dominio.

Vuelva al registrador de dominios y cree un nuevo registro TXT para el dominio, según la información de DNS copiada. Establezca el período de vida (TTL) en 3600 segundos (60 minutos) y, luego, guarde el registro.

>[!IMPORTANT]
>Puede registrar tantos nombres de dominio como desee. Sin embargo, cada dominio obtiene su propio registro TXT de Azure AD. Tenga cuidado al escribir la información del archivo TXT en el registrador de dominios. Si escribe por error información incorrecta o duplicada, deberá esperar hasta que expire el TTL (60 minutos) antes de volver a intentarlo.

## <a name="verify-your-custom-domain-name"></a>Comprobación del nombre de dominio personalizado

Después de registrar el nombre de dominio personalizado, asegúrese de que es válido en Azure AD. La propagación desde el registrador de dominios a Azure AD puede ser instantánea o puede tardar unos días, dependiendo del registrador de dominios.

Siga estos pasos para comprobar el nombre de dominio personalizado:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con una cuenta de administrador global para el directorio.

1. Busque y seleccione *Azure Active Directory* en cualquier página y, luego, seleccione **Nombres de dominio personalizado**.

1. En **Nombres de dominio personalizado**, seleccione el nombre de dominio personalizado. En este ejemplo, seleccione **contoso.com**.

    ![Fabrikam: página de nombres de dominio personalizados con Contoso resaltado](media/add-custom-domain/custom-blade-with-contoso-highlighted.png)

1. En la página **contoso.com**, seleccione **Comprobar** para asegurarse de que el dominio personalizado se ha registrado correctamente y es válido para Azure AD.

    ![Página de Contoso con la información de la entrada de DNS y el botón Comprobar](media/add-custom-domain/contoso-blade-with-dns-info-verify.png)

Después de comprobar su nombre de dominio personalizado, puede eliminar el archivo TXT o MX de comprobación.

## <a name="common-verification-issues"></a>Problemas comunes de comprobación

Si Azure AD no puede comprobar un nombre de dominio personalizado, pruebe las sugerencias siguientes:

- **Espere al menos una hora y vuelva a intentarlo**. Los registros de DNS deben propagarse antes de que Azure AD compruebe el dominio. Este proceso puede tardar una hora o más.

- **Asegúrese de que el registro de DNS es correcto.** Vuelva al sitio del registrador de nombres de dominio. Asegúrese de que exista la entrada y que coincida con la información de la entrada DNS proporcionada por Azure AD.

  Si no puede actualizar el registro en el sitio del registrador, comparta la entrada con alguien que tenga los permisos para agregar la entrada y comprobar que es correcta.

- **Asegúrese de que el nombre de dominio no está en uso en otro directorio.** Un nombre de dominio solo se puede comprobar en un directorio. Si el nombre de dominio se comprueba actualmente en otro directorio, no se puede comprobar también en el directorio nuevo. Para corregir este problema de duplicación, debe eliminar el nombre de dominio en el directorio antiguo. Para más información sobre la eliminación de nombres de dominio, consulte [Administración de nombres de dominio personalizados](../users-groups-roles/domains-manage.md).

- **Asegúrese de que no tiene ningún inquilino de Power BI no administrado.** Si los usuarios han activado Power BI a través del registro de autoservicio y han creado a un inquilino no administrado para la organización, debe asumir la administración como administrador interno o externo, mediante PowerShell. Para más información, vea [Adquisición de un directorio no administrado como administrador en Azure Active Directory](../users-groups-roles/domains-admin-takeover.md).

## <a name="next-steps"></a>Pasos siguientes

- Agregar otro administrador global al directorio. Para más información, consulte [Asignación de roles y administradores](active-directory-users-assign-role-azure-portal.md).

- Agregue usuarios a su dominio. Para más información, consulte [Incorporación o eliminación de usuarios](add-users-azure-active-directory.md).

- Administrar la información del nombre de dominio en Azure AD. Para más información, consulte [Administración de nombres de dominio personalizados](../users-groups-roles/domains-manage.md).

- Si tiene versiones locales de Windows Server que desea usar junto con Azure Active Directory, consulte [Integración de los directorios locales con Azure Active Directory](../connect/active-directory-aadconnect.md).
