# CSS

Configuraciones recomendadas para CSS

Para aditar apariencia de Scrollbar, implementar el siguiente código en el archivo CSS principal

```css
body::-webkit-scrollbar {
  width: 10px;
}

body::-webkit-scrollbar-thumb {
  background: rgba(255,255,255,0.3);
  border-radius: 10px;
}
```