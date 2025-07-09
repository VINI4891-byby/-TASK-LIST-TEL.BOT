from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes

TOKEN = 'TOKEN'

tasks = {}

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """ Обработчик команды '/start'. """
    await update.message.reply_text(
        "Привет! Я твой первый бот.\nИспользуй следующие команды:\n"
        "/add <описание> - добавить задачу\n"
        "/list - показать список задач\n"
        "/done <номер> - отметить задачу выполненной\n"
        "/delete <номер> - удалить задачу"
    )

async def add_task(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """ Обработчик команды '/add'. Добавляет новую задачу. """
    args = update.message.text.split(' ')
    if len(args) > 1:
        user_id = update.effective_user.id
        task = ' '.join(args[1:])
        if user_id not in tasks:
            tasks[user_id] = []  # Создаем пустой список задач для нового пользователя
        tasks[user_id].append(task)
        await update.message.reply_text(f"Твоя задача '{task}' была успешно добавлена!")
    else:
        await update.message.reply_text("Нужно ввести описание задачи после команды '/add'")

async def list_tasks(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """ Обработчик команды '/list'. Показывает список текущих задач. """
    user_id = update.effective_user.id
    if user_id in tasks and tasks[user_id]:
        response = "\n".join([f"{i + 1}. {task}" for i, task in enumerate(tasks[user_id])])
        await update.message.reply_text(response)
    else:
        await update.message.reply_text("У тебя пока нет задач.")

async def done_task(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """ Обработчик команды '/done'. Отмечает задачу выполненной. """
    args = update.message.text.split(' ')
    if len(args) > 1:
        user_id = update.effective_user.id
        try:
            index = int(args[1]) - 1
            if user_id in tasks and 0 <= index < len(tasks[user_id]):
                completed_task = tasks[user_id].pop(index)
                await update.message.reply_text(f"Твоя задача '{completed_task}' выполнена!\n")
                await list_tasks(update, context)  # Перечисляем оставшиеся задачи
            else:
                await update.message.reply_text("Неверный индекс задачи.")
        except ValueError:
            await update.message.reply_text("Укажи правильный номер задачи.")
    else:
        await update.message.reply_text("Нужно указать номер задачи после команды '/done'")

async def delete_task(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """ Обработчик команды '/delete'. Удаляет задачу. """
    args = update.message.text.split(' ')
    if len(args) > 1:
        user_id = update.effective_user.id
        try:
            index = int(args[1]) - 1
            if user_id in tasks and 0 <= index < len(tasks[user_id]):
                deleted_task = tasks[user_id].pop(index)
                await update.message.reply_text(f"Твоя задача '{deleted_task}' удалена!\n")
                await list_tasks(update, context)  # Обновляем список оставшихся задач
            else:
                await update.message.reply_text("Неверный индекс задачи.")
        except ValueError:
            await update.message.reply_text("Укажи правильный номер задачи.")
    else:
        await update.message.reply_text("Нужно указать номер задачи после команды '/delete'")

if __name__ == '__main__':
    application = ApplicationBuilder().token(TOKEN).build()

    # Регистрируем обработчики команд
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CommandHandler("add", add_task))
    application.add_handler(CommandHandler("list", list_tasks))
    application.add_handler(CommandHandler("done", done_task))
    application.add_handler(CommandHandler("delete", delete_task))

    application.run_polling()
