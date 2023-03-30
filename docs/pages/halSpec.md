@mainpage

# AudioCapture HAL Documentation

## Version and Version History

| Date | Author | Comment | Version |
| --- | --------- | --- | --- |
| 27/03/23 | Chaithen Sojan | First Release | 1.0.0 |

## Table of Contents

  * [Overview](#overview)
    + [Acronyms](#acronyms)
    + [Description](#description)
- [AudioCapture HAL Documentation](#audiocapture-hal-documentation)
  - [Version and Version History](#version-and-version-history)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
    - [Acronyms](#acronyms)
    - [Description](#description)
- [Component Runtime Execution Requirements](#component-runtime-execution-requirements)
  - [Initialization and Startup](#initialization-and-startup)
  - [Threading Model](#threading-model)
  - [Process Model](#process-model)
  - [Memory Model](#memory-model)
  - [Power Management Requirements](#power-management-requirements)
  - [Asynchronous Notification Model](#asynchronous-notification-model)
  - [Blocking calls](#blocking-calls)
  - [Internal Error Handling](#internal-error-handling)
  - [Persistence Model](#persistence-model)
- [Non-functional requirements](#non-functional-requirements)
  - [Logging and debugging requirements](#logging-and-debugging-requirements)
  - [Memory and performance requirements](#memory-and-performance-requirements)
  - [Quality Control](#quality-control)
  - [Licensing](#licensing)
  - [Build Requirements](#build-requirements)
  - [Variability Management](#variability-management)
  - [Platform or Product Customization](#platform-or-product-customization)
- [Interface API Documentation](#interface-api-documentation)
  - [Theory of operation](#theory-of-operation)
    - [Architecture Diagram](#architecture-diagram)
    - [Sequence Diagram](#sequence-diagram)
    - [State Diagram](#state-diagram)
## Overview

### Acronyms

- `API` - Application Programming Interface
- `HAL` - Hardware Abstraction layer
- `PCM` - Pulse Code Modulation
- `RDK` - Reference Development Kit
- `RMF` - RDK Media Framework
- `STB` - Set Top Box

### Description

AudioCapture `HAL` must deliver a constant stream of raw audio data (`PCM`) to the caller. This audio will have the active audio track
of whatever content is being played, and may include other sounds mixed in, such as voice-guidance, application-driven
audio overlays (eg: beeps) etc. The audio data delivered via this interface is required to track as closely as possible, i. e., minimal latency,
to the audio that's being rendered by the device at a given point of time.

# Component Runtime Execution Requirements

These requirements ensure that the `HAL` executes correctly within the run-time environment that it will be used in.

## Initialization and Startup

Caller is expected to have complete control over the lifecycle of AudioCapture `HAL` (from open to close). If `HAL` implementation has any run-time dependencies, reconciling them with the actual application that calls `HAL` APIs will be an integration exercise.

## Threading Model

This interface is required to be thread-safe and will be invoked from multiple caller threads. Data callback `RMF_AudioCaptureBufferReadyCb()` must originate in a thread that's separate from caller context(s). Caller will not make any `HAL` calls in the context of
`RMF_AudioCaptureBufferReadyCb()`. However, `RMF_AudioCapture_StatusChangeCb()` will trigger `HAL` `API` calls within that context.

## Process Model

Caller will take care of AudioCapture `HAL` initialization. The interface is expected to support a single instantiation with a single process. The  implementation should not manipulate any process-wide state, such as the current directory, locale, etc. Doing so could have unintended consequences for other threads within the process group.

## Memory Model

AudioCapture `HAL` is responsible for its own memory management. The buffer used for audio data passed by `RMF_AudioCaptureBufferReadyCb()` should be managed
by `HAL` after the callback returns.

## Power Management Requirements

This interface is not required to be involved in any power management funtionality.

## Asynchronous Notification Model

No asynchronous notifications are required.

## Blocking calls

The following callbacks may block depending on the caller's internal operations:

  1. `RMF_AudioCaptureBufferReadyCb()`
  2. `RMF_AudioCapture_StatusChangeCb()`
 
Caller will return as soon as possible from `RMF_AudioCaptureBufferReadyCb()` and `RMF_AudioCapture_StatusChangeCb()`.

## Internal Error Handling

All APIs must return errors synchronously as a return argument. `HAL` is responsible for managing its internal errors.

## Persistence Model

There is no requirement for `HAL` to persist any settings information. The necessary parameters will be passed with `RMF_AudioCapture_Start()` for every audio capture session.

# Non-functional requirements

The following non-functional requirements are required to be supported by this interface:

## Logging and debugging requirements

This interface is required to log all errors, warnings and critical informative messages that are necessary to debug/triage any issues impacting audio capture use cases.

## Memory and performance requirements

This interface is required to use only minimal memory/CPU resources while in closed/stopped state.

## Quality Control

* `HAL` implementation is required to perform static analysis, our preferred tool is Coverity.
* Have a zero-warning policy with regards to compiling. All warnings should be treated as error.
* Use of memory analysis tools like Valgrind are encouraged, to identify leaks/corruption.
* `HAL` tests will endeavour to create worst case scenarios to assist investigations.
* Improvements by any party to the testing suite are required to be fed back.

## Licensing

AudioCapture `HAL` header file is released under Apache 2.0 license. The `HAL` implementation may use any license compatible with the aforementioned header file.

## Build Requirements

This interface is required to build into shared library. The shared library should be named `librmfAudioCapture.so`.

## Variability Management

Any new `API` introduced should be implemented by all the 3rd party modules. Currently there is little to no variability expected across various implementations. If an attempt is made to configure the `HAL` in unsupported ways (via settings argument of `RMF_AudioCapture_Start()` `API`), the function must return error code for invalid parameter.

## Platform or Product Customization

The default settings returned via `RMF_AudioCapture_GetDefaultSettings()` can be configured with parameters that are favourable to `HAL`. Caller will typically not change any of these parameters unless strictly necessary.

# Interface API Documentation

Covered in Doxygen documentation.

## Theory of operation

Caller will configure RMF_AudioCapture interface with the necessary settings and start the capture. `HAL` will deliver audio and status updates
via the registered callbacks in a timely fashion. Calling `RMF_AudioCapture_Open()` is a necessary precondition for the remaining APIs to work. 
### Architecture Diagram

![RMF_AudioCapture Architecture Diagram](images/RMF_AudioCapture_HAL_architecture.png)

Following is a typical sequence of operation:
1. Open RMF_AudioCapture interface using `RMF_AudioCapture_Open()`. This is the cue for `HAL` to acquire the necessary resources.
2. Get default settings using `RMF_AudioCapture_GetDefaultSettings()`. This returns a struct of parameters favourable to the `HAL`. Application may tweak certain members of this struct and pass it with the start call.
2. Start audio capture using `RMF_AudioCapture_Start()`. RMF_AudioCapture `HAL` will continuously deliver audio data to application in real time via callback `RMF_AudioCaptureBufferReadyCb`.
3. When the audio stream is no longer needed, stop audio capture using `RMF_AudioCapture_Stop()`. This will stop the above callbacks.
4. Close RMF_AudioCapture interface using `RMF_AudioCapture_Close()`, releasing all resources.


### Sequence Diagram

![RMF_AudioCapture HAL sequence diagram](images/RMF_AudioCapture_HAL_sequence.png)

### State Diagram

![RMF_AudioCapture HAL state diagram](images/RMF_AudioCapture_HAL_state_machine.png)

