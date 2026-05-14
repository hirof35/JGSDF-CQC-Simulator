JGSDF CQC Simulator (自衛隊格闘技シミュレーター)
自衛隊の近接格闘（CQC）をテーマにした、コマンド選択型Webシミュレーターです。
Rust (iced) での構築に失敗した経験を糧に、TypeScript + Vue 3 で再構築された「不屈の精神」の結晶です。

⚔️ システム概要
プレイヤーは自衛官として、ナイフを所持した敵との近接格闘に挑みます。
敵の予備動作を読み、適切なアクションを選択して制圧を目指してください。

アクション
刺突 (Thrust): 基本攻撃。敵が待機状態（Idle）の時に有効。気力を10%回復。

防衛 (Defense): 敵が攻撃準備（Preparing）に入った瞬間にのみ使用可能。敵を捌いて大きな隙を作り、反撃ダメージを与えます。

【必殺・制圧突き】: 気力が100%に達した時のみ発動可能。一撃で状況を終了（勝利）させます。

ステータス
HP: 0になるとミッション失敗。

出血 (Bleeding): 敵の攻撃を受けるとスタックし、ターン毎にスリップダメージが発生します。

気力 (Stamina): 攻撃や防御に成功することで蓄積されます。

🛠️ 技術スタック
Frontend: Vue 3 (Composition API / <script setup>)

Language: TypeScript

Build Tool: Vite

Styling: Tailwind CSS

Theme: Dark (Stone-900 / Army-style)

🚀 開発環境のセットアップ
Bash
# 依存関係のインストール
npm install

# 開発サーバーの起動 (localhost:5173 または 5174)
npm run dev

# ビルド (本番用)
npm run build
📝 開発の経緯
当初は Rust の GUI ライブラリ iced を用いて開発を試みましたが、 iced 0.14 の厳格な型システムとライフタイムの壁により、11個のエラーを抱えたまま「状況終了」を余儀なくされました。
しかし、そのロジックを TypeScript に移植することで、より柔軟かつ視覚的に優れたシミュレーターとして完成させることに成功しました。

注意事項
tsconfig.json のパースエラーには十分注意してください。カンマ一つが勝敗を分けます。

このシミュレーターはあくまで娯楽用であり、実際の格闘技術を教示するものではありません。
<img width="1913" height="998" alt="スクリーンショット 2026-05-14 161604" src="https://github.com/user-attachments/assets/76f83fb2-285b-45f2-9650-48928de8d081" />
