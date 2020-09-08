---
title: Problemas de autenticación en Azure HDInsight
description: Problemas de autenticación en Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/24/2020
ms.openlocfilehash: 6473b6b69ba45138c30e4f2c8a7f1ba1a7f2ae7e
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816854"
---
# <a name="authentication-issues-in-azure-hdinsight"></a>Problemas de autenticación en Azure HDInsight

En este artículo se describen los pasos de solución de problemas y las posibles soluciones para los problemas que se producen al usar clústeres de Azure HDInsight.

En clústeres seguros respaldados por Azure Data Lake (Gen1 o Gen2), cuando los usuarios del dominio inician sesión en los servicios de clúster a través de la puerta de enlace de HDI (como el inicio de sesión en el portal de Apache Ambari), dicha puerta de enlace intentará obtener un token de OAuth de Azure Active Directory (Azure AD) en primer lugar y, a continuación, obtendrá un vale de Kerberos de Azure AD DS. La autenticación puede dar errores en cualquiera de estas fases. Este artículo está pensado para depurar de algunos de estos problemas.

Cuando se produzca un error en la autenticación, se le pedirán las credenciales. Si cancela este cuadro de diálogo, se imprimirá el mensaje de error. Estos son algunos de los escenarios comunes:

## <a name="invalid_grant-or-unauthorized_client-50126"></a>invalid_grant o unauthorized_client, 50126

### <a name="issue"></a>Problema

Se produce un error de inicio de sesión para los usuarios federados con el código de error 50126 (el inicio de sesión se realiza correctamente para los usuarios en la nube). El mensaje de error es similar al siguiente:

```
Reason: Bad Request, Detailed Response: {"error":"invalid_grant","error_description":"AADSTS70002: Error validating credentials. AADSTS50126: Invalid username or password\r\nTrace ID: 09cc9b95-4354-46b7-91f1-efd92665ae00\r\n Correlation ID: 4209bedf-f195-4486-b486-95a15b70fbe4\r\nTimestamp: 2019-01-28 17:49:58Z","error_codes":[70002,50126], "timestamp":"2019-01-28 17:49:58Z","trace_id":"09cc9b95-4354-46b7-91f1-efd92665ae00","correlation_id":"4209bedf-f195-4486-b486-95a15b70fbe4"}
```

### <a name="cause"></a>Causa

El código de error 50126 de Azure AD significa que el inquilino no ha establecido la directiva `AllowCloudPasswordValidation`.

### <a name="resolution"></a>Solución

El administrador de la compañía del inquilino de Azure AD debe habilitar Azure AD para usar los hash de contraseña para los usuarios respaldados por ADFS.  Aplique `AllowCloudPasswordValidationPolicy` como se muestra en el artículo [Uso de Enterprise Security Package en HDInsight](../domain-joined/apache-domain-joined-architecture.md).

---

## <a name="invalid_grant-or-unauthorized_client-50034"></a>invalid_grant o unauthorized_client, 50034

### <a name="issue"></a>Problema

El inicio de sesión produce un error con el código 50034. El mensaje de error es similar al siguiente:

```
{"error":"invalid_grant","error_description":"AADSTS50034: The user account Microsoft.AzureAD.Telemetry.Diagnostics.PII does not exist in the 0c349e3f-1ac3-4610-8599-9db831cbaf62 directory. To sign into this application, the account must be added to the directory.\r\nTrace ID: bbb819b2-4c6f-4745-854d-0b72006d6800\r\nCorrelation ID: b009c737-ee52-43b2-83fd-706061a72b41\r\nTimestamp: 2019-04-29 15:52:16Z", "error_codes":[50034],"timestamp":"2019-04-29 15:52:16Z","trace_id":"bbb819b2-4c6f-4745-854d-0b72006d6800", "correlation_id":"b009c737-ee52-43b2-83fd-706061a72b41"}
```

### <a name="cause"></a>Causa

El nombre de usuario es incorrecto (no existe). El usuario no está utilizando el mismo nombre de usuario que se usa en Azure Portal.

### <a name="resolution"></a>Solución

Use el mismo nombre de usuario que funciona en ese portal.

---

## <a name="invalid_grant-or-unauthorized_client-50053"></a>invalid_grant o unauthorized_client, 50053

### <a name="issue"></a>Problema

La cuenta de usuario está bloqueada; código de error 50053. El mensaje de error es similar al siguiente:

```
{"error":"unauthorized_client","error_description":"AADSTS50053: You've tried to sign in too many times with an incorrect user ID or password.\r\nTrace ID: 844ac5d8-8160-4dee-90ce-6d8c9443d400\r\nCorrelation ID: 23fe8867-0e8f-4e56-8764-0cdc7c61c325\r\nTimestamp: 2019-06-06 09:47:23Z","error_codes":[50053],"timestamp":"2019-06-06 09:47:23Z","trace_id":"844ac5d8-8160-4dee-90ce-6d8c9443d400","correlation_id":"23fe8867-0e8f-4e56-8764-0cdc7c61c325"}
```

### <a name="cause"></a>Causa

Demasiados intentos de inicio de sesión con una contraseña incorrecta.

### <a name="resolution"></a>Solución

Espere treinta minutos aproximadamente o detenga las aplicaciones que puedan estar intentando autenticarse.

---

## <a name="invalid_grant-or-unauthorized_client-50053-2"></a>invalid_grant o unauthorized_client, 50053 (#2)

### <a name="issue"></a>Problema

Contraseña expirada, código de error 50053. El mensaje de error es similar al siguiente:

```
{"error":"user_password_expired","error_description":"AADSTS50055: Password is expired.\r\nTrace ID: 241a7a47-e59f-42d8-9263-fbb7c1d51e00\r\nCorrelation ID: c7fe4a42-67e4-4acd-9fb6-f4fb6db76d6a\r\nTimestamp: 2019-06-06 17:29:37Z","error_codes":[50055],"timestamp":"2019-06-06 17:29:37Z","trace_id":"241a7a47-e59f-42d8-9263-fbb7c1d51e00","correlation_id":"c7fe4a42-67e4-4acd-9fb6-f4fb6db76d6a","suberror":"user_password_expired","password_change_url":"https://portal.microsoftonline.com/ChangePassword.aspx"}
```

### <a name="cause"></a>Causa

La contraseña ha expirado.

### <a name="resolution"></a>Solución

Cambie la contraseña en Azure Portal (en el sistema local) y espere treinta minutos para que la sincronización se ponga al día.

---

## <a name="interaction_required"></a>interaction_required

### <a name="issue"></a>Problema

Recibe el mensaje de error `interaction_required`.

### <a name="cause"></a>Causa

La directiva de acceso condicional o MFA se está aplicando al usuario. Como todavía no se admite la autenticación interactiva, el usuario o el clúster deben estar exentos del acceso condicional o MFA. Si decide excluir el clúster (directiva de exención basada en dirección IP), asegúrese de que `ServiceEndpoints` de AD está habilitado para esa red virtual.

### <a name="resolution"></a>Solución

Use la directiva de acceso condicional y excluya los clústeres de HDInisght de MFA, tal como se muestra en [Configurar un clúster de HDInsight con Enterprise Security Package mediante Azure Active Directory Domain Services](./apache-domain-joined-configure-using-azure-adds.md).

---

## <a name="sign-in-denied"></a>Inicio de sesión denegado

### <a name="issue"></a>Problema

El inicio de sesión se ha denegado.

### <a name="cause"></a>Causa

Para llegar a esta fase, la autenticación de OAuth no es un problema, pero sí lo es la autenticación Kerberos. Si este clúster está respaldado por ADLS, el inicio de sesión de OAuth se ha realizado correctamente antes de que se intente la autenticación Kerberos. En los clústeres de WASB, no se intenta el inicio de sesión de OAuth. Podría haber muchos motivos para el error de Kerberos, como que los hashes de contraseña no estén sincronizados, la cuenta de usuario se bloqueó en Azure AD DS, etc. Los hashes de contraseña solo se sincronizan cuando el usuario cambia la contraseña. Al crear la instancia de Azure AD DS, se iniciará la sincronización de las contraseñas que se cambian después de la creación. No sincronizará retroactivamente las contraseñas que se establecieron antes de su inicio.

### <a name="resolution"></a>Solución

Si cree que las contraseñas pueden no estar sincronizadas, pruebe a cambiar la contraseña y espere unos minutos a que se sincronicen.

Deberá intentar autenticarse (kinit) utilizando las mismas credenciales de usuario, desde una máquina que esté unida al dominio. Pruebe SSH en el nodo principal o perimetral con un usuario local y luego ejecute kinit.

---

## <a name="kinit-fails"></a>Error de kinit

### <a name="issue"></a>Problema

Error de kinit.

### <a name="cause"></a>Causa

Varía.

### <a name="resolution"></a>Solución

Para que kinit se realice correctamente, debe conocer su `sAMAccountName` (este es el nombre corto de la cuenta sin el dominio Kerberos). `sAMAccountName` es normalmente el prefijo de la cuenta (como bob en `bob@contoso.com`). Para algunos usuarios, podría ser diferente. Necesitará la capacidad de examinar el directorio o buscar en él para aprender su `sAMAccountName`.

Formas de encontrar `sAMAccountName`:

* Si puede iniciar sesión en Ambari mediante el administrador de Ambari local, consulte la lista de usuarios.

* Si tiene una [máquina Windows unida a un dominio](../../active-directory-domain-services/manage-domain.md), puede usar las herramientas estándar de AD de Windows para examinar. Esto requiere una cuenta profesional en el dominio.

* En el nodo principal, puede usar los comandos de SAMBA para buscar. Esto requiere una sesión Kerberos válida (kinit correcto). net ads search "(userPrincipalName=bob*)"

    Los resultados de la búsqueda y el examen deben mostrarle el atributo `sAMAccountName`. Además, podría examinar otros atributos como `pwdLastSet`, `badPasswordTime`, `userPrincipalName` etc. para ver si esas propiedades coinciden con lo que espera.

---

## <a name="kinit-fails-with-preauthentication-failure"></a>kinit produce un error de autenticación previa

### <a name="issue"></a>Problema

Kinit genera el error `Preauthentication`.

### <a name="cause"></a>Causa

Nombre de usuario o contraseña incorrectos.

### <a name="resolution"></a>Solución

Compruebe su nombre de usuario y contraseña. Compruebe también si hay otras propiedades descritas anteriormente. Para habilitar la depuración detallada, ejecute `export KRB5_TRACE=/tmp/krb.log` desde la sesión antes de intentar kinit.

---

## <a name="job--hdfs-command-fails-due-to-tokennotfoundexception"></a>Error del trabajo o comando HDFS debido a TokenNotFoundException

### <a name="issue"></a>Problema

El trabajo o comando HDFS produce un error debido a `TokenNotFoundException`.

### <a name="cause"></a>Causa

No se encontró el token de acceso de OAuth necesario para que el trabajo o el comando se ejecutara correctamente. El controlador ADLS/ABFS intentará recuperar el token de acceso de OAuth desde el servicio de credenciales antes de realizar las solicitudes de almacenamiento. Este token se registra cuando inicia sesión en el portal de Ambari con el mismo usuario.

### <a name="resolution"></a>Solución

Asegúrese de que ha iniciado sesión correctamente en el portal de Ambari una vez a través del nombre de usuario cuya identidad se usa para ejecutar el trabajo.

---

## <a name="error-fetching-access-token"></a>Error al recuperar el token de acceso

### <a name="issue"></a>Problema

El usuario recibe el mensaje de error `Error fetching access token`.

### <a name="cause"></a>Causa

Este error se produce de manera intermitente cuando los usuarios intentan acceder a ADLS Gen2 mediante listas de control de acceso y el token de Kerberos ha expirado.

### <a name="resolution"></a>Solución

* Para Azure Data Lake Storage Gen1, limpie la memoria caché del explorador y vuelva a iniciar sesión en Ambari.

* Para Azure Data Lake Storage Gen2, ejecute `/usr/lib/hdinsight-common/scripts/RegisterKerbTicketAndOAuth.sh <upn>` para el usuario con el que se está intentando iniciar sesión.

---

## <a name="next-steps"></a>Pasos siguientes

Si su problema no aparece o es incapaz de resolverlo, visite uno de nuestros canales para obtener ayuda adicional:

* Obtenga respuestas de expertos de Azure mediante el [soporte técnico de la comunidad de Azure](https://azure.microsoft.com/support/community/).

* Póngase en contacto con [@AzureSupport](https://twitter.com/azuresupport), la cuenta oficial de Microsoft Azure para mejorar la experiencia del cliente. Esta cuenta pone en contacto a la comunidad de Azure con los recursos adecuados: respuestas, soporte técnico y expertos.

* Si necesita más ayuda, puede enviar una solicitud de soporte técnico desde [Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Seleccione **Soporte técnico** en la barra de menús o abra la central **Ayuda + soporte técnico**. Para obtener información más detallada, revise [Creación de una solicitud de soporte técnico de Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request). La suscripción a Microsoft Azure incluye acceso al soporte técnico para facturación y administración de suscripciones. El soporte técnico se proporciona a través de uno de los [planes de soporte técnico de Azure](https://azure.microsoft.com/support/plans/).
