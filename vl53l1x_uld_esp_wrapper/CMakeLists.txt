idf_component_register(
  SRCS 
    "core/src/VL53L1X_api.c"
    "core/src/VL53L1X_calibration.c"
    "core/src/vl53l1_platform.c"
    "esp_wrapper/src/i2c_handler.c"
    "esp_wrapper/src/i2c_device_handler.c"
    "esp_wrapper/src/vl53l1x.c"

  INCLUDE_DIRS
    "core/include"
    "esp_wrapper/include"

  REQUIRES
    esp_driver_gpio  
    esp_driver_i2c
)