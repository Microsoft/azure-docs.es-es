---
title: Escritura de expresiones para asignaciones de atributos en Azure Active Directory | Microsoft Azure
description: Obtenga información sobre cómo usar asignaciones de expresiones para transformar valores de atributos en un formato aceptable durante el aprovisionamiento automático de objetos de aplicaciones SaaS en Azure Active Directory.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/21/2019
ms.author: mimart
ms.collection: M365-identity-device-management
ms.openlocfilehash: ec1994169891d5256436ac4de741339c865bb268
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824642"
---
# <a name="writing-expressions-for-attribute-mappings-in-azure-active-directory"></a>Escritura de expresiones para la asignación de atributos en Azure Active Directory
Al configurar el aprovisionamiento para una aplicación SaaS, uno de los tipos de asignaciones de atributos que puede especificar es una asignación de expresiones. En estos casos, debe escribir una expresión similar a un script que permite transformar los datos de los usuarios en formatos más aceptables para la aplicación SaaS.

## <a name="syntax-overview"></a>Información general sobre la sintaxis
La sintaxis de expresiones para asignaciones de atributos recuerda a las funciones de Visual Basic para Aplicaciones (VBA).

* Toda la expresión se tiene que definir en términos de funciones, que constan de un nombre seguido de argumentos entre paréntesis:  <br>
  *FunctionName(`<<argument 1>>`,`<<argument N>>`)*
* Es posible anidar funciones dentro de otras. Por ejemplo: <br> *FunctionOne(FunctionTwo(`<<argument1>>`))*
* Puede transformar tres tipos diferentes de argumentos en funciones:
  
  1. Atributos, que deben ir entre corchetes. Por ejemplo: [NombreAtributo]
  2. Constantes de cadena, que deben ir entre comillas. Por ejemplo: "Estados Unidos"
  3. Otras funciones. Por ejemplo: FunctionOne(`<<argument1>>`, FunctionTwo(`<<argument2>>`))
* Para las constantes de cadena, si necesita una barra diagonal inversa (\) o comillas dobles (") en la cadena, se deben convertirse con el símbolo de barra diagonal inversa (\). Por ejemplo: "Nombre de la empresa: \\"Contoso\\""

## <a name="list-of-functions"></a>Lista de funciones
[Append](#append) &nbsp;&nbsp;&nbsp;&nbsp; [FormatDateTime](#formatdatetime) &nbsp;&nbsp;&nbsp;&nbsp; [Join](#join) &nbsp;&nbsp;&nbsp;&nbsp; [Mid](#mid) &nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [NormalizeDiacritics](#normalizediacritics) [Not](#not) &nbsp;&nbsp;&nbsp;&nbsp; [Replace](#replace) &nbsp;&nbsp;&nbsp;&nbsp; [SelectUniqueValue](#selectuniquevalue)&nbsp;&nbsp;&nbsp;&nbsp; [SingleAppRoleAssignment](#singleapproleassignment)&nbsp;&nbsp;&nbsp;&nbsp; [Split](#split)&nbsp;&nbsp;&nbsp;&nbsp;[StripSpaces](#stripspaces) &nbsp;&nbsp;&nbsp;&nbsp; [Switch](#switch)&nbsp;&nbsp;&nbsp;&nbsp; [ToLower](#tolower)&nbsp;&nbsp;&nbsp;&nbsp; [ToUpper](#toupper)

- - -
### <a name="append"></a>Append
**Función:**<br>  Append(source, suffix)

**Descripción:**<br>  adopta un valor de la cadena de origen y anexa el sufijo al final de la misma.

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Normalmente el nombre del atributo del objeto de origen. |
| **suffix** |Obligatorio |String |La cadena que se va a anexar al final del valor de origen. |

- - -
### <a name="formatdatetime"></a>FormatDateTime
**Función:**<br>  FormatDateTime(source, inputFormat, outputFormat)

**Descripción:**<br>  adopta una cadena de fecha en un formato y la convierte a un formato distinto.

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Normalmente el nombre del atributo del objeto de origen. |
| **inputFormat** |Obligatorio |string |Formato esperado del valor de origen. Para conocer los formatos admitidos, consulte [https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx). |
| **outputFormat** |Obligatorio |String |Formato de la fecha de salida. |

- - -
### <a name="join"></a>Unir
**Función:**<br>  Join(separator, source1, source2, …)

**Descripción:**<br> Join() es similar a Append(), excepto en que puede combinar varios valores de cadena de **source** en una sola cadena, y cada valor estará separado por una cadena de **separator**.

Si uno de los valores de origen es un atributo multivalor, cada valor de ese atributo se unirá, separado por el valor del separador.

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **separator** |Obligatorio |String |Cadena utilizada para separar los valores de origen cuando se concatenan en una sola cadena. Puede ser "" si no es necesario ningún separador. |
| **source1  … sourceN** |Obligatorio, número variable de veces |string |Valores de cadena que se van a agrupar. |

- - -
### <a name="mid"></a>Mid
**Función:**<br>  Mid(source, start, length)

**Descripción:**<br>  devuelve una subcadena del valor de origen. Una subcadena es una cadena que contiene sólo algunos de los caracteres de la cadena de origen.

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Normalmente el nombre del atributo. |
| **start** |Obligatorio |integer |Índice de la cadena de **source** donde debe empezar la subcadena. El primer carácter de la cadena tendrá el índice de 1, el segundo carácter tendrá el índice de 2, y así sucesivamente. |
| **length** |Obligatorio |integer |Longitud de la subcadena. Si length acaba fuera de la cadena de **source**, la función devolverá una subcadena desde el índice de **start** hasta el final de la cadena de **source**. |

- - -
### <a name="normalizediacritics"></a>NormalizeDiacritics
**Función:**<br> NormalizeDiacritics(source)

**Descripción:**<br> Requiere un argumento de cadena. Devuelve la cadena, pero se reemplazan todos los caracteres diacríticos por sus equivalentes no diacríticos. Normalmente se usa para convertir nombres y apellidos que contienen caracteres diacríticos (acentos) en valores legales que se pueden emplear en diversos identificadores de usuario como, nombres principales de usuario, nombres de cuenta de SAM y direcciones de correo electrónico.

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |string | Normalmente un atributo de nombre o de apellido. |

- - -
### <a name="not"></a>not
**Función:**<br>  Not(source)

**Descripción:**<br> Invierte el valor booleano de **source**. Si el valor de **source** es "*True*", devuelve "*False*". De lo contrario, devuelve "*True*".

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |Cadena booleana |Los valores de **source** esperados son "True" o "False". |

- - -
### <a name="replace"></a>Reemplazar
**Función:**<br> Replace(source, oldValue, regexPattern, regexGroupName, replacementValue, replacementAttributeName, template)

**Descripción:**<br>
Reemplaza valores dentro de una cadena. Funciona de forma diferente dependiendo de los parámetros proporcionados:

* Cuando se proporcionan **oldValue** y **replacementValue**:
  
  * Reemplaza todas las ocurrencias de oldValue en el origen por replacementValue
* Cuando se proporcionan **oldValue** y **template**:
  
  * Reemplaza todas las ocurrencias de **oldValue** en **template** por el valor de **source**
* Cuando se proporcionan **regexPattern**, **regexGroupName** y **replacementValue**:
  
  * Reemplaza todos los valores que coinciden en oldValueRegexPattern en la cadena de origen con replacementValue
* Cuando se proporcionan **regexPattern**, **regexGroupName** y **replacementPropertyName**:
  
  * Si **source** no tiene un valor, se devuelve **source**.
  * Si **source** no tiene un valor, usa **regexPattern** y **regexGroupName** para extraer un valor de sustitución de la propiedad con **replacementPropertyName**. El valor de reemplazo se devuelve como resultado

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |string |Normalmente el nombre del atributo del objeto de origen. |
| **oldValue** |Opcional |string |Valor que se va a reemplazar en **source** o **template**. |
| **regexPattern** |Opcional |String |Patrón Regex del valor que se va a reemplazar en **source**. O bien, cuando se utiliza replacementPropertyName, patrón para extraer el valor de la propiedad de reemplazo. |
| **regexGroupName** |Opcional |string |Nombre del grupo dentro de **regexPattern**. Sólo cuando se utilice replacementPropertyName, extraeremos el valor de este grupo como replacementValue de la propiedad de reemplazo. |
| **replacementValue** |Opcional |String |Nuevo valor para  reemplazar uno anterior. |
| **replacementAttributeName** |Opcional |String |Nombre del atributo que se utilizará para el valor de reemplazo, cuando el origen no tiene ningún valor. |
| **template** |Opcional |String |Cuando se proporcione el valor de **template**, buscaremos **oldValue** dentro de la plantilla y lo reemplazaremos por el valor de origen. |

- - -
### <a name="selectuniquevalue"></a>SelectUniqueValue
**Función:**<br> SelectUniqueValue(uniqueValueRule1, uniqueValueRule2, uniqueValueRule3, …)

**Descripción:**<br> Requiere dos argumentos como mínimo, que son las reglas de generación de valor único definidas con expresiones. La función evalúa cada regla y, a continuación, comprueba la unicidad del valor generado en el directorio o la aplicación de destino. Se devolverá el primer valor único encontrado. Si todos los valores ya existen en el destino, la entrada se depositará y el motivo se anota en los registros de auditoría. No hay ningún límite superior para el número de argumentos que se pueden proporcionar.

> [!NOTE]
>1. Esta es una función de nivel superior, no se puede anidar.
>2. Esta función solo está destinada a usarse para creaciones de entradas. Al usarla con un atributo, establezca la propiedad **Apply Mapping** (Aplicar asignación) en **Solo durante la creación del objeto**.


**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **uniqueValueRule1  … uniqueValueRuleN** |Al menos se requieren dos, sin límite superior |string | Lista de reglas de generación de valor único para realizar la evaluación. |


- - -
### <a name="singleapproleassignment"></a>SingleAppRoleAssignment
**Función:**<br> SingleAppRoleAssignment([appRoleAssignments])

**Descripción:**<br> Devuelve una única función appRoleAssignment de la lista de todas las funciones appRoleAssignments asignadas a un usuario para una aplicación determinada. Esta función es necesaria para convertir el objeto appRoleAssignments en una cadena de nombre de rol único. Tenga en cuenta que el procedimiento recomendado consiste en asegurarse de que solo una función appRoleAssignment esté asignada a un usuario a la vez. Si se asignan varios roles, la cadena del rol devuelta podría no ser predecible. 

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **[appRoleAssignments]** |Obligatorio |string |Objeto **[appRoleAssignments]**. |

- - -
### <a name="split"></a>Dividir
**Función:**<br> Split(source, delimiter)

**Descripción:**<br> divide una cadena en una matriz de varios valores, usando el carácter delimitador especificado.

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |**de origen** que se actualiza. |
| **delimitador** |Obligatorio |String |Especifica el carácter que se usará para dividir la cadena (ejemplo: ","). |

- - -
### <a name="stripspaces"></a>StripSpaces
**Función:**<br>  StripSpaces(source)

**Descripción:**<br>  quita todos los caracteres de espacio (" ") de la cadena de origen.

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |**de origen** que se actualiza. |

- - -
### <a name="switch"></a>Switch
**Función:**<br>  Switch(source, defaultValue, key1, value1, key2, value2, …)

**Descripción:**<br> Cuando el valor de **source** coincide con una **key**, devuelve el **value** de dicha **key**. Si el valor de **source** no coincide con ninguna clave, devuelve **defaultValue**.  Los parámetros **key** y **value** siempre deben estar emparejados. La función espera siempre un número par de parámetros.

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |**Source** que se actualiza. |
| **defaultValue** |Opcional |string |Valor predeterminado que se usará si el origen no coincide con ninguna clave. Puede tratarse de una cadena vacía (""). |
| **key** |Obligatorio |String |**Key** con que se compara el valor de **source**. |
| **value** |Obligatorio |string |Valor de reemplazo para el **source** que coincide con la clave. |

- - -
### <a name="tolower"></a>ToLower
**Función:**<br> ToLower (origen, referencia cultural)

**Descripción:**<br> Toma un valor de cadena de *origen* y se convierte a minúsculas mediante las reglas de referencia cultural que se hayan especificado. Si no hay ninguna información de *referencia cultural* especificada, se usará la referencia cultural invariable.

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Normalmente el nombre del atributo del objeto de origen |
| **referencia cultural** |Opcional |string |El formato para el nombre de la referencia cultural según RFC 4646 es *languagecode2-country/regioncode2*, donde *languagecode2* es el código de idioma de dos letras y *country/regioncode2* es el código de referencia de subcultura de dos letras. Algunos ejemplos son a ja-JP para japonés (Japón) y en-US para inglés (Estados Unidos). En casos donde un código de idioma de dos letras no está disponible, se usa un código de tres letras derivado de ISO 639-2.|

- - -
### <a name="toupper"></a>ToUpper
**Función:**<br> ToUpper (origen, referencia cultural)

**Descripción:**<br> Toma un valor de cadena de *origen* y se convierte a mayúsculas mediante las reglas de referencia cultural que se hayan especificado. Si no hay ninguna información de *referencia cultural* especificada, se usará la referencia cultural invariable.

**Parámetros:**<br> 

| NOMBRE | Obligatorio/Repetición | Type | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Normalmente el nombre del atributo del objeto de origen. |
| **referencia cultural** |Opcional |String |El formato para el nombre de la referencia cultural según RFC 4646 es *languagecode2-country/regioncode2*, donde *languagecode2* es el código de idioma de dos letras y *country/regioncode2* es el código de referencia de subcultura de dos letras. Algunos ejemplos son a ja-JP para japonés (Japón) y en-US para inglés (Estados Unidos). En casos donde un código de idioma de dos letras no está disponible, se usa un código de tres letras derivado de ISO 639-2.|

## <a name="examples"></a>Ejemplos
### <a name="strip-known-domain-name"></a>Seccionar un nombre de dominio conocido
Debe seccionar un nombre de dominio conocido de correo electrónico de un usuario para obtener un nombre de usuario. <br>
 Por ejemplo, si el dominio es "contoso.com", puede usar la expresión siguiente:

**Expresión:** <br>
`Replace([mail], "@contoso.com", , ,"", ,)`

**Ejemplo de entrada y salida:** <br>

* **ENTRADA** (mail): "john.doe@contoso.com"
* **SALIDA**: "john.doe"

### <a name="append-constant-suffix-to-user-name"></a>Anexar sufijos constantes a nombres de usuario
Si está utilizando un espacio aislado de Salesforce, deberá anexar un sufijo adicional a los nombres de usuario antes de sincronizarlas.

**Expresión:** <br>
`Append([userPrincipalName], ".test")`

**Entrada/salida de ejemplo:** <br>

* **ENTRADA**: (userPrincipalName): "John.Doe@contoso.com"
* **ENTRADA**:  "John.Doe@contoso.com.test"

### <a name="generate-user-alias-by-concatenating-parts-of-first-and-last-name"></a>Generar el alias de usuario concatenando partes de nombre y apellidos
Debe generar un alias de usuario con las tres primeras letras del nombre del usuario y las cinco primeras letras del apellido del usuario.

**Expresión:** <br>
`Append(Mid([givenName], 1, 3), Mid([surname], 1, 5))`

**Entrada/salida de ejemplo:** <br>

* **INPUT** (givenName): "John"
* **INPUT** (surname): "Doe"
* **OUTPUT**:  "JohnDoe"

### <a name="remove-diacritics-from-a-string"></a>Quitar los signos diacríticos de una cadena
Necesita reemplazar caracteres que contienen acentos por otros equivalentes que no los contienen.

**Expresión:** <br>
NormalizeDiacritics([givenName])

**Entrada/salida de ejemplo:** <br>

* **INPUT** (givenName): "Zoë"
* **OUTPUT**:  "Zoe"

### <a name="split-a-string-into-a-multi-valued-array"></a>Dividir una cadena en una matriz con varios valores
Debe tomar una lista de cadenas delimitada con comas y dividir esas cadenas en una matriz que se pueda conectar a un atributo de varios valores, como el atributo PermissionSets de Salesforce. En este ejemplo, una lista de conjuntos de permisos se ha rellenado en extensionAttribute5 en Azure AD.

**Expresión:** <br>
Split([extensionAttribute5], ",")

**Entrada/salida de ejemplo:** <br>

* **INPUT** (extensionAttribute5): "PermissionSetOne, PermisionSetTwo"
* **OUTPUT**:  ["PermissionSetOne", "PermissionSetTwo"]

### <a name="output-date-as-a-string-in-a-certain-format"></a>Fecha de resultado como una cadena en un formato determinado
Desea enviar las fechas a una aplicación SaaS con un formato determinado. <br>
 Por ejemplo, desea dar formato a las fechas de ServiceNow.

**Expresión:** <br>

`FormatDateTime([extensionAttribute1], "yyyyMMddHHmmss.fZ", "yyyy-MM-dd")`

**Entrada/salida de ejemplo:**

* **INPUT** (extensionAttribute1): "20150123105347.1Z"
* **OUTPUT**:  "2015-01-23"

### <a name="replace-a-value-based-on-predefined-set-of-options"></a>Reemplazar un valor basado en un conjunto predefinido de opciones

Debe definir la zona horaria del usuario según el código de estado almacenado en Azure AD. <br>
 Si el código de estado no coincide con ninguna de las opciones predefinidas, use el valor predeterminado de "Australia/Sídney".

**Expresión:** <br>
`Switch([state], "Australia/Sydney", "NSW", "Australia/Sydney","QLD", "Australia/Brisbane", "SA", "Australia/Adelaide")`

**Entrada/salida de ejemplo:**

* **INPUT** (state): "QLD"
* **OUTPUT**: "Australia/Brisbane"

### <a name="replace-characters-using-a-regular-expression"></a>Reemplazar los caracteres con una expresión regular
Debe buscar los caracteres que coincidan con un valor de expresión regular y quitarlos.

**Expresión:** <br>

Replace([mailNickname], , "[a-zA-Z_]*", , "", , )

**Entrada/salida de ejemplo:**

* **INPUT** (mailNickname: "john_doe72"
* **OUTPUT**: "72"

### <a name="convert-generated-userprincipalname-upn-value-to-lower-case"></a>Conversión del valor generado de userPrincipalName (UPN) a minúsculas
En el ejemplo siguiente, el valor de UPN se genera mediante la concatenación de los campos de origen PreferredFirstName y PreferredLastName, y la función ToLower opera en la cadena generada para convertir todos los caracteres a minúsculas. 

`ToLower(Join("@", NormalizeDiacritics(StripSpaces(Join(".",  [PreferredFirstName], [PreferredLastName]))), "contoso.com"))`

**Entrada/salida de ejemplo:**

* **INPUT** (PreferredFirstName): "John"
* **INPUT** (PreferredLastName): "Smith"
* **OUTPUT**: "john.smith@contoso.com"

### <a name="generate-unique-value-for-userprincipalname-upn-attribute"></a>Generación de un valor único para el atributo userPrincipalName (UPN)
Según el nombre del usuario, el segundo nombre y el apellido, deberá generar un valor para el atributo UPN y comprobar su unicidad en el directorio de AD de destino antes de asignar el valor al atributo UPN.

**Expresión:** <br>

    SelectUniqueValue( 
        Join("@", NormalizeDiacritics(StripSpaces(Join(".",  [PreferredFirstName], [PreferredLastName]))), "contoso.com"), 
        Join("@", NormalizeDiacritics(StripSpaces(Join(".",  Mid([PreferredFirstName], 1, 1), [PreferredLastName]))), "contoso.com")
        Join("@", NormalizeDiacritics(StripSpaces(Join(".",  Mid([PreferredFirstName], 1, 2), [PreferredLastName]))), "contoso.com")
    )

**Entrada/salida de ejemplo:**

* **INPUT** (PreferredFirstName): "John"
* **INPUT** (PreferredLastName): "Smith"
* **OUTPUT**: "John.Smith@contoso.com" si el valor de UPN John.Smith@contoso.com no existe aún en el directorio
* **OUTPUT**: "J.Smith@contoso.com" si el valor de UPN John.Smith@contoso.com ya existe en el directorio
* **OUTPUT**: "Jo.Smith@contoso.com" si los dos valores de UPN anteriores ya existen en el directorio

## <a name="related-articles"></a>Artículos relacionados
* [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](user-provisioning.md)
* [Personalización de asignaciones de atributos para el aprovisionamiento de usuarios](customize-application-attributes.md)
* [Filtros de ámbito para el aprovisionamiento de usuario](define-conditional-rules-for-provisioning-user-accounts.md)
* [Uso de SCIM para habilitar el aprovisionamiento automático de usuarios y grupos de Azure Active Directory a aplicaciones](use-scim-to-provision-users-and-groups.md)
* [Notificaciones de aprovisionamiento de cuentas](user-provisioning.md)
* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS](../saas-apps/tutorial-list.md)
