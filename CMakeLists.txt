cmake_minimum_required(VERSION 3.21)

if(DEFINED LIB_PARENT_TARGET) # Use same target

  target_include_directories( ${LIB_PARENT_TARGET} PRIVATE include)

  target_sources( ${LIB_PARENT_TARGET} PRIVATE
      src/Module_Template.c)

else() # Create new target

  # add name of the library
  set(LIBRARY_NAME MODULETEMPLATE)

  # Bare-metal toolchains (arm-none-eabi) cannot link a test executable without
  # startup code / linker script: let CMake's compiler check build a static lib instead.
  set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

  PROJECT(${LIBRARY_NAME} C ASM)

  if (NOT PROJECT_IS_TOP_LEVEL)
    message(STATUS "This project [${LIBRARY_NAME}] has a top-level one called [${CMAKE_PROJECT_NAME}]")
  else()
    # Stand-alone build: provide the dependencies a parent project would normally supply
    message(STATUS "[${LIBRARY_NAME}] stand-alone build: using cfg_ex/ and module_libs/")

    # Additional modules (fetched by west into module_libs/, eg: GenUtl)
    #
    # add_library(GENUTL STATIC
    #   ${CMAKE_CURRENT_SOURCE_DIR}/module_libs/src/GenUtl.c
    # )
    # 
    # target_include_directories(GENUTL PUBLIC
    #   ${CMAKE_CURRENT_SOURCE_DIR}/module_libs/include
    # )
    # 
    # target_link_libraries(${LIBRARY_NAME}_CFG PUBLIC GENUTL)

    # Example configuration (mocked I2C callbacks)
    add_library(${LIBRARY_NAME}_CFG STATIC
      ${CMAKE_CURRENT_SOURCE_DIR}/cfg_ex/Module_Template_Cfg.c
    )
    target_include_directories(${LIBRARY_NAME}_CFG
      PUBLIC  ${CMAKE_CURRENT_SOURCE_DIR}/cfg_ex
      PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include
    )
    
  endif()

  # add the library source files
  add_library(${LIBRARY_NAME} STATIC
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Module_Template.c
    # ...
  )

  # add private include directory if necessary
  target_include_directories(${LIBRARY_NAME}
    PRIVATE
    # ...
  )

  # add public include directory
  target_include_directories(${LIBRARY_NAME}
    PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_SOURCE_DIR}/module_libs/include
  )

  # add library with cfg include
  target_link_libraries(${LIBRARY_NAME} PUBLIC ${LIBRARY_NAME}_CFG)

  # pow() lives in libm
  target_link_libraries(${LIBRARY_NAME} PUBLIC m)

endif()
