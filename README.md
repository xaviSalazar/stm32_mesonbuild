# Compile stm32 source files with MESON BUILD 

### Project inspired from @jhamberg repository name: stm32-meson

1. I modified many lines and arborescense to adopt one meson.build file.

2. Meson is a requirement to compile this example project.

3. Also you need to download the firmware header and source files
   for the board in this case STM32H563ZI.

```
meson build --cross-file stm32h5_board.ini
```

then go to build/

```
ninja
```
