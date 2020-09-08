---
title: 'Tutorial: Integración de Azure Active Directory con Qlik Sense Enterprise | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Qlik Sense Enterprise.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/03/2020
ms.author: jeedes
ms.openlocfilehash: 5344354c05547d6d2a5e2762c70a97cc4222c464
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88552363"
---
# <a name="tutorial-integrate-qlik-sense-enterprise-with-azure-active-directory"></a>Tutorial: Integración de Qlik Sense Enterprise con Azure Active Directory

En este tutorial, aprenderá a integrar Qlik Sense Enterprise con Azure Active Directory (Azure AD). Al integrar Qlik Sense Enterprise con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Qlik Sense Enterprise
* Permitir que los usuarios inicien sesión automáticamente en Qlik Sense Enterprise (inicio de sesión único) con sus cuentas de Azure AD
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede obtener la evaluación gratuita de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/).
* Suscripción habilitada para el inicio de sesión único (SSO) en Qlik Sense Enterprise

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. 
* Qlik Sense Enterprise admite el inicio de sesión único iniciado por **SP**.
* Qlik Sense Enterprise admite el aprovisionamiento **Just-In-Time**

* Una vez configurado Qlik Sense Enterprise, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="adding-qlik-sense-enterprise-from-the-gallery"></a>Adición de Qlik Sense Enterprise desde la galería

Para configurar la integración de Qlik Sense Enterprise en Azure AD, deberá agregar esta solución desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Qlik Sense Enterprise** en el cuadro de búsqueda.
1. Seleccione **Qlik Sense Enterprise** en el panel de resultados y, luego, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Qlik Sense Enterprise mediante un usuario de prueba llamado **Britta Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Qlik Sense Enterprise.

Para configurar y probar el inicio de sesión único de Azure AD con Qlik Sense Enterprise, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Qlik Sense Enterprise](#configure-qlik-sense-enterprise-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de Qlik Sense Enterprise](#create-qlik-sense-enterprise-test-user)** : para tener un homólogo de Britta Simon en Qlik Sense Enterprise que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Qlik Sense Enterprise**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<Fully Qualified Domain Name>:443{/virtualproxyprefix}/hub`.

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón:

    ```http
    https://<Fully Qualified Domain Name>.qlikpoc.com
    https://<Fully Qualified Domain Name>.qliksense.com
    ```

    c. En el cuadro de texto **URL de respuesta** , escriba una dirección URL con el siguiente patrón:

    `https://<Fully Qualified Domain Name>:443{/virtualproxyprefix}/samlauthn/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL de respuesta, el identificador y la dirección URL de inicio de sesión reales, que se explican más adelante en el tutorial, o póngase en contacto con el [equipo de soporte técnico de Qlik Sense Enterprise](https://www.qlik.com/us/services/support) para obtener estos valores. El puerto predeterminado de las direcciones URL es 443, pero se puede personalizar según las necesidades de la organización.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de federación de metadatos** en las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado Britta Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `Britta Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `BrittaSimon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, concederá acceso a Britta Simon a Qlik Sense Enterprise para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Qlik Sense Enterprise**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En la lista de usuarios del cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-qlik-sense-enterprise-sso"></a>Configuración del inicio de sesión único de Qlik Sense Enterprise

1. Prepare el archivo XML de metadatos de federación para que pueda cargarse en el servidor de Qlik Sense.

    > [!NOTE]
    > Antes de cargar los metadatos de IdP en el servidor de Qlik Sense, hay que editar el archivo para quitar información y, de este modo, garantizar que la conexión entre Azure AD y el servidor de Qlik se realice sin problemas.

    ![QlikSense][qs24]

    a. Abra el archivo FederationMetaData.xml que ha descargado de Azure Portal en un editor de texto.

    b. Busque el valor **RoleDescriptor**.  Hay cuatro entradas (dos pares de etiquetas de elementos de apertura y cierre).

    c. Elimine del archivo las etiquetas de RoleDescriptor y toda la información intermedia.

    d. Guarde el archivo y téngalo a mano, ya que vamos a utilizarlo más adelante en este documento.

2. Acceda a Qlik Sense Qlik Management Console (QMC) como un usuario que pueda crear configuraciones de proxy virtual.

3. En la consola QMC, haga clic en el elemento de menú **Virtual Proxies** (servidores proxy virtuales).

    ![QlikSense][qs6]

4. Haga clic en el botón **Create new** (Crear nuevo) situado en la parte inferior de la pantalla.

    ![QlikSense][qs7]

5. Aparece la pantalla de edición Virtual Proxy (Proxy virtual).  En la parte derecha de la pantalla hay un menú con el que puede hacer que se muestren las opciones de configuración.

    ![QlikSense][qs9]

6. Active la opción de menú Identificación y escriba la información de identificación de la configuración de proxy virtual de Azure.

    ![QlikSense][qs8]  

    a. El campo **Description** (Descripción) es un nombre descriptivo de la configuración del proxy virtual.  Escriba un valor para este campo.

    b. El campo **Prefix** (Prefijo) identifica el punto de conexión del proxy virtual para conectarse a Qlik Sense con el inicio de sesión único de Azure AD.  Escriba un nombre de prefijo único para este proxy virtual.

    c. **Session inactivity timeout (minutes)** (Tiempo de espera de inactividad de la sesión [minutos]) es el tiempo de espera de las conexiones que se establecen a través de este proxy virtual.

    d. **Session cookie header name** (Nombre de encabezado de cookie de sesión) es el nombre de la cookie que almacena el identificador de la sesión de Qlik Sense que recibe un usuario tras autenticarse correctamente.  El nombre debe ser único.

7. Haga clic en la opción de menú Autenticación para que se muestre.  Aparecerá la pantalla Autenticación.

    ![QlikSense][qs10]

    a. El menú desplegable **Anonymous access mode** (Modo de acceso anónimo) determina si los usuarios anónimos pueden acceder a Qlik Sense a través del proxy virtual.  La opción predeterminada es No anonymous user (Ningún usuario anónimo).

    b. El menú desplegable **Authentication method** (Método de autenticación) determina el esquema de autenticación que utilizará el proxy virtual.  Seleccione SAML en la lista desplegable.  Como resultado, aparecerán más opciones.

    c. En el campo **SAML host URI**(identificador URI de host SAML), especifique el nombre de host que tendrán que escribir los usuarios para acceder a Qlik Sense a través de este proxy virtual de SAML.  El nombre de host es el URI del servidor de Qlik Sense.

    d. En el campo **SAML entity ID** (Id. de entidad SAML), escriba el mismo valor que especificó en el campo de URI de host SAML.

    e. **SAML IdP metadata** (Metadatos de IdP de SAML) es el archivo que se editó anteriormente en la sección de **modificación de los metadatos de federación desde la configuración de Azure AD**.  **Antes de cargar los metadatos de IdP, hay que editar el archivo** para quitar información y, de este modo, garantizar que la conexión entre Azure AD y el servidor de Qlik se realice sin problemas.  **Consulte las instrucciones anteriores si aún no ha modificado el archivo.**  Si ya lo ha hecho, haga clic en el botón Examinar y seleccione el archivo de metadatos editado para cargarlo en la configuración de proxy virtual.

    f. Escriba el nombre de atributo o la referencia de esquema del atributo SAML que representa el **UserID** (identificador de usuario) que Azure AD enviará al servidor de Qlik Sense.  La información de referencia del esquema está disponible en las pantallas de la aplicación de Azure que aparecen tras el proceso de configuración.  Para usar el atributo name, escriba `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

    g. Escriba el valor del **directorio de usuarios** que se asociará a los usuarios cuando se autentiquen en el servidor de Qlik Sense a través de Azure AD.  Los valores codificados deben estar entre **corchetes**.  Para utilizar un atributo enviado en la aserción SAML de Azure AD, escriba el nombre del atributo en este cuadro de texto **sin** corchetes.

    h. El **algoritmo de firma SAML** establece la firma de certificados del proveedor de servicios (en este caso, el servidor de Qlik Sense) en la configuración de proxy virtual.  Si el servidor de Qlik Sense utiliza un certificado de confianza generado mediante el proveedor de servicios criptográficos AES y RSA mejorado de Microsoft, cambie el algoritmo de firma SAML a **SHA-256**.

    i. La sección de asignación de atributos SAML permite enviar otros atributos, como grupos, a Qlik Sense para utilizarse en reglas de seguridad.

8. Haga clic en la opción de menú **LOAD BALANCING** (Equilibrio de carga) para que se muestre.  Aparecerá la pantalla Equilibrio de carga.

    ![QlikSense][qs11]

9. Haga clic en el botón **Add new server node** (Agregar nuevo nodo de servidor), seleccione los nodos de motor a los que Qlik Sense enviará sesiones para equilibrar la carga y, luego, haga clic en el botón **Add** (Agregar).

    ![QlikSense][qs12]

10. Haga clic en la opción de menú Opciones avanzadas para que se muestre. Aparece la pantalla Opciones avanzadas.

    ![QlikSense][qs13]

    La lista de permitidos de Host identifica los nombres de host que se aceptan al conectarse al servidor de Qlik Sense.  **Escriba el nombre de host que especificarán los usuarios al conectarse al servidor de Qlik sentido.** El nombre de host es el mismo valor que el URI de host SAML, pero sin "https://".

11. Haga clic en el botón **Aplicar**.

    ![QlikSense][qs14]

12. Haga clic en Aceptar para aceptar el mensaje de advertencia que indica que se reiniciará el proxy vinculado al proxy virtual.

    ![QlikSense][qs15]

13. En la parte derecha de la pantalla, aparece el menú de elementos Asociado.  Haga clic en la opción de menú **Proxies** (Servidores proxy).

    ![QlikSense][qs16]

14. Aparecerá la pantalla Servidores proxy.  Haga clic en el botón **Link** (Vincular) de la parte inferior para vincular un proxy al proxy virtual.

    ![QlikSense][qs17]

15. Seleccione el nodo de proxy que admitirá esta conexión de proxy virtual y haga clic en el botón **Link** (Vincular).  Cuando termine el proceso de vinculación, el proxy aparecerá debajo de los servidores proxy asociados.

    ![QlikSense][qs18]
  
    ![QlikSense][qs19]

16. Cuando pasen entre unos cinco y diez segundos, se mostrará el mensaje Actualizar QMC.  Haga clic en el botón **Refresh QMC** (Actualizar QMC).

    ![QlikSense][qs20]

17. Cuando se actualice la consola QMC, haga clic en el elemento de menú **Virtual proxies** (Servidores proxy virtuales). La nueva entrada SAML virtual proxy (Proxy virtual de SAML) se muestra en la tabla que aparece en la pantalla.  Haga clic en la entrada de proxy virtual.

    ![QlikSense][qs51]

18. En la parte inferior de la pantalla, se activará el botón Download SP metadata (Descargar metadatos de SP).  Haga clic en el botón **Download SP metadata** (Descargar metadatos de SP) para guardar los metadatos en un archivo.

    ![QlikSense][qs52]

19. Abra el archivo de metadatos de SP.  Observe las entradas **entityID** y **AssertionConsumerService**.  Estos valores son equivalentes a los de **Identificador**, **URL de inicio de sesión** y **URL de respuesta** de la configuración de la aplicación de Azure AD. Pegue estos valores en la sección **Dominio y direcciones URL de Qlik Sense Enterprise** en la configuración de la aplicación de Azure AD; si no coinciden, se deben reemplazar en el Asistente para configuración de aplicaciones de Azure AD.

    ![QlikSense][qs53]

### <a name="create-qlik-sense-enterprise-test-user"></a>Creación de un usuario de prueba de Qlik Sense Enterprise

Qlik Sense Enterprise admite el **aprovisionamiento Just-in-Time**, los usuarios se agregan automáticamente al repositorio de "USERS" de Qlik Sense Enterprise, ya que usan la característica de inicio de sesión único. Además, los clientes pueden usar QMC y crear un UDC (conector de directorio de usuario) para rellenar previamente los usuarios en Qlik Sense Enterprise a partir del LDAP que se prefiera, como Active Directory y otros.

### <a name="test-sso"></a>Prueba de SSO

Al seleccionar el icono de Qlik Sense Enterprise en el Panel de acceso, debería iniciar sesión automáticamente en la aplicación Qlik Sense Enterprise para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

<!--Image references-->

[qs6]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_06.png
[qs7]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_07.png
[qs8]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_08.png
[qs9]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_09.png
[qs10]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_10.png
[qs11]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_11.png
[qs12]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_12.png
[qs13]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_13.png
[qs14]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_14.png
[qs15]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_15.png
[qs16]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_16.png
[qs17]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_17.png
[qs18]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_18.png
[qs19]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_19.png
[qs20]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_20.png
[qs24]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_24.png
[qs51]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_51.png
[qs52]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_52.png
[qs53]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_53.png
