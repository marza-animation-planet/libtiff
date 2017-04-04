import os
import sys
import excons
import excons.tools.zlib

env = excons.MakeBaseEnv()

out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"

tiff_deps = []

staticlib = (excons.GetArgument("libtiff-static", 1, int) != 0)

# ZLIB Setup ===================================================================

# Allow custom zlib setup using flags:
#   with-zlib, with-zlib-inc, with-zlib-lib, zlib-libname, zlib-libsuffix, zlib-static

zlib_help = "EXTERNAL "+excons.tools.zlib.GetOptionsString()
zlib_path = None
zlib_incdir, zlib_libdir = excons.GetDirs("zlib")

if zlib_incdir and zlib_libdir:
   zlib_libname = excons.GetArgument("zlib-libname", None)
   zlib_static = (excons.GetArgument("zlib-static", 0, int) != 0)

   if zlib_libname is None:
      zlib_libname = ("z" if sys.platform != "win32" else ("zlib" if zlib_static else "zdll"))
      zlib_libname += excons.GetArgument("zlib-libsuffix", "")

   if sys.platform == "win32":
      basename = zlib_libname + ".lib"
   else:
      basename = "lib" + zlib_libname + (".a" if zlib_static else ".so")

   zlib_path = "%s/%s" % (zlib_libdir, basename)
   if not os.path.isfile(zlib_path):
      zlib_path = None
      if sys.platform != "win32" and excons.arch_dir == "x64":
         zlib_path = "%s64/%s" % (zlib_libdir, basename)
         if not os.path.isfile(zlib_path):
            zlib_path = None

   # Setup defines in environment to be picked up by configure
   if zlib_path and not zlib_static:
      cflags = os.environ.get("CFLAGS", "") + " -DZLIB_DLL"
      os.environ["CFLAGS"] = cflags

   def RequireZlib(env):
      env.Append(CPPPATH=[zlib_incdir])
      env.Append(LIBPATH=[zlib_libdir])
      if not zlib_static:
         env.Append(CPPDEFINES=["ZLIB_DLL"])
         env.Append(LIBS=[zlib_libname])
      else:
         if not excons.StaticallyLink(env, zlib_libname, silent=True):
            env.Append(LIBS=[zlib_libname])

if zlib_path is None:
   excons.PrintOnce("Build zlib from sources ...")
   excons.Call("zlib", imp=["RequireZlib", "ZlibName"])
   zlib_incdir, zlib_libdir, zlib_path = out_incdir, out_libdir, ZlibName(static=True)
   tiff_deps.append("zlib")

# JPEG Setup ===================================================================

# Allow custom jpeg setup using flags:
#   with-jpeg, with-jpeg-inc, with-jpeg-lib, jpeg-libname, jpeg-libsuffix, jpeg-static

jpeg_help = """EXTERNAL JPEG OPTIONS
  with-jpeg=<path>     : Jpeg prefix.
  with-jpeg-inc=<path> : Jpeg headers directory.           [<prefix>/include]
  with-jpeg-lib=<path> : Jpeg libraries directory.         [<prefix>/lib]
  jpeg-static=0|1      : Link Jpeg statically.             [0]
  jpeg-libname=<str>   : Override Jpeg library name.       []
  jpeg-libsuffix=<str> : Default Jpeg library name suffix. []
                         (ignored when jpeg-libname is set)
                         (default name is 'jpeg' on osx and linux, 'jpeg-static' (static) or 'jpeg' (shared) on windows)"""
jpeg_path = None
jpeg_incdir, jpeg_libdir = excons.GetDirs("jpeg")

if jpeg_incdir and jpeg_libdir:
   jpeg_libname = excons.GetArgument("jpeg-libname", None)
   jpeg_static = (excons.GetArgument("jpeg-static", 0, int) != 0)

   if jpeg_libname is None:
      jpeg_libname = ("jpeg" if sys.platform != "win32" else ("jpeg-static" if jpeg_static else "jpeg"))
      jpeg_libname += excons.GetArgument("jpeg-libsuffix", "")

   if sys.platform == "win32":
      basename = jpeg_libname + ".lib"
   else:
      basename = "lib" + jpeg_libname + (".a" if jpeg_static else ".so")

   jpeg_path = "%s/%s" % (jpeg_libdir, basename)
   if not os.path.isfile(jpeg_path):
      jpeg_path = None
      if sys.platform != "win32" and excons.arch_dir == "x64":
         jpeg_path = "%s64/%s" % (jpeg_libdir, basename)
         if not os.path.isfile(jpeg_path):
            jpeg_path = None

   def RequireLibjpeg(env):
      env.Append(CPPPATH=[jpeg_incdir])
      env.Append(LIBPATH=[jpeg_libdir])
      if not jpeg_static:
         env.Append(LIBS=[jpeg_libname])
      else:
         if not excons.StaticallyLink(env, jpeg_libname, silent=True):
            env.Append(LIBS=[jpeg_libname])

if jpeg_path is None:
   excons.PrintOnce("Build jpeg from sources ...")
   excons.Call("libjpeg-turbo", imp=["RequireLibjpeg", "LibjpegName"])
   jpeg_incdir, jpeg_libdir, jpeg_path = out_incdir, out_libdir, LibjpegName(static=True)
   tiff_deps.append("libjpeg")

# JBIG Setup ===================================================================

jbig_help = """EXTERNAL JBIG OPTIONS
  with-jbig=<path>     : Jbig prefix.
  with-jbig-inc=<path> : Jbig headers directory.           [<prefix>/include]
  with-jbig-lib=<path> : Jbig libraries directory.         [<prefix>/lib]
  jbig-static=0|1      : Link Jbig statically.             [0]
  jbig-libname=<str>   : Override Jbig library name.       []
  jbig-libsuffix=<str> : Default Jbig library name suffix. []
                         (ignored when jbig-libname is set)
                         (default name is 'jbig')"""
jbig_path = ""
jbig_incdir = ""
jbig_libdir = ""

if excons.GetArgument("libtiff-jbig", 0, int):
   jbig_path = None
   jbig_incdir, jbig_libdir = excons.GetDirs("jbig")

   if jbig_incdir and jbig_libdir:
      jbig_libname = excons.GetArgument("jbig-libname", None)
      jbig_static = (excons.GetArgument("jbig-static", 0, int) != 0)

      if jbig_libname is None:
         jbig_libname = "jbig"
         jbig_libname += excons.GetArgument("jbig-libsuffix", "")

      if sys.platform == "win32":
         basename = jbig_libname + ".lib"
      else:
         basename = "lib" + jbig_libname + (".a" if jbig_static else ".so")

      jbig_path = "%s/%s" % (jbig_libdir, basename)
      if not os.path.isfile(jbig_path):
         jbig_path = None
         if sys.platform != "win32" and excons.arch_dir == "x64":
            jbig_path = "%s64/%s" % (jbig_libdir, basename)
            if not os.path.isfile(jbig_path):
               jbig_path = None

      def RequireJbig(env):
         env.Append(CPPPATH=[jbig_incdir])
         env.Append(LIBPATH=[jbig_libdir])
         if not jbig_static:
            env.Append(LIBS=[jbig_libname])
         else:
            if not excons.StaticallyLink(env, jbig_libname, silent=True):
               env.Append(LIBS=[jbig_libname])

   if jbig_path is None:
      excons.PrintOnce("Build jbig from sources ...")
      excons.Call("jbigkit", imp=["RequireJbig", "JbigName"])
      jbig_incdir, jbig_libdir, jbig_path = out_incdir, out_libdir, JbigName(static=True)
      tiff_deps.append("jbig")

else:
   def RequireJbig(env):
      pass

# TIFF library ==================================================================

prjs = [
   {  "name": "libtiff",
      "type": "cmake",
      "cmake-opts": {"BUILD_SHARED_LIBS": (0 if staticlib else 1),
                     "CMAKE_INSTALL_LIBDIR": "lib",
                     # External codes
                     "zlib": 1,     # zlib library
                     "ZLIB_INCLUDE_DIR": zlib_incdir,
                     "ZLIB_LIBRARY": zlib_path,
                     "jpeg": 1,     # jpeg library
                     "old-jpeg": 1, # requires jpeg support
                     "JPEG_INCLUDE_DIR": jpeg_incdir,
                     "JPEG_LIBRARY": jpeg_path,
                     "jpeg12": 0,   # 12bit jpeg support (JPEG12_* alternative library)
                     "lzma": 0,     # lzma library
                     "jbig": excons.GetArgument("libtiff-jbig", 0, int),
                     "JBIG_INCLUDE_DIR": jbig_incdir,
                     "JBIG_LIBRARY": jbig_path,
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
                     "cxx": 0},
      "cmake-srcs": excons.CollectFiles("libtiff", patterns=["*.c"], recursive=True),
      "deps": tiff_deps
   }
]

excons.AddHelpOptions(libtiff="""TIFF OPTIONS
  libtiff-static=0|1  : Toggle between static and shared library build [1]
  libtiff-jbig=0|1    : Build with JBIG support [0]

%s\n%s\n%s""" % (zlib_help, jpeg_help, jbig_help))
excons.DeclareTargets(env, prjs)

# ==============================================================================

def RequireLibtiff(env):
   env.Append(CPPPATH=[out_incdir])
   env.Append(LIBPATH=[out_libdir])
   if staticlib:
      if sys.platform == "win32":
         env.Append(LIBS=["tiff"])               
      else:
         if not env.StaticallyLink(env, "tiff", silent=True):
            env.Append(LIBS=["tiff"])
      RequireJbig(env)
      RequireLibjpeg(env)
      RequireZlib(env)
   else:
      env.Append(LIBS=["tiff"])

def LibtiffName():
   if sys.platform == "win32":
      basename = "tiff.lib"
   else:
      basename = "libtiff.a"
   return out_dir + "/" + basename

Export("RequireLibtiff LibtiffName")


