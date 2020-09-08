---
title: 'Tutorial: Integración de Azure Active Directory con Kontiki | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Kontiki.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/14/2019
ms.author: jeedes
ms.openlocfilehash: cb652cfdae0ce785d6a076b8b1592d9f700b3383
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549983"
---
# <a name="tutorial-azure-active-directory-integration-with-kontiki"></a>Tutorial: Integración de Azure Active Directory con Kontiki

En este tutorial, aprenderá cómo integrar Kontiki con Azure Active Directory (Azure AD).

La integración de Kontiki con Azure AD le proporciona las siguientes ventajas:

* Puede usar Azure AD para controlar quién tiene acceso a Kontiki.
* Puede permitir que los usuarios inicien sesión automáticamente en Kontiki (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS (software como servicio) con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Kontiki, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción de Azure AD, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Una suscripción de Kontiki habilitada para el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba e integrar Kontiki con Azure AD.

Kontiki admite las siguientes características:

* **Inicio de sesión único iniciado por SP**
* **Aprovisionamiento de usuarios Just-In-Time**

## <a name="add-kontiki-in-the-azure-portal"></a>Adición de Kontiki en Azure Portal

Para integrar Kontiki con Azure AD, debe agregar Kontiki a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Seleccione **Azure Active Directory** en el menú izquierdo.

    ![Opción de Azure Active Directory](common/select-azuread.png)

1. Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

1. Para agregar una aplicación, seleccione **Nueva aplicación**.

    ![Opción Nueva aplicación](common/add-new-app.png)

1. En el cuadro de búsqueda, escriba **Kontiki**. En los resultados de búsqueda, seleccione **Kontiki** y, luego, **Agregar**.

    ![Kontiki en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Kontiki con un usuario de prueba llamado **Britta Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Kontiki.

Para configurar y probar el inicio de sesión único de Azure AD con Kontiki, es preciso completar los siguientes bloques de creación:

| Tarea | Descripción |
| --- | --- |
| **[Configuración del inicio de sesión único en Azure AD](#configure-azure-ad-single-sign-on)** | Permite que los usuarios usen esta característica. |
| **[Configuración del inicio de sesión único de Kontiki](#configure-kontiki-single-sign-on)** | Configura los valores de inicio de sesión único en la aplicación. |
| **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** | Prueba el inicio de sesión único de Azure AD con el usuario Britta Simon. |
| **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** | Permite que Britta Simon use el inicio de sesión único de Azure AD. |
| **[Creación de un usuario de prueba de Kontiki](#create-a-kontiki-test-user)** | Crea un homólogo de Britta Simon en Kontiki que está vinculado a la representación del usuario en Azure AD. |
| **[Prueba de inicio de sesión único](#test-single-sign-on)** | Comprueba que la configuración funciona. |

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, configurará el inicio de sesión único de Azure AD con Kontiki en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en el panel de integración de la aplicación **Kontiki**, seleccione **Inicio de sesión único**.

    ![Configuración de la opción de inicio de sesión único](common/select-sso.png)

1. En el panel **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML** o **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

1. En el panel **Configurar el inicio de sesión único con SAML**, seleccione **Editar** (icono de lápiz) para abrir el panel **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En el panel **Configuración básica de SAML**, en el cuadro de texto **Dirección URL de inicio de sesión**, escriba una dirección URL con el patrón siguiente: `https://<companyname>.mc.eval.kontiki.com`

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Kontiki](common/sp-signonurl.png)

    > [!NOTE]
    > Póngase en contacto con el [equipo de soporte técnico de Kontiki](https://customersupport.kontiki.com/enterprise/contactsupport.html) para obtener el valor adecuado que se debe usar. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En el panel **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** junto a **XML de metadatos de federación**. Seleccione una opción de descarga según sus requisitos. Guarde el certificado en el equipo.

    ![Opción de descarga del certificado del XML de metadatos de federación](common/metadataxml.png)

1. En la sección **Configurar Kontiki**, copie las direcciones URL siguientes según sus necesidades:

    * URL de inicio de sesión
    * Identificador de Azure AD
    * URL de cierre de sesión

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-kontiki-single-sign-on"></a>Configuración del inicio de sesión único de Kontiki

Para configurar el inicio de sesión único en Kontiki, envíe el archivo del XML de metadatos de federación descargado y las direcciones URL apropiadas que copió de Azure Portal al [equipo de soporte técnico de Kontiki](https://customersupport.kontiki.com/enterprise/contactsupport.html). El equipo de soporte técnico de Kontiki usa la información que les envíe para asegurarse de que la conexión de inicio de sesión único de SAML está configurada correctamente en ambos lados.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

En esta sección, creará un usuario de prueba llamado Britta Simon en Azure Portal.

1. En Azure Portal, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.

    ![Opciones Usuarios y Todos los usuarios](common/users.png)

1. Seleccione **Nuevo usuario**.

    ![Opción Nuevo usuario](common/new-user.png)

1. En el panel **Usuario**, siga estos pasos:

    1. En el cuadro **Nombre**, escriba **BrittaSimon**.
  
    1. En el cuadro **Nombre de usuario**, escriba **brittasimon\@\<your-company-domain>.\<extension>** . Por ejemplo, **brittasimon\@contoso.com**.

    1. Active la casilla de verificación **Mostrar contraseña**. Anote el valor que se muestra en el cuadro **Contraseña**.

    1. Seleccione **Crear**.

    ![Panel Usuario](common/user-properties.png)

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, concederá acceso a Britta Simon a Kontiki para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** > **Kontiki**.

    ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

1. En la lista de aplicaciones, seleccione **Kontiki**.

    ![Kontiki en la lista de aplicaciones](common/all-applications.png)

1. En el menú, seleccione **Usuarios y grupos**.

    ![Opción Usuarios y grupos](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. Después, en el panel **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![El panel Agregar asignación](common/add-assign-user.png)

1. En el panel **Usuarios y grupos**, en la lista de usuarios, seleccione **Britta Simon**. Elija **Seleccionar**.

1. Si espera algún valor de rol en la aserción de SAML, en el panel **Seleccionar rol**, seleccione el rol adecuado para el usuario de la lista. Elija **Seleccionar**.

1. En el panel **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-kontiki-test-user"></a>Creación de un usuario de prueba de Kontiki

No hay ningún elemento de acción para que configure el aprovisionamiento de usuarios en Kontiki. Si un usuario asignado intenta iniciar sesión en Kontiki desde el portal Aplicaciones, Kontiki comprueba si el usuario existe. Si no se encuentra ninguna cuenta de usuario, Kontiki la crea automáticamente.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el portal Aplicaciones.

Después de configurar el inicio de sesión único, cuando se selecciona **Kontiki** en el portal Aplicaciones, se inicia automáticamente sesión en Kontiki. Para más información acerca del portal Aplicaciones, consulte [Acceso y uso de aplicaciones en el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte estos artículos:

- [Lista de tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)
- [Inicio de sesión único en aplicaciones en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)
- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
