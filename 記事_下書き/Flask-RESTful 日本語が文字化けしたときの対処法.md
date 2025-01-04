# Flask-RESTful 日本語が文字化けしたときの対処法

## 現象

- flask_restful.abortを使用してレスポンスを作成すると下記のような文字化けが発生した
{
    "message": "\u5f15\u304d\u6570name\u304c\u3042\u308a\u307e\u305b\u3093\u3002",
    "code": "400_3"
}

## 原因

FlaskはJSONレスポンスを生成する際にASCIIモードでエンコードするため非ASCII文字である日本語もASCIIモードで出力され文字化けする

## 対処法

- 下記設定を追加する

```python
flask_api.app.config['RESTFUL_JSON'] = {
    'ensure_ascii': False
}
```

```python
import flask
from flask_restful import Api

app = flask.Flask(__name__)
# Flaskの日本語文字化け対応
app.json.ensure_ascii = False

flask_api = Api(app)
# これを追加する!!!!!!!!!!!!!!!!!!!!!!!!!!!
flask_api.app.config['RESTFUL_JSON'] = {
    'ensure_ascii': False
}

if __name__ == "__main__":
    app.run(debug=False, host="0.0.0.0", port=8000)

# >>>>>>
# {
#     "message": "引き数nameがありません。",
#     "code": "400_3"
# }
```

## 参考URL

- My flask app doesn't return non-ascii character properly
  - <https://stackoverflow.com/questions/55903600/my-flask-app-doesnt-return-non-ascii-character-properly>
- 【Python × Flask】REST APIでjsonifyのレスポンスが文字化けする時の対応方法
  - <https://scrawledtechblog.com/python-flask-rest-api-jsonify-mojibake/>
- flask のjsonifyで日本語がutf-8で出力されてしまう
  - <https://note.com/ozzybot8/n/n94c9b2fceb7e>

## 環境

- Python 3.10.2
- Flask
  - Version: 2.3.2
- Flask-RESTful
  - Version: 0.3.10
