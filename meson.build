project('stm32_cross', 'c',
  version : '0.1',
  default_options : ['warning_level=3'])


# Include the required modules to compile the projects. 
stm32_hal_components = ['pcd', 'usb', 'adc', 'icache', 'eth','uart', 'dma', 'gpio', 'tim', 'tim_ex', 'rcc']

# Include to setup all the board dependencies.
subdir('stm32-meson')

# Include subdir with user defined projects.
subdir('Core')
