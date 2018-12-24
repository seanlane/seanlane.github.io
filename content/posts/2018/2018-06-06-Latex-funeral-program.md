---
date: 2018-06-06
title: Example for LaTeX Funeral or Memorial Program
commentIssueId: 10
aliases: /blog/2018/Latex-funeral-program
tags:
  - LaTeX
---

Another quick post, but something that I hope might be useful to others. Within the past couple weeks, my grandmother of 83 years passed away and my family held a memorial service in her honor. I was asked if I could help out in creating a program booklet or pamphlet that could be given out to the attendees, something that would describe the service itself as well as share a piece of my grandmother's life with them as we gathered to remember her. Grandma Arlene was a classy lady and I wanted to help her leave a lasting impression on all of those who could make it out to honor her life. As a graduate student in Computer Science, I felt like using LaTeX would be a great way to do so, though my Internet searches fell somewhat short of what I was looking for. We wanted a simple layout consisting of four "pages", two of which would be printed on a single side of standard sized US letter paper, and then folded into a four page pamphlet after printing. This could also serve well for someone looking for a LaTeX template for religious or other services where a 4 page booklet is desired.

The files for this project can be found on GitHub here: [Example for LaTeX Funeral or Memorial Program](https://github.com/seanlane/LaTeX-Funeral-Program)

The project makes use of standard LaTeX components as well as the `pgfornament` package to add some style to the program. The general process is as follows:

1. Edit the `main.tex` file as needed
  * Note that you will need to adjust paper size as needed to fit onto a half of the paper size you intend to use. As it currently stands, it's set to be printed on a 8.5 inches by 5.5 inches section of 8.5 inches by 11 inches sized US letter paper
  * Also modify your images as needed. My grandmother was a prodigious quilter, and we wanted to have one of her quilts serve as the background for the third page where the actual service is described.
2. Use the `Makefile` default command to make the main file, which will produce 4 pages on the size of paper specified in `main.tex` and then take those four pages and place them on both sides of US letter paper as described by `booklet.tex`.
3. I found that the pages of the output `booklet.pdf` were still oriented in a portrait orientation, so I used Apple Preview to rotate the pages. There may likely be a programmatic way to address that issue, but I never bothered to resolve it. 

Below are screenshots of the output.


{{< figure src="/images/2018/06/latex_funeral_program_pg1.png" title="First page" caption="First page of the final PDF output" >}}


{{< figure src="/images/2018/06/latex_funeral_program_pg2.png" title="Second page" caption="Second page of the final PDF output" >}}