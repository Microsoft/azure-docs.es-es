---
title: Introducción a Azure Attestation
description: Información general sobre Microsoft Azure Attestation, una solución para la atestación de entornos de ejecución de confianza (TEE)
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.custom: references_regions
ms.openlocfilehash: ad164f8af3e5506ae5ac9121010b99303286dd1f
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89320603"
---
# <a name="microsoft-azure-attestation-preview"></a>Microsoft Azure Attestation (versión preliminar)

Microsoft Azure Attestation (versión preliminar) es una solución para la atestación de entornos de ejecución de confianza (TEE), como los enclaves de [Intel® Software Guard Extensions](https://www.intel.com/content/www/us/en/architecture-and-technology/software-guard-extensions.html) (SGX) y los enclaves de [seguridad basada en la virtualización](/windows-hardware/design/device-experiences/oem-vbs) (VBS). La atestación del enclave es el proceso que permite comprobar si un enclave es seguro y confiable.

La atestación es un proceso que se utiliza para demostrar que se ha creado correctamente una instancia de los archivos binarios de software en una plataforma de confianza. Así, los usuarios de confianza remotos pueden tener la seguridad de que solo se ejecuta el software previsto en el hardware de confianza. Azure Attestation consta de un servicio y un marco de trabajo unificados orientados al cliente para la atestación.

Azure Attestation utiliza paradigmas de seguridad de vanguardia, como la [computación confidencial en Azure](../confidential-computing/overview.md) y la protección de inteligencia perimetral. Los clientes llevan tiempo solicitando la posibilidad de comprobar de forma independiente la ubicación de una máquina, la posición de una máquina virtual en esa máquina y el entorno en el que se ejecutan los enclaves en esa máquina virtual. Con Azure Attestation se podrán satisfacer estas y muchas otras solicitudes de los clientes.

Azure Attestation recibe la evidencia de entidades de proceso, las convierte en un conjunto de notificaciones, las valida con respecto a directivas configurables y genera pruebas criptográficas para aplicaciones basadas en notificaciones (por ejemplo, usuarios de confianza y autoridades de auditoría).

## <a name="use-cases"></a>Casos de uso

Azure Attestation proporciona completos servicios de atestación para varios entornos y diferentes casos de uso.

### <a name="sgx-attestation"></a>Atestación de SGX

SGX hace referencia al aislamiento de nivel de hardware, que se admite en ciertos modelos de CPU de Intel. SGX permite que el código se ejecute en compartimientos saneados conocidos como enclaves SGX. Después, el hardware administra los permisos de acceso y memoria para garantizar una superficie mínima de ataque con el aislamiento adecuado.

Las aplicaciones cliente se pueden diseñar para aprovechar las ventajas de los enclaves SGX y delegar tareas que afectan a la seguridad de modo que se lleven a cabo dentro de esos enclaves. Estas aplicaciones pueden usar Azure Attestation para establecer de forma rutinaria la confianza en el enclave y su capacidad para acceder a datos confidenciales.

### <a name="vbs-attestation"></a>Atestación de VBS

VBS es una arquitectura basada en software para la protección de memoria del enclave basada en Hyper-V. Esta protección evita que el código de administración del host, así como los administradores de servicios locales y en la nube, tengan acceso a los datos de un enclave VBS o afecten a su ejecución.

De forma similar a la tecnología SGX, Azure Attestation admite la validación de los enclaves VBS con respecto a directivas configuradas y la emisión de una declaración de certificación como prueba de validez.

### <a name="open-enclave"></a>Open Enclave
[Open Enclave](https://openenclave.io/sdk/) (OE) es una colección de bibliotecas destinadas a crear una única abstracción unificada de enclave para que los desarrolladores creen aplicaciones basadas en TEE. Ofrece un modelo de aplicación segura universal que reduce las características específicas de cada plataforma. Microsoft considera que esta colección es un paso esencial para democratizar las tecnologías de enclave basadas en hardware, como SGX, y aumentar su aceptación en Azure.

OE normaliza los requisitos específicos para la comprobación de una evidencia de enclave. Esto hace de OE un consumidor de atestación de Azure Attestation con gran capacidad de ajuste.

## <a name="azure-attestation-can-run-in-a-tee"></a>Azure Attestation se puede ejecutar en un TEE

Azure Attestation es fundamental en los escenarios de computación confidencial, ya que realiza las acciones siguientes:

- Comprueba si la evidencia del enclave es válida.
- Evalúa la evidencia del enclave con respecto a una directiva definida por el cliente.
- Administra y almacena directivas específicas del inquilino.
- Genera y firma un token que emplean los usuarios de confianza para interactuar con el enclave.

Azure Attestation se ha creado para ejecutarse en dos tipos de entornos:
- Azure Attestation se ejecuta en un entorno de ejecución de confianza habilitado para SGX.
- Azure Attestation se ejecuta en un entorno que no es de ejecución de confianza.

Los clientes de Azure Attestation han pedido que Microsoft esté de forma operativa fuera de una base informática de confianza. Con esto se pretende evitar que las entidades de Microsoft, como administradores de máquinas virtuales, administradores de hosts y desarrolladores de Microsoft, modifiquen las solicitudes de atestación, las directivas y los tokens emitidos por Azure Attestation. Azure Attestation también se ha creado para ejecutarse en un TEE, en el que las características de Azure Attestation, como la validación de ofertas, la generación de tokens y la firma de tokens, se mueven a un enclave SGX.

## <a name="why-use-azure-attestation"></a>Razones para usar Azure Attestation

Azure Attestation es la opción preferida para la atestación de los entornos de ejecución de confianza, ya que ofrece las siguientes ventajas: 

- Ofrece un marco unificado para la atestación de varios TEE, como los enclaves SGX y VBS.
- Es un servicio multiinquilino que permite la configuración de directivas y proveedores de atestación personalizados para restringir la generación de tokens.
- Ofrece proveedores predeterminados que pueden encargarse de la atestación sin que los usuarios deban realizar ninguna configuración.
- Protege sus datos mientras está en uso con la implementación en un enclave SGX.
- Es un servicio de alta disponibilidad que ofrecerá Acuerdo de Nivel de Servicio (SLA).

## <a name="business-continuity-and-disaster-recovery-bcdr-support"></a>Compatibilidad con la continuidad empresarial y recuperación ante desastres (BCDR)

La [continuidad empresarial y recuperación ante desastres ](/azure/best-practices-availability-paired-regions) (BCDR) para Azure Attestation permiten reducir las interrupciones del servicio que se producen como consecuencia de desastres o problemas de disponibilidad significativos en una región.

A continuación se muestran las regiones admitidas actualmente por BCDR
- Este de EE. UU. 2 = > emparejado con Centro de EE. UU.
- Centro de EE. UU. = > emparejado con Este de EE. UU. 2.

En circunstancias normales, los clústeres implementados en dos regiones funcionarán de forma independiente. Si se produce un error o una interrupción en una región, sucederá lo siguiente:

- La BCDR de Azure Attestation proporcionará una conmutación por error fluida, en la que los clientes no tendrán que realizar ningún paso adicional para la recuperación.
- El servicio de [Azure Traffic Manager](../traffic-manager/index.yml) para la región detectará que el sondeo de estado está degradado y cambiará el punto de conexión a la región emparejada.
- Las conexiones existentes no funcionarán y recibirán problemas de tiempo de espera o errores internos del servidor.
- Se bloquearán todas las operaciones del plano de control. Los clientes no podrán crear proveedores de atestación ni actualizar directivas en la región primaria.
- Todas las operaciones del plano de datos, incluidas las atestaciones, seguirán funcionando en la región primaria.

## <a name="next-steps"></a>Pasos siguientes
- Más información acerca de los [conceptos básicos de Azure Attestation](basic-concepts.md)
- [Creación y firma de una directiva de atestación](author-sign-policy.md)
- [Configuración de Azure Attestation con PowerShell](quickstart-powershell.md)

