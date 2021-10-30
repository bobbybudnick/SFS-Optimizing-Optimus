# SFS-Optimizing-Optimus
A method for getting the best 4k performance for Nvidia Optimus hardware

# Intro

I really wanted to get smooth 4k playback on my Nvidia Optimus laptop this time without having to go back to 1080p output.

**Configuration**  
Hisense 4k TV  
Hp Envy x360 m6 laptop  
Devuan Chimaera amd64 

**Test Regime With intel and modeset Drivers - No mpv Workarounds**  
test 1 - 4k scaled 1080p hdmi capture - fairly slow  
test 2 - 4k 60 fps big buck bunny - at or near full speed  
test 3 - 4k youtube video unbelievable beauty - totally unusable

**Utilization**  
intel_gpu_top shows no more than 70% render usage  
top shows 225%+ cpu usage  

**Remarks**  
single display output makes no difference on speed  
modesetting and intel driver makes no difference in speed  
vsync disable with drirc and xorg.conf makes no difference on speed  
when 3d is playing the mouse cursor sometimes disappears

**Complicating Factors**  
the idea that i had to force a modeline with xrandr and 4k was not a default option  
the idea that the screen is 4k at 30 fps like many lower end 4k tvs

**Initial Conclusion or How Little Did I Know**  
This laptop also has Nvidia graphics but i have never used it or gotten it to work to this point.  It seems like if i am counting on video acceleration then either one should be able to do it.  Unless i am supposed to find a chart on which acceleration modes either device uses under a certain driver and compare that to the codecs used in the playback tests.  That is a pretty high bar for just getting youtube to work without being slow.

# Explanation of Rendering Modes

This is a breakdown of all the possible techniques to use the various video hardware in an Nvidia Optimus laptop.

**intel or modesetting Driver Acceleration Method**  
use intel gpu for rendering and output  
it appears that intel gpus require 4k playback to be done in mpv instead of the browser  
the performance is much better with mpv than the browser  
however mpv and youtube-dl are slow downloading from youtube  
additionally lack of 4k browser streaming support is unacceptable  
needs xinitrc or xsessionrc changes for screen setup

**Application Handoff Acceleration Method**  
use intel gpu for rendering and output and nvidia gpu for select tasks  
glxgears is faster with intel with modesetting driver - 2000 fps  
glxgears is slower with optirun nvidia - 1400 fps  
however primusrun / optirun drops less frames and is more smooth at 1440p  
primusrun may be slightly faster than optirun  
nvidia-xrun was not tested  
that said 1440p streaming as a limit is bordering on unacceptable  
needs xinitrc or xsessionrc changes for screen setup
     
**primarygpu Acceleration Method**  
use the nvidia gpu for all rendering and use intel gpu only for output  
the proprietary nvidia driver should be installed  
one key was installing a 4.x series kernel  
setprovideroutputsource will give a bad value error otherwise  
a temporary distribution downgrade was warranted to get an old kernel and headers  
exact kernel used was 4.9.0-13-amd64  
a custom xorg.conf was required  
the alternate 10-nvidia-drm-outputclass.conf would probably have worked also  
not possible to watch at above 1080p and have a silent experience with no fan noise  
built in screen output option to scale by 2x2 is silently ignored in this mode  
nvidia-settings will crash the first time it runs  
needs xinitrc or xsessionrc changes for setting output sink and screen setup

# High Resolution Video Performance Tips

Without taking these optimization points into consideration any further configuration would be a waste.

**Disable Spectre/Meltdown Mitigations**  
this helped by about 5-20% with ps2 emulation before  
turns out that mitigations seem in fact about 5-20% faster

**Set cpupower To performance Governor**  
sudo cpupower frequency-set -g performance  
harder to script since it needs root

**Choose Prefer Max Performance In nvidia-settings**  
must be set at each login
turn aa off and override in nvidia-settings  
set opengl image settings to high performance in nvidia-settings

**Firefox Is Much Faster Than Falkon Which Is Itself Faster Than Chromium On Youtube**  
chromium slow when tested with intel/modesetting  
chromium still slow with intel/modesetting even with acceleration on and vulkan on/off  
however chromium is the fastest so far when configured for nvidia and vulkan
               
**kwin Compositing Disabled**  
this sped things up slightly  
this makes for more consistent results

**Keep the Laptop On a Cool Surface**  
give the exhaust from the fans somewhere to go  
laptop will use more power as it heats which will use more power which will heat...  
take steps to avoid any thermal or power related throttling

**More Possible Performance Tips**  
try disabling internal monitor since scaling is bugged with primarygpu method anyway  
try configuring firefox for hardware acceleration instead of chromium  
try nouveau instead of proprietary nvidia driver if it is supported

# High Performance Results

Each statement specifically addresses points of confusion encountered along the way.

**Test Regime With primarygpu Acceleration Method - All Tests With Firefox**  
test 1 - 4k scaled 1080p hdmi capture - see tearing section  
test 2 - 4k 60 fps big buck bunny - full speed with some minor tearing with vlc  
test 3 - 4k youtube video unbelievable beauty - totally unusable  
1440p youtube video unbelievable beauty - played over 10K frames and dropped zero  
on youtube a few frames are dropped momentarily when an ad appears at 1440p  
nvidia is marginally better than with intel/modesetting  
1.little better frames at 1440p  
2.little less tearing with hdmi capture  
3.big buck bunny shows hardware decoder working and is slightly better

**Chromium Browser Redeems Itself**  
turn on override software rendering list  
turn on gpu rasterization  
turn on hardware accelerated video decode  
turn on zero copy rasterizer  
turn on vulkan - this is one key - was about as slow as firefox until this  
test 3 - 4k youtube video unbelievable beauty - played over 10K frames and dropped 23

**Nvidia vp9 Confusion**  
vp9 is a codec often used by youtube especially with 4k video  
apparently nvidia devices that support vp9 will only support 8 and not 10 bit  
there is some information out there that vp9 will not work at all with a browser  
as has been shown there is at least some improvement with vp9 in acceleration

**The Video Decoding Problem**  
apparently gt108m chips - the 9400 - lack the hardware video decoders  
maybe all the nvidia optimus chips lack decoders  
the idea is the intel chip is supposed to do the decoding  
this manifests as a vdpau device creation failure in vlc  
this manifests as vdpauinfo gpu at bus id X does not have a supported video decoder  
this is why vulkan output in chromium works well  
vulkan is the only thing besides opengl that can use the gpu for decoding  
devuan packages an old version of vlc but newer versions have a vulkan output  
vlc with a vulkan output could work as well as chromium does with vulkan  
vlc even with opengl is slow and it could have used that for proper video decoding  
one key is that mpv by default performs much better than vlc with every possible tweak  
mpv might use vulkan but in the options it clearly has many tricks and hacks it can use

**Tearing**  
tearing is another issue altogether  
it was an issue with the previous install especially with hdmi capture  
it seems to correlate with lower performance  
the character of the tearing is a little less annoying than with intel/modesetting  
the overall resolution of the 1080p capture is improved by being upscaled to 4k  
any time a video is scaled outside native resolution there is potential for tearing  
tearing was seen on the old distribution also  
so one way to not see much tearing is use a video at the native resolution  
another way to not see much tearing is to not scale the video at all

**Key Points**  
installing a 4.x series kernel to avoid setprovideroutputsource bad value error  
use of vulkan and other features within chromium for best playback  
use of mpv instead of vlc for file playback and hdmi capture

**Final Conclusion**  
as mentioned before the 30 fps aspect of the big screen complicates tearing issues  
the display hardware likely may not drive 4k at 60 fps anyway  
in this way the computer and screen are a good fit for one another  
in other words until screen prices come down this should be a working solution  
performance problems have been fixed  
however a 4k 60 fps screen and computer combo would solve a lot of tearing problems  
this should be as good as it is going to get for now

# File Legend

**Use xsessionrc or xinitrc not both / use xorg.conf or xorg options file not both**  

**Local List**  
**xinitrc/xsessionrc**-set xorg run time options-xinitrc with startx or xessionrc with de  
**xorg.conf** - setup primarygpu acceleration method - lets xorg redirect to nvidia  
**10-nvidia-drm-outputclass.conf** - alternate options method for xorg redirect to nvidia  
**modelines** - troubleshooting techniques and modelines for tricky displays

**Releases**  
**kernel headers common** - needed to compile nvidia driver  
**kernel headers amd64** - needed to compile nvidia driver  
**kernel image** - earlier kernel to allow for switching output without errors

**External**  
HDMI capture example - https://github.com/bobbybudnick/SFS-AMAZONVIDEO-TO-LINUX

