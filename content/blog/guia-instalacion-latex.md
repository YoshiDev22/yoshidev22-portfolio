---
title: "LaTeX en Windows: Instala y escribe en 5 minutos"
date: 2026-06-06
draft: false
tags: ["latex", "tutorial", "windows", "herramientas"]
categories: ["Apuntes"]
description: "Guía mínima para instalar LaTeX en Windows con MiKTeX y TeXStudio y escribir tu primer documento."
---

## ¿Qué es y cuándo usarlo?

LaTeX es un sistema de composición de documentos orientado a calidad tipográfica.
Úsalo cuando necesites escribir fórmulas matemáticas, tesis, artículos científicos
o cualquier documento donde Word se quede corto o tu asesor lo pida jeje.

---

## Instalación

### 1. MiKTeX (el motor)

1. Descarga el instalador desde [miktex.org](https://miktex.org/download)
2. Durante la instalación activa: **Install missing packages on-the-fly → Yes**
3. Al terminar, abre **MiKTeX Console** y ejecuta **Update** para sincronizar paquetes

### 2. TeXStudio (el editor)

1. Descarga desde [texstudio.org](https://www.texstudio.org/)
2. Instala con las opciones por defecto
3. Abre TeXStudio → **Options → Configure TeXStudio → Build**
4. Verifica que **Default Compiler** esté en `pdfLaTeX`

---

## Estructura mínima de un documento

```latex
% ─── PREÁMBULO ────────────────────────────────────────
\documentclass[a4paper, 12pt]{article}  % tipo y formato del doc

\usepackage[spanish]{babel}             % idioma español
\usepackage[utf8]{inputenc}             % tildes y ñ
\usepackage{amsmath}                    % matemáticas extendidas

\title{Mi documento}
\author{Tu Nombre}
\date{\today}

% ─── CONTENIDO ────────────────────────────────────────
\begin{document}

\maketitle          % genera título, autor y fecha

\section{Introducción}
Escribe aquí tu contenido.

\end{document}
```

> Para compilar en TeXStudio: **F5** o el botón ▶▶

---

## Ejemplo funcional

```latex
\documentclass[a4paper, 12pt]{article}

\usepackage[spanish]{babel}
\usepackage[utf8]{inputenc}
\usepackage{amsmath}

\title{Apunte de Prueba}
\author{R\&D Blog}
\date{\today}

\begin{document}

\maketitle

\section{Texto}
LaTeX permite \textbf{negritas}, \textit{cursivas} y \texttt{código en línea}.

\section{Matemáticas}
Ecuación en línea: $E = mc^2$

Ecuación en bloque:
$$\int_{0}^{\infty} e^{-x^2}\, dx = \frac{\sqrt{\pi}}{2}$$

\section{Lista}
\begin{itemize}
    \item Elemento uno
    \item Elemento dos
\end{itemize}

\end{document}
```

---

## Referencia de comandos

### Formato de texto

| Comando | Resultado |
|---|---|
| `\textbf{texto}` | **negrita** |
| `\textit{texto}` | *cursiva* |
| `\underline{texto}` | <u>subrayado</u> |
| `\texttt{texto}` | `monoespaciado` |

### Matemáticas

| Uso | Sintaxis |
|---|---|
| En línea | `$f(x) = x^2$` |
| Bloque centrado | `$$f(x) = x^2$$` |
| Ecuación numerada | `\begin{equation} ... \end{equation}` |
| Fracción | `\frac{a}{b}` |
| Integral | `\int_{a}^{b}` |
| Sumatoria | `\sum_{i=0}^{n}` |
| Raíz | `\sqrt{x}` |

### Listas

| Tipo | Entorno |
|---|---|
| Sin orden | `\begin{itemize} ... \end{itemize}` |
| Numerada | `\begin{enumerate} ... \end{enumerate}` |
| Cada ítem | `\item texto` |

### Secciones

| Comando | Nivel |
|---|---|
| `\section{}` | Principal |
| `\subsection{}` | Secundario |
| `\subsubsection{}` | Terciario |