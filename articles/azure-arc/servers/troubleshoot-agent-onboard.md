---
title: Solución de problemas de conexión del agente de Azure Arc para servidores
description: En este artículo se explica cómo solucionar y resolver problemas relacionados con el agente de Connected Machine que surgen de Azure Arc para servidores (versión preliminar) al intentar conectarse al servicio.
ms.date: 07/20/2020
ms.topic: conceptual
ms.openlocfilehash: 5cd2560279085485a8ac7b285e4f601060a924f1
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88118015"
---
# <a name="troubleshoot-the-connected-machine-agent-connection-issues"></a>Solución de problemas de conexión del agente de Connected Machine

En este artículo se proporciona información sobre la solución y resolución de problemas que pueden producirse al intentar configurar el agente de Connected Machine de Azure Arc para servidores (versión preliminar) para Windows o Linux. Se incluyen los métodos de instalación tanto interactiva como a escala al configurar la conexión al servicio. Para obtener información general, consulte la [información general de Arc para servidores](./overview.md).

## <a name="agent-verbose-log"></a>Registro detallado del agente

Antes de seguir los pasos de solución de problemas que se describen más adelante en este artículo, la información mínima que necesita es el registro detallado. Contiene la salida de los comandos de la herramienta **azcmagent**, cuando se usa el argumento verbose (-v). Los archivos de registro se escriben en `%ProgramData%\AzureConnectedMachineAgent\Log\azcmagent.log` para Windows, y en `/var/opt/azcmagent/log/azcmagent.log` para Linux.

### <a name="windows"></a>Windows

A continuación encontrará un ejemplo del comando para habilitar el registro detallado con el agente de Connected Machine para Windows al realizar una instalación interactiva.

```
& "$env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe" connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --verbose
```

El siguiente es un ejemplo del comando para habilitar el registro detallado con el agente de Connected Machine para Windows al realizar una instalación a escala mediante una entidad de servicio.

```
& "$env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe" connect `
  --service-principal-id "{serviceprincipalAppID}" `
  --service-principal-secret "{serviceprincipalPassword}" `
  --resource-group "{ResourceGroupName}" `
  --tenant-id "{tenantID}" `
  --location "{resourceLocation}" `
  --subscription-id "{subscriptionID}"
  --verbose
```

### <a name="linux"></a>Linux

A continuación encontrará un ejemplo del comando para habilitar el registro detallado con el agente de Connected Machine para Linux al realizar una instalación interactiva.

>[!NOTE]
>Debe tener permisos de acceso *raíz* en máquinas Linux para ejecutar **azcmagent**.

```
azcmagent connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --verbose
```

El siguiente es un ejemplo del comando para habilitar el registro detallado con el agente de Connected Machine para Linux al realizar una instalación a escala mediante una entidad de servicio.

```
azcmagent connect \
  --service-principal-id "{serviceprincipalAppID}" \
  --service-principal-secret "{serviceprincipalPassword}" \
  --resource-group "{ResourceGroupName}" \
  --tenant-id "{tenantID}" \
  --location "{resourceLocation}" \
  --subscription-id "{subscriptionID}"
  --verbose
```

## <a name="agent-connection-issues-to-service"></a>Problemas de conexión del agente al servicio

En la tabla siguiente se enumeran algunos de los errores conocidos y las sugerencias sobre cómo solucionarlos y resolverlos.

|Message |Error |Causa probable |Solución |
|--------|------|---------------|---------|
|No se puede adquirir el flujo de dispositivo del token de autorización. |`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp 40.126.9.7:443: connect: network is unreachable.` |No se puede comunicar con el punto de conexión `login.windows.net`. | Verifique la conectividad al punto de conexión. |
|No se puede adquirir el flujo de dispositivo del token de autorización. |`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp 40.126.9.7:443: connect: network is Forbidden`. |Un proxy o firewall bloquean el acceso al punto de conexión `login.windows.net`. | Verifique la conectividad al punto de conexión y que no esté bloqueada por un firewall o un servidor proxy. |
|No se puede adquirir el flujo de dispositivo del token de autorización.  |`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp lookup login.windows.net: no such host`. | El objeto de directiva de grupo *Computer Configuration\ Administrative Templates\ System\ User Profiles\ Eliminar perfiles de usuario con una antigüedad superior al número de días especificado al reiniciar el sistema* está habilitado. | Compruebe que el GPO está habilitado y que tiene como destino la máquina afectada. Consulte la nota al pie <sup>[1](#footnote1)</sup> para más información. |
|No se puede adquirir el token de autorización del SPN. |`Failed to execute the refresh request. Error = 'Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/token?api-version=1.0: Forbidden'` |Un proxy o firewall bloquean el acceso al punto de conexión `login.windows.net`. |Verifique la conectividad al punto de conexión y que no esté bloqueada por un firewall o un servidor proxy. |
|No se puede adquirir el token de autorización del SPN. |`Invalid client secret is provided` |Secreto incorrecto o no válido de la entidad de servicio. |Verifique el secreto de la entidad de servicio. |
| No se puede adquirir el token de autorización del SPN. |`Application with identifier 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' was not found in the directory 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'. This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant` |Entidad de servicio o identificador de inquilino incorrectos. |Verifique la entidad de servicio o el identificador del inquilino.|
|Obtener respuesta a recursos de ARM |`The client 'username@domain.com' with object id 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' does not have authorization to perform action 'Microsoft.HybridCompute/machines/read' over scope '/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.HybridCompute/machines/MSJC01' or the scope is invalid. If access was recently granted, please refresh your credentials."}}" Status Code=403` |Credenciales o permisos incorrectos. |Verifique que usted o la entidad de servicio sean miembros del rol **Incorporación de Azure Connected Machine**. |
|No se puede aplicar AzcmagentConnectr al recurso de ARM. |`The subscription is not registered to use namespace 'Microsoft.HybridCompute'` |Los proveedores de recursos de Azure no están registrados. |Registre los [proveedores de recursos](./agent-overview.md#register-azure-resource-providers). |
|No se puede aplicar AzcmagentConnectr al recurso de ARM. |`Get https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.HybridCompute/machines/MSJC01?api-version=2019-03-18-preview:  Forbidden` |El servidor proxy o firewall bloquean el acceso al punto de conexión `management.azure.com`. |Verifique la conectividad al punto de conexión y que no esté bloqueada por un firewall o un servidor proxy. |

<a name="footnote1"></a><sup>1</sup>Si este GPO está habilitado y se aplica a máquinas con el agente de Connected Machine, se elimina el perfil de usuario asociado a la cuenta integrada especificada para el servicio *himds*. Como resultado, también elimina el certificado de autenticación usado para comunicarse con el servicio almacenado en caché en el almacén local de certificados durante 30 días. Antes del límite de 30 días, se realiza un intento de renovar el certificado. Para resolver este problema, siga los pasos para [anular el registro de la máquina](manage-agent.md#unregister-machine) y, luego, vuelva a registrarlo en el servicio mediante `azcmagent connect`.

## <a name="next-steps"></a>Pasos siguientes

Si su problema no aparece aquí o no puede resolverlo, intente obtener ayuda adicional mediante uno de los siguientes canales:

* Obtenga respuestas de expertos de Azure a través de [Preguntas y respuestas de Microsoft](/answers/topics/azure-arc.html).

* Póngase en contacto con [@AzureSupport](https://twitter.com/azuresupport), la cuenta oficial de Microsoft Azure para mejorar la experiencia del cliente. El Soporte técnico de Azure pone en contacto a la comunidad de Azure con respuestas, soporte técnico y expertos.

* Registrar un incidente de soporte técnico de Azure. Vaya al [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte técnico**.