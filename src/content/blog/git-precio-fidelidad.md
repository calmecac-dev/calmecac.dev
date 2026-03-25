---
title: "Git y el precio de la fidelidad"
description: "Por qué tu historial nunca es tan limpio como quisieras, qué lo causa, y cómo podría no ser así."
date: 2026-03-25
tags: ["git", "desarrollo", "open source"]
readingTime: 12
---

Git no fue diseñado. Fue destilado.

En abril de 2005, Linus Torvalds — el mismo wey que se aventó el kernel de Linux — se quedó sin herramienta de control de versiones después de un conflicto con la que usaban. Y en lugar de ponerse a buscar alternativas *—como haría una persona sensata—*, simplemente se montó en su macho y escribió Git en aproximadamente diez días. Sus objetivos eran muy concretos: velocidad, que los datos no se corrompieran, soporte para trabajo distribuido, y que aguantara el volumen brutal que genera el desarrollo del kernel de Linux.

No había documento de diseño. No había equipo de producto. No hubo nadie en una sala de juntas dibujando en una pizarra y hablando de "la visión". Había un problema concreto y un ingeniero con mucho café y pocas ganas de usar lo que ya existía.

Lo fascinante es que la estructura interna que Linus diseñó ese fin de semana sigue siendo esencialmente la misma hoy, veinte años después. Eso dice mucho de lo bien pensada que estaba. El problema — y eso explica muchas cosas — es que fue pensada para el workflow del kernel de Linux, y el mundo la adoptó para absolutamente todo lo demás sin cuestionarla ni siquiera tantito.
