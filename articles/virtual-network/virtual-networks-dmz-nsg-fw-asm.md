---
title: 'Ejemplo de red perimetral: compilación de una red perimetral para proteger las aplicaciones con un firewall y los NSG | Microsoft Docs'
description: Crear una red perimetral con un firewall y grupos de seguridad de red (NSG)
services: virtual-network
documentationcenter: na
author: tracsman
manager: rossort
editor: ''
ms.assetid: c78491c7-54ac-4469-851c-b35bfed0f528
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/01/2016
ms.author: jonor;sivae
ms.openlocfilehash: c1c64945aaa0bc4cd83cc769dab1c2a755896c01
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65603431"
---
# <a name="example-2-build-a-perimeter-network-to-protect-applications-with-a-firewall-and-nsgs"></a>Ejemplo 2: Creación de una red perimetral para proteger las aplicaciones con un firewall y los NSG
[Volver a la página de seguridad de red y servicios en la nube de Microsoft][HOME]

En este ejemplo se muestra cómo crear una red perimetral (también conocido como *DMZ*, *zona desmilitarizada*, y *subred filtrada*) con un firewall, cuatro equipos de Windows Server, Grupos de seguridad de red (NSG) y. Incluye detalles sobre cada uno de los comandos pertinentes para proporcionar una comprensión más profunda de cada paso. La sección "Escenarios de tráfico" proporciona una explicación paso a paso de cómo pasa el tráfico a través de las capas de defensa de la red perimetral. Por último, la sección "Referencias" proporciona el código completo e instrucciones sobre cómo crear este entorno para probar y experimentar con diferentes escenarios.

![Red perimetral de entrada con NVA y NSG][1]

## <a name="environment"></a>Entorno 
En este ejemplo se basa en un escenario con una suscripción de Azure que contiene estos elementos:

* Dos servicios en la nube: FrontEnd001 y BackEnd001.
* Una red virtual denominada CorpNetwork que tiene dos subredes: Front-end y back-end.
* Un NSG único que se aplica a ambas subredes.
* Una aplicación virtual de red: Un Firewall Barracuda NextGen conectado a la subred de front-end.
* Un equipo con Windows Server que representa un servidor de aplicaciones web: IIS01.
* Dos equipos con Windows Server que representan servidores back-end de aplicación: AppVM01 y AppVM02.
* Un equipo con Windows Server que representa un servidor DNS: DNS01.

> [!NOTE]
> Aunque este ejemplo usa un Firewall Barracuda NextGen, podrían usarse muchos dispositivos de red virtual.
> 
> 

El script de PowerShell en la sección "Referencias" de este artículo basa la mayor parte del entorno descrito aquí. Las máquinas virtuales y redes virtuales se crean mediante la secuencia de comandos, pero estos procesos no se describen en detalle en este documento.

Para crear el entorno, siga estos pasos:

1. Guarde el archivo XML de configuración de red incluido en la sección "Referencias" (actualizado con los nombres, ubicaciones y IP direcciones para que coincida con su escenario).
2. Actualice las variables definidas por el usuario en el script de PowerShell para que coincida con el entorno de que la secuencia de comandos se ejecutará (suscripciones, los nombres de servicio y así sucesivamente).
3. Ejecute el script de PowerShell.

> [!NOTE]
> La región especificada en el script de PowerShell debe coincidir con la región especificada en el archivo XML de configuración de red.
>
>

Después de la secuencia de comandos se ejecuta correctamente, pueden seguir estos pasos:

1. Configurar las reglas de firewall. Consulte la sección "Reglas de Firewall" de este artículo.
2. Si lo desea, puede configurar el servidor web y el servidor de aplicaciones con una aplicación web sencilla para permitir las pruebas con la configuración de red perimetral. Puede usar los scripts de dos aplicaciones que se proporcionan en la sección "Referencias".

La siguiente sección explica la mayoría de las instrucciones del script que se relacionan con los NSG.

## <a name="nsgs"></a>NSG
En este ejemplo, se crea un grupo de seguridad de red y se cargan después seis reglas.

> [!TIP]
> En general, debe crear primero las reglas específicas de "Permitir" y la última de las reglas de "Denegar" más genéricas. Los controles de la prioridad asignada qué reglas evalúan primero. Después de que se encuentra el tráfico que se aplica a una regla específica, no se evalúa ninguna otra regla. Pueden aplicar reglas NSG en la entrada o la dirección saliente (desde la perspectiva de la subred).
> 
> 

De forma declarativa, estas reglas se crean para el tráfico entrante:

1. Se permite el tráfico DNS interno (puerto 53).
2. Se permite el tráfico RDP (puerto 3389) desde internet a cualquier máquina virtual.
3. Se permite el tráfico HTTP (puerto 80) desde internet a NVA (firewall).
4. Se permite todo el tráfico (todos los puertos) desde IIS01 a AppVM01.
5. Se deniega todo el tráfico (todos los puertos) desde internet a toda la red virtual (ambas subredes).
6. Se deniega todo el tráfico (todos los puertos) desde la subred de front-end a la subred de back-end.

Con estas reglas enlazadas a cada subred, si tratara de una solicitud HTTP entrante desde internet al servidor web, ambas reglas 3 (permitir) y la regla 5 (denegar) podría parecer a aplicar, pero dado que la regla 3 tiene una prioridad más alta, solo se aplicará. Regla 5 no entran en juego. Por lo tanto, se permitirá la solicitud HTTP al firewall.

Si ese mismo tráfico estaba intentando tener acceso al servidor DNS01, regla 5 (denegar) sería la primera que se aplica, por lo que no se permite el tráfico a pasar al servidor. Regla 6 (denegar) bloquea la subred de front-end se comunique con la subred de back-end (excepto para el tráfico permitido en las reglas 1 y 4). Esto protege la red de back-end en caso de que un atacante ponga en peligro la aplicación web en el front-end. En ese caso, el atacante tendría acceso limitado a la red back-end "protegida". (El atacante podrá tener acceso a solo los recursos expuestos en el servidor AppVM01.)

Hay una regla de salida predeterminada que permite el tráfico a internet. En este ejemplo, estamos permitiendo el tráfico saliente y no modifica las reglas de salida. Para bloquear el tráfico en ambas direcciones, es necesario el enrutamiento definido por el usuario. Puede obtener información sobre esta técnica en otro ejemplo en el [documento de límites de seguridad principales][HOME].

Las reglas NSG descritas aquí son similares a las reglas de NSG en [ejemplo 1: creación de una red Perimetral simple con NSG][Example1]. Revise la descripción de NSG en ese artículo para una visión detallada de cada regla NSG y sus atributos.

## <a name="firewall-rules"></a>Reglas de firewall
Deberá instalar a un cliente de administración en un equipo para administrar el firewall y crear las configuraciones necesarias. Consulte la documentación de su firewall (o en otra) del proveedor sobre cómo administrar el dispositivo. El resto de esta sección describe la configuración del firewall, a través del cliente de administración del proveedor (no Azure portal o PowerShell).

Consulte [Barracuda NG Admin](https://techlib.barracuda.com/NG61/NGAdmin) para obtener instrucciones para descargar el cliente y conectarse al firewall Barracuda usado en este ejemplo.

Deberá crear las reglas de reenvío en el firewall. Dado que el escenario en este ejemplo solo enruta el tráfico de internet al firewall de entrada y, a continuación, en el servidor web, solo necesita una regla NAT de reenvío. En el firewall Barracuda usado en este ejemplo, la regla sería una regla NAT de destino (Dst NAT) para pasar este tráfico.

Para crear la regla siguiente (o para comprobar las reglas predeterminadas existentes), siga estos pasos:
1. En el panel de cliente de Barracuda NG Admin, en la pestaña Configuración, en el **configuración operativa** sección, seleccione **Ruleset**. 

   Una cuadrícula denominada **Main reglas** muestra existente activa y desactiva las reglas del firewall.

2. Para crear una nueva regla, seleccione el pequeño verde **+** situado en la esquina superior derecha de esta cuadrícula. (El firewall podría bloquearse. Si ve un botón marcado **bloqueo** y no se puede crear o editar reglas, seleccione el botón desbloquear el conjunto de reglas y permitir la edición.)
  
3. Para editar una regla existente, seleccione la regla con el botón secundario y, a continuación, seleccione **Editar regla**.

Crear una nueva regla un nombre parecido a **WebTraffic.** 

El icono de regla de NAT de destino tiene este aspecto:  ![Icono de NAT de destino][2]

La propia regla tendrá un aspecto similar al siguiente:

![Regla de Firewall][3]

Cualquier dirección entrante que alcance el firewall al intentar tener acceso a HTTP (puerto 80 o 443 para HTTPS) se envían fuera de la interfaz de DHCP1 Local IP del firewall y se redirige al servidor web con la dirección IP de 10.0.1.5. Dado que el tráfico se entra en el puerto 80 y vaya al servidor web en el puerto 80, no es necesario ningún cambio de puerto. Pero la lista de destino podría haber sido 10.0.1.5:8080 si el servidor web que escucha en el puerto 8080, que se traduciría en el puerto de entrada 80 en el firewall para el puerto 8080 de entrada en el servidor web.

También debe especificar un método de conexión. Para la regla de destino desde internet, Dynamic SNAT es el método más adecuado.

Aunque solo ha creado una regla, es importante establecer correctamente su prioridad. En la cuadrícula de todas las reglas del firewall, si esta regla nueva en la parte inferior (debajo de la regla BLOCKALL) nunca entra en juego. Asegúrese de que la nueva regla para el tráfico web está por encima de la regla BLOCKALL.

Una vez creada la regla, debe insertarla en el firewall y, a continuación, volver a activarla. Si no realiza estos pasos, el cambio de regla no surtirá efecto. La siguiente sección describe el proceso de activación e inserción.

## <a name="rule-activation"></a>Activación de reglas
Ahora que la regla se agrega al conjunto de reglas, deberá cargar el conjunto de reglas al firewall y actívelo.

![Activación de reglas de Firewall][4]

En la esquina superior derecha del cliente de administración, verá un grupo de botones. Seleccione **enviar cambios** para enviar el conjunto de reglas modificada al firewall y, a continuación, seleccione **activar**.

Ahora que ha activado el conjunto de reglas de firewall, el entorno está completando. Si lo desea, puede ejecutar los scripts de la aplicación de ejemplo en la sección "Referencias" para agregar una aplicación en el entorno. Si agrega una aplicación, puede probar los escenarios de tráfico que se describe en la sección siguiente.

> [!IMPORTANT]
> Tenga en cuenta que no llegue al servidor web directamente. Cuando un explorador solicita una página HTTP desde FrontEnd001.CloudApp.Net, el punto de conexión HTTP (puerto 80) pasa el tráfico al firewall, no al servidor web. El firewall, a continuación, debido a la regla que creó anteriormente, usa NAT para asignar la solicitud al servidor web.
> 
> 

## <a name="traffic-scenarios"></a>Escenarios de tráfico
#### <a name="allowed-web-to-web-server-through-the-firewall"></a>(Permitido) Web al servidor web a través del firewall
1. Un usuario de internet solicita una página HTTP desde FrontEnd001.CloudApp.Net (un servicio de nube a través de internet).
2. El servicio en la nube pasa el tráfico a través de un extremo abierto en el puerto 80 a interfaz local del firewall 10.0.1.4:80.
3. La subred de front-end comienza el procesamiento de la regla de entrada:
   1. No se aplica la regla de NSG 1 (DNS). Mover a la regla siguiente.
   2. No se aplica la regla NSG 2 (RDP). Mover a la regla siguiente.
   3. Aplicar la regla de NSG 3 (internet a firewall). Permitir el tráfico. Detener procesamiento de las reglas.
4. El tráfico llega a la dirección IP interna del firewall (10.0.1.4).
5. Un regla de reenvío de firewall se determina que es el tráfico del puerto 80 y lo redirige al servidor web IIS01.
6. Iis01 escucha el tráfico web, recibe la solicitud y comienza a procesar la solicitud.
7. Iis01 solicita información de la instancia de SQL Server en AppVM01.
8. No hay ninguna regla de salida en la subred front-end, por lo que se permite el tráfico.
9. La subred de back-end comienza el procesamiento de la regla de entrada:
   1. No se aplica la regla de NSG 1 (DNS). Mover a la regla siguiente.
   2. No se aplica la regla NSG 2 (RDP). Mover a la regla siguiente.
   3. No se aplica la regla de NSG 3 (internet a firewall). Mover a la regla siguiente.
   4. Regla NSG 4 (IIS01 a AppVM01) se aplican. Permitir el tráfico. Detener procesamiento de las reglas.
10. La instancia de SQL Server en AppVM01 recibe la solicitud y responde.
11. Dado que no hay ninguna regla de salida en la subred de back-end, se permite la respuesta.
12. La subred de front-end comienza el procesamiento de la regla de entrada:
    1. No hay ninguna regla NSG que se aplica al tráfico entrante desde la subred de back-end a la subred front-end, por lo que ninguna de las reglas NSG se aplican.
    2. La regla del sistema predeterminada que permite el tráfico entre subredes permite este tráfico, por lo que se permite el tráfico.
13. Iis01 recibe la respuesta de AppVM01, completa la respuesta HTTP y lo envía al solicitante.
14. Se trata de una sesión NAT del firewall, el destino de la respuesta es inicialmente para el firewall.
15. El firewall recibe la respuesta del servidor web y lo reenvía al usuario de internet.
16. Dado que no hay ninguna regla de salida en la subred de front-end, se permite la respuesta y el usuario de internet recibe la página web.

#### <a name="allowed-rdp-to-backend"></a>(Permitido) RDP a back-end
1. Un administrador del servidor en internet solicita una sesión RDP para AppVM01 en backend001.cloudapp.NET.:*xxxxx*, donde *xxxxx* es el número de puerto asignado de forma aleatoria para RDP a AppVM01. (Puede encontrar el puerto asignado en Azure portal o mediante PowerShell).
2. Dado que el firewall sólo está escuchando en la dirección FrontEnd001.CloudApp.Net, no participa con este flujo de tráfico.
3. La subred de back-end comienza el procesamiento de la regla de entrada:
   1. No se aplica la regla de NSG 1 (DNS). Mover a la regla siguiente.
   2. Aplicar la regla NSG 2 (RDP). Permitir el tráfico. Detener procesamiento de las reglas.
4. Porque no hay ninguna regla de salida, las reglas predeterminadas se aplican y devuelven se permite el tráfico.
5. La sesión de RDP está habilitada.
6. AppVM01 solicita un nombre de usuario y contraseña.

#### <a name="allowed-web-server-dns-lookup-on-dns-server"></a>(Permitido) Búsqueda DNS de servidor Web en el servidor DNS
1. La web server, IIS01, necesita datos de una fuente en www.data.gov, pero necesita resolver la dirección.
2. La configuración de red para que se muestra en la red virtual DNS01 (10.0.2.4 en la subred de back-end) como el servidor DNS principal. Iis01 envía la solicitud DNS a DNS01.
3. Dado que no hay ninguna regla de salida en la subred de front-end, se permite el tráfico.
4. La subred de back-end comienza el procesamiento de la regla de entrada:
   1. Se aplica la regla de NSG 1 (DNS). Permitir el tráfico. Detener procesamiento de las reglas.
5. El servidor DNS recibe la solicitud.
6. El servidor DNS no tiene la dirección en caché y consulta un servidor DNS de raíz en internet.
7. Dado que no hay ninguna regla de salida en la subred de back-end, se permite el tráfico.
8. El DNS de internet responde el servidor. Dado que la sesión se inició internamente, se permite la respuesta.
9. El servidor DNS almacena en caché la respuesta y responde a la solicitud de IIS01.
10. Dado que no hay ninguna regla de salida en la subred de back-end, se permite el tráfico.
11. La subred de front-end comienza el procesamiento de la regla de entrada:
    1. No hay ninguna regla NSG que se aplica al tráfico entrante desde la subred de back-end a la subred front-end, por lo que ninguna de las reglas NSG se aplican.
    2. La regla del sistema predeterminada que permite el tráfico entre subredes permite este tráfico, por lo que se permite el tráfico.
12. Iis01 recibe la respuesta de DNS01.

#### <a name="allowed-web-server-file-access-on-appvm01"></a>(Permitido) Acceso de archivo de servidor Web en AppVM01
1. Iis01 solicita un archivo en AppVM01.
2. Dado que no hay ninguna regla de salida en la subred de front-end, se permite el tráfico.
3. La subred de back-end comienza el procesamiento de la regla de entrada:
   1. No se aplica la regla de NSG 1 (DNS). Mover a la regla siguiente.
   2. No se aplica la regla NSG 2 (RDP). Mover a la regla siguiente.
   3. No se aplica la regla de NSG 3 (internet a firewall). Mover a la regla siguiente.
   4. Regla NSG 4 (IIS01 a AppVM01) se aplican. Permitir el tráfico. Detener procesamiento de las reglas.
4. AppVM01 recibe la solicitud y responde con el archivo (suponiendo que se autoriza el acceso).
5. Dado que no hay ninguna regla de salida en la subred de back-end, se permite la respuesta.
6. La subred de front-end comienza el procesamiento de la regla de entrada:
   1. No hay ninguna regla NSG que se aplica al tráfico entrante desde la subred de back-end a la subred front-end, por lo que ninguna de las reglas NSG se aplican.
   2. La regla del sistema predeterminada que permite el tráfico entre subredes permite este tráfico, por lo que se permite el tráfico.
7. Iis01 recibe el archivo.

#### <a name="denied-web-direct-to-web-server"></a>(Denegado) Web directa al servidor web
Dado que el servidor web IIS01 y el firewall están en el mismo servicio en la nube, comparten la misma dirección IP pública. Por lo que se dirige todo el tráfico HTTP al firewall. Mientras que una solicitud se atendió correctamente, no puede ir directamente al servidor web. Se aprueba, según lo previsto, a través del firewall en primer lugar. Vea el primer escenario en esta sección para el flujo de tráfico.

#### <a name="denied-web-to-backend-server"></a>(Denegado) Web a servidor back-end
1. Un usuario de internet intenta acceder a un archivo en AppVM01 a través del servicio backend001.cloudapp.NET.
2. Porque no hay ningún extremo abierto para uso compartido de archivos, esto no pase el servicio en la nube y no tener acceso al servidor.
3. Si los puntos de conexión están abiertos por alguna razón, la regla de NSG 5 (internet a red virtual) bloquea el tráfico.

#### <a name="denied-web-dns-lookup-on-the-dns-server"></a>(Denegado) Búsqueda de DNS Web en el servidor DNS
1. Un usuario de internet intenta buscar un registro DNS interno en DNS01 a través del servicio backend001.cloudapp.NET.
2. Porque no hay ningún extremo abierto para DNS, esto no pase el servicio en la nube y no tener acceso al servidor.
3. Si los puntos de conexión están abiertos por alguna razón, la regla de NSG 5 (internet a red virtual) bloquea el tráfico. (No se aplica la regla 1 [DNS] por dos motivos. En primer lugar, la dirección de origen es internet, y esta regla aplica solo cuando la red virtual local es el origen. En segundo lugar, esta regla es una regla de permiso, por lo que nunca deniega el tráfico).

#### <a name="denied-web-to-sql-access-through-the-firewall"></a>(Denegado) Web a acceso SQL a través del firewall
1. Un usuario de internet solicita datos SQL desde FrontEnd001.CloudApp.Net (un servicio de nube a través de internet).
2. Porque no hay ningún extremo abierto para SQL, esto no pase el servicio en la nube y no abarcará el firewall.
3. Si los puntos de conexión están abiertos por alguna razón, la subred de front-end inicia el procesamiento de la regla de entrada:
   1. No se aplica la regla de NSG 1 (DNS). Mover a la regla siguiente.
   2. No se aplica la regla NSG 2 (RDP). Mover a la regla siguiente.
   3. Aplicar la regla de NSG 3 (internet a firewall). Permitir el tráfico. Detener procesamiento de las reglas.
4. El tráfico llega a la dirección IP interna del firewall (10.0.1.4).
5. El firewall no tiene ninguna regla de reenvío para SQL y descarta el tráfico.

## <a name="conclusion"></a>Conclusión
En este ejemplo se muestra una manera relativamente sencilla para proteger la aplicación con un firewall y aislar la subred de back-end del tráfico entrante.

Puede encontrar más ejemplos e información general de red de los límites de seguridad [aquí][HOME].

## <a name="references"></a>Referencias
### <a name="full-script-and-network-config"></a>Configuración de red y la secuencia de comandos completa
Guarde el script completo en un archivo de script de PowerShell. Guarde el script de configuración de red en un archivo denominado NetworkConf2.xml.
Cambiar las variables definidas por el usuario según sea necesario. Ejecute el script y, a continuación, siga las instrucciones que aparecen en la sección "Reglas de Firewall" de este artículo.

#### <a name="full-script"></a>Script completo
Este script, en función de las variables definidas por el usuario, completar los pasos siguientes:

1. Conectarse a una suscripción de Azure.
2. Cree una cuenta de almacenamiento.
3. Crear una red virtual y dos subredes, tal como se define en el archivo de configuración de red.
4. Cree cuatro máquinas virtuales de Windows Server.
5. Configurar NSG. Estos pasos complete la configuración:
   * Crea un NSG.
   * Rellena el NSG con reglas.
   * Enlaza el NSG a las subredes adecuadas.

Debe ejecutar este script de PowerShell localmente en un equipo conectado a internet o servidor.

> [!IMPORTANT]
> Al ejecutar este script, podrían aparecer las advertencias y otros mensajes informativos en PowerShell. Sólo necesita preocuparse de los mensajes de error que aparecen en rojo.
> 
> 

```powershell
    <# 
     .SYNOPSIS
      Example of a perimeter network and Network Security Groups in an isolated network. (Azure only. No hybrid connections.)

     .DESCRIPTION
      This script will build out a sample perimeter network setup containing:
       - A default storage account for VM disks.
       - Two new cloud services.
       - Two subnets (the FrontEnd and BackEnd subnets).
       - A network virtual appliance (NVA): a Barracuda NextGen Firewall.
       - One server on the FrontEnd subnet (plus the NVA on the FrontEnd subnet).
       - Three servers on the BackEnd subnet.
       - Network Security Groups to allow/deny traffic patterns as declared.

      Before running the script, ensure the network configuration file is created in
      the directory referenced by the $NetworkConfigFile variable (or update the
      variable to reflect the path and file name of the config file being used).

     .Notes
      Security requirements are different for each use case and can be addressed in many ways. Be sure that any sensitive data or applications are behind
      the appropriate layer(s) of protection. This script serves as an example of some
      of the techniques that you can use, but it should not be used for all scenarios. You
      are responsible for assessing your security needs and the appropriate protections
      and then effectively implementing those protections.

      FrontEnd Service (FrontEnd subnet 10.0.1.0/24)
       myFirewall - 10.0.1.4
       IIS01      - 10.0.1.5

      BackEnd Service (BackEnd subnet 10.0.2.0/24)
       DNS01      - 10.0.2.4
       AppVM01    - 10.0.2.5
       AppVM02    - 10.0.2.6

    #>

    # Fixed Variables
        $LocalAdminPwd = Read-Host -Prompt "Enter Local Admin Password to be used for all VMs"
        $VMName = @()
        $ServiceName = @()
        $VMFamily = @()
        $img = @()
        $size = @()
        $SubnetName = @()
        $VMIP = @()

    # User-Defined Global Variables
      # These should be changed to reflect your subscription and services.
      # Invalid options will fail in the validation section.

      # Subscription Access Details
        $subID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

      # VM Account, Location, and Storage Details
        $LocalAdmin = "theAdmin"
        $DeploymentLocation = "Central US"
        $StorageAccountName = "vmstore02"

      # Service Details
        $FrontEndService = "FrontEnd001"
        $BackEndService = "BackEnd001"

      # Network Details
        $VNetName = "CorpNetwork"
        $FESubnet = "FrontEnd"
        $FEPrefix = "10.0.1.0/24"
        $BESubnet = "BackEnd"
        $BEPrefix = "10.0.2.0/24"
        $NetworkConfigFile = "C:\Scripts\NetworkConf2.xml"

      # VM Base Disk Image Details
        $SrvImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Windows Server 2012 R2 Datacenter'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}
        $FWImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Barracuda NextGen Firewall'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}

      # NSG Details
        $NSGName = "MyVNetSG"

    # User-Defined VM-Specific Config
        # To ensure proper NSG rule creation later in this script:
        #       - The web server must be VM 1.
        #       - The AppVM1 server must be VM 2.
        #       - The DNS server must be VM 4.
        #
        #       Otherwise the NSG rules in the last section of this
        #       script will need to be changed to match the modified
        #       VM array numbers ($i) so the NSG rule IP addresses
        #       are aligned to the associated VM IP addresses.

        # VM 0 - The Network Virtual Appliance (NVA)
          $VMName += "myFirewall"
          $ServiceName += $FrontEndService
          $VMFamily += "Firewall"
          $img += $FWImg
          $size += "Small"
          $SubnetName += $FESubnet
          $VMIP += "10.0.1.4"

        # VM 1 - The Web Server
          $VMName += "IIS01"
          $ServiceName += $FrontEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $FESubnet
          $VMIP += "10.0.1.5"

        # VM 2 - The First Application Server
          $VMName += "AppVM01"
          $ServiceName += $BackEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $BESubnet
          $VMIP += "10.0.2.5"

        # VM 3 - The Second Application Server
          $VMName += "AppVM02"
          $ServiceName += $BackEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $BESubnet
          $VMIP += "10.0.2.6"

        # VM 4 - The DNS Server
          $VMName += "DNS01"
          $ServiceName += $BackEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $BESubnet
          $VMIP += "10.0.2.4"

    # ----------------------------- #
    # No user-defined variables or   #
    # configuration past this point #
    # ----------------------------- #

      # Get your Azure accounts
        Add-AzureAccount
        Set-AzureSubscription –SubscriptionId $subID -ErrorAction Stop
        Select-AzureSubscription -SubscriptionId $subID -Current -ErrorAction Stop

      # Create storage account
        If (Test-AzureName -Storage -Name $StorageAccountName) { 
            Write-Host "Fatal Error: This storage account name is already in use, please pick a diffrent name." -ForegroundColor Red
            Return}
        Else {Write-Host "Creating Storage Account" -ForegroundColor Cyan 
              New-AzureStorageAccount -Location $DeploymentLocation -StorageAccountName $StorageAccountName}

      # Update subscription pointer to new storage account
        Write-Host "Updating Subscription Pointer to New Storage Account" -ForegroundColor Cyan 
        Set-AzureSubscription –SubscriptionId $subID -CurrentStorageAccountName $StorageAccountName -ErrorAction Stop

    # Validation
    $FatalError = $false

    If (-Not (Get-AzureLocation | Where {$_.DisplayName -eq $DeploymentLocation})) {
         Write-Host "This Azure Location was not found or available for use" -ForegroundColor Yellow
         $FatalError = $true}

    If (Test-AzureName -Service -Name $FrontEndService) { 
        Write-Host "The FrontEndService service name is already in use, please pick a different service name." -ForegroundColor Yellow
        $FatalError = $true}
    Else { Write-Host "The FrontEndService service name is valid for use." -ForegroundColor Green}

    If (Test-AzureName -Service -Name $BackEndService) { 
        Write-Host "The BackEndService service name is already in use, please pick a different service name." -ForegroundColor Yellow
        $FatalError = $true}
    Else { Write-Host "The BackEndService service name is valid for use." -ForegroundColor Green}

    If (-Not (Test-Path $NetworkConfigFile)) { 
        Write-Host 'The network config file was not found, please update the $NetworkConfigFile variable to point to the network config xml file.' -ForegroundColor Yellow
        $FatalError = $true}
    Else { Write-Host "The network config file was found" -ForegroundColor Green
            If (-Not (Select-String -Pattern $DeploymentLocation -Path $NetworkConfigFile)) {
                Write-Host 'The deployment location was not found in the network config file, please check the network config file to ensure the $DeploymentLocation variable is correct and the network config file matches.' -ForegroundColor Yellow
                $FatalError = $true}
            Else { Write-Host "The deployment location was found in the network config file." -ForegroundColor Green}}

    If ($FatalError) {
        Write-Host "A fatal error has occurred, please see the above messages for more information." -ForegroundColor Red
        Return}
    Else { Write-Host "Validation passed, now building the environment." -ForegroundColor Green}

    # Create virtual network
        Write-Host "Creating VNET" -ForegroundColor Cyan 
        Set-AzureVNetConfig -ConfigurationPath $NetworkConfigFile -ErrorAction Stop

    # Create services
        Write-Host "Creating Services" -ForegroundColor Cyan
        New-AzureService -Location $DeploymentLocation -ServiceName $FrontEndService -ErrorAction Stop
        New-AzureService -Location $DeploymentLocation -ServiceName $BackEndService -ErrorAction Stop

    # Build VMs
        $i=0
        $VMName | Foreach {
            Write-Host "Building $($VMName[$i])" -ForegroundColor Cyan
            If ($VMFamily[$i] -eq "Firewall") 
                { 
                New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] –InstanceSize $size[$i] | `
                    Add-AzureProvisioningConfig -Linux -LinuxUser $LocalAdmin -Password $LocalAdminPwd  | `
                    Set-AzureSubnet  –SubnetNames $SubnetName[$i] | `
                    Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
                    New-AzureVM –ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
                # Set up all the endpoints we'll need once we're up and running.
                # Note: Web traffic goes through the firewall, so we'll need to set up an HTTP endpoint.
                #       Also, the firewall will be redirecting web traffic to a new IP and port in a
                #       forwarding rule, so the HTTP endpoint here will have the same public and local
                #       port and the firewall will do the NATing and redirection as declared in the
                #       firewall rule.
                Add-AzureEndpoint -Name "MgmtPort1" -Protocol tcp -PublicPort 801  -LocalPort 801  -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "MgmtPort2" -Protocol tcp -PublicPort 807  -LocalPort 807  -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "HTTP"      -Protocol tcp -PublicPort 80   -LocalPort 80   -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                # Note: An SSH endpoint is automatically created on port 22 when the appliance is created.
                }
            Else
                {
                New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] –InstanceSize $size[$i] | `
                    Add-AzureProvisioningConfig -Windows -AdminUsername $LocalAdmin -Password $LocalAdminPwd  | `
                    Set-AzureSubnet  –SubnetNames $SubnetName[$i] | `
                    Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
                    Set-AzureVMMicrosoftAntimalwareExtension -AntimalwareConfiguration '{"AntimalwareEnabled" : true}' | `
                    Remove-AzureEndpoint -Name "PowerShell" | `
                    New-AzureVM –ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
                    # Note: A Remote Desktop endpoint is automatically created when each VM is created.
                }
            $i++
        }

    # Configure NSG
        Write-Host "Configuring the Network Security Group (NSG)" -ForegroundColor Cyan

      # Build the NSG
        Write-Host "Building the NSG" -ForegroundColor Cyan
        New-AzureNetworkSecurityGroup -Name $NSGName -Location $DeploymentLocation -Label "Security group for $VNetName subnets in $DeploymentLocation"

      # Add NSG rules
        Write-Host "Writing rules into the NSG" -ForegroundColor Cyan
        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable Internal DNS" -Type Inbound -Priority 100 -Action Allow `
            -SourceAddressPrefix VIRTUAL_NETWORK -SourcePortRange '*' `
            -DestinationAddressPrefix $VMIP[4] -DestinationPortRange '53' `
            -Protocol *

        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable RDP to $VNetName VNet" -Type Inbound -Priority 110 -Action Allow `
            -SourceAddressPrefix INTERNET -SourcePortRange '*' `
            -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '3389' `
            -Protocol *

        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable Internet to $($VMName[0])" -Type Inbound -Priority 120 -Action Allow `
            -SourceAddressPrefix Internet -SourcePortRange '*' `
            -DestinationAddressPrefix $VMIP[0] -DestinationPortRange '*' `
            -Protocol *

        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable $($VMName[1]) to $($VMName[2])" -Type Inbound -Priority 130 -Action Allow `
            -SourceAddressPrefix $VMIP[1] -SourcePortRange '*' `
            -DestinationAddressPrefix $VMIP[2] -DestinationPortRange '*' `
            -Protocol *

        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Isolate the $VNetName VNet from the Internet" -Type Inbound -Priority 140 -Action Deny `
            -SourceAddressPrefix INTERNET -SourcePortRange '*' `
            -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '*' `
            -Protocol *

        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Isolate the $FESubnet subnet from the $BESubnet subnet" -Type Inbound -Priority 150 -Action Deny `
            -SourceAddressPrefix $FEPrefix -SourcePortRange '*' `
            -DestinationAddressPrefix $BEPrefix -DestinationPortRange '*' `
            -Protocol *

        # Assign the NSG to the subnets
            Write-Host "Binding the NSG to both subnets" -ForegroundColor Cyan
            Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName -SubnetName $FESubnet -VirtualNetworkName $VNetName
            Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName -SubnetName $BESubnet -VirtualNetworkName $VNetName

    # Optional Post-Script Manual Configuration
      # Configure firewall
      # Install test web app (run post-build script on the IIS server)
      # Install back-end resources (run post-build script on AppVM01)
      Write-Host
      Write-Host "Build Complete!" -ForegroundColor Green
      Write-Host
      Write-Host "Optional Post-script Manual Configuration Steps" -ForegroundColor Gray
      Write-Host " - Configure Firewall" -ForegroundColor Gray
      Write-Host " - Install Test Web App (Run Post-Build Script on the IIS Server)" -ForegroundColor Gray
      Write-Host " - Install Backend resource (Run Post-Build Script on the AppVM01)" -ForegroundColor Gray
      Write-Host
```

#### <a name="network-config-file"></a>Archivo de configuración de red
Guarde este archivo XML con ubicaciones actualizadas y, a continuación, agregue un vínculo a este archivo en la variable $NetworkConfigFile en el script anterior.

```xml
    <NetworkConfiguration xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
      <VirtualNetworkConfiguration>
        <Dns>
          <DnsServers>
            <DnsServer name="DNS01" IPAddress="10.0.2.4" />
            <DnsServer name="Level3" IPAddress="209.244.0.3" />
          </DnsServers>
        </Dns>
        <VirtualNetworkSites>
          <VirtualNetworkSite name="CorpNetwork" Location="Central US">
            <AddressSpace>
              <AddressPrefix>10.0.0.0/16</AddressPrefix>
            </AddressSpace>
            <Subnets>
              <Subnet name="FrontEnd">
                <AddressPrefix>10.0.1.0/24</AddressPrefix>
              </Subnet>
              <Subnet name="BackEnd">
                <AddressPrefix>10.0.2.0/24</AddressPrefix>
              </Subnet>
            </Subnets>
            <DnsServersRef>
              <DnsServerRef name="DNS01" />
              <DnsServerRef name="Level3" />
            </DnsServersRef>
          </VirtualNetworkSite>
        </VirtualNetworkSites>
      </VirtualNetworkConfiguration>
    </NetworkConfiguration>
```

#### <a name="sample-application-scripts"></a>Scripts de aplicación de ejemplo
Si desea instalar una aplicación de ejemplo para este y otros ejemplos de la red perimetral, consulte el [script de la aplicación de ejemplo][SampleApp].

<!--Image References-->
[1]: ./media/virtual-networks-dmz-nsg-fw-asm/example2design.png "Red perimetral de entrada con grupo de seguridad de red"
[2]: ./media/virtual-networks-dmz-nsg-fw-asm/dstnaticon.png "Icono de NAT de destino"
[3]: ./media/virtual-networks-dmz-nsg-fw-asm/firewallrule.png "Regla de firewall"
[4]: ./media/virtual-networks-dmz-nsg-fw-asm/firewallruleactivate.png "Activación de reglas de firewall"

<!--Link References-->
[HOME]: ../best-practices-network-security.md
[SampleApp]: ./virtual-networks-sample-app.md
[Example1]: ./virtual-networks-dmz-nsg-asm.md
