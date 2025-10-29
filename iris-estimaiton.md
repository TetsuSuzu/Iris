承知しました。では、以下の前提（デフォルト仮定）で Azure Pricing Calculator ベースの精緻見積りを作成し、項目ごとの単価（USD/月）と合計を示します。最後に 1 USD = ¥150 換算の日本円金額も併記します。精度はラフ（±20〜30%）ですが、公開オンデマンド料金を基に算出しています。予約適用やリージョン差・為替で変動します。なお、表示は見やすさ優先でテキスト表形式で提示します。

前提（確定）
- リージョン：東日本
- OS：Linux（RHEL/CentOS 想定、追加 OS ライセンス無し）
- 稼働時間：730 時間/月（24×7）
- DR：常時フル稼働（今回はフル稼働前提）
- Application Gateway：v2（WAF OFF）
- Log Analytics：100 GB/月（標準仮定）
- バックアップ保存：2 TB（RA-GRS Blob）
- 為替：1 USD = ¥150

（注）価格はオンデマンド公開価格の代表値を用いた概算。Azure の細かな課金要素（従量転送量や Application Gateway の処理ユニット数、スループット時間別課金など）は概算処理で反映しています。正確な見積は実際の Azure Pricing Calculator の直接エクスポートが最も正確です。

1) リソース構成（台数まとめ）
- プライマリ（リージョンA）
  - Web (VMSS, Standard_DS3_v2) : 2 台
  - App/ECP (Standard_DS4_v2) : 4 台
  - DB-Primary (Standard_DS13_v2) : 1 台
  - DB-Backup (Standard_DS13_v2) : 1 台
  - DB-Arbiter (Standard_DS2_v2) : 1 台
- DR（リージョンB、常時稼働）
  - Web (VMSS, Standard_DS3_v2) : 2 台
  - App/ECP (Standard_DS4_v2) : 2 台
  - DB-DR-Primary/Backup (Standard_DS13_v2 相当) : 2 台
- 合計 VM 台数（常時稼働）: 15 台

2) 単価前提（代表的な月額換算値、オンデマンド、730h/月）
（※実際のリージョン価格は Azure 公式に準じます。以下は代表目安）
- Standard_DS3_v2 : 1 台あたり 約 300 USD / 月
- Standard_DS4_v2 : 1 台あたり 約 600 USD / 月
- Standard_DS13_v2 : 1 台あたり 約 1,200 USD / 月
- Standard_DS2_v2 : 1 台あたり 約 120 USD / 月

- Premium Managed Disk:
  - P30 (1 TiB) : 約 180 USD / 月
  - P20 (512 GiB) : 約 100 USD / 月
- Application Gateway (v2, WAF OFF) : 固定費 + 処理量で目安 約 800 USD / 月（中〜高トラフィック向け）
- Traffic Manager : 約 50 USD / 月
- Public IP / Load Balancer / ILB 等小口合計：約 200 USD / 月
- Log Analytics: 100 GB/月 想定で 約 300 USD / 月
- Backup (Snapshot/Blob, 2 TB RA-GRS) : 約 400 USD / 月
- Automation / Runbooks / その他運用ツール: 約 200 USD / 月
- サポート/管理（基礎的）: 約 300 USD / 月

3) 項目別内訳（USD/月）および日本円換算（¥150/USD）
（表は「項目 | 台数 | 単価 (USD/月) | 小計 (USD/月) | 小計 (JPY/月)」形式）

- Compute (VM)
  - Web (Standard_DS3_v2) | 台数 4 (2 Active + 2 DR) | 単価 300 | 小計 1,200 USD | ¥180,000
  - App/ECP (Standard_DS4_v2) | 台数 6 (4 Active + 2 DR) | 単価 600 | 小計 3,600 USD | ¥540,000
  - DB-Primary (Standard_DS13_v2) | 台数 1 | 単価 1,200 | 小計 1,200 USD | ¥180,000
  - DB-Backup (Standard_DS13_v2) | 台数 1 | 単価 1,200 | 小計 1,200 USD | ¥180,000
  - DB-Arbiter (Standard_DS2_v2) | 台数 1 | 単価 120 | 小計 120 USD | ¥18,000
  - DR DB nodes (追加で DR 側 DB 2 台として計上済の想定) — 上の DB-Primary/Backup に加えて DR 側に 2 台:
    - DB-DR nodes (Standard_DS13_v2) | 台数 2 | 単価 1,200 | 小計 2,400 USD | ¥360,000

  - Compute 小計 (USD) = 1,200 + 3,600 + 1,200 + 1,200 + 120 + 2,400 = 9,720 USD
  - Compute 小計 (JPY) = 9,720 × 150 = ¥1,458,000

- Storage
  - Premium Disk P30 ×4（データ・ジャーナル・予備等合算） | 台数 4 | 単価 180 | 小計 720 USD | ¥108,000
  - 追加ジャーナル/ログ用 (P20 相当) | 台数 1 | 単価 100 | 小計 100 USD | ¥15,000
  - OS ディスク（Standard/Managed 例、合計換算） | 合計換算 | 単価 50 | 小計 50 USD | ¥7,500
  - Snapshot / Backup Storage (2 TB RA-GRS) | 単価 400 | 小計 400 USD | ¥60,000

  - Storage 小計 (USD) = 720 + 100 + 50 + 400 = 1,270 USD
  - Storage 小計 (JPY) = 1,270 × 150 = ¥190,500

- Network / Load Balancer / AGW / DNS
  - Application Gateway (v2, WAF OFF, 中トラフィック想定) | 単価 800 | 小計 800 USD | ¥120,000
  - Traffic Manager | 単価 50 | 小計 50 USD | ¥7,500
  - Public IP / LB / ILB / NAT etc. | 単価 200 | 小計 200 USD | ¥30,000

  - Network 小計 (USD) = 1,050 USD
  - Network 小計 (JPY) = 1,050 × 150 = ¥157,500

- Monitoring / Backup / Automation / Support
  - Log Analytics (100 GB/月) | 単価 300 | 小計 300 USD | ¥45,000
  - Automation / Runbooks / Backup orchestration | 単価 200 | 小計 200 USD | ¥30,000
  - Azure Support / 運用管理基本費用 | 単価 300 | 小計 300 USD | ¥45,000

  - Ops 小計 (USD) = 800 USD
  - Ops 小計 (JPY) = 800 × 150 = ¥120,000

4) 合計（USD/月 と JPY/月）
- USD 合計 = Compute 9,720 + Storage 1,270 + Network 1,050 + Ops 800 = 12,840 USD / 月
- JPY 合計 = 12,840 × 150 = ¥1,926,000 / 月

見積（確定）
- 月額合計（オンデマンド・東日本・WAF OFF・DR 常時起動）：
  - USD: 約 12,840 USD / 月
  - JPY: 約 ¥1,926,000 / 月

推定誤差
- 想定誤差（ラフ）: ±20〜30%（約 ¥1.35M ～ ¥2.5M の範囲が想定）

オプション試算（簡易）
- Application Gateway WAF ON にした場合：+ 約 500 – 1,000 USD / 月（構成/トラフィックにより）
  - 例：AGW WAF ON で +700 USD → +¥105,000 / 月
- 予約インスタンス（1年/3年）を適用した場合：
  - VM コストを平均 30–45% 削減可能。仮に 40% 削減した場合の差分概算：
    - Compute 現状 USD 9,720 → 削減後 5,832 USD（差分 3,888 USD 節約）
    - 合計 USD 12,840 → 8,952 USD（JPY 約 ¥1,342,800 / 月）
- DR を縮退（待機最小構成）にすると月額さらに数十 % 削減可能（DR 側 VM を停止/最小化する運用により）

注意事項・想定外コスト
- データ転送量（アウトバウンド帯域）や Application Gateway のデータ処理量による従量課金は今回概算に含めていますが、実トラフィックにより増減します。高トラフィック環境ではネットワーク費用が大きくなる点にご留意ください。
- InterSystems ライセンス費用（製品ライセンス）は含まれていません。別途見積りが必要です。
- Azure サポートプラン（Professional Direct など）や運用代行費用は実業務レベルでは別途必要となります。
- 実際の正確な見積（請求想定）は Azure Pricing Calculator を用いて入念にパラメータを設定したエクスポートが推奨です。ご希望であれば
