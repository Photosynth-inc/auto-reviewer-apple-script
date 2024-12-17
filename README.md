GitLabのマージリクエストに対して、Anthropic Claudeを使った自動レビューを行うAppleScriptです。MCPを使用してADRベースのレビューを行い、結果をGitLabに自動投稿します。

## 前提条件

- macOS Sonoma以降
- GitLab Runner
- Claude Desktop App
- [set-electron-app-accessible](https://github.com/JonathanGawrych/set-electron-app-accessible)
  - Electronアプリへのアクセシビリティ制御に使用
  - コンパイル済みバイナリを同一ディレクトリに配置してください

## セットアップ

手順は省略させていただきます

1. GitLab Runnerのインストール
2. アクセシビリティの設定 (GitLab Runner)
3. スクリプトの配置

## .gitlab-ci.yml の設定例

```yaml
workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"' # MRイベントの場合のみ実行

auto_review:
  stage: build
  script:
    - osascript ./script/auto-reviewer.scpt ${CI_PROJECT_ID} ${CI_MERGE_REQUEST_IID} ${CI_PROJECT_DIR} & PID=$!;
    - osascript ./script/monitor-process-and-directory.scpt $PID ${CI_PROJECT_DIR}
  tags:
    - mac-os
  timeout: 300s
```

## スクリプトの構成

```
.
├── README.md
├── auto-reviewer.scpt            # メインスクリプト
├── monitor-process-and-directory.scpt  # プロセス監視用スクリプト
└── sentence_structure/          # レビューコメントの構文テンプレート
```

## 注意事項

- 初回実行時
  - macOSのアクセシビリティ権限の承認が必要です
  - 手動での承認操作が必要になります
- パフォーマンス
  - プロンプト実行完了時にUI要素の監視が重くなる現象が発生します
  - 対策として monitor-process-and-directory.scpt が並列実行されます
  - プロンプト完了は `.done` ファイルの出力で検知します

## 構文のカスタマイズ

`sentence_structure/` に好きな構文を入れるとランダムでプロンプトに採用されます

## 制限事項

- macOS Sonoma以降でのみ動作確認しています
- Claude Desktop Appの更新により動作しなくなる可能性があります
- GitLab Runner以外からの実行は想定していません

## License

MIT
