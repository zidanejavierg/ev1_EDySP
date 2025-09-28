# ev1_EDySP
# Tasks Estructura de datos y  su procesamiento
import datetime

# -------------------- Datos en memoria --------------------
lista_clientes, lista_salas, lista_reservas = {}, {}, {}

contador_folio, contador_clientes, contador_salas = 1, 1, 1

# -------------------- Funciones auxiliares --------------------
def generar_folio():
    global contador_folio
    folio_actual = contador_folio
    contador_folio += 1
    return folio_actual

def validar_fecha(cadena, minimo2dias=True):
    try:
        fecha_ingresada = datetime.datetime.strptime(cadena, "%Y-%m-%d").date()
        if minimo2dias and fecha_ingresada < datetime.date.today() + datetime.timedelta(days=2):
            return None
        return fecha_ingresada
    except:
        return None

def mostrar_clientes():
    if not lista_clientes:
        print("No hay clientes registrados.")
        return []
    ordenados = sorted(lista_clientes.items(), key=lambda x: (x[1]['apellidos'], x[1]['nombre']))
    print("\nLista de clientes registrados:")
    for clave, datos in ordenados:
        print(f"{clave}: {datos['apellidos']} {datos['nombre']}")
    return [c[0] for c in ordenados]

def elegir_cliente():
    while True:
        claves_disponibles = mostrar_clientes()
        if not claves_disponibles: return None
        opcion = input("Ingresa la clave del cliente (o C para cancelar): ").upper()
        if opcion == "C": return None
        if opcion.isdigit() and int(opcion) in claves_disponibles:
            return int(opcion)
        print("Clave inválida, vuelve a intentarlo.")

def mostrar_salas_disponibles(fecha, turno):
    disponibles = []
    for clave, datos in lista_salas.items():
        ocupado = any(r['sala']==clave and r['fecha']==fecha and r['turno']==turno for r in lista_reservas.values())
        if not ocupado:
            print(f"{clave}: {datos['nombre']} (Cupo {datos['cupo']})")
            disponibles.append(clave)
    if not disponibles: print("No hay salas disponibles.")
    return disponibles

def elegir_sala(fecha, turno):
    while True:
        disponibles = mostrar_salas_disponibles(fecha, turno)
        if not disponibles: return None
        opcion = input("Ingresa la clave de la sala (o C para cancelar): ").upper()
        if opcion == "C": return None
        if opcion.isdigit() and int(opcion) in disponibles:
            return int(opcion)
        print("Clave inválida, vuelve a intentarlo.")

def reporte_fecha(fecha):
    print(f"\nReservas para el {fecha}:")
    print("Folio | Cliente               | Sala        | Turno | Evento")
    print("-"*65)
    encontrado = False
    for folio,res in lista_reservas.items():
        if res['fecha']==fecha:
            cli = lista_clientes[res['cliente']]
            sala = lista_salas[res['sala']]
            print(f"{folio:<5} | {cli['apellidos']} {cli['nombre']:<15} | {sala['nombre']:<10} | {res['turno']}     | {res['evento']}")
            encontrado = True
    if not encontrado:
        print("No hay reservas registradas para esa fecha.")

# -------------------- Funcionalidades --------------------
def registrar_cliente():
    global contador_clientes
    while True:
        nombre = input("Nombre del cliente: ").strip()
        if nombre: break
        print("El nombre no puede quedar vacío.")
    while True:
        apellidos = input("Apellidos del cliente: ").strip()
        if apellidos: break
        print("Los apellidos no pueden quedar vacíos.")
    lista_clientes[contador_clientes] = {"nombre":nombre, "apellidos":apellidos}
    print(f"Cliente registrado con clave {contador_clientes}")
    contador_clientes += 1

def registrar_sala():
    global contador_salas
    while True:
        nombre = input("Nombre de la sala: ").strip()
        if nombre: break
        print("El nombre de la sala no puede quedar vacío.")
    while True:
        try:
            cupo = int(input("Cupo de la sala: "))
            if cupo > 0: break
            else: print("El cupo debe ser mayor a 0.")
        except:
            print("Ingresa un número válido.")
    lista_salas[contador_salas] = {"nombre":nombre, "cupo":cupo}
    print(f"Sala registrada con clave {contador_salas}")
    contador_salas += 1

def registrar_reserva():
    cliente = elegir_cliente()
    if not cliente: return
    fecha = None
    while not fecha:
        fecha = validar_fecha(input("Fecha de la reserva (YYYY-MM-DD): "))
        if not fecha: print("Fecha inválida. Debe ser al menos 2 días después de hoy.")
    turno_elegido = input("Turno (M=Matutino, V=Vespertino, N=Nocturno): ").upper()
    if turno_elegido not in ["M","V","N"]:
        print("Turno inválido."); return
    sala = elegir_sala(fecha,turno_elegido)
    if not sala: return
    evento=""
    while not evento.strip():
        evento=input("Nombre del evento: ")
        if not evento.strip(): print("El evento no puede quedar vacío.")
    folio = generar_folio()
    lista_reservas[folio] = {"cliente":cliente,"sala":sala,"fecha":fecha,"turno":turno_elegido,"evento":evento}
    print(f"Reserva registrada con folio {folio}")

def editar_evento():
    f1 = validar_fecha(input("Fecha inicio (YYYY-MM-DD): "), False)
    f2 = validar_fecha(input("Fecha fin (YYYY-MM-DD): "), False)
    if not f1 or not f2 or f1>f2: print("Rango inválido."); return
    rango = {f:r for f,r in lista_reservas.items() if f1<=r['fecha']<=f2}
    if not rango: print("No hay reservas en ese rango."); return
    while True:
        print("Folio | Evento                | Fecha")
        for folio,res in rango.items():
            print(f"{folio:<5} | {res['evento']:<20} | {res['fecha']}")
        op = input("Folio a editar (o C para cancelar): ").upper()
        if op=="C": return
        if op.isdigit() and int(op) in rango:
            nuevo=""
            while not nuevo.strip():
                nuevo=input("Nuevo nombre del evento: ")
                if not nuevo.strip(): print("El nombre no puede quedar vacío.")
            lista_reservas[int(op)]['evento']=nuevo
            print("Evento actualizado.")
            return
        print("Folio inválido, vuelve a intentarlo.")

def consultar_reservas():
    fecha = validar_fecha(input("Fecha (YYYY-MM-DD): "), False)
    if not fecha: print("Fecha inválida."); return
    reporte_fecha(fecha)

# -------------------- Menú --------------------
def menu():
    while True:
        print("\n--- Menú Principal ---")
        print("1. Registrar nueva reserva de sala")
        print("2. Editar el nombre de un evento")
        print("3. Consultar reservas por fecha")
        print("4. Registrar nuevo cliente")
        print("5. Registrar nueva sala")
        print("6. Salir del sistema")
        opcion = input("Elige una opción: ")
        if opcion=="1": registrar_reserva()
        elif opcion=="2": editar_evento()
        elif opcion=="3": consultar_reservas()
        elif opcion=="4": registrar_cliente()
        elif opcion=="5": registrar_sala()
        elif opcion=="6": break
        else: print("Opción inválida.")

# -------------------- Inicio --------------------
menu()
