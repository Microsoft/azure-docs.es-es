---
title: Evitar adquisiciones de subdominios con registros de alias de Azure DNS y comprobación de dominios personalizados de Azure App Service
description: Obtenga información sobre cómo evitar la amenaza común de alta gravedad que supone la adquisición de subdominios.
services: security
author: memildin
manager: rkarlin
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/23/2020
ms.author: memildin
ms.openlocfilehash: 3d63ccc2c47bca9410b5b9105b90aa1f0cf5854a
ms.sourcegitcommit: 14bf4129a73de2b51a575c3a0a7a3b9c86387b2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87439262"
---
# <a name="prevent-dangling-dns-entries-and-avoid-subdomain-takeover"></a>Evitar las entradas DNS pendientes y la adquisición de subdominios

En este artículo se describe la amenaza de seguridad común de la adquisición de subdominios y los pasos que puede seguir para mitigarla.


## <a name="what-is-subdomain-takeover"></a>¿Qué es la adquisición de subdominios?

Las adquisiciones de subdominios son una amenaza común muy grave para las organizaciones que crean y eliminan muchos recursos con regularidad. Una adquisición de subdominios puede producirse cuando tiene un registro DNS que apunta a un recurso de Azure desaprovisionado. Estos registros DNS también se conocen como entradas "DNS pendientes". Los registros CNAME son especialmente vulnerables a esta amenaza.

El siguiente es un escenario común de adquisición de subdominio:

1. Se crea un sitio web. 

    En este ejemplo, `app-contogreat-dev-001.azurewebsites.net`.

1. Se agrega una entrada CNAME al DNS que apunta al sitio web. 

    En este ejemplo, se creó el siguiente nombre descriptivo: `greatapp.contoso.com`.

1. Después de unos meses, el sitio ya no es necesario, por lo que se elimina **sin** eliminar la entrada DNS correspondiente. 

    La entrada DNS de CNAME ahora está "pendiente".

1. Casi inmediatamente después de eliminar el sitio, un actor de amenaza detecta el sitio que falta y crea su propio sitio web en `app-contogreat-dev-001.azurewebsites.net`.

    Ahora, el tráfico destinado a `greatapp.contoso.com` se dirige al sitio de Azure del actor de amenaza, y el actor de amenaza controla el contenido que se muestra. 

    Se han aprovechado del DNS pendiente y el subdominio "GreatApp" de Contoso ha sido víctima de la adquisición de subdominios. 

![Adquisición de subdominios desde un sitio web desaprovisionado](./media/subdomain-takeover/subdomain-takeover.png)



## <a name="the-risks-of-subdomain-takeover"></a>Riesgos de la adquisición de subdominios

Cuando un registro DNS apunta a un recurso que no está disponible, el propio registro debería quitarse de la zona DNS. Si no se ha eliminado, es un registro "DNS pendiente" y genera la posibilidad de que se produzca la adquisición de subdominios.

Las entradas DNS pendientes permiten a los actores de amenazas tomar el control del nombre DNS asociado para hospedar un servicio o un sitio web malintencionado. Las páginas y los servicios malintencionados del subdominio de una organización pueden dar lugar a:

- **Pérdida de control sobre el contenido del subdominio**: prensa negativa sobre la incapacidad de la organización para proteger su contenido, así como perjuicios para la marca y pérdida de confianza.

- **Recopilación de cookies de visitantes confiados**: es habitual que las aplicaciones web expongan cookies de sesión a subdominios (*.contoso.com), de modo que cualquier subdominio puede acceder a estas. Los actores de amenazas pueden usar la adquisición de subdominios para crear una página de aspecto auténtico, engañar a los usuarios confiados para que la visiten y recopilar sus cookies (incluso las cookies seguras). Una idea equivocada habitual es que el uso de certificados SSL protege su sitio y las cookies de los usuarios de una adquisición. Sin embargo, un actor de amenaza puede usar el subdominio secuestrado para solicitar y recibir un certificado SSL válido. Los certificados SSL válidos les conceden acceso a las cookies seguras y pueden aumentar aún más la legitimidad aparente del sitio malintencionado.

- **Campañas de suplantación de identidad (phishing)** : los subdominios de aspecto auténtico se pueden usar en campañas de suplantación de identidad. Esto se aplica a los sitios malintencionados y también a los registros MX, que permitirían que el actor de amenaza recibiera correos electrónicos dirigidos a un subdominio legítimo de una marca segura.

- **Más riesgos**: los sitios malintencionados se pueden usar para escalar a otros ataques clásicos, como XSS, CSRF, omisión de CORS, etc.



## <a name="preventing-dangling-dns-entries"></a>Impedir las entradas DNS pendientes

Asegurarse de que su organización ha implementado procesos para evitar las entradas DNS pendientes y las adquisiciones de subdominios resultantes es una parte fundamental del programa de seguridad.

A continuación se muestran las medidas preventivas que tiene a su disposición actualmente.


### <a name="use-azure-dns-alias-records"></a>Uso de registros de alias de Azure DNS

Los [registros de alias](https://docs.microsoft.com/azure/dns/dns-alias#scenarios) de Azure DNS pueden evitar referencias pendientes mediante un acoplamiento del ciclo de vida de un registro DNS con un recurso de Azure. Por ejemplo, considere un registro DNS que se califica como un registro de alias que apunte a una dirección IP pública o a un perfil de Traffic Manager. Si elimina los recursos subyacentes, el registro de alias de DNS se convierte en un conjunto de registros vacío. Ya no hace referencia al recurso eliminado. Es importante tener en cuenta que se aplican límites a lo que se puede proteger con los registros de alias. Actualmente, esta lista se limita a lo siguiente:

- Azure Front Door
- Perfiles de Traffic Manager
- Puntos de conexión de Azure Content Delivery Network (CDN)
- Direcciones IP públicas

A pesar de las ofertas de servicio limitadas actualmente, se recomienda usar registros de alias para defenderse de la adquisición de subdominios siempre que sea posible.

[Obtenga más información](https://docs.microsoft.com/azure/dns/dns-alias#capabilities) sobre las funcionalidades y los registros de alias de Azure DNS.



### <a name="use-azure-app-services-custom-domain-verification"></a>Uso de la comprobación de dominio personalizado de Azure App Service

Al crear entradas DNS para Azure App Service, cree un registro TXT asuid.{subdominio} con el identificador de comprobación de dominio. Cuando existe un registro TXT de este tipo, ninguna otra suscripción a Azure puede validar el dominio personalizado es decir, adquirirlo. 

Estos registros no impiden que alguien cree la instancia de Azure App Service con el mismo nombre que en la entrada CNAME. Sin la capacidad de demostrar la propiedad del nombre de dominio, los actores de amenazas no pueden recibir tráfico ni controlar el contenido.

[Obtenga más información](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain) sobre cómo asignar un nombre DNS personalizado existente a Azure App Service.



### <a name="build-and-automate-processes-to-mitigate-the-threat"></a>Compilación y automatización de procesos para mitigar la amenaza

A menudo, los desarrolladores y los equipos de operaciones deben ejecutar procesos de limpieza para evitar las amenazas de DNS pendientes. Los procedimientos siguientes le ayudarán a asegurarse de que su organización evita sufrir esta amenaza. 

- **Creación de procedimientos para la prevención:**

    - Instruya a los desarrolladores de aplicaciones para que redirijan las direcciones siempre que eliminen recursos.

    - Incluya "Quitar entrada DNS" en la lista de comprobaciones necesarias al retirar un servicio.

    - Incluya [bloqueos de eliminación](https://docs.microsoft.com/azure/azure-resource-manager/management/lock-resources) en los recursos que tengan una entrada DNS personalizada. Un bloqueo de eliminación sirve como indicador para quitar la asignación antes de desaprovisionar el recurso. Medidas como esta solo pueden funcionar si se combinan con programas educativos internos.

- **Creación de procedimientos para la detección:**

    - Revise los registros DNS con regularidad para asegurarse de que todos los subdominios están asignados a recursos de Azure:

        - Existentes: consulte las zonas DNS para ver los recursos que apuntan a subdominios de Azure, como *.azurewebsites.net o *.cloudapp.azure.com (vea [esta lista de referencia](azure-domains.md)).
        - De su propiedad: confirme que posee todos los recursos a los que se dirigen sus subdominios DNS.

    - Mantenga un catálogo de servicios de los puntos de conexión de nombre de dominio completo (FQDN) de Azure y los propietarios de la aplicación. Para compilar el catálogo de servicios, ejecute el siguiente script de consulta de Azure Resource Graph. Este script proyecta la información del punto de conexión de FQDN de los recursos a los que tiene acceso y los envía a un archivo CSV. Si tiene acceso a todas las suscripciones de su inquilino, el script tiene en cuenta todas estas suscripciones, tal y como se muestra en el siguiente script de ejemplo. Para limitar los resultados a un conjunto específico de suscripciones, edite el script como se muestra.

        >[!IMPORTANT]
        > **Permisos**: ejecute la consulta como un usuario con acceso a todas sus suscripciones de Azure. 
        >
        > **Limitaciones**: Azure Resource Graph tiene límites paginación y límites de ancho de banda que debe tener en cuenta si tiene un entorno de Azure de gran tamaño. [Obtenga más información](https://docs.microsoft.com/azure/governance/resource-graph/concepts/work-with-data) sobre el trabajo con grandes conjuntos de datos de recursos de Azure. El siguiente script de ejemplo usa el procesamiento por lotes de suscripciones para evitar estas limitaciones.

        ```powershell
        
            # Fetch the full array of subscription IDs.
            $subscriptions = Get-AzSubscription

            $subscriptionIds = $subscriptions.Id
                   # Output file path and names
                   $date = get-date
                   $fdate = $date.ToString("MM-dd-yyy hh_mm_ss tt")
                   $fdate #log to console
                   $rpath = [Environment]::GetFolderPath("MyDocuments") + '\' # Feel free to update your path.
                   $rname = 'Tenant_FQDN_Report_' + $fdate + '.csv' # Feel free to update the document name.
                   $fpath = $rpath + $rname
                   $fpath #This is the output file of FQDN report.

            # query
            $query = "where type in ('microsoft.network/frontdoors',
                                    'microsoft.storage/storageaccounts',
                                    'microsoft.cdn/profiles/endpoints',
                                    'microsoft.network/publicipaddresses',
                                    'microsoft.network/trafficmanagerprofiles',
                                    'microsoft.containerinstance/containergroups',
                                    'microsoft.apimanagement/service',
                                    'microsoft.web/sites',
                                    'microsoft.web/sites/slots')
                        | extend FQDN = case(
                            type =~ 'microsoft.network/frontdoors', properties['cName'],
                            type =~ 'microsoft.storage/storageaccounts', parse_url(tostring(properties['primaryEndpoints']['blob'])).Host,
                            type =~ 'microsoft.cdn/profiles/endpoints', properties['hostName'],
                            type =~ 'microsoft.network/publicipaddresses', properties['dnsSettings']['fqdn'],
                            type =~ 'microsoft.network/trafficmanagerprofiles', properties['dnsConfig']['fqdn'],
                            type =~ 'microsoft.containerinstance/containergroups', properties['ipAddress']['fqdn'],
                            type =~ 'microsoft.apimanagement/service', properties['hostnameConfigurations']['hostName'],
                            type =~ 'microsoft.web/sites', properties['defaultHostName'],
                            type =~ 'microsoft.web/sites/slots', properties['defaultHostName'],
                            '')
                        | project id, ['type'], name, FQDN
                        | where isnotempty(FQDN)";

            # Paging helper cursor
            $Skip = 0;
            $First = 1000;

            # If you have large number of subscriptions, process them in batches of 2,000.
            $counter = [PSCustomObject] @{ Value = 0 }
            $batchSize = 2000
            $response = @()

            # Group the subscriptions into batches.
            $subscriptionsBatch = $subscriptionIds | Group -Property { [math]::Floor($counter.Value++ / $batchSize) }

            # Run the query for each subscription batch with paging.
            foreach ($batch in $subscriptionsBatch)
            { 
                $Skip = 0; #Reset after each batch.

                $response += do { Start-Sleep -Milliseconds 500;   if ($Skip -eq 0) {$y = Search-AzGraph -Query $query -First $First -Subscription $batch.Group ; } `
                else {$y = Search-AzGraph -Query $query -Skip $Skip -First $First -Subscription $batch.Group } `
                $cont = $y.Count -eq $First; $Skip = $Skip + $First; $y; } while ($cont)
            }

            # View the completed results of the query on all subscriptions.
            $response | Export-Csv -Path $fpath -Append 

        ```

        Lista de tipos y sus valores `FQDNProperty` tal y como se especifica en la consulta de Resource Graph anterior:

        |Nombre del recurso  | `<ResourceType>`  | `<FQDNproperty>`  |
        |---------|---------|---------|
        |Azure Front Door|microsoft.network/frontdoors|properties.cName|
        |Azure Blob Storage|microsoft.storage/storageaccounts|properties.primaryEndpoints.blob|
        |Azure CDN|microsoft.cdn/profiles/endpoints|properties.hostName|
        |Direcciones IP públicas|microsoft.network/publicipaddresses|properties.dnsSettings.fqdn|
        |Administrador de tráfico de Azure|microsoft.network/trafficmanagerprofiles|properties.dnsConfig.fqdn|
        |Azure Container Instances|microsoft.containerinstance/containergroups|properties.ipAddress.fqdn|
        |Azure API Management|microsoft.apimanagement/service|properties.hostnameConfigurations.hostName|
        |Azure App Service|microsoft.web/sites|properties.defaultHostName|
        |Azure App Service - Slots|microsoft.web/sites/slots|properties.defaultHostName|


- **Creación de procedimientos para la corrección:**
    - Cuando se encuentran entradas DNS pendientes, su equipo debe investigar si se ha producido algún riesgo.
    - Investigue por qué la dirección no se reenrutó al retirar el recurso.
    - Elimine el registro DNS si ya no está en uso o apunte al recurso de Azure (FQDN) correcto que pertenece a su organización.
 

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los servicios relacionados y las características de Azure que puede usar para defenderse de la adquisición de subdominios, consulte las páginas siguientes.

- [Azure DNS admite el uso de registros de alias para dominios personalizados](https://docs.microsoft.com/azure/dns/dns-alias#prevent-dangling-dns-records)

- [Uso del identificador de comprobación de dominio al agregar dominios personalizados en Azure App Service](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain#get-domain-verification-id) 

- [Inicio rápido: Ejecutar la primera consulta de Resource Graph con Azure PowerShell](https://docs.microsoft.com/azure/governance/resource-graph/first-query-powershell)
