wayland_version = meson.project_version().split('.')
wayland_version_h = configuration_data()
wayland_version_h.set('WAYLAND_VERSION', meson.project_version())
wayland_version_h.set('WAYLAND_VERSION_MAJOR', wayland_version[0].to_int())
wayland_version_h.set('WAYLAND_VERSION_MINOR', wayland_version[1].to_int())
wayland_version_h.set('WAYLAND_VERSION_MICRO', wayland_version[2].to_int())
configure_file(
  input: 'wayland-version.h.in',
  output: 'wayland-version.h',
  configuration: wayland_version_h,
  install: true,
  install_dir: join_paths(get_option('prefix'), get_option('includedir')),
)

wayland_util = static_library(
  'wayland-util',
  sources: 'wayland-util.c',
)

wayland_util_dep = declare_dependency(
  link_with: wayland_util,
  include_directories: include_directories('.'),
)

if get_option('host_scanner')
  wayland_scanner = find_program('wayland-scanner')
  wayland_scanner_code_arg = 'code'
else
  configure_file(
    input: '../protocol/wayland.dtd',
    output: 'wayland.dtd.embed',
    configuration: configuration_data(),
  )

  wayland_scanner = executable(
    'wayland-scanner',
    'scanner.c',
    'dtddata.S',
    include_directories: [ root_inc, protocol_inc ],
    dependencies: [ expat_dep, libxml2_dep, wayland_util_dep, ],
    install: true,
  )
  wayland_scanner_code_arg = 'public-code'

  pkgconfig.generate(
    name: 'Wayland Scanner',
    description: 'Wayland scanner',
    version: meson.project_version(),
    variables: [
      'bindir=${prefix}/@0@'.format(get_option('bindir')),
      'datarootdir=${prefix}/@0@'.format(get_option('datadir')),
      'pkgdatadir=${prefix}/@0@/@1@'.format(get_option('datadir'), meson.project_name()),
      'wayland_scanner=${bindir}/wayland-scanner',
    ],
    filebase: 'wayland-scanner',
    install_dir: join_paths(get_option('prefix'), get_option('libdir'), 'pkgconfig'),
  )
endif

if get_option('libraries')
  mathlib_dep = cc.find_library('m', required: false)
  rt_dep = dependency('threads', required: false)

  wayland_protocol_xml = files('../protocol/wayland.xml')

  wayland_private = static_library(
    'wayland-private',
    sources: [
      'connection.c',
      'wayland-os.c',
    ],
    dependencies: [ ffi_dep, ],
  )

  wayland_private_dep = declare_dependency(
    link_with: wayland_private,
    include_directories: include_directories('.'),
  )

  generated_headers = [
    [ 'server header', ['server-header'], 'wayland-server-protocol.h', true, ],
    [ 'core server header', ['server-header', '-c'], 'wayland-server-protocol-core.h', false, ],
    [ 'client header', ['client-header'], 'wayland-client-protocol.h', true, ],
    [ 'core client header', ['client-header', '-c'], 'wayland-client-protocol-core.h', false, ],
  ]

  foreach gen: generated_headers
    target_name = gen[0]
    scanner_args = gen[1]
    output_file = gen[2]
    install_file = gen[3]
    install_dir = join_paths(get_option('prefix'), get_option('includedir'))
    var_name = output_file.underscorify()

    target = custom_target(
      target_name,
      command: [
        wayland_scanner, scanner_args, '@INPUT@', '@OUTPUT@',
      ],
      input: wayland_protocol_xml,
      output: output_file,
      install: install_file,
      install_dir: install_dir,
    )

    set_variable(var_name, target)
  endforeach

  wayland_protocol_c = custom_target('protocol source',
    command: [
      wayland_scanner, wayland_scanner_code_arg, '@INPUT@', '@OUTPUT@',
    ],
    input: wayland_protocol_xml,
    output: 'wayland-protocol.c',
  )

  wayland_server = library(
    'wayland-server',
    sources: [
      get_variable('wayland_server_protocol_core_h'),
      get_variable('wayland_server_protocol_h'),
      wayland_protocol_c,
      'wayland-server.c',
      'wayland-shm.c',
      'event-loop.c',
    ],
    version: '0.1.0',
    dependencies: [
      ffi_dep,
      wayland_private_dep,
      wayland_util_dep,
      mathlib_dep,
      rt_dep,
    ],
    include_directories: root_inc,
    install: true,
  )

  wayland_server_dep = declare_dependency(
    link_with: wayland_server,
    include_directories: [ root_inc, include_directories('.') ],
    dependencies: [ ffi_dep, mathlib_dep, rt_dep, ],
    sources: [
      get_variable('wayland_server_protocol_core_h'),
      get_variable('wayland_server_protocol_h'),
    ],
  )

  pkgconfig.generate(
    name: 'Wayland Server',
    description: 'Server side implementation of the Wayland protocol',
    version: meson.project_version(),
    libraries: wayland_server,
    filebase: 'wayland-server',
    install_dir: join_paths(get_option('prefix'), get_option('libdir'), 'pkgconfig'),
  )

  wayland_client = library(
    'wayland-client',
    sources: [
      get_variable('wayland_client_protocol_core_h'),
      get_variable('wayland_client_protocol_h'),
      wayland_protocol_c,
      'wayland-client.c',
    ],
    version: '0.3.0',
    dependencies: [
      ffi_dep,
      wayland_private_dep,
      wayland_util_dep,
      mathlib_dep,
      rt_dep,
    ],
    include_directories: root_inc,
    install: true,
  )

  pkgconfig.generate(
    name: 'Wayland Client',
    description: 'Wayland client side library',
    version: meson.project_version(),
    libraries: wayland_client,
    filebase: 'wayland-client',
    install_dir: join_paths(get_option('prefix'), get_option('libdir'), 'pkgconfig'),
  )

  wayland_client_dep = declare_dependency(
    link_with: wayland_client,
    include_directories: [ root_inc, include_directories('.') ],
    sources: [
      get_variable('wayland_client_protocol_core_h'),
      get_variable('wayland_client_protocol_h'),
    ],
  )

  install_headers([
    'wayland-util.h',
    'wayland-server.h',
    'wayland-server-core.h',
    'wayland-client.h',
    'wayland-client-core.h',
  ])
endif
