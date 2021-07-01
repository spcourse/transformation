# Data Transformation

Generally speaking, there are 2 types of data transformations:

1. **Filter, merge or restructure data.** These operations do not alter the content
of the data, but modify the general structure in some way. The simplest example
would be selecting specific rows from a data frame, but you might do something
more complex like combining data from several sources.

2. **Compute or extract new features:** For example, rescaling a column in a data
frame to change the temperature units from *Fahrenheit* to *Celsius*. This
changes the values inside the data frame cells, so it *does* modify the data
content. Another common transformation would be to compute an average or
maximum of a feature, extracting new content from the data.

For this module there are 2 assignments, both of which will require several
transformations of each type. The goal is to practice with enough different
types of transformations, such that by the end you will be well equipped to select
and build useful transformations for your own data processing projects.

The first assignment this week will be on transformations that might, at least partially,
seem familiar and is aimed to get you started with the module. The second
assignment will contain some more complex transformations, and is separated in two parts. As an additional
exercise, it will be useful to try and classify what exact transformation you are applying for each part of the module.

It is required to do both assignments as *Jupyter Notebooks*, as it can be
useful to inspect or plot the data while you are transforming it. Jupyter Notebooks also record the outcomes of earlier cells, which can save you a lot of time, as you won't have to load the data again every time you change something in your code!
