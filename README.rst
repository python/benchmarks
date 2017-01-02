##########################
The Python Benchmark Suite
##########################

.. image:: https://img.shields.io/pypi/v/performance.svg
   :alt: Latest release on the Python Cheeseshop (PyPI)
   :target: https://pypi.python.org/pypi/performance

.. image:: https://travis-ci.org/python/performance.svg?branch=master
   :alt: Build status of perf on Travis CI
   :target: https://travis-ci.org/python/performance

The ``performance`` project is intended to be an authoritative source of
benchmarks for all Python implementations. The focus is on real-world
benchmarks, rather than synthetic benchmarks, using whole applications when
possible.

* GitHub: https://github.com/python/performance (source code, issues)
* PyPI: https://pypi.python.org/pypi/performance

Other Python Benchmarks:

* CPython: `speed.python.org <https://speed.python.org/>`_ uses perf,
  performance and `Codespeed <https://github.com/tobami/codespeed/>`_ (Django
  web application)
* PyPy: `speed.pypy.org <http://speed.pypy.org/>`_
  uses `PyPy benchmarks <https://bitbucket.org/pypy/benchmarks>`_
* Pyston: `pyston-perf <https://github.com/dropbox/pyston-perf>`_
  and `speed.pyston.org <http://speed.pyston.org/>`_
* `Numba benchmarks <http://numba.pydata.org/numba-benchmark/>`_
* Cython: `Cython Demos/benchmarks
  <https://github.com/cython/cython/tree/master/Demos/benchmarks>`_
* pythran: `numpy-benchmarks
  <https://github.com/serge-sans-paille/numpy-benchmarks>`_

See also the `Python speed mailing list
<https://mail.python.org/mailman/listinfo/speed>`_ and the `Python perf module
<http://perf.readthedocs.io/>`_ (used by performance).


Installation
============

Command to install performance::

    python3 -m pip install performance

On Python 2, the ``virtualenv`` program (or the Python module) is required
to create virtual environments. On Python 3, the ``venv`` module of the
standard library is used.

At runtime, Python development files (header files) may be needed to install
some dependencies like ``dulwich_log`` or ``psutil``, to build their C
extension. Commands on Fedora to install dependencies:

* Python 2: ``sudo dnf install python-devel``
* Python 3: ``sudo dnf install python3-devel``
* PyPy: ``sudo dnf install pypy-devel``


Run benchmarks
==============

Commands to compare Python 2 and Python 3 performances::

    pyperformance run --python=python2 -o py2.json
    pyperformance run --python=python3 -o py3.json
    pyperformance compare py2.json py3.json

Note: ``python3 -m performance ...`` syntax works as well (ex: ``python3 -m
performance run -o py3.json``), but requires to install performance on each
tested Python version.

Actions::

    run                 Run benchmarks on the running python
    show                Display a benchmark file
    compare             Compare two benchmark files
    list                List benchmarks which run command would run
    list_groups         List all benchmark groups
    venv                Actions on the virtual environment

Common options
--------------

Options available to all commands::

  -p PYTHON, --python PYTHON
                        Python executable (default: use running Python)
  --venv VENV           Path to the virtual environment
  --inherit-environ VAR_LIST
                        Comma-separated list of environment variable names
                        that are inherited from the parent environment when
                        running benchmarking subprocesses.

run
---

Options of the ``run`` command::

  -r, --rigorous        Spend longer running tests to get more accurate
                        results
  -f, --fast            Get rough answers quickly
  -m, --track-memory    Track memory usage. This only works on Linux.
  -b BM_LIST, --benchmarks BM_LIST
                        Comma-separated list of benchmarks to run. Can contain
                        both positive and negative arguments:
                        --benchmarks=run_this,also_this,-not_this. If there
                        are no positive arguments, we'll run all benchmarks
                        except the negative arguments. Otherwise we run only
                        the positive arguments.
  --affinity CPU_LIST   Specify CPU affinity for benchmark runs. This way,
                        benchmarks can be forced to run on a given CPU to
                        minimize run to run variation. This uses the taskset
                        command.
  -o FILENAME, --output FILENAME
                        Run the benchmarks on only one interpreter and write
                        benchmark into FILENAME. Provide only baseline_python,
                        not changed_python.
  --append FILENAME     Add runs to an existing file, or create it if it
                        doesn't exist

show
----

Usage::

    show FILENAME


compare
-------

Options of the ``compare`` command::

  -v, --verbose         Print more output
  -O STYLE, --output_style STYLE
                        What style the benchmark output should take. Valid
                        options are 'normal' and 'table'. Default is normal.

list
----

Options of the ``list`` command::

  -b BM_LIST, --benchmarks BM_LIST
                        Comma-separated list of benchmarks to run. Can contain
                        both positive and negative arguments:
                        --benchmarks=run_this,also_this,-not_this. If there
                        are no positive arguments, we'll run all benchmarks
                        except the negative arguments. Otherwise we run only
                        the positive arguments.

Use ``python3 -m performance list -b all`` to list all benchmarks.


venv
----

Options of the ``venv`` command::

  -p PYTHON, --python PYTHON
                        Python executable (default: use running Python)
  --venv VENV           Path to the virtual environment

Actions of the ``venv`` command::

  show      Display the path to the virtual environment and it's status (created or not)
  create    Create the virtual environment
  recreate  Force the recreation of the the virtual environment
  remove    Remove the virtual environment


How to get stable benchmarks
============================

Advices helping to get make stable benchmarks:

* Run ``python3 -m perf system tune`` command
* See also advices in the perf documentation: `Stable and reliable benchmarks
  <http://perf.readthedocs.io/en/latest/perf.html#stable-and-reliable-benchmarks>`_
* Compile Python using LTO (Link Time Optimization) and PGO (profile guided optimizations)::

    ./configure --with-lto
    make profile-opt

  You should get the ``-flto`` option on GCC for example.

* Use the ``--rigorous`` option of the ``run`` command

Notes:

* Development versions of Python 2.7, 3.6 and 3.7 have a --enable-optimizations
  configure option
* --enable-optimizations doesn't enable LTO because of compiler bugs:
  http://bugs.python.org/issue28032
  (see also: http://bugs.python.org/issue28605)
* PGO is broken on Ubuntu 14.04 LTS with GCC 4.8.4-2ubuntu1~14.04:
  ``Modules/socketmodule.c:7743:1: internal compiler error: in edge_badness,
  at ipa-inline.c:895``
* If nohz_full kernel option is used, the CPU frequency must be fixed,
  otherwise the CPU frequency will be instable. See `Bug 1378529: intel_pstate
  driver doesn't support NOHZ_FULL
  <https://bugzilla.redhat.com/show_bug.cgi?id=1378529>`_.
* ASLR must *not* be disabled manually! (it's enabled by default on Linux)


Notes
=====

Tool for comparing the performance of two Python implementations.

pyperformance will run Student's two-tailed T test on the benchmark results at the 95%
confidence level to indicate whether the observed difference is statistically
significant.

Omitting the ``-b`` option will result in the default group of benchmarks being
run Omitting ``-b`` is the same as specifying `-b default`.

To run every benchmark pyperformance knows about, use ``-b all``. To see a full
list of all available benchmarks, use `--help`.

Negative benchmarks specifications are also supported: `-b -2to3` will run every
benchmark in the default group except for 2to3 (this is the same as
`-b default,-2to3`). `-b all,-django` will run all benchmarks except the Django
templates benchmark. Negative groups (e.g., `-b -default`) are not supported.
Positive benchmarks are parsed before the negative benchmarks are subtracted.

If ``--track_memory`` is passed, pyperformance will continuously sample the
benchmark's memory usage. This currently only works on Linux 2.6.16 and higher
or Windows with PyWin32. Because ``--track_memory`` introduces performance
jitter while collecting memory measurements, only memory usage is reported in
the final report.


Benchmarks
==========

Available Groups
----------------

Like individual benchmarks (see "Available benchmarks" below), benchmarks group
are allowed after the `-b` option. Use ``python3 -m performance list_groups``
to list groups and their benchmarks.

Available benchmark groups:

* ``2n3``: Benchmarks compatible with both Python 2 and Python 3
* ``all``: Group including all benchmarks
* ``apps``: "High-level" applicative benchmarks (2to3, Chameleon, Tornado HTTP)
* ``calls``: Microbenchmarks on function and method calls
* ``default``: Group of benchmarks run by default by the ``run`` command
* ``etree``: XML ElementTree
* ``math``: Float and integers
* ``regex``: Collection of regular expression benchmarks
* ``serialize``: Benchmarks on ``pickle`` and ``json`` modules
* ``startup``: Collection of microbenchmarks focused on Python interpreter
  start-up time.
* ``template``: Templating libraries

There is also a disabled ``threading`` group: collection of microbenchmarks for
Python's threading support. These benchmarks come in pairs: an iterative
version (iterative_foo), and a multithreaded version (threaded_foo).


Available Benchmarks
--------------------

- ``2to3`` - have the 2to3 tool translate itself.
- ``call_method`` - positional arguments-only method calls.
- ``call_method_slots`` - method calls on classes that use __slots__.
- ``call_method_unknown`` - method calls where the receiver cannot be predicted.
- ``call_simple`` - positional arguments-only function calls.
- ``chameleon`` - render a template using the ``chameleon`` module
- ``chaos`` - create chaosgame-like fractals
- ``crypto_pyaes`` - benchmark a pure-Python implementation of the AES
  block-cipher in CTR mode using the pyaes module.
- ``deltablue`` - DeltaBlue benchmark
- ``django_template`` - use the Django template system to build a 150x150-cell
  HTML table (``django.template`` module).
- ``dulwich_log``: Iterate on commits of the asyncio Git repository using
  the Dulwich module
- ``fannkuch``
- ``fastpickle`` - use the cPickle module to pickle a variety of datasets.
- ``fastunpickle`` - use the cPickle module to unnpickle a variety of datasets.
- ``float`` - artificial, floating point-heavy benchmark originally used
  by Factor.
- ``genshi``: Benchmark the ``genshi.template`` module

  * ``genshi_text``: Render template to plain text
  * ``genshi_xml``: Render template to XML

- ``go``: Go board game
- ``hexiom`` - Solver of Hexiom board game (level 25 by default)
- ``hg_startup`` - Get Mercurial's help screen.
- ``html5lib`` - parse the HTML 5 spec using html5lib.
- ``json_dumps`` - Benchmark ``json.dumps()``
- ``json_loads`` - Benchmark ``json.loads()``
- ``logging`` - Benchmarks on the ``logging`` module

  * ``logging_format``: Benchmark ``logger.warn(fmt, str)``
  * ``logging_simple``: Benchmark ``logger.warn(msg)``
  * ``logging_silent``: Benchmark ``logger.warn(msg)`` when the message is
    ignored

- ``mako`` - use the Mako template system to build a 150x150-cell HTML table.
- ``mdp`` - battle with damages and topological sorting of nodes in a graph
- ``meteor_contest`` - solver for Meteor Puzzle board
- ``nbody`` - the N-body Shootout benchmark. Microbenchmark for floating point
  operations.
- ``normal_startup`` - Measure the Python startup time
- ``nqueens`` - Simple, brute-force N-Queens solver
- ``pathlib`` - Test the performance of operations of the ``pathlib`` module.
  This benchmark stresses the creation of small objects, globbing, and system
  calls.
- ``pickle_dict`` - microbenchmark; use the cPickle module to pickle a lot of dicts.
- ``pickle_list`` - microbenchmark; use the cPickle module to pickle a lot of lists.
- ``pickle_pure_python`` - use the pure-Python pickle module to pickle a
  variety of datasets.
- ``pidigits`` - Calculating 2,000 digits of π.  This benchmark stresses
  big integer arithmetic.
- ``pybench`` - run the standard Python PyBench benchmark suite. This is
  considered an unreliable, unrepresentative benchmark; do not base decisions
  off it. It is included only for completeness.
- ``pyflate`` - Pyflate benchmark: tar/bzip2 decompressor in pure Python
- ``raytrace`` - Simple raytracer.
- ``regex_compile`` - stress the performance of Python's regex compiler,
  rather than the regex execution speed.
- ``regex_dna`` - regex DNA benchmark using "fasta" to generate the test case
- ``regex_effbot`` - some of the original benchmarks used to tune mainline
  Python's current regex engine.
- ``regex_v8`` - Python port of V8's regex benchmark.
- ``richards`` - the classic Richards benchmark.
- ``scimark``:

  * ``scimark_sor`` - scimark: `Successive over-relaxation (SOR)
    <https://en.wikipedia.org/wiki/Successive_over-relaxation>`_ benchmark
  * ``scimark_sparse_mat_mult`` - scimark: `sparse matrix
    <https://en.wikipedia.org/wiki/Sparse_matrix>`_ `multiplication
    <https://en.wikipedia.org/wiki/Matrix_multiplication_algorithm>`_ benchmark
  * ``scimark_monte_carlo`` - scimark: benchmark on the `Monte Carlo algorithm
    <https://en.wikipedia.org/wiki/Monte_Carlo_algorithm>`_ to compute the area
    of a disc
  * ``scimark_lu`` - scimark: `LU decomposition
    <https://en.wikipedia.org/wiki/LU_decomposition>`_ benchmark
  * ``scimark_fft`` - scimark: `Fast Fourier transform (FFT)
    <https://en.wikipedia.org/wiki/Fast_Fourier_transform>`_ benchmark

- ``spambayes`` - run a canned mailbox through a SpamBayes ham/spam classifier.
- ``spectral_norm`` - MathWorld: "Hundred-Dollar, Hundred-Digit Challenge
  Problems", Challenge #3.
- ``sqlalchemy_declarative`` - SQLAlchemy Declarative benchmark using SQLite
- ``sqlalchemy_imperative`` - SQLAlchemy Imperative benchmark using SQLite
- ``sqlite_synth`` - Benchmark Python aggregate for SQLite
- ``startup_nosite`` - Measure the Python startup time without importing
  the ``site`` module (``python -S``)
- ``sympy`` - Benchmark on the ``sympy`` module

  * ``sympy_expand``: Benchmark ``sympy.expand()``
  * ``sympy_integrate``: Benchmark ``sympy.integrate()``
  * ``sympy_str``: Benchmark ``str(sympy.expand())``
  * ``sympy_sum``: Benchmark ``sympy.summation()``

- ``telco`` - Benchmark the ``decimal`` module
- ``tornado_http`` - Benchmark HTTP server of the ``tornado`` module
- ``unpack_sequence`` - microbenchmark for unpacking lists and tuples.
- ``unpickle_list``
- ``unpickle_pure_python`` - use the pure-Python pickle module to unpickle a
  variety of datasets.
- ``xml_etree``: Benchmark the ``xml.etree`` module

  - ``xml_etree_generate``: Create an XML document
  - ``xml_etree_iterparse``: Benchmark ``etree.iterparse()``
  - ``xml_etree_parse``: Benchmark ``etree.parse()``
  - ``xml_etree_process``: Process an XML document

There are also two disabled benchmarks:

- ``threading_threaded_count`` - spin in a while loop, counting down
  from a large number in a thread.
- ``threading_iterative_count`` - spin in a while loop, counting down
  from a large number.


Changelog
=========

Version 0.5.0 (2016-11-16)
--------------------------

* Add ``mdp`` benchmark: battle with damages and topological sorting of nodes
  in a graph
* The ``default`` benchmark group now include all benchmarks but ``pybench``
* If a benchmark fails, log an error, continue to execute following
  benchmarks, but exit with error code 1.
* Remove deprecated benchmarks: ``threading_threaded_count`` and
  ``threading_iterative_count``. It wasn't possible to run them anyway.
* ``dulwich`` requirement is now optional since its installation fails
  on Windows.
* Upgrade requirements:

  - Mako: 1.0.5 => 1.0.6
  - Mercurial: 3.9.2 => 4.0.0
  - SQLAlchemy: 1.1.3 => 1.1.4
  - backports-abc: 0.4 => 0.5

Version 0.4.0 (2016-11-07)
--------------------------

* Add ``sqlalchemy_imperative`` benchmark: it wasn't registered properly
* The ``list`` command now only lists the benchmark that the ``run`` command
  will run. The ``list`` command gets a new ``-b/--benchmarks`` option.
* Rewrite the code creating the virtual environment to test correctly pip.
  Download and run ``get-pip.py`` if pip installation failed.
* Upgrade requirements:

  * perf: 0.8.2 => 0.9.0
  * Django: 1.10.2 => 1.10.3
  * Mako: 1.0.4 => 1.0.5
  * psutil: 4.3.1 => 5.0.0
  * SQLAlchemy: 1.1.2 => 1.1.3

* Remove ``virtualenv`` dependency

Version 0.3.2 (2016-10-19)
--------------------------

* Fix setup.py: include also ``performance/benchmarks/data/asyncio.git/``

Version 0.3.1 (2016-10-19)
--------------------------

* Add ``regex_dna`` benchmark
* The ``run`` command now fails with an error if no benchmark was run.
* genshi, logging, scimark, sympy and xml_etree scripts now run all
  sub-benchmarks by default
* Rewrite pybench using perf: remove the old legacy code to calibrate and run
  benchmarks, reuse perf.Runner API.
* Change heuristic to create the virtual environment, tried commands:

  * ``python -m venv``
  * ``python -m virtualenv``
  * ``virtualenv -p python``

* The creation of the virtual environment now ensures that pip works
  to detect "python3 -m venv" which doesn't install pip.
* Upgrade perf dependency from 0.7.12 to 0.8.2: update all benchmarks to
  the new perf 0.8 API (which introduces incompatible changes)
* Update SQLAlchemy from 1.1.1 to 1.1.2

Version 0.3.0 (2016-10-11)
--------------------------

New benchmarks:

* Add ``crypto_pyaes``: Benchmark a pure-Python implementation of the AES
  block-cipher in CTR mode using the pyaes module (version 1.6.0). Add
  ``pyaes`` dependency.
* Add ``sympy``: Benchmark on SymPy. Add ``scipy`` dependency.
* Add ``scimark`` benchmark
* Add ``deltablue``: DeltaBlue benchmark
* Add ``dulwich_log``: Iterate on commits of the asyncio Git repository using
  the Dulwich module. Add ``dulwich`` (and ``mpmath``) dependencies.
* Add ``pyflate``: Pyflate benchmark, tar/bzip2 decompressor in pure
  Python
* Add ``sqlite_synth`` benchmark: Benchmark Python aggregate for SQLite
* Add ``genshi`` benchmark: Render template to XML or plain text using the
  Genshi module. Add ``Genshi`` dependency.
* Add ``sqlalchemy_declarative`` and ``sqlalchemy_imperative`` benchmarks:
  SQLAlchemy Declarative and Imperative benchmarks using SQLite. Add
  ``SQLAlchemy`` dependency.

Enhancements:

* ``compare`` command now fails if the performance versions are different
* ``nbody``: add ``--reference`` and ``--iterations`` command line options.
* ``chaos``: add ``--width``, ``--height``, ``--thickness``, ``--filename``
  and ``--rng-seed`` command line options
* ``django_template``: add ``--table-size`` command line option
* ``json_dumps``: add ``--cases`` command line option
* ``pidigits``: add ``--digits`` command line option
* ``raytrace``: add ``--width``, ``--height`` and ``--filename`` command line
  options
* Port ``html5lib`` benchmark to Python 3
* Enable ``pickle_pure_python`` and ``unpickle_pure_python`` on Python 3
  (code was already compatible with Python 3)
* Creating the virtual environment doesn't inherit environment variables
  (especially ``PYTHONPATH``) by default anymore: ``--inherit-environ``
  command line option must now be used explicitly.

Bugfixes:

* ``chaos`` benchmark now also reset the ``random`` module at each sample
  to get more reproductible benchmark results
* Logging benchmarks now truncate the in-memory stream before each benchmark
  run

Rename benchmarks:

* Rename benchmarks to get a consistent name between the command line and
  benchmark name in the JSON file.
* Rename pickle benchmarks:

   - ``slowpickle`` becomes ``pickle_pure_python``
   - ``slowunpickle`` becomes ``unpickle_pure_python``
   - ``fastpickle`` becomes ``pickle``
   - ``fastunpickle`` becomes ``unpickle``

 * Rename ElementTree benchmarks: replace ``etree_`` prefix with
   ``xml_etree_``.
 * Rename ``hexiom2`` to ``hexiom_level25`` and explicitly pass ``--level=25``
   parameter
 * Rename ``json_load`` to ``json_loads``
 * Rename ``json_dump_v2`` to ``json_dumps`` (and remove the deprecated
   ``json_dump`` benchmark)
 * Rename ``normal_startup`` to ``python_startup``, and ``startup_nosite``
   to ``python_startup_no_site``
 * Rename ``threaded_count`` to ``threading_threaded_count``,
   rename ``iterative_count`` to ``threading_iterative_count``
 * Rename logging benchmarks:

   - ``silent_logging`` to ``logging_silent``
   - ``simple_logging`` to ``logging_simple``
   - ``formatted_logging`` to ``logging_format``

Minor changes:

* Update dependencies
* Remove broken ``--args`` command line option.


Version 0.2.2 (2016-09-19)
--------------------------

* Add a new ``show`` command to display a benchmark file
* Issue #11: Display Python version in compare. Display also the performance
  version.
* CPython issue #26383; csv output: don't truncate digits for timings shorter
  than 1 us
* compare: Use sample unit of benchmarks, format values in the table
  output using the unit
* compare: Fix the table output if benchmarks only contain a single sample
* Remove unused -C/--control_label and -E/--experiment_label options
* Update perf dependency to 0.7.11 to get Benchmark.get_unit() and
  BenchmarkSuite.get_metadata()

Version 0.2.1 (2016-09-10)
--------------------------

* Add ``--csv`` option to the ``compare`` command
* Fix ``compare -O table`` output format
* Freeze indirect dependencies in requirements.txt
* ``run``: add ``--track-memory`` option to track the memory peak usage
* Update perf dependency to 0.7.8 to support memory tracking and the new
  ``--inherit-environ`` command line option
* If ``virtualenv`` command fail, try another command to create the virtual
  environment: catch ``virtualenv`` error
* The first command to upgrade pip to version ``>= 6.0`` now uses the ``pip``
  binary rather than ``python -m pip`` to support pip 1.0 which doesn't support
  ``python -m pip`` CLI.
* Update Django (1.10.1), Mercurial (3.9.1) and psutil (4.3.1)
* Rename ``--inherit_env`` command line option to ``--inherit-environ`` and fix
  it

Version 0.2 (2016-09-01)
------------------------

* Update Django dependency to 1.10
* Update Chameleon dependency to 2.24
* Add the ``--venv`` command line option
* Convert Python startup, Mercurial startup and 2to3 benchmarks to perf scripts
  (bm_startup.py, bm_hg_startup.py and bm_2to3.py)
* Pass the ``--affinity`` option to perf scripts rather than using the
  ``taskset`` command
* Put more installer and optional requirements into
  ``performance/requirements.txt``
* Cached ``.pyc`` files are no more removed before running a benchmark.
  Use ``venv recreate`` command to update a virtual environment if required.
* The broken ``--track_memory`` option has been removed. It will be added back
  when it will be fixed.
* Add performance version to metadata
* Upgrade perf dependency to 0.7.5 to get ``Benchmark.update_metadata()``

Version 0.1.2 (2016-08-27)
--------------------------

* Windows is now supported
* Add a new ``venv`` command to show, create, recrete or remove the virtual
  environment.
* Fix pybench benchmark (update to perf 0.7.4 API)
* performance now tries to install the ``psutil`` module on CPython for better
  system metrics in metadata and CPU pinning on Python 2.
* The creation of the virtual environment now also tries ``virtualenv`` and
  ``venv`` Python modules, not only the virtualenv command.
* The development version of performance now installs performance
  with "pip install -e <path_to_performance>"
* The GitHub project was renamed from ``python/benchmarks``
  to ``python/performance``.

Version 0.1.1 (2016-08-24)
--------------------------

* Fix the creation of the virtual environment
* Rename pybenchmarks script to pyperformance
* Add -p/--python command line option
* Add __main__ module to be able to run: python3 -m performance

Version 0.1 (2016-08-24)
------------------------

* First release after the conversion to the perf module and move to GitHub
* Removed benchmarks

  - django_v2, django_v3
  - rietveld
  - spitfire (and psyco): Spitfire is not available on PyPI
  - pystone
  - gcbench
  - tuple_gc_hell


History
-------

Projected moved to https://github.com/python/performance in August 2016. Files
reorganized, benchmarks patched to use the perf module to run benchmark in
multiple processes.

Project started in December 2008 by Collin Winter and Jeffrey Yasskin for the
Unladen Swallow project. The project was hosted at
https://hg.python.org/benchmarks until Feb 2016
