#!python

# This file is part of godot-vive-pro-eye, a Godot Engine 4 driver for the HTC
# Vive Pro Eye eye tracking hardware.
#
# Copyright (c) 2019 Lehrstuhl für Informatik 2,
# Friedrich-Alexander-Universität Erlangen-Nürnberg (FAU)
# Author: Florian Jung (florian.jung@fau.de)
#
# Permission is hereby granted, free of charge, to any person obtaining
# a copy of this software and associated documentation files (the
# "Software"), to deal in the Software without restriction, including
# without limitation the rights to use, copy, modify, merge, publish,
# distribute, sublicense, and/or sell copies of the Software, and to
# permit persons to whom the Software is furnished to do so, subject to
# the following conditions:
#
# The above copyright notice and this permission notice shall be
# included in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
# EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
# MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
# IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
# CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
# TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
# SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

import os, subprocess


env = SConscript("godot-cpp/SConstruct")

# first check whether the SRanipal SDK has been installed correctly.

sranipal_bin_found = os.path.isdir("./thirdparty/vive_sranipal_sdk/bin")
sranipal_inc_found = os.path.isdir("./thirdparty/vive_sranipal_sdk/include")
sranipal_lib_found = os.path.isdir("./thirdparty/vive_sranipal_sdk/lib")

if not (sranipal_bin_found and sranipal_inc_found and sranipal_lib_found):
	print("Error: The SRanipal SDK was not installed correctly. (bin %sfound, include %sfound, lib %sfound)."  % ("" if sranipal_bin_found else "NOT ","" if sranipal_inc_found else "NOT ","" if sranipal_lib_found else "NOT "))
	print("Follow the instructions in SRanipal_SDK/README.txt and try again.")
	quit()


opts = Variables([], ARGUMENTS)

# Define our options
opts.Add(EnumVariable('target', "Compilation target", 'debug', ['d', 'debug', 'r', 'release']))
opts.Add(EnumVariable('platform', "Compilation platform", '', ['', 'windows', 'x11', 'linux', 'osx']))
opts.Add(EnumVariable('p', "Compilation target, alias for 'platform'", '', ['', 'windows', 'x11', 'linux', 'osx']))
opts.Add(BoolVariable('use_llvm', "Use the LLVM / Clang compiler", 'no'))
opts.Add(PathVariable('target_path', 'The path where the lib is installed.', './bin/'))
opts.Add(PathVariable('target_name', 'The library name.', 'libfaceeye', PathVariable.PathAccept))
opts.Add(BoolVariable('use_mingw', "Cross-compile for Windows", 'no'))

# Local dependency paths, adapt them to your setup
godot_headers_path = "godot-cpp/godot-headers/"
cpp_bindings_path = "godot-cpp/"
cpp_library = "libgodot-cpp"

# only support 64 at this time..
bits = 64

# Updates the environment with the option variables.
opts.Update(env)

if env['p'] != '':
    env['platform'] = env['p']

if env['platform'] == '':
    print("No valid target platform selected.")
    quit();

if env['platform'] != 'windows' or bits != 64:
	print("For now, only 64bit windows is supported. Sorry.")
	quit()

# Check our platform specifics
if env['platform'] == "osx":
    env['target_path'] += 'osx/'
    cpp_library += '.osx'
    if env['target'] in ('debug', 'd'):
        env.Append(CCFLAGS = ['-g','-O2', '-arch', 'x86_64'])
        env.Append(LINKFLAGS = ['-arch', 'x86_64'])
    else:
        env.Append(CCFLAGS = ['-g','-O3', '-arch', 'x86_64'])
        env.Append(LINKFLAGS = ['-arch', 'x86_64'])

elif env['platform'] in ('x11', 'linux'):
    # Process some arguments
    if env['use_llvm']:
        env['CC'] = 'clang'
        env['CXX'] = 'clang++'

    env['target_path'] += 'x11/'
    cpp_library += '.linux'
    if env['target'] in ('debug', 'd'):
        env.Append(CCFLAGS = ['-fPIC', '-g3','-Og', '-std=c++17'])
    else:
        env.Append(CCFLAGS = ['-fPIC', '-g','-O3', '-std=c++17'])
elif env['platform'] == "windows" and (env["use_mingw"] or env["use_llvm"]):
    # Don't Clone the environment. Because otherwise, SCons will pick up msvc stuff.
    env = Environment(ENV=os.environ, tools=["mingw"])
    opts.Update(env)
    # env = env.Clone(tools=['mingw']) 
    env.Append(CCFLAGS = ['-std=c++17'])
    env.Append(LINKFLAGS=[ 
        '-static-libgcc',
        '-static-libstdc++',
    ])
    cpp_library += '.windows'
elif env['platform'] == "windows":
    env['target_path'] += 'win64/'
    cpp_library += '.windows'
    # This makes sure to keep the session environment variables on windows,
    # that way you can run scons in a vs 2017 prompt and it will find all the required tools
    env.Append(ENV = os.environ)

    env.Append(CCFLAGS = ['-DWIN32', '-D_WIN32', '-D_WINDOWS', '-W3', '-GR', '-D_CRT_SECURE_NO_WARNINGS', "/std:c++17"])
    if env['target'] in ('debug', 'd'):
        env.Append(CCFLAGS = ['-EHsc', '-Zi', '-D_DEBUG', '-MDd'])
        env.Append(LINKFLAGS = ['/DEBUG'])
    else:
        env.Append(CCFLAGS = ['-O2', '-EHsc', '-DNDEBUG', '-MD'])

# sensible warning flags for g++ and clang++
if env['CXX'].endswith("clang++"):
    print("using clang specific warnflags")
    env.Append(CCFLAGS =
        [
            "-Weverything", "-pedantic",
            "-Wno-c++98-compat", "-Wno-c++98-compat-pedantic", "-Wno-c++98-c++11-compat", "-Wno-padded", "-Wno-exit-time-destructors", "-Wno-global-constructors", "-Wno-weak-vtables", "-Wno-switch-enum", "-Wswitch",
            "-Werror=header-hygiene", "-Werror=return-type", "-Werror=tautological-compare",
            "-fdiagnostics-color=always" # oh come on scons -.-
        ]
    )
elif env['CXX'].endswith("g++"):
    print("using gcc specific warnflags")
    env.Append(CCFLAGS =
        [
            "-Wall", "-Wextra", "-pedantic", "-Wno-unknown-pragmas",

            # https://stackoverflow.com/questions/5088460/flags-to-enable-thorough-and-verbose-g-warnings/9862800#9862800
            "-Wcast-align", "-Wcast-qual", "-Wctor-dtor-privacy", "-Wdisabled-optimization", "-Wformat=2", "-Winit-self", "-Wlogical-op", "-Wmissing-declarations", "-Wmissing-include-dirs", "-Wnoexcept", "-Wold-style-cast", "-Woverloaded-virtual", "-Wredundant-decls", "-Wshadow", "-Wsign-conversion", "-Wsign-promo", "-Wstrict-null-sentinel", "-Wstrict-overflow=5", "-Wswitch-default", "-Wundef", "-Wno-unused",

            # same source, including warnings that are absent:
            "-Wabi", "-Winvalid-pch", "-Wstack-protector", "-Wzero-as-null-pointer-constant", "-Wuseless-cast",
            "-Wno-switch-enum", "-Wno-padded",

            "-Werror=return-type", "-Werror=misleading-indentation", "-Werror=tautological-compare",
            "-fdiagnostics-color=always" # oh come on scons -.-
        ]
    )


if env['target'] in ('debug', 'd'):
    cpp_library += '.debug'
else:
    cpp_library += '.release'

cpp_library += '.' + "x86_64"

# make sure our binding library is properly included
env.Append(CPPPATH=[
	'.',
    'thirdparty',
	godot_headers_path,
	cpp_bindings_path + 'include/',
	cpp_bindings_path + 'include/core/',
	cpp_bindings_path + 'include/gen/',
    cpp_bindings_path + 'gen/include/',
	"vive_sranipal_sdk/include"
])

env.Append(LIBPATH=[
	cpp_bindings_path + 'bin/',
	"vive_sranipal_sdk/lib"
])

env.Append(LIBS=[
	cpp_library,
	"SRanipal"
])

sources = [
    "src/register_types.cpp",
    "src/face_eye.cpp"
]

library = env.SharedLibrary(target=env['target_path'] + env['target_name'] , source=sources)
install_sranipal = env.Install(env['target_path'], ["./thirdparty/vive_sranipal_sdk/bin/%s" % f for f in ["HTC_License", "libHTC_License.dll", "nanomsg.dll", "SRanipal.dll", "SRWorks_Log.dll", "ViveSR_Client.dll"]])

Default([library, install_sranipal])

# Generates help for the -h scons option.
Help(opts.GenerateHelpText(env))
