# ローカル Jupyter Notebook による Micro Speech 学習手順

この文書では、TensorFlow Lite Micro の Micro Speech 学習用 Notebook を、ローカル PC の Jupyter Notebook で実行する手順を説明します。

対象の Notebook は次のファイルです。

```text
tensorflow/lite/micro/examples/micro_speech/train/train_micro_speech_model.ipynb
```

## 1. 事前に知っておくこと

この学習用 Notebook は古い TensorFlow のサンプルコードを使用しています。元の README には、次の環境が記載されています。

```text
Python:     3.7
TensorFlow: 1.5
```

現在の Python や TensorFlow と完全には互換性がない可能性があります。特に、Python 3.12 へ TensorFlow 1.x をインストールすることは困難です。

ローカルの Python 3.12 を使用する場合は、TensorFlow 2.x で Notebook を動かせるか確認します。TensorFlow 1.x の API 互換エラーが出る場合は、次のいずれかを使用してください。

- Google Colaboratory
- Python 3.7 などの別環境
- Conda、pyenv、Docker で作った互換環境

システムの Python を置き換えず、学習専用の仮想環境を使用してください。

## 2. リポジトリの場所

この文書の例では、TFLM リポジトリを次の場所に置いているものとします。

```text
/mnt/hdd1/l0000900720/work_hdd/GitHub/tflite-micro-workspace/tflite-micro
```

環境によってパスが異なる場合は、自分の TFLM のルートディレクトリへ読み替えてください。

まずリポジトリのルートへ移動します。

```bash
cd /mnt/hdd1/l0000900720/work_hdd/GitHub/tflite-micro-workspace/tflite-micro
```

## 3. Python のバージョンを確認する

```bash
python3 --version
```

Python 3.12.3 の場合、Jupyter Notebook 自体は実行できます。ただし、Notebook が古い TensorFlow API を使用している場合は、セルの実行時に別の互換性エラーが出る可能性があります。

使用可能な Python を確認します。

```bash
python3.7 --version
conda --version
docker --version
```

`python3.7`、Conda、Docker のいずれかが利用できる場合は、古い学習環境を分離して作れます。利用できない場合は、Python 3.12 の仮想環境で TensorFlow 2.x を試すか、Google Colab を使用します。

## 4. Python 3.12 用の仮想環境を作る

Python 3.12 を使用する場合は、TFLM リポジトリのルートで次を実行します。

```bash
cd /mnt/hdd1/l0000900720/work_hdd/GitHub/tflite-micro-workspace/tflite-micro

python3 -m venv .venv-micro-speech-312
source .venv-micro-speech-312/bin/activate
```

仮想環境が有効になっていることを確認します。

```bash
python --version
which python
```

次のように、仮想環境内の Python が表示されれば正しい状態です。

```text
Python 3.12.3
.../tflite-micro/.venv-micro-speech-312/bin/python
```

`/usr/bin/python3` が表示される場合は、仮想環境が有効になっていません。

## 5. Jupyter と必要なパッケージをインストールする

まず pip を更新します。

```bash
python -m pip install --upgrade pip setuptools wheel
```

Jupyter と基本的なデータ処理パッケージをインストールします。

```bash
python -m pip install \
  notebook \
  ipykernel \
  numpy \
  matplotlib \
  scipy
```

Notebook が使用する TensorFlow をインストールします。

```bash
python -m pip install tensorflow
```

インストールできたことを確認します。

```bash
python -c "import tensorflow as tf; print(tf.__version__)"
```

TensorBoard のセルを使用するため、TensorBoard も確認します。

```bash
python -m pip install tensorboard
python -c "import tensorboard; print(tensorboard.__version__)"
```

## 6. Jupyter 関連パッケージの互換性

Jupyter 起動時に、次のようなエラーが発生することがあります。

```text
AttributeError: 'FileFindHandler' object has no attribute 'allowed_symlink_directory'
```

この場合は、Jupyter Server、Notebook、Tornado の組み合わせに問題がある可能性があります。仮想環境内で次のバージョンへそろえます。

```bash
python -m pip install --upgrade --force-reinstall \
  "notebook==7.4.7" \
  "jupyter_server==2.16.0" \
  "tornado==6.4.2"
```

その後、Jupyter を再起動します。

## 7. Jupyter カーネルを登録する

仮想環境を Jupyter のカーネルとして登録します。

```bash
python -m ipykernel install --user \
  --name micro-speech-312 \
  --display-name "Python (micro-speech, 3.12)"
```

Notebook の画面では、次の名前で表示されます。

```text
Python (micro-speech, 3.12)
```

## 8. Notebook を起動する

Notebook があるディレクトリへ移動します。

```bash
cd /mnt/hdd1/l0000900720/work_hdd/GitHub/tflite-micro-workspace/tflite-micro/tensorflow/lite/micro/examples/micro_speech/train
```

次のコマンドで起動します。

```bash
jupyter notebook train_micro_speech_model.ipynb
```

`jupyter` コマンドが別の環境を使用する可能性がある場合は、次の形式を使用します。

```bash
python -m notebook train_micro_speech_model.ipynb
```

ブラウザが自動で開かない場合は、ターミナルに表示された URL をブラウザで開きます。

```text
http://localhost:8888/tree?token=...
```

Jupyter を実行しているターミナルは、Notebook の作業中は閉じないでください。終了するときは、ターミナルで `Ctrl-C` を押します。

## 9. 正しいカーネルを選択する

Notebook を開いたら、メニューから次を選択します。

```text
Kernel → Change Kernel → Python (micro-speech, 3.12)
```

画面上部に表示されるカーネル名から選択する場合もあります。

カーネルを選択した後、最初に次のセルを実行してください。

```python
import sys

print(sys.executable)
print(sys.version)
```

次のように仮想環境内の Python が表示されることを確認します。

```text
.../.venv-micro-speech-312/bin/python
```

`/usr/bin/python3` が表示される場合は、システム Python を使用しています。カーネルを変更するか、仮想環境から再登録してください。

## 10. Notebook の実行順序

Notebook は基本的に上から順番に実行します。最初から一括実行するより、最初はセルごとに実行すると問題を発見しやすくなります。

おおまかな処理は次のとおりです。

1. Python パッケージをインポートする
2. 学習パラメータを設定する
3. TensorFlow リポジトリを取得する
4. データセットとログの保存先を設定する
5. TensorBoard を起動する
6. Speech Commands Dataset をダウンロードする
7. 音声を前処理する
8. モデルを学習する
9. 学習結果を評価する
10. モデルを量子化する
11. `.tflite` や C++ モデルを出力する

## 11. TensorFlow リポジトリの取得

Notebook の初期セルには、次のような処理があります。

```python
!git clone -q --depth 1 https://github.com/tensorflow/tensorflow
```

これは学習スクリプトを取得する処理です。成功すると、Notebook の起動ディレクトリから見て、次のパスが作られます。

```text
train/tensorflow/tensorflow/examples/speech_commands/train.py
```

すでに `tensorflow/` ディレクトリがある状態で同じセルを再実行すると、`destination path already exists` などのエラーになる場合があります。その場合は、既存のディレクトリを再利用するか、必要に応じて削除してから再実行します。

## 12. データセットのダウンロード

データセットの URL は次のとおりです。

```text
https://storage.googleapis.com/download.tensorflow.org/data/speech_commands_v0.02.tar.gz
```

Notebook のデータセット保存先は、通常次の設定です。

```python
DATASET_DIR = "dataset/"
```

学習セルは、次のスクリプトを実行します。

```python
!python tensorflow/tensorflow/examples/speech_commands/train.py \
  --data_dir={DATASET_DIR} \
  --wanted_words={WANTED_WORDS} \
  --silence_percentage={SILENT_PERCENTAGE} \
  --unknown_percentage={UNKNOWN_PERCENTAGE} \
  --preprocess={PREPROCESS} \
  --window_stride={WINDOW_STRIDE} \
  --model_architecture={MODEL_ARCHITECTURE} \
  --how_many_training_steps={TRAINING_STEPS} \
  --learning_rate={LEARNING_RATE} \
  --train_dir={TRAIN_DIR} \
  --summaries_dir={LOGS_DIR} \
  --verbosity={VERBOSITY} \
  --eval_step_interval={EVAL_STEP_INTERVAL} \
  --save_step_interval={SAVE_STEP_INTERVAL}
```

このセルを初めて実行すると、`train.py` がデータセットをダウンロードし、`dataset/` に展開してから学習を開始します。

Notebook を `train` ディレクトリから起動した場合、保存先は通常次の場所です。

```text
tensorflow/lite/micro/examples/micro_speech/train/dataset/
```

保存先を確認するには、Notebook で次を実行します。

```python
import os

print(os.getcwd())
print(os.path.abspath(DATASET_DIR))
```

ダウンロード後のデータを確認します。

```python
print(os.listdir(DATASET_DIR)[:10])
```

`yes`、`no`、`up`、`down` などのディレクトリが表示されれば、展開が完了しています。

## 13. データセットを毎回削除しない

Notebook の初期セルには、次のような削除処理があります。

```python
!rm -rf {DATASET_DIR} {LOGS_DIR} {TRAIN_DIR} {MODELS_DIR}
```

このセルを実行すると、データセットも削除されます。その後、学習セルを実行すると、データセットが再ダウンロードされます。

毎回のダウンロードを避けたい場合は、この削除セルを不用意に実行しないでください。途中から再開する場合も、既存の `dataset/` を削除しないようにします。

## 14. TensorBoard を使用する

学習中の損失や精度を確認するには、TensorBoard を使用します。

まず、仮想環境に TensorBoard があることを確認します。

```bash
source /mnt/hdd1/l0000900720/work_hdd/GitHub/tflite-micro-workspace/tflite-micro/.venv-micro-speech-312/bin/activate
python -m pip install tensorboard
```

Notebook では、ログディレクトリを指定して実行します。

```python
%load_ext tensorboard
%tensorboard --logdir {LOGS_DIR}
```

`ModuleNotFoundError: No module named 'tensorboard'` が出る場合は、次を Notebook のセルで実行する方法もあります。

```python
%pip install tensorboard
```

インストール後は、次を実行します。

```text
Kernel → Restart Kernel
```

その後、TensorBoard のセルを再実行してください。

TensorBoard のログ保存先を確認するには、次を実行します。

```python
print(LOGS_DIR)
```

## 15. 学習中に確認する項目

学習セルでは、損失と精度が表示されます。主に次を確認します。

- 学習用データの精度が上がっているか
- 検証用データの精度が上がっているか
- 学習用と検証用の精度に大きな差がないか
- 損失が発散していないか

学習データでは高精度なのに、検証データの精度が低い場合は過学習の可能性があります。

対策として、次を検討します。

- データの種類を増やす
- 複数の話者を含める
- 背景ノイズを追加する
- 学習ステップ数を減らす
- データ拡張を行う
- 学習率やモデル構造を調整する

## 16. 学習完了後に生成されるファイル

Notebook の設定やバージョンによって異なりますが、次のようなファイルが生成されます。

| ファイル | 用途 |
| --- | --- |
| `model.pb` | TensorFlow の Frozen GraphDef |
| `model.tflite` | TensorFlow Lite / TFLM 用モデル |
| `model.cc` | マイコンへ組み込む C/C++ 配列 |

出力されたファイルを検索するには、ターミナルで次を実行します。

```bash
find . -type f \( -name "*.pb" -o -name "*.tflite" -o -name "*.cc" \)
```

サイズを確認します。

```bash
find . -type f \( -name "*.tflite" -o -name "*.cc" \) -printf "%s bytes %p\\n"
```

Micro Speech では、モデルサイズ20 kB未満が目標です。ただし、モデルのファイルサイズだけでなく、Tensor Arena、音声バッファ、プログラム本体の RAM 使用量も確認してください。

## 17. TFLite モデルを C++ 配列へ変換する

学習した `model.tflite` を TFLM に組み込む場合は、`generate_cc_arrays` を使用します。

TFLM のルートディレクトリへ移動します。

```bash
cd /mnt/hdd1/l0000900720/work_hdd/GitHub/tflite-micro-workspace/tflite-micro
```

ツールをビルドします。

```bash
bazel build tensorflow/lite/micro/tools:generate_cc_arrays
```

モデルを変換します。

```bash
bazel-bin/tensorflow/lite/micro/tools/generate_cc_arrays \
  /tmp/custom_model.cc \
  /path/to/model.tflite
```

形式は次のとおりです。

```text
generate_cc_arrays [出力ファイル] [入力ファイル]
```

生成されたファイルを確認します。

```bash
head -n 20 /tmp/custom_model.cc
```

配列名やサイズ変数は、生成されたファイルの内容を確認してから C++ コードで使用してください。

## 18. TFLM でモデルを検証する

独自モデルを TFLM に組み込んだ後は、まず PC 上のテストを実行します。

```bash
bazel run tensorflow/lite/micro/examples/micro_speech:micro_speech_test
```

独自モデルへ置き換える場合は、次の項目も一致させます。

- モデル配列の参照先
- カテゴリ名
- カテゴリの順番
- 出力カテゴリ数
- テストの期待値
- 入力形状
- 入力と出力の量子化パラメータ
- Tensor Arena のサイズ

## 19. よくあるエラー

### `ModuleNotFoundError: No module named 'tensorflow'`

Notebook が別のカーネルを使用しています。次を実行して Python の場所を確認します。

```python
import sys
print(sys.executable)
```

`.venv-micro-speech-312/bin/python` でなければ、カーネルを `Python (micro-speech, 3.12)` に変更します。

### `ModuleNotFoundError: No module named 'tensorboard'`

TensorBoard が Notebook のカーネル環境にインストールされていません。Notebook のセルで次を実行します。

```python
%pip install tensorboard
```

その後、カーネルを再起動してから次を再実行します。

```python
%load_ext tensorboard
%tensorboard --logdir {LOGS_DIR}
```

### `AttributeError: 'FileFindHandler' object has no attribute 'allowed_symlink_directory'`

Jupyter Server、Notebook、Tornado の互換性問題の可能性があります。仮想環境で次を実行します。

```bash
python -m pip install --upgrade --force-reinstall \
  "notebook==7.4.7" \
  "jupyter_server==2.16.0" \
  "tornado==6.4.2"
```

### `FileNotFoundError` やデータセットが見つからない

Notebook の起動ディレクトリを確認します。

```python
import os
print(os.getcwd())
```

次の `train` ディレクトリから起動するのが安全です。

```text
tensorflow/lite/micro/examples/micro_speech/train
```

### `git clone` が既存ディレクトリで失敗する

`tensorflow/` ディレクトリがすでに存在する可能性があります。既存のディレクトリを再利用するか、内容が不要な場合だけ削除してから再実行します。

### TensorFlow の API エラー

次のようなエラーが出る場合があります。

```text
AttributeError: module 'tensorflow' has no attribute 'contrib'
ModuleNotFoundError: No module named 'tensorflow.examples'
```

これは、Notebook が TensorFlow 1.x の API を使用しているのに、TensorFlow 2.x を実行していることが原因の可能性があります。Python 3.12 の問題だけではなく、TensorFlow の世代差による互換性問題です。

この場合は、Colab または Python 3.7 などの互換環境を使用するのが現実的です。

## 20. 最小実行手順

Python 3.12 の仮想環境で試す場合の最小手順は次のとおりです。

```bash
cd /mnt/hdd1/l0000900720/work_hdd/GitHub/tflite-micro-workspace/tflite-micro

python3 -m venv .venv-micro-speech-312
source .venv-micro-speech-312/bin/activate

python -m pip install --upgrade pip setuptools wheel
python -m pip install notebook ipykernel numpy matplotlib scipy tensorflow tensorboard

python -m ipykernel install --user \
  --name micro-speech-312 \
  --display-name "Python (micro-speech, 3.12)"

cd tensorflow/lite/micro/examples/micro_speech/train
python -m notebook train_micro_speech_model.ipynb
```

Notebook を開いたら、カーネルを `Python (micro-speech, 3.12)` に変更し、次のセルで Python の場所を確認します。

```python
import sys
print(sys.executable)
```

## 21. 終了方法

Notebook の作業が終わったら、Jupyter を起動したターミナルで次を実行します。

```text
Ctrl-C
```

終了確認が表示されたら `y` を入力します。

仮想環境から抜ける場合は、次を実行します。

```bash
deactivate
```

## まとめ

ローカル Jupyter Notebook での学習は、次の順番で進めます。

1. Python のバージョンと利用可能な互換環境を確認する
2. 学習専用の仮想環境を作成する
3. Jupyter、TensorFlow、TensorBoard をインストールする
4. Jupyter カーネルを登録する
5. `train_micro_speech_model.ipynb` を `train` ディレクトリから起動する
6. 正しいカーネルを選択する
7. `sys.executable` で Python 環境を確認する
8. Notebook を上から順番に実行する
9. データセット、ログ、モデルの保存先を確認する
10. 学習結果を TensorBoard と精度で確認する
11. 必要に応じて `.tflite` を C++ 配列へ変換する
12. TFLM のテストで推論を確認する

Python 3.12 では Jupyter 自体は使用できますが、古い TensorFlow 学習コードとの互換性が最大の注意点です。互換性エラーが続く場合は、Colab または古い Python 環境へ切り替えてください。
