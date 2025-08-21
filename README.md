# Goldman Sachs Hackathon

This document outlines three problems from GSIH'25.

---

## Problem 1: Bit Happens 

This problem involves solving a three-part digital forensics problem by decoding hidden patterns in various file formats.

### Subtask 1: Bitmap Steganography
* [cite_start]**Problem**: Decode a Base64 encoded BMP file to extract a hidden number from an `ABC{...}` pattern[cite: 4].
* [cite_start]**Approach**: The process involves decoding the Base64 string into a BMP file structure[cite: 30]. [cite_start]The `PixelDataOffset` is read from the `BITMAPFILEHEADER` to locate the image payload[cite: 33, 44, 55]. [cite_start]The Least Significant Bit (LSB) is extracted from the blue channel of each pixel[cite: 4, 34, 65]. [cite_start]These bits are reassembled into bytes to form a message, from which the numeric pattern is extracted[cite: 4, 58].

| Block | Field | Width | Description |
| :--- | :--- | :--- | :--- |
| **BITMAPFILEHEADER** | `FileType` | 2 bytes | [cite_start]ASCII string, must be 'BM'[cite: 16, 38, 39]. |
| (Width: 14 bytes) [cite_start][cite: 50] | `FileSize` | 4 bytes | [cite_start]Entire file size in bytes[cite: 41, 42]. |
| | `Reserved` | 2 bytes | [cite_start]Unused, initialized to 0[cite: 46, 47, 49]. |
| | `Reserved` | 2 bytes | [cite_start]Unused, initialized to 0[cite: 52, 53, 54]. |
| | `PixelDataOffset` | 4 bytes | [cite_start]Offset to the start of the pixel data (payload)[cite: 55, 56, 57]. |

### Subtask 2: PCAP Pattern Search
* [cite_start]**Problem**: Decode a Base64 PCAP blob, find a potentially large number within an `ABC{...}` pattern, and compute `(n % 10007) + 3`[cite: 5]. [cite_start]If no pattern is found, the result is 0[cite: 5].
* [cite_start]**Approach**: The decoded PCAP data is treated as a single text blob[cite: 9]. [cite_start]A regular expression is used to find the first occurrence of the `ABC{...}` pattern[cite: 5, 8, 20]. [cite_start]The extracted number is handled using Java's `BigInteger` to support its large size [cite: 10][cite_start], and the final value is computed[cite: 68].

### Subtask 3: MysticLang VM Simulation
* [cite_start]**Problem**: Simulate a custom 8-bit virtual machine to retrieve a byte from a specific memory address after the program halts[cite: 6].
* [cite_start]**Approach**: The Base64 hex bytecode is decoded and parsed into opcodes[cite: 6]. [cite_start]A VM is initialized with 256 bytes of memory and 16 registers, all set to zero[cite: 11, 22]. [cite_start]The simulator fetches and executes instructions until the `HALT` opcode (0xFF) is encountered[cite: 23, 29]. [cite_start]The value at the requested memory address is then returned[cite: 73].

### [cite_start]Technology Selection: Java [cite: 70]
* [cite_start]**Reasoning**: Java was chosen for its strong byte-level control and endianness handling via `ByteBuffer` [cite: 75][cite_start], as well as its high performance for executing loops over millions of VM operations[cite: 76].

### Limitations & Improvements
1.  [cite_start]**No IP/TCP/UDP Reassembly**[cite: 78]:
    * [cite_start]**Current Fix**: The solution applies regex directly on the raw, reassembled PCAP blob decoded as ISO-8859-1 text[cite: 9, 78].
    * [cite_start]**Future Fix**: Integrate a proper packet capture library (like jNetPcap) to reassemble packet fragments before searching[cite: 78].
2.  [cite_start]**VM Stack Pointer Underflow**[cite: 79]:
    * [cite_start]**Current Fix**: The stack pointer (`%r15`) starts at 0 [cite: 11] [cite_start]and underflow wraps around due to `& 0xFF` masking[cite: 79, 80].
    * [cite_start]**Future Fix**: Initialize `%r15` to `0xFF` and add boundary checks to handle stack overflow/underflow explicitly[cite: 81].
3.  [cite_start]**Inefficient BMP LSB Scanning**[cite: 83]:
    * [cite_start]**Current Fix**: The entire bitstream from the BMP is collected before the regex search begins[cite: 84].
    * [cite_start]**Future Fix**: Implement an early exit to stop bit extraction as soon as the `ABC{...}` pattern is successfully matched[cite: 85, 86].

---

## Problem 2: Barcode Parsing - Code 39 Decoder

[cite_start]This problem decodes a Code 39 barcode from an image[cite: 89].

### Approach
[cite_start]The image processing pipeline involves several stages to handle real-world constraints like poor lighting and skewed angles[cite: 94].

1.  [cite_start]**Rotation Handling**: The input image is processed to correct for minor tilts[cite: 92, 113].
2.  [cite_start]**Grayscale & Thresholding**: The image is converted to grayscale, and Otsu's thresholding algorithm is applied to create a binary (black and white) image[cite: 98, 99, 110].
3.  [cite_start]**Scanline Processing**: A few rows of pixels near the image's center are scanned to find the barcode pattern[cite: 100, 115].
4.  [cite_start]**Width Encoding**: The widths of the black and white bars are measured and classified as narrow (N) or wide (W)[cite: 102].
5.  **Bidirectional Decoding**: The pattern is decoded into characters. [cite_start]The scan can be done in both directions to handle images that may be upside down[cite: 105].

### Example Character Decoding
| Widths (9) | Pattern (N/W) | Char |
| :--- | :--- | :--- |
| [3,1,2,1,3,4,5,2,4] | NNNNNWWNW | G |
| [3,1,4,1,3,2,5,4,2] | NNWNNNWWN | S |
| [3,1,4,5,3,2,2,1,4] | NNWWNNNNW | 2 |
| [3,1,3,5,4,2,4,1,2] | NNNWWNWNN | 0 |
| [3,1,4,5,3,2,2,1,4] | NNWWNNNNW | 2 |
| [5,1,2,5,4,2,2,1,3] | WNNWWNNNN | 5 |
[cite_start][cite: 97]

### [cite_start]Technology Selection: Python [cite: 104]
* [cite_start]**Reasoning**: Python was chosen for its fast prototyping capabilities and powerful imaging libraries like PIL and NumPy, which are ideal for iterative, logic-heavy pixel processing[cite: 107].

### Limitations & Improvements
1.  [cite_start]**Extreme Blur/Skew**[cite: 109]:
    * [cite_start]**Current Fix**: Otsu's thresholding provides some robustness[cite: 110].
    * [cite_start]**Future Fix**: Implement deblurring filters or morphological operations to enhance edges[cite: 111].
2.  [cite_start]**Significant Tilt (> ±5°)**[cite: 112]:
    * [cite_start]**Current Fix**: The current implementation handles only small-angle rotations[cite: 113].
    * [cite_start]**Future Fix**: Use a Hough-line transform to detect the barcode's angle and perform a full deskew[cite: 113].
3.  [cite_start]**Barcode Position and Size**[cite: 114]:
    * [cite_start]**Current Fix**: The algorithm only checks a few rows around the center of the image[cite: 115].
    * [cite_start]**Future Fix**: Implement a sliding-window search with early pruning to find barcodes anywhere in the image[cite: 115].

---

## Problem 3: Automated Intersection Detection in 2D Line Graphs

[cite_start]This problem counts the number of line-line intersections in a 512x512 JPEG image of a 2D graph, excluding the graph's axes[cite: 119].

### Approach
The detection method is based on a multi-scale windowing heuristic.

1.  [cite_start]**Preprocessing**: The input image is converted to grayscale and then binarized[cite: 132, 137].
2.  [cite_start]**Intersection Detection**: The algorithm slides `3x3`, `5x5`, and `7x7` windows across every pixel of the binary image[cite: 140].
3.  **Heuristic Validation**: A point is considered an intersection only if it satisfies a set of pixel density rules across the different window sizes. [cite_start]For example, a `7x7` window must not contain too many pixels (`<=30`) to reject thick blobs and axes[cite: 123, 155, 159]. [cite_start]This confirms a true crossing of thin lines[cite: 145].
4.  [cite_start]**Merging**: Detections that are within a 6-pixel radius of each other are merged to prevent counting a single intersection multiple times[cite: 136, 148, 149].

### Assumptions
* [cite_start]Axes are thicker than data lines (>= 3px)[cite: 150].
* [cite_start]Valid intersection points do not occur within a 6px radius of each other[cite: 152].
* [cite_start]L-shaped patterns are not considered intersections[cite: 153].

### Limitations & Improvements
1.  [cite_start]**Heuristic Threshold Sensitivity**[cite: 121]:
    * [cite_start]**Current Fix**: Thresholds are manually tuned for 512x512 images[cite: 122].
    * [cite_start]**Future Fix**: Implement adaptive or local thresholding methods (e.g., Sauvola or Otsu) for broader applicability[cite: 127, 128, 129].
2.  [cite_start]**Fixed Clustering Radius**[cite: 133]:
    * [cite_start]**Current Fix**: A static 6px merge radius is used[cite: 136].
    * [cite_start]**Future Fix**: Use connected-component labeling to find centroids or a dynamic radius based on local line density[cite: 156, 157].
3.  [cite_start]**Axis Handling**[cite: 158]:
    * [cite_start]**Current Fix**: Axes are implicitly ignored by skipping a 3px border and using the `7x7 <= 30` pixel rule[cite: 158, 159].
    * [cite_start]**Future Fix**: Explicitly detect and remove axes using techniques like the Hough transform or longest-line removal[cite: 160, 161].

### [cite_start]Alternative Solution: Graph Skeletonization [cite: 162]
* [cite_start]A more robust, general-purpose method involves using morphological thinning to reduce the graph lines to a 1-pixel wide skeleton[cite: 164].
* [cite_start]This skeleton can be treated as a mathematical graph, where intersection points are simply nodes with a degree of 3 or more[cite: 163, 165].
* [cite_start]This approach scales well to more complex diagrams and network maps[cite: 166].
