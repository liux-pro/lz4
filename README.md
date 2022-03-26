Work with Platform.io
------------
this project is forked from [here](https://github.com/lz4/lz4), a fast compression library, written in c.  
I just add `library.json`, which makes it work for platform.io.  
You can add this library to you embedded project by edit `platformio.ini`
```ini
lib_deps =
    liux-pro/lz4@^0.0.1
```
# some example on esp8266
## example1 
a python script that generate lz4 compressed data on pc.  
full code at https://github.com/liux-pro/monitor-server
```python
# pip install lz4
import lz4.block
compressedData = lz4.block.compress(bytes(data), store_size=False)
# then send compressedData to esp8266
```
on esp8266, decompress `compressedData` and save result in  `originData`, both length of them are required.  
full code at https://github.com/liux-pro/monitor
```c
#include "lz4.h"
LZ4_decompress_safe(compressedData, originData, lengthOfCompressedData, lengthOriginData);
```
## example2
Compress a string and then decompress. Based on example/simple_buffer.c. make sure to check LICENSE before you use is.
```c
#include <Arduino.h>
#include "lz4.h"

void setup()
{
    Serial.begin(115200);
    delay(1000);
}

void loop()
{
    const char *const src = "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Lorem ipsum dolor site amat.";
    const int src_size = (int)(strlen(src) + 1);
    const int max_dst_size = LZ4_compressBound(src_size);
    char *compressed_data = (char *)malloc((size_t)max_dst_size);
    const int compressed_data_size = LZ4_compress_default(src, compressed_data, src_size, max_dst_size);
    compressed_data = (char *)realloc(compressed_data, (size_t)compressed_data_size);

    /* Decompression */
    char *const regen_buffer = (char *)malloc(src_size);
    const int decompressed_size = LZ4_decompress_safe(compressed_data, regen_buffer, compressed_data_size, src_size);
    Serial.println(regen_buffer);
    free(compressed_data);
    free(regen_buffer);

    delay(1000);
}
```
LZ4 - Extremely fast compression
================================
LZ4 is lossless compression algorithm,
providing compression speed > 500 MB/s per core,
scalable with multi-cores CPU.
It features an extremely fast decoder,
with speed in multiple GB/s per core,
typically reaching RAM speed limits on multi-core systems.

Speed can be tuned dynamically, selecting an "acceleration" factor
which trades compression ratio for faster speed.
On the other end, a high compression derivative, LZ4_HC, is also provided,
trading CPU time for improved compression ratio.
All versions feature the same decompression speed.

LZ4 is also compatible with [dictionary compression](https://github.com/facebook/zstd#the-case-for-small-data-compression),
both at [API](https://github.com/lz4/lz4/blob/v1.8.3/lib/lz4frame.h#L481) and [CLI](https://github.com/lz4/lz4/blob/v1.8.3/programs/lz4.1.md#operation-modifiers) levels.
It can ingest any input file as dictionary, though only the final 64KB are used.
This capability can be combined with the [Zstandard Dictionary Builder](https://github.com/facebook/zstd/blob/v1.3.5/programs/zstd.1.md#dictionary-builder),
in order to drastically improve compression performance on small files.


LZ4 library is provided as open-source software using BSD 2-Clause license.


|Branch      |Status   |
|------------|---------|
|dev         | [![Build Status][travisDevBadge]][travisLink]    [![Build status][AppveyorDevBadge]][AppveyorLink]                                         |

[travisDevBadge]: https://travis-ci.org/lz4/lz4.svg?branch=dev "Continuous Integration test suite"
[travisLink]: https://travis-ci.org/lz4/lz4
[AppveyorDevBadge]: https://ci.appveyor.com/api/projects/status/github/lz4/lz4?branch=dev&svg=true "Windows test suite"
[AppveyorLink]: https://ci.appveyor.com/project/YannCollet/lz4-1lndh


Benchmarks
-------------------------

The benchmark uses [lzbench], from @inikep
compiled with GCC v8.2.0 on Linux 64-bits (Ubuntu 4.18.0-17).
The reference system uses a Core i7-9700K CPU @ 4.9GHz (w/ turbo boost).
Benchmark evaluates the compression of reference [Silesia Corpus]
in single-thread mode.

[lzbench]: https://github.com/inikep/lzbench
[Silesia Corpus]: http://sun.aei.polsl.pl/~sdeor/index.php?page=silesia

|  Compressor             | Ratio   | Compression | Decompression |
|  ----------             | -----   | ----------- | ------------- |
|  memcpy                 |  1.000  | 13700 MB/s  |  13700 MB/s   |
|**LZ4 default (v1.9.0)** |**2.101**| **780 MB/s**| **4970 MB/s** |
|  LZO 2.09               |  2.108  |   670 MB/s  |    860 MB/s   |
|  QuickLZ 1.5.0          |  2.238  |   575 MB/s  |    780 MB/s   |
|  Snappy 1.1.4           |  2.091  |   565 MB/s  |   1950 MB/s   |
| [Zstandard] 1.4.0 -1    |  2.883  |   515 MB/s  |   1380 MB/s   |
|  LZF v3.6               |  2.073  |   415 MB/s  |    910 MB/s   |
| [zlib] deflate 1.2.11 -1|  2.730  |   100 MB/s  |    415 MB/s   |
|**LZ4 HC -9 (v1.9.0)**   |**2.721**|    41 MB/s  | **4900 MB/s** |
| [zlib] deflate 1.2.11 -6|  3.099  |    36 MB/s  |    445 MB/s   |

[zlib]: http://www.zlib.net/
[Zstandard]: http://www.zstd.net/

LZ4 is also compatible and optimized for x32 mode,
for which it provides additional speed performance.


Installation
-------------------------

```
make
make install     # this command may require root permissions
```

LZ4's `Makefile` supports standard [Makefile conventions],
including [staged installs], [redirection], or [command redefinition].
It is compatible with parallel builds (`-j#`).

[Makefile conventions]: https://www.gnu.org/prep/standards/html_node/Makefile-Conventions.html
[staged installs]: https://www.gnu.org/prep/standards/html_node/DESTDIR.html
[redirection]: https://www.gnu.org/prep/standards/html_node/Directory-Variables.html
[command redefinition]: https://www.gnu.org/prep/standards/html_node/Utilities-in-Makefiles.html

### Building LZ4 - Using vcpkg

You can download and install LZ4 using the [vcpkg](https://github.com/Microsoft/vcpkg) dependency manager:

    git clone https://github.com/Microsoft/vcpkg.git
    cd vcpkg
    ./bootstrap-vcpkg.sh
    ./vcpkg integrate install
    vcpkg install lz4

The LZ4 port in vcpkg is kept up to date by Microsoft team members and community contributors. If the version is out of date, please [create an issue or pull request](https://github.com/Microsoft/vcpkg) on the vcpkg repository.

Documentation
-------------------------

The raw LZ4 block compression format is detailed within [lz4_Block_format].

Arbitrarily long files or data streams are compressed using multiple blocks,
for streaming requirements. These blocks are organized into a frame,
defined into [lz4_Frame_format].
Interoperable versions of LZ4 must also respect the frame format.

[lz4_Block_format]: doc/lz4_Block_format.md
[lz4_Frame_format]: doc/lz4_Frame_format.md


Other source versions
-------------------------

Beyond the C reference source,
many contributors have created versions of lz4 in multiple languages
(Java, C#, Python, Perl, Ruby, etc.).
A list of known source ports is maintained on the [LZ4 Homepage].

[LZ4 Homepage]: http://www.lz4.org
