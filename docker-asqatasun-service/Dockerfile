FROM docker-asqatasun-cli
MAINTAINER UCF Project <ucf-project@alterway.fr>

# Install node service
RUN curl -sL https://deb.nodesource.com/setup_6.x | bash - && \
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
    AUDITOR=asqatasun node ucf-audit-server/server.js
