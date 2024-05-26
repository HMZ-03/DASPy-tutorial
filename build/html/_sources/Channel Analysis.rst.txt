Channel Analysis
=============================================

DASPy's channel attribute analysis module: :ref:`Location Interpolation`, :ref:`Turning Point Detection`, :ref:`Channel Quality Checking`.


.. _Location Interpolation:

Location Interpolation
------------------------------

DASPy's channel position interpolation supports the joint use of two types of data: known points containing channel numbers (i.e. longitude & latitude & channel number) and track points without channel numbers (ie longitude & latitude), where the known points are necessary, and track points are optional to constrain the cable geometry. DASPy using `Universal Transverse Mercator projection <https://en.wikipedia.org/wiki/Universal_Transverse_Mercator_coordinate_system>`_ (UTM), interpolate in the plane coordinate system and back-project back to the `WGS84 Coordinate System <https://en.wikipedia.org/wiki/World_Geodetic_System>`_ 。

.. note::
    The example data is coordinates of the North cable of the RAPID dataset, which can be read online via:

    >>> import numpy as np
    >>> txt_url = 'http://piweb.ooirsn.uw.edu/das/processed/metadata/Geometry/OOI_RCA_DAS_channel_location/north_cable_latlon.txt'
    >>> track_pt = np.loadtxt(txt_url)[:, ::-1] # read in the track points and swap the two columns (let longitude precedes latitude)
    >>> known_pt = np.array([[*track_pt[0], 942], [*track_pt[-1], 32459]]) # the 0th track point corresponds to channel 942, the last track point corresponds to channel 32459, and the channel numbers of the remaining track points are unknown
    
    Or read after downloading from `<http://piweb.ooirsn.uw.edu/das/processed/metadata/Geometry/OOI_RCA_DAS_channel_location/north_cable_latlon.txt>`_.

::

    >>> from daspy.advanced_tools.channel import location_interpolation
    >>> interp_ch = location_interpolation(known_pt, track_pt=track_pt, dx=2) # input the track points and the known number points, the channel spacing is 2m, and get the interpolation result
    >>> print(interp_ch) # longitude, latitude, and channel number
    [[ -123.96715       45.2023       942.        ]
     [ -123.96717541    45.20229612   943.        ]
     [ -123.96720083    45.20229224   944.        ]
     ...
     [ -124.75097192    45.24242661 32457.        ]
     [ -124.75099513    45.2424183  32458.        ]
     [ -124.75101833    45.24241    32459.        ]]
    >>> 
    >>> import matplotlib.pyplot as plt # plotting
    >>> plt.scatter(interp_ch[:,0], interp_ch[:,1], c=interp_ch[:,2], cmap='jet')
    >>> plt.scatter(track_pt[:,0], track_pt[:,1], c='k', s=10)
    >>> plt.gca().set_aspect('equal')
    >>> plt.title('Interpolated channel location')
    >>> plt.show()

.. image:: ../media/chn_interp.png
    :width: 500


.. _Turning Point Detection:

Turning Point Detection
------------------------------

DASPy detect turning points detection through two methods: calculating the corner of the fiber through the channel coordinates, or finding channels with low adjacent channel cross-correlation values in the waveform record.

Calculated by channel latitude and longitude:

.. note::
    The example data is the DAS channel coordinates of the Brady geothermal field, which can be read online via:

    >>> import numpy as np
    >>> csv_url = 'https://raw.githubusercontent.com/HMZ-03/DASPy-data/main/Brady_DAS_Coordinates.csv'
    >>> geometry = np.loadtxt(csv_url, delimiter=',', skiprows=2, usecols=[5,4,3])
    >>> geometry = geometry[geometry[:,0] != 0] # exclude empty records
    
    or read after downloading from `<https://raw.githubusercontent.com/HMZ-03/DASPy-data/main/Brady_DAS_Coordinates.csv>`_.

::

    >>> from daspy.advanced_tools.channel import turning_points
    >>> turning_h, turning_v = turning_points(geometry, depth_info=True) # the data contains depth information, detect turning points both horizontally and vertically
    >>> 
    >>> import matplotlib.pyplot as plt # plotting
    >>> plt.scatter(geometry[:, 0], geometry[:, 1], c='y', s=5)
    >>> plt.scatter(geometry[turning_v, 0], geometry[turning_v, 1], c='g', s=5, label='vertical')
    >>> plt.scatter(geometry[turning_h, 0], geometry[turning_h, 1], c='r', s=5, label='horizontal')
    >>> plt.gca().set_aspect('equal')
    >>> plt.title('Turning points')
    >>> plt.legend()
    >>> plt.show()

.. image:: ../media/turning_points.png
    :width: 500


.. _Channel Quality Checking:

Channel Quality Checking
------------------------------

Sometimes there are areas with poor coupling conditions along optical fiber, such as reserved loops in communication optical cables, resulting in "bad channels". These "bad channels" usually correspond to areas on the waveform with abnormally low or abnormally high amplitudes. When the coupling conditions are unknown, DASPy can use a DAS record to check the channel quality and determine the so-called "bad channels":

.. note::
    The sample data is a 15-second traffic signal recorded by Ridgecrest DAS, which can be downloaded from `<https://data.caltech.edu/records/31emd-wmv98/files/Traffic_noise_figure_4.mat?download=1>`_ and read via the following method:

    >>> import scipy.io as scio
    >>> data = scio.loadmat('Traffic_noise_figure_4.mat')['Traffic_noise_figure_4'].T
    >>> sec = Section(data, 8, 250)

Call a function to check good and bad channels:

    >>> from daspy.advanced_tools.channel import channel_checking
    >>> good_chn, bad_chn = channel_checking(data)
    >>> print(bad_chn)
    [  11   12   13   14   18   19   20   21   22   23   81   82   83   84
    85   86   87   88   89  142  143  144  145  146  255  256  257  258
    259  260  261  262  263  264  265  266  267  268  269  270  454  455
    456  457  458  459  460  461  462  463  464  465  466  467  468  469
    470  471  472  664  665  666  667  668  669  842  843  844  845  846
    847  848  849  850  851  852  853  854  855  856  857  858  859  860
    861  862  863  864  865  866  867  868  869  870  871 1059 1060 1061
    1062 1063 1064 1065 1066]

Directly remove bad channels in the ``daspy.Section`` instance:

    >>> sec.channel_checking(use=True)
