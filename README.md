import tkinter as tk
from tkinter import scrolledtext
import threading
import time
import requests


# ============================
# TABELA MONEY/SEC
# ============================
money_table = {
    "La Grande Combinasion": 10_000_000,
    "Mariachi Corazoni": 12_500_000,
    "Swag Soda": 13_000_000,
    "Nuclearo Dinossauro": 15_000_000,
    "Los Combinasionas": 15_000_000,
    "Fishino Clownino": 15_500_000,
    "Tacorita Bicicleta": 16_500_000,
    "Las Sis": 17_500_000,
    "Los Planitos": 18_500_000,
    "Los Spooky Combinasionas": 20_000_000,
    "Los Hotspotsitos": 20_000_000,
    "Money Money Puggy": 21_000_000,
    "Los Mobilis": 22_000_000,
    "Los 67": 22_500_000,
    "Celularcini Viciosini": 22_500_000,
    "La Extinct Grande": 23_500_000,
    "Los Bros": 24_000_000,
    "La Spooky Grande": 24_500_000,
    "Chillin Chili": 25_000_000,
    "Chipso and Queso": 25_000_000,
    "Mieteteira Bicicleteira": 26_000_000,
    "Tralaledon": 27_500_000,
    "Gobblino Uniciclino": 27_500_000,
    "W or L": 30_000_000,
    "Los Puggies": 30_000_000,
    "La Jolly Grande": 30_000_000,
    "Esok Sekolah": 30_000_000,
    "Los Primos": 31_000_000,
    "Eviledon": 31_500_000,
    "Los Tacoritas": 32_000_000,
    "Tang Tang Keletang": 33_500_000,
    "La Taco Combinasion": 35_000_000,
    "Ketupat Kepat": 35_000_000,
    "Tictac Sahur": 37_500_000,
    "La Supreme Combinasion": 40_000_000,
    "Orcaledon": 40_000_000,
    "Ketchuru and Musturu": 42_500_000,
    "Lavadorito Spinito": 45_000_000,
    "Garama and Madundung": 50_000_000,
    "Spaghetti Tualetti": 60_000_000,
    "Los Spaghettis": 70_000_000,
    "Spooky and Pumpky": 80_000_000,
    "Fragrama and Chocrama": 100_000_000,
    "La Casa Boo": 100_000_000,
    "La Secret Combinasion": 125_000_000,
    "Burguro and Fryuro": 150_000_000,
    "Cooki and Milki": 155_000_000,
    "Capitano Moby": 160_000_000,
    "Headless Horseman": 175_000_000,
    "Dragon Cannelloni": 250_000_000,
    "Strawberry Elephant": 500_000_000,
    "Meowl": 400_000_000
}


running = False  # usado para parar o loop


# ============================
# ENVIAR EMBED
# ============================
def send_embed(webhook, players, server_id, job_id, money_sec):
    embed = {
        "title": "Finder Japa",
        "color": 0x00ff00,
        "fields": [
            {"name": "Players", "value": players},
            {"name": "Server ID", "value": server_id},
            {"name": "Money/sec", "value": f"${money_sec:,}/s"},
            {"name": "Job ID", "value": job_id},
        ]
    }
    requests.post(webhook, json={"embeds": [embed]})


# ============================
# MONITOR PRINCIPAL
# ============================
def monitor(webhook, place_id, log_box):
    global running

    log_box.insert(tk.END, "[LOG] Monitor iniciado...\n")
    MIN_PLAYERS = 7
    MAX_PLAYERS = 8

    while running:
        try:
            url = f"https://games.roblox.com/v1/games/{place_id}/servers/Public?sortOrder=Asc&limit=100"
            response = requests.get(url)

            if response.status_code == 429:
                log_box.insert(tk.END, "[ERRO] 429 - Too Many Requests\n")
                log_box.see(tk.END)
                time.sleep(10)
                continue

            data = response.json()
            found = False

            for server in data.get("data", []):
                players = server["playing"]

                if MIN_PLAYERS <= players <= MAX_PLAYERS:
                    found = True

                    job_id = server["id"]
                    server_id = server["id"]

                    names = [p.get("username", "") for p in server.get("playerList", [])]
                    money_sec = 0

                    for n in names:
                        if n in money_table:
                            money_sec = money_table[n]
                            break

                    send_embed(webhook, f"{players}/{server['maxPlayers']}", server_id, job_id, money_sec)

                    log_box.insert(tk.END, f"[FOUND] Server enviado! Players {players}\n")
                    log_box.see(tk.END)

            if not found:
                log_box.insert(tk.END, "[LOG] Nenhum servidor encontrado...\n")
                log_box.see(tk.END)

            time.sleep(10)

        except Exception as e:
            log_box.insert(tk.END, f"[ERRO] {e}\n")
            log_box.see(tk.END)
            time.sleep(10)


# ============================
# START & STOP
# ============================
def start_monitor():
    global running
    if running:
        return

    running = True

    webhook = webhook_entry.get()
    place_id = place_entry.get()

    t = threading.Thread(target=monitor, args=(webhook, place_id, log_box))
    t.daemon = True
    t.start()


def stop_monitor():
    global running
    running = False
    log_box.insert(tk.END, "[LOG] Monitor parado.\n")
    log_box.see(tk.END)


# ============================
# GUI
# ============================
root = tk.Tk()
root.title("Finder Japa | GUI Monitor")
root.geometry("500x550")

tk.Label(root, text="Webhook:").pack()
webhook_entry = tk.Entry(root, width=60)
webhook_entry.pack()

tk.Label(root, text="Place ID:").pack()
place_entry = tk.Entry(root, width=60)
place_entry.insert(0, "109983668079237")
place_entry.pack()

start_btn = tk.Button(root, text="START", bg="green", fg="white", command=start_monitor)
start_btn.pack(pady=5)

stop_btn = tk.Button(root, text="STOP", bg="red", fg="white", command=stop_monitor)
stop_btn.pack(pady=5)

log_box = scrolledtext.ScrolledText(root, width=60, height=25)
log_box.pack()

root.mainloop()
