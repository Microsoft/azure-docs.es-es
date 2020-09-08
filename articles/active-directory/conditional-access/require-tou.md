---
title: 'El acceso condicional requiere condiciones de uso: Azure Active Directory'
description: En este inicio rápido, aprenderá a exigir la aceptación de las condiciones de uso antes de que se conceda acceso a aplicaciones en la nube seleccionadas mediante el acceso condicional de Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: quickstart
ms.date: 11/21/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 53f99456449e778ff7c1fd3ab096e1afd3562c1e
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88948868"
---
# <a name="quickstart-require-terms-of-use-to-be-accepted-before-accessing-cloud-apps"></a>Inicio rápido: Solicitud de la aceptación de los términos de uso antes de acceder a aplicaciones en la nube

Antes de acceder a determinadas aplicaciones en la nube de su entorno, puede obtener el consentimiento de los usuarios en forma de aceptación de las condiciones de uso (CDU). El acceso condicional de Azure Active Directory (Azure AD) proporciona:

- Un método sencillo para configurar las condiciones de uso
- La opción para solicitar la aceptación de las condiciones de uso mediante una directiva de acceso condicional  

En este inicio rápido se muestra cómo configurar una [directiva de acceso condicional de Azure AD](./overview.md) que exija la aceptación de unas condiciones de uso para una aplicación en la nube seleccionada de su entorno.

![Creación de directiva](./media/require-tou/5555.png)

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar el escenario en este inicio rápido, necesita:

- **Acceso a una edición de Azure AD Premium**: el acceso condicional de Azure AD es una funcionalidad de Azure AD Premium.
- **Una cuenta de prueba denominada Isabella Simonsen**: si no sabe cómo crear una cuenta de prueba, consulte [Agregar usuarios basados en la nube](../fundamentals/add-users-azure-active-directory.md#add-a-new-user).

## <a name="test-your-sign-in"></a>Prueba del inicio de sesión

El objetivo de este paso es obtener una impresión de la experiencia de inicio de sesión sin una directiva de acceso condicional.

**Para probar el inicio de sesión:**

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como Isabella Simonsen.
1. Cierre la sesión.

## <a name="create-your-terms-of-use"></a>Creación de condiciones de uso

En esta sección se explican los pasos necesarios para crear unas condiciones de uso de ejemplo. Cuando cree unas condiciones de uso, seleccione el valor de **Exigir con plantillas de directiva de acceso condicional**. Al seleccionar **Directiva personalizada** se abre el cuadro de diálogo para crear una directiva de acceso condicional en cuanto se hayan creado las condiciones de uso.

**Para crear las condiciones de uso:**

1. En Microsoft Word, cree un nuevo documento.
1. Escriba **Mis condiciones de uso**y guárdelo en el equipo como **mytou.pdf**.
1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global, administrador de seguridad o administrador de acceso condicional.
1. En Azure Portal, en la barra de navegación izquierda, haga clic en **Azure Active Directory**.

   ![Azure Active Directory](./media/require-tou/02.png)

1. En la página **Azure Active Directory**, en la sección **Seguridad**, haga clic en **Acceso condicional**.

   ![Acceso condicional](./media/require-tou/03.png)

1. En la sección **Administrar**, haga clic en **Condiciones de uso**.

   ![Términos de uso](./media/require-tou/04.png)

1. En el menú de la parte superior, haga clic en **Nuevos términos**.

   ![Términos de uso](./media/require-tou/05.png)

1. En la página **Nuevos términos de uso**:

   ![Términos de uso](./media/require-tou/112.png)

   1. En el cuadro de texto **Nombre**, escriba un **Mis CDU**.
   1. En el cuadro de texto **Nombre para mostrar**, escriba **Mis CDU**.
   1. Cargue el archivo PDF de las condiciones de uso.
   1. En **Idioma**, seleccione **Inglés**.
   1. En **Requerir a los usuarios que expandan las condiciones de uso**, seleccione **Activado**.
   1. En **Exigir con plantillas de directiva de acceso condicional**, seleccione **Directiva personalizada**.
   1. Haga clic en **Crear**.

## <a name="create-your-conditional-access-policy"></a>Creación de la directiva de acceso condicional

En esta sección se muestra cómo crear la directiva de acceso condicional necesaria. En el escenario de esta guía de inicio rápido se usa:

- Azure Portal como marcador de posición para las aplicaciones en la nube que requieran que se acepte el CDU. 
- El usuario de ejemplo para probar la directiva de acceso condicional.  

En la directiva, establezca:

| Configuración | Value |
| --- | --- |
| Usuarios y grupos | Isabella Simonsen |
| Aplicaciones de nube | Microsoft Azure Management (Administración de Microsoft Azure) |
| Conceder acceso | Mi CDU |

![Creación de directiva](./media/require-tou/1234.png)

**Para configurar la directiva de acceso condicional:**

1. En la página **Nuevo**, en el cuadro de texto **Nombre**, escriba **Requerir CDU para Isabella**.

   ![Nombre](./media/require-tou/71.png)

1. En la sección **Asignación**, haga clic en **Usuarios y grupos**.

   ![Usuarios y grupos](./media/require-tou/06.png)

1. En la página **Usuarios y grupos**:

   ![Usuarios y grupos](./media/require-tou/24.png)

   1. Haga clic en **Seleccionar usuarios y grupos** y, luego, seleccione **Usuarios y grupos**.
   1. Haga clic en **Seleccionar**.
   1. En la página **Seleccionar**, seleccione **Isabella Simonsen** y, luego, haga clic en **Seleccionar**.
   1. En la página **Usuarios y grupos**, haga clic en **Listo**.
1. Haga clic en **Aplicaciones en la nube**.

   ![Aplicaciones de nube](./media/require-tou/08.png)

1. En la página **Aplicaciones en la nube**:

   ![Seleccionar aplicaciones en la nube](./media/require-tou/26.png)

   1. Haga clic en **Seleccionar aplicaciones**.
   1. Haga clic en **Seleccionar**.
   1. En la página **Seleccionar**, seleccione **Microsoft Azure Management** (Administración de Microsoft Azure) y, luego, haga clic en **Seleccionar**.
   1. En la página **Aplicaciones en la nube**, haga clic en **Listo**.
1. En la sección **Controles de acceso**, haga clic en **Conceder**.

   ![Controles de acceso](./media/require-tou/10.png)

1. En la página **Conceder**:

   ![Conceder](./media/require-tou/111.png)

   1. Seleccione **Conceder acceso**.
   1. Seleccione **Mis CDU**.
   1. Haga clic en **Seleccionar**.
1. En la sección **Habilitar directiva**, haga clic en **Activar**.

   ![Habilitar directiva](./media/require-tou/18.png)

1. Haga clic en **Crear**.

## <a name="evaluate-a-simulated-sign-in"></a>Evaluación de un inicio de sesión simulado

Ahora que ha configurado la directiva de acceso condicional, probablemente quiera saber si funciona según lo previsto. Como primer paso, use la herramienta de directivas What If de acceso condicional para simular un inicio de sesión del usuario de prueba. La simulación calcula el impacto que este inicio de sesión tiene en las directivas y genera un informe de simulación.  

Para inicializar la herramienta de evaluación de directivas **What If**, establezca:

- **Isabella Simonsen** como usuario
- **Microsoft Azure Management** (Administración de Microsoft Azure) como aplicación en la nube

Al hacer clic en **What If**, se crea un informe de simulación que muestra:

- **Requerir CDU para Isabella** en **Directivas que se aplicarán**
- **Mi CDU** en **Conceder controles**.

![Herramienta de directivas What If](./media/require-tou/79.png)

**Para evaluar la directiva de acceso condicional:**

1. En la página [Acceso condicional - Directivas](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ConditionalAccessBlade/Policies), en el menú de la parte superior, haga clic en **What If**.  

   ![What If](./media/require-tou/14.png)

1. Haga clic en **Usuarios**, seleccione **Isabella Simonsen** y, luego, haga clic en **Seleccionar**.

   ![Usuario](./media/require-tou/15.png)

1. Para seleccionar una aplicación en la nube:

   ![Aplicaciones de nube](./media/require-tou/16.png)

   1. Haga clic en **Aplicaciones en la nube**.
   1. En la **página Aplicaciones en la nube**, haga clic en **Seleccionar aplicaciones**.
   1. Haga clic en **Seleccionar**.
   1. En la página **Seleccionar**, seleccione **Microsoft Azure Management** (Administración de Microsoft Azure) y, luego, haga clic en **Seleccionar**.
   1. En la página Aplicaciones en la nube, haga clic en **Listo**.
1. Haga clic en **What If**.

## <a name="test-your-conditional-access-policy"></a>Prueba de la directiva de acceso condicional

En la sección anterior, ha aprendido a evaluar un inicio de sesión de simulado. Además de una simulación, también debe probar la directiva de acceso condicional para asegurarse de que funciona según lo previsto.

Para probar la directiva, intente iniciar sesión en [Azure Portal](https://portal.azure.com) con su cuenta de prueba **Isabella Simonsen**. Debería ver un cuadro de diálogo que requiera que acepte las condiciones de uso.

![Términos de uso](./media/require-tou/57.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sean necesarios, elimine el usuario de prueba y la directiva de acceso condicional:

- Si no sabe cómo eliminar un usuario de Azure AD, consulte [Eliminación de usuarios desde Azure AD](../fundamentals/add-users-azure-active-directory.md#delete-a-user).
- Para eliminar la directiva, selecciónela y, a continuación, haga clic en **Eliminar** en la barra de herramientas de acceso rápido.

    ![Multi-Factor Authentication](./media/require-tou/33.png)

- Para eliminar las condiciones de uso, selecciónelo y haga clic en **Eliminar términos** en la barra de herramientas de la parte superior.

    ![Multi-Factor Authentication](./media/require-tou/29.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Requerir MFA para aplicaciones específicas](../authentication/tutorial-enable-azure-mfa.md)