#
# Copyright 2018 Confluent Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.


FROM confluentinc/cp-kafka-connect:5.4.9-1-ubi8

USER root
RUN yum install -y unzip

USER appuser
WORKDIR /tmp
# Install debezium connector for Oracle (https://repo1.maven.org/maven2/io/debezium/debezium-connector-oracle/1.9.5.Final/debezium-connector-oracle-1.9.5.Final-plugin.tar.gz)
RUN wget https://repo1.maven.org/maven2/io/debezium/debezium-connector-oracle/1.9.5.Final/debezium-connector-oracle-1.9.5.Final-plugin.tar.gz
RUN tar -xzvf debezium-connector-oracle-1.9.5.Final-plugin.tar.gz -C /usr/share/java/

# Install JDBC driver for Oracle (https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html)
RUN wget https://download.oracle.com/otn_software/linux/instantclient/217000/instantclient-basic-linux.x64-21.7.0.0.0dbru.zip
RUN unzip instantclient-basic-linux.x64-21.7.0.0.0dbru.zip
RUN cp ./instantclient_21_7/ojdbc8.jar /usr/share/java/debezium-connector-oracle

USER root
RUN rm debezium-connector-oracle-1.9.5.Final-plugin.tar.gz
RUN rm instantclient-basic-linux.x64-21.7.0.0.0dbru.zip
RUN rm -rf instantclient_21_7

USER appuser