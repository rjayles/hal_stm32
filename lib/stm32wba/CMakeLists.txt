# Copyright (c) 2024 STMicroelectronics
#
# SPDX-License-Identifier: Apache-2.0

zephyr_compile_definitions( -DBLE )

zephyr_include_directories(BLE_TransparentMode/Core/Inc)
zephyr_include_directories(BLE_TransparentMode/STM32_WPAN/Target)
zephyr_include_directories(BLE_TransparentMode/System/Config/Debug_GPIO)
zephyr_include_directories(BLE_TransparentMode/System/Config/Log)
zephyr_include_directories(Common/WPAN/Interfaces)
zephyr_include_directories(Common/WPAN/Modules)
zephyr_include_directories(Common/WPAN/Modules/Flash)
zephyr_include_directories(Common/WPAN/Modules/RTDebug)
zephyr_include_directories(Common/WPAN/Modules/Log)
zephyr_include_directories(STM32_WPAN)
zephyr_include_directories(STM32_WPAN/ble/stack/include)
zephyr_include_directories(STM32_WPAN/ble/stack/include/auto)
zephyr_include_directories(STM32_WPAN/link_layer/ll_cmd_lib/config/ble_full)
zephyr_include_directories(STM32_WPAN/link_layer/ll_cmd_lib/inc)
zephyr_include_directories(STM32_WPAN/link_layer/ll_cmd_lib/inc/_40nm_reg_files)
zephyr_include_directories(STM32_WPAN/link_layer/ll_sys/inc)
zephyr_include_directories(Utilities/misc)
zephyr_include_directories(Utilities/tim_serv)
zephyr_include_directories(Utilities/trace/adv_trace)

zephyr_sources(BLE_TransparentMode/STM32_WPAN/Target/bpka.c)
zephyr_sources(BLE_TransparentMode/STM32_WPAN/Target/linklayer_plat.c)
zephyr_sources(BLE_TransparentMode/STM32_WPAN/Target/ll_sys_if.c)
zephyr_sources(BLE_TransparentMode/STM32_WPAN/Target/power_table.c)
zephyr_sources(Common/WPAN/Modules/Log/log_module.c)
zephyr_sources(Common/WPAN/Interfaces/hw_pka.c)
zephyr_sources(Common/WPAN/Interfaces/hw_pka_p256.c)
zephyr_sources(Common/WPAN/Modules/scm.c)
zephyr_sources(Common/WPAN/Modules/RTDebug/RTDebug.c)
zephyr_sources(STM32_WPAN/link_layer/ll_sys/src/ll_sys_cs.c)
zephyr_sources(STM32_WPAN/link_layer/ll_sys/src/ll_sys_dp_slp.c)
zephyr_sources(STM32_WPAN/link_layer/ll_sys/src/ll_sys_intf.c)
zephyr_sources(STM32_WPAN/link_layer/ll_sys/src/ll_sys_startup.c)

if(CONFIG_FLASH)
  zephyr_sources(Common/WPAN/Modules/Flash/flash_manager.c)
  zephyr_sources(Common/WPAN/Modules/Flash/flash_driver.c)
  zephyr_sources(Common/WPAN/Modules/stm_list.c)
  zephyr_sources(Common/WPAN/Modules/Flash/rf_timing_synchro.c)
endif()

set(STM32WBA_BLE_LIB_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../zephyr/blobs/stm32wba/lib)
set(STM32WBA_BLE_LIB_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/STM32_WPAN/ble/stack)
set(STM32WBA_LL_LIB_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/STM32_WPAN/link_layer/ll_cmd_lib)

add_library(stm32wba_ble_lib STATIC IMPORTED GLOBAL)
add_library(stm32wba_ll_lib STATIC IMPORTED GLOBAL)

add_dependencies(
  stm32wba_ble_lib
  stm32wba_ll_lib
)
set_target_properties(
  stm32wba_ble_lib PROPERTIES IMPORTED_LOCATION ${STM32WBA_BLE_LIB_DIR}/stm32wba_ble_stack_llo.a
)

# Setting the right Cube define according to the Zephyr configuration
if(NOT CONFIG_BT_STM32WBA_USE_TEMP_BASED_CALIB)
  zephyr_compile_definitions( -DUSE_TEMPERATURE_BASED_RADIO_CALIBRATION=0 )
endif()

# Selecting the proper version of link layer lib according the soc
# Checking all the soc variants and not simply relying on board name 

if(CONFIG_SOC_STM32WBA65XX)
  message(STATUS "STM32WBA6 link layer lib selected")
  set(LL_LIB "WBA6_LinkLayer_BLE_Full_lib.a")
endif()

if(CONFIG_SOC_STM32WBA55XX OR SOC_STM32WBA52XX)
  message(STATUS "STM32WBA5 link layer lib selected")
  set(LL_LIB "LinkLayer_BLE_Full_lib.a")
endif()

# Using the selected version of link layer lib
set_target_properties(stm32wba_ll_lib PROPERTIES IMPORTED_LOCATION ${STM32WBA_BLE_LIB_DIR}/${LL_LIB})
set_target_properties(stm32wba_ble_lib PROPERTIES INTERFACE_INCLUDE_DIRECTORIES ${STM32WBA_BLE_LIB_INCLUDE_DIR})
set_target_properties(stm32wba_ll_lib PROPERTIES INTERFACE_INCLUDE_DIRECTORIES ${STM32WBA_LL_LIB_INCLUDE_DIR})

target_link_libraries(app PUBLIC stm32wba_ble_lib)
target_link_libraries(app PUBLIC stm32wba_ll_lib)
