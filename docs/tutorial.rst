.. _tutorial:

BirdVoxClassify tutorial
========================

Introduction
------------
Welcome to the BirdVoxClassify tutorial! In this tutorial, we'll show how to use BirdVoxClassify
to classify the species of bird flight calls in audio clips . The supported audio formats
are those supported by the `pysoundfile` library, which is used for loading the audio (e.g. WAV, OGG, FLAC).

.. _using_library:

Using the Library
-----------------

You can simply compute bird species predictions out of the box, like so:

.. code-block:: python

    import birdvoxclassify as bvc
    import json
    filepath = '/path/to/file.wav'
    filepath_list = [
        '/path/to/file1.wav',
        '/path/to/file2.wav',
        '/path/to/file3.wav'
    ]
    ## Get prediction dictionary object
    # Prediction for a single file
    formatted_pred = process_file(filepath)
    # Prediction for a list of files
    formatted_pred = process_file(filepath_list)

    ## Save individual output files for each audio file
    process_file(filepath_list, output_dir='/output/dir')
    # Add a suffix to output filenames (e.g. /path/to/file1_<suffix>.json)
    process_file(filepath_list, output_dir='/output/dir', suffix='suffix')

    # Save a summary output file
    process_file(filepath_list, output_summary_path='/path/to/output/file.json')

    # Specify model (and taxonomy) to use
    formatted_pred = process_file(filepath, model_name=bvc.DEFAULT_MODEL_NAME)

    # Pre-load model and taxonomy
    model = bvc.load_classifier(bvc.DEFAULT_MODEL_NAME)
    taxonomy_path = bvc.get_taxonomy_path(bvc.DEFAULT_MODEL_NAME)
    with open(taxonomy_path, 'r') as f:
        taxonomy = json.load(f)
    formatted_pred = process_file(filepath, classifier=model, taxonomy=taxonomy)

    # Change batch size depending on computational resources
    formatted_pred = process_file(filepath, batch_size=32)


You can also compute predictions directly on loaded audio arrays:

.. code-block:: python

    import birdvoxclassify as bvc
    import soundfile as sf
    import json

    # Load audio
    audio, sr = sf.read('/path/to/file.wav')
    pcen = bvc.compute_pcen(audio, sr, input_format=True)
    # Load model and taxonomy
    model = bvc.load_classifier(bvc.DEFAULT_MODEL_NAME)
    taxonomy_path = bvc.get_taxonomy_path(bvc.DEFAULT_MODEL_NAME)
    with open(taxonomy_path, 'r') as f:
        taxonomy = json.load(f)

    # Get list of one-hot prediction array for each level of the taxonomy
    pred_list = bvc.predict(pcen, model)
    coarse_pred, medium_pred, fine_pred = pred_list

    # Format prediction in more interpretable format
    formatted_pred = bvc.format_pred(pred_list, taxonomy)


Using the Command Line Interface (CLI)
--------------------------------------

To compute embeddings for a single file via the command line run:

.. code-block:: shell

    $ birdvoxclassify /path/to/file.wav

This will print out the model prediction in JSON format.

You can also provide multiple input files or directories:

.. code-block:: shell

    $ birdvoxclassify /path/to/file1.wav /path/to/file2.wav /path/to/file3.wav

You can set the output directory for per-file output files as follows:

.. code-block:: shell

    $ birdvoxclassify /path/to/file1.wav /path/to/file2.wav /path/to/file3.wav --output-dir /output/dir

This will create an output files ``/output/dir/file1.json``, ``/output/dir/file2.json``, and ``/output/dir/file3.json``.

You can create a single summary output file as follows:

.. code-block:: shell

    $ birdvoxclassify /path/to/file1.wav /path/to/file2.wav /path/to/file3.wav --output-summary-path /output/summary/path.json

which will create a summary output file at ``/output/summary/path.json``.

You can specify the classifier model name as follows:

.. code-block:: shell

    $ birdvoxclassify /path/to/file.wav --classifier-name birdvoxclassify-flat-multitask-convnet_tv1hierarchical-2e7e1bbd434a35b3961e315cfe3832fc

If processing a large number of files, you can set the prediction batch size appropriately for your computational
resources as follows:

.. code-block:: shell

    $ birdvoxclassify /large/audio/dir --batch-size 128

You can append a suffix to the output files as follows:

.. code-block:: shell

    $ birdvoxclassify /path/to/file1.wav /path/to/file2.wav /path/to/file3.wav --output-dir /output/dir --suffix suffix

This will create an output files ``/output/dir/file1_suffix.json``, ``/output/dir/file2_suffix.json``, and ``/output/dir/file3_suffix.json``.

You can print verbose outputs by running:

.. code-block:: shell

    $ birdvoxclassify /path/to/file.wav --verbose

Finally, you can suppress non-error printouts by running:

.. code-block:: shell

    $ birdvoxclassify /path/to/file.wav --quiet
