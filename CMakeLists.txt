######################################################################################################################
#
# JeVois Smart Embedded Machine Vision Toolkit - Copyright (C) 2016 by Laurent Itti, the University of Southern
# California (USC), and iLab at USC. See http://iLab.usc.edu and http://jevois.org for information about this project.
#
# This file is part of the JeVois Smart Embedded Machine Vision Toolkit.  This program is free software; you can
# redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software
# Foundation, version 2.  This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY;
# without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public
# License for more details.  You should have received a copy of the GNU General Public License along with this program;
# if not, write to the Free Software Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301, USA.
#
# Contact information: Laurent Itti - 3641 Watt Way, HNB-07A - Los Angeles, BA 90089-2520 - USA.
# Tel: +1 213 740 3527 - itti@pollux.usc.edu - http://iLab.usc.edu - http://jevois.org
######################################################################################################################

# CMake build rules for JeVois Base library and executables

# You may provide the installed JeVois config root and version as:
#   cmake -DJEVOIS_CONFIG=/jevois/config ..

cmake_minimum_required(VERSION 3.1)

# Set vendor names, our modules will be placed in a directory by that name under /jevois/modules:
set(JEVOIS_VENDOR "JeVois")
set(JEVOIS_CONFIG "/jevois/config" CACHE STRING "Path to JeVois config to use")

# Include our helper functions, config, etc from the JeVois install:
set(CMAKE_MODULE_PATH ${JEVOIS_CONFIG})
include(jevois_config)
include(JeVois)

# Set project name, detects compiler (which has been set by our helper module). Then set some complation flags:
project(jevoisbase)
jevois_project_set_flags()

# Add parent of our base source directory to the include search path to allow #include <jevoibase/x/y> and to create the
# proper makefile dependencies:
set(JVB "${CMAKE_CURRENT_SOURCE_DIR}")
include_directories("${JVB}/..")

# Setup our library using the source files in src/Components and call it jevoisbase with version 1.0:
jevois_setup_library(src/Components jevoisbase 1.0)

# Setup our modules that are in src/Modules, make them depend on jevoisbase:
jevois_setup_modules(src/Modules jevoisbase)

# Now setup any additional includes and sources that we will add to the jevoisbase library for use by our individual
# modules, typically those are in the Contrib/ drectory:

########################################################################################################################
# OpenGL-ES libraries on platform (and they also exist on host):
target_link_libraries(jevoisbase GLESv2 EGL)

########################################################################################################################
# Add any needed boost libraries that are not already pulled in by libjevois:
target_link_libraries(jevoisbase boost_regex)

########################################################################################################################
# tiny-cnn support:
include_directories(Contrib)
include_directories(Contrib/tiny-dnn)
include_directories(Contrib/NNPACK/include)
include_directories(Contrib/pthreadpool/include)

########################################################################################################################
# ZZBar barcode / QR-code source files:

include_directories(Contrib/ZBar/include Contrib/ZBar/zbar)
target_sources(jevoisbase PRIVATE ${JVB}/Contrib/ZBar/zbar/processor.c ${JVB}/Contrib/ZBar/zbar/scanner.c
  ${JVB}/Contrib/ZBar/zbar/symbol.c ${JVB}/Contrib/ZBar/zbar/img_scanner.c ${JVB}/Contrib/ZBar/zbar/qrcode/rs.c
  ${JVB}/Contrib/ZBar/zbar/qrcode/isaac.c ${JVB}/Contrib/ZBar/zbar/qrcode/util.c
  ${JVB}/Contrib/ZBar/zbar/qrcode/qrdectxt.c ${JVB}/Contrib/ZBar/zbar/qrcode/bch15_5.c
  ${JVB}/Contrib/ZBar/zbar/qrcode/binarize.c ${JVB}/Contrib/ZBar/zbar/qrcode/qrdec.c ${JVB}/Contrib/ZBar/zbar/config.c
  ${JVB}/Contrib/ZBar/zbar/error.c ${JVB}/Contrib/ZBar/zbar/processor/posix.c ${JVB}/Contrib/ZBar/zbar/processor/lock.c
  ${JVB}/Contrib/ZBar/zbar/processor/null.c ${JVB}/Contrib/ZBar/zbar/convert.c ${JVB}/Contrib/ZBar/zbar/decoder/i25.c
  ${JVB}/Contrib/ZBar/zbar/decoder/qr_finder.c ${JVB}/Contrib/ZBar/zbar/decoder/code128.c
  ${JVB}/Contrib/ZBar/zbar/decoder/codabar.c ${JVB}/Contrib/ZBar/zbar/decoder/code39.c
  ${JVB}/Contrib/ZBar/zbar/decoder/databar.c ${JVB}/Contrib/ZBar/zbar/decoder/ean.c
  ${JVB}/Contrib/ZBar/zbar/decoder/code93.c ${JVB}/Contrib/ZBar/zbar/image.c ${JVB}/Contrib/ZBar/zbar/refcnt.c
  ${JVB}/Contrib/ZBar/zbar/decoder.c)
# ${JVB}/Contrib/ZBar/zbar/decoder/pdf417.c

# FIXME need to debug zbar as it gives some warnings
add_definitions(-Wparentheses -w)

########################################################################################################################
# cvEyeTracker eye-tracking

target_sources(jevoisbase PRIVATE ${JVB}/Contrib/cvEyeTracker-1.2.5/ransac_ellipse.cpp
  ${JVB}/Contrib/cvEyeTracker-1.2.5/remove_corneal_reflection.cpp ${JVB}/Contrib/cvEyeTracker-1.2.5/svd.c)

########################################################################################################################
# Neon-accelerated NE10 support:
include_directories(Contrib/Ne10/inc)

target_sources(jevoisbase PRIVATE ${JVB}/Contrib/Ne10/modules/imgproc/NE10_boxfilter.c)
if (JEVOIS_PLATFORM)
  target_sources(jevoisbase PRIVATE ${JVB}/Contrib/Ne10/modules/imgproc/NE10_boxfilter.neon.c)
endif (JEVOIS_PLATFORM)

########################################################################################################################
# VLfeat support:
include_directories(Contrib/vlfeat/vl)

# Add VLfeat sources shared among various algorithms:
target_sources(jevoisbase PRIVATE ${JVB}/Contrib/vlfeat/vl/host.c ${JVB}/Contrib/vlfeat/vl/generic.c
  ${JVB}/Contrib/vlfeat/vl/imopv.c)

# The source code for SSE2 convolution seems to be missing...
add_definitions(-DVL_DISABLE_SSE2)

# Other defs to make VLfeat comile:
add_definitions(-DVL_COMPILER_GNUC -DVL_ARCH_LITTLE_ENDIAN)

# Add VLfeat sources used by DenseSift module:
target_sources(jevoisbase PRIVATE ${JVB}/Contrib/vlfeat/vl/dsift.c ${JVB}/Contrib/vlfeat/vl/sift.c)

########################################################################################################################
# Fast optical flow
include_directories(Contrib/OF_DIS)
target_sources(jevoisbase PRIVATE ${JVB}/Contrib/OF_DIS/oflow.cpp ${JVB}/Contrib/OF_DIS/refine_variational.cpp
  ${JVB}/Contrib/OF_DIS/patchgrid.cpp ${JVB}/Contrib/OF_DIS/patch.cpp ${JVB}/Contrib/OF_DIS/FDF1.0.1/image.c
  ${JVB}/Contrib/OF_DIS/FDF1.0.1/opticalflow_aux.c ${JVB}/Contrib/OF_DIS/FDF1.0.1/solver.c)

# Select mode 1 (optical flow) and 1 channel (grayscale):
add_definitions(-DSELECTMODE=1 -DSELECTCHANNEL=1)

# Fix path assumptions in the optical flow code for Eigen:
if (JEVOIS_PLATFORM)
  add_definitions(-I${JEVOIS_BUILDROOT_BASE}/host/usr/arm-buildroot-linux-gnueabihf/sysroot/usr/include/eigen3)
else (JEVOIS_PLATFORM)
  add_definitions(-I/usr/include/eigen3)
endif (JEVOIS_PLATFORM)
  
# Enable OpenMP, which can accelerate the fast optical flow code. NOTE: With small images and when runnign on a fast
# Intel-based host, this actually slows down the code by 10x or so, probably the parallelism overhead is not worth it
# for small images. Yet, we enable it here as it seems to help on thje platform:
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fopenmp")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fopenmp")
add_definitions(-DWITH_OPENMP -DUSE_PARALLEL_ON_FLOWAGGR)

########################################################################################################################
# Link to OpenCV libraries for superpixels, aruco, and others (from pkg-config --libs opencv and deleting a few):
target_link_libraries(jevoisbase opencv_stitching opencv_superres opencv_videostab opencv_aruco opencv_bgsegm
  opencv_bioinspired opencv_ccalib opencv_dnn opencv_dpm opencv_fuzzy opencv_line_descriptor opencv_optflow
  opencv_plot opencv_reg opencv_saliency opencv_stereo opencv_structured_light opencv_rgbd
  opencv_surface_matching opencv_tracking opencv_datasets opencv_text opencv_face opencv_xfeatures2d opencv_shape
  opencv_video opencv_ximgproc opencv_calib3d opencv_features2d opencv_flann opencv_xobjdetect opencv_objdetect
  opencv_ml opencv_xphoto opencv_highgui opencv_videoio opencv_imgcodecs opencv_photo opencv_imgproc opencv_core)

########################################################################################################################
# Documentation:

add_custom_target(doc COMMAND doxygen "${CMAKE_CURRENT_SOURCE_DIR}/doc/doxygen.cfg"
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})

# This is to install the doc of jevois and jevoisbase to jevois.org, only works in ilab:
add_custom_target(docweb
  COMMAND ./scripts/docinstall.sh
  DEPENDS doc
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})

# Documentation files for our install / distribution package
set(DOC_FILES README INSTALL COPYING)
set(DOC_PATH "share/doc/jevoisbase")
install(FILES ${DOC_FILES} DESTINATION ${DOC_PATH} COMPONENT bin)

# Create packages (Debian, RPM): in build/, just type 'cpack' to create the package.
# To list the files created in a package, run: dpkg --contents <package.deb>
set(CPACK_PACKAGE_DESCRIPTION "JeVoisBase Embedded Machine Vision Modules")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY
  "JeVoisBase Embedded Machine Vision Modules from iLab at the University of Southern California")
set(CPACK_PACKAGE_NAME "jevoisbase")

set(CPACK_PACKAGE_CONTACT "Laurent Itti")
set(CPACK_PACKAGE_VENDOR "iLab at the University of Southern California")
set(CPACK_PACKAGE_VERSION_MAJOR "${JEVOIS_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${JEVOIS_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "0")
set(VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")
set(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA "${CMAKE_SOURCE_DIR}/scripts/postinst;${CMAKE_SOURCE_DIR}/scripts/prerm;")

set(CPACK_GENERATOR "DEB;")  # could be DEB;RPM;
set(CPACK_PACKAGE_FILE_NAME
  "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}-${CMAKE_SYSTEM_PROCESSOR}")
SET(CPACK_SOURCE_IGNORE_FILES "${CMAKE_BINARY_DIR}/*")
include(CPack)
