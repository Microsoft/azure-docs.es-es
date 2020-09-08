---
title: 'Azure Front Door: Compatibilidad con dominios con caracteres comodín'
description: Este artículo le ayudará a comprender cómo Azure Front Door admite la asignación y administración de dominios con caracteres comodín en la lista de dominios personalizados.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/10/2020
ms.author: duau
ms.openlocfilehash: 349c00b312ba7c3f18bab04fb059199732b4f1b3
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89398756"
---
# <a name="wildcard-domains"></a>Dominios con caracteres comodín

Aparte de los dominios y los subdominios de vértice, puede asignar un nombre de dominio con comodín a la lista de hosts de front-end o a los dominios personalizados del perfil de Front Door. La presencia de dominios con caracteres comodín en la configuración de Azure Front Door simplifica el comportamiento de enrutamiento del tráfico para varios subdominios de una API, aplicación o sitio web a partir de la misma regla de enrutamiento. No es necesario modificar la configuración para agregar o especificar cada subdominio por separado. Por ejemplo, puede definir el enrutamiento para `customer1.contoso.com`, `customer2.contoso.com` y `customerN.contoso.com` con la misma regla de enrutamiento y agregando el dominio con caracteres comodín `*.contoso.com`.

Entre los escenarios clave que se mejoran con la compatibilidad con los dominios con caracteres comodín se incluyen:

- No es necesario incorporar cada subdominio en el perfil de Azure Front Door y, después, habilitar HTTPS para enlazar un certificado para cada subdominio.
- Ya no es necesario cambiar la configuración de producción de Azure Front Door si una aplicación agrega un nuevo subdominio. Antes, se tenía que agregar el subdominio, enlazarle un certificado, asociar una directiva de firewall de aplicaciones web (WAF) y, a continuación, agregar el dominio a diferentes reglas de enrutamiento.

> [!NOTE]
> Actualmente, los dominios con caracteres comodín solo se admiten a través de la API, PowerShell y la CLI de Azure. La posibilidad de agregar y administrar dominios con caracteres comodín en Azure Portal no está disponible.

## <a name="adding-wildcard-domains"></a>Incorporación de dominios con caracteres comodín

Puede agregar un dominio con caracteres comodín en la sección de hosts de front-end o dominios. Al igual que con los subdominios, Azure Front Door valida que haya una asignación de registro CNAME para el dominio con caracteres comodín. Esta asignación DNS puede ser una asignación directa de registros CNAME como `*.contoso.com` asignado a `contoso.azurefd.net`. O bien, puede usar la asignación temporal afdverify. Por ejemplo, `afdverify.contoso.com` asignado a `afdverify.contoso.azurefd.net` valida la asignación de registro CNAME para el carácter comodín.

> [!NOTE]
> Azure DNS admite registros con caracteres comodín.

Puede agregar todos los subdominios de un solo nivel del dominio con caracteres comodín que quiera en los hosts de front-end, hasta el límite para los hosts de front-end. Esta funcionalidad podría ser necesaria para:

- Definir una ruta para un subdominio diferente del resto de los dominios (a partir del dominio comodín).

- Tener una directiva de WAF diferente para un subdominio específico. Por ejemplo, `*.contoso.com` permite agregar `foo.contoso.com` sin necesidad de volver a probar la propiedad del dominio. Pero no permite `foo.bar.contoso.com` porque no es un subdominio de nivel único de `*.contoso.com`. Para agregar `foo.bar.contoso.com` sin la validación adicional de la propiedad del dominio, se debe agregar `*.bar.contosonews.com`.

Puede agregar dominios con caracteres comodín y sus subdominios con ciertas limitaciones:

- Si se agrega un dominio con caracteres comodín a un perfil de Azure Front-Door:
  - El dominio con caracteres comodín no se puede agregar a un perfil de Azure Front-Door.
  - Los subdominios del primer nivel del dominio con caracteres comodín no se pueden agregar a otro perfil de Azure Front Door o a un perfil de Azure Content Delivery Network.
- Si se agrega un subdominio de un dominio con caracteres comodín a un perfil de Azure Front Door o de Azure Content Delivery Network, el dominio con caracteres comodín no se podrá agregar a ningún otro perfil de Azure Front Door.
- Si dos perfiles (de Azure Front Door o de Azure Content Delivery Network) tienen varios subdominios de un dominio raíz, no se pueden agregar dominios con caracteres comodín a ninguno de los perfiles.

## <a name="certificate-binding"></a>Enlace de certificado

Para aceptar el tráfico HTTPS en el dominio con caracteres comodín, debe habilitar HTTPS en ese dominio. El enlace de certificado para un dominio con caracteres comodín requiere un certificado comodín. Es decir, el nombre del sujeto del certificado también debe tener el dominio con caracteres comodín.

> [!NOTE]
> Actualmente, solo está disponible la opción de usar su propio certificado SSL personalizado para habilitar HTTPS para dominios con caracteres comodín. Los certificados administrados de Azure Front Door no se pueden usar para dominios con caracteres comodín.

También se puede usar el mismo certificado comodín de Azure Key Vault o los certificados administrados de Azure Front Door para los subdominios.

Si se agrega un subdominio para un dominio con caracteres comodín que ya tiene un certificado asociado, no se puede deshabilitar HTTPS para el subdominio. El subdominio usa el enlace de certificado para el dominio con caracteres comodín, a menos que otro certificado administrado de Key Vault o de Azure Front Door lo invalide.

## <a name="waf-policies"></a>Directivas de WAF

Las directivas de WAF se pueden adjuntar a dominios con caracteres comodín, de manera similar que a otros dominios. Se puede aplicar una directiva de WAF diferente a un subdominio de un dominio con caracteres comodín. En el caso de los subdominios, debe especificar la directiva de WAF que se va a usar, incluso si se trata de la misma directiva que para el dominio con caracteres comodín. Los subdominios *no* heredan automáticamente la directiva de WAF del dominio con caracteres comodín.

Si no quiere que una directiva de WAF se ejecute para un subdominio, puede crear una directiva de WAF vacía sin conjuntos de reglas personalizadas o administradas.

## <a name="routing-rules"></a>Reglas de enrutamiento

Al configurar una regla de enrutamiento, puede seleccionar un dominio con caracteres comodín como host de front-end. También puede disponer de un comportamiento de enrutamiento diferente para los dominios y subdominios con caracteres comodín. Como se describe en [Coincidencia de rutas con Azure Front Door](front-door-route-matching.md), se elige la coincidencia más específica para el dominio en diferentes reglas de enrutamiento del entorno de ejecución.

> [!IMPORTANT]
> Debe tener patrones de ruta de acceso coincidentes en las reglas de enrutamiento o los clientes recibirán errores. Por ejemplo, tiene dos reglas de enrutamiento, como la ruta 1 (`*.foo.com/*` asignada al grupo de back-end A) y la ruta 2 (`bar.foo.com/somePath/*` asignada al grupo de back-end B). A continuación, llega una solicitud para `bar.foo.com/anotherPath/*`. Azure Front Door selecciona la ruta 2 en función de una coincidencia de dominio más específica, solo para buscar patrones de ruta de acceso no coincidentes en las rutas.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear un perfil de Azure Front Door](quickstart-create-front-door.md).
- Aprenda a [agregar un dominio personalizado en Azure Front Door](front-door-custom-domain.md).
- Aprenda a [habilitar HTTPS en un dominio personalizado](front-door-custom-domain-https.md).
