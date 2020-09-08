---
title: Escalado y protección rápidos de una aplicación web mediante Azure Front Door y el firewall de aplicaciones web (WAF) de Azure | Microsoft Docs
description: Este artículo le ayuda a entender cómo usar el firewall de aplicaciones web con Azure Front Door Service.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/06/2020
ms.author: duau
ms.openlocfilehash: 7d2978b34d4c1e6cf85c65be2e9c3292ec704be4
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89398892"
---
# <a name="quickly-scale-and-protect-a-web-application-using-azure-front-door-and-azure-web-application-firewall-waf"></a>Escalado y protección rápidos de una aplicación web mediante Azure Front Door y el firewall de aplicaciones web (WAF) de Azure

En muchas aplicaciones web se ha producido un rápido aumento del tráfico en las últimas semanas con motivo de la COVID-19. Además, también se observa en ellas un aumento en el tráfico malintencionado, incluidos los ataques por denegación de servicio. Una manera eficaz de controlar a la vez la necesidad de escalado horizontal para las sobrecargas de tráfico y la protección ante los ataques consiste en configurar Azure Front Door con Azure WAF como un nivel de aceleración, almacenamiento en caché y seguridad por delante de la aplicación web. En este artículo se proporcionan instrucciones sobre cómo obtener rápidamente esta configuración de Azure Front Door con Azure WAF para cualquier aplicación web que se ejecute dentro o fuera de Azure. 

Usaremos la CLI de Azure para configurar WAF en este tutorial, pero todos estos pasos también son totalmente compatibles con Azure Portal, Azure PowerShell, Azure ARM y las API REST de Azure. 

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar. 

Las instrucciones de este blog usan la interfaz de la línea de comandos (CLI) de Azure. Consulte en esta guía una [introducción a la CLI de Azure](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest).

*Sugerencia: una forma sencilla y rápida de empezar a trabajar en la CLI de Azure es con [Bash en Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/quickstart).*

Asegúrese de que se agrega la extensión front-door a la CLI de Azure.

```azurecli-interactive 
az extension add --name front-door
```

Nota: Para obtener más detalles sobre los comandos que se enumeran a continuación, consulte la [referencia de la CLI de Azure para Front Door](https://docs.microsoft.com/cli/azure/ext/front-door/?view=azure-cli-latest).

## <a name="step-1-create-an-azure-front-door-afd-resource"></a>Paso 1: Creación de un recurso de Azure Front Door (AFD)


```azurecli-interactive 
az network front-door create --backend-address <>  --accepted-protocols <> --name <> --resource-group <>
```

**--backend-address**: la dirección de back-end es el nombre de dominio completo (FQDN) de la aplicación que desea proteger. Por ejemplo, miAplicación.contoso.com.

**--accepted-protocols**: los protocolos aceptados especifican los protocolos que desea que AFD admita para la aplicación web. Por ejemplo, --accepted-protocols Http Https.

**--name**: especifique un nombre para el recurso AFD.

**--resource-group**: el grupo de recursos en el que desea colocar este recurso AFD.  Para más información sobre los grupos de recursos, consulte el tema sobre administración de grupos de recursos en Azure.

En la respuesta que obtiene al ejecutar correctamente este comando, busque la clave "hostName" y anote el valor, ya que lo usará en un paso posterior. hostName es el nombre DNS del recurso AFD que creó.

## <a name="step-2-create-an-azure-waf-profile-to-use-with-azure-front-door-resources"></a>Paso 2: Creación de un perfil de Azure WAF para usarlo con recursos de Azure Front-Door

```azurecli-interactive 
az network front-door waf-policy create --name <>  --resource-group <>  --disabled false --mode Prevention
```

--name: especifique un nombre para la directiva de Azure WAF.

--resource-group: grupo de recursos en el que desea colocar este recurso WAF. 

El código anterior de la CLI creará una directiva WAF habilitada y en modo de prevención. 

Nota: es posible que también quiera crear el WAF en modo de detección y observar cómo detecta y registra solicitudes malintencionadas (y no las bloquea) antes de decidir cambiar al modo de protección.

En la respuesta que obtiene al ejecutar correctamente este comando, busque la clave "ID" y anote el valor, ya que lo usará en un paso posterior. El campo ID debe tener el siguiente formato:

/subscriptions/**Id. de la suscripción**/resourcegroups/**nombre del grupo de recursos**/providers/Microsoft.Network/frontdoorwebapplicationfirewallpolicies/**nombre de la directiva de WAF**

## <a name="step-3-add-managed-rulesets-to-this-waf-policy"></a>Paso 3: Incorporación de conjuntos de directivas administradas a esta directiva de WAF

En una directiva de WAF, puede agregar conjuntos de reglas administradas (es decir, un conjunto de reglas compiladas y administradas por Microsoft) y ofrecer protección integrada frente a grupos completos de amenazas. En este ejemplo, vamos a agregar dos de estos tipos de conjuntos de reglas: (1) un conjunto de reglas predeterminadas que protege contra las amenazas web comunes y (2) un conjunto de reglas de protección contra bots, que protege frente a bots malintencionados.

(1) Agregar el conjunto de reglas predeterminadas

```azurecli-interactive 
az network front-door waf-policy managed-rules add --policy-name <> --resource-group <> --type DefaultRuleSet --version 1.0
```

(2) Agregar el conjunto de reglas de administrador de bots

```azurecli-interactive 
az network front-door waf-policy managed-rules add --policy-name <> --resource-group <> --type Microsoft_BotManagerRuleSet --version 1.0
```

--policy-name: nombre que asignó al recurso WAF de Azure.

--resource-group: grupo de recursos en el que colocó este recurso WAF.

## <a name="step-4-associate-the-waf-policy-with-the-afd-resource"></a>Paso 4: Asociación de la directiva de WAF con el recurso AFD

En este paso, asociaremos la directiva de WAF que compilamos antes con el recurso AFD que está delante de la aplicación web.

```azurecli-interactive 
az network front-door update --name <> --resource-group <> --set frontendEndpoints[0].webApplicationFirewallPolicyLink='{"id":"<>"}'
```

--name: nombre que especificó para el recurso AFD.

--resource-group: grupo de recursos en el que colocó el recurso de Azure Front Door.

--set: aquí es donde actualiza el atributo WebApplicationFirewallPolicyLink para el valor de frontendEndpoint asociado con el recurso de AFD y le asigna la directiva de WAF recién creada. Puede encontrar el identificador de la directiva de WAF en la respuesta del paso 2 anterior.

Nota: el ejemplo anterior corresponde al caso en el que no usa un dominio personalizado.

Si no usa dominios personalizados para acceder a las aplicaciones web, puede omitir el paso 5. En ese caso, proporcionará a los usuarios finales el nombre de host que obtuvo en el paso 1 para ir a la aplicación web.

## <a name="step-5-configure-custom-domain-for-your-web-application"></a>Paso 5: Configuración del dominio personalizado para la aplicación web

Inicialmente, el nombre de dominio personalizado de la aplicación web (el que los clientes usan para hacer referencia a la aplicación; por ejemplo, www.contoso.com) apuntaba al lugar donde se ejecutaba antes de incluir AFD. Después de este cambio de arquitectura que supone agregar AFD+WAF delante de la aplicación, la entrada DNS correspondiente a ese dominio personalizado debe apuntar a este recurso AFD. Esto se puede hacer reasignando esta entrada del servidor DNS al nombre de host de AFD que anotó en el paso 1.

Los pasos específicos para actualizar los registros DNS dependerán del proveedor de servicios DNS, pero si usa Azure DNS para hospedar su nombre DNS, puede consultar la documentación sobre los [pasos para actualizar un registro DNS](https://docs.microsoft.com/azure/dns/dns-operations-recordsets-cli) y apuntar al valor de hostName de AFD. 

Una importante cuestión que hay que tener en cuenta es que, si necesita que los usuarios vayan al sitio web mediante el vértice de zona (por ejemplo, contoso.com), tiene que usar Azure DNS y su [tipo de registro de ALIAS](https://docs.microsoft.com/azure/dns/dns-alias) para hospedar el nombre DNS. 

Además, también debe actualizar la configuración de AFD para [agregarle este dominio personalizado](https://docs.microsoft.com/azure/frontdoor/front-door-custom-domain), de modo que AFD entienda esta asignación.

Por último, si usa un dominio personalizado para llegar a la aplicación web y desea habilitar el protocolo HTTPS, debe tener los [certificados para la configuración del dominio personalizado en AFD](https://docs.microsoft.com/azure/frontdoor/front-door-custom-domain-https). 

## <a name="step-6-lock-down-your-web-application"></a>Paso 6: Bloqueo de la aplicación web

Un procedimiento recomendado opcional que se puede seguir consiste en asegurarse de que solo los bordes de AFD puedan comunicarse con la aplicación web. Esta acción garantizará que nadie pueda omitir las protecciones de AFD y acceder a las aplicaciones directamente. Para lograr este bloqueo, visite la [sección de preguntas frecuentes de AFD](https://docs.microsoft.com/azure/frontdoor/front-door-faq) y consulte la pregunta sobre el modo de conseguir que AFD sea el único que tenga acceso al back-end.
