---
title: 'Tutorial: Integración de Azure Active Directory con Captive Portal de Palo Alto Networks | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Captive Portal de Palo Alto Networks.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/25/2018
ms.author: jeedes
ms.openlocfilehash: 1dad0ecc80302ae6b48d420664723a3a03fc9ea5
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554033"
---
# <a name="tutorial-azure-active-directory-integration-with-palo-alto-networks-captive-portal"></a>Tutorial: Integración de Azure Active Directory con Captive Portal de Palo Alto Networks

En este tutorial aprenderá a integrar Captive Portal de Palo Alto Networks con Azure Active Directory (Azure AD).

La integración de Captive Portal de Palo Alto Networks con Azure AD reporta las siguientes ventajas:

* En Azure AD, puede controlar quién tiene acceso a Captive Portal de Palo Alto Networks.
* En Captive Portal de Palo Alto Networks, los usuarios puede iniciar sesión (inicio de sesión único) con sus cuentas de usuario de Azure AD.
* Puede administrar sus cuentas en una única ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS (software como servicio) con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

Si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Prerrequisitos

Para integrar Azure AD con Captive Portal de Palo Alto Networks, se necesita lo siguiente:

* Una suscripción a Azure Active Directory. Si no tiene Azure AD, puede [obtener una versión de evaluación durante un mes](https://azure.microsoft.com/pricing/free-trial/).
* Una suscripción con inicio de sesión único (SSO) de Captive Portal de Palo Alto Networks.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

Captive Portal de Palo Alto Networks admite estos escenarios:

* **Inicio de sesión único iniciado por IDP**
* **Aprovisionamiento de usuarios Just-In-Time**

## <a name="add-palo-alto-networks-captive-portal-from-the-gallery"></a>Agregar Captive Portal de Palo Alto Networks desde la galería

Para comenzar, en la galería, agregue Captive Portal de Palo Alto Networks a la lista de aplicaciones SaaS administradas:

1. En el menú izquierdo de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![La opción Aplicaciones empresariales del menú](common/enterprise-applications.png)

3. Seleccione **Nueva aplicación**.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Captive Portal de Palo Alto Networks**. En los resultados de la búsqueda, seleccione **Palo Alto Networks - Captive Portal**y, después, seleccione **Agregar**.

     ![Palo Alto Networks - Captive Portal en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Captive Portal de Palo Alto Networks con un usuario de prueba llamado *Britta Simon*. Para que el inicio de sesión único funcione, es preciso establecer una relación entre un usuario de Azure AD y el mismo usuario de Captive Portal de Palo Alto Networks. 

Para configurar y probar el inicio de sesión único de Azure AD con Captive Portal, Palo Alto Networks, realice las siguientes tareas:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : permita al usuario utilizar esta característica.
2. **[Configuración del inicio de sesión único de Captive Portal de Palo Alto Networks](#configure-palo-alto-networks-captive-portal-single-sign-on)** : configure los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** : prueba el inicio de sesión único de Azure AD con el usuario *Britta Simon*.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** : configure Britta Simon para que use el inicio de sesión único de Azure AD.
5. **Creación de un usuario de prueba en Captive Portal de Palo Alto Networks**: Cree un usuario homólogo de *Britta Simon* en Captive Portal de Palo Alto Networks que esté vinculado al usuario de Azure AD.
6. **[Prueba de inicio de sesión único](#test-single-sign-on)** : compruebe que la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En primer lugar, habilite el inicio de sesión único de Azure AD en Azure Portal:

1. En la página de integración de la aplicación [Palo Alto Networks - Captive Portal](https://portal.azure.com/) de **Azure Portal**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el panel **Seleccione un método de inicio de sesión único**, seleccione **SAML**.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En el panel **Configurar el inicio de sesión único con SAML** , seleccione el icono de **edición**, el lápiz.

    ![El icono de edición con el lápiz](common/edit-urls.png)

4. En el panel **Configuración básica de SAML**, siga estos pasos:

    ![Panel Configuración básica de SAML de Captive Portal de Palo Alto Networks](common/idp-intiated.png)

   1. En **Identificador**, escriba una dirección URL con el patrón `https://<customer_firewall_host_name>/SAML20/SP`.

   2. En **URL de respuesta** , escriba una dirección URL con el patrón `https://<customer_firewall_host_name>/SAML20/SP/ACS`.

      > [!NOTE]
      > En este paso, actualice los valores de los marcadores de posición con las direcciones URL del identificador y de respuesta reales. Para obtener los valores reales, póngase en contacto con el [equipo de soporte técnico del cliente de Captive Portal de Palo Alto Networks](https://support.paloaltonetworks.com/support).

5. En la sección **Certificado de firma de SAML**, al lado de **XML de metadatos de federación**, seleccione **Descargar**. Guarde el archivo descargado en el equipo.

    ![El vínculo de descarga del XML de metadatos de federación](common/metadataxml.png)

### <a name="configure-palo-alto-networks-captive-portal-single-sign-on"></a>Configuración del inicio de sesión único de Captive Portal, de Palo Alto Networks

A continuación, configure el inicio de sesión único en Captive Portal de Palo Alto Networks:

1. En otra ventana del explorador web, inicie sesión en el sitio web de Palo Alto Networks como administrador.

2. Seleccione la pestaña **Device** (Dispositivo).

    ![La pestaña Device (Dispositivo) del sitio web de Palo Alto Networks](./media/paloaltonetworks-captiveportal-tutorial/tutorial_paloaltoadmin_admin1.png)

3. En el menú, seleccione **SAML Identity Provider** (Proveedor de identidades SAML) y, después, seleccione **Import** (Importar).

    ![El botón Import (Importar)](./media/paloaltonetworks-captiveportal-tutorial/tutorial_paloaltoadmin_admin2.png)

4. En el cuadro de diálogo **SAML Identity Provider Server Profile Import** (Importación de perfil de servidor del proveedor de identidades SAML), siga estos pasos:

    ![Configurar el inicio de sesión único en Palo Alto Networks](./media/paloaltonetworks-captiveportal-tutorial/tutorial_paloaltoadmin_admin3.png)

    1. En **Profile Name** (Nombre de perfil), escriba un nombre, como **AzureAD-CaptivePortal**.
    
    2. Al lado de **Identity Provider Metadata** (Metadatos del proveedor de identidades), seleccione **Browse** (Examinar). Seleccione el archivo metadata.xml que descargó en Azure Portal.
    
    3. Seleccione **Aceptar**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

A continuación creará un usuario de prueba llamado *Britta Simon* en Azure Portal:

1. En Azure Portal, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario**.

    ![El botón Nuevo usuario](common/new-user.png)

3. En el panel **Usuario**, siga estos pasos:

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    1. En **Nombre**, escriba **BrittaSimon**.
  
    2. En **Nombre de usuario**, escriba **BrittaSimon\@\<your_company_domain\>** . Por ejemplo, **BrittaSimon\@contoso.com**.

    3. En **Contraseña**, escriba una contraseña. Se recomienda conservar una copia de la contraseña que se escriba. Puede seleccionar la casilla **Mostrar contraseña** para ver la contraseña.

    4. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

A continuación, conceda acceso a Captive Portal de Palo Alto Networks para que Britta Simon pueda usar el inicio de sesión único Azure:

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba **Palo Alto Networks - Captive Portal** y seleccione la aplicación.

    ![El vínculo a Palo Alto Networks - Captive Portal en la lista de aplicaciones](common/all-applications.png)

3. En el menú, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Seleccione **Agregar usuario**. Después, en el panel **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![El panel Agregar asignación](common/add-assign-user.png)

5. En el panel **Usuarios y grupos**, en la lista **Usuarios**, seleccione **Britta Simon**. Elija **Seleccionar**.

6. Para agregar un valor de rol a la aserción SAML, en el panel **Seleccionar rol**, seleccione el rol pertinente para el usuario. Elija **Seleccionar**.

7. En el panel **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-palo-alto-networks-captive-portal-test-user"></a>Creación de un usuario de prueba de Captive Portal de Palo Alto Networks

A continuación, cree un usuario llamado *Britta Simon* en Captive Portal de Palo Alto Networks. Captive Portal de Palo Alto Networks admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. En esta sección no es preciso realizar ninguna tarea. Si un usuario no existe aún en Captive Portal de Palo Alto Networks, se crea después de la autenticación.

> [!NOTE]
> Si desea crear un usuario manualmente, póngase en contacto con el [equipo de soporte técnico del cliente de Captive Portal de Palo Alto Networks](https://support.paloaltonetworks.com/support).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

Captive Portal de Palo Alto Networks se instala detrás del firewall en una máquina virtual de Windows. Para probar el inicio de sesión único en Captive Portal de Palo Alto Networks, inicie sesión en la máquina virtual Windows mediante el protocolo de escritorio remoto (RDP). En la sesión de RDP, abra un explorador y vaya a cualquier sitio web. Se abre la dirección URL de inicio de sesión único y se le solicitará que se autentique. Tras realizar la autenticación podrá acceder a los sitios web.

## <a name="additional-resources"></a>Recursos adicionales

Para más información, consulte estos artículos:

- [Tutoriales acerca de la integración de aplicaciones SaaS en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)
- [Inicio de sesión único en aplicaciones en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)
- [Acceso condicional en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

