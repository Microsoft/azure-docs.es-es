---
title: Solución de problemas de un entorno de ejecución de integración autohospedado en Azure Data Factory
description: Conozca cómo solucionar problemas del entorno de ejecución de integración autohospedado en Azure Data Factory.
services: data-factory
author: nabhishek
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 08/05/2020
ms.author: abnarain
ms.openlocfilehash: 49d173e0d0f2b96c385b4325335483d25e9a7c2d
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87800720"
---
# <a name="troubleshoot-self-hosted-integration-runtime"></a>Solución de problemas del entorno de ejecución de integración autohospedado

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se exploran los métodos comunes de solución de problemas del entorno de ejecución de integración autohospedado en Azure Data Factory.

## <a name="gather-self-hosted-ir-logs-from-azure-data-factory"></a>Recopilación de registros de IR autohospedado en Azure Data Factory

En el caso de las actividades con error que se ejecutan en IR autohospedado o IR compartido, Azure Data Factory admite la visualización y carga de registros de errores. Puede seguir los pasos a continuación para obtener el identificador del informe de errores y, a continuación, escribir el identificador del informe para buscar problemas conocidos relacionados.

1. Vaya a la página **Ejecuciones de actividad**.

1. En la columna **ERROR**, haga clic en el botón siguiente.

    ![Página Ejecuciones de actividad](media/self-hosted-integration-runtime-troubleshoot-guide/activity-runs-page.png)

1. Verá registros relacionados para la ejecución de actividad con errores. Haga clic en el botón **Enviar registros** para obtener más ayuda.

    ![Enviar registros](media/self-hosted-integration-runtime-troubleshoot-guide/send-logs.png)

1. Puede elegir los registros que desea enviar. Par *IR autohospedado*, puede cargar los registros relacionados con la actividad con errores o todos los registros del nodo de IR autohospedado. Para *IR compartido*, solo puede cargar los registros relacionados con la actividad con errores.

    ![Elección de registros](media/self-hosted-integration-runtime-troubleshoot-guide/choose-logs.png)

1. Cuando se carguen los registros, conserve un registro del identificador del informe si necesita ayuda adicional para resolver el problema.

    ![Carga de registros](media/self-hosted-integration-runtime-troubleshoot-guide/upload-logs.png)

> [!NOTE]
> Las solicitudes de visualización y carga de registros se ejecutarán en todas las instancias de IR autohospedado en línea. Asegúrese de que todas las instancias de IR autohospedado estén en línea en caso de que falten registros. 


## <a name="self-hosted-ir-general-failure-or-error"></a>Error general de IR autohospedado

### <a name="tlsssl-certificate-issue"></a>Problema de certificado TLS/SSL

#### <a name="symptoms"></a>Síntomas

Al intentar habilitar el certificado TLS/SSL (avanzado) desde **Configuration Manager de IR autohospedado** -> **Acceso remoto desde la intranet**, después de seleccionar el certificado TLS/SSL, se muestra el siguiente error:

`Remote access settings are invalid. Identity check failed for outgoing message. The expected DNS identity of the remote endpoint was ‘abc.microsoft.com’ but the remote endpoint provided DNS claim ‘microsoft.com’. If this is a legitimate remote endpoint, you can fix the problem by explicitly specifying DNS identity ‘microsoft.com’ as the Identity property of EndpointAddress when creating channel proxy.`

En el caso anterior, el usuario usa el certificado con "microsoft.com" como último elemento.

#### <a name="cause"></a>Causa

Este es un problema conocido en WCF: La validación TLS/SSL de WCF solo comprueba el último nombre de DNS en SAN. 

#### <a name="resolution"></a>Solución

El certificado comodín es compatible con IR autohospedado de Azure Data Factory v2. Este problema se produce normalmente porque el certificado SSL no es correcto. El último nombre de DNS en SAN debe ser válido. Siga los pasos que se indican a continuación para comprobarlo. 
1.  Abra la consola de administración y compruebe el *asunto* y el *nombre alternativo del firmante* en los detalles del certificado. En el caso anterior, por ejemplo, el último elemento de *nombre alternativo del firmante*, que es "Nombre de DNS = microsoft.com.com", no es legítimo.
2.  Póngase en contacto con la empresa del problema del certificado para quitar el nombre DNS incorrecto.

### <a name="concurrent-jobs-limit-issue"></a>Problema de límite de trabajos simultáneos

#### <a name="symptoms"></a>Síntomas

Al intentar aumentar el límite de trabajos simultáneos desde la interfaz de usuario de Azure Data Factory, se bloquea como si se estuviese *actualizando* eternamente.
El valor máximo de trabajos simultáneos se estableció en 24 y quiere aumentar el número total para que los trabajos se puedan ejecutar con mayor rapidez. El valor mínimo que se puede especificar es 3 y el valor máximo que se puede especificar es 32. Ha aumentado el valor de 24 a 32 y, al pulsar el botón *Actualizar* en la interfaz de usuario, se queda atascado en *Actualizando* como se puede ver a continuación. Después de la actualización, el cliente todavía ve el valor como 24 y nunca se actualizó a 32.

![Estado Actualizando](media/self-hosted-integration-runtime-troubleshoot-guide/updating-status.png)

#### <a name="cause"></a>Causa

Existe una limitación para la configuración, ya que el valor depende del equipo logicCore y de la memoria. Simplemente, puede ajustarlo a un valor menor, como 24, y ver el resultado.

> [!TIP] 
> - Para obtener más información sobre el número de núcleos lógicos y cómo encontrar el número de núcleos lógicos de la máquina, consulte [este artículo](https://www.top-password.com/blog/find-number-of-cores-in-your-cpu-on-windows-10/).
> - Para obtener más información sobre cómo calcular math.log, consulte [este artículo](https://www.rapidtables.com/calc/math/Log_Calculator.html).


### <a name="self-hosted-ir-ha-ssl-certificate-issue"></a>Problema de certificado SSL de alta disponibilidad de IR autohospedado

#### <a name="symptoms"></a>Síntomas

El nodo de trabajo de IR autohospedado ha notificado el siguiente error:

`Failed to pull shared states from primary node net.tcp://abc.cloud.corp.Microsoft.com:8060/ExternalService.svc/. Activity ID: XXXXX The X.509 certificate CN=abc.cloud.corp.Microsoft.com, OU=test, O=Microsoft chain building failed. The certificate that was used has a trust chain that cannot be verified. Replace the certificate or change the certificateValidationMode. The revocation function was unable to check revocation because the revocation server was offline.`

#### <a name="cause"></a>Causa

Cuando tratamos casos relacionados con el protocolo de enlace SSL/TLS, es posible que surjan algunos problemas relacionados con la comprobación de la cadena de certificados. 

#### <a name="resolution"></a>Solución

- A continuación, se muestra una manera rápida e intuitiva de solucionar los errores de compilación de la cadena de certificados X.509.
 
    1. Exporte el certificado, que debe comprobarse. Vaya a Administrar certificado de equipo, busque el certificado que quiere comprobar y haga clic con el botón derecho en **Todas las tareas** -> **Exportar**.
    
        ![Exportar tareas](media/self-hosted-integration-runtime-troubleshoot-guide/export-tasks.png)

    2. Copie el certificado exportado en el equipo cliente. 
    3. En el lado cliente, ejecute el comando siguiente en CMD. Asegúrese de que ha reemplazado los marcadores de posición *\<certificate path>* y *\<output txt file path>* por las rutas de acceso relacionadas.
    
        ```
        Certutil -verify -urlfetch    <certificate path>   >     <output txt file path> 
        ```

        Por ejemplo:

        ```
        Certutil -verify -urlfetch c:\users\test\desktop\servercert02.cer > c:\users\test\desktop\Certinfo.txt
        ```
    4. Compruebe si hay algún error en el archivo txt de salida. Puede encontrar el resumen de errores al final del archivo txt.

        Por ejemplo: 

        ![Resumen de errores](media/self-hosted-integration-runtime-troubleshoot-guide/error-summary.png)

        Si no ve ningún error al final del archivo de registro, como se muestra a continuación, puede considerar que la cadena de certificados se ha compilado correctamente en el equipo cliente.
        
        ![No hay ningún error en el archivo de registro](media/self-hosted-integration-runtime-troubleshoot-guide/log-file.png)      

- Si tiene AIA, CDP y OCSP configurados en el archivo de certificado. Podemos comprobarlo de una forma más intuitiva.
 
    1. Puede obtener esta información comprobando los detalles de un certificado.
    
        ![Detalles del certificado](media/self-hosted-integration-runtime-troubleshoot-guide/certificate-detail.png)
    1. Ejecute el comando siguiente. Asegúrese de que ha reemplazado el marcador de posición *\<certificate path>* por la ruta de acceso relacionada del certificado.
    
        ```
          Certutil   -URL    <certificate path> 
        ```
    1. A continuación, se abrirá la **herramienta de recuperación de URL**. Para comprobar los certificados de AIA, CDP y OCSP, haga clic en el botón **Recuperar**.

        ![Botón de recuperación](media/self-hosted-integration-runtime-troubleshoot-guide/retrieval-button.png)
 
        La cadena de certificados se puede compilar correctamente si el certificado de AIA está "Comprobado" y el certificado de CDP o OCSP está "Comprobado".

        Si se produce un error al recuperar AIA y CDP, trabaje con el equipo de red para preparar el equipo cliente para conectarse a la dirección URL de destino. Será suficiente si se puede comprobar la ruta de acceso http o la ruta de acceso LDAP.

### <a name="self-hosted-ir-could-not-load-file-or-assembly"></a>IR autohospedado no pudo cargar el archivo o ensamblado

#### <a name="symptoms"></a>Síntomas

`Could not load file or assembly 'XXXXXXXXXXXXXXXX, Version=4.0.2.0, Culture=neutral, PublicKeyToken=XXXXXXXXX' or one of its dependencies. The system cannot find the file specified. Activity ID: 92693b45-b4bf-4fc8-89da-2d3dc56f27c3`
 
Por ejemplo: 

`Could not load file or assembly 'System.ValueTuple, Version=4.0.2.0, Culture=neutral, PublicKeyToken=XXXXXXXXX' or one of its dependencies. The system cannot find the file specified. Activity ID: 92693b45-b4bf-4fc8-89da-2d3dc56f27c3`

#### <a name="cause"></a>Causa

Si observa el monitor de procesos, puede ver el siguiente resultado:

[![Monitor de procesos](media/self-hosted-integration-runtime-troubleshoot-guide/process-monitor.png)](media/self-hosted-integration-runtime-troubleshoot-guide/process-monitor.png#lightbox)

> [!TIP] 
> Puede establecer un filtro como se muestra en la captura de pantalla siguiente.
> Nos indica que el archivo dll **System.ValueTuple** no se encuentra en la carpeta relacionada con GAC ni en las carpetas *C:\Program Files\Microsoft Integration Runtime\4.0\Gateway* o *C:\Program Files\Microsoft Integration Runtime\4.0\Shared*.
> Básicamente, cargará el archivo dll desde la carpeta *GAC* en primer lugar, a continuación, desde la carpeta *Shared* y, por último, desde la carpeta *Gateway*. Por lo tanto, puede colocar el archivo dll en cualquier ruta de acceso que pueda ser útil.

![Configuración de los filtros](media/self-hosted-integration-runtime-troubleshoot-guide/set-filters.png)

#### <a name="resolution"></a>Solución

Puede ver que **System.ValueTuple.dll** se encuentra en la carpeta *C:\Program Files\Microsoft Integration Runtime\4.0\Gateway\DataScan*. Copie **System.ValueTuple.dll** en la carpeta *C:\Program Files\Microsoft Integration Runtime\4.0\Gateway* para resolver el problema.

Puede usar el mismo método para resolver otros problemas de archivos o ensamblados que faltan.

#### <a name="more-information"></a>Más información

La razón por la que se ve el archivo System.ValueTuple.dll en *%windir%\Microsoft.NET\assembly* y *%windir%\assembly* es que es un comportamiento de .NET. 

En el siguiente error, puede ver claramente que el ensamblado *System.ValueTuple* no está ahí. Por lo tanto, este problema se produce cuando la aplicación intenta comprobar el ensamblado *System.ValueTuple.dll*.
 
`<LogProperties><ErrorInfo>[{"Code":0,"Message":"The type initializer for 'Npgsql.PoolManager' threw an exception.","EventType":0,"Category":5,"Data":{},"MsgId":null,"ExceptionType":"System.TypeInitializationException","Source":"Npgsql","StackTrace":"","InnerEventInfos":[{"Code":0,"Message":"Could not load file or assembly 'System.ValueTuple, Version=4.0.2.0, Culture=neutral, PublicKeyToken=XXXXXXXXX' or one of its dependencies. The system cannot find the file specified.","EventType":0,"Category":5,"Data":{},"MsgId":null,"ExceptionType":"System.IO.FileNotFoundException","Source":"Npgsql","StackTrace":"","InnerEventInfos":[]}]}]</ErrorInfo></LogProperties>`
 
Para obtener más información sobre GAC, consulte [este artículo](https://docs.microsoft.com/dotnet/framework/app-domains/gac).


### <a name="how-to-audit-self-hosted-ir-key-missing"></a>Cómo auditar la clave de IR autohospedada que falta

#### <a name="symptoms"></a>Síntomas

El entorno de ejecución de integración autohospedado se desconecta repentinamente sin clave y se muestra el mensaje de error en el registro de eventos: `Authentication Key is not assigned yet`

![Falta la clave de autenticación](media/self-hosted-integration-runtime-troubleshoot-guide/key-missing.png)

#### <a name="cause"></a>Causa

- Se elimina el nodo de IR autohospedado o el IR autohospedado lógico en el portal.
- Se realiza una desinstalación limpia.

#### <a name="resolution"></a>Solución

Si no se cumple ninguna de las causas anteriores, puede ir a la carpeta *%programdata%\Microsoft\Data Transfer\DataManagementGateway* y comprobar si se ha eliminado el archivo denominado **Configuraciones**. Si se ha eliminado, siga las instrucciones que se indican [aquí](https://www.netwrix.com/how_to_detect_who_deleted_file.html) para auditar quién elimina el archivo.

![Comprobación del archivo de configuraciones](media/self-hosted-integration-runtime-troubleshoot-guide/configurations-file.png)


### <a name="cannot-use-self-hosted-ir-to-bridge-two-on-premises-data-stores"></a>No se puede usar IR autohospedado para enlazar dos almacenes de datos locales

#### <a name="symptoms"></a>Síntomas

Después de crear IR autohospedados para los almacenes de datos de origen y de destino, querrá conectar los dos IR juntos para finalizar una copia. Si los almacenes de datos están configurados en redes virtuales diferentes o no pueden entender el mecanismo de puerta de enlace, se producirán errores como: *no se encuentra el controlador de origen en el IR de destino*; *el origen no puede tener acceso al IR de destino*.
 
#### <a name="cause"></a>Causa

IR autohospedado está diseñado como un nodo central de una actividad de copia, no un agente cliente que debe instalarse para cada almacén de datos.
 
En el caso anterior, el servicio vinculado para cada almacén de datos debe crearse con el mismo IR y este debe poder tener acceso a ambos almacenes de datos a través de la red. Con independencia de que IR esté instalado con el almacén de datos de origen, el almacén de datos de destino o en una tercera máquina, si se crean dos servicios vinculados con IR diferentes, pero se usan en la misma actividad de copia, se usará el destino IR y se deben instalar los controladores de ambos almacenes de datos en la máquina IR de destino.

#### <a name="resolution"></a>Solución

Instale los controladores para el origen y el destino en el IR de destino y asegúrese de que puede tener acceso al almacén de datos de origen.
 
Si el tráfico no puede pasar a través de la red entre dos almacenes de datos (por ejemplo, están configurados en dos redes virtuales), es posible que no finalice la copia en una actividad incluso con IR instalado. En ese caso, puede crear dos actividades de copia con dos IR, cada una en una red virtual: un IR para copiar del almacén de datos 1 a Azure Blob Storage, otro para copiar de Azure Blob Storage al almacén de datos 2. Esto podría simular el requisito de usar el IR para crear un puente que conecte dos almacenes de datos desconectados.


### <a name="credential-sync-issue-causes-credential-lost-from-ha"></a>El problema de sincronización de credenciales provoca la pérdida de credenciales de alta disponibilidad

#### <a name="symptoms"></a>Síntomas

Se han eliminado las credenciales del origen de datos "XXXXXXXXXX" del nodo actual de Integration Runtime con carga "al eliminar el servicio de vínculo en Azure Portal, o bien la tarea tiene una carga que no es correcta. Vuelva a crear un servicio de vínculo con las credenciales".

#### <a name="cause"></a>Causa

El IR autohospedado está integrado en el modo de alta disponibilidad con dos nodos, pero no en el estado de sincronización de credenciales, lo que significa que las credenciales almacenadas en el nodo del distribuidor no se sincronizan con otros nodos de trabajo. Si se produce una conmutación por error del nodo distribuidor al nodo de trabajo, pero las credenciales solo existían en el nodo distribuidor anterior, se producirá un error en la tarea al intentar obtener acceso a las credenciales y se alcanzará el error anterior.

#### <a name="resolution"></a>Solución

La única manera de evitar este problema es asegurarse de que los dos nodos están en el estado de sincronización de credenciales. De lo contrario, tendrá que reescribir las credenciales para el nuevo distribuidor.


### <a name="cannot-choose-the-certificate-due-to-private-key-missing"></a>No se puede elegir el certificado porque falta la clave privada

#### <a name="symptoms"></a>Síntomas

1.  Importe un archivo PFX al almacén de certificados.
2.  Al seleccionar el certificado a través de la interfaz de usuario de Configuration Manager de IR, se produce el siguiente error:

    ![Falta la clave privada](media/self-hosted-integration-runtime-troubleshoot-guide/private-key-missing.png)

#### <a name="cause"></a>Causa

- La cuenta de usuario tiene pocos privilegios y no puede tener acceso a la clave privada.
- El certificado se generó como firma, pero no como intercambio de claves.

#### <a name="resolution"></a>Solución

1.  Use una cuenta con privilegios que pueda tener acceso a la clave privada para que funcione la interfaz de usuario.
2.  Ejecute el comando siguiente para importar el certificado:
    
    ```
    certutil -importpfx FILENAME.pfx AT_KEYEXCHANGE
    ```


## <a name="self-hosted-ir-setup"></a>Configuración de IR autohospedado

### <a name="the-integration-runtime-registration-error"></a>Error de registro de Integration Runtime 

#### <a name="symptoms"></a>Síntomas

A veces, queremos ejecutar IR autohospedado en una cuenta diferente por los motivos siguientes:
- La directiva de la compañía no permite la cuenta de servicio.
- Se requiere autenticación.

Después de cambiar la cuenta de servicio en el panel de servicio, es posible que Integration Runtime deje de funcionar.

![Error de registro de IR](media/self-hosted-integration-runtime-troubleshoot-guide/ir-registration-error.png)

#### <a name="cause"></a>Causa

Hay muchos recursos que solo se conceden a la cuenta de servicio. Al cambiar la cuenta de servicio a otra cuenta, el permiso de todos los recursos dependientes sigue siendo el mismo.

#### <a name="resolution"></a>Solución

Vaya al registro de eventos de Integration Runtime para comprobar el error.

![Registro de eventos de IR](media/self-hosted-integration-runtime-troubleshoot-guide/ir-event-log.png)

Si el error se muestra como en el ejemplo anterior, *UnauthorizedAccessException*, siga las instrucciones siguientes:


1. Compruebe la cuenta del servicio de inicio de sesión de *DIAHostService* en el panel del servicio de Windows.

    ![Cuenta de servicio de inicio de sesión](media/self-hosted-integration-runtime-troubleshoot-guide/logon-service-account.png)

2. Compruebe si la cuenta de servicio de inicio de sesión tiene el permiso de lectura y escritura en la carpeta *%programdata%\Microsoft\DataTransfer\DataManagementGateway*.

    - De manera predeterminada, si no se ha cambiado la cuenta de inicio de sesión del servicio, debe tener el permiso de lectura y escritura.

        ![Permiso del servicio](media/self-hosted-integration-runtime-troubleshoot-guide/service-permission.png)

    - Si ha cambiado la cuenta de inicio de sesión del servicio, siga los pasos siguientes para mitigar el problema:
        1. Realice una desinstalación limpia de la instancia de IR autohospedado actual.
        1. Instale los bits de IR autohospedado.
        1. Siga las instrucciones siguientes para cambiar la cuenta de servicio: 
            1. Vaya a la carpeta de instalación de IR autohospedado y cambie a la carpeta: *Microsoft Integration Runtime\4.0\Shared*.
            1. Inicie una línea de comandos con privilegio elevado. Reemplace *\<user>* y *\<password>* por su propio nombre de usuario y contraseña y, a continuación, ejecute el comando siguiente:
                       
                ```
                dmgcmd.exe -SwitchServiceAccount "<user>" "<password>"
                ```
            1. Si quiere cambiar a una cuenta LocalSystem, asegúrese de usar un formato correcto para esta cuenta. A continuación, se muestra un ejemplo del formato correcto:

                ```
                dmgcmd.exe -SwitchServiceAccount "NT Authority\System" ""
                ```         
                **No** use el formato como se muestra a continuación:

                ```
                dmgcmd.exe -SwitchServiceAccount "LocalSystem" ""
                ```              
            1. Como alternativa, dado que el sistema local tiene más privilegios que el administrador, también puede cambiarlo directamente en "Servicios".
            1. Puede utilizar un usuario local o de dominio para la cuenta de inicio de sesión del servicio de IR.            
        1. Registre Integration Runtime.

Si el error se muestra del siguiente modo: *No se pudo iniciar el "servicio de Integration Runtime" (DIAHostService). Compruebe que dispone de suficientes privilegios para iniciar servicios del sistema*, siga las instrucciones siguientes:

1. Compruebe la cuenta del servicio de inicio de sesión de *DIAHostService* en el panel del servicio de Windows.
   
    ![Cuenta de servicio de inicio de sesión](media/self-hosted-integration-runtime-troubleshoot-guide/logon-service-account.png)

2. Compruebe si la cuenta de servicio de inicio de sesión tiene el permiso **Iniciar sesión como servicio** para iniciar el servicio de Windows:

    ![Inicio de sesión como servicio](media/self-hosted-integration-runtime-troubleshoot-guide/logon-as-service.png)

#### <a name="more-information"></a>Más información

Si ninguno de los dos patrones anteriores de la resolución se aplican en su caso, intente recopilar los siguientes registros de eventos de Windows: 
- Registros de aplicaciones y servicios > Integration Runtime
- Registros de Windows > Aplicación

### <a name="cannot-find-register-button-to-register-a-self-hosted-ir"></a>No se puede encontrar el botón de registro para registrar un IR autohospedado    

#### <a name="symptoms"></a>Síntomas

No se encontró el botón **Registrar** en la interfaz de usuario de Configuration Manager al registrar un IR autohospedado.

![No aparece el botón de registro](media/self-hosted-integration-runtime-troubleshoot-guide/no-register-button.png)

#### <a name="cause"></a>Causa

Desde el lanzamiento de *Integration Runtime 3.0*, se ha quitado el botón **Registrar** en un nodo de Integration Runtime existente para habilitar un entorno más limpio y seguro. Si un nodo se ha registrado en alguna instancia de Integration Runtime (ya sea en línea o no), para volver a registrarlo en otra instancia de Integration Runtime, debe desinstalar el nodo anterior y, a continuación, instalar y registrar el nodo.

#### <a name="resolution"></a>Solución

1. Vaya al panel de control para desinstalar la instancia de Integration Runtime existente.

    > [!IMPORTANT] 
    > En el proceso siguiente, seleccione Sí. No mantenga los datos durante el proceso de desinstalación.

    ![Eliminación de datos](media/self-hosted-integration-runtime-troubleshoot-guide/delete-data.png)

1. Si no tiene el MSI del instalador de Integration Runtime, vaya al [centro de descarga](https://www.microsoft.com/en-sg/download/details.aspx?id=39717) para descargar la instancia de Integration Runtime más reciente.
1. Instale el MSI y registre Integration Runtime.


### <a name="unable-to-register-the-self-hosted-ir-due-to-localhost"></a>No se puede registrar el IR autohospedado debido a localhost    

#### <a name="symptoms"></a>Síntomas

No se puede registrar el IR autohospedado en una máquina nueva cuando se obtiene get_LoopbackIpOrName.

**Depuración:** Se ha producido un error de tiempo de ejecución.
El inicializador de tipo para "Microsoft.DataTransfer.DIAgentHost.DataSourceCache" produjo una excepción.
Se produjo un error no recuperable durante una búsqueda de base de datos.
 
**Detalle de la excepción:** System.TypeInitializationException: El inicializador de tipo para "Microsoft.DataTransfer.DIAgentHost.DataSourceCache" produjo una excepción. ---> System.Net.Sockets.SocketException: Se produjo un error no recuperable durante una búsqueda de base de datos en System.Net.Dns.GetAddrInfo(nombre de la cadena).

#### <a name="cause"></a>Causa

Normalmente, el problema se produce al resolver localhost.

#### <a name="resolution"></a>Solución

Use Localhost 127.0.0.1 para hospedar el archivo y resolver este problema.


### <a name="self-hosted-setup-failed"></a>Error al instalar IR autohospedado    

#### <a name="symptoms"></a>Síntomas

No se puede desinstalar un IR existente, instalar un IR nuevo ni actualizar un IR existente a un IR nuevo.

#### <a name="cause"></a>Causa

La instalación depende del servicio de Windows Installer. Hay diferentes motivos que pueden causar problemas de instalación:
- No hay suficiente espacio en disco.
- Faltan permisos.
- El servicio NT está bloqueado por alguna razón.
- Hay un uso de la CPU demasiado alto.
- El archivo MSI se hospeda en una ubicación de red lenta.
- Algunos archivos del sistema o registros se tocan de forma no intencionada.


## <a name="self-hosted-ir-connectivity-issues"></a>Problemas de conectividad de IR autohospedado

### <a name="self-hosted-integration-runtime-cant-connect-to-cloud-service"></a>El entorno de ejecución de integración autohospedado no puede conectarse al servicio en la nube.

#### <a name="symptoms"></a>Síntomas

![Problema de conexión de IR autohospedado](media/self-hosted-integration-runtime-troubleshoot-guide/unable-to-connect-to-cloud-service.png)

#### <a name="cause"></a>Causa 

El entorno de ejecución de integración autohospedado no puede conectarse al servicio Data Factory (back-end). Este problema se debe normalmente a la configuración de red en el firewall.

#### <a name="resolution"></a>Solución

1. Compruebe si el servicio de entorno de ejecución de integración está en ejecución.
    
   ![Estado de ejecución del servicio IR autohospedado](media/self-hosted-integration-runtime-troubleshoot-guide/integration-runtime-service-running-status.png)
    
1. Si el servicio se está ejecutando, vaya al paso 3.

1. Si no hay ningún proxy configurado en el entorno de ejecución de integración autohospedado (la configuración predeterminada), ejecute el siguiente comando de PowerShell en la máquina donde está instalado el entorno de ejecución de integración autohospedado:

    ```powershell
    (New-Object System.Net.WebClient).DownloadString("https://wu2.frontend.clouddatahub.net/")
    ```
        
   > [!NOTE]     
   > La dirección URL del servicio puede variar en función de la ubicación de Data Factory. Puede encontrar la dirección URL del servicio en **ADF UI** > **Connections** > **Integration runtimes** > **Edit Self-hosted IR** > **Nodes** > **View Service URLs** (Interfaz de usuario de ADF > Conexiones > Entornos de ejecución de integración > Editar IR autohospedado > Nodos > Ver direcciones URL del servicio).
            
    La respuesta esperada es la siguiente:
            
    ![Respuesta del comando de PowerShell](media/self-hosted-integration-runtime-troubleshoot-guide/powershell-command-response.png)
            
1. Si no recibe la respuesta esperada, use uno de los métodos siguientes, según corresponda a su situación:
            
    * Si recibe el mensaje "No se pudo resolver el nombre remoto", hay un problema con el sistema de nombres de dominio (DNS). Póngase en contacto con su equipo de redes para solucionar este problema.
    * Si recibe el error "El certificado SSL/TLS no es de confianza", compruebe si el certificado de https://wu2.frontend.clouddatahub.net/ es de confianza en la máquina y luego instale el certificado público mediante el Administrador de certificados. Esta acción debería mitigar el problema.
    * Vaya **Windows** > **Visor de eventos (registros)**  > **Registros de aplicaciones y servicios** > **Integration Runtime** y compruebe si hay errores causados por DNS, una regla de firewall o la configuración de red de la empresa. (Si encuentra este tipo de error, fuerce el cierre de la conexión). Dado que cada compañía tiene una configuración de red personalizada, póngase en contacto con su equipo de red para solucionar estos problemas.

1. Si el proxy se ha configurado en el entorno de ejecución de integración autohospedado, compruebe si el servidor proxy puede acceder al punto de conexión de servicio. Para ver un comando de ejemplo, consulte [PowerShell, solicitudes web y servidores proxy](https://stackoverflow.com/questions/571429/powershell-web-requests-and-proxies).    
                
    ```powershell
    $user = $env:username
    $webproxy = (get-itemproperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet
    Settings').ProxyServer
    $pwd = Read-Host "Password?" -assecurestring
    $proxy = new-object System.Net.WebProxy
    $proxy.Address = $webproxy
    $account = new-object System.Net.NetworkCredential($user,[Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($pwd)), "")
    $proxy.credentials = $account
    $url = "https://wu2.frontend.clouddatahub.net/"
    $wc = new-object system.net.WebClient
    $wc.proxy = $proxy
    $webpage = $wc.DownloadData($url)
    $string = [System.Text.Encoding]::ASCII.GetString($webpage)
    $string
    ```

La respuesta esperada es la siguiente:
            
![Respuesta del comando de PowerShell 2](media/self-hosted-integration-runtime-troubleshoot-guide/powershell-command-response.png)

> [!NOTE] 
> Consideraciones del proxy:
> *    Compruebe si el servidor proxy debe colocarse en la lista de destinatarios seguros. Si es así, asegúrese de que [estos dominios](https://docs.microsoft.com/azure/data-factory/data-movement-security-considerations#firewall-requirements-for-on-premisesprivate-network) estén en la lista de destinatarios seguros.
> *    Compruebe que el certificado TLS/SSL de "wu2.frontend.clouddatahub.net/" es de confianza en el servidor proxy.
> *    Si usa la autenticación de Active Directory en el proxy, cambie la cuenta de servicio por la cuenta de usuario que pueda acceder al proxy como "Integration Runtime Service".

### <a name="error-message-self-hosted-integration-runtime-node-logical-shir-is-in-inactive-running-limited-state"></a>Mensaje de error: El nodo del entorno de ejecución de integración autohospedado/SHIR lógico está en estado inactivo/"en ejecución (limitado)"

#### <a name="cause"></a>Causa 

El nodo de tiempo de ejecución integrado autohospedado podría tener un estado **inactivo**, como se muestra en la siguiente captura de pantalla:

![Nodo de IR autohospedado inactivo](media/self-hosted-integration-runtime-troubleshoot-guide/inactive-self-hosted-ir-node.png)

Este comportamiento se produce cuando los nodos no pueden comunicarse entre sí.

#### <a name="resolution"></a>Solución

1. Inicie sesión en la máquina virtual hospedada en el nodo. En **Registros de aplicaciones y servicios** > **Integration Runtime**, abra el visor de eventos y filtre todos los registros de errores.

1. Compruebe si el registro de errores contiene el error siguiente: 
    
    ```
    System.ServiceModel.EndpointNotFoundException: Could not connect to net.tcp://xxxxxxx.bwld.com:8060/ExternalService.svc/WorkerManager. The connection attempt lasted for a time span of 00:00:00.9940994. TCP error code 10061: No connection could be made because the target machine actively refused it 10.2.4.10:8060. 
    System.Net.Sockets.SocketException: No connection could be made because the target machine actively refused it. 
    10.2.4.10:8060
    at System.Net.Sockets.Socket.DoConnect(EndPoint endPointSnapshot, SocketAddress socketAddress)
    at System.Net.Sockets.Socket.Connect(EndPoint remoteEP)
    at System.ServiceModel.Channels.SocketConnectionInitiator.Connect(Uri uri, TimeSpan timeout)
    ```
       
1. Si ve este error, ejecute lo siguiente en una línea de comandos: 

   ```
   telnet 10.2.4.10 8060
   ```
   
1. Si aparece el siguiente error, póngase en contacto con el departamento de TI para obtener ayuda para solucionar este problema. Una vez que pueda comunicarse con Telnet, póngase en contacto con Soporte técnico de Microsoft si todavía tiene problemas con el estado del nodo del entorno de ejecución integrador.
        
   ![Error de la línea de comandos](media/self-hosted-integration-runtime-troubleshoot-guide/command-line-error.png)
        
1. Compruebe si el registro de error contiene lo siguiente:

    ```
    Error log: Cannot connect to worker manager: net.tcp://xxxxxx:8060/ExternalService.svc/ No DNS entries exist for host azranlcir01r1. No such host is known Exception detail: System.ServiceModel.EndpointNotFoundException: No DNS entries exist for host xxxxx. ---> System.Net.Sockets.SocketException: No such host is known at System.Net.Dns.GetAddrInfo(String name) at System.Net.Dns.InternalGetHostByName(String hostName, Boolean includeIPv6) at System.Net.Dns.GetHostEntry(String hostNameOrAddress) at System.ServiceModel.Channels.DnsCache.Resolve(Uri uri) --- End of inner exception stack trace --- Server stack trace: at System.ServiceModel.Channels.DnsCache.Resolve(Uri uri)
    ```
    
1. Para solucionar este problema, pruebe uno de los métodos siguientes o ambos:
    - Colocar todos los nodos en el mismo dominio
    - Agregar la asignación de IP a host en todos los archivos de hosts de la máquina virtual hospedada

### <a name="connectivity-issue-between-self-hosted-ir-and-data-factory-or-self-hosted-ir-and-data-sourcesink"></a>Problema de conectividad entre el IR autohospedado y Data Factory, o bien el IR autohospedado y el origen de datos o receptor

Para solucionar el problema de conectividad de red, debe saber cómo [recopilar el seguimiento de red](#how-to-collect-netmon-trace), entender cómo usarlo y [analizar el seguimiento de netmon](#how-to-analyze-netmon-trace) antes de aplicar las herramientas de Netmon en casos reales desde el IR autohospedado.

#### <a name="symptoms"></a>Síntomas

En ocasiones, cuando se solucionan problemas de conectividad como el siguiente entre el IR autohospedado y Data Factory: 

![Error de la solicitud HTTP](media/self-hosted-integration-runtime-troubleshoot-guide/http-request-error.png)

O bien entre el IR autohospedado y el origen de datos o el receptor, encontrará los errores siguientes:

`Copy failed with error:Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Cannot connect to SQL Server: ‘IP address’`

`One or more errors occurred. An error occurred while sending the request. The underlying connection was closed: An unexpected error occurred on a receive. Unable to read data from the transport connection: An existing connection was forcibly closed by the remote host. An existing connection was forcibly closed by the remote host Activity ID.`

#### <a name="resolution"></a>Resolución:

Al encontrar los problemas anteriores, consulte las instrucciones siguientes para solucionarlos:

Profundice en el análisis con el seguimiento de netmon.
- En primer lugar, puede establecer el filtro para ver cualquier restablecimiento del servidor en el lado cliente. En el ejemplo siguiente, puede ver que el lado servidor es el servidor de Data Factory.

    ![Servidor de Data Factory](media/self-hosted-integration-runtime-troubleshoot-guide/data-factory-server.png)

- Al obtener el paquete de restablecimiento, puede encontrar la conversación si sigue el TCP.

    ![Búsqueda de la conversación](media/self-hosted-integration-runtime-troubleshoot-guide/find-conversation.png)

- Después, puede obtener la conversación entre el cliente y el servidor de Data Factory si quita el filtro.

    ![Obtención de la conversación](media/self-hosted-integration-runtime-troubleshoot-guide/get-conversation.png)
- En función del seguimiento de netmon recopilado, se puede saber que el total de TTL (TimeToLive) es 64. Según los **valores predeterminados de TTL y límite de salto** mencionados en [este artículo](https://packetpushers.net/ip-time-to-live-and-hop-limit-basics/) (como se extrae a continuación), se aprecia que es el sistema Linux el que restablece el paquete y provoca la desconexión.

    Los valores predeterminados de TTL y límite de salto varían entre distintos sistemas operativos; estos son los de algunos de ellos:
    - Kernel de Linux 2.4 (2001 aproximadamente): 255 para TCP, UDP e ICMP
    - Kernel de Linux 4.10 (2015): 64 para TCP, UDP e ICMP
    - Windows XP (2001): 128 para TCP, UDP e ICMP
    - Windows 10 (2015): 128 para TCP, UDP e ICMP
    - Windows Server 2008: 128 para TCP, UDP e ICMP
    - Windows Server 2019 (2018): 128 para TCP, UDP e ICMP
    - macOS (2001): 64 para TCP, UDP e ICMP

    ![TTL 61](media/self-hosted-integration-runtime-troubleshoot-guide/ttl-61.png)
    
    Pero en el ejemplo anterior se muestra como 61 en lugar de 64, porque cuando el paquete de red alcanza el destino, debe pasar por varios saltos, como enrutadores o dispositivos de red. El número de enrutadores o dispositivos de red se deduce en el valor TTL final.
    En este caso, se ve que es posible que el restablecimiento se haya enviado desde el sistema Linux con TTL 64.

- Es necesario comprobar el cuarto salto desde IR autohospedado para confirmar de dónde puede provenir el dispositivo de restablecimiento.
 
    *Paquete de red del sistema Linux A con TTL 64 -> B TTL 64 menos 1 = 63 -> C TTL 63 menos 1 = 62 -> TTL 62 menos 1 = 61 IR autohospedado*

- En una situación idónea, el valor TTL será 128, lo que significa que el sistema Windows ejecuta la instancia de Data Factory. Como se muestra en el ejemplo siguiente, *128 – 107 = 21 saltos*, lo que significa que se han enviado 21 saltos para el paquete desde Data Factory a IR autohospedado durante el protocolo de enlace TCP 3.
 
    ![TTL 107](media/self-hosted-integration-runtime-troubleshoot-guide/ttl-107.png)

    Por tanto, debe comunicarle al equipo de red que compruebe cuál es el cuarto salto desde el IR autohospedado. Si es el firewall como sistema Linux, compruebe en los registros por qué el dispositivo restablece el paquete después del protocolo de enlace TCP 3. Pero si no está seguro de dónde investigar, intente obtener el seguimiento de netmon conjunto del IR autohospedado y el firewall durante el período problemático para determinar qué dispositivo ha podido restablecer este paquete y provocar la desconexión. En este caso, también debe indicarle al equipo de red que avance.

### <a name="how-to-collect-netmon-trace"></a>Procedimiento para recopilar el seguimiento de netmon

1.  Descargue las herramientas de Netmon desde [este sitio web](https://www.microsoft.com/en-sg/download/details.aspx?id=4865) e instálelas en el equipo del servidor (en cualquier servidor que tenga el problema) y en el cliente (por ejemplo el IR autohospedado).

2.  Cree una carpeta, por ejemplo, en la ruta de acceso siguiente: *D:\netmon*. Asegúrese de que tiene espacio suficiente para guardar el registro.

3.  Capture la información de puerto y dirección IP. 
    1. Inicie un símbolo del sistema CMD.
    2. Seleccione Ejecutar como administrador y ejecute el comando siguiente:
       
        ```
        Ipconfig /all >D:\netmon\IP.txt
        netstat -abno > D:\netmon\ServerNetstat.txt
        ```

4.  Capture el seguimiento de netmon (paquete de red).
    1. Inicie un símbolo del sistema CMD.
    2. Seleccione Ejecutar como administrador y ejecute el comando siguiente:
        
        ```
        cd C:\Program Files\Microsoft Network Monitor 3
        ```
    3. Puede usar tres comandos diferentes para capturar la página de red:
        - Opción A: comando de archivo RoundRobin (capturará un solo archivo y sobrescribirá los registros antiguos).

            ```
            nmcap /network * /capture /file D:\netmon\ServerConnection.cap:200M
            ```         
        - Opción B: comando de archivo encadenado (creará un archivo si se alcanzan los 200 MB).
        
            ```
            nmcap /network * /capture /file D:\netmon\ServerConnection.chn:200M
            ```          
        - Opción C: comando de archivo programado.

            ```
            nmcap /network * /capture /StartWhen /Time 10:30:00 AM 10/28/2011 /StopWhen /Time 11:30:00 AM 10/28/2011 /file D:\netmon\ServerConnection.chn:200M
            ```  

5.  Presione **Ctrl+C** para detener la captura del seguimiento de Netmon.
 
> [!NOTE]
> Si solo puede recopilar el seguimiento de netmon en el equipo cliente, obtenga la dirección IP del servidor para ayudarle a analizar el seguimiento.

### <a name="how-to-analyze-netmon-trace"></a>Procedimiento para analizar el seguimiento de netmon

Al intentar telnet **8.8.8.8 888** con el seguimiento de netmon recopilado anterior, debería ver el seguimiento siguiente:

![seguimiento de netmon 1](media/self-hosted-integration-runtime-troubleshoot-guide/netmon-trace-1.png)

![seguimiento de netmon 2](media/self-hosted-integration-runtime-troubleshoot-guide/netmon-trace-2.png)
 

Esto significa que no puede establecer una conexión TCP con el lado servidor **8.8.8.8** en función del puerto **888**, por lo que verá dos paquetes **SynReTransmit** adicionales. Como el origen **SELF-HOST2** no ha podido establecer la conexión con **8.8.8.8** en el primer paquete, seguirá intentándolo.

> [!TIP]
> - Puede hacer clic en **Cargar filtro** -> **Filtro estándar** -> **Direcciones** -> **Direcciones IPv4**.
> - Escriba **Dirección IPv4 = = 8.8.8.8** como filtro y haga clic en **Aplicar**. Después, solo verá la comunicación desde el equipo local al destino **8.8.8.8**.

![direcciones de filtro 1](media/self-hosted-integration-runtime-troubleshoot-guide/filter-addresses-1.png)
        
![direcciones de filtro 2](media/self-hosted-integration-runtime-troubleshoot-guide/filter-addresses-2.png)

En el ejemplo siguiente se muestra el aspecto que tendría un escenario correcto. 

- Si Telnet **8.8.8.8 53** funciona correctamente sin ningún problema, puede ver el protocolo de enlace TCP 3 y, después, la sesión finaliza con el protocolo de enlace TCP 4.

    ![Escenario de ejemplo correcto 1](media/self-hosted-integration-runtime-troubleshoot-guide/good-scenario-1.png)
     
    ![Escenario de ejemplo correcto 2](media/self-hosted-integration-runtime-troubleshoot-guide/good-scenario-2.png)

- En función del protocolo de enlace TCP 3 anterior, puede ver el flujo de trabajo siguiente:

    ![Flujo de trabajo del protocolo de enlace TCP 3](media/self-hosted-integration-runtime-troubleshoot-guide/tcp-3-handshake-workflow.png)
 
- El protocolo de enlace TCP 4 para finalizar la sesión y su flujo de trabajo se mostrarán de esta manera:

    ![Protocolo de enlace TCP 4](media/self-hosted-integration-runtime-troubleshoot-guide/tcp-4-handshake.png)

    ![Flujo de trabajo del protocolo de enlace TCP 4](media/self-hosted-integration-runtime-troubleshoot-guide/tcp-4-handshake-workflow.png) 


## <a name="self-hosted-ir-sharing"></a>Uso compartido del entorno de ejecución de integración autohospedado

### <a name="share-self-hosted-ir-from-a-different-tenant-is-not-supported"></a>No se admite el uso compartido de IR autohospedado desde un inquilino diferente 

#### <a name="symptoms"></a>Síntomas

Es posible que observe otras fábricas de datos (en diferentes inquilinos) al intentar compartir el IR autohospedado desde la interfaz de usuario de Azure Data Factory, pero no puede compartir el IR autohospedado entre factorías de datos que se encuentran en distintos inquilinos.

#### <a name="cause"></a>Causa

El IR autohospedado no puede compartirse entre inquilinos.


## <a name="next-steps"></a>Pasos siguientes

Para obtener más ayuda para solucionar problemas, pruebe los siguientes recursos:

*  [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Página de preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/topics/azure-data-factory.html)
*  [Foro de Stack Overflow para Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)
*  [Guía de rendimiento de flujos de datos de asignación](concepts-data-flow-performance.md)
