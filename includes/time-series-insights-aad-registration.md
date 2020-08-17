---
title: archivo de inclusión
description: archivo de inclusión
ms.topic: include
ms.custom: include file
services: time-series-insights
ms.service: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: cshankar
ms.date: 02/03/2020
ms.openlocfilehash: b2e86a92c253bc1926386269e289cdb7ae578746
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080782"
---
1. En [Azure Portal](https://ms.portal.azure.com/), seleccione **Azure Active Directory** > **Registros de aplicaciones** > **Nuevo registro**.

   [![Nuevo registro de aplicaciones en Azure Active Directory](media/time-series-insights-aad-registration/active-directory-new-application-registration.png)](media/time-series-insights-aad-registration/active-directory-new-application-registration.png#lightbox)

    La aplicación se mostrará aquí después de registrarla.

1. Asigne un nombre a la aplicación y seleccione **Solo las cuentas de este directorio organizativo** para especificar los **tipos de cuenta admitidos** que pueden acceder a la API. Elija un URI válido al que redirigir a los usuarios después de autenticarse y, luego, seleccione **Registrar**.

   [![Creación de la aplicación en Azure Active Directory](media/time-series-insights-aad-registration/active-directory-registration.png)](media/time-series-insights-aad-registration/active-directory-registration.png#lightbox)

1. En la hoja **Información general** de la aplicación mostrada, aparece información importante de la aplicación de Azure Active Directory. Seleccione la aplicación en **Aplicaciones propias** y, luego, **Información general**.

   [![Copia del identificador de la aplicación](media/time-series-insights-aad-registration/active-directory-copy-application-id.png)](media/time-series-insights-aad-registration/active-directory-copy-application-id.png#lightbox)

   Copie el **identificador de la aplicación (cliente)** para usar en la aplicación cliente.

1. En la hoja **Autenticación** se especifican valores de configuración de autenticación importantes. 

    1. Seleccione **+ Agregar una plataforma** para agregar valores de **URI de redirección** y configurar la opción **Tokens de acceso**.

    1. Determine si la aplicación es un **cliente público** o no seleccionando **Sí** o **No**.

    1. Compruebe qué cuentas e inquilinos se admiten.

    [![Configuración de concesión implícita](media/time-series-insights-aad-registration/active-directory-auth-blade.png)](media/time-series-insights-aad-registration/active-directory-auth-blade.png#lightbox)

1. Después de seleccionar la plataforma adecuada, configure las opciones de **URI de redirección** y **Tokens de acceso** en el panel lateral a la derecha de la interfaz de usuario.

    1. El valor de **URI de redirección** debe coincidir con la dirección proporcionada en la solicitud de autenticación:

        * Si se trata de aplicaciones hospedadas en un entorno de desarrollo local, seleccione **Cliente público (móvil y escritorio)** . Asegúrese de establecer el **cliente público** en **Sí**.
        * En el caso de aplicaciones de página única hospedadas en Azure App Service, seleccione **Web**.

    1. Determine si procede agregar una **URL de cierre de sesión**.

    1. Marque **Tokens de acceso** o **Tokens de id.** para habilitar el flujo de concesión implícita.

    [![Creación de URI de redirección](media/time-series-insights-aad-registration/active-directory-auth-redirect-uri.png)](media/time-series-insights-aad-registration/active-directory-auth-redirect-uri.png#lightbox)

    Haga clic en **Configurar** y, a continuación, en **Guardar**.

1. Seleccione **Certificados y secretos** y, después, **Nuevo secreto de cliente** para crear una contraseña de aplicación que la aplicación cliente pueda usar para demostrar su identidad.

   [![Creación de un secreto de cliente](media/time-series-insights-aad-registration/active-directory-application-keys-save.png)](media/time-series-insights-aad-registration/active-directory-application-keys-save.png#lightbox)

   Se mostrará la contraseña del secreto de cliente. Copie la clave en el editor de texto que prefiera.

   > [!NOTE]
   > En su lugar, también puede importar un certificado. Para mejorar la seguridad, le recomendamos que use un certificado. Para usar un certificado, seleccione **Cargar certificado**.

1. Asocie la aplicación de Azure Active Directory a Azure Time Series Insights. Seleccione **Permisos de API** > **Agregar un permiso** > **API usadas en mi organización**. 

    [![Asociación de una API a la aplicación de Azure Active Directory](media/time-series-insights-aad-registration/active-directory-app-api-permission.png)](media/time-series-insights-aad-registration/active-directory-app-api-permission.png#lightbox)

   Escriba `Azure Time Series Insights` en la barra de búsqueda y, luego, seleccione `Azure Time Series Insights`.

1. A continuación, especifique el tipo de permiso de API que requiere la aplicación. De forma predeterminada, se resalta la opción **Permisos delegados**. Elija un tipo de permiso y, luego, seleccione **Agregar permisos**.

    [![Especificación del tipo de permiso de API que requiere la aplicación](media/time-series-insights-aad-registration/active-directory-app-permission-grant.png)](media/time-series-insights-aad-registration/active-directory-app-permission-grant.png#lightbox)
