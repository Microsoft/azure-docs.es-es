---
title: 'Reglas de pertenencia a grupos rellenados de forma dinámica: Azure AD | Microsoft Docs'
description: Creación de reglas de pertenencia para rellenar automáticamente grupos y creación de una referencia de regla.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: overview
ms.date: 08/13/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: f36e5d8974caea0eecff7e0b399b6aab5d200664
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797112"
---
# <a name="dynamic-membership-rules-for-groups-in-azure-active-directory"></a>Reglas de pertenencia dinámica a grupos de Azure Active Directory

En Azure Active Directory (Azure AD), puede crear reglas basadas en atributos complejos para habilitar la pertenencia dinámica a grupos. La pertenencia dinámica a grupos reduce la sobrecarga administrativa que conlleva agregar y eliminar usuarios. En este artículo se detallan las propiedades y la sintaxis para crear reglas de pertenencia dinámica para usuarios o dispositivos. Puede configurar una regla de pertenencia dinámica en grupos de seguridad o grupos de Microsoft 365.

Cuando cambia cualquier atributo de un usuario, el sistema evalúa todas las reglas de grupos dinámicos de un directorio para ver si la modificación desencadenaría adiciones o retiradas en el grupo. Si un usuario o dispositivo cumple una regla de un grupo, se agrega a este como miembro. Cuando ya no cumple la regla, se quita. No se puede agregar o quitar un miembro de un grupo dinámico de forma manual.

- Puede crear un grupo dinámico para dispositivos o usuarios, pero no se puede crear una regla que contenga tanto usuarios como dispositivos.
- No se puede crear un grupo de dispositivos basado en los atributos de los propietarios de los dispositivos. Las reglas de pertenencia de dispositivo solo pueden hacer referencia a atributos de dispositivos.

> [!NOTE]
> Esta característica requiere una licencia de Azure AD Premium P1 para cada usuario único que sea miembro de uno o varios grupos dinámicos. Aunque no es necesario asignar licencias a los usuarios para que sean miembros de los grupos dinámicos, es preciso tener el número mínimo de licencias en la organización de Azure AD para abarcarlos a todos. Por ejemplo, si tiene un total de 1000 usuarios únicos en todos los grupos dinámicos de la organización, necesitará al menos 1000 licencias de Azure AD Premium P1 para cumplir el requisito de licencia.
> No es necesaria ninguna licencia para los dispositivos que son miembros de un grupo de dispositivos dinámico.

## <a name="rule-builder-in-the-azure-portal"></a>Generador de reglas en Azure Portal

Azure AD proporciona un generador de reglas para crear y actualizar las reglas importantes con mayor rapidez. El generador de reglas admite la construcción de hasta cinco expresiones. Facilita la creación de reglas con unas cuantas expresiones sencillas; no obstante, no se puede usar para reproducir todas las reglas. En caso de que no admita la regla que quiere crear, puede usar el cuadro de texto.

Estos son algunos ejemplos de reglas o sintaxis avanzadas para las que se recomienda construir mediante el cuadro de texto:

- Regla con más de cinco expresiones
- La regla de subordinados directos
- Configuración de la [precedencia de operadores](groups-dynamic-membership.md#operator-precedence)
- [Reglas con expresiones complejas](groups-dynamic-membership.md#rules-with-complex-expressions); por ejemplo, `(user.proxyAddresses -any (_ -contains "contoso"))`

> [!NOTE]
> Es posible que el generador de reglas no pueda mostrar algunas reglas construidas en el cuadro de texto. En este caso, podría aparecer un mensaje. El generador de reglas no cambia la sintaxis admitida, la validación ni el procesamiento de reglas de grupos dinámicos de ninguna manera.

Para obtener instrucciones paso a paso, consulte [Creación o actualización de un grupo dinámico](groups-create-rule.md).

![Adición de una regla de pertenencia a un grupo dinámico](./media/groups-dynamic-membership/update-dynamic-group-rule.png)

### <a name="rule-syntax-for-a-single-expression"></a>Sintaxis de regla para una sola expresión

Una expresión única es la forma más sencilla de una regla de pertenencia y solo tiene los tres elementos mencionados anteriormente. Una regla con una expresión única tiene un aspecto similar al siguiente: `Property Operator Value`, donde la sintaxis de la propiedad es el nombre de object.property.

El siguiente es un ejemplo de una regla de pertenencia construida correctamente con una expresión única:

```
user.department -eq "Sales"
```

Los paréntesis son opcionales para una expresión única. La longitud total del cuerpo de la regla de pertenencia no puede superar los 2048 caracteres.

## <a name="constructing-the-body-of-a-membership-rule"></a>Construcción del cuerpo de una regla de pertenencia

Una regla de pertenencia que rellena automáticamente un grupo con usuarios o dispositivos es una expresión binaria que genera un resultado true o false. Los tres elementos de una regla simple son:

- Propiedad
- Operator
- Value

El orden de los elementos de una expresión es importante para evitar errores de sintaxis.

## <a name="supported-properties"></a>Propiedades admitidas

Hay tres tipos de propiedades que se pueden usar para construir una regla de pertenencia.

- Boolean
- String
- Colección de cadenas

Las siguientes son las propiedades de usuario que puede utilizar para crear una expresión única.

### <a name="properties-of-type-boolean"></a>Propiedades de tipo booleano

| Propiedades | Valores permitidos | Uso |
| --- | --- | --- |
| accountEnabled |true false |user.accountEnabled -eq true |
| dirSyncEnabled |true false |user.dirSyncEnabled -eq true |

### <a name="properties-of-type-string"></a>Propiedades de tipo cadena

| Propiedades | Valores permitidos | Uso |
| --- | --- | --- |
| city |Cualquier valor de cadena o *null* |(user.city -eq "value") |
| country |Cualquier valor de cadena o *null* |(user.country -eq "value") |
| companyName | Cualquier valor de cadena o *null* | (user.companyName -eq "value") |
| department |Cualquier valor de cadena o *null* |(user.department -eq "value") |
| DisplayName |Cualquier valor de cadena |(user.displayName -eq "value") |
| employeeId |Cualquier valor de cadena |(user.employeeId -eq "value")<br>(user.employeeId -ne *null*) |
| facsimileTelephoneNumber |Cualquier valor de cadena o *null* |(user.facsimileTelephoneNumber -eq "value") |
| givenName |Cualquier valor de cadena o *null* |(user.givenName -eq "value") |
| jobTitle |Cualquier valor de cadena o *null* |(user.jobTitle -eq "value") |
| mail |Cualquier valor de cadena o *null* (dirección SMTP del usuario) |(user.mail -eq "value") |
| mailNickName |Cualquier valor de cadena (alias de correo electrónico del usuario) |(user.mailNickName -eq "value") |
| mobile |Cualquier valor de cadena o *null* |(user.mobile -eq "value") |
| objectId |GUID del objeto de usuario |(user.objectId -eq "11111111-1111-1111-1111-111111111111") |
| onPremisesSecurityIdentifier | Identificador de seguridad local (SID) para los usuarios que se han sincronizado desde local a la nube. |(user.onPremisesSecurityIdentifier -eq "S-1-1-11-1111111111-1111111111-1111111111-1111111") |
| passwordPolicies |None DisableStrongPassword DisablePasswordExpiration DisablePasswordExpiration, DisableStrongPassword |(user.passwordPolicies -eq "DisableStrongPassword") |
| physicalDeliveryOfficeName |Cualquier valor de cadena o *null* |(user.physicalDeliveryOfficeName -eq "value") |
| postalCode |Cualquier valor de cadena o *null* |(user.postalCode -eq "value") |
| preferredLanguage |Código ISO 639-1 |(user.preferredLanguage -eq "en-US") |
| sipProxyAddress |Cualquier valor de cadena o *null* |(user.sipProxyAddress -eq "value") |
| state |Cualquier valor de cadena o *null* |(user.state -eq "value") |
| streetAddress |Cualquier valor de cadena o *null* |(user.streetAddress -eq "value") |
| surname |Cualquier valor de cadena o *null* |(user.surname -eq "value") |
| telephoneNumber |Cualquier valor de cadena o *null* |(user.telephoneNumber -eq "value") |
| usageLocation |Código de dos letras de país o región |(user.usageLocation -eq "US") |
| userPrincipalName |Cualquier valor de cadena |(user.userPrincipalName -eq "alias@domain") |
| userType |miembro invitado *null* |(user.userType -eq "Member") |

### <a name="properties-of-type-string-collection"></a>Propiedades de colección de cadenas de tipo

| Propiedades | Valores permitidos | Uso |
| --- | --- | --- |
| otherMails |Cualquier valor de cadena |(user.otherMails -contains "alias@domain") |
| proxyAddresses |SMTP: alias@domain smtp: alias@domain |(user.proxyAddresses -contains "SMTP: alias@domain") |

Para más información sobre las propiedades que se usan para las reglas de dispositivos, consulte [Reglas de dispositivos](#rules-for-devices).

## <a name="supported-expression-operators"></a>Operadores de expresión admitidos

En la tabla siguiente se enumeran todos los operadores admitidos y su sintaxis para una expresión única. Los operadores se pueden utilizar con o sin el prefijo de guion (-).

| Operator | Sintaxis |
| --- | --- |
| Not Equals |-ne |
| Equals |-eq |
| Not Starts With |-notStartsWith |
| Empieza por |-startsWith |
| Not Contains |-notContains |
| Contains |-contains |
| Not Match |-notMatch |
| Match |-match |
| En | -in |
| No en el | -notIn |

### <a name="using-the--in-and--notin-operators"></a>Uso de los operadores -In y -notIn

Si desea comparar el valor de un atributo de usuario con un número de valores diferentes, puede usar los operadores -In o -notIn. Utilice los símbolos entre corchetes "[" y "]" para comenzar y terminar la lista de valores.

 En el ejemplo siguiente, la expresión se evalúa como true si el valor de user.department es igual a cualquiera de los valores de la lista:

```
   user.department -in ["50001","50002","50003","50005","50006","50007","50008","50016","50020","50024","50038","50039","51100"]
```


### <a name="using-the--match-operator"></a>Uso del operador -match 
El operador **-coinciden** se utiliza para que coincida con cualquier expresión regular. Ejemplos:

```
user.displayName -match "Da.*"   
```
Da, Dav y David se evalúan como true, aDa se evalúa como false.

```
user.displayName -match ".*vid"
```
David se evalúa como true, Da como false.

## <a name="supported-values"></a>Valores admitidos

Los valores utilizados en una expresión pueden ser de varios tipos, incluidos:

* Cadenas
* Valor booleano: true, false
* Números
* Matrices: matriz numérica, matriz de cadenas

Al especificar un valor dentro de una expresión es importante utilizar la sintaxis correcta para evitar errores. Algunas sugerencias de sintaxis son:

* Las comillas dobles son opcionales a menos que el valor sea una cadena.
* Las operaciones de cadena y regex no distinguen mayúsculas de minúsculas.
* Cuando un valor de cadena contiene comillas dobles, se debe utilizar \` como carácter de escape de ambas. Por ejemplo, user.department -eq \`"Sales\`" es la sintaxis correcta cuando "Sales" es el valor.
* También puede realizar comprobaciones null, usando null como valor, por ejemplo, `user.department -eq null`.

### <a name="use-of-null-values"></a>Uso de valores nulos

Para especificar un valor nulo en una regla, puede usar el valor *null*. 

* Use -eq o -ne al comparar el valor *null* de una expresión.
* Use comillas alrededor de la palabra *null* solo si desea que se interprete como un valor de cadena literal.
* El operador -not no se puede usar como operador comparativo con valores null. Si lo hace, recibirá un error tanto si usa null como si usa $null.

La manera correcta de hacer referencia al valor null es como sigue:

```
   user.mail –ne null
```

## <a name="rules-with-multiple-expressions"></a>Reglas con varias expresiones

Una regla de pertenencia a grupos puede constar de más de una expresión única conectadas por los operadores lógicos -and, -or y -not. Los operadores lógicos también se pueden usar en combinación. 

Los siguientes son ejemplos de reglas de pertenencia construidas correctamente con varias expresiones:

```
(user.department -eq "Sales") -or (user.department -eq "Marketing")
(user.department -eq "Sales") -and -not (user.jobTitle -contains "SDE")
```

### <a name="operator-precedence"></a>Precedencia de operadores

Todos los operadores se enumeran a continuación, en orden de prioridad de mayor a menor. Los operadores en la misma línea tienen la misma prioridad:

```
-eq -ne -startsWith -notStartsWith -contains -notContains -match –notMatch -in -notIn
-not
-and
-or
-any -all
```

Este es un ejemplo de prioridad de operador en el que el usuario evalúa dos expresiones:

```
   user.department –eq "Marketing" –and user.country –eq "US"
```

Los paréntesis son necesarios solo si la precedencia no cumple sus requisitos. Por ejemplo, si desea que el departamento se evalúe primero, el ejemplo siguiente muestra cómo se deben usar los paréntesis para determinar el orden:

```
   user.country –eq "US" –and (user.department –eq "Marketing" –or user.department –eq "Sales")
```

## <a name="rules-with-complex-expressions"></a>Reglas con expresiones complejas

Una regla de pertenencia puede constar de expresiones complejas donde las propiedades, los operadores y los valores pueden adoptar formas más complejas. Las expresiones se consideran complejas cuando se cumple alguna de las siguientes condiciones:

* La propiedad consta de una colección de valores, especialmente las propiedades con varios valores
* Las expresiones usan los operadores -any y -all
* El valor de la expresión puede incluir por sí mismo una o varias expresiones

## <a name="multi-value-properties"></a>Propiedades de varios valores

Las propiedades de varios valores son colecciones de objetos del mismo tipo. Se pueden usar para crear reglas de pertenencia mediante los operadores lógicos -any y -all.

| Propiedades | Valores | Uso |
| --- | --- | --- |
| assignedPlans | Cada objeto de la colección expone las siguientes propiedades de cadena: capabilityStatus, service, servicePlanId |user.assignedPlans -any (assignedPlan.servicePlanId -eq "efb87545-963c-4e0d-99df-69c6916d9eb0" -and assignedPlan.capabilityStatus -eq "Enabled") |
| proxyAddresses| SMTP: alias@domain smtp: alias@domain | (user.proxyAddresses -any (\_ -contains "contoso")) |

### <a name="using-the--any-and--all-operators"></a>Uso de los operadores -any y -all

Puede usar los operadores -any y -all para aplicar una condición a uno o todos los elementos de la colección, respectivamente.

* -any (se satisface cuando al menos un elemento de la colección coincide con la condición)
* -all (se satisface cuando todos los elementos de la colección coinciden con la condición)

#### <a name="example-1"></a>Ejemplo 1

assignedPlans es una propiedad de varios valores que muestra todos los planes de servicio asignados al usuario. En la expresión siguiente se seleccionarán los usuarios que tienen el plan de servicio (como valor de GUID) Exchange Online (Plan 2) que también está en estado Habilitado:

```
user.assignedPlans -any (assignedPlan.servicePlanId -eq "efb87545-963c-4e0d-99df-69c6916d9eb0" -and assignedPlan.capabilityStatus -eq "Enabled")
```

Una regla como esta se puede utilizar para agrupar todos los usuarios en los que la funcionalidad de Microsoft 365 (u otro servicio en línea de Microsoft) está habilitada. A continuación, podría solicitar la admisión en el grupo con un conjunto de directivas.

#### <a name="example-2"></a>Ejemplo 2

En la expresión siguiente se seleccionan todos los usuarios que tengan algún plan de servicio asociado con el servicio de Intune (identificado por el nombre de servicio "SCO"):

```
user.assignedPlans -any (assignedPlan.service -eq "SCO" -and assignedPlan.capabilityStatus -eq "Enabled")
```

### <a name="using-the-underscore-_-syntax"></a>Con la sintaxis de guión bajo (\_)

La sintaxis de guión bajo (\_) coincide con las apariciones de un valor específico en una de las propiedades de la colección de cadenas multivalor para agregar usuarios o dispositivos a un grupo dinámico. Se usa con los operadores -any u -all.

Este es un ejemplo de uso del guión bajo (\_) en una regla para agregar miembros basados en user.proxyAddress (funciona de la misma forma para user.otherMails). Esta regla agrega cualquier usuario con la dirección de proxy que contiene "contoso" al grupo.

```
(user.proxyAddresses -any (_ -contains "contoso"))
```

## <a name="other-properties-and-common-rules"></a>Otras propiedades y reglas habituales

### <a name="create-a-direct-reports-rule"></a>Creación de una regla de "subordinados directos"

Puede crear un grupo con todos los subordinados directos de un administrador. Cuando los subordinados directos de un administrador cambien en el futuro, la pertenencia del grupo se ajustará automáticamente.

La regla de subordinados directos se construye mediante la sintaxis siguiente:

```
Direct Reports for "{objectID_of_manager}"
```

El siguiente es un ejemplo de una regla válida donde "62e19b97-8b3d-4d4a-a106-4ce66896a863" es el identificador de objeto del administrador:

```
Direct Reports for "62e19b97-8b3d-4d4a-a106-4ce66896a863"
```

Las siguientes sugerencias pueden ayudarle a usar correctamente la regla.

- El **identificador del administrador** es el identificador de objeto del administrador. Se puede encontrar en el **perfil** del administrador.
- Para que la regla funcione, asegúrese de que la propiedad de **administrador** esté establecida correctamente para los usuarios de la organización. Puede comprobar el valor actual en el **perfil** de usuario.
- Esta regla admite solo subordinados directos del administrador. En otras palabras, no se puede crear un grupo con los subordinados directos del administrador *y* con los subordinados de estos.
- Esta regla no se puede combinar con ninguna otra regla de pertenencia.

### <a name="create-an-all-users-rule"></a>Creación de una regla de "todos los usuarios"

Puede crear un grupo que contenga todos los usuarios de una organización mediante una regla de pertenencia. Cuando se agreguen o eliminen usuarios de la organización en el futuro, la pertenencia al grupo se ajustará automáticamente.

La regla de "Todos los usuarios" se construye con una expresión única con el operador -ne y el valor null. Esta regla agrega usuarios invitados de B2B, así como usuarios miembros al grupo.

```
user.objectId -ne null
```
Si quiere que el grupo excluya los usuarios invitados e incluya solo los miembros de la organización, puede usar la siguiente sintaxis:

```
(user.objectId -ne null) -and (user.userType -eq "Member")
```

### <a name="create-an-all-devices-rule"></a>Creación de una regla de "Todos los dispositivos"

Puede crear un grupo que contenga todos los dispositivos de una organización mediante una regla de pertenencia. Cuando se agreguen o eliminen dispositivos de la organización en el futuro, la pertenencia al grupo se ajustará automáticamente.

La regla de "Todos los dispositivos" se construye con una expresión única con el operador -ne y el valor null:

```
device.objectId -ne null
```

## <a name="extension-properties-and-custom-extension-properties"></a>Propiedades de extensión y propiedades de extensión personalizadas

Las reglas de pertenencia dinámica admiten los atributos de extensión y las propiedades de extensión personalizadas como propiedades de cadena. Los [atributos de extensión](/graph/api/resources/onpremisesextensionattributes?view=graph-rest-1.0) se sincronizan desde Windows Server AD local y tienen el formato "ExtensionAttributeX", donde X es igual a un número del 1 al 15. Este es un ejemplo de una regla que utiliza un atributo de extensión como propiedad:

```
(user.extensionAttribute15 -eq "Marketing")
```

Las [propiedades de extensión personalizadas](../hybrid/how-to-connect-sync-feature-directory-extensions.md) se sincronizan desde una instancia de Windows Server AD local o desde una aplicación SaaS conectada y presentan el formato `user.extension_[GUID]_[Attribute]`, donde:

* [GUID] es el identificador único de Azure AD para la aplicación que creó la propiedad en Azure AD
* [Attribute] es el nombre de la propiedad tal y como se creó

Un ejemplo de una regla que utiliza una propiedad de extensión personalizada es:

```
user.extension_c272a57b722d4eb29bfe327874ae79cb_OfficeNumber -eq "123"
```

El nombre de la propiedad personalizada se puede encontrar en el directorio mediante la consulta de una propiedad de usuario a través del explorador de Graph y la búsqueda del nombre en cuestión. También, ahora puede seleccionar el vínculo **Get custom extension properties** (Obtener propiedades de extensión personalizadas) en el generador de reglas dinámicas de grupos de usuarios para escribir un identificador de aplicación único y recibir la lista completa de propiedades de extensión personalizadas para usar al crear una regla de pertenencia dinámica. Esta lista también se puede actualizar esta lista para obtener nuevas propiedades de extensión personalizada para la aplicación.

## <a name="rules-for-devices"></a>Reglas de dispositivos

También puede crear una regla que selecciona objetos de dispositivo para la pertenencia de un grupo. No puede tener usuarios y dispositivos como miembros del grupo a la vez. 

> [!NOTE]
> El atributo **organizationalUnit** ya no aparece y no debe usarse. Intune establece esta cadena en casos concretos, pero Azure AD no la reconoce, por lo que no se agregan dispositivos a grupos basados este atributo.

> [!NOTE]
> systemlabels es un atributo de solo lectura que no se puede establecer con Intune.
>
> En Windows 10, el formato correcto del atributo deviceOSVersion es el siguiente: (device.deviceOSVersion -eq "10.0.17763"). El formato se puede validar con el cmdlet Get-MsolDevice PowerShell.

Pueden utilizarse los siguientes atributos del dispositivo.

 Atributo de dispositivo  | Valores | Ejemplo
 ----- | ----- | ----------------
 accountEnabled | true false | (device.accountEnabled -eq true)
 DisplayName | Cualquier valor de cadena |(device.displayName -eq "Rob iPhone")
 deviceOSType | Cualquier valor de cadena | (device.deviceOSType -eq "iPad") -or (device.deviceOSType -eq "iPhone")<br>(device.deviceOSType -contains "AndroidEnterprise")<br>(device.deviceOSType -eq "AndroidForWork")
 deviceOSVersion | Cualquier valor de cadena | (device.deviceOSVersion -eq "9.1")
 deviceCategory | un nombre de la categoría de dispositivo válido | (device.deviceCategory -eq "BYOD")
 deviceManufacturer | Cualquier valor de cadena | (device.deviceManufacturer -eq "Samsung")
 deviceModel | Cualquier valor de cadena | (device.deviceModel -eq "iPad Air")
 deviceOwnership | Personal, empresa, desconocido | (device.deviceOwnership -eq "Company")
 enrollmentProfileName | Nombre de perfil de inscripción de dispositivo de Apple, nombre de perfil de inscripción de dispositivo dedicado de Android Enterprise o nombre de perfil de Windows Autopilot | (device.enrollmentProfileName -eq "DEP iPhones")
 isRooted | true false | (device.isRooted -eq true)
 managementType | MDM (para dispositivos móviles)<br>PC (para equipos administrados por el agente de PC de Intune) | (device.managementType -eq "MDM")
 deviceId | un id. de dispositivo de Azure AD válido | (device.deviceId -eq "d4fe7726-5966-431c-b3b8-cddc8fdb717d")
 objectId | un id. de objeto de Azure AD válido |  (device.objectId -eq "76ad43c9-32c5-45e8-a272-7b58b58f596d")
 devicePhysicalIds | cualquier valor de cadena usado por Autopilot, como todos los dispositivos AutoPilot, OrderID o PurchaseOrderID  | (device.devicePhysicalIDs -any _ -contains "[ZTDId]") (device.devicePhysicalIds -any _ -eq "[OrderID]:179887111881") (device.devicePhysicalIds -any _ -eq "[PurchaseOrderId]:76222342342")
 systemLabels | cualquier cadena que coincida con la propiedad de dispositivo de Intune para etiquetar dispositivos del área de trabajo moderno | (device.systemLabels -contains "M365Managed")

> [!Note]  
> Para deviceOwnership, al crear grupos dinámicos para dispositivos, tiene que establecer el valor igual a "Compañía". En Intune la propiedad del dispositivo se representa en su lugar como Corporativa. Consulte [OwnerTypes](/intune/reports-ref-devices#ownertypes) para obtener más detalles. 

## <a name="next-steps"></a>Pasos siguientes

En estos artículos se proporciona información adicional sobre los grupos en Azure Active Directory.

- [Consulta de los grupos existentes](../fundamentals/active-directory-groups-view-azure-portal.md)
- [Crear un nuevo grupo y agregar miembros](../fundamentals/active-directory-groups-create-azure-portal.md)
- [Administración de la configuración de un grupo](../fundamentals/active-directory-groups-settings-azure-portal.md)
- [Administrar la pertenencia a grupos](../fundamentals/active-directory-groups-membership-azure-portal.md)
- [Administrar reglas dinámicas de los usuarios de un grupo](groups-create-rule.md)