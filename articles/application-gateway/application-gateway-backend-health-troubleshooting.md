---
title: Solución de problemas de estado del back-end en Azure Application Gateway
description: Se describe cómo solucionar los problemas de estado del back-end de Azure Application Gateway.
services: application-gateway
author: surajmb
ms.service: application-gateway
ms.topic: troubleshooting
ms.date: 06/09/2020
ms.author: surmb
ms.openlocfilehash: 531a7fd8547130b4897f3dad0900e1c27fb7fe9a
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132048"
---
<a name="troubleshoot-backend-health-issues-in-application-gateway"></a>Solución de problemas de estado del back-end en Application Gateway
==================================================

<a name="overview"></a>Información general
--------

De forma predeterminada, Application Gateway sondea los servidores back-end para comprobar su estado de mantenimiento y si están listos para atender solicitudes. Los usuarios pueden crear también sondeos personalizados para mencionar el nombre de host, la ruta de acceso que se va a sondear y los códigos de estado que se van a aceptar como correctos. En cada caso, si el servidor back-end no responde correctamente, Application Gateway lo marca como incorrecto y deja de reenviarle solicitudes. Una vez que el servidor comienza a responder correctamente, Application Gateway reanuda el reenvío de las solicitudes.

### <a name="how-to-check-backend-health"></a>Comprobación del estado del servidor back-end

Para comprobar el mantenimiento del grupo back-end, puede usar la página **Estado del back-end** en Azure Portal. O bien, puede utilizar [Azure PowerShell](https://docs.microsoft.com/powershell/module/az.network/get-azapplicationgatewaybackendhealth?view=azps-2.6.0), la [CLI](https://docs.microsoft.com/cli/azure/network/application-gateway?view=azure-cli-latest#az-network-application-gateway-show-backend-health) o una [API REST](https://docs.microsoft.com/rest/api/application-gateway/applicationgateways/backendhealth).

El estado recuperado por cualquiera de estos métodos puede ser uno de los siguientes:

- Healthy

- Unhealthy (Incorrecto)

- Desconocido

Si el estado de mantenimiento del back-end de un servidor es correcto, significa que Application Gateway reenviará las solicitudes a ese servidor. Pero si el mantenimiento del back-end de todos los servidores de un grupo back-end es incorrecto o desconocido, es posible que se produzcan problemas al intentar acceder a las aplicaciones. En este artículo se describen los síntomas, la causa y la resolución de cada uno de los errores mostrados.

<a name="backend-health-status-unhealthy"></a>Estado de mantenimiento del back-end: Unhealthy (Incorrecto)
-------------------------------

Si el estado de mantenimiento del back-end es incorrecto, la vista del portal se parecerá a la siguiente captura de pantalla:

![Estado del back-end de Application Gateway: incorrecto](./media/application-gateway-backend-health-troubleshooting/appgwunhealthy.png)

O bien, si usa una consulta de Azure PowerShell, la CLI o la API REST de Azure, obtendrá una respuesta similar a la siguiente:
```azurepowershell
PS C:\Users\testuser\> Get-AzApplicationGatewayBackendHealth -Name "appgw1" -ResourceGroupName "rgOne"
BackendAddressPools :
{Microsoft.Azure.Commands.Network.Models.PSApplicationGatewayBackendHealthPool}
BackendAddressPoolsText : [
{
                              "BackendAddressPool": {
                                "Id": "/subscriptions/536d30b8-665b-40fc-bd7e-68c65f816365/resourceGroups/rgOne/providers/Microsoft.Network/applicationGateways/appgw1/b
                          ackendAddressPools/appGatewayBackendPool"
                              },
                              "BackendHttpSettingsCollection": [
                                {
                                  "BackendHttpSettings": {
                                    "TrustedRootCertificates": [],
                                    "Id": "/subscriptions/536d30b8-665b-40fc-bd7e-68c65f816365/resourceGroups/rgOne/providers/Microsoft.Network/applicationGateways/appg
                          w1/backendHttpSettingsCollection/appGatewayBackendHttpSettings"
                                  },
                                  "Servers": [
                                    {
                                      "Address": "10.0.0.5",
                                      "Health": "Healthy"
                                    },
                                    {
                                      "Address": "10.0.0.6",
                                      "Health": "Unhealthy"
                                    }
                                  ]
                                }
                              ]
                            }
                        ]
```
Después de recibir un estado incorrecto para todos los servidores de un grupo back-end, las solicitudes no se reenvían a los servidores y Application Gateway devuelve un error "502 Puerta de enlace incorrecta" al cliente que realiza la solicitud. Para solucionar este problema, compruebe la columna **Detalles** de la pestaña **Estado del back-end**.

El mensaje que se muestra en la columna **Detalles** proporciona información más detallada sobre el problema y, según dicha información, podemos empezar a solucionarlo.

> [!NOTE]
> La solicitud de sondeo predeterminada se envía con el formato \<protocol\>://127.0.0.1:\<port\>/. Por ejemplo, http://127.0.0.1:80 para un sondeo HTTP en el puerto 80. Solo los códigos de estado HTTP del 200 al 399 se consideran correctos. El protocolo y el puerto de destino se heredan de la configuración de HTTP. Si quiere que Application Gateway sondee en un protocolo, un nombre de host o una ruta de acceso diferentes, y acepte como correcto un código de estado distinto, configure un sondeo personalizado y asócielo con la configuración de HTTP.

<a name="error-messages"></a>Mensajes de error
------------------------
#### <a name="backend-server-timeout"></a>Tiempo de espera del servidor back-end

**Mensaje**: Time taken by the backend to respond to application gateway\'s health probe is more than the time-out threshold in the probe setting. (El tiempo que tarda el servidor back-end en responder al sondeo de estado de Application Gateway es mayor que el umbral de tiempo de espera de la configuración de sondeo).

**Causa:** una vez que Application Gateway envía una solicitud de sondeo HTTP(S) al servidor back-end, espera su respuesta durante un período configurado. Si el servidor back-end no responde dentro del período configurado (el valor de tiempo de espera), se marca como incorrecto hasta que empieza a responder de nuevo en el período de tiempo de espera configurado.

**Resolución:** compruebe por qué el servidor back-end o la aplicación no responden dentro del período de espera configurado, y compruebe también las dependencias de la aplicación. Por ejemplo, compruebe si la base de datos tiene algún problema que pueda desencadenar un retraso en la respuesta. Si tiene constancia del comportamiento de la aplicación y debe responder solo después del valor de tiempo de espera, aumente el valor de tiempo de espera en la configuración de sondeo personalizado. Debe tener un sondeo personalizado para cambiar el valor de tiempo de espera. Para información sobre cómo configurar un sondeo personalizado, [consulte la página de documentación](https://docs.microsoft.com/azure/application-gateway/application-gateway-create-probe-portal).

Para aumentar el valor de tiempo de espera, siga estos pasos:

1.  Acceda directamente al servidor back-end y compruebe el tiempo que tarda el servidor en responder en esa página. Puede usar cualquier herramienta para acceder al servidor back-end, como un explorador con herramientas de desarrollo.

1.  Cuando haya averiguado el tiempo necesario para que responda la aplicación, seleccione la pestaña **Sondeos de estado** y seleccione el sondeo que está asociado a la configuración de HTTP.

1.  Especifique un valor de tiempo de espera mayor que el tiempo de respuesta de la aplicación, en segundos.

1.  Guarde la configuración de sondeo personalizada y compruebe si el estado del back-end aparece ahora como correcto.

#### <a name="dns-resolution-error"></a>Error de resolución DNS

**Mensaje**: Application Gateway could not create a probe for this backend. This usually happens when the FQDN of the backend has not been entered correctly. (Application Gateway no pudo crear un sondeo para este back-end. Esto suele ocurrir cuando el nombre de dominio completo del back-end no se ha escrito de forma apropiada). 

**Causa:** si el grupo back-end es de tipo Dirección IP/Nombre de dominio completo o App Service, Application Gateway resuelve la dirección IP del nombre de dominio completo especificado mediante el Sistema de nombres de dominio (personalizado o predeterminado de Azure) e intenta conectarse al servidor en el puerto TCP mencionado en la configuración de HTTP. Sin embargo, la aparición de este mensaje sugiere que Application Gateway no pudo resolver correctamente la dirección IP del nombre de dominio completo especificado.

**Resolución:**

1.  compruebe que el nombre de dominio completo especificado en el grupo back-end sea correcto y que sea un dominio público y, luego, intente resolverlo desde la máquina local.

1.  Si puede resolver la dirección IP, podría haber algún problema con la configuración DNS en la red virtual.

1.  Compruebe si la red virtual está configurada con un servidor DNS personalizado. Si es así, compruebe el servidor DNS para ver por qué no puede resolver la dirección IP del nombre de dominio completo especificado.

1.  Si usa el DNS predeterminado de Azure, compruebe con el registrador de nombres de dominio que se haya realizado correctamente una asignación apropiada de registro de dirección D o CNAME.

1.  Si el dominio es privado o interno, intente resolverlo desde una máquina virtual de la misma red virtual. Si puede resolverlo, reinicie Application Gateway y vuelva a comprobarlo. Para reiniciar Application Gateway, debe [detenerlo](https://docs.microsoft.com/powershell/module/azurerm.network/stop-azurermapplicationgateway?view=azurermps-6.13.0) e [iniciarlo](https://docs.microsoft.com/powershell/module/azurerm.network/start-azurermapplicationgateway?view=azurermps-6.13.0) mediante los comandos de PowerShell descritos en estos servicios vinculados.

#### <a name="tcp-connect-error"></a>Error de conexión TCP

**Mensaje**: Application Gateway could not connect to the backend.
Please check that the backend responds on the port used for the probe.
Also check whether any NSG/UDR/Firewall is blocking access to the Ip and port of this backend (Application Gateway no pudo conectarse al back-end. Compruebe que el back-end responde en el puerto usado para el sondeo. Compruebe también si NSG, UDR o el Firewall están bloqueando el acceso a la dirección IP y al puerto de este back-end).

**Causa:** después de la fase de resolución de DNS, Application Gateway intenta conectarse al servidor back-end en el puerto TCP establecido en la configuración de HTTP. Si Application Gateway no puede establecer una sesión TCP en el puerto especificado, el sondeo se marca como incorrecto con este mensaje.

**Solución:** si recibe este error, siga estos pasos:

1.  Compruebe si puede conectarse al servidor back-end en el puerto mencionado en la configuración HTTP mediante un explorador o PowerShell. Por ejemplo, ejecute el siguiente comando: `Test-NetConnection -ComputerName
    www.bing.com -Port 443`.

1.  Si el puerto mencionado no es el deseado, escriba el número de puerto correcto para que Application Gateway se conecte al servidor back-end.

1.  Si no puede conectarse tampoco en el puerto desde la máquina local, entonces:

    a.  Compruebe la configuración del grupo de seguridad de red (NSG) del adaptador de red y la subred del servidor back-end y si se permiten conexiones entrantes al puerto configurado. Si no están permitidas, cree una regla para hacerlo. Para aprender a crear reglas de NSG, [consulte la página de documentación](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic#create-security-rules).

    b.  Compruebe si la configuración de NSG de la subred de Application Gateway permite el tráfico saliente público y privado, de modo que se pueda realizar una conexión. Consulte la página del documento que se proporciona en el paso 3A para más información sobre cómo crear reglas de NSG.
    ```azurepowershell
            $vnet = Get-AzVirtualNetwork -Name "vnetName" -ResourceGroupName "rgName"
            Get-AzVirtualNetworkSubnetConfig -Name appGwSubnet -VirtualNetwork $vnet
    ```

    c.  Compruebe la configuración de las rutas definidas por el usuario (UDR) de Application Gateway y la subred del servidor back-end para ver si existe alguna anomalía de enrutamiento. Asegúrese de que la UDR no esté alejando el tráfico de la subred de back-end. Por ejemplo, busque rutas a las aplicaciones virtuales de red o rutas predeterminadas anunciadas en la subred de Application Gateway mediante Azure ExpressRoute o VPN.

    d.  Para buscar un adaptador de red en las rutas y reglas actuales, puede usar los siguientes comandos de PowerShell:
    ```azurepowershell
            Get-AzEffectiveNetworkSecurityGroup -NetworkInterfaceName "nic1" -ResourceGroupName "testrg"
            Get-AzEffectiveRouteTable -NetworkInterfaceName "nic1" -ResourceGroupName "testrg"
    ```
1.  Si no encuentra ningún problema con el grupo de NSG o la UDR, compruebe el servidor back-end para ver si existen problemas relacionados con la aplicación que impidan que los clientes establezcan una sesión TCP en los puertos configurados. Puntos que hay que comprobar:

    a.  Abra un símbolo del sistema (Win+R-\> cmd), escriba `netstat` y seleccione Entrar.

    b.  Compruebe si el servidor está escuchando en el puerto configurado. Por ejemplo:
    ```
            Proto Local Address Foreign Address State PID
            TCP 0.0.0.0:80 0.0.0.0:0 LISTENING 4
    ```
    c.  Si no está escuchando en el puerto configurado, compruebe la configuración del servidor web. Por ejemplo: enlaces de sitio en IIS, bloqueo del servidor en NGINX y host virtual en Apache.

    d.  Compruebe la configuración de firewall del sistema operativo para asegurarse de que se permite el tráfico entrante al puerto.

#### <a name="http-status-code-mismatch"></a>Error de coincidencia en el código de estado HTTP

**Mensaje**: Status code of the backend\'s HTTP response did not match the probe setting. Expected:{HTTPStatusCode0} Received:{HTTPStatusCode1}. (El código de estado de la respuesta HTTP del back-end no coincidía con la configuración de sondeo. Se esperaba: {HTTPStatusCode0} Se recibió: {HTTPStatusCode1}).

**Causa:** cuando haya establecido la conexión TCP y se haya realizado el protocolo de enlace TLS (si TLS está habilitado), Application Gateway enviará el sondeo como una solicitud GET HTTP al servidor back-end. Tal y como se ha mencionado anteriormente, el sondeo predeterminado será en \<protocol\>://127.0.0.1:\<port\>/, y se considera que los códigos de estado de respuesta en el intervalo de 200 a 399 son correctos. Si el servidor devuelve cualquier otro código de estado, se marcará como incorrecto con este mensaje.

**Solución:** según el código de respuesta del servidor back-end, puede realizar los pasos siguientes. A continuación se enumeran algunos de los códigos de estado comunes:

| **Error** | **Acciones** |
| --- | --- |
| Error de coincidencia de código de estado de sondeo: Recibido 401 | Compruebe si el servidor back-end necesita autenticación. Los sondeos de Application Gateway no pueden pasar credenciales para la autenticación. Permita \"HTTP 401\" en una coincidencia de código de estado de sondeo o sondee en una ruta de acceso en la que el servidor no requiera autenticación. | |
| Error de coincidencia de código de estado de sondeo: Recibido 403 | Acceso prohibido. Compruebe si se permite el acceso a la ruta de acceso en el servidor back-end. | |
| Error de coincidencia de código de estado de sondeo: Recibido 404 | Página no encontrada. Compruebe si la ruta de acceso del nombre de host es accesible en el servidor back-end. Cambie el nombre de host o el parámetro de ruta de acceso por un valor accesible. | |
| Error de coincidencia de código de estado de sondeo: Recibido 405 | Las solicitudes de sondeo de Application Gateway usan el método HTTP GET. Compruebe si el servidor lo permite. | |
| Error de coincidencia de código de estado de sondeo: Recibido 500 | Error interno del servidor Compruebe el estado del servidor back-end y si los servicios se están ejecutando. | |
| Error de coincidencia de código de estado de sondeo: Recibido 503 | Servicio no disponible. Compruebe el estado del servidor back-end y si los servicios se están ejecutando. | |

O bien, si cree que la respuesta es legítima y desea que Application Gateway acepte otros códigos de estado como correctos, puede crear un sondeo personalizado. Este enfoque es útil en situaciones en las que el sitio web de back-end necesita autenticación. Dado que las solicitudes de sondeo no incluyen ninguna credencial de usuario, generarán un error y el servidor back-end devolverá un código de estado HTTP 401.

Para crear un sondeo personalizado, siga [estos pasos](https://docs.microsoft.com/azure/application-gateway/application-gateway-create-probe-portal).

#### <a name="http-response-body-mismatch"></a>Error de coincidencia del cuerpo de la respuesta HTTP

**Mensaje**: Body of the backend\'s HTTP response did not match the probe setting. Received response body does not contain {string}. (El cuerpo de la respuesta HTTP del back-end no coincidía con la configuración de sondeo. El cuerpo de respuesta recibido no contiene {string}).

**Causa:** al crear un sondeo personalizado, tiene una opción para marcar un servidor back-end con estado correcto si hace coincidir una cadena del cuerpo de respuesta. Por ejemplo, puede configurar Application Gateway para que acepte "no autorizado" como una cadena que coincida. Si la respuesta del servidor back-end para la solicitud de sondeo contiene la cadena **no autorizado**, se marcará como correcta. De lo contrario, se marcará como incorrecta y se generará este mensaje.

**Solución:** Para resolver el problema, siga estos pasos:

1.  Acceda al servidor back-end localmente o desde una máquina cliente en la ruta de acceso de sondeo y compruebe el cuerpo de respuesta.

1.  Vea si el cuerpo de la respuesta de la configuración de sondeo personalizado de Application Gateway coincide con lo que se ha configurado.

1.  Si no coincide, cambie la configuración de sondeo para que tenga el valor de cadena correcto para aceptar.

Más información sobre la [coincidencia del sondeo de Application Gateway](https://docs.microsoft.com/azure/application-gateway/application-gateway-probe-overview#probe-matching).

>[!NOTE]
> En el caso de todos los mensajes de error relacionados con TLS, para obtener más información sobre el comportamiento de SNI y las diferencias entre las versiones 1 y 2 de la SKU, consulte la página de [información general de TLS](ssl-overview.md).


#### <a name="backend-server-certificate-invalid-ca"></a>Entidad de certificación no válida del certificado de servidor back-end

**Mensaje**: The server certificate used by the backend is not signed by a well-known Certificate Authority (CA). Allow the backend on the application gateway by uploading the root certificate of the server certificate used by the backend. (La entidad de certificación conocida no ha firmado el certificado de servidor usado por el back-end. Permita el back-end en la puerta de enlace de aplicaciones cargando el certificado raíz del certificado de servidor usado por el back-end).

**Causa:** SSL de un extremo a otro con Application Gateway v2 requiere que se compruebe el certificado del servidor back-end para que el estado del servidor se considere correcto.
Para que un certificado TLS o SSL sea de confianza, ese certificado del servidor back-end debe haberlo emitido una entidad de certificación incluida en el almacén de confianza de Application Gateway. Si no ha sido así (por ejemplo, se usaron certificados autofirmados), los usuarios deben cargar el certificado del emisor en Application Gateway.

**Solución:** siga estos pasos para exportar y cargar el certificado raíz de confianza en Application Gateway. (Estos pasos son para clientes de Windows).

1.  Inicie sesión en la máquina donde se hospeda la aplicación.

1.  Seleccione Win+R o haga clic con el botón derecho en el botón **Inicio** y seleccione **Ejecutar**.

1.  Escriba `certmgr.msc` y seleccione Entrar. También puede buscar Administrador de certificados en el menú **Inicio**.

1.  Busque el certificado, normalmente en `\Certificates - Current User\\Personal\\Certificates\`, y ábralo.

1.  Seleccione el certificado raíz y, luego, **Ver certificado**.

1.  En Propiedades del certificado, seleccione la pestaña **Detalles**.

1.  En la pestaña **Detalles**, seleccione la opción **Copiar a archivo** y guarde el archivo en el formato X.509 codificado en Base 64 (.CER).

1.  Abra la página **Configuración HTTP** de Application Gateway en Azure Portal.

1. Abra la configuración de HTTP, seleccione **Agregar certificado** y busque el archivo de certificado que se acaba de guardar.

1. Seleccione **Guardar** para guardar la configuración HTTP.

Como alternativa, para exportar el certificado raíz desde una máquina cliente, puede acceder directamente al servidor (omitiendo Application Gateway) mediante un explorador y exportar desde ahí el certificado raíz.

Para más información sobre cómo extraer y cargar certificados raíz de confianza en Application Gateway, consulte [Exportación del certificado raíz de confianza (para SKU V2) ](https://docs.microsoft.com/azure/application-gateway/certificates-for-backend-authentication#export-trusted-root-certificate-for-v2-sku).

#### <a name="trusted-root-certificate-mismatch"></a>Error de coincidencia de certificado raíz de confianza

**Mensaje**: The root certificate of the server certificate used by the backend does not match the trusted root certificate added to the application gateway. Ensure that you add the correct root certificate to whitelist the backend (El certificado raíz del certificado de servidor utilizado por el back-end no coincide con el certificado raíz de confianza que se ha agregado a Application Gateway. Asegúrese de agregar el certificado raíz correcto para incluir en la lista de elementos permitidos en el back-end).

**Causa:** SSL de un extremo a otro con Application Gateway v2 requiere que se compruebe el certificado del servidor back-end para que el estado del servidor se considere correcto.
Para que un certificado TLS o SSL sea de confianza, el certificado del servidor back-end debe haberlo emitido una entidad de certificación incluida en el almacén de confianza de Application Gateway. Si el certificado no lo ha emitido una entidad de certificación de confianza (por ejemplo, se usó un certificado autofirmado), los usuarios deben cargar el certificado del emisor en Application Gateway.

El certificado que se ha cargado en la configuración HTTP de Application Gateway debe coincidir con el certificado raíz del certificado del servidor back-end.

**Solución:** si recibe este mensaje de error, significa que hay una discrepancia entre el certificado que se ha cargado en Application Gateway y el que se cargó en el servidor back-end.

Siga los pasos del 1 al 11 del método anterior para cargar el certificado raíz de confianza correcto en Application Gateway.

Para más información sobre cómo extraer y cargar certificados raíz de confianza en Application Gateway, consulte [Exportación del certificado raíz de confianza (para SKU V2) ](https://docs.microsoft.com/azure/application-gateway/certificates-for-backend-authentication#export-trusted-root-certificate-for-v2-sku).
> [!NOTE]
> Este error también podría producirse si el servidor back-end no intercambia la cadena completa del certificado, incluido el elemento Raíz > Intermedio (si es aplicable) > Hoja durante el protocolo de enlace de TLS. Para comprobarlo, puede usar los comandos de OpenSSL desde cualquier cliente y conectarse al servidor back-end con los valores configurados en el sondeo de Application Gateway.

Por ejemplo:
```
OpenSSL> s_client -connect 10.0.0.4:443 -servername www.example.com -showcerts
```
Si la salida no muestra la cadena completa del certificado que se devuelve, vuelva a exportar el certificado con la cadena completa, incluido el certificado raíz. Configure ese certificado en el servidor back-end. 

```
  CONNECTED(00000188)\
  depth=0 OU = Domain Control Validated, CN = \*.example.com\
  verify error:num=20:unable to get local issuer certificate\
  verify return:1\
  depth=0 OU = Domain Control Validated, CN = \*.example.com\
  verify error:num=21:unable to verify the first certificate\
  verify return:1\
  \-\-\-\
  Certificate chain\
   0 s:/OU=Domain Control Validated/CN=*.example.com\
     i:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc./OU=http://certs.godaddy.com/repository//CN=Go Daddy Secure Certificate Authority - G2\
  \-----BEGIN CERTIFICATE-----\
  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\
  \-----END CERTIFICATE-----
```

#### <a name="backend-certificate-invalid-common-name-cn"></a>Nombre común (CN) del certificado de back-end no válido

**Mensaje**: The Common Name (CN) of the backend certificate does not match the host header of the probe. [El nombre común (CN) del certificado de back-end no coincide con el encabezado de host del sondeo].

**Causa:** Application Gateway comprueba si el nombre de host especificado en la configuración HTTP del back-end coincide con el del nombre común (CN) presentado por el certificado TLS o SSL del servidor back-end. Este es un comportamiento de la SKU Standard_v2 y WAF_v2 (versión 2). La Indicación de nombre de servidor (SNI) de la SKU Standard y WAF (versión 1) se establece como el nombre de dominio completo en la dirección del grupo back-end. Para obtener más información sobre el comportamiento de SNI y las diferencias entre las versiones 1 y 2 de la SKU, consulte la página de [información general sobre la terminación TLS y la TLS de extremo a extremo con Application Gateway](ssl-overview.md).

En la SKU v2, si hay un sondeo predeterminado (no se ha configurado ni asociado un sondeo personalizado), la SNI se establecerá a partir del nombre de host mencionado en la configuración HTTP. O bien, si se menciona "Seleccionar el nombre de host de la dirección de back-end" en la configuración HTTP, donde el grupo de direcciones de back-end contiene un nombre de dominio completo válido, se aplicará esta configuración.

Si hay un sondeo personalizado asociado a la configuración HTTP, la SNI se establecerá a partir del nombre de host mencionado en la configuración de sondeo personalizada. O bien, si elige **Seleccionar el nombre de host de la configuración de HTTP** en el sondeo personalizado, la SNI se establecerá a partir del nombre de host mencionado en la configuración HTTP.

En caso de que la opción para **elegir el nombre de host desde la dirección de back-end** se establezca en la configuración HTTP, el grupo de direcciones de back-end debe contener un nombre de dominio completo válido.

Si recibe este mensaje de error, significa que el CN del certificado de back-end no coincide con el nombre de host configurado en el sondeo personalizado o en la configuración HTTP (si se elige **Seleccionar el nombre de host de la configuración de HTTP**). Si usa un sondeo predeterminado, el nombre de host se establecerá como **127.0.0.1**. Si no desea ese valor, debe crear un sondeo personalizado y asociarlo a la configuración HTTP.

**Solución:**

Para solucionar el problema, siga estos pasos.

Para Windows:

1.  Inicie sesión en la máquina donde se hospeda la aplicación.

1.  Seleccione Win+R o haga clic con el botón derecho en el botón **Inicio** y seleccione **Ejecutar**.

1.  Escriba **certmgr.msc** y seleccione Entrar. También puede buscar Administrador de certificados en el menú **Inicio**.

1.  Busque el certificado (normalmente en `\Certificates - Current User\\Personal\\Certificates`) y ábralo.

1.  En la pestaña **Detalles**, compruebe el **asunto** del certificado.

1.  Compruebe el CN del certificado en los detalles y escriba el mismo en el campo de nombre de host del sondeo personalizado o en la configuración HTTP (si se elige **Seleccionar el nombre de host de la configuración de HTTP**). Si ese no es el nombre de host deseado para el sitio web, debe obtener un certificado para ese dominio o escribir el nombre de host correcto en la configuración del sondeo personalizado o de HTTP.

En Linux con OpenSSL:

1.  Ejecute este comando en OpenSSL:
    ```
    openssl x509 -in certificate.crt -text -noout
    ```

2.  En las propiedades mostradas, busque el CN del certificado y escriba el mismo en el campo Nombre de host de la configuración HTTP. Si ese no es el nombre de host deseado para el sitio web, debe obtener un certificado para ese dominio o escribir el nombre de host correcto en la configuración del sondeo personalizado o de HTTP.

#### <a name="backend-certificate-is-invalid"></a>El certificado de back-end no es válido.

**Mensaje**: Backend certificate is invalid. (El certificado de back-end no es válido). La fecha actual está dentro del intervalo de fechas \"Válido desde\" y \"Válido hasta\" en el certificado.

**Causa:** cada certificado viene con un intervalo de validez y la conexión HTTPS no será segura a menos que el certificado TLS o SSL del servidor sea válido. Los datos actuales deben estar dentro del intervalo **válido desde** y **válido hasta**. Si no es así, el certificado se considera no válido y se creará una incidencia de seguridad en la que Application Gateway marca el servidor back-end como incorrecto.

**Solución:** si el certificado TLS o SSL ha expirado, renuévelo con el proveedor y actualice la configuración del servidor con el nuevo certificado. Si se trata de un certificado autofirmado, debe generar un certificado válido y cargar el certificado raíz en la configuración HTTP de Application Gateway. Para ello, sigue estos pasos:

1.  Abra la configuración HTTP de Application Gateway en el portal.

1.  Seleccione la configuración que tenga el certificado expirado, seleccione **Agregar certificado** y abra el nuevo archivo de certificado.

1.  Elimine el certificado antiguo con el icono **Eliminar** situado junto a él y, luego, seleccione **Guardar**.

#### <a name="certificate-verification-failed"></a>Error en la verificación del certificado

**Mensaje**: The validity of the backend certificate could not be verified. Para averiguar el motivo, compruebe el diagnóstico de OpenSSL para el mensaje asociado al código de error {errorCode}.

**Causa:** este error se produce cuando Application Gateway no puede comprobar la validez del certificado.

**Solución:** para resolver este problema, compruebe que el certificado del servidor se ha creado correctamente. Por ejemplo, puede usar [OpenSSL](https://www.openssl.org/docs/man1.0.2/man1/verify.html) para comprobar el certificado y sus propiedades e intentar volver a cargarlo en la configuración HTTP de Application Gateway.

<a name="backend-health-status-unknown"></a>Estado de mantenimiento del back-end: desconocido
-------------------------------
Si el estado del back-end se muestra como desconocido, la vista del portal se parecerá a la siguiente captura de pantalla:

![Estado del back-end de Application Gateway: desconocido](./media/application-gateway-backend-health-troubleshooting/appgwunknown.png)

Este comportamiento puede producirse por uno o varios de los siguientes motivos:

1.  El grupo de seguridad de red de la subred de Application Gateway está bloqueando el acceso de entrada a los puertos 65503-65534 (SKU v1) o 65200-65535 (SKU v2) desde "Internet".
1.  El UDR de la subred de Application Gateway está establecido en la ruta predeterminada (0.0.0.0/0) y el próximo salto no se especifica como "Internet".
1.  La ruta predeterminada se anuncia mediante una conexión de ExpressRoute o VPN a una red virtual a través de BGP.
1.  El servidor DNS personalizado está configurado en una red virtual que no puede resolver nombres de dominio públicos.
1.  Application Gateway está en un estado incorrecto.

**Solución:**

1.  compruebe si el grupo de seguridad de red está bloqueando el acceso a los puertos 65503-65534 (SKU v1) o 65200-65535 (SKU v2) desde **Internet**:

    a.  En la pestaña **Overview** (Información general) de Application Gateway, seleccione el vínculo **Virtual Network/Subnet** (Red virtual/subred).

    b.  En la pestaña **Subnets** (Subredes) de la red virtual, seleccione la subred en la que se ha implementado Application Gateway.

    c.  Compruebe si hay un grupo de seguridad de red configurado.

    d.  Si se ha configurado un grupo de seguridad de red, busque ese recurso de grupo de seguridad de red en la pestaña **Search** (Buscar) o en **All resources** (Todos los recursos).

    e.  En la sección **Inbound Rules** (Reglas de entrada), agregue una regla de entrada para permitir el intervalo de puertos de destino 65503-65534 para la SKU v1 o 65200-65535 para SKU v2 con **Source** (Origen) como **Any** (Cualquiera) o **Internet**.

    f.  Seleccione **Save** (Guardar) y compruebe que puede ver el back-end como correcto. Como alternativa, puede hacerlo en [PowerShell o la CLI](https://docs.microsoft.com/azure/virtual-network/manage-network-security-group).

1.  Compruebe si el UDR tiene una ruta predeterminada (0.0.0.0/0) donde el próximo salto no esté establecido como **Internet**:
    
    a.  Siga los pasos 1a y 1b para determinar la subred.

    b.  Compruebe si hay algún UDR configurado. Si lo hay, busque el recurso en la barra de búsqueda o en **All resources** (Todos los recursos).

    c.  Compruebe si hay rutas predeterminadas (0.0.0.0/0) donde el próximo salto no esté establecido como **Internet**. Si el valor es **Virtual Appliance** (Aplicación virtual) o **Virtual Network Gateway** (Puerta de enlace de red virtual), debe asegurarse de que la aplicación virtual o el dispositivo local puedan enrutar correctamente el paquete de vuelta al destino de Internet sin modificarlo.

    d.  En caso contrario, cambie el próximo salto a **Internet**, seleccione **Guardar** y compruebe el mantenimiento del back-end.

1.  La ruta predeterminada anunciada por la conexión ExpressRoute o VPN a la red virtual a través de BGP:

    a.  Si tiene una conexión ExpressRoute o VPN a la red virtual a través de BGP y está anunciando una ruta predeterminada, debe asegurarse de que el paquete se redirige de vuelta al destino de Internet sin modificarlo. Para comprobarlo, use la opción **Connection Troubleshoot** (Solución de problemas de conexión) en el portal de Application Gateway.

    b.  Elija el destino manualmente, que será cualquier dirección IP enrutable a Internet como 1.1.1.1. Establezca el puerto de destino en cualquier cosa y compruebe la conectividad.

    c.  Si el próximo salto es una puerta de enlace de red virtual, es posible que haya una ruta predeterminada anunciada a través de ExpressRoute o VPN.

1.  Si hay un servidor DNS personalizado configurado en la red virtual, compruebe que el servidor (o servidores) puede resolver los dominios públicos. La resolución de nombres de dominios públicos podría ser necesaria en escenarios en los que Application Gateway deba ponerse en contacto con dominios externos como los servidores OCSP o para comprobar el estado de revocación del certificado.

1.  Para comprobar que Application Gateway está en un estado correcto y en ejecución, vaya a la opción **Resource Health** (Estado de los recursos) en el portal y compruebe si es **Healthy** (Correcto). Si ve el estado **Unhealthy** (Incorrecto) o **Degraded** (Degradado), [póngase en contacto con el soporte técnico](https://azure.microsoft.com/support/options/).

<a name="next-steps"></a>Pasos siguientes
----------

Más información sobre el [registro y el diagnóstico de Application Gateway](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics).
