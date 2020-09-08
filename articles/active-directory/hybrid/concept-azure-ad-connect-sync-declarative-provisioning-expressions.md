---
title: 'Azure AD Connect: expresiones de aprovisionamiento declarativo | Microsoft Docs'
description: Explica las expresiones declarativas de aprovisionamiento.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: e3ea53c8-3801-4acf-a297-0fb9bb1bf11d
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/18/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 02490839a9e35695ae2e8b3f750e139ad7413aa4
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89280220"
---
# <a name="azure-ad-connect-sync-understanding-declarative-provisioning-expressions"></a>Sincronización de Azure AD Connect: conocimiento de expresiones de aprovisionamiento declarativo
La sincronización de Azure AD Connect se basa en el aprovisionamiento declarativo, que se introdujo en Forefront Identity Manager 2010. Permite implementar la lógica de negocios de integración de identidades completa sin necesidad de escribir código compilado.

Una parte esencial del aprovisionamiento declarativo es el lenguaje de expresiones que se usa en flujos de atributos. El lenguaje usado es un subconjunto de Microsoft® Visual Basic® para Aplicaciones. Este lenguaje se usa en Microsoft Office, y los usuarios con experiencia en VBScript también lo reconocerán. El lenguaje de expresiones de aprovisionamiento declarativo solo utiliza funciones y no es un lenguaje estructurado. No hay métodos ni instrucciones. En su lugar, las funciones se anidan al flujo de programa rápido.

Para más información, vea [Referencia del lenguaje VBA para Office 2013](/office/vba/api/overview/language-reference).

Los atributos están fuertemente tipados. Las funciones solo aceptan atributos del tipo correcto. También distinguen mayúsculas de minúsculas. Es preciso usar correctamente las mayúsculas y minúsculas tanto en los nombres de función como en los nombres de atributo deben tener o se producirá un error.

## <a name="language-definitions-and-identifiers"></a>Identificadores y definiciones de idioma
* Las funciones tienen un nombre seguido de argumentos entre paréntesis: FunctionName(argumento 1, argumento N).
* Los atributos se especifican entre corchetes: [attributeName]
* Los parámetros se identifican por signos de porcentaje: %ParameterName%
* Las constantes de cadena aparecen entre comillas: por ejemplo, "Contoso" (Nota: se deben usar comillas rectas "", no tipográficas "")
* Los valores numéricos se expresan sin comillas y se espera que sean decimales. Los valores hexadecimales van precedidos de &H. Por ejemplo, 98052, &HFF
* Los valores booleanos se expresan con constantes: True, False.
* Las constantes y los literales integrados se expresan solo con su nombre: NULL, CRLF o IgnoreThisFlow

### <a name="functions"></a>Functions
El aprovisionamiento declarativo usa muchas funciones que ofrecen la posibilidad de transformar los valores de atributo. Dichas funciones se pueden anidar de manera que el resultado de una función se pase a otra función.

`Function1(Function2(Function3()))`

La lista completa de funciones se puede encontrar en la [referencia de funciones](reference-connect-sync-functions-reference.md).

### <a name="parameters"></a>Parámetros
Un parámetro se define por un conector o por un administrador mediante PowerShell. Los parámetros suelen contener valores que son diferentes de un sistema a otro, por ejemplo, el nombre del dominio en el que está ubicado el usuario. Dichos parámetros se pueden usar en flujos de atributos.

El conector de Active Directory proporcionó los siguientes parámetros para las reglas de sincronización entrantes:

| Nombre de parámetro | Comentario |
| --- | --- |
| Domain.Netbios |El formato Netbios del dominio importado actualmente, por ejemplo, FABRIKAMSALES |
| Domain.FQDN |El formato FQDN del dominio importado actualmente, por ejemplo, sales.fabrikam.com |
| Domain.LDAP |El formato LDAP del dominio importado actualmente, por ejemplo, DC=sales,DC=fabrikam,DC=com |
| Forest.Netbios |El formato Netbios del nombre de bosque importado actualmente, por ejemplo, FABRIKAMCORP |
| Forest.FQDN |El formato FQDN del nombre de bosque importado actualmente, por ejemplo, fabrikam.com |
| Forest.LDAP |El formato LDAP del nombre de bosque importado actualmente, por ejemplo, DC=fabrikam,DC=com |

El sistema proporciona el parámetro siguiente, que se usa para obtener el identificador del conector que se ejecuta actualmente:  
`Connector.ID`

Este es un ejemplo en el que se llena el dominio del atributo de metaverso con el nombre netbios del dominio en el que se encuentra el usuario:  
`domain` <- `%Domain.Netbios%`

### <a name="operators"></a>Operadores
Pueden utilizarse los siguientes operadores:

* **Comparación**: &lt;, &lt;=, &lt;&gt;, =, &gt;, &gt;=
* **Matemáticos**: +, -, \*, -
* **Cadena**: &amp; (concatenar)
* **Lógico**: &amp;&amp; (and), || (or)
* **Orden de evaluación**: ( )

Los operadores se evalúan de izquierda a derecha y tienen la misma prioridad de evaluación. Es decir, \* (multiplicador) no se evalúa antes que la resta (-). 2\*(5+3) no es lo mismo que 2\*5+3. Los paréntesis () se usan para cambiar el orden de evaluación cuando la evaluación de izquierda a derecha no es adecuada.

## <a name="multi-valued-attributes"></a>Atributos con varios valores
Las funciones pueden operar tanto atributos con un solo valor como con varios valores. Para los atributos con varios valores, la función opera sobre cada valor y aplica la misma función a cada valor.

Por ejemplo:  
`Trim([proxyAddresses])` Use la función Trim en todos los valores del atributo proxyAddress.  
`Word([proxyAddresses],1,"@") & "@contoso.com"` Para cada valor con un signo @-sign, reemplace el dominio por @contoso.com.  
`IIF(InStr([proxyAddresses],"SIP:")=1,NULL,[proxyAddresses])` Busque la dirección SIP y quítela de los valores.

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información sobre el modelo de configuración en el artículo de información sobre el [aprovisionamiento declarativo](concept-azure-ad-connect-sync-declarative-provisioning.md).
* Vea cómo se utiliza aprovisionamiento declarativo integrado en el artículo sobre la [configuración predeterminada](concept-azure-ad-connect-sync-default-configuration.md).
* Descubra cómo hacer un cambio práctico utilizando el aprovisionamiento declarativo en [Sincronización de Azure AD Connect: cómo realizar un cambio en la configuración predeterminada](how-to-connect-sync-change-the-configuration.md).

**Temas de introducción**

* [Sincronización de Azure AD Connect: comprender y personalizar la sincronización](how-to-connect-sync-whatis.md)
* [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md)

**Temas de referencia**

* [Azure AD Connect Sync: referencia de funciones](reference-connect-sync-functions-reference.md)