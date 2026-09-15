FROM docker.io/golang:1.27.1-trixie AS build_base

RUN apt-get update && apt-get install git -y

FROM build_base AS builder

WORKDIR /data/stolon/

RUN git init
RUN git remote add origin https://github.com/sorintlab/stolon
RUN git fetch --depth 1 origin bcea7a7548d5e2af95025cefbfd526f98c15b5e6 
RUN git checkout FETCH_HEAD

RUN go mod download
RUN make

FROM docker.io/postgres:18.6-trixie

RUN useradd -ms /bin/bash stolon

COPY --from=builder /data/stolon/bin/ /usr/local/bin

RUN chmod +x /usr/local/bin/stolon-keeper /usr/local/bin/stolon-sentinel /usr/local/bin/stolon-proxy /usr/local/bin/stolonctl
