---
title: Personalizar la interfaz del usuario
titleSuffix: Azure AD B2C
description: Obtenga información acerca de cómo personalizar la interfaz de usuario en las aplicaciones que usan Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/30/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 8c0b6ba4045e1e92d395752f106258d3c0709108
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87486750"
---
# <a name="customize-the-user-interface-in-azure-active-directory-b2c"></a>Personalización de la interfaz de usuario en Azure Active Directory B2C

La personalización de marca y la personalización de la interfaz de usuario que Azure Active Directory B2C (Azure AD B2C) muestra a los clientes ayuda a proporcionar una experiencia de usuario sin problemas en la aplicación. Estas experiencias incluyen las opciones de registro, inicio de sesión, edición de perfiles y restablecimiento de contraseñas. En este artículo se presentan los métodos de personalización de la interfaz de usuario (UI) para los flujos de usuario y las directivas personalizadas.

## <a name="ui-customization-in-different-scenarios"></a>Personalización de la interfaz de usuario en diferentes escenarios

Hay varias maneras de personalizar la interfaz de usuario del usuario que experimenta su aplicación, cada una de las cuales es adecuada para distintos escenarios.

### <a name="user-flows"></a>Flujos de usuario

Si usa [flujos de usuario](user-flow-overview.md), puede cambiar el aspecto de las páginas de flujo de usuario mediante *plantillas de diseño de página* o mediante su propio HTML y CSS. Ambos métodos se analizarán más adelante en este artículo.

Use [Azure Portal](tutorial-customize-ui.md) para configurar la personalización de la interfaz de usuario para los flujos de usuarios.

> [!TIP]
> Si solo desea modificar el logotipo del banner, la imagen de fondo y el color de fondo de las páginas de flujo de usuario, puede probar la característica [Personalización de marca de empresa (versión preliminar)](#company-branding-preview) que se describe más adelante en este artículo.

### <a name="custom-policies"></a>Directivas personalizadas

Si va a usar [directivas personalizadas](custom-policy-overview.md) para proporcionar en la aplicación las opciones de registro o inicio de sesión, de restablecimiento de contraseña o de edición de perfiles, use los [archivos de directiva para personalizar la interfaz de usuario](custom-policy-ui-customization.md).

Si necesita proporcionar contenido dinámico en función de la decisión de un cliente, use [directivas personalizadas que puedan cambiar el contenido de la página](custom-policy-ui-customization.md#configure-dynamic-custom-page-content-uri) según el parámetro que se envíe en una cadena de consulta. Por ejemplo, puede cambiar la imagen de fondo en la página de inicio de sesión o de registro de Azure AD B2C en función de un parámetro que se pasa desde la aplicación web o dispositivo móvil.

### <a name="javascript"></a>JavaScript

Puede habilitar el código JavaScript del lado cliente en ambos [flujos de usuario](user-flow-javascript-overview.md) y [directivas personalizadas](page-layout.md).

### <a name="sign-in-only-ui-customization"></a>Personalización de la interfaz de usuario de solo inicio de sesión

Si solo proporciona la opción de inicio de sesión junto con la página de restablecimiento de contraseña y los correos electrónicos de comprobación, use los mismos pasos de personalización que se deben usar en una [página de inicio de sesión de Azure AD](../active-directory/fundamentals/customize-branding.md).

Si los clientes intentan editar su perfil antes de iniciar sesión, se les redirige a una página que usted mismo puede personalizar siguiendo los mismos pasos que se usan para personalizar la página de inicio de sesión de Azure AD.

## <a name="page-layout-templates"></a>Plantillas de diseño de página

Los flujos de usuario proporcionan varias plantillas integradas que puede elegir para dar a sus páginas de experiencia del usuario una apariencia profesional. Estas plantillas de diseño también pueden servir como punto de partida para su propia personalización.

En **Personalizar** en el menú de la izquierda, seleccione **Diseños de página** y, después, seleccione **Plantilla**.

![Lista desplegable de selección de plantilla en la página flujo de usuario de Azure Portal](media/customize-ui-overview/template-selection.png)

Después, seleccione una plantilla de la lista. Estos son algunos ejemplos de las páginas de inicio de sesión de cada plantilla:

| Azul océano | Gris pizarra | Clásico |
|:-:|:-:|:-:|
|![Ejemplo de la plantilla Ocean Blue que se representa en la página de inicio de sesión de suscripción](media/customize-ui-overview/template-ocean-blue.png)|![Ejemplo de la plantilla Gris pizarra que se representa en la página de inicio de sesión de suscripción](media/customize-ui-overview/template-slate-gray.png)|![Ejemplo de la plantilla Clásica que se representa en la página de inicio de sesión de suscripción](media/customize-ui-overview/template-classic.png)|

Cuando elige una plantilla, el diseño seleccionado se aplica a todas las páginas del flujo de usuarios y el identificador URI de cada página es visible en el campo **URI de página personalizado**.

## <a name="custom-html-and-css"></a>HTML y CSS personalizados

Si quiere crear su propio diseño de directiva con código HTML y CSS personalizado, active la opción "Usar contenido de la página personalizada" para cada uno de los nombres de diseño presentes en la directiva. Siga las instrucciones siguientes en relación con las configuraciones de diseño personalizadas:

Azure AD B2C ejecuta código en el explorador del cliente con un enfoque denominado [Uso compartido de recursos entre orígenes (CORS)](https://www.w3.org/TR/cors/).

En tiempo de ejecución, el contenido se carga desde una dirección URL especificada en un flujo de usuario o directiva personalizada. Cada página de la experiencia del usuario carga su contenido desde la dirección URL que especifique para esa página. Cuando el contenido se carga desde la dirección URL se combina con un fragmento HTML insertado en Azure AD B2C y, a continuación, se muestra la página al cliente.

Revise la siguiente guía antes de usar sus propios archivos HTML y CSS para personalizar la interfaz de usuario:

- Azure AD B2C combina **contenido** HTML en las páginas. No copie ni intente cambiar el contenido predeterminado que proporciona Azure AD B2C. Es preferible compilar su propio contenido de HTML desde cero y usar el contenido predeterminado como referencia.
- **JavaScript** puede incluirse en el contenido personalizado tanto para los [flujos de usuario](user-flow-javascript-overview.md) como para las [directivas personalizadas](javascript-samples.md).
- Las **versiones de explorador** admitidas son las siguientes:
  - Internet Explorer 11 y 10 y Microsoft Edge
  - Compatibilidad limitada con Internet Explorer 9 y 8.
  - Google Chrome 42.0 y versiones posteriores
  - Mozilla Firefox 38.0 y versiones posteriores
  - Safari para iOS y macOS, versión 12 y posteriores
- No incluya **etiquetas de formulario** en el código HTML. Las etiquetas de formulario interfieren con las operaciones POST generadas por el código HTML que Azure AD B2C inserta.

### <a name="where-do-i-store-ui-content"></a>¿Dónde se puede almacenar el contenido de la interfaz de usuario?

Al utilizar sus propios archivos HTML y CSS para personalizar la interfaz de usuario, puede hospedar el contenido de la interfaz de usuario en cualquier punto de conexión HTTPS disponible públicamente que admita CORS. Por ejemplo, [Azure BLOB Storage](../storage/blobs/storage-blobs-introduction.md), servidores web, redes CDN, AWS S3 o sistemas de uso compartido de archivos.

Recuerde que podrá continuar trabajando siempre y cuando el contenido se hospede en un punto de conexión HTTPS disponible públicamente con [CORS habilitado](https://enable-cors.org/server.html). Debe usar una dirección URL absoluta cuando la especifique en el contenido.

> [!NOTE]
> Para más información sobre la creación de contenido HTML, la carga de contenido en Azure Blob Storage y la configuración de CORS, consulte la sección [Tutorial sobre el contenido de la página personalizada](custom-policy-ui-customization.md#custom-page-content-walkthrough) en el artículo de personalización de la interfaz de usuario.

## <a name="get-started-with-custom-html-and-css"></a>Introducción al código HTML y CSS personalizado

Para empezar a usar su propio código HTML y CSS en las páginas de experiencia del usuario, siga estas instrucciones.

- Cree contenido HTML bien formado con un elemento `<div id="api"></div>` vacío ubicado en algún lugar de `<body>`. Ese elemento marca el lugar en el que se inserta el contenido de Azure AD B2C. En el ejemplo siguiente se muestra una página mínima.

    ```html
    <!DOCTYPE html>
    <html>
      <head>
        <title>!Add your title here!</title>
        <link rel="stylesheet" href="https://mystore1.blob.core.windows.net/b2c/style.css">
      </head>
      <body>
        <h1>My B2C Application</h1>
        <div id="api"></div>   <!-- Leave this element empty because Azure AD B2C will insert content here. -->
      </body>
    </html>
    ```

- Use CSS para aplicar estilo a los elementos de la interfaz de usuario que inserta Azure AD B2C en la página. En el ejemplo siguiente se muestra un archivo CSS sencillo que también incluye la configuración de los elementos HTML insertados en el registro:

    ```css
    h1 {
      color: blue;
      text-align: center;
    }
    .intro h2 {
      text-align: center;
    }
    .entry {
      width: 400px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    .divider h2 {
      text-align: center;
    }
    .create {
      width: 400px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    ```

- Hospedar el contenido en un punto de conexión HTTPS (donde se permite CORS). Tenga en cuenta que los métodos de solicitud GET y OPTIONS se deben habilitar al configurar CORS.
- Cree o edite un flujo de usuario o una directiva personalizada para usar el contenido que haya creado.

### <a name="html-fragments-from-azure-ad-b2c"></a>Fragmentos HTML de Azure AD B2C

En la tabla siguiente se muestran fragmentos de HTML que Azure AD B2C combina en el elemento `<div id="api"></div>` que se encuentra en su contenido.

| Página insertada | Descripción de HTML |
| ------------- | ------------------- |
| Selección del proveedor de identidades | Contiene una lista de botones para los proveedores de identidades entre los que el usuario puede elegir durante el registro o el inicio de sesión. Estos botones incluyen proveedores de identidades sociales como Facebook y Google o las cuentas locales (que están basadas en una dirección de correo electrónico o un nombre de usuario). |
| Registro en una cuenta local | Contiene un formulario para el registro en una cuenta local basada en una dirección de correo electrónico o un nombre de usuario. El formulario puede contener diferentes controles de entrada, como un cuadro de entrada de texto, un cuadro de entrada de contraseña, un botón de radio, cuadros desplegables de selección única y casillas de verificación de selección múltiple. |
| Registro en una cuenta social | Puede aparecer al registrarse con una cuenta existente de un proveedor de identidades de redes sociales, como Facebook o Google. Se usa cuando se debe recopilar información adicional del cliente mediante un formulario de registro. |
| Inicio de sesión o registro unificado | Controla tanto el registro como el inicio de sesión de los clientes, los cuales pueden usar proveedores de identidades sociales como Facebook o Google, o cuentas locales. |
| Multi-Factor Authentication | Los usuarios pueden comprobar sus números de teléfono (mediante mensajes de texto o con la voz) durante el registro o el inicio de sesión. |
| Error | Proporciona información del error al cliente. |

## <a name="company-branding-preview"></a>Personalización de marca de empresa (versión preliminar)

Puede personalizar las páginas del flujo de usuario con un logotipo de banner, una imagen de fondo y un color de fondo mediante [Personalización de marca de empresa](../active-directory/fundamentals/customize-branding.md) de Azure Active Directory.

Antes de personalizar las páginas del flujo de usuario, debe configurar la personalización de marca de empresa en Azure Active Directory y, a continuación, habilitarla en los diseños de página de los flujos de usuario en Azure AD B2C.

[!INCLUDE [preview note](../../includes/active-directory-b2c-public-preview.md)]

### <a name="configure-company-branding"></a>Configuración de la personalización de marca de la compañía

Empiece por establecer el logotipo del banner, la imagen de fondo y el color de fondo en **Personalización de marca de empresa**.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. En **Administrar**, seleccione **Personalización de marca de empresa**.
1. Siga los pasos de [Incorporación de la personalización de marca en la página de inicio de sesión de Azure Active Directory de la organización](../active-directory/fundamentals/customize-branding.md).

Tenga en cuenta las siguientes cuestiones a la hora de configurar la personalización de marca de empresa en Azure AD B2C:

* La personalización de marca de empresa en Azure AD B2C está actualmente limitada a la personalización de la **imagen de fondo**, el **logotipo del banner** y el **color de fondo**. Las demás propiedades del panel de personalización de marca de empresa como, por ejemplo, las incluidas en **Configuración avanzada**, *no son compatibles*.
* En las páginas del flujo de usuario se muestra el color de fondo antes de cargar la imagen de fondo. Se recomienda elegir un color de fondo que coincida en la medida de lo posible con los colores de la imagen de fondo, para conseguir una experiencia de carga más fluida.
* El logotipo del banner aparece en los mensajes de correo electrónico de verificación que se envían a los usuarios cuando inician un flujo de usuario de inicio de sesión.

### <a name="enable-branding-in-user-flow-pages"></a>Habilitación de la personalización de marca en las páginas del flujo de usuario

Después de configurar la personalización de marca de la empresa, habilítela en los flujos de usuario.

1. En el menú izquierdo de Azure Portal, seleccione **Azure AD B2C**.
1. En **Directivas**, seleccione**Flujos de usuario (directivas)** .
1. Seleccione el flujo de usuario para el que desea habilitar la personalización de marca de empresa. La personalización de marca de la empresa **no es compatible** con los tipos de flujo de usuario *Inicio de sesión* estándar y *Edición de perfiles* estándar.
1. En **Personalizar**, seleccione **Diseños de página** y, a continuación, seleccione el diseño en el que desea personalizar la marca. Por ejemplo, seleccione **Página unificada de inicio de sesión o de registro**.
1. En **Versión de Diseño de página (versión preliminar)** , elija la versión **1.2.0** o posterior.
1. Seleccione **Guardar**.

Si desea personalizar la marca de todas las páginas del flujo de usuario, establezca la versión de cada diseño de página incluido en el flujo de usuario.

![Selección del diseño de página de Azure AD B2C en Azure Portal](media/customize-ui-overview/portal-02-page-layout-select.png)

Este ejemplo anotado muestra un logotipo de banner y una imagen de fondo personalizados en una página del flujo de usuario de *Registro e inicio de sesión* que usa la plantilla Azul océano:

![Página de registro o inicio de sesión con personalización de marca ofrecida por Azure AD B2C](media/customize-ui-overview/template-ocean-blue-branded.png)

### <a name="use-company-branding-assets-in-custom-html"></a>Uso de recursos de personalización de marca de empresa en HTML personalizado

Para usar los recursos de personalización de marca de la empresa en HTML personalizado, agregue las siguientes etiquetas fuera de la etiqueta `<div id="api">`:

```HTML
<img data-tenant-branding-background="true" />
<img data-tenant-branding-logo="true" alt="Company Logo" />
```

El origen de la imagen se reemplaza con el de la imagen de fondo y el logotipo del banner. Tal como se describe en la sección [Introducción al código HTML y CSS personalizado](#get-started-with-custom-html-and-css), utilice clases CSS para aplicar estilo a los recursos y colocarlos en la página.

## <a name="localize-content"></a>Localización del contenido

Puede localizar el contenido HTML si habilita la [personalización de idioma](user-flow-language-customization.md) en su inquilino de Azure AD B2C. Al habilitar esta característica, Azure AD B2C puede reenviar el parámetro `ui_locales` de OpenID Connect al punto de conexión. El servidor de contenido puede usar este parámetro para proporcionar páginas HTML específicas del idioma.

Como alternativa, puede extraer contenido de distintos lugares, según la configuración regional que use. En el punto de conexión habilitado para CORS, puede configurar una estructura de carpetas para hospedar contenido de idiomas concretos. Se llamará al adecuado si usa el valor de carácter comodín `{Culture:RFC5646}`.

Por ejemplo, el URI de la página personalizada podría ser similar al siguiente:

```http
https://contoso.blob.core.windows.net/{Culture:RFC5646}/myHTML/unified.html
```

Puede cargar la página en francés mediante la extracción de contenido de:

```http
https://contoso.blob.core.windows.net/fr/myHTML/unified.html
```

## <a name="examples"></a>Ejemplos

Puede encontrar varios archivos de plantilla de ejemplo en el repositorio [B2C-AzureBlobStorage-Client](https://github.com/azureadquickstarts/b2c-azureblobstorage-client) en GitHub.

Los archivos HTML y CSS de ejemplo de las plantillas se encuentran en el directorio [/sample_templates](https://github.com/AzureADQuickStarts/B2C-AzureBlobStorage-Client/tree/master/sample_templates).

## <a name="next-steps"></a>Pasos siguientes

- Si utiliza **flujos de usuario**, puede empezar a personalizar la interfaz de usuario con el tutorial:

    [Personalización de la interfaz de usuario de la aplicación en Azure Active Directory B2C](tutorial-customize-ui.md).
- Si usa **directivas personalizadas**, puede empezar a personalizar la interfaz de usuario con el artículo:

    [Personalización de la interfaz de usuario de la aplicación mediante una directiva personalizada en Azure Active Directory B2C](custom-policy-ui-customization.md).
