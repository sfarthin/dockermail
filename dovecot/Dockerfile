FROM mail-base

# prerequisites
RUN apt-get update

# install self-signed ssl certs
RUN apt-get install -y --force-yes ssl-cert

# Install postfix as MTA
RUN apt-get install -y --force-yes postfix

# Install dovecot as IMAP server
RUN apt-get install -y --force-yes dovecot-imapd

# Install SpamAssassin (Based on https://www.digitalocean.com/community/tutorials/how-to-configure-a-mail-server-using-postfix-dovecot-mysql-and-spamassasin)
RUN apt-get install -y spamassassin spamc
RUN adduser spamd --disabled-login

# ENABLE
RUN cp /etc/default/spamassassin spamassassin
RUN sed -e "s|ENABLED=0|ENABLED=1|g" spamassassin > /etc/default/spamassassin

# CRON
RUN cp /etc/default/spamassassin spamassassin
RUN sed -e "s|CRON=0|CRON=1|g" spamassassin > /etc/default/spamassassin

# OPTIONS
RUN cp /etc/default/spamassassin spamassassin
RUN sed -e "s|OPTIONS=.*|OPTIONS=\"--create-prefs --max-children 5 --username spamd --helper-home-dir \${SPAMD_HOME} -s \${SPAMD_HOME}spamd.log\"|g" spamassassin > /etc/default/spamassassin

# PIDFILE
RUN cp /etc/default/spamassassin spamassassin
RUN sed -e "s|PIDFILE=.*|PIDFILE=\"\${SPAMD_HOME}spamd.pid\"|g" spamassassin > /etc/default/spamassassin

# Add SPAMD_HOME
RUN echo 'SPAMD_HOME="/home/spamd/"' > spamassassin
RUN cat /etc/default/spamassassin >> spamassassin
RUN cp spamassassin /etc/default/spamassassin

# SpamAssassin settings
ADD spamassassin.cf /spamassassin.cf
RUN cp /spamassassin.cf /etc/spamassassin/local.cf

# postfix configuration
RUN echo "mail.docker.container" > /etc/mailname
ADD ./postfix.main.cf /etc/postfix/main.cf
ADD ./postfix.master.cf.append /etc/postfix/master-additional.cf
RUN cat /etc/postfix/master-additional.cf >> /etc/postfix/master.cf

# Adjustment to add spam assassin
RUN sed "s|#  -o syslog_name=postfix/submission|  -o content_filter=spamassassin|g" /etc/postfix/master.cf > master.cf
RUN cp master.cf /etc/postfix/master.cf

# configure mail delivery to dovecot
#ADD ./aliases /etc/postfix/virtual
#ADD ./domains /etc/postfix/virtual-mailbox-domains
RUN cp /aliases /etc/postfix/virtual
RUN cp /domains /etc/postfix/virtual-mailbox-domains

# todo: this could probably be done in one line
RUN mkdir /etc/postfix/tmp; awk < /etc/postfix/virtual '{ print $2 }' > /etc/postfix/tmp/virtual-receivers
RUN sed -r 's,(.+)@(.+),\2/\1/,' /etc/postfix/tmp/virtual-receivers > /etc/postfix/tmp/virtual-receiver-folders
RUN paste /etc/postfix/tmp/virtual-receivers /etc/postfix/tmp/virtual-receiver-folders > /etc/postfix/virtual-mailbox-maps

# give postfix the ownership of his files
RUN chown -R postfix:postfix /etc/postfix

# map virtual aliases and user/filesystem mappings
RUN postmap /etc/postfix/virtual
RUN postmap /etc/postfix/virtual-mailbox-maps

# add user vmail who own all mail folders
RUN groupadd -g 5000 vmail
RUN useradd -g vmail -u 5000 vmail -d /srv/vmail -m
RUN chown -R vmail:vmail /srv/vmail
RUN chmod u+w /srv/vmail

# dovecot configuration
ADD ./dovecot.mail /etc/dovecot/conf.d/10-mail.conf
ADD ./dovecot.ssl /etc/dovecot/conf.d/10-ssl.conf
ADD ./dovecot.auth /etc/dovecot/conf.d/10-auth.conf
ADD ./dovecot.master /etc/dovecot/conf.d/10-master.conf
ADD ./dovecot.lda /etc/dovecot/conf.d/15-lda.conf
ADD ./dovecot.imap /etc/dovecot/conf.d/20-imap.conf
# add verbose logging
#ADD ./internal/dovecot.logging /etc/dovecot/conf.d/10-logging.conf

# Add password file
RUN cp /passwords /etc/dovecot/passwd

# i'm not sure what expose actually does, so its mainly here for documentation
# smtp port for incoming mail
EXPOSE 25 
# imap port
EXPOSE 143
# smtp port for outgoing
EXPOSE 587

# todo: enable port 587 for outgoing mail, separate ports 25 and 587
# http://www.synology-wiki.de/index.php/Zusaetzliche_Ports_fuer_Postfix

# start necessary services for operation (dovecot -F starts dovecot in the foreground to prevent container exit)
 ENTRYPOINT chown -R vmail:vmail /srv/vmail; service rsyslog start; service spamassassin start; service postfix start; dovecot -F

