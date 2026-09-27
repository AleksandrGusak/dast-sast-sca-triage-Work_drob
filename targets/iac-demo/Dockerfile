FROM ubuntu:18.04
USER root
ADD https://example.com/install.sh /tmp/install.sh
RUN chmod +x /tmp/install.sh && /tmp/install.sh
EXPOSE 22
ENV SECRET_KEY="hardcoded_secret_12345"
CMD ["/bin/bash"]
