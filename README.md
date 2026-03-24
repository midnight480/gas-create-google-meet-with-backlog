# Google Meet URL作成 & Backlog連携アプリ

このプロジェクトは、Google Apps Script (GAS) を用いて、Google Meet の会議URLを作成し、Backlog の指定された課題に自動でコメント追加するツールです。

## 開発の背景

コミュニティ活動などでBacklogを活用している場合、**「何らかの課題をもとに会議を行う」** ことが一般的です。そのため本ツールは、単にGoogle MeetのURLを発行するだけでなく、Backlogの課題と紐付けることを前提とし、その指定を必須としています。

また、コミュニティの運用では関係者間でGoogle Calendarを共有して予定を管理することも多々あります。そこで、任意で**共有カレンダーID**を指定できるようにしました。これにより、1つのフォームから登録するだけで以下3つの作業を一括で自動化し、日々の運用を楽にすることを目指して作られています：
1. Google Meet のURL発行
2. Backlog 課題への追記（コメント）
3. 共有Googleカレンダーへの予定登録

## 主な機能

1. **Web UI による会議作成 (`Form.html`)**
   - 画面上から会議タイトル、日時、Backlog のスペースURL、課題キー、APIキーを入力すると、会議予定をGoogleカレンダーに作成し、発行されたGoogle MeetのURLをBacklog課題に自動コメントします。
2. **Web API エンドポイント (`doPost`)**
   - 外部システムからのPOSTリクエストを受け取り、自動的にデフォルト設定でGoogle Meet会議を発行し、Backlog課題へ連携します。

## 必要な Backlog 認証情報

本アプリ内で連携するためには、以下3つの情報が必要になります：
- **Backlog APIキー (`apiKey`)**: Backlogの個人設定から発行します
- **スペースURL (`spaceUrl`)**: 例 `https://your-space.backlog.com`
- **課題キー (`issueKey`)**: 例 `PROJECT-123`

※ 本コードの内部にはこれらの認証情報や個人のIDは**一切保存されていません**。すべて実行時のリクエストやユーザーフォーム入力から動的に取得されます。

## API リクエストの例 (curl)

設定したGASをウェブアプリとしてデプロイした後、以下のようにPOSTリクエストを送信して使用することも可能です。

```bash
curl -X POST "https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec" \
  -H "Content-Type: application/json" \
  -d '{"issueKey":"PROJECT-123","apiKey":"your_backlog_api_key","spaceUrl":"https://your-space.backlog.com"}'
```

## 自組織への新規導入・セットアップ手順 (他組織の方向け)

他の組織の方がこのツールを新たに自分たちの環境（別テナント）へ導入する場合は、以下の手順でそれぞれ独立したGASプロジェクトを作成・デプロイしてください。

1. **リポジトリのクローンとパッケージインストール**
   ```bash
   git clone git@github.com:midnight480/gas-create-google-meet-with-backlog-.git
   cd gas-create-google-meet-with-backlog-
   npm install
   ```
2. **Google アカウントへログイン**
   ```bash
   npx clasp login
   ```
3. **独自のGASプロジェクトを新規作成**
   ```bash
   npx clasp create --type webapp --title "Google Meet & Backlog連携ツール"
   ```
   ※実行すると、このディレクトリ内にデプロイ用の新しい `.clasp.json` が生成されます。
4. **コードを自身のGAS環境へ反映 (Push)**
   ```bash
   npx clasp push
   ```
5. **デプロイと初期設定 (ブラウザでの作業)**
   `npx clasp open` コマンドでGASエディタを開き、右上の「デプロイ」>「新しいデプロイ」から「ウェブアプリ」として発行してください。発行したURLを組織内の利用者に共有すれば完了です。

## 注意事項

- 本ツールは **Google Calendar API (v3)** を利用します。GASプロジェクトを紐付けているGoogle Cloud プロジェクト上で、Calendar API が「有効」になっていることを確認してください。
- Google Meet のURLを取得するために、カレンダーAPIでの `conferenceDataVersion=1` パラメータ及びOAuthスコープ制限が必要になります（`appsscript.json` に設定済み）。
- **デプロイと権限について:** 本ツールは「Google Workspaceでカレンダーの共有設定などを行う担当者」が、自身の権限でWebアプリとしてデプロイしてください。ツールはデプロイしたユーザーの認証情報を用いてGoogle MeetURLの発行やカレンダーの予定作成を行います。
- **セキュリティについて:** WebアプリとしてデプロイされたURLは推測されにくい文字列にはなりますが、Basic認証のような独自のアクセス制限などはかかっていません。デプロイ後のURLは必ず関係者のみに限定して共有してください。
- **利用者の運用について:** 本ツールを使うユーザー（Meetを発行したい人）は、デプロイ済みのURLにアクセスし、自身の「Backlog APIキー」を都度発行してフォームに入力するだけで利用できます。カレンダーやMeetなどの個別権限を利用者全員に付与する必要はありません。
