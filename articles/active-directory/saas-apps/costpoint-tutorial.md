---
title: 'Tutorial: Integración de Azure Active Directory con Costpoint | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Costpoint.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/06/2019
ms.author: jeedes
ms.openlocfilehash: 1db1589561968cf1e2974edcee2bfe1cccb4a327
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549257"
---
# <a name="tutorial-integrate-costpoint-with-azure-active-directory"></a>Tutorial: Integración de Costpoint con Azure Active Directory

En este tutorial, aprenderá a integrar Costpoint con Azure Active Directory (Azure AD). Al integrar Costpoint con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Costpoint.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Costpoint con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Costpoint.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. Costpoint admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="generate-costpoint-metadata"></a>Generación de metadatos de Costpoint

La configuración del inicio de sesión único de SAML de Costpoint se explica en la guía **DeltekCostpoint711Security.pdf**. Descargue esta guía desde el sitio de soporte técnico de Deltek Costpoint y consulte la sección **SAML Single Sign-on Setup** > **Configure SAML Single Sign-on between Costpoint and Microsoft Azure** (Configuración del inicio de sesión único de SAML -> Configuración del inicio de sesión único de SAML entre Costpoint y Microsoft Azure). Siga las instrucciones y genere un archivo **XML de metadatos de federación del proveedor de servicios de Costpoint**. 

![Utilidad de configuración de Costpoint](./media/costpoint-tutorial/config-utility.png)

## <a name="add-costpoint-from-the-gallery"></a>Adición de Costpoint desde la galería

Para integrar Costpoint con Azure AD, primero agregue Costpoint a la lista de aplicaciones SaaS administradas desde la galería en Azure Portal:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.

1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.

   ![Botón Azure Active Directory](common/select-azuread.png)

1. Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

   ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.

   ![Botón Nueva aplicación](common/add-new-app.png)

1. En la sección **Agregar desde la galería**, escriba **Costpoint** en el cuadro de búsqueda.

   ![Costpoint en la lista de resultados](common/search-new-app.png)

1. En la lista de resultados, seleccione **Costpoint** y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sgn-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Costpoint mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Costpoint.

Para configurar y probar el inicio de sesión único de Azure AD con Costpoint, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
1. **[Configuración de Costpoint](#configure-costpoint)** , para configurar el inicio de sesión único de SAML en la aplicación.
1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Creación de un usuario de prueba de Costpoint](#create-a-costpoint-test-user)** , para tener un homólogo de B.Simon en Costpoint que esté vinculado a la representación de este usuario en Azure AD.
1. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal:

1. En la página de integración de la aplicación **Costpoint**, seleccione **Inicio de sesión único**.

   ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

1. En la sección **Configuración básica de SAML**, si tiene el *archivo de metadatos del proveedor de servicios*, lleve a cabo estos pasos:

   > [!NOTE]
   > Puede obtener el archivo de metadatos del proveedor de servicios en [Generación de metadatos de Costpoint](#generate-costpoint-metadata). El uso del archivo se explica posteriormente en este tutorial.
 
   1. Seleccione el botón **Cargar el archivo de metadatos** y, a continuación, seleccione el archivo **XML de metadatos de federación del proveedor de servicios de Costpoint** generado anteriormente por Costpoint y, finalmente, seleccione el botón **Agregar** para cargar el archivo.

      ![Carga del archivo de metadatos](./media/costpoint-tutorial/upload-metadata.png)
    
   1. Cuando se haya cargado correctamente el archivo de metadatos, los valores **Identificador** y **URL de respuesta** se rellenarán automáticamente en la sección de Costpoint.

      > [!NOTE]
      > Si los valores **Identificador** y **Dirección URL de respuesta** no se rellenan automáticamente, escríbalos manualmente según sus necesidades. Compruebe que los valores de **Identificador (id. de entidad)** y **Dirección URL de respuesta (URL del Servicio de consumidor de aserciones)** estén establecidos correctamente y que la **dirección URL de ACS** sea una dirección URL válida de Costpoint que finalice con **/LoginServlet.cps**.

   1. Seleccione **Establecer direcciones URL adicionales**. En **Estado de la retransmisión**, escriba un valor que utilice el siguiente patrón:`system=[your system]` (por ejemplo, **system=DELTEKCP**).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el icono **Copiar** para copiar la **Dirección URL de metadatos de federación de aplicación** y guárdela en el Bloc de notas.

   ![Certificado de firma SAML](common/copy-metadataurl.png)

### <a name="configure-costpoint"></a>Configuración de Costpoint

1. Vuelva a la utilidad de configuración de Costpoint. En el cuadro de texto **IdP Federation Metadata XML** (XML de metadatos de federación de IdP), pegue el contenido del archivo de la *dirección URL de metadatos de federación de aplicación*. 

   ![Utilidad de configuración de Costpoint](./media/costpoint-tutorial/config-utility-idp.png)

1. Siga las instrucciones de la guía **DeltekCostpoint711Security.pdf** para finalizar la configuración de SAML de Costpoint.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba llamado "B.Simon" en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.

   ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

1. Seleccione **Nuevo usuario**.

   ![Botón Nuevo usuario](common/new-user.png)

1. En las propiedades de **usuario**, realice estos pasos:

   ![Cuadro de diálogo Usuario](common/user-properties.png)

   1. En el campo **Nombre**, escriba **B.Simon**.
   
   1. En el campo **Nombre de usuario**, escriba `b.simon\@yourcompanydomain.extension` (por ejemplo, B.Simon@contoso.com).
   
   1. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el campo **Contraseña**.
   
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir a B.Simon el acceso a Costpoint mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

1. En la lista de aplicaciones, seleccione **Costpoint**.

1. En la sección **Administrar** de la página de información general de la aplicación, seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. En el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

   ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, en la lista **Usuarios** seleccione **B.Simon**. A continuación, elija **Seleccionar**.

1. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista y, a continuación, elija **Seleccionar**.

1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-costpoint-test-user"></a>Creación de un usuario de prueba de Costpoint

En esta sección, creará un usuario en Costpoint. Suponga que el identificador de usuario es **B.SIMON** y que el nombre de usuario es **B.Simon**. Trabaje con el [equipo de atención al cliente de Costpoint](https://www.deltek.com/about/contact-us) para agregar el usuario a la plataforma de Costpoint. El usuario se debe crear y activar antes de poder usar el inicio de sesión único.

Una vez creado, la selección de **Método de autenticación** del usuario debe ser **Active Directory**, la casilla **Inicio de sesión único de SAML** debe estar activada y el nombre de usuario de Azure Active Directory debe ser **Active Directory or Certificate ID** (Active Directory o id. de certificado) (como se muestra en la captura de pantalla siguiente).

![Usuario de Costpoint](./media/costpoint-tutorial/costpoint-user.png)

### <a name="test-sso"></a>Prueba de SSO

Al seleccionar el icono de Costpoint en el Panel de acceso, debería iniciar sesión automáticamente en la aplicación de Costpoint ya que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones de SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
