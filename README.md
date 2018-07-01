# module\_cosmicweb

App for viewing cosmicweb particle data.

## Building

Clone the repo into your OSPRay modules directory, then run CMake to build
OSPRay and pass `-DOSPRAY_MODULE_COSMICWEB=ON` to build the module's core library,
`-DOSPRAY_MODULE_COSMICWEB_WORKER=ON` to build the render workers and 
`-DOSPRAY_MODULE_COSMICWEB_VIEWER=ON` to build the remote viewer client. 
GLFW 3+ and TurboJPEG 1.5.x+ are required, along with MPI. 
You can pass `-DTURBOJPEG_DIR` to the root directory of your
TurboJPEG installation directory if TurboJpeg is not installed in a standard 
location.

## Running the Remote Viewer

To run the remote viewer first start the render workers on your compute nodes.

```bash
mpirun -np <N> ./cosmicweb_worker \
       -port <port-to-listen-on>  \
       -cosmicweb <N-datasets>
```

These workers will start and load the data. Once they're ready to connect to
with the viewer, rank 0 will print out "Rank 0 now listening for client". You
can then start the viewer and pass it the hostname of rank 0 and the port
to connect to.

```bash
./cosmicweb_viewer -server <rank 0 hostname> -port <port to connect>
```

If the viewer cannot be connected to the worker, then try to create one ssh 
tunnel first.

```bash
ssh user@remote.server.com -L <local-port>:localhost:<remote-port> -N
```
