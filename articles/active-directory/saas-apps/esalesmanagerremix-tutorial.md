---
title: 'Tutorial: Integración de Azure Active Directory con E Sales Manager Remix | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y E Sales Manager Remix.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 06/12/2018
ms.author: jeedes
ms.openlocfilehash: dd5a200d1c17f0417b2ee88f7f905ee6de3cfe62
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88555396"
---
# <a name="integrate-azure-active-directory-with-e-sales-manager-remix"></a>Integración de Azure Active Directory con E Sales Manager Remix

En este tutorial, aprenderá a integrar Azure Active Directory (Azure AD) con E Sales Manager Remix.

La integración de Azure AD con E Sales Manager Remix proporciona las siguientes ventajas:

- Puede controlar en Azure AD quién tiene acceso a E Sales Manager Remix.
- Puede permitir que los usuarios inicien sesión automáticamente en E Sales Manager Remix (inicio de sesión único o SSO) con sus cuentas de Azure AD.
- Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con E Sales Manager Remix, necesita los siguientes elementos:

- Una suscripción de Azure AD
- Una suscripción habilitada para el inicio de sesión único en E Sales Manager Remix

> [!NOTE]
> *No* se recomienda usar un entorno de producción para probar los pasos de este tutorial.

Para probar los pasos de este tutorial, siga estas recomendaciones:

- No use el entorno de producción, salvo que sea necesario.
- Si no dispone de un entorno de prueba de Azure AD, puede [obtener una versión de prueba durante un mes](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Descripción del escenario
En este tutorial, puede probar el inicio de sesión único de Azure AD en un entorno de prueba. 

El escenario descrito en este tutorial consta de dos bloques de creación principales:

* Adición de E Sales Manager Remix desde la galería
* Configuración y comprobación del inicio de sesión único de Azure AD

## <a name="add-e-sales-manager-remix-from-the-gallery"></a>Adición de E Sales Manager Remix desde la galería
Para configurar la integración de Azure AD con E Sales Manager Remix, será preciso que haga lo siguiente para agregar E Sales Manager Remix desde la galería a la lista de aplicaciones SaaS administradas:

1. En el panel izquierdo de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**. 

    ![Botón Azure Active Directory][1]

1. Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Ventana Aplicaciones empresariales][2]
    
1. Para agregar una aplicación nueva, en la parte superior de la ventana, seleccione **Nueva aplicación**.

    ![Botón Nueva aplicación][3]

1. En el cuadro de búsqueda, escriba **E Sales Manager Remix**, seleccione **E Sales Manager Remix** en la lista de resultados y, a continuación, seleccione **Agregar**.

    ![E Sales Manager Remix en la lista de resultados](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con E Sales Manager Remix utilizando un usuario de prueba llamado "Britta Simon".

Para que el inicio de sesión único funcione, Azure AD necesita identificar al usuario de E Sales Manager Remix y a su homólogo en Azure AD. Es decir, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el mismo usuario de E Sales Manager Remix.

Para configurar y probar el inicio de sesión único de Azure AD con E Sales Manager Remix, es preciso completar los bloques de creación que se detallan en las siguientes cinco secciones:

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

Haga lo siguiente para habilitar el inicio de sesión único de Azure AD en Azure Portal y configurarlo en la aplicación E Sales Manager Remix:

1. En Azure Portal, en la página de integración de aplicaciones **E Sales Manager Remix**, seleccione **Inicio de sesión único**.

    ![Vínculo Inicio de sesión único][4]

1. En la ventana **Inicio de sesión único**, en el cuadro **Modo de inicio de sesión único**, seleccione **Inicio de sesión basado en SAML**.
 
    ![Ventana Inicio de sesión único](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_samlbase.png)

1. En **E Sales Manager Remix Domain and URLs** (Dominio y direcciones URL de E sales Manager Remix), haga lo siguiente:

    ![Información de dominio y direcciones URL de inicio de sesión único de E Sales Manager Remix](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_url.png)

    a. En el cuadro **Dirección URL de inicio de sesión**, escriba una dirección URL con el siguiente formato: *https://\<Server-Based-URL>/\<sub-domain>/esales-pc*.

    b. En el cuadro **Identificador**, escriba una dirección URL con el siguiente formato: *https://\<Server-Based-URL>/\<sub-domain>/* .

    c. Anote el valor del **identificador** para usarlo más adelante en este tutorial.
    
    > [!NOTE] 
    > Los valores anteriores no son reales. Debe actualizarlos con la dirección URL y el identificador reales de inicio de sesión. Para obtener estos valores, póngase en contacto con el [equipo de soporte al cliente de E Sales Manager Remix](mailto:esupport@softbrain.co.jp).

1. En **Certificado de firma de SAML**, seleccione **Certificado (Base64)** y, luego, guarde el archivo del certificado en el equipo.

    ![Vínculo de descarga del certificado (Base64)](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_certificate.png) 

1. Seleccione la casilla **View and edit all other user attributes** (Ver y editar todos los demás atributos de usuario) y seleccione el atributo **emailaddress**.
    
    ![Ventana Atributos de usuario](./media/esalesmanagerremix-tutorial/configure1.png)

    Se abre la ventana **Edit Attribute** (Editar atributo).

1. Copie los valores de **Namespace** (Espacio de nombres) y **Name** (Nombre). Genere el valor con el patrón *\<Namespace>/\<Name>* y guárdelo para usarlo más adelante en este tutorial.

    ![Ventana Edit Attribute](./media/esalesmanagerremix-tutorial/configure2.png)

1. En **E Sales Manager Remix Configuration** (Configuración de E Sales Manager Remix), seleccione **Configure E Sales Manager Remix** (Configurar E Sales Manager Remix).

    ![Configuración de E Sales Manager Remix](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_configure.png) 

    Se abre la ventana **Configure sign-on** (Configurar inicio de sesión).

1. En la sección **Quick Reference** (Referencia rápida), copie la dirección URL de cierre de sesión y la dirección URL del servicio de inicio de sesión único SAML.

1. Seleccione **Guardar**.

    ![Botón Guardar](./media/esalesmanagerremix-tutorial/tutorial_general_400.png)

1. Inicie sesión en la aplicación E Sales Manager Remix como administrador.

1. En la parte superior derecha, Seleccione **To Administrator Menu** (Menú para administrador).

    ![Comando "To Administrator Menu"](./media/esalesmanagerremix-tutorial/configure4.png)

1. En el panel izquierdo, seleccione **System settings** (Configuración del sistema) > **Cooperation with external system** (Cooperación con sistema externo).

    ![Vínculos a "System settings" y "Cooperation with external system"](./media/esalesmanagerremix-tutorial/configure5.png)
    
1. En la ventana **Cooperation with external system**, seleccione **SAML**.

    ![Ventana "Cooperation with external system"](./media/esalesmanagerremix-tutorial/configure6.png)

1. En **SAML authentication setting** (Configuración de la autenticación de SAML), haga lo siguiente:

    ![Sección "SAML authentication setting"](./media/esalesmanagerremix-tutorial/configure3.png)
    
    a. Seleccione la casilla **PC version** (Versión de PC).
    
    b. En la sección **Collaboration item** (Elemento de colaboración), en la lista desplegable, seleccione **email**.

    c. En el cuadro de texto **Collaboration item** (Elemento de colaboración), pegue el valor de notificación que ha copiado de Azure Portal (es decir, **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`** ).

    d. En el cuadro de texto **Issuer (entity ID)** (Emisor [Id. de entidad]), pegue el valor del identificador que copió de la sección **E Sales Manager Remix Domain and URLs** (Dominio y direcciones URL de E Sales Manager Remix) de Azure Portal.

    e. Para cargar el certificado descargado de Azure Portal, seleccione **File selection** (Selección de archivo).

    f. En el cuadro **ID provider login URL** (Dirección URL de inicio de sesión del proveedor de identidades), pegue la dirección URL del servicio de inicio de sesión único de SAML que copió de Azure Portal.

    g. En el cuadro **Identity Provider Logout URL** (Dirección URL de cierre de sesión del proveedor de identidades), pegue el valor de la dirección URL de cierre de sesión que copió de Azure Portal.

    h. Seleccione **Setting complete** (Configuración completada).

> [!TIP]
> Mientras configure la aplicación, puede leer una versión concisa de las instrucciones anteriores en [Azure Portal](https://portal.azure.com). Después de agregar la aplicación desde la sección **Active Directory** > **Aplicaciones empresariales**, seleccione la pestaña **Inicio de sesión único** y, después, acceda a la documentación insertada a través de la sección **Configuración** en la parte inferior. Para más información sobre la característica de documentación insertada, consulte la [documentación insertada de Azure AD]( https://go.microsoft.com/fwlink/?linkid=845985).
> 

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, creará el usuario de prueba Britta Simon en Azure Portal siguiendo estos pasos:

![Creación de un usuario de prueba de Azure AD][100]

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**.

    ![Vínculo de Azure Active Directory](./media/paloaltoadmin-tutorial/create_aaduser_01.png)

1. Para mostrar una lista de usuarios actuales, seleccione **Usuarios y grupos** > **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](./media/paloaltoadmin-tutorial/create_aaduser_02.png)

1. En la parte superior de la ventana **Todos los usuarios**, seleccione **Agregar**.

    ![Botón Agregar](./media/paloaltoadmin-tutorial/create_aaduser_03.png)
    
    Se abre la ventana **Usuario**.

1. En la ventana **Usuario**, haga lo siguiente:

    ![Ventana Usuario](./media/paloaltoadmin-tutorial/create_aaduser_04.png)

    a. En el cuadro **Nombre**, escriba **BrittaSimon**.

    b. En el cuadro de texto **Nombre de usuario**, escriba la dirección de correo electrónico del usuario Britta Simon.

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.

    d. Seleccione **Crear**.
 
### <a name="create-an-e-sales-manager-remix-test-user"></a>Creación de un usuario de prueba de E Sales Manager Remix

1. Inicie sesión en la aplicación E Sales Manager Remix como administrador.

1. Seleccione **To Administrator Menu** (Menú para administrador) en el menú de la esquina superior derecha.

    ![Configuración de E Sales Manager Remix](./media/esalesmanagerremix-tutorial/configure4.png)

1. Seleccione **Your company settings** (Configuración de su empresa) > **Maintenance of departments and employees** (Mantenimiento de departamentos y empleados) y después seleccione **Employees registered** (Empleados registrados).

    ![Pestaña "Employees registered"](./media/esalesmanagerremix-tutorial/user1.png)

1. En la sección **New employee registration** (Nuevo registro de empleado), haga lo siguiente:
    
    ![Sección "New employee registration"](./media/esalesmanagerremix-tutorial/user2.png)

    a. En el cuadro **Employee Name** (Nombre de empleado), escriba el nombre del usuario (por ejemplo, **Britta**).

    b. Rellene los demás campos necesarios.
    
    c. Si habilita SAML, el administrador no puede iniciar sesión desde la página de inicio de sesión. Seleccione la casilla **Admin Login** (Inicio de sesión de administrador) para conceder al usuario privilegios de inicio de sesión de administrador.

    d. Seleccione **Registration** (Registro).

1. Después, para iniciar sesión como administrador, inicie sesión como el usuario que tenga los permisos de administrador y, a continuación, en la esquina superior derecha, seleccione **To Administrator Menu** (Menú para administrador).

    ![Comando "To Administrator Menu"](./media/esalesmanagerremix-tutorial/configure4.png)

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, concederá a la usuaria Britta Simon acceso a E Sales Manager Remix pera permitirle que use el inicio de sesión único de Azure. Para ello, haga lo siguiente: 

![Asignación de rol de usuario][200] 

1. En Azure Portal, abra la vista **Aplicaciones**, vaya a la vista **Directorio** y seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Vínculos "Aplicaciones empresariales" y "Todas las aplicaciones"][201] 

1. En la lista **Aplicaciones**, seleccione **E Sales Manager Remix**.

    ![Vínculo de E Sales Manager Remix](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_app.png)  

1. En el panel izquierdo, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"][202]

1. Seleccione **Agregar** y, después, en el panel **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Panel Agregar asignación][203]

1. En la ventana **Usuarios y grupos**, en la lista **Usuarios**, seleccione **Britta Simon**.

1. Seleccione el botón **Seleccionar**.

1. En la ventana **Agregar asignación**, seleccione **Asignar**.
    
### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de E Sales Manager Remix en el panel de acceso, debería iniciar sesión automáticamente en la aplicación E Sales Manager Remix.

Para más información sobre el Panel de acceso, consulte la [introducción al Panel de acceso](../user-help/active-directory-saas-access-panel-introduction.md). 

## <a name="additional-resources"></a>Recursos adicionales

* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](tutorial-list.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/esalesmanagerremix-tutorial/tutorial_general_01.png
[2]: ./media/esalesmanagerremix-tutorial/tutorial_general_02.png
[3]: ./media/esalesmanagerremix-tutorial/tutorial_general_03.png
[4]: ./media/esalesmanagerremix-tutorial/tutorial_general_04.png

[100]: ./media/esalesmanagerremix-tutorial/tutorial_general_100.png

[200]: ./media/esalesmanagerremix-tutorial/tutorial_general_200.png
[201]: ./media/esalesmanagerremix-tutorial/tutorial_general_201.png
[202]: ./media/esalesmanagerremix-tutorial/tutorial_general_202.png
[203]: ./media/esalesmanagerremix-tutorial/tutorial_general_203.png

