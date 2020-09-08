---
title: 'Solución de problemas de rendimiento en los vínculos de red: Azure'
description: En esta página se proporciona un método de prueba normalizado del rendimiento de vínculos de red de Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: troubleshooting
ms.date: 12/20/2017
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 4d6dac98125827a303b8370b5a0a959ea425fb57
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401459"
---
# <a name="troubleshooting-network-performance"></a>Solución de problemas de rendimiento de red
## <a name="overview"></a>Información general
Azure ofrece formas estables y rápidas de conectarse desde la red local a Azure. Clientes grandes y pequeños usan correctamente métodos como VPN de sitio a sitio y ExpressRoute para ejecutar sus negocios en Azure. Pero, ¿qué ocurre cuando el rendimiento no satisface sus expectativas o la experiencia anterior? Este documento puede ayudar a normalizar la forma de probar su entorno específico y de crear una base de referencia para él.

En este documento se explica cómo puede probar la latencia de red y el ancho de banda de forma fácil y coherente entre dos hosts. En este documento también se proporcionan algunos consejos para examinar la red de Azure y ayudar a aislar puntos problemáticos. Las herramientas y el script de PowerShell descritos requieren dos hots en la red (en cualquiera de los extremos del vínculo que se va a probar). Un host debe ejecutar el Escritorio de Windows o Windows Server; el otro puede ejecutar Windows o Linux. 

>[!NOTE]
>El enfoque para solucionar problemas, las herramientas y los métodos utilizados son preferencias personales. En este documento se describen el enfoque y las herramientas que suelo utilizar. Puede que el enfoque sea distinto, pero no hay ningún inconveniente con usar enfoques diferentes para solucionar problemas. Sin embargo, si no tiene un enfoque establecido, este documento puede ayudarle a comenzar a compilar métodos, herramientas y preferencias propios para solucionar problemas de red.
>
>

## <a name="network-components"></a>Componentes de red
Antes de adentrarnos en la solución de problemas, veamos algunos componentes y términos comunes. Esta explicación garantiza que estamos pensando en cada componente de la cadena de un extremo a otro que permite la conectividad en Azure.
![1][1]

En el nivel más alto, describo los tres dominios de enrutamiento de red principales:

- la red de Azure (la nube azul de la derecha)
- Internet o WAN (la nube verde del centro)
- la red corporativa (la red naranja de la izquierda)

Vamos a analizar brevemente cada componente fijándonos en el diagrama de derecha a izquierda:
 - **Virtual Machine**: el servidor puede tener varias NIC; asegúrese de que todas las rutas estáticas, las rutas predeterminadas y la configuración del sistema operativo envían y reciben tráfico según lo previsto. Además, cada SKU de máquina virtual tiene una restricción de ancho de banda. Si usa una SKU de máquina virtual más pequeña, el ancho de banda disponible para la NIC limita el tráfico. Suelo usar una máquina virtual DS5v2 para las pruebas, que elimino una vez completadas las pruebas para ahorrar dinero, a fin de garantizar la disponibilidad de un ancho de banda apropiado en la máquina virtual.
 - **NIC**: asegúrese de que conoce la dirección IP privada asignada a la NIC en cuestión.
 - **NIC NSG**: puede haber NSG específicos aplicados a nivel de la NIC; asegúrese de que el conjunto de reglas de NSG es apropiado para el tráfico que intenta transmitir. Por ejemplo, asegúrese de que los puertos 5201 para iPerf, 3389 para RDP o 22 para SSH están abiertos para permitir la transferencia del tráfico de prueba.
 - **VNet Subnet**: la NIC está asignada a una subred específica; asegúrese de saber cuál es y las reglas asociadas a dicha subred.
 - **Subnet NSG**: al igual que la NIC, los NSG también pueden aplicarse a la subred. Asegúrese de que el conjunto de reglas de NSG es apropiado para el tráfico que intenta transmitir. (Para el tráfico entrante en la NIC, primero se aplica el NSG de la subred y luego el NSG de la NIC; a la inversa, para el tráfico saliente de la máquina virtual, primero se aplica el NSG de la NIC y luego el NSG de la subred).
 - **Subnet UDR**: las rutas definidas por el usuario puede dirigir el tráfico a un salto intermedio (como un firewall o un equilibrador de carga). Asegúrese de saber si existe una UDR para el tráfico y, en su caso, dónde se dirige y qué hará el siguiente salto con el tráfico. (Por ejemplo, un firewall puede pasar algún tráfico y denegar otro entre los mismos dos hosts).
 - **Gateway subnet / NSG / UDR**: de la misma manera que la subred de máquina virtual, la subred de puerta de enlace puede tener NSG y UDR. Asegúrese de que existan y de qué efectos tienen en el tráfico.
 - **VNet Gateway (ExpressRoute)** : una vez habilitado el emparejamiento (ExpressRoute) o VPN, existen muchas configuraciones que pueden afectar a que el tráfico se redirija o no y a cómo se lleva. Si tiene varios circuitos ExpressRoute o túneles VPN conectados a la misma puerta de enlace de red virtual, debe tener en cuenta la configuración del peso de la conexión, ya que este valor afecta a las preferencias de conexión y a la ruta que el tráfico sigue.
 - **Route Filter** (no se muestra): un filtro de ruta solo se aplica al Emparejamiento de Microsoft en ExpressRoute, pero es crítico para comprobar si no aparecen las rutas previstas en el Emparejamiento de Microsoft. 

En este punto, está en la parte WAN del vínculo. Este dominio de enrutamiento puede ser el proveedor de servicios, la WAN corporativa o Internet. Muchos saltos, tecnologías y empresas relacionados con estos vínculos pueden dificultar en cierta medida la solución de problemas. A menudo, trata de descartar Azure y las redes corporativas antes de pasar a este grupo de empresas y saltos.

En el diagrama anterior, en el extremo izquierdo, se muestra la red corporativa. Según el tamaño de su empresa, este dominio de enrutamiento puede corresponderse con unos cuantos dispositivos de red entre usted y la WAN o con varias capas de dispositivos de una red empresarial o de campus.

Dada la complejidad que plantean estos tres entornos de red de alto nivel distintos, suele ser una opción óptima empezar por los bordes y tratar de mostrar dónde el rendimiento es bueno y dónde se reduce. Este enfoque puede ayudar a identificar el dominio de enrutamiento de problemas de entre los tres disponibles y después centrar la solución de problemas en dicho entorno específico.

## <a name="tools"></a>Herramientas
La mayoría de los problemas de red se pueden analizar y aislar con herramientas básicas como ping y traceroute. Es raro que necesite indagar tanto como un análisis de paquetes como Wireshark. Para facilitar la solución de problemas, se desarrolló Azure Connectivity Toolkit (AzureCT) para incluir algunas de estas herramientas en un paquete sencillo. Para pruebas de rendimiento, me gusta usar iPerf y PSPing. iPerf es una herramienta de uso frecuente que se ejecuta en la mayoría de los sistemas operativos. iPerf es una buena opción para pruebas de rendimiento básicas y resulta bastante fácil de usar. PSPing es una herramienta Ping desarrollada por SysInternals. PSPing es una forma sencilla de ejecutar ICMP y TCP hace ping en un comando que también es fácil de utilizar. Ambas herramientas son ligeras y se "instalan" solo con copiar los archivos en un directorio del host.

He encapsulado todas estas herramientas y métodos en un módulo de PowerShell (AzureCT) que puede instalar y usar.

### <a name="azurect---the-azure-connectivity-toolkit"></a>AzureCT (Azure Connectivity Toolkit)
El módulo AzureCT de PowerShell tiene dos componentes: [pruebas de disponibilidad][Availability Doc] y [pruebas de rendimiento][Performance Doc]. En este documento solo se tratan las pruebas de rendimiento, por lo que vamos a centrarnos en dos comandos para medir el rendimiento de los vínculos en este módulo de PowerShell.

Hay tres pasos básicos para usar este kit de herramientas para pruebas de rendimiento. 1) Instalación del módulo de PowerShell, 2) Instalación de las aplicaciones auxiliares iPerf y PSPing y 3) Ejecución de la prueba de rendimiento.

1. Instalación del módulo de PowerShell

    ```powershell
    (new-object Net.WebClient).DownloadString("https://aka.ms/AzureCT") | Invoke-Expression
    
    ```

    Este comando descarga el módulo de PowerShell y lo instala de forma local.

2. Instalación de las aplicaciones auxiliares
    ```powershell
    Install-LinkPerformance
    ```
    Este comando de AzureCT instala iPerf y PSPing en un directorio nuevo: "C:\ACTTools"; además, abre los puertos del Firewall de Windows para permitir el tráfico ICMP y del puerto 5201 (iPerf).

3. Ejecución de la prueba de rendimiento

    En primer lugar, debe instalar y ejecutar iPerf en modo de servidor en el host remoto. Asegúrese también de que el host remoto escucha en los puertos 3389 (RDP para Windows) o 22 (SSH para Linux) y que se permite el tráfico a través del puerto 5201 para iPerf. Si el host remoto ejecuta Windows, instale AzureCT y ejecute el comando Install-LinkPerformance para configurar iPerf y las reglas de firewall necesarias para iniciar iPerf correctamente en modo de servidor. 
    
    Una vez preparada la máquina remota, abra PowerShell en la máquina local e inicie la prueba:
    ```powershell
    Get-LinkPerformance -RemoteHost 10.0.0.1 -TestSeconds 10
    ```

    Este comando ejecuta una serie de pruebas de carga y latencia simultáneas para ayudar a estimar la capacidad y latencia de ancho de banda del vínculo de red.

4. Revisión del resultado de las pruebas

    El formato del resultado de PowerShell tiene un aspecto similar al siguiente:

    ![4][4]

    Los resultados detallados de todas las pruebas de iPerf y PSPing están en archivos de texto individuales en el directorio de herramientas de AzureCT en "C:\ACTTools".

## <a name="troubleshooting"></a>Solución de problemas
Si la prueba de rendimiento no ofrece los resultados esperados, averigüe por qué debe aplicar un proceso detallado progresivo. Dado el número de componentes de la ruta, un enfoque sistemático suele ofrecer una ruta más rápida para la resolución que saltar de un lado a otro y realizar las mismas pruebas varias veces innecesariamente.

>[!NOTE]
>El escenario que se presenta aquí representa un problema de rendimiento, no de conectividad. Los pasos serían distintos si no pasara absolutamente nada de tráfico.
>
>

En primer lugar, cuestione sus hipótesis. ¿Su expectativa es razonable? Por ejemplo, si tiene un circuito ExpressRoute de 1 Gbps y 100 ms de latencia, resulta poco razonable esperar que se transmita todo el tráfico de 1 Gbps debido a las características de rendimiento de TCP a través de vínculos de latencia alta. Consulte la [sección Referencias](#references) para obtener más información sobre las hipótesis de rendimiento.

A continuación, recomiendo empezar a partir de los bordes entre los dominios de enrutamiento y tratar de aislar el problema en un único dominio de enrutamiento principal, a saber, la red corporativa, la red WAN o la red de Azure. Los usuarios suelen echar la culpa a la "caja negra" de la ruta, pero, aunque es fácil echar la culpa a la caja negra, esto puede retrasar bastante la resolución, sobre todo si el problema radica realmente en un área en la que existe la posibilidad de realizar cambios. Asegúrese de actuar correctamente antes de recurrir al proveedor de servicios o ISP.

Después de identificar el dominio de enrutamiento principal que parece contener el problema, debe crear un diagrama del área en cuestión. Un diagrama en una pizarra, en el Bloc de notas o en Visio ofrece un "mapa de trabajo" para permitir un enfoque metódico para aislar más el problema. Puede planear puntos de prueba y actualizar el mapa al borrar áreas o profundizar más a medida que las pruebas avanzan.

Ahora que tiene un diagrama, empiece a dividir la red en segmentos para limitar el problema. Averigüe dónde funciona y dónde no. Continúe moviendo los puntos de prueba para aislar el componente que genera el problema.

Además, no olvide examinar otras capas del modelo OSI. Es fácil centrarse en la red y en las capas 1 a 3 (física, datos y red), pero los problemas pueden llegar también hasta la capa 7 en el nivel de aplicación. Mantenga una mente abierta y verifique sus hipótesis.

## <a name="advanced-expressroute-troubleshooting"></a>Solución avanzada de problemas de ExpressRoute
Si no está seguro de dónde está realmente el perímetro de la nube, aislar los componentes de Azure puede ser un desafío. Cuando se utiliza ExpressRoute, el perímetro es un componente de red llamado Microsoft Enterprise Edge (MSEE). **Al usar ExpressRoute**, MSEE es el primer punto de contacto en la red de Microsoft y el último salto que sale de la red de Microsoft. Al crear un objeto de conexión entre la puerta de enlace de red virtual y el circuito ExpressRoute, lo que se hace realmente es establecer una conexión a MSEE. Reconocer MSEE en el primer o último salto (según la dirección que siga) es fundamental para aislar los problemas de la red de Azure, a fin de demostrar que el problema está en Azure o en un nivel inferior en la red WAN o corporativa. 

![2][2]

>[!NOTE]
> Tenga en cuenta que MSEE no se encuentra en la nube de Azure. ExpressRoute realmente se encuentra en el perímetro de la red de Microsoft, y no en Azure. Una vez conectado a ExpressRoute para una instancia de MSEE, se establece la conexión con la red de Microsoft, desde donde puede acceder a cualquier servicio en la nube, como Office 365 (con el Emparejamiento de Microsoft) o Azure (con el emparejamiento privado o el Emparejamiento de Microsoft).
>
>

Si dos redes virtuales (redes virtuales A y B en el diagrama) están conectadas al **mismo** circuito ExpressRoute, puede realizar una serie de pruebas para aislar el problema en Azure o para demostrar que el problema no está en Azure.
 
### <a name="test-plan"></a>Plan de pruebas
1. Ejecute la prueba Get-LinkPerformance entre VM1 y VM2. Esta prueba proporciona información sobre si el problema es local o no. Si la prueba genera resultados de latencia y ancho de banda aceptables, puede marcar la red virtual local como correcta.
2. Suponiendo que el tráfico de la red virtual local es correcto, ejecute la prueba Get-LinkPerformance entre VM1 y VM3. Esta prueba establece la conexión a través de la red de Microsoft en orden descendente hasta llegar a MSEE y vuelve a Azure. Si la prueba genera resultados de latencia y ancho de banda aceptables, puede marcar la red de Azure como correcta.
3. Si se descarta Azure, puede realizar una secuencia similar de pruebas en su red corporativa. Si estas pruebas también se realizan correctamente, es el momento de recurrir al proveedor de servicios o ISP para diagnosticar la conexión WAN. Ejemplo: Ejecute esta prueba entre dos sucursales o entre su escritorio y el servidor de un centro de datos. Según lo que vaya a probar, busque puntos de conexión (servidores, equipos, etc.) que puedan probar esa ruta.

>[!IMPORTANT]
> Es fundamental que, para cada prueba, indique la hora del día a la que se debe ejecutar la prueba y que registre los resultados en una ubicación común (como OneNote o Excel). Cada serie de pruebas deben tener resultados idénticos para que pueda comparar los datos resultantes entre ejecuciones de pruebas y no tener "carencia" de datos. La coherencia entre varias pruebas es la razón principal por la que uso AzureCT para solucionar problemas. El *truco* no está en los escenarios de carga exactos que ejecuto, sino en el hecho de que obtengo un *resultado de pruebas y datos coherente* de cada prueba. Registrar la hora y disponer de datos coherentes de cada prueba resulta especialmente útil si más tarde descubre que el problema es esporádico. Sea minucioso con la recopilación anticipada de datos y, de esta forma, evitar horas de pruebas repetidas de los mismos escenarios (aprendí esta lección hace muchos años).
>
>

## <a name="the-problem-is-isolated-now-what"></a>El problema ya está aislado, ¿ahora qué?
Cuanto más pueda aislar el problema, más fácil será solucionarlo; no obstante, a menudo se da el caso de que se llega a un punto en que ya no es posible profundizar más o continuar con la solución del problema. Ejemplo: encuentra un vínculo del proveedor de servicios que realiza saltos por toda Europa, pero esperaba que la ruta radicara en Asia. Ahora es cuando debe solicitar ayuda. A quién pregunte depende del dominio de enrutamiento en que ha aislado el problema o, todavía mejor, del hecho de que haya logrado aislar el problema en un componente específico.

Si se trata de problemas con la red corporativa, el departamento de TI o el proveedor de servicios que presta soporte técnico para la red (que puede ser el fabricante de hardware) puede ayudar con la configuración del dispositivo o la reparación del hardware.

Si se trata de un problema con la red WAN, compartir los resultados de la prueba con el proveedor de servicios o ISP puede ayudarlos a ponerse manos a la obra y evitar que realicen las mismas pruebas que usted ya ha hecho. Sin embargo, no debe ofenderse si desean verificar los resultados que usted ha obtenido. "Confiar pero verificar" es un buen principio para solucionar problemas según los resultados notificados por los usuarios.

Con Azure, una vez aislado el problema con toda la información posible, debe revisar la [documentación sobre la red de Azure][Network Docs] y, después, considerar si aún resulta necesario [abrir un vale de soporte][Ticket Link].

## <a name="references"></a>Referencias
### <a name="latencybandwidth-expectations"></a>Expectativas de ancho de banda y latencia
>[!TIP]
> La latencia geográfica (millas o kilómetros) entre los puntos de conexión probados es, con diferencia, el componente más importante de la latencia. Aunque también influye la latencia de equipos (componentes físicos y virtuales, número de saltos, etc.), la ubicación geográfica ha demostrado ser el componente más importante de la latencia general cuando se trata de conexiones WAN. También es importante tener en cuenta que la distancia se corresponde con la distancia del tendido de fibra, no con la distancia lineal o la que se mide en un mapa de carreteras. Es muy difícil calcular esta distancia con precisión. Por tanto, suelo usar una calculadora de distancia entre ciudades disponible en Internet, aunque sé que este método ofrece una medida bastante inexacta, pero resulta suficiente para crear una expectativa general.
>
>

Tengo una instalación de ExpressRoute en Seattle, Washington, en los Estados Unidos. En la siguiente tabla se reflejan la latencia y el ancho de banda obtenidos tras probar varias ubicaciones de Azure. He calculado la distancia geográfica entre cada extremo de la prueba.

Configuración de la prueba:
 - Un servidor físico que ejecuta Windows Server 2016 con una NIC de 10 Gbps, conectado a un circuito ExpressRoute.
 - Un circuito ExpressRoute premium de 10 Gbps en la ubicación identificada con el emparejamiento privado habilitado.
 - Una red virtual de Azure con una puerta de enlace UltraPerformance en la región especificada.
 - Una VM DS5v2 que ejecuta Windows Server 2016 en la red virtual. La máquina virtual no estaba unida a un dominio y se compiló a partir de la imagen predeterminada de Azure (sin optimización ni personalización) con AzureCT instalado.
 - Todas las pruebas se realizaron mediante el comando Get-LinkPerformance de AzureCT con una prueba de carga de cinco minutos para cada una de las seis series de pruebas. Por ejemplo:

    ```powershell
    Get-LinkPerformance -RemoteHost 10.0.0.1 -TestSeconds 300
    ```
 - En el flujo de datos de cada prueba, la carga se transfería desde el servidor físico local (cliente de iPerf en Seattle) hasta la máquina virtual de Azure (servidor de iPerf en la región de Azure especificada).
 - Los datos de la columna "Latencia" resultan de la prueba sin carga (una prueba de latencia de TCP sin la ejecución de iPerf).
 - Los datos de la columna "Ancho de banda máx." proceden de la prueba de carga del flujo de TCP 16 con un tamaño de ventana de 1 Mb.

![3][3]

### <a name="latencybandwidth-results"></a>Resultados de latencia y ancho de banda
>[!IMPORTANT]
> Estas cifras solo sirven como referencia general. Existen muchos factores que afectan a la latencia y, aunque estos valores son coherentes a lo largo del tiempo, hay condiciones en Azure o en la red de los proveedores de servicios que pueden enviar tráfico a través de diferentes rutas en cualquier momento, un hecho que puede afectar a la latencia y al ancho de banda. Por norma general, los efectos de estos cambios no generan diferencias importantes.
>
>

| ExpressRoute<br/>Location|Azure<br/>Region | Estimación<br/>Distancia (km) | Latencia|1 Sesión<br/>Ancho de banda | Máxima<br/>Ancho de banda |
| ------------------------------------------ | --------------------------- |  - | - | - | - |
| Seattle | Oeste de EE. UU. 2        |    191 km |   5 ms | 262,0 Mbits/s |  3,74 Gbits/s |
| Seattle | Oeste de EE. UU.          |  1094 km |  18 ms |  82,3 Mbits/s |  3,70 Gbits/s |
| Seattle | Centro de EE. UU.       |  2357 km |  40 ms |  38,8 Mbits/s |  2,55 Gbits/s |
| Seattle | Centro-sur de EE. UU. |  2877 km |  51 ms |  30,6 Mbits/s |  2,49 Gbits/s |
| Seattle | Centro-Norte de EE. UU |  2792 km |  55 ms |  27,7 Mbits/s |  2,19 Gbits/s |
| Seattle | Este de EE. UU. 2        |  3769 km |  73 ms |  21,3 Mbits/s |  1,79 Gbits/s |
| Seattle | Este de EE. UU.          |  3699 km |  74 ms |  21,1 Mbits/s |  1,78 Gbits/s |
| Seattle | Japón Oriental       |  7705 km | 106 ms |  14,6 Mbits/s |  1,22 Gbits/s |
| Seattle | Sur de Reino Unido 2         |  7708 km | 146 ms |  10,6 Mbits/s |   896 Mbits/s |
| Seattle | Oeste de Europa      |  7834 km | 153 ms |  10,2 Mbits/s |   761 Mbits/s |
| Seattle | Este de Australia   | 12 484 km | 165 ms |   9,4 Mbits/s |   794 Mbits/s |
| Seattle | Sudeste de Asia   | 12 989 km | 170 ms |   9,2 Mbits/s |   756 Mbits/s |
| Seattle | Sur de Brasil *   | 10 930 km | 189 ms |   8,2 Mbits/s |   699 Mbits/s |
| Seattle | Sur de la India      | 12 918 km | 202 ms |   7,7 Mbits/s |   634 Mbits/s |

\* La latencia de Brasil es un buen ejemplo de que la distancia lineal varía significativamente de la distancia del tenido de fibra. Esperaba que la latencia fuera de 160 ms aproximadamente, pero realmente es de 189 ms. Esta variación con lo que esperaba podría ser un indicio de que existe un problema de red en algún punto, pero lo más probable es que el tendido de fibra no llegue hasta Brasil en línea recta y que tenga una distancia de 1000 km o más para llegar a Brasil desde Seattle.

## <a name="next-steps"></a>Pasos siguientes
1. Descargue Azure Connectivity Toolkit de GitHub en [https://aka.ms/AzCT][ACT]
2. Siga las instrucciones para las [pruebas de rendimiento de los vínculos][Performance Doc]

<!--Image References-->
[1]: ./media/expressroute-troubleshooting-network-performance/network-components.png "Componentes de la red de Azure"
[2]: ./media/expressroute-troubleshooting-network-performance/expressroute-troubleshooting.png "Solución de problemas de ExpressRoute"
[3]: ./media/expressroute-troubleshooting-network-performance/test-diagram.png "Entorno de prueba de Perf"
[4]: ./media/expressroute-troubleshooting-network-performance/powershell-output.png "Salida de PowerShell"

<!--Link References-->
[Performance Doc]: https://github.com/Azure/NetworkMonitoring/blob/master/AzureCT/PerformanceTesting.md
[Availability Doc]: https://github.com/Azure/NetworkMonitoring/blob/master/AzureCT/AvailabilityTesting.md
[Network Docs]: https://docs.microsoft.com/azure/index
[Ticket Link]: https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview
[ACT]: https://aka.ms/AzCT
