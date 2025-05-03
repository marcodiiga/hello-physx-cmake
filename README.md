```bash
$ cmake -S . -B build && cmake --build build --parallel $(nproc)
# with install
# cmake -S . -B build && cmake --build build --parallel $(nproc) && cmake --install build --prefix ./install_test
# on windows
#  cmake -S . -B build -DCMAKE_BUILD_TYPE=Release;  cmake --build build --config Release; cmake --install build --prefix ./install_test
# physx engine gets checked out, configures and builds..
$ cd build && ./hello-physx-cmake
Starting PhysX up..
GPU acceleration is available
PhysX set up
Shutting down..
```