CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

PROJECT(QBus_py CXX)

SET(CMAKE_VERBOSE_MAKEFILE TRUE)
SET(CMAKE_CXX_FLAGS "-O2 -DNDEBUG")

SET(CXX_SRC_DIR "${PROJECT_SOURCE_DIR}/../cxx/src")

# find include directories and libraries
FIND_PACKAGE(PythonLibs REQUIRED)
MESSAGE(STATUS "PYTHON_INCLUDE_DIRS=${PYTHON_INCLUDE_DIRS}")
MESSAGE(STATUS "PYTHON_LIBRARIES=${PYTHON_LIBRARIES}")

SET(CMAKE_FIND_LIBRARY_SUFFIXES .a)
SET(CMAKE_PREFIX_PATH "${CXX_SRC_DIR}/../thirdparts/local")

FIND_PATH(LIBRDKAFKA_INCLUDE_DIR NAMES "librdkafka/rdkafka.h")
FIND_LIBRARY(LIBRDKAFKA_LIB NAMES rdkafka)

IF(NOT LIBRDKAFKA_INCLUDE_DIR OR NOT LIBRDKAFKA_LIB)
    MESSAGE(FATAL_ERROR "Can't find librdkafka")
ENDIF()

MESSAGE(STATUS "LIBRDKAFKA_INCLUDE_DIR=${LIBRDKAFKA_INCLUDE_DIR}")
MESSAGE(STATUS "LIBRDKAFKA_LIB=${LIBRDKAFKA_LIB}")

FIND_PATH(LOG4CPLUS_INCLUDE_DIR NAMES "log4cplus/logger.h")
FIND_LIBRARY(LOG4CPLUS_LIB NAMES NAMES log4cplus)

IF(NOT LOG4CPLUS_INCLUDE_DIR OR NOT LOG4CPLUS_LIB)
    MESSAGE(FATAL_ERROR "Can't find log4cplus")
ENDIF()

MESSAGE(STATUS "LOG4CPLUS_INCLUDE_DIR=${LOG4CPLUS_INCLUDE_DIR}")
MESSAGE(STATUS "LOG4CPLUS_LIB=${LOG4CPLUS_LIB}")

# compile target shared library
SET(PROJECT_ALL_FILES
    ${CXX_SRC_DIR}/../util/logger.cc
    ${CXX_SRC_DIR}/qbus_helper.cc
    ${CXX_SRC_DIR}/qbus_config.cc
    ${CXX_SRC_DIR}/qbus_record_msg.cc
    ${CXX_SRC_DIR}/qbus_consumer_imp.cc
    ${CXX_SRC_DIR}/qbus_consumer.cc
    ${CXX_SRC_DIR}/qbus_producer.cc
    ${CXX_SRC_DIR}/qbus_topic_partition_set.cc
    ${CXX_SRC_DIR}/qbus_rdkafka.cc
    qbus_wrap.cxx
    )

SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR})
ADD_LIBRARY(${PROJECT_NAME} SHARED ${PROJECT_ALL_FILES})

INCLUDE_DIRECTORIES(${CXX_SRC_DIR})
INCLUDE_DIRECTORIES(${CXX_SRC_DIR}/..)
INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_DIRS})
INCLUDE_DIRECTORIES(${LIBRDKAFKA_INCLUDE_DIR})
INCLUDE_DIRECTORIES(${LOG4CPLUS_INCLUDE_DIR})

TARGET_LINK_LIBRARIES(${PROJECT_NAME}
    ${PYTHON_LIBRARIES}
    ${LIBRDKAFKA_LIB}
    ${LOG4CPLUS_LIB}
    dl
    pthread
    rt
    z
    )
