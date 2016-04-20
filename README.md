# ireactor-win32-kmod
the base repository for linux-bsd-win32 abi "work together"
# Abstract

Chalenges:
-proxy all native sys calls over kernel specific syscalls -transparent proxy to X(?) server in order to provide native ui calls -transparent proxy to video accelerated hardware -reverse proxy to real hardware in order to use (or at least allow to install ) native device drivers (for example video accelerated hw ) -other not listed right now challenges to solve.

How is this different/similar to vm
This kernel ecosystem is intended to provide a direct kernel and hw acces. Using this you will be able to run for example VirtualBox from win32 alongside VirtualBox for linux.

Even the layer will have acces to low level kernel features (this mean there will be some security issues to deal with ), everything will run in a separated namespace - there will be provided a c:\ hdd not virtualised but maped to specific directory

The overload for the kernel in order to translate system calls must be minimal and most of functions will be only overlapped to real kernel function.

This means at the end: 1. You will have same performance as running reactos or windows on real hardware 2. You will be able to run Windows32/64 application as on real hardware 3. There will be no or just a small load on the main kernel itself as the most load will be directed to the hardware 4. You will be able to run windows as a server without any gui 5. you will be able to map real hardware or virtual hardware on linux to real hardware on windows and viceversa - for example if you have unmaintained driver shit you will have the opportunity to use windows drivers (and install it ) and use the device from windows tree in linux application (when a device will be added to windows device tree, a coresponding device will be created on sysfs )

and more

Why not wine
This is not exactly like wine but somehow similar -wine use the real kernel in order to -wine have no support to device translation -wine does not understand some low level function this layer will provide. -wine need some extraconfiguration.

This will provide basicaly a windows system in kernelspace and more.


#Documentation and source of inspiration

https://www.winehq.org/

http://www.longene.org/en/develop.php
