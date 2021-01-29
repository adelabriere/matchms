.. matchms documentation master file, created by
   sphinx-quickstart on Tue Apr  7 09:16:44 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to matchms's documentation!
===================================

Matchms is an open-access Python package to import, process, clean, and compare mass spectrometry data (MS/MS). It allows to implement and run an easy-to-follow, easy-to-reproduce workflow from raw mass spectra to pre- and post-processed spectral data. 

.. toctree::
   :maxdepth: 3
   :caption: Contents:

   API <api/matchms.rst>

Introduction
============

Matchms was designed to easily build custom spectra processing pipelines and to compute spectra similarities (see flowchart). Spectral data can be imported from common formats such mzML, mzXML, msp, metabolomics-USI, MGF, or json (e.g. GNPS-syle json files). Matchms then provides filters for metadata cleaning and checking, as well as for basic peak filtering. Finally, matchms was build to import and apply different similarity measures to compare large amounts of spectra. This includes common Cosine scores, but can also easily be extended by custom measures.

.. image:: _static/flowchart_matchms.png
  :width: 800
  :alt: matchms workflow illustration

Installation
============

Prerequisites:  

- Python 3.7 or 3.8
- Anaconda

Install matchms from Anaconda Cloud with

.. code-block:: console

  # install matchms in a new virtual environment to avoid dependency clashes
  conda create --name matchms python=3.7
  conda activate matchms
  conda install --channel nlesc --channel bioconda --channel conda-forge matchms

Example
=======

Below is a small example of using matchms to calculate the Cosine score between mass Spectrums in the `tests/pesticides.mgf <https://github.com/matchms/matchms/blob/master/tests/pesticides.mgf>`_ file.

.. testcode::

    from matchms.importing import load_from_mgf
    from matchms.filtering import default_filters
    from matchms.filtering import normalize_intensities
    from matchms import calculate_scores
    from matchms.similarity import CosineGreedy

    # Read spectrums from a MGF formatted file, for other formats see https://matchms.readthedocs.io/en/latest/api/matchms.importing.html 
    file = load_from_mgf("../tests/pesticides.mgf")

    # Apply filters to clean and enhance each spectrum
    spectrums = []
    for spectrum in file:
        # Apply default filter to standardize ion mode, correct charge and more.
        # Default filter is fully explained at https://matchms.readthedocs.io/en/latest/api/matchms.filtering.html .
        spectrum = default_filters(spectrum)
        # Scale peak intensities to maximum of 1
        spectrum = normalize_intensities(spectrum)
        spectrums.append(spectrum)

    # Calculate Cosine similarity scores between all spectrums
    # For other similarity score methods see https://matchms.readthedocs.io/en/latest/api/matchms.similarity.html .
    scores = calculate_scores(references=spectrums,
                              queries=spectrums,
                              similarity_function=CosineGreedy())

    # Print the calculated scores for each spectrum pair
    for score in scores:
        (reference, query, score) = score
        # Ignore scores between same spectrum and
        # pairs which have less than 20 peaks in common
        if reference is not query and score['matches'] >= 20:
            print(f"Reference scan id: {reference.metadata['scans']}")
            print(f"Query scan id: {query.metadata['scans']}")
            print(f"Score: {score['score']:.4f}")
            print(f"Number of matching peaks: {score['matches']}")
            print("----------------------------")

Should output

.. testoutput::

    Reference scan id: 1320
    Query scan id: 1372
    Score: 0.9143
    Number of matching peaks: 25
    ----------------------------
    Reference scan id: 2833
    Query scan id: 1320
    Score: 0.0144
    Number of matching peaks: 20
    ...

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
