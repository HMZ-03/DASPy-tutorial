Wavefield Decomposition
=============================================

The wavefield recorded by DAS usually has complex components. We often need to separate the wavefield according to apparent velocity to extract some of the required signals, such as separating seismic signals and traffic signals, separating direct wave and scattered wave, etc. Due to the good equidistant characteristics of DAS, the wavefield recorded by DAS can be quickly and easily decomposed using image processing technology. DASPy uses two-dimensional fast Fourier transform (called FK transform in seismic data processing, see :ref:`FK Filtering` ) and fast discrete curvelet transform technology (see :ref:`Curvelet Windowing Technology` ) to decompose the DAS wavefield.

.. note::
    The example data for this Section is the 30-second M1.79 earthquake signal recorded by Ridgecrest DAS, which can be downloaded from `<https://data.caltech.edu/records/31emd-wmv98/files/EQ_raw_figure_1.mat?download=1>`_, read and preprocess via:


    >>> import scipy.io as scio
    >>> from daspy import Section
    >>> data = scio.loadmat('EQ_raw_figure_1.mat')['EQ_raw'].T # read the data
    >>> sec = Section(data, 8, 250) # build Section instance
    >>> sec.spike_removal() # remove spikes
    >>> sec.channel_checking(use=True) # remove bad channels


.. _FK Filtering:

FK Filtering
------------------------------

The FK filtering method uses two-dimensional fast Fourier transform to convert the data into the frequency-wavenumber (FK) domain, multiplies the FK domain by the filter window and then inversely transforms it back to the time-distance domain. If the apparent wave speed of traffic signals in the area is less than 1000m/s and the apparent wave speed of seismic waves is greater than 1000m/s, you can use ``vmin=(800, 1200)`` to filter out the traffic signals. The first output is seismic waves, and the second outputs are traffic signals. ``800`` and ``1200`` are the taper boundaries:

    >>> sec_eq, sec_tf = sec.fk_filter(vmin=(800, 1200), mode='decompose')

FK filtering often causes various artifacts, particularly edge artifacts caused by discontinuities at the waveform's edge, and star-like artifacts originating from discontinuities in the FK domain. To minimize these artifacts, DASPy employs cosine tapers (e.g. Tukey window) on the waveforms, as well as the filtering window in the FK domain.

Plot the waveforms and FK spectra of the original waveform, the separated direct wave and the scattered wave:

    >>> import matplotlib.pyplot as plt
    >>> fig, ax = plt.subplots(3, 2, figsize=(6,6), sharex='col', sharey='col', dpi=200)
    >>> sec.plot(ax=ax[0,0], xlabel=False)
    >>> sec_dr.plot(ax=ax[1,0], xlabel=False)
    >>> sec_sc.plot(ax=ax[2,0])
    >>> from daspy.basic_tools.visualization import plot
    >>> vmin, vmax = 0, 2e5
    >>> plot(fk, obj='fk', f=f, k=k, ax=ax[0,1], vmin=vmin, vmax=vmax,xlabel=False)
    >>> plot(fk*mask, obj='fk', f=f, k=k, ax=ax[1,1], vmin=vmin, vmax=vmax, xlabel=False)
    >>> plot(fk*(1-mask), obj='fk', f=f, k=k, ax=ax[2,1], vmin=vmin, ylim=[0,30], vmax=vmax)
    >>> plt.tight_layout()
    >>> plt.show()

.. image:: ../media/fk_filter.png
    :width: 700


.. _Curvelet Windowing Technology:

Curvelet Windowing Technology
------------------------------

The curvelet transform can similarly achieve the effect of decomposing the wave field at the apparent wave velocity.

Separate seismic waves and traffic signals at a speed of 1000m/s:

    >>> sec_eq, sec_tf = sec.curvelet_windowing(mode='decompose', vmin=1000)

Plot the original waveform, the separated direct wave, and the scattered wave:

    >>> import matplotlib.pyplot as plt
    >>> fig, ax = plt.subplots(1, 3, figsize=(8,3), sharex='row', sharey='row', dpi=200)
    >>> plot_kwargs = dict(vmax=1, colorbar=False)
    >>> sec.plot(ax=ax[0], title='Raw', **plot_kwargs)
    >>> sec_eq.plot(ax=ax[1], title='Earthquake', ylabel=False, **plot_kwargs)
    >>> sec_tf.plot(ax=ax[2], title='Traffic', ylabel=False, **plot_kwargs)
    >>> plt.tight_layout()
    >>> plt.show()

.. image:: ../media/curvelet_windowing.png
    :width: 700