---
title: Preguntas más frecuentes sobre Azure Active Directory Connect | Microsoft Docs
description: En este artículo se responden a las preguntas más frecuentes sobre Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 4e47a087-ebcd-4b63-9574-0c31907a39a3
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 08/23/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: fe4350a721b025858493739e78011458f89f33ee
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89276344"
---
# <a name="azure-active-directory-connect-faq"></a>Preguntas más frecuentes sobre Azure Active Directory Connect

## <a name="general-installation"></a>Instalación general

**P: ¿Cómo puedo reforzar mi servidor Azure AD Connect para reducir la superficie de ataque de seguridad?**

Microsoft recomienda reforzar su servidor Azure AD Connect para reducir la superficie de ataque de seguridad para este componente esencial de su entorno de TI.  Siga las recomendaciones que tiene a continuación y podrá reducir los riesgos de seguridad de su organización.

* Implemente Azure AD Connect en un servidor unido a un dominio y restrinja el acceso administrativo a los administradores del dominio o a otros grupos de seguridad rigurosamente controlados.

Para obtener más información, consulte: 

* [Protección de los grupos de administradores](/windows-server/identity/ad-ds/plan/security-best-practices/appendix-g--securing-administrators-groups-in-active-directory)

* [Protección de las cuentas predefinidas de administrador](/windows-server/identity/ad-ds/plan/security-best-practices/appendix-d--securing-built-in-administrator-accounts-in-active-directory)

* [Mejora de la seguridad y el sostenimiento al reducir las superficies de ataque](/windows-server/identity/securing-privileged-access/securing-privileged-access#2-reduce-attack-surfaces )

* [Reducción de la superficie de ataque de Active Directory](/windows-server/identity/ad-ds/plan/security-best-practices/reducing-the-active-directory-attack-surface)

**P: ¿Funcionará la instalación si el administrador global de Azure Active Directory (Azure AD) tiene habilitada la autenticación en dos fases (2FA)?**  
A partir de las compilaciones de febrero de 2016, se admite este escenario.

**P: ¿Existe alguna manera de instalar Azure AD Connect de manera desatendida?**  
La instalación de Azure AD Connect se admite solo cuando se usa el Asistente para la instalación. No se admite la instalación silenciosa desatendida.

**P: Tengo un bosque en el que no se puede establecer comunicación con un dominio. ¿Cómo se puede instalar Azure AD Connect?**  
A partir de las compilaciones de febrero de 2016, se admite este escenario.

**P: ¿Funciona el agente de estado de Azure Active Directory Domain Services (Azure AD DS) en Server Core?**  
Sí. Después de instalar el agente, puede completar el proceso de registro mediante el siguiente cmdlet de PowerShell: 

`Register-AzureADConnectHealthADDSAgent -Credentials $cred`

**P: ¿Admite Azure AD Connect la sincronización de dos dominios a uno de Azure AD?**  
Sí, este escenario se admite. Consulte [Varios dominios](how-to-connect-install-multiple-domains.md).
 
**P: ¿Se pueden tener varios conectores para el mismo dominio de Active Directory en Azure AD Connect?**  
No, no se admiten varios conectores para el mismo dominio de AD. 

**P: ¿Se puede mover la base de datos de Azure AD Connect de la base de datos local a una instancia remota de SQL Server?**    
Sí, en los pasos siguientes se proporcionan instrucciones generales sobre cómo hacerlo. Actualmente estamos trabajando en un documento más detallado.
1. Realice una copia de seguridad de la base de datos LocalDB ADSync.
La manera más sencilla de hacerlo es usar SQL Server Management Studio instalado en el mismo equipo que Azure AD Connect. Conéctese a *(LocalDb).\ADSync* y, a continuación, realice una copia de seguridad de la base de datos ADSync.

2. Restaure la base de datos ADSync en la instancia remota de SQL Server.

3. Instale Azure AD Connect en la instancia remota de [SQL Database](how-to-connect-install-existing-database.md) existente.
   En el artículo se muestra cómo migrar para usar una instancia local de SQL Database. Si realiza la migración para usar una instancia remota de SQL Database, en el paso 5 del proceso también debe especificar una cuenta de servicio existente para la ejecución del servicio de sincronización de Windows. Esta cuenta de servicio del motor de sincronización se describe aquí:
   
      **Uso de una cuenta de servicio existente**: de forma predeterminada, Azure AD Connect usa una cuenta de servicio virtual para que la utilicen los servicios de sincronización. Si usa una instancia remota de SQL Server o un proxy que requiera autenticación, debe usar una cuenta de servicio administrada o una cuenta de servicio en el dominio, y conocer la contraseña. En esos casos, especifique la cuenta que se va a usar. Asegúrese de que los usuarios que ejecuten la instalación sean administradores del sistema de SQL para que se pueden crear credenciales de inicio de sesión para la cuenta de servicio. Para más información, consulte [Azure AD Connect: cuentas y permisos](reference-connect-accounts-permissions.md#adsync-service-account). 
   
      Con la versión más reciente, el administrador SQL puede realizar ahora el aprovisionamiento de la base de datos fuera de banda y luego el administrador de Azure AD Connect puede instalarlo con derechos de propietario de la base de datos. Para más información, consulte [Instalación de Azure AD Connect con permisos de administrador delegado de SQL](how-to-connect-install-sql-delegation.md).

Para simplificar las cosas, se recomienda que los usuarios que instalen Azure AD Connect sean administradores del sistema en SQL. Sin embargo, con las últimas compilaciones se pueden usar administradores de SQL delegados, como se describe en [Instalación de Azure AD Connect con permisos de administrador delegado de SQL](how-to-connect-install-sql-delegation.md).

**P: ¿Cuáles son algunos de los procedimientos recomendados del campo?**  

El siguiente es un documento informativo que presenta algunos de los procedimientos recomendados que el departamento de ingeniería, de soporte técnico y consultoría han desarrollado a lo largo de los años.  Se lo mostramos en una lista con viñetas a las que puede hacer referencia rápidamente.  Aunque esta lista trata de ser exhaustiva, es posible que haya procedimientos recomendados adicionales que aún no se hayan incluido en la lista.

- Si usa la versión completa de SQL, debe seguir en la versión local y no usar la remota.
    - Menor número de saltos
    - Más fácil de solucionar
    - Menor complejidad
    - Debe designar recursos para SQL y permitir la sobrecarga de Azure AD Connect y el sistema operativo.
- Omita el Proxy si es posible; si no, debe asegurarse de que el valor del tiempo de espera sea superior a 5 minutos.
- Si se requiere un proxy, debe agregarlo al archivo machine.config.
- Esté al tanto de los trabajos y el mantenimiento locales de SQL y asegúrese de controlar cómo afectarán a Azure AD Connect, especialmente a la hora de realizar la indexación.
- Asegúrese de que DNS puede resolverse externamente.
- Asegúrese de que las [especificaciones del servidor](how-to-connect-install-prerequisites.md#hardware-requirements-for-azure-ad-connect) se recomienden si está usando servidores físicos o virtuales.
- Si está utilizando un servidor virtual, asegúrese de que los recursos necesarios están dedicados.
- Asegúrese de que el disco y la configuración del disco cumplen los procedimientos recomendados de SQL Server.
- Instale y configure Azure AD Connect Health para la supervisión.
- Use el umbral de eliminación integrado en Azure AD Connect.
- Revise cuidadosamente las actualizaciones de la versión y así poder estar preparado para todos los cambios y los nuevos atributos que se puedan agregar.
- Haga una copia de seguridad de todo.
    - Copia de seguridad de las claves
    - Copia de seguridad de las reglas de sincronización
    - Copia de seguridad de la configuración del servidor
    - Copia de seguridad de una instancia de SQL Database
- Asegúrese de que no haya agentes de copia de seguridad de terceros que realicen copias de seguridad de SQL sin VSS Writer de SQL (que es común en servidores virtuales con instantáneas de terceros).
- Limite la cantidad de reglas de sincronización personalizadas que se usan, ya que agregan complejidad.
- Trate los servidores de Azure AD Connect como servidores de nivel 0.
- Desconfíe de la modificación de las reglas de sincronización en la nube si no comprende bien el impacto que pueden suponer ni los controladores de negocios adecuados.
- Asegúrese de que la dirección URL correcta y los puertos de Firewall estén abiertos para obtener soporte técnico de Azure AD Connect y Azure AD Connect Health.
- Aproveche el atributo filtrado en la nube para solucionar problemas y evitar objetos fantasma.
- Con el servidor de almacenamiento provisional, asegúrese de que está usando el documentador de configuración de Azure AD Connect para mantener la coherencia entre los servidores.
- Los servidores de almacenamiento provisional deben estar en centros de datos separados (ubicaciones físicas).
- Los servidores de almacenamiento provisional no están diseñados para ser una solución de alta disponibilidad, pero puede tener varios servidores de almacenamiento provisional.
- La introducción de un servidor de almacenamiento provisional de tipo "Lag" podría mitigar algunos posibles tiempos de inactividad en caso de error.
- Pruebe y valide todas las actualizaciones en el servidor de almacenamiento provisional primero.
- Valide siempre las exportaciones antes de pasar al servidor de almacenamiento provisional.  Aproveche el servidor de almacenamiento provisional para las importaciones completas y las sincronizaciones completas a fin de reducir el impacto empresarial
- Mantenga la consistencia de la versión entre los servidores de Azure AD Connect tanto como sea posible. 

**P: ¿Puedo permitir que Azure AD Connect cree la cuenta del conector de Azure AD en la máquina del grupo de trabajo?**
No.  Para permitir que Azure AD Connect cree automáticamente la cuenta del conector de Azure AD, la máquina debe estar unida al dominio.  

## <a name="network"></a>Red
**P: Tengo un firewall, un dispositivo de red u otro elemento que limita el tiempo que las conexiones pueden permanecer abiertas en mi red. ¿Cuál debería ser mi umbral de tiempo de expiración del cliente al usar Azure AD Connect?**  
Todo el software de redes, dispositivos físicos o cualquier otra cosa que limite el tiempo máximo que las conexiones permanecen abiertas deben usar un umbral de un mínimo de cinco minutos (300 segundos) para la conectividad entre el servidor donde está instalado el cliente de Azure AD Connect y Azure Active Directory. Esta recomendación también se aplica a todas las herramientas de sincronización de identidades de Microsoft publicadas anteriormente.

**P: ¿Se admiten los dominios de una sola etiqueta (SLD)?**  
Aunque no se recomienda esta configuración de red ([consulte el artículo](https://support.microsoft.com/help/2269810/microsoft-support-for-single-label-domains)), se admite el uso de la sincronización de Azure AD Connect con un dominio de una sola etiqueta, siempre que la configuración de red para el dominio de un solo nivel funcione correctamente.

**P: ¿Se admiten los bosques con dominios de AD no contiguos?**  
No, Azure AD Connect no admite bosques locales que contengan espacios de nombres no contiguos.

**P: ¿Se admiten los nombres de NetBios "con puntos"?**  
No, Azure AD Connect no admite bosques o dominios locales en los que el nombre de NetBios contenga un punto (.).

**P: ¿Se admiten entornos puros de IPv6?**  
No, Azure AD Connect no admite entornos puros de IPv6.

**P: Tengo un entorno de varios bosques y la red entre los dos bosques es mediante NAT (traducción de direcciones de red). ¿Usa Azure AD Connect entre estos dos bosques admitidos?**</br>
No, no se admite el uso de Azure AD Connect sobre NAT. 

## <a name="federation"></a>Federación
**P: ¿Qué debo hacer si recibo un correo electrónico que me pide que renueve el certificado de Office 365?**  
Para obtener instrucciones acerca de cómo renovar el certificado, consulte el artículo sobre la [renovación de certificados](how-to-connect-fed-o365-certs.md).

**P: Tengo establecido "Actualizar automáticamente usuario confianza" para el usuario de confianza de Office 365. ¿Es necesario realizar alguna acción cuando se implemente automáticamente mi certificado de firma de tokens?**  
Siga las instrucciones que se describen en el artículo sobre la [renovación de certificados](how-to-connect-fed-o365-certs.md).

## <a name="environment"></a>Entorno
**P: ¿Se admite cambiar el nombre del servidor después de haber instalado Azure AD Connect?**  
No. Cambiar el nombre de servidor provoca que el motor de sincronización no se pueda conectar a la instancia de SQL Database y no se pueda iniciar el servicio.

**P: ¿Se admiten las reglas de sincronización Next Generation Cryptographic (NGC) en una máquina habilitada para FIPS?**  
No.  No se admiten.

**P. Si he deshabilitado un dispositivo sincronizado (por ejemplo, HAADJ) en Azure Portal, ¿por qué se vuelve a habilitar?**<br>
Los dispositivos sincronizados se pueden crear o controlar en el entorno local. Si un dispositivo sincronizado está habilitado en el entorno local, es posible que se vuelva a habilitar en Azure Portal incluso si un administrador lo deshabilitó anteriormente. Para deshabilitar un dispositivo sincronizado, use Active Directory local para deshabilitar la cuenta de equipo.

**P. Si bloqueo el inicio de sesión de usuario en el portal de Office 365 o Azure AD para los usuarios sincronizados, ¿por qué se desbloquea al iniciar sesión de nuevo?**<br>
Los usuarios sincronizados se pueden crear o controlar en el entorno local. Si la cuenta está habilitada en el entorno local, puede desbloquear el bloque de inicio de sesión colocado por el administrador.

## <a name="identity-data"></a>Datos de identidad
**P: ¿Por qué el atributo userPrincipalName (UPN) de Azure AD no coincide con el UPN local?**  
Para obtener más información, consulte estos artículos:

* [Usernames in Office 365, Azure, or Intune don't match the on-premises UPN or alternate login ID](https://support.microsoft.com/kb/2523192) (Los nombres de usuario de Office 365, Azure o Intune no coinciden con el UPN local o la id. de inicio de sesión alternativo)
* [Changes aren't synced by the Azure Active Directory sync tool after you change the UPN of a user account to use a different federated domain](https://support.microsoft.com/kb/2669550) (La herramienta de sincronización de Azure Active Directory no sincroniza los cambios después de cambiar el UPN de una cuenta de usuario para usar otro dominio federado)

También puede configurar Azure AD para permitir que el motor de sincronización actualice el UPN, como se describe en [Características del servicio de sincronización de Azure AD Connect](how-to-connect-syncservice-features.md).

**P: ¿Se admiten coincidencias parciales de objetos de contacto o grupo de Azure AD locales con objetos de contacto o grupo de Azure AD existentes?**  
Sí, esta coincidencia parcial se basa en la propiedad proxyAddress. No se admite la coincidencia parcial en grupos no habilitados para correo.

**P: ¿Se admite configurar manualmente el atributo ImmutableId en objetos de contacto o grupo de Azure AD existentes para conseguir una coincidencia exacta con objetos de contacto o grupo de Azure AD locales?**  
No, establecer manualmente el atributo ImmutableId en un objeto de contacto o grupo de Azure AD existente para conseguir una coincidencia exacta no se admite actualmente.

## <a name="custom-configuration"></a>Configuración personalizada
**P: ¿Dónde se documentan los cmdlets de PowerShell para Azure AD Connect?**  
A excepción de los cmdlets documentados en este sitio, el resto de cmdlets de PowerShell que se encuentran en Azure AD Connect no se admiten para uso del cliente.

**P: ¿Puedo usar la opción "Server export/server import" (Exportación/importación de servidor) que se encuentra en Synchronization Service Manager para mover la configuración entre servidores?**  
No. Esta opción no recupera todas las opciones de configuración y no debe usarse. En su lugar, use el asistente para crear la configuración base en el segundo servidor y utilizar el editor de reglas de sincronización para generar scripts de PowerShell para mover cualquier regla personalizada entre servidores. Para más información, consulte [Migración oscilante](how-to-upgrade-previous-version.md#swing-migration).

**P: ¿Se pueden almacenar en caché las contraseñas de la página de inicio de sesión de Azure y se puede impedir este almacenamiento en caché porque contenga un elemento de entrada de contraseña con el atributo *autocomplete = "false"* ?**  
Actualmente, no se admite la modificación de los atributos HTML del campo **Contraseña**, incluida la etiqueta de autocompletar. Estamos trabajando en una característica que permita JavaScript personalizado, que permita agregar cualquier atributo al campo **Contraseña**.

**P: En la página de inicio de sesión de Azure, se muestran nombres de usuario de los usuarios que han iniciado sesión anteriormente de forma correcta. ¿Este comportamiento se puede desactivar?**  
Actualmente no se admite la modificación de los atributos HTML del campo de entrada **Contraseña**, incluida la etiqueta de autocompletar. Estamos trabajando en una característica que permita JavaScript personalizado, que permita agregar cualquier atributo al campo **Contraseña**.

**P: ¿Existe alguna manera de evitar las sesiones simultáneas?**  
No.

## <a name="auto-upgrade"></a>Actualización automática

**P: ¿Cuáles son las ventajas y consecuencias de usar la actualización automática?**  
Se recomienda a todos los clientes que habiliten la actualización automática para la instalación de Azure AD Connect. Las ventajas son que siempre se reciben las revisiones más recientes, incluidas las actualizaciones de seguridad para vulnerabilidades detectadas en Azure AD Connect. El proceso de actualización es sencillo y se realiza automáticamente en cuanto se encuentra disponible una versión nueva. Muchos miles de clientes de Azure AD Connect usan la actualización automática con cada nueva versión.

El proceso de actualización automática siempre establece en primer lugar si una instalación es apta para actualización automática. Si es apta, se realiza y prueba la actualización. El proceso también incluye la búsqueda de cambios personalizados en las reglas y los factores de entorno específicos. Si las pruebas revelan que una actualización no se realizó correctamente, se restaura automáticamente la versión anterior.

Según el tamaño del entorno, el proceso puede tardar un par de horas. Mientras la actualización está en curso, no se produce ninguna sincronización entre Windows Server Active Directory y Azure AD.

**P: Recibí un correo electrónico para comunicarme que mi versión de actualización automática ya no funciona y que tengo que instalar una nueva. ¿Por qué tengo que hacerlo?**  
El año pasado se publicó una versión de Azure AD Connect que, en determinadas circunstancias, podría haber deshabilitado la característica de actualización automática en el servidor. Se ha corregido el problema en Azure AD Connect versión 1.1.750.0. Si se ha visto afectado por el problema, puede mitigarlo con la ejecución de un script de PowerShell para repararlo o actualizando manualmente a la versión más reciente de Azure AD Connect. 

Para ejecutar el script de PowerShell, [descargue el script](https://aka.ms/repairaadconnect) y ejecútelo en el servidor de Azure AD Connect en una ventana de PowerShell administrativa. Para obtener información sobre cómo ejecutar el script, [vea este breve vídeo](https://aka.ms/repairaadcau).

Para actualizar manualmente, debe descargar y ejecutar la última versión del archivo AADConnect.msi.
 
-  Si su versión actual es anterior a la 1.1.750.0, [descargue y actualice a la última versión](https://www.microsoft.com/download/details.aspx?id=47594).
- Si la versión de Azure AD Connect es 1.1.750.0 o posterior, no es necesario realizar ninguna otra acción. Ya está usando la versión que contiene la corrección de la actualización automática. 

**P: Recibí un correo electrónico donde se me pide que actualice a la última versión para volver a habilitar la actualización automática. Uso a versión 1.1.654.0. ¿Tengo que actualizarla?**  
Sí, tiene que actualizar a la versión 1.1.750.0 o posterior para volver a habilitar la actualización automática. [Descargar y actualizar a la versión más reciente](https://www.microsoft.com/download/details.aspx?id=47594).

**P: Recibí un correo electrónico donde se me pide que actualice a la última versión para volver a habilitar la actualización automática. Si usé PowerShell para habilitar la actualización automática, ¿también necesito instalar la última versión?**  
Sí, deberá actualizar a la versión 1.1.750.0 o posterior. Habilitar el servicio de actualización automática con PowerShell no mitiga el problema de actualización automática detectado en las versiones anteriores a la 1.1.750.0.

**P: ¿Quiero actualizar a una versión más reciente, pero no estoy seguro de quién instaló Azure AD Connect y no disponemos del nombre de usuario y la contraseña. ¿Necesito estos datos?**
No es necesario saber el nombre de usuario y la contraseña que se usó inicialmente para actualizar Azure AD Connect. Use cualquier cuenta de Azure AD que tenga el rol de administrador global.

**P: ¿Cómo puedo saber qué versión de Azure AD Connect estoy usando?**  
Para comprobar qué versión de Azure AD Connect está instalada en el servidor, vaya al Panel de control y busque la versión instalada de Microsoft Azure AD Connect seleccionando **Programas** > **Programas y características**, tal como se muestra a continuación:

![Versión de Azure AD Connect en el Panel de control](./media/reference-connect-faq/faq1.png)

**P: ¿Cómo actualizo a la versión más reciente de Azure AD Connect?**  
Para saber cómo actualizar a la versión más reciente, consulte [Azure AD Connect: actualización de una versión anterior a la versión más reciente](how-to-upgrade-previous-version.md) 

**P: Ya actualizamos a la última versión de Azure AD Connect el año pasado. ¿Es necesario actualizarla de nuevo?**  
El equipo de Azure AD Connect realiza actualizaciones frecuentes en el servicio. Para beneficiarse de las correcciones de errores y las actualizaciones de seguridad, así como de las nuevas características, es importante mantener al día el servidor con la versión más reciente. Si habilita la actualización automática, se actualiza automáticamente la versión del software. Para encontrar el historial de versiones de Azure AD Connect, consulte [Azure AD Connect: historial de versiones](reference-connect-version-history.md).

**P: ¿Cuánto tarda en realizarse la actualización y en qué afecta a mis usuarios?**  
El tiempo necesario para la actualización depende del tamaño de su inquilino. Para las organizaciones más grandes, podría ser mejor realizar la actualización por la noche o durante el fin de semana. Durante la actualización, no se realiza ninguna actividad de sincronización.

**P: Creo que actualicé a Azure AD Connect, pero en el portal de Office todavía se menciona DirSync. ¿Por qué ocurre esto?**  
El equipo de Office está trabajando para que las actualizaciones del portal de Office reflejen el nombre actual del producto. No refleja la herramienta de sincronización que se usa.

**P: Mi estado de la actualización automática indica "Suspendida". ¿Por qué está suspendida? ¿Debo habilitarla?**  
Se introdujo un error en una versión anterior que, en determinadas circunstancias, deja el estado de la actualización automática establecido en "Suspendida". Técnicamente, es posible habilitarla manualmente, pero requiere realizar varios pasos complejos. Lo mejor que se puede hacer es instalar la versión más reciente de Azure AD Connect.

**P: Mi empresa tiene requisitos estrictos de administración de cambios y me gustaría estar al tanto de los nuevos lanzamientos. ¿Puedo controlar cuándo se lanza la actualización automática?**  
No, no existe esta característica actualmente. La característica se está evaluando para futuras versiones.

**P: ¿Recibiré un correo electrónico si la actualización automática presenta algún error? ¿Cómo puedo saber si la actualización se realizó correctamente?**  
No se le notificará el resultado de la actualización. La característica se está evaluando para futuras versiones.

**P: ¿Se publica alguna escala de tiempo de la previsión de lanzamiento de actualizaciones automáticas?**  
La actualización automática es el primer paso del proceso de lanzamiento de una nueva versión. Cada vez que hay una nueva versión, se insertan automáticamente las actualizaciones. Las versiones más recientes de Azure AD Connect se anuncian con anticipación en la [hoja de ruta de Azure AD](../fundamentals/whats-new.md).

**P: ¿La actualización automática también actualiza Azure AD Connect Health?**  
Sí, la actualización automática también actualiza Azure AD Connect Health.

**P: ¿También se actualizan automáticamente los servidores de Azure AD Connect en modo de almacenamiento provisional?**  
Sí, puede actualizar automáticamente un servidor de Azure AD Connect que se encuentre en modo de almacenamiento provisional.

**P: Si la actualización automática genera algún error y mi servidor de Azure AD Connect no se inicia, ¿qué debo hacer?**  
En raras ocasiones, el servicio Azure AD Connect no se inicia después de realizar la actualización. En esos casos, reiniciar el servidor suele corregir el problema. Si el servicio Azure AD Connect sigue sin iniciarse, abra un vale de soporte. Para obtener más información, consulte [Create a service request to contact Office 365 support](/archive/blogs/praveenkumar/how-to-create-service-requests-to-contact-office-365-support) (Crear una solicitud de servicio para ponerse en contacto con el soporte técnico de Office 365). 

**P: No estoy seguro de los riesgos derivados de la actualización a una versión más reciente de Azure AD Connect. ¿Puede llamarme para ayudarme con la actualización?**  
Si necesita ayuda para actualizar a una versión más reciente de Azure AD Connect, abra un vale de soporte; en [Create a service request to contact Office 365 support](/archive/blogs/praveenkumar/how-to-create-service-requests-to-contact-office-365-support) (Crear una solicitud de servicio para ponerse en contacto con el soporte técnico de Office 365).

## <a name="operational-best-practice"></a>Prácticas recomendadas de uso    
A continuación se muestran algunos procedimientos recomendados que debe implementar al sincronizar entre Windows Server Active Directory y Azure Active Directory.

**Aplique Multi-Factor Authentication para todas las cuentas sincronizadas**. Azure Multi-Factor Authentication ayuda a proteger el acceso a los datos y aplicaciones, y al mismo tiempo mantiene la simplicidad para los usuarios. Proporciona más seguridad, ya que requiere una segunda forma de autenticación y ofrece autenticación segura a través de una gama de métodos de autenticación fáciles de usar. Puede que los usuarios se vean desafiados por la MFA en función de las decisiones de configuración que tome un administrador. Puede leer más sobre MFA aquí: https://www.microsoft.com/security/business/identity/mfa?rtc=1

[Siga las instrucciones del seguridad del servidor de Azure AD Connect](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material). El servidor de Azure AD Connect contiene datos de identidad críticos y debería tratarse como un componente de nivel 0, tal como se documenta en el **modelo de nivel administrativo de Active Directory**. Consulte también nuestras [instrucciones para proteger el servidor AADConnect](./how-to-connect-install-prerequisites.md#azure-ad-connect-server).

**Habilite PHS para la detección de credenciales filtradas**. La sincronización de hash de contraseña también habilita la [detección de credenciales filtradas](../identity-protection/concept-identity-protection-risks.md) para cuentas híbridas. Microsoft trabaja con investigadores de la Web oscura y organismos de aplicación de la ley para encontrar pares de nombre de usuario y contraseña disponibles públicamente. Si alguno de estos pares coincide con el de sus usuarios, la cuenta asociada pasa a ser de alto riesgo. 


## <a name="troubleshooting"></a>Solución de problemas
**P: ¿Cómo puedo obtener ayuda con Azure AD Connect?**

[Buscar en Microsoft Knowledge Base (KB)](https://www.microsoft.com/en-us/search/result.aspx?q=azure+active+directory+connect)

* Busque en KB soluciones técnicas a los problemas de break-fix más comunes sobre soporte técnico de Azure AD Connect.

[Página de preguntas y respuestas de Microsoft sobre Azure Active Directory](/answers/topics/azure-active-directory.html)

* Busque preguntas y respuestas técnicas o formule sus propias preguntas en [la comunidad de Azure AD](/answers/topics/azure-active-directory.html).

[Obtener soporte técnico para Azure AD](../fundamentals/active-directory-troubleshooting-support-howto.md)

**P: ¿Por qué veo los eventos 6311 y 6401 después de los errores de paso de sincronización?**

Los eventos 6311: **El servidor encontró un error inesperado al realizar una operación de devolución de llamada** y 6401: **El controlador del agente de administración encontró un error inesperado**, se registran siempre después de un error de paso de sincronización. Para resolver estos errores, debe limpiar los errores de los pasos de sincronización.  Para más información, consulte [Solución de errores durante la sincronización](tshoot-connect-sync-errors.md) y [Solución de problemas de sincronización de objetos con Azure AD Connect Sync](tshoot-connect-objectsync.md).