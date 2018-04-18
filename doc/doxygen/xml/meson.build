# dot_png: list of PNG targets
# dot_map: list of MAP targets
foreach gv: dot_gv
  name = gv[0]
  infile = gv[1]

  dot_png += custom_target(name + '.png',
    command: [ dot, '-Tpng', '-o@OUTPUT@', '@INPUT@' ],
    input: infile,
    output: name + '.png',
  )

  dot_map += custom_target(name + '.map',
    command: [ dot, '-Tcmapx_np', '-o@OUTPUT@', '@INPUT@' ],
    input: infile,
    output: name + '.map',
  )
endforeach
