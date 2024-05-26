Basic Processing
=============================================

DASPy integrates three modules: :ref:`Preprocessing`, :ref:`Filtering` and :ref:`Frequency Attribute` to meet basic data processing needs.


.. _Preprocessing:

Preprocessing
------------------------------

DASPy integrates a large number of commonly used DAS preprocessing tools, including ``phase2strain`` , ``normalization`` , ``demeaning`` , ``detrending`` , ``stacking`` , ``cosine_taper`` , ``downsampling`` , ``trimming`` , ``padding`` , ``time_integration`` and ``time_differential`` 。

    >>> from daspy import read
    >>> sec = read()
    >>> sec.detrending()

When processing the data, the corresponding attributes will be automatically changed if necessary:

    >>> sec.data_type
    'strain rate'
    >>> sec.time_integration()
    >>> sec.data_type
    'strain'


.. _Filtering:

Filtering
------------------------------

DASPy contains commonly used filtering methods, including ``bandpass`` , ``bandstop`` , ``lowpass`` , ``highpass`` , ``envelope`` and `lowpass_cheby_2`` (mainly used for low-pass filtering before downsampling):

    >>> sec.bandpass(1, 15)


.. _Frequency Attribute:

Frequency Attribute
------------------------------

The frequency domain attribute analysis supported by DASPy includes spectrum (x-t), spectrogram (f-t) and frequency-wavenumber spectrum (f-k), which are calculated using ``spectrum``, ``spectrogram`` and ``fk-transform`` respectively. method calculation. ``spectrogram`` calculates the average spectrogram of all channels by default. You can use the ``xmin`` and ``xmax`` parameters to limit the starting and ending channels of the average spectrum:

    >>> spec, f = sec.spectrum()
    >>> Zxx, f, t = sec.spectrogram(xmin=2600, xmax=2620)
    >>> fk, f, k = sec.fk_transform()

If you only need to plot these three frequency domain spectra without outputting the calculation results, you can directly use the ``plot`` method. For details, see :doc:`Visualization` .
