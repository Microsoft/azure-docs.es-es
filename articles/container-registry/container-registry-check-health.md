---
title: Comprobación del estado de un registro
description: Obtenga información sobre la ejecución de comando de diagnóstico rápido para identificar problemas comunes del uso de un registro de contenedor de Azure, incluidas la configuración de Docker local y la conectividad al registro.
ms.topic: article
ms.date: 07/02/2019
ms.openlocfilehash: f27a99818260553cbd7ba26158db0064c145a21f
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88245390"
---
# <a name="check-the-health-of-an-azure-container-registry"></a>Comprobación del mantenimiento de un registro de contenedor de Azure

Cuando se usa un registro de contenedor de Azure, en ocasiones pueden surgir problemas. Por ejemplo, es posible que no pueda extraer una imagen de contenedor debido a un problema con Docker en su entorno local. O bien, un problema de red podría impedirle conectarse al registro. 

Como primer paso de diagnóstico, ejecute el comando [az acr check-helth][az-acr-check-health] para obtener información sobre el estado del entorno y el acceso opcional a un registro de destino. Este comando está disponible en la CLI de Azure versión 2.0.67 o posterior. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli].

Para obtener instrucciones adicionales sobre la solución de problemas del registro, consulte:
* [Solución de problemas de inicio de sesión del registro](container-registry-troubleshoot-login.md)
* [Solución de problemas de red con el registro](container-registry-troubleshoot-access.md)
* [Solución de problemas de rendimiento del registro](container-registry-troubleshoot-performance.md)

## <a name="run-az-acr-check-health"></a>Ejecución del comando az acr check-health

Los ejemplos siguientes muestran distintas formas de ejecutar el comando `az acr check-health`.

> [!NOTE]
> Si el comando se ejecuta en Azure Cloud Shell, no se comprueba el entorno local. Sin embargo, puede comprobar el acceso a un registro de destino.

### <a name="check-the-environment-only"></a>Comprobación solo del entorno

Para comprobar el demonio de Docker local, la versión de la CLI y la configuración de cliente de Helm, ejecute el comando sin parámetros adicionales:

```azurecli
az acr check-health
```

### <a name="check-the-environment-and-a-target-registry"></a>Comprobación del entorno y de un registro de destino

Para comprobar el acceso a un registro, así como realizar comprobaciones del entorno local, pase el nombre de un registro de destino. Por ejemplo:

```azurecli
az acr check-health --name myregistry
```

## <a name="error-reporting"></a>Notificación de errores

El comando registra información en la salida estándar. Si se detecta un problema, proporciona un código de error y una descripción. Para obtener más información acerca de los códigos y posibles soluciones, consulte la [referencia del error](container-registry-health-error-reference.md).

De forma predeterminada, el comando se detiene cuando encuentra un error. También puede ejecutar el comando para que proporcione la salida de todas las comprobaciones de mantenimiento, incluso si se encuentran errores. Agregue el parámetro `--ignore-errors` como se muestra en los ejemplos siguientes:

```azurecli
# Check environment only
az acr check-health --ignore-errors

# Check environment and target registry
az acr check-health --name myregistry --ignore-errors
```      

Salida del ejemplo:

```console
$ az acr check-health --name myregistry --ignore-errors --yes

Docker daemon status: available
Docker version: Docker version 18.09.2, build 6247962
Docker pull of 'mcr.microsoft.com/mcr/hello-world:latest' : OK
ACR CLI version: 2.2.9
Helm version:
Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
DNS lookup to myregistry.azurecr.io at IP 40.xxx.xxx.162 : OK
Challenge endpoint https://myregistry.azurecr.io/v2/ : OK
Fetch refresh token for registry 'myregistry.azurecr.io' : OK
Fetch access token for registry 'myregistry.azurecr.io' : OK
```  



## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los códigos de error que devuelve el comando [az acr check-health][az-acr-check-health], consulte la [referencia de errores de la comprobación de mantenimiento](container-registry-health-error-reference.md).

Vea las [preguntas más frecuentes](container-registry-faq.md) para conocer las preguntas más habituales y otros problemas conocidos sobre Azure Container Registry.





<!-- LINKS - internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-check-health]: /cli/azure/acr#az-acr-check-health
