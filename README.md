# Telegram Message Forwarder Bot
A telegram bot, which can forward messages from channel, group or chat to another channel, group or chat automatically.

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/viperadnan-git/telegram-message-forwarder-bot/)

## Configuration
To configure this bot you have to use the file template at [`sample.config.toml`](./sample.config.toml). Rename it to `config.toml` and fill the values as described below.
If you want to pass the values as environment variables, then pass the content of the config.toml file as environment variable `CONFIG`.

#### Pyrogram Section
 
_all_media'] else 'Send All Media: Off',
            reply_markup=call.message.reply_markup
        )
    elif call.data == 'send_all_text':
        settings['transform_options']['send_all_text'] = not settings['transform_options']['send_all_text']
        bot.edit_message_text(
            chat_id=call.message.chat.id,
            message_id=call.message.message_id,
            text='Send All Text: On' if settings['transform_options']['send_all_text'] else 'Send All Text: Off',
            reply_markup=call.message.reply_markup
        )

@bot.message_handler(commands=['should_forward'])
def toggle_forwarding(message):
    global settings
    settings['should_forward'] = not settings['should_forward']
    bot.send_message(message.chat.id, "Forwarding is now " + ("on" if settings['should_forward'] else "off"))

def transform_message(message, settings):
    if not settings['should_forward']:
        return None

    text = message.text.strip()
    
    if settings['transform_options']['replace_username']:
        text = text.replace('@' + message.from_user.username, MY_USERNAME)
    
    if settings['transform_options']['replace_links']:
        text = text.replace(MY_URL, MY_URL)
    
    if settings['transform_options']['disable_preview']:
        text = text.replace(MY_URL, MY_URL)
        text = text.replace(MY_URL, MY_URL)
    
    if settings['end_text']:
        text += f"\n{settings['end_text']}"
    
    return text

def forward_message(message, settings):
    if not settings['should_forward']:
        return None

    text = transform_message(message, settings)

    if text is not None:
        for outgoing_chat in settings['outgoing_chats']:
            try:
                bot.send_message(outgoing_chat, text, disable_web_page_preview=settings['transform_options']['disable_preview'])
            except Exception as e:
                print(f"Error forwarding to {outgoing_chat}: {e}")

def check_and_forward_messages(settings):
    for incoming_chat in settings['incoming_chats']:
        try:
            updates = bot.get_updates(offset=-1, timeout=1, allowed_updates=["message"])
            for update in updates:
                message = update.message
                if message.chat.id == incoming_chat:
                    forward_message(message, settings)
        except Exception as e:
            print(f"Error getting updates from {incoming_chat}: {e}")

def main():
    while True:
        current_hour, current_minute = map(int, time.strftime("%H:%M").split(':'))
        scheduled_hour, scheduled_minute = map(int, settings['schedule'].split(':'))

        if current_hour == scheduled_hour and current_minute == scheduled_minute:
            check_and_forward_messages(settings)

        time.sleep(60)  # Check every minute

if __name__ == "__main__":
    main()

bot.polling(none_stop=True)
- `api_id` - Your Telegram API ID.
- `api_hash` - Your Telegram API Hash.
- `session_string` - (Optional if `bot_token` is provided) Session string of your Telegram account. You can get it by running [`get_session.py`](./get_session.py) file.
- `bot_token` - (Optional if `session_string` is provided) Bot token of your Telegram bot. You can get it by creating a bot using [BotFather](https://t.me/BotFather).
- `sudo_users` - (Optional) List of user ids of users who can use the bot. You can get your user id by sending `/id` command to [Rose](https://t.me/MissRose_bot).

```toml
[pyrogram]
api_id = 12345                                                      # required
api_hash = "0123456789abcdef0123456789abcdef"                       # required
bot_token = "123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ"                  # either bot_token or session_string is required
session_string = "1BVn1-ABCD1234efgh5678IJKLmnoPQRsTUVwxyZxxxxxxxx" # either bot_token or session_string is required
sudo_users = [123456789, 123456789]                                 # optional
```

#### Chats Section
- `from` - Chat id of the chat from which messages will be forwarded. You can get it by sending `/id` command to [Rose](https://t.me/MissRose_bot).
- `to` - Chat id of the chat to which messages will be forwarded. You can get it by sending `/id` command to [Rose](https://t.me/MissRose_bot).
- `replace` - (Optional) A dictionary of strings to replace in the message. The key is the string to be replaced and the value is the string to replace with. This is optional.

You can add multiple chats by using the following format.
```toml
[[chats]]
from = -100123456789        # required
to = 123456789              # required
replace = { "old" = "new" } # optional

[[chats]]
from = [123456789, -100123456789]               # required
to = 123456789                                  # required
replace = { "only_apply_to_this_chat" = "new" } # optional

[[chats]]
from = -100123456789            # required
to = [123456789, -100123456789] # required
```

Note: The chats should be in the format of `int` or `list` of `int`. If you want to use usernames, then you have to use the `chat_id` of the chat. You can get it by sending `/id` command to [Rose](https://t.me/MissRose_bot).


### Note
- Supported identifier for a chat should be the chat id.
- Use `/forward` command to forward older messages. For message older than 2 days you have to login as a user and set the `session_string` variable in pyrogram section. Command usage - `/forward <Chat ID/Username/Message Link> <Limit, No. of Messages to forward> <ID of the last message of from chat to avoid repetition>`

## Deployment

### Clone the repository
```bash
git clone <repo-url> telegram-message-forwarder-bot
cd telegram-message-forwarder-bot
```

### Install the requirements
Install the required Python Modules in your machine.
```bash
pip3 install -r requirements.txt
```

### Start the bot
With python3.10 or later.
```bash
python3 -m bot
```

### Docker
Build the image.
```bash
docker build -t telegram-message-forwarder-bot .
```

Run the container.
```bash
docker run -d --name telegram-message-forwarder-bot telegram-message-forwarder-bot
```

## Contributing
- Fork the repository.
- Create a new branch.
- Make your changes.
- Commit and push the changes to your fork.
- Create a pull request.

## Support
- Ask in the [Telegram Group](https://t.me/ViperCommunity) or open an issue.

### Copyright & License
- Copyright &copy; 2021 &mdash; [Adnan Ahmad](https://github.com/viperadnan-git)
- Licensed under the terms of the [GNU General Public License Version 3 &dash; 29 June 2007](./LICENSE)
