---
title: Guía de implementación de FortiGate | Microsoft Docs
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y FortiGate SSL VPN.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 18a3d9d5-d81c-478c-be7e-ef38b574cb88
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 08/11/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 999e19ffad1d18e163881c844cbf30f8b7fef574
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88657165"
---
# <a name="fortigate-deployment-guide"></a>Guía de implementación de FortiGate

## <a name="contents"></a>Contenido

- Canje de la licencia de FortiGate
- Descarga del firmware
- Implementación de la máquina virtual de FortiGate
   - Establecimiento de una dirección IP pública estática y asignación de un nombre de dominio completo
   - Creación de una regla del grupo de seguridad de red de entrada para el puerto TCP
- Creación de una aplicación de Azure personalizada para FortiGate
- Preparación para la coincidencia de grupos
   - Creación de grupos para los usuarios
- Configuración de la máquina virtual de FortiGate
   - Instalación de la licencia
   - Actualización del firmware
   - Cambio del puerto de administración a TCP
   - Carga del certificado de firma de SAML para Azure Active Directory
   - Carga y configuración de un certificado SSL personalizado
   - Configuración de la línea de comandos
   - Creación de portales de VPN y de la directiva de firewall
- Prueba del inicio de sesión con Azure

## <a name="redeeming-the-fortigate-license"></a>Canje de la licencia de FortiGate

El producto Fortinet FortiGate Next-Generation Firewall está disponible como una máquina virtual en IaaS de Azure. Existen dos modos de licencia para esta máquina virtual:

- Pay-as-you-go (PAYG)
- Traiga su propia licencia (BYOL)

A la vez que se asocia con Fortinet para proporcionar una guía de acceso híbrido seguro (SHA), Fortinet puede proporcionar licencias a los miembros del equipo de acceso híbrido seguro de puesta en producción de Azure AD. En los casos en los que no se proporcione ninguna licencia, la implementación del modo Pago por uso también funcionará.

En los casos en los que se haya emitido una licencia, Fortinet proporciona un código de registro que se debe canjear en línea.

![FortiGate SSL VPN](registration-code.png)

1. Regístrese en https://support.fortinet.com/.
2. Después del registro, inicie sesión en https://support.fortinet.com/.
3. Vaya a **Asset** (Recurso) -> **Register/Activate** (Registrar/Activar).
4. Escriba el código de registro proporcionado por Fortinet.
5. Especifique el código de registro, seleccione **The product will be used by a non-government user** (El producto lo utilizará un usuario no gubernamental) y haga clic en **Next** (Siguiente).
6. Escriba una descripción del producto (por ejemplo, FortiGate), establezca el asociado de Fortinet como **Other** (Otro) - > **Microsoft** y haga clic en **Next** (Siguiente).
7. Acepte el **contrato de registro del producto Fortinet** y haga clic en **Next** (Siguiente).
8. Acepte los **términos** y haga clic en **Confirm** (Confirmar).
9. Haga clic en **License File Download** (Descargar archivo de licencia) y guarde la licencia para más adelante.


## <a name="download-firmware"></a>Descarga del firmware

En el momento de escribir este artículo, la máquina virtual de Azure de Fortinet FortiGate no se incluye con la versión de firmware necesaria para la autenticación de SAML. La versión más reciente debe obtenerse de Fortinet.

1. Inicie sesión en https://support.fortinet.com/
2. Vaya a **Download** (Descargar) - > **Firmware Images** (Imágenes de firmware).
3. Haga clic en **Download** (Descargar) a la derecha de **Release Notes** (Notas de la versión).
4. Haga clic en **v6.**
5. Haga clic en **6.**
6. Haga clic en **6.4.**
7. Para descargar **FGT_VM64_AZURE-v6-build1637-FORTINET.out** haga clic en el vínculo **HTTPS** en la misma fila.
8. Guarde el archivo para más adelante.


## <a name="deploy-the-fortigate-vm"></a>Implementación de la máquina virtual de FortiGate

1. Vaya a https://portal.azure.com e inicie sesión en la suscripción en la que desea implementar la máquina virtual de FortiGate.
2. Cree un nuevo grupo de recursos o abra el grupo de recursos en el que desea implementar la máquina virtual de FortiGate.
3. Haga clic en **Agregar**.
4. Escriba "Forti" en el cuadro de diálogo **Buscar en el Marketplace** y seleccione **Fortinet FortiGate Next-** **Generation Firewall**.
5. Seleccione el plan de software (BYOL si tiene una licencia o PAYG en caso contrario) y haga clic en **Crear**.
6. Rellene la configuración de la máquina virtual.

    ![FortiGate SSL VPN](virtual-machine.png)

7. Establezca el tipo de autenticación en **Contraseña** y proporcione credenciales administrativas para la máquina virtual.
8. Haga clic en **Revisar y crear**.
9. Haga clic en **Crear**
10. Espere a que la implementación de la máquina virtual se complete.


### <a name="set-a-statuc-public-ip-address-and-assign-a-fully-qualified-domain-name"></a>Establecimiento de una dirección IP pública estática y asignación de un nombre de dominio completo

Para una experiencia de usuario coherente, es conveniente establecer la dirección IP pública asignada a la máquina virtual de FortiGate para que se asigne de forma estática. Además, la asignación a un nombre de dominio completo también es útil por las mismas razones.

_Establecimiento de una dirección IP pública estática_

1. Vaya a https://portal.azure.com y abra la configuración de la máquina virtual de FortiGate
2. En la pantalla **Información general**, haga clic en la dirección IP pública.

    ![FortiGate SSL VPN](public-ip-address.png)

3. Haga clic en **Estática** y después en **Guardar**.

_Asignación de un nombre de dominio completo_

Si posee un nombre de dominio enrutable públicamente para el entorno en el que se va a implementar la máquina virtual de FortiGate, cree un registro de host (A) para la máquina virtual que se asigna a la dirección IP pública que se ha asignado de forma estática anteriormente.

### <a name="create-a-new-inbound-network-security-group-rule-for-tcp-port"></a>Creación de una regla del grupo de seguridad de red de entrada para el puerto TCP

1. Vaya a https://portal.azure.com y abra la configuración de la máquina virtual de FortiGate
2. En el menú de la izquierda, haga clic en **Redes**. Se mostrará la interfaz de red y las reglas del puerto de entrada.
3. Haga clic en **Agregar regla de puerto de entrada**.
4. Cree una nueva regla de puerto de entrada para TCP 8443.

    ![FortiGate SSL VPN](port-rule.png)

5. Haga clic en **Agregar**.


## <a name="create-a-custom-azure-app-for-fortigate"></a>Creación de una aplicación de Azure personalizada para FortiGate

1. Vaya a https://portal.azure.com y abra la hoja de Azure Active Directory del inquilino que proporcionará la identidad para los inicios de sesión de FortiGate.
2. Haga clic en **Aplicaciones empresariales** en el menú de la izquierda.
3. Haga clic en **Nueva aplicación**.
4. Haga clic en **Aplicación situada fuera de la galería**.
5. Proporcione un nombre (por ejemplo, FortiGate) y haga clic en **Agregar**.
6. Haga clic en **Usuarios y grupos** en el menú de la izquierda.
7. Agregue los usuarios que podrán iniciar sesión y haga clic en **Asignar**.
8. Haga clic en **Inicio de sesión único** en el menú de la izquierda.
9. Haga clic en **SAML**.
10. En la sección **Configuración básica de SAML**, haga clic en el lápiz para editar la configuración.
11. Configuración
    - El identificador (identificador de entidad) para que sea `https://<address>/remote/saml/metadata`.
    - La dirección URL de respuesta (URL del Servicio de consumidor de aserciones) para que sea `https://<address>/remote/saml/login`
    - La URL de cierre de sesión para que sea `https://<address>/remote/saml/logout`

    Donde `address` es el nombre de dominio completo o la dirección IP pública asignada a la máquina virtual de FortiGate.

    Guarde cada una de estas direcciones URL para su uso posterior:

    - El identificador de entidad
    - URL de respuesta
    - URL de cierre de sesión
12. Haga clic en **Guardar**
13. Cierre la configuración básica de SAML
14. Under **3: Certificado de firma de SAML**, descargue el **Certificado (Base64)** y guárdelo para más adelante.
15. En **4: Set up (App Name)** (4: Configurar [Nombre de la aplicación]), copie la dirección URL de inicio de sesión de Azure, el identificador de Azure AD y la dirección URL de cierre de sesión de Azure y guárdelos para más tarde.
    - Dirección URL de inicio de sesión de Azure
    - Identificador de Azure AD
    - Dirección URL de cierre de sesión de Azure
16. En **2: Atributos y notificaciones de usuario**, haga clic en el lápiz para editar la configuración.
17. Haga clic en **Agregar nueva notificación**.
18. Establezca el nombre en **nombre de usuario**.
19. Establezca el atributo de origen en **user.userprincipalname**.
20. Haga clic en **Guardar**
21. Haga clic en **Agregar una notificación de grupo**.
22. Seleccione **Todos los grupos**.
23. Marque **Personalizar nombre de la notificación del grupo**.
24. Establezca el nombre en **grupo**.
25. Haga clic en **Guardar**


## <a name="prepare-for-group-matching"></a>Preparación para la coincidencia de grupos

FortiGate permite diferentes experiencias del portal de usuarios después del inicio de sesión en función de la pertenencia a grupos. Por ejemplo, puede haber una experiencia para el grupo Marketing y otra para el grupo Finanzas.

Configure esto de la siguiente manera:

### <a name="create-groups-for-users"></a>Creación de grupos para los usuarios

1. Vaya a https://portal.azure.com y abra la hoja de Azure Active Directory del inquilino que proporcionará la identidad para los inicios de sesión de FortiGate.
2. Haga clic en **Grupos**.
3. Haga clic en **Nuevo grupo**.
4. Cree un grupo con
    - Tipo de grupo = Seguridad
    - Nombre del grupo = `a meaningful name`
    - Descripción del grupo = `a meaningful description for the group`
    - Tipo de pertenencia = Asignado.
    - Miembros = `users for the user experience that will map to this group`
5. Repita los pasos 3 y 4 para las experiencias de usuario adicionales.
6. Una vez creados los grupos, seleccione cada grupo y registre el identificador de objeto de cada uno.
7. Guarde estos identificadores de objeto y los nombres de grupo para más adelante.


## <a name="configure-the-fortigate-vm"></a>Configuración de la máquina virtual de FortiGate

### <a name="install-the-license"></a>Instalación de la licencia

1. Vaya a `https://<address>`.

    Aquí `address` es el nombre de dominio completo o la dirección IP pública asignada a la máquina virtual de FortiGate.

2. Continúe después de los errores de certificado.
3. Inicie sesión con las credenciales de administrador proporcionadas durante la implementación de la máquina virtual de FortiGate
4. Si la implementación usa el modelo BYOL, se mostrará un mensaje para cargar una licencia. Seleccione el archivo de licencia creado anteriormente y cárguelo, haga clic en **Aceptar** y reinicie la máquina virtual de FortiGate:

    ![FortiGate SSL VPN](license.png)

5. Después del reinicio, inicie sesión de nuevo con las credenciales de administrador para validar la licencia.

### <a name="update-firmware"></a>Actualización del firmware

1. Vaya a `https://<address>`.

    Aquí `address` es el nombre de dominio completo o la dirección IP pública asignada a la máquina virtual de FortiGate.

2. Continúe después de los errores de certificado.
3. Inicie sesión con las credenciales de administrador proporcionadas durante la implementación de la máquina virtual de FortiGate
4. En el menú de la izquierda, haga clic en **System** (Sistema).
5. En el menú de la izquierda que aparece bajo System (Sistema), haga clic en **Firmware**.
6. En la página Firmware Management (Administración de firmware), haga clic en **Browse** (Examinar) y seleccione el archivo de firmware descargado anteriormente.
7. Ignore la advertencia y haga clic en **Backup config and upgrade** (Realizar copia de seguridad de la configuración y actualizar):

    ![FortiGate SSL VPN](backup-configure-upgrade.png)

8. Haga clic en **Continue** (Continuar).
9. Cuando se le pida que guarde la configuración de FortiGate (como un archivo .conf), haga clic en **Save** (Guardar).
10. Espere a que se cargue el firmware para que se aplique y para que se reinicie la máquina virtual de FortiGate.
11. Una vez reiniciada, inicie sesión de nuevo con las credenciales de administrador.
12. Cuando se le pida que realice la configuración del panel, haga clic en **Later** (Más tarde).
13. Cuando comience el vídeo del tutorial, haga clic en **OK** (Aceptar).

### <a name="change-the-management-port-to-tcp"></a>Cambio del puerto de administración a TCP

1. Vaya a `https://<address>`.

    Aquí `address` es el nombre de dominio completo o la dirección IP pública asignada a la máquina virtual de FortiGate.

2. Continúe después de los errores de certificado.
3. Inicie sesión con las credenciales de administrador proporcionadas durante la implementación de la máquina virtual de FortiGate
4. En el menú de la izquierda, haga clic en **System** (Sistema).
5. En Administration Settings (Configuración de administración), cambie el puerto HTTPS a **8443**.
6. Haga clic en **Aplicar**.
7. Una vez que se aplica el cambio, el explorador intentará volver a cargar la página Administration (Administración), pero se producirá un error. A partir de ahora, la dirección de esta página será `https://<address>`.

    ![FortiGate SSL VPN](certificate.png)

### <a name="upload-the-azure-active-directory-saml-signing-certificate"></a>Carga del certificado de firma de SAML para Azure Active Directory

1. Vaya a `https://<address>`.

    Aquí `address` es el nombre de dominio completo o la dirección IP pública asignada a la máquina virtual de FortiGate.

2. Continúe después de los errores de certificado.
3. Inicie sesión con las credenciales de administrador proporcionadas durante la implementación de la máquina virtual de FortiGate
4. En el menú de la izquierda, haga clic en **System** (Sistema).
5. En System (Sistema), haga clic en **Certificates** (Certificados).
6. Haga clic en **Import** (Importar) - > **Remote Certificate** (Certificado remoto)
7. Busque el certificado descargado de la implementación de la aplicación personalizada FortiGate en el inquilino de Azure, selecciónelo y haga clic en **OK** (Aceptar).

### <a name="upload-and-configure-a-custom-ssl-certificate"></a>Carga y configuración de un certificado SSL personalizado

Es posible que quiera configurar la máquina virtual de FortiGate con su propio certificado SSL que admita el nombre de dominio completo que está usando. Si tiene acceso a un certificado SSL empaquetado con la clave privada en formato .PFX, se puede usar para este propósito.

1. Vaya a `https://<address>`.

    Aquí `address` es el nombre de dominio completo o la dirección IP pública asignada a la máquina virtual de FortiGate.

2. Continúe después de los errores de certificado.
3. Inicie sesión con las credenciales de administrador proporcionadas durante la implementación de la máquina virtual de FortiGate
4. En el menú de la izquierda, haga clic en **System** (Sistema).
5. En System (Sistema), haga clic en **Certificates** (Certificados).
6. Haga clic en **Import** (Importar) - > **Local Certificate** (Certificado local).
7. Haga clic en **PKCS #12 Certificate** (Certificado PKCS núm. 12).
8. Vaya al archivo .PFX que contiene el certificado SSL y la clave privada.
9. Proporcione la contraseña PFX.
10. Proporcione un nombre descriptivo para el certificado.
11. Haga clic en **Aceptar**
12. En el menú de la izquierda, haga clic en **System** (Sistema).
13. En System (Sistema), haga clic en **Settings** (Configuración).
14. En Administration Settings (Configuración de administración), expanda la lista desplegable situada junto al certificado de servidor HTTPS y seleccione el certificado SSL importado anteriormente.
15. Haga clic en **Aplicar**.
16. Cierre la ventana del explorador y después vaya de nuevo a `https://<address>`.
17. Inicie sesión con las credenciales de administrador de FortiGate y observe el certificado SSL correcto en uso.


### <a name="perform-command-line-configuration"></a>Configuración de la línea de comandos

_Configuración de la línea de comandos para la autenticación SAML_

1. Vaya a https://portal.azure.com y abra la configuración de la máquina virtual de FortiGate.
2. En el menú de la izquierda, haga clic en **Consola serie**.
3. Inicie de sesión en la consola serie con las credenciales de administrador de la máquina virtual de FortiGate.

    En el paso siguiente, se necesitarán las direcciones URL registradas anteriormente. Concretamente:

    - El identificador de entidad
    - URL de respuesta
    - URL de cierre de sesión
    - Dirección URL de inicio de sesión de Azure
    - Identificador de Azure AD
    - Dirección URL de cierre de sesión de Azure
4. En la consola serie ejecute los siguientes comandos:

    ```
    config user saml
    edit azure
    set entity-id <Entity ID>
    set single-sign-on-url <Reply URL>
    set single-logout-url <Logout URL>
    set idp-single-sign-on-url <Azure Login URL>
    set idp-entity-id <Azure AD Identifier>
    set idp-single-logout-url <Azure Logout URL>
    set idp-cert REMOTE_Cert_
    set user-name username
    set group-name group
    end
    ```
    > [!NOTE]
    > La dirección URL de cierre de sesión de Azure contiene un carácter "?". Esto requiere una secuencia de clave especial para que se proporcione correctamente a la consola serie de FortiGate. La dirección URL suele ser:

    `https://login.microsoftonline.com/common/wsfederation?wa=wsignout1`

    Para proporcionar esto en la consola serie, escriba

    ```
    set idp-single-logout-url https://login.microsoftonline.com/common/wsfederation
    ```
    Después pulse CTRL+V

    A continuación, pegue el resto de la dirección URL para completar la línea.

    ```
    set idp-single-logout-url
    https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.
    ```

5. Para confirmar la configuración, ejecute

    ```
    show user saml
    ```

_Configuración de la línea de comandos para la coincidencia de grupos_

1. Vaya a https://portal.azure.com y abra la configuración de la máquina virtual de FortiGate.
2. En el menú de la izquierda, haga clic en **Consola serie**.
3. Inicie de sesión en la consola serie con las credenciales de administrador de la máquina virtual de FortiGate.
4. En la consola serie ejecute los siguientes comandos:

    ```
    config user group
    edit <group 1 name>
    set member azure
    config match
    edit 1
    set server-name azure
    set group-name <group 1 Object Id>
    next
    end
    next
    ```

    Repita estos comandos a partir de "edit `group 1 name`" para cada grupo adicional que tendrá una experiencia de portal diferente en FortiGate.

_Configuración de la línea de comandos para el tiempo de espera de autenticación_

1. Vaya a https://portal.azure.com y abra la configuración de la máquina virtual de FortiGate.
2. En el menú de la izquierda, haga clic en **Consola serie**.
3. Inicie de sesión en la consola serie con las credenciales de administrador de la máquina virtual de FortiGate.
4. En la consola serie ejecute los siguientes comandos:

    ```
    config system global
    set remoteauthtimeout 60
    end
    ```
### <a name="create-vpn-portals-and-firewall-policy"></a>Creación de portales de VPN y de la directiva de firewall

1. Vaya a `https://<address>`.

    Aquí `address` es el nombre de dominio completo o la dirección IP pública asignada a la máquina virtual de FortiGate.

2. Inicie sesión con las credenciales de administrador proporcionadas durante la implementación de la máquina virtual de FortiGate
3. En el menú de la izquierda, haga clic en **VPN**.
4. En VPN, haga clic en **SSL-VPN Portals** (Portales SSL-VPN)
5. Haga clic en **Create New** (Crear nuevo).
6. Proporcione un nombre (que normalmente coincide con el grupo de Azure que se usa para proporcionar la experiencia de portal personalizada).
7. Haga clic en el signo más ( **+** ) situado junto a Source IP Pools (Grupos de direcciones IP de origen), seleccione el grupo predeterminado y haga clic en **Close** (Cerrar).
8. Personalice la experiencia para este grupo. Para las pruebas, esto puede consistir en la personalización del mensaje y el tema del portal. Aquí también puede crear marcadores personalizados que dirijan a los usuarios a recursos internos.
9. Haga clic en **Aceptar**
10. Repita los pasos 5 a 9 para cada grupo de Azure que tendrá una experiencia de portal personalizada.
11. En VPN, haga clic en **SSL-VPN Settings** (Configuración de SSL-VPN)
12. Haga clic en el signo más ( **+** ) junto a Listen on Interfaces (Escuchar en interfaces).
13. Seleccione **Port1** y haga clic en **Close** (Cerrar).


14. Si se instaló previamente un certificado SSL personalizado, cambie el certificado de servidor para que utilice el certificado SSL personalizado en el menú desplegable.
15. En Authentication/Portal Mapping (Autenticación/Asignación del portal), haga clic en **Create New** (Crear nuevo).
16. Elija el primer grupo de Azure y hágalo coincidir con el portal del mismo nombre.
17. Haga clic en **Aceptar**
18. Repita los pasos del 15 al 17 para cada par de grupo y portal de Azure
19. En Authentication/Portal Mapping (Autenticación/Asignación del portal), edite **All Other Users/Groups** (Todos los demás usuarios/Grupos).
20. Establezca el portal en **full-access** (acceso completo)
21. Haga clic en **Aceptar**
22. Haga clic en **Aplicar**.
23. Desplácese hasta la parte superior de la página SSL-VPN Setting (Configuración de SSL-VPN) y haga clic en la advertencia que indica que **no existen directivas de SSL-VPN**
     **. Haga clic aquí para crear una nueva directiva SSL-VPN con esta configuración**.
24. Proporcione un nombre, como **VPN Grp**.
25. Establezca Outgoing Interface (Interfaz de salida) en **port** (puerto)
26. Haga clic en **Source** (Origen).
27. En Address (Dirección), seleccione **all** (todas).
28. En User (Usuario), seleccione el primer grupo de Azure.
29. Haga clic en **Cerrar**.
30. Haga clic en **Destination** (Destino).
31. En Address (Dirección), esta suele ser la red interna. Seleccione login.microsoft.com para realizar pruebas.
32. Haga clic en **Cerrar**.
33. Haga clic en **Service** (Servicio).
34. Haga clic en **All** (Todos).
35. Haga clic en **Cerrar**.
36. Haga clic en **Aceptar**
37. En el menú de la izquierda, haga clic en **Policy & Objects** (Directiva y objetos).
38. En Policy & Objects (Directiva y objetos), haga clic en **Directiva de firewall**.
39. Expanda **SSL-VPN tunnel interface (Interfaz de túnel de SSL-VPN) (ssl.root) -> port (puerto)**
40. Haga clic con el botón derecho en la directiva de VPN creada anteriormente ( **VPN Grp 1** ) y seleccione **Copy** (Copiar).
41. Haga clic con el botón derecho en la directiva de VPN y seleccione **Paste** (Pegar) -> **Below** (Debajo).
42. Edite la nueva directiva, proporcionándole un nombre diferente (por ejemplo **VPN Grp2**) y cambiando el grupo al que se aplica (otro grupo de Azure).
43. Haga clic con el botón derecho en la nueva directiva y establezca el estado en **Habilitado**.


## <a name="test-sign-in-using-azure"></a>Prueba del inicio de sesión con Azure

1. Mediante una sesión InPrivate del explorador, vaya a `https://<address>`. 
2. El inicio de sesión debería redirigir a Azure Active Directory para el inicio de sesión.
3. Después de proporcionar las credenciales para un usuario que se ha asignado a la aplicación FortiGate en el inquilino de Azure, debería aparecer el portal de usuarios adecuado.

    ![FortiGate SSL VPN](test-sign-in.png)
