





## Introducción

Los principios de la ingeniería de software, del libro de Robert C. Martin [Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882), adaptado para Python. Esta no es una guía de estilo, en cambio, es una guía para crear software que sea reutilizable, comprensible y que se pueda mejorar con el tiempo.

No hay que seguir tan estrictamente todos los principios en este libro, y vale la pena mencionar que hacia muchos de ellos habrá controversia en cuanto al consentimiento. Estas son reflexiones hechas después de muchos años de experiencia colectiva de los autores de _Clean Code_.

Cuando la arquitectura de software llegue a ser tan vieja como la arquitectura en sí misma, quizás tengamos reglas más estrictas para seguir. Hasta entonces, dejemos que estas guías sirvan como ejemplo para medir la calidad del código en Python que tú y tu equipo producen.

Una cosa más: saber esto no te hará un mejor ingeniero inmediatamente, y tampoco trabajar con estas herramientas durante muchos años garantiza que nunca te equivocarás. Cualquier código empieza primero como un borrador, como arcilla mojada moldeándose en su forma final. Por último, arreglamos las imperfecciones cuando lo repasamos con nuestros compañeros de trabajo. No seas tan duro contigo mismo por los borradores iniciales que aún necesitan mejorar. ¡Trabaja más duro para mejorar el programa!


## Variables

__Utiliza nombres significativos y pronunciables para las variables__

Mal hecho:
```python
yyyymmdstr = time.strftime("%Y-%m-%d")
```
Bien hecho:
```python
current_date = time.strftime("%Y-%m-%d")
```

---

__Utiliza un solo vocabulario para las variables del mismo tipo__

Mal hecho:
```python
get_user_info()
get_client_data()
get_customer_record()
```

Bien hecho:
```python
get_user_info()
get_user_data()
get_user_record()
```

---

__Utiliza nombres buscables__

Nosotros leemos mucho más código que jamás escribiremos. Es importante que el código que escribimos sea legible y buscable. Cuando nombramos a las variables de manera buscable y legible, facilitamos la lectura de nuestro código.

Mal hecho:
```python
# Que carajos es 864000
time.sleep(864000)
```

Bien hecho:
```python
# Declarar como variable global
SECONDS_IN_A_DAY = 60 * 60 * 24

time.sleep(SECONDS_IN_A_DAY)
```

---

__Utiliza variables explicativas__

Mal hecho:
```python
address = 'One Infinite Loop, Cupertino 95014'
city_zip_code_regex = r'^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$'
matches = re.match(city_zip_code_regex, address)

save_city_zip_code(matches[1], matches[2])
```

Bien hecho:

Nombrando subpatrones se disminuye la dependencia de expresiones regulares

```python
address = 'One Infinite Loop, Cupertino 95014'
city_zip_code_regex = r'^[^,\\]+[,\\\s]+(?P<city>.+?)\s*(?P<zip_code>\d{5})?$'
matches = re.match(city_zip_code_regex, address)

save_city_zip_code(matches['city'], matches['zip_code'])
```

---

__Evitar el mapeo mental__

Explícito es mejor que implícito

Mal hecho:
```python
seq = ('Austin', 'New York', 'San Francisco')

for item in seq:
    do_stuff()
    do_some_other_stuff()
    # ...
    dispatch(item)
```

Bien hecho:
```python
locations = ('Austin', 'New York', 'San Francisco')

for location in locations:
    do_stuff()
    do_some_other_stuff()
    # ...
    dispatch(location)
```

---

__No incluyas contexto innecesario__
Si tu clase/objeto te dice algo, no lo repitas en el nombre de variable también.

Mal hecho:
```python
class Car:
    car_model: str
    car_color: str
```

Bien hecho:
```python
class Car:
    model: str
    color: str
```

---

__Utiliza argumentos predefinidos en vez de utilizar condicionales__
Los argumentos predefinidos muchas veces son más organizados que utilizar los condicionales.

Mal hecho:
```python
def create_micro_brewery(name):
    name = "Hipster Brew Co." if name is None else name
    slug = hashlib.sha1(name.encode()).hexdigest()
    # etc.
```

Bien hecho:
```python
def create_micro_brewery(name = "Hipster Brew Co."):
    slug = hashlib.sha1(name.encode()).hexdigest()
    # etc.
```

## Funciones

__Argumentos de funciones (2 o menos idealmente)__
Limitar la cantidad de parámetros de tus funciones es muy importante ya que realizar pruebas de tu código sera mas sencillo. Tener mas de dos argumentos te conduce a una explosión combinatoria en la que debes probar toneladas de casos diferentes con cada argumento por separado.

Cero argumentos es el caso ideal. Uno o dos argumentos esta bien, pero tres o más debe evitarse, ya que esto indica que su función esta tratando de hacer demasiado. En otros casos, es mejor refactorizar y crear un objeto para encapsular las funciones extras. Se puede usar un objeto si necesitas muchos argumentos.

Mal hecho:
```python
def create_menu(title, body, button_text, cancellable):
    # ...
```

Bien hecho:
```python
from dataclasses import astuple, dataclass


@dataclass
class MenuConfig:
    """A configuration for the Menu.

    Attributes:
        title: The title of the Menu.
        body: The body of the Menu.
        button_text: The text for the button label.
        cancellable: Can it be cancelled?
    """
    title: str
    body: str
    button_text: str
    cancellable: bool = False

def create_menu(config: MenuConfig):
    title, body, button_text, cancellable = astuple(config)
    # ...


create_menu(
    MenuConfig(
        title="My delicious menu",
        body="A description of the various items on the menu",
        button_text="Order now!"
    )
)
```

---

__Las funciones debe cumplir una sola tarea__

Esta regla es la más importante en la ingeniería de software. Cuando las funciones sirven para hacer más de una cosa, se vuelven mas complejas de leer y probar. Aislar las funciones para que cumplan con solo una acción, volvera su código mucho más limpio.

> Entender tan solo esta regla, lo convertirá en un mejor desarrollador.

Mal hecho:
```python
def email_clients(clients: List[Client]):
    """Filter active clients and send them an email.
    """
    for client in clients:
        if client.active:
            email(client)
```

Bien hecho:
```python
def active_clients(clients: List[Client]) -> Generator[Client]:
    """Only active clients.
    """
    return (client for client in clients if client.active)


def email_client(clients: Iterator[Client]) -> None:
    """Send an email to a given list of clients.
    """
    for client in clients:
        email(client)
```

---

__Los nombres de las funciones deben explicar lo que hacen__

Mal hecho:
```python
class Email:
    def handle(self) -> None:
        # Do something...

message = Email()
# What is this supposed to do again?
message.handle()
```

Bien hecho:
```python
class Email:
    def send(self) -> None:
        """Send this message.
        """

message = Email()
message.send()
```

---

__Las funciones deben tener solo un nivel de abstracción__

Cuando tienes más de un nivel de abstracción tus funciones suelen servir para hacer demasiadas cosas. Crear varias funciones pequeñas traera como resultado mejor reutilización y una comprobación más sencilla.

Mal hecho:
```python
def parse_better_js_alternative(code: str) -> None:
    regexes = [
        # ...
    ]

    statements = regexes.split()
    tokens = []
    for regex in regexes:
        for statement in statements:
            # ...

    ast = []
    for token in tokens:
        # Lex.

    for node in ast:
        # Parse.
```

Bien hecho:
```python
REGEXES = (
   # ...
)


def parse_better_js_alternative(code: str) -> None:
    tokens = tokenize(code)
    syntax_tree = parse(tokens)

    for node in syntax_tree:
        # Parse.


def tokenize(code: str) -> list:
    statements = code.split()
    tokens = []
    for regex in REGEXES:
        for statement in statements:
           # Append the statement to tokens.

    return tokens


def parse(tokens: list) -> list:
    syntax_tree = []
    for token in tokens:
        # Append the parsed token to the syntax tree.

    return syntax_tree
```

---

__Eliminar código duplicado__

Haz todo lo que puedas para evitar duplicar tu código. El código duplicado es malo ya que deberá actulizar en varios lugares si es necesario un cambio.

Mal hecho:
```python
numbers = [99, 18, 22]

square_number = numbers[0] * numbers[0]
print("The square number of {} is {}".format(numbers[0], square_number))

square_number = numbers[1] * numbers[1]
print("The square number of {} is {}".format(numbers[1], square_number))

square_number = numbers[2] * numbers[2]
print("The square number of {} is {}".format(numbers[2], square_number))
```

Bien hecho:
```python
def calculate_square_number(number):
    return number * number

numbers = [99, 18, 22]
for number in numbers:
    print("The square number of {} is {}".format(
        number, calculate_square_number(number)
    ))
```

---

__No utilices 'flags' como parámetros de las funciones__

La mayoría de las veces una *flag* indica que las funciones estan haciendo más de una cosa. Como se menciono antes **las funciones deben hacer una sola cosa**. Si es necesario divida sus funciones en varias mas pequeñas.

Mal hecho:
```python
from pathlib import Path

def create_file(name: str, temp: bool) -> None:
    if temp:
        Path('./temp/' + name).touch()
    else:
        Path(name).touch()
```

Bien hecho:
```python
from pathlib import Path

def create_file(name: str) -> None:
    Path(name).touch()

def create_temp_file(name: str) -> None:
    Path('./temp/' + name).touch()
```

---

__Evita que las funciones produzcan efectos secundarios__

Una función produce un efecto secundario si hace cualquier cosa más que solo tomar un valor y devolverlo. Un efecto secundario podría ser escribir a un archivo, modificar una variable global, o accidentalmente enviar todo tu dinero a un desconocido.

Las funciones necesitan tener efectos secundarios a menudo, puede que sea necesario escribir en un archivo. En ese caso, hay que centrarce en el porque se esta haciendo esto. No tengas varias funciones y clases que esbriben en un archivo en particular. Es mejor crear un servicio que se dedique a eso.

Mal hecho:
```python

name = 'Ryan McDermott'

def split_into_first_and_last_name() -> None:
    global name
    name = name.split()

split_into_first_and_last_name()

print(name)  # ['Ryan', 'McDermott']
```

Bien hecho:
```python
def split_into_first_and_last_name(name: str) -> None:
    return name.split()

name = 'Ryan McDermott'
new_name = split_into_first_and_last_name(name)

print(name)  # 'Ryan McDermott'
print(new_name)  # ['Ryan', 'McDermott']
```

---

__Favorece a la programación funcional en vez de la programación imperativa__

Python no es un idioma funcional tal como es Haskell, pero tiene su propio sabor funcional. Los lenguajes funcionales son más limpios y fáciles de testear. Favorece este estilo de programación todas los vaces que puedas.

Mal hecho:
```python
programmer_output = [
    {
        "name": "Uncle Boddy",
        "lines_code": 500,
    },
    {
        "name": "Suize Q",
        "lines_code": 1500
    },
    {
        "name": "Jimmy Gosling",
        "lines_code": 150
    },
    {
        "name": "Gracie Hopper",
        "lines_code": 1000
    }
]

total_output = 0
for i in range(0, len(programmer_output)):
    total_output += programmer_output[i]["lines_code"]
```

Bien hecho:
```python
programmer_output = [
    {
        "name": "Uncle Boddy",
        "lines_code": 500,
    },
    {
        "name": "Suize Q",
        "lines_code": 1500
    },
    {
        "name": "Jimmy Gosling",
        "lines_code": 150
    },
    {
        "name": "Gracie Hopper",
        "lines_code": 1000
    }
]

total_output = sum(
    map(lambda item: item["lines_code"], programmer_output)
)
```

---

__Evitar condicinales negativos__

Mal hecho:
```python
def is_DOM_node_not_present(node):
    #...

if not is_DOM_node_not_present(node):
    # is present
```

Bien hecho:
```python
def is_DOM_node_present(node):
    #...

if is_DOM_node_present(node):
    # is present
```

---

__Evita los condicionales__

Esto parece un reto imposible. Al escuchar esto por primera vez, la mayoría de la gente dirá: "¿Que se supone que voy hacer sin una declaración `if`?" Bueno, la respuesta es que puedes utilizar para lograr los mismos objetivos en distintos escenarios. La segunda pregunta es "Bueno, me parece bien, pero ¿porque debería hacer esto?" La respuesta yace en un concepto que ya hemos aprendido: **una función debe hacer solo una cosa**. Cuando tienes clases y funciones que contienen declaracioónes `if`, das a entender que tu función hace más de una sola cosa.

Mal hecho:
```python
class Airplane:
    #...

    def get_cruising_altitude(self):

        if self.type == "777":
            return self.get_max_altitude() - self.get_passenger_count()
        
        elif self.type = "Air Force one":
            return self.get_max_altitude()
       
       elif self.type == "Cessna":
            return self.get_max_altitude() - self.get_fuel_expenditure()
```

Bien hecho:
```python
class Airplane:
    #...

class Boeing777(Airplane):

    def get_cruising_altitude(self):
        return self.get_max_altitude() - self.get_passenger_count()

class AirForceOne(Airplane):

    def get_cruising_altitude(self):
        return self.get_max_altitude()

class Cessna(Airplane):

    def get_cruising_altitude(self):
        return self.get_max_altitude() - self.get_fuel_expenditure()
```

---

__Evita la comprobación de tipos (parte 1)__

Python acepta cualquier tipo de argumento en sus funciones. A veces te aprovechas de esta libertad y tiendes a hacer comprobación de `tipos` dentro de tus funciones. Hay muchas maneras de evitar hacer esto. La primera cosa a considerar son las APIs consistentes.

Mal hecho:
```python
def travel_to_texas(vehicle):
    if isinstance(vehicle, Bicycle):
        vehicle.pedal(current_location, new_location)
    
    elif isinstance(vehicle, Car):
        vehicle.drive(current_location, new_location)
```

Bien hecho:
```python
def travel_to_texas(vehicle):
    vehicle.move(current_location, new_location)
```

---

__Evita la comprobación de tipos (parte 2)__

Mal hecho:
```python
def combine(x, y):
    if type(x) is int and type(y) is int \
        or type(x) is str and type(y) is str:
        return x + y

    raise ValueError("Must be of type String or Number")
```

Bien hecho:
```python
def combine(x, y):
    return int(x) + int(y)
```

---

__Elimina el código muerto__

El código muerto es tan elegante como el código duplicado. No hay razón para guardarlo. Si no se usa, ¡eliminalo!.

Mal hecho:
```python
def old_get_request(url):
    # ...

def new_get_request(url):
    # ...

req = new_get_request("decoxviii.github.io")
```

Bien hecho:
```python
def get_request(url):
    # ...

req = get_request("decoxviii.github.io")
```

## Objetos y estructuras de datos

__Utiliza getters y setters__
Utilizar los getters y setters para acceder a datos dentro de los objetos es mejor que simplemente buscar una propiedad. ¿Porque? Bueno, el proposito principal de usar getters y setters es la encapsulación de datos.

Mal hecho:
```python
class Developer:
    def __init__(self, age = 0):
        self._age = age
    
    def get_age(self):
        return self._age
    
    def set_age(self, age):
        self._age = age

john = Developer()
john.set_age(21)
print(john.get_age())
```

Bien hecho:
```python
class Developer:
    def __init__(self, age = 0):
        self._age = age
    
    @property
    def age(self):
        return self._age
    
    @age.setter
    def age(self, age):
        self._age = age

john = Developer()
john.age = 21
print(john.age)
```

## Clases

__Prefiere composición en vez de la herencia__

En la mayoria de los casos es mejor la composición en lugar de la herencia. Hay muchas razones para utilizar estos dos modelos. El punto importante aquí es que tu mente naturalmente quiere utilizar la herencia, así que intenta pensar si composición también podría resolver tu problema.

Puede que te preguntes "¿Cuando debería utilizar la herencia?" Bueno, depende de tu problema en el momento, pero esta sería una lista decente de cuando tiene mayor sentido utilizarla.

1. Tu herencia representa una relación de "es-un" y no de "tener-un" (Humano->Animal vs Usuario->Detalles del Usuario)
2. Puedes reutilizar tu código de las clases bases (Los humanos pueden moverse como todos los animales)
3. Quieres hacer cambios globales a las clases derivadas con cambiar una clase base.

Mal hecho:
```python
class Employee:
    def __init__(self, name=None, email=None):
        self.name = name
        self.email = email
    
    # ...

# Mal porque Employees "tiene" tax data.
# EmployeeTaxData no es un tipo de Employee
class EmployeeTaxData(Employee):
    def __init__(self, ssn, salary):
        super(EmployeeTaxData, self).__init__()
        self.ssn = ssn
        self.salary = salary
    # ...
```

Bien hecho:
```python
class EmployeeTaxData:
    def __init__(self, ssn, salary):
        self.ssn = ssn
        self.salary = salary
    
    # ...

class Employee:
    def __init__(self, name, email):
        self.name = name
        self.email = email
    
    def set_tax_data(self, ssn, salary):
        self.tax_data = EmployeeTaxData(ssn, salary)
    
    # ...
```

## SOLID

__El principio único de resnponsabilidad (SRP)__

Como se menciona en _Clean Code_, "Nunca debe existir más que una sola razón para cambiar una clase". Vale la pena decir que es normal querer llenar una 'clase' con muchas funciones, igual que cuando solo te permiten llevar una maleta en el vuelo. El problema existe en que tu 'clase' no estará cohesiva conceptualmente y existirá muchas razones para cambiarse. Minimizar la cantidad de veces que necesitas cambiar una clase es importante. Es importante ya que con demasiada funcionalidad viene dificultad de modificarlo y entender cómo afecta a otros módulos dependientes en tu programa.

Mal hecho:
```python
class UserSettings:
    
    def __init__(self, user):
        self.user = user
    
    def verify_credentials(self):
        # ...
    
    def change_settings(self, settings):
        if self.verify_credentials():
            # ...
```

Bien hecho:
```python
class UserAuth:
    def __init__(self, user):
        self.user = user
    
    def verify_credentials(self):
        # ...


class UserSettings:
    def __init__(self, user):
        self.user = user
        self.auth = UserAuth(user)
    
    def change_settings(self, settings):
        if self.auth.verify_credentials():
            # ...
```

---

__Principio de abierto/cerrado (OCP)__

Como dijo Bertrand Meyer, "las entidades de software (clases, módulos, funciones, etc.) deben abrirse para extensión, pero cerrarse para modificación. ¿Qué significa eso? Bueno, este principio básicamente nos dice que debes permitir que tus usuarios introduzcan nuevas funcionalidades sin cambiar el código existente.

Mal hecho:
```python
class Employee:
    def __init__(self, name, work_area):
        self.name = name
        self.work_area = work_area
    # ...

    def get_break_time(self):

        if self.work_area == "marketing":
            return "15:00:00"
        
        if self.work_area == "accounting":
            return "13:00:00"
```

Bien hecho:
```python
class Employee:
    def __init__(self, name):
        self.name  = name
    
    # ...

class MarketingArea(Employee):
    def __init__(self, name):
        super().__init__(name)
        self.break_time = "15:00:00"
    
    # ...

class AccountingArea(Employee):
    def __init__(self, name):
        super().__init__(name)
        self.break_time = "13:00:00"

    # ...
```

---

__El principio de sustitución de Liskov (LSP)__

Formalmente se define como _"Si __S__ es un subtipo de __T__, entonces los objetos de tipo __T__ en un programa de computadora pueden ser sustituidos por objetos de tipo __S__"_ (es decir, los objetos de tipo S pueden sustituir objetos de tipo T), sin alterar ninguna de las propiedades deseables de ese programa (la corrección, la tarea que realiza, etc.).

Otra explicación para es concepto es: Cada `clase` que hereda de otra puede usarse como su padre sin necesidad de conocer las diferencias entre ellas. Puede que aun estes confundido, así que miremos al modelo clásico de rectángulo-cuadro. Matemáticamente, un cuadro es un rectángulo, pero si lo modelas como una relación de "es-un" con la herencia, te meterás en problemas rápidamente.

Mal hecho:
```python
class Rectangle:
    def __init__(self):
        self.width = 0
        self.height = 0
    
    def set_color(self, color):
        # ...
    
    def render(self, area):
        # ...
    
    def set_width(self, width):
        self.width = width
    
    def set_height(self, height):
        self.height = height
    
    def get_area(self):
        return self.width * self.height

class Square(Rectangle):

    def __init__(self):
        super().__init__()

    def set_width(self, width):
        self.height = width
        self.width = width
    
    def set_height(self, height):
        self.height = height
        self.width = height


def render_large_rectangles(rectangles):
    for rectangle in rectangles:
        rectangle.set_width(4)
        rectangle.set_height(5)

        area = rectangle.get_area()
        rectangle.render(area)

rectangles = [ Rectangle(), Rectangle(), Square() ]
render_large_rectangles(rectangles)
```

Bien hecho:
```python
class Shape:
    def set_color(self):
        # ...
    
    def render(self, area):
        # ...
    

class Rectangle(Shape):
    def __init__(self, width, height):
        super().__init__()
        self.width = width
        self.height = height
    
    def get_area(self):
        return self.width * self.height
    
class Square(Shape):
    def __init__(self, length):
        super().__init__()
        self.length = length

    
    def get_area(self):
        return self.length ** 2


def render_large_shapes(shapes):
    for shape in shapes:
        area = shape.get_area()
        shape.render(area)
        print(area)
    
shapes = [Rectangle(4, 5), Rectangle(2, 5), Square(5)]
render_large_shapes(shapes)
```

---

__El principio de segregación en cuanto a las interfaces (ISP)__

Este principio establece que los clientes de un programa dado sólo deberían conocer de éste aquellos métodos que realmente usan, y no aquellos que no necesitan usar. El ISP fue concebido para mantener a un sistema desacoplado respecto a los sistemas de los que depende, y así resulte más fácil refactorizarlo, modificarlo y redesplegarlo.

Mal hecho:
```python
class IShape:
    def draw_square(self):
        raise NotImplementedError
    
    def draw_circle(self):
        raise NotImplementedError

class Circle(IShape):
    def draw_square(self):
        # ...

    def draw_circle(self):
        # ...

class Square(IShape):
    def draw_square(self):
        # ...
    
    def draw_circle(self):
        # ...
```

Bien hecho:
```python
class IShape:
    def draw(self):
        raise NotImplementedError

class Circle(IShape):
    def draw(self):
        # ...

class Square(IShape):
    def draw(self):
        # ...
```

---

__Principio de la inversión de las dependencias__

Este principio declara dos cosas esenciales:

1. Los módulos de alto nivel no deben depender de los módulos de bajo nivel. Ambos deben depender de las abstracciones.
2. Las abstracciones no deben depender de los detalles. Los detalles deben depender de las abstracciones.

Llega un punto en el desarrollo de software donde nuestra aplicación estará compuesta en gran parte por módulos.

Cuando esto sucede, tenemos que aclarar las cosas mediante el uso de inyección de dependencia.

Componentes de alto nivel que dependen de componentes de bajo nivel para funcionar.

Mal hecho:
```python
class InventoryRequester:
    def __init__(self):
        self.REQ_METHODS = ["HTTPS"]
    
    def request_item(self, item):
        # ...


class InventoryTracker:
    def __init__(self, items):
        self.items = items
        self.requester = InventoryRequester()
    
    def request_items(self):
        for item in self.items:
            self.requester.request_item(item)


inventory_tracker = InventoryTracker(["apples", "bananas"])
inventory_tracker.request_items()
```

Bien hecho:
```python
class InventoryTracker:
    def __init__(self, items, requester):
        self.items = items
        self.requester = requester

    def request_items(self):
        for item in self.items:
            self.requester.request_item(item)


class InventoryRequesterV1:
    def __init__(self):
        self.REQ_METHODS = ["HTTPS"]

    def request_item(self, item):
        # ...
        pass


class InventoryRequesterV2:
    def __init__(self):
        self.REQ_METHODS = ["WS"]

    def request_item(self, item):
        # ...
        pass


inventory_tracker = InventoryTracker(["apples", "bananas"], InventoryRequesterV2())
inventory_tracker.request_items()
```

## Errores

__No ignores ningun error__

Si no haces nada al identificar algún error no podras arreglarlo o resolverlo. Si metes tu código en un `try/except` significa que un error puede ocurrir allí y podras crear una solución por si acaso.

Mal hecho:
```python
try:
    function_that_might_throw()
except Exception as e:
    raise e
```

Bien hecho:
```python
try:
    function_that_might_throw()
except ZeroDivisionError:
    # ...
except Exception as e:
    raise e
```

## Formatear

Formatear es subjetivo. Como muchas reglas en esta guía, no hay que seguirlas 100%. El punto clave es: NO DISCUTAS sobre formatear. Hay muchas herramientas para facilitar esto. ¡Utiliza una de estas herramientas! Te desperdicias de tu propio tiempo y el tiempo de los demás cuando discuten sobre formatear.

Para las cosas que no tienen relevancia al formateo automático (indentación, tabulos y espacios, comillas dobles y sencillas, etc.), busca aquí para aconsejarte.

__Utiliza capitalización consistente__

La capitalización puede decirte muchas cosas sobre tus variables, funciones, etc. Estas reglas son subjetivas, así que puedes hacer lo que quieras. El punto es, __sin importar lo que escojas hacer se consistente__

Mal hecho:
```python
DAYS_IN_WEEK = 7
days_in_month = 30

songs = ['Back In Black', 'Stairway to Heaven', 'Hey Jude']
Artists = ['ACDC', 'Led Zeppelin', 'The Beatles']

def eraseDatabase():
    # ...

def restore_database():
    # ...

class animal():
    # ...

class Alpaca():
    # ...
```

Bien hecho:
```python
DAYS_IN_WEEK = 7
DAYS_IN_MONTH = 30

songs = ['Back In Black', 'Stairway to Heaven', 'Hey Jude']
artists = ['ACDC', 'Led Zeppelin', 'The Beatles']

def erase_database():
    # ...

def restore_database():
    # ...

class Animal():
    # ...

class Alpaca():
    # ...
```

---

__Los llamadores y llamantes de las funciones deben existir cercas__

Si una función le llama a otra, manten esas funciones verticalmente cerca.

Mal hecho:
```python
class Example:

    def one(self):
        # ...
    
    def three(self):
        # ...
    
    def two(self):
        one()
        # ...
```

Bien hecho:
```python
class Example:

    def one(self):
        # ...
    
    def two(self):
        one()
        # ...
    
    def three(self):
        # ...
    

```

## Comentarios

__Solamente comenta las cosas que tienen una lógica compleja__

Los comentarios existen para pedir perdón, pero no son un requisito. El código bueno más que nada se documenta sí mismo.

Mal hecho:
```python
def get_ord(data: str):
    # iterar cada caracter del string data
    for char in data:
        # recibe un carácter y imprime un entero que representa 
        # el número unicode que representa a ese carácter
        print(ord(char))
```

Bien hecho:
```python
def get_ord(data: str):
    for char in data:
        print(ord(char))
```

---

__No dejes código inutilizado en tus archivos__

El control de versiones existe por una razón. Deja el código viejo en tu historia (git)

Mal hecho:
```python
def make():
    # this_is_no_longer_useful()
    # this_is_not_useful_either()
    this_is_useful_to_me()
```

Bien hecho:
```python
def make():
    this_is_useful_to_me()
```

---

__No escribir comentarios de jornada__

Ojo: ¡Utiliza el control de versiones (git)! No hay necesidad de hacer comentarios de jornada. En cambio, utiliza `git log` para recuperar una historia de lo que has hecho.

Mal hecho:
```python
"""
2019-07-01: Refactorizar
2019-07-02: Remover 'print()' de la función combine
"""

def combine(a, b):
    return a + b
```

Bien hecho:
```python
def combine(a, b):
    return a + b
```

---

__Evitar los marcadores posicionales__

Los marcadores posicionales suelen dificultar las cosas. Deja que las funciones, los nombres de tus variables, la indentación adecuada y el formatear cree una estructura visual para tu código.

Mal hecho:
```python
#####################
# Main function
######################
def main():
    pass
```

Bien hecho:
```python
def main():
    pass
```
