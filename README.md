# ESPAsyncWebServer 
[![Build Status](https://travis-ci.org/me-no-dev/ESPAsyncWebServer.svg?branch=master)](https://travis-ci.org/me-no-dev/ESPAsyncWebServer) ![](https://github.com/me-no-dev/ESPAsyncWebServer/workflows/ESP%20Async%20Web%20Server%20CI/badge.svg) [![Codacy Badge](https://api.codacy.com/project/badge/Grade/395dd42cfc674e6ca2e326af3af80ffc)](https://www.codacy.com/manual/me-no-dev/ESPAsyncWebServer?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=me-no-dev/ESPAsyncWebServer&amp;utm_campaign=Badge_Grade)

A fork of the [ESPAsyncWebServer](https://github.com/ESP32Async/ESPAsyncWebServer) library by [@ESP32Async](https://github.com/ESP32Async). Within this fork, I simply added a reference to the current AsyncWebServerRequest instance to the onDisconnect handler of AsyncWebServerRequest object. I needed this tiny addon to be able to use the _tempObject pointer provided by AsyncWebServerRequest to store an object-instance created by the new operator. By default, ESPAsyncWebServer handles freeing _tempObject in all cases by calling free(), but i needed to properly destroy my instance with the delete operator. In case the incoming request gets handled all the way to the end, I can for sure do that in the onRequest handler right before or after sending my responste to the client. But this changes once the client suddenly disconnects during processing of his request. In case of a file upload (which may take a few seconds), the onUpload handler is stopped and the onRequest handler will never get executed, thereby the destruction of my object will never happen. Luckily there is one thing that will always get executed when a client disconnects: the onDisconnect handler. Inside this handler, i now have a reference to the current request and can get hold of _tempObject again and properly destruct my object.

For help and support [![Join the chat at https://gitter.im/me-no-dev/ESPAsyncWebServer](https://badges.gitter.im/me-no-dev/ESPAsyncWebServer.svg)](https://gitter.im/me-no-dev/ESPAsyncWebServer?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Async HTTP and WebSocket Server for ESP8266 Arduino

For ESP8266 it requires [ESPAsyncTCP](https://github.com/me-no-dev/ESPAsyncTCP)
To use this library you might need to have the latest git versions of [ESP8266](https://github.com/esp8266/Arduino) Arduino Core

For ESP32 it requires [AsyncTCP](https://github.com/me-no-dev/AsyncTCP) to work
To use this library you might need to have the latest git versions of [ESP32](https://github.com/espressif/arduino-esp32) Arduino Core

## Table of contents
- [ESPAsyncWebServer](#espasyncwebserver)
  - [Table of contents](#table-of-contents)
  - [Installation](#installation)
    - [Using PlatformIO](#using-platformio)
  - [Why should you care](#why-should-you-care)
  - [Important things to remember](#important-things-to-remember)
  - [Principles of operation](#principles-of-operation)
    - [The Async Web server](#the-async-web-server)
    - [Request Life Cycle](#request-life-cycle)
    - [Rewrites and how do they work](#rewrites-and-how-do-they-work)
    - [Handlers and how do they work](#handlers-and-how-do-they-work)
    - [Responses and how do they work](#responses-and-how-do-they-work)
    - [Template processing](#template-processing)
  - [Libraries and projects that use AsyncWebServer](#libraries-and-projects-that-use-asyncwebserver)
  - [Request Variables](#request-variables)
    - [Common Variables](#common-variables)
    - [Headers](#headers)
    - [GET, POST and FILE parameters](#get-post-and-file-parameters)
    - [FILE Upload handling](#file-upload-handling)
    - [Body data handling](#body-data-handling)
    - [JSON body handling with ArduinoJson](#json-body-handling-with-arduinojson)
  - [Responses](#responses)
    - [Redirect to another URL](#redirect-to-another-url)
    - [Basic response with HTTP Code](#basic-response-with-http-code)
    - [Basic response with HTTP Code and extra headers](#basic-response-with-http-code-and-extra-headers)
    - [Basic response with string content](#basic-response-with-string-content)
    - [Basic response with string content and extra headers](#basic-response-with-string-content-and-extra-headers)
    - [Send large webpage from PROGMEM](#send-large-webpage-from-progmem)
    - [Send large webpage from PROGMEM and extra headers](#send-large-webpage-from-progmem-and-extra-headers)
    - [Send large webpage from PROGMEM containing templates](#send-large-webpage-from-progmem-containing-templates)
    - [Send large webpage from PROGMEM containing templates and extra headers](#send-large-webpage-from-progmem-containing-templates-and-extra-headers)
    - [Send binary content from PROGMEM](#send-binary-content-from-progmem)
    - [Respond with content coming from a Stream](#respond-with-content-coming-from-a-stream)
    - [Respond with content coming from a Stream and extra headers](#respond-with-content-coming-from-a-stream-and-extra-headers)
    - [Respond with content coming from a Stream containing templates](#respond-with-content-coming-from-a-stream-containing-templates)
    - [Respond with content coming from a Stream containing templates and extra headers](#respond-with-content-coming-from-a-stream-containing-templates-and-extra-headers)
    - [Respond with content coming from a File](#respond-with-content-coming-from-a-file)
    - [Respond with content coming from a File and extra headers](#respond-with-content-coming-from-a-file-and-extra-headers)
    - [Respond with content coming from a File containing templates](#respond-with-content-coming-from-a-file-containing-templates)
    - [Respond with content using a callback](#respond-with-content-using-a-callback)
    - [Respond with content using a callback and extra headers](#respond-with-content-using-a-callback-and-extra-headers)
    - [Respond with content using a callback containing templates](#respond-with-content-using-a-callback-containing-templates)
    - [Respond with content using a callback containing templates and extra headers](#respond-with-content-using-a-callback-containing-templates-and-extra-headers)
    - [Chunked Response](#chunked-response)
    - [Chunked Response containing templates](#chunked-response-containing-templates)
    - [Print to response](#print-to-response)
    - [ArduinoJson Basic Response](#arduinojson-basic-response)
    - [ArduinoJson Advanced Response](#arduinojson-advanced-response)
  - [Serving static files](#serving-static-files)
    - [Serving specific file by name](#serving-specific-file-by-name)
    - [Serving files in directory](#serving-files-in-directory)
    - [Serving static files with authentication](#serving-static-files-with-authentication)
    - [Specifying Cache-Control header](#specifying-cache-control-header)
    - [Specifying Date-Modified header](#specifying-date-modified-header)
    - [Specifying Template Processor callback](#specifying-template-processor-callback)
  - [Param Rewrite With Matching](#param-rewrite-with-matching)
  - [Using filters](#using-filters)
    - [Serve different site files in AP mode](#serve-different-site-files-in-ap-mode)
    - [Rewrite to different index on AP](#rewrite-to-different-index-on-ap)
    - [Serving different hosts](#serving-different-hosts)
    - [Determine interface inside callbacks](#determine-interface-inside-callbacks)
  - [Bad Responses](#bad-responses)
    - [Respond with content using a callback without content length to HTTP/1.0 clients](#respond-with-content-using-a-callback-without-content-length-to-http10-clients)
  - [Async WebSocket Plugin](#async-websocket-plugin)
    - [Async WebSocket Event](#async-websocket-event)
    - [Methods for sending data to a socket client](#methods-for-sending-data-to-a-socket-client)
    - [Direct access to web socket message buffer](#direct-access-to-web-socket-message-buffer)
    - [Limiting the number of web socket clients](#limiting-the-number-of-web-socket-clients)
  - [Async Event Source Plugin](#async-event-source-plugin)
    - [Setup Event Source on the server](#setup-event-source-on-the-server)
    - [Setup Event Source in the browser](#setup-event-source-in-the-browser)
  - [Scanning for available WiFi Networks](#scanning-for-available-wifi-networks)
  - [Remove handlers and rewrites](#remove-handlers-and-rewrites)
  - [Setting up the server](#setting-up-the-server)
    - [Setup global and class functions as request handlers](#setup-global-and-class-functions-as-request-handlers)
    - [Methods for controlling websocket connections](#methods-for-controlling-websocket-connections)
    - [Adding Default Headers](#adding-default-headers)
    - [Path variable](#path-variable)

## Installation

### Using PlatformIO

[PlatformIO](http://platformio.org) is an open source ecosystem for IoT development with cross platform build system, library manager and full support for Espressif ESP8266/ESP32 development. It works on the popular host OS: Mac OS X, Windows, Linux 32/64, Linux ARM (like Raspberry Pi, BeagleBone, CubieBoard).

1. Install [PlatformIO IDE](http://platformio.org/platformio-ide)
2. Create new project using "PlatformIO Home > New Project"
3. Update dev/platform to staging version:
   - [Instruction for Espressif 8266](http://docs.platformio.org/en/latest/platforms/espressif8266.html#using-arduino-framework-with-staging-version)
   - [Instruction for Espressif 32](http://docs.platformio.org/en/latest/platforms/espressif32.html#using-arduino-framework-with-staging-version)
 4. Add "ESP Async WebServer" to project using [Project Configuration File `platformio.ini`](http://docs.platformio.org/page/projectconf.html) and [lib_deps](http://docs.platformio.org/page/projectconf/section_env_library.html#lib-deps) option:

```ini
[env:stable]
platform = https://github.com/pioarduino/platform-espressif32/releases/download/stable/platform-espressif32.zip
lib_compat_mode = strict
lib_ldf_mode = chain
lib_deps =
  ESP32Async/AsyncTCP
  ESP32Async/ESPAsyncWebServer
```

### ESP8266 / pioarduino

```ini
[env:stable]
platform = espressif8266
lib_compat_mode = strict
lib_ldf_mode = chain
lib_deps =
  ESP32Async/ESPAsyncTCP
  ESP32Async/ESPAsyncWebServer
```

### Unofficial dependencies

**AsyncTCPSock**

AsyncTCPSock can be used instead of AsyncTCP by excluding AsyncTCP from the library dependencies and adding AsyncTCPSock instead:

```ini
lib_compat_mode = strict
lib_ldf_mode = chain
lib_deps =
  https://github.com/ESP32Async/AsyncTCPSock/archive/refs/tags/v1.0.3-dev.zip
  ESP32Async/ESPAsyncWebServer
lib_ignore =
  AsyncTCP
  ESP32Async/AsyncTCP
```

**RPAsyncTCP**

RPAsyncTCP replaces AsyncTCP to provide support for RP2040(+WiFi) and RP2350(+WiFi) boards. For example - Raspberry Pi Pico W and Raspberry Pi Pico 2W.

```ini
lib_compat_mode = strict
lib_ldf_mode = chain
platform = https://github.com/maxgerhardt/platform-raspberrypi.git
board = rpipicow
board_build.core = earlephilhower
lib_deps =
  ayushsharma82/RPAsyncTCP@^1.3.1
  ESP32Async/ESPAsyncWebServer
lib_ignore =
  lwIP_ESPHost
build_flags = ${env.build_flags}
  -Wno-missing-field-initializers
```

## Important recommendations for build options

Most of the crashes are caused by improper use or configuration of the AsyncTCP library used for the project.
Here are some recommendations to avoid them and build-time flags you can change.

`CONFIG_ASYNC_TCP_MAX_ACK_TIME` - defines a timeout for TCP connection to be considered alive when waiting for data.
In some bad network conditions you might consider increasing it.

`CONFIG_ASYNC_TCP_QUEUE_SIZE` - defines the length of the queue for events related to connections handling.
Both the server and AsyncTCP library were optimized to control the queue automatically. Do NOT try blindly increasing the queue size, it does not help you in a way you might think it is. If you receive debug messages about queue throttling, try to optimize your server callbacks code to execute as fast as possible.
Read #165 thread, it might give you some hints.

`CONFIG_ASYNC_TCP_RUNNING_CORE` - CPU core thread affinity that runs the queue events handling and executes server callbacks. Default is ANY core, so it means that for dualcore SoCs both cores could handle server activities. If your server's code is too heavy and unoptimized or you see that sometimes
server might affect other network activities, you might consider to bind it to the same core that runs Arduino code (1) to minimize affect on radio part. Otherwise you can leave the default to let RTOS decide where to run the thread based on priority

`CONFIG_ASYNC_TCP_STACK_SIZE` - stack size for the thread that runs sever events and callbacks. Default is 16k that is a way too much waste for well-defined short async code or simple static file handling. You might want to cosider reducing it to 4-8k to same RAM usage. If you do not know what this is or not sure about your callback code demands - leave it as default, should be enough even for very hungry callbacks in most cases.

> [!NOTE]
> This relates to ESP32 only, ESP8266 uses different ESPAsyncTCP lib that does not has this build options

I personally use the following configuration in my projects:

```c++
  -D CONFIG_ASYNC_TCP_MAX_ACK_TIME=5000   // (keep default)
  -D CONFIG_ASYNC_TCP_PRIORITY=10         // (keep default)
  -D CONFIG_ASYNC_TCP_QUEUE_SIZE=64       // (keep default)
  -D CONFIG_ASYNC_TCP_RUNNING_CORE=1      // force async_tcp task to be on same core as Arduino app (default is any core)
  -D CONFIG_ASYNC_TCP_STACK_SIZE=4096     // reduce the stack size (default is 16K)
```

If you need to serve chunk requests with a really low buffer (which should be avoided), you can set `-D ASYNCWEBSERVER_USE_CHUNK_INFLIGHT=0` to disable the in-flight control.
