---
title: Versiones de diseño de página
titleSuffix: Azure AD B2C
description: Historial de versiones de diseño de página de personalización de interfaz de usuario en directivas personalizadas.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 08/24/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 868d99a82009dc8545fc24ad1cfa1da3959da131
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88852073"
---
# <a name="page-layout-versions"></a>Versiones de diseño de página

Los paquetes de diseño de página se actualizan periódicamente para incluir correcciones y mejoras en sus elementos de la página. El registro de cambios siguiente especifica los cambios introducidos en cada versión.

[!INCLUDE [active-directory-b2c-public-preview](../../includes/active-directory-b2c-public-preview.md)]

## <a name="self-asserted-page-selfasserted"></a>Página autoafirmada (selfasserted)

**2.1.0**

- Correcciones de localización y accesibilidad.

**2.0.0**

- Se ha agregado compatibilidad para los [controles de visualización](display-controls.md) en las directivas personalizadas.

**1.2.0**

- Los campos de nombre de usuario/correo electrónico y contraseña ahora usan el elemento HTML `form` para permitir que Microsoft Edge e Internet Explorer (IE) guarden correctamente esta información.
- Se ha agregado un retraso configurable en la validación de entradas de usuario para mejorar la experiencia del usuario.
- Correcciones de accesibilidad
- Ahora puede agregar el atributo `data-preload="true"` [en las etiquetas HTML](custom-policy-ui-customization.md#guidelines-for-using-custom-page-content) para controlar el orden de carga de CSS y JavaScript.
  - Cargue los archivos CSS vinculados al mismo tiempo que la plantilla HTML para que no "vacile" durante la carga de los archivos.
  - Controle el orden en el que se capturan y ejecutan las etiquetas `script` antes de la carga de la página.
- El campo de correo electrónico es ahora `type=email` y los teclados para móviles proporcionarán las sugerencias correctas.
- Compatibilidad con la traducción de Chrome

**1.1.0**

- Se ha quitado la alerta de cancelación.
- Clase CSS para elementos de error.
- Se ha mejorado la lógica para mostrar u ocultar errores.
- Se ha quitado la hoja CSS predeterminada.

**1.0.0**

- Versión inicial

## <a name="unified-sign-in-sign-up-page-with-password-reset-link-unifiedssp"></a>Página de registro de inicio de sesión unificado con el vínculo de restablecimiento de contraseña (unifiedssp)

**2.1.0**

- Se ha agregado compatibilidad con varios vínculos de registro.
- Se ha agregado compatibilidad con la validación de entradas de usuario según las reglas de predicado definidas en la directiva.

**1.2.0**

- Los campos de nombre de usuario/correo electrónico y contraseña ahora usan el elemento HTML `form` para permitir que Microsoft Edge e Internet Explorer (IE) guarden correctamente esta información.
- Correcciones de accesibilidad
- Ahora puede agregar el atributo `data-preload="true"` [en las etiquetas HTML](custom-policy-ui-customization.md#guidelines-for-using-custom-page-content) para controlar el orden de carga de CSS y JavaScript.
  - Cargue los archivos CSS vinculados al mismo tiempo que la plantilla HTML para que no "vacile" durante la carga de los archivos.
  - Controle el orden en el que se capturan y ejecutan las etiquetas `script` antes de la carga de la página.
- El campo de correo electrónico es ahora `type=email` y los teclados para móviles proporcionarán las sugerencias correctas.
- Compatibilidad con la traducción de Chrome

**1.1.0**

- Se ha agregado el control para mantener la sesión iniciada (KMSI)

**1.0.0**

- Versión inicial

## <a name="mfa-page-multifactor"></a>Página de MFA (multifactor)

**1.2.1**

- Correcciones de accesibilidad en plantillas predeterminadas

**1.2.0**

- Correcciones de accesibilidad
- Ahora puede agregar el atributo `data-preload="true"` [en las etiquetas HTML](custom-policy-ui-customization.md#guidelines-for-using-custom-page-content) para controlar el orden de carga de CSS y JavaScript.
  - Cargue los archivos CSS vinculados al mismo tiempo que la plantilla HTML para que no "vacile" durante la carga de los archivos.
  - Controle el orden en el que se capturan y ejecutan las etiquetas `script` antes de la carga de la página.
- El campo de correo electrónico es ahora `type=email` y los teclados para móviles proporcionarán las sugerencias correctas.
- Compatibilidad con la traducción de Chrome

**1.1.0**

- Se ha quitado el botón “Confirmar código”.
- El campo de entrada del código ahora solo acepta la entrada de hasta seis (6) caracteres.
- La página intentará comprobar el código introducido automáticamente cuando se introduzca un código de 6 dígitos, sin tener que hacer clic en ningún botón.
- Si el código es incorrecto, el campo de entrada se borra automáticamente.
- Después de tres (3) intentos con un código incorrecto, B2C reenvía un error al usuario de confianza.
- Correcciones de accesibilidad
- Se ha quitado la hoja CSS predeterminada.

**1.0.0**

- Versión inicial

## <a name="exception-page-globalexception"></a>Página de excepciones (globalexception)

**1.2.0**

- Correcciones de accesibilidad
- Ahora puede agregar el atributo `data-preload="true"` [en las etiquetas HTML](custom-policy-ui-customization.md#guidelines-for-using-custom-page-content) para controlar el orden de carga de CSS y JavaScript.
  - Cargue los archivos CSS vinculados al mismo tiempo que la plantilla HTML para que no "vacile" durante la carga de los archivos.
  - Controle el orden en el que se capturan y ejecutan las etiquetas `script` antes de la carga de la página.
- El campo de correo electrónico es ahora `type=email` y los teclados para móviles proporcionarán las sugerencias correctas.
- Compatibilidad con la traducción de Chrome

**1.1.0**

- Corrección de accesibilidad.
- Se ha quitado el mensaje predeterminado cuando no hay ningún contacto de la directiva.
- Se ha quitado la hoja CSS predeterminada.

**1.0.0**

- Versión inicial

## <a name="other-pages-providerselection-claimsconsent-unifiedssd"></a>Otras páginas (ProviderSelection, ClaimsConsent, UnifiedSSD)

**1.2.0**

- Correcciones de accesibilidad
- Ahora puede agregar el atributo `data-preload="true"` [en las etiquetas HTML](custom-policy-ui-customization.md#guidelines-for-using-custom-page-content) para controlar el orden de carga de CSS y JavaScript.
  - Cargue los archivos CSS vinculados al mismo tiempo que la plantilla HTML para que no "vacile" durante la carga de los archivos.
  - Controle el orden en el que se capturan y ejecutan las etiquetas `script` antes de la carga de la página.
- El campo de correo electrónico es ahora `type=email` y los teclados para móviles proporcionarán las sugerencias correctas.
- Compatibilidad con la traducción de Chrome

**1.0.0**

- Versión inicial

## <a name="next-steps"></a>Pasos siguientes

Para obtener detalles sobre cómo personalizar la interfaz de usuario de las aplicaciones en directivas personalizadas, vea [Personalización de la interfaz de usuario de la aplicación mediante una directiva personalizada](custom-policy-ui-customization.md).
