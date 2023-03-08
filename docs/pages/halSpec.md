@mainpage

# RMF_AudioCapture HAL Documentation

# Version and Version History


1.0.0 Initial Revision covers existing RMF_AudioCapture HAL implementation.

# Description
RMF AudioCapture is the RDK-V interface designed to tap into the audio output of an STB/TV. When in use, the underlying
SoC implementation will deliver a constant stream of raw audio data (PCM). This audio will have the active audio track
of whatever content is being played, and may include other sounds mixed in, such as voice-guidance, application-driven
audio overlays (eg: beeps) etc. RDK will attempt to configure audio format parameters (channels, bit rate etc.) to its liking
but not all combinations are expected to be supported by the HAL.

![RMF_AudioCapture Architecture Diagram](RMF_AudioCapture_HAL_architecture.png)

The actual application that uses RMF_AudioCapture HAL is called Audio Capture Manager (ACM) (process name: audiocapturemgr).
ACM leverages audio capture capabilities to support multiple use cases:
1. Music identification
2. Bluetooth audio output (in conjunction with RDK-V's Bluetooth manager component)
3. It is possible to create other applications using ACM's public APIs that can take advantage of real-time audio streams.
 
# Component Runtime Execution Requirements
Audio Capture Manager has runtime dependencies on IARMBus and will not launch until this bus is up. If RMF_AudioCapture HAL as any
run-time requirements, they need to be added as dependencies of Audio Capture Manager as well. For eg., if RMF_AudioCapture HAL requires
any background service to be running or any files/pipes to be created to work, the necessary interlocks must be added to Audio Capture Manager.
Most such requirements can be met by tweaking the systemd service file used by audiocapturemgr service.

## Theory of operation

RMF_AudioCapture HAL API is initialized by Audio Capture Manager application. ACM has complete control over the lifecycle of RMF_AudioCapture HAL.
Following is a typical sequence of operation by ACM:
1. Open RMF_AudioCapture interface using RMF_AudioCapture_Open(). This is the cue for HAL to acquire the necessary resources.
2. Get default settings using RMF_AudioCapture_GetDefaultSettings(). This returns a struct of parameters favourable to the HAL. ACM will tweak certain members of this struct and pass it with the start() call.
2. Start audio capture using RMF_AudioCapture_Start. RMF_AudioCapture HAL will continuously deliver audio data to ACM in real time via callback RMF_AudioCaptureBufferReadyCb.
3. When ACM no longer needs the audio stream, stop audio capture using RMF_AudioCapture_Stop(). This will stop the above callbacks.
4. Close RMF_AudioCapture interface using RMF_AudioCapture_Close(), releasing all resources.

Calling RMF_AudioCapture_Open() is a necessary precondition for the remaining APIs to work. 
#### Sequence Diagram

![RMF_AudioCapture HAL sequence diagram](RMF_AudioCapture_HAL_sequence.png)

#### State Diagram

![RMF_AudioCapture HAL state diagram](RMF_AudioCapture_HAL_state_machine.png)
## Threading Model

Audio Capture Manager is a multi-threaded application and RMF_AudioCapture HAL is expected to be thread-safe when invoked from various ACM threads. Data callback RMF_AudioCaptureBufferReadyCb must originate in a thread that's separate from ACM context(s). ACM does not make any HAL calls in the context of
RMF_AudioCaptureBufferReadyCb. However, there is a possibility that RMF_AudioCapture_StatusChangeCb will trigger HAL API calls within that context.

## Process Model

Audio Capture Manager application takes care of RMF_AudioCapture HAL initialization. At any point of time, only a single instance of audiocapturemgr application is expected to run. The HAL implementation should not manipulate any process-wide state, such as the current directory, locale, etc. Doing so could have unintended consequences for other threads within the process group.

## Memory Model

RMF_AudioCapture HAL is responsible for its own memory management. The buffer used for audio data passed by RMF_AudioCaptureBufferReadyCb should be managed
by HAL after the callback returns.

## Power Management Requirements

RMF_AudioCapture HAL does not affect any power management functions. In controlled power transitions, ACM may choose to stop and/or close the audio capture
interface when device transitions from ON to standby power states. However, this choice is dictated by ACM's consumer and should not be relied upon. Should the
device transition back to ON state, RMF_AudioCapture HAL is expected to resume normal operation, i. e., if it was actively captureing audio when power state changed to standby, it must resume doing so.

## Asynchronous Notification Model
No asynchronous notifications are supported.

## Blocking calls

The following callbacks may block depending on Audio Capture Manager's internal operations.
  1. RMF_AudioCaptureBufferReadyCb
  2. RMF_AudioCapture_StatusChangeCb
ACM takes pains to return as soon as possible from RMF_AudioCaptureBufferReadyCb callback, as any delays inside this function has the potential to
impact audio experience. RMF_AudioCapture_StatusChangeCb has no such guarantees.

## Internal Error Handling
All HAL API's should return error synchronously as a return argument where possible. HAL is responsible for managing its internal errors.

## Persistence Model
There is no requirement for HAL to persist any setting information. It is application's responsibility to configure the audio capture interface to its
liking every time.


# Nonfunctional requirements
Following non functional requirement should be supported by the RMF_AudioCapture HAL component:

## Logging and debugging requirements

RMF_AudioCapture HAL component must log all errors, warnings and critical informative messages that are necessary to debug/triage any issues impacting audio capture use cases.

## Memory and performance requirements

RMF_AudioCapture HAL must use only minimal memory/CPU resources while in closed state. RMF_AudioCapture HAL must use only minimal CPU resources while in stopped stage.


## Quality Control

HAL implementation should pass Coverity, Black duck scan, Valgrind checks without any issue. HAL must take care not to introduce any memory leaks or corruption.
HAL implementation should pass tests available at: https://gerrit.teamccp.com/plugins/gitiles/rdk/components/generic/tdk/generic/+/stable2/framework/web-app/fileStore/testscriptsRDKV/component/audiocapturemgr/

## Licensing

RMF_AudioCapture HAL implementation is expected to be released under the Apache License. 

## Build Requirements

HAL source code should be built in Linux Yocto environment and should be delivered as a shared library.
  

## Variability Management

Any new API introduced should be implemented by all the 3rd party modules and RDK generic code should be compatible with specific version of RMF_AudioCapture HAL. Currently there is little to no variability in RDK_AudioCapture HAL capabilties across implementations. If an attempt is made to configure the HAL in unsupported ways (via settings argument RMF_AudioCapture_Start() API), the function must return error code for invalid parameter.

## Platform or Product Customization
The default settings returned via RMF_AudioCapture_GetDefaultSettings() can be configured with parameters that are favourable to HAL. RDK will typically not
change any of these parameters unless strictly necessary.

# Interface API Documentation

Covered as per Doxygen documentation.

## Theory of operation and key concepts

Covered as per "Description" section.

### UML Diagrams

