🌐 **言語 / Language:** [한국어](../README.md) | [English](./README_en.md) | [日本語](./README_ja.md) | [中文](./README_zh.md)

---

# AWS AI IaaS 統合 AI ソリューション提案書（ブローシャー）
**Lex · Rekognition · Textract · Bedrock · Kendra · Lambda ベース**  
**OCR / 文書自動化 · PDF 出力 · QR コード · 顧客別 AI エージェント開発**

---

## 1. 概要（エグゼクティブサマリー）

本書は、AWS のマネージド AI サービスとサーバーレス / クラウドネイティブ技術スタックを組み合わせ、顧客ごとの要件（業務プロセス、データガバナンス、セキュリティ、SLA）に対応した AI ソリューションを設計・構築・運用するための統合提供モデルをまとめた提案ブローシャーです。

主なサービス領域は以下のとおりです。

1. 会話型 AI（コールセンター / チャットボット）  
2. ビジョン AI（顔 / オブジェクト / テキスト認識）  
3. ドキュメント AI（OCR · データ抽出）  
4. 生成 AI / RAG（社内ドキュメントに基づく回答）  
5. ドキュメント自動出力（PDF）  
6. QR コードによる業務連携  
7. 顧客別 AI エージェント開発  

---

## 2. ソリューションポートフォリオ

### 2.1 会話型 AI — Amazon Lex ベースの業務チャットボット / 音声ボット

- マルチターン対話（インテント / スロット）で業務パラメーターを収集 → Lambda 実行（内部 API / DB 連携）
- コールセンター / IVR / カスタマーサポート：FAQ、注文・配送照会、アカウント・権限管理、受付・苦情対応、予約・変更
- 音声入出力拡張：Transcribe（音声 → テキスト）、Polly（テキスト → 音声）連携

### 2.2 ビジョン AI — Amazon Rekognition ベースの画像 / 動画分析

- 顔検出 / 分析：DetectFaces で顔の位置・信頼度・ランドマーク・姿勢・遮蔽などの属性を返却
- 顔照合 / 本人確認：登録写真 vs. 提出写真の照合、重複登録検出、入退室認証
- ラベル（オブジェクト / シーン）検出およびテキスト検出によるコンテンツ自動タグ付け / 検査パイプライン構成

### 2.3 ドキュメント AI — Amazon Textract OCR / 文書データ化

- スキャン PDF / 画像からテキスト・手書き・テーブル・フォームデータを自動抽出
- 伝票・領収書・契約書・身分証・点検表・検収書などのデータ化と検証ワークフローを提供
- ルールベース精製 + LLM 後処理（フィールド標準化 / 要約 / 異常値検出）による品質向上

### 2.4 生成 AI / RAG — Bedrock · Kendra ベースの社内知識 Q&A

- 社内文書（S3 / ポータル / Wiki / ポリシー / マニュアル）の検索（Kendra）+ 生成型回答（Bedrock）
- 回答根拠（参照リンク / ドキュメントスニペット）を提供；ポリシーガードレール（禁止ワード / PII 等）を適用
- 顧客別テナント / ナレッジベースの分離（権限・暗号化・ロギングを含む）

### 2.5 PDF 出力 — レポート / 証明書 / 検収書 / 契約書の自動生成

- テンプレートベースの PDF レンダリング（表 / 書式 / 印影 / 署名画像を含む）、大量バッチ出力
- 生成履歴 / バージョン管理、透かし、パスワード / 権限設定、監査ログ
- 成果物保管（S3）+ ダウンロード API + 有効期限ポリシー（Lifecycle）

### 2.6 QR コード — オフライン-オンライン業務連携

- 文書 / 資産 / 現場設備 / 入館証 / 作業指示書に QR コードを挿入 → モバイルで即時照会・登録・承認
- QR コードにはトークン / 署名ベースの短縮 URL（有効期限 / 1回限り / 権限設定）を適用可能
- スキャンイベントログ / 業務履歴の自動収集（現場 DX）

### 2.7 顧客別 AI エージェント開発 — 業務オーケストレーションエージェント

- 顧客システム（API / DB / グループウェア）のツールを安全に接続し、「会話で業務を実行」
- 例：「契約書を要約して承認申請して」「現場点検 PDF を生成して担当者に配布して」
- ポリシーベースのツール使用（権限 / 承認 / 監査）、テナント分離、プロンプト / 知識 / ツールセットのカスタマイズ

---

## 3. リファレンスアーキテクチャ（代表的な構成）

### 3.1 イベント駆動型ビジョン / ドキュメント処理パイプライン

- S3 アップロード → （S3 イベント）Lambda → Rekognition / Textract → 結果保存（DynamoDB / RDS / OpenSearch）→ 通知（SNS）
- トラフィック急増 / 再処理が必要な場合：S3 → SQS → Lambda（バッファリング、**DLQ**、リトライ）パターンを適用

### 3.2 Lex 業務チャットボット + 内部システム連携

- Lex（インテント / スロット）→ Fulfillment Lambda → 内部 API（ERP / CRM / グループウェア）→ 結果応答
- 長時間タスク：進捗更新メッセージ + 非同期ジョブ（キュー / Step Functions）で UX を改善

### 3.3 RAG ナレッジボット（社内文書ベース）

- 文書収集 / 精製 → インデックス作成（Kendra / ベクター）→ クエリ → 検索結果 → Bedrock 応答生成 + ポリシー（ガードレール）

---

## 4. 構築 / 運用方法論

- **フェーズ 1（2〜4 週間）**：要件 / セキュリティ / データ分類、PoC（精度 / レイテンシ / コスト）
- **フェーズ 2（4〜8 週間）**：MVP 構築（コアシナリオ / 権限 / 監査 / デプロイ自動化）
- **フェーズ 3（継続）**：高度化（品質、ユーザーフィードバック、知識 / ツール拡張、運用指標ベースの最適化）

---

## 5. 必要な AWS 技術スタック（チェックリスト）

### 5.1 コンピューティング / 統合
- AWS Lambda、（オプション）Step Functions、EventBridge
- API Gateway / ALB、SQS、SNS

### 5.2 データ / ストレージ
- S3（原本 / 成果物）、DynamoDB（メタ / 履歴）、RDS（業務データ）、（オプション）OpenSearch（検索）
- KMS（暗号化）、S3 Lifecycle / バージョン管理

### 5.3 AI サービス
- Lex（会話）、Rekognition（ビジョン）、Textract（OCR）、Bedrock（生成型）、Kendra（検索 / RAG）
- （オプション）Transcribe / Polly（音声）

### 5.4 セキュリティ / ネットワーキング
- IAM（最小権限）、VPC / セキュリティグループ、（可能であれば）VPC エンドポイント / PrivateLink、WAF
- Secrets Manager / Parameter Store、CloudTrail（監査）

### 5.5 オブザーバビリティ / 運用
- CloudWatch Logs / Metrics / Alarms、X-Ray（分散トレース）、障害アラート連携

### 5.6 DevOps / IaC
- CloudFormation / CDK / Terraform、CodeBuild / CodePipeline または GitHub Actions

---

## 6. 代表的な適用シナリオ

- **現場点検**：QR スキャン → チェックリスト入力 → 写真アップロード → Rekognition / Textract 自動分析 → 点検 PDF 自動生成・配布
- **入退室 / 本人確認**：アプリで撮影 → Rekognition 顔照合 → 通過時に入館証 PDF / QR 発行 + 監査ログ保存
- **文書自動化**：契約書 / 伝票 PDF アップロード → Textract 抽出 → フィールド検証 → RAG による関連規定 / ガイドの自動提示
- **カスタマーサポート**：Lex 音声ボットが問い合わせ受付 → 内部システムチケット作成 → 進捗状況をチャットで案内
- **社内ナレッジボット**：ポリシー / マニュアルベースの検索 + 要約回答（Bedrock / Kendra）+ 根拠リンクを提供

---

## 7. 参考（公式ドキュメント / サービス説明）

- [提案書 PDF を開く](./AWS_AI_Solution_Company_Brochure_ko.pdf)

- Amazon Rekognition DetectFaces API: https://docs.aws.amazon.com/rekognition/latest/APIReference/API_DetectFaces.html  
- AWS Lambda — Amazon S3 イベント通知を Lambda で処理: https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html  
- Amazon Lex V2 — Lambda の統合: https://docs.aws.amazon.com/lexv2/latest/dg/lambda.html  
- Amazon Textract OCR: https://aws.amazon.com/textract/ocr/  
- Amazon Textract 概要: https://aws.amazon.com/textract/  
