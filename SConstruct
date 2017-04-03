import os
import sys
import excons

env = excons.MakeBaseEnv()

out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"

staticlib = (excons.GetArgument("static", 1, int) != 0)

excons.Call("libjpeg-turbo")
excons.Call("zlib")

zlibname = ("zlibstatic.lib" if sys.platform == "win32" else "libz.a")
jpegname = ("jpeg-static.lib" if sys.platform == "win32" else "libjpeg.a")


if not env.CMakeConfigure("libtiff", opts={"BUILD_SHARED_LIBS": (0 if staticlib else 1),
                                           "CMAKE_INSTALL_LIBDIR": "lib",
                                           # External codes
                                           "zlib": 1,     # zlib library
                                           "ZLIB_INCLUDE_DIR": out_incdir,
                                           "ZLIB_LIBRARY": out_libdir + "/" + zlibname,
                                           "jpeg": 1,     # jpeg library
                                           "old-jpeg": 1, # requires jpeg support
                                           "JPEG_INCLUDE_DIR": out_incdir,
                                           "JPEG_LIBRARY": out_libdir + "/" + jpegname,
                                           "jpeg12": 0,   # 12bit jpeg support (JPEG12_* alternative library)
                                           "lzma": 0,     # lzma library
                                           "jbig": 0,     # jbig library
                                           "pixarlog": 1, # requires zlib support
                                           # Internal codes
                                           "ccitt": 1,
                                           "packbits": 1,
                                           "lzw": 1,
                                           "thunder": 1,
                                           "next": 1,
                                           "logluv": 1,
                                           # other options
                                           "gl": 0,
                                           "doc": 0,
                                           "mdi": 1,
                                           "cxx": 0}):
   sys.exit(1)

target = env.CMake(env.CMakeOutputs(),
                   env.CMakeInputs(dirs=["libtiff"],
                                   patterns=[".h", ".hxx", ".c", ".cmake.in"]))

env.Alias("libtiff", target)
env.Depends(target, "zlib")
env.Depends(target, "libjpeg")

env.CMakeClean()

excons.SyncCache()


def RequireLibtiff(env):
   env.Append(CPPPATH=[out_incdir])
   env.Append(LIBPATH=[out_libdir])
   if staticlib:
      if sys.platform != "win32":
         env.StaticallyLink(env, "tiff", silent=True)
         env.StaticallyLink(env, "jpeg", silent=True)
         env.StaticallyLink(env, "z", silent=True)
      else:
         env.Append(LIBS=["tiff", "jpeg-static", "zlibstatic"])
   else:
      env.Append(LIBS=["tiff"])

Export("RequireLibtiff")


