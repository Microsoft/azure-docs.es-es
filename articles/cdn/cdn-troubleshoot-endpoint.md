---
title: 'Solución de problemas de puntos de conexión de Azure CDN: código de estado 404'
description: Aprenda a solucionar problemas relacionados con los puntos de conexión de Azure Content Delivery Network (CDN) que devuelven códigos de estado de respuesta HTTP 404.
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: b588a1eb-ab69-4fc7-ae4d-157c3e46f4a8
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: d6ad0b8b37bd4f04c22ed52d4ac6717202f22889
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192486"
---
# <a name="troubleshooting-azure-cdn-endpoints-that-return-a-404-status-code"></a>Solución de problemas de puntos de conexión de Azure CDN que devuelven un código de estado 404
La información de este artículo le permite solucionar problemas relacionados con los puntos de conexión de Azure Content Delivery Network (CDN) que devuelven códigos de estado de respuesta HTTP 404.

Si necesita más ayuda en cualquier momento con este artículo, puede ponerse en contacto con los expertos de Azure en [los foros de MSDN Azure y de desbordamiento de pila](https://azure.microsoft.com/support/forums/). Como alternativa, también puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Soporte técnico**.

## <a name="symptom"></a>Síntoma
Ha creado un perfil de red CDN y un punto de conexión, pero el contenido no parece estar disponible en la red CDN. Los usuarios que intentan acceder a su contenido a través de la dirección URL de CDN reciben un código de estado HTTP 404. 

## <a name="cause"></a>Causa
Hay varias causas posibles, por nombrar algunas:

* El origen del archivo no es visible para la red CDN.
* El punto de conexión no está configurado correctamente, lo cual provoca que la red CDN busque en el lugar incorrecto.
* El host rechaza el encabezado de host de la red CDN.
* El punto de conexión no ha tenido tiempo de propagarse a través de la red CDN.

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas
> [!IMPORTANT]
> Después de crear un punto de conexión de CDN, este no estará disponible para su uso de forma inmediata; el registro puede tardar en propagarse a través de la red CDN.
> - En los perfiles **Azure CDN Estándar de Microsoft**, la propagación se completa normalmente en diez minutos. 
> - En los perfiles de **Azure CDN estándar**, la propagación normalmente se completa en un minuto. 
> - En los perfiles **Azure CDN de Verizon estándar** y **Azure CDN de Verizon premium**, la propagación se completa normalmente en 90 minutos. 
> 
> Si realiza los pasos detallados en este documento y sigue obteniendo errores 404, espere unas horas para probar nuevamente antes de abrir una incidencia de soporte técnico.
> 
> 

### <a name="check-the-origin-file"></a>Comprobación del archivo de origen
En primer lugar, compruebe que el archivo para almacenar en caché está disponible en el servidor de origen y es accesible públicamente en Internet. La forma más rápida de hacerlo es abrir un explorador en una sesión en modo privado o incógnito y buscar directamente el archivo. Escriba o pegue la dirección URL en el cuadro de dirección y vea si esto da como resultado el archivo esperado. Por ejemplo, suponga que tiene un archivo en una cuenta de Azure Storage, accesible en https:\//cdndocdemo.blob.core.windows.net/publicblob/lorem.txt. Si el contenido de este archivo se puede cargar correctamente, pasa la prueba.

![¡Hecho!](./media/cdn-troubleshoot-endpoint/cdn-origin-file.png)

> [!WARNING]
> Aunque esta es la manera más rápida y sencilla de comprobar que el archivo está disponible públicamente, algunas configuraciones de red de su organización podrían dar la impresión de que este archivo está disponible públicamente cuando, en realidad, solo está visible para los usuarios de la red (incluso si está hospedado en Azure). Para asegurarse de que este no es el caso, pruebe el archivo con un explorador externo, por ejemplo, un dispositivo móvil que no esté conectado a la red de su organización o una máquina virtual en Azure.
> 
> 

### <a name="check-the-origin-settings"></a>Comprobación de la configuración de origen
Ahora que hemos comprobado que el archivo está disponible públicamente en Internet, compruebe la configuración de origen. En [Azure Portal](https://portal.azure.com), vaya al perfil de CDN y seleccione el punto de conexión cuyos problemas desea solucionar. En la página resultante **Punto de conexión**, seleccione el origen.  

![Página de punto de conexión con origen resaltado](./media/cdn-troubleshoot-endpoint/cdn-endpoint.png)

Aparece la página **Origen**. 

![Página de origen](./media/cdn-troubleshoot-endpoint/cdn-origin-settings.png)

#### <a name="origin-type-and-hostname"></a>Tipo de origen y nombre del host
Compruebe que los valores de **Tipo de origen** y **Nombre de host de origen** sean correctos. En este ejemplo, https:\//cdndocdemo.blob.core.windows.net/publicblob/lorem.txt, la parte de nombre de host de la dirección URL es *cdndocdemo.blob.core.windows.net*, lo cual es correcto. Como los orígenes Azure Storage, Web App y Servicio en la nube usan un valor de lista desplegable en el campo **Nombre de host de origen**, no existe el problema de errores de ortografía. Sin embargo, si usa un origen personalizado, asegúrese de escribir correctamente el nombre de host.

#### <a name="http-and-https-ports"></a>Puertos HTTP y HTTPS
Compruebe sus puertos **HTTP** y **HTTPS**. En la mayoría de los casos, los puertos 80 y 443 son correctos y no requerirá ningún cambio.  Sin embargo, si el servidor de origen está escuchando en un puerto diferente, este deberá estar representado aquí. Si no está seguro, vea la dirección URL del archivo de origen. Las especificaciones de HTTP y HTTPS usan los puertos 80 y 443 como valores predeterminados. En la dirección URL de ejemplo, https:\//cdndocdemo.blob.core.windows.net/publicblob/lorem.txt, no se especifica un puerto, por lo que se asume el valor predeterminado de 443 y la configuración es correcta.  

Sin embargo, suponga que la dirección URL del archivo de origen que se ha probado anteriormente es http:\//www.contoso.com:8080/file.txt. Observe la parte *:8080* al final del segmento de nombre de host. Ese número indica al explorador que use el puerto 8080 para conectarse al servidor web en www\.contoso.com, por lo tanto, deberá escribir *8080* en el campo **Puerto HTTP**. Es importante tener en cuenta que esta configuración de puerto solo afecta al puerto que usa el punto de conexión para recupera información del origen.

> [!NOTE]
> Los puntos de conexión de **Azure CDN Estándar de Akamai** no permiten el intervalo completo de puertos TCP para los orígenes.  Para obtener una lista de los puertos de origen que no se permiten, consulte [Azure CDN from Akamai Allowed Origin Ports](/previous-versions/azure/mt757337(v=azure.100))(Puertos de origen permitidos de la red CDN de Azure de Akamai).  
> 
> 

### <a name="check-the-endpoint-settings"></a>Comprobación de la configuración del punto de conexión
En la página **Punto de conexión**, seleccione el botón **Configurar**.

![Página Punto de conexión con el botón Configurar resaltado](./media/cdn-troubleshoot-endpoint/cdn-endpoint-configure-button.png)

Aparece la página **Configurar** del punto de conexión de CDN.

![Página Configurar](./media/cdn-troubleshoot-endpoint/cdn-configure.png)

#### <a name="protocols"></a>Protocolos
En **Protocolos**, compruebe que está seleccionado el protocolo utilizado por los clientes. Como el mismo protocolo utilizado por el cliente es el que se usa para acceder al origen, es importante tener configurados correctamente los puertos de origen en la sección anterior. El punto de conexión de CDN solo escucha en los puertos HTTP y HTTPS predeterminados (80 y 443), con independencia de los puertos de origen.

Volvamos al ejemplo hipotético con http:\//www.contoso.com:8080/file.txt.  Como recordará, Contoso especificó *8080* como puerto HTTP, pero supongamos también que se especificó *44300* como puerto HTTPS.  Si crearon un punto de conexión llamado *contoso*, el nombre de host de su punto de conexión de CDN sería *contoso.azureedge.net*.  Una solicitud para http:\//contoso.azureedge.net/file.txt es una solicitud HTTP, así que el punto de conexión usaría HTTP en el puerto 8080 para recuperarlo del origen.  Una solicitud segura sobre HTTPS, https:\//contoso.azureedge.net/file.txt, provocaría que el punto de conexión usara HTTPS en el puerto 44300 al recuperar el archivo del origen.

#### <a name="origin-host-header"></a>Encabezado del host de origen
El **encabezado de host de origen** es el valor del encabezado del host que se envía al origen con cada solicitud.  En la mayoría de los casos, debe ser el mismo que el **nombre de host de origen** que se comprobó anteriormente.  Un valor incorrecto en este campo normalmente no causará errores 404, pero puede provocar otros errores 4xx, dependiendo de lo que el origen espera.

#### <a name="origin-path"></a>Ruta de acceso de origen
Por último, debemos comprobar la **ruta de acceso de origen**.  De manera predeterminada esta estará en blanco.  Solo debe usar este campo si quiere restringir el ámbito de los recursos hospedados en el origen que desea que estén disponibles en la red CDN.  

En el punto de conexión de ejemplo, queremos que todos los recursos de la cuenta de almacenamiento estén disponibles, por lo que la **ruta de acceso de origen** se dejó en blanco.  Esto significa que una solicitud a https:\//cdndocdemo.azureedge.net/publicblob/lorem.txt da como resultado una conexión desde el punto de conexión a cdndocdemo.core.windows.net que solicita */publicblob/lorem.txt*.  Del mismo modo, una solicitud a https:\//cdndocdemo.azureedge.net/donotcache/status.png da lugar a que el punto de conexión solicite */donotcache/status.png* del origen.

Pero, ¿y si no quiere usar la red CDN con todas las rutas de acceso en el origen?  Supongamos que solo quisiera exponer la ruta de acceso *publicblob*.  Si escribimos */publicblob* en el campo **Origin path** (Ruta de acceso de origen), el punto de conexión insertará */publicblob* antes de realizar cada solicitud al origen.  Esto significa que la solicitud a https:\//cdndocdemo.azureedge.net/publicblob/lorem.txt ahora tomará realmente la parte de la solicitud de la dirección URL, */publicblob/lorem.txt* y anexará */publicblob* al principio. El resultado es una solicitud a */publicblob/publicblob/lorem.txt* desde el origen.  Si no se puede resolver esa ruta de acceso en un archivo real, el origen devolverá un error 404.  La dirección URL correcta para recuperar lorem.txt en este ejemplo sería realmente https:\//cdndocdemo.azureedge.net/lorem.txt.  Observe que no se incluye la ruta de acceso */publicblob*, ya que la parte de la solicitud de la dirección URL es */lorem.txt* y el punto de conexión agrega */publicblob*, lo que da lugar a que */publicblob/lorem.txt* sea la solicitud que se pasa al origen.

