---
title: 'Tutorial: Integración de Azure Active Directory con Amazon Business | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Amazon Business.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/16/2019
ms.author: jeedes
ms.openlocfilehash: 0db43f438eaff3e783b829eabe7152c5f13e943d
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548303"
---
# <a name="tutorial-integrate-amazon-business-with-azure-active-directory"></a>Tutorial: Integración de Amazon Business con Azure Active Directory

En este tutorial, aprenderá a integrar Amazon Business con Azure Active Directory (Azure AD). Si integra [Amazon Business](https://www.amazon.com/b2b/info/amazon-business?layout=landing) con Azure AD, podrá:

* Controlar en Azure AD quién tiene acceso a Amazon Business.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Amazon Business con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede obtener la evaluación gratuita de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/).
* Suscripción habilitada para el inicio de sesión único en Amazon Business. Vaya a la página de [Amazon Business](https://www.amazon.com/business/register/org/landing?ref_=ab_reg_mlp) para crear una cuenta de Amazon Business.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y a probar el inicio de sesión único de Azure AD en una cuenta de Amazon Business existente.

* Amazon Business admite el inicio de sesión único iniciado por **SP e IDP**
* Amazon Business admite el aprovisionamiento de usuarios **Just In Time**.

## <a name="adding-amazon-business-from-the-gallery"></a>Adición de Amazon Business desde la galería

Para configurar la integración de Amazon Business en Azure AD, es preciso agregar Amazon Business desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Amazon Business** en el cuadro de búsqueda.
1. Seleccione **Amazon Business** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Amazon Business mediante un usuario de prueba llamado **B.Simon**.

Para configurar y probar el inicio de sesión único de Azure AD con Amazon Business, es preciso completar los siguientes pasos de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
2. **[Configuración del inicio de sesión único de Amazon Business](#configure-amazon-business-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Amazon Business](#create-amazon-business-test-user)** , para tener un homólogo de B.Simon en Amazon Business que esté vinculado a la representación de este usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Amazon Business**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar en el modo iniciado por **IDP**, siga estos pasos:

    1. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con uno de los patrones siguientes:
    
       | URL | Region |
       |-|-|
       | `https://www.amazon.com`| Norteamérica |
       | `https://www.amazon.co.jp`| Este de Asia |
       | `https://www.amazon.de`| Europa |

    1. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con uno de los siguientes patrones:
    
       | URL | Region |
       |-|-|
       | `https://www.amazon.com/bb/feature/sso/action/3p_redirect?idpid={idpid}`| Norteamérica |
       | `https://www.amazon.co.jp/bb/feature/sso/action/3p_redirect?idpid={idpid}`| Este de Asia |
       | `https://www.amazon.de/bb/feature/sso/action/3p_redirect?idpid={idpid}`| Europa |

       > [!NOTE]
       > El valor de dirección URL de respuesta no es real. Actualice este valor con la dirección URL de respuesta real. El valor de `<idpid>` lo obtendrá de la sección de configuración del inicio de sesión único en Amazon Business, que se explica más adelante en el tutorial. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. Si desea configurar la aplicación en modo iniciado por **SP**, deberá agregar la dirección URL completa que se proporciona en la configuración de Amazon Business a la **URL de inicio de sesión** en la sección **Establecer direcciones URL adicionales**.

1. La siguiente captura de muestra la lista de atributos predeterminados. Para editar los atributos, haga clic en el icono **Editar** en la sección **Atributos y notificaciones de usuario**.

    ![Atributos](media/amazon-business-tutorial/map-attribute3.png)

1. Edite los atributos y copie el valor del **espacio de nombres** de estos atributos en el Bloc de notas.

    ![Atributos](media/amazon-business-tutorial/map-attribute4.png)

1. Además de lo anterior, la aplicación Amazon Business espera que se usen algunos atributos más en la respuesta SAML. En la sección **Atributos y notificaciones de usuario** del cuadro de diálogo **Notificaciones de grupos**, siga estos pasos:

    a. Haga clic en el **lápiz** junto a **Groups returned in claim** (Grupos devueltos en la notificación).

    ![imagen](./media/amazon-business-tutorial/config04.png)

    ![imagen](./media/amazon-business-tutorial/config05.png)

    b. Seleccione **Todos los grupos** en la lista de selección.

    c. Seleccione **Id. de grupo** como **Atributo de origen**.

    d. Active la casilla **Personalizar nombre de la notificación del grupo** y escriba el nombre del grupo según el requisito de la organización.

    e. Haga clic en **Save**(Guardar).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos** y seleccione **Descargar** para descargar el certificado y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Amazon Business**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-amazon-business-sso"></a>Configuración del inicio de sesión único de Amazon Business

1. En otra ventana del explorador web, inicie sesión en el sitio de Amazon Business de la compañía como administrador.

1. Haga clic en **User profile** (Perfil de usuario) y seleccione **Business Settings** (Configuración de empresa).

    ![Perfil de usuario](media/amazon-business-tutorial/user-profile.png)

1. En el **asistente para integración del sistema**, seleccione **Inicio de sesión único (SSO)** .

    ![Inicio de sesión único (SSO)](media/amazon-business-tutorial/sso-settings.png)

1. En el **Asistente para configurar SSO**, seleccione el proveedor según los requisitos de su organización y haga clic en **Siguiente**.

    ![Grupo predeterminado](media/amazon-business-tutorial/default-group1.png)
    
    > [!NOTE]
    > Aunque Microsoft ADFS es una opción de la lista, no funciona con el inicio de sesión único de Azure AD.

1. En el asistente **New user account defaults** (Nuevas cuentas de usuario predeterminadas), seleccione el **grupo predeterminado** y, a continuación, seleccione **Default Buying Role** (Rol de compra predeterminado) según el rol de usuario en la organización y haga clic en **Next** (Siguiente).

    ![Grupo predeterminado](media/amazon-business-tutorial/dafault-group2.png)

1. En el asistente **Upload your metadata file** (Asistente para cargar archivos de metadatos), haga clic en **Browse**·(Examinar) para cargar el archivo **Metadata XML** (XML de metadatos), que ha descargado de Azure Portal, y haga clic en **Upload** (Cargar).

    ![Datos de conexión](media/amazon-business-tutorial/connection-data1.png)

1. Después de cargar el archivo de metadatos descargado, los campos de la sección de **datos de conexión** se rellenarán automáticamente. Después, haga clic en **Next** (Siguiente).

    ![Datos de conexión](media/amazon-business-tutorial/connection-data2.png)

1. En el asistente para **cargar la instrucción de atributo**, haga clic en **Skip** (Omitir).

    ![Atributos](media/amazon-business-tutorial/map-attribute1.png)

1. En el **asistente para la asignación de atributos**, agregue los campos de requisitos haciendo clic en la opción **+ Add a field** (+ Agregar un campo). Agregue los valores de atributo, incluido el espacio de nombres, que ha copiado de la sección **Atributos y reclamaciones del usuario** de Azure Portal al campo de **nombre de atributo de SAML** y haga clic en **Next** (Siguiente).

    ![Atributos](media/amazon-business-tutorial/map-attribute2.png)

1. En el **asistente para datos de conexión de Amazon**, haga clic en **Next** (Siguiente).

    ![Conexión](media/amazon-business-tutorial/amazon-connect.png)

1. Compruebe el **estado** de los pasos que se han configurado y haga clic en **Start testing** (Iniciar pruebas).

    ![Conexión](media/amazon-business-tutorial/sso-connection1.png)

1. En el **Asistente para probar la conexión de SSO**, haga clic en **Test** (Prueba).

    ![Conexión](media/amazon-business-tutorial/sso-connection2.png)

1. En el **Asistente para direcciones URL iniciadas por IDP**, antes de hacer clic en **Activar**, copie el valor asignado a **idpid** y péguelo en el parámetro **idpid** en **Dirección URL de respuesta** de la sección **Configuración básica de SAML** en el Azure Portal.

    ![Conexión](media/amazon-business-tutorial/sso-connection3.png)

1. En el asistente **Are you ready to switch to active  SSO?** (¿Está listo para cambiar al inicio de sesión único de SSO?), active la casilla **I have fully tested SSO and am ready to go live** (He probado completamente el inicio de sesión único y estoy listo para empezar a funcionar) y haga clic en **Switch to active** (Cambiar a activo).

    ![Conexión](media/amazon-business-tutorial/sso-connection4.png)

1. Por último, en la sección de **detalles de conexión de SSO**, el **estado** se muestra como **activo**.

    ![Conexión](media/amazon-business-tutorial/sso-connection5.png)
    
    > [!NOTE]
    > Si desea configurar la aplicación en modo iniciado por **SP**, complete el paso siguiente y pegue la dirección URL de inicio de sesión de la captura de pantalla anterior en el cuadro de texto **URL de inicio de sesión** de la sección **Establecer direcciones URL adicionales** en Azure Portal. Utilice el siguiente formato:
    >
    > `https://www.amazon.<TLD>/bb/feature/sso/action/start?domain_hint=<uniqueid>`
    
### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

> [!NOTE]
> Los administradores deben crear los usuarios de prueba en su inquilino si es necesario. En los pasos siguientes se muestra cómo crear un usuario de prueba.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="create-an-azure-ad-security-group-in-the-azure-portal"></a>Creación de un grupo de seguridad de Azure AD en Azure Portal

1. Haga clic en **Azure Active Directory > Todos los grupos**.

    ![Creación de un grupo de seguridad de Azure AD](./media/amazon-business-tutorial/all-groups-tab.png)

1. Haga clic en **Nuevo grupo**:

    ![Creación de un grupo de seguridad de Azure AD](./media/amazon-business-tutorial/new-group-tab.png)

1. Rellene **Tipo de grupo**, **Nombre del grupo**, **Descripción del grupo** y **Tipo de pertenencia**. Haga clic en la flecha para seleccionar los miembros y, a continuación, busque o haga clic en el miembro que desee agregar al grupo. Haga clic en **Seleccionar** para agregar los miembros seleccionados y, a continuación, haga clic en **Crear**.

    ![Creación de un grupo de seguridad de Azure AD](./media/amazon-business-tutorial/group-information.png)

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a Amazon Business mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Amazon Business**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

    >[!NOTE]
    > Si no se asignan los usuarios en Azure AD, se obtiene el siguiente error.

    ![Vínculo de Agregar usuario](media/amazon-business-tutorial/assign-user.png)

### <a name="assign-the-azure-ad-security-group-in-the-azure-portal"></a>Asignación del grupo de seguridad de Azure AD en Azure Portal

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Amazon Business**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **Amazon Business**.

    ![El vínculo de Amazon Business en la lista Aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en **Agregar usuario**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. Busque el grupo de seguridad que desea usar y, a continuación, haga clic en el grupo para agregarlo a la sección Seleccionar miembros. Haga clic en **Seleccionar** y después en **Asignar**.

    ![Búsqueda del grupo de seguridad](./media/amazon-business-tutorial/assign-group.png)

    > [!NOTE]
    > Compruebe las notificaciones en la barra de menús a fin de que se le informe cuando el grupo se asigne correctamente a la aplicación empresarial en Azure Portal.

### <a name="create-amazon-business-test-user"></a>Creación de un usuario de prueba de Amazon Business

En esta sección se crea un usuario llamado B.Simon en Amazon Business. Amazon Business admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en Amazon Business, se crea uno nuevo después de la autenticación.

### <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Amazon Business en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Amazon Business para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
