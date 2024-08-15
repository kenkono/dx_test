FROM openjdk:8  # ベースイメージとしてOpenJDK 8を使用

# Debianのソースリストを設定し、パッケージリストを更新
RUN echo "deb http://deb.debian.org/debian/ bullseye main contrib" > /etc/apt/sources.list && \
    echo "deb http://deb.debian.org/debian/ bullseye-updates main contrib" >> /etc/apt/sources.list && \
    apt-get update && \
    # 必要なパッケージをインストール
    apt-get install -y sudo wget build-essential libreadline-dev libncursesw5-dev libssl-dev \
    libsqlite3-dev libgdbm-dev libbz2-dev liblzma-dev zlib1g-dev uuid-dev libffi-dev libdb-dev \
    python3.9 python3.9-dev python3-distutils locales default-mysql-client git vim postgresql-client && \
    # ロケールを設定
    localedef -f UTF-8 -i ja_JP ja_JP.UTF-8 && \
    # pipをインストール
    curl -k https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
    python3 get-pip.py && \
    pip install -U pip findspark confluent-kafka==1.7.0 avro==1.11.0 pdfminer.six==20211012 \
    pandas==1.3.5 openpyxl==3.0.9 xlrd==2.0.1 && \
    # pysparkユーザーにsudo権限を付与
    echo pyspark ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/pyspark && \
    chmod 0440 /etc/sudoers.d/pyspark && \
    # 必要なディレクトリを作成し、パーミッションを設定
    mkdir -p /home/pyspark /tmp/spark-events /var/log/spark /var/log/digdag && \
    chmod 777 -R /home/pyspark /tmp/spark-events /var/log/spark /var/log/digdag && \
    # pysparkグループを作成
    groupadd pyspark && \
    # pysparkユーザーを作成
    useradd -s /bin/bash pyspark -g pyspark && \
    # PostgreSQLのソースリストを設定
    touch /etc/apt/sources.list.d/pgdg.list && \
    echo "deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main" >> /etc/apt/sources.list.d/pgdg.list

    docker run -it --name my-pyspark-container my-pyspark-image

    docker run -it --name my-pyspark-container my-pyspark-image

# 環境変数を設定
ENV LANG ja_JP.UTF-8
ENV LANGUAGE ja_JP:ja
ENV LC_ALL ja_JP.UTF-8
ENV TZ JST-9

# ユーザーを切り替え
USER pyspark


# Sparkをセットアップ
WORKDIR /home/pyspark
RUN wget --no-check-certificate https://archive.apache.org/dist/spark/spark-3.2.4/spark-3.2.4-bin-hadoop3.2.tgz && \
    tar -xzvf spark-3.2.4-bin-hadoop3.2.tgz && \
    ln -s spark-3.2.4-bin-hadoop3.2 spark && \
    # MySQLコネクタをダウンロードして配置
    wget --no-check-certificate https://downloads.mysql.com/archives/get/p/3/file/mysql-connector-java-5.1.49.tar.gz && \
    tar -xzvf mysql-connector-java-5.1.49.tar.gz && \
    mv mysql-connector-java-5.1.49/mysql-connector-java-5.1.49.jar /home/pyspark/spark/jars

# 設定ファイルをコピー
COPY ./share/hive-site.xml /home/pyspark/spark/conf/
COPY ./share/log4j.properties /home/pyspark/spark/conf/
COPY ./hiveddls/2.3.0_utf-8.sql /home/pyspark/
COPY ./hiveddls/txn.sql /home/pyspark/
COPY ./kafka/kafka_2.13-3.0.2.tgz /home/pyspark/

# Kafkaをセットアップ
RUN tar -xzvf kafka_2.13-3.0.2.tgz && \
    ln -s kafka_2.13-3.0.2 kafka

# librdkafkaをセットアップ
RUN git clone https://github.com/edenhill/librdkafka.git
WORKDIR /home/pyspark/librdkafka/
RUN ./configure --install-deps && \
    ./configure --prefix=/usr && \
    make -j && \
    sudo make install

# ユーザー環境をセットアップ
USER pyspark
RUN echo "export SPARK_HOME=/home/pyspark/spark/" >> ~/.bashrc && \
    echo "export PATH=${PATH}:/home/pyspark/spark/bin" >> ~/.bashrc && \
    echo 'alias python="python3"' >> ~/.bashrc && \
    echo 'alias pip="pip3"' >> ~/.bashrc

WORKDIR /home/pyspark/