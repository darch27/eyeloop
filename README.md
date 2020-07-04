# EyeLoop [![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0) [![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/dwyl/esta/issues) [![Build Status](https://travis-ci.com/simonarvin/eyeloop.svg?branch=master)](https://travis-ci.com/simonarvin/eyeloop)
<p align="center">
<img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/logo.svg?raw=true" width = "300">
</p>
<p align="center">
<img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/eyeloop%20overview.svg?raw=true" width = "700">
</p>

<p align="center">
    <img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/sample_1.gif?raw=true" align="center" height="150">&nbsp; <img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/sample_2.gif?raw=true" align="center" height="150">&nbsp; <img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/sample_3.gif?raw=true" align="center" height="150">
  </p>

EyeLoop is a Python 3-based eye-tracker tailored specifically to dynamic, closed-loop experiments on consumer-grade hardware. This software is actively maintained: Users are encouraged to contribute to its development.


## Installation
Install EyeLoop simply by cloning the repository:
```
git clone https://github.com/simonarvin/eyeloop.git
```

**Dependencies:**

- numpy: ```python pip install numpy```

- opencv: ```python pip install opencv-python```

## Software logic
<p align="center">
<img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/software%20logic.svg?raw=true" width = "500">
</p>

EyeLoop consists of two domains: the engine and the optional modules. The engine performs the eye-tracking, whereas the modules are used to:

- Experiments
- Data acqusition
- Feed video sequence to the engine
- etc.

Thus, the modules import and/or extract data from the engine.

## Getting started
EyeLoop is initiated through the command-line interface.
```
python eyeloop.py
```
To access the video sequence, EyeLoop must be connected to an appropriate *importer class* module. Usually, the default opencv importer class (*cv*) is sufficient. For some machine vision cameras, however, a vimba-based importer (*vimba*) is neccessary.
```
python eyeloop.py --importer cv/vimba
```
> [Click here](https://github.com/simonarvin/eyeloop/blob/master/importers/README.md) for more information on importers.

To perform offline eye-tracking, we pass the video argument ```--video``` with the path of the video sequence:
```
python eyeloop.py --video [file]/[folder]
```
<p align="right">
    <img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/models.svg?raw=true" align="right" height="150">
</p>

EyeLoop can be used on a multitude of eye types, including rodents, human and non-human primates. Specifically, users can suit their eye-tracking session to any species using the ```--model``` argument.

```
python eyeloop.py --model ellipsoid/circular
```
> In general, the ellipsoid pupil model is best suited for rodents, whereas the circular model is best suited for primates.

To see all command-line arguments, pass:

```
python eyeloop.py --help
```

## Designing your first experiment

<p align="center">
    <img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/setup.svg?raw=true" align="center" height="250">
</p>

In EyeLoop, experiments are built by stacking modules. For your first experiment, we'll need a data acquisition class to log the data, and an experiment class to produce a stimulus. Both classes are passed to eyeloop.py as an *extractor array*:

```python
extractors = [Experiment(), DAQ()]
ENGINE.load_extractors(extractors)
```

Extractors are instantiated at start-up. At every subsequent time-step, the extractor's ```fetch()``` function is called by the engine.
```python
class Extractor:
    def __init__(self) -> None:
        ...
    def fetch(self, engine) -> None:
        ...
```
```fetch()``` gains access to all eye-tracking data in real-time via the engine pointer.

> [Click here](https://github.com/simonarvin/eyeloop/blob/master/extractors/README.md) for more information on extractor modules.

### Data acquisition class ###

For our data acquisition class, we define the directory path for our log using the instantiator:
```python
class DAQ:
    def __init__(self) -> None:
        self.log_path = ...
```
Then, we extract the eye-tracking data via ```fetch()``` and save it in JSON format.
```python
    ...
    def fetch(self, engine) -> None:
        self.log.write(json.dumps(engine.dataout)+"\n")
```

>Note, EyeLoop includes a standard data-parser utility (*parser.py*), that may be used to convert JSON into CSV. By using *parser.py*, users can easily compute and plot the eye's angular coordinates and the size of the pupil.

### Experiment class ###

For our experiment class, we'll design a simple *open-loop* where the brightness of a PC monitor is linked to the phase of the sine function. We define the sine frequency and phase using the instantiator:
```python
class Experiment:
    def __init__(self) -> None:
        self.frequency = ...
        self.phase = 0
```
By using ```fetch()```, we shift the phase of the sine function at every time-step, and use this to control the brightness of a cv-render.
```python
    ...
    def fetch(self, engine) -> None: 
        self.phase += self.frequency
        sine = numpy.sin(self.phase) * .5 + .5
        brightness = numpy.ones((height, width), dtype=float) * sine
        cv2.imshow("Experiment", source)
```

That's it! Test your experiment using:
```
python eyeloop.py
```
> See [Examples](https://github.com/simonarvin/eyeloop/blob/master/examples) for demo recordings and experimental designs.

## Graphical user interface
The default graphical user interface in EyeLoop is [*minimum-gui*.](https://github.com/simonarvin/eyeloop/blob/master/guis/minimum/README.md)

> EyeLoop is compatible with custom graphical user interfaces through its modular logic. [Click here](https://github.com/simonarvin/eyeloop/blob/master/guis/README.md) for instruction on how to build your own.

## Known issues
None yet.

## References
If you use any of this code or data, please cite [Arvin et al. 2020] (url).

## License
This project is licensed under the GNU General Public License v3.0. Note that the software is provided "as is", without warranty of any kind, express or implied. 

## Authors
    
**Lead Developer:**
Simon Arvin, sarv@dandrite.au.dk
<p align="right">
    <img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/constant.svg?raw=true" align="right" height="180">
    </p>

**Researchers:**
    
- Simon Arvin, sarv@dandrite.au.dk
- Rune Rasmussen, runerasmussen@biomed.au.dk
- Keisuke Yonehara, keisuke.yonehara@dandrite.au.dk

**Corresponding Author:**
Keisuke Yonehera, keisuke.yonehara@dandrite.au.dk
 
---
<p align="center">
    <img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/aarhusuniversity.svg?raw=true" align="center" height="40">&nbsp;&nbsp;&nbsp;&nbsp;
    <img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/dandrite.svg?raw=true" align="center" height="40">&nbsp;&nbsp;&nbsp;&nbsp;
    <img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/nordicembl.svg?raw=true" align="center" height="40">
</p>
<p align="center">
    <a href="http://www.yoneharalab.com">
    <img src="https://github.com/simonarvin/eyeloop/blob/master/misc/imgs/yoneharalab.svg?raw=true" align="center" height="18">&nbsp;&nbsp;&nbsp;&nbsp;
    </a>
    </p>