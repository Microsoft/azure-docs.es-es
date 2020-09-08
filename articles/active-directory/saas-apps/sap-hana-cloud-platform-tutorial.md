---
title: 'Tutorial: Integración de Azure Active Directory con SAP Cloud Platform | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SAP Cloud Platform.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/17/2018
ms.author: jeedes
ms.openlocfilehash: c2738e1a6168440adee79ebaa599a313600153a4
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88546770"
---
# <a name="tutorial-azure-active-directory-integration-with-sap-cloud-platform"></a>Tutorial: Integración de Azure Active Directory con SAP Cloud Platform

En este tutorial, aprenderá a integrar SAP Cloud Platform con Azure Active Directory (Azure AD).
La integración de SAP Cloud Platform con Azure AD proporciona las siguientes ventajas:

* En Azure AD se puede controlar quién tiene acceso a SAP Cloud Platform.
* Puede permitir que los usuarios inicien sesión automáticamente en SAP Cloud Platform (inicio de sesión único) con su cuenta de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con SAP Cloud Platform, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para inicio de sesión único en SAP Cloud Platform

Después de completar este tutorial, los usuarios de Azure AD que haya asignado a SAP Cloud Platform podrán realizar un inicio de sesión único en la aplicación con la información del artículo de [introducción al panel de acceso](../user-help/active-directory-saas-access-panel-introduction.md).

>[!IMPORTANT]
>Deberá implementar su propia aplicación o suscribirse a una aplicación en su cuenta de SAP Cloud Platform para probar el inicio de sesión único. En este tutorial, se implementa una aplicación en la cuenta.
> 

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SAP Cloud Platform admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-sap-cloud-platform-from-the-gallery"></a>Adición de SAP Cloud Platform desde la galería

Para configurar la integración de SAP Cloud Platform en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar SAP Cloud Platform desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **SAP Cloud Platform**, seleccione **SAP Cloud Platform** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![SAP Cloud Platform en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con SAP Cloud Platform con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de SAP Cloud Platform.

Para configurar y probar el inicio de sesión único de Azure AD con SAP Cloud Platform, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de SAP Cloud Platform](#configure-sap-cloud-platform-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de SAP Cloud Platform](#create-sap-cloud-platform-test-user)** : para tener un homólogo de Britta Simon en SAP Cloud Platform vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con SAP Cloud Platform, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **SAP Cloud Platform**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de SAP Cloud Platform](common/sp-identifier-reply.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL que utilizan los usuarios para iniciar sesión en la aplicación **SAP Cloud Platform**. Se trata de la URL específica de la cuenta de un recurso protegido en su aplicación SAP Cloud Platform. La dirección URL se parece al siguiente patrón: `https://<applicationName><accountName>.<landscape host>.ondemand.com/<path_to_protected_resource>`
      
    >[!NOTE]
    >Se trata de la URL de la aplicación SAP Cloud Platform que requiere la autenticación del usuario.
    > 

    - `https://<subdomain>.hanatrial.ondemand.com/<instancename>`
    - `https://<subdomain>.hana.ondemand.com/<instancename>`

    b. En el cuadro de texto **Identificador** proporcionará el de su aplicación SAP Cloud Platforn, escriba una dirección URL conforme a uno de los siguientes patrones: 

    - `https://hanatrial.ondemand.com/<instancename>`
    - `https://hana.ondemand.com/<instancename>`
    - `https://us1.hana.ondemand.com/<instancename>`
    - `https://ap1.hana.ondemand.com/<instancename>`

    c. En el cuadro de texto **URL de respuesta** , escriba una dirección URL con el siguiente patrón:

    - `https://<subdomain>.hanatrial.ondemand.com/<instancename>`
    - `https://<subdomain>.hana.ondemand.com/<instancename>`
    - `https://<subdomain>.us1.hana.ondemand.com/<instancename>`
    - `https://<subdomain>.dispatcher.us1.hana.ondemand.com/<instancename>`
    - `https://<subdomain>.ap1.hana.ondemand.com/<instancename>`
    - `https://<subdomain>.dispatcher.ap1.hana.ondemand.com/<instancename>`
    - `https://<subdomain>.dispatcher.hana.ondemand.com/<instancename>`

    > [!NOTE] 
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, el identificador y la dirección URL de respuesta reales. Póngase en contacto con el [equipo de atención al cliente de SAP Cloud Platform](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/5dd739823b824b539eee47b7860a00be.html) para obtener la dirección URL de inicio de sesión y el identificador. La dirección URL de respuesta se obtiene de la sección de administración de confianza que se explica más adelante en el tutorial.
    > 
4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

### <a name="configure-sap-cloud-platform-single-sign-on"></a>Configuración del inicio de sesión único de SAP Cloud Platform

1. En otra ventana del explorador web, inicie sesión en SAP Cloud Platform Cockpit en `https://account.<landscape host>.ondemand.com/cockpit` (por ejemplo: https://account.hanatrial.ondemand.com/cockpit) ).

2. Haga clic en la pestaña **Trust** (Confianza).
   
    ![Trust (Confianza)](./media/sap-hana-cloud-platform-tutorial/ic790800.png "Trust")

3. En la sección Trust Management (Administración de confianza), en **Local Service Provider** (Proveedor de servicios local), realice los pasos siguientes:

    ![Trust Management (Administración de confianza)](./media/sap-hana-cloud-platform-tutorial/ic793931.png "Administración de confianza")
   
    a. Haga clic en **Editar**.

    b. En **Configuration Type** (Tipo de configuración), seleccione **Custom** (Personalizado).

    c. Como **Local Provider Name**(Nombre de proveedor local), deje el valor predeterminado. Copie este valor y péguelo en el campo **Identificador** de la configuración de Azure AD para SAP Cloud Platform.

    d. Para generar una **Signing Key** (Clave de firma) y un par de claves **Signing Certificate** (Certificado de firma), haga clic en **Generate Key Pair** (Generar par de claves).

    e. En **Principal Propagation** (Propagación de entidad de seguridad), seleccione **Disabled** (Deshabilitada).

    f. En **Force Authentication** (Forzar autenticación), seleccione **Disabled** (Deshabilitado).

    g. Haga clic en **Save**(Guardar).

4. Después de guardar la configuración de **Local Service Provider** (Proveedor de servicios local), realice lo siguiente para obtener la dirección URL de respuesta:
   
    ![Get Metadata (Obtener metadatos)](./media/sap-hana-cloud-platform-tutorial/ic793930.png "Get Metadata")

    a. Para descargar el archivo de metadatos de SAP Cloud Platform, haga clic en **Get Metadata** (Obtener metadatos).

    b. Abra el archivo de metadatos de SAP Cloud Platform descargado y busque la etiqueta **ns3:AssertionConsumerService**.
 
    c. Copie el valor del atributo **Location** y péguelo en el campo **URL de respuesta** de la configuración de Azure AD para SAP Cloud Platform.

5. Haga clic en la pestaña **Trusted Identity Provider** (Proveedor de identidades de confianza) y en **Add Trusted Identity Provider** (Agregar proveedor de identidad de confianza).
   
    ![Trust Management (Administración de confianza)](./media/sap-hana-cloud-platform-tutorial/ic790802.png "Administración de confianza")
   
    >[!NOTE]
    >Para administrar la lista de proveedores de identidades de confianza, deberá haber elegido el tipo de configuración personalizada en la sección del proveedor de servicios local. Para el tipo de configuración predeterminado, tendrá una confianza implícita y no editable para el servicio de id. de SAP. Para Ninguno, no tiene ninguna configuración de confianza.
    > 
    > 

6. Haga clic en la pestaña **General** y en **Browse** (Examinar) para cargar el archivo de metadatos descargados.
    
    ![Trust Management (Administración de confianza)](./media/sap-hana-cloud-platform-tutorial/ic793932.png "Administración de confianza")
    
    >[!NOTE]
    >Después de cargar el archivo de metadatos, los valores de **Dirección URL de inicio de sesión único**, **Dirección URL de cierre de sesión único** y **Certificado de firma** se rellenan automáticamente.
    > 
     
7. Haga clic en la pestaña **Attributes** (Atributos).

8. En la pestaña **Attributes** (Atributos), realice los pasos siguientes:
    
    ![Atributos](./media/sap-hana-cloud-platform-tutorial/ic790804.png "Atributos") 

    a. Haga clic en **Add Assertion-Based Attribute**(Agregar atributo basado en la aserción) y agregue los siguientes atributos basados en aserción:
       
    | Atributo de aserción | Atributo de entidad de seguridad |
    | --- | --- |
    | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname` |firstname |
    | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname` |lastname |
    | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |email |
   
     >[!NOTE]
     >La configuración de los atributos depende de cómo se desarrollen las aplicaciones en SCP, es decir, qué atributos se esperan en la respuesta de SAML y con qué nombre (atributo de la entidad de seguridad) se accede a este atributo en el código.
     > 
    
    b. El **atributo predeterminado** de la captura de pantalla solo es para fines ilustrativos. No es necesario para que el escenario funcione.  
 
    c. Los nombres y valores para el **Principal Attribute** (Atributo principal) que se muestran en la captura de pantalla dependen de cómo se desarrolle la aplicación. Es posible que la aplicación requiera diferentes asignaciones.

### <a name="assertion-based-groups"></a>Grupos basados en aserción

Como paso opcional, puede configurar grupos basados en aserciones para su proveedor de identidades de Azure Active Directory.

El uso de grupos en SAP Cloud Platform permite asignar dinámicamente uno o más usuarios a uno o más roles en las aplicaciones SAP Cloud Platform, en función de los valores de los atributos de la aserción de SAML 2.0. 

Por ejemplo, si la aserción contiene el atributo "*contrato=temporal*", puede que desee que se agreguen todos los usuarios afectados al grupo "*TEMPORAL*". El grupo "*TEMPORAL*" puede contener uno o varios roles de una o más de las aplicaciones implementadas en la cuenta de SAP Cloud Platform.
 
Use grupos basados en aserciones si quiere asignar de manera simultánea muchos usuarios a uno o más roles de las aplicaciones en su cuenta de SAP Cloud Platform. Si quiere asignar un número único o pequeño de usuarios a roles específicos, se recomienda asignarlos directamente en la pestaña "**Autorizaciones**" de SAP Cloud Platform Cockpit.

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

En esta sección, concederá acceso a Britta Simon a SAP Cloud Platform para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **SAP Cloud Platform**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **SAP Cloud Platform**.

    ![Vínculo a SAP Cloud Platform en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-sap-cloud-platform-test-user"></a>Creación de un usuario de prueba de SAP Cloud Platform

Para permitir que los usuarios de Azure AD inicien sesión en SAP Cloud Platform, debe asignarles roles en SAP Cloud Platform.

**Para asignar un rol a un usuario, lleve a cabo los siguientes pasos:**

1. Inicie sesión en de **SAP Cloud Platform Cockpit**.

2. Lleve a cabo los siguientes pasos:
   
    ![Authorizations (Autorizaciones)](./media/sap-hana-cloud-platform-tutorial/ic790805.png "Autorizaciones")
   
    a. Haga clic en **Authorization**(Autorización).

    b. Haga clic en la pestaña **Usuarios**.

    c. En el cuadro de texto **User** (Usuario), escriba la dirección de correo electrónico del usuario.

    d. Haga clic en **Assign** (Asignar) para asignar el usuario a un rol.

    e. Haga clic en **Save**(Guardar).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de SAP Cloud Platform del panel de acceso, debería iniciar sesión automáticamente en la aplicación SAP Cloud Platform para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

