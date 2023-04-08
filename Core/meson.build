
headers_inc = include_directories( 'Inc' )

stm32_cross = executable('stm32_cross.elf',
  'Src/main.c',
  'Src/stm32h5xx_hal_msp.c',
  'Src/stm32h5xx_it.c',
  'Src/syscalls.c',
  'Src/sysmem.c',
  
  include_directories : [
    headers_inc,
  ],
  
  dependencies : [
  	stm32_cube_dep, 
  	stm32_cmsis_dep
  ]
  
 ) # end executable
 
 
objcopy = find_program('arm-none-eabi-objcopy')
 
custom_target(
    'stm32_cross.bin',
    input	     : [stm32_cross],
    output           : ['stm32_cross.bin'],
    build_by_default : true,
    command          : [objcopy, '-O', 'binary', '@INPUT@', '@OUTPUT@'],
    depends          : [stm32_cross])
