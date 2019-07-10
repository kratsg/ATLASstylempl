# ATLAS-style matplotlib

## Requirements

```
pip install matplotlib
```

## How does it work?

All the magic is in [.matplotlib/stylelib/atlas.mplstyle](.matplotlib/stylelib/atlas.mplstyle). Matplotlib comes with [rcParams](https://matplotlib.org/users/customizing.html) stylesheets that allow you to customize *most* of the default formatting. This is done by pointing matplotlib to your stylesheet location via the `$MPLCONFIGDIR` environment variable when using matplotlib. In Python code, this can be done with something like this:

```python
# to use atlas-style, need to point matplotlib at location for it
import os
os.environ['MPLCONFIGDIR'] = os.path.realpath(os.path.join(os.path.dirname(__file__),'.matplotlib'))
import matplotlib.pyplot as plt
plt.style.use('atlas')
```

and the ATLAS style should be loaded, assuming [.matplotlib](.matplotlib/) is in the same directory as the python script you're using. If, for example, you preferred to clone this repository directly:

```python
os.environ['MPLCONFIGDIR'] = os.path.realpath(os.path.join(os.path.dirname(__file__),'ATLASstylempl','.matplotlib'))
```

should work. Modify this as needed. The point is that the environment variable should just be changed before importing `matplotlib` and the stylesheet will get loaded.

## Caveats

### sans-serif not found?

Are you getting warnings about sans-serif not found? Read the following posts:
- https://olgabotvinnik.com/blog/how-to-set-helvetica-as-the-default-sans-serif-font-in/
- https://gist.github.com/alexrudy/a7982903a2fb2ab0dde3

The tl;dr is that on Macs, the Helvetica font (which ATLAS uses by default) is `Helvetica.dfont` rather than a true-type font. Really, it's just stored as a [different format](https://xkcd.com/927/) that (unsurprisingly) matplotlib doesn't support. Instead, the solution is to convert from `Helvetica.dfont` to `Helvetica.ttf` so that `matplotlib` will just use it. This is done with the following steps:

1. `brew install fondu`
2. Find Helvetica on your system (`/System/Library/Fonts/Helvetica.dfont`) or use `FontBook.app`
3. Copy font over so you don't mess up the original copy: `mkdir ~/font_copies; cp /System/Library/Fonts/Helvetica.dfont ~/font_copies`
4. Go to where `matplotlib` stores the true-type fonts: `cd $(python -c "import matplotlib, os; print(os.path.dirname(matplotlib.matplotlib_fname()))")/fonts/ttf`
5. Run fondu to convert the `dfont`: `sudo fondu -show ~/font_copies`
6. Check that it worked: `ls -1 Helvetica*`
7. Clear matplotlib's cache: `rm ~/.matplotlib/fontList.cache`
8. Rebuild matplotlib's cache: `python -c "import matplotlib.font_manager; matplotlib.font_manager._rebuild()"`
9. Remove the temporary folder you made: `rm -rf ~/font_copies`
10. Have a :beer:

### x/y axis labels aren't padded correctly?

The way I've designed the stylesheet was to try and keep the settings uniform for setting up labels. Therefore, you should add vertical/horizontal alignment like so:

```python
fig, ax = plt.subplots()

ax.set_xlabel(r'x-label', horizontalalignment='right', x=1.0, verticalalignment='bottom', y=0.0)
ax.set_ylabel(r'y-label', horizontalalignment='right', y=1.0, verticalalignment='bottom', x=0.0)
```

### why didn't you do so-and-so with the stylesheet too???

What cannot be set with `rcParams`:
- markeredgewidth
- axis-label alignment
- axis-label offset
- axis-ticks

If you want to change something about individual axes, you can only do it from within the Python code itself.


### How do I add the ATLAS label? (internal, prelim, etc..)

So this isn't too hard that I didn't want to complicate this repository a lot, but... here's how I would do it

```python
fig, ax = plt.subplots()

# dictionaries to hold the styles for re-use
text_fd = dict(ha='left', va='center')
atlas_fd = dict(weight='bold', style='italic', size=24, **text_fd)
internal_fd = dict(size=24, **text_fd)

# actually drawing the text
ax.text(x,y,'ATLAS', fontdict=atlas_fd)
ax.text(x+shift,y, 'Internal', fontdict=internal_fd)
```

The `Internal` or `Preliminary` or `Simulation` labels can be added horizontally after the `ATLAS` label by some number of units (depending on what you're plotting).

## Credit

Most of the credit should go to the [rootpy](https://github.com/rootpy/rootpy/tree/457e074056a916fff848978ef68b7f5107856e47/rootpy/plotting/style/atlas) project for figuring out most of these styles. I've tweaked the stylesheet there based on the current evolution/iteration of the ATLAS style as of mid-July 2019.
