cmake_minimum_required(VERSION 2.6)

if( APPLE )
    # Needed to force use of full library paths
    set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
endif( APPLE )

set(PROJECT_NAME "thrift_js_server")
project(${PROJECT_NAME})

set(CMAKE_MODULE_PATH 
    ${CMAKE_MODULE_PATH}
)

if(UNIX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DHAVE_INTTYPES_H -DHAVE_NETINET_IN_H")
    include_directories("/usr/local/include/thrift")
endif()

include_directories(${CMAKE_SOURCE_DIR}/gen-cpp)

# GENERATE_STUBS( shared.thrift cpp )
# GENERATE_STUBS( tutorial.thrift cpp )

# If the generator supports multiple configurations (e.g., MSVC and Xcode)
# limit it to the two standard modes
if(CMAKE_CONFIGURATION_TYPES)
    # Just use Release and Debug modes
    set(CMAKE_CONFIGURATION_TYPES Debug Release)
    # Force the variable to be used
    set(CMAKE_CONFIGURATION_TYPES "${CMAKE_CONFIGURATION_TYPES}" CACHE 
    STRING "Reset the configuration types" FORCE)
endif()

#####################################################################
# Platform Config -- Apple
#####################################################################
if(APPLE)
    # C++0x requires Clang
    # Recognize the new C++ standard. (requires LLVM 3.0/Clang)
    set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LANGUAGE_STANDARD "c++0x")
    # Silence warnings about auto keyword
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-c++11-extensions -Wno-c++11-narrowing")
endif()

#####################################################################
# Platform Config -- Win32
#####################################################################
if(WIN32)
    LIST(APPEND CMAKE_FIND_LIBRARY_PREFIXES "lib" "")
    # avoid window's min/max functions clobbering the standard versions
    add_definitions(-DNOMINMAX)
    add_definitions(-D_USE_MATH_DEFINES)
endif()
IF(MSVC)
       #SET(CMAKE_C_FLAGS_DEBUG   "${CMAKE_C_FLAGS_DEBUG} /EHa")
       SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /EHa")
       SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /EHa")
ENDIF(MSVC)

#####################################################################
# BOOST
#####################################################################
set(BOOST_MIN_VERSION "1.44")
list(APPEND PROJECT_BOOST_COMPONENTS system thread) 
 #unit_test_framework)
 #graph
find_package(Boost ${BOOST_MIN_VERSION} COMPONENTS ${PROJECT_BOOST_COMPONENTS})

if( Boost_FOUND )
    INCLUDE_DIRECTORIES(${Boost_INCLUDE_DIRS})
    ADD_DEFINITIONS( ${Boost_LIB_DIAGNOSTIC_DEFINITIONS} )
    ADD_DEFINITIONS( "-DHAS_BOOST" )
    LINK_DIRECTORIES(${Boost_LIBRARY_DIRS})
else()
    #a bit ugly but for now boost is included in i2p redistributable
    if(WIN32)
        SET(Boost_INCLUDE_DIRS "${I2P_INCLUDE}")
        SET(Boost_LIBRARY_DIRS "${I2P_LIB}")
        INCLUDE_DIRECTORIES(${I2P_INCLUDE})
        LINK_DIRECTORIES(${I2P_LIB})
        MESSAGE("BOOST OK")
    else()
        message(SEND_ERROR "Failed to find boost!")
    endif()
endif()

MESSAGE(${Boost_LIBRARIES})

###################################################################
# Project files
#####################################################################
set( THRIFT_HDRS
    ./gen-cpp/Calculator.h
    ./gen-cpp/shared_constants.h
    ./gen-cpp/shared_types.h
    ./gen-cpp/SharedService.h
    ./gen-cpp/tutorial_constants.h
    ./gen-cpp/tutorial_types.h
)
SOURCE_GROUP( ThriftGeneratedHeaders FILES ${THRIFT_HDRS})

set( THRIFT_SRCS
    ./gen-cpp/Calculator.cpp    
    ./gen-cpp/shared_constants.cpp
    ./gen-cpp/shared_types.cpp
    ./gen-cpp/SharedService.cpp    
    ./gen-cpp/tutorial_constants.cpp
    ./gen-cpp/tutorial_types.cpp
)
SOURCE_GROUP( ThriftGeneratedSources FILES ${THRIFT_SRCS})

set( HDRS
    ${THRIFT_HDRS}
    ./TVirtualTransport.h
    ./THttpServer.h
    ./THttpTransport.h
)

set( SRCS
    ${THRIFT_SRCS}
    ./CppServer.cpp
    ./THttpServer.cpp  
    ./THttpTransport.cpp
)

####################################################################
# Executables
#####################################################################
# add_executable( ${PROJECT_NAME} 
#    ${HDRS} ${SRCS} 
#    ${SERVER_HDRS} ${SERVER_SRCS} 
# )

add_executable( CppServer 
  ./CppServer.cpp
  ${HDRS} 
  ${SRCS}     
)

if(UNIX)
  target_link_libraries(CppServer
    ${Boost_LIBRARIES}
    thrift
    )
endif()

if(WIN32)
  target_link_libraries(CppServer
    ${Boost_LIBRARIES}
    )
endif()