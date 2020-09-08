---
title: 'Sincronización de Azure AD Connect: Referencia de funciones | Microsoft Docs'
description: Referencia de expresiones declarativas de aprovisionamiento en la sincronización de Azure AD Connect
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 4f525ca0-be0e-4a2e-8da1-09b6b567ed5f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 07/12/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6d8ec8eec28d66cf93608393ddca45f78460d831
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279778"
---
# <a name="azure-ad-connect-sync-functions-reference"></a>Sincronización de Azure AD Connect: Referencia de funciones
En Azure AD Connect, las funciones se usan para manipular un valor de atributo durante la sincronización.  
La sintaxis de las funciones se expresa con el siguiente formato:  
`<output type> FunctionName(<input type> <position name>, ..)`

Si una función está sobrecargada y acepta varias sintaxis, se enumeran todas las sintaxis válidas.  
Las funciones están fuertemente tipadas y comprueban que el tipo pasado coincide con el tipo documentado.  
Si el tipo no coincide, se produce un error.

Los tipos se expresan con la siguiente sintaxis:

* **bin** : binario
* **bool** : booleano
* **dt** : fecha y hora UTC
* **enum** : enumeración de constantes conocidas
* **exp** : expresión, que se espera que evalúe en un booleano
* **mvbin** : binario de varios valores
* **mvstr** : cadena multivalor
* **mvref** : referencia multivalor
* **num** : numérico
* **ref** : referencia
* **str** : cadena
* **var** : una variante de (casi) cualquier otro tipo
* **void** : no devuelve un valor

Las funciones con los tipos **mvbin**, **mvstr** y **mvref** solo pueden funcionar en atributos con varios valores. Las funciones con **bin**, **str** y **ref** funcionan en atributos con un solo valor y con varios valores.

## <a name="functions-reference"></a>Referencia de funciones

* **Certificate**
  * [CertExtensionOids](#certextensionoids)
  * [CertFormat](#certformat)
  * [CertFriendlyName](#certfriendlyname)
  * [CertHashString](#certhashstring)
  * [CertIssuer](#certissuer)
  * [CertIssuerDN](#certissuerdn)
  * [CertIssuerOid](#certissueroid)
  * [CertKeyAlgorithm](#certkeyalgorithm)
  * [CertKeyAlgorithmParams](#certkeyalgorithmparams)
  * [CertNameInfo](#certnameinfo)
  * [CertNotAfter](#certnotafter)
  * [CertNotBefore](#certnotbefore)
  * [CertPublicKeyOid](#certpublickeyoid)
  * [CertPublicKeyParametersOid](#certpublickeyparametersoid)
  * [CertSerialNumber](#certserialnumber)
  * [CertSignatureAlgorithmOid](#certsignaturealgorithmoid)
  * [CertSubject](#certsubject)
  * [CertSubjectNameDN](#certsubjectnamedn)
  * [CertSubjectNameOid](#certsubjectnameoid)
  * [CertThumbprint](#certthumbprint)
  * [CertVersion](#certversion)
  * [IsCert](#iscert)
* **Conversión**
  * [CBool](#cbool)
  * [CDate](#cdate)
  * [CGuid](#cguid)
  * [ConvertFromBase64](#convertfrombase64)
  * [ConvertToBase64](#converttobase64)
  * [ConvertFromUTF8Hex](#convertfromutf8hex)
  * [ConvertToUTF8Hex](#converttoutf8hex)
  * [CNum](#cnum)
  * [CRef](#cref)
  * [CStr](#cstr)
  * [StringFromGuid](#stringfromguid)
  * [StringFromSid](#stringfromsid)
* **Fecha y hora**
  * [DateAdd](#dateadd)
  * [DateFromNum](#datefromnum)
  * [FormatDateTime](#formatdatetime)
  * [Now](#now)
  * [NumFromDate](#numfromdate)
* **Directorio**
  * [DNComponent](#dncomponent)
  * [DNComponentRev](#dncomponentrev)
  * [EscapeDNComponent](#escapedncomponent)
* **Evaluación**
  * [IsBitSet](#isbitset)
  * [IsDate](#isdate)
  * [IsEmpty](#isempty)
  * [IsGuid](#isguid)
  * [IsNull](#isnull)
  * [IsNullOrEmpty](#isnullorempty)
  * [IsNumeric](#isnumeric)
  * [IsPresent](#ispresent)
  * [IsString](#isstring)
* **Matemáticas**
  * [BitAnd](#bitand)
  * [BitOr](#bitor)
  * [RandomNum](#randomnum)
* **Multivalor**
  * [Contains](#contains)
  * [Recuento](#count)
  * [Elemento](#item)
  * [ItemOrNull](#itemornull)
  * [Join](#join)
  * [RemoveDuplicates](#removeduplicates)
  * [Dividir](#split)
* **Flujo de programa**
  * [Error](#error)
  * [IIF](#iif)
  * [Select](#select)
  * [Switch](#switch)
  * [Where](#where)
  * [With](#with)
* **Texto**
  * [GUID](#guid)
  * [InStr](#instr)
  * [InStrRev](#instrrev)
  * [LCase](#lcase)
  * [Left](#left)
  * [Len](#len)
  * [LTrim](#ltrim)
  * [Mid](#mid)
  * [PadLeft](#padleft)
  * [PadRight](#padright)
  * [PCase](#pcase)
  * [Sustituya](#replace)
  * [ReplaceChars](#replacechars)
  * [Right](#right)
  * [RTrim](#rtrim)
  * [Trim](#trim)
  * [UCase](#ucase)
  * [Word](#word)

---
### <a name="bitand"></a>BitAnd
**Descripción:**  
: la función BitAnd establece los bits especificados en un valor.

**Sintaxis:**  
`num BitAnd(num value1, num value2)`

* value1, value2: valores numéricos que deberían estar unidos por el operador AND

**Observaciones**:  
: esta función convierte ambos parámetros en la representación binaria y establece un bit en los siguientes valores:

* 0: si uno o ambos de los bits correspondientes en *value1* y *value2* son 0.
* 1: si ambos de los bits correspondientes son 1.

En otras palabras, devuelve 0 en todos los casos excepto cuando los bits correspondientes de los dos parámetros son 1.

**Ejemplo**:  
`BitAnd(&HF, &HF7)`  
devuelve 7 porque los valores hexadecimales "F" y "F7" se evalúan en este valor.

---
### <a name="bitor"></a>BitOr
**Descripción:**  
: la función BitOr establece los bits especificados en un valor.

**Sintaxis:**  
`num BitOr(num value1, num value2)`

* value1, value2: valores numéricos que deberían estar unidos por el operador OR

**Observaciones**:  
: esta función convierte ambos parámetros en la representación binaria y establece un bit en 1 si uno o ambos de los bits correspondientes de la máscara y marca son 1, y en 0 si son 0. En otras palabras, devuelve 1 en todos los casos excepto cuando los bits correspondientes de ambos parámetros son 0.

---
### <a name="cbool"></a>CBool
**Descripción:**  
: la función CBool devuelve un valor booleano basado en la expresión evaluada.

**Sintaxis:**  
`bool CBool(exp Expression)`

**Observaciones**:  
Si la expresión se evalúa en un valor distinto de cero, CBool devuelve True. En caso contrario, devuelve False.

**Ejemplo**:  
`CBool([attrib1] = [attrib2])`  

Devuelve True si ambos atributos tienen el mismo valor.

---
### <a name="cdate"></a>CDate
**Descripción:**  
: la función CDate devuelve un valor DateTime UTC a partir de una cadena. DateTime no es un tipo de atributo nativo en Sincronización pero lo usan algunas funciones.

**Sintaxis:**  
`dt CDate(str value)`

* Valor: una cadena con una fecha, hora y opcionalmente, una zona horaria

**Observaciones**:  
: la cadena devuelta siempre es UTC.

**Ejemplo**:  
`CDate([employeeStartTime])`  
devuelve un valor DateTime basado en la hora de inicio del empleado.

`CDate("2013-01-10 4:00 PM -8")`  
Devuelve un valor DateTime que representa "2013-01-11 12:00 AM".


---
### <a name="certextensionoids"></a>CertExtensionOids
**Descripción:**  
Devuelve los valores de Oid de todas las extensiones esenciales de un objeto de certificado.

**Sintaxis:**  
`mvstr CertExtensionOids(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certformat"></a>CertFormat
**Descripción:**  
Devuelve el nombre del formato de este certificado X.509v3.

**Sintaxis:**  
`str CertFormat(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certfriendlyname"></a>CertFriendlyName
**Descripción:**  
Devuelve el alias asociado de un certificado.

**Sintaxis:**  
`str CertFriendlyName(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certhashstring"></a>CertHashString
**Descripción:**  
Devuelve el valor de hash SHA1 del certificado X.509v3 en forma de una cadena hexadecimal.

**Sintaxis:**  
`str CertHashString(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certissuer"></a>CertIssuer
**Descripción:**  
Devuelve el nombre de la entidad de certificación que emitió el certificado X.509v3.

**Sintaxis:**  
`str CertIssuer(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certissuerdn"></a>CertIssuerDN
**Descripción:**  
Devuelve el nombre distintivo del emisor del certificado.

**Sintaxis:**  
`str CertIssuerDN(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certissueroid"></a>CertIssuerOid
**Descripción:**  
Devuelve el Oid del emisor del certificado.

**Sintaxis:**  
`str CertIssuerOid(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certkeyalgorithm"></a>CertKeyAlgorithm
**Descripción:**  
Devuelve la información del algoritmo de clave de este certificado X.509v3 como una cadena.

**Sintaxis:**  
`str CertKeyAlgorithm(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certkeyalgorithmparams"></a>CertKeyAlgorithmParams
**Descripción:**  
Devuelve los parámetros del algoritmo de clave del certificado X.509v3 como una cadena hexadecimal.

**Sintaxis:**  
`str CertKeyAlgorithm(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certnameinfo"></a>CertNameInfo
**Descripción:**  
Devuelve el asunto y el emisor de nombres de un certificado.

**Sintaxis:**  
`str CertNameInfo(binary certificateRawData, str x509NameType, bool includesIssuerName)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).
*   X509NameType: el valor X509NameType para el sujeto.
*   includesIssuerName: true para incluir el nombre del emisor; en caso contrario, false.

---
### <a name="certnotafter"></a>CertNotAfter
**Descripción:**  
Devuelve la fecha en hora local después de la cual un certificado ya no es válido.

**Sintaxis:**  
`dt CertNotAfter(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certnotbefore"></a>CertNotBefore
**Descripción:**  
Devuelve la fecha en hora local después de la cual un certificado se convierte en válido.

**Sintaxis:**  
`dt CertNotBefore(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certpublickeyoid"></a>CertPublicKeyOid
**Descripción:**  
Devuelve el Oid de la clave pública del certificado X.509v3.

**Sintaxis:**  
`str CertKeyAlgorithm(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certpublickeyparametersoid"></a>CertPublicKeyParametersOid
**Descripción:**  
Devuelve el Oid de los parámetros de clave pública del certificado X.509v3.

**Sintaxis:**  
`str CertPublicKeyParametersOid(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certserialnumber"></a>CertSerialNumber
**Descripción:**  
Devuelve el número de serie del certificado X.509v3.

**Sintaxis:**  
`str CertSerialNumber(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certsignaturealgorithmoid"></a>CertSignatureAlgorithmOid
**Descripción:**  
Devuelve el Oid del algoritmo utilizado para crear la firma de un certificado.

**Sintaxis:**  
`str CertSignatureAlgorithmOid(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certsubject"></a>CertSubject
**Descripción:**  
Obtiene el nombre distintivo del sujeto de un certificado.

**Sintaxis:**  
`str CertSubject(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certsubjectnamedn"></a>CertSubjectNameDN
**Descripción:**  
Devuelve el nombre distintivo del sujeto de un certificado.

**Sintaxis:**  
`str CertSubjectNameDN(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certsubjectnameoid"></a>CertSubjectNameOid
**Descripción:**  
Devuelve el Oid del nombre del sujeto de un certificado.

**Sintaxis:**  
`str CertSubjectNameOid(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certthumbprint"></a>CertThumbprint
**Descripción:**  
Devuelve la huella digital de un certificado.

**Sintaxis:**  
`str CertThumbprint(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="certversion"></a>CertVersion
**Descripción:**  
Devuelve la versión de formato X.509 de un certificado.

**Sintaxis:**  
`str CertThumbprint(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).

---
### <a name="cguid"></a>CGuid
**Descripción:**  
: la función CGuid convierte la representación de cadena de un GUID en su representación binaria.

**Sintaxis:**  
`bin CGuid(str GUID)`

* Una cadena con un formato que sigue este patrón: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx o {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

---
### <a name="contains"></a>Contains
**Descripción:**  
: la función Contains busca una cadena dentro de un atributo multivalor.

**Sintaxis:**  
`num Contains (mvstring attribute, str search)` - distingue mayúsculas y minúsculas  
`num Contains (mvstring attribute, str search, enum Casetype)`  
`num Contains (mvref attribute, str search)` - distingue mayúsculas y minúsculas

* attribute: el atributo con varios valores de búsqueda.
* search: cadena para buscar en el atributo.
* Casetype: CaseInsensitive o CaseSensitive.

Devuelve el índice en el atributo de varios valores donde se ha encontrado la cadena. Si no se encuentra la cadena, se devuelve 0.

**Observaciones**:  
: para los atributos de cadena multivalor, la búsqueda encuentra subcadenas en los valores.  
Para los atributos de referencia, la cadena de búsqueda debe coincidir exactamente con el valor para que se considere una coincidencia.

**Ejemplo**:  
`IIF(Contains([proxyAddresses],"SMTP:")>0,[proxyAddresses],Error("No primary SMTP address found."))`  
Si el atributo proxyAddresses tiene una dirección de correo electrónico principal (indicada mediante las mayúsculas "SMTP:"), devuelve el atributo proxyAddress. De lo contrario, devuelve un error.

---
### <a name="convertfrombase64"></a>ConvertFromBase64
**Descripción:**  
: la función ConvertFromBase64 convierte el valor codificado en base64 especificado en una cadena normal.

**Sintaxis:**  
`str ConvertFromBase64(str source)` da por hecho que se usará Unicode para la codificación.  
`str ConvertFromBase64(str source, enum Encoding)`

* source: cadena codificada en Base64  
* Encoding: Unicode, ASCII, UTF8

**Ejemplo**  
`ConvertFromBase64("SABlAGwAbABvACAAdwBvAHIAbABkACEA")`  
`ConvertFromBase64("SGVsbG8gd29ybGQh", UTF8)`

Ambos ejemplos devuelven "*Hola a todos*"

---
### <a name="convertfromutf8hex"></a>ConvertFromUTF8Hex
**Descripción:**  
: la función ConvertFromUTF8Hex convierte el valor codificado hexadecimal UTF8 especificado en una cadena.

**Sintaxis:**  
`str ConvertFromUTF8Hex(str source)`

* source: cadena codificada de 2 bytes UTF8

**Observaciones**:  
La diferencia entre esta función y ConvertFromBase64([],UTF8) es que el resultado es descriptivo para el atributo DN.  
Azure Active Directory utiliza este formato como DN.

**Ejemplo**:  
`ConvertFromUTF8Hex("48656C6C6F20776F726C6421")`  
devuelve "*Hello world!* ".

---
### <a name="converttobase64"></a>ConvertToBase64
**Descripción:**  
La función ConvertToBase64 convierte una cadena en una en base64 de Unicode.  
Convierte el valor de una matriz de enteros en su representación de cadena equivalente que se codifica con dígitos de base 64.

**Sintaxis:**  
`str ConvertToBase64(str source)`

**Ejemplo**:  
`ConvertToBase64("Hello world!")`  
devuelve "SABlAGwAbABvACAAdwBvAHIAbABkACEA".

---
### <a name="converttoutf8hex"></a>ConvertToUTF8Hex
**Descripción:**  
: la función ConvertToUTF8Hex convierte una cadena en un valor codificado hexadecimal UTF8.

**Sintaxis:**  
`str ConvertToUTF8Hex(str source)`

**Observaciones**:  
: Azure Active Directory usa el formato de salida de esta función como formato de atributo DN.

**Ejemplo**:  
`ConvertToUTF8Hex("Hello world!")`  
devuelve 48656C6C6F20776F726C6421.

---
### <a name="count"></a>Count
**Descripción:**  
: la función Count devuelve el número de elementos en un atributo multivalor.

**Sintaxis:**  
`num Count(mvstr attribute)`

---
### <a name="cnum"></a>CNum
**Descripción:**  
: la función CNum toma una cadena y devuelve un tipo de datos numérico.

**Sintaxis:**  
`num CNum(str value)`

---
### <a name="cref"></a>CRef
**Descripción:**  
: convierte una cadena en un atributo de referencia.

**Sintaxis:**  
`ref CRef(str value)`

**Ejemplo**:  
`CRef("CN=LC Services,CN=Microsoft,CN=lcspool01,CN=Pools,CN=RTC Service," & %Forest.LDAP%)`

---
### <a name="cstr"></a>CStr
**Descripción:**  
: la función CStr se convierte en un tipo de datos de cadena.

**Sintaxis:**  
`str CStr(num value)`  
`str CStr(ref value)`  
`str CStr(bool value)`  

* value: puede ser un valor numérico, un atributo de referencia o un valor booleano.

**Ejemplo**:  
`CStr([dn])`  
podría devolver "cn=Joe,dc=contoso,dc=com"

---
### <a name="dateadd"></a>DateAdd
**Descripción:**  
: devuelve un valor Date que contiene una fecha a la que se ha agregado un intervalo de tiempo especificado.

**Sintaxis:**  
`dt DateAdd(str interval, num value, dt date)`

* interval: expresión de cadena que es el intervalo de tiempo que desea agregar. La cadena debe tener uno de los siguientes valores:
  * yyyy Año
  * q Trimestre
  * m Mes
  * y Día del año
  * d Día
  * w Día de la semana
  * ww Semana
  * h Hora
  * n Minuto
  * s Segundo
* value: el número de unidades que desea agregar. Puede ser positivo (para obtener fechas futuras) o negativo (para obtener fechas del pasado).
* date: DateTime que representa la fecha a la que se agrega el intervalo.

**Ejemplo**:  
`DateAdd("m", 3, CDate("2001-01-01"))`  
agrega 3 meses y devuelve un valor DateTime que representa "2001-04-01".

---
### <a name="datefromnum"></a>DateFromNum
**Descripción:**  
: la función DateFromNum convierte un valor con formato de fecha de AD en un tipo DateTime.

**Sintaxis:**  
`dt DateFromNum(num value)`

**Ejemplo**:  
`DateFromNum([lastLogonTimestamp])`  
`DateFromNum(129699324000000000)`  
devuelve un valor DateTime que representa 2012-01-01 23:00:00.

---
### <a name="dncomponent"></a>DNComponent
**Descripción:**  
: la función DNComponent devuelve el valor de un componente DN especificado empezando por la izquierda.

**Sintaxis:**  
`str DNComponent(ref dn, num ComponentNumber)`

* dn: el atributo de referencia que interpretar
* ComponentNumber: el componente en DN para devolver

**Ejemplo**:  
`DNComponent(CRef([dn]),1)`  
si dn es "cn=Joe,ou=…", devuelve Joe.

---
### <a name="dncomponentrev"></a>DNComponentRev
**Descripción:**  
: la función DNComponentRev devuelve el valor de un componente DN especificado empezando por la derecha (final).

**Sintaxis:**  
`str DNComponentRev(ref dn, num ComponentNumber)`  
`str DNComponentRev(ref dn, num ComponentNumber, enum Options)`

* dn: el atributo de referencia que interpretar
* ComponentNumber: el componente en DN que devolver
* Opciones: DC, ignora todos los componentes con "dc="

**Ejemplo**:  
Si dn es "cn=Joe,ou=Atlanta,ou=GA,ou=US, dc=contoso,dc=com"  
`DNComponentRev(CRef([dn]),3)`  
`DNComponentRev(CRef([dn]),1,"DC")`  
Ambos devuelven US.

---
### <a name="error"></a>Error
**Descripción:**  
: la función Error se usa para devolver un error personalizado.

**Sintaxis:**  
`void Error(str ErrorMessage)`

**Ejemplo**:  
`IIF(IsPresent([accountName]),[accountName],Error("AccountName is required"))`  
si el atributo accountName no está presente, se produce un error en el objeto.

---
### <a name="escapedncomponent"></a>EscapeDNComponent
**Descripción:**  
: la función EscapeDNComponent toma un componente de DN y genera secuencias de escape para poder representarlo en LDAP.

**Sintaxis:**  
`str EscapeDNComponent(str value)`

**Ejemplo**:  
`EscapeDNComponent("cn=" & [displayName]) & "," & %ForestLDAP%)`  
se asegura de que el objeto se puede crear en un directorio LDAP incluso si el atributo displayName tiene caracteres para los que deben generarse secuencias de escape en LDAP.

---
### <a name="formatdatetime"></a>FormatDateTime
**Descripción:**  
: la función FormatDateTime se usa para dar formato a un valor DateTime en una cadena con un formato especificado.

**Sintaxis:**  
`str FormatDateTime(dt value, str format)`

* value: un valor con el formato DateTime
* format: una cadena que representa el formato de conversión.

**Observaciones**:  
Los valores posibles para el formato pueden encontrarse aquí: [Formatos de fecha y hora personalizados para la función FORMAT](/dax/custom-date-and-time-formats-for-the-format-function).

**Ejemplo**:  

`FormatDateTime(CDate("12/25/2007"),"yyyy-mm-dd")`  
Genera "2007-12-25".

`FormatDateTime(DateFromNum([pwdLastSet]),"yyyyMMddHHmmss.0Z")`  
Puede generar "20140905081453.0Z".

---
### <a name="guid"></a>Guid
**Descripción:**  
La función GUID genera un nuevo GUID aleatorio.

**Sintaxis:**  
`str Guid()`

---
### <a name="iif"></a>IIF
**Descripción:**  
: la función IIF devuelve uno de un conjunto de valores posibles según una condición especificada.

**Sintaxis:**  
`var IIF(exp condition, var valueIfTrue, var valueIfFalse)`

* condition: cualquier valor o expresión que pueda evaluarse en True o False.
* valueIfTrue: si la condición se evalúa como true, el valor devuelto.
* valueIfFalse: si la condición se evalúa como false, el valor devuelto.

**Ejemplo**:  
`IIF([employeeType]="Intern","t-" & [alias],[alias])`  
 Si el usuario está en prácticas, devuelve el alias de un usuario al que agrega "t-" al principio. De lo contrario, el alias del usuario se queda como está.

---
### <a name="instr"></a>InStr
**Descripción:**  
: la función InStr busca la primera repetición de una subcadena en una cadena.

**Sintaxis:**  

`num InStr(str stringcheck, str stringmatch)`  
`num InStr(str stringcheck, str stringmatch, num start)`  
`num InStr(str stringcheck, str stringmatch, num start , enum compare)`

* stringcheck: cadena que se va a buscar
* stringmatch: cadena que se tiene que encontrar
* start: posición de inicio para encontrar la subcadena
* compare: vbTextCompare o vbBinaryCompare

**Observaciones**:  
: devuelve la posición en la que se encontró la subcadena o 0 si no se encuentra.

**Ejemplo**:  
`InStr("The quick brown fox","quick")`  
se evalúa en 5.

`InStr("repEated","e",3,vbBinaryCompare)`  
Se evalúa en 7.

---
### <a name="instrrev"></a>InStrRev
**Descripción:**  
: la función InStrRev busca la última repetición de una subcadena en una cadena.

**Sintaxis:**  
`num InstrRev(str stringcheck, str stringmatch)`  
`num InstrRev(str stringcheck, str stringmatch, num start)`  
`num InstrRev(str stringcheck, str stringmatch, num start, enum compare)`

* stringcheck: cadena que se va a buscar
* stringmatch: cadena que se tiene que encontrar
* start: posición de inicio para encontrar la subcadena
* compare: vbTextCompare o vbBinaryCompare

**Observaciones**:  
: devuelve la posición en la que se encontró la subcadena o 0 si no se encuentra.

**Ejemplo**:  
`InStrRev("abbcdbbbef","bb")`  
devuelve 7.

---
### <a name="isbitset"></a>IsBitSet
**Descripción:**  
la función IsBitSet prueba si un bit se establece o no.

**Sintaxis:**  
`bool IsBitSet(num value, num flag)`

* value: un valor numérico que es evaluated.flag: un valor numérico que tiene el bit que se va a evaluar

**Ejemplo**:  
`IsBitSet(&HF,4)`  
devuelve True porque el bit "4" se establece en el valor hexadecimal "F".

---
### <a name="isdate"></a>IsDate
**Descripción:**  
si la expresión se puede evaluar como un tipo DateTime, la función IsDate se evalúa en True.

**Sintaxis:**  
`bool IsDate(var Expression)`

**Observaciones**:  
se usa para determinar si CDate() será correcto.

---
### <a name="iscert"></a>IsCert
**Descripción:**  
Devuelve true si los datos sin procesar se pueden serializar en el objeto de certificado X509Certificate2. NET.

**Sintaxis:**  
`bool CertThumbprint(binary certificateRawData)`  
*   certificateRawData: representación de matriz de bytes de un certificado X.509. La matriz de bytes puede ser de datos X.509 codificados en Base64 o codificados en binario (DER).
---
### <a name="isempty"></a>IsEmpty
**Descripción:**  
si el atributo está presente en CS o MV, pero se evalúa en una cadena vacía, la función IsEmpty se evalúa en True.

**Sintaxis:**  
`bool IsEmpty(var Expression)`

---
### <a name="isguid"></a>IsGuid
**Descripción:**  
si la cadena se pudo convertir en un GUID, la función IsGuid evaluada en True.

**Sintaxis:**  
`bool IsGuid(str GUID)`

**Observaciones**:  
un GUID se define como una cadena que sigue uno de estos patrones: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx or {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}.

Se usa para determinar si CGuid() es correcto.

**Ejemplo**:  
`IIF(IsGuid([strAttribute]),CGuid([strAttribute]),NULL)`  
Si StrAttribute tiene un formato GUID, devuelve una representación binaria. De lo contrario, devuelve un valor Null.

---
### <a name="isnull"></a>IsNull
**Descripción:**  
si la expresión se evalúa como Null, la función IsNull devuelve True.

**Sintaxis:**  
`bool IsNull(var Expression)`

**Observaciones**:  
para un atributo, un valor Null se expresa mediante la ausencia del atributo.

**Ejemplo**:  
`IsNull([displayName])`  
devuelve True si el atributo no está presente en CS o MV.

---
### <a name="isnullorempty"></a>IsNullOrEmpty
**Descripción:**  
si la expresión es Null o una cadena vacía, la función IsNullOrEmpty devuelve True.

**Sintaxis:**  
`bool IsNullOrEmpty(var Expression)`

**Observaciones**:  
para un atributo, esto se evaluaría como True si el atributo no está presente o está presente, pero es una cadena vacía.  
La función contraria a esta es IsPresent.

**Ejemplo**:  
`IsNullOrEmpty([displayName])`  
devuelve True si el atributo no está presente o si es una cadena vacía en CS o MV.

---
### <a name="isnumeric"></a>IsNumeric
**Descripción:**  
la función IsNumeric devuelve un valor booleano que indica si una expresión se puede evaluar como un tipo de número.

**Sintaxis:**  
`bool IsNumeric(var Expression)`

**Observaciones**:  
se usa para determinar si CNum() analizará correctamente la expresión.

---
### <a name="isstring"></a>IsString
**Descripción:**  
si la expresión se puede evaluar en un tipo de cadena, la función IsString se evalúa en True.

**Sintaxis:**  
`bool IsString(var expression)`

**Observaciones**:  
se usa para determinar si CStr() puede analizar correctamente la expresión.

---
### <a name="ispresent"></a>IsPresent
**Descripción:**  
si la expresión se evalúa como una cadena que no es Null y no está vacía, la función IsPresent devuelve True.

**Sintaxis:**  
`bool IsPresent(var expression)`

**Observaciones**:  
la función contraria a esta es IsNullOrEmpty.

**Ejemplo**:  
`Switch(IsPresent([directManager]),[directManager], IsPresent([skiplevelManager]),[skiplevelManager], IsPresent([director]),[director])`

---
### <a name="item"></a>Elemento
**Descripción:**  
la función Item devuelve un elemento de un atributo o una cadena de varios valores.

**Sintaxis:**  
`var Item(mvstr attribute, num index)`

* attribute: atributo de varios valores
* index: índice para un elemento en la cadena de varios valores.

**Observaciones**:  
la función Item es útil con la función Contains puesto que esta última función devuelve el índice a un elemento en el atributo de varios valores.

Se produce un error si el índice está fuera de los límites.

**Ejemplo**:  
`Mid(Item([proxyAddresses],Contains([proxyAddresses], "SMTP:")),6)`  
devuelve la dirección de correo electrónico principal.

---
### <a name="itemornull"></a>ItemOrNull
**Descripción:**  
la función ItemOrNull devuelve un elemento de un atributo o una cadena de varios valores.

**Sintaxis:**  
`var ItemOrNull(mvstr attribute, num index)`

* attribute: atributo de varios valores
* index: índice para un elemento en la cadena de varios valores.

**Observaciones**:  
la función ItemOrNull es útil con la función Contains puesto que esta última función devuelve el índice a un elemento en el atributo de varios valores.

Si el índice está fuera de los límites, devuelve un valor Null.

---
### <a name="join"></a>Join
**Descripción:**  
la función Join toma una cadena de varios valores y devuelve una cadena de valor único con un separador especificado insertado entre cada elemento.

**Sintaxis:**  
`str Join(mvstr attribute)`  
`str Join(mvstr attribute, str Delimiter)`

* : atributo de varios valores que contiene cadenas para combinar.
* delimiter: cualquier cadena utilizada para separar las subcadenas en la cadena devuelta. Si se omite, se usa el carácter de espacio (""). Si delimitador es una cadena de longitud cero ("") o nada, todos los elementos de la lista se concatenan sin delimitadores.

**Comentarios:**  
hay paridad entre las funciones Join y Split. La función Join toma una matriz de cadenas y las combina con una cadena de delimitación para devolver una sola cadena. La función Split toma una cadena y la separa en el delimitador para devolver una matriz de cadenas. Sin embargo, una diferencia clave es que Join puede concatenar cadenas con cualquier cadena de delimitación, mientras que attribute puede separar solo cadenas mediante un delimitador de carácter único.

**Ejemplo**:  
`Join([proxyAddresses],",")`  
Podría devolver: "SMTP:john.doe@contoso.com,smtp:jd@contoso.com"

---
### <a name="lcase"></a>LCase
**Descripción:**  
la función LCase convierte todos los caracteres de una cadena a minúsculas.

**Sintaxis:**  
`str LCase(str value)`

**Ejemplo**:  
`LCase("TeSt")`  
devuelve "test".

---
### <a name="left"></a>Left
**Descripción:**  
la función Left devuelve un número especificado de caracteres desde la izquierda de una cadena.

**Sintaxis:**  
`str Left(str string, num NumChars)`

* string: la cadena desde la que devolver los caracteres
* NumChars: un número que identifica el número de caracteres que devolver desde el principio (izquierdo) de la cadena

**Observaciones**:  
una cadena que contiene los primeros caracteres numChars de la cadena:

* Con numChars = 0, se devuelve una cadena vacía.
* Con numChars < 0, se devuelve una cadena de entrada.
* Si la cadena es null, se devuelve una cadena vacía.

Si la cadena contiene menos caracteres que el número especificado en numChars, se devuelve una cadena idéntica a la cadena (es decir, que contiene todos los caracteres en el parámetro 1).

**Ejemplo**:  
`Left("John Doe", 3)`  
devuelve “Joh”.

---
### <a name="len"></a>Len
**Descripción:**  
la función Len devuelve un número de caracteres de una cadena.

**Sintaxis:**  
`num Len(str value)`

**Ejemplo**:  
`Len("John Doe")`  
devuelve 8.

---
### <a name="ltrim"></a>LTrim
**Descripción:**  
la función LTrim quita los espacios en blanco del principio de una cadena.

**Sintaxis:**  
`str LTrim(str value)`

**Ejemplo**:  
`LTrim(" Test ")`  
devuelve "Test".

---
### <a name="mid"></a>Mid
**Descripción:**  
la función Mid devuelve un número especificado de caracteres desde una posición especificada en una cadena.

**Sintaxis:**  
`str Mid(str string, num start, num NumChars)`

* string: la cadena desde la que devolver los caracteres
* start: un número que identifica la posición de inicio en una cadena desde la que devolver los caracteres
* NumChars: un número que identifica el número de caracteres que devolver desde la posición en una cadena

**Observaciones**:  
devuelve caracteres numChars que comienzan por la posición de inicio en una cadena.  
Una cadena que contiene caracteres numChars de la posición de inicio en una cadena:

* Con numChars = 0, se devuelve una cadena vacía.
* Con numChars < 0, se devuelve una cadena de entrada.
* Con start >,  la longitud de la cadena devuelve la cadena de entrada.
* Con start <= 0, se devuelve la cadena de entrada.
* Si la cadena es null, se devuelve una cadena vacía.

Si no hay caracteres numChar restantes en la cadena de la posición de inicio, se devuelve el máximo de caracteres posible.

**Ejemplo**:  
`Mid("John Doe", 3, 5)`  
devuelve “hn Do”.

`Mid("John Doe", 6, 999)`  
Devuelve "Doe".

---
### <a name="now"></a>Ahora
**Descripción:**  
la función Now devuelve DateTime que especifica la fecha y hora actuales, según la fecha y hora del sistema del equipo.

**Sintaxis:**  
`dt Now()`

---
### <a name="numfromdate"></a>NumFromDate
**Descripción:**  
la función NumFromDate devuelve una fecha en formato de fecha de AD.

**Sintaxis:**  
`num NumFromDate(dt value)`

**Ejemplo**:  
`NumFromDate(CDate("2012-01-01 23:00:00"))`  
devuelve 129699324000000000.

---
### <a name="padleft"></a>PadLeft
**Descripción:**  
la función PadLeft rellena en la parte izquierda una cadena con una longitud especificada mediante un carácter de espaciado interno proporcionado.

**Sintaxis:**  
`str PadLeft(str string, num length, str padCharacter)`

* string: la cadena que rellenar.
* length: un número entero que representa la longitud deseada de la cadena.
* padCharacter: una cadena que consta de un solo carácter que se usará como el carácter controlador

**Observaciones**:

* Si la longitud de cadena es inferior a la longitud, padCharacter se anexa repetidamente  al principio (izquierda) de la cadena hasta que tenga una longitud igual a la longitud.
* PadCharacter puede ser un carácter de espacio, pero no puede ser un valor Null.
* Si la longitud de cadena es igual o mayor que la longitud, la cadena se devuelve sin cambios.
* Si la cadena tiene una longitud mayor que o igual que la longitud, se devuelve una cadena idéntica a la cadena.
* Si la longitud de cadena es menor que la longitud, se devuelve una cadena nueva de la longitud deseada que contiene una cadena rellenada con un padCharacter.
* Si la cadena es Null, la función devuelve una cadena vacía.

**Ejemplo**:  
`PadLeft("User", 10, "0")`  
devuelve "000000User".

---
### <a name="padright"></a>PadRight
**Descripción:**  
la función PadRight rellena en la parte derecha una cadena con una longitud especificada mediante un carácter de espaciado interno proporcionado.

**Sintaxis:**  
`str PadRight(str string, num length, str padCharacter)`

* string: la cadena que rellenar.
* length: un número entero que representa la longitud deseada de la cadena.
* padCharacter: una cadena que consta de un solo carácter que se usará como el carácter controlador

**Observaciones**:

* Si la longitud de cadena es inferior a la longitud, padCharacter se anexa repetidamente  al final (derecha) de la cadena hasta que tenga una longitud igual a la longitud.
* PadCharacter puede ser un carácter de espacio, pero no puede ser un valor Null.
* Si la longitud de cadena es igual o mayor que la longitud, la cadena se devuelve sin cambios.
* Si la cadena tiene una longitud mayor que o igual que la longitud, se devuelve una cadena idéntica a la cadena.
* Si la longitud de cadena es menor que la longitud, se devuelve una cadena nueva de la longitud deseada que contiene una cadena rellenada con un padCharacter.
* Si la cadena es Null, la función devuelve una cadena vacía.

**Ejemplo**:  
`PadRight("User", 10, "0")`  
devuelve "User000000".

---
### <a name="pcase"></a>PCase
**Descripción:**  
la función PCase convierte el primer carácter de cada palabra delimitada por espacios de una cadena a mayúsculas, y todos los demás caracteres se convierten a minúsculas.

**Sintaxis:**  
`String PCase(string)`

**Observaciones**:

* Esta función no proporciona por el momento un uso de mayúsculas correcto para convertir una palabra que esté completamente en mayúsculas, como un acrónimo.

**Ejemplo**:  
`PCase("TEsT")`  
devuelve "test".

`PCase(LCase("TEST"))`  
devuelve "Test".

---
### <a name="randomnum"></a>RandomNum
**Descripción:**  
la función RandomNum devuelve un número aleatorio entre un intervalo especificado.

**Sintaxis:**  
`num RandomNum(num start, num end)`

* start: un número que identifica el límite inferior del valor aleatorio que se va a generar
* end: un número que identifica el límite superior del valor aleatorio que generar

**Ejemplo**:  
`Random(100,999)`  
puede devolver 734.

---
### <a name="removeduplicates"></a>RemoveDuplicates
**Descripción:**  
la función RemoveDuplicates toma una cadena multivalor y garantiza que cada valor es único.

**Sintaxis:**  
`mvstr RemoveDuplicates(mvstr attribute)`

**Ejemplo**:  
`RemoveDuplicates([proxyAddresses])`  
devuelve un atributo proxyAddress saneado donde se han quitado todos los valores duplicados.

---
### <a name="replace"></a>Replace
**Descripción:**  
la función Replace reemplaza todas las apariciones de una cadena por otra cadena.

**Sintaxis:**  
`str Replace(str string, str OldValue, str NewValue)`

* string: una cadena en la que se reemplazarán los valores.
* OldValue: la cadena que se va a buscar y reemplazar.
* NewValue: la cadena que reemplazar.

**Observaciones**:  
la función reconoce los siguientes monikers especiales:

* \n: nueva línea
* \r: retorno de carro
* \t: tabulación

**Ejemplo**:  
`Replace([address],"\r\n",", ")`  
reemplaza CRLF por una coma y un espacio, y podría originar "One Microsoft Way, Redmond, WA, USA".

---
### <a name="replacechars"></a>ReplaceChars
**Descripción:**  
la función ReplaceChars reemplaza todas las apariciones de caracteres encontrados en la cadena ReplacePattern.

**Sintaxis:**  
`str ReplaceChars(str string, str ReplacePattern)`

* string: una cadena por la que reemplazar los caracteres.
* ReplacePattern: una cadena que contiene un diccionario con caracteres que reemplazar.

El formato es {source1}:{target1},{source2}:{target2},{sourceN},{targetN} donde source es el carácter que buscar y target la cadena por la que reemplazar.

**Observaciones**:

* La función toma cada aparición de los orígenes definidos y los reemplaza por los destinos.
* El origen debe ser exactamente un carácter (unicode).
* El origen no puede estar vacío ni puede tener más de un carácter (error de análisis).
* El destino puede tener varios caracteres: por ejemplo, ö:oe, β:ss.
* El destino puede estar vacío, lo que indica que el carácter debe quitarse.
* El origen distingue mayúsculas y minúsculas, y debe ser una coincidencia exacta.
* La coma (,) y los dos puntos (:) son caracteres reservados y no se puede reemplazar utilizando esta función.
* Se omiten los espacios y otros caracteres en blanco en la cadena ReplacePattern.

**Ejemplo**:  
`%ReplaceString% = ’:,Å:A,Ä:A,Ö:O,å:a,ä:a,ö,o`

`ReplaceChars("Räksmörgås",%ReplaceString%)`  
Devuelve Raksmorgas.

`ReplaceChars("O’Neil",%ReplaceString%)`  
Devuelve "ONeil"; se define la marca única para quitarla.

---
### <a name="right"></a>Right
**Descripción:**  
la función Right devuelve un número especificado de caracteres desde la derecha (final) de una cadena.

**Sintaxis:**  
`str Right(str string, num NumChars)`

* string: la cadena desde la que devolver los caracteres
* NumChars: un número que identifica el número de caracteres que devolver desde el final (derecho) de la cadena

**Observaciones**:  
los caracteres NumChars se devuelven a partir de la última posición de la cadena.

Una cadena que contiene los últimos caracteres numChars de la cadena:

* Con numChars = 0, se devuelve una cadena vacía.
* Con numChars < 0, se devuelve una cadena de entrada.
* Si la cadena es null, se devuelve una cadena vacía.

Si la cadena contiene menos caracteres que el número especificado en NumChars, se devuelve una cadena idéntica a la cadena.

**Ejemplo**:  
`Right("John Doe", 3)`  
devuelve "Doe".

---
### <a name="rtrim"></a>RTrim
**Descripción:**  
la función RTrim quita los espacios en blanco del final de una cadena.

**Sintaxis:**  
`str RTrim(str value)`

**Ejemplo**:  
`RTrim(" Test ")`  
devuelve "Test".

---
### <a name="select"></a>Seleccionar
**Descripción:**  
Procesa todos los valores de un atributo de valor múltiple (o el resultado de una expresión) según la función especificada.

**Sintaxis:**  
`mvattr Select(variable item, mvattr attribute, func function)`  
`mvattr Select(variable item, exp expression, func function)`

* item: representa un elemento del atributo de valor múltiple
* attribute: el atributo de valor múltiple
* expression: una expresión que devuelve una colección de valores
* condition: cualquier función que pueda procesar un elemento en el atributo

**Ejemplos:**  
`Select($item,[otherPhone],Replace($item,"-",""))`  
Devuelva todos los valores del atributo de valor múltiple otherPhone una vez que se han quitado los guiones (-).

---
### <a name="split"></a>Dividir
**Descripción:**  
la función Split toma una cadena separada por un delimitador y la convierte en una cadena multivalor.

**Sintaxis:**  
`mvstr Split(str value, str delimiter)`  
`mvstr Split(str value, str delimiter, num limit)`

* value: la cadena con un carácter delimitador que separar.
* delimiter: carácter único que usar como delimitador.
* limit: número máximo de valores que se pueden devolver.

**Ejemplo**:  
`Split("SMTP:john.doe@contoso.com,smtp:jd@contoso.com",",")`  
devuelve una cadena multivalor con dos elementos útiles para el atributo proxyAddress.

---
### <a name="stringfromguid"></a>StringFromGuid
**Descripción:**  
la función StringFromGuid toma un GUID binario y lo convierte en una cadena.

**Sintaxis:**  
`str StringFromGuid(bin GUID)`

---
### <a name="stringfromsid"></a>StringFromSid
**Descripción:**  
la función StringFromSid convierte una matriz de bytes o una matriz de bytes multivalor que contiene un identificador de seguridad en una cadena o en una cadena multivalor.

**Sintaxis:**  
`str StringFromSid(bin ObjectSID)`  

---
### <a name="switch"></a>Switch
**Descripción:**  
la función Switch se utiliza para devolver un único valor según las condiciones evaluadas.

**Sintaxis:**  
`var Switch(exp expr1, var value1[, exp expr2, var value … [, exp expr, var valueN]])`

* expr: expresión variante que desea evaluar.
* value: valor que se va a devolver si la expresión correspondiente es True.

**Observaciones**:  
la lista de argumentos de la función Switch consta de pares de expresiones y valores. Las expresiones se evalúan de izquierda a derecha y se devuelve el valor asociado a la primera expresión que evaluar en True. Si los elementos no están emparejados correctamente, se produce un error en tiempo de ejecución.

Por ejemplo, si expr1 es True, Switch devuelve value1. Si expr-1 es False, pero expr-2 es True, Switch devuelve value-2, etc.

Switch devuelve Nothing si:

* Ninguna de las expresiones son True.
* La primera expresión True tiene un valor correspondiente que es Null.

Switch evalúa todas las expresiones, aunque devuelva solo una de ellas. Por este motivo, debe vigilar efectos secundarios no deseados. Por ejemplo, si la evaluación de cualquier expresión tiene como resultado una división por cero, se produce un error.

El valor puede ser también la función Error que devolvería una cadena personalizada.

**Ejemplo**:  
`Switch([city] = "London", "English", [city] = "Rome", "Italian", [city] = "Paris", "French", True, Error("Unknown city"))`  
Devuelve el idioma hablado en las ciudades más importantes. De lo contrario, devuelve un error.

---
### <a name="trim"></a>Trim
**Descripción:**  
la función Trim quita los espacios en blanco del principio y del final de una cadena.

**Sintaxis:**  
`str Trim(str value)`  

**Ejemplo**:  
`Trim(" Test ")`  
devuelve "test".

`Trim([proxyAddresses])`  
Quita los espacios del principio y del final de cada valor en el atributo proxyAddress.

---
### <a name="ucase"></a>UCase
**Descripción:**  
: la función UCase convierte todos los caracteres de una cadena a mayúsculas.

**Sintaxis:**  
`str UCase(str string)`

**Ejemplo**:  
`UCase("TeSt")`  
devuelve "test".

---
### <a name="where"></a>Where

**Descripción:**  
Devuelve un subconjunto de valores de un atributo de valor múltiple (o el resultado de una expresión) según la función especificada.

**Sintaxis:**  
`mvattr Where(variable item, mvattr attribute, exp condition)`  
`mvattr Where(variable item, exp expression, exp condition)`  
* item: representa un elemento del atributo de valor múltiple
* attribute: el atributo de valor múltiple
* condition: cualquier expresión que pueda evaluarse en True o False
* expression: una expresión que devuelve una colección de valores

**Ejemplo**:  
`Where($item,[userCertificate],CertNotAfter($item)>Now())`  
Devuelva los valores de certificado en el atributo de valor múltiple userCertificate que no han expirado.

---
### <a name="with"></a>With
**Descripción:**  
La función With proporciona una manera de simplificar una expresión compleja usando una variable para representar una subexpresión que aparece una o varias veces en la expresión compleja.

**Sintaxis:** 
`With(var variable, exp subExpression, exp complexExpression)`  
* variable: representa la subexpresión.
* subExpression: la que la variable representa.
* complexExpression: una expresión compleja.

**Ejemplo**:  
`With($unExpiredCerts,Where($item,[userCertificate],CertNotAfter($item)>Now()),IIF(Count($unExpiredCerts)>0,$unExpiredCerts,NULL))`  
Es funcionalmente equivalente a:  
`IIF (Count(Where($item,[userCertificate],CertNotAfter($item)>Now()))>0, Where($item,[userCertificate],CertNotAfter($item)>Now()),NULL)`  
Que devuelve solo los valores de los certificados que no han expirado en el atributo userCertificate.


---
### <a name="word"></a>Word
**Descripción:**  
: la función Word devuelve una palabra incluida en una cadena, según los parámetros que describen los delimitadores que se usarán y el número de palabras que se devolverán.

**Sintaxis:**  
`str Word(str string, num WordNumber, str delimiters)`

* string: la cadena desde la que devolver una palabra.
* WordNumber: un número que identifica qué número de palabras debe devolver.
* delimiters: una cadena que representa los delimitadores que deben usarse para identificar palabras

**Observaciones**:  
: cada cadena de caracteres en la cadena separada por uno de los caracteres en los delimitadores se identifica como palabras:

* Si el número es < 1, se devuelve una cadena vacía.
* Si la cadena es Null, se devuelve una cadena vacía.

Si la cadena contiene menos palabras o si la cadena no contiene palabras identificadas por los delimitadores, se devuelve una cadena vacía.

**Ejemplo**:  
`Word("The quick brown fox",3," ")`  
devuelve "brown".

`Word("This,string!has&many separators",3,",!&#")`  
devolvería "has".

## <a name="additional-resources"></a>Recursos adicionales
* [Explicación de las expresiones declarativas de aprovisionamiento](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)
* [Sincronización de Azure AD Connect: personalización de las opciones de sincronización](how-to-connect-sync-whatis.md)
* [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md)