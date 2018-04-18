# Here be dragons
doxygen = find_program('doxygen')

dot = find_program('dot')
dot_gv = [
  [ 'wayland-architecture', files('dot/wayland-architecture.gv') ],
  [ 'x-architecture', files('dot/x-architecture.gv') ],
]

# This is a workaround for Meson's custom_target() directive, which
# currently does not support outputs pointing to a sub-directory
dot_png = []
dot_map = []
subdir('xml')

doxygen = configuration_data()
doxygen.set('VERSION', meson.project_version())
doxygen.set('top_builddir', meson.build_root())
wayland_doxygen = configure_file(
  input: 'wayland.doxygen.in',
  output: 'wayland.doxygen',
  configuration: doxygen,
)

shared_files = files([
  '../../src/wayland-util.h',
])

client_files = files([
  '../../src/wayland-client.c',
  '../../src/wayland-client.h',
  '../../src/wayland-client-core.h',
])

server_files = files([
  '../../src/event-loop.c',
  '../../src/wayland-server.c',
  '../../src/wayland-server.h',
  '../../src/wayland-server-core.h',
  '../../src/wayland-shm.c',
])

extra_client_files = [
  'mainpage.dox',
  get_variable('wayland_client_protocol_h'),
]

extra_server_files = [
  'mainpage.dox',
  get_variable('wayland_server_protocol_h'),
]

# formats = [ format_name, [ [ section_name, files ] ] ]
formats = [
  [
    'xml', [
      [ 'Client', shared_files + client_files, ],
      [ 'Server', shared_files + server_files, ],
    ]
  ],
  [
    'html', [
      [ 'Client', shared_files + client_files + extra_client_files, ],
      [ 'Server', shared_files + server_files + extra_server_files, ],
    ]
  ],
]

gen_doxygen = find_program('gen-doxygen.py')
foreach f: formats
  f_name = f[0]
  sections = f[1]

  foreach s: sections
    s_name = s[0]
    s_files = s[1]

    # We do not really need an output file, but Meson
    # will complain if one is not set, so we use a
    # dummy 'stamp' file
    custom_target('@0@-@1@-doc'.format(f_name, s_name),
      command: [
        gen_doxygen,
        '--builddir=@OUTDIR@',
        '--section=@0@'.format(s_name),
        '--output-format=@0@'.format(f_name),
        wayland_doxygen,
        '@INPUT@',
      ],
      input: s_files,
      output: '@0@-@1@-doc.stamp'.format(f_name, s_name),
      depends: [dot_png, dot_map],
      build_by_default: true,
    )
  endforeach
endforeach

man_files = shared_files + client_files + server_files
custom_target('man-pages',
  command: [
    gen_doxygen,
    '--builddir=@OUTDIR@',
    '--output-format=man',
    wayland_doxygen,
    '@INPUT@',
  ],
  input: man_files,
  output: 'man',
  build_by_default: true,
  install: true,
  install_dir: join_paths(get_option('prefix'), get_option('datadir')),
)
