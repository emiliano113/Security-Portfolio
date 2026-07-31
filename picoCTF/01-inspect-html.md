# picoCTF - Inspect HTML

- **Categoría:** Web Exploitation
- **Dificultad:** Easy
- **Plataforma:** picoCTF

---

## 📝 Descripción del Desafío
El reto consiste en analizar una página web simple y encontrar la bandera (`flag`) oculta en el código fuente de la aplicación.

---

## 🔍 Análisis y Explotación

1. Se accedió a la URL provista por el laboratorio.
2. Utilizando la herramienta de desarrollo del navegador (**DevTools**) o la opción `Ver código fuente de la página` (`Ctrl + U`), se inspeccionó el HTML.
3. Se buscó entre las etiquetas HTML y comentarios del desarrollador.
4. Se identificó la bandera en clara visión dentro de los comentarios de la estructura HTML:

```html
<!-- picoCTF{...} -->
