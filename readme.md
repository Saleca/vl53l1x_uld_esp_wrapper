# VL53L1X_ULD_ESP_WRAPPER

I created this wrapper to *hopefully* simplify the use of vl53l1x time of flight sensors in esp-idf

The component is not complete, and **there will be breaking changes**, but there are two potential uses for this component:
- as demonstrated bellow and in the `example.c` its possible to have the sensor configured and measuring with minimal set up required.
- using the raw API, bare in mind that the API requires platform specific implementation for I2C and WAIT function(s) that can be found in `core/src/vl53l1_platform.c` or simply by assigning functions to the pointers at the bottom of the file.

> future chages include  rearrangement of astraction layer (*potential breaking changes*), sensor config handle to include default values for timings and modes of operation, although the aim is to make it simple, I designed it like this to allow multiple sensors under multiple i2c bus but bare in mind i have done very little testing. 

# how to include on your esp-idf project

## using espressif registry

add the dependency and include the component on the cmake file for example
```yml
# /main/idf_component.yml
dependencies:
  idf:
      version: '>=5.5.0'
  saleca/vl53l1x_uld_esp_wrapper: ^0.1.0 # <-- add the component as a dependency
```
```cmake
# /main/CMakeLists.txt
idf_component_register(
    SRCS 
        "main.c"
                    
    INCLUDE_DIRS 
        "."
                    
    REQUIRES
        vl53l1x_uld_esp_wrapper # <-- add the component to the REQUIRES list
        esp_driver_gpio) 
```

## using github
download the component into the project, for example in `/components/vl53l1x_uld_esp_wrapper` and add the component to the REQUIRES lists as shown above

# how to get a measurement

first the `vl53l1x_uld_esp_wrapper` component has to be initialized with at least `scl_gpio` and `sda_gpio` in `vl53l1x_handle.i2c_handle`, this will initialize the platform functions and the `driver/i2c_master.h` component. 
```c
vl53l1x_handle_t vl53l1x_handle = VL53L1X_INIT;
vl53l1x_i2c_handle_t vl53l1x_i2c_handle = VL53L1X_I2C_INIT;
vl53l1x_i2c_handle.scl_gpio = SCL_GPIO;
vl53l1x_i2c_handle.sda_gpio = SDA_GPIO;

vl53l1x_handle.i2c_handle = &vl53l1x_i2c_handle;
if (!vl53l1x_init(&vl53l1x_handle))
{
    ESP_LOGE(TAG, "vl53l1x initialization failed");
    return;
}
```

then to add the sensor(s), just assign the `vl53l1x_handle_t` address from before to a `vl53l1x_device_handle_t`.
optionally change the address if not default `vl53l1x_device.i2c_address = 0x01;` the initialization will fail if address is not correct but there will be a logw in case some other address is found.

```c
vl53l1x_device_handle_t vl53l1x_device = VL53L1X_DEVICE_INIT;
vl53l1x_device.vl53l1x_handle = &vl53l1x_handle;
if (!vl53l1x_add_device(&vl53l1x_device))
{
    ESP_LOGE(TAG, "failed to add vl53l1x device 0x%02X", vl53l1x_device.i2c_address);
    return;
}
```

and finaly to get a measurement just call `vl53l1x_get_mm` with the `vl53l1x_device_handle_t` address from before.

```c
if (vl53l1x_handle.initialized)
{
    ESP_LOGI(TAG, "distance: %d mm", vl53l1x_get_mm(&vl53l1x_device));
}
```
