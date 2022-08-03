---
title: "Next.js(TS) + next-authでCognito認証をする方法【2022/08】"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs","react","nextauth","aws","cognito"]
published: true
---

# 目的
* Next.js(TS) + next-authを使用してCognito認証を実装する。
* 最近の仕様やGUIで、初心者向け公開されている記事がなかった為、執筆<br />
    最近next-authのプロパティ名や、Cognito周りのGUIが変わっている。

# 想定読者
* 初心者
* まずはNext.js + Cognitoで認証したい人
  (Next.js自体の構築は、本記事では触れません。)

# 注意事項
* ここでの設定のみでは、セキュリティー的に危険な可能性があります。
  自己判断自己責任にてお願いいたします。
* 記載されている内容で実施できない場合、仕様が変更されている可能性があります。<br />
  公式ドキュメントを確認してください。

# 実施方法

## AWS Cognitoの手順

1. Cognito画面を開きます。
!["Cognito画面への遷移"](/images/articles/next-auth-hello/1.png)

2. GUIを最新版にする。
下記画面のような表示ではない場合、新しいコンソールに切り替えてください。
（画面内に切り替え用のリンクが表示されているはずです。）
!["コンソールの確認"](/images/articles/next-auth-hello/2.png)

3. 「ユーザープールを作成」を実行
4. サインインエクスペリエンスを設定
!["サインインエクスペリエンスの設定"](/images/articles/next-auth-hello/3.png)
    * プロバイダーのタイプ: Cognitoユーザープール
    * Cognito ユーザープールのサインインオプション : Eメール

5. セキュリティ要件を設定
!["セキュリティ要件を設定"](/images/articles/next-auth-hello/4.png)
    * パスワードポリシー: Cognitoのデフォルト
    * 多要素認証 : MFAなし
    * ユーザーアカウントの復旧 : とりあえず試したいだけの場合、なんでも良いです

6. サインアップエクスペリエンスを設定
!["サインアップエクスペリエンスを設定"](/images/articles/next-auth-hello/5.png)
    * 必須の属性: `nickname`を選択
    　emailはサインインオプションとなっている為、不要
    * 他の項目はそのままでOK

7. メッセージ配信を設定
!["メッセージ配信を設定"](/images/articles/next-auth-hello/6.png)
    * E メールプロバイダー : Cognito で E メールを送信
    * 送信元 E メールアドレス : (デフォルトのまま) `no-reply@verificationemail.com`
    * 返信先 E メールアドレス : 未指定もしくはご自身のメールアドレス

8. アプリケーションを統合
    * ユーザープール名: 任意のプール名を入力

!["ホストの設定"](/images/articles/next-auth-hello/7.png)
    * ホストされた認証ページ : Cognito のホストされた UI を使用
    * ドメインタイプ : Cognito ドメインを使用する
    * Cognito ドメイン : `https://{任意のドメイン}`
<br />
!["アプリケーションクライアント"](/images/articles/next-auth-hello/8.png)
    * アプリケーションタイプ : 秘密クライアント
    * アプリケーションクライアント名 : 任意のクライアント名
    * クライアントのシークレット : クライアントのシークレットを生成する
    * 許可されているコールバック URL : `http://localhost:3000/api/auth/callback/cognito`
<br />
!["高度なアプリケーションクライアントの設定"](/images/articles/next-auth-hello/9.png)
    * IDプロバイダー : Cognitoユーザープール
    * OAuth 2.0 権限タイプ : 認証コード付与
    * OpenID 接続スコープ : `OpenId` `Eメール` `プロファイル`

9. 確認して → ユーザープールを作成

## Next.js(next-auth)の手順

1. next-authをインストールする <br>
`npm install next-auth` 

2. `pages/auth/[...nextauth].ts` を作成する <br />
```
import NextAuth from 'next-auth';
import CognitoProvider from "next-auth/providers/cognito"

export default NextAuth({
  providers: [
    CognitoProvider({
      clientId: '画像1参照',
      clientSecret: '画像1参照',
      issuer: '画像3参照'
    })
  ]
})
```

3. 「2.」で作成したファイルのそれぞれの設定値を確認する: clientId, clientSecret<br>
    1. AWS→Cognito→ユーザープールで先ほど作成したユーザープールを選択
    2. 「アプリケーションの統合」を選択
    3. 「アプリクライアントと分析」に１アプリリンクが表示されている為、選択
    !["クライアント表示"](/images/articles/next-auth-hello/10.png)
    4. 転記する
        * クライアントID : clientId
        * クライアントのシークレット : clientSecret
!["クライアントIDとクライアントシークレット"](/images/articles/next-auth-hello/11.png)

4. 「2.」で作成したファイルのそれぞれの設定値を確認する: issuer
    1. 下記設定値がベースとなる。「Region」と「user pool Id」を調べる
       `https://cognito-idp.{Region}.amazonaws.com/{user pool Id}`
    2. Regionを調べる 
       AWSコンソールの画面右上に地域が表示されているのでクリック。現在表示されているリージョンのIdが表示されます。
       この場合「ap-northeast-1」となります。
    !["リージョン表示"](/images/articles/next-auth-hello/12.png)
    3. user pool Idを調べる
    　 Cognito→ユーザープール→作成したユーザープールを選択
       「ユーザープール ID」が記載されています。
    !["user pool id 表示"](/images/articles/next-auth-hello/13.png)

    4. urlを組み立ててissuerに転記する。
    例) ※user pool idが`hoge`だった場合
       `https://cognito-idp.ap-northeast-1.amazonaws.com/hoge`

5. signIn呼び出し部分の作成
   適当なpageを作成して、下記をコピー

```
import { useSession, signIn, signOut } from "next-auth/react"
import React from "react"

export default function Component() {
  const { data: session, status } = useSession()
  
  if (session) {
    return (
      <>
        Signed in as {session.user?.email} <br />
        <button onClick={() => signOut()}>Sign out</button>
      </>
    )
  }
  return (
    <>
      Not signed in <br />
      <button onClick={() => signIn()}>Sign in</button>
    </>
  )
}
```


6. 実行確認
   
   1. 起動してsignIn処理を描いたページへ
   !["signIn 1"](/images/articles/next-auth-hello/14.png)
   2. signUpする
   !["signIn 2"](/images/articles/next-auth-hello/15.png)
   !["signIn 3"](/images/articles/next-auth-hello/16.png)
   !["signIn 4"](/images/articles/next-auth-hello/17.png)
   3. 確認コードが入力したメールアドレスに届いている為、入力して「Confirm Account」を実行
   !["signIn 5"](/images/articles/next-auth-hello/18.png)
   4. signIn済みになったら成功です！！
   !["signIn 6"](/images/articles/next-auth-hello/19.png)

   5. user pool上からもユーザーが追加できたことを確認できます。
   !["signIn 6"](/images/articles/next-auth-hello/20.png)


# 追加作業
このままでは、`pages/auth/[...nextauth].ts`にセキュアな情報が記載されており、GitHubなどに上げることが出来ません。<br />
ここまで長くてお疲れかと思いますが、もう一息頑張りましょう。<br />

1. アプリケーションのルートに`.env.local`を作成
2. 現在`pages/auth/[...nextauth].ts`に記載されているセキュアな情報を環境変数として定義。
```
COGNITO_CLIENT_ID=
COGNITO_CLIENT_SECRET=
COGNITO_ISSUER=
```

3. `pages/auth/[...nextauth].ts`に記載されているセキュアな情報を環境変数に置き換える。

```
import NextAuth from 'next-auth';
import CognitoProvider from "next-auth/providers/cognito"

export default NextAuth({
  providers: [
    CognitoProvider({
      clientId: process.env.COGNITO_CLIENT_ID || '' ,
      clientSecret: process.env.COGNITO_CLIENT_SECRET || '',
      issuer: process.env.COGNITO_ISSUER 
    })
  ]
})
```
※ TypeScriptを有効にしている場合、型定義を通す必要がある為、 `|| ''` として対応している。



# 参考

### 公式系
* next-auth GitHub : https://github.com/nextauthjs/next-auth
* next-authの最近変更があった箇所 : https://next-auth.js.org/getting-started/upgrade-v4

### Zenn記事
* [Next.js+TSでNextAuth.jsを使ってcognito認証を行う](https://zenn.dev/yui/articles/b48a74ed717c6c)
* [Next.js アプリに Cognito の認証を良い感じに設定する](https://zenn.dev/tatsurom/articles/next-auth-cognito)

### Qiita記事
* [NextAuth.js v4 + Amazon Cognitoを試してみる](https://qiita.com/junkor-1011/items/035f41aae0c25bad4483)


