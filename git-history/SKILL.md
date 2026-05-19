---
name: git-history
description: git履歴の整理(rebase/squash/fixup/cherry-pick/reword)を支援する。コミット粒度の見直し、main取り込み戦略(rebase vs merge)、間違いコミットの修復、conflict解消方針。git履歴整理・rebase・squash・cherry-pickの相談で使う。
---

# Git History

git 履歴の整理・修復を安全に行うための手順を提供する。

## 目的別の選択

| やりたいこと | コマンド |
| :--- | :--- |
| 直前コミットのメッセージを変える | `git commit --amend` |
| 直前コミットに追加変更を入れる | `git add ... && git commit --amend --no-edit` |
| 複数コミットを 1 つに潰す | `git rebase -i <base>` で `squash` / `fixup` |
| コミット順序を入れ替える | `git rebase -i <base>` で並び替え |
| 過去のコミットを編集 | `git rebase -i <base>` で `edit` |
| 別ブランチの 1 コミットを取り込む | `git cherry-pick <sha>` |
| main の最新を取り込む(履歴を直線化) | `git rebase origin/main` |
| main の最新を取り込む(マージコミットで残す) | `git merge origin/main` |
| 間違いコミットを取り消す(履歴は残す) | `git revert <sha>` |
| 間違いコミットを履歴から消す | `git rebase -i` で削除(force-push 必要) |
| 直前の操作を取り消したい | `git reflog` → `git reset --hard HEAD@{n}` |

## 安全な作業フロー

### 1. 作業前のバックアップ

```bash
# 現ブランチを別名で保存(失敗しても戻せる)
git branch backup/feature-x
```

### 2. interactive rebase

```bash
# main から分岐後のコミットを編集
git rebase -i origin/main

# エディタで pick → squash/fixup/reword/edit/drop を選ぶ
# pick   = そのまま
# reword = メッセージだけ変更
# edit   = コミットを止めて編集
# squash = 前のコミットに統合(メッセージも統合)
# fixup  = 前のコミットに統合(このコミットのメッセージは捨てる)
# drop   = 削除
```

### 3. force-push(自分のブランチのみ)

```bash
# --force-with-lease は他人の push を上書きしない
git push --force-with-lease origin feature-x
```

## main 取り込み戦略

| 戦略 | 履歴 | 向き |
| :--- | :--- | :--- |
| `git rebase origin/main` | 直線化、main の上に乗せ直し | feature ブランチ、レビュー前 |
| `git merge origin/main` | マージコミットが残る | 公開・共有ブランチ、conflict が複雑なとき |

長寿命ブランチや複数人で触るブランチは merge を選ぶ(rebase は他人の commit を書き換えるので)。

## conflict 解消の方針

1. **理解してから直す。** 両側の意図を読んでから手を入れる
2. **`git rerere` を有効に。** 同じ conflict を繰り返す場合に解消結果を記憶
3. **「動くか」を確認。** マージしただけで満足せず、コンパイル + テストで検証
4. **大規模 conflict は中断。** `git rebase --abort` / `git merge --abort` で戻ってから対策

## やってはいけないこと

- **共有ブランチで rebase。** main/develop など他人が触るブランチを rebase + force-push する
- **`git push --force` のまま。** `--force-with-lease` を使う
- **`git reset --hard` を確認せず実行。** uncommit な変更を消し飛ばす
- **`git clean -fd` の盲打ち。** 必要なファイルまで消えうる。`-n` でドライラン後に実行

## 出力フォーマット

```markdown
## git 履歴整理手順: <目的>

### 現状
- 現ブランチ: `feature/x`
- 整理対象: 直近 5 コミット
- 共有状況: まだ自分しか触っていない

### バックアップ
\`\`\`bash
git branch backup/feature-x-before-rebase
\`\`\`

### 手順
1. `git rebase -i origin/main`
2. エディタで以下のように編集:
   \`\`\`
   pick  a1b2c3 feat: ...
   fixup d4e5f6 fix typo
   squash 7g8h9i wip
   reword j0k1l2 add tests
   \`\`\`
3. ...

### 検証
- [ ] `git log --oneline -10` で履歴が想定通り
- [ ] テストが通る
- [ ] backup ブランチが残っている

### push
\`\`\`bash
git push --force-with-lease origin feature/x
\`\`\`
```

## 原則

- **歴史改変は自分のブランチだけ。** 他人が触る可能性のあるブランチでは rebase しない。
- **バックアップを取ってから始める。** 失敗しても reflog があるが、ブランチで残すほうが分かりやすい。
- **小さく試す。** 一気に 20 コミットを整理するより、5 コミット × 4 回に分ける。
- **commit メッセージを直す機会と思え。** rebase 中は `reword` でメッセージを推敲できる絶好の機会。
- **conflict は機械的に解決しない。** 「両方残せばいいんでしょ」で消える挙動を見逃しがち。

## 関連スキル

- `commit-msg` — rebase 中に reword するとき、メッセージの作り方
- `pr-description` — 整理した履歴を反映した PR 説明
- `code-review` — 履歴整理後にコードの実態が変わっていないかセルフレビュー
