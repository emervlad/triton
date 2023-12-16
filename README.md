## Nvidia Triton Server

В качестве модели был взят distilbert на ~88Mb параметров.
Для того, чтобы конвертировать модель в onnx, необходимо ввести команду:
``
python export_model.py
``
После этого в головной директории должен появиться файл `distilbert_embedder.onnx`
Копируем его в `model_repository/onnx-distilbert/1/model.onnx` и в `model_repository/vino-distilbert/1/model.onnx`
Для конвертации в TensorRT сначала поднимаем контейнер:
```angular2html
docker run -it --rm --gpus 1 -v {absolute_to_model_repository}:/models nvcr.io/nvidia/tensorrt:23.04-py3
```
Используем данную команду для конвертации в trt-fp32:
```angular2html
trtexec --onnx=/models/onnx-rubert/1/model.onnx --saveEngine=/models/onnx-distilbert/1/model.plain --minShap
es=INPUT_IDS:1x16,ATTENTION_MASK:1x16 --optShapes=INPUT_IDS:8x16,ATTENTION_MASK:8x16 --maxShapes=INPUT_IDS:8x16,ATTENTION_MASK:8x16 --profilingVerbosity=detailed
```
Для конвертации в trt-fp16 (trt-int8) добавляем к этой команде флаг `--fp16` (`--int8`).
Перемещаем соответствующие файлы под новыми именами в `model_repository/{model_name}/1/model.onnx`.
Выходим из контейнера и стартуем Triton Server:
```angular2html
docker-compose up
```
Все пять моделей должны иметь статус READY
