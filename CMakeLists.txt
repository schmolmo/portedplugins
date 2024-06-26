####################################################################################################
# CMakeLists file for Chen
# Generated by Mads Kjeldgaard
# 2021-02-19
####################################################################################################

####################################################################################################
# basic project config
cmake_minimum_required(VERSION 3.8)
set(project_name "PortedPlugins")
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake_modules ${CMAKE_MODULE_PATH})
set(CMAKE_OSX_DEPLOYMENT_TARGET "10.10" CACHE STRING "Minimum OS X deployment version")

set(CMAKE_CXX_STANDARD 20)

# Add CPM
set(CPM_DOWNLOAD_VERSION 0.36.0)
set(CPM_DOWNLOAD_LOCATION "${CMAKE_BINARY_DIR}/cmake/CPM_${CPM_DOWNLOAD_VERSION}.cmake")

if(NOT (EXISTS ${CPM_DOWNLOAD_LOCATION}))
	message(STATUS "Downloading CPM.cmake")
	file(DOWNLOAD https://github.com/TheLartians/CPM.cmake/releases/download/v${CPM_DOWNLOAD_VERSION}/CPM.cmake ${CPM_DOWNLOAD_LOCATION})
endif()

include(${CPM_DOWNLOAD_LOCATION})



####################################################################################################
# load modules
include(SuperColliderServerPlugin RESULT_VARIABLE server_plugin_found)
if(NOT server_plugin_found)
  message(FATAL_ERROR "Could not find server plugin functions module")
endif()

include(SuperColliderCompilerConfig RESULT_VARIABLE compiler_config_found)
if(NOT compiler_config_found)
  message(FATAL_ERROR "Could not find compiler config module")
endif()

# Windows - puts redistributable DLLs in install directory
include(InstallRequiredSystemLibraries)

sc_check_sc_path("${SC_PATH}")
message(STATUS "Found SuperCollider: ${SC_PATH}")
set(
  SC_PATH "${SC_PATH}" CACHE PATH
  "Path to SuperCollider source. Relative paths are treated as relative to this script" FORCE
)

include("${SC_PATH}/SCVersion.txt")
message(STATUS "Building plugins for SuperCollider version: ${SC_VERSION}")

# set project here to avoid SCVersion.txt clobbering our version info
project(${project_name})
sc_do_initial_compiler_config() # do after setting project so compiler ID is available

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT OR NOT CMAKE_INSTALL_PREFIX)
  message(WARNING "No install prefix provided, defaulting to $BUILD_DIR/install")
  set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install" CACHE PATH "Install prefix" FORCE)
endif()

message(STATUS "Install directory set to: ${CMAKE_INSTALL_PREFIX}")

# include_directories(${CMAKE_CURRENT_SOURCE_DIR}/DaisySP/Source/Dynamics)

####################################################################################################
# options
option(SUPERNOVA "Build plugins for supernova" ON)
option(SCSYNTH "Build plugins for scsynth" ON)
option(NATIVE "Optimize for native architecture" OFF)
option(STRICT "Use strict warning flags" OFF)
option(NOVA_SIMD "Build plugins with nova-simd support." ON)

# Exclude / include certain experimental / work in progress plugins
option(USE_DAISYSP_LIB "Build plugins that depend on the DaisySP library" ON)
option(INCPULSE "Include AnalogPulseShaper plugin" OFF)
option(INCWERNER "Include Werner plugin" OFF)
option(INCPHASORMODAL "Include PhasorModal plugin" ON)

####################################################################################################
# include libraries

if(NOVA_SIMD)
  add_definitions(-DNOVA_SIMD)
  include_directories(${SC_PATH}/external_libraries/nova-simd)
endif()

####################################################################################################
# DAISYSP LIBRARY
####################################################################################################

# Link plugin with DaisySP library
function(sc_link_with_daisy plugin_name)
  message(STATUS "Linking ${plugin_name} with DaisySP library")
  if(SCSYNTH)
    target_include_directories("${plugin_name}_scsynth" PUBLIC include/daisysp)
    target_link_libraries("${plugin_name}_scsynth" DaisySP)
  endif()

  if(SUPERNOVA)
    target_include_directories("${plugin_name}_supernova" PUBLIC include/daisysp)
    target_link_libraries("${plugin_name}_supernova" DaisySP)
  endif()
endfunction()

####################################################################################################
# Plugins
####################################################################################################

# Install additional help files
install(
  DIRECTORY "plugins/HelpSource"
  DESTINATION "${project_name}/HelpSource"
)

# LPG
set(
  LPG_cpp_files
  plugins/LPG/LPG.cpp
  plugins/LPG/LPG.hpp
)

set(
  LPG_sc_files
  plugins/LPG/LPG.sc
)

set(
  LPG_schelp_files
  plugins/LPG/LPG.schelp
)

sc_add_server_plugin(
  "${project_name}" # desination directory
  "LPG" # target name
  "${LPG_cpp_files}"
  "${LPG_sc_files}"
  "${LPG_schelp_files}"
)

# Chen
set(
  Chen_cpp_files
  plugins/Chen/Chen.cpp
  plugins/Chen/Chen.hpp
)
set(
  Chen_sc_files
  plugins/Chen/Chen.sc
)
set(
  Chen_schelp_files
  plugins/Chen/Chen.schelp
)

sc_add_server_plugin(
  "${project_name}" # desination directory
  "Chen" # target name
  "${Chen_cpp_files}"
  "${Chen_sc_files}"
  "${Chen_schelp_files}"
)
# Rongs
set(
  Rongs_cpp_files
  plugins/Rongs/Rongs.hpp
  plugins/Rongs/Rongs.cpp
)
set(
  Rongs_sc_files
  plugins/Rongs/Rongs.sc
)
set(
  Rongs_schelp_files
  plugins/Rongs/Rongs.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "Rongs" # target name
  "${Rongs_cpp_files}"
  "${Rongs_sc_files}"
  "${Rongs_schelp_files}"
)

# AnalogPhaser
set(
  AnalogPhaser_cpp_files
  plugins/ChowDSP/AnalogPhaser.hpp
  plugins/ChowDSP/AnalogPhaser.cpp
)
set(
  AnalogPhaser_sc_files
  plugins/ChowDSP/AnalogPhaser.sc
)
set(
  AnalogPhaser_schelp_files
  plugins/ChowDSP/AnalogPhaser.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "AnalogPhaser" # target name
  "${AnalogPhaser_cpp_files}"
  "${AnalogPhaser_sc_files}"
  "${AnalogPhaser_schelp_files}"
)

# AnalogPhaserMod
set(
  AnalogPhaserMod_cpp_files
  plugins/ChowDSP/AnalogPhaserMod.hpp
  plugins/ChowDSP/AnalogPhaserMod.cpp
)
set(
  AnalogPhaserMod_sc_files
  plugins/ChowDSP/AnalogPhaserMod.sc
)
set(
  AnalogPhaserMod_schelp_files
  plugins/ChowDSP/AnalogPhaserMod.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "AnalogPhaserMod" # target name
  "${AnalogPhaserMod_cpp_files}"
  "${AnalogPhaserMod_sc_files}"
  "${AnalogPhaserMod_schelp_files}"
)

if(INCPHASORMODAL)
  # PhasorModal
  set(
    PhasorModal_cpp_files
    plugins/ChowDSP/PhasorModal.hpp
    plugins/ChowDSP/PhasorModal.cpp
  )
  set(
    PhasorModal_sc_files
    plugins/ChowDSP/PhasorModal.sc
  )
  set(
    PhasorModal_schelp_files
    plugins/ChowDSP/PhasorModal.schelp
  )

  sc_add_server_plugin(
    "${project_name}" # destination directory
    "PhasorModal" # target name
    "${PhasorModal_cpp_files}"
    "${PhasorModal_sc_files}"
    "${PhasorModal_schelp_files}"
  )
endif()

if(INCPULSE)
  # AnalogPulseShaper
  set(
    AnalogPulseShaper_cpp_files
    plugins/ChowDSP/AnalogPulseShaper.hpp
    plugins/ChowDSP/AnalogPulseShaper.cpp
  )
  set(
    AnalogPulseShaper_sc_files
    plugins/ChowDSP/AnalogPulseShaper.sc
  )
  set(
    AnalogPulseShaper_schelp_files
    plugins/ChowDSP/AnalogPulseShaper.schelp
  )

  sc_add_server_plugin(
    "${project_name}" # destination directory
    "AnalogPulseShaper" # target name
    "${AnalogPulseShaper_cpp_files}"
    "${AnalogPulseShaper_sc_files}"
    "${AnalogPulseShaper_schelp_files}"
  )
endif()

# AnalogTape
set(
  AnalogTape_cpp_files
  plugins/ChowDSP/AnalogTape.hpp
  plugins/ChowDSP/AnalogTape.cpp
  # plugins/ChowDSP/HysteresisProcessing.hpp
  # plugins/ChowDSP/HysteresisProcessing.cpp
)
set(
  AnalogTape_sc_files
  plugins/ChowDSP/AnalogTape.sc
)
set(
  AnalogTape_schelp_files
  plugins/ChowDSP/AnalogTape.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "AnalogTape" # target name
  "${AnalogTape_cpp_files}"
  "${AnalogTape_sc_files}"
  "${AnalogTape_schelp_files}"
)

# AnalogVintageDistortion
set(
  AnalogVintageDistortion_cpp_files
  plugins/ChowDSP/AnalogVintageDistortion.hpp
  plugins/ChowDSP/AnalogVintageDistortion.cpp
)
set(
  AnalogVintageDistortion_sc_files
  plugins/ChowDSP/AnalogVintageDistortion.sc
)
set(
  AnalogVintageDistortion_schelp_files
  plugins/ChowDSP/AnalogVintageDistortion.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "AnalogVintageDistortion" # target name
  "${AnalogVintageDistortion_cpp_files}"
  "${AnalogVintageDistortion_sc_files}"
  "${AnalogVintageDistortion_schelp_files}"
)



# AnalogLoss
set(
  AnalogLoss_cpp_files
  plugins/ChowDSP/AnalogLoss.hpp
  plugins/ChowDSP/AnalogLoss.cpp
)
set(
  AnalogLoss_sc_files
  plugins/ChowDSP/AnalogLoss.sc
)
set(
  AnalogLoss_schelp_files
  plugins/ChowDSP/AnalogLoss.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "AnalogLoss" # target name
  "${AnalogLoss_cpp_files}"
  "${AnalogLoss_sc_files}"
  "${AnalogLoss_schelp_files}"
)



# AnalogDegrade
set(
  AnalogDegrade_cpp_files
  plugins/ChowDSP/AnalogDegrade.hpp
  plugins/ChowDSP/AnalogDegrade.cpp
)
set(
  AnalogDegrade_sc_files
  plugins/ChowDSP/AnalogDegrade.sc
)
set(
  AnalogDegrade_schelp_files
  plugins/ChowDSP/AnalogDegrade.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "AnalogDegrade" # target name
  "${AnalogDegrade_cpp_files}"
  "${AnalogDegrade_sc_files}"
  "${AnalogDegrade_schelp_files}"
)


# AnalogChew
set(
  AnalogChew_cpp_files
  plugins/ChowDSP/AnalogChew.hpp
  plugins/ChowDSP/AnalogChew.cpp
)
set(
  AnalogChew_sc_files
  plugins/ChowDSP/AnalogChew.sc
)
set(
  AnalogChew_schelp_files
  plugins/ChowDSP/AnalogChew.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "AnalogChew" # target name
  "${AnalogChew_cpp_files}"
  "${AnalogChew_sc_files}"
  "${AnalogChew_schelp_files}"
)

# AnalogFoldOsc
set(
  AnalogFoldOsc_cpp_files
  plugins/AnalogFoldOsc/AnalogFoldOsc.hpp
  plugins/AnalogFoldOsc/AnalogFoldOsc.cpp
  plugins/AnalogFoldOsc/buchla259cell.cpp
  plugins/AnalogFoldOsc/buchla259cell.hpp
  plugins/AnalogFoldOsc/buchla259waveshaper.cpp
  plugins/AnalogFoldOsc/buchla259waveshaper.hpp
)

set(
  AnalogFoldOsc_sc_files
  plugins/AnalogFoldOsc/AnalogFoldOsc.sc
)
set(
  AnalogFoldOsc_schelp_files
  plugins/AnalogFoldOsc/AnalogFoldOsc.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "AnalogFoldOsc" # target name
  "${AnalogFoldOsc_cpp_files}"
  "${AnalogFoldOsc_sc_files}"
  "${AnalogFoldOsc_schelp_files}"
)

# VadimFilter
set(
  VadimFilter_cpp_files
  plugins/VadimFilter/VadimFilter.hpp
  plugins/VadimFilter/VadimFilter.cpp
)
set(
  VadimFilter_sc_files
  plugins/VadimFilter/VadimFilter.sc
)
set(
  VadimFilter_schelp_files
  plugins/VadimFilter/VadimFilter.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "VadimFilter" # target name
  "${VadimFilter_cpp_files}"
  "${VadimFilter_sc_files}"
  "${VadimFilter_schelp_files}"
)
# LockhartWavefolder
set(
  LockhartWavefolder_cpp_files
  plugins/LockhartWavefolder/LockhartCell.cpp
  plugins/LockhartWavefolder/LockhartCell.hpp
  plugins/LockhartWavefolder/LockhartWavefolder.cpp
  plugins/LockhartWavefolder/LockhartWavefolder.hpp
)

set(
  LockhartWavefolder_sc_files
  plugins/LockhartWavefolder/LockhartWavefolder.sc
)
set(
  LockhartWavefolder_schelp_files
  plugins/LockhartWavefolder/LockhartWavefolder.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "LockhartWavefolder" # target name
  "${LockhartWavefolder_cpp_files}"
  "${LockhartWavefolder_sc_files}"
  "${LockhartWavefolder_schelp_files}"
)
# NonlinearFilter
set(
  NonlinearFilter_cpp_files
  plugins/NonlinearFilter/Biquad.h
  plugins/NonlinearFilter/EQFilter.cpp
  plugins/NonlinearFilter/EQFilter.h
  plugins/NonlinearFilter/FBFilter.h
  plugins/NonlinearFilter/NonlinearFilter.cpp
  plugins/NonlinearFilter/NonlinearFilter.hpp
  plugins/NonlinearFilter/Saturators.h
)

set(
  NonlinearFilter_sc_files
  plugins/NonlinearFilter/NonlinearFilter.sc
)
set(
  NonlinearFilter_schelp_files
  plugins/NonlinearFilter/NonlinearFilter.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "NonlinearFilter" # target name
  "${NonlinearFilter_cpp_files}"
  "${NonlinearFilter_sc_files}"
  "${NonlinearFilter_schelp_files}"
)
# VADiodeFilter
set(
  VADiodeFilter_cpp_files
  plugins/odinfilters/DiodeFilter.h
  plugins/odinfilters/DiodeFilter.cpp
  plugins/odinfilters/VAOnePoleFilter.h
  plugins/odinfilters/VAOnePoleFilter.cpp
  plugins/VADiodeFilter/VADiodeFilter.hpp
  plugins/VADiodeFilter/VADiodeFilter.cpp
)
set(
  VADiodeFilter_sc_files
  plugins/VADiodeFilter/VADiodeFilter.sc
)
set(
  VADiodeFilter_schelp_files
  plugins/VADiodeFilter/VADiodeFilter.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "VADiodeFilter" # target name
  "${VADiodeFilter_cpp_files}"
  "${VADiodeFilter_sc_files}"
  "${VADiodeFilter_schelp_files}"
)

# VAKorg35
set(
  VAKorg35_cpp_files
  plugins/VAKorg35/VAKorg35.hpp
  plugins/VAKorg35/VAKorg35.cpp
  plugins/odinfilters/Korg35Filter.h
  plugins/odinfilters/Korg35Filter.cpp
  plugins/odinfilters/VAOnePoleFilter.h
  plugins/odinfilters/VAOnePoleFilter.cpp
)
set(
  VAKorg35_sc_files
  plugins/VAKorg35/VAKorg35.sc
)
set(
  VAKorg35_schelp_files
  plugins/VAKorg35/VAKorg35.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "VAKorg35" # target name
  "${VAKorg35_cpp_files}"
  "${VAKorg35_sc_files}"
  "${VAKorg35_schelp_files}"
)

# VALadder
set(
  VALadder_cpp_files
  plugins/VALadder/VALadder.hpp
  plugins/VALadder/VALadder.cpp
  plugins/odinfilters/LadderFilter.h
  plugins/odinfilters/LadderFilter.cpp
  plugins/odinfilters/VAOnePoleFilter.h
  plugins/odinfilters/VAOnePoleFilter.cpp
)
set(
  VALadder_sc_files
  plugins/VALadder/VALadder.sc
)
set(
  VALadder_schelp_files
  plugins/VALadder/VALadder.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "VALadder" # target name
  "${VALadder_cpp_files}"
  "${VALadder_sc_files}"
  "${VALadder_schelp_files}"
)

# VASEM12
set(
  VASEM12_cpp_files
  plugins/VASEM12/VASEM12.hpp
  plugins/VASEM12/VASEM12.cpp
  plugins/odinfilters/SEMFilter12.h
  plugins/odinfilters/SEMFilter12.cpp
  plugins/odinfilters/VAOnePoleFilter.h
  plugins/odinfilters/VAOnePoleFilter.cpp
)
set(
  VASEM12_sc_files
  plugins/VASEM12/VASEM12.sc
)
set(
  VASEM12_schelp_files
  plugins/VASEM12/VASEM12.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "VASEM12" # target name
  "${VASEM12_cpp_files}"
  "${VASEM12_sc_files}"
  "${VASEM12_schelp_files}"
)

# VA1Pole
set(
  VA1Pole_cpp_files
  plugins/VA1Pole/VA1Pole.hpp
  plugins/VA1Pole/VA1Pole.cpp
  plugins/odinfilters/VAOnePoleFilter.h
  plugins/odinfilters/VAOnePoleFilter.cpp
)

set(
  VA1Pole_sc_files
  plugins/VA1Pole/VA1Pole.sc
)
set(
  VA1Pole_schelp_files
  plugins/VA1Pole/VA1Pole.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "VA1Pole" # target name
  "${VA1Pole_cpp_files}"
  "${VA1Pole_sc_files}"
  "${VA1Pole_schelp_files}"
)

# Fverb
set(
  Fverb_cpp_files
  plugins/Fverb/fverb.cpp
)
set(
  Fverb_sc_files
  plugins/Fverb/Fverb.sc
)
set(
  Fverb_schelp_files
  plugins/Fverb/Fverb.schelp
)

sc_add_server_plugin(
  "${project_name}" # destination directory
  "Fverb" # target name
  "${Fverb_cpp_files}"
  "${Fverb_sc_files}"
  "${Fverb_schelp_files}"
)



#--------------------------------------------------------------------#
#                      DAISY-DEPENDENT-PLUGINS                       #
#--------------------------------------------------------------------#

# Only run if USE_DAISYSP_LIB is set to ON
if(USE_DAISYSP_LIB)

	# Print message to let user know these are being built
	message(STATUS "Adding DaisySP plugins")

	# Add DaisySP library
	CPMAddPackage(
		NAME daisysp
		GITHUB_REPOSITORY "electro-smith/DaisySP"
		GIT_TAG v0.0.2
		)

	include_directories("include/daisysp")

	# VarShapeOsc
	set(
		VarShapeOsc_cpp_files
		plugins/VarShapeOsc/VarShapeOsc.hpp
		plugins/VarShapeOsc/VarShapeOsc.cpp
		)
	set(
		VarShapeOsc_sc_files
		plugins/VarShapeOsc/VarShapeOsc.sc
		)
	set(
		VarShapeOsc_schelp_files
		plugins/VarShapeOsc/VarShapeOsc.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"VarShapeOsc" # target name
		"${VarShapeOsc_cpp_files}"
		"${VarShapeOsc_sc_files}"
		"${VarShapeOsc_schelp_files}"
		)
	sc_link_with_daisy("VarShapeOsc")

	# VosimOsc
	set(
		VosimOsc_cpp_files
		plugins/VosimOsc/VosimOsc.hpp
		plugins/VosimOsc/VosimOsc.cpp
		)
	set(
		VosimOsc_sc_files
		plugins/VosimOsc/VosimOsc.sc
		)
	set(
		VosimOsc_schelp_files
		plugins/VosimOsc/VosimOsc.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"VosimOsc" # target name
		"${VosimOsc_cpp_files}"
		"${VosimOsc_sc_files}"
		"${VosimOsc_schelp_files}"
		)

	sc_link_with_daisy("VosimOsc")

	# ZOsc
	set(
		ZOsc_cpp_files
		plugins/ZOsc/ZOsc.hpp
		plugins/ZOsc/ZOsc.cpp
		)
	set(
		ZOsc_sc_files
		plugins/ZOsc/ZOsc.sc
		)
	set(
		ZOsc_schelp_files
		plugins/ZOsc/ZOsc.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"ZOsc" # target name
		"${ZOsc_cpp_files}"
		"${ZOsc_sc_files}"
		"${ZOsc_schelp_files}"
		)
	sc_link_with_daisy("ZOsc")

	# AnalogBassDrum
	set(
		AnalogBassDrum_cpp_files
		plugins/AnalogBassDrum/AnalogBassDrum.hpp
		plugins/AnalogBassDrum/AnalogBassDrum.cpp
		)
	set(
		AnalogBassDrum_sc_files
		plugins/AnalogBassDrum/AnalogBassDrum.sc
		)
	set(
		AnalogBassDrum_schelp_files
		plugins/AnalogBassDrum/AnalogBassDrum.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"AnalogBassDrum" # target name
		"${AnalogBassDrum_cpp_files}"
		"${AnalogBassDrum_sc_files}"
		"${AnalogBassDrum_schelp_files}"
		)
	sc_link_with_daisy("AnalogBassDrum")

	# AnalogSnareDrum
	set(
		AnalogSnareDrum_cpp_files
		plugins/AnalogSnareDrum/AnalogSnareDrum.hpp
		plugins/AnalogSnareDrum/AnalogSnareDrum.cpp
		)
	set(
		AnalogSnareDrum_sc_files
		plugins/AnalogSnareDrum/AnalogSnareDrum.sc
		)
	set(
		AnalogSnareDrum_schelp_files
		plugins/AnalogSnareDrum/AnalogSnareDrum.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"AnalogSnareDrum" # target name
		"${AnalogSnareDrum_cpp_files}"
		"${AnalogSnareDrum_sc_files}"
		"${AnalogSnareDrum_schelp_files}"
		)
	sc_link_with_daisy("AnalogSnareDrum")
	# StringVoice
	set(
		StringVoice_cpp_files
		plugins/StringVoice/StringVoice.hpp
		plugins/StringVoice/StringVoice.cpp
		)
	set(
		StringVoice_sc_files
		plugins/StringVoice/StringVoice.sc
		)
	set(
		StringVoice_schelp_files
		plugins/StringVoice/StringVoice.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"StringVoice" # target name
		"${StringVoice_cpp_files}"
		"${StringVoice_sc_files}"
		"${StringVoice_schelp_files}"
		)
	sc_link_with_daisy("StringVoice")

	# NeoFormant
	set(
		NeoFormant_cpp_files
		plugins/NeoFormant/NeoFormant.hpp
		plugins/NeoFormant/NeoFormant.cpp
		)
	set(
		NeoFormant_sc_files
		plugins/NeoFormant/NeoFormant.sc
		)
	set(
		NeoFormant_schelp_files
		plugins/NeoFormant/NeoFormant.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"NeoFormant" # target name
		"${NeoFormant_cpp_files}"
		"${NeoFormant_sc_files}"
		"${NeoFormant_schelp_files}"
		)
	sc_link_with_daisy("NeoFormant")

	# Resonator
	set(
		Resonator_cpp_files
		plugins/Resonator/Resonator.hpp
		plugins/Resonator/Resonator.cpp
		)
	set(
		Resonator_sc_files
		plugins/Resonator/Resonator.sc
		)
	set(
		Resonator_schelp_files
		plugins/Resonator/Resonator.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"Resonator" # target name
		"${Resonator_cpp_files}"
		"${Resonator_sc_files}"
		"${Resonator_schelp_files}"
		)
	sc_link_with_daisy("Resonator")
	# NeoVarSawOsc
	set(
		NeoVarSawOsc_cpp_files
		plugins/NeoVarSawOsc/NeoVarSawOsc.hpp
		plugins/NeoVarSawOsc/NeoVarSawOsc.cpp
		)
	set(
		NeoVarSawOsc_sc_files
		plugins/NeoVarSawOsc/NeoVarSawOsc.sc
		)
	set(
		NeoVarSawOsc_schelp_files
		plugins/NeoVarSawOsc/NeoVarSawOsc.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"NeoVarSawOsc" # target name
		"${NeoVarSawOsc_cpp_files}"
		"${NeoVarSawOsc_sc_files}"
		"${NeoVarSawOsc_schelp_files}"
		)
	sc_link_with_daisy("NeoVarSawOsc")

	# OscBank
	set(
		OscBank_cpp_files
		plugins/OscBank/OscBank.hpp
		plugins/OscBank/OscBank.cpp
		)
	set(
		OscBank_sc_files
		plugins/OscBank/OscBank.sc
		)
	set(
		OscBank_schelp_files
		plugins/OscBank/OscBank.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"OscBank" # target name
		"${OscBank_cpp_files}"
		"${OscBank_sc_files}"
		"${OscBank_schelp_files}"
		)
	sc_link_with_daisy("OscBank")
	# HarmonicOsc
	set(
		HarmonicOsc_cpp_files
		plugins/HarmonicOsc/HarmonicOsc.hpp
		plugins/HarmonicOsc/HarmonicOsc.cpp
		)
	set(
		HarmonicOsc_sc_files
		plugins/HarmonicOsc/HarmonicOsc.sc
		)
	set(
		HarmonicOsc_schelp_files
		plugins/HarmonicOsc/HarmonicOsc.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"HarmonicOsc" # target name
		"${HarmonicOsc_cpp_files}"
		"${HarmonicOsc_sc_files}"
		"${HarmonicOsc_schelp_files}"
		)
	sc_link_with_daisy("HarmonicOsc")
	# BLOsc
	set(
		BLOsc_cpp_files
		plugins/BLOsc/BLOsc.hpp
		plugins/BLOsc/BLOsc.cpp
		)
	set(
		BLOsc_sc_files
		plugins/BLOsc/BLOsc.sc
		)
	set(
		BLOsc_schelp_files
		plugins/BLOsc/BLOsc.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"BLOsc" # target name
		"${BLOsc_cpp_files}"
		"${BLOsc_sc_files}"
		"${BLOsc_schelp_files}"
		)
	sc_link_with_daisy("BLOsc")
	# DCompressor
	set(
		DCompressor_cpp_files
		plugins/DCompressor/DCompressor.hpp
		plugins/DCompressor/DCompressor.cpp
		)
	set(
		DCompressor_sc_files
		plugins/DCompressor/DCompressor.sc
		)
	set(
		DCompressor_schelp_files
		plugins/DCompressor/DCompressor.schelp
		)

	sc_add_server_plugin(
		"${project_name}" # destination directory
		"DCompressor" # target name
		"${DCompressor_cpp_files}"
		"${DCompressor_sc_files}"
		"${DCompressor_schelp_files}"
		)

	sc_link_with_daisy("DCompressor")
endif()

#--------------------------------------------------------------------#
