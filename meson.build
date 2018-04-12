project('wayland', 'c', 'cpp',
  version: '1.15.90',
  license: 'MIT',
  meson_version: '>= 0.45.0',
  default_options: [
    'warning_level=2',
    'buildtype=debugoptimized',
  ])

config_h = configuration_data()
config_h.set_quoted('PACKAGE', meson.project_name())
config_h.set_quoted('PACKAGE_NAME', meson.project_name())
config_h.set_quoted('PACKAGE_VERSION', meson.project_version())
config_h.set_quoted('VERSION', meson.project_version())
config_h.set_quoted('PACKAGE_URL', 'http://wayland.freedesktop.org')
config_h.set_quoted('PACKAGE_BUGREPORT', 'https://bugs.freedesktop.org/enter_bug.cgi?product=Wayland&component=@0@&version=@1@'.format(meson.project_name(), meson.project_version()))

cc = meson.get_compiler('c')

compiler_flags = cc.get_supported_arguments([
  '-Wno-unused-parameter',
  '-Wstrict-prototypes',
  '-Wmissing-prototypes',
  '-fvisibility=hidden',
])

add_project_arguments(compiler_flags, language: 'c')

foreach h: [ 'sys/prctl.h', 'dlfcn.h', 'memory.h', 'sys/stat.h', 'unistd.h', ]
  config_h.set('HAVE_' + h.underscorify().to_upper(), cc.has_header(h))
endforeach

foreach f: [ 'accept4', 'mkostemp', 'posix_fallocate', 'prctl', ]
  config_h.set('HAVE_' + f.underscorify().to_upper(), cc.has_function(f))
endforeach

if get_option('libraries')
  ffi_dep = dependency('libffi')

  decls = [
    [ 'sys/signalfd.h', 'SFD_CLOEXEC' ],
    [ 'sys/timerfd.h', 'TFD_CLOEXEC' ],
    [ 'time.h', 'CLOCK_MONOTONIC' ],
  ]

  foreach d: decls
    if not cc.has_header_symbol(d[0], d[1])
      error('@0@ is needed to compile wayland libraries'.format(d[1]))
    endif
  endforeach

  config_h.set('HAVE_EXECINFO_H', cc.has_header('execinfo.h'))
endif

expat_dep = dependency('expat', required: false)
if not expat_dep.found()
  if not cc.has_header('expat.h')
    error('Cannot find expat.h header. Please install expat')
  endif

  expat_dep = cc.find_library('expat')
  if not cc.has_function('XML_ParserCreate', dependencies: expat_dep)
    error('Cannot find expat library. Please install expat')
  endif
endif

if get_option('dtd_validation')
  libxml2_dep = dependency('libxml-2.0')
  config_h.set('HAVE_LIBXML', 1)
endif

configure_file(
  output: 'config.h',
  configuration: config_h,
)

add_project_arguments([ '-DHAVE_CONFIG_H' ], language: 'c')

pkgconfig = import('pkgconfig')

root_inc = include_directories('.')
protocol_inc = include_directories('protocol')
src_inc = include_directories('src')

subdir('src')

if get_option('libraries')
  subdir('cursor')
  subdir('egl')
  subdir('tests')
endif

if get_option('documentation')
  subdir('doc')
endif

install_data([
  'wayland-scanner.mk',
  'protocol/wayland.xml',
  'protocol/wayland.dtd',
])

install_data(
  [ 'wayland-scanner.m4' ],
  install_dir: join_paths(get_option('prefix'), get_option('datadir'), 'aclocal'),
)
