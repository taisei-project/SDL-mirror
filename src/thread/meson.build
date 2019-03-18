c_sources += files(
    'SDL_thread.c',
)

if not use_threads
    subdir('generic')
elif platform_is_windows
    subdir('windows')
elif platform_is_psp
    subdir('psp')
else
    subdir('pthread')
endif
