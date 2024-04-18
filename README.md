# Laravel(API)と Next.js(フロント)を docker 経由で構築ができるテンプレート

-   CORS→OK
-   ほっとリロード →OK
-   一つの docker-compose でフロント、バックエンド、DB など一括で起動可能

## 開発の流れ

-   Laravel（Laravel Breeze）は`/`(直下)のファイルで行う
    -   Laravel（API）は http://localhost
    -   API モードで使用するため、`/api`以降でデータをやり取りする
-   frontend（Next.js の開発）は`/web`で行う
    -   フロントは http://localhost:3000
    -   初期段階では昔の Laravel と同じ UI が表示されているが、全て next.js のフロントエンドである

## 開発環境構築時に最初にやること

1. `.env.example`を`.env`としてコピー

    - `/`(直下)と、`/web`の２つある

2. `docker-compose up -d`を実行

    - この時点ではまだサーバーが正常起動しない（はず）

3. PHP に必要なライブラリを設定
    - `docker-compose exec laravel.test composer install`
    - または laravel.test コンテナで`composer install`

以上で Docker の構築が完了し、開発ができるようになる

## 以下は最初の環境構築時にやったこと（今後実行する必要はない）

https://note.com/shinyeah/n/ndd9088ed746b これを見て構築したもの

1. `curl -s "https://laravel.build/{ディレクトリ名}" | bash` で、新しいディレクトリが作成される
2. `cd {ディレクトリ名} && git clone https://github.com/laravel/breeze-next.git frontend` で、フロントエンドのデータを追加、また、.env.sample を.env.local としてコピーする
3. docker-compose.yml に、以下の内容を追記

```yml
frontend:
    image: "node"
    volumes:
        - "./web:/var/www/html"
    working_dir: "/var/www/html"
    ports:
        - "3000:3000"
    command: bash -c "npm install && npm run dev"
phpmyadmin:
    depends_on:
        - mysql
    image: phpmyadmin/phpmyadmin
    restart: always
    ports:
        - "5005:80"
    environment:
        PMA_HOST: mysql
        MYSQL_ROOT_PASSWORD: password
    networks:
        - sail
```

4. `docker-compose up -d`で Docker を起動（以降は Docker Desktop で実行可能）

5. laravel を API モードで使用するため、以下コマンドを実行

```bash
docker-compose exec laravel.test bash # docker imageの中に入る

composer require laravel/breeze --dev

php artisan breeze:install api
```
