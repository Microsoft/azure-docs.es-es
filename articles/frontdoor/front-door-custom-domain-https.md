---
title: 'Tutorial: Configuración de HTTPS en un dominio personalizado para Azure Front Door | Microsoft Docs'
description: En este tutorial, aprenderá a habilitar y deshabilitar HTTPS en su configuración de Azure Front Door para un dominio personalizado.
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/05/2018
ms.author: duau
ms.openlocfilehash: 3956a843e67dba82486f350fc4380d4c8f6065f1
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399827"
---
# <a name="tutorial-configure-https-on-a-front-door-custom-domain"></a>Tutorial: Configuración de HTTPS en un dominio personalizado de Front Door

En este tutorial se muestra cómo habilitar el protocolo HTTPS en un dominio personalizado asociado a un entorno de Front Door en la sección de hosts de front-end. Mediante el protocolo HTTPS en el dominio personalizado (por ejemplo, https:\//www.contoso.com), se garantiza que los datos confidenciales se entregan de manera segura a través del cifrado TLS/SSL cuando se envían por Internet. Cuando el explorador web se conecta a un sitio web a través de HTTPS, valida el certificado de seguridad del sitio web y comprueba que lo ha emitido una entidad de certificación legítima. Este proceso aporta seguridad y protege las aplicaciones web de posibles ataques.

De forma predeterminada, Azure Front Door admite HTTPS en el nombre de host predeterminado de una instancia de Front Door. Por ejemplo, si crea una instancia de Front Door (como `https://contoso.azurefd.net`), HTTPS se habilita automáticamente para las solicitudes realizadas a `https://contoso.azurefd.net`. Sin embargo, una vez incorporado el dominio personalizado "www.contoso.com", deberá habilitar además HTTPS para este host de front-end.   

Algunos de los atributos clave de la característica de HTTPS personalizado son:

- Sin costo adicional: la adquisición o renovación de certificados no tiene costos y el tráfico HTTPS no supone un costo adicional. 

- Habilitación simple: el aprovisionamiento en un solo clic está disponible desde [Azure Portal](https://portal.azure.com). También puede utilizar la API de REST u otras herramientas de desarrollo para habilitar la característica.

- La administración completa de certificados está disponible: toda la adquisición y administración de certificados se controla de manera automática. Los certificados se aprovisionan y se renuevan automáticamente antes de la expiración, lo que elimina los riesgos de interrupción del servicio debido a la expiración de un certificado.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> - Habilitar el protocolo HTTPS en su dominio personalizado
> - Usar un certificado administrado por AFD 
> - Usar su propio certificado, es decir, un certificado TLS/SSL personalizado
> - Validar el dominio
> - Deshabilitar el protocolo HTTPS en su dominio personalizado


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para poder completar los pasos de este tutorial, primero debe crear una instancia de Front Door e incorporar al menos un dominio personalizado. Para más información, consulte el [Tutorial: Incorporación de un dominio personalizado a Front Door](front-door-custom-domain.md).

## <a name="tlsssl-certificates"></a>Certificados TLS/SSL

Para habilitar el protocolo HTTPS para entregar contenido en un dominio personalizado de Front Door de forma segura, debe usar un certificado TLS/SSL. Puede usar un certificado que esté administrado por Azure Front Door o usar su propio certificado.


### <a name="option-1-default-use-a-certificate-managed-by-front-door"></a>Opción 1 (valor predeterminado): use un certificado administrado por Front Door

Cuando usa un certificado administrado por Azure Front Door, la característica HTTPS se puede activar con solo unos pocos clics. Azure Front Door administra completamente las tareas de administración de certificados, como adquisición y renovación. Después de habilitar la característica, el proceso se inicia inmediatamente. Si el dominio personalizado ya está asignado al host de front-end predeterminado de Front Door (`{hostname}.azurefd.net`), no es necesario realizar ninguna otra acción. Front Door procesa los pasos y completa la solicitud automáticamente. Sin embargo, si su dominio personalizado se asigna en otra parte, debe usar el correo electrónico para validar la propiedad del dominio.

Para habilitar HTTPS en un dominio personalizado, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), vaya a su perfil de **Front Door**.

2. En la lista de hosts de front-end, seleccione el dominio personalizado donde desea habilitar HTTPS para que contenga el dominio personalizado.

3. En la sección **Custom domain HTTPS** (HTTPS de dominio personalizado), haga clic en **Enabled** (Habilitado) y seleccione **Front Door managed** (Front Door administrado) como origen del certificado.

4. Haga clic en Guardar.

5. Continúe y [valide el dominio](#validate-the-domain).

> [!NOTE]
> En el caso de los certificados administrados por AFD, se aplica el límite de 64 caracteres de DigiCert. Se producirá un error de validación si se supera ese límite.


### <a name="option-2-use-your-own-certificate"></a>Opción 2: Usar su propio certificado

Puede usar su propio certificado para habilitar la característica HTTPS. Este proceso se realiza a través de una integración con Azure Key Vault, lo cual permite almacenar de forma segura los certificados. Azure Front Door usa este mecanismo de seguridad para obtener el certificado y requiere algunos pasos adicionales. Cuando cree el certificado TLS/SSL, debe usar una entidad de certificación (CA) permitida. Si no lo hace, si usa una entidad de certificación no permitida, la solicitud se rechazará. Para ver una lista de las entidades de certificación permitidas, consulte [Allowed certificate authorities for enabling custom HTTPS on Azure Front Door](front-door-troubleshoot-allowed-ca.md) (Autoridades de certificación permitidas para habilitar HTTPS personalizado en Azure Front Door).

#### <a name="prepare-your-azure-key-vault-account-and-certificate"></a>Preparación de la cuenta y el certificado de Azure Key Vault
 
1. Azure Key Vault: debe tener una cuenta de Azure Key Vault ejecutándose en la misma suscripción que su instancia de Front Door para la que quiera habilitar HTTPS personalizado. Cree una cuenta de Azure Key Vault si no tiene una.

> [!WARNING]
> Azure Front Door actualmente solo admite cuentas de Key Vault en la misma suscripción que la configuración de Front Door. Si elige un almacén de claves en una suscripción diferente que la de Front Door, obtendrá un error.

2. Certificados de Azure Key Vault: si ya tiene un certificado, puede cargarlo directamente en la cuenta de Azure Key Vault o puede crear uno nuevo directamente mediante Azure Key Vault a partir de una de las entidades de certificación asociadas con la que se integra Azure Key Vault. Cargue el certificado como un objeto de **certificado** en lugar de como un **secreto**.

> [!NOTE]
> Para su propio certificado TLS/SSL, Front Door no admite certificados con algoritmos de criptografía de EC.

#### <a name="register-azure-front-door"></a>Registro en Azure Front Door

Registre la entidad de servicio para Azure Front Door como una aplicación en Azure Active Directory mediante PowerShell.

> [!NOTE]
> Esta acción requiere permisos de administrador global y solo debe realizarse **una vez** por inquilino.

1. Si es necesario, instale [Azure PowerShell](/powershell/azure/install-az-ps) en PowerShell en la máquina local.

2. En PowerShell, ejecute el siguiente comando:

     `New-AzADServicePrincipal -ApplicationId "ad0e1c7e-6d38-4ba4-9efd-0bc77ba9f037"`              

#### <a name="grant-azure-front-door-access-to-your-key-vault"></a>Concesión a Azure Front Door de acceso al almacén de claves
 
Conceda a Azure Front Door permisos para acceder a los certificados ubicados en su cuenta de Azure Key Vault.

1. En su cuenta de almacén de claves, en Settings (Configuración), seleccione **Access policies** (Directivas de acceso) y, luego, seleccione **Add new** (Agregar nueva) para crear una nueva directiva.

2. En **Select principal** (Seleccionar entidad de seguridad), busque **ad0e1c7e-6d38-4ba4-9efd-0bc77ba9f037** y elija **Microsoft.Azure.Frontdoor**. Haga clic en **Seleccionar**.

3. En **Permisos de secretos**, seleccione **Obtener** para permitir que Front Door recupere el certificado.

4. En **Permisos de certificado**, seleccione **Obtener** para permitir que Front Door recupere el certificado.

5. Seleccione **Aceptar**. 

    Azure Front Door puede acceder ahora a este almacén de claves y a los certificados que se almacenan en él.
 
#### <a name="select-the-certificate-for-azure-front-door-to-deploy"></a>Selección del certificado de Azure Front Door que se va a implementar
 
1. Vuelva a su instancia de Front Door en el portal. 

2. En la lista de dominios personalizados, seleccione el dominio personalizado para el que desea habilitar HTTPS.

    Aparece la página **Dominio personalizado**.

3. En el tipo de administración de certificados, seleccione **Use my own certificate** (Usar mi propio certificado). 

4. Azure Front Door requiere que la suscripción de la cuenta de Key Vault sea la misma que la de la puerta de entrada. Seleccione un almacén de claves, un certificado (secreto) y la versión del certificado.

    Azure Front Door muestra la siguiente información: 
    - Las cuentas del almacén de claves de su identificador de suscripción. 
    - Los certificados (secretos) en el almacén de claves seleccionado. 
    - Las versiones de certificado disponibles. 
 
5. Si usa su propio certificado, no se requiere la validación del dominio. Continúe con [Esperar a la propagación](#wait-for-propagation).

## <a name="validate-the-domain"></a>Validar el dominio

Si ya tiene un dominio personalizado en uso que está asignado a su punto de conexión personalizado con un registro CNAME o usa su propio certificado, continúe con  
[El dominio personalizado está asignado a su instancia de Front Door](#custom-domain-is-mapped-to-your-front-door-by-a-cname-record). En caso contrario, si la entrada de registro CNAME para el dominio ya no existe o no contiene el subdominio afdverify, continúe con [El dominio personalizado no está asignado a la instancia de Front Door](#custom-domain-is-not-mapped-to-your-front-door).

### <a name="custom-domain-is-mapped-to-your-front-door-by-a-cname-record"></a>El dominio personalizado está asignado a la instancia de Front Door mediante un registro CNAME

Cuando agregó un dominio personalizado a los hosts de front-end de Front Door, creó un registro CNAME en la tabla DNS de su registrador de dominios para asignarlo al nombre de host .azurefd.net predeterminado de Front Door. Si ese registro CNAME todavía existe, pero no contiene el subdominio afdverify, la entidad de certificación DigiCert lo usa para validar la titularidad de su dominio personalizado. 

Si usa su propio certificado, no se requiere la validación del dominio.

El registro CNAME debe tener el formato siguiente, donde *Nombre* es el nombre de dominio personalizado y *Valor* es el nombre de host .azurefd.net predeterminado de Front Door:

| Nombre            | Tipo  | Value                 |
|-----------------|-------|-----------------------|
| <www.contoso.com> | CNAME | contoso.azurefd.net |

Para obtener más información sobre los registros CNAME, consulte [Creación de los registros DNS de CNAME](https://docs.microsoft.com/azure/cdn/cdn-map-content-to-custom-domain).

Si el registro CNAME tiene el formato correcto, DigiCert automáticamente comprueba su nombre de dominio personalizado y crea un certificado dedicado para el nombre de dominio. DigitCert no enviará un correo electrónico de comprobación, y usted no tendrá que aprobar la solicitud. El certificado tiene una validez de un año y se renovará automáticamente antes de expirar. Continúe con [Esperar a la propagación](#wait-for-propagation). 

La validación automática suele tarda unos minutos. Si no ve su dominio validado en una hora, abra una incidencia de soporte técnico.

>[!NOTE]
>Si tiene un registro de autorización de entidad de certificación (CAA) con el proveedor de DNS, debe incluir DigiCert como entidad de certificación válida. Un registro CAA permite que los propietarios de dominios especifiquen con sus proveedores de DNS qué entidades de certificación están autorizadas para emitir certificados para su dominio. Si una entidad de certificación recibe un pedido de un certificado para un dominio que posee un registro CAA y dicha entidad no figura como emisor autorizado, no se le permite emitir el certificado para ese dominio o subdominio. Para obtener información acerca de cómo administrar registros de CAA, consulte [Manage CAA records](https://support.dnsimple.com/articles/manage-caa-record/) (Administrar registros de CAA). Para obtener una herramienta de registros de CAA, consulte [CAA Record Helper](https://sslmate.com/caa/) (Asistente de registros de CAA).

### <a name="custom-domain-is-not-mapped-to-your-front-door"></a>El dominio personalizado no está asignado a su instancia de Front Door

Si la entrada del registro CNAME para el punto de conexión ya no existe o contiene el subdominio afdverify, siga el resto de las instrucciones que aparecen en este paso.

Después de habilitar HTTPS en un dominio personalizado, la entidad de certificación DigiCert valida la propiedad del dominio, para lo que se pondrá en contacto con el usuario inscrito, según la información de [WHOIS](http://whois.domaintools.com/) del dominio. El contacto se realiza a través de la dirección de correo electrónico (de forma predeterminada) o el número de teléfono que aparece en el registro WHOIS. Para que HTTPS pueda activarse en un dominio personalizado, antes es preciso completar la validación del dominio. Dispone de seis días laborables para aprobar el dominio. Se cancelarán automáticamente las solicitudes que no estén aprobadas en seis días laborables. 

![Registro WHOIS](./media/front-door-custom-domain-https/whois-record.png)

DigiCert también envía un mensaje de verificación a direcciones de correo electrónico adicionales. Si la información del usuario inscrito de WHOIS es privada, asegúrese de que puede realizar las aprobaciones directamente desde una de las direcciones siguientes:

admin@&lt;su-nombre-de-dominio.com&gt;  
administrator@&lt;su-nombre-de-dominio.com&gt;  
webmaster@&lt;su-nombre-de-dominio.com&gt;  
hostmaster@&lt;su-nombre-de-dominio.com&gt;  
postmaster@&lt;su-nombre-de-dominio.com&gt;  

Debería recibir un correo electrónico en unos minutos, de forma similar al ejemplo siguiente, solicitando que apruebe la solicitud. Si utiliza un filtro de correo no deseado, agregue admin@digicert.com a la lista de permitidos. Si no recibe un correo electrónico en 24 horas, póngase en contacto con el equipo de soporte técnico de Microsoft.

Al hacer clic en el vínculo de aprobación, obtendrá acceso a un formulario de aprobación en línea. Siga las instrucciones que aparecen en el formulario; tiene dos opciones de comprobación:

- Puede aprobar todos los pedidos futuros realizados a través de la misma cuenta del mismo dominio raíz, por ejemplo, contoso.com. Se recomienda este método si piensa agregar más dominios personalizados al mismo dominio raíz.

- Puede aprobar solo el nombre de host específico usado en esta solicitud. Para las solicitudes posteriores se requiere una aprobación adicional.

Tras la aprobación, DigiCert completa la creación del certificados para el nombre de dominio personalizado. El certificado tiene una validez de un año y se renovará automáticamente antes de expirar.

## <a name="wait-for-propagation"></a>Esperar a la propagación

Una vez que el nombre de dominio se valide, la característica HTTPS del dominio personalizado tarda entre 6 y 8 horas en estar activa. Una vez completado el proceso, el estado de HTTPS personalizado en Azure Portal se establece en **Habilitado** y los cuatro pasos de la operación en el cuadro de diálogo del dominio personalizado se marcan como completados. El dominio personalizado ahora está listo para usar HTTPS.

### <a name="operation-progress"></a>Progreso de la operación

En la tabla siguiente se muestra el progreso de la operación que se produce cuando se habilita HTTPS. Después de habilitar HTTPS, los cuatro pasos de la operación aparecen en el cuadro de diálogo del dominio personalizado. A medida que cada paso se activa, aparecen detalles de subpasos adicionales bajo el paso. No se producirán todos estos subpasos. Después de que un paso se completa correctamente, aparece una marca de verificación verde junto a él. 

| Paso de la operación | Detalles del subpaso de la operación | 
| --- | --- |
| 1 Envío de la solicitud | Envío de la solicitud |
| | Se está enviando la solicitud HTTPS. |
| | Se ha enviado correctamente la solicitud HTTPS. |
| 2 Validación del dominio | El dominio se validará automáticamente si tiene una asignación CNAME al host de front-end .azurefd.net predeterminado de Front Door. En caso contrario, se enviará una solicitud de verificación al correo electrónico que aparece en el registro de su dominio (registrante WHOIS). Compruebe el dominio lo antes posible. |
| | La propiedad del dominio se ha validado correctamente. |
| | La solicitud de validación de la propiedad del dominio ha expirado (el cliente probablemente no respondió dentro del plazo de 6 días). HTTPS no se habilitará en el dominio. * |
| | El cliente rechazó la solicitud de validación de la propiedad del dominio. HTTPS no se habilitará en el dominio. * |
| 3 Aprovisionamiento del certificado | La entidad de certificación está emitiendo actualmente el certificado necesario para habilitar HTTPS en el dominio. |
| | El certificado se ha emitido y actualmente se está implementando en Front Door. Este proceso puede tardar hasta 1 hora. |
| | El certificado se ha implementado correctamente en Front Door. |
| 4 Finalizado | Se habilitó correctamente HTTPS en el dominio. |

\* Este mensaje no aparecerá a menos que se haya producido un error. 

Si se produce un error antes de que se envíe la solicitud, verá el mensaje de error siguiente:

<code>
We encountered an unexpected error while processing your HTTPS request. Please try again and contact support if the issue persists.
</code>



## <a name="clean-up-resources---disable-https"></a>Clean up resources - disable HTTPS (Limpiar recursos: deshabilitar HTTPS)

En los pasos anteriores, habilitó el protocolo HTTPS en su dominio personalizado. Si ya no desea usar su dominio personalizado con HTTPS, puede deshabilitar HTTPS mediante estos pasos:

### <a name="disable-the-https-feature"></a>Deshabilitación de la característica HTTPS 

1. En [Azure Portal](https://portal.azure.com), vaya a la configuración de **Azure Front Door**.

2. En la lista de hosts de front-end, haga clic en el dominio personalizado en el que desea deshabilitar HTTPS.

3. Haga clic en **Disabled** (Deshabilitado) para deshabilitar HTTPS y haga clic en **Save** (Guardar).

### <a name="wait-for-propagation"></a>Esperar a la propagación

Cuando se deshabilita la característica HTTPS del dominio personalizado, puede tardar hasta 6 a 8 horas para que surta efecto. Una vez completado el proceso, el estado de HTTPS personalizado en Azure Portal se establece en **Deshabilitado** y los tres pasos de la operación en el cuadro de diálogo del dominio personalizado se marcan como completados. El dominio personalizado ya no usa HTTPS.

#### <a name="operation-progress"></a>Progreso de la operación

En la tabla siguiente se muestra el progreso de la operación que se produce cuando se deshabilita HTTPS. Después de deshabilitar HTTPS, los tres pasos de la operación aparecen en el cuadro de diálogo del dominio personalizado. A medida que cada paso se activa, aparecen detalles adicionales bajo el paso. Después de que un paso se completa correctamente, aparece una marca de verificación verde junto a él. 

| Progreso de la operación | Detalles de la operación | 
| --- | --- |
| 1 Envío de la solicitud | Enviando la solicitud |
| 2 Desaprovisionamiento del certificado | Eliminando certificado |
| 3 Finalizado | Certificado eliminado |

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

1. *¿Quién es el proveedor de certificados y qué tipo de certificado se utiliza?*

    Se usa un certificado único o dedicado, proporcionado por Digicert, para el dominio personalizado. 

2. *¿Usa TLS/SSL basado en IP o SNI?*

    Azure Front Door usa SNI TLS/SSL.

3. *¿Qué debo hacer si no recibo el correo electrónico de comprobación de dominio de DigiCert?*

    Si tiene una entrada CNAME para el dominio personalizado, que apunte directamente a su nombre de host del punto de conexión (y no utiliza el nombre del subdominio afdverify), no recibirá un correo electrónico de comprobación de dominio. La validación se produce automáticamente. En caso contrario, si no tiene una entrada CNAME y no ha recibido un correo electrónico en 24 horas, póngase en contacto con el servicio de soporte técnico de Microsoft.

4. *¿Son menos seguros los certificados SAN que los certificados dedicados?*
    
    Los certificados SAN siguen los mismos estándares de cifrado y seguridad que los certificados dedicados. Todos los certificados TLS/SSL emitidos usan SHA-256 para mejorar la seguridad del servidor.

5. *¿Necesito un registro de autorización de entidad de certificación con mi proveedor de DNS?*

    No, actualmente no se requiere un registro de autorización de entidad de certificación. Pero si tiene uno, debe incluir DigiCert como entidad de certificación válida.


## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).
