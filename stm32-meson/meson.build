fs = import('fs')

memory_map = { }
chip_type_map = {}

memory_map_array = meson.get_cross_property('memory_map')
foreach device_string : memory_map_array
  v = device_string.split(',')
  memory_values = [v[2].to_int(), v[3].to_int()]
  memory_map += {v[0]: memory_values}
  chip_type_map += {v[0]: v[1]}
endforeach


if get_option('stm32_flash_origin') != ''
  stm32_flash_origin = get_option('stm32_flash_origin')
endif

if get_option('stm32_ram_origin') != ''
  stm32_ram_origin = get_option('stm32_ram_origin')
endif

if get_option('stm32_flash_size') != ''
  stm32_flash_size = get_option('stm32_flash_size')
endif

if get_option('stm32_ram_size') != ''
  stm32_ram_size = get_option('stm32_ram_size')
endif

if get_option('stm32_min_stack_size') != ''
  stm32_min_stack_size = get_option('stm32_min_stack_size')
endif

if get_option('stm32_min_heap_size') != ''
  stm32_min_heap_size = get_option('stm32_min_heap_size')
endif

if get_option('stm32_ccram_origin') != ''
  stm32_ccram_origin = get_option('stm32_ccram_origin')
endif

if get_option('stm32_ccram_size') != ''
  stm32_ccram_size = get_option('stm32_ccram_size')
endif

if get_option('stm32_cube_dir') != ''
  stm32_cube_dir = get_option('stm32_cube_dir')
endif

if get_option('stm32_chip') != ''
  stm32_chip = get_option('stm32_chip')
endif

if stm32_cube_dir == ''
  message('stm32_cube_dir must point to valid cube directory')
endif

# Extract the chip family from the stm32_chip.
stm32_chip_family = 'STM32' + stm32_chip[5] + stm32_chip[6]

toolchain_family = 'STM32' + memory_map_array[0][5] + memory_map_array[0][6] 
if toolchain_family != stm32_chip_family
  error(stm32_chip + ' is not supported by the ' + toolchain_family + ' cross file')
endif

if stm32_chip not in chip_type_map
  error(stm32_chip + ' is included in the ' + toolchain_family + ' cross file')
endif

stm32_chip_series = stm32_chip_family + stm32_chip[7] + stm32_chip[8]
stm32_chip_type = chip_type_map[stm32_chip]

stm32_flash_origin = '0x08000000'
stm32_ram_origin = '0x20000000'

stm32_flash_size_kb = memory_map[stm32_chip][0]
stm32_ram_size_kb = memory_map[stm32_chip][1]

conf_data = configuration_data()
conf_data.set('stm32_flash_origin', stm32_flash_origin)
conf_data.set('stm32_flash_size', '@0@K'.format(stm32_flash_size_kb))
conf_data.set('stm32_ram_origin', stm32_ram_origin)
conf_data.set('stm32_ram_size', '@0@K'.format(stm32_ram_size_kb))
conf_data.set('stm32_min_heap_size', '0')
conf_data.set('stm32_min_stack_size', '0x200')
conf_data.set('stm32_ccram_def', '')
conf_data.set('stm32_ccram_section', '')

stm32_linker_script = configure_file(input : 'script.ld.in',
  output : 'script.ld',
  configuration : conf_data)
linker_script = '@0@'.format(stm32_linker_script)
stm32_linker_script_path = join_paths(meson.build_root(), linker_script)

# Add the liker script to all the executable linking.
stm32_link_args = [ '-T', stm32_linker_script_path]

add_project_link_arguments(stm32_link_args, language : ['c', 'cpp'] )

# Create list of include directories required for 
stm32_cube_inc = include_directories(
  [
    stm32_cube_dir + '/Drivers/CMSIS/Include',
    stm32_cube_dir + '/Drivers/' + stm32_chip_family + 'xx_HAL_Driver/Inc/',
    stm32_cube_dir + '/Drivers/CMSIS/Device/ST/' + stm32_chip_family + 'xx/Include/',
  ])

# Store a list of all the board src files.
stm32_cube_src = []
stm32_cmsis_src = []

if stm32_hal_components.length() != 0
  # Add default component.
  stm32_hal_components += ['cortex']
  
  # Create prefix for all the Driver source files.
  stm32_driver_src_prefix = stm32_cube_dir + '/Drivers/' + stm32_chip_family + 'xx_HAL_Driver/Src/' + stm32_chip_family.to_lower() + 'xx_hal'
  
  stm32_driver_src_ll_prefix = stm32_cube_dir + '/Drivers/' + stm32_chip_family + 'xx_HAL_Driver/Src/' + stm32_chip_family.to_lower() + 'xx_ll'
  
  # Add the hal component.
  stm32_cube_src += stm32_driver_src_prefix + '.c'
  
  # Add all the requested compononts.
  foreach component : stm32_hal_components
  
    if fs.exists(stm32_driver_src_prefix + '_' + component + '.c')
      stm32_cube_src += stm32_driver_src_prefix + '_' + component + '.c'
    endif
    if fs.exists(stm32_driver_src_prefix + '_' + component + '_ex.c') 
      stm32_cube_src += stm32_driver_src_prefix + '_' + component + '_ex.c'
    endif
    if fs.exists(stm32_driver_src_ll_prefix + '_' + component + '.c')
      stm32_cube_src += stm32_driver_src_ll_prefix + '_' + component + '.c'
    endif
    
  endforeach
endif

stm32_template_dir = stm32_cube_dir + '/Drivers/CMSIS/Device/ST/' + stm32_chip_family + 'xx/Source/Templates'

stm32_cmsis_system = stm32_template_dir + '/system_' + stm32_chip_family.to_lower() + 'xx.c'
stm32_cmsis_startup = stm32_template_dir + '/gcc/startup_' + stm32_chip_series.to_lower() + stm32_chip_type.to_lower() + '.s'

stm32_cmsis_src += stm32_cmsis_system
stm32_cmsis_src += stm32_cmsis_startup

stm32_cube_args = ['-D' + stm32_chip_family, '-D' + stm32_chip_series + stm32_chip_type, '-DUSE_FULL_LL_DRIVER']

stm32_cube_dep = declare_dependency(
  sources : [stm32_cube_src],
  compile_args : stm32_cube_args,
  include_directories : [stm32_cube_inc],
  )

stm32_cmsis_dep = declare_dependency(
  sources : [stm32_cmsis_src],
  compile_args : stm32_cube_args,
  include_directories : [stm32_cube_inc],
  )
