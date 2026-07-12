# JavaScript en la web — Uso y abuso

Relacionado: [[HTTP-Peticiones-y-Respuestas]] · [[Arquitectura-Web]] · [[Herramientas-Burp-Suite]] · [[Ofensiva-Pentesting]]

## Qué es
JS es un lenguaje **interpretado** (se ejecuta directo en el navegador, sin compilación previa). Es el "cerebro" del front end: permite tomar decisiones y hacer la página dinámica e interactiva. Se ejecuta mayormente del **lado del cliente**, por eso es fácil de inspeccionar y manipular desde el navegador → esto tiene implicaciones de seguridad.

## Conceptos básicos
```javascript
// Salida por consola
console.log("Hello, World!");

// Variable y tipo de dato
let age = 25;              // Number

// Control de flujo
if (age >= 18) {
    console.log("You are an adult.");
} else {
    console.log("You are a minor.");
}

// Función
function greet(name) {
    console.log("Hello, " + name + "!");
}
greet("Bob");
```

Ejecutar en la **consola de Chrome**: `Ctrl + Shift + I` → pestaña **Console** → pegar código → Enter.
```javascript
let x = 5;
let y = 10;
let result = x + y;
console.log("The result is: " + result);   // The result is: 15
```

## Integrar JS en HTML

### Interno (dentro del HTML, en `<script>`)
Bueno para aprender; el script puede ir en `<head>` (carga antes) o `<body>` (interactúa con elementos ya cargados).
```html
<body>
    <h1>Addition of Two Numbers</h1>
    <p id="result"></p>
    <script>
        let x = 5;
        let y = 10;
        let result = x + y;
        document.getElementById("result").innerHTML = "The result is: " + result;
    </script>
</body>
```

### Externo (archivo `.js` aparte)
Mantiene el HTML limpio y permite reutilizar el código en varias páginas. Se enlaza con el atributo `src`.
```html
<!-- script.js contiene la lógica -->
<script src="script.js"></script>
```

### Verificar interno vs externo (para pentesting)
Clic derecho → **View Page Source**:
- `<script>` **sin** `src` → JS interno.
- `<script src="...">` → JS externo (archivo separado).

## Abusar de funciones de diálogo
JS ofrece `alert`, `prompt` y `confirm`. Si no se implementan con cuidado, se pueden explotar (ej. **XSS**).

```javascript
alert("Hello THM");                    // caja con OK (mensaje/aviso)

name = prompt("What is your name?");   // pide input; devuelve el valor o null
alert("Hello " + name);

confirm("Do you want to proceed?");    // devuelve true (OK) o false (Cancel)
```

### Cómo lo explota un atacante
Un archivo HTML "inofensivo" enviado por correo puede traer JS molesto o malicioso:
```html
<script>
    for (let i = 0; i < 500; i++) {
        alert("Hacked");     // fuerza a cerrar 500 alertas
    }
</script>
```
Lección: **ejecuta solo archivos JS de fuentes confiables**.

## Bypass de control de flujo
Estructuras: `if-else`, `switch`, y bucles `for`, `while`, `do...while`.

```html
<script>
    age = prompt("What is your age")
    if (age >= 18) {
        document.getElementById("message").innerHTML = "You are an adult.";
    } else {
        document.getElementById("message").innerHTML = "You are a minor.";
    }
</script>
```

### Bypass de logins client-side
Si la autenticación se hace **solo en JS** (usuario/contraseña comparados en el navegador), es trivial saltarla: basta leer el **código fuente** para ver las credenciales en claro (`admin` / `ComplexPassword`). Por eso **nunca** se debe confiar la autenticación al cliente.

## Archivos minificados y ofuscados

### Minificación
Reduce el tamaño del código (quita espacios, saltos de línea) → carga más rápido. Los archivos suelen llamarse `*.min.js`.

### Ofuscación
Convierte el código en algo **no legible por humanos** pero **funcional** para el navegador. Dificulta el reversing (no lo impide).
```javascript
// Antes:  alert("Welcome to THM");
// Después (ofuscado): (function(_0x114713,_0x2246f2){var _0x51a830=_0x33bf...})(...)
```
- **Ofuscar** online: obfuscator.io y similares.
- **Deofuscar** online: herramientas de "deobfuscate javascript" recuperan un equivalente legible.

> Truco CTF: valores ofuscados como `age=0x1*0x247e+0x35*-0x2e+-0x1ae3;` son solo aritmética en hexadecimal → evalúa la expresión para obtener el valor real (aquí `21`).

## Buenas prácticas (defensa)
1. **No confiar solo en validación client-side**: valida también en el **servidor** (el usuario puede desactivar/manipular JS). Ver [[HTTP-Peticiones-y-Respuestas]].
2. **No incluir librerías no confiables**: atacantes suben librerías con nombres parecidos a los legítimos (typosquatting de paquetes).
3. **No hardcodear secretos**: API keys, tokens o credenciales en el JS son visibles en el código fuente.
   ```javascript
   // MAL:
   const privateAPIKey = 'pk_TryHackMe-1337';
   ```
4. **Minificar y ofuscar** el JS en producción: reduce tamaño y eleva el esfuerzo del atacante (no es protección absoluta).
