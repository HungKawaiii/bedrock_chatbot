# LLM主導のエージェント（ReAct）

## エージェント (ReAct) とは

エージェントは、大規模言語モデル（LLM）を中心的な計算エンジンとして使用する高度なAIシステムです。LLMの推論能力を、計画や道具の使用などの追加機能と組み合わせて、複雑なタスクを自律的に実行します。エージェントは、複雑な質問を分解し、ステップバイステップの解決策を生成し、外部のツールやAPIと対話して情報を収集したり、サブタスクを実行したりできます。

この例では、[ReAct (Reasoning + Acting)](https://www.promptingguide.ai/techniques/react) アプローチを使用してエージェントを実装しています。ReActにより、エージェントは推論とアクションを反復的なフィードバックループで組み合わせることで、複雑なタスクを解決できます。エージェントは、思考、行動、観察の3つの重要なステップを繰り返し実行します。LLMを使用して現在の状況を分析し、次のアクションを決定し、利用可能なツールやAPIを使用してアクションを実行し、観察された結果から学習します。この継続的なプロセスにより、エージェントは動的な環境に適応し、タスク解決の精度を向上させ、コンテキストを意識した解決策を提供できます。

## サンプルユースケース

ReAct を使用するエージェントは、さまざまなシナリオで正確で効率的な解決策を提供できます。

### テキストからSQL

ユーザーが「最後の四半期の総売上」を要求します。エージェントはこのリクエストを解釈し、SQLクエリに変換し、データベースに対して実行し、結果を提示します。

### 財務予測

財務アナリストは、次の四半期の収益予測を作成する必要があります。エージェントは関連するデータを収集し、財務モデルを使用して必要な計算を実行し、予測の正確性を確保しながら、詳細な予測レポートを生成します。

## Agent機能の使用方法

カスタムチャットボットのAgent機能を有効にするには、次の手順に従ってください：

1. カスタムボット画面のAgent セクションに移動します。

2. Agentセクションで、Agentが使用できる利用可能なツールのリストが表示されます。デフォルトでは、すべてのツールは無効になっています。

3. ツールを有効にするには、目的のツールの横にあるスイッチを簡単にオンにします。ツールが有効になると、Agentはそのツールにアクセスでき、ユーザーリクエストの処理に使用できます。

![](./imgs/agent_tools.png)

> [!重要]
> Agentセクションの任意のツールを有効にすると、自動的に["Knowledge"機能](https://aws.amazon.com/what-is/retrieval-augmented-generation/)もツールとして処理されることに注意することが重要です。これは、LLMが自律的にユーザーリクエストに応答するために「Knowledge」を使用するかどうかを決定し、利用可能なツールの1つとして考慮することを意味します。

4. デフォルトでは、「インターネット検索」ツールが提供されています。このツールにより、Agentはインターネットから情報を取得して、ユーザーの質問に回答できます。

![](./imgs/agent1.png)
![](./imgs/agent2.png)

このツールは[DuckDuckGo](https://duckduckgo.com/)に依存しており、速度制限があります。概念実証（PoC）またはデモ目的には適していますが、本番環境で使用する場合は、別の検索APIを使用することをお勧めします。

5. Agent機能を拡張するために、独自のカスタムツールを開発して追加できます。カスタムツールの作成と統合の詳細については、[独自のツールを開発する方法](#how-to-develop-your-own-tools)セクションを参照してください。

## 独自のツールを開発する方法

エージェント用の独自のカスタムツールを開発するには、以下のガイドラインに従ってください：

- `AgentTool`クラスを継承する新しいクラスを作成します。インターフェースはLangChainと互換性がありますが、この例の実装では独自の`AgentTool`クラスを提供しており、それを継承する必要があります（[ソース](../backend/app/agents/tools/agent_tool.py)）。

- [BMI計算ツール](../examples/agents/tools/bmi/bmi.py)の実装例を参照してください。この例は、ユーザー入力に基づいてBMI（ボディマス指数）を計算するツールの作成方法を示しています。

  - ツールで宣言された名前と説明は、LLMがユーザーの質問に答えるためにどのツールを使用するかを評価する際に使用されます。つまり、LLMを呼び出すプロンプトに組み込まれています。そのため、できるだけ正確に記述することをお勧めします。

- [オプション] カスタムツールを実装した後、テストスクリプト（[例](../examples/agents/tools/bmi/test_bmi.py)）を使用してその機能を検証することをお勧めします。このスクリプトは、ツールが期待通りに機能することを確認するのに役立ちます。

- カスタムツールの開発とテストが完了したら、実装ファイルを[backend/app/agents/tools/](../backend/app/agents/tools/)フォルダに移動します。次に、[backend/app/agents/utils.py](../backend/app/agents/utils.py)を開き、`get_available_tools`を編集して、ユーザーが開発したツールを選択できるようにします。

- [オプション] フロントエンド用の明確な名前と説明を追加します。このステップはオプションですが、このステップを行わない場合、ツールで宣言された名前と説明がLLM用に使用されますが、ユーザー向けではありません。そのため、ユーザーエクスペリエンスを向上させるために専用の説明を追加することをお勧めします。

  - i18nファイルを編集します。[en/index.ts](../frontend/src/i18n/en/index.ts)を開き、`agent.tools`に独自の`name`と`description`を追加します。
  - また、`xx/index.ts`も編集します。`xx`は目的の言語コードを表します。

- `npx cdk deploy`を実行して変更をデプロイします。これにより、カスタムツールがカスタムボット画面で利用可能になります。

## 貢献

**ツールライブラリへの貢献を歓迎しています！** 有用で適切に実装されたツールを開発した場合、issueやプルリクエストを送信してプロジェクトに貢献することを検討してください。