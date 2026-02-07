# Codigo-Fuente-Programa-GRUPO23

mport tkinter as tk
from tkinter import ttk, messagebox
import numpy as np
import pandas as pd
from datetime import datetime
import os
import math
import hashlib
import json
import webbrowser
import subprocess

# ============================================================================
# SISTEMA DE ACCESO DUAL - MODO ALUMNO Y MODO DOCENTE
# ============================================================================

class SistemaAcceso:
    def __init__(self):
        self.config_file = "config_ecollajta.json"
        self.usuario_actual = None
        self.modo_actual = "alumno"
        self.cargar_configuracion()
    
    def cargar_configuracion(self):
        config_default = {
            "docente_usuario": "admin",
            "docente_password": self.encriptar_sha256("admin123"),
            "ultimo_modo": "alumno",
            "version": "1.0",
            "actualizacion_url": "https://github.com/tu_usuario/ecollajta"
        }
        
        try:
            if os.path.exists(self.config_file):
                with open(self.config_file, 'r') as f:
                    self.config = json.load(f)
                    for key in config_default:
                        if key not in self.config:
                            self.config[key] = config_default[key]
            else:
                self.config = config_default
                self.guardar_configuracion()
        except:
            self.config = config_default
    
    def guardar_configuracion(self):
        try:
            with open(self.config_file, 'w') as f:
                json.dump(self.config, f, indent=4)
        except:
            pass
    
    def encriptar_sha256(self, texto):
        return hashlib.sha256(texto.encode()).hexdigest()
    
    def verificar_docente(self, usuario, password):
        usuario_correcto = self.config["docente_usuario"]
        password_correcta = self.config["docente_password"]
        password_hash = self.encriptar_sha256(password)
        
        if usuario == usuario_correcto and password_hash == password_correcta:
            self.usuario_actual = usuario
            self.modo_actual = "docente"
            self.config["ultimo_modo"] = "docente"
            self.guardar_configuracion()
            return True
        return False
    
    def cambiar_password_docente(self, password_actual, password_nueva):
        actual_hash = self.encriptar_sha256(password_actual)
        if actual_hash == self.config["docente_password"]:
            self.config["docente_password"] = self.encriptar_sha256(password_nueva)
            self.guardar_configuracion()
            return True, "Contraseña cambiada exitosamente"
        return False, "Contraseña actual incorrecta"

class VentanaSeleccionModo:
    def __init__(self, root, sistema_acceso, callback_inicio):
        self.root = root
        self.sistema = sistema_acceso
        self.callback_inicio = callback_inicio
        
        self.root.title("🏫 CALCULADORA ECOLLAJTA - GRUPO 23")
        self.root.geometry("500x400")
        self.root.configure(bg='#1a1a2e')
        self.center_window()
        
        self.root.protocol("WM_DELETE_WINDOW", self.salir_aplicacion)
        self.crear_interfaz()
    
    def center_window(self):
        self.root.update_idletasks()
        width = 500
        height = 400
        x = (self.root.winfo_screenwidth() // 2) - (width // 2)
        y = (self.root.winfo_screenheight() // 2) - (height // 2)
        self.root.geometry(f'{width}x{height}+{x}+{y}')
    
    def crear_interfaz(self):
        # Logo/Title
        title_frame = tk.Frame(self.root, bg='#1a1a2e')
        title_frame.pack(pady=20)
        
        tk.Label(title_frame, 
                text="🏭 CALCULADORA ECOLLAJTA",
                font=("Arial", 22, "bold"),
                bg='#1a1a2e',
                fg='#00b4d8').pack()
        
        tk.Label(title_frame,
                text="Grupo 23 - Cadenas de Markov + Algoritmo Voraz",
                font=("Arial", 11),
                bg='#1a1a2e',
                fg='#caf0f8').pack(pady=5)
        
        tk.Label(title_frame,
                text="SELECCIONE MODO DE ACCESO",
                font=("Arial", 12, "bold"),
                bg='#1a1a2e',
                fg='#f72585').pack(pady=10)
        
        # Frame principal
        main_frame = tk.Frame(self.root, bg='#1a1a2e')
        main_frame.pack(pady=10, padx=20, fill=tk.BOTH, expand=True)
        
        # ===== MODO ALUMNO =====
        alumno_frame = tk.Frame(main_frame, bg='#16213e', relief=tk.RAISED, bd=2)
        alumno_frame.pack(fill=tk.BOTH, expand=True, pady=(0, 10))
        
        # Icono y título
        icono_alumno = tk.Frame(alumno_frame, bg='#4361ee', height=60)
        icono_alumno.pack(fill=tk.X)
        
        tk.Label(icono_alumno, text="👨‍🎓", font=("Arial", 24),
                bg='#4361ee', fg='white').pack(pady=10)
        
        tk.Label(alumno_frame, text="MODO ALUMNO",
                font=("Arial", 14, "bold"),
                bg='#16213e', fg='white').pack(pady=(10, 5))

        tk.Button(alumno_frame, text="🎓 ENTRAR COMO ALUMNO",
                 command=self.entrar_como_alumno,
                 font=("Arial", 11, "bold"),
                 bg='#4361ee', fg='white',
                 padx=20, pady=10,
                 cursor='hand2').pack(pady=15)
        
        # ===== MODO DOCENTE =====
        docente_frame = tk.Frame(main_frame, bg='#16213e', relief=tk.RAISED, bd=2)
        docente_frame.pack(fill=tk.BOTH, expand=True)
        
        # Icono y título
        icono_docente = tk.Frame(docente_frame, bg='#7209b7', height=60)
        icono_docente.pack(fill=tk.X)
        
        tk.Label(icono_docente, text="👨‍🏫", font=("Arial", 24),
                bg='#7209b7', fg='white').pack(pady=10)
        
        tk.Label(docente_frame, text="MODO DOCENTE",
                font=("Arial", 14, "bold"),
                bg='#16213e', fg='white').pack(pady=(10, 5))

        tk.Button(docente_frame, text="🔐 ENTRAR COMO DOCENTE",
                 command=self.entrar_como_docente,
                 font=("Arial", 11, "bold"),
                 bg='#7209b7', fg='white',
                 padx=20, pady=10,
                 cursor='hand2').pack(pady=15)
        
        # Info
        info_frame = tk.Frame(self.root, bg='#1a1a2e')
        info_frame.pack(pady=10)
        
        tk.Label(info_frame,
                text="Docente: Usa credenciales proporcionadas por el Grupo 23",
                font=("Arial", 8, "italic"),
                bg='#1a1a2e',
                fg='#8d99ae').pack()
        
        tk.Label(info_frame,
                text="© Grupo 23 - Versión " + self.sistema.config["version"],
                font=("Arial", 8),
                bg='#1a1a2e',
                fg='#8d99ae').pack(pady=5)
    
    def entrar_como_alumno(self):
        self.sistema.modo_actual = "alumno"
        self.sistema.usuario_actual = None
        self.sistema.config["ultimo_modo"] = "alumno"
        self.sistema.guardar_configuracion()
        self.root.destroy()
        self.callback_inicio(self.sistema.modo_actual)
    
    def entrar_como_docente(self):
        self.mostrar_login_docente()
    
    def mostrar_login_docente(self):
        login_win = tk.Toplevel(self.root)
        login_win.title("🔐 Acceso Docente")
        login_win.geometry("350x250")
        login_win.configure(bg='#1a1a2e')
        login_win.transient(self.root)
        login_win.grab_set()
        
        x = self.root.winfo_x() + (self.root.winfo_width() // 2) - (350 // 2)
        y = self.root.winfo_y() + (self.root.winfo_height() // 2) - (250 // 2)
        login_win.geometry(f"350x250+{x}+{y}")
        
        tk.Label(login_win, text="ACCESO DOCENTE",
                font=("Arial", 16, "bold"),
                bg='#1a1a2e',
                fg='#f72585').pack(pady=15)
        
        frame = tk.Frame(login_win, bg='#16213e', padx=20, pady=20)
        frame.pack(pady=10, padx=20, fill=tk.BOTH, expand=True)
        
        tk.Label(frame, text="Usuario:",
                font=("Arial", 10, "bold"),
                bg='#16213e',
                fg='#caf0f8').pack(anchor='w', pady=(0, 5))
        
        usuario_var = tk.StringVar(value="admin")
        tk.Entry(frame, textvariable=usuario_var,
                font=("Arial", 11),
                bg='#0f3460', fg='white').pack(fill=tk.X, pady=(0, 15))
        
        tk.Label(frame, text="Contraseña:",
                font=("Arial", 10, "bold"),
                bg='#16213e',
                fg='#caf0f8').pack(anchor='w', pady=(0, 5))
        
        password_var = tk.StringVar()
        password_entry = tk.Entry(frame, textvariable=password_var,
                                 font=("Arial", 11),
                                 bg='#0f3460', fg='white',
                                 show="•")
        password_entry.pack(fill=tk.X, pady=(0, 15))
        password_entry.focus()
        
        def verificar():
            usuario = usuario_var.get().strip()
            password = password_var.get()
            
            if not usuario or not password:
                messagebox.showwarning("Advertencia", "Complete todos los campos")
                return
            
            if self.sistema.verificar_docente(usuario, password):
                messagebox.showinfo("Éxito", f"¡Bienvenido, {usuario}!")
                login_win.destroy()
                self.root.destroy()
                self.callback_inicio(self.sistema.modo_actual)
            else:
                messagebox.showerror("Error", "Credenciales incorrectas")
                password_var.set("")
                password_entry.focus()
        
        btn_frame = tk.Frame(frame, bg='#16213e')
        btn_frame.pack(pady=20)
        
        tk.Button(btn_frame, text="🔐 Ingresar",
                 command=verificar,
                 bg='#7209b7', fg='white',
                 font=("Arial", 10, "bold"),
                 padx=20).pack(side=tk.LEFT, padx=5)
        
        tk.Button(btn_frame, text="↩️ Volver",
                 command=login_win.destroy,
                 bg='#495057', fg='white',
                 font=("Arial", 10),
                 padx=20).pack(side=tk.LEFT, padx=5)
        
        login_win.bind('<Return>', lambda e: verificar())
    
    def salir_aplicacion(self):
        if messagebox.askyesno("Salir", "¿Está seguro que desea salir?"):
            self.root.destroy()

class CalculadoraEcoLlajta:
    def __init__(self, root, modo="alumno"):
        self.root = root
        self.modo = modo  # "alumno" o "docente"
        
        # Configurar título y colores según modo
        if modo == "docente":
            self.root.title("CALCULADORA ECOLLAJTA - Grupo 23 (MODO DOCENTE)")
            self.root.configure(bg='#2C3E50')
            self.color_fondo = '#2C3E50'
        else:
            self.root.title("CALCULADORA ECOLLAJTA - Grupo 23 (MODO ALUMNO)")
            self.root.configure(bg='#34495E')
            self.color_fondo = '#34495E'
        
        self.root.geometry("1200x700")
        
        # Meta personalizable por usuario (inicialmente 4000g = 22.22 macetas)
        self.meta_usuario = 4000.0
        self.gramos_por_maceta = 180.0
        
        # Centrar ventana
        self.center_window()

        self.matriz_p = np.array([
            [0.1, 0.6, 0.3],
            [0.2, 0.5, 0.3],
            [0.4, 0.4, 0.2]
        ])
        
        self.puntos = pd.DataFrame({
            'Nodo': [1, 2, 3, 4, 5, 6, 7, 8],
            'Punto': ['Restaurante "A"', 'Pensión Central', 'Casa (Disperso)', 
                     'Pastelería', 'Cafetería Univ.', 'Mercado Local', 'Sky Box', 'Ramen'],
            'Cantidad_g': [1200, 800, 400, 2000, 900, 1800, 600, 1500],
            'Tiempo_min': [15, 20, 25, 10, 12, 25, 10, 18],
            'Capacidad_Max_g': [1500, 1000, 600, 2500, 1200, 2000, 800, 2000],
            'Estado': ['Medio', 'Medio', 'Vacío', 'Lleno', 'Medio', 'Lleno', 'Vacío', 'Medio']
        })
        
        # Orden original de los nodos (para mantener en la interfaz)
        self.orden_original_nodos = list(self.puntos['Nodo'])
        
        self.dias_transcurridos = 1
        self.dia_sugerido = 3.333333333333333
        
        # Variables para las restricciones
        self.total_carga_prioritaria = 0.0
        self.total_tiempo_prioritario = 0.0
        self.nodos_seleccionados = []
        self.meta_cumplida = False
        self.tiempo_cumplido = False
        
        # NUEVAS VARIABLES AÑADIDAS
        self.carga_total_sistema = 0.0
        self.deficit_meta = 0.0
        self.nodos_expansion_necesarios = 0
        
        # Variables para macetas
        self.macetas_deseadas = int(self.meta_usuario / self.gramos_por_maceta)
        self.deficit_macetas = 0.0
        
        # Calcular todo al inicio
        self.calcular_todo()
        
        # Agregar indicador de modo
        self.create_modo_indicator()
        
        # Crear interfaz
        self.create_widgets()
    
    def center_window(self):
        """Centra la ventana en la pantalla"""
        self.root.update_idletasks()
        width = 1200
        height = 700
        x = (self.root.winfo_screenwidth() // 2) - (width // 2)
        y = (self.root.winfo_screenheight() // 2) - (height // 2)
        self.root.geometry(f'{width}x{height}+{x}+{y}')
    
    def create_modo_indicator(self):
        """Crea indicador del modo actual en la ventana"""
        self.modo_frame = tk.Frame(self.root, bg='#1a1a2e')
        self.modo_frame.pack(side=tk.TOP, fill=tk.X, pady=(0, 5))
        
        if self.modo == "docente":
            texto = "👑 MODO DOCENTE - ACCESO COMPLETO"
            color = '#9B59B6'
            # Agregar botón de administración
            tk.Button(self.modo_frame, text="⚙️ Panel Admin",
                     command=self.mostrar_panel_admin,
                     bg='#E74C3C', fg='white',
                     font=("Arial", 9)).pack(side=tk.RIGHT, padx=10)
        else:
            texto = "👨‍🎓 MODO ALUMNO - ACCESO BÁSICO"
            color = '#3498DB'
        
        tk.Label(self.modo_frame, text=texto,
                font=("Arial", 10, "bold"),
                bg=color, fg='white',
                padx=20, pady=5).pack(fill=tk.X)
    
    def mostrar_panel_admin(self):
        """Muestra panel de administración solo en modo docente"""
        if self.modo != "docente":
            messagebox.showwarning("Acceso Restringido", 
                                  "Esta función solo está disponible en modo Docente")
            return
        
        admin_win = tk.Toplevel(self.root)
        admin_win.title("👑 Panel de Administración - Docente")
        admin_win.geometry("500x400")
        admin_win.configure(bg='#2C3E50')
        
        tk.Label(admin_win, text="PANEL DE ADMINISTRACIÓN DOCENTE",
                font=("Arial", 16, "bold"),
                bg='#2C3E50', fg='#F1C40F').pack(pady=15)
        
        frame = tk.Frame(admin_win, bg='#34495E', padx=20, pady=20)
        frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=10)
        
        # Funciones para docente
        funciones = [
            ("📁 Abrir Carpeta del Programa", self.abrir_carpeta_programa),
            ("👁️ Ver Código Fuente Completo", self.mostrar_codigo_completo),
            ("🔄 Cambiar Contraseña Docente", self.cambiar_password_docente),
            ("🌐 Ver Repositorio en GitHub", self.abrir_github),
            ("📊 Estadísticas de Uso", self.mostrar_estadisticas),
            ("🔒 Cerrar Sesión Docente", self.cerrar_sesion_docente)
        ]
        
        for texto, comando in funciones:
            tk.Button(frame, text=texto,
                     command=comando,
                     bg='#8E44AD', fg='white',
                     font=("Arial", 10),
                     width=30,
                     pady=8).pack(pady=5)
    
    def abrir_carpeta_programa(self):
        """Abre la carpeta donde está el programa"""
        try:
            ruta = os.path.dirname(os.path.abspath(__file__))
            if os.name == 'nt':  # Windows
                subprocess.run(['explorer', ruta])
            elif os.name == 'posix':  # Linux/Mac
                subprocess.run(['xdg-open', ruta])
            messagebox.showinfo("Éxito", f"Carpeta abierta:\n{ruta}")
        except:
            messagebox.showerror("Error", "No se pudo abrir la carpeta")
    
    def mostrar_codigo_completo(self):
        """Muestra el código fuente completo"""
        try:
            with open(__file__, 'r', encoding='utf-8') as f:
                codigo = f.read()
            
            codigo_win = tk.Toplevel(self.root)
            codigo_win.title("📝 Código Fuente Completo")
            codigo_win.geometry("800x600")
            
            # Text widget con scroll
            text_frame = tk.Frame(codigo_win)
            text_frame.pack(fill=tk.BOTH, expand=True, padx=5, pady=5)
            
            scrollbar = tk.Scrollbar(text_frame)
            scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
            
            text_widget = tk.Text(text_frame, wrap=tk.WORD,
                                 yscrollcommand=scrollbar.set,
                                 font=("Consolas", 10),
                                 bg='#1e1e1e', fg='#d4d4d4')
            text_widget.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
            scrollbar.config(command=text_widget.yview)
            
            # Insertar código con colores básicos
            text_widget.insert(1.0, codigo)
            text_widget.config(state=tk.DISABLED)
            
            # Botón para copiar
            tk.Button(codigo_win, text="📋 Copiar Código",
                     command=lambda: self.copiar_al_portapapeles(codigo),
                     bg='#27AE60', fg='white').pack(pady=10)
            
        except Exception as e:
            messagebox.showerror("Error", f"No se pudo leer el código: {str(e)}")
    
    def copiar_al_portapapeles(self, texto):
        """Copia texto al portapapeles"""
        self.root.clipboard_clear()
        self.root.clipboard_append(texto)
        messagebox.showinfo("Éxito", "Código copiado al portapapeles")
    
    def cambiar_password_docente(self):
        """Permite cambiar la contraseña del docente"""
        sistema = SistemaAcceso()
        
        pass_win = tk.Toplevel(self.root)
        pass_win.title("🔐 Cambiar Contraseña Docente")
        pass_win.geometry("350x250")
        pass_win.configure(bg='#2C3E50')
        
        tk.Label(pass_win, text="CAMBIAR CONTRASEÑA DOCENTE",
                font=("Arial", 12, "bold"),
                bg='#2C3E50', fg='white').pack(pady=10)
        
        frame = tk.Frame(pass_win, bg='#34495E', padx=20, pady=20)
        frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=10)
        
        tk.Label(frame, text="Contraseña Actual:",
                font=("Arial", 10),
                bg='#34495E', fg='white').pack(anchor='w')
        
        pass_actual_var = tk.StringVar()
        tk.Entry(frame, textvariable=pass_actual_var,
                font=("Arial", 10),
                bg='white', fg='black',
                show="•").pack(fill=tk.X, pady=(0, 10))
        
        tk.Label(frame, text="Nueva Contraseña:",
                font=("Arial", 10),
                bg='#34495E', fg='white').pack(anchor='w')
        
        pass_nueva_var = tk.StringVar()
        tk.Entry(frame, textvariable=pass_nueva_var,
                font=("Arial", 10),
                bg='white', fg='black',
                show="•").pack(fill=tk.X, pady=(0, 10))
        
        tk.Label(frame, text="Confirmar Nueva:",
                font=("Arial", 10),
                bg='#34495E', fg='white').pack(anchor='w')
        
        pass_confirmar_var = tk.StringVar()
        tk.Entry(frame, textvariable=pass_confirmar_var,
                font=("Arial", 10),
                bg='white', fg='black',
                show="•").pack(fill=tk.X, pady=(0, 15))
        
        def cambiar():
            actual = pass_actual_var.get()
            nueva = pass_nueva_var.get()
            confirmar = pass_confirmar_var.get()
            
            if nueva != confirmar:
                messagebox.showerror("Error", "Las nuevas contraseñas no coinciden")
                return
            
            exito, mensaje = sistema.cambiar_password_docente(actual, nueva)
            if exito:
                messagebox.showinfo("Éxito", mensaje)
                pass_win.destroy()
            else:
                messagebox.showerror("Error", mensaje)
        
        tk.Button(frame, text="🔄 Cambiar Contraseña",
                 command=cambiar,
                 bg='#E74C3C', fg='white',
                 font=("Arial", 10, "bold")).pack()
    
    def abrir_github(self):
        """Abre el repositorio en GitHub"""
        try:
            webbrowser.open("https://github.com/tu_usuario/ecollajta")
        except:
            messagebox.showinfo("GitHub", "URL: https://github.com/tu_usuario/ecollajta")
    
    def mostrar_estadisticas(self):
        """Muestra estadísticas de uso"""
        messagebox.showinfo("Estadísticas", 
                          "Estadísticas de uso:\n\n"
                          "• Modo actual: " + ("Docente" if self.modo == "docente" else "Alumno") + "\n"
                          "• Último acceso: " + datetime.now().strftime("%Y-%m-%d %H:%M:%S") + "\n"
                          "• Nodos en sistema: 8\n"
                          "• Versión: 1.0")
    
    def cerrar_sesion_docente(self):
        """Cierra sesión de docente y vuelve a selección de modo"""
        if messagebox.askyesno("Cerrar Sesión", 
                              "¿Cerrar sesión docente y volver a selección de modo?"):
            self.root.destroy()
            iniciar_aplicacion()
    
    def calcular_todo(self):
        # 1. Calcular Matriz A
        self.matriz_a = np.array([
            [1 - self.matriz_p[0, 0], -self.matriz_p[0, 1], 1],
            [-self.matriz_p[1, 0], 1 - self.matriz_p[1, 1], 1]
        ])
        
        # 2. Calcular día sugerido
        try:
            A_sistema = np.array([
                [1 - self.matriz_p[0, 0], -self.matriz_p[0, 1]],
                [-self.matriz_p[1, 0], 1 - self.matriz_p[1, 1]]
            ])
            b_sistema = np.array([1, 1])
            solucion = np.linalg.solve(A_sistema, b_sistema)
            self.dia_sugerido = solucion[0]
        except:
            self.dia_sugerido = 3.333333333333333
        
        # 3. Determinar si recolectar
        self.sistema_activo = self.dias_transcurridos >= self.dia_sugerido
        self.decision = "SISTEMA ACTIVO: RECOLECTAR" if self.sistema_activo else "SISTEMA EN ESPERA"
        
        # 4. Calcular Carga Estimada (FÓRMULA EXACTA)
        if self.dia_sugerido > 0:
            self.puntos['Carga_g'] = (self.puntos['Cantidad_g'] / self.dia_sugerido) * self.dias_transcurridos
        else:
            self.puntos['Carga_g'] = self.puntos['Cantidad_g']
        
        # ===== MODIFICACIÓN: LIMITAR CARGA POR CAPACIDAD MÁXIMA =====
        # La carga no puede exceder la capacidad máxima
        self.puntos['Carga_g'] = self.puntos[['Carga_g', 'Capacidad_Max_g']].min(axis=1)
        # ===== FIN DE MODIFICACIÓN =====
        
        # ===== MODIFICACIÓN: ESTADOS SEGÚN CARGA =====
        # 0-600g = Vacío | 600-1500g = Medio | 1500+ = Lleno
        def determinar_estado(carga):
            if carga <= 600:
                return 'Vacío'
            elif 600 < carga <= 1500:
                return 'Medio'
            else:
                return 'Lleno'
        
        # Aplicar función a cada punto
        self.puntos['Estado'] = self.puntos['Carga_g'].apply(determinar_estado)
        
        # 5. Calcular Ratio (g/min)
        self.puntos['Ratio'] = self.puntos['Carga_g'] / self.puntos['Tiempo_min']
        
        # 6. Calcular Prioridad
        self.puntos['Prioridad'] = self.puntos['Ratio'].rank(ascending=False).astype(int)
        
        # 7. Ordenar por prioridad (pero mantener el orden original para visualización)
        # Primero ordenamos por prioridad para cálculos
        self.puntos_para_calculos = self.puntos.sort_values('Prioridad').copy()
        
        # 8. Calcular restricciones usando el orden por prioridad
        self.calcular_restricciones()
        
        # ===== AÑADIDO: CALCULAR NUEVAS MÉTRICAS =====
        self.calcular_metricas_adicionales()
        
        # ===== AÑADIDO: CALCULAR MÉTRICAS DE MACETAS =====
        self.calcular_metricas_macetas()
    
    def calcular_metricas_adicionales(self):
        """Calcula las nuevas métricas añadidas"""
        # 1. Carga total del sistema
        self.carga_total_sistema = self.puntos['Carga_g'].sum()
        
        # 2. Déficit respecto a meta (ahora personalizable)
        self.deficit_meta = max(0, self.meta_usuario - self.carga_total_sistema)
        
        # 3. Nodos de expansión necesarios (estimación)
        if self.deficit_meta > 0:
            carga_promedio = self.puntos['Carga_g'].mean()
            if carga_promedio > 0:
                self.nodos_expansion_necesarios = math.ceil(self.deficit_meta / carga_promedio)
            else:
                self.nodos_expansion_necesarios = 0
        else:
            self.nodos_expansion_necesarios = 0
    
    def calcular_metricas_macetas(self):
        """Calcula métricas relacionadas con las macetas"""
        # 1. Calcular cuántas macetas se pueden hacer con la carga total del sistema
        self.macetas_posibles = self.carga_total_sistema / self.gramos_por_maceta
        
        # 2. Calcular déficit de macetas (cuántas faltan para las deseadas)
        self.deficit_macetas = max(0, self.macetas_deseadas - self.macetas_posibles)
        
        # 3. Calcular eficiencia de recolección para macetas
        if self.total_carga_prioritaria > 0:
            self.macetas_prioritarias = self.total_carga_prioritaria / self.gramos_por_maceta
        else:
            self.macetas_prioritarias = 0
    
    def calcular_restricciones(self):
        """Calcula las restricciones de carga y tiempo"""
        # Obtener nodos con mayor prioridad (hasta alcanzar meta_usuario o 60 min)
        self.nodos_seleccionados = []
        self.total_carga_prioritaria = 0.0
        self.total_tiempo_prioritario = 0.0
        
        # Usar el DataFrame ordenado por prioridad para cálculos
        for _, punto in self.puntos_para_calculos.iterrows():
            if self.total_carga_prioritaria < self.meta_usuario:
                carga_punto = punto['Carga_g']
                tiempo_punto = punto['Tiempo_min']
                
                # Verificar si agregar este punto excede 60 min
                if self.total_tiempo_prioritario + tiempo_punto <= 60:
                    self.nodos_seleccionados.append(punto['Nodo'])
                    self.total_carga_prioritaria += carga_punto
                    self.total_tiempo_prioritario += tiempo_punto
                else:
                    break
        
        # Verificar si se cumple la meta
        self.meta_cumplida = self.total_carga_prioritaria >= self.meta_usuario
        self.tiempo_cumplido = self.total_tiempo_prioritario <= 60
    
    def create_widgets(self):
        """Crea la interfaz gráfica"""
        # TÍTULO
        title_frame = tk.Frame(self.root, bg=self.color_fondo)
        title_frame.pack(fill=tk.X, pady=10)
        
        tk.Label(title_frame, 
                text="🏭 CALCULADORA ECOLLAJTA - GRUPO 23",
                font=("Arial", 18, "bold"),
                bg=self.color_fondo,
                fg='#F1C40F').pack()
        
        tk.Label(title_frame,
                text="Cadenas de Markov + Algoritmo Voraz",
                font=("Arial", 12),
                bg=self.color_fondo,
                fg='#BDC3C7').pack()
        
        tk.Label(title_frame,
                text="📊 Estados: 0-600g=Vacío | 600-1500g=Medio | 1500+g=Lleno",
                font=("Arial", 10, "italic"),
                bg=self.color_fondo,
                fg='#BDC3C7').pack(pady=5)
        
        tk.Label(title_frame,
                text="🌱 NUEVO: Sistema de macetas - 180g por maceta",
                font=("Arial", 10, "italic"),
                bg=self.color_fondo,
                fg='#2ECC71').pack(pady=2)
        
        # FRAME PRINCIPAL
        main_frame = tk.Frame(self.root, bg='#34495E')
        main_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)
        
        # PANEL IZQUIERDO
        left_panel = tk.Frame(main_frame, bg='#34495E')
        left_panel.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=(0, 5))
        
        # PANEL DERECHO
        right_panel = tk.Frame(main_frame, bg='#34495E')
        right_panel.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True, padx=(5, 0))
        
        # ===== PANEL IZQUIERDO =====
        
        # 1. MATRIZ DE PROBABILIDADES
        matriz_frame = tk.LabelFrame(left_panel, text="📊 MATRIZ DE PROBABILIDADES (P)",
                                    font=("Arial", 10, "bold"),
                                    bg='#2C3E50', fg='white',
                                    padx=10, pady=10)
        matriz_frame.pack(fill=tk.BOTH, pady=(0, 10))
        
        self.create_matriz_section(matriz_frame)
        
        # 2. PARÁMETROS
        params_frame = tk.LabelFrame(left_panel, text="⚙️ PARÁMETROS",
                                    font=("Arial", 10, "bold"),
                                    bg='#2C3E50', fg='white',
                                    padx=10, pady=10)
        params_frame.pack(fill=tk.BOTH, pady=(0, 10))
        
        self.create_params_section(params_frame)
        
        # 3. MATRIZ A
        matriz_a_frame = tk.LabelFrame(left_panel, text="🧮 MATRIZ A (SISTEMA LINEAL)",
                                      font=("Arial", 10, "bold"),
                                      bg='#2C3E50', fg='white',
                                      padx=10, pady=10)
        matriz_a_frame.pack(fill=tk.BOTH)
        
        self.create_matriz_a_section(matriz_a_frame)
        
        # ===== PANEL DERECHO =====
        
        # 1. RESULTADOS
        results_frame = tk.LabelFrame(right_panel, text="📈 RESULTADOS",
                                     font=("Arial", 10, "bold"),
                                     bg='#2C3E50', fg='white',
                                     padx=10, pady=10)
        results_frame.pack(fill=tk.X, pady=(0, 10))
        
        self.create_results_section(results_frame)
        
        # 2. TABLA DE PUNTOS
        table_frame = tk.LabelFrame(right_panel, text="📍 PUNTOS DE RECOLECCIÓN",
                                   font=("Arial", 10, "bold"),
                                   bg='#2C3E50', fg='white',
                                   padx=10, pady=10)
        table_frame.pack(fill=tk.BOTH, expand=True, pady=(0, 10))
        
        self.create_table_section(table_frame)
        
        # 3. RESTRICCIONES
        restricciones_frame = tk.LabelFrame(right_panel, text="⚠️ RESTRICCIONES",
                                          font=("Arial", 10, "bold"),
                                          bg='#2C3E50', fg='white',
                                          padx=10, pady=10)
        restricciones_frame.pack(fill=tk.X, pady=(0, 10))
        
        self.create_restricciones_section(restricciones_frame)
        
        # 4. SISTEMA DE MACETAS (NUEVA SECCIÓN)
        macetas_frame = tk.LabelFrame(right_panel, text="🌱 SISTEMA DE MACETAS",
                                     font=("Arial", 10, "bold"),
                                     bg='#2C3E50', fg='#27AE60',
                                     padx=10, pady=10)
        macetas_frame.pack(fill=tk.X, pady=(0, 10))
        
        self.create_macetas_section(macetas_frame)
        
        # 5. NUEVAS MÉTRICAS AÑADIDAS
        metricas_frame = tk.LabelFrame(right_panel, text="📊 MÉTRICAS DEL SISTEMA",
                                      font=("Arial", 10, "bold"),
                                      bg='#2C3E50', fg='#F1C40F',
                                      padx=10, pady=10)
        metricas_frame.pack(fill=tk.X, pady=(0, 10))
        
        self.create_metricas_section(metricas_frame)
        
        # 6. BOTONES
        buttons_frame = tk.Frame(right_panel, bg='#34495E')
        buttons_frame.pack(fill=tk.X)
        
        self.create_buttons_section(buttons_frame)
    
    def create_matriz_section(self, parent):
        """Crea sección para editar matriz P"""
        frame = tk.Frame(parent, bg='#2C3E50')
        frame.pack()
        
        # Encabezados
        estados = ['Vacío', 'Medio', 'Lleno']
        
        for j in range(3):
            tk.Label(frame, text=estados[j],
                    font=("Arial", 9, "bold"),
                    bg='#2C3E50', fg='white',
                    width=10).grid(row=0, column=j+1, padx=2, pady=2)
        
        # Entradas para la matriz
        self.entradas_matriz = []
        
        for i in range(3):
            tk.Label(frame, text=f"Desde {estados[i]}",
                    font=("Arial", 9),
                    bg='#2C3E50', fg='white',
                    width=12, anchor='w').grid(row=i+1, column=0, padx=5, pady=5, sticky='w')
            
            row_entries = []
            for j in range(3):
                var = tk.StringVar()
                var.set(f"{self.matriz_p[i, j]:.3f}")
                entry = tk.Entry(frame, textvariable=var,
                                font=("Arial", 10),
                                bg='white', fg='black',
                                width=10, justify='center')
                entry.grid(row=i+1, column=j+1, padx=5, pady=5)
                row_entries.append(var)
            
            self.entradas_matriz.append(row_entries)
        
        # Botón para actualizar
        tk.Button(parent, text="🔄 ACTUALIZAR MATRIZ",
                 command=self.actualizar_matriz,
                 bg='#3498DB', fg='white',
                 font=("Arial", 10, "bold"),
                 padx=10, pady=5).pack(pady=(10, 0))
    
    def create_params_section(self, parent):
        """Crea sección de parámetros"""
        frame = tk.Frame(parent, bg='#2C3E50')
        frame.pack(fill=tk.X)
        
        tk.Label(frame, text="Días transcurridos:",
                font=("Arial", 10),
                bg='#2C3E50', fg='white').pack(side=tk.LEFT, padx=(0, 10))
        
        self.dias_var = tk.StringVar()
        self.dias_var.set(str(self.dias_transcurridos))
        
        tk.Entry(frame, textvariable=self.dias_var,
                font=("Arial", 10),
                bg='white', fg='black',
                width=10).pack(side=tk.LEFT, padx=(0, 10))
        
        tk.Button(frame, text="Actualizar",
                 command=self.actualizar_dias,
                 bg='#27AE60', fg='white',
                 font=("Arial", 10)).pack(side=tk.LEFT)
    
    def create_matriz_a_section(self, parent):
        """Crea sección para mostrar Matriz A"""
        frame = tk.Frame(parent, bg='#2C3E50')
        frame.pack()
        
        # Mostrar Matriz A
        text_widget = tk.Text(frame, height=3, width=40,
                             font=("Courier", 10),
                             bg='white', fg='black')
        text_widget.pack(pady=5)
        
        # Insertar valores
        matriz_str = ""
        for i in range(2):
            fila = f"[{self.matriz_a[i, 0]:.3f}  {self.matriz_a[i, 1]:.3f}  {self.matriz_a[i, 2]:.1f}]"
            matriz_str += fila + "\n"
        
        text_widget.insert(1.0, matriz_str)
        text_widget.config(state=tk.DISABLED)
    
    def create_results_section(self, parent):
        """Crea sección de resultados"""
        frame = tk.Frame(parent, bg='#2C3E50')
        frame.pack(fill=tk.X)
        
        # Día sugerido
        tk.Label(frame, text="Día Sugerido de Recolección:",
                font=("Arial", 11, "bold"),
                bg='#2C3E50', fg='white').pack(anchor='w')
        
        self.dia_label = tk.Label(frame, text=f"{self.dia_sugerido:.6f}",
                                 font=("Arial", 12, "bold"),
                                 bg='#2C3E50', fg='#2ECC71')
        self.dia_label.pack(anchor='w', pady=(0, 10))
        
        # Decisión
        color = '#2ECC71' if self.sistema_activo else '#E74C3C'
        tk.Label(frame, text=self.decision,
                font=("Arial", 12, "bold"),
                bg=color, fg='white',
                padx=20, pady=10).pack(fill=tk.X)
        
        # Info estados
        info_frame = tk.Frame(parent, bg='#2C3E50')
        info_frame.pack(fill=tk.X, pady=10)
        
        tk.Label(info_frame,
                text="📊 DISTRIBUCIÓN DE ESTADOS:",
                font=("Arial", 10, "bold"),
                bg='#2C3E50', fg='white').pack(anchor='w')
        
        # Contar estados
        vacios = (self.puntos['Estado'] == 'Vacío').sum()
        medios = (self.puntos['Estado'] == 'Medio').sum()
        llenos = (self.puntos['Estado'] == 'Lleno').sum()
        
        tk.Label(info_frame,
                text=f"Vacío: {vacios} puntos | Medio: {medios} puntos | Lleno: {llenos} puntos",
                font=("Arial", 10),
                bg='#2C3E50', fg='#BDC3C7').pack(anchor='w')
    
    def create_table_section(self, parent):
        """Crea tabla de puntos"""
        # Frame para tabla y scrollbar
        container = tk.Frame(parent, bg='#2C3E50')
        container.pack(fill=tk.BOTH, expand=True)
        
        # Definir columnas
        columns = ['Nodo', 'Punto', 'Cantidad (g)', 'Tiempo (min)', 
                  'Capacidad Max (g)', 'Estado', 'Carga (g)', 'Prioridad', 'Ratio']
        
        # Crear Treeview
        self.tree = ttk.Treeview(container, columns=columns, show='headings', height=8)
        
        # Configurar columnas
        widths = [50, 120, 90, 90, 110, 70, 80, 70, 70]
        for idx, col in enumerate(columns):
            self.tree.heading(col, text=col)
            self.tree.column(col, width=widths[idx], anchor='center')
        
        # Configurar orden
        self.tree.heading('Nodo', text='Nodo', command=lambda: self.ordenar_por_columna('Nodo', False))
        self.tree.heading('Prioridad', text='Prioridad', command=lambda: self.ordenar_por_columna('Prioridad', False))
        
        # Scrollbars
        vsb = ttk.Scrollbar(container, orient="vertical", command=self.tree.yview)
        hsb = ttk.Scrollbar(container, orient="horizontal", command=self.tree.xview)
        self.tree.configure(yscrollcommand=vsb.set, xscrollcommand=hsb.set)
        
        # Grid
        self.tree.grid(row=0, column=0, sticky='nsew')
        vsb.grid(row=0, column=1, sticky='ns')
        hsb.grid(row=1, column=0, sticky='ew', columnspan=2)
        
        container.grid_rowconfigure(0, weight=1)
        container.grid_columnconfigure(0, weight=1)
        
        # Llenar tabla
        self.actualizar_tabla()
        
        # Botones para editar
        btn_frame = tk.Frame(parent, bg='#2C3E50')
        btn_frame.pack(fill=tk.X, pady=(10, 0))
        
        tk.Button(btn_frame, text="✏️ Editar Punto",
                 command=self.editar_punto,
                 bg='#F39C12', fg='white',
                 font=("Arial", 10)).pack(side=tk.LEFT, padx=5)
        
        tk.Button(btn_frame, text="➕ Añadir Nuevo Nodo",
                 command=self.aniadir_nuevo_nodo,
                 bg='#9B59B6', fg='white',
                 font=("Arial", 10)).pack(side=tk.LEFT, padx=5)
        
        tk.Button(btn_frame, text="🗑️ Eliminar Nodo",
                 command=self.eliminar_nodo,
                 bg='#E74C3C', fg='white',
                 font=("Arial", 10)).pack(side=tk.LEFT, padx=5)
        
        tk.Button(btn_frame, text="🔄 Recalcular Todo",
                 command=self.recalcular_todo,
                 bg='#3498DB', fg='white',
                 font=("Arial", 10)).pack(side=tk.LEFT, padx=5)
    
    def ordenar_por_columna(self, col, reverse):
        """Ordena la tabla por una columna específica"""
        items = [(self.tree.set(item, col), item) for item in self.tree.get_children('')]
        
        if col == 'Nodo':
            items.sort(key=lambda x: int(x[0]), reverse=reverse)
        elif col == 'Prioridad':
            items.sort(key=lambda x: int(x[0]), reverse=reverse)
        elif col == 'Carga (g)' or col == 'Ratio':
            items.sort(key=lambda x: float(x[0]), reverse=reverse)
        else:
            items.sort(key=lambda x: x[0], reverse=reverse)
        
        for index, (_, item) in enumerate(items):
            self.tree.move(item, '', index)
        
        self.tree.heading(col, command=lambda: self.ordenar_por_columna(col, not reverse))
    
    def create_restricciones_section(self, parent):
        """Crea sección de restricciones"""
        frame = tk.Frame(parent, bg='#2C3E50')
        frame.pack(fill=tk.X)
        
        # Nodos seleccionados
        tk.Label(frame, text="Nodos Priorizados:",
                font=("Arial", 10, "bold"),
                bg='#2C3E50', fg='white').pack(anchor='w')
        
        nodos_ordenados = sorted(self.nodos_seleccionados)
        self.nodos_label = tk.Label(frame, 
                                   text=f"{', '.join(map(str, nodos_ordenados))}",
                                   font=("Arial", 10),
                                   bg='#2C3E50', fg='#F1C40F')
        self.nodos_label.pack(anchor='w', pady=(0, 5))
        
        # Restricción 1: Carga mínima - MODIFICADO: Solo dice "Meta:" sin los 4000g
        carga_frame = tk.Frame(frame, bg='#2C3E50')
        carga_frame.pack(fill=tk.X, pady=2)
        
        carga_color = '#27AE60' if self.meta_cumplida else '#E74C3C'
        carga_texto = "✓ CUMPLE META" if self.meta_cumplida else "✗ NO CUMPLE META"
        
        tk.Label(carga_frame, text="⚖️ Carga Total (Meta:):",  # MODIFICADO AQUÍ
                font=("Arial", 9, "bold"),
                bg='#2C3E50', fg='white').pack(side=tk.LEFT)
        
        self.carga_label = tk.Label(carga_frame,
                                   text=f"{self.total_carga_prioritaria:.1f} / {self.meta_usuario:.0f} g → {carga_texto}",
                                   font=("Arial", 9, "bold"),
                                   bg=carga_color, fg='white',
                                   padx=10, pady=2)
        self.carga_label.pack(side=tk.LEFT, padx=(10, 0))
        
        # Restricción 2: Tiempo máximo
        tiempo_frame = tk.Frame(frame, bg='#2C3E50')
        tiempo_frame.pack(fill=tk.X, pady=2)
        
        tiempo_color = '#27AE60' if self.tiempo_cumplido else '#E74C3C'
        tiempo_texto = "✓ TIEMPO OK" if self.tiempo_cumplido else "✗ EXCEDE LÍMITE"
        
        tk.Label(tiempo_frame, text="⏱️ Tiempo Total:",
                font=("Arial", 9, "bold"),
                bg='#2C3E50', fg='white').pack(side=tk.LEFT)
        
        self.tiempo_label = tk.Label(tiempo_frame,
                                    text=f"{self.total_tiempo_prioritario:.1f} / 60 min → {tiempo_texto}",
                                    font=("Arial", 9, "bold"),
                                    bg=tiempo_color, fg='white',
                                    padx=10, pady=2)
        self.tiempo_label.pack(side=tk.LEFT, padx=(10, 0))
    
    def create_macetas_section(self, parent):
        """Crea sección para sistema de macetas"""
        frame = tk.Frame(parent, bg='#2C3E50')
        frame.pack(fill=tk.X)
        
        # Información de conversión
        info_label = tk.Label(frame, 
                             text="🌱 Cada maceta requiere 180g de cáscara de huevo",
                             font=("Arial", 9, "bold"),
                             bg='#2C3E50', fg='#2ECC71')
        info_label.pack(anchor='w', pady=(0, 10))
        
        # Frame para entrada de macetas
        entrada_frame = tk.Frame(frame, bg='#2C3E50')
        entrada_frame.pack(fill=tk.X, pady=5)
        
        tk.Label(entrada_frame, text="Macetas deseadas:",
                font=("Arial", 10, "bold"),
                bg='#2C3E50', fg='white').pack(side=tk.LEFT, padx=(0, 10))
        
        # Variable para número de macetas
        self.macetas_var = tk.StringVar(value=str(self.macetas_deseadas))
        macetas_entry = tk.Entry(entrada_frame, textvariable=self.macetas_var,
                                font=("Arial", 10),
                                bg='white', fg='black',
                                width=10)
        macetas_entry.pack(side=tk.LEFT, padx=(0, 10))
        
        tk.Button(entrada_frame, text="🌱 Calcular Meta",
                 command=self.actualizar_meta_desde_macetas,
                 bg='#27AE60', fg='white',
                 font=("Arial", 10)).pack(side=tk.LEFT)
        
        # Separador
        tk.Frame(frame, height=2, bg='#34495E').pack(fill=tk.X, pady=10)
        
        # Resultados de macetas
        # 1. Equivalencia
        equivalencia_frame = tk.Frame(frame, bg='#2C3E50')
        equivalencia_frame.pack(fill=tk.X, pady=2)
        
        tk.Label(equivalencia_frame, text="📊 Equivalencia:",
                font=("Arial", 9, "bold"),
                bg='#2C3E50', fg='white').pack(side=tk.LEFT)
        
        self.equivalencia_label = tk.Label(equivalencia_frame,
                                          text=f"{self.macetas_deseadas} macetas = {self.meta_usuario:.0f}g",
                                          font=("Arial", 9),
                                          bg='#2C3E50', fg='#F1C40F')
        self.equivalencia_label.pack(side=tk.LEFT, padx=(10, 0))
        
        # 2. Macetas posibles con carga total
        macetas_posibles_frame = tk.Frame(frame, bg='#2C3E50')
        macetas_posibles_frame.pack(fill=tk.X, pady=2)
        
        tk.Label(macetas_posibles_frame, text="🏭 Macetas posibles (sistema total):",
                font=("Arial", 9, "bold"),
                bg='#2C3E50', fg='white').pack(side=tk.LEFT)
        
        self.macetas_posibles_label = tk.Label(macetas_posibles_frame,
                                              text=f"{self.macetas_posibles:.1f} macetas",
                                              font=("Arial", 9, "bold"),
                                              bg='#2980B9', fg='white',
                                              padx=10, pady=2)
        self.macetas_posibles_label.pack(side=tk.LEFT, padx=(10, 0))
        
        # 3. Déficit de macetas
        deficit_macetas_frame = tk.Frame(frame, bg='#2C3E50')
        deficit_macetas_frame.pack(fill=tk.X, pady=2)
        
        tk.Label(deficit_macetas_frame, text="📉 Déficit de macetas:",
                font=("Arial", 9, "bold"),
                bg='#2C3E50', fg='white').pack(side=tk.LEFT)
        
        deficit_color = '#27AE60' if self.deficit_macetas == 0 else '#E74C3C'
        deficit_texto = "✓ SIN DÉFICIT" if self.deficit_macetas == 0 else f"✗ FALTAN {self.deficit_macetas:.1f} macetas"
        
        self.deficit_macetas_label = tk.Label(deficit_macetas_frame,
                                             text=f"{self.deficit_macetas:.1f} → {deficit_texto}",
                                             font=("Arial", 9, "bold"),
                                             bg=deficit_color, fg='white',
                                             padx=10, pady=2)
        self.deficit_macetas_label.pack(side=tk.LEFT, padx=(10, 0))
        
        # 4. Macetas con ruta prioritaria
        macetas_prioritarias_frame = tk.Frame(frame, bg='#2C3E50')
        macetas_prioritarias_frame.pack(fill=tk.X, pady=2)
        
        tk.Label(macetas_prioritarias_frame, text="🚚 Macetas (ruta prioritaria):",
                font=("Arial", 9, "bold"),
                bg='#2C3E50', fg='white').pack(side=tk.LEFT)
        
        self.macetas_prioritarias_label = tk.Label(macetas_prioritarias_frame,
                                                  text=f"{self.macetas_prioritarias:.1f} macetas",
                                                  font=("Arial", 9, "bold"),
                                                  bg='#8E44AD', fg='white',
                                                  padx=10, pady=2)
        self.macetas_prioritarias_label.pack(side=tk.LEFT, padx=(10, 0))
        
        # 5. Recomendación
        recomendacion_frame = tk.Frame(frame, bg='#2C3E50')
        recomendacion_frame.pack(fill=tk.X, pady=5)
        
        if self.deficit_macetas == 0:
            texto_recomendacion = f"✅ ¡Suficiente para {self.macetas_deseadas} macetas!"
            color_recomendacion = '#27AE60'
        else:
            texto_recomendacion = f"⚠️ Necesitas {self.deficit_meta:.0f}g más ({self.deficit_macetas:.1f} macetas)"
            color_recomendacion = '#E74C3C'
        
        self.recomendacion_label = tk.Label(recomendacion_frame,
                                           text=texto_recomendacion,
                                           font=("Arial", 9, "bold"),
                                           bg=color_recomendacion, fg='white',
                                           padx=10, pady=5)
        self.recomendacion_label.pack(fill=tk.X)
    
    def create_metricas_section(self, parent):
        """Crea sección de nuevas métricas añadidas"""
        frame = tk.Frame(parent, bg='#2C3E50')
        frame.pack(fill=tk.X)
        
        # 1. CARGA TOTAL DEL SISTEMA
        carga_total_frame = tk.Frame(frame, bg='#2C3E50')
        carga_total_frame.pack(fill=tk.X, pady=2)
        
        tk.Label(carga_total_frame, text="📦 Carga Total del Sistema:",
                font=("Arial", 9, "bold"),
                bg='#2C3E50', fg='white').pack(side=tk.LEFT)
        
        self.carga_total_label = tk.Label(carga_total_frame,
                                         text=f"{self.carga_total_sistema:.1f} g",
                                         font=("Arial", 9, "bold"),
                                         bg='#2980B9', fg='white',
                                         padx=10, pady=2)
        self.carga_total_label.pack(side=tk.LEFT, padx=(10, 0))
        
        # 2. DÉFICIT RESPECTO A META - MODIFICADO: Solo dice "Meta:"
        deficit_frame = tk.Frame(frame, bg='#2C3E50')
        deficit_frame.pack(fill=tk.X, pady=2)
        
        self.deficit_text_label = tk.Label(deficit_frame,
                                          text="📉 Déficit respecto a meta:",  # MODIFICADO AQUÍ
                                          font=("Arial", 9, "bold"),
                                          bg='#2C3E50', fg='white')
        self.deficit_text_label.pack(side=tk.LEFT)
        
        deficit_color = '#27AE60' if self.deficit_meta == 0 else '#E74C3C'
        deficit_valor_texto = "✓ SIN DÉFICIT" if self.deficit_meta == 0 else f"✗ FALTAN {self.deficit_meta:.1f} g"
        
        self.deficit_label = tk.Label(deficit_frame,
                                     text=f"{self.deficit_meta:.1f} g → {deficit_valor_texto}",
                                     font=("Arial", 9, "bold"),
                                     bg=deficit_color, fg='white',
                                     padx=10, pady=2)
        self.deficit_label.pack(side=tk.LEFT, padx=(10, 0))
        
        # 3. NODOS DE EXPANSIÓN NECESARIOS
        expansion_frame = tk.Frame(frame, bg='#2C3E50')
        expansion_frame.pack(fill=tk.X, pady=2)
        
        tk.Label(expansion_frame, text="🏗️ Nodos de expansión necesarios:",
                font=("Arial", 9, "bold"),
                bg='#2C3E50', fg='white').pack(side=tk.LEFT)
        
        expansion_texto = f"{self.nodos_expansion_necesarios} nodos" if self.deficit_meta > 0 else "No se necesitan"
        self.expansion_label = tk.Label(expansion_frame,
                                       text=expansion_texto,
                                       font=("Arial", 9, "bold"),
                                       bg='#8E44AD', fg='white',
                                       padx=10, pady=2)
        self.expansion_label.pack(side=tk.LEFT, padx=(10, 0))
    
    def create_buttons_section(self, parent):
        """Crea botones de acción"""
        tk.Button(parent, text="💾 Exportar a Excel",
                 command=self.exportar_excel,
                 bg='#27AE60', fg='white',
                 font=("Arial", 11, "bold"),
                 padx=20, pady=10).pack(side=tk.LEFT, padx=10)
        
        # Botón de "Ver Código" solo para docente
        if self.modo == "docente":
            tk.Button(parent, text="👁️ Ver Código Fuente",
                     command=self.mostrar_codigo_completo,
                     bg='#F39C12', fg='white',
                     font=("Arial", 11, "bold"),
                     padx=20, pady=10).pack(side=tk.LEFT, padx=10)
        
        tk.Button(parent, text="🔄 Reiniciar Valores",
                 command=self.reiniciar_valores,
                 bg='#E74C3C', fg='white',
                 font=("Arial", 11, "bold"),
                 padx=20, pady=10).pack(side=tk.LEFT, padx=10)
        
        tk.Button(parent, text="🚪 Salir",
                 command=self.root.quit,
                 bg='#95A5A6', fg='white',
                 font=("Arial", 11, "bold"),
                 padx=20, pady=10).pack(side=tk.LEFT, padx=10)
    
    def actualizar_matriz(self):
        """Actualiza la matriz desde las entradas con validación"""
        try:
            nueva_matriz = np.zeros((3, 3))
            
            for i in range(3):
                suma_fila = 0.0
                for j in range(3):
                    texto = self.entradas_matriz[i][j].get().strip()
                    texto = texto.replace(',', '.')
                    valor = float(texto)
                    
                    if valor < 0 or valor > 1:
                        messagebox.showerror("Error", f"Valor {valor} debe estar entre 0 y 1")
                        return
                    
                    nueva_matriz[i, j] = valor
                    suma_fila += valor
                
                if suma_fila > 1.0:
                    messagebox.showerror("Error", 
                                        f"Suma de fila {i+1} = {suma_fila:.3f}\n"
                                        f"La suma no debe exceder 1.0")
                    for j in range(3):
                        self.entradas_matriz[i][j].set(f"{self.matriz_p[i, j]:.3f}")
                    return
            
            self.matriz_p = nueva_matriz
            self.recalcular_todo()
            messagebox.showinfo("Éxito", "Matriz actualizada correctamente")
            
        except ValueError:
            messagebox.showerror("Error", "Ingrese números válidos (ej: 0.5)")
    
    def actualizar_dias(self):
        """Actualiza los días transcurridos"""
        try:
            dias = int(self.dias_var.get())
            if dias >= 0:
                self.dias_transcurridos = dias
                self.recalcular_todo()
                messagebox.showinfo("Éxito", f"Días actualizados a {dias}")
            else:
                messagebox.showerror("Error", "Los días no pueden ser negativos")
        except ValueError:
            messagebox.showerror("Error", "Ingrese un número entero válido")
    
    def actualizar_meta_desde_macetas(self):
        """Actualiza la meta desde el número de macetas"""
        try:
            macetas = int(self.macetas_var.get())
            if macetas > 0:
                # Convertir macetas a gramos
                nueva_meta = macetas * self.gramos_por_maceta
                self.meta_usuario = nueva_meta
                self.macetas_deseadas = macetas
                
                self.recalcular_todo()
                
                # Mostrar mensaje detallado
                mensaje = f"🌱 Meta actualizada:\n\n"
                mensaje += f"• Macetas deseadas: {macetas}\n"
                mensaje += f"• Gramos necesarios: {nueva_meta:.0f}g\n"
                mensaje += f"• Carga total del sistema: {self.carga_total_sistema:.1f}g\n"
                mensaje += f"• Macetas posibles: {self.macetas_posibles:.1f}\n"
                
                if self.deficit_macetas > 0:
                    mensaje += f"\n⚠️ Déficit: {self.deficit_macetas:.1f} macetas\n"
                    mensaje += f"   ({self.deficit_meta:.0f}g adicionales necesarios)"
                else:
                    mensaje += f"\n✅ ¡Suficiente para todas las macetas!"
                    if self.macetas_posibles > macetas:
                        mensaje += f"\n   (Sobran {self.macetas_posibles - macetas:.1f} macetas)"
                
                messagebox.showinfo("Meta Actualizada", mensaje)
                
            else:
                messagebox.showerror("Error", "El número de macetas debe ser mayor a 0")
        except ValueError:
            messagebox.showerror("Error", "Ingrese un número entero válido de macetas")
    
    def aniadir_nuevo_nodo(self):
        """Añade un nuevo nodo de recolección"""
        add_win = tk.Toplevel(self.root)
        add_win.title("➕ Añadir Nuevo Punto de Recolección")
        add_win.geometry("400x450")
        add_win.configure(bg='#2C3E50')
        
        tk.Label(add_win, text="NUEVO PUNTO DE RECOLECCIÓN",
                font=("Arial", 14, "bold"),
                bg='#2C3E50', fg='#F1C40F').pack(pady=10)
        
        # Campos
        tk.Label(add_win, text="Nombre del punto:",
                font=("Arial", 10),
                bg='#2C3E50', fg='white').pack()
        
        nombre_var = tk.StringVar(value="Nuevo Restaurante")
        tk.Entry(add_win, textvariable=nombre_var,
                font=("Arial", 10), width=30).pack(pady=5)
        
        tk.Label(add_win, text="Cantidad inicial (g):",
                font=("Arial", 10),
                bg='#2C3E50', fg='white').pack()
        
        cantidad_var = tk.StringVar(value="1000")
        tk.Entry(add_win, textvariable=cantidad_var,
                font=("Arial", 10), width=15).pack(pady=5)
        
        tk.Label(add_win, text="Tiempo de recolección (min):",
                font=("Arial", 10),
                bg='#2C3E50', fg='white').pack()
        
        tiempo_var = tk.StringVar(value="15")
        tk.Entry(add_win, textvariable=tiempo_var,
                font=("Arial", 10), width=15).pack(pady=5)
        
        tk.Label(add_win, text="Capacidad máxima (g):",
                font=("Arial", 10),
                bg='#2C3E50', fg='white').pack()
        
        capacidad_var = tk.StringVar(value="1500")
        tk.Entry(add_win, textvariable=capacidad_var,
                font=("Arial", 10), width=15).pack(pady=5)
        
        tk.Label(add_win, text="El estado se calculará automáticamente",
                font=("Arial", 9, "italic"),
                bg='#2C3E50', fg='#BDC3C7').pack()
        
        def guardar_nuevo_nodo():
            try:
                nombre = nombre_var.get().strip()
                if not nombre:
                    messagebox.showerror("Error", "El nombre no puede estar vacío")
                    return
                
                cantidad = float(cantidad_var.get())
                tiempo = float(tiempo_var.get())
                capacidad = float(capacidad_var.get())
                
                if cantidad <= 0 or tiempo <= 0 or capacidad <= 0:
                    messagebox.showerror("Error", "Los valores deben ser mayores a 0")
                    return
                
                if cantidad > capacidad:
                    messagebox.showwarning("Advertencia", 
                                          "La cantidad inicial es mayor que la capacidad máxima.\n"
                                          "Se limitará a la capacidad máxima.")
                
                nuevo_nodo = self.puntos['Nodo'].max() + 1
                
                nuevo_punto = pd.DataFrame({
                    'Nodo': [nuevo_nodo],
                    'Punto': [nombre],
                    'Cantidad_g': [cantidad],
                    'Tiempo_min': [tiempo],
                    'Capacidad_Max_g': [capacidad],
                    'Estado': ['Medio'],
                    'Carga_g': [0],
                    'Ratio': [0],
                    'Prioridad': [0]
                })
                
                self.puntos = pd.concat([self.puntos, nuevo_punto], ignore_index=True)
                self.recalcular_todo()
                add_win.destroy()
                messagebox.showinfo("Éxito", f"✅ Punto '{nombre}' añadido como Nodo {nuevo_nodo}")
                
            except ValueError:
                messagebox.showerror("Error", "Ingrese valores numéricos válidos")
        
        tk.Button(add_win, text="➕ Añadir Punto",
                 command=guardar_nuevo_nodo,
                 bg='#27AE60', fg='white',
                 font=("Arial", 12, "bold"),
                 padx=20, pady=10).pack(pady=20)
    
    def eliminar_nodo(self):
        """Elimina un nodo seleccionado"""
        seleccion = self.tree.selection()
        if not seleccion:
            messagebox.showwarning("Advertencia", "Seleccione un nodo para eliminar")
            return
        
        item = seleccion[0]
        valores = self.tree.item(item, 'values')
        nodo = int(valores[0])
        nombre = valores[1]
        
        respuesta = messagebox.askyesno("Confirmar", 
                                       f"¿Eliminar el nodo {nodo} - '{nombre}'?\n\n"
                                       f"Esta acción no se puede deshacer.")
        
        if respuesta:
            self.puntos = self.puntos[self.puntos['Nodo'] != nodo]
            self.recalcular_todo()
            messagebox.showinfo("Éxito", f"✅ Nodo {nodo} eliminado")
    
    def actualizar_tabla(self):
        """Actualiza la tabla con datos actuales"""
        for item in self.tree.get_children():
            self.tree.delete(item)
        
        puntos_ordenados = self.puntos.sort_values('Nodo')
        
        for _, punto in puntos_ordenados.iterrows():
            tags = ('selected',) if punto['Nodo'] in self.nodos_seleccionados else ()
            
            valores = [
                int(punto['Nodo']),
                punto['Punto'],
                f"{punto['Cantidad_g']:.0f}",
                f"{punto['Tiempo_min']:.0f}",
                f"{punto['Capacidad_Max_g']:.0f}",
                punto['Estado'],
                f"{punto['Carga_g']:.1f}",
                f"{int(punto['Prioridad'])}",
                f"{punto['Ratio']:.2f}"
            ]
            self.tree.insert("", tk.END, values=valores, tags=tags)
        
        self.tree.tag_configure('selected', background='#E8F4F8')
    
    def editar_punto(self):
        """Edita un punto seleccionado"""
        seleccion = self.tree.selection()
        if not seleccion:
            messagebox.showwarning("Advertencia", "Seleccione un punto para editar")
            return
        
        item = seleccion[0]
        valores = self.tree.item(item, 'values')
        nodo = int(valores[0])
        
        idx = self.puntos[self.puntos['Nodo'] == nodo].index[0]
        punto = self.puntos.loc[idx]
        
        edit_win = tk.Toplevel(self.root)
        edit_win.title(f"Editar Punto {nodo}")
        edit_win.geometry("300x350")
        edit_win.configure(bg='#2C3E50')
        
        tk.Label(edit_win, text=f"Editando: {punto['Punto']}",
                font=("Arial", 12, "bold"),
                bg='#2C3E50', fg='white').pack(pady=10)
        
        tk.Label(edit_win, text="Cantidad (g):",
                font=("Arial", 10),
                bg='#2C3E50', fg='white').pack()
        
        cant_var = tk.StringVar(value=str(int(punto['Cantidad_g'])))
        tk.Entry(edit_win, textvariable=cant_var,
                font=("Arial", 10)).pack(pady=5)
        
        tk.Label(edit_win, text="Tiempo (min):",
                font=("Arial", 10),
                bg='#2C3E50', fg='white').pack()
        
        tiempo_var = tk.StringVar(value=str(int(punto['Tiempo_min'])))
        tk.Entry(edit_win, textvariable=tiempo_var,
                font=("Arial", 10)).pack(pady=5)
        
        tk.Label(edit_win, text="Capacidad Máxima (g):",
                font=("Arial", 10),
                bg='#2C3E50', fg='white').pack()
        
        capacidad_var = tk.StringVar(value=str(int(punto['Capacidad_Max_g'])))
        tk.Entry(edit_win, textvariable=capacidad_var,
                font=("Arial", 10)).pack(pady=5)
        
        tk.Label(edit_win, text="Estado se calcula automáticamente:",
                font=("Arial", 9, "italic"),
                bg='#2C3E50', fg='#BDC3C7').pack()
        
        tk.Label(edit_win, 
                text="0-600g=Vacío | 600-1500g=Medio | 1500+g=Lleno",
                font=("Arial", 8, "italic"),
                bg='#2C3E50', fg='#7F8C8D').pack()
        
        def guardar():
            try:
                self.puntos.loc[idx, 'Cantidad_g'] = float(cant_var.get())
                self.puntos.loc[idx, 'Tiempo_min'] = float(tiempo_var.get())
                self.puntos.loc[idx, 'Capacidad_Max_g'] = float(capacidad_var.get())
                
                self.recalcular_todo()
                edit_win.destroy()
                messagebox.showinfo("Éxito", "Punto actualizado")
            except ValueError:
                messagebox.showerror("Error", "Ingrese valores válidos")
        
        tk.Button(edit_win, text="💾 Guardar",
                 command=guardar,
                 bg='#27AE60', fg='white',
                 font=("Arial", 10)).pack(pady=20)
    
    def recalcular_todo(self):
        """Recalcula todo y actualiza interfaz"""
        self.calcular_todo()
        self.actualizar_tabla()
        self.dia_label.config(text=f"{self.dia_sugerido:.6f}")
        
        # Actualizar etiquetas de restricciones
        nodos_ordenados = sorted(self.nodos_seleccionados)
        self.nodos_label.config(text=f"{', '.join(map(str, nodos_ordenados))}")
        
        # Restricciones
        carga_color = '#27AE60' if self.meta_cumplida else '#E74C3C'
        carga_texto = "✓ CUMPLE META" if self.meta_cumplida else "✗ NO CUMPLE META"
        self.carga_label.config(
            text=f"{self.total_carga_prioritaria:.1f} / {self.meta_usuario:.0f} g → {carga_texto}",
            bg=carga_color
        )
        
        tiempo_color = '#27AE60' if self.tiempo_cumplido else '#E74C3C'
        tiempo_texto = "✓ TIEMPO OK" if self.tiempo_cumplido else "✗ EXCEDE LÍMITE"
        self.tiempo_label.config(
            text=f"{self.total_tiempo_prioritario:.1f} / 60 min → {tiempo_texto}",
            bg=tiempo_color
        )
        
        # Métricas
        self.carga_total_label.config(text=f"{self.carga_total_sistema:.1f} g")
        
        # MODIFICADO: Solo dice "Meta:" en el texto
        self.deficit_text_label.config(text="📉 Déficit respecto a meta:")
        
        deficit_color = '#27AE60' if self.deficit_meta == 0 else '#E74C3C'
        deficit_texto = "✓ SIN DÉFICIT" if self.deficit_meta == 0 else f"✗ FALTAN {self.deficit_meta:.1f} g"
        self.deficit_label.config(
            text=f"{self.deficit_meta:.1f} g → {deficit_texto}",
            bg=deficit_color
        )
        
        expansion_texto = f"{self.nodos_expansion_necesarios} nodos" if self.deficit_meta > 0 else "No se necesitan"
        self.expansion_label.config(text=expansion_texto)
        
        # Sección de macetas
        self.equivalencia_label.config(
            text=f"{self.macetas_deseadas} macetas = {self.meta_usuario:.0f}g"
        )
        
        self.macetas_posibles_label.config(
            text=f"{self.macetas_posibles:.1f} macetas"
        )
        
        deficit_macetas_color = '#27AE60' if self.deficit_macetas == 0 else '#E74C3C'
        deficit_macetas_texto = "✓ SIN DÉFICIT" if self.deficit_macetas == 0 else f"✗ FALTAN {self.deficit_macetas:.1f} macetas"
        self.deficit_macetas_label.config(
            text=f"{self.deficit_macetas:.1f} → {deficit_macetas_texto}",
            bg=deficit_macetas_color
        )
        
        self.macetas_prioritarias_label.config(
            text=f"{self.macetas_prioritarias:.1f} macetas"
        )
        
        if self.deficit_macetas == 0:
            texto_recomendacion = f"✅ ¡Suficiente para {self.macetas_deseadas} macetas!"
            color_recomendacion = '#27AE60'
        else:
            texto_recomendacion = f"⚠️ Necesitas {self.deficit_meta:.0f}g más ({self.deficit_macetas:.1f} macetas)"
            color_recomendacion = '#E74C3C'
        
        self.recomendacion_label.config(
            text=texto_recomendacion,
            bg=color_recomendacion
        )
    
    def exportar_excel(self):
        """Exporta a Excel"""
        try:
            filename = "resultados_ecollajta.xlsx"
            
            with pd.ExcelWriter(filename, engine='openpyxl') as writer:
                # Hoja 1: Resumen
                vacios = (self.puntos['Estado'] == 'Vacío').sum()
                medios = (self.puntos['Estado'] == 'Medio').sum()
                llenos = (self.puntos['Estado'] == 'Lleno').sum()
                
                resumen = pd.DataFrame({
                    'Parámetro': ['Días Transcurridos', 'Día Sugerido', 'Decisión', 
                                 'Puntos Vacíos', 'Puntos Medios', 'Puntos Llenos',
                                 'Carga Total Prioritaria (g)', 'Tiempo Total Prioritario (min)',
                                 f'Meta Cumplida ({self.meta_usuario:.0f}g)', 
                                 'Tiempo Cumplido (≤60min)',
                                 'Nodos Seleccionados',
                                 '--- SISTEMA DE MACETAS ---',
                                 'Macetas deseadas',
                                 'Gramos por maceta',
                                 'Meta total (g)',
                                 'Macetas posibles (sistema total)',
                                 'Déficit de macetas',
                                 'Macetas (ruta prioritaria)',
                                 '--- MÉTRICAS DEL SISTEMA ---',
                                 'Carga Total del Sistema (g)',
                                 f'Déficit respecto a meta',
                                 'Nodos de Expansión Necesarios',
                                 '--- SISTEMA ---',
                                 'Total de nodos en el sistema'],
                    'Valor': [self.dias_transcurridos, f"{self.dia_sugerido:.6f}", 
                             self.decision, vacios, medios, llenos,
                             f"{self.total_carga_prioritaria:.1f}", f"{self.total_tiempo_prioritario:.1f}",
                             'SÍ' if self.meta_cumplida else 'NO',
                             'SÍ' if self.tiempo_cumplido else 'NO',
                             ', '.join(map(str, sorted(self.nodos_seleccionados))),
                             '',
                             self.macetas_deseadas,
                             f"{self.gramos_por_maceta}",
                             f"{self.meta_usuario:.0f}",
                             f"{self.macetas_posibles:.1f}",
                             f"{self.deficit_macetas:.1f}",
                             f"{self.macetas_prioritarias:.1f}",
                             '',
                             f"{self.carga_total_sistema:.1f}",
                             f"{self.deficit_meta:.1f}",
                             self.nodos_expansion_necesarios,
                             '',
                             len(self.puntos)]
                })
                resumen.to_excel(writer, sheet_name='Resumen', index=False)
                
                # Hoja 2: Matriz P
                matriz_p_df = pd.DataFrame(self.matriz_p,
                                          index=['Vacío', 'Medio', 'Lleno'],
                                          columns=['Vacío', 'Medio', 'Lleno'])
                matriz_p_df.to_excel(writer, sheet_name='Matriz_P')
                
                # Hoja 3: Puntos
                puntos_ordenados = self.puntos.sort_values('Nodo')
                puntos_ordenados.to_excel(writer, sheet_name='Puntos', index=False)
                
                # Hoja 4: Conversión Macetas
                conversion = pd.DataFrame({
                    'Macetas': list(range(1, 21)),
                    'Gramos necesarios': [i * 180 for i in range(1, 21)],
                    'Equivalencia': [f"{i} macetas = {i*180}g" for i in range(1, 21)]
                })
                conversion.to_excel(writer, sheet_name='Conversión_Macetas', index=False)
            
            messagebox.showinfo("Éxito", f"Datos exportados a {filename}")
            
        except Exception as e:
            messagebox.showerror("Error", f"No se pudo exportar: {str(e)}")
    
    def reiniciar_valores(self):
        """Reinicia todos los valores"""
        respuesta = messagebox.askyesno("Confirmar", "¿Reiniciar todos los valores?")
        
        if respuesta:
            self.matriz_p = np.array([
                [0.1, 0.6, 0.3],
                [0.2, 0.5, 0.3],
                [0.4, 0.4, 0.2]
            ])
            
            for i in range(3):
                for j in range(3):
                    self.entradas_matriz[i][j].set(f"{self.matriz_p[i, j]:.3f}")
            
            self.dias_transcurridos = 1
            self.dias_var.set("1")
            
            self.meta_usuario = 4000.0
            self.macetas_deseadas = int(4000 / 180)
            self.macetas_var.set(str(self.macetas_deseadas))
            
            self.puntos = pd.DataFrame({
                'Nodo': [1, 2, 3, 4, 5, 6, 7, 8],
                'Punto': ['Restaurante "A"', 'Pensión Central', 'Casa (Disperso)', 
                         'Pastelería', 'Cafetería Univ.', 'Mercado Local', 'Sky Box', 'Ramen'],
                'Cantidad_g': [1200, 800, 400, 2000, 900, 1800, 600, 1500],
                'Tiempo_min': [15, 20, 25, 10, 12, 25, 10, 18],
                'Capacidad_Max_g': [1500, 1000, 600, 2500, 1200, 2000, 800, 2000],
                'Estado': ['Medio', 'Medio', 'Vacío', 'Lleno', 'Medio', 'Lleno', 'Vacío', 'Medio']
            })
            
            self.recalcular_todo()
            messagebox.showinfo("Éxito", "Valores reiniciados")

# ============================================================================
# FUNCIONES PRINCIPALES
# ============================================================================

def iniciar_calculadora(modo):
    """Inicia la calculadora en el modo especificado"""
    root_calc = tk.Tk()
    app = CalculadoraEcoLlajta(root_calc, modo)
    root_calc.mainloop()

def iniciar_aplicacion():
    """Función principal que inicia la selección de modo"""
    sistema = SistemaAcceso()
    root_seleccion = tk.Tk()
    ventana_seleccion = VentanaSeleccionModo(root_seleccion, sistema, iniciar_calculadora)
    root_seleccion.mainloop()

# ============================================================================
# EJECUCIÓN
# ============================================================================

if __name__ == "__main__":
    iniciar_aplicacion()
