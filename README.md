# Структура проекта

- `real_estate_model/` содержит весь основной код пакета: `train.py`, `predict.py`, `config.py` (загрузка параметров обучения и предсказания из файла `config.yml`). Здесь же находится директория `trained_models/`, куда при обучении будут попадать веса новых моделей, и в которой уже есть одна обученная модель.
- `requirements/` содержит все необходимые зависимости для использования (`requirements.txt`) и тестирования (`test_requirements.txt`) пакета.
- `tests/` содержит код тестов, запускаемый при помощи pytest.
- `webapp/` содержит код веб-интерфейса.

`predict.py` имеет функционал для загрузки состояния модели из конфига (`load_model`), предсказания с помощью предзагруженной модели (`predict_raw`) и предсказания моделью, автоматически загружаемой из файла, прописанном в конфиге (`predict`).

`config.py` имеет функции `fetch_config_from_yaml` для полного считывания параметров конфига и `set_config_field` для изменения значения конкретного параметра (не поддерживает списки).


# Установка

Пакет опубликован на [PyPI](https://pypi.org/project/RealEstate-package/) и устанавливается командой:

```
pip install RealEstate-package
```


# Использование

Код запускается и тестируется при помощи tox. Параметры обучения и предсказания находятся в `real_estate_model/config.yml`.


## Обучение

Перед началом обучения:

- НЕОБХОДИМО скачать [датасет](https://www.kaggle.com/datasets/mrdaniilak/russia-real-estate-20182021) и прописать абсолютный путь к csv-файлу в конфиге в поле `dataset` (можно сделать это программно с помощью функции `config.set_config_field`);
- ОПЦИОНАЛЬНО настроить другие параметры обучения:
    + `trained_model_path`: относительный путь сохранения обученных моделей, по умолчанию `./trained_models/` (имя генерируется с помощью timestamp);
    + `variables_to_drop`: список переменных, не учитываемых при обучении (проект рассчитан на предсказание стоимости недвижимости, поэтому она игнорируется автоматически);
    + `random_state`;
    + `test_size`: доля тестовой выборки для функции `train_test_split`;
    + `num_boost_round`: внутренний параметр модели, отвечающий за количество раундов (итераций);
    + `early_stopping_rounds`: внутренний параметр модели, отвечающий за раннюю остановку обучения, если качество не улучшалось в течение указанного количества раундов;
    + `verbose_eval`: внутренний параметр модели, отвечающий за частоту вывода информации об обучении (в количестве раундов);
    + `metric`: внутренний параметр модели функции оценки качества, по умолчанию `rmse`, прочие метрики доступны по [ссылке](https://lightgbm.readthedocs.io/en/latest/Parameters.html#metric).

Обучение запускается из корневой директории проекта командой:

```
tox -e train
```


## Предсказание

Для предсказания в конфиге указывается путь к весам обученной модели (по умолчанию `trained_models/lgb_model.txt`) в поле `predict_model` и список переменных, на которых она обучалась (`predictors`).

```
tox -e predict
```


# Веб-приложение

Основной функционал доступен в веб-приложении, которое запускается командой:

```
tox -e webapp
```

Предоставляется возможность:
- предсказать значение стоимости недвижимости по некоторым параметрам во вкладке `/predict`;
- обучить новую модель во вкладке `/train` (параметры обучения будут взяты из конфига). По окончании обучения будет выведена информация об абсолютном и относительном путях сохранённой модели и о её качестве на train- и test-выборках (R2-score).
- изменить модель для предсказания в `/change_model`, передав относительный путь к файлу с весами на вход.
- изменить количество итераций обучения в `/change_iterations`.


# Датасет

Для обучения существующей модели использовался датасет:
https://www.kaggle.com/datasets/mrdaniilak/russia-real-estate-20182021
