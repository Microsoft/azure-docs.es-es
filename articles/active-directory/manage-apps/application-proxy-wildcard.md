---
title: Aplicaciones con comodín en Azure Active Directory Application Proxy | Microsoft Docs
description: Aprenda a usar aplicaciones con comodín en el proxy de aplicación de Azure Active Directory.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/06/2018
ms.author: mimart
ms.reviewer: harshja
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: fc82f69c8dee4cc8c45e9fcf7fbf6fa184ad72b6
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65783053"
---
# <a name="wildcard-applications-in-the-azure-active-directory-application-proxy"></a>Aplicaciones con comodín en Azure Active Directory Application Proxy 

En Azure Active Directory (Azure AD), la configuración de un gran número de aplicaciones locales puede rápidamente dificultar la administración e incorpora riesgos innecesarios de errores de configuración si muchas de ellas requieren la misma configuración. Con el [proxy de aplicación de Azure AD](application-proxy.md), puede resolver este problema con aplicaciones con comodín para publicar y administrar muchas aplicaciones a la vez. Se trata de una solución que permite:

-   Simplificar la sobrecarga administrativa
-   Reducir el número de posibles errores de configuración
-   Permitir a los usuarios acceder con seguridad a más recursos

En este artículo se proporciona la información necesaria para configurar la publicación de la aplicación con comodín en el entorno.


## <a name="create-a-wildcard-application"></a>Creación de una aplicación con comodín 

Puede crear una aplicación con comodín (*) si tiene un grupo de aplicaciones con la misma configuración. Los posibles candidatos a aplicación con comodín son aquellas que comparten la configuración siguiente:

- Grupo de usuarios que pueden acceder a ellas
- Método de inicio de sesión único
- Protocolo de acceso (http, https)

Puede publicar aplicaciones con caracteres comodín si tanto las direcciones URL internas como las externas tienen el siguiente formato:

> http(s)://*.\<domain\>

Por ejemplo: `http(s)://*.adventure-works.com`. Aunque las direcciones URL internas y externas pueden usar dominios diferentes, como procedimiento recomendado, deben ser iguales. Al publicar la aplicación, verá un error si una de las direcciones URL no tiene un carácter comodín.

Si tiene otras aplicaciones con distintos valores de configuración, debe publicar estas excepciones como aplicaciones independientes para sobrescribir los valores predeterminados establecidos para el comodín. Las aplicaciones sin caracteres comodín siempre tienen prioridad sobre las que los tienen. Desde la perspectiva de la configuración, se trata "simplemente" de aplicaciones normales.

La creación de una aplicación comodín se basa en el mismo [flujo de publicación de aplicaciones](application-proxy-add-on-premises-application.md) que está disponible para las demás aplicaciones. La única diferencia es que se incluye un carácter comodín en las direcciones URL y, quizá, la configuración del inicio de sesión único.


## <a name="prerequisites"></a>Requisitos previos

### <a name="custom-domains"></a>Dominios personalizados

Mientras que los[dominios personalizados](application-proxy-configure-custom-domain.md) son opcionales para todas las demás aplicaciones, son un requisito previo para las aplicaciones con comodín. La creación de dominios personalizados requiere:

1. La creación de un dominio comprobado en Azure 
2. La carga de un certificado SSL con formato PFX para el proxy de aplicación.

Considere la posibilidad de usar un certificado con comodín para que coincida con la aplicación que va a crear. Como alternativa, también puede utilizar un certificado que solo incluya aplicaciones específicas. En este caso, mediante esta aplicación con comodín solo se podrá acceder a las aplicaciones incluidas en el certificado.

Por motivos de seguridad, se trata de un requisito de disco duro y no se admiten caracteres comodín para las aplicaciones que no puedan utilizar un dominio personalizado para la dirección URL externa.

### <a name="dns-updates"></a>Actualizaciones del servicio de nombres de dominio

Al usar dominios personalizados, debe crear una entrada DNS con un registro CNAME para la dirección URL externa (por ejemplo, `*.adventure-works.com`) que apunte a la dirección URL externa del punto de conexión del proxy de la aplicación. Para las aplicaciones con comodín, este registro debe apuntar a las direcciones URL externas pertinentes:

> `<yourAADTenantId>.tenant.runtime.msappproxy.net`

Para confirmar que ha configurado CNAME correctamente, puede usar [nslookup](https://docs.microsoft.com/windows-server/administration/windows-commands/nslookup) en uno de los puntos de conexión de destino, por ejemplo, `expenses.adventure-works.com`.  La respuesta debe incluir el alias ya mencionado (`<yourAADTenantId>.tenant.runtime.msappproxy.net`).


## <a name="considerations"></a>Consideraciones


### <a name="accepted-formats"></a>Formatos aceptados

Para las aplicaciones con comodín, la **URL interna** debe tener el formato `http(s)://*.<domain>`. 

![AppId](./media/application-proxy-wildcard/22.png)


Al configurar una **dirección URL externa**, debe utilizar el formato siguiente: `https://*.<custom domain>` 

![AppId](./media/application-proxy-wildcard/21.png)

No se admiten otras posiciones de comodín, varios comodines ni otras cadenas de expresiones regulares, que provocan errores.


### <a name="excluding-applications-from-the-wildcard"></a>Exclusión de aplicaciones del comodín

Para excluir una aplicación de la aplicación con comodín:

- Publique la excepción de aplicación como aplicación normal 
- Habilite el comodín solo para aplicaciones específicas mediante la configuración del servicio de nombres de dominio  


La publicación de una aplicación como aplicación normal es el método preferido para excluir una aplicación de un comodín. Debe publicar las aplicaciones excluidas antes que las aplicaciones con comodín para asegurarse de que las excepciones se aplican desde el principio. La aplicación más específica siempre tiene prioridad: una aplicación que se publica como `budgets.finance.adventure-works.com` tiene prioridad sobre `*.finance.adventure-works.com`, que a su vez tiene prioridad sobre `*.adventure-works.com`. 

También puede limitar el carácter comodín para que funcione solo para aplicaciones específicas mediante la administración del servicio de nombres de dominio. Como procedimiento recomendado, debe crear una entrada CNAME que incluya un carácter comodín y coincida con el formato de la dirección URL externa configurada. Sin embargo, en su lugar puede hacer que las direcciones URL de aplicación específica apunten a los caracteres comodín. Por ejemplo, en lugar de `*.adventure-works.com`, haga que `hr.adventure-works.com`, `expenses.adventure-works.com` y `travel.adventure-works.com individually` apunten a `000aa000-11b1-2ccc-d333-4444eee4444e.tenant.runtime.msappproxy.net`. 

Si utiliza esta opción, necesitará otra entrada CNAME para el valor `AppId.domain`, por ejemplo, `00000000-1a11-22b2-c333-444d4d4dd444.adventure-works.com`, que apunte a la misma ubicación. **AppId** se encuentra en la página de propiedades de la aplicación de la aplicación con comodín:

![AppId](./media/application-proxy-wildcard/01.png)



### <a name="setting-the-homepage-url-for-the-myapps-panel"></a>Configuración de la dirección URL de página de inicio para el panel MyApps

La aplicación con comodín se representa con un solo icono en el [panel MyApps](https://myapps.microsoft.com). De forma predeterminada, este icono está oculto. Para mostrar el icono y que los usuarios vayan a una página específica:

1. Siga las directrices para [configurar una dirección URL de página principal](application-proxy-configure-custom-home-page.md).
2. Establezca **Show Application** en **true** en la página de propiedades de la aplicación.

### <a name="kerberos-constrained-delegation"></a>Delegación limitada de Kerberos

Para las aplicaciones que usan la [delegación restringida de Kerberos como método de inicio de sesión único](application-proxy-configure-single-sign-on-with-kcd.md), puede que el nombre de entidad de seguridad de servicio que aparece para el método de inicio de sesión único también necesite un carácter comodín. Por ejemplo, el nombre de entidad de seguridad de servicio podría ser: `HTTP/*.adventure-works.com`. Debe tener los nombres de entidad de seguridad de servicio individuales configurados en los servidores back-end (por ejemplo, `http://expenses.adventure-works.com and HTTP/travel.adventure-works.com`).



## <a name="scenario-1-general-wildcard-application"></a>Escenario 1: aplicación con comodín general

En este escenario, tiene tres aplicaciones que desea publicar:

- `expenses.adventure-works.com`
- `hr.adventure-works.com`
- `travel.adventure-works.com`

Las tres aplicaciones:

- Las utilizan todos los usuarios
- Usan la *autenticación integrada de Windows* 
- Tienen las mismas propiedades


Puede publicar la aplicación con comodín al seguir los pasos descritos en [Publicación de aplicaciones mediante el proxy de aplicación de Azure AD](application-proxy-add-on-premises-application.md). En este escenario se presupone que:

- Hay un inquilino con el identificador: `000aa000-11b1-2ccc-d333-4444eee4444e`. 

- Se ha configurado un dominio comprobado denominado `adventure-works.com`.

- Se ha creado una entrada **CNAME** que hace que `*.adventure-works.com` apunte a `000aa000-11b1-2ccc-d333-4444eee4444e.tenant.runtime.msappproxy.net`.

Al seguir los [pasos documentados](application-proxy-add-on-premises-application.md), se crea una aplicación de proxy de aplicación en el inquilino. En este ejemplo, el carácter comodín se encuentra en los campos siguientes:

- Dirección URL interna:

    ![Dirección URL interna](./media/application-proxy-wildcard/42.png)


- Dirección URL externa:

    ![Dirección URL externa](./media/application-proxy-wildcard/43.png)

 
- Nombre de entidad de seguridad de servicio de la aplicación interno: 

    ![Configuración del nombre de entidad de seguridad de servicio](./media/application-proxy-wildcard/44.png)


Al publicar la aplicación con comodín, ahora tendrá acceso a las tres aplicaciones; para ello, vaya a las direcciones URL habituales (por ejemplo, `travel.adventure-works.com`).

La configuración implementa la siguiente estructura:

![AppId](./media/application-proxy-wildcard/05.png)

| Color | DESCRIPCIÓN |
| ---   | ---         |
| Azul  | Aplicaciones publicadas explícitamente y visibles en Azure Portal. |
| Gris  | Aplicaciones accesibles desde la aplicación primaria. |




## <a name="scenario-2-general-wildcard-application-with-exception"></a>Escenario 2: aplicación con comodín general con excepción

En este escenario, además de las tres aplicaciones generales tiene otra, `finance.adventure-works.com`, que solo debe ser accesible para el departamento financiero. Con la estructura de aplicación actual, la aplicación financiera sería accesible mediante la aplicación con comodín y para todos los empleados. Para cambiar esto, excluya la aplicación de la aplicación con comodín; para ello, configure la financiera como aplicación independiente con permisos más restrictivos.



Tiene que asegurarse de que existen registros CNAME que hacen que `finance.adventure-works.com` apunte a un punto de conexión específico (que se especifica en la página del proxy de aplicación de esta). En este escenario, `finance.adventure-works.com`apunta a `https://finance-awcycles.msappproxy.net/`. 

Al seguir los [pasos documentados](application-proxy-add-on-premises-application.md), este escenario requiere la siguiente configuración:


- En la **dirección URL interna**, establezca **finance** en lugar de un carácter comodín. 

    ![Dirección URL interna](./media/application-proxy-wildcard/52.png)

- En la **dirección URL externa**, establezca **finance** en lugar de un carácter comodín. 

    ![Dirección URL externa](./media/application-proxy-wildcard/53.png)

- En el nombre de entidad de seguridad de servicio interno de la aplicación, establezca **finance** en lugar de un carácter comodín.

    ![Configuración del nombre de entidad de seguridad de servicio](./media/application-proxy-wildcard/54.png)


La configuración implementa el siguiente escenario:

![Escenario](./media/application-proxy-wildcard/09.png)

Dado que `finance.adventure-works.com` es una dirección URL más específica que `*.adventure-works.com`, tiene prioridad. Los usuarios que visiten `finance.adventure-works.com` tiene la experiencia que se especifica en la aplicación Finance Resources. En este caso, solo los empleados del departamento financiero tienen acceso a `finance.adventure-works.com`.

Si tiene varias aplicaciones financieras publicadas y `finance.adventure-works.com` como dominio comprobado, puede publicar otra aplicación con comodín `*.finance.adventure-works.com`. Dado que esta es más específico que la dirección `*.adventure-works.com` genérica, tiene prioridad si un usuario accede a una aplicación del dominio financiero.


## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de:

- Los **dominios personalizados**,consulte [Uso de dominios personalizados en el proxy de la aplicación de Azure AD](application-proxy-configure-custom-domain.md).

- La **publicación de aplicaciones**, consulte [Publicación de aplicaciones mediante el proxy de aplicación de Azure AD](application-proxy-add-on-premises-application.md).


