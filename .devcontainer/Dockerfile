FROM ghcr.io/linuxcontainers/debian-slim:latest AS downloader

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils \
    && apt-get install -y \
    curl \
    unzip \
	wget

ARG FANTOM_URL=https://github.com/fantom-lang/fantom/releases/download/v1.0.78/fantom-1.0.78.zip
RUN mkdir -p /tmp/dc-downloads /tmp/dc-extracted/fantom \
    && curl -o /tmp/dc-downloads/fantom.zip $FANTOM_URL -L \
    && unzip -d /tmp/dc-extracted/ /tmp/dc-downloads/fantom.zip

ARG TARGET_JAVA_VERSION=11

# Download Microsoft OpenJDK
RUN arch="$(dpkg --print-architecture)" \
	&& case "$arch" in \
		"amd64") \
			jdkUrl="https://aka.ms/download-jdk/microsoft-jdk-${TARGET_JAVA_VERSION}-linux-x64.tar.gz"; \
			;; \
		"arm64") \
			jdkUrl="https://aka.ms/download-jdk/microsoft-jdk-${TARGET_JAVA_VERSION}-linux-aarch64.tar.gz"; \
			;; \
		*) echo >&2 "error: unsupported architecture: '$arch'"; exit 1 ;; \
	esac \
	\
	&& wget --progress=dot:giga -O msopenjdk.tar.gz "${jdkUrl}" \
	&& wget --progress=dot:giga -O sha256sum.txt "${jdkUrl}.sha256sum.txt" \
	\
	&& sha256sumText=$(cat sha256sum.txt) \
	&& sha256=$(expr substr "${sha256sumText}" 1 64) \
	&& echo "${sha256} msopenjdk.tar.gz" | sha256sum --strict --check - \
	&& rm sha256sum.txt* \
	\
	&& mkdir -p /tmp/dc-extracted/java \
	&& tar --extract \
		--file msopenjdk.tar.gz \
		--directory /tmp/dc-extracted/java \
		--strip-components 1 \
		--no-same-owner \
	&& rm msopenjdk.tar.gz* 

FROM ghcr.io/linuxcontainers/debian-slim:latest as devcontainer

# set fantom location
ENV FANTOM_PATH=/opt/fantom
ENV PATH=$FANTOM_PATH/bin:${PATH}

ARG TARGET_JAVA_VERSION=11
ENV JAVA_HOME /usr/lib/jvm/msopenjdk-${TARGET_JAVA_VERSION}
ENV PATH ${JAVA_HOME}/bin:${PATH}

# Copy from our other container
COPY --from=downloader /tmp/dc-extracted/fantom-1.0.78 $FANTOM_PATH
COPY --from=downloader /tmp/dc-extracted/java $JAVA_HOME

RUN ln -s ${JAVA_HOME} /usr/local/openjdk-${TARGET_JAVA_VERSION}

RUN bash $FANTOM_PATH/adm/unixsetup