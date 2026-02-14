---
marp: true
theme: uncover
class: invert
style: |
  section {
    background: #01a9f4;
    color: white;
    font-family: 'Helvetica Neue', 'Hiragino Sans', 'Noto Sans JP', sans-serif;
  }
  h1 {
    color: white;
    font-weight: 800;
    font-size: 2.2em;
  }
  h2 {
    color: white;
    font-weight: 700;
    font-size: 1.6em;
  }
  h3 {
    color: rgba(255,255,255,0.85);
    font-weight: 600;
  }
  li {
    font-size: 0.85em;
    line-height: 1.5;
  }
  code {
    background: rgba(0,0,0,0.2);
    color: #FFD54F;
    padding: 2px 8px;
    border-radius: 4px;
  }
  table {
    font-size: 0.65em;
    color: white;
  }
  th {
    background: rgba(0,0,0,0.3);
  }
  td {
    background: rgba(0,0,0,0.1);
  }
  .accent {
    color: #FFD54F;
  }
  blockquote {
    border-left: 4px solid #FFD54F;
    background: rgba(0,0,0,0.15);
    padding: 0.5em 1em;
    font-style: normal;
  }
  footer {
    color: rgba(255,255,255,0.6);
    font-size: 0.6em;
  }
paginate: true
footer: '#jawsug_yokohama'
---

<!-- _paginate: false -->
<!-- _footer: '' -->

# AIで爆速DR設計
## 東京→大阪 Migration

JAWS-UG Yokohama #99

2026-02-14

<span class="accent">#jawsugyokohama</span> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;**@ijin** **@yoshidashingo**

---

# 今日のお話

## AI-DLCでDR/BCP設計を爆速化

- 東京 → 大阪リージョン Migration
- AI-DLC ワークフローの実践
- 調査・要件定義・設計の自動化

---

# プロジェクト概要

**リテール企業のDR/BCP**
- 東京 (ap-northeast-1) → 大阪 (ap-northeast-3)
- 別AWSアカウント・別Organization
- 脅威モデル: ランサムウェア
- RTO 48h / RPO 24h

---

# 移行対象の規模

|  | リソース |
|---|---|
| EC2 | **64台** |
| RDS | **6台 / 33 TB** (SQL Server + PostgreSQL) |
| FSx | **20 TB** |
| S3 | **7 buckets** |
| ALB/NLB | **9 LBs** / 47 listeners |
| Step Functions | **1,000+** |
| Cognito | User Pool + Identity Pool |
| Subnet / SG | **44+ / 250+** |

---

<!-- _paginate: false -->

# これ、手作業で設計すると...

---

# 従来のアプローチ

- インフラ調査: **1-2週間**
- 要件定義: **1-2週間**
- アーキテクチャ設計: **2-3週間**
- 詳細設計 (10ユニット): **3-4週間**
- レビュー・修正: **1-2週間**

## 合計: **<span style="color:#FFFFFF; text-shadow: 2px 2px 4px rgba(0,0,0,0.3);">2-3ヶ月</span>**

---

<!-- _paginate: false -->

# Enter: AI-DLC

---

# AI-DLC とは

**AI-Driven Development Lifecycle**
AWS発のAIネイティブ開発方法論

- **AIが実行し、人間が監視する**
  AIがワークフローを駆動、人間が検証・承認
- **3フェーズ**: Inception → Construction → Operations
- SDLCの再定義

> AWS Blog: ai-driven-development-life-cycle

---

# 今回のDR設計でやったこと

AI-DLCのワークフローに沿って実施:

1. **Reverse Engineering** — 既存環境の自動分析
2. **Requirements Analysis** — SME Q&A → 要件書生成
3. **Architecture Design** — 包括的設計書
4. **Functional Design** — ユニット別詳細設計
5. **Code Generation** — Terraform HCL (進行中)

---

# Reverse Engineering

**既存環境の自動分析 (70+ファイル)**

- 膨大な既存ドキュメントを **Agent並列解析**
- `aws` CLI × 9カテゴリで環境を網羅的に収集
- ドキュメント ⇔ CLI結果の **Cross Reference**
  乖離を即座に発見

---

# Requirements Analysis

**22分野の技術Q&Aを自動生成**

- ネットワーク / セキュリティ / バックアップ
- 10個のブロッカー質問を特定
- 回答 → 要件定義書を自動生成

---

# Architecture Design

**包括的DR設計書を自動生成**

- ネットワーク設計 (VPC, Peering, FW)
- セキュリティ設計 (KMS, IAM, SG, WAF)
- データ保護設計 (Backup CopyAction, S3 CRR)
- コンピュート / DB / ストレージ設計
- DR発動リカバリフロー (Phase 0-5)

---

# Functional Design

**10ユニットの詳細設計**

| 平時 (6ユニット) | DR発動 (4ユニット) |
|---|---|
| network | database-recovery |
| security | storage-recovery |
| data-protection | compute-recovery |
| load-balancer | service-activation |
| app-services | |
| rds-config | |

---

<!-- _paginate: false -->

# AIが発見した重大事実

---

# 現環境の課題

AIが監査データから自動検出:

- Backup CopyAction: **未設定** (36プラン全て)
- S3 CRR: **未設定** (全バケット)
- Vault Lock: **未設定** (ランサムウェア脆弱)
- RDS z1d / r5b: **大阪で使用不可**

> 人手の調査では見落としがちな
> クロスリージョンの制約を自動発見

---

# Before / After

| フェーズ | 従来 | AI-DLC |
|---|---|---|
| 調査・分析 | 1-2週間 | 時間単位 |
| 要件定義 | 1-2週間 | 時間単位 |
| アーキテクチャ設計 | 2-3週間 | 時間単位 |
| 詳細設計 (10ユニット) | 3-4週間 | 時間単位 |
| **合計** | **2-3ヶ月** | **数日間** |

> ボトルネックはAIではなく **人間のレイテンシー** (レビュー・回答待ち)

---

# aidlc-cc-plugin

**AI-DLCワークフローをClaude Code pluginとして作っていた**

`github.com/ijin/aidlc-cc-plugin`

- `/aidlc` でワークフロー開始
- Inception → Construction を自動駆動
- セッション再開・監査証跡も自動
- とても便利だった！

---

# アーキテクチャ

```
 東京 (ap-northeast-1)          大阪 (ap-northeast-3)
 Account A                      DR Account (別Org)
┌───────────────────┐         ┌───────────────────┐
│ VPC-1             │         │ DR-VPC-1          │
│ 基幹系 (14 EC2)   │ Peering │ ALB/NLB (平時)    │
│ RDS x4 (20TB)     │◄───────►│ KMS/IAM/SG (平時) │
└───────────────────┘         └───────────────────┘
┌───────────────────┐  Daily  ┌───────────────────┐
│ VPC-2             │ Backup  │ DR-VPC-2          │
│ 店舗系 (39 EC2)   │  Copy   │ ALB x3 (平時)     │
│ RDS x2 (13TB)     │ ======> │ Vault Lock (平時) │
│ FSx (20TB)        │ S3 CRR  │                   │
└───────────────────┘  AMI    └───────────────────┘
```

---

# まとめ

- **AI-DLCで設計フェーズを劇的に短縮** — 2-3ヶ月 → 数日間
- **インフラ設計にも有効** — アプリ開発だけじゃない
- **ワークフローの再現性** — aidlc-cc-plugin で誰でも実行可能
- **リアルタイムコラボレーションが鍵** — 人間のレイテンシーがボトルネック。リアルタイム協働で真価発揮

---

<!-- _paginate: false -->
<!-- _footer: '' -->

# Thank You!

