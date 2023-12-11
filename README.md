# django-clickhouse-logger

## Описание

Этот проект предоставляет возможность логировать ошибки Django в базу данных Clickhouse.
Логи записываются через отложенные задачи в celery, в отдельной очереди, чтобы не нагружать другие воркеры.


### Установка

Установите через pip:

```bash
pip install clickhouse-django-logger  # Установка стабильной версии
pip install -U git+https://github.com/Sobolev5/django-clickhouse-logger.git@master  # Установка версии разработки
```
Добавьте clickhouse_django_logger в INSTALLED_APPS:
```bash
INSTALLED_APPS = INSTALLED_APPS + ("clickhouse_django_logger",)
```

Установите переменные среды для Clickhouse logger в settings.py:
```bash
DJANGO_CLICKHOUSE_LOGGER_HOST = "clickhouse"
DJANGO_CLICKHOUSE_LOGGER_PORT = 8123
DJANGO_CLICKHOUSE_LOGGER_USER = "default"
DJANGO_CLICKHOUSE_LOGGER_PASSWORD = "default"
# Для создания отдельной очереди для celery:
CELERY_TASK_QUEUES = {
    'logger': {
        'exchange': 'logger',
        'exchange_type': 'direct',
        'binding_key': 'logger',
    },
}
```
Добавьте Clickhouse logger в конфигурацию логгера в settings.py:
```bash
LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        'verbose': {
            'format': '%(levelname)s %(asctime)s %(module)s %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(message)s'
        },
    },
    "handlers": {
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'error': {
            'level': 'INFO',
            'class': 'logging.handlers.WatchedFileHandler',
            'filename': 'logs/error.log',
            'formatter': 'verbose'
        },
        'mail_admins': {
            'level': 'ERROR',
            'class': 'django.utils.log.AdminEmailHandler',
        },
        'info': {
            'level': 'INFO',
            'class': 'logging.handlers.WatchedFileHandler',
            'filename': 'logs/info.log',
            'formatter': 'verbose'
        },
        "django_clickhouse_logger": {
            "level": "INFO",
            "class": "clickhouse_django_logger.handlers.LoggerHandler",
        },
    },
    "loggers": {
        'django.request': {
            'handlers': ['mail_admins', 'django_clickhouse_logger', 'console'],
            'level': 'ERROR',
            'propagate': False,
        },
        'django.template': {
            'handlers': ['django_clickhouse_logger', 'mail_admins'],
            'level': 'ERROR',
            'propagate': False,
        },
        'info': {
            'handlers': ['django_clickhouse_logger'],
            'level': 'INFO',
        },
        'celery': {
            'handlers': ['console', 'django_clickhouse_logger'],
            'level': 'ERROR',
            'propagate': True
        },
    },

}
```
### Контакты
Если у вас есть вопросы или предложения, не стесняйтесь связаться со мной по электронной почте: singleservice2022@gmail.com