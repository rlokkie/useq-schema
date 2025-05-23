# useq-schema

[![License](https://img.shields.io/pypi/l/useq-schema.svg?color=green)](https://github.com/pymmcore-plus/useq-schema/raw/main/LICENSE)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/useq-schema)](https://pypi.org/project/useq-schema)
[![PyPI](https://img.shields.io/pypi/v/useq-schema.svg?color=green)](https://pypi.org/project/useq-schema)
[![Conda](https://img.shields.io/conda/vn/conda-forge/useq-schema)](https://anaconda.org/conda-forge/useq-schema)
[![tests](https://github.com/pymmcore-plus/useq-schema/actions/workflows/ci.yml/badge.svg)](https://github.com/pymmcore-plus/useq-schema/actions/workflows/ci.yml)
[![docs](https://github.com/pymmcore-plus/useq-schema/actions/workflows/docs.yml/badge.svg)](https://pymmcore-plus.github.io/useq-schema/)
[![codecov](https://codecov.io/gh/pymmcore-plus/useq-schema/branch/main/graph/badge.svg)](https://codecov.io/gh/pymmcore-plus/useq-schema)

*An open, implementation-agnostic schema for describing multi-dimensional
microscopy experiments.*

**Documentation: <https://pymmcore-plus.github.io/useq-schema/>**

## Rationale

The `useq-schema` library defines a structured schema to represent a sequence of
microscope acquisition events. By adopting this schema, various microscopy
software tools can facilitate interoperability, allowing end users to
potentially switch between different control backends with ease. *The goal is to
encourage a shared standard, making it straightforward for developers to adopt
useq-schema and enhance compatibility across tools.*

> [!IMPORTANT]
>
> **Hey developers! :wave: Not convinced?  Don't leave yet!**  
>
> We are particularly interested in feedback from developers of microscopy-control
> software.
> 
> If you are considering supporting `useq-schema` in your software, but don't
> see all the fields in `MDAEvent` that you would need to support your complex use case,
> please [open an issue](https://github.com/pymmcore-plus/useq-schema/issues/new) or pull
> request to discuss additional features.
>
> :carrot: The carrot for you?
> 
> Anyone who is already using `useq-schema` to describe a sequence of events
> in some other software (that supports it) can easily try out your solution, with
> (hopefully) minimal changes to their code.

## `useq.MDAEvent`

The primary "event" object is `useq.MDAEvent`.  This represents a single event
that a microscope should perform, including preparation of the hardware, and
execution of the event (such as an image acquisition).

```python
from useq import MDAEvent

event = MDAEvent(
    channel="DAPI",
    exposure=100,
    x_pos=100.0,
    y_pos=100.0,
    z_pos=30.0,
    min_start_time=10.0,
    ... # multiple other fields
)
```

Downstream libraries that aim to support useq-schema should support driving
hardware based on an `Iterable[MDAEvent]`. See [`useq.MDAEvent`
documentation](https://pymmcore-plus.github.io/useq-schema/schema/event/) for
more details.

<details>

<summary>Similar objects in existing software packages</summary>

- For [micro-manager](https://github.com/micro-manager/micro-manager), this
  object is most similar (though not *that* similar) to the events generated by
  [`generate-acq-sequence`](https://github.com/micro-manager/micro-manager/blob/2b0f51a2f916112d39c6135ad35a112065f8d58d/acqEngine/src/main/clj/org/micromanager/sequence_generator.clj#L410)
  in the clojure acquisition engine.
- For [pycro-manager](https://github.com/micro-manager/pycro-manager), this
  object is similar to an individual [acquisition event
  `dict`](https://pycro-manager.readthedocs.io/en/latest/apis.html#acquisition-event-specification)
  generated by
  [`multi_d_acquisition_events`](https://github.com/micro-manager/pycro-manager/blob/63cf209a8907fd23932ee9f8016cb6a2b61b45aa/pycromanager/acquire.py#L605),
  (and, `useq` provides a `to_pycromanager()` method that converts an `MDAEvent` into a
  single pycro-manager event dict)
- *your object here?...*

</details>

## `useq.MDASequence`

`useq.MDASequence` is a declarative representation of an multi-dimensional
experiment.  It represents a sequence of events: as might be generated by the
multidimensional acquisition GUI in most microscope software.  It is composed of
["plans" for each axis in the
experiment](https://pymmcore-plus.github.io/useq-schema/schema/axes/) (such as a
Time Plan, a Z Plan, a list of channels and positions, etc.).  A
`useq.MDASequence` object is itself iterable, and yields `MDAEvent` objects.

See [`useq.MDASequence` documentation](https://pymmcore-plus.github.io/useq-schema/schema/sequence/)
for more details.

<details>

<summary>Similar objects in existing software packages</summary>

- For [micro-manager](https://github.com/micro-manager/micro-manager), this
  object is most similar to
  [`org.micromanager.acquisition.SequenceSettings`](https://github.com/micro-manager/micro-manager/blob/2b0f51a2f916112d39c6135ad35a112065f8d58d/mmstudio/src/main/java/org/micromanager/acquisition/SequenceSettings.java#L39),
  (generated by clicking the "Acquire!" button in the Multi-D Acquisition GUI)
- For [pycro-manager](https://github.com/micro-manager/pycro-manager), this
  object is similar to the
  [`multi_d_acquisition_events`](https://github.com/micro-manager/pycro-manager/blob/63cf209a8907fd23932ee9f8016cb6a2b61b45aa/pycromanager/acquire.py#L605)
  convenience function, (and `useq` provides a `to_pycromanager()`method that
  converts an `MDASequence` to a list of pycro-manager events)
- *your object here?...*

</details>

### Example usage

```python
from useq import MDASequence

mda_seq = MDASequence(
    stage_positions=[(100, 100, 30), (200, 150, 35)],
    channels=["DAPI", "FITC"],
    time_plan={'interval': 1, 'loops': 20},
    z_plan={"range": 4, "step": 0.5},
    axis_order='tpcz',
)
```

The `MDASequence` object is iterable, yielding `MDAEvent` objects in the order
specified by the `axis_order` attribute.

```python
>>> events = list(mda_seq)

>>> print(len(events))
720 

>>> print(events[:3])
[MDAEvent(
    channel=Channel(config='DAPI'),
    index=mappingproxy({'t': 0, 'p': 0, 'c': 0, 'z': 0}),
    min_start_time=0.0,
    x_pos=100.0,
    y_pos=100.0,
    z_pos=28.0,
 ),
 MDAEvent(
    channel=Channel(config='DAPI'),
    index=mappingproxy({'t': 0, 'p': 0, 'c': 0, 'z': 1}),
    min_start_time=0.0,
    x_pos=100.0,
    y_pos=100.0,
    z_pos=28.5,
 ),
 MDAEvent(
    channel=Channel(config='DAPI'),
    index=mappingproxy({'t': 0, 'p': 0, 'c': 0, 'z': 2}),
    min_start_time=0.0,
    x_pos=100.0,
    y_pos=100.0,
    z_pos=29.0,
 )]
 ```

Both `MDAEvent` and `MDASequence` objects are pydantic models, so they can be
easily serialized to and from json or yaml.

```py
print(mda_seq.yaml())
```

```yaml
axis_order: tpcz
channels:
- config: DAPI
- config: FITC
stage_positions:
- x: 100.0
  y: 100.0
  z: 30.0
- x: 200.0
  y: 150.0
  z: 35.0
time_plan:
  interval: 0:00:01
  loops: 20
z_plan:
  range: 4.0
  step: 0.5
```

## Installation

```bash
pip install useq-schema
```

or, with conda:

```bash
conda install -c conda-forge useq-schema
```

## Executing useq-schema experiments with pymmcore-plus

[pymmcore-plus](https://github.com/pymmcore-plus/pymmcore-plus) implements an
acquisition engine that can execute an iterable of `MDAEvents` using
micro-manager in a pure python environment (no Java required).

```python
from pymmcore_plus import CMMCorePlus

core = CMMCorePlus()
core.loadSystemConfiguration()  # loads demo by default

core.mda.run(mda_seq)  # run the experiment

# or, construct a sequence of MDAEvents anyway you like
events = [MDAEvent(...), MDAEvent(...), ...]
core.mda.run(events)
```

This can be considered a "reference implementation" of an engine that supports useq-schema. 

See [pymmcore-plus documentation](https://pymmcore-plus.github.io/pymmcore-plus/examples/mda/) for details.
