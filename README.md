import asyncio
import websockets
import sys

async def receive_messages(websocket):
    """Функция для постоянного приема сообщений от сервера"""
    try:
        async for message in websocket:
            # Очищаем текущую строку ввода, выводим сообщение и возвращаем приглашение к вводу
            print(f"\r{message}")
            print("Вы: ", end="", flush=True)
    except websockets.ConnectionClosed:
        print("\n[КЛИЕНТ] Соединение с сервером разорвано.")

async def send_messages(websocket, name):
    """Функция для отправки сообщений на сервер"""
    # Переводим стандартный ввод в асинхронный режим для работы в консоли
    loop = asyncio.get_event_loop()
    
    while True:
        # Асинхронно ждем ввода текста от пользователя
        message = await loop.run_in_executor(None, input, "Вы: ")
        if message.strip():
            full_message = f"{name}: {message}"
            try:
                await websocket.send(full_message)
            except websockets.ConnectionClosed:
                break

async def main():
    uri = "ws://localhost:8765"
    
    name = input("Введите ваше имя для чата: ")
    print(f"[КЛИЕНТ] Подключение к серверу {uri}...")
    
    try:
        async with websockets.connect(uri) as websocket:
            print("[КЛИЕНТ] Успешно подключено! Начните вводить сообщения.")
            
            # Запускаем параллельно две задачи: прием и отправку
            await asyncio.gather(
                receive_messages(websocket),
                send_messages(websocket, name)
            )
    except ConnectionRefusedError:
        print("[КЛИЕНТ] Не удалось подключиться. Убедитесь, что сервер запущен.")

if __name__ == "__main__":
    # Исправление для корректной работы input() в asyncio на Windows
    if sys.platform == 'win32':
        asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
        
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("\n[КЛИЕНТ] Выход из чата.")

