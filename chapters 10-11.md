# Chapter 10

pandas is the central tool in Python for data analysis and data handling. This chapter introduces time series, the concept of dataframes, and how to access and visualize data. pandas interacts smoothly with NumPy and Matplotlib.

Main tasks:

- A guiding example: solar cells
- NumPy arrays and pandas DataFrames
- Creating and modifying dataframes
- Working with dataframes (plotting, calculations, grouping)

## 10.1 A guiding example: Solar cells

To illustrate pandas we use production data from solar cell panels on the roof of a private house in the south of Sweden. Three data files are provided, each a **time series** (data depending on time):

| File | Content | Frequency | Separator |
| --- | --- | --- | --- |
| `solarWatts.dat` | Electricity production in watts | per minute | `;` |
| `price.dat` | Hourly electricity price in Swedish crowns (SEK) | hourly | `;` |
| `rates.dat` | Daily conversion rate SEK → Euro | daily | `;` |

Each file has a header line. For example `solarWatts.dat`:

```
Date;Watt
2019-10-20 08:22:00 ; 44.0
2019-10-20 08:23:00 ; 61.0
2019-10-20 08:24:00 ; 42.0
```

The data is not collected at the same time points and there might be missing data. We want to extract information such as the maximal/minimal production per day, hours of sun per month, the sunniest day, and some economic information, and to present the data graphically.

## 10.2 NumPy arrays and pandas dataframes

A NumPy array:

```python
A = array([[1., 2., 3.],
           [4., 5., 6.]])
```

is displayed as a plain grid and its elements are accessed by counting rows and columns, e.g. `A[0, 1]`. The same data can be converted to a pandas `DataFrame`:

```python
import pandas as pd
A = array([[1., 2., 3.],
           [4., 5., 6.]])
AF = pd.DataFrame(A)
```

A pandas dataframe has extra labels for the rows and columns, called **index** and **columns**. These are the metadata of a dataframe. They can be set explicitly:

```python
AF.columns = ['C1', 'C2', 'C3']
AF.index = ['R1', 'R2']
```

giving a labelled table:

```
     C1   C2   C3
R1  1.0  2.0  3.0
R2  4.0  5.0  6.0
```

### 10.2.1 Indexing rules

Similar to the way dictionaries use keys, pandas dataframes use row labels (the index) and column labels to access values, through the `loc` attribute:

```python
AF.loc['R1', 'C2']                      # returns 2.0
AF.loc[['R1', 'R2'], ['C1', 'C2']]      # returns a subframe
AF.loc['R1']                            # returns a pandas Series (a row)
AF.loc['R1'].loc['C1']                  # returns 1.0
```

An entire column is addressed directly with `AF['C1']` (returns a `Series`), or as an attribute `AF.C1`. A single column is an instance of the pandas datatype `Series` and has no column label:

```python
type(AF.C1) == pd.Series   # True
```

Classical integer-position indexing is available through `iloc`:

```python
AF.iloc[[0], [1]]          # returns the subframe with element C2/R1
```

The key rules, in agreement with NumPy:

| Access | Result |
| --- | --- |
| `loc`/`iloc` with a pair of single labels | a single element (`AF.loc['R1', 'C2']`) |
| `loc`/`iloc` with list arguments or slices | a dataframe (`AF.loc['R1':, 'C2':]`) |
| single column `AF['C1']` or `AF.C1` | a `Series` |

Note that `loc` and `iloc` are not methods but attributes implementing `__getitem__`, which is why brackets `[]` are used instead of parentheses.

## 10.3 Creating and modifying dataframes

### 10.3.1 Creating a dataframe from imported data

The data files are in CSV format, here using a semicolon as separator (in many countries the comma is the decimal separator). We want the dates as the index, automatically converted to pandas `Timestamp` objects, and we want pandas to recognize the ISO date format `YY-MM-DD`:

```python
solarWatts = pd.read_csv("solarWatts.dat",
                         sep=';',
                         index_col='Date',
                         parse_dates=[0], infer_datetime_format=True)
```

`read_csv` is the central import tool and has many more parameters. The resulting dataframe has more than 200,000 records. We can inspect it:

```python
solarWatts.iloc[0]          # first record
solarWatts.index[-1]        # last index, a Timestamp('2020-06-27 17:54:00')
```

`Timestamp` objects make it easy to compute with dates, define ranges, and compare dates. Their difference is a `Timedelta`:

```python
solarWatts.index[1] - solarWatts.index[0]   # Timedelta('0 days 00:01:00')
```

Because pandas is compatible with NumPy, we can apply `numpy.diff` to the index (returning a `timedelta64[ns]` array), convert to minutes, and find the largest gap and its date:

```python
max(numpy.diff(solarWatts.index).astype('timedelta64[m]'))
solarWatts.iloc[np.argmax(np.diff(solarWatts.index))]
```

### 10.3.2 Setting the index

The default index of a dataframe is the automatically generated row numbers. We build a dataframe from a list of lists:

```python
towns = [['Stockholm', 'Sweden', 188, 975904],
         ['Malmö',     'Sweden', 322, 316588],
         ['Oslo',      'Norway', 481, 693491],
         ['Bergen',    'Norway', 464, 283929]]
town = pd.DataFrame(towns, columns=['City', 'Country', 'area', 'population'])
```

A column can be chosen as the index with `set_index`. With `drop=False` the column is duplicated (kept in the data); with `drop=True` (default) it is moved out of the data section:

```python
town.set_index('City', drop=False)   # duplicating
town.set_index('City', drop=True)    # dropping the column and making it the index
```

The parameter `inplace` changes the dataframe directly without generating a new object. pandas allows several columns to be used as a **multi-index**, opening hierarchical indexing:

```python
town.set_index(['Country', 'City'], inplace=True)
```

The first index (`Country`) has the higher hierarchy level. We can then address a whole group or a particular row:

```python
town.loc['Sweden']                  # all Swedish towns
town.loc[('Sweden', 'Malmö')]       # a particular one
```

### 10.3.3 Deleting entries

Entries are deleted with the method `drop`. The parameter `axis` specifies whether a row (`axis=0`) or a column (`axis=1`) is removed:

```python
town = pd.DataFrame(towns, columns=['City', 'Country', 'area', 'population'])
town.set_index('City', inplace=True)
town.drop('Bergen', axis=0)    # delete an entire row
town.drop('area', axis=1)      # delete a column
```

### 10.3.4 Merging dataframes

The three files give three dataframes `solarWatts`, `price`, and `rates`. They are merged into one with `pd.merge`:

```python
solar_all = pd.merge(solarWatts, price, how='outer', sort=True, on='Date')
solar_all = pd.merge(solar_all, rates, how='outer', sort=True, on='Date')
```

The common column `Date` becomes the new index. The parameter `how='outer'` takes the **union** of both index columns, and `sort=True` sorts the index. Because `solarWatts` has per-minute data while the price changes hourly, missing entries appear, filled automatically with `NaN`:

```
                     Watt  SEK  Euro_SEK
Date
2019-10-06 15:03:00  4145.0  NaN  NaN
2019-10-06 15:04:00  5784.0  NaN  NaN
```

### 10.3.5 Missing data in a dataframe

The way missing data is indicated depends on the column datatype:

| Datatype | Missing value |
| --- | --- |
| numeric | `NaN` |
| timestamp | `NaT` |
| other non-numeric | `None` |

The method `isnull` returns a Boolean dataframe with `True` at every place with missing data. We demonstrate the handling methods on a small dataframe:

```python
frame = pd.DataFrame(array([[1., -5.,  3.,  NaN],
                            [3.,  4.,  NaN, 17.],
                            [6.,  8.,  11.,  7.]]),
                     columns=['a', 'b', 'c', 'd'])
```

| Operation | Effect |
| --- | --- |
| `frame.dropna(axis=0)` | drop all rows containing missing data |
| `frame.dropna(axis=1)` | drop all columns containing missing data |
| `frame.fillna(method='pad', axis=0)` | pad with data from the previous row |
| `frame.fillna(method='bfill', ...)` | back-fill from the following row |
| `frame.interpolate(axis=0, method='linear')` | interpolate numeric data column-wise |

Values that cannot be padded or interpolated remain `NaN`. Different methods can be applied per column using `inplace`:

```python
frame['c'].fillna(method='pad', inplace=True)
frame['d'].fillna(method='bfill', inplace=True)
```

For the solar cell example, we pad the hourly/daily values that became `NaN` after merging:

```python
solar_all['SEK'].fillna(method='pad', axis=0, inplace=True)
solar_all['Euro_SEK'].fillna(method='pad', axis=0, inplace=True)
```

The `Watt` column still contains `NaN` outside daylight periods; these are simply ignored in plots.

## 10.4 Working with dataframes

We now turn to data interpretation: visualization, simple calculations, and grouping.

### 10.4.1 Plotting from dataframes

pandas' `plot` command is built upon `plot` from `matplotlib.pyplot` and accepts the same parameters (e.g. `linestyle`, `marker`). The x-axis data is taken from the index unless specified otherwise. Indexing with a full day is a short form of slicing:

```python
solar_all.loc['2020-05-16'].plot(y='SEK')
solar_all.loc['2020-05-16 00:00':'2020-05-16 23:59']   # equivalent slice
```

Line plots leave gaps where data is missing. Working with an axis object lets us modify labels and legend:

```python
ax1 = solar_all.loc['2020-06-20':'2020-06-21'].plot(None, 'Watt')
ax1.set_ylabel('Power')
ax1.legend(['Power [W]'])
```

### 10.4.2 Calculations within dataframes

Simple calculations operate on columns directly (elementwise). Functions may be built-in, NumPy, or user-defined (e.g. lambdas). Here we convert watts to kilowatts and SEK to Euros:

```python
solar_converted = pd.DataFrame()
solar_converted['kW'] = solar_all['Watt'] / 1000
solar_converted['Euro'] = solar_all['SEK'] / solar_all['Euro_SEK']
```

NumPy's universal functions apply to entire columns:

```python
import numpy as np
np.max(solar_all['Watt'])                            # returns 12574
print(solar_all.index[np.argmax(solar_all['Watt'])])  # 2020-05-16 10:54:00
```

Missing data marked by `NaN` is treated as if it were not there. As not all methods have this property, it can be safer to replace `NaN` with 0:

```python
solar_all['Watt'].fillna(value=0., inplace=True)
```

For general user-defined functions, the dataframe method `apply` performs row- or column-wise operations on the entire dataframe.

### 10.4.3 Grouping data

Grouping is one of pandas' essential features. We form groups and aggregate the data with `groupby` and `agg`. The following reports the peak power per day and the average price:

```python
solar_day = solar_all.groupby(solar_all.index.date).agg({'Watt': 'max',
                                                          'SEK': 'mean'})
```

The result can be visualized as a bar plot, building an axis object to relabel the ticks:

```python
solar_day.index = pd.to_datetime(solar_day.index, format='%Y-%m-%d')
ax = solar_day.loc['2020-06-01':'2020-06-30'].plot.bar('Watt')
ax.set_xticklabels([tf.strftime("%m-%d")
                    for tf in solar_day.loc['2020-06-01':'2020-06-30'].index])
```

We can also group by hours, for instance to check whether the price has two peaks per day:

```python
solar_hour = solar_all.groupby(solar_all.index.hour).agg({'SEK': mean})
ax = solar_hour.plot(marker='*')
ax.set_title('The average energy price change on a day')
ax.set_xlabel('hour of day')
ax.set_ylabel('SEK/kWh')
```

Grouping is often the starting point for further computations. To get the hourly energy output (kWh) from the per-minute power, we group hierarchically by year, month, day, and hour, then integrate by summing:

```python
grouping_list = [solar_all.index.year, solar_all.index.month,
                 solar_all.index.day, solar_all.index.hour]
solar_hour = solar_all.groupby(grouping_list)
# integrating by summing up the data
solar_hour = solar_hour.agg({'Watt': sum})
solar_hour = solar_hour / (1000 * 60)   # conversion from Wmin to kWh
```

The hierarchical index is then addressed with a tuple:

```python
ax = solar_hour['Watt'].loc[(2020, 6, 19)].plot.bar()
ax.set_title('Energy production on June, 19 2020')
ax.set_xlabel('Hour')
ax.set_ylabel('Energy [kWh]')
```

Alternatively, `scipy.integrate.simps` could be passed to `agg` for the integration of discrete data, but as it does not treat missing data, all `NaN` values would have to be replaced with 0 first.

## 10.5 Summary

This chapter gave a brief introduction to pandas, showing how the NumPy array concept is extended to dataframes. Guided by the solar cell energy example, we covered the first steps with pandas: setting up a dataframe from a file, merging frames, treating missing data, grouping data, and making calculations.

# Chapter 11

Graphical user interfaces (GUIs) are convenient tools to input user data to a Python program. This chapter shows how to add tools such as sliders and buttons using the **Matplotlib** module. Although alternatives such as Tkinter exist for more sophisticated GUIs, Matplotlib is an ideal low-threshold entry point. The roles of events, slider movements, mouse clicks, and their interaction with **call-back functions** are explained.

Main topics:

- A guiding example to widgets
- The button widget and mouse events

## 11.1 A guiding example to widgets

The guiding example is a widget with a slider bar at the top: moving the blue bar changes a frequency value (between 1 and 5), and the sine wave displayed in the plot window updates accordingly. The widget consists of three parts:

- A figure object with an axes object and the plot
- An axes object containing a slider object
- A call-back function for updating the plot when the slider value changes

First, we create a figure, an axes object for the plot, and an initial sine curve:

```python
from matplotlib.pyplot import *
fig = figure(figsize=(4, 2))
ax = axes([0.1, 0.15, 0.8, 0.7])   # axes for the plot
omega = float(input('Give a value for $\omega$ between 1 and 5:\n'))
x = linspace(-2*pi, 2*pi, 800)
ax.set_ylim(-1.2, 1.2)
lines, = ax.plot(x, sin(2.*pi*omega*x))
```

Next, we add a second axes object and place a slider into it:

```python
from matplotlib.widgets import Slider
sld_ax = axes([0.2, 0.9, 0.6, 0.05])   # axes for slider
sld = Slider(sld_ax, '$\omega$ [Hz]', 1., 5., valinit=1.)
omega = sld.val
```

The `Slider` constructor uses the slider axes, a label, and minimum/maximum values. The slider has an attribute `val` holding the value at the current slider position; initially it is set to `valinit`.

The core part is the **call-back function**, which updates the plot whenever the slider value changes. The slider method `on_changed` defines the action; the call-back is called with the slider value `val` as its single argument:

```python
def update_frequency(omega):
    lines.set_ydata(np.sin(2.*pi*omega*x))

sld.on_changed(update_frequency)
```

Putting all the parts together (no `input` needed any more), and combining string formatting with LaTeX commands in the legend:

```python
from matplotlib.pyplot import *
from matplotlib.widgets import Slider

fig = figure(figsize=(4, 2))
sld_ax = axes([0.2, 0.9, 0.6, 0.05])   # axes for slider
ax = axes([0.1, 0.15, 0.8, 0.7])       # axes for the plot
ax.xaxis.set_label_text('Time [s]')
ax.yaxis.set_label_text('Amplitude [m]')
sld = Slider(sld_ax, '$\omega$ [Hz]', 1., 5., valinit=1.5)
omega = sld.val
x = linspace(-2*pi, 2*pi, 800)
ax.set_ylim(-1.2, 1.2)
lines, = ax.plot(x, sin(2.*pi*omega*x), label=f'$\sin(2\pi\; {omega} x)$ ')
ax.legend()

def update_frequency(omega):
    lines.set_ydata(np.sin(2.*pi*omega*x))
    ax.legend((lines,), (f'$\sin(2\pi\; {omega} x)$',))

sld.on_changed(update_frequency)
```

### 11.1.1 Changing a value with a slider bar

The most important slider attribute is its value, `val`, communicated to the call-back. Other attributes:

| Attribute | Meaning |
| --- | --- |
| `valmin`, `valmax` | the limits of the value given by the slider |
| `valinit` | the initial slider position |
| `valstep` | stepping to make the change discrete |
| `valfmt` | formatting string for how the value is displayed |
| `orientation` | `'horizontal'` (default) or `'vertical'` |

A slider with explicit attributes:

```python
sld = Slider(sld_ax, label='$\omega$ [Hz]', valmin=1., valmax=5.,
             valinit=1.5, valfmt='%1.1f', valstep=0.1)
```

The format `%1.1f` displays a floating-point number with one digit on each side of the decimal point.

**An example with two sliders** — one for amplitude, one for frequency, in vertical mode:

```python
sldo_ax = axes([0.95, 0.15, 0.01, 0.6])   # axes for frequency slider
slda_ax = axes([0.85, 0.15, 0.01, 0.6])   # axes for amplitude slider

sld_omega = Slider(sldo_ax, label='$\omega$ [Hz]', valmin=1.,
                   valmax=5., valinit=1.5, valfmt='%1.1f',
                   valstep=0.1, orientation='vertical')
sld_amp = Slider(slda_ax, label='$a$ [m]', valmin=0.5,
                 valmax=2.5, valinit=1.0, valfmt='%1.1f',
                 valstep=0.1, orientation='vertical')
```

Both sliders have different call-back functions, each using its own slider value as the argument and the other slider's value as a global variable:

```python
def update_frequency(omega):
    lines.set_ydata(sld_amp.val*sin(2.*pi*omega*x))
    ax.legend((lines,), (f'${sld_amp.val} \sin(2\pi\; {omega} x)$',))

def update_amplitude(amp):
    lines.set_ydata(amp*sin(2.*pi*sld_omega.val*x))
    ax.legend((lines,), (f'${amp} \sin(2\pi\; {sld_omega.val} x)$',))
    ax.set_ylim(-(amp+0.2), amp+0.2)

sld_omega.on_changed(update_frequency)
sld_amp.on_changed(update_amplitude)
```

## 11.2 The button widget and mouse events

Some actions require waiting until a change has effect. It is often more user-friendly to first collect the changes with sliders and then trigger the update with a button.

### 11.2.1 Updating curve parameters with a button

A complicated graphical output might take time to update. In such cases, set the parameters with sliders and then press a button to initiate the update, using the `Button` widget:

```python
from matplotlib.widgets import Button
button_ax = axes([0.85, 0.01, 0.05, 0.05])   # axes for update button
btn = Button(button_ax, 'Update', hovercolor='red')
```

The button is labelled `Update` and turns red when the mouse hovers over it. It has the method `on_clicked` (used instead of the slider's `on_changed`):

```python
def update(event):
    lines.set_ydata(sld_amp.val*sin(2.*pi*sld_omega.val*x))
    ax.legend((lines,),
              (f'${sld_amp.val:1.1f} \sin(2\pi\; {sld_omega.val:1.1f} x)$',))

btn.on_clicked(update)
```

The call-back function has a single parameter, `event`. It is not used here, but it could assign different actions depending on how the mouse was clicked (single click, double click, right-button, or left-button).

### 11.2.2 Mouse events and textboxes

We can also catch a mouse event without using a button, by connecting a general button click to a call-back function. The example picks points on the sine curve by mouse clicks and displays their coordinates in a textbox; a right-click also marks the point with a red circle.

First, a `TextBox` widget with no label but some initial text. Its attribute `val` contains the text:

```python
from matplotlib.widgets import TextBox
textbox_ax = axes([0.85, 0.6, 0.1, 0.15])
txtbx = TextBox(textbox_ax, label='', initial='Clicked on:\nx=--\ny=--')
```

We change `val` depending on the position of a mouse click. A `Line2D` object `points` is initialized with empty data lists, which defines the plot style (red circles):

```python
points, = ax.plot([], [], 'ro')

def onclick(event):
    if event.inaxes == ax:
        txtbx.set_val(
            f'clicked on:\nx={event.xdata:1.1f}\ny={event.ydata:1.1f}')
        if event.button == 3:   # mouse button right
            points.set_xdata([event.xdata])
            points.set_ydata([event.ydata])
    else:
        txtbx.set_val('clicked on:\noutside axes\n area')
    fig.canvas.draw_idle()

cid = fig.canvas.mpl_connect('button_press_event', onclick)
```

As no button widget is used, we couple an event to a call-back with the canvas method `mpl_connect`. The call-back reacts to the mouse click position. Useful `event` attributes:

| Attribute | Meaning |
| --- | --- |
| `event.inaxes` | the axes object in which the click occurred |
| `event.button` | which mouse button was pressed (3 = right) |
| `event.xdata`, `event.ydata` | the coordinates of the mouse click |

The canvas method `draw_idle` redraws the figure after the update.

## 11.3 Summary

This chapter covered the basic principles of GUI programming within Matplotlib. Through a guiding example we learned about widgets — sliders, buttons, textboxes — together with mouse events and the call-back functions that connect user actions to updates of the plot.
