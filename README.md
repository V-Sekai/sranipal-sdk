# Godot driver for the HTC Vive Pro Eye equipment

This project provides a Godot driver for the HTC Vive Pro Eye and Facial tracker. It uses the
[SRanipal framework](https://developer.vive.com/resources/knowledgebase/vive-sranipal-sdk/).

## Directory structure

```
./
|-- bin                            Output directory for the binary plugin
|   |-- x11                        which has its sources in native/
|   `-- win64
|
`-- native                         C++ source files for the binary plugin
	|-- godot-cpp                  Godot C++ bindings
	|-- SRanipal_SDK               Placeholder directory for the SRanipal files
	`-- src                        The actual source code
```

## Installation

### Dependencies

**WARNING WORK IN PROGRESS**

**Not working as of 2022-07-03**

Install the **scons** build system.

Download and extract the
[SRanipal SDK](https://developer.vive.com/resources/knowledgebase/vive-sranipal-sdk/)'s
`bin\`, `lib\` and `include\` directories to `native\SRanipal_SDK`. For details, refer
to [the README in that directory](native/SRanipal_SDK/README.txt).

### Preparation

Run `git submodule update --init --recursive` to initialize the submodules.

Then open the Visual Studio Native Tools prompt, and do:

```
cd native/godot-cpp
scons -c      # cleans up any previous build
(path/to/godot --gdnative-generate-json-api godot_api.json # build up-to-date bindings. ONLY NEEDED if you know what you are doing)
scons -j8 p=windows headers=godot_headers generate_bindings=yes bits=64
```

### Building

After having prepared everything, type into a Visual Studio Native Tools prompt:

```
scons p=windows use_mingw=yes use_llvm=yes
```

This should create `bin\win64\libgdexample.dll`

## Running

If the build has succeeded, you can open the project in godot. Either run
`path/to/godot -e`, while being in the repo's main directory, or open the
project from godot's GUI.

You may need to install the SRanipal runtime, which is available from the SDK download
page. You may need to perform a calibration first.

The demo application shows a scene with some objects and three coloured spheres.
These spheres follow your gaze and correspond to your right, left, and combined gaze.
If you close one of the eyes, the corresponding sphere's size gets gradually smaller.

# Documentation

Sorry, currrently, you can only read the comments in the [header file](native/src/myclass.h).

For use in Godot, the method names are the same. Just create a _Node_-object and attach
the _MyClass.gdns_ script to it. Then you can call `ThatObject.update_eye_data()` and
`ThatObject.get_gaze_direction(0)` from your own GDscripts.

# License

The code offered in this repository is licensed under the MIT license. Note, however,
that by linking in the SRanipal SDK, you have to agree to their license agreement,
which may have an impact on the conditions you are allowed to redistribute the resulting
binaries under.
