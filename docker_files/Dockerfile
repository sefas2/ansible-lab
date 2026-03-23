FROM frrouting/frr:latest
WORKDIR /app
COPY start.sh /app/start.sh
RUN chmod +x /app/start.sh
RUN apk add --no-cache openssh \
    && ssh-keygen -A \
    && echo "PermitRootLogin yes" >> /etc/ssh/sshd_config \
    && echo "root:cisco" | chpasswd \
    && sed -i 's|root:/bin/ash|root:/usr/bin/vtysh|' /etc/passwd \
    && echo "/usr/bin/vtysh" >> /etc/shells \
    && sed -i 's|ospfd=no|ospfd=yes|' /etc/frr/daemons
EXPOSE 22
CMD ["sh", "start.sh"]