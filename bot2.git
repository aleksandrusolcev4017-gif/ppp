import logging
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, CallbackQueryHandler, ContextTypes
import sqlite3
import asyncio

# Настройка логирования
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger=logging.getLogger( __name__ )

# Токен бота
BOT_TOKEN=""

# Каналы для подписки (username каналов БЕЗ @)
CHANNELS=[
    {"name" : "MellstroyGame(MGM)", "username" : "projectMGM"},
    {"name" : "Создатель MGM", "username" : "createrMGM"}
]


# Инициализация базы данных
def init_db() :
    conn=sqlite3.connect( 'users.db' )
    cursor=conn.cursor()
    cursor.execute( '''
        CREATE TABLE IF NOT EXISTS users (
            user_id INTEGER PRIMARY KEY,
            username TEXT,
            first_name TEXT,
            last_name TEXT,
            subscribed BOOLEAN DEFAULT FALSE,
            registered_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    ''' )
    conn.commit()
    conn.close()


# Правильная проверка подписки на каналы
async def check_subscription(user_id: int, context: ContextTypes.DEFAULT_TYPE) -> bool :
    try :
        for channel in CHANNELS :
            try :
                # Проверяем статус пользователя в канале
                chat_member=await context.bot.get_chat_member(
                    chat_id=f"@{channel['username']}",
                    user_id=user_id
                )

                # Статусы, которые считаются подписанными
                if chat_member.status in ['left', 'kicked', 'restricted'] :
                    logger.info( f"Пользователь {user_id} не подписан на @{channel['username']}" )
                    return False

            except Exception as e :
                logger.error( f"Ошибка при проверке канала @{channel['username']}: {e}" )
                return False

        logger.info( f"Пользователь {user_id} подписан на все каналы" )
        return True

    except Exception as e :
        logger.error( f"Критическая ошибка проверки подписки: {e}" )
        return False


# Команда /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None :
    user=update.effective_user
    user_id=user.id

    # Сохраняем пользователя в БД
    conn=sqlite3.connect( 'users.db' )
    cursor=conn.cursor()
    cursor.execute( '''
        INSERT OR REPLACE INTO users (user_id, username, first_name, last_name)
        VALUES (?, ?, ?, ?)
    ''', (user_id, user.username, user.first_name, user.last_name) )
    conn.commit()
    conn.close()

    # Проверяем подписку
    is_subscribed=await check_subscription( user_id, context )

    if is_subscribed :
        # Пользователь подписан - показываем приветствие и ссылку
        await send_welcome_message( update, context )
    else :
        # Пользователь не подписан - просим подписаться
        await ask_for_subscription( update, context )


# Красивое приветственное сообщение
async def send_welcome_message(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None :
    user=update.effective_user
    first_name=user.first_name or "друг"

    welcome_text=f"""
✨ *Ну шо ты мой маленький... НУ ПРИВЕЕЕТ* ✨

*{first_name}*, ты попал в самый крутой крипто-проект *MellstroyGame*! 🚀

🎮 *Что тут происходит?*
Ты сможешь играя в игру зарабатывать крипто-монеты *MGM*! Да-да, именно зарабатывать, просто кликая и получая кайф! 💰

🪙 *Что такое MGM?*
*MGM* - это мем-монета, посвященная легендарному стримеру *Меллстрою*! 
Это не просто токен, это целая культура! 🌟

📊 *Токеномика:*
• *Всего монет:* 1 ТРИЛЛИОН MGM 🪙
• *40%* - на ликвидность 💧
• *30%* - на раздачу сообществу 🎁  
• *30%* - создателю проекта ⚡

🎯 *Как начать играть?*
Просто нажми кнопку ниже и погрузись в мир MGM! Там тебя ждут:
• Кликер с Меллстроем 🖱️
• Система рангов 🏆
• Магазин скинов 🛍️
• Бонусы каждые 5 минут 🎁
• И многое другое! ✨

🎊 *Добро пожаловать в семью MGM, {first_name}!* 🎊

*P.S.* Не забывай заходить каждый день за бонусами! 🔥
    """

    # Создаем кнопку для перехода к игре
    keyboard=[
        [InlineKeyboardButton( "🎮 НАЧАТЬ ИГРАТЬ!", url="t.me/mellstroyGM_bot/play" )],
        [InlineKeyboardButton( "📢 Наши каналы", callback_data="channels" )],
        [InlineKeyboardButton( "ℹ️ О проекте", callback_data="about" )]
    ]
    reply_markup=InlineKeyboardMarkup( keyboard )

    if hasattr( update, 'message' ) and update.message :
        await update.message.reply_text(
            welcome_text,
            reply_markup=reply_markup,
            parse_mode='Markdown',
            disable_web_page_preview=True
        )
    else :
        await update.callback_query.edit_message_text(
            welcome_text,
            reply_markup=reply_markup,
            parse_mode='Markdown',
            disable_web_page_preview=True
        )


# Запрос на подписку
async def ask_for_subscription(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None :
    user=update.effective_user
    first_name=user.first_name or "друг"

    subscription_text=f"""
🤝 *Привет, {first_name}!* 

Чтобы получить доступ к игре *MellstroyGame*, нужно подписаться на наши официальные каналы!

📢 *Обязательные каналы для подписки:*
    """

    # Добавляем информацию о каждом канале
    buttons=[]
    for channel in CHANNELS :
        subscription_text+=f"\n• [{channel['name']}](https://t.me/{channel['username']})"
        buttons.append( [InlineKeyboardButton(
            f"📢 {channel['name']}",
            url=f"https://t.me/{channel['username']}"
        )] )

    subscription_text+="""

⚡ *После подписки нажми кнопку «ПРОВЕРИТЬ ПОДПИСКУ» ниже*

🎮 *Что тебя ждет в игре:*
• Заработок MGM монет 💰
• Увлекательный кликер 🖱️
• Система достижений 🏆
• Магазин скинов 🛍️
• Ежедневные бонусы 🎁
    """

    buttons.append( [InlineKeyboardButton( "✅ ПРОВЕРИТЬ ПОДПИСКУ", callback_data="check_subscription" )] )

    reply_markup=InlineKeyboardMarkup( buttons )

    if hasattr( update, 'message' ) and update.message :
        await update.message.reply_text(
            subscription_text,
            reply_markup=reply_markup,
            parse_mode='Markdown',
            disable_web_page_preview=True
        )
    else :
        await update.callback_query.edit_message_text(
            subscription_text,
            reply_markup=reply_markup,
            parse_mode='Markdown',
            disable_web_page_preview=True
        )


# Обработчик callback-ов
async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None :
    query=update.callback_query
    await query.answer()

    user_id=query.from_user.id

    if query.data == "check_subscription" :
        # Проверяем подписку
        is_subscribed=await check_subscription( user_id, context )

        if is_subscribed :
            # Обновляем статус в БД
            conn=sqlite3.connect( 'users.db' )
            cursor=conn.cursor()
            cursor.execute( 'UPDATE users SET subscribed = TRUE WHERE user_id = ?', (user_id,) )
            conn.commit()
            conn.close()

            # Показываем приветствие
            await send_welcome_message_from_callback( query, context )
        else :
            await query.edit_message_text(
                "❌ *Вы не подписаны на все каналы!*\n\n"
                "Пожалуйста, подпишитесь на все указанные каналы и нажмите проверку снова.\n\n"
                "*Убедитесь что:*\n"
                "• Вы подписались на ВСЕ каналы\n"
                "• Не вышли из каналов\n"
                "• Нажали кнопки подписки выше",
                parse_mode='Markdown',
                reply_markup=InlineKeyboardMarkup( [[
                    InlineKeyboardButton( "🔄 ПРОВЕРИТЬ СНОВА", callback_data="check_subscription" )
                ]] )
            )

    elif query.data == "channels" :
        channels_text="📢 *Наши официальные каналы:*\n\n"
        for channel in CHANNELS :
            channels_text+=f"• [{channel['name']}](https://t.me/{channel['username']})\n"

        channels_text+="\n*Подпишись чтобы получить доступ к игре!*"

        await query.edit_message_text(
            channels_text,
            parse_mode='Markdown',
            disable_web_page_preview=True,
            reply_markup=InlineKeyboardMarkup( [[
                InlineKeyboardButton( "✅ ПРОВЕРИТЬ ПОДПИСКУ", callback_data="check_subscription" )
            ]] )
        )

    elif query.data == "about" :
        about_text="""
ℹ️ *О проекте MellstroyGame*

🎮 *MellstroyGame* - это инновационная крипто-игровая экосистема, построенная вокруг мем-монеты MGM, посвященной культовому стримеру Меллстрою.

🪙 *MGM Token:*
• Общий объем: 1,000,000,000,000 MGM
• Распределение: 40% ликвидность, 30% комьюнити, 30% создатель
• Цель: Создание выбрано комьюнити вокруг контента Меллстроя

🎯 *Фичи игры:*
• Увлекательный кликер посвещенный Меллстрою
• Система рангов и достижений
• Магазин уникальных скинов
• Автокликеры и бусты
• Бонусы каждые 5 минут

🚀 *Наша миссия:*
Объединить аудитории Меллстроя через игровую механику и создать сильное комьюнити вокруг токена MGM!
        """
        await query.edit_message_text(
            about_text,
            parse_mode='Markdown',
            reply_markup=InlineKeyboardMarkup( [[
                InlineKeyboardButton( "🎮 НАЧАТЬ ИГРАТЬ", url="t.me/mellstroyGM_bot/play" )
            ]] )
        )


# Приветствие из callback
async def send_welcome_message_from_callback(query, context: ContextTypes.DEFAULT_TYPE) -> None :
    user=query.from_user
    first_name=user.first_name or "друг"

    welcome_text=f"""
✨ *Ну шо ты мой маленький... НУ ПРИВЕЕЕТ* ✨

*{first_name}*, спасибо за подписку! Ты официально в семье MGM! 🚀

🎮 *Что тут происходит?*
Ты сможешь играя в игру зарабатывать крипто-монеты *MGM*! 💰

🪙 *Что такое MGM?*
*MGM* - это мем-монета, посвященная легендарному стримеру *Меллстрою*! 🌟

🎯 *Как начать играть?*
Просто нажми кнопку ниже и погрузись в мир MGM!

🎊 *Добро пожаловать в семью MGM, {first_name}!* 🎊
    """

    keyboard=[
        [InlineKeyboardButton( "🎮 НАЧАТЬ ИГРАТЬ!", url="t.me/mellstroyGM_bot/play" )],
        [InlineKeyboardButton( "📢 Наши каналы", callback_data="channels" )],
        [InlineKeyboardButton( "ℹ️ О проекте", callback_data="about" )]
    ]
    reply_markup=InlineKeyboardMarkup( keyboard )

    await query.edit_message_text(
        welcome_text,
        reply_markup=reply_markup,
        parse_mode='Markdown',
        disable_web_page_preview=True
    )


# Команда /stats (только для админов)
async def stats(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None :
    # Проверяем админские права
    ADMINS=[123456789]  # Замените на ID админов

    if update.effective_user.id not in ADMINS :
        await update.message.reply_text( "❌ Эта команда только для администраторов." )
        return

    conn=sqlite3.connect( 'users.db' )
    cursor=conn.cursor()

    # Получаем статистику
    cursor.execute( 'SELECT COUNT(*) FROM users' )
    total_users=cursor.fetchone()[0]

    cursor.execute( 'SELECT COUNT(*) FROM users WHERE subscribed = TRUE' )
    subscribed_users=cursor.fetchone()[0]

    conn.close()

    stats_text=f"""
📊 *Статистика бота:*

👥 Всего пользователей: *{total_users}*
✅ Подписавшихся: *{subscribed_users}*
❌ Неподписавшихся: *{total_users - subscribed_users}*

📈 Конверсия: *{(subscribed_users / total_users * 100) if total_users > 0 else 0:.1f}%*
    """

    await update.message.reply_text( stats_text, parse_mode='Markdown' )


# Обработчик ошибок
async def error_handler(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None :
    logger.error( msg="Exception while handling an update:", exc_info=context.error )


# Основная функция
def main() -> None :
    # Инициализируем БД
    init_db()

    # Создаем Application
    application=Application.builder().token( BOT_TOKEN ).build()

    # Добавляем обработчики
    application.add_handler( CommandHandler( "start", start ) )
    application.add_handler( CommandHandler( "stats", stats ) )
    application.add_handler( CallbackQueryHandler( button_handler ) )

    # Обработчик ошибок
    application.add_error_handler( error_handler )

    # Запускаем бота
    print( "🤖 Бот запущен..." )
    application.run_polling()


if __name__ == '__main__' :

    main()
