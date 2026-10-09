# Keep Prospector status data in advertisements while idle.
# Apply only to the Poached Egg central build.
if(CONFIG_SHIELD_POACHED_EGGS_LEFT AND CONFIG_ZMK_STATUS_ADVERTISEMENT)
  set(prospector_status_source
      "${ZEPHYR_PROSPECTOR_ZMK_MODULE_MODULE_DIR}/src/status_advertisement.c")

  if(NOT EXISTS "${prospector_status_source}")
    message(FATAL_ERROR "Prospector status source was not found")
  endif()

  file(READ "${prospector_status_source}" prospector_status_content)
  set(prospector_status_before "bool send_name = !in_burst &&")
  set(prospector_status_after "bool send_name = is_active && !in_burst &&")

  string(FIND "${prospector_status_content}" "${prospector_status_after}"
         prospector_status_already_fixed)
  if(prospector_status_already_fixed GREATER_EQUAL 0)
    message(STATUS "Poached Egg: Prospector idle status fix already applied")
  else()
    string(FIND "${prospector_status_content}" "${prospector_status_before}"
           prospector_status_patch_position)
    if(prospector_status_patch_position LESS 0)
      message(FATAL_ERROR "Prospector source differs from the expected v2.2.3 code")
    endif()

    string(REPLACE "${prospector_status_before}" "${prospector_status_after}"
           prospector_status_content "${prospector_status_content}")
    file(WRITE "${prospector_status_source}" "${prospector_status_content}")
    message(STATUS "Poached Egg: applied Prospector idle status fix")
  endif()
endif()
