#About the windows bootup process

The windows boot up process imply the folowing stages (some stages combined):

1. the bootloader load specified executable and give it the control.
2. the loader make some system checks, device detects (1 stage )
3. the loader load some low level device drivers
4. the loader execute some low level programs or scheduled tasks (for example chkdsk on "root" )
5. device detect second stage 
6. the loader begin to start win32 services (device in userspace)
7. and also start the 
8. some other user space initialisation are made
9. the user interface is presented - login maybe ;other checks; automate startup applications
10. after login some 

#Interprocess comunication to provide 
Inside the win32 ecosystem, we have to provide:
sockets,
pipes,
other specific ways of interprocess communication




