# OnvifDeviceSetupForIdiots

The task that I needed to do just once was to create a very simple NVIDIA Jetson based device that will stream internally generated MJPEG (not H.264) videos  to ONVIF clients. Since I am an idiot non familiar with this technology and **not** interested in it, I wanted a simple out-of-box solution. Surprisingly, I could not find an integral solution so had to spend days before I got something working.

My main misunderstanding was the architecture of this device. I thought about ONVIF server as all-in-one software. But actually ONVIF server is separate from streaming part. So at least two pieces of software must be up and running on ONVIF server (source) device: one to provide information about server, anothre one to do actual streaming.

## Steps to recreate the solution

1. Get and build the latest available gstreamer1.0, gst-rtsp-server, gst-plugins-base, gst-plugins-good, gst-plugins-bad, gst-libav. Make sure that all packages have exactly the same version. 1.20.1 worked for me. Use wget to fetch
> wget https://gstreamer.freedesktop.org/src/gst-rtsp-server/gst-rtsp-server-1.20.1.tar.xz
> 
> wget https://gstreamer.freedesktop.org/src/gstreamer/gstreamer-1.20.1.tar.xz
> 
> wget https://gstreamer.freedesktop.org/src/gst-libav/gst-libav-1.20.1.tar.xz
> 
> wget https://gstreamer.freedesktop.org/src/gst-plugins-good/gst-plugins-good-1.20.1.tar.xz
> 
> wget https://gstreamer.freedesktop.org/src/gst-plugins-base/gst-plugins-base-1.20.1.tar.xz
> 
> wget https://gstreamer.freedesktop.org/src/gst-plugins-bad/gst-plugins-bad-1.20.1.tar.xz

To build the stuff you will probably need a recent meson (ensure you do not run an ancient version probably residing on your installation):
> wget https://github.com/mesonbuild/meson/releases/download/0.61.4/meson-0.61.4.tar.gz

Use 

>meson builddir
>cd builddir 
>ninja

2. Get, build (see below), install and run https://github.com/KoynovStas/onvif_srvd . This will be a server that responds to ONVIF clients' queries and tells them where to get the required stream. You will probably need autotools-1.16 and its dependencies, and gSOAP 2.8.91 (edit GSOAP_VERSION in Makefile). Do not forget to build with 

> make WSSE_ON=1

otherwise you will get the error in your ONVIF viewer:

**The data in element 'Security' must be understood but cannot be processed**

Before build untar gsoap-2.8.91 into onvif_srvd/gsoap-2.8 and build it with

> cd gsoap-2.8/gsoap/src
> make -f MakefileManual 
> cd gsoap-2.8/gsoap/wsdl  
> make -f MakefileManual 

> sudo ./onvif_srvd --ifs eth0 --scope onvif/www.onvif.org/name/TestDev --scope onvif://www.onvif.org/Profile/S --name RTSP --width 640 --height 480 --url rtsp://%s:8554/unicast --type JPEG --no_fork --port 8080

3. Replace gst-rtsp-server-1.20.1/examples/test-appsrc.c with the file from this repo and build it with ninja. This will be a streaming RTSP server that sends out red and blue screens at 7 fps from port 8554, as onvif_srvd advertizes above.
4. Run 
>GST_DEBUG=4 gst-rtsp-server-1.20.1/builddir/examples/test-appsrc 
5. Download ONVIF Device Manager kindly provided by Synesis from https://sourceforge.net/projects/onvifdm/
6. Run ONVIF Device Manager and point it to http://<your_device_IP>:8080/onvif/device_service
7. Enjoy.

