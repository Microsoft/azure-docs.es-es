---
title: Versiones de Kubernetes compatibles en Azure Kubernetes Service
description: Obtener información sobre la directiva de soporte técnico de la versión de Kubernetes y el ciclo de vida de los clústeres en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 07/08/2020
author: palma21
ms.author: jpalma
ms.openlocfilehash: 45c60069c27a2bb9b34cc53eb394bd44a04bd0ba
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2020
ms.locfileid: "89144624"
---
# <a name="supported-kubernetes-versions-in-azure-kubernetes-service-aks"></a>Versiones de Kubernetes compatibles en Azure Kubernetes Service (AKS)

La Comunidad de Kubernetes libera versiones secundarias aproximadamente cada tres meses. Estas versiones incluyen nuevas características y mejoras. Versiones de revisión son más frecuentes (a veces semanales) y solo están previstas para correcciones de errores críticos en una versión secundaria. Estas versiones de revisión incluyen correcciones para vulnerabilidades de seguridad o errores importantes.

## <a name="kubernetes-versions"></a>Versiones de Kubernetes

Kubernetes usa el esquema de control de versiones [Versionamiento Semántico](https://semver.org/), lo que significa que cada versión de Kubernetes sigue este esquema de numeración:

```
[major].[minor].[patch]

Example:
  1.17.7
  1.17.8
```

Cada número en la versión indica compatibilidad general con la versión anterior:

* Las versiones principales cambian cuando cambia una API incompatible o la compatibilidad con versiones anteriores deja de funcionar.
* Las versiones secundarias cambian cuando se hacen cambios de funcionalidad que son compatibles con otras versiones anteriores secundarias.
* Las versiones de revisión cambian cuando se hacen correcciones de errores compatibles con versiones anteriores.

Los usuarios deben querer ejecutar la última versión de revisión de la versión secundaria que están ejecutando; por ejemplo, si su clúster de producción usa la versión **`1.17.7`** y **`1.17.8`** es la última versión de revisión disponible para la serie *1.17*, debe actualizar a **`1.17.8`** en cuanto pueda asegurarse de que el clúster esté completamente revisado y sea compatible.

## <a name="kubernetes-version-support-policy"></a>Directiva de soporte técnico de versión de Kubernetes

AKS define una versión disponible con carácter general, como una versión habilitada en todas las mediciones de SLO o SLA y cuando está disponible en todas las regiones. AKS es compatible con tres versiones secundarias en disponibilidad general de Kubernetes:

* La versión secundaria más reciente en disponibilidad general publicada en AKS (a la que nos referiremos como N). 
* Dos versiones secundarias anteriores. 
* Cada versión secundaria compatible también admite un máximo de dos (2) revisiones estables.
* AKS también puede admitir versiones preliminares que se etiquetan explícitamente y están sujetas a los [términos y condiciones de la versión preliminar][preview-terms].

> [!NOTE]
> AKS usa prácticas de implementación segura que implican la implementación gradual de regiones. Esto significa que pueden pasar hasta 10 días hábiles hasta que haya una nueva versión disponible en todas las regiones.

La ventana admitida de las versiones de Kubernetes en AKS se conoce como "N-2": (N [versión más reciente] - 2 [versiones secundarias]).

Por ejemplo, si AKS presenta *1.17.a* hoy, también se proporciona compatibilidad para las versiones siguientes:

Nueva versión secundaria    |    Lista de versiones admitidas
-----------------    |    ----------------------
1.17.a               |    1.17.a, 1.17.b, 1.16.c, 1.16.d, 1.15.e, 1.15.f

Donde ".letter" es representativo de las versiones de revisión.

Cuando se introduce una nueva versión secundaria, la versión secundaria y la versión de revisión compatibles más antiguas quedan obsoletas y se retiran. Por ejemplo, si la lista de versiones que se admiten actualmente es:

```
1.17.a
1.17.b
1.16.c
1.16.d
1.15.e
1.15.f
```

Y AKS publica 1,18.\*, todas las versiones 1.15.\* se quitarán y dejarán de ser compatibles en 30 días.

> [!NOTE]
> Tenga en cuenta que si los clientes ejecutan una versión incompatible de Kubernetes, se les pedirá que actualicen al solicitar soporte técnico para el clúster. Los clústeres que ejecutan versiones de Kubernetes no admitidas no están cubiertos por el [las directivas de soporte técnico de AKS](./support-policies.md).

Además de lo anterior, AKS admite un máximo de dos versiones de **revisión** de una versión secundaria determinada. Así pues, dadas las siguientes versiones admitidas:

```
Current Supported Version List
------------------------------
1.17.8, 1.17.7, 1.16.10, 1.16.9
```

Si AKS publica `1.17.9` y `1.16.11`, las versiones de revisión más antiguas quedan en desuso y se eliminan, y la lista de versiones admitidas pasa a ser:

```
New Supported Version List
----------------------
1.17.*9*, 1.17.*8*, 1.16.*11*, 1.16.*10*
```

### <a name="supported-kubectl-versions"></a>Versiones de `kubectl` admitidas

Puede usar una versión secundaria de `kubectl` que sea inmediatamente anterior o posterior a la versión de *kube-apiserver*, lo que es coherente con la [directiva de compatibilidad de Kubernetes para kubectl](https://kubernetes.io/docs/setup/release/version-skew-policy/#kubectl).

Por ejemplo, si la versión de *kube-apiserver* es *1.17*, puede usar las versiones de `kubectl` comprendidas entre *1.16* y *1.18* con esa versión de *kube-apiserver*.

Para instalar o actualizar la versión de `kubectl`, ejecute `az aks install-cli`.

## <a name="release-and-deprecation-process"></a>Proceso de publicación y desuso

Puede consultar las próximas publicaciones de versiones y las que se quedarán obsoletas en el [Calendario publicación de AKS Kubernetes](#aks-kubernetes-release-calendar).

Para nuevas versiones **secundarias** de Kubernetes
1. AKS publica un anuncio previo con la fecha planificada de la nueva publicación de una versión y el respectivo desuso de una versión antigua en las [notas de la versión de AKS](https://aka.ms/aks/releasenotes) al menos 30 días antes de la eliminación.
2. AKS publica una [notificación de estado del servicio](../service-health/service-health-overview.md) disponible para todos los usuarios con acceso al portal y AKS, y envía un correo electrónico a los administradores de suscripciones con las fechas de eliminación de versión planeadas.
3. Los usuarios tienen **30 días** a partir de la eliminación de una versión para actualizar a una versión secundaria compatible para seguir recibiendo soporte técnico.

Para nuevas versiones de **revisión** de Kubernetes
  * Debido a la naturaleza urgente de las versiones de revisión, se pueden introducir en el servicio a medida que estén disponibles.
  * En general, AKS no realiza grandes comunicaciones para la publicación de las nuevas versiones de revisión. Sin embargo, AKS supervisa y valida constantemente las revisiones de CVE disponibles para admitirlas en AKS de manera puntual. Si se encuentra una revisión crítica o se requiere una acción del usuario, AKS enviará una notificación a los usuarios para que actualicen a la revisión recién disponible.
  * Los usuarios tienen **30 días** desde el momento en que se quita una versión de revisión de AKS para actualizarla a una revisión compatible y seguir recibiendo soporte técnico.

### <a name="supported-versions-policy-exceptions"></a>Excepciones de directiva de versiones admitidas

AKS se reserva el derecho de agregar o eliminar versiones nuevas/existentes donde se hayan identificado uno o varios problemas de seguridad o errores críticos que afecten a la producción sin previo aviso.

Las versiones de revisión específicas se pueden omitir, o su implantación puede verse acelerada en función de la gravedad del error o el problema de seguridad.

## <a name="azure-portal-and-cli-versions"></a>Versiones de Azure Portal y CLI

Al implementar un clúster de AKS en Azure Portal o con la CLI de Azure, el clúster siempre se establece de forma predeterminada en la versión secundaria n-1 y la revisión más reciente. Por ejemplo, si AKS admite *1.17.a*, *1.17.b*, *1.16.c*, *1.16.d*, *1.15.e* y *1.15.f*, la versión predeterminada seleccionada es la *1.16.c*.

Para averiguar qué versiones están disponibles actualmente para su suscripción y región, utilice el comando [az aks get-versions][az-aks-get-versions]. En el ejemplo siguiente se enumeran las versiones de Kubernetes disponibles para la región *EastUS*:

```azurecli-interactive
az aks get-versions --location eastus --output table
```

## <a name="aks-kubernetes-release-calendar"></a>Calendario de publicación de AKS Kubernetes

Para ver el historial de versiones anteriores, consulte [aquí](https://en.wikipedia.org/wiki/Kubernetes#History).

|  Versión de K8s | Versión anterior  | Versión preliminar de AKS  | Disponibilidad general de AKS  | Fin de la vida útil |
|--------------|-------------------|--------------|---------|-------------|
| 1.17  | 9 de diciembre de 2019  | Enero de 2019   | Julio de 2020  | 1.20 disponibilidad general | 
| 1.18  | 23 de marzo de 2020  | Mayo de 2020   | Agosto de 2020  | 1.21 disponibilidad general | 
| 1.19  | 4 de agosto de 2020  | Agosto de 2020   | Noviembre de 2020  | 1.22 disponibilidad general | 
| 1.20  | *noviembre de 2020    | *diciembre de 2020   | *enero de 2021  | 1.23 Disponibilidad general | 

\* Confirmación de fecha de versión anterior pendiente.

## <a name="faq"></a>Preguntas más frecuentes

**¿Qué ocurre cuando un cliente actualiza un clúster de Kubernetes con una versión secundaria que no es compatible?**

Si está en el versión *n-3* o anterior, está fuera del soporte técnico y se le pedirá que la actualice. Si la actualización de la versión n-3 a n-2 se completa correctamente, estará dentro de nuestras directivas de soporte técnico. Por ejemplo:

- Si la versión de AKS admitida más antigua es la*1.15.a* y la suya es la *1.14.b* o anterior, no es compatible.
- Si la actualización de *1.14.b* a *1.15.a* se realiza correctamente, estará de nuevo dentro de nuestras directivas de soporte técnico.

No se admite el cambio a una versión anterior.

**¿Qué significa "fuera de soporte técnico"?**

"Fuera de soporte técnico" significa que la versión que está ejecutando está fuera de la lista de versiones admitidas, y se le pedirá que actualice el clúster a una versión compatible cuando solicite soporte técnico, a menos que estén en el período de gracia de 30 días después de que la versión haya quedado en desuso. Además, AKS no ofrece ningún entorno de ejecución ni otras garantías para los clústeres que estén fuera de la lista de versiones admitidas.

**¿Qué ocurre cuando un cliente escala un clúster de Kubernetes con una versión secundaria que no es compatible?**

En el caso de las versiones secundarias que no admite AKS, el escalado horizontal o vertical debería seguir funcionando, aunque no hay garantías de Calidad de servicio, por lo que se recomienda encarecidamente actualizar para que el clúster vuelva a ser compatible.

**¿Puede un cliente permanecer en una versión de Kubernetes para siempre?**

Si un clúster ha estado sin soporte técnico durante más de tres (3) versiones secundarias y se ha descubierto que presenta riesgos para la seguridad, Azure se pondrá en contacto con usted para que lo actualice con antelación. Si no hace nada, Azure se reserva el derecho de actualizar automáticamente el clúster en su nombre.

**¿Qué versión admite el plano de control si el grupo de nodos no está en una de las versiones de AKS compatibles?**

El plano de control debe estar dentro de una ventana de versiones de todos los grupos de nodos. Para obtener más información sobre la actualización del plano de control o de los grupos de nodos, visite la documentación sobre [cómo actualizar grupos de nodos](use-multiple-node-pools.md#upgrade-a-cluster-control-plane-with-multiple-node-pools).

**¿Puedo saltarme varias versiones de AKS durante la actualización del clúster?**

Cuando se actualiza un clúster de AKS compatible, no pueden omitirse las versiones secundarias de Kubernetes. Por ejemplo, se permiten las actualizaciones entre *1.12.x* -> *1.13.x* o *1.13.x* -> *1.14.x*, pero no entre *1.12.x* -> *1.14.x*.

Para actualizar de *1.12.x* -> *1.14.x*, la primera actualización sería de *1.12.x* -> *1.13.x* y, después, de *1.13.x* -> *1.14.x*.

Solo se pueden omitir varias versiones cuando se actualiza una versión que no es compatible a otra que sí lo es. Por ejemplo, cuando se actualiza de la versión *1.10.x*, que no es compatible, a la versión *1.15.x*, que sí lo es.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo actualizar el clúster, vea [Actualización de un clúster de Azure Kubernetes Service (AKS)][aks-upgrade].

<!-- LINKS - External -->
[aks-engine]: https://github.com/Azure/aks-engine
[azure-update-channel]: https://azure.microsoft.com/updates/?product=kubernetes-service

<!-- LINKS - Internal -->
[aks-upgrade]: upgrade-cluster.md
[az-aks-get-versions]: /cli/azure/aks#az-aks-get-versions
[preview-terms]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
