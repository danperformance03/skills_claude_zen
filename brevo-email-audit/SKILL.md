---
name: brevo-email-audit
description: >
  Audita y corrige un archivo HTML de email para que sea compatible con Brevo y funcione
  correctamente en todos los clientes de correo (Outlook Windows, Outlook mobile iOS,
  Gmail, Apple Mail, iOS Mail, Android). Úsalo siempre que el usuario suba o pegue un
  HTML de email y pida revisarlo, adaptarlo, corregirlo o prepararlo para enviar por Brevo.
  También aplica cuando digan frases como "revisa este correo", "prepara el HTML para Brevo",
  "arregla el email", "verifica que esté bien", "que funcione en Outlook y mobile", o
  cuando compartan capturas de un email con errores visuales. El input es un archivo HTML.
  El output es el HTML corregido listo para copiar en Brevo.
---

# Brevo Email Audit

Recibiste un archivo HTML de email. Tu tarea es auditarlo y entregarlo corregido y listo
para envío por Brevo. Sigue el proceso en orden.

---

## PASO 1 — LEER EL ARCHIVO

Lee el HTML completo antes de hacer cualquier cambio. Si está en `/mnt/user-data/uploads/`,
cópialo primero a `/home/claude/` antes de editar.

---

## PASO 2 — AUDITORÍA

Revisa cada punto de esta lista. Anota los problemas encontrados para reportárselos al usuario.

### 2.1 Preheader
- [ ] Existe un preheader después del `<body>`
- [ ] Usa `<span>`, no `<div>`
- [ ] Tiene `mso-hide:all` en el style inline
- [ ] El color del texto coincide con el fondo del email (invisible)
- [ ] El texto es diferente al subject line

**Corrección si falla:**
```html
<span style="display:none;font-size:1px;color:#FONDO;line-height:1px;max-height:0;overflow:hidden;mso-hide:all;">
  Texto de preheader aquí.
</span>
```

---

### 2.2 Estructura HTML — Layout en tablas
- [ ] Todo el layout usa `<table>` / `<tr>` / `<td>`. Sin `<div>` estructurales
- [ ] Cada tabla tiene: `role="presentation"` `cellpadding="0"` `cellspacing="0"` `border="0"`
- [ ] Ancho del contenedor principal: **600px fijo**
- [ ] Todos los estilos críticos son **inline** en cada elemento

---

### 2.3 Reset CSS en `<style>`
Verificar que existan estas tres líneas:
```css
body, table, td, a { -webkit-text-size-adjust: 100%; -ms-text-size-adjust: 100%; }
table, td { mso-table-lspace: 0pt; mso-table-rspace: 0pt; }
img { -ms-interpolation-mode: bicubic; border: 0; display: block; }
```

---

### 2.4 Imágenes
- [ ] Cada `<img>` tiene atributo `alt=""`
- [ ] Cada `<img>` tiene `width` explícito en píxeles
- [ ] Cada `<img>` tiene `style="display:block"`
- [ ] Las URLs de imágenes son públicas (no Firebase con token, no localhost, no rutas relativas)

> ⚠️ Firebase Storage URLs con `?alt=media&token=...` pueden expirar o requerir auth.
> Recomendar mover a Brevo Biblioteca de imágenes o GitHub Pages si se detectan.

---

### 2.5 Tipografías
- [ ] Las fuentes personalizadas tienen URL pública accesible (Google Fonts, GitHub Pages, Vercel)
- [ ] Cada declaración de fuente tiene **fallback**: `'MiFuente', Arial, sans-serif`
- [ ] Si usa Google Fonts: cargada con `<link>` en el head Y `@import` en el `<style>`

---

### 2.6 Título principal — Regla crítica Outlook mobile iOS ⚠️
Esta es la regla más importante. Outlook mobile iOS **ignora @media queries y condicionales MSO**.
El título se renderiza siempre con el `font-size` base, con todos los `<br>` forzados.

**Verificar:**
- [ ] El `font-size` base del título es seguro para ~360px de ancho sin depender de `@media`
- [ ] No hay dos versiones del título intercambiadas con `display:none` por `@media`
- [ ] No hay `<br>` con clase que se oculta en mobile (`display:none` en `<br>` no funciona)
- [ ] No se usan condicionales `<!--[if mso]>` para controlar el título en mobile

**Tabla de tamaños seguros:**

| Caracteres del título | font-size base seguro |
|---|---|
| Hasta 20 caracteres | 28px |
| 20–30 caracteres | 24px |
| Más de 30 caracteres | 20px |

**Patrón correcto — un solo elemento:**
```html
<h1 class="hero-title" style="font-family:Georgia,serif; font-size:28px; line-height:36px;">
  Título del Email
</h1>
```
```css
@media only screen and (max-width:620px) {
  .hero-title { font-size: 20px !important; line-height: 28px !important; }
}
```

**Patrones que fallan en Outlook mobile iOS — corregir si se encuentran:**

| Patrón encontrado | Por qué falla | Corrección |
|---|---|---|
| Dos `<h1>` swapeados con `display:none` por `@media` | Outlook mobile muestra ambos o ninguno | Un solo `<h1>` con font-size base seguro |
| `<!--[if mso]>` para controlar título en mobile | Solo funciona en Outlook Windows | Usar un solo elemento con tamaño base correcto |
| `<br class="x">` con `display:none` en mobile | El `<br>` siempre se renderiza en Outlook iOS | Eliminar o ajustar font-size |
| `font-size` grande solo reducido vía `@media` | Outlook iOS no procesa `@media` | Reducir el font-size base |

---

### 2.7 Imagen de fondo en hero (si aplica)
- [ ] El `<td>` tiene atributo `background="URL"` además del `style`
- [ ] Existe bloque VML para Outlook Windows
- [ ] Existe configuración de píxeles en `<head>` para Outlook

---

### 2.8 Responsive mobile
- [ ] Existe `@media only screen and (max-width: 600px o 620px)` en el `<style>`
- [ ] El contenedor principal tiene clase para ancho 100% en mobile
- [ ] El `font-size` del `@media` es **menor** que el base (no igual)

---

### 2.9 Footer legal
- [ ] Tiene nombre/identidad del remitente
- [ ] Tiene link de cancelar suscripción (puede ser `{{ unsubscribe }}` para Brevo)

---

## PASO 3 — CORREGIR

Aplica todas las correcciones directamente en el archivo. Usa `str_replace` para cada cambio
puntual. No reescribas el archivo completo a menos que haya más de 5 problemas estructurales.

---

## PASO 4 — REPORTAR Y ENTREGAR

Entrega el HTML corregido con `present_files`. Luego reporta con una tabla concisa:
