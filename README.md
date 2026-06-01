# transformer-text-gen-service
Transformer decoderのService化



## memo
- Root: 「どのURLにアクセスしたときに、どの処理を実行するか」を決めるもの
- シリアライズ／デシリアライズ: データの形式を変換する処理
- シリアライズ（serialize）：プログラム内のオブジェクト（Pythonのdictやクラスインスタンスなど）を、ネットワークで送れる形式（JSONなど）に変換すること
- デシリアライズ（deserialize）：ネットワークから受け取ったデータ（JSONなど）を、プログラム内のオブジェクト（Pythonのdictやクラスインスタンスなど）に変換すること
- FastAPIでは、pydanticの BaseModel を使うことで、このシリアライズ／デシリアライズを自動でやってくれます