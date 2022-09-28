+++
title = "Mediapipe C++ library"
date = ""
showthedate = false
author = ""
cover = ""
tags = []
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++

Mediapipe is a project by Google, it provides solutions for different types of tracking.
This is basically a c++ library which exposes functions to utilize that library.

The main purpose is to simplify the usage of mediapipe by abstracting a lot of stuff and to be able to use mediapipe in other projects which may use different build systems. With this it is also possible to build plugins/modules for game engines easily allowing use of tracking features inside games/applications built using them.

For now, this has only been built and tested in Linux, more specifically Manjaro KDE ( Though the build and usage process is same for every distribution). Windows should also work and a lot of things should directly translate with the only major difference being the file extension from .so ( Linux ) to .dll ( Windows ) but it hasn't yet been tested in it.


## Getting started:

### Building manually:

First clone the repository or download zip from github:

```
git clone https://github.com/purgeme/mediapipe_cpp_lib
```

Next, run the setup script:

```
./setup.sh
```

Next, run the built script:

```
./build.sh
```

### Download models and graphs

Run download_models.sh and extract_graphs.sh to download model and graph files in mediapipe_models and mediapipe_graphs directories respectively.

If familier with mediapipe and its graphs, you can also build and use your own graphs.


### Using the library:

The library uses model files from mediapipe_models folder so it should be present in the parent folder from where gmod_api.h is being used.

To use the library in any C++ project, add the library to system or add path of library to LD_LIBRARY_PATH environment variable. Also, copy and import gmod_api.h, this provides all functions exposed in the library.

The data is returned as a void pointer ( this was done for performance reasons as otherwise it would require copying all data to a C++ array in the library ). The pointer needs to type-casted to required data type. These data types are imported from header files generated by mediapipe while compiling and are not provided directly.

These files are automatically copied after compilling the library into import_files directory. Since these files were compiled by mediapipe with specific version protobuf, make sure you have the same version installed on system or download it and add it to include directories while compiling your project.

Current version of protobuf: 3.19.1

Download link: https://github.com/protocolbuffers/protobuf/releases/download/v3.19.1/protobuf-cpp-3.19.1.zip

( It's a lot to do to just use the library, but a lot of these things will be automated later. )

#### Example:

Here we track face landmarks. A demo example is also provided in demo directory.

The landmarks in mediapipe are stored in NormalizedLandmarkList data type. First we include it:

```
#include "mediapipe/framework/formats/landmark.pb.h"
```

Create IGMOD object. We will use this object to do the rest.

```
IGMOD* test = CreateGMOD();
```

Set camera properties.

```
test->set_camera_props(1, 640, 480, 30);
test->set_camera(true);
test->set_overlay(false);
```

Create an observer for tracking.

```
auto obs = test->create_observer("face_landmarks");
```

Add callback functions. It is important to add callback functions for both presence and packet, even if they are empty.

```
obs->SetPresenceCallback([](class IObserver* observer, bool present){});
obs->SetPacketCallback([](class IObserver* observer){ 
    const mediapipe::NormalizedLandmarkList* data = (mediapipe::NormalizedLandmarkList*)(observer->GetData()); 
    size_t message_type = observer->GetMessageType(); 
    std::cout << typeid(message_type).name() << std::endl;
    std::cout << data->landmark(0).x() << " ; " << message_type << std::endl; 
});
```

Start tracking. Here provide the graph file for tracking.

```
test->start("mediapipe_graphs/holistic_tracking_cpu.pbtxt");
```

To stop tracking properly:

```
test->stop();
```

## Functions:

All the exposed functions and classes are present in gmod_api.h file.

These are the function required for using the library and what they do:

- **CreateGMOD():** Returns an object to be stored in variable of IGMOD type. This object is the main access point to all the functions in the library.

- **start(char\* filename):** This function takes path to a mediapipe graph file ( .pbtxt ) and starts tracking for that graph. The tracking is started in a new thread so as to not interrupt the main thread because it runs in a while loop.

- **stop():** This function stops the tracking and kills the thread.

- **create_observer(const char\* stream_name):** Creates, adds and returns an observer object which then observers data on the stream_name given to it.

- **set_camera_props(int cam_id, int cam_resx, int cam_resy, int cam_fps):** Takes and sets properties for the input webcam.

- **get_camera():** Gets if the camera view is enabled.

- **set_camera():** Sets the camera view should be enabled.

- **get_overlay():** Gets if the overlay view is enabled.

- **set_overlay():** Sets the overlay view should be enabled.

- **SetPresenceCallback(std::function<void(class IObserver\*, bool)> in_presence_callback):** Takes a function pointer as input, this function will be run when the presence packet for the observers stream is recieved.

- **SetPacketCallback(std::function<void(class IObserver\*)> in_packet_callback):** Takes a function pointer as input, this function will be run when the data packet for the observers stream is recieved.

- **GetData():** Gets the data for the observer's stream as void pointer which can then be casted into the required data type.


## TODO:

- [ ] Improve documentation, make it clearer.
- [ ] Create Releases.
- [ ] Clean code.
- [ ] Document data types and examples for each solution.
- [ ] Test on windows.
- [ ] GPU support.