# transformer-text-gen-service
Transformer decoderのService化


## 進捗
- [x] setup (docker/Dockerfile, docker/requirements.txt, docker-compose.yml)
- [x] FastAPI Server動作確認
    - [x] main.py (最小Sample)
    - [x] Docker composeで環境を起動
    - [x] http://localhost:8000 で"Hello World"が返ってくる
    - [x] http://localhost:8000/generate で固定のResponseが返ってくる
- [x] FastAPIにTransformerモデルを連携し、文章生成APIを完成させる
    - [x] 推論用のコードをコピー (モデル定義（TransformerDecoder など）, トークナイザ（AutoTokenizer）, モデル読み込み（from_pretrained または load_state_dict）, 生成関数（generate メソッド）)
    - [x] FastAPIの /generate エンドポイントから、前回のTransformerモデルを呼び出して文章生成を行う
    - [x] 生成結果をJSONで返す
    - [x] curl で動作確認し、「文章生成Web API」として完成させる
        - `
        $ curl -X POST "http://localhost:8000/generate" \
        -H "Content-Type: application/json" \
        -d '{"prompt": "私は", "max_length": 50}'
        `
        - `
        {"prompt":"私は","generated_text":"私はこの“男は別れる”とは? なぜ“女\"いい”をあらわれたことも、女友達と感じる。女が心行かれたりばり、仕事にくことから誘われていたら、なかなか私より友","temperature":1.0,"top_k":50}
        `
- [x] Gradio UIの実装
    - [x] app/gradio.py でGradio UIを作成
    - [x] Docker composeにServiceを追加
    - [x] 動作確認 (`docker compose up --build` で再起動, ブラウザで `http://localhost:7860` にアクセスしGradio UIを確認, プロンプトを入力して生成結果を確認)
- [ ] Dockerイメージのビルドとレジストリへのpush


## memo

#### FastAPI
- Root: 「どのURLにアクセスしたときに、どの処理を実行するか」を決めるもの
- シリアライズ／デシリアライズ: データの形式を変換する処理
- シリアライズ（serialize）：プログラム内のオブジェクト（Pythonのdictやクラスインスタンスなど）を、ネットワークで送れる形式（JSONなど）に変換すること
- デシリアライズ（deserialize）：ネットワークから受け取ったデータ（JSONなど）を、プログラム内のオブジェクト（Pythonのdictやクラスインスタンスなど）に変換すること
- FastAPIでは、pydanticの BaseModel を使うことで、このシリアライズ／デシリアライズを自動でやってくれます
- 
```bash
curl -X POST "http://localhost:8000/generate" \
  -H "Content-Type: application/json" \
  -d '{"prompt":"こんにちは"}'
```
- `curl`: HTTPリクエストを送るコマンドラインツール
- `-X POST`: HTTPメソッドをPOSTに指定
- `"http://localhost:8000/generate"`：送信先のURL
- `-H "Content-Type: application/json"`：リクエストのヘッダーを指定（JSON形式）, 「送るデータがJSON形式ですよ」
- `-d '{"prompt":"こんにちは"}'`：リクエストボディ（送るデータ）を指定, FastAPIはこれを GenerateRequest の prompt フィールドとして受け取ります
    - `-d '{"prompt":"こんにちは", "temperature": 0.5}'`とかもOK

#### Gradio
- `demo = gr.Interface()`: 「どんな入力を受け取って、どんな関数を実行し、どんな出力を表示するか」 を定義
- `fn=gradio_generate`: 実行する関数
- `inputs=[gr.Textbox(label="プロンプト", placeholder="文章の続きを入力してください"),...]`: 入力UI（テキストボックスやスライダーなど）
- `outputs=gr.Textbox(label="生成結果")`: 出力UI（テキストボックスなど）
- `title="Transformer文章生成モデル"`: ページタイトル
- `description="プロンプトを入力すると、文章の続きを生成します。"`: ページの説明文
- `demo.launch(server_name="0.0.0.0", server_port=7860, share=False)`: Gradioサーバの起動
    - server_name="0.0.0.0"：すべてのIPアドレスからアクセス可能
    - server_port=7860：ポート番号（http://localhost:7860）
    - share=False：公開URLを作成しない（ローカルでのみアクセス可能）

## Error
- `api-1  | ImportError: cannot import name 'JapaneseTokenizer' from 'data.tokenizer' (unknown location)`
    - importできない問題
    - app.data.tokenizerで行けるが，前Projectや生成コマンドで動いていた状態から変えたくない
    - `    environment:  - PYTHONPATH=/code/app`で成功
        - コンテナ内で `data/` や `utils/` をPythonパスから見えるようにする必要があります
        - docker-compose.yml の api サービスに環境変数を追加
        - これで、コンテナ内のPythonが /code と /code/app をパスに含めてくれます
- ` ImportError: cannot import name 'HfFolder' from 'huggingface_hub' (/usr/local/lib/python3.10/site-packages/huggingface_hub/__init__.py)`
    - Version互換性問題
    - HfFolder クラスは huggingface_hub v1.0.x で削除された
    - Gradioが古いバージョンの huggingface_hub に依存しているため、HfFolder を参照しようとしてエラーに
    - requirements試行錯誤
        - `gradio==3.50.2`, `huggingface_hub>=0.19.3,<2.0`: NG(gradio古いError)
        - `gradio`, `huggingface_hub>=0.19.3,<2.0`: NG(HfFolderがないError)
        - `gradio`, `huggingface_hub<1.0.0`: NG(Transformers==5.7.0に反する)
        - `gradio`, `huggingface_hub<1.0.0`, `transformers==4.30.2`: NG(gradioとfastAPIとの互換性Error)
        - `gradio==3.50.2`, `huggingface_hub<1.0.0`, `transformers==4.30.2`: NG(gradioとfastAPIとの互換性Error)
        - Gradio 3.50.2は2023年ごろRelease, Pydantic 1系を前提に動いているらしい
        - `huggingface_hub==0.19.4`, `pydantic==1.10.13`: OK(起動時Error出ず)
        - 