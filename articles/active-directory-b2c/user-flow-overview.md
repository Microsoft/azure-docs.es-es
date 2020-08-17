---
title: Flujos de usuario de Azure Active Directory B2C | Microsoft Docs
titleSuffix: Azure AD B2C
description: Obtenga más información sobre el marco de directiva extensible de Azure Active Directory B2C y cómo crear distintos flujos de usuario.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 07/30/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 7a7736602fafb740d1d76fa09fd26da25e4ff9f5
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87481604"
---
# <a name="user-flows-in-azure-active-directory-b2c"></a>Flujos de usuario en Azure Active Directory B2C

Para ayudarle a configurar las tareas más comunes de identidad para sus aplicaciones, el portal de Azure AD B2C incluye directivas predefinidas y configurables denominadas **flujos de usuario**. Un flujo de usuario permite determinar cómo interactúan los usuarios con su aplicación cuando realizan acciones como iniciar sesión, registrarse, editar un perfil o restablecer una contraseña. Con los flujos de usuario, puede controlar las siguientes funcionalidades:

- Tipos de cuenta utilizados para iniciar sesión, como cuentas de redes sociales (como Facebook) o cuentas locales
- Atributos que se recopilarán del consumidor durante el registro, como el nombre, código postal y número de calzado
- Azure Multi-Factor Authentication
- Personalización de la interfaz de usuario
- Información que la aplicación recibe como notificaciones en un token

Puede crear muchos flujos de usuario de diferentes tipos en su inquilino y usarlos en sus aplicaciones según sea necesario. Los flujos de usuario se pueden volver a usar en todas las aplicaciones. Esta flexibilidad le permite definir y modificar las experiencias de identidad con cambios mínimos o ningún cambio en el código. La aplicación desencadena un flujo de usuario mediante una solicitud de autenticación HTTP estándar que incluye un parámetro de flujo de usuario. Se recibe un [token](tokens-overview.md) personalizado como respuesta.

En los ejemplos siguientes se muestra el parámetro de cadena de consulta "p" que especifica el flujo de usuario que se usará:

```
https://contosob2c.b2clogin.com/contosob2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e      // Your registered Application ID
&redirect_uri=https%3A%2F%2Flocalhost%3A44321%2F    // Your registered Reply URL, url encoded
&response_mode=form_post                            // 'query', 'form_post' or 'fragment'
&response_type=id_token
&scope=openid
&nonce=dummy
&state=12345                                        // Any value provided by your application
&p=b2c_1_siup                                       // Your sign-up user flow
```

```
https://contosob2c.b2clogin.com/contosob2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e      // Your registered Application ID
&redirect_uri=https%3A%2F%2Flocalhost%3A44321%2F    // Your registered Reply URL, url encoded
&response_mode=form_post                            // 'query', 'form_post' or 'fragment'
&response_type=id_token
&scope=openid
&nonce=dummy
&state=12345                                        // Any value provided by your application
&p=b2c_1_siin                                       // Your sign-in user flow
```

## <a name="user-flow-versions"></a>Versiones del flujo de usuario

Azure AD B2C incluye varios tipos de flujos de usuario:

- **Registro e inicio de sesión**: controla las experiencias de registro y de inicio de sesión con una sola configuración. A los usuarios se les lleva por el proceso correcto según el contexto. También se incluyen flujos de usuario de **registro** o **inicio de sesión** independientes. Sin embargo, suele recomendarse el flujo de usuario de registro o inicio de sesión combinado.
- **Edición de perfiles**: permite a los usuarios editar su información de perfil.
- **Restablecimiento de contraseña**: le permite configurar si es posible o no para los usuarios restablecer su contraseña y de qué forma pueden hacerlo.

La mayoría de los tipos de flujo de usuario cuentan con una versión **Recomendada** y una versión **Estándar**. Para obtener detalles, consulte [Versiones del flujo de usuario](user-flow-versions.md).

> [!IMPORTANT]
> Si ha trabajado con flujos de usuario en Azure AD B2C antes, observará que hemos cambiado nuestra forma de hacer referencia a las versiones del flujo de usuario. Anteriormente, se ofrecían las versiones V1 (para entornos de producción) y V1.1 y V2 (preliminares). Ahora, hemos consolidado los flujos de usuario en dos versiones:
>
>- Los flujos de usuario **Recomendados** son las nuevas versiones preliminares de los flujos de usuario. Se prueban exhaustivamente y combinan todas las características de las versiones **V2** y **V1.1** heredadas. A partir de ahora, los nuevos flujos de usuario recomendados se mantendrán y actualizarán. Una vez que se traslade a estos nuevos flujos de usuario recomendados, tendrá acceso a las nuevas características a medida que se vayan lanzando.
>- Los flujos de usuario **Estándar**, anteriormente conocidos como **V1**, son flujos de usuario disponibles con carácter general y para entornos de producción. Si sus flujos de usuario son críticos y dependen de versiones altamente estables, puede seguir usando flujos de usuario estándar, sabiendo que estas versiones no se mantendrán ni actualizarán.
>
>Todas las versiones preliminares de flujos de usuario heredadas (V1.1 y V2) estarán en proceso de desuso el **1 de agosto de 2021**. Siempre que sea posible, recomendamos encarecidamente que [cambie a los nuevos flujos de usuario **Recomendados**](user-flow-versions.md#how-to-switch-to-a-new-recommended-user-flow) lo antes posible para que pueda aprovechar siempre las últimas características y actualizaciones.

## <a name="linking-user-flows"></a>Vinculación de los flujos de usuario

Un flujo de usuario de **inicio de sesión o de registro** con cuentas locales incluye el vínculo **¿Olvidó la contraseña?** en la primera página de la experiencia. Al hacer clic en este vínculo, no se desencadena automáticamente ningún flujo de usuario de restablecimiento de contraseña,

sino que se devuelve a la aplicación el código de error `AADB2C90118`. La aplicación debe controlar este código de error mediante la ejecución de un flujo de usuario específico que restablezca la contraseña. Para ver un ejemplo, eche un vistazo a un [ejemplo sencillo de ASP.NET](https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIDConnect-DotNet-SUSI) que muestra la vinculación de flujos de usuario.

## <a name="email-address-storage"></a>Almacenamiento de la dirección de correo electrónico

Es posible que se necesite una dirección de correo electrónico como parte de un flujo de usuario. Si el usuario se autentica con un proveedor de identidades de redes sociales, la dirección de correo electrónico se almacena en la propiedad **otherMails**. Si una cuenta local se basa en un nombre de usuario, la dirección de correo electrónico se almacena en una propiedad de detalles de autenticación sólida. Si una cuenta local se basa en una dirección de correo electrónico, dicha dirección se almacena en la propiedad **signInNames**.

No se garantiza que la dirección de correo electrónico pueda comprobarse en cualquiera de estos casos. Un administrador de inquilinos puede deshabilitar la comprobación de correo electrónico en las directivas básicas para las cuentas locales. Incluso si la comprobación de la dirección de correo electrónico está habilitada, las direcciones no se comprueban si provienen de un proveedor de identidades de redes sociales y aún no se cambian.

Solo las propiedades **otherMails** y **signInNames** se exponen a través de Microsoft Graph API. La dirección de correo electrónico en la propiedad de detalles de autenticación sólida no está disponible.

## <a name="next-steps"></a>Pasos siguientes

Para crear los flujos de usuario recomendados, siga las instrucciones en el [Tutorial: Creación de un flujo de usuario](tutorial-create-user-flows.md).
