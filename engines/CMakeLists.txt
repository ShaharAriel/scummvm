cmake_minimum_required(VERSION 3.12.3)
project(ScummVM)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
find_package(PkgConfig QUIET)
include(CMakeParseArguments)

set(SCUMMVM_LIBS)

macro(find_feature)
	set(_OPTIONS_ARGS)
	set(_ONE_VALUE_ARGS name findpackage_name include_dirs_var libraries_var)
	set(_MULTI_VALUE_ARGS pkgconfig_name libraries)
	cmake_parse_arguments(_feature "${_OPTIONS_ARGS}" "${_ONE_VALUE_ARGS}" "${_MULTI_VALUE_ARGS}" ${ARGN})

	if (_feature_pkgconfig_name AND PKG_CONFIG_FOUND)
		pkg_check_modules(${_feature_name} REQUIRED ${_feature_pkgconfig_name})
		include_directories(${${_feature_name}_INCLUDE_DIRS})
		list(APPEND SCUMMVM_LIBS ${${_feature_name}_LIBRARIES})
	endif()

	if (NOT ${_feature_name}_FOUND)
		if (_feature_findpackage_name)
			find_package(${_feature_findpackage_name} REQUIRED)
		endif()
		if (_feature_include_dirs_var)
			include_directories(${${_feature_include_dirs_var}} REQUIRED)
		endif()
		if (_feature_libraries_var)
			list(APPEND SCUMMVM_LIBS ${${_feature_libraries_var}})
		endif()
		if (_feature_libraries)
			list(APPEND SCUMMVM_LIBS ${_feature_libraries})
		endif()
	endif()
endmacro()

function(add_engine engine_name)
	string(TOUPPER ${engine_name} _engine_var)
	set(_enable_engine_var "ENABLE_${_engine_var}")
	if(NOT ${_enable_engine_var})
		return()
	endif()
	foreach(arg IN LISTS ARGN)
		list(APPEND eng_files ${engine_name}/${arg})
	endforeach()
	add_library(${engine_name} ${eng_files})

	# Generate definitions for the engine
	add_definitions(-D${_enable_engine_var})
	foreach(SUB_ENGINE IN LISTS SUB_ENGINES_${_engine_var})
		add_definitions(-DENABLE_${SUB_ENGINE})
	endforeach(SUB_ENGINE)
	file(APPEND "engines/plugins_table.h" "#if PLUGIN_ENABLED_STATIC(${_engine_var})\n")
	file(APPEND "engines/plugins_table.h" "LINK_PLUGIN(${_engine_var})\n")
	file(APPEND "engines/plugins_table.h" "#endif\n")

	# Enable C++11
	set_property(TARGET ${engine_name} PROPERTY CXX_STANDARD 11)
	set_property(TARGET ${engine_name} PROPERTY CXX_STANDARD_REQUIRED ON)

	# Link against the engine
	target_link_libraries(scummvm ${engine_name})
endfunction()

# Define the engines and subengines
set(ENGINES SCUMM ACCESS ADL AGI AGOS AGS ASYLUM BBVS BLADERUNNER BURIED CGE CGE2 CINE COMPOSER CRUISE CRYOMNI3D DIRECTOR DRACI DRAGONS DRASCULA DREAMWEB GLK GNAP GOB GRIFFON GRIM GROOVIE HADESCH HDB HOPKINS HUGO HYPNO ILLUSIONS KYRA LAB LURE MADE MADS MOHAWK MORTEVIELLE MYST3 NEVERHOOD NGI PARALLACTION PEGASUS PETKA PLUMBERS PRINCE PRIVATE QUEEN SAGA SCI SHERLOCK SKY STARK SUPERNOVA SWORD1 SWORD2 SWORD25 TEENAGENT TINSEL TITANIC TOLTECS TONY TOON TOUCHE TRECISION TSAGE TUCKER TWINE ULTIMA VOYEUR WINTERMUTE XEEN ZVISION)
set(SUB_ENGINES_SCUMM SCUMM_7_8 HE)
set(SUB_ENGINES_AGOS AGOS2)
set(SUB_ENGINES_CRYOMNI3D VERSAILLES)
set(SUB_ENGINES_GRIM MONKEY4)
set(SUB_ENGINES_GROOVIE GROOVIE2)
set(SUB_ENGINES_KYRA LOL EOB)
set(SUB_ENGINES_MADS)
set(SUB_ENGINES_MOHAWK MYST MYSTME RIVEN)
set(SUB_ENGINES_SAGA IHNM)
set(SUB_ENGINES_SCI SCI32)
set(SUB_ENGINES_WINTERMUTE FOXTAIL HEROCRAFT WME3D)

# Generate options for the engines
foreach(ENGINE IN LISTS ENGINES)
	option(ENABLE_${ENGINE} "Enable ${ENGINE}" ON)
endforeach(ENGINE)

set(ScummVM_SOURCE_DIR C:/Users/sariel/Projects/Fun/ScummVM/scummvm)
message(${ScummVM_SOURCE_DIR})
include_directories(${ScummVM_SOURCE_DIR} ${ScummVM_SOURCE_DIR}/engines $ENV{SCUMMVM_LIBS}/include .)

# Libraries and features

find_feature(name sdl pkgconfig_name sdl2 findpackage_name SDL2 include_dirs_var IGNORE libraries_var SDL2_LIBRARIES libraries IGNORE)
# Depending on how SDL2 was built, there can be either and imported target or flags variables
# Define the flags variables from the imported target if necessary
if (TARGET SDL2::SDL2)
	get_target_property(SDL2_INCLUDE_DIRS SDL2::SDL2 INTERFACE_INCLUDE_DIRECTORIES)
	get_target_property(SDL2_LIBRARIES SDL2::SDL2 LOCATION)
endif()
include_directories(${SDL2_INCLUDE_DIRS})

find_feature(name zlib pkgconfig_name zlib findpackage_name ZLIB include_dirs_var ZLIB_INCLUDE_DIRS libraries_var ZLIB_LIBRARIES libraries IGNORE)
add_definitions(-DUSE_ZLIB)
find_feature(name mad pkgconfig_name mad findpackage_name IGNORE include_dirs_var IGNORE libraries_var IGNORE libraries mad)
add_definitions(-DUSE_MAD)
find_feature(name fribidi pkgconfig_name fribidi findpackage_name IGNORE include_dirs_var IGNORE libraries_var IGNORE libraries fribidi)
add_definitions(-DUSE_FRIBIDI)
find_feature(name ogg pkgconfig_name ogg findpackage_name IGNORE include_dirs_var IGNORE libraries_var IGNORE libraries ogg)
add_definitions(-DUSE_OGG)
find_feature(name vorbis pkgconfig_name vorbisfile vorbis findpackage_name IGNORE include_dirs_var IGNORE libraries_var IGNORE libraries vorbisfile vorbis)
add_definitions(-DUSE_VORBIS)
find_feature(name flac pkgconfig_name flac findpackage_name IGNORE include_dirs_var IGNORE libraries_var IGNORE libraries FLAC)
add_definitions(-DUSE_FLAC)
find_feature(name png pkgconfig_name libpng findpackage_name PNG include_dirs_var PNG_INCLUDE_DIRS libraries_var PNG_LIBRARIES libraries IGNORE)
add_definitions(-DUSE_PNG)
find_feature(name mpeg2 pkgconfig_name libmpeg2 findpackage_name MPEG2 include_dirs_var MPEG2_INCLUDE_DIRS libraries_var MPEG2_mpeg2_LIBRARY libraries IGNORE)
add_definitions(-DUSE_MPEG2)
find_feature(name theoradec pkgconfig_name theoradec findpackage_name IGNORE include_dirs_var IGNORE libraries_var IGNORE libraries theoradec)
add_definitions(-DUSE_THEORADEC)
find_feature(name freetype2 pkgconfig_name freetype2 findpackage_name Freetype include_dirs_var FREETYPE_INCLUDE_DIRS libraries_var FREETYPE_LIBRARIES libraries IGNORE)
add_definitions(-DUSE_FREETYPE2)
find_feature(name jpeg pkgconfig_name libjpeg findpackage_name JPEG include_dirs_var JPEG_INCLUDE_DIRS libraries_var JPEG_LIBRARIES libraries IGNORE)
add_definitions(-DUSE_JPEG)
find_feature(name fluidsynth pkgconfig_name fluidsynth findpackage_name IGNORE include_dirs_var IGNORE libraries_var IGNORE libraries fluidsynth)
add_definitions(-DUSE_FLUIDSYNTH)
find_feature(name libcurl pkgconfig_name libcurl findpackage_name CURL include_dirs_var CURL_INCLUDE_DIRS libraries_var CURL_LIBRARIES libraries IGNORE)
add_definitions(-DUSE_LIBCURL)
find_feature(name sdlnet pkgconfig_name SDL2_net findpackage_name IGNORE include_dirs_var IGNORE libraries_var IGNORE libraries SDL2_net)
add_definitions(-DUSE_SDL_NET)
add_definitions(-DUSE_BINK)
add_definitions(-DUSE_SCALERS)
add_definitions(-DUSE_HQ_SCALERS)
add_definitions(-DUSE_EDGE_SCALERS)
add_definitions(-DUSE_ASPECT)
add_definitions(-DUSE_RGB_COLOR)
add_definitions(-DUSE_HIGHRES)
add_definitions(-DUSE_MT32EMU)
add_definitions(-DUSE_LUA)
add_definitions(-DUSE_TINYGL)
find_feature(name opengl pkgconfig_name IGNORE findpackage_name OpenGL include_dirs_var OPENGL_INCLUDE_DIR libraries_var OPENGL_gl_LIBRARY libraries IGNORE)
add_definitions(-DUSE_OPENGL)
add_definitions(-DUSE_OPENGL_GAME)
add_definitions(-DUSE_OPENGL_SHADERS)
add_definitions(-DUSE_TASKBAR)
add_definitions(-DUSE_CLOUD)
add_definitions(-DUSE_TRANSLATION)
add_definitions(-DUSE_SYSDIALOGS)
add_definitions(-DUSE_DETECTLANG)
add_definitions(-DUSE_TTS)
add_definitions(-DBUILTIN_RESOURCES)
add_definitions(-DDETECTION_FULL)
add_definitions(-DUSE_DETECTION_FEATURES_STATIC)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-long-long -Wno-multichar -Wno-unknown-pragmas -Wno-reorder -Wpointer-arith -Wcast-qual -Wcast-align -Wshadow -Wnon-virtual-dtor -Wwrite-strings -fno-exceptions -fcheck-new")
if (WIN32)
	add_definitions(-DWIN32)
else()
	add_definitions(-DPOSIX)
endif()
add_definitions(-DSDL_BACKEND)
add_definitions(-DUSE_SDL2)
add_definitions(-DUSE_GLAD)
add_definitions(-DDETECTION_STATIC)
# Generate "engines/plugins_table.h"
file(REMOVE "engines/plugins_table.h")
file(APPEND "engines/plugins_table.h" "/* This file is automatically generated by CMake */\n")

list(APPEND SCUMMVM_LIBS scummvm-detection)
add_library(scummvm-detection
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/access.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/asurface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/bubble_box.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/char.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/data.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/decompress.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/files.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/player.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/resources.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/room.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/scripts.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/access/video.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/adl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/adl_v2.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/adl_v3.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/adl_v4.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/adl_v5.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/disk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/disk_image_helpers.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/display.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/display_a2.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/adl/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/agi.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/appleIIgs_timedelay_overwrite.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/inv.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/keyboard.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/logic.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/lzw.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/mouse_cursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/opcodes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/palette.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/picture.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/preagi.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/preagi_mickey.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/preagi_troll.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/preagi_winnie.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/sound_2gs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/sound_coco3.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/sound_midi.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/sound_pcjr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/sound_sarien.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/sprite.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/systemui.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/view.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/wagparser.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/wagparser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agi/words.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/agos.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/intern.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/intern_detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/midi.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/obsolete.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/agos/vga.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ags/achievements_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ags/ags.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ags/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ags/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ags/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ags/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ags/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ags/game_scanner.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ags/globals.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ags/metaengine.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ags/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ags/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/asylum/asylum.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/asylum/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/asylum/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/asylum/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/asylum/eventhandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/asylum/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/asylum/respack.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/asylum/shared.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/asylum/staticres.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/avalanche.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/avalot.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/background.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/clock.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/closing.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/dropdown.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/enums.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/ghostroom.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/help.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/highscore.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/mainmenu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/nim.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/parser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/sequence.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/shootemup.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/avalanche/timer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bbvs/bbvs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bbvs/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bbvs/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bbvs/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bbvs/gamemodule.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bbvs/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bbvs/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bbvs/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bbvs/spritemodule.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/actor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/actor_clues.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/actor_combat.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/actor_dialogue_queue.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/actor_walk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/adpcm_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/ambient_sounds.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/archive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/aud_stream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/audio_cache.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/audio_mixer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/audio_player.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/audio_speech.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/bladerunner.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/boundingbox.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/chapters.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/color.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/combat.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/crimes_database.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/decompress_lcw.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/decompress_lzo.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/dialogue_menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/fog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/framelimiter.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/game_constants.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/game_flags.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/game_info.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/image.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/item.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/item_pickup.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/items.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/light.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/lights.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/matrix.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/mouse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/movement_track.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/obstacles.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/outtake.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/overlays.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/rect_float.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/regions.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/savefile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/scene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/scene_objects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/screen_effects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/set.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/set_effects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/settings.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/shape.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/slice_animations.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/slice_renderer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/subtitles.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/suspects_database.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/text_resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/time.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/vector.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/view.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/vqa_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/vqa_player.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/waypoints.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/bladerunner/zbuffer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/agent_evaluation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/aidata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/animdata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/avi_frames.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/biochip_right.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/biochip_view.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/bookdata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/buried.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/complete.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/credits.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/death.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/fbcdata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/frame_window.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/gameui.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/global_flags.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/inndata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/invdata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/inventory_info.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/inventory_window.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/livetext.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/main_menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/message.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/navarrow.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/navdata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/overview.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/resources.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/scene_view.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/snddata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/sprtdata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/title_sequence.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/video_window.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/buried/window.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/bitmap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/cge.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/cge_main.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/fileio.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/game.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/general.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/snail.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/talk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/vga13h.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/vmenu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge/walk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/bitmap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/cge2.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/cge2_main.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/fileio.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/general.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/hero.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/map.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/snail.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/spare.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/talk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/vga13h.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cge2/vmenu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/ani_dat.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/atds.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/chewy.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/cursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/data.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/defines.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/detail.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/effect.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/gedclass.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/globals.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/main.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/mcga.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/mcga_graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/memory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/menus.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/mouse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/movclass.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/ngsdefs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/ngshext.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/ngstypes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/object.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/object_extra.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/room.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/timer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/chewy/types.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/anim.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/bg.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/bg_list.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/cine.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/gfx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/main_loop.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/msg.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/object.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/pal.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/part.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/prc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/rel.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/texte.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/unpack.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cine/various.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/composer/composer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/composer/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/composer/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/composer/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/composer/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/composer/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/composer/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/composer/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/actor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/background.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/backgroundIncrust.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/cell.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/cruise.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/cruise_main.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/ctp.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/dataLoader.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/function.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/gfxModule.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/linker.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/mainDraw.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/mouse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/object.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/overlay.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/perso.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/polys.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/stack.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/staticres.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/various.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/vars.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cruise/volume.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryo/cryo.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryo/cryolib.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryo/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryo/defs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryo/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryo/eden.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryo/eden_graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryo/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryo/platdefs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryo/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/cryomni3d.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/datstream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/dialogs_manager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/fixed_image.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/font_manager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/mouse_boxes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/objects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/omni3d.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/sprites.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/cryomni3d/wam_parser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/archive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/cast.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/castmember.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/channel.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/cursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/detection_paths.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/director.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/frame.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/graphics-data.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/images.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/movie.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/score.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/sprite.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/stxt.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/types.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/util.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/director/window.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/champion.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/dialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/dm.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/dungeonman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/eventman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/gfx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/group.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/loadsave.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/lzw.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/menus.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/movesens.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/objectman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/projexpl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/sounds.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dm/timeline.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/barchive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/draci.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/game.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/mouse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/sprite.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/surface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/draci/walking.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/actor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/actorfiles.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/actorresource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/background.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/bag.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/bigfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/credits.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/cursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/cutscene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/dragonflg.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/dragonimg.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/dragonini.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/dragonobd.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/dragonrms.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/dragons.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/dragonvar.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/midimusicplayer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/minigame1.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/minigame2.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/minigame3.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/minigame4.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/minigame5.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/scene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/scriptopcodes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/sequenceopcodes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/specialopcodes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/strplayer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/talk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dragons/vabsound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/drascula/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/drascula/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/drascula/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/drascula/drascula.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/drascula/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dreamweb/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dreamweb/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dreamweb/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dreamweb/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dreamweb/dreamweb.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dreamweb/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dreamweb/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dreamweb/structs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/blorb.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/conf.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/fonts.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/game_description.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/glk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/glk_api.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/glk_types.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/jumps.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/pc_speaker.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/picture.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/quetzal.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/raw_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/selection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/speech.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/streams.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/time.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/unicode.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/unicode_gen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/utils.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/window_graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/window_pair.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/window_text_buffer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/window_text_grid.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/glk/windows.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gnap/character.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gnap/datarchive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gnap/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gnap/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gnap/fontdata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gnap/gamesys.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gnap/gnap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gnap/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gnap/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gnap/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gnap/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_adi2.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_adi4.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_adibou.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_ajworld.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_bargon.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_crousti.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_dynasty.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_fallback.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_fascin.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_geisha.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_gob1.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_gob2.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_gob3.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_inca2.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_lit.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_littlered.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_onceupon.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_playtoons.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_urban.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_ween.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/detection/tables_woodruff.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/anifile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/aniobject.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/backbuffer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/cheater.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/cmpfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/databases.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/dataio.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/dbase.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/decfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/draw.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/expression.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/game.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/global.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/gob.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/goblin.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/hotspots.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/iniconfig.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/init.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/inter.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/map.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/mult.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/palanim.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/resources.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/rxyfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/scenery.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/surface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/totfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/util.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/variables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/video.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/gob/videoplayer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/griffon/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/griffon/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/griffon/griffon.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/griffon/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/actor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/bitmap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/color.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/colormap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/costume.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/gfx_base.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/gfx_opengl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/gfx_opengl_shaders.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/gfx_tinygl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/grim.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/inputdialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/iris.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/keyframe.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/lab.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/lipsync.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/localize.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/lua.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/lua_v1.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/material.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/md5check.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/md5checkdialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/model.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/object.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/objectstate.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/patchr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/pool.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/primitives.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/registry.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/savegame.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/sector.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/set.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/sprite.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/textobject.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/grim/textsplit.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/cursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/groovie.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/lzss.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/groovie/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/ambient.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/baptr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/enums.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/event.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/gfx_context.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/hadesch.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/herobelt.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/hotzone.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/persistent.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/pod_file.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/pod_image.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/table.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/tag_file.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hadesch/video.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/ai-player.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/ai.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/file-manager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/gfx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/hdb.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/input.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/lua-script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/map.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/mpc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hdb/window.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/anim.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/computer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/files.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/globals.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/hopkins.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/lines.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/objects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hopkins/talk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/display.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/file.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/game.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/hugo.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/intro.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/mouse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/object.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/parser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/route.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/schedule.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hugo/util.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hypno/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hypno/grammar.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hypno/hypno.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hypno/libfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hypno/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hypno/tokens_arc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/hypno/tokens_mis.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/actor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/actor_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/actor_view_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/animation_mega_set.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/barriers.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/bone.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/breath.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/cluster_manager_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/configfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/custom_logics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/debug_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/direct_input.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/drawpoly_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/event_list.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/event_manager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/event_timer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/floors.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/fn_routines.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/game_script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/game_volume.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/gameover.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/global_objects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/global_objects_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/global_objects_psx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/global_switches.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/global_vars.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/graphic_prims.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/icb.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/icon_list.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/icon_list_manager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/icon_menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/icon_menu_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/jpeg_decode.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/keyboard.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/light.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/light_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/line_of_sight.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/loadscrn.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/main_menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/map_marker.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/mission.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/mission_functions.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/mouse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/movie_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/non_ad_module.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/object_structs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/options_manager_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/p4.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/p4_generic.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/p4_generic_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/player.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/prim_route_builder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/protocol.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/remora.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/remora_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/remora_sprite.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/res_man.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/res_man_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/route_manager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/session.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/set.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/set_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/shade.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/shade_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/shadow.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/shadow_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/shake.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/softskin_pc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/sound_logic.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/sound_logic_entry.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/sound_lowlevel.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/speech.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/stage_draw.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/stage_view.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/string_vest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/surface_manager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/text_sprites.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/timer_func.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/tracer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/icb/zsupport.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/actor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/camera.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/cursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/dictionary.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/fileresourcereader.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/fixedpoint.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/gamarchive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/gamestate.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/gamresourcereader.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/illusions.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/input.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/menusystem.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/pathfinder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/resourcereader.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/resourcesystem.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/screentext.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/scriptopcodes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/scriptstack.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/sequenceopcodes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/specialcode.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/textdrawer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/thread.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/time.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/illusions/updatefunctions.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/kingdom/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/kingdom/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/kingdom/kingdom.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/kingdom/logic.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/kingdom/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/kyra/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/kyra/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/kyra/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/kyra/kyra_v1.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/kyra/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/anim.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/dispman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/eventman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/image.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/interface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/intro.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/lab.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/labsets.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/processroom.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/speciallocks.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lab/utils.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lastexpress/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lastexpress/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lastexpress/drawable.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lastexpress/eventhandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lastexpress/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lastexpress/helpers.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lastexpress/lastexpress.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lastexpress/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lastexpress/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lastexpress/shared.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lilliput/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lilliput/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lilliput/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lilliput/lilliput.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lilliput/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lilliput/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lilliput/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lilliput/stream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/animseq.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/decode.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/disk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/fights.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/game.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/hotspots.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/intro.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/lure.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/luredefs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/memory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/palette.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/res.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/res_struct.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/room.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/scripts.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/strings.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/lure/surface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/container.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/controls.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/dialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/gui.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/hufflists.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/image.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/macventure.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/prebuilt_dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/stringtable.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/windows.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/macventure/world.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/database.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/made.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/pmvplayer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/redreader.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/screenfx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/scriptfuncs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/made/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/action.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/assets.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/audio.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/camera.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/compression.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/conversations.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/game.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/game_data.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/globals.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/hotspots.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/mads.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/menu_views.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/messages.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/msurface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/palette.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/player.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/rails.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/resources.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/scene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/scene_data.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/sequence.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/sprites.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/staticres.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mads/user_interface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/bitmap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/cstime.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/cstime_cases.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/cstime_game.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/cstime_graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/cstime_ui.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/cstime_view.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/cursors.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/installer_archive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/livingbooks.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/livingbooks_code.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/livingbooks_graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/livingbooks_lbx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/mohawk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/myst.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/myst_actions.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/myst_areas.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/myst_card.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/myst_graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/myst_metaengine.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/myst_scripts.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/myst_sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/myst_state.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/resource_cache.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/riven.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/riven_actions.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/riven_card.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/riven_graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/riven_graphics_detection_enums.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/riven_inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/riven_metaengine.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/riven_saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/riven_scripts.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/riven_sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/riven_stack.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/riven_video.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/video.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mohawk/view.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/mortevielle.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/mouse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/outtext.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mortevielle/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/animationdecoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/assets.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/conversationlinelist.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/encryptedfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/game.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/gamedata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/gamescreen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/guiscreen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/hardcodedstrings.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/inventoryitemdefinitionlist.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/mutationofjb.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/room.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/timer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/mutationofjb/util.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/ambient.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/archive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/cursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/database.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/effects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/gfx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/gfx_opengl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/gfx_opengl_shaders.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/gfx_opengl_texture.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/gfx_tinygl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/gfx_tinygl_texture.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/hotspot.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/movie.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/myst3.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/node.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/nodecube.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/nodeframe.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/puzzles.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/scene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/state.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/subtitles.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/myst3/transition.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/commontypes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/constants.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/cursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/decompress.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/dialogueresponses.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/iff.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/input.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/nancy.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/renderobject.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/time.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/util.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/nancy/video.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/background.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/blbarchive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/diskplayerscene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/entity.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/gamemodule.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/gamevars.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/klaymen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/menumodule.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/messages.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/microtiles.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/module.h C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/mouse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/navigationscene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/neverhood.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/palette.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/resourceman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/scene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/smackerplayer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/smackerscene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/sprite.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/neverhood/staticdata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/anihandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/behavior.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/constants.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/floaters.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/gameloader.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/gfx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/input.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/interaction.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/messages.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/modal.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/motion.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/ngi.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/ngiarchive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/objectnames.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/objects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/scene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/scenes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/statics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ngi/utils.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/disk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/exec.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/gui.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/input.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/objects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/parallaction.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/parser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/parallaction/walk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/chase.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/compass.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/constants.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/cursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/elements.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/energymonitor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/fader.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/gamestate.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/hotspot.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/input.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/interaction.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/interface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/movie.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/notification.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/pegasus.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/scoring.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/surface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/timers.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/transition.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/types.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pegasus/util.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/base.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/big_dialogue.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/file_mgr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/flc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/petka.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/q_manager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/q_system.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/video.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/petka/walk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/archive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/audio_info_mgr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/cel_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/constants.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/cursor_mgr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/director.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/file.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/pda_mgr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/pink.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/resource_mgr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/pink/utils.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/playground3d/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/playground3d/gfx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/playground3d/gfx_opengl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/playground3d/gfx_opengl_shaders.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/playground3d/gfx_tinygl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/playground3d/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/playground3d/playground3d.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/plumbers/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/plumbers/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/plumbers/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/plumbers/plumbers.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/archive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/common.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/cursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/curve_values.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/decompress.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/flags.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/hero.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/hero_set.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/mhwanh.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/mob.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/musNum.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/object.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/option_text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/prince.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/pscr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/prince/variatxt.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/private/decompiler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/private/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/private/grammar.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/private/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/private/private.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/private/symbol.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/private/tokens.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/bankman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/command.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/credits.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/cutaway.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/defs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/display.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/grid.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/input.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/journal.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/logic.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/midiadlib.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/queen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/state.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/structs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/talk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/queen/walk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/actor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/displayinfo.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/gfx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/interface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/isomap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/itedata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/objectmap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/palanim.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/puzzle.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/render.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/saga.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/scene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/shared_detection_defines.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/sndres.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga/sprite.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/actor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/annoy.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/assign.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/audio.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/automap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/band.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/beegee.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/blitters.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/button.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/calender.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/cmisc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/code.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/combat.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/contain.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/dice.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/display.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/dispnode.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/document.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/effects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/enchant.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/floating.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/fontlib.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/fta.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/gamerate.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/gbevel.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/gblitter.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/gdraw.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/gfx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/gpointer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/grabinfo.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/grequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/gtextbox.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/hresmgr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/idtypes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/imagcach.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/input.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/intrface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/loadmsg.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/localize.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/magic.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/mainmap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/mapfeatr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/messager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/mission.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/modal.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/motion.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/mouseimg.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/msgbox.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/objects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/objproto.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/oncall.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/palette.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/panel.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/patrol.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/player.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/priqueue.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/program.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/property.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/rect.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/saga2.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/sensor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/setup.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/shorten.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/speech.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/speldefs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/spellbuk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/spellio.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/spells.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/spelshow.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/spelvals.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/sprite.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/stimtype.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/tagnoise.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/target.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/task.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/tcoords.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/terrain.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/tile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/tileline.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/tileload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/tilemode.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/tilevect.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/timers.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/tower.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/towerfta.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/towerwin.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/transit.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/tromode.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/uidialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/uimetrcs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/uitext.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/vbacksav.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/vdraw.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/version.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/videobox.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/vpage.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/vpal.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/saga2/weapons.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sci/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sci/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sci/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sci/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sci/detection_options.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sci/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sci/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sci/event.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sci/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sci/sci.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sci/util.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/actor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/actor_he.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/akos.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/base-costume.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/bomp.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/boxes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/cdda.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/charset.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/costume.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/detection_internal.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/detection_steam.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/file.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/file_nes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/gfx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/help.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/ks_check.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/metaengine.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/nut_renderer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/object.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/obsolete.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/scumm-md5.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/scumm.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/scumm_v0.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/scumm_v2.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/scumm_v3.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/scumm_v4.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/scumm_v5.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/scumm_v6.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/scumm_v7.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/scumm_v8.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/usage_bits.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/util.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/scumm/verbs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/fixed_text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/fonts.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/image_file.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/journal.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/map.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/objects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/people.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/resources.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/scene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/sherlock.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/surface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/talk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sherlock/user_interface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/autoroute.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/compact.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/control.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/disk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/grid.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/intro.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/logic.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/mouse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/rnc_deco.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/sky.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/skydefs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/struc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sky/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/builtin.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/csludge.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/cursors.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/errors.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/event.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/fileset.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/floor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/fonttext.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/freeze.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/function.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/functionlist.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/hsi.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/imgloader.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/language.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/main_loop.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/moreio.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/movie.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/newfatal.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/objtypes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/people.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/region.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/savedata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/sludge.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/sludger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/speech.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/sprbanks.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/sprites.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/statusba.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/timing.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/variable.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/version.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sludge/zbuffer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/stark/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/stark/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/stark/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/stark/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/stark/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/stark/resourcereference.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/stark/savemetadata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/stark/scene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/stark/stark.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/action.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/awaymission.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/bitmap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/bridge.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/common.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/fixedint.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/items.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/iwfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/lzss.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/object.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/room.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/soundeffects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/space.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/sprite.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/startrek.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/startrek/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/supernova/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/supernova/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/supernova/game-manager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/supernova/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/supernova/imageid.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/supernova/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/supernova/msn_def.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/supernova/resman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/supernova/room.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/supernova/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/supernova/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/supernova/supernova.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/collision.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/control.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/eventman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/logic.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/memman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/mouse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/object.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/objectman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/resman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/router.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/sword1.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/sworddefs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/swordres.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword1/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/controls.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/defs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/detection_internal.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/header.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/interpreter.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/logic.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/maketext.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/memory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/mouse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/object.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/resman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/router.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword2/sword2.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword25/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword25/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword25/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword25/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword25/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/sword25/sword25.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/actor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/dialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/objects.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/pack.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/resources.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/scene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/segment.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/surface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/surface_list.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/teenagent/teenagent.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/cloud.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/config-params.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/config.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/fs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/midi.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/misc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/networking.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/savegame.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/speech.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/template.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/testbed.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/testsuite.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/testbed/webserver.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/actors.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/adpcm.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/anim.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/background.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/bmv.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/cliprect.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/config.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/cursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/drives.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/dw.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/faders.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/film.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/handle.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/heapmem.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/mareels.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/move.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/movers.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/multiobj.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/object.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/palette.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/pcode.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/pdisplay.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/pid.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/play.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/polygons.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/savescn.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/scene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/sched.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/scn.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/scroll.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/strres.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/sysvar.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/timers.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/tinlib.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/tinsel.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tinsel/token.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/continue_save_dialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/game_location.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/game_manager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/game_state.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/game_view.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/input_handler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/input_translator.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/main_game_window.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/room_flags.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/titanic.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/titanic/translation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/menu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/microtiles.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/movie.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/music.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/palette.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/render.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/segmap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toltecs/toltecs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/custom.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/game.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/gfxcore.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/gfxengine.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/globals.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/input.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/inventory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/loc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/resid.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/tony.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/tonychar.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/utils.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tony/window.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/anim.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/audio.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/character.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/conversation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/drew.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/flux.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/hotspot.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/movie.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/path.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/picture.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/resource.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/script_func.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/state.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/subtitles.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/tools.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/toon/toon.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/touche/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/touche/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/touche/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/touche/midi.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/touche/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/touche/touche.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/actor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/animmanager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/animtype.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/defines.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/dialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/fastfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/logic.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/pathfinding3d.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/renderer3d.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/scheduler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/struct.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/trecision.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/trecision/video.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/converse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/core.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/globals.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/resources.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/scenes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/staticres.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/tsage.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tsage/user_interface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tucker/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tucker/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tucker/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tucker/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tucker/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/tucker/tucker.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/twine/achievements_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/twine/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/twine/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/twine/holomap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/twine/input.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/twine/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/twine/movies.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/twine/shared.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/twine/slideshow.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/twine/text.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/twine/twine.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ultima/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ultima/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ultima/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ultima/metaengine.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/ultima/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/data.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/files.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/staticres.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/voyeur/voyeur.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/design.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/detection.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/dialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/entities.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/gui.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/randomhat.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/script.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/wage.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wage/world.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/achievements_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/coll_templ.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/dcgf.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/dctypes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/keymapper_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/persistent.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/platform_osystem.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/wintermute.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/wintermute/wintypes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/character.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/combat.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/cutscenes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/files.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/interface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/interface_minimap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/interface_scene.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/item.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/locations.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/map.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/party.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/patcher.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/resources.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/saves.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/scripts.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/sound_driver.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/sound_driver_adlib.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/spells.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/sprites.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/subtitles.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/window.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/xeen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/xeen/xsurface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/zvision/detection.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/zvision/detection.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/zvision/detection_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/zvision/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/zvision/zvision.h

)

add_executable(scummvm
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/LICENSES/COPYING.BSD C:/Users/sariel/Projects/Fun/ScummVM/scummvm/LICENSES/COPYING.FREEFONT C:/Users/sariel/Projects/Fun/ScummVM/scummvm/LICENSES/COPYING.GLAD C:/Users/sariel/Projects/Fun/ScummVM/scummvm/LICENSES/COPYING.ISC C:/Users/sariel/Projects/Fun/ScummVM/scummvm/LICENSES/COPYING.LGPL C:/Users/sariel/Projects/Fun/ScummVM/scummvm/LICENSES/COPYING.LUA C:/Users/sariel/Projects/Fun/ScummVM/scummvm/LICENSES/COPYING.MIT C:/Users/sariel/Projects/Fun/ScummVM/scummvm/LICENSES/COPYING.OFL C:/Users/sariel/Projects/Fun/ScummVM/scummvm/LICENSES/COPYING.TINYGL
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/3do.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/3do.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/aac.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/aac.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/ac3.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/adpcm.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/adpcm.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/adpcm_intern.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/aiff.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/aiff.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/asf.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/asf.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/codec.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/flac.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/flac.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/g711.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/g711.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/iff_sound.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/iff_sound.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/mac_snd.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/mac_snd.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/mp3.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/mp3.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/qdm2.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/qdm2.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/qdm2data.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/quicktime.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/quicktime.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/quicktime_intern.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/raw.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/raw.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/util.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/voc.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/voc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/vorbis.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/vorbis.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/wave.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/wave.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/wave_types.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/wma.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/wma.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/wmadata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/xa.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/xa.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/xan_dpcm.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/decoders/xan_dpcm.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/infogrames.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/infogrames.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/maxtrax.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/maxtrax.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/mod_xm_s3m.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/mod_xm_s3m.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/module.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/module.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/module_mod_xm_s3m.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/module_mod_xm_s3m.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/paula.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/paula.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/protracker.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/protracker.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/rjp1.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/rjp1.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/soundfx.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/soundfx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/tfmx.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mods/tfmx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/pc98_audio.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/pc98_audio.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/pcm_common.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/pcm_common.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/sega_audio.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/sega_audio.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/towns_audio.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/towns_audio.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/towns_euphony.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/towns_euphony.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/towns_pc98_driver.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/towns_pc98_driver.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/towns_pc98_fmsynth.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/towns_pc98_fmsynth.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fmtowns_pc98/towns_pc98_plugins.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/c_interface/c_interface.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/c_interface/c_interface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/c_interface/c_types.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/c_interface/cpp_interface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/sha1/sha1.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/sha1/sha1.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/srchelper/srctools/src/FIRResampler.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/srchelper/srctools/src/IIR2xResampler.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/srchelper/srctools/src/LinearResampler.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/srchelper/srctools/src/ResamplerModel.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/srchelper/srctools/src/SincResampler.cpp

	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/srchelper/InternalResampler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/srchelper/InternalResampler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/srchelper/SamplerateAdapter.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/srchelper/SoxrAdapter.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Analog.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Analog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/BReverbModel.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/BReverbModel.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Enumerations.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/File.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/File.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/FileStream.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/FileStream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/LA32FloatWaveGenerator.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/LA32FloatWaveGenerator.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/LA32Ramp.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/LA32Ramp.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/LA32WaveGenerator.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/LA32WaveGenerator.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/MemoryRegion.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/MidiEventQueue.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/MidiStreamParser.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/MidiStreamParser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Part.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Part.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Partial.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Partial.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/PartialManager.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/PartialManager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Poly.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Poly.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/ROMInfo.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/ROMInfo.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/SampleRateConverter.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/SampleRateConverter.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Structures.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Synth.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Synth.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/TVA.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/TVA.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/TVF.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/TVF.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/TVP.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/TVP.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Tables.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/Types.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/VersionTagging.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/config.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/globals.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/internals.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/mmath.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32/mt32emu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/opl/dbopl.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/opl/dbopl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/opl/dosbox.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/opl/dosbox.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/opl/mame.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/opl/mame.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/opl/nuked.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/opl/nuked.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/appleiigs.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/cms.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/cms.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/eas.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/emumidi.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/fluidsynth.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/mt32.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/pcspk.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/pcspk.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/sid.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/sid.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/softsynth/wave6581.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/vab/psxspu.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/vab/psxspu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/vab/vab.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/vab/vab.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/common.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/rawfile.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/rawfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/rifffile.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/rifffile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/sf2file.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/sf2file.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/synthfile.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/synthfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/vgmcoll.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/vgmcoll.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/vgminstrset.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/vgminstrset.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/vgmitem.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/vgmitem.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/vgmsamp.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/soundfont/vgmsamp.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/adlib.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/adlib_ms.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/adlib_ms.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/audiostream.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/audiostream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/fmopl.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/fmopl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mididrv.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mididrv.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mididrv_ms.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mididrv_ms.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/midiparser.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/midiparser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/midiparser_qt.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/midiparser_qt.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/midiparser_smf.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/midiparser_xmidi.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/midiplayer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/midiplayer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/miles.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/miles_adlib.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/miles_midi.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mixer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mixer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mixer_intern.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mpu401.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mpu401.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mt32gm.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/mt32gm.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/musicplugin.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/musicplugin.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/null.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/null.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/rate.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/rate.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/rwopl3.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/timestamp.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/audio/timestamp.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/audiocd/default/default-audiocd.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/audiocd/default/default-audiocd.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/audiocd/audiocd-stream.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/audiocd/audiocd-stream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/audiocd/audiocd.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/box/boxlistdirectorybyidrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/box/boxlistdirectorybyidrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/box/boxstorage.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/box/boxstorage.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/box/boxtokenrefresher.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/box/boxtokenrefresher.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/box/boxuploadrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/box/boxuploadrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/dropbox/dropboxcreatedirectoryrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/dropbox/dropboxcreatedirectoryrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/dropbox/dropboxinforequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/dropbox/dropboxinforequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/dropbox/dropboxlistdirectoryrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/dropbox/dropboxlistdirectoryrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/dropbox/dropboxstorage.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/dropbox/dropboxstorage.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/dropbox/dropboxtokenrefresher.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/dropbox/dropboxtokenrefresher.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/dropbox/dropboxuploadrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/dropbox/dropboxuploadrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/googledrive/googledrivelistdirectorybyidrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/googledrive/googledrivelistdirectorybyidrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/googledrive/googledrivestorage.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/googledrive/googledrivestorage.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/googledrive/googledrivetokenrefresher.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/googledrive/googledrivetokenrefresher.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/googledrive/googledriveuploadrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/googledrive/googledriveuploadrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/id/idcreatedirectoryrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/id/idcreatedirectoryrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/id/iddownloadrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/id/iddownloadrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/id/idlistdirectoryrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/id/idlistdirectoryrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/id/idresolveidrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/id/idresolveidrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/id/idstorage.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/id/idstorage.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/id/idstreamfilerequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/id/idstreamfilerequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/onedrive/onedrivecreatedirectoryrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/onedrive/onedrivecreatedirectoryrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/onedrive/onedrivelistdirectoryrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/onedrive/onedrivelistdirectoryrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/onedrive/onedrivestorage.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/onedrive/onedrivestorage.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/onedrive/onedrivetokenrefresher.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/onedrive/onedrivetokenrefresher.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/onedrive/onedriveuploadrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/onedrive/onedriveuploadrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/basestorage.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/basestorage.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/cloudicon.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/cloudicon.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/cloudicon_data.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/cloudicon_disabled_data.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/cloudmanager.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/cloudmanager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/downloadrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/downloadrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/folderdownloadrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/folderdownloadrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/iso8601.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/iso8601.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/savessyncrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/savessyncrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/storage.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/storage.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/storagefile.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/storagefile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/cloud/storageinfo.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/dialogs/gtk/gtk-dialogs.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/dialogs/gtk/gtk-dialogs.h

	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/events/default/default-events.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/events/default/default-events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/events/sdl/legacy-sdl-events.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/events/sdl/legacy-sdl-events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/events/sdl/sdl-events.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/events/sdl/sdl-events.h

	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/chroot/chroot-fs-factory.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/chroot/chroot-fs-factory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/chroot/chroot-fs.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/chroot/chroot-fs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/posix/posix-fs-factory.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/posix/posix-fs-factory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/posix/posix-fs.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/posix/posix-fs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/posix/posix-iostream.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/posix/posix-iostream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/posix-drives/posix-drives-fs-factory.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/posix-drives/posix-drives-fs-factory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/posix-drives/posix-drives-fs.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/posix-drives/posix-drives-fs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/abstract-fs.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/abstract-fs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/fs-factory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/stdiostream.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/fs/stdiostream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/pipelines/clut8.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/pipelines/clut8.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/pipelines/fixed.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/pipelines/fixed.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/pipelines/pipeline.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/pipelines/pipeline.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/pipelines/shader.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/pipelines/shader.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/context.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/debug.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/framebuffer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/framebuffer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/opengl-graphics.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/opengl-graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/opengl-sys.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/shader.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/shader.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/texture.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/opengl/texture.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/openglsdl/openglsdl-graphics.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/openglsdl/openglsdl-graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/sdl/sdl-graphics.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/sdl/sdl-graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/surfacesdl/surfacesdl-graphics.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/surfacesdl/surfacesdl-graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/default-palette.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/graphics.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics/windowed.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics3d/opengl/framebuffer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics3d/opengl/framebuffer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics3d/opengl/surfacerenderer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics3d/opengl/surfacerenderer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics3d/opengl/texture.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics3d/opengl/texture.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics3d/opengl/tiledsurface.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics3d/opengl/tiledsurface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics3d/openglsdl/openglsdl-graphics3d.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/graphics3d/openglsdl/openglsdl-graphics3d.h

	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/action.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/action.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/hardware-input.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/hardware-input.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/input-watcher.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/input-watcher.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/keymap.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/keymap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/keymapper-defaults.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/keymapper.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/keymapper.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/remap-widget.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/remap-widget.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/standard-actions.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/standard-actions.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/virtual-mouse.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/keymapper/virtual-mouse.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/log/log.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/log/log.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/midi/alsa.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/midi/dmedia.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/midi/seq.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/midi/sndio.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/midi/stmidi.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/midi/timidity.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/mixer/sdl/sdl-mixer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/mixer/sdl/sdl-mixer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/mixer/mixer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/mutex/sdl/sdl-mutex.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/mutex/sdl/sdl-mutex.h

	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/connectionmanager.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/connectionmanager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/curljsonrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/curljsonrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/curlrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/curlrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/networkreadstream.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/networkreadstream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/postrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/postrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/request.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/request.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/session.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/session.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/sessionrequest.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/curl/sessionrequest.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/basehandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/createdirectoryhandler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/createdirectoryhandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/downloadfilehandler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/downloadfilehandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/filesajaxpagehandler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/filesajaxpagehandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/filesbasehandler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/filesbasehandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/filespagehandler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/filespagehandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/indexpagehandler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/indexpagehandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/listajaxhandler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/listajaxhandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/resourcehandler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/resourcehandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/uploadfilehandler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlers/uploadfilehandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/client.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/client.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/getclienthandler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/getclienthandler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlerutils.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/handlerutils.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/localwebserver.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/localwebserver.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/reader.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/reader.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/uploadfileclienthandler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/networking/sdl_net/uploadfileclienthandler.h

	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/platform/sdl/posix/posix-main.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/platform/sdl/posix/posix.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/platform/sdl/posix/posix.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/platform/sdl/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/platform/sdl/sdl-sys.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/platform/sdl/sdl-window.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/platform/sdl/sdl-window.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/platform/sdl/sdl.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/platform/sdl/sdl.h

	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/plugins/posix/posix-provider.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/plugins/posix/posix-provider.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/plugins/sdl/sdl-provider.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/plugins/sdl/sdl-provider.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/plugins/dynamic-plugin.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/saves/default/default-saves.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/saves/default/default-saves.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/saves/posix/posix-saves.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/saves/posix/posix-saves.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/saves/savefile.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/taskbar/unity/unity-taskbar.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/taskbar/unity/unity-taskbar.h

	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/timer/default/default-timer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/timer/default/default-timer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/timer/sdl/sdl-timer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/timer/sdl/sdl-timer.h

	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/base-backend.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/base-backend.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/modular-backend.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/modular-backend.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/backends/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/base/commandLine.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/base/commandLine.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/base/internal_plugins.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/base/internal_version.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/base/main.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/base/main.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/base/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/base/plugins.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/base/plugins.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/base/test_new_standards.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/base/version.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/base/version.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/encodings/singlebyte.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/COPYRIGHT
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/double_serialization.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/double_serialization.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lapi.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lapi.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lauxlib.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lauxlib.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lbaselib.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lcode.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lcode.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/ldblib.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/ldebug.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/ldebug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/ldo.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/ldo.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lfunc.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lfunc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lgc.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lgc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/linit.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/liolib.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/llex.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/llex.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/llimits.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lmathlib.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lmem.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lmem.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/loadlib.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lobject.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lobject.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lopcodes.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lopcodes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/loslib.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lparser.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lparser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lstate.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lstate.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lstring.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lstring.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lstrlib.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/ltable.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/ltable.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/ltablib.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/ltm.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/ltm.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lua.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lua_persist.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lua_persistence.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lua_persistence_util.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lua_persistence_util.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lua_unpersist.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/luaconf.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lualib.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lvm.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lvm.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lzio.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/lzio.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/scummvm_file.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/lua/scummvm_file.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/achievements.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/achievements.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/algorithm.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/archive.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/archive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/array.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/base-str.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/base-str.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/bitarray.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/bitstream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/bufferedstream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/callback.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/config-manager.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/config-manager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/coroutines.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/coroutines.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/cosinetables.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/cosinetables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/crc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/dcl.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/dcl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/dct.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/dct.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/debug-channels.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/debug.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/debug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/enc-internal.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/endian.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/error.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/error.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/events.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/events.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/fft.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/fft.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/file.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/file.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/forbidden.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/foreach.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/frac.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/fs.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/fs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/func.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/gui_options.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/gui_options.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/hash-ptr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/hash-str.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/hashmap.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/hashmap.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/huffman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/iff_container.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/iff_container.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/ini-file.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/ini-file.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/initializer_list.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/installshield_cab.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/installshield_cab.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/installshieldv3_archive.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/installshieldv3_archive.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/inttypes.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/json.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/json.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/keyboard.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/language.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/language.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/list.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/list_intern.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/localization.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/localization.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/macresman.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/macresman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/math.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/md5.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/md5.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/mdct.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/mdct.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/memory.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/memorypool.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/memorypool.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/memstream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/mutex.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/mutex.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/noncopyable.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/osd_message_queue.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/osd_message_queue.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/pack-end.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/pack-start.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/path.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/path.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/platform.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/platform.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/ptr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/punycode.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/punycode.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/queue.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/quicktime.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/quicktime.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/random.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/random.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/rational.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/rational.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/rdft.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/rdft.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/recorderfile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/rect.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/rendermode.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/rendermode.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/safe-bool.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/savefile.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/scummsys.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/serializer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/sinetables.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/sinetables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/sinewindows.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/sinewindows.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/singleton.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/span.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/stack.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/str-array.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/str-enc.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/str-enc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/str.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/str.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/stream.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/stream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/streamdebug.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/streamdebug.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/stuffit.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/stuffit.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/substream.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/system.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/system.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/taskbar.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/text-to-speech.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/text-to-speech.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/textconsole.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/textconsole.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/timer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/tokenizer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/tokenizer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/translation.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/translation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/type-traits.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/types.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/unarj.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/unarj.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/unicode-bidi.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/unicode-bidi.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/unzip.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/unzip.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/updates.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/ustr.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/ustr.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/util.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/util.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/winexe.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/winexe.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/winexe_ne.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/winexe_ne.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/winexe_pe.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/winexe_pe.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/xmlparser.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/xmlparser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/zlib.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/common/zlib.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/advancedDetector.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/advancedDetector.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/detection_table.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dialogs.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/dialogs.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/engine.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/engine.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/game.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/game.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/logo_data.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/metaengine.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/metaengine.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/obsolete.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/obsolete.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/plugins_table.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/savestate.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/savestate.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/engines/util.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/amigafont.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/amigafont.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/bdf.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/bdf.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/consolefont.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/macfont.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/macfont.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/newfont.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/newfont_big.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/ttf.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/ttf.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/winfont.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fonts/winfont.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/datafiles.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macbutton.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macbutton.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macfontmanager.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macfontmanager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macmenu.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macmenu.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/mactext.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/mactext.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/mactextwindow.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/mactextwindow.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macwidget.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macwidget.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macwindow.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macwindow.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macwindowborder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macwindowborder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macwindowmanager.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macgui/macwindowmanager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/opengl/context.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/opengl/context.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/opengl/glad.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/opengl/shader.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/opengl/shader.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/opengl/system_headers.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/aspect.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/aspect.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/dotmatrix.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/dotmatrix.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/downscaler.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/downscaler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/edge.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/edge.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/hq.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/hq.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/hq2x_i386.asm
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/hq3x_i386.asm
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/intern.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/normal.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/normal.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/pm.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/pm.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/sai.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/sai.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/scale2x.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/scale2x.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/scale3x.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/scale3x.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/scalebit.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/scalebit.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/thumbnail_intern.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/tv.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler/tv.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/api.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/arrays.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/clear.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/clip.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/get.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/gl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/image_util.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/init.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/light.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/list.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/matrix.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/memory.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/misc.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/opinfo.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/pixelbuffer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/pixelbuffer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/select.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/specbuf.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/texelbuffer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/texelbuffer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/texture.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/tinygl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/vertex.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/zblit.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/zblit.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/zblit_public.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/zbuffer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/zbuffer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/zdirtyrect.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/zdirtyrect.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/zgl.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/zline.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/zmath.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/zmath.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/tinygl/ztriangle.cpp
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/VectorRenderer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/VectorRenderer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/VectorRendererSpec.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/VectorRendererSpec.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/colormasks.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/conversion.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/conversion.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/cursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/cursorman.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/cursorman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/font.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/font.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fontman.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/fontman.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/framelimiter.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/framelimiter.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/korfont.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/korfont.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/larryScale.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/larryScale.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/maccursor.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/maccursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/macega.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/managed_surface.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/managed_surface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/mode.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/nine_patch.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/nine_patch.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/palette.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/pixelformat.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/pixelformat.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/primitives.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/primitives.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/renderer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/renderer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scaler.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scalerplugin.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/scalerplugin.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/screen.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/screen.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/sjis.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/sjis.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/surface.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/surface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/svg.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/svg.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/thumbnail.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/thumbnail.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/transform_struct.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/transform_struct.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/transform_tools.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/transform_tools.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/transparent_surface.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/transparent_surface.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/wincursor.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/wincursor.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/yuv_to_rgb.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/graphics/yuv_to_rgb.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/AccelerateInterpolator.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/AlphaAnimation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/Animation.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/Animation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/DeccelerateInterpolator.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/Drawable.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/Interpolator.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/ParallelAnimation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/RepeatAnimationWrapper.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/RepeatAnimationWrapper.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/ScaleAnimation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/SequenceAnimationComposite.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/SequenceAnimationComposite.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/animation/WaitForConditionAnimation.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/editable.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/editable.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/edittext.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/edittext.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/grid.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/grid.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/groupedlist.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/groupedlist.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/list.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/list.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/popup.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/popup.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/scrollbar.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/scrollbar.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/scrollcontainer.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/scrollcontainer.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/tab.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widgets/tab.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/EventRecorder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/EventRecorder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/MetadataParser.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/MetadataParser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/ThemeEngine.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/ThemeEngine.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/ThemeEval.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/ThemeEval.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/ThemeLayout.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/ThemeLayout.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/ThemeParser.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/ThemeParser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/Tooltip.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/Tooltip.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/about.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/about.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/browser.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/browser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/chooser.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/chooser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/console.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/console.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/credits.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/debugger.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/debugger.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/dialog.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/dialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/downloaddialog.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/downloaddialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/downloadiconsdialog.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/downloadiconsdialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/editgamedialog.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/editgamedialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/editrecorddialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/error.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/error.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/filebrowser-dialog.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/filebrowser-dialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/fluidsynth-dialog.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/fluidsynth-dialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/gui-manager.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/gui-manager.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/launcher.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/launcher.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/massadd.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/massadd.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/message.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/message.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/object.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/object.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/onscreendialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/options.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/options.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/predictivedialog.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/predictivedialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/recorderdialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/remotebrowser.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/remotebrowser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/saveload-dialog.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/saveload-dialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/saveload.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/saveload.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/themebrowser.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/themebrowser.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/unknown-game-dialog.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/unknown-game-dialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/updates-dialog.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widget.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/gui/widget.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo/get_bits.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo/indeo.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo/indeo.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo/indeo_dsp.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo/indeo_dsp.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo/mem.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo/mem.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo/vlc.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo/vlc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/bmp_raw.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/bmp_raw.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/cdtoons.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/cdtoons.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/cinepak.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/cinepak.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/cinepak_tables.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/codec.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/codec.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/hlz.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/hlz.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo3.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo3.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo4.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo4.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo5.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/indeo5.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/jyv1.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/jyv1.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/mjpeg.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/mjpeg.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/mpeg.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/mpeg.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/msrle.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/msrle.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/msrle4.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/msrle4.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/msvideo1.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/msvideo1.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/qtrle.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/qtrle.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/rpza.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/rpza.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/smc.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/smc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/svq1.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/svq1.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/svq1_cb.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/svq1_vlc.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/truemotion1.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/truemotion1.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/truemotion1data.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/xan.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/codecs/xan.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/bmp.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/bmp.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/cel_3do.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/cel_3do.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/gif.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/gif.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/iff.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/iff.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/image_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/jpeg.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/jpeg.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/pcx.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/pcx.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/pict.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/pict.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/png.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/png.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/tga.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/image/tga.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/aabb.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/aabb.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/angle.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/angle.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/frustum.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/frustum.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/glmath.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/glmath.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/line2d.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/line2d.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/line3d.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/line3d.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/mathfwd.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/matrix.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/matrix3.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/matrix3.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/matrix4.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/matrix4.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/plane.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/plane.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/quat.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/quat.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/ray.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/ray.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/rect2d.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/rect2d.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/rotation3d.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/squarematrix.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/transform.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/utils.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/vector.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/vector2d.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/vector2d.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/vector3d.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/vector3d.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/vector4d.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/math/vector4d.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/3do_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/3do_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/avi_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/avi_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/bink_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/bink_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/binkdata.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/coktel_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/coktel_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/dxa_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/dxa_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/flic_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/flic_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/hnm_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/hnm_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/module.mk
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/mpegps_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/mpegps_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/mve_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/mve_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/paco_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/paco_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/psx_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/psx_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/qt_data.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/qt_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/qt_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/smk_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/smk_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/theora_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/theora_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/video_decoder.cpp C:/Users/sariel/Projects/Fun/ScummVM/scummvm/video/video_decoder.h
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/AUTHORS
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/COPYING
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/COPYRIGHT
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/NEWS.md
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/README.md
	C:/Users/sariel/Projects/Fun/ScummVM/scummvm/config.h
)

# Engines libraries handling
#add_engine(access
#	/amazon/amazon_game.cpp /amazon/amazon_game.h
#	/amazon/amazon_logic.cpp /amazon/amazon_logic.h
#	/amazon/amazon_player.cpp /amazon/amazon_player.h
#	/amazon/amazon_resources.cpp /amazon/amazon_resources.h
#	/amazon/amazon_room.cpp /amazon/amazon_room.h
#	/amazon/amazon_scripts.cpp /amazon/amazon_scripts.h
#	/martian/martian_game.cpp /martian/martian_game.h
#	/martian/martian_player.cpp /martian/martian_player.h
#	/martian/martian_resources.cpp /martian/martian_resources.h
#	/martian/martian_room.cpp /martian/martian_room.h
#	/martian/martian_scripts.cpp /martian/martian_scripts.h
#	/video/movie_decoder.cpp /video/movie_decoder.h
#	/access.cpp /access.h
#	/animation.cpp /animation.h
#	/asurface.cpp /asurface.h
#	/bubble_box.cpp /bubble_box.h
#	/char.cpp /char.h
#	/data.cpp /data.h
#	/debugger.cpp /debugger.h
#	/decompress.cpp /decompress.h
#	/detection.h
#	/detection_tables.h
#	/events.cpp /events.h
#	/files.cpp /files.h
#	/font.cpp /font.h
#	/inventory.cpp /inventory.h
#	/metaengine.cpp
#	/module.mk
#	/player.cpp /player.h
#	/resources.cpp /resources.h
#	/room.cpp /room.h
#	/screen.cpp /screen.h
#	/scripts.cpp /scripts.h
#	/sound.cpp /sound.h
#	/video.cpp /video.h
#)
#
#add_engine(adl
#	/adl.cpp /adl.h
#	/adl_v2.cpp /adl_v2.h
#	/adl_v3.cpp /adl_v3.h
#	/adl_v4.cpp /adl_v4.h
#	/adl_v5.cpp /adl_v5.h
#	/console.cpp /console.h
#	/detection.h
#	/disk.cpp /disk.h
#	/disk_image_helpers.h
#	/display.cpp /display.h
#	/display_a2.cpp /display_a2.h
#	/graphics.h
#	/hires1.cpp
#	/hires2.cpp
#	/hires4.cpp
#	/hires5.cpp
#	/hires6.cpp
#	/metaengine.cpp
#	/module.mk
#	/sound.cpp /sound.h
#)
#
#add_engine(agi
#	/agi.cpp /agi.h
#	/appleIIgs_timedelay_overwrite.h
#	/checks.cpp
#	/console.cpp /console.h
#	/cycle.cpp
#	/detection.h
#	/detection_tables.h
#	/font.cpp /font.h
#	/global.cpp
#	/graphics.cpp /graphics.h
#	/inv.cpp /inv.h
#	/keyboard.cpp /keyboard.h
#	/loader_v1.cpp
#	/loader_v2.cpp
#	/loader_v3.cpp
#	/logic.cpp /logic.h
#	/lzw.cpp /lzw.h
#	/menu.cpp /menu.h
#	/metaengine.cpp
#	/module.mk
#	/motion.cpp
#	/mouse_cursor.h
#	/objects.cpp
#	/op_cmd.cpp
#	/op_dbg.cpp
#	/op_test.cpp
#	/opcodes.cpp /opcodes.h
#	/palette.h
#	/picture.cpp /picture.h
#	/preagi.cpp /preagi.h
#	/preagi_mickey.cpp /preagi_mickey.h
#	/preagi_troll.cpp /preagi_troll.h
#	/preagi_winnie.cpp /preagi_winnie.h
#	/saveload.cpp
#	/sound.cpp /sound.h
#	/sound_2gs.cpp /sound_2gs.h
#	/sound_coco3.cpp /sound_coco3.h
#	/sound_midi.cpp /sound_midi.h
#	/sound_pcjr.cpp /sound_pcjr.h
#	/sound_sarien.cpp /sound_sarien.h
#	/sprite.cpp /sprite.h
#	/systemui.cpp /systemui.h
#	/text.cpp /text.h
#	/view.cpp /view.h
#	/wagparser.h
#	/words.cpp /words.h
#)
#
#add_engine(agos
#	/drivers/accolade/adlib.cpp /drivers/accolade/adlib.h
#	/drivers/accolade/driverfile.cpp
#	/drivers/accolade/mididriver.h
#	/drivers/accolade/mt32.cpp /drivers/accolade/mt32.h
#	/drivers/accolade/pc98.cpp
#	/drivers/simon1/adlib.cpp /drivers/simon1/adlib.h
#
#	/agos.cpp /agos.h
#	/animation.cpp /animation.h
#	/charset-fontdata.cpp
#	/charset.cpp
#	/contain.cpp
#	/cursor.cpp
#	/debug.cpp /debug.h
#	/debugger.cpp /debugger.h
#	/detection.h
#	/detection_tables.h
#	/draw.cpp
#	/event.cpp
#	/feeble.cpp
#	/gfx.cpp
#	/icons.cpp
#	/input.cpp
#	/input_pn.cpp
#	/intern.h
#	/intern_detection.h
#	/items.cpp
#	/menus.cpp
#	/metaengine.cpp
#	/midi.cpp /midi.h
#	/midiparser_s1d.cpp
#	/module.mk
#	/obsolete.h
#	/oracle.cpp
#	/pn.cpp
#	/res.cpp
#	/res_ami.cpp
#	/res_snd.cpp
#	/rooms.cpp
#	/saveload.cpp
#	/script.cpp
#	/script_dp.cpp
#	/script_e1.cpp
#	/script_e2.cpp
#	/script_ff.cpp
#	/script_pn.cpp
#	/script_pp.cpp
#	/script_s1.cpp
#	/script_s2.cpp
#	/script_ww.cpp
#	/sound.cpp /sound.h
#	/string.cpp
#	/string_pn.cpp
#	/subroutine.cpp
#	/verb.cpp
#	/verb_pn.cpp
#	/vga.cpp /vga.h
#	/vga_e2.cpp
#	/vga_ff.cpp
#	/vga_pn.cpp
#	/vga_s1.cpp
#	/vga_s2.cpp
#	/vga_ww.cpp
#	/window.cpp
#	/zones.cpp
#)
#
#add_engine(ags
#	/engine/ac/dynobj/all_dynamic_classes.h
#	/engine/ac/dynobj/all_script_classes.h
#	/engine/ac/dynobj/cc_ags_dynamic_object.cpp /engine/ac/dynobj/cc_ags_dynamic_object.h
#	/engine/ac/dynobj/cc_audio_channel.cpp /engine/ac/dynobj/cc_audio_channel.h
#	/engine/ac/dynobj/cc_audio_clip.cpp /engine/ac/dynobj/cc_audio_clip.h
#	/engine/ac/dynobj/cc_character.cpp /engine/ac/dynobj/cc_character.h
#	/engine/ac/dynobj/cc_dialog.cpp /engine/ac/dynobj/cc_dialog.h
#	/engine/ac/dynobj/cc_dynamic_array.cpp /engine/ac/dynobj/cc_dynamic_array.h
#	/engine/ac/dynobj/cc_dynamic_object.cpp /engine/ac/dynobj/cc_dynamic_object.h
#	/engine/ac/dynobj/cc_dynamic_object_addr_and_manager.h
#	/engine/ac/dynobj/cc_gui.cpp /engine/ac/dynobj/cc_gui.h
#	/engine/ac/dynobj/cc_gui_object.cpp /engine/ac/dynobj/cc_gui_object.h
#	/engine/ac/dynobj/cc_hotspot.cpp /engine/ac/dynobj/cc_hotspot.h
#	/engine/ac/dynobj/cc_inventory.cpp /engine/ac/dynobj/cc_inventory.h
#	/engine/ac/dynobj/cc_object.cpp /engine/ac/dynobj/cc_object.h
#	/engine/ac/dynobj/cc_region.cpp /engine/ac/dynobj/cc_region.h
#	/engine/ac/dynobj/cc_serializer.cpp /engine/ac/dynobj/cc_serializer.h
#	/engine/ac/dynobj/managed_object_pool.cpp /engine/ac/dynobj/managed_object_pool.h
#	/engine/ac/dynobj/script_audio_channel.h
#	/engine/ac/dynobj/script_camera.cpp /engine/ac/dynobj/script_camera.h
#	/engine/ac/dynobj/script_containers.h
#	/engine/ac/dynobj/script_date_time.cpp /engine/ac/dynobj/script_date_time.h
#	/engine/ac/dynobj/script_dialog.h
#	/engine/ac/dynobj/script_dialog_options_rendering.cpp /engine/ac/dynobj/script_dialog_options_rendering.h
#	/engine/ac/dynobj/script_dict.cpp /engine/ac/dynobj/script_dict.h
#	/engine/ac/dynobj/script_drawing_surface.cpp /engine/ac/dynobj/script_drawing_surface.h
#	/engine/ac/dynobj/script_dynamic_sprite.cpp /engine/ac/dynobj/script_dynamic_sprite.h
#	/engine/ac/dynobj/script_file.cpp /engine/ac/dynobj/script_file.h
#	/engine/ac/dynobj/script_gui.h
#	/engine/ac/dynobj/script_hotspot.h
#	/engine/ac/dynobj/script_inv_item.h
#	/engine/ac/dynobj/script_mouse.h
#	/engine/ac/dynobj/script_object.h
#	/engine/ac/dynobj/script_overlay.cpp /engine/ac/dynobj/script_overlay.h
#	/engine/ac/dynobj/script_region.h
#	/engine/ac/dynobj/script_set.cpp /engine/ac/dynobj/script_set.h
#	/engine/ac/dynobj/script_string.cpp /engine/ac/dynobj/script_string.h
#	/engine/ac/dynobj/script_system.h
#	/engine/ac/dynobj/script_user_object.cpp /engine/ac/dynobj/script_user_object.h
#	/engine/ac/dynobj/script_view_frame.cpp /engine/ac/dynobj/script_view_frame.h
#	/engine/ac/dynobj/script_viewport.cpp /engine/ac/dynobj/script_viewport.h
#	/engine/ac/statobj/ags_static_object.cpp /engine/ac/statobj/ags_static_object.h
#	/engine/ac/statobj/static_array.cpp /engine/ac/statobj/static_array.h
#	/engine/ac/statobj/static_object.h
#	/engine/ac/asset_helper.h
#	/engine/ac/audio_channel.cpp /engine/ac/audio_channel.h
#	/engine/ac/audio_clip.cpp /engine/ac/audio_clip.h
#	/engine/ac/button.cpp /engine/ac/button.h
#	/engine/ac/cd_audio.cpp /engine/ac/cd_audio.h
#	/engine/ac/character.cpp /engine/ac/character.h
#	/engine/ac/character_cache.h
#	/engine/ac/character_extras.cpp /engine/ac/character_extras.h
#	/engine/ac/character_info_engine.cpp
#	/engine/ac/date_time.cpp /engine/ac/date_time.h
#	/engine/ac/dialog.cpp /engine/ac/dialog.h
#	/engine/ac/dialog_options_rendering.cpp /engine/ac/dialog_options_rendering.h
#	/engine/ac/display.cpp /engine/ac/display.h
#	/engine/ac/draw.cpp /engine/ac/draw.h
#	/engine/ac/draw_software.cpp /engine/ac/draw_software.h
#	/engine/ac/drawing_surface.cpp /engine/ac/drawing_surface.h
#	/engine/ac/dynamic_sprite.cpp /engine/ac/dynamic_sprite.h
#	/engine/ac/event.cpp /engine/ac/event.h
#	/engine/ac/file.cpp /engine/ac/file.h
#	/engine/ac/game.cpp /engine/ac/game.h
#	/engine/ac/game_setup.cpp /engine/ac/game_setup.h
#	/engine/ac/game_state.cpp /engine/ac/game_state.h
#	/engine/ac/global_api.cpp
#	/engine/ac/global_audio.cpp /engine/ac/global_audio.h
#	/engine/ac/global_button.cpp /engine/ac/global_button.h
#	/engine/ac/global_character.cpp /engine/ac/global_character.h
#	/engine/ac/global_date_time.cpp /engine/ac/global_date_time.h
#	/engine/ac/global_debug.cpp /engine/ac/global_debug.h
#	/engine/ac/global_dialog.cpp /engine/ac/global_dialog.h
#	/engine/ac/global_display.cpp /engine/ac/global_display.h
#	/engine/ac/global_drawing_surface.cpp /engine/ac/global_drawing_surface.h
#	/engine/ac/global_dynamic_sprite.cpp /engine/ac/global_dynamic_sprite.h
#	/engine/ac/global_file.cpp /engine/ac/global_file.h
#	/engine/ac/global_game.cpp /engine/ac/global_game.h
#	/engine/ac/global_gui.cpp /engine/ac/global_gui.h
#	/engine/ac/global_hotspot.cpp /engine/ac/global_hotspot.h
#	/engine/ac/global_inv_window.cpp /engine/ac/global_inv_window.h
#	/engine/ac/global_inventory_item.cpp /engine/ac/global_inventory_item.h
#	/engine/ac/global_label.cpp /engine/ac/global_label.h
#	/engine/ac/global_listbox.cpp /engine/ac/global_listbox.h
#	/engine/ac/global_mouse.cpp /engine/ac/global_mouse.h
#	/engine/ac/global_object.cpp /engine/ac/global_object.h
#	/engine/ac/global_overlay.cpp /engine/ac/global_overlay.h
#	/engine/ac/global_palette.cpp /engine/ac/global_palette.h
#	/engine/ac/global_parser.cpp /engine/ac/global_parser.h
#	/engine/ac/global_record.cpp /engine/ac/global_record.h
#	/engine/ac/global_region.cpp /engine/ac/global_region.h
#	/engine/ac/global_room.cpp /engine/ac/global_room.h
#	/engine/ac/global_screen.cpp /engine/ac/global_screen.h
#	/engine/ac/global_slider.cpp /engine/ac/global_slider.h
#	/engine/ac/global_string.cpp /engine/ac/global_string.h
#	/engine/ac/global_textbox.cpp /engine/ac/global_textbox.h
#	/engine/ac/global_timer.cpp /engine/ac/global_timer.h
#	/engine/ac/global_translation.cpp /engine/ac/global_translation.h
#	/engine/ac/global_video.cpp /engine/ac/global_video.h
#	/engine/ac/global_view_frame.cpp /engine/ac/global_view_frame.h
#	/engine/ac/global_viewport.cpp /engine/ac/global_viewport.h
#	/engine/ac/global_walk_behind.cpp /engine/ac/global_walk_behind.h
#	/engine/ac/global_walkable_area.cpp /engine/ac/global_walkable_area.h
#	/engine/ac/gui.cpp /engine/ac/gui.h
#	/engine/ac/gui_control.cpp /engine/ac/gui_control.h
#	/engine/ac/gui_inv.cpp
#	/engine/ac/hotspot.cpp /engine/ac/hotspot.h
#	/engine/ac/interface_button.cpp
#	/engine/ac/interface_element.cpp
#	/engine/ac/inv_window.cpp /engine/ac/inv_window.h
#	/engine/ac/inventory_item.cpp /engine/ac/inventory_item.h
#	/engine/ac/label.cpp /engine/ac/label.h
#	/engine/ac/lip_sync.h
#	/engine/ac/listbox.cpp /engine/ac/listbox.h
#	/engine/ac/math.cpp /engine/ac/math.h
#	/engine/ac/mouse.cpp /engine/ac/mouse.h
#	/engine/ac/move_list.cpp /engine/ac/move_list.h
#	/engine/ac/object.cpp /engine/ac/object.h
#	/engine/ac/object_cache.h
#	/engine/ac/overlay.cpp /engine/ac/overlay.h
#	/engine/ac/parser.cpp /engine/ac/parser.h
#	/engine/ac/path_helper.h
#	/engine/ac/properties.cpp /engine/ac/properties.h
#	/engine/ac/region.cpp /engine/ac/region.h
#	/engine/ac/rich_game_media.cpp /engine/ac/rich_game_media.h
#	/engine/ac/room.cpp /engine/ac/room.h
#	/engine/ac/room_object.cpp /engine/ac/room_object.h
#	/engine/ac/room_status.cpp /engine/ac/room_status.h
#	/engine/ac/route_finder.cpp /engine/ac/route_finder.h
#	/engine/ac/route_finder_impl.cpp /engine/ac/route_finder_impl.h
#	/engine/ac/route_finder_impl_legacy.cpp /engine/ac/route_finder_impl_legacy.h
#	/engine/ac/route_finder_jps.cpp /engine/ac/route_finder_jps.h
#	/engine/ac/runtime_defines.h
#	/engine/ac/screen.cpp /engine/ac/screen.h
#	/engine/ac/screen_overlay.cpp /engine/ac/screen_overlay.h
#	/engine/ac/script_containers.cpp
#	/engine/ac/slider.cpp /engine/ac/slider.h
#	/engine/ac/speech.cpp /engine/ac/speech.h
#	/engine/ac/sprite.cpp /engine/ac/sprite.h
#	/engine/ac/sprite_cache_engine.cpp
#	/engine/ac/sprite_list_entry.h
#	/engine/ac/string.cpp /engine/ac/string.h
#	/engine/ac/sys_events.cpp /engine/ac/sys_events.h
#	/engine/ac/system.cpp /engine/ac/system.h
#	/engine/ac/textbox.cpp /engine/ac/textbox.h
#	/engine/ac/timer.cpp /engine/ac/timer.h
#	/engine/ac/top_bar_settings.h
#	/engine/ac/translation.cpp /engine/ac/translation.h
#	/engine/ac/view_frame.cpp /engine/ac/view_frame.h
#	/engine/ac/viewport_script.cpp
#	/engine/ac/walk_behind.cpp /engine/ac/walk_behind.h
#	/engine/ac/walkable_area.cpp /engine/ac/walkable_area.h
#	/engine/debugging/ags_editor_debugger.h
#	/engine/debugging/console_output_target.cpp /engine/debugging/console_output_target.h
#	/engine/debugging/debug.cpp
#	/engine/debugging/debug_log.h
#	/engine/debugging/debugger.h
#	/engine/debugging/dummy_ags_debugger.h
#	/engine/debugging/file_based_ags_debugger.cpp /engine/debugging/file_based_ags_debugger.h
#	/engine/debugging/log_file.cpp /engine/debugging/log_file.h
#	/engine/debugging/message_buffer.cpp /engine/debugging/message_buffer.h
#	/engine/device/mouse_w32.cpp /engine/device/mouse_w32.h
#	/engine/font/fonts_engine.cpp
#	/engine/game/game_init.cpp /engine/game/game_init.h
#	/engine/game/savegame.cpp /engine/game/savegame.h
#	/engine/game/savegame_components.cpp /engine/game/savegame_components.h
#	/engine/game/savegame_internal.h
#	/engine/game/savegame_v321.cpp
#	/engine/game/viewport.cpp /engine/game/viewport.h
#	/engine/gfx/ali_3d_scummvm.cpp /engine/gfx/ali_3d_scummvm.h
#	/engine/gfx/blender.cpp /engine/gfx/blender.h
#	/engine/gfx/color_engine.cpp
#	/engine/gfx/ddb.h
#	/engine/gfx/gfx_defines.h
#	/engine/gfx/gfx_driver_base.cpp /engine/gfx/gfx_driver_base.h
#	/engine/gfx/gfx_driver_factory.cpp /engine/gfx/gfx_driver_factory.h
#	/engine/gfx/gfx_driver_factory_base.h
#	/engine/gfx/gfx_mode_list.h
#	/engine/gfx/gfx_util.cpp /engine/gfx/gfx_util.h
#	/engine/gfx/gfxfilter.h
#	/engine/gfx/gfxfilter_scaling.cpp /engine/gfx/gfxfilter_scaling.h
#	/engine/gfx/gfxfilter_scummvm_renderer.cpp /engine/gfx/gfxfilter_scummvm_renderer.h
#	/engine/gfx/graphics_driver.h
#	/engine/gui/animating_gui_button.cpp /engine/gui/animating_gui_button.h
#	/engine/gui/csci_dialog.cpp /engine/gui/csci_dialog.h
#	/engine/gui/gui_dialog.cpp /engine/gui/gui_dialog.h
#	/engine/gui/gui_dialog_defines.h
#	/engine/gui/gui_dialog_internal_defs.h
#	/engine/gui/gui_engine.cpp
#	/engine/gui/my_controls.h
#	/engine/gui/my_label.cpp /engine/gui/my_label.h
#	/engine/gui/my_listbox.cpp /engine/gui/my_listbox.h
#	/engine/gui/my_push_button.cpp /engine/gui/my_push_button.h
#	/engine/gui/my_textbox.cpp /engine/gui/my_textbox.h
#	/engine/gui/new_control.cpp /engine/gui/new_control.h
#	/engine/main/config.cpp /engine/main/config.h
#	/engine/main/engine.cpp /engine/main/engine.h
#	/engine/main/engine_setup.cpp /engine/main/engine_setup.h
#	/engine/main/game_file.cpp /engine/main/game_file.h
#	/engine/main/game_run.cpp /engine/main/game_run.h
#	/engine/main/game_start.cpp /engine/main/game_start.h
#	/engine/main/graphics_mode.cpp /engine/main/graphics_mode.h
#	/engine/main/main.cpp /engine/main/main.h
#	/engine/main/quit.cpp /engine/main/quit.h
#	/engine/main/update.cpp /engine/main/update.h
#	/engine/media/audio/ambient_sound.cpp /engine/media/audio/ambient_sound.h
#	/engine/media/audio/audio.cpp /engine/media/audio/audio.h
#	/engine/media/audio/audio_defines.h
#	/engine/media/audio/audio_system.h
#	/engine/media/audio/clip_my_midi.cpp /engine/media/audio/clip_my_midi.h
#	/engine/media/audio/queued_audio_item.cpp /engine/media/audio/queued_audio_item.h
#	/engine/media/audio/sound.cpp /engine/media/audio/sound.h
#	/engine/media/audio/sound_clip.cpp /engine/media/audio/sound_clip.h
#	/engine/media/video/video.cpp /engine/media/video/video.h
#
#	/engine/platform/base/ags_platform_driver.cpp /engine/platform/base/ags_platform_driver.h
#	/engine/platform/base/sys_main.cpp /engine/platform/base/sys_main.h
#	/engine/platform/scummvm/scummvm_platform_driver.cpp
#
#	/engine/script/cc_instance.cpp /engine/script/cc_instance.h
#	/engine/script/executing_script.cpp /engine/script/executing_script.h
#	/engine/script/exports.cpp /engine/script/exports.h
#	/engine/script/non_blocking_script_function.h
#	/engine/script/runtime_script_value.cpp /engine/script/runtime_script_value.h
#	/engine/script/script.cpp /engine/script/script.h
#	/engine/script/script_api.cpp /engine/script/script_api.h
#	/engine/script/script_engine.cpp
#	/engine/script/script_runtime.cpp /engine/script/script_runtime.h
#	/engine/script/system_imports.cpp /engine/script/system_imports.h
#
#	/lib/aastr-0.1.1/aarot.cpp
#	/lib/aastr-0.1.1/aastr.cpp /lib/aastr-0.1.1/aastr.h
#	/lib/aastr-0.1.1/aautil.cpp /lib/aastr-0.1.1/aautil.h
#	/lib/alfont/alfont.cpp /lib/alfont/alfont.h
#	/lib/allegro/aintern.h
#	/lib/allegro/alconfig.h
#	/lib/allegro/base.h
#	/lib/allegro/color.cpp /lib/allegro/color.h
#	/lib/allegro/config.cpp /lib/allegro/config.h
#	/lib/allegro/draw.cpp /lib/allegro/draw.h
#	/lib/allegro/error.h
#	/lib/allegro/file.cpp /lib/allegro/file.h
#	/lib/allegro/fixed.cpp /lib/allegro/fixed.h
#	/lib/allegro/flood.cpp /lib/allegro/flood.h
#	/lib/allegro/fmaths.h
#	/lib/allegro/gfx.cpp /lib/allegro/gfx.h
#	/lib/allegro/graphics.cpp
#	/lib/allegro/math.cpp
#	/lib/allegro/rotate.cpp /lib/allegro/rotate.h
#	/lib/allegro/surface.cpp /lib/allegro/surface.h
#	/lib/allegro/system.cpp /lib/allegro/system.h
#	/lib/allegro/unicode.cpp /lib/allegro/unicode.h
#	/lib/std/algorithm.h
#	/lib/std/array.h
#	/lib/std/chrono.h
#	/lib/std/functional.h
#	/lib/std/initializer_list.h
#	/lib/std/limits.h
#	/lib/std/list.h
#	/lib/std/map.h
#	/lib/std/math.h
#	/lib/std/memory.h
#	/lib/std/mutex.h
#	/lib/std/queue.h
#	/lib/std/regex.h
#	/lib/std/set.h
#	/lib/std/std.cpp
#	/lib/std/thread.h
#	/lib/std/type_traits.h
#	/lib/std/unordered_set.h
#	/lib/std/utility.h
#	/lib/std/vector.h
#	/lib/std/xtr1common.h
#	/lib/std/xutility.h
#	/lib/system/datetime.cpp /lib/system/datetime.h
#	/lib/allegro.h
#	/plugins/ags_agi/ags_agi.cpp /plugins/ags_agi/ags_agi.h
#	/plugins/ags_blend/ags_blend.cpp /plugins/ags_blend/ags_blend.h
#	/plugins/ags_clipboard/ags_clipboard.cpp /plugins/ags_clipboard/ags_clipboard.h
#	/plugins/ags_controller/ags_controller.cpp /plugins/ags_controller/ags_controller.h
#	/plugins/ags_creditz/ags_creditz.cpp /plugins/ags_creditz/ags_creditz.h
#	/plugins/ags_creditz/ags_creditz1.cpp /plugins/ags_creditz/ags_creditz1.h
#	/plugins/ags_creditz/ags_creditz2.cpp /plugins/ags_creditz/ags_creditz2.h
#	/plugins/ags_creditz/drawing.cpp /plugins/ags_creditz/drawing.h
#	/plugins/ags_fire/ags_fire.cpp /plugins/ags_fire/ags_fire.h
#	/plugins/ags_flashlight/ags_flashlight.cpp /plugins/ags_flashlight/ags_flashlight.h
#	/plugins/ags_flashlight/gfx.cpp
#	/plugins/ags_galaxy_steam/ags_galaxy_steam.cpp /plugins/ags_galaxy_steam/ags_galaxy_steam.h
#	/plugins/ags_galaxy_steam/ags_wadjeteye_steam.cpp /plugins/ags_galaxy_steam/ags_wadjeteye_steam.h
#	/plugins/ags_joy/ags_joy.cpp /plugins/ags_joy/ags_joy.h
#	/plugins/ags_nickenstien_gfx/ags_nickenstien_gfx.cpp /plugins/ags_nickenstien_gfx/ags_nickenstien_gfx.h
#	/plugins/ags_pal_render/ags_pal_render.cpp /plugins/ags_pal_render/ags_pal_render.h
#	/plugins/ags_pal_render/pal_render.h
#	/plugins/ags_pal_render/raycast.cpp /plugins/ags_pal_render/raycast.h
#	/plugins/ags_parallax/ags_parallax.cpp /plugins/ags_parallax/ags_parallax.h
#	/plugins/ags_shell/ags_shell.cpp /plugins/ags_shell/ags_shell.h
#	/plugins/ags_snow_rain/ags_snow_rain.cpp /plugins/ags_snow_rain/ags_snow_rain.h
#	/plugins/ags_snow_rain/weather.cpp /plugins/ags_snow_rain/weather.h
#	/plugins/ags_sock/ags_sock.cpp /plugins/ags_sock/ags_sock.h
#	/plugins/ags_sprite_font/ags_sprite_font.cpp /plugins/ags_sprite_font/ags_sprite_font.h
#	/plugins/ags_sprite_font/ags_sprite_font_clifftop.cpp /plugins/ags_sprite_font/ags_sprite_font_clifftop.h
#	/plugins/ags_sprite_font/character_entry.cpp /plugins/ags_sprite_font/character_entry.h
#	/plugins/ags_sprite_font/color.cpp /plugins/ags_sprite_font/color.h
#	/plugins/ags_sprite_font/sprite_font.cpp /plugins/ags_sprite_font/sprite_font.h
#	/plugins/ags_sprite_font/sprite_font_renderer.cpp /plugins/ags_sprite_font/sprite_font_renderer.h
#	/plugins/ags_sprite_font/sprite_font_renderer_clifftop.cpp /plugins/ags_sprite_font/sprite_font_renderer_clifftop.h
#	/plugins/ags_sprite_font/variable_width_font.cpp /plugins/ags_sprite_font/variable_width_font.h
#	/plugins/ags_sprite_font/variable_width_sprite_font.cpp /plugins/ags_sprite_font/variable_width_sprite_font.h
#	/plugins/ags_sprite_font/variable_width_sprite_font_clifftop.cpp /plugins/ags_sprite_font/variable_width_sprite_font_clifftop.h
#	/plugins/ags_tcp_ip/ags_tcp_ip.cpp /plugins/ags_tcp_ip/ags_tcp_ip.h
#	/plugins/ags_touch/ags_touch.cpp /plugins/ags_touch/ags_touch.h
#	/plugins/ags_wadjet_util/ags_wadjet_util.cpp /plugins/ags_wadjet_util/ags_wadjet_util.h
#	/plugins/ags_waves/ags_waves.cpp /plugins/ags_waves/ags_waves.h
#	/plugins/ags_waves/data.cpp
#	/plugins/ags_waves/draw.cpp
#	/plugins/ags_waves/sound.cpp
#	/plugins/ags_waves/vars.h
#	/plugins/ags_waves/warper.cpp
#	/plugins/ags_waves/weather.cpp
#	/plugins/core/audio_channel.cpp /plugins/core/audio_channel.h
#	/plugins/core/audio_clip.cpp /plugins/core/audio_clip.h
#	/plugins/core/button.cpp /plugins/core/button.h
#	/plugins/core/character.cpp /plugins/core/character.h
#	/plugins/core/core.cpp /plugins/core/core.h
#	/plugins/core/date_time.cpp /plugins/core/date_time.h
#	/plugins/core/dialog.cpp /plugins/core/dialog.h
#	/plugins/core/dialog_options_rendering_info.cpp /plugins/core/dialog_options_rendering_info.h
#	/plugins/core/drawing_surface.cpp /plugins/core/drawing_surface.h
#	/plugins/core/dynamic_sprite.cpp /plugins/core/dynamic_sprite.h
#	/plugins/core/file.cpp /plugins/core/file.h
#	/plugins/core/game.cpp /plugins/core/game.h
#	/plugins/core/global_api.cpp /plugins/core/global_api.h
#	/plugins/core/gui.cpp /plugins/core/gui.h
#	/plugins/core/gui_control.cpp /plugins/core/gui_control.h
#	/plugins/core/hotspot.cpp /plugins/core/hotspot.h
#	/plugins/core/inv_window.cpp /plugins/core/inv_window.h
#	/plugins/core/inventory_item.cpp /plugins/core/inventory_item.h
#	/plugins/core/label.cpp /plugins/core/label.h
#	/plugins/core/listbox.cpp /plugins/core/listbox.h
#	/plugins/core/maths.cpp /plugins/core/maths.h
#	/plugins/core/mouse.cpp /plugins/core/mouse.h
#	/plugins/core/object.cpp /plugins/core/object.h
#	/plugins/core/overlay.cpp /plugins/core/overlay.h
#	/plugins/core/parser.cpp /plugins/core/parser.h
#	/plugins/core/region.cpp /plugins/core/region.h
#	/plugins/core/room.cpp /plugins/core/room.h
#	/plugins/core/slider.cpp /plugins/core/slider.h
#	/plugins/core/string.cpp /plugins/core/string.h
#	/plugins/core/system.cpp /plugins/core/system.h
#	/plugins/core/textbox.cpp /plugins/core/textbox.h
#	/plugins/core/view_frame.cpp /plugins/core/view_frame.h
#	/plugins/ags_plugin.cpp /plugins/ags_plugin.h
#	/plugins/plugin_base.cpp /plugins/plugin_base.h
#	/plugins/plugin_engine.h
#	/plugins/plugin_object_reader.cpp /plugins/plugin_object_reader.h
#	/plugins/serializer.h
#	/shared/ac/dynobj/script_audio_clip.cpp /shared/ac/dynobj/script_audio_clip.h
#	/shared/ac/audio_clip_type.cpp /shared/ac/audio_clip_type.h
#	/shared/ac/character_info.cpp /shared/ac/character_info.h
#	/shared/ac/common.cpp /shared/ac/common.h
#	/shared/ac/common_defines.h
#	/shared/ac/dialog_topic.cpp /shared/ac/dialog_topic.h
#	/shared/ac/game_setup_struct.cpp /shared/ac/game_setup_struct.h
#	/shared/ac/game_setup_struct_base.cpp /shared/ac/game_setup_struct_base.h
#	/shared/ac/game_struct_defines.h
#	/shared/ac/game_version.h
#	/shared/ac/interface_button.h
#	/shared/ac/interface_element.h
#	/shared/ac/inventory_item_info.cpp /shared/ac/inventory_item_info.h
#	/shared/ac/keycode.cpp /shared/ac/keycode.h
#	/shared/ac/mouse_cursor.cpp /shared/ac/mouse_cursor.h
#	/shared/ac/old_game_setup_struct.h
#	/shared/ac/sprite_cache.cpp /shared/ac/sprite_cache.h
#	/shared/ac/sprite_file.cpp /shared/ac/sprite_file.h
#	/shared/ac/view.cpp /shared/ac/view.h
#	/shared/ac/words_dictionary.cpp /shared/ac/words_dictionary.h
#	/shared/core/asset.cpp /shared/core/asset.h
#	/shared/core/asset_manager.cpp /shared/core/asset_manager.h
#	/shared/core/def_version.h
#	/shared/core/platform.h
#	/shared/core/types.h
#	/shared/debugging/debug_manager.cpp /shared/debugging/debug_manager.h
#	/shared/debugging/out.h
#	/shared/debugging/output_handler.h
#	/shared/font/ags_font_renderer.h
#	/shared/font/fonts.cpp /shared/font/fonts.h
#	/shared/font/ttf_font_renderer.cpp /shared/font/ttf_font_renderer.h
#	/shared/font/wfn_font.cpp /shared/font/wfn_font.h
#	/shared/font/wfn_font_renderer.cpp /shared/font/wfn_font_renderer.h
#	/shared/game/custom_properties.cpp /shared/game/custom_properties.h
#	/shared/game/interactions.cpp /shared/game/interactions.h
#	/shared/game/main_game_file.cpp /shared/game/main_game_file.h
#	/shared/game/plugin_info.h
#	/shared/game/room_file.cpp /shared/game/room_file.h
#	/shared/game/room_file_base.cpp
#	/shared/game/room_file_deprecated.cpp
#	/shared/game/room_struct.cpp /shared/game/room_struct.h
#	/shared/game/room_version.h
#	/shared/game/tra_file.cpp /shared/game/tra_file.h
#	/shared/gfx/allegro_bitmap.cpp /shared/gfx/allegro_bitmap.h
#	/shared/gfx/bitmap.cpp /shared/gfx/bitmap.h
#	/shared/gfx/gfx_def.h
#	/shared/gfx/image.cpp /shared/gfx/image.h
#	/shared/gui/gui_button.cpp /shared/gui/gui_button.h
#	/shared/gui/gui_defines.h
#	/shared/gui/gui_inv.cpp /shared/gui/gui_inv.h
#	/shared/gui/gui_label.cpp /shared/gui/gui_label.h
#	/shared/gui/gui_listbox.cpp /shared/gui/gui_listbox.h
#	/shared/gui/gui_main.cpp /shared/gui/gui_main.h
#	/shared/gui/gui_object.cpp /shared/gui/gui_object.h
#	/shared/gui/gui_slider.cpp /shared/gui/gui_slider.h
#	/shared/gui/gui_textbox.cpp /shared/gui/gui_textbox.h
#	/shared/script/cc_error.cpp /shared/script/cc_error.h
#	/shared/script/cc_options.cpp /shared/script/cc_options.h
#	/shared/script/cc_script.cpp /shared/script/cc_script.h
#	/shared/script/script_common.h
#	/shared/util/aligned_stream.cpp /shared/util/aligned_stream.h
#	/shared/util/bbop.h
#	/shared/util/buffered_stream.cpp /shared/util/buffered_stream.h
#	/shared/util/compress.cpp /shared/util/compress.h
#	/shared/util/data_ext.cpp /shared/util/data_ext.h
#	/shared/util/data_stream.cpp /shared/util/data_stream.h
#	/shared/util/directory.cpp /shared/util/directory.h
#	/shared/util/error.h
#	/shared/util/file.cpp /shared/util/file.h
#	/shared/util/file_stream.cpp /shared/util/file_stream.h
#	/shared/util/geometry.cpp /shared/util/geometry.h
#	/shared/util/ini_file.cpp /shared/util/ini_file.h
#	/shared/util/ini_util.cpp /shared/util/ini_util.h
#	/shared/util/lzw.cpp /shared/util/lzw.h
#	/shared/util/math.h
#	/shared/util/memory.h
#	/shared/util/memory_compat.h
#	/shared/util/memory_stream.cpp /shared/util/memory_stream.h
#	/shared/util/multi_file_lib.cpp /shared/util/multi_file_lib.h
#	/shared/util/path.cpp /shared/util/path.h
#	/shared/util/proxy_stream.cpp /shared/util/proxy_stream.h
#	/shared/util/scaling.h
#	/shared/util/stdio_compat.cpp /shared/util/stdio_compat.h
#	/shared/util/stream.cpp /shared/util/stream.h
#	/shared/util/string.cpp /shared/util/string.h
#	/shared/util/string_compat.cpp /shared/util/string_compat.h
#	/shared/util/string_types.h
#	/shared/util/string_utils.cpp /shared/util/string_utils.h
#	/shared/util/text_reader.h
#	/shared/util/text_stream_reader.cpp /shared/util/text_stream_reader.h
#	/shared/util/text_stream_writer.cpp /shared/util/text_stream_writer.h
#	/shared/util/text_writer.h
#	/shared/util/version.cpp /shared/util/version.h
#	/shared/util/wgt2_allg.cpp /shared/util/wgt2_allg.h
#
#	/achievements_tables.h
#	/ags.cpp /ags.h
#	/console.cpp /console.h
#	/detection.h
#	/detection_tables.h
#	/dialogs.cpp
#	/events.cpp /events.h
#	/game_scanner.cpp /game_scanner.h
#	/globals.cpp /globals.h
#	/metaengine.cpp /metaengine.h
#	/module.mk
#	/music.cpp /music.h
#)
#
#add_engine(asylum
#	/puzzles/board.cpp /puzzles/board.h
#	/puzzles/boardkeyhidesto.cpp /puzzles/boardkeyhidesto.h
#	/puzzles/boardsalvation.cpp /puzzles/boardsalvation.h
#	/puzzles/boardyouth.cpp /puzzles/boardyouth.h
#	/puzzles/clock.cpp /puzzles/clock.h
#	/puzzles/fisherman.cpp /puzzles/fisherman.h
#	/puzzles/hivecontrol.cpp /puzzles/hivecontrol.h
#	/puzzles/hivemachine.cpp /puzzles/hivemachine.h
#	/puzzles/lock.cpp /puzzles/lock.h
#	/puzzles/morguedoor.cpp /puzzles/morguedoor.h
#	/puzzles/pipes.cpp /puzzles/pipes.h
#	/puzzles/puzzle.cpp /puzzles/puzzle.h
#	/puzzles/puzzles.cpp /puzzles/puzzles.h
#	/puzzles/tictactoe.cpp /puzzles/tictactoe.h
#	/puzzles/timemachine.cpp /puzzles/timemachine.h
#	/puzzles/vcr.cpp /puzzles/vcr.h
#	/puzzles/wheel.cpp /puzzles/wheel.h
#	/puzzles/writings.cpp /puzzles/writings.h
#	/resources/actor.cpp /resources/actor.h
#	/resources/data.cpp /resources/data.h
#	/resources/encounters.cpp /resources/encounters.h
#	/resources/inventory.cpp /resources/inventory.h
#	/resources/object.cpp /resources/object.h
#	/resources/polygons.cpp /resources/polygons.h
#	/resources/script.cpp /resources/script.h
#	/resources/special.cpp /resources/special.h
#	/resources/worldstats.cpp /resources/worldstats.h
#	/system/config.cpp /system/config.h
#	/system/cursor.cpp /system/cursor.h
#	/system/graphics.cpp /system/graphics.h
#	/system/savegame.cpp /system/savegame.h
#	/system/screen.cpp /system/screen.h
#	/system/sound.cpp /system/sound.h
#	/system/speech.cpp /system/speech.h
#	/system/text.cpp /system/text.h
#	/views/insertdisc.cpp /views/insertdisc.h
#	/views/menu.cpp /views/menu.h
#	/views/resviewer.cpp /views/resviewer.h
#	/views/scene.cpp /views/scene.h
#	/views/scenetitle.cpp /views/scenetitle.h
#	/views/video.cpp /views/video.h
#	/asylum.cpp /asylum.h
#	/console.cpp /console.h
#	/detection_tables.h
#	/eventhandler.h
#	/metaengine.cpp
#	/module.mk
#	/respack.cpp /respack.h
#	/shared.h
#	/staticres.h
#)
#
#add_engine(bbvs
#	/minigames/bbairguitar.cpp /minigames/bbairguitar.h
#	/minigames/bbairguitar_anims.cpp
#	/minigames/bbant.cpp /minigames/bbant.h
#	/minigames/bbant_anims.cpp
#	/minigames/bbloogie.cpp /minigames/bbloogie.h
#	/minigames/bbloogie_anims.cpp
#	/minigames/bbtennis.cpp /minigames/bbtennis.h
#	/minigames/bbtennis_anims.cpp
#	/minigames/minigame.cpp /minigames/minigame.h
#	/bbvs.cpp /bbvs.h
#	/detection.h
#	/dialogs.cpp /dialogs.h
#	/gamemodule.cpp /gamemodule.h
#	/graphics.cpp /graphics.h
#	/logic.cpp
#	/metaengine.cpp
#	/module.mk
#	/saveload.cpp
#	/scene.cpp
#	/sound.cpp /sound.h
#	/spritemodule.cpp /spritemodule.h
#	/videoplayer.cpp
#	/walk.cpp
#)
#
#add_engine(bladerunner
#	/script/ai/answering_machine.cpp
#	/script/ai/baker.cpp
#	/script/ai/blimp_guy.cpp
#	/script/ai/bryant.cpp
#	/script/ai/bullet_bob.cpp
#	/script/ai/chew.cpp
#	/script/ai/clovis.cpp
#	/script/ai/crazylegs.cpp
#	/script/ai/dektora.cpp
#	/script/ai/desk_clerk.cpp
#	/script/ai/dispatcher.cpp
#	/script/ai/early_q.cpp
#	/script/ai/early_q_bartender.cpp
#	/script/ai/fish_dealer.cpp
#	/script/ai/free_slot_a.cpp
#	/script/ai/free_slot_b.cpp
#	/script/ai/gaff.cpp
#	/script/ai/general_doll.cpp
#	/script/ai/generic_walker_a.cpp
#	/script/ai/generic_walker_b.cpp
#	/script/ai/generic_walker_c.cpp
#	/script/ai/gordo.cpp
#	/script/ai/governor_kolvig.cpp
#	/script/ai/grigorian.cpp
#	/script/ai/guzza.cpp
#	/script/ai/hanoi.cpp
#	/script/ai/hasan.cpp
#	/script/ai/hawkers_barkeep.cpp
#	/script/ai/hawkers_parrot.cpp
#	/script/ai/holloway.cpp
#	/script/ai/howie_lee.cpp
#	/script/ai/hysteria_patron1.cpp
#	/script/ai/hysteria_patron2.cpp
#	/script/ai/hysteria_patron3.cpp
#	/script/ai/insect_dealer.cpp
#	/script/ai/isabella.cpp
#	/script/ai/izo.cpp
#	/script/ai/klein.cpp
#	/script/ai/lance.cpp
#	/script/ai/leon.cpp
#	/script/ai/lockup_guard.cpp
#	/script/ai/lucy.cpp
#	/script/ai/luther.cpp
#	/script/ai/maggie.cpp
#	/script/ai/male_announcer.cpp
#	/script/ai/marcus.cpp
#	/script/ai/mccoy.cpp
#	/script/ai/mia.cpp
#	/script/ai/moraji.cpp
#	/script/ai/murray.cpp
#	/script/ai/mutant1.cpp
#	/script/ai/mutant2.cpp
#	/script/ai/mutant3.cpp
#	/script/ai/newscaster.cpp
#	/script/ai/officer_grayford.cpp
#	/script/ai/officer_leary.cpp
#	/script/ai/photographer.cpp
#	/script/ai/rachael.cpp
#	/script/ai/rajif.cpp
#	/script/ai/runciter.cpp
#	/script/ai/sadik.cpp
#	/script/ai/sebastian.cpp
#	/script/ai/sergeant_walls.cpp
#	/script/ai/shoeshine_man.cpp
#	/script/ai/steele.cpp
#	/script/ai/taffy.cpp
#	/script/ai/taffy_patron.cpp
#	/script/ai/teenager.cpp
#	/script/ai/the_bard.cpp
#	/script/ai/transient.cpp
#	/script/ai/tyrell.cpp
#	/script/ai/tyrell_guard.cpp
#	/script/ai/zuben.cpp
#	/script/scene/ar01.cpp
#	/script/scene/ar02.cpp
#	/script/scene/bb01.cpp
#	/script/scene/bb02.cpp
#	/script/scene/bb03.cpp
#	/script/scene/bb04.cpp
#	/script/scene/bb05.cpp
#	/script/scene/bb06.cpp
#	/script/scene/bb07.cpp
#	/script/scene/bb08.cpp
#	/script/scene/bb09.cpp
#	/script/scene/bb10.cpp
#	/script/scene/bb11.cpp
#	/script/scene/bb12.cpp
#	/script/scene/bb51.cpp
#	/script/scene/ct01.cpp
#	/script/scene/ct02.cpp
#	/script/scene/ct03.cpp
#	/script/scene/ct04.cpp
#	/script/scene/ct05.cpp
#	/script/scene/ct06.cpp
#	/script/scene/ct07.cpp
#	/script/scene/ct08.cpp
#	/script/scene/ct09.cpp
#	/script/scene/ct10.cpp
#	/script/scene/ct11.cpp
#	/script/scene/ct12.cpp
#	/script/scene/ct51.cpp
#	/script/scene/dr01.cpp
#	/script/scene/dr02.cpp
#	/script/scene/dr03.cpp
#	/script/scene/dr04.cpp
#	/script/scene/dr05.cpp
#	/script/scene/dr06.cpp
#	/script/scene/hc01.cpp
#	/script/scene/hc02.cpp
#	/script/scene/hc03.cpp
#	/script/scene/hc04.cpp
#	/script/scene/hf01.cpp
#	/script/scene/hf02.cpp
#	/script/scene/hf03.cpp
#	/script/scene/hf04.cpp
#	/script/scene/hf05.cpp
#	/script/scene/hf06.cpp
#	/script/scene/hf07.cpp
#	/script/scene/kp01.cpp
#	/script/scene/kp02.cpp
#	/script/scene/kp03.cpp
#	/script/scene/kp04.cpp
#	/script/scene/kp05.cpp
#	/script/scene/kp06.cpp
#	/script/scene/kp07.cpp
#	/script/scene/ma01.cpp
#	/script/scene/ma02.cpp
#	/script/scene/ma04.cpp
#	/script/scene/ma05.cpp
#	/script/scene/ma06.cpp
#	/script/scene/ma07.cpp
#	/script/scene/ma08.cpp
#	/script/scene/nr01.cpp
#	/script/scene/nr02.cpp
#	/script/scene/nr03.cpp
#	/script/scene/nr04.cpp
#	/script/scene/nr05.cpp
#	/script/scene/nr06.cpp
#	/script/scene/nr07.cpp
#	/script/scene/nr08.cpp
#	/script/scene/nr09.cpp
#	/script/scene/nr10.cpp
#	/script/scene/nr11.cpp
#	/script/scene/ps01.cpp
#	/script/scene/ps02.cpp
#	/script/scene/ps03.cpp
#	/script/scene/ps04.cpp
#	/script/scene/ps05.cpp
#	/script/scene/ps06.cpp
#	/script/scene/ps07.cpp
#	/script/scene/ps09.cpp
#	/script/scene/ps10.cpp
#	/script/scene/ps11.cpp
#	/script/scene/ps12.cpp
#	/script/scene/ps13.cpp
#	/script/scene/ps14.cpp
#	/script/scene/ps15.cpp
#	/script/scene/rc01.cpp
#	/script/scene/rc02.cpp
#	/script/scene/rc03.cpp
#	/script/scene/rc04.cpp
#	/script/scene/rc51.cpp
#	/script/scene/tb02.cpp
#	/script/scene/tb03.cpp
#	/script/scene/tb05.cpp
#	/script/scene/tb06.cpp
#	/script/scene/tb07.cpp
#	/script/scene/ug01.cpp
#	/script/scene/ug02.cpp
#	/script/scene/ug03.cpp
#	/script/scene/ug04.cpp
#	/script/scene/ug05.cpp
#	/script/scene/ug06.cpp
#	/script/scene/ug07.cpp
#	/script/scene/ug08.cpp
#	/script/scene/ug09.cpp
#	/script/scene/ug10.cpp
#	/script/scene/ug12.cpp
#	/script/scene/ug13.cpp
#	/script/scene/ug14.cpp
#	/script/scene/ug15.cpp
#	/script/scene/ug16.cpp
#	/script/scene/ug17.cpp
#	/script/scene/ug18.cpp
#	/script/scene/ug19.cpp
#	/script/ai_script.cpp /script/ai_script.h
#	/script/esper_script.cpp /script/esper_script.h
#	/script/init_script.cpp /script/init_script.h
#	/script/kia_script.cpp /script/kia_script.h
#	/script/police_maze.cpp /script/police_maze.h
#	/script/scene_script.cpp /script/scene_script.h
#	/script/script.cpp /script/script.h
#	/script/vk_script.cpp /script/vk_script.h
#	/ui/elevator.cpp /ui/elevator.h
#	/ui/end_credits.cpp /ui/end_credits.h
#	/ui/esper.cpp /ui/esper.h
#	/ui/kia.cpp /ui/kia.h
#	/ui/kia_log.cpp /ui/kia_log.h
#	/ui/kia_section_base.cpp /ui/kia_section_base.h
#	/ui/kia_section_clues.cpp /ui/kia_section_clues.h
#	/ui/kia_section_crimes.cpp /ui/kia_section_crimes.h
#	/ui/kia_section_diagnostic.cpp /ui/kia_section_diagnostic.h
#	/ui/kia_section_help.cpp /ui/kia_section_help.h
#	/ui/kia_section_load.cpp /ui/kia_section_load.h
#	/ui/kia_section_pogo.cpp /ui/kia_section_pogo.h
#	/ui/kia_section_save.cpp /ui/kia_section_save.h
#	/ui/kia_section_settings.cpp /ui/kia_section_settings.h
#	/ui/kia_section_suspects.cpp /ui/kia_section_suspects.h
#	/ui/scores.cpp /ui/scores.h
#	/ui/spinner.cpp /ui/spinner.h
#	/ui/ui_check_box.cpp /ui/ui_check_box.h
#	/ui/ui_component.h
#	/ui/ui_container.cpp /ui/ui_container.h
#	/ui/ui_dropdown.cpp /ui/ui_dropdown.h
#	/ui/ui_image_picker.cpp /ui/ui_image_picker.h
#	/ui/ui_input_box.cpp /ui/ui_input_box.h
#	/ui/ui_scroll_box.cpp /ui/ui_scroll_box.h
#	/ui/ui_slider.cpp /ui/ui_slider.h
#	/ui/vk.cpp /ui/vk.h
#	/actor.cpp /actor.h
#	/actor_clues.cpp /actor_clues.h
#	/actor_combat.cpp /actor_combat.h
#	/actor_dialogue_queue.cpp /actor_dialogue_queue.h
#	/actor_walk.cpp /actor_walk.h
#	/adpcm_decoder.cpp /adpcm_decoder.h
#	/ambient_sounds.cpp /ambient_sounds.h
#	/archive.cpp /archive.h
#	/aud_stream.cpp /aud_stream.h
#	/audio_cache.cpp /audio_cache.h
#	/audio_mixer.cpp /audio_mixer.h
#	/audio_player.cpp /audio_player.h
#	/audio_speech.cpp /audio_speech.h
#	/bladerunner.cpp /bladerunner.h
#	/boundingbox.cpp /boundingbox.h
#	/chapters.cpp /chapters.h
#	/color.cpp /color.h
#	/combat.cpp /combat.h
#	/crimes_database.cpp /crimes_database.h
#	/debugger.cpp /debugger.h
#	/decompress_lcw.cpp /decompress_lcw.h
#	/decompress_lzo.cpp /decompress_lzo.h
#	/detection_tables.h
#	/dialogue_menu.cpp /dialogue_menu.h
#	/fog.cpp /fog.h
#	/font.cpp /font.h
#	/framelimiter.cpp /framelimiter.h
#	/game_constants.h
#	/game_flags.cpp /game_flags.h
#	/game_info.cpp /game_info.h
#	/image.cpp /image.h
#	/item.cpp /item.h
#	/item_pickup.cpp /item_pickup.h
#	/items.cpp /items.h
#	/light.cpp /light.h
#	/lights.cpp /lights.h
#	/matrix.cpp /matrix.h
#	/metaengine.cpp
#	/module.mk
#	/mouse.cpp /mouse.h
#	/movement_track.cpp /movement_track.h
#	/music.cpp /music.h
#	/obstacles.cpp /obstacles.h
#	/outtake.cpp /outtake.h
#	/overlays.cpp /overlays.h
#	/rect_float.h
#	/regions.cpp /regions.h
#	/savefile.cpp /savefile.h
#	/scene.cpp /scene.h
#	/scene_objects.cpp /scene_objects.h
#	/screen_effects.cpp /screen_effects.h
#	/set.cpp /set.h
#	/set_effects.cpp /set_effects.h
#	/settings.cpp /settings.h
#	/shape.cpp /shape.h
#	/slice_animations.cpp /slice_animations.h
#	/slice_renderer.cpp /slice_renderer.h
#	/subtitles.cpp /subtitles.h
#	/suspects_database.cpp /suspects_database.h
#	/text_resource.cpp /text_resource.h
#	/time.cpp /time.h
#	/vector.h
#	/view.cpp /view.h
#	/vqa_decoder.cpp /vqa_decoder.h
#	/vqa_player.cpp /vqa_player.h
#	/waypoints.cpp /waypoints.h
#	/zbuffer.cpp /zbuffer.h
#)
#
#add_engine(buried
#	/demo/demo_menu.cpp /demo/demo_menu.h
#	/demo/features.cpp /demo/features.h
#	/demo/movie_scene.cpp /demo/movie_scene.h
#	/environ/agent3_lair.cpp
#	/environ/ai_lab.cpp
#	/environ/alien.cpp
#	/environ/castle.cpp
#	/environ/da_vinci.cpp
#	/environ/future_apartment.cpp
#	/environ/mayan.cpp
#	/environ/scene_base.cpp /environ/scene_base.h
#	/environ/scene_common.cpp /environ/scene_common.h
#	/environ/scene_factory.cpp
#	/agent_evaluation.cpp /agent_evaluation.h
#	/aidata.h
#	/animdata.h
#	/avi_frames.cpp /avi_frames.h
#	/biochip_right.cpp /biochip_right.h
#	/biochip_view.cpp /biochip_view.h
#	/bookdata.h
#	/buried.cpp /buried.h
#	/complete.cpp /complete.h
#	/console.cpp /console.h
#	/credits.cpp /credits.h
#	/death.cpp /death.h
#	/detection_tables.h
#	/fbcdata.h
#	/frame_window.cpp /frame_window.h
#	/gameui.cpp /gameui.h
#	/global_flags.h
#	/graphics.cpp /graphics.h
#	/inndata.h
#	/invdata.h
#	/inventory_info.cpp /inventory_info.h
#	/inventory_window.cpp /inventory_window.h
#	/livetext.cpp /livetext.h
#	/main_menu.cpp /main_menu.h
#	/message.h
#	/metaengine.cpp
#	/module.mk
#	/navarrow.cpp /navarrow.h
#	/navdata.h
#	/overview.cpp /overview.h
#	/resources.h
#	/saveload.cpp
#	/scene_view.cpp /scene_view.h
#	/snddata.h
#	/sound.cpp /sound.h
#	/sprtdata.h
#	/title_sequence.cpp /title_sequence.h
#	/video_window.cpp /video_window.h
#	/window.cpp /window.h
#)
#
#add_engine(cge
#	/bitmap.cpp /bitmap.h
#	/cge.cpp /cge.h
#	/cge_main.cpp /cge_main.h
#	/console.cpp /console.h
#	/events.cpp /events.h
#	/fileio.cpp /fileio.h
#	/game.cpp /game.h
#	/general.h
#	/metaengine.cpp
#	/module.mk
#	/snail.cpp /snail.h
#	/sound.cpp /sound.h
#	/talk.cpp /talk.h
#	/text.cpp /text.h
#	/vga13h.cpp /vga13h.h
#	/vmenu.cpp /vmenu.h
#	/walk.cpp /walk.h
#)
#
#add_engine(cge2
#	/bitmap.cpp /bitmap.h
#	/cge2.cpp /cge2.h
#	/cge2_main.cpp /cge2_main.h
#	/console.cpp /console.h
#	/events.cpp /events.h
#	/fileio.cpp /fileio.h
#	/general.h
#	/hero.cpp /hero.h
#	/inventory.cpp
#	/map.cpp /map.h
#	/metaengine.cpp
#	/module.mk
#	/saveload.cpp
#	/snail.cpp /snail.h
#	/sound.cpp /sound.h
#	/spare.cpp /spare.h
#	/talk.cpp /talk.h
#	/text.cpp /text.h
#	/toolbar.cpp
#	/vga13h.cpp /vga13h.h
#	/vmenu.cpp /vmenu.h
#)
#
#add_engine(cine
#	/anim.cpp /anim.h
#	/bg.cpp /bg.h
#	/bg_list.cpp /bg_list.h
#	/cine.cpp /cine.h
#	/console.cpp /console.h
#	/detection.h
#	/detection_tables.h
#	/gfx.cpp /gfx.h
#	/main_loop.cpp /main_loop.h
#	/metaengine.cpp
#	/module.mk
#	/msg.cpp /msg.h
#	/object.cpp /object.h
#	/pal.cpp /pal.h
#	/part.cpp /part.h
#	/prc.cpp /prc.h
#	/rel.cpp /rel.h
#	/saveload.cpp /saveload.h
#	/script.h
#	/script_fw.cpp
#	/script_os.cpp
#	/sound.cpp /sound.h
#	/texte.cpp /texte.h
#	/unpack.cpp /unpack.h
#	/various.cpp /various.h
#)
#
#add_engine(composer
#	/composer.cpp /composer.h
#	/console.cpp /console.h
#	/detection.h
#	/detection_tables.h
#	/graphics.cpp /graphics.h
#	/metaengine.cpp
#	/module.mk
#	/resource.cpp /resource.h
#	/saveload.cpp
#	/scripting.cpp
#)
#
#add_engine(cruise
#	/actor.cpp /actor.h
#	/background.cpp /background.h
#	/backgroundIncrust.cpp /backgroundIncrust.h
#	/cell.cpp /cell.h
#	/cruise.cpp /cruise.h
#	/cruise_main.cpp /cruise_main.h
#	/ctp.cpp /ctp.h
#	/dataLoader.cpp /dataLoader.h
#	/debugger.cpp /debugger.h
#	/decompiler.cpp
#	/delphine-unpack.cpp
#	/detection.h
#	/font.cpp /font.h
#	/function.cpp /function.h
#	/gfxModule.cpp /gfxModule.h
#	/linker.cpp /linker.h
#	/mainDraw.cpp /mainDraw.h
#	/menu.cpp /menu.h
#	/metaengine.cpp
#	/module.mk
#	/mouse.cpp /mouse.h
#	/object.cpp /object.h
#	/overlay.cpp /overlay.h
#	/perso.cpp /perso.h
#	/polys.cpp /polys.h
#	/saveload.cpp /saveload.h
#	/script.cpp /script.h
#	/sound.cpp /sound.h
#	/stack.cpp /stack.h
#	/staticres.cpp /staticres.h
#	/various.cpp /various.h
#	/vars.cpp /vars.h
#	/volume.cpp /volume.h
#)
#
#add_engine(cryomni3d
#	/fonts/cryoextfont.cpp /fonts/cryoextfont.h
#	/fonts/cryofont.cpp /fonts/cryofont.h
#	/image/hlz.cpp /image/hlz.h
#	/versailles/data.cpp
#	/versailles/dialogs.cpp
#	/versailles/dialogs_manager.cpp /versailles/dialogs_manager.h
#	/versailles/documentation.cpp /versailles/documentation.h
#	/versailles/engine.cpp /versailles/engine.h
#	/versailles/logic.cpp
#	/versailles/menus.cpp
#	/versailles/music.cpp
#	/versailles/saveload.cpp
#	/versailles/toolbar.cpp /versailles/toolbar.h
#	/cryomni3d.cpp /cryomni3d.h
#	/datstream.cpp /datstream.h
#	/detection.h
#	/detection_tables.h
#	/dialogs_manager.cpp /dialogs_manager.h
#	/fixed_image.cpp /fixed_image.h
#	/font_manager.cpp /font_manager.h
#	/metaengine.cpp
#	/module.mk
#	/mouse_boxes.cpp /mouse_boxes.h
#	/objects.cpp /objects.h
#	/omni3d.cpp /omni3d.h
#	/sprites.cpp /sprites.h
#	/wam_parser.cpp /wam_parser.h
#)
#
#add_engine(director
#	/lingo/xlibs/cdromxobj.cpp /lingo/xlibs/cdromxobj.h
#	/lingo/xlibs/fileio.cpp /lingo/xlibs/fileio.h
#	/lingo/xlibs/flushxobj.cpp /lingo/xlibs/flushxobj.h
#	/lingo/xlibs/fplayxobj.cpp /lingo/xlibs/fplayxobj.h
#	/lingo/xlibs/labeldrvxobj.cpp /lingo/xlibs/labeldrvxobj.h
#	/lingo/xlibs/memoryxobj.cpp /lingo/xlibs/memoryxobj.h
#	/lingo/xlibs/movemousexobj.cpp /lingo/xlibs/movemousexobj.h
#	/lingo/xlibs/orthoplayxobj.cpp /lingo/xlibs/orthoplayxobj.h
#	/lingo/xlibs/palxobj.cpp /lingo/xlibs/palxobj.h
#	/lingo/xlibs/popupmenuxobj.cpp /lingo/xlibs/popupmenuxobj.h
#	/lingo/xlibs/registercomponent.cpp /lingo/xlibs/registercomponent.h
#	/lingo/xlibs/serialportxobj.cpp /lingo/xlibs/serialportxobj.h
#	/lingo/xlibs/soundjam.cpp /lingo/xlibs/soundjam.h
#	/lingo/xlibs/videodiscxobj.cpp /lingo/xlibs/videodiscxobj.h
#	/lingo/xlibs/winxobj.cpp /lingo/xlibs/winxobj.h
#	/lingo/xlibs/xplayanim.cpp /lingo/xlibs/xplayanim.h
#	/lingo/lingo-ast.h
#	/lingo/lingo-builtins.cpp /lingo/lingo-builtins.h
#	/lingo/lingo-bytecode.cpp /lingo/lingo-bytecode.h
#	/lingo/lingo-code.cpp /lingo/lingo-code.h
#	/lingo/lingo-codegen.cpp /lingo/lingo-codegen.h
#	/lingo/lingo-events.cpp
#	/lingo/lingo-funcs.cpp
#	/lingo/lingo-gr.cpp /lingo/lingo-gr.h
#	/lingo/lingo-lex.cpp
#	/lingo/lingo-object.cpp /lingo/lingo-object.h
#	/lingo/lingo-patcher.cpp
#	/lingo/lingo-preprocessor.cpp
#	/lingo/lingo-the.cpp /lingo/lingo-the.h
#	/lingo/lingo-utils.cpp /lingo/lingo-utils.h
#	/lingo/lingo.cpp /lingo/lingo.h
#	/archive.cpp /archive.h
#	/cast.cpp /cast.h
#	/castmember.cpp /castmember.h
#	/channel.cpp /channel.h
#	/cursor.cpp /cursor.h
#	/debugger.cpp /debugger.h
#	/detection.h
#	/detection_paths.h
#	/detection_tables.h
#	/director.cpp /director.h
#	/events.cpp
#	/fonts.cpp
#	/frame.cpp /frame.h
#	/graphics-data.h
#	/graphics.cpp
#	/images.cpp /images.h
#	/metaengine.cpp
#	/module.mk
#	/movie.cpp /movie.h
#	/resource.cpp
#	/score.cpp /score.h
#	/sound.cpp /sound.h
#	/sprite.cpp /sprite.h
#	/stxt.cpp /stxt.h
#	/tests.cpp
#	/transitions.cpp
#	/types.h
#	/util.cpp /util.h
#	/window.cpp /window.h
#)
#
#add_engine(draci
#	/animation.cpp /animation.h
#	/barchive.cpp /barchive.h
#	/console.cpp /console.h
#	/draci.cpp /draci.h
#	/font.cpp /font.h
#	/game.cpp /game.h
#	/metaengine.cpp
#	/module.mk
#	/mouse.cpp /mouse.h
#	/music.cpp /music.h
#	/saveload.cpp /saveload.h
#	/screen.cpp /screen.h
#	/script.cpp /script.h
#	/sound.cpp /sound.h
#	/sprite.cpp /sprite.h
#	/surface.cpp /surface.h
#	/walking.cpp /walking.h
#)
#
#add_engine(dragons
#	/actor.cpp /actor.h
#	/actorfiles.h
#	/actorresource.cpp /actorresource.h
#	/background.cpp /background.h
#	/bag.cpp /bag.h
#	/bigfile.cpp /bigfile.h
#	/credits.cpp /credits.h
#	/cursor.cpp /cursor.h
#	/cutscene.cpp /cutscene.h
#	/detection.h
#	/dragonflg.cpp /dragonflg.h
#	/dragonimg.cpp /dragonimg.h
#	/dragonini.cpp /dragonini.h
#	/dragonobd.cpp /dragonobd.h
#	/dragonrms.cpp /dragonrms.h
#	/dragons.cpp /dragons.h
#	/dragonvar.cpp /dragonvar.h
#	/font.cpp /font.h
#	/inventory.cpp /inventory.h
#	/metaengine.cpp
#	/midimusicplayer.cpp /midimusicplayer.h
#	/minigame1.cpp /minigame1.h
#	/minigame2.cpp /minigame2.h
#	/minigame3.cpp /minigame3.h
#	/minigame4.cpp /minigame4.h
#	/minigame5.cpp /minigame5.h
#	/module.mk
#	/saveload.cpp
#	/scene.cpp /scene.h
#	/screen.cpp /screen.h
#	/scriptopcodes.cpp /scriptopcodes.h
#	/sequenceopcodes.cpp /sequenceopcodes.h
#	/sound.cpp /sound.h
#	/specialopcodes.cpp /specialopcodes.h
#	/strplayer.cpp /strplayer.h
#	/talk.cpp /talk.h
#	/vabsound.cpp /vabsound.h
#)
#
#add_engine(drascula
#	/actors.cpp
#	/animation.cpp
#	/console.cpp /console.h
#	/converse.cpp
#	/detection.h
#	/drascula.cpp /drascula.h
#	/graphics.cpp
#	/interface.cpp
#	/metaengine.cpp
#	/module.mk
#	/objects.cpp
#	/palette.cpp
#	/resource.cpp
#	/rooms.cpp
#	/saveload.cpp
#	/sound.cpp
#	/talk.cpp
#)
#
#add_engine(dreamweb
#	/backdrop.cpp
#	/console.cpp /console.h
#	/detection.h
#	/detection_tables.h
#	/dreamweb.cpp /dreamweb.h
#	/keypad.cpp
#	/metaengine.cpp
#	/module.mk
#	/monitor.cpp
#	/mouse.cpp
#	/newplace.cpp
#	/object.cpp
#	/pathfind.cpp
#	/people.cpp
#	/print.cpp
#	/rain.cpp
#	/saveload.cpp
#	/sound.cpp /sound.h
#	/sprite.cpp
#	/structs.h
#	/stubs.cpp
#	/talk.cpp
#	/titles.cpp
#	/use.cpp
#	/vgafades.cpp
#	/vgagrafx.cpp
#)
#
#add_engine(glk
#	/adrift/adrift.cpp /adrift/adrift.h
#	/adrift/detection.cpp /adrift/detection.h
#	/adrift/detection_tables.h
#	/adrift/os_glk.cpp /adrift/os_glk.h
#	/adrift/scare.h
#	/adrift/scdebug.cpp
#	/adrift/scevents.cpp
#	/adrift/scexpr.cpp
#	/adrift/scgamest.cpp /adrift/scgamest.h
#	/adrift/scinterf.cpp
#	/adrift/sclibrar.cpp
#	/adrift/sclocale.cpp
#	/adrift/scmemos.cpp
#	/adrift/scnpcs.cpp
#	/adrift/scobjcts.cpp
#	/adrift/scparser.cpp
#	/adrift/scprintf.cpp
#	/adrift/scprops.cpp
#	/adrift/scprotos.h
#	/adrift/scresour.cpp
#	/adrift/screstrs.cpp
#	/adrift/scrunner.cpp
#	/adrift/sctaffil.cpp
#	/adrift/sctafpar.cpp
#	/adrift/sctasks.cpp
#	/adrift/scutils.cpp
#	/adrift/scvars.cpp
#	/adrift/serialization.cpp /adrift/serialization.h
#	/adrift/sxfile.cpp
#	/adrift/sxglob.cpp
#	/adrift/sxprotos.h
#	/adrift/sxutils.cpp
#	/advsys/advsys.cpp /advsys/advsys.h
#	/advsys/definitions.h
#	/advsys/detection.cpp /advsys/detection.h
#	/advsys/detection_tables.h
#	/advsys/game.cpp /advsys/game.h
#	/advsys/glk_interface.cpp /advsys/glk_interface.h
#	/advsys/vm.cpp /advsys/vm.h
#	/agt/agil.cpp
#	/agt/agility.h
#	/agt/agt.cpp /agt/agt.h
#	/agt/agtread.cpp
#	/agt/agxfile.cpp
#	/agt/auxfile.cpp
#	/agt/config.h
#	/agt/debugcmd.cpp
#	/agt/detection.cpp /agt/detection.h
#	/agt/detection_tables.h
#	/agt/disassemble.cpp
#	/agt/exec.cpp /agt/exec.h
#	/agt/filename.cpp
#	/agt/gamedata.cpp
#	/agt/interface.cpp
#	/agt/interp.h
#	/agt/metacommand.cpp
#	/agt/object.cpp
#	/agt/os_glk.cpp
#	/agt/parser.cpp
#	/agt/runverb.cpp
#	/agt/savegame.cpp
#	/agt/token.cpp
#	/agt/util.cpp
#	/agt/vars.cpp
#	/alan2/acode.h
#	/alan2/alan2.cpp /alan2/alan2.h
#	/alan2/alan_version.cpp /alan2/alan_version.h
#	/alan2/args.cpp /alan2/args.h
#	/alan2/debug.cpp /alan2/debug.h
#	/alan2/decode.cpp /alan2/decode.h
#	/alan2/detection.cpp /alan2/detection.h
#	/alan2/detection_tables.h
#	/alan2/exe.cpp /alan2/exe.h
#	/alan2/glkio.cpp /alan2/glkio.h
#	/alan2/inter.cpp /alan2/inter.h
#	/alan2/main.cpp /alan2/main.h
#	/alan2/params.cpp /alan2/params.h
#	/alan2/parse.cpp /alan2/parse.h
#	/alan2/reverse.cpp /alan2/reverse.h
#	/alan2/rules.cpp /alan2/rules.h
#	/alan2/stack.cpp /alan2/stack.h
#	/alan2/sysdep.cpp /alan2/sysdep.h
#	/alan2/types.cpp /alan2/types.h
#	/alan2/version.h
#	/alan3/acode.cpp /alan3/acode.h
#	/alan3/act.cpp /alan3/act.h
#	/alan3/actor.cpp /alan3/actor.h
#	/alan3/alan3.cpp /alan3/alan3.h
#	/alan3/alan_version.cpp /alan3/alan_version.h
#	/alan3/alt_info.cpp /alan3/alt_info.h
#	/alan3/attribute.cpp /alan3/attribute.h
#	/alan3/checkentry.cpp /alan3/checkentry.h
#	/alan3/class.cpp /alan3/class.h
#	/alan3/compatibility.cpp /alan3/compatibility.h
#	/alan3/container.cpp /alan3/container.h
#	/alan3/current.cpp /alan3/current.h
#	/alan3/debug.cpp /alan3/debug.h
#	/alan3/decode.cpp /alan3/decode.h
#	/alan3/detection.cpp /alan3/detection.h
#	/alan3/detection_tables.h
#	/alan3/dictionary.cpp /alan3/dictionary.h
#	/alan3/event.cpp /alan3/event.h
#	/alan3/exe.cpp /alan3/exe.h
#	/alan3/fnmatch.cpp /alan3/fnmatch.h
#	/alan3/glkio.cpp /alan3/glkio.h
#	/alan3/instance.cpp /alan3/instance.h
#	/alan3/inter.cpp /alan3/inter.h
#	/alan3/lists.cpp /alan3/lists.h
#	/alan3/literal.cpp /alan3/literal.h
#	/alan3/location.cpp /alan3/location.h
#	/alan3/main.cpp /alan3/main.h
#	/alan3/memory.cpp /alan3/memory.h
#	/alan3/msg.cpp /alan3/msg.h
#	/alan3/options.cpp /alan3/options.h
#	/alan3/output.cpp /alan3/output.h
#	/alan3/parameter_position.cpp /alan3/parameter_position.h
#	/alan3/params.cpp /alan3/params.h
#	/alan3/parse.cpp /alan3/parse.h
#	/alan3/resources.h
#	/alan3/reverse.cpp /alan3/reverse.h
#	/alan3/rules.cpp /alan3/rules.h
#	/alan3/save.cpp /alan3/save.h
#	/alan3/scan.cpp /alan3/scan.h
#	/alan3/score.cpp /alan3/score.h
#	/alan3/set.cpp /alan3/set.h
#	/alan3/stack.cpp /alan3/stack.h
#	/alan3/state.cpp /alan3/state.h
#	/alan3/state_stack.cpp /alan3/state_stack.h
#	/alan3/syntax.cpp /alan3/syntax.h
#	/alan3/sysdep.cpp /alan3/sysdep.h
#	/alan3/syserr.cpp /alan3/syserr.h
#	/alan3/types.cpp /alan3/types.h
#	/alan3/utils.cpp /alan3/utils.h
#	/alan3/version.h
#	/alan3/word.cpp /alan3/word.h
#	/archetype/archetype.cpp /archetype/archetype.h
#	/archetype/array.cpp /archetype/array.h
#	/archetype/crypt.cpp /archetype/crypt.h
#	/archetype/detection.cpp /archetype/detection.h
#	/archetype/detection_tables.h
#	/archetype/error.cpp /archetype/error.h
#	/archetype/expression.cpp /archetype/expression.h
#	/archetype/game_stat.cpp /archetype/game_stat.h
#	/archetype/heap_sort.cpp /archetype/heap_sort.h
#	/archetype/id_table.cpp /archetype/id_table.h
#	/archetype/interpreter.cpp /archetype/interpreter.h
#	/archetype/keywords.cpp /archetype/keywords.h
#	/archetype/linked_list.cpp /archetype/linked_list.h
#	/archetype/misc.cpp /archetype/misc.h
#	/archetype/parser.cpp /archetype/parser.h
#	/archetype/saveload.cpp /archetype/saveload.h
#	/archetype/semantic.cpp /archetype/semantic.h
#	/archetype/statement.h
#	/archetype/string.cpp /archetype/string.h
#	/archetype/sys_object.cpp /archetype/sys_object.h
#	/archetype/timestamp.cpp /archetype/timestamp.h
#	/archetype/token.cpp /archetype/token.h
#	/comprehend/charset.cpp /comprehend/charset.h
#	/comprehend/comprehend.cpp /comprehend/comprehend.h
#	/comprehend/debugger.cpp /comprehend/debugger.h
#	/comprehend/debugger_dumper.cpp /comprehend/debugger_dumper.h
#	/comprehend/detection.cpp /comprehend/detection.h
#	/comprehend/detection_tables.h
#	/comprehend/dictionary.cpp /comprehend/dictionary.h
#	/comprehend/draw_surface.cpp /comprehend/draw_surface.h
#	/comprehend/file_buf.cpp /comprehend/file_buf.h
#	/comprehend/game.cpp /comprehend/game.h
#	/comprehend/game_cc.cpp /comprehend/game_cc.h
#	/comprehend/game_data.cpp /comprehend/game_data.h
#	/comprehend/game_oo.cpp /comprehend/game_oo.h
#	/comprehend/game_opcodes.cpp /comprehend/game_opcodes.h
#	/comprehend/game_tm.cpp /comprehend/game_tm.h
#	/comprehend/game_tr1.cpp /comprehend/game_tr1.h
#	/comprehend/game_tr2.cpp /comprehend/game_tr2.h
#	/comprehend/pics.cpp /comprehend/pics.h
#	/glulx/accel.cpp
#	/glulx/detection.cpp /glulx/detection.h
#	/glulx/detection_tables.h
#	/glulx/exec.cpp
#	/glulx/float.cpp /glulx/float.h
#	/glulx/funcs.cpp
#	/glulx/gestalt.cpp
#	/glulx/glkop.cpp
#	/glulx/glulx.cpp /glulx/glulx.h
#	/glulx/glulx_types.h
#	/glulx/heap.cpp
#	/glulx/operand.cpp
#	/glulx/search.cpp
#	/glulx/serial.cpp
#	/glulx/string.cpp
#	/glulx/vm.cpp
#	/hugo/detection.cpp /hugo/detection.h
#	/hugo/detection_tables.h
#	/hugo/heexpr.cpp
#	/hugo/heglk.cpp
#	/hugo/hemedia.cpp
#	/hugo/hemisc.cpp
#	/hugo/heobject.cpp
#	/hugo/heparse.cpp
#	/hugo/heres.cpp
#	/hugo/herun.cpp
#	/hugo/heset.cpp
#	/hugo/htokens.cpp /hugo/htokens.h
#	/hugo/hugo.cpp /hugo/hugo.h
#	/hugo/hugo_defines.h
#	/hugo/hugo_types.h
#	/hugo/resource_archive.cpp /hugo/resource_archive.h
#	/hugo/stringfn.cpp /hugo/stringfn.h
#	/jacl/constants.h
#	/jacl/csv.h
#	/jacl/detection.cpp /jacl/detection.h
#	/jacl/detection_tables.h
#	/jacl/display.cpp
#	/jacl/encapsulate.cpp
#	/jacl/errors.cpp
#	/jacl/findroute.cpp
#	/jacl/glk_saver.cpp
#	/jacl/interpreter.cpp
#	/jacl/jacl.cpp /jacl/jacl.h
#	/jacl/jacl_main.cpp
#	/jacl/jpp.cpp
#	/jacl/language.h
#	/jacl/libcsv.cpp
#	/jacl/loader.cpp
#	/jacl/logging.cpp
#	/jacl/parser.cpp
#	/jacl/prototypes.h
#	/jacl/resolvers.cpp
#	/jacl/types.h
#	/jacl/utils.cpp
#	/jacl/version.h
#	/level9/bitmap.cpp
#	/level9/detection.cpp /level9/detection.h
#	/level9/detection_tables.h
#	/level9/level9.cpp /level9/level9.h
#	/level9/level9_main.cpp /level9/level9_main.h
#	/level9/os_glk.cpp /level9/os_glk.h
#	/magnetic/detection.cpp /magnetic/detection.h
#	/magnetic/detection_tables.h
#	/magnetic/emu.cpp
#	/magnetic/glk.cpp
#	/magnetic/magnetic.cpp /magnetic/magnetic.h
#	/magnetic/magnetic_defs.h
#	/magnetic/magnetic_types.h
#	/quest/detection.cpp /quest/detection.h
#	/quest/detection_tables.h
#	/quest/geas_file.cpp /quest/geas_file.h
#	/quest/geas_glk.cpp /quest/geas_glk.h
#	/quest/geas_impl.h
#	/quest/geas_runner.cpp /quest/geas_runner.h
#	/quest/geas_state.cpp /quest/geas_state.h
#	/quest/geas_util.cpp /quest/geas_util.h
#	/quest/limit_stack.h
#	/quest/quest.cpp /quest/quest.h
#	/quest/read_file.cpp /quest/read_file.h
#	/quest/reserved_words.h
#	/quest/streams.cpp /quest/streams.h
#	/quest/string.cpp /quest/string.h
#	/scott/detection.cpp /scott/detection.h
#	/scott/detection_tables.h
#	/scott/scott.cpp /scott/scott.h
#	/tads/tads2/appctx.h
#	/tads/tads2/built_in.cpp /tads/tads2/built_in.h
#	/tads/tads2/character_map.cpp /tads/tads2/character_map.h
#	/tads/tads2/command_line.cpp /tads/tads2/command_line.h
#	/tads/tads2/data.cpp /tads/tads2/data.h
#	/tads/tads2/debug.cpp /tads/tads2/debug.h
#	/tads/tads2/error.cpp /tads/tads2/error.h
#	/tads/tads2/error_handling.cpp /tads/tads2/error_handling.h
#	/tads/tads2/error_message.cpp
#	/tads/tads2/execute_command.cpp
#	/tads/tads2/file_io.cpp /tads/tads2/file_io.h
#	/tads/tads2/get_string.cpp
#	/tads/tads2/lib.h
#	/tads/tads2/line_source.h
#	/tads/tads2/line_source_file.cpp /tads/tads2/line_source_file.h
#	/tads/tads2/list.cpp /tads/tads2/list.h
#	/tads/tads2/ltk.cpp /tads/tads2/ltk.h
#	/tads/tads2/memory_cache.cpp /tads/tads2/memory_cache.h
#	/tads/tads2/memory_cache_heap.cpp /tads/tads2/memory_cache_heap.h
#	/tads/tads2/memory_cache_loader.h
#	/tads/tads2/memory_cache_swap.cpp /tads/tads2/memory_cache_swap.h
#	/tads/tads2/object.cpp /tads/tads2/object.h
#	/tads/tads2/opcode.h
#	/tads/tads2/os.cpp /tads/tads2/os.h
#	/tads/tads2/output.cpp
#	/tads/tads2/play.cpp /tads/tads2/play.h
#	/tads/tads2/post_compilation.cpp /tads/tads2/post_compilation.h
#	/tads/tads2/property.h
#	/tads/tads2/qa_scriptor.cpp
#	/tads/tads2/regex.cpp /tads/tads2/regex.h
#	/tads/tads2/run.cpp /tads/tads2/run.h
#	/tads/tads2/runstat.cpp
#	/tads/tads2/runtime_app.cpp /tads/tads2/runtime_app.h
#	/tads/tads2/runtime_driver.cpp
#	/tads/tads2/string_resources.h
#	/tads/tads2/tads2.cpp /tads/tads2/tads2.h
#	/tads/tads2/text_io.h
#	/tads/tads2/tokenizer.cpp /tads/tads2/tokenizer.h
#	/tads/tads2/tokenizer_hash.cpp
#	/tads/tads2/vocabulary.cpp /tads/tads2/vocabulary.h
#	/tads/tads2/vocabulary_parser.cpp
#	/tads/tads3/tads3.cpp /tads/tads3/tads3.h
#	/tads/detection.cpp /tads/detection.h
#	/tads/detection_tables.h
#	/tads/os_banners.cpp /tads/os_banners.h
#	/tads/os_buffer.cpp /tads/os_buffer.h
#	/tads/os_filetype.h
#	/tads/os_frob_tads.cpp /tads/os_frob_tads.h
#	/tads/os_glk.cpp /tads/os_glk.h
#	/tads/os_parse.cpp /tads/os_parse.h
#	/tads/tads.cpp /tads/tads.h
#	/zcode/bitmap_font.cpp /zcode/bitmap_font.h
#	/zcode/config.cpp /zcode/config.h
#	/zcode/detection.cpp /zcode/detection.h
#	/zcode/detection_tables.h
#	/zcode/frotz_types.h
#	/zcode/glk_interface.cpp /zcode/glk_interface.h
#	/zcode/mem.cpp /zcode/mem.h
#	/zcode/pics.cpp /zcode/pics.h
#	/zcode/pics_decoder.cpp /zcode/pics_decoder.h
#	/zcode/processor.cpp /zcode/processor.h
#	/zcode/processor_buffer.cpp
#	/zcode/processor_input.cpp
#	/zcode/processor_maths.cpp
#	/zcode/processor_mem.cpp
#	/zcode/processor_objects.cpp
#	/zcode/processor_screen.cpp
#	/zcode/processor_streams.cpp
#	/zcode/processor_table.cpp
#	/zcode/processor_text.cpp
#	/zcode/processor_variables.cpp
#	/zcode/processor_windows.cpp
#	/zcode/quetzal.cpp /zcode/quetzal.h
#	/zcode/screen.cpp /zcode/screen.h
#	/zcode/sound_folder.cpp /zcode/sound_folder.h
#	/zcode/windows.cpp /zcode/windows.h
#	/zcode/zcode.cpp /zcode/zcode.h
#	/blorb.cpp /blorb.h
#	/conf.cpp /conf.h
#	/debugger.cpp /debugger.h
#	/detection.cpp /detection.h
#	/events.cpp /events.h
#	/fonts.cpp /fonts.h
#	/game_description.h
#	/glk.cpp /glk.h
#	/glk_api.cpp /glk_api.h
#	/glk_dispa.cpp
#	/glk_types.h
#	/jumps.h
#	/metaengine.cpp
#	/module.mk
#	/pc_speaker.cpp /pc_speaker.h
#	/picture.cpp /picture.h
#	/quetzal.cpp /quetzal.h
#	/raw_decoder.cpp /raw_decoder.h
#	/screen.cpp /screen.h
#	/selection.cpp /selection.h
#	/sound.cpp /sound.h
#	/speech.cpp /speech.h
#	/streams.cpp /streams.h
#	/time.cpp /time.h
#	/unicode.cpp /unicode.h
#	/unicode_gen.cpp /unicode_gen.h
#	/utils.cpp /utils.h
#	/window_graphics.cpp /window_graphics.h
#	/window_pair.cpp /window_pair.h
#	/window_text_buffer.cpp /window_text_buffer.h
#	/window_text_grid.cpp /window_text_grid.h
#	/windows.cpp /windows.h
#)
#
#add_engine(gnap
#	/scenes/arcade.cpp /scenes/arcade.h
#	/scenes/group0.cpp /scenes/group0.h
#	/scenes/group1.cpp /scenes/group1.h
#	/scenes/group2.cpp /scenes/group2.h
#	/scenes/group3.cpp /scenes/group3.h
#	/scenes/group4.cpp /scenes/group4.h
#	/scenes/group5.cpp /scenes/group5.h
#	/scenes/groupcs.cpp /scenes/groupcs.h
#	/scenes/intro.cpp /scenes/intro.h
#	/scenes/scenecore.cpp /scenes/scenecore.h
#	/character.cpp /character.h
#	/datarchive.cpp /datarchive.h
#	/debugger.cpp /debugger.h
#	/fontdata.h
#	/gamesys.cpp /gamesys.h
#	/gnap.cpp /gnap.h
#	/grid.cpp
#	/menu.cpp
#	/metaengine.cpp
#	/module.mk
#	/music.cpp /music.h
#	/resource.cpp /resource.h
#	/sound.cpp /sound.h
#)
#
#add_engine(gob
#	/demos/batplayer.cpp /demos/batplayer.h
#	/demos/demoplayer.cpp /demos/demoplayer.h
#	/demos/scnplayer.cpp /demos/scnplayer.h
#	/minigames/geisha/diving.cpp /minigames/geisha/diving.h
#	/minigames/geisha/evilfish.cpp /minigames/geisha/evilfish.h
#	/minigames/geisha/meter.cpp /minigames/geisha/meter.h
#	/minigames/geisha/mouth.cpp /minigames/geisha/mouth.h
#	/minigames/geisha/oko.cpp /minigames/geisha/oko.h
#	/minigames/geisha/penetration.cpp /minigames/geisha/penetration.h
#	/minigames/geisha/submarine.cpp /minigames/geisha/submarine.h
#
#	/pregob/onceupon/abracadabra.cpp /pregob/onceupon/abracadabra.h
#	/pregob/onceupon/babayaga.cpp /pregob/onceupon/babayaga.h
#	/pregob/onceupon/brokenstrings.h
#	/pregob/onceupon/chargenchild.cpp /pregob/onceupon/chargenchild.h
#	/pregob/onceupon/onceupon.cpp /pregob/onceupon/onceupon.h
#	/pregob/onceupon/palettes.h
#	/pregob/onceupon/parents.cpp /pregob/onceupon/parents.h
#	/pregob/onceupon/stork.cpp /pregob/onceupon/stork.h
#	/pregob/onceupon/title.cpp /pregob/onceupon/title.h
#	/pregob/gctfile.cpp /pregob/gctfile.h
#	/pregob/pregob.cpp /pregob/pregob.h
#	/pregob/seqfile.cpp /pregob/seqfile.h
#	/pregob/txtfile.cpp /pregob/txtfile.h
#	/save/saveconverter.cpp /save/saveconverter.h
#	/save/saveconverter_v2.cpp
#	/save/saveconverter_v3.cpp
#	/save/saveconverter_v4.cpp
#	/save/savefile.cpp /save/savefile.h
#	/save/savehandler.cpp /save/savehandler.h
#	/save/saveload.cpp /save/saveload.h
#	/save/saveload_ajworld.cpp
#	/save/saveload_fascin.cpp
#	/save/saveload_geisha.cpp
#	/save/saveload_inca2.cpp
#	/save/saveload_playtoons.cpp
#	/save/saveload_v2.cpp
#	/save/saveload_v3.cpp
#	/save/saveload_v4.cpp
#	/save/saveload_v6.cpp
#	/save/saveload_v7.cpp
#	/sound/adlib.cpp /sound/adlib.h
#	/sound/adlplayer.cpp /sound/adlplayer.h
#	/sound/bgatmosphere.cpp /sound/bgatmosphere.h
#	/sound/cdrom.cpp /sound/cdrom.h
#	/sound/infogrames.cpp /sound/infogrames.h
#	/sound/musplayer.cpp /sound/musplayer.h
#	/sound/pcspeaker.cpp /sound/pcspeaker.h
#	/sound/protracker.cpp /sound/protracker.h
#	/sound/sound.cpp /sound/sound.h
#	/sound/soundblaster.cpp /sound/soundblaster.h
#	/sound/sounddesc.cpp /sound/sounddesc.h
#	/sound/soundmixer.cpp /sound/soundmixer.h
#	/anifile.cpp /anifile.h
#	/aniobject.cpp /aniobject.h
#	/backbuffer.cpp /backbuffer.h
#	/cheater.cpp /cheater.h
#	/cheater_geisha.cpp
#	/cmpfile.cpp /cmpfile.h
#	/console.cpp /console.h
#	/databases.cpp /databases.h
#	/dataio.cpp /dataio.h
#	/dbase.cpp /dbase.h
#	/decfile.cpp /decfile.h
#	/draw.cpp /draw.h
#	/draw_bargon.cpp
#	/draw_fascin.cpp
#	/draw_playtoons.cpp
#	/draw_v1.cpp
#	/draw_v2.cpp
#	/expression.cpp /expression.h
#	/game.cpp /game.h
#	/global.cpp /global.h
#	/gob.cpp /gob.h
#	/goblin.cpp /goblin.h
#	/goblin_v1.cpp
#	/goblin_v2.cpp
#	/goblin_v3.cpp
#	/goblin_v4.cpp
#	/hotspots.cpp /hotspots.h
#	/iniconfig.cpp /iniconfig.h
#	/init.cpp /init.h
#	/init_fascin.cpp
#	/init_geisha.cpp
#	/init_v1.cpp
#	/init_v2.cpp
#	/init_v3.cpp
#	/init_v4.cpp
#	/init_v6.cpp
#	/init_v7.cpp
#	/inter.cpp /inter.h
#	/inter_bargon.cpp
#	/inter_fascin.cpp
#	/inter_geisha.cpp
#	/inter_inca2.cpp
#	/inter_littlered.cpp
#	/inter_playtoons.cpp
#	/inter_v1.cpp
#	/inter_v2.cpp
#	/inter_v3.cpp
#	/inter_v4.cpp
#	/inter_v5.cpp
#	/inter_v6.cpp
#	/inter_v7.cpp
#	/map.cpp /map.h
#	/map_v1.cpp
#	/map_v2.cpp
#	/metaengine.cpp
#	/module.mk
#	/mult.cpp /mult.h
#	/mult_v1.cpp
#	/mult_v2.cpp
#	/palanim.cpp /palanim.h
#	/resources.cpp /resources.h
#	/rxyfile.cpp /rxyfile.h
#	/scenery.cpp /scenery.h
#	/scenery_v1.cpp
#	/scenery_v2.cpp
#	/script.cpp /script.h
#	/surface.cpp /surface.h
#	/totfile.cpp /totfile.h
#	/util.cpp /util.h
#	/variables.cpp /variables.h
#	/video.cpp /video.h
#	/video_v1.cpp
#	/video_v2.cpp
#	/video_v6.cpp
#	/videoplayer.cpp /videoplayer.h
#)
#
#add_engine(griffon
#	/combat.cpp
#	/console.cpp /console.h
#	/cutscenes.cpp
#	/dialogs.cpp
#	/draw.cpp
#	/engine.cpp
#	/gfx.cpp
#	/griffon.cpp /griffon.h
#	/input.cpp
#	/logic.cpp
#	/metaengine.cpp
#	/module.mk
#	/resources.cpp
#	/saveload.cpp
#	/sound.cpp
#)
#
#add_engine(grim
#	/costume/anim_component.cpp /costume/anim_component.h
#	/costume/bitmap_component.cpp /costume/bitmap_component.h
#	/costume/chore.cpp /costume/chore.h
#	/costume/colormap_component.cpp /costume/colormap_component.h
#	/costume/component.cpp /costume/component.h
#	/costume/head.cpp /costume/head.h
#	/costume/keyframe_component.cpp /costume/keyframe_component.h
#	/costume/lua_var_component.cpp /costume/lua_var_component.h
#	/costume/main_model_component.cpp /costume/main_model_component.h
#	/costume/material_component.cpp /costume/material_component.h
#	/costume/mesh_component.cpp /costume/mesh_component.h
#	/costume/model_component.cpp /costume/model_component.h
#	/costume/sound_component.cpp /costume/sound_component.h
#	/costume/sprite_component.cpp /costume/sprite_component.h
#	/emi/costume/emianim_component.cpp /emi/costume/emianim_component.h
#	/emi/costume/emichore.cpp /emi/costume/emichore.h
#	/emi/costume/emihead.cpp /emi/costume/emihead.h
#	/emi/costume/emiluacode_component.cpp /emi/costume/emiluacode_component.h
#	/emi/costume/emiluavar_component.cpp /emi/costume/emiluavar_component.h
#	/emi/costume/emimesh_component.cpp /emi/costume/emimesh_component.h
#	/emi/costume/emiskel_component.cpp /emi/costume/emiskel_component.h
#	/emi/costume/emisprite_component.cpp /emi/costume/emisprite_component.h
#	/emi/costume/emitexi_component.cpp /emi/costume/emitexi_component.h
#	/emi/sound/codecs/scx.cpp /emi/sound/codecs/scx.h
#	/emi/sound/aifftrack.cpp /emi/sound/aifftrack.h
#	/emi/sound/emisound.cpp /emi/sound/emisound.h
#	/emi/sound/mp3track.cpp /emi/sound/mp3track.h
#	/emi/sound/scxtrack.cpp /emi/sound/scxtrack.h
#	/emi/sound/track.cpp /emi/sound/track.h
#	/emi/sound/vimatrack.cpp /emi/sound/vimatrack.h
#	/emi/animationemi.cpp /emi/animationemi.h
#	/emi/costumeemi.cpp /emi/costumeemi.h
#	/emi/emi.cpp /emi/emi.h
#	/emi/emi_registry.cpp /emi/emi_registry.h
#	/emi/layer.cpp /emi/layer.h
#	/emi/lua_v2.cpp /emi/lua_v2.h
#	/emi/lua_v2_actor.cpp
#	/emi/lua_v2_sound.cpp
#	/emi/modelemi.cpp /emi/modelemi.h
#	/emi/poolsound.cpp /emi/poolsound.h
#	/emi/skeleton.cpp /emi/skeleton.h
#	/imuse/imuse.cpp /imuse/imuse.h
#	/imuse/imuse_mcmp_mgr.cpp /imuse/imuse_mcmp_mgr.h
#	/imuse/imuse_music.cpp
#	/imuse/imuse_script.cpp
#	/imuse/imuse_sndmgr.cpp /imuse/imuse_sndmgr.h
#	/imuse/imuse_tables.cpp /imuse/imuse_tables.h
#	/imuse/imuse_track.cpp /imuse/imuse_track.h
#	/lua/lapi.cpp /lua/lapi.h
#	/lua/lauxlib.cpp /lua/lauxlib.h
#	/lua/lbuffer.cpp
#	/lua/lbuiltin.cpp /lua/lbuiltin.h
#	/lua/ldo.cpp /lua/ldo.h
#	/lua/lfunc.cpp /lua/lfunc.h
#	/lua/lgc.cpp /lua/lgc.h
#	/lua/liolib.cpp
#	/lua/llex.cpp /lua/llex.h
#	/lua/lmathlib.cpp
#	/lua/lmem.cpp /lua/lmem.h
#	/lua/lobject.cpp /lua/lobject.h
#	/lua/lopcodes.h
#	/lua/lparser.h
#	/lua/lrestore.cpp
#	/lua/lsave.cpp
#	/lua/lstate.cpp /lua/lstate.h
#	/lua/lstring.cpp /lua/lstring.h
#	/lua/lstrlib.cpp
#	/lua/lstx.cpp /lua/lstx.h
#	/lua/ltable.cpp /lua/ltable.h
#	/lua/ltask.cpp /lua/ltask.h
#	/lua/ltm.cpp /lua/ltm.h
#	/lua/lua.h
#	/lua/luadebug.h
#	/lua/lualib.h
#	/lua/lundump.cpp /lua/lundump.h
#	/lua/lvm.cpp /lua/lvm.h
#	/lua/lzio.cpp /lua/lzio.h
#	/movie/codecs/blocky16.cpp /movie/codecs/blocky16.h
#	/movie/codecs/blocky8.cpp /movie/codecs/blocky8.h
#	/movie/codecs/codec48.cpp /movie/codecs/codec48.h
#	/movie/codecs/smush_decoder.cpp /movie/codecs/smush_decoder.h
#	/movie/codecs/vima.cpp /movie/codecs/vima.h
#	/movie/bink.cpp /movie/bink.h
#	/movie/movie.cpp /movie/movie.h
#	/movie/mpeg.cpp /movie/mpeg.h
#	/movie/quicktime.cpp /movie/quicktime.h
#	/movie/smush.cpp /movie/smush.h
#	/remastered/commentary.cpp /remastered/commentary.h
#	/remastered/hotspot.cpp /remastered/hotspot.h
#	/remastered/lua_remastered.cpp /remastered/lua_remastered.h
#	/remastered/overlay.cpp /remastered/overlay.h
#	/update/lang_filter.cpp /update/lang_filter.h
#	/update/mscab.cpp /update/mscab.h
#	/update/packfile.cpp /update/packfile.h
#	/update/update.cpp /update/update.h
#	/actor.cpp /actor.h
#	/animation.cpp /animation.h
#	/bitmap.cpp /bitmap.h
#	/color.cpp /color.h
#	/colormap.cpp /colormap.h
#	/costume.cpp /costume.h
#	/debug.cpp /debug.h
#	/debugger.cpp /debugger.h
#	/detection.h
#	/font.cpp /font.h
#	/gfx_base.cpp /gfx_base.h
#	/gfx_opengl.cpp /gfx_opengl.h
#	/gfx_opengl_shaders.cpp /gfx_opengl_shaders.h
#	/gfx_tinygl.cpp /gfx_tinygl.h
#	/grim.cpp /grim.h
#	/grim_controls.cpp
#	/inputdialog.cpp /inputdialog.h
#	/iris.cpp /iris.h
#	/keyframe.cpp /keyframe.h
#	/lab.cpp /lab.h
#	/lipsync.cpp /lipsync.h
#	/localize.cpp /localize.h
#	/lua.cpp /lua.h
#	/lua_v1.cpp /lua_v1.h
#	/lua_v1_actor.cpp
#	/lua_v1_graphics.cpp
#	/lua_v1_set.cpp
#	/lua_v1_sound.cpp
#	/lua_v1_text.cpp
#	/material.cpp /material.h
#	/md5check.cpp /md5check.h
#	/md5checkdialog.cpp /md5checkdialog.h
#	/metaengine.cpp
#	/model.cpp /model.h
#	/module.mk
#	/object.cpp /object.h
#	/objectstate.cpp /objectstate.h
#	/patchr.cpp /patchr.h
#	/pool.h
#	/primitives.cpp /primitives.h
#	/registry.cpp /registry.h
#	/resource.cpp /resource.h
#	/savegame.cpp /savegame.h
#	/sector.cpp /sector.h
#	/set.cpp /set.h
#	/sound.cpp /sound.h
#	/sprite.cpp /sprite.h
#	/textobject.cpp /textobject.h
#	/textsplit.cpp /textsplit.h
#)
#
#add_engine(groovie
#	/logic/beehive.cpp /logic/beehive.h
#	/logic/cake.cpp /logic/cake.h
#	/logic/cell.cpp /logic/cell.h
#	/logic/gallery.cpp /logic/gallery.h
#	/logic/mousetrap.cpp /logic/mousetrap.h
#	/logic/othello.cpp /logic/othello.h
#	/logic/pente.cpp /logic/pente.h
#	/logic/tlcgame.cpp /logic/tlcgame.h
#	/logic/triangle.cpp /logic/triangle.h
#	/logic/winerack.cpp /logic/winerack.h
#	/video/player.cpp /video/player.h
#	/video/roq.cpp /video/roq.h
#	/video/vdx.cpp /video/vdx.h
#	/cursor.cpp /cursor.h
#	/debug.cpp /debug.h
#	/detection.h
#	/font.cpp /font.h
#	/graphics.cpp /graphics.h
#	/groovie.cpp /groovie.h
#	/lzss.cpp /lzss.h
#	/metaengine.cpp
#	/module.mk
#	/music.cpp /music.h
#	/resource.cpp /resource.h
#	/saveload.cpp /saveload.h
#	/script.cpp /script.h
#)
#
#add_engine(hadesch
#	/rooms/monster/cyclops.cpp
#	/rooms/monster/illusion.cpp
#	/rooms/monster/projectile.cpp
#	/rooms/monster/typhoon.cpp
#	/rooms/argo.cpp
#	/rooms/athena.cpp
#	/rooms/catacombs.cpp
#	/rooms/credits.cpp
#	/rooms/crete.cpp
#	/rooms/daedalus.cpp
#	/rooms/ferry.cpp
#	/rooms/hadesthrone.cpp
#	/rooms/intro.cpp
#	/rooms/medisle.cpp
#	/rooms/medusa.cpp
#	/rooms/minos.cpp
#	/rooms/minotaur.cpp
#	/rooms/monster.cpp /rooms/monster.h
#	/rooms/olympus.cpp
#	/rooms/options.cpp
#	/rooms/priam.cpp
#	/rooms/quiz.cpp
#	/rooms/riverstyx.cpp
#	/rooms/seriphos.cpp
#	/rooms/trojan.cpp
#	/rooms/troy.cpp
#	/rooms/volcano.cpp
#	/rooms/walloffame.cpp
#	/ambient.cpp /ambient.h
#	/baptr.cpp /baptr.h
#	/detection_tables.h
#	/enums.h
#	/event.h
#	/gfx_context.cpp /gfx_context.h
#	/hadesch.cpp /hadesch.h
#	/herobelt.cpp /herobelt.h
#	/hotzone.cpp /hotzone.h
#	/metaengine.cpp
#	/module.mk
#	/persistent.cpp /persistent.h
#	/pod_file.cpp /pod_file.h
#	/pod_image.cpp /pod_image.h
#	/table.cpp /table.h
#	/tag_file.cpp /tag_file.h
#	/video.cpp /video.h
#)
#
#add_engine(hdb
#	/ai-bots.cpp
#	/ai-cinematic.cpp
#	/ai-funcs.cpp
#	/ai-init.cpp
#	/ai-inventory.cpp
#	/ai-lists.cpp
#	/ai-player.cpp /ai-player.h
#	/ai-use.cpp
#	/ai-waypoint.cpp
#	/ai.h
#	/detection.h
#	/file-manager.cpp /file-manager.h
#	/gfx.cpp /gfx.h
#	/hdb.cpp /hdb.h
#	/input.cpp /input.h
#	/lua-script.cpp /lua-script.h
#	/map.cpp /map.h
#	/menu.cpp /menu.h
#	/metaengine.cpp
#	/module.mk
#	/mpc.h
#	/saveload.cpp
#	/sound.cpp /sound.h
#	/window.cpp /window.h
#)
#
#add_engine(hopkins
#	/anim.cpp /anim.h
#	/computer.cpp /computer.h
#	/debugger.cpp /debugger.h
#	/detection.h
#	/detection_tables.h
#	/dialogs.cpp /dialogs.h
#	/events.cpp /events.h
#	/files.cpp /files.h
#	/font.cpp /font.h
#	/globals.cpp /globals.h
#	/graphics.cpp /graphics.h
#	/hopkins.cpp /hopkins.h
#	/lines.cpp /lines.h
#	/menu.cpp /menu.h
#	/metaengine.cpp
#	/module.mk
#	/objects.cpp /objects.h
#	/saveload.cpp /saveload.h
#	/script.cpp /script.h
#	/sound.cpp /sound.h
#	/talk.cpp /talk.h
#)
#
#add_engine(hugo
#	/console.cpp /console.h
#	/detection.h
#	/dialogs.cpp /dialogs.h
#	/display.cpp /display.h
#	/file.cpp /file.h
#	/file_v1d.cpp
#	/file_v1w.cpp
#	/file_v2d.cpp
#	/file_v2w.cpp
#	/file_v3d.cpp
#	/game.h
#	/hugo.cpp /hugo.h
#	/intro.cpp /intro.h
#	/inventory.cpp /inventory.h
#	/metaengine.cpp
#	/module.mk
#	/mouse.cpp /mouse.h
#	/object.cpp /object.h
#	/object_v1d.cpp
#	/object_v1w.cpp
#	/object_v2d.cpp
#	/object_v3d.cpp
#	/parser.cpp /parser.h
#	/parser_v1d.cpp
#	/parser_v1w.cpp
#	/parser_v2d.cpp
#	/parser_v3d.cpp
#	/route.cpp /route.h
#	/schedule.cpp /schedule.h
#	/sound.cpp /sound.h
#	/text.cpp /text.h
#	/util.cpp /util.h
#)
#
#add_engine(hypno
#	/boyz/boyz.cpp
#	/spider/arcade.cpp
#	/spider/hard.cpp
#	/spider/spider.cpp
#	/spider/talk.cpp
#	/wet/arcade.cpp
#	/wet/hard.cpp
#	/wet/wet.cpp
#	/actions.cpp
#	/arcade.cpp
#	/cursors.cpp
#	/grammar.h
#	/grammar_arc.cpp
#	/grammar_mis.cpp
#	/hypno.cpp /hypno.h
#	/lexer_arc.cpp
#	/lexer_mis.cpp
#	/libfile.cpp /libfile.h
#	/metaengine.cpp
#	/module.mk
#	/scene.cpp
#	/tokens_arc.h
#	/tokens_mis.h
#	/video.cpp
#)
#
#add_engine(illusions
#	/bbdou/bbdou_bubble.cpp /bbdou/bbdou_bubble.h
#	/bbdou/bbdou_credits.cpp /bbdou/bbdou_credits.h
#	/bbdou/bbdou_credits_staticdata.cpp
#	/bbdou/bbdou_cursor.cpp /bbdou/bbdou_cursor.h
#	/bbdou/bbdou_foodctl.cpp /bbdou/bbdou_foodctl.h
#	/bbdou/bbdou_inventory.cpp /bbdou/bbdou_inventory.h
#	/bbdou/bbdou_menukeys.cpp /bbdou/bbdou_menukeys.h
#	/bbdou/bbdou_specialcode.cpp /bbdou/bbdou_specialcode.h
#	/bbdou/bbdou_triggerfunctions.cpp /bbdou/bbdou_triggerfunctions.h
#	/bbdou/bbdou_videoplayer.cpp /bbdou/bbdou_videoplayer.h
#	/bbdou/gamestate_bbdou.cpp /bbdou/gamestate_bbdou.h
#	/bbdou/illusions_bbdou.cpp /bbdou/illusions_bbdou.h
#	/bbdou/menusystem_bbdou.cpp /bbdou/menusystem_bbdou.h
#	/bbdou/scriptopcodes_bbdou.cpp /bbdou/scriptopcodes_bbdou.h
#	/duckman/duckman_credits.cpp /duckman/duckman_credits.h
#	/duckman/duckman_dialog.cpp /duckman/duckman_dialog.h
#	/duckman/duckman_inventory.cpp /duckman/duckman_inventory.h
#	/duckman/duckman_screenshakereffects.cpp /duckman/duckman_screenshakereffects.h
#	/duckman/duckman_specialcode.cpp /duckman/duckman_specialcode.h
#	/duckman/duckman_videoplayer.cpp /duckman/duckman_videoplayer.h
#	/duckman/gamestate_duckman.cpp /duckman/gamestate_duckman.h
#	/duckman/illusions_duckman.cpp /duckman/illusions_duckman.h
#	/duckman/menusystem_duckman.cpp /duckman/menusystem_duckman.h
#	/duckman/propertytimers.cpp /duckman/propertytimers.h
#	/duckman/scriptopcodes_duckman.cpp /duckman/scriptopcodes_duckman.h
#	/resources/actorresource.cpp /resources/actorresource.h
#	/resources/backgroundresource.cpp /resources/backgroundresource.h
#	/resources/fontresource.cpp /resources/fontresource.h
#	/resources/genericresource.cpp /resources/genericresource.h
#	/resources/midiresource.cpp /resources/midiresource.h
#	/resources/scriptresource.cpp /resources/scriptresource.h
#	/resources/soundresource.cpp /resources/soundresource.h
#	/resources/talkresource.cpp /resources/talkresource.h
#	/threads/abortablethread.cpp /threads/abortablethread.h
#	/threads/causethread_duckman.cpp /threads/causethread_duckman.h
#	/threads/scriptthread.cpp /threads/scriptthread.h
#	/threads/talkthread.cpp /threads/talkthread.h
#	/threads/talkthread_duckman.cpp /threads/talkthread_duckman.h
#	/threads/timerthread.cpp /threads/timerthread.h
#	/actor.cpp /actor.h
#	/camera.cpp /camera.h
#	/cursor.cpp /cursor.h
#	/detection.h
#	/dictionary.cpp /dictionary.h
#	/fileresourcereader.cpp /fileresourcereader.h
#	/fixedpoint.cpp /fixedpoint.h
#	/gamarchive.cpp /gamarchive.h
#	/gamestate.cpp /gamestate.h
#	/gamresourcereader.cpp /gamresourcereader.h
#	/graphics.cpp /graphics.h
#	/illusions.cpp /illusions.h
#	/input.cpp /input.h
#	/menusystem.cpp /menusystem.h
#	/metaengine.cpp
#	/module.mk
#	/pathfinder.cpp /pathfinder.h
#	/resourcereader.h
#	/resourcesystem.cpp /resourcesystem.h
#	/saveload.cpp
#	/screen.cpp /screen.h
#	/screentext.cpp /screentext.h
#	/scriptopcodes.cpp /scriptopcodes.h
#	/scriptstack.cpp /scriptstack.h
#	/sequenceopcodes.cpp /sequenceopcodes.h
#	/sound.cpp /sound.h
#	/specialcode.cpp /specialcode.h
#	/textdrawer.cpp /textdrawer.h
#	/thread.cpp /thread.h
#	/time.cpp /time.h
#	/updatefunctions.cpp /updatefunctions.h
#)

add_engine(kyra
	/engine/chargen.cpp
	/engine/darkmoon.cpp /engine/darkmoon.h
	/engine/eob.cpp /engine/eob.h
	/engine/eobcommon.cpp /engine/eobcommon.h
	/engine/item.h
	/engine/items_eob.cpp
	/engine/items_hof.cpp
	/engine/items_lok.cpp
	/engine/items_lol.cpp
	/engine/items_mr.cpp
	/engine/items_v2.cpp
	/engine/kyra_hof.cpp /engine/kyra_hof.h
	/engine/kyra_lok.cpp /engine/kyra_lok.h
	/engine/kyra_mr.cpp /engine/kyra_mr.h
	/engine/kyra_rpg.cpp /engine/kyra_rpg.h
	/engine/kyra_v1.cpp
	/engine/kyra_v2.cpp /engine/kyra_v2.h
	/engine/lol.cpp /engine/lol.h
	/engine/magic_eob.cpp
	/engine/scene_eob.cpp
	/engine/scene_hof.cpp
	/engine/scene_lok.cpp
	/engine/scene_lol.cpp
	/engine/scene_mr.cpp
	/engine/scene_rpg.cpp
	/engine/scene_v1.cpp
	/engine/scene_v2.cpp
	/engine/sprites.cpp /engine/sprites.h
	/engine/sprites_eob.cpp
	/engine/sprites_lol.cpp
	/engine/sprites_rpg.cpp
	/engine/timer.cpp /engine/timer.h
	/engine/timer_eob.cpp
	/engine/timer_hof.cpp
	/engine/timer_lok.cpp
	/engine/timer_lol.cpp
	/engine/timer_mr.cpp
	/engine/timer_rpg.cpp
	/engine/util.cpp /engine/util.h
	/graphics/animator_hof.cpp
	/graphics/animator_lok.cpp /graphics/animator_lok.h
	/graphics/animator_mr.cpp
	/graphics/animator_tim.cpp
	/graphics/animator_v2.cpp
	/graphics/screen.cpp /graphics/screen.h
	/graphics/screen_eob.cpp /graphics/screen_eob.h
	/graphics/screen_eob_amiga.cpp
	/graphics/screen_eob_pc98.cpp
	/graphics/screen_eob_segacd.cpp /graphics/screen_eob_segacd.h
	/graphics/screen_eob_towns.cpp
	/graphics/screen_hof.cpp /graphics/screen_hof.h
	/graphics/screen_lok.cpp /graphics/screen_lok.h
	/graphics/screen_lol.cpp /graphics/screen_lol.h
	/graphics/screen_mr.cpp /graphics/screen_mr.h
	/graphics/screen_v2.cpp /graphics/screen_v2.h
	/graphics/vqa.cpp /graphics/vqa.h
	/graphics/wsamovie.cpp /graphics/wsamovie.h
	/gui/debugger.cpp /gui/debugger.h
	/gui/gui.cpp /gui/gui.h
	/gui/gui_eob.cpp /gui/gui_eob.h
	/gui/gui_eob_segacd.cpp /gui/gui_eob_segacd.h
	/gui/gui_hof.cpp /gui/gui_hof.h
	/gui/gui_lok.cpp /gui/gui_lok.h
	/gui/gui_lol.cpp /gui/gui_lol.h
	/gui/gui_mr.cpp /gui/gui_mr.h
	/gui/gui_rpg.cpp
	/gui/gui_v1.cpp /gui/gui_v1.h
	/gui/gui_v2.cpp /gui/gui_v2.h
	/gui/saveload.cpp
	/gui/saveload_eob.cpp
	/gui/saveload_hof.cpp
	/gui/saveload_lok.cpp
	/gui/saveload_lol.cpp
	/gui/saveload_mr.cpp
	/gui/saveload_rpg.cpp
	/resource/resource.cpp /resource/resource.h
	/resource/resource_intern.cpp /resource/resource_intern.h
	/resource/resource_segacd.cpp /resource/resource_segacd.h
	/resource/staticres.cpp
	/resource/staticres_eob.cpp
	/resource/staticres_lol.cpp
	/resource/staticres_rpg.cpp
	/script/script.cpp /script/script.h
	/script/script_eob.cpp /script/script_eob.h
	/script/script_hof.cpp
	/script/script_lok.cpp
	/script/script_lol.cpp
	/script/script_mr.cpp
	/script/script_tim.cpp /script/script_tim.h
	/script/script_v1.cpp
	/script/script_v2.cpp
	/sequence/seqplayer_eob_segacd.cpp /sequence/seqplayer_eob_segacd.h
	/sequence/seqplayer_lok.cpp /sequence/seqplayer_lok.h
	/sequence/sequences_darkmoon.cpp
	/sequence/sequences_eob.cpp
	/sequence/sequences_hof.cpp /sequence/sequences_hof.h
	/sequence/sequences_lok.cpp
	/sequence/sequences_lol.cpp
	/sequence/sequences_mr.cpp
	/sequence/sequences_v2.cpp
	/sound/drivers/adlib.cpp
	/sound/drivers/audiomaster2.cpp /sound/drivers/audiomaster2.h
	/sound/drivers/audstream.cpp
	/sound/drivers/halestorm.cpp /sound/drivers/halestorm.h
	/sound/drivers/mlalf98.cpp /sound/drivers/mlalf98.h
	/sound/drivers/pc_base.h
	/sound/drivers/pcspeaker_v1.cpp
	/sound/drivers/pcspeaker_v2.cpp
	/sound/drivers/segacd.cpp /sound/drivers/segacd.h
	/sound/sound.cpp /sound/sound.h
	/sound/sound_amiga_eob.cpp
	/sound/sound_amiga_lok.cpp
	/sound/sound_digital_mr.cpp /sound/sound_digital_mr.h
	/sound/sound_intern.h
	/sound/sound_lok.cpp
	/sound/sound_lol.cpp
	/sound/sound_mac_lok.cpp
	/sound/sound_mac_res.h
	/sound/sound_pc98_eob.cpp
	/sound/sound_pc98_lok.cpp
	/sound/sound_pc98_v2.cpp
	/sound/sound_pc_midi.cpp
	/sound/sound_pc_v1.cpp /sound/sound_pc_v1.h
	/sound/sound_segacd_eob.cpp
	/sound/sound_towns_darkmoon.cpp
	/sound/sound_towns_lok.cpp
	/text/text.cpp /text/text.h
	/text/text_eob_segacd.cpp /text/text_eob_segacd.h
	/text/text_hof.cpp /text/text_hof.h
	/text/text_lok.cpp
	/text/text_lol.cpp /text/text_lol.h
	/text/text_mr.cpp /text/text_mr.h
	/text/text_rpg.cpp /text/text_rpg.h
	/detection.h
	/detection_tables.h
	/kyra_v1.h
	/metaengine.cpp
	/module.mk
)
#
#add_engine(lab
#	/anim.cpp /anim.h
#	/console.cpp /console.h
#	/detection.h
#	/dispman.cpp /dispman.h
#	/engine.cpp
#	/eventman.cpp /eventman.h
#	/image.cpp /image.h
#	/interface.cpp /interface.h
#	/intro.cpp /intro.h
#	/lab.cpp /lab.h
#	/labsets.cpp /labsets.h
#	/map.cpp
#	/metaengine.cpp
#	/module.mk
#	/music.cpp /music.h
#	/processroom.cpp /processroom.h
#	/resource.cpp /resource.h
#	/savegame.cpp
#	/special.cpp
#	/speciallocks.cpp /speciallocks.h
#	/utils.cpp /utils.h
#)

add_engine(lure
	/animseq.cpp /animseq.h
	/debugger.cpp /debugger.h
	/decode.cpp /decode.h
	/detection.h
	/disk.cpp /disk.h
	/events.cpp /events.h
	/fights.cpp /fights.h
	/game.cpp /game.h
	/hotspots.cpp /hotspots.h
	/intro.cpp /intro.h
	/lure.cpp /lure.h
	/luredefs.h
	/memory.cpp /memory.h
	/menu.cpp /menu.h
	/metaengine.cpp
	/module.mk
	/palette.cpp /palette.h
	/res.cpp /res.h
	/res_struct.cpp /res_struct.h
	/room.cpp /room.h
	/screen.cpp /screen.h
	/scripts.cpp /scripts.h
	/sound.cpp /sound.h
	/strings.cpp /strings.h
	/surface.cpp /surface.h
)
#
#add_engine(made
#	/console.cpp /console.h
#	/database.cpp /database.h
#	/detection.h
#	/detection_tables.h
#	/graphics.cpp /graphics.h
#	/made.cpp /made.h
#	/metaengine.cpp
#	/module.mk
#	/music.cpp /music.h
#	/pmvplayer.cpp /pmvplayer.h
#	/redreader.cpp /redreader.h
#	/resource.cpp /resource.h
#	/screen.cpp /screen.h
#	/screenfx.cpp /screenfx.h
#	/script.cpp /script.h
#	/scriptfuncs.cpp /scriptfuncs.h
#	/sound.cpp /sound.h
#)
#
#add_engine(mads
#	/dragonsphere/dragonsphere_scenes.cpp /dragonsphere/dragonsphere_scenes.h
#	/dragonsphere/dragonsphere_scenes1.cpp /dragonsphere/dragonsphere_scenes1.h
#	/dragonsphere/game_dragonsphere.cpp /dragonsphere/game_dragonsphere.h
#	/dragonsphere/globals_dragonsphere.cpp /dragonsphere/globals_dragonsphere.h
#	/nebular/dialogs_nebular.cpp /nebular/dialogs_nebular.h
#	/nebular/game_nebular.cpp /nebular/game_nebular.h
#	/nebular/globals_nebular.cpp /nebular/globals_nebular.h
#	/nebular/menu_nebular.cpp /nebular/menu_nebular.h
#	/nebular/nebular_scenes.cpp /nebular/nebular_scenes.h
#	/nebular/nebular_scenes1.cpp /nebular/nebular_scenes1.h
#	/nebular/nebular_scenes2.cpp /nebular/nebular_scenes2.h
#	/nebular/nebular_scenes3.cpp /nebular/nebular_scenes3.h
#	/nebular/nebular_scenes4.cpp /nebular/nebular_scenes4.h
#	/nebular/nebular_scenes5.cpp /nebular/nebular_scenes5.h
#	/nebular/nebular_scenes6.cpp /nebular/nebular_scenes6.h
#	/nebular/nebular_scenes7.cpp /nebular/nebular_scenes7.h
#	/nebular/nebular_scenes8.cpp /nebular/nebular_scenes8.h
#	/nebular/sound_nebular.cpp /nebular/sound_nebular.h
#	/phantom/game_phantom.cpp /phantom/game_phantom.h
#	/phantom/globals_phantom.cpp /phantom/globals_phantom.h
#	/phantom/phantom_scenes.cpp /phantom/phantom_scenes.h
#	/phantom/phantom_scenes1.cpp /phantom/phantom_scenes1.h
#	/phantom/phantom_scenes2.cpp /phantom/phantom_scenes2.h
#	/phantom/phantom_scenes3.cpp /phantom/phantom_scenes3.h
#	/phantom/phantom_scenes4.cpp /phantom/phantom_scenes4.h
#	/phantom/phantom_scenes5.cpp /phantom/phantom_scenes5.h
#	/action.cpp /action.h
#	/animation.cpp /animation.h
#	/assets.cpp /assets.h
#	/audio.cpp /audio.h
#	/camera.cpp /camera.h
#	/compression.cpp /compression.h
#	/conversations.cpp /conversations.h
#	/debugger.cpp /debugger.h
#	/detection.h
#	/detection_tables.h
#	/dialogs.cpp /dialogs.h
#	/events.cpp /events.h
#	/font.cpp /font.h
#	/game.cpp /game.h
#	/game_data.cpp /game_data.h
#	/globals.cpp /globals.h
#	/hotspots.cpp /hotspots.h
#	/inventory.cpp /inventory.h
#	/mads.cpp /mads.h
#	/menu_views.cpp /menu_views.h
#	/messages.cpp /messages.h
#	/metaengine.cpp
#	/module.mk
#	/msurface.cpp /msurface.h
#	/palette.cpp /palette.h
#	/player.cpp /player.h
#	/rails.cpp /rails.h
#	/resources.cpp /resources.h
#	/scene.cpp /scene.h
#	/scene_data.cpp /scene_data.h
#	/screen.cpp /screen.h
#	/sequence.cpp /sequence.h
#	/sound.cpp /sound.h
#	/sprites.cpp /sprites.h
#	/staticres.cpp /staticres.h
#	/user_interface.cpp /user_interface.h
#)
#
#add_engine(mohawk
#	/myst_stacks/channelwood.cpp /myst_stacks/channelwood.h
#	/myst_stacks/credits.cpp /myst_stacks/credits.h
#	/myst_stacks/demo.cpp /myst_stacks/demo.h
#	/myst_stacks/dni.cpp /myst_stacks/dni.h
#	/myst_stacks/intro.cpp /myst_stacks/intro.h
#	/myst_stacks/makingof.cpp /myst_stacks/makingof.h
#	/myst_stacks/mechanical.cpp /myst_stacks/mechanical.h
#	/myst_stacks/menu.cpp /myst_stacks/menu.h
#	/myst_stacks/myst.cpp /myst_stacks/myst.h
#	/myst_stacks/preview.cpp /myst_stacks/preview.h
#	/myst_stacks/selenitic.cpp /myst_stacks/selenitic.h
#	/myst_stacks/slides.cpp /myst_stacks/slides.h
#	/myst_stacks/stoneship.cpp /myst_stacks/stoneship.h
#	/riven_stacks/aspit.cpp /riven_stacks/aspit.h
#	/riven_stacks/bspit.cpp /riven_stacks/bspit.h
#	/riven_stacks/domespit.cpp /riven_stacks/domespit.h
#	/riven_stacks/gspit.cpp /riven_stacks/gspit.h
#	/riven_stacks/jspit.cpp /riven_stacks/jspit.h
#	/riven_stacks/ospit.cpp /riven_stacks/ospit.h
#	/riven_stacks/pspit.cpp /riven_stacks/pspit.h
#	/riven_stacks/rspit.cpp /riven_stacks/rspit.h
#	/riven_stacks/tspit.cpp /riven_stacks/tspit.h
#	/bitmap.cpp /bitmap.h
#	/console.cpp /console.h
#	/cstime.h
#	/cstime_cases.h
#	/cstime_game.h
#	/cstime_graphics.h
#	/cstime_ui.h
#	/cstime_view.h
#	/cursors.cpp /cursors.h
#	/detection.h
#	/detection_tables.h
#	/dialogs.cpp /dialogs.h
#	/graphics.cpp /graphics.h
#	/installer_archive.cpp /installer_archive.h
#	/livingbooks.cpp /livingbooks.h
#	/livingbooks_code.cpp /livingbooks_code.h
#	/livingbooks_graphics.cpp /livingbooks_graphics.h
#	/livingbooks_lbx.cpp /livingbooks_lbx.h
#	/metaengine.cpp
#	/module.mk
#	/mohawk.cpp /mohawk.h
#	/myst.cpp /myst.h
#	/myst_actions.h
#	/myst_areas.cpp /myst_areas.h
#	/myst_card.cpp /myst_card.h
#	/myst_graphics.cpp /myst_graphics.h
#	/myst_metaengine.cpp /myst_metaengine.h
#	/myst_scripts.cpp /myst_scripts.h
#	/myst_sound.cpp /myst_sound.h
#	/myst_state.cpp /myst_state.h
#	/resource.cpp /resource.h
#	/resource_cache.cpp /resource_cache.h
#	/riven.cpp /riven.h
#	/riven_actions.h
#	/riven_card.cpp /riven_card.h
#	/riven_graphics.cpp /riven_graphics.h
#	/riven_graphics_detection_enums.h
#	/riven_inventory.cpp /riven_inventory.h
#	/riven_metaengine.cpp /riven_metaengine.h
#	/riven_saveload.cpp /riven_saveload.h
#	/riven_scripts.cpp /riven_scripts.h
#	/riven_sound.cpp /riven_sound.h
#	/riven_stack.cpp /riven_stack.h
#	/riven_vars.cpp
#	/riven_video.cpp /riven_video.h
#	/sound.cpp /sound.h
#	/video.cpp /video.h
#	/view.cpp /view.h
#)
#
#add_engine(mortevielle
#	/actions.cpp
#	/debugger.cpp /debugger.h
#	/detection.h
#	/detection_tables.h
#	/dialogs.cpp /dialogs.h
#	/graphics.cpp /graphics.h
#	/menu.cpp /menu.h
#	/metaengine.cpp
#	/module.mk
#	/mortevielle.cpp /mortevielle.h
#	/mouse.cpp /mouse.h
#	/outtext.cpp /outtext.h
#	/saveload.cpp /saveload.h
#	/sound.cpp /sound.h
#	/utils.cpp
#)
#
#add_engine(myst3
#	/ambient.cpp /ambient.h
#	/archive.cpp /archive.h
#	/console.cpp /console.h
#	/cursor.cpp /cursor.h
#	/database.cpp /database.h
#	/detection.h
#	/effects.cpp /effects.h
#	/gfx.cpp /gfx.h
#	/gfx_opengl.cpp /gfx_opengl.h
#	/gfx_opengl_shaders.cpp /gfx_opengl_shaders.h
#	/gfx_opengl_texture.cpp /gfx_opengl_texture.h
#	/gfx_tinygl.cpp /gfx_tinygl.h
#	/gfx_tinygl_texture.cpp /gfx_tinygl_texture.h
#	/hotspot.cpp /hotspot.h
#	/inventory.cpp /inventory.h
#	/menu.cpp /menu.h
#	/metaengine.cpp
#	/module.mk
#	/movie.cpp /movie.h
#	/myst3.cpp /myst3.h
#	/node.cpp /node.h
#	/nodecube.cpp /nodecube.h
#	/nodeframe.cpp /nodeframe.h
#	/puzzles.cpp /puzzles.h
#	/scene.cpp /scene.h
#	/script.cpp /script.h
#	/sound.cpp /sound.h
#	/state.cpp /state.h
#	/subtitles.cpp /subtitles.h
#	/transition.cpp /transition.h
#)
#
#add_engine(neverhood
#	/modules/module1000.cpp /modules/module1000.h
#	/modules/module1000_sprites.cpp /modules/module1000_sprites.h
#	/modules/module1100.cpp /modules/module1100.h
#	/modules/module1100_sprites.cpp /modules/module1100_sprites.h
#	/modules/module1200.cpp /modules/module1200.h
#	/modules/module1200_sprites.cpp /modules/module1200_sprites.h
#	/modules/module1300.cpp /modules/module1300.h
#	/modules/module1300_sprites.cpp /modules/module1300_sprites.h
#	/modules/module1400.cpp /modules/module1400.h
#	/modules/module1400_sprites.cpp /modules/module1400_sprites.h
#	/modules/module1500.cpp /modules/module1500.h
#	/modules/module1600.cpp /modules/module1600.h
#	/modules/module1600_sprites.cpp /modules/module1600_sprites.h
#	/modules/module1700.cpp /modules/module1700.h
#	/modules/module1700_sprites.cpp /modules/module1700_sprites.h
#	/modules/module1800.cpp /modules/module1800.h
#	/modules/module1900.cpp /modules/module1900.h
#	/modules/module1900_sprites.cpp /modules/module1900_sprites.h
#	/modules/module2000.cpp /modules/module2000.h
#	/modules/module2000_sprites.cpp /modules/module2000_sprites.h
#	/modules/module2100.cpp /modules/module2100.h
#	/modules/module2100_sprites.cpp /modules/module2100_sprites.h
#	/modules/module2200.cpp /modules/module2200.h
#	/modules/module2200_sprites.cpp /modules/module2200_sprites.h
#	/modules/module2300.cpp /modules/module2300.h
#	/modules/module2400.cpp /modules/module2400.h
#	/modules/module2400_sprites.cpp /modules/module2400_sprites.h
#	/modules/module2500.cpp /modules/module2500.h
#	/modules/module2500_sprites.cpp /modules/module2500_sprites.h
#	/modules/module2600.cpp /modules/module2600.h
#	/modules/module2600_sprites.cpp /modules/module2600_sprites.h
#	/modules/module2700.cpp /modules/module2700.h
#	/modules/module2700_sprites.cpp /modules/module2700_sprites.h
#	/modules/module2800.cpp /modules/module2800.h
#	/modules/module2800_sprites.cpp /modules/module2800_sprites.h
#	/modules/module2900.cpp /modules/module2900.h
#	/modules/module2900_sprites.cpp /modules/module2900_sprites.h
#	/modules/module3000.cpp /modules/module3000.h
#	/modules/module3000_sprites.cpp /modules/module3000_sprites.h
#	/background.cpp /background.h
#	/blbarchive.cpp /blbarchive.h
#	/console.cpp /console.h
#	/detection.h
#	/diskplayerscene.cpp /diskplayerscene.h
#	/entity.cpp /entity.h
#	/gamemodule.cpp /gamemodule.h
#	/gamevars.cpp /gamevars.h
#	/graphics.cpp /graphics.h
#	/klaymen.cpp /klaymen.h
#	/menumodule.cpp /menumodule.h
#	/messages.h
#	/metaengine.cpp
#	/microtiles.cpp /microtiles.h
#	/module.h /module.mk
#	/module_scene.cpp
#	/mouse.cpp /mouse.h
#	/navigationscene.cpp /navigationscene.h
#	/neverhood.cpp /neverhood.h
#	/palette.cpp /palette.h
#	/resource.cpp /resource.h
#	/resourceman.cpp /resourceman.h
#	/saveload.cpp
#	/scene.cpp /scene.h
#	/screen.cpp /screen.h
#	/smackerplayer.cpp /smackerplayer.h
#	/smackerscene.cpp /smackerscene.h
#	/sound.cpp /sound.h
#	/sprite.cpp /sprite.h
#	/staticdata.cpp /staticdata.h
#)
#
#add_engine(ngi
#	/fullpipe/scene01.cpp
#	/fullpipe/scene02.cpp
#	/fullpipe/scene03.cpp
#	/fullpipe/scene04.cpp
#	/fullpipe/scene05.cpp
#	/fullpipe/scene06.cpp
#	/fullpipe/scene07.cpp
#	/fullpipe/scene08.cpp
#	/fullpipe/scene09.cpp
#	/fullpipe/scene10.cpp
#	/fullpipe/scene11.cpp
#	/fullpipe/scene12.cpp
#	/fullpipe/scene13.cpp
#	/fullpipe/scene14.cpp
#	/fullpipe/scene15.cpp
#	/fullpipe/scene16.cpp
#	/fullpipe/scene17.cpp
#	/fullpipe/scene18and19.cpp
#	/fullpipe/scene20.cpp
#	/fullpipe/scene21.cpp
#	/fullpipe/scene22.cpp
#	/fullpipe/scene23.cpp
#	/fullpipe/scene24.cpp
#	/fullpipe/scene25.cpp
#	/fullpipe/scene26.cpp
#	/fullpipe/scene27.cpp
#	/fullpipe/scene28.cpp
#	/fullpipe/scene29.cpp
#	/fullpipe/scene30.cpp
#	/fullpipe/scene31.cpp
#	/fullpipe/scene32.cpp
#	/fullpipe/scene33.cpp
#	/fullpipe/scene34.cpp
#	/fullpipe/scene35.cpp
#	/fullpipe/scene36.cpp
#	/fullpipe/scene37.cpp
#	/fullpipe/scene38.cpp
#	/fullpipe/sceneDbg.cpp
#	/fullpipe/sceneFinal.cpp
#	/fullpipe/sceneIntro.cpp
#	/fullpipe/sceneIntroDemo.cpp
#	/anihandler.cpp /anihandler.h
#	/behavior.cpp /behavior.h
#	/console.cpp /console.h
#	/constants.h
#	/detection.h
#	/floaters.cpp /floaters.h
#	/gameloader.cpp /gameloader.h
#	/gfx.cpp /gfx.h
#	/init.cpp
#	/input.cpp /input.h
#	/interaction.cpp /interaction.h
#	/inventory.cpp /inventory.h
#	/lift.cpp
#	/messagehandlers.cpp
#	/messages.cpp /messages.h
#	/metaengine.cpp
#	/modal.cpp /modal.h
#	/module.mk
#	/motion.cpp /motion.h
#	/ngi.cpp /ngi.h
#	/ngiarchive.cpp /ngiarchive.h
#	/objectnames.h
#	/objects.h
#	/scene.cpp /scene.h
#	/scenes.cpp /scenes.h
#	/sound.cpp /sound.h
#	/stateloader.cpp
#	/statesaver.cpp
#	/statics.cpp /statics.h
#	/utils.cpp /utils.h
#)
#
#add_engine(parallaction
#	/adlib.cpp
#	/balloons.cpp
#	/callables_br.cpp
#	/callables_ns.cpp
#	/debug.cpp /debug.h
#	/detection.h
#	/dialogue.cpp
#	/disk.h
#	/disk_br.cpp
#	/disk_ns.cpp
#	/exec.cpp /exec.h
#	/exec_br.cpp
#	/exec_ns.cpp
#	/font.cpp
#	/gfxbase.cpp
#	/graphics.cpp /graphics.h
#	/gui.cpp /gui.h
#	/gui_br.cpp
#	/gui_ns.cpp
#	/input.cpp /input.h
#	/inventory.cpp /inventory.h
#	/metaengine.cpp
#	/module.mk
#	/objects.cpp /objects.h
#	/parallaction.cpp /parallaction.h
#	/parallaction_br.cpp
#	/parallaction_ns.cpp
#	/parser.cpp /parser.h
#	/parser_br.cpp
#	/parser_ns.cpp
#	/saveload.cpp /saveload.h
#	/sound.h
#	/sound_br.cpp
#	/sound_ns.cpp
#	/staticres.cpp
#	/walk.cpp /walk.h
#)
#
#add_engine(pegasus
#	/ai/ai_action.cpp /ai/ai_action.h
#	/ai/ai_area.cpp /ai/ai_area.h
#	/ai/ai_condition.cpp /ai/ai_condition.h
#	/ai/ai_rule.cpp /ai/ai_rule.h
#	/items/biochips/aichip.cpp /items/biochips/aichip.h
#	/items/biochips/arthurchip.cpp /items/biochips/arthurchip.h
#	/items/biochips/biochipitem.cpp /items/biochips/biochipitem.h
#	/items/biochips/mapchip.cpp /items/biochips/mapchip.h
#	/items/biochips/mapimage.cpp /items/biochips/mapimage.h
#	/items/biochips/opticalchip.cpp /items/biochips/opticalchip.h
#	/items/biochips/pegasuschip.cpp /items/biochips/pegasuschip.h
#	/items/biochips/retscanchip.cpp /items/biochips/retscanchip.h
#	/items/biochips/shieldchip.cpp /items/biochips/shieldchip.h
#	/items/inventory/airmask.cpp /items/inventory/airmask.h
#	/items/inventory/gascanister.cpp /items/inventory/gascanister.h
#	/items/inventory/inventoryitem.cpp /items/inventory/inventoryitem.h
#	/items/inventory/keycard.cpp /items/inventory/keycard.h
#	/items/autodragger.cpp /items/autodragger.h
#	/items/inventory.cpp /items/inventory.h
#	/items/inventorypicture.cpp /items/inventorypicture.h
#	/items/item.cpp /items/item.h
#	/items/itemdragger.cpp /items/itemdragger.h
#	/items/itemlist.cpp /items/itemlist.h
#	/neighborhood/caldoria/caldoria.cpp /neighborhood/caldoria/caldoria.h
#	/neighborhood/caldoria/caldoria4dsystem.cpp /neighborhood/caldoria/caldoria4dsystem.h
#	/neighborhood/caldoria/caldoriabomb.cpp /neighborhood/caldoria/caldoriabomb.h
#	/neighborhood/caldoria/caldoriamessages.cpp /neighborhood/caldoria/caldoriamessages.h
#	/neighborhood/caldoria/caldoriamirror.cpp /neighborhood/caldoria/caldoriamirror.h
#	/neighborhood/mars/canyonchase.cpp /neighborhood/mars/canyonchase.h
#	/neighborhood/mars/constants.h
#	/neighborhood/mars/energybeam.cpp /neighborhood/mars/energybeam.h
#	/neighborhood/mars/gravitoncannon.cpp /neighborhood/mars/gravitoncannon.h
#	/neighborhood/mars/hermite.cpp /neighborhood/mars/hermite.h
#	/neighborhood/mars/mars.cpp /neighborhood/mars/mars.h
#	/neighborhood/mars/planetmover.cpp /neighborhood/mars/planetmover.h
#	/neighborhood/mars/reactor.cpp /neighborhood/mars/reactor.h
#	/neighborhood/mars/robotship.cpp /neighborhood/mars/robotship.h
#	/neighborhood/mars/shuttleenergymeter.cpp /neighborhood/mars/shuttleenergymeter.h
#	/neighborhood/mars/shuttlehud.cpp /neighborhood/mars/shuttlehud.h
#	/neighborhood/mars/shuttleweapon.cpp /neighborhood/mars/shuttleweapon.h
#	/neighborhood/mars/spacechase3d.cpp /neighborhood/mars/spacechase3d.h
#	/neighborhood/mars/spacejunk.cpp /neighborhood/mars/spacejunk.h
#	/neighborhood/mars/tractorbeam.cpp /neighborhood/mars/tractorbeam.h
#	/neighborhood/mars/tunnelpod.cpp /neighborhood/mars/tunnelpod.h
#	/neighborhood/norad/alpha/ecrmonitor.cpp /neighborhood/norad/alpha/ecrmonitor.h
#	/neighborhood/norad/alpha/fillingstation.cpp /neighborhood/norad/alpha/fillingstation.h
#	/neighborhood/norad/alpha/noradalpha.cpp /neighborhood/norad/alpha/noradalpha.h
#	/neighborhood/norad/alpha/panorama.cpp /neighborhood/norad/alpha/panorama.h
#	/neighborhood/norad/alpha/panoramascroll.cpp /neighborhood/norad/alpha/panoramascroll.h
#	/neighborhood/norad/alpha/subchase.cpp /neighborhood/norad/alpha/subchase.h
#	/neighborhood/norad/delta/globegame.cpp /neighborhood/norad/delta/globegame.h
#	/neighborhood/norad/delta/noraddelta.cpp /neighborhood/norad/delta/noraddelta.h
#	/neighborhood/norad/constants.h
#	/neighborhood/norad/norad.cpp /neighborhood/norad/norad.h
#	/neighborhood/norad/noradelevator.cpp /neighborhood/norad/noradelevator.h
#	/neighborhood/norad/pressuredoor.cpp /neighborhood/norad/pressuredoor.h
#	/neighborhood/norad/pressuretracker.cpp /neighborhood/norad/pressuretracker.h
#	/neighborhood/norad/subcontrolroom.cpp /neighborhood/norad/subcontrolroom.h
#	/neighborhood/norad/subplatform.cpp /neighborhood/norad/subplatform.h
#	/neighborhood/prehistoric/prehistoric.cpp /neighborhood/prehistoric/prehistoric.h
#	/neighborhood/tsa/fulltsa.cpp /neighborhood/tsa/fulltsa.h
#	/neighborhood/tsa/tinytsa.cpp /neighborhood/tsa/tinytsa.h
#	/neighborhood/wsc/moleculebin.cpp /neighborhood/wsc/moleculebin.h
#	/neighborhood/wsc/wsc.cpp /neighborhood/wsc/wsc.h
#	/neighborhood/door.cpp /neighborhood/door.h
#	/neighborhood/exit.cpp /neighborhood/exit.h
#	/neighborhood/extra.cpp /neighborhood/extra.h
#	/neighborhood/hotspotinfo.cpp /neighborhood/hotspotinfo.h
#	/neighborhood/neighborhood.cpp /neighborhood/neighborhood.h
#	/neighborhood/spot.cpp /neighborhood/spot.h
#	/neighborhood/turn.cpp /neighborhood/turn.h
#	/neighborhood/view.cpp /neighborhood/view.h
#	/neighborhood/zoom.cpp /neighborhood/zoom.h
#	/chase.cpp /chase.h
#	/compass.cpp /compass.h
#	/console.cpp /console.h
#	/constants.h
#	/cursor.cpp /cursor.h
#	/detection.h
#	/elements.cpp /elements.h
#	/energymonitor.cpp /energymonitor.h
#	/fader.cpp /fader.h
#	/gamestate.cpp /gamestate.h
#	/graphics.cpp /graphics.h
#	/hotspot.cpp /hotspot.h
#	/input.cpp /input.h
#	/interaction.cpp /interaction.h
#	/interface.cpp /interface.h
#	/menu.cpp /menu.h
#	/metaengine.cpp
#	/module.mk
#	/movie.cpp /movie.h
#	/notification.cpp /notification.h
#	/pegasus.cpp /pegasus.h
#	/scoring.h
#	/sound.cpp /sound.h
#	/surface.cpp /surface.h
#	/timers.cpp /timers.h
#	/transition.cpp /transition.h
#	/types.h
#	/util.cpp /util.h
#)
#
#add_engine(petka
#	/interfaces/dialog_interface.cpp /interfaces/dialog_interface.h
#	/interfaces/interface.cpp /interfaces/interface.h
#	/interfaces/main.cpp /interfaces/main.h
#	/interfaces/map.cpp /interfaces/map.h
#	/interfaces/panel.cpp /interfaces/panel.h
#	/interfaces/save_load.cpp /interfaces/save_load.h
#	/interfaces/sequence.cpp /interfaces/sequence.h
#	/interfaces/startup.cpp /interfaces/startup.h
#	/objects/heroes.cpp /objects/heroes.h
#	/objects/object.cpp /objects/object.h
#	/objects/object_bg.cpp /objects/object_bg.h
#	/objects/object_case.cpp /objects/object_case.h
#	/objects/object_cursor.cpp /objects/object_cursor.h
#	/objects/object_star.cpp /objects/object_star.h
#	/objects/text.cpp /objects/text.h
#	/base.h
#	/big_dialogue.cpp /big_dialogue.h
#	/detection_tables.h
#	/file_mgr.cpp /file_mgr.h
#	/flc.cpp /flc.h
#	/metaengine.cpp
#	/module.mk
#	/petka.cpp /petka.h
#	/q_manager.cpp /q_manager.h
#	/q_system.cpp /q_system.h
#	/saveload.cpp
#	/sound.cpp /sound.h
#	/video.cpp /video.h
#	/walk.cpp /walk.h
#)
#
#add_engine(plumbers
#	/console.cpp /console.h
#	/metaengine.cpp
#	/module.mk
#	/plumbers.cpp /plumbers.h
#)

add_engine(prince
	/animation.cpp /animation.h
	/archive.cpp /archive.h
	/common.h
	/cursor.cpp /cursor.h
	/curve_values.h
	/debugger.cpp /debugger.h
	/decompress.cpp /decompress.h
	/detection.h
	/draw.cpp
	/flags.cpp /flags.h
	/font.cpp /font.h
	/graphics.cpp /graphics.h
	/hero.cpp /hero.h
	/hero_set.h
	/inventory.cpp
	/metaengine.cpp
	/mhwanh.cpp /mhwanh.h
	/mob.cpp /mob.h
	/module.mk
	/musNum.h
	/music.cpp /music.h
	/object.cpp /object.h
	/option_text.h
	/prince.cpp /prince.h
	/pscr.cpp /pscr.h
	/resource.cpp /resource.h
	/saveload.cpp
	/script.cpp /script.h
	/sound.cpp
	/variatxt.cpp /variatxt.h
	/videoplayer.cpp
	/walk.cpp
)

add_engine(private
	/code.cpp
	/cursors.cpp
	/decompiler.cpp /decompiler.h
	/funcs.cpp
	/grammar.cpp /grammar.h
	/lexer.cpp
	/metaengine.cpp
	/module.mk
	/private.cpp /private.h
	/symbol.cpp /symbol.h
	/tokens.h
)

add_engine(queen
	/bankman.cpp /bankman.h
	/command.cpp /command.h
	/credits.cpp /credits.h
	/cutaway.cpp /cutaway.h
	/debug.cpp /debug.h
	/defs.h
	/detection.h
	/display.cpp /display.h
	/graphics.cpp /graphics.h
	/grid.cpp /grid.h
	/input.cpp /input.h
	/journal.cpp /journal.h
	/logic.cpp /logic.h
	/metaengine.cpp
	/midiadlib.cpp /midiadlib.h
	/module.mk
	/music.cpp /music.h
	/musicdata.cpp
	/queen.cpp /queen.h
	/resource.cpp /resource.h
	/restables.cpp
	/sound.cpp /sound.h
	/state.cpp /state.h
	/structs.h
	/talk.cpp /talk.h
	/walk.cpp /walk.h
)

add_engine(saga
	/actor.cpp /actor.h
	/actor_path.cpp
	/actor_walk.cpp
	/animation.cpp /animation.h
	/console.cpp /console.h
	/detection.h
	/detection_tables.h
	/displayinfo.h
	/events.cpp /events.h
	/font.cpp /font.h
	/font_map.cpp
	/gfx.cpp /gfx.h
	/image.cpp
	/input.cpp
	/interface.cpp /interface.h
	/introproc_ihnm.cpp
	/introproc_ite.cpp
	/isomap.cpp /isomap.h
	/itedata.cpp /itedata.h
	/metaengine.cpp
	/module.mk
	/music.cpp /music.h
	/objectmap.cpp /objectmap.h
	/palanim.cpp /palanim.h
	/puzzle.cpp /puzzle.h
	/render.cpp /render.h
	/resource.cpp /resource.h
	/resource_res.cpp
	/resource_rsc.cpp
	/saga.cpp /saga.h
	/saveload.cpp
	/scene.cpp /scene.h
	/script.cpp /script.h
	/sfuncs.cpp
	/sfuncs_ihnm.cpp
	/shared_detection_defines.h
	/sndres.cpp /sndres.h
	/sound.cpp /sound.h
	/sprite.cpp /sprite.h
	/sthread.cpp
)

add_engine(sci
	/engine/features.cpp /engine/features.h
	/engine/file.cpp /engine/file.h
	/engine/gc.cpp /engine/gc.h
	/engine/guest_additions.cpp /engine/guest_additions.h
	/engine/hoyle5poker.cpp /engine/hoyle5poker.h
	/engine/kernel.cpp /engine/kernel.h
	/engine/kernel_tables.h
	/engine/kevent.cpp
	/engine/kfile.cpp
	/engine/kgraphics.cpp
	/engine/kgraphics32.cpp
	/engine/klists.cpp
	/engine/kmath.cpp
	/engine/kmenu.cpp
	/engine/kmisc.cpp
	/engine/kmovement.cpp
	/engine/kparse.cpp
	/engine/kpathing.cpp
	/engine/kscripts.cpp
	/engine/ksound.cpp
	/engine/kstring.cpp
	/engine/kvideo.cpp
	/engine/message.cpp /engine/message.h
	/engine/object.cpp /engine/object.h
	/engine/savegame.cpp /engine/savegame.h
	/engine/script.cpp /engine/script.h
	/engine/script_patches.cpp /engine/script_patches.h
	/engine/scriptdebug.cpp /engine/scriptdebug.h
	/engine/seg_manager.cpp /engine/seg_manager.h
	/engine/segment.cpp /engine/segment.h
	/engine/selector.cpp /engine/selector.h
	/engine/state.cpp /engine/state.h
	/engine/static_selectors.cpp
	/engine/tts.cpp /engine/tts.h
	/engine/vm.cpp /engine/vm.h
	/engine/vm_types.cpp /engine/vm_types.h
	/engine/workarounds.cpp /engine/workarounds.h
	/graphics/animate.cpp /graphics/animate.h
	/graphics/cache.cpp /graphics/cache.h
	/graphics/celobj32.cpp /graphics/celobj32.h
	/graphics/compare.cpp /graphics/compare.h
	/graphics/controls16.cpp /graphics/controls16.h
	/graphics/controls32.cpp /graphics/controls32.h
	/graphics/coordadjuster.cpp /graphics/coordadjuster.h
	/graphics/cursor.cpp /graphics/cursor.h
	/graphics/cursor32.cpp /graphics/cursor32.h
	/graphics/fontkorean.cpp /graphics/fontkorean.h
	/graphics/fontsjis.cpp /graphics/fontsjis.h
	/graphics/frameout.cpp /graphics/frameout.h
	/graphics/helpers.h
	/graphics/helpers_detection_enums.h
	/graphics/lists32.h
	/graphics/maccursor32.cpp /graphics/maccursor32.h
	/graphics/maciconbar.cpp /graphics/maciconbar.h
	/graphics/menu.cpp /graphics/menu.h
	/graphics/paint16.cpp /graphics/paint16.h
	/graphics/paint32.cpp /graphics/paint32.h
	/graphics/palette.cpp /graphics/palette.h
	/graphics/palette32.cpp /graphics/palette32.h
	/graphics/picture.cpp /graphics/picture.h
	/graphics/plane32.cpp /graphics/plane32.h
	/graphics/portrait.cpp /graphics/portrait.h
	/graphics/ports.cpp /graphics/ports.h
	/graphics/remap.cpp /graphics/remap.h
	/graphics/remap32.cpp /graphics/remap32.h
	/graphics/scifont.cpp /graphics/scifont.h
	/graphics/scifx.cpp /graphics/scifx.h
	/graphics/screen.cpp /graphics/screen.h
	/graphics/screen_item32.cpp /graphics/screen_item32.h
	/graphics/text16.cpp /graphics/text16.h
	/graphics/text32.cpp /graphics/text32.h
	/graphics/transitions.cpp /graphics/transitions.h
	/graphics/transitions32.cpp /graphics/transitions32.h
	/graphics/video32.cpp /graphics/video32.h
	/graphics/view.cpp /graphics/view.h
	/parser/grammar.cpp
	/parser/said.cpp
	/parser/vocabulary.cpp /parser/vocabulary.h
	/resource/decompressor.cpp /resource/decompressor.h
	/resource/resource.cpp /resource/resource.h
	/resource/resource_audio.cpp
	/resource/resource_intern.h
	/resource/resource_patcher.cpp /resource/resource_patcher.h
	/sound/decoders/sol.cpp /sound/decoders/sol.h
	/sound/drivers/adlib.cpp
	/sound/drivers/amigamac0.cpp
	/sound/drivers/amigamac1.cpp
	/sound/drivers/cms.cpp
	/sound/drivers/fb01.cpp
	/sound/drivers/fmtowns.cpp
	/sound/drivers/gm_names.h
	/sound/drivers/macmixer.h
	/sound/drivers/map-mt32-to-gm.h
	/sound/drivers/midi.cpp
	/sound/drivers/mididriver.h
	/sound/drivers/pc9801.cpp
	/sound/drivers/pcjr.cpp
	/sound/audio.cpp /sound/audio.h
	/sound/audio32.cpp /sound/audio32.h
	/sound/midiparser_sci.cpp /sound/midiparser_sci.h
	/sound/music.cpp /sound/music.h
	/sound/soundcmd.cpp /sound/soundcmd.h
	/sound/sync.cpp /sound/sync.h
	/video/robot_decoder.cpp /video/robot_decoder.h
	/video/seq_decoder.cpp /video/seq_decoder.h
	/console.cpp /console.h
	/debug.h
	/detection.h
	/detection_options.h
	/detection_tables.h
	/dialogs.cpp /dialogs.h
	/event.cpp /event.h
	/metaengine.cpp
	/module.mk
	/sci.cpp /sci.h
	/util.cpp /util.h
)

add_engine(scumm
	/he/logic/baseball2001.cpp
	/he/logic/basketball.cpp
	/he/logic/football.cpp
	/he/logic/funshop.cpp
	/he/logic/moonbase_logic.cpp
	/he/logic/puttrace.cpp
	/he/logic/soccer.cpp
	/he/moonbase/ai_defenseunit.cpp /he/moonbase/ai_defenseunit.h
	/he/moonbase/ai_main.cpp /he/moonbase/ai_main.h
	/he/moonbase/ai_node.cpp /he/moonbase/ai_node.h
	/he/moonbase/ai_pattern.h
	/he/moonbase/ai_targetacquisition.cpp /he/moonbase/ai_targetacquisition.h
	/he/moonbase/ai_traveller.cpp /he/moonbase/ai_traveller.h
	/he/moonbase/ai_tree.cpp /he/moonbase/ai_tree.h
	/he/moonbase/ai_types.cpp /he/moonbase/ai_types.h
	/he/moonbase/ai_weapon.cpp /he/moonbase/ai_weapon.h
	/he/moonbase/distortion.cpp
	/he/moonbase/moonbase.cpp /he/moonbase/moonbase.h
	/he/moonbase/moonbase_fow.cpp
	/he/moonbase/net_defines.h
	/he/moonbase/net_main.cpp /he/moonbase/net_main.h
	/he/animation_he.cpp /he/animation_he.h
	/he/cup_player_he.cpp /he/cup_player_he.h
	/he/floodfill_he.cpp /he/floodfill_he.h
	/he/intern_he.h
	/he/localizer.cpp /he/localizer.h
	/he/logic_he.cpp /he/logic_he.h
	/he/palette_he.cpp
	/he/resource_he.cpp /he/resource_he.h
	/he/script_v100he.cpp
	/he/script_v60he.cpp
	/he/script_v70he.cpp
	/he/script_v71he.cpp
	/he/script_v72he.cpp
	/he/script_v80he.cpp
	/he/script_v90he.cpp
	/he/sound_he.cpp /he/sound_he.h
	/he/sprite_he.cpp /he/sprite_he.h
	/he/wiz_he.cpp /he/wiz_he.h
	/imuse/drivers/amiga.cpp /imuse/drivers/amiga.h
	/imuse/drivers/fmtowns.cpp /imuse/drivers/fmtowns.h
	/imuse/drivers/mac_m68k.cpp /imuse/drivers/mac_m68k.h
	/imuse/drivers/pcspk.cpp /imuse/drivers/pcspk.h
	/imuse/imuse.cpp /imuse/imuse.h
	/imuse/imuse_internal.h
	/imuse/imuse_part.cpp
	/imuse/imuse_player.cpp
	/imuse/instrument.cpp /imuse/instrument.h
	/imuse/sysex.h
	/imuse/sysex_samnmax.cpp
	/imuse/sysex_scumm.cpp
	/imuse_digi/dimuse_bndmgr.cpp /imuse_digi/dimuse_bndmgr.h
	/imuse_digi/dimuse_cmds.cpp
	/imuse_digi/dimuse_codecs.cpp /imuse_digi/dimuse_codecs.h
	/imuse_digi/dimuse_defs.h
	/imuse_digi/dimuse_dispatch.cpp
	/imuse_digi/dimuse_engine.cpp /imuse_digi/dimuse_engine.h
	/imuse_digi/dimuse_fades.cpp /imuse_digi/dimuse_fades.h
	/imuse_digi/dimuse_files.cpp /imuse_digi/dimuse_files.h
	/imuse_digi/dimuse_groups.cpp /imuse_digi/dimuse_groups.h
	/imuse_digi/dimuse_internalmixer.cpp /imuse_digi/dimuse_internalmixer.h
	/imuse_digi/dimuse_scripts.cpp
	/imuse_digi/dimuse_sndmgr.cpp /imuse_digi/dimuse_sndmgr.h
	/imuse_digi/dimuse_streamer.cpp
	/imuse_digi/dimuse_tables.cpp /imuse_digi/dimuse_tables.h
	/imuse_digi/dimuse_tracks.cpp
	/imuse_digi/dimuse_triggers.cpp /imuse_digi/dimuse_triggers.h
	/imuse_digi/dimuse_utils.cpp
	/imuse_digi/dimuse_wave.cpp
	/imuse_digi/dimuse_waveout.cpp
	/insane/insane.cpp /insane/insane.h
	/insane/insane_ben.cpp
	/insane/insane_enemy.cpp
	/insane/insane_iact.cpp
	/insane/insane_scenes.cpp
	/players/player_ad.cpp /players/player_ad.h
	/players/player_apple2.cpp /players/player_apple2.h
	/players/player_he.cpp /players/player_he.h
	/players/player_mac.cpp /players/player_mac.h
	/players/player_mod.cpp /players/player_mod.h
	/players/player_nes.cpp /players/player_nes.h
	/players/player_pce.cpp /players/player_pce.h
	/players/player_sid.cpp /players/player_sid.h
	/players/player_towns.cpp /players/player_towns.h
	/players/player_v1.cpp /players/player_v1.h
	/players/player_v2.cpp /players/player_v2.h
	/players/player_v2a.cpp /players/player_v2a.h
	/players/player_v2base.cpp /players/player_v2base.h
	/players/player_v2cms.cpp /players/player_v2cms.h
	/players/player_v3a.cpp /players/player_v3a.h
	/players/player_v3m.cpp /players/player_v3m.h
	/players/player_v4a.cpp /players/player_v4a.h
	/players/player_v5m.cpp /players/player_v5m.h
	/smush/channel.cpp /smush/channel.h
	/smush/codec1.cpp
	/smush/codec20.cpp
	/smush/codec37.cpp /smush/codec37.h
	/smush/codec47.cpp /smush/codec47.h
	/smush/saud_channel.cpp
	/smush/smush_font.cpp /smush/smush_font.h
	/smush/smush_mixer.cpp /smush/smush_mixer.h
	/smush/smush_player.cpp /smush/smush_player.h
	/actor.cpp /actor.h
	/actor_he.h
	/akos.cpp /akos.h
	/base-costume.cpp /base-costume.h
	/bomp.cpp /bomp.h
	/boxes.cpp /boxes.h
	/camera.cpp
	/cdda.cpp /cdda.h
	/charset-fontdata.cpp
	/charset.cpp /charset.h
	/costume.cpp /costume.h
	/cursor.cpp
	/debugger.cpp /debugger.h
	/detection.h
	/detection_internal.h
	/detection_steam.h
	/detection_tables.h
	/dialogs.cpp /dialogs.h
	/file.cpp /file.h
	/file_nes.cpp /file_nes.h
	/gfx.cpp /gfx.h
	/gfx_mac.cpp
	/gfx_towns.cpp
	/help.cpp /help.h
	/input.cpp
	/ks_check.cpp /ks_check.h
	/metaengine.cpp /metaengine.h
	/midiparser_ro.cpp
	/module.mk
	/music.h
	/nut_renderer.cpp /nut_renderer.h
	/object.cpp /object.h
	/obsolete.h
	/palette.cpp
	/resource.cpp /resource.h
	/resource_v2.cpp
	/resource_v3.cpp
	/resource_v4.cpp
	/room.cpp
	/saveload.cpp
	/script.cpp /script.h
	/script_v0.cpp
	/script_v2.cpp
	/script_v3.cpp
	/script_v4.cpp
	/script_v5.cpp
	/script_v6.cpp
	/script_v8.cpp
	/scumm-md5.h
	/scumm.cpp /scumm.h
	/scumm_v0.h
	/scumm_v2.h
	/scumm_v3.h
	/scumm_v4.h
	/scumm_v5.h
	/scumm_v6.h
	/scumm_v7.h
	/scumm_v8.h
	/sound.cpp /sound.h
	/string.cpp
	/usage_bits.cpp /usage_bits.h
	/util.cpp /util.h
	/vars.cpp
	/verbs.cpp /verbs.h
)

add_engine(sherlock
	/scalpel/3do/scalpel_3do_screen.cpp /scalpel/3do/scalpel_3do_screen.h
	/scalpel/drivers/adlib.cpp
	/scalpel/drivers/mididriver.h
	/scalpel/drivers/mt32.cpp
	/scalpel/tsage/logo.cpp /scalpel/tsage/logo.h
	/scalpel/tsage/resources.cpp /scalpel/tsage/resources.h
	/scalpel/scalpel.cpp /scalpel/scalpel.h
	/scalpel/scalpel_darts.cpp /scalpel/scalpel_darts.h
	/scalpel/scalpel_debugger.cpp /scalpel/scalpel_debugger.h
	/scalpel/scalpel_fixed_text.cpp /scalpel/scalpel_fixed_text.h
	/scalpel/scalpel_inventory.cpp /scalpel/scalpel_inventory.h
	/scalpel/scalpel_journal.cpp /scalpel/scalpel_journal.h
	/scalpel/scalpel_map.cpp /scalpel/scalpel_map.h
	/scalpel/scalpel_people.cpp /scalpel/scalpel_people.h
	/scalpel/scalpel_saveload.cpp /scalpel/scalpel_saveload.h
	/scalpel/scalpel_scene.cpp /scalpel/scalpel_scene.h
	/scalpel/scalpel_screen.cpp /scalpel/scalpel_screen.h
	/scalpel/scalpel_talk.cpp /scalpel/scalpel_talk.h
	/scalpel/scalpel_user_interface.cpp /scalpel/scalpel_user_interface.h
	/scalpel/settings.cpp /scalpel/settings.h
	/tattoo/tattoo.cpp /tattoo/tattoo.h
	/tattoo/tattoo_darts.cpp /tattoo/tattoo_darts.h
	/tattoo/tattoo_debugger.cpp /tattoo/tattoo_debugger.h
	/tattoo/tattoo_fixed_text.cpp /tattoo/tattoo_fixed_text.h
	/tattoo/tattoo_inventory.cpp /tattoo/tattoo_inventory.h
	/tattoo/tattoo_journal.cpp /tattoo/tattoo_journal.h
	/tattoo/tattoo_map.cpp /tattoo/tattoo_map.h
	/tattoo/tattoo_people.cpp /tattoo/tattoo_people.h
	/tattoo/tattoo_resources.cpp /tattoo/tattoo_resources.h
	/tattoo/tattoo_scene.cpp /tattoo/tattoo_scene.h
	/tattoo/tattoo_screen.cpp /tattoo/tattoo_screen.h
	/tattoo/tattoo_talk.cpp /tattoo/tattoo_talk.h
	/tattoo/tattoo_user_interface.cpp /tattoo/tattoo_user_interface.h
	/tattoo/widget_base.cpp /tattoo/widget_base.h
	/tattoo/widget_credits.cpp /tattoo/widget_credits.h
	/tattoo/widget_files.cpp /tattoo/widget_files.h
	/tattoo/widget_foolscap.cpp /tattoo/widget_foolscap.h
	/tattoo/widget_inventory.cpp /tattoo/widget_inventory.h
	/tattoo/widget_lab.cpp /tattoo/widget_lab.h
	/tattoo/widget_options.cpp /tattoo/widget_options.h
	/tattoo/widget_password.cpp /tattoo/widget_password.h
	/tattoo/widget_quit.cpp /tattoo/widget_quit.h
	/tattoo/widget_talk.cpp /tattoo/widget_talk.h
	/tattoo/widget_text.cpp /tattoo/widget_text.h
	/tattoo/widget_tooltip.cpp /tattoo/widget_tooltip.h
	/tattoo/widget_verbs.cpp /tattoo/widget_verbs.h
	/animation.cpp /animation.h
	/debugger.cpp /debugger.h
	/detection.h
	/detection_tables.h
	/events.cpp /events.h
	/fixed_text.cpp /fixed_text.h
	/fonts.cpp /fonts.h
	/image_file.cpp /image_file.h
	/inventory.cpp /inventory.h
	/journal.cpp /journal.h
	/map.cpp /map.h
	/metaengine.cpp
	/module.mk
	/music.cpp /music.h
	/objects.cpp /objects.h
	/people.cpp /people.h
	/resources.cpp /resources.h
	/saveload.cpp /saveload.h
	/scene.cpp /scene.h
	/screen.cpp /screen.h
	/sherlock.cpp /sherlock.h
	/sound.cpp /sound.h
	/surface.cpp /surface.h
	/talk.cpp /talk.h
	/user_interface.cpp /user_interface.h
)

add_engine(sky
	/music/adlibchannel.cpp /music/adlibchannel.h
	/music/adlibmusic.cpp /music/adlibmusic.h
	/music/gmchannel.cpp /music/gmchannel.h
	/music/gmmusic.cpp /music/gmmusic.h
	/music/mt32music.cpp /music/mt32music.h
	/music/musicbase.cpp /music/musicbase.h
	/autoroute.cpp /autoroute.h
	/compact.cpp /compact.h
	/control.cpp /control.h
	/debug.cpp /debug.h
	/disk.cpp /disk.h
	/grid.cpp /grid.h
	/hufftext.cpp
	/intro.cpp /intro.h
	/logic.cpp /logic.h
	/metaengine.cpp
	/module.mk
	/mouse.cpp /mouse.h
	/rnc_deco.cpp /rnc_deco.h
	/screen.cpp /screen.h
	/sky.cpp /sky.h
	/skydefs.h
	/sound.cpp /sound.h
	/struc.h
	/text.cpp /text.h
)

add_engine(stark
	/formats/biff.cpp /formats/biff.h
	/formats/biffmesh.cpp /formats/biffmesh.h
	/formats/dds.cpp /formats/dds.h
	/formats/iss.cpp /formats/iss.h
	/formats/tm.cpp /formats/tm.h
	/formats/xarc.cpp /formats/xarc.h
	/formats/xmg.cpp /formats/xmg.h
	/formats/xrc.cpp /formats/xrc.h
	/gfx/driver.cpp /gfx/driver.h
	/gfx/faderenderer.h
	/gfx/opengl.cpp /gfx/opengl.h
	/gfx/openglactor.cpp /gfx/openglactor.h
	/gfx/openglfade.cpp /gfx/openglfade.h
	/gfx/openglprop.cpp /gfx/openglprop.h
	/gfx/opengls.cpp /gfx/opengls.h
	/gfx/openglsactor.cpp /gfx/openglsactor.h
	/gfx/openglsfade.cpp /gfx/openglsfade.h
	/gfx/openglsprop.cpp /gfx/openglsprop.h
	/gfx/openglssurface.cpp /gfx/openglssurface.h
	/gfx/openglsurface.cpp /gfx/openglsurface.h
	/gfx/opengltexture.cpp /gfx/opengltexture.h
	/gfx/renderentry.cpp /gfx/renderentry.h
	/gfx/surfacerenderer.cpp /gfx/surfacerenderer.h
	/gfx/texture.cpp /gfx/texture.h
	/gfx/tinygl.cpp /gfx/tinygl.h
	/gfx/tinyglactor.cpp /gfx/tinyglactor.h
	/gfx/tinyglbitmap.cpp /gfx/tinyglbitmap.h
	/gfx/tinyglfade.cpp /gfx/tinyglfade.h
	/gfx/tinyglprop.cpp /gfx/tinyglprop.h
	/gfx/tinyglsurface.cpp /gfx/tinyglsurface.h
	/gfx/tinygltexture.cpp /gfx/tinygltexture.h
	/model/animhandler.cpp /model/animhandler.h
	/model/model.cpp /model/model.h
	/model/skeleton_anim.cpp /model/skeleton_anim.h
	/movement/followpath.cpp /movement/followpath.h
	/movement/followpathlight.cpp /movement/followpathlight.h
	/movement/movement.cpp /movement/movement.h
	/movement/shortestpath.cpp /movement/shortestpath.h
	/movement/stringpullingpath.cpp /movement/stringpullingpath.h
	/movement/turn.cpp /movement/turn.h
	/movement/walk.cpp /movement/walk.h
	/resources/anim.cpp /resources/anim.h
	/resources/animhierarchy.cpp /resources/animhierarchy.h
	/resources/animscript.cpp /resources/animscript.h
	/resources/animsoundtrigger.cpp /resources/animsoundtrigger.h
	/resources/bonesmesh.cpp /resources/bonesmesh.h
	/resources/bookmark.cpp /resources/bookmark.h
	/resources/camera.cpp /resources/camera.h
	/resources/command.cpp /resources/command.h
	/resources/container.cpp /resources/container.h
	/resources/dialog.cpp /resources/dialog.h
	/resources/direction.cpp /resources/direction.h
	/resources/floor.cpp /resources/floor.h
	/resources/floorface.cpp /resources/floorface.h
	/resources/floorfield.cpp /resources/floorfield.h
	/resources/fmv.cpp /resources/fmv.h
	/resources/image.cpp /resources/image.h
	/resources/item.cpp /resources/item.h
	/resources/knowledge.cpp /resources/knowledge.h
	/resources/knowledgeset.cpp /resources/knowledgeset.h
	/resources/layer.cpp /resources/layer.h
	/resources/level.cpp /resources/level.h
	/resources/light.cpp /resources/light.h
	/resources/lipsync.cpp /resources/lipsync.h
	/resources/location.cpp /resources/location.h
	/resources/object.cpp /resources/object.h
	/resources/path.cpp /resources/path.h
	/resources/pattable.cpp /resources/pattable.h
	/resources/root.cpp /resources/root.h
	/resources/script.cpp /resources/script.h
	/resources/scroll.cpp /resources/scroll.h
	/resources/sound.cpp /resources/sound.h
	/resources/speech.cpp /resources/speech.h
	/resources/string.cpp /resources/string.h
	/resources/textureset.cpp /resources/textureset.h
	/services/archiveloader.cpp /services/archiveloader.h
	/services/dialogplayer.cpp /services/dialogplayer.h
	/services/diary.cpp /services/diary.h
	/services/fontprovider.cpp /services/fontprovider.h
	/services/gamechapter.cpp /services/gamechapter.h
	/services/gameinterface.cpp /services/gameinterface.h
	/services/gamemessage.cpp /services/gamemessage.h
	/services/global.cpp /services/global.h
	/services/resourceprovider.cpp /services/resourceprovider.h
	/services/services.cpp /services/services.h
	/services/settings.cpp /services/settings.h
	/services/stateprovider.cpp /services/stateprovider.h
	/services/staticprovider.cpp /services/staticprovider.h
	/services/userinterface.cpp /services/userinterface.h
	/tools/abstractsyntaxtree.cpp /tools/abstractsyntaxtree.h
	/tools/block.cpp /tools/block.h
	/tools/command.cpp /tools/command.h
	/tools/decompiler.cpp /tools/decompiler.h
	/ui/menu/dialogmenu.cpp /ui/menu/dialogmenu.h
	/ui/menu/diaryindex.cpp /ui/menu/diaryindex.h
	/ui/menu/diarypages.cpp /ui/menu/diarypages.h
	/ui/menu/fmvmenu.cpp /ui/menu/fmvmenu.h
	/ui/menu/locationscreen.cpp /ui/menu/locationscreen.h
	/ui/menu/mainmenu.cpp /ui/menu/mainmenu.h
	/ui/menu/saveloadmenu.cpp /ui/menu/saveloadmenu.h
	/ui/menu/settingsmenu.cpp /ui/menu/settingsmenu.h
	/ui/world/actionmenu.cpp /ui/world/actionmenu.h
	/ui/world/button.cpp /ui/world/button.h
	/ui/world/clicktext.cpp /ui/world/clicktext.h
	/ui/world/dialogpanel.cpp /ui/world/dialogpanel.h
	/ui/world/fmvscreen.cpp /ui/world/fmvscreen.h
	/ui/world/gamescreen.cpp /ui/world/gamescreen.h
	/ui/world/gamewindow.cpp /ui/world/gamewindow.h
	/ui/world/inventorywindow.cpp /ui/world/inventorywindow.h
	/ui/world/topmenu.cpp /ui/world/topmenu.h
	/ui/cursor.cpp /ui/cursor.h
	/ui/dialogbox.cpp /ui/dialogbox.h
	/ui/screen.h
	/ui/window.cpp /ui/window.h
	/visual/effects/bubbles.cpp /visual/effects/bubbles.h
	/visual/effects/effect.cpp /visual/effects/effect.h
	/visual/effects/fireflies.cpp /visual/effects/fireflies.h
	/visual/effects/fish.cpp /visual/effects/fish.h
	/visual/actor.cpp /visual/actor.h
	/visual/explodingimage.cpp /visual/explodingimage.h
	/visual/flashingimage.cpp /visual/flashingimage.h
	/visual/image.cpp /visual/image.h
	/visual/prop.cpp /visual/prop.h
	/visual/smacker.cpp /visual/smacker.h
	/visual/text.cpp /visual/text.h
	/visual/visual.h
	/console.cpp /console.h
	/debug.h
	/detection.h
	/metaengine.cpp
	/module.mk
	/resourcereference.cpp /resourcereference.h
	/savemetadata.cpp /savemetadata.h
	/scene.cpp /scene.h
	/stark.cpp /stark.h
)

add_engine(supernova
	/supernova1/rooms.cpp /supernova1/rooms.h
	/supernova1/state.cpp /supernova1/state.h
	/supernova1/stringid.h
	/supernova2/rooms.cpp /supernova2/rooms.h
	/supernova2/state.cpp /supernova2/state.h
	/supernova2/stringid.h
	/console.cpp /console.h
	/game-manager.cpp /game-manager.h
	/graphics.cpp /graphics.h
	/imageid.h
	/metaengine.cpp
	/module.mk
	/msn_def.h
	/resman.cpp /resman.h
	/room.cpp /room.h
	/screen.cpp /screen.h
	/sound.cpp /sound.h
	/supernova.cpp /supernova.h
)

add_engine(sword1
	/animation.cpp /animation.h
	/collision.h
	/console.cpp /console.h
	/control.cpp /control.h
	/debug.cpp /debug.h
	/eventman.cpp /eventman.h
	/logic.cpp /logic.h
	/memman.cpp /memman.h
	/menu.cpp /menu.h
	/metaengine.cpp
	/module.mk
	/mouse.cpp /mouse.h
	/music.cpp /music.h
	/object.h
	/objectman.cpp /objectman.h
	/resman.cpp /resman.h
	/router.cpp /router.h
	/screen.cpp /screen.h
	/sound.cpp /sound.h
	/staticres.cpp
	/sword1.cpp /sword1.h
	/sworddefs.h
	/swordres.h
	/text.cpp /text.h
)

add_engine(sword2
	/animation.cpp /animation.h
	/anims.cpp
	/console.cpp /console.h
	/controls.cpp /controls.h
	/debug.cpp /debug.h
	/defs.h
	/detection.h
	/detection_internal.h
	/events.cpp
	/function.cpp
	/header.cpp /header.h
	/icons.cpp
	/interpreter.cpp /interpreter.h
	/layers.cpp
	/logic.cpp /logic.h
	/maketext.cpp /maketext.h
	/memory.cpp /memory.h
	/menu.cpp
	/metaengine.cpp
	/module.mk
	/mouse.cpp /mouse.h
	/music.cpp
	/object.h
	/palette.cpp
	/protocol.cpp
	/render.cpp
	/resman.cpp /resman.h
	/router.cpp /router.h
	/saveload.cpp /saveload.h
	/screen.cpp /screen.h
	/scroll.cpp
	/sound.cpp /sound.h
	/speech.cpp
	/sprite.cpp
	/startup.cpp
	/sword2.cpp /sword2.h
	/sync.cpp
	/walker.cpp
)

add_engine(sword25
	/fmv/movieplayer.cpp /fmv/movieplayer.h
	/fmv/movieplayer_script.cpp
	/gfx/image/art.cpp /gfx/image/art.h
	/gfx/image/image.h
	/gfx/image/imgloader.cpp /gfx/image/imgloader.h
	/gfx/image/renderedimage.cpp /gfx/image/renderedimage.h
	/gfx/image/swimage.cpp /gfx/image/swimage.h
	/gfx/image/vectorimage.cpp /gfx/image/vectorimage.h
	/gfx/image/vectorimagerenderer.cpp
	/gfx/animation.cpp /gfx/animation.h
	/gfx/animationdescription.cpp /gfx/animationdescription.h
	/gfx/animationresource.cpp /gfx/animationresource.h
	/gfx/animationtemplate.cpp /gfx/animationtemplate.h
	/gfx/animationtemplateregistry.cpp /gfx/animationtemplateregistry.h
	/gfx/bitmap.cpp /gfx/bitmap.h
	/gfx/bitmapresource.h
	/gfx/dynamicbitmap.cpp /gfx/dynamicbitmap.h
	/gfx/fontresource.cpp /gfx/fontresource.h
	/gfx/graphicengine.cpp /gfx/graphicengine.h
	/gfx/graphicengine_script.cpp
	/gfx/microtiles.cpp /gfx/microtiles.h
	/gfx/panel.cpp /gfx/panel.h
	/gfx/renderobject.cpp /gfx/renderobject.h
	/gfx/renderobjectmanager.cpp /gfx/renderobjectmanager.h
	/gfx/renderobjectptr.h
	/gfx/renderobjectregistry.h
	/gfx/rootrenderobject.h
	/gfx/screenshot.cpp /gfx/screenshot.h
	/gfx/staticbitmap.cpp /gfx/staticbitmap.h
	/gfx/text.cpp /gfx/text.h
	/gfx/timedrenderobject.cpp /gfx/timedrenderobject.h
	/input/inputengine.cpp /input/inputengine.h
	/input/inputengine_script.cpp
	/kernel/common.h
	/kernel/filesystemutil.cpp /kernel/filesystemutil.h
	/kernel/inputpersistenceblock.cpp /kernel/inputpersistenceblock.h
	/kernel/kernel.cpp /kernel/kernel.h
	/kernel/kernel_script.cpp
	/kernel/objectregistry.h
	/kernel/outputpersistenceblock.cpp /kernel/outputpersistenceblock.h
	/kernel/persistable.h
	/kernel/persistenceblock.h
	/kernel/persistenceservice.cpp /kernel/persistenceservice.h
	/kernel/resmanager.cpp /kernel/resmanager.h
	/kernel/resource.cpp /kernel/resource.h
	/kernel/resservice.h
	/kernel/service.h
	/math/geometry.h
	/math/geometry_script.cpp
	/math/line.h
	/math/polygon.cpp /math/polygon.h
	/math/region.cpp /math/region.h
	/math/regionregistry.cpp /math/regionregistry.h
	/math/vertex.cpp /math/vertex.h
	/math/walkregion.cpp /math/walkregion.h
	/package/packagemanager.cpp /package/packagemanager.h
	/package/packagemanager_script.cpp
	/script/lua_extensions.cpp
	/script/luabindhelper.cpp /script/luabindhelper.h
	/script/luacallback.cpp /script/luacallback.h
	/script/luascript.cpp /script/luascript.h
	/script/script.h
	/sfx/soundengine.cpp /sfx/soundengine.h
	/sfx/soundengine_script.cpp
	/console.cpp /console.h
	/detection.h
	/detection_tables.h
	/metaengine.cpp
	/module.mk
	/sword25.cpp /sword25.h
)

add_engine(teenagent
	/actor.cpp /actor.h
	/animation.cpp /animation.h
	/callbacks.cpp
	/console.cpp /console.h
	/dialog.cpp /dialog.h
	/font.cpp /font.h
	/inventory.cpp /inventory.h
	/metaengine.cpp
	/module.mk
	/music.cpp /music.h
	/objects.cpp /objects.h
	/pack.cpp /pack.h
	/resources.cpp /resources.h
	/scene.cpp /scene.h
	/segment.cpp /segment.h
	/surface.cpp /surface.h
	/surface_list.cpp /surface_list.h
	/teenagent.cpp /teenagent.h
)

add_engine(tinsel
	/noir/lzss.cpp /noir/lzss.h
	/actors.cpp /actors.h
	/adpcm.cpp /adpcm.h
	/anim.cpp /anim.h
	/background.cpp /background.h
	/bg.cpp
	/bmv.cpp /bmv.h
	/cliprect.cpp /cliprect.h
	/config.cpp /config.h
	/cursor.cpp /cursor.h
	/debugger.cpp /debugger.h
	/detection.h
	/detection_tables.h
	/dialogs.cpp /dialogs.h
	/drives.cpp /drives.h
	/dw.h
	/effect.cpp
	/events.cpp /events.h
	/faders.cpp /faders.h
	/film.h
	/font.cpp /font.h
	/graphics.cpp /graphics.h
	/handle.cpp /handle.h
	/heapmem.cpp /heapmem.h
	/mareels.cpp /mareels.h
	/metaengine.cpp
	/module.mk
	/move.cpp /move.h
	/movers.cpp /movers.h
	/multiobj.cpp /multiobj.h
	/music.cpp /music.h
	/object.cpp /object.h
	/palette.cpp /palette.h
	/pcode.cpp /pcode.h
	/pdisplay.cpp /pdisplay.h
	/pid.h
	/play.cpp /play.h
	/polygons.cpp /polygons.h
	/saveload.cpp
	/savescn.cpp /savescn.h
	/scene.cpp /scene.h
	/sched.cpp /sched.h
	/scn.cpp /scn.h
	/scroll.cpp /scroll.h
	/sound.cpp /sound.h
	/strres.cpp /strres.h
	/sysvar.cpp /sysvar.h
	/text.cpp /text.h
	/timers.cpp /timers.h
	/tinlib.cpp /tinlib.h
	/tinsel.cpp /tinsel.h
	/token.cpp /token.h
)
#
#add_engine(titanic
#	/carry/arm.cpp /carry/arm.h
#	/carry/auditory_centre.cpp /carry/auditory_centre.h
#	/carry/bowl_ear.cpp /carry/bowl_ear.h
#	/carry/brain.cpp /carry/brain.h
#	/carry/bridge_piece.cpp /carry/bridge_piece.h
#	/carry/carry.cpp /carry/carry.h
#	/carry/carry_parrot.cpp /carry/carry_parrot.h
#	/carry/central_core.cpp /carry/central_core.h
#	/carry/chicken.cpp /carry/chicken.h
#	/carry/crushed_tv.cpp /carry/crushed_tv.h
#	/carry/ear.cpp /carry/ear.h
#	/carry/eye.cpp /carry/eye.h
#	/carry/feathers.cpp /carry/feathers.h
#	/carry/fruit.cpp /carry/fruit.h
#	/carry/glass.cpp /carry/glass.h
#	/carry/hammer.cpp /carry/hammer.h
#	/carry/head_piece.cpp /carry/head_piece.h
#	/carry/hose.cpp /carry/hose.h
#	/carry/hose_end.cpp /carry/hose_end.h
#	/carry/key.cpp /carry/key.h
#	/carry/liftbot_head.cpp /carry/liftbot_head.h
#	/carry/long_stick.cpp /carry/long_stick.h
#	/carry/magazine.cpp /carry/magazine.h
#	/carry/maitred_left_arm.cpp /carry/maitred_left_arm.h
#	/carry/maitred_right_arm.cpp /carry/maitred_right_arm.h
#	/carry/mouth.cpp /carry/mouth.h
#	/carry/napkin.cpp /carry/napkin.h
#	/carry/nose.cpp /carry/nose.h
#	/carry/note.cpp /carry/note.h
#	/carry/parcel.cpp /carry/parcel.h
#	/carry/perch.cpp /carry/perch.h
#	/carry/phonograph_cylinder.cpp /carry/phonograph_cylinder.h
#	/carry/phonograph_ear.cpp /carry/phonograph_ear.h
#	/carry/photograph.cpp /carry/photograph.h
#	/carry/plug_in.cpp /carry/plug_in.h
#	/carry/speech_centre.cpp /carry/speech_centre.h
#	/carry/sweets.cpp /carry/sweets.h
#	/carry/vision_centre.cpp /carry/vision_centre.h
#	/core/background.cpp /core/background.h
#	/core/click_responder.cpp /core/click_responder.h
#	/core/dont_save_file_item.cpp /core/dont_save_file_item.h
#	/core/drop_target.cpp /core/drop_target.h
#	/core/file_item.cpp /core/file_item.h
#	/core/game_object.cpp /core/game_object.h
#	/core/game_object_desc_item.cpp /core/game_object_desc_item.h
#	/core/link_item.cpp /core/link_item.h
#	/core/list.cpp /core/list.h
#	/core/mail_man.cpp /core/mail_man.h
#	/core/message_target.cpp /core/message_target.h
#	/core/multi_drop_target.cpp /core/multi_drop_target.h
#	/core/named_item.cpp /core/named_item.h
#	/core/node_item.cpp /core/node_item.h
#	/core/project_item.cpp /core/project_item.h
#	/core/resource_key.cpp /core/resource_key.h
#	/core/room_item.cpp /core/room_item.h
#	/core/saveable_object.cpp /core/saveable_object.h
#	/core/static_image.cpp /core/static_image.h
#	/core/tree_item.cpp /core/tree_item.h
#	/core/turn_on_object.cpp /core/turn_on_object.h
#	/core/turn_on_play_sound.cpp /core/turn_on_play_sound.h
#	/core/turn_on_turn_off.cpp /core/turn_on_turn_off.h
#	/core/view_item.cpp /core/view_item.h
#	/game/gondolier/gondolier_base.cpp /game/gondolier/gondolier_base.h
#	/game/gondolier/gondolier_chest.cpp /game/gondolier/gondolier_chest.h
#	/game/gondolier/gondolier_face.cpp /game/gondolier/gondolier_face.h
#	/game/gondolier/gondolier_mixer.cpp /game/gondolier/gondolier_mixer.h
#	/game/gondolier/gondolier_slider.cpp /game/gondolier/gondolier_slider.h
#	/game/maitred/maitred_arm_holder.cpp /game/maitred/maitred_arm_holder.h
#	/game/maitred/maitred_body.cpp /game/maitred/maitred_body.h
#	/game/maitred/maitred_legs.cpp /game/maitred/maitred_legs.h
#	/game/maitred/maitred_prod_receptor.cpp /game/maitred/maitred_prod_receptor.h
#	/game/parrot/parrot_lobby_controller.cpp /game/parrot/parrot_lobby_controller.h
#	/game/parrot/parrot_lobby_link_updater.cpp /game/parrot/parrot_lobby_link_updater.h
#	/game/parrot/parrot_lobby_object.cpp /game/parrot/parrot_lobby_object.h
#	/game/parrot/parrot_lobby_view_object.cpp /game/parrot/parrot_lobby_view_object.h
#	/game/parrot/parrot_loser.cpp /game/parrot/parrot_loser.h
#	/game/parrot/parrot_nut_bowl_actor.cpp /game/parrot/parrot_nut_bowl_actor.h
#	/game/parrot/parrot_nut_eater.cpp /game/parrot/parrot_nut_eater.h
#	/game/parrot/parrot_perch_holder.cpp /game/parrot/parrot_perch_holder.h
#	/game/parrot/parrot_trigger.cpp /game/parrot/parrot_trigger.h
#	/game/parrot/player_meets_parrot.cpp /game/parrot/player_meets_parrot.h
#	/game/pet/pet.cpp /game/pet/pet.h
#	/game/pet/pet_class1.cpp /game/pet/pet_class1.h
#	/game/pet/pet_class2.cpp /game/pet/pet_class2.h
#	/game/pet/pet_class3.cpp /game/pet/pet_class3.h
#	/game/pet/pet_lift.cpp /game/pet/pet_lift.h
#	/game/pet/pet_monitor.cpp /game/pet/pet_monitor.h
#	/game/pet/pet_pellerator.cpp /game/pet/pet_pellerator.h
#	/game/pet/pet_position.cpp /game/pet/pet_position.h
#	/game/pet/pet_sentinal.cpp /game/pet/pet_sentinal.h
#	/game/pet/pet_sounds.cpp /game/pet/pet_sounds.h
#	/game/pet/pet_transition.cpp /game/pet/pet_transition.h
#	/game/pet/pet_transport.cpp /game/pet/pet_transport.h
#	/game/pet/pet_val_base.h
#	/game/pickup/pick_up.cpp /game/pickup/pick_up.h
#	/game/pickup/pick_up_bar_glass.cpp /game/pickup/pick_up_bar_glass.h
#	/game/pickup/pick_up_hose.cpp /game/pickup/pick_up_hose.h
#	/game/pickup/pick_up_lemon.cpp /game/pickup/pick_up_lemon.h
#	/game/pickup/pick_up_speech_centre.cpp /game/pickup/pick_up_speech_centre.h
#	/game/pickup/pick_up_vis_centre.cpp /game/pickup/pick_up_vis_centre.h
#	/game/placeholder/bar_shelf_vis_centre.cpp /game/placeholder/bar_shelf_vis_centre.h
#	/game/placeholder/lemon_on_bar.cpp /game/placeholder/lemon_on_bar.h
#	/game/placeholder/place_holder.cpp /game/placeholder/place_holder.h
#	/game/placeholder/tv_on_bar.cpp /game/placeholder/tv_on_bar.h
#	/game/sgt/armchair.cpp /game/sgt/armchair.h
#	/game/sgt/basin.cpp /game/sgt/basin.h
#	/game/sgt/bedfoot.cpp /game/sgt/bedfoot.h
#	/game/sgt/bedhead.cpp /game/sgt/bedhead.h
#	/game/sgt/chest_of_drawers.cpp /game/sgt/chest_of_drawers.h
#	/game/sgt/desk.cpp /game/sgt/desk.h
#	/game/sgt/deskchair.cpp /game/sgt/deskchair.h
#	/game/sgt/drawer.cpp /game/sgt/drawer.h
#	/game/sgt/sgt_doors.cpp /game/sgt/sgt_doors.h
#	/game/sgt/sgt_nav.cpp /game/sgt/sgt_nav.h
#	/game/sgt/sgt_navigation.cpp /game/sgt/sgt_navigation.h
#	/game/sgt/sgt_restaurant_doors.cpp /game/sgt/sgt_restaurant_doors.h
#	/game/sgt/sgt_state_control.cpp /game/sgt/sgt_state_control.h
#	/game/sgt/sgt_state_room.cpp /game/sgt/sgt_state_room.h
#	/game/sgt/sgt_tv.cpp /game/sgt/sgt_tv.h
#	/game/sgt/sgt_upper_doors_sound.cpp /game/sgt/sgt_upper_doors_sound.h
#	/game/sgt/toilet.cpp /game/sgt/toilet.h
#	/game/sgt/vase.cpp /game/sgt/vase.h
#	/game/sgt/washstand.cpp /game/sgt/washstand.h
#	/game/transport/exit_pellerator.h
#	/game/transport/gondolier.cpp /game/transport/gondolier.h
#	/game/transport/lift.cpp /game/transport/lift.h
#	/game/transport/lift_indicator.cpp /game/transport/lift_indicator.h
#	/game/transport/pellerator.cpp /game/transport/pellerator.h
#	/game/transport/service_elevator.cpp /game/transport/service_elevator.h
#	/game/transport/transport.cpp /game/transport/transport.h
#	/game/announce.cpp /game/announce.h
#	/game/annoy_barbot.cpp /game/annoy_barbot.h
#	/game/arb_background.cpp /game/arb_background.h
#	/game/arboretum_gate.cpp /game/arboretum_gate.h
#	/game/auto_animate.cpp /game/auto_animate.h
#	/game/bar_bell.cpp /game/bar_bell.h
#	/game/bar_menu.cpp /game/bar_menu.h
#	/game/bar_menu_button.cpp /game/bar_menu_button.h
#	/game/belbot_get_light.cpp /game/belbot_get_light.h
#	/game/bomb.cpp /game/bomb.h
#	/game/bottom_of_well_monitor.cpp /game/bottom_of_well_monitor.h
#	/game/bowl_unlocker.cpp /game/bowl_unlocker.h
#	/game/brain_slot.cpp /game/brain_slot.h
#	/game/bridge_door.cpp /game/bridge_door.h
#	/game/bridge_view.cpp /game/bridge_view.h
#	/game/broken_pell_base.cpp /game/broken_pell_base.h
#	/game/broken_pellerator.cpp /game/broken_pellerator.h
#	/game/broken_pellerator_froz.cpp /game/broken_pellerator_froz.h
#	/game/cage.cpp /game/cage.h
#	/game/captains_wheel.cpp /game/captains_wheel.h
#	/game/cdrom.cpp /game/cdrom.h
#	/game/cdrom_computer.cpp /game/cdrom_computer.h
#	/game/cdrom_tray.cpp /game/cdrom_tray.h
#	/game/cell_point_button.cpp /game/cell_point_button.h
#	/game/chev_code.cpp /game/chev_code.h
#	/game/chev_panel.cpp /game/chev_panel.h
#	/game/chicken_cooler.cpp /game/chicken_cooler.h
#	/game/chicken_dispensor.cpp /game/chicken_dispensor.h
#	/game/close_broken_pel.cpp /game/close_broken_pel.h
#	/game/code_wheel.cpp /game/code_wheel.h
#	/game/computer.cpp /game/computer.h
#	/game/computer_screen.cpp /game/computer_screen.h
#	/game/cookie.cpp /game/cookie.h
#	/game/credits.cpp /game/credits.h
#	/game/credits_button.cpp /game/credits_button.h
#	/game/dead_area.cpp /game/dead_area.h
#	/game/desk_click_responder.cpp /game/desk_click_responder.h
#	/game/doorbot_elevator_handler.cpp /game/doorbot_elevator_handler.h
#	/game/doorbot_home_handler.cpp /game/doorbot_home_handler.h
#	/game/ear_sweet_bowl.cpp /game/ear_sweet_bowl.h
#	/game/eject_phonograph_button.cpp /game/eject_phonograph_button.h
#	/game/elevator_action_area.cpp /game/elevator_action_area.h
#	/game/emma_control.cpp /game/emma_control.h
#	/game/empty_nut_bowl.cpp /game/empty_nut_bowl.h
#	/game/end_credit_text.cpp /game/end_credit_text.h
#	/game/end_credits.cpp /game/end_credits.h
#	/game/end_explode_ship.cpp /game/end_explode_ship.h
#	/game/end_game_credits.cpp /game/end_game_credits.h
#	/game/end_sequence_control.cpp /game/end_sequence_control.h
#	/game/fan.cpp /game/fan.h
#	/game/fan_control.cpp /game/fan_control.h
#	/game/fan_decrease.cpp /game/fan_decrease.h
#	/game/fan_increase.cpp /game/fan_increase.h
#	/game/fan_noises.cpp /game/fan_noises.h
#	/game/floor_indicator.cpp /game/floor_indicator.h
#	/game/games_console.cpp /game/games_console.h
#	/game/get_lift_eye2.cpp /game/get_lift_eye2.h
#	/game/glass_smasher.cpp /game/glass_smasher.h
#	/game/hammer_clip.cpp /game/hammer_clip.h
#	/game/hammer_dispensor.cpp /game/hammer_dispensor.h
#	/game/hammer_dispensor_button.cpp /game/hammer_dispensor_button.h
#	/game/head_slot.cpp /game/head_slot.h
#	/game/head_smash_event.cpp /game/head_smash_event.h
#	/game/head_smash_lever.cpp /game/head_smash_lever.h
#	/game/head_spinner.cpp /game/head_spinner.h
#	/game/idle_summoner.cpp /game/idle_summoner.h
#	/game/leave_sec_class_state.cpp /game/leave_sec_class_state.h
#	/game/lemon_dispensor.cpp /game/lemon_dispensor.h
#	/game/light.cpp /game/light.h
#	/game/light_switch.cpp /game/light_switch.h
#	/game/little_lift_button.cpp /game/little_lift_button.h
#	/game/long_stick_dispenser.cpp /game/long_stick_dispenser.h
#	/game/missiveomat.cpp /game/missiveomat.h
#	/game/missiveomat_button.cpp /game/missiveomat_button.h
#	/game/movie_tester.cpp /game/movie_tester.h
#	/game/music_console_button.cpp /game/music_console_button.h
#	/game/music_room_phonograph.cpp /game/music_room_phonograph.h
#	/game/music_room_stop_phonograph_button.cpp /game/music_room_stop_phonograph_button.h
#	/game/music_system_lock.cpp /game/music_system_lock.h
#	/game/musical_instrument.cpp /game/musical_instrument.h
#	/game/nav_helmet.cpp /game/nav_helmet.h
#	/game/nav_helmet_off.cpp /game/nav_helmet_off.h
#	/game/nav_helmet_on.cpp /game/nav_helmet_on.h
#	/game/navigation_computer.cpp /game/navigation_computer.h
#	/game/no_nut_bowl.cpp /game/no_nut_bowl.h
#	/game/nose_holder.cpp /game/nose_holder.h
#	/game/null_port_hole.cpp /game/null_port_hole.h
#	/game/nut_replacer.cpp /game/nut_replacer.h
#	/game/pet_disabler.cpp /game/pet_disabler.h
#	/game/phonograph.cpp /game/phonograph.h
#	/game/phonograph_lid.cpp /game/phonograph_lid.h
#	/game/place_holder_item.cpp /game/place_holder_item.h
#	/game/play_music_button.cpp /game/play_music_button.h
#	/game/play_on_act.cpp /game/play_on_act.h
#	/game/port_hole.cpp /game/port_hole.h
#	/game/record_phonograph_button.cpp /game/record_phonograph_button.h
#	/game/replacement_ear.cpp /game/replacement_ear.h
#	/game/reserved_table.cpp /game/reserved_table.h
#	/game/restaurant_cylinder_holder.cpp /game/restaurant_cylinder_holder.h
#	/game/restaurant_phonograph.cpp /game/restaurant_phonograph.h
#	/game/sauce_dispensor.cpp /game/sauce_dispensor.h
#	/game/search_point.cpp /game/search_point.h
#	/game/season_background.cpp /game/season_background.h
#	/game/season_barrel.cpp /game/season_barrel.h
#	/game/seasonal_adjustment.cpp /game/seasonal_adjustment.h
#	/game/service_elevator_window.cpp /game/service_elevator_window.h
#	/game/ship_setting.cpp /game/ship_setting.h
#	/game/ship_setting_button.cpp /game/ship_setting_button.h
#	/game/show_cell_points.cpp /game/show_cell_points.h
#	/game/speech_dispensor.cpp /game/speech_dispensor.h
#	/game/splash_animation.cpp /game/splash_animation.h
#	/game/starling_puret.cpp /game/starling_puret.h
#	/game/start_action.cpp /game/start_action.h
#	/game/stop_phonograph_button.cpp /game/stop_phonograph_button.h
#	/game/sub_glass.cpp /game/sub_glass.h
#	/game/sub_wrapper.cpp /game/sub_wrapper.h
#	/game/sweet_bowl.cpp /game/sweet_bowl.h
#	/game/television.cpp /game/television.h
#	/game/third_class_canal.cpp /game/third_class_canal.h
#	/game/throw_tv_down_well.cpp /game/throw_tv_down_well.h
#	/game/titania_still_control.cpp /game/titania_still_control.h
#	/game/tow_parrot_nav.cpp /game/tow_parrot_nav.h
#	/game/up_lighter.cpp /game/up_lighter.h
#	/game/useless_lever.cpp /game/useless_lever.h
#	/game/variable_list.cpp /game/variable_list.h
#	/game/volume_control.cpp /game/volume_control.h
#	/game/wheel_button.cpp /game/wheel_button.h
#	/game/wheel_hotspot.cpp /game/wheel_hotspot.h
#	/game/wheel_spin.cpp /game/wheel_spin.h
#	/game/wheel_spin_horn.cpp /game/wheel_spin_horn.h
#	/gfx/act_button.cpp /gfx/act_button.h
#	/gfx/changes_season_button.cpp /gfx/changes_season_button.h
#	/gfx/chev_left_off.cpp /gfx/chev_left_off.h
#	/gfx/chev_left_on.cpp /gfx/chev_left_on.h
#	/gfx/chev_right_off.cpp /gfx/chev_right_off.h
#	/gfx/chev_right_on.cpp /gfx/chev_right_on.h
#	/gfx/chev_send_rec_switch.cpp /gfx/chev_send_rec_switch.h
#	/gfx/edit_control.cpp /gfx/edit_control.h
#	/gfx/elevator_button.cpp /gfx/elevator_button.h
#	/gfx/get_from_succ.cpp /gfx/get_from_succ.h
#	/gfx/helmet_on_off.cpp /gfx/helmet_on_off.h
#	/gfx/home_photo.cpp /gfx/home_photo.h
#	/gfx/icon_nav_action.cpp /gfx/icon_nav_action.h
#	/gfx/icon_nav_butt.cpp /gfx/icon_nav_butt.h
#	/gfx/icon_nav_down.cpp /gfx/icon_nav_down.h
#	/gfx/icon_nav_image.cpp /gfx/icon_nav_image.h
#	/gfx/icon_nav_left.cpp /gfx/icon_nav_left.h
#	/gfx/icon_nav_receive.cpp /gfx/icon_nav_receive.h
#	/gfx/icon_nav_right.cpp /gfx/icon_nav_right.h
#	/gfx/icon_nav_send.cpp /gfx/icon_nav_send.h
#	/gfx/icon_nav_up.cpp /gfx/icon_nav_up.h
#	/gfx/keybrd_butt.cpp /gfx/keybrd_butt.h
#	/gfx/move_object_button.cpp /gfx/move_object_button.h
#	/gfx/music_control.cpp /gfx/music_control.h
#	/gfx/music_slider.cpp /gfx/music_slider.h
#	/gfx/music_slider_pitch.cpp /gfx/music_slider_pitch.h
#	/gfx/music_slider_speed.cpp /gfx/music_slider_speed.h
#	/gfx/music_switch.cpp /gfx/music_switch.h
#	/gfx/music_switch_inversion.cpp /gfx/music_switch_inversion.h
#	/gfx/music_switch_reverse.cpp /gfx/music_switch_reverse.h
#	/gfx/music_voice_mute.cpp /gfx/music_voice_mute.h
#	/gfx/send_to_succ.cpp /gfx/send_to_succ.h
#	/gfx/sgt_selector.cpp /gfx/sgt_selector.h
#	/gfx/slider_button.cpp /gfx/slider_button.h
#	/gfx/small_chev_left_off.cpp /gfx/small_chev_left_off.h
#	/gfx/small_chev_left_on.cpp /gfx/small_chev_left_on.h
#	/gfx/small_chev_right_off.cpp /gfx/small_chev_right_off.h
#	/gfx/small_chev_right_on.cpp /gfx/small_chev_right_on.h
#	/gfx/st_button.cpp /gfx/st_button.h
#	/gfx/status_change_button.cpp /gfx/status_change_button.h
#	/gfx/text_control.cpp /gfx/text_control.h
#	/gfx/text_down.cpp /gfx/text_down.h
#	/gfx/text_skrew.cpp /gfx/text_skrew.h
#	/gfx/text_up.cpp /gfx/text_up.h
#	/gfx/toggle_button.cpp /gfx/toggle_button.h
#	/gfx/toggle_switch.cpp /gfx/toggle_switch.h
#	/messages/auto_sound_event.cpp /messages/auto_sound_event.h
#	/messages/bilge_auto_sound_event.cpp /messages/bilge_auto_sound_event.h
#	/messages/bilge_dispensor_event.cpp /messages/bilge_dispensor_event.h
#	/messages/door_auto_sound_event.cpp /messages/door_auto_sound_event.h
#	/messages/messages.cpp /messages/messages.h
#	/messages/mouse_messages.cpp /messages/mouse_messages.h
#	/messages/pet_messages.h
#	/messages/service_elevator_door.cpp /messages/service_elevator_door.h
#	/moves/call_pellerator.cpp /moves/call_pellerator.h
#	/moves/enter_bomb_room.cpp /moves/enter_bomb_room.h
#	/moves/enter_bridge.cpp /moves/enter_bridge.h
#	/moves/enter_exit_first_class_state.cpp /moves/enter_exit_first_class_state.h
#	/moves/enter_exit_mini_lift.cpp /moves/enter_exit_mini_lift.h
#	/moves/enter_exit_sec_class_mini_lift.cpp /moves/enter_exit_sec_class_mini_lift.h
#	/moves/enter_exit_view.cpp /moves/enter_exit_view.h
#	/moves/enter_sec_class_state.cpp /moves/enter_sec_class_state.h
#	/moves/exit_arboretum.cpp /moves/exit_arboretum.h
#	/moves/exit_bridge.cpp /moves/exit_bridge.h
#	/moves/exit_lift.cpp /moves/exit_lift.h
#	/moves/exit_pellerator.cpp /moves/exit_pellerator.h
#	/moves/exit_state_room.cpp /moves/exit_state_room.h
#	/moves/exit_tiania.cpp /moves/exit_tiania.h
#	/moves/move_player_in_parrot_room.cpp /moves/move_player_in_parrot_room.h
#	/moves/move_player_to.cpp /moves/move_player_to.h
#	/moves/move_player_to_from.cpp /moves/move_player_to_from.h
#	/moves/multi_move.cpp /moves/multi_move.h
#	/moves/pan_from_pel.cpp /moves/pan_from_pel.h
#	/moves/restaurant_pan_handler.cpp /moves/restaurant_pan_handler.h
#	/moves/restricted_move.cpp /moves/restricted_move.h
#	/moves/scraliontis_table.cpp /moves/scraliontis_table.h
#	/moves/trip_down_canal.cpp /moves/trip_down_canal.h
#	/npcs/barbot.cpp /npcs/barbot.h
#	/npcs/bellbot.cpp /npcs/bellbot.h
#	/npcs/bilge_succubus.cpp /npcs/bilge_succubus.h
#	/npcs/callbot.cpp /npcs/callbot.h
#	/npcs/character.cpp /npcs/character.h
#	/npcs/deskbot.cpp /npcs/deskbot.h
#	/npcs/doorbot.cpp /npcs/doorbot.h
#	/npcs/liftbot.cpp /npcs/liftbot.h
#	/npcs/maitre_d.cpp /npcs/maitre_d.h
#	/npcs/mobile.cpp /npcs/mobile.h
#	/npcs/parrot.cpp /npcs/parrot.h
#	/npcs/parrot_succubus.cpp /npcs/parrot_succubus.h
#	/npcs/robot_controller.cpp /npcs/robot_controller.h
#	/npcs/starlings.cpp /npcs/starlings.h
#	/npcs/succubus.cpp /npcs/succubus.h
#	/npcs/summon_bots.cpp /npcs/summon_bots.h
#	/npcs/titania.cpp /npcs/titania.h
#	/npcs/true_talk_npc.cpp /npcs/true_talk_npc.h
#	/pet_control/pet_control.cpp /pet_control/pet_control.h
#	/pet_control/pet_conversations.cpp /pet_control/pet_conversations.h
#	/pet_control/pet_drag_chev.cpp /pet_control/pet_drag_chev.h
#	/pet_control/pet_element.cpp /pet_control/pet_element.h
#	/pet_control/pet_frame.cpp /pet_control/pet_frame.h
#	/pet_control/pet_gfx_element.cpp /pet_control/pet_gfx_element.h
#	/pet_control/pet_glyphs.cpp /pet_control/pet_glyphs.h
#	/pet_control/pet_graphic.cpp /pet_control/pet_graphic.h
#	/pet_control/pet_graphic2.cpp /pet_control/pet_graphic2.h
#	/pet_control/pet_inventory.cpp /pet_control/pet_inventory.h
#	/pet_control/pet_inventory_glyphs.cpp /pet_control/pet_inventory_glyphs.h
#	/pet_control/pet_leaf.cpp /pet_control/pet_leaf.h
#	/pet_control/pet_load.cpp /pet_control/pet_load.h
#	/pet_control/pet_load_save.cpp /pet_control/pet_load_save.h
#	/pet_control/pet_mode_off.cpp /pet_control/pet_mode_off.h
#	/pet_control/pet_mode_on.cpp /pet_control/pet_mode_on.h
#	/pet_control/pet_mode_panel.cpp /pet_control/pet_mode_panel.h
#	/pet_control/pet_pannel1.cpp /pet_control/pet_pannel1.h
#	/pet_control/pet_pannel2.cpp /pet_control/pet_pannel2.h
#	/pet_control/pet_pannel3.cpp /pet_control/pet_pannel3.h
#	/pet_control/pet_quit.cpp /pet_control/pet_quit.h
#	/pet_control/pet_real_life.cpp /pet_control/pet_real_life.h
#	/pet_control/pet_remote.cpp /pet_control/pet_remote.h
#	/pet_control/pet_remote_glyphs.cpp /pet_control/pet_remote_glyphs.h
#	/pet_control/pet_rooms.cpp /pet_control/pet_rooms.h
#	/pet_control/pet_rooms_glyphs.cpp /pet_control/pet_rooms_glyphs.h
#	/pet_control/pet_save.cpp /pet_control/pet_save.h
#	/pet_control/pet_section.cpp /pet_control/pet_section.h
#	/pet_control/pet_show_translation.cpp /pet_control/pet_show_translation.h
#	/pet_control/pet_slider.cpp /pet_control/pet_slider.h
#	/pet_control/pet_sound.cpp /pet_control/pet_sound.h
#	/pet_control/pet_starfield.cpp /pet_control/pet_starfield.h
#	/pet_control/pet_translation.cpp /pet_control/pet_translation.h
#	/sound/audio_buffer.cpp /sound/audio_buffer.h
#	/sound/auto_music_player.cpp /sound/auto_music_player.h
#	/sound/auto_music_player_base.cpp /sound/auto_music_player_base.h
#	/sound/auto_sound_player.cpp /sound/auto_sound_player.h
#	/sound/auto_sound_player_adsr.cpp /sound/auto_sound_player_adsr.h
#	/sound/background_sound_maker.cpp /sound/background_sound_maker.h
#	/sound/bird_song.cpp /sound/bird_song.h
#	/sound/dome_from_top_of_well.cpp /sound/dome_from_top_of_well.h
#	/sound/enter_view_toggles_other_music.cpp /sound/enter_view_toggles_other_music.h
#	/sound/gondolier_song.cpp /sound/gondolier_song.h
#	/sound/music_player.cpp /sound/music_player.h
#	/sound/music_room.cpp /sound/music_room.h
#	/sound/music_room_handler.cpp /sound/music_room_handler.h
#	/sound/music_room_instrument.cpp /sound/music_room_instrument.h
#	/sound/music_song.cpp /sound/music_song.h
#	/sound/node_auto_sound_player.cpp /sound/node_auto_sound_player.h
#	/sound/proximity.cpp /sound/proximity.h
#	/sound/qmixer.cpp /sound/qmixer.h
#	/sound/restricted_auto_music_player.cpp /sound/restricted_auto_music_player.h
#	/sound/room_auto_sound_player.cpp /sound/room_auto_sound_player.h
#	/sound/room_trigger_auto_music_player.cpp /sound/room_trigger_auto_music_player.h
#	/sound/season_noises.cpp /sound/season_noises.h
#	/sound/seasonal_music_player.cpp /sound/seasonal_music_player.h
#	/sound/sound.cpp /sound/sound.h
#	/sound/sound_manager.cpp /sound/sound_manager.h
#	/sound/titania_speech.cpp /sound/titania_speech.h
#	/sound/trigger_auto_music_player.cpp /sound/trigger_auto_music_player.h
#	/sound/view_auto_sound_player.cpp /sound/view_auto_sound_player.h
#	/sound/view_toggles_other_music.cpp /sound/view_toggles_other_music.h
#	/sound/water_lapping_sounds.cpp /sound/water_lapping_sounds.h
#	/sound/wave_file.cpp /sound/wave_file.h
#	/star_control/base_stars.cpp /star_control/base_stars.h
#	/star_control/camera.cpp /star_control/camera.h
#	/star_control/const_boundaries.cpp /star_control/const_boundaries.h
#	/star_control/constellations.cpp /star_control/constellations.h
#	/star_control/error_code.h
#	/star_control/flight_manager_base.cpp /star_control/flight_manager_base.h
#	/star_control/flight_manager_marked.cpp /star_control/flight_manager_marked.h
#	/star_control/flight_manager_unmarked.cpp /star_control/flight_manager_unmarked.h
#	/star_control/fmatrix.cpp /star_control/fmatrix.h
#	/star_control/fpoint.cpp /star_control/fpoint.h
#	/star_control/fpose.cpp /star_control/fpose.h
#	/star_control/frange.cpp /star_control/frange.h
#	/star_control/frect.cpp /star_control/frect.h
#	/star_control/fvector.cpp /star_control/fvector.h
#	/star_control/matrix_inv.h
#	/star_control/matrix_transform.cpp /star_control/matrix_transform.h
#	/star_control/motion_control.cpp /star_control/motion_control.h
#	/star_control/motion_control_marked.cpp /star_control/motion_control_marked.h
#	/star_control/motion_control_unmarked.cpp /star_control/motion_control_unmarked.h
#	/star_control/orientation_changer.cpp /star_control/orientation_changer.h
#	/star_control/star_closeup.cpp /star_control/star_closeup.h
#	/star_control/star_control.cpp /star_control/star_control.h
#	/star_control/star_crosshairs.cpp /star_control/star_crosshairs.h
#	/star_control/star_field.cpp /star_control/star_field.h
#	/star_control/star_field_base.cpp /star_control/star_field_base.h
#	/star_control/star_markers.cpp /star_control/star_markers.h
#	/star_control/star_ref.cpp /star_control/star_ref.h
#	/star_control/star_view.cpp /star_control/star_view.h
#	/star_control/surface_area.cpp /star_control/surface_area.h
#	/star_control/surface_fader.cpp /star_control/surface_fader.h
#	/star_control/viewport.cpp /star_control/viewport.h
#	/support/avi_surface.cpp /support/avi_surface.h
#	/support/credit_text.cpp /support/credit_text.h
#	/support/direct_draw.cpp /support/direct_draw.h
#	/support/direct_draw_surface.cpp /support/direct_draw_surface.h
#	/support/exe_resources.cpp /support/exe_resources.h
#	/support/files_manager.cpp /support/files_manager.h
#	/support/fixed_queue.h
#	/support/font.cpp /support/font.h
#	/support/image.cpp /support/image.h
#	/support/image_decoders.cpp /support/image_decoders.h
#	/support/mouse_cursor.cpp /support/mouse_cursor.h
#	/support/movie.cpp /support/movie.h
#	/support/movie_clip.cpp /support/movie_clip.h
#	/support/movie_event.cpp /support/movie_event.h
#	/support/movie_manager.cpp /support/movie_manager.h
#	/support/movie_range_info.cpp /support/movie_range_info.h
#	/support/rect.cpp /support/rect.h
#	/support/screen_manager.cpp /support/screen_manager.h
#	/support/simple_file.cpp /support/simple_file.h
#	/support/string.cpp /support/string.h
#	/support/string_parser.cpp /support/string_parser.h
#	/support/strings.cpp /support/strings.h
#	/support/text_cursor.cpp /support/text_cursor.h
#	/support/time_event_info.cpp /support/time_event_info.h
#	/support/transparency_surface.cpp /support/transparency_surface.h
#	/support/video_surface.cpp /support/video_surface.h
#	/true_talk/barbot_script.cpp /true_talk/barbot_script.h
#	/true_talk/bellbot_script.cpp /true_talk/bellbot_script.h
#	/true_talk/deskbot_script.cpp /true_talk/deskbot_script.h
#	/true_talk/dialogue_file.cpp /true_talk/dialogue_file.h
#	/true_talk/doorbot_script.cpp /true_talk/doorbot_script.h
#	/true_talk/liftbot_script.cpp /true_talk/liftbot_script.h
#	/true_talk/maitred_script.cpp /true_talk/maitred_script.h
#	/true_talk/parrot_script.cpp /true_talk/parrot_script.h
#	/true_talk/script_handler.cpp /true_talk/script_handler.h
#	/true_talk/script_support.cpp /true_talk/script_support.h
#	/true_talk/succubus_script.cpp /true_talk/succubus_script.h
#	/true_talk/title_engine.cpp /true_talk/title_engine.h
#	/true_talk/true_talk_manager.cpp /true_talk/true_talk_manager.h
#	/true_talk/tt_action.cpp /true_talk/tt_action.h
#	/true_talk/tt_adj.cpp /true_talk/tt_adj.h
#	/true_talk/tt_concept.cpp /true_talk/tt_concept.h
#	/true_talk/tt_concept_node.cpp /true_talk/tt_concept_node.h
#	/true_talk/tt_hist.cpp /true_talk/tt_hist.h
#	/true_talk/tt_major_word.cpp /true_talk/tt_major_word.h
#	/true_talk/tt_node.cpp /true_talk/tt_node.h
#	/true_talk/tt_npc_script.cpp /true_talk/tt_npc_script.h
#	/true_talk/tt_parser.cpp /true_talk/tt_parser.h
#	/true_talk/tt_picture.cpp /true_talk/tt_picture.h
#	/true_talk/tt_pronoun.cpp /true_talk/tt_pronoun.h
#	/true_talk/tt_quotes.cpp /true_talk/tt_quotes.h
#	/true_talk/tt_quotes_tree.cpp /true_talk/tt_quotes_tree.h
#	/true_talk/tt_response.cpp /true_talk/tt_response.h
#	/true_talk/tt_room_script.cpp /true_talk/tt_room_script.h
#	/true_talk/tt_script_base.cpp /true_talk/tt_script_base.h
#	/true_talk/tt_scripts.cpp /true_talk/tt_scripts.h
#	/true_talk/tt_sentence.cpp /true_talk/tt_sentence.h
#	/true_talk/tt_sentence_node.cpp /true_talk/tt_sentence_node.h
#	/true_talk/tt_string.cpp /true_talk/tt_string.h
#	/true_talk/tt_string_node.cpp /true_talk/tt_string_node.h
#	/true_talk/tt_synonym.cpp /true_talk/tt_synonym.h
#	/true_talk/tt_talker.cpp /true_talk/tt_talker.h
#	/true_talk/tt_title_script.cpp /true_talk/tt_title_script.h
#	/true_talk/tt_vocab.cpp /true_talk/tt_vocab.h
#	/true_talk/tt_word.cpp /true_talk/tt_word.h
#	/continue_save_dialog.cpp /continue_save_dialog.h
#	/debugger.cpp /debugger.h
#	/detection.h
#	/detection_tables.h
#	/events.cpp /events.h
#	/game_location.cpp /game_location.h
#	/game_manager.cpp /game_manager.h
#	/game_state.cpp /game_state.h
#	/game_view.cpp /game_view.h
#	/input_handler.cpp /input_handler.h
#	/input_translator.cpp /input_translator.h
#	/main_game_window.cpp /main_game_window.h
#	/metaengine.cpp
#	/module.mk
#	/room_flags.cpp /room_flags.h
#	/titanic.cpp /titanic.h
#	/translation.h
#)
#
#add_engine(toltecs
#	/animation.cpp /animation.h
#	/console.cpp /console.h
#	/detection.h
#	/menu.cpp /menu.h
#	/metaengine.cpp
#	/microtiles.cpp /microtiles.h
#	/module.mk
#	/movie.cpp /movie.h
#	/music.cpp /music.h
#	/palette.cpp /palette.h
#	/render.cpp /render.h
#	/resource.cpp /resource.h
#	/saveload.cpp
#	/screen.cpp /screen.h
#	/script.cpp /script.h
#	/segmap.cpp /segmap.h
#	/sound.cpp /sound.h
#	/sprite.cpp
#	/toltecs.cpp /toltecs.h
#)
#
#add_engine(tony
#	/mpal/expr.cpp /mpal/expr.h
#	/mpal/loadmpc.cpp /mpal/loadmpc.h
#	/mpal/lzo.cpp /mpal/lzo.h
#	/mpal/memory.cpp /mpal/memory.h
#	/mpal/mpal.cpp /mpal/mpal.h
#	/mpal/mpaldll.h
#	/mpal/mpalutils.cpp /mpal/mpalutils.h
#	/custom.cpp /custom.h
#	/debugger.cpp /debugger.h
#	/detection.h
#	/detection_tables.h
#	/font.cpp /font.h
#	/game.cpp /game.h
#	/gfxcore.cpp /gfxcore.h
#	/gfxengine.cpp /gfxengine.h
#	/globals.cpp /globals.h
#	/input.cpp /input.h
#	/inventory.cpp /inventory.h
#	/loc.cpp /loc.h
#	/metaengine.cpp
#	/module.mk
#	/resid.h
#	/sound.cpp /sound.h
#	/tony.cpp /tony.h
#	/tonychar.cpp /tonychar.h
#	/utils.cpp /utils.h
#	/window.cpp /window.h
#)
#
#add_engine(toon
#	/anim.cpp /anim.h
#	/audio.cpp /audio.h
#	/character.cpp /character.h
#	/console.cpp /console.h
#	/conversation.cpp /conversation.h
#	/drew.cpp /drew.h
#	/flux.cpp /flux.h
#	/font.cpp /font.h
#	/hotspot.cpp /hotspot.h
#	/metaengine.cpp
#	/module.mk
#	/movie.cpp /movie.h
#	/path.cpp /path.h
#	/picture.cpp /picture.h
#	/resource.cpp /resource.h
#	/script.cpp /script.h
#	/script_func.cpp /script_func.h
#	/state.cpp /state.h
#	/subtitles.cpp /subtitles.h
#	/text.cpp /text.h
#	/tools.cpp /tools.h
#	/toon.cpp /toon.h
#)
#
#add_engine(touche
#	/console.cpp /console.h
#	/graphics.cpp /graphics.h
#	/menu.cpp
#	/metaengine.cpp
#	/midi.cpp /midi.h
#	/module.mk
#	/opcodes.cpp
#	/resource.cpp
#	/saveload.cpp
#	/staticres.cpp
#	/touche.cpp /touche.h
#)
#
#add_engine(trecision
#	/actor.cpp /actor.h
#	/animmanager.cpp /animmanager.h
#	/animtype.cpp /animtype.h
#	/console.cpp /console.h
#	/defines.h
#	/dialog.cpp /dialog.h
#	/fastfile.cpp /fastfile.h
#	/graphics.cpp /graphics.h
#	/inventory.cpp
#	/logic.cpp /logic.h
#	/metaengine.cpp
#	/module.mk
#	/pathfinding3d.cpp /pathfinding3d.h
#	/renderer3d.cpp /renderer3d.h
#	/resource.cpp
#	/saveload.cpp
#	/scheduler.cpp /scheduler.h
#	/script.cpp
#	/sound.cpp /sound.h
#	/struct.cpp /struct.h
#	/text.cpp /text.h
#	/trecision.cpp /trecision.h
#	/utils.cpp
#	/video.cpp /video.h
#)
#
#add_engine(tsage
#	/blue_force/blueforce_dialogs.cpp /blue_force/blueforce_dialogs.h
#	/blue_force/blueforce_logic.cpp /blue_force/blueforce_logic.h
#	/blue_force/blueforce_scenes0.cpp /blue_force/blueforce_scenes0.h
#	/blue_force/blueforce_scenes1.cpp /blue_force/blueforce_scenes1.h
#	/blue_force/blueforce_scenes2.cpp /blue_force/blueforce_scenes2.h
#	/blue_force/blueforce_scenes3.cpp /blue_force/blueforce_scenes3.h
#	/blue_force/blueforce_scenes4.cpp /blue_force/blueforce_scenes4.h
#	/blue_force/blueforce_scenes5.cpp /blue_force/blueforce_scenes5.h
#	/blue_force/blueforce_scenes6.cpp /blue_force/blueforce_scenes6.h
#	/blue_force/blueforce_scenes7.cpp /blue_force/blueforce_scenes7.h
#	/blue_force/blueforce_scenes8.cpp /blue_force/blueforce_scenes8.h
#	/blue_force/blueforce_scenes9.cpp /blue_force/blueforce_scenes9.h
#	/blue_force/blueforce_speakers.cpp /blue_force/blueforce_speakers.h
#	/ringworld/ringworld_demo.cpp /ringworld/ringworld_demo.h
#	/ringworld/ringworld_dialogs.cpp /ringworld/ringworld_dialogs.h
#	/ringworld/ringworld_logic.cpp /ringworld/ringworld_logic.h
#	/ringworld/ringworld_scenes1.cpp /ringworld/ringworld_scenes1.h
#	/ringworld/ringworld_scenes10.cpp /ringworld/ringworld_scenes10.h
#	/ringworld/ringworld_scenes2.cpp /ringworld/ringworld_scenes2.h
#	/ringworld/ringworld_scenes3.cpp /ringworld/ringworld_scenes3.h
#	/ringworld/ringworld_scenes4.cpp /ringworld/ringworld_scenes4.h
#	/ringworld/ringworld_scenes5.cpp /ringworld/ringworld_scenes5.h
#	/ringworld/ringworld_scenes6.cpp /ringworld/ringworld_scenes6.h
#	/ringworld/ringworld_scenes8.cpp /ringworld/ringworld_scenes8.h
#	/ringworld/ringworld_speakers.cpp /ringworld/ringworld_speakers.h
#	/ringworld2/ringworld2_airduct.cpp /ringworld2/ringworld2_airduct.h
#	/ringworld2/ringworld2_dialogs.cpp /ringworld2/ringworld2_dialogs.h
#	/ringworld2/ringworld2_logic.cpp /ringworld2/ringworld2_logic.h
#	/ringworld2/ringworld2_outpost.cpp /ringworld2/ringworld2_outpost.h
#	/ringworld2/ringworld2_scenes0.cpp /ringworld2/ringworld2_scenes0.h
#	/ringworld2/ringworld2_scenes1.cpp /ringworld2/ringworld2_scenes1.h
#	/ringworld2/ringworld2_scenes2.cpp /ringworld2/ringworld2_scenes2.h
#	/ringworld2/ringworld2_scenes3.cpp /ringworld2/ringworld2_scenes3.h
#	/ringworld2/ringworld2_speakers.cpp /ringworld2/ringworld2_speakers.h
#	/ringworld2/ringworld2_vampire.cpp /ringworld2/ringworld2_vampire.h
#	/sherlock/sherlock_logo.cpp /sherlock/sherlock_logo.h
#	/converse.cpp /converse.h
#	/core.cpp /core.h
#	/debugger.cpp /debugger.h
#	/detection.h
#	/detection_tables.h
#	/dialogs.cpp /dialogs.h
#	/events.cpp /events.h
#	/globals.cpp /globals.h
#	/graphics.cpp /graphics.h
#	/metaengine.cpp
#	/module.mk
#	/resources.cpp /resources.h
#	/saveload.cpp /saveload.h
#	/scenes.cpp /scenes.h
#	/screen.cpp /screen.h
#	/sound.cpp /sound.h
#	/staticres.cpp /staticres.h
#	/tsage.cpp /tsage.h
#	/user_interface.cpp /user_interface.h
#)
#
#add_engine(tucker
#	/console.cpp /console.h
#	/detection.h
#	/graphics.cpp /graphics.h
#	/locations.cpp
#	/metaengine.cpp
#	/module.mk
#	/resource.cpp
#	/saveload.cpp
#	/sequences.cpp
#	/staticres.cpp
#	/tucker.cpp /tucker.h
#)
#
#add_engine(twine
#	/audio/music.cpp /audio/music.h
#	/audio/sound.cpp /audio/sound.h
#	/debugger/console.cpp /debugger/console.h
#	/debugger/debug.cpp /debugger/debug.h
#	/debugger/debug_grid.cpp /debugger/debug_grid.h
#	/debugger/debug_scene.cpp /debugger/debug_scene.h
#	/menu/interface.cpp /menu/interface.h
#	/menu/menu.cpp /menu/menu.h
#	/menu/menuoptions.cpp /menu/menuoptions.h
#	/parser/anim.cpp /parser/anim.h
#	/parser/blocklibrary.cpp /parser/blocklibrary.h
#	/parser/body.cpp /parser/body.h
#	/parser/bodytypes.h
#	/parser/entity.cpp /parser/entity.h
#	/parser/holomap.cpp /parser/holomap.h
#	/parser/parser.cpp /parser/parser.h
#	/parser/sprite.cpp /parser/sprite.h
#	/parser/text.cpp /parser/text.h
#	/renderer/redraw.cpp /renderer/redraw.h
#	/renderer/renderer.cpp /renderer/renderer.h
#	/renderer/screens.cpp /renderer/screens.h
#	/renderer/shadeangletab.h
#	/resources/hqr.cpp /resources/hqr.h
#	/resources/lzss.cpp /resources/lzss.h
#	/resources/resources.cpp /resources/resources.h
#	/scene/actor.cpp /scene/actor.h
#	/scene/animations.cpp /scene/animations.h
#	/scene/collision.cpp /scene/collision.h
#	/scene/extra.cpp /scene/extra.h
#	/scene/gamestate.cpp /scene/gamestate.h
#	/scene/grid.cpp /scene/grid.h
#	/scene/movements.cpp /scene/movements.h
#	/scene/scene.cpp /scene/scene.h
#	/script/script_life_v1.cpp /script/script_life_v1.h
#	/script/script_move_v1.cpp /script/script_move_v1.h
#	/achievements_tables.h
#	/detection.h
#	/holomap.cpp /holomap.h
#	/input.cpp /input.h
#	/metaengine.cpp
#	/module.mk
#	/movies.cpp /movies.h
#	/shared.cpp /shared.h
#	/slideshow.cpp /slideshow.h
#	/text.cpp /text.h
#	/twine.cpp /twine.h
#)
#
#add_engine(ultima
#	/nuvie/actors/actor.cpp /nuvie/actors/actor.h
#	/nuvie/actors/actor_manager.cpp /nuvie/actors/actor_manager.h
#	/nuvie/actors/md_actor.cpp /nuvie/actors/md_actor.h
#	/nuvie/actors/se_actor.cpp /nuvie/actors/se_actor.h
#	/nuvie/actors/u6_actor.cpp /nuvie/actors/u6_actor.h
#	/nuvie/actors/u6_actor_types.h
#	/nuvie/actors/u6_work_types.h
#	/nuvie/actors/wou_actor.cpp /nuvie/actors/wou_actor.h
#	/nuvie/conf/config_node.h
#	/nuvie/conf/configuration.cpp /nuvie/conf/configuration.h
#	/nuvie/conf/misc.cpp /nuvie/conf/misc.h
#	/nuvie/core/anim_manager.cpp /nuvie/core/anim_manager.h
#	/nuvie/core/book.cpp /nuvie/core/book.h
#	/nuvie/core/converse.cpp /nuvie/core/converse.h
#	/nuvie/core/converse_interpret.cpp /nuvie/core/converse_interpret.h
#	/nuvie/core/converse_speech.cpp /nuvie/core/converse_speech.h
#	/nuvie/core/cursor.cpp /nuvie/core/cursor.h
#	/nuvie/core/debug.cpp
#	/nuvie/core/debugger.cpp /nuvie/core/debugger.h
#	/nuvie/core/effect.cpp /nuvie/core/effect.h
#	/nuvie/core/effect_manager.cpp /nuvie/core/effect_manager.h
#	/nuvie/core/egg_manager.cpp /nuvie/core/egg_manager.h
#	/nuvie/core/events.cpp /nuvie/core/events.h
#	/nuvie/core/game.cpp /nuvie/core/game.h
#	/nuvie/core/game_clock.cpp /nuvie/core/game_clock.h
#	/nuvie/core/look.cpp /nuvie/core/look.h
#	/nuvie/core/magic.cpp /nuvie/core/magic.h
#	/nuvie/core/map.cpp /nuvie/core/map.h
#	/nuvie/core/nuvie_defs.cpp /nuvie/core/nuvie_defs.h
#	/nuvie/core/obj.cpp /nuvie/core/obj.h
#	/nuvie/core/obj_manager.cpp /nuvie/core/obj_manager.h
#	/nuvie/core/party.cpp /nuvie/core/party.h
#	/nuvie/core/player.cpp /nuvie/core/player.h
#	/nuvie/core/tile_manager.cpp /nuvie/core/tile_manager.h
#	/nuvie/core/timed_event.cpp /nuvie/core/timed_event.h
#	/nuvie/core/u6_objects.h
#	/nuvie/core/weather.cpp /nuvie/core/weather.h
#	/nuvie/files/nuvie_bmp_file.cpp /nuvie/files/nuvie_bmp_file.h
#	/nuvie/files/nuvie_file_list.cpp /nuvie/files/nuvie_file_list.h
#	/nuvie/files/nuvie_io.cpp /nuvie/files/nuvie_io.h
#	/nuvie/files/nuvie_io_file.cpp /nuvie/files/nuvie_io_file.h
#	/nuvie/files/tmx_map.cpp /nuvie/files/tmx_map.h
#	/nuvie/files/u6_bmp.cpp /nuvie/files/u6_bmp.h
#	/nuvie/files/u6_lib_n.cpp /nuvie/files/u6_lib_n.h
#	/nuvie/files/u6_lzw.cpp /nuvie/files/u6_lzw.h
#	/nuvie/files/u6_shape.cpp /nuvie/files/u6_shape.h
#	/nuvie/files/utils.cpp /nuvie/files/utils.h
#	/nuvie/fonts/bmp_font.cpp /nuvie/fonts/bmp_font.h
#	/nuvie/fonts/conv_font.cpp /nuvie/fonts/conv_font.h
#	/nuvie/fonts/font.cpp /nuvie/fonts/font.h
#	/nuvie/fonts/font_manager.cpp /nuvie/fonts/font_manager.h
#	/nuvie/fonts/u6_font.cpp /nuvie/fonts/u6_font.h
#	/nuvie/fonts/wou_font.cpp /nuvie/fonts/wou_font.h
#	/nuvie/gui/widgets/background.cpp /nuvie/gui/widgets/background.h
#	/nuvie/gui/widgets/command_bar.cpp /nuvie/gui/widgets/command_bar.h
#	/nuvie/gui/widgets/command_bar_new_ui.cpp /nuvie/gui/widgets/command_bar_new_ui.h
#	/nuvie/gui/widgets/console.cpp /nuvie/gui/widgets/console.h
#	/nuvie/gui/widgets/converse_gump.cpp /nuvie/gui/widgets/converse_gump.h
#	/nuvie/gui/widgets/converse_gump_wou.cpp /nuvie/gui/widgets/converse_gump_wou.h
#	/nuvie/gui/widgets/fps_counter.cpp /nuvie/gui/widgets/fps_counter.h
#	/nuvie/gui/widgets/gui_widget.cpp /nuvie/gui/widgets/gui_widget.h
#	/nuvie/gui/widgets/map_window.cpp /nuvie/gui/widgets/map_window.h
#	/nuvie/gui/widgets/msg_scroll.cpp /nuvie/gui/widgets/msg_scroll.h
#	/nuvie/gui/widgets/msg_scroll_new_ui.cpp /nuvie/gui/widgets/msg_scroll_new_ui.h
#	/nuvie/gui/gui.cpp /nuvie/gui/gui.h
#	/nuvie/gui/gui_area.cpp /nuvie/gui/gui_area.h
#	/nuvie/gui/gui_button.cpp /nuvie/gui/gui_button.h
#	/nuvie/gui/gui_callback.h
#	/nuvie/gui/gui_console.cpp /nuvie/gui/gui_console.h
#	/nuvie/gui/gui_dialog.cpp /nuvie/gui/gui_dialog.h
#	/nuvie/gui/gui_drag_area.h
#	/nuvie/gui/gui_drag_manager.cpp /nuvie/gui/gui_drag_manager.h
#	/nuvie/gui/gui_font.cpp /nuvie/gui/gui_font.h
#	/nuvie/gui/gui_load_image.cpp /nuvie/gui/gui_load_image.h
#	/nuvie/gui/gui_scroll_bar.cpp /nuvie/gui/gui_scroll_bar.h
#	/nuvie/gui/gui_scroller.cpp /nuvie/gui/gui_scroller.h
#	/nuvie/gui/gui_status.h
#	/nuvie/gui/gui_text.cpp /nuvie/gui/gui_text.h
#	/nuvie/gui/gui_text_input.cpp /nuvie/gui/gui_text_input.h
#	/nuvie/gui/gui_text_toggle_button.cpp /nuvie/gui/gui_text_toggle_button.h
#	/nuvie/gui/gui_types.h
#	/nuvie/gui/gui_yes_no_dialog.cpp /nuvie/gui/gui_yes_no_dialog.h
#	/nuvie/gui/the_font.h
#	/nuvie/keybinding/key_actions.cpp /nuvie/keybinding/key_actions.h
#	/nuvie/keybinding/keys.cpp /nuvie/keybinding/keys.h
#	/nuvie/keybinding/keys_enum.h
#	/nuvie/menus/audio_dialog.cpp /nuvie/menus/audio_dialog.h
#	/nuvie/menus/cheats_dialog.cpp /nuvie/menus/cheats_dialog.h
#	/nuvie/menus/game_menu_dialog.cpp /nuvie/menus/game_menu_dialog.h
#	/nuvie/menus/gameplay_dialog.cpp /nuvie/menus/gameplay_dialog.h
#	/nuvie/menus/input_dialog.cpp /nuvie/menus/input_dialog.h
#	/nuvie/menus/video_dialog.cpp /nuvie/menus/video_dialog.h
#	/nuvie/misc/actor_list.h
#	/nuvie/misc/call_back.h
#	/nuvie/misc/iavl_tree.cpp /nuvie/misc/iavl_tree.h
#	/nuvie/misc/map_entity.h
#	/nuvie/misc/sdl_compat.cpp /nuvie/misc/sdl_compat.h
#	/nuvie/misc/u6_line_walker.cpp /nuvie/misc/u6_line_walker.h
#	/nuvie/misc/u6_list.cpp
#	/nuvie/misc/u6_llist.h
#	/nuvie/misc/u6_misc.cpp /nuvie/misc/u6_misc.h
#	/nuvie/pathfinder/actor_path_finder.cpp /nuvie/pathfinder/actor_path_finder.h
#	/nuvie/pathfinder/astar_path.cpp /nuvie/pathfinder/astar_path.h
#	/nuvie/pathfinder/combat_path_finder.cpp /nuvie/pathfinder/combat_path_finder.h
#	/nuvie/pathfinder/dir_finder.cpp /nuvie/pathfinder/dir_finder.h
#	/nuvie/pathfinder/party_path_finder.cpp /nuvie/pathfinder/party_path_finder.h
#	/nuvie/pathfinder/path.cpp /nuvie/pathfinder/path.h
#	/nuvie/pathfinder/path_finder.cpp /nuvie/pathfinder/path_finder.h
#	/nuvie/pathfinder/sched_path_finder.cpp /nuvie/pathfinder/sched_path_finder.h
#	/nuvie/pathfinder/seek_path.cpp /nuvie/pathfinder/seek_path.h
#	/nuvie/pathfinder/u6_astar_path.cpp /nuvie/pathfinder/u6_astar_path.h
#	/nuvie/portraits/portrait.cpp /nuvie/portraits/portrait.h
#	/nuvie/portraits/portrait_md.cpp /nuvie/portraits/portrait_md.h
#	/nuvie/portraits/portrait_se.cpp /nuvie/portraits/portrait_se.h
#	/nuvie/portraits/portrait_u6.cpp /nuvie/portraits/portrait_u6.h
#	/nuvie/save/obj_list.h
#	/nuvie/save/save_game.cpp /nuvie/save/save_game.h
#	/nuvie/screen/dither.cpp /nuvie/screen/dither.h
#	/nuvie/screen/game_palette.cpp /nuvie/screen/game_palette.h
#	/nuvie/screen/scale.cpp /nuvie/screen/scale.h
#	/nuvie/screen/screen.cpp /nuvie/screen/screen.h
#	/nuvie/screen/surface.cpp /nuvie/screen/surface.h
#	/nuvie/script/script.cpp /nuvie/script/script.h
#	/nuvie/script/script_actor.cpp /nuvie/script/script_actor.h
#	/nuvie/script/script_cutscene.cpp /nuvie/script/script_cutscene.h
#	/nuvie/sound/adplug/adplug_player.cpp /nuvie/sound/adplug/adplug_player.h
#	/nuvie/sound/adplug/emu_opl.cpp /nuvie/sound/adplug/emu_opl.h
#	/nuvie/sound/adplug/fm_opl.cpp /nuvie/sound/adplug/fm_opl.h
#	/nuvie/sound/adplug/mid.cpp /nuvie/sound/adplug/mid.h
#	/nuvie/sound/adplug/opl.h
#	/nuvie/sound/adplug/opl_class.cpp /nuvie/sound/adplug/opl_class.h
#	/nuvie/sound/adplug/silent_opl.h
#	/nuvie/sound/adplug/u6m.cpp /nuvie/sound/adplug/u6m.h
#	/nuvie/sound/decoder/adlib_sfx_stream.cpp /nuvie/sound/decoder/adlib_sfx_stream.h
#	/nuvie/sound/decoder/fm_towns_decoder_stream.cpp /nuvie/sound/decoder/fm_towns_decoder_stream.h
#	/nuvie/sound/decoder/pc_speaker.cpp /nuvie/sound/decoder/pc_speaker.h
#	/nuvie/sound/decoder/pc_speaker_stream.cpp /nuvie/sound/decoder/pc_speaker_stream.h
#	/nuvie/sound/decoder/random_collection_audio_stream.cpp /nuvie/sound/decoder/random_collection_audio_stream.h
#	/nuvie/sound/decoder/u6_adplug_decoder_stream.cpp /nuvie/sound/decoder/u6_adplug_decoder_stream.h
#	/nuvie/sound/adlib_sfx_manager.cpp /nuvie/sound/adlib_sfx_manager.h
#	/nuvie/sound/custom_sfx_manager.cpp /nuvie/sound/custom_sfx_manager.h
#	/nuvie/sound/origin_fx_adib_driver.cpp /nuvie/sound/origin_fx_adib_driver.h
#	/nuvie/sound/pc_speaker_sfx_manager.cpp /nuvie/sound/pc_speaker_sfx_manager.h
#	/nuvie/sound/sfx.h
#	/nuvie/sound/sfx_manager.h
#	/nuvie/sound/song.cpp /nuvie/sound/song.h
#	/nuvie/sound/song_adplug.cpp /nuvie/sound/song_adplug.h
#	/nuvie/sound/sound.h
#	/nuvie/sound/sound_manager.cpp /nuvie/sound/sound_manager.h
#	/nuvie/sound/towns_sfx_manager.cpp /nuvie/sound/towns_sfx_manager.h
#	/nuvie/usecode/u6_object_types.h
#	/nuvie/usecode/u6_usecode.cpp /nuvie/usecode/u6_usecode.h
#	/nuvie/usecode/usecode.cpp /nuvie/usecode/usecode.h
#	/nuvie/views/actor_view.cpp /nuvie/views/actor_view.h
#	/nuvie/views/container_view_gump.cpp /nuvie/views/container_view_gump.h
#	/nuvie/views/container_widget.cpp /nuvie/views/container_widget.h
#	/nuvie/views/container_widget_gump.cpp /nuvie/views/container_widget_gump.h
#	/nuvie/views/doll_view_gump.cpp /nuvie/views/doll_view_gump.h
#	/nuvie/views/doll_widget.cpp /nuvie/views/doll_widget.h
#	/nuvie/views/draggable_view.cpp /nuvie/views/draggable_view.h
#	/nuvie/views/inventory_font.h
#	/nuvie/views/inventory_message.h
#	/nuvie/views/inventory_view.cpp /nuvie/views/inventory_view.h
#	/nuvie/views/inventory_widget.cpp /nuvie/views/inventory_widget.h
#	/nuvie/views/map_editor_view.cpp /nuvie/views/map_editor_view.h
#	/nuvie/views/md_sky_strip_widget.cpp /nuvie/views/md_sky_strip_widget.h
#	/nuvie/views/party_view.cpp /nuvie/views/party_view.h
#	/nuvie/views/portrait_view.cpp /nuvie/views/portrait_view.h
#	/nuvie/views/portrait_view_gump.cpp /nuvie/views/portrait_view_gump.h
#	/nuvie/views/scroll_view_gump.cpp /nuvie/views/scroll_view_gump.h
#	/nuvie/views/scroll_widget_gump.cpp /nuvie/views/scroll_widget_gump.h
#	/nuvie/views/sign_view_gump.cpp /nuvie/views/sign_view_gump.h
#	/nuvie/views/spell_view.cpp /nuvie/views/spell_view.h
#	/nuvie/views/spell_view_gump.cpp /nuvie/views/spell_view_gump.h
#	/nuvie/views/sun_moon_ribbon.cpp /nuvie/views/sun_moon_ribbon.h
#	/nuvie/views/sun_moon_strip_widget.cpp /nuvie/views/sun_moon_strip_widget.h
#	/nuvie/views/vew.cpp
#	/nuvie/views/view.h
#	/nuvie/views/view_manager.cpp /nuvie/views/view_manager.h
#	/nuvie/meta_engine.cpp /nuvie/meta_engine.h
#	/nuvie/nuvie.cpp /nuvie/nuvie.h
#	/shared/actions/action.cpp /shared/actions/action.h
#	/shared/actions/huh.cpp /shared/actions/huh.h
#	/shared/actions/pass.cpp /shared/actions/pass.h
#	/shared/conf/conf_serializer.h
#	/shared/conf/xml_node.cpp /shared/conf/xml_node.h
#	/shared/conf/xml_tree.cpp /shared/conf/xml_tree.h
#	/shared/core/base_object.cpp /shared/core/base_object.h
#	/shared/core/character.cpp /shared/core/character.h
#	/shared/core/file.cpp /shared/core/file.h
#	/shared/core/game_state.h
#	/shared/core/lzw.cpp /shared/core/lzw.h
#	/shared/core/map.cpp /shared/core/map.h
#	/shared/core/message_target.cpp /shared/core/message_target.h
#	/shared/core/mouse_cursor.cpp /shared/core/mouse_cursor.h
#	/shared/core/named_item.cpp /shared/core/named_item.h
#	/shared/core/party.cpp /shared/core/party.h
#	/shared/core/rect.h
#	/shared/core/str.cpp /shared/core/str.h
#	/shared/core/tree_item.cpp /shared/core/tree_item.h
#	/shared/core/utils.cpp /shared/core/utils.h
#	/shared/core/widgets.cpp /shared/core/widgets.h
#	/shared/early/font_resources.cpp /shared/early/font_resources.h
#	/shared/early/game.cpp /shared/early/game.h
#	/shared/early/game_base.cpp /shared/early/game_base.h
#	/shared/early/ultima_early.cpp /shared/early/ultima_early.h
#	/shared/engine/data_archive.cpp /shared/engine/data_archive.h
#	/shared/engine/debugger.cpp /shared/engine/debugger.h
#	/shared/engine/events.cpp /shared/engine/events.h
#	/shared/engine/input_handler.cpp /shared/engine/input_handler.h
#	/shared/engine/input_translator.cpp /shared/engine/input_translator.h
#	/shared/engine/messages.cpp /shared/engine/messages.h
#	/shared/engine/resources.cpp /shared/engine/resources.h
#	/shared/engine/ultima.cpp /shared/engine/ultima.h
#	/shared/gfx/bitmap.cpp /shared/gfx/bitmap.h
#	/shared/gfx/character_input.cpp /shared/gfx/character_input.h
#	/shared/gfx/dungeon_surface.cpp /shared/gfx/dungeon_surface.h
#	/shared/gfx/font.cpp /shared/gfx/font.h
#	/shared/gfx/info.cpp /shared/gfx/info.h
#	/shared/gfx/popup.cpp /shared/gfx/popup.h
#	/shared/gfx/screen.cpp /shared/gfx/screen.h
#	/shared/gfx/sprites.cpp /shared/gfx/sprites.h
#	/shared/gfx/text_cursor.cpp /shared/gfx/text_cursor.h
#	/shared/gfx/text_input.cpp /shared/gfx/text_input.h
#	/shared/gfx/viewport_dungeon.cpp /shared/gfx/viewport_dungeon.h
#	/shared/gfx/viewport_map.cpp /shared/gfx/viewport_map.h
#	/shared/gfx/visual_container.cpp /shared/gfx/visual_container.h
#	/shared/gfx/visual_item.cpp /shared/gfx/visual_item.h
#	/shared/gfx/visual_surface.cpp /shared/gfx/visual_surface.h
#	/shared/maps/creature.cpp /shared/maps/creature.h
#	/shared/maps/dungeon_creature.h
#	/shared/maps/dungeon_widget.h
#	/shared/maps/map.cpp /shared/maps/map.h
#	/shared/maps/map_base.cpp /shared/maps/map_base.h
#	/shared/maps/map_tile.cpp /shared/maps/map_tile.h
#	/shared/maps/map_widget.cpp /shared/maps/map_widget.h
#	/shared/std/containers.h
#	/shared/std/string.cpp /shared/std/string.h
#
#	/ultima0/core/resources.cpp /ultima0/core/resources.h
#	/ultima0/game.cpp /ultima0/game.h
#	/ultima0/resources.cpp /ultima0/resources.h
#	/ultima1/actions/action.cpp /ultima1/actions/action.h
#	/ultima1/actions/attack.cpp /ultima1/actions/attack.h
#	/ultima1/actions/cast.h
#	/ultima1/actions/map_action.h
#	/ultima1/actions/move.cpp /ultima1/actions/move.h
#	/ultima1/actions/quit.cpp /ultima1/actions/quit.h
#	/ultima1/actions/ready.cpp /ultima1/actions/ready.h
#	/ultima1/actions/stats.cpp /ultima1/actions/stats.h
#	/ultima1/core/debugger.cpp /ultima1/core/debugger.h
#	/ultima1/core/game_state.h
#	/ultima1/core/messages.h
#	/ultima1/core/party.cpp /ultima1/core/party.h
#	/ultima1/core/quests.cpp /ultima1/core/quests.h
#	/ultima1/core/resources.cpp /ultima1/core/resources.h
#	/ultima1/core/widget_player.h
#	/ultima1/maps/map.cpp /ultima1/maps/map.h
#	/ultima1/maps/map_base.cpp /ultima1/maps/map_base.h
#	/ultima1/maps/map_city_castle.cpp /ultima1/maps/map_city_castle.h
#	/ultima1/maps/map_dungeon.cpp /ultima1/maps/map_dungeon.h
#	/ultima1/maps/map_overworld.cpp /ultima1/maps/map_overworld.h
#	/ultima1/maps/map_tile.cpp /ultima1/maps/map_tile.h
#	/ultima1/spells/blink.cpp /ultima1/spells/blink.h
#	/ultima1/spells/create.cpp /ultima1/spells/create.h
#	/ultima1/spells/destroy.cpp /ultima1/spells/destroy.h
#	/ultima1/spells/kill_magic_missile.cpp /ultima1/spells/kill_magic_missile.h
#	/ultima1/spells/ladder_down.cpp /ultima1/spells/ladder_down.h
#	/ultima1/spells/ladder_up.cpp /ultima1/spells/ladder_up.h
#	/ultima1/spells/open_unlock.cpp /ultima1/spells/open_unlock.h
#	/ultima1/spells/prayer.cpp /ultima1/spells/prayer.h
#	/ultima1/spells/spell.cpp /ultima1/spells/spell.h
#	/ultima1/spells/steal.cpp /ultima1/spells/steal.h
#	/ultima1/u1dialogs/armoury.cpp /ultima1/u1dialogs/armoury.h
#	/ultima1/u1dialogs/buy_sell_dialog.cpp /ultima1/u1dialogs/buy_sell_dialog.h
#	/ultima1/u1dialogs/combat.cpp /ultima1/u1dialogs/combat.h
#	/ultima1/u1dialogs/dialog.cpp /ultima1/u1dialogs/dialog.h
#	/ultima1/u1dialogs/drop.cpp /ultima1/u1dialogs/drop.h
#	/ultima1/u1dialogs/full_screen_dialog.cpp /ultima1/u1dialogs/full_screen_dialog.h
#	/ultima1/u1dialogs/grocery.cpp /ultima1/u1dialogs/grocery.h
#	/ultima1/u1dialogs/king.cpp /ultima1/u1dialogs/king.h
#	/ultima1/u1dialogs/magic.cpp /ultima1/u1dialogs/magic.h
#	/ultima1/u1dialogs/ready.cpp /ultima1/u1dialogs/ready.h
#	/ultima1/u1dialogs/stats.cpp /ultima1/u1dialogs/stats.h
#	/ultima1/u1dialogs/tavern.cpp /ultima1/u1dialogs/tavern.h
#	/ultima1/u1dialogs/transports.cpp /ultima1/u1dialogs/transports.h
#	/ultima1/u1dialogs/weaponry.cpp /ultima1/u1dialogs/weaponry.h
#	/ultima1/u1gfx/drawing_support.cpp /ultima1/u1gfx/drawing_support.h
#	/ultima1/u1gfx/info.cpp /ultima1/u1gfx/info.h
#	/ultima1/u1gfx/sprites.cpp /ultima1/u1gfx/sprites.h
#	/ultima1/u1gfx/status.cpp /ultima1/u1gfx/status.h
#	/ultima1/u1gfx/text_cursor.cpp /ultima1/u1gfx/text_cursor.h
#	/ultima1/u1gfx/view_char_gen.cpp /ultima1/u1gfx/view_char_gen.h
#	/ultima1/u1gfx/view_game.cpp /ultima1/u1gfx/view_game.h
#	/ultima1/u1gfx/view_title.cpp /ultima1/u1gfx/view_title.h
#	/ultima1/u1gfx/viewport_dungeon.cpp /ultima1/u1gfx/viewport_dungeon.h
#	/ultima1/u1gfx/viewport_map.cpp /ultima1/u1gfx/viewport_map.h
#	/ultima1/u6gfx/game_view.cpp /ultima1/u6gfx/game_view.h
#	/ultima1/widgets/attack_effect.cpp /ultima1/widgets/attack_effect.h
#	/ultima1/widgets/bard.cpp /ultima1/widgets/bard.h
#	/ultima1/widgets/dungeon_chest.cpp /ultima1/widgets/dungeon_chest.h
#	/ultima1/widgets/dungeon_coffin.cpp /ultima1/widgets/dungeon_coffin.h
#	/ultima1/widgets/dungeon_item.cpp /ultima1/widgets/dungeon_item.h
#	/ultima1/widgets/dungeon_monster.cpp /ultima1/widgets/dungeon_monster.h
#	/ultima1/widgets/dungeon_player.cpp /ultima1/widgets/dungeon_player.h
#	/ultima1/widgets/dungeon_widget.cpp /ultima1/widgets/dungeon_widget.h
#	/ultima1/widgets/guard.cpp /ultima1/widgets/guard.h
#	/ultima1/widgets/king.cpp /ultima1/widgets/king.h
#	/ultima1/widgets/merchant.cpp /ultima1/widgets/merchant.h
#	/ultima1/widgets/merchant_armor.h
#	/ultima1/widgets/merchant_armour.cpp /ultima1/widgets/merchant_armour.h
#	/ultima1/widgets/merchant_grocer.cpp /ultima1/widgets/merchant_grocer.h
#	/ultima1/widgets/merchant_magic.cpp /ultima1/widgets/merchant_magic.h
#	/ultima1/widgets/merchant_tavern.cpp /ultima1/widgets/merchant_tavern.h
#	/ultima1/widgets/merchant_transport.cpp /ultima1/widgets/merchant_transport.h
#	/ultima1/widgets/merchant_weapons.cpp /ultima1/widgets/merchant_weapons.h
#	/ultima1/widgets/overworld_monster.cpp /ultima1/widgets/overworld_monster.h
#	/ultima1/widgets/overworld_widget.cpp /ultima1/widgets/overworld_widget.h
#	/ultima1/widgets/person.cpp /ultima1/widgets/person.h
#	/ultima1/widgets/princess.cpp /ultima1/widgets/princess.h
#	/ultima1/widgets/transport.cpp /ultima1/widgets/transport.h
#	/ultima1/widgets/urban_player.cpp /ultima1/widgets/urban_player.h
#	/ultima1/widgets/urban_widget.cpp /ultima1/widgets/urban_widget.h
#	/ultima1/widgets/wench.cpp /ultima1/widgets/wench.h
#	/ultima1/game.cpp /ultima1/game.h
#	/ultima4/controllers/alpha_action_controller.cpp /ultima4/controllers/alpha_action_controller.h
#	/ultima4/controllers/camp_controller.cpp /ultima4/controllers/camp_controller.h
#	/ultima4/controllers/combat_controller.cpp /ultima4/controllers/combat_controller.h
#	/ultima4/controllers/controller.cpp /ultima4/controllers/controller.h
#	/ultima4/controllers/game_controller.cpp /ultima4/controllers/game_controller.h
#	/ultima4/controllers/inn_controller.cpp /ultima4/controllers/inn_controller.h
#	/ultima4/controllers/intro_controller.cpp /ultima4/controllers/intro_controller.h
#	/ultima4/controllers/key_handler_controller.cpp /ultima4/controllers/key_handler_controller.h
#	/ultima4/controllers/menu_controller.cpp /ultima4/controllers/menu_controller.h
#	/ultima4/controllers/read_choice_controller.cpp /ultima4/controllers/read_choice_controller.h
#	/ultima4/controllers/read_dir_controller.cpp /ultima4/controllers/read_dir_controller.h
#	/ultima4/controllers/read_int_controller.cpp /ultima4/controllers/read_int_controller.h
#	/ultima4/controllers/read_player_controller.cpp /ultima4/controllers/read_player_controller.h
#	/ultima4/controllers/read_string_controller.cpp /ultima4/controllers/read_string_controller.h
#	/ultima4/controllers/reagents_menu_controller.cpp /ultima4/controllers/reagents_menu_controller.h
#	/ultima4/controllers/wait_controller.cpp /ultima4/controllers/wait_controller.h
#	/ultima4/controllers/ztats_controller.cpp /ultima4/controllers/ztats_controller.h
#	/ultima4/conversation/conversation.cpp /ultima4/conversation/conversation.h
#	/ultima4/conversation/dialogueloader.cpp /ultima4/conversation/dialogueloader.h
#	/ultima4/conversation/dialogueloader_hw.cpp /ultima4/conversation/dialogueloader_hw.h
#	/ultima4/conversation/dialogueloader_lb.cpp /ultima4/conversation/dialogueloader_lb.h
#	/ultima4/conversation/dialogueloader_tlk.cpp /ultima4/conversation/dialogueloader_tlk.h
#	/ultima4/core/lzw/hash.cpp /ultima4/core/lzw/hash.h
#	/ultima4/core/lzw/lzw.cpp /ultima4/core/lzw/lzw.h
#	/ultima4/core/lzw/u4decode.cpp /ultima4/core/lzw/u4decode.h
#	/ultima4/core/config.cpp /ultima4/core/config.h
#	/ultima4/core/coords.h
#	/ultima4/core/debugger.cpp /ultima4/core/debugger.h
#	/ultima4/core/debugger_actions.cpp /ultima4/core/debugger_actions.h
#	/ultima4/core/observable.h
#	/ultima4/core/observer.h
#	/ultima4/core/settings.cpp /ultima4/core/settings.h
#	/ultima4/core/types.h
#	/ultima4/core/utils.cpp /ultima4/core/utils.h
#	/ultima4/events/event_handler.cpp /ultima4/events/event_handler.h
#	/ultima4/events/timed_event_mgr.cpp /ultima4/events/timed_event_mgr.h
#	/ultima4/filesys/rle.cpp /ultima4/filesys/rle.h
#	/ultima4/filesys/savegame.cpp /ultima4/filesys/savegame.h
#	/ultima4/filesys/u4file.cpp /ultima4/filesys/u4file.h
#	/ultima4/game/armor.cpp /ultima4/game/armor.h
#	/ultima4/game/aura.cpp /ultima4/game/aura.h
#	/ultima4/game/codex.cpp /ultima4/game/codex.h
#	/ultima4/game/context.cpp /ultima4/game/context.h
#	/ultima4/game/creature.cpp /ultima4/game/creature.h
#	/ultima4/game/death.cpp /ultima4/game/death.h
#	/ultima4/game/game.cpp /ultima4/game/game.h
#	/ultima4/game/item.cpp /ultima4/game/item.h
#	/ultima4/game/moongate.cpp /ultima4/game/moongate.h
#	/ultima4/game/names.cpp /ultima4/game/names.h
#	/ultima4/game/object.cpp /ultima4/game/object.h
#	/ultima4/game/person.cpp /ultima4/game/person.h
#	/ultima4/game/player.cpp /ultima4/game/player.h
#	/ultima4/game/portal.cpp /ultima4/game/portal.h
#	/ultima4/game/script.cpp /ultima4/game/script.h
#	/ultima4/game/spell.cpp /ultima4/game/spell.h
#	/ultima4/game/weapon.cpp /ultima4/game/weapon.h
#	/ultima4/gfx/image.cpp /ultima4/gfx/image.h
#	/ultima4/gfx/imageloader.cpp /ultima4/gfx/imageloader.h
#	/ultima4/gfx/imageloader_fmtowns.cpp /ultima4/gfx/imageloader_fmtowns.h
#	/ultima4/gfx/imageloader_png.cpp /ultima4/gfx/imageloader_png.h
#	/ultima4/gfx/imageloader_u4.cpp /ultima4/gfx/imageloader_u4.h
#	/ultima4/gfx/imagemgr.cpp /ultima4/gfx/imagemgr.h
#	/ultima4/gfx/scale.cpp /ultima4/gfx/scale.h
#	/ultima4/gfx/screen.cpp /ultima4/gfx/screen.h
#	/ultima4/gfx/textcolor.h
#	/ultima4/map/annotation.cpp /ultima4/map/annotation.h
#	/ultima4/map/area.h
#	/ultima4/map/city.cpp /ultima4/map/city.h
#	/ultima4/map/direction.cpp /ultima4/map/direction.h
#	/ultima4/map/dungeon.cpp /ultima4/map/dungeon.h
#	/ultima4/map/location.cpp /ultima4/map/location.h
#	/ultima4/map/map.cpp /ultima4/map/map.h
#	/ultima4/map/map_tile.cpp /ultima4/map/map_tile.h
#	/ultima4/map/maploader.cpp /ultima4/map/maploader.h
#	/ultima4/map/mapmgr.cpp /ultima4/map/mapmgr.h
#	/ultima4/map/movement.cpp /ultima4/map/movement.h
#	/ultima4/map/shrine.cpp /ultima4/map/shrine.h
#	/ultima4/map/tile.cpp /ultima4/map/tile.h
#	/ultima4/map/tileanim.cpp /ultima4/map/tileanim.h
#	/ultima4/map/tilemap.cpp /ultima4/map/tilemap.h
#	/ultima4/map/tileset.cpp /ultima4/map/tileset.h
#	/ultima4/map/xml_map.h
#	/ultima4/sound/music.cpp /ultima4/sound/music.h
#	/ultima4/sound/sound.cpp /ultima4/sound/sound.h
#	/ultima4/views/dungeonview.cpp /ultima4/views/dungeonview.h
#	/ultima4/views/imageview.cpp /ultima4/views/imageview.h
#	/ultima4/views/menu.cpp /ultima4/views/menu.h
#	/ultima4/views/menuitem.cpp /ultima4/views/menuitem.h
#	/ultima4/views/stats.cpp /ultima4/views/stats.h
#	/ultima4/views/textview.cpp /ultima4/views/textview.h
#	/ultima4/views/tileview.cpp /ultima4/views/tileview.h
#	/ultima4/views/view.cpp /ultima4/views/view.h
#	/ultima4/meta_engine.cpp /ultima4/meta_engine.h
#	/ultima4/ultima4.cpp /ultima4/ultima4.h
#	/ultima8/audio/audio_channel.cpp /ultima8/audio/audio_channel.h
#	/ultima8/audio/audio_mixer.cpp /ultima8/audio/audio_mixer.h
#	/ultima8/audio/audio_process.cpp /ultima8/audio/audio_process.h
#	/ultima8/audio/audio_sample.cpp /ultima8/audio/audio_sample.h
#	/ultima8/audio/cru_music_process.cpp /ultima8/audio/cru_music_process.h
#	/ultima8/audio/midi_player.cpp /ultima8/audio/midi_player.h
#	/ultima8/audio/music_flex.cpp /ultima8/audio/music_flex.h
#	/ultima8/audio/music_process.cpp /ultima8/audio/music_process.h
#	/ultima8/audio/raw_audio_sample.cpp /ultima8/audio/raw_audio_sample.h
#	/ultima8/audio/sonarc_audio_sample.cpp /ultima8/audio/sonarc_audio_sample.h
#	/ultima8/audio/sound_flex.cpp /ultima8/audio/sound_flex.h
#	/ultima8/audio/speech_flex.cpp /ultima8/audio/speech_flex.h
#	/ultima8/audio/u8_music_process.cpp /ultima8/audio/u8_music_process.h
#	/ultima8/conf/config_file_manager.cpp /ultima8/conf/config_file_manager.h
#	/ultima8/convert/crusader/convert_shape_crusader.cpp /ultima8/convert/crusader/convert_shape_crusader.h
#	/ultima8/convert/crusader/convert_usecode_crusader.h
#	/ultima8/convert/crusader/convert_usecode_regret.h
#	/ultima8/convert/u8/convert_shape_u8.cpp /ultima8/convert/u8/convert_shape_u8.h
#	/ultima8/convert/u8/convert_usecode_u8.h
#	/ultima8/convert/convert_shape.cpp /ultima8/convert/convert_shape.h
#	/ultima8/convert/convert_usecode.h
#	/ultima8/filesys/archive.cpp /ultima8/filesys/archive.h
#	/ultima8/filesys/archive_file.cpp /ultima8/filesys/archive_file.h
#	/ultima8/filesys/file_system.cpp /ultima8/filesys/file_system.h
#	/ultima8/filesys/flex_file.cpp /ultima8/filesys/flex_file.h
#	/ultima8/filesys/named_archive_file.h
#	/ultima8/filesys/raw_archive.cpp /ultima8/filesys/raw_archive.h
#	/ultima8/filesys/savegame.cpp /ultima8/filesys/savegame.h
#	/ultima8/filesys/u8_save_file.cpp /ultima8/filesys/u8_save_file.h
#	/ultima8/games/cru_game.cpp /ultima8/games/cru_game.h
#	/ultima8/games/game.cpp /ultima8/games/game.h
#	/ultima8/games/game_data.cpp /ultima8/games/game_data.h
#	/ultima8/games/game_info.cpp /ultima8/games/game_info.h
#	/ultima8/games/start_crusader_process.cpp /ultima8/games/start_crusader_process.h
#	/ultima8/games/start_u8_process.cpp /ultima8/games/start_u8_process.h
#	/ultima8/games/treasure_loader.cpp /ultima8/games/treasure_loader.h
#	/ultima8/games/u8_game.cpp /ultima8/games/u8_game.h
#	/ultima8/graphics/fonts/font.cpp /ultima8/graphics/fonts/font.h
#	/ultima8/graphics/fonts/font_manager.cpp /ultima8/graphics/fonts/font_manager.h
#	/ultima8/graphics/fonts/font_shape_archive.cpp /ultima8/graphics/fonts/font_shape_archive.h
#	/ultima8/graphics/fonts/jp_font.cpp /ultima8/graphics/fonts/jp_font.h
#	/ultima8/graphics/fonts/jp_rendered_text.cpp /ultima8/graphics/fonts/jp_rendered_text.h
#	/ultima8/graphics/fonts/rendered_text.cpp /ultima8/graphics/fonts/rendered_text.h
#	/ultima8/graphics/fonts/shape_font.cpp /ultima8/graphics/fonts/shape_font.h
#	/ultima8/graphics/fonts/shape_rendered_text.cpp /ultima8/graphics/fonts/shape_rendered_text.h
#	/ultima8/graphics/fonts/tt_font.cpp /ultima8/graphics/fonts/tt_font.h
#	/ultima8/graphics/fonts/ttf_rendered_text.cpp /ultima8/graphics/fonts/ttf_rendered_text.h
#	/ultima8/graphics/anim_dat.cpp /ultima8/graphics/anim_dat.h
#	/ultima8/graphics/avi_player.cpp /ultima8/graphics/avi_player.h
#	/ultima8/graphics/base_soft_render_surface.cpp /ultima8/graphics/base_soft_render_surface.h
#	/ultima8/graphics/cycle_process.cpp /ultima8/graphics/cycle_process.h
#	/ultima8/graphics/fade_to_modal_process.cpp /ultima8/graphics/fade_to_modal_process.h
#	/ultima8/graphics/frame_id.cpp /ultima8/graphics/frame_id.h
#	/ultima8/graphics/gump_shape_archive.cpp /ultima8/graphics/gump_shape_archive.h
#	/ultima8/graphics/inverter_process.cpp /ultima8/graphics/inverter_process.h
#	/ultima8/graphics/main_shape_archive.cpp /ultima8/graphics/main_shape_archive.h
#	/ultima8/graphics/movie_player.h
#	/ultima8/graphics/palette.cpp /ultima8/graphics/palette.h
#	/ultima8/graphics/palette_fader_process.cpp /ultima8/graphics/palette_fader_process.h
#	/ultima8/graphics/palette_manager.cpp /ultima8/graphics/palette_manager.h
#	/ultima8/graphics/raw_shape_frame.cpp /ultima8/graphics/raw_shape_frame.h
#	/ultima8/graphics/render_surface.cpp /ultima8/graphics/render_surface.h
#	/ultima8/graphics/shape.cpp /ultima8/graphics/shape.h
#	/ultima8/graphics/shape_archive.cpp /ultima8/graphics/shape_archive.h
#	/ultima8/graphics/shape_frame.cpp /ultima8/graphics/shape_frame.h
#	/ultima8/graphics/shape_info.cpp /ultima8/graphics/shape_info.h
#	/ultima8/graphics/skf_player.cpp /ultima8/graphics/skf_player.h
#	/ultima8/graphics/soft_render_surface.cpp /ultima8/graphics/soft_render_surface.h
#	/ultima8/graphics/texture.h
#	/ultima8/graphics/type_flags.cpp /ultima8/graphics/type_flags.h
#	/ultima8/graphics/wpn_ovlay_dat.cpp /ultima8/graphics/wpn_ovlay_dat.h
#	/ultima8/graphics/xform_blend.cpp /ultima8/graphics/xform_blend.h
#	/ultima8/gumps/widgets/button_widget.cpp /ultima8/gumps/widgets/button_widget.h
#	/ultima8/gumps/widgets/edit_widget.cpp /ultima8/gumps/widgets/edit_widget.h
#	/ultima8/gumps/widgets/sliding_widget.cpp /ultima8/gumps/widgets/sliding_widget.h
#	/ultima8/gumps/widgets/text_widget.cpp /ultima8/gumps/widgets/text_widget.h
#	/ultima8/gumps/ask_gump.cpp /ultima8/gumps/ask_gump.h
#	/ultima8/gumps/bark_gump.cpp /ultima8/gumps/bark_gump.h
#	/ultima8/gumps/book_gump.cpp /ultima8/gumps/book_gump.h
#	/ultima8/gumps/computer_gump.cpp /ultima8/gumps/computer_gump.h
#	/ultima8/gumps/container_gump.cpp /ultima8/gumps/container_gump.h
#	/ultima8/gumps/credits_gump.cpp /ultima8/gumps/credits_gump.h
#	/ultima8/gumps/cru_ammo_gump.cpp /ultima8/gumps/cru_ammo_gump.h
#	/ultima8/gumps/cru_credits_gump.cpp /ultima8/gumps/cru_credits_gump.h
#	/ultima8/gumps/cru_demo_gump.cpp /ultima8/gumps/cru_demo_gump.h
#	/ultima8/gumps/cru_energy_gump.cpp /ultima8/gumps/cru_energy_gump.h
#	/ultima8/gumps/cru_health_gump.cpp /ultima8/gumps/cru_health_gump.h
#	/ultima8/gumps/cru_inventory_gump.cpp /ultima8/gumps/cru_inventory_gump.h
#	/ultima8/gumps/cru_menu_gump.cpp /ultima8/gumps/cru_menu_gump.h
#	/ultima8/gumps/cru_pickup_area_gump.cpp /ultima8/gumps/cru_pickup_area_gump.h
#	/ultima8/gumps/cru_pickup_gump.cpp /ultima8/gumps/cru_pickup_gump.h
#	/ultima8/gumps/cru_stat_gump.cpp /ultima8/gumps/cru_stat_gump.h
#	/ultima8/gumps/cru_status_gump.cpp /ultima8/gumps/cru_status_gump.h
#	/ultima8/gumps/cru_weapon_gump.cpp /ultima8/gumps/cru_weapon_gump.h
#	/ultima8/gumps/desktop_gump.cpp /ultima8/gumps/desktop_gump.h
#	/ultima8/gumps/difficulty_gump.cpp /ultima8/gumps/difficulty_gump.h
#	/ultima8/gumps/fast_area_vis_gump.cpp /ultima8/gumps/fast_area_vis_gump.h
#	/ultima8/gumps/game_map_gump.cpp /ultima8/gumps/game_map_gump.h
#	/ultima8/gumps/gump.cpp /ultima8/gumps/gump.h
#	/ultima8/gumps/gump_notify_process.cpp /ultima8/gumps/gump_notify_process.h
#	/ultima8/gumps/inverter_gump.cpp /ultima8/gumps/inverter_gump.h
#	/ultima8/gumps/item_relative_gump.cpp /ultima8/gumps/item_relative_gump.h
#	/ultima8/gumps/keypad_gump.cpp /ultima8/gumps/keypad_gump.h
#	/ultima8/gumps/main_menu_process.cpp /ultima8/gumps/main_menu_process.h
#	/ultima8/gumps/menu_gump.cpp /ultima8/gumps/menu_gump.h
#	/ultima8/gumps/message_box_gump.cpp /ultima8/gumps/message_box_gump.h
#	/ultima8/gumps/mini_stats_gump.cpp /ultima8/gumps/mini_stats_gump.h
#	/ultima8/gumps/minimap_gump.cpp /ultima8/gumps/minimap_gump.h
#	/ultima8/gumps/modal_gump.cpp /ultima8/gumps/modal_gump.h
#	/ultima8/gumps/movie_gump.cpp /ultima8/gumps/movie_gump.h
#	/ultima8/gumps/paged_gump.cpp /ultima8/gumps/paged_gump.h
#	/ultima8/gumps/paperdoll_gump.cpp /ultima8/gumps/paperdoll_gump.h
#	/ultima8/gumps/quit_gump.cpp /ultima8/gumps/quit_gump.h
#	/ultima8/gumps/readable_gump.cpp /ultima8/gumps/readable_gump.h
#	/ultima8/gumps/resizable_gump.cpp /ultima8/gumps/resizable_gump.h
#	/ultima8/gumps/scroll_gump.cpp /ultima8/gumps/scroll_gump.h
#	/ultima8/gumps/shape_viewer_gump.cpp /ultima8/gumps/shape_viewer_gump.h
#	/ultima8/gumps/slider_gump.cpp /ultima8/gumps/slider_gump.h
#	/ultima8/gumps/target_gump.cpp /ultima8/gumps/target_gump.h
#	/ultima8/gumps/translucent_gump.cpp /ultima8/gumps/translucent_gump.h
#	/ultima8/gumps/u8_save_gump.cpp /ultima8/gumps/u8_save_gump.h
#	/ultima8/gumps/weasel_dat.cpp /ultima8/gumps/weasel_dat.h
#	/ultima8/gumps/weasel_gump.cpp /ultima8/gumps/weasel_gump.h
#	/ultima8/kernel/delay_process.cpp /ultima8/kernel/delay_process.h
#	/ultima8/kernel/kernel.cpp /ultima8/kernel/kernel.h
#	/ultima8/kernel/mouse.cpp /ultima8/kernel/mouse.h
#	/ultima8/kernel/object.cpp /ultima8/kernel/object.h
#	/ultima8/kernel/object_manager.cpp /ultima8/kernel/object_manager.h
#	/ultima8/kernel/process.cpp /ultima8/kernel/process.h
#	/ultima8/misc/box.h
#	/ultima8/misc/classtype.h
#	/ultima8/misc/common_types.h
#	/ultima8/misc/debugger.cpp /ultima8/misc/debugger.h
#	/ultima8/misc/direction.h
#	/ultima8/misc/direction_util.h
#	/ultima8/misc/encoding.cpp /ultima8/misc/encoding.h
#	/ultima8/misc/id_man.cpp /ultima8/misc/id_man.h
#	/ultima8/misc/istring.h
#	/ultima8/misc/pent_include.h
#	/ultima8/misc/point3.h
#	/ultima8/misc/rect.h
#	/ultima8/misc/stream_util.h
#	/ultima8/misc/util.cpp /ultima8/misc/util.h
#	/ultima8/usecode/bit_set.cpp /ultima8/usecode/bit_set.h
#	/ultima8/usecode/byte_set.cpp /ultima8/usecode/byte_set.h
#	/ultima8/usecode/global_storage.h
#	/ultima8/usecode/intrinsics.h
#	/ultima8/usecode/regret_intrinsics.h
#	/ultima8/usecode/remorse_intrinsics.h
#	/ultima8/usecode/u8_intrinsics.h
#	/ultima8/usecode/uc_list.cpp /ultima8/usecode/uc_list.h
#	/ultima8/usecode/uc_machine.cpp /ultima8/usecode/uc_machine.h
#	/ultima8/usecode/uc_process.cpp /ultima8/usecode/uc_process.h
#	/ultima8/usecode/uc_stack.cpp /ultima8/usecode/uc_stack.h
#	/ultima8/usecode/usecode.cpp /ultima8/usecode/usecode.h
#	/ultima8/usecode/usecode_flex.cpp /ultima8/usecode/usecode_flex.h
#	/ultima8/world/actors/actor.cpp /ultima8/world/actors/actor.h
#	/ultima8/world/actors/actor_anim.h
#	/ultima8/world/actors/actor_anim_process.cpp /ultima8/world/actors/actor_anim_process.h
#	/ultima8/world/actors/actor_bark_notify_process.cpp /ultima8/world/actors/actor_bark_notify_process.h
#	/ultima8/world/actors/ambush_process.cpp /ultima8/world/actors/ambush_process.h
#	/ultima8/world/actors/anim_action.cpp /ultima8/world/actors/anim_action.h
#	/ultima8/world/actors/animation.cpp /ultima8/world/actors/animation.h
#	/ultima8/world/actors/animation_tracker.cpp /ultima8/world/actors/animation_tracker.h
#	/ultima8/world/actors/attack_process.cpp /ultima8/world/actors/attack_process.h
#	/ultima8/world/actors/auto_firer_process.cpp /ultima8/world/actors/auto_firer_process.h
#	/ultima8/world/actors/avatar_death_process.cpp /ultima8/world/actors/avatar_death_process.h
#	/ultima8/world/actors/avatar_gravity_process.cpp /ultima8/world/actors/avatar_gravity_process.h
#	/ultima8/world/actors/avatar_mover_process.cpp /ultima8/world/actors/avatar_mover_process.h
#	/ultima8/world/actors/battery_charger_process.cpp /ultima8/world/actors/battery_charger_process.h
#	/ultima8/world/actors/clear_feign_death_process.cpp /ultima8/world/actors/clear_feign_death_process.h
#	/ultima8/world/actors/combat_dat.cpp /ultima8/world/actors/combat_dat.h
#	/ultima8/world/actors/combat_process.cpp /ultima8/world/actors/combat_process.h
#	/ultima8/world/actors/cru_avatar_mover_process.cpp /ultima8/world/actors/cru_avatar_mover_process.h
#	/ultima8/world/actors/cru_healer_process.cpp /ultima8/world/actors/cru_healer_process.h
#	/ultima8/world/actors/cru_pathfinder_process.cpp /ultima8/world/actors/cru_pathfinder_process.h
#	/ultima8/world/actors/grant_peace_process.cpp /ultima8/world/actors/grant_peace_process.h
#	/ultima8/world/actors/guard_process.cpp /ultima8/world/actors/guard_process.h
#	/ultima8/world/actors/heal_process.cpp /ultima8/world/actors/heal_process.h
#	/ultima8/world/actors/loiter_process.cpp /ultima8/world/actors/loiter_process.h
#	/ultima8/world/actors/main_actor.cpp /ultima8/world/actors/main_actor.h
#	/ultima8/world/actors/monster_info.h
#	/ultima8/world/actors/npc_dat.cpp /ultima8/world/actors/npc_dat.h
#	/ultima8/world/actors/pace_process.cpp /ultima8/world/actors/pace_process.h
#	/ultima8/world/actors/pathfinder.cpp /ultima8/world/actors/pathfinder.h
#	/ultima8/world/actors/pathfinder_process.cpp /ultima8/world/actors/pathfinder_process.h
#	/ultima8/world/actors/quick_avatar_mover_process.cpp /ultima8/world/actors/quick_avatar_mover_process.h
#	/ultima8/world/actors/resurrection_process.cpp /ultima8/world/actors/resurrection_process.h
#	/ultima8/world/actors/rolling_thunder_process.cpp /ultima8/world/actors/rolling_thunder_process.h
#	/ultima8/world/actors/scheduler_process.cpp /ultima8/world/actors/scheduler_process.h
#	/ultima8/world/actors/surrender_process.cpp /ultima8/world/actors/surrender_process.h
#	/ultima8/world/actors/targeted_anim_process.cpp /ultima8/world/actors/targeted_anim_process.h
#	/ultima8/world/actors/teleport_to_egg_process.cpp /ultima8/world/actors/teleport_to_egg_process.h
#	/ultima8/world/actors/treasure_info.h
#	/ultima8/world/actors/u8_avatar_mover_process.cpp /ultima8/world/actors/u8_avatar_mover_process.h
#	/ultima8/world/actors/weapon_overlay.h
#	/ultima8/world/armour_info.h
#	/ultima8/world/bobo_boomer_process.cpp /ultima8/world/bobo_boomer_process.h
#	/ultima8/world/camera_process.cpp /ultima8/world/camera_process.h
#	/ultima8/world/container.cpp /ultima8/world/container.h
#	/ultima8/world/coord_utils.h
#	/ultima8/world/create_item_process.cpp /ultima8/world/create_item_process.h
#	/ultima8/world/crosshair_process.cpp /ultima8/world/crosshair_process.h
#	/ultima8/world/current_map.cpp /ultima8/world/current_map.h
#	/ultima8/world/damage_info.cpp /ultima8/world/damage_info.h
#	/ultima8/world/destroy_item_process.cpp /ultima8/world/destroy_item_process.h
#	/ultima8/world/egg.cpp /ultima8/world/egg.h
#	/ultima8/world/egg_hatcher_process.cpp /ultima8/world/egg_hatcher_process.h
#	/ultima8/world/fire_type.cpp /ultima8/world/fire_type.h
#	/ultima8/world/fire_type_table.cpp /ultima8/world/fire_type_table.h
#	/ultima8/world/fireball_process.cpp /ultima8/world/fireball_process.h
#	/ultima8/world/get_object.cpp /ultima8/world/get_object.h
#	/ultima8/world/glob_egg.cpp /ultima8/world/glob_egg.h
#	/ultima8/world/gravity_process.cpp /ultima8/world/gravity_process.h
#	/ultima8/world/item.cpp /ultima8/world/item.h
#	/ultima8/world/item_factory.cpp /ultima8/world/item_factory.h
#	/ultima8/world/item_selection_process.cpp /ultima8/world/item_selection_process.h
#	/ultima8/world/item_sorter.cpp /ultima8/world/item_sorter.h
#	/ultima8/world/loop_script.h
#	/ultima8/world/map.cpp /ultima8/world/map.h
#	/ultima8/world/map_glob.cpp /ultima8/world/map_glob.h
#	/ultima8/world/missile_tracker.cpp /ultima8/world/missile_tracker.h
#	/ultima8/world/monster_egg.cpp /ultima8/world/monster_egg.h
#	/ultima8/world/snap_process.cpp /ultima8/world/snap_process.h
#	/ultima8/world/sort_item.h
#	/ultima8/world/split_item_process.cpp /ultima8/world/split_item_process.h
#	/ultima8/world/sprite_process.cpp /ultima8/world/sprite_process.h
#	/ultima8/world/super_sprite_process.cpp /ultima8/world/super_sprite_process.h
#	/ultima8/world/target_reticle_process.cpp /ultima8/world/target_reticle_process.h
#	/ultima8/world/teleport_egg.cpp /ultima8/world/teleport_egg.h
#	/ultima8/world/weapon_info.h
#	/ultima8/world/world.cpp /ultima8/world/world.h
#	/ultima8/world/world_point.h
#	/ultima8/meta_engine.cpp /ultima8/meta_engine.h
#	/ultima8/ultima8.cpp /ultima8/ultima8.h
#	/detection.h
#	/detection_tables.h
#	/metaengine.cpp /metaengine.h
#	/module.mk
#)
#
#add_engine(voyeur
#	/animation.cpp /animation.h
#	/data.cpp /data.h
#	/debugger.cpp /debugger.h
#	/detection.h
#	/detection_tables.h
#	/events.cpp /events.h
#	/files.cpp /files.h
#	/files_threads.cpp
#	/metaengine.cpp
#	/module.mk
#	/screen.cpp /screen.h
#	/sound.cpp /sound.h
#	/staticres.cpp /staticres.h
#	/voyeur.cpp /voyeur.h
#	/voyeur_game.cpp
#)
#
#add_engine(wintermute
#	/ad/ad_actor.cpp /ad/ad_actor.h
#	/ad/ad_actor_3dx.cpp /ad/ad_actor_3dx.h
#	/ad/ad_attach_3dx.cpp /ad/ad_attach_3dx.h
#	/ad/ad_block.cpp /ad/ad_block.h
#	/ad/ad_entity.cpp /ad/ad_entity.h
#	/ad/ad_game.cpp /ad/ad_game.h
#	/ad/ad_generic.cpp /ad/ad_generic.h
#	/ad/ad_geom_ext.cpp /ad/ad_geom_ext.h
#	/ad/ad_geom_ext_node.cpp /ad/ad_geom_ext_node.h
#	/ad/ad_inventory.cpp /ad/ad_inventory.h
#	/ad/ad_inventory_box.cpp /ad/ad_inventory_box.h
#	/ad/ad_item.cpp /ad/ad_item.h
#	/ad/ad_layer.cpp /ad/ad_layer.h
#	/ad/ad_node_state.cpp /ad/ad_node_state.h
#	/ad/ad_object.cpp /ad/ad_object.h
#	/ad/ad_object_3d.cpp /ad/ad_object_3d.h
#	/ad/ad_path.cpp /ad/ad_path.h
#	/ad/ad_path3d.cpp /ad/ad_path3d.h
#	/ad/ad_path_point.cpp /ad/ad_path_point.h
#	/ad/ad_path_point3d.cpp /ad/ad_path_point3d.h
#	/ad/ad_region.cpp /ad/ad_region.h
#	/ad/ad_response.cpp /ad/ad_response.h
#	/ad/ad_response_box.cpp /ad/ad_response_box.h
#	/ad/ad_response_context.cpp /ad/ad_response_context.h
#	/ad/ad_rot_level.cpp /ad/ad_rot_level.h
#	/ad/ad_scale_level.cpp /ad/ad_scale_level.h
#	/ad/ad_scene.cpp /ad/ad_scene.h
#	/ad/ad_scene_geometry.cpp /ad/ad_scene_geometry.h
#	/ad/ad_scene_node.cpp /ad/ad_scene_node.h
#	/ad/ad_scene_state.cpp /ad/ad_scene_state.h
#	/ad/ad_sentence.cpp /ad/ad_sentence.h
#	/ad/ad_sprite_set.cpp /ad/ad_sprite_set.h
#	/ad/ad_talk_def.cpp /ad/ad_talk_def.h
#	/ad/ad_talk_holder.cpp /ad/ad_talk_holder.h
#	/ad/ad_talk_node.cpp /ad/ad_talk_node.h
#	/ad/ad_types.h
#	/ad/ad_walkplane.cpp /ad/ad_walkplane.h
#	/ad/ad_waypoint_group.cpp /ad/ad_waypoint_group.h
#	/ad/ad_waypoint_group3d.cpp /ad/ad_waypoint_group3d.h
#	/base/file/base_disk_file.cpp /base/file/base_disk_file.h
#	/base/file/base_file.cpp /base/file/base_file.h
#	/base/file/base_file_entry.cpp /base/file/base_file_entry.h
#	/base/file/base_package.cpp /base/file/base_package.h
#	/base/file/base_save_thumb_file.cpp /base/file/base_save_thumb_file.h
#	/base/file/base_savefile_manager_file.cpp /base/file/base_savefile_manager_file.h
#	/base/file/dcpackage.h
#	/base/font/base_font.cpp /base/font/base_font.h
#	/base/font/base_font_bitmap.cpp /base/font/base_font_bitmap.h
#	/base/font/base_font_storage.cpp /base/font/base_font_storage.h
#	/base/font/base_font_truetype.cpp /base/font/base_font_truetype.h
#	/base/gfx/3ds/camera3d.cpp /base/gfx/3ds/camera3d.h
#	/base/gfx/3ds/light3d.cpp /base/gfx/3ds/light3d.h
#	/base/gfx/3ds/loader3ds.cpp /base/gfx/3ds/loader3ds.h
#	/base/gfx/3ds/mesh3ds.cpp /base/gfx/3ds/mesh3ds.h
#	/base/gfx/opengl/base_render_opengl3d.cpp /base/gfx/opengl/base_render_opengl3d.h
#	/base/gfx/opengl/base_render_opengl3d_shader.cpp /base/gfx/opengl/base_render_opengl3d_shader.h
#	/base/gfx/opengl/base_surface_opengl3d.cpp /base/gfx/opengl/base_surface_opengl3d.h
#	/base/gfx/opengl/mesh3ds_opengl.cpp /base/gfx/opengl/mesh3ds_opengl.h
#	/base/gfx/opengl/mesh3ds_opengl_shader.cpp /base/gfx/opengl/mesh3ds_opengl_shader.h
#	/base/gfx/opengl/meshx_opengl.cpp /base/gfx/opengl/meshx_opengl.h
#	/base/gfx/opengl/meshx_opengl_shader.cpp /base/gfx/opengl/meshx_opengl_shader.h
#	/base/gfx/opengl/shadow_volume_opengl.cpp /base/gfx/opengl/shadow_volume_opengl.h
#	/base/gfx/opengl/shadow_volume_opengl_shader.cpp /base/gfx/opengl/shadow_volume_opengl_shader.h
#	/base/gfx/osystem/base_render_osystem.cpp /base/gfx/osystem/base_render_osystem.h
#	/base/gfx/osystem/base_surface_osystem.cpp /base/gfx/osystem/base_surface_osystem.h
#	/base/gfx/osystem/render_ticket.cpp /base/gfx/osystem/render_ticket.h
#	/base/gfx/x/active_animation.cpp /base/gfx/x/active_animation.h
#	/base/gfx/x/animation.cpp /base/gfx/x/animation.h
#	/base/gfx/x/animation_channel.cpp /base/gfx/x/animation_channel.h
#	/base/gfx/x/animation_set.cpp /base/gfx/x/animation_set.h
#	/base/gfx/x/frame_node.cpp /base/gfx/x/frame_node.h
#	/base/gfx/x/loader_x.cpp /base/gfx/x/loader_x.h
#	/base/gfx/x/material.cpp /base/gfx/x/material.h
#	/base/gfx/x/meshx.cpp /base/gfx/x/meshx.h
#	/base/gfx/x/modelx.cpp /base/gfx/x/modelx.h
#	/base/gfx/base_image.cpp /base/gfx/base_image.h
#	/base/gfx/base_renderer.cpp /base/gfx/base_renderer.h
#	/base/gfx/base_renderer3d.cpp /base/gfx/base_renderer3d.h
#	/base/gfx/base_surface.cpp /base/gfx/base_surface.h
#	/base/gfx/shadow_volume.cpp /base/gfx/shadow_volume.h
#	/base/particles/part_emitter.cpp /base/particles/part_emitter.h
#	/base/particles/part_force.cpp /base/particles/part_force.h
#	/base/particles/part_particle.cpp /base/particles/part_particle.h
#	/base/scriptables/debuggable/debuggable_script.cpp /base/scriptables/debuggable/debuggable_script.h
#	/base/scriptables/debuggable/debuggable_script_engine.cpp /base/scriptables/debuggable/debuggable_script_engine.h
#	/base/scriptables/dcscript.h
#	/base/scriptables/script.cpp /base/scriptables/script.h
#	/base/scriptables/script_engine.cpp /base/scriptables/script_engine.h
#	/base/scriptables/script_ext_array.cpp /base/scriptables/script_ext_array.h
#	/base/scriptables/script_ext_date.cpp /base/scriptables/script_ext_date.h
#	/base/scriptables/script_ext_directory.cpp /base/scriptables/script_ext_directory.h
#	/base/scriptables/script_ext_file.cpp /base/scriptables/script_ext_file.h
#	/base/scriptables/script_ext_math.cpp /base/scriptables/script_ext_math.h
#	/base/scriptables/script_ext_mem_buffer.cpp /base/scriptables/script_ext_mem_buffer.h
#	/base/scriptables/script_ext_object.cpp /base/scriptables/script_ext_object.h
#	/base/scriptables/script_ext_string.cpp /base/scriptables/script_ext_string.h
#	/base/scriptables/script_opcodes.h
#	/base/scriptables/script_stack.cpp /base/scriptables/script_stack.h
#	/base/scriptables/script_value.cpp /base/scriptables/script_value.h
#	/base/sound/base_sound.cpp /base/sound/base_sound.h
#	/base/sound/base_sound_buffer.cpp /base/sound/base_sound_buffer.h
#	/base/sound/base_sound_manager.cpp /base/sound/base_sound_manager.h
#	/base/base.cpp /base/base.h
#	/base/base_active_rect.cpp /base/base_active_rect.h
#	/base/base_animation_transition_time.cpp /base/base_animation_transition_time.h
#	/base/base_dynamic_buffer.cpp /base/base_dynamic_buffer.h
#	/base/base_engine.cpp /base/base_engine.h
#	/base/base_fader.cpp /base/base_fader.h
#	/base/base_file_manager.cpp /base/base_file_manager.h
#	/base/base_frame.cpp /base/base_frame.h
#	/base/base_game.cpp /base/base_game.h
#	/base/base_game_custom_actions.h
#	/base/base_game_music.cpp /base/base_game_music.h
#	/base/base_game_settings.cpp /base/base_game_settings.h
#	/base/base_keyboard_state.cpp /base/base_keyboard_state.h
#	/base/base_named_object.cpp /base/base_named_object.h
#	/base/base_object.cpp /base/base_object.h
#	/base/base_parser.cpp /base/base_parser.h
#	/base/base_persistence_manager.cpp /base/base_persistence_manager.h
#	/base/base_point.cpp /base/base_point.h
#	/base/base_quick_msg.cpp /base/base_quick_msg.h
#	/base/base_region.cpp /base/base_region.h
#	/base/base_script_holder.cpp /base/base_script_holder.h
#	/base/base_scriptable.cpp /base/base_scriptable.h
#	/base/base_sprite.cpp /base/base_sprite.h
#	/base/base_string_table.cpp /base/base_string_table.h
#	/base/base_sub_frame.cpp /base/base_sub_frame.h
#	/base/base_surface_storage.cpp /base/base_surface_storage.h
#	/base/base_transition_manager.cpp /base/base_transition_manager.h
#	/base/base_viewport.cpp /base/base_viewport.h
#	/base/save_thumb_helper.cpp /base/save_thumb_helper.h
#	/base/saveload.cpp /base/saveload.h
#	/base/timer.cpp /base/timer.h
#	/debugger/listing_providers/basic_source_listing_provider.cpp /debugger/listing_providers/basic_source_listing_provider.h
#	/debugger/listing_providers/blank_listing.cpp /debugger/listing_providers/blank_listing.h
#	/debugger/listing_providers/blank_listing_provider.cpp /debugger/listing_providers/blank_listing_provider.h
#	/debugger/listing_providers/cached_source_listing_provider.cpp /debugger/listing_providers/cached_source_listing_provider.h
#	/debugger/listing_providers/source_listing.cpp /debugger/listing_providers/source_listing.h
#	/debugger/listing_providers/source_listing_provider.h
#	/debugger/breakpoint.cpp /debugger/breakpoint.h
#	/debugger/debugger_controller.cpp /debugger/debugger_controller.h
#	/debugger/error.cpp /debugger/error.h
#	/debugger/listing.cpp /debugger/listing.h
#	/debugger/listing_provider.h
#	/debugger/script_monitor.cpp /debugger/script_monitor.h
#	/debugger/watch.cpp /debugger/watch.h
#	/debugger/watch_instance.cpp /debugger/watch_instance.h
#	/ext/dll_dlltest.cpp
#	/ext/dll_geturl.cpp
#	/ext/dll_httpconnect.cpp
#	/ext/dll_img.cpp
#	/ext/dll_installutil.cpp
#	/ext/dll_kernel32.cpp
#	/ext/dll_shell32.cpp
#	/ext/dll_tools.cpp
#	/ext/externals.h
#	/ext/plugins.h
#	/ext/scene_achievements.cpp
#	/ext/scene_achievements_tables.h
#	/ext/scene_hooks.h
#	/ext/wme_3fstatistics.cpp /ext/wme_3fstatistics.h
#	/ext/wme_commandlinehelper.cpp /ext/wme_commandlinehelper.h
#	/ext/wme_galaxy.cpp /ext/wme_galaxy.h
#	/ext/wme_steam.cpp /ext/wme_steam.h
#	/math/floatpoint.h
#	/math/math_util.cpp /math/math_util.h
#	/math/matrix4.cpp /math/matrix4.h
#	/math/rect32.h
#	/math/vector2.cpp /math/vector2.h
#	/system/sys_class.cpp /system/sys_class.h
#	/system/sys_class_registry.cpp /system/sys_class_registry.h
#	/system/sys_instance.cpp /system/sys_instance.h
#	/ui/ui_button.cpp /ui/ui_button.h
#	/ui/ui_edit.cpp /ui/ui_edit.h
#	/ui/ui_entity.cpp /ui/ui_entity.h
#	/ui/ui_object.cpp /ui/ui_object.h
#	/ui/ui_text.cpp /ui/ui_text.h
#	/ui/ui_tiled_image.cpp /ui/ui_tiled_image.h
#	/ui/ui_window.cpp /ui/ui_window.h
#	/utils/crc.cpp /utils/crc.h
#	/utils/path_util.cpp /utils/path_util.h
#	/utils/string_util.cpp /utils/string_util.h
#	/utils/utils.cpp /utils/utils.h
#	/video/subtitle_card.cpp /video/subtitle_card.h
#	/video/video_player.cpp /video/video_player.h
#	/video/video_subtitler.cpp /video/video_subtitler.h
#	/video/video_theora_player.cpp /video/video_theora_player.h
#	/achievements_tables.h
#	/coll_templ.h
#	/dcgf.h
#	/dctypes.h
#	/debugger.cpp /debugger.h
#	/detection.h
#	/detection_tables.h
#	/keymapper_tables.h
#	/metaengine.cpp
#	/module.mk
#	/persistent.cpp /persistent.h
#	/platform_osystem.cpp /platform_osystem.h
#	/wintermute.cpp /wintermute.h
#	/wintypes.h
#)
#
#add_engine(xeen
#	/dialogs/credits_screen.cpp /dialogs/credits_screen.h
#	/dialogs/dialogs.cpp /dialogs/dialogs.h
#	/dialogs/dialogs_awards.cpp /dialogs/dialogs_awards.h
#	/dialogs/dialogs_char_info.cpp /dialogs/dialogs_char_info.h
#	/dialogs/dialogs_control_panel.cpp /dialogs/dialogs_control_panel.h
#	/dialogs/dialogs_copy_protection.cpp /dialogs/dialogs_copy_protection.h
#	/dialogs/dialogs_create_char.cpp /dialogs/dialogs_create_char.h
#	/dialogs/dialogs_difficulty.cpp /dialogs/dialogs_difficulty.h
#	/dialogs/dialogs_dismiss.cpp /dialogs/dialogs_dismiss.h
#	/dialogs/dialogs_exchange.cpp /dialogs/dialogs_exchange.h
#	/dialogs/dialogs_info.cpp /dialogs/dialogs_info.h
#	/dialogs/dialogs_input.cpp /dialogs/dialogs_input.h
#	/dialogs/dialogs_items.cpp /dialogs/dialogs_items.h
#	/dialogs/dialogs_map.cpp /dialogs/dialogs_map.h
#	/dialogs/dialogs_message.cpp /dialogs/dialogs_message.h
#	/dialogs/dialogs_party.cpp /dialogs/dialogs_party.h
#	/dialogs/dialogs_query.cpp /dialogs/dialogs_query.h
#	/dialogs/dialogs_quests.cpp /dialogs/dialogs_quests.h
#	/dialogs/dialogs_quick_fight.cpp /dialogs/dialogs_quick_fight.h
#	/dialogs/dialogs_quick_ref.cpp /dialogs/dialogs_quick_ref.h
#	/dialogs/dialogs_spells.cpp /dialogs/dialogs_spells.h
#	/dialogs/dialogs_whowill.cpp /dialogs/dialogs_whowill.h
#	/dialogs/please_wait.cpp /dialogs/please_wait.h
#	/swordsofxeen/swordsofxeen.cpp /swordsofxeen/swordsofxeen.h
#	/swordsofxeen/swordsofxeen_menu.cpp /swordsofxeen/swordsofxeen_menu.h
#	/worldofxeen/clouds_cutscenes.cpp /worldofxeen/clouds_cutscenes.h
#	/worldofxeen/darkside_cutscenes.cpp /worldofxeen/darkside_cutscenes.h
#	/worldofxeen/worldofxeen.cpp /worldofxeen/worldofxeen.h
#	/worldofxeen/worldofxeen_cutscenes.cpp /worldofxeen/worldofxeen_cutscenes.h
#	/worldofxeen/worldofxeen_menu.cpp /worldofxeen/worldofxeen_menu.h
#	/character.cpp /character.h
#	/combat.cpp /combat.h
#	/cutscenes.cpp /cutscenes.h
#	/debugger.cpp /debugger.h
#	/detection.h
#	/detection_tables.h
#	/events.cpp /events.h
#	/files.cpp /files.h
#	/font.cpp /font.h
#	/interface.cpp /interface.h
#	/interface_minimap.cpp /interface_minimap.h
#	/interface_scene.cpp /interface_scene.h
#	/item.cpp /item.h
#	/locations.cpp /locations.h
#	/map.cpp /map.h
#	/metaengine.cpp
#	/module.mk
#	/party.cpp /party.h
#	/patcher.cpp /patcher.h
#	/resources.cpp /resources.h
#	/saves.cpp /saves.h
#	/screen.cpp /screen.h
#	/scripts.cpp /scripts.h
#	/sound.cpp /sound.h
#	/sound_driver.cpp /sound_driver.h
#	/sound_driver_adlib.cpp /sound_driver_adlib.h
#	/spells.cpp /spells.h
#	/sprites.cpp /sprites.h
#	/subtitles.cpp /subtitles.h
#	/window.cpp /window.h
#	/xeen.cpp /xeen.h
#	/xsurface.cpp /xsurface.h
#)
#
#add_engine(zvision
#	/core/clock.cpp /core/clock.h
#	/core/console.cpp /core/console.h
#	/core/events.cpp
#	/file/lzss_read_stream.cpp /file/lzss_read_stream.h
#	/file/save_manager.cpp /file/save_manager.h
#	/file/search_manager.cpp /file/search_manager.h
#	/file/zfs_archive.cpp /file/zfs_archive.h
#	/graphics/cursors/cursor.cpp /graphics/cursors/cursor.h
#	/graphics/cursors/cursor_manager.cpp /graphics/cursors/cursor_manager.h
#	/graphics/effects/fog.cpp /graphics/effects/fog.h
#	/graphics/effects/light.cpp /graphics/effects/light.h
#	/graphics/effects/wave.cpp /graphics/effects/wave.h
#	/graphics/graphics_effect.h
#	/graphics/render_manager.cpp /graphics/render_manager.h
#	/graphics/render_table.cpp /graphics/render_table.h
#	/scripting/controls/fist_control.cpp /scripting/controls/fist_control.h
#	/scripting/controls/hotmov_control.cpp /scripting/controls/hotmov_control.h
#	/scripting/controls/input_control.cpp /scripting/controls/input_control.h
#	/scripting/controls/lever_control.cpp /scripting/controls/lever_control.h
#	/scripting/controls/paint_control.cpp /scripting/controls/paint_control.h
#	/scripting/controls/push_toggle_control.cpp /scripting/controls/push_toggle_control.h
#	/scripting/controls/safe_control.cpp /scripting/controls/safe_control.h
#	/scripting/controls/save_control.cpp /scripting/controls/save_control.h
#	/scripting/controls/slot_control.cpp /scripting/controls/slot_control.h
#	/scripting/controls/titler_control.cpp /scripting/controls/titler_control.h
#	/scripting/effects/animation_effect.cpp /scripting/effects/animation_effect.h
#	/scripting/effects/distort_effect.cpp /scripting/effects/distort_effect.h
#	/scripting/effects/music_effect.cpp /scripting/effects/music_effect.h
#	/scripting/effects/region_effect.cpp /scripting/effects/region_effect.h
#	/scripting/effects/syncsound_effect.cpp /scripting/effects/syncsound_effect.h
#	/scripting/effects/timer_effect.cpp /scripting/effects/timer_effect.h
#	/scripting/effects/ttytext_effect.cpp /scripting/effects/ttytext_effect.h
#	/scripting/actions.cpp /scripting/actions.h
#	/scripting/control.cpp /scripting/control.h
#	/scripting/inventory.cpp
#	/scripting/menu.cpp /scripting/menu.h
#	/scripting/puzzle.h
#	/scripting/scr_file_handling.cpp
#	/scripting/script_manager.cpp /scripting/script_manager.h
#	/scripting/scripting_effect.h
#	/sound/midi.cpp /sound/midi.h
#	/sound/zork_raw.cpp /sound/zork_raw.h
#	/text/string_manager.cpp /text/string_manager.h
#	/text/subtitles.cpp /text/subtitles.h
#	/text/text.cpp /text/text.h
#	/text/truetype_font.cpp /text/truetype_font.h
#	/video/rlf_decoder.cpp /video/rlf_decoder.h
#	/video/video.cpp
#	/video/zork_avi_decoder.cpp /video/zork_avi_decoder.h
#	/detection.h
#	/detection_tables.h
#	/metaengine.cpp
#	/module.mk
#	/zvision.cpp /zvision.h
#)

# Libraries
target_link_libraries(scummvm ${SDL2_LIBRARIES} ${SCUMMVM_LIBS})
if (WIN32)
	target_sources(scummvm PUBLIC C:/Users/sariel/Projects/Fun/ScummVM/scummvm/dists/scummvm.rc)
	target_link_libraries(scummvm winmm)
endif()

if (WIN32)
	target_link_libraries(scummvm sapi ole32)
endif()
set_property(TARGET scummvm PROPERTY CXX_STANDARD 11)
set_property(TARGET scummvm PROPERTY CXX_STANDARD_REQUIRED ON)
