FROM node:6
MAINTAINER UCF Project <ucf-project@alterway.fr>

# Install AATT
# Note: There are no versions/tags so we make sure to checkout the
# commit HASH that we tested and know to work
RUN git clone https://github.com/paypal/AATT.git && \
    cd AATT/ && \
    git checkout 6bfa87ace2ad345ede9b107c0f4c63d47b33c2f2 && \
    npm install --unsafe-perm --quiet && \
    git submodule init && \
    git submodule update

WORKDIR AATT/

# Default configuration
ENV DEBUG "AATT*"
ENV http_port 3000

EXPOSE 3000

CMD ["node", "app.js"]
