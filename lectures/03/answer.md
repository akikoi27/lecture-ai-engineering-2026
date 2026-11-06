# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：
　　第1回(app.askへの質問)の回答
      $ uv run python -m app.ask "AIE2026の講座の宿題にある「3レベル」とは何ですか？"
        [model] gemini-3.5-flash-lite
        [質問] AIE2026の講座の宿題にある「3レベル」とは何ですか？

        AIE2026（AIエンジニア育成講座などの2026年目標のプログラムを指していると思われます）の宿題における「3レベル」という言葉は、一般的に**受講生のスキルや理解度、あるいは課題の難易度に合わせて用意された3段階のステージ（レベル1〜3）**を指しています。

        具体的な内容は講座のカリキュントによって異なりますが、多くのプログラミングやAIの講座では、以下のような基準で「3レベル」が設定されています。
        (以下略)

    第3回(app.agent)の回答
      $ uv run python -m app.agent "AIE2026の講座の宿題にある「3レベル」とは何ですか？"
        [model] gemini-3.5-flash-lite
        AIE2026の講座における「3レベル」とは、宿題の難易度や取り組み方の段階を示す構成のことです。

        具体的には以下の3つのレベルに分かれています。

        1. **基礎**：講義内容の確認を中心としたレベル
        2. **実装**：自分でコードを書くレベル
        3. **発展**：さらに調査や応用的な試行を行うレベル

        3つのうち**どれか1つ**に取り組めば、その回の宿題は完了となります（原則として採点は行われません）。ご自身の理解度や使える時間に合わせて、お好きなレベルを選んで取り組んでみてくださいね。

        関連する回：第1回

- `--trace` で見えたツール呼び出し（ツール名と引数）：
     --- trace ---
        [tool call]   search_glossary({'query': 'LoRA'})
        [tool call]   search_glossary({'query': 'QLoRA'})
        [tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
        [tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
        [usage] RunUsage(cost=Decimal('0.0007888'), details={'text_prompt_tokens': 1196}, input_text_tokens=1196, input_tokens=1196, output_tokens=172, requests=2, tool_calls=2)


- 1つの質問での LLM 呼び出し回数（`requests`）：
     2回

- 追加したツールと、docstring に書いた説明：

## 宿題

- 取り組んだレベル：基礎
- やったこと：下記のようにagent.pyのINSTRUCTIONを変更し、同じ質問をした時の挙動を変えることを試した。
　　　　　　　具体的には、用語の意味を答えるのではなく、その用語が出てくる回の要約を表示して、
　　　　　　　その講義回を勉強するように誘導する挙動にすることを目指した。

＜Before＞
     INSTRUCTIONS = """\
     あなたはAIエンジニアリング講座（AIE2026）の学習支援エージェントです。
     - 用語や講座に固有のことを答える前に、必ず search_glossary で用語集を確認する。
     - 回の内容を聞かれたら get_lecture_summary を使う。
     - 用語集に見つからないことは、推測で断定せず「用語集にはない」と伝える。
     - 回答の最後に、関係する回（第N回）を示す。
     - 宿題の答えそのものは書かず、考え方のヒントで導く。
     """

＜After＞
     INSTRUCTIONS = """\
     あなたはAIエンジニアリング講座（AIE2026）の学習支援エージェントです。
     - 用語や講座に固有のことを聞かれたら、その用語には答えず、必ず search_glossary で用語集を確認し、その用語が登場する回のサマリを返す。
     - 回のサマリの取得には get_lecture_summary を使う。
     - 用語集に見つからないことは、推測で断定せず「用語集にはない」と伝える。
     - 回答の最後に、関係する回（第N回）を示す。
     - 宿題の答えそのものは書かず、考え方のヒントで導く。
     """

- 結果：

＜Before＞
     $ uv run python -m app.agent --trace "LoRAとQLoRAの違いは？"
     [model] gemini-3.5-flash-lite
     LoRAとQLoRAの違いは以下の通りです。

     * **LoRA**: 事前学習済みの重みを固定したまま、重みの更新分を小さな低ランク行列（アダプタ）の積で表し、その部分だけを学習する手法です。学習パラメータや保存ファイルを大幅に削減できます。
     * **QLoRA**: ベースモデルの重みを4ビットなどに量子化（省メモリ化）したうえで、LoRAによる学習を行う手法です。LoRAの手法に量子化を組み合わせることで、より大きなモデルを小さなGPUメモリでも扱えるようにします。

     関係する回：第2回

     --- trace ---
     [tool call]   search_glossary({'query': 'LoRA'})
     [tool call]   search_glossary({'query': 'QLoRA'})
     [tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
     [tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
     [usage] RunUsage(cost=Decimal('0.0007888'), details={'text_prompt_tokens': 1196}, input_text_tokens=1196, input_tokens=1196, output_tokens=172, requests=2, tool_calls=2)

＜After＞
     $ uv run python -m app.agent --trace "LoRAとQLoRAの違いは？"
     [model] gemini-3.5-flash-lite
     LoRAとQLoRAの主な違いは、ベースモデルの重みをどのように扱うかにあります。

     - **LoRA**
       事前学習済みの重みは固定したまま、追加の低ランク行列（アダプタ）だけを学習することで、学習パラメータ数や保存容量を大幅に削減する手法です。

     - **QLoRA**
       LoRAの仕組みに加え、ベースモデルの重みを4ビットなどに量子化（省メモリ化）して保持したうえで、LoRAによる学習を行います。これにより、より大きなモデルを小さなGPUメモリでも扱えるようにする手法です。

     関係する回：第2回

     --- trace ---
     [tool call]   search_glossary({'query': 'LoRA'})
     [tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
     [tool call]   get_lecture_summary({'lecture': 2})
     [tool return] get_lecture_summary -> # 第2回 ファインチューニングとLoRA

     ## この回のねらい

     - 事前学習とファインチューニングの違い、ファインチューニングが向く場面と向かない場面を理解する
     - LoRAの考え方を理解し、Google Colab上で小さなモデルを…
     [usage] RunUsage(cost=Decimal('0.0010766'), details={'text_prompt_tokens': 2172}, input_text_tokens=2172, input_tokens=2172, output_tokens=170, requests=3, tool_calls=2)

- 気づき・疑問：
   ・INSTRUCTIONを少し変えただけでは、「用語に答えない」という、本来の意図とは違う動きに完全にはならなかった
   ・ただし講義回を判定してその回のサマリを表示するというのは、実現できた。
   　全く同じ質問であっても、異なるツールを使用して回答を作った様子がtrace上も確認できた。
