---
title: 'Tutorial: Configuración de LDAP seguro para Azure Active Directory Domain Services | Microsoft Docs'
description: En este tutorial aprenderá a configurar el protocolo ligero de acceso a directorios seguro (LDAPS) para un dominio administrado de Azure Active Directory Domain Services.
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: tutorial
ms.date: 07/06/2020
ms.author: iainfou
ms.openlocfilehash: 61e2d4607ebe1b688b2874220a170b2539a2226e
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87404181"
---
# <a name="tutorial-configure-secure-ldap-for-an-azure-active-directory-domain-services-managed-domain"></a>Tutorial: Configuración de LDAP seguro para un dominio administrado de Azure Active Directory Domain Services

Para comunicarse con el dominio administrado de Azure Active Directory Domain Services (Azure AD DS), se usa el protocolo ligero de acceso a directorios (LDAP). De forma predeterminada, el tráfico LDAP no está cifrado, lo que supone un problema de seguridad para muchos entornos.

Gracias a Azure AD DS, puede configurar el dominio administrado para que use el protocolo ligero de acceso a directorios seguro (LDAPS). Cuando se usa LDAP seguro, el tráfico se cifra. LDAP seguro también se conoce como LDAP sobre Capa de sockets seguros (SSL) o seguridad de la capa de transporte (TLS).

En este tutorial se muestra cómo configurar LDAPS para un dominio administrado de Azure AD DS.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un certificado digital para su uso con Azure AD DS
> * Habilitar LDAP seguro para Azure AD DS
> * Configurar LDAP seguro para su uso a través de la red pública de Internet
> * Enlazar y probar LDAP seguro para un dominio administrado

Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, [cree y configure un dominio administrado de Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* La herramienta *LDP.exe* instalada en el equipo.
    * Si es necesario, [instale las Herramientas de administración remota del servidor (RSAT)][rsat] para *Active Directory Domain Services y LDAP*.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

En este tutorial configurará LDAP seguro para el dominio administrado mediante Azure Portal. Para empezar a trabajar, primero inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="create-a-certificate-for-secure-ldap"></a>Creación de un certificado para LDAP seguro

Cuando se usa LDAP seguro, se utiliza un certificado digital para cifrar la comunicación. Este certificado digital se aplica al dominio administrado y permite que herramientas como *LDP.exe* utilicen una comunicación cifrada segura al consultar los datos. Hay dos maneras de crear un certificado para el acceso de LDAP seguro al dominio administrado:

* Un certificado de una entidad de certificación (CA) pública o de una entidad de certificación empresarial.
    * Si la organización obtiene los certificados de una entidad de certificación pública, obtenga de ella el certificado de LDAP seguro. Si usa una entidad de certificación empresarial, obténgalo de esta.
    * Una entidad de certificación pública solo funciona cuando se usa un nombre DNS personalizado con el dominio administrado. Si el nombre de dominio DNS del dominio administrado termina en *.onmicrosoft.com*, no se puede crear un certificado digital para proteger la conexión con este dominio predeterminado. Microsoft es el propietario del dominio *.onmicrosoft.com*, por lo que una entidad de certificación pública no emitirá un certificado. En este escenario, cree un certificado autofirmado y úselo para configurar LDAP seguro.
* Un certificado autofirmado que haya creado usted mismo.
    * Este enfoque es adecuado para realizar pruebas y es el que se muestra en este tutorial.

El certificado que solicite o cree debe cumplir los siguientes requisitos. Si se habilita LDAP seguro sin un certificado válido, se producirán problemas en el dominio administrado:

* **Emisor de confianza**: el certificado debe ser emitido por una autoridad de confianza para los equipos que se conectan al dominio administrado mediante LDAP seguro. Esta entidad puede ser una entidad de certificación pública o empresarial de confianza para estos equipos.
* **Duración** : el certificado debe ser válido al menos para los próximos 3-6 meses. El acceso LDAP seguro a su dominio administrado se interrumpe cuando expira el certificado.
* **Nombre del firmante**: el nombre del firmante del certificado debe ser el dominio administrado. Por ejemplo, si el dominio se denomina *aaddscontoso.com*, el nombre del firmante del certificado debe ser * *.aaddscontoso.com*.
    * El nombre DNS o el nombre alternativo del firmante del certificado debe ser un certificado comodín, para asegurarse de que LDAP seguro funcione correctamente con Azure AD Domain Services. Los controladores de dominio usan nombres aleatorios y se pueden quitar o agregar para asegurarse de que el servicio sigue estando disponible.
* **Uso de claves**: el certificado debe configurarse para las *firmas digitales* y el *cifrado de claves*.
* **Propósito del certificado**: el certificado debe ser válido para la autenticación del servidor TLS.

Hay varias herramientas disponibles para crear un certificado autofirmado, como OpenSSL, Keytool, MakeCert, el cmdlet [New-SelfSignedCertificate][New-SelfSignedCertificate], etc.

En este tutorial crearemos un certificado autofirmado para LDAP seguro mediante el cmdlet [New-SelfSignedCertificate][New-SelfSignedCertificate].

Abra una ventana de PowerShell como **Administrador** y ejecute los siguientes comandos. Reemplace la variable *$dnsName* por el nombre DNS que usa su propio dominio administrado, como *aaddscontoso.com*:

```powershell
# Define your own DNS name used by your managed domain
$dnsName="aaddscontoso.com"

# Get the current date to set a one-year expiration
$lifetime=Get-Date

# Create a self-signed certificate for use with Azure AD DS
New-SelfSignedCertificate -Subject *.$dnsName `
  -NotAfter $lifetime.AddDays(365) -KeyUsage DigitalSignature, KeyEncipherment `
  -Type SSLServerAuthentication -DnsName *.$dnsName, $dnsName
```

La siguiente salida de ejemplo muestra que el certificado se generó correctamente y se almacenó en el almacén de certificados local (*LocalMachine\MY*):

```output
PS C:\WINDOWS\system32> New-SelfSignedCertificate -Subject *.$dnsName `
>>   -NotAfter $lifetime.AddDays(365) -KeyUsage DigitalSignature, KeyEncipherment `
>>   -Type SSLServerAuthentication -DnsName *.$dnsName, $dnsName.com

   PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\MY

Thumbprint                                Subject
----------                                -------
959BD1531A1E674EB09E13BD8534B2C76A45B3E6  CN=aaddscontoso.com
```

## <a name="understand-and-export-required-certificates"></a>Información sobre los certificados necesarios y exportación de estos

Para usar LDAP seguro, el tráfico de red se cifra mediante la infraestructura de clave pública (PKI).

* En el dominio administrado se aplica una **clave privada**.
    * Esta clave privada se usa para *descifrar* el tráfico LDAP seguro. La clave privada solo debe aplicarse al dominio administrado y no debe distribuirse por los equipos cliente.
    * Un certificado que incluye la clave privada utiliza el formato de archivo *.PFX*.
    * El algoritmo de cifrado del certificado debe ser *TripleDES-SHA1*.
* En los equipos cliente se aplica una **clave pública**.
    * Esta clave pública se utiliza para *cifrar* el tráfico LDAP seguro. La clave pública se puede distribuir a los equipos cliente.
    * Los certificados sin la clave privada usan el formato de archivo *.CER*.

Estas dos claves, *privada* y *pública*, garantizan que solo los equipos adecuados pueden comunicarse correctamente entre sí. Si se usa una entidad de certificación pública o empresarial, se emite un certificado que incluye la clave privada y se puede aplicar a un dominio administrado. Los equipos cliente ya deben conocer la clave pública y confiar en ella.

En este tutorial ha creado un certificado autofirmado con la clave privada, por lo que debe exportar los componentes públicos y privados correspondientes.

### <a name="export-a-certificate-for-azure-ad-ds"></a>Exportación de un certificado para Azure AD DS

Antes de poder usar el certificado digital creado en el paso anterior con el dominio administrado, exporte el certificado a un archivo de certificado *.PFX*, que incluye la clave privada.

1. Para abrir el cuadro de diálogo *Ejecutar*, presione las teclas **Windows** + **R**.
1. Escriba **mmc** en el cuadro de diálogo *Ejecutar* y, después, seleccione **Aceptar** para abrir Microsoft Management Console (MMC).
1. En la solicitud de **Control de cuentas de usuario**, seleccione **Sí** para iniciar MMC como administrador.
1. En el menú **Archivo**, seleccione **Agregar o quitar complemento**.
1. En el asistente para el **complemento Certificados**, elija **Cuenta de equipo** y seleccione **Siguiente**.
1. En la página **Seleccionar equipo**, elija **Equipo local: (el equipo en el que se está ejecutando esta consola)** y seleccione **Finalizar**.
1. En el cuadro de diálogo **Agregar o quitar complementos**, seleccione **Aceptar** para agregar el complemento de certificados a MMC.
1. En la ventana de MMC, expanda **Raíz de consola**. Seleccione **Certificados (equipo local)** y, después, expanda el nodo **Personal**, seguido del nodo **Certificados**.

    ![Apertura del almacén de certificados personales en Microsoft Management Console](./media/tutorial-configure-ldaps/open-personal-store.png)

1. Se muestra el certificado autofirmado creado en el paso anterior, como *aaddscontoso.com*. Seleccione con el botón derecho este certificado y, a continuación, elija **Todas las tareas > Exportar**.

    ![Exportación de certificado en Microsoft Management Console](./media/tutorial-configure-ldaps/export-cert.png)

1. En el **Asistente para exportación de certificados**, seleccione **Siguiente**.
1. Se debe exportar la clave privada del certificado. Si el certificado exportado no incluye la clave privada, se produce un error en la acción para habilitar LDAP seguro para el dominio administrado.

    En la página **Exportar la clave privada**, elija **Exportar la clave privada** y, después, seleccione **Siguiente**.
1. Los dominios administrados solo admiten el formato de archivo de certificado *.PFX*, que incluye la clave privada. No exporte el certificado con el formato de archivo de certificado *.CER*, que no la incluye.

    En la página **Formato de archivo de exportación**, seleccione **Intercambio de información personal: PKCS #12 (.PFX)** como formato de archivo para el certificado exportado. Active la casilla *Incluir todos los certificados en la ruta de certificación (si es posible)* :

    ![Selección de la opción para exportar el certificado en el archivo PKCS 12 (.PFX)](./media/tutorial-configure-ldaps/export-cert-to-pfx.png)

1. Como este certificado se utiliza para descifrar datos, debe controlar cuidadosamente el acceso. Se puede usar una contraseña para proteger su uso. Sin la contraseña correcta, el certificado no se puede aplicar a un servicio.

    En la página **Seguridad**, elija la opción de **Contraseña** para proteger el archivo de certificado *.PFX*. El algoritmo de cifrado debe ser *TripleDES-SHA1*. Escriba y confirme una contraseña y, después, seleccione **Siguiente**. Esta contraseña se usará en la sección siguiente para habilitar LDAP seguro para el dominio administrado.
1. En la página **Archivo que se va a exportar**, especifique el nombre de archivo y la ubicación donde desea exportar el certificado como, por ejemplo, *C:\Users\nombreDeCuenta\azure-ad-ds.pfx*. Anote la contraseña y la ubicación del archivo *.PFX*, ya que necesitará esta información en los pasos siguientes.
1. En la página de revisión, seleccione **Finalizar** para exportar el certificado a un archivo de certificado *.PFX*. Se muestra un cuadro de diálogo de confirmación cuando el certificado se ha exportado correctamente.
1. Mantenga MMC abierto para usarlo en la sección siguiente.

### <a name="export-a-certificate-for-client-computers"></a>Exportación de un certificado para equipos cliente

Los equipos cliente deben confiar en el emisor del certificado de LDAP seguro para poder conectarse correctamente al dominio administrado mediante LDAPS. Los equipos cliente necesitan un certificado para cifrar correctamente los datos que descifra Azure AD DS. Si usa una entidad de certificación pública, el equipo debe confiar automáticamente en estos emisores de certificados y tener un certificado correspondiente.

En este tutorial se usa un certificado autofirmado y, en el paso anterior, se generó un certificado que incluye la clave privada. Ahora exportaremos y después instalaremos el certificado autofirmado en el almacén de certificados de confianza del equipo cliente:

1. Vuelva a MMC, al almacén de *Certificados (equipo local) > Personal > Certificados*. Se muestra el certificado autofirmado creado en un paso anterior, como *aaddscontoso.com*. Seleccione con el botón derecho este certificado y, a continuación, elija **Todas las tareas > Exportar**.
1. En el **Asistente para exportación de certificados**, seleccione **Siguiente**.
1. Dado que no necesita la clave privada en los equipos cliente, en la página **Exportar la clave privada** elija **No exportar la clave privada** y, después, seleccione **Siguiente**.
1. En la página **Formato de archivo de exportación**, seleccione **X.509 codificado base 64 (.CER)** como formato de archivo para el certificado exportado:

    ![Selección de la opción para exportar el certificado en el formato de archivo X.509 codificado base 64 (.CER)](./media/tutorial-configure-ldaps/export-cert-to-cer-file.png)

1. En la página **Archivo que se va a exportar**, especifique el nombre de archivo y la ubicación donde desea exportar el certificado como, por ejemplo, *C:\Users\nombreDeCuenta\azure-ad-ds-client.cer*.
1. En la página de revisión, seleccione **Finalizar** para exportar el certificado a un archivo de certificado *.CER*. Se muestra un cuadro de diálogo de confirmación cuando el certificado se ha exportado correctamente.

Ahora el archivo de certificado *.CER* se puede distribuir a los equipos cliente que necesitan confiar en la conexión LDAP segura con el dominio administrado. Instalemos el certificado en el equipo local.

1. Abra el Explorador de archivos y vaya a la ubicación donde guardó el archivo de certificado *.CER* como, por ejemplo, *C:\Users\nombreDeCuenta\azure-ad-ds-client.cer*.
1. Seleccione con el botón derecho el archivo de certificado *.CER* y, después, elija **Instalar certificado**.
1. En el **Asistente para importar certificados**, elija la opción para almacenar el certificado en el *Equipo local* y, después, seleccione **Siguiente**:

    ![Selección de la opción para importar el certificado en el almacén del equipo local](./media/tutorial-configure-ldaps/import-cer-file.png)

1. Cuando se le solicite, elija **Sí** para permitir que el equipo realice cambios.
1. Elija **Seleccionar automáticamente el almacén de certificados según el tipo de certificado** y, después, seleccione **Siguiente**.
1. En la página de revisión, seleccione **Finalizar** para importar el archivo de certificado *.CER*. Se muestra un cuadro de diálogo de confirmación cuando el certificado se ha importado correctamente.

## <a name="enable-secure-ldap-for-azure-ad-ds"></a>Habilitar LDAP seguro para Azure AD DS

Ahora que ya contamos con un certificado digital creado y exportado que incluye la clave privada y con el equipo cliente configurado para confiar en la conexión, es necesario habilitar LDAP seguro en el dominio administrado. Para habilitar LDAP seguro en un dominio administrado, lleve a cabo los siguientes pasos de configuración:

1. En [Azure Portal](https://portal.azure.com), escriba *domain services* en el cuadro **Buscar recursos**. En el resultado de la búsqueda, seleccione **Azure AD Domain Services**.
1. Elija el dominio administrado como, por ejemplo, *aaddscontoso.com*.
1. En el lado izquierdo de la ventana de Azure AD DS, elija **LDAP seguro**.
1. De forma predeterminada, se deshabilita el acceso de LDAP seguro a su dominio administrado. Cambie **LDAP seguro** a **Habilitar**.
1. De forma predeterminada, el acceso de LDAP seguro al dominio administrado a través de Internet está deshabilitado. Cuando se habilita el acceso LDAP seguro público, el dominio es susceptible de recibir ataques por fuerza bruta para obtener la contraseña a través de Internet. En el paso siguiente, se configura un grupo de seguridad de red para bloquear el acceso solo a los intervalos de direcciones IP de origen necesarios.

    Cambie **Permitir el acceso mediante LDAP seguro a través de Internet** a **Habilitar**.

1. Seleccione el icono de carpeta situado junto a **Archivo .PFX con certificado de LDAP seguro**. Navegue a la ruta de acceso del archivo *.PFX* y seleccione el certificado creado en un paso anterior que incluye la clave privada.

    > [!IMPORTANT]
    > Como se indicó en la sección anterior sobre los requisitos de certificados, no se puede usar un certificado de una entidad de certificación pública con el dominio *.onmicrosoft.com* predeterminado. Microsoft es el propietario del dominio *.onmicrosoft.com*, por lo que una entidad de certificación pública no emitirá un certificado.
    >
    > Asegúrese de que el certificado tiene el formato adecuado. Si no es así, la plataforma Azure generará errores de validación de certificado al habilitar LDAP seguro.

1. Escriba la **Contraseña para descifrar el archivo .PFX** establecida en un paso anterior, cuando se exportó el certificado a un archivo *.PFX*.
1. Seleccione **Guardar** para habilitar LDAP seguro.

    ![Habilitación de LDAP seguro para un dominio administrado en Azure Portal](./media/tutorial-configure-ldaps/enable-ldaps.png)

Se muestra una notificación que indica que se está configurando LDAP seguro para el dominio administrado. Hasta que se complete la operación, no podrá modificar ninguna otra configuración para este dominio.

La habilitación de LDAP seguro para el dominio administrado tarda unos minutos. Si el certificado de LDAP seguro que proporciona no cumple los criterios necesarios, se produce un error en la acción que habilita LDAP seguro para el dominio administrado.

Algunos motivos habituales del error que el nombre de dominio sea incorrecto, que el algoritmo de cifrado del certificado no sea *TripleDES-SHA1* o que el certificado expire pronto o ya haya expirado. Puede volver a crear el certificado con parámetros válidos y, a continuación, habilitar LDAP seguro con este certificado actualizado.

## <a name="lock-down-secure-ldap-access-over-the-internet"></a>Bloqueo del acceso LDAP seguro a través de Internet

Cuando se habilita el acceso LDAP seguro a través de Internet al dominio administrado, se crea una amenaza de seguridad. El dominio administrado es accesible desde Internet en el puerto TCP 636. Se recomienda restringir el acceso al dominio administrado a aquellas direcciones IP conocidas específicas del entorno. Se puede usar una regla de grupo de seguridad de red de Azure para limitar el acceso a LDAP seguro.

Crearemos una regla para permitir el acceso LDAP seguro de entrada a través del puerto TCP 636 desde un conjunto especificado de direcciones IP. En el resto de tráfico entrante de Internet se aplica una regla *DenyAll* predeterminada con una prioridad más baja, por lo que solo las direcciones especificadas pueden llegar al dominio administrado mediante LDAP seguro.

1. En Azure Portal, seleccione *Grupos de recursos* en la navegación lateral izquierda.
1. Elija el grupo de recursos, por ejemplo *miGrupoDeRecursos*, y luego seleccione el grupo de seguridad de red, por ejemplo *aaads-nsg*.
1. Se muestra la lista de reglas de seguridad de entrada y salida existentes. En el lado izquierdo de las ventanas del grupo de seguridad de red, elija **Configuración > Reglas de seguridad de entrada**.
1. Seleccione **Agregar** y, a continuación, cree una regla para permitir el puerto *TCP* *636*. Para mejorar la seguridad, elija el origen como *Direcciones IP* y, después, especifique su propia dirección IP o el intervalo de direcciones IP válidas para la organización.

    | Configuración                           | Value        |
    |-----------------------------------|--------------|
    | Source                            | Direcciones IP |
    | Intervalos de direcciones IP de origen y CIDR | Una dirección o un intervalo de direcciones IP válidas para el entorno |
    | Source port ranges                | *            |
    | Destination                       | Any          |
    | Intervalos de puertos de destino           | 636          |
    | Protocolo                          | TCP          |
    | Acción                            | Allow        |
    | Priority                          | 401          |
    | Nombre                              | AllowLDAPS   |

1. Cuando esté preparado, seleccione **Agregar** para guardar y aplicar la regla.

    ![Creación de una regla de grupo de seguridad de red para proteger el acceso a LDAPS a través de Internet](./media/tutorial-configure-ldaps/create-inbound-nsg-rule-for-ldaps.png)

## <a name="configure-dns-zone-for-external-access"></a>Configuración de la zona DNS para el acceso externo

Ahora que ya ha habilitado el acceso LDAP seguro a través de Internet, debe actualizar la zona DNS para que los equipos cliente puedan encontrar este dominio administrado. En la pestaña **Propiedades** del dominio administrado se muestra el valor de *Dirección IP externa de LDAP seguro*:

![Consulta de la dirección IP externa de LDAP seguro para el dominio administrado en Azure Portal](./media/tutorial-configure-ldaps/ldaps-external-ip-address.png)

Configure el proveedor DNS externo para crear un registro de host, por ejemplo *ldaps*, para resolver en esta dirección IP externa. Para hacer antes una prueba local en el equipo, puede crear una entrada en el archivo de hosts de Windows. Para editar correctamente el archivo de hosts en la máquina local, abra el *Bloc de notas* como administrador y, después, abra el archivo *C:\Windows\System32\drivers\etc\hosts*.

La siguiente entrada de DNS de ejemplo, ya sea con el proveedor DNS externo o en el archivo de hosts local, resuelve el tráfico de *ldaps.aaddscontoso.com* en la dirección IP externa de *168.62.205.103*:

```
168.62.205.103    ldaps.aaddscontoso.com
```

## <a name="test-queries-to-the-managed-domain"></a>Prueba de consultas en el dominio administrado

Para conectar y enlazar con el dominio administrado y buscar en LDAP, puede usar la herramienta *LDP.exe*. Esta herramienta se incluye en el paquete de Herramientas de administración remota del servidor (RSAT). Para más información, consulte la sección sobre [instalación de las Herramientas de administración remota del servidor][rsat].

1. Abra *LDP.exe* y conéctese al dominio administrado. Seleccione **Conexión** y elija **Conectar**.
1. Escriba el nombre de dominio DNS de LDAP seguro del dominio administrado creado en el paso anterior, por ejemplo *ldaps.aaddscontoso.com*. Para usar LDAP seguro, establezca **Puerto** en *636* y, después, active la casilla para **SSL**.
1. Seleccione **Aceptar** para conectarse al dominio administrado.

Después, enlace al dominio administrado. Los usuarios (y las cuentas de servicio) no pueden realizar enlaces LDAP simples si se ha desactivado la sincronización de hash de contraseñas NTLM en el dominio administrado. Para más información acerca de cómo deshabilitar la sincronización de hash de contraseñas NTLM, consulte el artículo sobre [protección de un dominio administrado][secure-domain].

1. Seleccione la opción de menú **Conexión** y, después, elija **Enlazar**.
1. Proporcione las credenciales de una cuenta de usuario que pertenezca al grupo *Administradores de DC de AAD* como, por ejemplo, *contosoadmin*. Escriba la contraseña de la cuenta de usuario y, a continuación, escriba el dominio, como *aaddscontoso.com*.
1. En **Tipo de enlace**, elija la opción *Enlazar con credenciales*.
1. Seleccione **Aceptar** para enlazar con el dominio administrado.

Para ver los objetos almacenados en el dominio administrado:

1. Seleccione la opción de menú **Ver** y, a continuación, elija **Árbol**.
1. Deje en blanco el campo *DN base* y, después, seleccione **Aceptar**.
1. Elija un contenedor, como *AADDC Users*, seleccione con el botón derecho el contenedor y elija **Buscar**.
1. Deje los campos rellenados previamente con los valores establecidos y, a continuación, seleccione **Ejecutar**. Los resultados de la consulta aparecen en la ventana de la derecha, tal como se muestra en la siguiente salida de ejemplo:

    ![Búsqueda de objetos en el dominio administrado con LDP.exe](./media/tutorial-configure-ldaps/ldp-query.png)

Para consultar directamente un contenedor específico, en el menú **Ver > Árbol**, puede especificar un **DN base** como *OU=AADDC Users,DC=AADDSCONTOSO,DC=COM* o bien *OU=AADDC Computers,DC=AADDSCONTOSO,DC=COM*. Para más información sobre el formato y la creación de consultas, lea sobre los [conceptos básicos de las consultas LDAP][ldap-query-basics].

## <a name="clean-up-resources"></a>Limpieza de recursos

Si en este tutorial ha agregado una entrada DNS en el archivo de hosts local del equipo para probar la conectividad, quite esta entrada y agregue un registro formal en la zona DNS. Para quitar la entrada del archivo de hosts local, realice los pasos siguientes:

1. En el equipo local, abra el *Bloc de notas* como administrador.
1. Vaya al archivo *C:\Windows\System32\drivers\etc\hosts* y ábralo.
1. Elimine la línea del registro que ha agregado, como `168.62.205.103    ldaps.aaddscontoso.com`.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear un certificado digital para su uso con Azure AD DS
> * Habilitar LDAP seguro para Azure AD DS
> * Configurar LDAP seguro para su uso a través de la red pública de Internet
> * Enlazar y probar LDAP seguro para un dominio administrado

> [!div class="nextstepaction"]
> [Configuración de la sincronización de hash de contraseñas para un entorno de Azure AD híbrido](tutorial-configure-password-hash-sync.md)

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[secure-domain]: secure-your-domain.md

<!-- EXTERNAL LINKS -->
[rsat]: /windows-server/remote/remote-server-administration-tools
[ldap-query-basics]: /windows/desktop/ad/creating-a-query-filter
[New-SelfSignedCertificate]: /powershell/module/pkiclient/new-selfsignedcertificate
