Processing continuous data
===============================================

When processing continuous-time signals such as various types of filtering, downsampling, cosine taper, time differentiation or integration, processing each single file and then splicing then will cause data discontinuity. Using the ``collection`` class to process continuous data can solve the continuity problem of these operations.

Collection
------------------------------

The ``Collection`` class is used to store continuous data belonging to the same acquisition. It saves the list of continuous data file paths ``flist`` and the start time of each file ``ftime``, as well as the metadata information of the acquisition, including the number of channels ``nch``, the track spacing ``dx``, the gauge length ``gauge_length``, the sampling rate ``fs``, the number of sampling points of a single file ``nt`` and the duration of a single file ``flength``. When initializing a class instance, by default, the metadata of the second data file is read to obtain all metadata information, and the start time of each file ``ftime`` is automatically calculated based on the start time ``start_time`` of the second data:

    >>> from daspy import Collection
    >>> coll = Collection('data/*.h5')
    >>> coll
           flist: 2608 files
                  [data/TEST_2000m_4m_1_5000Hz_200Hz_UTC8_202312132304.h5,
                   data/TEST_2000m_4m_1_5000Hz_200Hz_UTC8_202312132305.h5,
                   ...,
                   data/TEST_2000m_4m_1_5000Hz_200Hz_UTC8_202312151831.h5]
           ftime: 2023-12-13 23:04:00.558582+00:00 to 2023-12-15 18:32:00.558582+00:00
         flength: 60.0
             nch: 1956
              nt: 12000
              dx: 1.022494912147522
              fs: 200.0
    gauge_length: 4.0

- Set ``meta_from_file='all'`` to read metadata for each file. DASPy will check if all metadata are consistent, data is continuous, and save and calculate required properties. Metadata of various types can also be specified directly. This method is very time-consuming for large amounts of continuous data.
- Set ``timeinfo_format`` to get the start time from the file name, check for continuity and calculate ``flength``. ``timeinfo_format='DAS_%Y-%m-%dT%H:%M:%S%z.h5'`` indicates the time corresponding to the file name, ``timeinfo_format=(slice(33:45), '%Y%m%d%H%M%S')`` indicates the time corresponding to the slice of the file name.

Data Selection
------------------------------

Select data for a certain time period from the ``Collection`` class:

    >>> from daspy import DASDateTime
    >>> coll.select(stime = DASDateTime(2023, 12, 14, 12), etime=DASDateTime(2023, 12, 14, 13))
           flist: 61 files
                  [data/TEST_2000m_4m_1_5000Hz_200Hz_UTC8_202312141159.h5,
                   data/TEST_2000m_4m_1_5000Hz_200Hz_UTC8_202312141200.h5,
                   ...,
                   data/TEST_2000m_4m_1_5000Hz_200Hz_UTC8_202312141259.h5]
           ftime: 2023-12-13 23:04:00.558582+00:00 to 2023-12-15 18:32:00.558582+00:00
         flength: 60.0
             nch: 1956
              nt: 12000
              dx: 1.022494912147522
              fs: 200.0
    gauge_length: 4.0

Setting ``readsec=True`` will directly read the data of the required period as an instance of the ``Section`` class and return it.


Continuous Data Processing
------------------------------

The ``Collection.process`` method can read all files in turn as ``Section`` class instances, perform a series of required processing, add a suffix and save to the target directory. This method will solve data discontinuity by caching filter states and other methods. An example of low-pass filtering all data and downsampling by 5 times in both time and space domains, and then integrating:

    >>> operations = [['downsampling', dict(tint=5, xint=5, lowpass_filter=True)],
                      ['time_integration', dict()]]
    >>> coll.process(operations, savepath='../processed', suffix='_pro')