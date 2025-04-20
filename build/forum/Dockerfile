FROM php:8.2-apache
RUN set -eux; \
    usermod --uid 1000 www-data; \
    groupmod --gid 1001 www-data
ENV PHPBB_MAJOR_VERSION 3.3
ENV PHPBB_VERSION 3.3.15

# TODO: Find or debug production libzip, libicu and libpng packages (should be libzip4, libicu67 and libpng16-16, but all throw version errors when installing PHP extensions that depend on them)
### INSTALL SETUP DEPENDENCIES ###
RUN set -eux; \
    apt update; \
    apt install -y --no-install-recommends \
        unzip \
        zlib1g-dev \
        libpng-dev \
    ; \
    rm -rf /var/lib/apt/lists/*

### INSTALL PHP EXTENSIONS ###
RUN set -eux; \
    \
    savedAptMark="$(apt-mark showmanual)"; \
    \
    docker-php-ext-install -j "$(nproc)" \
        opcache \
        mysqli \
        gd \
    ; \
    rm -r /tmp/pear; \
    \
    # Reset apt-mark's "manual" list so "purge --auto-remove" removes all build deps
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual $savedAptMark; \
    ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
        | awk '/=>/ { print $3 }' \
        | sort -u \
        | xargs -r dpkg-query -S \
        | cut -d: -f1 \
        | sort -u \
        | xargs -rt apt-mark manual; \
    \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /var/lib/apt/lists/*

COPY opcache.ini /usr/local/etc/php/conf.d/opcache.ini

### CONFIGURE APACHE ###

RUN a2enmod rewrite

WORKDIR /var/www/html

#### phpBB ####

### DOWNLOAD AND EXTRACT phpBB ###

RUN set -eux; \
    curl -fSL "https://download.phpbb.com/pub/release/${PHPBB_MAJOR_VERSION}/${PHPBB_VERSION}/phpBB-${PHPBB_VERSION}.tar.bz2" -o phpbb.tar.bz2; \
    tar xjvf phpbb.tar.bz2; \
    rm phpbb.tar.bz2; \
    mv phpBB3/ board/

### CUSTOMIZE ###

## STYLES ##
WORKDIR /var/www/html/board/styles/
RUN set -eux; \
    curl -fSL https://www.phpbb.com/customise/db/download/207511 -o cabot-damaio.zip; \
    unzip '*.zip'; \
    rm *.zip

## EXTENSIONS ##
WORKDIR /var/www/html/board/ext/
RUN set -eux; \
    curl -fSL https://www.phpbb.com/customise/db/download/184456 -o rmcgirr83-activity24hours.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/202386 -o mattf-abbc3.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/201496 -o hifikabin-attachmentfluffbuster.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/205024 -o rmcgirr83-autodbbackup.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/206731 -o phpbb-boardannouncements.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/183386 -o rmcgirr83-bdaycake.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/184401 -o raul-categoryimage.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/203566 -o phpbb-collapsiblecategories.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/191431 -o dmzx-copyrightextended.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/197491 -o chris1278-counter.zip; \
    curl -fSL https://github.com/cabot/damaioconfiguration/releases/download/v1.0.1/cabot_damaioconfiguration.zip -o cabot-damaioconfig.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/203899 -o cabot-discordnotifs.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/204688 -o lukewcs-extmgrplus.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/177261 -o davidiq-forumbanners.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/187236 -o rmcgirr83-genders.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/195666 -o luoning-htmlentities.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/205018 -o crizzo-ipanon.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/205255 -o hifikabin-largefont.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/168166 -o senky-massnotif.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/182261 -o kasimi-mchat.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/156941 -o kasimi-movetopicswhenlocked.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/142621 -o kasimi-markpostunread.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/166116 -o kasimi-postnumbers.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/179336 -o rxu-membershiptimespan.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/173621 -o studio-nore.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/201626 -o mattf-passstrength.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/201261 -o gwenythllewelyn-postlocalstorage.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/205027 -o hifikabin-scrollquotedtext.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/202633 -o hifikabin-showtransferedperms.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/200036 -o mattf-similartopics.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/201276 -o paul-simplementions.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/202071 -o primehalo-primenotify.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/198236 -o primehalo-primepostrevisions.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/197371 -o mattf-pmboxbars.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/179661 -o flashtvr-flairs.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/191446 -o cabot-replytoggle.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/192101 -o dmzx-scheduledmaintenance.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/204724 -o alfredoramos-seo.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/180681 -o rmcgirr83-separatebots.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/205282 -o rmcgirr83-stopspam.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/174896 -o mattf-topicpreview.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/183726 -o paul-2fa.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/187256 -o rmcgirr83-upcomingbdays.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/187941 -o rmcgirr83-whoposted.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/206857 -o phpbb-webpushnotifications.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/205945 -o caforum-clickableheaderbar.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/205936 -o alex75-smiliesscrollbox.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/205258 -o andreask-ium.zip; \
    curl -fSL https://www.phpbb.com/customise/db/download/192771 -o phpbbmodders-adduser.zip; \
    curl -fSL https://github.com/kasimi/phpbb-ext-mchatonlinebanners/archive/refs/tags/1.1.2.zip -o kasimi-mchatonlinebanners.zip; \
    curl -fSL https://github.com/kasimi/phpbb-ext-mchaticonsmenu/archive/refs/tags/1.2.2.zip -o kasimi-mchaticonsmenu.zip; \
    curl -fSL https://github.com/kasimi/phpbb-ext-mchatinforumsandtopics/archive/refs/tags/1.2.1.zip -o kasimi-mchatinforumsandtopics.zip; \
    unzip '*.zip'; \
    rm *.zip

### CONFIGURE PERMISSIONS AND OWNERSHIP ###

WORKDIR /var/www/html/board/

COPY phpbb/phpbb-install.yml phpbb-install.yml

RUN set -eux; \
    chmod 666 config.php; \
    chmod 644 app.php common.php composer.json composer.lock cron.php faq.php feed.php .htaccess index.php mcp.php memberlist.php posting.php report.php search.php ucp.php viewforum.php viewonline.php viewtopic.php web.config; \
    chmod -R 755 adm/ assets/ bin/ config/ docs/ download/ ext/ images/ includes/ install/ language/ phpbb/ styles/ vendor/; \
    chmod -R 777 files/ cache/ store/ images/avatars/upload/; \
    chown -R www-data:www-data ./; \
    chown www-data:www-data phpbb-install.yml
