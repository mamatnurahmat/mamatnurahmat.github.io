---
layout: post
title: Temporal Update Note!
---

Repository temporal disimpan pada repo [git-ops](https://bitbucket.org/loyaltoid/gitops-k8s/src/master/) pada branch tertentu sesuai dengan project.
Pada saat dokumentasi ini dibuat update temporal sudah di lakukan pada project crypto yang disimpan pada [git-ops](https://bitbucket.org/loyaltoid/gitops-k8s/src/temporal-crypto/) pada branch **temporal-crypto**.

## <b>Catatan Penting Untuk Update Version Temporal</b>
Temporal dengan versi tertentu harus juga di sesuaikan dengan versi schema database agar tidak ada error pada database temporal.
??? tip "Versi temporal dan schema DB"
    ``` sh
    Upgrade MySQL schema version 1.6
    https://github.com/temporalio/temporal/releases/tag/v1.14.5  

    Upgrade MySQL schema version 1.8 
    https://github.com/temporalio/temporal/releases/tag/v1.15.2  
    https://github.com/temporalio/temporal/releases/tag/v1.16.2  

    Upgrade MySQL schema version 1.9
    https://github.com/temporalio/temporal/releases/tag/v1.17.5  
    https://github.com/temporalio/temporal/releases/tag/v1.18.0 
    https://github.com/temporalio/temporal/releases/tag/v1.19.1   
    https://github.com/temporalio/temporal/releases/tag/v1.20.0
    ```

## <b>1. Step Update Version Temporal</b>

!!! info "Information"
    Proses update versi temporal pada dokumentasi ini dilakukan pada temporal-crypto

#### 1.1. Backup Database
Pastikan database temporal di backup terlebih dahulu.

#### 1.2. Clone Temporal 
Clone temporal berdasarkan versi temporal yang ada di deploy dengan script sebagai berikut :

??? tip "Clone Temporal v1.20.0"
    ``` sh
    export TEMPORAL_VERSION="v1.20.0"
    git clone https://github.com/temporalio/helm-charts.git -b $TEMPORAL_VERSION temporal-$TEMPORAL_VERSION 
    cd temporal-$TEMPORAL_VERSION
    ```

!!! info "Information"
    Tolong pastikan versi temporal yang akan di deploy harus dengan versi stable

#### 1.3. Update Dependensi Chart Helm
Pada directory **temporal-$TEMPORAL_VERSION** (temporal-v1.20.0) lakukan update dependensi chart dengan perintah sebagai berikut :
??? tip "Update Dependensi Chart"
    ``` sh
    helm dep update 
    ```

#### 1.4. Konfigurasi Helm Temporal
Pada step ini untuk mengambil values konfigurasi pada helm yangada dengan nama **temporal-crypto** di namespace **temporal-crypto** dan menyimpannya dalam file **temporal-crypto.values.yaml** yang dimana untuk dilakukan upgrade versi chart helm pada step berikutnya, dengan perintah sebagai berikut : 
??? tip "Konfigurasi Helm Temporal"
    ``` sh
    helm -n temporal-crypto get values temporal-crypto > temporal-crypto.values.yaml
    ```
Setelah itu copy file **temporal-crypto-frontend.yaml** dan **temporal-crypto-web.yaml** , dengan perintah sebagai berikut :

??? tip "Copy File"
    ``` sh
    cp ../temporal-crypto-frontend.yaml .
    cp ../temporal-crypto-web.yaml .
    ```

#### 1.5. Upgrade Chart Helm Temporal
Lakukan upgrade chart helm **temporal-crypto** pada namespace **temporal-crypto** dengan mengset ***prometheus.enabled*** dan ***grafana.enabled*** menjadi **false** agar pada proses upgrade chart helm di nonkatifkan sementara hingga proses upgrade selesai, dengan perintah sebagai berikut : 
??? tip "Upgrade Chart Helm"
    ``` sh
    helm -n temporal-crypto upgrade temporal-crypto -f temporal-crypto.values.yaml --set prometheus.enabled=false --set grafana.enabled=false .
    ```

#### 1.6. Deploy Service Temporal
Deploy temporal dengan versi yang telah di tetapkan pada step awal dan pastikan manifest service telah di set dengan menggunakan ***nodeport***, dengan berintah sebagai berikut : 
??? tip "Deploy Temporal"
    ``` sh
    kubectl -n temporal-crypto patch svc temporal-crypto-frontend -p "$(cat temporal-crypto-frontend.yaml)
    kubectl -n temporal-crypto patch svc temporal-crypto-web -p $(cat temporal-crypto-web.yaml)
    ```

## <b>2. Update Schema Database SQL</b>
#### 2.1. Update Schame Database SQL
Sesuaikan krendesial terlebih dahulu dengan benar lalu update schame databasenya, dengan perintah sebagai berikut :
??? tip "Update schema DB"
    ``` sh
    export SQL_PLUGIN=mysql
    export SQL_HOST=192.172.1.2
    export SQL_PORT=3306
    export SQL_USER=temporaluser
    export SQL_PASSWORD=Pwtemporal123!

    SQL_DATABASE=temporal temporal-sql-tool update -schema-dir schema/mysql/v57/temporal/versioned
    SQL_DATABASE=temporal_visibility temporal-sql-tool update -schema-dir schema/mysql/v57/visibility/versioned
    ```