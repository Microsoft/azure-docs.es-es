---
title: P+F de API Management de Azure | Microsoft Docs
description: Conozca las respuestas a las preguntas más frecuentes (P+F), los patrones y los procedimientos recomendados en Azure API Management.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 2fa193cd-ea71-4b33-a5ca-1f55e5351e23
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/19/2017
ms.author: apimpm
ms.openlocfilehash: c49e92cda89cfc1d72a0550c2a53430f3e6f2844
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87050333"
---
# <a name="azure-api-management-faqs"></a>P+F de Azure API Management
Obtenga respuestas a preguntas comunes, patrones y procedimientos recomendados para Azure API Management.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes
* [¿Qué significa que una característica se encuentra en su versión preliminar?](#what-does-it-mean-when-a-feature-is-in-preview)
* [¿Cómo se puede proteger la conexión entre la puerta de enlace de API Management y mis servicios back-end?](#how-can-i-secure-the-connection-between-the-api-management-gateway-and-my-back-end-services)
* [¿Cómo se puede copiar mi instancia de servicio de API Management en una nueva instancia?](#how-do-i-copy-my-api-management-service-instance-to-a-new-instance)
* [¿Es posible administrar mi instancia de API Management mediante programación?](#can-i-manage-my-api-management-instance-programmatically)
* [¿Cómo se puede agregar un usuario al grupo de administradores?](#how-do-i-add-a-user-to-the-administrators-group)
* [¿Por qué la directiva que deseo agregar no está habilitada en el editor de directivas?](#why-is-the-policy-that-i-want-to-add-unavailable-in-the-policy-editor)
* [¿Cómo se configuran varios entornos en una sola API?](#how-do-i-set-up-multiple-environments-in-a-single-api)
* [¿Se puede usar SOAP con API Management?](#can-i-use-soap-with-api-management)
* [¿Se puede configurar un servidor de autorización de OAuth 2.0 con seguridad AD FS?](#can-i-configure-an-oauth-20-authorization-server-with-ad-fs-security)
* [¿Qué método de enrutamiento utiliza API Management en implementaciones en varias ubicaciones geográficas?](#what-routing-method-does-api-management-use-in-deployments-to-multiple-geographic-locations)
* [¿Se puede usar una plantilla de Azure Resource Manager para crear una instancia del servicio API Management?](#can-i-use-an-azure-resource-manager-template-to-create-an-api-management-service-instance)
* [¿Se puede usar un certificado TLS/SSL autofirmado para un back-end?](#can-i-use-a-self-signed-tlsssl-certificate-for-a-back-end)
* [¿Por qué se obtiene un error de autenticación al intentar clonar un repositorio?](#why-do-i-get-an-authentication-failure-when-i-try-to-clone-a-git-repository)
* [¿Funciona API Management con Azure ExpressRoute?](#does-api-management-work-with-azure-expressroute)
* [¿Por qué es necesaria una subred dedicada en las redes virtuales de estilo Resource Manager cuando API Management está implementado en ellas?](#why-do-we-require-a-dedicated-subnet-in-resource-manager-style-vnets-when-api-management-is-deployed-into-them)
* [¿Cuál es el tamaño de subred mínimo necesario al implementar API Management en una red virtual?](#what-is-the-minimum-subnet-size-needed-when-deploying-api-management-into-a-vnet)
* [¿Se puede mover un servicio API Management de una suscripción a otra?](#can-i-move-an-api-management-service-from-one-subscription-to-another)
* [¿Existen restricciones de la importación de mi API o problemas conocidos con ella?](#are-there-restrictions-on-or-known-issues-with-importing-my-api)

### <a name="what-does-it-mean-when-a-feature-is-in-preview"></a>¿Qué significa que una característica se encuentra en su versión preliminar?
Cuando una característica está en su versión preliminar, significa que estamos buscando activamente comentarios acerca de cómo funciona la característica. Una característica en versión preliminar está funcionalmente completa, pero es posible que hagamos cambios importantes en respuesta a los comentarios de los clientes. Se recomienda no depender de una característica que está en su versión preliminar en el entorno de producción.

### <a name="how-can-i-secure-the-connection-between-the-api-management-gateway-and-my-back-end-services"></a>¿Cómo se puede proteger la conexión entre la puerta de enlace de API Management y mis servicios back-end?
Tiene varias opciones para proteger la conexión entre la puerta de enlace de API Management y mis servicios back-end. Puede:

* Use la autenticación básica HTTP. Para más información, consulte [Importación y publicación de la primera API](import-and-publish.md).
* Use la autenticación mutua de TLS como se describe en [Cómo asegurar servicios back-end con la autenticación de certificados de cliente en Azure API Management](api-management-howto-mutual-certificates.md).
* Utilice la lista blanca IP en su servicio back-end. En todos los niveles de API Management, a excepción del nivel de consumo, la dirección IP de la puerta de enlace permanece constante, con algunas salvedades que se describen en [el artículo sobre la documentación de IP](api-management-howto-ip-addresses.md).
* Conecte la instancia de API Management a Azure Virtual Network.

### <a name="how-do-i-copy-my-api-management-service-instance-to-a-new-instance"></a>¿Cómo se puede copiar mi instancia de servicio de API Management en una nueva instancia?
Tiene varias opciones si desea copiar una instancia de API Management a una nueva instancia. Puede:

* Usar la función de copia de seguridad y restauración de API Management. Para más información, consulte [Procedimiento para implementar la recuperación ante desastres mediante copias de seguridad y restauración del servicio en Azure API Management](api-management-howto-disaster-recovery-backup-restore.md).
* Crear su propia característica de copia de seguridad y restauración mediante la [API de REST de API Management](/rest/api/apimanagement/). Usar la API de REST para guardar y restaurar las entidades de la instancia de servicio que desee.
* Descargar la configuración del servicio mediante Git y cargarla en una nueva instancia. Para más información, consulte [Guardado y configuración del servicio API Management mediante Git](api-management-configuration-repository-git.md).

### <a name="can-i-manage-my-api-management-instance-programmatically"></a>¿Es posible administrar mi instancia de API Management mediante programación?
Sí, puede administrar API Management mediante programación utilizando:

* La [API de REST de API Management](/rest/api/apimanagement/).
* El [SDK de la biblioteca de administración del servicio Microsoft Azure ApiManagement](https://aka.ms/apimsdk).
* Los cmdlets de [implementación del servicio](/powershell/module/wds) y [administración del servicio](/powershell/azure/servicemanagement/overview) de PowerShell.

### <a name="how-do-i-add-a-user-to-the-administrators-group"></a>¿Cómo se puede agregar un usuario al grupo de administradores?
A continuación se indica cómo agregar un usuario al grupo de administradores:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Vaya al grupo de recursos que tiene la instancia de API Management que desea actualizar.
3. En API Management, asigne el rol **Colaborador de Service API Management**.

Ahora el colaborador recién agregado puede usar los [cmdlets](/powershell/azure/) de Azure PowerShell. A continuación se indica cómo iniciar sesión como administrador:

1. Utilice el cmdlet `Connect-AzAccount` para iniciar sesión.
2. Establezca el contexto para la suscripción que tiene el servicio mediante `Set-AzContext -SubscriptionID <subscriptionGUID>`.
3. Obtenga la dirección URL de inicio de sesión único mediante `Get-AzApiManagementSsoToken -ResourceGroupName <rgName> -Name <serviceName>`.
4. Utilice la dirección URL para acceder al portal de administración.

### <a name="why-is-the-policy-that-i-want-to-add-unavailable-in-the-policy-editor"></a>¿Por qué la directiva que deseo agregar no está habilitada en el editor de directivas?
Si la directiva que desea agregar aparece atenuada o sombreada en el editor de directivas, asegúrese de que está en el ámbito correcto para la directiva. Cada instrucción de la directiva está diseñada para su uso en determinados ámbitos y secciones de la directiva. Para revisar las secciones y los ámbitos de una directiva, consulte en la sección sobre el uso de la directiva en [API Management policies](./api-management-policies.md) (Directivas de API Management).

### <a name="how-do-i-set-up-multiple-environments-in-a-single-api"></a>¿Cómo se configuran varios entornos en una sola API?
Para configurar varios entornos; por ejemplo, un entorno de prueba y un entorno de producción, en una sola API, tiene dos opciones. Puede:

* Hospedar diferentes API en el mismo inquilino.
* Hospedar las mismas API en diferentes inquilinos.

### <a name="can-i-use-soap-with-api-management"></a>¿Se puede usar SOAP con API Management?
Ahora se admite el [paso a través de SOAP](https://azure.microsoft.com/blog/soap-pass-through/). Los administradores pueden importar el WSDL de su servicio SOAP, y Azure API Management creará un front-end SOAP. Ahora hay documentación del portal para desarrolladores, la consola de prueba, las directivas y el análisis disponible para los servicios SOAP.

### <a name="can-i-configure-an-oauth-20-authorization-server-with-ad-fs-security"></a>¿Se puede configurar un servidor de autorización de OAUth 2.0 con seguridad AD FS?
Para más información sobre cómo configurar un servidor de autorización de OAuth 2.0 con la seguridad de Servicios de federación de Active Directory (AD FS), consulte [Using ADFS in API Management](https://phvbaars.wordpress.com/2016/02/06/using-adfs-in-api-management/) (Uso de ADFS en API Management).

### <a name="what-routing-method-does-api-management-use-in-deployments-to-multiple-geographic-locations"></a>¿Qué método de enrutamiento utiliza API Management en implementaciones en varias ubicaciones geográficas?
API Management usa el [método de enrutamiento de tráfico de rendimiento](../traffic-manager/traffic-manager-routing-methods.md#performance) en las implementaciones en varias ubicaciones geográficas. El tráfico entrante se enrutará a la puerta de enlace de API más cercana. Si una región se queda sin conexión, el tráfico entrante se enruta automáticamente a la siguiente puerta de enlace más cercana. Aprenda más acerca de los métodos de enrutamiento en [Métodos de enrutamiento de tráfico de Traffic Manager](../traffic-manager/traffic-manager-routing-methods.md).

### <a name="can-i-use-an-azure-resource-manager-template-to-create-an-api-management-service-instance"></a>¿Se puede usar una plantilla de Azure Resource Manager para crear una instancia del servicio API Management?
Sí. Consulte las plantillas de inicio rápido del [servicio Azure API Management](https://aka.ms/apimtemplate).

### <a name="can-i-use-a-self-signed-tlsssl-certificate-for-a-back-end"></a>¿Se puede usar un certificado TLS/SSL autofirmado para un back-end?
Sí. Puede hacerse a través de PowerShell o enviando el certificado directamente a la API. Esta operación deshabilitará la validación de la cadena de certificados y le permitirá usar certificados autofirmados o firmados de forma privada cuando se comunique con los servicios back-end desde API Management.

#### <a name="powershell-method"></a>Método de Powershell ####
Utilice el cmdlet de PowerShell [`New-AzApiManagementBackend`](/powershell/module/az.apimanagement/new-azapimanagementbackend) (con el nuevo back-end) o [`Set-AzApiManagementBackend`](/powershell/module/az.apimanagement/set-azapimanagementbackend) (con el back-end existente) y establezca el parámetro `-SkipCertificateChainValidation` en `True`.

```powershell
$context = New-AzApiManagementContext -resourcegroup 'ContosoResourceGroup' -servicename 'ContosoAPIMService'
New-AzApiManagementBackend -Context  $context -Url 'https://contoso.com/myapi' -Protocol http -SkipCertificateChainValidation $true
```

#### <a name="direct-api-update-method"></a>Método de actualización directo con la API ####
1. Cree una entidad de [back-end](/rest/api/apimanagement/) mediante API Management.     
2. Establezca la propiedad **skipCertificateChainValidation** en **true**.     
3. Si ya no desea permitir los certificados autofirmados, puede eliminar la entidad de back-end o establecer la propiedad **skipCertificateChainValidation** en **false**.

### <a name="why-do-i-get-an-authentication-failure-when-i-try-to-clone-a-git-repository"></a>¿Por qué se obtiene un error de autenticación al intentar clonar un repositorio?
Si utiliza el Administrador de credenciales de Git o si está intentando clonar un repositorio Git con Visual Studio, se puede encontrar un problema conocido en el cuadro de diálogo de credenciales de Windows. El cuadro de diálogo limita la longitud de la contraseña a 127 caracteres y se trunca la contraseña generada por Microsoft. Estamos trabajando para acortar la contraseña. De momento, use Git Bash para clonar el repositorio Git.

### <a name="does-api-management-work-with-azure-expressroute"></a>¿Funciona API Management con Azure ExpressRoute?
Sí. API Management funciona con Azure ExpressRoute.

### <a name="why-do-we-require-a-dedicated-subnet-in-resource-manager-style-vnets-when-api-management-is-deployed-into-them"></a>¿Por qué es necesaria una subred dedicada en las redes virtuales de estilo Resource Manager cuando API Management está implementado en ellas?
El requisito de subred dedicada para API Management procede del hecho de que se basa en el modelo de implementación clásica (capa V1 de PAAS). Aunque podemos implementarlo en una red virtual de Resource Manager (capa V2), hay consecuencias derivadas de ello. El modelo de implementación clásica de Azure no está estrechamente vinculado al modelo de Resource Manager, así que si crea un recurso en la capa V2, la capa V1 no lo sabe y pueden surgir problemas, por ejemplo, que API Management intente usar una IP que ya está asignada a una NIC (creada en V2).
Para más información sobre la diferencia entre el modelo de implementación clásica y el modelo de Resource Manager de Azure consulte las [diferencias en los modelos de implementación](../azure-resource-manager/management/deployment-models.md).

### <a name="what-is-the-minimum-subnet-size-needed-when-deploying-api-management-into-a-vnet"></a>¿Cuál es el tamaño de subred mínimo necesario al implementar API Management en una red virtual?
El tamaño de subred mínimo necesario para implementar API Management es [/29](../virtual-network/virtual-networks-faq.md#configuration), que es el mínimo que admite Azure.

### <a name="can-i-move-an-api-management-service-from-one-subscription-to-another"></a>¿Se puede mover un servicio API Management de una suscripción a otra?
Sí. Para más información, consulte vea [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md).

### <a name="are-there-restrictions-on-or-known-issues-with-importing-my-api"></a>¿Existen restricciones de la importación de mi API o problemas conocidos con ella?
[Problemas conocidos y restricciones](api-management-api-import-restrictions.md) para formatos Open API(Swagger), WSDL y WADL.
