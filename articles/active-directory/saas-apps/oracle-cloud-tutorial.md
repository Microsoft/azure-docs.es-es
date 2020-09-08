---
title: 'Tutorial: Integración de Azure Active Directory con Oracle Cloud Infrastructure Console | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Oracle Cloud Infrastructure Console.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/16/2020
ms.author: jeedes
ms.openlocfilehash: d4a478c402346e33a4f27ff2a363de75285b435b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543846"
---
# <a name="tutorial-integrate-oracle-cloud-infrastructure-console-with-azure-active-directory"></a>Tutorial: Integración de Oracle Cloud Infrastructure Console con Azure Active Directory

En este tutorial, aprenderá a integrar Oracle Cloud Infrastructure Console con Azure Active Directory (Azure AD). Cuando integra Oracle Cloud Infrastructure Console con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Oracle Cloud Infrastructure Console.
* Permitir que los usuarios inicien sesión automáticamente en Oracle Cloud Infrastructure Console con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para inicio de sesión único (SSO) en Oracle Cloud Infrastructure Console.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Oracle Cloud Infrastructure Console admite inicio de sesión único iniciado por **SP**.
* Una vez configurado Oracle Cloud Infrastructure Console, puede aplicar controles de sesión, que protegen la filtración y la infiltración de la información confidencial de la organización en tiempo real. Los controles de sesión proceden del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="adding-oracle-cloud-infrastructure-console-from-the-gallery"></a>Adición de Oracle Cloud Infrastructure Console desde la galería

Para configurar la integración de Oracle Cloud Infrastructure Console con Azure AD, debe agregar esta solución desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Oracle Cloud Infrastructure Console** en el cuadro de búsqueda.
1. Seleccione **Oracle Cloud Infrastructure Console** en el panel de resultados y agréguelo a la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Oracle Cloud Infrastructure Console mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Oracle Cloud Infrastructure Console.

Para configurar y probar el inicio de sesión único de Azure AD con Oracle Cloud Infrastructure Console, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B. Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B. Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración de Oracle Cloud Infrastructure Console](#configure-oracle-cloud-infrastructure-console)** , para configurar los valores de inicio de sesión único en el lado de la aplicación.
    1. **[Creación de un usuario de prueba de Oracle Cloud Infrastructure Console](#create-oracle-cloud-infrastructure-console-test-user)** , para tener un homólogo de B. Simon en Oracle Cloud Infrastructure Console que esté vinculado a la representación del usuario en Azure AD.
1. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Oracle Cloud Infrastructure Console**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configuración básica de SAML**, especifique los valores de los siguientes campos:

   > [!NOTE]
   > Puede obtener el archivo de metadatos del proveedor de servicios en la sección de **configuración del inicio de sesión único de Oracle Cloud Infrastructure Console** del tutorial.
    
   1. Haga clic en **Cargar el archivo de metadatos**.

   1. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

   1. Una vez que se haya cargado correctamente el archivo de metadatos, los valores de **Identificador** y **URL de respuesta** se rellenan automáticamente en el cuadro de texto de la sección **Configuración básica de SAML**.
    
      > [!NOTE]
      > Si los valores **Identificador** y **URL de respuesta** no se rellenan automáticamente, hágalo manualmente según sus necesidades.

      En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://console.<REGIONNAME>.oraclecloud.com/`

      > [!NOTE]
      > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico al cliente de Oracle Cloud Infrastructure Console](https://www.oracle.com/support/advanced-customer-support/products/cloud.html) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

   ![Vínculo de descarga del certificado](common/metadataxml.png)

1. La aplicación Oracle Cloud Infrastructure Console espera las aserciones de SAML en un formato específico, que requiere que agregue asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo Atributos de usuario.

   ![imagen](common/edit-attribute.png)

1. Además de lo anterior, la aplicación Oracle Cloud Infrastructure Console espera que se devuelvan algunos atributos más en la respuesta de SAML. En la sección **User Attributes & Claims** (Atributos y notificaciones del usuario) del cuadro de diálogo **Group Claims (Preview)** (Notificaciones de grupo [versión preliminar]), siga estos pasos:

   1. Haga clic en el **lápiz** junto a **Valor de identificador de nombre**.

   1. Seleccione **Persistente** en **Elija el formato del identificador de nombre**.
 
   1. Haga clic en **Save**(Guardar).

      ![imagen](./media/oracle-cloud-tutorial/config07.png)
    
      ![imagen](./media/oracle-cloud-tutorial/config11.png)

   1. Haga clic en el **lápiz** junto a **Groups returned in claim** (Grupos devueltos en la notificación).

   1. Seleccione **Grupos de seguridad** en la lista de selección.

   1. Seleccione **Atributo de origen** de **Id. de grupo**.

   1. Marque **Personalizar nombre de la notificación del grupo**.

   1. En el cuadro de texto **Nombre**, escriba **groupName**.

   1. En el cuadro de texto **Espacio de nombres (opcional)** , escriba `https://auth.oraclecloud.com/saml/claims`.

   1. Haga clic en **Save**(Guardar).

      ![imagen](./media/oracle-cloud-tutorial/config08.png)

1. En la sección **Configurar Oracle Cloud Infrastructure Console**, copie las direcciones URL adecuadas según sus necesidades.

   ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)



### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B. Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B. Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B. Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir a B. Simon utilizar el inicio de sesión único de Azure, para lo que le concederá acceso a Oracle Cloud Infrastructure Console.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Oracle Cloud Infrastructure Console**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

   ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B. Simon** en la lista de usuarios y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-oracle-cloud-infrastructure-console"></a>Configuración de Oracle Cloud Infrastructure Console

1. En otra ventana del explorador web, inicie sesión en Oracle Cloud Infrastructure Console como administrador.

1. Haga clic en el lado izquierdo del menú y luego en **Identity** (Identidad) y, después, vaya a **Federation** (Federación).

   ![Configuración](./media/oracle-cloud-tutorial/config01.png)

1. Guarde el **archivo de metadatos del proveedor de servicios** haciendo clic en el enlace **Download this document** (Descargar este documento), cárguelo en la sección **Configuración básica de SAML** de Azure Portal y, después, haga clic en **Add Identity Provider** (Agregar proveedor de identidades).

   ![Configuración](./media/oracle-cloud-tutorial/config02.png)

1. En el elemento emergente **Add Identity Provider** (Agregar proveedor de identidades), realice los pasos siguientes:

   ![Configuración](./media/oracle-cloud-tutorial/config03.png)

   1. En el cuadro de texto **NAME** (NOMBRE), escriba el nombre.

   1. En el cuadro de texto **DESCRIPTION** (DESCRIPCIÓN), escriba la descripción.

   1. Seleccione **MICROSOFT ACTIVE DIRECTORY FEDERATION SERVICE (ADFS) OR SAML 2.0 COMPLIANT IDENTITY PROVIDER** (SERVICIOS DE FEDERACIÓN DE MICROSOFT ACTIVE DIRECTORY (ADFS) O PROVEEDOR DE IDENTIDADES COMPATIBLE CON SAML 2.0) como **TYPE** (TIPO).

   1. Haga clic en **Browse** (Examinar) para cargar XML de metadatos de federación que descargó de Azure Portal.

   1. Haga clic en **Continue** (Continuar) y en la sección **Set Identity Provider** (Establecer proveedor de identidades), realice los siguientes pasos:

      ![Configuración](./media/oracle-cloud-tutorial/config09.png)

   1. **IDENTITY PROVIDER GROUP** (Grupo de proveedores de identidades) debe seleccionarse como Grupo personalizado. El ID DE GRUPO debe ser el GUID del grupo de Azure Active Directory. El grupo se debe asignar al grupo correspondiente en el campo **OCI GROUP** (GRUPO OCI).

   1. Puede asignar varios grupos según su configuración en Azure Portal y las necesidades de su organización. Haga clic en **+ Add mapping** (+ Agregar asignación) para agregar los grupos que necesite.

   1. Haga clic en **Enviar**.
   
### <a name="create-oracle-cloud-infrastructure-console-test-user"></a>Creación de usuario de prueba de Oracle Cloud Infrastructure Console

 Oracle Cloud Infrastructure Console admite el aprovisionamiento Just-In-Time, de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. No se crea un nuevo usuario durante un intento de acceso y tampoco hay necesidad de crearlo.

### <a name="test-sso"></a>Prueba de SSO

Cuando seleccione el icono de Oracle Cloud Infrastructure Console en el panel de acceso, se le redirigirá a la página de inicio de sesión de Oracle Cloud Infrastructure Console. Seleccione **IDENTITY PROVIDER** (PROVEEDOR DE IDENTIDADES) en el menú desplegable y haga clic en **Continue** (Continuar) tal como se muestra a continuación para iniciar sesión. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

![Configuración](./media/oracle-cloud-tutorial/config10.png)

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Protección de Oracle Cloud Infrastructure Console con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
