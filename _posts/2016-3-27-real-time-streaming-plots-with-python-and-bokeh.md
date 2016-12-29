---
layout: post
title: Real Time Streaming Plots with Python and Bokeh
---

Playing with Bokeh to build a continuously updating plotting library.

> Bokeh is a Python interactive visualization library that targets modern web browsers for presentation. Its goal is to provide elegant, concise construction of novel graphics in the style of D3.js, but also deliver this capability with high-performance interactivity over very large or streaming datasets.

- Bokeh.pydata.org

Bokeh [(github)](https://github.com/bokeh/bokeh) is a really great way to produce interactive and visually appealing web graphics and apps purely in Python. It's a new library and I've run into bugs here and the documentation could be better but it's open source and very straight-forward to use. I've included references to where I found things to help others debug their code.

*Note - In the post I'm using [Anaconda Python 3.5](https://docs.continuum.io/anaconda/), [Jupyter Notebooks](https://jupyter.org/), and [Bokeh 0.11.1](http://bokeh.pydata.org/en/latest/). For this example to work you need to launch the Bokeh Server from your command prompt or terminal.*

# Real Time Plotting and ColumnDataSource

Bokeh comes with the ability to "link" data together. It allows two different charts to share the same value as another, and if the value updates from one chart it will update for the other.

```python
import numpy as np
from bokeh.plotting import figure
from bokeh.models.sources import ColumnDataSource
output_notebook() #you can also output to a file

antialias = 10
N = 10
x = np.linspace(0, N, N*antialias)
y1 = np.sin(x)
y2 = np.cos(x)

myfigure = figure(plot_width=800, plot_height=400)
datacoords = ColumnDataSource(data=dict(x=x, y1=y1, y2=y2))
linea = myfigure.line("x", "y1", source=datacoords)
lineb = myfigure.line("x", "y2", source=datacoords)
```

This code initializes a Figure with two lines that use a [ColumnDataSource](http://bokeh.pydata.org/en/0.10.0/docs/reference/models/sources.html#bokeh.models.sources.ColumnDataSource) object as their data. When we change this data we can change what is displayed on the graph.

## Creating an Update Loop

```python
from bokeh.driving import linear

@linear(m=0.05, b=0) #step will increment by 0.05 every time
def update(step):
     new_x = np.linspace(step, N+step, N*antialias)
     new_y1 = np.sin(new_x)
     new_y2 = np.cos(new_x)

     linea.data_source.data["x"] = new_x
     linea.data_source.data["y1"] = new_y1
     lineb.data_source.data["y2"] = new_y2
```

This next part is the callback function. There are a few restrictions to what you are allowed to do in the callback and these restrictions reduce the amount of freedom you have with animation.

linea.data_source.data["x"] returns an [ndarray](https://docs.scipy.org/doc/numpy-1.10.0/reference/generated/numpy.ndarray.html) which you can't add new elements to. That, along with the ndarray's incompatibility with python lists and indexing makes animating graphs a lot complicated. The work around is to change the array to a new one that we generate. I generated mine with sin(), cos(), and linspace() but you could also have this callback pull data from some external source.

#### Callback Restriction 1: Do not edit line.data_source

linea.data_source returns a ColumnDataSource object but you can't add values to it or swap it our for a new one like we can with our ndarray or else you get Undefined Behavior. Sometimes your graph will flicker back and forth, sometimes it won't move at all, but it does not work.

#### Callback Restriction 2: You can only read and update, not reassign, global data

Due to the way the Python's scoping works if I have a variable declared in an outer scope and access it's value from a function within that scope, the function will have access to that variable's data. The catch is that if the inner function tries to reassign any data then that variable becomes a local variable and your function loses access to that data. The explanation is better described [here](http://www.python-course.eu/python3_global_vs_local_variables.php). This makes it more difficult to update the graph how you would like.

## Adding Callback Functionality

```python
from bokeh.client import push_session
from bokeh.plotting import curdoc

# open a session to keep our local document in sync with server
session = push_session(curdoc())
curdoc().add_periodic_callback(update, 10) #period in ms
session.show() # open the document in a browser
session.loop_until_closed()
```

I got this code from looking at examples and tutorials on bokeh. I have no idea how they found this code because this part of Bokeh is not very documented and I had to look around in the library code to find out how it all fit together.

## Problems with Jupyter Notebooks

session.loop_until_closed() can cause the Python kernel to crash and will lead to undefined behavior. I highly recommend restarting the kernel and clearing the output after every time you run the program.

# Full Code

```python
import numpy as np
from bokeh.plotting import figure, curdoc
from bokeh.models.sources import ColumnDataSource
from bokeh.client import push_session
from bokeh.driving import linear

antialias = 10
N = 10
x = np.linspace(0, N, N*antialias)
y1 = np.sin(x)
y2 = np.cos(x)

myfigure = figure(plot_width=800, plot_height=400)
datacoords = ColumnDataSource(data=dict(x=x, y1=y1, y2=y2))
linea = myfigure.line("x", "y1", source=datacoords)
lineb = myfigure.line("x", "y2", source=datacoords)

@linear(m=0.05, b=0) #step will increment by 0.05 every time
def update(step):
    new_x = np.linspace(step, N+step, N*antialias)
    new_y1 = np.sin(new_x)
    new_y2 = np.cos(new_x)

    linea.data_source.data["x"] = new_x
    linea.data_source.data["y1"] = new_y1
    lineb.data_source.data["y2"] = new_y2

# open a session to keep our local document in sync with server
session = push_session(curdoc())
curdoc().add_periodic_callback(update, 10) #period in ms
session.show()
session.loop_until_closed()
```
