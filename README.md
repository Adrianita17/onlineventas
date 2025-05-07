# onlineventas
import sys
import os
from PySide6.QtWidgets import (
    QApplication, QMainWindow, QWidget, QLabel, QPushButton,
    QVBoxLayout, QScrollArea, QGridLayout, QHBoxLayout,
    QDialog, QLineEdit, QTextEdit, QMessageBox
)
from PySide6.QtGui import QIcon, QPixmap
from PySide6.QtCore import Qt

# --------------------
# Rutas de Recursos
# --------------------

RUTA_RECURSOS = os.path.join(os.path.dirname(__file__), 'recursos')

# --------------------
# Modelo de Producto
# --------------------

class Producto:
    def __init__(self, nombre, descripcion, precio, imagen):
        self.nombre = nombre
        self.descripcion = descripcion
        self.precio = precio
        self.imagen = imagen  # ruta al archivo de imagen

# --------------------
# Productos por Categoría
# --------------------

PRODUCTOS = {
    'Ropa Hombre': [
        Producto('Camisa Hombre', 'Algodón 100%', 30000, os.path.join(RUTA_RECURSOS, 'camisa_hombre.jpg')),
        Producto('Pantalón Hombre', 'Jean stretch', 70000, os.path.join(RUTA_RECURSOS, 'pantalon_hombre.jpg')),
        Producto('Chaqueta Hombre', 'Cuero sintético', 120000, os.path.join(RUTA_RECURSOS, 'chaqueta_hombre.jpg')),
        Producto('Sudadera Hombre', 'Con capucha', 50000, os.path.join(RUTA_RECURSOS, 'sudadera_hombre.jpg')),
        Producto('Polo Hombre', 'Poliéster', 25000, os.path.join(RUTA_RECURSOS, 'polo_hombre.jpg')),
    ],
    'Ropa Mujer': [
        Producto('Blusa Mujer', 'Seda', 35000, os.path.join(RUTA_RECURSOS, 'blusa_mujer.jpg')),
        Producto('Falda Mujer', 'Denim', 45000, os.path.join(RUTA_RECURSOS, 'falda_mujer.jpg')),
        Producto('Vestido Mujer', 'Algodón', 80000, os.path.join(RUTA_RECURSOS, 'vestido_mujer.jpg')),
        Producto('Chaqueta Mujer', 'Tweed', 90000, os.path.join(RUTA_RECURSOS, 'chaqueta_mujer.jpg')),
        Producto('Pantalón Mujer', 'Lino', 60000, os.path.join(RUTA_RECURSOS, 'pantalon_mujer.jpg')),
    ],
    'Accesorios Hombre': [
        Producto('Gorra Negra', 'Ajustable', 18000, os.path.join(RUTA_RECURSOS, 'gorra.jpg')),
        Producto('Cinturón Hombre', 'Cuero', 25000, os.path.join(RUTA_RECURSOS, 'cinturon_hombre.jpg')),
        Producto('Reloj Hombre', 'Analógico', 150000, os.path.join(RUTA_RECURSOS, 'reloj_hombre.jpg')),
    ],
    'Accesorios Mujer': [
        Producto('Collar Elegante', 'Perlas', 22000, os.path.join(RUTA_RECURSOS, 'collar.jpg')),
        Producto('Pendientes Mujer', 'Agujeros', 15000, os.path.join(RUTA_RECURSOS, 'pendientes_mujer.jpg')),
        Producto('Pulsera Mujer', 'Cuero', 18000, os.path.join(RUTA_RECURSOS, 'pulsera_mujer.jpg')),
    ],
    'Calzado Unisex': [
        Producto('Zapatillas Deportivas', 'Cómodas y ligeras', 50000, os.path.join(RUTA_RECURSOS, 'zapatillas.jpg')),
        Producto('Botas Unisex', 'Impermeables', 80000, os.path.join(RUTA_RECURSOS, 'botas_unisex.jpg')),
    ]
}

# --------------------
# Gestión de Carrito Simplificado
# --------------------

class Carrito:
    _items = []

    @classmethod
    def agregar(cls, producto):
        cls._items.append(producto)

    @classmethod
    def eliminar(cls, idx):
        if 0 <= idx < len(cls._items):
            cls._items.pop(idx)

    @classmethod
    def obtener(cls):
        return cls._items.copy()

    @classmethod
    def total(cls):
        return sum(p.precio for p in cls._items)

    @classmethod
    def vaciar(cls):
        cls._items.clear()

# --------------------
# Diálogo Finalizar Compra
# --------------------

class FormularioDialog(QDialog):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Finalizar Compra")
        self.setFixedSize(400, 300)
        layout = QVBoxLayout(self)
        layout.addWidget(QLabel("Nombre Completo:"))
        self.nombre = QLineEdit(); layout.addWidget(self.nombre)
        layout.addWidget(QLabel("Correo Electrónico:"))
        self.correo = QLineEdit(); layout.addWidget(self.correo)
        layout.addWidget(QLabel("Dirección de Entrega:"))
        self.direccion = QTextEdit(); layout.addWidget(self.direccion)

        btn_confirmar = QPushButton("Confirmar")
        btn_confirmar.setStyleSheet("background-color: #4CAF50; color: white; font-weight: bold;")
        btn_confirmar.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'checkout_icon.png')))
        btn_confirmar.clicked.connect(self.confirmar)
        layout.addWidget(btn_confirmar)

    def confirmar(self):
        if not self.nombre.text() or not self.correo.text() or not self.direccion.toPlainText():
            QMessageBox.warning(self, "Campos incompletos", "Complete todos los campos.")
            return
        QMessageBox.information(self, "Éxito", "Compra realizada con éxito.")
        Carrito.vaciar()
        self.accept()

# --------------------
# Diálogo Carrito
# --------------------

class CarritoDialog(QDialog):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Carrito de Compras")
        self.setFixedSize(450, 500)
        self.layout = QVBoxLayout(self)
        self._actualizar()

    def _actualizar(self):
        # limpiar
        for i in reversed(range(self.layout.count())):
            w = self.layout.itemAt(i).widget()
            if w: w.setParent(None)
        # items
        for idx, p in enumerate(Carrito.obtener()):
            row = QHBoxLayout()
            row.addWidget(QLabel(f"{p.nombre} — ${p.precio:,}"))
            btn_eliminar = QPushButton("Eliminar")
            btn_eliminar.setStyleSheet("background-color: #F44336; color: white; font-weight: bold;")
            btn_eliminar.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'delete_icon.png')))
            btn_eliminar.clicked.connect(lambda _, i=idx: self._eliminar(i))
            row.addWidget(btn_eliminar)
            container = QWidget(); container.setLayout(row)
            self.layout.addWidget(container)
        # total y finalizar
        total_label = QLabel(f"Total: ${Carrito.total():,}")
        total_label.setAlignment(Qt.AlignRight)
        self.layout.addWidget(total_label)
        btn_fin = QPushButton("Finalizar Compra")
        btn_fin.setStyleSheet("background-color: #009688; color: white; font-weight: bold;")
        btn_fin.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'checkout_icon.png')))
        btn_fin.clicked.connect(self._finalizar)
        self.layout.addWidget(btn_fin)
        btn_regresar = QPushButton("Regresar")
        btn_regresar.setStyleSheet("background-color: #FF9800; color: white; font-weight: bold;")
        btn_regresar.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'back_icon.png')))
        btn_regresar.clicked.connect(self.close)
        self.layout.addWidget(btn_regresar)

    def _eliminar(self, i):
        Carrito.eliminar(i)
        self._actualizar()

    def _finalizar(self):
        dlg = FormularioDialog()
        if dlg.exec():
            self._actualizar()

# --------------------
# Diálogo Opciones de Ropa
# --------------------

class DialogoOpcionesRopa(QDialog):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Selecciona una Categoría de Ropa")
        self.setFixedSize(300, 150)

        layout = QVBoxLayout(self)

        btn_hombre = QPushButton("Ropa Hombre")
        btn_hombre.setStyleSheet("background-color: #4CAF50; color: white; font-weight: bold;" )
        btn_hombre.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'mens_icon.png')))
        btn_hombre.clicked.connect(self.abrir_ropa_hombre)
        layout.addWidget(btn_hombre)

        btn_mujer = QPushButton("Ropa Mujer")
        btn_mujer.setStyleSheet("background-color: #2196F3; color: white; font-weight: bold;")
        btn_mujer.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'womens_icon.png')))
        btn_mujer.clicked.connect(self.abrir_ropa_mujer)
        layout.addWidget(btn_mujer)

    def abrir_ropa_hombre(self):
        productos_hombre = PRODUCTOS['Ropa Hombre']
        cat = VentanaCatalogo(productos_hombre, 'Ropa Hombre')
        cat.show()
        self.close()

    def abrir_ropa_mujer(self):
        productos_mujer = PRODUCTOS['Ropa Mujer']
        cat = VentanaCatalogo(productos_mujer, 'Ropa Mujer')
        cat.show()
        self.close()

# --------------------
# Catálogo en Cuadrícula
# --------------------

class VentanaCatalogo(QWidget):
    def __init__(self, productos, categoria):
        super().__init__()
        self.setWindowTitle(f"Catálogo — {categoria}")
        self.setMinimumSize(600, 400)
        self.productos = productos
        self._init_ui()

    def _init_ui(self):
        v = QVBoxLayout(self)
        scroll = QScrollArea(); scroll.setWidgetResizable(True)
        cont = QWidget(); grid = QGridLayout(cont); grid.setSpacing(12)
        filas = columnas = 0
        for p in self.productos:
            w = self._crear_widget(p)
            grid.addWidget(w, filas, columnas)
            columnas = (columnas + 1) % 3
            if columnas == 0: filas += 1
        scroll.setWidget(cont); v.addWidget(scroll)

        btn_regresar = QPushButton("Regresar")
        btn_regresar.setStyleSheet("background-color: #FF9800; color: white; font-weight: bold;")
        btn_regresar.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'back_icon.png')))
        btn_regresar.clicked.connect(self.close)
        v.addWidget(btn_regresar)

    def _crear_widget(self, p):
        w = QWidget(); vb = QVBoxLayout(w)
        lbl = QLabel()
        if os.path.exists(p.imagen):
            pix = QPixmap(p.imagen).scaled(150, 150, Qt.KeepAspectRatio, Qt.SmoothTransformation)
            lbl.setPixmap(pix)
        else:
            lbl.setText("Sin imagen")
        lbl.setAlignment(Qt.AlignCenter)
        title = QLabel(p.nombre); title.setAlignment(Qt.AlignCenter); title.setStyleSheet("font-weight:bold;")
        price = QLabel(f"${p.precio:,}"); price.setAlignment(Qt.AlignCenter)
        btn = QPushButton("Agregar al carrito")
        btn.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'cart_icon.png')))
        btn.setStyleSheet("background-color: #FF5722; color: white; font-weight: bold;")
        btn.clicked.connect(lambda _, prod=p: QMessageBox.information(self, "Agregado", f"{prod.nombre} agregado.") or Carrito.agregar(prod))
        vb.addWidget(lbl); vb.addWidget(title); vb.addWidget(price); vb.addWidget(btn)
        return w

# --------------------
# Splash de Bienvenida
# --------------------

class VentanaBienvenida(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Bienvenida a FashionShopping Online")
        self.setFixedSize(600, 400)
        self.setStyleSheet("background-color: black;")
        titulo = QLabel("¡Bienvenido a FashionShopping Online!", self)
        titulo.setStyleSheet("font-size:24px; color:white; font-weight: bold;")
        titulo.setAlignment(Qt.AlignCenter); titulo.setGeometry(0, 50, 600, 50)
        btn = QPushButton("Entrar", self)
        btn.setStyleSheet("background-color: #FF4081; color: white; font-weight: bold;")
        btn.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'enter_icon.png')))
        btn.setGeometry(250, 300, 100, 40)
        btn.clicked.connect(self._abrir_main)

    def _abrir_main(self):
        self.main = VentanaPrincipal()
        self.main.show()
        self.close()

# --------------------
# Ventana Principal
# --------------------

class VentanaPrincipal(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("FashionShopping Online")
        self.setFixedSize(400, 300)
        self._init_ui()

    def _init_ui(self):
        central = QWidget(); v = QVBoxLayout(central)
        btn_ropa = QPushButton("Ropa")
        btn_ropa.setStyleSheet("background-color: #3F51B5; color: white; font-weight: bold;")
        btn_ropa.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'clothes_icon.png')))
        btn_ropa.clicked.connect(self._abrir_ropa)
        v.addWidget(btn_ropa)
        btn_accesorios_hombre = QPushButton("Accesorios Hombre")
        btn_accesorios_hombre.setStyleSheet("background-color: #673AB7; color: white; font-weight: bold;")
        btn_accesorios_hombre.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'mens_accessories_icon.png')))
        btn_accesorios_hombre.clicked.connect(self._abrir_accesorios_hombre)
        v.addWidget(btn_accesorios_hombre)
        btn_accesorios_mujer = QPushButton("Accesorios Mujer")
        btn_accesorios_mujer.setStyleSheet("background-color: #009688; color: white; font-weight: bold;")
        btn_accesorios_mujer.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'womens_accessories_icon.png')))
        btn_accesorios_mujer.clicked.connect(self._abrir_accesorios_mujer)
        v.addWidget(btn_accesorios_mujer)
        btn_calzado = QPushButton("Calzado Unisex")
        btn_calzado.setStyleSheet("background-color: #FF9800; color: white; font-weight: bold;")
        btn_calzado.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'shoes_icon.png')))
        btn_calzado.clicked.connect(self._abrir_calzado)
        v.addWidget(btn_calzado)
        btn_cart = QPushButton("Ver Carrito")
        btn_cart.setIcon(QIcon(os.path.join(RUTA_RECURSOS, 'cart_icon.png')))
        btn_cart.setStyleSheet("background-color: #FF5722; color: white; font-weight: bold;")
        btn_cart.clicked.connect(lambda: CarritoDialog().exec())
        v.addWidget(btn_cart)
        self.setCentralWidget(central)
        central.setLayout(v)

    def _abrir_ropa(self):
        dialogo = DialogoOpcionesRopa()
        dialogo.exec()

    def _abrir_accesorios_hombre(self):
        cat = VentanaCatalogo(PRODUCTOS['Accesorios Hombre'], 'Accesorios Hombre')
        cat.show()

    def _abrir_accesorios_mujer(self):
        cat = VentanaCatalogo(PRODUCTOS['Accesorios Mujer'], 'Accesorios Mujer')
        cat.show()

    def _abrir_calzado(self):
        cat = VentanaCatalogo(PRODUCTOS['Calzado Unisex'], 'Calzado Unisex')
        cat.show()

# --------------------
# Ejecución de la Aplicación
# --------------------

if __name__ == "__main__":
    app = QApplication(sys.argv)
    ventana_bienvenida = VentanaBienvenida()
    ventana_bienvenida.show()
    sys.exit(app.exec())
