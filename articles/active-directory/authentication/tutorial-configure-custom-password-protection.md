---
title: Configuración de listas personalizadas de protección de contraseñas de Azure Active Directory
description: En este tutorial, aprenderá a configurar listas personalizadas de protección de contraseñas prohibidas de Azure Active Directory para restringir palabras comunes en su entorno.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: tutorial
ms.date: 07/13/2020
ms.author: iainfou
author: iainfoulds
ms.reviewer: rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: 24e0cfa4227ba5d7c38449d3cf1304add5ae5c1f
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88718890"
---
# <a name="tutorial-configure-custom-banned-passwords-for-azure-active-directory-password-protection"></a>Tutorial: Configuración de contraseñas prohibidas personalizadas para la protección con contraseña de Azure Active Directory

Los usuarios suelen crear contraseñas que usan palabras comunes locales, como una escuela, un equipo deportivo o una persona famosa. Estas contraseñas son fáciles de adivinar y poco seguras contra ataques basados en diccionarios. Para aplicar contraseñas seguras en su organización, la lista personalizada de contraseñas prohibidas de Azure Active Directory (Azure AD) permite agregar cadenas específicas para evaluar y bloquear. Se produce un error en una solicitud de cambio de contraseña si hay una coincidencia en la lista personalizada de contraseñas prohibidas.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Habilitar contraseñas prohibidas personalizadas
> * Agregar entradas a la lista personalizada de contraseñas prohibidas
> * Probar los cambios de contraseña con una contraseña prohibida

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesitará los siguientes recursos y privilegios:

* Un inquilino de Azure AD activo con al menos una licencia de Azure AD Premium P1 o de prueba habilitada.
    * Si es preciso, [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Una cuenta con privilegios de *administrador global*.
* Un usuario que no sea administrador con una contraseña que conozca, como *usuarioDePrueba*. Para probar un evento de cambio de contraseña, use esta cuenta en este tutorial.
    * Si necesita crear un usuario, consulte [Inicio rápido: Incorporación de nuevos usuarios a Azure Active Directory](../fundamentals/add-users-azure-active-directory.md).
    * Para probar una operación de cambio de contraseña con una contraseña prohibida, el inquilino de Azure AD debe estar [configurado para el autoservicio de restablecimiento de contraseña](tutorial-enable-sspr.md).

## <a name="what-are-banned-password-lists"></a>¿Qué son las listas de contraseñas prohibidas?

Azure AD incluye una lista global de contraseñas prohibidas. El contenido de la lista global de contraseñas prohibidas no se basa en ningún origen de datos externo, sino que se basa en los resultados continuos de la telemetría y el análisis de seguridad de Azure AD. Cuando un usuario o administrador intenta cambiar o restablecer sus credenciales, se comprueba la contraseña deseada en la lista de contraseñas prohibidas. Se produce un error en la solicitud de cambio de contraseña si hay una coincidencia en la lista global de contraseñas prohibidas. No se puede editar esta lista global de contraseñas prohibidas predeterminada.

Para que tenga flexibilidad en cuanto a las contraseñas que se permiten, también puede definir una lista personalizada de contraseñas prohibidas. La lista personalizada de contraseñas prohibidas funciona junto con la lista global de contraseñas prohibidas para aplicar contraseñas seguras en su organización. Se pueden agregar términos específicos de la organización a la lista personalizada de contraseñas prohibidas, como los ejemplos siguientes:

* Nombres de marca
* Nombres de producto
* Ubicaciones, por ejemplo, la oficina central de la empresa
* Términos internos específicos de la empresa
* Abreviaturas que tienen un significado específico en la empresa

Si un usuario intenta restablecer una contraseña por una que esté en la lista personalizada o global de contraseñas prohibidas, verá uno de los siguientes mensajes de error:

* *La contraseña contiene una palabra, una frase o un patrón que resultan fáciles de adivinar. Vuelva a intentarlo con otra contraseña.*
* *No puede usar esta contraseña porque contiene palabras o caracteres que el administrador ha bloqueado. Vuelva a intentarlo con otra contraseña.*

La lista personalizada de contraseñas prohibidas se limita a un máximo de 1000 términos. No está diseñada para bloquear listas grandes de contraseñas. Para maximizar las ventajas de la lista personalizada de contraseñas prohibidas, consulte los [conceptos de la lista personalizada de contraseñas prohibidas](concept-password-ban-bad.md#custom-banned-password-list) y la [introducción al algoritmo de evaluación de contraseñas](concept-password-ban-bad.md#how-are-passwords-evaluated).

## <a name="configure-custom-banned-passwords"></a>Configuración de contraseñas prohibidas personalizadas

Vamos a habilitar la lista personalizada de contraseñas prohibidas y agregar algunas entradas. Puede agregar entradas adicionales a la lista personalizada de contraseñas prohibidas en cualquier momento.

Para habilitar la lista personalizada de contraseñas prohibidas y agregar entradas a ella, complete los pasos siguientes:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) mediante una cuenta con permisos de *administrador global*.
1. Busque y seleccione **Azure Active Directory** y, a continuación, elija **Seguridad** en el menú del lado izquierdo.
1. En el encabezado del menú **Administrar**, seleccione **Métodos de autenticación** y, a continuación, **Protección con contraseña**.
1. En **Exigir lista personalizada**, seleccione la opción *Sí*.
1. Agregue cadenas a la **Lista personalizada de contraseñas prohibidas**, una cadena por línea. Las siguientes consideraciones y limitaciones se aplican a la lista personalizada de contraseñas prohibidas:

    * La lista personalizada de contraseñas prohibidas puede contener hasta 1000 términos.
    * La lista personalizada de contraseñas prohibidas distingue mayúsculas de minúsculas.
    * La lista personalizada de contraseñas prohibidas tiene en cuenta la sustitución de caracteres comunes, como "o" y "0", o "a" y "\@".
    * La longitud mínima de la cadena es de cuatro caracteres, mientras que la máxima es de 16 caracteres.

    Especifique sus propias contraseñas personalizadas para prohibirlas, tal como se muestra en el ejemplo siguiente.

    [ ![Modifique la lista personalizada de contraseñas prohibidas en Métodos de autenticación de Azure Portal](media/tutorial-configure-custom-password-protection/enable-configure-custom-banned-passwords-cropped.png) ](media/tutorial-configure-custom-password-protection/enable-configure-custom-banned-passwords.png#lightbox)

1. En **Habilitar protección con contraseña en Windows Server Active Directory**, establezca la opción en *No*.
1. Para habilitar las contraseñas prohibidas personalizadas y las entradas, seleccione **Guardar**.

Las actualizaciones a la lista personalizada de contraseñas prohibidas pueden tardar varias horas en aplicarse.

Para un entorno híbrido, también puede [implementar la protección con contraseña de Azure AD en un entorno local](howto-password-ban-bad-on-premises-deploy.md). Se usan las mismas listas personalizadas y globales de contraseñas prohibidas para las solicitudes de cambio de contraseña en la nube y en el entorno local.

## <a name="test-custom-banned-password-list"></a>Prueba de la lista personalizada de contraseñas prohibidas

Para ver la lista personalizada de contraseñas prohibidas en acción, intente cambiar la contraseña por una variación de una que haya agregado en la sección anterior. Cuando Azure AD intenta procesar el cambio de contraseña, la contraseña se compara con una entrada de la lista personalizada de contraseñas prohibidas. A continuación, se muestra un error al usuario.

> [!NOTE]
> Para que un usuario pueda restablecer su contraseña en el portal basado en web, el inquilino de Azure AD debe estar [configurado para el autoservicio de restablecimiento de contraseña](tutorial-enable-sspr.md). Si es necesario, el usuario puede entonces [registrarse para SSPR en https://aka.ms/ssprsetup](https://aka.ms/ssprsetup).

1. Vaya a la página **Mis aplicaciones** en [https://myapps.microsoft.com](https://myapps.microsoft.com).
1. En la esquina superior derecha, seleccione su nombre y, a continuación, elija **Perfil** en el menú desplegable.

    ![Seleccionar perfil](media/tutorial-configure-custom-password-protection/myapps-profile.png)

1. En la página **Perfil**, seleccione **Cambiar contraseña**.
1. En la página **Cambiar contraseña**, escriba la contraseña existente (anterior). Escriba y confirme una nueva contraseña que se encuentre en la lista personalizada de contraseñas prohibidas definida en la sección anterior y, a continuación, seleccione **Enviar**.
1. Se devuelve un mensaje de error que indica que el administrador ha bloqueado la contraseña, tal como se muestra en el ejemplo siguiente:

    ![Aparece un mensaje de error al intentar usar una contraseña que forma parte de la lista personalizada de contraseñas prohibidas.](media/tutorial-configure-custom-password-protection/password-change-error.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si decide que ya no desea utilizar la lista personalizada de contraseñas prohibidas que ha configurado como parte de este tutorial, realice los siguientes pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque y seleccione **Azure Active Directory** y, a continuación, elija **Seguridad** en el menú del lado izquierdo.
1. En el encabezado del menú **Administrar**, seleccione **Métodos de autenticación** y, a continuación, **Protección con contraseña**.
1. En **Exigir lista personalizada**, seleccione la opción *No*.
1. Para actualizar la configuración personalizada de contraseñas prohibidas, seleccione **Guardar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha habilitado y configurado listas personalizadas de protección de contraseñas para Azure AD. Ha aprendido a:

> [!div class="checklist"]
> * Habilitar contraseñas prohibidas personalizadas
> * Agregar entradas a la lista personalizada de contraseñas prohibidas
> * Probar los cambios de contraseña con una contraseña prohibida

> [!div class="nextstepaction"]
> [Habilitación de la autenticación multifactor de Azure en función del riesgo](./tutorial-enable-azure-mfa.md)