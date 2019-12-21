
HAL: lights.c
把新文件上传到服务器, 所在目录:
hardware/libhardware/modules/lignts/lights.c
hardware/libhardware/modules/lignts/Android.mk

Android.mk内容如下:
# Copyright (C) 2012 The Android Open Source Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := lights.tiny4412

# HAL module implementation stored in
# hw/<VIBRATOR_HARDWARE_MODULE_ID>.default.so
LOCAL_MODULE_RELATIVE_PATH := hw
LOCAL_C_INCLUDES := hardware/libhardware
LOCAL_SRC_FILES := lights.c
LOCAL_SHARED_LIBRARIES := liblog
LOCAL_MODULE_TAGS := eng

include $(BUILD_SHARED_LIBRARY)


修改
vi vendor/friendly-arm/tiny4412/device-tiny4412.mk

ifeq ($(BOARD_USES_PWMLIGHTS),false)
#PRODUCT_COPY_FILES += \
#       $(VENDOR_PATH)/proprietary/lights.tiny4412.so:system/lib/hw/lights.tiny4412.so
endif

编译：
$ mmm hardware/libhardware/modules/lights
$ make snod
$ ./gen-img.sh

内核也需要修改:
1. drivers/leds/led-class.c :0644改为0666
static struct device_attribute led_class_attrs[] = {
	__ATTR(brightness, 0666, led_brightness_show, led_brightness_store),
	__ATTR(max_brightness, 0444, led_max_brightness_show, NULL),
#ifdef CONFIG_LEDS_TRIGGERS
	__ATTR(trigger, 0666, led_trigger_show, led_trigger_store),
#endif
	__ATTR_NULL,
};
2. drivers/leds/ledtrig-timer.c :0644改为0666
#if defined(CONFIG_MACH_IPCAM)
static DEVICE_ATTR(delay_on, 0666, led_delay_on_show, led_delay_on_store);
static DEVICE_ATTR(delay_off, 0666, led_delay_off_show, led_delay_off_store);
#else
static DEVICE_ATTR(delay_on, 0666, led_delay_on_show, led_delay_on_store);
static DEVICE_ATTR(delay_off, 0666, led_delay_off_show, led_delay_off_store);
#endif

make zImage


用这个命令确保我们自己编写的lignts.c已经编译进了system.img
diff vendor/friendly-arm/tiny4412/proprietary/lights.tiny4412.so out/target/product/tiny4412/system/lib/hw/lights.tiny4412.so

logcat lights:V *:S 


