---
title: Campos de registro de Azure Portal para aplicaciones de desarrollo personalizado
description: Guía para registrar una aplicación de desarrollo personalizado con Azure AD
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: conceptual
ms.date: 06/28/2019
ms.author: ryanwi
ms.openlocfilehash: 1469fcfa68b10353b78d31ccc758c61bf7746692
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88120701"
---
# <a name="azure-portal-registration-fields-for-custom-developed-apps"></a>Campos de registro de Azure Portal para aplicaciones de desarrollo personalizado

Este artículo contiene una breve descripción de todos los campos disponibles en el formulario de registro de aplicaciones de [Azure Portal](https://portal.azure.com).

## <a name="register-a-new-application"></a>Registro de una nueva aplicación

-   Para registrar una nueva aplicación, vaya a [Azure Portal](https://portal.azure.com).

-   En el panel de navegación izquierdo, haga clic en **Azure Active Directory**.

-   Haga clic en **Registros de aplicaciones** y en **Agregar**.

-   Se abrirá el formulario de registro de aplicaciones.

## <a name="fields-in-the-application-registration-form"></a>Campos del formulario de registro de aplicaciones

| Campo            | Descripción                                                                              |
|------------------|------------------------------------------------------------------------------------------|
| Nombre             | Nombre de la aplicación. Debe tener un mínimo de cuatro caracteres.                |
| Tipos de cuenta admitidos| Seleccione las cuentas que quiere que admita la aplicación: cuentas solo en este directorio de la organización, cuentas en cualquier directorio de la organización o cuentas en cualquier directorio de la organización y cuentas personales de Microsoft.  |
| URI de redireccionamiento (opcional) | Seleccione el tipo de aplicación que se va a compilar, **Web** o **Cliente público (móvil y escritorio)** y, a continuación, escriba el identificador URI de redireccionamiento (o la dirección URL de respuesta) para la aplicación. Para aplicaciones web, proporcione la dirección URL base de la aplicación. Por ejemplo, http://localhost:31544 podría ser la dirección URL de una aplicación web que se ejecuta en la máquina local. Los usuarios utilizan esta dirección URL para iniciar sesión en una aplicación cliente web. Para aplicaciones cliente públicas, proporcione el identificador URI que utiliza Azure AD para devolver las respuestas de los tokens. Escriba un valor específico para la aplicación, como myapp://auth. Si desea ejemplos específicos de aplicaciones web o aplicaciones nativa, visite nuestras [guías de inicio rápido](./index.yml).|

Cuando haya rellenado los campos anteriores, la aplicación se registrará en Azure Portal y se le redirigirá a la página de información general de la aplicación. Las páginas de configuración en el panel de la izquierda en **Administrar** tiene más campos para poder personalizar la aplicación. Las tablas siguientes describen todos los campos. En función de si ha creado una aplicación web o una aplicación cliente pública, es posible que solo aparezca un subconjunto de estos campos.

### <a name="overview"></a>Información general

| Campo           | Descripción        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Identificador de aplicación  | Cuando registra una aplicación, Azure AD le asigna un identificador de aplicación. Este identificador puede usarse para identificar la aplicación en las solicitudes de autenticación que se envían a Azure AD, así como para acceder a ciertos recursos, como Graph API.                                                          |
| URI del identificador de la aplicación      | Debe ser un URI único. Normalmente, tendrá el formato **https://&lt;inquilino\_nombre&gt;/&lt;aplicación\_nombre&gt;.** Se utiliza durante el flujo de concesión de autorizaciones como identificador único a fin de especificar el recurso para el que se debe emitir el token. También se convierte en la notificación "aud" del token de acceso emitido. |

### <a name="branding"></a>Personalización de marca

| Campo           | Descripción        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Cargar nuevo logotipo | Puede usar este campo para cargar un logotipo en la aplicación. El logotipo debe tener el formato .bmp, .jpg o .png y el tamaño del archivo debe ser inferior a 100 KB. Las dimensiones de la imagen deben ser 215 x 215 píxeles y las de la imagen central, 94 x 94 píxeles.|
| URL de página principal   | Se trata de la dirección URL de inicio de sesión especificada durante el registro de la aplicación.|

### <a name="authentication"></a>Authentication

| Campo           | Descripción        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| URL de cierre de sesión      | Es la dirección URL de cierre de sesión único. Azure AD envía una solicitud de cierre de sesión a esta dirección URL cuando el usuario termina la sesión en Azure AD con cualquier otra aplicación registrada.|
| Tipos de cuenta admitidos  | Este parámetro especifica si la aplicación pueden utilizarla varios inquilinos. Normalmente, esto significa que las organizaciones externas pueden usar la aplicación si la registran en su inquilino y conceden acceso a los datos de la organización.|
| URL de redireccionamiento      | Las direcciones URL de redireccionamiento o respuesta son puntos de conexión en los que Azure AD devuelve los tokens que solicita la aplicación. En el caso de las aplicaciones nativas, aquí es donde se envía al usuario después de una autorización correcta. Azure°AD comprueba que el URI de redireccionamiento que proporciona la aplicación en la solicitud de OAuth 2.0 coincide con uno de los valores registrados en el portal.|

### <a name="certificates-and-secrets"></a>Certificados y secretos

| Campo           | Descripción        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Secretos de cliente            | Puede crear secretos de cliente, o claves, para acceder mediante programación a las API web protegidas por Azure AD sin interacción del usuario. En la página **Nuevo secreto de cliente**, escriba la descripción de la clave y la fecha de vencimiento. A continuación, haga clic en Guardar para generar la clave. No olvide guardar la clave en un lugar seguro, ya que no podrá acceder a ella más adelante.             |

## <a name="next-steps"></a>Pasos siguientes

[Administración de aplicaciones con Azure Active Directory](../manage-apps/what-is-application-management.md)