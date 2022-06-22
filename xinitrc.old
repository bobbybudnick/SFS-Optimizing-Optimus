#set custom modeline for 4k 30 fps reduced blanking
xrandr --newmode "3840x2160" 297.00 3840 4016 4104 4400 2160 2168 2178 2250 +hsync +

#modesetting mode
#xrandr --addmode HDMI-1 3840x2160
#xrandr --output eDP-1 --mode 1920x1080 --scale 2x2 --output HDMI-1 --mode 3840x2160

#primarygpu mode
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
xrandr --newmode "3840x2160" 297.00 3840 4016 4104 4400 2160 2168 2178 2250 +hsync +
xrandr --addmode HDMI-1-1 3840x2160
xrandr --output eDP-1-1 --mode 1920x1080 --scale 2x2 --output HDMI-1-1 --mode 3840x2160

#intel mode
#xrandr --addmode HDMI1 3840x2160
#xrandr --output eDP1 --mode 1920x1080 --scale 2x2 --output HDMI1 --mode 3840x2160

#start de
sleep 1
exec startplasma-x11

