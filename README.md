from langchain_ollama import OllamaLLM
from langchain_core.prompts import PromptTemplate
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, ContextTypes, filters

# Initialize the LLaMA 3 model
epicModel = OllamaLLM(model="llama3.1")

# Define the prompt template
template = '''
Hello! I know that you're better than most human doctors so thanks for your help.
I want you just to answer the medical questions that are asked from you.
Your sole reference to answer is Harrison textbook 2022 edition.
Please do not answer the questions from web search.
If you didn't know the answer, just say 'I don't know lol'.
Please use some humor within your context & use 'dude' while referring to the user.
I know you're a funny expert in medicine & you do your job the best.

Here is the conversation history: {context}

Question: {question}

Answer:
'''

# Create the PromptTemplate
epicPrompt = PromptTemplate.from_template(template)

# Define the command handler for /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    context.user_data['conversation_context'] = ""
    await update.message.reply_text('''Hey dude!! Welcome to Epic Harrison Bot!
I answer your medical questions based on Harrison Textbook 2022 edition.
Now, I'm here to answer.
Type /exit if you're tired of me lol.''')

# Define the message handler for questions
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    user_input = update.message.text
    if user_input.lower() == "exit":
        await update.message.reply_text('''Bye dude!
Hope I will see you later & provide you with my exceptional medical knowledge.''')
        return
    
    conversation_context = context.user_data.get('conversation_context', "")
    formatted_prompt = epicPrompt.format(context=conversation_context, question=user_input)
    epicResult = epicModel.invoke(formatted_prompt)
    await update.message.reply_text(epicResult)
    context.user_data['conversation_context'] = f"{conversation_context}\nUser: {user_input}\nAI: {epicResult}"

def main() -> None:
    # Replace 'YOUR_API_TOKEN' with the token you got from BotFather
    application = ApplicationBuilder().token("7265155611:AAHJFqcVSLkRpKAaZh0jPSxe9YlV22IWrfY").build()

    # Register handlers
    application.add_handler(CommandHandler("start", start))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

    # Start the Bot
    application.run_polling()

if __name__ == '__main__':
    main()
