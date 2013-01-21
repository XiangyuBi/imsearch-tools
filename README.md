Web Image Downloader Tools
============================================

Authors:

 + Ken Chatfield, University of Oxford – <ken@robots.ox.ac.uk>
 + Kevin McGuinness, Dublin City University – <kevin.mcguinness@eeng.dcu.ie>

Copyright 2010-2013, all rights reserved.

Installation Instructions
-------------------------
 + Install the following Python dependencies:
     - `gevent`
     - `requests`
     - `pil` (Python Imaging Library)
     - `numpy`
     - `scipy`
     - `pyzmq` (ZeroMQ)
 + Add `imsearchtools` directory to your `PYTHON_PATH`
 + Update `authentication.py` in the `/imsearchtools/engines/api_credentials` directory
   with appropriate API keys for each method (see section below for how to obtain keys –
   note some default keys are provided, but it would be greatly appreciated if you could
   generate your own if using any of the API functions, as they are linked to my personal
   accounts *-Ken*)
   
Usage Instructions
------------------

### 1. Querying web engine for image URLs

    >> import imsearchtools
    >> google_searcher = imsearchtools.query.GoogleWebSearch()
    >> results = google_searcher.query('car')
    >> results
    [{'image_id': '43e9644258865f9eedacf08e73f552fa',
      'url': 'http://asset3.cbsistatic.com/cnwk.1d/i/tim/2012/09/19/35446285_620x433.jpg'},
     {'image_id': 'cfd0ae160c4de2ebbd4b71fd9254d6df',
      'url': 'http://asset0.cbsistatic.com/cnwk.1d/i/tim/2012/08/15/35414204_620x433.jpg'},
     … ]
     
Currently the following search services are supported:

 + **GoogleWebSearch( )** – Image search using Google, extracted direct from the web
     - Preferred method for Google Image search
 + **GoogleAPISearch ( )** – Image search using Google, using the *Google Custom Search API*
     - A limit of 100 images per search is imposed
     - The results are different (slightly worse) than when using direct extraction
     - A 'custom search engine' must be created to use the API, with a list of sites to
       search specified during creation. However, selecting 'search these sites + entire
       web' in the options appears to give identical results to those returned by
       `GoogleAPISearch()`
     - Details and authentication key available at:
       <https://developers.google.com/custom-search/v1/overview/>
 + **GoogleOldAPISearch ( )** – Image search using Google, using the *Google Image Search API*
     - The *Google Image Search API* is now deprecated
     - A limit of 64 images per search is imposed
     - There is a higher default limit on number of free requests/day than with the
       new API
     - Details and authentication key available at:
       <https://developers.google.com/image-search/>
 + **BingAPISearch ( )** – Image search using Bing, using the *Bing Search API*
     - Details and authentication key available at:
       <http://www.bing.com/developers/>
 + **FlickrAPISearch ( )** – Image search using Flickr, using the *Flickr API*
     - Provides text search of Flickr photos by associated tags
     - Details and authentication key available at:
       <http://www.flickr.com/services/api/>
       
A test script `query_test.py` is provided which can be used to visualize the difference
between the methods:

    $ python query_test.py <query>

### 2. Verifying and downloading retrieved image URLs

Given the `results` array returned by `<web_service>.query(q)`, all URLs can be processed
and downloaded to local storage using the `process.ImageGetter()` class:

    ...
    >> getter = imsearchtools.process.ImageGetter()
    >> paths = getter.process_urls(results, '/path/to/save/images')
    >> paths
    [{'clean_fn': '/path/43e9644258865f9eedacf08e73f552fa-clean.jpg',
      'image_id': '43e9644258865f9eedacf08e73f552fa',
      'orig_fn': '/path/43e9644258865f9eedacf08e73f552fa.jpg',
      'thumb_fn': '/path/43e9644258865f9eedacf08e73f552fa-thumb-90x90.jpg',
      'url': 'http://asset3.cbsistatic.com/cnwk.1d/i/tim/2012/09/19/35446285_620x433.jpg'},
     {'clean_fn': '/path/cfd0ae160c4de2ebbd4b71fd9254d6df-clean.jpg',
      'image_id': 'cfd0ae160c4de2ebbd4b71fd9254d6df',
      'orig_fn': '/path/cfd0ae160c4de2ebbd4b71fd9254d6df.jpg',
      'thumb_fn': '/path/cfd0ae160c4de2ebbd4b71fd9254d6df-thumb-90x90.jpg',
      'url': 'http://asset0.cbsistatic.com/cnwk.1d/i/tim/2012/08/15/35414204_620x433.jpg'},
      … ]

`process_urls` returns a list of dictionaries, with each dictionary containing details of
images which were successfully downloaded:

 + `url` is the source URL for the image, the same as returned from `<web_service>.query(q)`
 + `image_id` is a unique identifier for the image, the same as returned from
   `<web_service>.query(q)`
 + `orig_fn` is the path of the original file downloaded direct from `url` – this image is
   unverified, and depending on the source URL may be corrupt
 + `clean_fn` is the path to a verified copy of `orig_fn`, which has been standardized
   according to the class options
 + `thumb_fn` is the path to a thumbnail version of `orig_fn`
 
A test script `download_test.py` is provided which can be used to demonstrate the usage of
the `process.ImageGetter()` class:

    $ python download_test.py
    
#### Configuring verification and download settings

Options for image verification and thumbnail generation can be customized by passing an
instance of the `process.ImageProcessorSettings` class to `process.ImageGetter()` during
initialization e.g.:

    >> opts = imsearchtools.process.ImageProcessorSettings()
    >> opts.filter['max_height'] = 600   # set maximum image size to 800x600
    >> opts.filter['max_width'] = 800    #    (discarding larger images)
    >> opts.conversion['format'] = 'png' # change output format
    >> opts.thumbnail['height'] = 50     # change width and height of thumbnails to 50x50
    >> opts.thumbnail['width'] = 50
    >> opts.thumbnail['pad_to_size'] = False # don't add padding to thumbnails
    >> getter = imsearchtools.process.ImageGetter(opts)
    
#### Adding a callback for post image download

Optionally, a callback function can be added which will be called immediately after each
image is downloaded and processed when using `process.ImageGetter.process_urls()`. To
do this, specify the callback when calling `process_urls()`:

    import imsearchtools
    
    def callback_func(out_dict):
        import json
        print json.dumps(out_dict)
        sleep(0.1)
    
    google_searcher = imsearchtools.query.GoogleWebSearch()
    results = google_searcher.query('car')
    
    getter = imsearchtools.process.ImageGetter()
    getter.process_urls(results, '/path/to/save/images',
                        completion_func=callback_func,
                        completion_worker_count=8)
                        
The form of the callback should be `f(out_dict)` where `out_dict` is a dictionary of the
same form as a single entry in the list returned from `process_urls()`.

The callbacks will be executed using a pool of worker processes, the size of which is
determined by the `completion_worker_count` parameter. If it is not specified, by default
*N* workers will be launched where *N* is the number of CPUs on the local system.

HTTP Service
------------

A simple HTTP interface to the library is provided by `imsearch_http_service.py` and
can be launched by calling:

    python imsearch_http_service.py

    
Revision History
----------------

 + *Jan 2013*
     - Fixed issue with timeout by migrating from `restkit` to `requests` library
     - Added missing `gevent` monkey-patching to provide speed boost
     - Added HTTP service (including callback modules)
 + *Oct 2012*
     - Added support for Bing, the new Google API and Flickr, updated to new interface
     - Added `gevent` async support
     - Updated code for downloading and verification of images
     - Added documentation
 + *May 2011*
     - Updated Google web search method due to updates
 + *Nov 2010*
     - Original version with support for Google Image Search API + scraping
 