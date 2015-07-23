---
layout: post
title: "create universal framework"
date: 2015-07-22 19:45:58 +0800
comments: true
categories: iOS
---

xcode很有意思，当选择平台是iOS device时，framework支持的architecture是arm系列的，然而如果是iOS simulator的话，又仅产生支持x86_64系列的architecture。

那么当我们如何编译framework时使它可以支持所有architecture呢

[参考资料1](https://medium.com/@syshen/create-an-ios-universal-framework-148eb130a46c)

[参考资料2](https://kodmunki.wordpress.com/2015/03/04/cocoa-touch-frameworks-for-ios8-remix/)

<!--more-->

要点如下：

在build phases里增加一个脚本

    ######################
    # Options
    ######################

    REVEAL_ARCHIVE_IN_FINDER=false

    FRAMEWORK_NAME="${PROJECT_NAME}"

	SIMULATOR_LIBRARY_PATH="${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/${FRAMEWORK_NAME}.framework"
	
	DEVICE_LIBRARY_PATH="${BUILD_DIR}/${CONFIGURATION}-iphoneos/${FRAMEWORK_NAME}.framework"
	
	UNIVERSAL_LIBRARY_DIR="${BUILD_DIR}/${CONFIGURATION}-iphoneuniversal"

	FRAMEWORK="${UNIVERSAL_LIBRARY_DIR}/${FRAMEWORK_NAME}.framework"


    ######################
    # Build Frameworks
    ######################

	xcodebuild -workspace ${PROJECT_NAME}.xcworkspace -scheme ${PROJECT_NAME} -sdk iphonesimulator -configuration ${CONFIGURATION} clean build CONFIGURATION_BUILD_DIR=${BUILD_DIR}/${CONFIGURATION}-iphonesimulator 2>&1

	xcodebuild -workspace ${PROJECT_NAME}.xcworkspace -scheme ${PROJECT_NAME} -sdk iphoneos -configuration ${CONFIGURATION} clean build CONFIGURATION_BUILD_DIR=${BUILD_DIR}/${CONFIGURATION}-iphoneos 2>&1

    ######################
    # Create directory for universal
    ######################

	rm -rf "${UNIVERSAL_LIBRARY_DIR}"

	mkdir "${UNIVERSAL_LIBRARY_DIR}"

	mkdir "${FRAMEWORK}"


    ######################
    # Copy files Framework
    ######################

    cp -r "${DEVICE_LIBRARY_PATH}/." "${FRAMEWORK}"


    ######################
    # Make an universal binary
    ######################

	lipo "${SIMULATOR_LIBRARY_PATH}/${FRAMEWORK_NAME}" "${DEVICE_LIBRARY_PATH}/${FRAMEWORK_NAME}" -create -output "${FRAMEWORK}/${FRAMEWORK_NAME}" | echo

    # For Swift framework, Swiftmodule needs to be copied in the universal framework
    if [ -d "${SIMULATOR_LIBRARY_PATH}/Modules/${FRAMEWORK_NAME}.swiftmodule/" ]; then
		cp -f ${SIMULATOR_LIBRARY_PATH}/Modules/${FRAMEWORK_NAME}.swiftmodule/* "${FRAMEWORK}/Modules/${FRAMEWORK_NAME}.swiftmodule/" | echo
	fi
                                                                      
	if [ -d "${DEVICE_LIBRARY_PATH}/Modules/${FRAMEWORK_NAME}.swiftmodule/" ]; then
		cp -f ${DEVICE_LIBRARY_PATH}/Modules/${FRAMEWORK_NAME}.swiftmodule/* "${FRAMEWORK}/Modules/${FRAMEWORK_NAME}.swiftmodule/" | echo
	fi

    ######################
    # On Release, copy the result to release directory
    ######################

	OUTPUT_DIR="${PROJECT_DIR}/Output/${FRAMEWORK_NAME}-${CONFIGURATION}-iphoneuniversal/"

	rm -rf "$OUTPUT_DIR"
	mkdir -p "$OUTPUT_DIR"

	cp -r "${FRAMEWORK}" "$OUTPUT_DIR"

	if [ ${REVEAL_ARCHIVE_IN_FINDER} = true ]; then
		open "${OUTPUT_DIR}/"
	fi
