import os
import sys
import excons


env = excons.MakeBaseEnv()

staticlib = (excons.GetArgument("libtiff-static", 1, int) != 0)
usejbig = excons.GetArgument("libtiff-jbig", 0, int)

out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"

cfg_deps = []

cmake_opts = {"BUILD_SHARED_LIBS": (0 if staticlib else 1),
              "CMAKE_INSTALL_LIBDIR": "lib",
              # External codes
              "zlib": 1,
              "jpeg": 1,
              "old-jpeg": 1,
              "jpeg12": 0,
              "lzma": 0,
              "jbig": usejbig,
              "pixarlog": 1,
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
              "cxx": 0}

# ZLIB Setup ===================================================================

def ZlibLibname(static):
   return ("z" if sys.platform != "win32" else ("zlib" if static else "zdll"))

def ZlibDefines(static):
   return ([] if static else ["ZLIB_DLL"])

rv = excons.cmake.ExternalLibRequire(cmake_opts, name="zlib", libnameFunc=ZlibLibname, definesFunc=ZlibDefines)
if rv is None:
   excons.PrintOnce("Build zlib from sources ...")
   excons.Call("zlib", imp=["RequireZlib", "ZlibPath"])
   cfg_deps.append(excons.cmake.OutputsCachePath("zlib"))
   cmake_opts["ZLIB_LIBRARY"] = ZlibPath(static=True)
   cmake_opts["ZLIB_INCLUDE_DIR"] = out_incdir
   def ZlibRequire(env):
      RequireZlib(env, static=True)
else:
   ZlibRequire = rv

# JPEG Setup ===================================================================

def JpegLibname(static):
   name = "jpeg"
   if sys.platform == "win32" and static:
      name += "-static"
   return name

rv = excons.cmake.ExternalLibRequire(cmake_opts, name="jpeg", libnameFunc=JpegLibname)
if rv is None:
   excons.PrintOnce("Build jpeg from sources ...")
   excons.Call("libjpeg-turbo", imp=["RequireLibjpeg", "LibjpegPath"])
   if sys.platform == "win32":
      cfg_deps.append(excons.cmake.OutputsCachePath("libjpeg"))
   else:
      cfg_deps.append(excons.automake.OutputsCachePath("libjpeg"))
   cmake_opts["JPEG_LIBRARY"] = LibjpegPath(static=True)
   cmake_opts["JPEG_INCLUDE_DIR"] = out_incdir
   def JpegRequire(env):
      RequireLibjpeg(env, static=True)
else:
   JpegRequire = rv

# JBIG Setup ===================================================================

if usejbig:
   def JbigLibname(static):
      return "jbig"

   rv = excons.cmake.ExternalLibRequire(cmake_opts, name="jbig", libnameFunc=JbigLibname)
   if rv is None:
      excons.PrintOnce("Build jbig from sources ...")
      excons.Call("jbigkit", imp=["RequireJbig", "JbigPath"])
      libpath = JbigPath()
      cfg_deps.append(libpath)
      cmake_opts["JBIG_LIBRARY"] = libpath
      cmake_opts["JBIG_INCLUDE_DIR"] = out_incdir
      def JbigRequire(env):
         RequireJbig(env)
   else:
      JbigRequire = rv

# TIFF library ==================================================================

prjs = [
   {  "name": "libtiff",
      "type": "cmake",
      "cmake-opts": cmake_opts,
      "cmake-cfgs": excons.CollectFiles(".", patterns=["CMakeLists.txt"], recursive=True, exclude=["zlib", "libjpeg-turbo"]) + cfg_deps,
      "cmake-srcs": excons.CollectFiles("libtiff", patterns=["*.c"], recursive=True)
   }
]

excons.AddHelpOptions(libtiff="""TIFF OPTIONS
  libtiff-static=0|1  : Toggle between static and shared library build [1]
  libtiff-jbig=0|1    : Build with JBIG support [0]

%s\n\n%s\n\n%s""" % (excons.ExternalLibHelp("zlib"),
                 excons.ExternalLibHelp("jpeg"),
                 excons.ExternalLibHelp("jbig")))
excons.DeclareTargets(env, prjs)

# ==============================================================================

def LibtiffName():
   return "tiff"

def LibtiffPath():
   name = LibtiffName()
   if sys.platform == "win32":
      libname = name + ".lib"
   else:
      libname = "lib" + name + (".a" if staticlib else excons.SharedLibraryLinkExt())
   return out_libdir + "/" + libname

def RequireLibtiff(env):
   env.Append(CPPPATH=[out_incdir])
   env.Append(LIBPATH=[out_libdir])
   excons.Link(env, LibtiffName(), static=staticlib, force=True, silent=True)
   if staticlib:
      if usejbig:
         JbigRequire(env)
      JpegRequire(env)
      ZlibRequire(env)

Export("LibtiffName LibtiffPath RequireLibtiff")


