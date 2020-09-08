---
title: 'Tutorial: Integración de Azure Active Directory con SAP HANA| Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SAP HANA.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/27/2018
ms.author: jeedes
ms.openlocfilehash: fc3b5b096e339d0c0f4bca7afc92d7315246fcd4
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88552295"
---
# <a name="tutorial-azure-active-directory-integration-with-sap-hana"></a>Tutorial: Integración de Azure Active Directory con SAP HANA

En este tutorial, obtendrá información sobre cómo integrar SAP HANA con Azure Active Directory (Azure AD).
La integración de SAP HANA con Azure AD proporciona las siguientes ventajas:

* En Azure AD se puede controlar quién tiene acceso a SAP HANA.
* Puede permitir que los usuarios inicien sesión automáticamente en SAP HANA (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con SAP HANA, necesita los siguientes elementos:

- Una suscripción de Azure AD
- Una suscripción de SAP HANA habilitada para el inicio de sesión único (SSO)
- Una instancia de HANA en ejecución en cualquier IaaS pública, instancias locales, VM de Azure o instancias grandes de SAP en Azure
- La interfaz web de administración de XSA, así como HANA Studio instalados en la instancia HANA

> [!NOTE]
> Para probar los pasos de este tutorial, no se recomienda el uso de un entorno de producción de SAP HANA. Pruebe primero la integración en el entorno de desarrollo o de ensayo de la aplicación y, después, use el entorno de producción.

Para probar los pasos de este tutorial, siga estas recomendaciones:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en SAP HANA

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SAP HANA admite SSO iniciado por **IDP**
* SAP HANA admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-sap-hana-from-the-gallery"></a>Adición de SAP HANA desde la galería

Para configurar la integración de SAP HANA en Azure AD, será preciso que agregue SAP HANA desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar SAP HANA desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **SAP HANA**, seleccione **SAP HANA** en el panel de resultados y, luego, haga clic en el botón **Agregar** para añadir la aplicación.

     ![SAP HANA en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con SAP HANA con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de SAP HANA.

Para configurar y probar el inicio de sesión único de Azure AD con SAP HANA, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de SAP HANA](#configure-sap-hana-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de SAP HANA](#create-sap-hana-test-user)** : el objetivo es tener un homólogo de Britta Simon en SAP HANA que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con SAP HANA, siga estos pasos:

1. En la página de integración de la aplicación **SAP HANA** de [Azure Portal](https://portal.azure.com/), haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de SAP HANA](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba lo siguiente: `HA100`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<Customer-SAP-instance-url>/sap/hana/xs/saml/login.xscfunc`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte al cliente de SAP HANA](https://cloudplatform.sap.com/contact.html) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. La aplicación SAP HANA espera las aserciones de SAML en un formato específico. Configure las siguientes notificaciones para esta aplicación. Puede administrar los valores de estos atributos en la sección **Atributos de usuario** de la página de integración de aplicaciones. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Atributos de usuario**.

    ![imagen](common/edit-attribute.png)

6. En la sección **Atributos del usuario** del cuadro de diálogo **Atributos y notificaciones de usuario**, siga estos pasos:
 
    a. Haga clic en el **icono Editar** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    ![imagen](./media/saphana-tutorial/tutorial_usermail.png)

    ![imagen](./media/saphana-tutorial/tutorial_usermailedit.png)

    b. En la lista **Transformación**, seleccione **ExtractMailPrefix()** .

    c. Desde la lista **Parámetro 1**, seleccione **user.mail**.

    d. Haga clic en **Save**(Guardar).

7. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

### <a name="configure-sap-hana-single-sign-on"></a>Configuración de inicio de sesión único de SAP HANA

1. Para configurar el inicio de sesión único en SAP HANA, inicie sesión en la **consola web de HANA XSA** en el punto de conexión HTTPS respectivo.

    > [!NOTE]
    > En la configuración predeterminada, la dirección URL redirige la solicitud a una pantalla de inicio de sesión, que requiere las credenciales de un usuario de base de datos de SAP HANA autenticado. El usuario que inicie sesión debe tener los permisos para realizar las tareas de administración de SAML.

2. En la interfaz web de XSA, vaya al **proveedor de identidades SAML**. A partir de ahí, seleccione el botón **+** en la parte inferior de la pantalla para mostrar el panel **Add Identity Provider Info** (Agregar información del proveedor de identidades). A continuación, siga estos pasos:

    ![Agregación del proveedor de identidades](./media/saphana-tutorial/sap1.png)

    a. En el panel **Add Identity Provider Info** (Agregar información del proveedor de identidades), pegue el contenido del archivo XML de metadatos, que descargó desde Azure Portal, en el cuadro **Metadatos**.

    ![Adición de la configuración del proveedor de identidades](./media/saphana-tutorial/sap2.png)

    b. Si el contenido del documento XML es válido, el proceso de análisis extrae la información necesaria para los campos **Asunto, Id. de entidad y Emisor** del área de pantalla **Datos generales**. También extrae la información necesaria para los campos de dirección URL del área de pantalla **Destino**, por ejemplo, los campos **URL base y Dirección URL de inicio de sesión único (*)** .

    ![Adición de la configuración del proveedor de identidades](./media/saphana-tutorial/sap3.png)

    c. En el cuadro **Nombre** del área de pantalla **Datos generales**, escriba un nombre para el nuevo proveedor de identidades de SSO de SAML.

    > [!NOTE]
    > El nombre del proveedor de identidades de SAML es obligatorio y debe ser único. Aparece en la lista de proveedores de identidades de SAML disponibles que se muestra cuando se selecciona SAML como método de autenticación que debe usarse para aplicaciones SAP HANA XS. Por ejemplo, puede hacer eso en el área de pantalla **Autenticación** de la herramienta XS Artifact Administration.

3. Seleccione **Guardar** para guardar los detalles del proveedor de identidades de SAML y para agregar el nuevo proveedor de identidades de SAML a la lista de conocidos.

    ![Botón Guardar](./media/saphana-tutorial/sap4.png)

4. En HANA Studio, en las propiedades del sistema de la pestaña **Configuración**, filtre las opciones de configuración por **saml**. A continuación, ajuste **assertion_timeout** de **10 s** a **120 s**.

    ![assertion_timeout setting](./media/saphana-tutorial/sap7.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba **brittasimon\@yourcompanydomain.extension**.  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a SAP HANA.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **SAP HANA**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **SAP HANA**.

    ![Vínculo a SAP HANA en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-sap-hana-test-user"></a>Creación de un usuario de prueba en SAP HANA

Para permitir que los usuarios de Azure AD inicien sesión en SAP HANA, debe aprovisionarlos en SAP HANA.
SAP HANA admite el **aprovisionamiento Just-In-Time**, que está habilitado de forma predeterminada.

Si tiene que crear manualmente un usuario, siga los pasos siguientes:

>[!NOTE]
>Puede cambiar la autenticación externa que usa el usuario. Pueden autenticarse con un sistema externo, como Kerberos. Para obtener información detallada acerca de las identidades externas, póngase en contacto con su [administrador de dominio](https://cloudplatform.sap.com/contact.html).

1. Abra [SAP HANA Studio](https://help.sap.com/viewer/a2a49126a5c546a9864aae22c05c3d0e/2.0.01/en-us) como administrador y, después, active el usuario de base de datos de SSO de SAML.

    ![Crear usuario](./media/saphana-tutorial/sap5.png)

2. Seleccione la casilla invisible a la izquierda de **SAML** y, después, seleccione el vínculo **Configurar**.

3. Seleccione **Agregar** para agregar el proveedor de identidades de SAML.  Seleccione el proveedor de identidades de SAML adecuado y, después, seleccione **Aceptar**.

4. Agregue la **Identidad externa** (en este caso, BrittaSimon) o elija **Cualquiera**. Después, seleccione **Aceptar**.

   > [!Note]
   > Si no está seleccionada la casilla **Cualquiera**, el nombre de usuario en HANA debe coincidir exactamente con el nombre del usuario en UPN antes del sufijo de dominio. (Por ejemplo, BrittaSimon@contoso.com se convierte en BrittaSimon en HANA.)

5. Para realizar pruebas, asigne todos los roles de **XS** para el usuario.

    ![Asignación de roles](./media/saphana-tutorial/sap6.png)

    > [!TIP]
    > Debe conceder permisos adecuados solo para los casos de uso.

6. Guarde el usuario.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de SAP HANA en el panel de acceso y debería iniciar sesión automáticamente en la versión de SAP HANA para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

