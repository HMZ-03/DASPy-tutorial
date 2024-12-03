Reading DAS Data
=============================================


Reading Data from Files
------------------------------

The ``read function`` is used to read multiple types of DAS data files. The supported data formats are shown in the following table:

+--------+------+-------+------------------------------------------------------------------------------------+
| format | read | write | source                                                                             |
+========+======+=======+====================================================================================+
| HDF5   | √    | √     | OptaSense, Silixa, Febus, AP Sensing, ASN, Sintela, Smart Earth Sensing and AI4EPS |
+--------+------+-------+------------------------------------------------------------------------------------+
| TDMS   | √    | √     | Silixa and the Institute of Semiconductors, CAS                                    |
+--------+------+-------+------------------------------------------------------------------------------------+
| SEG-Y  | √    | √     | OptaSens and Silixa                                                                |
+--------+------+-------+------------------------------------------------------------------------------------+
| PICKLE | √    | √     | daspy.Section or numpy.ndarray class instances saved in binary                     |
+--------+------+-------+------------------------------------------------------------------------------------+
| NPY    | √    | √     | numpy.ndarray class instances saved in binary                                      |
+--------+------+-------+------------------------------------------------------------------------------------+

By default, the function will output a ``Section`` class instance containing data and metadata:

    >>> from daspy import read
    >>> sec = read() # Read waveform example

You can set ``output_type='array'`` to output data in ``numpy.array`` format, in which case metadata will be output in ``dict`` format. Setting ``ch1`` or/and ``ch2`` limits the range of channels to read.

    >>> data, metadata = read(output_type='array', ch1=2700, ch2=2800) # set channel range

When only metadata is needed, you can set ``headonly=True`` to save reading time. In this case, the returned data will be an all-zero array of the same size as the original data.

Customize a Read Function
----------------------------------------------------------

If DASPy does not support reading your data, you can create a custom read function. Although the function can complete the reading with only the parameter ``fname`` , we recommend that users allow the function to input parameters ``headonly``, ``ch1`` and ``ch2`` to ensure full compatibility:

    >>> import h5py
    >>> from daspy import DASDateTime
    >>> 
    >>> def read_new_h5(fname, headonly=False, **kwargs):
    >>>     with h5py.File(fname, 'r') as h5_file:
    >>>         start_channel = h5_file.attrs['channel_start']
    >>>         end_channel = h5_file.attrs['channel_end']
    >>>         ch1 = kwargs.pop('ch1', start_channel)
    >>>         ch2 = kwargs.pop('ch2', end_channel)
    >>>         dch = kwargs.pop('dch', 1)
    >>>         if headonly:
    >>>             data = np.zeros_like(h5_file['raw_data'])
    >>>         else:
    >>>             data = h5_file['raw_data'][ch1-start_channel:ch2-start_channel:dch]
    >>>         dx = h5_file.attrs['channel spacing m']
    >>>         metadata = {'dx': dx, 'fs': h5_file.attrs['sampling rate Hz'],
    >>>                     'gauge_length': h5_file.attrs['GL m'],
    >>>                     'start_channel': start_channel + ch1,
    >>>                     'start_distance': (start_channel + ch1) * dx,
    >>>                     'start_time': DASDateTime.strptime(h5_file.attrs['starttime'], '%Y-%m-%dT%H:%M:%S.%f'),
    >>>                     'scale': h5_file.attrs['scale factor to strain']}
    >>>     return data, metadata

The required keywords in ``metadata`` are ``dx`` and ``fs`` (which can be temporarily set to ``None`` if they are not included in the file metadata), and the remaining parameters can be read on demand. This read function can then be input as the ``ftype`` parameter to the ``read`` function or used to construct a ``Collection`` class (See :doc:`Handling Continuous Data` for details).

    >>> sec = read(fname, ftype=read_new_h5, headonly=True)
    >>> coll = Collection('../data/*.h5', ftype=read_new_h5)

Converting from Instances of Other Packages
----------------------------------------------------------

It's supported to streaming data from `ObsPy <https://docs.obspy.org/>`_ , `DASCore <https://dascore.org/>`_ and `lightguide <https://github.com/pyrocko/lightguide>`_ to DASPy:

    >>> from daspy import Section
    >>> sec_obspy = Section.from_obspy_stream(st) # convert obspy.core.stream.Stream instance to daspy.section instance
    >>> sec_dascore = Section.from_dascore_patch(patch) # convert dascore.core.patch.Patch instance to daspy.section instance
    >>> sec_lightguide = Section.from_lightguide_blast(blast) # convert lightguide.blast.Blast instance to daspy.section instance

Section
------------------------------

The functions of DASPy can be implemented by calling functions (process-oriented) or through class methods (object-oriented). We recommend an object-oriented approach to operate on the ``Section`` class of the DAS data body. All user-facing functions and class methods in DASPy have the same name and are nouns (such as ``normalization`` and ``Section.normalization``).

The attributes of the ``Section`` class save the DAS data and header information, among which the waveform data ``data``, channel spacing ``dx``, and sampling rate ``fs`` must be set (but can be temporarily Set to ``None`` ); the starting channel number ``start_channel``, the starting distance ``start_distance``, and the starting time ``start_time`` default to 0 if not set; the event starting time ``origin_time``, gauge length ``gauge_length``, data type ``data_type``, data scale ``scale``, array geometry ``geometry``, turning point channel number ``turning_channels``, other header information ``headers`` are not necessary.

Accessing meta data：

    >>> print(sec)
                  data: shape(500, 5000)
                    dx: 1 m
                    fs: 100.0 Hz
         start_channel: 2500
        start_distance: 2520 m
            start_time: 2016-03-21 07:37:30.532309+00:00
           origin_time: 2016-03-21 07:37:10.535000+00:00
             data_type: strain rate

In addition, the data size ``shape``, the number of channels ``nch``, the number of sampling points ``nt``, the end channel number ``end_channel``, the end distance ``end_distance`` and the end time ``end_time`` will Automatically calculated and can be called as a property.


DASDateTime
------------------------------

DASPy create the ``DASDateTime`` class to represent the time information of the data, including the start time ``start_time``, the end time ``end_time`` and the event start time ``origin_time``.

``DASDateTime`` is a subclass of the ``datetime.DateTime`` class and inherits all methods of ``datetime.DateTime``:

    >>> from daspy.core import DASDateTime
    >>> DASDateTime.strptime('2021-03-19T1:52:23', '%Y-%m-%dT%H:%M:%S')
    DASDateTime(2021, 03, 19, 1, 52, 23)

DASPy has built-in local time zone ``local_tz`` and utc time zone ``utc`` for specifying the time zone:

    >>> from daspy.core.dasdatetime import utc, local_tz
    >>> DASDateTime.fromtimestamp(1616089943, tz=utc)
    DASDateTime(2021, 3, 18, 17, 52, 23, tzinfo=datetime.timezone.utc)

Use ``datetime.timezone(datetime.timedelta(hours=h))`` to create other time zones if needed.

You may use the ``local``, ``utc``, and ``remove_tz`` methods to convert or remove timezone information:

    >>> time = DASDateTime.strptime('2021-03-19T1:52:23Z', '%Y-%m-%dT%H:%M:%S%z')
    >>> time.local()
    DASDateTime(2021, 3, 19, 9, 52, 23, tzinfo=datetime.timezone(datetime.timedelta(seconds=28800)))
    >>> time.remove_tz()
    DASDateTime(2021, 3, 19, 1, 52, 23)

In addition to the addition and subtraction operations between ``datetime.datetime`` and ``datetime.timedelta`` supported by the parent class itself, ``DASDateTime`` also supports input numbers and iterable objects ``Iterable`` to calculate additions and subtraction. All time differences are expressed in seconds (s), and problems with unspecified time zones are automatically handled:

    >>> DASDateTime(2021, 3, 24, 14, 28, 0, 0) + 100
    DASDateTime(2021, 3, 24, 14, 29, 40)
    >>> DASDateTime(2021, 3, 24, 14, 28, 0, 0) + [10, 20, 30]
    [DASDateTime(2021, 3, 24, 14, 28, 10), DASDateTime(2021, 3, 24, 14, 28, 20), DASDateTime(2021, 3, 24, 14, 28, 30)]
    >>> DASDateTime(2021, 3, 24, 14, 28, 0, 0) - 100
    DASDateTime(2021, 3, 24, 14, 26, 20)
    >>> DASDateTime(2021, 3, 24, 14, 28, 0, 0) - DASDateTime(2021, 3, 19, 1, 52, 23)
    477337.0

``DASDateTime`` instances can be converted to parent class ``datetime.datetime`` instances when necessary:

    >>> DASDateTime(2021, 3, 19, 1, 52, 23).convert_to_datetime()
    datetime.datetime(2021, 3, 19, 1, 52, 23)