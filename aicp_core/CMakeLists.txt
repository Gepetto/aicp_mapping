cmake_minimum_required(VERSION 2.8.3)

project(aicp_core)
add_compile_options(-std=c++11)
set(AICP_CORE_LIB ${PROJECT_NAME})

# Find system dependencies (buildtime dependencies)
find_package(catkin REQUIRED)
find_package(libpointmatcher REQUIRED)
find_package(PCL REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(octomap REQUIRED)
find_package(OpenCV 3.0 REQUIRED)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES aicpRegistration# libraries which will be visible outside 
            aicpOverlap
            aicpClassification
            aicpUtils 
  CATKIN_DEPENDS
  DEPENDS libpointmatcher
          PCL
          OCTOMAP
          OpenCV) # dependencies which will be taken outside
  
  
#################
# Utils library #
#################
add_library(aicpUtils SHARED src/utils/common.cpp
                             src/utils/timing.cpp
                             src/utils/cloudIO.cpp
                             src/utils/fileIO.cpp
                             src/utils/icpMonitor.cpp
                             src/utils/filteringUtils.cpp)
TARGET_INCLUDE_DIRECTORIES(aicpUtils SYSTEM PUBLIC ${libpointmatcher_INCLUDE_DIRS} 
                                                   ${PCL_INCLUDE_DIRS}
                                                   include)
TARGET_INCLUDE_DIRECTORIES(aicpUtils PUBLIC $<INSTALL_INTERFACE:include/aicp_utils>)
target_link_libraries(aicpUtils ${libpointmatcher_LIBRARIES}
                                ${PCL_LIBRARIES})


##################
# Octree-overlap #
##################
add_library(aicpOverlap SHARED src/overlap/octrees_overlap.cpp)
TARGET_INCLUDE_DIRECTORIES(aicpOverlap SYSTEM PUBLIC ${OCTOMAP_INCLUDE_DIRS}
                                                     ${PCL_INCLUDE_DIRS}
                                                     include)
TARGET_INCLUDE_DIRECTORIES(aicpOverlap PUBLIC $<INSTALL_INTERFACE:include/aicp_overlap>)
target_link_libraries(aicpOverlap ${PCL_LIBRARIES}
                                  ${OCTOMAP_LIBRARIES})


#################
# Classification #
#################
add_library(aicpClassification SHARED src/classification/svm.cpp)
TARGET_INCLUDE_DIRECTORIES(aicpClassification SYSTEM PUBLIC ${EIGEN3_INCLUDE_DIRS}
                                                            include)
TARGET_INCLUDE_DIRECTORIES(aicpClassification PUBLIC $<INSTALL_INTERFACE:include/aicp_classification>)
target_link_libraries(aicpClassification ${OpenCV_LIBS})
                                         
add_executable(aicp_classification_main src/classification/main.cpp)
target_link_libraries(aicp_classification_main aicpClassification
                                               aicpUtils
                                               yaml-cpp)
add_executable(aicp_classification_example src/classification/example.cpp)
target_link_libraries(aicp_classification_example aicpClassification
                                                  aicpUtils
                                                  yaml-cpp)
                                                  
                                                  
################  
# Registration #
################
add_library(aicpRegistration SHARED src/registration/pointmatcher_registration.cpp
                                    src/registration/aligned_cloud.cpp
                                    src/registration/aligned_clouds_graph.cpp
                                    src/registration/app.cpp
                                    src/registration/yaml_configurator.cpp)
TARGET_INCLUDE_DIRECTORIES(aicpRegistration SYSTEM PUBLIC ${libpointmatcher_INCLUDE_DIRS}
                                                          ${PCL_INCLUDE_DIRS}
                                                          include)
TARGET_INCLUDE_DIRECTORIES(aicpRegistration PUBLIC $<INSTALL_INTERFACE:include/aicp_registration>)
target_link_libraries(aicpRegistration ${libpointmatcher_LIBRARIES}
                                       ${PCL_LIBRARIES}
                                       yaml-cpp)



#############
# Unit test #
#############
add_executable(aicp_test test/aicp_test.cpp)
target_link_libraries(aicp_test aicpRegistration aicpOverlap aicpClassification aicpUtils)


#############
## Install ##
#############

install(TARGETS aicpRegistration aicpOverlap aicpUtils aicpClassification 
        ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

install(DIRECTORY include/aicp_registration/
        DESTINATION include/aicp_registration)

install(DIRECTORY include/aicp_overlap/
        DESTINATION include/aicp_overlap)

install(DIRECTORY include/aicp_classification/
        DESTINATION include/aicp_classification)

install(DIRECTORY include/aicp_utils/
        DESTINATION include/aicp_utils)

INSTALL(DIRECTORY config
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

###############  
# Extra Tools #
###############
#add_subdirectory(src/tools)
