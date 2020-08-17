---
title: Protección con contraseña de Azure Active Directory
description: Obtenga información sobre cómo prohibir dinámicamente las contraseñas no seguras de su entorno con Protección con contraseña de Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/16/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6f0f7571cf9f8d355330c4acf425e38ce215e840
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87050877"
---
# <a name="eliminate-bad-passwords-using-azure-active-directory-password-protection"></a>Eliminación de contraseñas incorrectas mediante Protección con contraseña de Azure Active Directory

En general, las guías de seguridad recomiendan que no se use la misma contraseña en varios lugares, para que sea compleja y para evitar contraseñas sencillas, como *contraseña123*. Puede proporcionar a los usuarios una [guía sobre cómo elegir contraseñas](https://www.microsoft.com/research/publication/password-guidance), pero las contraseñas fáciles o no seguras a menudo se siguen usando. La característica Protección con contraseña de Azure AD detecta y bloquea las contraseñas no seguras conocidas y sus variantes, y también puede bloquear otros términos poco seguros específicos de una organización.

Con Protección con contraseña de Azure AD, las listas globales de contraseñas prohibidas se aplican automáticamente a todos los usuarios de la nube. Para satisfacer sus necesidades empresariales y de seguridad, puede definir entradas en una lista personalizada de contraseñas prohibidas. Cuando los usuarios cambian o restablecen sus contraseñas, se consultan estas listas de contraseñas prohibidas para exigir el uso de contraseñas seguras.

Debe usar características adicionales, como [Azure Multi-Factor Authentication](concept-mfa-howitworks.md), no solo confiar en las contraseñas seguras aplicadas por Protección con contraseña de Azure AD. Para obtener más información sobre el uso de varios niveles de seguridad para los eventos de inicio de sesión, consulte [Su Contra$eña no importa](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Your-Pa-word-doesn-t-matter/ba-p/731984).

> [!IMPORTANT]
> En este artículo teórico se explica al administrador cómo funciona Protección con contraseña de Azure AD. Si es un usuario final registrado para el autoservicio de restablecimiento de contraseña y necesita volver a su cuenta, visite [https://aka.ms/sspr](https://aka.ms/sspr).
>
> Si el equipo de TI no ha habilitado la capacidad para restablecer su propia contraseña, póngase en contacto con el departamento de soporte técnico para obtener ayuda adicional.

## <a name="global-banned-password-list"></a>Lista global de contraseñas prohibidas

El equipo de Azure AD Identity Protection analiza constantemente los datos de telemetría de seguridad de Azure AD en búsqueda de contraseñas poco seguras o en peligro. En concreto, el análisis busca los términos que a menudo se usan como base para las contraseñas no seguras. Cuando se encuentran términos débiles, se agregan a la *lista global de contraseñas prohibidas*. El contenido de la lista global de contraseñas prohibidas no se basa en ningún origen de datos externo, sino en los resultados de la telemetría y el análisis de seguridad de Azure AD.

Siempre que se cambia o se restablece una contraseña para cualquier usuario de cualquier inquilino de Azure AD, se usa la versión actual de la lista global de contraseñas prohibidas para validar la seguridad de la contraseña. Esta comprobación de la validación genera contraseñas más seguras para todos los clientes de Azure AD.

La lista global de contraseñas prohibidas se aplica automáticamente a todos los usuarios de la nube de un inquilino de Azure AD. No hay nada para habilitar ni configurar, y no se puede deshabilitar.

> [!NOTE]
> Los delincuentes cibernéticos también usan estrategias similares en sus ataques para identificar contraseñas no seguras comunes y sus variaciones. Para mejorar la seguridad, Microsoft no publica el contenido de la lista global de contraseñas prohibidas.

## <a name="custom-banned-password-list"></a>Lista personalizada de contraseñas prohibidas

Algunas organizaciones quieren mejorar la seguridad y agregan sus propias personalizaciones a la lista global de contraseñas prohibidas. Para agregar sus propias entradas, puede usar la *lista personalizada de contraseñas prohibidas*. Los términos agregados a la lista personalizada de contraseñas prohibidas deben dirigirse a términos específicos de la organización, como los ejemplos siguientes:

- Nombres de marca
- Nombres de producto
- Ubicaciones, por ejemplo, la oficina central de la empresa
- Términos internos específicos de la empresa
- Abreviaturas que tienen un significado específico en la empresa

Cuando se agreguen términos a la lista personalizada de contraseñas prohibidas, se combinan con los términos de la lista global de contraseñas prohibidas. Los eventos de cambio o restablecimiento de una contraseña se validan luego frente al conjunto combinado de estas listas de contraseñas prohibidas.

> [!NOTE]
> La lista personalizada de contraseñas prohibidas se limita a un máximo de 1000 términos. El diseño no permite bloquear listas muy grandes de contraseñas.
>
> Para aprovechar al máximo las ventajas de la lista personalizada de contraseñas prohibidas, primero comprenda [cómo se evalúan las contraseñas](#how-are-passwords-evaluated), para luego agregar términos a la lista personalizada de contraseñas prohibidas. Este enfoque le permite detectar y bloquear de manera eficaz un gran número de contraseñas no seguras y sus variantes.

![Modifique la lista personalizada de contraseñas prohibidas en Métodos de autenticación](./media/tutorial-configure-custom-password-protection/enable-configure-custom-banned-passwords-cropped.png)

Veamos un cliente denominado *Contoso*. La empresa tiene sede en Londres y crea un producto denominado *Widget*. Para este ejemplo de cliente, sería una pérdida de tiempo y menos seguro intentar bloquear variantes específicas de términos como los siguientes:

- "Contoso!1"
- "Contoso@London"
- "ContosoWidget"
- "!Contoso"
- "LondonHQ"

En su lugar, es mucho más eficaz y seguro bloquear solo los términos básicos de la clave, como los siguientes:

- "Contoso"
- "London"
- "Widget"

Luego, el algoritmo de validación de contraseñas bloqueará automáticamente las variantes y las combinaciones no seguras.

Para empezar a usar una lista personalizada de contraseñas prohibidas, complete el siguiente tutorial:

> [!div class="nextstepaction"]
> [Tutorial: Configuración de contraseñas prohibidas personalizadas](tutorial-configure-custom-password-protection.md)

## <a name="password-spray-attacks-and-third-party-compromised-password-lists"></a>Ataques de pulverización de contraseñas y listas de contraseñas en peligro de terceros

Protección con contraseña de Azure AD le ayuda a defenderse contra los ataques de difusión de contraseña. La mayoría de los ataques de difusión de contraseña no intentan atacar una cuenta individual determinada más de algunas veces. Este comportamiento aumentaría la probabilidad de detección, ya sea mediante el bloqueo de la cuenta u otros medios.

Por lo tanto, la mayoría de los ataques de difusión de contraseña se basan en el envío de un número pequeño de las contraseñas menos seguras conocidas en cada una de las cuentas de una empresa. Esta técnica permite al atacante buscar rápidamente una cuenta en peligro y evitar posibles umbrales de detección.

Protección con contraseña de Azure AD bloquea de forma eficaz todas las contraseñas no seguras conocidas que se puedan usar en los ataques de difusión de contraseña. Esta protección se basa en los datos de telemetría de seguridad del mundo real de Azure AD para crear la lista global de contraseñas prohibidas.

Existen sitios web de terceros que indican millones de contraseñas que se han puesto en peligro a través de vulneraciones de seguridad de dominio público. Es habitual que los productos de validación de contraseñas de terceros se basen en la comparación de fuerza bruta con esos millones de contraseñas. Sin embargo, estas técnicas no son la mejor manera de mejorar la seguridad general de las contraseñas dadas las estrategias típicas que utilizan los atacantes de difusión de contraseña.

> [!NOTE]
> La lista global de contraseñas prohibidas no se basa en ningún origen de datos de terceros, incluidas las listas de contraseñas en peligro.

Aunque la lista global prohibida es pequeña en comparación con algunas listas masivas de terceros, se ha obtenido de los datos de telemetría de seguridad del mundo real relativos a los ataques de difusión de contraseña reales. Este enfoque mejora la seguridad y la eficacia generales, y el algoritmo de validación de contraseñas también utiliza técnicas de coincidencia aproximada inteligente. Como consecuencia, Protección con contraseña de Azure AD El detecta y bloquea de manera eficaz millones de las contraseñas poco seguras más comunes que se usan en su empresa.

## <a name="on-premises-hybrid-scenarios"></a>Escenarios híbridos locales

Muchas organizaciones tienen un modelo de identidad híbrido que incluye entornos locales de Active Directory Domain Services (AD DS). Para ampliar las ventajas de seguridad de Protección con contraseña de Azure AD al entorno de AD DS, puede instalar los componentes en los servidores locales. Estos agentes requieren eventos de cambio de contraseña en el entorno de AD DS local para cumplir con la misma directiva de contraseñas que los usuarios que solo están en la nube.

Para obtener más información, consulte [Aplicación de Protección con contraseña de Azure AD en AD DS](concept-password-ban-bad-on-premises.md).

## <a name="how-are-passwords-evaluated"></a>Cómo se evalúan las contraseñas

Cuando un usuario cambia o restablece su contraseña, se comprueba la solidez y la complejidad de la nueva contraseña validándola con la combinación de la lista global y la lista personalizada de contraseñas prohibidas.

Incluso si una contraseña de usuario contiene una contraseña prohibida, la contraseña podría aceptarse si la contraseña global es lo suficientemente segura. Una contraseña recién configurada atraviesa los pasos siguientes para evaluar su solidez general a fin de determinar si debe aceptarse o rechazarse.

### <a name="step-1-normalization"></a>Paso 1: Normalización

En primer lugar, una nueva contraseña pasa por un proceso de normalización. Esta técnica permite que un pequeño conjunto de contraseñas prohibidas se asigne a un conjunto mucho más grande de contraseñas potencialmente inseguras.

La normalización tiene las dos partes siguientes:

* Todas las letras mayúsculas se cambian a minúsculas.
* Luego, se realizan sustituciones de caracteres comunes, como en el ejemplo siguiente:

   | Letra original | Letra sustituida |
   |-----------------|--------------------|
   | 0               | o                  |
   | 1               | l                  |
   | $               | s                  |
   | \@              | a                  |

Considere el ejemplo siguiente:

* La contraseña "blank" está prohibida.
* Un usuario intenta cambiar la contraseña a "Bl@nK".
* Aunque "Bl@nk" no está prohibida, el proceso de normalización convierte esta contraseña en "blank".
* Esta contraseña se rechazaría.

### <a name="step-2-check-if-password-is-considered-banned"></a>Paso 2: Comprobar si la contraseña se considera prohibida

A continuación, se examina una contraseña para buscar otro comportamiento coincidente y se genera una puntuación. Esta puntuación final determina si se acepta o rechaza la solicitud de cambio de contraseña.

#### <a name="fuzzy-matching-behavior"></a>Comportamiento de la coincidencia aproximada

La coincidencia aproximada se usa en la contraseña normalizada para identificar si contiene una contraseña que se encuentra en la lista global o personalizada de contraseñas prohibidas. El proceso de búsqueda de coincidencias se basa en una distancia de edición de una (1) comparación.

Considere el ejemplo siguiente:

* La contraseña "abcdef" está prohibida.
* Un usuario intenta cambiar la contraseña a una de las siguientes:

   * "abcdeg": *el último carácter cambió de "f" a "g"*
   * "abcdefg": *se anexó "g" al final*
   * "abcde": *la "f" final se ha eliminado*

* Ninguna de las contraseñas anteriores coincide específicamente con la contraseña prohibida "abcdef".

    Sin embargo, dado que cada ejemplo se encuentra dentro de una distancia de 1 edición del término prohibido "abcdef", se considera que todas coinciden con "abcdef".
* Estas contraseñas se rechazarían.

#### <a name="substring-matching-on-specific-terms"></a>Coincidencia de subcadenas (en términos específicos)

La coincidencia de subcadenas se usa en la contraseña normalizada para comprobar el nombre y el apellido del usuario, así como el nombre del inquilino. La coincidencia de nombres de inquilino no se realiza al validar contraseñas en un controlador de dominio de AD DS para escenarios híbridos locales.

> [!IMPORTANT]
> La coincidencia de subcadenas solo se aplica a los nombres y otros términos que tienen al menos cuatro caracteres de longitud.

Considere el ejemplo siguiente:

* Un usuario llamado Poll que desea restablecer su contraseña a "p0LL23fb".
* Después de la normalización, esta contraseña se convertiría en "poll23fb".
* La coincidencia de subcadenas detecta que la contraseña contiene el nombre del usuario "Pol".
* Aunque "poll23fb" no se encontraba específicamente en ninguna lista de contraseñas prohibidas, la coincidencia de subcadenas encontró "Pol" en la contraseña.
* Esta contraseña se rechazaría.

#### <a name="score-calculation"></a>Cálculo de puntuación

El siguiente paso consiste en identificar todas las instancias de contraseñas prohibidas en la nueva contraseña normalizada del usuario. Se asignan puntos según los siguientes criterios:

1. A cada contraseña prohibida que se encuentre en la contraseña de un usuario se le concede un punto.
1. Cada carácter único restante recibe un punto.
1. Una contraseña debe tener al menos cinco (5) puntos para que se acepte.

En los dos escenarios de ejemplo siguientes, Contoso usa Protección con contraseñas de Azure AD y tiene "contoso" en la lista personalizada de contraseñas prohibidas. Supongamos también que "blank" está en la lista global.

En el siguiente escenario de ejemplo, un usuario cambia su contraseña a "C0ntos0Blank12".

* Después de la normalización, esta contraseña se convierte en "contosoblank12".
* El proceso de coincidencia encuentra que esta contraseña contiene dos contraseñas prohibidas: contoso y blank.
* La puntuación de esta contraseña entonces sería la siguiente:

    *[contoso] + [blank] + [1] + [2] = 4 puntos*

* Dado que esta contraseña tiene menos de cinco (5) puntos, se rechaza.

Echemos un vistazo a un ejemplo ligeramente diferente para mostrar cómo la complejidad adicional en una contraseña puede acumular el número necesario de puntos para que se acepte. En el siguiente escenario de ejemplo, un usuario cambia su contraseña a "ContoS0Bl@nkf9!".

* Después de la normalización, esta contraseña se convierte en "contosoblankf9!".
* El proceso de coincidencia encuentra que esta contraseña contiene dos contraseñas prohibidas: contoso y blank.
* La puntuación de esta contraseña entonces sería la siguiente:

    *[contoso] + [blank] + [f] + [9] + [!] = 5 puntos*

* Dado que esta contraseña tiene al menos cinco (5) puntos, se acepta.

> [!IMPORTANT]
> El algoritmo de contraseñas prohibidas, junto con la lista global de contraseñas prohibidas, puede cambiar (y de hecho cambia) en cualquier momento en Azure en función de la investigación y el análisis de seguridad en curso.
>
> En el caso del servicio de agente del controlador de dominio local en escenarios híbridos, los algoritmos actualizados solo surten efecto después de volver a instalar el software del agente del controlador de dominio.

## <a name="what-do-users-see"></a>Lo que ven los usuarios

Si un usuario intenta restablecer una contraseña a algo que estaría prohibido, se mostrará el siguiente mensaje de error:

*"La contraseña contiene una palabra, una frase o un patrón que resultan fáciles de adivinar. Vuelva a intentarlo con otra contraseña".*

## <a name="license-requirements"></a>Requisitos de licencia

| Usuarios | Protección con contraseña de Azure AD con la lista global de contraseñas prohibidas | Protección con contraseña de Azure AD con la lista personalizada de contraseñas prohibidas|
|-------------------------------------------|---------------------------|---------------------------|
| Usuarios solo en la nube                          | Azure AD Free             | Azure AD Premium (P1 o P2) |
| Usuarios sincronizados desde AD DS local | Azure AD Premium (P1 o P2) | Azure AD Premium (P1 o P2) |

> [!NOTE]
> Los usuarios de AD DS local que no se sincronizan con Azure AD también se benefician de Protección con contraseña de Azure AD según las licencias existentes para los usuarios sincronizados.

Puede encontrar información adicional sobre licencias, incluidos los costos, en la [página de precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="next-steps"></a>Pasos siguientes

Para empezar a usar una lista personalizada de contraseñas prohibidas, complete el siguiente tutorial:

> [!div class="nextstepaction"]
> [Tutorial: Configuración de contraseñas prohibidas personalizadas](tutorial-configure-custom-password-protection.md)

Además, también puede [habilitar Protección con contraseña de Azure AD local](howto-password-ban-bad-on-premises-deploy.md).
