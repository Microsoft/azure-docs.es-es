---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con EBSCO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y EBSCO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/11/2019
ms.author: jeedes
ms.openlocfilehash: 2395f3c8b46f69105a81cd2d866ee4e330f4791e
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88555508"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ebsco"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con EBSCO

En este tutorial aprenderá a integrar EBSCO con Azure Active Directory (Azure AD). Al integrar EBSCO con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a EBSCO.
* Permitir que los usuarios inicien sesión automáticamente en EBSCO con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en EBSCO.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* EBSCO admite el inicio de sesión único iniciado por **SP e IDP**.
* EBSCO admite el aprovisionamiento de usuarios **Just-In-Time**

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-ebsco-from-the-gallery"></a>Adición de EBSCO desde la galería

Para configurar la integración de EBSCO en Azure AD, será preciso agregar EBSCO desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **EBSCO** en el cuadro de búsqueda.
1. Seleccione **EBSCO** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-ebsco"></a>Configuración y prueba del inicio de sesión único de Azure AD para EBSCO

Configure y pruebe el inicio de sesión único de Azure AD con EBSCO utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de EBSCO.

Para configurar y probar el inicio de sesión único de Azure AD con EBSCO, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de EBSCO](#configure-ebsco-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación del usuario de prueba en EBSCO](#create-ebsco-test-user)** , para tener un homólogo de B.Simon en EBSCO que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **EBSCO**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    En el cuadro de texto **Identificador**, escriba una dirección URL: `pingsso.ebscohost.com`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `http://search.ebscohost.com/login.aspx?authtype=sso&custid=<unique EBSCO customer ID>&profile=<profile ID>`

    > [!NOTE]
    > El valor de la dirección URL de inicio de sesión no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico para clientes de EBSCO](mailto:support@ebsco.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

    o   **Elementos únicos:**  

    o   **Custid** = escriba el identificador de cliente único de EBSCO 

    o   **Profile** = los clientes pueden personalizar el vínculo para dirigir a los usuarios a un perfil específico (en función de lo que compran en EBSCO). Pueden especificar un identificador de perfil específico. Los identificadores principales son eds (servicio de detección de EBSCO) y ehost (bases de datos EBSOCOhost). Se proporcionan [aquí](https://help.ebsco.com/interfaces/EBSCOhost/EBSCOhost_FAQs/How_do_I_set_up_direct_links_to_EBSCOhost_profiles_and_or_databases#profile) instrucciones.

1. EBSCO espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

    > [!Note]
    > El atributo **name** es obligatorio y se asigna con el valor del **identificador de nombre** en la aplicación EBSCO. Se agrega de forma predeterminada, por lo que no es necesario agregarla manualmente.

1. Además de lo anterior, la aplicación EBSCO espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen|
    | ---------------| --------------- |
    | Nombre   | user.givenname |
    | Apellidos   | user.surname |
    | Email   | user.mail |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar EBSCO**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a B.Simon para que use el inicio de sesión único de Azure concediéndole acceso a EBSCO.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **EBSCO**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-ebsco-sso"></a>Configuración del inicio de sesión único de EBSCO

Para configurar el inicio de sesión único en **EBSCO**, es preciso enviar el **XML de metadatos de federación** descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de EBSCO](mailto:support@ebsco.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-ebsco-test-user"></a>Creación del usuario de prueba en EBSCO

En el caso de EBSCO, el aprovisionamiento de usuarios es automática.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

Azure AD pasa los datos necesarios a la aplicación de EBSCO. El aprovisionamiento de usuarios de EBSCO puede ser automático O requerir un formulario de un solo uso. Depende de si el cliente tiene muchas cuentas EBSCOhost preexistentes con configuraciones personales guardadas. Esto mismo puede analizarse con el [equipo de soporte técnico de EBSCO](mailto:support@ebsco.com) durante la implementación. En cualquier caso, el cliente no tiene que crear las cuentas de EBSCOhost antes de la prueba.

   > [!Note]
   > Puede automatizar el aprovisionamiento o la personalización de usuarios de EBSCOhost. Póngase en contacto con el [equipo de soporte técnico de EBSCO](mailto:support@ebsco.com) sobre el aprovisionamiento de usuarios Just-In-Time.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

1. Al hacer clic en el icono de EBSCO en el Panel de acceso, debería iniciar sesión automáticamente en su aplicación EBSCO.
Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](../user-help/active-directory-saas-access-panel-introduction.md).

1. Cuando inicie sesión en la aplicación, haga clic en el botón **Iniciar sesión** en la esquina superior derecha.

    ![Inicio de sesión en EBSCO en la lista de aplicaciones](./media/ebsco-tutorial/tutorial_ebsco_signin.png)

1. Recibirá un aviso único para emparejar el inicio de sesión institucional o de SAML y **enlazar la cuenta MyEBSCOhost existente a la cuenta de su institución ahora** O BIEN para **crear una nueva cuenta MyEBSCOhost y vincularla a la cuenta de su institución**. La cuenta se utiliza para la personalización en la aplicación EBSCOhost. Seleccione la opción **Crear una cuenta** y verá que el formulario de personalización está completado previamente con los valores de la respuesta saml, tal como se muestra en la captura de pantalla siguiente. Haga clic en **'Continuar'** para guardar esta selección.
    
     ![Usuario de EBSCO en la lista de aplicaciones](./media/ebsco-tutorial/tutorial_ebsco_user.png)

1. Después de completar la instalación anterior, limpie las cookies y la memoria caché y vuelva a iniciar sesión. No tendrá que iniciar sesión manualmente de nuevo ya que la configuración de personalización se recuerda.

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe EBSCO con Azure AD](https://aad.portal.azure.com/)