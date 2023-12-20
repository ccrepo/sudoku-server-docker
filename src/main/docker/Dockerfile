# Dockerfile

# Use the official Ubuntu 22.04 base image
FROM ubuntu:22.04

# Environment
ARG SUDOKU_DOCKER_GITUSER=default_value

# Info
RUN bash -c "echo starting using github user ${SUDOKU_DOCKER_GITUSER} "

# Install apt-utils
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y apt-utils && \
    rm -rf /var/lib/apt/lists/*

# Install necessary build tools for C++
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y openjdk-19-jdk && \
    rm -rf /var/lib/apt/lists/*

# Install tomcat
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y tomcat9 && \
    rm -rf /var/lib/apt/lists/*

# Install GCC 11 and other named tools
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y gcc-11 g++-11 git gradle curl wget && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y build-essential && \
    rm -rf /var/lib/apt/lists/*

# Set the working directory
WORKDIR /usr/local/src

# Clone sudoku-lib
RUN git clone https://github.com/${SUDOKU_DOCKER_GITUSER}/sudoku-lib.git

# Set the working directory
WORKDIR /usr/local/src/sudoku-lib/bin

# Build
RUN ./c
RUN ./j

# Check artefacts
RUN if [ ! -e /usr/local/src/sudoku-lib/sudoku/build/lib/main/debug/linux/libsudoku.so ]; then echo "missing shared lib "; exit 1; fi

# Copy shared lib into place
RUN cp -f /usr/local/src/sudoku-lib/sudoku/build/lib/main/debug/linux/libsudoku.so /usr/lib/libsudoku.so

# Check shared lib copy
RUN if [ ! -f /usr/lib/libsudoku.so ]; then echo "missing shared lib copy "; exit 1; fi

# Set the working directory
WORKDIR /usr/local/src

# Clone sudoku-lib
RUN git clone https://github.com/${SUDOKU_DOCKER_GITUSER}/sudoku-server.git

# Set the working directory
WORKDIR /usr/local/src/sudoku-server/bin

# Build
RUN ./c

# Check war
RUN if [ ! -e /usr/local/src/sudoku-server/sudoku-server/build/libs/sudoku.war ]; then echo "missing war file "; exit 1; fi

# Copy war into place
RUN cp /usr/local/src/sudoku-server/sudoku-server/build/libs/sudoku.war /var/lib/tomcat9/webapps/sudoku.war

# Check war copy
RUN if [ ! -f /var/lib/tomcat9/webapps/sudoku.war ]; then echo "missing war copy "; exit 1; fi

# Config
RUN cp -f /var/lib/tomcat9/conf/server.xml /var/lib/tomcat9/conf/server.xml.bak

# Check config
RUN if [ ! -f /var/lib/tomcat9/conf/server.xml ]; then echo "missing server.xml "; exit 1; fi

RUN if [ ! -f /var/lib/tomcat9/conf/server.xml.bak ]; then echo "missing server.xml.bak "; exit 1; fi

# Customise config
RUN awk 'BEGIN { found = 0; RS = "\0"; ORS = ""; }                         \
     /<\/Host>[[:space:]\n]*<\/Engine>/ && !found {                        \
         sub(/<\/Host>[[:space:]\n]*<\/Engine>/, "<Context docBase=\"sudoku\" path=\"/sudoku\" reloadable=\"true\" /></Host></Engine>");  \
         found = 1;                                                        \	 
     }                                                                     \
     { print; }' /var/lib/tomcat9/conf/server.xml.bak > /var/lib/tomcat9/conf/server.xml

# Empty default app 
RUN echo "" >  /var/lib/tomcat9/webapps/ROOT/index.html

# Expose the default Tomcat port
EXPOSE 8080

# Set CATALINA_BASE and update PATH to include catalina.sh
ENV CATALINA_BASE /var/lib/tomcat9
ENV CATALINA_HOME /usr/share/tomcat9
ENV PATH $CATALINA_HOME/bin:$PATH

# Start Tomcat when the container starts
CMD ["catalina.sh", "run"]