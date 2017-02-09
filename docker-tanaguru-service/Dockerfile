FROM varkal/tanaguru
MAINTAINER UCF Project <ucf-project@alterway.fr>

# Configure cli
WORKDIR /root/tanaguru-3.0.5.i386
ADD ./cnf/tanaguru.conf conf/context/tanaguru.conf

# Install node service
RUN apt update && \
    apt install -y curl && \
    curl -sL https://deb.nodesource.com/setup_6.x | bash - && \
    apt install -y nodejs

# Add UCF node.js server
RUN apt install -y git && \
    git clone https://github.com/UCF-project/ucf-audit-server.git && \
    cd ucf-audit-server && \
    npm i --production

# Default configuration
ENV DEBUG "UAT:Server*"
ENV PORT 3000

EXPOSE 3000

# Start all services
CMD service mysql start && \
    sleep 5 && \
    service xvfb start && \
    AUDITOR=tanaguru node ucf-audit-server/server.js
