Strain-velocity Conversion
=============================================

DAS measures strain or strain rate, whereas traditional seismological studies typically use displacement, velocity or acceleration. Strain can be converted to particle velocity by multiplying the apparent phase velocity. Based on this principle, DASPy integrates three methods for converting strain/strain rate into velocity/acceleration (hereinafter collectively referred to as strain-velocity conversion): :ref:`FK Rescaling Method`, :ref:`Curvelet Transform Method` and :ref:`Time-Domain Slowness Detection Method`.

.. note::
    The example data for this section is the M4.3 earthquake recorded by a section of DAS in the Brady geothermal experimental field, which can be downloaded from `<https://raw.githubusercontent.com/HMZ-03/DASPy-data/main/Brady_DAS_M4.3_2688_2826.pkl>`_; The data recorded at the same time by No.165 node seismometer for comparison can be downloaded from `<https://raw.githubusercontent.com/HMZ-03/DASPy-data/main/Brady_node_M4.3_165.sac>`_. The distance between DAS channel 2781 and the seismometer at this node is 2.6 meters. The preprocessing that has been done on the DAS data includes: intercepting a certain linear segment (channels 2688 to 2826), removing the timing error (1.048 seconds), integrating the strain rate into strain, downsampling from 1000Hz to 500Hz, and bandpass filtering to 1~5Hz, trimmed from 19.99 seconds to 70.01 seconds after the earthquake origin (allowing a time difference of ±0.01 seconds when calculating cross-correlation with node seismometer data). The preprocessing that has been done on the node seismometer includes: rotating to the DAS axial component, filtering to 1~5Hz, and trimmed 20 seconds to 70 seconds after the earthquake occurs. Read via (requires `ObsPy <https://docs.obspy.org/>`_ installed):

    >>> import obspy
    >>> from daspy import read
    >>> st = obspy.read('Brady_node_M4.3_165.sac')
    >>> sec = read('Brady_DAS_M4.3_2688_2826.pkl')

.. _FK Rescaling Method:

FK Rescaling Method
------------------------------

The FK rescaling method achieves strain-to-velocity conversion by multiplying each point in the FK domain by its corresponding apparent velocity (slope in the FK domain).

::

    >>> sec_fk = sec.copy().fk_rescaling(fmax=(5, 6))


.. _Curvelet Transform Method:

Curvelet Transform Method
------------------------------

The basis function of the curvelet transform appears as a wedge with a certain velocity range in the FK domain. The curvelet transform method multiplies each curvelet coefficient by the median velocity of its basis function to achieve strain-velocity conversion.

    >>> sec_cv = sec.copy().curvelet_conversion()


.. _Time-Domain Slowness Detection Method:

Time-Domain Slowness Detection Method
------------------------------------------------------------

The time domain slowness determination method obtains the apparent speed at each time step by searching for the maximum semblance.

    >>> sec_ts = sec.copy().slant_stacking(L=10, frqlow=1, frqhigh=5, channel=2781) # only convert channel 2781 and save into the Section instance for efficiency


Comparison of Three Methods
------------------------------

    Define a function for calculating the cross-correlation coefficient and shift:

    >>> import numpy as np
    >>> def X_corr(a, b):
    >>>     M, N = len(a), len(b)
    >>>     b = (b - np.mean(b)) / np.std(b) / N
    >>>     cc = np.zeros(M-N+1)
    >>>     for i in range(M-N+1):
    >>>         a_p = a[i:i+N]
    >>>         a_p = (a_p - np.mean(a_p)) / np.std(a_p)
    >>>         cc[i] = np.correlate(a_p, b, 'valid')[0]
    >>>     return np.argmax(cc), max(cc)

    Plot the strain-velocity conversion results for the three methods:

    >>> import matplotlib.pyplot as plt
    >>> node_data = st[0].data
    >>> nt = len(node_data)
    >>> dt1, cc1 = X_corr(sec_fk.channel_data(2781), node_data)
    >>> fk_data = sec_fk.channel_data(2781)[dt1:dt1+nt]
    >>> dt2, cc2 = X_corr(sec_cv.channel_data(2781), node_data)
    >>> cv_data = sec_cv.channel_data(2781)[dt2:dt2+nt]
    >>> dt3, cc3 = X_corr(sec_ts.channel_data(2781), node_data)
    >>> ts_data = sec_ts.channel_data(2781)[dt3:dt3+nt]
    >>> 
    >>> fig, ax = plt.subplots(3, 1, figsize=(6, 8), dpi=250)
    >>> t = np.arange(nt)/sec.fs + 20
    >>> c = ['#999999', '#20639B', '#ED553B', '#F6D55C']
    >>> ax[0].set_title('M4.3 Earthquake')
    >>> ax[0].plot(t, node_data, c=c[0], lw=0.5)
    >>> ax[0].plot(t, fk_data-20, c=c[1], lw=0.5)
    >>> ax[0].plot(t, cv_data-40, c=c[2], lw=0.5)
    >>> ax[0].plot(t, ts_data-60, c=c[3], lw=0.5)
    >>> ax[0].text(22, 5, f'node data')
    >>> ax[0].text(22, -15, f'fk_res: cc={cc1:.2f}')
    >>> ax[0].text(22, -35, f'curvelet: cc={cc2:.2f}')
    >>> ax[0].text(22, -55, f'slowness: cc={cc3:.2f}')
    >>> ax[0].set_xlim([20, 70])
    >>> ax[0].set_ylim([-80, 20])
    >>> ax[0].set_yticks(np.arange(-60, 20, 20))
    >>> ax[0].set_ylabel('Velocity (nm/s)')
    >>> 
    >>> ax[1].set_title('P-wave')
    >>> ax[1].plot(t, node_data-3, c=c[0], lw=1)
    >>> ax[1].plot(t, node_data, c=c[0], lw=1)
    >>> ax[1].plot(t, node_data+3, c=c[0], lw=1)
    >>> ax[1].plot(t, fk_data+3, c=c[1], lw=1)
    >>> ax[1].plot(t, cv_data, c=c[2], lw=1)
    >>> ax[1].plot(t, ts_data-3, c=c[3], lw=1)
    >>> ax[1].set_xlim([27, 32])
    >>> ax[1].set_ylim([-6, 6])
    >>> ax[1].set_yticks(np.arange(-3, 6, 3))
    >>> ax[1].set_ylabel('Velocity (nm/s)')
    >>> 
    >>> ax[2].set_title('S-wave')
    >>> ax[2].plot(t, node_data+20, c=c[0], lw=1)
    >>> ax[2].plot(t, node_data, c=c[0], lw=1)
    >>> ax[2].plot(t, node_data-20, c=c[0], lw=1)
    >>> ax[2].plot(t, fk_data+20, c=c[1], lw=1)
    >>> ax[2].plot(t, cv_data, c=c[2], lw=1)
    >>> ax[2].plot(t, ts_data-20, c=c[3], lw=1)
    >>> ax[2].set_xlim([46, 56])
    >>> ax[2].set_ylim([-40, 40])
    >>> ax[2].set_yticks(np.arange(-20, 40, 20))
    >>> ax[2].set_ylabel('Velocity (nm/s)')
    >>> 
    >>> plt.tight_layout()
    >>> plt.show()

.. image:: ../media/strain2vel.png
    :width: 700