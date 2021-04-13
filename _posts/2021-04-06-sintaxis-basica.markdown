---
title: "Markdown: Sintáxis básica"
author: pancho
categories:
  - Jekyll
tags:
  - Jekyll
  - markdown
  - md
---
# Sintáxis básica en Jekyll - Markdown

## Texto

| Objeto            |           ¿Cómo?               |
|:------------------|:------------------------------:|
|   Salto de línea  |   Doble espacio                |
| [Link](noobchamp.github.io) | \[texto](http://example.com) |
| `Destacar` | \`palabra(s)` |
| ---<cite>Cita</cite> | ---\<cite>Cita</cite>
| _enfatizar_ | \_enfatizar_ | 
| **Negrita** | \**Negrita** |



# Header one
\# Header one
## Header two
\## Header two
### Header three
\### Header three
#### Header four
\#### Header four
##### Header five
\##### Header five
###### Header six
\###### Header six

## Blockquotes

Una línea blockquote:

> Divide y vencerás

\> Divide y vencerás

Multi line blockquote with a cite reference:

> La gente malgasta su tiempo y luego quieren más.
> Quieren días con más horas, años con más días, vidas con más años.
> Porque si tuvieran ese tiempo de más podrían corregir cualquier error.
> <footer><strong>Ekko</strong> &mdash; La verdad sobre el tiempo es esta: si no puedes sacarle todo el partido a un momento dado, no te mereces ni un segundo de más.</footer>

```liquid
> La gente malgasta su tiempo y luego quieren más.
> Quieren días con más horas, años con más días, vidas con más años.
> Porque si tuvieran ese tiempo de más podrían corregir cualquier error.
> <footer><strong>Ekko</strong> &mdash; No necesito horas ni días ni años... solo unos segundos</footer>
```

## Tablas

| Header1 | Header2 | Header3 |
|:--------|:-------:|--------:|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|-----------------------------|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|=============================|
| Foot1   | Foot2   | Foot3   |

```liquid
| Header1 | Header2 | Header3 |
|:--------|:-------:|--------:|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|-----------------------------|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|=============================|
| Foot1   | Foot2   | Foot3   |
```

## Listas desordenadas (anidada)

  * List item one 
      * List item one 
          * List item one
          * List item two
          * List item three
          * List item four
      * List item two
      * List item three
      * List item four
  * List item two
  * List item three
  * List item four

## Listas ordenadas (anidada)

  1. List item one 
      1. List item one 
          1. List item one
          2. List item two
          3. List item three
          4. List item four
      2. List item two
      3. List item three
      4. List item four
  2. List item two
  3. List item three
  4. List item four

```liquid
  * List item one 
      * List item one 
          * List item one
          * List item two
          * List item three
          * List item four
      * List item two
      * List item three
      * List item four
  * List item two
  * List item three
  * List item four

  1. List item one 
      1. List item one 
          1. List item one
          2. List item two
          3. List item three
          4. List item four
      2. List item two
      3. List item three
      4. List item four
  2. List item two
  3. List item three
  4. List item four
```

## Direcciones

<address>
  Calle de la piruleta<br /> Casa de la gominola<br /> País Feliz
</address>

## Abreviación 
La abreviación de CSS  "Cascading Style Sheets".
*[CSS]: Cascading Style Sheets

```liquid
La abreviación de CSS  "Cascading Style Sheets".
*[CSS]: Cascading Style Sheets
```

## Cita

"Esto es una cita" ---<cite>Pepe</cite>
```liquid
"Esto es una cita" ---<cite>Pepe</cite>
```
