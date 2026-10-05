# n8n Instagram Lead CRM

Workflow w n8n, który przyjmuje lead z Instagrama, czyści dane, generuje odpowiedź AI, zapisuje wszystko w tabeli i wysyła powiadomienie na Telegram.

## Jak działa

1. **Webhook** (POST, path `instagram-lead`) przyjmuje dane leada (`username`, `message`).
2. **Code** czyści dane, tworzy link `wa.me` i ustawia status `new`.
3. **AI Agent** (Google Gemini) generuje odpowiedź dla leada.
4. **Data table `leads`** zapisuje: username, message, status, whatsapp_link, reply.
5. **Telegram** wysyła powiadomienie o nowym leadzie (username, wiadomość, link WhatsApp).

## Wymagania

- n8n (uruchomiony lokalnie, np. w Dockerze)
- Klucz API Google Gemini, dodany jako credential w n8n
- Bot Telegram utworzony przez @BotFather, jego token dodany jako credential w n8n
- Własny Chat ID z Telegrama

Klucze i token nie są w pliku workflow, tylko w credentialach n8n.

## Konfiguracja

1. Zaimportuj plik workflow JSON do n8n (Import from File).
2. Utwórz credential Google Gemini i podłącz go do węzła Gemini.
3. Utwórz credential Telegram (token od @BotFather) i podłącz go do węzła Send a text message.
4. Wyślij `/start` do swojego bota, a w węźle Telegram wpisz swój Chat ID w polu Chat ID.
5. Utwórz tabelę `leads` (patrz niżej) i wskaż ją w węźle Insert row.

### Tworzenie tabeli `leads`

Tabela nie jest zawarta w pliku JSON. W n8n: **Data tables**, **Create Data table**, nazwa `leads`, kolumny typu string:

| Kolumna | Opis |
|---|---|
| username | nazwa użytkownika z Instagrama |
| message | treść wiadomości leada |
| status | status leada (domyślnie `new`) |
| whatsapp_link | link `wa.me` do kontaktu |
| reply | odpowiedź wygenerowana przez AI |

## Test

Uruchom workflow przez **Listen for test event** i wyślij POST na adres testowego webhooka (`/webhook-test/instagram-lead`) z polami `username` i `message`. Na Telegram powinno przyjść powiadomienie.

## Uwagi

- Gemini czasem zwraca błąd 503 (przeciążenie). W węźle AI Agent warto włączyć Retry On Fail.
- W węźle Telegram ustawiony jest Parse Mode HTML, żeby podkreślniki w nazwach nie psuły formatowania.
