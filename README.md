import tkinter as tk
import random
import time
import math

class WheelOfFortune:
    def __init__(self, master):
        self.master = master
        self.master.title("Çark Oyunu")
        self.master.configure(bg="lightblue")  # Arka plan rengi
        
        # Pastel renkler listesi
        self.colors = [
            '#FFB3BA', '#FFDFDA', '#FFFFBE', '#BAFFC9', '#BAE1FF',
            '#C9C9FF', '#FFC9DE', '#FFFFD1', '#B3FFD9', '#FFD1B3'
        ]
        
        self.name_entries = []
        self.create_name_input_fields()
        
        self.canvas = tk.Canvas(master, width=600, height=600, bg="lightblue")
        self.canvas.pack()

        self.angle = 0
        self.speed = 10.0  # Daha hızlı dönme hızı
        self.spin_duration = 8  # 8 saniye boyunca dönecek
        
        self.spin_button = tk.Button(master, text="Çarkı Çevir", command=self.prepare_wheel, bg="orange", font=("Arial", 14, "bold"))
        self.spin_button.pack()

        self.names = []

    def create_name_input_fields(self):
        frame = tk.Frame(self.master, bg="lightblue")
        frame.pack()

        for i in range(10):
            label = tk.Label(frame, text=f"Kişi {i+1}:", bg="lightblue", font=("Arial", 12, "bold"))
            label.grid(row=i, column=0)
            entry = tk.Entry(frame, bg=self.colors[i], font=("Arial", 12, "bold"))
            entry.grid(row=i, column=1)
            self.name_entries.append(entry)

    def prepare_wheel(self):
        # Kişi listesini ve renkleri oluştur
        self.names = [entry.get().strip() for entry in self.name_entries if entry.get().strip() != ""]
        if len(self.names) == 0:
            return

        # Renklerle isimlerin eşleştirilmesi
        self.active_colors = [self.colors[i] for i in range(len(self.names))]
        
        # Çarkı çevir
        self.spin_wheel()

    def spin_wheel(self):
        self.start_time = time.time()
        self.end_time = self.start_time + self.spin_duration
        self.update_wheel()
    
    def update_wheel(self):
        current_time = time.time()
        if current_time < self.end_time:
            self.angle += self.speed
            self.canvas.delete("all")
            self.draw_wheel()
            self.master.after(50, self.update_wheel)
        else:
            self.show_result()
    
    def draw_wheel(self):
        # Çarkı çiz
        self.canvas.create_oval(100, 100, 500, 500, width=5, outline="black", fill="white")  # Dış çemberi kalınlaştır
        angle_step = 360 / len(self.names)
        for i, name in enumerate(self.names):
            start_angle = self.angle + i * angle_step
            end_angle = start_angle + angle_step
            color = self.active_colors[i]
            self.canvas.create_arc(100, 100, 500, 500, start=start_angle, extent=angle_step, fill=color, outline="black", width=2)
            text_angle = (start_angle + end_angle) / 2
            x = 300 + 200 * math.cos(math.radians(text_angle))
            y = 300 - 200 * math.sin(math.radians(text_angle))
            self.canvas.create_text(x, y, text=name, fill='black', font=("Arial", 16, "bold"))  # Font boyutunu artır

    def show_result(self):
        # Çarktan elenen kişiyi belirle
        winner_angle = (self.angle + 90) % 360  # Çarkın üst kısmındaki açı
        angle_step = 360 / len(self.names)
        eliminated_index = int(winner_angle / angle_step) % len(self.names)
        eliminated_name = self.names[eliminated_index]
        
        # Elenen kişiyi listeden bul ve üstünü çiz
        for entry in self.name_entries:
            if entry.get().strip() == eliminated_name:
                entry.config(font=("Arial", 12, "bold", "overstrike"))  # Elenen kişinin fontunu kalın yap
                break
        
        # Çarktan ismi ve rengini çıkar
        self.names.pop(eliminated_index)
        self.active_colors.pop(eliminated_index)
        if len(self.names) > 1:
            self.master.after(2000, self.spin_wheel)  # 2 saniye sonra yeniden çevir
        else:
            winner_name = self.names[0]
            self.winner_label = tk.Label(self.master, text=f"{winner_name.upper()} KAZANDI!", font=("Arial", 24, "bold"), bg="yellow")
            self.winner_label.pack()

if __name__ == "__main__":
    root = tk.Tk()
    app = WheelOfFortune(root)
    root.mainloop()
