FROM php:8.3-apache

# 環境変数の取得または設定
ARG WWWUSER=1000
ARG WWWGROUP=1000
ARG NODE_VERSION=22

# Apacheのユーザーとグループを設定
ENV APACHE_RUN_USER=work
ENV APACHE_RUN_GROUP=work

RUN \
    # アップデートしておく
    apt-get update \

    # コンテナ内の新規ユーザー作成
    && groupadd --gid ${WWWGROUP} work \
    && useradd --uid ${WWWUSER} --gid ${WWWGROUP} -m -s /bin/bash work \

    # ホームディレクトリからアクセスしやすいようにシンボリックシンクを作っておく
    && ln -s /var/www/html /home/work/htdocs \
    && chown -h work. /home/work/htdocs \

    # コンテナ内で作成したユーザーにsudo権限を付与
    && apt-get install -y sudo \
    && echo 'work ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/work \
    # NOPASSWDはパスワード無しでsudoを実行できるオプション
    && chmod 440 /etc/sudoers.d/work \

    # 日本語を含む多言語が使用できるように設定
    && apt-get install -y locales \
    && locale-gen C.UTF-8 && /usr/sbin/update-locale LANG=C.UTF-8 \

    # タイムゾーンを日本標準時に変更
    && ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime \
    && echo 'Asia/Tokyo' > /etc/timezone \

    # viエディタで編集できるようにインストール
    && apt-get install -y vim \

    # composerコマンドを使用できるようにインストールする
    && apt-get install -y git unzip \
    && curl -sLS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer \

    # npmコマンドを使用できるようにインストールする
    && curl -sLS https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash \
    && apt-get install -y nodejs \
    && npm install --global yarn \

    # mysqlコマンドを使用できるようにインストールする
    && apt-get install -y default-mysql-client \
    # 常にSSLを使わないようにmy.cnfファイルに追記
    && printf "\n[client]\nssl=0\n" >> /etc/mysql/my.cnf \

    # psqlコマンドを使用できるようにインストールする
    && apt-get install -y postgresql-client \

    # wgetコマンドを使用できるようにインストールする
    && apt-get install -y wget \

    # zipコマンドを使用できるようにインストールする
    && apt-get install -y zip \

    # sendmailでmailpitホストに転送できるようにmhsendmailをインストール
    && curl -sSL https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_amd64 -o mhsendmail \
    && chmod +x mhsendmail \
    && mv mhsendmail /usr/local/bin/mhsendmail \

    # 必要なPHP拡張ライブラリをインストール
    && apt-get install -y libonig-dev libicu-dev libpng-dev libpq-dev libzip-dev \
    && docker-php-ext-install mbstring intl gd exif pdo_mysql mysqli pdo_pgsql pgsql zip bcmath pcntl \
    && apt-get install -y libmagickwand-dev \
    && [ $http_proxy ] && pear config-set http_proxy $http_proxy || true \
    && pecl install -f imagick xdebug \
    && docker-php-ext-enable imagick xdebug \

    # Apacheモジュールの有効化
    && a2enmod cgi \
    && ln -s /usr/bin/perl /usr/local/bin/perl \
    && a2enmod rewrite \

    # HTTPS有効化
    && apt-get install -y ssl-cert \
    && make-ssl-cert generate-default-snakeoil \
    && a2enmod ssl \
    && a2ensite default-ssl \
    # workユーザーから参照できるようにパーミッションを変更
    && chmod 755 /etc/ssl/private \
    && chmod 644 /etc/ssl/private/ssl-cert-snakeoil.key \

    # キャッシュされたパッケージファイル等を削除
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# 環境変数セット
ENV LANG=C.UTF-8 LANGUAGE=C.UTF-8 LC_ALL=C.UTF-8

# デフォルトユーザーを変更
USER ${WWWUSER}:${WWWGROUP}

# Podman環境への対応
CMD sudo -E apache2ctl start && tail -f /dev/null
