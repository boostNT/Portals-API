# Portals-API
Это неофициальная документация по апи [порталс](https://t.me/portals), одной из площадок для торговли телеграм нфт подарками.

###### для ленивых рабочий код на питоне [**тут**](#полный-рабочий-код-на-питоне)

### При заходе в приложение, для получения подарков идет гет запрос по этому юрл:

```python
https://portals-market.com
```

#### Чтобы искать необходимые вам подарки, нужно указывать параметры в юрл запросе.

# Возможные фильтры:
- **filter_by_collections** - Строка с названиями нфт подарков, разделённых запятой, если их больше одного.
- **filter_by_models** - Строка с моделями нфт подарков, разделённых запятой, если их больше одного (показывает подарки даже если не было выбрано название). 
- **filter_by_backdrops** - Строка с фонами нфт подарков, разделённых запятой, если их больше одного.
- **external_collection_number** - Число, по которому будет искать подарки (не конкретно по нему, а все номера в которых содержится число которое вы указали).
- **min_price** - Мин. цена для подарков.
- **max_price** - Макс. цена для подарков.
- **limit** - Макс. кол-во подарков, которое вы получите в ответе (лимит - 100).
- **offset** - Оффсет. Номер, с которого начинать просматривать подарки.
- **status** - Статус подарков. Указываете listed/unlisted (на продаже/не на продаже), или если хотите получить всё вместе то не указываете в принципе
- **sort_by** - Сортировка (см. ниже).

# Сортировка:
Каждый из параметров сортировки принимает значение **desc**/**asc** (по убыванию/по возрастанию) (кроме _listed_at_, он принимает только **desc**) 
- **listed_at** - По времени выставления.
- **price** - По цене.
- **external_collection_number** - По номеру подарка.
- **model_rarity** - По редкости модели.\
В запросе это выглядит так:
```python
https://portals-market.com/api/nfts/search?offset=0&limit=20&sort_by=price+desc&status=listed
```
(выведет самые дорогие подарки)


## Но и этого недостаточно
К сожалению, чтобы как-то успешно взаимодействовать с апи порталс через запросы, нужен заголовок с токеном, который берётся при заходе в веб приложение.

Получить его можно, подделав реальный вход через пирограм ↓
```python
import asyncio
import aiohttp
from urllib.parse import unquote
from pyrogram import Client
from pyrogram.raw.functions.messages import RequestAppWebView
from pyrogram.raw.types import InputBotAppShortName, InputUser, InputBotAppID

api_id = 12123123 # замени с https://my.telegram.org/auth
api_hash = 'fok2jg4h83okpglr' # замени с https://my.telegram.org/auth
client = Client('main', api_id=api_id, api_hash=api_hash)


async def get_auth_token(client: Client):
    async with client:
        bot_entity = await client.get_users('portals')
        bot = InputUser(user_id=bot_entity.id, access_hash=bot_entity.raw.access_hash)
        peer = await client.resolve_peer('portals')
        bot_app = InputBotAppShortName(bot_id=bot, short_name='market')
        web_view = await client.invoke(
            RequestAppWebView(
                peer=peer,
                app=bot_app,
                platform="android",
            )
        )
        init_data = unquote(web_view.url.split('tgWebAppData=', 1)[1].split('&tgWebAppVersion', 1)[0])
        token = f'tma {init_data}'
        return token


async def main():
    token = await get_auth_token(client)
    print(token)

loop = asyncio.get_event_loop()
loop.run_until_complete(main())
```

Либо вы можете зайти в веб тг, нажать f12, открыть вкладку network, зайти в аппку порталса, найти любой гет запрос на `https://portals-market.com` (например `https://portals-market.com/api/nfts/search`), и в Headers в Request Headers найти поле Authorization и взять этот токен.   

![image](https://github.com/user-attachments/assets/2539bdfe-95b0-4365-96de-18b2096b4ab5)   

   
   
Время действия токена неизвестно, но точно более суток.


### Полный рабочий код на питоне:
```python
import asyncio
from urllib.parse import unquote
from curl_cffi import requests
from pyrogram import Client
from pyrogram.raw.functions.messages import RequestAppWebView
from pyrogram.raw.types import InputBotAppShortName, InputUser


api_id = 12123123 # замени с https://my.telegram.org/auth
api_hash = 'fok2jg4h83okpglr' # замени с https://my.telegram.org/auth
client = Client('main', api_id, api_hash)
PORTALS_API_URL = 'https://portals-market.com/api'


async def main():
    token = await get_auth_token(client)
    # token = '' # Или если взяли через веб тг

    headers = {'Authorization': token}

    try:
        r=requests.get(f'{PORTALS_API_URL}/nfts/search?offset=0&limit=20&filter_by_backdrops=Black&sort_by=price desc&status=listed', headers=headers)
        rj=r.json()
        gifts = rj['results']
        print(gifts)
    except Exception as e:
        print(e)

    # Код выведет 20 самых дорогих подарков на чёрном фоне



async def get_auth_token(client: Client):
    async with client:
        bot_entity = await client.get_users('portals')
        bot = InputUser(user_id=bot_entity.id, access_hash=bot_entity.raw.access_hash)
        peer = await client.resolve_peer('portals')
        bot_app = InputBotAppShortName(bot_id=bot, short_name='market')
        web_view = await client.invoke(
            RequestAppWebView(
                peer=peer,
                app=bot_app,
                platform="android",
            )
        )
        init_data = unquote(web_view.url.split('tgWebAppData=', 1)[1].split('&tgWebAppVersion', 1)[0])
        token = f'tma {init_data}'
        return token

loop = asyncio.get_event_loop()
loop.run_until_complete(main())
```

Тг для связи - [@bxxst](t.me/bxxst)

