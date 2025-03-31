# Ajedrez
import chess
import chess.svg
import random
import tkinter as tk
from tkinter import messagebox
from PIL import Image, ImageTk
import io
import cairosvg

class AjedrezDidactico:
    def __init__(self, root):
        self.root = root
        self.root.title("Robot de Ajedrez Didáctico")
        self.board = chess.Board()
        self.nivel_dificultad = 1  # 1-3 (principiante, intermedio, avanzado)
        self.modo_aprendizaje = "aperturas"  # aperturas, tácticas, finales
        
        # Diccionario de aperturas comunes con explicaciones
        self.aperturas = {
            "e4": {
                "nombre": "Apertura del Peón de Rey",
                "explicacion": "Una de las aperturas más populares. Controla el centro y libera el alfil y la dama.",
                "continuaciones": {
                    "e5": "Respuesta simétrica, también luchando por el centro.",
                    "c5": "Defensa Siciliana. Contraataca el centro sin jugar simétrico.",
                    "e6": "Defensa Francesa. Sólida pero algo restrictiva para el alfil de casillas blancas."
                }
            },
            "d4": {
                "nombre": "Apertura del Peón de Dama",
                "explicacion": "Controla el centro y prepara un desarrollo más lento pero sólido.",
                "continuaciones": {
                    "d5": "Respuesta simétrica, buscando equilibrio.",
                    "Nf6": "Defensa India. Flexible y permite varias estructuras.",
                    "f5": "Defensa Holandesa. Agresiva pero puede debilitar el enroque."
                }
            },
            "c4": {
                "nombre": "Apertura Inglesa",
                "explicacion": "Controla d5 sin comprometer el centro. Flexible y posicional.",
                "continuaciones": {
                    "e5": "Juego abierto y activo.",
                    "c5": "Juego simétrico que lleva a posiciones equilibradas."
                }
            }
        }
        
        # Diccionario de trampas y "manzanitas envenenadas" comunes
        self.trampas = {
            "Gambito de Dama": {
                "posicion": "rnbqkbnr/ppp1pppp/8/3p4/2PP4/8/PP2PPPP/RNBQKBNR b KQkq - 0 2",
                "trampa": "Si negras capturan dxc4, blancas pueden recuperar el peón con ventaja mediante e3 y luego Alfil x c4",
                "explicacion": "Esta es una manzanita envenenada clásica. El peón parece libre, pero capturarlo permite un desarrollo rápido a las blancas."
            },
            "Mate del Pastor": {
                "posicion": "rnbqkbnr/pppp1ppp/8/4p3/4P3/5N2/PPPP1PPP/RNBQKB1R b KQkq - 1 2",
                "trampa": "Si negras juegan incorrectamente, pueden sufrir un mate rápido con Bc4 y Dh5",
                "explicacion": "Una trampa clásica para principiantes. Las blancas amenazan mate en f7 si las negras no defienden adecuadamente."
            }
        }
        
        # Configuración de la interfaz
        self.setup_ui()
        
    def setup_ui(self):
        # Frame principal
        main_frame = tk.Frame(self.root)
        main_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)
        
        # Frame izquierdo para el tablero
        board_frame = tk.Frame(main_frame)
        board_frame.pack(side=tk.LEFT, padx=10)
        
        # Tablero
        self.canvas = tk.Canvas(board_frame, width=400, height=400, bg="white")
        self.canvas.pack()
        
        # Frame derecho para controles e información
        control_frame = tk.Frame(main_frame)
        control_frame.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True, padx=10)
        
        # Sección de información de jugada
        info_frame = tk.LabelFrame(control_frame, text="Información", padx=5, pady=5)
        info_frame.pack(fill=tk.X, pady=5)
        
        self.info_text = tk.Text(info_frame, height=10, width=40, wrap=tk.WORD)
        self.info_text.pack(fill=tk.BOTH, expand=True)
        self.info_text.insert(tk.END, "¡Bienvenido al Robot de Ajedrez Didáctico!\n")
        self.info_text.insert(tk.END, "Estoy aquí para jugar y enseñarte ajedrez.\n")
        self.info_text.insert(tk.END, "Empecemos con algunas aperturas básicas.")
        self.info_text.config(state=tk.DISABLED)
        
        # Sección de controles
        control_panel = tk.LabelFrame(control_frame, text="Controles", padx=5, pady=5)
        control_panel.pack(fill=tk.X, pady=5)
        
        # Modo de aprendizaje
        tk.Label(control_panel, text="Modo de aprendizaje:").grid(row=0, column=0, sticky=tk.W)
        self.modo_var = tk.StringVar(value="aperturas")
        tk.Radiobutton(control_panel, text="Aperturas", variable=self.modo_var, value="aperturas", command=self.cambiar_modo).grid(row=0, column=1, sticky=tk.W)
        tk.Radiobutton(control_panel, text="Tácticas", variable=self.modo_var, value="tacticas", command=self.cambiar_modo).grid(row=0, column=2, sticky=tk.W)
        
        # Nivel de dificultad
        tk.Label(control_panel, text="Dificultad:").grid(row=1, column=0, sticky=tk.W)
        self.nivel_var = tk.IntVar(value=1)
        tk.Radiobutton(control_panel, text="Principiante", variable=self.nivel_var, value=1).grid(row=1, column=1, sticky=tk.W)
        tk.Radiobutton(control_panel, text="Intermedio", variable=self.nivel_var, value=2).grid(row=1, column=2, sticky=tk.W)
        
        # Entrada para movimientos
        move_frame = tk.Frame(control_panel)
        move_frame.grid(row=2, column=0, columnspan=3, pady=5, sticky=tk.W+tk.E)
        
        tk.Label(move_frame, text="Tu jugada:").pack(side=tk.LEFT)
        self.move_entry = tk.Entry(move_frame, width=10)
        self.move_entry.pack(side=tk.LEFT, padx=5)
        self.move_button = tk.Button(move_frame, text="Mover", command=self.hacer_jugada_usuario)
        self.move_button.pack(side=tk.LEFT)
        
        # Botones adicionales
        button_frame = tk.Frame(control_panel)
        button_frame.grid(row=3, column=0, columnspan=3, pady=5)
        
        self.nueva_partida_btn = tk.Button(button_frame, text="Nueva Partida", command=self.nueva_partida)
        self.nueva_partida_btn.pack(side=tk.LEFT, padx=5)
        
        self.consejo_btn = tk.Button(button_frame, text="Pedir Consejo", command=self.dar_consejo)
        self.consejo_btn.pack(side=tk.LEFT, padx=5)
        
        self.trampa_btn = tk.Button(button_frame, text="Mostrar Trampa", command=self.mostrar_trampa)
        self.trampa_btn.pack(side=tk.LEFT, padx=5)
        
        # Actualizar el tablero
        self.actualizar_tablero()
        
    def actualizar_tablero(self):
        # Generar SVG del tablero
        svg_data = chess.svg.board(self.board, size=400)
        
        # Convertir SVG a PNG
        png_data = cairosvg.svg2png(bytestring=svg_data.encode('utf-8'))
        
        # Convertir PNG a imagen para Tkinter
        img = Image.open(io.BytesIO(png_data))
        self.board_image = ImageTk.PhotoImage(img)
        
        # Mostrar en el canvas
        self.canvas.create_image(0, 0, anchor=tk.NW, image=self.board_image)
        
    def actualizar_info(self, texto):
        self.info_text.config(state=tk.NORMAL)
        self.info_text.delete(1.0, tk.END)
        self.info_text.insert(tk.END, texto)
        self.info_text.config(state=tk.DISABLED)
        
    def cambiar_modo(self):
        modo = self.modo_var.get()
        if modo == "aperturas":
            self.actualizar_info("Modo de Aperturas activado.\nTe ayudaré a aprender las principales aperturas y sus ideas estratégicas.")
        elif modo == "tacticas":
            self.actualizar_info("Modo de Tácticas activado.\nTe mostraré manzanitas envenenadas y trucos tácticos comunes.")
            
    def hacer_jugada_usuario(self):
        movimiento = self.move_entry.get().strip()
        try:
            move = chess.Move.from_uci(movimiento)
            if move in self.board.legal_moves:
                # Verificar si es una "manzanita envenenada"
                self.detectar_manzanita_envenenada(move)
                
                # Hacer el movimiento
                self.board.push(move)
                self.move_entry.delete(0, tk.END)
                
                # Si es principio de partida y estamos en modo aperturas, dar explicación
                if self.board.fullmove_number <= 3 and self.modo_var.get() == "aperturas":
                    self.explicar_apertura()
                
                # Respuesta del robot
                self.hacer_jugada_robot()
                
                # Actualizar tablero
                self.actualizar_tablero()
            else:
                messagebox.showerror("Error", "Movimiento ilegal")
        except Exception as e:
            messagebox.showerror("Error", f"Error al procesar el movimiento: {str(e)}")
            
    def detectar_manzanita_envenenada(self, move):
        # Detección básica de capturas que podrían ser manzanitas envenenadas
        if self.board.is_capture(move):
            pieza_a_capturar = self.board.piece_at(move.to_square)
            if pieza_a_capturar:
                # Verificar si la captura deja al rey expuesto o permite una táctica
                board_temp = self.board.copy()
                board_temp.push(move)
                
                # Verificar si hay jaque inmediato después de capturar
                if board_temp.is_check():
                    self.actualizar_info("¡Cuidado! Esa captura deja a tu rey en jaque.\n"
                                        "Esta es una 'manzanita envenenada' clásica.\n"
                                        "Siempre verifica si una captura expone a tu rey.")
                    return True
                
                # Verificar si hay una horqueta después de la captura
                posibles_respuestas = list(board_temp.legal_moves)
                for respuesta in posibles_respuestas:
                    if board_temp.is_capture(respuesta) and board_temp.piece_at(respuesta.to_square).piece_type > pieza_a_capturar.piece_type:
                        self.actualizar_info("¡Atención! Al capturar esa pieza, tu oponente puede ganar material superior.\n"
                                            "Esta es una 'manzanita envenenada'. La pieza parece fácil de tomar,\n"
                                            "pero hay una trampa táctica detrás.")
                        return True
        return False
    
    def explicar_apertura(self):
        # Convertir posición a FEN para identificar apertura
        fen = self.board.fen().split(' ')[0]  # Solo usar la parte de posición
        
        # Buscar la posición en las aperturas conocidas
        for ap_key, apertura in self.aperturas.items():
            if ap_key in ' '.join([move.uci() for move in self.board.move_stack]):
                info = f"Estás jugando la {apertura['nombre']}.\n{apertura['explicacion']}\n\n"
                
                # Agregar posibles continuaciones
                info += "Posibles continuaciones:\n"
                for cont_key, desc in apertura['continuaciones'].items():
                    info += f"- {cont_key}: {desc}\n"
                    
                self.actualizar_info(info)
                return
        
        # Si no se encuentra en las aperturas conocidas
        self.actualizar_info("Estás explorando una apertura no estándar o variante menos común.\n"
                            "Recuerda los principios de apertura:\n"
                            "1. Controla el centro\n"
                            "2. Desarrolla tus piezas\n"
                            "3. Protege tu rey (enroque)")
                
    def hacer_jugada_robot(self):
        if self.board.is_game_over():
            resultado = "Tablas"
            if self.board.is_checkmate():
                resultado = "Ganas tú" if self.board.turn == chess.BLACK else "Gano yo"
            messagebox.showinfo("Fin de la partida", f"Partida terminada. Resultado: {resultado}")
            return
            
        # Elegir una jugada según el nivel
        if self.nivel_var.get() == 1:  # Principiante - jugadas simples
            jugadas = list(self.board.legal_moves)
            move = random.choice(jugadas)
        else:  # Intermedio - jugadas más inteligentes
            # Preferir capturas y jaques
            capturas = [m for m in self.board.legal_moves if self.board.is_capture(m)]
            jaques = [m for m in self.board.legal_moves if self.board.gives_check(m)]
            
            if jaques and random.random() > 0.5:
                move = random.choice(jaques)
            elif capturas:
                move = random.choice(capturas)
            else:
                move = random.choice(list(self.board.legal_moves))
        
        # Hacer la jugada
        self.board.push(move)
        
        # Dar feedback educativo
        if self.board.is_check():
            self.actualizar_info(f"¡Jaque! Mi movimiento {move.uci()} ataca a tu rey.\n"
                                "Debes proteger tu rey o salir del jaque.")
        elif self.board.is_capture(move):
            pieza_capturada = self.board.piece_at(move.to_square)
            self.actualizar_info(f"He capturado una pieza con {move.uci()}.\n"
                                "Siempre busca oportunidades para capturar material\n"
                                "cuando sea seguro hacerlo.")
        elif self.board.fullmove_number <= 3 and self.modo_var.get() == "aperturas":
            # Dar un consejo sobre la apertura
            self.explicar_apertura()
        else:
            self.actualizar_info(f"He jugado {move.uci()}.\n"
                                "Es tu turno. Piensa en el desarrollo de tus piezas\n"
                                "y el control del centro.")
                
    def nueva_partida(self):
        self.board = chess.Board()
        self.actualizar_tablero()
        self.actualizar_info("¡Nueva partida comenzada!\nEstoy listo para enseñarte mientras jugamos.")
        
    def dar_consejo(self):
        if self.board.is_game_over():
            self.actualizar_info("La partida ha terminado. Iniciemos una nueva para seguir aprendiendo.")
            return
            
        # Analizar posición para dar un consejo
        if self.board.fullmove_number <= 5:
            self.actualizar_info("Consejo de apertura:\n"
                                "1. Desarrolla tus piezas hacia el centro\n"
                                "2. No muevas la misma pieza varias veces en la apertura\n"
                                "3. El enroque temprano suele ser una buena idea para proteger al rey")
        else:
            if len(self.board.piece_map()) < 10:  # Final
                self.actualizar_info("Consejo de final:\n"
                                    "1. Activa tu rey, es una pieza fuerte en el final\n"
                                    "2. Los peones pasados son muy valiosos\n"
                                    "3. La oposición de reyes es un concepto clave")
            else:  # Medio juego
                self.actualizar_info("Consejo de medio juego:\n"
                                    "1. Busca debilidades en la estructura de peones rival\n"
                                    "2. Centraliza tus piezas para maximizar su actividad\n"
                                    "3. Antes de cada captura, verifica si hay alguna táctica oculta")
                
    def mostrar_trampa(self):
        # Mostrar una trampa aleatoria del diccionario
        if self.trampas:
            trampa_nombre, trampa_info = random.choice(list(self.trampas.items()))
            
            texto = f"Trampa: {trampa_nombre}\n\n"
            texto += f"{trampa_info['trampa']}\n\n"
            texto += f"Explicación: {trampa_info['explicacion']}"
            
            self.actualizar_info(texto)
            
            # Opcionalmente, mostrar la posición de la trampa
            respuesta = messagebox.askyesno("Ver posición", "¿Quieres ver la posición de esta trampa en el tablero?")
            if respuesta:
                self.board = chess.Board(trampa_info['posicion'])
                self.actualizar_tablero()

# Función principal para ejecutar la aplicación
def main():
    root = tk.Tk()
    app = AjedrezDidactico(root)
    root.mainloop()

if __name__ == "__main__":
    main()
