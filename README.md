# Uniplot
[![Build Status](https://github.com/olavolav/uniplot/workflows/Unit%20Tests/badge.svg)](https://github.com/olavolav/uniplot/actions?query=workflow%3A"Unit+Tests")
[![PyPI Version](https://badge.fury.io/py/uniplot.svg)](https://pypi.org/project/uniplot/)
[![PyPI Downloads](https://pepy.tech/badge/uniplot)](https://pepy.tech/project/uniplot)

Lightweight plotting to the terminal. 4x resolution via Unicode.

[![uniplot demo GIF](https://github.com/olavolav/uniplot/raw/master/resource/uniplot-demo.gif)](https://asciinema.org/a/Ldgn5pHOgxPJmIf2ZvlfIPR3L)

When working with production data science code it can be handy to have plotting
tool that does not rely on graphics dependencies or works only in a Jupyter notebook.

The **use case** that this was built for is to have plots as part of your data science /
machine learning CI pipeline - that way whenever something goes wrong, you get not only
the error and backtrace but also plots that show what the problem was.


## Features

* Unicode drawing, so 4x the resolution (pixels) of usual ASCII plots
* Super simple API
* Interactive mode (pass `interactive=True`)
* Color mode (pass `color=True`) useful in particular when plotting multiple series
* It's fast: Plotting 1M data points takes 100ms thanks to NumPy magic
* Only one dependency: NumPy (but you have that anyway don't you)

Please note that Unicode drawing will work correctly only when using a font that
fully supports the [Box-drawing character set](https://en.wikipedia.org/wiki/Box-drawing_character).
Please refer to [this page for a (incomplete) list of supported fonts](https://www.fileformat.info/info/unicode/block/block_elements/fontsupport.htm).


## Examples

Note that all the examples are without color and plotting only a single series od data. For using color see the GIF example above.

### Plot sine wave

```
>>> import math
>>> x = [math.sin(i/20)+i/300 for i in range(600)]
>>> from uniplot import plot
>>> plot(x, title="Sine wave")
                          Sine wave
┌────────────────────────────────────────────────────────────┐
│                                                    ▟▀▚     │
│                                                   ▗▘ ▝▌    │
│                                       ▗▛▜▖        ▞   ▐    │
│                                       ▞  ▜       ▗▌    ▌   │ 2
│                           ▟▀▙        ▗▘  ▝▌      ▐     ▜   │
│                          ▐▘ ▝▖       ▞    ▜      ▌     ▝▌  │
│              ▗▛▜▖        ▛   ▜      ▗▌    ▝▌    ▐▘      ▜  │
│              ▛  ▙       ▗▘   ▝▖     ▐      ▚    ▞       ▝▌ │
│  ▟▀▖        ▐▘  ▝▖      ▟     ▚     ▌      ▝▖  ▗▌        ▜▄│ 1
│ ▐▘ ▐▖       ▛    ▙      ▌     ▐▖   ▗▘       ▚  ▞           │
│ ▛   ▙      ▗▘    ▐▖    ▐       ▙   ▞        ▝▙▟▘           │
│▐▘   ▐▖     ▐      ▌    ▛       ▐▖ ▗▘                       │
│▞     ▌     ▌      ▐   ▗▘        ▜▄▛                        │
│▌─────▐────▐▘───────▙──▞────────────────────────────────────│ 0
│       ▌   ▛        ▝▙▟▘                                    │
│       ▜  ▐▘                                                │
│        ▙▄▛                                                 │
└────────────────────────────────────────────────────────────┘
         100       200       300       400       500       600
```

### Plot global temperature data

Here we're using Pandas to load and prepare gloabl temperature data from the [Our World in Data GitHub repository](https://github.com/owid/owid-datasets).

First we load the data, rename a column and and filter the data:
```
>>> import pandas as pd
>>> uri = "https://github.com/owid/owid-datasets/raw/master/datasets/Global%20average%20temperature%20anomaly%20-%20Hadley%20Centre/Global%20average%20temperature%20anomaly%20-%20Hadley%20Centre.csv"
>>> data = pd.read_csv(uri)
>>> data = data.rename(columns={"Global average temperature anomaly (Hadley Centre)": "Global"})
>>> data = data[data.Entity == "median"]
```

Then we can plot it:
```
>>> from uniplot import plot
>>> plot(xs=data.Year, ys=data.Global, lines=True, title="Global normalized land-sea temperature anomaly [C]", y_min=-0.8, y_max=0.8)
      Global normalized land-sea temperature anomaly [C]
┌────────────────────────────────────────────────────────────┐
│                                                          ▞▀│ 0.8
│                                                         ▐  │
│                                           ▖     ▗   ▗  ▗▌  │
│                                          ▗▜  ▞▀▄▘▀▚▐▘▚▄▘   │
│                                          ▞▝▖▐      ▘ ▝     │ 0.4
│                                    ▞▖  ▞▖▌ ▀▘              │
│                              ▄  ▗▞▄▘▐ ▞ ▜                  │
│    ▖                ▗    ▗▄▀▄▜  ▞ ▝  ▀▘                    │
│▖──▞▐──▗▀▀▄▀▀▌───▗▚▖─▞▖──▙▞──▝─▙▞▘──────────────────────────│ 0
│▝▖▞  ▌ ▐  ▘  ▚ ▞▀▞ ▚▐ ▌ ▐ ▘                                 │
│ ▝   ▝▖▌     ▐▞    ▝▌ ▚▀▞                                   │
│      ▝▘                ▘                                   │
│                                                            │ -0.4
│                                                            │
│                                                            │
│                                                            │
│                                                            │ -0.8
└────────────────────────────────────────────────────────────┘
1,950    1,960    1,970   1,980    1,990    2,000   2,010
```

Next for example we might be interested to plot how far the most recent temperature value is outside of the distribution of pre-1970 values:
```
>>> temperature_pre1980 = data[data["Year"] <= 1980].Global
>>> current_temperature = data.Global.iloc[-1]
>>> histogram(temperature_pre1980, title="Distribution of pre-1980 values. Line indicates current", x_min=-1, x_max=1, bins=5, x_gridlines=[current_temperature])
   Distribution of pre-1980 values. Line indicates current
┌────────────────────────────────────────────────────────────┐
│                            ▐▀▀▌                     │      │ 10
│                            ▐  ▌                     │      │
│                            ▐  ▌                     │      │
│                          ▛▀▀  ▌                     │      │
│                          ▌    ▌                     │      │
│                       ▐▀▀▘    ▌                     │      │
│                       ▐       ▌                     │      │
│                       ▐       ▌                     │      │
│                       ▐       ▀▀▜                   │      │
│                       ▐         ▐                   │      │
│                       ▐         ▐                   │      │
│                    ▗▄▄▟         ▐                   │      │
│                    ▐            ▐                   │      │
│                    ▐            ▐                   │      │
│                    ▐            ▐                   │      │
│                    ▐            ▐                   │      │
│▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▐▁▁▁▁▁▁▁▁▁▁▁▁▐▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁│▁▁▁▁▁▁│ 0
└────────────────────────────────────────────────────────────┘
-1                             0                             1
```


## Parameters

The `plot` function accepts a number of parameters, all listed below. Note that only
`ys` is required, all others are optional.

There is also a `plot_to_string` function with the same signature, if you want the result as a list of strings, to include the output elsewhere.

### Data

* `xs` - The x coordinates of the points to plot. Can either be `None`, or a list or NumPy array for plotting a single series, or a list of those for plotting multiple series. Defaults to `None`, meaning that the x axis will be just the sample index of
`ys`.
* `ys` - The y coordinates of the points to plot. Can either be a list or NumPy array for plotting a single series, or a list of those for plotting multiple series.

### Options

In alphabetical order:

* `color` - Draw series in color. Defaults to `False` when plotting a single series, and to `True` when plotting multiple.
* `height` - The height of the plotting region, in characters. Default is `17`.
* `interactive` - Enable interactive mode. Defaults to `False`.
* `legend_labels` - Labels for the series. Can be `None` or a list of strings. Defaults to `None`.
* `lines` - Enable lines between points. Can either be `True` or `False`, or a list of those values for plotting multiple series. Defaults to `False`.
* `line_length_hard_cap` - Enforce a hard limit on the number of characters per line of the plot area. This may override the `width` option if there is not enough space. Defaults to `None`.
* `title` - The title of the plot. Defaults to `None`.
* `width` - The width of the plotting region, in characters. Default is `60`. Note that if the `line_length_hard_cap` option is used and there is not enough space, the actual width may be smaller.
* `x_gridlines` - A list of x values that have a vertical line for better orientation. Defaults to `[0]`.
* `x_max` - Maximum x value of the view. Defaults to a value that shows all data points.
* `x_min` - Minimum x value of the view. Defaults to a value that shows all data points.
* `y_gridlines` - A list of y values that have a horizontal line for better orientation. Defaults to `[0]`.
* `y_max` - Maximum y value of the view. Defaults to a value that shows all data points.
* `y_min` - Minimum y value of the view. Defaults to a value that shows all data points.


## Experimental features

For convenience there is also a `histogram` function that accepts one or more series and
plots bar-chart like histograms. It will automatically discretize the series into a
number of bins given by the `bins` option and display the result.

When calling the `histogram` function, the `lines` option is `True` by default.

Example:

```
>>> import numpy as np
>>> x = np.sin(np.linspace(1, 1000))
>>> from uniplot import histogram
>>> histogram(x)
┌────────────────────────────────────────────────────────────┐
│   ▛▀▀▌                       │                   ▐▀▀▜      │ 5
│   ▌  ▌                       │                   ▐  ▐      │
│   ▌  ▌                       │                   ▐  ▐      │
│   ▌  ▀▀▀▌                    │                ▐▀▀▀  ▝▀▀▜   │ 4
│   ▌     ▌                    │                ▐        ▐   │
│   ▌     ▌                    │                ▐        ▐   │
│   ▌     ▙▄▄▄▄▄▖              │          ▗▄▄▄  ▐        ▐   │ 3
│   ▌           ▌              │          ▐  ▐  ▐        ▐   │
│   ▌           ▌              │          ▐  ▐  ▐        ▐   │
│   ▌           ▌              │          ▐  ▐  ▐        ▐   │
│   ▌           ▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▜  ▐▀▀▀  ▝▀▀▀        ▐   │ 2
│   ▌                          │    ▐  ▐                 ▐   │
│   ▌                          │    ▐  ▐                 ▐   │
│   ▌                          │    ▐▄▄▟                 ▐   │ 1
│   ▌                          │                         ▐   │
│   ▌                          │                         ▐   │
│▄▄▄▌▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁│▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▐▄▄▄│ 0
└────────────────────────────────────────────────────────────┘
     -1                        0                       1
```


## Installation

Install via pip using:

```
pip install uniplot
```


## Roadmap

Coming up:

* Fill area under curve
* Log scales
* Add generated page with list of supported fonts
* Auto-detect color mode depending on terminal capabilities
* Possibly: Fallback to ASCII characters

Input is always welcome, let me know what is most needed for this to be as useful as possible.


## Contributing

Clone this repository, and install dependecies via `pip3 install -e .\[dev\]`.
You can run the tests vie `./run_tests` to make sure your setup is good. Then
proceed with issues, PRs etc. the usual way.
