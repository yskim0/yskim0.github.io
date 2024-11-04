---
title: 딥러닝 모델을 Flask 서버에 올리기 (+ MongoDB, GCP 가상 인스턴스 사용하기)
categories: [Side Project, Capstone]
tags: 
 - Capstone
 - Visualization
 - Recommendation
---

이화여대 2021-1학기 캡스톤디자인프로젝트B 그로쓰7팀 Ewha Visualization Recommendation Program(ERP) 기술 튜토리얼에 관한 글입니다.


본 포스팅은 시각화 추천 프로그램(Visualization Recommendation Program)을 개발하는 과정 중, 
이전 학기에 학습한 Seq2Seq 모델을 Flask를 활용하여 서버에 올리고 MongoDB를 활용하여 데이터베이스를 관리하고, 이 외에도 프로젝트에 필요한 api를 구현하는 과정에 대하여 작성되었습니다.

---
> Contents 

1. About Our Project
2. Flask
3. GCP
4. Conclusion

---

# About Our Project...(and goal)

안녕하세요. 이 블로그 자체가 굉장히 오랜만이네요.
5월 말에 졸업 프로젝트 관련 글을 올린 적이 있는데, 그 글의 연장선 상에 있는 글이라 생각하시면 될 것 같습니다.


이전 학기에 학습한 Data2Vis 모델을 가지고, 실제 프로젝트에 활용하기 위하여 백엔드 개발 및 DB 구성에 대하여 작성해보겠습니다.
본론에 앞서, 저희 프로젝트 목표에 대해 다시 상기시켜드리고자 합니다. 


저희 팀은 `Data Visualization` 연구에서도 자동으로 plot(chart)을 그려줄 수 있는 **(딥러닝을 이용한) Data Visualization Recommendation**을 프로젝트 주제로 하였습니다. 자세히는 저희가 제공하는 샘플 데이터 또는 사용자가 원하는 데이터셋을 upload 했을 때, 저희의 Visualization Recommendation 모델이 데이터셋을 **해석**하여 여러 개의 plot을 그려주어 사용자에게 추천해주는 것입니다.


**즉 요약하자면,**
- 데이터셋(e.g., csv, tsv, json)을 `input`으로 하여
- 저희의 딥러닝 모델이 해당 데이터셋에 대한 적절한 `chart(visualization) recommendation`을 k개 리스트업하여 보여주고
- 사용자가 chart를 선택
    - 이를 활용하여 데이터 요약 보고서를 작성

의 기능을 구현한 웹 어플리케이션을 구현하는 것이 목표입니다.


본 포스팅에서는 딥러닝 모델이 준비 완료된 상태에서 이 모델을 Flask를 활용하여 어떻게 서버에 디폴로이하는지에 관하여 작성되었습니다.

---

# Flask

Flask는 파이썬 기반의 웹 프레임워크입니다. 플라스크는 간편하고 경량화된 장점을 가지고 있어, 딥러닝 모델을 사용하는 웹 프로젝트에 많이 쓰입니다.


이전에 환경 충돌을 피하기 위해 anaconda를 통해 가상 환경을 생성합니다.

## 가상환경 생성 및 필요 라이브러리 설치

```
conda create -n data2vis python=3.6
conda activate data2vis
conda install flask
```
플라스크 이외에 딥러닝 모델을 구동시키기 위해 필요한 Requirements또한 설치합니다. 이는 data2vis github의 레포지터리를 클론해서 `requirements.txt` 파일이 필요합니. 여러 가지 라이브러리를 설치하기 위해 생성한 가상환경의 경로로 pip를 활용하여 설치합니다.

```
$ conda env list
$ /Users/{name}/opt/anaconda3/envs/data2vis/bin/pip install -r requirements.txt
```
conda env list를 통해 가상환경의 경로를 알아내고, 이 경로를 복사-붙여넣기 하여 `/bin/pip` 폴더에 requirements로 패키지를 설치합니다.

이제 필요한 라이브러리는 다 설치했습니다.


## 필요 API 

본 팀 프로젝트를 개발하기 위해 필요한 주요 기능은 다음과 같습니다.
1. 파일 업로드 -> DB에 저장
2. 업로드된 파일을 모델에 Inference 하기
3. 사용자가 저장하고자 한 plot들을 DB에 저장
4. 유저가 업로드한 파일 히스토리를 불러오기


본 팀은 따로 프론트엔드 개발자가 있으므로, 백엔드 단에서는 구현한 기능 실험만을 위한 화면 구성을 하겠습니다.


### 파일 업로드

```py
@app.route('/')
def index():
    return '''
    <form method=post action="/upload" enctype=multipart/form-data>
      <input type=file name=file>
      <input type=submit value=Upload>
    </form>
    '''
```
![image](https://user-images.githubusercontent.com/48315997/141688419-dd044131-f25c-4911-8a40-8acbd4dae1d0.png)

- 일단은 로컬에서 돌리므로, 위 코드를 통해 `http://127.0.0.1:5000/`에 들어가면 위와 같이 파일 업로드를 할 수 있는 폼이 보여질 것입니다.

```py
@app.route('/upload', methods=['POST'])
def upload_file():
    file = request.files['file'] # file 타입의 name
    user_id = request.args['user_id'] # front에서 전달받은 user_id

    if file and allowed_file(file.filename):
        generated_plots = [] # 빈 Array // inference 과정에서 생성할 예정이므로.
        save_arr = []
        mongo.save_file(file.filename, file)

        file_type = file.filename.rsplit('.', 1)[1].lower() # json, csv, tsv
        data_url = 'http://127.0.0.1:5000/file/'+file.filename

        if file_type == 'json':
            data = pd.read_json(data_url)
        elif file_type == 'tsv':
            data = pd.read_csv(data_url, sep='\t')
        elif file_type == 'csv':
            data = pd.read_csv(data_url)
            
        cols = list(data.columns)
        _id = mongo.db.data.insert({'userId': user_id, 'filename' : file.filename, 'generated_plots' : generated_plots, 'cols':cols, 'save_arr':save_arr})
        return jsonify({'message' : 'Success(file uploaded and updated DB)', 'userId' : user_id ,'fileId' : str(ObjectId(_id))}), 200
    return jsonify({'message' : 'Failed(no file OR not allowed file type)'}), 400
```
- 파일을 Flask로 넘겨주는 것이므로 POST method를 사용함.
- `mongo.save_file`을 통해 DB에 파일을 저장함.
- 이 때 전달되는 request에는 파일과 DB에 저장할 때 필요한 user id를 받는다고 가정함.
- 업로드할 수 있는 파일의 포맷은 `json, tsv, csv`로 한정함.
- mongoDB의 `data` collection에 해당 파일과 userId, filename 등이 저장됨.
- response를 통해 해당 api가 잘 진행됐는지 확인할 수 있게 함.


<img width="363" alt="스크린샷 2021-11-15 오전 1 17 22" src="https://user-images.githubusercontent.com/48315997/141689133-f1a21888-1bc3-49bb-a336-252e354e988c.png">
> data collection에 저장된 것을 확인할 수 있음

<img width="362" alt="스크린샷 2021-11-15 오전 1 18 43" src="https://user-images.githubusercontent.com/48315997/141689189-48090fa0-75e6-4f73-b296-cc33221329a7.png">
> file 자체가 저장된 곳임



### 업로드된 파일을 모델에 Inference 하기 

이전 학기에 학습한 Data2Vis 모델을 불러와서, 사용자가 업로드한 데이터를 인풋으로 받아 Plot code를 아웃풋으로 주는 api를 구현합니다.


아래 코드는 Data2Vis 공식 레포지터리에서 사용된 inference 코드를 가져와서 필요한 부분만 남기고 수정했습니다.

```py
# Function to run inference.
def run_inference():
    # tf.reset_default_graph()
    with graph.as_default():
        saver = tf.train.Saver()
        checkpoint_path = loaded_checkpoint_path
        if not checkpoint_path:
            checkpoint_path = tf.train.latest_checkpoint(model_dir_input)

        def session_init_op(_scaffold, sess): 
            ########### restore model ckpt
            saver.restore(sess, checkpoint_path)
            tf.logging.info("Restored model from %s", checkpoint_path)

        scaffold = tf.train.Scaffold(init_fn=session_init_op)
        session_creator = tf.train.ChiefSessionCreator(scaffold=scaffold)
        with tf.train.MonitoredSession(
                session_creator=session_creator, hooks=hooks) as sess:
            sess.run([])
        # print(" ****** decoded string ", decoded_string)
        return decoded_string

destination_file = "test.txt"
# Setup Input parameters
input_pipeline_dict = {
    'class': 'ParallelTextInputPipeline',
    'params': {
        'source_delimiter': '',
        'target_delimiter': '',
        'source_files': [destination_file]
    }
}

model_dir_input = './vizmodel/'

input_task_list = [{'class': 'DecodeText', 'params': {'delimiter': ''}}]

dump_attention_task = {
    'class': 'DumpAttention',
    'params': {
        'dump_plots': False,
        'output_dir': "attention_plot"
    }
}

beam_width = 15 # default setting

#  {'class': 'DumpBeams', 'params': {'file': ['out.npz']}}]
model_params = "{'inference.beam_search.beam_width': 5}"
batch_size = 32
loaded_checkpoint_path = None

session_creator = None
hooks = []
decoded_string = ""


fl_tasks = _maybe_load_yaml(str(input_task_list))
fl_input_pipeline = _maybe_load_yaml(str(input_pipeline_dict))

# Load saved training options
train_options = training_utils.TrainOptions.load(model_dir_input)

# Create the model
model_cls = locate(train_options.model_class) or \
    getattr(models, train_options.model_class)
model_params = train_options.model_params
if (beam_width != 1):
    model_params["inference.beam_search.beam_width"] = beam_width
model_params = _deep_merge_dict(model_params, _maybe_load_yaml(model_params))
model = model_cls(params=model_params, mode=tf.contrib.learn.ModeKeys.INFER)

print("========model params ==========\n", model_params)

def _handle_attention(attention_scores):
    print(">>> Saved attention scores")


def _save_prediction_to_dict(output_string):
    global decoded_string
    decoded_string = output_string


# Load inference tasks
for tdict in fl_tasks:
    if not "params" in tdict:
        tdict["params"] = {}
    task_cls = locate(str(tdict["class"])) or getattr(tasks, str(
        tdict["class"]))
    if (str(tdict["class"]) == "DecodeText"):
        task = task_cls(
            tdict["params"], callback_func=_save_prediction_to_dict)
    elif (str(tdict["class"]) == "DumpAttention"):
        task = task_cls(tdict["params"], callback_func=_handle_attention)

    hooks.append(task)

input_pipeline_infer = input_pipeline.make_input_pipeline_from_def(
    fl_input_pipeline,
    mode=tf.contrib.learn.ModeKeys.INFER,
    shuffle=False,
    num_epochs=1)

# Create the graph used for inference
predictions, _, _ = create_inference_graph(
    model=model, input_pipeline=input_pipeline_infer, batch_size=batch_size)

graph = tf.get_default_graph()
```

- `model_dir_input = './vizmodel/'` 이 변수에는 학습된 모델(`.pth`)의 경로를 적는다.
- `beam_width = 15` 나오게 되는 output(plot code)의 개수를 설정함.

위의 inference 함수를 사용하여 넘겨지는 file에 따라 플랏 코드를 생성함.

```py
@app.route('/inference/<file_id>')
def inference(file_id):
    file = mongo.db.data.find_one({"_id" : ObjectId(file_id)}) # db 찾음
    if file:
        try:
            file_name = file['filename']
            file_id = file['_id']
            print(file_name, file_id)

            data_url = 'http://127.0.0.1:5000/file/'+file_name
            """
            this data(r) -> convert to json all!             
            """
            json_data = convert_to_json(file_name, data_url)
            ################################
            source_data = json.loads(json_data)

            generated_plots = []

            f_names = data_utils.generate_field_types(source_data)

            fnorm_result = data_utils.forward_norm(source_data, destination_file,
                                                   f_names)

            run_inference()
            num = 0

            for row in decoded_string:
                decoded_post = data_utils.backward_norm(row, f_names)
                vega_spec = json.loads(decoded_post)
                vega_spec['data'] = {'values' : source_data}
                generated_plots.append(vega_spec)
                num+=1
            
            print(file_name, file_id)

            mongo.db.data.update(
                {'_id' : file_id},
                {'$set' : {'generated_plots' : generated_plots}}
                
            )
            print(file_name, file_id)

            return jsonify({'message': 'Success(generated recommended plots)', 'plots':generated_plots}), 200

        except:
            return jsonify({'message' : 'Fail(Inference failed)'}), 500
    return jsonify({'message':'Fail(no file detected)'}), 400
```
- 프론트엔드에 plot code를 전달해야 하므로 GET method를 사용함.
- file 자체를 전달하기 위해 file id를 알아야 하는데, 이를 위해 주소에 인자를 받을 수 있도록 함.
- 학습된 모델은 plot code를 생성할 때, json 데이터를 사용하므로 inference 전에 json 포맷의 파일이 아니라면 json 파일로 변경하여 모델에 넘겨줌.
- 해당 file id에 해당되는 데이터의 `generated_plots` 필드에 모델을 통해 생성된 코드 배열을 업데이트함.
- response를 통해 해당 api가 잘 진행됐는지 확인할 수 있게 함.

<img width="338" alt="스크린샷 2021-11-15 오전 1 19 51" src="https://user-images.githubusercontent.com/48315997/141689215-1fa984f3-30be-4187-907e-8b6c4ab19831.png">
> `generated_plots` 필드에 Vega-lite 그래머의 플랏 코드가 생성된 것을 확인할 수 있음.


### 사용자가 저장하고자 한 plot들을 DB에 저장
```py
@app.route('/save', methods=['POST'])
def save():
    user_id = request.args['user_id'] # front에서 전달받은 user_id
    file_id = request.args['file_id'] # front에서 전달받은 file_id
    plot_code = request.args['plot'] # front에서 전달받은 plot code

    data = mongo.db.data
    dist_data = data.find_one(
        {'_id' : file_id}
    )
    
    if dist_data:
        try:
            save_arr = dist_data['save_arr']
            save_arr.append(dist)
            data.update(
                {'_id' : file_id},
                {'$set' : {'save_arr' : save_arr}}
            )
            return jsonify({'message': 'Success(saved img url)', 'file_id' : file_id, 'save_arr':save_arr}), 200
        except:
            return jsonify({'message' : 'Fail(save failed)'}), 500
    return jsonify({'message':'Fail(no file detected)'}), 400
```
- 프론트엔드에서 사용자가 저장하고자 하는 plot을 전달받으므로 POST method를 사용함.
- DB에 업데이트 하기 위해 user_id, file_id, 저장하고 싶은 plot_code 를 받음.
- 앞에서 받은 file id를 토대로 데이터베이스에서 find함.
- 찾은 데이터의 `save_arr` 필드에 전달받은 plot code를 append하여 전달함.
- response를 통해 해당 api가 잘 진행됐는지 확인할 수 있게 함.


### 유저가 업로드한 파일 히스토리를 불러오기

```py
@app.route('/get_upload_history/<user_id>')
def get_upload_history(user_id):
    data = mongo.db.data
    history = []
    user_data_all = data.find({'userId':user_id})
    try :
        for x in user_data_all:
            file_name = x['filename']
            file_id = x['_id']
            time = x.get('_id').generation_time
            history.append([file_id, file_name, time])
        return jsonify({'message': 'data upload histroy loaded.', 'history':history}), 200
    except:
        return jsonify({'message': 'data upload histroy failed.'}), 400
```
- 프론트엔드에 유저가 업로드한 파일 히스토리를 전달해야 하므로 GET method를 사용함.
- 유저와 관련된 정보를 알아야 하는데, 이를 위해 주소에 user_id를 받을 수 있도록 함.
- 유저가 업로드한 파일명, 파일 아이디, 시간을 담아 배열로 전달함.
- response를 통해 해당 api가 잘 진행됐는지 확인할 수 있게 함.

### Flask 서버 실행
```py
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'
app = Flask(__name__)
cors = CORS(app, resources={r"/api/*": {"origins": "*"}})

app.config["MONGO_URI"] = MONGODB_URL
mongo = PyMongo(app)

...

if __name__ == '__main__':
    app.secret_key = SECRET_KEY
    app.run(debug=True, threaded=True)
```
위 코드 실행 시 flask 서버가 실행되게 됩니다. 로컬에서 테스트하는 것이므로 http://127.0.0.1:5000 입력 시 만든 웹 어플리케이션을 볼 수 있습니다.
<img width="691" alt="스크린샷 2021-11-15 오전 1 50 31" src="https://user-images.githubusercontent.com/48315997/141690292-418ab68c-35c3-4f8e-995f-e505b48edab4.png">
 

## 협업을 위한 Notion과 Postman
여기까지가 현재까지 본 팀의 프로젝트에서 필요한 백엔드 api입니다. 
원활한 협업을 위해 노션과 postman을 작성하였습니다.

<img width="1003" alt="스크린샷 2021-11-15 오전 1 32 39" src="https://user-images.githubusercontent.com/48315997/141689681-5753aa02-2def-4b7a-a5b4-0aede801af8d.png">

<img width="1288" alt="스크린샷 2021-11-15 오전 1 33 40" src="https://user-images.githubusercontent.com/48315997/141689711-1e1399d0-cc9f-4d28-b749-661afc2c3f39.png">



# GCP

## 가상 인스턴스 생성
![image](https://user-images.githubusercontent.com/48315997/141690017-ac1a4e4e-f476-4452-b338-08a92c596320.png)


딥러닝 모델을 사용해 inference하므로, GPU가 달린 가상 인스턴스를 생성합니다.
인스턴스는 [이 분의 블로그](https://jeinalog.tistory.com/8)를 따라 동일하게 생성하였습니다.
여기에 추가로 본 팀의 프로젝트는 nodejs 서버도 사용하므로 방화벽 규칙에 8080 port도 허용하도록 설정하였습니다. 
- GPU : P80 1개
- Ubuntu 16.04 LTS
- SSD 100GB
- Memory 16GB

## gcloud를 통해 터미널로 가상 인스턴스에 접속
![image](https://user-images.githubusercontent.com/48315997/141689848-407f6ffc-d1c2-4139-9b98-1a3e0578452d.png)
- 가상인스턴스의 gcloud 명령어를 복사하여 터미널에 붙여넣기 하면, 로그인 과정을 거쳐 가상 인스턴스에 접속할 수 있습니다.
```bash
yeonsookim@YEONSOOui-MacBookPro  ~  gcloud beta compute ssh --zone "us-central1-a" "erp"  --project "erp1886"
```

![image](https://user-images.githubusercontent.com/48315997/141690115-d92061c9-e7bd-4495-8373-1eb837cbd923.png)
- 개발한 flask 코드가 잘 돌아가는지 확인함. 가상 인스턴스의 외부 ip주소를 주소창에 치고 flask 포트 번호를 뒤에 붙여 확인함.

프론트엔드와 연결하여 파일 업로드 및 inference 과정을 테스트해본 결과 DB에 정상 업로드되는 것을 확인할 수 있었습니다.
<img width="451" alt="스크린샷 2021-11-15 오전 1 52 03" src="https://user-images.githubusercontent.com/48315997/141690336-f3a6f97e-f792-412e-bb70-17bfdcc79228.png">


## 서버 백그라운드로 RUN
```bash
# 백그라운드 실행
(erp-server) yeonsookim@erp:~/ERP-Server$ nohup python server.py &
```
`nohup &` 을 통해 서버에서도 백그라운드로 계속 돌아가게 함.


---

# Conclusion



현재 저희는 추천 플랏 생성 프로세스까지는 프론트엔드-백엔드 연결이 완료되었습니다. 이제 대시보드 상에서의 오류를 해결하고 리팩토링할 일만 남아있습니다. 

<img width="1088" alt="스크린샷 2021-11-15 오전 1 57 22" src="https://user-images.githubusercontent.com/48315997/141690504-83b8d991-7b24-4947-a24f-6a7b893cde79.png">

