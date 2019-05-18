---
title: Solución de problemas y errores de la replicación de Azure a Azure en Azure Site Recovery| Microsoft Docs
description: Solución de problemas y errores al replicar máquinas virtuales de Azure para la recuperación ante desastres
services: site-recovery
author: sujayt
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 04/08/2019
ms.author: sujayt
ms.openlocfilehash: 3c87e159022b6dcf13daf2a2659c88c0529a8f48
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796425"
---
# <a name="troubleshoot-azure-to-azure-vm-replication-issues"></a>Solución de problemas de replicación de máquinas virtuales de Azure a Azure

En este artículo se describen los problemas comunes de Azure Site Recovery al replicar y recuperar máquinas virtuales de Azure de una región a otra y se explica cómo solucionarlos. Para más información sobre las configuraciones admitidas, consulte la [matriz de compatibilidad para la replicación de máquinas virtuales de Azure](site-recovery-support-matrix-azure-to-azure.md).

## <a name="list-of-errors"></a>Lista de errores
- **[Problemas de cuota de recursos de Azure (código de error 150097)](#azure-resource-quota-issues-error-code-150097)**
- **[Certificados raíz de confianza (código de error 151066)](#trusted-root-certificates-error-code-151066)**
- **[Conectividad saliente para Site Recovery (código de error 151195)](#issue-1-failed-to-register-azure-virtual-machine-with-site-recovery-151195-br)**

## <a name="azure-resource-quota-issues-error-code-150097"></a>Problemas de cuota de recursos de Azure (código de error 150097)
La suscripción debe estar habilitada para crear máquinas virtuales de Azure en la región de destino que se va a utilizar como región de recuperación ante desastres. Además, la suscripción debe tener habilitada suficiente cuota para crear máquinas virtuales de un tamaño específico. De forma predeterminada, Site Recovery adopta el mismo tamaño en la máquina virtual de destino que en la de origen. Si el tamaño coincidente no está disponible, se selecciona automáticamente el tamaño más aproximado posible. Si no hay ningún tamaño coincidente que admita la configuración de la máquina virtual de origen, aparece este mensaje de error:

**Código de error** | **Causas posibles:** | **Recomendación**
--- | --- | ---
150097<br></br>**Mensaje**: No se pudo habilitar la replicación para la máquina virtual VmName. | -El identificador de suscripción no puede habilitarse para crear ninguna máquina virtual en la ubicación de la región de destino.</br></br>-El identificador de suscripción no puede habilitarse o no tiene cuota suficiente para crear tamaños de máquina virtual específicos en la ubicación de la región de destino.</br></br>-No se encuentra un tamaño adecuado de máquina virtual de destino que coincida con el número de tarjetas NIC de la máquina virtual de origen (2) para el id. de suscripción en la ubicación de la región de destino.| Para habilitar la creación de máquinas virtuales con el tamaño adecuado en la ubicación de destino de la suscripción, póngase en contacto con el [soporte de facturación de Azure](https://docs.microsoft.com/azure/azure-supportability/resource-manager-core-quotas-request). Una vez habilitada, vuelva a intentar la operación con error.

### <a name="fix-the-problem"></a>Corrección del problema
Para permitir que su suscripción cree máquinas virtuales de los tamaños necesarios en la ubicación de destino, puede ponerse en contacto con el [soporte de facturación de Azure](https://docs.microsoft.com/azure/azure-supportability/resource-manager-core-quotas-request).

Si la ubicación de destino tiene restricciones de capacidad, deshabilite la replicación y habilítela en una ubicación diferente donde su suscripción tenga cuota suficiente para crear máquinas virtuales con los tamaños necesarios.

## <a name="trusted-root-certificates-error-code-151066"></a>Certificados raíz de confianza (código de error 151066)

Si los certificados raíz de confianza más recientes no están presentes en la máquina virtual, el trabajo de habilitar la replicación podría dar error. Sin los certificados, la autenticación y la autorización de las llamadas al servicio de Site Recovery desde la máquina virtual darán error. Aparece el mensaje de error al habilitar la replicación para el trabajo de Site Recovery:

**Código de error** | **Causa posible** | **Recomendaciones**
--- | --- | ---
151066<br></br>**Mensaje**: Error de configuración de Site Recovery. | Los certificados raíz de confianza necesarios usados para la autorización y la autenticación no están presentes en la máquina. | -En máquinas virtuales que ejecutan el sistema operativo Windows, asegúrese de que los certificados raíz de confianza estén presentes en la máquina. Para más información, consulte [Configurar raíces de confianza y certificados no permitidos](https://technet.microsoft.com/library/dn265983.aspx).<br></br>-En máquinas virtuales que ejecutan el sistema operativo Linux, siga las instrucciones relativas a los certificados raíz de confianza publicados por el distribuidor de versiones de dicho sistema operativo.

### <a name="fix-the-problem"></a>Corrección del problema
**Windows**

Instale las actualizaciones de Windows más recientes en la máquina virtual para que todos los certificados raíz de confianza están presentes en la máquina. Si se encuentra en un entorno desconectado, siga el proceso de actualización estándar de Windows en su organización para obtener los certificados. Si los certificados necesarios no están presentes en la máquina virtual, las llamadas al servicio de Site Recovery darán error por razones de seguridad.

Para obtener los certificados raíz más recientes y la lista actualizada de revocación de certificados en las máquinas virtuales, siga el proceso habitual de administración de actualizaciones de certificados o de administración de actualizaciones de Windows.

Para comprobar que el problema se haya resuelto, vaya a login.microsoftonline.com en su máquina virtual mediante un explorador.

**Linux**

Para obtener los certificados raíz de confianza y la lista de revocación de certificados más recientes en la máquina virtual, siga las instrucciones proporcionadas por su distribuidor de Linux.

Como SuSE Linux usa vínculos simbólicos para mantener una lista de certificados, siga estos pasos:

1.  Inicie sesión con un usuario raíz.

2.  Ejecute este comando para cambiar el directorio.

      ``# cd /etc/ssl/certs``

1. Compruebe si está presente el certificado raíz de CA de Symantec.

      ``# ls VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem``

2. Si no se encuentra, ejecute el siguiente comando para descargar el archivo. Compruebe si hay errores y siga la acción recomendada para los errores de red.

      ``# wget https://www.symantec.com/content/dam/symantec/docs/other-resources/verisign-class-3-public-primary-certification-authority-g5-en.pem -O VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem``

3. Compruebe si está presente el certificado raíz de CA de Baltimore.

      ``# ls Baltimore_CyberTrust_Root.pem``

4. Si no se encuentra, descárguelo.  

    ``# wget https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem -O Baltimore_CyberTrust_Root.pem``

5. Compruebe si está presente el certificado DigiCert_Global_Root_CA.

    ``# ls DigiCert_Global_Root_CA.pem``

6. Si no se encuentra, ejecute los siguientes comandos para descargarlo.

    ``# wget http://www.digicert.com/CACerts/DigiCertGlobalRootCA.crt``

    ``# openssl x509 -in DigiCertGlobalRootCA.crt -inform der -outform pem -out DigiCert_Global_Root_CA.pem``

7. Ejecute el script rehash para actualizar los hashes del firmante del certificado para los certificados descargados recientemente.

    ``# c_rehash``

8.  Compruebe si los hashes del firmante, como los symlink, se crean para los certificados.

    - Get-Help

      ``# ls -l | grep Baltimore``

    - Salida

      ``lrwxrwxrwx 1 root root   29 Jan  8 09:48 3ad48a91.0 -> Baltimore_CyberTrust_Root.pem
      -rw-r--r-- 1 root root 1303 Jun  5  2014 Baltimore_CyberTrust_Root.pem``

    - Get-Help

      ``# ls -l | grep VeriSign_Class_3_Public_Primary_Certification_Authority_G5``

    - Salida

      ``-rw-r--r-- 1 root root 1774 Jun  5  2014 VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem
      lrwxrwxrwx 1 root root   62 Jan  8 09:48 facacbc6.0 -> VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem``

    - Get-Help

      ``# ls -l | grep DigiCert_Global_Root``

    - Salida

      ``lrwxrwxrwx 1 root root   27 Jan  8 09:48 399e7759.0 -> DigiCert_Global_Root_CA.pem
      -rw-r--r-- 1 root root 1380 Jun  5  2014 DigiCert_Global_Root_CA.pem``

9.  Creación de una copia del archivo VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem con el nombre de archivo b204d74a.0

    ``# cp VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem b204d74a.0``

10. Creación de una copia del archivo Baltimore_CyberTrust_Root.pem con el nombre de archivo 653b494a.0

    ``# cp Baltimore_CyberTrust_Root.pem 653b494a.0``

13. Creación de una copia del archivo DigiCert_Global_Root_CA.pem con el nombre de archivo 3513523f.0

    ``# cp DigiCert_Global_Root_CA.pem 3513523f.0``  


14. Compruebe si los archivos están presentes.  

    - Get-Help

      ``# ls -l 653b494a.0 b204d74a.0 3513523f.0``

    - Salida

      ``-rw-r--r-- 1 root root 1774 Jan  8 09:52 3513523f.0
      -rw-r--r-- 1 root root 1303 Jan  8 09:52 653b494a.0
      -rw-r--r-- 1 root root 1774 Jan  8 09:52 b204d74a.0``


## <a name="outbound-connectivity-for-site-recovery-urls-or-ip-ranges-error-code-151037-or-151072"></a>Conectividad saliente para direcciones URL o intervalos IP de Site Recovery (código de error 151037 o 151072)

Para que la replicación de Site Recovery funcione, la máquina virtual debe disponer de conectividad saliente a direcciones URL o intervalos IP específicos. Si la máquina virtual está detrás de un firewall o usa reglas de grupo de seguridad de red (NSG) para controlar la conectividad saliente, puede encontrarse alguno de estos problemas.

### <a name="issue-1-failed-to-register-azure-virtual-machine-with-site-recovery-151195-br"></a>Problema 1: no se pudo registrar la máquina virtual de Azure en Site Recovery (151195) </br>
- **Causa posible** </br>
  - No se puede establecer conexión con los puntos de conexión de Site Recovery por un error de resolución de DNS.
  - Normalmente, este problema se produce al volver a establecer la protección tras una conmutación por error de la máquina virtual y no se puede acceder al servidor DNS desde la región de recuperación ante desastres (DR).

- **Resolución**
   - Si utiliza un DNS personalizado, asegúrese de que el servidor DNS es accesible desde la región de recuperación ante desastres. Para comprobar si el DNS está personalizado, vaya a la máquina virtual > red de recuperación ante desastres > servidores DNS. Intente acceder al servidor DNS desde la máquina virtual. Si no puede, intente que esté accesible conmutando por error el servidor DNS o creando la línea del sitio entre la red de recuperación ante desastres y DNS.

    ![com-error](./media/azure-to-azure-troubleshoot-errors/custom_dns.png)


### <a name="issue-2-site-recovery-configuration-failed-151196"></a>Problema 2: error de configuración de Site Recovery (151196)
- **Causa posible** </br>
  - No se puede establecer conexión con los puntos de conexión de autenticación e identidad IP4 de Office 365.

- **Resolución**
  - Azure Site Recovery debe tener acceso a los intervalos de direcciones IP de Office 365 para la autenticación.
    Si utiliza un proxy que actúe como firewall o reglas de grupos de seguridad de red (NSG) de Azure para controlar la conectividad de salida de la red en la máquina virtual, no olvide permitir la comunicación con los intervalos de IP de O365. Cree una regla de grupos de seguridad de red basada en la [etiqueta de servicio de Azure Active Directory (AAD)](../virtual-network/security-overview.md#service-tags) para permitir el acceso a todas las direcciones IP correspondientes a AAD.
      - Si en el futuro se agregan nuevas direcciones a Azure Active Directory (AAD), tendrá que crear nuevas reglas de grupos de seguridad de red.

> [!NOTE]
> Si las máquinas virtuales se encuentran detrás **estándar** equilibrador de carga interno, a continuación, que no podrá tener acceso a las direcciones IP de Office 365, es decir, Login.micorsoftonline.com de forma predeterminada. Cambie a **básica** interno el tipo de equilibrador de carga o crear acceso salientes como se mencionó en el [artículo](https://aka.ms/lboutboundrulescli).

### <a name="issue-3-site-recovery-configuration-failed-151197"></a>Problema 3: error de configuración de Site Recovery (151197)
- **Causa posible** </br>
  - No se puede establecer conexión con los puntos de conexión del servicio Azure Site Recovery.

- **Resolución**
  - Azure Site Recovery necesita tener acceso a [intervalos de IP de Site Recovery](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-about-networking#outbound-connectivity-for-ip-address-ranges) en función de la región. Asegúrese de que los intervalos de IP están accesibles desde la máquina virtual.


### <a name="issue-4-a2a-replication-failed-when-the-network-traffic-goes-through-on-premises-proxy-server-151072"></a>Problema 4: Error en la replicación A2A cuando sale el tráfico de red a través del servidor de proxy de forma local (151072)
- **Causa posible** </br>
  - La configuración de proxy personalizada no es válida y el agente de Mobility Service de ASR no detectó automáticamente la configuración de proxy de Internet Explorer


- **Resolución**
  1. El agente de Mobility Service detecta la configuración de proxy de Internet Explorer en Windows y /etc/environment en Linux.
  2. Si prefiere configurar el proxy solo para Mobility Service de ASR, puede proporcionar los detalles de este en ProxyInfo.conf en:</br>
     - ``/usr/local/InMage/config/`` en ***Linux***
     - ``C:\ProgramData\Microsoft Azure Site Recovery\Config`` en ***Windows***
  3. ProxyInfo.conf debe tener la configuración de proxy en el siguiente formato INI.</br>
                *[proxy]*</br>
                *Address=http://1.2.3.4*</br>
                *Port=567*</br>
  4. El agente de Mobility Service de ASR solo admite ***servidores proxy no autenticados***.


### <a name="fix-the-problem"></a>Corrección del problema
Para incluir en la lista de permitidos [las direcciones URL necesarias](azure-to-azure-about-networking.md#outbound-connectivity-for-urls) o los [intervalos IP necesarios](azure-to-azure-about-networking.md#outbound-connectivity-for-ip-address-ranges), siga los pasos del [documento de instrucciones para redes](site-recovery-azure-to-azure-networking-guidance.md).

## <a name="disk-not-found-in-the-machine-error-code-150039"></a>Disco no encontrado en la máquina (código de error 150039)

Se debe inicializar un nuevo disco asociado a la máquina virtual.

**Código de error** | **Causas posibles:** | **Recomendaciones**
--- | --- | ---
150039<br></br>**Mensaje**: El disco de datos (DiskName) (DiskUri) con el número de unidad lógica (LUN) (LUNValue) no se ha asignado a un disco correspondiente que se notifica desde la máquina virtual como que tiene el mismo valor de LUN. | -Un nuevo disco de datos se asoció a la máquina virtual, pero no se ha inicializado.</br></br>-El disco de datos dentro de la máquina virtual no está notificando correctamente el valor LUN con el que el disco se asoció a la máquina virtual.| Asegúrese de que se inicializan los discos de datos y, a continuación, vuelva a intentar la operación.</br></br>Para Windows: [adjunte e inicialice un disco nuevo](https://docs.microsoft.com/azure/virtual-machines/windows/attach-managed-disk-portal).</br></br>Para Linux: [inicialice un nuevo disco de datos en Linux](https://docs.microsoft.com/azure/virtual-machines/linux/add-disk).

### <a name="fix-the-problem"></a>Corrección del problema
Asegúrese de que se hayan inicializado los discos de datos y, a continuación, vuelva a intentar la operación:

- Para Windows: [adjunte e inicialice un disco nuevo](https://docs.microsoft.com/azure/virtual-machines/windows/attach-managed-disk-portal).
- En Linux: [Agregue un nuevo disco de datos en Linux](https://docs.microsoft.com/azure/virtual-machines/linux/add-disk).

Si el problema persiste, póngase en contacto con el soporte técnico.

## <a name="one-or-more-disks-are-available-for-protectionerror-code-153039"></a>Uno o más discos que están disponibles para la protección (código de error 153039)
- **Causa posible** </br>
  - Si uno o varios de los discos agregados recientemente a la máquina virtual después de la protección. 
  - Si se han inicializado uno o varios discos más adelante después de la protección de la máquina virtual.

### <a name="fix-the-problem"></a>Corrección del problema
Puede elegir proteger los discos o ignorar la advertencia para que el estado de replicación de la máquina virtual correcto de nuevo.</br>
1. Para proteger los discos. Vaya a elementos replicados > máquina virtual > discos > haga clic en el disco no protegido > habilitar la replicación.
 ![add_disks](./media/azure-to-azure-troubleshoot-errors/add-disk.png)
2. Para descartar la advertencia. Vaya a elementos replicados > máquina virtual > haga clic en la alerta de descartar bajo la sección de información general.
![dismiss_warning](./media/azure-to-azure-troubleshoot-errors/dismiss-warning.png)
## <a name="unable-to-see-the-azure-vm-or-resource-group--for-selection-in-enable-replication"></a>No se puede ver la máquina virtual de Azure o grupo de recursos para la selección en "Habilitar replicación"

 **Causa 1:  el grupo de recursos y la máquina virtual de origen están en ubicaciones distintas** <br>
Azure Site Recovery actualmente mandatos que las máquinas virtuales y grupo de recursos de la región de origen deben estar en la misma ubicación. Si no es así, no podrá encontrar la máquina virtual durante el momento de la protección. Como alternativa, puede habilitar la replicación de la máquina virtual en lugar de almacén de Recovery services. Vaya a Sourece VM > Propiedades > habilitar la replicación y recuperación ante desastres.

**Causa 2: el grupo de recursos no forma parte de la suscripción seleccionada** <br>
Es posible que no pueda encontrar el grupo de recursos en el momento de la protección si no forma parte de la suscripción especificada. Asegúrese de que el grupo de recursos pertenece a la suscripción que se usa.

 **Causa 3: configuración obsoleta** <br>
Si no ve la VM que quiere habilitar para la replicación, podría deberse a que se dejó una configuración obsoleta de Site Recovery en la VM de Azure. Esta situación podría darse en los siguientes casos:

- Habilitó la replicación para la máquina virtual de Azure mediante Site Recovery y luego eliminó el almacén de Site Recovery sin deshabilitar explícitamente la replicación en la máquina virtual.
- Habilitó la replicación para la máquina virtual de Azure mediante Site Recovery y luego eliminó el grupo de recursos que contenía el almacén de Site Recovery sin deshabilitar explícitamente la replicación en la máquina virtual.

### <a name="fix-the-problem"></a>Corrección del problema

>[!NOTE]
>
>Asegúrese de actualizar el módulo "" AzureRM.Resources"" antes de usar el siguiente script.

Puede usar el artículo sobre cómo [quitar el script de configuración de ASR obsoleto](https://github.com/AsrOneSdk/published-scripts/blob/master/Cleanup-Stale-ASR-Config-Azure-VM.ps1) y quitar la configuración de Site Recovery obsoleta en la máquina virtual de Azure. Debería poder ver la VM después de quitar la configuración obsoleta.

## <a name="unable-to-select-virtual-machine-for-protection"></a>No se puede seleccionar la máquina virtual para la protección
 **Causa 1:  la máquina virtual tiene alguna extensión instalada en estado Con errores o No responde** <br>
 Vaya a Máquinas virtuales > Configuración > Extensiones y compruebe si hay alguna extensión en un estado con errores. Desinstale la extensión con errores y vuelva a intentar proteger la máquina virtual.<br>
 **Causa 2:  [el estado de aprovisionamiento de la máquina virtual no es válido](#vms-provisioning-state-is-not-valid-error-code-150019)**

## <a name="vms-provisioning-state-is-not-valid-error-code-150019"></a>El estado de aprovisionamiento de la máquina virtual no es válido (código de error 150019)

Para habilitar la replicación en la máquina virtual, el estado de aprovisionamiento debe ser **Correcto**. Puede comprobar el estado de la máquina virtual con los pasos siguientes.

1.  Seleccione el **Explorador de recursos** de **todos los servicios** en Azure Portal.
2.  Expanda la lista **Suscripciones** y seleccione su suscripción.
3.  Expanda la lista **ResourceGroups** y seleccione el grupo de recursos de la máquina virtual.
4.  Expanda la lista **Recursos** lista y seleccione la máquina virtual.
5.  Compruebe el campo **provisioningState** en la vista Instancia de lado derecho.

### <a name="fix-the-problem"></a>Corrección del problema

- Si **provisioningState** es **Error**, póngase en contacto con soporte técnico con detalles para solucionar el problema.
- Si **provisioningState** es **Actualizando**, puede que se esté implementando otra extensión. Compruebe si hay operaciones en curso en la máquina virtual, espere a que finalicen y reintente el trabajo **Habilitar replicación** para la recuperación del sitio que presenta el error.

## <a name="unable-to-select-target-virtual-network---network-selection-tab-is-grayed-out"></a>No se puede seleccionar la red virtual de destino: pestaña selección de red atenuada.

**Causa 1: si la máquina virtual está conectada a una red que ya está asignada a una "red de destino".**
- Si la máquina virtual de origen forma parte de una red virtual y otra máquina virtual de la misma red virtual ya está asignada a una red del grupo de recursos de destino, se deshabilitará la lista desplegable de selección de red de manera predeterminada.

![Network_Selection_greyed_out](./media/site-recovery-azure-to-azure-troubleshoot/unabletoselectnw.png)

**Causa 2: si ha protegido previamente la máquina virtual mediante Azure Site Recovery y deshabilitado la replicación.**
 - Deshabilitar la replicación de una máquina virtual no elimina la asignación de red. Debe eliminarse desde el almacén de Recovery Services donde está protegida. </br>
 Vaya al almacén de Recovery Services > Site Recovery Infrastructure (Infraestructura de Site Recovery) > Asignaciones de red. </br>
 ![Delete_NW_Mapping](./media/site-recovery-azure-to-azure-troubleshoot/delete_nw_mapping.png)
 - La red de destino configurada durante la instalación de la recuperación ante desastres puede cambiarse después, una vez protegida la máquina virtual. </br>
 ![Modify_NW_mapping](./media/site-recovery-azure-to-azure-troubleshoot/modify_nw_mapping.png)
 - Tenga en cuenta que cambiar la asignación de red afecta a todas las máquinas virtuales protegidas que usen esa asignación de red específica.


## <a name="comvolume-shadow-copy-service-error-error-code-151025"></a>Error del servicio de instantáneas de volumen/COM+ (código de error 151025)

**Código de error** | **Causas posibles:** | **Recomendaciones**
--- | --- | ---
151025<br></br>**Mensaje**: no se pudo instalar la extensión de Site Recovery | - Servicio de "aplicación del sistema COM+" deshabilitado.</br></br>- Servicio de instantáneas de volumen desactivado.| Establezca los servicios de aplicación del sistema COM+ y de instantáneas de volumen en modo de inicio automático o manual.

### <a name="fix-the-problem"></a>Corrección del problema

Puede abrir la consola de los servicios y asegurarse de que la aplicación del sistema COM+ y el servicio de instantáneas de volumen no se encuentran en "Desactivado" en "Tipo de inicio".
  ![com-error](./media/azure-to-azure-troubleshoot-errors/com-error.png)

## <a name="unsupported-managed-disk-size-error-code-150172"></a>Tamaño del disco administrado no compatible (código de error 150172)


**Código de error** | **Causas posibles:** | **Recomendaciones**
--- | --- | ---
150172<br></br>**Mensaje**: Protection couldn't be enabled for the virtual machine as it has (DiskName) with size (DiskSize) that is lesser than the minimum supported size 1024 GB [No se pudo habilitar la protección para la máquina virtual, ya que tiene un disco (DiskName) con un tamaño (DiskSize), que es menor que el tamaño mínimo admitido de 1024 MB]. | -El tamaño del disco es menor que el tamaño admitido de 1024 MB| Asegúrese de que los tamaños de disco están dentro del intervalo de tamaño admitido y reintente la operación.

## <a name="enable-protection-failed-as-device-name-mentioned-in-the-grub-configuration-instead-of-uuid-error-code-151126"></a>No se pudo habilitar la protección debido a que el nombre del dispositivo se menciona en la configuración de GRUB en lugar del UUID (código de error 151126)

**Causa posible:** </br>
Los archivos de configuración de GRUB ("/boot/grub/menu.lst", "/boot/grub/grub.cfg", "/boot/grub2/grub.cfg" o "/etc/default/grub") pueden contener el valor de los parámetros **root** y **resume** como nombres reales del dispositivo, en lugar del UUID. Site Recovery exige el enfoque de UUID, ya que el nombre de los dispositivos puede cambiar al reiniciar la VM, debido a que la VM puede no mostrar el mismo nombre tras la conmutación por error, lo que puede generar problemas. Por ejemplo: </br>


- La línea siguiente procede del archivo GRUB **/boot/grub2/grub.cfg**. <br>
  *linux   /boot/vmlinuz-3.12.49-11-default **root=/dev/sda2**  ${extra_cmdline} **resume=/dev/sda1** splash=silent quiet showopts*


- La siguiente línea es del archivo GRUB **/boot/grub/menu.lst**
  *kernel /boot/vmlinuz-3.0.101-63-default **root=/dev/sda2** **resume=/dev/sda1** splash=silent crashkernel=256M-:128M showopts vga=0x314*

Si observa la cadena en negrita anterior, GRUB tiene nombres de dispositivos reales para los parámetros "root" y "resume", en lugar del UUID.

**Solución:**<br>
Los nombres de dispositivo se deben reemplazar con el UUID correspondiente.<br>


1. Buscar el UUID del dispositivo, ejecute el comando "blkid \<nombre del dispositivo >". Por ejemplo:<br>
   ```
   blkid /dev/sda1
   ```<br>
   ```/dev/sda1: UUID="6f614b44-433b-431b-9ca1-4dd2f6f74f6b" TYPE="swap" ```<br>
   ```blkid /dev/sda2```<br>
   ```/dev/sda2: UUID="62927e85-f7ba-40bc-9993-cc1feeb191e4" TYPE="ext3"
   ```<br>



1. Now replace the device name with its UUID in the format like "root=UUID=\<UUID>". For example, if we replace the device names with UUID for root and resume parameter mentioned above in the files "/boot/grub2/grub.cfg", "/boot/grub2/grub.cfg" or "/etc/default/grub: then the lines in the files looks like. <br>
   *kernel /boot/vmlinuz-3.0.101-63-default **root=UUID=62927e85-f7ba-40bc-9993-cc1feeb191e4** **resume=UUID=6f614b44-433b-431b-9ca1-4dd2f6f74f6b** splash=silent crashkernel=256M-:128M showopts vga=0x314*
1. Restart the protection again

## Enable protection failed as device mentioned in the GRUB configuration doesn't exist(error code 151124)
**Possible Cause:** </br>
The GRUB configuration files ("/boot/grub/menu.lst", "/boot/grub/grub.cfg", "/boot/grub2/grub.cfg" or "/etc/default/grub") may contain the parameters "rd.lvm.lv" or "rd_LVM_LV" to indicate the LVM device that should be discovered at the time of booting. If these LVM devices doesn't exist, then the protected system itself will not boot and stuck in the boot process. Even the same will be observed with the failover VM. Below are few examples:

Few examples: </br>

1. The following line is from the GRUB file **"/boot/grub2/grub.cfg"** on RHEL7. </br>
   *linux16 /vmlinuz-3.10.0-957.el7.x86_64 root=/dev/mapper/rhel_mup--rhel7u6-root ro crashkernel=128M\@64M **rd.lvm.lv=rootvg/root rd.lvm.lv=rootvg/swap** rhgb quiet LANG=en_US.UTF-8*</br>
   Here the highlighted portion shows that the GRUB has to detect two LVM devices with names **"root"** and **"swap"** from the volume group "rootvg".
1. The following line is from the GRUB file **"/etc/default/grub"** on RHEL7 </br>
   *GRUB_CMDLINE_LINUX="crashkernel=auto **rd.lvm.lv=rootvg/root rd.lvm.lv=rootvg/swap** rhgb quiet"*</br>
   Here the highlighted portion shows that the GRUB has to detect two LVM devices with names **"root"** and **"swap"** from the volume group "rootvg".
1. The following line is from the GRUB file **"/boot/grub/menu.lst"** on RHEL6 </br>
   *kernel /vmlinuz-2.6.32-754.el6.x86_64 ro root=UUID=36dd8b45-e90d-40d6-81ac-ad0d0725d69e rd_NO_LUKS LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_LVM_LV=rootvg/lv_root  KEYBOARDTYPE=pc KEYTABLE=us rd_LVM_LV=rootvg/lv_swap rd_NO_DM rhgb quiet* </br>
   Here the highlighted portion shows that the GRUB has to detect two LVM devices with names **"root"** and **"swap"** from the volume group "rootvg".<br>

**How to Fix:**<br>

If the LVM device doesn't exist, fix either by creating it or remove the parameter for the same from the GRUB configuration files and then retry the enable protection. </br>

## Site recovery mobility service update completed with warnings ( error code 151083)
Site Recovery mobility service has many components, one of which is called filter driver. Filter driver gets loaded into system memory only at a time of system reboot. Whenever there are  site recovery mobility service updates that has filter driver changes, we update the machine but still gives you warning that some fixes require a reboot. It means that the filter driver fixes can only be realized when a new filter driver is loaded which can happen only at the time of system reboot.<br>
**Please note** that this is just a warning and existing replication keeps on working even after the new agent update. You can choose to reboot anytime you want to get the benefits of new filter driver but if you don't reboot than also old filter driver keeps on working. Apart from filter driver, **benefits of  any other enhancements and fixes in mobility service get realized without any reboot when the agent gets updated.**  


## Protection couldn't be enabled as replica managed disk 'diskname-replica' already exists without expected tags in the target resource group( error code 150161

**Cause**: It can occur if the  virtual machine was protected earlier in the past and during disabling the replication, replica disk was not cleaned due to some reason.</br>
**How to fix:**
Delete the mentioned replica disk in the error message and restart the failed protection job again.

## Next steps
[Replicate Azure virtual machines](site-recovery-replicate-azure-to-azure.md)
