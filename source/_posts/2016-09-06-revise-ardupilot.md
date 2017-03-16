---
title: ArduPilot修改记录
date: 2016-09-06 21:21:40
categories: 弄点工具
tags: 
  - ArduPilot
---



下载原版ArduPilot，并进行初始化：

```sh
$ git clone https://github.com/ArduPilot/ardupilot.git
$ cd ardupilot
$ git submodule init
$ git submodule update
```



## 原版ArduPilot修改为DM1

在`libraries/AP_HAL_Linux/`目录下增加文件`RCInput_DM1.h`，`RCInput_DM1.cpp`，`RCOutput_DM1.h`，`RCOutput_DM1.cpp`，`pak_stm32.h`



在`libraries/AP_HAL_Linux/HAL_Linux_Class.cpp`文件中修改

```cpp
#elif CONFIG_HAL_BOARD_SUBTYPE == HAL_BOARD_SUBTYPE_LINUX_DM1
static RCInput_DM1 rcinDriver;

#elif CONFIG_HAL_BOARD_SUBTYPE == HAL_BOARD_SUBTYPE_LINUX_DM1
static RCOutput_DM1 rcoutDriver;
```



在`libraries/AP_HAL_Linux/Scheduler.cpp`文件中修改

```cpp
#if CONFIG_HAL_BOARD_SUBTYPE == HAL_BOARD_SUBTYPE_LINUX_NAVIO ||    \
    CONFIG_HAL_BOARD_SUBTYPE == HAL_BOARD_SUBTYPE_LINUX_ERLEBRAIN2 || \
    CONFIG_HAL_BOARD_SUBTYPE == HAL_BOARD_SUBTYPE_LINUX_BH || \
    CONFIG_HAL_BOARD_SUBTYPE == HAL_BOARD_SUBTYPE_LINUX_PXFMINI || \
    CONFIG_HAL_BOARD_SUBTYPE == HAL_BOARD_SUBTYPE_LINUX_DM1
#define APM_LINUX_RCIN_RATE             2000
#define APM_LINUX_TONEALARM_RATE        100
#define APM_LINUX_IO_RATE               50
```



在`libraries/AP_HAL_Linux/SPIDevice.cpp`文件中增加

```cpp
#elif CONFIG_HAL_BOARD_SUBTYPE == HAL_BOARD_SUBTYPE_LINUX_DM1
SPIDesc SPIDeviceManager::_device[] = {
    SPIDesc("mpu9250", 0, 0, SPI_MODE_0, 8, SPI_CS_KERNEL, 1*MHZ, 10*MHZ),
    SPIDesc("stm32", 0, 1, SPI_MODE_0, 8, SPI_CS_KERNEL,  1*MHZ, 10*MHZ),
};
```



在`libraries/AP_HAL/AP_HAL_Boards.h`文件中增加DM1定义

```cpp
#define HAL_BOARD_SUBTYPE_LINUX_DM1        1016
```

```cpp
#elif CONFIG_HAL_BOARD_SUBTYPE == HAL_BOARD_SUBTYPE_LINUX_DM1
#define HAL_BOARD_LOG_DIRECTORY "/var/APM/logs"
#define HAL_BOARD_TERRAIN_DIRECTORY "/var/APM/terrain"
#define HAL_INS_DEFAULT HAL_INS_MPU9250_SPI
#define HAL_INS_MPU9250_NAME "mpu9250"
#define HAL_BARO_DEFAULT HAL_BARO_MS5611_I2C
#define HAL_BARO_MS5611_I2C_BUS 1
#define HAL_BARO_MS5611_I2C_ADDR 0x77
#define HAL_BARO_MS5611_USE_TIMER false
#define HAL_COMPASS_DEFAULT HAL_COMPASS_AK8963_MPU9250
```



在`mk/environ.mk`文件中增加

```cpp
ifneq ($(findstring dm1, $(MAKECMDGOALS)),)
HAL_BOARD = HAL_BOARD_LINUX
HAL_BOARD_SUBTYPE = HAL_BOARD_SUBTYPE_LINUX_DM1
endif
```



在`mk/help.mk`文件中增加

```cpp
	@echo "  dm1 - the BPi + NavIO2 cape combination"
```



在`mk/targets.mk`文件中增加

```cpp
dm1: HAL_BOARD = HAL_BOARD_LINUX
dm1: TOOLCHAIN = RPI
dm1: BUILDSYS_DEPRECATED = 1
dm1: all
```



在`Tools/ardupilotwaf/boards.py`文件中添加对dm1的支持，关于waf的使用方法可以参考[waf编译工具使用方法](http://blog.guantau.com/2016/09/06/waf-tool/)

```python
class dm1(linux):
    toolchain = 'arm-linux-gnueabihf'

    def configure_env(self, cfg, env):
        super(dm1, self).configure_env(cfg, env)

        env.DEFINES.update(
            CONFIG_HAL_BOARD_SUBTYPE = 'HAL_BOARD_SUBTYPE_LINUX_DM1',
        )
```









下面是在早期ArduPilot上面所做的修改

## libraries/AP_HAL_Linux/HAL_Linux_Class.cpp

目前RCInput_DM1还有问题，将其注释掉
```cpp
//guantau
// #elif CONFIG_HAL_BOARD_SUBTYPE == HAL_BOARD_SUBTYPE_LINUX_DM1
// static RCInput_DM1 rcinDriver;
```

## libraries/AP_InertialSensor/AP_InertialSensor_MPU9250.h
添加函数`accumulate()`，该函数在AP_InertialSensor_Backend.h中申明为纯虚函数
```cpp
//guantau
void accumulate() override;
```


## libraries/AP_InertialSensor/AP_InertialSensor_MPU9250.cpp

实现函数`accumulate()`，用于非定时读取情况下手动读取数据
```cpp
//guantau
void AP_InertialSensor_MPU9250::accumulate()
{
    _poll_data();
}
```

在函数`_hardware_init()`中，修改初始化代码

```CPP
bool AP_InertialSensor_MPU9250::_hardware_init(void)
{
    if (!_dev->get_semaphore()->take(100)) {
        AP_HAL::panic("MPU6000: Unable to get semaphore");
    }

    // initially run the bus at low speed
    _dev->set_speed(AP_HAL::Device::SPEED_LOW);

    uint8_t value = _register_read(MPUREG_WHOAMI);
    if (value != MPUREG_WHOAMI_MPU9250 && value != MPUREG_WHOAMI_MPU9255) {
        hal.console->printf("MPU9250: unexpected WHOAMI 0x%x\n", (unsigned)value);
        _dev->get_semaphore()->give();
        _dev->set_speed(AP_HAL::Device::SPEED_HIGH);
        return false;
    }

    // Clock Source
    _register_write(MPUREG_PWR_MGMT_1, BIT_PWR_MGMT_1_CLK_XGYRO);
    hal.scheduler->delay(100);

    // Enable Acc & Gyro
    _register_write(MPUREG_PWR_MGMT_2, 0x00);
    hal.scheduler->delay(100);

    // Use DLPF set Gyroscope bandwidth 184Hz, temperature bandwidth 188Hz
    _register_write(MPUREG_CONFIG, BITS_DLPF_CFG_188HZ);
    hal.scheduler->delay(100);

    // Gyro scale 2000º/s
    _register_write(MPUREG_GYRO_CONFIG, BITS_GYRO_FS_2000DPS);
    hal.scheduler->delay(100);

    // RM-MPU-9250A-00.pdf, pg. 15, select accel full scale 16g
    _register_write(MPUREG_ACCEL_CONFIG, 0x18);
    hal.scheduler->delay(100);

    // Set Acc Data Rates, Enable Acc LPF , Bandwidth 184Hz
    _register_write(MPUREG_ACCEL_CONFIG_2, 0x09);
    hal.scheduler->delay(100);

    // clear interrupt on any read, and hold the data ready pin high
    // until we clear the interrupt
    _register_write(MPUREG_INT_PIN_CFG, 0x30);
    hal.scheduler->delay(100);

    // Chip reset
    // uint8_t tries;
    // for (tries = 0; tries < 5; tries++) {
        //uint8_t user_ctrl = _register_read(MPUREG_USER_CTRL);

        /* First disable the master I2C to avoid hanging the slaves on the
         * aulixiliar I2C bus - it will be enabled again if the AuxiliaryBus
         * is used */
        //if (user_ctrl & BIT_USER_CTRL_I2C_MST_EN) {
        //    _register_write(MPUREG_USER_CTRL, user_ctrl & ~BIT_USER_CTRL_I2C_MST_EN);
        //    hal.scheduler->delay(10);
        //}

        // reset device
        //_register_write(MPUREG_PWR_MGMT_1, BIT_PWR_MGMT_1_DEVICE_RESET);
        //hal.scheduler->delay(100);

        /* bus-dependent initialization */
        //if (_dev->bus_type == AP_HAL::Device::BUS_TYPE_SPI) {
            /* Disable I2C bus if SPI selected (Recommended in Datasheet to be
             * done just after the device is reset) */
            //_register_write(MPUREG_USER_CTRL, BIT_USER_CTRL_I2C_IF_DIS);
        //}

        // Wake up device and select GyroZ clock. Note that the
        // MPU9250 starts up in sleep mode, and it can take some time
        // for it to come out of sleep

        // _register_write(MPUREG_PWR_MGMT_1, BIT_PWR_MGMT_1_CLK_ZGYRO);
        // hal.scheduler->delay(5);

        // // check it has woken up
        // if (_register_read(MPUREG_PWR_MGMT_1) == BIT_PWR_MGMT_1_CLK_ZGYRO) {
            // break;
        // }

        // hal.scheduler->delay(10);
        // if (_data_ready()) {
            // break;
        // }

// #if MPU9250_DEBUG
        // _dump_registers();
// #endif
    // }
    // if (tries == 5) {
        // hal.console->println("Failed to boot MPU9250 5 times");
        // goto fail_tries;
    // }

    _dev->set_speed(AP_HAL::Device::SPEED_HIGH);
    _dev->get_semaphore()->give();

    return true;

// fail_tries:
// fail_whoami:
    // _dev->get_semaphore()->give();
    // _dev->set_speed(AP_HAL::Device::SPEED_HIGH);
    // return false;
}
```



在函数`start()`中，去掉初始化代码，去掉定时读取的代码

```cpp
void AP_InertialSensor_MPU9250::start()
{
    hal.scheduler->suspend_timer_procs();

    if (!_dev->get_semaphore()->take(100)) {
        AP_HAL::panic("MPU92500: Unable to get semaphore");
    }

    // initially run the bus at low speed
    _dev->set_speed(AP_HAL::Device::SPEED_LOW);

    // // only used for wake-up in accelerometer only low power mode
    // _register_write(MPUREG_PWR_MGMT_2, 0x00);
    // hal.scheduler->delay(1);

    // // disable sensor filtering
    // _register_write(MPUREG_CONFIG, BITS_DLPF_CFG_256HZ_NOLPF2);

    // // set sample rate to 1kHz, and use the 2 pole filter to give the
    // // desired rate
    // _register_write(MPUREG_SMPLRT_DIV, DEFAULT_SMPLRT_DIV); // guantau
    // hal.scheduler->delay(1);

    // // Gyro scale 2000º/s
    // _register_write(MPUREG_GYRO_CONFIG, BITS_GYRO_FS_2000DPS);
    // hal.scheduler->delay(1);

    _product_id = AP_PRODUCT_ID_MPU9250;

    // // RM-MPU-9250A-00.pdf, pg. 15, select accel full scale 16g
    // _register_write(MPUREG_ACCEL_CONFIG,3<<3);

    // // configure interrupt to fire when new data arrives
    // _register_write(MPUREG_INT_ENABLE, BIT_RAW_RDY_EN);

    // // clear interrupt on any read, and hold the data ready pin high
    // // until we clear the interrupt
    // uint8_t value = _register_read(MPUREG_INT_PIN_CFG);
    // value |= BIT_INT_RD_CLEAR | BIT_LATCH_INT_EN;
    // _register_write(MPUREG_INT_PIN_CFG, value);

    // now that we have initialised, we set the bus speed to high
    _dev->set_speed(AP_HAL::Device::SPEED_HIGH);

    _dev->get_semaphore()->give();

    // grab the used instances
    _gyro_instance = _imu.register_gyro(DEFAULT_SAMPLE_RATE);
    _accel_instance = _imu.register_accel(DEFAULT_SAMPLE_RATE);

    hal.scheduler->resume_timer_procs();

    // start the timer process to read samples
    // hal.scheduler->register_timer_process(
        // FUNCTOR_BIND_MEMBER(&AP_InertialSensor_MPU9250::_poll_data, void));

    //guantau
    _poll_data();
}
```



在函数`_configure_slaves()`中修改初始化代码

```cpp
void AP_MPU9250_AuxiliaryBus::_configure_slaves()
{
    auto &backend = AP_InertialSensor_MPU9250::from(_ins_backend);

    // guantau
    // I2C Master mode
    backend._register_write(MPUREG_USER_CTRL, BIT_USER_CTRL_I2C_MST_EN);
    hal.scheduler->delay(100);

    // I2C configuration multi-master  IIC 400KHz
    backend._register_write(MPUREG_I2C_MST_CTRL, I2C_MST_CLOCK_400KHZ);
    hal.scheduler->delay(100);

    // Set the I2C slave addres of AK8963 and set for write.
    backend._register_write(MPUREG_I2C_SLV0_ADDR, 0x0C);
    hal.scheduler->delay(100);

    // I2C slave 0 register address from where to begin data transfer
    backend._register_write(MPUREG_I2C_SLV0_REG, 0x0B);
    hal.scheduler->delay(100);

    // Reset AK8963
    backend._register_write(MPUREG_I2C_SLV0_DO, 0x01);
    hal.scheduler->delay(100);

    // Enable I2C and set 1 byte
    backend._register_write(MPUREG_I2C_SLV0_CTRL, 0x81);
    hal.scheduler->delay(100);

    // I2C slave 0 register address from where to begin data transfer
    backend._register_write(MPUREG_I2C_SLV0_REG, 0x0A);
    hal.scheduler->delay(100);

    // Register value to continuous measurement in 16bit
    backend._register_write(MPUREG_I2C_SLV0_DO, 0x12);
    hal.scheduler->delay(100);

    // Enable I2C and set 1 byte
    backend._register_write(MPUREG_I2C_SLV0_CTRL, 0x81);
    hal.scheduler->delay(100);

//    /* Enable the I2C master to slaves on the auxiliary I2C bus*/
//    uint8_t user_ctrl = backend._register_read(MPUREG_USER_CTRL);
//    backend._register_write(MPUREG_USER_CTRL, user_ctrl | BIT_USER_CTRL_I2C_MST_EN);
//
//    /* stop condition between reads; clock at 400kHz */
//    backend._register_write(MPUREG_I2C_MST_CTRL,
//                            I2C_MST_CLOCK_400KHZ | I2C_MST_P_NSR);
//
//    /* Hard-code divider for internal sample rate, 1 kHz, resulting in a
//     * sample rate of 100Hz */
//    backend._register_write(MPUREG_I2C_SLV4_CTRL, 9);
//
//    /* All slaves are subject to the sample rate */
//    backend._register_write(MPUREG_I2C_MST_DELAY_CTRL,
//                            I2C_SLV0_DLY_EN | I2C_SLV1_DLY_EN |
//                            I2C_SLV2_DLY_EN | I2C_SLV3_DLY_EN);
}
```

## libraries/AP_Compass/AP_Compass_AK8963.h

添加函数`accumulate()`，该函数在AP_Compass_Backend.h中申明为纯虚函数

```cpp
//guantau
void accumulate() override;
```

## libraries/AP_Compass/AP_Compass_AK8963.cpp

实现函数`accumulate()`，用于非定时读取情况下手动读取数据

```cpp
//guantau
void AP_Compass_AK8963::accumulate()
{
    _update();
}
```

修改初始化代码`init()`，去除定时读取的任务

```cpp
bool AP_Compass_AK8963::init()
{
    ...
    ...
    /* timer needs to be called every 10ms so set the freq_div to 10 */
    // _timesliced = hal.scheduler->register_timer_process(FUNCTOR_BIND_MEMBER(&AP_Compass_AK8963::_update, void), 10);
    //guantau
    _update();
	...
	...
}
```





## libraries/AP_HAL_Linux/examples/BusTest/BusTest.cpp

在函数`loop()`中，若循环读取SPI总线的时延过小，则会出现死机的情况。

