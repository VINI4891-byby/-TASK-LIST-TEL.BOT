from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes

TOKEN = 'YOUR_TOKEN_HERE'
tasks = []
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Обработчик команды '/start'"""
    await update.message.reply_text(
        "Привет! Я твой первый бот.\nИспользуйте следующие команды:\n/add - добавить задачу\n/list - показать список задач\n/done - отметить задачу выполненной\n/delete - удалить задачу"
    )
async def add_task(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Обработчик команды '/add'. Добавляет новую задачу."""
    task = update.message.text.split(' ', 1)[1]
    tasks.append(task)
    await update.message.reply_text(f"Твоя задача '{task}' была успешно добавлена!")

async def list_tasks(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Обработчик команды '/list'. Показывает список текущих задач."""
    if not tasks:
        await update.message.reply_text("У тебя пока нет задач.")
    else:
        response = "\n".join([f"{i+1}. {task}" for i, task in enumerate(tasks)])
        await update.message.reply_text(response)
async def done_task(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Обработчик команды '/done'. Отмечает задачу выполненной."""
    try:
        index = int(update.message.text.split(' ', 1)[1]) - 1
        if 0 <= index < len(tasks):
            completed_task = tasks.pop(index)
            await update.message.reply_text(f"Твоя задача '{completed_task}' отмечена как выполненная!")
        else:
            await update.message.reply_text("Неверный индекс задачи.")
    except ValueError:
        await update.message.reply_text("Укажите номер задачи.")
async def delete_task(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Обработчик команды '/delete'. Удаляет задачу."""
    try:
        index = int(update.message.text.split(' ', 1)[1]) - 1
        if 0 <= index < len(tasks):
            deleted_task = tasks.pop(index)
            await update.message.reply_text(f"Твоя задача '{deleted_task}' удалена.")
        else:
            await update.message.reply_text("Неверный индекс задачи.")
    except ValueError:
        await update.message.reply_text("Укажите номер задачи.")
if __name__ == '__main__':
    application = ApplicationBuilder().token(TOKEN).build()

    application.add_handler(CommandHandler("start", start))
    application.add_handler(CommandHandler("add", add_task))  
    application.add_handler(CommandHandler("list", list_tasks)) 
    application.add_handler(CommandHandler("done", done_task))  
    application.add_handler(CommandHandler("delete", delete_task)) 

    application.run_polling()
