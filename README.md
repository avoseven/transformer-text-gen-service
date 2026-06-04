# transformer-text-gen-service
Transformer decoderのService化


## 進捗
- [x] setup (docker/Dockerfile, docker/requirements.txt, docker-compose.yml)
- [x] FastAPI Server動作確認
    - [x] main.py (最小Sample)
    - [x] Docker composeで環境を起動
    - [x] http://localhost:8000 で"Hello World"が返ってくる
    - [x] http://localhost:8000/generate で固定のResponseが返ってくる
- [ ] FastAPIにTransformerモデルを連携し、文章生成APIを完成させる
    - [ ] 推論用のコードをコピー (モデル定義（TransformerDecoder など）, トークナイザ（AutoTokenizer）, モデル読み込み（from_pretrained または load_state_dict）, 生成関数（generate メソッド）)
    - [ ] FastAPIの /generate エンドポイントから、前回のTransformerモデルを呼び出して文章生成を行う
    - [ ] 生成結果をJSONで返す
    - [ ] curl で動作確認し、「文章生成Web API」として完成させる


## memo
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