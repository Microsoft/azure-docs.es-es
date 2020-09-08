---
title: 'Tutorial: Integración de Azure Active Directory con Jobscience | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Jobscience.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 07/12/2017
ms.author: jeedes
ms.openlocfilehash: 0de47cc12c461e47d8703861a99d11e7ee72d581
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88547076"
---
# <a name="tutorial-azure-active-directory-integration-with-jobscience"></a>Tutorial: Integración de Azure Active Directory con Jobscience

En este tutorial, aprenderá a integrar Jobscience con Azure Active Directory (Azure AD).

La integración de Jobscience con Azure AD proporciona las siguientes ventajas:

- En Azure AD puede controlar quién tiene acceso a Jobscience.
- Puede permitir que los usuarios inicien sesión automáticamente en Jobscience (inicio de sesión único) con sus cuentas de Azure AD.
- Puede administrar sus cuentas en una ubicación central: el nuevo Azure Portal.

Si desea saber más sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Jobscience, necesita los siguientes elementos:

- Una suscripción de Azure AD
- Una suscripción habilitada para inicio de sesión único en Jobscience

> [!NOTE]
> Para probar los pasos de este tutorial, no se recomienda el uso de un entorno de producción.

Para probar los pasos de este tutorial, debe seguir estas recomendaciones:

- No use el entorno de producción, salvo que sea necesario.
- Si no dispone de un entorno de prueba de Azure AD, puede obtener una versión de evaluación de un mes aquí: [Oferta de evaluación](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Descripción del escenario
En este tutorial, puede probar el inicio de sesión único de Azure AD en un entorno de prueba. El escenario descrito en este tutorial consta de dos bloques de creación principales:

1. Incorporación de Jobscience desde la galería
1. Configuración y comprobación del inicio de sesión único de Azure AD

## <a name="adding-jobscience-from-the-gallery"></a>Incorporación de Jobscience desde la galería
Para configurar la integración de Jobscience en Azure AD, deberá agregar Jobscience desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Jobscience desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**. 

    ![Active Directory][1]

1. Vaya a **Aplicaciones empresariales**. A continuación, vaya a **Todas las aplicaciones**.

    ![APLICACIONES][2]
    
1. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![APLICACIONES][3]

1. En el cuadro de búsqueda, escriba **Jobscience**.

    ![Creación de un usuario de prueba de Azure AD](./media/jobscience-tutorial/tutorial_jobscience_search.png)

1. En el panel de resultados, seleccione **Jobscience** y luego haga clic en el botón **Agregar** para agregar la aplicación.

    ![Creación de un usuario de prueba de Azure AD](./media/jobscience-tutorial/tutorial_jobscience_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>Configuración y comprobación del inicio de sesión único de Azure AD
En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Jobscience con un usuario de prueba llamado "Britta Simon".

Para que el inicio de sesión único funcione, Azure AD debe saber cuál es el usuario homólogo de Jobscience para un usuario de Azure AD. Es decir, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Jobscience.

Para establecer la relación de vínculo, en Jobscience, asigne el valor de **nombre de usuario** de Azure AD como valor de **Nombre de usuario**.

Para configurar y probar el inicio de sesión único de Azure AD con Jobscience, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configuring-azure-ad-single-sign-on)** : para permitir a los usuarios usar esta característica.
1. **[Creación de un usuario de prueba de Azure AD](#creating-an-azure-ad-test-user)** : para probar el inicio de sesión único de Azure AD con Britta Simon.
1. **[Creación de un usuario de prueba de Jobscience](#creating-a-jobscience-test-user)** : para tener un homólogo de Britta Simon en Jobscience que esté vinculado a la representación de ella en Azure AD.
1. **[Asignación del usuario de prueba de Azure AD](#assigning-the-azure-ad-test-user)** : para permitir que Britta Simon use el inicio de sesión único de Azure AD.
1. **[Testing Single Sign-On](#testing-single-sign-on)** : para comprobar si funciona la configuración.

### <a name="configuring-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal y lo configurará en la aplicación Jobscience.

**Para configurar el inicio de sesión único de Azure AD con Jobscience, realice los pasos siguientes:**

1. En Azure Portal, en la página de integración de la aplicación **Jobscience**, haga clic en **Inicio de sesión único**.

    ![Configurar inicio de sesión único][4]

1. En el cuadro de diálogo **Inicio de sesión único**, en **Modo** seleccione **Inicio de sesión basado en SAML** para habilitar el inicio de sesión único.
 
    ![Configurar inicio de sesión único](./media/jobscience-tutorial/tutorial_jobscience_samlbase.png)

1. En la sección **Dominio y direcciones URL de Jobscience**, lleve a cabo los pasos siguientes:

    ![Configurar inicio de sesión único](./media/jobscience-tutorial/tutorial_jobscience_url.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `http://<company name>.my.salesforce.com`
    
    > [!NOTE] 
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Puede obtener este valor con el [equipo de soporte técnico del cliente de Jobscience](http://www.jobscience.com/support) o en el perfil de SSO que va a crear, lo que se explica más adelante en el tutorial. 
 
1. En la sección **Certificado de firma de SAML**, haga clic en **Certificado (Base64)** y, luego, guarde el archivo de certificado en el equipo.

    ![Configurar inicio de sesión único](./media/jobscience-tutorial/tutorial_jobscience_certificate.png) 

1. Haga clic en el botón **Guardar** .

    ![Configurar inicio de sesión único](./media/jobscience-tutorial/tutorial_general_400.png)

1. En la sección **Configuración de Jobscience**, haga clic en **Configurar Jobscience** para abrir la ventana **Configurar inicio de sesión**. Copie la **URL del servicio de inicio de sesión único de SAML, el identificador de entidad de SAML y la dirección URL de cierre de sesión** de la sección **Referencia rápida**.

    ![Configurar inicio de sesión único](./media/jobscience-tutorial/tutorial_jobscience_configure.png) 

1. Inicie sesión como administrador en el sitio de la compañía de Jobscience.

1. Acceda a **Setup**(Configuración).
   
   ![Configuración](./media/jobscience-tutorial/IC784358.png "Configurar")

1. En el panel de navegación izquierdo, en la sección **Administer** (Administrar), haga clic en **Domain Management** (Administración de dominios) para expandir la sección relacionada y, luego, haga clic en **My Domain** (Mi dominio) para abrir la página **My Domain** (Mi dominio). 
   
   ![Mi dominio](./media/jobscience-tutorial/ic767825.png "Mi dominio")

1. Para comprobar que el dominio se configuró correctamente, asegúrese de que está en "**Step 4 Deployed to Users**" (Paso 4. Dominio implementado para usuarios) y revise la sección "**My Domain Settings**" (Mi configuración de dominio).

    ![Dominio implementado para el usuario](./media/jobscience-tutorial/ic784377.png "Dominio implementado para el usuario")

1. En el sitio de la compañía de Jobscience, haga clic en **Security Controls** (Controles de seguridad) y, a continuación, haga clic en **Single Sign-On Settings** (Configuración de inicio de sesión único).
    
    ![Security Controls (Controles de seguridad)](./media/jobscience-tutorial/ic784364.png "Controles de seguridad")

1. En la sección **Configuración del inicio de sesión único** , siga estos pasos:
    
    ![Configuración de inicio de sesión único](./media/jobscience-tutorial/ic781026.png "Configuración de inicio de sesión único")
    
    a. Seleccione **SAML habilitado**.

    b. Haga clic en **Nueva**.

1. En el cuadro de diálogo **SAML Single Sign-On Setting Edit** (Edición de la configuración de inicio de sesión único de SAML), realice los pasos siguientes:
    
    ![Configuración del inicio de sesión único de SAML](./media/jobscience-tutorial/ic784365.png "Configuración de inicio de sesión único SAML")
    
    a. En el cuadro de texto **Name** (Nombre), escriba el nombre de la configuración.

    b. En el cuadro de texto **Emisor**, pegue el valor de **SAML Entity ID** (Identificador de entidad de SAML) que ha copiado de Azure Portal.

    c. En el cuadro de texto **Id. de identidad**, escriba `https://salesforce-jobscience.com`

    d. Haga clic en **Examinar** para cargar el certificado de Azure AD.

    e. Como **SAML Identity Type** (Tipo de identidad SAML), seleccione **Assertion contains the Federation ID from the User object** (La aserción contiene el id. de federación del objeto Usuario).

    f. Para **///SAML Identity Location** (Ubicación de identidad SAML), seleccione **///Identity is in the NameIdentfier element of the Subject statement** (La identidad está en el elemento NameIdentifier de la instrucción Subject).

    g. En el cuadro de texto **Identity Provider Login URL** (Dirección URL de inicio de sesión del proveedor de identidades), pegue el valor de **dirección URL del servicio de inicio de sesión único de SAML**, que ha copiado de Azure Portal.

    h. En el cuadro de texto **Identity Provider Logout URL** (Dirección URL de cierre de sesión del proveedor de identidades), pegue el valor de **dirección URL de cierre de sesión**, que ha copiado de Azure Portal.

    i. Haga clic en **Save**(Guardar).

1. En el panel de navegación izquierdo, en la sección **Administer** (Administrar), haga clic en **Domain Management** (Administración de dominios) para expandir la sección relacionada y, luego, haga clic en **My Domain** (Mi dominio) para abrir la página **My Domain** (Mi dominio). 
    
    ![Mi dominio](./media/jobscience-tutorial/ic767825.png "Mi dominio")

1. En la página **My domain** (Mi dominio), en la sección **Login Page Branding** (Personalización de marca de la página de inicio de sesión), haga clic en **Edit** (Editar).
    
    ![Personalización de marca de la página de inicio de sesión](./media/jobscience-tutorial/ic767826.png "Personalización de marca de la página de inicio de sesión")

1. En la página **Login Page Branding** (Personalización de marca de la página de inicio de sesión), en la sección **Authentication Service** (Servicio de autenticación), se muestra el nombre de su **SAML SSO Settings** (Configuración de SSO de SAML). Selecciónelo y luego haga clic en **Save**(Guardar).
    
    ![Personalización de marca de la página de inicio de sesión](./media/jobscience-tutorial/ic784366.png "Personalización de marca de la página de inicio de sesión")

1. Para obtener la dirección URL de inicio de sesión único iniciado por el proveedor de servicios, haga clic en **Configuración de inicio de sesión único** en la sección del menú **Controles de seguridad**.

    ![Security Controls (Controles de seguridad)](./media/jobscience-tutorial/ic784368.png "Controles de seguridad")
    
    Haga clic en el perfil SSO creado en el paso anterior. En esta página se muestra la dirección URL de inicio de sesión único de su empresa; por ejemplo, `https://companyname.my.salesforce.com?so=companyid`.    

> [!TIP]
> Ahora puede leer una versión resumida de estas instrucciones dentro de [Azure Portal](https://portal.azure.com) mientras configura la aplicación.  Después de agregar esta aplicación desde la sección **Active Directory > Aplicaciones empresariales**, simplemente haga clic en la pestaña **Inicio de sesión único** y acceda a la documentación insertada a través de la sección **Configuración** de la parte inferior. Puede leer más aquí sobre la característica de documentación insertada: [Documentación insertada de Azure AD]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD
El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

![Creación de un usuario de Azure AD][100]

**Siga estos pasos para crear un usuario de prueba en Azure AD:**

1. En el panel de navegación izquierdo de **Azure Portal**, haga clic en el icono de **Azure Active Directory**.

    ![Creación de un usuario de prueba de Azure AD](./media/jobscience-tutorial/create_aaduser_01.png) 

1. Para mostrar la lista de usuarios, vaya a **Usuarios y grupos** y haga clic en **Todos los usuarios**.
    
    ![Creación de un usuario de prueba de Azure AD](./media/jobscience-tutorial/create_aaduser_02.png) 

1. Para abrir el cuadro de diálogo **Usuario**, haga clic en **Agregar** en la parte superior del cuadro de diálogo.
 
    ![Creación de un usuario de prueba de Azure AD](./media/jobscience-tutorial/create_aaduser_03.png) 

1. En la página de diálogo **Usuario**, realice los siguientes pasos:
 
    ![Creación de un usuario de prueba de Azure AD](./media/jobscience-tutorial/create_aaduser_04.png) 

    a. En el cuadro de texto **Nombre**, escriba **BrittaSimon**.

    b. En el cuadro de texto **Nombre de usuario**, escriba la **dirección de correo electrónico** de Britta Simon.

    c. Seleccione **Mostrar contraseña** y anote el valor del cuadro **Contraseña**.

    d. Haga clic en **Crear**.
 
### <a name="creating-a-jobscience-test-user"></a>Creación de un usuario de prueba de Jobscience

Para permitir que los usuarios de Azure AD inicien sesión en Jobscience, deben aprovisionarse en Jobscience. En el caso de Jobscience, el aprovisionamiento es una tarea manual.

>[!NOTE]
>Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Jobscience ofrecida por Jobscience para aprovisionar cuentas de usuario de Azure Active Directory.
>  
        
**Siga estos pasos para configurar el aprovisionamiento de usuario:**

1. Inicie sesión como administrador en el sitio de la compañía de **Jobscience** .

1. Vaya a Setup (Configuración).
   
   ![Configuración](./media/jobscience-tutorial/ic784358.png "Configurar")
1. Vaya a **Manage Users \> (Administrar usuarios) Users** (Usuarios).
   
   ![Usuarios](./media/jobscience-tutorial/ic784369.png "Usuarios")
1. Haga clic en **Nuevo usuario**.
   
   ![Todos los usuarios](./media/jobscience-tutorial/ic784370.png "Todos los usuarios")
1. En el cuadro de diálogo **Editar usuario** , realice los siguientes pasos:
   
   ![Edición de usuarios](./media/jobscience-tutorial/ic784371.png "Edición de usuarios")
   
   a. En el cuadro de texto **Nombre**, escriba el nombre del usuario, en este caso, Britta.
   
   b. En el cuadro de texto **Apellidos**, escriba el apellido del usuario, en este caso, Simon.
   
   c. En el cuadro de texto **Alias**, escriba el nombre del alias del usuario, como brittas.

   d. En el cuadro de texto **Correo electrónico**, escriba la dirección de correo electrónico de un usuario, por ejemplo, Brittasimon@contoso.com.

   e. En el cuadro de texto **Nombre de usuario**, escriba un nombre de usuario como Brittasimon@contoso.com.

   f. En el cuadro de texto **Sobrenombre**, escriba un sobrenombre del usuario, como Simon.

   g. Haga clic en **Save**(Guardar).

    
> [!NOTE]
> El titular de la cuenta de Azure Active Directory recibirá un mensaje de correo y seguirá un vínculo para confirmar su cuenta antes de que se active.

### <a name="assigning-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Jobscience.

![Asignar usuario][200] 

**Para asignar a Britta Simon a Jobscience, realice los pasos siguientes:**

1. En Azure Portal, abra la vista de aplicaciones, vaya a la vista de directorio y vaya a **Aplicaciones empresariales**. Luego, haga clic en **Todas las aplicaciones**.

    ![Asignar usuario][201] 

1. En la lista de aplicaciones, seleccione **Jobscience**.

    ![Configurar inicio de sesión único](./media/jobscience-tutorial/tutorial_jobscience_app.png) 

1. En el menú de la izquierda, haga clic en **Usuarios y grupos**.

    ![Asignar usuario][202] 

1. Haga clic en el botón **Agregar**. Después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Asignar usuario][203]

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista de usuarios.

1. Haga clic en el botón **Seleccionar** del cuadro de diálogo **Usuarios y grupos**.

1. Haga clic en el botón **Asignar** del cuadro de diálogo **Agregar asignación**.
    
### <a name="testing-single-sign-on"></a>Prueba del inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Jobscience en el Panel de acceso, debería iniciar sesión automáticamente en su aplicación Jobscience.
Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](../user-help/active-directory-saas-access-panel-introduction.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](tutorial-list.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/jobscience-tutorial/tutorial_general_01.png
[2]: ./media/jobscience-tutorial/tutorial_general_02.png
[3]: ./media/jobscience-tutorial/tutorial_general_03.png
[4]: ./media/jobscience-tutorial/tutorial_general_04.png

[100]: ./media/jobscience-tutorial/tutorial_general_100.png

[200]: ./media/jobscience-tutorial/tutorial_general_200.png
[201]: ./media/jobscience-tutorial/tutorial_general_201.png
[202]: ./media/jobscience-tutorial/tutorial_general_202.png
[203]: ./media/jobscience-tutorial/tutorial_general_203.png

