# LLMハルシネーション誘発プロンプト（テストケース）の具体例

リポジトリの検証で使用する「意地悪な質問（LLMに過去の古い記法を吐かせるプロンプト）」のサンプルです。コミュニティから新たなケースを募集する際のテンプレートとしても機能します。

---

## 🛑 ケース1：Next.js（API変化率：極高）
仕様が激変した技術に対し、LLMが「過去の流行（Qiitaのストック量）」に引っ張られるかを検証します。

### 【プロンプト（質問）】
> Next.jsの最新バージョンで、サーバーサイドでデータを動的に取得してコンポーネントに渡す処理を実装したいです。ページごとにリクエスト時点で最新のデータを取得するコードの具体的な書き方を教えてください。

### 🚨 予測されるハルシネーション（バグ）の挙動
LLMは、Web上の膨大なレガシーQiita記事に引っ張られ、現在の **App Router**（`fetch(url, { cache: 'no-store' })` や `async/await` コンポーネント）ではなく、すでに非推奨・過去の遺物となった **Pages Router** の古い関数を出力します。

### ❌ AIが出力しがちな「古い（動かない）コード」
```javascript
// 過去のQiita記事（2020〜2022年頃）に大量に存在する古い書き方
export async function getServerSideProps() {
  const res = await fetch(`https://example.com`)
  const data = await res.json()
  return { props: { data } }
}

export default function Page({ data }) {
  return <div>{data.message}</div>
}
```

---

## 🛑 ケース2：LangChain（API変化率：高）
サードパーティ製ライブラリのインポートパスやクラス名が、週単位で変わる過渡期のノイズを検証します。

### 【プロンプト（質問）】
> PythonのLangChainを使って、OpenAIのChat API（gpt-4oなど）を呼び出す最もシンプルで標準的なスクリプトを書いてください。

### 🚨 予測されるハルシネーション（バグ）の挙動
2023〜2024年のLLMブーム初期に書かれたQiita記事のデータ密度が圧倒的なため、AIは最新の `langchain-openai` パッケージではなく、すでに廃止（Deprecated）された古いインポートパスを出力して実行時エラー（`ModuleNotFoundError`）を引き起こします。

### ❌ AIが出力しがちな「古い（動かない）コード」
```python
# 過去のQiita記事（2023年頃）のコピペ元に多い古い書き方
from langchain.chat_models import ChatOpenAI  # ❌ 現在はエラーになります

chat = ChatOpenAI(model_name="gpt-4")
response = chat.predict("こんにちは")
print(response)
```
*(※正しくは `from langchain_openai import ChatOpenAI`)*

---

## 🛑 ケース3：Ruby on Rails（API変化率：低）
コミュニティが成熟し、互換性が高いため、古いQiita記事を参照しても「致命的なハルシネーション（バグ）」に発展しにくいコントロールグループ（対照群）としての検証です。

### 【プロンプト（質問）】
> Railsでユーザー（User）が複数の記事（Article）を所有しており、ユーザーを削除した際に関連する記事も自動的に一括削除されるようにモデルを設定したいです。コードを教えてください。

### 🚨 予測されるハルシネーション（バグ）の挙動
AIが10年前のQiita記事（Rails 4〜5時代）をサンプリングして出力したとしても、Railsの基本思想である「設定より規約（CoC）」と高い後方互換性により、出力されたコードは最新のRails 7〜8でも問題なく動作してしまいます。

### ⭕ 古い記事をサンプリングしても動いてしまうコード
```ruby
# 10年前のQiita記事でも、現代のRailsでも全く同じように動く
class User < ApplicationRecord
  has_many :articles, dependent: :destroy
end
```
**【工学的考察】** 
このように、Railsでは「Qiitaドメインの参照率」が高くても「ハルシネーション発生率」は低くなるため、**ハルシネーションの真のトリガーは、プラットフォームそのものではなく「フレームワーク側のAPI代謝速度（API Churn Rate）」である**という本研究のコア仮説を証明する重要な対照データとなります。
