---
title: Execution Options
format: html
---

## Output Options

There are a wide variety of options available for customizing output from executed code. All of these options can be specified either globally (in the document front-matter) or per code-block. For example, here's a modification of the Python example to specify that we don't want to "echo" the code into the output document:

``` {.yaml}
---
title: "My Document"
execute:
  echo: false
jupyter: python3
---
```

Note that we can override this option on a per code-block basis. For example:

```` {.python}
```{python}
#| echo: true

import matplotlib.pyplot as plt
plt.plot([1,2,3,4])
plt.show()
```
````

Code block options are included in a special comment at the top of the block (lines at the top prefaced with `#|` are considered options).

Options available for customizing output include:

| Option    | Description                                                                                                                                  |
|-----------|----------------------------------------------------------------------------------------------------------------------------------------------|
| `eval`    | Evaluate the code chunk (if `false`, just echos the code into the output).                                                                   |
| `echo`    | Include the source code in output                                                                                                            |
| `output`  | Include the results of executing the code in the output                                                                                      |
| `warning` | Include warnings in the output.                                                                                                              |
| `error`   | Include errors in the output (note that this implies that errors executing code will not halt processing of the document).                   |
| `include` | Catch all for preventing any output (code or results) from being included (e.g. `include: false` suppresses all output from the code block). |

Here's a Knitr example with some of these additional options included:

```` {.markdown}
---
title: "Knitr Document"
execute:
  echo: false
---

```{r}
#| warning: false

library(ggplot2)
ggplot(airquality, aes(Temp, Ozone)) + 
        geom_point() + 
        geom_smooth(method = "loess", se = FALSE)
```

```{r}
summary(airquality)
```
````

::: {.callout-tip}
When using the Knitr engine, you can also use any of the available native options (e.g. `collapse`, `tidy`, `comment`, etc.). See the [Knitr options documentation](https://yihui.org/knitr/options/) for additional details. You can include these native options in option comment blocks as shown above, or on the same line as the `{r}` as shown in the Knitr documentation.
:::

## Figure Options

There are a number of ways to control the default width and height of figures generated from code. First, it's important to know that Quarto sets a default width and height for figures appropriate to the target output format. Here are the defaults (expressed in inches):

| Format                  | Default   |
|-------------------------|-----------|
| Default                 | 7 x 5     |
| HTML Slides             | 9.5 x 6.5 |
| HTML Slides (reveal.js) | 9 x 5     |
| PDF                     | 6.5 x 4.5 |
| PDF Slides (Beamer)     | 10 x 7    |
| PowerPoint              | 7.5 x 5.5 |
| MS Word, ODT, RTF       | 5 x 4     |
| EPUB                    | 5 x 4     |
| Hugo                    | 8 x 5     |

These defaults were chosen to provide attractive well proportioned figures, but feel free to experiment to see whether you prefer another default size. You can change the default sizes using the `fig-width` and `fig-height` options. For example:

``` {.yaml}
---
title: "My Document"
format: 
  html:
    fig-width: 8
    fig-height: 6
  pdf:
    fig-width: 7
    fig-height: 5
---
```

How do these sizes make their way into the engine-level defaults for generating figures? This differs by engine:

-   For the Knitr engine, these values become the default values for the `fig.width` and `fig.height` chunk options. You can override these default values via chunk level options.

-   For the Jupyter engine, these values are used to set the [Matplotlib](https://matplotlib.org/stable/tutorials/introductory/customizing.html) `figure.figsize` rcParam (you can of course manually override these defaults for any given plot).

    If you are using another graphics library with Jupyter and want to utilize these values, you can read them from `QUARTO_FIG_WIDTH` and `QUARTO_FIG_HEIGHT` environment variables.

### Caption and Alt Text

You can specify the caption and alt text for figures generated from code using the `fig.cap` and `fig.alt` options. For example, here we add these options to a Python code cell that creates a plot:

```` {.python}
```{python}
#| fig.cap: "Polar axis plot"
#| fig.alt: "A line plot on a polar axis"

import numpy as np
import matplotlib.pyplot as plt

r = np.arange(0, 2, 0.01)
theta = 2 * np.pi * r
fig, ax = plt.subplots(subplot_kw={'projection': 'polar'})
ax.plot(theta, r)
ax.set_rticks([0.5, 1, 1.5, 2])
ax.grid(True)
plt.show()
```
````

## Intermediates

On the way from markdown input to final output, there are some intermediate files that are created and automatically deleted at the end of rendering. You can use the following options to keep these intermediate files:

| Option       | Description                                                                                    |
|--------------|------------------------------------------------------------------------------------------------|
| `keep-md`    | Keep the markdown file generated by executing code.                                            |
| `keep-ipynb` | Keep the notebook file generated from executing code (applicable only to markdown input files) |

For example, here we specify that we want to keep both the markdown intermediate file after rendering:

``` {.markdown}
---
title: "My Document"
execute:
  keep-md: true
jupyter: python3
---
```

## Engine Options

### Caching

Quarto integrates with the [Jupyter Cache](https://jupyter-cache.readthedocs.io/en/latest/) and [Knitr Cache](https://bookdown.org/yihui/rmarkdown-cookbook/cache.html) to to cache time consuming code chunks. Note that to use Jupyter Cache you'll want to install the `jupyter-cache` package:

``` {.bash}
$ pip install jupyter-cache
```

To enable caching for a document just add the `cache` option:

``` {.yaml}
execute: 
  cache: true
```

You can also use \`quarto\` command line options to control caching behavior without changing the document's code. Use options to force the use of caching on all chunks, disable the use of caching on all chunks (even if it's specified in options), or to force a refresh of the cache even if it has not been invalidated:

``` {.bash}
$ quarto render example.qmd --cache 
$ quarto render example.qmd --no-cache 
$ quarto render example.qmd --cache-refresh 
```

Note that for Jupyter, the cache for a document is invalidated if any of the code blocks change. For Knitr, invalidation occurs on a per-cell basis.

### Disabling Execution

In some cases, you may want to prevent execution entirely. This is especially useful if you author using a standard notebook editor (e.g. JupyterLab) and plan on executing chunks only within the notebook UI. Specify `execute: false` to skip execution when rendering (you'll naturally still get the output that was generated within the notebook editor):

``` {.yaml}
execute: false
```

If you are temporarily disabling execution and don't want to overwrite other `execute` options, you can alternatively just add `enabled: false` to the `execute` options:

``` {.yaml}
execute:
  enabled: false
  echo: true
  warning: false
```

### Jupyter Kernel

The Jupyter kernel is determined using the `jupyter` metdata option. For example, to use the [Xeus Python](https://github.com/jupyter-xeus/xeus-python) kernel, do this:

``` {.markdown}
---
title: "My Document"
jupyter: xpython
---
```

Note that you can also provide a full `kerenlspec`, for example:

``` {.markdown}
---
title: "My Document"
jupyter: 
  kernelspec:
    name: "xpython"
    language: "python"
    display_name: "Python 3.7 (XPython)"
---
```

If no Jupyter kernel is specified, then the kernel is determined by finding an available kernel that supports the language of the first executable code block found within the file (e.g. ```` ```{python} ````).

### Jupyter Daemon

To mitigate the \~ 2 second start-up time for the Jupyter Python kernel (and potentially much longer start-up times for other kernels), Quarto keeps a daemon with a running Jupyter kernel for each document rendered interactively. This enables subsequent renders to proceed immediately without having to wait for kernel start-up.

No daemon is created when documents are rendered without an active tty or when they are part of a batch rendering (e.g. in a [Quarto Project](../getting-started/quarto-projects.md)).

You can customize this behavior using the `daemon` execution option. Set it to `false` to prevent the use of a daemon, or set it to a value (in seconds) to determine the period after which the daemon will timeout (the default is 300 seconds). For example:

``` {.markdown}
execute:
  daemon: false
```

``` {.markdown}
execute:
  daemon: 60
```

You can also force an existing daemon to restart using the `--execute-daemon-restart` command line flag to `quarto render`:

``` {.bash}
$ quarto render document.ipynb --execute-daemon-restart 
```