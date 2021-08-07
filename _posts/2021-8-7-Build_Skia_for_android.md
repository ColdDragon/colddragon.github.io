안드로이드용 skia 엔진 빌드

1. Download
  a. Install depot_tools and Git
    > git clone 'https://chromium.googlesource.com/chromium/tools/depot_tools.git'
    > export PATH="${PWD}/depot_tools:${PATH}" 
  b. Clone the Skia repository
    > git clone https://skia.googlesource.com/skia.git
    > cd skia
    > git fetch
    > python2 tools/git-sync-deps
2. Build
  a. is_official_build and Third-party Dependencies
    > direct build : is_official_build = false
    > is_component_build=true
    > is_debug=true
  b. bin/gn gen out/Shared --args='is_official_build=true is_component_build=true'
  c. Android build argument
    > bin/gn gen out/arm   --args='ndk="/tmp/ndk" target_cpu="arm"'
    > bin/gn gen out/arm64 --args='ndk="/tmp/ndk" target_cpu="arm64"'
    > bin/gn gen out/x64   --args='ndk="/tmp/ndk" target_cpu="x64"'
    > bin/gn gen out/x86   --args='ndk="/tmp/ndk" target_cpu="x86"'
  c. check build arguments(gn/skia.gni)
    > bin/gn args out/Debug --list
  d. ninja -C out/Static
