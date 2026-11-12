# Napari-SpatialData on Open OnDemand (Selkies streaming)

An Open OnDemand Batch Connect app that runs napari with the
[napari-spatialdata](https://github.com/scverse/napari-spatialdata) plugin in
an Apptainer container on a compute node, and streams it to the browser as a
single application window. It follows the same design as the [QuPath app](https://github.com/kir-rescomp/ood-app-qupath
: the stream is produced by [Selkies](https://github.com/selkies-project/selkies)
(H.264 over a WebSocket) and passes through the portal's `/rnode` reverse
proxy without additional ports or services.


## How a session runs

1. `before.sh` chooses a free port and creates the session token (OnDemand's
   `password`, carried by the "Open napari" link) and a master token used only
   to provision it.
2. `script.sh` starts `apptainer exec --cleanenv <sif> selkies-session`. Inside
   the container, `selkies-session` starts an Xvfb display, runs
   `napari-session` on it, and starts the Selkies server on the chosen port.
3. `after.sh` waits for the port and registers the session token; only then is
   the session reported as running. Connections without the token are refused.
4. `napari-session` starts Openbox and `napari-spatialdata-start`, which opens
   the selected SpatialData store in the same way as
   `python -m napari_spatialdata view <store>`, or an empty viewer when no
   store is given. The main window is maximised and kept fitted to the browser
   window.
5. When napari exits, the launcher writes `napari.exited` in the session
   directory; `script.sh` stops the container and the job ends.

