# syntax = docker/dockerfile:1.3

FROM arm32v7/debian:bullseye AS builder

RUN apt-get update && apt-get install -y \
      build-essential \
      cmake \
      git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*


FROM builder as raspebrrypi-userland-cache

RUN cd /root \
    && git clone --depth 1 https://github.com/raspberrypi/userland.git \
    && cd userland \
    && sed -i -e 's/sudo //' buildme \
    # add picamera patch here
    && sed -i "/target_link_libraries(mmal mmal_core mmal_util mmal_vc_client vcos mmal_components)/i\SET(CMAKE_SHARED_LINKER_FLAGS \"-Wl,--no-as-needed\")" interface/mmal/CMakeLists.txt \ 
    && ./buildme \
    && rm -rf /root/userland


FROM arm32v7/debian:bullseye-slim AS prod

COPY --from=raspebrrypi-userland-cache /opt/vc /opt/vc

RUN echo "/opt/vc/lib" > /etc/ld.so.conf.d/00-vcms.conf \
    && ldconfig

ENV PATH /opt/vc/bin:/opt/vc/lib:$PATH

CMD ["bash"]
