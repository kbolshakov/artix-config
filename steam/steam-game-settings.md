## This file is to keep track of the custom vars and settings for specific games that require changes

### Mafia the Old Country
GAMESCOPE RUN COMMAND:
```
RADV_PERFTEST=gpl PROTON_ENABLE_NVAPI=0 %command%
```

DE RUN COMMAND:
```
RADV_PERFTEST=gpl PROTON_ENABLE_NVAPI=0 gamescope -f -W 2560 -H 1440 -- mangohud %command%
RADV_PERFTEST=gpl PROTON_ENABLE_NVAPI=0 mangohud %command%
```

Settings: use Intel Xe for scaling...

### Hogwarts Legasy
GAMESCOPE RUN COMMAND:
```
RADV_PERFTEST=gpl PROTON_ENABLE_NVAPI=0 %command%
```

DE RUN COMMAND:
```
RADV_PERFTEST=gpl PROTON_ENABLE_NVAPI=0 mangohud %command%
```

Settings: use Intel Xe for scaling...


### TEMP scratchpad for commands
```
ls /var/cache/pacman/pkg/*libdrm*
MESA_VK_WSI_PRESENT_MODE=fifo SteamDeck=1 gamescope --adaptive-sync --mangoapp -f -r 72 -W 2560 -H 1440 -- %command%


```