# Hands On reCAPTCHA Enterprise on Cloud Armor



## 概要

reCAPTCHA Enterprise を Cloud Armor を使って実装する

公式ブログ [Cloud Armor と reCAPTCHA Enterprise でアプリを bot から保護する](https://cloud.google.com/blog/ja/products/identity-security/bot-management-with-google-cloud)

以下のブログに詳細な詳細な手順などを記載している

+ Hejdaの見る夢 / Cloud Armor でお手軽に reCAPTCHA を実装してみる
  + https://iganari.hatenablog.com/entry/2022/01/04/234308

## やってみる


## 環境変数を用意しておく

+ Web アプリを作成時にしようした環境変数を設定しておきます

```
export _gcp_pj_id='Your GCP Project ID'
export _common='check-serverless-neg'
export _region='asia-northeast1'
```

## API の有効化

+ 自分の GCP Project 内で reCAPTCHA Enterprise の API の有効化します

```
gcloud beta services enable recaptchaenterprise.googleapis.com --project ${_gcp_pj_id}
```

+ 有効化出来ているかの確認します

```
gcloud services list --enabled --project ${_gcp_pj_id} | grep recaptchaenterprise
```

## Cloud Armor とそのルールを作成

+ Cloud Armor をのポリシーを作成します

```
gcloud beta compute security-policies create ${_common} --project ${_gcp_pj_id}
```

+ Cloud Armor 内のルールを作成します

```
gcloud beta compute security-policies rules create 1000 \
  --security-policy ${_common} \
  --expression "request.path.matches('/app')" \
  --action redirect \
  --redirect-type google-recaptcha \
  --project ${_gcp_pj_id}
```

+ ルールの確認します
  + ローカルに吐き出します

```
gcloud beta compute security-policies export ${_common} --file-name=${_common}.json --file-format=json --project ${_gcp_pj_id}
cat ${_common}.json
```

## Cloud Armror をアタッチ

+ App Engine 用の Backend Service check-serverless-neg-backend-service-app に Cloud Armor を設定します

```
gcloud compute backend-services update check-serverless-neg-backend-service-app \
  --security-policy ${_common} \
  --global \
  --project ${_gcp_pj_id}
```

---> 設定はこれで完了です :)


## 削除方法

+ [WIP] Delete Cloud Armor Policy

```
gcloud beta compute security-policies delete ${_common} --project ${_gcp_pj_id} -q
```

+ サーバレス系を削除する

```
https://github.com/iganari/handson-serverless-neg
```
