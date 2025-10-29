**前提**
- リージョン：東日本
- OS：Linux（RHEL または SUSE）
- Caché バージョン：2016.1 以上（InterSystems 製品ライセンス費用は含まない）
- 稼働時間：730 時間/月（24×7）
- 外部トラフィック：中程度（Application Gateway 相当の処理量を想定）
- ストレージ：Premium Managed Disks（データ・ジャーナル別）
- 監視：Log Analytics（100 GB/月）
- Backup：日次スナップショット、保持 30 日（バックアップストレージ 2 TB想定）
- 為替：1 USD = ¥150

- <img width="735" height="738" alt="image" src="https://github.com/user-attachments/assets/499e355c-0ad6-455e-a8cf-997e15fae538" />


**構成**
- External Load Balancer（Azure External LB） ← Public IP
  - Web Servers（VM） x 3
- Web Servers: Linux (Apache 2.4, CSP Gateway) ×3
- ECP Clients: VM ×3（TrakCare EPS / Caché）
- Database Mirrors: VM ×2（Primary / Backup, Caché 2016.1+）
- Arbiter: VM ×1（Mirror Arbiter）
- Internal Load Balancer（ILB）: DB ミラー / 内部 TCP 用ヘルスプローブ

台数サマリ（常時稼働）
- Web VM: 3 台
- ECP VM: 3 台
- DB ミラー: 2 台
- Arbiter: 1 台
- 合計 VM 台数 = 9 台

VM 種類（推奨サンプル）
- Web: Standard_DS3_v2（4 vCPU / 14 GB）
- ECP: Standard_DS4_v2（8 vCPU / 28 GB）
- DB Primary / Backup: Standard_DS13_v2（8 vCPU / 56 GB）または GS 系（高 IOPS が必要な場合）
- Arbiter: Standard_DS2_v2（2 vCPU / 7 GB）

単価前提（730 時間/月）
- Standard_DS3_v2 : 約 300 USD / 月
- Standard_DS4_v2 : 約 600 USD / 月
- Standard_DS13_v2: 約 1,200 USD / 月
- Standard_DS2_v2 : 約 120 USD / 月
- Premium Managed Disk:
  - P30 (1 TiB) : 約 180 USD / 月
  - P20 (512 GiB): 約 100 USD / 月
- External / Internal Load Balancer, Public IP 等合算: 約 150 USD / 月
- Application Gateway 相当（L7 ヘルス / SSL 終端等）: 目安 600 USD / 月（中程度トラフィック）
- Log Analytics (100 GB/月): 300 USD / 月
- Backup Storage (2 TB RA-GRS): 400 USD / 月
- Automation / Runbooks: 150 USD / 月
- 運用サポート・基本: 250 USD / 月

項目別内訳（USD/月 → JPY/月（×150））
- Compute (VM)
  - Web (Standard_DS3_v2) | 3 台 × 300 = 900 USD → ¥135,000
  - ECP (Standard_DS4_v2) | 3 台 × 600 = 1,800 USD → ¥270,000
  - DB-Primary (Standard_DS13_v2) | 1 × 1,200 = 1,200 USD → ¥180,000
  - DB-Backup (Standard_DS13_v2) | 1 × 1,200 = 1,200 USD → ¥180,000
  - Arbiter (Standard_DS2_v2) | 1 × 120 = 120 USD → ¥18,000

  - Compute 小計 = 900 + 1,800 + 1,200 + 1,200 + 120 = 5,220 USD
  - Compute 小計 (JPY) = 5,220 × 150 = ¥783,000

- Storage
  - データディスク (P30) ×2（Primary/Backup 各1） = 2 × 180 = 360 USD → ¥54,000
  - ジャーナルディスク (P20) ×2（Primary/Backup 各1） = 2 × 100 = 200 USD → ¥30,000
  - OS ディスク等合算 = 9 × 20 = 180 USD → ¥27,000（VMごとのOSディスク換算）
  - Snapshot / Backup Storage (2 TB RA-GRS) = 400 USD → ¥60,000

  - Storage 小計 = 360 + 200 + 180 + 400 = 1,140 USD
  - Storage 小計 (JPY) = 1,140 × 150 = ¥171,000

- Networking / Load Balancing
  - Application Gateway 相当（L7） = 600 USD → ¥90,000
  - External/Internal LB, Public IP, NAT 等合算 = 150 USD → ¥22,500

  - Network 小計 = 750 USD → ¥112,500

- Monitoring / Backup / Automation / Support
  - Log Analytics (100 GB/月) = 300 USD → ¥45,000
  - Automation / Runbooks = 150 USD → ¥22,500
  - 運用サポート(基本) = 250 USD → ¥37,500

  - Ops 小計 = 700 USD → ¥105,000

合計（USD/月 と JPY/月）
- USD 合計 = Compute 5,220 + Storage 1,140 + Network 750 + Ops 700 = 7,810 USD / 月
- JPY 合計 = 7,810 × 150 = ¥1,171,500 / 月

見積（単一リージョン構成、オンデマンド）
- 月額合計（目安）：約 7,810 USD → 約 ¥1,171,500
- 想定誤差：±20〜30%（概ね ¥820,000 ～ ¥1,520,000 のレンジ）

補足・注意事項
- オンデマンド単価の概算。予約インスタンス（1年/3年）適用や RI/Savings Plan を使えば VM コストは 30〜60% 削減可能。
- DB の IOPS 要件により DB VM やディスク数は増減します。大量トランザクションや大容量 DB（数 TB 以上）がある場合は Premium Disk 数や VM を上げる必要があり、費用は大きく増えます。
- Application Gateway の実際の課金はトラフィック次第で増減します。WAF を ON にする場合はさらに + 約 ¥50k～¥150k/月 程度+
- InterSystems ライセンス費（Caché/TrakCare 等）は含まれていません。
- 可用性強化（複数 AZ 配置、DR リージョン同期/非同期構成）を行うとコストは上乗せになります。

