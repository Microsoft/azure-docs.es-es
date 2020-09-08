---
title: 'Emparejamiento privado de Azure ExpressRoute: Configuración del modo de transporte IPsec: hosts de Windows'
description: Cómo habilitar el modo de transporte de IPsec entre máquinas virtuales Windows de Azure y los hosts locales de Windows a través del emparejamiento privado de ExpressRoute mediante GPO y unidades organizativas.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 10/17/2018
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 026b7ee6cf8061c7cff25b4f9f8d46b6ec3e6a8d
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89396495"
---
# <a name="configure-ipsec-transport-mode-for-expressroute-private-peering"></a>Configuración del modo de transporte de IPsec para emparejamiento privado de ExpressRoute

Este artículo le ayudará a crear túneles de IPsec en modo de transporte a través del emparejamiento privado de ExpressRoute entre máquinas virtuales de Azure que ejecutan Windows y hosts locales de Windows. Los pasos descritos en este artículo crean esta configuración mediante objetos de directiva de grupo. Aunque es posible crear esta configuración sin usar unidades organizativas (UO) ni objetos de directiva de grupo (GPO), la combinación de unidades organizativas y GPO le ayudará a simplificar el control de las directivas de seguridad y le permitirá escalar verticalmente de forma rápida. Según los pasos descritos en este artículo, se supone que ya tiene una configuración de Active Directory y que está familiarizado con el uso de unidades organizativas y GPO.

## <a name="about-this-configuration"></a>Acerca de esta configuración

La configuración en los pasos siguientes utiliza una única red virtual de Azure (VNet) con el emparejamiento privado de ExpressRoute. Sin embargo, esta configuración puede abarcar varias redes virtuales de Azure y redes locales. Este artículo le ayudará a definir una directiva de cifrado IPsec, así como a aplicarla a un grupo de máquinas virtuales de Azure y hosts en el entorno local que forman parte de la misma unidad organizativa. Configure el cifrado entre las máquinas virtuales de Azure (vm1 y vm2) y el host1 local solo para tráfico HTTP con el puerto de destino 8080. Se pueden crear diferentes tipos de directiva IPsec según sus requisitos.

### <a name="working-with-ous"></a>Funcionamiento con unidades organizativas 

La directiva de seguridad asociada a una unidad organizativa se inserta en los equipos a través de GPO. A continuación, se indican algunas ventajas del uso de unidades organizativas en lugar de aplicar directivas a un único host:

* La asociación de una directiva a una unidad organizativa garantiza que los equipos que pertenecen a la misma UO obtengan las mismas directivas.
* El cambio de la directiva de seguridad asociada a la unidad organizativa aplicará los cambios a todos los hosts de la UO.

### <a name="diagrams"></a>Diagramas

El siguiente diagrama muestra la interconexión y el espacio de direcciones IP asignadas. Las máquinas virtuales de Azure y el host local ejecutan Windows 2016. Las máquinas virtuales de Azure y el host1 local forman parte del mismo dominio. Las máquinas virtuales de Azure y los hosts locales pueden resolver nombres correctamente mediante DNS.

[![1]][1]

Este diagrama muestra los túneles de IPsec en tránsito en el emparejamiento privado de ExpressRoute.

[![4]][4]

### <a name="working-with-ipsec-policy"></a>Funcionamiento con directivas IPsec

En Windows, el cifrado se asocia a la directiva IPsec. La directiva IPsec determina qué tráfico IP está protegido y el mecanismo de seguridad que se aplica a los paquetes IP.
**Las directivas IPSec** están formadas por los siguientes elementos: **Listas de filtros**, **Acciones de filtrado** y **Reglas de seguridad**.

Al configurar la directiva IPsec, es importante comprender la terminología siguiente de directiva IPsec:

* **Directiva de IPsec:** una colección de reglas. Solo puede haber una directiva activa ("asignada") en un momento dado. Cada directiva puede tener una o varias reglas, que pueden estar activas simultáneamente. Se puede asignar solo una directiva IPsec activa al mismo tiempo a un equipo. Sin embargo, dentro de la directiva IPsec, puede definir varias acciones que pueden tomarse en distintas situaciones. Cada conjunto de reglas de IPsec está asociado a una lista de filtros que afecta al tipo de tráfico de red al que se aplica la regla.

* **Listas de filtros:** son la agrupación de uno o varios filtros. Una lista puede incluir varios filtros. El filtro define si la comunicación está permitida, protegida o bloqueada según los intervalos de direcciones IP, los protocolos o incluso puertos de protocolo específicos. Cada filtro coincide con un determinado conjunto de condiciones; por ejemplo, los paquetes enviados desde una subred determinada a un equipo concreto en un puerto de destino específico. Cuando las condiciones de red coinciden con uno o varios de esos filtros, se activa la lista de filtros. Cada filtro se define dentro de una lista de filtros específicos. Los filtros no se pueden compartir entre listas de filtros. Sin embargo, una lista de filtros determinada se puede incorporar a varias directivas IPsec. 

* **Acciones de filtrado:** un método de seguridad define un conjunto de algoritmos de seguridad, protocolos y la clave que un equipo ofrece durante las negociaciones de IKE. Las acciones de filtrado son listas de métodos de seguridad clasificados por orden de preferencia.  Cuando un equipo negocia una sesión de IPsec, acepta o envía propuestas en función de la configuración de seguridad almacenada en la lista de acciones de filtro.

* **Reglas de seguridad:** las reglas determinan cómo y cuándo una directiva IPsec protege la comunicación. Usa la **lista de filtros** y las **acciones de filtrado** para crear una regla de IPsec para compilar la conexión IPsec. Cada directiva puede tener una o varias reglas, que pueden estar activas simultáneamente. Cada regla contiene una lista de filtros IP y una colección de acciones de seguridad que tienen lugar cuando se produce una coincidencia con esa lista de filtros:
  * Acciones de filtrado IP
  * Métodos de autenticación
  * Configuración del túnel IP
  * Tipos de conexión

[![5]][5]

## <a name="before-you-begin"></a>Antes de empezar

Asegúrese de que se cumplen los siguientes requisitos previos:

* Debe tener una configuración funcional de Active Directory que pueda usar para implementar la configuración de directiva de grupo. Para obtener más información acerca de los GPO, consulte [Group Policy Objects](https://msdn.microsoft.com/library/windows/desktop/aa374162(v=vs.85).aspx) (Objetos de directiva de grupo).

* Tiene que tener un circuito ExpressRoute activo.
  * Para más información acerca de cómo configurar un circuito ExpressRoute, consulte [Creación de un circuito ExpressRoute](expressroute-howto-circuit-arm.md). 
  * Compruebe que el proveedor de conectividad ha habilitado el circuito. 
  * Compruebe que dispone de un emparejamiento privado de Azure configurado para el circuito. Consulte el artículo de [configuración del enrutamiento](expressroute-howto-routing-arm.md) para obtener instrucciones sobre el enrutamiento. 
  * Compruebe que ha creado y aprovisionado totalmente una red virtual y una puerta de enlace de red virtual. Siga las instrucciones para [crear una puerta de enlace de red virtual en ExpressRoute](expressroute-howto-add-gateway-resource-manager.md). Una puerta de enlace de red virtual para ExpressRoute usa el GatewayType "ExpressRoute", no VPN.

* La puerta de enlace de red virtual de ExpressRoute debe estar conectada al circuito ExpressRoute. Para obtener más información, consulte [Conexión de una red virtual a un circuito ExpressRoute](expressroute-howto-linkvnet-arm.md).

* Compruebe que las máquinas virtuales Windows de Azure se implementan en la red virtual.

* Compruebe que hay conectividad entre los hosts locales y las máquinas virtuales de Azure.

* Compruebe que las máquinas virtuales Windows de Azure y los hosts locales pueden usar DNS para resolver los nombres correctamente.

### <a name="workflow"></a>Flujo de trabajo

1. Cree un GPO y asócielo a la unidad organizativa.
2. Defina una **acción de filtrado** de IPsec.
3. Defina una **lista de filtros** de IPsec.
4. Cree una directiva IPsec con **reglas de seguridad**.
5. Asigne el GPO de IPsec a la unidad organizativa.

### <a name="example-values"></a>Valores de ejemplo

* **Nombre de dominio:** ipsectest.com

* **Unidad organizativa:** IPSecOU

* **Equipo de Windows local:** host1

* **Máquinas virtuales Windows de Azure:** vm1, vm2

## <a name="1-create-a-gpo"></a><a name="creategpo"></a>1. Creación de un GPO

1. Para crear un nuevo GPO vinculado a una unidad organizativa, abra el complemento de administración de directivas de grupo y busque la unidad organizativa a la que se vinculará el GPO. En el ejemplo, la unidad organizativa se denomina **IPSecOU**. 

   [![9]][9]
2. En el complemento de administración de directivas de grupo, seleccione la unidad organizativa y haga clic en ella con el botón derecho. En la lista desplegable, haga clic en "**Crear un GPO en este dominio y vincularlo aquí**".

   [![10]][10]
3. Asigne un nombre intuitivo al GPO para poder localizarlo fácilmente más tarde. Haga clic en **Aceptar** para crear y vincular el GPO.

   [![11]][11]

## <a name="2-enable-the-gpo-link"></a><a name="enablelink"></a>2. Habilitación del vínculo de GPO

Para aplicar el GPO a la unidad organizativa, no solo se debe vincular el GPO a esta, sino que también se debe habilitar el vínculo.

1. Busque el GPO que ha creado, haga clic en él con el botón derecho y seleccione **Editar** en la lista desplegable.
2. Para aplicar el GPO a la unidad organizativa, seleccione **Vínculo habilitado**.

   [![12]][12]

## <a name="3-define-the-ip-filter-action"></a><a name="filteraction"></a>3. Definición de la acción de filtrado IP

1. En la lista desplegable, haga clic en **Directiva de seguridad de IP en Active Directory** y, a continuación, haga clic en **Administrar listas de filtros IP y acciones de filtrado...** .

   [![15]][15]
2. En la pestaña "**Administrar acciones de filtrado**", haga clic en **Agregar**.

   [![16]][16]

3. En el **Asistente para acciones de filtrado de seguridad IP**, haga clic en **Siguiente**.

   [![17]][17]
4. Asigne un nombre intuitivo a la acción de filtrado para poder encontrarla más tarde. En este ejemplo, la acción de filtrado se denomina **myEncryption**. También puede agregar una descripción. A continuación, haga clic en **Siguiente**.

   [![18]][18]
5. **Negociar la seguridad** le permite definir el comportamiento si IPsec no se puede establecer con otro equipo. Seleccione **Negociar la seguridad** y, a continuación, haga clic en **Siguiente**.

   [![19]][19]
6. En la página **Comunicación con equipos no compatibles con IPsec**, seleccione **No permitir comunicación no segura** y, a continuación, haga clic en **Siguiente**.

   [![20]][20]
7. En la página **Tráfico IP y seguridad**, seleccione **Personalizado** y, a continuación, haga clic en **Configuración...** .

   [![21]][21]
8. En la página **Configuración de método de seguridad personalizado**, seleccione **Integridad y cifrado de datos (ESP): SHA1, 3DES**. A continuación, haga clic en **Aceptar**.

   [![22]][22]
9. En la página **Administrar acciones de filtrado**, puede ver que el filtro **myEncryption** se agregó correctamente. Haga clic en **Cerrar**.

   [![23]][23]

## <a name="4-define-an-ip-filter-list"></a><a name="filterlist1"></a>4. Definición de una lista de filtros IP

Cree una lista de filtros que especifique el tráfico HTTP cifrado con el puerto de destino 8080.

1. Para calificar qué tipos de tráfico deben cifrarse, utilice una **lista de filtros IP**. En la pestaña **Administrar listas de filtros IP**, haga clic en **Agregar** para agregar una nueva lista de filtros IP.

   [![24]][24]
2. En el campo **Nombre:** , escriba un nombre para la lista de filtros IP. Por ejemplo, **azure-onpremises-HTTP8080**. A continuación, haga clic en **Agregar**.

   [![25]][25]
3. En la página **Descripción de filtro IP y propiedad reflejada**, seleccione **Reflejado**. La configuración reflejada coincide con los paquetes en ambas direcciones, lo que permite la comunicación bidireccional. A continuación, haga clic en **Siguiente**.

   [![26]][26]
4. En el página **Origen del tráfico IP**, en la lista desplegable **Dirección de origen:** , elija **Una dirección IP o subred específica**. 

   [![27]][27]
5. Especifique la dirección de origen **Dirección IP o subred:** del tráfico IP y, a continuación, haga clic en **Siguiente**.

   [![28]][28]
6. Especifique la **Dirección de destino:** dirección IP o subred. A continuación, haga clic en **Siguiente**.

   [![29]][29]
7. En la página **Tipo de protocolo IP**, seleccione **TCP**. A continuación, haga clic en **Siguiente**.

   [![30]][30]
8. En la página **Puerto de protocolo IP**, seleccione **Desde cualquier puerto** y **A este puerto:** . Escriba **8080** en el cuadro de texto. Esta configuración especifica que solo se cifrará el tráfico HTTP en el puerto de destino 8080. A continuación, haga clic en **Siguiente**.

   [![31]][31]
9. Consulte la lista de filtros IP.  La configuración de la lista de filtros IP **azure-onpremises-HTTP8080** desencadena el cifrado de todo el tráfico que coincide con los criterios siguientes:

   * Cualquier dirección de origen en 10.0.1.0/24 (subred2 de Azure)
   * Cualquier dirección de destino en 10.2.27.0/25 (subred local)
   * Protocolo TCP
   * Puerto de destino 8080

   [![32]][32]

## <a name="5-edit-the-ip-filter-list"></a><a name="filterlist2"></a>5. Edición de la lista de filtros IP

Para cifrar el mismo tipo de tráfico en dirección opuesta (desde el host local a la máquina virtual de Azure), es necesario un segundo filtro IP. El proceso de configuración del nuevo filtro es el mismo proceso que el usado para configurar el primer filtro IP. Las únicas diferencias son la subred de origen y de destino.

1. Para agregar un nuevo filtro IP a la lista de filtros IP, seleccione **Editar**.

   [![33]][33]
2. En la página **Lista de filtros IP**, haga clic en **Agregar**.

   [![34]][34]
3. Cree un segundo filtro IP mediante la configuración del ejemplo siguiente:

   [![35]][35]
4. Después de crear el segundo filtro IP, la lista de filtros IP tendrá el aspecto siguiente:

   [![36]][36]

Si el cifrado es necesario entre una ubicación local y una subred de Azure para proteger una aplicación, en lugar de modificar la lista de filtros IP existente, puede agregar una nueva lista de filtros IP en su lugar. La asociación de dos listas de filtros IP a la misma directiva IPsec proporciona una mayor flexibilidad porque una lista de filtros IP específica se puede modificar o quitar en cualquier momento sin que afecte a otras listas de filtros IP.

## <a name="6-create-an-ipsec-security-policy"></a><a name="ipsecpolicy"></a>6. Creación de una directiva de seguridad IPsec 

Cree una directiva IPsec con reglas de seguridad.

1. Seleccione la opción **Políticas de seguridad IP en Active Directory** asociada a la unidad organizativa. Haga clic con el botón derecho y seleccione **Crear directiva de seguridad IP**.

   [![37]][37]
2. Asigne un nombre a la directiva de seguridad. Por ejemplo, **policy-azure-onpremises**. A continuación, haga clic en **Siguiente**.

   [![38]][38]
3. Haga clic en **Siguiente** sin seleccionar la casilla.

   [![39]][39]
4. Compruebe que la casilla **Editar propiedades** está seleccionada y, a continuación, haga clic en **Finalizar**.

   [![40]][40]

## <a name="7-edit-the-ipsec-security-policy"></a><a name="editipsec"></a>7. Edición de la directiva de seguridad IPsec

Agregue a la directiva IPsec la **lista de filtros IP** y la **acción de filtrado** que configuró previamente.

1. En la pestaña **Reglas** de las propiedades de la política HTTP, haga clic en **Agregar**.

   [![41]][41]
2. En la página de bienvenida, haga clic en **Siguiente**.

   [![42]][42]
3. Una regla proporciona la opción para definir el modo IPsec: modo de túnel o modo de transporte.

   * En el modo de túnel, el paquete original se encapsula mediante un conjunto de encabezados IP. Dicho modo protege la información de enrutamiento interna al cifrar el encabezado IP del paquete original. El modo de túnel está ampliamente implementado entre las puertas de enlace en escenarios VPN de sitio a sitio. En la mayoría de casos, este modo se usa para el cifrado de un extremo a otro entre los hosts.

   * El modo de transporte solo cifra la carga útil y el finalizador ESP; en cambio, el encabezado IP del paquete original no se cifra. En el modo de transporte, el origen IP y el destino IP de los paquetes no se modifican.

   Seleccione **Esta regla no especifica un túnel** y, a continuación, haga clic en **Siguiente**.

   [![43]][43]
4. **Tipo de red** define qué conexión de red se asocia a la directiva de seguridad. Seleccione **Todas las conexiones de red** y, a continuación, haga clic en **Siguiente**.

   [![44]][44]
5. Seleccione la lista de filtros IP que creó anteriormente, **azure-onpremises-HTTP8080** y, a continuación, haga clic en **Siguiente**.

   [![45]][45]
6. Seleccione la acción de filtrado **myEncryption** existente que creó anteriormente.

   [![46]][46]
7. Windows admite cuatro tipos distintos de autenticaciones: Kerberos, certificados, NTLMv2 y clave previamente compartida. Dado que trabajamos con hosts unidos a un dominio, seleccione **Valor predeterminado de Active Directory (protocolo Kerberos V5)** y, a continuación, haga clic en **Siguiente**.

   [![47]][47]
8. La nueva directiva crea la regla de seguridad: **azure-onpremises-HTTP8080**. Haga clic en **OK**.

   [![48]][48]

La directiva IPsec requiere que todas las conexiones HTTP en el puerto de destino 8080 utilicen el modo de transporte de IPsec. Dado que HTTP es un protocolo de texto sin cifrado, tener habilitada la directiva de seguridad garantiza que los datos se cifren cuando se transfieren a través del emparejamiento privado de ExpressRoute. La directiva de seguridad IP para Active Directory es más difícil de configurar que Firewall de Windows con seguridad avanzada, pero permite más personalización de la conexión IPsec.

## <a name="8-assign-the-ipsec-gpo-to-the-ou"></a><a name="assigngpo"></a>8. Asignación del GPO de IPsec a la unidad organizativa

1. Consulte la directiva. La directiva de grupo de seguridad está definida, pero aún no se ha asignado.

   [![49]][49]
2. Para asignar la directiva de grupo de seguridad a la unidad organizativa **IPSecOU**, haga clic con el botón derecho en la directiva de seguridad y elija **Asignar**.
   Cada equipo que pertenezca a la unidad organizativa tendrá asignada la directiva de grupo de seguridad.

   [![50]][50]

## <a name="check-traffic-encryption"></a><a name="checktraffic"></a>Comprobación del cifrado de tráfico

Para comprobar el GPO de cifrado aplicado en la unidad organizativa, instale IIS en todas las máquinas virtuales de Azure y en el host1. Cada IIS se personaliza para responder a solicitudes HTTP en el puerto 8080.
Para comprobar el cifrado, puede instalar un analizador de protocolos (sniffer) de red (por ejemplo, Wireshark) en todos los equipos de la unidad organizativa.
Un script de PowerShell funciona como un cliente HTTP para generar solicitudes HTTP en el puerto 8080:

```powershell
$url = "http://10.0.1.20:8080"
while ($true) {
try {
[net.httpWebRequest]
$req = [net.webRequest]::create($url)
$req.method = "GET"
$req.ContentType = "application/x-www-form-urlencoded"
$req.TimeOut = 60000

$start = get-date
[net.httpWebResponse] $res = $req.getResponse()
$timetaken = ((get-date) - $start).TotalMilliseconds

Write-Output $res.Content
Write-Output ("{0} {1} {2}" -f (get-date), $res.StatusCode.value__, $timetaken)
$req = $null
$res.Close()
$res = $null
} catch [Exception] {
Write-Output ("{0} {1}" -f (get-date), $_.ToString())
}
$req = $null

# uncomment the line below and change the wait time to add a pause between requests
#Start-Sleep -Seconds 1
}

```
La siguiente captura de red muestra que los resultados de host1 local con filtro ESP para mostrar coinciden solo con el tráfico cifrado:

[![51]][51]

Si ejecuta el script de PowerShell en el entorno local (cliente HTTP), la captura de red de la máquina virtual de Azure mostrará un seguimiento similar.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de ExpressRoute, consulte [P+F de ExpressRoute](expressroute-faqs.md).

<!--Image References-->

[1]: ./media/expressroute-howto-ipsec-transport-private-windows/network-diagram.png "Modo de transporte IPsec de diagrama de red a través de ExpressRoute"
[4]: ./media/expressroute-howto-ipsec-transport-private-windows/ipsec-interesting-traffic.png "Tráfico IPsec interesante"
[5]: ./media/expressroute-howto-ipsec-transport-private-windows/windows-ipsec.png "Directiva IPsec de Windows"
[9]: ./media/expressroute-howto-ipsec-transport-private-windows/ou.png "Unidad organizativa de la directiva de grupo"
[10]: ./media/expressroute-howto-ipsec-transport-private-windows/create-gpo-ou.png "Creación de un GPO asociado a la unidad organizativa"
[11]: ./media/expressroute-howto-ipsec-transport-private-windows/gpo-name.png "Asignación de un nombre al GPO asociado a la unidad organizativa"
[12]: ./media/expressroute-howto-ipsec-transport-private-windows/edit-gpo.png "Edición del GPO"
[15]: ./media/expressroute-howto-ipsec-transport-private-windows/manage-ip-filter-list-filter-actions.png "Administrar listas de filtros IP y acciones de filtrado"
[16]: ./media/expressroute-howto-ipsec-transport-private-windows/add-filter-action.png "Agregar acción de filtrado"
[17]: ./media/expressroute-howto-ipsec-transport-private-windows/action-wizard.png "Asistente para acciones"
[18]: ./media/expressroute-howto-ipsec-transport-private-windows/filter-action-name.png "Nombre de la acción de filtrado"
[19]: ./media/expressroute-howto-ipsec-transport-private-windows/filter-action.png "Acción de filtrado"
[20]: ./media/expressroute-howto-ipsec-transport-private-windows/filter-action-no-ipsec.png "Se ha establecido la especificación del comportamiento en una conexión insegura"
[21]: ./media/expressroute-howto-ipsec-transport-private-windows/security-method.png "Mecanismo de seguridad"
[22]: ./media/expressroute-howto-ipsec-transport-private-windows/custom-security-method.png "Método de seguridad personalizado"
[23]: ./media/expressroute-howto-ipsec-transport-private-windows/filter-actions-list.png "Lista de filtros de acción"
[24]: ./media/expressroute-howto-ipsec-transport-private-windows/add-new-ip-filter.png "Agregar una nueva lista de filtros IP"
[25]: ./media/expressroute-howto-ipsec-transport-private-windows/ip-filter-http-traffic.png "Agregar tráfico HTTP al filtro IP"
[26]: ./media/expressroute-howto-ipsec-transport-private-windows/match-both-direction.png "Coincidencia de paquetes en ambas direcciones"
[27]: ./media/expressroute-howto-ipsec-transport-private-windows/source-address.png "Selección de la subred de origen"
[28]: ./media/expressroute-howto-ipsec-transport-private-windows/source-network.png "Red de origen"
[29]: ./media/expressroute-howto-ipsec-transport-private-windows/destination-network.png "Red de destino"
[30]: ./media/expressroute-howto-ipsec-transport-private-windows/protocol.png "Protocolo"
[31]: ./media/expressroute-howto-ipsec-transport-private-windows/source-port-and-destination-port.png "Puerto de origen y destino"
[32]: ./media/expressroute-howto-ipsec-transport-private-windows/ip-filter-list.png "Lista de filtros"
[33]: ./media/expressroute-howto-ipsec-transport-private-windows/ip-filter-for-http.png "Lista de filtros IP con tráfico HTTP"
[34]: ./media/expressroute-howto-ipsec-transport-private-windows/ip-filter-add-second-entry.png "Adición de un segundo filtro IP"
[35]: ./media/expressroute-howto-ipsec-transport-private-windows/ip-filter-second-entry.png "Segunda entrada de lista de filtros IP"
[36]: ./media/expressroute-howto-ipsec-transport-private-windows/ip-filter-list-2entries.png "Segunda entrada de lista de filtros IP"
[37]: ./media/expressroute-howto-ipsec-transport-private-windows/create-ip-security-policy.png "Creación de la directiva de seguridad IP"
[38]: ./media/expressroute-howto-ipsec-transport-private-windows/ipsec-policy-name.png "Nombre de la directiva IPsec"
[39]: ./media/expressroute-howto-ipsec-transport-private-windows/security-policy-wizard.png "Asistente para directivas IPsec"
[40]: ./media/expressroute-howto-ipsec-transport-private-windows/edit-security-policy.png "Edición de la directiva IPsec"
[41]: ./media/expressroute-howto-ipsec-transport-private-windows/add-new-rule.png "Agregar nueva regla de seguridad a la directiva IPsec"
[42]: ./media/expressroute-howto-ipsec-transport-private-windows/create-security-rule.png "Crear una nueva regla de seguridad"
[43]: ./media/expressroute-howto-ipsec-transport-private-windows/transport-mode.png "Modo de transporte"
[44]: ./media/expressroute-howto-ipsec-transport-private-windows/network-type.png "Tipo de red"
[45]: ./media/expressroute-howto-ipsec-transport-private-windows/selection-filter-list.png "Selección de la lista de filtros IP existente"
[46]: ./media/expressroute-howto-ipsec-transport-private-windows/selection-filter-action.png "Selección de la acción de filtrado existente"
[47]: ./media/expressroute-howto-ipsec-transport-private-windows/authentication-method.png "Selección del método de autenticación"
[48]: ./media/expressroute-howto-ipsec-transport-private-windows/security-policy-completed.png "Final del proceso de creación de la directiva de seguridad"
[49]: ./media/expressroute-howto-ipsec-transport-private-windows/gpo-not-assigned.png "Política IPsec vinculada al GPO, pero sin asignar"
[50]: ./media/expressroute-howto-ipsec-transport-private-windows/gpo-assigned.png "Política IPsec asignada al GPO"
[51]: ./media/expressroute-howto-ipsec-transport-private-windows/encrypted-traffic.png "Captura del tráfico cifrado de IPsec"
