import tkinter as tk
from tkinter import ttk
import serial
import serial.tools.list_ports
import time
import threading
from matplotlib.figure import Figure
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg

# Fungsi untuk membaca data serial dari Arduino
def read_serial():
    while True:
        if ser and ser.is_open and ser.in_waiting > 0:
            line = ser.readline().decode('utf-8').strip()
            if line:
                try:
                    data = list(map(float, line.split(',')))
                    update_graph(data)
                except ValueError:
                    pass

# Fungsi untuk memperbarui grafik
def update_graph(data):
    global time_list, rpm_list, start_time

    current_time = time.time() - start_time
    actual_rpm = data[0]

    # Periksa apakah set_rpm_var memiliki nilai yang valid
    try:
        target_rpm = float(set_rpm_var.get())
    except ValueError:
        target_rpm = 0.0  # Jika tidak valid, set target_rpm ke 0.0

    time_list.append(current_time)
    rpm_list.append(actual_rpm)

    line.set_data(time_list, rpm_list)
    ax.relim()
    ax.autoscale_view()
    rpm_display.set(f"{actual_rpm:.2f}")

    # Perhitungan overshoot dan lainnya
    error = abs(target_rpm - actual_rpm)
    error_display.set(f"Error: {error:.2f}")

    if len(rpm_list) > 1:
        peak_rpm = max(rpm_list)
        if target_rpm != 0:
            overshoot = ((peak_rpm - target_rpm) / target_rpm) * 100 if peak_rpm > target_rpm else 0
        else:
            overshoot = 0
        overshoot_display.set(f"Overshoot: {overshoot:.2f}%")

        peak_time = time_list[rpm_list.index(peak_rpm)]
        peak_display.set(f"Peak Time: {peak_time:.2f}s")

        rise_time = next((t for r, t in zip(rpm_list, time_list) if r >= 0.9 * target_rpm), None)
        rise_display.set(f"Rise Time: {rise_time:.2f}s" if rise_time else "Rise Time: N/A")

        settling_time = next((t for r, t in zip(reversed(rpm_list), reversed(time_list)) if abs(r - target_rpm) <= 0.05 * target_rpm), None)
        settling_display.set(f"Settling Time: {settling_time:.2f}s" if settling_time else "Settling Time: N/A")

    canvas.draw()

# Fungsi untuk mengirim parameter PID ke Arduino
def send_parameters():
    if ser and ser.is_open:
        try:
            kp = kp_var.get()
            ki = ki_var.get()
            kd = kd_var.get()
            setpoint = set_rpm_var.get()
            if kp and ki and kd and setpoint:
                ser.write(f"{kp},{ki},{kd},{setpoint}\n".encode('utf-8'))
        except Exception as e:
            print(f"Error sending parameters: {e}")

# Fungsi untuk menghentikan atau menjalankan motor
def toggle_motor():
    global motor_running
    if ser and ser.is_open:
        motor_running = not motor_running
        ser.write(b"START\n" if motor_running else b"STOP\n")
        motor_status.set("Running" if motor_running else "Stopped")

# Fungsi untuk memulai komunikasi serial
def connect_serial():
    global ser
    port = com_var.get()
    try:
        ser = serial.Serial(port, 9600, timeout=1)
        time.sleep(2)
        serial_status.set(f"Connected to {port}")
        start_reading_thread()
    except Exception as e:
        serial_status.set(f"Error: {e}")

def start_reading_thread():
    serial_thread = threading.Thread(target=read_serial, daemon=True)
    serial_thread.start()

def refresh_com_ports():
    ports = [port.device for port in serial.tools.list_ports.comports()]
    com_menu["values"] = ports
    com_var.set(ports[0] if ports else "No COM ports")

# Fungsi untuk mereset grafik
def reset_graph():
    global time_list, rpm_list, start_time
    time_list.clear()
    rpm_list.clear()
    line.set_data([], [])
    start_time = time.time()  # Reset the time counter
    canvas.draw()

# Data grafik dan status
start_time = time.time()
time_list = []
rpm_list = []
max_points = 100

ser = None
motor_running = False

# GUI Tkinter
root = tk.Tk()
root.title("PID Motor Control")
root.geometry("1200x800")

# Frame pemilihan port COM
com_frame = tk.Frame(root)
com_frame.pack(side=tk.TOP, fill=tk.X, padx=10, pady=5)

tk.Label(com_frame, text="Select COM Port:").pack(side=tk.LEFT, padx=5)
com_var = tk.StringVar()
com_menu = ttk.Combobox(com_frame, textvariable=com_var, state="readonly")
com_menu.pack(side=tk.LEFT, padx=5)
refresh_com_ports()
tk.Button(com_frame, text="Refresh", command=refresh_com_ports).pack(side=tk.LEFT, padx=5)
tk.Button(com_frame, text="Connect", command=connect_serial).pack(side=tk.LEFT, padx=5)

serial_status = tk.StringVar(value="No connection")
tk.Label(com_frame, textvariable=serial_status, fg="blue").pack(side=tk.LEFT, padx=10)

# Frame parameter PID dan informasi motor
param_frame = tk.Frame(root)
param_frame.pack(side=tk.LEFT, padx=10, pady=(10, 20), anchor="n")  # Menyatukan PID Parameters dan Motor Information

tk.Label(param_frame, text="PID Parameters", font=("Arial", 14, "bold")).pack(pady=(0, 5))

kp_var = tk.DoubleVar(value=1.0)
ki_var = tk.DoubleVar(value=0.5)
kd_var = tk.DoubleVar(value=1.0)
set_rpm_var = tk.DoubleVar(value=40.0)

tk.Label(param_frame, text="Kp:").pack()
tk.Entry(param_frame, textvariable=kp_var).pack()

tk.Label(param_frame, text="Ki:").pack()
tk.Entry(param_frame, textvariable=ki_var).pack()

tk.Label(param_frame, text="Kd:").pack()
tk.Entry(param_frame, textvariable=kd_var).pack()

tk.Label(param_frame, text="Target RPM:").pack()
tk.Entry(param_frame, textvariable=set_rpm_var).pack()

tk.Button(param_frame, text="Send Parameters", command=send_parameters).pack(pady=5)
tk.Button(param_frame, text="Start/Stop Motor", command=toggle_motor).pack(pady=5)
motor_status = tk.StringVar(value="Stopped")
tk.Label(param_frame, textvariable=motor_status).pack(pady=5)

tk.Button(param_frame, text="Reset Graph", command=reset_graph).pack(pady=5)

tk.Label(param_frame, text="Motor Information", font=("Arial", 14, "bold")).pack(pady=(20, 5))  # Memisahkan dari PID Parameters

rpm_display = tk.StringVar(value="0.00")
error_display = tk.StringVar(value="Error: 0.00")
overshoot_display = tk.StringVar(value="Overshoot: 0.00%")
peak_display = tk.StringVar(value="Peak Time: 0.00s")
rise_display = tk.StringVar(value="Rise Time: 0.00s")
settling_display = tk.StringVar(value="Settling Time: 0.00s")

tk.Label(param_frame, text="RPM:").pack(anchor="w")
tk.Entry(param_frame, textvariable=rpm_display, state="readonly").pack(fill=tk.X, padx=5)

tk.Label(param_frame, text="Error:").pack(anchor="w")
tk.Entry(param_frame, textvariable=error_display, state="readonly").pack(fill=tk.X, padx=5)

tk.Label(param_frame, text="Overshoot:").pack(anchor="w")
tk.Entry(param_frame, textvariable=overshoot_display, state="readonly").pack(fill=tk.X, padx=5)

tk.Label(param_frame, text="Peak Time:").pack(anchor="w")
tk.Entry(param_frame, textvariable=peak_display, state="readonly").pack(fill=tk.X, padx=5)

tk.Label(param_frame, text="Rise Time:").pack(anchor="w")
tk.Entry(param_frame, textvariable=rise_display, state="readonly").pack(fill=tk.X, padx=5)

tk.Label(param_frame, text="Settling Time:").pack(anchor="w")
tk.Entry(param_frame, textvariable=settling_display, state="readonly").pack(fill=tk.X, padx=5)

# Frame untuk grafik
graph_frame = tk.Frame(root)
graph_frame.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True, padx=10, pady=(0, 5))

fig = Figure(figsize=(8, 6), dpi=100)
ax = fig.add_subplot(111)
ax.set_title("Kecepatan Motor dalam RPM")
ax.set_xlabel("Waktu (s)")
ax.set_ylabel("Nilai RPM")
line, = ax.plot([], [], label="Nilai Aktual RPM")
ax.legend()

canvas = FigureCanvasTkAgg(fig, master=graph_frame)
canvas.get_tk_widget().pack(fill=tk.BOTH, expand=True)
canvas.draw()

root.mainloop()
