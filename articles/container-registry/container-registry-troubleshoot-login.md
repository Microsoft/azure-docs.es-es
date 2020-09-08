---
title: Solución de problemas de inicio de sesión en el registro
description: Síntomas, causas y resolución de problemas comunes al iniciar sesión en un registro de contenedor de Azure
ms.topic: article
ms.date: 08/11/2020
ms.openlocfilehash: 8fbb96be8223001ac52db47788c31609e9b86e35
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88226981"
---
# <a name="troubleshoot-registry-login"></a>Solución de problemas de inicio de sesión del registro

Este artículo le ayuda a solucionar los problemas que pueden surgir al iniciar sesión en un registro de contenedor de Azure. 

## <a name="symptoms"></a>Síntomas

Puede encontrarse con uno o varios de los siguientes:

* No se puede iniciar sesión en el registro con `docker login`, `az acr login`, o ambos
* No se puede iniciar sesión en el registro y se recibe el error `unauthorized: authentication required` o `unauthorized: Application not registered with AAD`
* No se puede iniciar sesión en el registro y se recibe el error de la CLI de Azure `Could not connect to the registry login server`
* No es posible insertar o extraer imágenes y se recibe el error de Docker `unauthorized: authentication required`
* No se puede acceder al registro desde Azure Kubernetes Service, Azure DevOps u otro servicio de Azure
* No se puede acceder al registro y recibe un error `Error response from daemon: login attempt failed with status: 403 Forbidden`. Consulte [Solución de problemas de red con el registro](container-registry-troubleshoot-access.md)
* No se puede acceder a la configuración del registro ni verla en Azure Portal; tampoco se puede administrar el registro mediante la CLI de Azure

## <a name="causes"></a>Causas

* Docker no está configurado correctamente en el entorno: [solución](#check-docker-configuration)
* El registro no existe o el nombre es incorrecto: [solución](#specify-correct-registry-name)
* Las credenciales del registro no son válidas: [solución](#confirm-credentials-to-access-registry)
* Las credenciales no están autorizadas para las operaciones de inserción, extracción o de Azure Resource Manager: [solución](#confirm-credentials-are-authorized-to-access-registry)
* Las credenciales han expirado: [solución](#check-that-credentials-arent-expired)

## <a name="further-diagnosis"></a>Diagnóstico detallado 

Ejecute el comando [az acr check-helth](/cli/azure/acr#az-acr-check-health) para tener más información sobre el estado del entorno y el acceso opcional a un registro de destino. Por ejemplo, diagnostique errores de configuración de Docker o problemas de inicio de sesión de Azure Active Directory. 

Puede encontrar ejemplos de comandos en [Comprobación del mantenimiento de un registro de contenedor de Azure](container-registry-check-health.md). Si se notifican errores, revise la [referencia de error](container-registry-health-error-reference.md) y las siguientes secciones para ver las soluciones recomendadas.

> [!NOTE]
> Algunos errores de autenticación o autorización también pueden producirse si hay configuraciones de firewall o de red que impiden el acceso al registro. Consulte [Solución de problemas de red con el registro](container-registry-troubleshoot-access.md).

## <a name="potential-solutions"></a>Posibles soluciones

### <a name="check-docker-configuration"></a>Comprobar la configuración de Docker

La mayoría de los flujos de autenticación de Azure Container Registry requieren una instalación de Docker local, de forma que pueda autenticarse en el registro para realizar operaciones como insertar y extraer imágenes. Confirme que el cliente y el demonio de la CLI de Docker (motor de Docker) se ejecutan en su entorno. Necesita la versión del cliente de Docker 18.03 o posterior.

Vínculos relacionados:

* [Información general sobre la autenticación](container-registry-authentication.md#authentication-options)
* [Preguntas frecuentes sobre Container Registry](container-registry-faq.md)

### <a name="specify-correct-registry-name"></a>Especificar el nombre de registro correcto

Al usar `docker login`, proporcione el nombre completo del servidor de inicio de sesión del registro, por ejemplo, *myregistry.azurecr.io*. Asegúrese de usar solo letras minúsculas. Ejemplo:

```console
docker login myregistry.azurecr.io
```

Al usar [az acr login](/cli/azure/acr#az-acr-login) con una identidad de Azure Active Directory, primero [inicie sesión en la CLI de Azure](/cli/azure/authenticate-azure-cli) y, luego, especifique el nombre del recurso de Azure del registro. El nombre del recurso es el nombre que se proporcionó cuando se creó el registro, por ejemplo, *myregistry* (sin un sufijo de dominio). Ejemplo:

```azurecli
az acr login --name myregistry
```

Vínculos relacionados:

* [az acr login se ha ejecutado correctamente, pero docker ha producido el error: no autorizado: se necesita autorización](container-registry-faq.md#az-acr-login-succeeds-but-docker-fails-with-error-unauthorized-authentication-required )

### <a name="confirm-credentials-to-access-registry"></a>Confirmar las credenciales para acceder al registro

Compruebe la validez de las credenciales que usa para su escenario o que le proporcionó un propietario del registro. Otros problemas posibles:

* Si usa una entidad de servicio de Active Directory, asegúrese de utilizar las credenciales correctas en el inquilino de Active Directory:
  * Nombre de usuario: identificador de aplicación de la entidad de servicio (también denominado *identificador de cliente*)
  * Contraseña: contraseña de la entidad de servicio (también denominada *secreto de cliente*)
* Si usa un servicio de Azure, como Azure Kubernetes Service o Azure DevOps, para acceder al registro, confirme la configuración del registro del servicio.
* Si ejecutó `az acr login` con la opción `--expose-token`, que habilita el inicio de sesión del registro sin usar el demonio de Docker, asegúrese de autenticarse con el nombre de usuario `00000000-0000-0000-0000-000000000000`.

Vínculos relacionados:

* [Información general sobre la autenticación](container-registry-authentication.md#authentication-options)
* [Inicio de sesión individual con Azure AD](container-registry-authentication.md#individual-login-with-azure-ad)
* [Inicio de sesión con la entidad de servicio](container-registry-auth-service-principal.md)
* [Inicio de sesión con una identidad administrada](container-registry-authentication-managed-identity.md)
* [Inicio de sesión con un token de ámbito de repositorio](container-registry-repository-scoped-permissions.md)
* [Inicio de sesión con la cuenta de administrador](container-registry-authentication.md#admin-account)
* [Códigos de error de autenticación y autorización de Azure AD](../active-directory/develop/reference-aadsts-error-codes.md)
* Referencia de [az acr login](/cli/azure/acr#az-acr-login)

### <a name="confirm-credentials-are-authorized-to-access-registry"></a>Confirmar que las credenciales están autorizadas para acceder al registro

Confirme los permisos del registro asociados a las credenciales, como el rol de RBAC `AcrPull` para extraer imágenes del registro, o el rol `AcrPush` para insertar imágenes. 

Para acceder a un registro en el portal o administrar el registro mediante la CLI de Azure, se necesita como mínimo el rol `Reader` para realizar operaciones de Azure Resource Manager.

El usuario o el propietario del registro deben tener privilegios suficientes en la suscripción para agregar o quitar asignaciones de roles.

Vínculos relacionados:

* [Roles y permisos de RBAC: Azure Container Registry](container-registry-roles.md)
* [Inicio de sesión con un token de ámbito de repositorio](container-registry-repository-scoped-permissions.md)
* [Incorporación o eliminación de asignaciones de roles de Azure con Azure Portal](../role-based-access-control/role-assignments-portal.md)
* [Uso del portal para crear una aplicación de Azure AD y una entidad de servicio con acceso a los recursos](../active-directory/develop/howto-create-service-principal-portal.md)
* [Creación de un secreto de aplicación](../active-directory/develop/howto-create-service-principal-portal.md#create-a-new-application-secret)
* [Códigos de autenticación y autorización de Azure AD](../active-directory/develop/reference-aadsts-error-codes.md)

### <a name="check-that-credentials-arent-expired"></a>Comprobar que las credenciales no hayan expirado

Los tokens y las credenciales de Active Directory pueden expirar después de períodos definidos, lo que impide el acceso al registro. Para habilitar el acceso, es posible que sea necesario restablecer o volver a generar las credenciales.

* Si usa una identidad de AD individual, una identidad administrada o una entidad de servicio para el inicio de sesión del registro, el token de AD expira al cabo de 3 horas. Vuelva a iniciar sesión en el registro.  
* Si usa una entidad de servicio de AD con un secreto de cliente expirado, el propietario de la suscripción o el administrador de la cuenta deben restablecer las credenciales o generar una nueva entidad de servicio.
* Si usa un [token de ámbito de repositorio](container-registry-repository-scoped-permissions.md), es posible que el propietario del registro tenga que restablecer una contraseña o generar un nuevo token.

Vínculos relacionados:

* [Restablecimiento de las credenciales de la entidad de servicio](/cli/azure/ad/sp/credential#az-ad-sp-credential-reset)
* [Regeneración de contraseñas de token](container-registry-repository-scoped-permissions.md#regenerate-token-passwords)
* [Inicio de sesión individual con Azure AD](container-registry-authentication.md#individual-login-with-azure-ad)

## <a name="advanced-troubleshooting"></a>Pasos detallados para solucionar problemas de conexión a Escritorio remoto a máquinas virtuales Windows en Azure

Si la [colección de registros de recursos](container-registry-diagnostics-audit-logs.md) está habilitada en el registro, revise el registro ContainterRegistryLoginEvents. Este registro almacena los eventos y el estado de autenticación, incluida la identidad entrante y la dirección IP. Consulte el registro para conocer los [errores de autenticación del registro](container-registry-diagnostics-audit-logs.md#registry-authentication-failures). 

Vínculos relacionados:

* [Registros para la evaluación y auditoría de diagnóstico](container-registry-diagnostics-audit-logs.md)
* [Preguntas frecuentes sobre Container Registry](container-registry-faq.md)
* [Procedimientos recomendados para Azure Container Registry](container-registry-best-practices.md)

## <a name="next-steps"></a>Pasos siguientes

Si no encuentra aquí una solución para su problema, vea las siguientes opciones.

* Otros temas de solución de problemas del registro incluyen:
  * [Solución de problemas de red con el registro](container-registry-troubleshoot-access.md)
  * [Solución de problemas de rendimiento del registro](container-registry-troubleshoot-performance.md)
* Opciones de [soporte técnico de la comunidad](https://azure.microsoft.com/support/community/)
* [Preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/products/)
* [Abra una incidencia de soporte técnico](https://azure.microsoft.com/support/create-ticket/): según la información proporcionada, se podría ejecutar un diagnóstico rápido de los errores de autenticación en el registro


