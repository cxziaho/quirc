# QUIRC Vita
Rewrote the makefile to support Vita compilation.  
  
Read README.orig for original instructions or see the [demo](https://github.com/cxziaho/qrdemo) for example usage.
    
## Building   
Assuming you have the [VitaSDK](http://vitasdk.org) toolchain:  
```  
git clone https://github.com/cxziaho/quirc-vita.git  
cd quirc-vita  
make
make install
```
  
Link your application with `-lquirc` (or just `quirc` in CMake) and include `quirc.h` in your code.  
  
## Usage (from original documentation)

  
All of the library's functionality is exposed through a single header
file, which you should include:

    #include <quirc.h>

To decode images, you'll need to instantiate a ``struct quirc``
object, which is done with the ``quirc_new`` function. Later, when you
no longer need to decode anything, you should release the allocated
memory with ``quirc_destroy``:

    struct quirc *qr;

    qr = quirc_new();
    if (!qr) {
	    perror("Failed to allocate memory");
	    abort();
    }

    /* ... */

    quirc_destroy(qr);

Having obtained a decoder object, you need to set the image size that
you'll be working with, which is done using ``quirc_resize``:

    if (quirc_resize(qr, 640, 480) < 0) {
	    perror("Failed to allocate video memory");
	    abort();
    }

``quirc_resize`` and ``quirc_new`` are the only library functions
which allocate memory. If you plan to process a series of frames (or a
video stream), you probably want to allocate and size a single decoder
and hold onto it to process each frame.
  
Processing frames is done in two stages. The first stage is an
image-recognition stage called identification, which takes a grayscale
image and searches for QR codes. Using ``quirc_begin`` and
``quirc_end``, you can feed a grayscale image directly into the buffer
that ``quirc`` uses for image processing:

    uint8_t *image;
    int w, h;

    image = quirc_begin(qr, &w, &h);

    /* Fill out the image buffer here.
     * image is a pointer to a w*h bytes.
     * One byte per pixel, w pixels per line, h lines in the buffer.
     */

    quirc_end(qr);

Note that ``quirc_begin`` simply returns a pointer to a previously
allocated buffer. The buffer will contain uninitialized data. After
the call to ``quirc_end``, the decoder holds a list of detected QR
codes which can be queried via ``quirc_count`` and ``quirc_extract``.

At this point, the second stage of processing occurs -- decoding. This
is done via the call to ``quirc_decode``, which is not associated with
a decoder object.

    int num_codes;
    int i;

    /* We've previously fed an image to the decoder via
     * quirc_begin/quirc_end.
     */

    num_codes = quirc_count(qr);
    for (i = 0; i < num_codes; i++) {
	    struct quirc_code code;
	    struct quirc_data data;
	    quirc_decode_error_t err;

	    quirc_extract(qr, i, &code);

	    /* Decoding stage */
	    err = quirc_decode(&code, &data);
	    if (err)
		    printf("DECODE FAILED: %s\n", quirc_strerror(err));
	    else
		    printf("Data: %s\n", data.payload);
    }

``quirc_code`` and ``quirc_data`` are flat structures which don't need
to be initialized or freed after use.