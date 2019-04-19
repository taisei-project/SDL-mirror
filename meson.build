project('sdl2', 'c',
    version : '2.0.9',
    license : 'zlib',
    meson_version : '>=0.48.0',
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

opt_atomic = get_option('use_atomic')
opt_audio = get_option('use_audio')
opt_cpuinfo = get_option('use_cpuinfo')
opt_events = get_option('use_events')
opt_file = get_option('use_file')
opt_filesystem = get_option('use_filesystem')
opt_haptic = get_option('use_haptic')
opt_joystick = get_option('use_joystick')
opt_loadso = get_option('use_loadso')
opt_power = get_option('use_power')
opt_render = get_option('use_render')
opt_sensor = get_option('use_sensor')
opt_threads = get_option('use_threads')
opt_timers = get_option('use_timers')
opt_video = get_option('use_video')

opt_video_x11 = get_option('use_video_x11')
opt_video_wayland = get_option('use_video_wayland')
opt_video_opengl = get_option('use_video_opengl')
opt_video_openglesv2 = get_option('use_video_openglesv2')
opt_video_vulkan = get_option('use_video_vulkan')

if opt_video.disabled()
    opt_video_x11 = opt_video
    opt_video_wayland = opt_video
    opt_video_opengl = opt_video
    opt_video_openglesv2 = opt_video
    opt_video_vulkan = opt_video
    opt_render = opt_video
endif

opt_audio_alsa = get_option('use_audio_alsa')
opt_audio_pulseaudio = get_option('use_audio_pulseaudio')

if opt_audio.disabled()
    opt_audio_alsa = opt_audio
    opt_audio_pulseaudio = opt_audio
endif

# BEGIN awful shitfucked GL hacks

khronos_dep = declare_dependency(
    include_directories : include_directories(join_paths('src', 'video', 'khronos'))
)

dummy_found_dep = declare_dependency()
dummy_notfound_dep = dependency('lalala this dependency does not exist lalala', required : false)

if dummy_notfound_dep.found()
    error('You are not funny')
endif

gl_req_hack = opt_video_opengl.enabled() ? false : opt_video_opengl
gles2_req_hack = opt_video_openglesv2.enabled() ? false : opt_video_openglesv2

#if opt_video_opengl.disabled()
    #gl_dep = dummy_notfound_dep
#else
    # just assume we can use it... :/
    #gl_dep = dummy_found_dep
#endif

gl_dep = dependency('gl', required : opt_video_opengl)

glesv2_dep = dependency('glesv2', required : gles2_req_hack)
if not glesv2_dep.found() and not opt_video_openglesv2.disabled()
    # we have the headers, we'll load the lib dynamically (which might be something like ANGLE)
    glesv2_dep = khronos_dep
    message('Using embedded GLESv2 headers')
endif

egl_dep = dependency('egl', required : opt_video_openglesv2)
if not egl_dep.found()
    egl_dep = dependency('egl', required : opt_video_wayland)
endif
if not egl_dep.found() and not opt_video_openglesv2.disabled()
    # we have the headers, we'll load the lib dynamically (which might be something like ANGLE)
    egl_dep = khronos_dep
    message('Using embedded EGL headers')
endif

# we don't want to link against these!
gl_dep = gl_dep.partial_dependency(compile_args : true, includes : true, sources : true)
glesv2_dep = glesv2_dep.partial_dependency(compile_args : true, includes : true, sources : true)

gl_deps = [
    egl_dep,
    gl_dep,
    glesv2_dep,
]

# END awful shitfucked GL hacks

alsa_dep = dependency('alsa', required : opt_audio_alsa)
libpulse_simple_dep = dependency('libpulse-simple', required : opt_audio_pulseaudio)
vulkan_dep = dependency('vulkan', required : opt_video_vulkan)
x11_dep = dependency('x11', required : opt_video_x11)
xext_dep = dependency('xext', required : opt_video_x11)
xi_dep = dependency('xi', required : opt_video_x11)
xinerama_dep = dependency('xinerama', required : opt_video_x11)
xkbcommon_dep = dependency('xkbcommon', required : opt_video_wayland)
xrandr_dep = dependency('xrandr', required : opt_video_x11)
threads_dep = dependency('threads', required : opt_threads)
iconv_dep = dependency('iconv', required : false)

x11_deps = [
    x11_dep,
    xext_dep,
    xi_dep,
    xinerama_dep,
    xrandr_dep,
]

wayland_found = true

wayland_client_dep = dependency('wayland-client', required : opt_video_wayland)
wayland_cursor_dep = dependency('wayland-cursor', required : opt_video_wayland)
wayland_egl_dep = dependency('wayland-egl', required : opt_video_wayland)
wayland_protocols_dep = dependency('wayland-protocols', required : opt_video_wayland)
wayland_scanner_dep = dependency('wayland-scanner', required : opt_video_wayland)

if wayland_scanner_dep.found()
    wayland_scanner = find_program(wayland_scanner_dep.get_pkgconfig_variable('wayland_scanner'), required : opt_video_wayland)

    if not wayland_scanner.found()
        wayland_scanner = find_program('wayland-scanner', required : opt_video_wayland)
    endif
else
    wayland_scanner = find_program('wayland-scanner', required : opt_video_wayland)
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

core_deps = [
    cc.find_library('m', required : false),
    cc.find_library('dl', required : false),
]

extra_deps = []

if meson.get_cross_property('threads_dep_is_broken', false)
    if opt_threads.disabled()
        threads_dep = dummy_notfound_dep
    else
        threads_dep = dummy_found_dep
    endif
endif

core_subsystems = {
    'atomic'        : { 'opt' : opt_atomic,     'deps' : [], 'opt_deps' : [] },
    'audio'         : { 'opt' : opt_audio,      'deps' : [], 'opt_deps' : [alsa_dep, libpulse_simple_dep] },
    'cpuinfo'       : { 'opt' : opt_cpuinfo,    'deps' : [], 'opt_deps' : [] },
    'events'        : { 'opt' : opt_events,     'deps' : [], 'opt_deps' : x11_deps + wayland_deps },
    'file'          : { 'opt' : opt_file,       'deps' : [], 'opt_deps' : [] },
    'filesystem'    : { 'opt' : opt_filesystem, 'deps' : [], 'opt_deps' : [] },
    'haptic'        : { 'opt' : opt_haptic,     'deps' : [], 'opt_deps' : [] },
    'joystick'      : { 'opt' : opt_joystick,   'deps' : [], 'opt_deps' : [] },
    'loadso'        : { 'opt' : opt_loadso,     'deps' : [], 'opt_deps' : [] },
    'power'         : { 'opt' : opt_power,      'deps' : [], 'opt_deps' : [] },
    'render'        : { 'opt' : opt_render,     'deps' : [], 'opt_deps' : [vulkan_dep] + x11_deps + wayland_deps + gl_deps },
    'sensor'        : { 'opt' : opt_sensor,     'deps' : [], 'opt_deps' : [] },
    'threads'       : { 'opt' : opt_threads,    'deps' : [threads_dep], 'opt_deps' : [] },
    'timers'        : { 'opt' : opt_timers,     'deps' : [], 'opt_deps' : [] },
    'video'         : { 'opt' : opt_video,      'deps' : [], 'opt_deps' : [vulkan_dep] + x11_deps + wayland_deps + gl_deps },
}

enabled_subsystems = []

cdata = configuration_data()

foreach ss_name, subsystem : core_subsystems
    ss_enabled = true

    if not subsystem['opt'].disabled()
        foreach dep : subsystem['deps']
            if not dep.found()
                warning('Subsystem "@0@" disabled due to missing dependency "@1@"'.format(ss_name, dep.name()))

                if subsystem['opt'].enabled()
                    error('Subsystem @0@ couldn\'t be enabled'.format(ss_name))
                endif

                ss_enabled = false
            endif
        endforeach
    else
        ss_enabled = false
    endif

    if ss_enabled
        foreach dep : subsystem['deps']
            if dep.found() and not extra_deps.contains(dep) and dep != dummy_found_dep
                core_deps += dep
            endif
        endforeach

        foreach dep : subsystem['opt_deps']
            if dep.found() and not extra_deps.contains(dep) and dep != dummy_found_dep
                extra_deps += dep
            endif
        endforeach

        message('Subsystem "@0@" is ENABLED'.format(ss_name))
        enabled_subsystems += ss_name
    else
        cdata.set('SDL_@0@_DISABLED'.format(ss_name.to_upper()), 1)
        message('Subsystem "@0@" is DISABLED'.format(ss_name))
    endif
endforeach

# sigh...
# https://github.com/mesonbuild/meson/issues/1889
# Let's try to cure the superproject std
unfuck_std = '''
#define _DEFAULT_SOURCE
#define _GNU_SOURCE
#define _ISOC11_SOURCE
#define _ISOC99_SOURCE
#define _POSIX_C_SOURCE 200809L
#define _XOPEN_SOURCE 700
#undef __STRICT_ANSI__
'''

cdata.set('SIZEOF_VOIDP', cc.sizeof('void*'))

check_headers = [
    'alloca.h',
    'altivec.h',
    'audioclient.h',
    'ctype.h',
    'dbus/dbus.h',
    'ddraw.h',
    'dinput.h',
    'dsound.h',
    'dxgi.h',
    'endpointvolume.h',
    'fcitx/frontend.h',
    'float.h',
    'ibus/ibus.h',
    'iconv.h',
    'immintrin.h',
    'inttypes.h',
    'libsamplerate.h',
    'libudev.h',
    'libunwind.h',
    'limits.h',
    'malloc.h',
    'math.h',
    'memory.h',
    'mmdeviceapi.h',
    'pthread.h',
    'pthread_np.h',
    'semaphore.h',
    'signal.h',
    'stdarg.h',
    'stdint.h',
    'stdio.h',
    'stdlib.h',
    'string.h',
    'strings.h',
    'sys/types.h',
    'wchar.h',
    'xinput.h',
]

foreach h : check_headers
    if cc.has_header(h)
        cdata.set('HAVE_@0@'.format(h.to_upper().underscorify()), 1)
    endif
endforeach

check_functions = [
    ['#include <alloca.h>', 'alloca'],
    ['#include <iconv.h>', 'iconv'],
    ['#include <math.h>', 'acos'],
    ['#include <math.h>', 'acosf'],
    ['#include <math.h>', 'asin'],
    ['#include <math.h>', 'asinf'],
    ['#include <math.h>', 'atan'],
    ['#include <math.h>', 'atan2'],
    ['#include <math.h>', 'atan2f'],
    ['#include <math.h>', 'atanf'],
    ['#include <math.h>', 'ceil'],
    ['#include <math.h>', 'ceilf'],
    ['#include <math.h>', 'copysign'],
    ['#include <math.h>', 'copysignf'],
    ['#include <math.h>', 'cos'],
    ['#include <math.h>', 'cosf'],
    ['#include <math.h>', 'exp'],
    ['#include <math.h>', 'expf'],
    ['#include <math.h>', 'fabs'],
    ['#include <math.h>', 'fabsf'],
    ['#include <math.h>', 'floor'],
    ['#include <math.h>', 'floorf'],
    ['#include <math.h>', 'fmod'],
    ['#include <math.h>', 'fmodf'],
    ['#include <math.h>', 'log'],
    ['#include <math.h>', 'logf'],
    ['#include <math.h>', 'pow'],
    ['#include <math.h>', 'powf'],
    ['#include <math.h>', 'scalbn'],
    ['#include <math.h>', 'scalbnf'],
    ['#include <math.h>', 'sin'],
    ['#include <math.h>', 'sinf'],
    ['#include <math.h>', 'sqrt'],
    ['#include <math.h>', 'sqrtf'],
    ['#include <math.h>', 'tan'],
    ['#include <poll.h>', 'poll'],
    ['#include <pthread.h>', 'pthread_set_name_np'],
    ['#include <pthread.h>', 'pthread_setname_np'],
    ['#include <pthread.h>\n#include <semaphore.h>', 'sem_timedwait'],
    ['#include <setjmp.h>', 'setjmp'],
    ['#include <signal.h>', 'sigaction'],
    ['#include <stdio.h>', 'fopen64'],
    ['#include <stdio.h>', 'fseeko'],
    ['#include <stdio.h>', 'fseeko64'],
    ['#include <stdio.h>', 'snprintf'],
    ['#include <stdio.h>', 'sscanf'],
    ['#include <stdio.h>', 'vsnprintf'],
    ['#include <stdio.h>', 'vsscanf'],
    ['#include <stdlib.h>', '_i64toa'],
    ['#include <stdlib.h>', '_uit64toa'],
    ['#include <stdlib.h>', '_uitoa'],
    ['#include <stdlib.h>', '_ultoa'],
    ['#include <stdlib.h>', 'abs'],
    ['#include <stdlib.h>', 'atof'],
    ['#include <stdlib.h>', 'atoi'],
    ['#include <stdlib.h>', 'calloc'],
    ['#include <stdlib.h>', 'free'],
    ['#include <stdlib.h>', 'getenv'],
    ['#include <stdlib.h>', 'itoa'],
    ['#include <stdlib.h>', 'malloc'],
    ['#include <stdlib.h>', 'putenv'],
    ['#include <stdlib.h>', 'qsort'],
    ['#include <stdlib.h>', 'realloc'],
    ['#include <stdlib.h>', 'setenv'],
    ['#include <stdlib.h>', 'strtod'],
    ['#include <stdlib.h>', 'strtol'],
    ['#include <stdlib.h>', 'strtoll'],
    ['#include <stdlib.h>', 'strtoul'],
    ['#include <stdlib.h>', 'strtoull'],
    ['#include <stdlib.h>', 'unsetenv'],
    ['#include <stlib.h>', '_ltoa'],
    ['#include <string.h>', '_stricmp'],
    ['#include <string.h>', '_strlwr'],
    ['#include <string.h>', '_strnicmp'],
    ['#include <string.h>', '_strrev'],
    ['#include <string.h>', '_strupr'],
    ['#include <string.h>', 'memcmp'],
    ['#include <string.h>', 'memcpy'],
    ['#include <string.h>', 'memmove'],
    ['#include <string.h>', 'memset'],
    ['#include <string.h>', 'strchr'],
    ['#include <string.h>', 'strcmp'],
    ['#include <string.h>', 'strdup'],
    ['#include <string.h>', 'strlcat'],
    ['#include <string.h>', 'strlcpy'],
    ['#include <string.h>', 'strlen'],
    ['#include <string.h>', 'strncmp'],
    ['#include <string.h>', 'strrchr'],
    ['#include <string.h>', 'strstr'],
    ['#include <strings.h>', 'bcopy'],
    ['#include <strings.h>', 'index'],
    ['#include <strings.h>', 'rindex'],
    ['#include <strings.h>', 'strcasecmp'],
    ['#include <strings.h>', 'strncasecmp'],
    ['#include <sys/auxv.h>', 'getauxval'],
    ['#include <sys/mman.h>', 'mprotect'],
    ['#include <sys/sysctl.h>', 'sysctlbyname'],
    ['#include <time.h>', 'clock_gettime'],
    ['#include <time.h>', 'nanosleep'],
    ['#include <unistd.h>', 'getpagesize'],
    ['#include <unistd.h>', 'sysconf'],
    ['#include <wchar.h>', 'wcscmp'],
    ['#include <wchar.h>', 'wcslcat'],
    ['#include <wchar.h>', 'wcslcpy'],
    ['#include <wchar.h>', 'wcslen'],
]

foreach f : check_functions
    prefix = f[0]
    func = f[1]

    if cc.has_function(func, prefix : prefix, dependencies : core_deps)
        cdata.set('HAVE_@0@'.format(func.underscorify().to_upper()), 1)
    endif
endforeach

if iconv_dep.found()
    core_deps += iconv_dep
else
    cdata.set('HAVE_ICONV', false)
    cdata.set('HAVE_ICONV_H', false)
endif


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

if cc.has_header_symbol('pthread.h', 'PTHREAD_MUTEX_RECURSIVE')
    cdata.set('SDL_THREAD_PTHREAD_RECURSIVE_MUTEX', 1)
endif

if cc.has_member('struct sigaction', 'sa_sigaction', prefix : '#include <signal.h>')
    cdata.set('HAVE_SA_SIGACTION', 1)
endif

sys_default_audio_driver = ['dummy']
sys_default_filesystem = ['dummy']
sys_default_haptic = ['dummy']
sys_default_input = []
sys_default_joystick = ['dummy']
sys_default_loadso = ['dummy']
sys_default_power = []
sys_default_sensor = ['dummy']
sys_default_thread = []
sys_default_timer = ['dummy']
sys_default_video = []
sys_default_video_driver = ['dummy']
sys_default_video_render = []

sys_audio_driver = sys_default_audio_driver
sys_filesystem = sys_default_filesystem
sys_haptic = sys_default_haptic
sys_input = sys_default_input
sys_joystick = sys_default_joystick
sys_loadso = sys_default_loadso
sys_power = sys_default_power
sys_sensor = sys_default_sensor
sys_thread = sys_default_thread
sys_timer = sys_default_timer
sys_video = sys_default_video
sys_video_driver = sys_default_video_driver
sys_video_render = sys_default_video_render

sys_gl_wsi = []

modular_subsystems = [
    'filesystem',
    'haptic',
    'joystick',
    'loadso',
    'power',
    'sensor',
    'video',
]

fake_subsystems = []
fake_subsystems_map = {
    'events'  : ['input'],
    'audio'   : ['audio_driver'],
    'video'   : ['video_driver'],
    'render'  : ['video_render'],
    'threads' : ['thread'],
    'timers'  : ['timer'],
}

all_modular_subsystems = modular_subsystems

foreach real_ss, fake_ss : fake_subsystems_map
    if enabled_subsystems.contains(real_ss)
        fake_subsystems += fake_ss
    endif
    all_modular_subsystems += fake_ss
endforeach

sys_dynamic_api = true

if platform_is_unixlike
    # Some sane defaults
    sys_filesystem = ['unix']
    sys_loadso = ['dlopen']
    sys_timer = ['unix']
    sys_thread = ['pthread']
endif

if platform_is_linux
    sys_haptic = ['linux']
    sys_input = ['linuxev', 'linuxkd']
    sys_joystick = ['linux']
    sys_power = ['linux']
elif platform_is_windows
    sys_audio_driver = ['wasapi']  # XXX: winmm needed?
    sys_filesystem = ['windows']
    sys_haptic = []
    sys_joystick = []  # XXX: winmm needed?
    sys_loadso = ['windows']
    sys_power = ['windows']
    sys_thread = ['windows']
    sys_timer = ['windows']
    sys_video_driver = ['windows']
    sys_video_render = ['d3d', 'd3d11']
    sys_gl_wsi = ['wgl']
    extra_deps += [
        # TODO: maybe filter this out conditionally...
        cc.find_library('dxerr8'),
        cc.find_library('dxguid'),
        cc.find_library('imm32'),
        cc.find_library('ole32'),
        cc.find_library('oleaut32'),
        cc.find_library('user32'),
        cc.find_library('version'),
        cc.find_library('winmm'),
    ]

    # XInput is loaded dynamically
    if not get_option('use_joystick_xinput').disabled()
        sys_joystick += 'xinput'
        sys_haptic += 'xinput'
    endif

    dinput = cc.find_library('dinput8', required : get_option('use_joystick_dinput'))

    if dinput.found()
        extra_deps += dinput
        sys_joystick += 'dinput'
        sys_haptic += 'dinput'
    endif
elif platform_is_macos
    sys_audio_driver = ['coreaudio']
    sys_filesystem = ['cocoa']
    sys_gl_wsi = ['cgl']
    sys_haptic = ['iokit']
    sys_joystick = ['iokit']
    sys_power = ['macosx']
    sys_video_driver = ['cocoa']
    extra_deps += dependency('appleframeworks', modules : [
        # TODO: maybe filter this out conditionally...
        'AudioToolbox',
        'AudioUnit',
        'Carbon',
        'Cocoa',
        'CoreAudio',
        'CoreFoundation',
        'CoreGraphics',
        'CoreServices',
        'ForceFeedback',
        'Foundation',
        'IOKit',
        'QuartzCore',
    ])
elif platform_is_emscripten
    sys_audio_driver = ['emscripten']
    sys_filesystem = ['emscripten']
    sys_joystick = ['emscripten']
    sys_power = ['emscripten']
    sys_video_driver = ['emscripten']
    sys_dynamic_api = false
else
    error('Unknown platform, please add.')
endif

if sys_dynamic_api
    cdata.set('SDL_DYNAMIC_API', 1)
endif

if extra_deps.contains(gl_dep)
    sys_video += 'opengl'
    sys_video_render += 'ogl'
endif

if extra_deps.contains(glesv2_dep)
    sys_video += 'opengl_es2'
    sys_video_render += 'ogl_es2'
endif

if extra_deps.contains(egl_dep)
    sys_video += 'opengl_egl'
endif

if extra_deps.contains(vulkan_dep)
    sys_video += 'vulkan'
endif

if extra_deps.contains(x11_dep)
    sys_video_driver += 'x11'
    sys_gl_wsi += 'glx'

    # FIXME: test for this?
    sys_video_driver += 'x11_supports_generic_events'

    if extra_deps.contains(xi_dep)
        sys_video_driver += 'x11_xinput2'

        # FIXME: test for this?
        sys_video_driver += 'x11_xinput2_supports_multitouch'
    endif

    if extra_deps.contains(xrandr_dep)
        sys_video_driver += 'x11_xrandr'
    endif

    if extra_deps.contains(xinerama_dep)
        sys_video_driver += 'x11_xinerama'
    endif

    if cc.has_function('XkbKeycodeToKeysym', dependencies : x11_dep)
        sys_video_driver += 'x11_has_xkbkeycodetokeysym'
    endif
endif

if wayland_found and extra_deps.contains(wayland_client_dep)
    sys_video_driver += 'wayland'
endif

if extra_deps.contains(gl_dep) or extra_deps.contains(glesv2_dep)
    foreach wsi : sys_gl_wsi
        sys_video += ['opengl_@0@'.format(wsi)]
    endforeach
endif

if extra_deps.contains(alsa_dep)
    cdata.set('SDL_AUDIO_DRIVER_ALSA', 1)
    sys_audio_driver += 'alsa'
endif

if extra_deps.contains(libpulse_simple_dep)
    sys_audio_driver += 'pulseaudio'
endif

if get_option('use_dummies')
    sys_audio_driver += 'dummy'
    sys_video_driver += 'dummy'
    sys_joystick += 'dummy'
    sys_sensor += 'dummy'
endif

foreach ss : all_modular_subsystems
    var_want = 'sys_@0@'.format(ss)
    var_default = 'sys_default_@0@'.format(ss)

    if not (enabled_subsystems + fake_subsystems).contains(ss)
        set_variable(var_want, get_variable(var_default))
    endif

    dedup_list = []
    foreach backend : get_variable(var_want)
        if not dedup_list.contains(backend)
            dedup_list += backend
        endif
    endforeach

    message('@1@:  @0@'.format(', '.join(dedup_list), ss))

    foreach backend : dedup_list
        cfg_var = 'SDL_@0@_@1@'.format(ss.to_upper(), backend.to_upper())
        cdata.set(cfg_var, 1)
        # message('Setting @0@ to 1'.format(cfg_var))
    endforeach
endforeach

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

main_c_args = ['-Dmain=SDL_main']

sdl2 = library('sdl2',
    all_sources,
    include_directories : core_inc,
    implicit_include_directories : false,
    c_args : core_args,
    link_args : core_ldflags,
    dependencies : [core_deps, extra_deps],
    override_options : ['c_std=none'],
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
    link_with : sdl2,
    link_whole : sdl2main,
    include_directories : core_inc,
    compile_args : main_c_args,
)

sdl2main_dep = declare_dependency(
    link_whole : sdl2main,
    include_directories : core_inc,
    compile_args : main_c_args,
)
