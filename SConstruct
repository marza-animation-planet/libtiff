import os
import sys
import excons


env = excons.MakeBaseEnv()

staticlib = (excons.GetArgument("libtiff-static", 1, int) != 0)

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
              "jbig": 1,
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
   zlibstatic = excons.GetArgument("zlib-static", 1, 0)
   cmake_opts["ZLIB_LIBRARY"] = ZlibPath(static=zlibstatic)
   cmake_opts["ZLIB_INCLUDE_DIR"] = out_incdir
   def ZlibRequire(env):
      RequireZlib(env, static=zlibstatic)
else:
   ZlibRequire = rv

# JPEG Setup ===================================================================

def JpegLibname(static):
   name = "jpeg"
   if sys.platform == "win32" and static:
      name += "-static"
   return name

rv = excons.cmake.ExternalLibRequire(cmake_opts, name="libjpeg", libnameFunc=JpegLibname)
if rv is None:
   excons.PrintOnce("Build libjpeg from sources ...")
   excons.Call("libjpeg-turbo", imp=["RequireLibjpeg", "LibjpegPath"])
   if sys.platform == "win32":
      cfg_deps.append(excons.cmake.OutputsCachePath("libjpeg"))
   else:
      cfg_deps.append(excons.automake.OutputsCachePath("libjpeg"))
   jpegstatic = excons.GetArgument("libjpeg-static", 1, 0)
   cmake_opts["JPEG_LIBRARY"] = LibjpegPath(static=jpegstatic)
   cmake_opts["JPEG_INCLUDE_DIR"] = out_incdir
   def JpegRequire(env):
      RequireLibjpeg(env, static=jpegstatic)
else:
   JpegRequire = rv

# JBIG Setup ===================================================================

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
  libtiff-static=0|1 : Toggle between static and shared library build [1]""")
excons.AddHelpOptions(ext_zlib=excons.ExternalLibHelp("zlib"))
excons.AddHelpOptions(ext_libjpeg=excons.ExternalLibHelp("libjpeg"))
excons.AddHelpOptions(ext_jbig=excons.ExternalLibHelp("jbig"))

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
      JbigRequire(env)
      JpegRequire(env)
      ZlibRequire(env)

Export("LibtiffName LibtiffPath RequireLibtiff")


