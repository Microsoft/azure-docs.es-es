---
title: Sincronizar la hora en las máquinas virtuales de Windows en Azure
description: Sincronizar la hora en las máquinas virtuales de Windows.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 09/17/2018
ms.author: cynthn
ms.openlocfilehash: 830bdd45be4b0365ac45bc3ea366b99a34882a4c
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88871486"
---
# <a name="time-sync-for-windows-vms-in-azure"></a>Sincronizar la hora en las máquinas virtuales de Windows en Azure

La sincronización de la hora es importante para la seguridad y la correlación de eventos. A veces se usa en la implementación de transacciones distribuidas. La precisión de hora entre varios sistemas de equipos se logra mediante la sincronización. Además, la sincronización puede verse afectada por varias cosas, incluyendo las operaciones de reinicio y el tráfico de red entre el origen de la hora y el equipo encargado de capturar la hora. 

Por ello, Azure ya cuenta con la infraestructura que ejecuta Windows Server 2016. Windows Server 2016 ha mejorado los algoritmos que se usan para corregir la hora y la condición del reloj local, y así poder sincronizarse con la hora UTC.  Asimismo, Windows Server 2016 también ha mejorado el servicio VMICTimeSync que administra el modo de sincronizar las máquinas virtuales con el host, para obtener la hora exacta. Las mejoras incluyen una hora inicial más precisa en el inicio o la restauración de la máquina virtual, así como la interrupción de la corrección de la latencia para las muestras proporcionadas para la hora de Windows (W32time). 


>[!NOTE]
>Para obtener información general rápida del servicio Horario de Windows, eche un vistazo a este [vídeo de información general de alto nivel](https://aka.ms/WS2016TimeVideo).
>
> Para obtener más información, consulte el artículo [Hora de Windows Server 2016 precisa](/windows-server/networking/windows-time-service/accurate-time). 

## <a name="overview"></a>Información general

La precisión del reloj de un equipo se mide en función de lo cerca que esté ese reloj de la hora universal coordinada (UTC) estándar. La UTC se define mediante una muestra internacional de relojes atómicos precisos que solo se retrasan un segundo en 300 años. Aún así, es necesario tener hardware especializado para leer la UTC directamente. En cambio, los servidores horarios se sincronizan con la UTC y se accede a ellos desde otros equipos para proporcionar escalabilidad y robustez. Cada equipo tiene un servicio de sincronización de hora en ejecución que sabe qué servidores horarios debe usar y que comprueba periódicamente si el reloj del equipo necesita ser corregido ajustándolo si fuera necesario. 

Los hosts de Azure se sincronizan con los servidores horarios internos de Microsoft que ajustan su hora a partir de los dispositivos Stratum 1 que pertenecen a Microsoft, y que cuentan con antenas GPS. Las máquinas virtuales en Azure pueden depender de su host para proporcionar la hora exacta (*hora del host*) a la máquina virtual, o la máquina virtual puede obtener directamente la hora de un servidor horario, o de una combinación de ambas opciones. 

Las interacciones de la máquina virtual con el host también pueden afectar al reloj. Durante [el mantenimiento de conservación de la memoria](../maintenance-and-updates.md#maintenance-that-doesnt-require-a-reboot) las máquinas virtuales se detienen hasta 30 segundos. Por ejemplo, antes de que comience el proceso de mantenimiento que dura 28 segundos, el reloj de la máquina virtual muestra las 10:00:00 A.M. Cuando se reanuda la máquina virtual, el reloj de la misma todavía muestra las 10:00:00 A.M., por lo que tendría 28 segundos de retraso. Para corregir esto, el servicio VMICTimeSync supervisa lo que sucede en el host y solicita que se realicen cambios en las máquinas virtuales para compensar este retraso.

El servicio VMICTimeSync funciona en modo de ejemplo o sincronización y solamente influirá en el reloj desde ese momento en adelante. En el modo de ejemplo, que requiere que W32time esté en ejecución, el servicio VMICTimeSync sondea el host cada 5 segundos y proporciona ejemplos de tiempo a W32time. Aproximadamente cada 30 segundos, el servicio W32time toma la última muestra de tiempo y la utiliza para influir en el reloj del invitado. El modo de sincronización se activa si un invitado se ha reanudado o si el reloj de un invitado alcanza un retraso de más de cinco segundos respecto del reloj del host. Cuando el servicio W32time se ejecuta correctamente, este último caso no debe producirse nunca.

Por supuesto, si la sincronización de la hora no funciona, el reloj de la máquina virtual solo acumularía errores. Cuando solo hay una máquina virtual, este efecto puede no ser significativo a menos que la carga de trabajo requiera un cronometraje altamente preciso. Aún así, en la mayoría de los casos hay varias máquinas virtuales conectadas entre sí que usan la hora para realizar el seguimiento de las transacciones, por lo que esta hora debe ser consistente en toda la implementación. Cuando la hora de las máquinas virtuales es diferente, puede ver los siguientes efectos:

- Se produce un error en la autenticación. Los protocolos de seguridad como Kerberos o la tecnología dependiente de los certificados confían en que la hora sea consistente en todos los sistemas. 
- Es muy difícil averiguar qué ha ocurrido en un sistema si los registros (u otros datos) tienen horas diferentes. Parecería que el mismo evento ocurrió en diferentes momentos, lo que dificulta la correlación.
- Si el reloj está atrasado, es posible que la facturación no se calcule correctamente.

Los mejores resultados para las implementaciones de Windows se logran utilizando Windows Server 2016 como sistema operativo invitado, lo que garantiza que pueda usar las últimas mejoras en la sincronización horaria.

## <a name="configuration-options"></a>Opciones de configuración

Existen tres opciones para configurar la sincronización horaria de las máquinas virtuales de Windows alojadas en Azure:

- Hora del host y time.windows.com. Esta es la configuración predeterminada que se usa en las imágenes de Azure Marketplace.
- Solo para el host.
- Use otro servidor horario externo con o sin usar la hora del host.


### <a name="use-the-default"></a>Usar el valor predeterminado

De forma predeterminada, las imágenes de la máquina virtual del sistema operativo Windows están configuradas para que w32time se sincronice desde dos orígenes: 

- El proveedor NtpClient, que obtiene información de time.windows.com.
- El servicio VMICTimeSync, que se usa para comunicar la hora del host a las máquinas virtuales y hacer correcciones después de pausar la máquina virtual para el proceso de mantenimiento. Los hosts de Azure usan dispositivos Stratum 1 de Microsoft para mantener la exactitud de la hora.

Es preferible que el proveedor horario esté establecido en el siguiente orden de prioridad en w32time: nivel de capa, retraso de raíz, dispersión de raíz, desplazamiento de la hora. En la mayoría de los casos, W32Time en una máquina virtual de Azure prefiere el tiempo de host debido a la evaluación que haría para comparar ambos orígenes de hora. 

En cuanto a las máquinas unidas al dominio, el propio dominio establece una jerarquía de sincronización de hora, pero la raíz del bosque aún necesita saber la hora de alguna parte y las siguientes consideraciones seguirían contando con el valor "true".


### <a name="host-only"></a>Solo para el host 

Debido a que time.windows.com es un servidor NTP público, la hora de sincronización del mismo requiere enviar tráfico a través de Internet, por lo que los distintos retrasos en los paquetes que se envían pueden afectar negativamente la calidad de la sincronización de la hora. Si elimina time.windows.com y cambia a la sincronización solo para el host, puede mejorar los resultados de sincronización de hora.

Tenga en cuenta que solo merece la pena cambiar a la sincronización de hora solo para el host si tiene problemas al sincronizar la hora con la configuración predeterminada. Pruebe la sincronización solo para el host y compruebe si así mejora la sincronización de hora en la máquina virtual. 

### <a name="external-time-server"></a>Servidor de hora externo

Si tiene requisitos de sincronización horaria específica, también hay una opción que le permitirá usar servidores horarios externos. Los servidores horarios externos pueden proporcionar una hora específica, que puede ser útil para escenarios de prueba; gracias a ello, asegurará la uniformidad de la hora de las máquinas hospedadas en centros de datos que no sean de Microsoft, o podrá controlar los segundos intercalares de una manera especial.

Puede combinar servidores externos con el servicio VMICTimeSync y VMICTimeProvider para proporcionar resultados similares a los de la configuración predeterminada. 

## <a name="check-your-configuration"></a>Compruebe la configuración


Compruebe si el proveedor horario NtpClient está configurado para usar servidores NTP explícitos (NTP) o la sincronización de hora del dominio (NT5DS).

```
w32tm /dumpreg /subkey:Parameters | findstr /i "type"
```

Si la máquina virtual está usando NTP, verá el siguiente resultado:

```
Value Name                 Value Type          Value Data
Type                       REG_SZ              NTP
```

Para ver a qué servidor horario está usando el proveedor horario NtpClient, escriba lo siguiente en un símbolo del sistema con privilegios elevados:

```
w32tm /dumpreg /subkey:Parameters | findstr /i "ntpserver"
```

Si la máquina virtual está usando el valor predeterminado, el resultado tendrá el siguiente aspecto:

```
NtpServer                  REG_SZ              time.windows.com,0x8
```


Para ver qué proveedor horario está utilizando actualmente.

```
w32tm /query /source
```


Estos son los resultados que podría ver y lo que significan:
    
- **time.windows.com**: en la configuración predeterminada, w32time obtiene la hora de time.windows.com. La calidad de la sincronización de la hora depende de la conectividad de Internet, y se ve afectada por los retrasos de los paquetes. Esta es la salida habitual que obtendría en una máquina física.
- **Proveedor de sincronización de hora de IC de la máquina virtual**: la máquina virtual sincroniza la hora desde el host. Esta es la salida habitual que obtendría en una máquina virtual que se ejecuta en Azure. 
- *Su servidor de dominio*: la máquina actual está en un dominio y el dominio define la jerarquía de sincronización de hora.
- *Algún otro servidor*: w32time se configuró explícitamente para obtener la hora de otro servidor. La calidad de la sincronización de la hora depende de la calidad del servidor horario.
- **Reloj local CMOS**: el reloj no está sincronizado. Puede obtener este resultado si w32time no ha tenido tiempo suficiente para iniciarse tras un reinicio, o cuando no están disponibles todos los orígenes de la hora configurados.


## <a name="opt-in-for-host-only-time-sync"></a>Habilitar la sincronización de hora desde un único host

Azure trabaja constantemente para mejorar la sincronización de la hora en los hosts y puede garantizar que toda la infraestructura de sincronización de hora se encuentra en centros de datos propiedad de Microsoft. Si tiene problemas con la sincronización horaria de la configuración predeterminada porque usa time.windows.com como fuente principal para obtener la hora, puede usar los siguientes comandos para habilitar la sincronización horaria des de un único host.

Marque el proveedor de VMIC como habilitado. 

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\TimeProviders\VMICTimeProvider /v Enabled /t REG_DWORD /d 1 /f
```

Marque el proveedor NTPClient como deshabilitado.

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\TimeProviders\NtpClient /v Enabled /t REG_DWORD /d 0 /f
```

Reinicie el servicio w32time.

```
net stop w32time && net start w32time
```


## <a name="windows-server-2012-and-r2-vms"></a>Máquinas virtuales de Windows Server 2012 y R2 

Windows Server 2012 y Windows Server 2012 R2 tienen diferentes configuraciones predeterminadas para la sincronización horaria. El servicio w32time se configura por defecto para obtener una baja sobrecarga del servicio en lugar de la hora precisa. 

Si desea mover sus implementaciones de Windows Server 2012 y 2012 R2 para que usen los valores predeterminados más recientes y que así antepongan la precisión horaria, puede aplicar la siguiente configuración.

Actualice el sondeo de w32time y actualice los intervalos para que coincidan con la configuración de Windows Server 2016.

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\Config /v MinPollInterval /t REG_DWORD /d 6 /f
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\Config /v MaxPollInterval /t REG_DWORD /d 10 /f
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\Config /v UpdateInterval /t REG_DWORD /d 100 /f
w32tm /config /update
```

Para que w32time pueda usar los nuevos intervalos de sondeo, es necesario que las instancias de NtpServers estén marcadas con su uso. Si los servidores están anotados con una máscara de indicador de bits de 0x1, esto anularía este mecanismo y w32time usaría SpecialPollInterval en su lugar. Asegúrese de que los servidores NTP especificados estén utilizando una marca 0x8 o ninguna marca:

Compruebe las marcas que se utilizan para los servidores NTP.

```
w32tm /dumpreg /subkey:Parameters | findstr /i "ntpserver"
```

## <a name="next-steps"></a>Pasos siguientes

A continuación le ofrecemos varios vínculos para obtener más detalles sobre la sincronización horaria:

- [Windows Time Service Tools and Settings](/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings) (Herramientas y configuración del servicio de hora de Windows)
- [Windows Server 2016 Improvements ](/windows-server/networking/windows-time-service/windows-server-2016-improvements) (Mejoras de Windows Server 2016)
- [Accurate Time for Windows Server 2016](/windows-server/networking/windows-time-service/accurate-time) (Hora exacta para Windows Server 2016)
- [Support boundary to configure the Windows Time service for high-accuracy environments](/windows-server/networking/windows-time-service/support-boundary) (Límite del soporte técnico para configurar el servicio de hora de Windows en entornos de alta precisión)
