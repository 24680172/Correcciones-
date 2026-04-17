<div align="justify">
  
# Estructura del Proyecto
A continuación, se detallan los archivos principales y su contenido técnico basado en los documentos proporcionados:
  ```
POS_TAP/
├── main.py
├── data_manager.py
└── views/
    ├── gastos_view.py
    ├── dashboard_view.py
    └── historial_view.py
```
## Núcleo de la Aplicación
  **main.py**<br>
Este archivo es el orquestador. Configura la ventana de Flet, instancia el DataManager y gestiona la navegación entre las diferentes vistas mediante un NavigationRail.
```python
import flet as ft
from core.data_manager import DataManager
# ... importación de vistas

def main(page: ft.Page):
    page.title = "POS_TAP - Taller Flet"
    dm = DataManager() # Cerebro de datos único
    
    content_area = ft.Container(expand=True)

    def change_route(e):
        idx = e.control.selected_index
        # Lógica para intercambiar VentasView, GastosView, etc.
        page.update()

    # Barra lateral de navegación
    sidebar = ft.NavigationRail(
        destinations=[
            ft.NavigationRailDestination(icon=ft.Icons.SHOPPING_CART, label="Ventas"),
            ft.NavigationRailDestination(icon=ft.Icons.PAYMENT, label="Gastos"),
            # ... otros destinos
        ],
        on_change=change_route
    )
    page.add(ft.Row([sidebar, content_area], expand=True))

ft.run(main)
  ```
## Módulo de Ventas
**ventas_view.py**<br>
Gestiona el catálogo de productos y el carrito de compras. Incluye diálogos para agregar o eliminar platillos del inventario.

CartItemRow: Clase personalizada para manejar cada renglón del ticket (sumar, restar, borrar).

_cobrar: Método que utiliza el DataManager para persistir la transacción.
```python
class VentasView(ft.Container):
    def __init__(self, page, data_manager):
        self.dm = data_manager
        self.carrito = {}
        # Carga el inventario desde el JSON
        self.inventario = self.dm.get_inventario()
```
## Módulo de Analíticas
**dashboard_view.py**<br>
Transforma los datos en indicadores visuales (KPIs) y gráficos de barras para el análisis de ventas y gastos.
```python
class DashboardView(ft.Container):
    def _build_ui(self):
        data = self.dm.get_kpis_y_graficos()
        # Cálculo de Ganancia: Ventas - Gastos
        ganancia = data['ventas_hoy'] - data['gastos_hoy']
        
        # Gráfico de barras para el Top de Productos
        barras = ft.Column(controls=[...])
```
**Módulo de Gastos e Historial**
- gastos_view.py: Formulario con validación para registrar egresos. Asegura que el monto sea un número válido antes de procesarlo.
- historial_view.py: Lista cronológica de las ventas del día. Formatea la hora, los productos vendidos y el total.

## Persistencia de Datos (Archivos JSON)
El sistema utiliza archivos JSON como base de datos local para mantener la información sincronizada:

**inventario.json**<br>
Almacena el catálogo con precios y existencias.
```json
{
    "Mole Poblano": {"precio": 45, "stock": 98},
    "Tlayuda Oaxaquena": {"precio": 55, "stock": 100}
}
```
**ventas.json**
Guarda el registro histórico de transacciones.

```JSON
[
    {
        "fecha": "2026-04-16",
        "hora": "09:45",
        "productos": {"Mole Poblano": 1, "Enchiladas Verdes": 1},
        "total": 80
    }
]
```
**gastos.json**
Registra los egresos operativos (Luz, Agua, etc.).

```JSON
[
    {"fecha": "2026-04-16", "concepto": "LUZ", "monto": 900.0}
]
```

# Errores
A continuación, presento un desglose organizado de los 6 fallos detectados y sus soluciones:

1. **Interfaz y Framework (gastos_view.py)** <br>
El primer bloque se centra en la arquitectura técnica y el manejo de datos de entrada.

  - Importación de Iconos: El error radicaba en el uso de rutas de importación obsoletas para los iconos de Flet. Con las actualizaciones del framework, los iconos se centralizaron en ft.Icons. La solución fue simplificar el acceso, eliminando dependencias redundantes y apuntando directamente al espacio de nombres correcto.

  - Gestión de Tipos (String vs Float): Este es un error clásico de "limpieza de datos". Aunque el programa convertía el monto ingresado a decimal (float), al momento de registrar el gasto, seguía enviando el valor original de la caja de texto (un string). Al no usar la variable ya procesada, el sistema probablemente fallaba al realizar cálculos posteriores. La solución fue asegurar que la función registrar_gasto recibiera la variable convertida.

2. **Lógica y Visualización del Dashboard (dashboard_view.py)** <br>
Aquí los errores afectaban directamente la interpretación de los datos financieros por parte del usuario.
- Cálculo Invertido de Ganancias: Un error de aritmética básica con gran impacto. La fórmula restaba ventas a los gastos ($Gastos - Ventas$), lo que mostraba números negativos a pesar de tener éxito comercial. Se corrigió invirtiendo la operación para obtener la utilidad real: $Ventas - Gastos$.
- Escalado de Gráficos (Barras): El sistema asignaba píxeles de forma literal a la cantidad de ventas. Esto hacía que cantidades pequeñas fueran invisibles (1px de ancho) y cantidades grandes se desbordaran. Se implementó una regla de tres simple para normalizar los valores dentro de un rango de 4 a 220 píxeles, garantizando que el gráfico sea siempre legible y estético.

3. **Manejo de Objetos y Formato (historial_view.py)** <br>
El último bloque trata sobre la experiencia de usuario (UX) y la integridad de la Programación Orientada a Objetos (POO).

- Formato de Diccionarios: Al mostrar el historial, el programa imprimía el objeto crudo de Python (ej. {'Taco': 2}). Para el usuario final, esto es confuso. La solución empleó una técnica de formateo mediante un list comprehension y el método .join(), transformando los datos en un texto legible y elegante: 2x Taco, 1x Agua.

- Referencia a Atributos de Clase: El Bug 6 era un error de "scope" o alcance. Se intentaba manipular una lista local en lugar del atributo de la instancia (self.lista). Esto causaba que la interfaz no se refrescara correctamente. Al redirigir el flujo hacia el método _recargar() y apuntar a los controles de la instancia, se recuperó la reactividad de la aplicación.
</div>
