import mysql.connector
from mysql.connector import Error

class ConexionBD:
    def __init__(self):
        self.host = "localhost"
        self.user = "root"
        self.password = ""
        self.database = "almacen_inteligente"
        self.inicializar_bd()

    def inicializar_bd(self):
        try:
            conn = mysql.connector.connect(
                host=self.host,
                user=self.user,
                password=self.password
            )
            cursor = conn.cursor()
            cursor.execute(f"CREATE DATABASE IF NOT EXISTS {self.database}")
            cursor.execute(f"USE {self.database}")

            cursor.execute("""
                CREATE TABLE IF NOT EXISTS ubicaciones (
                    id_ubicacion INT AUTO_INCREMENT PRIMARY KEY,
                    pasillo INT NOT NULL,
                    nivel INT NOT NULL,
                    ocupado BOOLEAN DEFAULT FALSE
                )
            """)

            cursor.execute("""
                CREATE TABLE IF NOT EXISTS productos (
                    codigo VARCHAR(50) PRIMARY KEY,
                    nombre VARCHAR(100) NOT NULL,
                    rotacion INT NOT NULL,
                    id_ubicacion INT UNIQUE,
                    FOREIGN KEY (id_ubicacion) REFERENCES ubicaciones(id_ubicacion) ON DELETE SET NULL
                )
            """)

            cursor.execute("SELECT COUNT(*) FROM ubicaciones")
            if cursor.fetchone()[0] == 0:
                cursor.executemany(
                    "INSERT INTO ubicaciones (pasillo, nivel, ocupado) VALUES (%s, %s, %s)",
                    [(0, 0, False), (0, 1, False), (1, 0, False), (1, 1, False)]
                )

            conn.commit()
            cursor.close()
            conn.close()
        except Error as e:
            print(f"\n[ERROR AL INICIALIZAR BD] {e}")

    def conectar(self):
        try:
            conexion = mysql.connector.connect(
                host=self.host,
                user=self.user,
                password=self.password,
                database=self.database
            )
            return conexion
        except Error as e:
            print(f"\n[ERROR DE CONEXIÓN] No se pudo conectar a MySQL: {e}")
            return None


class Producto:
    def __init__(self, codigo, nombre, rotacion, id_ubicacion=None):
        self.codigo = codigo
        self.nombre = nombre
        self.rotacion = rotacion
        self.id_ubicacion = id_ubicacion


class Almacen:
    def __init__(self):
        self.db = ConexionBD()

    def registrar_producto(self, producto):
        conn = self.db.conectar()
        if not conn:
            return
        try:
            cursor = conn.cursor()
            cursor.execute("SELECT id_ubicacion, pasillo, nivel FROM ubicaciones WHERE ocupado = FALSE LIMIT 1")
            ubicacion = cursor.fetchone()
            
            if not ubicacion:
                print("\n[AVISO] ¡El almacén está lleno! No hay ubicaciones disponibles.")
                return

            id_ubic, pasillo, nivel = ubicacion

            sql_insert = "INSERT INTO productos (codigo, nombre, rotacion, id_ubicacion) VALUES (%s, %s, %s, %s)"
            cursor.execute(sql_insert, (producto.codigo, producto.nombre, producto.rotacion, id_ubic))

            cursor.execute("UPDATE ubicaciones SET ocupado = TRUE WHERE id_ubicacion = %s", (id_ubic,))
            
            conn.commit()
            print(f"\n[ÉXITO] '{producto.nombre}' registrado en Pasillo {pasillo}, Nivel {nivel}.")
        except Error as e:
            print(f"\n[ERROR AL REGISTRAR] Verifique que el código no esté duplicado. Detalle: {e}")
        finally:
            conn.close()

    def listar_productos_con_ubicacion(self):
        conn = self.db.conectar()
        if not conn:
            return
        try:
            cursor = conn.cursor()
            sql_join = """
                SELECT p.codigo, p.nombre, p.rotacion, u.pasillo, u.nivel 
                FROM productos p
                INNER JOIN ubicaciones u ON p.id_ubicacion = u.id_ubicacion
                ORDER BY p.rotacion DESC
            """
            cursor.execute(sql_join)
            resultados = cursor.fetchall()

            if not resultados:
                print("\n[AVISO] No hay productos registrados en la base de datos.")
            else:
                print("\n--- INVENTARIO ORDENADO POR ROTACIÓN (CONSULTA JOIN) ---")
                for fila in resultados:
                    print(f"Código: {fila[0]} | Producto: {fila[1]} | Rotación: {fila[2]} | Ubicación: Pasillo {fila[3]}, Nivel {fila[4]}")
        except Error as e:
            print(f"\n[ERROR AL CONSULTAR] {e}")
        finally:
            conn.close()

    def actualizar_producto(self, codigo, nuevo_nombre, nueva_rotacion):
        conn = self.db.conectar()
        if not conn:
            return
        try:
            cursor = conn.cursor()
            sql_update = "UPDATE productos SET nombre = %s, rotacion = %s WHERE codigo = %s"
            cursor.execute(sql_update, (nuevo_nombre, nueva_rotacion, codigo))
            conn.commit()

            if cursor.rowcount > 0:
                print(f"\n[OK] Producto con código '{codigo}' actualizado correctamente.")
            else:
                print(f"\n[AVISO] No se encontró ningún producto con el código '{codigo}'.")
        except Error as e:
            print(f"\n[ERROR AL ACTUALIZAR] {e}")
        finally:
            conn.close()

    def eliminar_producto(self, codigo):
        conn = self.db.conectar()
        if not conn:
            return
        try:
            cursor = conn.cursor()
            cursor.execute("SELECT id_ubicacion FROM productos WHERE codigo = %s", (codigo,))
            resultado = cursor.fetchone()

            if not resultado:
                print(f"\n[AVISO] No existe un producto con código '{codigo}'.")
                return

            id_ubicacion = resultado[0]

            cursor.execute("DELETE FROM productos WHERE codigo = %s", (codigo,))
            
            if id_ubicacion:
                cursor.execute("UPDATE ubicaciones SET ocupado = FALSE WHERE id_ubicacion = %s", (id_ubicacion,))

            conn.commit()
            print(f"\n[OK] Producto '{codigo}' eliminado y estante liberado.")
        except Error as e:
            print(f"\n[ERROR AL ELIMINAR] {e}")
        finally:
            conn.close()

    def ver_espacios_vacios(self):
        conn = self.db.conectar()
        if not conn:
            return
        try:
            cursor = conn.cursor()
            cursor.execute("SELECT pasillo, nivel FROM ubicaciones WHERE ocupado = FALSE ORDER BY pasillo ASC, nivel ASC")
            vacios = cursor.fetchall()

            if not vacios:
                print("\n[AVISO] No quedan espacios vacíos. El almacén está lleno.")
            else:
                print(f"\n--- ESPACIOS DISPONIBLES ({len(vacios)} libres) ---")
                for fila in vacios:
                    print(f"Pasillo {fila[0]} | Nivel {fila[1]} -> [LIBRE]")
        except Error as e:
            print(f"\n[ERROR AL CONSULTAR ESPACIOS] {e}")
        finally:
            conn.close()


if __name__ == "__main__":
    almacen = Almacen()

    while True:
        print("\n=== SISTEMA DE ALMACÉN INTELIGENTE (MySQL + POO) ===")
        print("1. Registrar producto (INSERT)")
        print("2. Ver inventario ordenado y ubicaciones (SELECT + JOIN)")
        print("3. Actualizar producto (UPDATE)")
        print("4. Eliminar producto (DELETE)")
        print("5. Ver espacios vacíos (SELECT)")
        print("6. Salir")

        opcion = input("Selecciona una opción (1-6): ")

        if opcion == "1":
            cod = input("Código de barras: ").strip()
            nom = input("Nombre del producto: ").strip()
            try:
                rot = int(input("Nivel de rotación (número entero, ej. 90): "))
                nuevo_prod = Producto(cod, nom, rot)
                almacen.registrar_producto(nuevo_prod)
            except ValueError:
                print("\n[ERROR] La rotación debe ser un número entero.")

        elif opcion == "2":
            almacen.listar_productos_con_ubicacion()

        elif opcion == "3":
            cod = input("Ingresa el código del producto a modificar: ").strip()
            nom = input("Nuevo nombre del producto: ").strip()
            try:
                rot = int(input("Nuevo nivel de rotación: "))
                almacen.actualizar_producto(cod, nom, rot)
            except ValueError:
                print("\n[ERROR] La rotación debe ser un número entero.")

        elif opcion == "4":
            cod = input("Ingresa el código del producto a eliminar: ").strip()
            almacen.eliminar_producto(cod)

        elif opcion == "5":
            almacen.ver_espacios_vacios()

        elif opcion == "6":
            print("\n¡Saliendo del sistema!")
            break
        else:
            print("\n[ERROR] Opción inválida, intenta de nuevo.")
