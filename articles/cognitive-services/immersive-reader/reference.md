---
title: Referencia del SDK del Lector inmersivo
titleSuffix: Azure Cognitive Services
description: El SDK del Lector inmersivo contiene una biblioteca de JavaScript que permite integrar dicho lector en la aplicación.
services: cognitive-services
author: metanMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: reference
ms.date: 06/20/2019
ms.author: metan
ms.openlocfilehash: 73322cdee151969e6e765690284bbffc1c871f4e
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90090200"
---
# <a name="immersive-reader-javascript-sdk-reference-v11"></a>Referencia del SDK de JavaScript de Lector inmersivo (v 1.1)

El SDK del Lector inmersivo contiene una biblioteca de JavaScript que permite integrar dicho lector en la aplicación.

## <a name="functions"></a>Functions

El SDK expone las funciones:

- [`ImmersiveReader.launchAsync(token, subdomain, content, options)`](#launchasync)

- [`ImmersiveReader.close()`](#close)

- [`ImmersiveReader.renderButtons(options)`](#renderbuttons)

<br>

## <a name="launchasync"></a>launchAsync

Inicia el Lector inmersivo dentro de un `iframe` en la aplicación web. Tenga en cuenta que el tamaño del contenido está limitado a un máximo de 50 MB.

```typescript
launchAsync(token: string, subdomain: string, content: Content, options?: Options): Promise<LaunchResponse>;
```

#### <a name="launchasync-parameters"></a>Parámetros de launchAsync

| Nombre | Tipo | Descripción |
| ---- | ---- |------------ |
| `token` | string | Token de autenticación de Azure AD Consulte [Procedimiento: Creación de un recurso del Lector inmersivo](./how-to-create-immersive-reader.md) para más detalles. |
| `subdomain` | string | El subdominio personalizado del recurso Lector inmersivo en Azure. Consulte [Procedimiento: Creación de un recurso del Lector inmersivo](./how-to-create-immersive-reader.md) para más detalles. |
| `content` | [Contenido](#content) | Un objeto que contiene el contenido que se mostrará en el Lector inmersivo. |
| `options` | [Opciones](#options) | Opciones para configurar ciertos comportamientos del Lector inmersivo. Opcional. |

#### <a name="returns"></a>Devuelve

Devuelve `Promise<LaunchResponse>`, que se resuelve cuando el lector inmersivo se carga. El objeto `Promise` se resuelve en un objeto [`LaunchResponse`](#launchresponse).

#### <a name="exceptions"></a>Excepciones

El valor devuelto de `Promise` se rechazará con un objeto [`Error`](#error) si el Lector inmersivo no se puede cargar. Para más información, consulte los [códigos de error](#error-codes).

<br>

## <a name="close"></a>close

Cierra el lector inmersivo.

Un caso de uso de ejemplo de esta función se produce si el botón para salir se oculta al establecer ```hideExitButton: true``` en [options](#options). Después, un botón distinto (por ejemplo, la flecha atrás de un encabezado para dispositivos móviles) puede llamar a esta función ```close``` cuando se hace clic en ella.

```typescript
close(): void;
```

<br>

## <a name="immersive-reader-launch-button"></a>Botón de inicio del Lector inmersivo

El SDK proporciona el estilo predeterminado para el botón para iniciar el Lector inmersivo. Use el atributo de clase `immersive-reader-button` para habilitar este estilo. Consulte [Procedimiento: Personalización del botón del Lector inmersivo](./how-to-customize-launch-button.md) para más detalles.

```html
<div class='immersive-reader-button'></div>
```

#### <a name="optional-attributes"></a>Atributos opcionales

Use los siguientes atributos para configurar la apariencia y sensibilidad del botón.

| Atributo | Descripción |
| --------- | ----------- |
| `data-button-style` | Establece el estilo del botón. Puede ser `icon`, `text` o `iconAndText`. Su valor predeterminado es `icon`. |
| `data-locale` | Establece la configuración regional. Por ejemplo, `en-US` o `fr-FR`. El valor predeterminado es el idioma inglés `en`. |
| `data-icon-px-size` | Establece el tamaño del icono en píxeles. El valor predeterminado es 20 píxeles. |

<br>

## <a name="renderbuttons"></a>renderButtons

La función ```renderButtons``` no es necesaria si usa la guía [Procedimiento: Personalización del botón del Lector inmersivo](./how-to-customize-launch-button.md).

Esta función crea los estilos y actualiza los elementos del botón de lector inmersivo del documento. Si se proporciona ```options.elements```, los botones se representarán dentro de cada elemento proporcionado en ```options.elements```. El uso del parámetro ```options.elements``` resulta útil cuando en varias secciones del documento se inicia el Lector inmersivo y se quiere una manera simplificada de presentar varios botones con el mismo estilo o se quiere representar los botones con un modelo de diseño sencillo y coherente. Para usar esta función con el parámetro [renderButtons options](#renderbuttons-options), llame a ```ImmersiveReader.renderButtons(options: RenderButtonsOptions);``` en la carga de la página, como se muestra en el fragmento de código siguiente. De lo contrario, los botones se representarán dentro de los elementos del documento que tengan la clase ```immersive-reader-button```, tal como se muestra en [Procedimiento: Personalización del botón del Lector inmersivo](./how-to-customize-launch-button.md).

```typescript
// This snippet assumes there are two empty div elements in
// the page HTML, button1 and button2.
const btn1: HTMLDivElement = document.getElementById('button1');
const btn2: HTMLDivElement = document.getElementById('button2');
const btns: HTMLDivElement[] = [btn1, btn2];
ImmersiveReader.renderButtons({elements: btns});
```

Consulte el apartado [Atributos opcionales](#optional-attributes) anterior, para más opciones de representación. Para usar estas opciones, agregue cualquiera de los atributos de opción a cada ```HTMLDivElement``` en el código HTML de la página.

```typescript
renderButtons(options?: RenderButtonsOptions): void;
```

#### <a name="renderbuttons-parameters"></a>Parámetros de renderButtons

| Nombre | Tipo | Descripción |
| ---- | ---- |------------ |
| `options` | [Opciones de renderButtons](#renderbuttons-options) | Opciones para configurar ciertos comportamientos de la función renderButtons. Opcional. |

### <a name="renderbuttons-options"></a>Opciones de renderButtons

Opciones para representar los botones del lector inmersivo.

```typescript
{
    elements: HTMLDivElement[];
}
```

#### <a name="renderbuttons-options-parameters"></a>Parámetros de opciones de renderButtons

| Configuración | Tipo | Descripción |
| ------- | ---- | ----------- |
| Elementos | HTMLDivElement[] | Elementos para representar los botones del Lector inmersivo. |

##### `-elements`
```Parameters
Type: HTMLDivElement[]
Required: false
```

<br>

## <a name="launchresponse"></a>LaunchResponse

Contiene la respuesta de la llamada a `ImmersiveReader.launchAsync`. Tenga en cuenta que se puede acceder a una referencia al elemento `iframe` que contiene el Lector inmersivo a través de `container.firstChild`.

```typescript
{
    container: HTMLDivElement;
    sessionId: string;
}
```

#### <a name="launchresponse-parameters"></a>Parámetros de LaunchResponse

| Configuración | Tipo | Descripción |
| ------- | ---- | ----------- |
| contenedor | HTMLDivElement | Elemento HTML que contiene el iframe del Lector inmersivo. |
| sessionID | String | Identificador único global de esta sesión, que se usa para la depuración. |
 
## <a name="error"></a>Error

Contiene información sobre un error.

```typescript
{
    code: string;
    message: string;
}
```

#### <a name="error-parameters"></a>Parámetros de error

| Configuración | Tipo | Descripción |
| ------- | ---- | ----------- |
| código | String | Uno de un conjunto de códigos de error. Consulte [Códigos de error](#error-codes). |
| message | String | Representación legible del error. |

#### <a name="error-codes"></a>Códigos de error

| Código | Descripción |
| ---- | ----------- |
| BadArgument | El argumento proporcionado no es válido, consulte el parámetro `message` del [Error](#error). |
| Tiempo de espera | El Lector inmersivo no se pudo cargar en el tiempo de espera especificado. |
| TokenExpired | El token suministrado en caché ha expirado. |
| Limitado | Se ha superado el límite de frecuencia de llamadas. |

<br>

## <a name="types"></a>Tipos

### <a name="content"></a>Contenido

Contiene el contenido que se mostrará en el Lector inmersivo.

```typescript
{
    title?: string;
    chunks: Chunk[];
}
```

#### <a name="content-parameters"></a>Parámetros de contenido

| Nombre | Tipo | Descripción |
| ---- | ---- |------------ |
| title | String | Texto del título que se muestra en la parte superior del Lector inmersivo (opcional) |
| fragmentos | [Chunk[]](#chunk) | Matriz de fragmentos |

##### `-title`
```Parameters
Type: String
Required: false
Default value: "Immersive Reader" 
```

##### `-chunks`
```Parameters
Type: Chunk[]
Required: true
Default value: null 
```

<br>

### <a name="chunk"></a>Fragmento

Un único fragmento de datos, que se pasará al contenido del lector inmersivo.

```typescript
{
    content: string;
    lang?: string;
    mimeType?: string;
}
```

#### <a name="chunk-parameters"></a>Parámetros de fragmentos

| Nombre | Tipo | Descripción |
| ---- | ---- |------------ |
| contenido | String | Cadena con el contenido enviado al Lector inmersivo. |
| lang | String | Idioma del texto, el valor está en formato de etiqueta de idioma IETF BCP 47, por ejemplo, en es-ES. El idioma se detectará automáticamente si no se especifica. Consulte [Idiomas admitidos](#supported-languages). |
| mimeType | string | Se admiten los formatos de texto sin formato, MathML, HTML y Microsoft Word DOCX. Para obtener más información, consulte [Tipos MIME admitidos](#supported-mime-types). |

##### `-content`
```Parameters
Type: String
Required: true
Default value: null 
```

##### `-lang`
```Parameters
Type: String
Required: false
Default value: Automatically detected 
```

##### `-mimeType`
```Parameters
Type: String
Required: false
Default value: "text/plain"
```

#### <a name="supported-mime-types"></a>Tipos MIME compatibles

| Tipo MIME | Descripción |
| --------- | ----------- |
| text/plain | Texto sin formato. |
| text/html | Contenido HTML. [Más información](#html-support)|
| application/mathml+xml | Lenguaje de marcado matemático (MathML). [Más información](./how-to/display-math.md).
| application/vnd.openxmlformats-officedocument.wordprocessingml.document | Documento de Microsoft Word con formato .docx.


<br>

## <a name="options"></a>Opciones

Contiene propiedades que configuran ciertos comportamientos del Lector inmersivo.

```typescript
{
    uiLang?: string;
    timeout?: number;
    uiZIndex?: number;
    useWebview?: boolean;
    onExit?: () => any;
    allowFullscreen?: boolean;
    hideExitButton?: boolean;
    cookiePolicy?: CookiePolicy;
    disableFirstRun?: boolean;
    readAloudOptions?: ReadAloudOptions;
    translationOptions?: TranslationOptions;
    displayOptions?: DisplayOptions;
    preferences?: string;
    onPreferencesChanged?: (value: string) => any;
    customDomain?: string;
}
```

#### <a name="options-parameters"></a>Parámetros de opciones

| Nombre | Tipo | Descripción |
| ---- | ---- |------------ |
| uiLang | String | Idioma de la interfaz de usuario, el valor está en formato de etiqueta de idioma IETF BCP 47, por ejemplo, en es-ES. Si no se especifica, el valor predeterminado es el idioma del explorador. |
| timeout | Number | Duración (en milisegundos) antes de que [launchAsync](#launchasync) produzca un error de tiempo de espera (el valor predeterminado es 15 000 ms). Este tiempo de espera solo se aplica al lanzamiento inicial de la página del lector, donde se observa que se ha realizado correctamente cuando se abre la página del lector y se inicia el indicador giratorio. No es necesario ajustar el tiempo de espera. |
| uiZIndex | Number | Índice Z del iframe que se va a crear (el valor predeterminado es 1000). |
| useWebview | Boolean| Use una etiqueta de vista web en lugar de un iframe para la compatibilidad con aplicaciones de Chrome (el valor predeterminado es false). |
| onExit | Función | Se ejecuta cuando se cierra el Lector inmersivo. |
| allowFullscreen | Boolean | La capacidad de alternar a la pantalla completa (el valor predeterminado es true). |
| hideExitButton | Boolean | Indica si se va a ocultar o no la flecha del botón de salida del Lector inmersivo (el valor predeterminado es false). Solo debe ser true si hay un mecanismo alternativo proporcionado para salir del Lector inmersivo (por ejemplo, la flecha atrás de la barra de herramientas móvil). |
| cookiePolicy | [CookiePolicy](#cookiepolicy-options) | Configuración para el uso de cookies del Lector inmersivo (el valor predeterminado es *CookiePolicy.Disable*). Es responsabilidad de la aplicación host obtener cualquier consentimiento del usuario necesario según la directiva de cumplimiento de cookies de Europa. Consulte [Opciones de la directiva de cookies](#cookiepolicy-options). |
| disableFirstRun | Boolean | Deshabilita la experiencia de primera ejecución. |
| readAloudOptions | [ReadAloudOptions](#readaloudoptions) | Opciones para configurar la lectura en voz alta. |
| translationOptions | [TranslationOptions](#translationoptions) | Opciones para configurar la traducción. |
| displayOptions | [DisplayOptions](#displayoptions) | Opciones para configurar el tamaño del texto, la fuente, etc. |
| preferences | String | Cadena devuelta por onPreferencesChanged que representa las preferencias del usuario en el Lector inmersivo. Consulte [Parámetros de configuración](#settings-parameters) y [Procedimiento: Almacenamiento de las preferencias de usuario](./how-to-store-user-preferences.md) para más información. |
| onPreferencesChanged | Función | Se ejecuta cuando las preferencias del usuario han cambiado. Consulte [Procedimiento: Almacenamiento de las preferencias de usuario](./how-to-store-user-preferences.md) para más información. |
| customDomain | String | Reservado para uso interno. Dominio personalizado en el que se hospeda la aplicación web del Lector inmersivo (el valor predeterminado es null). |

##### `-uiLang`
```Parameters
Type: String
Required: false
Default value: User's browser language 
```

##### `-timeout`
```Parameters
Type: Number
Required: false
Default value: 15000
```

##### `-uiZIndex`
```Parameters
Type: Number
Required: false
Default value: 1000
```

##### `-onExit`
```Parameters
Type: Function
Required: false
Default value: null
```

##### `-preferences`

> [!CAUTION]
> **IMPORTANTE** No intente cambiar mediante programación los valores de la cadena `-preferences` enviada a la aplicación del Lector inmersivo o desde ella, ya que puede provocar un comportamiento inesperado y dar lugar a una experiencia de usuario degradada para los clientes.

```Parameters
Type: String
Required: false
Default value: null
```

##### `-onPreferencesChanged`
```Parameters
Type: Function
Required: false
Default value: null
```

##### `-customDomain`
```Parameters
Type: String
Required: false
Default value: null
```

<br>

## <a name="readaloudoptions"></a>ReadAloudOptions

```typescript
type ReadAloudOptions = {
    voice?: string;
    speed?: number;
    autoplay?: boolean;
};
```

#### <a name="readaloudoptions-parameters"></a>Parámetros de ReadAloudOptions

| Nombre | Tipo | Descripción |
| ---- | ---- |------------ |
| voice | String | Voz, "mujer" u "hombre". No todos los idiomas admiten ambos géneros. |
| velocidad | Number | La velocidad de reproducción debe ser entre 0,5 y 2,5, ambos inclusive. |
| autoPlay | Boolean | Se inicia automáticamente la lectura en voz alta cuando se carga el Lector inmersivo. |

##### `-voice`
```Parameters
Type: String
Required: false
Default value: "Female" or "Male" (determined by language) 
Values available: "Female", "Male"
```

##### `-speed`
```Parameters
Type: Number
Required: false
Default value: 1
Values available: 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2, 2.25, 2.5
```

> [!NOTE]
> Debido a las limitaciones del explorador, la reproducción automática no se admite en Safari.

<br>

## <a name="translationoptions"></a>TranslationOptions

```typescript
type TranslationOptions = {
    language: string;
    autoEnableDocumentTranslation?: boolean;
    autoEnableWordTranslation?: boolean;
};
```

#### <a name="translationoptions-parameters"></a>Parámetros de TranslationOptions

| Nombre | Tipo | Descripción |
| ---- | ---- |------------ |
| language | String | Establece el idioma de traducción, el valor está en formato de etiqueta de idioma IETF BCP 47, por ejemplo, fr-FR, es-MX, ZH-Hans-CN. Necesario para habilitar automáticamente la traducción de documentos o palabras. |
| autoEnableDocumentTranslation | Boolean | Traducción automática de todo el documento. |
| autoEnableWordTranslation | Boolean | Habilitación automática de la traducción de palabras. |

##### `-language`
```Parameters
Type: String
Required: true
Default value: null 
Values available: See the Supported Languages section
```

<br>

## <a name="displayoptions"></a>DisplayOptions

```typescript
type DisplayOptions = {
    textSize?: number;
    increaseSpacing?: boolean;
    fontFamily?: string;
};
```

#### <a name="displayoptions-parameters"></a>Parámetros de DisplayOptions

| Nombre | Tipo | Descripción |
| ---- | ---- |------------ |
| textSize | Number | Establece el tamaño de texto elegido. |
| increaseSpacing | Boolean | Establece si el espaciado de texto está activado o desactivado. |
| fontFamily | String | Establece la fuente elegida ("Calibri", "ComicSans" o "Sitka"). |

##### `-textSize`
```Parameters
Type: Number
Required: false
Default value: 20, 36 or 42 (Determined by screen size)
Values available: 14, 20, 28, 36, 42, 48, 56, 64, 72, 84, 96
```

##### `-fontFamily`
```Parameters
Type: String
Required: false
Default value: "Calibri"
Values available: "Calibri", "Sitka", "ComicSans"
```

<br>

## <a name="cookiepolicy-options"></a>Opciones de CookiePolicy

```typescript
enum CookiePolicy { Disable, Enable }
```

**La configuración que se muestra a continuación solo es informativa**. El Lector inmersivo almacena su configuración, o preferencias de usuario, en cookies. La opción *cookiePolicy* **deshabilita** el uso de cookies de manera predeterminada para cumplir con las leyes de cumplimiento de cookies de Europa. Si desea volver a habilitar las cookies y restaurar la funcionalidad predeterminada para las preferencias de usuario del Lector inmersivo, deberá asegurarse de que el sitio web o la aplicación obtienen el consentimiento adecuado del usuario para habilitar las cookies. Después, para volver a habilitar las cookies en el Lector inmersivo, debe establecer explícitamente la opción *cookiePolicy* en *CookiePolicy.Enable* al iniciar el Lector inmersivo. En la tabla siguiente se describe qué configuración almacena el Lector inmersivo en su cookie cuando está habilitada la opción *cookiePolicy*.

#### <a name="settings-parameters"></a>Parámetros de configuración

| Configuración | Tipo | Descripción |
| ------- | ---- | ----------- |
| textSize | Number | Establece el tamaño de texto elegido. |
| fontFamily | String | Establece la fuente elegida ("Calibri", "ComicSans" o "Sitka"). |
| textSpacing | Number | Establece si el espaciado de texto está activado o desactivado. |
| formattingEnabled | Boolean | Establece si el formato HTML de texto está activado o desactivado. |
| tema | String | Establece el tema elegido (por ejemplo, "Claro", "Oscuro"...). |
| syllabificationEnabled | Boolean | Establece si la silabación está activada o desactivada. |
| nounHighlightingEnabled | Boolean | Establece si el resaltado de sustantivos está activado o desactivado. |
| nounHighlightingColor | String | Establece el color elegido para el resaltado de sustantivos. |
| verbHighlightingEnabled | Boolean | Establece si el resaltado de verbos está activado o desactivado. |
| verbHighlightingColor | String | Establece el color elegido para el resaltado de verbos. |
| adjectiveHighlightingEnabled | Boolean | Establece si el resaltado de adjetivos está activado o desactivado. |
| adjectiveHighlightingColor | String | Establece el color elegido para el resaltado de adjetivos. |
| adverbHighlightingEnabled | Boolean | Establece si el resaltado de adverbios está activado o desactivado. |
| adverbHighlightingColor | String | Establece el color elegido para el resaltado de adverbios. |
| pictureDictionaryEnabled | Boolean | Establece si el Diccionario de imágenes está activado o desactivado. |
| posLabelsEnabled | Boolean | Establece si la etiqueta de texto de superíndice de cada parte de voz resaltada está activada o desactivada.  |

<br>

## <a name="supported-languages"></a>Idiomas compatibles

La característica de traducción del Lector inmersivo admite muchos idiomas. Consulte [este artículo](https://www.onenote.com/learningtools/languagesupport) para obtener más detalles.

<br>

## <a name="html-support"></a>Compatibilidad con HTML

Cuando se habilita el formato, el siguiente contenido se representará como HTML en el Lector inmersivo.

| HTML | Contenido admitido |
| --------- | ----------- |
| Estilos de fuente | Negrita, cursiva, subrayado, código, tachado, superíndice, subíndice |
| Listas sin ordenar | Disco, círculo, cuadrado |
| Listas ordenadas | Decimal, alfabético en mayúsculas, alfabético en minúsculas, romano superior, romano inferior |

Las etiquetas no admitidas se representarán de la misma forma. Las imágenes y las tablas actualmente no se admiten.

<br>

## <a name="browser-support"></a>Compatibilidad con exploradores

Use las versiones más recientes de los siguientes exploradores para disfrutar de la mejor experiencia con el Lector inmersivo.

* Microsoft Edge
* Internet Explorer 11
* Google Chrome
* Mozilla Firefox
* Apple Safari

<br>

## <a name="next-steps"></a>Pasos siguientes

* Explorar el [SDK del Lector inmersivo en GitHub](https://github.com/microsoft/immersive-reader-sdk)
* [Inicio rápido: Creación de una aplicación web que inicia el Lector inmersivo (C#)](./quickstarts/client-libraries.md?pivots=programming-language-csharp)
