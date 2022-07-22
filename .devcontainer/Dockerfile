FROM node

# node.js の環境変数を定義する
# 本番環境では production
ENV NODE_ENV=development

# 雛形を生成するのに必要なパッケージのインストール
RUN npm install -g npm zenn-cli && apt-get update && apt-get install -y vim 

# ディレクトリを移動する
WORKDIR /app

# ポート 3000:node, 8000:zenn-cli
EXPOSE 3000 8000