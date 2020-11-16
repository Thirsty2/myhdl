ARG IMAGE="alpine:3.12"
# Note that some commands such as 'apk add' won't work in arbitrary images.  For instance,
# 'apk add' works fine under alpine, but not under ubuntu (at least not by default.)
# In other words, the following 'apk add' sections assume an alpine based image or a derivitive
#
FROM $IMAGE AS base
#
# Place anything that is common to both the build and execution environment in base 
#
RUN apk add --no-cache \
        python3 \
        py3-pip \
        zlib \
        bzip2 \
        readline \
        libgcc \
        libstdc++ \
        libhistory \
        make
	
FROM base as builder

RUN apk add --no-cache \
	autoconf \
	build-base \
	bison \
	bzip2-dev \
	flex \
	git \
	gperf \
	readline-dev \
	zlib-dev

WORKDIR /work

#download sources
RUN git clone --branch v11-branch https://github.com/steveicarus/iverilog.git
RUN wget https://github.com/myhdl/myhdl/archive/0.11.tar.gz && tar xzvf 0.11.tar.gz && rm 0.11.tar.gz

#build icarus
RUN cd iverilog && sh autoconf.sh && ./configure && make -j4 && make install

#build myhdl cosim
RUN cd myhdl-0.11/cosimulation/icarus && make


FROM base as release-candidate

COPY --from=builder /usr/local /usr/local/
COPY --from=builder /work/myhdl-0.11 /myhdl
COPY --from=builder /work/myhdl-0.11/cosimulation/icarus/myhdl.vpi /usr/local/lib/ivl/myhdl.vpi

RUN  pip3 install pytest-xdist -e /myhdl
#
# There is intentionally no entrypoint and no user in this image.  This makes it easier to
# use this as the parent image to customize a new image with an entrypoint or user for a particular use case. 