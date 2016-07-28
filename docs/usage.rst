Usage
=====

This package should nicely integrate with your existing python code,
thus makes it easy to participate in the `SISEC MUS
tasks <https://sisec.inria.fr/home/2016-professionally-produced-music-recordings>`__.
The core of this package is calling a user-provided function that
separates the mixtures from the DSD into several estimated target
sources.


Providing a compatible function
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The core of this package consists of calling a user-provided function
which separates the mixtures from the dsdtools into estimated target
sources.

-  The function will take an dsdtools ``Track`` object which can be used
   from inside your algorithm.
-  Participants can access
-  ``Track.audio``, representing the stereo mixture as an ``np.ndarray``
   of ``shape=(nun_sampl, 2)``
-  ``Track.rate``, the sample rate
-  ``Track.path``, the absolute path of the mixture which might be handy
   to process with external applications, so that participants don't
   need to write out temporary wav files.
-  The function needs to return a python ``Dict`` which consists of
   target name (``key``) and the estimated target as audio arrays with
   same shape as the mixture (``value``).
-  It is the users choice which target sources they want to provide for
   a given mixture. Supported targets are
   ``['vocals', 'accompaniment', 'drums', 'bass', 'other']``.
-  Please make sure that the returned estimates do have the same sample
   rate as the mixture track.

Here is an example for such a function separating the mixture into a
**vocals** and **accompaniment** track.

.. code:: python

    def my_function(track):

        # get the audio mixture as numpy array shape=(nun_sampl, 2)
        track.audio

        # compute voc_array, acc_array
        # ...

        return {
            'vocals': voc_array,
            'accompaniment': acc_array
        }

Create estimates for SiSEC evaluation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Setting up dsdtools
'''''''''''''''''''

Simply import the dsdtools package in your main python function:

.. code:: python

   import dsdtools

   dsd = dsdtools.DB(
       root_dir='path/to/dsdtools/',
   )

The ``root_dir`` is the path to the dsdtools dataset folder. Instead of
``root_dir`` it can also be set system-wide. Just
``export DSD_PATH=/path/to/dsdtools`` inside your terminal environment.

Test if your separation function generates valid output
'''''''''''''''''''''''''''''''''''''''''''''''''''''''

Before you run the full DSD100, which might take very long, participants
can test their separation function by running:

.. code:: python

   dsd.test(my_function)

This test makes sure the user provided output is compatible to the
dsdtools framework. The function returns ``True`` if the test succeeds.

Processing the full DSD100
''''''''''''''''''''''''''

To process all 100 DSD tracks and saves the results to the
``estimates_dir``:

.. code:: python

    dsd.run(my_function, estimates_dir="path/to/estimates")

Processing training and testing subsets separately
''''''''''''''''''''''''''''''''''''''''''''''''''

Algorithms which make use of machine learning techniques can use the
training subset and then apply the algorithm on the test data:

.. code:: python

    dsd.run(my_training_function, subsets="Dev")
    dsd.run(my_test_function, subsets="Test")

If you want to exclude tracks from the training you can specify track ids as
``dsdtools.DB(..., valid_ids=[1, 2]`` object. Those tracks are then not
included in ``Dev`` but are returned for ``subsets="Valid"``.


Processing single or multiple DSD100 tracks
'''''''''''''''''''''''''''''''''''''''''''

.. code:: python

    dsd.run(my_function, ids=30)
    dsd.run(my_function, ids=[1, 2, 3])
    dsd.run(my_function, ids=range(90, 99))

Note, that the provided list of ids can be overridden if the user sets a
terminal environment variable ``DSD_ID=1``.

Use multiple cores
''''''''''''''''''

Python Multiprocessing
""""""""""""""""""""""

To speed up the processing, ``run`` can make use of multiple CPUs:

.. code:: python

    dsd.run(my_function, parallel=True, cpus=4)

Note: We use the python builtin multiprocessing package, which sometimes
is unable to parallelize the user provided function to
`PicklingError <http://stackoverflow.com/a/8805244>`__.

GNU Parallel
""""""""""""

    `GNU parallel <http://www.gnu.org/software/parallel>`__ is a shell
    tool for executing jobs in parallel using one or more computers. A
    job can be a single command or a small script that has to be run for
    each of the lines in the input. The typical input is a list of
    files, a list of hosts, a list of users, a list of URLs, or a list
    of tables. A job can also be a command that reads from a pipe. GNU
    parallel can then split the input and pipe it into commands in
    parallel.

By running only one ``id`` in each python process the dsdtools set can
easily be processed with GNU parallel using multiple CPUs without any
further modifications to your code:

.. code:: bash

    parallel --bar 'DSD_ID={0} python main.py' ::: {1..100}


Compute the bss\_eval measures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The official SISEC evaluation relies on *MATLAB* because currently there
does not exist a
`bss\_eval <http://bass-db.gforge.inria.fr/bss_eval/>`__ implementation
for python which produces indentical results. Therefore please run
``dsd100_eval_only.m`` from the `DSD100 Matlab
scripts <https://github.com/faroit/dsd100mat>`__ after you have
processed and saved your estimates with *dsdtoolspy*.


Evaluation in python
^^^^^^^^^^^^^^^^^^^^

.. warning:: Evaluation in python is not supported yet
