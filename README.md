```bash
$ cmake -S . -B build && cmake --build build --parallel $(nproc)
# physx engine gets checked out, configures and builds..
$ cd build && ./hello-physx-cmake
Starting PhysX up..
GPU acceleration is available
PhysX set up
Shutting down..
```