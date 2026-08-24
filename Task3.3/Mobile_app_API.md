# API для мобильного приложения 

## Спецификация API
```
openapi: 3.2.0

info:
  title: Здоровье+ Appointment API
  description: API мобильного приложения для работы с записями пациента
  version: 1.0.0
servers:
  - url: https://api.health-plus.ru/api/v1
paths:
  /appointments:
    get:
      summary: Получение списка записей пациента
      description: >
        Возвращает записи текущего аутентифицированного пациента.
        Поддерживает фильтрацию по статусу и пагинацию.
      tags:
        - Appointments
      security:
        - bearerAuth: []
      parameters:
        - $ref: '#/components/parameters/status'
        - $ref: '#/components/parameters/limit'
        - $ref: '#/components/parameters/offset'
      responses:
        '200':
          $ref: '#/components/responses/AppointmentsResponse'
        '401':
          $ref: '#/components/responses/UnauthorizedResponse'
        '500':
          $ref: '#/components/responses/InternalErrorResponse'

components:

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  parameters:
    status:
      name: status
      in: query
      required: false
      description: Фильтр записей по статусу
      schema:
        type: string
        enum:
          - upcoming
          - completed
          - cancelled
      example: upcoming
    limit:
      name: limit
      in: query
      required: false
      description: Количество записей в ответе
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
      example: 20
    offset:
      name: offset
      in: query
      required: false
      description: Количество записей, которые необходимо пропустить
      schema:
        type: integer
        minimum: 0
        default: 0
      example: 0

  responses:

    AppointmentsResponse:
      description: Список записей успешно получен
      content:
        application/json:
          schema:
            type: object
            properties:
              limit:
                type: integer
                description: Количество записей в ответе
                example: 20
              offset:
                type: integer
                description: Количество пропущенных записей
                example: 0
              total:
                type: integer
                description: Общее количество записей
                example: 1
              appointments:
                type: array
                items:
                  $ref: '#/components/schemas/Appointment'

    UnauthorizedResponse:
      description: Пользователь не аутентифицирован или токен недействителен
      content:
        application/json:
          schema:
            type: object
            required:
              - code
              - message
            properties:
              code:
                type: string
                description: Код ошибки
                example: "UNAUTHORIZED"
              message:
                type: string
                description: Описание ошибки
                example: "Access token is missing, expired or invalid"

    InternalErrorResponse:
      description: Внутренняя ошибка сервера
      content:
        application/json:
          schema:
            type: object
            required:
              - code
              - message
            properties:
              code:
                type: string
                description: Код ошибки
                example: "INTERNAL_ERROR"
              message:
                type: string
                description: Описание ошибки
                example: "Unable to retrieve appointments"

  schemas:

    Appointment:
      type: object
      required:
        - id
        - status
        - startAt
        - doctor
        - clinic
      properties:
        id:
          type: string
          description: Уникальный идентификатор записи
          example: "42"
        status:
          type: string
          enum:
            - upcoming
            - completed
            - cancelled
          example: upcoming
        startAt:
          type: string
          format: date-time
          description: Дата и время начала приёма
          example: "2026-08-28T14:30:00+03:00"
        doctor:
          type: object
          properties:
            id:
              type: string
              example: "15"
            name:
              type: string
              example: "Иванова Анна Сергеевна"
            specialty:
              type: string
              example: "Терапевт"
        clinic:
          type: object
          properties:
            id:
              type: string
              example: "21"
            name:
              type: string
              example: "Здоровье+"
```

## Обоснование
1. Для пагинации выбран limit/offset, поскольку список записей пациента обычно относительно небольшой и клиенту удобна простая постраничная загрузка. 

2. Версия API указана в URL — /api/v1. Это наиболее простой и наглядный вариант, который решает задачу интеграции мобильного приложения в достаточной мере без ибыточных доработок. 
