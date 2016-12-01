FROM java:8-jdk

MAINTAINER Adriano Bonat <adrianob@gmail.com>

ENV GOCD_VERSION=16.5.0-3305
ENV GOCD_AGENT_SHA1=90e1f9c7281c6cd7ddc384b831faeaf347331ab9
ENV GOCD_AGENT_URL=https://download.go.cd/binaries/${GOCD_VERSION}/deb/go-agent-${GOCD_VERSION}.deb

RUN apt-get update; apt-get install -y git maven Xvfb

#==============
#Google chrome
#==============
ARG CHROME_VERSION="google-chrome-stable"
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
  && echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list \
  && apt-get update -qqy \
  && apt-get -qqy install \
    ${CHROME_VERSION:-google-chrome-stable} \
  && rm /etc/apt/sources.list.d/google-chrome.list \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/*
#==================
# Chrome webdriver
#==================
ARG CHROME_DRIVER_VERSION=2.25
RUN wget --no-verbose -O /tmp/chromedriver_linux64.zip https://chromedriver.storage.googleapis.com/$CHROME_DRIVER_VERSION/chromedriver_linux64.zip \
  && rm -rf /opt/selenium/chromedriver \
  && unzip /tmp/chromedriver_linux64.zip -d /opt/selenium \
  && rm /tmp/chromedriver_linux64.zip \
  && mv /opt/selenium/chromedriver /opt/selenium/chromedriver-$CHROME_DRIVER_VERSION \
  && chmod 755 /opt/selenium/chromedriver-$CHROME_DRIVER_VERSION \
  && ln -fs /opt/selenium/chromedriver-$CHROME_DRIVER_VERSION /usr/local/bin/chromedriver


RUN set -x \
 # Download/Install/Cleanup Go Agent
 && curl -fSL "$GOCD_AGENT_URL" -o go-agent.deb \
 && echo "$GOCD_AGENT_SHA1 go-agent.deb" | sha1sum -c - \
 && dpkg -i --debug=10 go-agent.deb \
 && rm go-agent.deb

RUN set -x \
 # Uses default environment values from start script
 && rm /etc/default/go-agent

 RUN apt-key adv --keyserver hkp://pool.sks-keyservers.net --recv-keys 023EDB0B
 RUN echo deb http://dl.bintray.com/gauge/gauge-deb stable main | tee -a /etc/apt/sources.list
 RUN apt-get update
 RUN apt-get install -y gauge
 RUN gauge_setup
 RUN gauge --install java
 RUN gauge -v
 RUN Xvfb -ac :99 -screen 0 1280x1024x16 & \
  export DISPLAY=:99
COPY start-agent.sh /

#ENV JAVA_HOME="/usr/lib/jvm/open-jdk"  \

    # Agent will look for server at this hostname
ENV GO_SERVER=go-server \

    # Need to set this otherwise work dir will be /usr/share/go-agent
    # and auto register won't work
    AGENT_WORK_DIR=/var/lib/go-agent

CMD ["/start-agent.sh"]
