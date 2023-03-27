@mainpage

# RMF_AudioCapture HAL Documentation

## Version and Version History

| Date | Author | Comment | Version |
| --- | --------- | --- | --- |
| 27/03/23 | Chaithen Sojan | First Release | 1.0.0 |

## Acronyms

1. API - Application Programming Interface
1. HAL - Hardware Abstraction layer
1. PCM - Pulse Code Modulation
1. RDK - Reference Development Kit
1. RMF - RDK Media Framework
1. STB - Set Top Box

## Description
RMF AudioCapture is an interface designed to tap into the audio output of an STB/TV. When in use, the underlying
HAL implementation must deliver a constant stream of raw audio data (PCM). This audio will have the active audio track
of whatever content is being played, and may include other sounds mixed in, such as voice-guidance, application-driven
audio overlays (eg: beeps) etc. The audio data delivered via this interface is expected to track as closely as possible, i. e., minimal latency,
to the audio that's being rendered by the device at any point of time.


# Component Runtime Execution Requirements
These requirements ensure that the component implementing the interface, executes correctly within the run-time environment that it will be used.
## Initialization and Startup
Caller is expected to have complete control over the lifecycle of RMF AudioCapture HAL (from open to close). If HAL implementation has any run-time dependencies, reconciling them with the actual application that calls HAL APIs will be an integration exercise.

## Threading Model

RMF_AudioCapture HAL is expected to be thread-safe when invoked from various caller threads. Data callback RMF_AudioCaptureBufferReadyCb must originate in a thread that's separate from caller context(s). Caller does not make any HAL calls in the context of
RMF_AudioCaptureBufferReadyCb. However, there is a possibility that RMF_AudioCapture_StatusChangeCb will trigger HAL API calls within that context.

## Process Model

Caller will take care of RMF_AudioCapture HAL initialization. At any point of time, only a single instance of application using this HAL is expected to run. The HAL implementation should not manipulate any process-wide state, such as the current directory, locale, etc. Doing so could have unintended consequences for other threads within the process group.

## Memory Model

RMF_AudioCapture HAL is responsible for its own memory management. The buffer used for audio data passed by RMF_AudioCaptureBufferReadyCb should be managed
by HAL after the callback returns.

## Power Management Requirements

RMF_AudioCapture HAL does not affect any power management functions. In controlled power transitions, caller may choose to stop and/or close the audio capture
interface when device transitions from ON to standby power states. However, this choice is dictated by application's internal logic and should not be relied upon. Should the
device transition back to ON state, RMF_AudioCapture HAL is expected to resume normal operation, i. e., if it was actively capturing audio when power state changed to standby, it must resume doing so.

## Asynchronous Notification Model
No asynchronous notifications are supported.

## Blocking calls

The following callbacks may block depending on application's internal operations.
  1. RMF_AudioCaptureBufferReadyCb
  2. RMF_AudioCapture_StatusChangeCb
Caller will take pains to return as soon as possible from RMF_AudioCaptureBufferReadyCb callback. The callback RMF_AudioCapture_StatusChangeCb offers no such guarantees.

## Internal Error Handling
All HAL API's should return error synchronously as a return argument where possible. HAL is responsible for managing its internal errors.

## Persistence Model
There is no requirement for HAL to persist any settings information. The necessary parameters will be passed with RMF_AudioCapture_Start() for every audio capture session.


# Nonfunctional requirements


## Logging and debugging requirements

RMF_AudioCapture HAL component must log all errors, warnings and critical informative messages that are necessary to debug/triage any issues impacting audio capture use cases.

## Memory and performance requirements

RMF_AudioCapture HAL must use only minimal memory/CPU resources while in closed state. RMF_AudioCapture HAL must use only minimal CPU resources while in stopped stage.


## Quality Control

HAL implementation should perform static analysis, our preferred tool is Coverity.
* Have a zero-warning policy with regards to compiling. All warnings should be treated as error.
* Use of memory analysis tools like Valgrind are encouraged, to identify leaks/corruption.
* HAL tests will endeavour to create worst case scenarios to assist investigations.

## Licensing

RMF_AudioCapture HAL header file is released under Apache 2.0 license. The HAL implementation may use any license compatible with the aforementioned header file.

## Build Requirements

HAL source code should be built in Linux Yocto environment and should be delivered as a shared library.
  

## Variability Management

Any new API introduced should be implemented by all the 3rd party modules. Currently there is little to no variability expected across various implementations. If an attempt is made to configure the HAL in unsupported ways (via settings argument RMF_AudioCapture_Start() API), the function must return error code for invalid parameter.

## Platform or Product Customization
The default settings returned via RMF_AudioCapture_GetDefaultSettings() can be configured with parameters that are favourable to HAL. Caller will typically not change any of these parameters unless strictly necessary.

# Interface API Documentation
Covered in Doxygen documentation.

## Theory of operation
Caller will configure RMF_AudioCapture interface with the necessary settings and start the capture. HAL will deliver audio and status updates
via the registered callbacks in a timely fashion. 
#### Architecture Diagram
![RMF_AudioCapture Architecture Diagram](images/RMF_AudioCapture_HAL_architecture.png)

Following is a typical sequence of operation:
1. Open RMF_AudioCapture interface using RMF_AudioCapture_Open(). This is the cue for HAL to acquire the necessary resources.
2. Get default settings using RMF_AudioCapture_GetDefaultSettings(). This returns a struct of parameters favourable to the HAL. Application may tweak certain members of this struct and pass it with the start() call.
2. Start audio capture using RMF_AudioCapture_Start. RMF_AudioCapture HAL will continuously deliver audio data to application in real time via callback RMF_AudioCaptureBufferReadyCb.
3. When the audio stream is no longer needed, stop audio capture using RMF_AudioCapture_Stop(). This will stop the above callbacks.
4. Close RMF_AudioCapture interface using RMF_AudioCapture_Close(), releasing all resources.
Calling RMF_AudioCapture_Open() is a necessary precondition for the remaining APIs to work. 

#### Sequence Diagram

![RMF_AudioCapture HAL sequence diagram](images/RMF_AudioCapture_HAL_sequence.png)

#### State Diagram

![RMF_AudioCapture HAL state diagram](images/RMF_AudioCapture_HAL_state_machine.png)

