---
title: 'Apéndice F: servicios de enlace de RTOS de GUIX'
description: Obtenga información sobre los servicios de enlace de RTOS de GUIX.
author: philmea
ms.author: philmea
ms.date: 05/19/2020
ms.topic: article
ms.service: rtos
ms.openlocfilehash: 1d94dbb9d7d53ec3e1900188142974cc981dfea9
ms.sourcegitcommit: e3d42e1f2920ec9cb002634b542bc20754f9544e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/22/2021
ms.locfileid: "104815566"
---
# <a name="appendix-f---guix-rtos-binding-services"></a><span data-ttu-id="9a0d6-103">Apéndice F: servicios de enlace de RTOS de GUIX</span><span class="sxs-lookup"><span data-stu-id="9a0d6-103">Appendix F - GUIX RTOS Binding Services</span></span>

<span data-ttu-id="9a0d6-104">GUIX requiere que el RTOS subyacente preste servicios de subprocesos o tareas, exclusión mutua, cola de eventos y control de tiempo.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-104">GUIX requires thread or tasking services, mutex, event queue, and timing services providing by the underlying RTOS.</span></span> <span data-ttu-id="9a0d6-105">De forma predeterminada, GUIX está configurado para que el sistema operativo en tiempo real ThreadX preste estos servicios.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-105">By default GUIX is configured to utilize the ThreadX real time operating system to provide these services.</span></span> <span data-ttu-id="9a0d6-106">Para migrar GUIX a otro sistema operativo, el desarrollador debe definir la directiva de preprocesador GX_DISABLE_THREADX_BINDING y recompilar la biblioteca de GUIX para eliminar las dependencias de ThreadX.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-106">To port GUIX to another operating system, the developer should # define the pre-processor directive GX_DISABLE_THREADX_BINDING and rebuild the GUIX library to remove the ThreadX dependencies.</span></span> <span data-ttu-id="9a0d6-107">Además, el desarrollador deberá proporcionar las siguientes definiciones de macro y funciones auxiliares.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-107">In addition, the developer will need to provide the following macro definitions and supporting functions.</span></span> <span data-ttu-id="9a0d6-108">Pueden encontrarse ejemplos de estas definiciones de macro y funciones auxiliares en los archivos gx_system_rtos_bind.h y gx_system_rtos_bind.c, donde se proporciona un ejemplo de integración de RTOS genérica.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-108">Examples of these macro definitions and supporting functions can be found in the files gx_system_rtos_bind.h and gx_system_rtos_bind.c, which provide an example generic rtos integration.</span></span>

<span data-ttu-id="9a0d6-109">Macros de integración de sistemas:</span><span class="sxs-lookup"><span data-stu-id="9a0d6-109">System Integration macros:</span></span>

<span data-ttu-id="9a0d6-110">**GX_RTOS_BINDING_INITIALIZE**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-110">**GX_RTOS_BINDING_INITIALIZE**</span></span>

<span data-ttu-id="9a0d6-111">Esta macro se invoca durante la inicialización del sistema.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-111">This macro is invoked during system initialization.</span></span> <span data-ttu-id="9a0d6-112">La macro se debe definir con el fin de llamar a cualquier función necesaria para preparar los recursos o servicios del sistema del RTOS antes de usarlos.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-112">The macro should be defined to call any function needed to prepare your rtos system services or rtos resources prior to use.</span></span> <span data-ttu-id="9a0d6-113">Esta es la oportunidad del enlace de preparar los recursos de RTOS que usará GUIX.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-113">This is the binding’s opportunity to prepare the rtos resources that GUIX will use.</span></span>

<span data-ttu-id="9a0d6-114">**GX_SYSTEM_THREAD_START**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-114">**GX_SYSTEM_THREAD_START**</span></span>

<span data-ttu-id="9a0d6-115">Esta macro se invoca cuando debe empezar a ejecutarse la tarea o el subproceso de GUIX.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-115">This macro is invoked when the GUIX task or thread should start executing.</span></span> <span data-ttu-id="9a0d6-116">Esta macro se debe definir para llamar a una función que iniciará la ejecución del subproceso de GUIX.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-116">This macro should be defined to call a function which will start the GUIX thread running.</span></span> <span data-ttu-id="9a0d6-117">El punto de entrada al subproceso de GUIX se pasa a la función llamada.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-117">The entry point to the GUIX thread is passed to the called function.</span></span> <span data-ttu-id="9a0d6-118">La firma de la función llamada debe ser la siguiente:</span><span class="sxs-lookup"><span data-stu-id="9a0d6-118">The signature of the called function must be</span></span>

<span data-ttu-id="9a0d6-119">**UINT *function_name*(VOID (thread_entry_point)(VOID));**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-119">**UINT *function_name*(VOID (thread_entry_point)(VOID));**</span></span>

<span data-ttu-id="9a0d6-120">Esta función debe devolver GX_SUCCESS si el subproceso se ha iniciado correctamente o GX_FAILURE en caso contrario.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-120">This function should return GX_SUCCESS if the thread is successfully started, or GX_FAILURE.</span></span>

<span data-ttu-id="9a0d6-121">**GX_EVENT_PUSH**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-121">**GX_EVENT_PUSH**</span></span>

<span data-ttu-id="9a0d6-122">Esta macro se invoca para insertar un evento en la cola de eventos FIFO que usa GUIX.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-122">This macro is invoked to push an event into the FIFO event queue used by GUIX.</span></span> <span data-ttu-id="9a0d6-123">Al migrar a un nuevo RTOS, es su responsabilidad implementar esta cola de eventos de una manera segura para los subprocesos.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-123">When porting to a new rtos, it is your responsibility to implement this event queue in a thread-safe manner.</span></span> <span data-ttu-id="9a0d6-124">Es preciso copiar y eliminar estructuras GX_EVENT en la cola, es decir, una cola de punteros GX_EVENT no servirá, ya que los eventos de GUIX pueden ser variables automáticas desde el punto de vista del productor de eventos.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-124">GX_EVENT structures must be copied into this queue and copied out of this queue, i.e. a queue of GX_EVENT pointers will not work, since GUIX events can be automatic variables from the view of the event producer.</span></span> <span data-ttu-id="9a0d6-125">La firma de la función a la que llama esta macro debe ser la siguiente:</span><span class="sxs-lookup"><span data-stu-id="9a0d6-125">The signature of the function called by this macro must be:</span></span>

<span data-ttu-id="9a0d6-126">\**UINT *function_name* (GX_EVENT *event_ptr);**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-126">\**UINT *function_name* (GX_EVENT *event_ptr);**</span></span>

<span data-ttu-id="9a0d6-127">Esta función debe devolver GX_SUCCESS si el evento se inserta en la cola de eventos o GX_FAILURE en caso contrario.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-127">This function should return GX_SUCCESS if the event is pushed into the event queue, otherwise it should return GX_FAILURE.</span></span>

<span data-ttu-id="9a0d6-128">**GX_EVENT_POP**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-128">**GX_EVENT_POP**</span></span>

<span data-ttu-id="9a0d6-129">Esta macro se invoca para eliminar el evento principal (el más antiguo) de la cola de eventos de GUIX y copiarlo en la ubicación solicitada.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-129">This macro is invoked to remove the head (oldest) event from the GUIX event queue and copy it into the requested location.</span></span> <span data-ttu-id="9a0d6-130">Esta función debe ser capaz de bloquear un evento u, opcionalmente, de esperar a que este se produzca, si no hay eventos en la cola en un momento dado.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-130">This function must be able to optionally block or wait for an event if no events are currently in the event queue.</span></span> <span data-ttu-id="9a0d6-131">La firma de la función que invoca esta macro debe ser la siguiente:</span><span class="sxs-lookup"><span data-stu-id="9a0d6-131">The signature of the function invoked by this macro must be</span></span>

<span data-ttu-id="9a0d6-132">UINT function_name(GX_EVENT \*put_event, GX_BOOL wait)</span><span class="sxs-lookup"><span data-stu-id="9a0d6-132">UINT function_name(GX_EVENT \*put_event, GX_BOOL wait)</span></span>

<span data-ttu-id="9a0d6-133">Si el parámetro “wait” es GX_TRUE, la función no debe devolver nada hasta que no se proporcione un evento.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-133">If the wait parameter == GX_TRUE, the function should not return until an event is provided.</span></span> <span data-ttu-id="9a0d6-134">Si el parámetro “wait” es GX_FALSE, la función debe devolver el estado inmediatamente, con o sin evento.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-134">If the wait parameter is GX_FALSE, the function should return immediately with or without an event.</span></span>

<span data-ttu-id="9a0d6-135">Si se recupera un evento de la cola, se debe copiar en la ubicación put_event. El estado devuelto será GX_SUCCESS.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-135">If an event is retrieved from the queue, it should copied into the put_event location and the return status is GX_SUCCESS.</span></span> <span data-ttu-id="9a0d6-136">De lo contrario, el estado debe ser GX_FAILURE.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-136">Otherwise the return status should be GX_FAILURE.</span></span>

<span data-ttu-id="9a0d6-137">**GX_EVENT_FOLD**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-137">**GX_EVENT_FOLD**</span></span>

<span data-ttu-id="9a0d6-138">GUIX invoca esta macro para plegar un evento en la cola de eventos FIFO.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-138">This macro is invoked by GUIX to fold an event into the FIFO event queue.</span></span> <span data-ttu-id="9a0d6-139">“Plegar” un evento significa que, si ya existe un evento del mismo tipo en la cola, esa entrada se actualiza para que contenga la carga del nuevo.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-139">Folding an event means that if an event of the same type already exists in the queue, that entry is update to contain the payload of the new event.</span></span> <span data-ttu-id="9a0d6-140">Si no se encuentra en la cola un evento ya existente del mismo tipo, se inserta en ella uno nuevo.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-140">If an existing event of the same type is not found in the queue, a new event is pushed into the queue.</span></span> 

<span data-ttu-id="9a0d6-141">En el caso de los enlaces que no puedan implementar la característica de plegado de eventos, es admisible simplemente invocar GX_EVENT_PUSH.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-141">For bindings that cannot implement the event fold feature, it is acceptable to simply invoke the GX_EVENT_PUSH.</span></span>

<span data-ttu-id="9a0d6-142">**GX_TIMER_START**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-142">**GX_TIMER_START**</span></span>

<span data-ttu-id="9a0d6-143">Esta macro se invoca cuando GUIX necesita recibir una entrada periódica de temporizador.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-143">This macro is invoked when GUIX needs to receive periodic timer input.</span></span> <span data-ttu-id="9a0d6-144">Esta macro debe invocar un servicio que inicie el servicio de temporizador periódico de RTOS de bajo nivel.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-144">This macro should invoke a service that starts the low-level RTOS periodic timer service.</span></span> <span data-ttu-id="9a0d6-145">Si el servicio de temporizador de RTOS no se puede detener e iniciar fácilmente, es admisible, pero menos eficaz, dejar este servicio en ejecución de manera continua.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-145">If the RTOS timer service cannot be easily stopped and started, it is acceptable but less efficient to leave this service running at all times.</span></span>

<span data-ttu-id="9a0d6-146">Cada vez que el servicio de temporizador de RTOS de bajo nivel expire, el enlace debe llamar a la función del sistema GUIX _gx_system_timer_expiration (0);. La llamada periódica a esta función es lo que activa los servicios de temporizador del widget de temporizador de GUIX de alto nivel.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-146">When the low-level RTOS timer service periodically expires, the binding must call the GUIX system function _gx_system_timer_expiration(0); Calling this function periodically is what drives the high-level GUIX timer widget timer services.</span></span>

<span data-ttu-id="9a0d6-147">**GX_TIMER_STOP**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-147">**GX_TIMER_STOP**</span></span>

<span data-ttu-id="9a0d6-148">Esta macro se invoca cuando GUIX ya no necesita un temporizador periódico (es decir, no hay en ejecución temporizadores de GUIX activos).</span><span class="sxs-lookup"><span data-stu-id="9a0d6-148">This macro is invoked when GUIX no longer needs a periodic timer (i.e. there are no active GUIX timers running).</span></span> <span data-ttu-id="9a0d6-149">Si el servicio de temporizador de RTOS no se puede detener e iniciar fácilmente, es admisible, pero menos eficaz, dejar este servicio en ejecución de manera continua y definir esta macro para no hacer nada.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-149">If the RTOS timer service cannot be easily stopped and started, it is acceptable but less efficient to leave this service running at all times and define this macro to do nothing.</span></span>

<span data-ttu-id="9a0d6-150">**GX_SYSTEM_MUTEX_LOCK**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-150">**GX_SYSTEM_MUTEX_LOCK**</span></span>

<span data-ttu-id="9a0d6-151">GUIX invoca esta macro durante secciones de código críticas para impedir que otra tarea reemplace y modifique estructuras de datos comunes y puedan producirse daños.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-151">This macro is invoked by GUIX during critical code sections to prevent another task from  pre-empting and modifying common data structures, potentially causing corruption.</span></span> <span data-ttu-id="9a0d6-152">Esta macro debe llamar a una función que implemente el servicio de bloqueo de recursos de RTOS adecuado.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-152">This macro should call a function that implements the suitable RTOS resource locking service.</span></span>

<span data-ttu-id="9a0d6-153">Si nunca usa servicios de API de GUIX fuera del subproceso de la solución, puede definir esta macro para no hacer nada.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-153">If you never utilize any GUIX API services outside of the GUIX thread, you can define this macro to do nothing.</span></span>

<span data-ttu-id="9a0d6-154">**GX_SYSTEM_MUTEX_UNLOCK**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-154">**GX_SYSTEM_MUTEX_UNLOCK**</span></span>

<span data-ttu-id="9a0d6-155">Esta macro se invoca al final de secciones de código críticas y debe desbloquear el recurso de GUIX mediante el servicio de RTOS subyacente adecuado.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-155">This macro is invoked at the end of critical code sections, and should unlock the GUIX resource using the suitable underlying RTOS service.</span></span> <span data-ttu-id="9a0d6-156">Si nunca usa servicios de API de GUIX fuera del subproceso de la solución, puede definir esta macro para no hacer nada.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-156">If you never utilize any GUIX API services outside of the GUIX thread, you can define this macro to do nothing.</span></span>

<span data-ttu-id="9a0d6-157">**GX_SYSTEM_TIME_GET**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-157">**GX_SYSTEM_TIME_GET**</span></span>

<span data-ttu-id="9a0d6-158">Esta macro debe llamar a una función que devuelva la hora del sistema en “tics”, que suele ser el número de interrupciones del temporizador de bajo nivel que se han producido desde el inicio del sistema.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-158">This macro should call a function that returns the current system time is “system ticks”, which is usually the number of low-level timer interrupts that have occurred since system startup.</span></span> <span data-ttu-id="9a0d6-159">Este servicio se usa para calcular la velocidad del lápiz de eventos táctiles para los gestos de entrada táctil.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-159">This service is used to calculate touch event pen speed for touch input gestures.</span></span> <span data-ttu-id="9a0d6-160">La firma de la función que invoca esta macro debe ser la siguiente:</span><span class="sxs-lookup"><span data-stu-id="9a0d6-160">The signature of the function invoked by this macro must be:</span></span>

<span data-ttu-id="9a0d6-161">**ULONG *function_name*(VOID);**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-161">**ULONG *function_name*(VOID);**</span></span>

<span data-ttu-id="9a0d6-162">**GX_CURRENT_THREAD**</span><span class="sxs-lookup"><span data-stu-id="9a0d6-162">**GX_CURRENT_THREAD**</span></span>

<span data-ttu-id="9a0d6-163">Esta macro se invoca para identificar el subproceso que se está ejecutando.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-163">This macro is invoked to identify the currently executing thread.</span></span> <span data-ttu-id="9a0d6-164">El servicio al que llama esta macro debe devolver un valor “void*”, lo que significa que el tipo de datos que usa el sistema operativo para identificar el subproceso en ejecución se debe convertir a un valor “void*” para que se devuelva a GUIX.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-164">The service called by this macro must return a void \*, meaning that the data type used by your operating system to identify the current execution thread must be cast to a void \* to be returned to GUX.</span></span>

<span data-ttu-id="9a0d6-165">En los archivos gx_system_rtos_bind.h y gx_system_rtos_bind.c se implementa un ejemplo de enlace de RTOS genérico.</span><span class="sxs-lookup"><span data-stu-id="9a0d6-165">A complete example of a generic RTOS binding is implemented in the filed gx_system_rtos_bind.h and gx_system_rtos_bind.c</span></span>