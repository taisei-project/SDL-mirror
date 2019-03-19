project('sdl2', 'c',
  version : '2.0.9',
  license : 'zlib',
  meson_version : '>=0.47.0',
  default_options: ['c_std=gnu99']
)

# Based on https://github.com/mesonbuild/sdl2/blob/2.0.3/meson.build

varr = meson.project_version().split('.')
version_major = varr[0]
version_minor = varr[1]
version_micro = varr[2]

cc = meson.get_compiler('c')

# BEGIN platform detection logic

platform_is_linux = (host_machine.system() == 'linux')
# FIXME: detect this properly
platform_is_linux_desktop = platform_is_linux
platform_is_android = false

platform_is_darwin = (host_machine.system() == 'darwin')
# FIXME: detect this properly
platform_is_macos = platform_is_darwin
platform_is_ios = false

platform_is_windows = (host_machine.system() == 'windows')
# FIXME: detect this properly
platform_is_win32 = platform_is_windows
platform_is_winrt = false

platform_is_dragonflybsd = (host_machine.system() == 'dragonfly')
platform_is_freebsd = (host_machine.system() == 'freebsd')
platform_is_netbsd = (host_machine.system() == 'netbsd')

platform_is_bsd = (
    platform_is_dragonflybsd or
    platform_is_freebsd or
    platform_is_netbsd or
false)

platform_is_haiku = (host_machine.system() == 'haiku')

platform_is_emscripten = (host_machine.system() == 'emscripten')

platform_is_psp = (host_machine.system() == 'psp')

# FIXME: verify this
platform_is_nacl = (host_machine.system() == 'nacl')

platform_is_unixlike = (
    platform_is_bsd or
    platform_is_darwin or
    platform_is_emscripten or
    platform_is_haiku or  # FIXME: verify this
    platform_is_linux or
false)

# END platform detection logic

thread_dep = dependency('threads', required : false)
use_threads = thread_dep.found() and get_option('enable_threads')

if platform_is_emscripten
    use_threads = false
endif

core_deps = [
    cc.find_library('m', required : false),
    cc.find_library('dl', required : false),
    thread_dep,
]

alsa_dep = dependency('alsa', required : false)
egl_dep = dependency('egl', required : false)
gl_dep = dependency('gl', required : false)
glesv2_dep = dependency('glesv2', required : false)
libpulse_simple_dep = dependency('libpulse-simple', required : false)
vulkan_dep = dependency('vulkan', required : false)
x11_dep = dependency('x11', required : false)
xext_dep = dependency('xext', required : false)
xi_dep = dependency('xi', required : false)
xinerama_dep = dependency('xinerama', required : false)
xkbcommon_dep = dependency('xkbcommon', required : false)
xrandr_dep = dependency('xrandr', required : false)

gl_deps = [
    egl_dep,
    gl_dep,
    glesv2_dep,
]

x11_deps = [
    x11_dep,
    xext_dep,
    xi_dep,
    xinerama_dep,
    xrandr_dep,
]

wayland_found = true

wayland_client_dep = dependency('wayland-client', required : false)
wayland_cursor_dep = dependency('wayland-cursor', required : false)
wayland_egl_dep = dependency('wayland-egl', required : false)
wayland_protocols_dep = dependency('wayland-protocols', required : false)
wayland_scanner_dep = dependency('wayland-scanner', required : false)

if wayland_scanner_dep.found()
    wayland_scanner = find_program(wayland_scanner_dep.get_pkgconfig_variable('wayland_scanner'))

    if not wayland_scanner.found()
        wayland_scanner = find_program('wayland-scanner')
    endif
else
    wayland_scanner = find_program('wayland-scanner')
endif

if wayland_scanner.found()
    wayland_deps = [
        wayland_client_dep,
        wayland_cursor_dep,
        wayland_egl_dep,
        wayland_protocols_dep,

        egl_dep,
        xkbcommon_dep,
    ]

    foreach wdep : wayland_deps
        if not wdep.found()
            wayland_found = false
        endif
    endforeach

    if not wayland_found
        wayland_deps = []
    endif
else
    wayland_deps = []
    wayland_found = false
endif

cc = meson.get_compiler('c')
cdata = configuration_data()

if not get_option('enable_atomic')
    cdata.set('SDL_ATOMIC_DISABLED', 1) endif
if not get_option('enable_audio')
    cdata.set('SDL_AUDIO_DISABLED', 1) endif
if not get_option('enable_cpuinfo')
    cdata.set('SDL_CPUINFO_DISABLED', 1) endif
if not get_option('enable_events')
    cdata.set('SDL_EVENTS_DISABLED', 1) endif
if not get_option('enable_file')
    cdata.set('SDL_FILE_DISABLED', 1) endif
if not get_option('enable_filesystem')
    cdata.set('SDL_FILESYSTEM_DISABLED', 1) endif
if not get_option('enable_haptic')
    cdata.set('SDL_HAPTIC_DISABLED', 1) endif
if not get_option('enable_joystick')
    cdata.set('SDL_JOYSTICK_DISABLED', 1) endif
if not get_option('enable_loadso')
    cdata.set('SDL_LOADSO_DISABLED', 1) endif
if not get_option('enable_power')
    cdata.set('SDL_POWER_DISABLED', 1) endif
if not get_option('enable_render')
    cdata.set('SDL_RENDER_DISABLED', 1) endif
if not get_option('enable_sensor')
    cdata.set('SDL_SENSOR_DISABLED', 1) endif
if not get_option('enable_timers')
    cdata.set('SDL_TIMERS_DISABLED', 1) endif
if not get_option('enable_video')
    cdata.set('SDL_VIDEO_DISABLED', 1) endif
if not use_threads
    cdata.set('SDL_THREADS_DISABLED', 1) endif

cdata.set('SIZEOF_VOIDP', cc.sizeof('void*'))

check_headers = [
    'alloca.h',
    'ctype.h',
    'float.h',
    'iconv.h',
    'inttypes.h',
    'limits.h',
    'malloc.h',
    'math.h',
    'memory.h',
    'signal.h',
    'stdarg.h',
    'stdint.h',
    'stdio.h',
    'stdlib.h',
    'strings.h',
    'string.h',
    'sys/types.h',
    'wchar.h',
    'pthread/np.h',
    'libunwind.h',
]

foreach h : check_headers
    if cc.has_header(h)
        cdata.set('HAVE_@0@'.format(h.to_upper().underscorify()), 1)
    endif
endforeach

check_functions = [
    ['HAVE_MALLOC', 'malloc', '#include<stdlib.h>'],
    ['HAVE_CALLOC', 'calloc', '#include<stdlib.h>'],
    ['HAVE_REALLOC', 'realloc', '#include<stdlib.h>'],
    ['HAVE_FREE', 'free', '#include<stdlib.h>'],
    ['HAVE_ALLOCA', 'alloca', '#include<alloca.h>'],
    ['HAVE_GETENV', 'getenv', '#include<stdlib.h>'],
    ['HAVE_SETENV', 'setenv', '#include<stdlib.h>'],
    ['HAVE_PUTENV', 'putenv', '#include<stdlib.h>'],
    ['HAVE_UNSETENV', 'unsetenv', '#include<stdlib.h>'],
    ['HAVE_QSORT', 'qsort', '#include<stdlib.h>'],
    ['HAVE_ABS', 'abs', '#include<stdlib.h>'],
    ['HAVE_BCOPY', 'bcopy', '#include<strings.h>'],
    ['HAVE_MEMSET', 'memset', '#include<string.h>'],
    ['HAVE_MEMCPY', 'memcpy', '#include<string.h>'],
    ['HAVE_MEMMOVE', 'memmove', '#include<string.h>'],
    ['HAVE_MEMCMP', 'memcmp', '#include<string.h>'],
    ['HAVE_WCSLEN', 'wcslen', '#include<wchar.h>'],
    ['HAVE_WCSLCPY', 'wcslcpy', '#include<wchar.h>'],
    ['HAVE_WCSLCAT', 'wcslcat', '#include<wchar.h>'],
    ['HAVE_WCSCMP', 'wcscmp', '#include<wchar.h>'],
    ['HAVE_STRLEN', 'strlen', '#include<string.h>'],
    ['HAVE_STRLCPY', 'strlcpy', '#include<string.h>'],
    ['HAVE_STRLCAT', 'strlcat', '#include<string.h>'],
    ['HAVE_STRDUP', 'strdup', '#include<string.h>'],
    ['HAVE__STRREV', '_strrev', '#include<string.h>'],
    ['HAVE__STRUPR', '_strupr', '#include<string.h>'],
    ['HAVE__STRLWR', '_strlwr', '#include<string.h>'],
    ['HAVE_INDEX', 'index', '#include<strings.h>'],
    ['HAVE_RINDEX', 'rindex', '#include<strings.h>'],
    ['HAVE_STRCHR', 'strchr', '#include<string.h>'],
    ['HAVE_STRRCHR', 'strrchr', '#include<string.h>'],
    ['HAVE_STRSTR', 'strstr', '#include<string.h>'],
    ['HAVE_ITOA', 'itoa', '#include<stdlib.h>'],
    ['HAVE__LTOA', '_ltoa', '#include<stlib.h>'],
    ['HAVE__UITOA', '_uitoa', '#include<stdlib.h>'],
    ['HAVE__ULTOA', '_ultoa', '#include<stdlib.h>'],
    ['HAVE_STRTOL', 'strtol', '#include<stdlib.h>'],
    ['HAVE_STRTOUL', 'strtoul', '#include<stdlib.h>'],
    ['HAVE__I64TOA', '_i64toa', '#include<stdlib.h>'],
    ['HAVE__UI64TOA', '_uit64toa', '#include<stdlib.h>'],
    ['HAVE_STRTOLL', 'strtoll', '#include<stdlib.h>'],
    ['HAVE_STRTOULL', 'strtoull', '#include<stdlib.h>'],
    ['HAVE_STRTOD', 'strtod', '#include<stdlib.h>'],
    ['HAVE_ATOI', 'atoi', '#include<stdlib.h>'],
    ['HAVE_ATOF', 'atof', '#include<stdlib.h>'],
    ['HAVE_STRCMP', 'strcmp', '#include<string.h>'],
    ['HAVE_STRNCMP', 'strncmp', '#include<string.h>'],
    ['HAVE__STRICMP', '_stricmp', '#include<string.h>'],
    ['HAVE_STRCASECMP', 'strcasecmp', '#include<strings.h>'],
    ['HAVE__STRNICMP', '_strnicmp', '#include<string.h>'],
    ['HAVE_STRNCASECMP', 'strncasecmp', '#include<strings.h>'],
    ['HAVE_SSCANF', 'sscanf', '#include<stdio.h>'],
    ['HAVE_VSSCANF', 'vsscanf', '#include<stdio.h>'],
    ['HAVE_SNPRINTF', 'snprintf', '#include<stdio.h>'],
    ['HAVE_VSNPRINTF', 'vsnprintf', '#include<stdio.h>'],
    ['HAVE_ATAN', 'atan', '#include<math.h>'],
    ['HAVE_ATANF', 'atanf', '#include<math.h>'],
    ['HAVE_ATAN2', 'atan2', '#include<math.h>'],
    ['HAVE_ATAN2F', 'atan2f', '#include<math.h>'],
    ['HAVE_ACOS', 'acos', '#include<math.h>'],
    ['HAVE_ACOSF', 'acosf', '#include<math.h>'],
    ['HAVE_ASIN', 'asin', '#include<math.h>'],
    ['HAVE_ASINF', 'asinf', '#include<math.h>'],
    ['HAVE_CEIL', 'ceil', '#include<math.h>'],
    ['HAVE_CEIL', 'ceilf', '#include<math.h>'],
    ['HAVE_COPYSIGN', 'copysign', '#include<math.h>'],
    ['HAVE_COPYSIGNF', 'copysignf', '#include<math.h>'],
    ['HAVE_COS', 'cos', '#include<math.h>'],
    ['HAVE_COSF', 'cosf', '#include<math.h>'],
    ['HAVE_EXP', 'exp', '#include<math.h>'],
    ['HAVE_EXPF', 'expf', '#include<math.h>'],
    ['HAVE_FABS', 'fabs', '#include<math.h>'],
    ['HAVE_FABSF', 'fabsf', '#include<math.h>'],
    ['HAVE_FLOOR', 'floor', '#include<math.h>'],
    ['HAVE_FLOORF', 'floorf', '#include<math.h>'],
    ['HAVE_FMOD', 'fmod', '#include<math.h>'],
    ['HAVE_FMODF', 'fmodf', '#include<math.h>'],
    ['HAVE_LOG', 'log', '#include<math.h>'],
    ['HAVE_LOGF', 'logf', '#include<math.h>'],
    ['HAVE_POW', 'pow', '#include<math.h>'],
    ['HAVE_POWF', 'powf', '#include<math.h>'],
    ['HAVE_SCALBN', 'scalbn', '#include<math.h>'],
    ['HAVE_SCALBNF', 'scalbnf', '#include<math.h>'],
    ['HAVE_SIN', 'sin', '#include<math.h>'],
    ['HAVE_SINF', 'sinf', '#include<math.h>'],
    ['HAVE_SQRT', 'sqrt', '#include<math.h>'],
    ['HAVE_SQRTF', 'sqrtf', '#include<math.h>'],
    ['HAVE_TANF', 'tan', '#include<math.h>'],
    ['HAVE_FOPEN64', 'fopen64', '#include<stdio.h>'],
    ['HAVE_FSEEKO', 'fseeko', '#include<stdio.h>'],
    ['HAVE_FSEEKO64', 'fseeko64', '#include<stdio.h>'],
    ['HAVE_SIGACTION', 'sigaction', '#include<signal.h>'],
    # check token HAVE_SA_SIGACTION
    ['HAVE_SETJMP', 'setjmp', '#include<setjmp.h>'],
    ['HAVE_NANOSLEEP', 'nanosleep', '#include<time.h>'],
    ['HAVE_SYSCONF', 'sysconf', '#include<unistd.h>'],
    ['HAVE_SYSCTLBYNAME', 'sysctlbyname', '#include<sys/sysctl.h>'],
    ['HAVE_CLOCK_GETTIME', 'clock_gettime', '#include<time.h>'],
    ['HAVE_GETPAGESIZE', 'getpagesize', '#include<unistd.h>'],
    ['HAVE_MPROTECT', 'mprotect', '#include<sys/mman.h>'],
    ['HAVE_ICONV', 'iconv', '#include<iconv.h>'],
    # check token HAVE_ICONV
    ['HAVE_PTHREAD_SETNAME_NP', 'pthread_setname_np', '#include<pthread.h>'],
    ['HAVE_PTHREAD_SET_NAME_NP', 'pthread_set_name_np', '#include<pthread.h>'],
    # check token HAVE_SEM_TIMEDWAIT
    ['HAVE_GETAUXVAL', 'getauxval', '#include<sys/auxv.h>'],
    ['HAVE_POLL', 'poll', '#include<poll.h>'],
]

foreach f : check_functions
    if cc.has_function(f.get(1), prefix : f.get(2), dependencies : core_deps)
        cdata.set(f.get(0), 1)
    endif
endforeach

cdata.set('HAVE_LIBC', 1)
cdata.set('HAVE_M_PI', cc.has_header_symbol('math.h', 'M_PI'))

core_args = ['-DUSING_GENERATED_CONFIG_H']
core_ldflags = []

cdata.set10('HAVE_GCC_ATOMICS',
    cc.links('int main() { int x; return __atomic_load_n(&x, __ATOMIC_SEQ_CST); }',
        name : '__atomic_load_n test'
    )
)

cdata.set10('HAVE_GCC_SYNC_LOCK_TEST_AND_SET',
    cc.links('int main() { int x; return __sync_lock_test_and_set(&x, 1); }',
        name : '__sync_lock_test_and_set test'
    )
)

cdata.set('SDL_THREAD_PTHREAD_RECURSIVE_MUTEX', 1)

cdata.set('SDL_AUDIO_DRIVER_DUMMY', 1)
cdata.set('SDL_VIDEO_DRIVER_DUMMY', 1)
cdata.set('SDL_JOYSTICK_DUMMY', 1)
cdata.set('SDL_SENSOR_DUMMY', 1)

if platform_is_linux
    cdata.set('SDL_POWER_LINUX', 1)
    cdata.set('SDL_THREAD_PTHREAD', 1)
    cdata.set('SDL_JOYSTICK_LINUX', 1)
    cdata.set('SDL_INPUT_LINUXEV', 1)
    cdata.set('SDL_INPUT_LINUXKD', 1)
    if cdata.get('SDL_HAPTIC_DISABLED', 0) == 0
        cdata.set('SDL_HAPTIC_LINUX', 1) endif
    cdata.set('SDL_TIMER_UNIX', 1)
    cdata.set('SDL_FILESYSTEM_UNIX', 1)
    cdata.set('SDL_DYNAMIC_API', 1)
    cdata.set('SDL_LOADSO_DLOPEN', 1)
elif platform_is_windows
    cdata.set('SDL_THREAD_WINDOWS', 1)
    cdata.set('SDL_JOYSTICK_DINPUT', 1)
    cdata.set('SDL_HAPTIC_DINPUT', 1)
    cdata.set('SDL_TIMER_WINDOWS', 1)
    cdata.set('SDL_FILESYSTEM_WINDOWS', 1)
    cdata.set('SDL_DYNAMIC_API', 1)
    cdata.set('SDL_POWER_WINDOWS', 1)
    cdata.set('SDL_LOADSO_WINDOWS', 1)
    cdata.set('SDL_VIDEO_DRIVER_WINDOWS', 1)
    cdata.set('SDL_VIDEO_RENDER_D3D', 1)
    cdata.set('SDL_VIDEO_RENDER_D3D11', 1)
    cdata.set('SDL_AUDIO_DRIVER_WASAPI', 1)
    core_deps += [
        cc.find_library('imm32'),
        cc.find_library('version'),
        cc.find_library('wasapi'),
        cc.find_library('winmm'),
    ]
elif platform_is_macos
    cdata.set('SDL_POWER_MACOSX', 1)
    cdata.set('SDL_THREAD_PTHREAD', 1)
    cdata.set('SDL_JOYSTICK_IOKIT', 1)
    cdata.set('SDL_HAPTIC_IOKIT', 1)
    cdata.set('SDL_TIMER_UNIX', 1)
    cdata.set('SDL_FILESYSTEM_UNIX', 1)
    cdata.set('SDL_DYNAMIC_API', 1)
    cdata.set('SDL_LOADSO_DLOPEN', 1)
    cdata.set('SDL_VIDEO_DRIVER_COCOA', 1)
    cdata.set('SDL_AUDIO_DRIVER_COREAUDIO', 1)

    if gl_dep.found()
        cdata.set('SDL_VIDEO_OPENGL_CGL', 1)
    endif

    core_deps += dependency('appleframeworks', modules : [
        'CoreFoundation',
        'Foundation',
        'QuartzCore',
        'CoreAudio',
        'AudioUnit',
        'AudioToolbox',
        'ForceFeedback',
        'IOKit',
        'Cocoa',
        'Carbon',
        'CoreGraphics'
    ])
else
    error('Unknown platform, please add.')
endif

if gl_dep.found()
    cdata.set('SDL_VIDEO_OPENGL', 1)
    cdata.set('SDL_VIDEO_RENDER_OGL', 1)
endif

if glesv2_dep.found()
    cdata.set('SDL_VIDEO_OPENGL_ES2', 1)
    cdata.set('SDL_VIDEO_RENDER_OGL_ES2', 1)
endif

if egl_dep.found() and (gl_dep.found() or glesv2_dep.found())
    cdata.set('SDL_VIDEO_OPENGL_EGL', 1)
endif

if vulkan_dep.found()
    cdata.set('SDL_VIDEO_VULKAN', 1)
endif

if x11_dep.found()
    cdata.set('SDL_VIDEO_DRIVER_X11', 1)

    # FIXME: test for this?
    cdata.set('SDL_VIDEO_DRIVER_X11_SUPPORTS_GENERIC_EVENTS', 1)

    if xi_dep.found()
        cdata.set('SDL_VIDEO_DRIVER_X11_XINPUT2', 1)

        # FIXME: test for this?
        cdata.set('SDL_VIDEO_DRIVER_X11_XINPUT2_SUPPORTS_MULTITOUCH', 1)
    endif

    if gl_dep.found() or glesv2_dep.found()
        cdata.set('SDL_VIDEO_OPENGL_GLX', 1)
    endif

    if xrandr_dep.found()
        cdata.set('SDL_VIDEO_DRIVER_X11_XRANDR', 1)
        # cdata.set('SDL_VIDEO_DRIVER_X11_DYNAMIC_XRANDR', 1)
    endif

    if xinerama_dep.found()
        cdata.set('SDL_VIDEO_DRIVER_X11_XINERAMA', 1)
        # cdata.set('SDL_VIDEO_DRIVER_X11_DYNAMIC_XINERAMA', 1)
    endif
endif

if wayland_found
    cdata.set('SDL_VIDEO_DRIVER_WAYLAND', 1)
endif

if alsa_dep.found()
    cdata.set('SDL_AUDIO_DRIVER_ALSA', 1)
    # cdata.set('SDL_AUDIO_DRIVER_ALSA_DYNAMIC', 1)
endif

if libpulse_simple_dep.found()
    cdata.set('SDL_AUDIO_DRIVER_PULSEAUDIO', 1)
endif

core_inc = [include_directories('include')]
c_sources = []
cxx_sources = []
objc_sources = []
main_sources = []

subdir('include')
subdir('src')
subdir('wayland-protocols')

all_sources = [c_sources]

if platform_is_darwin
    add_languages('objc')
    all_sources += [objc_sources]
endif

if platform_is_winrt or platform_is_haiku
    add_languages('cpp')
    all_sources += [cxx_sources]
endif

core_deps += [
    x11_deps,
    wayland_deps,
    gl_deps,
    alsa_dep,
    libpulse_simple_dep,
]

main_c_args = ['-Dmain=SDL_main']

sdl2 = library('sdl2',
    all_sources,
    include_directories : core_inc,
    implicit_include_directories : false,
    c_args : core_args,
    link_args : core_ldflags,
    dependencies : core_deps,
    install : false,
    build_by_default : not meson.is_subproject(),
)

sdl2main = static_library('sdl2main',
    main_sources,
    include_directories : core_inc,
    implicit_include_directories : false,
    c_args : [core_args, main_c_args],
    install : false,
    build_by_default : not meson.is_subproject(),
)

sdl2_dep = declare_dependency(
    link_with : [sdl2, sdl2main],
    include_directories : core_inc,
    compile_args : main_c_args,
)

sdl2main_dep = declare_dependency(
    link_with : sdl2main,
    include_directories : core_inc,
    compile_args : main_c_args,
)
