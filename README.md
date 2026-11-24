
name: ESCO VPN APK Build

on:
  workflow_dispatch:
  push:
    branches: [ "main" ]

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Setup Android SDK
        uses: android-actions/setup-android@v3
        with:
          packages: 'platforms;android-37 build-tools;37.0.0 platform-tools'

      - name: Install NDK
        run: |
          sdkmanager "ndk;30.0.16248370"
          echo "NDK_HOME=$ANDROID_HOME/ndk/30.0.16248370" >> $GITHUB_ENV

      - name: Build native tunnel library
        run: |
          bash compile-hevtun.sh
          cp -r libs V2rayNG/app/

      - name: Download Xray library
        run: |
          cd AndroidLibXrayLite
          TAG=$(git describe --tags --abbrev=0)
          cd ..
          mkdir -p V2rayNG/app/libs
          curl -L \
            "https://github.com/2dust/AndroidLibXrayLite/releases/download/${TAG}/libv2ray.aar" \
            -o V2rayNG/app/libs/libv2ray.aar

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'

      - name: Build APK
        run: |
          cd V2rayNG
          chmod +x gradlew
          ./gradlew assemblePlaystoreDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: ESCO-VPN-APK
          path: V2rayNG/app/build/outputs/apk/playstore/debug/*.apk
