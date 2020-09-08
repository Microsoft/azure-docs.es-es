---
title: Configurar un dispositivo de Azure Migrate para Hyper-V
description: Obtenga información sobre cómo configurar un dispositivo de Azure Migrate para evaluar y migrar VM de Hyper-V.
ms.topic: article
ms.date: 03/23/2020
ms.openlocfilehash: 21d88c4a2b2095fe677fe479bd7320f7a494db9e
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88929957"
---
# <a name="set-up-an-appliance-for-hyper-v-vms"></a>Configuración de un dispositivo para VM de Hyper-V

Siga este artículo para configurar el dispositivo de Azure Migrate para la evaluación de VM de Hyper-V con la herramienta [Azure Migrate:Server Assessment](migrate-services-overview.md#azure-migrate-server-assessment-tool).

El [dispositivo de Azure Migrate](migrate-appliance.md) es un dispositivo ligero que usa Azure Migrate: Server Assessment/Migration para detectar máquinas virtuales de Hyper-V locales y enviar datos de rendimiento y metadatos de máquinas virtuales a Azure.

Puede implementar el dispositivo mediante un par de métodos:

- Configúrelo en una VM de Hyper-V mediante un disco duro virtual descargado. Este es el método que se describe en este artículo.
- Configúrelo en una VM de Hyper-V o en una máquina física con un script del instalador de PowerShell. Debe usarse [este método](deploy-appliance-script.md) si no se puede configurar una VM mediante un disco duro virtual, o si se encuentra en Azure Government.

Una vez creada la aplicación, compruebe que se puede conectar a Azure Migrate:Server Assessment, configúrela por primera vez y regístrela en el proyecto de Azure Migrate.

## <a name="appliance-deployment-vhd"></a>Implementación del dispositivo (VHD)

Para configurar el dispositivo mediante una plantilla de VHD, haga lo siguiente:

- Proporcione un nombre de dispositivo y genere una clave del proyecto de Azure Migrate en el portal.
- Descargue un disco duro virtual de Hyper-V comprimido desde Azure Portal.
- Crear el dispositivo y comprobar que se puede conectar a Azure Migrate Server Assessment.
- Configure el dispositivo por primera vez y regístrelo en el proyecto de Azure Migrate mediante la clave del proyecto de Azure Migrate.

### <a name="generate-the-azure-migrate-project-key"></a>Generación de la clave del proyecto de Azure Migrate

1. En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Assessment**, seleccione **Detectar**.
2. En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , seleccione **Sí, con Hyper-V**.
3. En **1: Generar la clave del proyecto de Azure Migrate**, proporcione un nombre para el dispositivo de Azure Migrate que configurará para la detección de máquinas virtuales de Hyper-V. El nombre debe ser alfanumérico con 14 caracteres como máximo.
1. Haga clic en **Generar clave** para iniciar la creación de los recursos de Azure necesarios. No cierre la página Detectar máquinas durante la creación de recursos.
1. Después de la creación correcta de los recursos de Azure, se genera una **clave del proyecto Azure Migrate**.
1. Copie la clave, ya que la necesitará para completar el registro del dispositivo durante su configuración.

### <a name="download-the-vhd"></a>Descarga del disco duro virtual

En **2: Descargar dispositivo de Azure Migrate**, seleccione el archivo .VHD y haga clic en **Descargar**. 

   ![Selecciones para Detectar máquinas](./media/tutorial-assess-hyper-v/servers-discover.png)


   ![Selecciones para Generar clave](./media/tutorial-assess-hyper-v/generate-key-hyperv.png)


### <a name="verify-security"></a>Comprobación de la seguridad

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En la máquina en la que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash del disco duro virtual.
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo de uso: ```C:\>CertUtil -HashFile C:\AzureMigrate\AzureMigrate.vhd SHA256```
3.  El código hash generado debe coincidir con esta [configuración](./tutorial-assess-hyper-v.md#verify-security) para la versión 2.19.11.12 del dispositivo.




## <a name="create-the-appliance-vm"></a>Creación de la máquina virtual del dispositivo

Importe el archivo descargado y cree la VM.

1. Extraiga el archivo del disco duro virtual comprimido en la carpeta del host de Hyper-V que hospedará la VM del dispositivo. Se extraen tres carpetas.
2. Abra el administrador de Hyper-V. En **Acciones**, haga clic en **Importar máquina virtual**.

    ![Implementar disco duro virtual](./media/how-to-set-up-appliance-hyper-v/deploy-vhd.png)

2. En el Asistente para importar máquinas virtuales > **Antes de comenzar**, haga clic en **Siguiente**.
3. En **Buscar carpeta**, especifique la carpeta que contiene el disco duro virtual extraído. A continuación, haga clic en **Siguiente**.
1. En **Seleccionar máquina virtual**, haga clic en **Siguiente**.
2. En **Elegir tipo de importación**, haga clic en **Copiar la máquina virtual (crear un identificador único nuevo)** . A continuación, haga clic en **Siguiente**.
3. En **Elegir destino**, deje la configuración predeterminada. Haga clic en **Next**.
4. En **Carpetas de almacenamiento**, deje la configuración predeterminada. Haga clic en **Next**.
5. En **Elegir red**, especifique el conmutador virtual que usará la VM. El conmutador necesita conectividad a Internet para enviar datos a Azure.
6. En **Resumen**, revise los valores de configuración. Haga clic en **Finalizar**.
7. En Administrador de Hyper-V > **Máquinas virtuales**, inicie la VM.


### <a name="verify-appliance-access-to-azure"></a>Comprobación de que el dispositivo puede acceder a Azure

Asegúrese de que la VM del dispositivo pueda conectarse a las direcciones URL de Azure para las nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls).

### <a name="configure-the-appliance"></a>Configuración del dispositivo

Configure el dispositivo por primera vez.

> [!NOTE]
> Si configura la aplicación mediante un [script de PowerShell](deploy-appliance-script.md), en lugar del disco duro virtual descargado, los dos primeros pasos de este procedimiento no son pertinentes.

1. En Administrador de Hyper-V > **Máquinas virtuales**, haga clic con el botón derecho en la VM > **Conectar**.
2. Especifique el idioma, la zona horaria y la contraseña del dispositivo.
3. Abra un explorador en cualquier equipo que pueda conectarse a la máquina virtual y abra la dirección URL de la aplicación web del dispositivo: **https://*nombre o dirección IP del dispositivo*: 44368**.

   Como alternativa, puede abrir la aplicación desde el escritorio del dispositivo, para lo que debe hacer clic en el acceso directo de la aplicación.
1. Acepte los **términos de licencia** y lea la información de terceros.
1. En la aplicación web > **Set up prerequisites** (Configurar los requisitos previos ), realice las siguientes operaciones:
    - **Connectivity** (Conectividad): la aplicación comprueba que la máquina virtual tiene acceso a Internet. Si la máquina virtual usa un proxy:
      - Haga clic en **Configurar el proxy** y especifique la dirección del proxy (con los formatos http://ProxyIPAddress o http://ProxyFQDN) ) y el puerto de escucha.
      - Especifique las credenciales si el proxy requiere autenticación.
      - Solo se admite un proxy HTTP.
      - Si ha agregado detalles del proxy o ha deshabilitado el proxy o la autenticación, haga clic en **Guardar** para desencadenar la comprobación de conectividad.
    - **Time sync** (Sincronización de hora): Se comprueba la hora. Para que la detección funcione correctamente, la hora del dispositivo debe estar sincronizada con la hora de Internet.
    - **Instalación de actualizaciones**: Azure Migrate Server Assessment comprueba que el dispositivo tiene instaladas las actualizaciones más recientes. Una vez finalizada la comprobación, puede hacer clic en **Ver servicios del dispositivo** para ver el estado y las versiones de los componentes que se ejecutan en el dispositivo.

### <a name="register-the-appliance-with-azure-migrate"></a>Registro del dispositivo en Azure Migrate

1. Pegue la **clave del proyecto de Azure Migrate** copiada desde el portal. Si no tiene la clave, vaya a **Server Assessment > Detectar > Administrar los dispositivos existentes**, seleccione el nombre del dispositivo que proporcionó en el momento de la generación de la clave y copie la clave correspondiente.
1. Haga clic en **Iniciar sesión**. Se abrirá un mensaje de inicio de sesión de Azure en una nueva pestaña del explorador. Si no aparece, asegúrese de que ha deshabilitado el bloqueador de elementos emergentes en el explorador.
1. En la nueva pestaña, inicie sesión con su nombre de usuario y contraseña de Azure.
   
   No se admite el inicio de sesión con un PIN.
3. Después de iniciar sesión correctamente, vuelva a la aplicación web. 
4. Si la cuenta de usuario de Azure que se usa para el registro tiene los [permisos](tutorial-prepare-hyper-v.md#prepare-azure) adecuados en los recursos de Azure creados durante la generación de la clave, se iniciará el registro del dispositivo.
1. Una vez que el dispositivo se ha registrado correctamente, puede ver los detalles de registro haciendo clic en **Ver detalles**.



### <a name="delegate-credentials-for-smb-vhds"></a>Delegación de credenciales para discos duros virtuales de SMB

Si va a ejecutar discos duros virtuales en SMB, debe habilitar la delegación de credenciales desde el dispositivo a los hosts de Hyper-V. Para ello, desde el dispositivo:

1. En la VM del dispositivo, ejecute este comando. HyperVHost1 y HyperVHost2 son nombres de host de ejemplo.

    ```
    Enable-WSManCredSSP -Role Client -DelegateComputer HyperVHost1.contoso.com HyperVHost2.contoso.com -Force
    ```

2. También puede hacerlo en el Editor de directivas de grupo local en el dispositivo:
    - En **Directiva de equipo local** > **Configuración del equipo**, haga clic en **Plantillas administrativas** > **Sistema** > **Delegación de credenciales**.
    - Haga doble clic en **Permitir delegación de credenciales nuevas** y seleccione **Habilitado**.
    - En **Opciones**, haga clic en **Mostrar** y agregue cada host de Hyper-V que desee detectar en la lista, con **wsman/** como prefijo.
    - En **Delegación de credenciales**, haga doble clic en **Permitir la delegación de credenciales nuevas con autenticación solo NTLM de servidor**. De nuevo, agregue cada host de Hyper-V que quiera detectar en la lista, con **wsman/** como prefijo.

## <a name="start-continuous-discovery"></a>Inicio de detección continua

Conéctese desde el dispositivo a los hosts o clústeres de Hyper-V e inicie la detección de VM.

1. En **Paso 1: Proporcionar las credenciales del host de Hyper-V**, haga clic en **Agregar credenciales** para especificar un nombre descriptivo para las credenciales, agregue un **nombre de usuario** y una **contraseña** para el host o el clúster de Hyper-V que utilizará el dispositivo para detectar máquinas virtuales. Haga clic en **Guardar**.
1. Si quiere agregar varias credenciales a la vez, haga clic en **Agregar más** para guardar y agregar más credenciales. Se admiten varias credenciales para la detección de máquinas virtuales de Hyper-V.
1. En el **Paso 2: Proporcionar los detalles del host o el clúster de Hyper-V**, haga clic en **Agregar origen de detección** para especificar la **dirección IP o el FQDN** del host o el clúster de Hyper-V y el nombre descriptivo de las credenciales para conectarse al host o al clúster.
1. Puede **Agregar un solo elemento** cada vez o **Agregar varios elementos** de una sola vez. También hay una opción para proporcionar los detalles del host o el clúster de Hyper-V a través de **Importar CSV**.

    ![Selecciones para Agregar origen de detección](./media/tutorial-assess-hyper-v/add-discovery-source-hyperv.png)

    - Si elige **Agregar un solo elemento**, debe especificar el nombre descriptivo de las credenciales y la **dirección IP o el FQDN** del host o el clúster de Hyper-V y hacer clic en **Guardar**.
    - Si elige **Agregar varios elementos** _(opción seleccionada de manera predeterminada)_ , puede agregar varios registros a la vez mediante la especificación de la **dirección IP o el FQDN** del host o el clúster de Hyper-V con el nombre descriptivo de las credenciales en el cuadro de texto. **Compruebe** los registros agregados y haga clic en **Guardar**.
    - Si elige **Importar CSV**, puede descargar un archivo de plantilla CSV, rellenar el archivo con la **dirección IP o el FQDN** del host o el clúster de Hyper-V y el nombre descriptivo de las credenciales. A continuación, importe el archivo en el dispositivo, **compruebe** los registros del archivo y haga clic en **Guardar**.

1. Al hacer clic en Guardar, el dispositivo intentará validar la conexión a los hosts o clústeres agregados de Hyper-V y mostrar el **estado de validación** en la tabla en cada host o clúster.
    - En el caso de los hosts o clústeres validados correctamente, puede ver más detalles si hace clic en su dirección IP o FQDN.
    - Si se produce un error de validación para un host, haga clic en **Error en la validación** en la columna Estado de la tabla para revisar el error. Corrija el problema y vuelva a validar.
    - Para quitar hosts o clústeres, seleccione **Eliminar**.
    - No se puede quitar un host específico de un clúster. Solo puede quitar todo el clúster.
    - Puede agregar un clúster, incluso si hay problemas con hosts específicos del clúster.
1. Puede **volver a validar** la conectividad a los hosts o clústeres en cualquier momento antes de iniciar la detección.
1. Haga clic en **Iniciar detección** para dar comienzo a la detección de máquinas virtuales de los hosts o clústeres validados correctamente. Una vez que la detección se ha iniciado correctamente, puede comprobar el estado de detección en cada host o clúster de la tabla.

De esta forma comienza la detección. Los metadatos de los servidores detectados tardan alrededor de 2 minutos por host en aparecer en Azure Portal.

## <a name="verify-vms-in-the-portal"></a>Comprobación de VM en el portal

Una vez finalizada la detección, puede verificar que las VM aparezcan en el portal.

1. Abra el panel de Azure Migrate.
2. En la página **Azure Migrate - Servidores** > **Azure Migrate: Server Assessment**, haga clic en el icono que muestra el número de **servidores detectados**.


## <a name="next-steps"></a>Pasos siguientes

Pruebe la [evaluación de Hyper-V](tutorial-assess-hyper-v.md) con Azure Migrate Server Assessment.
