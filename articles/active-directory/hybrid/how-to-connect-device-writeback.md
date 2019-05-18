---
title: 'Azure AD Connect: habilitación de la escritura diferida de dispositivo | Microsoft Docs'
description: En este documento se describe cómo habilitar la escritura diferida de dispositivo con Azure AD Connect
services: active-directory
documentationcenter: ''
author: billmath
manager: femila
editor: curtand
ms.assetid: c0ff679c-7ed5-4d6e-ac6c-b2b6392e7892
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/08/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 977b599c26e8bb586cc47bd2f0aac80034f22834
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65785727"
---
# <a name="azure-ad-connect-enabling-device-writeback"></a>Azure AD Connect: habilitación de la escritura diferida de dispositivo
> [!NOTE]
> Se necesita una suscripción a Azure AD Premium para la reescritura de dispositivos.
> 
> 

En la siguiente documentación se ofrece información sobre cómo habilitar la característica de escritura diferida de dispositivo en Azure AD Connect. La escritura diferida de dispositivo se usa en los siguientes escenarios:

* Habilitar [Windows Hello para empresas mediante implementación híbrida de confianza de certificado](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-hybrid-cert-trust-prereqs#device-registration)
* Habilite el acceso condicional basado en dispositivos para aplicaciones protegida de ADFS (2012 R2 o superior) (relaciones de confianza para usuario autenticado).

Esto ofrece seguridad adicional y la garantía de que el acceso a las aplicaciones solo se concede para dispositivos de confianza. Para más información sobre el acceso condicional, consulte [Administración de riesgos con el acceso condicional](../active-directory-conditional-access-azure-portal.md) y [Configuración del acceso condicional local mediante el Registro de dispositivos de Azure Active Directory](../../active-directory/active-directory-device-registration-on-premises-setup.md).

> [!IMPORTANT]
> <li>Los dispositivos deben encontrarse en el mismo bosque que los usuarios. Puesto que los dispositivos deben volver a escribirse en un único bosque, esta característica no admite actualmente una implementación con varios bosques de usuarios.</li>
> <li>Solo se puede agregar un objeto de configuración de registro de dispositivo al bosque de Active Directory local. Esta característica no es compatible con una topología en la que el Active Directory local está sincronizado a varios directorios de Azure AD.</li>

## <a name="part-1-install-azure-ad-connect"></a>Parte 1: Instalación de Azure AD Connect
Instale Azure AD Connect mediante la configuración rápida o personalizada. Microsoft recomienda empezar con todos los usuarios y grupos sincronizados correctamente antes de habilitar la escritura diferida de dispositivos.

## <a name="part-2-enable-device-writeback-in-azure-ad-connect"></a>Parte 2: habilitar la escritura diferida de dispositivos en Azure AD Connect
1. Ejecute de nuevo el Asistente para la instalación. Seleccione **Configurar opciones de dispositivo** en la página Tareas adicionales y haga clic en **Siguiente**. 

    ![Configurar opciones de dispositivo](./media/how-to-connect-device-writeback/deviceoptions.png)

    >[!NOTE]
    > La nueva opción Configurar opciones de dispositivo está disponible únicamente en la versión 1.1.819.0 y posteriores.

2. En la página de opciones del dispositivo, seleccione **Configurar la escritura diferida de dispositivo**. La opción **Deshabilitar la escritura diferida de dispositivo** no estará disponible hasta que se habilite la escritura diferida de dispositivo. Haga clic en **Siguiente** para ir a la siguiente página del asistente.
    ![Elección de la operación de dispositivos](./media/how-to-connect-device-writeback/configuredevicewriteback1.png)

3. En la página de escritura diferida, verá el dominio suministrado como el bosque de escritura diferida de dispositivos predeterminado.
   ![Instalación personalizada del bosque de destino de escritura diferida de dispositivos](./media/how-to-connect-device-writeback/writebackforest.png)

4. La página **Contenedor de dispositivos** ofrece la opción de preparar Active Directory usando una de las dos opciones disponibles:

     a. **Proporcionar credenciales de administrador de empresa**: si se proporcionan las credenciales de administrador de empresa para el bosque donde deben volver a escribirse los dispositivos, Azure AD Connect preparará el bosque automáticamente durante la configuración de la escritura diferida de dispositivos.

    b. **Descargar el script de PowerShell**: Azure AD Connect genera automáticamente un script de PowerShell que puede preparar Active Directory para la escritura diferida de dispositivos. En caso de que no puedan proporcionarse las credenciales de administrador de empresa en Azure AD Connect, es recomendable descargar el script de PowerShell. Proporcione el script de PowerShell descargado **CreateDeviceContainer.psq** al administrador de empresa del bosque donde se volverán a escribir los dispositivos.
    ![Preparar el bosque de Active Directory](./media/how-to-connect-device-writeback/devicecontainercreds.png)
    
    Para preparar el bosque de Active Directory, se realizan las siguientes operaciones:
    * Si todavía no existen, se crean y configuran nuevos contenedores y objetos en CN=Device Registration Configuration,CN=Services,CN=Configuration,[forest-dn].
    * Si todavía no existen, se crean y se configuran nuevos contenedores y objetos en CN=RegisteredDevices,[domain-dn]. Los objetos de dispositivo se crearán en este contenedor.
    * Establece los permisos necesarios en la cuenta de Azure AD Connector, para administrar dispositivos en su Active Directory.
    * Solo necesita ejecutarse en un bosque, incluso si se está instalando Azure AD Connect en varios bosques.

## <a name="verify-devices-are-synchronized-to-active-directory"></a>Comprobar que los dispositivos están sincronizados con Active Directory
La reescritura de dispositivos debería funcionar ahora correctamente. Tenga en cuenta que se puede tardar hasta tres horas en que los objetos de dispositivos se vuelvan a escribir en AD.  Para comprobar que los dispositivos que se están sincronizados correctamente, siga este procedimiento después de completar las reglas de sincronización:

1. Inicie el Centro de administración de Active Directory.
2. Expanda RegisteredDevices dentro del dominio que se está federando.

   ![Dispositivos registrados del centro de administración de Active Directory](./media/how-to-connect-device-writeback/devicewriteback5.png)

3. Los dispositivos actualmente registrados aparecerán en la lista.

   ![Lista de dispositivos registrados del centro de administración de Active Directory](./media/how-to-connect-device-writeback/devicewriteback6.png)

## <a name="enable-conditional-access"></a>Habilitar el acceso condicional
Encontrará a su disposición instrucciones detalladas para habilitar este escenario en [Configuración del acceso condicional local mediante el registro de dispositivos de Azure Active Directory](../../active-directory/active-directory-device-registration-on-premises-setup.md).

## <a name="troubleshooting"></a>solución de problemas
### <a name="the-writeback-checkbox-is-still-disabled"></a>La casilla de reescritura sigue deshabilitada
Si la casilla para la reescritura de dispositivos no se habilita a pesar de haber seguido los pasos anteriores, los siguientes pasos le guiarán por lo que el Asistente para la instalación comprueba antes de habilitar la casilla.

En primer lugar:

* El bosque donde están los dispositivos deben tener el esquema de bosque actualizado al nivel de Windows 2012 R2 para que existan el objeto de dispositivo y los atributos asociados.
* Si el Asistente para la instalación ya se está ejecutando, algunos cambios no se detectarán. En este caso, complete el Asistente para la instalación y ejecútelo de nuevo.
* Asegúrese de que la cuenta que proporciona en el script de inicialización sea realmente la del usuario correcto usado por Active Directory Connector. Para ello, siga estos pasos:
  * En el menú Inicio, abra **Servicio de sincronización**.
  * Abra la pestaña **Conectores** .
  * Busque el conector con el tipo Dominio de Active Directory y selecciónelo.
  * En **Acciones**, seleccione **Propiedades**.
  * Vaya a **Conexión al bosque de Active Directory**. Compruebe que el dominio y el nombre de usuario especificados en esta pantalla coinciden con la cuenta proporcionada para el script.
    ![Cuenta de conector en Sync Service Manager](./media/how-to-connect-device-writeback/connectoraccount.png)

Compruebe la configuración en Active Directory:

* Asegúrese de que el servicio de registro del dispositivo se encuentra en la siguiente ubicación (CN=DeviceRegistrationService,CN=Servicios de registro del dispositivo,CN=Configuración de registro del dispositivo,CN=Servicios,CN=Configuración) en el contexto de nomenclatura de la configuración.

![Solución de problemas, DeviceRegistrationService en el espacio de nombres de configuración](./media/how-to-connect-device-writeback/troubleshoot1.png)

* Compruebe que haya un solo objeto de configuración; para ello, busque en el espacio de nombres de configuración. Si hay más de uno, elimine el duplicado.

![Solución de problemas, buscar objetos duplicados](./media/how-to-connect-device-writeback/troubleshoot2.png)

* En el objeto Servicio de registro del dispositivo, asegúrese de que exista el atributo msDS-DeviceLocation y que tenga un valor. Busque en esta ubicación y asegúrese de que exista con el tipo de objeto msDS-DeviceContainer.

![Solución de problemas, msDS-DeviceLocation](./media/how-to-connect-device-writeback/troubleshoot3.png)

![Solución de problemas, clase de objeto RegisteredDevices](./media/how-to-connect-device-writeback/troubleshoot4.png)

* Compruebe que la cuenta usada por el conector de Active Directory tenga los permisos necesarios en el contenedor Dispositivos registrados encontrado mediante el paso anterior. Este es el permiso esperado en este contenedor:

![Solución de problemas, comprobar los permisos en el contenedor](./media/how-to-connect-device-writeback/troubleshoot5.png)

* Compruebe que la cuenta de Active Directory tenga permisos en el objeto CN=Configuración de registro del dispositivo, CN=Servicios, CN=Configuración.

![Solución de problemas, comprobar los permisos en la configuración del registro de dispositivos](./media/how-to-connect-device-writeback/troubleshoot6.png)

## <a name="additional-information"></a>Información adicional
* [Administración de riesgos con el acceso condicional](../active-directory-conditional-access-azure-portal.md)
* [Configuración del acceso condicional local mediante el registro de dispositivos de Azure Active Directory](../../active-directory/active-directory-device-registration-on-premises-setup.md)

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).

