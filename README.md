This is a clone of Simple OpenGL Image Library from http://lonesock.net/soil.html which hasn't changed since July 7, 2008.

I wanted to work with the code and seeing it was MIT license and the original svn repo is offline, I figured it was acceptable to post it here.

Features:
=========

* Readable Image Formats:
  * BMP - non-1bpp, non-RLE (from stb_image documentation)
  * PNG - non-interlaced (from stb_image documentation)
  * JPG - JPEG baseline (from stb_image documentation)
  * TGA - greyscale or RGB or RGBA or indexed, uncompressed or RLE
  * DDS - DXT1/2/3/4/5, uncompressed, cubemaps (can't read 3D DDS files yet)
  * PSD - (from stb_image documentation)
  * HDR - converted to LDR, unless loaded with *HDR* functions (RGBE or RGBdivA or RGBdivA2) 
* Writeable Image Formats:
  * TGA - Greyscale or RGB or RGBA, uncompressed
  * BMP - RGB, uncompressed
  * DDS - RGB as DXT1, or RGBA as DXT5 
* Can load an image file directly into a 2D OpenGL texture, optionally performing the following functions:
  * Can generate a new texture handle, or reuse one specified
  * Can automatically rescale the image to the next largest power-of-two size
  * Can automatically create MIPmaps
  * Can scale (not simply clamp) the RGB values into the "safe range" for NTSC displays (16 to 235, as recommended here)
  * Can multiply alpha on load (for more correct blending / compositing)
  * Can flip the image vertically
  * Can compress and upload any image as DXT1 or DXT5 (if EXT_texture_compression_s3tc is available), using an internal (very fast!) compressor
  * Can convert the RGB to YCoCg color space (useful with DXT5 compression: see this link from NVIDIA)
  * Will automatically downsize a texture if it is larger than GL_MAX_TEXTURE_SIZE
  * Can directly upload DDS files (DXT1/3/5/uncompressed/cubemap, with or without MIPmaps). Note: directly uploading the compressed DDS image will disable the other options (no flipping, no pre-multiplying alpha, no rescaling, no creation of MIPmaps, no auto-downsizing)
  * Can load rectangluar textures for GUI elements or splash screens (requires GL_ARB/EXT/NV_texture_rectangle) 
* Can decompress images from RAM (e.g. via PhysicsFS or similar) into an OpenGL texture (same features as regular 2D textures, above)
* Can load cube maps directly into an OpenGL texture (same features as regular 2D textures, above)
  * Can take six image files directly into an OpenGL cube map texture
  * Can take a single image file where width = 6*height (or vice versa), split it into an OpenGL cube map texture 
* No external dependencies
* Tiny
* Cross platform (Windows, *nix, Mac OS X)
* Public Domain 

Usage:
=======

SOIL is meant to be used as a static library (as it's tiny and in the public domain). You can use the static library file included in the zip (libSOIL.a works for MinGW and Microsoft compilers...feel free to rename it to SOIL.lib if that makes you happy), or compile the library yourself. The code is cross-platform and has been tested on Windows, Linux, and Mac. (The heaviest testing has been on the Windows platform, so feel free to email me if you find any issues with other platforms.)

Simply include SOIL.h in your C or C++ file, link in the static library, and then use any of SOIL's functions. The file SOIL.h contains simple doxygen style documentation. (If you use the static library, no other header files are needed besides SOIL.h) Below are some simple usage examples:

load an image file directly as a new OpenGL texture

    GLuint tex_2d = SOIL_load_OGL_texture
    (
      "img.png",
      SOIL_LOAD_AUTO,
      SOIL_CREATE_NEW_ID,
      SOIL_FLAG_MIPMAPS | SOIL_FLAG_INVERT_Y | SOIL_FLAG_NTSC_SAFE_RGB | SOIL_FLAG_COMPRESS_TO_DXT
    );

check for an error during the load process

    if( 0 == tex_2d )
    {
      printf( "SOIL loading error: '%s'\n", SOIL_last_result() );
    }

load another image, but into the same texture ID, overwriting the last one

    tex_2d = SOIL_load_OGL_texture
    (
      "some_other_img.dds",
      SOIL_LOAD_AUTO,
      tex_2d,
      SOIL_FLAG_DDS_LOAD_DIRECT
    );

load 6 images into a new OpenGL cube map, forcing RGB

    GLuint tex_cube = SOIL_load_OGL_cubemap
    (
      "xp.jpg",
      "xn.jpg",
      "yp.jpg",
      "yn.jpg",
      "zp.jpg",
      "zn.jpg",
      SOIL_LOAD_RGB,
      SOIL_CREATE_NEW_ID,
      SOIL_FLAG_MIPMAPS
    );

load and split a single image into a new OpenGL cube map, default format
face order = East South West North Up Down => "ESWNUD", case sensitive!

    GLuint single_tex_cube = SOIL_load_OGL_single_cubemap
    (
      "split_cubemap.png",
      "EWUDNS",
      SOIL_LOAD_AUTO,
      SOIL_CREATE_NEW_ID,
      SOIL_FLAG_MIPMAPS
    );

actually, load a DDS cubemap over the last OpenGL cube map, default format
try to load it directly, but give the order of the faces in case that fails
the DDS cubemap face order is pre-defined as SOIL_DDS_CUBEMAP_FACE_ORDER

    single_tex_cube = SOIL_load_OGL_single_cubemap
    (
      "overwrite_cubemap.dds",
      SOIL_DDS_CUBEMAP_FACE_ORDER,
      SOIL_LOAD_AUTO,
      single_tex_cube,
      SOIL_FLAG_MIPMAPS | SOIL_FLAG_DDS_LOAD_DIRECT
    );

load an image as a heightmap, forcing greyscale (so channels should be 1)

    int width, height, channels;
    unsigned char *ht_map = SOIL_load_image
    (
      "terrain.tga",
      &width, &height, &channels,
      SOIL_LOAD_L
    );

save that image as another type

    int save_result = SOIL_save_image
    (
      "new_terrain.dds",
      SOIL_SAVE_TYPE_DDS,
      width, height, channels,
      ht_map
    );

save a screenshot of your awesome OpenGL game engine, running at 1024x768

    save_result = SOIL_save_screenshot
    (
      "awesomenessity.bmp",
      SOIL_SAVE_TYPE_BMP,
      0, 0, 1024, 768
    );

loaded a file via PhysicsFS, need to decompress the image from RAM,
where it's in a buffer: unsigned char *image_in_RAM

    GLuint tex_2d_from_RAM = SOIL_load_OGL_texture_from_memory
    (
      image_in_RAM,
      image_in_RAM_bytes,
      SOIL_LOAD_AUTO,
      SOIL_CREATE_NEW_ID,
      SOIL_FLAG_MIPMAPS | SOIL_FLAG_INVERT_Y | SOIL_FLAG_COMPRESS_TO_DXT
    );

done with the heightmap, free up the RAM

    SOIL_free_image_data( ht_map );
