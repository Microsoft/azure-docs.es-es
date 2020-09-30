---
title: Solución de problemas de la implementación o detección de un dispositivo de Azure Migrate
description: Obtenga ayuda para implementar el dispositivo de Azure Migrate y detectar máquinas.
author: musa-57
ms.manager: abhemraj
ms.author: hamusa
ms.topic: troubleshooting
ms.date: 01/02/2020
ms.openlocfilehash: 9d4f30c5946e07b9f71c007d6f3f61efcae28637
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89296522"
---
# <a name="troubleshoot-the-azure-migrate-appliance-and-discovery"></a>Solución de problemas del dispositivo de Azure Migrate y la detección

Este artículo le ayuda a solucionar los problemas que surgen al implementar el dispositivo de [Azure Migrate](migrate-services-overview.md) y el uso de dicho dispositivo para detectar máquinas locales.


## <a name="whats-supported"></a>¿Qué se admite?

[Revise](migrate-appliance.md) los requisitos de compatibilidad del dispositivo.


## <a name="invalid-ovf-manifest-entry"></a>"Entrada de manifiesto de OVF no válida"

Si recibe el error "The provided manifest file is invalid: Invalid OVF manifest entry" (El archivo de manifiesto proporcionado no es válido: entrada de manifiesto de OVF no válida), haga lo siguiente:

1. Compruebe si el archivo OVA del dispositivo de Azure Migrate se descargó correctamente; para ello, compruebe el valor hash. [Más información](./tutorial-prepare-vmware.md). Si el valor hash no coincide, vuelva a descargar el archivo OVA e intente de nuevo la implementación.
2. Si la implementación sigue sin funcionar y utiliza el cliente de VMware vSphere para implementar el archivo OVF, intente implementarlo mediante el cliente web de vSphere. Si la implementación sigue sin funcionar, intente usar otro explorador web.
3. Si usa el cliente web de vSphere e intenta implementarlo en vCenter Server 6.5 o 6.7, pruebe a implementar el archivo OVA directamente en el host ESXi:
   - Conéctese directamente al host ESXi (en lugar de vCenter Server) con el cliente web (https://<*dirección IP del host*>/ui).
   - En **Home** > **Inventory** (Inicio > Inventario), seleccione **File** > **Deploy OVF template** (Archivo > Implementar plantilla OVF). Vaya al archivo OVA y complete la implementación.
4. Si la implementación sigue sin funcionar, póngase en contacto con el soporte técnico de Azure Migrate.

## <a name="cant-connect-to-the-internet"></a>No puedo conectarme a Internet

Esto puede ocurrir si la máquina del dispositivo está detrás de un proxy.

- Asegúrese de proporcionar las credenciales de autorización en caso de que el servidor proxy las necesite.
- Si usa un proxy de firewall basado en dirección URL para controlar la conectividad de salida, agregue [estas direcciones URL](migrate-appliance.md#url-access) a una lista de permitidos.
- Si usa un servidor proxy de interceptación para conectarse a Internet, importe el certificado del servidor proxy en la máquina virtual del dispositivo mediante [estos pasos](./migrate-appliance.md).

## <a name="cant-sign-into-azure-from-the-appliance-web-app"></a>No puede iniciar sesión en Azure desde la aplicación web del dispositivo

El error "Lo sentimos, pero tenemos problemas para iniciar su sesión" aparece si se usa una cuenta de Azure incorrecta para iniciar sesión en Azure. Este error se produce por dos posibles motivos:

- Si inicia sesión en la aplicación web del dispositivo para la nube pública con las credenciales de la cuenta de usuario para el portal de la nube de Government.
- Si inicia sesión en la aplicación web del dispositivo para la nube del gobierno con las credenciales de la cuenta de usuario para el portal de la nube privada.

Asegúrese de que usa las credenciales correctas.

##  <a name="datetime-synchronization-error"></a>Error de sincronización de fecha y hora

Un error sobre la sincronización de fecha y hora (802) indica que el reloj del servidor podría estar desincronizado con la hora actual en más de cinco minutos. Cambie la hora del reloj de la máquina virtual del recopilador para que coincida con la hora actual:

1. Abra un símbolo del sistema de administrador en la máquina virtual.
2. Para comprobar la zona horaria, ejecute **w32tm /tz**.
3. Para sincronizar la hora, ejecute **w32tm /resync**.


## <a name="unabletoconnecttoserver"></a>"UnableToConnectToServer"

Si obtiene este error de conexión, es posible que no pueda conectarse a vCenter Server *NombreDelServidor*.com:9443. Los detalles del error indican que no hay ningún punto de conexión escuchando en `https://\*servername*.com:9443/sdk` que pueda aceptar el mensaje.

- Compruebe si tiene en ejecución la versión más reciente del dispositivo. Si no es así, actualice el dispositivo a la [versión más reciente](./migrate-appliance.md).
- Si el problema persiste con la última versión, es posible que el dispositivo no pueda resolver el nombre de vCenter Server especificado o que el puerto especificado sea incorrecto. De forma predeterminada, si no se especifica el puerto, el recopilador intentará conectar con el número de puerto 443.

    1. Haga ping a *NombreDelServidor*.com desde el dispositivo.
    2. Si se produce un error en el paso 1, intente conectarse a vCenter Server mediante la dirección IP.
    3. Identifique el número de puerto correcto para conectarse a vCenter Server.
    4. Compruebe que vCenter Server está en funcionamiento.


## <a name="error-6005260039-appliance-might-not-be-registered"></a>Error 60052/60039: Es posible que el dispositivo no esté registrado

- Se produce el error 60052: "The appliance might not be registered successfully to the Azure Migrate project" (El dispositivo podría no estar registrado correctamente en el proyecto de Azure Migrate) si la cuenta de Azure usada para registrar el dispositivo no tiene permisos suficientes.
    - Asegúrese de que la cuenta de usuario de Azure que se ha usado para registrar el dispositivo tiene al menos el permiso de colaborador de la suscripción.
    - [Más información](./migrate-appliance.md#appliance---vmware) sobre los roles y permisos de Azure necesarios.
- Puede producirse el error 60039, "The appliance might not be registered successfully to the Azure Migrate project" (El dispositivo podría no estar registrado correctamente en el proyecto de Azure Migrate) si se produce un error de registro porque no se puede encontrar el proyecto de Azure Migrate utilizado para registrar el dispositivo.
    - En Azure Portal, compruebe si el proyecto existe en el grupo de recursos.
    - Si el proyecto no existe, cree un nuevo proyecto de Azure Migrate en el grupo de recursos y vuelva a registrar el dispositivo. [Obtenga información sobre](./how-to-add-tool-first-time.md#create-a-project-and-add-a-tool) cómo crear un nuevo proyecto.

## <a name="error-6003060031-key-vault-management-operation-failed"></a>Error 60030/60031: Error durante la operación de administración de Key Vault

Si recibe el error 60030 o 60031, "An Azure Key Vault management operation failed" (Se ha producido un error en la operación de administración de Azure Key Vault), haga lo siguiente:
- Asegúrese de que la cuenta de usuario de Azure que se ha usado para registrar el dispositivo tiene al menos el permiso de colaborador de la suscripción.
- Asegúrese de que la cuenta tiene acceso al almacén de claves especificado en el mensaje de error y vuelva a intentar la operación.
- Si el problema persiste, póngase en contacto con el servicio de soporte técnico de Microsoft.
- [Más información](./migrate-appliance.md#appliance---vmware) sobre los roles y permisos de Azure necesarios.

## <a name="error-60028-discovery-couldnt-be-initiated"></a>Error 60028: No se pudo iniciar la detección.

Error 60028: "Discovery couldn't be initiated because of an error. The operation failed for the specified list of hosts or clusters" (No se pudo iniciar la detección debido a un error. Se produjo un error en la operación para la lista especificada de hosts o clústeres) indica que no se pudo iniciar la detección en los hosts listados en el error debido a un problema de acceso o recuperación de la información de la máquina virtual. El resto de los hosts se agregaron correctamente.

- Vuelva a agregar los hosts enumerados en el error mediante la opción **Agregar host**.
- Si se produce un error de validación, revise las instrucciones de solución para corregir los errores e intente la opción **Guardar e iniciar la detección** de nuevo.

## <a name="error-60025-azure-ad-operation-failed"></a>El error 60025: Error en una operación de Azure AD 
El error 60025: "An Azure AD operation failed. The error occurred while creating or updating the Azure AD application" (Error en una operación de Azure AD. El error ha ocurrido al crear o actualizar la aplicación de Azure AD) aparece cuando la cuenta de usuario de Azure que se ha utilizado para iniciar la detección es diferente de la que se ha usado para registrar el dispositivo. Realice una de las siguientes acciones:

- Asegúrese de que la cuenta de usuario que inicia la detección es la misma que se ha usado para registrar el dispositivo.
- Proporcione permisos de acceso de la aplicación de Azure Active Directory a la otra cuenta de usuario para la que se produce el error en la operación de detección.
- Elimine el grupo de recursos creado previamente para el proyecto de Azure Migrate. Cree otro grupo de recursos para empezar de nuevo.
- [Aprenda más](./migrate-appliance.md#appliance---vmware) sobre los permisos de aplicación de Azure Active Directory.


## <a name="error-50004-cant-connect-to-host-or-cluster"></a>Se produce el error 50004: No se puede conectar al host o clúster

Se produce el error 50004: No se puede conectar a un host o un clúster porque no se puede resolver el nombre del servidor. Código de error WinRM: 0x803381B9" si el servicio Azure DNS del dispositivo no puede resolver el nombre de host o el clúster que ha proporcionado.

- Si ve este error en el clúster, añada el nombre de dominio completo en el clúster.
- También puede ver este error en los hosts de un clúster. Esto indica que el dispositivo puede conectarse al clúster, pero el clúster devuelve nombres de host que no son nombres de dominio completos. Para resolver este error, actualice el archivo de hosts del dispositivo; para ello, agregue una asignación de la dirección IP y los nombres de host:
    1. Abra el Bloc de notas como administrador.
    2. Abra el archivo C:\Windows\System32\Drivers\etc\hosts.
    3. Agregue la dirección IP y el nombre de host en una fila. Repita el procedimiento para cada host o clúster en el que observe este error.
    4. Guarde el archivo de hosts y ciérrelo.
    5. Compruebe si el dispositivo puede conectarse a los hosts mediante la aplicación de administración del dispositivo. Después de 30 minutos, debería ver la información más reciente sobre estos hosts en Azure Portal.


## <a name="error-60001-unable-to-connect-to-server"></a>Error 60001: No se puede establecer la conexión con el servidor 

- Asegúrese de que haya conectividad entre el dispositivo y el servidor.
- Si se trata de un servidor Linux, asegúrese de que la autenticación basada en contraseña está habilitada mediante los pasos siguientes:
    1. Inicie sesión en la máquina Linux y abra el archivo de configuración SSH con el comando "vi/etc/ssh/sshd_config".
    2. Establezca la opción "PasswordAuthentication" en Sí. Guarde el archivo.
    3. Reinicie el servicio SSH ejecutando "service sshd restart".
- Si es un servidor de Windows, asegúrese de que el puerto 5985 está abierto para permitir llamadas WMI remotas.
- Si detecta un servidor Linux GCP y usa un usuario raíz, utilice los siguientes comandos para cambiar la configuración predeterminada del inicio de sesión raíz.
    1. Inicie sesión en la máquina Linux y abra el archivo de configuración SSH con el comando "vi/etc/ssh/sshd_config".
    2. Establezca la opción "PermitRootLogin" en Sí.
    3. Reinicie el servicio SSH ejecutando "service sshd restart".

## <a name="error-no-suitable-authentication-method-found"></a>Error: No se encontró ningún método de autenticación adecuado

Asegúrese de que la autenticación basada en contraseña está habilitada en el servidor Linux mediante los pasos siguientes:
    1. Inicie sesión en la máquina Linux y abra el archivo de configuración SSH con el comando "vi/etc/ssh/sshd_config".
    2. Establezca la opción "PasswordAuthentication" en Sí. Guarde el archivo.
    3. Reinicie el servicio SSH ejecutando "service sshd restart".


## <a name="discovered-vms-not-in-portal"></a>Máquinas virtuales detectadas no en el portal

Si el estado de detección es "Detección en curso", pero aún no ve las máquinas virtuales en el portal, espere unos minutos:
- Una máquina virtual de VMware tarda aproximadamente 15 minutos.
- Cada host agregado tarda aproximadamente dos minutos en la detección de máquinas virtuales de Hyper-V.

Si espera y el estado no cambia, seleccione **Actualizar** en la pestaña **Servidores**. Se debería mostrar el recuento de los servidores detectados en Azure Migrate: Server Assessment y Azure Migrate: Server Migration.

Si esto no funciona y está detectando servidores VMware:

- Compruebe que la cuenta de vCenter especificada tiene los permisos establecidos correctamente y que tiene acceso al menos a una máquina virtual.
- Azure Migrate no puede detectar máquinas virtuales de VMware si a la cuenta de vCenter se le ha concedido acceso en el nivel de carpeta de máquina virtual de vCenter. [Obtenga más información](set-discovery-scope.md) sobre el ámbito de la detección.

## <a name="vm-data-not-in-portal"></a>Datos de VM que no están en el portal

Si las máquinas virtuales detectadas no aparecen en el portal o si los datos de estas están obsoletos, espere unos minutos. Los cambios realizados en los datos de configuración de las máquinas virtuales detectadas tardan 30 minutos, como máximo, en aparecer en el portal. Los cambios en los datos de las aplicaciones pueden tardar pocas horas en aparecer. Si no hay datos después de ese tiempo, intente actualizar como se indica a continuación

1. En **Servidores** > **Azure Migrate Server Assessment**, seleccione **Información general**.
2. En **Administrar**, seleccione **Agent Health**.
3. Seleccione **Actualizar agente**.
4. Espere a que se complete la operación de actualización. Ahora debería ver la información actualizada.

## <a name="deleted-vms-appear-in-portal"></a>Las máquinas virtuales eliminadas aparecen en el portal

Si elimina las máquinas virtuales y siguen apareciendo en el portal, espere treinta minutos. Si siguen apareciendo, actualice como se describió anteriormente.

## <a name="i-do-not-see-performance-data-for-some-network-adapters-on-my-physical-servers"></a>No veo los datos de rendimiento de algunos adaptadores de red en los servidores físicos.

Esto puede ocurrir si el servidor físico tiene habilitada la virtualización de Hyper-V. Debido a una brecha en el producto, el rendimiento de la red se captura en los adaptadores de red virtuales descubiertos.

## <a name="error-the-file-uploaded-is-not-in-the-expected-format"></a>Error: el archivo cargado no tiene el formato esperado
Algunas herramientas tienen una configuración regional que crea el archivo CSV con un punto y coma como delimitador. Cambie la configuración para asegurarse de que el delimitador sea una coma.

## <a name="i-imported-a-csv-but-i-see-discovery-is-in-progress"></a>He importado un archivo CSV pero veo "Detección en curso"
Este estado aparece si se produce un error en la carga de CSV debido a un error de validación. Intente importar de nuevo el archivo CSV. Puede descargar el informe de errores de la carga anterior y seguir las instrucciones de corrección del archivo para corregir los errores. El informe de errores se puede descargar en la sección "Detalles de importación" de la página "Descubrir máquinas".

## <a name="do-not-see-application-details-even-after-updating-guest-credentials"></a>No se ven los detalles de la aplicación incluso después de actualizar las credenciales de invitado
La detección de aplicaciones se ejecuta cada 24 horas. Si quiere ver los detalles de inmediato, actualice tal como se indica a continuación. Puede tardar unos minutos, en función del número de máquinas virtuales detectadas.

1. En **Servidores** > **Azure Migrate Server Assessment**, seleccione **Información general**.
2. En **Administrar**, seleccione **Agent Health**.
3. Seleccione **Actualizar agente**.
4. Espere a que se complete la operación de actualización. Ahora debería ver la información actualizada.

## <a name="unable-to-export-application-inventory"></a>No se puede exportar el inventario de aplicaciones
Asegúrese de que el usuario que descarga el inventario del portal tiene privilegios de colaborador en la suscripción.

## <a name="no-suitable-authentication-method-found-to-complete-authentication-publickey"></a>No se encontró ningún método de autenticación adecuado para completar la autenticación (publickey)
La autenticación basada en claves no funcionará, use la autenticación de contraseña.

## <a name="common-app-discovery-errors"></a>Errores comunes de detección de aplicaciones

Azure Migrate admite la detección de aplicaciones, roles y características mediante Azure Migrate: Server Assessment. Actualmente solo se admite la detección de aplicaciones para VMware. [Obtenga más información](how-to-discover-applications.md) sobre los requisitos y los pasos para configurar la detección de aplicaciones.

Los errores de detección de aplicaciones típicos se resumen en la tabla. 

**Error** | **Causa** | **Acción**
--- | --- | ---
9000: No se puede detectar el estado de la herramienta de VMware.     |   Es posible que las herramientas de VMware no estén instaladas o estén dañadas.    |   Asegúrese de que las herramientas de VMware estén instaladas y en ejecución en la máquina virtual.
9001: Herramientas de VMware no instaladas.     |   Es posible que las herramientas de VMware no estén instaladas o estén dañadas.    |   Asegúrese de que las herramientas de VMware estén instaladas y en ejecución en la máquina virtual.
9002: Las herramientas de VMware no se están ejecutando.   |   Es posible que las herramientas de VMware no estén instaladas o estén dañadas.    |   Asegúrese de que las herramientas de VMware estén instaladas y en ejecución en la máquina virtual.
9003: Tipo de sistema operativo no admitido para la detección de máquinas virtuales invitadas.    |   El sistema operativo que se ejecuta en el servidor no es Windows ni Linux.    |   Los tipos de sistema operativo admitidos son solo Windows y Linux. Si el servidor es Windows o Linux, compruebe el tipo de sistema operativo especificado en vCenter Server.
9004: No se está ejecutando la máquina virtual.     |   La máquina virtual está apagada.  |   Asegúrese de que la VM esté encendida.
9005: Tipo de sistema operativo no admitido para la detección de máquinas virtuales invitadas.    |   Tipo de sistema operativo no admitido para la detección de máquinas virtuales invitadas.     |   Los tipos de sistema operativo admitidos son solo Windows y Linux.
9006: La dirección URL para descargar el archivo de metadatos del invitado está vacía.     |   Esto puede ocurrir si el agente de detección no funciona según lo previsto.    |   El problema debería resolverse automáticamente en 24 horas. Si el problema persiste, póngase en contacto con el Soporte técnico de Microsoft.
9007: No se encuentra el proceso que ejecuta la tarea de detección en la máquina virtual invitada.   |   Esto puede ocurrir si el agente de detección no funciona correctamente.   |   El problema debería resolverse automáticamente en 24 horas. Si el problema persiste, póngase en contacto con el Soporte técnico de Microsoft.
9008: No se puede recuperar el estado del proceso de la máquina virtual invitada.   |   El problema puede producirse debido a un error interno.   |   El problema debería resolverse automáticamente en 24 horas. Si el problema persiste, póngase en contacto con el Soporte técnico de Microsoft.
9009: El UAC de Windows ha impedido la ejecución de la tarea de detección en el servidor.  |   La configuración del Control de cuentas de usuario (UAC) de Windows en el servidor es restrictiva e impide la detección de las aplicaciones instaladas.  |   En la configuración "Control de cuentas de usuario" del servidor configure la opción de UAC para que esté en uno de los dos niveles inferiores.
9010: La máquina virtual está apagada.     |   La máquina virtual está apagada.  |   Asegúrese de que la VM esté encendida.
9011: Archivo de metadatos detectado no encontrado en el sistema de archivos de la máquina virtual invitada.    |   El problema puede producirse debido a un error interno.   |   El problema debería resolverse automáticamente en 24 horas. Si el problema persiste, póngase en contacto con el Soporte técnico de Microsoft.
9012: El archivo de metadatos detectado está vacío.     |   El problema puede producirse debido a un error interno.   |   El problema debería resolverse automáticamente en 24 horas. Si el problema persiste, póngase en contacto con el Soporte técnico de Microsoft.
9013: Se crea un nuevo perfil temporal para cada inicio de sesión.    |   Se crea un nuevo perfil temporal para cada inicio de sesión en la máquina virtual de VMware.    |   Para resolverlo, póngase en contacto con el servicio de Soporte técnico de Microsoft.
9014: No se pueden recuperar los metadatos del sistema de archivos de la máquina virtual invitada.     |   No hay conectividad con el host ESXi.    |   Asegúrese de que el dispositivo puede conectarse al puerto 443 en el host ESXi que ejecuta la máquina virtual.
9015: El rol Operaciones de invitado no está habilitado en la cuenta de usuario de vCenter   |   El rol operaciones de invitado no está habilitado en la cuenta de usuario de vCenter.   |   Asegúrese de que el rol Operaciones de invitado está habilitado en la cuenta de usuario de vCenter.
9016: No se puede detectar porque el agente de operaciones de invitado no está actualizado.   |   Las herramientas de VMware no están instaladas correctamente o no están actualizadas.    |   Asegúrese de que las herramientas de VMware están correctamente instaladas y actualizadas.
9017: No se encuentra el archivo con los metadatos detectados en la máquina virtual.  |   El problema puede producirse debido a un error interno.   |   Para resolverlo, póngase en contacto con el servicio de Soporte técnico de Microsoft.
9018: PowerShell no está instalado en las máquinas virtuales invitadas.  |   PowerShell aún no está disponible en la máquina virtual invitada.    |   Instale PowerShell en la máquina virtual invitada.
9019: No se puede realizar la detección debido a errores de operación de la máquina virtual invitada.     |   No se pudo realizar la operación de invitado de VMware en la máquina virtual.    |   Asegúrese de que las credenciales de la máquina virtual sean válidas y que el nombre de usuario proporcionado en las credenciales de la máquina virtual invitada esté en formato UPN.
9020: Se ha denegado el permiso se creación de archivo.    |   El rol asociado al usuario o a la directiva de grupo restringe al usuario a la hora de crear el archivo de la carpeta.    |   Compruebe si el usuario invitado proporcionado tiene permiso de creación para el archivo de la carpeta. Consulte las **notificaciones** en Server Assessment para ver el nombre de la carpeta.
9021: No se puede crear el archivo en la ruta de acceso temporal del sistema.     |   La herramienta de VMware reporta la ruta de acceso temporal del sistema en lugar de la ruta temporal de los usuarios.    |   Actualice la versión de las herramientas de VMware a una versión superior a 10287 (formato de cliente NGC/VI).
9022: Se ha denegado el acceso al objeto WMI.    |   El rol asociado al usuario o a la directiva de grupo restringe el acceso del usuario al objeto WMI.  |   Póngase en contacto con el soporte técnico de Microsoft.
9023: No se puede ejecutar PowerShell ya que valor de la variable de entorno SystemRoot está vacío.    |   El valor de la variable de entorno SystemRoot está vacío para la máquina virtual invitada.     |   Para resolverlo, póngase en contacto con el servicio de Soporte técnico de Microsoft.
9024: No se puede descubrir ya que el valor de la variable de entorno TEMP está vacío.    |   El valor de la variable de entorno TEMP está vacío para la máquina virtual invitada.   |   Póngase en contacto con el soporte técnico de Microsoft.
9025: PowerShell está dañado en las máquinas virtuales invitadas.  |   PowerShell está dañado en las máquinas virtuales invitadas.    |   Vuelva a instalar PowerShell en la máquina virtual invitada y compruebe si PowerShell puede ejecutarse en ella.
9026: No se pueden ejecutar operaciones de invitado en la máquina virtual.  |   El estado de la máquina virtual no permite la ejecución de operaciones de invitado en la máquina virtual.   |   Para resolverlo, póngase en contacto con el servicio de Soporte técnico de Microsoft.
9027: El agente de operaciones de invitado no se ejecuta en la máquina virtual.   |   No se pudo establecer contacto con el agente de operaciones de invitado que se ejecuta en la máquina virtual.    |   Para resolverlo, póngase en contacto con el servicio de Soporte técnico de Microsoft.
9028: No se puede crear el archivo porque no hay suficiente almacenamiento en disco en la máquina virtual.     |   No hay suficiente espacio en el disco.   |   Asegúrese de que hay suficiente espacio disponible en el almacenamiento en disco de la máquina virtual.
9029: Las credenciales de la máquina virtual invitada no tienen acceso a PowerShell.   |   El acceso a PowerShell no está disponible para el usuario.     |   Asegúrese de que el usuario agregado en el dispositivo puede acceder a PowerShell en la máquina virtual invitada.
9030: No se pueden recopilar los metadatos detectados porque el host ESXi está desconectado.     |   El host ESXi está desconectado.   |   Asegúrese de que el host ESXi que ejecuta la máquina virtual está conectado.
9031: No se pueden recopilar los metadatos detectados porque el host ESXi no responde.   |   El host remoto tiene un estado no válido.    |   Asegúrese de que el host ESXi que ejecuta la máquina virtual se está ejecutando y está conectado.
9032: No se puede detectar debido a un error interno.   |   El problema puede producirse debido a un error interno.   |   Para resolverlo, póngase en contacto con el servicio de Soporte técnico de Microsoft.
9033: No se puede detectar porque el nombre de usuario de la máquina virtual contiene caracteres no válidos.     |   Se han detectado caracteres no válidos en el nombre de usuario.   |   Proporcione de nuevo la credencial de máquina virtual asegurándose de que no hay caracteres no válidos.
9034: El nombre de usuario proporcionado no está en formato UPN.    |   El nombre de usuario no está en formato UPN.  |   Asegúrese de que el nombre de usuario está en formato de nombre principal de usuario (UPN).
9035: No se puede detectar porque el modo de lenguaje de PowerShell no está establecido en 'Lenguaje completo'.  |   El modo de lenguaje de PowerShell en la máquina virtual invitada no se estableció en idioma completo.   |   Asegúrese de que el modo de lenguaje de PowerShell se estableció en 'Lenguaje completo'.
10000: El tipo de sistema operativo no es compatible.   |   El sistema operativo que se ejecuta en el servidor no es Windows ni Linux.    |   Los tipos de sistema operativo admitidos son solo Windows y Linux.
10001: No se encuentra el script de detección de servidores en el dispositivo.    |   La detección no funciona según lo esperado.   |   Para resolverlo, póngase en contacto con el servicio de Soporte técnico de Microsoft.
10002: La tarea de detección no se ha completado a tiempo.     |   El agente de detección no funciona según lo esperado.     |   El problema debería resolverse automáticamente en 24 horas. Si el problema persiste, póngase en contacto con el Soporte técnico de Microsoft.
10003: El proceso que ejecuta la tarea de detección salió con un error.    |   El proceso que ejecuta la tarea de detección salió con un error.  |   El problema debería resolverse automáticamente en 24 horas. Si el problema persiste, póngase en contacto con el soporte técnico de Microsoft.
10004: No se proporcionó ninguna credencial para el tipo de sistema operativo de invitado.  |   No se proporcionaron las credenciales para acceder a las máquinas de este tipo de sistema operativo en el dispositivo de Azure Migrate.    |   Agregar credenciales para equipos en el dispositivo.
10005: Las credenciales proporcionadas no son válidas.   |   Las credenciales proporcionadas para que el dispositivo tenga acceso al servidor son incorrectas.  |   Actualice las credenciales proporcionadas en el dispositivo y asegúrese de que permiten acceder al servidor.
10006: El almacén de credenciales no admite el tipo de sistema operativo invitado.  |   El sistema operativo que se ejecuta en el servidor no es Windows ni Linux.    |   Los tipos de sistema operativo admitidos son solo Windows y Linux.
10007: No se pueden procesar los metadatos detectados.    |   Error al intentar deserializar el código JSON.    |   Para resolverlo, póngase en contacto con el servicio de Soporte técnico de Microsoft.
10008: No se puede crear un archivo en el servidor.    |  El problema puede producirse debido a un error interno.    |   Para resolverlo, póngase en contacto con el servicio de Soporte técnico de Microsoft.
10009: No se pueden escribir metadatos detectados en un archivo del servidor.  |   El problema puede producirse debido a un error interno.   |   Para resolverlo, póngase en contacto con el servicio de Soporte técnico de Microsoft.




## <a name="next-steps"></a>Pasos siguientes
Configuración de un dispositivo para [VMware](how-to-set-up-appliance-vmware.md), [Hyper-V](how-to-set-up-appliance-hyper-v.md), o [servidores físicos](how-to-set-up-appliance-physical.md).
