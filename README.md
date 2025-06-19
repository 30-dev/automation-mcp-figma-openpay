---

getfigma: 'YOUR\_FIGMA\_FILE\_KEY'
applyTo: '\*\*'
---------------

# Mailing Instructions for MCP

## Preliminary Interaction Phases

Before processing Figma content, the agent should follow this question flow:

1. **Template Selection**

   * "Which mailing template will you use as a base?" Provide the path or filename (it must include a table with the placeholder `content-mailing`).
2. **Confirm Working Path**

   * "Is the base path `2025/June/Ecommerce/HM-Link-de-Pago` correct?" Validate or correct.
3. **Receive URLs**

   * "Please provide the Figma URLs (exactly two)."
4. **Determine Target Folder**

   * For each URL extract the suffix after the last underscore (e.g., `OM`, `BBVA`, or any other).
   * If no clear suffix is found, ask: "Which folder should we place this in?"
5. **List and Confirm Frames**

   * Open the URL in Figma. List detected frames and ask if they are the ones to process.
6. **Create Folder Structure**

   * For each confirmed frame, create a subfolder named exactly as the frame under the target folder (`SUFFIX`), and inside it create `assets/` and an empty `index.html`.
7. **Generate HTML Snippets**

   * "Shall we generate the HTML rows to insert **only** inside the `content-mailing` table of the template?"
8. **Width and Consistency Check**

   * Verify that all widths are fixed (min/max 600 px), no relative values (%, em, vw), and inline styles match specifications.
9. **Insert into Template**

   * Insert the HTML blocks (inline CSS, fixed widths min/max 600 px) into the `content-mailing` placeholder without altering other parts of the template.
10. **Optimize Images**

* Copy `optimizar.bat` into each `assets/` folder and, after confirmation, run it to optimize images.

11. **Final Validation**

* Display the folder and file tree and ask: "Is everything ready to move to the MD stage?"

---

## 0. Automatic Width Adjustment with Side Padding

* MCP will detect any side padding in `mail_tabla_principal_contenedor` (or its child frames) and adjust the **inner width** so that:

  > absolute width (600 px) = inner width + left padding + right padding

* **Example:** with `padding: 0 60px`, the available inner width becomes 480 px.

## 1. Configure the Root Frame

1. Create a **Frame (F)** and rename it to:

   ```
   mail_tabla_principal_contenedor
   ```
2. Enable **Auto Layout** (Shift + A):

   * Direction: Vertical
   * Spacing between items: 0
   * Padding: 20 px (top/bottom), 0 px (left/right)
3. In **Constraints**:

   * Width: Fixed → Min: 600 px; Max: 600 px
   * Height: Hug contents

> MCP will read this Frame and generate:
>
> ```html
> <table width="600" style="min-width:600px; max-width:600px;">
> ```

## 2. Create and Name Main Sections

| Section             | Layer Name                   | Type                   |
| ------------------- | ---------------------------- | ---------------------- |
| Brand / Logo        | `bg_brand`                   | Frame (not rendered)   |
| Header              | `bg_fila_header`             | Frame / Text           |
| Subject / Preheader | `mail_subject_preheader`     | Frame (not rendered)   |
| Separator           | `fila_separador`             | Rectangle, 1 px height |
| Subtitle            | `fila_subtitulo_descripcion` | Text, 18 px semibold   |
| Description         | `fila_descripcion`           | Text, 14 px regular    |
| Info + Button Block | `bg_fila_informacion`        | Frame + Auto Layout    |
| CTA Button          | `fila_boton`                 | Component / Button     |
| Contact (footer)    | `fila_contacto`              | Text, 12 px small      |

> **Note:** Order these layers **inside** `mail_tabla_principal_contenedor`.

## 3. Component Design Details

### 3.1 Frames `bg_*`

**Global rule:** Any Figma frame whose name starts with `bg_` must **not** render internal content. It serves only as a visual container with a background image.

#### 🛠️ MCP Output:

```html
<tr>
  <td id="bg_<name>" width="600"
      style="min-width:600px; max-width:600px;
             background-image:url('assets/bg_<name>.jpg');
             background-size:cover;
             background-position:center;
             height:<FRAME_HEIGHT>px;">
  </td>
</tr>
```

---

#### 🖼️ Export Rules for `bg_` Assets

| Parameter         | Value                   |
| ----------------- | ----------------------- |
| **Format**        | JPEG                    |
| **Scale**         | 2x                      |
| **Filename**      | `bg_<name>.jpg`         |
| **Expected Path** | `/assets/bg_<name>.jpg` |
| **Usage in HTML** | Inline background-image |

> ⚠️ The `height` value must exactly match the original frame height in Figma.

---

#### ✅ Example:

Frame: `bg_hero`
Height: `350px`

**MCP Output:**

```html
<tr>
  <td id="bg_hero" width="600"
      style="min-width:600px; max-width:600px;
             background-image:url('assets/bg_hero.jpg');
             background-size:cover;
             background-position:center;
             height:350px;">
  </td>
</tr>
```

---

#### 🚫 Common Errors

* ❌ Frame name does not start with `bg_`.
* ❌ Asset not exported as JPEG.
* ❌ Missing `height` in inline style.
* ❌ Expecting internal content to render (MCP ignores it).

---

### 3.2 Frame `mail_subject_preheader`

* This Frame does **not** render in the HTML.
* It defines **Subject** and **Preheader**, but does **not** generate `<tr>` or `<td>`.

### 3.3 Rows and Columns

#### 3.3.1 Special Handling for `fila`, `columna` and `tabla` Frames

* **Any frame whose name contains “fila”** (case-insensitive) is treated as a **row** and becomes a `<tr>` wrapper.
* **Any frame whose name contains “columna”** is treated as a **cell container** and produces a `<td>` wrapper around its contents within a `<tr>`.
* **Any frame whose name contains “tabla”** is treated as a **nested table**, generating a `<table>` element (fixed width 600 px) inside the parent `<td>`.

```html
<!-- Example: Frame named "tabla_ofertas" -->
<tr>
  <td>
    <table width="600" style="min-width:600px;max-width:600px;">
      <!-- inner rows/columns -->
    </table>
  </td>
</tr>
```

### 3.3 Rows and Columns

* Each design **row** becomes a `<tr>`.
* Each **cell** becomes a `<td>`.
* Ensure all content (text, buttons) is within Frames or Components to produce proper `<td>` elements.

### 3.4 Frame `bg_fila_header`

* Frame (+ Auto Layout if icon + text).
* Contains an inner Text layer also named `bg_fila_header`.
* Font-size: 20 px; Weight: Bold.
* Constraints: Fill container / Hug contents.

MCP Output:

```html
<tr><td id="bg_fila_header">{{ header_text }}</td></tr>
```

### 3.5 Content Rows (repeat per block)

1. **Separator** `fila_separador`: Rectangle, 1 px high, fill `#DDDDDD`.
2. **Subtitle** `fila_subtitulo_descripcion`: Text, 18 px semibold.
3. **Description** `fila_descripcion`: Text, 14 px regular; line-height: 1.5.

MCP Output for each trio:

```html
<tr><td id="fila_separador"></td></tr>
<tr><td id="fila_subtitulo_descripcion">...</td></tr>
<tr><td id="fila_descripcion">...</td></tr>
```

### 3.6 Frame `bg_fila_informacion`

* Child Frame with Vertical Auto Layout; Padding: 20 px.
* Contains in order:

  1. `fila_separador`
  2. `fila_subtitulo_descripcion`
  3. `fila_descripcion`
  4. `fila_boton` (Button component)
  5. `fila_contacto` (footer text)

MCP Output:

```html
<tr><td id="bg_fila_informacion">
  <!-- inner blocks -->
</td></tr>
```

## 4. Publish and Activate Library

1. Go to **Assets → Library** in Figma.
2. Publish this file as **My-Mailing-Library**.
3. Enable the library in other files and drag in components.

## 5. Sample Generated HTML

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <title>Email Template</title>
</head>
<body style="margin:0;padding:0;background:#ffffff;">
  <table width="600" cellpadding="0" cellspacing="0" border="0" align="center" style="min-width:600px;max-width:600px;">
    <!-- Brand / Logo -->
    <tr>
      <td id="bg_brand" style="padding:20px 0;text-align:center;background-image:url('assets/bg_brand.jpg');background-size:cover;background-position:center;">
      </td>
    </tr>
    <!-- Header -->
    <tr>
      <td id="bg_fila_header" style="padding:20px;text-align:center;font-size:20px;font-weight:bold;">
        {{ header_text }}
      </td>
    </tr>
    <!-- Dynamic Content -->
    {{content_blocks}}
    <!-- Info + CTA -->
    <tr>
      <td id="bg_fila_informacion" style="padding:20px;">
        <table width="100%" cellpadding="0" cellspacing="0" border="0">
          <tr><td id="fila_separador" style="border-top:1px solid #DDDDDD;"></td></tr>
        </table>
        <p id="fila_subtitulo_descripcion" style="margin:15px 0 5px;font-size:18px;font-weight:600;">
          {{ info_subtitle }}
        </p>
        <p id="fila_descripcion" style="margin:0 0 15px;font-size:14px;line-height:1.5;">
          {{ info_description }}
        </p>
        <p id="fila_boton" style="text-align:center;margin:0 0 15px;">
          <a href="{{ cta_url }}" style="display:inline-block;padding:12px 24px;background:#007BFF;color:#ffffff;text-decoration:none;font-weight:600;border-radius:4px;">
            {{ cta_text }}
          </a>
        </p>
        <p id="fila_contacto" style="margin:0;font-size:12px;color:#666666;text-align:center;">
          {{ company_email }} | {{ company_phone }}
        </p>
      </td>
    </tr>
  </table>
</body>
</html>
```
