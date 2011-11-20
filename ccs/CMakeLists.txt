# vim: set sw=4: -*- mode: cmake; tab-width: 4; indent-tabs-mode: nil -*-
#
# Copyright (C) 2011 Marco Craveiro <marco.craveiro@gmail.com>
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of version 2 of the GNU General Public License as
# published by the Free Software Foundation.
#
# This program is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA
# 02110-1301 USA
#
cmake_minimum_required(VERSION 2.6 FATAL_ERROR)

# prohibit in-source builds: i.e. users are not allowed to perform
# builds either in the source root directory or in any of its
# sub-directories.
string(REGEX MATCH "^${CMAKE_SOURCE_DIR}" source_build ${CMAKE_BINARY_DIR})
if (source_build)
    message(FATAL_ERROR "Attempting to build in the source directory."
        "Please create a separate build directory and invoke cmake from there.")
endif()

project(tsdai)

#
# useful vars
#

# staging directory
set(stage_dir ${CMAKE_BINARY_DIR}/stage)
file(MAKE_DIRECTORY ${stage_dir})

# ensure cmake dumps binaries in the right places
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${stage_bin_dir})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${stage_bin_dir})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${stage_bin_dir})

# version info
set(TSDAI_MAJOR_VERSION 0)
set(TSDAI_MINOR_VERSION 0)
set(TSDAI_PATCH_VERSION 1)
set(TSDAI_VERSION "${TSDAI_MAJOR_VERSION}")
set(TSDAI_VERSION "${TSDAI_VERSION}.${TSDAI_MINOR_VERSION}")
set(TSDAI_VERSION "${TSDAI_VERSION}.${TSDAI_PATCH_VERSION}")

#
# useful variables
#

# chapters
set(chapters_dir "${CMAKE_SOURCE_DIR}/chapters")
set(chapters "")
file(GLOB chapters ABSOLUTE "${chapters_dir}/" "${chapters_dir}/*")

# templates
set(templates_dir "${CMAKE_SOURCE_DIR}/templates")
set(templates "")
file(GLOB templates ABSOLUTE "${templates_dir}/" "${templates_dir}/*")

# figures
set(figures_dir "${CMAKE_SOURCE_DIR}/figures")
set(figures "")
file(GLOB figures ABSOLUTE "${figures_dir}/" "${figures_dir}/*")

#
# targets
#

# copy chapters
add_custom_target(copy_chapters ALL cp ${chapters} ${stage_dir})
add_dependencies(copy_chapters DEPENDS ${chapters})

# copy templates
add_custom_target(copy_templates ALL cp ${templates} ${stage_dir})
add_dependencies(copy_templates DEPENDS ${templates})

# copy figures
add_custom_target(copy_figures ALL cp ${figures} ${stage_dir})
add_dependencies(copy_figures DEPENDS ${figures})

# process figures
add_custom_target(process_figures ALL
    epstopdf ${stage_dir}/*.eps)
add_dependencies(process_figures DEPENDS ${copy_figures})

# tex
add_custom_target(typeset ALL
    cd ${stage_dir} && pdflatex book.tex && pdflatex book.tex)
add_dependencies(typeset DEPENDS process_figures copy_chapters copy_templates)
