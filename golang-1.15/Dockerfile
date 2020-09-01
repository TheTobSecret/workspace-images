FROM ubuntu:latest as base

ARG DEBIAN_FRONTEND=noninteractive

RUN mkdir /var/lib/apt/dazzle-marks

RUN apt-get update > /dev/null

RUN apt-get upgrade -y > /dev/null

RUN apt-get install -y --no-install-recommends apt-utils > /dev/null

RUN apt-get install -y locales > /dev/null \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8

ENV LANG en_US.utf8

RUN apt-get install -y \
        unzip \
        build-essential \
        htop \
        jq \
        man-db \
        software-properties-common \
        sudo \
        time \
        vim \
        multitail \
        lsof \
        coreutils \
        > /dev/null

RUN rm -rf /var/lib/apt/lists/*

### Git ###
RUN add-apt-repository -y ppa:git-core/ppa \
    && apt-get install -yq git
### Gitpod user ###
# '-l': see https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user
RUN useradd -l -u 33333 -G sudo -md /home/gitpod -s /bin/bash -p gitpod gitpod \
    # passwordless sudo for users in the 'sudo' group
    && sed -i.bkp -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers
ENV HOME=/home/gitpod
WORKDIR $HOME
# custom Bash prompt
RUN { echo "PS1='\[\e]0;\u \w\a\]\[\033[01;32m\]\u\[\033[00m\] \[\033[01;34m\]\w\[\033[00m\] \\\$ '" ; } >> .bashrc

### Gitpod user (2) ###
USER gitpod
# use sudo so that user does not get sudo usage info on (the first) login
RUN sudo echo "Running 'sudo' for Gitpod: success" && \
    # create .bashrc.d folder and source it in the bashrc
    mkdir /home/gitpod/.bashrc.d && \
    (echo; echo "for i in \$(ls \$HOME/.bashrc.d/*); do source \$i; done"; echo) >> /home/gitpod/.bashrc

### Apache, PHP and Nginx ###
LABEL dazzle/layer=tool-nginx
USER root
RUN apt-get update && apt-get install -y -qq \
        nginx \
        nginx-extras \
    && cp /var/lib/dpkg/status /var/lib/apt/dazzle-marks/tool-nginx.status \
    && rm -rf /var/lib/apt/lists/* /tmp/* \
    && mkdir /var/run/nginx \
    && chown -R gitpod:gitpod /etc/nginx /var/run/nginx /var/lib/nginx/ /var/log/nginx/

### Go ###
USER root
RUN apt-get update && apt-get install curl -y -qq

USER gitpod
ENV GO_VERSION=1.15
RUN curl -fsSL https://storage.googleapis.com/golang/go$GO_VERSION.linux-amd64.tar.gz | tar xzs 

USER root



RUN sudo mv go /usr/local/

USER gitpod

ENV GOROOT=/usr/local/go
ENV GOPATH=$HOME/go
ENV PATH=$GOROOT/bin:$GOPATH/bin:$PATH
# install VS Code Go tools from https://github.com/Microsoft/vscode-go/blob/0faec7e5a8a69d71093f08e035d33beb3ded8626/src/goInstallTools.ts#L19-L45
RUN go get -u -v \
        github.com/mdempsky/gocode \
        github.com/uudashr/gopkgs/cmd/gopkgs \
        github.com/ramya-rao-a/go-outline \
        github.com/acroca/go-symbols \
        golang.org/x/tools/cmd/guru \
        golang.org/x/tools/cmd/gorename \
        github.com/fatih/gomodifytags \
        github.com/haya14busa/goplay/cmd/goplay \
        github.com/josharian/impl \
        github.com/tylerb/gotype-live \
        github.com/rogpeppe/godef \
        github.com/zmb3/gogetdoc \
        golang.org/x/tools/cmd/goimports \
        github.com/sqs/goreturns \
        winterdrache.de/goformat/goformat \
        golang.org/x/lint/golint \
        github.com/cweill/gotests/... \
        github.com/alecthomas/gometalinter \
        honnef.co/go/tools/... \
        github.com/mgechev/revive \
        github.com/sourcegraph/go-langserver \
        github.com/go-delve/delve/cmd/dlv \
        github.com/davidrjenni/reftools/cmd/fillstruct \
        github.com/godoctor/godoctor && \
    GO111MODULE=on go get -u -v \
        golang.org/x/tools/gopls@v0.4.4 && \
    go get -u -v -d github.com/stamblerre/gocode && \
    go build -o $GOPATH/bin/gocode-gomod github.com/stamblerre/gocode && \
    rm -rf $GOPATH/src && \
    sudo rm -rf $GOPATH/pkg && \
    rm -rf /home/gitpod/.cache/go
