FROM debian:9
RUN apt-get update \
  && apt-get install -y openjdk-8-jdk cmake curl wget git unzip python

ARG BUILD_USER=builder
ARG BUILD_UID=2001
ARG ANDROID_COMPILE_SDK=29
ARG ANDROID_BUILD_TOOLS=
ARG ANDROID_SDK_TOOLS=4333796 
ARG ANDROID_NDK_TOOLS=21.1.6352462
RUN useradd --home-dir /workspace \
            --shell /bin/bash \
            --create-home \
            --uid $BUILD_UID\
            $BUILD_USER
USER $BUILD_USER
WORKDIR /workspace

RUN wget --quiet --output-document=/tmp/sdk-tools-linux.zip https://dl.google.com/android/repository/sdk-tools-linux-$ANDROID_SDK_TOOLS.zip \
    && unzip /tmp/sdk-tools-linux.zip -d $HOME/.android \
    && mkdir -p $HOME/.android \ 
    && touch $HOME/.android/repositories.cfg \
    && mkdir $HOME/.android/licenses \
    && echo y | $HOME/.android/tools/bin/sdkmanager "ndk;$ANDROID_NDK_TOOLS" >/dev/null \
    && echo y | $HOME/.android/tools/bin/sdkmanager "platform-tools" "platforms;android-$ANDROID_COMPILE_SDK" >/dev/null \
    && echo y | $HOME/.android/tools/bin/sdkmanager "system-images;android-$ANDROID_COMPILE_SDK;default;x86" >/dev/null 

RUN curl https://sh.rustup.rs -sSf | sh -s -- -y \
    && . $HOME/.cargo/env \
    && rustup target add armv7-linux-androideabi \
    && rustup target add aarch64-linux-android \
    && rustup target add i686-linux-android

ENV ANDROID_HOME=/workspace/.android
ENV ANDROID_SDK_ROOT=/workspace/.android
ENV PATH=~/.cargo/bin/:~/.android/tools/bin/:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ENV CARGO_HOME=/workspace/.cargo

## This would be the CI steps
RUN git clone https://github.com/zcash/zcash-android-wallet-sdk.git \
  && cd zcash-android-wallet-sdk \
  && ./gradlew ciBuild
