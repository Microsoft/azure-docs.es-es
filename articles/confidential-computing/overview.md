---
title: Introducción a la computación confidencial de Azure
description: Introducción a la computación confidencial de Azure (ACC)
services: virtual-machines
author: JBCook
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: overview
ms.date: 04/06/2020
ms.author: JenCook
ms.openlocfilehash: 4e92f974ce7d6c03143276808c4ca4d09d607a84
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835823"
---
# <a name="confidential-computing-on-azure"></a>Computación confidencial en Azure

La computación confidencial de Azure permite aislar los datos confidenciales mientras se procesan en la nube. Muchos sectores usan la computación confidencial para proteger sus datos. Algunas de las cargas de trabajo a las que va enfocada son:

- Protección de los datos financieros
- Protección de la información del paciente
- Ejecución de procesos de aprendizaje automático en información confidencial
- Realización de algoritmos en conjuntos de datos cifrados desde varios orígenes


## <a name="overview"></a>Información general
<p><p>


> [!VIDEO https://www.youtube.com/embed/rT6zMOoLEqI]

Sabemos que es importante proteger los datos en la nube, y somos conscientes de sus preocupaciones. Estas son solo algunas preguntas que nuestros clientes pueden plantearse al mover cargas de trabajo confidenciales a la nube: 

- ¿Cómo tener la seguridad de que Microsoft no puede acceder a los datos que no están cifrados?
- ¿Cómo se evitan las amenazas de seguridad de los administradores con privilegios dentro de mi empresa?
- ¿Cuáles son otras formas de evitar que otros usuarios accedan a los datos confidenciales de los clientes?

Microsoft Azure ayuda a reducir la superficie expuesta a los ataques para conseguir una mayor protección de los datos. Azure ya ofrece muchas herramientas para proteger los [**datos en reposo**](../security/fundamentals/encryption-atrest.md) mediante modelos como el cifrado del lado cliente y el cifrado del lado servidor. Además, Azure ofrece mecanismos para cifrar los [**datos en tránsito**](../security/fundamentals/data-encryption-best-practices.md#protect-data-in-transit) mediante protocolos seguros como TLS y HTTPS. En esta página se presenta un tercer tipo de cifrado de datos: el cifrado de los **datos en uso**.


## <a name="introduction-to-confidential-computing"></a>Introducción a la computación confidencial <a id="intro to acc"></a>

La computación confidencial es un término del sector que definió el [Consorcio de Computación Confidencial](https://confidentialcomputing.io/) (CCC), una fundación dedicada a definir y acelerar la adopción de la computación confidencial. El CCC define la informática confidencial como la protección de los datos en uso mediante la realización de cálculos en un entorno de ejecución de confianza (TEE) basado en hardware.

Un TEE es un entorno que aplica la ejecución de solo código autorizado. Los datos de TEE no se pueden leer ni alterar con ningún código ajeno a ese entorno.

### <a name="enclaves"></a>Enclaves

Los enclaves son partes protegidas del procesador y la memoria de hardware. No existen ninguna forma de ver los datos o el código del interior del enclave, ni siquiera con un depurador. Si los intentos de código que no son de confianza modifican el contenido de la memoria del enclave, el entorno se deshabilita y se deniegan las operaciones.

Al desarrollar aplicaciones, puede usar [herramientas de software](#oe-sdk) para blindar partes del código y de los datos que contiene el enclave. Estas herramientas garantizan que nadie que no pertenezca al entorno de confianza pueda ver o modificar el código y los datos. 

Básicamente, puede considerar un enclave como una caja protegida. El código y los datos cifrados se colocan en la caja. Por fuera de la caja, no puede ver nada. Al enclave se le proporciona una clave para descifrar los datos, luego se procesan estos y se cifran de nuevo, antes de enviarse fuera del enclave.

### <a name="attestation"></a>Atestación

Querrá que le comprueben y validen que su entorno de confianza es seguro. Esta comprobación es el proceso de atestación. 

La atestación permite que un usuario de confianza tenga mayor seguridad de que el software (1) se ejecuta en un enclave y (2) que el enclave está actualizado y es seguro. Por ejemplo, un enclave pide al hardware subyacente que genere una credencial que incluye la prueba de que el enclave existe en la plataforma. Luego, se puede dar al informe un segundo enclave que comprueba que el informe se generó en la misma plataforma.

La atestación debe implementarse mediante un servicio de atestación seguro que sea compatible con el software del sistema y el silicio. Los [servicios de atestación y aprovisionamiento de Intel](https://software.intel.com/sgx/attestation-services) son compatibles con las máquinas virtuales de computación confidencial de Azure.


## <a name="using-azure-for-cloud-based-confidential-computing"></a>Uso de Azure para la computación confidencial basada en la nube <a id="cc-on-azure"></a>

La computación confidencial de Azure le permite aprovechar las funcionalidades de computación confidencial en un entorno virtualizado. Ahora puede usar herramientas, software e infraestructura de nube para crear a partir de hardware seguro. 

### <a name="virtual-machines"></a>Virtual Machines

Azure es el primer proveedor de nube que ofrece computación confidencial en un entorno virtualizado. Hemos desarrollado máquinas virtuales que funcionan como una capa de abstracción entre el hardware y la aplicación. Puede ejecutar cargas de trabajo a escala y con opciones de redundancia y disponibilidad.  

#### <a name="intel-sgx-enabled-virtual-machines"></a>Virtual Machines habilitado para Intel SGX

En las máquinas virtuales de computación confidencial de Azure, una parte del hardware de la CPU se reserva para una parte del código y los datos de la aplicación. Esta parte restringida es el enclave. 

![Modelo de máquina virtual](media/overview/hardware-backed-enclave.png)

La infraestructura de la computación confidencial de Azure se compone actualmente de una SKU de especialidad de máquinas virtuales (VM). Estas máquinas virtuales se ejecutan en procesadores Intel con la extensión Software Guard Extension (Intel SGX). [Intel SGX](https://intel.com/sgx) es el componente que hace posible la mayor protección que se proporciona con la computación confidencial. 

En la actualidad, Azure ofrece la [serie DCsv2](https://docs.microsoft.com/azure/virtual-machines/dcv2-series) basada en la tecnología Intel SGX para la creación de enclaves basados en hardware. Puede compilar aplicaciones seguras basadas en el enclave para que se ejecuten en la serie DCsv2 de máquinas virtuales con el fin de proteger los datos de la aplicación y el código en uso. 

[Lea más](virtual-machine-solutions.md) sobre la implementación de máquinas virtuales de computación confidencial de Azure con enclaves de confianza basados en hardware.

## <a name="application-development"></a>Desarrollo de aplicaciones <a id="application-development"></a>

Para aprovechar la eficacia de los enclaves y los entornos aislados, deberá usar herramientas que admitan la computación confidencial. Hay varias herramientas que admiten el desarrollo de aplicaciones de enclave. Por ejemplo, puede usar estos marcos de código abierto: 

- [Open Enclave Software Development Kit (SDK)](https://github.com/openenclave/openenclave)
- [El marco de trabajo del Consorcio de Computación Confidencial (CCF)](https://github.com/Microsoft/CCF)

### <a name="overview"></a>Información general

Una aplicación compilada con enclaves se divide en dos sentidos:
1. Un componente que no es de confianza (el host)
1. Un componente que es de confianza (el enclave)

**El host** es donde se ejecuta la aplicación de enclave y es un entorno que no es de confianza. El host no puede acceder al código del enclave implementado en el host. 

**El enclave** es el lugar en el que se ejecuta el código de aplicación y sus datos o memoria en caché. Los cálculos seguros deben realizarse en los enclaves para garantizar que los secretos y los datos confidenciales estén protegidos. 

Durante el diseño de la aplicación, es importante identificar y determinar qué parte de la aplicación debe ejecutarse en los enclaves. El código que elija incluir en el componente de confianza está aislado del resto de la aplicación. Una vez que se inicializa el enclave y el código se carga en la memoria, ese código no se puede leer ni cambiar desde los componentes que no son de confianza. 

### <a name="open-enclave-software-development-kit-oe-sdk"></a>Open Enclave Software Development Kit (OE SDK) <a id="oe-sdk"></a>

Use una biblioteca o un marco de trabajo compatible con el proveedor si quiere escribir código que se ejecute en un enclave. [Open Enclave SDK](https://github.com/openenclave/openenclave) (OE SDK) es un SDK de código abierto que permite la abstracción sobre diferente hardware habilitado para la computación confidencial. 

OE SDK se compila como una única capa de abstracción en cualquier hardware de cualquier CSP. OE SDK se puede usar sobre las máquinas virtuales de computación confidencial de Azure para crear y ejecutar aplicaciones a partir de los enclaves.

## <a name="next-steps"></a>Pasos siguientes

Implemente una máquina virtual de la serie DCsv2 e instale OE SDK en ella.

> [!div class="nextstepaction"]
> [Implementación de una máquina virtual de computación confidencial en Azure Marketplace](quick-create-marketplace.md)
