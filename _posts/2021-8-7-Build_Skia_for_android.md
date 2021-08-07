## 안드로이드용 skia 엔진 빌드

- Download
  1. Install depot_tools and Git
	  -  `bash-3.2$ git clone 'https://chromium.googlesource.com/chromium/tools/depot_tools.git'`
	  - `bash-3.2$ export PATH="${PWD}/depot_tools:${PATH}"` 
  2. Clone the Skia repository
	  - `bash-3.2$ git clone https://skia.googlesource.com/skia.git`
	  - `bash-3.2$ cd skia`
	  - `bash-3.2$ git fetch`
	  - `bash-3.2$ python2 tools/git-sync-deps`
- Build
  1. is_official_build and Third-party Dependencies
	 - direct build : is_official_build = false
	 - is_component_build=true
	 - is_debug=true
  2. create shared library
	  - `bash-3.2$ bin/gn gen out/Shared --args='is_official_build=true is_component_build=true'`
  3. Android build argument
	  - `bash-3.2$ bin/gn gen out/arm   --args='ndk="/tmp/ndk" target_cpu="arm"'`
	  - `bash-3.2$ bin/gn gen out/arm64 --args='ndk="/tmp/ndk" target_cpu="arm64"'`
	  - `bash-3.2$ bin/gn gen out/x64   --args='ndk="/tmp/ndk" target_cpu="x64"'`
	  - `bash-3.2$ bin/gn gen out/x86   --args='ndk="/tmp/ndk" target_cpu="x86"'`
  4. check build arguments(gn/skia.gni)
	  - `bash-3.2$ bin/gn args out/Debug --list`
  5. build
	  - `bash-3.2$ ninja -C out/Static`
