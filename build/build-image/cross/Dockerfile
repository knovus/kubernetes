FROM us.gcr.io/k8s-artifacts-prod/build-image/kube-cross:v1.13.9-5

RUN  apt-get update && apt-get install -y --no-install-recommends \
#            libc6-dev:i386 \
            crossbuild-essential-arm64 \
            crossbuild-essential-i386