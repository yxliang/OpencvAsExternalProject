cmake_minimum_required(VERSION 3.0)

project(opencv VERSION 3.4.0)
string(TOUPPER ${PROJECT_NAME} PROJECT_NAME_UPPER)

option(BUILD_${PROJECT_NAME_UPPER} "Build ${PROJECT_NAME}" ${BUILD_ALL})
option(${PROJECT_NAME_UPPER}_BUILD_SHARED_LIBS "Build shared libraries for ${PROJECT_NAME}" ${BUILD_SHARED_LIBS})
option(${PROJECT_NAME_UPPER}_WITH_CUDA "Build ${PROJECT_NAME} with CUDA" ON)
option(${PROJECT_NAME_UPPER}_CLONE_FFMPEG "Clone ffmpeg windows dependency instead of downloading it" OFF)
option(${PROJECT_NAME_UPPER}_WITH_PYTHON "Build ${PROJECT_NAME} with python support" ON)
option(${PROJECT_NAME_UPPER}_WITH_CONTRIB "Build ${PROJECT_NAME} with contrib module" ON)

set( MKL_ROOT_DIR "C:/Program Files (x86)/IntelSWTools/compilers_and_libraries/windows/mkl" )
set( JPEG_INCLUDE_DIR "c:/Program Files/libjpeg-turbo64/include" )
set( JPEG_LIBRARY "c:/Program Files/libjpeg-turbo64/lib/jpeg-static.lib" )
set( 3rdlib_DIR "D:/caffe-builder/build/libraries")

set(${PROJECT_NAME_UPPER}_CMAKE_ARGS
    -DBUILD_DOCS=OFF
    -DBUILD_PERF_TESTS=OFF
    -DBUILD_TESTS=OFF
	-DBUILD_opencv_apps=OFF
	-DOPENCV_ENABLE_NONFREE=ON
    -DBUILD_PACKAGE=OFF
    -DBUILD_WITH_STATIC_CRT=OFF
    -DWITH_CUDA:BOOL=${${PROJECT_NAME_UPPER}_WITH_CUDA}
    -DCUDA_ARCH_BIN="3.0 3.5 3.7 5.0 5.2 6.0 6.1"  #"3.0 3.5 3.7 5.0 5.2 6.0 6.1 7.0"
    -DCUDA_ARCH_PTX=" " #"6.1"
    -DBUILD_opencv_python2:BOOL=${${PROJECT_NAME_UPPER}_WITH_PYTHON}
    -DBUILD_opencv_python3:BOOL=${${PROJECT_NAME_UPPER}_WITH_PYTHON}
    # -DBUILD_ZLIB=OFF
    # -DZLIB_INCLUDE_DIR=${ZLIB_INCLUDE_DIR}
    # -DZLIB_LIBRARY_RELEASE=${ZLIB_LIBRARY_RELEASE}
    # -DZLIB_LIBRARY_DEBUG=${ZLIB_LIBRARY_DEBUG}
	-DMKL_ROOT_DIR=${MKL_ROOT_DIR}
	-DBUILD_JPEG:BOOL=OFF
	-DWITH_JPEG:BOOL=ON
 	-DJPEG_INCLUDE_DIR=${JPEG_INCLUDE_DIR}
	-DJPEG_LIBRARY=${JPEG_LIBRARY}
	-DGLOG_INCLUDE_DIR=${3rdlib_DIR}/include
	-DGLOG_LIBRARY=${3rdlib_DIR}/lib/glog.lib
   )

# set(${PROJECT_NAME_UPPER}_DEPENDS zlib)

if(CMAKE_CL_64)
    set(OpenCV_ARCH x64)
    set(OpenCV_TBB_ARCH intel64)
elseif((CMAKE_GENERATOR MATCHES "ARM") OR ("${arch_hint}" STREQUAL "ARM") OR (CMAKE_VS_EFFECTIVE_PLATFORMS MATCHES "ARM|arm"))
# see Modules/CmakeGenericSystem.cmake
    set(OpenCV_ARCH ARM)
else()
    set(OpenCV_ARCH x86)
    set(OpenCV_TBB_ARCH ia32)
endif()

set(OpenCV_ARCH ${OpenCV_ARCH} PARENT_SCOPE)

if(MSVC_VERSION EQUAL 1400)
    set(OpenCV_RUNTIME vc8)
elseif(MSVC_VERSION EQUAL 1500)
    set(OpenCV_RUNTIME vc9)
elseif(MSVC_VERSION EQUAL 1600)
    set(OpenCV_RUNTIME vc10)
elseif(MSVC_VERSION EQUAL 1700)
    set(OpenCV_RUNTIME vc11)
elseif(MSVC_VERSION EQUAL 1800)
    set(OpenCV_RUNTIME vc12)
elseif(MSVC_VERSION EQUAL 1900)
    set(OpenCV_RUNTIME vc14)
endif()

set(OpenCV_RUNTIME ${OpenCV_RUNTIME} PARENT_SCOPE)

if(${PROJECT_NAME_UPPER}_BUILD_SHARED_LIBS)
    set(OpenCV_STATIC OFF)
    set(OpenCV_LIB_PREFIX lib)
else()
    set(OpenCV_STATIC ON)
    set(OpenCV_LIB_PREFIX staticlib)
endif()



if(BUILD_${PROJECT_NAME_UPPER})

    set(${PROJECT_NAME_UPPER}_CONFIG
		"
		# ${PROJECT_NAME_UPPER} config
		set(${PROJECT_NAME_UPPER}_DIR \${CMAKE_CURRENT_LIST_DIR}  CACHE PATH \"\")
		set(${PROJECT_NAME}_DIR \${CMAKE_CURRENT_LIST_DIR}  CACHE PATH \"\")
		set(OpenCV_DIR \${CMAKE_CURRENT_LIST_DIR}  CACHE PATH \"\")
		set(OpenCV_STATIC ${OpenCV_STATIC} CACHE BOOL \"\")
		"
		PARENT_SCOPE)

    include(ExternalProject)
    if(${PROJECT_NAME_UPPER}_CLONE_FFMPEG)
        # workaround to get ffmpeg under firewall restrictions
        set(_ext_proj_ffmpeg ${PROJECT_NAME}_clone_ffmpeg)
        ExternalProject_Add(
            ${_ext_proj_ffmpeg}
            DOWNLOAD_COMMAND git clone https://github.com/opencv/opencv_3rdparty.git -c core.autocrlf=false
            UPDATE_COMMAND cd ${DOWNLOAD_DIR}/opencv_3rdparty && git checkout ffmpeg/master_20150703
            DOWNLOAD_DIR ${DOWNLOAD_DIR}
            CONFIGURE_COMMAND ""
            BUILD_COMMAND ""
            INSTALL_COMMAND ""
        )
        ExternalProject_Get_Property("${_ext_proj_ffmpeg}" SOURCE_DIR)
        set(OPENCV_FFMPEG_URL file:///${DOWNLOAD_DIR}/opencv_3rdparty/ffmpeg/)
        list(APPEND ${PROJECT_NAME_UPPER}_CMAKE_ARGS -DOPENCV_FFMPEG_URL=${OPENCV_FFMPEG_URL})
        list(APPEND ${PROJECT_NAME}_targets  ${_ext_proj_ffmpeg})
    endif()

    if(${PROJECT_NAME_UPPER}_WITH_CONTRIB)
        ExternalProject_Add(
            ${PROJECT_NAME}_download_contrib
            # URL  "https://github.com/opencv/opencv_contrib/archive/3.1.0.tar.gz"
            # DOWNLOAD_NAME opencv-contrib-3.1.0.zip
			URL ${CMAKE_SOURCE_DIR}/download/opencv_contrib-3.4.1.zip
            DOWNLOAD_DIR ${DOWNLOAD_DIR}
            CONFIGURE_COMMAND ""
            BUILD_COMMAND ""
            INSTALL_COMMAND ""
        )

        ExternalProject_Get_Property("${PROJECT_NAME}_download_contrib" SOURCE_DIR)

        list(APPEND
            ${PROJECT_NAME_UPPER}_CMAKE_ARGS
            -DBUILD_opencv_aruco:BOOL=OFF
            -DBUILD_opencv_bgsegm:BOOL=OFF
            -DBUILD_opencv_bioinspired:BOOL=OFF
            -DBUILD_opencv_ccalib:BOOL=OFF
            -DBUILD_opencv_cnn_3dobj:BOOL=OFF
            -DBUILD_opencv_contrib_world:BOOL=OFF
            -DBUILD_opencv_cvv:BOOL=OFF
            -DBUILD_opencv_datasets:BOOL=ON
            -DBUILD_opencv_dnn_modern:BOOL=ON
            -DBUILD_opencv_dnn_easily_fooled:BOOL=OFF
            -DBUILD_opencv_dpm:BOOL=ON
            -DBUILD_opencv_face:BOOL=ON
            -DBUILD_opencv_freetype:BOOL=OFF
            -DBUILD_opencv_fuzzy:BOOL=OFF
            -DBUILD_opencv_hdf:BOOL=OFF
            -DBUILD_opencv_img_hash:BOOL=OFF
            -DBUILD_opencv_line_descriptor:BOOL=ON
            -DBUILD_opencv_matlab:BOOL=OFF
            -DBUILD_opencv_optflow:BOOL=OFF
            -DBUILD_opencv_phase_unwrapping:BOOL=OFF
            -DBUILD_opencv_plot:BOOL=OFF
            -DBUILD_opencv_reg:BOOL=OFF
            -DBUILD_opencv_rgbd:BOOL=OFF
            -DBUILD_opencv_saliency:BOOL=OFF
            -DBUILD_opencv_sfm:BOOL=OFF
            -DBUILD_opencv_stereo:BOOL=OFF
            -DBUILD_opencv_structured_light:BOOL=OFF
            -DBUILD_opencv_surface_matching:BOOL=OFF
            -DBUILD_opencv_text:BOOL=OFF
            -DBUILD_opencv_tracking:BOOL=OFF
            -DBUILD_opencv_xfeatures2d:BOOL=ON
            -DBUILD_opencv_ximgproc:BOOL=ON
            -DBUILD_opencv_xobjdetect:BOOL=ON
            -DBUILD_opencv_xphoto:BOOL=ON
            -DOPENCV_EXTRA_MODULES_PATH:PATH=${SOURCE_DIR}/modules
        )
        list(APPEND ${PROJECT_NAME}_targets ${PROJECT_NAME}_download_contrib)
    endif()

    ExternalProject_Add(
        ${PROJECT_NAME}_download
        # URL  "https://github.com/opencv/opencv/archive/3.1.0.zip"
        # DOWNLOAD_NAME opencv-3.1.0.zip
		URL ${CMAKE_SOURCE_DIR}/download/opencv-3.4.1.zip
        DOWNLOAD_DIR ${DOWNLOAD_DIR}
        # PATCH_COMMAND ${CMAKE_COMMAND} -E copy_if_different
                      # "${CMAKE_CURRENT_SOURCE_DIR}/patches/CMakeLists.txt"
                      # "CMakeLists.txt"
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
    )
    list(APPEND ${PROJECT_NAME}_targets ${PROJECT_NAME}_download)

    ExternalProject_Get_Property("${PROJECT_NAME}_download" SOURCE_DIR)

    foreach(_config ${BUILD_CONFIGURATION_TYPES})
        string(TOLOWER ${_config} _config_lower)
        ExternalProject_Add(
            ${PROJECT_NAME}_${_config_lower}
            DEPENDS ${PROJECT_NAME}_download
            # make it depend on other configure projects
            # so we don't conflict in downloading ippcv and ffmpeg
            ${${PROJECT_NAME}_targets}
            ${${PROJECT_NAME_UPPER}_DEPENDS}
            SOURCE_DIR ${SOURCE_DIR}
            DOWNLOAD_COMMAND ""
            CMAKE_ARGS -DCMAKE_BUILD_TYPE=${_config}
                    -DBUILD_SHARED_LIBS=${${PROJECT_NAME_UPPER}_BUILD_SHARED_LIBS}
                    -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
                    -DCMAKE_PREFIX_PATH=${CMAKE_INSTALL_PREFIX}
                    -DCMAKE_MODULE_PATH=${CMAKE_MODULE_PATH}
                    -DCMAKE_DEBUG_POSTFIX=d
                    ${${PROJECT_NAME_UPPER}_CMAKE_ARGS}
            BUILD_COMMAND ${CMAKE_COMMAND} --build . --target install --config ${_config}
            INSTALL_COMMAND ${CMAKE_COMMAND} --build . --target install --config ${_config}
        )
        if(PYTHON_EXECUTABLE)
            ExternalProject_Add_Step(${PROJECT_NAME}_${_config_lower} relocate
            COMMAND ${PYTHON_EXECUTABLE}
                    ${CMAKE_CURRENT_SOURCE_DIR}/../../replace_absolute_paths.py
                    ${CMAKE_INSTALL_PREFIX}/share/OpenCV/${OpenCV_ARCH}/${OpenCV_RUNTIME}/${OpenCV_LIB_PREFIX}/OpenCVModules-${_config_lower}.cmake
            DEPENDEES build install
            COMMENT "Running relocation script"
            )
            ExternalProject_Add_StepTargets(${PROJECT_NAME}_${_config_lower} relocate)
        endif()

        ExternalProject_Add_StepTargets(${PROJECT_NAME}_${_config_lower} build install)

        list(APPEND ${PROJECT_NAME}_targets ${PROJECT_NAME}_${_config_lower})
    endforeach()

    add_custom_target(${PROJECT_NAME})
    add_dependencies(${PROJECT_NAME} ${${PROJECT_NAME}_targets})
	# MESSAGE(STATUS "ARGS: ${${PROJECT_NAME_UPPER}_CMAKE_ARGS}")

endif()
