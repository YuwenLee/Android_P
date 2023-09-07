Table of Contents
=================
  * [Integrate Prebuilt APK](#Integrate_Prebuilt_APK)
  * [build_image.py](#build_image.py)

Integrate Prebuilt APK
======================
Trace <b>build/make/core/prebuilt_internal.mk</b> to figure out how the build process handles the makefile (<b>Android.mk</b>) of a pre-built APK:
<pre>
LOCAL_PATH := $(call my-dir)

# XXX APK
include $(CLEAR_VARS)

# Module name should match apk name to be installed.
LOCAL_MODULE := XXX
ifeq ($(TARGET_PRODUCT),SKUA)
LOCAL_SRC_FILES := $(LOCAL_MODULE)_a.apk
else ifeq ($(TARGET_PRODUCT),SKUB)
LOCAL_SRC_FILES := $(LOCAL_MODULE)_b.apk
else ifeq ($(TARGET_PRODUCT),SKUC)
LOCAL_SRC_FILES := $(LOCAL_MODULE)_c.apk
else
LOCAL_SRC_FILES := $(LOCAL_MODULE).apk
endif
LOCAL_MODULE_PATH := $(TARGET_OUT_PRODUCT_APPS_PRIVILEGED)
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_CERTIFICATE := platform
LOCAL_PRIVILEGED_MODULE := true
LOCAL_DEX_PREOPT := true

include $(BUILD_PREBUILT)
</pre>

build_image.py
==============
<b>build_image.py</b> would be invoked when <br>
&nbsp; &nbsp; [1]. make systemimage and <br>
&nbsp; &nbsp; [2]. make target-files-package (adding the system image to target files first)<br>
The whole picture looks like: <br>
&nbsp; &nbsp; <img src="https://github.com/YuwenLee/Android_P/blob/master/pic/makefile_add-img-to-target-files.png" width=720/> <br>
It takes at least 3 arguments (input_dir, prop-dictionary, and output) and then invokes mkuserimg_mke2fs.sh to generate the output image.
<pre>
  build_image.py \
      out/target/product/abc/system \
      system_image_info.txt \
      obj/PACKING/targetfiles/system.img \
      out/target/product/abc/system
</pre>
[1]. input_dir
--------------
The source of the file system. Take the image of system as example, the input directory looks like
Note:
The symbolic links product and vendor are created in the procedure of making $(BUILT_SYSTEMIMAGE)

[2]. prop-dictionary
--------------------
A lookup table used to generate the command to make the image. In the makefile (LINUX/android/build/core/Makefile), the table is generated by generate-userimage-prop-dictionary:
<pre>
$(if $(BOARD_SYSTEMIMAGE_PARTITION_SIZE), \
   $(hide) echo "system_size=$(BOARD_SYSTEMIMAGE_PARTITION_SIZE)" \
   >> $(1))
</pre>
A complete dictionary looks like

[3]. out_file
-------------
Path of the output image file. This parameter is used as the 2nd argument passed to the build_command (ie mkuserimg_mke2fs.sh) :

[4]. target_out (optional)
--------------------------
According the comment in the script, this parameter is "the path of the product out directory to read device specific FS config files".
In the script, this parameter is used as -D option of the build_command. The arguments to the build command (mkuserimg.sh) are:
When building the system.img, build_image.py uses the yellow parts of the above prop-dictionary to generate the command:
