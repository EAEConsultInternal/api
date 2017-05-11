Описание формата API системы Владелец.Онлайн
# О сервисе
Владелец.Онлайн – это SaaS-сервис для автоматического заполнения реквизитов контрагентов на основе сведений из ЕГРЮЛ.
# Общие сведения
  - Базовый URL для методов API расположен по адресу https://api.vladelets.online
  - Обмен данными с сервисом  осуществляется по протоколу HTTPS.
  - Формат данных ответа – JSON.
  - Параметры в запроса должны быть в кодировке UTF-8 и закодированы для передачи в URL (http://en.wikipedia.org/wiki/Percent-encoding).
# Аутентификация
Для доступа к методам сервиса требуется передавать аутентификационный JWT-токен в заголовке Authorization с использованием схемы Bearer. Срок действия токена – 365 дней.
Для получения токена обратитесь на почту sales@vladelets.online.

# Методы
В этом разделе описаны возможные запросы к API и форматы ответа на них.
### Поиск российских организаций
#### Запрос
`GET /v1/companies`
#### Параметры
| Параметр | Тип | Формат | Описание |
| ------ | ------ | ------ | ------ |
| `inn` | GET | строка 10 цифр, необязательный | ИНН организации |
| `kpp` | GET | строка 9 цифр, необязательный | КПП организации |
| `ogrn` | GET | строка из 13 цифр, необязательный | ОГРН организации |
| `name` | GET | строка, необязательный | Наименование организации 

В запросе должен быть передан по крайней мере один из этих параметров. В случае передачи двух и более параметров поиск будет производиться по соответствию всех переданных значений (логическое "И").
#### Пример запроса
`https://api.vladelets.online/v1/companies?inn=7734517839&kpp=773401001`
#### Пример ответа
```json
{
   "results":[
      {
         "full_name":"ЕАЕ",
         "compact_name":"ООО \"ЕАЕ\"",
         "inn":"7734517839",
         "kpp":"773401001",
         "ogrn":"1047796788930",
         "okpo":"75577697",
         "okato":"77401367000",
         "registration_date":"2004-10-19",
         "tax_inspection_code":"5906",
         "is_bank":false,
         "address":{
            "full":"123308, г. Москва, ул. Демьяна Бедного, владение 24, к. 2а, строение III, кв. 3",
            "postal_code":"123308",
            "country":"Российская Федерация",
            "federal_district":"Центральный",
            "region":{
               "full":"Пермский край",
               "code":"77",
               "name":"Пермский",
               "type":"край"
            },
            "area": "Пермский район",
            "city":{
               "full":"г. Пермь",
               "name":"Пермь",
               "type":"город"
            },
            "street":"ул. Демьяна Бедного",
            "house":"владение 24",
            "housing":"к. 2а, строение III",
            "room":"кв. 3"
         },
         "form":{
            "compact_form":"ООО",
            "full_form":"Общество с ограниченной ответственностью",
            "code":"12300"
         },
         "head":{
            "position":"Руководитель",
            "inn":"771811057155",
            "last_name":"Иванов",
            "first_name":"Владимир",
            "middle_name":"Иванович"
         },
         "okved":[
            {
               "code":"63.11.1",
               "description":"Деятельность по созданию и использованию баз данных и информационных ресурсов",
               "is_primary":true
            },
            {
               "code":"68.32",
               "description":"Управление недвижимым имуществом за вознаграждение или на договорной основе"
            },
            {
               "code":"95.11",
               "description":"Ремонт компьютеров и периферийного компьютерного оборудования"
            }
         ]
      }
   ]
}
```

### Комплексная проверка юридического лица

#### Запрос
`GET /v1/compliance/legal`

#### Параметры запроса
| Параметр | Тип | Формат | Описание |
| ------ | ------ | ------ | ------ |
| `ogrn` | GET | строка из 13 цифр, необязательный | ОГРН организации |
| `inn` | GET | строка 10 цифр, необязательный | ИНН организации |
| `kpp` | GET | строка 9 цифр, необязательный | КПП организации |

В запросе должен быть передан по крайней мере один из этих параметров, достаточный для идентификации одной организации – ОГРН или ИНН+КПП.

#### Параметры ответа
Ответ содержит базовую справку об организации и найденные риски, связанные с различными аспектами организации или ее структуры. Проверка всех рисков производится по запрашиваемой организации и всей структуре ее владения "вверх" (учредители компании) и "вниз" (учрежденные компании). Найденные риски содержатся в поле `occurrences` и могут принимать следующие значения:

| Код риска | Описание |
| ------ | ------ |
| `terrorist` | В отношении физического или юридического лица имеются сведения о причастности к терроризму |
| `same_manager_founder` | Руководитель и единственный участник являются одним лицом |
| `frequent_change_of_founders` | За последние 3 года в организации происходила частая смена учредителей (меняются чаще чем раз в год) |
| `disqualified_management` | В состав исполнительных органов юридического лица входят дисквалифицированные лица |
| `recent_change_of_management` | За последний год сменился руководитель организации |
| `mass_management_person` | В состав исполнительных органов юридического лица входит физическое лицо, являющееся массовым руководителем |
| `mass_management_legal` | Юридическое лицо, имеющее признаки массовой управляющей компании |
| `mass_founder` | Физическое или юридическое лицо, имеющее признаки массового учредителя |
| `mass_address` | Юридический адрес организации находится в перечне адресов, указанных при государственной регистрации в качестве места нахождения несколькими юридическими лицами |
| `invalid_address` | Юридическое лицо находится в перечне юридических лиц, связь с которыми по указанному ими адресу (месту нахождения), внесенному в ЕГРЮЛ, отсутствует |
| `min_capital` | Организация имеет уставный капитал, равный минимальному размеру уставного капитала для данной организационно-правовой формы |
| `liquidation_reorganization` | Организация находится в стадии реорганизации или ликвидации |
| `liquidated_founder` | Организация имеет ликвидированное юридическое лицо в цепочке учредителей |
| `recently_registered` | Период деятельности организации с момента регистрации составляет не более 3 месяцев |
| `dishonest_contractor` | Организация находится в реестре недобросовестных поставщиков |
| `operational_risk` | Риски, связанные с проведением клиентом определенного вида операций |

Каждый риск содержит в поле `meta` дополнительную информацию, которая идентифицирует субъект риска

| Параметр | Описание |
| ------ | ------ |
| `type` | Тип объекта, может принимать следующие значения: <ul><li>`legal` – юридическое лицо</li><li>`person` – физическое лицо</li></ul> |
| `relation` | Описывает положение субъекта риска в цепочке владения проверяемой компании, может принимать следующие значения: <ul><li>`self` – проверяемая организация</li><li>`founder` – является прямым или косвенным учредителем проверяемой организации</li><li>`established` – является дочерней организацией по отношению к проверяемой</li></ul> |
| `inn` | ИНН субъекта риска – физического или юридического лица |
| `ogrn` | ОГРН субъекта риска – юридического лица. Если субъектом является физическое лицо, в поле `ogrn` приходит `null` |
| `name` | Наименование организации или имя физического лица |
| `text` | Оригинальная строка из соответствующего реестра для возможности ручной проверки |

#### Пример запроса
`https://api.vladelets.online/v1/compliance/legal?ogrn=1027700043502`

#### Пример ответа
```json
{
  "error": null,
  "company": "<Объект, идентичный результату ответа на запрос «Поиск российских организаций»>",
  "compliance": {
     "has_risks": true,
     "occurrences": [
      {
         "risk": "terrorist",
         "title": "Террористические списки",
         "description": "Физическое лицо находится в перечне организаций и физических лиц, в отношении которых имеются сведения об их причастности к экстремистской деятельности или терроризму",
         "meta": {
            "type": "person",
            "relation": "founder",
            "inn": 1111111111,
            "ogrn": null,            
            "name": "Абдулаев Абдула Мусаевич",
            "text": "АБДУЛАЕВ АБДУЛА МУСАЕВИЧ*, 23.11.1976 г.р. , Г.МАХАЧКАЛА РЕСПУБЛИКИ ДАГЕСТАН"
         }
      },
      {
         "...": "..."
      }
   ]
  }
}
```

По вопросам работы с API обращайтесь на support@vladelets.online
